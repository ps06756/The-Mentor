# Problem Bank: Data Inconsistency Debugging

## Problem 1: Timezone Mismatch Between UTC and Local Time

### Problem Statement
The revenue dashboard shows $1.2M for March. Finance shows $1.05M. Part of the discrepancy ($45,000) is caused by a timezone issue. The database stores timestamps in UTC. Finance reports in US Eastern Time. Find the exact mismatch and fix it.

### Solution Walkthrough

**The Timezone Trap:**
```
Database:  event_timestamp is stored in UTC
Dashboard: WHERE event_timestamp >= '2026-03-01' AND event_timestamp < '2026-04-01'
           This is March 1 00:00 UTC to March 31 23:59:59 UTC

Finance:   March 1 00:00 ET to March 31 23:59:59 ET
           = March 1 05:00 UTC to April 1 03:59:59 UTC (ET = UTC-5 / UTC-4 DST)
```

**The Overlap:**
```
Dashboard includes but Finance excludes:
  March 1 00:00 UTC to March 1 04:59 UTC (= Feb 28 7pm-11:59pm ET)
  These are February transactions in ET but March in UTC

Dashboard excludes but Finance includes:
  March 31 20:00 ET to March 31 23:59 ET (= April 1 00:00-03:59 UTC)
  These are March transactions in ET but April in UTC
```

**Investigation Query:**
```sql
-- Transactions in dashboard's March but Finance's February
SELECT SUM(amount) FROM transactions
WHERE event_timestamp >= '2026-03-01 00:00:00 UTC'
  AND event_timestamp < '2026-03-01 05:00:00 UTC';
-- Result: $12,000 (included in dashboard, excluded by Finance)

-- Transactions in Finance's March but dashboard's April
SELECT SUM(amount) FROM transactions
WHERE event_timestamp >= '2026-04-01 00:00:00 UTC'
  AND event_timestamp < '2026-04-01 04:00:00 UTC';
-- Result: $33,000 (excluded from dashboard, included by Finance)

-- Net dashboard overcount: $12,000 - (-$33,000) = not quite right
-- Actually: Dashboard has $12K extra from Feb-ET and misses $33K from Mar-ET
-- But the net effect depends on the direction of the discrepancy
-- Dashboard is $150K HIGHER, so the timezone contributes $45K (complex multi-day overlap)
```

**Fix:**
```sql
-- Correct dashboard query: convert to reporting timezone first
SELECT SUM(amount) FROM transactions
WHERE event_timestamp AT TIME ZONE 'US/Eastern' >= '2026-03-01'
  AND event_timestamp AT TIME ZONE 'US/Eastern' < '2026-04-01';
```

**Prevention:**
- Standardize: all financial queries use `AT TIME ZONE 'US/Eastern'`
- Create a view: `revenue_by_month_et` that pre-converts timezones
- Add data quality test: dashboard total must match Stripe monthly summary within 0.5%
- Document the reporting timezone convention in the data team's style guide

### Follow-up Questions
- "What about Daylight Saving Time? Does the offset change mid-March?"
- "Should the database store timestamps in ET instead of UTC?"
- "How would you handle a business that operates across multiple timezones?"

---

## Problem 2: Duplicate Webhook Events

### Problem Statement
Stripe webhooks are delivered at-least-once, meaning some events arrive more than once. The pipeline's deduplication was accidentally disabled 2 weeks ago. Find the duplicate transactions and quantify the revenue overcount.

### Solution Walkthrough

**Discovery:**
```sql
-- Find duplicate events
SELECT event_id, COUNT(*) as occurrences, SUM(amount) as total_amount
FROM transactions
WHERE event_date >= '2026-03-01' AND event_date < '2026-04-01'
GROUP BY event_id
HAVING COUNT(*) > 1
ORDER BY occurrences DESC;

-- Results:
-- 1,200 event_ids with duplicates
-- Most are counted twice, some three times
-- Total overcount: $80,000
```

**Root Cause Investigation:**
```sql
-- When did duplicates start appearing?
SELECT DATE(created_at), COUNT(*) - COUNT(DISTINCT event_id) as duplicate_count
FROM transactions
WHERE event_date >= '2026-02-01'
GROUP BY DATE(created_at)
ORDER BY DATE(created_at);

-- Results:
-- Feb 1 - Mar 3: 0 duplicates per day (dedup working)
-- Mar 4 onwards: 50-120 duplicates per day (dedup broken)
-- Mar 4 corresponds to the migration date
```

**Fix (Immediate):**
```sql
-- Remove duplicates, keeping the first occurrence
DELETE FROM transactions
WHERE id NOT IN (
    SELECT MIN(id) FROM transactions GROUP BY event_id
);
-- Removed: 1,200 rows, $80,000 overcount eliminated
```

**Fix (Long-term):**
```sql
-- Re-enable dedup: add unique constraint
ALTER TABLE transactions ADD CONSTRAINT uq_event_id UNIQUE (event_id);

-- Idempotent insert pattern
INSERT INTO transactions (event_id, amount, ...)
VALUES ('evt_123', 100.00, ...)
ON CONFLICT (event_id) DO NOTHING;
```

**Prevention:**
- Add data quality alert: `duplicate_rate = duplicates / total > 0.1%` triggers PagerDuty
- Migration checklist: verify all constraints and indexes after migration
- Weekly reconciliation job comparing pipeline count vs Stripe event count

### Follow-up Questions
- "What if some duplicates were already aggregated into downstream reports?"
- "How do you handle duplicate detection for events without a unique event_id?"
- "What's the difference between at-least-once and exactly-once delivery?"

---

## Problem 3: Refunds Not Reflected in Revenue Pipeline

### Problem Statement
Finance subtracts refunds from gross revenue. The dashboard pipeline only counts charges, not refunds. March had $25,000 in refunds. Quantify the impact and fix the pipeline.

### Solution Walkthrough

**Discovery:**
```sql
-- Dashboard calculation (current - wrong):
SELECT SUM(amount) as gross_revenue FROM transactions
WHERE event_type = 'charge.succeeded'
  AND event_date >= '2026-03-01' AND event_date < '2026-04-01';
-- Result: $1,200,000 (after fixing timezone and duplicates, this becomes $1,075,000)

-- Missing refunds:
SELECT SUM(amount) as total_refunds FROM transactions
WHERE event_type = 'refund.created'
  AND event_date >= '2026-03-01' AND event_date < '2026-04-01';
-- Result: $25,000

-- Correct net revenue:
-- $1,075,000 - $25,000 = $1,050,000 (matches Finance!)
```

**Fix:**
```sql
-- Correct dashboard query:
SELECT
    SUM(CASE WHEN event_type = 'charge.succeeded' THEN amount ELSE 0 END)
  - SUM(CASE WHEN event_type = 'refund.created' THEN amount ELSE 0 END)
  AS net_revenue
FROM transactions
WHERE event_date AT TIME ZONE 'US/Eastern' >= '2026-03-01'
  AND event_date AT TIME ZONE 'US/Eastern' < '2026-04-01';
-- Result: $1,050,000 (matches Finance!)
```

**Prevention:**
- Clearly label dashboard metrics: "Gross Revenue" vs "Net Revenue"
- Pipeline should always compute both gross and net
- Monthly reconciliation: compare pipeline net revenue vs Stripe's payout report
- Add dbt test: `net_revenue = gross_revenue - refunds - chargebacks`

### Follow-up Questions
- "What about partial refunds? How do you handle those?"
- "Should chargebacks also be included? What about disputes?"
- "How would you handle refunds that cross month boundaries (charged in Feb, refunded in March)?"

---

## Problem 4: Combined Multi-Factor Discrepancy

### Problem Statement
The full $150,000 discrepancy is caused by three factors simultaneously. Decompose the discrepancy into its component parts and prove that they account for the full difference.

### Solution Walkthrough

**Reconciliation Table:**

| Factor | Dashboard Impact | Amount |
|--------|-----------------|--------|
| Timezone mismatch (UTC vs ET) | Dashboard includes extra transactions | +$45,000 |
| Duplicate webhook events | Events counted multiple times | +$80,000 |
| Refunds not subtracted | Dashboard shows gross, Finance shows net | +$25,000 |
| **Total Discrepancy** | | **+$150,000** |

**Corrected Dashboard:** $1,200,000 - $45,000 - $80,000 - $25,000 = $1,050,000 = Finance number

**CFO-Friendly Explanation:**
"The dashboard was showing $150K more than the true number for three reasons: (1) A timezone conversion bug included some April transactions in the March count ($45K). (2) A software migration two weeks ago caused some transactions to be counted twice ($80K). (3) The dashboard showed gross revenue without subtracting refunds, while Finance correctly uses net revenue ($25K). We've fixed all three issues and the numbers now match."

### Follow-up Questions
- "How do you ensure this reconciliation is correct?"
- "Should you present this to the board with an error bar?"
- "How do you prevent multi-factor discrepancies from happening again?"

---

## Sample Session Flow

### Opening
**Interviewer**: "Hey, I just got pulled out of a meeting by the CFO. The revenue dashboard says $1.2M for March. Finance says $1.05M. The board meeting is in 3 hours and we need to figure out which number is right and why they're different. You're the best data debugger I know. Help me."

### Phase 1: Discrepancy (5 minutes)
**Interviewer**: "Here are the two numbers. $150K difference. Where do you start?"

*[Candidate lists hypotheses]*

### Phase 2: Investigation (20 minutes)
**Interviewer**: "Good hypotheses. Let's start with the timezone one. What query would you run?"

*[Walk through each hypothesis with SQL results]*

### Phase 3: Resolution (10 minutes)
**Interviewer**: "You found all three causes. They add up to $150K. Now I need you to explain this to the CFO in 2 minutes. Go."

### Phase 4: Prevention (10 minutes)
**Interviewer**: "Great. How do we make sure this never happens again?"

*[Generate scorecard]*
