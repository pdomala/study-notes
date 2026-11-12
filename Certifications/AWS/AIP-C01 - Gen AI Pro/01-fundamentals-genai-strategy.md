[← Back to index](README.md)

# 🧩 01 · Fundamentals & GenAI strategy

> Exam domain: Foundations

## Key points
- **GenAI lifecycle**: define use case → select FM → adapt (prompt / RAG / fine-tune) → evaluate → deploy & monitor. Shortlist use cases on business value vs feasibility.
- **AWS GenAI stack**: apps (Amazon Q, Quick Suite) → tools to build (Bedrock, AgentCore) → infrastructure (SageMaker AI, Trainium/Inferentia). Little AI expertise → start at the top.
- **Customisation cost ladder**: prompt engineering, then RAG, then fine-tuning, then training from scratch (cost and tailoring both go up). Align with human preferences → **RLHF**.
- **Workflow vs agent**: ~95% of decision paths should be **deterministic** (Step Functions/Lambda); use agents only for judgment calls.
- Exam scope: model development/training, advanced ML and feature engineering are **out of scope**. Read every scenario for its constraint (least ops overhead, cost, latency, residency).

## Notes

### GenAI capabilities, challenges & application categories
- **Capabilities:** adaptability, responsiveness, simplicity, creativity/exploration, data efficiency, personalisation, scalability
- **Challenges:** regulatory violations, social risks, data security & privacy, toxicity, **hallucinations**, interpretability, **nondeterminism**
- 4 GenAI areas: **new experiences** (assistants, contact-centre analytics) · **productivity** (conversational search, summarisation) · **insights** (doc processing, content moderation) · **creativity** (image gen, video enhancement)
- Basic flow: user → app builds prompt → LLM → app → user
- Bedrock NLP use cases: **text summarisation** (digest large text for data exploration/cleaning, explain model behaviour, faster reports) · **text generation** (augment training data, generate code, explain models, draft content; assistants as **natural-language interface** to data/models) · **question answering** (automate doc reading, answer analytical questions, generate snippets; **RAG assistants** query knowledge bases on demand)
- Better summaries via **Bedrock + LangChain + RAG with persistent embeddings** (context awareness, key-info retention)
- GenAI lets data scientists move up to **strategic solution design** by automating repetitive implementation work; example solutions: **AI assistants**, **supply chain optimisers**, **personalised recommendation systems**

### Business value & case studies
- McKinsey: GenAI worth **$2.6–4.4 trillion/yr**; **75% of value in 4 functions**:
  - **Software engineering** → code suggestions/testing/docs (up to 57% productivity, 27% higher success rate)
  - **Customer operations** → chatbots, real-time agent assist, automated ticket resolution, 24/7
  - **R&D** → drug discovery, material simulation, design optimization
  - **Sales & marketing** → personalized content, **dynamic pricing**, campaign optimization
- Case studies (recognise the pattern):
  - **DoorDash** → contact centre on **Bedrock (Claude) + Amazon Connect**; 50x testing capacity, latency ≤2.5 s
  - **Tapestry** → Bedrock + **Transcribe + Translate** for feedback apps; 10x faster GenAI dev
  - **Chunghwa Telecom** → Bedrock for SDLC docs, English tutor, marketing; −13% doc time
  - **Bynder** → Bedrock **Titan Multimodal Embeddings** search over 175M assets; −75% search time

### Adoption strategy — AWS CAF-AI & path selection
- **CAF-AI = Cloud Adoption Framework for AI/ML/GenAI** → structured adoption across **6 perspectives**:
  - **Business** → pick high-impact use cases aligned to strategy
  - **People** → AI literacy, exec sponsorship, culture
  - **Governance** → responsible AI, data governance, model validation, AI policies
  - **Platform** → use the **3-layer GenAI stack**; choose abstraction level by expertise/customisation
  - **Security** → encryption, access mgmt, model security, compliance/privacy
  - **Operations** → **MLOps**, monitoring, scaling
- Assessment tools: **Cloud Maturity Assessment (CMA)**, **Experience-Based Acceleration (EBA)**
- Customer path selection:
  - **Ready-to-use** (Quick, Kiro, Transform) → fastest, lowest risk & control, **highest ongoing cost**, lock-in
  - **Custom** (**AgentCore + Strands + Bedrock Guardrails + AWS services**) → full control & proprietary IP; needs strong team, slow, high initial cost
  - **Partner-led** (APN + GenAI Innovation Center) → regulated/complex/multi-agent, limited in-house expertise

### Defining & evaluating use cases
- Defining a use case sets the path: **define the problem, gather requirements, align stakeholders** → translate business needs into technical specs
- Business use case = structured narrative of how a system behaves from an **actor/stakeholder** view (functional requirements); starts with a short descriptive **use case name**
- **Key metrics:** cost savings, time savings, quality improvement (accuracy, coherence), customer satisfaction (CSAT, **NPS**, sentiment), productivity gains (output volume, error rates, completion time)
- **Approaches:** process automation, augmented decision-making, personalisation, creative content generation, exploratory analysis/innovation
- **Use case evaluation — 5 steps (in order):**
  1. **Idea generation** → cross-functional workshops, competitor trends, customer feedback
  2. **Initial screening** → high-level filter of infeasible / low-impact ideas → shortlist
  3. **Detailed assessment** → prioritise by **business impact, technical feasibility, org readiness**
  4. **Portfolio balancing** → mix of **quick wins + transformative** use cases; resources & risk
  5. **Roadmap development** → success metrics, timelines, resources

### GenAI application lifecycle & deployment considerations
- **1 Define use case → 2 Select FM → 3 Improve performance → 4 Evaluate → 5 Deploy** (iterative)
- After deployment: collect **user feedback, usage data, metrics** → retrain / fine-tune / update; stages get **revisited** as needs change or new models appear
- Deploying: consider architecture, scalability, security, UX
  - **Cost** — pay for what you use, no minimums
  - **Regions** — model availability is **Region-limited**
  - **Quotas** — check service quotas for the account
  - **Security** — in AWS = **shared responsibility**; model accessed outside AWS → evaluate **data leaving the account**

### AWS GenAI stack, Layer 1 apps & SageMaker Unified Studio

| Layer | What | Services |
| --- | --- | --- |
| **1 – Applications** (no deep expertise) | Ready-to-use AI apps | **Amazon Quick Suite**, **Kiro**, **Amazon Connect AI agents** |
| **2 – Tools to build** | Build your own GenAI apps/agents | **Amazon Bedrock**, **Bedrock AgentCore**, **Bedrock Knowledge Bases** |
| **3 – Infrastructure** | Train / run FMs at scale | **SageMaker (Unified Studio)**, **Trainium**, **Inferentia**, **Trainium3** |

- Start at the layer that fits; move between layers as needs grow
- Agentic view of the stack: **Applications** (Kiro, Amazon Quick, AWS Transform, Amazon Connect, Marketplace) → **SDKs** (Strands Agents, Nova Act) → **Bedrock + AgentCore** → **Infrastructure**
- **Bedrock** = fully managed, many FMs via **one API**, serverless, private customisation, enterprise security · **AgentCore** = production agent platform · **Knowledge Bases** = fully managed RAG + **GraphRAG**
- **Trainium** → training, up to **4x faster / 2x lower cost** vs GPU · **Inferentia** → inference, up to **30% lower cost/inference**, high throughput low latency · **Trainium3** → next-gen training (3nm, ~40% more energy-efficient, 2x compute vs Trainium2) · also GPU P/G families
- Compute options: serverless (Lambda, Bedrock, Step Functions) · containers (ECS/Fargate, EKS)
- **Quick Suite** → GenAI search + agents + automations + BI over company data; **40+ connectors** (SharePoint, Confluence, S3, Salesforce) — no data movement
- **Kiro** → AI coding; also **auto-generates unit + integration tests** (incl. legacy code)
- **Connect AI** → analyses **100% of conversations** (voice + chat): sentiment, categorisation, compliance
- **SageMaker Unified Studio** → one governed workspace for data eng + analytics + ML + GenAI (Kiro-accelerated)
  - **SageMaker Lakehouse** unifies S3 data lakes + Redshift + third-party sources
  - Data processing (**Athena, EMR, Glue**) · model dev · GenAI dev (**Bedrock** inside) · SQL analytics (**Redshift**) · governance & security

### FM & LLM fundamentals (glossary)
- **AI** ⊃ **ML** (learns patterns from data) ⊃ **Deep Learning** (multi-layer neural nets) · **GenAI** = creates new content rather than classify/predict
- Supervised / unsupervised / reinforcement learning = labelled data / structure in unlabelled data / learning from rewards
- **FM** = very large model pretrained on huge corpora via **self-supervised learning** (labels auto-generated from data structure), adaptable to many downstream tasks (traditional ML = only the task trained for); **LLM** = text-only subset of FMs that predicts the next token · **multimodal** model = more than one modality (Nova Lite/Pro, Claude)
- Stages: **training** (self-supervised + **RLHF** to align with human preferences) → **fine-tuning** (adjust weights) → **prompt engineering** (no weight change)
- **Text-to-text** = LLMs (summarise, extract, Q&A, generate); modern LLMs don't need classic NLP steps (tokenise/stem/lemmatise/stop-words)
- **Text-to-image** = **diffusion**: forward diffusion adds noise → **U-Net** predicts noise → reverse diffusion denoises into new image (Nova Canvas, Stable Diffusion)
- LLM layers: **embedding layer** (text → vectors) · **feedforward layer** · **attention mechanism** (core of transformer; self-attention lets each token weigh every other → explains long-context cost)
- **Token** ≈ ¾ English word; **1,000 tokens ≈ 750 words**; cost/limits per token; **tokeniser differs per model** → counts differ
- **Context window** = max tokens (input + output) per request; overflow → truncation errors
- **Parameters/weights**: more ≈ more capable, slower, pricier · **pre-training** = out of scope to perform · **base vs instruct/chat** model → pick instruct for apps
- **TTFT** = time-to-first-token (streaming improves perceived latency) · **hallucination** = fluent but false output · **knowledge cutoff** → why RAG exists · **reasoning models** / extended thinking = more tokens, cost vs quality
- **Pre-trained FM limitations:** **hallucinations** (lacks industry/product knowledge) · **stuck in time** (offline training) · **lack of enterprise context** (public data only) · **lower domain accuracy** · **low interpretability** (no sources)
- LLMs are **stateless** (app must pass conversation as context) and weak on **out-of-domain data** + **remembering conversational context** → hallucinations; one prompt often not enough → **chaining**
- FM use cases: chatbots/virtual assistants, call analytics, agent assist, content creation, code, summarisation

### Anatomy of a GenAI application & common patterns
- Centre: **FM**; around it → frontend web/mobile app, **FM interface**, **ML environment**, model training, **enterprise datasets**, **vector database**, **text & image embeddings**, **long-term memory / prompt history store**; **governance & security** integrated into **every** component
- **FM interface** = usually a managed **API**; self-hosting needs ML env with **accelerated compute instances**; prompts + inference params sent via API
- FMs trained on **public** data by design → need **enterprise datasets** as context for tailored answers
- **Too much context** → confusion/hallucinations, longer prompts, **higher cost + latency** → pass the *right amount*: one known doc → pass it directly · pick docs from a store → **vector search**; retrieved context **reduces hallucinations** (= RAG)
- **Frontend** → constructs prompts, calls FM API, **pre-/post-processing**, graceful failure handling
- **Text/code generation** → prompt via playground or API; code: SQL gen, explain/translate, bug fixing, optimisation
- **Summarisation** → small doc: pass whole text · large doc: chunk + chain (map-reduce, see [03 · Prompt engineering](03-prompt-engineering-prompt-management.md))
- **Question answering**: **generic base** FM (no retrieval, **lacks personalisation** → bad for medical/legal/financial) vs **personalised/specific** → **RAG** (solves token-limit problem)
- **AI assistant** → user query + **chat history** → FM → response: **Basic** = zero-shot · **Prompt template** = context in template · **Persona** = defined role (e.g. career coach) · **Contextual-aware** = context from **external file via embeddings**

### Data foundations — 3 pillars
- **Data quality** → accuracy, completeness, consistency; validation pipelines + monitoring (poor quality = biased models, lost trust)
- **Data governance** → access/usage/retention policies, **lineage**, audit → compliance
- **Data architecture** → scalable batch + real-time; data lake + warehouse; versioning, integration

### Deterministic workflows vs agents — right-sizing
- ~**95% of production decision paths should be deterministic**, ~5% need agentic judgment; right-sizing (Step Functions + Lambda for predictable flows) → up to **60% cost savings** vs all-agentic
- Trend: deterministic orchestration (**Step Functions**) runs the workflow; GenAI/agents called as **targeted tool calls** → cheaper, reliable, easier to debug
- 4-question test — use an **agent** if: workflow **unpredictable** · needs **reasoning/exceptions** · needs **human-like dialogue** · must **learn/improve**. Otherwise → traditional tools
- Example: **IT ticket resolution → agent**; **password reset → traditional automation**
- Workflow = fixed predefined steps vs agent = model decides the steps → prefer workflows when steps are known

### MLOps, CI/CD & success metrics
- **MLOps lifecycle:** version models + data + code · automated test/validation · **staged deploy with rollback** · monitoring & alerting
- **CI/CD:** automated train/eval · IaC (**CloudFormation**) · automated security/compliance checks · plug into DevOps
- **Technical metrics:** accuracy/precision/recall, latency/throughput, availability, **cost per inference + TCO**
- **Business metrics:** adoption/engagement, process efficiency, revenue/cost savings, CSAT

### AI-DLC in operations
- AI-DLC (AI-Driven Development Lifecycle) extends into operations: **AI does the repetitive work, humans approve critical decisions, AI executes** — same pattern across deployment, observability, incident response → reactive firefighting → proactive, consistent ops; operations = extension of the collaborative, **human-validated** development approach → **end-to-end AI-DLC consistency**
- **AI-Driven Deployment** → AI **packages Units of Work into independently deployable components**, manages cross-environment config (dev → staging → prod); **developer approves the deployment plan before any rollout**
- **AI-Driven Observability** → use AI-DLC to **build the monitoring system itself** (vendor selection, metrics collection, dashboards, alerting); human oversight for compliance; why → **consistency** (same approach builds the app and its supporting systems)
- **AI-Driven Incident Response** → AI **analyses logs + telemetry**, identifies patterns, **recommends** actions → **human reviews, applies judgment, decides → then AI executes**; code fix needed → hand over to **development teams**, who use AI-DLC to diagnose, fix, deploy

## Exam triggers
- "AI recommends an action in prod" → **human approves first, AI executes**
- "Who approves rollout?" → **developer approves AI's deployment plan**
- "AI packages modules" → **Units of Work → independently deployable units**
- "Why AI-DLC for monitoring?" → **consistency**
- "Incident needs a code change" → **back to dev team via AI-DLC**

## Flashcards
<details>
<summary><b>Q:</b> Should an application use a base model or an instruct/chat model?</summary>

An instruct/chat model, which is tuned to follow instructions; a base model is a raw next-token predictor.

*D0 · Base vs instruct*
</details>

<details>
<summary><b>Q:</b> Which are Bedrock NLP application use cases: text summarisation, serverless file processing, text generation, backup/restore, question answering, text-to-speech?</summary>

Text summarisation, text generation and question answering.

*D1 · Bedrock NLP use cases*
</details>

<details>
<summary><b>Q:</b> What happens when too much context is passed to an FM?</summary>

Confusion/hallucinations, longer prompts, higher cost and latency – pass the right amount, using vector search to select relevant docs.

*D1 · Context size tradeoff*
</details>

<details>
<summary><b>Q:</b> What is a context window and what happens when it overflows?</summary>

The maximum tokens (input + output) per request; overflow causes truncation errors.

*D0 · Context window*
</details>

<details>
<summary><b>Q:</b> Which AWS services add data quality checks to a GenAI data preparation pipeline?</summary>

AWS Glue Data Quality and AWS Glue DataBrew.

*D1 · Data quality checks*
</details>

<details>
<summary><b>Q:</b> What is a diffusion model?</summary>

An image/video generator that learns to remove noise step by step (e.g. Nova Canvas, Stable Diffusion).

*D0 · Diffusion models*
</details>

<details>
<summary><b>Q:</b> What is an embedding and what does dimensionality trade off?</summary>

A numeric vector capturing meaning (similar meaning = nearby vectors). Higher dimensionality (e.g. 1024 vs 256) is richer but costs more storage and latency.

*D0 · Embeddings*
</details>

<details>
<summary><b>Q:</b> What is a foundation model versus an LLM?</summary>

An FM is a large model pre-trained on broad data and adaptable to many tasks; an LLM is an FM specialised in text that predicts the next token.

*D0 · Foundation model*
</details>

<details>
<summary><b>Q:</b> What are the five stages of the GenAI application lifecycle in order?</summary>

Define use case → Select FM → Improve performance → Evaluate → Deploy; iterative, with feedback loops after deployment.

*D1 · GenAI lifecycle*
</details>

<details>
<summary><b>Q:</b> A company with limited AI expertise wants quick wins from GenAI without custom development. Which layer of the AWS GenAI stack should it start at?</summary>

Layer 1 – Applications (e.g. Amazon Quick Suite, Kiro, Amazon Connect AI agents). Layer 2 (Bedrock, AgentCore, KB) is for building your own apps; Layer 3 (SageMaker, Trainium, Inferentia) is infrastructure.

*D1 · GenAI stack layers*
</details>

<details>
<summary><b>Q:</b> What is a golden dataset?</summary>

A curated set of questions with known good answers used for regression testing.

*D0 · Golden dataset*
</details>

<details>
<summary><b>Q:</b> Compare on-demand, provisioned throughput and batch inference.</summary>

On-demand: pay per token, subject to quotas/throttling. Provisioned throughput: reserved model units for guaranteed capacity, required for some custom models. Batch: asynchronous bulk jobs at lower cost, not real-time.

*D0 · Inference modes*
</details>

<details>
<summary><b>Q:</b> Why does RAG exist in relation to knowledge cutoff?</summary>

Models know nothing after their training data cutoff; RAG retrieves current/private data into the prompt to ground answers.

*D0 · Knowledge cutoff*
</details>

<details>
<summary><b>Q:</b> What are PEFT and LoRA?</summary>

Parameter-efficient fine-tuning trains a small set of extra weights; LoRA/adapters are popular PEFT methods - small, swappable add-ons to a base model.

*D0 · PEFT / LoRA*
</details>

<details>
<summary><b>Q:</b> What is the difference between prompt injection and a jailbreak?</summary>

Prompt injection is user input that overrides your instructions; a jailbreak is input crafted to bypass safety rules.

*D0 · Prompt injection*
</details>

<details>
<summary><b>Q:</b> Business users need GenAI search over company data plus agents, automations and BI, using 40+ connectors without moving data. Which service?</summary>

Amazon Quick Suite.

*D1 · Quick Suite*
</details>

<details>
<summary><b>Q:</b> What is the ReAct pattern and what is return of control?</summary>

ReAct is a Reason -> Act -> Observe loop; return of control means the agent hands the tool call back to your application to execute.

*D0 · ReAct agents*
</details>

<details>
<summary><b>Q:</b> What are RLHF and DPO used for?</summary>

Aligning model outputs with human preferences.

*D0 · RLHF / DPO*
</details>

<details>
<summary><b>Q:</b> Teams switch between disconnected data, ML, GenAI and BI tools and want one shared governed workspace. What should they use?</summary>

Amazon SageMaker Unified Studio (with SageMaker Lakehouse unifying S3 data lakes, Redshift and third-party sources; Bedrock built in).

*D1 · SageMaker Unified Studio*
</details>

<details>
<summary><b>Q:</b> How should a ThrottlingException (429) be handled?</summary>

Retry with exponential backoff plus jitter.

*D0 · Throttling*
</details>

<details>
<summary><b>Q:</b> What is a token and why does it matter on the exam?</summary>

A chunk of text (~3/4 of an English word); cost and limits are per token, and tokenisers differ per model so counts vary across models.

*D0 · Tokens*
</details>

<details>
<summary><b>Q:</b> Roughly how many words are 1,000 tokens?</summary>

About 750 words.

*D1 · Tokens to words*
</details>

<details>
<summary><b>Q:</b> What is the difference between top-p and top-k sampling?</summary>

Top-p samples from the smallest token set whose probabilities sum to p; top-k samples from the k most likely tokens. Lower values = more focused/predictable.

*D0 · Top-p vs top-k*
</details>

<details>
<summary><b>Q:</b> What is self-attention and why does it matter for long contexts?</summary>

The transformer mechanism letting each token weigh every other token; it explains why long contexts are costly.

*D0 · Transformer attention*
</details>

<details>
<summary><b>Q:</b> A workshop produced 50 GenAI ideas. What is the next step, and what follows once there is a shortlist of 15?</summary>

Initial screening (filter infeasible/low-impact), then detailed assessment (business impact, technical feasibility, org readiness), followed by portfolio balancing and roadmap.

*D1 · Use case evaluation*
</details>

<details>
<summary><b>Q:</b> What is the difference between zero-shot, few-shot and chain-of-thought prompting?</summary>

Zero-shot: instruction only. Few-shot: include input->output examples. Chain-of-thought: ask the model to reason step by step before answering.

*D0 · Zero vs few-shot*
</details>

---
[← Back to index](README.md)
