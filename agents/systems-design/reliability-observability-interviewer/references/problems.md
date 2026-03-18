# Problem Bank: Reliability & Observability

## Problem 1: Observability Stack Design

### Problem Statement
You are the SRE for a microservices platform with 20 services. The CEO just complained that "checkout is slow" for some users but not others. You have no observability tooling in place. Design a complete observability stack and explain how you would use it to diagnose this specific issue.

### Solution Walkthrough

**The Three Pillars of Observability:**

**1. Metrics (Prometheus + Grafana)**
- Instrument each service with RED metrics:
  - **Rate**: Requests per second
  - **Errors**: Error rate (4xx, 5xx)
  - **Duration**: Latency percentiles (p50, p95, p99)
- Create dashboards per service showing:
  - Request rate over time
  - Error rate over time
  - Latency percentiles over time
- Set up alerts:
  - p99 latency > 500ms for 5 minutes -> PagerDuty alert
  - Error rate > 5% for 2 minutes -> PagerDuty alert

**2. Distributed Tracing (Jaeger / OpenTelemetry)**
- Inject Trace ID at the API Gateway
- Propagate via headers: `X-Trace-ID`, `X-Span-ID`, `X-Parent-Span-ID`
- Each service creates spans for:
  - Incoming HTTP/gRPC requests
  - Outgoing HTTP/gRPC calls
  - Database queries
  - Cache lookups
- Trace visualization shows the waterfall of a single request

**3. Centralized Logging (ELK Stack / Datadog Logs)**
- Structured JSON logging in every service
- Include Trace ID in every log line
- Ship logs to centralized aggregator
- Enable full-text search and filtering

**Diagnosing "Checkout is Slow":**

Step 1: Check Grafana dashboard for Checkout Service
- p50 latency: 100ms (normal)
- p99 latency: 8,000ms (10x normal!) -> Tail latency problem

Step 2: Look at Checkout Service dependencies
- Payment Service p99: 7,500ms -> This is the bottleneck

Step 3: Pull a slow trace from Jaeger
```
Checkout Service (8200ms)
  -> Validate Cart (50ms)
  -> Payment Service (7900ms)      <-- SLOW
     -> External Bank API (7800ms) <-- ROOT CAUSE
  -> Send Confirmation (250ms)
```

Step 4: Check Payment Service logs filtered by Trace ID
```
[WARN] External bank API response time: 7800ms (timeout threshold: 10000ms)
[INFO] Bank API returned: {"status": "processing", "retry_after": 5}
```

Root cause: External bank API is experiencing intermittent slowness.

### Follow-up: What Do We Do About It?
- Short term: Circuit breaker on Payment -> Bank API calls
- Fallback: Queue payments for async processing, show user "Payment processing..."
- Long term: Negotiate SLA with bank API provider, consider backup payment processor

---

## Problem 2: Designing Retry Logic That Doesn't Kill Your System

### Problem Statement
Your Order Service calls the Inventory Service to reserve items. The Inventory Service occasionally fails with 503 (Service Unavailable). Your team implemented retries, but during the last outage, the retries made things worse — the Inventory Service was overwhelmed and took 30 minutes to recover instead of the expected 2 minutes.

Design a retry strategy that helps recovery instead of hindering it.

### Solution Walkthrough

**What Went Wrong: The Thundering Herd**
- 1,000 req/s hitting Inventory Service
- Inventory goes down, returns 503
- All 1,000 clients retry immediately
- Inventory starts recovering, gets hit with 2,000 req/s (original + retries)
- Falls over again
- Clients retry again: 3,000 req/s
- Cascading failure: system never recovers

**Proper Retry Strategy:**

**1. Exponential Backoff**
```
Attempt 1: Wait 1 second
Attempt 2: Wait 2 seconds
Attempt 3: Wait 4 seconds
Attempt 4: Wait 8 seconds
Max attempts: 4
Max wait: 16 seconds
```

**2. Add Jitter (Randomness)**
```
Attempt 1: Wait random(0, 1) seconds    e.g., 0.7s
Attempt 2: Wait random(0, 2) seconds    e.g., 1.3s
Attempt 3: Wait random(0, 4) seconds    e.g., 2.8s
Attempt 4: Wait random(0, 8) seconds    e.g., 5.1s
```
This spreads retries over time instead of synchronized bursts.

**3. Circuit Breaker (Fail Fast)**
- After 5 consecutive failures in 10 seconds -> Circuit OPEN
- While OPEN: Don't even attempt the call, return fallback immediately
- After 30 seconds: Try 1 request (HALF-OPEN)
- If success: Circuit CLOSED, resume normal traffic
- If fail: Circuit OPEN again for 30 seconds

**4. Idempotency**
- Every request includes an idempotency key: `X-Idempotency-Key: order-123-reserve`
- Inventory Service checks if this reservation was already processed
- If yes: Return cached response (don't double-reserve)
- Prevents duplicate side effects from retries

**5. Rate Limiting / Token Bucket at Client**
- Each client limits itself to N retries per minute
- Prevents any single client from overwhelming the service

### Back-of-Envelope: Impact Comparison

| Strategy | Load on Inventory During Outage |
|----------|--------------------------------|
| No retries | 0 req/s (users get errors) |
| Immediate retries (3x) | 3,000 req/s (3x normal) |
| Immediate retries + jitter | ~1,500 req/s (spread out) |
| Exp backoff + jitter | ~200 req/s (decreasing over time) |
| Circuit breaker | 0 req/s after trip (best for recovery) |

---

## Problem 3: Multi-Region Disaster Recovery Design

### Problem Statement
Your company's e-commerce platform generates $10M/day in revenue. The business requires:
- RPO: 5 minutes (max 5 minutes of data loss)
- RTO: 1 hour (max 1 hour of downtime)

Current setup: Single region (us-east-1), single PostgreSQL database, 50 application servers.

Design the disaster recovery architecture.

### Solution Walkthrough

**Architecture: Warm Standby (Pilot Light + Auto-Scaling)**

**Primary Region (us-east-1):**
- 50 application servers (auto-scaling group)
- PostgreSQL primary (RDS Multi-AZ)
- Redis cluster (ElastiCache)
- S3 for static assets

**Secondary Region (us-west-2):**
- 5 application servers (pilot light — minimum footprint)
- PostgreSQL read replica (async replication, ~1-2 minute lag)
- Redis replica
- S3 cross-region replication
- Auto-scaling configured to scale to 50 servers

**RPO Analysis:**
- PostgreSQL async replication lag: ~1-2 minutes
- RPO target: 5 minutes
- Margin: 3-4 minutes -> Acceptable
- For tighter RPO: Use synchronous replication (but adds latency to writes)

**RTO Breakdown:**
| Step | Time | Running Total |
|------|------|---------------|
| Detect failure (health checks) | 2 min | 2 min |
| Promote DB replica to primary | 5 min | 7 min |
| Scale application servers (5 -> 50) | 10 min | 17 min |
| Update DNS (Route 53 failover) | 2 min | 19 min |
| Warm up caches | 5 min | 24 min |
| Validation and smoke tests | 10 min | 34 min |
| **Total** | | **34 min** |

Target RTO: 1 hour. Actual: ~34 minutes. Margin: 26 minutes -> Acceptable.

**Failover Process:**
1. Route 53 health checks detect us-east-1 is down
2. Automated runbook triggers (AWS Lambda / Step Functions):
   a. Promote PostgreSQL replica to primary in us-west-2
   b. Trigger auto-scaling to increase capacity
   c. Update DNS to point to us-west-2
3. Manual validation:
   a. Run smoke tests against us-west-2
   b. Verify data integrity
   c. Announce recovery to stakeholders

**Cost Analysis:**
- Secondary region (pilot light): ~$5K/month
  - 5 small EC2 instances: $500
  - RDS read replica: $3,000
  - Redis replica: $1,000
  - Misc (networking, storage): $500
- Revenue protected: $10M/day = $416K/hour
- ROI: $5K/month to protect $416K/hour of revenue

### Follow-up Questions
- "What if we need RPO of 0 (zero data loss)?" -> Synchronous replication + Active-Active
- "What about stateful sessions?" -> Externalize sessions to Redis with cross-region replication
- "How do we test this?" -> Chaos engineering, game days, regular failover drills

---

## Problem 4: Defining SLOs for the Checkout Service

### Problem Statement
The product team says "checkout should be fast and reliable." Turn this into measurable SLOs with SLIs and explain how you'd implement error budgets.

### Solution Walkthrough

**SLI (Service Level Indicators) — What We Measure:**

| SLI | Definition | Measurement |
|-----|-----------|-------------|
| Availability | % of requests that return non-5xx | `count(status < 500) / count(all_requests)` |
| Latency | % of requests completing under threshold | `count(duration < 500ms) / count(all_requests)` |
| Correctness | % of checkouts that result in correct order | `count(successful_orders) / count(checkout_attempts)` |

**SLO (Service Level Objectives) — What We Promise:**

| SLO | Target | Window |
|-----|--------|--------|
| Availability | 99.9% | 30-day rolling |
| Latency (p50) | < 200ms | 30-day rolling |
| Latency (p99) | < 1,000ms | 30-day rolling |
| Correctness | 99.99% | 30-day rolling |

**Error Budget Calculation:**
- 99.9% availability over 30 days
- Total minutes in 30 days: 43,200
- Allowed downtime: 43,200 * 0.001 = **43.2 minutes**
- If we've used 30 minutes: 13.2 minutes remaining
- If budget exhausted: Freeze deployments, focus on reliability

**Implementation:**
1. Prometheus collects metrics from all checkout service instances
2. Recording rules calculate SLI percentages every minute
3. Grafana dashboard shows:
   - Current SLO compliance (% of budget remaining)
   - Burn rate (how fast we're consuming budget)
   - Projected budget exhaustion date
4. Alerts:
   - Budget burn rate > 2x normal -> Slack notification
   - Budget remaining < 25% -> PagerDuty alert
   - Budget exhausted -> Deploy freeze, incident review

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hi, I'm the SRE lead. I've been on-call for this platform for three years and I've seen every kind of failure you can imagine. Today I want to see how you think about keeping systems alive when everything is trying to kill them. Let's jump right in."

### Phase 1: Observability (10 minutes)
**Interviewer**: "We have 20 microservices. A user just tweeted that checkout is slow. How do you figure out why? You can ask me questions about our current setup."

*[Candidate discusses observability approach]*

**Interviewer**: "You mentioned distributed tracing. How does the trace get propagated across services? What happens if one service doesn't propagate the trace ID?"

### Phase 2: Fault Tolerance (15 minutes)
**Interviewer**: "The Payment API we depend on is occasionally timing out. Our checkout service runs out of threads and stops serving any requests — even ones that don't need the Payment API. Two questions: why does this happen, and how do we prevent it?"

*[Candidate discusses circuit breakers, bulkheads]*

**Interviewer**: "Good, you mentioned retries. But during the last outage, retries made things worse. The Inventory Service was getting hammered with 3x normal traffic. How do we fix the retry strategy?"

### Phase 3: Disaster Recovery (10 minutes)
**Interviewer**: "Our primary region just had a major outage. RPO is 5 minutes, RTO is 1 hour. Walk me through exactly what happens in the next 60 minutes."

### Phase 4: Defining Reliability (10 minutes)
**Interviewer**: "The VP of Engineering asked you: 'Is checkout reliable?' What's your answer, and how do you define 'reliable' in a measurable way?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
