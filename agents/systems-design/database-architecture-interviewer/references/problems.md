# Problem Bank: Database Architecture

This document contains the complete problem bank with solutions and walkthroughs for the Database Architecture interviewer skill.

---

## Problem 1: Choosing Storage Engine

**Question**: "If I am building a system to ingest 100,000 metrics per second from IoT devices, but only reading them occasionally to generate daily reports, what kind of storage engine should I use?"

### Walkthrough

**Root Cause**: This is a write-heavy workload. B-Trees require updating pages in place, which causes disk seeks and is suboptimal for high write throughput.

**Ideal Answer**:
Use an LSM-tree based database like Cassandra, InfluxDB, or RocksDB. They excel at high-throughput write workloads because writes are sequential (append-only), avoiding the random I/O overhead of B-trees.

**Key Concepts**:
- B-Trees: read-optimized, in-place updates, random I/O on writes
- LSM Trees: write-optimized, append-only, sequential I/O, compaction overhead
- MemTable: in-memory buffer that flushes to SSTables on disk
- Compaction: background process that merges SSTables to reclaim space and improve reads

### Hints (Progressive)
1. "Think about whether this workload is read-heavy or write-heavy."
2. "B-Trees require updating pages in place, which causes disk seeks. Is there an append-only structure that is better for fast writes?"
3. "Log-Structured Merge (LSM) trees buffer writes in memory and flush them sequentially to disk."
4. Full solution above.

---

## Problem 2: Sharding Strategy

**Question**: "We need to shard a users table. How do you decide the shard key?"

### Walkthrough

**Common Mistake**: Sharding by creation date creates a hot spot on the newest shard since all new users go to the same node.

**Ideal Answer**:
Choose a shard key based on your most common access pattern. For a users table, lookups are almost always by UserID. Therefore, hash-based sharding on `user_id` is best to ensure even data and load distribution across nodes.

**Key Concepts**:
- Hash-based sharding: even distribution, but no range queries
- Range-based sharding: supports range queries, but risk of hot spots
- Directory-based sharding: flexible lookup table, but adds a point of failure
- Consistent hashing: minimizes data movement when adding/removing nodes

### Hints (Progressive)
1. "What happens if we shard by 'Creation Date'? Where do all the new users go?"
2. "If you shard by creation date, you create a hot spot on the newest shard. We want even distribution."
3. "Hashing the User ID distributes data evenly, but makes range queries difficult."
4. Full solution above.

---

## Problem 3: Mitigating Replication Lag

**Question**: "A user updates their profile picture, the page refreshes, and they see their old picture because the read hit a replica that hasn't caught up yet. How do you fix this?"

### Walkthrough

**Root Cause**: Asynchronous replication means replicas may be seconds behind the primary. A read immediately after a write may hit a stale replica.

**Ideal Answer**:
Implement "Read-your-own-writes" consistency. When a user updates their profile, set a cookie or cache entry with the timestamp of the write. For the next X seconds, or if the replica's timestamp is older than the write timestamp, route that specific user's reads to the Primary DB. All other users can read from the replica.

**Key Concepts**:
- Asynchronous vs synchronous replication tradeoffs
- Read-your-own-writes consistency
- Monotonic reads: ensure a user never sees data go "backward"
- Causal consistency: related operations are seen in order

### Hints (Progressive)
1. "How can the application know which node has the latest data?"
2. "Can we force certain reads to go to the primary node temporarily?"
3. "We could use 'Read-your-own-writes' consistency."
4. Full solution above.

---

## Problem 4: Transaction Isolation Levels

**Question**: "Two users simultaneously try to book the last available seat on a flight. Both read that 1 seat is available. Both proceed to book it. How do you prevent overbooking?"

### Walkthrough

**Ideal Answer**:
Use Serializable isolation level or optimistic concurrency control with version numbers. Alternatively, use `SELECT ... FOR UPDATE` to acquire a row-level lock before checking availability.

**Key Concepts**:
- Read Committed: prevents dirty reads, but allows non-repeatable reads
- Repeatable Read: prevents dirty and non-repeatable reads, but allows phantom reads
- Serializable: prevents all anomalies, but reduces concurrency
- Optimistic locking: check version on write, retry on conflict
- Pessimistic locking: `SELECT FOR UPDATE` to lock rows during the transaction

---

## Problem 5: SQL vs NoSQL Decision

**Question**: "We're building a social network. We need to store user profiles, posts, and friend relationships. We expect to query 'show me all posts from my friends, sorted by time.' Should we use SQL or NoSQL?"

### Walkthrough

**Ideal Answer**:
This depends on scale. At moderate scale, a relational database (PostgreSQL) handles the JOIN between friends and posts efficiently with proper indexing. At massive scale, you might denormalize: maintain a per-user "feed" table (fan-out on write) in a NoSQL store like Cassandra for fast reads, while keeping the source-of-truth relationships in SQL.

**Key Concepts**:
- JOINs are expensive at scale but powerful for flexible queries
- Denormalization trades write complexity for read performance
- Fan-out on write vs fan-out on read
- Polyglot persistence: use different databases for different access patterns

---

## Sample Interview Session

### Opening
"Welcome! Let's dive into databases. I want to start with the fundamentals -- can you walk me through how a B-tree differs from an LSM tree? And given a system that ingests 100,000 IoT metrics per second but only generates daily reports, which would you choose?"

### Follow-up Probes
- "You said you'd use Cassandra. Why not MongoDB or Postgres for this workload?"
- "Now we need to shard our users table across 8 nodes. Walk me through your sharding strategy."
- "A user updates their profile and immediately sees stale data. What's happening and how do you fix it?"

### Wrap-up
Generate scorecard based on the Evaluation Rubric. Highlight strengths, improvement areas, and recommended resources.
