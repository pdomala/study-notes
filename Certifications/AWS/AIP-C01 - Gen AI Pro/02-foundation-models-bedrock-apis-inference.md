[← Back to index](README.md)

# 🤖 02 · Foundation models, Bedrock APIs & inference options

> Exam domain: D1 / D2

## Key points
- **FM selection criteria**: modality, languages, size/complexity, context length, customisation support, latency, cost, Regional availability, responsible AI. Real-time / mobile app → **latency** decides.
- Swap models without code changes → **Converse API** (one request shape for all models; ConverseStream for streaming). InvokeModel = model-specific body.
- Throttling / Regional capacity → **cross-Region inference profile** (no code change).
- **Model lifecycle**: Active → **Legacy** (≥6 months notice, no new customers, **no new Provisioned Throughput**) → **EOL** (requests fail, **no auto-migration**).
- Inference options: **on-demand** (pay per token) · **Provisioned Throughput** (guaranteed capacity; required for custom/distilled models) · **batch** (async, cheaper, no user waiting).
- Response length = **hard** limit · length penalty = **soft** limit. Factual/deterministic → low temperature/top_p/top_k; tune **one parameter at a time**.
- Bedrock never uses your prompts/outputs to train base models or shares them with providers. Private VPC access → **PrivateLink**. **CloudWatch** = usage metrics · **CloudTrail** = API activity.

## Notes

### Selecting a foundation model
- First decision: **pre-trained vs build from scratch**; pre-trained = head start, faster convergence, better generalisation — but may carry **bias** / miss domain nuance
- **Selection criteria:**
  - **Cost** — licensing, inference compute, customisation
  - **Modality** — text / image / audio / multimodal
  - **Latency** — real-time vs batch-tolerant
  - **Multi-lingual support**
  - **Model size** — bigger = better on complex tasks, more compute
  - **Model complexity** — harder to deploy/optimise; simpler for constrained envs
  - **Customisation** — can it be fine-tuned?
  - **Input/output length** (context window)
  - **Responsibility** — bias, misinformation, training-data provenance
  - **Deployment & integration** — tooling, infra compatibility
- Bedrock offers multiple providers — compare against criteria and **re-review as models evolve**
- Quick picks: **latency-critical → Nova Micro** · **balanced → Nova Pro** · **complex reasoning → Nova Premier**; **right-size** — don't default to the biggest model

### Bedrock model catalogue & Marketplace
- **FM** = large model trained on broad data, adaptable to many tasks via **prompt engineering / fine-tuning**
- Bedrock = **100+ FMs**, swap models without rewriting app code (single API)
- **Third-party via Bedrock:** Anthropic (Claude), AI21 (Jurassic/Jamba), Cohere, Meta (Llama 3/4), Mistral (Mistral Large), Stability AI, DeepSeek-R1, plus Amazon (Titan, Nova)
- **Bedrock Marketplace** → **100+ specialised FMs** (biomedical, financial docs, niche language pairs) with the **same Bedrock APIs, infra & security controls**

### Amazon Nova family
- **Understanding** (text/image/video in → text out):
  - **Micro** — text-only, **lowest latency/cost**, 128K
  - **Lite** — low-cost **multimodal** (img/video/text), high throughput (doc images, charts, short video), 300K
  - **Pro** — balanced accuracy/speed/cost, multimodal, complex reasoning, 300K
  - **Premier** — most capable, complex reasoning; **teacher for model distillation**; **1M-token context** (≈500 images / 90 min video)
- **Creative** (image/video out): **Canvas** image gen + editing (inpaint, outpaint, bg removal, colour-guided) + **watermarking** · **Reel** short video gen (style + camera-motion control) + watermarking
- **Sonic** — **speech-to-speech**, real-time low-latency voice, no separate STT/TTS · **Nova 2 Sonic** — real-time conversational AI (customer support, voice assistants)
- **Nova Act** — model trained to take actions in a **web browser** (multi-step tasks)
- **Nova Multimodal Embeddings** — text/image/video/audio/doc → embedding; semantic search, agentic RAG
- **Nova 2 Lite** = **recommended production model** — hybrid reasoning with adjustable **thinking effort**, built-in **web grounding + code execution**, multimodal, long context (figures quoted as 300K or 1M tokens); also best base model for Nova Forge

### Other model families — Titan, Claude, Jamba, Stable Diffusion
- **Titan Text** → summarise, generate, classify, QA, extraction; body `inputText` + `textGenerationConfig`; `stopSequences` pipe-separated (max 20 chars)
- **Titan Text Embeddings V2** (`amazon.titan-embed-text-v2:0`) → dimensions **256 / 512 / 1024**, `normalize` option; search & personalisation
- **Titan Multimodal Embeddings G1** → text + image in the **same semantic space** (search images by text/image)
- **Titan Image Generator** v1: text-to-image, editing, mask, **inpainting** (fill missing areas), **outpainting** (extend boundaries); v2 adds **reference-image guidance** + **background removal**
- **AI21 Jamba 1.5** → hybrid **SSM-Transformer**, **256K** context; Mini 52B / Large 398B; supports `invoke_model` + `converse`; param `n` = number of responses
- **Anthropic Claude** → Messages API (role + content), text + image input; tiers **Haiku** (fast/cheap) · **Sonnet** (balanced) · **Opus** (most capable) · **Fable** (advanced reasoning + extra safety guardrails); extra params `max_tokens` + **extended thinking** (`thinking` token budget for complex reasoning)
- **Stable Diffusion 3.5 Large** (`stability.sd3-5-large-v1:0`) → 8B params, 1MP; images, art, logos; modes **text-to-image** (`aspect_ratio`) / **image-to-image** (`strength` 0–1, default 0.5); `negative_prompt`, `seed`; output base64; SDXL 1.0 deprecated

### Model lifecycle (Active → Legacy → EOL)
- Lifecycle state in `modelLifecycle` field of `GetFoundationModel` / `ListFoundationModels`
- Every model available **≥12 months from launch** before EOL
- **Active** → recommended for new work; some only reachable via **CRIS** in certain Regions
- **Legacy** → advance notice, **≥6 months** before EOL
  - ❌ new customers can't use it · inactive existing customers may lose access · ❌ **no new Provisioned Throughput**
  - EOL after 1 Feb 2026 → **public extended access**: after ≥3 mo in Legacy, ≥3 more months at **higher price** (≥3 mo notice)
- **EOL** → unavailable in **all Regions**, requests **fail**; **migration is NOT automatic** (update the model ID)
- Customised model on **Provisioned Throughput (Model Units)** → supported **12 months after base model EOL** if MUs bought **before the Legacy date**

### Bedrock APIs — InvokeModel vs Converse & streaming
- **Control plane** client `bedrock` → `ListFoundationModels` (models + metadata)
- **Runtime** client `bedrock-runtime`:
  - `InvokeModel` → inference for **text, image and embedding** models; **model-specific body**
  - `InvokeModelWithResponseStream` → same but **streamed chunks** (header `x-amzn-bedrock-accept-type`)
  - `Converse` / `ConverseStream` → **write once, swap models**; one request shape for all models; `inferenceConfig`, system prompts; model-unique params still passable
- Both `InvokeModel` and `Converse` send messages to a model; token-by-token chat UX → `InvokeModelWithResponseStream` / `ConverseStream`

### Cross-Region inference (CRIS) & inference profiles
- Use an **inference profile ARN / cross-Region model ID**; routes across Regions in the configured Region set → higher throughput, fewer capacity failures, **no code changes**
- Inference profile = FM + routing rules across one or more Regions (throughput + cost tracking)
- Some Active models are only reachable via CRIS in certain Regions

### Inference & deployment options
- **Real-time:** Bedrock **on-demand** (pay per token, variable load; subject to quotas/throttling) · **Provisioned Throughput** (reserved model units, consistent/guaranteed performance; required for some custom models) · **SageMaker endpoints** (custom models)
- **Batch:** Bedrock **batch inference** (asynchronous bulk jobs, lower cost, not real-time) · **SageMaker Batch Transform** · **AWS Batch** (containers) · **EMR** (distributed); group requests for throughput, lower per-request cost
- Cost components: On-Demand vs **Provisioned Throughput** · storage + **cross-Region data transfer**
- Model availability is **Region-limited**; check **service quotas**; pay for what you use, no minimums
- **Throttling (429 / ThrottlingException)** → retry with **exponential backoff + jitter**; check/raise **service quotas**; bursts → **cross-Region inference**; sustained load (e.g. load tests) → **Provisioned Throughput**

### Calling Bedrock models from LangChain
- Packages: `pip install langchain-aws langchain-community` (production deployments → see [08 · AgentCore](08-bedrock-agentcore.md) (LangChain vs AgentCore))
- **LLM class** (`BedrockLLM`) → text in, text out; abstraction across providers; pass `model_id`, `model_kwargs` (temperature, maxTokenCount), `streaming`
- **Chat models** (`ChatBedrock`) → take **messages** (`HumanMessage`) → returns `AIMessage` with token usage; for chatbots/assistants
- **Text embedding models** (`BedrockEmbeddings`, e.g. `amazon.titan-embed-text-v2:0`) → vector of floats capturing **semantic meaning**

## Exam triggers
- "Real-time", "mobile", "interactive" → **latency** drives FM choice
- "Swap models without rewriting code" → **Converse API**
- "Throttling (429) / traffic bursts / regional capacity / higher throughput / throttled during load tests" → **service quotas** (request increase) + **exponential backoff** + **cross-Region inference profile**; sustained load → **Provisioned Throughput**
- "Token-by-token UX / chat streaming" → **InvokeModelWithResponseStream / ConverseStream**
- "Model deprecated, requests failing" → model hit **EOL**; update the model ID (no auto-migration)
- "Can't create Provisioned Throughput" → model is **Legacy**
- "Model not available" → check **Region** availability
- "Niche/industry model not in core catalog" → **Bedrock Marketplace**
- "Real-time voice assistant" → **Nova Sonic**
- "Cheapest text-only model" → **Nova Micro**
- "Image search by text or image" → **Titan Multimodal Embeddings**

## Flashcards
<details>
<summary><b>Q:</b> Which Amazon model is trained to take actions in a web browser?</summary>

Amazon Nova Act.

*D2 · Browser automation model*
</details>

<details>
<summary><b>Q:</b> A team wants to write code once and swap foundation models without rewriting request bodies. Which API?</summary>

Converse / ConverseStream (with inferenceConfig and system prompts). InvokeModel requires a model-specific body.

*D2 · Converse API*
</details>

<details>
<summary><b>Q:</b> An app hits throttling and regional capacity limits on Bedrock. How can it gain throughput without code changes?</summary>

Use cross-Region inference (CRIS) by calling an inference profile ARN/ID, which routes requests across Regions in the profile's Region set.

*D4 · Cross-Region inference*
</details>

<details>
<summary><b>Q:</b> How long is a customised model on Provisioned Throughput supported after its base model's EOL?</summary>

12 months after base model EOL, if the Model Units were purchased before the Legacy date.

*D2 · Custom model on PT*
</details>

<details>
<summary><b>Q:</b> What is public extended access for models with EOL after 1 Feb 2026?</summary>

After ≥3 months in Legacy, the model remains available ≥3 more months at a higher price, with ≥3 months notice.

*D2 · Extended access*
</details>

<details>
<summary><b>Q:</b> A company needs real-time translation in a mobile app. Which FM selection criterion is most important?</summary>

Latency.

*D1 · FM selection criterion*
</details>

<details>
<summary><b>Q:</b> A marketing team needs images, art and logos. Which model family fits, and which parameter controls image-to-image change?</summary>

Stable Diffusion (e.g. SD 3.5 Large); strength parameter 0–1 (default 0.5) controls image-to-image change.

*D1 · Image generation model*
</details>

<details>
<summary><b>Q:</b> What are the ranges/defaults for temperature, topP and Titan maxTokenCount?</summary>

temperature 0–1 (default 0); topP 0–1 (default 1); Titan maxTokenCount 0–8,000 (default 512) – a hard limit, whereas length penalty is a soft limit.

*D1 · Inference param defaults*
</details>

<details>
<summary><b>Q:</b> Which model types can InvokeModel run inference for?</summary>

Text, image and embedding models, using a model-specific request body.

*D2 · InvokeModel scope*
</details>

<details>
<summary><b>Q:</b> You cannot create new Provisioned Throughput for a model. What lifecycle state is it in, and what notice applies?</summary>

Legacy – at least 6 months notice before EOL; new customers can't use it and no new Provisioned Throughput can be created. Every model is available ≥12 months from launch.

*D2 · Legacy model limits*
</details>

<details>
<summary><b>Q:</b> A team needs a niche industry model (e.g. biomedical) not in the core catalog but wants the same Bedrock APIs and security controls. What should it use?</summary>

Amazon Bedrock Marketplace (100+ specialised FMs).

*D1 · Marketplace models*
</details>

<details>
<summary><b>Q:</b> Requests to a Bedrock model suddenly fail in all Regions. What happened and what must you do?</summary>

The model reached End-of-Life; migration is not automatic, so you must update to a new model ID.

*D2 · Model lifecycle EOL*
</details>

<details>
<summary><b>Q:</b> A retailer needs image search by text or by image in the same semantic space. Which model?</summary>

Amazon Titan Multimodal Embeddings G1.

*D1 · Multimodal search*
</details>

<details>
<summary><b>Q:</b> Which Nova model is the recommended production default, offering hybrid reasoning with adjustable thinking effort plus built-in web grounding and code execution?</summary>

Amazon Nova 2 Lite. It is also the best base model for Nova Forge.

*D1 · Nova 2 Lite*
</details>

<details>
<summary><b>Q:</b> Match the Nova model to the need: latency-critical text-only, balanced accuracy/speed/cost, complex reasoning.</summary>

Latency-critical → Nova Micro (text-only, 128K); balanced → Nova Pro (300K, multimodal); complex reasoning → Nova Premier.

*D1 · Nova model selection*
</details>

<details>
<summary><b>Q:</b> Which APIs provide token-by-token streamed responses for chat UX?</summary>

InvokeModelWithResponseStream or ConverseStream.

*D2 · Streaming responses*
</details>

<details>
<summary><b>Q:</b> What output dimensions does Titan Text Embeddings V2 support?</summary>

256, 512 or 1024 (with a normalize option).

*D1 · Titan embeddings dims*
</details>

<details>
<summary><b>Q:</b> Which model provides real-time speech-to-speech without separate STT/TTS?</summary>

Amazon Nova Sonic.

*D1 · Voice model*
</details>

---
[← Back to index](README.md)
