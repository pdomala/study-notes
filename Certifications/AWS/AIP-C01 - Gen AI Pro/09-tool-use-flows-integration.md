[← Back to index](README.md)

# 🔀 09 · Tool use, Flows & integration patterns

> Exam domain: D2

## Key points
- **Tool use (Converse)**: model returns **stopReason = tool_use** + `toolUse` (toolUseId, name, input) → **your app runs the tool and handles errors** → reply as a **user** message with `toolResult` (same toolUseId, `status: error` on failure) → finished = **end_turn**.
- Parallel tool calls → return **all results in one toolResult message**. Force a tool in tests → **toolChoice**. Wrong/missed tool → improve **toolSpec.description**.
- **Bedrock Flows** = visual low-code chain of prompt/agent/KB/Lambda/Lex nodes. Branch → **Condition** · array items → **Iterator + Collector** · repeat → **Loop**.
- Flows deploy: **DRAFT** + **TSTALIASID** for testing → publish **immutable version** → **alias** → **InvokeFlow**; roll back by repointing the alias.
- Messaging: **SNS** = fan-out broadcast · **SQS** = buffer / reliable queue · **EventBridge** = content-based routing · (MQ = AMQP/JMS legacy migration; ⚠️ listed as out of scope in the exam guide). GenAI call longer than API Gateway's **29 s** → **SQS buffer + DynamoDB by message ID** (poll) or **WebSocket** (push).
- Decoupling: **temporal** (queue at peak) · **spatial** (different Regions) · **synchronization** (not online together) · **interface** (standard message format). Transient FM failures → **exponential backoff + circuit breaker + model fallback**.
- Step Functions: same input, different branches → **Parallel** · same steps per array item → **Map** (MaxConcurrency 0 = unlimited, 1 = sequential) · **Standard** ≤ 1 yr, priced per transition, required for **waitForTaskToken** · **Express** ≤ 5 min, 100K+/s · cut cost → **Express nested inside Standard** · rollback → **saga** via Catch/Choice · type is fixed → copy to a new state machine.

## Notes
### Why tool use — evolution & model support
- **Tool use / function calling** = FM returns a **structured JSON request** to run an external tool; the **application** runs it and sends the result back
- FMs are standalone and only know their **training data** → no real-time data (weather, latest financials) and **no internet access by default**
- Ways to augment without retraining: **file upload** in prompt · **RAG** (vector-matched chunks + prompt) · **few-shot** learning · **function calling**
- Function calling popularised with **GPT-4 (Mar 2023)**; model emits JSON → app runs function → result back to model → may call more tools
- Tools turn chatbots into **assistants** (current data) and **agents** (act on the user's behalf via APIs)
- Bedrock offers FMs from **AWS and third parties** (Nova, Claude, Llama, Mistral, Cohere, AI21) + **custom imported models**; **not every model supports function calling** → check "supported models and model features"
- Typical positioning: Claude = strong at complex tool orchestration/multi-step reasoning · Llama = code generation / dev-focused tool use
- Bedrock benefits for tool use: **fine-grained IAM**, **VPC integration**, **CloudTrail** audit, native integration with **S3, DynamoDB, Lambda** (no extra auth/middleware), auto scaling, regional availability

### Tool-use concepts & toolSpec
- **Tool** = external capability the FM can leverage (e.g. calculator for precise maths)
- **API** = bridge between FM and tool; for Bedrock → **Converse API** (bedrock-runtime, one interface for all message models)
- **toolSpec** = `name` + `description` + `inputSchema.json` (JSON Schema: `type`, `properties`, `required`)
- **`toolSpec.description` is what the FM uses to decide whether to call the tool** → say what it does, when to use / not use it, and constraints
- Each `properties.<x>.description` should explain the parameter and its effect · `required` = mandatory inputs
- **AI application** implements Converse: passes tool definitions + prompt, **invokes the tools**, **handles tool errors**, returns the final answer to the user

### Converse tool-use loop — requests, results, errors & parallel calls
- Converse = **write once, works across all models that support messages**; model-specific inference params can still be passed (e.g. Llama 3.1 tool prompt guidance)
- Define tools in `toolConfig = {"tools": [{"toolSpec": …}, …]}` → multiple tools registered per request (e.g. `user_location`, `current_weather`)
- `messages` = array of `{role, content:[{text}]}` → can hold multiple messages
- Call: `boto3.client('bedrock-runtime').converse(modelId=…, messages=…, toolConfig=…)`
- **Force a tool** (e.g. Claude 3) → `toolChoice` in the tool config → useful for **testing tools in development**
- Model wants a tool → `stopReason = "tool_use"`; `output.message.content[].toolUse` = `{toolUseId, name, input}` → app should **loop through all toolUse entries**; `input` values follow the `inputSchema` the model was given
- **Returning results**: send a **`role: user`** message with `toolResult` blocks `{toolUseId, content:[{json:…}] or [{text:…}]}`
  - **All results for the previous turn's toolUse blocks go in the same message**
  - Error → `toolResult` with text (e.g. "City not found.") and **`status: "error"`** — the app handles errors, not the FM
  - **Append** the result message to `messages` **immediately after** the assistant message (keeps conversation state), then call converse again with the same `toolConfig`
- Sequential chain: `user_location` → city → `current_weather(city)` → final answer with **`stopReason = "end_turn"`**
- **Parallel tool use**: one response can contain **multiple toolUse blocks** (e.g. `current_weather` for Seattle **and** Kansas City) → app runs each, then **combines all results into one `toolResult` message**
- Tools can be called **sequentially or in parallel**, and the **same tool many times**

### Amazon Bedrock Flows — what it is & node types
- GenAI workflow = structured process turning inputs into outputs: **data collection/prep → model selection/training → generation → post-processing** (refine raw output: resolution, grammar, coherence)
  - Model families mentioned: **GANs, VAEs, Transformers**; training is compute-heavy (GPUs)
  - Challenges: **consistent quality**, compute cost, ethical/legal (bias, misuse), continuous retraining → use **feedback loops**
- **Bedrock Flows** = **visual, low-code** builder: drag-and-drop nodes on a canvas, configure each, link them via **input/output connections** to define the sequence (vs code-heavy Step Functions / LangGraph; AgentCore vs Flows → See [07 · Agents](07-agents-multi-agent-frameworks.md))
- Integrates **FMs, Agents, Knowledge Bases, Guardrails** + **Lambda, Lex** (and S3) → **orchestrator** of data between components; built on **serverless/managed** services → scalable, cost-effective
- **Node types**: **Flow input / Flow output** · **Prompt** · **Agent** · **Knowledge base** · **Lambda** · **Lex** · **S3 storage / S3 retrieval** · **Condition** (branching) · **Iterator + Collector** (process array items, gather results) · **Loop** (repeat until condition) · **Inline code**
- **Why Flows**: **ease of use** (technical + non-technical users) · **enterprise integration** with AWS services (secure, compliant) · **speed to market** (prototype → test → deploy)

### Bedrock Flows — create, configure & test
- Prerequisites: **IAM permissions** for Bedrock; enable **model access** for the FM (e.g. Claude 3 Haiku); model availability **varies by Region**, some models need extra approval / specific pricing
- Bedrock console → **Flows → Create flow** → name, description, **service role** (auto-create new or use existing) · optional **customer managed KMS key**
- Starter canvas = **Flow input → Prompt node → Flow output**
- Prompt node: **Define in node** (inline) or use a **Prompt Management** prompt (centrally versioned); choose model; message uses `{{genre}}` / `{{number}}` → **variables become node inputs**
- Flow input **Type = Object** → invocation expects JSON e.g. `{"genre":"pop","number":3}`; input fields map to node inputs
- Prompt node output = `modelCompletion` (**string**, can't be modified) → connect to Flow output `document`
- **Save** → flow prepared for testing
- Edit flow details (name, description, service role) → **Edit** in Flow details · nodes/connections → **Edit in Flow builder** → Save / Save and exit
- Test pane (right side) → paste JSON input → **Run** → output appears; malformed JSON = syntax error; tests run against the **working draft** via **TSTALIASID**

### Bedrock Flows — versions, aliases & APIs
- New flow → **working draft (DRAFT)** + test alias **TSTALIASID** pointing to it; all edits go to the draft
- **Version** = **immutable snapshot**, numbered from **1** → console: Versions → **Publish version**; lifecycle: create / view / delete
- To change production → **new version from draft**, then point the alias to it (a deployed version can't be edited)
- **Alias** = stable named **pointer to a version**; apps send **InvokeFlow** requests (flow ID + alias ID) to the alias → switch versions without app changes; **rollback = repoint alias to an earlier version**
- Create alias → name + description → **create a new version** or **use an existing version**; lifecycle: create / view / modify / delete
- Everything also doable via **Bedrock APIs**: **CreateFlow**, **CreateFlowVersion**, **CreateFlowAlias**, **InvokeFlow**

### Messaging for agentic AI & the 4 decoupling types
- Agents coordinate through **messaging, not direct calls**
- Messaging enables agents via: **event broadcasting** (share state without direct connections) · **asynchronous processing** (react to events, no polling) · **reliable delivery** (guarantees) · **scalable coordination** (add instances horizontally)
- **4 decoupling types**:
  - **Temporal** — different time scales; queued when busy → resilience at peak load
  - **Spatial** — different **Regions/AZs** → geo distribution, DR, hybrid/multi-cloud
  - **Synchronization** — no need to be online at the same time; messages persist → maintenance without downtime
  - **Interface** — standard message formats → change/replace agents built on different tech
- Decoupling lets producer and consumer **scale independently**

### SNS vs SQS vs Amazon MQ vs EventBridge
- **SNS** = high-throughput **pub/sub** fan-out, SMS, email, mobile push → broadcasting & coordination
  - Notify when SageMaker training/inference or **Rekognition** jobs finish/fail · operational alerts · multi-channel (SMS, email, HTTP, SQS, Kinesis, Lambda)
- **SQS** = **queue** storing messages between components at any volume → reliable task processing
  - Batch-inference task queue · decouple multi-service workflows (Rekognition result → SQS → Lambda) · ETL / enrichment
- **Amazon MQ** = managed **ActiveMQ / RabbitMQ** broker; standard protocols **JMS, NMS, AMQP, STOMP, MQTT, WebSocket** → easy migration, hybrid, enterprise integration
  - Integration layer across microservices/legacy · event sourcing · ETL · route via EventBridge
- **Routers** (abstract producers from consumers) = **EventBridge, SNS** · **Stores/buffers** (hold until consumer ready) = **SQS, MQ**
- Example enterprise workflow: **MQ ingests real-time data** → Kinesis → SageMaker real-time inference → **SNS notifications** → SQS + Lambda batch → S3

### Serverless GenAI async & batch patterns (SNS/SQS/EventBridge)
- **Buffered asynchronous request/response** → time-intensive individual requests needing precise output (text-to-video/music, medical/scientific analysis, virtual worlds)
  - **REST + SQS**: API Gateway → SQS → Lambda/EC2/Fargate batches → DynamoDB record → LLM → result stored by **message ID**; frontend **polls** with message ID → avoids **API Gateway 29 s** timeout (AppSync GraphQL also works)
  - **WebSocket + SQS**: same, but middleware **pushes the result back** over API Gateway WebSocket → **no client polling**
  - **Multimodal parallel fan-out**: **SNS or EventBridge** rules send sub-tasks to multiple LLMs/agents/data sources in parallel → lower total generation time (SNS → SQS fan-out)
- **Non-interactive batch** → large volumes, scheduled or event-triggered, no user waiting (bulk image enhancement, monthly reports, weekly review analysis) → **Step Functions / Glue** pipeline (e.g. SQS → EventBridge Pipes → Step Functions); needs repeatability, parallelism, dependency management

### Integrating messaging with AI services — routing, retries & circuit breakers
- Messages trigger **Bedrock / SageMaker** calls; **queue requests** to absorb **LLM latency + API rate limits**
- Conversational agents: store state/session; link turns with **message correlation IDs**
- Content workflows: upload event → analysis pipeline; route by content type/language/complexity using **message attributes**
- Errors: **exponential backoff + circuit breaker**; **fallback to alternate models** via message routing
- Agent request volume (10–100×) → rate limiting + circuit breakers → See [07 · Agents](07-agents-multi-agent-frameworks.md) (Reliability, safeguards & HITL)
- SageMaker: distribute inference across instances (CloudWatch-driven routing) · **A/B testing** via message routing = gradual rollout of model versions (the expected answer, rather than canary) · event-driven retraining (data arrival, drift, schedule) · topics broadcast training-complete · content-based routing for multi-model

### Messaging case studies — Clario & Snoonu (Amazon MQ)
- **Clario** (clinical document analysis): complex, non-standard trial **protocol documents** (tables, images, diagrams) took weeks to review
  - **4 pillars**: **Parsing** (Textract, Comprehend) · **Retrieval** (embeddings + **OpenSearch** vector engine, chunking strategies) · **Prompting** (**zero-/few-shot**, Bedrock) · **Generation** (choose LLM by context size, reasoning, latency)
  - Flow: on-prem → **Direct Connect** (encrypted) → S3 (SSE) → orchestration engine on **EKS** → Textract → OpenSearch → **Amazon MQ launches async document-classification task** → Classification agent (Bedrock) → Protocol Analysis agent (graph config in **RDS**, custom models on **SageMaker** + Bedrock) → structured results in RDS
  - Best practices: iterate incrementally · validate with representative test sets · **optimise the 4 pillars before fine-tuning / continued pre-training** · tailor per use case
- **Snoonu** (personalised product discovery): popularity rules → one global **Amazon Personalize** model → **separate models per vertical** (Marketplace, Food, Groceries) = key precision win
  - Training cadence 3 weeks → weekly → **daily** (Personalize usually recommends weekly)
  - Recipes: **user-personalization** (home/cart/PDP) · **similar-items** (frequently bought together) · **personalized-ranking** (category pages)
  - Real time: Amplitude → **Kinesis Data Streams** → Lambda (schema validation, **PutEvents** to event tracker, trigger refresh) · **ElastiCache (Redis)** with event-triggered invalidation instead of TTL · postprocess for geo/stock filters
  - **Amazon MQ (RabbitMQ) logs recommendation requests** → compare pre/post filtering, monitor
  - Results: +1,600% add-to-cart (groceries) · GMV 47× model cost · +30% basket size

### Step Functions — why patterns, Parallel state & Map state
- Without patterns every step runs **sequentially** → slow; no handling for multi-item orders, failures/rollback, or cost
- Running example: online retailer order processing (verify inventory, payment, delivery; notify fulfillment, customer); decide which steps are **independent** (parallel) vs **dependent** (sequential) — never ship in parallel with a billing check that might be denied
- **Parallel state** → **fixed number of branches**, **same input** to every branch; waits until **all branches reach a terminal state** before `Next`
  - Fields: **Branches** (required; each has `StartAt` + `States`) · `ResultPath` · `ResultSelector` (applied **before** ResultPath, replaces the result) · `Catch` (fallback state on error) · `Retry`
  - Examples: look up address + phone in parallel; verify payment + reserve inventory in parallel; Parallel states can be **nested**
  - Scale intuition: 1,000 log files × 1 min = 16+ h sequential vs ~1 min parallel
- **Map state (dynamic parallelism)** → same steps **for each element of an input array**; a **sub-state machine looping over the array**, returns an **array of outputs**
  - Messaging uses: **fan-out** (e.g. SQS messages → one Lambda each) · **scatter-gather** (broadcast, then aggregate — e.g. transcode 10 × 500 MB files, join into 5 GB)
  - Fields: **Iterator** = the **only required field** (sub-workflow) · **ItemsPath** (where the array is, Map-specific) · **MaxConcurrency** · ResultPath · ResultSelector · Catch · Retry
  - **MaxConcurrency 0 = default = no limit** · **1 = one at a time, in order** · 10 = max 10 concurrent
  - Example: `ProcessAllItems` Map over `$.items` (MaxConcurrency 3) → CheckAvailability (Retry on TimeOut, backoff 2) → PrepareForDelivery → StartDelivery → SendOrderSummary

### Step Functions — nested workflows & Standard vs Express
- Task state starts another state machine via `arn:aws:states:::states:startExecution` (Step Functions calls **its own API**)
- Benefits: separate high-level vs task-level workflows · reuse instead of repeating · **library of modular workflows** · simpler to edit/troubleshoot · **cut cost by mixing Standard + Express**
- Link child to parent: pass `"AWS_STEP_FUNCTIONS_STARTED_BY_EXECUTION_ID.$": "$$.Execution.Id"` (context object) → console links parent ↔ child (optional but aids tracing)
- Retry on `StepFunctions.ExecutionLimitExceeded`
- Example: separate human-intervention fulfillment workflow; **Express children inside a Standard parent** to save cost

|  | **Standard** | **Express** |
| --- | --- | --- |
| Duration | up to **1 year** | up to **5 min** |
| History | visual + full history in console | **CloudWatch Logs** |
| Rate | 2,000+/s | **100,000+/s** |
| Pricing | **per state transition** | per **execution count, duration, memory** |
| Use for | long-running, durable, **auditable** (EMR, payments) | high-volume event processing (IoT ingest, streaming, mobile backends) |
| Wait-for-callback | ✅ | ❌ |

### Step Functions — orchestration + choreography (EventBridge)
- Split the app into **bounded contexts** (DDD subdomains): Retail (order, inventory, payment) · Fulfillment · Customer support
- **Orchestration** = Step Functions coordinates steps **within one bounded context**; only knows its own workflow; result = an event
- **Choreography** = **EventBridge** routes events **between** subdomains via rules → subscribers
- Better **together**: workflow runs tasks → `arn:aws:states:::events:putEvents` → EventBridge rules fan out (works in Standard + Express; can be **cross-account**)
- Removes the need for a **Lambda to call EventBridge**; consumers subscribe to events instead of polling `DescribeExecution` → **decoupled**
- Integration patterns after putEvents: **Request-Response** (move on after HTTP response — Standard + Express) · **Wait-for-callback** `.waitForTaskToken` + `$$.Task.Token` (**Standard only**)
- Example: home delivery order → direct-to-customer shipping workflow; in-store pickup order → store pickup workflow, chosen by EventBridge rules

### Step Functions — saga pattern & best practices
- Microservices can't use **ACID** transactions → risk of partial transactions
- **Saga** = **failure-management** pattern: each step publishes an event; on failure run **compensating transactions** to undo completed steps (e.g. release held flight seat + hotel room if payment fails)
- Use when: data consistency across microservices **without tight coupling** · **long-lived transactions** that shouldn't block others · need **rollback** on failure
- Implement with **Choice state** (route on previous result) or **Catch field** (in Task/Parallel/Map; more effort but adds **Retry** before the failure path); `States.ALL` in Catch = catch any error
- Order example: Inventory update fails → **Revert inventory** (DynamoDB) → **Remove order** → Fail
- **Best practices**:
  - **Enable X-Ray tracing** (new or existing state machine) — especially for hard-to-debug **saga + nested**
  - Choose **Standard vs Express up front** — type **can't be changed** after creation; convert via console **"Copy to new"**
  - **Combine patterns**, e.g. **Express workflow inside a Map state** (dynamic parallelism + nested) — expand the sub-workflow instead of chaining inside the Map

## Exam triggers
- "Model needs real-time/external data or must take an action" → **tool use / function calling** via **Converse**
- "Same tool code for any model" → **Converse** (vs InvokeModel model-specific payloads)
- "Model keeps picking the wrong tool / never calls it" → improve **`toolSpec.description`** (+ parameter descriptions)
- "Make sure a specific tool is called (testing)" → **`toolChoice`**
- "How do you know the model wants a tool?" → **`stopReason = tool_use`** · done → **`end_turn`**
- "Match result to request" → **`toolUseId`** · "tool failed" → `toolResult` with **`status: error`** (app handles it, not the FM)
- "Model returned several toolUse blocks" → run all, return **all results in one user message**
- "Who executes the tool?" → **your application** (Lambda, etc.), never Bedrock/the FM
- "Chain prompts + KB + Lambda visually, little code" → **Bedrock Flows** (vs code-heavy → Step Functions / LangGraph)
- "Route based on model output / input value" → **Condition** node
- "Process each item in a list" → **Iterator** (+ **Collector**) · "repeat until done" → **Loop**
- "Reuse versioned prompt templates in a flow" → Prompt node using **Prompt Management**
- "Promote to prod / instant rollback without app changes" → **publish version + alias**, repoint alias
- "Edit a deployed version" → can't: versions are **immutable** → new version from draft
- "Test latest changes" → **TSTALIASID** → working draft
- "App calls a flow" → **InvokeFlow** with **flow ID + alias ID**
- "Encrypt flow" → **customer managed KMS key** · flow permissions → **service role**
- "Broadcast / fan-out to many agents or models" → **SNS** (or EventBridge rules)
- "Buffer requests, reliable task processing, absorb LLM rate limits" → **SQS**
- "AMQP / JMS / MQTT / STOMP, migrate ActiveMQ/RabbitMQ, hybrid" → **Amazon MQ**
- "GenAI request > 29 s via API Gateway" → **API Gateway → SQS → Lambda → DynamoDB by message ID**
- "Avoid client polling for async result" → **API Gateway WebSocket**
- "Scheduled bulk generation (reports, image batch)" → **Step Functions / Glue batch pipeline**
- "Agents in different Regions" → **spatial** · "agents offline at different times" → **synchronization** · "different tech stacks" → **interface** · "peak load queueing" → **temporal**
- "Link turns of a conversation across services" → **correlation ID** · "route by type/metadata" → **message attributes**
- "Transient model/API failures" → **exponential backoff + circuit breaker + model fallback**
- "Independent steps, same input" → **Parallel state**
- "Same steps per item in an array" → **Map state** (Iterator required, ItemsPath, MaxConcurrency)
- "MaxConcurrency 0" → **unlimited** · "1" → **sequential, in order**
- "Reuse workflows / cut cost" → **nested** with **Express children in a Standard parent**
- "Trace child to parent" → `AWS_STEP_FUNCTIONS_STARTED_BY_EXECUTION_ID` = `$$.Execution.Id`
- "Route results to other domains/accounts, decoupled" → **putEvents → EventBridge rules**
- "Wait for an external callback" → `.waitForTaskToken` → **Standard only**
- "Roll back a partial distributed transaction" → **saga** (Choice or Catch → compensating steps)
- "Long-running / auditable / up to 1 yr" → **Standard** · "high-volume / ≤ 5 min / cheap" → **Express**
- "Wrong workflow type chosen" → can't change → **Copy to new**
- "Debug complex/nested flows" → **X-Ray**

## Flashcards
<details>
<summary><b>Q:</b> Bursty traffic is hitting LLM API rate limits and latency spikes. Which messaging service absorbs this and how should transient failures be handled?</summary>

Queue requests in Amazon SQS; handle transient model/API failures with exponential backoff, a circuit breaker and fallback to alternate models via message routing.

*D4 · Absorbing LLM limits*
</details>

<details>
<summary><b>Q:</b> Which characteristics define agentic AI?</summary>

Proactive, adaptable (domain/context aware), collaborative (with humans and other agents) and specialized (hyperspecialised agents) - autonomous and goal-driven rather than reactive/predefined.

*D2 · Agentic AI traits*
</details>

<details>
<summary><b>Q:</b> AI agents must integrate with existing enterprise systems that communicate over AMQP and JMS, with minimal re-engineering. Which service should be used?</summary>

Amazon MQ - managed ActiveMQ/RabbitMQ broker supporting JMS, NMS, AMQP, STOMP, MQTT and WebSocket for easy migration and hybrid integration.

*D2 · Amazon MQ protocols*
</details>

<details>
<summary><b>Q:</b> A text-to-video GenAI request takes several minutes, but it is exposed via API Gateway REST, which times out. What architecture fixes this?</summary>

Buffered asynchronous request/response: API Gateway -> SQS -> Lambda/EC2/Fargate workers -> LLM -> store result in DynamoDB keyed by message ID; the client polls with the message ID, avoiding the 29 s API Gateway limit.

*D2 · API Gateway timeout*
</details>

<details>
<summary><b>Q:</b> A team wants to chain prompts, a Knowledge Base and a Lambda function visually with little code. What should they use?</summary>

Amazon Bedrock Flows — a low-code, serverless visual builder that orchestrates nodes (vs code-heavy Step Functions/LangGraph).

*D2 · Bedrock Flows use*
</details>

<details>
<summary><b>Q:</b> Which error name in a Catch clause catches any error?</summary>

`States.ALL`.

*D2 · Catch all errors*
</details>

<details>
<summary><b>Q:</b> A team created a Standard state machine but now needs Express. What can they do?</summary>

The type cannot be changed after creation; use the console Copy to new to create a new state machine of the other type.

*D2 · Change workflow type*
</details>

<details>
<summary><b>Q:</b> How do you link the turns of a conversation as messages flow across multiple services, and how do you route by content type or language?</summary>

Use message correlation IDs to link turns; use message attributes for content-based routing by type, language or complexity.

*D2 · Correlation IDs*
</details>

<details>
<summary><b>Q:</b> What best practice helps debug hard-to-trace saga and nested Step Functions workflows?</summary>

Enable AWS X-Ray tracing on the state machine.

*D5 · Debug nested flows*
</details>

<details>
<summary><b>Q:</b> Which decoupling type applies to agents in different Regions, agents offline at different times, agents on different tech stacks, and queuing at peak load?</summary>

Different Regions = spatial; offline at different times = synchronization; different tech stacks = interface; peak-load queuing = temporal.

*D2 · Decoupling types*
</details>

<details>
<summary><b>Q:</b> In an example enterprise AI workflow (ingest -&gt; Kinesis -&gt; SageMaker inference -&gt; notifications -&gt; batch -&gt; S3), which service ingests real-time data and which sends notifications?</summary>

Amazon MQ ingests the real-time data (Kinesis is the next processing step); Amazon SNS sends notifications.

*D2 · Enterprise workflow ingest*
</details>

<details>
<summary><b>Q:</b> A flow must route based on a model output or input value. Which node?</summary>

Condition node.

*D2 · Flows branching*
</details>

<details>
<summary><b>Q:</b> The Flow input node type is Object and the prompt uses <code>{{genre}}</code> and <code>{{number}}</code>. What does invocation expect?</summary>

A JSON object, e.g. `{"genre":"pop","number":3}`; prompt variables become node inputs mapped from input fields.

*D2 · Flows input format*
</details>

<details>
<summary><b>Q:</b> Which components can a Bedrock flow integrate beyond FMs?</summary>

Agents, Knowledge Bases, Guardrails, Lambda, Lex, S3 storage/retrieval and inline code — so custom code and external services are supported.

*D2 · Flows integrations*
</details>

<details>
<summary><b>Q:</b> Which Bedrock APIs create a flow, publish a version and create an alias?</summary>

CreateFlow, CreateFlowVersion and CreateFlowAlias.

*D2 · Flows lifecycle APIs*
</details>

<details>
<summary><b>Q:</b> Which Flows nodes process each item in an array and gather the results, and which repeats until a condition is met?</summary>

Iterator + Collector for arrays; Loop node for repeat-until-condition.

*D2 · Flows list processing*
</details>

<details>
<summary><b>Q:</b> How can a flow reuse centrally versioned prompt templates?</summary>

Configure the Prompt node to use a prompt from Bedrock Prompt Management (instead of defining it in the node).

*D2 · Flows prompt reuse*
</details>

<details>
<summary><b>Q:</b> How do you promote a new flow to production and roll back instantly without changing the application?</summary>

Publish a version and point an alias to it; apps call the alias, so rollback = repoint the alias to an earlier version.

*D2 · Flows rollback*
</details>

<details>
<summary><b>Q:</b> How are a flow's permissions and encryption configured?</summary>

A service role (auto-created or existing) grants permissions; an optional customer managed KMS key encrypts the flow.

*D3 · Flows security*
</details>

<details>
<summary><b>Q:</b> Which alias is used to test the latest edits of a flow?</summary>

TSTALIASID, which points to the working draft (DRAFT).

*D2 · Flows test alias*
</details>

<details>
<summary><b>Q:</b> A deployed flow version needs a fix. Can you edit it?</summary>

No — versions are immutable snapshots. Edit the draft, publish a new version, then point the alias to it.

*D2 · Flows versions*
</details>

<details>
<summary><b>Q:</b> During development you need to ensure a specific tool is always called. Which setting?</summary>

`toolChoice` in the tool configuration.

*D5 · Forcing a tool*
</details>

<details>
<summary><b>Q:</b> Which API and identifiers does an application use to run a Bedrock flow?</summary>

InvokeFlow with the flow ID and alias ID.

*D2 · Invoking flows*
</details>

<details>
<summary><b>Q:</b> Which service notifies teams when SageMaker training/inference or Rekognition jobs finish or fail, over SMS, email, HTTP, SQS or Lambda?</summary>

Amazon SNS (multi-channel notifications).

*D2 · Job completion alerts*
</details>

<details>
<summary><b>Q:</b> How do you link a nested child execution to its parent in the Step Functions console?</summary>

Pass `AWS_STEP_FUNCTIONS_STARTED_BY_EXECUTION_ID.$` set to `$$.Execution.Id` (context object) in the StartExecution input.

*D2 · Link child execution*
</details>

<details>
<summary><b>Q:</b> Which Map state field is the only required one, and which field tells it where the input array is?</summary>

Iterator (the sub-workflow) is the only required field; ItemsPath identifies the array.

*D2 · Map state fields*
</details>

<details>
<summary><b>Q:</b> What do Map state MaxConcurrency values 0 and 1 mean?</summary>

0 (the default) means no limit; 1 means items are processed one at a time, in order.

*D2 · MaxConcurrency values*
</details>

<details>
<summary><b>Q:</b> Are Bedrock FMs pre-configured to access the internet for real-time data?</summary>

No — use tool use/function calling (or RAG) to bring in real-time or external data.

*D2 · Model internet access*
</details>

<details>
<summary><b>Q:</b> How do you gradually roll out a new SageMaker model version using messaging?</summary>

A/B testing via message routing between model versions (the expected answer, rather than canary).

*D2 · Model version rollout*
</details>

<details>
<summary><b>Q:</b> Can a model invoke the same tool multiple times and multiple tools in one response via Converse?</summary>

Yes to both; multiple tools can be defined per request and called sequentially or in parallel.

*D2 · Multiple tool invocations*
</details>

<details>
<summary><b>Q:</b> Which error should a parent state machine retry on when starting child executions?</summary>

`StepFunctions.ExecutionLimitExceeded`.

*D2 · Nested retry error*
</details>

<details>
<summary><b>Q:</b> A long-running auditable order workflow contains high-volume short sub-steps. How can cost be reduced while reusing modular workflows?</summary>

Use nested workflows: Express child state machines started from a Standard parent via `arn:aws:states:::states:startExecution`.

*D4 · Nested workflow cost*
</details>

<details>
<summary><b>Q:</b> A company generates monthly reports and bulk image enhancements on a schedule with no user waiting. Which processing pattern and services fit?</summary>

Non-interactive batch processing using a Step Functions or Glue pipeline (e.g. SQS -> EventBridge Pipes -> Step Functions) for repeatability, parallelism and dependency management.

*D2 · Non-interactive batch*
</details>

<details>
<summary><b>Q:</b> A Step Functions workflow in one bounded context must notify other domains (possibly in other accounts) in a decoupled way without a Lambda in between. What should it use?</summary>

The optimized integration `arn:aws:states:::events:putEvents` to publish to EventBridge, whose rules route events to subscribers (choreography), works in Standard and Express and cross-account.

*D2 · Orchestration + choreography*
</details>

<details>
<summary><b>Q:</b> A multimodal request must be split into sub-tasks processed by several LLMs and agents in parallel to cut total generation time. Which services route the sub-tasks?</summary>

SNS (typically SNS -> SQS fan-out) or EventBridge rules to fan sub-tasks out in parallel.

*D2 · Parallel fan-out*
</details>

<details>
<summary><b>Q:</b> When does a Parallel state proceed to its Next state, and in which order are ResultSelector and ResultPath applied?</summary>

Only after all branches reach a terminal state. ResultSelector is applied before ResultPath.

*D2 · Parallel state behaviour*
</details>

<details>
<summary><b>Q:</b> A model response contains multiple toolUse blocks. What should the app do?</summary>

Run each tool and return all results in a single user message containing multiple toolResult blocks.

*D2 · Parallel tool calls*
</details>

<details>
<summary><b>Q:</b> What is the difference between a Step Functions Parallel state and a Map state?</summary>

Parallel runs a fixed number of different branches, all with the same input; Map runs the same steps for each element of an input array (dynamic parallelism) and returns an array of outputs.

*D2 · Parallel vs Map*
</details>

<details>
<summary><b>Q:</b> What is the output of a Prompt node in Bedrock Flows?</summary>

`modelCompletion`, a string that cannot be modified, typically connected to the Flow output `document`.

*D2 · Prompt node output*
</details>

<details>
<summary><b>Q:</b> How must tool results be sent back in Converse?</summary>

As a `role: user` message with `toolResult` blocks carrying the matching `toolUseId`, appended right after the assistant message, then call converse again with the same `toolConfig`.

*D2 · Returning tool results*
</details>

<details>
<summary><b>Q:</b> Which AWS messaging services act as routers and which act as stores/buffers?</summary>

Routers (abstract producers from consumers): EventBridge and SNS. Stores/buffers (hold messages until the consumer is ready): SQS and Amazon MQ.

*D2 · Routers vs buffers*
</details>

<details>
<summary><b>Q:</b> Which TWO Step Functions mechanisms implement the saga pattern for rolling back a partially completed distributed transaction?</summary>

Choice state (route on previous result) and the Catch field (on Task/Parallel/Map, also allowing Retry first) leading to compensating transactions.

*D2 · Saga pattern*
</details>

<details>
<summary><b>Q:</b> A monitoring system must broadcast the same alert to many independent monitoring agents at once. Which AWS messaging service fits best?</summary>

Amazon SNS - high-throughput pub/sub fan-out that broadcasts one message to many subscribers.

*D2 · SNS vs SQS vs MQ*
</details>

<details>
<summary><b>Q:</b> Compare Step Functions Standard and Express workflows on duration, rate, pricing and history.</summary>

Standard: up to 1 year, 2,000+/s, priced per state transition, full visual history, supports wait-for-callback. Express: up to 5 min, 100,000+/s, priced per execution count/duration/memory, history in CloudWatch Logs, no wait-for-callback.

*D2 · Standard vs Express*
</details>

<details>
<summary><b>Q:</b> A tool call fails (e.g., city not found). Who handles it and how is it reported to the model?</summary>

The application handles it, returning a `toolResult` with error text and `status: "error"` — the FM does not handle tool errors.

*D5 · Tool error handling*
</details>

<details>
<summary><b>Q:</b> A model keeps picking the wrong tool or never calls it. What should you improve first?</summary>

The `toolSpec.description` (what it does, when to use/not use, constraints) and the parameter descriptions — the FM uses these to decide.

*D2 · Tool selection fix*
</details>

<details>
<summary><b>Q:</b> Before building function calling on a Bedrock model, what should you verify?</summary>

That the model supports tool use — not every model supports function calling (check supported models and features).

*D1 · Tool support by model*
</details>

<details>
<summary><b>Q:</b> How does an application know the model wants to call a tool, and when it is finished?</summary>

`stopReason = "tool_use"` with toolUse blocks; the final answer has `stopReason = "end_turn"`.

*D2 · Tool-use signal*
</details>

<details>
<summary><b>Q:</b> What are the components of a toolSpec in Converse?</summary>

`name`, `description` and `inputSchema.json` (JSON Schema with `type`, `properties`, `required`).

*D2 · toolSpec structure*
</details>

<details>
<summary><b>Q:</b> A workflow must pause until an external system or human returns a result. Which integration pattern and workflow type are required?</summary>

Wait-for-callback with `.waitForTaskToken` and `$$.Task.Token`; it is supported only in Standard workflows.

*D2 · Wait for callback*
</details>

<details>
<summary><b>Q:</b> In an async SQS-buffered GenAI pipeline, how can results be delivered without the client polling?</summary>

Use an API Gateway WebSocket API so the middleware pushes the result back to the client when processing completes.

*D2 · WebSocket async results*
</details>

<details>
<summary><b>Q:</b> In Bedrock tool use, who executes the tool?</summary>

Your application (e.g., Lambda) — never Bedrock or the FM, which only emits a structured JSON request.

*D2 · Who executes tools*
</details>

---
[← Back to index](README.md)
