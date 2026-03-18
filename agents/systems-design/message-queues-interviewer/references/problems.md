# Problem Bank: Message Queues & Event Streaming

This document contains the complete problem bank with solutions and walkthroughs for the Message Queues & Event Streaming interviewer skill.

---

## Problem 1: RabbitMQ vs Kafka

**Question**: "We have a video rendering pipeline. Users upload videos, and we put a job on a queue for worker servers to process. Should we use Kafka or RabbitMQ?"

### Walkthrough

**Key Distinction**: This is a work queue pattern, not an event streaming pattern.

**Ideal Answer**:
Use RabbitMQ (or AWS SQS). This is a classic "work queue" pattern. We want multiple workers to pull jobs, process them, and delete them from the queue. We don't care about the ordering of the jobs, and we don't need to replay them.

**Key Concepts**:
- RabbitMQ: smart broker, dumb consumer. Messages are deleted after ACK.
- Kafka: dumb broker, smart consumer. Messages are retained in an append-only log.
- Work queue pattern: distribute tasks to a pool of workers, each task processed once
- Event streaming pattern: broadcast events to multiple consumers, replay capability needed

### Hints (Progressive)
1. "Do multiple different systems need to read this video rendering job, or just the render workers?"
2. "Do we need to keep the job around after it's successfully rendered?"
3. "Kafka is an append-only log meant for broadcasting events. RabbitMQ is a message broker meant for distributing work queues."
4. Full solution above.

---

## Problem 2: Poison Pills

**Question**: "A consumer reads a message from a RabbitMQ queue. Due to a bug in the JSON payload, the consumer throws an exception and crashes. The message is not ACKed. What happens next, and how do we stop the system from being stuck forever?"

### Walkthrough

**Root Cause**: An unacknowledged message gets requeued by RabbitMQ. The next consumer picks it up, crashes on the same bad payload, requeues it -- creating an infinite loop.

**Ideal Answer**:
Use a Dead Letter Queue (DLQ). Configure the consumer to catch the exception, log it, and explicitly NACK (reject) the message without requeuing, OR configure the queue with a `max_deliveries` policy. Once the limit is hit, the broker moves the message to a DLQ where engineers can inspect the bad payload.

**Key Concepts**:
- Dead Letter Queue: a holding area for messages that can't be processed
- max_deliveries / delivery count: tracks how many times a message has been delivered
- NACK without requeue: explicitly reject a message
- Alerting on DLQ depth: notify engineers when poison pills accumulate

### Hints (Progressive)
1. "If the message isn't ACKed, what does RabbitMQ do with it?"
2. "RabbitMQ will requeue it. The next consumer picks it up, crashes, requeues it... infinite loop."
3. "How can we tell the queue to stop trying after X attempts?"
4. Full solution above.

---

## Problem 3: Guaranteed Ordering

**Question**: "In Kafka, how do we guarantee that all events for a specific `user_id` are processed in the exact order they were generated?"

### Walkthrough

**Key Insight**: Kafka only guarantees ordering within a single Partition, not across the entire topic.

**Ideal Answer**:
When the Producer sends the message, it must use the `user_id` as the message Key. Kafka hashes the key (`hash(user_id) % num_partitions`) to determine the partition. Because `User A` always hashes to the same partition, and a partition is consumed sequentially by a single worker thread, ordering is guaranteed.

**Key Concepts**:
- Partition-level ordering: Kafka's fundamental ordering guarantee
- Message keys: determine partition assignment
- Consumer group assignment: one partition = one consumer within a group
- Partition rebalancing: happens when consumers join/leave, can cause brief reprocessing

### Hints (Progressive)
1. "Does Kafka guarantee ordering across the entire topic?"
2. "No, Kafka only guarantees ordering within a single Partition."
3. "How do we make sure all events for `User A` go to the same Partition?"
4. Full solution above.

---

## Problem 4: At-Least-Once Delivery and Idempotency

**Question**: "Our worker reads a message, charges the user's credit card via Stripe, and then crashes before acknowledging the message. What happens next?"

### Walkthrough

**Root Cause**: The message broker will redeliver the message since it was never ACKed. The worker will process it again, potentially charging the user twice.

**Ideal Answer**:
Implement idempotency. Before processing, generate or use the message's unique ID as an idempotency key. Store it in a database table (`processed_payments`). On redelivery, check if the key exists -- if so, skip processing and ACK the message. Also pass the idempotency key to Stripe so they deduplicate on their end.

**Key Concepts**:
- At-least-once: messages may be delivered more than once, consumer must handle duplicates
- Idempotency key: a unique identifier that prevents duplicate processing
- Outbox pattern: write to DB and message queue atomically
- Exactly-once is a myth for external side effects (Stripe, emails, etc.)

---

## Problem 5: Consumer Scaling Limits in Kafka

**Question**: "We have a Kafka topic with 4 partitions. Our team wants to add 10 consumer instances to our consumer group to speed up processing. Will this work?"

### Walkthrough

**Ideal Answer**:
No. In a Kafka consumer group, each partition is assigned to exactly one consumer. With 4 partitions and 10 consumers, only 4 consumers will be active -- the other 6 will sit idle. To scale to 10 consumers, you need at least 10 partitions.

**Key Concepts**:
- Consumer group parallelism is bounded by partition count
- Over-provisioning partitions: plan for future scale but be aware of overhead
- Partition rebalancing: triggered when consumers join or leave
- Alternatives: sub-partition processing within a consumer using thread pools

---

## Sample Interview Session

### Opening
"Hey, welcome! Let's get into it. We're building an order processing system for an e-commerce platform. When a user places an order, we need to charge their card, update inventory, send a confirmation email, and notify the warehouse. Should we use Kafka or RabbitMQ for this? Walk me through your thinking."

### Follow-up Probes
- "The payment worker crashes after charging the card but before ACKing. What happens?"
- "User A updates their shipping address twice in quick succession. How do we ensure the updates are applied in order?"
- "A malformed order message keeps crashing our consumer. How do we stop it from blocking the entire queue?"

### Wrap-up
Generate scorecard based on the Evaluation Rubric. Highlight strengths, improvement areas, and recommended resources.
