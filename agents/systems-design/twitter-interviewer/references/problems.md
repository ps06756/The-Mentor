# Twitter/Social Media Feed -- Problem Bank, Walkthroughs & Calculations

---

## Complete System Design Walkthrough

### 1. Requirements & Back-of-the-Envelope

- **Scale**: 500M monthly active users, 200M daily active users.
- **Write Throughput**: 500M tweets/day = ~6,000 tweets/sec average, ~60,000 tweets/sec peak.
- **Read Throughput**: 200M DAU * ~20 timeline refreshes/day = 4B timeline reads/day = ~46,000 reads/sec average.
- **Fan-out Write Amplification**: Average user has ~200 followers. 6,000 tweets/sec * 200 = 1.2M cache writes/sec for fan-out.
- **Storage**: Average tweet ~300 bytes (text + metadata). 500M tweets/day * 300B = 150GB/day = ~55TB/year (text only, excluding media).
- **Latency**: Timeline load < 200ms. Tweet post acknowledgment < 500ms (fan-out can be async).

### 2. Tweet Storage Service

- **Primary Store**: Sharded by tweet_id (Snowflake ID: timestamp + machine + sequence).
- **Schema**: tweet_id (PK), user_id, content, media_urls[], created_at, reply_to, retweet_of, like_count, retweet_count.
- **Database**: Wide-column store (Cassandra or ScyllaDB) for high write throughput.
- **Media**: Store in object storage (S3), reference URLs in tweet record. Serve via CDN.
- **Indexing**: Secondary index on user_id for "user's tweets" page. Time-based partitioning for efficient range scans.

### 3. Social Graph Service

- **Data Model**: Adjacency list. Two tables: `following(user_id, followee_id, created_at)` and `followers(user_id, follower_id, created_at)`.
- **Storage**: Graph database (e.g., TAO at Facebook) or sharded MySQL/Vitess (Twitter's actual approach).
- **Sharding**: Shard by user_id. A "follow" operation writes to both the follower's shard and the followee's shard.
- **Caching**: Hot follower lists cached in Redis. Celebrity follower lists are too large to cache entirely -- paginate.
- **Scale**: Average user follows ~400 accounts. 500M users * 400 = 200B edges.

### 4. Timeline Generation (Hybrid Fan-out)

**Write Path (Normal Users)**:
1. User A posts tweet T.
2. Tweet Service persists T and publishes event to Kafka.
3. Fan-out Service consumes event, queries Social Graph for A's followers.
4. For each follower, LPUSH tweet_id to their Redis timeline list (capped at 800).
5. If follower count > celebrity threshold (e.g., 500K), skip fan-out for this tweet.

**Read Path**:
1. User B opens timeline.
2. Timeline API reads B's Redis timeline list (precomputed portion).
3. Timeline API checks if B follows any celebrities. If so, fetches their recent tweets from the Celebrity Tweet Cache.
4. Merge, deduplicate (retweets), and rank.
5. Return top N tweets with pagination cursor.

**Ranking**:
- Initial sort by timestamp (chronological baseline).
- Score adjustment: affinity (how often B interacts with A), engagement (likes + retweets in first hour), recency decay.
- For ML-ranked feeds: feature vector per tweet, lightweight model inference at read time, heavier model for pre-scoring in batch.

---

## Problem 1: Home Timeline at Scale

**Difficulty**: Hard

**Problem Statement**: Design the system that generates a user's home timeline. The system must handle 200M DAU each loading their timeline ~20 times per day, with sub-200ms p99 latency.

**Architecture Walkthrough**:

```
Post Path:
  User -> Tweet Service -> Kafka -> Fan-out Service -> Redis (per-user lists)
                                                    -> Celebrity Tweet Cache

Read Path:
  User -> Timeline API -> Redis (precomputed list)
                       -> Celebrity Tweet Cache (merge)
                       -> Ranking Service (score & sort)
                       -> Response
```

**Key Decisions**:
- Fan-out on write for users with < 500K followers (covers 99.9% of users).
- Fan-out on read for celebrity tweets (merging a handful of celebrity feeds is fast).
- Redis list per user, capped at 800 tweet IDs. Full tweet objects fetched in batch from Tweet Cache (Memcached).
- Pagination via cursor (last seen tweet_id + timestamp).

**Failure Modes**:
- Redis node failure: Rebuild timeline from Social Graph + Tweet Store (expensive but rare). Use Redis Cluster with replicas.
- Fan-out lag: During spikes, fan-out queue depth grows. Users see slightly stale timelines. Acceptable trade-off.

---

## Problem 2: Trending Topics Detection

**Difficulty**: Hard

**Problem Statement**: Design a system that identifies trending topics in real time from the global tweet stream, with regional granularity (trending in NYC vs trending globally).

**Architecture Walkthrough**:

```
Tweet Stream -> Kafka -> Flink/Kafka Streams -> Sliding Window Counters
                                              -> Acceleration Scoring
                                              -> Spam Filtering
                                              -> Regional Aggregation
                                              -> Redis (top-N per region)
                                              -> API
```

**Key Decisions**:
- Extract entities: hashtags, mentioned topics (NLP), URLs.
- Maintain two counters per topic per region: short window (5 min) and long window (24 hr).
- Acceleration = short_count / (long_count / 288). Threshold > 5x means trending.
- Use Count-Min Sketch for memory-efficient approximate counting of the long tail.
- Top-K maintained with a min-heap per region.
- Cache results in Redis with 60-second TTL. Trending topics do not need to update faster than that.

**Capacity**:
- 6,000 tweets/sec, each with ~1.5 entities on average = 9,000 entity events/sec.
- Counter storage: 1M unique entities * 2 windows * ~50 regions = 100M counters. At 16 bytes each = ~1.6GB. Fits in memory.

---

## Problem 3: Celebrity Fan-out Problem

**Difficulty**: Hard

**Problem Statement**: A user with 50 million followers posts a tweet. Design the system to deliver this tweet to all followers' timelines without causing a cascade failure.

**Architecture Walkthrough**:

```
Celebrity tweets -> Tweet Service -> Tweet Store
                                  -> Celebrity Tweet Cache (per-celebrity sorted set in Redis)
                                  -> (NO fan-out to individual timelines)

When follower reads timeline:
  Timeline API -> Precomputed timeline (Redis list)
               -> For each celebrity followed: fetch from Celebrity Tweet Cache
               -> Merge & rank
               -> Return
```

**Key Decisions**:
- Celebrity threshold: dynamically computed. Start with 500K followers. Adjust based on fan-out queue depth.
- Celebrity Tweet Cache: Redis sorted set per celebrity, scored by timestamp. Keep last 200 tweets.
- At read time, average user follows ~5 celebrities. 5 Redis ZRANGEBYSCORE calls add ~5ms. Acceptable.
- Pre-warm: For users who follow many celebrities, periodically pre-merge in a background job.

**Trade-offs**:
- Shifts cost from write amplification (50M writes) to read amplification (extra lookups per timeline read).
- Read amplification is bounded and parallelizable. Write amplification for celebrities is unbounded.
- Edge case: A user who follows 1000 celebrities. Solution: cap celebrity merges at N, show "based on your interests" for the rest, or pre-merge periodically.

---

## Problem 4: Tweet Search

**Difficulty**: Medium-Hard

**Problem Statement**: Design a system that allows users to search for tweets by keyword with sub-500ms latency.

**Architecture Walkthrough**:

```
Indexing Path:
  Tweet Service -> Kafka -> Search Indexer -> Elasticsearch / Lucene Cluster

Query Path:
  User -> Search API -> Query Parser -> Elasticsearch (scatter-gather across shards)
                                      -> Rank by relevance + recency + engagement
                                      -> Return top results
```

**Key Decisions**:
- Use Elasticsearch (or a custom Lucene-based system like Twitter's EarlyBird).
- Index partitioning: time-based (recent tweets in hot shards, older tweets in cold shards). Most searches care about recent tweets.
- Inverted index on tokenized tweet content.
- Real-time indexing: tweets searchable within seconds of posting (near-real-time Lucene segment refresh).
- Ranking: BM25 for text relevance, boosted by user verification status, engagement metrics, and recency.
- Typeahead/autocomplete: separate trie-based service for query suggestions.

**Capacity**:
- Index size: 300B avg tweet * 500M tweets/day * 7 days hot = ~1TB hot index. Sharded across cluster.
- Query throughput: ~10,000 searches/sec. Scatter-gather across 20 shards = 200K shard queries/sec total.
