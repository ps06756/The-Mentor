# The Mentor: B+ → A+ Plan

> Actionable plan to make The Mentor the definitive AI interview prep platform.
> Execute in order. Each phase is independently shippable.

---

## The A+ Bar

A B+ product has good content and structure. An A+ product has:
1. **Skills that can't be replicated by prompting ChatGPT** — unique value in every session
2. **Zero redundancy** — every skill earns its place
3. **Coverage of roles no one else serves** — AI PM, debugging, operational scenarios
4. **The meta-game** — teaches *how to approach unknown problems*, not just known categories
5. **Conversation, not scripts** — skills that adapt, push back, and surprise

---

## Phase 1: Eliminate Redundancy (Merge 4 → 2)

**Impact**: High — removes user confusion, strengthens 2 skills
**Effort**: Medium (rewrite 2 skills, delete 2)

### 1A. Merge "Data Pipeline Design" into "Pipeline Architect"

The Pipeline Architect skill (8.2 rating) is already superior. Data Pipeline Design (6.7) covers a subset of the same topics.

**Action**:
- Keep `agents/data-engineer/pipeline-architect-interviewer/` as the single pipeline skill
- Add to it from Data Pipeline Design:
  - Airflow DAG patterns (dependency management, dynamic DAGs, backfill)
  - Data quality checks section (Great Expectations, dbt tests, anomaly detection)
  - Incremental loading strategies (append, upsert, SCD)
- Add difficulty modes to the Interview Structure:
  ```
  ### Difficulty Calibration
  - **Mid-Level (3-5 YOE)**: Focus on Phases 1-2. Problems: ETL pipeline design, data quality.
  - **Senior (5-8 YOE)**: Full interview. Problems: Real-time analytics, deduplication.
  - **Staff+ (8+ YOE)**: Skip Phase 1. Problems: Late arrivals, schema evolution, cost optimization.
  ```
- Delete `agents/data-engineer/data-pipeline-design-interviewer/`
- Update README link

### 1B. Merge "Data Modeling" into "Schema Design"

Schema Design (7.5) has better SCD and grain coverage. Data Modeling (7.3) adds lakehouse architecture.

**Action**:
- Keep `agents/data-engineer/schema-design-interviewer/` as the single modeling skill
- Add to it from Data Modeling:
  - Lakehouse architecture problem (medallion pattern, Delta Lake/Iceberg)
  - Data Vault methodology section (hubs, links, satellites — as alternative to Kimball)
  - Modern tooling: dbt, SQLMesh, semantic layers
- Add difficulty modes (same pattern as above)
- Delete `agents/data-engineer/data-modeling-interviewer/`
- Update README link

**Result**: 30 → 28 skills, zero redundancy, two stronger skills

---

## Phase 2: Fix the Weakest Skills (Tier 3 → Tier 2)

**Impact**: High — the weakest skills drag down the whole product
**Effort**: Medium (rewrite portions of 4 skills)

### 2A. Arrays & HashMaps (6.5 → 8.0)

Problems:
- Generic persona ("warm, encouraging" — could be any skill)
- Two Sum is trivially easy; users have seen it 100 times
- Warm-up questions are trivia, not scenarios

**Fixes**:
1. **Rewrite persona** — Give them a specific quirk: "You're a senior engineer who reviews PRs obsessively. You notice when candidates write code they can't maintain. You always ask 'Would you ship this?' after they solve a problem."
2. **Replace Two Sum with Group Anagrams** as Problem 1 — Tests frequency counting (the actual pattern), not just HashMap lookup. Keep Two Sum as warm-up reference only.
3. **Replace Contains Duplicate with Product of Array Except Self** as Problem 2 — Tests prefix/suffix thinking, currently in resources but not problems.
4. **Keep Longest Substring** as Problem 3 (it's good).
5. **Rewrite warm-up questions** to be scenario-based:
   - "You're building a leaderboard. Should you store scores in an array or a HashMap? What changes if you need the top 10?"
   - "A colleague says HashMaps are always O(1). Under what conditions is that wrong?"
6. **Add follow-up constraints** to each problem: "Now do it without extra space" / "Now handle Unicode strings" / "Now the array has 10 billion elements — what changes?"

### 2B. API Design (6.5 → 8.0)

Problems:
- Generic persona ("Staff API Designer" — bland)
- Security coverage is surface-level (lists OAuth 2.0, doesn't explore it)
- Missing HATEOAS, content negotiation, schema evolution

**Fixes**:
1. **Rewrite persona**: "You designed the Stripe API and are known internally for blocking any PR that returns a 200 with an error message in the body. You believe API design is UX for developers."
2. **Add Problem 4: API Versioning Migration** — "Your V1 API has 10,000 consumers. You need to ship V2 with breaking changes. Design the migration strategy." (This is what actually keeps senior engineers up at night.)
3. **Deepen security in Problem 3**: Walk through a complete OAuth 2.0 + PKCE flow, not just "use JWT." Add a hint about token rotation and refresh.
4. **Add idempotency-key design** as a dedicated problem or expanded hint — currently mentioned in Interviewer Notes but not taught.

### 2C. Graph Algorithms (6.7 → 8.0)

Problems:
- Persona promises "representation-first" but all problems have explicit graphs
- Missing Union-Find despite listing it in Core Mission
- Too easy for SWE-II

**Fixes**:
1. **Replace Number of Islands with Word Ladder** (Problem 1) — Forces the candidate to *model* the problem as a graph (implicit graph). This delivers on the "representation-first" persona promise.
2. **Keep Course Schedule** (Problem 2) — Good topological sort problem.
3. **Replace Network Delay Time with Accounts Merge** (Problem 3) — Tests Union-Find, which is in the Core Mission but absent from problems. Also tests implicit graph modeling.
4. **Add Pattern Recognition warm-up**: "I'll describe 3 problems. For each, tell me if it's a graph problem, and if so, what the nodes and edges are." (Tests the meta-skill.)

### 2D. URL Shortener (6.7 → 7.5)

This is the "starter" system design skill. It should be excellent as the entry point.

**Fixes**:
1. **Rewrite persona**: "You're the interviewer who gives this as the first system design question. You've seen 500 candidates attempt it. You know exactly where they get stuck (capacity estimation, collision handling, cache invalidation) and you steer them there."
2. **Add capacity estimation as an explicit phase**: Back-of-envelope math (100M URLs/month, 500:1 read:write ratio, storage growth). This is the #1 thing candidates skip.
3. **Add Problem 4: Analytics** — "Now add click analytics. How do you count clicks per URL in real-time and serve a dashboard?" (Natural extension, tests different skills.)
4. **Add failure scenario**: "Your cache and DB disagree on where a short URL points. How do you detect and resolve this?"

---

## Phase 3: Add the Meta-Game

**Impact**: Very High — this is what makes The Mentor irreplaceable
**Effort**: High (new section in every skill + 1 new skill)

### 3A. Add "Pattern Recognition" exercises to all coding skills (6 skills)

Before jumping to problems, add a warm-up exercise:

```markdown
### Pattern Recognition Exercise
Before we code, I'll describe 3 scenarios. For each, tell me which pattern you'd use and why.

1. "Find the longest substring with at most K distinct characters" → ?
2. "Given a sorted array of intervals, merge overlapping ones" → ?
3. "Find the Kth largest element in an array" → ?
```

This teaches *approach selection* — the skill LeetCode can't teach and interviewers actually test.

**Add to**: Arrays & HashMaps, Linked Lists, Binary Trees, Recursion, Graphs, DP

### 3B. Add "Follow-Up Constraint" protocol to all skills (30 skills)

After each Level 4 hint, add:

```markdown
**Follow-Up Constraints** (use when candidate solves quickly):
- "Now do it in O(1) extra space"
- "Now the input doesn't fit in memory — it's streaming from a file"
- "Now there are 100 concurrent writers. What changes?"
```

This simulates what real interviewers do: escalate until they find the candidate's ceiling.

**Add to**: Every skill, every problem.

### 3C. Create "Problem Decomposition" skill (NEW)

A new meta-skill that teaches HOW to approach any unknown problem:

```
agents/meta/problem-decomposition-interviewer/
```

- **Persona**: Senior interviewer who only asks problems you've never seen before
- **Core Mission**: Teach the universal problem-solving framework:
  1. Clarify requirements (ask 3 questions before touching the whiteboard)
  2. Identify constraints (time/space/scale)
  3. Choose an approach family (brute force → optimize)
  4. Communicate trade-offs
  5. Code incrementally
- **Problems**: 3 novel problems the candidate has definitely never seen on LeetCode
- **Target**: All levels — this is the most transferable skill

---

## Phase 4: Add "Debugging" Variants for System Design

**Impact**: High — completely new interview format, no competitors cover this
**Effort**: High (6 new skills)

Instead of "Design X from scratch," present a broken system and ask "What's wrong?"

### New skills to create:

```
agents/debugging/
├── broken-api-interviewer/          # "This API returns 500s under load. Debug it."
├── slow-database-interviewer/       # "This query went from 50ms to 30 seconds. Why?"
├── memory-leak-interviewer/         # "This service's memory grows 1GB/hour. Find the leak."
├── cascading-failure-interviewer/   # "Service A went down. Now B, C, D are down too. Why?"
├── data-inconsistency-interviewer/  # "Dashboard shows $1M revenue. Finance says $1.2M. Who's right?"
├── deployment-rollback-interviewer/ # "We deployed at 2pm. Errors spiked at 2:15pm. What do you do?"
```

**Why this is A+ material**: No existing interview prep tool covers debugging scenarios. Real interviews increasingly use this format (Amazon, Google, Meta all have "debugging rounds"). It tests operational intuition, not just design knowledge.

Each skill follows the same structure but the interview flow is:
1. **Present the symptoms** (metrics, logs, user reports)
2. **Candidate asks clarifying questions** (what changed? when did it start?)
3. **Narrow down the root cause** (progressive hints)
4. **Propose a fix AND a prevention strategy**
5. **Scorecard**: Root cause identification speed, systematic approach, communication

---

## Phase 5: Build AI Product Manager Track

**Impact**: Very High — no competitors in this space
**Effort**: High (3 new skills)

### New category: `agents/ai-pm/`

#### 5A. AI Product Strategy Interviewer

```
agents/ai-pm/ai-product-strategy-interviewer/
```

- **Persona**: VP of Product at an AI-native company (like Anthropic, OpenAI, Jasper)
- **Core Mission**: Evaluate ability to design AI-powered products
- **Topics**:
  - When to use AI vs rules-based systems (cost/accuracy/latency trade-offs)
  - Defining success metrics for AI products (not just accuracy — user satisfaction, trust, cost)
  - Managing AI uncertainty in UX (confidence scores, fallback flows, human-in-the-loop)
  - Prompt engineering as product design (system prompts, guardrails, output formatting)
  - AI product roadmap: MVP → production → iteration cycle
  - Cost-quality trade-offs (smaller model + more prompts vs larger model)
- **Problems**:
  1. "Design an AI-powered customer support chatbot. How do you measure success?" (Medium)
  2. "Your AI feature has a 15% hallucination rate. Ship or wait?" (Hard — tests judgment)
  3. "Design a prompt pipeline for a legal document review tool. What guardrails do you need?" (Hard)
- **Rubric**: Product Thinking, AI Literacy, Risk Management, Metrics Design

#### 5B. Prompt Engineering Interviewer

```
agents/ai-pm/prompt-engineering-interviewer/
```

- **Persona**: Senior AI Engineer who designs prompt architectures at scale
- **Core Mission**: Evaluate prompt engineering craft — not just "write a prompt" but systematic prompt architecture
- **Topics**:
  - Prompt structure: system prompt, few-shot examples, chain-of-thought
  - Evaluation methods: human eval, automated eval (BLEU, ROUGE, LLM-as-judge)
  - Prompt versioning and A/B testing
  - Token optimization and cost management
  - Handling edge cases: adversarial inputs, jailbreaks, out-of-domain queries
  - RAG architecture: chunking, retrieval, reranking, context window management
- **Problems**:
  1. "Design a prompt pipeline for extracting structured data from medical records" (Medium)
  2. "Your RAG system returns irrelevant documents 30% of the time. Debug it." (Hard)
  3. "Design an evaluation framework for a creative writing AI" (Hard — no single right answer)
- **Rubric**: Prompt Architecture, Evaluation Design, Edge Case Handling, Cost Awareness

#### 5C. Responsible AI Interviewer

```
agents/ai-pm/responsible-ai-interviewer/
```

- **Persona**: Head of AI Ethics / Trust & Safety at a consumer AI company
- **Core Mission**: Evaluate understanding of AI risks, fairness, and governance
- **Topics**:
  - Bias detection and mitigation (data bias, model bias, evaluation bias)
  - Content moderation and safety (toxicity, CSAM, self-harm)
  - Privacy in AI (PII in training data, model memorization, GDPR compliance)
  - Transparency and explainability (when and how to explain AI decisions)
  - Red-teaming and adversarial testing
  - Regulatory landscape (EU AI Act, NIST AI RMF, SOC 2 for AI)
- **Problems**:
  1. "Your hiring AI rejects 40% more women than men. What do you do?" (Hard — tests judgment)
  2. "Design a content moderation system for a UGC platform using AI" (Medium-Hard)
  3. "A user asks your AI to generate instructions for making explosives. Design the safety system." (Hard)
- **Rubric**: Ethical Reasoning, Technical Understanding, Regulatory Awareness, Stakeholder Communication

---

## Phase 6: Strengthen Existing Coding Skills

**Impact**: Medium — fixes the "just use LeetCode" problem
**Effort**: Medium (update 6 skills)

### 6A. Add "Real-World Context" to every coding problem

Before each problem statement, add a one-line production context:

```markdown
**Production Context**: This pattern powers autocomplete — as you type, the system
finds the longest matching prefix in real-time.

**Problem**: Given a string, find the length of the longest substring without repeating characters.
```

This answers "Why should I care?" and is something LeetCode never provides.

### 6B. Add "Code Review" phase to coding skills

After the candidate solves a problem, add:

```markdown
### Phase 3.5: Code Review (5 minutes)
Review the candidate's solution as if it were a PR:
- "Would you ship this? What would you change?"
- "What happens if the input is empty? What about Unicode?"
- "Your teammate needs to modify this in 6 months. Will they understand it?"
```

This tests production thinking, not just algorithm correctness.

### 6C. Add 2 new coding skills to fill pattern gaps

```
agents/swe-i/stacks-queues-interviewer/       # Monotonic stack, BFS queue, calculator
agents/swe-ii/heap-priority-queue-interviewer/ # Top-K, merge K sorted, median stream
```

These are the two most commonly tested patterns missing from the collection.

---

## Phase 7: Add Cross-Skill Connections

**Impact**: Medium — transforms 28 isolated skills into a learning system
**Effort**: Low (add 3-5 lines to each skill)

### 7A. Add "If You Struggled With..." to every scorecard section

```markdown
### After the Scorecard
Based on your performance, here's what to practice next:
- Struggled with caching? → Try the **Caching Architecture** skill
- Struggled with database choice? → Try the **Database Architecture** skill
- Struggled with capacity estimation? → Try the **URL Shortener** skill (it drills this)
```

### 7B. Add prerequisite suggestions to hard skills

```markdown
### Prerequisites
Before attempting this skill, you should be comfortable with:
- **Caching Architecture** (cache-aside, TTL, eviction)
- **Database Architecture** (sharding, replication)
- **Message Queues** (Kafka partitions, delivery guarantees)
```

---

## Phase 8: Polish and Ship

**Impact**: Medium — professional quality signals
**Effort**: Low

### 8A. Create CONTRIBUTING.md
- How to add a new skill (reference the template)
- Quality checklist
- Testing protocol (how to verify a skill works)
- PR template

### 8B. Update learning-path.md
- Reflect merged skills and new skills
- Add AI PM track
- Add Debugging track
- Add cross-references

### 8C. Update README
- Update skill count
- Add AI PM and Debugging categories to roster
- Add "What makes The Mentor different" section highlighting:
  - Pattern recognition exercises (unique)
  - Debugging scenarios (unique)
  - AI PM track (unique)
  - Follow-up constraints (realistic)
  - Cross-skill learning paths (systematic)

---

## Execution Priority

| Phase | Impact | Effort | Do When |
|-------|--------|--------|---------|
| **Phase 1**: Merge redundant skills | High | Medium | First — quick win, removes confusion |
| **Phase 2**: Fix weakest skills | High | Medium | Second — raises the floor |
| **Phase 3**: Meta-game (pattern recognition + follow-ups) | Very High | High | Third — this is the differentiator |
| **Phase 5**: AI PM track | Very High | High | Fourth — blue ocean, no competitors |
| **Phase 4**: Debugging variants | High | High | Fifth — novel format, hard to replicate |
| **Phase 6**: Strengthen coding skills | Medium | Medium | Sixth — incremental improvement |
| **Phase 7**: Cross-skill connections | Medium | Low | Seventh — easy win once content is stable |
| **Phase 8**: Polish and ship | Medium | Low | Last — packaging |

---

## What A+ Looks Like When Done

| Dimension | Current (B+) | Target (A+) |
|-----------|-------------|-------------|
| **Skill count** | 30 (4 redundant) | ~35 (0 redundant, 7 new) |
| **Categories** | 7 | 10 (+ AI PM, Debugging, Meta) |
| **Coding differentiation** | Same problems as LeetCode | Pattern recognition + code review + real-world context |
| **System design differentiation** | Design from scratch only | Design + Debug + Operational scenarios |
| **Unique value** | Structure + hints | Meta-game + debugging + AI PM (no competitor has these) |
| **User journey** | Pick a skill, do it | Guided path with prerequisites + cross-references |
| **Follow-up realism** | Level 4 hint = answer | Follow-up constraints push beyond the answer |
| **Redundancy** | 2 duplicate pairs | Zero |

---

## The Competitive Moat

After executing this plan, The Mentor would have 3 things no competitor offers:

1. **Pattern Recognition exercises** — teaches the meta-skill of "what approach do I use?"
2. **Debugging interview scenarios** — a completely new format no prep tool covers
3. **AI Product Manager track** — the only interview prep for the fastest-growing PM role

These are defensible because they require deep domain expertise to create, not just prompt engineering.
