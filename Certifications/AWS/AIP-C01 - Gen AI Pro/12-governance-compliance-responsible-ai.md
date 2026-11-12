[← Back to index](README.md)

# ⚖️ 12 · Governance, compliance & responsible AI

> Exam domain: D3

## Key points
- **Scoping Matrix**: 1 Consumer app (ChatGPT/PartyRock) · 2 Enterprise app (Amazon Q) · 3 Pre-trained model via API (Bedrock) · 4 Fine-tuned · 5 Self-trained. 1–2 = **Buy**, 3–5 = **Build**. Fine-tuning data is customer-only in **Scope 4**, training data in **Scope 5**; user data is always the customer's.
- Scope 1 vs 2 differ by **contract/business relationship**. Scope 1 controls = web proxy/CASB + DLP + training. Scope 2 = opt out of training, EULA, SOC 2/ISO 27001. Scope 5 = you own the EULA, differential privacy, model cards, checkpoints, **Clarify** for bias.
- Framework stack: **EU AI Act** = regulation · **ISO 42001** = AI management standard · **NIST AI RMF / 600-1** = risk frameworks · **MITRE ATLAS** (tactics → techniques → mitigations) / **OWASP Top 10 LLM** = threats. US federal → **NIST 800-53** · health → **HIPAA** · EU data → **GDPR** · cards → **PCI DSS**.
- Governance services: **Artifact** = AWS compliance reports · **Audit Manager** (GenAI Best Practices Framework v2) = automated evidence · **Config** = configuration history · **CloudTrail** = who called what · **SageMaker Model Cards** = intended use, risk rating, evaluation.
- Responsible AI dimensions: fairness, explainability, privacy & security, safety, controllability, veracity & robustness, governance, transparency (**scalability is not one**). Bias fixes: **TIED** (clarify prompt) · **counterfactual data augmentation** · **equalized odds**.
- GenAI security has three aspects: security **OF** GenAI · GenAI **FOR** security · security **FROM** GenAI threats. Start any AI security strategy from the **Shared Responsibility Model**.

## Notes
### Security vs governance vs compliance; AI governance framework
- **Security** → confidentiality, integrity, availability (**CIA**) of data, assets, infra
- **Governance** → organisation adds value and **manages risk**
- **Compliance** → normative **adherence to requirements** across the org
- Governance framework: **AI governance board** (cross-functional: legal, compliance, privacy, AI SMEs) → **define roles & responsibilities** → **policies & procedures across the whole AI lifecycle**
- **GRC** = align IT with business goals + manage risk + meet regs; challenges: innovation vs controls, evolving regs, agility, cost efficiency
- CAF-AI Governance perspective → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (Adoption strategy — AWS CAF-AI)

### Compliance standards
- AWS supports **143** security standards/certifications; the customer decides its own risk tolerance
- **NIST 800-53** → **US federal** information systems (formal evaluation/approval for CIA)
- **ENISA** → EU cybersecurity agency, certification schemes
- **ISO** → security management practices based on **ISO/IEC 27002** (ISO 27001 certification checked for Scope 2 vendors)
- **SOC** reports → **independent third-party** assessments of AWS controls
- **HIPAA** → US **protected health information**
- **GDPR** → **EU citizens'** privacy/personal data (also **CCPA** in California)
- **PCI DSS** → **card data**, run by the PCI Security Standards Council (Amex, Discover, JCB, Mastercard, Visa)
- Other AI-relevant: **ISO/IEC 27090, 38507:2022, 23053:2022**; no single global AI regime; domain regs e.g. **SEC** (finance), **GDPR/FCRA** (explainability of decisions)

### Why AI compliance is harder; regulated workloads
- **Complexity/opacity** · **dynamism** (changes after deployment) · **emergent capabilities** · **unique risks** (algorithmic bias from biased training data or human bias) · **algorithm accountability** laws (EU AI Act, NYC Automated Decision Systems law)
- **Regulated workload** → must comply with HIPAA/GDPR/PCI DSS etc.; industries: financial services, healthcare, aerospace; examples: **HR, safety, inspection/regulatory**; regulated processes (FDA reporting), outcomes (**mortgage/credit decisions**), usage (safety-critical), liabilities
- Indicators: **ML Guardrails** "project context" red → probably regulated · ask: need to **audit**? **archive** for a period? predictions = a **record**? source data has **org-restricted classifications**?

### AI regulation & framework stack
- Layered stack (bottom → top):
  1. GenAI applications
  2. **Regulation** — non-negotiable, country/region laws (**EU AI Act**, Canadian **AIDA**)
  3. **Compliance standards** — business requirements (**ISO 42001** AI management system, ISO 27090, ISO 38507)
  4. **Security & risk mgmt frameworks** — **NIST AI RMF**, **NIST AI 600-1** (GenAI risk profile), **OWASP LLM AI Cybersecurity & Governance Checklist**
  5. **Threat mitigation guidance** — **OWASP Top 10 for LLM Apps**, **MITRE ATLAS**, **AWS Well-Architected** (threat-framework detail → See [11 · Security](11-security-identity-agent-security.md))
- GenAI-specific = OWASP Top 10 LLM + OWASP checklist + NIST AI 600-1 + NIST AI RMF
- Clarify follows **ISO 42001** guidance for bias/explainability

### AWS governance & compliance services; multi-account governance
- **AWS Config** → detailed view + **history** of resource configurations and relationships → misconfig detection, audits, change impact; **Config rules** for policy compliance
- **AWS Audit Manager** → continuous auditing, **automates evidence collection** vs controls, multi-cloud/hybrid evidence, stakeholder reviews; prebuilt **AWS Generative AI Best Practices Framework v2** for Bedrock + SageMaker AI
- **AWS Artifact** → **on-demand download of AWS compliance reports** (ISO, PCI, SOC) for auditors/regulators; also ISV (Marketplace) due diligence
- **AWS CloudTrail** → records **actions by users, roles, services** (console, CLI, SDK/API) → who did what, to which resource, when, from which IP; audit-worthy event storage, **SQL analysis** of logs; no trail → **Event history** (recent only); does **not** capture inference content (prompts/responses → model invocation logging, see [13 · Cost, latency & monitoring](13-cost-latency-monitoring.md))
  - Watch **guardrail config changes** (bypass attempts, e.g. who removed a guardrail); key events: InvokeModel / InvokeEndpoint / Lambda Invoke · CreateModel / UpdateEndpoint · CreateUser / AttachRolePolicy / AssumeRole · S3 / DynamoDB Get/Put · also SageMaker console + API calls, plus non-API events for automatic model tuning
- **CloudWatch** → usage metrics + dashboards (pair: CloudWatch = metrics, CloudTrail = API activity)
- **AWS Trusted Advisor** → best-practice checks: **cost, performance, resilience, security, operational excellence, service limits**
- Practices: **CloudTrail + CloudWatch + Config** together; compliance with SOC, HIPAA, GDPR
- **GRC 3 phases**: **Define** (**Artifact**, Compliance Center, Well-Architected) → **Deploy & operate** (**Control Tower, Organizations, Config, Security Hub**) → **Measure & assess** (**Audit Manager, CloudTrail**)
- **Organizations** → IAM Identity Center, **SCPs** as max-permission boundaries, **RAM** to share subnets, org-wide CloudTrail + Backup
- **Control Tower** → set up/govern multi-account env (landing zone); preconfigured groups & permission sets, 3P IdPs (Okta, Entra ID), **preventive + detective guardrails**
- re:Invent 2025 governance launches: **Control Tower controls-only** (deploy **750+ managed controls** without a full landing zone) · Control Tower **auto-enroll** accounts · **Control Library** (all Config rules + Security Hub controls in the Control Catalog) · Organizations **direct account transfer** (**2 steps** invite → accept instead of 7; invites count against quota; **14 days** to accept; delegated admins de-registered first) · Organizations **transfer billing for Channel Partners**

### Data governance — strategies, concepts & data foundations
- **Data quality & integrity** → standards, validation/cleansing, **lineage & provenance**
- **Protection & privacy** → privacy policies, access control, encryption, **breach response** procedures
- **Lifecycle management** → classify/catalog by sensitivity, **retention & disposition**, backup & recovery
- **Responsible AI** → frameworks for bias/fairness/transparency/accountability, monitor & audit, train teams
- **Governance structures** → data governance council; **data stewards, owners, custodians**
- **Sharing & collaboration** → sharing agreements, **data virtualization/federation** (access without losing ownership)
- Concepts: **lifecycle** (collection → processing → storage → consumption → disposal/archive) · **logging** (inputs, outputs, perf metrics, system events) · **residency** (privacy regs, **sovereignty**, proximity to compute) · **monitoring** (quality, anomalies, **data drift** = input distribution changes) · **analysis** (stats, visualisation, **EDA**) · **retention** (regulation, retraining history, storage cost)
- 3 data-foundation pillars → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (Data foundations)
- **Data quality metrics**: **completeness, accuracy, timeliness (currency), consistency** + validation checks, profiling, feedback loop, lineage
- Secure data-engineering lifecycle (automation + access control around everything): **collection** (Kinesis, DMS, Glue) → **prep/cleaning** (**EMR / Glue**, most time-consuming) → **quality checks** (**Glue DataBrew, Glue Data Quality**) → visualisation/analysis (QuickSight, Neptune) → **IaC** (CloudFormation) · continuous monitoring (**CloudWatch**) · pipelines via **Glue workflows**
- GenAI data governance: govern which data trains/infers; fine-tuned model inherits the classification of its tuning data (scope detail → See [11 · Security](11-security-identity-agent-security.md))

### Implementing governance — policies, reviews, transparency, monitoring
- **Policies** → data mgmt, training, output validation, safety, human oversight, IP, bias, privacy; review regularly
- **Review cadence** → technical + legal + responsible-AI reviews; monthly/quarterly/bi-annual by risk; diverse stakeholders
- **Review strategies** → technical (performance, data quality, robustness) + non-technical (policy, RAI, regulation); validate outputs **before deployment**; intervention decision framework
- **Transparency standards** → publish model/training-data/decision info, document capabilities/limitations/intended use, feedback channels (e.g. **AI Service Cards**)
- **Team training** → bias mitigation, RAI, cross-functional sharing, ongoing certification
- **Monitoring** → performance (**accuracy, precision, recall, F1, latency**), infrastructure (CPU/GPU/mem, network, storage, logs), **bias & fairness**, compliance & RAI
- Precision = TP / predicted positives · Recall = TP / actual positives · **F1 = harmonic mean**
- Governance strategies = **clear GenAI policies** + **high transparency standards** (infra monitoring / security controls are not governance strategies)

### Data & model lineage, provenance & Model Cards
- **Source citation** → attribute training data sources (datasets, DBs) + **licences/terms/permissions**
- **Documenting data origins** → provenance: collection process, curation/cleaning, preprocessing/transforms → reveals bias/limitations
- Tools: **data lineage** (origin → transforms → movement) · **cataloging** (datasets, models, sources, licences, metadata) · **model cards**
- **SageMaker Model Cards** → intended use, **risk rating**, training details & metrics, evaluation results, recommendations → governance, audit support (required with data provenance for Scope 5)

### Responsible AI dimensions; RAI vs securing GenAI
- **8 responsible AI dimensions**: **Fairness · Explainability · Privacy & security · Safety · Controllability · Veracity & robustness · Governance · Transparency**
  - Transparency → **AI Service Cards** · Controllability → detect/remove hallucinations · Robustness → correct even under adversarial inputs
  - **Scalability is NOT a dimension** (classic distractor)
- **Securing GenAI** = subset of responsible AI → goal = **CIA** (threats + techniques → See [11 · Security](11-security-identity-agent-security.md) (AI-specific threats))
- Differences: **scope** (RAI broader) · **objective** (RAI = benefit society; security = protect the system)
- 3 aspects of GenAI security:
  - **Security OF GenAI** → protect models, training data, inference (poisoning, extraction)
  - **GenAI FOR security** → use GenAI for security work (synthetic data, malware detection, IR automation)
  - **Security FROM GenAI threats** → defend against misuse (deepfakes, AI phishing, malicious code)
- GenAI challenges beyond security: hallucinations · IP · data privacy · bias/fairness · interpretability · ethics; creativity use needs **authenticity / content labelling** of AI vs human work

### Ethics in prompt engineering
- AI ethics fundamentals: **fairness** · **transparency & explainability** · **privacy & data protection** · **accountability & governance**
- Prompt-eng risks: **manipulation/persuasion**, **amplifying biases**, **misrepresentation** (seeming more confident/human than it is), **power dynamics/access**
- Guidelines: **safety by design** (ethics from the start, test for bias/harm) · **transparency** (document choices & limits) · **monitoring** (feedback loops, update prompts) · **safeguards** for high-stakes domains · **collaboration** with SMEs, communities, ethicists
- Domain prompts need **regulatory & ethical awareness** (remind of limits, human oversight) — healthcare (**HIPAA**, confidentiality), finance (**SEC, Basel**), legal (jurisdiction)
- Case study: resume tool that subtly exaggerates skills = **deception**

### Bias & fairness — sources, mitigation, Clarify
- Bias enters via **biased prompts** (assumptions) or **biased training data** (even with neutral prompts); KB biases also flow into RAG output
- **Insufficient data → low model confidence** → toxicity filters/ranking prefer confident outputs → minority groups excluded → **cycle of bias**
- **3 mitigation techniques**:
  - **Update the prompt** — explicit guidance, few-shot clarification, **TIED** (Text-to-Image Disambiguation: model asks clarifying questions to remove ambiguity)
  - **Enhance the dataset** — diverse pronouns/examples, **counterfactual data augmentation** (swap attributes); images: **Detect → Segment → Augment**
  - **Training techniques** — **equalized odds** (match **TPR and FPR** across groups), **fairness as a model objective** (alongside energy efficiency, inference time), red teaming, RLHF, fair loss functions
- Bias = unfair predictions (e.g. recruitment favouring a gender) · explainability = **why** (loan denied: income 40%, credit 35%, tenure 25%) · drivers: **GDPR/FCRA**, risk, trust
- **SageMaker Clarify** → bias in data prep **without code** (pick features like age/gender) + visual report · **pre-training bias** (data) · **post-training bias** (algorithm/hyperparams) · **Shapley values** (feature impact) · **PDPs** (prediction change as one feature varies); processing job reads S3 · bias tool for Scope 5
- **Data Wrangler** → rebalance: **random undersampling, random oversampling, SMOTE**
- Fairness is a **process** (stakeholder consensus) and **by design** across the whole ML lifecycle (ongoing bias monitoring → See [14 · Evaluation & testing](14-evaluation-testing-troubleshooting.md))

## Exam triggers
- "Download SOC/ISO/PCI reports for auditor / Marketplace ISV due diligence" → **AWS Artifact**
- "Automate audit evidence collection" → **Audit Manager** · "Audit-ready GenAI evidence" → **Audit Manager GenAI Best Practices Framework v2**
- "Who called the Bedrock API / who changed X / who changed or removed a guardrail" → **CloudTrail** · "How was this resource configured last month" → **Config**
- "Document intended use + risk rating / model provenance" → **SageMaker Model Cards**
- "Input data distribution changes over time" → **data drift**
- "Data quality checks in pipeline" → **Glue Data Quality / DataBrew**
- "Non-negotiable AI law" → **EU AI Act** (regulation) · "AI management system standard" → **ISO 42001** · "GenAI risk profile" → **NIST AI 600-1**
- "Multi-account landing zone with guardrails" → **Control Tower** · "Restrict max permissions across accounts" → **SCPs**
- "Detect bias / explain prediction" → **SageMaker Clarify** · "Rebalance dataset" → **Data Wrangler (SMOTE, over/undersampling)**
- "Ambiguous text-to-image prompt yields biased images" → **TIED** · "Equal error rates across groups" → **equalized odds** · "Expand dataset by swapping attributes" → **counterfactual data augmentation**

## Flashcards
<details>
<summary><b>Q:</b> Place in the AI governance stack: EU AI Act, ISO 42001, NIST AI RMF / NIST AI 600-1, MITRE ATLAS.</summary>

EU AI Act = regulation (non-negotiable); ISO 42001 = compliance standard (AI management system); NIST AI RMF / AI 600-1 = security & risk management frameworks; MITRE ATLAS = threat mitigation guidance.

*D3 · AI regulation stack*
</details>

<details>
<summary><b>Q:</b> Which security considerations are specific to AI rather than generic IT?</summary>

Fake content generation, prompt injection and model weaknesses (not phishing, DDoS or ransomware).

*D3 · AI-specific threats*
</details>

<details>
<summary><b>Q:</b> A company wants continuous auditing that automatically collects evidence against controls, including hybrid/multi-cloud. Which service?</summary>

AWS Audit Manager.

*D3 · Audit evidence automation*
</details>

<details>
<summary><b>Q:</b> A team needs a multi-account security baseline that includes Bedrock. What reference?</summary>

AWS Security Reference Architecture (SRA), which has a GenAI section focused on Bedrock.

*D3 · AWS SRA*
</details>

<details>
<summary><b>Q:</b> An auditor needs AWS SOC, ISO and PCI reports, and the team must do due diligence on Marketplace ISVs. Which service?</summary>

AWS Artifact - on-demand download of AWS (and ISV) compliance reports.

*D3 · Compliance reports download*
</details>

<details>
<summary><b>Q:</b> Distinguish 'who called the Bedrock API' from 'how was this resource configured last month'.</summary>

Who called an API -> AWS CloudTrail (actions by users, roles, services). Configuration history and relationships -> AWS Config.

*D3 · Config vs CloudTrail*
</details>

<details>
<summary><b>Q:</b> Which dataset technique expands training data by swapping attributes (e.g. pronouns) to reduce bias?</summary>

Counterfactual data augmentation (for images: Detect → Segment → Augment).

*D3 · Counterfactual augmentation*
</details>

<details>
<summary><b>Q:</b> Which training-time fairness technique matches true positive and false positive rates across groups?</summary>

Equalized odds.

*D3 · Equalized odds*
</details>

<details>
<summary><b>Q:</b> A GenAI solution is for a US federal information system requiring formal CIA evaluation. Which standard applies?</summary>

NIST (NIST 800-53).

*D3 · Federal compliance*
</details>

<details>
<summary><b>Q:</b> What data classification does a fine-tuned model receive?</summary>

It inherits the classification of its fine-tuning data - use the highest level present.

*D3 · Fine-tuned classification*
</details>

<details>
<summary><b>Q:</b> Name governance practices for GenAI apps.</summary>

Manage/audit FM access, monitor and log access, log requests + responses for compliance/explainability, periodically audit with test data and simulated prompt injection, document processes, implement guardrails.

*D3 · GenAI governance practices*
</details>

<details>
<summary><b>Q:</b> Which services provide audit-ready GenAI compliance evidence and document a model's intended use and risk rating?</summary>

AWS Audit Manager with the AWS Generative AI Best Practices Framework v2; SageMaker Model Cards.

*D3 · Governance evidence*
</details>

<details>
<summary><b>Q:</b> Which of these are governance strategies: clear GenAI policies, transparency standards, infrastructure monitoring, security controls?</summary>

Clear GenAI policies and high transparency standards.

*D3 · Governance strategies*
</details>

<details>
<summary><b>Q:</b> How should the Scoping Matrix be integrated with existing security policies?</summary>

Review existing policies and add AI considerations (do not replace them); reference the matrix and define governance roles.

*D3 · Policy integration*
</details>

<details>
<summary><b>Q:</b> Which is NOT a responsible AI dimension: fairness, explainability, controllability, scalability, veracity &amp; robustness?</summary>

Scalability. The 8 dimensions are fairness, explainability, privacy & security, safety, controllability, veracity & robustness, governance, transparency.

*D3 · RAI dimensions*
</details>

<details>
<summary><b>Q:</b> Employees use a public GenAI chat app for brainstorming. What controls fit?</summary>

Web proxy/CASB to restrict traffic, DLP policies, acceptable use policy, awareness training, no PII/confidential IP in prompts, re-review ToS regularly. Direct model modification is not possible.

*D3 · Scope 1 controls*
</details>

<details>
<summary><b>Q:</b> What distinguishes Scope 2 (enterprise app) from Scope 1 (consumer app)?</summary>

An enterprise agreement/contract (business relationship) with the vendor; data flow is the same and Scope 1 controls still apply.

*D3 · Scope 1 vs 2*
</details>

<details>
<summary><b>Q:</b> A company adopts a SaaS app with embedded GenAI under contract (e.g. Amazon Q Business). Name key controls.</summary>

Opt out of training on your data, legal review of EULA/indemnification, check SOC 2/ISO 27001, vendor secure SDLC, RBAC; for Amazon Q Business use IAM Identity Center and VPC endpoints.

*D3 · Scope 2 controls*
</details>

<details>
<summary><b>Q:</b> A hospital builds an EHR summariser on a Bedrock FM. What legal/privacy step comes before querying the model?</summary>

Anonymise sensitive data so no PII is sent; add Guardrails denied topics and content filter thresholds.

*D3 · Scope 3 legal*
</details>

<details>
<summary><b>Q:</b> What resilience measures apply to a Scope 3 app calling an FM API?</summary>

Error handling and fallbacks, awareness of API availability/limits, multi-Region for HA, longer client timeouts for long completions, retain prompt/completion logs.

*D2 · Scope 3 resilience*
</details>

<details>
<summary><b>Q:</b> Name risks specific to fine-tuning (Scope 4).</summary>

Overfitting, bias, poisoning of fine-tuning data, drift; also check that the base-model licence permits fine-tuning.

*D3 · Scope 4 risks*
</details>

<details>
<summary><b>Q:</b> What extra controls apply when training a model from scratch (Scope 5)?</summary>

Verify rights to all training data, own the EULA, VPC-isolated training/inference, end-to-end encryption, model cards + data provenance, differential privacy, checkpoints, secure decommissioning.

*D3 · Scope 5 controls*
</details>

<details>
<summary><b>Q:</b> Classify: staff using ChatGPT/PartyRock; Amazon Q; app calling an FM on Bedrock; fine-tuning on SageMaker JumpStart; training from scratch on SageMaker AI.</summary>

Scope 1 Consumer; Scope 2 Enterprise; Scope 3 Pre-trained; Scope 4 Fine-tuned; Scope 5 Self-trained. Scopes 1-2 = Buy, 3-5 = Build.

*D3 · Scope examples*
</details>

<details>
<summary><b>Q:</b> In the GenAI Security Scoping Matrix, what are scopes 1-5?</summary>

1 consumer app, 2 enterprise app, 3 pre-trained models (Bedrock/JumpStart), 4 fine-tuned models, 5 self-trained models; 1-2 are prebuilt, 3-5 self-built.

*D3 · Scoping Matrix*
</details>

<details>
<summary><b>Q:</b> In the Generative AI Security Scoping Matrix, in which scope does the customer control fine-tuning data, and in which the training data?</summary>

Fine-tuning data -> Scope 4 (fine-tuned models). Training data -> only Scope 5 (self-trained). User data is always the customer's.

*D3 · Scoping Matrix data*
</details>

<details>
<summary><b>Q:</b> Which two measures secure training data (as opposed to improving data quality)?</summary>

Controlling access and ensuring integrity. Cleaning/inconsistency checks are data quality, not security.

*D3 · Securing training data*
</details>

<details>
<summary><b>Q:</b> Classify: safeguards against deepfake creation; automating malware detection with GenAI; protecting model weights from extraction.</summary>

Security FROM GenAI threats; GenAI FOR security; Security OF GenAI (goal = CIA of models and data).

*D3 · Security OF/FOR/FROM*
</details>

<details>
<summary><b>Q:</b> A fine-tuned model trained on proprietary data needs training-pipeline protection and model artifact protection. Which security scope applies?</summary>

Scope 4 (fine-tuned models): training data security, model artifact protection, custom guardrails. Scope 3 = pre-trained models (IAM, VPC endpoints, encryption).

*D3 · Security scoping*
</details>

<details>
<summary><b>Q:</b> What should be the starting point when planning a security strategy for a new AI solution?</summary>

The Shared Responsibility Model (AWS = security OF the cloud; customer = security IN the cloud) plus the required security services.

*D3 · Shared Responsibility start*
</details>

<details>
<summary><b>Q:</b> Ambiguous text-to-image prompts produce biased images. Which prompt-level mitigation has the model ask clarifying questions?</summary>

The TIED framework (Text-to-Image Disambiguation).

*D3 · TIED framework*
</details>

<details>
<summary><b>Q:</b> A customer asks for their PII to be removed from a fine-tuned model. What is the reality and prevention?</summary>

Records cannot be selectively deleted; removal requires a full retrain. Anonymise data before fine-tuning and never tune on PII.

*D3 · Unlearning PII*
</details>

---
[← Back to index](README.md)
