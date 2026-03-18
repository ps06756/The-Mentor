# AI Product Strategy -- Scenario Bank & Detailed Walkthroughs

---

## How to Use This Bank

Each scenario below includes the question, what the interviewer is evaluating, a detailed walkthrough at the Senior AI PM level, and common pitfalls. Adapt the depth and scope to the candidate's experience level.

---

## Scenario 1: AI-Powered Customer Support Chatbot

**Question**: "A mid-size e-commerce company handles 50,000 support tickets per month with 200 agents. Design an AI-powered solution to reduce ticket volume by 40%."

**Evaluates**: Product scoping, AI vs non-AI decision-making, metrics design, user experience for probabilistic systems.

**Detailed Walkthrough**:
- **Clarify the problem**: Before designing anything, understand the ticket distribution. Typically 30-40% of e-commerce tickets are order status, 15-20% are returns/exchanges, 10-15% are product questions, and the rest are edge cases (billing disputes, fraud, complaints). Not all categories are equally suited for AI.
- **Tiered approach**: Tier 1 (fully automated) -- order status, tracking, password resets. These have deterministic answers from backend systems; AI handles the natural language interface. Tier 2 (AI-assisted) -- returns and product questions. AI drafts a response, human agent reviews and sends. Tier 3 (human only) -- complaints, billing disputes, fraud. These require empathy and judgment.
- **Why AI, not just better rules?**: The current rule-based bot likely fails on paraphrasing, multi-intent queries, and context. AI handles "Where is my order? Also, can I return the blue shirt from that order?" in one interaction.
- **Key metrics**: Primary -- ticket deflection rate (target: 40%). Guardrails -- CSAT for AI-handled tickets (must be within 5% of human baseline), escalation rate (target: below 15%), time-to-resolution for AI-handled tickets. Leading indicator -- AI confidence score distribution over time (should shift right).
- **Data flywheel**: Every time an agent edits an AI draft, that becomes training signal. Every escalation becomes a failure case to analyze. Monthly review of the "AI got wrong" bucket to identify systematic gaps.
- **Roadmap**: MVP (month 1-2) -- AI handles order status queries only, with human fallback. V2 (month 3-4) -- expand to returns, add AI-assisted drafting for agents. V3 (month 5-6) -- personalized responses based on customer history, proactive outreach for common issues.

**Common Pitfalls**:
- Designing a generic chatbot without segmenting ticket types
- Ignoring the economics (cost per AI-handled ticket vs human-handled ticket)
- No plan for when the AI is wrong or uncertain
- Treating deflection rate as the only metric (ignoring customer satisfaction)

---

## Scenario 2: Ship or Wait -- Hallucination Rate Decision

**Question**: "Your AI feature has a 15% hallucination rate on edge cases. The CEO wants to ship next week. A competitor just launched something similar. What do you do?"

**Evaluates**: Risk management, stakeholder communication, graduated rollout strategy, business judgment under pressure.

**Detailed Walkthrough**:
- **First, segment the risk**: Is it 15% across all queries, or 15% on a specific subset? Typically, hallucination rates are not uniform. Analyze the distribution: if 80% of hallucinations occur on 20% of query types, you can ship the safe subset.
- **Assess blast radius**: What happens when the AI hallucinates? Wrong product recommendation (low risk) vs wrong refund amount (high risk) vs wrong medical information (unacceptable risk). The domain determines the calculus.
- **Propose a graduated plan to the CEO**: (1) Ship to 5% of users on the high-confidence query types only. (2) Monitor hallucination reports and user feedback for 2 weeks. (3) Define graduation criteria: hallucination rate below 5% and no severity-1 issues for 2 consecutive weeks before expanding to 25%. (4) Full rollout at 100% when hallucination rate is below 3%.
- **UX mitigations**: Add a subtle disclaimer ("AI-generated -- verify important details"). Provide a thumbs up/down feedback mechanism. Make human escalation one click away. Design the UI so AI responses look like suggestions, not authoritative answers.
- **Stakeholder communication**: Frame it as "we are shipping next week" not "we are delaying." The graduated rollout IS shipping. Show the CEO the competitor's reviews -- rushed AI products often get backlash that is worse than launching second.
- **Kill criteria**: Define what would make you pull the feature entirely. Two or more severity-1 incidents (user harmed by hallucination) in any 7-day window triggers an automatic pause.

**Common Pitfalls**:
- Binary ship/wait thinking without exploring the middle ground
- Caving to CEO pressure without a risk framework
- Ignoring the competitor launch as irrelevant (it is a real factor) or panicking about it
- No plan for monitoring or rollback after shipping

---

## Scenario 3: Legal Document Review Tool

**Question**: "Design a prompt pipeline for a legal document review tool that helps lawyers identify risky clauses in contracts. What guardrails do you need?"

**Evaluates**: High-stakes AI design, pipeline architecture thinking, human-in-the-loop design, evaluation methodology.

**Detailed Walkthrough**:
- **User and problem**: Corporate lawyers reviewing 50-100 page contracts. Current process takes 4-6 hours per contract. They are looking for non-standard clauses, missing protections, and deviations from approved language.
- **Pipeline design**: Stage 1 (extraction) -- break the document into individual clauses, classify each by type (indemnification, liability cap, termination, IP assignment, non-compete, etc.). Stage 2 (comparison) -- use RAG to compare each clause against a library of standard/approved language for that clause type. Stage 3 (risk scoring) -- assign a risk level (low/medium/high/critical) to each clause based on deviation from standard and the clause type's inherent risk. Stage 4 (human review) -- present flagged clauses to the lawyer with the AI's assessment, the specific text, and the reference standard language.
- **Guardrails**: (a) The AI never produces a final opinion -- it always surfaces findings for human review. (b) All high-risk and critical findings require lawyer sign-off. (c) A random sample of low-risk findings (10%) is reviewed by a lawyer weekly to catch false negatives. (d) Every AI assessment includes a citation to the specific contract text and the reference standard. (e) Adversarial testing: run the system against contracts with known risky clauses deliberately embedded to verify recall.
- **Evaluation**: Build a golden set of 200+ pre-reviewed contracts with expert-labeled risky clauses. Measure precision (of the clauses flagged, how many are actually risky?) and recall (of the known risky clauses, how many did the AI catch?). For legal, recall is more important than precision -- missing a risky clause is worse than over-flagging.
- **Key metrics**: Time savings per contract (target: reduce review from 5 hours to 1.5 hours), recall on risky clauses (target: 95%+), false positive rate (target: below 30%), lawyer trust score (quarterly survey).

**Common Pitfalls**:
- Designing the AI to produce a final legal opinion rather than assisting the lawyer
- No evaluation methodology or golden test set
- Ignoring the adversarial testing angle (what if the AI misses a critical clause?)
- Not thinking about the audit trail for regulated industries

---

## Scenario 4: AI Recommendation Engine -- Engagement vs Conversion Paradox

**Question**: "Your AI recommendation engine increases user engagement (time on site up 25%) but purchase conversion drops 8%. Diagnose and fix this."

**Evaluates**: Metric decomposition, understanding of AI behavior at scale, ability to diagnose counterintuitive results, system thinking.

**Detailed Walkthrough**:
- **Diagnose before fixing**: The AI is optimizing for engagement (clicks, time on site) but this is not aligned with the business objective (purchases). This is a classic proxy metric failure.
- **Hypotheses to investigate**: (1) The AI is recommending aspirational/browsing products (luxury items users click but do not buy) rather than purchase-intent products. (2) The AI is showing too much variety, creating choice paralysis. (3) The AI is surfacing products that are out of stock, out of price range, or not available in the user's size/location. (4) The recommendations are so engaging that users spend time browsing but get distracted from their original purchase intent.
- **Data analysis approach**: Segment by user intent (browsing vs buying -- inferred from session behavior). Compare recommendation click-through by segment. Analyze the purchase funnel: are users adding to cart and abandoning, or never adding to cart? Check whether the engagement increase comes from new users (exploration) or returning users (retained but not converting).
- **Fixes**: (a) Add conversion as a component of the optimization objective, not just engagement. (b) Implement a multi-objective ranking: 60% purchase likelihood, 30% engagement, 10% exploration. (c) For users with high purchase intent (items in cart, repeat visits to product pages), prioritize purchase-driving recommendations over exploration. (d) Add guardrail metrics to the recommendation system that automatically alert when conversion drops more than 5%.
- **Roadmap**: Week 1 -- deep data analysis and hypothesis validation. Week 2-3 -- implement multi-objective ranking with A/B test. Week 4 -- evaluate results. Target: restore conversion to baseline while retaining at least 15% of the engagement gain.

**Common Pitfalls**:
- Jumping to a fix without diagnosing the root cause
- Treating engagement and conversion as always aligned
- Proposing to simply optimize for conversion (loses the engagement benefit)
- Not considering different user segments and intents

---

## Scenario 5: Building an AI Writing Assistant -- From Zero to Product-Market Fit

**Question**: "You are the PM for a new AI writing assistant aimed at content marketers. You have a capable LLM, a small team, and 6 months of runway. What do you build first, and how do you find product-market fit?"

**Evaluates**: MVP scoping, AI product iteration, understanding of LLM application development, go-to-market thinking, evaluation design.

**Detailed Walkthrough**:
- **Narrow the wedge**: "Content marketers" is too broad. Pick a specific wedge: blog post drafting, social media content generation, or email copy. Each has different workflows, quality bars, and competitive landscapes. Choose based on: (a) which has the most painful current workflow, (b) which is easiest to evaluate quality, and (c) which has the clearest distribution channel.
- **MVP (month 1-2)**: Focus on one content type (e.g., blog post first drafts from a brief). Build the minimum loop: user inputs a brief (topic, audience, tone, key points), AI generates a draft, user edits, user provides feedback (thumbs up/down + optional comments). Ship to 20 beta users.
- **Evaluation framework**: AI writing quality is notoriously hard to measure. Use a multi-signal approach: (1) edit distance (how much does the user change the draft?), (2) time from brief to published post, (3) user retention (do they come back?), (4) qualitative feedback (weekly calls with beta users). Do NOT rely solely on automated metrics like BLEU or ROUGE -- they correlate poorly with perceived quality for creative writing.
- **Iteration (month 2-4)**: Based on beta feedback, iterate on the two biggest pain points. Common ones: tone is wrong, structure is generic, facts are hallucinated. For each, determine if it is a prompt engineering fix, a RAG addition (feed in company style guide, past posts), or a UX fix (give users more control over the generation).
- **Product-market fit signals**: (a) Users complete the workflow (brief to published) without abandoning. (b) Weekly retention above 40% after 4 weeks. (c) Users start requesting features (sign of investment in the tool). (d) Edit distance decreases over time for returning users (the AI learns their style through few-shot examples or fine-tuning).
- **V2 (month 4-6)**: Expand to adjacent content types. Add brand voice training (RAG over past content). Build a content calendar integration. Introduce team features if demand exists.

**Common Pitfalls**:
- Trying to build a general-purpose writing tool instead of picking a narrow wedge
- Using automated quality metrics (BLEU, ROUGE) as the primary evaluation for creative writing
- Building features before talking to users
- Not investing in evaluation infrastructure from day one
- Ignoring the competitive landscape (many AI writing tools exist -- what is the differentiation?)

---

## Quick Reference: Scenario-to-Skill Mapping

| Scenario | Primary Skill | Secondary Skill | Difficulty |
|----------|--------------|-----------------|------------|
| 1. Customer Support Chatbot | AI vs Non-AI Decision | Metrics Design | Medium |
| 2. Ship or Wait (Hallucination) | Risk Management | Stakeholder Communication | Hard |
| 3. Legal Document Review | High-Stakes AI Design | Evaluation Methodology | Hard |
| 4. Engagement vs Conversion | Metric Decomposition | System Thinking | Medium-Hard |
| 5. AI Writing Assistant | MVP Scoping | Product-Market Fit | Medium |

---

## Tips for Evaluating Answers

- **Framework test**: Does the candidate have a structured approach to AI product decisions, or do they wing it? Look for repeatable frameworks, not memorized answers.
- **Specificity test**: Can they name specific metrics with targets, specific user segments, specific rollout percentages? Vague answers like "we would monitor quality" are red flags.
- **Trade-off articulation**: AI product management is fundamentally about trade-offs (speed vs quality, cost vs performance, autonomy vs control). Strong candidates name the trade-off explicitly and explain their reasoning.
- **Failure mode thinking**: Do they proactively consider what happens when the AI is wrong? This is the single biggest differentiator between good and great AI PMs.
- **Business sense**: Can they connect AI capabilities to business outcomes? A candidate who designs a technically impressive system that does not solve a real user problem is missing the point.
