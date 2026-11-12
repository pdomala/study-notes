[← Back to index](README.md)

# 🛠️ 04 · Model customisation (fine-tuning, distillation, RFT)

> Exam domain: D1

## Key points
- Domain terminology + consistent format + labelled data → **fine-tuning**, not RAG. Data changes frequently → **RAG**.
- Prompt-based fine-tuning = **labelled, single task** · continued pre-training = **unlabelled domain data, multi-task**.
- **Distillation**: teacher and student must be from the **same provider**, deployed on **Provisioned Throughput**; Bedrock auto-generates up to **15K** prompt-response pairs (can use invocation logs).
- **RFT** (reinforcement fine-tuning): no labelled data, reward signals instead · **RLVR** = rule-based grader (code/math) · **RLAIF** = AI judge (subjective).
- Customisation ladder: RAG → fine-tune (labelled) → purpose-built LLM. **Nova Forge** = custom frontier model on Nova with your data and reward functions.
- Security: fine-tuned model = **Scope 4**, inherits the **highest classification** of its tuning data; can't unlearn PII without a full retrain → **anonymise first**; check the base-model licence allows fine-tuning.

## Notes
### Customisation spectrum (cheapest → deepest)
- Order: **prompt engineering → RAG / GraphRAG → fine-tuning → continued pre-training → distillation / Nova Forge → train from scratch** (cost ↑, tailoring ↑; from scratch is out of exam scope to perform)
- **Prompt engineering** — zero-shot · few-shot (2–5 examples) · chain-of-thought · role-based; guides the model, no weight change (see [03 · Prompt engineering](03-prompt-engineering-prompt-management.md))
- **RAG** — vector DB of embeddings → similarity search → inject context → grounded answer (external/fresh knowledge, no retraining)
- **GraphRAG** → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Advanced retrieval — GraphRAG)
- Fix ladder for pre-trained FM limits (least → most complex): **RAG** (curated data) → **fine-tune** (curated **labelled** data) → **purpose-built LLM** (train FM on your data)
- **Cost trade-off:** pre-trained = cheap, may lack accuracy ↔ heavy fine-tune / from scratch = more accurate, much more expensive (compute, data, expertise)

### RAG vs fine-tuning
- **RAG** → retriever finds relevant info → **augmented prompt** → FM; **no weight changes** → cheaper, handles **frequently changing data** (temporary intelligence)
  - RAG limits → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Why RAG)
- **Fine-tuning** → **permanent** change to the model weights; use when you need **domain terminology**, **consistent output format**, task-specific performance, brand behaviour, and have labelled data

| Criteria | RAG | Fine-tuning |
| --- | --- | --- |
| When | Custom data, FM lacks domain knowledge | Highly custom/proprietary domain; also **reduces latency** (less context) |
| Effort | Medium | High |
| Cost | Low–medium | Low–high (model size) |

### Fine-tuning types — instruction, continued pre-training, RLHF, PEFT
- **Fine-tuning** = further train a pre-trained model on task/domain data (e.g. medical research → fine-tune on medical journal articles)
- **Instruction / prompt-based fine-tuning** ("customisation / prompt-based learning") → **labelled** prompt-response pairs as instructions, modifies weights, few epochs, **single task, doesn't generalise**; prompt tuning is a type of this; changes style/behaviour
- **Continued pre-training / domain adaptation** → **unlabelled** domain data, adapts to **multiple tasks**, learns domain terminology / adds domain knowledge; needs ML env + compute
- **RLHF** (and **DPO**) → human feedback → aligned with human preferences
- **PEFT** = parameter-efficient fine-tuning, trains a small set of extra weights · **LoRA / adapters** = popular PEFT methods (small, swappable add-ons to a base model)
- Bedrock custom model training for supported models (customisation trains a **private copy** of the base model); **SageMaker for advanced** FT
- **Custom model import** → bring your own fine-tuned open weights into Bedrock · **Model registry** (SageMaker Model Registry) → versioning + approval of models

### Building a model from scratch
- Define architecture → curate huge dataset → random-init weights → train to minimise loss
- Only when no suitable pre-trained model exists or exceptional requirements; **highest cost, time, expertise**

### Model distillation
- Large **teacher** (e.g. Nova Premier / Nova Pro) → small **student** (e.g. Nova Lite); near-teacher accuracy for a task at student speed/cost (performance vs speed vs cost)
- Steps: pick teacher + student → supply use-case prompts → teacher generates responses → student fine-tuned on them → deploy
- No manual labelling: Bedrock generates up to **15,000 prompt-response pairs**
- Training data: **JSONL prompts in S3** or **historical invocation logs** (filter by metadata)
- ⚠️ Teacher & student **same provider** (Amazon, Anthropic, Meta)
- ⚠️ Deploy on **Provisioned Throughput** (hourly); evaluate before full rollout
- Use cases: high-concurrency chatbots, real-time summarisation, ticket/email classification, high-volume enterprise search

### Reinforcement fine-tuning (RFT)
- Trains from **feedback / reward functions** instead of big labelled datasets
- ~**66% accuracy gain** avg over base; works with small models (e.g. **Nova 2 Lite**); data stays in AWS
- **RLVR** (verifiable rewards) → **rule-based graders**, objective tasks (code, math)
- **RLAIF** (AI feedback) → **AI judges**, subjective tasks (instruction following, moderation)
- Data: **Bedrock API logs**, uploaded **JSONL**, **S3**; real-time training metrics, templates, playground

### Nova Forge & where to customise (Forge vs SageMaker AI vs Bedrock)
- GA Dec 2025; build **domain-specific Nova models on your own data**; from **~$100K/yr** vs hundreds of millions from scratch; months → weeks; us-east-1
- 5 capabilities: **access checkpoints across all dev phases**, **blend proprietary data with Nova-curated training data**, **RL with custom reward functions**, **push-button recipes (SageMaker AI)**, **custom safety guardrails**
- Best base model for Forge = **Nova 2 Lite**
- **Forge vs SageMaker AI vs Bedrock:**
  - **All three:** reinforcement fine-tuning, SFT with PEFT (labelled data)
  - **Forge + SageMaker only:** continued **pre-training**, continued **mid-training**
  - **Forge only:** RL with reward functions in *your* environment, **full-weights SFT**, any training **blended with original Nova training data**
  - Bedrock = lightest (PEFT / reinforcement FT); SageMaker = open-weight + Nova; Forge = deepest Nova customisation

### Using custom models
- Custom / distilled models are served on **Provisioned Throughput** (Model Units); support after base-model EOL → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Model lifecycle)
- LangChain: custom models (continued pre-training / fine-tuning) → `model_id` = **custom model ARN** from the **provisioned** custom model + `provider`

## Exam triggers
- "Domain terminology", "highly accurate", "cost-effective customisation" → **fine-tuning**
- "Align with human preferences" → **RLHF**
- "No suitable model exists" → **train from scratch** (most expensive)
- "Labelled examples for one task" → **prompt-based fine-tuning**
- "Lots of unlabelled domain text" → **continued pretraining**
- "Data changes frequently" → **RAG**, not fine-tuning
- "Big model too slow/expensive at high volume, keep accuracy" / "smaller cheaper model from a big one" → **model distillation** (e.g. Nova Premier as teacher)
- "Distillation with teacher from Anthropic, student from Meta" → ❌ **not allowed** (same provider)
- "No labelled data, have a grader/reward" → **RFT**: objective → **RLVR**, subjective → **RLAIF**
- "Use a fine-tuned Bedrock model in LangChain" → **custom model ARN** (provisioned)

## Flashcards
<details>
<summary><b>Q:</b> Order these by cost and tailoring: fine-tuning, prompt engineering, training from scratch, RAG.</summary>

Prompt engineering → RAG → fine-tuning → training from scratch (highest cost, time and expertise; only when no suitable pre-trained model exists).

*D1 · Customisation cost order*
</details>

<details>
<summary><b>Q:</b> Order RAG, fine-tuning and purpose-built LLM from least to most complex, and what data does fine-tuning need?</summary>

RAG (curated data) → fine-tune (curated labelled data) → purpose-built LLM (train an FM on your data).

*D1 · Customisation ladder*
</details>

<details>
<summary><b>Q:</b> Can you distil with an Anthropic teacher and a Meta student?</summary>

No – teacher and student must be from the same provider.

*D1 · Distillation constraint*
</details>

<details>
<summary><b>Q:</b> What training data sources can Bedrock model distillation use?</summary>

JSONL prompts in S3 or historical invocation logs (filterable by metadata); no manual labelling needed.

*D1 · Distillation data*
</details>

<details>
<summary><b>Q:</b> Distinguish prompt-based fine-tuning from continued pretraining.</summary>

Prompt-based fine-tuning uses labelled prompt-response pairs for a single task (doesn't generalise). Continued pretraining uses unlabelled domain data to adapt to multiple tasks and learn domain terminology.

*D1 · Fine-tuning types*
</details>

<details>
<summary><b>Q:</b> A model is good at general medicine but weak on oncology terminology, needs consistent output format, and the team has a large annotated dataset. RAG, few-shot, or fine-tuning?</summary>

Fine-tuning – suited to domain terminology, consistent output format and task-specific performance when labelled data exists.

*D1 · Fine-tuning vs RAG*
</details>

<details>
<summary><b>Q:</b> Which customisation capabilities are available in Nova Forge and SageMaker AI but NOT Bedrock?</summary>

Continued pre-training and continued mid-training. All three support reinforcement fine-tuning and SFT with PEFT; Bedrock is the lightest option.

*D1 · Forge vs SageMaker vs Bedrock*
</details>

<details>
<summary><b>Q:</b> A large model is too slow and expensive at high volume, but accuracy must be kept. What should be used, and how is it deployed?</summary>

Model distillation – teacher generates responses (up to 15,000 prompt-response pairs) to fine-tune a smaller student; deploy on Provisioned Throughput.

*D4 · Model distillation*
</details>

<details>
<summary><b>Q:</b> A company wants a custom frontier model built on Nova using proprietary data and custom reward functions, in weeks rather than months, far cheaper than training from scratch. What should it use?</summary>

Amazon Nova Forge (from ~$100K/yr, us-east-1). It uniquely offers full-weights SFT, RL with reward functions in your own environment, and blending with Nova's original training data.

*D1 · Nova Forge*
</details>

<details>
<summary><b>Q:</b> What is the key difference between prompt engineering and fine-tuning?</summary>

Prompt engineering guides an already-trained model without changing weights (fast, cheap); fine-tuning adjusts weights with task data (expensive compute and time).

*D1 · Prompt eng vs fine-tuning*
</details>

<details>
<summary><b>Q:</b> Data changes frequently. RAG or fine-tuning?</summary>

RAG – provides temporary context at retrieval time; fine-tuning permanently changes weights.

*D1 · RAG vs fine-tuning*
</details>

<details>
<summary><b>Q:</b> No labelled data is available, but there is a grader. When do you use RLVR vs RLAIF in reinforcement fine-tuning?</summary>

RLVR (rule-based verifiable rewards) for objective tasks like code/math; RLAIF (AI judges) for subjective tasks like instruction following or moderation. RFT averages ~66% accuracy gain.

*D1 · RFT RLVR vs RLAIF*
</details>

---
[← Back to index](README.md)
