# Problem Bank: Microservices Architecture

## Problem 1: Service Boundaries — E-Commerce Decomposition

### Problem Statement
You have a monolithic e-commerce application with the following modules: Users, Orders, Inventory, Payments, Shipping, Notifications, and Reviews. The application is struggling to scale — deploys take 2 hours, a bug in the Reviews module brought down the entire system last week, and the team has grown to 40 engineers stepping on each other's toes.

**Task**: Identify service boundaries using Domain-Driven Design (DDD). Which modules become services? Which stay together? Justify your decisions.

### Solution Walkthrough

**Step 1: Identify Bounded Contexts**
Map out which data and logic naturally belong together:

- **Order Context**: Orders, Order Items, Order Status
- **Inventory Context**: Stock levels, Warehouse locations, Reservations
- **Payment Context**: Payment processing, Refunds, Credit card tokens
- **User/Identity Context**: Authentication, User profiles, Preferences
- **Shipping Context**: Shipping labels, Tracking, Carrier integration
- **Notification Context**: Email, SMS, Push notifications
- **Review Context**: Product reviews, Ratings

**Step 2: Evaluate Coupling**
- Orders and Inventory are tightly coupled during checkout (reservation). However, they have different scaling needs — Inventory is read-heavy, Orders are write-heavy during sales.
- Notifications is a pure consumer — it reacts to events from other services. Good candidate for a separate service.
- Reviews are almost entirely independent. They can be a separate service with minimal coupling.

**Step 3: Recommended Service Boundaries**

| Service | Owns | Communicates With |
|---------|------|-------------------|
| User Service | Auth, profiles | All (identity provider) |
| Order Service | Orders, order items | Inventory (sync), Payment (saga), Shipping (event) |
| Inventory Service | Stock, reservations | Order (sync response) |
| Payment Service | Transactions, refunds | Order (saga participant) |
| Shipping Service | Labels, tracking | Order (event consumer) |
| Notification Service | Templates, delivery | All (event consumer) |
| Review Service | Reviews, ratings | Product catalog (read) |

**Step 4: Migration Strategy — Strangler Fig**
1. Start with the least coupled service: **Reviews** or **Notifications**
2. Put an API Gateway in front of the monolith
3. Route `/api/reviews/*` to the new Review Service
4. Gradually extract more services
5. The monolith shrinks until it can be decommissioned

### Follow-up Questions
- "What if the Order Service needs the user's name for an order confirmation? Does it call the User Service every time?"
  - Answer: Event-carried state transfer. User Service publishes `UserUpdated` events. Order Service caches a local read-only copy of relevant user data.
- "What happens if the Notification Service is down? Do we lose notifications?"
  - Answer: Use a durable message queue (Kafka, RabbitMQ with persistence). Messages are retained until the Notification Service comes back.

---

## Problem 2: Saga Pattern — Distributed Order Processing

### Problem Statement
Design the checkout flow for an e-commerce system using the Saga Pattern. The flow must:
1. Create an order (Order Service)
2. Reserve inventory (Inventory Service)
3. Charge the customer's card (Payment Service)
4. Confirm the order (Order Service)

If any step fails, all previous steps must be compensated (rolled back).

### Solution Walkthrough

**Choreography-Based Saga:**

```
Happy Path:
  Order Service      -> Pub: OrderCreated
  Inventory Service  -> Sub: OrderCreated -> Reserve Items -> Pub: InventoryReserved
  Payment Service    -> Sub: InventoryReserved -> Charge Card -> Pub: PaymentSucceeded
  Order Service      -> Sub: PaymentSucceeded -> Confirm Order -> Pub: OrderConfirmed

Failure Path (Payment Fails):
  Payment Service    -> Sub: InventoryReserved -> Charge Card FAILS -> Pub: PaymentFailed
  Inventory Service  -> Sub: PaymentFailed -> Compensate: Un-reserve Items -> Pub: InventoryReleased
  Order Service      -> Sub: PaymentFailed -> Compensate: Mark Order FAILED
```

**Orchestration-Based Saga:**

```
Order Saga Orchestrator:
  1. Call Order Service: Create Order (PENDING)
  2. Call Inventory Service: Reserve Items
     - If FAIL: Call Order Service: Cancel Order -> END
  3. Call Payment Service: Charge Card
     - If FAIL: Call Inventory Service: Release Items -> Call Order Service: Cancel Order -> END
  4. Call Order Service: Confirm Order (APPROVED)
```

**Trade-offs:**

| Aspect | Choreography | Orchestration |
|--------|-------------|---------------|
| Coupling | Loose (event-driven) | Tighter (orchestrator knows all steps) |
| Complexity | Hard to trace flow | Centralized, easier to debug |
| Single Point of Failure | None | Orchestrator |
| Best For | Simple sagas (3-4 steps) | Complex sagas (5+ steps) |

### Back-of-Envelope Calculation
- Assume 1,000 orders/second at peak
- Each saga step takes ~50ms
- Happy path: 4 steps * 50ms = 200ms end-to-end
- With Kafka event propagation latency: add ~10ms per hop = ~240ms total
- Thread pool sizing: 1,000 req/s * 0.24s = 240 concurrent threads needed per service

---

## Problem 3: Cascading Failure Prevention

### Problem Statement
Your microservices architecture has the following call chain:
```
Web App -> API Gateway -> Order Service -> Payment Service -> External Bank API
```
The External Bank API starts responding in 30 seconds instead of 100ms. Within 5 minutes, the entire system is down. Diagnose the problem and design a solution.

### Solution Walkthrough

**Diagnosis: Thread Pool Exhaustion Cascade**
1. Payment Service threads block waiting for Bank API (30s each)
2. Payment Service thread pool fills up (e.g., 200 threads * 30s = can only handle ~7 req/s)
3. Order Service calls to Payment Service start timing out
4. Order Service thread pool fills up waiting for Payment Service
5. API Gateway calls to Order Service start timing out
6. API Gateway thread pool fills up
7. Web App gets no responses -> users see errors

**Solution: Defense in Depth**

Layer 1: **Timeouts**
- Set aggressive timeouts at each level
- Payment -> Bank API: 2s timeout
- Order -> Payment: 3s timeout
- Gateway -> Order: 5s timeout

Layer 2: **Circuit Breakers**
- Payment Service wraps Bank API calls in a circuit breaker
- Threshold: 5 failures in 10 seconds -> OPEN
- Open state: Return fallback (e.g., "Payment processing queued")
- Half-open after: 30 seconds

Layer 3: **Bulkheads**
- Separate thread pools for different downstream calls
- Order Service: 100 threads for Payment, 100 threads for Inventory
- If Payment pool exhausts, Inventory calls still work

Layer 4: **Fallback & Graceful Degradation**
- When circuit is OPEN, offer alternatives:
  - Queue the payment for async processing
  - Show "We'll confirm your order shortly" instead of hard failure

### Back-of-Envelope: Impact of Circuit Breaker
- Without CB: 1,000 req/s * 30s wait = 30,000 blocked threads = system crash
- With CB (trips after 5 failures): 5 slow requests * 30s + remaining requests fail fast (10ms) = system stays up, only ~5 users experience 30s wait

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hey, welcome! I'm Alex, I lead the platform team here. Today we're going to talk about microservices — not the buzzwordy version, but the real engineering challenges. Sound good? Let's dive in."

### Phase 1: Decomposition (15 minutes)
**Interviewer**: "So here's the situation. We have a monolithic e-commerce application — Users, Orders, Inventory, Payments, all in one big Django app with a single PostgreSQL database. We're growing fast. Last week, a junior dev pushed a bad migration to the Reviews table and it took down the whole checkout flow. How would you start breaking this apart?"

*[Candidate discusses service boundaries]*

**Interviewer**: "Interesting. You split Users, Orders, Inventory, and Payments into separate services. But I notice you have the Order Service calling the User Service synchronously to get the user's name for the confirmation email. What happens if the User Service is down?"

### Phase 2: Communication & Data (15 minutes)
**Interviewer**: "Good, you mentioned event-driven communication. Let's go deeper. A customer clicks 'Buy Now'. Walk me through the exact sequence of events across your services. What happens if the payment fails after inventory is already reserved?"

*[Candidate discusses Saga pattern]*

**Interviewer**: "You mentioned the Saga pattern with compensating transactions. Do you prefer choreography or orchestration? What are the trade-offs?"

### Phase 3: Resilience (10 minutes)
**Interviewer**: "Let's talk about what happens when things go wrong. The Payment Service depends on an external bank API. That API starts responding in 30 seconds instead of 100ms. Walk me through what happens to your system."

### Phase 4: API Gateway (5 minutes)
**Interviewer**: "Last thing — you have a mobile app and a web app hitting these services. Do they call each service directly? What sits in between?"

### Closing (3 minutes)
**Interviewer**: "Great discussion. Let me give you some feedback..."

*[Generate scorecard]*
