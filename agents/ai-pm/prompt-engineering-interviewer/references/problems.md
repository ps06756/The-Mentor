# Prompt Engineering & LLM Architecture -- Scenario Bank & Detailed Walkthroughs

---

## How to Use This Bank

Each scenario below includes the question, what the interviewer is evaluating, a detailed walkthrough at the Senior AI Engineer level, and common pitfalls. Adapt the depth and scope to the candidate's experience level.

---

## Scenario 1: Structured Data Extraction from Medical Records

**Question**: "Design a prompt pipeline for extracting structured data (JSON) from unstructured medical records. The records include OCR'd handwritten notes, lab results, and doctor's narratives."

**Evaluates**: Multi-stage pipeline design, handling noisy input, output schema design, validation, domain-specific challenges.

**Detailed Walkthrough**:
- **Understand the input**: Medical records are messy. OCR'd handwritten notes have misspellings, abbreviations ("pt" for patient, "hx" for history), and formatting inconsistencies. Lab results are semi-structured but vary by lab. Doctor's narratives are free text with medical jargon.
- **Pipeline design**: Stage 1 (preprocessing) -- clean OCR artifacts, segment the document into sections (demographics, vitals, labs, medications, clinical notes) using a classification prompt with few-shot examples of each section type. Stage 2 (extraction) -- for each section type, use a specialized prompt. Lab results get a structured extraction prompt with explicit JSON schema. Clinical notes get a narrative extraction prompt that identifies diagnoses, symptoms, and treatment plans. Stage 3 (normalization) -- standardize extracted values: convert units, map medication names to RxNorm codes, map diagnoses to ICD-10 codes. Stage 4 (validation) -- check cross-field consistency (a medication listed under "current medications" should not contradict the allergy list), validate ranges (heart rate of 800 is impossible), and flag low-confidence extractions.
- **Prompt structure for extraction**: System message defines the role ("You are a medical data extraction system"), the output schema (explicit JSON with field descriptions and types), and constraints ("Extract only what is explicitly stated. Do not infer or assume."). User message contains the section text. Few-shot examples cover: clean input, OCR errors, abbreviations, and missing data.
- **Evaluation**: Build a golden set of 500 manually annotated records. Measure per-field extraction accuracy, with separate metrics for each section type. Track false positives (extracted something that was not there) and false negatives (missed something that was). For critical fields (allergies, medications), target 98%+ recall.
- **Edge cases**: Contradictory information in the same record, records in multiple languages, records with redacted sections, extremely long records that exceed context windows (need a chunking strategy that respects medical record structure).

**Common Pitfalls**:
- Trying to extract everything in one prompt pass
- Not handling OCR errors (misspellings, garbled text)
- No validation layer -- trusting the LLM output blindly
- Using generic extraction prompts instead of section-specific ones
- Ignoring token limits for long medical records

---

## Scenario 2: Debugging RAG Retrieval Quality

**Question**: "Your RAG system returns irrelevant documents 30% of the time. Users are complaining that the AI gives answers based on wrong information. Debug and fix it."

**Evaluates**: Systematic debugging methodology, understanding of retrieval systems, root cause analysis, measurement-driven improvement.

**Detailed Walkthrough**:
- **Build the debugging dataset**: Sample 200 queries where users flagged irrelevant results. For each query, record: the user query, the retrieved chunks (with relevance scores), the generated response, and the user's feedback. Manually label each retrieved chunk as relevant or irrelevant.
- **Categorize failure modes**: Analyze the 200 samples and categorize failures. Common categories: (1) Chunking issues (40%) -- chunks are too large and contain multiple topics, so a chunk about "Topic A" gets retrieved because it also mentions "Topic B" in passing. (2) Query-document vocabulary gap (25%) -- users ask "How do I fix a slow database?" but the document says "Optimizing PostgreSQL query performance." (3) Embedding model limitations (20%) -- the embedding model does not understand domain-specific terminology. (4) Stale or duplicate content (10%) -- the index contains outdated documents or multiple versions of the same document. (5) Other (5%) -- ambiguous queries, no relevant document exists.
- **Targeted fixes**: For chunking issues -- switch from fixed-size chunking (500 tokens) to semantic chunking (split on paragraph boundaries, topic shifts). Add chunk overlap (100 tokens) to preserve context at boundaries. For vocabulary gap -- add a query rewriting step: use an LLM to expand the user query into 2-3 alternative phrasings before embedding. For embedding model -- evaluate domain-specific embedding models or fine-tune on your corpus. For stale content -- implement a deduplication pipeline and a freshness score that deprioritizes old documents.
- **Additional improvements**: Add a reranking step using a cross-encoder model (e.g., Cohere Rerank or a fine-tuned cross-encoder). Cross-encoders are more accurate than bi-encoders for relevance scoring but too slow for initial retrieval. Use bi-encoder for top-100 retrieval, then cross-encoder to rerank to top-5.
- **Measurement**: After each fix, measure retrieval recall@5 and precision@5 on the debugging dataset. Also measure end-to-end: does the generated response quality improve? Track the irrelevance complaint rate in production as the ultimate metric.

**Common Pitfalls**:
- Changing the LLM model instead of fixing retrieval (the LLM is only as good as the context it receives)
- Making multiple changes at once without measuring each independently
- Not building an evaluation dataset before making changes
- Ignoring chunking strategy (the most common and impactful failure mode)
- Over-indexing on embedding model choice when the real issue is data quality

---

## Scenario 3: Evaluating Creative Writing AI

**Question**: "Design an evaluation framework for a creative writing AI that helps users write fiction (short stories, novel chapters). How do you measure quality?"

**Evaluates**: Understanding of evaluation methodology for subjective tasks, multi-signal evaluation design, practical trade-offs between evaluation approaches.

**Detailed Walkthrough**:
- **Define quality dimensions**: Creative writing quality is multi-dimensional. Define 5-7 specific dimensions: (1) Coherence -- does the text flow logically? Are there contradictions? (2) Engagement -- is it interesting to read? Does it hold attention? (3) Style consistency -- does it maintain the requested tone/voice throughout? (4) Originality -- does it avoid cliches and predictable patterns? (5) Instruction following -- does it match the user's prompt (genre, length, characters, plot points)? (6) Character voice -- do characters sound distinct and believable? (7) Technical quality -- grammar, spelling, sentence variety.
- **Three-tier evaluation framework**: Tier 1 (automated, every change) -- LLM-as-judge scoring. Design a detailed rubric for each dimension (1-5 scale with specific criteria for each score). Run against a benchmark set of 100 diverse prompts. Calibrate the LLM judge against 500 human judgments to understand its biases and reliability per dimension. Automated metrics like perplexity and repetition rate as sanity checks. Tier 2 (weekly, during development) -- human evaluation. Recruit 5 annotators (ideally writers or avid readers). Evaluate a stratified sample of 50 outputs across genres and complexity levels. Compute inter-annotator agreement (target: Cohen's kappa above 0.7). Use comparative evaluation ("Which of these two outputs is better and why?") in addition to absolute scoring. Tier 3 (monthly, for releases) -- user A/B testing. Show users outputs from the current and candidate model. Measure preference rate, completion rate (does the user use the AI output or rewrite from scratch?), retention (do users come back?), and qualitative feedback.
- **Benchmark design**: The benchmark set should cover: multiple genres (sci-fi, romance, mystery, literary fiction), multiple formats (short story, chapter, dialogue, description), multiple difficulty levels (simple scene, complex multi-character interaction, plot twist), and adversarial cases (contradictory instructions, extremely long prompts, prompts requesting problematic content).
- **Regression detection**: Any prompt change must pass all three tiers sequentially. Tier 1 is a gate -- if automated scores drop, do not proceed. Tier 2 catches nuances that automated evaluation misses. Tier 3 is the final validation with real users.

**Common Pitfalls**:
- Relying solely on BLEU/ROUGE (these measure surface similarity, not quality)
- Not calibrating LLM-as-judge against human judgments
- Using a single overall quality score instead of dimensional evaluation
- Not investing in diverse, representative benchmark prompts
- Ignoring user behavioral signals (the ultimate measure of quality is whether users find the output useful)

---

## Scenario 4: Prompt System for Automated Code Review

**Question**: "Design a prompt system that reviews pull requests and identifies bugs, security vulnerabilities, and style issues. It should work across Python, TypeScript, and Go."

**Evaluates**: Multi-language handling, prompt chaining for complex analysis, false positive management, integration with developer workflows.

**Detailed Walkthrough**:
- **Architecture**: Do not try to review an entire PR in one prompt. Break it into stages: Stage 1 (diff parsing) -- parse the PR diff to extract changed files, added lines, removed lines, and surrounding context. Stage 2 (per-file analysis) -- for each changed file, run a focused review prompt. Stage 3 (cross-file analysis) -- analyze interactions between changed files (e.g., a function signature change that affects callers in other files). Stage 4 (aggregation and prioritization) -- combine findings, deduplicate, prioritize by severity.
- **Per-file review prompt design**: System message: "You are a senior code reviewer. Review the following code change and identify: (1) bugs (logic errors, off-by-one, null pointer), (2) security vulnerabilities (injection, auth bypass, data exposure), (3) performance issues, (4) style/readability concerns. For each finding, specify the line number, severity (critical/high/medium/low), category, and a brief explanation. Only flag issues you are confident about -- false positives erode trust." Include language-specific context in the system message (e.g., for Python: common pitfalls with mutable default arguments, asyncio patterns). Provide 3 few-shot examples per language: one with real bugs, one with style issues only, and one clean code (to calibrate against false positives).
- **False positive management**: This is the make-or-break issue. Developers will disable a review tool that generates too many false positives. Target: false positive rate below 15%. Strategies: (a) confidence thresholds -- only surface findings above 0.8 confidence, (b) severity-based routing -- show critical/high findings inline, batch medium/low findings in a summary, (c) feedback loop -- let developers dismiss findings with a reason, use dismissals to improve prompts, (d) per-repository calibration -- learn which types of findings a team cares about.
- **Evaluation**: Build a benchmark of 200 PRs with known bugs, vulnerabilities, and clean code. Measure precision (of flagged issues, how many are real?), recall (of known issues, how many are caught?), and developer satisfaction (quarterly survey). Run the system against the benchmark after every prompt change.

**Common Pitfalls**:
- Trying to review the entire PR in one prompt (token limits, quality degradation)
- Not addressing false positive rate (the number one reason developers disable automated review tools)
- Language-agnostic prompts that miss language-specific patterns
- No feedback loop from developer dismissals
- Ignoring context (a "bug" in a test file is different from a bug in production code)

---

## Scenario 5: Token Optimization for a High-Volume Classification System

**Question**: "You have a prompt-based content classification system that processes 1 million items per day. Each classification costs $0.002 in tokens. The CFO wants to cut costs by 60% without reducing accuracy. How do you approach this?"

**Evaluates**: Cost optimization strategies, model selection, caching, prompt compression, system architecture for efficiency.

**Detailed Walkthrough**:
- **Current cost analysis**: 1M items/day at $0.002 each equals $2,000/day or $60,000/month. The 60% reduction target is $24,000/month, meaning the cost needs to drop to $0.0008 per item or find ways to classify fewer items with the LLM.
- **Strategy 1 -- Model routing (expected savings: 30-40%)**: Not all items need the most capable (expensive) model. Analyze the classification difficulty distribution. Typically 60-70% of items are easy (clear-cut category), 20-25% are medium, and 10-15% are hard. Route easy items to a smaller/cheaper model (Claude Haiku, GPT-3.5 Turbo). Route medium to a mid-tier model. Route hard items to the full model. Use a lightweight classifier (or even a rules engine) as the router.
- **Strategy 2 -- Prompt compression (expected savings: 15-25%)**: Audit the current prompt. Remove redundant instructions. Replace verbose few-shot examples with shorter, more targeted ones. Use structured output format to reduce output tokens. If the system message is 2,000 tokens, challenge every line -- can the same instruction be given in fewer tokens?
- **Strategy 3 -- Caching (expected savings: 10-30% depending on input diversity)**: Implement semantic caching. If a new item is sufficiently similar to a recently classified item, reuse the classification. Use an embedding model to compute similarity and set a threshold. For exact duplicates or near-duplicates, this is free savings.
- **Strategy 4 -- Batch processing (expected savings: 5-15%)**: If latency is not critical, batch multiple items into a single prompt. Classify 5-10 items per API call. This amortizes the system message tokens across multiple classifications. Test carefully -- batch size can affect accuracy.
- **Strategy 5 -- Distillation (longer-term, expected savings: 70-80%)**: Use the LLM to generate labels for a training set, then fine-tune a small traditional ML model (or fine-tune a small LLM) for the classification task. The fine-tuned model handles routine cases, and the large LLM handles only the hardest cases.
- **Implementation order**: Start with prompt compression (fastest, no infrastructure changes). Then add caching. Then implement model routing. Distillation is a longer project but has the highest ROI. Measure accuracy after each change using the evaluation set.

**Common Pitfalls**:
- Suggesting only one optimization strategy instead of a layered approach
- Cutting prompt quality to save tokens without measuring the accuracy impact
- Not considering caching (many classification workloads have significant input overlap)
- Ignoring model routing (the single highest-impact cost optimization for most systems)
- Proposing fine-tuning without acknowledging the upfront cost and maintenance burden

---

## Quick Reference: Scenario-to-Skill Mapping

| Scenario | Primary Skill | Secondary Skill | Difficulty |
|----------|--------------|-----------------|------------|
| 1. Medical Record Extraction | Pipeline Design | Domain-Specific Prompting | Medium |
| 2. RAG Debugging | Systematic Debugging | Retrieval Architecture | Hard |
| 3. Creative Writing Evaluation | Evaluation Framework Design | Subjective Quality Measurement | Hard |
| 4. Automated Code Review | Multi-Stage Prompting | False Positive Management | Medium-Hard |
| 5. Token Cost Optimization | Cost Optimization | System Architecture | Medium-Hard |

---

## Tips for Evaluating Answers

- **Pipeline thinking test**: Does the candidate break complex tasks into sub-tasks with distinct prompts, or do they try to solve everything in one giant prompt? Pipeline thinking is the hallmark of production experience.
- **Evaluation-first mindset**: Does the candidate talk about evaluation early in their design, or only when asked? Engineers who have shipped prompt systems know that evaluation infrastructure is the foundation.
- **Specificity test**: Can they name specific chunking strategies, embedding models, reranking approaches, or evaluation metrics? Vague answers like "we would use RAG" without specifics suggest theoretical rather than practical knowledge.
- **Trade-off articulation**: For every design choice, can they articulate what they are giving up? "I would use a cross-encoder reranker" is good. "I would use a cross-encoder reranker for accuracy, accepting the latency cost of 200ms per query, which is acceptable for our use case" is great.
- **Production awareness**: Do they think about monitoring, versioning, rollback, and cost? Or do they design systems as if they will run once and never change?
