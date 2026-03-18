# Problem Bank

## Problem 1: E-Commerce Sales Schema

**Business Context**:
Design a data warehouse for a large e-commerce company (Amazon-scale):
- 100M products across 50 categories
- 1M orders/day, 5M order line items/day
- Products can have multiple sellers (marketplace model)
- Promotions/coupons applied at order or line item level
- Returns happen up to 30 days after purchase
- Need to support: Revenue reporting, seller analytics, promotion effectiveness

**Your Task**:
Design the schema following this structure:

---

**Business Requirements**:
```
┌─────────────────────────────────────────────────────────────┐
│                    BUSINESS REQUIREMENTS                     │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  Key Metrics to Support:                                     │
│   • Gross Merchandise Value (GMV) by day/week/month          │
│   • Revenue by product category and seller                   │
│   • Promotion effectiveness (lift vs control)                │
│   • Return rate by product and category                      │
│   • Seller performance (fulfillment time, ratings)           │
│                                                              │
│  Query Patterns:                                             │
│   • Time-series aggregation (90% of queries)                 │
│   • Slice by geography, category, seller (80% of queries)    │
│   • Drill-down from category → product (60% of queries)      │
│   • Return/refund analysis (20% of queries)                  │
│                                                              │
│  Data Volume:                                                │
│   • 5M order line items/day = 1.8B/year                     │
│   • 5-year retention required                                │
│   • ~9B fact table rows total                               │
│                                                              │
│  SCD Considerations:                                         │
│   • Product prices change frequently (daily sales)           │
│   • Sellers change their fulfillment settings                │
│   • Categories get reorganized yearly                        │
│   • Customer addresses change (shipping/billing)             │
│                                                              │
└─────────────────────────────────────────────────────────────┘
```

---

**Fact and Dimension Tables**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         DIMENSION TABLES                                 │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  dim_product (SCD Type 2 for price, Type 1 for description)             │
│  ─────────────────────────────────────────────────────────              │
│  • product_sk (PK, surrogate)                                            │
│  • product_id (natural key)                                              │
│  • product_name                                                          │
│  • category_fk → dim_category                                            │
│  • brand_fk → dim_brand                                                  │
│  • base_price                                                            │
│  • current_status (active/discontinued)                                  │
│  • start_date, end_date, is_current (SCD Type 2)                         │
│                                                                          │
│  dim_category (SCD Type 2 - reorganizations happen)                     │
│  ─────────────────────────────────────────────                          │
│  • category_sk (PK)                                                      │
│  • category_id                                                           │
│  • category_name                                                         │
│  • parent_category_fk (self-reference for hierarchy)                     │
│  • category_level (1=department, 2=category, 3=subcategory)              │
│  • start_date, end_date, is_current                                      │
│                                                                          │
│  dim_seller (SCD Type 2 for settings)                                   │
│  ─────────────────────────────────────                                  │
│  • seller_sk (PK)                                                        │
│  • seller_id                                                             │
│  • seller_name                                                           │
│  • fulfillment_type (FBA/MFN)                                            │
│  • seller_rating                                                         │
│  • start_date, end_date, is_current                                      │
│                                                                          │
│  dim_customer (SCD Type 2 for address)                                  │
│  ─────────────────────────────────────                                  │
│  • customer_sk (PK)                                                      │
│  • customer_id                                                           │
│  • customer_name                                                         │
│  • email                                                                 │
│  • shipping_city, shipping_state, shipping_country                       │
│  • customer_segment (new/returning/VIP)                                  │
│  • first_purchase_date                                                   │
│  • start_date, end_date, is_current                                      │
│                                                                          │
│  dim_date (Standard date dimension)                                     │
│  ─────────────────────────────────                                      │
│  • date_sk (PK, format: YYYYMMDD)                                        │
│  • full_date                                                             │
│  • day_of_week, day_of_month, day_of_year                               │
│  • week_of_year, month, quarter, year                                   │
│  • is_weekend, is_holiday, fiscal_year, fiscal_quarter                  │
│                                                                          │
│  dim_promotion (SCD Type 1 - promotions are temporal, not historical)   │
│  ─────────────────────────────────────────────────────────────────      │
│  • promotion_sk (PK)                                                     │
│  • promotion_id                                                          │
│  • promotion_name                                                        │
│  • promotion_type (percentage, fixed amount, BOGO)                       │
│  • discount_value                                                        │
│  • start_date, end_date                                                  │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────────────────────────────┐
│                           FACT TABLES                                    │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  fct_order_line_items (Primary Transaction Fact)                        │
│  ───────────────────────────────────────────────                        │
│  GRAIN: One row per order line item                                      │
│  ─────────────────────────────────                                       │
│  • order_line_item_sk (PK)                                               │
│  • order_id (degenerate dimension)                                       │
│  • line_item_number (1, 2, 3...)                                         │
│                                                                          │
│  -- Foreign Keys to Dimensions                                           │
│  • order_date_fk → dim_date                                              │
│  • product_fk → dim_product                                              │
│  • seller_fk → dim_seller                                                │
│  • customer_fk → dim_customer                                            │
│  • promotion_fk → dim_promotion (nullable)                               │
│                                                                          │
│  -- Degenerate Dimensions                                                │
│  • order_status (pending, shipped, delivered, returned)                  │
│                                                                          │
│  -- Measures (all additive)                                              │
│  • quantity                                                              │
│  • unit_price (price at time of sale - snapshot)                         │
│  • discount_amount                                                       │
│  • sales_amount (quantity × unit_price - discount)                       │
│  • tax_amount                                                            │
│  • shipping_amount                                                       │
│  • total_amount (sales + tax + shipping)                                 │
│  • estimated_cost (for gross margin calc)                                │
│                                                                          │
│  -- Audit/Metadata                                                       │
│  • etl_load_date                                                         │
│  • source_system                                                         │
│                                                                          │
│  PARTITION BY: order_date (monthly partitions)                           │
│  CLUSTER BY: category_fk (derived from product join)                     │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  fct_daily_inventory (Periodic Snapshot Fact)                           │
│  ────────────────────────────────────────────                           │
│  GRAIN: One row per product per seller per day                           │
│  ─────────────────────────────────────────────                           │
│  • snapshot_date_fk → dim_date                                           │
│  • product_fk → dim_product                                              │
│  • seller_fk → dim_seller                                                │
│  • warehouse_fk → dim_warehouse                                          │
│                                                                          │
│  -- Measures                                                             │
│  • quantity_on_hand                                                      │
│  • quantity_reserved                                                     │
│  • quantity_available                                                    │
│  • days_of_inventory (calculated)                                        │
│                                                                          │
│  Use case: Inventory trend analysis without scanning all transactions    │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  fct_returns (Transaction Fact - separate or consolidated?)              │
│  ─────────────────────────────────────────────────────────────           │
│  DECISION: Consolidate into fct_order_line_items with return flags       │
│                                                                          │
│  Alternative: Separate fact table if return analysis is primary use case │
│  • return_date_fk → dim_date                                             │
│  • original_order_fk (reference to original sale)                        │
│  • return_reason_fk → dim_return_reason                                  │
│  • return_amount                                                         │
│  • restocking_fee                                                        │
│  • refund_amount                                                         │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**SCD Strategy**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      SCD STRATEGY BY DIMENSION                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Dimension        │ SCD Type │ Attributes Tracked │ Attributes Overwritten│
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_product      │ Type 2   │ base_price         │ product_name,         │
│                   │          │                    │ description, images   │
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_category     │ Type 2   │ parent_category,   │ category_name         │
│                   │          │ category_level     │ (minor updates)       │
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_seller       │ Type 2   │ fulfillment_type,  │ seller_name           │
│                   │          │ seller_rating      │                       │
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_customer     │ Type 2   │ shipping_address   │ email, phone          │
│                   │          │ (city, state)      │                       │
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_promotion    │ Type 1   │ N/A (no history    │ all attributes        │
│                   │          │ needed)            │                       │
│  ─────────────────┼──────────┼────────────────────┼──────────────────────│
│  dim_date         │ N/A      │ Static dimension   │ N/A                   │
│                                                                          │
│  SCD Type 2 Implementation Details:                                      │
│  ─────────────────────────────────────                                   │
│  • Use surrogate keys (product_sk) for fact table FKs                    │
│  • Include start_date, end_date, is_current columns                      │
│  • end_date = '9999-12-31' for current records                           │
│  • Update logic: Close old record (set end_date, is_current=N)           │
│                  Insert new record (new SK, start_date=today)            │
│                                                                          │
│  Special Case: Product Price History                                     │
│  • Price changes daily during sales → lots of Type 2 rows                │
│  • Alternative: Store price in fact table (snapshot at sale time)        │
│  • We do both: dim_product.price for current, fact for historical        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Sample Queries**:

```sql
-- Query 1: Daily GMV by Category (Time-series aggregation)
-- Expected: < 5 seconds for 1 year of data
-- Optimization: Partition pruning on order_date, clustering on category

SELECT
    d.full_date,
    c.category_name,
    SUM(f.sales_amount) as gmv,
    COUNT(DISTINCT f.order_id) as order_count
FROM fct_order_line_items f
JOIN dim_date d ON f.order_date_fk = d.date_sk
JOIN dim_product p ON f.product_fk = p.product_sk
JOIN dim_category c ON p.category_fk = c.category_sk
WHERE d.full_date BETWEEN '2023-01-01' AND '2023-12-31'
  AND c.is_current = 'Y'  -- Use current category hierarchy
GROUP BY 1, 2
ORDER BY 1, 2;

-- Query 2: Promotion Effectiveness (Lift analysis)
-- Compare sales during promotion vs baseline

WITH promoted_sales AS (
    SELECT
        p.promotion_name,
        SUM(f.sales_amount) as revenue,
        SUM(f.quantity) as units_sold
    FROM fct_order_line_items f
    JOIN dim_promotion p ON f.promotion_fk = p.promotion_sk
    WHERE f.order_date_fk BETWEEN 20231101 AND 20231130
    GROUP BY 1
),
baseline_sales AS (
    SELECT
        AVG(daily_revenue) as avg_daily_revenue
    FROM (
        SELECT
            order_date_fk,
            SUM(sales_amount) as daily_revenue
        FROM fct_order_line_items
        WHERE promotion_fk IS NULL  -- Non-promoted sales
          AND order_date_fk BETWEEN 20230901 AND 20231031
        GROUP BY 1
    )
)
SELECT
    ps.promotion_name,
    ps.revenue,
    ps.units_sold,
    bs.avg_daily_revenue * 30 as baseline_revenue,
    (ps.revenue - bs.avg_daily_revenue * 30) / (bs.avg_daily_revenue * 30) * 100 as lift_pct
FROM promoted_sales ps
CROSS JOIN baseline_sales bs;

-- Query 3: Return Rate by Category
-- Note: Returns stored as negative amounts in fact table

SELECT
    c.category_name,
    COUNT(CASE WHEN f.order_status = 'returned' THEN 1 END) as return_count,
    COUNT(*) as total_orders,
    COUNT(CASE WHEN f.order_status = 'returned' THEN 1 END) * 100.0 / COUNT(*) as return_rate_pct
FROM fct_order_line_items f
JOIN dim_product p ON f.product_fk = p.product_sk
JOIN dim_category c ON p.category_fk = c.category_sk
WHERE f.order_date_fk >= 20230101
GROUP BY 1
ORDER BY 4 DESC;

-- Query 4: Cohort Analysis (Customer retention)
-- Group customers by first purchase month, track retention

WITH first_purchases AS (
    SELECT
        customer_fk,
        MIN(order_date_fk) as first_purchase_date
    FROM fct_order_line_items
    GROUP BY 1
),
cohort_activity AS (
    SELECT
        fp.customer_fk,
        DATE_TRUNC('month', fp.first_purchase_date) as cohort_month,
        DATE_TRUNC('month', f.order_date_fk) as activity_month,
        PERIOD_DIFF(
            DATE_FORMAT(f.order_date_fk, '%Y%m'),
            DATE_FORMAT(fp.first_purchase_date, '%Y%m')
        ) as months_since_first
    FROM fct_order_line_items f
    JOIN first_purchases fp ON f.customer_fk = fp.customer_fk
    GROUP BY 1, 2, 3, 4
)
SELECT
    cohort_month,
    months_since_first,
    COUNT(DISTINCT customer_fk) as active_customers,
    COUNT(DISTINCT customer_fk) * 100.0 /
        FIRST_VALUE(COUNT(DISTINCT customer_fk)) OVER (
            PARTITION BY cohort_month
            ORDER BY months_since_first
        ) as retention_pct
FROM cohort_activity
GROUP BY 1, 2
ORDER BY 1, 2;
```

---

**Performance Considerations**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                      PERFORMANCE OPTIMIZATION STRATEGY                   │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. PARTITIONING STRATEGY                                                │
│  ─────────────────────────                                               │
│  Table                    │ Partition Key      │ Granularity │ Reason   │
│  ─────────────────────────┼────────────────────┼─────────────┼──────────│
│  fct_order_line_items     │ order_date         │ Monthly     │ Time-    │
│                           │                    │             │ series   │
│                           │                    │             │ queries  │
│  fct_daily_inventory      │ snapshot_date      │ Monthly     │ Same     │
│  dim_product              │ start_date         │ Yearly      │ SCD rows │
│                                                                          │
│  2. CLUSTERING/SORTKEYS                                                  │
│  ────────────────────────                                                │
│  Table                    │ Cluster Columns                   │ Benefit │
│  ─────────────────────────┼───────────────────────────────────┼─────────│
│  fct_order_line_items     │ category_fk, seller_fk            │ Filter  │
│                           │                                   │ pruning │
│  dim_product              │ category_fk, brand_fk             │ Join    │
│                           │                                   │ optimization│
│                                                                          │
│  3. PRE-AGGREGATED TABLES (Rollups)                                      │
│  ─────────────────────────────────                                       │
│  Dashboard queries were too slow on base fact table (9B rows)            │
│                                                                          │
│  agg_daily_sales (created via nightly batch)                             │
│  ─────────────────────────────────                                       │
│  • date_fk, category_fk, seller_fk                                       │
│  • total_sales, total_orders, total_units                                │
│  • avg_order_value                                                       │
│  • Row count: ~500K (vs 9B) → 18,000x reduction                         │
│  • Query time: < 1 second (vs 45 seconds)                                │
│                                                                          │
│  Trade-off: Data freshness (daily) vs query speed                        │
│  Solution: Real-time for ops, pre-aggregated for BI                      │
│                                                                          │
│  4. INDEXING (where supported)                                           │
│  ─────────────────────────────                                           │
│  • dim_product(product_id) - natural key lookups                         │
│  • dim_customer(customer_id) - natural key lookups                       │
│  • fct_order_line_items(order_id) - order lookup queries                 │
│                                                                          │
│  5. COLUMNAR STORAGE                                                     │
│  ───────────────────                                                     │
│  All tables use columnar format (Parquet/ORC)                            │
│  Benefit: Only read columns needed for query                             │
│  Example: SELECT category_name, SUM(sales)                               │
│           → Only reads 2 columns from fact table                         │
│                                                                          │
│  6. MATERIALIZED VIEWS                                                   │
│  ─────────────────────                                                   │
│  CREATE MATERIALIZED VIEW mv_category_daily_sales AS                     │
│  SELECT order_date_fk, category_fk, SUM(sales_amount)                    │
│  FROM fct_order_line_items f                                             │
│  JOIN dim_product p ON f.product_fk = p.product_sk                       │
│  GROUP BY 1, 2;                                                          │
│                                                                          │
│  Auto-refreshed daily, used by 80% of BI dashboards                      │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Trade-offs Analysis**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                         TRADE-OFFS ANALYSIS                              │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Star Schema vs Snowflake Schema                                      │
│  ───────────────────────────────────                                     │
│                                                                          │
│  We chose: STAR SCHEMA                                                   │
│                                                                          │
│  Star Schema Benefits:                                                   │
│  ✓ Simpler queries (fewer joins)                                         │
│  ✓ Better query performance                                              │
│  ✓ Easier for analysts to understand                                     │
│                                                                          │
│  Snowflake Schema Benefits:                                              │
│  ✓ Less storage (normalized dimensions)                                  │
│  ✓ Easier dimension maintenance (update in one place)                    │
│                                                                          │
│  Decision rationale: Query performance > storage cost for this use case  │
│  Storage is cheap, analyst productivity is valuable                      │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  2. Single Fact Table vs Multiple Fact Tables                            │
│  ─────────────────────────────────────────────                           │
│                                                                          │
│  We chose: SINGLE FACT TABLE (fct_order_line_items)                      │
│  with consolidated returns logic                                         │
│                                                                          │
│  Single Table Benefits:                                                  │
│  ✓ Simpler model for analysts                                            │
│  ✓ Can analyze sales and returns together                                │
│  ✓ One ETL pipeline to maintain                                          │
│                                                                          │
│  Multiple Tables Benefits:                                               │
│  ✓ Returns could have different grain (return vs line item)              │
│  ✓ Separate optimization strategies for each table                       │
│                                                                          │
│  Decision rationale: Returns share same grain (line item level)          │
│  If returns were at order level, separate table would be better          │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  3. Price in Dimension vs Price in Fact                                  │
│  ─────────────────────────────────────────                               │
│                                                                          │
│  We chose: BOTH                                                          │
│                                                                          │
│  dim_product.base_price: Current price (for catalog queries)             │
│  fct_order_line_items.unit_price: Historical price at sale time          │
│                                                                          │
│  Price in Dimension Benefits:                                            │
│  ✓ Current price always available                                        │
│  ✓ Can track price changes over time (SCD Type 2)                        │
│                                                                          │
│  Price in Fact Benefits:                                                 │
│  ✓ Accurate historical revenue calculations                              │
│  ✓ No need to join for price at transaction time                         │
│                                                                          │
│  Decision rationale: Historical accuracy is critical for revenue         │
│  Dimension price useful for "what's the current price" queries           │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  4. SCD Type 2 for All vs Selective SCD                                  │
│  ─────────────────────────────────────────                               │
│                                                                          │
│  We chose: SELECTIVE SCD                                                 │
│                                                                          │
│  Type 2 (track history): price, category, seller settings                │
│  Type 1 (overwrite): product name, description (only corrections)        │
│                                                                          │
│  Full Type 2 Benefits:                                                   │
│  ✓ Complete audit trail                                                  │
│  ✓ Can answer any historical question                                    │
│                                                                          │
│  Selective SCD Benefits:                                                 │
│  ✓ Smaller dimension tables                                              │
│  ✓ Faster dimension joins                                                │
│  ✓ Less ETL complexity                                                   │
│                                                                          │
│  Decision rationale: Product descriptions don't affect analytics         │
│  History only matters for attributes that drive business decisions       │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Problem 2: Clickstream Event Schema

**Business Context**:
Design a schema for website analytics (like Google Analytics):
- 100M events/day (page views, clicks, scrolls)
- Events have: user_id, session_id, timestamp, event_type, properties (JSON)
- Need to support: Funnel analysis, path analysis, session metrics
- 2-year data retention
- Query patterns: 90% are last 30 days, 10% are historical analysis

**Key Challenge**: How to handle the high cardinality of session_id and user_id in a fact table?

**Solution Approach**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    CLICKSTREAM SCHEMA SOLUTION                           │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Challenge: 100M events/day × 730 days = 73B rows                       │
│  session_id has high cardinality (millions per day)                     │
│  Can't use traditional star schema (dimension tables would be huge)      │
│                                                                          │
│  Solution: DEGENERATE DIMENSIONS + AGGREGATION TABLES                    │
│                                                                          │
│  fct_events (Raw Event Fact Table)                                       │
│  ─────────────────────────────────                                       │
│  GRAIN: One row per event                                                │
│  • event_id (degenerate dimension)                                       │
│  • session_id (degenerate - not a FK!)                                   │
│  • user_id (degenerate - not a FK!)                                      │
│  • event_timestamp                                                       │
│  • event_type_fk → dim_event_type                                        │
│  • page_url_fk → dim_page (SCD Type 1, limited cardinality)              │
│  • device_fk → dim_device                                                │
│  • geo_fk → dim_geography                                                │
│  • properties_json (semi-structured)                                     │
│                                                                          │
│  -- Degenerate dimensions (stored in fact, not separate dim tables)      │
│  • session_id: Used for session analysis, but no session dimension       │
│  • user_id: Used for user analysis, but no user dimension (too big)      │
│                                                                          │
│  Why degenerate?                                                         │
│  • Session dimension would have 100M+ rows/day                           │
│  • User dimension would have 500M+ rows over 2 years                     │
│  • These aren't descriptive - they don't have attributes to look up      │
│  • Store IDs in fact table for filtering/grouping                        │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Aggregation Tables (For Performance)                                    │
│  ───────────────────────────────────                                     │
│                                                                          │
│  agg_daily_sessions (Periodic Snapshot)                                  │
│  ─────────────────────────────────────                                   │
│  GRAIN: One row per session per day                                      │
│  • date_fk                                                               │
│  • session_id (degenerate)                                               │
│  • user_id (degenerate)                                                  │
│  • session_start_time, session_end_time                                  │
│  • duration_seconds                                                      │
│  • page_views_count                                                      │
│  • events_count                                                          │
│  • bounce_flag (only 1 page view)                                        │
│  • device_fk, geo_fk, traffic_source_fk                                  │
│                                                                          │
│  Row reduction: 100M events → 10M sessions = 10x reduction               │
│  Use for: Session-level analysis, bounce rate, session duration          │
│                                                                          │
│  agg_hourly_page_views (Aggregated Fact)                                 │
│  ─────────────────────────────────────                                   │
│  GRAIN: One row per page per hour                                        │
│  • hour_fk (date + hour)                                                 │
│  • page_fk                                                               │
│  • device_fk                                                             │
│  • views_count                                                           │
│  • unique_visitors_count (approximate via HyperLogLog)                   │
│                                                                          │
│  Row reduction: 100M events → ~100K rows = 1000x reduction               │
│  Use for: Dashboard charts, traffic trends                               │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Funnel Analysis Pattern                                                 │
│  ─────────────────────────                                               │
│  How to answer: "How many users completed checkout after viewing cart?"  │
│                                                                          │
│  Option 1: Self-join (expensive on 73B rows)                             │
│  Option 2: Funnel table (pre-computed)                                   │
│                                                                          │
│  agg_funnel_steps (Pre-computed Funnel)                                  │
│  ─────────────────────────────────────                                   │
│  • date_fk                                                               │
│  • user_id                                                               │
│  • step_1_viewed_product (timestamp)                                     │
│  • step_2_added_to_cart (timestamp)                                      │
│  • step_3_viewed_checkout (timestamp)                                    │
│  • step_4_completed_purchase (timestamp)                                 │
│  • time_to_cart (seconds)                                                │
│  • time_to_checkout (seconds)                                            │
│  • time_to_purchase (seconds)                                            │
│                                                                          │
│  Built via: Nightly batch job that scans previous day's events           │
│  in chronological order per user, tracking state machine                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Problem 3: Data Mesh - Conformed Dimensions

**Business Context**:
Your company is adopting data mesh. Three domains own their data:
- **Product Domain**: Product catalog, inventory, pricing
- **Marketing Domain**: Campaigns, leads, attribution
- **Sales Domain**: Opportunities, quotes, orders

Each domain has their own data warehouse. But executives need cross-domain reports like "Which marketing campaigns drove sales of which product categories?"

**Your Task**: Design the conformed dimensions and cross-domain fact tables.

**Solution Approach**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DATA MESH - CONFORMED DIMENSIONS                      │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  Data Mesh Principle: Domain ownership + Federated governance            │
│                                                                          │
│  ┌─────────────────────────────────────────────────────────────────┐    │
│  │                     FEDERATED GOVERNANCE                         │    │
│  │  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐          │    │
│  │  │  Product     │  │  Marketing   │  │    Sales     │          │    │
│  │  │   Domain     │  │   Domain     │  │   Domain     │          │    │
│  │  │              │  │              │  │              │          │    │
│  │  │ • dim_product│  │ • dim_campaign│  │ • dim_sales_rep       │    │
│  │  │ • dim_category│  │ • dim_channel │  │ • dim_territory       │    │
│  │  │ • fct_inventory│  │ • fct_leads   │  │ • fct_opportunities   │    │
│  │  └──────┬───────┘  └──────┬───────┘  └──────┬───────┘          │    │
│  │         │                 │                 │                   │    │
│  │         └─────────────────┼─────────────────┘                   │    │
│  │                           ▼                                     │    │
│  │              ┌─────────────────────────┐                        │    │
│  │              │   CONFORMED DIMENSIONS  │                        │    │
│  │              │  (centrally governed)   │                        │    │
│  │              ├─────────────────────────┤                        │    │
│  │              │ • dim_date              │                        │    │
│  │              │ • dim_customer          │                        │    │
│  │              │ • dim_geography         │                        │    │
│  │              │ • dim_organization      │                        │    │
│  │              └─────────────────────────┘                        │    │
│  └─────────────────────────────────────────────────────────────────┘    │
│                                                                          │
│  CONFORMED DIMENSION: dim_customer                                       │
│  ─────────────────────────────────                                       │
│  Owned by: Data Platform Team (or Customer Domain if exists)             │
│  Used by: All domains                                                    │
│                                                                          │
│  Attributes:                                                             │
│  • customer_sk (surrogate key - this is the conformed key!)              │
│  • customer_id (natural key from source system)                          │
│  • customer_name                                                         │
│  • customer_type (prospect/customer/churned)                             │
│  • industry_fk → dim_industry                                            │
│  • geography_fk → dim_geography                                          │
│  • first_touch_date (from Marketing domain)                              │
│  • first_purchase_date (from Sales domain)                               │
│  • lifetime_value (computed from Sales domain)                           │
│                                                                          │
│  Synchronization:                                                        │
│  • Marketing domain publishes lead_created events → updates first_touch  │
│  • Sales domain publishes order_completed events → updates LTV           │
│  • Daily batch job reconciles across domains                             │
│                                                                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  CROSS-DOMAIN FACT TABLE: Campaign Attribution                           │
│  ─────────────────────────────────────────────                           │
│                                                                          │
│  fct_campaign_attribution                                                │
│  GRAIN: One row per campaign touch leading to a sale                     │
│                                                                          │
│  Dimensions from Marketing Domain:                                       │
│  • campaign_fk → marketing.dim_campaign                                  │
│  • channel_fk → marketing.dim_channel                                    │
│                                                                          │
│  Dimensions from Product Domain:                                         │
│  • product_fk → product.dim_product                                      │
│  • category_fk → product.dim_category                                    │
│                                                                          │
│  Dimensions from Sales Domain:                                           │
│  • sales_rep_fk → sales.dim_sales_rep                                    │
│                                                                          │
│  CONFORMED Dimensions (shared):                                          │
│  • customer_fk → shared.dim_customer                                     │
│  • date_fk → shared.dim_date                                             │
│  • geography_fk → shared.dim_geography                                   │
│                                                                          │
│  Measures:                                                               │
│  • attributed_revenue (using multi-touch attribution model)              │
│  • touch_sequence (1st touch, 2nd touch, etc.)                           │
│  • days_to_conversion                                                    │
│                                                                          │
│  How it's built:                                                         │
│  1. Marketing domain publishes: fct_lead_touches (customer_id, timestamp)│
│  2. Sales domain publishes: fct_orders (customer_id, timestamp, revenue) │
│  3. Attribution engine joins using conformed customer_id                 │
│  4. Apply attribution model (first-touch, last-touch, linear, etc.)      │
│                                                                          │
│  Governance:                                                             │
│  • Schema changes require approval from all domain owners                │
│  • Conformed dimensions have SLAs (99.9% freshness)                      │
│  • Breaking changes require 30-day notice                                │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Session Flow Example

**You**: "Welcome! Today we'll design a data warehouse schema together. I'm looking for your thought process - how you approach business requirements, make grain decisions, and optimize for real-world query patterns. Don't worry about getting everything perfect; I'm here to explore the trade-offs with you. Ready to start?"

**Candidate**: "Yes, let's do it!"

**You**: "Great! Here's the scenario: [present business problem]. Before we design any tables, what questions do you have about the business and the questions they need to answer?"

[Let candidate ask questions, provide answers about requirements]

**You**: "Good questions! Now, let's identify the business processes we need to model. What are the key events or transactions?"

[Guide through identifying facts and dimensions]

**You**: "Excellent. Let's focus on the sales transaction. What's the grain of that fact table? Be specific."

[Probe on grain definition - most important decision]

**You**: "Perfect - one row per order line item. Now let's design the dimension tables. Starting with the product dimension - which attributes would you include, and how would you handle changes to product price over time?"

[Continue through each dimension, discussing SCD strategy]

**You**: "This is shaping up well. Now let's talk about performance. The business users complain that their monthly reports take 10 minutes to run. What optimizations would you consider?"

[Discuss partitioning, clustering, pre-aggregation]
