# Rate Limiter — Problem Bank, Walkthroughs & Calculations

---

## 📝 Complete System Design Walkthrough

### 1. Requirements & Back-of-the-Envelope
- **Scale**: 1M RPM (Requests Per Minute) -> ~16,000 RPS.
- **Rules**: Multiple tiers (Free: 10/min, Pro: 1000/min).
- **Latency**: Must add < 5ms to request path.

### 2. Algorithm Choice (Token Bucket)
- Memory per user: ~20 bytes (Hash: `{ "tokens": 5, "last_refill": 16788822 }`).
- 10 Million active users = 200MB memory in Redis. Highly efficient.

### 3. Architecture
- **Client** -> **Load Balancer** -> **API Gateway** (Runs Rate Limiter) -> **Backend Services**.
- API Gateways share state via a **Redis Cluster**.

### 4. Detailed Flow
1. Request arrives at API Gateway.
2. Gateway identifies user (API Key, IP, UserID).
3. Gateway executes Lua script on Redis:
   - Calculate time since `last_refill`.
   - Add new tokens (up to max capacity).
   - If tokens > 0, decrement and return ALLOW (along with `X-RateLimit-Remaining` headers).
   - If tokens == 0, return DENY.
4. If DENY, Gateway returns HTTP 429. If ALLOW, proxy to Backend.

### 5. Scaling & Resilience
- **Fail-open vs Fail-closed**: If Redis dies, Gateway should fail-open (allow requests) so the business doesn't go down, relying on auto-scaling backends to absorb the load.
- **Sharding**: Shard Redis by UserID/IP using consistent hashing.
