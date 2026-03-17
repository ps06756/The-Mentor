# 🎓 The Mentor

> **AI-Powered Interview Preparation for Software Engineers**

A collection of specialized AI skills for Claude Code and other agentic solutions to help you prepare for software engineering interviews at top tech companies.

---

## 🌟 What is this?

**The Mentor** is an open-source repository of AI interviewers—specialized prompts and instructions that transform your AI coding assistant into an expert technical interviewer. Each "skill" represents a different interview domain, difficulty level, and role type.

### Why Use The Mentor?

- 🎯 **Role-Specific Practice**: Interviewers tailored for SWE-I, Data Engineer, Frontend, Backend, and more
- 🔄 **Adaptive Difficulty**: Questions adjust based on your performance
- 💡 **Intelligent Hints**: 4-level hint system when you get stuck
- 📊 **Visual Explanations**: ASCII diagrams and Remotion video components for complex concepts
- 📈 **Progress Tracking**: Know exactly where to improve
- 🎭 **Realistic Personas**: Interviewers with distinct styles and approaches

---

## 🚀 Quick Start

### Option 1: Use with Claude Code

1. **Install Claude Code** if you haven't already
2. **Copy a skill file** from the `roles/` directory
3. **Start your interview session**:

```bash
# Example: Start Arrays & HashMaps practice for SWE-I
cat roles/swe-i/arrays-hashmaps-interviewer.md | claude
```

Or simply tell Claude:
> "Load the skill from `roles/swe-i/arrays-hashmaps-interviewer.md` and start interviewing me"

### Option 2: Use with Other AI Assistants

Each skill is a markdown file with clear instructions. Copy the content and paste it into your AI assistant of choice (ChatGPT, Claude.ai, etc.)

### Option 3: Create Your Own Skill

1. Copy `templates/skill-template.md`
2. Fill in your topic, role, and questions
3. Add it to the appropriate `roles/` directory
4. Submit a PR!

---

## 📚 Roster

### 🌱 Entry Level (SWE-I / SWE-Intern)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Arrays & HashMaps](./roles/swe-i/arrays-hashmaps-interviewer.md) | Data Structures | Easy-Medium | Two pointers, sliding window, frequency counting |
| Linked Lists | Data Structures | Easy | Reversal, merging, cycle detection |
| Binary Trees | Trees | Easy-Medium | Traversals, BFS/DFS, basic operations |
| Recursion Basics | Algorithms | Easy | Base cases, call stacks, simple problems |

### 🚀 Mid Level (SWE-II / Backend / Frontend)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [URL Shortener](./roles/systems-design/url-shortener-interviewer.md) | System Design | Medium | Distributed systems, scaling, trade-offs |
| [Rate Limiter](./roles/systems-design/rate-limiter-interviewer.md) | System Design | Medium | Token bucket, sliding window, Redis |
| Graph Algorithms | Algorithms | Medium | BFS, DFS, Dijkstra, topological sort |
| Dynamic Programming | Algorithms | Medium-Hard | Memoization, tabulation, common patterns |

### 🏗️ Specialized Roles

#### Data Engineer

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [SQL Optimization](./roles/data-engineer/sql-optimization-interviewer.md) | Database | Medium-Hard | Indexing, query plans, schema design |
| Data Pipeline Design | Data Engineering | Medium | ETL/ELT, Apache Airflow, data quality |
| Data Modeling | Data Engineering | Medium | Star schema, snowflake, data warehouses |

#### Systems Architecture & Distributed Systems

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| [Database Architecture](./roles/systems-design/database-architecture-interviewer.md) | Databases | Medium-Hard | SQL vs NoSQL, Indexing, ACID, Sharding |
| [Microservices Architecture](./roles/systems-design/microservices-architecture-interviewer.md) | Architecture | Medium-Hard | DDD, API Gateways, Sagas, Resilience |
| [Distributed Systems Core](./roles/systems-design/distributed-systems-interviewer.md) | Dist. Systems | Hard | CAP Theorem, Quorums, Consensus, Clocks |
| [Caching Architecture](./roles/systems-design/caching-architecture-interviewer.md) | Caching | Medium-Hard | Topologies, Eviction, Consistency, Stampedes |
| [Message Queues](./roles/systems-design/message-queues-interviewer.md) | Messaging | Medium-Hard | Kafka vs RabbitMQ, DLQs, Idempotency, Ordering |
| [API Design & Gateways](./roles/systems-design/api-design-interviewer.md) | API Design | Medium | REST, Pagination, Auth, API Gateways |
| [Networking & Load Balancing](./roles/systems-design/networking-load-balancing-interviewer.md) | Networking | Medium-Hard | OSI Layers, L4/L7 LBs, TLS, Consistent Hashing |

#### DevOps / SRE

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| Kubernetes Fundamentals | Infrastructure | Medium | Pods, services, deployments |
| CI/CD Pipeline Design | DevOps | Medium | GitHub Actions, Jenkins, testing strategies |
| Monitoring & Alerting | SRE | Medium | Prometheus, Grafana, SLIs/SLOs/SLAs |

#### Machine Learning Engineer

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| ML System Design | ML Engineering | Hard | Feature stores, model serving, A/B testing |
| Deep Learning Interview | ML Theory | Hard | CNNs, RNNs, Transformers, training dynamics |

### 👑 Senior+ Level (SWE-III / Senior / Staff)

| Skill | Topic | Difficulty | Description |
|-------|-------|------------|-------------|
| Design Twitter/X | System Design | Hard | Feed generation, fan-out, consistency |
| [Design Uber](./roles/systems-design/uber-interviewer.md) | System Design | Hard | Real-time tracking, matching, maps |
| Design a Search Engine | System Design | Hard | Indexing, ranking, query understanding |
| Leadership Principles | Behavioral | All Levels | STAR method, cross-functional collaboration |

---

## 🎯 How It Works

### Skill Structure

Each skill follows a consistent format:

```
🎭 Persona
   └── Who the AI interviewer is, their style, approach

🎯 Core Mission  
   └── What you'll learn and practice

📋 Interview Structure
   └── Phases: Warm-up → Core Concepts → Problem Solving → Wrap-up

🔧 Interactive Elements
   └── ASCII diagrams, Remotion components for visual learning

💡 Hint System
   └── 4 levels: Gentle nudge → Direction → Partial solution → Full walkthrough

📝 Problem Bank
   └── Curated questions with optimal solutions and follow-ups

🏆 Evaluation Rubric
   └── How to assess performance and identify weak areas

📚 Resources
   └── Books, courses, and practice problems for further study
```

### Hint System Explained

When you're stuck, the interviewer provides hints at increasing detail levels:

| Level | Type | Example |
|-------|------|---------|
| **1** | Gentle Nudge | *"Think about the time complexity. What data structure gives O(1) lookups?"* |
| **2** | Direction | *"This sounds like a dynamic programming problem. Can you identify the subproblems?"* |
| **3** | Partial Solution | *"Try using two pointers - one at start, one at end, moving towards each other."* |
| **4** | Full Walkthrough | Step-by-step explanation with pseudocode |

**Pro tip**: Try to solve with Level 1 hints first. The struggle is where learning happens!

---

## 🎨 Visual Learning

### ASCII Diagrams

Every skill includes visual explanations:

```
Two Pointers Pattern:
Array: [1, 2, 3, 4, 5, 6], Target: 7

Left →                    ← Right
  1     2  3  4  5     6
  1+6=7 ✓ Found!
```

### Remotion Components

For complex animations, we provide [Remotion](https://www.remotion.dev/) React components:

```tsx
// Example: Visualizing consistent hashing
export const ConsistentHashingDemo = () => {
  const frame = useCurrentFrame();
  // Animation logic...
  return <div>{/* Visual representation */}</div>;
};
```

Render these to video for:
- Pre-study review
- Sharing explanations with study groups
- Building your own tutorial content

---

## 🛠️ For Contributors

### Adding a New Skill

1. **Fork the repository**
2. **Copy the template**:
   ```bash
   cp templates/skill-template.md roles/{role-name}/{skill-name}.md
   ```
3. **Fill in the template** following our guidelines
4. **Test your skill** with Claude Code
5. **Submit a PR** with:
   - Clear description of what the skill covers
   - Test notes (how you verified it works)
   - Any Remotion components included

### Skill Quality Checklist

- [ ] Clear, consistent persona defined
- [ ] 3-4 difficulty-appropriate problems
- [ ] All 4 hint levels for each problem
- [ ] At least 2 visual diagrams (ASCII or Remotion)
- [ ] Evaluation rubric included
- [ ] Resources section with further reading
- [ ] Tested with at least one AI assistant

### Directory Structure

```
The-Mentor/
├── README.md                 # This file
├── LICENSE                   # MIT License
├── templates/
│   └── skill-template.md     # Template for new skills
├── roles/
│   ├── swe-i/                # Entry level
│   ├── swe-ii/               # Mid level
│   ├── swe-iii/              # Senior level
│   ├── swe-intern/           # Intern level
│   ├── data-engineer/        # Data engineering
│   ├── backend-engineer/     # Backend focus
│   ├── frontend-engineer/    # Frontend focus
│   ├── devops-sre/           # Infrastructure
│   ├── machine-learning/     # ML/AI roles
│   ├── systems-design/       # System design interviews
│   ├── principal/            # Principal+ level
│   └── senior-staff/         # Senior Staff level
├── examples/                 # Example usage sessions
└── integrations/             # Claude Code, VS Code, etc.
```

---

## 📊 Interview Preparation Roadmap

### Week 1-2: Fundamentals (SWE-I Level)
- [ ] Arrays & HashMaps
- [ ] Linked Lists
- [ ] Binary Trees
- [ ] Recursion

### Week 3-4: Intermediate (SWE-II Level)
- [ ] Graph Algorithms
- [ ] Dynamic Programming
- [ ] Sorting & Searching
- [ ] System Design (URL Shortener)

### Week 5-6: Advanced Topics
- [ ] System Design (Twitter, Uber)
- [ ] Advanced DP
- [ ] Specialized role skills

### Week 7+: Mock Interviews
- [ ] Full mock interviews using combined skills
- [ ] Behavioral practice
- [ ] Resume review

---

## 🤝 Integrations

### Claude Code

Add to your `.claude/settings.json`:
```json
{
  "skills": [
    {
      "name": "arrays-hashmaps-swe-i",
      "path": "/path/to/roles/swe-i/arrays-hashmaps-interviewer.md"
    }
  ]
}
```

### VS Code

Coming soon: Extension for quick skill selection and session management.

### Other Tools

Each skill is just markdown - use with:
- Cursor
- GitHub Copilot
- ChatGPT
- Claude.ai web interface
- Any AI assistant that accepts system prompts

---

## 📖 Example Session

**You**: Load the Arrays & HashMaps skill and start interviewing me

**Claude**: [Loads skill persona]

> 🎓 **Arrays & HashMaps Interviewer**
> 
> Welcome! I'm here to help you master fundamental array and hashmap problems. We'll start with some warm-up questions, then work through problems together.
>
> Let's begin: What's the time complexity of looking up an element in a HashMap, and why?

**You**: I think it's O(1)?

**Claude**: Exactly! Though technically it's amortized O(1)... [continues with follow-up]

[45 minutes of guided practice with hints when needed]

**Claude**: Great session! Here's your evaluation:
- Problem Understanding: 4/5
- Solution Approach: 4/5
- Code Quality: 3/5
- [Specific feedback and resources]

---

## 🌟 Success Stories

> "The Mentor helped me identify that I was jumping to code without thinking through edge cases. After 3 sessions focusing on that, I passed my Google L4 interview!" - *Software Engineer, Seattle*

> "The SQL optimization skill is incredibly detailed. The hint system helped me understand query planning in a way that LeetCode never did." - *Data Engineer, NYC*

> "I used the system design skills to practice with friends. The visual diagrams made it so much easier to explain complex concepts." - *Senior Engineer, SF*

---

## 📜 License

MIT License - see [LICENSE](./LICENSE) file for details.

Contributions welcome! Please read our [Contributing Guide](./CONTRIBUTING.md) (coming soon).

---

## 🔗 Related Projects

- [System Design Primer](https://github.com/donnemartin/system-design-primer) - Learn system design
- [NeetCode](https://neetcode.io/) - Practice problems by pattern
- [Blind 75](https://www.teamblind.com/post/New-Year-Gift---Curated-List-of-Top-75-LeetCode-Questions-to-Save-Your-Time-OaM1orEU) - Essential problem list

---

## 💬 Community

- **Issues**: Report bugs or request skills via GitHub Issues
- **Discussions**: Share your interview experiences and study tips
- **Discord**: [Join our community](https://discord.gg/thementor) (coming soon)

---

<p align="center">
  <strong>Ready to ace your next interview?</strong><br>
  Pick a skill from the <a href="#-roster">Roster</a> and start practicing!
</p>

<p align="center">
  ⭐ Star this repo if it helps you land your dream job! ⭐
</p>
