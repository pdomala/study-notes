[← Back to index](README.md)

# 🕸️ 07 · Agents, multi-agent systems & frameworks

> Exam domain: D2

## Key points
- Agent types: **reactive** → **deliberate** (plan first) → **goal-driven** (adapts). Coordination: **handoff** (sequential) · **parallel** · **hierarchical** (supervisor). Sequential agents building on each other → **pipeline**.
- Go multi-agent when the task is **highly parallel**, **exceeds one context window**, or needs **many complex tools**; small/uniform task → **single agent**.
- Bedrock multi-agent: **supervisor** plans → delegates → aggregates. Collaborators **don't share memory** (session-scoped), so orchestration **passes state explicitly**. Agents overwriting each other → **shared state, single source of truth**.
- Runaway loops → **rate-limit planner calls** / max iterations · transient errors → **Step Functions retry/fallback** · contain mistakes with **confidence threshold → human approval**, $ thresholds, rollback; increase autonomy gradually.
- Bedrock Agents: change config at runtime → **Inline Agents** · beyond ReAct → **Custom Orchestration** · action groups = Lambda / OpenAPI schema / return of control. **Agents Classic closed to new customers (30 Jul 2026)** → build on **AgentCore**.
- Protocols: **MCP** = agent ↔ tool · **A2A** = agent ↔ agent · **AG-UI** = stream agent output to UI. MCP hosting: stateless/light → **Lambda**, complex → **ECS**.
- Frameworks: **Strands Agents** (model-driven, AWS-native) · **AWS Agent Squad** (multi-agent routing) · LangChain = prototype, **LangGraph** replaces legacy AgentExecutor. LangChain: loader = ingest · retriever = fetch · memory = prior turns · agent = tools (ReAct) · RouterChain = pick prompt.
- **AI-DLC ops**: AI recommends/plans → **human approves** → AI executes (never auto-restart/rollout); developer approves the deployment plan.

## Notes

### Agents vs agentic AI — definitions, loop & traits
- **AI agent** = single program, specific task, limited autonomy (chatbot, recommender) · **agentic AI** = **autonomous, goal-directed**; reasons, plans, calls tools/APIs, coordinates agents/systems and iterates to the goal with minimal human input, without step-by-step prompting; makes contextual decisions & adapts
- **AWS definition of AI agents** → autonomous software that uses AI to **reason, plan and complete tasks**, **evaluating results and adjusting** toward goals (iterative)
- **Agent loop**: goals/instructions + tools (with descriptions) + context → agent → LLM prompt → **actions on environment → observation → repeat**
- **Chatbot = reactive** (answers queries) · **agent = proactive, goal-oriented** — plans multi-step, uses tools, **adapts to intermediate results**
- **5 traits**: autonomy · goal-oriented · multi-step reasoning · tool integration · adaptive behaviour
- **4 characteristics of agentic AI**:
  - **Proactive** — anticipates (e.g. reroute shipments before delays)
  - **Adaptable** — domain/context aware (e.g. healthcare terminology + regs)
  - **Collaborative** — with humans + other agents
  - **Specialized** — many **hyperspecialised** agents (e.g. compliance / fraud / portfolio)
- GenAI flow: Analyze → Summarize (advises; human input every step) · agentic flow: **Analyze → Recommend → Plan → Execute → Monitor → Adapt** (acts; human sets **goals and boundaries**)
- **Agent roles** when breaking complex processes into orchestrated steps: **task coordination** (order + dependencies) · **reporting & logging** · **scalability & concurrency** · **integration & communication** (APIs, queues)
- Related prompt techniques: **ReAct** (reason + act) · **ART** (Automatic Reasoning & Tool-use)
- **Production readiness**: reliability, scalability, security, governance → **handle edge cases, consistent performance under load, audit trails**
- **Context persistence** = remembering past interactions to personalise → AgentCore Memory (see [08 · AgentCore](08-bedrock-agentcore.md))

### Autonomy, maturity & right-sizing (deterministic vs agentic)
- **Autonomy spectrum**: L1 responsive → L2 suggestive (human-directed) → L3 proactive (monitors) → L4 autonomous (learns/adapts)
- **4 AWS enablers**: reasoning models · data/knowledge integration · infra + protocols (**MCP, A2A**) · dev & orchestration tools
- **Maturity**: rule-based RPA → GenAI assistants → goal-driven agents → fully autonomous; most enterprises sit between RPA and assistants; avoid "**agent washing**"
- → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (Deterministic workflows vs agents: ~95/5 right-sizing, 60% savings, 4-question agent test)
- **Adding AI to static workflows**: decision gates at branches with **existing logic kept as fallback**, **manual override**, log every AI decision, clear rollback
- **Building agent ecosystems**: Phase 1 single agent (MCP/OpenAPI, self-description) → Phase 2 orchestrator chaining agents (decomposition, retries, context passing) → Phase 3 **dynamic agent selection + parallel execution**

### Agent types & multi-agent orchestration patterns
- **Agent types** (simplest → most complex):
  - **Reactive** — if-this-then-that, no reasoning (e.g. on AgentCore Runtime using S3/DynamoDB as tools via Gateway)
  - **Deliberate** — plan/reason before acting (Strands on AgentCore Runtime + reasoning model calling Bedrock FMs)
  - **Goal-driven** — high-level objective across tasks, adapts strategy (AgentCore orchestrating specialists, keeps context with Memory)
- **Coordination patterns**: **Handoff** = sequential relay A → B → C · **Parallel** = simultaneous tasks · **Hierarchical** = goal-driven supervisor over specialists
- **Multi-agent orchestration patterns**:
  - **Hierarchical** → master/supervisor agent coordinates subordinates
  - **Peer-to-peer** → agents collaborate as equals, negotiate tasks
  - **Pipeline** → sequential, each agent adds value (e.g. research → write → fact-check → optimise)
  - **Marketplace** → dynamic agent selection by capability, availability, performance
- Multi-agent flow types (4): **serialization** (sequential) · **routing** · **orchestration** · **parallelization**
- **3-stage pattern**: **Input (analyse) → Core (create) → Output (validate/format)** → fault isolation, per-stage scaling, A/B test models/prompts

### Single agent vs multi-agent system (MAS) — when & why
- **MAS** = specialised agents with **narrow scope + scoped tools**, coordinated by an **orchestration layer**; mirrors human teams (planner · researcher · writer · reviewer)
- A single agent handles everything (interpret, reason, call tools, respond) → fine for simple flows, **inefficient as complexity grows** or when specialised reasoning is needed
- ✅ **Go multi-agent** when: heavy **parallelisation** · info **exceeds one context window** · **many complex tools** · specialised reasoning (research, summarise, creative) · **different tools/permissions per role** · subtasks that depend on each other's outputs · must be **maintainable/auditable**
- ❌ **Single agent** for **small or highly uniform tasks** (MAS adds overhead)
- **Orchestration layer**: execution order + context transfer · manages agent state across task boundaries · **restricts tool access per agent scope** → **security** (scoped tool permissions) + **traceability** (logs/events in Bedrock console)
- **Benefits**: **specialization** (accuracy, modular reasoning) · **reusability** (one summarizer agent across research + support flows) · **scalability** (add a review/compliance agent without touching core logic) · **debuggability** (trace logs per agent) · **tool-scoped reasoning** (only the retriever agent gets DB access; LLM-only roles isolated) · **explainability** (visible intermediate steps)

### Amazon Bedrock Agents (Classic) — anatomy, action groups & capabilities
- ⚠️ Bedrock Agents is now **Bedrock Agents Classic** — **closed to new customers since 30 Jul 2026**; new builds → **AgentCore** (Harness = migration target; see [08 · AgentCore](08-bedrock-agentcore.md)). Supervisor/collaborator concepts are still examinable
- **Agent anatomy**:
  - **Instructions** → purpose, scope, constraints, tone, tool-use behaviour
  - **Foundation model** → any Bedrock FM, assigned **per agent**
  - **Action groups** → callable operations via **Lambda** or **OpenAPI schema**
  - **Knowledge base** (optional) → RAG grounding on enterprise data
  - **Session memory** → persists **per session**, **not shared across agents** unless orchestration passes it
  - Each component testable, securable, maintainable independently
- **Action group / tool types**: **Lambda** → backend logic, workflows, data transforms · **OpenAPI** → internal/3rd-party REST (endpoints, inputs, auth in the schema; API Gateway needs signed requests + error handling) · **Knowledge Base** → RAG grounding
- Agent picks an action → builds a parameterised request → Bedrock invokes it securely; monitor via orchestration logs + **CloudWatch**
- Best practice: **one functional domain per action group** · **least-privilege IAM** per tool · one agent's tools **not available to others** unless shared via orchestration
- **Agent capabilities**:
  - **Inline Agents** → configure at runtime, no pre-deployment
  - **Custom Orchestration** → go beyond the default **ReAct** loop
  - **Code Interpretation** → generate + execute code for data analysis
  - **Multi-agent Collaboration (GA)** → parallel processing, **supervisor** pattern
- Invoke programmatically: `bedrock-agent-runtime` → `invoke_agent` (`agentId`, `agentAliasId`, `sessionId`, `inputText`)

### Bedrock multi-agent collaboration — supervisor & collaborators
- Bedrock model = **supervisor (planner) + collaborators**: supervisor **decomposes the task**, **plans → delegates → aggregates**; collaborators do retrieval / summarise / write / validate
- **Supervisor–collaborator workflow**:

1. User request
2. **Planning** — supervisor splits the goal, picks collaborators
3. **Delegation** — orchestration passes input + context
4. **Execution** — collaborators use their own FM + action groups
5. **Aggregation** — supervisor merges → final answer

- Example pipeline: Research → Summarizer → Writer, supervisor assembles
- Agents **don't talk directly** and **don't share memory** — the orchestration layer **explicitly passes state**
- **Collaboration modes**: **Supervisor** (plans, calls collaborators, synthesises) vs **Supervisor with routing** (simple requests routed straight to one collaborator; falls back to supervisor mode when complex)
- Collaborators are attached by **agent alias**

> Note: check the current Bedrock documentation for collaboration mode details.

### Agent memory, state & conversation context
- Agent memory is **session-level only** · **no implicit sharing** between agents
- **Explicit context passing** (summaries, variables, JSON) supervisor → collaborators
- **Stateless resets**: a new invocation resets memory by default (security, repeatability)
- Treat agent memory as a **bounded workspace** → design what context is *passed*, not what *persists*; short-lived session memory avoids contamination between runs
- → See [03 · Prompt engineering](03-prompt-engineering-prompt-management.md) (prompt history store, Session Management APIs)
- → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (Anatomy of a GenAI application: AI assistant types)
- Cross-session / shared agent memory → **AgentCore Memory** ([08 · AgentCore](08-bedrock-agentcore.md))

### Use case — multi-agent grant-writing assistant
- **Analyzer** (parse RFP → JSON: funder, deadline, eligibility, format; LLM only) → **Outliner** (sections + word limits) → **Narrative Writer** (KB of impact reports → avoids hallucination) → **Compliance** (tone/keywords/readability, optional validation Lambda) → **Planner/Supervisor** aggregates
- Lambda preprocessor extracts text from PDF/DOCX → S3
- Production hardening: **per-agent IAM roles** · **Step Functions** for parallel section drafting · **HITL review** between Compliance and Planner · CloudWatch latency/error dashboards · KB from approved, current sources

### Build, test & operate multi-agent systems
- Console: Bedrock → Agents → Create Agent (name, FM, instruction, optional KB, session memory); **test each agent alone before chaining**
- Planner agent settings: multi-agent chaining · scoped memory · trace-level logging
- Planner logic can live in a **versioned JSON config** or an **orchestration Lambda** → change flows **without redeploying agents**; **version-control orchestration JSON** for rollback
- Test via console **Test agent** (expand trace per step) or SDK `bedrock-agent-runtime` → `invoke_agent`
- Start with a **2-agent prototype**, then add tools, memory, conditions · minimal chains first · explicit system prompts · **log intermediate states**
- **Governance & operations**:
  - Disable cross-agent memory → prevents **data leakage**
  - Structured logs → CloudWatch Logs or S3 via **Firehose**
  - **Rate-limit planner calls** (quotas / Lambda throttling) → stops **runaway agent loops** & cost blowouts
  - **Retry & fallback** → Step Functions retry policies
  - Sandbox tool Lambdas in a **VPC with restricted egress**
  - Optional DynamoDB **agent observability table** (agent, model version, invocation ID, latency, error)
  - **EventBridge** to auto-trigger a compliance agent after generation events
  - Compare models for an agent → **Bedrock Model Evaluation**
- Iteration cycle: design → test → evaluate → refine → deploy (versioned)

### Reliability, safeguards & human-in-the-loop
- Agent hallucination mitigation: **human oversight** (approve critical actions) · **validation checkpoints** (compare vs trusted data) · **confidence thresholds**
- AWS tooling (detail in [08 · AgentCore](08-bedrock-agentcore.md) and [10 · Guardrails](10-guardrails-hallucination.md)): **Bedrock Guardrails** (works for Bedrock, fine-tuned and **externally hosted** models) · AgentCore **Observability** (OpenTelemetry) · AgentCore **Evaluations** (pre-deploy + continuous)
- **Coordination failures** (duplicate actions, overwrites, conflicting records) → **consensus protocols** · **conflict resolution** (priority rules) · **state management** (single source of truth, e.g. DynamoDB)
- **Contain costly mistakes**: **confidence scoring** (<90% → human approval) · **approval thresholds** (auto-refund ≤ $100, above needs approval) · **rollback** (revert to last verified value)
- **Deploy reliably**: start with simple reactive agents → approval workflows for high-impact actions → continuous monitoring → **gradual autonomy** based on demonstrated reliability
- PoC → production adds: performance at scale, expanded security, multi-user context management, governance
- **Human-in-the-loop** → approvals for critical actions, escalation on uncertainty, feedback loops
- Agents generate **10–100× more requests** than humans → protect services with **rate limiting + circuit breakers**
- Additional in-scope items from the exam guide (Task 2.1):
  - Step Functions implementing **ReAct / chain-of-thought** and agent design patterns (2.1.2, 2.5.5)
  - Safeguards: Step Functions **stopping conditions**, Lambda **timeouts**, **IAM resource boundaries**, **circuit breakers** (2.1.3)
  - Model coordination: specialised FMs, **ensemble aggregation**, model selection frameworks (2.1.4)
  - Human-in-the-loop: **Step Functions approval workflows** + **API Gateway feedback collection** (2.1.5)
  - Tool reliability: Strands API custom behaviours, Lambda **parameter validation + error handling** (2.1.6)

### Agent-ready tools & protocols (MCP · A2A · AG-UI)
- Make tools self-describing: **API-first** (OpenAPI specs) · **JSON schemas** with validation rules · function-calling standards / **MCP** · **discovery** (tool registries, semantic tags) · **structured errors** with suggested corrections + fallbacks
- **Protocols**: **MCP** = agent ↔ tools · **A2A** = agent ↔ agent · **AG-UI** = stream agent output to a UI → AgentCore supports all three natively
- **MCP** = open standard for **agent↔tool**: **standardised tool interface**, dynamic **tool discovery**, **cross-framework interoperability**; tools exposed as **MCP servers**
- **A2A** = open protocol (Google → **Linux Foundation**) for **agent-to-agent** communication across frameworks/platforms
- **Tool integration patterns**: **Lambda** → serverless custom tools, auto-scale, pay-per-use · **API Management** → central gateway for external APIs (rate limiting, auth, monitoring) · **MCP** · **A2A**
- **MCP hosting**: **Lambda = stateless lightweight MCP server** · **ECS = complex MCP server** · MCP client libraries (exam guide 2.1.7)
- Converting APIs/Lambda into MCP tools behind one endpoint → AgentCore Gateway (see [08 · AgentCore](08-bedrock-agentcore.md))

### Strands Agents, the AWS agentic stack & build paths
- **Strands Agents SDK** = AWS **open-source** agent framework; **model-driven / model-first** — the LLM is the orchestrator and decides which tools to use and when (FM drives the agent loop; not rule/template-based)
  - **Python + TypeScript** · **20+ built-in tools** · session management · few lines of code: natural-language instructions + tools (Python functions, AWS services, **MCP tools**)
  - Native **MCP**, integrates **Bedrock / Lambda / Step Functions**, **any provider** (Bedrock, OpenAI…), multimodal, custom tools → **deploys onto AgentCore** (runtime, memory, observability); best for fully autonomous agents on AWS
- **Build/connect** = Strands + MCP · **operate at scale** = **AgentCore** (scaling, identity, MCP tool access, memory/sessions, observability, guardrails; any framework — LangGraph, CrewAI, Strands — any model; Runtime, Memory, Gateway, Identity, Policy, Observability, Evaluations etc. → see [08 · AgentCore](08-bedrock-agentcore.md))
- **AWS Agent Squad** = multi-agent orchestration/routing framework (exam guide 2.1.1, 2.5.5)
- → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (AWS GenAI stack: agentic stack layers, compute options, customer path selection) · Nova Act → [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Nova family)
- Bedrock orchestration options: **AgentCore** (code) / **Flows** (visual)
- **Supporting AWS services**: **Lambda** (serverless agents) · **Step Functions** (workflow + state) · **EventBridge** (event triggers) · **DynamoDB** (agent state/sessions) · Aurora Serverless · **Neptune** (graph reasoning) · **S3 Vectors** (cheap RAG) · **API Gateway** (agent ↔ system) · **AppSync** (real-time multi-agent sync) · **SQS/SNS** (agent messaging)

### LangChain — overview, models, prompt templates & indexes
- Why LangChain (LLMs stateless, out-of-domain data, chaining) → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (FM & LLM fundamentals)
- **LangChain** = open-source framework (**Python, TypeScript, JavaScript**) → **orchestration layer** between prompt and Bedrock FM; manages **context** and **sequencing of steps** across dev → productionisation → deployment; adds **conversation chains**
- Components: **models, prompt templates, indexes, memory, chains, agents**
- Use cases: RAG QA chatbots, summarisation, code gen, info extraction, query analysis, API interaction; start with **LCEL**
- AWS integrations: chat models, LLMs, embeddings, document loaders, vector stores, retrievers, tools, memory, graphs, callbacks, chains
- → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Calling Bedrock models from LangChain: packages, BedrockLLM, ChatBedrock, BedrockEmbeddings) · [04 · Model customisation](04-model-customisation.md) (custom model ARN) · [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (embedding distance; LangChain loaders, retrievers, Kendra, vector stores) · [03 · Prompt engineering](03-prompt-engineering-prompt-management.md) (PromptTemplate) · [08 · AgentCore](08-bedrock-agentcore.md) (production package `amazon-bedrock-agentcore`)

### LangChain — memory, chains & agents (ReAct)
- Memory (ConversationBufferMemory) & chains (LCEL, LLMChain, RouterChain, chunk + chain) → See [03 · Prompt engineering](03-prompt-engineering-prompt-management.md) (LangChain prompt templates, chains, memory)
- **Agents**: LLMs weak at **rule-based math, logic, reasoning** → agents call external tools (search, calculator, APIs, DBs) and **run code**
  - Agent = **reasoning engine** deciding which actions + order = **LLM + tools + stopping condition**; **toolkits = group of 3–5 tools**
  - **ReAct** (reason + act) loop: decide action → run → observe → until stop
  - Built-in tools e.g. **LLMMathChain**; custom tools via `Tool.from_function(func, name, description)`
  - `create_react_agent` + `AgentExecutor(max_iterations=10, handle_parsing_errors=True)` → `max_iterations` stops loops
  - ⚠️ **AgentExecutor = legacy**, not flexible → **transition to LangGraph** (highly controllable agent runtime)
- → See [08 · AgentCore](08-bedrock-agentcore.md) (LangChain vs AgentCore: prototype → LangChain, production → AgentCore Runtime)

### AI-DLC (AI-Driven Development Lifecycle) in operations
- → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (AI-DLC in operations: deployment, observability, incident response)

## Exam triggers
- "Task too big for one context window / highly parallel / many complex tools" → **multi-agent with a supervisor**
- "Complex task needs different expertise + different tool permissions" / "agents team with a coordinator" → **multi-agent collaboration (supervisor + collaborators)**
- "Simple, uniform task" → **single agent** (MAS adds overhead)
- "Collaborator needs context from an earlier step" → orchestration **explicitly passes state** (no implicit shared memory)
- "Prevent data leakage between agents" → **session-scoped memory, no cross-agent memory**, least-privilege tools per agent
- "Agents overwrite each other's updates" → shared **state management** (single source of truth, e.g. DynamoDB) + conflict resolution
- "Which agent failed?" → **agent-level traces/logs** (Bedrock console trace, CloudWatch)
- "Agents stuck looping / runaway cost" → **rate-limit planner invocations**
- "Transient model/network errors in an agent pipeline" → **Step Functions retry/fallback**
- "Draft many sections in parallel" → **Step Functions** parallel branches
- "Human approval before final output" → **HITL stage** before supervisor aggregation
- "Limit risk of autonomous financial actions" → **approval thresholds + human-in-the-loop + rollback**
- "Agent calls a REST API" → **OpenAPI action group** · "custom backend logic" → **Lambda action group** · "ground on company docs" → **Knowledge Base**
- "Change workflow without redeploying agents" → **versioned orchestration JSON / orchestration Lambda**
- "Change agent config without redeploying / at runtime" → **Inline Agents**
- "Non-ReAct workflow logic" → **Custom Orchestration**
- "Stream agent output to the front end" → **AG-UI**
- "Math / logic / live data the LLM can't do" → **agent + tools** (ReAct)
- "LangChain agent stuck in a loop" → **max_iterations**
- "Need controllable custom agent runtime" → **LangGraph** (not AgentExecutor)

## Flashcards
<details>
<summary><b>Q:</b> An agent must call an internal REST API whose endpoints and auth are documented; another needs custom data transformation logic. Which action group type for each?</summary>

REST API -> OpenAPI schema action group; custom backend logic -> Lambda action group. Grounding on company documents -> Knowledge Base.

*D2 · Action group types*
</details>

<details>
<summary><b>Q:</b> What two capabilities do LangChain agents add over plain LLMs?</summary>

Interacting with external sources (search, calculators, APIs, DBs) and running code; they use ReAct to pick tools until a stopping condition.

*D2 · Agent capabilities*
</details>

<details>
<summary><b>Q:</b> How do you prevent data leakage between agents in a multi-agent system?</summary>

Use session-scoped memory with no cross-agent memory sharing, and least-privilege IAM/tools per agent; one agent's tools are not available to others unless shared via orchestration.

*D3 · Agent data leakage*
</details>

<details>
<summary><b>Q:</b> A LangChain agent gets stuck in an endless tool loop. What setting fixes it?</summary>

Set max_iterations on AgentExecutor (e.g. 10), plus handle_parsing_errors=True.

*D5 · Agent loop fix*
</details>

<details>
<summary><b>Q:</b> A collaborator agent needs context produced by an earlier agent in the chain. How does it get it?</summary>

The orchestration layer/supervisor must explicitly pass state (summaries, variables, JSON). Agents do not talk directly and session memory is not shared across agents.

*D2 · Agent memory sharing*
</details>

<details>
<summary><b>Q:</b> Which protocol is used for agent-to-tools, agent-to-agent, and streaming agent output to a UI?</summary>

MCP (agent ↔ tools), A2A (agent ↔ agent), AG-UI (stream to UI) — all supported natively by AgentCore.

*D2 · Agent protocols*
</details>

<details>
<summary><b>Q:</b> An agent orchestrates a multi-step data pipeline where steps depend on each other. Which agent role is this?</summary>

Task coordination (managing order and dependencies). Other roles: reporting/logging, scalability/concurrency, integration/communication.

*D2 · Agent roles*
</details>

<details>
<summary><b>Q:</b> Multiple agents overwrite each other's updates and create conflicting records. What fixes this?</summary>

Shared state management with a single source of truth (e.g., DynamoDB), plus conflict resolution (priority rules) and consensus protocols.

*D2 · Agent state conflicts*
</details>

<details>
<summary><b>Q:</b> How should tools be designed so agents can use them reliably?</summary>

API-first OpenAPI specs, JSON schemas with validation, MCP/function-calling standards, discovery via registries/semantic tags, and structured errors with suggested corrections.

*D2 · Agent-ready tools*
</details>

<details>
<summary><b>Q:</b> A team needs a highly controllable custom agent runtime in LangChain. Should it use AgentExecutor?</summary>

No – AgentExecutor is legacy; migrate to LangGraph.

*D2 · AgentExecutor vs LangGraph*
</details>

<details>
<summary><b>Q:</b> An AI assistant passes context from an external file via embeddings. Which assistant type is this?</summary>

Contextual-aware AI assistant (others: basic zero-shot, prompt template, persona).

*D2 · AI assistant types*
</details>

<details>
<summary><b>Q:</b> During an incident, AI recommends restarting a service. What happens next under AI-DLC?</summary>

Humans review, apply operational judgment and approve; then AI executes. AI does not restart immediately.

*D5 · AI incident response*
</details>

<details>
<summary><b>Q:</b> Why use AI-DLC to build the observability system itself?</summary>

Consistency - the same approach that builds the app builds the systems that support it, with human oversight for compliance.

*D4 · AI-DLC observability*
</details>

<details>
<summary><b>Q:</b> What is the core pattern of AI-DLC in operations?</summary>

AI does repetitive work and recommends, humans approve critical decisions, AI executes.

*D2 · AI-DLC ops principle*
</details>

<details>
<summary><b>Q:</b> In AI-Driven Deployment under AI-DLC, what is the developer's role?</summary>

Approve the AI's deployment plan before any rollout to staging or production; AI packages Units of Work into independently deployable components and manages cross-environment config.

*D2 · AI-driven deployment*
</details>

<details>
<summary><b>Q:</b> What components make up an Amazon Bedrock Agent?</summary>

Instructions, a foundation model (assigned per agent), action groups (Lambda or OpenAPI schema), an optional Knowledge Base, and per-session memory.

*D2 · Bedrock Agent anatomy*
</details>

<details>
<summary><b>Q:</b> What build/test practice is recommended before chaining agents?</summary>

Test each agent alone first, start with a minimal 2-agent prototype, then add tools, memory and conditions; log intermediate states.

*D2 · Building agents*
</details>

<details>
<summary><b>Q:</b> A team wants to change the multi-agent workflow ordering without redeploying any agents. How?</summary>

Keep planner logic in a versioned orchestration JSON config or an orchestration Lambda; version-control the JSON for rollback.

*D2 · Change flow without redeploy*
</details>

<details>
<summary><b>Q:</b> A team wants to compare which FM works best for one agent in the pipeline. What should they use?</summary>

Amazon Bedrock Model Evaluation.

*D5 · Compare agent models*
</details>

<details>
<summary><b>Q:</b> Name the three multi-agent coordination patterns.</summary>

Handoff (sequential A → B → C), parallel (simultaneous tasks), hierarchical (goal-driven supervisor over specialists).

*D2 · Coordination patterns*
</details>

<details>
<summary><b>Q:</b> How do you use a fine-tuned Bedrock model in LangChain?</summary>

Pass the provisioned custom model ARN as model_id along with provider.

*D2 · Custom model in LangChain*
</details>

<details>
<summary><b>Q:</b> Agent workflow logic must go beyond the default ReAct loop. What should be used?</summary>

Custom Orchestration in Bedrock Agents.

*D2 · Custom Orchestration*
</details>

<details>
<summary><b>Q:</b> A startup with a strong team and proprietary algorithms needs full control over its agents. Which path?</summary>

Custom agents with AgentCore + Strands (+ Bedrock Guardrails and AWS services).

*D2 · Customer path*
</details>

<details>
<summary><b>Q:</b> A team wants to minimise cost and improve reliability for a mostly predictable workflow with occasional judgment calls. What architecture is recommended?</summary>

Deterministic orchestration (Step Functions + Lambda) for ~95% of paths, calling agents as targeted tools for the ~5% needing judgment – up to 60% cost savings vs all-agentic.

*D2 · Deterministic vs agentic*
</details>

<details>
<summary><b>Q:</b> What does a LangChain document loader do?</summary>

Loads documents from different sources (DB, online, local, e.g. S3FileLoader) to the LLM to generate embeddings.

*D2 · Document loader*
</details>

<details>
<summary><b>Q:</b> A compliance agent should run automatically after each generation event. Which service triggers it?</summary>

Amazon EventBridge.

*D2 · Event-triggered agents*
</details>

<details>
<summary><b>Q:</b> Compliance requires a person to approve generated grant sections before the final document is assembled. Where does this go?</summary>

Add a human-in-the-loop (HITL) review stage between the Compliance agent and the Planner/Supervisor aggregation.

*D3 · Human approval stage*
</details>

<details>
<summary><b>Q:</b> An AI-DLC incident investigation shows a code change is needed. What is the flow?</summary>

Hand over to the development team, who use AI-DLC to diagnose, fix and deploy.

*D5 · Incident needs code fix*
</details>

<details>
<summary><b>Q:</b> A team needs to change agent configuration at runtime without pre-deploying the agent. Which Bedrock Agents capability fits?</summary>

Inline Agents.

*D2 · Inline Agents*
</details>

<details>
<summary><b>Q:</b> Which SDK client/operation and parameters invoke a Bedrock agent programmatically?</summary>

bedrock-agent-runtime invoke_agent with agentId, agentAliasId, sessionId and inputText.

*D2 · invoke_agent parameters*
</details>

<details>
<summary><b>Q:</b> A RAG app needs enterprise semantic search with SharePoint and Confluence connectors as its retriever. What should it use?</summary>

Amazon Kendra (AmazonKendraRetriever), paired with ConversationalRetrievalChain.

*D2 · Kendra retriever*
</details>

<details>
<summary><b>Q:</b> Which are LangChain components: chains, Ground Truth, memory, HPO, agents?</summary>

Chains, memory and agents (plus models, prompt templates, indexes).

*D2 · LangChain components*
</details>

<details>
<summary><b>Q:</b> A chatbot must answer 'closest beach?' using the earlier mention of Los Angeles. Which LangChain component?</summary>

Memory – ConversationBufferMemory with ConversationChain (not agents).

*D2 · LangChain memory*
</details>

<details>
<summary><b>Q:</b> What is the preferred way to compose chains in LangChain?</summary>

LCEL (LangChain Expression Language), a declarative approach; legacy subclassed chains like LLMChain are older.

*D2 · LCEL*
</details>

<details>
<summary><b>Q:</b> How should risk from autonomous financial actions (e.g., refunds) be contained?</summary>

Confidence scoring (<90% → human approval), approval thresholds (e.g., auto-refund ≤ $100), human-in-the-loop and rollback to last verified value.

*D3 · Limiting agent risk*
</details>

<details>
<summary><b>Q:</b> Which two tasks commonly cause LLM hallucinations that LangChain helps address?</summary>

Handling out-of-domain data and remembering conversational context (LLMs are stateless).

*D2 · LLM limitations*
</details>

<details>
<summary><b>Q:</b> Per the exam guide, where should a lightweight stateless vs complex MCP server be hosted?</summary>

Lambda for a stateless lightweight MCP server; ECS for a complex MCP server.

*D2 · MCP server hosting*
</details>

<details>
<summary><b>Q:</b> Agents run research → write → fact-check → optimise, each using the previous agent's output. Which orchestration pattern is this?</summary>

Pipeline (sequential). Others: hierarchical (supervisor), peer-to-peer (equals negotiate), marketplace (dynamic selection by capability).

*D2 · Multi-agent patterns*
</details>

<details>
<summary><b>Q:</b> A conversation exceeds the context window and the company needs an audit trail of prompts. Which component?</summary>

A prompt history store – persists conversation state and supports audit/compliance and debugging.

*D2 · Prompt history store*
</details>

<details>
<summary><b>Q:</b> Agents generate 10–100x more requests than humans. How should backend services be protected?</summary>

Rate limiting and circuit breakers.

*D4 · Protect services from agents*
</details>

<details>
<summary><b>Q:</b> LangChain agents work in a prototype; the team now needs production scale with managed infra. What should it use?</summary>

Amazon Bedrock AgentCore Runtime (amazon-bedrock-agentcore package).

*D2 · Prototype vs production*
</details>

<details>
<summary><b>Q:</b> A multi-agent grant-writing pipeline must retry on transient model/network errors and draft many sections in parallel. Which service?</summary>

AWS Step Functions: retry/fallback policies and parallel branches.

*D2 · Retries and parallelism*
</details>

<details>
<summary><b>Q:</b> In LangChain, what fetches relevant docs for a query vs stores/queries embeddings?</summary>

Retriever fetches relevant documents (e.g. AmazonKendraRetriever); vector store holds embeddings (OpenSearch Serverless, Aurora pgvector) and can be exposed via .as_retriever().

*D2 · Retriever vs vector store*
</details>

<details>
<summary><b>Q:</b> Which LangChain chain selects a prompt template based on user input?</summary>

RouterChain.

*D2 · RouterChain*
</details>

<details>
<summary><b>Q:</b> Agents are stuck looping and costs are spiking. What control should be applied?</summary>

Rate-limit planner (supervisor) invocations using quotas or Lambda throttling to stop runaway agent loops and cost blowouts.

*D4 · Runaway agent loops*
</details>

<details>
<summary><b>Q:</b> An agent built with LangGraph must persist and resume multi-turn state, recover from the last checkpoint, and fork sessions. Which Bedrock feature?</summary>

Bedrock Session Management APIs (checkpointing integration with LangGraph/LlamaIndex; IAM, KMS CMK optional, tenant isolation).

*D2 · Session Management*
</details>

<details>
<summary><b>Q:</b> What characterises the Strands Agents SDK approach to orchestration?</summary>

Model-driven: the LLM is the orchestrator and dynamically decides which tools to use and when (not rule/template-based). Open source, Python + TypeScript, 20+ built-in tools, deploys onto AgentCore.

*D2 · Strands Agents*
</details>

<details>
<summary><b>Q:</b> What is Strands Agents?</summary>

An open-source, model-first agent SDK (FM drives the loop) with native MCP, integrating Bedrock, Lambda and Step Functions, and supporting any model provider.

*D2 · Strands Agents*
</details>

<details>
<summary><b>Q:</b> What is the difference between Bedrock multi-agent 'Supervisor' and 'Supervisor with routing' modes?</summary>

Supervisor plans, calls collaborators and synthesises; Supervisor with routing sends simple requests straight to one collaborator and falls back to full supervisor mode for complex requests. Collaborators are attached by agent alias.

*D2 · Supervisor with routing*
</details>

<details>
<summary><b>Q:</b> What are the five steps of the Bedrock supervisor-collaborator workflow?</summary>

User request -> Planning (supervisor splits goal and picks collaborators) -> Delegation (orchestration passes input + context) -> Execution (collaborators use own FM and action groups) -> Aggregation (supervisor merges into final answer).

*D2 · Supervisor workflow*
</details>

<details>
<summary><b>Q:</b> How should tool Lambdas used by agents be sandboxed?</summary>

Run them in a VPC with restricted egress, with least-privilege per-agent IAM roles.

*D3 · Tool Lambda isolation*
</details>

<details>
<summary><b>Q:</b> When should you choose a multi-agent system with a supervisor?</summary>

When tasks need heavy parallelisation, information exceeds one context window, or many complex tools are involved.

*D2 · When multi-agent*
</details>

<details>
<summary><b>Q:</b> A task requires distinct expertise (research, summarisation, writing) and each role needs different tool permissions, with auditability. Single agent or multi-agent?</summary>

Multi-agent (supervisor + collaborators). A single agent is preferred only for small or highly uniform tasks because MAS adds overhead.

*D2 · When to use MAS*
</details>

<details>
<summary><b>Q:</b> A multi-agent workflow returns a wrong result. How do you find which agent failed?</summary>

Use agent-level traces/logs: the Bedrock console Test agent trace (expand each step) and CloudWatch logs per agent.

*D5 · Which agent failed*
</details>

---
[← Back to index](README.md)
