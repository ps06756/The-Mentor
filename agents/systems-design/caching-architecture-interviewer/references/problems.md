# Problem Bank: Caching Architecture

This document contains the complete problem bank with solutions and walkthroughs for the Caching Architecture interviewer skill.

---

## Problem 1: Cache Penetration

**Question**: "Attackers are requesting user profiles for User IDs that don't exist (e.g., ID=999999). It misses the cache, hits the database, returns null, and doesn't get cached. Our DB is being overwhelmed. How do we fix this?"

### Walkthrough

**Root Cause**: Non-existent keys bypass the cache entirely and always hit the database.

**Solution 1 - Cache Negative Results**:
Cache the key `user:999999` with a value of `NULL` and a short TTL (e.g., 30 seconds). This prevents repeated lookups for the same non-existent key.

**Solution 2 - Bloom Filter**:
A Bloom Filter is a highly memory-efficient data structure that can tell you with 100% certainty if an item does *not* exist. Put it in front of the cache. If the filter says "Not exists", immediately return 404 without hitting Redis or the DB.

**Key Concepts**:
- Bloom Filters: probabilistic data structure, no false negatives, possible false positives
- Negative caching: simple but requires short TTL to avoid masking newly created entries
- Input validation: reject obviously invalid IDs before they reach the cache layer

### Hints (Progressive)
1. "Why doesn't the system cache the fact that the user doesn't exist?"
2. "You can cache negative results. What does that look like?"
3. "Cache the key `user:999999` with a value of `NULL` and a short TTL (e.g., 30 seconds)."
4. Full Bloom Filter solution above.

---

## Problem 2: Cache Stampede (Thundering Herd)

**Question**: "A highly popular item's cache key expires. 5,000 concurrent requests hit the API, all get a cache miss, and all query the database simultaneously. The DB crashes. How do we prevent this?"

### Walkthrough

**Root Cause**: TTL expiration on a hot key causes all concurrent readers to simultaneously fall through to the database.

**Ideal Answer**:
Implement a Mutex/Lock. When a cache miss occurs, the thread attempts to acquire a Redis lock for `lock:item_123`. The thread that gets the lock queries the DB and updates the cache. The other threads wait 50ms and check the cache again.

**Alternative**: Use "Probabilistic Early Expiration" where threads randomly decide to refresh the cache *before* it actually expires.

**Key Concepts**:
- Distributed locking with Redis SETNX
- Stale-while-revalidate pattern
- Probabilistic early expiration
- Background refresh workers for critical hot keys

### Hints (Progressive)
1. "We only need *one* request to query the database and update the cache. The other 4,999 requests should wait or get alternative data."
2. "How do we elect a 'leader' among those 5,000 threads to do the DB query?"
3. "Use a distributed lock (e.g., Redis SETNX)."
4. Full solution above.

---

## Problem 3: In-Memory vs Distributed Cache

**Question**: "Should we use an in-memory cache (like a ConcurrentHashMap in our Java app) or a distributed cache (like Redis)?"

### Walkthrough

**Ideal Answer**:
Use a multi-level cache (L1/L2). Put a small, fast in-memory cache (L1) in the app with a very short TTL (e.g., 5 seconds) to handle massive spikes. Fall back to a larger Distributed Cache (L2, Redis) for global consistency, then fall back to the DB.

**Key Concepts**:
- In-memory: nanosecond latency, lost on deploy, duplicated per server
- Distributed: millisecond latency, shared across servers, survives deploys
- L1/L2 strategy: best of both worlds
- Consistency challenges with multi-level caches

### Hints (Progressive)
1. "What happens to the in-memory cache when you deploy a new version of the app?"
2. "If you have 10 application servers, an in-memory cache means the DB gets queried 10 times for the same data (once per server)."
3. "In-memory is much faster (nanoseconds vs milliseconds) but hard to keep consistent."
4. Full solution above.

---

## Problem 4: Cache Consistency Race Condition

**Question**: "Two threads are updating the same user's profile simultaneously. Thread A updates the DB to V2 and then sets the cache to V2. Thread B reads the DB (gets V1 due to timing) and sets the cache to V1 after Thread A. Now the cache holds stale data. How do you prevent this?"

### Walkthrough

**Ideal Answer**:
Always DELETE the cache key on writes, never SET. Let the next reader populate it via Cache-Aside. This eliminates the race condition because the worst case is an extra cache miss, not stale data.

**Key Concepts**:
- DELETE vs SET on write: DELETE is always safe
- Cache-Aside pattern naturally handles this
- Write-Through can also handle this if properly serialized
- Version numbers or CAS (Compare-And-Swap) for advanced scenarios

---

## Sample Interview Session

### Opening
"Hey, glad to have you here! Let's jump right in. We have a slow API endpoint that aggregates user stats -- things like total posts, follower count, and engagement metrics. Response times are around 2 seconds. How would you speed this up?"

### Follow-up Probes
- "Where should the cache live? In the app's memory or in Redis?"
- "What happens when a user gains a new follower? How do we keep the cached stats accurate?"
- "Our Redis cluster just went down. What happens to our application?"
- "A celebrity with 10 million followers just posted. Walk me through what happens when their profile cache key expires."

### Wrap-up
Generate scorecard based on the Evaluation Rubric. Highlight strengths, improvement areas, and recommended resources.
