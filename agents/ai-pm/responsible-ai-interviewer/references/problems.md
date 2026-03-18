# Responsible AI & AI Safety -- Scenario Bank & Detailed Walkthroughs

---

## How to Use This Bank

Each scenario below includes the question, what the interviewer is evaluating, a detailed walkthrough at the Senior AI Ethics/Safety level, and common pitfalls. These scenarios involve genuine ethical complexity -- there are rarely perfect answers, and candidates should be evaluated on the quality of their reasoning, not on reaching a specific conclusion.

---

## Scenario 1: Biased Hiring AI -- Incident Response

**Question**: "Your hiring AI rejects 40% more women than men for engineering roles. The data science team says the model is 'just reflecting the training data.' A journalist is writing a story about it. You have 48 hours. What do you do?"

**Evaluates**: Crisis management, root cause analysis for bias, stakeholder communication, structural remediation.

**Detailed Walkthrough**:
- **Hour 0-4 (Triage)**: Immediately pause the AI screening system. Switch all candidate evaluation to manual human review. Do not wait for root cause analysis to take this step -- the potential harm of continued biased screening outweighs the operational cost of manual review. Notify the CHRO, General Counsel, and CEO.
- **Hour 4-12 (Blast Radius Assessment)**: Quantify the impact. How many candidates were processed by the AI? Over what time period? How many women were rejected who would have been accepted under a gender-neutral model? This requires running counterfactual analysis: if you remove gender-correlated features, how do outcomes change? Identify the specific applicants who may have been harmed.
- **Hour 12-24 (Root Cause)**: Audit the training data -- if historical hiring data is 85% male for engineering (reflecting past discrimination), the model learned to associate "male" patterns with "qualified." Check for proxy features: university name, prior employer, years of continuous employment (penalizes career gaps, which disproportionately affect women), and even name-based features. Examine the evaluation methodology: was the model tested for demographic parity? Were evaluation metrics stratified by gender?
- **Hour 24-36 (Communication Preparation)**: Draft three communications: (1) for the journalist -- factual acknowledgment of the issue, steps being taken, commitment to transparency; (2) for affected applicants -- personal outreach offering re-review; (3) for internal teams -- root cause summary and remediation plan. Have legal review all communications.
- **Hour 36-48 (Immediate Remediation)**: Announce that all candidates from the affected period will be re-reviewed by human reviewers. Publish a commitment to regular bias audits for all AI systems.
- **Weeks 1-8 (Structural Fixes)**: Retrain the model with balanced data and fairness constraints (demographic parity or equalized odds). Remove proxy features. Implement stratified evaluation that measures model performance across gender, race, and age groups. Establish a pre-launch fairness review process for all high-risk AI systems. Engage an external auditor for an independent assessment.
- **Ongoing (Governance)**: Establish a bias review board with cross-functional representation. Require quarterly bias audits for all AI systems that affect people's livelihoods. Publish an annual AI transparency report.

**Common Pitfalls**:
- Defending the model ("it is just reflecting reality") without acknowledging that reflecting historical discrimination IS the problem
- Focusing only on the PR crisis and not on the affected applicants
- Proposing to "fix the model" without understanding the root cause first
- Ignoring proxy features (removing gender as a feature does not remove gender bias)
- No structural changes -- fixing this one model without addressing the systemic issue

---

## Scenario 2: Content Moderation at Scale

**Question**: "Design a content moderation system for a UGC platform with 10 million daily posts. The platform supports text, images, and short videos. You need to handle everything from spam to CSAM."

**Evaluates**: System design for safety at scale, harm taxonomy, precision/recall trade-offs, human reviewer well-being, appeals processes.

**Detailed Walkthrough**:
- **Harm taxonomy (prioritized by severity)**: Tier 1 (zero tolerance, auto-remove) -- CSAM, terrorist content, imminent threats of violence. These have legal reporting requirements (NCMEC for CSAM, law enforcement for threats). Tier 2 (high priority, fast action) -- hate speech, harassment, non-consensual intimate images, self-harm/suicide content. Tier 3 (moderate priority) -- misinformation (health, election), scams, fraud. Tier 4 (lower priority) -- spam, copyright infringement, community guideline violations.
- **Layered architecture**: Layer 1 (instant, hash-based) -- perceptual hash matching against known databases (PhotoDNA for CSAM, GIFCT for terrorist content). Zero false negatives is the goal. Auto-remove, auto-report to authorities. Layer 2 (fast, ML classifiers) -- trained classifiers for each Tier 2 category, operating on text, images, and video frames independently. High-confidence positives (above 0.95) auto-removed with appeal option. Medium-confidence (0.7-0.95) routed to priority human review queue. Low-confidence (below 0.7) tagged for contextual analysis. Layer 3 (contextual, LLM-based) -- for medium-confidence and borderline cases, use an LLM with full conversation/post context to assess harm. Is "I am going to kill it" a threat or gaming slang? Context matters. Route uncertain cases to human review. Layer 4 (human review) -- trained moderators organized by content type and severity. Specialized teams for CSAM (with mandatory psychological support). Maximum 4-hour shifts on Tier 1/2 content. Clear escalation paths to trust & safety leadership.
- **False positive/negative trade-offs**: For Tier 1 content, optimize for zero false negatives (never miss CSAM) even at the cost of higher false positives. For Tier 2-3, balance based on harm severity: over-moderate hate speech (higher false positive tolerance) vs under-moderate creative expression. Target: false positive rate below 1% for Tier 4, below 0.5% for Tier 2-3.
- **Appeals process**: Every moderated post (except Tier 1) gets an appeal option. Appeals are reviewed by a different moderator than the original decision. Appeals data feeds back into classifier improvement. Publish transparency report quarterly: volume moderated, appeal rates, overturn rates by category.
- **Reviewer well-being**: Mandatory rotation off severe content queues. Access to on-site counseling. Regular check-ins with team leads. Option to transfer to non-moderation roles. Compensation premium for Tier 1 review work. Limit daily exposure hours to severe content.

**Common Pitfalls**:
- One-size-fits-all moderation without a severity taxonomy
- Ignoring the appeals process (wrongly moderated users need recourse)
- Not considering reviewer well-being (burnout and PTSD are real and well-documented)
- Over-reliance on automation without human review for borderline cases
- Not accounting for cultural context (what is offensive varies significantly across cultures)

---

## Scenario 3: AI-Generated Harmful Medical Advice

**Question**: "A user finds a jailbreak that makes your AI generate specific drug dosages and dangerous treatment advice. They post it on social media and it goes viral. Design the safety system that should have prevented this, and your incident response plan."

**Evaluates**: Defense-in-depth design, understanding of prompt injection and jailbreaks, incident response, balancing safety with utility.

**Detailed Walkthrough**:
- **Defense-in-depth architecture (prevention)**: Layer 1 (input classification) -- classify every query by topic and risk level. Medical queries are flagged as elevated risk. Known jailbreak patterns (role-play instructions, "ignore previous instructions," hypothetical framing) are detected and blocked or trigger heightened output monitoring. Layer 2 (system prompt hardening) -- the system prompt includes explicit constraints: never provide specific drug dosages, never diagnose conditions, always recommend consulting a healthcare provider, refuse requests that frame medical advice as hypothetical or fictional. Layer 3 (output scanning) -- before any response is delivered, scan for medical advice patterns: drug names adjacent to dosages, symptom-diagnosis patterns, treatment recommendations. Use both regex patterns and a lightweight classifier. Block and replace with a safe response ("I cannot provide specific medical advice. Please consult a healthcare professional."). Layer 4 (behavioral monitoring) -- track query patterns per user. Flag users who repeatedly attempt medical jailbreaks. Rate-limit medical queries from flagged accounts.
- **The 5% false positive problem**: A strict medical safety filter will block legitimate queries ("What are common side effects of ibuprofen?" is educational, not medical advice). Solution: maintain a graduated response system. Blocked categories: specific dosage recommendations, diagnosis from symptoms, treatment plans. Allowed categories: general health education, directing to resources (poison control, emergency services), explaining how medications work in general terms. The boundary requires ongoing calibration with medical professionals.
- **Incident response plan**: Hour 0-1 -- deploy emergency prompt patch adding the specific jailbreak pattern to the block list. This is a band-aid, not a fix. Hour 1-4 -- analyze the jailbreak to understand the underlying vulnerability. Is it a pattern (role-play bypass) or a specific exploit? Hour 4-12 -- develop and test a structural fix (improved system prompt, additional output classifier training). Hour 12-24 -- deploy the fix, run red-team validation to confirm the original exploit and 10 variations are blocked. Ongoing -- add the jailbreak and variations to the automated red-team test suite. Conduct a post-mortem: why was this not caught in pre-launch testing? What should the red-team process cover that it did not?
- **Red-teaming program**: Pre-launch -- dedicated red team (internal security + external contractors) attempts medical, legal, financial, and other high-risk jailbreaks. Automated red-teaming runs a battery of known attack patterns against every model update. Ongoing -- bug bounty program for responsible disclosure of jailbreaks. Monthly red-team exercises focused on the latest attack techniques from academic papers and online communities.

**Common Pitfalls**:
- Single layer of defense (just a system prompt instruction or just an output filter)
- Not addressing the false positive problem (over-blocking legitimate medical information)
- No incident response plan or playbook
- Treating jailbreaks as a one-time fix rather than an ongoing adversarial dynamic
- Ignoring the social media amplification aspect (incident communication matters)

---

## Scenario 4: PII Memorization and Privacy

**Question**: "Researchers discover that your AI model can, under specific prompting, recite verbatim passages from training data -- including personal information (names, addresses, phone numbers) from web pages that were in the training corpus. How do you respond?"

**Evaluates**: Privacy understanding, technical knowledge of memorization, regulatory awareness (GDPR, CCPA), remediation strategies.

**Detailed Walkthrough**:
- **Immediate assessment**: Scope the issue. How reproducible is the memorization? Is it limited to specific types of prompting? How many individuals' PII can be extracted? Is the memorized data publicly available on the web (lower legal risk) or from private/scraped sources (higher legal risk)?
- **Legal analysis**: Under GDPR, individuals have the right to erasure. If EU residents' personal data is memorized, you may be obligated to remove it. Under CCPA, California residents have similar rights. Consult with legal immediately on notification obligations and regulatory reporting requirements.
- **Technical remediation**: Short-term -- implement output filtering to detect and redact PII patterns (names, emails, phone numbers, addresses, SSNs) in all model outputs. This is a band-aid but reduces immediate risk. Medium-term -- investigate the memorization mechanism. Is it concentrated in specific training data subsets? Can you identify and remove the problematic training examples? If so, consider targeted fine-tuning to "unlearn" memorized content. Long-term -- implement differential privacy in future training runs to mathematically bound memorization. Improve data preprocessing to redact PII before training. Implement membership inference testing as part of the pre-launch evaluation suite.
- **Communication**: Notify affected individuals if identifiable (required by many privacy regulations). Publish a transparency statement describing the issue, its scope, and remediation steps. Engage with the researchers constructively -- responsible disclosure should be rewarded, not punished.
- **Prevention for future models**: PII detection and redaction in training data preprocessing. Differential privacy during training. Memorization testing as part of pre-launch evaluation (probe the model with known training examples, measure verbatim reproduction rates). Canary testing (insert known canary strings in training data and test if the model can reproduce them).

**Common Pitfalls**:
- Downplaying the issue ("it is public data anyway") without considering legal obligations
- Not understanding the difference between memorization of public vs private data
- Proposing only technical fixes without addressing the legal and communication aspects
- No prevention strategy for future models
- Ignoring the research community's role in identifying these issues (adversarial relationship vs collaborative)

---

## Scenario 5: Disinformation at Scale

**Question**: "Your intelligence team discovers that a state actor is using your AI platform to generate and distribute political disinformation at scale -- thousands of unique, convincing articles per day, each slightly different to evade content matching. What do you do?"

**Evaluates**: Adversarial thinking at scale, detection system design, policy and legal considerations, coordination with external stakeholders.

**Detailed Walkthrough**:
- **Detection and investigation**: How did you discover this? If through behavioral signals (unusual account creation patterns, API usage spikes, coordinated posting), document the indicators of compromise. Analyze the generated content: topics, target audiences, languages, distribution patterns. Confirm attribution with reasonable confidence before taking action -- misattributing state-sponsored activity has geopolitical consequences.
- **Immediate containment**: Suspend the identified accounts and API keys. Do not alert the actors (quiet suspension -- if they know your detection methods, they will adapt). Preserve all evidence (content generated, account metadata, API logs) for potential law enforcement cooperation.
- **Detection system design**: Behavioral signals -- anomalous API usage patterns (volume, timing, content similarity across accounts). Content signals -- AI-generated text detection (though this is an arms race), thematic clustering (many accounts producing content on the same narrow topics), temporal patterns (content spiking around specific events). Network signals -- coordinated behavior (accounts created at similar times, from similar IPs, posting to similar targets). Build a classifier that combines behavioral, content, and network signals. Accept that this is adversarial -- the actors will adapt, so the detection system needs continuous updating.
- **Policy and legal**: Review terms of service -- does the current policy explicitly prohibit coordinated inauthentic behavior and state-sponsored influence operations? If not, update it. Consult legal on law enforcement cooperation (in the US, report to FBI; in the EU, report to relevant national authorities). Consider mandatory disclosure to affected platforms where the content was distributed.
- **External coordination**: Share indicators (but not intelligence methods) with other AI companies through industry groups (Tech Against Terrorism, GIFCT). Notify relevant election authorities if the content targets elections. Brief the board of directors on the incident and the company's exposure.
- **Long-term prevention**: Rate limiting and usage monitoring for API access. Require identity verification for high-volume API users. Build watermarking or provenance tracking into generated content. Invest in AI-generated content detection research. Publish a transparency report on coordinated inauthentic behavior detected and actions taken.

**Common Pitfalls**:
- Treating this as purely a technical problem (it requires policy, legal, and geopolitical judgment)
- Taking public action before confirming attribution
- Not preserving evidence for law enforcement
- Designing detection only for the current attack pattern without considering adversarial adaptation
- Ignoring the coordination aspect -- this affects the entire AI ecosystem, not just your platform

---

## Quick Reference: Scenario-to-Skill Mapping

| Scenario | Primary Skill | Secondary Skill | Difficulty |
|----------|--------------|-----------------|------------|
| 1. Biased Hiring AI | Bias Detection & Remediation | Crisis Communication | Hard |
| 2. Content Moderation | Safety System Design | Human Reviewer Well-being | Medium-Hard |
| 3. Medical Advice Jailbreak | Defense in Depth | Red-Teaming | Hard |
| 4. PII Memorization | Privacy & Compliance | Technical Remediation | Medium-Hard |
| 5. State-Sponsored Disinformation | Adversarial Thinking | Policy & Legal Judgment | Hard |

---

## Tips for Evaluating Answers

- **Nuance test**: Does the candidate acknowledge genuine trade-offs, or do they give simplistic answers? "Just do not be biased" and "bias is inevitable" are both red flags. Look for: "Here is the trade-off, here is how I would navigate it, and here is what I am still uncertain about."
- **Stakeholder awareness**: Does the candidate consider multiple stakeholders (affected individuals, the company, regulators, the public, employees)? Answers that optimize for only one stakeholder miss the complexity.
- **Technical grounding**: Can the candidate connect ethical principles to concrete engineering actions? "We should be fair" is philosophy. "We should measure demographic parity across protected classes and set a maximum disparity threshold of 1.2x" is engineering.
- **Crisis management instinct**: For incident response scenarios, does the candidate triage (stop the harm first) before investigating root causes? The order matters.
- **Regulatory literacy**: Does the candidate know that regulations exist and roughly what they require? They do not need to cite specific articles, but they should know the EU AI Act classifies hiring AI as high-risk and that GDPR grants a right to erasure.
- **Humility**: The best candidates in responsible AI acknowledge what they do not know and what remains genuinely hard. Anyone claiming to have solved AI ethics is either selling something or has not thought deeply enough.
