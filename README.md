# The Interview Mentor

**AI mock interviewers that push back on vague answers — just like real interviewers do.**

You know the algorithms. You've solved hundreds of LeetCode problems. But in a real interview, you say "I'd use a cache here" and the interviewer asks: *"What eviction policy? What's your TTL? What happens during a thundering herd?"* — and you freeze.

The Interview Mentor is a collection of 40 open-source AI interviewer agents. Each one is an expert in a specific domain — system design, algorithms, data engineering, debugging, ML, behavioral — and they don't let shallow answers slide.

**Works with Claude Code, ChatGPT, Claude.ai, Cursor, or any AI assistant.**

<p align="center">
<a href="#try-it-in-60-seconds">Try it now</a> · <a href="#choose-your-path">Choose your path</a> · <a href="#all-40-skills">Browse all skills</a> · <a href="./CONTRIBUTING.md">Contribute</a>
</p>

---

## What this actually feels like

Here's a real interaction with the Uber System Design interviewer:

> **Interviewer:** Walk me through the core user flows for an MVP ride-sharing platform — what does a rider do, what does a driver do, and where do those two paths intersect?
>
> **You:** The rider opens the app, requests a ride, we find the nearest driver, they get matched, and the driver picks them up.
>
> **Interviewer:** That's the happy path. But let me push on "find the nearest driver" — you have 1 million active drivers sending GPS pings every 4 seconds. That's 250,000 writes per second. How do you store and query that location data efficiently? A regular SQL database won't cut it.
>
> **You:** We could use Redis?
>
> **Interviewer:** Redis is a good starting point. But *how* do you query it spatially? If a rider is at coordinates (40.7, -74.0), you can't just scan all 1 million drivers. What data structure lets you find drivers within 2km without scanning everything?
>
> **You:** ...geohashing?
>
> **Interviewer:** Now we're getting somewhere. Walk me through how geohashing works and what happens when a rider is near the boundary of a geohash cell.

**That's the difference.** This isn't a chatbot that asks you questions and says "great answer!" It finds the gaps in what you *think* you know.

---

## Try it in 60 seconds

### Option A: Copy-paste into any AI (no install needed)

This works with **ChatGPT, Claude.ai, Gemini, or any AI assistant**:

1. Pick a skill from the [catalog below](#all-40-skills) and click the link
2. Copy the entire SKILL.md content
3. Paste it into your AI assistant as the first message
4. Say: **"Start the interview"**

That's it. The AI will adopt the interviewer persona and begin.

**Try these first:**
- [System Design: URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) — the classic starter
- [Coding: Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md) — entry-level warmup
- [Debugging: Broken API](./agents/debugging/broken-api-interviewer/SKILL.md) — "your API is returning 500s, fix it"

### Option B: Use with Claude Code (more features)

```bash
git clone https://github.com/ps06756/The-Interview-Mentor.git
cd The-Interview-Mentor

# Load a skill category
claude --plugin-dir ./agents/systems-design

# Start your interview
> "Use the uber-interviewer skill and start my mock interview."
```

> See [docs/use-with-claude-code.md](./docs/use-with-claude-code.md) for full setup details.

---

## Choose your path

| I'm preparing for... | Start here |
|----------------------|------------|
| **New grad / first SWE job** | [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md) → [Linked Lists](./agents/swe-i/linked-lists-interviewer/SKILL.md) → [Binary Trees](./agents/swe-i/binary-trees-interviewer/SKILL.md) |
| **Mid-level SWE (L4/SWE-II)** | [URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) → [Dynamic Programming](./agents/swe-ii/dynamic-programming-interviewer/SKILL.md) → [Rate Limiter](./agents/systems-design/rate-limiter-interviewer/SKILL.md) |
| **Senior/Staff (L5-L7)** | [Design Uber](./agents/systems-design/uber-interviewer/SKILL.md) → [Design Twitter](./agents/systems-design/twitter-interviewer/SKILL.md) → [Distributed Systems](./agents/systems-design/distributed-systems-interviewer/SKILL.md) |
| **Data Engineer** | [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer/SKILL.md) → [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer/SKILL.md) → [Schema Design](./agents/data-engineer/schema-design-interviewer/SKILL.md) |
| **DevOps / SRE** | [Kubernetes](./agents/devops-sre/kubernetes-interviewer/SKILL.md) → [CI/CD Pipeline](./agents/devops-sre/cicd-pipeline-interviewer/SKILL.md) → [Monitoring](./agents/devops-sre/monitoring-alerting-interviewer/SKILL.md) |
| **ML Engineer** | [ML System Design](./agents/ml-engineer/ml-system-design-interviewer/SKILL.md) → [Deep Learning](./agents/ml-engineer/deep-learning-interviewer/SKILL.md) |
| **AI Product Manager** | [AI Product Strategy](./agents/ai-pm/ai-product-strategy-interviewer/SKILL.md) → [Prompt Engineering](./agents/ai-pm/prompt-engineering-interviewer/SKILL.md) |
| **Behavioral prep** | [Leadership Principles](./agents/behavioral/leadership-principles-interviewer/SKILL.md) |
| **I don't know where to start** | [Problem Decomposition](./agents/meta/problem-decomposition-interviewer/SKILL.md) — teaches you how to approach *any* unknown problem |

> For a structured 8-week curriculum, see [references/learning-path.md](./references/learning-path.md).

---

## Why this is different

| Generic interview prep | The Interview Mentor |
|----------------------|---------------------|
| Asks a question, you answer, it says "correct!" | Catches you saying "I'd use a cache" and asks about eviction policies, TTL, and thundering herds |
| One difficulty level | Calibrates to you — goes easier if you're struggling, harder if you're breezing through |
| You get stuck and just see the answer | 4-level hint system: nudge → direction → partial solution → full walkthrough |
| No feedback on weak areas | Generates a scorecard rating you across multiple dimensions (Novice / Intermediate / Expert) |
| Same questions every time | 40 specialized interviewers with distinct personas and deep domain expertise |
| Only covers coding | Coding + system design + data engineering + DevOps + ML + AI PM + debugging + behavioral |

---

## All 40 skills

### Coding (8 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer/SKILL.md) | Easy-Medium | Frequency counting, prefix/suffix products, sliding window |
| [Linked Lists](./agents/swe-i/linked-lists-interviewer/SKILL.md) | Easy | Reversal, merging, cycle detection, fast/slow pointers |
| [Binary Trees](./agents/swe-i/binary-trees-interviewer/SKILL.md) | Easy-Medium | Traversals, BFS/DFS, BST validation |
| [Recursion & Backtracking](./agents/swe-i/recursion-basics-interviewer/SKILL.md) | Easy | Call stacks, base cases, backtracking |
| [Stacks & Queues](./agents/swe-i/stacks-queues-interviewer/SKILL.md) | Easy-Medium | Monotonic stack, expression evaluation |
| [Graph Algorithms](./agents/swe-ii/graph-algorithms-interviewer/SKILL.md) | Medium | BFS, DFS, implicit graphs, union-find |
| [Dynamic Programming](./agents/swe-ii/dynamic-programming-interviewer/SKILL.md) | Medium-Hard | Memoization, tabulation, knapsack, LCS |
| [Heaps & Priority Queues](./agents/swe-ii/heap-priority-queue-interviewer/SKILL.md) | Medium | Top-K, merge K sorted, median stream |

### System Design (13 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [URL Shortener](./agents/systems-design/url-shortener-interviewer/SKILL.md) | Medium | Capacity estimation, hashing, caching, analytics |
| [Rate Limiter](./agents/systems-design/rate-limiter-interviewer/SKILL.md) | Medium | Token bucket, sliding window, Redis atomicity |
| [Database Architecture](./agents/systems-design/database-architecture-interviewer/SKILL.md) | Medium-Hard | SQL vs NoSQL, B-Trees vs LSM, ACID, sharding |
| [Caching Architecture](./agents/systems-design/caching-architecture-interviewer/SKILL.md) | Medium-Hard | Cache-aside, write-through, thundering herd, bloom filters |
| [API Design](./agents/systems-design/api-design-interviewer/SKILL.md) | Medium | REST, pagination, versioning, idempotency, OAuth |
| [Message Queues](./agents/systems-design/message-queues-interviewer/SKILL.md) | Medium-Hard | Kafka vs RabbitMQ, exactly-once myth, dead letter queues |
| [Microservices](./agents/systems-design/microservices-architecture-interviewer/SKILL.md) | Medium-Hard | DDD, sagas, circuit breakers, service boundaries |
| [Distributed Systems](./agents/systems-design/distributed-systems-interviewer/SKILL.md) | Hard | CAP theorem, quorums, Raft, vector clocks, fencing tokens |
| [Networking & Load Balancing](./agents/systems-design/networking-load-balancing-interviewer/SKILL.md) | Medium-Hard | L4/L7 LBs, TLS termination, consistent hashing |
| [Reliability & Observability](./agents/systems-design/reliability-observability-interviewer/SKILL.md) | Medium-Hard | Circuit breakers, SLOs, exponential backoff with jitter |
| [Design Uber](./agents/systems-design/uber-interviewer/SKILL.md) | Hard | Geospatial indexing, real-time matching, concurrency |
| [Design Twitter](./agents/systems-design/twitter-interviewer/SKILL.md) | Hard | Fan-out on write vs read, timeline ranking |
| [Design a Search Engine](./agents/systems-design/search-engine-interviewer/SKILL.md) | Hard | Crawling, inverted index, TF-IDF, autocomplete |

### Data Engineering (3 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer/SKILL.md) | Medium-Hard | EXPLAIN plans, composite indexes, partitioning |
| [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer/SKILL.md) | Medium-Hard | Kafka/Flink, Airflow DAGs, exactly-once, late arrivals |
| [Schema Design](./agents/data-engineer/schema-design-interviewer/SKILL.md) | Medium-Hard | Star schemas, SCDs, grain, lakehouse architecture |

### DevOps / SRE (3 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [Kubernetes](./agents/devops-sre/kubernetes-interviewer/SKILL.md) | Medium | Pods, deployments, HPA, CrashLoopBackOff debugging |
| [CI/CD Pipeline](./agents/devops-sre/cicd-pipeline-interviewer/SKILL.md) | Medium | Blue-green, canary, database migrations in CI |
| [Monitoring & Alerting](./agents/devops-sre/monitoring-alerting-interviewer/SKILL.md) | Medium | SLOs, burn-rate alerts, alert fatigue |

### ML Engineering (2 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [ML System Design](./agents/ml-engineer/ml-system-design-interviewer/SKILL.md) | Hard | Feature stores, model serving, A/B testing, drift |
| [Deep Learning](./agents/ml-engineer/deep-learning-interviewer/SKILL.md) | Hard | Transformers, training dynamics, convergence debugging |

### AI Product Management (3 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [AI Product Strategy](./agents/ai-pm/ai-product-strategy-interviewer/SKILL.md) | Hard | When to use AI, success metrics, ship-vs-wait decisions |
| [Prompt Engineering](./agents/ai-pm/prompt-engineering-interviewer/SKILL.md) | Hard | RAG architecture, evaluation frameworks, token optimization |
| [Responsible AI](./agents/ai-pm/responsible-ai-interviewer/SKILL.md) | Hard | Bias mitigation, content moderation, EU AI Act |

### Debugging & Incident Response (6 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [Broken API](./agents/debugging/broken-api-interviewer/SKILL.md) | Medium-Hard | 500 errors under load, connection pools, deadlocks |
| [Slow Database](./agents/debugging/slow-database-interviewer/SKILL.md) | Medium-Hard | Query regression, stale statistics, lock contention |
| [Memory Leak](./agents/debugging/memory-leak-interviewer/SKILL.md) | Medium-Hard | Unbounded caches, listener leaks, OOM debugging |
| [Cascading Failure](./agents/debugging/cascading-failure-interviewer/SKILL.md) | Hard | Thread pool exhaustion, retry storms, missing circuit breakers |
| [Data Inconsistency](./agents/debugging/data-inconsistency-interviewer/SKILL.md) | Hard | Timezone mismatches, duplicate events, reconciliation |
| [Deployment Rollback](./agents/debugging/deployment-rollback-interviewer/SKILL.md) | Medium-Hard | Failed deploys, incompatible migrations, feature flags |

### Behavioral & Meta (2 skills)

| Skill | Difficulty | What it tests |
|-------|-----------|---------------|
| [Leadership Principles](./agents/behavioral/leadership-principles-interviewer/SKILL.md) | All Levels | STAR method, ownership, cross-functional collaboration |
| [Problem Decomposition](./agents/meta/problem-decomposition-interviewer/SKILL.md) | All Levels | How to approach any unknown problem — pattern recognition, structured thinking |

---

## How each session works

Every interviewer follows the same structure:

1. **Warm-up** (5-10 min) — foundational questions to calibrate your level
2. **Deep dive** (15-20 min) — core concepts with visual explanations and follow-ups
3. **Problem solving** (20-30 min) — realistic problems with progressive hints
4. **Scorecard** (5 min) — rated across multiple dimensions with specific improvement areas

When you're stuck, ask for a hint:

| Level | What you get |
|-------|-------------|
| 1 | A nudge toward the right concept |
| 2 | The approach family you should consider |
| 3 | Pseudocode or a partial solution |
| 4 | Full walkthrough with explanation |

---

## FAQ

**Do I need Claude Code?**
No. The fastest way to start is copying a SKILL.md file into ChatGPT, Claude.ai, or any AI. Claude Code gives a more integrated experience but isn't required.

**Does this work with ChatGPT?**
Yes. Copy the content of any SKILL.md file, paste it as the first message, and say "Start the interview." Works with GPT-4, GPT-4o, Claude, Gemini, and other capable models.

**Is this only for coding interviews?**
No. We cover system design, data engineering, DevOps, ML, AI product management, debugging/incident response, and behavioral interviews — 40 skills total.

**How do I add a new skill?**
See [CONTRIBUTING.md](./CONTRIBUTING.md). Copy the [skill template](./templates/skill-template/SKILL.md), fill it in, test it, submit a PR.

**Where should I start?**
If you're not sure, start with [Problem Decomposition](./agents/meta/problem-decomposition-interviewer/SKILL.md) — it teaches you how to approach *any* problem, then directs you to specific skills based on your gaps.

---

## Created by

| **Pratik Singhal** | **Abhishek Garg** |
|:------------------:|:-----------------:|
| Senior Engineer at Route 53, AWS | Senior Engineering Manager at Merge.dev |
| [LinkedIn](https://www.linkedin.com/in/ps06756/) | [LinkedIn](https://www.linkedin.com/in/abhishek-garg-09040574/) |

Built from experience conducting hundreds of real interviews. We kept seeing the same pattern: candidates who knew the material but couldn't communicate it under pressure. Mock interviews fix that — but human mock interviewers are inconsistent, expensive, and hard to schedule. AI solves all three.

---

## Contributing

We welcome new skills, improvements to existing ones, and bug fixes. See [CONTRIBUTING.md](./CONTRIBUTING.md) for the full guide.

Quick version:
```bash
cp -r templates/skill-template agents/{category}/{skill-name}-interviewer
# Fill in the template, test with Claude Code or ChatGPT, submit a PR
```

---

## License

MIT — see [LICENSE](./LICENSE).

<p align="center">
  <strong>Stop practicing alone. Start finding your gaps.</strong><br>
  <a href="#try-it-in-60-seconds">Try it now</a>
</p>
