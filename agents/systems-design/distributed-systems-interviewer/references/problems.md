# Distributed Systems — Problem Bank, Walkthroughs & Calculations

---

## 📝 Complete System Design Walkthrough

### Problem: CAP Theorem Application
**Scenario**: Designing a shopping cart for an e-commerce site.

**Back-of-the-Envelope**:
- Users: 100M active users
- Cart operations: ~500M/day (~6,000/sec)
- Cart size: ~1KB per user = 100GB total
- Replication factor: 3 across 2+ datacenters

**Walkthrough**:
1. Network partition occurs between datacenter A and B.
2. CP behavior: Reject writes to maintain consistency. Users get errors when adding items to cart. Lost revenue.
3. AP behavior: Accept writes on both sides. After partition heals, merge carts (union of items). Users might see temporary inconsistencies but never lose items.
4. Amazon chose AP for Dynamo (shopping cart). Better to accept the write and resolve conflicts later.

### Problem: Quorum Configuration
**Scenario**: Distributed database with N=5 replicas.

**Back-of-the-Envelope**:
- Strict quorum: R + W > N
- Options for N=5:
  - W=3, R=3: Balanced. Tolerates 2 node failures for reads and writes.
  - W=5, R=1: Fast reads, but any single node failure blocks writes.
  - W=1, R=5: Fast writes, but any single node failure blocks reads.
  - W=2, R=4: Slightly faster writes, slower reads.

**Walkthrough**:
1. For high write availability + strong consistency: W=2, R=4.
2. For balanced: W=3, R=3 (most common).
3. Sloppy quorum: When strict quorum nodes aren't available, write to any N nodes with hinted handoff. Improves availability but sacrifices strong consistency guarantee.

### Problem: Leader Election with Fencing
**Scenario**: Distributed system with leader writing to shared storage.

**Back-of-the-Envelope**:
- GC pauses: 10ms-30s (worst case with full GC)
- Lease TTL: typically 10-30 seconds
- Clock skew between nodes: up to 100ms (NTP) or <7ms (TrueTime)

**Walkthrough**:
1. Leader 1 holds lease with Epoch=4.
2. Leader 1 enters 30s GC pause.
3. Lease expires. Consensus system (Raft) elects Leader 2 with Epoch=5.
4. Leader 2 begins writing to storage with Epoch=5.
5. Leader 1 wakes up, tries to write with Epoch=4.
6. Storage node rejects: "Already seen Epoch 5, rejecting Epoch 4."
7. Key insight: The fencing token must be checked by the storage layer, not just the client.

---

## 🎬 Sample Session Scenarios

### Scenario 1: Vector Clock Conflict Resolution
**Setup**: Three nodes (A, B, C) in an eventually consistent system.

```
Event 1: Client writes x=1 to Node A
  A: [1,0,0]

Event 2: A replicates to B
  B: [1,1,0]

Event 3: Client writes x=2 to Node A (concurrent with Event 4)
  A: [2,0,0]

Event 4: Client writes x=3 to Node C (concurrent with Event 3)
  C: [0,0,1]

Conflict: A has [2,0,0] with x=2, C has [0,0,1] with x=3.
Neither vector dominates the other -> concurrent writes -> conflict!
Resolution options: Last-Writer-Wins (LWW), application-level merge, or CRDTs.
```

### Scenario 2: Network Partition During Raft Consensus
**Setup**: 5-node Raft cluster. Partition splits into {A, B} and {C, D, E}.

```
Before partition:
  Leader: A, Term: 5

After partition:
  {A, B}: A is still leader but can't reach majority (needs 3/5).
          A cannot commit new entries. Reads may be stale.

  {C, D, E}: Nodes timeout waiting for heartbeat from A.
             Election starts. C wins with Term 6.
             C can commit entries (has majority 3/5).

After partition heals:
  A discovers Term 6 > Term 5. A steps down.
  A and B receive committed entries from C and update their logs.
```
