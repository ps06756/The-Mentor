# Problem Bank: Cascading Failure Debugging

## Problem 1: Thread Pool Exhaustion from Missing Circuit Breaker

### Problem Statement
Payment Service depends on an external Bank API. The Bank API started responding in 30 seconds instead of 100ms. Within 5 minutes, Payment Service, Order Service, Cart Service, and the Web Frontend are all down. No code was deployed. Trace the cascade and design the fix.

### Solution Walkthrough

**The Cascade Chain:**

```
Step 1: Bank API slows to 30s response time
Step 2: Payment Service
  - 200 thread pool, each thread now blocked 30s (was 100ms)
  - Max throughput: 200/30 = 6.7 req/s (was 200/0.1 = 2,000 req/s)
  - 99.7% capacity loss
  - All 200 threads occupied -> new requests queue -> queue fills -> 503 errors

Step 3: Order Service
  - Calls Payment Service, gets timeouts/503s
  - Retries 3x with no backoff -> 3x amplification
  - Order Service threads now blocked waiting for Payment (no timeout set)
  - Order Service thread pool exhausted -> 503 errors

Step 4: Cart Service
  - Calls Order Service for price calculation
  - Order Service returns 503 -> Cart retries 3x
  - Cart thread pool exhausted -> 504 gateway timeout

Step 5: Web Frontend
  - All backend calls fail -> shows blank page
  - 100% user impact
```

**Immediate Mitigation (do these NOW):**
1. Enable circuit breaker on Payment -> Bank API calls (if available as feature flag)
2. If no circuit breaker exists: block Bank API calls at the network level (iptables/security group)
3. Restart Order Service and Cart Service (they'll recover once Payment stops blocking them)
4. Enable maintenance page on Web Frontend for affected features

**Root Cause Fix:**
1. Add circuit breaker: Payment -> Bank API
   - Trip after 50% failure rate over 10 seconds
   - Open state: return "payment pending" immediately
   - Half-open: test 5 requests after 30 seconds
2. Add explicit timeouts on ALL downstream calls
   - Payment -> Bank API: 500ms timeout (was infinite)
   - Order -> Payment: 1s timeout
   - Cart -> Order: 2s timeout
3. Add bulkhead isolation
   - Separate thread pool for Bank API calls (50 threads)
   - Remaining 150 threads serve other Payment endpoints
4. Fix retry policy
   - Exponential backoff with jitter
   - Max 3 retries with retry budget (max 10% retry traffic)

**Prevention:**
- Mandatory timeout policy: every outbound HTTP call must have explicit timeout
- Circuit breaker library in service template (opt-out requires VP approval)
- Chaos testing: monthly "slow dependency" game day
- Service mesh (Istio/Envoy) with default timeout and circuit breaker policies

### Follow-up Questions
- "The circuit breaker is open. Payment returns 'payment pending'. What does the user see?"
- "How do you process the queued payments once the Bank API recovers?"
- "What if the Bank API is slow for some requests but fast for others (intermittent)?"

---

## Problem 2: Retry Storm with Cross-Layer Amplification

### Problem Statement
During the outage described above, monitoring shows that Payment Service received 12,000 req/s during the incident -- 6x its normal 2,000 req/s. No additional users were on the site. Where is the extra traffic coming from?

### Solution Walkthrough

**Traffic Amplification Analysis:**

```
Normal traffic flow:
  Users -> Frontend -> Cart -> Order -> Payment: 2,000 req/s

During outage (with retries):
  Layer 1: Order Service retries Payment 3x
    - Payment receives: 2,000 * 3 = 6,000 req/s from Order

  Layer 2: Cart Service retries Order 3x
    - Order receives: 2,000 * 3 = 6,000 req/s from Cart
    - Each of those retries Payment 3x: 6,000 * 3 = 18,000 req/s (!)
    - But Order is also failing, so not all retries reach Payment

  Layer 3: Frontend retries Cart 2x
    - Further amplification upstream

  Observed at Payment: ~12,000 req/s (some dropped at Order before reaching Payment)
  Amplification factor: 6x
```

**The Thundering Herd Effect:**
- All retries happen simultaneously (no jitter)
- Every time Payment briefly recovers, it gets slammed by queued retries
- Recovery is impossible: every brief improvement is punished with a burst

**Fix: Retry Budget Pattern**
```
Service-Level Retry Budget:
  - Track: (retry_requests / total_requests) over 60-second window
  - If retry_ratio > 10%: stop retrying, fail immediately
  - This caps the amplification factor at 1.1x (instead of 3x per layer)

Per-Request Retry Header:
  X-Retry-Count: 0  (original request)
  X-Retry-Count: 1  (first retry)
  X-Retry-Count: 2  (second retry)

  Rule: If X-Retry-Count >= 2 when received, do NOT retry downstream
  This prevents cross-layer amplification
```

**Implementation:**
```java
// Retry budget implementation
class RetryBudget {
    private final RateLimiter retryLimiter;
    private final AtomicLong totalRequests = new AtomicLong();
    private final AtomicLong retryRequests = new AtomicLong();

    boolean allowRetry() {
        double ratio = retryRequests.get() / (double) totalRequests.get();
        return ratio < 0.10; // 10% retry budget
    }
}
```

### Follow-up Questions
- "What's the difference between a retry budget and a circuit breaker?"
- "Can retry amplification happen with async/message-queue-based communication?"
- "How do you test for retry storms before they happen in production?"

---

## Problem 3: Cascading Timeout Without Deadline Propagation

### Problem Statement
Frontend has a 10-second timeout for API calls. Cart Service has a 30-second timeout for Order calls. Order Service has a 60-second timeout for Payment calls. During the outage, Frontend times out at 10 seconds but Order Service is still processing the request for another 50 seconds. Resources are wasted on requests the user has already abandoned. Fix the timeout architecture.

### Solution Walkthrough

**The Problem: Misaligned Timeouts**
```
Frontend (10s timeout) -> Cart (30s timeout) -> Order (60s timeout) -> Payment (no timeout)

User clicks checkout at T=0:
  T=0s:  Frontend calls Cart
  T=10s: Frontend times out, shows error to user. USER IS GONE.
  T=10s: Cart is still waiting for Order (has 20s left on its 30s timeout)
  T=30s: Cart times out, logs error. But nobody is listening.
  T=30s: Order is still waiting for Payment (has 30s left on its 60s timeout)
  T=60s: Order times out. Resources wasted for 50 seconds after user left.
  T=60s: Payment is still processing (no timeout). Thread blocked indefinitely.
```

**Fix: Deadline Propagation**
```
Principle: Inner timeouts must be shorter than outer timeouts.
Correct configuration:
  Frontend: 10s total
  Cart: 8s (leaves 2s for Frontend processing)
  Order: 5s (leaves 3s for Cart processing)
  Payment: 3s (leaves 2s for Order processing)

Implementation: Propagate deadline via header
  X-Request-Deadline: 2026-03-18T14:00:10Z  (absolute time)

  Each service checks: if now() > deadline, return 504 immediately
  Each service sets downstream timeout to: min(own_timeout, deadline - now())
```

**Deadline Propagation in Practice:**
```
T=0s: Frontend sets X-Request-Deadline: T+10s
T=0s: Cart receives request, checks deadline (10s left), calls Order with timeout=min(8s, 10s)=8s
T=1s: Order receives request, checks deadline (9s left), calls Payment with timeout=min(5s, 9s)=5s
T=2s: Payment receives request, checks deadline (8s left), calls Bank with timeout=min(3s, 8s)=3s
T=5s: Payment times out on Bank -> returns error to Order
T=6s: Order returns error to Cart
T=7s: Cart returns error to Frontend
T=8s: Frontend shows error to user (within 10s budget!)
```

**Prevention:**
- Timeout policy: documented maximum per tier (edge: 10s, mid-tier: 5s, backend: 2s)
- Deadline propagation via gRPC deadlines or HTTP headers
- Monitoring: alert if any request's total latency exceeds the frontend timeout
- Service mesh can enforce timeouts at the infrastructure level

### Follow-up Questions
- "How does gRPC handle deadline propagation natively?"
- "What happens if a service sets a timeout longer than the propagated deadline?"
- "How do you handle long-running operations (like payment processing) that legitimately take > 10s?"

---

## Sample Session Flow

### Opening
**Interviewer**: "We're in a war room. Payment Service went sideways 12 minutes ago and now everything is down. I'm the incident commander. 100% of users are seeing errors. The VP is on the call. What's your first move?"

### Phase 1: Triage (5 minutes)
*[Candidate asks about the timeline and affected services]*
**Interviewer**: "Here's what we know so far..."
*[Present the timeline and service map]*
**Interviewer**: "Where do you start?"

### Phase 2: Cascade Analysis (15 minutes)
**Interviewer**: "Good, you identified that Payment is the origin. But here's my question: Payment is ONE service. It handles payments. Why is the Cart page also down? Cart doesn't need payments to show a shopping cart."

*[Candidate traces the cascade]*

### Phase 3: Mitigation (10 minutes)
**Interviewer**: "You understand the cascade. Now stop the bleeding. The Bank API is still slow and we can't fix that. What do we do RIGHT NOW?"

### Phase 4: Postmortem (15 minutes)
**Interviewer**: "Incident resolved. Write me the postmortem. I want root causes, action items with owners, and a timeline. Make it blameless."

*[Generate scorecard]*
