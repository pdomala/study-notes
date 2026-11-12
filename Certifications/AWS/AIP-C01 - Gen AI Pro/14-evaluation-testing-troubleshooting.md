[← Back to index](README.md)

# 🧪 14 · Evaluation, testing & troubleshooting

> Exam domain: D5

## Key points
- Metrics: **ROUGE** = summarisation (recall) · **BLEU** = translation (precision) · **BERTScore** = semantic similarity · **perplexity** = next-token fit. Subjective quality at scale → **LLM-as-a-judge**.
- **RAGAS**: faithfulness = answer vs context · answer relevancy = answer vs question · context precision = relevant chunks ranked top · context recall = contexts vs **ground truth**.
- **Bedrock Evaluations**: automatic, human (own team / AWS-managed), LLM-as-a-judge, RAG (retrieve-only and retrieve-and-generate). Use the evaluation API as a **CI/CD gate**.
- **Nova LLM-as-a-Judge**: pairwise binary preference (A vs B or tie), run as a **SageMaker AI training job + recipe**, ~3% bias vs humans; winrate = Model B preferred; **95% CI overlapping 0.5 = inconclusive**; high stderr → more eval data.
- **AgentCore Evaluations**: observability = what happened, evaluation = how well. **Online** = live sessions from CloudWatch Logs after idle timeout (default 15 min), ≤ 10 evaluators, ≤ 5 log groups · **On-demand Evaluate API** = regression suite / CI/CD gate. Levels **TOOL_CALL · TRACE · SESSION**. Wrong tool → Tool Selection Accuracy · bad args → Tool Parameter Accuracy · goal → Goal Success Rate · domain policy → **custom evaluator**.
- **SageMaker Model Monitor**: data quality (drift) · model quality (needs ground truth) · bias · feature attribution (SHAP). Rebalance data → **Data Wrangler**. Endpoint fails at creation = config, at runtime = resources.
- Troubleshooting map: context overflow → chunking/summarisation · irrelevant answers → embeddings, chunk size, metadata filters, reranker · format drift → schema validation + templates · throttling → backoff / PT / cross-Region · hallucination → grounding check + KB · missing traces → CloudWatch **Transaction Search** + IAM.

## Notes

### Evaluation approaches & benchmark datasets
- Lifecycle position of evaluation → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (GenAI application lifecycle)
- Align evaluation with **business objectives**
- **Human evaluation** — coherence, relevance, factuality; **gold standard** but slow + expensive
- **Benchmark datasets:** **GLUE** (language understanding) · **SuperGLUE** (harder, compositional) · **SQuAD** (question answering) · **WMT** (machine translation)
- **Automated metrics** — fast, scalable, but miss nuance
- Best practice: combine automated metrics **with human evaluation**
- RAG evaluation must cover **retrieval and generation**; practise **continuous evaluation & refinement**

### Automated text metrics & their limits
- **Perplexity** — how well the model predicts the next token
- **ROUGE** — **recall**-oriented overlap vs reference → **summarisation** (+ translation)
- **BLEU** — **precision**-oriented n-gram overlap → **machine translation**
- **BERTScore** — embedding-based **semantic similarity** vs reference (catches paraphrases)
- **F1** — classification / NER; **exact match** also used programmatically
- Limits: perplexity / BLEU / accuracy / rule-based metrics are **statistical** → fail when tasks need **subjective judgment**, **contextual understanding**, **alignment with business requirements**
- **LLM-as-a-judge** = use an LLM's reasoning to evaluate other models **flexibly and at scale** → nuanced assessments aligned with human preferences; biggest gain for **summarisation, content generation, agents**

### Amazon Bedrock Evaluations & Model Evaluation API
- **Model Evaluation API** → `CreateEvaluationJob` / `GetEvaluationJob`; **automatic** (built-in metrics: **accuracy, robustness, toxicity**) or **human** evals; fits **CI/CD** (repeatable model comparison)
- Evaluates: FMs, Marketplace, custom, imported, **prompt routers**, Provisioned Throughput models
- **Model evaluation** types:
  - **LLM-as-a-judge** (correctness, completeness, harmfulness)
  - **Programmatic / automatic** (BERTScore, F1, exact match; accuracy, robustness, toxicity)
  - **Human** (own team or **AWS-managed workforce**)
- **RAG / Knowledge Base evaluations**: **Retrieval only** (context relevance, coverage) · **Retrieve & Generate** (end-to-end, custom pipeline or KB) → need **expected texts/responses (ground truth)**

### RAG evaluation with RAGAS
- **RAGAS** = open source, pairs with the **Retrieve API**; inputs: question, contexts, generated answer, ground truth

| Metric | Measures | Inputs |
| --- | --- | --- |
| **Faithfulness** | Answer factually consistent with **context** (hallucination check) | answer + context |
| **Answer relevancy** | Answer addresses the **question**; penalises incomplete/redundant | question + context + answer |
| **Context precision** | Relevant chunks **ranked at top** (signal-to-noise) | question + ground truth + contexts |
| **Context recall** | Retrieved context covers the **ground truth** | **contexts + ground truth** |

- All scores **0–1**, higher = better
- Guardrails contextual grounding only checks individual responses → end-to-end RAG quality → **RAGAS**

### Amazon Nova LLM-as-a-Judge (SageMaker AI)
- **Pairwise binary overall preference** judge: Model A vs Model B side by side → pick better or **tie** → aggregate → **win rate + CI** · low-latency general preference, automated scoring
- Validated to stay **impartial across model families** (no architectural bias toward Nova)
- Training: **SFT on human-preference datasets** → **RL stages** → multi-annotator review of response pairs → calibrated to **broad human consensus** · data: knowledge, creativity, coding, maths, specialised domains, **toxicity**; **90+ languages**
- Bias study: **10,000+ human-preference judgments**, **75 third-party models** → **~3% aggregate bias** (+ = favours Nova, − = opposite; 95% CI)
- Benchmarks (positionally swapped responses reconciled); strong on **chatbot** evals (PPE):

| Benchmark | Nova Judge | Meta J1 8B | Nova Micro |
| --- | --- | --- | --- |
| JudgeBench | 0.45 | 0.42 | 0.37 |
| PPE | 0.68 | 0.60 | 0.60 |
| Eval Bias | 0.76 | – | 0.56 |
| LLMBar | 0.67 | – | 0.55 |
| CodeUltraFeedback | 0.64 | – | – |

- Workflow: **1 Dataset prep** (prompt + response_A + response_B, **JSONL**) → **2 SageMaker recipe** (strategy, judge model, inference settings) → **3 Run as SageMaker training job** (pre-built Nova containers) → **4 Analyse** (preference distribution, win rates, CIs)
- Metrics:
  - Core: `a_scores`, `b_scores`, `ties`, `inference_error` (judge couldn't produce a valid judgment)
  - Confidence: `winrate` = share of valid comparisons where **Model B** preferred · `lower_rate` / `upper_rate` = **95% CI** · `score` (≈ B wins, customisable)
  - `*_stderr`: smaller = reliable · larger = **need more eval data**
  - **CI overlapping 0.5 = inconclusive** → evaluate further
- Setup: quota ≥ **1 × ml.g5.12xlarge** for SageMaker training (Service Quotas) · IAM role trusted by **`sagemaker.amazonaws.com` + `bedrock.amazonaws.com`** (SageMakerFullAccess, S3FullAccess, BedrockFullAccess) · sample repo `aws-samples/amazon-nova-samples` → customization/SageMakerTrainingJobs/Amazon-Nova-LLM-As-A-Judge
- Example setup: candidates **Qwen2.5 1.5B on a SageMaker endpoint** (HuggingFaceModel) vs **Claude 3.7 Sonnet on Bedrock** (cross-Region `us.` model ID) · **SQuAD** questions → `llm_judge.jsonl` → S3
- Execution: SageMaker **PyTorch estimator** (`training_recipe`, `image_uri`, `instance_type`, `output_path`) → `estimator.fit(inputs={"train": evalInput})` → fully managed: loads judge → processes dataset → computes metrics → writes to **S3**
- `plot_nova_judge_results` → **6 panels**: score distribution · winrate + 95% CI · preference pie · A vs B counts · winrate gauge · summary table
- Use cases: **model selection pipelines**, **continuous evaluation (catch regressions)**, agentic/domain systems, stakeholder reporting · cleanup: delete the SageMaker endpoint + notebook instance

### AgentCore Evaluation — why & architecture
- Agents return **structurally correct but functionally wrong** output → no errors, no alarms, but trust/business/safety degrade (**silent failures**)
- Traditional metrics say "system up, latency low, no errors" — can't say whether intent was understood, answer correct, decision safe, right tool chosen
- **Observability** = "what happened" (security-camera footage: traces, tool calls, latency, errors, token usage, flow) · **Evaluation** = "how well" (grading against a rubric: correctness, helpfulness, safety, tool use) · **need both**
- Manual review = **slow** (weeks to spot), **resource-intensive** (domain experts, DS, QA), **reactive** (after complaints), **incomplete** (can't cover every session/tool call)
- AgentCore Evaluation = continuous sampling + automated **built-in (13) / custom / third-party** evaluators (modes **on-demand, batch or online**; pre-deploy + continuous) + structured LLM-as-a-Judge + dashboards/alerts → detect silent failures before customers, trace-level root cause, safe model upgrades
- **Online** mode: agent session (OTEL/OpenInference) → spans to **CloudWatch** → session **inactive X min (default 15)** → **sampling rules** (percentage + conditional filters) → evaluators run on **Bedrock** → scores to **CloudWatch Metrics (EMF)** + **Logs** (explanations) → dashboards/alerts · **no agent code changes**
- **On-demand** mode: **Evaluate API** on static test cases, captured traces, regression suites, side-by-side model comparison → CI/CD, block PR merges
- Components: **Agent Runtime** (Strands/LangGraph, OTEL) → **CloudWatch Log ingestion** (data source for online) → **Control plane** (create evaluators + online eval configs: definitions, rating scales, model configs, sampling) → **Execution layer** (after inactivity, filters + sampling, Bedrock scoring) → **Metrics & monitoring** (EMF, dashboards, alarms, trace viewer in Observability)
- Works for agents on **AgentCore Runtime and outside AgentCore**; results shown in the agent's CloudWatch dashboard

### Evaluation levels, reason-first judge & 13 built-in evaluators
- Levels:
  - **TOOL_CALL** = which tool, correct parameters, used appropriately (e.g. `web_search` vs `calculate_trip_budget`)
  - **TRACE** = one agent turn incl. all its tool calls, inputs/outputs (e.g. **faithfulness** to retrieved docs)
  - **SESSION** = multi-turn end-to-end: **goal success**, multi-step workflows, safety across the session, escalation/emotional calibration
- **Reason-first LLM-as-a-Judge**: analyses context → interprets behaviour → compares to criteria → **step-by-step reasoning** → **then** numeric/categorical score
  - Why: **lower scoring variance**, transparent/auditable artifacts, reproducibility, debuggable evaluator decisions
  - Reasoning stored in **CloudWatch Logs** for every scored trace

| Category | Evaluators |
| --- | --- |
| Response quality | **Correctness** (factual) · **Faithfulness** (grounded in sources) · **Helpfulness** · **Response Relevance** · **Conciseness** · **Coherence** · **Instruction Following** |
| Safety & risk | **Refusal** · **Harmfulness** · **Stereotyping** |
| Task completion | **Goal Success Rate** |
| Tool usage | **Tool Selection Accuracy** · **Tool Parameter Accuracy** |

- IDs look like `Builtin.Helpfulness`, `Builtin.Correctness`
- Examples: supply chain right latency/errors but wrong forecast model → **Tool Selection Accuracy** · mortgage nice tone, outdated regs → **Correctness + Faithfulness** · support misses frustration/cancel signal → **Helpfulness**

### Preparing traces & online evaluation config
- Evaluation only as good as the traces → instrument with **OTEL or OpenInference** (Strands, LangGraph, custom runtimes)
- Spans needed: user messages, agent responses, **tool calls + parameters**, tool results, intermediate reasoning
- Minimum per span: **session ID (persistent across conversation)**, **trace ID** per turn, **span ID**, tool metadata (params + outputs), model response text → missing → sessions **skipped or mis-scored**
- Config needs **log group name(s)** + **service name** (consistent label for agent/system)
- Multi-agent: one rule can watch **up to 5 log groups**; spans correlated by **shared session IDs**
- **IAM execution role**: trust policy lets AgentCore Evaluation assume it · permissions = **read source log groups**, **write to `/aws/bedrock-agentcore/evaluations/...`**, **invoke Bedrock judge models** · reusable across configs, can't modify agents
- Validate first: inspect sample traces in CloudWatch — if you can follow query → reasoning → tools → response, the evaluator can too
- Online config = which sessions, which evaluators, which log groups, when a session is complete
- Console fields: **name**, **"start evaluating upon creation" (default = disabled)**, description, **session idle timeout** · data source = AgentCore Runtime agent + **endpoint** (auto-selects its log group) or extra log groups · evaluators (built-in + custom) · **filters + sampling rate**
- Pick evaluators by purpose/risk: support → helpfulness, instruction following, correctness · supply chain → tool selection/parameter accuracy · mortgage/compliance → correctness, faithfulness, refusal
- **Up to 10 evaluators per config**; start with built-ins, add custom later
- Sampling: **percentage** (start **1–5%**, e.g. 10%) · **conditional** (keywords "refund/cancel/urgent", specific tool, latency > 3 s, tool error) · **hybrid** (5% all + 100% financial/safety)
- Session idle timeout: default **15 min**, range **1–1440 min** · short for brief consumer bots, long for loan processing/multi-step workflows → avoids scoring mid-conversation
- First results **30–45 min** after enabling (waits for inactivity + processing); runs until disabled

### Interpreting evaluation results & alarms
- **CloudWatch Metrics (EMF)** = aggregated scores per evaluator → trends, regressions, **percentiles** (lower tail, key for Refusal/Harmfulness)
- **CloudWatch Logs** = per-trace LLM-as-a-Judge reasoning → **why** it changed
- Filter logs by **`evaluatorId`**, **`label`**, score range, **`sessionId`**, **`traceId`** → lowest scores usually reveal the pattern in 1–2 traces
- Loop: **metric drop → evaluator reasoning → inspect trace (Observability console) → fix**; results reference sessionId/traceId/spanId
- Alarms: stricter on correctness (< 0.8), looser on helpfulness (< 0.6), **zero tolerance on Harmfulness/Stereotyping**, multi-metric alarms · notify via SNS/email/ChatOps/PagerDuty
- Alarm on **average AND minimum** score in a window → catches long-tail outliers
- Fixes: prompts, tool schemas, KB indexing, guardrails, model choice, parameter extraction, escalation rules → re-check scores
- Example: travel helpfulness 0.88 → 0.69; judge shows fallback to generic `web_search` instead of budget tool → prompt update had de-emphasised specialised tools

### Custom evaluators
- Use for domain rules/policies (investment guidelines, clinical protocols, brand tone, churn signals, deprecated legal info)
- 3 parts: **evaluator instructions** (prompt template = rubric, reason-first) · **rating scale** (numeric 0.0–1.0 or labels with definitions) · **model config** (`modelId`, `inferenceConfig` max tokens/temperature/top-p/stop sequences, `additionalModelRequestFields`)
- Good instructions: **clear, context-aware, structured (step-by-step), grounded (objective criteria), deterministic**
- Scale choice: **dashboards → numerical** · **alerts → categorical** · **human review → hybrid** · both types can coexist in one config
- Judge settings: **low temperature 0.0–0.2**, higher max tokens for reasoning, tight stop sequences · most pick **Claude Sonnet 4.5** (any supported model, incl. custom fine-tuned)
- Create via **`CreateEvaluator`** (CLI, Boto3, SDK, IaC) → returns **evaluatorId** → behaves like built-ins (online, CI/CD, dashboards), no re-instrumenting
- Iterate: scores clustering around **0.5** → unclear instructions · long explanations → ambiguous criteria · misread domain terms → refine instructions/thresholds/params/examples

### On-demand Evaluate API & CI/CD gating
- For changes **before** deploy: model switch, prompt/policy edits, tool schema changes, KB refresh, new capabilities
- Regression suite (typically **25–200 scenarios**): critical paths, edge cases, safety cases, **past failure reproductions**, high-value workflows
- Request: **one `evaluatorId` per call** (can score **many traces**) + `evaluationInput` (spans/traces) + optional `evaluationTargets` (specific trace/span IDs)
- Response: **`value`** (score) · **`label`** · **`explanation`** · **`context`** (session/trace/span IDs) · **`tokenUsage`** (cost modelling)
- Compare **Baseline v1 vs Candidate v2** on the same suite (e.g. new model too generous with refunds → regression)
- CI/CD: PR → Evaluate API on high-priority cases → below threshold → **block PR** + surface the judge's reasoning = "quality firewall"

### Agent evaluation case studies & best practices
- Pattern: **ops metrics normal, eval scores drop**
  - **Supply chain**: Tool Selection 0.94 → 0.78, param accuracy stable → **tool choice, not parsing**; prompt over-emphasised seasonality → explicit tool rules + custom evaluator
  - **Mortgage multi-agent**: doc agent misclassifies bank statements (threshold change) + Q&A cites 2022 rules (KB index priorities) → cross-agent visibility
  - **Travel**: prompt simplification removed tool-usage rules → defaulted to web_search → restore rules + examples + custom "Budget Accuracy"
  - **SaaS support**: helpfulness 4.4 → 2.1, goal success 92% → 67% → stale **routing API**
  - **E-commerce**: helpfulness steady, **goal success 78% → 61%** → stale inventory API → value of **multi-metric** evaluation
- Sampling: start **1–5% random** → add **conditional** for refunds, escalation keywords, medical/compliance, failure-prone tools → **100% sparingly** (new feature rollout, safety-critical, pre-launch) · typical blend: 3% random + 100% high-impact + conditional safety triggers
- Cost drivers: sampling volume, **number of evaluators**, trace length, reasoning length → lower sampling on stable workflows, fewer/relevant evaluators, concise numeric scales, **hierarchical evaluation** (more evaluators on critical sessions)
- Evaluators drift → **quarterly review** (score distribution, outlier reasoning, consistency, business-rule alignment)
- Scale: consistent service names/log groups/session IDs · **dedicated config per agent** · cross-agent only when needed · **team-owned evaluator sets**
- Governance baselines: Correctness ≥ 0.85 · Helpfulness ≥ 0.70 · Goal Success ≥ 0.80 · Tool Selection ≥ 0.90 · shared dashboards · **evaluation sign-off** before major updates
- Rhythm: weekly trend reviews · monthly anomaly investigations · quarterly evaluator revisions · CI/CD gating

### Testing methods & quality signals
- MLOps lifecycle & CI/CD → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (MLOps, CI/CD & success metrics)
- Test-driven pipeline (expected results, FP/FN, pass-rate gate), versioning, app-level signals (retry rate, abandonment, feedback), HITL vs LLM-as-a-judge → See [10 · Guardrails](10-guardrails-hallucination.md) (Validating changes; Judging correctness & adaptive guardrails)
- Quality monitoring: prompt-structure effects, relevance/coherence metrics, user feedback, compare FMs, track fine-tuning impact

### Bedrock troubleshooting
- **Integration issues → fix**:
  - **API** (SDK mismatch, wrong endpoint) → verify creds/perms, request format, retries
  - **Auth** (permission errors, role assumption, cross-account) → identity + **resource-based policies**, **role trust policy**, ARNs
  - **Network** (VPC) → **VPC endpoints**, NACLs/SGs, DNS
  - **Performance** (latency, timeouts, **throttling**) → CloudWatch metrics, client timeouts + graceful fallback, **service quotas**
  - **SDK** → **exponential backoff**, client cleanup, changelog, version compatibility
- Model not available (Region), throttling, model EOL → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Model lifecycle; Inference & deployment options) · data leaving the account → [01 · Fundamentals](01-fundamentals-genai-strategy.md) (lifecycle & deployment considerations)

### SageMaker Model Monitor, drift & bias tools
- **Model Monitor** 5 types:
  - **Data quality** → input stats drift vs baseline, missing values, type mismatch
  - **Model quality** → accuracy/precision/recall, **needs ground-truth labels**
  - **Model bias** → fairness across groups (DPPL, DI), pre + post-training
  - **Feature attribution** → **SHAP** drift, explainability
  - **Custom** → own metrics, scripts, schedules, preprocessing
- Setup flow: **enable data capture** (request, response, or both) → **baseline** stats + constraints from training data → **schedule** monitoring
- Drift detection 3 stages: **data quality monitoring** (profile from training data vs live) → **model quality monitoring** (capture actuals, e.g. forecast vs real demand a week later) → **drift evaluation** (rules/thresholds → **CloudWatch** notifications; drift data in **S3** → **Athena + QuickSight**)
- Endpoint monitoring: latency, errors, utilisation, invocations; operational/invocation/health metrics (depend on endpoint type)
- Bias vs explainability, **SageMaker Clarify**, **Data Wrangler** rebalancing, fairness by design → See [12 · Governance](12-governance-compliance-responsible-ai.md) (Bias & fairness — sources, mitigation, Clarify)

### SageMaker endpoint deployment troubleshooting
- Pre-deploy checklist: artifact format, IAM, VPC, quotas, container requirements, memory/storage
- **Artifacts**: correct `model.tar.gz` structure, serialization, size limits → optimise model, bigger instance
- **Infra**: instance quota / VPC limits → **Service Quotas**, VPC, SGs
- **Config**: missing deps, wrong entry point, env vars → **CloudWatch logs**
- Logs: **container** logs (init errors, runtime exceptions) + **instance** logs (utilisation, system errors)
- Performance (after deploy): slow inference, queuing, CPU/GPU, memory leaks → right-size, **auto scaling**, batch inference
- **Endpoint creation failure = configuration** (artifact/container/IAM) · **runtime failure = resource constraints** (memory) → debug with logs, health, smaller instance test
- IAM: missing perms, invalid trust relationship · VPC: SGs, NACLs, endpoints

## Exam triggers
- "Summarisation quality" → **ROUGE**
- "Translation" → **BLEU**
- "Semantic / paraphrase similarity" → **BERTScore**
- "Next-token fit" → **perplexity**
- "Gold standard but costly" → **human evaluation**
- "Repeatable model comparison in CI/CD" → **Model Evaluation API (CreateEvaluationJob)**
- "Answer contradicts / not supported by retrieved docs" → low **Faithfulness** (RAGAS metric; AgentCore built-in evaluator)
- "Answer off-topic/incomplete" → low **answer relevancy**
- "Retrieval missing needed facts" → low **context recall**
- "Relevant chunks ranked low" → low **context precision**
- "Compare two models on subjective quality at scale" → **LLM-as-a-judge (pairwise)**
- "Nova LLM-as-a-Judge runs where?" → **SageMaker AI training job + recipe**
- "winrate CI includes 0.5" → **no significant winner, add data**
- "High stderr" → **more evaluation examples**
- "winrate refers to" → **Model B** preferred
- "Catch quality regressions over time" → **scheduled/continuous judge evaluation**
- "Agent has low latency and no errors but users get wrong answers" → **AgentCore Evaluation** (observability alone can't detect)
- "Continuously score live production quality" → **online evaluation config**
- "Test prompt/model change before deploy / block PR" → **on-demand Evaluate API**
- "Right answer, wrong tool" → **Tool Selection Accuracy**
- "Wrong arguments to tool" → **Tool Parameter Accuracy**
- "Did user achieve goal across conversation" → **Goal Success Rate (SESSION)**
- "Organisation-specific policy / brand tone / compliance" → **custom evaluator** (instructions + rating scale + model config, `CreateEvaluator`)
- "Consistent, auditable judge scores" → **reason-first LLM-as-a-Judge, low temperature**
- "Why did score drop?" → **CloudWatch Logs** reasoning · "trend/alarm" → **CloudWatch Metrics (EMF)**
- "Sessions skipped or mis-scored" → missing **session ID / tool call spans** in OTEL/OpenInference traces
- "Evaluation scoring mid-conversation" → increase **session idle timeout** (default 15, 1–1440 min)
- "Multi-agent workflow scored as one session" → multiple log groups (**≤ 5**) + **shared session ID**
- "Reduce evaluation cost" → lower sampling %, fewer evaluators, conditional/hierarchical sampling — **not 100% sampling**
- "Evaluator scores cluster at 0.5" → **rewrite unclear instructions**
- "No results right after enabling" → normal: **30–45 min** (and config is **disabled by default** unless the start box is checked)
- "Evaluation IAM role needs" → **read source logs + write evaluation log group + invoke Bedrock model**
- "Production inputs drift from training" → **Model Monitor data quality**
- "Accuracy degrading" → **Model Monitor model quality (needs ground truth)**
- "Feature importance shift" → **feature attribution (SHAP)**
- "Endpoint fails at creation" → **config** (artifact/container/IAM) · "fails at runtime" → **resources** (memory) → check **CloudWatch logs**

## Flashcards
<details>
<summary><b>Q:</b> What evaluation types does Amazon Bedrock Evaluations offer for models and RAG?</summary>

Model: LLM-as-a-judge, programmatic (BERTScore, F1, exact match), human (own or AWS-managed workforce). RAG: Retrieval only, or Retrieve & Generate end-to-end.

*D5 · Bedrock evaluation types*
</details>

<details>
<summary><b>Q:</b> Match benchmarks to tasks: GLUE, SuperGLUE, SQuAD, WMT.</summary>

GLUE = language understanding; SuperGLUE = harder/compositional understanding; SQuAD = question answering; WMT = machine translation.

*D5 · Benchmark datasets*
</details>

<details>
<summary><b>Q:</b> Which built-in AgentCore evaluator checks that an answer is supported by retrieved documents, and which checks the user achieved their goal across a conversation?</summary>

Faithfulness (grounded in sources); Goal Success Rate (SESSION level).

*D5 · Built-in evaluators*
</details>

<details>
<summary><b>Q:</b> A team must detect bias and explain predictions, then rebalance an imbalanced dataset. Which SageMaker tools fit?</summary>

SageMaker Clarify for pre/post-training bias, Shapley values and PDPs; Data Wrangler to rebalance via random undersampling, random oversampling or SMOTE.

*D3 · Clarify vs Data Wrangler*
</details>

<details>
<summary><b>Q:</b> Relevant chunks are retrieved but ranked low. Which metric captures this?</summary>

Context precision (relevant chunks ranked at the top; signal-to-noise). Scores range 0–1, higher is better.

*D5 · Context precision*
</details>

<details>
<summary><b>Q:</b> Which inputs does the RAGAS context recall metric require, and what does it measure?</summary>

Contexts + ground truth; it measures whether the retrieved context covers the ground truth.

*D5 · Context recall inputs*
</details>

<details>
<summary><b>Q:</b> How can LLM-as-a-judge help catch quality regressions over time?</summary>

Run scheduled/continuous judge evaluations within model selection pipelines to detect regressions.

*D5 · Continuous evaluation*
</details>

<details>
<summary><b>Q:</b> A bank needs to score agents against organisation-specific policy adherence. What should it create and what are its parts?</summary>

A custom evaluator via `CreateEvaluator`: evaluator instructions (rubric), rating scale (numeric or categorical) and model config (modelId, inferenceConfig).

*D5 · Custom evaluator*
</details>

<details>
<summary><b>Q:</b> What is data drift and why monitor it?</summary>

A change in the input data distribution over time; it degrades model performance and is part of data monitoring in governance.

*D5 · Data drift*
</details>

<details>
<summary><b>Q:</b> A model is not available in the target account, or requests are being throttled. What should you check?</summary>

Model unavailable → Region availability (models are Region-limited). Throttling → service quotas.

*D5 · Deployment troubleshooting*
</details>

<details>
<summary><b>Q:</b> A SageMaker endpoint fails at creation versus at runtime. What is the typical cause of each?</summary>

Creation failure = configuration (artifact format, container, IAM); runtime failure = resource constraints (e.g. memory). Check container and instance logs in CloudWatch.

*D5 · Endpoint failures*
</details>

<details>
<summary><b>Q:</b> What permissions does the AgentCore Evaluation execution role need?</summary>

Read source log groups, write to `/aws/bedrock-agentcore/evaluations/...`, and invoke Bedrock judge models; its trust policy lets AgentCore Evaluation assume it.

*D3 · Evaluation IAM role*
</details>

<details>
<summary><b>Q:</b> What are the three AgentCore evaluation levels?</summary>

TOOL_CALL (tool choice and parameters), TRACE (one agent turn, e.g. faithfulness), SESSION (multi-turn end-to-end, e.g. goal success).

*D5 · Evaluation levels*
</details>

<details>
<summary><b>Q:</b> What are the AgentCore online evaluation limits for evaluators and log groups, and when do first results appear?</summary>

Up to 10 evaluators per config, up to 5 log groups per rule; first results 30-45 minutes after enabling.

*D5 · Evaluation limits*
</details>

<details>
<summary><b>Q:</b> What is the recommended sampling strategy for online evaluation cost and coverage?</summary>

Start with low-percentage random sampling (1-5%) plus conditional rules for high-risk workflows; use 100% sparingly. Lower cost with fewer evaluators and hierarchical evaluation.

*D4 · Evaluation sampling*
</details>

<details>
<summary><b>Q:</b> Nova judge results show a large <code>*_stderr</code>. What should be done?</summary>

Add more evaluation examples; smaller stderr means more reliable results.

*D5 · High stderr*
</details>

<details>
<summary><b>Q:</b> What is the best way to ensure quality of a state-of-the-art summarisation model, and what is its drawback?</summary>

Human evaluation – the gold standard for coherence, relevance, factuality, but slow and expensive. Best practice: combine with automated metrics.

*D5 · Human evaluation*
</details>

<details>
<summary><b>Q:</b> The winrate 95% confidence interval overlaps 0.5. How should this be interpreted?</summary>

Inconclusive - no significant winner; evaluate further with more data.

*D5 · Inconclusive result*
</details>

<details>
<summary><b>Q:</b> What does the <code>inference_error</code> metric indicate in Nova judge results?</summary>

The number of cases where the judge could not produce a valid judgment.

*D5 · `inference_error` metric*
</details>

<details>
<summary><b>Q:</b> What aggregate bias did Nova LLM-as-a-Judge show versus human annotations?</summary>

About 3% aggregate bias, measured over 10,000+ human-preference judgments across 75 third-party models.

*D5 · Judge bias*
</details>

<details>
<summary><b>Q:</b> What do Bedrock RAG/Knowledge Base evaluations require that automatic model evaluations don't?</summary>

Expected texts/responses (ground truth) so both retrieval and generation can be evaluated.

*D5 · KB evaluations*
</details>

<details>
<summary><b>Q:</b> A file in the S3 data source fails to ingest into a Bedrock KB. What should you check first?</summary>

File size (must be under 50 MB) and that the type is supported (.txt, .md, .pdf, .html, .doc/.docx, .csv, .xls/.xlsx).

*D5 · KB file rejected*
</details>

<details>
<summary><b>Q:</b> A team wants repeatable model comparisons in a CI/CD pipeline. Which Bedrock API?</summary>

Model Evaluation API – `CreateEvaluationJob` / `GetEvaluationJob` – with automatic (accuracy, robustness, toxicity) or human evaluation.

*D5 · Model Evaluation API*
</details>

<details>
<summary><b>Q:</b> What are the steps to set up SageMaker Model Monitor?</summary>

Enable data capture (request, response or both) -> create baseline statistics and constraints from training data -> schedule monitoring; drift results to CloudWatch and S3.

*D5 · Model Monitor setup*
</details>

<details>
<summary><b>Q:</b> Which SageMaker Model Monitor type detects production input drift, accuracy degradation, and feature-importance shift?</summary>

Data quality (input drift vs baseline); model quality (needs ground-truth labels); feature attribution (SHAP drift). Others: model bias and custom.

*D5 · Model Monitor types*
</details>

<details>
<summary><b>Q:</b> How do you evaluate a multi-agent workflow as one session in AgentCore Evaluation?</summary>

Point the rule at multiple log groups (up to 5) and correlate spans with a shared session ID.

*D5 · Multi-agent evaluation*
</details>

<details>
<summary><b>Q:</b> An online evaluation config was created but shows no results. What are the likely reasons?</summary>

Results take 30-45 min, and the config is disabled by default unless "start evaluating upon creation" was checked.

*D5 · No results yet*
</details>

<details>
<summary><b>Q:</b> A team wants to compare two candidate models on subjective quality at scale using Nova LLM-as-a-Judge. How is the evaluation run?</summary>

As a SageMaker AI training job using a recipe and the pre-built Nova container (PyTorch estimator), reading a JSONL dataset of prompt/response_A/response_B from S3 and writing metrics to S3.

*D5 · Nova judge execution*
</details>

<details>
<summary><b>Q:</b> What quota and IAM trust are needed to run Nova LLM-as-a-Judge?</summary>

At least 1 x ml.g5.12xlarge for SageMaker training (Service Quotas) and an IAM role trusted by `sagemaker.amazonaws.com` and `bedrock.amazonaws.com`.

*D5 · Nova judge setup*
</details>

<details>
<summary><b>Q:</b> How was the Nova judge model trained?</summary>

SFT on human-preference datasets followed by RL stages and multi-annotator review, calibrated to broad human consensus; covers 90+ languages.

*D5 · Nova judge training*
</details>

<details>
<summary><b>Q:</b> What kind of comparison does Amazon Nova LLM-as-a-Judge perform?</summary>

Pairwise binary overall preference: compares Model A vs Model B responses side by side, picks the better one or a tie, and aggregates into win rate with confidence intervals.

*D5 · Nova judge type*
</details>

<details>
<summary><b>Q:</b> An agent shows low latency and no errors, but users get wrong answers. What capability is needed?</summary>

AgentCore Evaluation - observability shows what happened; evaluation scores how well (correctness, helpfulness, tool use) to catch silent failures.

*D5 · Observability vs evaluation*
</details>

<details>
<summary><b>Q:</b> When should you use AgentCore online evaluation versus the on-demand Evaluate API?</summary>

Online: continuously score sampled live production sessions from CloudWatch Logs. On-demand Evaluate API: score curated regression cases before deploying a prompt/model change, e.g. CI/CD PR gating.

*D5 · Online vs on-demand*
</details>

<details>
<summary><b>Q:</b> Which metric measures how well a model predicts the next token?</summary>

Perplexity.

*D5 · Perplexity*
</details>

<details>
<summary><b>Q:</b> A new model scores lower on a custom Policy Adherence evaluator before deployment. What is the first step?</summary>

Examine the reason-first explanation from the judge (not deploy anyway, loosen thresholds or disable the evaluator).

*D5 · Pre-deploy regression*
</details>

<details>
<summary><b>Q:</b> Define precision, recall and F1.</summary>

Precision = TP / predicted positives; Recall = TP / actual positives; F1 = harmonic mean of precision and recall.

*D5 · Precision vs recall*
</details>

<details>
<summary><b>Q:</b> A RAG answer contradicts the retrieved documents. Which RAGAS metric will be low?</summary>

Faithfulness (answer factual consistency with context).

*D5 · RAGAS metrics*
</details>

<details>
<summary><b>Q:</b> Which rating scale type suits dashboards, alerts and human review?</summary>

Dashboards -> numerical; alerts -> categorical; human review -> hybrid.

*D5 · Rating scale choice*
</details>

<details>
<summary><b>Q:</b> Why does AgentCore use reason-first LLM-as-a-Judge, and what judge settings are recommended?</summary>

Reasoning before scoring lowers variance and gives auditable explanations; use low temperature 0.0-0.2 and enough max tokens (commonly Claude Sonnet 4.5).

*D5 · Reason-first judge*
</details>

<details>
<summary><b>Q:</b> In a self-trained SageMaker model, which tools assess bias and monitor performance drift?</summary>

SageMaker Clarify for bias; SageMaker Model Monitor for drift.

*D5 · SageMaker Clarify vs Monitor*
</details>

<details>
<summary><b>Q:</b> An evaluator metric dropped. Where do you find why?</summary>

CloudWatch Logs holds per-trace judge reasoning (filter by evaluatorId, label, score, sessionId, traceId); CloudWatch Metrics (EMF) holds aggregated trends for alarms.

*D5 · Score drop root cause*
</details>

<details>
<summary><b>Q:</b> A custom evaluator's scores cluster around 0.5. What should be done?</summary>

Rewrite the evaluator instructions - they are unclear.

*D5 · Scores cluster 0.5*
</details>

<details>
<summary><b>Q:</b> Which metric uses embeddings to measure semantic similarity to a reference, catching paraphrases?</summary>

BERTScore.

*D5 · Semantic similarity metric*
</details>

<details>
<summary><b>Q:</b> Online evaluation is scoring conversations mid-way through. What should be changed, and what are the default and range?</summary>

Increase the session idle timeout; default 15 minutes, range 1-1440 minutes.

*D5 · Session idle timeout*
</details>

<details>
<summary><b>Q:</b> Online evaluation skips or mis-scores sessions. What is the likely cause?</summary>

Traces lack required OTEL/OpenInference fields such as a persistent session ID, trace ID, span ID or tool call metadata.

*D5 · Sessions skipped*
</details>

<details>
<summary><b>Q:</b> Which automated metric is used to evaluate text summarisation?</summary>

ROUGE – recall-oriented overlap with a reference.

*D5 · Summarisation metric*
</details>

<details>
<summary><b>Q:</b> What is the main limitation of BLEU, perplexity and accuracy-style metrics for evaluating GenAI output?</summary>

They are statistical and cannot assess subjective judgment, contextual understanding or alignment with business requirements; LLM-as-a-judge provides nuanced assessments aligned with human preferences.

*D5 · Traditional metric limits*
</details>

<details>
<summary><b>Q:</b> Which metric is precision-oriented n-gram overlap used for machine translation?</summary>

BLEU.

*D5 · Translation metric*
</details>

<details>
<summary><b>Q:</b> In Nova LLM-as-a-Judge output, what does winrate represent?</summary>

The share of valid comparisons in which Model B was preferred; `lower_rate` and `upper_rate` give its 95% confidence interval.

*D5 · Winrate meaning*
</details>

<details>
<summary><b>Q:</b> An agent returns correct-looking answers but calls a generic <code>web_search</code> instead of the specialised tool. Which built-in evaluator detects this?</summary>

Tool Selection Accuracy (Tool Parameter Accuracy covers wrong arguments).

*D5 · Wrong tool selected*
</details>

---
[← Back to index](README.md)
