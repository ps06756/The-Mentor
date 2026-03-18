# Problem Bank: API Design & Gateway

This document contains the complete problem bank with solutions and walkthroughs for the API Design & Gateway interviewer skill.

---

## Problem 1: REST Naming Conventions

**Question**: "Design the endpoint to update a user's email address."

### Walkthrough

**Common Mistakes**:
- Using verbs in URLs: `POST /updateEmail`
- Using `PUT` for partial updates

**Ideal Answer**:
Use `PATCH /users/{id}` with a JSON payload of `{ "email": "new@email.com" }`.

**Key Concepts**:
- REST URLs should be nouns (resources), HTTP methods provide the verb
- `PUT` implies replacing the entire resource; `PATCH` is for partial updates
- Status codes: `200 OK` for success, `422 Unprocessable Entity` for validation errors

### Hints (Progressive)
1. "Should the URL contain a verb like `/updateEmail`?"
2. "In REST, the URL should be a noun (resource), and the HTTP method provides the verb."
3. "What HTTP method is used for partial updates?"
4. Full solution above.

---

## Problem 2: Pagination in Real-Time Feeds

**Question**: "We use `?page=2&limit=20` to fetch the next page of a social media feed. Users are complaining they see duplicate posts when they scroll down. Why?"

### Walkthrough

**Root Cause**: Offset-based pagination breaks when new items are inserted during browsing. If 1 new post is added at the top, the item at index 20 shifts to index 21. Requesting `OFFSET 20` returns that shifted item, which the user already saw on page 1.

**Ideal Answer**:
Use Cursor-based pagination. Instead of `page=2`, the client sends the ID of the last item they saw: `?after_id=9876`. The database queries `WHERE id < 9876 ORDER BY id DESC LIMIT 20`. This is immune to new items being inserted at the top.

**Key Concepts**:
- Offset pagination: simple but fragile with real-time data
- Cursor pagination: stable, uses a pointer (usually the last item's ID or timestamp)
- Tradeoff: cursor pagination doesn't support "jump to page 5"

### Hints (Progressive)
1. "What happens to the offsets if a new post is inserted at the top of the feed while they are reading page 1?"
2. "If 1 new post is added, the item that was at index 20 is now at index 21."
3. "If they request `OFFSET 20`, they will get the item that shifted to index 21, which they already saw on page 1."
4. Full solution above.

---

## Problem 3: API Gateway Auth

**Question**: "A user sends a JWT token to the API Gateway. The Gateway validates it. How does the downstream 'Orders Microservice' know which user made the request without validating the token again?"

### Walkthrough

**Ideal Answer**:
The API Gateway validates the JWT, extracts the `user_id`, and attaches it as an HTTP header (e.g., `X-User-Id: 123`) before proxying the request to the Orders Microservice. The microservice blindly trusts the `X-User-Id` header (assuming network security/mTLS prevents external spoofing of this header).

**Key Concepts**:
- Gateway as the single point of authentication
- Internal headers for passing identity context
- mTLS between services to prevent header spoofing
- The microservice never sees or validates the original JWT

### Hints (Progressive)
1. "The microservice shouldn't have to parse and validate the signature of the JWT if the Gateway already did it."
2. "How can the Gateway pass data to the backend over HTTP?"
3. "The Gateway can strip the external JWT and add custom HTTP headers."
4. Full solution above.

---

## Problem 4: API Versioning Strategy

**Question**: "We have a public REST API used by 500 external developers. We need to make a breaking change to the `/users` endpoint response format. How do we roll this out without breaking existing clients?"

### Walkthrough

**Ideal Answer**:
Use URL-based versioning (`/v1/users`, `/v2/users`) or header-based versioning (`Accept: application/vnd.myapi.v2+json`). Announce a deprecation timeline for v1 (e.g., 6 months). Run both versions in parallel. Use the API Gateway to route traffic to the correct backend based on the version.

**Key Concepts**:
- URL versioning: explicit, easy to understand, but clutters URLs
- Header versioning: cleaner URLs, harder to discover
- Deprecation policies: sunset headers, migration guides
- Never break existing clients without warning

---

## Problem 5: Idempotency in Payment APIs

**Question**: "A client calls `POST /payments` to charge a user $50. The server processes the payment, but the client's network drops before receiving the response. The client retries. How do we prevent the user from being charged twice?"

### Walkthrough

**Ideal Answer**:
Require an `Idempotency-Key` header. The client generates a unique key (e.g., UUID) per payment intent. The server stores the key and result. On retry, if the key already exists, return the stored result without re-processing.

**Key Concepts**:
- Idempotency keys: client-generated unique identifiers
- Server-side deduplication: store key -> result mapping
- TTL on idempotency records (e.g., 24 hours)
- This pattern is used by Stripe, PayPal, and other payment APIs

---

## Sample Interview Session

### Opening
"Hi! Thanks for joining. I'm excited to dig into API design with you today. Let's start with a practical scenario -- we're building a Twitter clone. I'd like you to design the API to fetch a user's feed, and the API to post a new tweet. Walk me through the endpoints, HTTP methods, request/response bodies, and status codes."

### Follow-up Probes
- "What status code do you return when the tweet exceeds the character limit?"
- "How do you handle a feed where new tweets are being added constantly while the user is scrolling?"
- "Our frontend team complains that the REST API requires 5 round trips to render the profile page. How can we fix this?"

### Wrap-up
Generate scorecard based on the Evaluation Rubric. Highlight strengths, improvement areas, and recommended resources.
