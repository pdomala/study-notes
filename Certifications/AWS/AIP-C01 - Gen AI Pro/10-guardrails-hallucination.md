[← Back to index](README.md)

# 🛡️ 10 · Guardrails & hallucination management

> Exam domain: D3

## Key points
- Policy map: hallucination/off-topic in RAG → **contextual grounding check** (grounding + relevance) · PII → **sensitive info filter** (+ regex) · competitor names → **word filter** (exact match) · banned subject → **denied topic** · jailbreak/injection → **content filter: prompt attacks** · mathematical verification against rules → **Automated Reasoning checks** · Titan/Nova Canvas images → **watermark detection**.
- Apply inline via **InvokeModel/Converse** (input + output) or **ApplyGuardrail API** = evaluate any text **without calling a model** (also for non-Bedrock models). Apps reference **guardrail ID + published version** (draft only for testing).
- Filter **strength is inverse to confidence**: HIGH strength blocks LOW+MEDIUM+HIGH confidence; LOW strength blocks HIGH only. Action **block** vs **detect** (log only). PII mask on **output only** → the model still sees the PII.
- Tuning: domain terms in the grey zone → **lower strength + denied topics + word filters** (layering); keep **prompt attack HIGH (input only)**. Grounding/relevance start at **0.7** (0.6–0.8 sweet spot; 0.95 blocks paraphrases).
- KPI = **intervention rate = InvocationsIntervened / Invocations** (`AWS/Bedrock/Guardrails`); drill down by GuardrailPolicyType / ContentSource / Version; spikes → **anomaly detection alarm**. Test safely in **detect mode** + a curated **ApplyGuardrail** suite. Retry rate rising → **false positives**. Per-context strictness → **Lambda routing across guardrail versions**.
- Hallucination types: **factual** · **contextual** (off-prompt) · **conceptual** (invented entity) · **attribution** (wrong source) · **temporal** (outdated) · **bias-induced**. Mitigate in layers: model, input, output guardrails + KB grounding.

## Notes
### Why FMs need guardrails — 4 risks & layered protection
- Traditional ML = **labelled** data, **single task**; FMs = **unlabelled** internet-scale pre-training, **many tasks**, **open-ended / non-deterministic** output (same question → different answers) → new risk class
- Value dimensions: **creativity** (needs authenticity / labelling of AI vs human work) · **productivity** (e.g. Amazon Q for developers, secure code) · **connectivity** (chatbots, meeting summaries — keep privacy)
- **4 GenAI challenges**:
  - **Undesirable/irrelevant topics** → stay in scope (airline assistant answers bookings/baggage/gates, refuses stock tips or restaurant recs)
  - **Toxicity & safety** → harmful/offensive output (e.g. trashing a competitor) → brand + trust damage
  - **Privacy** → PII (name, address, DOB) must be **redacted**, e.g. in customer-service summaries
  - **Hallucination** → confident wrong answers → detect and filter before the user
- Customer questions to design for: hallucination, **traceability**, traditional **authN/authZ**, staying on-topic, **prompt injection / content leaking**
- Weak links in a basic GenAI app: **application code** (poor logic, no input validation) · **prompt** (hijacking/injection) · **model** (misinterpreting context)
- Layers: **model-level** (FM provider's built-in protection, baseline only — e.g. Claude trained to say "I'm not sure"; defines knowledge boundaries) · **prompt-level** (system instructions: stick to task & context) · **application-level** (monitor input in + output out → **Bedrock Guardrails** sits here)
- Hallucination framing of the same layers: **model-level** → **input** guardrails (first filter before the model: harmful content, denied topics, word blocks, **PII masking**) → **output** guardrails (between model and user: **block**, return **custom message**, or **redact**)
- Guardrail flow: input prompt → **input check** → model inference → **output check** (Automated Reasoning sits in the output layer) = defense in depth
- Guardrails block **more harmful content than FM native protection alone** and filter more hallucinations in **RAG/summarisation**; behaviour is **consistent across models** (model-agnostic); evaluates **every request AND response in real time**

### Guardrails policy types (what each one does)
- **Content filters** → **Hate, Insults, Sexual, Violence, Misconduct, Prompt Attack**; **text and image** (configured separately; multimodal toxicity); per-category **strength NONE / LOW / MEDIUM / HIGH** (= threshold/aggressiveness); action **block** or **detect** (allow + log)
- **Prompt attack filter** → catches injection + jailbreak ("ignore your system instructions…", persona override); applies to **input only**
- **Denied topics** → defined in **natural language** (e.g. block finance + healthcare on a tech-adviser bot; no investment advice for a bank); **intent-based**, applies to prompts or responses
- **Word filters** → **exact match** words/phrases + **profanity**; org-specific restricted terms (e.g. **competitor names**)
- **Sensitive information filters** → **block or mask (redact)** PII (probabilistic detection) + **custom regex** (internal IDs); credit cards detected via **Luhn**-valid patterns
- **Contextual grounding check** → hallucination filter: **grounding** (supported by the reference source) + **relevance** (answers the query); for **RAG, summarisation, info extraction**; with Bedrock KB the **retrieved results are the reference source**
- **Automated Reasoning checks** → **mathematically/logically verify** response accuracy against policy rules → precision-critical apps; detects factual errors with verifiable reasoning
- **Image content filter / multimodal guardrails** → toxic images, per-category strength; **image filtering GA** (inappropriate visuals, cross-modal consistency); **video on roadmap**
- **Custom guardrails** → industry restrictions, brand safety, regulatory compliance, custom keyword/topic filters
- Separate Bedrock safeguard — **watermark detection** → tells whether an image was generated by **Titan Image Generator / Nova Canvas** (returns a **confidence score**) vs human-made
- Guardrails protect inputs and outputs **excluding reasoning content blocks**
- Content-filtering areas: harmful content · sensitive info · quality (spam, gibberish, off-topic) → **Bedrock** built-ins · **Lambda** custom/event-driven · **SageMaker** custom models · **Comprehend** (sentiment, PII, moderation) · **Rekognition** (image/video moderation)
- Multiple guardrails per use case, **reusable across FMs**; design guidelines: layered security + audit, performance/cost, UX (clear errors, graceful fallbacks, feedback)

### Applying guardrails — integration paths, versions & testing
- Path 1: **InvokeModel / Converse** with a guardrail → evaluates the **incoming prompt AND outgoing response**
- Path 2: **ApplyGuardrail API** → independent evaluation of any text **without calling a model** (document processing, **non-Bedrock models**) — set **source = INPUT or OUTPUT**
- Knowledge Bases: pass `guardrailId` + version in **RetrieveAndGenerate** config
- Lifecycle **create → test → deploy**: edit the **working DRAFT** → **create version** (immutable snapshot) → code (boto3) passes **guardrailIdentifier + guardrailVersion**; draft only for testing
- Versions = config snapshots → **A/B testing, rollback, tuning history**
- Console test window: pick guardrail draft + model, run prompt → **trace** shows which category intervened and why
- **Batch evaluation** → upload a **CSV of prompts** to test blocked vs passed at scale
- Test with sample prompts before prod; **monitor guardrail invocation metrics in CloudWatch**
- Lessons: a **system prompt alone can be bypassed** by a persona-override prompt; the guardrail still blocks because it checks **both directions** — if input slips through, the **output** filter catches it · masking PII on **output only** → model still sees the PII in the prompt

### Filter strength vs confidence; false positives vs false negatives
- Content filters are **classifiers** returning confidence **NONE** (clearly safe) → **LOW / MEDIUM** (gray zone) → **HIGH** (clearly harmful)
- **Filter strength** (NONE/LOW/MEDIUM/HIGH) ≠ confidence; strength is **inversely related** to the confidence needed to block

| Filter strength | Blocks confidence | Allows |
| --- | --- | --- |
| NONE | nothing | everything |
| LOW | HIGH | NONE, LOW, MEDIUM |
| MEDIUM | MEDIUM, HIGH | NONE, LOW |
| HIGH | LOW, MEDIUM, HIGH | NONE only |

- Domain language lives in the gray zone: healthcare "cutting / self-harm assessment", gaming "kill streak / headshot", legal "assault charges" → the classifier isn't wrong, it just **doesn't understand your domain**
- **False positive** (type 1) = legit content blocked → user drop-off, support tickets, lost trust · **False negative** (type 2) = harmful content passes → brand damage, compliance violations
- No perfect setting → **risk tolerance**: financial = tolerate more **FPs** (safety first) · creative writing = tolerate more **FNs** (usability first)
- **Confidence gap**: configuring is step one; operating consistently in production is the real challenge
- **Iterative tuning lifecycle**: **Monitor** (something's wrong) → **Debug** (what) → **Tune** (thresholds/policies) → **Test** (before users) → **Deploy** (new version) → repeat — ongoing ops, not one-time config

### Debugging & policy layering (composition)
- Read the **trace**: filter type, category, confidence, action
- Example: "best approach for dealing with self-inflicted wounds" → VIOLENCE **MEDIUM** confidence + MEDIUM strength → blocked
- Fix 1: VIOLENCE → **LOW** → clinical query passes ✅ but "make injuries look like an accident" (MEDIUM) now passes ❌ → **one threshold can't solve both FP and FN**
- **Policy composition** = tuning is about composing policies, not finding the perfect threshold; each layer covers the others' blind spots:
  - **Content filter (LOW)** → broad probabilistic catch of clearly harmful
  - **Denied topics** → **intent-based**, not keywords
  - **Word filters** → **exact-match** safety net for known bad phrases
  - **System prompt** → behavioural defense at the model
- Keep **prompt attack at HIGH** (input only) even when relaxing other filters
- Healthcare example: start strict (all filters **HIGH** in/out; prompt attack input only; denied topics **prescription writing, medical diagnosis, physical injury planning**; word filters recreational drugs, buying meds online without prescription, self-medication) → all 8 clinical prompts blocked (VIOLENCE MEDIUM) → violence + misconduct **LOW**, hate/insult/sexual **MEDIUM**, prompt attack **HIGH** → all pass
- Strict-only blocked legit clinical prompts **and missed the diagnosis request** (no content-filter confidence) → layered (LOW + denied topics + words) got all right; v1 (all LOW) beat v2 (violence/misconduct MEDIUM)
- Test-suite gap: "recreational drugs" blocked (word match) but "marijuana" passed (misconduct LOW only) → add **marijuana to the word filter** or as a **denied-topic example phrase**, rerun suite

### Contextual grounding thresholds (RAG)
- Two independent thresholds **0–1**: **grounding** (factually supported by source) · **relevance** (answers the query) → tune **independently**
- Grounded but irrelevant → correct facts, wrong question (ibuprofen manufacturing process) · relevant but ungrounded → answers the question with fabricated facts (side effects not in source)
- Too high (e.g. **0.95**) → blocks valid **paraphrased** answers · too low (e.g. **0.3**) → hallucinations pass
- Sweet spot **0.6–0.8**; **start at 0.7** for both; legal/compliance higher, general Q&A lower
- Strategy: monitor InvocationsIntervened filtered by **contextual grounding policy on OUTPUT** → sample blocks → hallucinated or just paraphrased? → adjust each threshold → compare in detect mode
- On KB apps, contextual grounding filters **over 75% of hallucinations**; Guardrails on KB apps also give denied topics, PII redaction, content/word filters and **prompt injection/jailbreak** protection
- End-to-end RAG quality beyond Guardrails → **RAGAS**

### Guardrails metrics, dashboards & alarms
- CloudWatch namespace **`AWS/Bedrock/Guardrails`**
- Metrics: **Invocations** · **InvocationsIntervened** · **InvocationLatency** (perf impact) · **InvocationClientErrors** (config issues) · **InvocationServerErrors** · **InvocationThrottles** (quota → request increase) · **TextUnitCount** (cost/billing units)
- **Intervention rate = InvocationsIntervened / Invocations** = **most important tuning indicator**
  - High → too restrictive (FPs) · Low → too permissive *or* genuinely clean traffic
  - Spike → attack, concept/content drift, or bad config change
  - Tells you *something* is wrong, not *what* to fix
- Diagnostic dimensions: **GuardrailPolicyType** (which policy fires most) · **GuardrailContentSource** (INPUT vs OUTPUT vs both) · **GuardrailVersion** (did a recent change cause it?)
- Example: healthcare bot, intervention rate **40%**, **85%** from content policy, source = **input** → violence filter blocking medical terms → loosen it (e.g. 12/30 = 40%)
- Dashboard, 3 questions: **Blocking too much?** (intervention rate by policy type) · **Is something changing?** (rate over time) · **What's it costing?** (latency + TextUnitCount → cost-to-performance ratio)
- Alarms: **high intervention rate** (over X → overblocking, review thresholds) · **intervention spike** (InvocationsIntervened + **CloudWatch anomaly detection** → attack/content shift) · **latency breach** → establish a **baseline first**, alarm relative to it

### Validating changes — detect mode & test-driven suites
- Policy actions: **BLOCK** · **MASK** (sensitive info) · **NONE** = **detect mode**
- Detect mode evaluates but **takes no action**; returns the full trace (confidence, filter type, would-be outcome) → find FP/FN, compare thresholds side by side, zero user impact
- Set action NONE **per policy, on input AND output** (content filters, each denied topic, word filters)
- Trace examples: action NONE, **detected = true**, LOW confidence + LOW strength → no action · oxycodone prescription → misconduct MEDIUM + prescription topic detected, **no action**
- **Test-driven pipeline**: curated dataset of legit + harmful prompts with **expected results** → run through **ApplyGuardrail API** → compare actual vs expected
  - Expected pass but INTERVENED → **false positive** · expected block but NONE → **false negative**
  - Iterate → new version → rerun → deploy only when **pass rate meets acceptance criteria**
  - ⚠️ ApplyGuardrail **consumes policy units = cost** → factor into test design
- A simple test helper wraps `bedrock-runtime.apply_guardrail` (guardrail ID, version, source, content) + CloudWatch metric pull

### Judging correctness & adaptive guardrails
- CloudWatch shows intervention **frequency**; only user/app signals tell you if it was **correct**
- **User behaviour**: **retry rate** (rephrasing after a block → FPs; track similar-intent consecutive requests in a session) · **session abandonment after intervention** (UX cost of FP; correlate intervention event with session + timestamp) · **feedback/escalation** (thumbs, report button)
- **Trace-derived**: **confidence distribution over time** (most blocks on LOW = too tight, only HIGH = too soft; drift) · **policy type breakdown over time** (custom metric on GuardrailPolicyType vs baseline)
- **Human-in-the-loop**: sample blocked + allowed, humans label → **precision, recall, F1** → inform thresholds; accurate but **doesn't scale**
- **LLM-as-a-judge**: second model judges whether the guardrail decision was correct in domain context → scales to thousands, consistent; **complements** humans (not a replacement); the judge may need its own guardrail (e.g. **Claude 3 Haiku** via InvokeModel returns verdict + confidence 0–1 + reason)
- **Adaptive guardrails**: multiple guardrail **versions** per risk profile (relaxed vs strict) + lightweight classifier (e.g. **Lambda**) routes by context (clinical → relaxed, general chat → strict)
  - **Not native** → custom pattern; costs latency, money, complexity → usually overkill; for high-volume apps with costly FPs in context-heavy domains
  - ≠ **Intelligent prompt routing** (routes between **models of the same family** for cost/quality, not between guardrails)

### Hallucinations — definition & causes
- Output that looks **plausible/authoritative** but is **inaccurate, inconsistent or fabricated**, presented as fact
- Root cause: LLMs **predict the next probable token** from probability distributions — no true understanding, can't tell fact from possibility, equal confidence for right and wrong
- Training data has a **cutoff date** → no real-time knowledge, stale info presented as current
- Same probabilistic nature is **useful for creativity** (ads, creative writing) but harmful where accuracy matters (**healthcare, finance, legal**)
- Even low-stakes hallucinations **erode user trust** → manage via detection, mitigation and user communication
- **4 causes**:
  - **Training data issues** → outdated, biased, incomplete, contradictory → model fills gaps instead of admitting ignorance
  - **Model limitations** → pattern recognition, not reasoning; can't verify facts or separate correlation from causation
  - **Token size constraints** → fixed **context window** forces truncation; long outputs lose earlier details → contradictions
  - **Nuanced language** → metaphors, sarcasm, abstract/causal reasoning → misinterpretation
- Problems RAG fixes (facts, outdated info, unreliable sources, terminology confusion) → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Why RAG)

### Six hallucination types
- **Factual** → wrong verifiable facts (dates, stats, events)
- **Contextual** → correct in isolation but **irrelevant to / contradicts the prompt or context** (asked about cloud computing → talks about weather clouds)
- **Conceptual** → **invents** non-existent concepts/entities/relationships (e.g. a fake AWS service mashing up real ones) — most deceptive
- **Attribution** → wrong citations, misquotes, idea credited to the wrong person/org (academic, journalism)
- **Temporal** → knowledge-cutoff issues: outdated info as current, "upcoming" for already released → fix with **RAG**
- **Bias-induced** → output reflects **training-data bias**, skewed representation of groups

### Hallucination mitigation & patterns by application type
- No single fix → **layered safety net**: **inference settings** → **prompt engineering** → **RAG** (real-time grounding) → **evaluation** → **guardrails**
- Other mitigations: **self-consistency / CoT**, **domain fine-tuning**, **human review**, **feedback loops**
- Prompt-level hallucination fixes (default "I don't know" output, Claude guidance, ReAct) → See [03 · Prompt engineering](03-prompt-engineering-prompt-management.md) · KB citations → [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Knowledge Bases overview)
- **Internal tools** (summarisers, code assistants) → ground in company docs via **Kendra / OpenSearch**; key guardrails = **sensitive info filters + contextual grounding**; employee **feedback loop** to flag hallucinations
- **Customer-facing** (banking chatbot, product descriptions) → multi-layer: Bedrock FM + Guardrails → **validation layer vs trusted sources** → **CloudWatch alarms** on grounding-check detections
- **Regulated** (healthcare/finance/legal) → **SageMaker** custom domain evaluation models + **stricter confidence thresholds** + **mandatory human review** + **CloudTrail** audit trail
- Service roles: **Bedrock** = FMs + guardrails · **SageMaker** = custom/evaluation models · **Lambda** = real-time validation · **CloudWatch** = monitoring/alerts
- Hallucination management is **ongoing** across the lifecycle, not an afterthought

## Exam triggers
- "Detect responses not supported by source docs / RAG answers drifting / block hallucinated KB answers" → **contextual grounding check**
- "Response accurate but off-topic to the user's question" → grounding check **relevance** score failed
- "RAG blocks valid paraphrased answers" → **lower grounding threshold** (sweet spot 0.6–0.8, start 0.7)
- "Mask SSN / DOB in prompts and responses" → **sensitive information filter** (custom IDs → **regex**)
- "Redact rather than block" → sensitive info filter action = **mask**
- "Block competitor names / specific words" → **word filter** (exact match)
- "No investment advice / block a subject described in plain English" → **denied topic**
- "Harmful request with no content-filter signal (diagnosis, prescription)" → **denied topic** · "specific term slips through" → **word filter**
- "Jailbreak / prompt injection / ignore previous instructions / persona override" → **content filter – prompt attacks** (system prompt alone isn't enough)
- "Prompt attack filter when relaxing others" → keep **HIGH**, **input only**
- "Mathematically verify response is correct vs policy rules" → **Automated Reasoning checks**
- "Check content without invoking a model / docs pipeline / non-Bedrock model" → **ApplyGuardrail API**
- "Was this image AI-generated by Titan / Nova Canvas?" → **watermark detection**
- "App must use a stable guardrail config" → publish a **version**, pass **guardrail ID + version**
- "Legit domain terms blocked (healthcare/gaming/legal) / too many false positives" → **lower content filter strength + add denied topics/word filters** (not remove filter, not retrain model)
- "Monitor / test new thresholds without blocking users" → **detect mode (action NONE)**
- "Regression test before deploy" → **curated dataset + ApplyGuardrail**, gate on pass rate
- "Is the guardrail tuned right?" → **intervention rate** · "input or output?" → **GuardrailContentSource** · "which policy?" → **GuardrailPolicyType** · "after a change?" → **GuardrailVersion**
- "Sudden spike in blocks" → **anomaly detection alarm on InvocationsIntervened**
- "Users retry/rephrase after blocks" → **false positives**
- "Judge guardrail correctness at scale" → **LLM-as-a-judge** (+ HITL sample for precision/recall/F1)
- "Different strictness per context" → **custom adaptive routing (Lambda + multiple guardrail versions)**, not intelligent prompt routing
- "Outdated info presented as current" → **temporal** hallucination → fix with **RAG**
- "Invented service/entity" → **conceptual** · "wrong citation" → **attribution**
- "Regulated industry hallucination controls / audit trail" → **CloudTrail** + **mandatory human review**

## Flashcards
<details>
<summary><b>Q:</b> An app needs different guardrail strictness per context (clinical vs general chat). What approach fits?</summary>

A custom adaptive pattern: a lightweight classifier (e.g. Lambda) routes to multiple guardrail versions. It is not native and is different from intelligent prompt routing.

*D3 · Adaptive guardrails*
</details>

<details>
<summary><b>Q:</b> A document-processing pipeline (or a non-Bedrock model) must be checked by guardrails without invoking a Bedrock model. What should be used?</summary>

The ApplyGuardrail API, specifying source = INPUT or OUTPUT.

*D3 · ApplyGuardrail API*
</details>

<details>
<summary><b>Q:</b> A precision-critical app must mathematically/logically verify responses against policy rules. Which Guardrails feature?</summary>

Automated Reasoning checks.

*D3 · Automated Reasoning*
</details>

<details>
<summary><b>Q:</b> Distinguish conceptual and attribution hallucinations.</summary>

Conceptual = invents non-existent concepts/entities (e.g., a fake AWS service); attribution = wrong citations, misquotes or ideas credited to the wrong source.

*D5 · Conceptual vs attribution*
</details>

<details>
<summary><b>Q:</b> Which categories and modalities do Guardrails content filters cover?</summary>

Hate, insults, sexual, violence, misconduct and prompt attacks, on both text and images, with per-category strength and block or detect actions.

*D3 · Content filter scope*
</details>

<details>
<summary><b>Q:</b> A RAG app's answers drift away from the source documents. Which Bedrock Guardrails safeguard detects this?</summary>

Contextual grounding check (output layer) — scores grounding against the reference source (retrieved KB results) and relevance to the query.

*D3 · Contextual grounding check*
</details>

<details>
<summary><b>Q:</b> Asked about cloud computing, the model answers about weather clouds. What type of hallucination is this?</summary>

Contextual — correct in isolation but irrelevant to or contradicting the prompt/context.

*D5 · Contextual hallucination*
</details>

<details>
<summary><b>Q:</b> A harmful request (e.g. asking for a diagnosis or prescription) triggers no content-filter signal, and a specific term like 'marijuana' slips through. What should be added?</summary>

A denied topic for intent-based requests like diagnosis/prescription; add the specific term to a word filter (or as a denied-topic example phrase).

*D3 · Denied topic vs word*
</details>

<details>
<summary><b>Q:</b> How can new guardrail thresholds be tested on real traffic without affecting users?</summary>

Detect mode: set the policy action to NONE (per policy, on input and output); it evaluates and returns the full trace without blocking.

*D5 · Detect mode*
</details>

<details>
<summary><b>Q:</b> A healthcare chatbot's legitimate clinical prompts are blocked by the violence filter. What is the recommended fix?</summary>

Lower the content filter strength (e.g. to LOW) and add denied topics and word filters to cover the gaps (policy layering), rather than removing the filter.

*D3 · Domain false positives*
</details>

<details>
<summary><b>Q:</b> A Guardrails content filter detects VIOLENCE at MEDIUM confidence and the filter strength is HIGH. Is the content blocked?</summary>

Yes. HIGH strength blocks LOW, MEDIUM and HIGH confidence; MEDIUM blocks MEDIUM and HIGH; LOW blocks only HIGH.

*D3 · Filter strength*
</details>

<details>
<summary><b>Q:</b> What are the four GenAI challenges that guardrails address?</summary>

Undesirable/irrelevant topics, toxicity & safety, privacy (PII), and hallucination.

*D3 · Four GenAI risks*
</details>

<details>
<summary><b>Q:</b> How does risk tolerance shape guardrail tuning for a financial app versus a creative writing app?</summary>

Financial: tolerate more false positives (safety first). Creative writing: tolerate more false negatives (usability first).

*D3 · FP vs FN tolerance*
</details>

<details>
<summary><b>Q:</b> Responses are factually grounded but off-topic to the user's question. Which part of the grounding check flags this?</summary>

The relevance score of the contextual grounding check.

*D3 · Grounding relevance score*
</details>

<details>
<summary><b>Q:</b> A RAG app's contextual grounding check at 0.95 refuses accurate answers that exist in the knowledge base. Why, and what is the recommended starting point?</summary>

Accurate paraphrases score below a strict threshold. Lower it; sweet spot 0.6-0.8, start at 0.7 for grounding and relevance (too low, e.g. 0.3, lets hallucinations pass).

*D3 · Grounding threshold*
</details>

<details>
<summary><b>Q:</b> A response scores grounding 0.85 and relevance 0.3. What does that mean?</summary>

The response is factually supported by the source but does not answer the question.

*D3 · Grounding vs relevance*
</details>

<details>
<summary><b>Q:</b> Which CloudWatch dimension shows whether guardrail interventions happen on input or output, and which show the policy and the version?</summary>

GuardrailContentSource (input vs output); GuardrailPolicyType (which policy); GuardrailVersion (whether a change caused it).

*D4 · Guardrail dimensions*
</details>

<details>
<summary><b>Q:</b> If a malicious prompt slips past input filters, can a guardrail still stop harmful content?</summary>

Yes - guardrails evaluate both the incoming prompt and the outgoing response, so the output filter can block it.

*D3 · Guardrail directions*
</details>

<details>
<summary><b>Q:</b> What are the two ways to apply Bedrock Guardrails?</summary>

Inline with InvokeModel/Converse (evaluates prompt and response), or standalone via the ApplyGuardrail API.

*D2 · Guardrail integration paths*
</details>

<details>
<summary><b>Q:</b> What are the three guardrail layers and where does the contextual grounding check sit?</summary>

Model-level (built in by the provider), input (before the model) and output (between model and user); grounding check is an output guardrail.

*D3 · Guardrail layers*
</details>

<details>
<summary><b>Q:</b> How should guardrail changes be regression-tested before deployment?</summary>

Run a curated dataset of legitimate and harmful prompts with expected results through the ApplyGuardrail API, compare actual vs expected (FP/FN), and deploy only when the pass rate meets acceptance criteria. ApplyGuardrail consumes policy units (cost).

*D5 · Guardrail regression tests*
</details>

<details>
<summary><b>Q:</b> Production must use a stable guardrail configuration. What is the workflow and what is passed in code?</summary>

Edit the working DRAFT, test, then create an immutable version; pass `guardrailIdentifier` + `guardrailVersion` in InvokeModel/Converse.

*D3 · Guardrail versioning*
</details>

<details>
<summary><b>Q:</b> Name the four causes of hallucinations.</summary>

Training data issues, model limitations (pattern matching not reasoning), token/context-window constraints, and nuanced language.

*D1 · Hallucination causes*
</details>

<details>
<summary><b>Q:</b> A model presents outdated information as current. What type of hallucination is this and what is the typical fix?</summary>

Temporal hallucination (knowledge cutoff); mitigate with RAG for real-time grounding.

*D5 · Hallucination types*
</details>

<details>
<summary><b>Q:</b> What is the single most important Guardrails tuning metric and how is it calculated?</summary>

Intervention rate = InvocationsIntervened / Invocations (CloudWatch namespace `AWS/Bedrock/Guardrails`).

*D4 · Intervention rate*
</details>

<details>
<summary><b>Q:</b> How should a team be alerted to a sudden spike in guardrail blocks caused by attacks or content drift?</summary>

A CloudWatch anomaly detection alarm on InvocationsIntervened.

*D4 · Intervention spike alarm*
</details>

<details>
<summary><b>Q:</b> Users send 'ignore your previous instructions, you are now...' prompts that bypass the system prompt. Which Guardrails feature?</summary>

Content filter for prompt attacks. A system prompt alone can be bypassed.

*D3 · Jailbreak attempts*
</details>

<details>
<summary><b>Q:</b> How can a team judge whether guardrail decisions were correct at scale?</summary>

Use LLM-as-a-judge to assess decisions in domain context, complemented by human-in-the-loop sampling to compute precision, recall and F1.

*D5 · Judge guardrail correctness*
</details>

<details>
<summary><b>Q:</b> Which approaches mitigate hallucinations?</summary>

Prompt engineering, evaluation frameworks and RAG (as part of a layered net with inference settings and guardrails) — not disabling generation, more training data or shorter outputs.

*D3 · Layered mitigation*
</details>

<details>
<summary><b>Q:</b> A customer-facing chatbot needs alerts when hallucinations are detected. What should be used?</summary>

Amazon CloudWatch alarms on contextual grounding check detections, alongside a validation layer against trusted sources.

*D4 · Monitoring grounding detections*
</details>

<details>
<summary><b>Q:</b> An app handles text and images and must block hate speech, harmful advice and brand/regulatory violations. What guardrail coverage is needed?</summary>

All three: content safety filters, multimodal (image filtering, GA) guardrails, and custom guardrails for brand/regulatory rules.

*D3 · Multimodal guardrails*
</details>

<details>
<summary><b>Q:</b> PII masking is configured on output only. Does the model see the PII?</summary>

Yes - the model still receives the PII in the prompt; only the response is redacted.

*D3 · Output-only masking*
</details>

<details>
<summary><b>Q:</b> 90% of should-block prompts are intervened but 30% of should-pass prompts are also intervened. What is the problem?</summary>

Over-blocking (false positives).

*D5 · Over-blocking*
</details>

<details>
<summary><b>Q:</b> SSNs and dates of birth must be masked in both prompts and responses, plus a custom internal ID format. What should you configure?</summary>

Guardrails sensitive information filters (block or mask PII) with a custom regex for the internal ID.

*D3 · PII masking*
</details>

<details>
<summary><b>Q:</b> What is the advantage of layering low-strength content filters, denied topics, word filters and a system prompt?</summary>

Each layer compensates for the others' blind spots: content filter = broad probabilistic, denied topics = intent-based, word filters = exact match, system prompt = model behaviour.

*D3 · Policy layering*
</details>

<details>
<summary><b>Q:</b> When relaxing Guardrails content filters, how should the prompt attack filter be configured?</summary>

Keep it at HIGH; it applies to input only.

*D3 · Prompt attack filter*
</details>

<details>
<summary><b>Q:</b> A healthcare GenAI app needs stricter hallucination controls. What is recommended?</summary>

SageMaker custom domain evaluation models, stricter confidence thresholds, mandatory human review and a CloudTrail audit trail.

*D3 · Regulated industry pattern*
</details>

<details>
<summary><b>Q:</b> After a guardrail change, retry rate rises 25% while abandonment is flat. What does this indicate?</summary>

Users are rephrasing after false-positive blocks.

*D5 · Retry rate signal*
</details>

<details>
<summary><b>Q:</b> How can you tell whether an image was generated by Titan Image or Nova Canvas?</summary>

Bedrock watermark detection, which returns a confidence score.

*D3 · Watermark detection*
</details>

<details>
<summary><b>Q:</b> A bank must block competitor names and also prevent any investment advice. Which Guardrails features?</summary>

Word filter (exact match) for competitor names; denied topic for investment advice.

*D3 · Word vs denied topic*
</details>

---
[← Back to index](README.md)
