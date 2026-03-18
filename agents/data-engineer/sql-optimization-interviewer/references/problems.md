# Problem Bank: SQL Optimization

## Problem 1: Index Strategy Design

### Scenario
E-commerce analytics dashboard needs:
1. Search orders by customer email
2. Filter orders by date range and status
3. Sort by total amount
4. Join with products for category analysis

### Table Schema
```sql
CREATE TABLE orders (
  order_id BIGINT PRIMARY KEY,
  customer_id INT NOT NULL,
  customer_email VARCHAR(255),
  order_date TIMESTAMP,
  status VARCHAR(50),
  total_amount DECIMAL(10,2),
  product_id INT,
  shipping_address JSONB
);
```

### Questions
1. What indexes would you create? Why?
2. What trade-offs do indexes have?
3. How would you handle the LIKE query on email (e.g., '%@gmail.com')?

### Solution Walkthrough

**Step 1: Analyze Access Patterns**
- Customer lookups: WHERE customer_id = ? ORDER BY order_date DESC
- Dashboard filtering: WHERE order_date BETWEEN ? AND ? AND status = ?
- Sorting: ORDER BY total_amount DESC
- Email search: WHERE customer_email LIKE '%@gmail.com'

**Step 2: Design Indexes**
```sql
-- For customer lookups (composite: filter + sort in one index)
CREATE INDEX idx_orders_customer ON orders (customer_id, order_date DESC);

-- For dashboard filtering (composite index, column order matters!)
-- Most selective column first if all are equality checks
-- Range column last
CREATE INDEX idx_orders_date_status ON orders (order_date, status, total_amount DESC);

-- Partial index for active orders only (saves space!)
CREATE INDEX idx_orders_pending ON orders (order_date)
WHERE status IN ('pending', 'processing');

-- For email search - consider trigram index for LIKE patterns
CREATE INDEX idx_orders_email_trgm ON orders USING gin (customer_email gin_trgm_ops);
```

**Step 3: Trade-off Analysis**

| Trade-off | Impact |
|-----------|--------|
| Write amplification | Each INSERT/UPDATE must update all indexes |
| Storage | Each index adds ~10-30% of table size |
| Maintenance | VACUUM/REINDEX overhead |
| Planner time | More indexes = more plans to evaluate |

**Back-of-Envelope: Index Size**
- 100M orders, each order_id is 8 bytes
- Simple B-tree index: ~8 bytes key + ~8 bytes pointer = 16 bytes per entry
- 100M * 16 bytes = 1.6 GB per single-column index
- Composite index (3 columns): ~40 bytes per entry = 4 GB
- 5 indexes on 100M rows: ~10-15 GB total index storage

---

## Problem 2: Query Rewrite for Performance

### Slow Query (takes 2 minutes)
```sql
SELECT
  DATE(created_at) as day,
  COUNT(*) as order_count,
  SUM(total_amount) as revenue
FROM orders
WHERE created_at >= '2024-01-01'
GROUP BY DATE(created_at)
ORDER BY day;
```

### Issues
1. `DATE(created_at)` function on column prevents index usage
2. No index on `created_at`
3. Open-ended range scan (no upper bound)

### Solution Walkthrough

**Step 1: Add Index**
```sql
CREATE INDEX idx_orders_created_at ON orders (created_at);
```

**Step 2: Rewrite Query**
```sql
-- Optimized: bounded range + index-friendly grouping
SELECT
  date_trunc('day', created_at) as day,
  COUNT(*) as order_count,
  SUM(total_amount) as revenue
FROM orders
WHERE created_at >= '2024-01-01'
  AND created_at < '2025-01-01'  -- Add upper bound!
GROUP BY date_trunc('day', created_at)
ORDER BY day;
```

**Step 3: For even better performance, use a covering index**
```sql
-- Covering index: DB can answer query from index alone (Index-Only Scan)
CREATE INDEX idx_orders_covering_daily
ON orders (created_at, total_amount);
```

**Step 4: For extreme performance, pre-aggregate**
```sql
-- Materialized view for daily aggregates
CREATE MATERIALIZED VIEW daily_revenue AS
SELECT
  date_trunc('day', created_at) as day,
  COUNT(*) as order_count,
  SUM(total_amount) as revenue
FROM orders
GROUP BY date_trunc('day', created_at);

-- Refresh nightly or on-demand
REFRESH MATERIALIZED VIEW CONCURRENTLY daily_revenue;
```

**Performance Comparison:**
| Approach | Time (100M rows) | Notes |
|----------|-------------------|-------|
| Original query (no index) | ~120s | Full table scan |
| With index + bounded range | ~5s | Index range scan |
| With covering index | ~2s | Index-only scan |
| Materialized view | ~50ms | Pre-computed |

**Alternative approach using range queries:**
```sql
-- Or use range queries which definitely use indexes
SELECT
  '2024-01-01'::date as day,
  COUNT(*) as order_count,
  SUM(total_amount) as revenue
FROM orders
WHERE created_at >= '2024-01-01'
  AND created_at < '2024-01-02'
UNION ALL
SELECT '2024-01-02'::date, ...
```

---

## Problem 3: Star Schema Design

### Business Requirement
Build a data warehouse for an e-commerce company with:
- 100M orders per year
- Need to analyze by: time, customer demographics, product category, geography
- Support complex aggregations and trend analysis

### Task
Design star schema with fact and dimension tables.

### Solution

```sql
-- Fact table (measurements)
CREATE TABLE fact_orders (
  order_key BIGINT PRIMARY KEY,
  date_key INT REFERENCES dim_date(date_key),
  customer_key INT REFERENCES dim_customer(customer_key),
  product_key INT REFERENCES dim_product(product_key),
  geography_key INT REFERENCES dim_geography(geography_key),

  -- Measures
  quantity INT,
  unit_price DECIMAL(10,2),
  discount_amount DECIMAL(10,2),
  sales_amount DECIMAL(10,2),
  cost_amount DECIMAL(10,2)
);

-- Dimension tables (context)
CREATE TABLE dim_date (
  date_key INT PRIMARY KEY,
  full_date DATE,
  day_of_week INT,
  day_name VARCHAR(10),
  month_number INT,
  month_name VARCHAR(10),
  quarter INT,
  year INT,
  is_weekend BOOLEAN,
  is_holiday BOOLEAN
);

CREATE TABLE dim_customer (
  customer_key INT PRIMARY KEY,
  customer_id VARCHAR(50),
  first_name VARCHAR(100),
  last_name VARCHAR(100),
  email VARCHAR(255),
  registration_date DATE,
  customer_segment VARCHAR(50),
  lifetime_value_tier VARCHAR(20)
);

CREATE TABLE dim_product (
  product_key INT PRIMARY KEY,
  product_id VARCHAR(50),
  product_name VARCHAR(255),
  category VARCHAR(100),
  subcategory VARCHAR(100),
  brand VARCHAR(100),
  supplier_key INT
);

CREATE TABLE dim_geography (
  geography_key INT PRIMARY KEY,
  city VARCHAR(100),
  state VARCHAR(100),
  country VARCHAR(100),
  region VARCHAR(50),
  postal_code VARCHAR(20)
);
```

### Follow-up Questions and Answers

**Q1: Why star schema vs normalized schema?**
- Star schema is optimized for reads (analytical queries)
- Fewer JOINs needed (fact -> dimension, never dimension -> dimension)
- Columnar databases (Redshift, BigQuery) are optimized for this pattern
- Normalized schema is better for OLTP (transactional) workloads

**Q2: How would you partition the fact table?**
```sql
-- Range partition by date for time-series queries
CREATE TABLE fact_orders (
  order_key BIGINT,
  date_key INT,
  ...
) PARTITION BY RANGE (date_key);

-- Monthly partitions
CREATE TABLE fact_orders_202401
PARTITION OF fact_orders
FOR VALUES FROM (20240101) TO (20240201);
```

**Q3: TYPE 1 vs TYPE 2 Slowly Changing Dimensions?**

| Type | Approach | Example | Trade-off |
|------|---------|---------|-----------|
| Type 1 | Overwrite old value | Customer changes email -> update in place | Loses history |
| Type 2 | Add new row with versioning | Customer changes segment -> new row with effective_date | Preserves history, more storage |

Type 2 example:
```sql
CREATE TABLE dim_customer (
  customer_key INT PRIMARY KEY,       -- Surrogate key (auto-increment)
  customer_id VARCHAR(50),            -- Natural key
  customer_segment VARCHAR(50),
  effective_date DATE,
  end_date DATE,                      -- NULL means current
  is_current BOOLEAN DEFAULT TRUE
);

-- Customer 'C001' was "Silver", became "Gold" on 2024-06-01:
-- Row 1: customer_key=1, customer_id='C001', segment='Silver', effective='2023-01-01', end='2024-05-31', is_current=FALSE
-- Row 2: customer_key=2, customer_id='C001', segment='Gold', effective='2024-06-01', end=NULL, is_current=TRUE
```

---

## Problem 4: Partitioning Strategy for Event Logs

### Scenario
Event logs table growing by 10M rows/day. Queries usually access last 7 days. Table currently has 3 billion rows and queries are taking minutes.

### Solution Walkthrough

**Step 1: Choose Partition Strategy**
- Range partitioning by event_time (daily partitions)
- Most queries filter by recent dates -> partition pruning eliminates 99%+ of data

**Step 2: Implement**
```sql
CREATE TABLE events (
  event_id BIGINT,
  event_time TIMESTAMP NOT NULL,
  user_id INT,
  event_type VARCHAR(50),
  data JSONB
) PARTITION BY RANGE (event_time);

-- Create daily partitions
CREATE TABLE events_20240101
PARTITION OF events
FOR VALUES FROM ('2024-01-01') TO ('2024-01-02');

CREATE TABLE events_20240102
PARTITION OF events
FOR VALUES FROM ('2024-01-02') TO ('2024-01-03');
-- ... automated via cron/pg_partman
```

**Step 3: Automate partition management**
```sql
-- Using pg_partman (PostgreSQL extension)
SELECT partman.create_parent(
  p_parent_table := 'public.events',
  p_control := 'event_time',
  p_type := 'range',
  p_interval := '1 day',
  p_premake := 7  -- Create 7 days ahead
);
```

**Step 4: Data lifecycle**
```sql
-- Drop old partitions instead of DELETE (instant, no vacuum needed)
DROP TABLE events_20231201;  -- Drops 10M rows instantly

-- vs DELETE (slow, generates WAL, requires vacuum)
DELETE FROM events WHERE event_time < '2023-12-02';  -- Takes hours on 10M rows
```

### Back-of-Envelope: Query Performance

**Without partitioning (3 billion rows):**
- Query: WHERE event_time > NOW() - INTERVAL '7 days'
- Even with index: 3B rows in B-tree = ~33 levels deep
- Range scan touches millions of pages
- Estimated: 30-120 seconds

**With daily partitions:**
- Query touches 7 partitions * 10M rows = 70M rows
- 70M / 3B = 2.3% of data scanned
- Estimated: 1-3 seconds (50-100x improvement)
- Partition pruning: planner eliminates 293+ partitions before execution

---

## Problem 5: EXPLAIN Plan Analysis

### Given EXPLAIN Output
```
Sort  (cost=189456.78..189706.78 rows=100000 width=45)
  Sort Key: total_amount DESC
  ->  Hash Join  (cost=3245.00..178932.00 rows=100000 width=45)
        Hash Cond: (o.customer_id = c.customer_id)
        ->  Seq Scan on orders o  (cost=0.00..164256.00 rows=100000 width=28)
              Filter: ((created_at >= '2024-01-01') AND (status = 'completed'))
              Rows Removed by Filter: 9900000
        ->  Hash  (cost=2245.00..2245.00 rows=80000 width=21)
              ->  Seq Scan on customers c  (cost=0.00..2245.00 rows=80000 width=21)
```

### Analysis Questions
1. What is the most expensive operation?
2. Why is the orders table doing a Seq Scan?
3. How many rows are scanned vs returned from orders?
4. What indexes would you add?

### Solution

**1. Most expensive operation:** Seq Scan on orders (cost=164256, scans 10M rows to return 100K)

**2. Why Seq Scan:** No index on (created_at, status). The optimizer decides a full table scan is cheaper than no index.

**3. Scan efficiency:** 10M scanned, 100K returned = 1% selectivity. 99% of rows are thrown away. This is a clear candidate for an index.

**4. Recommended indexes:**
```sql
-- Composite index matching the WHERE clause
CREATE INDEX idx_orders_created_status
ON orders (created_at, status);

-- Even better: include sort column to avoid separate sort step
CREATE INDEX idx_orders_created_status_amount
ON orders (created_at, status, total_amount DESC);

-- Best: covering index to enable Index-Only Scan
CREATE INDEX idx_orders_covering
ON orders (created_at, status, total_amount DESC)
INCLUDE (customer_id, order_id, quantity);
```

**Expected EXPLAIN after indexing:**
```
Limit  (cost=0.56..4532.10 rows=100 width=45)
  ->  Nested Loop  (cost=0.56..45321.00 rows=100000 width=45)
        ->  Index Scan Backward using idx_orders_created_status_amount on orders o
              Index Cond: ((created_at >= '2024-01-01') AND (status = 'completed'))
        ->  Index Scan using customers_pkey on customers c
              Index Cond: (customer_id = o.customer_id)
```

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hi! I'm the data engineering lead. Today we'll be talking about SQL optimization -- not just writing queries, but making them fast at scale. I care about *why* things are slow, not just how to fix them. Let's start."

### Phase 1: Warm-up (10 minutes)
**Interviewer**: "Walk me through what happens when you run `SELECT * FROM orders WHERE customer_id = 123`. Start from when the database receives the query."

*[Candidate discusses parser, optimizer, executor, storage engine]*

**Interviewer**: "Good. You mentioned the optimizer chooses an execution plan. What information does it use to decide between a sequential scan and an index scan?"

### Phase 2: Schema Design (20 minutes)
**Interviewer**: "We're building an analytics platform for an e-commerce company. 100 million orders per year. We need to analyze by time period, customer segment, product category, and geography. How would you design the schema?"

*[Candidate designs star schema]*

**Interviewer**: "You chose a star schema. Why not a normalized 3NF schema? What are the trade-offs?"

### Phase 3: Query Optimization (25 minutes)
**Interviewer**: "Here's a query that takes 45 seconds on 100M rows. Walk me through how you'd diagnose and fix it."

*[Present slow ETL query, discuss EXPLAIN output]*

**Interviewer**: "You added a composite index. Why did you put `created_at` before `status` in the index? Would it matter if we swapped them?"

### Phase 4: System Design Connection (5 minutes)
**Interviewer**: "This query feeds a daily dashboard. How would you architect this at scale? Would you run it directly against the production database?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*

---

## Warm-up and Follow-up Question Bank

### Warm-up Questions
- "Walk me through what happens when you run a SELECT query"
- "What's the difference between B-Tree and Hash indexes?"
- "When would you denormalize data?"

### Intermediate Questions
- "How would you diagnose a query that suddenly became slow after working fine for months?"
- "What's the difference between a covering index and a composite index?"
- "Explain the N+1 query problem and how to solve it"

### Advanced Questions
- "Design a partitioning strategy for an event logs table growing by 10M rows/day"
- "How would you optimize a query that joins five tables with 100M+ rows each?"
- "Explain how MVCC works and its impact on query performance"
