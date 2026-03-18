# Problem Bank

## Problem 1: End-to-End Pipeline Design

**Business Context**:
You're building a data platform for a food delivery app (like DoorDash/UberEats):
- 1 million orders/day, spikes to 3x during lunch/dinner
- Need real-time driver location tracking (sub-second latency)
- Need order analytics for restaurants (15-min delay acceptable)
- Need daily business reports
- Budget: $50K/month infrastructure cost

**Your Task**:
Design the complete pipeline with:
1. Requirements Summary (extract key constraints)
2. Architecture Diagram (text-based)
3. Component Breakdown (tool choices with justification)
4. Trade-offs Analysis
5. Failure Scenarios

---

**Requirements Summary**:
```
┌─────────────────────────────────────────────────────────────┐
│                    REQUIREMENTS SUMMARY                      │
├─────────────────────────────────────────────────────────────┤
│ Data Volume:                                                 │
│   • Normal: 1M orders/day = ~11 orders/sec average          │
│   • Peak: 3x = ~33 orders/sec                               │
│   • Driver location: 100K drivers × 1 update/sec = 100K/sec │
│                                                              │
│ Latency SLAs:                                                │
│   • Driver tracking: < 100ms (real-time streaming)          │
│   • Restaurant analytics: < 15 min (near real-time)         │
│   • Business reports: < 24 hours (batch)                    │
│                                                              │
│ Consistency Model:                                           │
│   • Driver tracking: Eventual consistency OK                │
│   • Order counts: Exactly-once semantics required           │
│                                                              │
│ Cost Constraint: $50K/month                                  │
└─────────────────────────────────────────────────────────────┘
```

---

**Architecture Diagram**:
```
┌─────────────────────────────────────────────────────────────────────────┐
│                           INGESTION LAYER                                │
│  ┌─────────────────┐  ┌─────────────────┐  ┌─────────────────┐         │
│  │  Mobile Apps    │  │  Restaurant     │  │  Driver Apps    │         │
│  │  (Order events) │  │  (Menu updates) │  │  (GPS location) │         │
│  └────────┬────────┘  └────────┬────────┘  └────────┬────────┘         │
│           │                    │                    │                  │
│           └────────────────────┼────────────────────┘                  │
│                                ▼                                        │
│                    ┌─────────────────────┐                              │
│                    │   Kafka Cluster     │                              │
│                    │   • 3 brokers       │                              │
│                    │   • Replication: 3  │                              │
│                    │   • Retention: 7d   │                              │
│                    └──────────┬──────────┘                              │
│                               │                                         │
└───────────────────────────────┼─────────────────────────────────────────┘
                                │
┌───────────────────────────────┼─────────────────────────────────────────┐
│                          PROCESSING LAYER                                │
│                               │                                         │
│           ┌───────────────────┼───────────────────┐                     │
│           ▼                   ▼                   ▼                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐           │
│  │  Flink Cluster  │ │  Kafka Streams  │ │  Spark Batch    │           │
│  │  (Driver loc)   │ │  (Order enrich) │ │  (Daily agg)    │           │
│  │                 │ │                 │ │                 │           │
│  │ Latency: <100ms │ │ Latency: <15min │ │ Runs: Daily     │           │
│  │ State: RocksDB  │ │ Window: 5min    │ │ Schedule:       │           │
│  └────────┬────────┘ └────────┬────────┘ │ Airflow         │           │
│           │                   │          └────────┬────────┘           │
│           ▼                   ▼                   ▼                     │
│  ┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐           │
│  │  Redis Cluster  │ │  ClickHouse     │ │  Snowflake      │           │
│  │  (Geospatial)   │ │  (Analytics)    │ │  (Data Warehouse)│          │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
                                │
┌───────────────────────────────┼─────────────────────────────────────────┐
│                         SERVING LAYER                                    │
│                               │                                         │
│  ┌─────────────────┐ ┌────────┴────────┐ ┌─────────────────┐           │
│  │  Driver App API │ │  Restaurant     │ │  BI Dashboard   │           │
│  │  (Nearest driver│ │  Dashboard      │ │  (Looker/Tableau)│          │
│  │   search)       │ │  (Order stats)  │ │                 │           │
│  └─────────────────┘ └─────────────────┘ └─────────────────┘           │
│                                                                         │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Component Breakdown**:

| Layer | Component | Choice | Justification |
|-------|-----------|--------|---------------|
| **Ingestion** | Message Queue | **Kafka** | Industry standard, great ecosystem. Can handle 100K+ msgs/sec. Replay capability for reprocessing. Cost-effective at scale. |
| | Alternative | Pulsar | Better geo-replication, but smaller community |
| **Processing** | Real-time | **Flink** | True streaming (not micro-batch). Stateful processing with checkpointing. Advanced windowing for late arrivals. Backpressure handling. |
| | Near real-time | **Kafka Streams** | Same ecosystem as Kafka. Good for stream enrichment. Simpler operational model. |
| | Batch | **Spark on EMR** | Cost-effective for daily jobs. Can auto-terminate after run. Good SQL support via Spark SQL. |
| **Storage** | Hot serving | **Redis** | Sub-millisecond latency. Geospatial queries (Redis Geo). TTL for automatic cleanup. |
| | Analytics | **ClickHouse** | Columnar storage for fast aggregations. Real-time ingestion. Cost-effective vs Snowflake for this use case. |
| | Warehouse | **Snowflake** | Separation of compute/storage. Automatic scaling. Great for BI workloads. |
| **Orchestration** | Pipeline | **Airflow** | Industry standard. Good dependency management. Backfill support. |

---

**Trade-offs Analysis**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                        TRADE-OFFS ANALYSIS                               │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  1. Kafka vs Kinesis                                                       │
│     ✓ Kafka: Better cost at scale, more control, open source              │
│     ✗ Kafka: Operational overhead (need to manage brokers)                │
│     ✓ Kinesis: Fully managed, less operational burden                     │
│     ✗ Kinesis: More expensive at high throughput, 7-day retention limit   │
│     → Decision: Kafka for cost control and 7+ day retention needs         │
│                                                                          │
│  2. Flink vs Spark Streaming                                               │
│     ✓ Flink: True streaming, better state management, lower latency       │
│     ✗ Flink: Smaller talent pool, steeper learning curve                  │
│     ✓ Spark: More familiar to data engineers, unified batch/streaming     │
│     ✗ Spark: Micro-batch (100ms+ latency), less suitable for <100ms SLA   │
│     → Decision: Flink for driver tracking, Spark for batch                │
│                                                                          │
│  3. ClickHouse vs Snowflake for Analytics                                  │
│     ✓ ClickHouse: 10x cheaper, better ingestion performance               │
│     ✗ ClickHouse: Self-managed, harder to scale storage                   │
│     ✓ Snowflake: Fully managed, infinite scale, better for BI             │
│     ✗ Snowflake: Expensive for high-frequency queries                     │
│     → Decision: ClickHouse for 15-min analytics, Snowflake for daily BI   │
│                                                                          │
│  4. Cost vs Latency Trade-off                                              │
│     • Sub-100ms SLA requires in-memory processing (Redis, Flink)          │
│     • Could use DynamoDB instead of Redis: easier but 10x more expensive  │
│     • Keep hot data in Redis (24h), archive to S3 for cost savings        │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

**Failure Scenarios**:

```
┌─────────────────────────────────────────────────────────────────────────┐
│                       FAILURE SCENARIOS & RECOVERY                       │
├─────────────────────────────────────────────────────────────────────────┤
│                                                                          │
│  SCENARIO 1: Kafka Broker Failure                                        │
│  ───────────────────────────────                                         │
│  Impact: Loss of ingestion during broker restart                         │
│  Detection: Kafka exporter metrics (under-replicated partitions)         │
│  Recovery:                                                               │
│    • Automatic: Leader election to replicas (seconds)                    │
│    • If all replicas lost: Data loss for uncommitted messages            │
│  Prevention:                                                             │
│    • Replication factor = 3 across AZs                                   │
│    • Min.insync.replicas = 2                                             │
│    • Producer acks=all                                                   │
│                                                                          │
│  SCENARIO 2: Flink Job Failure (Driver Tracking)                         │
│  ─────────────────────────────────                                       │
│  Impact: Driver locations not updating                                   │
│  Detection: Flink checkpoint failures, lag alerts                        │
│  Recovery:                                                               │
│    • Automatic restart from last checkpoint (state in RocksDB/S3)        │
│    • Reprocess from Kafka (replay capability)                            │
│  Prevention:                                                             │
│    • Enable exactly-once processing with checkpointing                   │
│    • Set restart strategy: fixed-delay with 3 attempts                   │
│    • Alert on consumer lag > 30 seconds                                  │
│                                                                          │
│  SCENARIO 3: Schema Evolution Breaking Pipeline                          │
│  ─────────────────────────────────────                                   │
│  Impact: Parsing failures, data loss                                     │
│  Detection: Dead letter queue growing, parsing error metrics             │
│  Recovery:                                                               │
│    • Fix schema registry compatibility                                   │
│    • Replay failed messages from DLQ                                     │
│  Prevention:                                                             │
│    • Schema registry with BACKWARD compatibility                         │
│    • CI/CD checks for schema changes                                     │
│    • Use Avro with default values for new fields                         │
│                                                                          │
│  SCENARIO 4: 10x Traffic Spike (Viral Event)                             │
│  ─────────────────────────────────────                                   │
│  Impact: Backpressure, consumer lag, potential data loss                 │
│  Detection: Kafka consumer lag alerts, Flink backpressure metrics        │
│  Recovery:                                                               │
│    • Auto-scale Flink task managers (K8s HPA)                            │
│    • Scale Kafka partitions (requires planning)                          │
│    • If overwhelmed: Enable sampling (process 10% of events)             │
│  Prevention:                                                             │
│    • Over-provision by 3x normal peak                                    │
│    • Load testing with realistic traffic patterns                        │
│    • Circuit breakers on non-critical paths                              │
│                                                                          │
│  SCENARIO 5: Late Data Arrivals (Mobile Offline Mode)                    │
│  ─────────────────────────────────────────                               │
│  Impact: Incomplete aggregations in real-time views                      │
│  Detection: Watermark lag metrics                                        │
│  Recovery:                                                               │
│    • Side output late data to separate stream                            │
│    • Nightly batch job corrects aggregates                               │
│    • Reconciliation job compares real-time vs batch                      │
│  Prevention:                                                             │
│    • Set allowed lateness (e.g., 24 hours)                               │
│    • Document data quality expectations (eventual consistency)           │
│                                                                          │
└─────────────────────────────────────────────────────────────────────────┘
```

---

## Problem 2: Cost-Optimized Batch Pipeline

**Business Context**:
You need to process 10TB of clickstream data daily for analytics:
- Current: Running Spark on EMR 24/7 ($15K/month)
- Target: Reduce cost to <$3K/month
- SLA: Data available by 9 AM for analysts
- Can tolerate 2-hour delay during failures

**Task**: Redesign for cost optimization while maintaining reliability.

**Solution Approach**:
```
Cost Optimization Strategy:

Current (Expensive):
  EMR cluster: 10 r5.2xlarge running 24/7 = ~$15K/month

Optimized:
  1. Use Spot instances for workers (70% savings)
  2. Auto-terminate after job completion
  3. Use Graviton instances (20% savings)
  4. Partition data intelligently (less data to scan)

Architecture:
  S3 (Raw Data) → AWS Glue (Serverless Spark) → S3 (Processed)
                    │
                    └── CloudWatch Events trigger at 2 AM
                    └── Auto-scaling based on data size

  Cost: ~$800/month for Glue + S3

Reliability:
  - Glue has built-in retry (3 attempts)
  - Spot interruption handling with checkpointing
  - SNS alerts on failure
  - Manual rerun capability via Step Functions
```

---

## Problem 3: Exactly-Once Payment Pipeline

**Business Context**:
You're building a payment processing pipeline:
- Must process each payment exactly once (financial data)
- 10,000 payments/minute during peak
- Need idempotency for 7 days (retry window)
- Downstream systems: Accounting DB, Notification service, Fraud detection

**Task**: Design for exactly-once semantics across multiple consumers.

**Solution Approach**:
```
Exactly-Once Architecture:

┌─────────────┐     ┌─────────────┐     ┌─────────────────────────────┐
│   Kafka     │────▶│   Flink     │────▶│  Transactional Outbox       │
│  (Payments) │     │  (Enrich)   │     │  (idempotent writes)        │
└─────────────┘     └─────────────┘     └─────────────────────────────┘
                                                │
                          ┌─────────────────────┼─────────────────────┐
                          ▼                     ▼                     ▼
                   ┌─────────────┐      ┌─────────────┐      ┌─────────────┐
                   │  Accounting │      │ Notification│      │   Fraud     │
                   │     DB      │      │   Service   │      │  Detection  │
                   └─────────────┘      └─────────────┘      └─────────────┘

Key Mechanisms:

1. Idempotency Keys
   - Payment ID serves as idempotency key
   - Store processed IDs in Redis (7-day TTL)
   - Check Redis before processing

2. Transactional Outbox Pattern
   - Write to business table + outbox table in same transaction
   - Separate publisher reads outbox and publishes to Kafka
   - Ensures no lost messages on DB failure

3. Two-Phase Commit (for critical flows)
   - Prepare phase: Reserve funds
   - Commit phase: Complete transaction
   - Rollback on failure

4. Monitoring
   - Reconciliation job: Sum(Kafka) == Sum(DB) every hour
   - Alert on discrepancies > 0.01%
```

---

## Session Flow Example

**You**: "Welcome! Today we'll design a data pipeline together. I'll present a business scenario, and I'd like you to walk me through your design process. Don't worry about getting everything perfect - I'm here to help you think through the trade-offs. Ready to start?"

**Candidate**: "Yes, let's do it!"

**You**: "Great! Here's the scenario: [present problem]. Before we dive into the architecture, what questions do you have about the requirements?"

[Let candidate ask clarifying questions, provide answers]

**You**: "Good questions! Now let's start with the high-level architecture. Can you sketch out the main layers?"

[Guide through each layer, probing on tool choices]

**You**: "Interesting choice of Flink. What would be the trade-offs if we used Spark Streaming instead?"

[Push back gently if needed, help them discover blind spots]

**You**: "Now let's talk about failure modes. What happens if your Kafka cluster loses a broker?"

[Continue with failure scenarios...]
