# 🎓 The Mentor

> **100+ AI Interview Agents for Claude Code**

Stop practicing alone. Get realistic, adaptive mock interviews with expert AI agents that interview like engineers from Google, Meta, Amazon, and other top tech companies.

---

## 👥 Created By

This project was built by engineers who've conducted hundreds of interviews at top tech companies:

| **Pratik Singhal** | **Abhishek Garg** |
|:------------------:|:-----------------:|
| Senior Engineer at Route53, AWS | Senior Engineering Manager at Merge.dev |
| [LinkedIn](https://www.linkedin.com/in/ps06756/) | [LinkedIn](https://www.linkedin.com/in/abhishek-garg-09040574/) |

We created The Mentor because we believe interview preparation should be accessible, realistic, and effective.

---

## 🎯 Why Mock Interviews Matter

**The "Thinking Out Loud" Gap**

You know the algorithms. You've solved hundreds of LeetCode problems. But in a real interview, your mind goes blank. You freeze. You know the answer but can't communicate it under pressure.

This is the #1 reason qualified candidates fail interviews.

**How AI Agents Help You Prepare**

- **Realistic Pressure Simulation** — Practice in an environment that feels like the real thing, so the actual interview feels familiar
- **Adaptive Follow-Ups** — Agents don't just ask canned questions. They probe deeper based on your answers, just like real interviewers
- **Unlimited Practice** — No scheduling, no awkwardness, no cost per session. Practice daily until you're confident
- **Instant Detailed Feedback** — Get specific, actionable feedback immediately after each session, not days later
- **Safe Environment** — Stumble, make mistakes, and learn from them without consequences

> "The difference between a good engineer and a hired engineer often isn't knowledge—it's communication under pressure. Mock interviews close that gap."

---

## ✨ What Makes The Mentor Different

| Feature | What You Get |
|---------|--------------|
| **🎭 Role-Specific Expertise** | Each agent specializes in one domain (databases, system design, algorithms) and goes deeper than generic prep |
| **🔄 Adaptive Difficulty** | Questions adjust based on your performance—get challenged at exactly the right level |
| **💡 4-Level Hint System** | Stuck? Get hints from gentle nudges to full walkthroughs. Learn, don't just memorize |
| **📊 Visual Explanations** | ASCII diagrams and visual components for complex concepts |
| **🎤 Distinct Personas** | Each interviewer has their own style—some friendly, some rigorous, some focused on trade-offs |
| **🎯 Evaluation Rubrics** | Know exactly where you stand and what to improve |

---

## 🚀 How to Use

### Step 1: Download the Repository

Download the latest release as a ZIP file:

```bash
curl -L https://github.com/ps06756/The-Mentor/archive/refs/tags/v1.0.0.zip -o The-Mentor.zip
unzip The-Mentor.zip
```

Or manually download from: https://github.com/ps06756/The-Mentor/archive/refs/tags/v1.0.0.zip

### Step 2: Install in Claude Code

Open Claude Code and run the following commands:

```
/plugin install <path_to_the_directory_downloaded>
```

Then install the coding interview agent:

```
/plugin install coding-interview-agent
```

### Step 3: Start Preparing

Once installed, you can start practicing with prompts like:

> *"Can you help me prepare for system design interview using coding-interview-agent?"*

Or for other topics:

> *"Can you help me prepare for SQL optimization interview using coding-interview-agent?"*

> *"Can you help me prepare for distributed systems interview using coding-interview-agent?"*

The agent will take over and conduct a realistic mock interview tailored to your chosen topic.

---

## 📚 The Roster

### 💾 **Databases**

Master query optimization, indexing strategies, ACID vs BASE, sharding patterns, and CAP theorem trade-offs.

| Agent | Focus Area | Difficulty |
|-------|-----------|------------|
| [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer.md) | Query plans, indexing, execution analysis | Medium-Hard |
| [Schema Design](./agents/data-engineer/schema-design-interviewer.md) | Dimensional modeling, SCDs, star schema | Medium-Hard |
| [Database Architecture](./agents/systems-design/database-architecture-interviewer.md) | SQL vs NoSQL, ACID, sharding strategies | Medium-Hard |

**What you'll master:** How to choose the right database, optimize slow queries, design schemas that scale, and handle trade-offs between consistency and availability.

---

### 🏗️ **System Design**

Learn to design scalable, reliable systems that handle millions of users.

| Agent | Focus Area | Difficulty |
|-------|-----------|------------|
| [Uber/Lyft Design](./agents/systems-design/uber-interviewer.md) | Real-time tracking, matching algorithms, maps | Hard |
| [URL Shortener](./agents/systems-design/url-shortener-interviewer.md) | Distributed systems, scaling, trade-offs | Medium |
| [Rate Limiter](./agents/systems-design/rate-limiter-interviewer.md) | Token bucket, sliding window, Redis | Medium |
| [Caching Architecture](./agents/systems-design/caching-architecture-interviewer.md) | Topologies, eviction, consistency | Medium-Hard |
| [Message Queues](./agents/systems-design/message-queues-interviewer.md) | Kafka vs RabbitMQ, DLQs, idempotency | Medium-Hard |
| [API Design](./agents/systems-design/api-design-interviewer.md) | REST, pagination, auth, gateways | Medium |

**What you'll master:** Trade-off analysis, scalability patterns, failure mode handling, and how to communicate your design decisions clearly.

---

### 📊 **Data Engineering**

Build end-to-end data pipelines that process terabytes reliably.

| Agent | Focus Area | Difficulty |
|-------|-----------|------------|
| [Pipeline Architect](./agents/data-engineer/pipeline-architect-interviewer.md) | ETL/ELT, Kafka/Flink, scaling, failure modes | Medium-Hard |
| [Schema Design](./agents/data-engineer/schema-design-interviewer.md) | Data modeling, warehouses, lakehouses | Medium-Hard |
| [SQL Optimization](./agents/data-engineer/sql-optimization-interviewer.md) | Query tuning for analytics workloads | Medium-Hard |

**What you'll master:** Pipeline orchestration, data quality strategies, streaming vs batch trade-offs, and designing for data reliability.

---

### 🌐 **Distributed Systems**

Understand the fundamentals that power modern cloud infrastructure.

| Agent | Focus Area | Difficulty |
|-------|-----------|------------|
| [Distributed Systems Core](./agents/systems-design/distributed-systems-interviewer.md) | CAP theorem, quorums, consensus, clocks | Hard |
| [Microservices Architecture](./agents/systems-design/microservices-architecture-interviewer.md) | DDD, API gateways, sagas, resilience | Medium-Hard |
| [Networking & Load Balancing](./agents/systems-design/networking-load-balancing-interviewer.md) | OSI layers, L4/L7 LBs, consistent hashing | Medium-Hard |
| [Reliability & Observability](./agents/systems-design/reliability-observability-interviewer.md) | Circuit breakers, RED metrics, RTO/RPO | Medium-Hard |

**What you'll master:** Consensus algorithms, failure detection, service discovery, and designing for reliability at scale.

---

### 💻 **Algorithms & Data Structures (SWE-I)**

Build a strong foundation for entry-level interviews.

| Agent | Focus Area | Difficulty |
|-------|-----------|------------|
| [Arrays & HashMaps](./agents/swe-i/arrays-hashmaps-interviewer.md) | Two pointers, sliding window, frequency counting | Easy-Medium |

**What you'll master:** Pattern recognition, time/space complexity analysis, and translating ideas to clean code.

---

## 🎓 Deep Domain Mastery

Each agent focuses on **ONE specific area** and goes deeper than generic interview prep:

| Domain | Depth of Coverage |
|--------|-------------------|
| **Databases** | Query optimization → Index strategies → Execution plans → ACID/BASE → Sharding → CAP trade-offs → Replication |
| **System Design** | Requirements → Estimation → API design → Data model → High-level design → Deep dives → Trade-offs |
| **Data Engineering** | Pipeline architecture → Streaming vs Batch → Schema evolution → Data quality → Failure recovery |
| **Distributed Systems** | Consensus → Consistency models → Failure modes → Clock synchronization → Partition tolerance |
| **Algorithms** | Pattern recognition → Edge cases → Complexity analysis → Code optimization → Follow-up variations |

> **Why depth matters:** Interviewers at top companies don't ask superficial questions. They keep probing until they find the boundary of your knowledge. Our agents do the same—so you know exactly where your gaps are before the real interview.

---

## 🏆 Who This Is For

- **New Grads & Interns** preparing for their first SWE role
- **Mid-Level Engineers** (SWE-II) aiming for promotions or new opportunities
- **Senior Engineers** (SWE-III) targeting staff-level positions
- **Specialists** in Data Engineering, ML, DevOps, and SRE roles
- **Anyone** who knows the material but struggles with interview pressure

---

## 📊 By The Numbers

<div align="center">

| 💯 **100+** | 🎯 **15+** | 🚀 **All Levels** | 🤖 **Claude Native** |
|-------------|-----------|-------------------|---------------------|
| Specialized Agents | Technical Domains | Entry to Staff+ | Code Integration |

</div>

---

## 🤝 Integrations

### Claude Code (Recommended)

Claude Code natively supports custom agents via the `claude skill add` command.

```bash
# Add any interviewer from the agents/ directory
claude skill add https://raw.githubusercontent.com/ps06756/The-Mentor/main/agents/systems-design/uber-interviewer.md
```

Then start your interview:
> *"Begin my system design interview using the uber-interviewer skill."*

### Other AI Assistants

Each agent is a markdown file with clear instructions. Copy the content and paste it into ChatGPT, Claude.ai, or any AI assistant.

### VS Code (Cline) & Cursor

Save the skill file to your workspace and reference it:
> *"Use the instructions in `@uber-interviewer.md` to conduct a mock interview with me."*

---

## 🛠️ For Contributors

### Adding a New Agent

1. **Fork the repository**
2. **Copy the template**:
   ```bash
   cp templates/skill-template.md agents/{role-name}/{agent-name}.md
   ```
3. **Fill in the template** following our guidelines
4. **Test your agent** with Claude Code
5. **Submit a PR**

### Agent Quality Checklist

- [ ] Clear, consistent persona defined
- [ ] 3-4 difficulty-appropriate problems
- [ ] All 4 hint levels for each problem
- [ ] At least 2 visual diagrams (ASCII or Remotion)
- [ ] Evaluation rubric included
- [ ] Resources section with further reading
- [ ] Tested with at least one AI assistant

---

## 📜 License

MIT License - see [LICENSE](./LICENSE) file for details.

<p align="center">
  <strong>Ready to ace your next interview?</strong><br>
  Pick an agent from the <a href="#-the-roster">Roster</a> and start practicing!
</p>

<p align="center">
  ⭐ Star this repo if it helps you land your dream job! ⭐
</p>
