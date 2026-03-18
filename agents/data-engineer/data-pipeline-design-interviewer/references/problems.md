# Problem Bank

## Problem 1: Multi-Source Sales Pipeline

**Business Context**:
You are building a daily pipeline for a retail company:
- 3 source systems: POS (PostgreSQL), e-commerce (MySQL), returns (REST API)
- Target: Snowflake warehouse with a unified sales fact table
- Volume: 5M transactions/day across all sources
- SLA: Data available by 7 AM for morning dashboards
- Complications: POS system has duplicates, e-commerce has late-arriving updates, API has rate limits

**Your Task**:
Design the complete pipeline with:
1. DAG structure with task dependencies
2. Extract strategy for each source
3. Data quality checks at each stage
4. Deduplication and reconciliation logic
5. Failure recovery plan

---

**DAG Structure**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                    DAILY UNIFIED SALES PIPELINE                          │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│   ┌────────────────┐                                                     │
│   │ check_sources  │  (Sensor: verify all 3 sources are reachable)       │
│   └───────┬────────┘                                                     │
│           │                                                              │
│     ┌─────┼──────────────┐                                               │
│     ▼     ▼              ▼                                               │
│ ┌────────┐ ┌──────────┐ ┌──────────┐                                    │
│ │extract │ │ extract  │ │ extract  │  (Parallel, independent sources)   │
│ │_pos    │ │_ecommerce│ │_returns  │                                    │
│ │        │ │          │ │ (paginated│                                    │
│ │        │ │          │ │  API)    │                                    │
│ └───┬────┘ └────┬─────┘ └────┬─────┘                                    │
│     │           │            │                                           │
│     ▼           ▼            ▼                                           │
│ ┌────────┐ ┌──────────┐ ┌──────────┐                                    │
│ │validate│ │ validate │ │ validate │  (Source-specific checks)          │
│ │_pos    │ │_ecommerce│ │_returns  │                                    │
│ └───┬────┘ └────┬─────┘ └────┬─────┘                                    │
│     │           │            │                                           │
│     └───────────┼────────────┘                                           │
│                 ▼                                                         │
│          ┌─────────────┐                                                 │
│          │ deduplicate │  (Cross-source dedup on transaction_id)         │
│          │ _and_merge  │                                                 │
│          └──────┬──────┘                                                 │
│                 ▼                                                         │
│          ┌─────────────┐                                                 │
│          │ transform   │  (Build unified fact table)                     │
│          │ _fact_sales │                                                 │
│          └──────┬──────┘                                                 │
│                 ▼                                                         │
│          ┌─────────────┐                                                 │
│          │ validate    │  (Cross-source reconciliation)                  │
│          │ _output     │                                                 │
│          └──────┬──────┘                                                 │
│                 ▼                                                         │
│          ┌─────────────┐                                                 │
│          │ swap_staging│  (Atomic swap to production)                    │
│          │ _to_prod    │                                                 │
│          └──────┬──────┘                                                 │
│                 ▼                                                         │
│          ┌─────────────┐                                                 │
│          │ notify      │  (Slack + email summary)                        │
│          └─────────────┘                                                 │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Extract Strategies**:

| Source | Strategy | Details |
|--------|----------|---------|
| POS (PostgreSQL) | Incremental (watermark) | `WHERE updated_at > last_watermark`. POS generates duplicates, so dedup with `ROW_NUMBER() OVER (PARTITION BY txn_id ORDER BY updated_at DESC)` |
| E-commerce (MySQL) | Incremental (watermark + overlap) | `WHERE updated_at > last_watermark - INTERVAL '10 minutes'`. Overlap handles late updates. MERGE into staging. |
| Returns API | Full daily extract (paginated) | API has no incremental endpoint. Rate limit: 100 req/min. Use exponential backoff. Extract full day, then merge. |

---

**Data Quality Checks**:
```
Stage 1: Post-Extract (per source)
  - Row count > 0
  - Row count within 2 standard deviations of 30-day average
  - No nulls in primary key columns
  - Date range matches expected extraction window
  - Schema matches expected (column names, types)

Stage 2: Post-Dedup
  - Zero duplicate transaction_ids across all sources
  - Total row count = sum of source counts minus expected overlap

Stage 3: Post-Transform
  - Revenue totals within 5% of previous day (flag, do not block)
  - All foreign keys resolve to dimension tables
  - No negative quantities or amounts (unless returns)
  - Date distribution matches expected pattern

Stage 4: Pre-Swap Reconciliation
  - Source system totals match warehouse totals (within tolerance)
  - Compare against independent accounting system if available
```

---

## Problem 2: CDC Pipeline with Debezium

**Business Context**:
Your team wants to move from batch ETL (nightly full reloads) to near-real-time CDC:
- Source: PostgreSQL with 20 tables, largest has 200M rows
- Target: Snowflake via S3
- Requirement: Data freshness < 15 minutes
- Constraint: Cannot add triggers or modify source schema

**Task**: Design the CDC pipeline architecture.

**Solution Approach**:
```
CDC Architecture:

┌──────────────┐     ┌──────────────┐     ┌──────────┐     ┌──────────┐
│  PostgreSQL  │────>│   Debezium   │────>│  Kafka   │────>│  S3      │
│  (WAL logs)  │     │  Connector   │     │  Topics  │     │  (JSON/  │
│              │     │              │     │  (1 per  │     │  Parquet)│
│              │     │              │     │   table) │     │          │
└──────────────┘     └──────────────┘     └──────────┘     └────┬─────┘
                                                                │
                                                                ▼
                                                          ┌──────────┐
                                                          │Snowflake │
                                                          │Snowpipe  │
                                                          │(auto-    │
                                                          │ ingest)  │
                                                          └────┬─────┘
                                                               │
                                                               ▼
                                                         ┌───────────┐
                                                         │  MERGE    │
                                                         │  task     │
                                                         │ (every    │
                                                         │  15 min)  │
                                                         └───────────┘

Key Design Decisions:

1. WAL-based CDC (no triggers needed):
   - Debezium reads PostgreSQL WAL (Write-Ahead Log)
   - Zero impact on source database performance
   - Captures INSERT, UPDATE, DELETE operations

2. Kafka as buffer:
   - Decouples source from destination
   - Handles backpressure during spikes
   - Enables replay if Snowflake is down
   - One topic per source table

3. S3 as staging layer:
   - Kafka Connect S3 Sink (batches every 5 minutes)
   - Parquet format for efficient Snowflake loading
   - Partitioned by date/hour for lifecycle management

4. Snowpipe for continuous loading:
   - Auto-triggered by S3 event notifications
   - Loads into raw/staging tables
   - Near-real-time (< 1 minute after S3 write)

5. Scheduled MERGE for final tables:
   - Airflow task runs every 15 minutes
   - Applies CDC operations (insert/update/delete)
   - Maintains SCD Type 2 where needed
```

---

## Problem 3: Airflow DAG Failure Recovery

**Business Context**:
Your Airflow DAG failed at 3 AM on a Tuesday. The failure was in the `transform_orders` task. You need to investigate and recover. The downstream tasks (`validate_output`, `load_to_prod`, `notify`) did not run.

**Task**: Walk through the debugging and recovery process.

**Solution Walkthrough**:
```
Step-by-Step Recovery:

1. TRIAGE (5 minutes)
   - Check Airflow UI: task instance logs for transform_orders
   - Check task duration: did it timeout or error immediately?
   - Check upstream tasks: did extract tasks complete successfully?
   - Common causes:
     a. Snowflake warehouse suspended (auto-suspend timeout)
     b. Source data quality issue (unexpected nulls, schema change)
     c. Out-of-memory on Airflow worker
     d. Network timeout to warehouse

2. DIAGNOSE (10 minutes)
   - Read the full stack trace in task logs
   - If SQL error: check the specific SQL that failed
   - If timeout: check warehouse activity and query history
   - If OOM: check data volume -- was there a spike?
   - If schema error: compare source schema vs expected schema

3. FIX (varies)
   - Schema change: update transformation SQL, test locally
   - Data quality: add upstream check, fix data if possible
   - Timeout: increase timeout, resume warehouse, optimize query
   - OOM: increase worker resources or optimize transformation

4. RECOVER (10 minutes)
   - Clear the failed task in Airflow UI (or CLI: airflow tasks clear)
   - This reruns transform_orders and all downstream tasks
   - Verify idempotency: will rerunning produce correct results?
     - If using DELETE + INSERT: safe to rerun
     - If using INSERT only: risk of duplicates, need manual cleanup

5. VERIFY (15 minutes)
   - Check output row counts match expectations
   - Run reconciliation query against source
   - Confirm dashboards show correct data
   - Check SLA: did we make the 8 AM deadline?

6. PREVENT (next sprint)
   - Add the missing check that would have caught this earlier
   - Update runbook with this failure scenario
   - Consider adding circuit breaker for this failure mode
```

---

## Problem 4: Schema Evolution Handling

**Business Context**:
Your upstream team frequently adds, removes, and renames columns in the source database. Last month, three pipeline failures were caused by schema changes. The data team is frustrated.

**Task**: Design a schema evolution strategy that prevents pipeline breakdowns.

**Solution Approach**:
```
Schema Evolution Strategy:

1. DETECTION: Schema Change Monitoring
   - Daily job compares current source schema vs stored schema snapshot
   - Detects: added columns, removed columns, type changes, renames
   - Stores schema history in metadata table for audit trail

2. TOLERANCE: Defensive Pipeline Design
   - Extract: SELECT explicit columns, not SELECT *
   - Transform: Use COALESCE(new_column, default) for optional fields
   - Load: Target table uses ALTER TABLE ADD COLUMN for new fields
   - Principle: Additive changes are safe, breaking changes require review

3. CONTRACTS: Data Contracts with Upstream Teams
   - Document expected schema in version-controlled YAML
   - Breaking changes require 2-week notice + migration plan
   - CI/CD check: schema compatibility test before deployment

4. AUTOMATION: Self-Healing Pipeline
   - If new column detected: auto-add to staging, alert team
   - If column removed: log warning, use NULL, alert team
   - If type changed: halt pipeline, require manual review

   Schema compatibility matrix:
   ┌─────────────────────┬────────────┬───────────────────┐
   │ Change Type         │ Auto-heal? │ Action            │
   ├─────────────────────┼────────────┼───────────────────┤
   │ Column added        │ Yes        │ Add to staging    │
   │ Column removed      │ Yes        │ NULL + alert      │
   │ Type widened        │ Yes        │ Cast + alert      │
   │ (int -> bigint)     │            │                   │
   │ Type narrowed       │ No         │ Halt + review     │
   │ (varchar -> int)    │            │                   │
   │ Column renamed      │ No         │ Halt + review     │
   │ Table dropped       │ No         │ Halt + escalate   │
   └─────────────────────┴────────────┴───────────────────┘
```

---

## Problem 5: Pipeline Testing Strategy

**Business Context**:
Your team has 30 Airflow DAGs in production with zero tests. Last quarter, 12 pipeline failures were caused by bugs that could have been caught by testing. Management wants a testing strategy.

**Task**: Design a testing framework for data pipelines.

**Solution Approach**:
```
Testing Pyramid for Data Pipelines:

                    ┌───────────┐
                    │  End-to-  │  (Few, slow, expensive)
                    │  End      │  Run nightly in staging
                    ├───────────┤
                    │Integration│  (Some, moderate speed)
                    │  Tests    │  Run on PR merge
                ┌───┴───────────┴───┐
                │    Unit Tests     │  (Many, fast, cheap)
                │                   │  Run on every commit
                └───────────────────┘

Layer 1: Unit Tests (run in seconds)
  - Test individual SQL transformations with small datasets
  - Test Python functions (parsing, validation logic)
  - Use dbt tests for column-level assertions
  - Example: Given 5 rows of input, does the transform produce expected output?

Layer 2: Integration Tests (run in minutes)
  - Test DAG structure: all tasks have correct dependencies
  - Test connections: can we reach all source/target systems?
  - Test idempotency: running the pipeline twice produces same result
  - Example: Load test data into staging Snowflake, run transform, check output

Layer 3: End-to-End Tests (run in staging environment)
  - Full pipeline run with production-like data (sampled or anonymized)
  - Verify SLA timing: does the pipeline complete within budget?
  - Verify data quality checks fire correctly on bad data
  - Example: Run full DAG in staging, compare output to expected baseline

CI/CD Integration:
  - Pre-commit: SQL linting (sqlfluff), Python linting
  - PR check: Unit tests + DAG validation (airflow dags test)
  - Post-merge: Integration tests in staging
  - Pre-deploy: End-to-end test with approval gate
  - Post-deploy: Smoke test (run pipeline with small dataset)
```

---

## Session Flow Example

**You**: "Welcome! I'm excited to talk about pipeline design with you today. Let's jump right in. Here's a question I like to start with: what's the difference between ETL and ELT, and when would you choose one over the other?"

**Candidate**: "ETL transforms data before loading, ELT loads first then transforms..."

**You**: "Good start. You've got the mechanics right. Now, with modern cloud warehouses like Snowflake or BigQuery having massive compute power, how does that shift the ETL vs ELT decision? What are the practical implications?"

[Let candidate elaborate, probe on cost, flexibility, and auditability]

**You**: "Great discussion. Now let's put this into practice. Here's the scenario: [present problem]. Before we design the DAG, what questions would you want to ask about the source systems?"

[Guide through design, probing on task granularity and error handling]

**You**: "I like that you added parallel extraction tasks. Now, what data quality checks would you add, and where in the DAG would you place them?"

[Push on quality, staging tables, and safe loading patterns]

**You**: "Solid design. One last curve ball: this pipeline takes 4 hours. The business now wants hourly refreshes. What changes?"

[Explore incremental loading strategies]
