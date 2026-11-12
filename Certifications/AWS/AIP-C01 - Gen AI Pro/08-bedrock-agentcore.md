[← Back to index](README.md)

# 🧱 08 · Amazon Bedrock AgentCore

> Exam domain: D2

## Key points
- Map: **Runtime** (host/scale, voice streaming) · **Gateway** (tools + access control) · **Memory** · **Identity** (OAuth) · **Policy** (Cedar) · **Evaluations** (13 evaluators) · **Observability** · Code Interpreter · Browser · plus Harness, Registry, Optimization, Payments.
- **Runtime**: each session in its own **Firecracker microVM**; idle **5 min → suspended** (state kept), **15 min → terminated**, **8 h max**; 500 active sessions/account. Cross-session state → **Memory**. `agentcore configure` creates **IAM execution role + ECR repo**.
- **Gateway**: OpenAPI / Lambda / Smithy / MCP-server targets → MCP tools at `/mcp`; built-in throttling, multitenancy, request/response transformation; hundreds of tools → **semantic search** instead of listTools; **3LO OAuth** for user consent.
- Never forward the user's JWT downstream (**confused deputy**) → act-on-behalf tokens via Identity. Tools without OAuth (e.g. AWS services) → Gateway + Identity give uniform OAuth/MCP.
- **Identity**: inbound = user → agent (IAM/OAuth via Cognito/Okta/Entra) · outbound = agent → tools (IAM, OAuth, API key) from the **token vault**; custom claims support multi-tenancy.
- **Memory**: short-term = raw events/checkpoints (sync) · long-term = facts, preferences, summaries extracted async into namespaces; built-in strategies **user preferences · summarisation · semantic** (or custom).
- **Code Interpreter** = sandboxed code execution · **Browser** = web automation with session replay (web-UI-only systems).
- **Guardrails = filter content** · **AgentCore Policy = govern actions** (Cedar or natural language, intercepts every tool call through Gateway). **Harness** = model + system prompt + tools in one API (migration target from Agents Classic). **Registry** = governed catalog · **Optimization** = A/B test prompts/tool descriptions · **Payments** = x402 + spending limits.

## Notes

### Overview — what AgentCore is and why
- **AgentCore** = modular, composable services to build, deploy & operate **production-grade agents** securely at scale with **any framework** (Strands, LangChain/LangGraph, CrewAI) + **any model** (in or outside Bedrock); services usable **together or independently** and added **incrementally without rewriting agent code**
- Why: open-source frameworks (context, tool execution, multi-agent collaboration, orchestration) make **PoCs** easy; production needs the "undifferentiated heavy lifting": securely **execute & scale** code · **remember** past interactions · **identity & access** for agents and tools · agentic **tool use** · **discover/connect** custom tools · **understand & audit** every interaction
- Provides the plumbing you'd otherwise build: session mgmt, identity, memory, observability, security, web automation, code execution
- Pillars: **time to value** (no infra) · **flexible** (any framework/model) · **trusted** (secure, scalable, reliable)
- Architecture: App ↔ **Runtime** (framework + instructions + local tools + context) ↔ Gateway / Browser / Code Interpreter, with **Identity**, **Memory**, **Observability** around it · **Policy** sits alongside Identity on the tool path (Runtime → Identity/Policy → Gateway/MCP, Browser, Code Interpreter) · **Evaluations** sits on top of Observability (OTEL)
- Building blocks: original 7 = **Runtime · Memory · Identity · Gateway · Code Interpreter · Browser · Observability** → re:Invent 2025 added **Policy + Evaluations (Preview)** → later **Harness · Registry · Optimization · Payments**
- Stack position → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (AWS GenAI stack) · Strands + MCP build path and MCP / A2A / AG-UI protocols → See [07 · Agents](07-agents-multi-agent-frameworks.md) (Strands; Agent-ready tools & protocols)
- **Bedrock Agents → "Bedrock Agents Classic"**, closed to new customers since **30 Jul 2026** → AgentCore is the recommended path; **Harness** = migration target
- Benefits: **flexible integration** (model/framework/tool agnostic, auto-scales) · **accelerated development** (fully managed) · **enhanced monitoring** (dashboards, CloudWatch) · **security** (enterprise-grade, session isolation)
- Use cases: **equip agents with tools & capabilities** (internal/external tools + remembering users) · **deploy securely at scale** (any framework/protocol/model, agent identity & access) · **test & monitor** (token usage, latency, session duration, error rates)

### Runtime
- Secure **serverless** agent hosting; real-time → **multi-hour** workloads, extended runtime **up to 8 hours** (vs Lambda 15 min); fast cold starts; **multi-modal payloads**; framework-agnostic (LangGraph, CrewAI, Strands)
- Deploy any open-source-framework agent; PoC → prod in a few lines; supports **MCP / A2A / AG-UI** (connect agents via A2A)
- **VPC, PrivateLink, CloudFormation**; built-in identity, integrates with existing IdPs, VPC + identity propagation
- **True session isolation**: each session in its own **Firecracker microVM** (compute + memory + filesystem) — other serverless may share a microVM across sessions → local files/state could leak
  - Alternative framing: each agent in its own **isolated container** (process + filesystem isolation), CPU/GPU resource mgmt, auto **horizontal/vertical scaling**
- Stateful **within** a session; **out-of-session state → AgentCore Memory**
- Lifecycle: **5 min idle → suspended** (state, files, env vars kept; CPU cut) · **15 min → session timeout** (environment terminated) · **8 h max** session
- Default limits: **500 active sessions** per account · **15 min** request timeout · **60 min** streaming
- **Bi-directional streaming** for natural **voice** conversations with real-time interruption
- Deploy flow: agent code + **Runtime decorator** + Observability/Identity config → **Dockerfile** → image in **ECR** → **Runtime Agent** + **Runtime Endpoint** → app invokes endpoint
- Python SDK `pip install bedrock-agentcore`: `BedrockAgentCoreApp()` + `@app.entrypoint` + `app.run()` → keep your agent logic, zero infra
- Entry-point function receives the **JSON payload**; to use another framework, **swap the agent invocation inside the entry point**

### Gateway
- Solves the **M × N** integration problem (every app × every tool) via **MCP** → agent uses one unified API / one secure endpoint
- Transforms **APIs, Lambda, services → agent-ready MCP tools** without custom code; target types:
  - **OpenAPI** (REST, third-party APIs) · **Smithy** models (AWS services) · **Lambda** (systems without standard APIs / multi-step logic) · existing **MCP servers** · internal APIs
- Agent (MCP client) → `/mcp` over **streamable-HTTP**: **list tools, invoke tool, search**
- **Dual auth** (ingress + egress): **inbound** IAM or OAuth token (IdP: Cognito, Okta, Auth0) · **outbound** per target: IAM (Lambda/Smithy), API key or OAuth token (OpenAPI/MCP) via **resource credentials provider**; logs to **CloudWatch**
- Uses **Identity** to give OAuth to tools that lack it (e.g. AWS services) → uniform MCP interface
- Built-in cross-cutting: authN/Z, **throttling**, request/response **transformation**, **multitenancy**, **tool selection** (most relevant tools for the task)
- **Semantic tool search**: Gateway auto-indexes tools; `list tools` returns all (e.g. 300+) vs `search` returns the few most relevant → less context, better **accuracy, speed, cost**
- Can front other Gateways, Runtime-hosted agents/MCP servers, public MCP servers → **unified MCP gateway**
- Also: request routing, native MCP, central **tool registry** (discovery, versioning, testing), load-balances across Lambda, external APIs, MCP tools
- re:Invent 2025: **Gateway Policy Engine** = fine-grained access control · Gateway tools get **3LO (three-legged OAuth)** for user consent
- Best practices: `exceptionLevel: DEBUG` on CreateGateway · timeouts on tool calls · start **read-only**, add mutation only with HITL · **least privilege** · clear tool descriptions · **search instead of listTools** at scale · group tools by use case / outbound authorizer / domain · confirm sensitive ops, validate tool results before the LLM
- Security: zero-trust · **don't propagate incoming JWTs** to outgoing calls (**confused deputy**) → **act-on-behalf tokens** · detect **tool poisoning** · multi-tenancy = tenant ID in secure token + fine-grained access

### Identity
- Challenges solved: access control, **delegation**, identity lifecycle at scale, SaaS/3P integration (Google, ServiceNow, Jira, GitHub, Salesforce, Slack), governance
- Distinct identities for agents (**agent identity directory**); **reuses existing IdPs** (Okta, Entra ID, Cognito, Auth0) → no user migration or rebuilt auth flows; **secure token vault** → reduces **consent fatigue**; just-enough access & delegation to AWS + 3P tools
- **Inbound auth** = users → agent via app (**IAM/OAuth**) — AuthN: who is this user? · AuthZ: may they call this agent?
- **Outbound auth** = agent → tools/resources (**IAM** for AWS, **OAuth / API keys** for external) — AuthN: is this agent who it claims? · AuthZ: may it access this resource **on behalf of the user**?
- Components: **secure token vault**, token caching, **workload identity**, OAuth2 authorizer, **resource credential provider** (e.g. agent on Runtime calling Azure OpenAI GPT)
- Flow:
  1. **Agent identity directory** → each agent has an agent identity
  2. User → customer IdP (Auth0, Entra ID, Okta, Cognito) issues access token → **inbound auth** checked by the Runtime/Gateway **IAM/OAuth authorizer**
  3. **Agent token exchange** → swaps external token for an **AWS-generated workload access token** passed to the hosted agent/tools
  4. **Outbound auth** → **resource credential providers** get creds for external resources (OAuth or API keys)
  5. **Resource token vault** stores those tokens · AWS resources reached via the **IAM execution role**
- Without Identity every tool call uses the **same access level** → create **workload identity** → configure **credential providers** → Python **decorator** on functions needing a token
- **Token vault** stores OAuth 2.0 user tokens (user consent to act on their behalf; refresh) and **API keys**
- **Custom (OAuth) claims** → multi-tenant auth rules
- Works with both **Runtime** and **Gateway**; admins integrate IdPs/auth servers + view detailed **auth activity log**
- Auth patterns (SigV4 / 2LO / 3LO) and OWASP agentic threats → see [11 · Security](11-security-identity-agent-security.md)

### Memory
- Without memory each conversation starts cold → with memory: contextual intelligence, user preferences, knowledge retention (**context persistence**)
- **Short-term (raw) memory**: multi-turn conversation context within a session, session attributes, **agent state / checkpointing**, retrieve messages — written as **events** (sync)
- **Long-term (intelligent) memory**: **user preferences, session summary, semantic facts**, behaviour customisation — **automatic async extraction** from events → memory records; persists **across sessions**
- Strategies: built-in **user preferences · summarisation · semantic** + **custom** extraction; multiple strategies per conversation
- **Episodic memory** (re:Invent 2025) → agents learn from past experiences
- Fully managed, **built-in encryption**, **namespace**-based segmentation/sharing; retrieval by **semantic query** + filter by namespace; handles embeddings & consolidation
- Memory stores **shareable across agents** (collaborating agents) and sessions
- Up to **30 days** retention, unique memory IDs
- Also: vector storage, semantic search, context mgmt (conversation history, task state, learned preferences), multimodal memory
- Usage: create memory client + store → write turns to short-term → load recent turns; new session starts blank → add long-term strategy

### Policy
- New block (Preview, re:Invent 2025) for trusted production agents, launched with Evaluations
- **Deterministic rules** written in **natural language** (converted to **Cedar**) or Cedar directly
- **Intercepts every tool call via Gateway**; sits alongside **Identity** on the tool path · **Gateway Policy Engine** = fine-grained access control for business rules
- **Guardrails filter content · Policy governs actions**

### Evaluations (overview & architecture)
- → See [14 · Evaluation](14-evaluation-testing-troubleshooting.md) (AgentCore Evaluation: why & architecture, online vs on-demand Evaluate API, levels + 13 built-in evaluators, online config limits, trace prerequisites, IAM role, custom evaluators)

### Observability
- Trace / debug / monitor each step; step-by-step visualisation; **OpenTelemetry**-compatible — OTEL traces of reasoning, inputs, outputs, tool calls → export to existing platforms
- Dashboards: sessions, traces, spans, errors · **token usage, latency, session duration, error rates**; integrates with **CloudWatch**
- End-to-end **traces & spans** → lower **MTTD / MTTR**
- Works for agents on Runtime **and hosted elsewhere**; integrates with **3P observability tools**
- On by default in Runtime → enable **CloudWatch Transaction Search** (account-wide, cost-effective trace ingestion via Application Signals pricing); re-launch with observability enabled; IAM role needs **CloudWatch permissions**
- Evaluation → trace debugging loop → See [14 · Evaluation](14-evaluation-testing-troubleshooting.md) (Interpreting evaluation results)

### Browser & Code Interpreter
- **Browser** = cloud browser for agents to **interact with websites at scale** (systems with web UI only)
  - Serverless, low-latency sessions, 0 → hundreds concurrent · **VM-level isolation** per session, VPC modes, secure credentials
  - **Live streaming / live view + session replay** for audit; commands logged to **CloudTrail**
  - Related: **Nova Act** = model trained to take actions in a web browser
- **Code Interpreter** = write/run code in an **isolated sandbox** for workflows, calculations, data analysis, charts
  - **JavaScript, TypeScript, Python** runtimes with common libs + custom packages
  - Large datasets via **S3**; configurable instance types & session properties

### Harness, Registry, Optimization, Payments
- **Harness** → managed agent loop: **model + system prompt + tools in one API call**, no container/orchestration code, isolated microVM with filesystem + shell · **migration target from Bedrock Agents Classic** · Harness = fastest; **Runtime** = bring your own framework code
- **Registry** → governed catalog of agents, MCP servers, tools, skills (**publish → review → approve**; hybrid semantic + keyword search)
- **Optimization** → AI recommendations + versioned config bundles + **A/B tests** on system prompts & tool descriptions (traffic split via Gateway); builds on Evaluations + Observability
- **Payments** → agent microtransactions via **x402**, wallet integration, **spending limits**

### Pricing, regions & shared responsibility
- **Pricing**: **consumption-based**, no upfront / minimum fees; pay per service used (each service independent); other AWS services billed normally
- **Regions (at time of writing)**: us-east-1 (N. Virginia), us-west-2 (Oregon), eu-central-1 (Frankfurt), **ap-southeast-2 (Sydney)** — check current list
- **Security**: **AWS shared responsibility model** — AWS secures the infrastructure; you own your content + service security configuration

### CLI / SDK walkthrough — Strands prototype → production
- Prototype: customer support agent drafting email replies from CRM, order history, KBs → productionised step by step:

1. **Runtime** — starter toolkit CLI in a venv → `agentcore configure` auto-creates **IAM execution role** + **ECR repo** (stores container image) → launch locally → deploy → check endpoint status → invoke
2. **Memory** — memory client + store; short-term turns; long-term strategies for new sessions
3. **Identity** — workload identity → credential providers → decorator on token-needing functions
4. **Gateway** — reach CRM/order systems over MCP (Smithy / Lambda / internal / OpenAPI targets)
5. **Code Interpreter** (calculations) + **Browser** (web-UI-only integrations)
6. **Observability** — traces & spans, dashboards; Transaction Search + CloudWatch IAM permissions
7. Result: prototype → production **incrementally, existing agent code kept**

### LangChain vs AgentCore
- LangChain integrates with Bedrock via `langchain-aws` / `langchain-community`; ⚠️ **production deployments →** `amazon-bedrock-agentcore` (scalability + managed agent infra)

| Scenario | Use |
| --- | --- |
| Prototyping / development | **LangChain agents** |
| Production deployment at scale | **AgentCore Runtime** |
| Managed memory across sessions | **AgentCore Memory** |
| Tool/API integration with access control | **AgentCore Gateway** |
| Agent monitoring & evaluation | **AgentCore Evaluations** |

## Exam triggers
- "New agent / migrate off Bedrock Agents" → **AgentCore** (**Harness** = fastest, no orchestration code; **Runtime** = bring your own framework)
- "Run any agent framework in prod with managed scaling/identity/observability" → **AgentCore**
- "Prototype" → LangChain · "production scale" → **AgentCore Runtime**
- "Long-running agent, hours of work" → **Runtime** (8 h) — not Lambda (15 min)
- "State/files must not leak between users' sessions / isolate each user session" → **Runtime microVM session isolation**
- "Voice agent / natural conversation" → **Runtime bi-directional streaming**
- "`agentcore configure` creates" → **IAM execution role + ECR repository**
- "Persist context across sessions / agent forgets context in new session" → **Memory** (Runtime state dies at 15 min idle / 8 h)
- "Remember preferences / summaries" → **long-term memory** strategies · "checkpoint conversation / within session" → **short-term** events
- "Remember user across sessions / share memory between collaborating agents" → **Memory long-term** (shared stores)
- "Agent remembers / learns from past interactions" → **Memory (episodic)**
- "Expose existing REST APIs / Lambda as agent tools without code" → **Gateway** targets (OpenAPI / Lambda / Smithy → MCP)
- "Turn Lambda/APIs into MCP tools with ingress + egress auth, throttling, multitenancy" → **Gateway**
- "Tools with fine-grained access control" → **Gateway** (Policy Engine)
- "Hundreds of tools bloating the prompt" → Gateway **semantic search**
- "Tool with no OAuth (AWS service) behind uniform MCP interface" → **Gateway + Identity**
- "Agent tool needs user consent to a 3rd-party API" → **Gateway 3LO OAuth**
- "Agent calls 3P SaaS on user's behalf / OAuth APIs without re-prompting users for consent / store 3P OAuth tokens & API keys" → **Identity outbound OAuth + token vault**
- "Users sign in to the agent" / "who is this user, may they invoke the agent?" → **Identity inbound auth** · "Agent calls tools/APIs" → **outbound auth**
- "Agent must act with the user's scoped permissions" → **Identity**
- "Keep existing Okta/Entra users" → **AgentCore Identity** (no migration)
- "Multi-tenant auth rules from IdP claims" → **Identity custom claims**
- "Forwarding user JWT downstream" → ❌ confused deputy → **act-on-behalf tokens**
- "Restrict which tools/actions an agent may execute, under conditions / business rules" → **AgentCore Policy** (Cedar / natural language) — not Guardrails
- "Agent must run generated code safely / calculations / data analysis" → **Code Interpreter**
- "Navigate websites + session replay / system only has a web UI" → **Browser** (or **Nova Act**)
- "Trace reasoning + tool calls into existing observability stack / find bottlenecks" → **Observability (OTEL)**
- "Traces not showing up" → enable **observability in config** + **CloudWatch Transaction Search** + check **IAM role permissions**
- "Measure agent quality before and after deploy / score production quality" → **Evaluations**
- "A/B test prompts or tool descriptions" → **Optimization**
- "Org-wide catalog of approved MCP servers/agents" → **Registry**

## Flashcards
<details>
<summary><b>Q:</b> An organisation needs a governed catalog of approved MCP servers, agents and tools. What should it use?</summary>

AgentCore Registry (publish → review → approve; hybrid semantic + keyword search).

*D3 · Agent catalog governance*
</details>

<details>
<summary><b>Q:</b> Which AgentCore service measures agent quality before and after deployment, and which A/B tests prompts and tool descriptions?</summary>

AgentCore Evaluations (built-in/custom/third-party evaluators; on-demand, batch or online) and AgentCore Optimization (A/B tests via Gateway traffic split).

*D5 · Agent evaluation*
</details>

<details>
<summary><b>Q:</b> Which AgentCore capability enables agent microtransactions, and with what protocol?</summary>

AgentCore Payments, using x402 with wallet integration and spending limits.

*D2 · Agent payments*
</details>

<details>
<summary><b>Q:</b> List the AgentCore building blocks after re:Invent 2025.</summary>

Runtime, Memory, Identity, Gateway, Code Interpreter, Browser, Observability, plus new Policy and Evaluations.

*D2 · AgentCore components*
</details>

<details>
<summary><b>Q:</b> What does the AgentCore starter toolkit <code>agentcore configure</code> command create?</summary>

An IAM execution role and an ECR repository (stores the container image).

*D2 · agentcore configure*
</details>

<details>
<summary><b>Q:</b> Agent tool calls through AgentCore Gateway must obey business rules with fine-grained access control. Which capability?</summary>

AgentCore Policy (Gateway policy engine), which sits alongside Identity on the tool path.

*D2 · AgentCore Policy*
</details>

<details>
<summary><b>Q:</b> How is AgentCore priced?</summary>

Consumption-based per service used, with no upfront or minimum fees; other AWS services are billed normally.

*D4 · AgentCore pricing*
</details>

<details>
<summary><b>Q:</b> A new customer wants to build agents on AWS in late 2026; Bedrock Agents Classic is unavailable. What is recommended and what is the migration target?</summary>

AgentCore; Bedrock Agents Classic closed to new customers since 30 Jul 2026, and AgentCore Harness is the migration target.

*D2 · Bedrock Agents migration*
</details>

<details>
<summary><b>Q:</b> An agent navigates websites and auditors need to review its actions. Which service and features?</summary>

AgentCore Browser — VM-isolated sessions with live streaming, session replay, and commands logged to CloudTrail.

*D3 · Browser audit*
</details>

<details>
<summary><b>Q:</b> An agent must safely run model-generated code for data analysis on large S3 datasets. Which service?</summary>

AgentCore Code Interpreter — isolated sandbox with JavaScript, TypeScript and Python runtimes, reading data from S3.

*D2 · Code Interpreter*
</details>

<details>
<summary><b>Q:</b> A design forwards the user's incoming JWT to downstream tool calls. What is the risk and fix?</summary>

Confused deputy risk; don't propagate incoming JWTs — use act-on-behalf tokens instead.

*D3 · Confused deputy*
</details>

<details>
<summary><b>Q:</b> Runtime state disappears after idle timeout. Where should out-of-session state persist?</summary>

AgentCore Memory.

*D2 · Cross-session state*
</details>

<details>
<summary><b>Q:</b> An agent tool needs user consent to access a third-party API via AgentCore Gateway. What mechanism?</summary>

Gateway three-legged OAuth (3LO).

*D2 · Gateway 3LO*
</details>

<details>
<summary><b>Q:</b> What inbound and outbound auth options does AgentCore Gateway support?</summary>

Inbound: IAM or OAuth (IdPs like Cognito, Okta, Auth0). Outbound per target: IAM (Lambda/Smithy), API key or OAuth (OpenAPI/MCP) via resource credentials provider.

*D3 · Gateway auth*
</details>

<details>
<summary><b>Q:</b> What tool-exposure practices are recommended for AgentCore Gateway security?</summary>

Start read-only, add mutations only with human-in-the-loop, least privilege, confirm sensitive ops, validate tool results before the LLM, set tool timeouts.

*D3 · Gateway best practice*
</details>

<details>
<summary><b>Q:</b> What built-in cross-cutting features does AgentCore Gateway provide?</summary>

AuthN/Z (dual inbound + outbound), throttling, request/response transformation, multitenancy and tool selection.

*D2 · Gateway cross-cutting*
</details>

<details>
<summary><b>Q:</b> Which setting gives detailed errors when troubleshooting AgentCore Gateway?</summary>

`exceptionLevel: DEBUG` on CreateGateway (logs go to CloudWatch).

*D5 · Gateway debugging*
</details>

<details>
<summary><b>Q:</b> Existing Lambda functions and APIs must become MCP tools with ingress and egress auth. Which service?</summary>

AgentCore Gateway.

*D2 · Gateway MCP conversion*
</details>

<details>
<summary><b>Q:</b> How do agents connect to AgentCore Gateway and what operations are available?</summary>

As MCP clients to the `/mcp` endpoint over streamable-HTTP: list tools, invoke tool, search.

*D2 · Gateway protocol*
</details>

<details>
<summary><b>Q:</b> An agent with 300+ tools suffers from a bloated prompt, poor accuracy and high cost. What Gateway feature helps?</summary>

Semantic tool search — use search instead of listTools to return only the most relevant tools.

*D4 · Gateway semantic search*
</details>

<details>
<summary><b>Q:</b> Which Gateway target type fits AWS services vs systems without standard APIs vs third-party APIs?</summary>

Smithy models for AWS services; Lambda for systems without standard APIs or multi-step logic; OpenAPI for third-party APIs.

*D2 · Gateway target types*
</details>

<details>
<summary><b>Q:</b> When choose AgentCore Harness over AgentCore Runtime?</summary>

Harness = managed agent loop (model + system prompt + tools in one API call, no container/orchestration code); Runtime = bring your own framework code.

*D2 · Harness vs Runtime*
</details>

<details>
<summary><b>Q:</b> An agent must call third-party SaaS on the user's behalf and users must sign in to the agent. Which AgentCore Identity capabilities apply?</summary>

Inbound auth (IAM/OAuth via IdP) for users → agent; outbound OAuth/API keys with the token vault for agent → SaaS.

*D3 · Identity inbound/outbound*
</details>

<details>
<summary><b>Q:</b> Can AgentCore services be adopted one at a time for an existing Strands prototype?</summary>

Yes — services are modular and composable and can be added incrementally without rewriting agent code.

*D2 · Incremental adoption*
</details>

<details>
<summary><b>Q:</b> An agent must run multi-hour tasks; Lambda's 15-minute limit is too short. What should host it and what is the max session?</summary>

AgentCore Runtime — serverless with sessions up to 8 hours.

*D2 · Long-running agents*
</details>

<details>
<summary><b>Q:</b> An agent forgets user preferences in a new session. What should be configured?</summary>

AgentCore Memory long-term with built-in strategies: user preferences, summarisation, semantic (or custom).

*D2 · Memory long-term strategies*
</details>

<details>
<summary><b>Q:</b> Distinguish AgentCore short-term and long-term memory.</summary>

Short-term = raw events (conversation context, agent state/checkpointing) written synchronously; long-term = asynchronously extracted user preferences, session summaries and semantic facts.

*D2 · Memory types*
</details>

<details>
<summary><b>Q:</b> AgentCore agent traces are not showing up in CloudWatch. What should you check?</summary>

Enable observability in the config, enable CloudWatch Transaction Search, and ensure the IAM role has CloudWatch permissions.

*D5 · Missing traces*
</details>

<details>
<summary><b>Q:</b> What does AgentCore Observability provide for operations teams?</summary>

End-to-end traces and spans, built-in dashboards (token usage, latency, session duration, error rates), OpenTelemetry export — lowering MTTD/MTTR.

*D4 · Observability metrics*
</details>

<details>
<summary><b>Q:</b> Can AgentCore Observability monitor agents not hosted on AgentCore Runtime?</summary>

Yes — it works for agents on Runtime and hosted elsewhere, and integrates with third-party observability tools.

*D4 · Observability scope*
</details>

<details>
<summary><b>Q:</b> An agent must be restricted from executing certain tool actions under specific conditions. Use Guardrails or AgentCore Policy?</summary>

AgentCore Policy — deterministic rules (natural language or Cedar) intercepting every tool call via Gateway. Guardrails filter content; Policy governs actions.

*D3 · Policy vs Guardrails*
</details>

<details>
<summary><b>Q:</b> How is an agent packaged for AgentCore Runtime using the Python SDK?</summary>

Wrap code with `BedrockAgentCoreApp()` + `@app.entrypoint` + `app.run()`, build a Docker image to ECR, then create a Runtime agent and endpoint the app invokes.

*D2 · Runtime deployment*
</details>

<details>
<summary><b>Q:</b> What are AgentCore Runtime's idle suspension, session timeout and max session values?</summary>

5 min idle → suspended (state kept), 15 min → session timeout (environment terminated), 8 h maximum session.

*D4 · Runtime lifecycle limits*
</details>

<details>
<summary><b>Q:</b> What are the default AgentCore Runtime active session, request timeout and streaming limits?</summary>

500 active sessions per account, 15-minute request timeout, 60-minute streaming.

*D4 · Runtime quotas*
</details>

<details>
<summary><b>Q:</b> Without AgentCore Identity, what access level do agent tool calls use, and how does Identity fix it?</summary>

Every tool call uses the same access level; Identity adds workload identity, credential providers and a token vault so the agent acts with the user's scoped permissions.

*D3 · Scoped tool access*
</details>

<details>
<summary><b>Q:</b> State and files must not leak between different users' agent sessions. Which AgentCore feature provides this?</summary>

AgentCore Runtime session isolation — each session runs in its own Firecracker microVM (compute, memory, filesystem).

*D3 · Session isolation*
</details>

<details>
<summary><b>Q:</b> Under the shared responsibility model for AgentCore, what does the customer own?</summary>

Their content and the security configuration of the services; AWS secures the infrastructure.

*D3 · Shared responsibility*
</details>

<details>
<summary><b>Q:</b> How do you run a different agent framework on AgentCore Runtime?</summary>

Swap the agent invocation inside the entry-point function, which receives the JSON payload.

*D2 · Switching frameworks*
</details>

<details>
<summary><b>Q:</b> A tool (e.g., an AWS service) has no OAuth support but must sit behind a uniform MCP interface. What combination is used?</summary>

AgentCore Gateway + AgentCore Identity.

*D2 · Tool lacks OAuth*
</details>

<details>
<summary><b>Q:</b> Which AgentCore capability supports natural voice conversations?</summary>

AgentCore Runtime bi-directional streaming.

*D2 · Voice agents*
</details>

<details>
<summary><b>Q:</b> An agent must integrate with a system that only has a web UI and no API. Which AgentCore service?</summary>

AgentCore Browser.

*D2 · Web UI only system*
</details>

---
[← Back to index](README.md)
