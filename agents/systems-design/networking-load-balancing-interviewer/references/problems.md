# Problem Bank: Networking & Load Balancing

## Problem 1: The Life of a Request — Full Network Trace

### Problem Statement
A user in New York types `https://www.example.com/products` into their browser. Trace the complete network journey from keystroke to rendered page. Include every protocol layer involved.

### Solution Walkthrough

**Step 1: DNS Resolution**
1. Browser checks local DNS cache
2. OS checks `/etc/hosts` and local resolver cache
3. Query goes to configured DNS resolver (e.g., ISP's DNS or 8.8.8.8)
4. Recursive resolution:
   - Root DNS server (`.`) -> returns NS for `.com`
   - `.com` TLD server -> returns NS for `example.com`
   - `example.com` authoritative server -> returns A record: `93.184.216.34`
5. Response cached with TTL (e.g., 300 seconds)

**Step 2: TCP 3-Way Handshake**
```
Client (NYC)                    Server (93.184.216.34)
  | ---- SYN (Seq=1000) ------> |    RTT ~30ms (NYC to server)
  | <--- SYN-ACK (Seq=5000, Ack=1001) -- |
  | ---- ACK (Ack=5001) ------> |
```
Total: ~1.5 RTT = ~45ms

**Step 3: TLS 1.3 Handshake**
```
Client                          Server
  | ---- ClientHello ------------> |    (TLS version, cipher suites, key share)
  | <--- ServerHello, Cert, Finished -- |  (server cert, key share, encrypted)
  | ---- Finished ----------------> |
```
TLS 1.3: 1 RTT (~30ms). TLS 1.2 would be 2 RTTs.

**Step 4: HTTP Request**
```
GET /products HTTP/1.1
Host: www.example.com
Accept: text/html
Cookie: session=abc123
```

**Step 5: Server Processing**
- Request hits Load Balancer (L7)
- LB terminates TLS, reads HTTP path `/products`
- Routes to appropriate backend server
- Backend queries database, builds HTML
- Response sent back through LB

**Step 6: HTTP Response**
```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 45230
```

**Total Time Breakdown:**
| Phase | Time |
|-------|------|
| DNS lookup (cache miss) | ~100ms |
| TCP handshake | ~45ms |
| TLS handshake (1.3) | ~30ms |
| HTTP request + server processing | ~200ms |
| **Total (first byte)** | **~375ms** |

### Follow-up Questions
- "How does HTTP/2 change this?" (Multiplexing, header compression, single TCP connection)
- "How does HTTP/3/QUIC change this?" (UDP-based, 0-RTT resumption, no head-of-line blocking)
- "What if we used a CDN?" (Edge server closer to user, cached content, reduced latency)

---

## Problem 2: Designing a Global Load Balancing Architecture

### Problem Statement
You are building a global web application serving users from North America, Europe, and Asia. You need:
- Sub-100ms latency for all users
- 99.99% availability
- Ability to handle 1M requests/second globally

Design the load balancing architecture from edge to backend.

### Solution Walkthrough

**Layer 1: Global DNS Load Balancing**
- Use GeoDNS (e.g., Route 53 latency-based routing)
- US users resolve to US edge IPs
- EU users resolve to EU edge IPs
- Asia users resolve to Asia edge IPs

**Layer 2: Anycast + CDN Edge**
- Deploy edge PoPs (Points of Presence) in each region
- Use Anycast: same IP advertised via BGP from all PoPs
- Traffic automatically routes to nearest PoP
- CDN handles static content at the edge

**Layer 3: Regional L4 Load Balancers**
- Each region has L4 LBs (e.g., AWS NLB, or IPVS/LVS)
- L4 for speed — just forward TCP packets
- Active-Active pair with Keepalived for HA

**Layer 4: Application L7 Load Balancers**
- Behind L4, deploy L7 LBs (Nginx, Envoy)
- TLS termination happens here
- Route by URL path: `/api/*` -> API servers, `/static/*` -> CDN origin
- Rate limiting, authentication at this layer

**Layer 5: Backend Server Pool**
- Multiple backend servers per region
- Algorithm: Least Connections (for even load distribution)
- Health checks every 5 seconds; remove unhealthy servers

**Architecture Diagram:**
```
Users (Global)
      |
  [ GeoDNS / Anycast ]
      |
  +---+---+---+
  |   |   |   |
 US  EU  Asia  (Edge PoPs)
  |   |   |
 [L4 LB Pair] (Keepalived, Active-Active)
  |
 [L7 LB Pool] (Nginx/Envoy, TLS termination)
  |
 [Backend Servers] (Least Connections)
```

### Back-of-Envelope Calculation

**Traffic Distribution:**
- 1M req/s globally
- US: 400K req/s, EU: 350K req/s, Asia: 250K req/s

**L4 LB Capacity (per region, US example):**
- Modern L4 LB (IPVS/LVS): handles 1M+ packets/sec easily
- 2 LBs in Active-Active = 200K req/s each = well within capacity

**L7 LB Capacity (per region, US example):**
- Nginx: ~50K req/s per instance (with TLS termination)
- Need: 400K / 50K = 8 Nginx instances minimum
- With 2x headroom: 16 Nginx instances

**Backend Servers (per region, US example):**
- Each backend: ~2K req/s (CPU-bound application logic)
- Need: 400K / 2K = 200 servers
- With headroom: 250 servers

---

## Problem 3: Consistent Hashing for Cache Layer

### Problem Statement
You have a distributed cache layer (like Memcached) with 10 cache servers. Requests are mapped to servers using `hash(key) % N`. When you add an 11th server, almost all cached data is invalidated. Design a better approach.

### Solution Walkthrough

**The Problem with Modulo Hashing:**
- With N=10: `hash("user:123") % 10 = 7` -> Server 7
- With N=11: `hash("user:123") % 11 = 2` -> Server 2
- ~90% of keys map to different servers = massive cache miss storm

**Consistent Hashing Solution:**

1. **Hash Ring**: Place servers on a ring (0 to 2^32)
   - Server A at position hash("ServerA") = 1000
   - Server B at position hash("ServerB") = 5000
   - Server C at position hash("ServerC") = 9000

2. **Key Mapping**: Hash the key, walk clockwise to find the next server
   - hash("user:123") = 3000 -> walks clockwise -> Server B (at 5000)

3. **Adding a Server**: Only keys between the new server and its predecessor are remapped
   - Add Server D at position 7000
   - Only keys between 5001-7000 move from Server C to Server D
   - ~1/N keys remapped instead of ~(N-1)/N

4. **Virtual Nodes**: Each physical server gets multiple positions on the ring
   - Server A -> hash("ServerA-0"), hash("ServerA-1"), ..., hash("ServerA-149")
   - 150 virtual nodes per server ensures even distribution
   - Without virtual nodes, distribution can be very uneven

**Impact Analysis:**
| Metric | Modulo Hash | Consistent Hash |
|--------|-------------|-----------------|
| Keys remapped (add 1 server to 10) | ~90% | ~10% |
| Keys remapped (remove 1 from 10) | ~90% | ~10% |
| Distribution evenness | Good | Requires virtual nodes |
| Implementation complexity | Simple | Moderate |

### Back-of-Envelope: Cache Miss Storm
- 10 cache servers, 100GB total cached data
- Adding 1 server with modulo: 90GB invalidated
- At 10K req/s with 95% cache hit rate:
  - Normal: 500 req/s hit database
  - After modulo rehash: 9,500 req/s hit database = 19x increase = likely DB crash
- With consistent hashing: 10GB invalidated
  - After rehash: ~1,450 req/s hit database = 2.9x increase = manageable

---

## Sample Session Flow

### Opening (2 minutes)
**Interviewer**: "Hi there! I'm the infrastructure lead. Today we're going to talk about how traffic moves through a system. I care a lot about the fundamentals — no hand-waving allowed. Let's start from the very beginning."

### Phase 1: Network Fundamentals (10 minutes)
**Interviewer**: "A user types `www.example.com` into their browser and hits Enter. Walk me through every network step until the first byte of HTML arrives. Be specific about protocols."

*[Candidate walks through DNS, TCP, TLS, HTTP]*

**Interviewer**: "Good. You mentioned TLS. Where exactly does TLS termination happen in a typical production setup? And what's the performance implication?"

### Phase 2: Load Balancing (15 minutes)
**Interviewer**: "Now we have 100 backend servers. We need a load balancer. Should we use Layer 4 or Layer 7? What's the actual difference in how they handle packets?"

*[Candidate discusses L4 vs L7]*

**Interviewer**: "You chose L7. Our marketing team wants `/blog` to go to WordPress servers and `/api` to go to Node.js servers. Can we do that at L4?"

### Phase 3: Algorithms & State (10 minutes)
**Interviewer**: "We're building a chat app with WebSockets. Once a user connects, they need to stay on the same server. How do we handle this with a load balancer?"

### Phase 4: High Availability (10 minutes)
**Interviewer**: "Everything sounds great, but what if the load balancer itself dies? Now it's a single point of failure. How do we make the LB highly available?"

### Closing (3 minutes)
**Interviewer**: "Let me give you some feedback..."

*[Generate scorecard]*
