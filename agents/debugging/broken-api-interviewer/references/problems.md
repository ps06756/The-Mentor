# Problem Bank: Broken API Debugging

## Problem 1: Connection Pool Exhaustion from Slow Dependency

### Problem Statement
The checkout API is returning 500 errors for 30% of requests. The database CPU is at 20% (healthy), but the application logs show `PoolExhaustedException: Timeout waiting for idle object`. The error started 2 hours ago after deploying v2.4.0. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- Database is healthy (low CPU, normal query times)
- Connection pool is maxed out (20/20 active, 0 idle)
- Average connection hold time: 5.2 seconds (was 80ms before deploy)

**Root Cause:**
The v2.4.0 deploy added a new synchronous call to `inventory-service` inside the checkout flow. This call happens while holding a database connection. The inventory-service has intermittent latency spikes (p99 = 5 seconds), causing the DB connection to be held for the duration.

**Diagnosis Steps:**
1. Check connection pool metrics: `active=20, idle=0, pending=847`
2. Check thread dump: 20 threads blocked on `inventoryService.checkStock()`
3. Check inventory-service latency: p50=50ms, p99=5200ms
4. Correlate: The deploy added `inventoryService.checkStock()` inside the DB transaction

**Fix (Immediate):**
```
1. Move inventory check OUTSIDE the database transaction
2. Add 500ms timeout to inventory-service call
3. Add circuit breaker: if inventory-service fails, allow checkout (check inventory async)
```

**Fix (Long-term):**
- Connection pool monitoring with alerts at 80% utilization
- Mandatory timeouts on all outbound HTTP calls (enforced by lint rule)
- Load testing that includes slow dependency simulation
- Separate connection pools for critical vs non-critical paths

### Follow-up Questions
- "What if the inventory service is down completely, not just slow?"
- "How do you size a connection pool? What factors matter?"
- "What's the difference between connection pool exhaustion and database overload?"

---

## Problem 2: Null Pointer from Breaking API Contract Change

### Problem Statement
After deploying v2.4.0, exactly 30% of checkout requests fail with `NullPointerException` at `PaymentProcessor.java:142`. The other 70% succeed fine. The error rate is perfectly stable at 30%. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- Error rate is exactly 30%, not fluctuating -- suggests a deterministic condition
- Stack trace points to `discountMetadata.getType()` -- a new field
- The field was added in the v2.4.0 PR

**Root Cause:**
The new code accesses `payment.getDiscountMetadata().getType()` without null checking. The `discountMetadata` field is only populated when a coupon code is applied. Approximately 30% of checkouts don't use coupons, so `discountMetadata` is null for those requests.

**Diagnosis Steps:**
1. Read the stack trace: NPE at `discountMetadata.getType()`
2. Check the git diff for v2.4.0: new code accesses `discountMetadata` without null check
3. Check data: `SELECT COUNT(*) FROM checkouts WHERE coupon_code IS NULL` -> 30.2%
4. Correlation confirmed: error rate matches non-coupon checkout rate

**Fix (Immediate):**
```java
// Before (broken):
String type = payment.getDiscountMetadata().getType();

// After (fixed):
String type = Optional.ofNullable(payment.getDiscountMetadata())
    .map(DiscountMetadata::getType)
    .orElse("NONE");
```

**Fix (Long-term):**
- Add contract tests between checkout and payment service
- Make `discountMetadata` explicitly `@Nullable` in the API schema
- Add static analysis rule: no method calls on potentially null objects without checks
- Require integration tests that cover both coupon and non-coupon checkout paths

### Follow-up Questions
- "How would you catch this in code review?"
- "What's the difference between this being a client bug vs a server bug?"
- "If you can't deploy a fix immediately, what's a faster mitigation?"

---

## Problem 3: Database Deadlock Under Concurrent Writes

### Problem Statement
Some checkout requests hang for exactly 30 seconds, then fail with a database error. The frequency increases during peak traffic. Requests that succeed are fast (< 200ms). The error logs show `ERROR: deadlock detected`. Diagnose and fix.

### Solution Walkthrough

**Initial Clues:**
- Requests either succeed quickly or fail after exactly 30 seconds (lock timeout)
- `deadlock detected` in PostgreSQL logs
- Frequency correlates with traffic volume (more concurrency = more deadlocks)
- Started after v2.4.0 deploy

**Root Cause:**
The v2.4.0 deploy changed the checkout flow to update the `orders` table before the `inventory` table. The payment webhook handler (unchanged) still updates `inventory` before `orders`. Under concurrent traffic, Transaction A holds a lock on `orders` and waits for `inventory`, while Transaction B holds `inventory` and waits for `orders` -- classic deadlock.

**Diagnosis Steps:**
1. Check PostgreSQL logs: `LOG: process 12345 detected deadlock while waiting for ShareLock on transaction 67890`
2. Run `SELECT * FROM pg_stat_activity WHERE wait_event_type = 'Lock'`
3. Identify the two conflicting queries and their lock ordering
4. Check git diff: the checkout flow now does `UPDATE orders` then `UPDATE inventory` (was reversed)
5. Check webhook handler: still does `UPDATE inventory` then `UPDATE orders`

**Deadlock Visualization:**
```
Transaction A (Checkout):          Transaction B (Webhook):
1. UPDATE orders SET ...           1. UPDATE inventory SET ...
   -> Acquires lock on orders         -> Acquires lock on inventory
2. UPDATE inventory SET ...        2. UPDATE orders SET ...
   -> BLOCKED (B holds lock)          -> BLOCKED (A holds lock)
   -> DEADLOCK!                       -> DEADLOCK!
```

**Fix (Immediate):**
```sql
-- Ensure all transactions lock tables in the same order:
-- Always: inventory FIRST, then orders
BEGIN;
  UPDATE inventory SET quantity = quantity - 1 WHERE product_id = 123;
  UPDATE orders SET status = 'confirmed' WHERE order_id = 456;
COMMIT;
```

**Fix (Long-term):**
- Document lock ordering convention: always acquire locks alphabetically by table name
- Add integration test that runs checkout and webhook handler concurrently
- Add deadlock detection alert: if `deadlock detected` appears in DB logs, page on-call
- Consider using optimistic locking (version columns) instead of pessimistic locks

### Follow-up Questions
- "How is a deadlock different from lock contention?"
- "What are the tradeoffs between optimistic and pessimistic locking?"
- "How would you handle this if the two transactions are in different microservices?"

---

## Sample Session Flow

### Opening
**Interviewer**: "Hey, I just got paged. The checkout API is on fire -- 30% 5xx rate, revenue is tanking. I've got the Datadog dashboard open and I need another pair of eyes. Let's figure this out. What's the first thing you want to look at?"

### Phase 1: Triage (10 minutes)
*[Candidate asks about recent changes]*
**Interviewer**: "Yeah, we deployed v2.4.0 about two hours ago. Changelog says it added discount code support to checkout. Want to see the error logs?"

*[Reveal log snippet with mixed errors]*

### Phase 2: Narrowing Down (15 minutes)
**Interviewer**: "Good, you spotted the two different error types. Let's focus on one. Which do you want to chase first, and why?"

*[Based on candidate's choice, drill into that root cause]*

### Phase 3: Fix (10 minutes)
**Interviewer**: "OK, you found it. Revenue is still dropping. What's your immediate fix? Walk me through exactly what you'd do right now."

### Phase 4: Prevention (10 minutes)
**Interviewer**: "Fire's out. What goes in the postmortem? What process changes stop this class of bug from ever shipping?"

*[Generate scorecard]*
