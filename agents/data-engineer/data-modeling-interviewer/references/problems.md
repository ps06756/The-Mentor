# Problem Bank

## Problem 1: E-Commerce Star Schema

**Business Context**:
An e-commerce marketplace connects buyers and sellers:
- 10,000 sellers, 2 million buyers, 500,000 products
- 50,000 orders/day with an average of 3 items per order
- Business needs: revenue dashboards, seller performance scorecards, buyer behavior analysis
- Analysts use Looker for dashboards and ad-hoc SQL for deep dives

**Your Task**:
Design a star schema that supports these business questions:
1. Total revenue by product category, by month
2. Top 10 sellers by revenue in each region
3. Average order value by customer cohort (signup month)
4. Return rate by product and seller
5. Year-over-year revenue growth by category

---

**Schema Design**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    E-COMMERCE STAR SCHEMA                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  FACT TABLE:                                                             │
│  fact_order_items                                                        │
│    Grain: one row per product per order                                  │
│    ┌──────────────────────────────────────────┐                          │
│    │ order_item_key      (surrogate PK)       │                          │
│    │ date_key            (FK -> dim_date)      │                          │
│    │ buyer_key           (FK -> dim_buyer)     │                          │
│    │ seller_key          (FK -> dim_seller)    │                          │
│    │ product_key         (FK -> dim_product)   │                          │
│    │ order_id            (degenerate dimension)│                          │
│    │ quantity            (additive fact)        │                          │
│    │ unit_price          (semi-additive fact)   │                          │
│    │ discount_amount     (additive fact)        │                          │
│    │ shipping_cost       (additive fact)        │                          │
│    │ net_revenue         (derived: qty * price │                          │
│    │                       - discount)          │                          │
│    │ is_returned         (boolean flag)         │                          │
│    │ return_date_key     (FK -> dim_date, null) │                          │
│    └──────────────────────────────────────────┘                          │
│                                                                          │
│  DIMENSION TABLES:                                                       │
│                                                                          │
│  dim_date                          dim_buyer                             │
│  ┌──────────────────┐              ┌──────────────────┐                  │
│  │ date_key (PK)    │              │ buyer_key (PK)   │                  │
│  │ full_date        │              │ buyer_id         │                  │
│  │ day_of_week      │              │ buyer_name       │                  │
│  │ day_name         │              │ email            │                  │
│  │ month_number     │              │ signup_date      │                  │
│  │ month_name       │              │ signup_cohort    │                  │
│  │ quarter          │              │   (YYYY-MM)      │                  │
│  │ year             │              │ city             │                  │
│  │ fiscal_quarter   │              │ state            │                  │
│  │ is_weekend       │              │ country          │                  │
│  │ is_holiday       │              │ lifetime_orders  │                  │
│  └──────────────────┘              └──────────────────┘                  │
│                                                                          │
│  dim_seller                        dim_product                           │
│  ┌──────────────────┐              ┌──────────────────┐                  │
│  │ seller_key (PK)  │              │ product_key (PK) │                  │
│  │ seller_id        │              │ product_id       │                  │
│  │ seller_name      │              │ product_name     │                  │
│  │ signup_date      │              │ category_name    │                  │
│  │ city             │              │ subcategory_name │                  │
│  │ state            │              │ brand_name       │                  │
│  │ region           │              │ current_price    │                  │
│  │ seller_tier      │              │ weight_kg        │                  │
│  │   (gold/silver)  │              │ is_active        │                  │
│  └──────────────────┘              └──────────────────┘                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Sample Queries**:

```sql
-- Q1: Revenue by category by month
SELECT
    d.year,
    d.month_name,
    p.category_name,
    SUM(f.net_revenue) AS total_revenue
FROM fact_order_items f
JOIN dim_date d ON f.date_key = d.date_key
JOIN dim_product p ON f.product_key = p.product_key
GROUP BY d.year, d.month_name, p.category_name
ORDER BY d.year, d.month_number, total_revenue DESC;

-- Q2: Top 10 sellers by revenue per region
WITH ranked AS (
    SELECT
        s.region,
        s.seller_name,
        SUM(f.net_revenue) AS total_revenue,
        ROW_NUMBER() OVER (PARTITION BY s.region ORDER BY SUM(f.net_revenue) DESC) AS rn
    FROM fact_order_items f
    JOIN dim_seller s ON f.seller_key = s.seller_key
    GROUP BY s.region, s.seller_name
)
SELECT * FROM ranked WHERE rn <= 10;

-- Q3: Average order value by signup cohort
SELECT
    b.signup_cohort,
    AVG(order_total) AS avg_order_value
FROM (
    SELECT f.order_id, MAX(b.signup_cohort) AS signup_cohort, SUM(f.net_revenue) AS order_total
    FROM fact_order_items f
    JOIN dim_buyer b ON f.buyer_key = b.buyer_key
    GROUP BY f.order_id
) sub
GROUP BY signup_cohort
ORDER BY signup_cohort;

-- Q4: Return rate by product and seller
SELECT
    p.product_name,
    s.seller_name,
    COUNT(*) AS total_items,
    SUM(CASE WHEN f.is_returned THEN 1 ELSE 0 END) AS returned_items,
    ROUND(SUM(CASE WHEN f.is_returned THEN 1 ELSE 0 END) * 100.0 / COUNT(*), 2) AS return_rate_pct
FROM fact_order_items f
JOIN dim_product p ON f.product_key = p.product_key
JOIN dim_seller s ON f.seller_key = s.seller_key
GROUP BY p.product_name, s.seller_name
HAVING COUNT(*) >= 100
ORDER BY return_rate_pct DESC;
```

---

## Problem 2: SaaS Subscription Model

**Business Context**:
A B2B SaaS company with 5,000 customers needs analytics on:
- Monthly Recurring Revenue (MRR) trends
- Churn rate and churn reasons
- Expansion and contraction revenue
- Cohort analysis (retention by signup month)
- Account manager performance

Source data: subscriptions table (with status history), customers, plans, invoices, and a subscription_events log.

**Task**: Design a model optimized for subscription analytics.

---

**Schema Design**:
```
FACT TABLE: fact_subscription_monthly
  Grain: one row per subscription per calendar month

  ┌────────────────────────────────────────────────┐
  │ snapshot_key           (surrogate PK)           │
  │ month_key              (FK -> dim_month)         │
  │ subscription_key       (FK -> dim_subscription)  │
  │ customer_key           (FK -> dim_customer)      │
  │ plan_key               (FK -> dim_plan)          │
  │ mrr_amount             (monthly revenue)          │
  │ arr_amount             (annualized revenue)       │
  │ status                 (active|churned|paused)    │
  │ months_since_start     (running count)            │
  │ mrr_change_amount      (vs previous month)        │
  │ change_category        (new|expansion|contraction │
  │                         |churn|reactivation|none) │
  └────────────────────────────────────────────────┘

DIMENSION TABLES:

  dim_month                         dim_customer
  ┌──────────────────┐              ┌──────────────────────┐
  │ month_key (PK)   │              │ customer_key (PK)    │
  │ month_start      │              │ customer_id          │
  │ month_end        │              │ company_name         │
  │ quarter          │              │ industry             │
  │ year             │              │ employee_count_band  │
  │ fiscal_quarter   │              │ signup_date          │
  └──────────────────┘              │ signup_cohort        │
                                    │ account_manager      │
  dim_plan                          │ region               │
  ┌──────────────────┐              └──────────────────────┘
  │ plan_key (PK)    │
  │ plan_name        │              dim_subscription
  │ tier             │              ┌──────────────────────┐
  │ billing_period   │              │ subscription_key (PK)│
  │ list_price       │              │ subscription_id      │
  │ is_current       │              │ start_date           │
  └──────────────────┘              │ end_date             │
                                    │ original_plan        │
                                    │ current_plan         │
                                    └──────────────────────┘
```

---

**Building the Monthly Snapshot**:
```sql
-- How to build fact_subscription_monthly from event data:

-- Step 1: Generate a spine of all months
WITH months AS (
    SELECT DISTINCT DATE_TRUNC('month', d) AS month_start
    FROM generate_series('2022-01-01', CURRENT_DATE, INTERVAL '1 month') d
),

-- Step 2: Get subscription state at each month-end
subscription_state AS (
    SELECT
        m.month_start,
        s.subscription_id,
        s.customer_id,
        s.plan_id,
        s.mrr_amount,
        s.status,
        DATEDIFF('month', s.start_date, m.month_start) AS months_since_start
    FROM months m
    CROSS JOIN subscriptions s
    WHERE s.start_date <= m.month_start
      AND (s.end_date IS NULL OR s.end_date > m.month_start)
),

-- Step 3: Calculate month-over-month changes
with_changes AS (
    SELECT
        *,
        mrr_amount - LAG(mrr_amount) OVER (
            PARTITION BY subscription_id ORDER BY month_start
        ) AS mrr_change_amount,
        CASE
            WHEN LAG(subscription_id) OVER (
                PARTITION BY subscription_id ORDER BY month_start
            ) IS NULL THEN 'new'
            WHEN mrr_amount > LAG(mrr_amount) OVER (
                PARTITION BY subscription_id ORDER BY month_start
            ) THEN 'expansion'
            WHEN mrr_amount < LAG(mrr_amount) OVER (
                PARTITION BY subscription_id ORDER BY month_start
            ) THEN 'contraction'
            ELSE 'none'
        END AS change_category
    FROM subscription_state
)

SELECT * FROM with_changes;
```

---

**Key Metrics Queries**:
```sql
-- Total MRR by month
SELECT month_start, SUM(mrr_amount) AS total_mrr
FROM fact_subscription_monthly
WHERE status = 'active'
GROUP BY month_start ORDER BY month_start;

-- MRR breakdown by change category
SELECT
    month_start,
    SUM(CASE WHEN change_category = 'new' THEN mrr_amount END) AS new_mrr,
    SUM(CASE WHEN change_category = 'expansion' THEN mrr_change_amount END) AS expansion_mrr,
    SUM(CASE WHEN change_category = 'contraction' THEN mrr_change_amount END) AS contraction_mrr,
    SUM(CASE WHEN change_category = 'churn' THEN mrr_change_amount END) AS churned_mrr
FROM fact_subscription_monthly
GROUP BY month_start;

-- Cohort retention (percentage of original cohort still active)
SELECT
    c.signup_cohort,
    f.months_since_start,
    COUNT(DISTINCT CASE WHEN f.status = 'active' THEN f.subscription_key END) AS active_subs,
    COUNT(DISTINCT CASE WHEN f.months_since_start = 0 THEN f.subscription_key END)
        OVER (PARTITION BY c.signup_cohort) AS cohort_size,
    ROUND(active_subs * 100.0 / cohort_size, 1) AS retention_pct
FROM fact_subscription_monthly f
JOIN dim_customer c ON f.customer_key = c.customer_key
GROUP BY c.signup_cohort, f.months_since_start
ORDER BY c.signup_cohort, f.months_since_start;
```

---

## Problem 3: Medallion Architecture Migration

**Business Context**:
A company is migrating 50 dbt models from Snowflake to Databricks with Delta Lake:
- Current state: staging models, intermediate models, mart models (star schemas)
- Data scientists need access to cleaned but non-aggregated data for ML
- Some models need hourly refresh, most need daily
- Semi-structured data (JSON logs) needs to be incorporated

**Task**: Design the medallion layer architecture and map existing models.

---

**Solution**:
```
Layer Mapping:

┌─────────────────────────────────────────────────────────────────┐
│  CURRENT (Snowflake + dbt)    │  TARGET (Databricks + Delta)   │
├───────────────────────────────┼─────────────────────────────────┤
│  sources (raw schemas)         │  BRONZE layer                  │
│  staging models (stg_*)        │  SILVER layer (cleaned)        │
│  intermediate models (int_*)   │  SILVER layer (conformed)      │
│  mart models (fct_*, dim_*)    │  GOLD layer                    │
│  (no equivalent)               │  GOLD layer (agg_* summaries)  │
└───────────────────────────────┴─────────────────────────────────┘

BRONZE Layer:
  Purpose: Raw data landing zone
  Format: Delta tables (append-only)
  Schema: Matches source exactly (schema-on-read for JSON)
  Refresh: Continuous or hourly via Auto Loader
  Retention: 90 days (configurable per source)
  Tables:
    bronze.raw_orders        -- from PostgreSQL CDC
    bronze.raw_customers     -- from PostgreSQL CDC
    bronze.raw_events        -- from Kafka (JSON logs)
    bronze.raw_payments      -- from Stripe API

SILVER Layer:
  Purpose: Cleaned, typed, conformed data
  Format: Delta tables (merge/upsert)
  Schema: Strongly typed, documented columns
  Refresh: Hourly (incremental merge)
  Used by: Data scientists (ML features), analytics engineers
  Tables:
    silver.orders            -- cleaned, deduplicated, typed
    silver.customers         -- SCD Type 2 (track changes)
    silver.events            -- JSON exploded into columns
    silver.payments          -- normalized, reconciled

GOLD Layer:
  Purpose: Business-level models for BI
  Format: Delta tables, partitioned and Z-ordered
  Schema: Star schemas (fact + dimension tables)
  Refresh: Hourly (key dashboards) or daily (standard)
  Used by: Analysts, dashboards, reports
  Tables:
    gold.fact_orders         -- grain: order line item
    gold.fact_events         -- grain: user event
    gold.dim_customer        -- SCD Type 2
    gold.dim_product         -- current state
    gold.dim_date            -- static reference
    gold.agg_daily_revenue   -- pre-aggregated for fast dashboards
    gold.agg_cohort_metrics  -- pre-computed cohort analysis
```

---

**Physical Optimization**:
```sql
-- Partitioning: use for date-based queries (reduces data scanned)
CREATE TABLE gold.fact_orders
USING DELTA
PARTITIONED BY (order_date)
AS SELECT * FROM silver.orders_transformed;

-- Z-ordering: use for high-cardinality filters
OPTIMIZE gold.fact_orders ZORDER BY (customer_id, product_id);

-- File compaction: run after many small writes
OPTIMIZE gold.fact_orders WHERE order_date >= '2024-01-01';

-- Vacuum: remove old file versions (saves storage)
VACUUM gold.fact_orders RETAIN 168 HOURS;

-- Caching: for frequently queried tables
CACHE SELECT * FROM gold.agg_daily_revenue WHERE date >= DATEADD(month, -3, CURRENT_DATE);
```

---

## Problem 4: Many-to-Many Relationship Modeling

**Business Context**:
In your e-commerce model, a product can now belong to multiple categories (e.g., a "Yoga Mat" is in both "Fitness" and "Home"). Your current model has category_name as a column on dim_product, which only supports one category per product.

**Task**: Redesign the model to support many-to-many while keeping queries simple.

**Solution**:
```
Option A: Bridge Table (Recommended for complex analysis)

  dim_product                bridge_product_category      dim_category
  ┌──────────────┐           ┌───────────────────┐        ┌──────────────┐
  │ product_key  │◄──────────┤ product_key       │        │ category_key │
  │ product_name │           │ category_key      ├───────►│ category_name│
  │ brand        │           │ weight_factor     │        │ parent_cat   │
  └──────────────┘           │ is_primary        │        └──────────────┘
                             └───────────────────┘

  weight_factor: Allocates revenue proportionally
    - Yoga Mat: Fitness (0.6), Home (0.4)
    - Prevents double-counting in revenue reports

  Query (revenue by category, no double-counting):
    SELECT c.category_name, SUM(f.net_revenue * b.weight_factor) AS revenue
    FROM fact_order_items f
    JOIN bridge_product_category b ON f.product_key = b.product_key
    JOIN dim_category c ON b.category_key = c.category_key
    GROUP BY c.category_name;

Option B: Array Column (Simpler, if warehouse supports it)

  dim_product
  ┌──────────────────────────┐
  │ product_key              │
  │ product_name             │
  │ primary_category         │
  │ categories (ARRAY)       │  -- ['Fitness', 'Home']
  └──────────────────────────┘

  Query (Snowflake):
    SELECT c.value AS category, SUM(f.net_revenue) AS revenue
    FROM fact_order_items f
    JOIN dim_product p ON f.product_key = p.product_key,
    LATERAL FLATTEN(p.categories) c
    GROUP BY c.value;
    -- Note: This WILL double-count revenue for multi-category products

Option C: Primary Category Only (Simplest, if acceptable)

  Keep category_name on dim_product but define it as the PRIMARY category.
  Add a separate reporting table for "products by all categories" analysis.
  Most dashboards only need the primary category.
```

---

## Problem 5: Slowly Changing Dimension Implementation

**Business Context**:
Your dim_customer table needs to track historical changes. When a customer moves from New York to California, historical orders should still show "New York" as the location at the time of purchase, but current reports should show "California."

**Task**: Implement SCD Type 2 with proper surrogate keys and effective dates.

**Solution**:
```
SCD Type 2 Implementation:

dim_customer (Type 2)
┌─────────────────────────────────────────────────────┐
│ customer_key          (surrogate PK, auto-increment) │
│ customer_id           (natural key / business key)   │
│ customer_name                                        │
│ email                                                │
│ city                                                 │
│ state                                                │
│ country                                              │
│ segment                                              │
│ effective_date        (when this version started)    │
│ expiration_date       (when this version ended)      │
│ is_current            (boolean: TRUE for latest)     │
└─────────────────────────────────────────────────────┘

Example data:
  customer_key | customer_id | name  | state | effective    | expiration   | is_current
  1001         | C-42        | Alice | NY    | 2023-01-15   | 2024-06-01   | FALSE
  1002         | C-42        | Alice | CA    | 2024-06-01   | 9999-12-31   | TRUE

How fact table links:
  - fact_order_items.customer_key = 1001 (orders placed while in NY)
  - fact_order_items.customer_key = 1002 (orders placed after move to CA)

MERGE logic for updates:
  -- Step 1: Expire old rows
  UPDATE dim_customer
  SET expiration_date = CURRENT_DATE, is_current = FALSE
  WHERE customer_id IN (SELECT customer_id FROM staging_customers WHERE has_changed = TRUE)
    AND is_current = TRUE;

  -- Step 2: Insert new rows
  INSERT INTO dim_customer (customer_id, customer_name, city, state, country, segment,
                            effective_date, expiration_date, is_current)
  SELECT customer_id, customer_name, city, state, country, segment,
         CURRENT_DATE, '9999-12-31', TRUE
  FROM staging_customers
  WHERE has_changed = TRUE;

Common pitfalls:
  1. Late-arriving facts: Order arrives for customer_key that has been expired
     Solution: Look up correct customer_key using order_date BETWEEN effective_date AND expiration_date

  2. Multiple changes in one day: Customer moves twice in one batch
     Solution: Use timestamp instead of date for effective/expiration, or keep only latest

  3. Dimension table growth: Popular customer has 50 versions
     Solution: Only track changes to analytically relevant columns (state, segment),
               not irrelevant ones (last_login_date)
```

---

## Session Flow Example

**You**: "Welcome! Let's dive into data modeling today. I'd like to start with a foundational question: What's normalization? And when would you choose to denormalize?"

**Candidate**: "Normalization eliminates redundancy by splitting data into related tables..."

**You**: "Right, that's the textbook answer. Now, in a data warehouse context, why do most analytics models intentionally denormalize? What's the practical benefit when an analyst is writing a query?"

[Let candidate think through the join cost and query simplicity trade-off]

**You**: "Good thinking. Let's put this into practice. Here's the scenario: [present e-commerce problem]. Start by telling me: what is the grain of your fact table?"

[Guide through schema design, probing on dimension choices and fact types]

**You**: "Solid model. Now, an analyst writes this query to get revenue by category and it takes 45 seconds on 2 billion rows. What modeling techniques could speed this up?"

[Explore partitioning, pre-aggregation, materialized views]

**You**: "One more twist: the business now says a product can belong to multiple categories. How does that change your model?"

[Explore bridge tables, array columns, and double-counting prevention]
