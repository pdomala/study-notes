[← Back to index](README.md)

# ⚡ 13 · Cost, latency & monitoring

> Exam domain: D4

## Key points
- Latency metrics: **TTFT** = chat responsiveness · **OTPS** = generation speed · **E2E** = batch/doc generation. Bedrock `latencyMs` excludes network time.
- Faster with the same model and quality → **`performanceConfig {"latency": "optimized"}`** via an inference profile (quota exceeded → falls back to standard).
- **Prompt caching**: `cachePoint` after the static prefix · exact match · ~**1K token** minimum · **5 min TTL** reset on each hit · hit = cacheReadInputTokens > 0 · up to −90% cost. Not for short or dynamic prompts.
- **Intelligent prompt routing**: same model family, router ARN as `modelId`, `responseQualityDifference` + fallback model, up to −30% cost. Other cost levers: smaller models, cap maxTokens, batch inference, distillation, semantic cache.
- Monitoring: prompts/responses → **model invocation logging** (CloudWatch Logs/S3); **CloudTrail never logs inference content**. Token/latency/cost dashboards → **CloudWatch GenAI observability**. Job state changes → **EventBridge**. Spend by team → cost allocation tags + Cost Explorer.
- Agents generate 10–100x more requests → protect backends with throttling/quotas. Throttled (429) → exponential backoff, cross-Region inference or Provisioned Throughput.

## Notes

### Latency metrics & benchmarking
- **TTFT** (time to first token) = prompt sent → first output token · most direct driver of **perceived** responsiveness · measured with streaming (`ConverseStream` / `InvokeModelWithResponseStream`)
- **OTPS** (output tokens per second) = token generation rate **after** the first token · varies by model architecture, prompt complexity, output length · network bandwidth can bottleneck at high OTPS
- **End-to-end (E2E)** = request → final token · includes network RTT, authN/Z, inference, streaming, middleware (guardrails, caching)
- Streaming chatbot → optimise **TTFT** · backend processing / document generation / waiting on full output / SLA → **E2E**
- Use these metrics to **benchmark** optimisation techniques
- Bedrock's reported `latencyMs` = time **inside Bedrock only** (excludes network, stream setup, client) → your measured TTFT is higher and closer to user experience
- Fair benchmark: same prompt/system message, **run many times, use the median**, log Bedrock latency **and** your own TTFT/OTPS
- **Benchmark before optimising** (own prompts/models/load, baseline first) → AWS open-source **Model Latency Benchmarking** tool in the **amazon-bedrock-samples** GitHub repo (models, Regions, standard vs optimized, inference profiles, TTFT, E2E)
- Looping benchmark tests → **throttling** → **Provisioned Throughput** or raise **service quotas**
- Every +100 ms of latency costs engagement/sales — LLM apps are no different; responsiveness can be a differentiator

### What drives latency & how to cut it
- **Model size**: bigger = slower → don't use a large model if a small one does the job
- **Tokenisation differs by model**: same prompt → 100 tokens in one, 150 in another → looks slower, affects latency **and cost**
- **Reasoning models** = slower but better on complex tasks (trade-off)
- Prompt length, task complexity, **output length** and config choices all add up
- **Multi-step workflows** stack latency (classify → retrieve → generate → validate) → parallelise independent steps, make non-critical steps async
- Pre/post-processing adds small delays: input validation, authN/Z, **Guardrails**, **KB retrieval**, output formatting
- **Region**: run inference **close to users** (same Region < cross-Region < intercontinental)
- Infra at scale: load balancers, queues, caches, monitoring all affect performance
- Non-determinism → output length varies per call → mitigate with **streaming** + **maxTokens** limit + **monitoring**
- **Agent latency fixes**: smaller model · instruct short answers (fewer output tokens) · pre/post-processing adds time · VPC Lambda → **PrivateLink** · **cross-Region inference** for bursts/throughput · **InvokeModelWithResponseStream** streaming · **fewer KB chunks** (reranker → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Advanced retrieval))

### Latency-optimized inference
- Tells Bedrock to prioritise speed (**may cost a bit more**) → **lower TTFT + higher OTPS**
- **Same model, same prompt, same output quality/behaviour** → only change is `performanceConfig={"latency": "optimized"}` on Converse/ConverseStream
- **Default = standard** inference unless you opt in
- **Quota exceeded → automatic fallback to standard, charged the standard rate** (no error)
- Served latency config visible in the **API response + CloudTrail**; metrics in **CloudWatch** → monitor fallbacks
- Supported models via **inference profiles** → use the profile ID with Region prefix (e.g. `us.amazon.nova-pro-v1:0`), not the raw model ID · check docs for model/Region support
- Best for chat, real-time assistants, streaming UIs, search, autocomplete

### Prompt caching
- Long static context (instructions, big document) reprocessed every call → **TTFT worsens with prompt length** + token cost
- Mark static parts with **cache checkpoints** (`{"cachePoint": {"type": "default"}}`) → everything **before** the checkpoint = cached **prompt prefix**
- Flow: request → has marked prefix? → cache lookup → **exact match = hit** (skip reprocessing) · **miss** → process normally → if ≥ model's **minimum tokens** (~**1,000** for many models) → create checkpoint, else no caching
- **TTL 5 minutes, reset on every hit**
- **Multiple checkpoints** (model-dependent): instructions → cachePoint → document → cachePoint → user question → only new parts processed (partial hits when some layers change)
- Checkpoints can go in `system` and/or `messages` content
- Verify: `cacheReadInputTokens > 0` and `cacheWriteInputTokens = 0` → served from cache · first call shows write tokens · sum in **CloudWatch** for hit rate
- Still zero after repeats → **prompt below token minimum** or **misaligned checkpoints**
- No app logic change; Bedrock manages the cache (GA)
- Up to **85% latency reduction, 90% cost savings** for repetitive context (cache repeated system prompts/context)
- Good fit: document Q&A, support KB, code gen with standard libs, **multi-turn** with persistent context, long system prompts (>1K tokens), legal/compliance text
- **Not useful**: short prompts, **highly dynamic** input, single-turn where the prompt changes entirely each call
- Design: **static content first**, logical segmentation, **identical formatting**, minimal variation in cached sections · alert on cache hit-rate drops / cost anomalies

### Intelligent prompt routing
- Single serverless endpoint that routes each request between **models in the same family** (e.g. Claude Sonnet ↔ Haiku, Nova Lite ↔ Pro) · not all models supported
- Uses a **quality-estimation model** to predict each model's response quality per request → picks best quality/cost combo
- Up to **30% cost reduction** + lower latency (simple prompts go to the smaller model) without accuracy loss
- **Default routers** = AWS-managed, pre-configured, no setup → good for experimentation
- **Custom routers** = pick **2 models** from a family + **routing criteria** + **fallback model**
- `responseQualityDifference: 10` → use the bigger model only if predicted **≥10% better** than the fallback
- **Fallback** used when the router can't make a confident decision → usually the **smaller/cheaper** model
- Create: `bedrock.create_prompt_router(promptRouterName, models=[modelArn…], routingCriteria, fallbackModel)` (control-plane `bedrock` client)
- Use: pass the **router ARN as `modelId`** to Converse/ConverseStream (`bedrock-runtime`)
- Response metadata **`trace`** shows which model was invoked
- Example: simple Yosemite trails question → Haiku · complex multi-day itinerary → Sonnet
- ≠ adaptive guardrail routing (routes between guardrail versions — custom Lambda pattern); prompt routing routes between **models of the same family** for cost/quality

### Inference options, capacity & throughput
- → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Inference & deployment options; Cross-Region inference & inference profiles; Model lifecycle — Legacy/PT rules)

### Cost optimisation levers & trade-offs
- Trade-off triangle: **quality vs speed vs cost** — no one-size-fits-all
  - Speed-first: chatbots, completion, live support, games, **voice assistants**
  - Quality-first: complex reasoning, deep summarisation, critical decisions
  - Cost-first: high-volume batch, background generation, non-critical tasks
- Optimisation checklist: 1 **benchmark first** → 2 **stream** outputs (`converse_stream`; TTFT matters more than total time in most UX) → 3 **latency-optimized inference** for user-facing chat/search/autocomplete → 4 **cap `maxTokens`** to actual need (long outputs cost more and delay) → 5 **right-size the model** (don't use Opus when Haiku will do; smaller model + smart prompt can beat bigger + generic; variable complexity → **prompt routing**) → 6 **prompt caching** for predictable/repeated prompts · treat tuning like engineering: **measure, iterate, validate with data**
- Other cost levers → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Nova right-sizing picks) · [04 · Model customisation](04-model-customisation.md) (distillation) · [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (semantic cache) · [01 · Fundamentals](01-fundamentals-genai-strategy.md) (deterministic vs agentic 95/5; Trainium/Inferentia)

### Vector store & RAG storage cost
- → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Amazon S3 Vectors as the KB vector store; Embedding models — dims/float32 vs binary; Parsing; Chunking; KB service role permissions & cleanup)

### Why GenAI monitoring is different & what to monitor
- **Why it's hard**:
  - **Performance variability** → hard baselines; model vs infra issue unclear
  - **Cost** → unpredictable consumption, per-user/per-feature cost tracking, budget alerts
  - **Quality** → no standard output metric, hallucination detection hard to automate, drift
  - **Operational complexity** → many services, end-to-end tracing, bottleneck isolation
- **Unique aspects**: **token-based** throughput (tokens/s, not req/s) · prompt changes affect latency/resources · variable latency · hallucination detection · **context window utilization** · per-user cost
- Use-case metrics: **text/LLM** = QA metrics + output validation · **image** = generation time, quality, style consistency · **audio/video** = streaming, bandwidth · **code** = IDE integration, completion accuracy, security validation
- Essentials to monitor:
  - **Infra** → GPU/CPU, network, storage
  - **Model performance** → quality, hallucination rate, temperature impact, context retention
  - **Inference** → token generation latency, prompt-to-output ratio, batching, quantization impact
  - **Data pipeline** → preprocessing, feature extraction, vector DB
  - **RAG** → retrieval accuracy, grounding, embedding cache
  - **Cost** → token economics, caching, right-sizing, quantization ROI

### KPIs, success metrics & Well-Architected GenAI Lens
- KPI families: **performance** (response time, throughput) · **quality** (output quality, model health) · **business** (engagement, cost efficiency) · **operational** (system health, security) · **resource** (infra, scaling efficiency)
- Technical & business success metrics → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (MLOps, CI/CD & success metrics)
- **Well-Architected** 6 pillars + **Generative AI Lens**:
  - **Operational Excellence** → MLOps, feature stores, automated prompt testing, model versioning
  - **Security** → strong authN/Z, secure enclaves, **monitor prompt injection**, content filtering
  - **Reliability** → multi-Region, **circuit breakers + fallbacks**, async for long tasks, retries
  - **Performance** → distillation, **caching**, **Inferentia**
  - **Cost** → serverless, **optimise prompts to cut tokens**
  - **Sustainability** → use FMs (no training), low-carbon Regions, efficient batch

### CloudWatch, GenAI observability & alarms
- **CloudWatch** = usage metrics + dashboards: near real-time metrics + logs, alarms, auto-scaling actions; advanced = **anomaly detection**, **composite alarms**, **metric math**; end-to-end = ServiceLens + X-Ray + **cross-account observability**
- **CloudWatch GenAI observability** — 2 pre-built views:
  - **Model Invocations** → usage, token consumption (total/avg/input/output), latency (avg/**P90/P99**), errors, throttles, cost by app/role/user + invocation logs table
  - **Bedrock AgentCore agents** → **Agent** view · **Session** view · **Traces** view (spans)
  - Works with EKS, ECS, AgentCore, on-prem; Strands/LangGraph/CrewAI via **OTEL**; ADOT SDK auto-instruments, **no extra collector**
- Bedrock monitoring best practices: CloudWatch (API calls, latency, errors) · CloudTrail (security, compliance audit) · Cost Explorer + **tagging** · exponential backoff retries · **AWS Config rules** for policy compliance · monitor latency, request/response time, token usage, errors, cost/request → **baselines + reviews**
- SageMaker metrics kept **15 months**; console search only shows metrics updated in the last **2 weeks**; container **stdout/stderr → CloudWatch Logs** (training, processing, endpoints, transform, notebooks)
- Implementation tips: sampling rates, **dimension strategy**, retention · persona-based dashboards · **composite alarms** (fewer false positives) · **percentile alarms (p95/p99)** for latency · **Synthetics canaries** test endpoints · **anomaly detection** for seasonal patterns / sudden spikes · **Metric Streams → S3** for long-term analysis
- Latency alarms → **establish a baseline first**, alarm relative to it

### Logging, tracing & audit
- **Model invocation logging** = actual inputs/outputs for **all invocations in the account per Region** → **CloudWatch Logs and/or S3** · **KB logging** = data ingestion visibility
- **CloudTrail** = API activity, **never inference content** → detail (key events, guardrail changes, CloudTrail + CloudWatch + Config) See [12 · Governance](12-governance-compliance-responsible-ai.md) (AWS governance & compliance services) · GuardDuty anomaly detection → [11 · Security](11-security-identity-agent-security.md) (Threat detection & IR)
- **EventBridge** → state changes for **model customization** and **batch inference** jobs
- **X-Ray**: AgentCore distributed tracing, trace-context propagation, custom instrumentation via **ADOT SDK**; requires **CloudWatch Transaction Search** enabled · SageMaker: trace via **API Gateway stage**, **Lambda** pre/post-processing, X-Ray SDK in custom training scripts, **Step Functions** pipelines, Glue
- **ADOT** = AWS Distro for CNCF **OpenTelemetry** → instrument once, send to CloudWatch, X-Ray, OpenSearch, Managed Prometheus · AgentCore supports **OpenInference, OpenLLMetry, OpenLit, Traceloop**
- **Strands Agents SDK** records **agent trajectories** and emits **OTEL** traces/metrics/logs (X-Ray, CloudWatch, Jaeger)
- **Logs Insights** queries:
  - errors: `filter @message like /error/ | parse @message "error_type: *" as error_type | stats count(*) by error_type | sort count(*) desc`
  - tokens: `parse @message "tokens_used: *" as tokens | stats sum(tokens), avg(tokens) by bin(1h)`

### Cost monitoring & attribution
- **Cost allocation tags** → Billing / **Cost Explorer**
- **Application inference profiles** = usage tracking + tags flow to Billing → Bedrock spend by cost centre / app / environment
- **AWS Budgets** → tag-based budgets & alerts
- GenAI observability Model Invocations view → **cost by app/role/user**; track **cost per request** in CloudWatch against baselines
- Prompt-cache hit rate (sum `cacheReadInputTokens`) + alerts on hit-rate drops / cost anomalies

## Exam triggers
- "Chat app feels slow before text appears" / "long wait for output" → **TTFT** → **streaming** (ConverseStream / InvokeModelWithResponseStream) + **latency-optimized inference**
- "Batch / document generation SLA" → **end-to-end latency**
- "Faster responses, same model & quality, no code rewrite" → **`performanceConfig latency: optimized`** via an **inference profile**
- "Latency-optimized quota exceeded" → **falls back to standard, standard price** (no error)
- "Bedrock latencyMs lower than user sees" → it **excludes network/client** time
- "Same long system prompt / document on every request" → **prompt caching** with `cachePoint` after the static prefix
- "Cache never hits" → prefix **under token minimum**, **not an exact match**, wrong checkpoint placement, or **>5 min** gap
- "Confirm cache hit" → **cacheReadInputTokens > 0, cacheWriteInputTokens = 0**
- "Mix of simple and complex prompts, cut cost without hand-coded routing" → **intelligent prompt routing** (same family, router ARN as modelId)
- "Which model handled the request?" → router **trace** metadata
- "Reduce latency + cost of long responses" → **lower maxTokens**
- "See the actual prompts/responses" → **Bedrock model invocation logging** (CloudWatch Logs / S3), **not CloudTrail**
- "Out-of-the-box token/latency/cost dashboard" → **CloudWatch GenAI observability (Model Invocations)** · agents/sessions/traces → **AgentCore agents view**
- "X-Ray traces missing for AgentCore" → enable **CloudWatch Transaction Search**
- "Vendor-neutral instrumentation" → **ADOT / OpenTelemetry**
- "Alert on customization/batch job completion" → **EventBridge**
- "Latency SLO" → **p95/p99 percentile alarm**
- "Fewer false alarms" → **composite alarm**
- "Bedrock cost per team/app" → **application inference profiles + cost allocation tags** → Cost Explorer / **Budgets**

## Flashcards
<details>
<summary><b>Q:</b> A Bedrock agent is slow. Which fixes reduce its latency?</summary>

Smaller model, instruct shorter answers, PrivateLink for VPC Lambda, cross-Region inference for bursts, InvokeModelWithResponseStream streaming, and fewer KB chunks.

*D4 · Agent latency fixes*
</details>

<details>
<summary><b>Q:</b> Which CloudWatch alarm types suit latency SLOs and reducing false alarms?</summary>

Percentile alarms (p95/p99) for latency; composite alarms to reduce false positives; anomaly detection for seasonal patterns.

*D4 · Alarm best practices*
</details>

<details>
<summary><b>Q:</b> Which best practice balances quality, latency and cost?</summary>

Use intelligent prompt routing and cache repeatable prompts (not always using the highest-quality model, disabling streaming or raising max tokens).

*D4 · Balance quality/cost*
</details>

<details>
<summary><b>Q:</b> Which Bedrock features act as cost levers?</summary>

Model distillation, prompt caching and Intelligent Prompt Routing.

*D4 · Bedrock cost levers*
</details>

<details>
<summary><b>Q:</b> Why is Bedrock's reported latencyMs lower than the TTFT users experience?</summary>

latencyMs measures only time inside Bedrock, excluding network, stream setup and client time. Benchmark by running many times and using the median.

*D4 · Bedrock latencyMs*
</details>

<details>
<summary><b>Q:</b> What tool does AWS provide to benchmark Bedrock latency across models, Regions and standard vs optimized inference?</summary>

The open-source Model Latency Benchmarking tool in the amazon-bedrock-samples GitHub repo (measures TTFT and E2E).

*D4 · Benchmark tool*
</details>

<details>
<summary><b>Q:</b> cacheReadInputTokens stays at zero after repeated identical requests. What are the likely causes?</summary>

Prefix below the model's token minimum, misaligned checkpoints, a prefix that is not an exact match, or more than 5 minutes between calls.

*D5 · Cache never hits*
</details>

<details>
<summary><b>Q:</b> What are the prompt cache TTL and typical minimum prefix size?</summary>

TTL 5 minutes, reset on every cache hit; minimum is model-dependent, about 1,000 tokens for many models.

*D4 · Cache TTL & minimum*
</details>

<details>
<summary><b>Q:</b> Which services provide Bedrock usage metrics and which records API activity (who called what)?</summary>

CloudWatch = usage metrics and dashboards; CloudTrail = API activity/auditing.

*D4 · CloudWatch vs CloudTrail*
</details>

<details>
<summary><b>Q:</b> Finance needs Bedrock spend by team, application and environment with budget alerts. What should be used?</summary>

Application inference profiles with cost allocation tags flowing to Billing/Cost Explorer, plus tag-based AWS Budgets.

*D4 · Cost per team*
</details>

<details>
<summary><b>Q:</b> What do you configure for a custom prompt router, and how do you invoke it?</summary>

Two models from one family, routing criteria (e.g. `responseQualityDifference: 10` = use bigger model only if predicted at least 10% better) and a fallback model; pass the router ARN as `modelId` to Converse/ConverseStream.

*D4 · Custom prompt router*
</details>

<details>
<summary><b>Q:</b> A team wants an out-of-the-box dashboard of Bedrock token usage, latency (P90/P99), errors and throttles. What should they use?</summary>

CloudWatch GenAI observability - the Model Invocations view; the Bedrock AgentCore agents view gives Agent, Session and Traces views.

*D4 · GenAI observability*
</details>

<details>
<summary><b>Q:</b> Who removed a guardrail from a Bedrock application, and how would suspicious Bedrock API use from a new location be detected?</summary>

CloudTrail records who/what/when/IP of API calls (Event history if no trail); GuardDuty detects anomalous activity such as removing guardrails after login from a new location.

*D3 · Guardrail change audit*
</details>

<details>
<summary><b>Q:</b> A workload mixes simple and complex prompts. The team wants to cut cost without hand-coding routing logic. What Bedrock feature fits?</summary>

Intelligent prompt routing - a serverless router that uses a quality-estimation model to send each request to a model in the same family (e.g. Haiku vs Sonnet); up to 30% cost reduction.

*D4 · Intelligent prompt routing*
</details>

<details>
<summary><b>Q:</b> An auditor needs to see the actual prompts and model responses sent to Bedrock. What should be enabled?</summary>

Bedrock model invocation logging to CloudWatch Logs and/or S3 - CloudTrail does not capture inference content.

*D4 · Invocation logging*
</details>

<details>
<summary><b>Q:</b> How can a team be alerted when a Bedrock model customization or batch inference job changes state?</summary>

Amazon EventBridge state-change events.

*D4 · Job state alerts*
</details>

<details>
<summary><b>Q:</b> Which latency metric matters most for a streaming chatbot and which for a backend document-generation SLA?</summary>

Streaming chatbot: TTFT (time to first token). Backend/batch/document generation: end-to-end latency. OTPS measures generation rate after the first token.

*D4 · Latency metrics*
</details>

<details>
<summary><b>Q:</b> What happens when latency-optimized inference quota is exceeded?</summary>

Bedrock automatically falls back to standard inference and charges the standard rate (no error); the served config appears in the API response and CloudTrail.

*D4 · Latency quota fallback*
</details>

<details>
<summary><b>Q:</b> A team wants faster Bedrock responses with the same model, prompt and output quality and no code rewrite. What should they do?</summary>

Set `performanceConfig = {latency: optimized}` on Converse/ConverseStream, using the model's inference profile ID (e.g. `us.amazon.nova-pro-v1:0`); it lowers TTFT and raises OTPS but may cost more.

*D4 · Latency-optimized inference*
</details>

<details>
<summary><b>Q:</b> How do you reduce latency and cost driven by overly long model outputs?</summary>

Cap maxTokens to actual need (and stream); also right-size the model.

*D4 · Long responses cost*
</details>

<details>
<summary><b>Q:</b> Every request includes the same long system prompt and document. How can latency and cost be cut with minimal app changes?</summary>

Bedrock prompt caching: place a `cachePoint {type: default}` after the static prefix; up to 85% latency reduction and 90% cost savings.

*D4 · Prompt caching use*
</details>

<details>
<summary><b>Q:</b> How can you tell which model an intelligent prompt router actually invoked?</summary>

From the trace in the response metadata.

*D4 · Router model used*
</details>

<details>
<summary><b>Q:</b> How long are SageMaker CloudWatch metrics retained, and where do container stdout/stderr go?</summary>

Metrics are kept 15 months (console search shows those updated in the last 2 weeks); container stdout/stderr go to CloudWatch Logs.

*D4 · SageMaker metrics retention*
</details>

<details>
<summary><b>Q:</b> The same questions are asked repeatedly against a RAG app. How can latency and cost be reduced?</summary>

Use a semantic cache to reuse retrieval and FM output for similar repeated queries.

*D4 · Semantic cache*
</details>

<details>
<summary><b>Q:</b> Users complain a chat app feels slow before any text appears. What should be optimised?</summary>

TTFT - stream responses with ConverseStream/`converse_stream` and use latency-optimized inference.

*D4 · Slow first text*
</details>

<details>
<summary><b>Q:</b> Latency benchmark loops start getting throttled. What is the remedy?</summary>

Use Provisioned Throughput or request service quota increases.

*D4 · Throttled load tests*
</details>

<details>
<summary><b>Q:</b> How do you confirm a Bedrock response was served from the prompt cache?</summary>

cacheReadInputTokens > 0 and cacheWriteInputTokens = 0 (the first call shows write tokens). Sum in CloudWatch for hit rate.

*D4 · Verify cache hit*
</details>

<details>
<summary><b>Q:</b> When is prompt caching not useful?</summary>

Short prompts, highly dynamic input, or single-turn calls where the prompt changes entirely each time.

*D4 · When not to cache*
</details>

---
[← Back to index](README.md)
