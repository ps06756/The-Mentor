# URL Shortener — Problem Bank, Walkthroughs & Calculations

---

## 📝 Complete System Design Walkthrough

### Step 1: Requirements

**Functional Requirements**:
1. Create short URL from long URL
2. Redirect short URL to original URL
3. Custom short URLs (optional)
4. URL expiration (optional)
5. Analytics (optional)

**Non-Functional Requirements**:
1. High availability (99.99% uptime)
2. Low latency (<100ms for redirect, <500ms for create)
3. Scale: 100M new URLs/month, 10B redirects/month
4. Short URLs should be unpredictable (security)

**Back-of-Envelope Calculations**:
```
New URLs: 100M/month = ~40/second (peak ~400/second)
Redirects: 10B/month = ~4000/second (peak ~40,000/second)

Storage:
- Each URL record: ~500 bytes (short_code, long_url, metadata)
- 100M/month × 500 bytes = 50GB/month
- 5 years = 3TB

Bandwidth:
- Create: 400 req/s × 1KB = 400KB/s incoming
- Redirect: 40K req/s × 1KB = 40MB/s outgoing
```

---

### Step 2: API Design

```
POST /api/v1/urls
Request:
{
  "long_url": "https://example.com/very/long/path",
  "custom_alias": "mylink",      // optional
  "expiration_days": 30          // optional
}

Response: 201 Created
{
  "short_code": "a3f5b2c",
  "short_url": "https://short.io/a3f5b2c",
  "long_url": "https://example.com/very/long/path",
  "created_at": "2024-01-15T10:30:00Z",
  "expires_at": "2024-02-14T10:30:00Z"
}

GET /{short_code}
Response: 302 Redirect to long_url
        or 404 if not found
        or 410 if expired

GET /api/v1/urls/{short_code}/stats
Response:
{
  "short_code": "a3f5b2c",
  "clicks": 15420,
  "unique_visitors": 12300,
  "referrers": {...},
  "countries": {...}
}
```

---

### Step 3: Database Design

**Schema (SQL - MySQL/PostgreSQL)**:
```sql
CREATE TABLE urls (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  short_code VARCHAR(10) UNIQUE NOT NULL,
  long_url VARCHAR(2048) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  expires_at TIMESTAMP NULL,
  user_id INT,
  click_count BIGINT DEFAULT 0,

  INDEX idx_short_code (short_code),
  INDEX idx_expires_at (expires_at)
);

-- For analytics
CREATE TABLE url_clicks (
  id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
  short_code VARCHAR(10) NOT NULL,
  clicked_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  ip_address VARBINARY(16),
  user_agent VARCHAR(512),
  referrer VARCHAR(2048),
  country_code CHAR(2),

  INDEX idx_short_code_time (short_code, clicked_at)
);
```

**NoSQL Alternative (DynamoDB/Cassandra)**:
```javascript
// URLs table
{
  short_code: "a3f5b2c",      // Partition Key
  long_url: "https://...",
  created_at: 1705317000,
  expires_at: 1707909000,
  click_count: 15420
}

// Time-series for analytics (separate table)
{
  short_code: "a3f5b2c",
  hour_bucket: "2024-01-15-10",  // Partition Key
  click_count: 150,
  unique_visitors: 120
}
```

---

### Step 4: URL Generation Deep Dive

**Base62 Encoding Approach (Recommended)**:

```python
import string

BASE62 = string.ascii_letters + string.digits  # a-zA-Z0-9

def encode_base62(num):
    """Convert integer to base62 string"""
    if num == 0:
        return BASE62[0]

    result = []
    while num > 0:
        num, remainder = divmod(num, 62)
        result.append(BASE62[remainder])

    return ''.join(reversed(result))

def decode_base62(s):
    """Convert base62 string back to integer"""
    result = 0
    for char in s:
        result = result * 62 + BASE62.index(char)
    return result

# Usage
counter = 125000000  # From distributed counter
short_code = encode_base62(counter)  # "8H9jK2"
```

**Distributed Counter Options**:
1. **ZooKeeper**: Sequential nodes, reliable but slower
2. **Redis**: INCR command, fast but need persistence
3. **Database**: Auto-increment, simplest but single point of contention
4. **Pre-allocated Ranges**: Each app server gets a range (e.g., Server A: 1-1M, Server B: 1M-2M)

---

### Step 5: Caching Strategy

**Multi-Layer Caching**:

```
User Request
    │
    ▼
Browser Cache (Cache-Control: max-age=3600)
    │
    ▼
CDN Edge Cache (CloudFlare, etc.)
    │
    ▼
Application Cache (Redis Cluster)
    ├─ L1: Hot URLs (LRU, 99% hit rate for top 1%)
    ├─ L2: Recent URLs (TTL 1 hour)
    └─ L3: Bloom Filter (avoid DB lookups for non-existent codes)
    │
    ▼
Database (MySQL Primary-Replica)
```

**Cache Warming**:
- Pre-populate cache for new URLs
- Async job to refresh expiring cache entries

---

### Step 6: Scaling Deep Dive

**Horizontal Scaling**:
```
┌─────────────────────────────────────────┐
│           Load Balancer (HAProxy)       │
│    Health checks, least-connections     │
└───────────────────┬─────────────────────┘
                    │
    ┌───────────────┼───────────────┐
    ▼               ▼               ▼
┌───────┐      ┌───────┐      ┌───────┐
│ App 1 │      │ App 2 │      │ App 3 │
└───┬───┘      └───┬───┘      └───┬───┘
    │               │               │
    └───────────────┼───────────────┘
                    ▼
            ┌─────────────┐
            │ Redis Cluster│
            │ (6 nodes)    │
            └─────────────┘
                    │
            ┌───────┴───────┐
            ▼               ▼
    ┌─────────────┐  ┌─────────────┐
    │ MySQL Primary│  │ MySQL Replica│
    │ (writes)     │  │ (reads × 3)  │
    └─────────────┘  └─────────────┘
```

**Sharding Strategy**:
```
Consistent Hashing Ring (0 to 2^32-1)

Server A handles hash range: 0 - 1,431,655,765
Server B handles hash range: 1,431,655,766 - 2,863,311,530
Server C handles hash range: 2,863,311,531 - 4,294,967,295

hash("a3f5b2c") = 1,234,567,890 → Server A
hash("x9y2z1w") = 3,500,000,000 → Server C

Adding Server D: Only 1/4 of keys need to move (those near D's position)
```

---

## 🎬 Sample Session Flow

**You**: "Let's design a URL shortener like bit.ly. Before we dive into the solution, tell me what questions you have about the requirements."

**Candidate**: "How many URLs are we creating per day? How many redirects?"

**You**: "Good questions. Let's say 100M new URLs per month and 10B redirects per month. What else?"

**Candidate**: "Do we need custom short URLs? Analytics?"

**You**: "Great - custom URLs are a nice-to-have, analytics too. Let's start with the core flow. Walk me through what happens when a user submits a long URL."

[Continue with API design, then data model, then scaling...]
