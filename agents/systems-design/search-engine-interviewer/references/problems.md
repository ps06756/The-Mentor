# Search Engine -- Problem Bank, Walkthroughs & Calculations

---

## Complete System Design Walkthrough

### 1. Requirements & Back-of-the-Envelope

- **Corpus Size**: 10 billion web pages (roughly the indexable web).
- **Average Page Size**: 50KB raw HTML, ~5KB extracted text.
- **Raw Storage**: 10B * 50KB = 500TB (compressed ~100TB).
- **Index Size**: Inverted index is typically 20-30% of raw text. 10B * 5KB * 0.25 = ~12.5TB uncompressed, ~3TB compressed.
- **Crawl Rate**: To re-crawl the entire web every 30 days: 10B / 30 / 86400 = ~3,800 pages/sec. To crawl 1B pages/day: ~11,500 pages/sec.
- **Query Throughput**: 100,000 QPS (Google scale). Start with 10,000 QPS for interview scope.
- **Query Latency**: p50 < 100ms, p99 < 500ms.

### 2. Web Crawling Architecture

- **URL Frontier**: Priority queue (by PageRank or freshness score) + per-domain politeness queues (FIFO with rate limit).
- **Crawler Nodes**: Stateless workers that pull URLs from the Frontier, fetch pages, parse HTML, extract links and text.
- **Politeness**: Respect robots.txt (cache per domain, refresh daily). Min-interval between requests to same domain (1-2 seconds).
- **Deduplication**: URL normalization + Bloom filter (seen URLs). Content dedup via SimHash or MinHash (near-duplicate detection).
- **Freshness**: Track Last-Modified / ETag headers. Estimate change frequency per page. Re-crawl high-change pages more often.
- **DNS Resolution**: Local DNS cache to avoid bottlenecking on DNS lookups (each crawl = 1 DNS lookup otherwise).

### 3. Indexing Pipeline

- **Document Processing**: Strip HTML tags, extract text, title, headings, anchor text from inbound links.
- **Tokenization**: Lowercase, split on whitespace/punctuation, stem (Porter stemmer), remove stop words (optional -- modern engines keep them).
- **Inverted Index Construction**: MapReduce or streaming pipeline.
  - Map: (term, doc_id, term_frequency, positions) for each term in each document.
  - Reduce: Group by term, sort posting list by doc_id, compress.
- **Posting List Format**: `term -> [(doc_id, tf, [pos1, pos2, ...]), ...]` sorted by doc_id.
- **Compression**: Delta-encode doc_ids (sorted, so deltas are small). Variable-byte or PForDelta encoding. Typically 4:1 compression.
- **Skip Pointers**: Every sqrt(N) entries, store a skip pointer to enable O(sqrt(N)) intersection instead of O(N).
- **Sharding**: Document-partitioned (each shard has a complete index for a subset of documents). Alternative: term-partitioned (each shard owns certain terms), but this complicates multi-term queries.

### 4. Ranking Pipeline

**Stage L0 -- Index-Time Scoring**:
- BM25(query, document) computed during posting list traversal.
- Fast, covers all matching documents.

**Stage L1 -- Lightweight Re-ranking**:
- Applied to top ~1000 candidates from L0.
- Features: BM25 score, PageRank, freshness, domain authority, URL depth.
- Model: Linear model or small gradient-boosted tree.

**Stage L2 -- Heavy ML Re-ranking**:
- Applied to top ~50 candidates from L1.
- Features: BERT-based query-document relevance score, click-through rate prediction, user context.
- Model: Deep neural network (transformer-based).

### 5. Serving Architecture

- **Index Servers**: Shard the index across N machines. Each holds a complete inverted index for its document partition.
- **Query Flow**: Query arrives at Query Service -> Parse & understand -> Scatter to all N index shards -> Each shard returns top-K -> Gather & merge -> L1 re-rank -> L2 re-rank -> Return results.
- **Replication**: Each shard replicated 3x for availability and load balancing.
- **Caching**: Cache popular query results (top 1% of queries = ~30% of traffic). Cache at multiple levels: CDN, query result cache, posting list cache.

---

## Problem 1: Web Crawling at Scale

**Difficulty**: Hard

**Problem Statement**: Design a distributed web crawler that can crawl 1 billion pages per day, respecting robots.txt and politeness constraints, while avoiding duplicate content.

**Architecture Walkthrough**:

```
Seed URLs -> URL Frontier (Priority + Politeness Queues)
                  |
                  v
         Crawler Workers (1000s of nodes)
           |          |           |
           v          v           v
        Fetch      Parse     Extract Links
           |          |           |
           v          v           v
     robots.txt   HTML->Text   URL Normalize
     check        extraction   + Bloom Filter
           |          |           |
           v          v           v
     Rate Limiter  Doc Store   New URLs -> Frontier
                   SimHash
                   (content dedup)
```

**Key Decisions**:
- URL Frontier partitioned by domain (consistent hashing). Each crawler node owns a set of domains.
- Bloom filter with 10B entries, 1% false positive rate = ~1.2GB memory. Acceptable.
- SimHash for content dedup: compute 64-bit fingerprint, store in lookup table, flag pages with Hamming distance < 3 as duplicates.
- Crawler nodes are stateless. They pull from the Frontier, fetch, parse, and push results. Easy to scale horizontally.

**Capacity**:
- 1B pages/day = ~11,500 pages/sec.
- Average page fetch: 200ms (network) + 50ms (parse) = 250ms per page.
- With 1000 crawler nodes, each handling 100 concurrent connections: 100,000 pages/sec capacity. Comfortable headroom.

---

## Problem 2: Inverted Index Design

**Difficulty**: Hard

**Problem Statement**: Design the inverted index that supports multi-term boolean and ranked queries across 10 billion documents with sub-100ms latency.

**Architecture Walkthrough**:

```
Build Path:
  Documents -> Tokenizer -> (term, doc_id, tf, positions)
                         -> MapReduce / Stream Processor
                         -> Sorted Posting Lists
                         -> Compressed + Skip Pointers
                         -> Distributed to Index Shards

Query Path:
  Query: "distributed systems"
    -> Tokenize: ["distributed", "systems"]
    -> Scatter to all shards
    -> Each shard:
         Fetch posting list for "distributed"  [D1, D5, D12, D44, ...]
         Fetch posting list for "systems"      [D2, D5, D13, D44, ...]
         Intersect using merge + skip pointers -> [D5, D44, ...]
         Score each result (BM25)
         Return top-K
    -> Gather, merge top-K from all shards, global re-rank
```

**Key Decisions**:
- Document-partitioned sharding: 10B docs / 10,000 shards = 1M docs per shard. Index per shard ~300MB. Fits in memory.
- Posting list compression: delta encoding reduces average integer from 32 bits to ~8 bits. Variable-byte coding. 4:1 compression ratio.
- Skip pointers: for a posting list of length N, place skip pointers every sqrt(N) entries. Intersection of two lists of length M and N: O(M * sqrt(N)) instead of O(M + N) when M << N.
- Phrase queries: use positional index. Check that positions are consecutive after intersection.

**Failure Modes**:
- Shard failure: replicas serve queries. Rebuild from document store.
- Index corruption: checksum per posting list segment. Rebuild affected segment.
- Hot terms ("the", "is"): these posting lists are huge but often pruned by combining with rarer terms first. For single common-term queries, use cached results.

---

## Problem 3: Query Autocomplete

**Difficulty**: Medium-Hard

**Problem Statement**: Design the autocomplete system that suggests the top 10 most relevant queries as the user types each character, with sub-50ms latency.

**Architecture Walkthrough**:

```
Offline Pipeline:
  Query Logs -> Aggregate (count per query, 7-day window)
             -> Filter (remove offensive, low-freq)
             -> Build Trie (top-10 per node precomputed)
             -> Serialize & distribute to serving nodes

Online Serving:
  User types "dist" -> Trie lookup at node d->i->s->t
                    -> Return precomputed top-10:
                       ["distributed systems", "distance calculator",
                        "district court", ...]
                    -> Latency: single hash lookup + memory read < 5ms
```

**Key Decisions**:
- Precompute top-K at each trie node (avoids subtree traversal at query time).
- Trie built from top 50M queries. Total nodes ~500M. At ~100 bytes per node (character + children pointers + top-10 list) = ~50GB. Shard across machines or use memory-mapped files.
- Shard by prefix range: "a*" on shard 1, "b*" on shard 2, etc. Single-shard lookup per query.
- Freshness: Rebuild trie every 4-6 hours from updated query logs. Blue-green deployment (serve old trie while building new one).
- Personalization: Blend global suggestions with user's recent queries (client-side cache of recent queries, boost matching ones).

**Capacity**:
- 10,000 QPS * ~4 keystrokes per query = 40,000 autocomplete requests/sec.
- Each request is a single trie node lookup: O(1) after traversing prefix. Sub-5ms.
- CDN caching for top 1-2 character prefixes (26 * 26 = 676 entries) handles most traffic.

---

## Problem 4: Spell Correction

**Difficulty**: Medium-Hard

**Problem Statement**: Design a spell correction system that fixes typos in search queries (e.g., "restarants" -> "restaurants") with high accuracy and low latency.

**Architecture Walkthrough**:

```
Query: "restarants near me"
  -> Tokenize: ["restarants", "near", "me"]
  -> For each token, check if it exists in the dictionary
  -> "restarants" NOT in dictionary
  -> Generate candidates within edit distance 1-2:
       Edit distance 1: "restaurants", "restaraunts", ...
       Edit distance 2: (many more candidates)
  -> Filter candidates: keep only those in dictionary
  -> Rank candidates by:
       1. Edit distance (prefer distance 1)
       2. Language model probability P("restaurants near me") vs P("restarants near me")
       3. Query log frequency (how often was "restaurants" searched?)
  -> Best correction: "restaurants"
  -> Corrected query: "restaurants near me"
```

**Key Decisions**:
- Dictionary: top 500K terms from the index + top 10M queries from query logs.
- Candidate generation: precompute all terms within edit distance 2 for each dictionary term (SymSpell approach). Store in hash map. O(1) lookup at query time.
- Language model: bigram/trigram model trained on query logs. P(w_i | w_{i-1}) helps disambiguate ("ice cream" not "ice cram").
- Confidence threshold: only correct if the corrected query is significantly more likely than the original. Avoid over-correction.
- "Did you mean?" vs auto-correct: auto-correct for high-confidence corrections, suggest for lower confidence.

**Capacity**:
- Dictionary lookup: O(1) hash map lookup. Sub-1ms.
- Language model scoring: precomputed bigram table. Sub-1ms.
- Total spell correction overhead: < 5ms per query. Negligible compared to index lookup.
