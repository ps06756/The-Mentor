# Problem Bank: Slow Database Debugging

## Problem 1: Missing Index After Table Migration

### Problem Statement
A query that returns orders for a specific customer used to take 50ms. After a table migration 3 weeks ago, it now takes 30 seconds. The table has 100M rows. The query, schema, and hardware are unchanged. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- Query: `SELECT * FROM orders WHERE customer_id = 12345 AND status = 'pending' ORDER BY created_at DESC LIMIT 10`
- EXPLAIN ANALYZE shows `Seq Scan` instead of `Index Scan`
- `\di orders*` shows no indexes on the orders table

**Root Cause:**
The migration used `CREATE TABLE orders_new AS SELECT * FROM orders`, which copies data but not indexes, constraints, triggers, or defaults. After `DROP TABLE orders; ALTER TABLE orders_new RENAME TO orders;`, the table had no indexes.

**Diagnosis Steps:**
1. Run `EXPLAIN ANALYZE` on the slow query: Seq Scan, 30 seconds
2. Check for indexes: `SELECT indexname FROM pg_indexes WHERE tablename = 'orders'` -> empty
3. Check migration script: found `CREATE TABLE ... AS SELECT` pattern
4. Confirm: the original table had `idx_orders_customer_status(customer_id, status, created_at)`

**Fix:**
```sql
-- Create the index concurrently (doesn't lock the table)
CREATE INDEX CONCURRENTLY idx_orders_customer_status
  ON orders(customer_id, status, created_at DESC);

-- Verify
EXPLAIN ANALYZE SELECT * FROM orders
  WHERE customer_id = 12345 AND status = 'pending'
  ORDER BY created_at DESC LIMIT 10;
-- Should show: Index Scan using idx_orders_customer_status
-- Time: ~5ms
```

**Prevention:**
- Migration scripts must recreate all indexes, constraints, and triggers
- Add CI check: compare `pg_indexes` before and after migration, fail if any are missing
- Use `ALTER TABLE` for schema changes when possible (avoids full table rebuild)
- Maintain an "index inventory" document reviewed during migration planning

### Follow-up Questions
- "Why `CREATE INDEX CONCURRENTLY` instead of `CREATE INDEX`?"
- "What if the table is being written to heavily during index creation?"
- "How would you design the migration to avoid losing indexes in the first place?"

---

## Problem 2: Stale Statistics Causing Bad Query Plans

### Problem Statement
An index exists on the orders table, but the query planner chooses a Seq Scan for a query that should use the index. The EXPLAIN output shows `estimated rows = 100` but `actual rows = 47,823`. What's wrong?

### Solution Walkthrough

**Initial Clues:**
- EXPLAIN ANALYZE output:
  ```
  Seq Scan on orders (cost=0.00..2847392.00 rows=100 width=245)
    (actual time=12000..30200 rows=47823 loops=1)
    Filter: (customer_id = 12345 AND status = 'pending')
    Rows Removed by Filter: 99952177
  ```
- Note: `rows=100` (estimated) vs `rows=47823` (actual) -- 478x off!

**Root Cause:**
Table statistics are stale. The last `ANALYZE` ran when the table had 1M rows. A bulk import added 99M rows, but autovacuum's ANALYZE threshold wasn't triggered because `autovacuum_analyze_threshold` was set too high for this table size.

**Diagnosis Steps:**
1. Check statistics freshness:
   ```sql
   SELECT relname, last_analyze, last_autoanalyze, n_live_tup
   FROM pg_stat_user_tables WHERE relname = 'orders';
   -- last_analyze: 2025-12-01 (3 months ago!)
   -- n_live_tup: 1,200,000 (stale -- actual is 100M)
   ```
2. The planner thinks there are only 1.2M rows, so it estimates 100 matches for this customer
3. With 100 estimated rows, Seq Scan appears cheaper than Index Scan (wrong!)
4. After ANALYZE: planner estimates 48,000 rows, chooses Index Scan

**Fix:**
```sql
ANALYZE orders;

-- Verify the plan changed
EXPLAIN ANALYZE SELECT * FROM orders
  WHERE customer_id = 12345 AND status = 'pending'
  ORDER BY created_at DESC LIMIT 10;
-- Now shows: Index Scan, ~5ms
```

**Prevention:**
- Run `ANALYZE` after every bulk data load
- Tune autovacuum settings for large tables:
  ```sql
  ALTER TABLE orders SET (
    autovacuum_analyze_scale_factor = 0.01,  -- ANALYZE after 1% change (default is 10%)
    autovacuum_analyze_threshold = 10000
  );
  ```
- Add monitoring: alert if `last_analyze` is older than 24 hours for tables > 10M rows
- Include `ANALYZE` in ETL and bulk import pipelines

### Follow-up Questions
- "What exactly are table statistics? Where are they stored?"
- "Why does the planner sometimes choose Seq Scan even when an index exists?"
- "How does `random_page_cost` affect this decision?"

---

## Problem 3: Lock Contention from Zombie Transaction

### Problem Statement
All queries on the orders table are slow -- not just one query. Simple `SELECT` statements that should take 5ms are taking 10-30 seconds. The query plans look optimal. The database CPU and memory are fine. What's happening?

### Solution Walkthrough

**Initial Clues:**
- ALL queries on orders are slow, not just complex ones
- Simple primary key lookups are also slow: `SELECT * FROM orders WHERE id = 1` -> 12 seconds
- `pg_stat_activity` shows many queries in `Lock` wait state
- Database CPU: 15%, Memory: 40%, Disk I/O: normal

**Root Cause:**
A developer opened a psql session 4 hours ago for debugging, ran `BEGIN; UPDATE orders SET status = 'test' WHERE id = 999;`, and forgot to `COMMIT` or `ROLLBACK`. This transaction holds a `RowExclusiveLock` on the orders table. Every other transaction that needs to modify orders (and some that just read, depending on isolation level) is blocked.

**Diagnosis Steps:**
1. Check for blocked queries:
   ```sql
   SELECT pid, state, query, age(clock_timestamp(), query_start) AS duration
   FROM pg_stat_activity
   WHERE wait_event_type = 'Lock' AND relation::regclass::text = 'orders';
   ```
2. Find the blocking transaction:
   ```sql
   SELECT blocked_locks.pid AS blocked_pid,
          blocking_locks.pid AS blocking_pid,
          blocking_activity.query AS blocking_query,
          age(clock_timestamp(), blocking_activity.xact_start) AS blocking_duration
   FROM pg_locks blocked_locks
   JOIN pg_locks blocking_locks ON blocked_locks.locktype = blocking_locks.locktype
   JOIN pg_stat_activity blocking_activity ON blocking_locks.pid = blocking_activity.pid
   WHERE NOT blocked_locks.granted;
   ```
3. Result: PID 54321 has been `idle in transaction` for 4 hours

**Fix:**
```sql
-- Terminate the zombie transaction
SELECT pg_terminate_backend(54321);
-- All blocked queries immediately complete
```

**Prevention:**
- Set `idle_in_transaction_session_timeout = '5min'` in `postgresql.conf`
- Add monitoring alert: any transaction open > 10 minutes
- Restrict direct database access: use connection poolers (PgBouncer) with statement timeouts
- Require read-only replicas for debugging and data exploration

### Follow-up Questions
- "What's the difference between a lock and a deadlock?"
- "Could this happen with read-only queries?"
- "How does MVCC in PostgreSQL relate to this problem?"

---

## Sample Session Flow

### Opening
**Interviewer**: "Hey, I'm the DBA for this platform. I've been tuning queries since PostgreSQL 8.4, and I've seen every way a database can go slow. Today we've got a fun one. A query that used to be snappy is now bringing the whole app to its knees. Walk me through how you'd figure out what happened."

### Phase 1: Symptom (5 minutes)
**Interviewer**: "Here's what we know: `SELECT * FROM orders WHERE customer_id = 12345 AND status = 'pending' ORDER BY created_at DESC LIMIT 10` went from 50ms to 30 seconds. Table grew from 1M to 100M rows. Same query, same schema. What do you want to look at first?"

### Phase 2: Query Plan (15 minutes)
*[Candidate asks for EXPLAIN ANALYZE]*
**Interviewer**: "Good instinct. Here's the output..."
*[Reveal query plan showing Seq Scan with bad estimates]*
**Interviewer**: "What does this tell you? Walk me through each line."

### Phase 3: Fix (15 minutes)
**Interviewer**: "You've identified the issue. How do you fix it? And I don't just mean 'make this query fast' -- I mean 'make sure this never happens again.'"

### Phase 4: Scaling (10 minutes)
**Interviewer**: "The table is growing by 5M rows per month. In a year it'll be 160M rows. What's the long-term plan?"

*[Generate scorecard]*
