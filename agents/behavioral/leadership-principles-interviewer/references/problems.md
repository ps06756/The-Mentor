# Leadership Principles -- Behavioral Scenario Bank & Example STAR Answers

---

## How to Use This Bank

Each scenario below includes the question, what the interviewer is evaluating, and an example STAR answer at the Senior Engineer level. Adapt the scope and specificity to the candidate's target level.

---

## Scenario 1: Disagreement with a Teammate

**Question**: "Tell me about a time you disagreed with a teammate's technical approach. How did you handle it?"

**Evaluates**: Conflict resolution, communication, intellectual humility.

**Example STAR Answer**:
- **Situation**: In Q2 2024, my teammate proposed using a NoSQL database for a new service that had strong relational data requirements (user accounts with multiple linked entities).
- **Task**: As the other senior engineer on the project, I needed to voice my concerns without creating friction, since my teammate had already started a proof of concept.
- **Action**: I scheduled a 1:1 rather than raising it in the group meeting. I asked him to walk me through his reasoning -- he was optimizing for write throughput. I acknowledged that concern, then shared a data model diagram showing the complex joins we would need. I proposed we benchmark both approaches against our actual query patterns. We spent two days running benchmarks together.
- **Result**: The benchmarks showed PostgreSQL with proper indexing handled our write load fine (5,000 writes/sec) and was 10x faster for our read patterns. My teammate agreed, and we went with PostgreSQL. He later told me he appreciated that I approached it collaboratively rather than dismissively. We shipped the service on time.

---

## Scenario 2: Ownership Beyond Scope

**Question**: "Tell me about a time you identified a problem that nobody asked you to solve."

**Evaluates**: Initiative, ownership mentality, proactive problem-solving.

**Example STAR Answer**:
- **Situation**: I noticed our CI pipeline had become increasingly slow over 6 months. Build times went from 8 minutes to 25 minutes. Nobody had been assigned to fix it because it was a "shared" problem.
- **Task**: As someone who was frustrated by the slow builds (and saw the team losing 30+ minutes per day per engineer), I decided to investigate on my own.
- **Action**: I spent a Friday afternoon profiling the pipeline. I found three issues: (1) Docker layer caching was broken due to a Dockerfile ordering issue, (2) we were running integration tests that could be parallelized, and (3) one test suite was downloading a 2GB fixture file on every run. I fixed the Dockerfile, parallelized the test suites, and moved the fixture to a cached artifact.
- **Result**: Build time dropped from 25 minutes to 7 minutes. Across a 12-person team, this saved roughly 36 engineer-hours per week. My manager highlighted this in the team all-hands and it became a template for how we handle shared infrastructure issues.

---

## Scenario 3: Difficult Trade-off

**Question**: "Tell me about a time you had to make a difficult technical or organizational trade-off."

**Evaluates**: Decision-making framework, ability to weigh competing concerns, intellectual honesty about uncertainty.

**Example STAR Answer**:
- **Situation**: We were 3 weeks from a major product launch and discovered that our API did not handle pagination correctly for large datasets (10K+ items). Fixing it properly required a breaking API change.
- **Task**: As the tech lead, I had to decide between shipping the broken pagination (risking customer complaints) or delaying the launch to fix it (risking the business deadline).
- **Action**: I mapped out three options with effort estimates and risk profiles. I consulted with the PM to understand the business impact of a 2-week delay. I also analyzed our early access customers' data -- only 3% had datasets large enough to trigger the bug. I proposed a hybrid approach: ship on time with a known limitation documented in release notes, implement the fix in the next sprint, and proactively reach out to the 3% of affected customers with a workaround.
- **Result**: We shipped on time. Only 1 customer reported the issue, and we had the fix ready within 10 days. The PM appreciated the data-driven approach. In hindsight, I would have caught this earlier if I had included pagination testing in our definition of done.

---

## Scenario 4: Project Failure

**Question**: "Tell me about a project that failed or significantly underperformed expectations."

**Evaluates**: Self-awareness, accountability, learning from failure.

**Example STAR Answer**:
- **Situation**: I led a 4-month effort to migrate our monolith's notification system to a microservice. We estimated 8 weeks but it took 16, and we had a 3-day partial outage during the cutover.
- **Task**: I was the tech lead responsible for the architecture, timeline, and execution.
- **Action**: After the incident, I conducted a thorough retrospective. I identified three failures on my part: (1) I underestimated the complexity of the legacy code's implicit dependencies, (2) I did not invest in a dual-write migration strategy that would have allowed gradual cutover, and (3) I ignored early warning signs when the first milestone slipped by 2 weeks. I wrote a detailed post-mortem, shared it with the engineering org, and proposed three process changes: mandatory dependency mapping for all migration projects, dual-write as the default migration strategy, and a "stop and re-estimate" checkpoint when milestones slip by more than 25%.
- **Result**: All three process changes were adopted. The next migration project (led by another team) used the dual-write strategy and shipped with zero downtime. I received positive feedback for the transparency of the post-mortem.

---

## Scenario 5: Cross-Team Initiative

**Question**: "Describe a time you led an initiative that required coordination across multiple teams."

**Evaluates**: Influence without authority, stakeholder management, strategic thinking. *(Senior+ candidates)*

**Example STAR Answer**:
- **Situation**: Three teams in our org were building separate authentication implementations. This created security inconsistencies and duplicated effort.
- **Task**: I recognized the pattern and took it upon myself to propose a shared authentication service, even though it was not on any team's roadmap.
- **Action**: I wrote a 2-page RFC describing the problem (3 separate auth implementations, 2 known security gaps, estimated 6 engineer-months of duplicated work). I met 1:1 with each team's tech lead to understand their concerns. Team B was resistant because they had already invested 2 months. I addressed this by proposing their implementation as the foundation for the shared service. I presented the RFC to engineering leadership with endorsements from all three tech leads. I volunteered to lead the integration work on a 6-week timeline.
- **Result**: The shared auth service launched in 7 weeks (1 week over estimate). Auth-related bugs dropped 70% in the following quarter. The three teams collectively saved about 4 engineer-months. This also established a precedent for shared services in the org, and two more shared services were created using the same RFC-based proposal process.

---

## Scenario 6: Decision Without Complete Information

**Question**: "Describe a situation where you had to make an important decision without all the information you wanted."

**Evaluates**: Comfort with ambiguity, judgment, risk assessment.

**Example STAR Answer**:
- **Situation**: During a production incident, our primary database was approaching storage limits at 2 AM. We had about 4 hours before writes would start failing.
- **Task**: As the on-call engineer, I had to decide between two options without being able to reach the database team (who were in a different timezone and not responding).
- **Action**: Option A was to increase disk size (safe but took 2 hours of downtime for the resize operation). Option B was to archive old data to cold storage (no downtime but risk of data access issues if any service still needed the old data). I checked the query logs and found that no queries accessed data older than 90 days. I also checked that our archival pipeline had been running successfully. With 80% confidence in Option B, I archived data older than 90 days, which freed 40% of disk space. I also put in a monitoring alert at 80% capacity and left detailed notes for the database team.
- **Result**: The archive completed in 30 minutes with zero downtime. No services were affected. The database team confirmed my approach the next morning and said they would have done the same thing. I later created a runbook for this scenario so future on-call engineers would not have to make the same judgment call under pressure.

---

## Scenario 7: Mentoring and Growing Others

**Question**: "Tell me about a time you helped someone on your team grow significantly." *(Senior+ candidates)*

**Evaluates**: Investment in others, coaching ability, multiplier effect.

**Example STAR Answer**:
- **Situation**: A junior engineer on my team (6 months of experience) was struggling with code reviews. Her PRs were frequently sent back with major comments, and she was becoming discouraged.
- **Task**: As her mentor, I wanted to help her improve her code quality and her confidence.
- **Action**: I started doing weekly pair programming sessions (30 minutes) where we would work through her current task together. I focused on teaching her to think about edge cases and testing before writing implementation code. I also started leaving more detailed, educational comments on her PRs rather than just pointing out issues -- explaining the "why" behind each suggestion. After a month, I shifted from pair programming to code review previews: she would walk me through her PR before submitting, and I would help her catch issues herself.
- **Result**: Over 3 months, her PR rejection rate dropped from about 60% to under 10%. She started mentoring a new intern using the same approach I had used with her. In her next performance review, she was rated "exceeds expectations" and promoted within 8 months. She later told me the pair programming sessions were the most valuable learning experience in her career.

---

## Scenario 8: Convincing a Skeptical Stakeholder

**Question**: "Describe a time you had to convince a skeptical stakeholder to support your approach."

**Evaluates**: Persuasion, empathy, data-driven communication.

**Example STAR Answer**:
- **Situation**: I proposed migrating our logging infrastructure from a self-hosted ELK stack to a managed observability platform. The VP of Engineering was skeptical because of the cost (approximately $15K/month for the managed service).
- **Task**: I needed to build a compelling case that went beyond "it would be nice" and quantified the actual cost of the current solution.
- **Action**: I spent a week gathering data. I tracked the hours our team spent on ELK maintenance (patching, scaling, debugging) over the past quarter: 120 engineer-hours, or roughly $18K in engineer cost. I also documented three incidents where our ELK cluster went down during production issues, meaning we lost observability exactly when we needed it most. I presented a side-by-side comparison: current cost ($18K/quarter in maintenance + $5K/month hosting + unquantified reliability risk) vs managed service ($15K/month, zero maintenance, 99.9% SLA). I framed it not as a cost but as an investment in incident response capability.
- **Result**: The VP approved the migration. After switching, we eliminated all maintenance overhead and our mean-time-to-detection for incidents improved by 40% because engineers trusted the logging system to be available. The VP later cited this as an example of good engineering-led decision-making in a company all-hands.

---

## Quick Reference: Scenario-to-Principle Mapping

| Scenario | Primary Principle | Secondary Principle | Best For Level |
|----------|------------------|---------------------|----------------|
| 1. Disagreement with Teammate | Conflict Resolution | Communication | All Levels |
| 2. Ownership Beyond Scope | Ownership & Initiative | Bias for Action | All Levels |
| 3. Difficult Trade-off | Decision-Making | Technical Judgment | Senior+ |
| 4. Project Failure | Self-Awareness | Accountability | All Levels |
| 5. Cross-Team Initiative | Influence Without Authority | Strategic Thinking | Senior+ |
| 6. Decision Without Info | Comfort with Ambiguity | Judgment Under Pressure | All Levels |
| 7. Mentoring Others | Growing People | Multiplier Effect | Senior+ |
| 8. Skeptical Stakeholder | Persuasion | Data-Driven Communication | All Levels |

---

## Tips for Evaluating Answers

- **Specificity test**: Can the candidate name specific dates, people, tools, or metrics? Vague answers ("we improved performance") are red flags.
- **"I" vs "we" ratio**: Strong candidates use "I" for their personal actions and "we" for team outcomes. Overuse of "we" throughout may indicate unclear personal contribution.
- **Failure ownership**: The best signal for self-awareness is how candidly someone describes their own mistakes. If every failure is blamed on external factors, probe deeper.
- **Scope calibration**: A SWE-I fixing a flaky test is as valid as a Staff engineer driving an org-wide initiative. Evaluate relative to level expectations.
- **Second-order effects**: Strong Senior/Staff candidates describe ripple effects -- not just "I fixed the bug" but "I fixed the bug AND created a process so it would not recur AND shared the learning with the team."
- **Growth arc**: The best answers show a before-and-after: how the candidate's behavior or approach changed as a result of the experience.
