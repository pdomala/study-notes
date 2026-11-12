[← Back to index](README.md)

# 🔐 11 · Security, identity & agent security

> Exam domain: D3

## Key points
- Defense in depth with **7 layers** (foundation = policies). Input validation layers: **edge (WAF) → app (Lambda/Comprehend) → model (Bedrock Guardrails)**.
- **Excessive agency** and **insecure output handling** are top OWASP LLM risks. **Indirect** prompt injection arrives via ingested data. **OWASP Agentic AI threats (15)** ≠ OWASP Top 10 for LLMs: goal manipulation → Guardrails; tool misuse, privilege compromise, repudiation, spoofing → AgentCore. Never give agents admin "just in case": **least privilege**.
- Agent auth: **SigV4** = act as the authenticated caller, no delegation · **2LO (client credentials)** = agent acts as **itself** · **3LO** = agent acts **on behalf of the user** with consent. Flow: IdP token → authorizer → token exchange (workload access token) → credential provider → token vault.
- No long-term keys: dev CLI → **`aws login`** · workload → external API → **STS outbound federation (JWT)** · coding assistant writes IAM → **IAM Policy Autopilot** · SaaS secret rotation → **Secrets Manager managed external secrets**.
- PII: S3 → **Macie** · logs → **CloudWatch Logs data protection** · text → **Comprehend** · redact on read → **S3 Object Lambda**. Persona ML roles → **SageMaker Role Manager**. Bots/scrapers → **WAF Bot Control**; legit AI agents on CloudFront → **WAF Web Bot Auth**.
- Detection: **Security Hub** = aggregate & prioritise · **GuardDuty** = suspicious Bedrock API use, attack sequences, malware scanning of S3/backups · **Inspector** = CVEs · **Detective** = forensics · **Security Lake** = security data lake · **Security Agent** = design/code review + pen test.
- Identity: customers → **Cognito**, workforce → **IAM Identity Center**. Multi-account guardrails → **Organizations SCPs + Control Tower**. DDoS → **Shield** (Standard free, L3/L4). Ransomware → **AWS Backup** immutable vaults. Multi-account baseline incl. Bedrock → **AWS SRA**.

## Notes

### Shared Responsibility Model
- **AWS = security OF the cloud** → physical DCs, hardware/network, global infra (Regions, AZs, edge), compute/storage/DB/network software, host OS & virtualisation, operational security of AWS services
- **Customer = security IN the cloud** → customer data & encryption (client- and server-side), platform/apps/**IAM**, **guest OS, network & firewall config** (security groups), network traffic protection
- Split varies by **which services** you choose: **EC2** (most customer work, most control) → VPC → RDS → EMR → S3 → **DynamoDB** (most AWS-managed, least customisation); moving to **abstracted/managed** services = more built-in security best practice
- Security is **Job Zero**; start every engagement / AI security strategy from the Shared Responsibility Model
- Why secure AI: sensitive data, **adversarial attacks** / IP theft, integration into critical decisions
- AWS staff can't access customer data without permission; customer picks Region + encryption
- Market context: AWS holds **143** security/compliance certifications · **2,500+** controls audited annually · **FIPS 140**-validated encryption hardware; fewer than 5% of migrations use AWS-native security services → bake automation in from day one
- 3 AWS security benefits: **most secure infrastructure** (Artifact, Config, Security Hub + GuardDuty) · **security automation** (IAM Access Analyzer, GuardDuty + Security Hub multi-account, Inspector) · **end-to-end security & guidance**

### Defense in depth — 7 layers
- Classic view (inner → outer):
  1. **Data protection** → at rest **KMS** / CMK + **S3 versioning** for data & models · in transit **ACM / AWS Private CA** · keep in VPC with **PrivateLink**
  2. **Identity & access management** → **IAM** (only authorised users/apps/services)
  3. **Application protection** → **Shield**, **Cognito**
  4. **Network & edge** → **VPC**, **WAF**
  5. **Infrastructure protection** → IAM, user groups, **network ACLs**
  6. **Threat detection & incident response** → detect **Security Hub, GuardDuty** · respond **Lambda, EventBridge**
  7. **Policies, procedures & awareness** → least privilege via **IAM Access Analyzer** + **short-term credentials**
- Agentic view (bottom → top): **Policies, procedures & awareness** (foundation: governance & compliance · legal & privacy · risk mgmt · controls · resilience) → **IAM** → **Infrastructure** (VPC security groups, **Network Firewall**) → **Network & edge** (**WAF**, **Shield Advanced**) → **Application + Data protection** (**Guardrails** on prompts + responses, input/output **validation**, **Automated Reasoning**) → **Threat detection & IR**
- Agentic threat detection: monitor prompts/completions · **who accesses data sources & tools** · **anomaly detection** in workloads · monitor **3P tool definitions** agents put into the LLM request · limit access to app data
- Threat-intel feeds: **Shield, GuardDuty, Security Incident Response, WAF, Network Firewall, Route 53 Resolver DNS Firewall, VPC**
- If one control fails, other layers isolate/slow/stop the attacker (no lateral movement, privilege escalation, exfiltration)
- Start defense in depth with **accounts/organization**, then Bedrock/SageMaker security features
- **Input validation layers**: **edge** (sanitise, length, rate limit — **WAF**) → **application** (content, format, business rules — **Lambda**) → **model** (context, semantics, intent — **Bedrock Guardrails**)

### Generative AI Security Scoping Matrix — scopes & disciplines

| Scope | What | Example | Customer controls |
| --- | --- | --- | --- |
| 1 Consumer app | Public 3rd-party GenAI (free/paid), use per ToS | Staff use ChatGPT / **PartyRock** / Midjourney | Own data only |
| 2 Enterprise app | 3rd-party SaaS with embedded GenAI + **business relationship/contract** | **Amazon Q**, scheduling app drafts agendas | Own data only |
| 3 Pre-trained | Your app on an FM via API | Support chatbot on **Claude via Bedrock** (or JumpStart) | Data + **application** |
| 4 Fine-tuned | Fine-tune a 3rd-party FM with your data | Bedrock custom models / **SageMaker JumpStart** | Data + app + **fine-tuning data + fine-tuned model** |
| 5 Self-trained | Train from scratch on data you own | Industry LLM on **SageMaker AI** | **Everything** incl. training data |

- 1–2 = **Buy** (prebuilt), 3–5 = **Build** (self-built); more ownership = more responsibility; every scope **inherits the controls of the scopes before it**
- Data control: **user data** → always customer · **fine-tuning data** → provider in 1–2, **customer in 4** · **training data** → provider in 1–4, **customer only in 5**
- 5 security disciplines across all scopes: **Governance & compliance** (policies, reporting) · **Legal & privacy** (regs, IP; data shared? training data source?) · **Risk management** (threats e.g. insecure output handling, sensitive info disclosure) · **Controls** (who can use which FMs, inference-endpoint access) · **Resilience** (availability, SLAs, Region)
- Data flow (Scopes 1 & 2): prompt → optional plugins/custom data → app formats FM prompt → FM completes → app post-processes → user
- Using the matrix: identify project → **determine scope** (by ownership/control) → assess 5 disciplines → security plan prioritised by criticality
- Integrate with policies: **review existing policies and add AI considerations** (infosec, AI acceptable use) — don't replace them; reference the matrix; define governance roles
- **Service-specific security** → each service's User Guide security chapter (e.g. Bedrock Security User Guide)
- **AWS Security Reference Architecture (SRA)** → prescriptive multi-account security blueprint, includes a **GenAI section focused on Bedrock** · **SRA Verify** validates an existing env · **SRA Code Library** builds · integrates with Landing Zone Accelerator + Well-Architected

### Scoping Matrix — baseline & per-scope controls
- **Baseline for all scopes**: data classification & handling (public/internal/confidential/regulated, audits) · least privilege, **RBAC + MFA**, access reviews · encryption **AES-256 at rest**, **TLS 1.2+** in transit, **KMS** · logging/monitoring (all access, model interactions, data changes) → **CloudWatch** alerts · input sanitisation (prompt injection) + output filtering + content moderation · pen tests, **AI-specific threat modelling**, patching · AI-specific **incident response plan** + tabletop exercises · **privacy by design** (privacy impact assessments, data minimisation, consent) · employee training · vendor assessments + contract clauses · resilience: know model/API limits, **don't lose prompt/completion logs**, include vector/graph DBs in DR plans
- **Scope 1 — Consumer** (provider controls app + FM + data; often not enterprise-ready; treat data as public):
  - Controls: **web proxy / CASB** to restrict traffic · **DLP** on allowed data types · awareness training · access controls
  - Governance: review ToS, data handling, certifications · **acceptable use policy**
  - Legal: data residency · right to be forgotten · **no PII / confidential / IP** · commercial use of outputs · **re-review ToS regularly** (can change without notice) · will provider use your prompts?
  - Risk: data exposure, IP leakage, unsafe/biased output · Resilience: provider availability, **raise client timeouts** for long completions
  - Case: agency staff using **PartyRock** → proxy, DLP, AUP, review ToS, review outputs before client use, fallback method; **direct model modification is not possible**
- **Scope 2 — Enterprise** (key difference = **enterprise agreement/contract**; same data flow; Scope 1 controls still apply):
  - Controls: negotiated contractual security requirements · **vendor secure SDLC** · configurable app controls (**RBAC**)
  - Governance: contract vs your data governance · what data vendor uses · **downstream 3P services** · certifications **SOC 2, ISO 27001**
  - Legal: privacy/IP/**indemnification** · data residency · **opt out of training on your data** · legal review of **EULA** · acceptable data classification
  - Risk: residual risk after contract, threat-model embedded features, IR playbooks
  - Case **Amazon Q Business** → RBAC via **IAM Identity Center**, **VPC endpoints**, encryption, opt-out, residency
- **Scope 3 — Pre-trained** (you control **app + data**; provider controls model + its training data):
  - Flow: user → app (authN/Z) → retrieve + filter your data → build prompt → FM → post-process/filter → user
  - Controls: sanitise/validate input · secure transport/storage to model API · strict access to **inference endpoints** · monitor **data leakage / model inversion** · **RAG** for accuracy · Bedrock built-in security, **IAM**, **VPC endpoints**, API security
  - Governance: data selection policies, versioning + change mgmt, regs (**HIPAA**), audit outputs for bias
  - Legal: model ToS · **anonymise before sending** · IP of outputs · **never tune on PII — no "unlearning" without full retrain**
  - Risk: bias, **prompt injection**, drift, secure redeploys, extend threat model with **OWASP**
  - Resilience: **error handling + fallbacks**, API availability, **multi-Region** HA
  - Case: EHR summariser on **Bedrock** → anonymise, no PII sent, Guardrails **denied topics + content filter thresholds**, custom medical-language filters
- **Scope 4 — Fine-tuned** (adds **fine-tuning data + tuned model**; all Scope 3 controls apply):
  - Controls: sanitise tuning data · secure storage/transmission of base + tuned model · **strict access to fine-tuning + model** · monitor leakage/inversion · secure training pipelines, encrypted model storage, **custom guardrails**, custom validation
  - Governance: tuned model **inherits the classification of its fine-tuning data** (use the **highest**)
  - Legal: base-model licence **permits fine-tuning?** · anonymise · IP · no PII (records **can't be selectively deleted**)
  - Risk: **overfitting**, bias, **poisoning of fine-tuning data**, drift · Resilience: **copy models across Regions**; Regional API differences may need extra tuning
  - Case: fintech fine-tunes Claude on Bedrock → **KMS** for base + custom model artifacts, encrypted S3, **IAM** on tuning jobs, **CloudWatch**, SEC compliance, multi-Region backups
- **Scope 5 — Self-trained** (you take on **all model-provider duties**: choose, collect, clean, curate training data; rare for GenAI, common for predictive ML):
  - Controls: **end-to-end encryption** of data + artifacts · secure whole ML pipeline · **isolated (VPC) compute** for training & inference · monitor training
  - Governance: **model cards + data provenance** · validation/approval before deploy · lifecycle audits · model inherits training-data classification
  - Legal: verify rights to **all** training data · **you own the EULA** · **differential privacy** · GDPR/CCPA
  - Risk: bias, adversarial + model inversion, drift, **secure decommissioning** · Resilience: **save checkpoints**
  - Case (SageMaker): KMS · VPC isolation · IAM · CloudTrail + CloudWatch · model review board · **Config** audits · **Clarify** (bias) · **Model Monitor** (drift) · checkpoints · **multi-AZ** · Systems Manager for IR

### Threat frameworks — OWASP Top 10 for LLMs & MITRE ATLAS
- **OWASP Top 10 for LLMs** (2025 edition for LLM apps): prompt injection · **insecure output handling** · **training data poisoning** · model DoS · supply chain · sensitive info disclosure · insecure plugin design · **excessive agency** · **overreliance** · model theft
  - Impact + **technology-agnostic** mitigations; refreshed regularly (OWASP GenAI site); does **not** recommend specific architectures
  - Use it: understand → assess posture → prioritise by likelihood × impact → define **shared responsibility** (you / cloud / model provider) → implement controls (input validation, output handling, access, supply chain) → monitor & improve
- **MITRE ATLAS** (Adversarial Threat Landscape for AI Systems) = **modified MITRE ATT&CK** for AI/ML; components: **threat model · risk assessment · mitigation strategies · use cases**
  - Matrix: **Tactics** = kill-chain steps across the top (**14**) → **Techniques** under each (**82**; "&" = from ATT&CK) → **Mitigations** per technique
  - Example: tactic **Privilege Escalation** → technique **LLM Prompt Injection** (direct / indirect) → mitigation **Adversarial Input Detection** (block atypical queries / malicious IPs)
  - Use it: familiarise → org assessment → map your controls → prioritise → implement → integrate into risk/vuln/IR processes → monitor
- Other: **AWS CAF Security Perspective**; GenAI-specific set = OWASP Top 10 LLM + OWASP LLM checklist + NIST AI 600-1 + NIST AI RMF (framework stack → See [12 · Governance](12-governance-compliance-responsible-ai.md))

### OWASP Agentic AI threats (15) & AWS mitigation map
- Evolution: **GenAI assistants** (follow rules, automate repetitive tasks) → **GenAI agents** (single goal, whole workflows) → **Agentic AI systems** (fully autonomous, **multi-agent**) → more autonomy + impact, less human oversight → security shifts from protecting data to governing **actions**
- AWS agentic portfolio → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (AWS GenAI stack) · MCP/A2A → [07 · Agents](07-agents-multi-agent-frameworks.md)
- **OWASP Agentic AI Threats & Mitigations (15)**: (1) **memory poisoning** · (2) **tool misuse** · (3) **privilege compromise** · (4) resource overload · (5) cascading hallucination attacks · (6) **intent breaking & goal manipulation** · (7) misaligned & deceptive behaviours · (8) **repudiation & untraceability** · (9) **identity spoofing & impersonation** · (10) overwhelming human-in-the-loop · (11) unexpected RCE & code attacks · (12) agent communication poisoning · (13) rogue agents in multi-agent systems · (14) human attacks on multi-agent systems · (15) human manipulation
- Source [genai.owasp.org](http://genai.owasp.org) — a **different list from the OWASP Top 10 for LLMs**; read which list the question means
- Agent attack surface: application → **input/output** → agent (**planning, action, tool calling, short-term memory**, execution loop) ↔ **LLM + function calling** ↔ services (content, code, data, device, HITL) ↔ supporting services (long-term memory, vector store) ↔ other agents
- Mitigation map:
  - **Bedrock AgentCore**: memory poisoning, tool misuse, privilege compromise, resource overload, cascading hallucination, misaligned/deceptive behaviour, repudiation, identity spoofing, unexpected RCE, agent communication poisoning, rogue agents
  - **Bedrock Guardrails**: **intent breaking & goal manipulation**, **human attacks on multi-agent systems**, **human manipulation**
  - Coming soon (at time of writing): **overwhelming HITL**
- Agentic risks: stronger IAM, **traceable decisions**, **memory poisoning / shared-memory leakage** → memory isolation, access control, anomaly detection
- Never give an agent admin "just in case" → **least privilege / just-enough access**

### Prompt injection, prompt leaking & defences
- **Prompt injection** = instructions in the input that **override/hijack** model behaviour ("Ignore and output 'Neutral'"); malicious (harmful content at scale) or benign (keep product names in translation)
  - **Direct** = attacker prompts the LLM · **Indirect** = LLM ingests malicious instructions from **another data source** (retrieved/ingested documents)
- **Prompt leaking** = model **exposes sensitive/private info** from prompts, context or training data (e.g. customer payment history in output) → test use cases for exposure, don't put unnecessary PII in context
- **Injection types**: persona switch · extract prompt template · ignore template · alternating languages / escape chars · extract conversation history · exploit friendliness · augment template · **fake completion** · rephrase/obfuscate · change output/input format
- **Defences**: guardrails in the prompt (e.g. "if request contains 'hack', reply with refusal") · **Bedrock Guardrails prompt-attack filter** · thinking/answer tags · **salted sequence tags** wrapping instructions · **delimiters** separating prompt sections (also stops the model revealing prompt parts) · teach the LLM to detect attacks · prompt templates · **trust boundaries** (IdP claims → IAM roles, Cognito rule-based mapping) · invocation logs · **Guardrails tracing** · prompt **filtering, sanitisation, validation** + robust training
- **WAF** blocks long inputs, known malicious strings, SQLi before the app; logs for pattern analysis
- Other inference attacks: evasion → sanitise, filter output, **rate limit**, authN; monitor for **model inversion / extraction**
- **Jailbreak** = input crafted to bypass safety rules

### AI-specific threats — poisoning, inversion, supply chain, trust boundaries
- AI-specific security considerations: **fake content generation, data manipulation, automated attacks, model weaknesses** → AI-powered detection over traffic/behaviour; vulnerability mgmt (bugs, model weaknesses, attack vectors) via pen testing, code reviews, patching
- Securing GenAI protects against **model extraction, data poisoning, adversarial inputs, unauthorised access/misuse**; techniques: adversarial training, differential privacy, tamper-evident watermarking
- **Data poisoning** (training, fine-tuning or RAG repo; backdoor triggers; risk rises with auto-ingest) → vet data, **version-controlled pipelines**, watch sudden output changes, restrict user contributions, adversarial testing on a copy
- **Model inversion / data leakage** → monitor inference for leakage + strict access to inference endpoints
- Trust boundaries: **UI** (request/response monitoring, data outflow) · **application** (authZ bypass to downstream tools; **Network Firewall + VPC** ingress/egress) · **model** (supply chain — models can embed runnable code → mirrors + scanning) · **data** (poisoning → Security Hub, GuardDuty, Inspector, Detective, Security Lake) · **organizational** (threat model with **Threat Composer**, security chaos engineering)
- Data privacy: governance over which data trains/infers · classify + restrict categories · anonymise / synthetic data · **watermarking + output monitoring** to detect leakage · filter secrets/PII
- Pipeline security: encrypt at rest + TLS in transit, least privilege + RBAC, masking/tokenisation, audit logs, anomaly detection
- Well-Architected GenAI Lens (all pillars incl. security) → See [13 · Cost & monitoring](13-cost-latency-monitoring.md) (KPIs & Well-Architected GenAI Lens)

### Identity & access — IAM, Cognito, Identity Center, re:Invent 2025 identity launches
- Goal: **who can access what** → identity + access + resource mgmt toward **Zero Trust / least privilege**; approach: choose identity source (AWS or existing IdP) → managed/custom policies → scale with boundaries across accounts → **continuous analysis** to refine
- **IAM** → users/groups/roles/policies, guardrails, fine-grained access, **temporary credentials** for workloads; Bedrock: allow/deny models, which services can infer, console login
- **IAM Access Analyzer** → unused/external access, policy validation, **custom policy checks pre-deployment**, refine on actual usage; least privilege with **IAM Identity Center + Access Analyzer**
- **Cognito** → **customer** (CIAM) identity, social/enterprise IdPs, B2B multi-tenancy, role-based access to S3/DynamoDB/Lambda — *not* for workforce · **IAM Identity Center** → workforce SSO
- **Cognito M2M pricing** (GA 3 Nov) → pay only for **successful token requests**, not per M2M app client
- Zero trust: **Verified Access** (no VPN) + **Verified Permissions** (fine-grained authz)
- **SageMaker Role Manager** → persona-based roles: **data scientist, MLOps, SageMaker AI compute**
- **IAM Policy Autopilot** → **MCP server** letting coding assistants (Kiro, Claude Code, Cursor, any MCP tool) author/troubleshoot IAM policies via **deterministic code analysis** → valid IAM actions, baseline policies fast
- **STS outbound identity federation** → `GetWebIdentityToken` → short-lived signed **JWT** (**60 s – 1 h**) to auth to **external services** instead of stored API keys · verified via public **JWKS** (**ES384 / RS256**) · IAM condition keys (audience, algorithm, lifetime) · **no cost, all Regions incl. GovCloud + China**
- **IAM temporary delegation** (select ISV partners) → time-limited privileged access for partner setup, optional admin approval
- **`aws login`** → Console sign-in creds for CLI/SDK/3P tools, browser-based, **no long-term access keys**, temp tokens auto-expire
- Troubleshooting auth/network errors → See [14 · Evaluation & troubleshooting](14-evaluation-testing-troubleshooting.md) (Bedrock troubleshooting)

### Agent authentication — inbound/outbound, SigV4 vs 2LO vs 3LO
- Inbound vs outbound auth (AuthN/AuthZ questions) → See [08 · AgentCore](08-bedrock-agentcore.md) (Identity)

| Pattern | Acts as | Use | Example |
| --- | --- | --- | --- |
| **AWS credentials (SigV4)** | authenticated user | direct user → AWS service, own AWS creds, **no token exchange/delegation** | user accesses S3 or invokes agent on AgentCore Runtime |
| **2LO (client credentials) = M2M** | the **service** | server ↔ server, client app auths directly with auth server | agent calls company KB / weather API with its own creds |
| **3LO** | the **user** | user consent via auth server, app acts on user's behalf | agent calls Google/Slack for the user |

- IAM questions to ask: trust between agents in multi-agent systems? user/group/agent-specific granular permissions? strict tool access verification & policies? preventing agent or human **impersonation**? which **memory & DB systems** each user may access?
- Identity services: **Cognito** (customer identity + **M2M**) · **AgentCore Identity** (identity & credential mgmt for agents) · **IAM** (access to AWS resources)
- AgentCore Identity flow, token vault, custom claims, Gateway 3LO, Gateway Policy Engine → See [08 · AgentCore](08-bedrock-agentcore.md) (Identity; Gateway; Policy)

### Network & edge protection
- **PrivateLink** (VPC interface endpoint) → private VPC connectivity to Bedrock, no internet; VPC Lambda calling Bedrock → PrivateLink also cuts latency
- Practices: **VPC endpoints, security groups, NACLs**; layers: security groups (host) → **Network Firewall** (VPC-wide) → app protection (exploits, DDoS, bots) → centralised policy → visibility
- **Network Firewall** → managed, auto-scaling, custom or AWS-managed rules, CloudWatch metrics; **deep packet / TLS inspection** to stop exfiltration (+ VPC + PrivateLink to Bedrock)
- **WAF** → filter traffic, account-takeover fraud, **Bot Control** (scrapers/crawlers); + **Shield Advanced** + **Firewall Manager** for central management
- **Shield Standard** → **free**, L3/L4 DDoS, best with **CloudFront + Route 53**, inline mitigation, no latency impact · **Shield Advanced** → managed DDoS protection
- re:Invent 2025 launches:
  - **Network Firewall Proxy (Preview)** → **explicit forward proxy** for egress: DNS domain, header/URL/IP filtering, anti-SNI spoofing, content-type controls; attaches to NAT Gateway via PrivateLink; HTTP/S, **optional TLS decryption** → prevent exfiltration
  - Network Firewall **flexible cost allocation** (charge TGW + NFW processing back to VPC/app teams) · **partner managed rules** in Marketplace (Trend, Fortinet, Check Point, Infoblox, Rapid7, Lumen, ThreatSTOP)
  - **Route 53 DNS Firewall DGA protection** (blocks Domain Generation Algorithm domains) · **Route 53 Global Resolver (Preview)** → anycast DNS from anywhere, filter **DNS exfiltration**, encrypt, log
  - **WAF Web Bot Auth** → **cryptographic verification of AI agents** hitting CloudFront (IETF standard), allow verified agents, block bad bots, **no extra cost**
  - **CloudFront flat-rate plans** → one monthly price, **no overages**, bundles CDN + WAF + DDoS + Route 53 DNS + CloudWatch logs + S3 credits
  - **VPC encryption controls** → **monitor** mode (visibility via flow logs) vs **enforce** mode (block unencrypted) within/across VPCs in a Region; Nitro **AES-256-GCM**, no perf penalty; HIPAA/PCI/FedRAMP; auto-migrates NLB/ALB/Fargate

### Threat detection & incident response services
- **Security Hub** = command centre / **CSPM** → single findings dashboard, aggregates + correlates (threats, vulns, controls, configs, network exposure, sensitive data) → prioritise + **automated response/playbooks**
  - re:Invent 2025: **Security Hub (AI-powered)** unified CSPM, near-real-time risk analytics · **simplified pricing** = one bill (vs separate GuardDuty/Inspector/CSPM), resource-based, built-in cost estimator
- 4 foundational services for any workload: **Security Hub · KMS · GuardDuty · Shield Advanced**
- **GuardDuty** → threat detection: credential exfiltration, **suspicious Bedrock/SageMaker API usage** (e.g. login from a new location then **removing Guardrails** or changing the training-data S3 bucket)
  - **Extended Threat Detection attack sequences** → multistage findings with timelines, **MITRE ATT&CK** mappings; scenarios: compromised identity, S3 exfil, EKS, **NEW: EC2 + ECS** (crypto-mining, reverse shells, container escape)
  - **Malware Protection for S3 – on-demand** → scan **existing** objects, detect, tag, **quarantine** (S3 data pipelines)
  - **Malware Protection for AWS Backup** → agentless scans of EC2/EBS/S3 backups, auto + on-demand (incremental), verify **before restore**
- **Inspector** → continuous scanning of EC2, container images, Lambda: **package CVEs, code vulns, network reachability**; risk score from **NVD** adjusted to your env
- **Detective** → forensic investigation · **Security Lake** → security data lake · **Macie** → sensitive data · **Security Incident Response** → incident mgmt
- **Security Incident Response AI investigative agent** → correlates evidence across AWS data sources, builds timeline, actionable summaries → **hours → minutes**
- **AWS Security Agent (Preview)** → **design review** (PDF/DOCX/images, **max 5 files**, before coding) · **code review** (e.g. GitHub) · **on-demand context-aware pen testing**
- Automate IR/compliance: **Security Hub, Config, Audit Manager, Artifact**; respond with **Lambda + EventBridge**
- CloudTrail (guardrail config changes, key events; ≠ inference content) → See [12 · Governance](12-governance-compliance-responsible-ai.md) (AWS governance & compliance services)
- **Bedrock abuse detection** → fully automated, **no human review/access** to inputs/outputs; classifiers on Titan + 3P models, pattern detection, **CSAM** hash matching

### Data protection, encryption, PII & resilience
- **Bedrock data privacy**: data **not used for service improvement / base-model training**, **not shared** with model providers; encrypted **in transit + at rest**; customisation trains a **private copy**; custom models encrypted with **KMS**
- **KMS** (AWS-managed or customer-managed keys) → encrypts base + custom model artifacts · **ACM** public/private SSL/TLS certs (with CloudFront/ELB) · **Private CA post-quantum ML-DSA** certs (GA 10 Nov) for the quantum-safe PKI transition
- Practices: **IAM least privilege** · encryption in transit + at rest, **DLP** · privacy-enhancing tech: masking, obfuscation, **differential privacy**, encryption, **tokenization**, secure multi-party computation · **AWS Privacy Reference Architecture (PRA)**
- **Access control** → RBAC + fine-grained permissions, SSO/MFA/IAM, log all access, least-privilege reviews · **data integrity** → schema/referential/business-rule validation, backup & recovery, transactions/atomicity, lineage & audit trails
- Securing training data = **control access + ensure integrity** (cleaning/inconsistency checks = data quality, not security)
- **PII services**: **Macie** (ML discovery of PII/PHI/financial data in **S3**, e.g. before training/fine-tuning; DBs → extract to an S3 data lake first) · **Comprehend** (detect + **redact PII** in text; pre-trained **prompt safety classifier** before the LLM; sentiment, moderation) · **CloudWatch Logs data protection policies** (audit + **mask PII in logs**) · **S3 Object Lambda** (redact on retrieval) · **Glue DataBrew** (visual cleaning) · **Rekognition** (image/video moderation)
- **Secrets Manager managed external secrets** → 3P SaaS creds, **managed rotation** via partner code (no custom Lambda), standard pricing, IAM/CloudWatch/CloudTrail/GuardDuty controls
- Data-protection challenges: evolving regs, regional variation, reporting burden; top worry = **ransomware + business continuity**; 4 areas: critical assets (KMS, Macie) · automation (Backup) · integration (ACM) · compliance (**Backup Audit Manager**)
- **AWS Backup** → central policy-based backups, **cross-Region/account** recovery + restore testing, **immutable backups & logically air-gapped vaults** (ransomware), compliance reporting

## Exam triggers
- "Staff use ChatGPT / PartyRock / public chatbot" → **Scope 1** → web proxy/**CASB** + **DLP** + AUP
- "Amazon Q (Business) / SaaS with GenAI under enterprise contract" → **Scope 2** → opt out of training, EULA review, SOC 2/ISO 27001
- "Our app calls a Bedrock FM (Claude via API) + RAG" → **Scope 3**
- "Fine-tune FM with our data / SageMaker JumpStart fine-tune / customer controls fine-tuning data" → **Scope 4**
- "Train from scratch / customer controls training data" → **Scope 5**
- "Classification of a fine-tuned model?" → **same as (highest) fine-tuning data**
- "Remove a customer's PII from a trained model" → **impossible without full retrain** → anonymise **before** tuning
- "Privacy-preserving training" → **differential privacy** · "resume long training" → **checkpoints**
- "Multi-account security baseline incl. Bedrock" → **AWS SRA**
- "Adversary tactics/techniques against ML" → **MITRE ATLAS** · "Top LLM app vulnerabilities" → **OWASP Top 10 for LLMs**
- "Model given too much autonomy" → OWASP **excessive agency** · "Unsanitised model output used downstream" → **insecure output handling** · "Malicious data in training set" → **training data poisoning**
- "Input tries to override instructions" → **prompt injection** · "Output reveals private/system data" → **prompt leaking**
- "Indirect prompt injection" → malicious instructions in **retrieved/ingested data**, not the user prompt
- "Model inversion / data leakage" → monitor inference + strict endpoint access
- OWASP **Agentic** threats (15) ≠ OWASP **Top 10 for LLMs** — read which list the question means
- "Goal hijacking / human manipulation of agents" → **Bedrock Guardrails** · "Tool misuse, privilege compromise, untraceable actions" → **AgentCore** (Identity, Gateway/Policy, Observability)
- "Agent given broad perms just in case" → wrong → **least privilege** (privilege compromise / excessive agency)
- "Agent calls Slack/Google on behalf of the user with consent" → **3LO OAuth** (Gateway 3LO) · "Agent calls an internal API as itself" → **2LO / client credentials (M2M)** · "User calls AWS directly, no delegation" → **SigV4**
- "Machine-to-machine auth for customer apps" → **Cognito M2M**
- "Customer-facing sign-in, social IdPs" → **Cognito** · "Workforce SSO" → **IAM Identity Center**
- "Coding assistant writes least-privilege IAM" → **IAM Policy Autopilot (MCP server)**
- "AWS workload calls external API without stored keys" → **STS outbound federation (JWT)** · "Developer CLI without access keys" → **`aws login`**
- "Persona-based ML roles" → **SageMaker Role Manager**
- "Reach Bedrock without internet / private connectivity from VPC" → **PrivateLink**
- "Inspect egress TLS" → **Network Firewall** · "VPC-wide traffic filtering" → **Network Firewall** · "Egress domain filtering via explicit proxy" → **Network Firewall Proxy**
- "DNS exfiltration" → **Route 53 Global Resolver / DNS Firewall**
- "Block bots/scrapers" → **WAF Bot Control** · "Verify legitimate AI agents/crawlers hitting CloudFront" → **WAF Web Bot Auth**
- "Block long/malicious requests before the app" → **WAF**
- "Free DDoS L3/L4" → **Shield Standard**
- "Block unencrypted VPC traffic" → **VPC encryption controls (enforce)**
- "Single pane for findings + prioritise" → **Security Hub** · "Centralise security logs for analytics" → **Security Lake**
- "Scan Lambda/ECR/EC2 for CVEs" → **Inspector** · "Forensic investigation" → **Detective** · "Suspicious API/credential use (e.g. from a new location)" → **GuardDuty**
- "Security review of design docs before code / on-demand pen test" → **AWS Security Agent** · "Speed up incident investigation" → **Security Incident Response AI agent**
- "Scan existing S3 objects for malware" → **GuardDuty Malware Protection for S3 on-demand** · "Clean backup before restore" → **GuardDuty for AWS Backup**
- "Find PII in S3 (before fine-tuning)" → **Macie** · "PII in logs" → **CloudWatch Logs data protection policy** · "PII in text" → **Comprehend**
- "Rotate 3rd-party SaaS credentials without Lambda" → **Secrets Manager managed external secrets**
- "Quantum-safe certificates" → **Private CA ML-DSA**
- "Ransomware-proof backups" → **AWS Backup immutable / air-gapped vault** · "Backup compliance reports" → **Backup Audit Manager**
- Bedrock abuse detection = **automated, no human access** to your prompts/outputs

## Flashcards
<details>
<summary><b>Q:</b> An agent must call Slack on behalf of the user with consent; another calls an internal weather API as itself; a user calls an AWS service directly. Which auth pattern each?</summary>

On behalf of user with consent -> 3LO OAuth; as itself -> 2LO client credentials (M2M); direct user to AWS without delegation -> SigV4.

*D3 · 3LO vs 2LO vs SigV4*
</details>

<details>
<summary><b>Q:</b> How does Amazon Bedrock abuse detection handle customer prompts and outputs?</summary>

It is fully automated with classifiers, pattern detection and CSAM hash matching - no human review of or access to inputs/outputs.

*D3 · Abuse detection*
</details>

<details>
<summary><b>Q:</b> Is it acceptable to give an agent admin access 'just in case'?</summary>

No - apply least privilege / just-enough access (otherwise privilege compromise / excessive agency).

*D3 · Admin just in case*
</details>

<details>
<summary><b>Q:</b> In agentic defense in depth, which controls apply at the application/data layers and which is the foundation?</summary>

Guardrails, input/output validation and Automated Reasoning at application/data layers; policies, procedures & awareness are the foundation.

*D3 · Agent defense in depth*
</details>

<details>
<summary><b>Q:</b> Map identity services: customer identity + M2M; identity and credential management for agents; access to AWS resources.</summary>

Amazon Cognito; AgentCore Identity; AWS IAM.

*D3 · Agent identity services*
</details>

<details>
<summary><b>Q:</b> What should threat detection monitor for agentic workloads?</summary>

Prompts/completions, who accesses data sources and tools, anomalies in workloads, and third-party tool definitions agents put into the LLM request.

*D3 · Agent threat monitoring*
</details>

<details>
<summary><b>Q:</b> In AgentCore Identity, what happens to the user's external IdP token after inbound auth?</summary>

Agent token exchange swaps it for an AWS-generated workload access token passed to the hosted agent/tools.

*D3 · Agent token exchange*
</details>

<details>
<summary><b>Q:</b> Tool misuse, privilege compromise and untraceable agent actions are concerns. Which AWS offering mitigates them?</summary>

Amazon Bedrock AgentCore (Identity, Gateway/Policy, Observability).

*D3 · AgentCore mitigations*
</details>

<details>
<summary><b>Q:</b> Which OWASP agentic threats are mitigated by Bedrock Guardrails rather than AgentCore?</summary>

Intent breaking & goal manipulation, human attacks on multi-agent systems, and human manipulation.

*D3 · Agentic threat mitigation*
</details>

<details>
<summary><b>Q:</b> A healthcare provider is drowning in security alerts from multiple services. With least manual effort, what should it use?</summary>

AWS Security Hub to centralise, prioritise and automate response (not manual GuardDuty review).

*D3 · Alert overload*
</details>

<details>
<summary><b>Q:</b> Developers need CLI/SDK access without long-term access keys. What should they use?</summary>

`aws login` - browser-based, uses console sign-in credentials, issues auto-expiring temporary tokens.

*D3 · aws login*
</details>

<details>
<summary><b>Q:</b> How does an agent hosted on AgentCore reach AWS resources (vs external resources)?</summary>

Via its IAM execution role; external resources use outbound OAuth/API keys from resource credential providers stored in the token vault.

*D3 · AWS resources from agent*
</details>

<details>
<summary><b>Q:</b> A team wants security review of design documents before coding and on-demand context-aware pen testing. Which service?</summary>

AWS Security Agent (Preview): design review (max 5 files), code review and penetration testing.

*D3 · AWS Security Agent*
</details>

<details>
<summary><b>Q:</b> Daily backups with 7-year retention and proof of compliance are required. What should be used?</summary>

AWS Backup with automated policies, retention rules and compliance reporting (Backup Audit Manager).

*D3 · Backup compliance*
</details>

<details>
<summary><b>Q:</b> What encryption baseline applies to all GenAI scopes?</summary>

AES-256 at rest and TLS 1.2+ in transit, with keys in AWS KMS.

*D3 · Baseline encryption*
</details>

<details>
<summary><b>Q:</b> Is customer data sent to Bedrock used to train base models or shared with model providers?</summary>

No. Data is not used for service improvement, not shared with providers, encrypted in transit and at rest; customisation trains a private copy, encrypted with KMS.

*D3 · Bedrock data privacy*
</details>

<details>
<summary><b>Q:</b> A growing multi-Region organisation needs central identity, fine-grained access and access analysis for internal teams. What approach?</summary>

IAM first, then IAM Access Analyzer to refine permissions on actual usage (not Cognito, which is for customer identity).

*D3 · Central identity refine*
</details>

<details>
<summary><b>Q:</b> Why store metadata alongside chunks in a vector store?</summary>

For references/traceability of answers and to enforce ACL requirements (restrict sensitive info by authorisation level).

*D3 · Chunk metadata ACL*
</details>

<details>
<summary><b>Q:</b> Customer-facing sign-in with social IdPs vs workforce SSO - which service for each?</summary>

Customer-facing (CIAM) -> Amazon Cognito; workforce SSO -> IAM Identity Center.

*D3 · Cognito vs Identity Center*
</details>

<details>
<summary><b>Q:</b> Which service encrypts base and custom (fine-tuned) model artifacts on Bedrock?</summary>

AWS KMS (with encrypted S3, IAM on tuning jobs, CloudWatch monitoring).

*D3 · Custom model encryption*
</details>

<details>
<summary><b>Q:</b> How do you mitigate data poisoning in training, fine-tuning or RAG repositories?</summary>

Vet data, use version-controlled pipelines, watch for sudden output changes, restrict user contributions and do adversarial testing on a copy.

*D3 · Data poisoning*
</details>

<details>
<summary><b>Q:</b> A team must inspect outbound TLS traffic from a GenAI workload to prevent data exfiltration. Which service?</summary>

AWS Network Firewall (deep packet / TLS inspection), with VPC and PrivateLink to Bedrock.

*D3 · Egress exfiltration*
</details>

<details>
<summary><b>Q:</b> Outbound traffic must go through an explicit forward proxy with domain/URL filtering and optional TLS decryption to prevent exfiltration. Which feature?</summary>

AWS Network Firewall Proxy (Preview).

*D3 · Egress proxy*
</details>

<details>
<summary><b>Q:</b> Which capability correlates evidence across AWS data sources and builds a timeline to cut incident investigation from hours to minutes?</summary>

AWS Security Incident Response AI investigative agent.

*D3 · Faster IR investigation*
</details>

<details>
<summary><b>Q:</b> Which GuardDuty capability correlates signals into multistage findings with MITRE ATT&amp;CK mappings, now including EC2 and ECS?</summary>

GuardDuty Extended Threat Detection (attack sequences).

*D3 · GuardDuty attack sequences*
</details>

<details>
<summary><b>Q:</b> Developers want their coding assistant (Kiro, Claude Code, Cursor) to generate valid least-privilege IAM policies. What should they use?</summary>

IAM Policy Autopilot, an MCP server that uses deterministic code analysis to produce valid IAM actions.

*D3 · IAM Policy Autopilot*
</details>

<details>
<summary><b>Q:</b> What is indirect prompt injection?</summary>

The LLM ingests malicious instructions from another data source (e.g. retrieved/ingested documents), rather than the attacker prompting it directly. ATLAS mitigation: adversarial input detection.

*D3 · Indirect prompt injection*
</details>

<details>
<summary><b>Q:</b> How do you mitigate prompt injection?</summary>

Add guardrails in the prompt (e.g. refusal rule for specific requests) plus model-level protection such as the Bedrock Guardrails prompt-attack filter; use delimiters to separate sections.

*D3 · Injection mitigation*
</details>

<details>
<summary><b>Q:</b> What are the three input validation layers for GenAI apps and the service at each?</summary>

Edge (sanitise, length, rate limit - AWS WAF) -> application (content, format, business rules - Lambda) -> model (context, semantics, intent - Bedrock Guardrails).

*D3 · Input validation layers*
</details>

<details>
<summary><b>Q:</b> Which service provides machine-to-machine auth for customer applications, priced per successful token request?</summary>

Amazon Cognito M2M (pricing: pay only for successful token requests).

*D3 · M2M for customer apps*
</details>

<details>
<summary><b>Q:</b> A GenAI data pipeline must scan existing S3 objects for malware and quarantine them. Which feature?</summary>

GuardDuty Malware Protection for S3 on-demand scanning (detect, tag, quarantine).

*D3 · Malware in S3 data*
</details>

<details>
<summary><b>Q:</b> An attacker injects false data into an agent's long-term memory to influence future actions. Which OWASP agentic threat is this?</summary>

Memory poisoning (threat 1), mitigated by AgentCore.

*D3 · Memory poisoning*
</details>

<details>
<summary><b>Q:</b> How do you mitigate model inversion / data leakage for an FM-based app?</summary>

Monitor inference for leakage and enforce strict access to inference endpoints.

*D3 · Model inversion*
</details>

<details>
<summary><b>Q:</b> A multi-account healthcare startup needs consistent controls and audit logs. What should it use?</summary>

AWS Control Tower guardrails (preventive + detective) plus AWS CloudTrail.

*D3 · Multi-account controls*
</details>

<details>
<summary><b>Q:</b> Multi-tenant authorisation rules must be driven by claims from the IdP token. Which feature?</summary>

AgentCore Identity custom claims.

*D3 · Multi-tenant agent auth*
</details>

<details>
<summary><b>Q:</b> An e-commerce site faces network attacks and must stay available during peaks with central management. Which services?</summary>

AWS Network Firewall + AWS Shield Standard via centralised policies.

*D3 · Network attacks + DDoS*
</details>

<details>
<summary><b>Q:</b> An LLM agent is given too much autonomy and permissions. Which OWASP Top 10 for LLMs risk is this?</summary>

Excessive agency. (Unsanitised model output used downstream = insecure output handling; malicious training data = training data poisoning.)

*D3 · OWASP excessive agency*
</details>

<details>
<summary><b>Q:</b> How does the OWASP Agentic AI Threats &amp; Mitigations list differ from the OWASP Top 10 for LLMs?</summary>

Agentic list has 15 threats (e.g. memory poisoning, tool misuse, rogue agents, overwhelming HITL); Top 10 for LLMs covers prompt injection, excessive agency, insecure output handling etc.

*D3 · OWASP lists*
</details>

<details>
<summary><b>Q:</b> A team wants adversary tactics and techniques against ML systems mapped to mitigations, vs a list of top LLM app vulnerabilities. Which framework for each?</summary>

Tactics/techniques -> MITRE ATLAS (ATT&CK adapted for AI: 14 tactics, 82 techniques). Top LLM vulnerabilities -> OWASP Top 10 for LLMs (technology-agnostic; does not recommend architectures).

*D3 · OWASP vs ATLAS*
</details>

<details>
<summary><b>Q:</b> A team needs persona-based IAM roles for data scientists, MLOps and SageMaker compute. Which feature?</summary>

Amazon SageMaker Role Manager.

*D3 · Persona ML roles*
</details>

<details>
<summary><b>Q:</b> Before fine-tuning, a team must discover PII/PHI/financial data in their training data in S3. Which service?</summary>

Amazon Macie (data in databases should first be extracted to an S3 data lake).

*D3 · PII before fine-tuning*
</details>

<details>
<summary><b>Q:</b> Which service finds PII in S3, masks PII in logs, and detects/redacts PII in text?</summary>

Amazon Macie (S3); CloudWatch Logs data protection policies (logs); Amazon Comprehend (text, also a prompt safety classifier).

*D3 · PII services*
</details>

<details>
<summary><b>Q:</b> Which feature validates IAM policies and runs custom policy checks before deployment, and finds unused/external access?</summary>

IAM Access Analyzer.

*D3 · Pre-deploy policy checks*
</details>

<details>
<summary><b>Q:</b> Bedrock must be reached from a VPC without traversing the internet. What should be used?</summary>

AWS PrivateLink (VPC interface endpoint).

*D3 · Private connectivity*
</details>

<details>
<summary><b>Q:</b> Distinguish prompt injection from prompt leaking.</summary>

Prompt injection = input instructions override/hijack model behaviour. Prompt leaking = model exposes sensitive/private info from prompts, context or training data.

*D3 · Prompt injection vs leaking*
</details>

<details>
<summary><b>Q:</b> How do you make backups resilient to ransomware?</summary>

AWS Backup immutable backups and logically air-gapped vaults, with cross-Region/account copies.

*D3 · Ransomware backups*
</details>

<details>
<summary><b>Q:</b> A company wants agents to authenticate users from existing Okta/Entra ID without migrating users. Which service?</summary>

AgentCore Identity, which reuses existing IdPs (Okta, Entra ID, Cognito, Auth0).

*D3 · Reuse existing IdP*
</details>

<details>
<summary><b>Q:</b> Third-party SaaS credentials used by an app must be rotated without writing a custom Lambda. Which feature?</summary>

AWS Secrets Manager managed external secrets (partner-provided rotation).

*D3 · SaaS secret rotation*
</details>

<details>
<summary><b>Q:</b> How do you restrict the maximum permissions across all accounts in an organisation?</summary>

Service control policies (SCPs) in AWS Organizations.

*D3 · SCP boundaries*
</details>

<details>
<summary><b>Q:</b> A team wants to centralise security logs from many sources into a data lake for analytics. Which service?</summary>

Amazon Security Lake.

*D3 · Security Lake*
</details>

<details>
<summary><b>Q:</b> How does the customer's share of responsibility change from EC2 to DynamoDB?</summary>

EC2 = most customer work and control; more abstracted managed services (e.g., DynamoDB) = AWS handles more, with more built-in security best practice.

*D3 · Shared responsibility by service*
</details>

<details>
<summary><b>Q:</b> What does AWS Shield Standard provide and at what cost?</summary>

Free L3/L4 DDoS protection for all customers, best used with CloudFront and Route 53, inline mitigation with no latency impact.

*D3 · Shield Standard*
</details>

<details>
<summary><b>Q:</b> An AWS workload must authenticate to an external service without storing API keys. What should it use?</summary>

STS outbound identity federation: `GetWebIdentityToken` returns a short-lived signed JWT (60 s - 1 h) verified via JWKS (ES384/RS256); no cost.

*D3 · STS outbound federation*
</details>

<details>
<summary><b>Q:</b> Map: suspicious Bedrock API/credential usage; forensic investigation of an incident.</summary>

Suspicious API/credential use -> Amazon GuardDuty. Forensic investigation -> Amazon Detective.

*D3 · Threat detection services*
</details>

<details>
<summary><b>Q:</b> Users are repeatedly prompted for consent and the agent needs to store third-party OAuth tokens and API keys securely. What should be used?</summary>

AgentCore Identity resource token vault (with resource credential providers), minimising consent fatigue.

*D3 · Token vault*
</details>

<details>
<summary><b>Q:</b> A regulated workload must block unencrypted traffic within/across VPCs. What should be used?</summary>

VPC encryption controls in enforce mode (monitor mode gives visibility only).

*D3 · VPC encryption controls*
</details>

<details>
<summary><b>Q:</b> Which service continuously scans EC2, container images and Lambda for package CVEs, code vulnerabilities and network reachability?</summary>

Amazon Inspector, with risk scores from NVD adjusted to your environment.

*D3 · Vulnerability scanning*
</details>

<details>
<summary><b>Q:</b> A public GenAI chat app is being scraped by bots. Which control?</summary>

AWS WAF with Bot Control (plus Shield Advanced and Firewall Manager for central management).

*D3 · Web app protection*
</details>

<details>
<summary><b>Q:</b> A site on CloudFront must allow legitimate AI agents while blocking bad bots. Which feature verifies the agents cryptographically?</summary>

AWS WAF Web Bot Auth (IETF standard, no extra cost); bad bots in general -> WAF Bot Control.

*D3 · Web Bot Auth*
</details>

<details>
<summary><b>Q:</b> Which services provide zero-trust access without VPN and fine-grained application authorisation?</summary>

AWS Verified Access (no VPN) and Amazon Verified Permissions (fine-grained authz).

*D3 · Zero trust access*
</details>

---
[← Back to index](README.md)
