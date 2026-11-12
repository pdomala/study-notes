[← Back to index](README.md)

# 🎯 16 · Exam Triggers (rapid drill)

408 de-duplicated exam triggers, grouped by topic. Each cue is something you would spot in a question; expand it to see what to pick. Drill these fast: aim for under 5 seconds each.

## 🧩 Fundamentals & GenAI strategy

<details>
<summary>Answer questions from our docs, up-to-date info</summary>

→ **RAG**
</details>

<details>
<summary>Align with human preferences</summary>

→ **RLHF**
</details>

<details>
<summary>No suitable model exists</summary>

→ **train from scratch (most expensive)**
</details>

## 🤖 Foundation models, Bedrock APIs & inference options

<details>
<summary>Real-time, mobile, interactive</summary>

→ **latency drives FM choice**
</details>

<details>
<summary>Model not available</summary>

→ **check Region**
</details>

<details>
<summary>Factual, deterministic output</summary>

→ **low temperature / top\_p / top\_k**
</details>

<details>
<summary>Creative, diverse output</summary>

→ **high temperature / top\_p / top\_k**
</details>

<details>
<summary>Swap models without rewriting code</summary>

→ **Converse API**
</details>

<details>
<summary>Token-by-token UX / chat streaming</summary>

→ **InvokeModelWithResponseStream / ConverseStream**
</details>

<details>
<summary>Model deprecated, requests failing</summary>

→ **model hit EOL; you must update model ID (no auto-migration)**
</details>

<details>
<summary>Can't create Provisioned Throughput</summary>

→ **model is Legacy**
</details>

<details>
<summary>Niche/industry model not in core catalog</summary>

→ **Bedrock Marketplace**
</details>

<details>
<summary>Real-time voice assistant</summary>

→ **Nova Sonic**
</details>

<details>
<summary>Cheapest text-only model</summary>

→ **Nova Micro**
</details>

<details>
<summary>Long wait for output</summary>

→ **InvokeModelWithResponseStream**
</details>

## ✍️ Prompt engineering & Prompt Management

<details>
<summary>No examples given</summary>

→ **zero-shot**
</details>

<details>
<summary>Include a few labelled examples</summary>

→ **few-shot**
</details>

<details>
<summary>Show reasoning steps</summary>

→ **CoT (chain-of-thought)**
</details>

<details>
<summary>Multiple reasoning paths, pick most consistent answer</summary>

→ **self-consistency**
</details>

<details>
<summary>Explore branches / planning / lookahead</summary>

→ **Tree of thoughts**
</details>

<details>
<summary>Needs external tools/APIs to act, reduce hallucinated facts</summary>

→ **ReAct**
</details>

<details>
<summary>Task library of demos + tools</summary>

→ **ART (Automatic Reasoning & Tool-use)**
</details>

<details>
<summary>Up-to-date / proprietary data, no retraining</summary>

→ **RAG**
</details>

<details>
<summary>Too many few-shot examples for context window</summary>

→ **dynamic example selection by semantic similarity**
</details>

<details>
<summary>Stop generation at a marker</summary>

→ **stop sequences**
</details>

<details>
<summary>Reduce repetition</summary>

→ **frequency/presence penalty**
</details>

<details>
<summary>Instructions must persist across all turns and override user</summary>

→ **system prompt**
</details>

<details>
<summary>Skip Claude's preamble / force format</summary>

→ **pre-fill assistant response + XML tags**
</details>

<details>
<summary>Long document + question</summary>

→ **document first, instructions last**
</details>

<details>
<summary>Input tries to override instructions</summary>

→ **prompt injection**
</details>

<details>
<summary>Output reveals private/system data</summary>

→ **prompt leaking**
</details>

<details>
<summary>Version, compare, reuse prompt templates with variables</summary>

→ **Bedrock Prompt Management (invoke by ARN)**
</details>

<details>
<summary>Summarise doc bigger than context window</summary>

→ **chunk + chain (map-reduce)**
</details>

<details>
<summary>Reuse versioned prompt templates in a flow</summary>

→ **Prompt node using Prompt Management**
</details>

## 🛠️ Model customisation (fine-tuning, distillation, RFT)

<details>
<summary>Domain terminology, highly accurate, cost-effective customisation</summary>

→ **fine-tuning**
</details>

<details>
<summary>Smaller cheaper model from a big one</summary>

→ **distillation (Nova Premier as teacher)**
</details>

<details>
<summary>Labelled examples for one task</summary>

→ **prompt-based fine-tuning**
</details>

<details>
<summary>Lots of unlabelled domain text</summary>

→ **continued pretraining**
</details>

<details>
<summary>Data changes frequently</summary>

→ **RAG, not fine-tuning**
</details>

<details>
<summary>Big model too slow/expensive at high volume, keep accuracy</summary>

→ **model distillation**
</details>

<details>
<summary>Distillation with teacher from Anthropic, student from Meta</summary>

→ **not allowed (teacher and student must be same provider)**
</details>

<details>
<summary>No labelled data, have a grader/reward</summary>

→ **RFT: objective → RLVR, subjective → RLAIF**
</details>

<details>
<summary>Use a fine-tuned Bedrock model in LangChain</summary>

→ **custom model ARN (provisioned)**
</details>

## 📚 RAG, Knowledge Bases, chunking & embeddings

<details>
<summary>Multi-hop root-cause analysis / compliance tracing</summary>

→ **GraphRAG on Neptune Analytics**
</details>

<details>
<summary>Lowest cost, millions of docs, archival but searchable, can tolerate subsecond latency</summary>

→ **S3 Vectors**
</details>

<details>
<summary>Ingestion fails after connecting existing index</summary>

→ **embedding dimension mismatch**
</details>

<details>
<summary>Filtering fails / metadata too large</summary>

→ **move AMAZON\_BEDROCK\_TEXT to non-filterable (nonFilterableMetadataKeys)**
</details>

<details>
<summary>Need to change chunking</summary>

→ **recreate the data source/KB**
</details>

<details>
<summary>Image search by text or image</summary>

→ **Titan Multimodal Embeddings**
</details>

<details>
<summary>Conversation exceeds context window / need audit trail of prompts</summary>

→ **prompt history store**
</details>

<details>
<summary>Search across text, images, video, audio in one space</summary>

→ **Nova Multimodal Embeddings**
</details>

<details>
<summary>Docs full of tables/charts, 100+ languages</summary>

→ **Cohere Embed v4**
</details>

<details>
<summary>Vectors in existing Postgres</summary>

→ **Aurora/RDS + pgvector**
</details>

<details>
<summary>QA needs domain-specific/personalised answers</summary>

→ **RAG, not base FM**
</details>

<details>
<summary>Query embeddings</summary>

→ **vector store**
</details>

<details>
<summary>Managed RAG, minimal code, citations</summary>

→ **Knowledge Bases + RetrieveAndGenerate**
</details>

<details>
<summary>Custom generation / own prompt / LangChain / custom rerank</summary>

→ **Retrieve + InvokeModel**
</details>

<details>
<summary>Multi-turn KB chat</summary>

→ **session ID (built-in session context)**
</details>

<details>
<summary>Answers mix years/departments/tenants</summary>

→ **metadata filter (sidecar .metadata.json + re-sync)**
</details>

<details>
<summary>Retrieved chunks relevant but poorly ordered / too many to FM</summary>

→ **reranker model**
</details>

<details>
<summary>Complex multi-part question gets weak retrieval</summary>

→ **query decomposition/reformulation**
</details>

<details>
<summary>PDFs with tables / scanned docs</summary>

→ **FM parsing**
</details>

<details>
<summary>Nested legal/manual docs</summary>

→ **hierarchical chunking**
</details>

<details>
<summary>Unclear section boundaries</summary>

→ **semantic chunking**
</details>

<details>
<summary>Bespoke split logic</summary>

→ **custom chunking Lambda**
</details>

<details>
<summary>Reduce vector storage cost, keep accuracy</summary>

→ **Titan V2 at 512/256 dims (~99% / ~97%)**
</details>

<details>
<summary>Search images with text</summary>

→ **multimodal embeddings (Nova / Cohere Embed v4)**
</details>

<details>
<summary>Relationship-heavy data</summary>

→ **Neptune Analytics GraphRAG**
</details>

<details>
<summary>Keep KB fresh automatically</summary>

→ **Lambda-triggered ingestion job (schedule/event + queue)**
</details>

<details>
<summary>Search small, return big context</summary>

→ **hierarchical chunking**
</details>

<details>
<summary>Docs with HTML/Markdown/code structure</summary>

→ **structural chunking**
</details>

<details>
<summary>Fixed chunks losing context</summary>

→ **add overlap / recursive character splitting**
</details>

<details>
<summary>Traceable answers / ACL on chunks</summary>

→ **store metadata with chunks**
</details>

<details>
<summary>KB file rejected</summary>

→ **check > 50 MB / unsupported file type**
</details>

<details>
<summary>Ground on company docs</summary>

→ **Knowledge Base**
</details>

## 🔎 Vector search with OpenSearch

<details>
<summary>High QPS, millisecond latency, hybrid / keyword search</summary>

→ **OpenSearch Serverless (not S3 Vectors)**
</details>

<details>
<summary>In-memory open-source vector store</summary>

→ **FAISS/Chroma**
</details>

<details>
<summary>Exact keywords/product codes missed</summary>

→ **hybrid search → OpenSearch Serverless**
</details>

<details>
<summary>Metadata filtering on an OpenSearch Serverless KB</summary>

→ **FAISS engine**
</details>

<details>
<summary>ANN with no training, low latency, high recall</summary>

→ **HNSW**
</details>

<details>
<summary>Bucketed search, reduce memory with PQ</summary>

→ **IVF (needs training)**
</details>

<details>
<summary>Compress vectors 4× in OpenSearch</summary>

→ **byte quantization**
</details>

<details>
<summary>Reduce IVF memory</summary>

→ **product quantization**
</details>

<details>
<summary>Exact keyword/product-code match + meaning</summary>

→ **hybrid search (BM25 + k-NN, normalised)**
</details>

<details>
<summary>Keyword-efficient semantic search via inverted index</summary>

→ **neural sparse search**
</details>

<details>
<summary>Chat over docs with memory</summary>

→ **conversational search (RAG + memory)**
</details>

<details>
<summary>Generate embeddings inside OpenSearch via Bedrock</summary>

→ **neural search pipeline + ML connector**
</details>

<details>
<summary>Pre-computed embeddings in S3 → OpenSearch</summary>

→ **OpenSearch Ingestion**
</details>

<details>
<summary>Bring-your-own OpenSearch Serverless index for KB</summary>

→ **FAISS, dimensions match model, text + metadata fields**
</details>

<details>
<summary>KB hybrid search falls back to semantic</summary>

→ **OpenSearch Serverless index needs a filterable text field**
</details>

## 🕸️ Agents, multi-agent systems & frameworks

<details>
<summary>Change agent config without redeploying</summary>

→ **Inline Agents**
</details>

<details>
<summary>Non-ReAct workflow logic</summary>

→ **Custom Orchestration**
</details>

<details>
<summary>Agents team with a coordinator</summary>

→ **Multi-agent collaboration (supervisor)**
</details>

<details>
<summary>Persist/resume multi-turn state, LangGraph checkpointing, fork session</summary>

→ **Session Management APIs**
</details>

<details>
<summary>Fetch relevant docs for a query</summary>

→ **retriever**
</details>

<details>
<summary>Load docs from S3/DB</summary>

→ **document loader**
</details>

<details>
<summary>Store/summarise previous turns</summary>

→ **memory (ConversationBufferMemory), not agents**
</details>

<details>
<summary>Math / logic / live data the LLM can't do</summary>

→ **agent + tools (ReAct)**
</details>

<details>
<summary>Pick prompt template based on input</summary>

→ **RouterChain**
</details>

<details>
<summary>Agent stuck in a loop</summary>

→ **max\_iterations**
</details>

<details>
<summary>Need controllable custom agent runtime</summary>

→ **LangGraph (not AgentExecutor)**
</details>

<details>
<summary>Enterprise semantic search with SharePoint/Confluence connectors as RAG retriever</summary>

→ **Amazon Kendra**
</details>

<details>
<summary>Prototype agents</summary>

→ **LangChain**
</details>

<details>
<summary>Task too big for one context window / highly parallel / many tools</summary>

→ **multi-agent with a supervisor**
</details>

<details>
<summary>Agents overwrite each other's updates</summary>

→ **shared state management (single source of truth, e.g. DynamoDB) + conflict resolution**
</details>

<details>
<summary>Limit risk of autonomous financial actions</summary>

→ **approval thresholds + human-in-the-loop + rollback**
</details>

<details>
<summary>Stream agent output to the front end</summary>

→ **AG-UI**
</details>

<details>
<summary>Complex task needs different expertise + different tool permissions</summary>

→ **multi-agent (supervisor + collaborators)**
</details>

<details>
<summary>Simple, uniform task</summary>

→ **single agent (MAS adds overhead)**
</details>

<details>
<summary>Collaborator needs context from earlier step</summary>

→ **orchestration explicitly passes state (no implicit shared memory)**
</details>

<details>
<summary>Prevent data leakage between agents</summary>

→ **session-scoped memory, no cross-agent memory, least-privilege tools per agent**
</details>

<details>
<summary>Agents stuck looping / runaway cost</summary>

→ **rate-limit planner invocations**
</details>

<details>
<summary>Human approval before final output</summary>

→ **HITL stage before supervisor aggregation**
</details>

<details>
<summary>Agent calls REST API</summary>

→ **OpenAPI action group**
</details>

<details>
<summary>Custom backend logic</summary>

→ **Lambda action group**
</details>

<details>
<summary>Change workflow without redeploying agents</summary>

→ **versioned orchestration JSON / orchestration Lambda**
</details>

<details>
<summary>AI recommends an action in prod</summary>

→ **human approves first, AI executes**
</details>

<details>
<summary>Who approves rollout?</summary>

→ **developer approves AI's deployment plan**
</details>

<details>
<summary>AI packages modules</summary>

→ **Units of Work → independently deployable units**
</details>

<details>
<summary>Why AI-DLC for monitoring?</summary>

→ **consistency**
</details>

<details>
<summary>Incident needs code change</summary>

→ **back to dev team via AI-DLC**
</details>

<details>
<summary>Agents in different Regions</summary>

→ **spatial decoupling**
</details>

<details>
<summary>Agents offline at different times</summary>

→ **synchronization decoupling**
</details>

<details>
<summary>Agents on different tech stacks</summary>

→ **interface decoupling**
</details>

## 🧱 Amazon Bedrock AgentCore

<details>
<summary>Voice agent / natural conversation</summary>

→ **AgentCore Runtime bi-directional streaming**
</details>

<details>
<summary>Restrict agent actions with policy</summary>

→ **AgentCore Policy (Cedar)**
</details>

<details>
<summary>Agent remembers past interactions</summary>

→ **AgentCore Memory (episodic)**
</details>

<details>
<summary>Tools with fine-grained access control</summary>

→ **AgentCore Gateway**
</details>

<details>
<summary>Run any agent framework in prod with managed scaling/identity/observability</summary>

→ **AgentCore**
</details>

<details>
<summary>Production scale agents</summary>

→ **AgentCore Runtime**
</details>

<details>
<summary>Turn Lambda/APIs into MCP tools with ingress+egress auth</summary>

→ **AgentCore Gateway**
</details>

<details>
<summary>Agent must run generated code safely</summary>

→ **AgentCore Code Interpreter**
</details>

<details>
<summary>Browse websites, session replay</summary>

→ **AgentCore Browser**
</details>

<details>
<summary>Share memory across collaborating agents</summary>

→ **AgentCore Memory**
</details>

<details>
<summary>Long-running agent, hours of work</summary>

→ **AgentCore Runtime (8 h), not Lambda (15 min)**
</details>

<details>
<summary>State/files must not leak between users' sessions</summary>

→ **Runtime microVM session isolation**
</details>

<details>
<summary>Persist context across sessions</summary>

→ **AgentCore Memory (Runtime state dies at 15 min idle / 8 h)**
</details>

<details>
<summary>Expose existing REST/Lambda as tools without code</summary>

→ **Gateway targets (OpenAPI / Lambda / Smithy)**
</details>

<details>
<summary>Hundreds of tools bloating the prompt</summary>

→ **Gateway semantic search**
</details>

<details>
<summary>Agent calls 3P SaaS on user's behalf</summary>

→ **Identity outbound OAuth + token vault**
</details>

<details>
<summary>Users sign in to agent</summary>

→ **Identity inbound auth**
</details>

<details>
<summary>Forwarding user JWT downstream</summary>

→ **confused deputy (bad) → use act-on-behalf tokens**
</details>

<details>
<summary>Remember preferences / summaries</summary>

→ **long-term memory strategies**
</details>

<details>
<summary>Checkpoint conversation</summary>

→ **short-term memory events**
</details>

<details>
<summary>New agent / migrate off Bedrock Agents</summary>

→ **AgentCore (Harness = fastest, no orchestration code; Runtime = bring your own framework)**
</details>

<details>
<summary>Restrict which tools/actions an agent may execute, under conditions</summary>

→ **AgentCore Policy (Cedar), not Guardrails**
</details>

<details>
<summary>Expose existing REST APIs / Lambda functions as agent tools</summary>

→ **AgentCore Gateway (MCP)**
</details>

<details>
<summary>Agent calls third-party OAuth APIs without re-prompting the user</summary>

→ **AgentCore Identity token vault**
</details>

<details>
<summary>Remember user across sessions / share context between agents</summary>

→ **AgentCore Memory long-term**
</details>

<details>
<summary>Agent must navigate websites</summary>

→ **Browser / Nova Act**
</details>

<details>
<summary>Trace reasoning + tool calls into existing observability stack</summary>

→ **AgentCore Observability (OTEL)**
</details>

<details>
<summary>Measure agent quality before and after deploy</summary>

→ **AgentCore Evaluations**
</details>

<details>
<summary>A/B test prompts or tool descriptions</summary>

→ **AgentCore Optimization**
</details>

<details>
<summary>Org-wide catalog of approved MCP servers/agents</summary>

→ **AgentCore Registry**
</details>

<details>
<summary>Each user session isolated to prevent data leakage</summary>

→ **AgentCore Runtime**
</details>

<details>
<summary>Remember preferences across sessions</summary>

→ **Memory long-term (user-preference strategy)**
</details>

<details>
<summary>Remember context within a session</summary>

→ **Memory short-term**
</details>

<details>
<summary>Agent acting on behalf of user with OAuth consent / storing API keys</summary>

→ **Identity token vault**
</details>

<details>
<summary>Turn Lambda / OpenAPI / Smithy into MCP tools with throttling + multitenancy</summary>

→ **Gateway**
</details>

<details>
<summary>Tool with no OAuth (AWS service) behind uniform MCP interface</summary>

→ **Gateway + Identity**
</details>

<details>
<summary>agentcore configure creates</summary>

→ **IAM execution role + ECR repository**
</details>

<details>
<summary>Agent must act with the user's scoped permissions (not one shared access level)</summary>

→ **AgentCore Identity (workload identity + credential providers)**
</details>

<details>
<summary>System only has a web UI, no API</summary>

→ **AgentCore Browser (or Nova Act)**
</details>

<details>
<summary>Agent tool calls must obey business rules / fine-grained access</summary>

→ **AgentCore Policy (Gateway policy engine)**
</details>

<details>
<summary>Score agent quality in production</summary>

→ **AgentCore Evaluations**
</details>

<details>
<summary>Agent tool needs user consent to a 3rd-party API</summary>

→ **Gateway 3LO OAuth**
</details>

<details>
<summary>Multi-tenant auth rules from IdP claims</summary>

→ **AgentCore Identity custom claims**
</details>

<details>
<summary>Avoid re-prompting users for consent / store 3P OAuth tokens & API keys</summary>

→ **AgentCore Identity token vault**
</details>

<details>
<summary>Keep existing Okta/Entra users</summary>

→ **AgentCore Identity (no migration)**
</details>

<details>
<summary>Tool misuse, privilege compromise, untraceable actions</summary>

→ **AgentCore (Identity, Gateway/Policy, Observability)**
</details>

<details>
<summary>Agent has low latency and no errors but users get wrong answers</summary>

→ **AgentCore Evaluation (observability alone can't detect)**
</details>

<details>
<summary>Agents/sessions/traces view</summary>

→ **AgentCore agents view**
</details>

<details>
<summary>X-Ray traces missing for AgentCore</summary>

→ **enable CloudWatch Transaction Search**
</details>

## 🔀 Tool use, Flows & integration patterns

<details>
<summary>Chain prompts + KB + Lambda visually, little code</summary>

→ **Bedrock Flows (vs code-heavy → Step Functions / LangGraph)**
</details>

<details>
<summary>Route based on model output / input value</summary>

→ **Condition node**
</details>

<details>
<summary>Process each item in a list</summary>

→ **Iterator (+ Collector)**
</details>

<details>
<summary>Repeat until done</summary>

→ **Loop node**
</details>

<details>
<summary>Promote to prod / instant rollback without app changes</summary>

→ **publish version + alias, repoint alias**
</details>

<details>
<summary>Edit a deployed version</summary>

→ **can't: versions are immutable → new version from draft**
</details>

<details>
<summary>Test latest changes</summary>

→ **TSTALIASID → working draft**
</details>

<details>
<summary>App calls a flow</summary>

→ **InvokeFlow with flow ID + alias ID**
</details>

<details>
<summary>Flow permissions</summary>

→ **service role**
</details>

<details>
<summary>Model needs real-time/external data or must take an action</summary>

→ **tool use / function calling via Converse**
</details>

<details>
<summary>Model keeps picking the wrong tool / never calls it</summary>

→ **improve toolSpec.description (+ parameter descriptions)**
</details>

<details>
<summary>Make sure a specific tool is called (testing)</summary>

→ **toolChoice**
</details>

<details>
<summary>How do you know the model wants a tool?</summary>

→ **stopReason = tool\_use**
</details>

<details>
<summary>Tool-use conversation is done</summary>

→ **stopReason = end\_turn**
</details>

<details>
<summary>Match tool result to request</summary>

→ **toolUseId**
</details>

<details>
<summary>Tool failed</summary>

→ **toolResult with status: error (app handles it, not the FM)**
</details>

<details>
<summary>Model returned several toolUse blocks</summary>

→ **run all, return all results in one user message**
</details>

<details>
<summary>Who executes the tool?</summary>

→ **your application (Lambda, etc.), never Bedrock/the FM**
</details>

<details>
<summary>Same tool code for any model</summary>

→ **Converse (vs InvokeModel model-specific payloads)**
</details>

<details>
<summary>Transient model/network errors</summary>

→ **Step Functions retry/fallback**
</details>

<details>
<summary>Draft many sections in parallel</summary>

→ **Step Functions parallel branches**
</details>

<details>
<summary>Broadcast / fan-out to many agents or models</summary>

→ **SNS (or EventBridge rules)**
</details>

<details>
<summary>Buffer requests, reliable task processing, absorb LLM rate limits</summary>

→ **SQS**
</details>

<details>
<summary>AMQP / JMS / MQTT / STOMP, migrate ActiveMQ/RabbitMQ, hybrid</summary>

→ **Amazon MQ**
</details>

<details>
<summary>GenAI request > 29 s via API Gateway</summary>

→ **API Gateway → SQS → Lambda → DynamoDB by message ID**
</details>

<details>
<summary>Avoid client polling for async result</summary>

→ **API Gateway WebSocket**
</details>

<details>
<summary>Scheduled bulk generation (reports, image batch)</summary>

→ **Step Functions / Glue batch pipeline**
</details>

<details>
<summary>Peak load queueing</summary>

→ **temporal decoupling**
</details>

<details>
<summary>Link turns of a conversation across services</summary>

→ **correlation ID**
</details>

<details>
<summary>Route by type/metadata</summary>

→ **message attributes**
</details>

<details>
<summary>Transient model/API failures</summary>

→ **exponential backoff + circuit breaker + model fallback**
</details>

<details>
<summary>Independent steps, same input</summary>

→ **Parallel state**
</details>

<details>
<summary>Same steps per item in an array</summary>

→ **Map state (Iterator required, ItemsPath, MaxConcurrency)**
</details>

<details>
<summary>MaxConcurrency 0</summary>

→ **unlimited**
</details>

<details>
<summary>MaxConcurrency 1</summary>

→ **sequential, in order**
</details>

<details>
<summary>Reuse workflows / cut cost</summary>

→ **nested workflows with Express children in Standard parent**
</details>

<details>
<summary>Trace child to parent execution</summary>

→ **AWS\_STEP\_FUNCTIONS\_STARTED\_BY\_EXECUTION\_ID = `$$.Execution.Id`**
</details>

<details>
<summary>Route results to other domains/accounts, decoupled</summary>

→ **putEvents → EventBridge rules**
</details>

<details>
<summary>Wait for an external callback</summary>

→ **.waitForTaskToken → Standard only**
</details>

<details>
<summary>Roll back partial distributed transaction</summary>

→ **saga (Choice or Catch → compensating steps)**
</details>

<details>
<summary>Long-running/auditable/up to 1 yr</summary>

→ **Standard workflow**
</details>

<details>
<summary>High-volume/≤5 min/cheap</summary>

→ **Express workflow**
</details>

<details>
<summary>Wrong workflow type chosen</summary>

→ **can't change → Copy to new**
</details>

## 🛡️ Guardrails & hallucination management

<details>
<summary>Block hallucinated KB answers</summary>

→ **Guardrails contextual grounding**
</details>

<details>
<summary>Detect responses not supported by source docs / RAG answers drifting</summary>

→ **contextual grounding check**
</details>

<details>
<summary>Response off-topic to the user's question</summary>

→ **grounding check relevance score**
</details>

<details>
<summary>Mask SSN / DOB in prompts and responses</summary>

→ **sensitive information filter (custom IDs → regex)**
</details>

<details>
<summary>Block competitor names</summary>

→ **word filter**
</details>

<details>
<summary>No investment advice</summary>

→ **denied topic**
</details>

<details>
<summary>Jailbreak / prompt injection</summary>

→ **content filter – prompt attacks**
</details>

<details>
<summary>Outdated info presented as current</summary>

→ **temporal hallucination → fix with RAG**
</details>

<details>
<summary>Invented service/entity</summary>

→ **conceptual hallucination**
</details>

<details>
<summary>Wrong citation</summary>

→ **attribution hallucination**
</details>

<details>
<summary>Regulated industry audit trail</summary>

→ **CloudTrail + human review**
</details>

<details>
<summary>Ignore previous instructions / persona override</summary>

→ **content filter – prompt attacks (system prompt alone isn't enough)**
</details>

<details>
<summary>Mathematically verify response is correct vs policy rules</summary>

→ **Automated Reasoning checks**
</details>

<details>
<summary>Answer drifts from retrieved/reference docs</summary>

→ **contextual grounding check**
</details>

<details>
<summary>Check content without invoking a model / docs pipeline / non-Bedrock model</summary>

→ **ApplyGuardrail API**
</details>

<details>
<summary>Was this image AI-generated by Titan/Nova Canvas?</summary>

→ **watermark detection**
</details>

<details>
<summary>Block a subject described in plain English</summary>

→ **denied topic**
</details>

<details>
<summary>Redact rather than block</summary>

→ **sensitive info filter action = mask**
</details>

<details>
<summary>Too many false positives</summary>

→ **lower filter strength**
</details>

<details>
<summary>Monitor without blocking</summary>

→ **detect action**
</details>

<details>
<summary>App must use a stable guardrail config</summary>

→ **publish version, pass ID + version**
</details>

<details>
<summary>Goal hijacking / human manipulation of agents</summary>

→ **Bedrock Guardrails**
</details>

<details>
<summary>Legit domain terms blocked (healthcare/gaming/legal)</summary>

→ **lower content filter strength + add denied topics/word filters (not remove filter, not retrain model)**
</details>

<details>
<summary>Is the guardrail tuned right?</summary>

→ **intervention rate**
</details>

<details>
<summary>Input or output intervention?</summary>

→ **GuardrailContentSource**
</details>

<details>
<summary>Which policy fires?</summary>

→ **GuardrailPolicyType**
</details>

<details>
<summary>Behaviour changed after a config change?</summary>

→ **GuardrailVersion**
</details>

<details>
<summary>Sudden spike in blocks</summary>

→ **anomaly detection alarm on InvocationsIntervened**
</details>

<details>
<summary>Test new thresholds without affecting users</summary>

→ **detect mode (action NONE)**
</details>

<details>
<summary>Regression test guardrail before deploy</summary>

→ **curated dataset + ApplyGuardrail, gate on pass rate**
</details>

<details>
<summary>Harmful request with no content-filter signal (e.g., diagnosis, prescription)</summary>

→ **denied topic**
</details>

<details>
<summary>Specific term slips through</summary>

→ **word filter**
</details>

<details>
<summary>RAG blocks valid paraphrased answers</summary>

→ **lower grounding threshold (start 0.7)**
</details>

<details>
<summary>Accurate but off-question answer</summary>

→ **relevance threshold failed**
</details>

<details>
<summary>Users retry/rephrase after blocks</summary>

→ **false positives**
</details>

<details>
<summary>Judge guardrail correctness at scale</summary>

→ **LLM-as-a-judge (+ HITL sample for precision/recall/F1)**
</details>

<details>
<summary>Different strictness per context</summary>

→ **custom adaptive routing (Lambda + multiple guardrail versions), not intelligent prompt routing**
</details>

<details>
<summary>Prompt attack filter scope and strength</summary>

→ **input only; keep HIGH even when relaxing others**
</details>

<details>
<summary>Who changed/removed a guardrail?</summary>

→ **CloudTrail**
</details>

## 🔐 Security, identity & agent security

<details>
<summary>Private connectivity from VPC</summary>

→ **PrivateLink**
</details>

<details>
<summary>Encrypt flow</summary>

→ **customer managed KMS key**
</details>

<details>
<summary>Scan Lambda/ECR/EC2 for CVEs</summary>

→ **Amazon Inspector**
</details>

<details>
<summary>Forensic investigation</summary>

→ **Amazon Detective**
</details>

<details>
<summary>Suspicious API/credential use</summary>

→ **GuardDuty**
</details>

<details>
<summary>Find PII in S3 before fine-tuning</summary>

→ **Amazon Macie**
</details>

<details>
<summary>Reach Bedrock without internet</summary>

→ **PrivateLink**
</details>

<details>
<summary>Inspect egress TLS</summary>

→ **Network Firewall**
</details>

<details>
<summary>Block bots/scrapers</summary>

→ **WAF Bot Control**
</details>

<details>
<summary>Persona-based ML roles</summary>

→ **SageMaker Role Manager**
</details>

<details>
<summary>Model given too much autonomy</summary>

→ **OWASP excessive agency**
</details>

<details>
<summary>Unsanitised model output used downstream</summary>

→ **insecure output handling**
</details>

<details>
<summary>Malicious data in training set</summary>

→ **training data poisoning**
</details>

<details>
<summary>Adversary tactics against AI</summary>

→ **MITRE ATLAS**
</details>

<details>
<summary>Single pane for findings + prioritise</summary>

→ **Security Hub**
</details>

<details>
<summary>Centralise security logs for analytics</summary>

→ **Security Lake**
</details>

<details>
<summary>Free DDoS L3/L4</summary>

→ **Shield Standard**
</details>

<details>
<summary>VPC-wide traffic filtering</summary>

→ **Network Firewall**
</details>

<details>
<summary>Ransomware-proof backups</summary>

→ **AWS Backup immutable / air-gapped vault**
</details>

<details>
<summary>Backup compliance reports</summary>

→ **Backup Audit Manager**
</details>

<details>
<summary>Multi-account landing zone with guardrails</summary>

→ **Control Tower**
</details>

<details>
<summary>Restrict max permissions across accounts</summary>

→ **SCPs**
</details>

<details>
<summary>Customer-facing sign-in, social IdPs</summary>

→ **Cognito**
</details>

<details>
<summary>Workforce SSO</summary>

→ **IAM Identity Center**
</details>

<details>
<summary>Top LLM app vulnerabilities</summary>

→ **OWASP Top 10 for LLMs**
</details>

<details>
<summary>Indirect prompt injection</summary>

→ **malicious instructions in retrieved/ingested data, not the user prompt**
</details>

<details>
<summary>Model inversion / data leakage</summary>

→ **monitor inference + strict endpoint access**
</details>

<details>
<summary>Verify legitimate AI agents/crawlers hitting CloudFront</summary>

→ **WAF Web Bot Auth (vs bad bots → WAF Bot Control)**
</details>

<details>
<summary>Coding assistant writes least-privilege IAM</summary>

→ **IAM Policy Autopilot (MCP server)**
</details>

<details>
<summary>AWS workload calls external API without stored keys</summary>

→ **STS outbound federation (JWT)**
</details>

<details>
<summary>Developer CLI without access keys</summary>

→ **aws login**
</details>

<details>
<summary>Security review of design docs before code / on-demand pen test</summary>

→ **AWS Security Agent**
</details>

<details>
<summary>Speed up incident investigation</summary>

→ **Security Incident Response AI agent**
</details>

<details>
<summary>Scan existing S3 objects for malware</summary>

→ **GuardDuty Malware Protection for S3 on-demand**
</details>

<details>
<summary>Clean backup before restore</summary>

→ **GuardDuty Malware Protection for AWS Backup**
</details>

<details>
<summary>Rotate 3rd-party SaaS credentials without Lambda</summary>

→ **Secrets Manager managed external secrets**
</details>

<details>
<summary>Block unencrypted VPC traffic</summary>

→ **VPC encryption controls (enforce mode)**
</details>

<details>
<summary>Egress domain filtering via explicit proxy</summary>

→ **Network Firewall Proxy**
</details>

<details>
<summary>DNS exfiltration</summary>

→ **Route 53 Global Resolver / DNS Firewall**
</details>

<details>
<summary>Quantum-safe certificates</summary>

→ **Private CA ML-DSA**
</details>

<details>
<summary>Who is this user / may they invoke the agent?</summary>

→ **inbound auth (IAM SigV4 or OAuth/JWT from Cognito/Okta/Entra)**
</details>

<details>
<summary>Agent calls tools/APIs</summary>

→ **outbound auth**
</details>

<details>
<summary>Agent calls Slack/Google on behalf of the user with consent</summary>

→ **3LO OAuth**
</details>

<details>
<summary>Agent calls an internal API as itself</summary>

→ **2LO / client credentials (M2M)**
</details>

<details>
<summary>User calls AWS service directly, no delegation</summary>

→ **SigV4**
</details>

<details>
<summary>Machine-to-machine auth for customer apps</summary>

→ **Cognito M2M**
</details>

<details>
<summary>Agent given broad perms just in case</summary>

→ **wrong → least privilege (OWASP: privilege compromise / excessive agency)**
</details>

<details>
<summary>OWASP Agentic threats vs OWASP Top 10 for LLMs: which list?</summary>

→ **They are different lists (Agentic = 15 threats); read which list the question means**
</details>

<details>
<summary>Suspicious Bedrock API use from new location</summary>

→ **GuardDuty**
</details>

<details>
<summary>Block long/malicious requests before app</summary>

→ **WAF**
</details>

<details>
<summary>PII in S3</summary>

→ **Macie**
</details>

<details>
<summary>PII in logs</summary>

→ **CloudWatch Logs data protection policy**
</details>

<details>
<summary>PII in text</summary>

→ **Comprehend**
</details>

<details>
<summary>Does Bedrock abuse detection involve human access to prompts/outputs?</summary>

→ **No: automated, no human access to your prompts/outputs**
</details>

## ⚖️ Governance, compliance & responsible AI

<details>
<summary>Ambiguous text-to-image prompt yields biased images</summary>

→ **TIED**
</details>

<details>
<summary>Equal error rates across groups</summary>

→ **equalized odds**
</details>

<details>
<summary>Expand dataset by swapping attributes</summary>

→ **counterfactual data augmentation**
</details>

<details>
<summary>Download SOC/ISO/PCI reports for auditor</summary>

→ **AWS Artifact**
</details>

<details>
<summary>Automate audit evidence collection</summary>

→ **AWS Audit Manager**
</details>

<details>
<summary>Who called the Bedrock API / who changed X</summary>

→ **CloudTrail**
</details>

<details>
<summary>How was this resource configured last month</summary>

→ **AWS Config**
</details>

<details>
<summary>Document intended use + risk rating of model</summary>

→ **SageMaker Model Cards**
</details>

<details>
<summary>Customer controls fine-tuning data</summary>

→ **Scope 4**
</details>

<details>
<summary>Customer controls training data</summary>

→ **Scope 5**
</details>

<details>
<summary>Claude via Bedrock API</summary>

→ **Scope 3**
</details>

<details>
<summary>Data quality checks in pipeline</summary>

→ **Glue Data Quality / DataBrew**
</details>

<details>
<summary>Using ChatGPT/PartyRock</summary>

→ **Scope 1**
</details>

<details>
<summary>Amazon Q / SaaS with GenAI</summary>

→ **Scope 2**
</details>

<details>
<summary>SageMaker JumpStart fine-tune</summary>

→ **Scope 4**
</details>

<details>
<summary>Non-negotiable AI law</summary>

→ **EU AI Act (regulation)**
</details>

<details>
<summary>AI management system standard</summary>

→ **ISO 42001**
</details>

<details>
<summary>GenAI risk profile</summary>

→ **NIST AI 600-1**
</details>

<details>
<summary>Public chatbot / PartyRock used by staff</summary>

→ **Scope 1 → web proxy/CASB + DLP + AUP**
</details>

<details>
<summary>SaaS with GenAI under enterprise contract / Amazon Q Business</summary>

→ **Scope 2 → opt out of training, EULA review, SOC 2/ISO 27001**
</details>

<details>
<summary>Our app calls a Bedrock FM + RAG</summary>

→ **Scope 3**
</details>

<details>
<summary>Fine-tune FM with our data</summary>

→ **Scope 4**
</details>

<details>
<summary>Train from scratch</summary>

→ **Scope 5**
</details>

<details>
<summary>Classification of a fine-tuned model?</summary>

→ **same as (highest) fine-tuning data**
</details>

<details>
<summary>Remove a customer's PII from a trained model</summary>

→ **impossible without full retrain → anonymise before tuning**
</details>

<details>
<summary>Bias assessment in SageMaker</summary>

→ **SageMaker Clarify**
</details>

<details>
<summary>Resume long training</summary>

→ **SageMaker checkpoints**
</details>

<details>
<summary>Privacy-preserving training</summary>

→ **differential privacy**
</details>

<details>
<summary>Document model provenance/intended use</summary>

→ **model cards**
</details>

<details>
<summary>Multi-account security baseline incl. Bedrock</summary>

→ **AWS SRA (Security Reference Architecture)**
</details>

<details>
<summary>Detect bias / explain prediction</summary>

→ **SageMaker Clarify**
</details>

<details>
<summary>Audit-ready GenAI evidence</summary>

→ **Audit Manager GenAI Best Practices Framework**
</details>

## ⚡ Cost, latency & monitoring

<details>
<summary>Throttling</summary>

→ **check service quotas**
</details>

<details>
<summary>Throttling / regional capacity / higher throughput</summary>

→ **Cross-Region inference profile**
</details>

<details>
<summary>Same questions asked repeatedly, cut latency/cost</summary>

→ **semantic cache**
</details>

<details>
<summary>Debug complex/nested flows</summary>

→ **X-Ray**
</details>

<details>
<summary>Chat app feels slow before text appears</summary>

→ **TTFT → streaming + latency-optimized inference**
</details>

<details>
<summary>Batch / document generation SLA</summary>

→ **end-to-end latency**
</details>

<details>
<summary>Faster responses, same model & quality, no code rewrite</summary>

→ **performanceConfig latency: optimized (via inference profile)**
</details>

<details>
<summary>Latency-optimized quota exceeded</summary>

→ **falls back to standard, standard price (no error)**
</details>

<details>
<summary>Bedrock latencyMs lower than user sees</summary>

→ **it excludes network/client time**
</details>

<details>
<summary>Same long system prompt / document on every request</summary>

→ **prompt caching with cachePoint after static prefix**
</details>

<details>
<summary>Cache never hits</summary>

→ **prefix under token minimum, not an exact match, wrong checkpoint placement, or >5 min gap**
</details>

<details>
<summary>Confirm cache hit</summary>

→ **cacheReadInputTokens > 0, cacheWriteInputTokens = 0**
</details>

<details>
<summary>Mix of simple and complex prompts, cut cost without hand-coded routing</summary>

→ **intelligent prompt routing (same family, router ARN as modelId)**
</details>

<details>
<summary>Which model handled the request?</summary>

→ **router trace metadata**
</details>

<details>
<summary>Throttled during load tests</summary>

→ **Provisioned Throughput / quota increase**
</details>

<details>
<summary>Reduce latency + cost of long responses</summary>

→ **lower maxTokens**
</details>

<details>
<summary>See the actual prompts/responses</summary>

→ **Bedrock model invocation logging (CloudWatch Logs / S3), not CloudTrail**
</details>

<details>
<summary>Out-of-the-box token/latency/cost dashboard</summary>

→ **CloudWatch GenAI observability (Model Invocations)**
</details>

<details>
<summary>Vendor-neutral instrumentation</summary>

→ **ADOT / OTEL**
</details>

<details>
<summary>Alert on customization/batch job completion</summary>

→ **EventBridge**
</details>

<details>
<summary>Latency SLO</summary>

→ **p95/p99 percentile alarm**
</details>

<details>
<summary>Fewer false alarms</summary>

→ **composite alarm**
</details>

<details>
<summary>Bedrock cost per team/app</summary>

→ **application inference profiles + cost allocation tags → Cost Explorer / Budgets**
</details>

<details>
<summary>Throttling / traffic bursts</summary>

→ **service quotas + backoff + cross-Region inference**
</details>

## 🧪 Evaluation, testing & troubleshooting

<details>
<summary>Summarisation quality</summary>

→ **ROUGE**
</details>

<details>
<summary>Translation quality</summary>

→ **BLEU**
</details>

<details>
<summary>Semantic / paraphrase similarity</summary>

→ **BERTScore**
</details>

<details>
<summary>Next-token fit</summary>

→ **perplexity**
</details>

<details>
<summary>Gold standard but costly</summary>

→ **human evaluation**
</details>

<details>
<summary>Repeatable model comparison in CI/CD</summary>

→ **CreateEvaluationJob**
</details>

<details>
<summary>Answer contradicts retrieved docs</summary>

→ **low faithfulness**
</details>

<details>
<summary>Answer off-topic/incomplete</summary>

→ **low answer relevancy**
</details>

<details>
<summary>Retrieval missing needed facts</summary>

→ **low context recall**
</details>

<details>
<summary>Relevant chunks ranked low</summary>

→ **low context precision**
</details>

<details>
<summary>Traces not showing up</summary>

→ **enable observability in config, CloudWatch Transaction Search, check IAM role permissions**
</details>

<details>
<summary>Which agent failed?</summary>

→ **agent-level traces/logs (Bedrock console trace, CloudWatch)**
</details>

<details>
<summary>Input data distribution changes over time</summary>

→ **data drift**
</details>

<details>
<summary>Drift monitoring</summary>

→ **SageMaker Model Monitor**
</details>

<details>
<summary>Compare two models on subjective quality at scale</summary>

→ **LLM-as-a-judge (pairwise)**
</details>

<details>
<summary>Where does Nova LLM-as-a-Judge run?</summary>

→ **SageMaker AI training job + recipe**
</details>

<details>
<summary>winrate CI includes 0.5</summary>

→ **no significant winner, add data**
</details>

<details>
<summary>High stderr</summary>

→ **more evaluation examples**
</details>

<details>
<summary>What does winrate refer to?</summary>

→ **Model B preferred**
</details>

<details>
<summary>Catch quality regressions over time</summary>

→ **scheduled/continuous judge evaluation**
</details>

<details>
<summary>Continuously score live production quality</summary>

→ **online evaluation config**
</details>

<details>
<summary>Test prompt/model change before deploy / block PR</summary>

→ **on-demand Evaluate API**
</details>

<details>
<summary>Right answer, wrong tool</summary>

→ **Tool Selection Accuracy**
</details>

<details>
<summary>Wrong arguments to tool</summary>

→ **Tool Parameter Accuracy**
</details>

<details>
<summary>Answer not supported by retrieved docs</summary>

→ **Faithfulness**
</details>

<details>
<summary>Did user achieve goal across conversation</summary>

→ **Goal Success Rate (SESSION level)**
</details>

<details>
<summary>Organisation-specific policy / brand tone / compliance</summary>

→ **custom evaluator (instructions + rating scale + model config, CreateEvaluator)**
</details>

<details>
<summary>Consistent, auditable judge scores</summary>

→ **reason-first LLMaJ, low temperature**
</details>

<details>
<summary>Why did score drop?</summary>

→ **CloudWatch Logs reasoning**
</details>

<details>
<summary>Trend / alarm on scores</summary>

→ **CloudWatch Metrics (EMF)**
</details>

<details>
<summary>Sessions skipped or mis-scored</summary>

→ **missing session ID / tool call spans in OTEL/OpenInference traces**
</details>

<details>
<summary>Evaluation scoring mid-conversation</summary>

→ **increase session idle timeout (default 15, 1–1440 min)**
</details>

<details>
<summary>Multi-agent workflow scored as one session</summary>

→ **multiple log groups (≤ 5) + shared session ID**
</details>

<details>
<summary>Reduce evaluation cost</summary>

→ **lower sampling %, fewer evaluators, conditional/hierarchical sampling (not 100% sampling)**
</details>

<details>
<summary>Evaluator scores cluster at 0.5</summary>

→ **rewrite unclear instructions**
</details>

<details>
<summary>No results right after enabling</summary>

→ **normal: 30–45 min (and config is disabled by default unless the start box is checked)**
</details>

<details>
<summary>What permissions does the evaluation IAM role need?</summary>

→ **read source logs + write evaluation log group + invoke Bedrock model**
</details>

<details>
<summary>Production inputs drift from training</summary>

→ **Model Monitor data quality**
</details>

<details>
<summary>Accuracy degrading</summary>

→ **Model Monitor model quality (needs ground truth)**
</details>

<details>
<summary>Feature importance shift</summary>

→ **Model Monitor feature attribution (SHAP)**
</details>

<details>
<summary>Rebalance dataset</summary>

→ **Data Wrangler (SMOTE, over/undersampling)**
</details>

<details>
<summary>SageMaker endpoint fails at creation</summary>

→ **configuration (artifact/container/IAM); check CloudWatch logs**
</details>

<details>
<summary>SageMaker endpoint fails at runtime</summary>

→ **resources (memory); check CloudWatch logs**
</details>

---
[← Back to index](README.md)
