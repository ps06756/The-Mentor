# Example: System Design Interview (URL Shortener)

A realistic transcript showing the AI interviewer pushing back on vague answers.

---

**Interviewer:** Hey, thanks for joining. Let's design a URL shortening service — think Bitly or TinyURL. Before we jump into architecture, what do you see as the core requirements?

**You:** We need to take long URLs and make them short, then redirect when someone visits the short URL.

**Interviewer:** Good start, but you're describing the feature, not the requirements. Let me push you — what are the *non-functional* requirements? How many URLs per day? What's the read/write ratio? What latency can we tolerate on the redirect path?

**You:** Hmm, maybe a million new URLs per day? And redirects should be fast.

**Interviewer:** "Fast" isn't a number. Let's do the math together. If we have 100 million URLs per month and a 100:1 read-to-write ratio, what's our QPS for reads?

**You:** That's... 100 million divided by 30 days, divided by 86,400 seconds... about 40 writes per second. So 4,000 reads per second.

**Interviewer:** Now we're talking. 4,000 QPS is manageable for a single server, but we'll want redundancy. Keep those numbers in mind — they'll drive every design decision. Now, how do you generate the short URL?

**You:** I'd hash the long URL with MD5 and take the first 7 characters.

**Interviewer:** Okay, three problems with that. First, what happens when two different URLs hash to the same first 7 characters? Second, MD5 produces hex — is that the best character set for a short URL? Third, hashing is deterministic — is that a feature or a bug?

**You:** I didn't think about collisions...

**Interviewer:** That's what I'm here for. Let's compare three approaches: hash-based, counter-based with Base62 encoding, and random generation. Walk me through the pros and cons of each.

*[Interview continues for 45 minutes with follow-up constraints, caching design, and database sharding...]*

---

**Scorecard:**

| Area | Rating | Notes |
|------|--------|-------|
| Requirements | Intermediate | Needed prompting to quantify scale, but got there |
| API Design | Intermediate | Clean REST endpoints, missed idempotency |
| URL Generation | Novice | Jumped to MD5 without considering alternatives |
| Caching | Intermediate | Proposed Redis cache-aside, discussed TTL |
| Trade-offs | Novice | Didn't proactively compare options |

**Strengths:** Good instinct for caching, willing to do math when prompted, communicated thought process clearly.

**Work on:** Always quantify before designing. Compare at least 2 approaches for every major decision. Think about failure modes before being asked.
