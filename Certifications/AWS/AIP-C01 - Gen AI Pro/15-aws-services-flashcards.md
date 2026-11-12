[← Back to index](README.md)

# ☁️ 15 · AWS Services Flashcards

> One card per AWS service (189 services): its GenAI relevance and the scenario cue that should make you pick it on the AIP-C01 exam. Click a service to reveal.

## 🆕 Additional in-scope services (from the exam guide)

Services from the official in-scope services list and the services named in the Domain 1–5 skill statements.

<details>
<summary><b>AWS AppConfig</b> 🆕</summary>

**GenAI relevance:** Managed dynamic configuration (feature flags, config profiles) deployed to running apps with validation and rollback. In GenAI it holds the current model ID, prompt version or routing rules so you can switch FMs/providers without a code change or redeploy.

**Pick it when:** dynamic model selection / provider switching without code modifications (Skill 1.2.2: Lambda + API Gateway + AppConfig)

*Exam guide skill 1.2.2*
</details>

<details>
<summary><b>Amazon AppFlow</b> 🆕</summary>

**GenAI relevance:** No-code managed data transfer between SaaS apps (Salesforce, ServiceNow, Zendesk, Slack, SAP…) and AWS (S3, Redshift). Used to pull enterprise SaaS data into S3 for Knowledge Base ingestion or to sync FM outputs back.

**Pick it when:** move SaaS/CRM data into S3 for RAG with no code; scheduled or event-triggered SaaS sync

*Exam guide skill 1.4.4, 2.3.1*
</details>

<details>
<summary><b>Amazon MSK (Managed Streaming for Apache Kafka)</b> 🆕</summary>

**GenAI relevance:** Managed Apache Kafka. Streams real-time events (change data, clickstreams, logs) into GenAI pipelines, e.g. near-real-time embedding updates for vector stores or event-driven agent triggers where Kafka is already the enterprise standard.

**Pick it when:** existing Kafka; real-time change detection feeding vector store refresh

*Exam guide skill 1.4.5, 2.3.1*
</details>

<details>
<summary><b>AWS Lambda@Edge</b> 🆕</summary>

**GenAI relevance:** Runs Lambda functions at CloudFront edge locations on viewer/origin requests and responses. GenAI uses: edge caching of deterministic responses, request hashing/normalisation, auth or header checks before requests reach the FM backend.

**Pick it when:** edge caching, low-latency request preprocessing close to users

*Exam guide skill 4.1.4*
</details>

<details>
<summary><b>AWS App Runner</b> 🆕</summary>

**GenAI relevance:** Fully managed service that runs a web app or API from a container image or source repo with built-in load balancing, TLS and autoscaling. Quick way to host a GenAI front end / API that calls Bedrock without managing ECS or EKS.

**Pick it when:** deploy containerised GenAI web app/API with least operational overhead

*Exam guide skill 2.2*
</details>

<details>
<summary><b>AWS Outposts</b> 🆕</summary>

**GenAI relevance:** AWS infrastructure and services running in your own data centre. Used when data must stay on-premises for residency/compliance while still integrating with cloud FMs (hybrid architectures).

**Pick it when:** on-premises data residency; data must not leave the facility; hybrid cloud

*Exam guide skill 2.3.4*
</details>

<details>
<summary><b>AWS Wavelength</b> 🆕</summary>

**GenAI relevance:** AWS compute and storage embedded in telecom 5G networks for single-digit-ms latency to mobile devices. GenAI edge deployments of small models / inference close to mobile users.

**Pick it when:** ultra-low latency to 5G mobile devices; edge deployment

*Exam guide skill 2.3.4*
</details>

<details>
<summary><b>AWS Auto Scaling</b> 🆕</summary>

**GenAI relevance:** Scales EC2, ECS, DynamoDB and SageMaker endpoint capacity on target metrics. For GenAI: auto-scaling self-hosted model endpoints/containers on GenAI-specific metrics (concurrency, tokens, GPU utilisation) for bursty traffic.

**Pick it when:** auto-scaling configurations optimised for GenAI traffic patterns

*Exam guide skill 4.1.3, 4.2.5*
</details>

<details>
<summary><b>Elastic Load Balancing (ELB)</b> 🆕</summary>

**GenAI relevance:** Distributes traffic across targets (ALB for HTTP/WebSocket, NLB for TCP). Fronts self-hosted model servers or MCP servers on ECS/EC2; ALB supports long-lived connections for streaming.

**Pick it when:** load balance self-hosted inference containers / MCP servers

*Exam guide skill 2.2, 2.1.7*
</details>

<details>
<summary><b>AWS Global Accelerator</b> 🆕</summary>

**GenAI relevance:** Static anycast IPs routing users over the AWS global network to the nearest healthy Regional endpoint. Improves latency and gives fast multi-Region failover for GenAI APIs.

**Pick it when:** multi-Region GenAI API failover with static IPs; global latency

*Exam guide skill 1.2.3*
</details>

<details>
<summary><b>Amazon DynamoDB Streams</b> 🆕</summary>

**GenAI relevance:** Ordered change log of DynamoDB item changes, consumed by Lambda. Triggers incremental re-embedding / vector store sync when source records change, or downstream agent workflows.

**Pick it when:** real-time change detection → incremental vector update

*Exam guide skill 1.4.5*
</details>

<details>
<summary><b>AWS Amplify</b> 🆕</summary>

**GenAI relevance:** Framework + hosting for full-stack web/mobile apps, including Amplify UI components (and AI Kit for chat/conversation UIs). Declarative UI to put GenAI features in front of users fast.

**Pick it when:** accessible AI interfaces; declarative UI components (Skill 2.5.2)

*Exam guide skill 2.5.2*
</details>

<details>
<summary><b>AWS CDK</b> 🆕</summary>

**GenAI relevance:** Infrastructure as code in TypeScript/Python etc. that synthesises CloudFormation. Provision Bedrock KBs, agents, Lambda tools and pipelines repeatably across environments.

**Pick it when:** IaC for GenAI stacks; repeatable environments

*Exam guide skill Recommended knowledge: IaC*
</details>

<details>
<summary><b>AWS CodePipeline</b> 🆕</summary>

**GenAI relevance:** Managed CI/CD orchestration. Builds GenAI delivery pipelines: prompt/agent/model changes → build → automated evaluation & security scans → staged deploy with rollback.

**Pick it when:** CI/CD for GenAI components with quality gates and rollback (Skill 2.3.5)

*Exam guide skill 2.3.5, 1.2.4*
</details>

<details>
<summary><b>AWS CodeBuild</b> 🆕</summary>

**GenAI relevance:** Managed build/test runner. In GenAI pipelines it runs unit tests, prompt regression suites, Bedrock/AgentCore evaluation jobs and security scans as pipeline stages.

**Pick it when:** automated testing stage; run evaluation as a CI quality gate

*Exam guide skill 2.3.5, 5.1.4*
</details>

<details>
<summary><b>AWS CodeDeploy</b> 🆕</summary>

**GenAI relevance:** Automates deployments to EC2, ECS and Lambda with canary/linear/blue-green strategies and automatic rollback on alarms. Safe rollout of new model versions or GenAI app code.

**Pick it when:** canary / blue-green deployment; rollback strategies for failed deployments

*Exam guide skill 1.2.4, 5.1.9*
</details>

<details>
<summary><b>AWS CodeArtifact</b> 🆕</summary>

**GenAI relevance:** Managed private package repository (pip, npm, Maven). Hosts vetted versions of GenAI SDKs/frameworks (LangChain, Strands) and internal libraries for secure supply chain.

**Pick it when:** private package repo; approved library versions in CI/CD

*Exam guide skill 2.3.5*
</details>

<details>
<summary><b>Amazon Q Developer</b> 🆕</summary>

**GenAI relevance:** Generative AI coding assistant: code generation/refactoring, tests, explanations and troubleshooting in IDE/CLI/console. Exam guide names it for developer productivity and GenAI-specific error pattern recognition.

**Pick it when:** generate/refactor code; AI component testing; troubleshoot GenAI errors (Skills 2.5.4, 2.5.6)

*Exam guide skill 2.5.4, 2.5.6*
</details>

<details>
<summary><b>Amazon Augmented AI (A2I)</b> 🆕</summary>

**GenAI relevance:** Human review workflows for ML predictions: send low-confidence or sampled outputs to private workforce, vendors or Mechanical Turk. Human-in-the-loop review of FM outputs and annotation of response quality.

**Pick it when:** human review of low-confidence outputs; annotation workflows; human feedback

*Exam guide skill 5.1.3, 2.1.5*
</details>

<details>
<summary><b>Amazon SageMaker Ground Truth</b> 🆕</summary>

**GenAI relevance:** Data labelling service with human workforces (incl. RLHF-style preference ranking). Builds labelled/golden datasets for fine-tuning and evaluation.

**Pick it when:** label data / create golden dataset / human preference data

*Exam guide skill 5.1.3, 5.1.5*
</details>

<details>
<summary><b>Amazon SageMaker Model Registry</b> 🆕</summary>

**GenAI relevance:** Catalog of model versions with approval status, metadata and lineage; integrates with CI/CD for deploy and rollback. Version and govern fine-tuned/custom FMs.

**Pick it when:** version custom models, approve, deploy, rollback (Skill 1.2.4)

*Exam guide skill 1.2.4*
</details>

<details>
<summary><b>Amazon SageMaker Neo</b> 🆕</summary>

**GenAI relevance:** Compiles/optimises trained models for a specific target hardware (cloud instances or edge devices) for faster, smaller inference.

**Pick it when:** optimise model for target hardware / edge

*Exam guide skill 2.2.2*
</details>

<details>
<summary><b>Amazon SageMaker Processing</b> 🆕</summary>

**GenAI relevance:** Managed jobs that run data pre/post-processing and evaluation scripts on scalable compute. Used for multimodal data preparation and batch embedding/evaluation pipelines.

**Pick it when:** data processing workflows for text/image/audio/tabular (Skill 1.3.2)

*Exam guide skill 1.3.2*
</details>

<details>
<summary><b>Amazon Q Business Apps</b> 🆕</summary>

**GenAI relevance:** Lightweight GenAI apps that business users create from natural-language descriptions inside Amazon Q Business, grounded in company data.

**Pick it when:** non-developers build simple GenAI apps on enterprise data

*Exam guide skill In-scope list*
</details>

<details>
<summary><b>AWS Chatbot (Amazon Q Developer in chat applications)</b> 🆕</summary>

**GenAI relevance:** Delivers CloudWatch alarms and AWS notifications into Slack/Teams/Chime and lets teams run commands from chat. ChatOps for GenAI incidents (throttling, guardrail spikes, cost anomalies).

**Pick it when:** send alarms to Slack/Teams; ChatOps

*Exam guide skill 4.3*
</details>

<details>
<summary><b>Amazon CloudWatch Synthetics</b> 🆕</summary>

**GenAI relevance:** Canaries that run scripted user journeys against endpoints on a schedule. Synthetic user workflows to validate GenAI app behaviour/latency during and after FM updates.

**Pick it when:** synthetic user workflows; deployment validation (Skill 5.1.9)

*Exam guide skill 5.1.9*
</details>

<details>
<summary><b>AWS Cost Anomaly Detection</b> 🆕</summary>

**GenAI relevance:** ML-based monitoring of spend that alerts on unusual cost spikes per service/account/tag. Catches runaway token usage or agent loops.

**Pick it when:** cost anomaly detection; unexpected Bedrock spend spike (Skill 4.3.2)

*Exam guide skill 4.3.2*
</details>

<details>
<summary><b>Amazon Managed Grafana</b> 🆕</summary>

**GenAI relevance:** Managed Grafana dashboards over CloudWatch, Prometheus, X-Ray etc. Business-impact and operational GenAI dashboards for stakeholders.

**Pick it when:** custom dashboards / visualisation for stakeholders (Skills 4.3.1, 5.1.8)

*Exam guide skill 4.3.1, 5.1.8*
</details>

<details>
<summary><b>AWS Service Catalog</b> 🆕</summary>

**GenAI relevance:** Curated portfolio of approved, pre-configured IaC products teams can self-provision. Standardised, governed GenAI building blocks (approved KB/agent/guardrail templates).

**Pick it when:** standardised approved components; self-service with governance (Skill 1.1.3)

*Exam guide skill 1.1.3*
</details>

<details>
<summary><b>AWS Systems Manager</b> 🆕</summary>

**GenAI relevance:** Operations hub: Parameter Store, Session Manager, Run Command, patching, automation runbooks. Parameter Store can hold model IDs/prompt config; runbooks automate remediation.

**Pick it when:** config parameters; automated remediation runbooks

*Exam guide skill 3.3.4*
</details>

<details>
<summary><b>AWS Well-Architected Tool (Generative AI Lens)</b> 🆕</summary>

**GenAI relevance:** Reviews workloads against Well-Architected pillars; the Generative AI Lens adds GenAI-specific best practices. Exam guide names it for standardised technical components.

**Pick it when:** standardised architecture review; WA Tool GenAI Lens (Skill 1.1.3)

*Exam guide skill 1.1.3*
</details>

<details>
<summary><b>AWS DataSync</b> 🆕</summary>

**GenAI relevance:** Online data transfer from on-premises NFS/SMB/HDFS or other clouds to S3/EFS/FSx, scheduled and incremental. Moves on-prem document stores into S3 for KB ingestion.

**Pick it when:** migrate/sync on-premises file shares to S3 for RAG

*Exam guide skill 1.4.4*
</details>

<details>
<summary><b>AWS Transfer Family</b> 🆕</summary>

**GenAI relevance:** Managed SFTP/FTPS/FTP/AS2 endpoints backed by S3/EFS. Lets partners/legacy systems drop documents into S3 that then flow into GenAI processing.

**Pick it when:** partners send files via SFTP into the GenAI pipeline

*Exam guide skill 2.3.1*
</details>

<details>
<summary><b>AWS Encryption SDK</b> 🆕</summary>

**GenAI relevance:** Client-side encryption library (envelope encryption with KMS keys). Encrypt sensitive prompt/response data or documents before storing or sending.

**Pick it when:** client-side / application-level encryption of sensitive data

*Exam guide skill 3.2*
</details>

<details>
<summary><b>AWS Lake Formation</b> 🆕</summary>

**GenAI relevance:** Fine-grained (table/column/row/cell) access control and governance over the data lake catalog. Restricts what data GenAI pipelines and users can read.

**Pick it when:** granular data access for FM data (Skill 3.2.1)

*Exam guide skill 3.2.1*
</details>

<details>
<summary><b>AWS Glue Data Catalog & data lineage</b> 🆕</summary>

**GenAI relevance:** Central metadata catalog to register data sources; Glue tracks lineage. Exam guide uses it for data source tracking/traceability and compliance.

**Pick it when:** register data sources; automatic data lineage; source attribution (Skills 3.3.1, 3.3.2)

*Exam guide skill 3.3.1, 3.3.2*
</details>

<details>
<summary><b>Amazon S3 storage features (Lifecycle, Intelligent-Tiering, Cross-Region Replication)</b> 🆕</summary>

**GenAI relevance:** Lifecycle policies expire/transition objects → enforce retention of prompts, logs and training data (Skill 3.2.2). Intelligent-Tiering auto-moves infrequently used documents/logs to cheaper tiers. CRR replicates KB source data/artifacts to another Region for resilience or residency.

**Pick it when:** data retention policy → Lifecycle; unknown access pattern cost → Intelligent-Tiering; cross-Region copy → CRR

*Exam guide skill 3.2.2*
</details>

<details>
<summary><b>Amazon EBS & Amazon EFS</b> 🆕</summary>

**GenAI relevance:** EBS = block volumes for EC2 (model weights, local vector indexes on self-hosted inference). EFS = shared NFS file system across containers/Lambda (shared model files, documents).

**Pick it when:** self-hosted model storage; shared file system for containers/Lambda

*Exam guide skill 2.2.2*
</details>

## ⚠️ Listed as out of scope in the exam guide

The exam guide lists **Amazon MQ, Amazon Redshift, AWS Batch, AWS Budgets, AWS Direct Connect and AWS Trusted Advisor** as out of scope. They still have cards below (tagged), but deprioritise them.

## Bedrock & FMs

<details>
<summary><b>Amazon Bedrock</b></summary>

**GenAI relevance:** Fully managed, serverless access to many FMs through one API with private customisation and enterprise security; core Layer 2 build tool. Managed access to FMs from AWS and third parties (Nova, Claude, Llama, Mistral, Cohere, AI21) plus custom imported models, with IAM, VPC and CloudTrail integration.

**Pick it when:** managed FMs, single API, swap models, no infrastructure; managed FM access, serverless model inference, no infrastructure
</details>

<details>
<summary><b>Amazon Bedrock Converse & InvokeModel APIs</b></summary>

**GenAI relevance:** Unified request format (Converse/ConverseStream) across models with inferenceConfig and system prompts. Runs inference with model-specific request bodies for text, image and embedding models; InvokeModelWithResponseStream streams output.

**Pick it when:** write once, swap models without rewriting code; model-specific body, embeddings call, streaming chunks
</details>

<details>
<summary><b>Amazon Bedrock cross-Region inference</b></summary>

**GenAI relevance:** Inference profiles route requests across Regions for higher throughput and resilience with no code changes. Inference profiles route an FM across Regions for throughput; required ID for latency-optimized inference; application inference profiles + tags give per-app cost tracking; cross-Region inference handles bursts.

**Pick it when:** throttling, regional capacity, inference profile ARN; traffic bursts, cost by team/app, us. prefixed model ID
</details>

<details>
<summary><b>Amazon Bedrock Intelligent Prompt Routing</b></summary>

**GenAI relevance:** Cost lever routing prompts to suitable models. Routes each request between two models of the same family via quality estimation; up to 30% cost reduction; router ARN as modelId; fallback model; trace shows chosen model.

**Pick it when:** route requests between models to save cost; mix of simple and complex prompts, cut cost without custom routing
</details>

<details>
<summary><b>Amazon Bedrock latency-optimized inference</b></summary>

**GenAI relevance:** performanceConfig latency: optimized lowers TTFT and raises OTPS for the same model/quality via inference profile ID; falls back to standard at standard price when quota exceeded.

**Pick it when:** faster responses same model, real-time chat/autocomplete
</details>

<details>
<summary><b>Amazon Bedrock Marketplace</b></summary>

**GenAI relevance:** 100+ specialised FMs accessible through the same Bedrock APIs and security controls.

**Pick it when:** niche/industry model not in core catalog
</details>

<details>
<summary><b>Amazon Bedrock model customization</b></summary>

**GenAI relevance:** Trains a private copy of a base model via labelled fine-tuning, unlabelled continued pretraining, or RFT with RLVR/RLAIF rewards. Fine-tuning FMs with customer data (Scope 4); model inherits data classification; artifacts encrypted with KMS.

**Pick it when:** domain terminology, consistent format, reward function without labelled data; fine-tune Claude on Bedrock, Scope 4
</details>

<details>
<summary><b>Amazon Bedrock Model Distillation</b></summary>

**GenAI relevance:** Teacher model generates up to 15,000 prompt-response pairs to fine-tune a smaller same-provider student; deployed on Provisioned Throughput. Cost lever creating smaller, cheaper models.

**Pick it when:** big model too slow/expensive at high volume, keep accuracy; reduce inference cost with smaller model
</details>

<details>
<summary><b>Amazon Bedrock prompt caching</b></summary>

**GenAI relevance:** Caches repeated system prompts/context for up to 90% cost reduction and lower latency. Cost lever caching repeated prompt prefixes.

**Pick it when:** same system prompt/context reused across many requests; repeated long prompts, cut cost/latency
</details>

<details>
<summary><b>Amazon Bedrock Prompt Management</b></summary>

**GenAI relevance:** Central catalog to create, test, version (immutable) and compare prompts with variables; invoked by prompt ARN; works with Prompt Flows. Stores reusable versioned prompts that Flows Prompt nodes can reference.

**Pick it when:** version/reuse prompt templates, compare variants; reuse versioned prompt templates
</details>

<details>
<summary><b>Amazon Bedrock Provisioned Throughput</b></summary>

**GenAI relevance:** Reserved Model Units for consistent performance; required for distilled and custom models; not purchasable for Legacy models. Reserved model units for guaranteed capacity; remedy for throttling; required for some custom models.

**Pick it when:** consistent performance, custom/distilled model hosting; throttling at steady high load, guaranteed capacity
</details>

<details>
<summary><b>Amazon Nova</b></summary>

**GenAI relevance:** Amazon FM family: Micro, Lite, Pro, Premier, Nova 2 Lite (understanding), Canvas/Reel (creative), Sonic (speech-to-speech), Act (browser actions), Multimodal Embeddings.

**Pick it when:** lowest latency text (Micro), balanced (Pro), reasoning/teacher (Premier), voice (Sonic)
</details>

<details>
<summary><b>Amazon Nova Act</b></summary>

**GenAI relevance:** Model trained to take actions in a web browser. Vertically integrated agent SDK in the AWS agentic portfolio.

**Pick it when:** agent automates browser actions; vertically integrated agent SDK
</details>

<details>
<summary><b>Amazon Nova Multimodal Embeddings</b></summary>

**GenAI relevance:** Single embedding model for text, documents, images, video and audio enabling cross-modal retrieval. Embeds text, image and video into one vector space for cross-modal search.

**Pick it when:** search across text/images/video/audio in one space; search images/video with text
</details>

<details>
<summary><b>Amazon Titan Embeddings (Text V2, Multimodal)</b></summary>

**GenAI relevance:** Text embedding model with 256/512/1024 dimensions, 8192-token max input; common KB embedding model. Embeds text and images into the same semantic space.

**Pick it when:** text embeddings for RAG, configurable dimensions; image search by text or image
</details>

<details>
<summary><b>Amazon Titan Text & Image Generator</b></summary>

**GenAI relevance:** Titan Text for summarise/generate/classify/QA; Titan Image Generator for text-to-image, inpainting, outpainting, background removal.

**Pick it when:** inpainting/outpainting, Amazon text model
</details>

<details>
<summary><b>AWS Marketplace</b></summary>

**GenAI relevance:** Source of third-party vector DBs such as Pinecone.

**Pick it when:** third-party vector DB (Pinecone)
</details>

<details>
<summary><b>Cohere Embed (v4, English, Multilingual)</b></summary>

**GenAI relevance:** KB embedding models; v4 is multimodal (text + images) with 256–1536 dims and int8/binary types.

**Pick it when:** multimodal/multilingual embeddings
</details>

## RAG & Vector stores

<details>
<summary><b>Amazon Aurora / RDS PostgreSQL (pgvector)</b></summary>

**GenAI relevance:** Stores vectors alongside relational data via pgvector; supported LangChain vector store. pgvector vector search on RDS PostgreSQL.

**Pick it when:** vectors in existing PostgreSQL; vectors in existing PostgreSQL on RDS
</details>

<details>
<summary><b>Amazon Bedrock Data Automation</b></summary>

**GenAI relevance:** Advanced KB parser for complex documents with tables, images and visuals. Listed as a Bedrock customisation/data capability in the agentic stack.

**Pick it when:** parse documents with tables/images for RAG; extract insights from unstructured content
</details>

<details>
<summary><b>Amazon Bedrock Knowledge Bases</b></summary>

**GenAI relevance:** Fully managed RAG (ingest, chunk, embed, store, retrieve) with GraphRAG, multimodal, metadata filtering, Retrieve and RetrieveAndGenerate APIs; up to 5 data sources. Fully managed RAG: ingestion (chunk, embed, store), retrieval and prompt augmentation with citations and built-in session context.

**Pick it when:** managed RAG, Q&A over company docs; managed RAG, minimal code, citations, multi-turn KB chat
</details>

<details>
<summary><b>Amazon Bedrock Rerank API / reranker models</b></summary>

**GenAI relevance:** Reorders retrieved chunks by relevance so fewer, better chunks reach the FM (text only).

**Pick it when:** relevant chunks poorly ordered, reduce chunks sent to FM
</details>

<details>
<summary><b>Amazon DocumentDB</b></summary>

**GenAI relevance:** Document database option for vector storage.

**Pick it when:** vectors in document DB
</details>

<details>
<summary><b>Amazon ElastiCache</b></summary>

**GenAI relevance:** In-memory cache with vector search capability. Redis cache for recommendations with event-triggered invalidation instead of TTL (Snoonu).

**Pick it when:** in-memory vector/cache; low-latency cache, event-driven invalidation
</details>

<details>
<summary><b>Amazon MemoryDB</b></summary>

**GenAI relevance:** In-memory durable database with vector search.

**Pick it when:** in-memory low-latency vector search
</details>

<details>
<summary><b>Amazon Neptune Analytics (GraphRAG)</b></summary>

**GenAI relevance:** Knowledge-graph retrieval over entities and relationships for multi-hop reasoning. Graph database/analytics with vector support (also Neptune ML) for graph-based retrieval.

**Pick it when:** multi-hop, root-cause analysis, compliance tracing, relationships; graph + vector search
</details>

<details>
<summary><b>Amazon OpenSearch Serverless</b></summary>

**GenAI relevance:** Vector engine for high-QPS, millisecond-latency and hybrid/keyword search RAG; LangChain OpenSearchVectorSearch. Default KB vector store (quick-create); billions of vectors, ms latency; required for KB hybrid search (filterable text field) and FAISS for metadata filters.

**Pick it when:** hybrid search, millisecond latency, high QPS; hybrid search in KB, no cluster management vector store
</details>

<details>
<summary><b>Amazon OpenSearch Service</b></summary>

**GenAI relevance:** Managed OpenSearch domains usable as vector DB. Vector search with HNSW/IVF, FAISS/Lucene, dense, sparse, hybrid and conversational search; neural search pipelines with ML connectors to Bedrock.

**Pick it when:** vector DB on managed OpenSearch cluster; k-NN, BM25 + vector hybrid, neural sparse, quantization
</details>

<details>
<summary><b>Amazon S3 Vectors</b></summary>

**GenAI relevance:** Native vector storage/query in S3 with up to 90% lower cost, subsecond latency, semantic-only search; KB vector store via Quick create. Low-cost KB-supported vector store for RAG.

**Pick it when:** lowest-cost vector store, archival but searchable, millions of docs; cheap vector storage for RAG
</details>

## Agents & AgentCore

<details>
<summary><b>Agent2Agent (A2A) protocol</b></summary>

**GenAI relevance:** Open protocol for agent-to-agent communication across frameworks/platforms.

**Pick it when:** agents from different frameworks talk to each other
</details>

<details>
<summary><b>Amazon Bedrock AgentCore</b></summary>

**GenAI relevance:** Modular platform to deploy and operate production agents with any framework and any model; consumption-based pricing. Modular, framework- and model-agnostic services to deploy and operate agents securely at scale; consumption pricing.

**Pick it when:** production agents at scale, any framework; production agents, any framework/model, migrate off Bedrock Agents
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Browser</b></summary>

**GenAI relevance:** Cloud browser for agents to use websites at scale, with CloudTrail logging and session replay. VM-isolated cloud browser with live view, session replay and CloudTrail logging.

**Pick it when:** agent browses web apps, session replay; web UI only, navigate websites, replay
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Code Interpreter</b></summary>

**GenAI relevance:** Runs agent-generated code in an isolated sandbox. Isolated sandbox running JS/TS/Python with S3 data access for calculations and analysis.

**Pick it when:** execute generated code safely; run model-generated code safely
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Evaluations</b></summary>

**GenAI relevance:** Continuous agent quality monitoring with 13 built-in evaluators (Preview). Built-in, custom or third-party evaluators run on-demand, batch or online; results feed Observability.

**Pick it when:** benchmark/evaluate agent quality continuously; measure agent quality pre- and post-deploy
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Gateway</b></summary>

**GenAI relevance:** Converts APIs/Lambda/services into MCP tools with tool registry, discovery and ingress+egress auth. Converts OpenAPI, Smithy, Lambda and MCP servers into MCP tools with inbound/outbound auth, throttling, multitenancy and semantic tool search.

**Pick it when:** turn Lambda/APIs into MCP tools, tool access control; expose APIs/Lambda as agent tools, hundreds of tools
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Harness</b></summary>

**GenAI relevance:** Managed agent loop: model + system prompt + tools in one API call, no orchestration code; migration target from Bedrock Agents Classic.

**Pick it when:** fastest agent, no container, migrate Bedrock Agents
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Identity</b></summary>

**GenAI relevance:** Inbound and outbound agent authN/authZ, integrates existing IdPs, custom OAuth claims. Inbound/outbound auth for agents with IdP integration, workload identity, credential providers and a token vault for OAuth tokens and API keys.

**Pick it when:** agent auth with existing IdP, OAuth to downstream tools; act on user's behalf, third-party OAuth, consent fatigue
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Memory</b></summary>

**GenAI relevance:** Short-term and long-term (episodic) memory, up to 30 days retention, shareable across collaborating agents. Short-term raw events and long-term extracted preferences/summaries/semantic facts; namespaces, shareable across agents.

**Pick it when:** agent remembers past interactions, shared memory; remember across sessions, context persistence, checkpoint
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Observability</b></summary>

**GenAI relevance:** End-to-end tracing and debugging of agent steps, OpenTelemetry-compatible, integrates with CloudWatch. OpenTelemetry traces/spans of reasoning and tool calls with dashboards for tokens, latency, sessions and errors; works for agents hosted anywhere.

**Pick it when:** trace agent steps, OpenTelemetry; trace agent execution path, find bottlenecks
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Optimization</b></summary>

**GenAI relevance:** AI recommendations and A/B tests on system prompts and tool descriptions via Gateway traffic split.

**Pick it when:** A/B test prompts or tool descriptions
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Payments</b></summary>

**GenAI relevance:** Agent microtransactions via x402 with wallet integration and spending limits.

**Pick it when:** agent pays for services, microtransactions
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Policy</b></summary>

**GenAI relevance:** Converts natural-language boundaries into Cedar policies to restrict agent actions (Preview). Deterministic natural-language or Cedar rules intercepting every tool call via Gateway.

**Pick it when:** restrict agent actions, Cedar; restrict which actions/tools an agent may execute
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Registry</b></summary>

**GenAI relevance:** Governed catalog of agents, MCP servers, tools and skills with approval workflow.

**Pick it when:** org-wide catalog of approved MCP servers/agents
</details>

<details>
<summary><b>Amazon Bedrock AgentCore Runtime</b></summary>

**GenAI relevance:** Serverless, framework-agnostic agent hosting with isolated sessions/containers, auto-scaling, extended runtime and bi-directional streaming. Serverless agent hosting with per-session Firecracker microVM isolation, up to 8 h sessions, MCP/A2A/AG-UI.

**Pick it when:** deploy LangGraph/CrewAI/Strands agent to production, voice streaming, session isolation; long-running agent, session isolation, deploy any framework
</details>

<details>
<summary><b>Amazon Bedrock Agents</b></summary>

**GenAI relevance:** Managed agents with Inline Agents, Custom Orchestration (beyond ReAct), code interpretation and multi-agent collaboration (supervisor). Managed agents that can attach KBs and support multi-agent collaboration with a supervisor; closed to new customers since 30 Jul 2026.

**Pick it when:** runtime-configured agents, supervisor agent, non-ReAct orchestration; legacy managed agents, migrate to AgentCore
</details>

<details>
<summary><b>Amazon Bedrock Session Management APIs</b></summary>

**GenAI relevance:** Fully managed conversation state with checkpointing for LangGraph/LlamaIndex, replay/fork, IAM, KMS and tenant isolation.

**Pick it when:** persist/resume multi-turn state, checkpoint, fork session
</details>

<details>
<summary><b>AWS Agent Squad</b></summary>

**GenAI relevance:** Multi-agent orchestration framework named in the exam guide.

**Pick it when:** multi-agent orchestration framework
</details>

<details>
<summary><b>LangChain (langchain-aws)</b></summary>

**GenAI relevance:** Open-source orchestration framework with BedrockLLM, ChatBedrock, BedrockEmbeddings, retrievers, memory, chains and agents; prototype layer before AgentCore.

**Pick it when:** conversation chains, prototype RAG/agents, LCEL
</details>

<details>
<summary><b>LangGraph</b></summary>

**GenAI relevance:** Controllable agent runtime replacing legacy AgentExecutor; integrates with Bedrock Session Management checkpointing; runs on AgentCore Runtime.

**Pick it when:** controllable custom agent runtime, checkpointing
</details>

<details>
<summary><b>Model Context Protocol (MCP)</b></summary>

**GenAI relevance:** Open standard for agent-to-tool communication with dynamic tool discovery across frameworks; tools exposed as MCP servers.

**Pick it when:** standardised tool interface across frameworks
</details>

<details>
<summary><b>Strands Agents</b></summary>

**GenAI relevance:** AWS open-source, model-driven agent framework (Python/TypeScript, 20+ tools, MCP tools) deployable on AgentCore. Open-source model-first agent SDK with native MCP, any model provider; pairs with AgentCore.

**Pick it when:** model decides which tools to use, few lines of code; open-source AWS agent SDK, custom autonomous agents
</details>

## Guardrails & safety

<details>
<summary><b>Amazon Bedrock Automated Reasoning checks</b></summary>

**GenAI relevance:** Guardrails capability listed in the Bedrock stack for verifying responses. Mathematically/logically verify responses against rules to detect factual errors.

**Pick it when:** formal verification of responses; verifiable reasoning, precision-critical, policy rules
</details>

<details>
<summary><b>Amazon Bedrock Guardrails</b></summary>

**GenAI relevance:** Content filters, denied topics, word filters, PII mask/block, contextual grounding checks, prompt-attack filter and image filtering on every request and response, consistent across FMs. Content filters (incl. prompt attacks), denied topics, word filters, sensitive information filters, contextual grounding and image filters; works with Bedrock, fine-tuned and external models.

**Pick it when:** block harmful content, PII redaction, hallucination grounding, prompt injection; block topics, mask PII, jailbreak, hallucination filter
</details>

<details>
<summary><b>Amazon Bedrock Guardrails contextual grounding check</b></summary>

**GenAI relevance:** Output-layer check scoring grounding against source and relevance to query; filters >75% of hallucinations in KB apps.

**Pick it when:** responses not supported by source docs, RAG hallucination
</details>

<details>
<summary><b>Amazon Bedrock watermark detection</b></summary>

**GenAI relevance:** Detects whether an image was generated by Titan Image / Nova Canvas, with a confidence score.

**Pick it when:** was this image AI-generated
</details>

## Integration & Orchestration

<details>
<summary><b>Amazon API Gateway</b></summary>

**GenAI relevance:** Connects agents to systems; feedback collection for human-in-the-loop. Fronts REST APIs called by OpenAPI action groups (signed requests + error handling).

**Pick it when:** agent-to-system API, feedback endpoint; agent calls REST API
</details>

<details>
<summary><b>Amazon Bedrock Flows (Prompt Flows)</b></summary>

**GenAI relevance:** Workflow orchestration that can use managed prompts from Prompt Management. Low-code visual serverless builder linking prompt, agent, KB, Lambda, Lex, condition, iterator and loop nodes; versions + aliases, InvokeFlow.

**Pick it when:** chain managed prompts into a flow; visual low-code workflow chaining prompts/KB/Lambda
</details>

<details>
<summary><b>Amazon EventBridge</b></summary>

**GenAI relevance:** Event triggers for agents. Event-driven triggering of agents (e.g. compliance agent after generation) and automated incident response.

**Pick it when:** event-driven agent trigger; trigger agent on event
</details>

<details>
<summary><b>Amazon MQ</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Managed ActiveMQ/RabbitMQ broker integrating agents with enterprise/legacy systems over JMS, NMS, AMQP, STOMP, MQTT, WebSocket; launched async classification tasks (Clario) and logged recommendation requests (Snoonu).

**Pick it when:** AMQP, JMS, MQTT, STOMP, migrate ActiveMQ/RabbitMQ, hybrid enterprise integration
</details>

<details>
<summary><b>Amazon SNS</b></summary>

**GenAI relevance:** Agent messaging; queue buffers bursts of KB sync events. Pub/sub fan-out to broadcast events/alerts to many agents or models in parallel; notifies when SageMaker/Rekognition jobs finish or fail and delivers evaluation/guardrail alarms.

**Pick it when:** agent messaging, event burst buffering; broadcast, fan-out, notify many subscribers, multi-channel alerts (SMS/email/HTTP)
</details>

<details>
<summary><b>Amazon SQS</b></summary>

**GenAI relevance:** Agent messaging; queue buffers bursts of KB sync events. Durable queue buffering long-running or bursty GenAI requests to absorb LLM latency and rate limits; batch-inference task queue.

**Pick it when:** agent messaging, event burst buffering; buffer requests, reliable task queue, avoid API Gateway 29 s timeout, absorb throttling
</details>

<details>
<summary><b>AWS AppSync</b></summary>

**GenAI relevance:** Real-time multi-agent synchronisation. GraphQL alternative to REST for the buffered async request/response pattern.

**Pick it when:** real-time multi-agent sync; GraphQL front end for async GenAI
</details>

<details>
<summary><b>AWS Lambda</b></summary>

**GenAI relevance:** Serverless custom agent tools, KB custom transformation before ingestion, deterministic workflow steps. Custom KB chunking, ingestion triggers, Flows nodes, tool execution, Gateway targets, lightweight MCP servers, real-time validation.

**Pick it when:** serverless tool for agent, custom chunk transformation; custom chunking, run tools, trigger sync, stateless MCP server
</details>

<details>
<summary><b>AWS Step Functions</b></summary>

**GenAI relevance:** Deterministic orchestration of workflows that call GenAI/agents as targeted tools. Code-defined workflows with state; ReAct patterns, stopping conditions, human approval workflows.

**Pick it when:** predictable workflow, cheaper/reliable vs all-agentic; code-heavy orchestration, approval workflow, stopping conditions
</details>

## AI services

<details>
<summary><b>Amazon Comprehend</b></summary>

**GenAI relevance:** NLP parsing; detects and redacts PII in text; pre-trained prompt safety classifier before the LLM; sentiment and moderation.

**Pick it when:** PII in text, prompt safety classification
</details>

<details>
<summary><b>Amazon Connect</b></summary>

**GenAI relevance:** Contact centre with AI agents that analyse 100% of voice/chat conversations (sentiment, categorisation, compliance).

**Pick it when:** contact centre GenAI, conversation analytics
</details>

<details>
<summary><b>Amazon Kendra</b></summary>

**GenAI relevance:** Fully managed enterprise semantic search with connectors (S3, SharePoint, Confluence, web); used as a RAG retriever (AmazonKendraRetriever). Enterprise search used to ground internal tools and reduce hallucinations.

**Pick it when:** enterprise semantic search with SharePoint/Confluence connectors; ground internal assistant in company docs
</details>

<details>
<summary><b>Amazon Lex</b></summary>

**GenAI relevance:** Conversational bot node available in Bedrock Flows.

**Pick it when:** chatbot node in a flow
</details>

<details>
<summary><b>Amazon Personalize</b></summary>

**GenAI relevance:** Recommendation models (user-personalization, similar-items, personalized-ranking); separate models per vertical improved precision.

**Pick it when:** personalised recommendations
</details>

<details>
<summary><b>Amazon Q Business</b></summary>

**GenAI relevance:** Enterprise GenAI app (Scope 2); secured with IAM Identity Center RBAC, VPC endpoints, encryption, training opt-out.

**Pick it when:** SaaS GenAI under enterprise contract, Scope 2
</details>

<details>
<summary><b>Amazon Quick Suite</b></summary>

**GenAI relevance:** Layer 1 app: GenAI search, agents, automations and BI over company data via 40+ connectors without data movement. Ready-to-use agentic applications: fastest, lowest risk, highest ongoing cost.

**Pick it when:** business users, search + agents + BI over company data; ready-to-use agents, fast, low budget
</details>

<details>
<summary><b>Amazon Rekognition</b></summary>

**GenAI relevance:** Image/video moderation; job results trigger SNS/SQS workflows.

**Pick it when:** image/video content moderation
</details>

<details>
<summary><b>Amazon Textract</b></summary>

**GenAI relevance:** Parses complex documents (tables, images) in the parsing pillar.

**Pick it when:** extract text/tables from documents
</details>

<details>
<summary><b>Amazon Transcribe</b></summary>

**GenAI relevance:** Speech-to-text used with Bedrock in feedback apps (Tapestry case).

**Pick it when:** transcribe audio before LLM processing
</details>

<details>
<summary><b>Amazon Translate</b></summary>

**GenAI relevance:** Machine translation used with Bedrock (Tapestry case).

**Pick it when:** translate text in GenAI pipeline
</details>

<details>
<summary><b>PartyRock</b></summary>

**GenAI relevance:** Public consumer GenAI playground; example of Scope 1.

**Pick it when:** staff using public GenAI app, Scope 1
</details>

## ML platform (SageMaker)

<details>
<summary><b>Amazon Nova Forge</b></summary>

**GenAI relevance:** Builds custom frontier Nova models with full-weights SFT, RL with custom rewards, and blending with Nova training data.

**Pick it when:** custom frontier model on Nova, proprietary data + custom reward functions
</details>

<details>
<summary><b>Amazon Nova LLM-as-a-Judge</b></summary>

**GenAI relevance:** Pairwise Model A vs B preference judge run as a SageMaker AI training job with recipe; outputs a/b/ties, winrate (B preferred), 95% CI, stderr; ~3% aggregate bias.

**Pick it when:** compare two models on subjective quality at scale, win rate
</details>

<details>
<summary><b>Amazon SageMaker AI</b></summary>

**GenAI relevance:** Advanced fine-tuning (incl. continued pre/mid-training), SageMaker endpoints for custom models, Batch Transform for batch inference, push-button recipes. Hosts custom/domain evaluation models for regulated hallucination management.

**Pick it when:** advanced/open-weight fine-tuning, host custom model endpoint, batch transform; custom domain evaluation model, regulated industry
</details>

<details>
<summary><b>Amazon SageMaker Clarify</b></summary>

**GenAI relevance:** Bias detection for models. Pre- and post-training bias detection, Shapley values and PDPs for explainability.

**Pick it when:** bias assessment; detect bias, explain predictions
</details>

<details>
<summary><b>Amazon SageMaker Data Wrangler</b></summary>

**GenAI relevance:** Rebalances datasets with random undersampling, oversampling or SMOTE.

**Pick it when:** imbalanced dataset, SMOTE
</details>

<details>
<summary><b>Amazon SageMaker JumpStart</b></summary>

**GenAI relevance:** Fine-tune pre-trained models (Scope 4). Source of pre-trained models (Scoping Matrix scope 3).

**Pick it when:** fine-tune FM on SageMaker, Scope 4; pre-trained open models to deploy
</details>

<details>
<summary><b>Amazon SageMaker Unified Studio</b></summary>

**GenAI relevance:** Single governed workspace for data engineering, analytics, ML and GenAI (Bedrock inside), with SageMaker Lakehouse.

**Pick it when:** one workspace for data, ML, GenAI, BI
</details>

## Security & Identity

<details>
<summary><b>Amazon Cognito</b></summary>

**GenAI relevance:** IdP for AgentCore Gateway/Identity inbound OAuth. Customer identity (CIAM) with social/enterprise IdPs and M2M auth; can be the IdP for agent inbound auth.

**Pick it when:** user sign-in for agent, OAuth inbound; customer-facing sign-in, M2M tokens
</details>

<details>
<summary><b>Amazon Detective</b></summary>

**GenAI relevance:** Forensic investigation of security findings.

**Pick it when:** forensic investigation
</details>

<details>
<summary><b>Amazon GuardDuty</b></summary>

**GenAI relevance:** Threat detection incl. suspicious Bedrock/SageMaker API and credential usage; Extended Threat Detection attack sequences; malware protection for S3 and AWS Backup. ML/threat-intel detection of suspicious Bedrock API activity, e.g. removing guardrails after login from a new location.

**Pick it when:** suspicious API/credential use, malware in S3; anomalous Bedrock API use, threat detection
</details>

<details>
<summary><b>Amazon Inspector</b></summary>

**GenAI relevance:** Continuous vulnerability scanning of EC2, container images, Lambda.

**Pick it when:** scan for CVEs
</details>

<details>
<summary><b>Amazon Macie</b></summary>

**GenAI relevance:** ML discovery of PII/PHI/financial data in S3 before training or fine-tuning. Sensitive data discovery jobs finding PII in S3.

**Pick it when:** find PII in S3 training data; PII in S3 buckets
</details>

<details>
<summary><b>Amazon SageMaker Role Manager</b></summary>

**GenAI relevance:** Persona-based roles for data scientists, MLOps and SageMaker compute.

**Pick it when:** persona-based ML roles
</details>

<details>
<summary><b>Amazon Security Lake</b></summary>

**GenAI relevance:** Centralised security data lake for analytics.

**Pick it when:** centralise security logs
</details>

<details>
<summary><b>Amazon Verified Permissions</b></summary>

**GenAI relevance:** Fine-grained application authorisation for zero trust.

**Pick it when:** fine-grained app authz
</details>

<details>
<summary><b>AWS Certificate Manager (ACM)</b></summary>

**GenAI relevance:** Public/private TLS certificates for in-transit encryption.

**Pick it when:** TLS certificates
</details>

<details>
<summary><b>AWS Firewall Manager</b></summary>

**GenAI relevance:** Central management of WAF/Shield policies.

**Pick it when:** central firewall policy management
</details>

<details>
<summary><b>AWS IAM</b></summary>

**GenAI relevance:** Least-privilege access to models and APIs; KB service role permissions (bedrock:InvokeModel, s3, s3vectors, kms). Fine-grained permissions, KB/Flow service roles, AgentCore execution role, inbound/outbound auth.

**Pick it when:** least privilege, allow/deny model access, service role; least privilege, service role, execution role
</details>

<details>
<summary><b>AWS KMS</b></summary>

**GenAI relevance:** Customer managed keys for custom models, KB source data, S3 Vectors (SSE-KMS) and session data. Customer managed keys encrypt flows and KB session data.

**Pick it when:** customer managed key encryption; encrypt flow / session with CMK
</details>

<details>
<summary><b>AWS Private Certificate Authority</b></summary>

**GenAI relevance:** Private CA; now issues post-quantum ML-DSA certificates.

**Pick it when:** quantum-safe certificates
</details>

<details>
<summary><b>AWS Secrets Manager</b></summary>

**GenAI relevance:** Stores secrets; managed external secrets rotate third-party SaaS credentials without custom Lambda.

**Pick it when:** rotate SaaS credentials without Lambda
</details>

<details>
<summary><b>AWS Security Agent</b></summary>

**GenAI relevance:** Frontier agent for design review, code review and on-demand pen testing.

**Pick it when:** security review of design docs, on-demand pen test
</details>

<details>
<summary><b>AWS Security Hub</b></summary>

**GenAI relevance:** CSPM command centre aggregating, correlating and prioritising findings with automated response; AI-powered prioritisation and single bill. Part of data trust-boundary defenses against poisoning alongside Inspector, Detective and Security Lake.

**Pick it when:** single pane for findings, alert overload; centralised security findings
</details>

<details>
<summary><b>AWS Security Incident Response</b></summary>

**GenAI relevance:** Incident management service; AI investigative agent correlates evidence across sources, hours to minutes.

**Pick it when:** speed up incident investigation
</details>

<details>
<summary><b>AWS Shield</b></summary>

**GenAI relevance:** Shield Standard free L3/L4 DDoS; Shield Advanced managed DDoS protection at the network/edge layer.

**Pick it when:** DDoS protection
</details>

<details>
<summary><b>AWS STS outbound identity federation</b></summary>

**GenAI relevance:** GetWebIdentityToken issues short-lived JWTs (60 s - 1 h) so workloads authenticate to external services without stored keys.

**Pick it when:** call external API without stored API keys
</details>

<details>
<summary><b>AWS Verified Access</b></summary>

**GenAI relevance:** Zero-trust application access without VPN.

**Pick it when:** access without VPN
</details>

<details>
<summary><b>AWS WAF</b></summary>

**GenAI relevance:** Filters web traffic to GenAI apps; Bot Control blocks scrapers; Web Bot Auth cryptographically verifies legitimate AI agents on CloudFront. Edge input validation blocking long inputs, known malicious strings and SQLi before the GenAI app.

**Pick it when:** block bots/scrapers, verify AI agents; block malicious/long requests at the edge, rate limit
</details>

<details>
<summary><b>IAM Access Analyzer</b></summary>

**GenAI relevance:** Finds unused/external access, validates policies, custom pre-deployment checks to refine least privilege.

**Pick it when:** refine permissions on actual usage
</details>

<details>
<summary><b>IAM Identity Center</b></summary>

**GenAI relevance:** Workforce SSO; RBAC for Amazon Q Business.

**Pick it when:** workforce SSO
</details>

## Governance & Compliance

<details>
<summary><b>Amazon SageMaker Model Cards</b></summary>

**GenAI relevance:** Document intended use, risk rating, training details, evaluation results for governance and audit. Documents intended use, risk rating, training details and evaluation results.

**Pick it when:** document model provenance/intended use/risk rating; document intended use and risk
</details>

<details>
<summary><b>AWS Artifact</b></summary>

**GenAI relevance:** On-demand download of AWS and ISV compliance reports (SOC, ISO, PCI).

**Pick it when:** download compliance reports for auditor
</details>

<details>
<summary><b>AWS Audit Manager</b></summary>

**GenAI relevance:** Continuous auditing with automated evidence collection against controls. Prebuilt AWS Generative AI Best Practices Framework v2 for Bedrock and SageMaker AI with automatic evidence collection.

**Pick it when:** automate audit evidence; audit-ready GenAI compliance evidence
</details>

<details>
<summary><b>AWS CloudTrail</b></summary>

**GenAI relevance:** Records Bedrock API activity for auditing/troubleshooting; AgentCore Browser logging. Audit trail of Bedrock API calls and AgentCore Browser commands.

**Pick it when:** who called which API, audit trail; audit trail, regulated industry
</details>

<details>
<summary><b>AWS Config</b></summary>

**GenAI relevance:** Configuration compliance monitoring as part of GenAI security practices. Resource configuration history and relationships for audits and misconfiguration detection.

**Pick it when:** track resource configuration compliance; how was resource configured last month
</details>

<details>
<summary><b>AWS Control Tower</b></summary>

**GenAI relevance:** Multi-account landing zone with preventive and detective guardrails; controls-only experience (750+ controls).

**Pick it when:** multi-account guardrails
</details>

<details>
<summary><b>AWS Organizations</b></summary>

**GenAI relevance:** Multi-account management with SCPs as permission boundaries; direct account transfer between orgs.

**Pick it when:** restrict max permissions across accounts
</details>

<details>
<summary><b>AWS Security Reference Architecture (SRA)</b></summary>

**GenAI relevance:** Prescriptive multi-account security blueprint with a GenAI section focused on Bedrock.

**Pick it when:** multi-account security baseline incl. Bedrock
</details>

<details>
<summary><b>AWS Trusted Advisor</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Best-practice checks across cost, performance, resilience, security, operational excellence, service limits.

**Pick it when:** best-practice checks
</details>

## Monitoring & Ops

<details>
<summary><b>Amazon Bedrock Evaluations</b></summary>

**GenAI relevance:** CreateEvaluationJob/GetEvaluationJob for automatic (accuracy, robustness, toxicity) or human model evaluation; CI/CD friendly. Automatic, human, LLM-as-a-judge and RAG/KB evaluations (retrieval + generation, needs expected responses).

**Pick it when:** repeatable model comparison, automated evaluation in pipeline; evaluate model or RAG quality, LLM-as-judge
</details>

<details>
<summary><b>Amazon CloudWatch</b></summary>

**GenAI relevance:** Bedrock usage metrics/dashboards, guardrail invocation metrics, KB sync logs, AgentCore observability integration. Alarms on grounding detections, Gateway logs, AgentCore dashboards; Transaction Search enables trace ingestion.

**Pick it when:** usage metrics, monitor guardrails, sync logs; alerts on hallucination, agent traces missing
</details>

<details>
<summary><b>Amazon CloudWatch Logs Insights</b></summary>

**GenAI relevance:** Queries logs for error types and token usage over time.

**Pick it when:** query logs, aggregate errors/tokens
</details>

<details>
<summary><b>Amazon Data Firehose</b></summary>

**GenAI relevance:** Streams structured agent logs to S3.

**Pick it when:** deliver agent logs to S3
</details>

<details>
<summary><b>Amazon SageMaker Model Monitor</b></summary>

**GenAI relevance:** Monitors model performance drift in production. Data quality, model quality (needs ground truth), model bias, feature attribution (SHAP) and custom monitoring; data capture -> baseline -> schedule.

**Pick it when:** drift monitoring; production data drift, accuracy degrading, feature importance shift
</details>

<details>
<summary><b>AWS Budgets</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Tag-based budgets and alerts for Bedrock spend.

**Pick it when:** budget alerts per team/app
</details>

<details>
<summary><b>AWS Cost Explorer</b></summary>

**GenAI relevance:** Analyses Bedrock spend by cost allocation tags.

**Pick it when:** cost by tag/team
</details>

<details>
<summary><b>AWS Distro for OpenTelemetry (ADOT)</b></summary>

**GenAI relevance:** Vendor-neutral OTEL instrumentation sending traces to CloudWatch, X-Ray, OpenSearch, Managed Prometheus; auto-instruments agents; AgentCore also supports OpenInference, OpenLLMetry, OpenLit, Traceloop.

**Pick it when:** instrument once, vendor-neutral tracing
</details>

<details>
<summary><b>AWS X-Ray</b></summary>

**GenAI relevance:** Distributed tracing for Step Functions (saga/nested), AgentCore, API Gateway, Lambda and SageMaker pipelines.

**Pick it when:** trace end-to-end requests, debug nested workflows
</details>

<details>
<summary><b>Service Quotas</b></summary>

**GenAI relevance:** Request increases for throttling, guardrail InvocationThrottles, latency-optimized quotas and SageMaker instance quotas (e.g. ml.g5.12xlarge).

**Pick it when:** throttling, instance quota exceeded
</details>

## Compute & Networking

<details>
<summary><b>Amazon CloudFront</b></summary>

**GenAI relevance:** CDN fronting apps with WAF/Shield; flat-rate pricing plans.

**Pick it when:** edge protection for app
</details>

<details>
<summary><b>Amazon EC2</b></summary>

**GenAI relevance:** Alternative compute for batch-processing queued GenAI requests.

**Pick it when:** self-managed workers for queue processing
</details>

<details>
<summary><b>Amazon ECS / EKS / AWS Fargate</b></summary>

**GenAI relevance:** Container hosting for agents and complex MCP servers. Serverless container workers processing queued GenAI requests.

**Pick it when:** complex MCP server, containerised agents; serverless containers for async workers
</details>

<details>
<summary><b>Amazon Route 53</b></summary>

**GenAI relevance:** DNS; DNS Firewall (DGA protection) and Global Resolver stop DNS exfiltration.

**Pick it when:** DNS exfiltration
</details>

<details>
<summary><b>Amazon VPC</b></summary>

**GenAI relevance:** Network isolation controls for GenAI workloads. Network isolation for tool Lambdas, training and inference; security groups; VPC encryption controls enforce encrypted traffic.

**Pick it when:** network isolation; restricted egress, isolated training, block unencrypted traffic
</details>

<details>
<summary><b>AWS Batch</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Container-based batch processing for batch inference.

**Pick it when:** batch inference with containers
</details>

<details>
<summary><b>AWS Direct Connect</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Encrypted private link from on-prem to AWS for document ingestion (Clario).

**Pick it when:** on-prem to AWS dedicated connection
</details>

<details>
<summary><b>AWS Inferentia & Trainium</b></summary>

**GenAI relevance:** Training accelerator, up to 4x faster / 2x lower cost vs GPU; Trainium3 = 3nm, ~40% more energy-efficient, 2x compute vs Trainium2. Inference accelerator, up to 30% lower cost per inference with high throughput and low latency.

**Pick it when:** cost-effective FM training at scale; low-cost high-throughput self-hosted inference
</details>

<details>
<summary><b>AWS Network Firewall</b></summary>

**GenAI relevance:** VPC-wide traffic filtering and TLS inspection to prevent exfiltration; Proxy (Preview) adds explicit egress forward proxy. Controls VPC ingress/egress at the application trust boundary.

**Pick it when:** inspect egress TLS, VPC-wide filtering, explicit proxy; egress control for GenAI apps
</details>

<details>
<summary><b>AWS PrivateLink</b></summary>

**GenAI relevance:** Private connectivity from VPC to Bedrock without internet; Scope 3 security control. Private connectivity for Bedrock and AgentCore Runtime/Browser.

**Pick it when:** private access, no internet traversal; keep agent traffic private
</details>

## Data & Storage

<details>
<summary><b>Amazon Athena</b></summary>

**GenAI relevance:** Serverless SQL over data lake within SageMaker Unified Studio. Queries Model Monitor drift data in S3.

**Pick it when:** query data lake with SQL; SQL over drift results in S3
</details>

<details>
<summary><b>Amazon DynamoDB</b></summary>

**GenAI relevance:** Listed as a vector/storage option for GenAI apps (e.g. metadata, state). Agent state/session store and single source of truth; native tool target.

**Pick it when:** key-value store for GenAI app data; agent state, avoid conflicting updates
</details>

<details>
<summary><b>Amazon EMR</b></summary>

**GenAI relevance:** Distributed data processing and batch inference within SageMaker Unified Studio.

**Pick it when:** distributed batch processing
</details>

<details>
<summary><b>Amazon Kinesis Data Streams</b></summary>

**GenAI relevance:** Real-time event streaming into Lambda/Personalize and SageMaker inference pipelines.

**Pick it when:** real-time clickstream/event ingestion processing
</details>

<details>
<summary><b>Amazon OpenSearch Ingestion</b></summary>

**GenAI relevance:** Loads pre-computed embeddings from S3 into OpenSearch.

**Pick it when:** embeddings generated externally in S3
</details>

<details>
<summary><b>Amazon Redshift</b> ⚠️ <i>Listed as out of scope in the exam guide</i></summary>

**GenAI relevance:** Data warehouse for SQL analytics, unified via SageMaker Lakehouse.

**Pick it when:** warehouse analytics
</details>

<details>
<summary><b>Amazon S3</b></summary>

**GenAI relevance:** Source data for Knowledge Bases (same or cross-account), JSONL training/distillation data, metadata.json sidecars, KB log delivery. Primary KB data source (files \<50 MB) with .metadata.json sidecars; Flows S3 nodes; Code Interpreter data input.

**Pick it when:** KB data source, training data location; document source for RAG, metadata sidecar
</details>

<details>
<summary><b>Amazon S3 Object Lambda</b></summary>

**GenAI relevance:** Redacts PII on retrieval from S3.

**Pick it when:** redact data at read time
</details>

<details>
<summary><b>Amazon SageMaker Lakehouse</b></summary>

**GenAI relevance:** Unifies S3 data lakes, Redshift and third-party sources for AI/analytics.

**Pick it when:** unify data lake + warehouse
</details>

<details>
<summary><b>AWS Backup</b></summary>

**GenAI relevance:** Policy-based cross-Region/account backups, immutable/air-gapped vaults, Backup Audit Manager compliance reporting.

**Pick it when:** ransomware-proof backups, retention compliance
</details>

<details>
<summary><b>AWS Glue</b></summary>

**GenAI relevance:** Data processing/ETL within SageMaker Unified Studio for data foundations. Data collection/prep and workflows; Glue Data Quality and DataBrew for quality checks.

**Pick it when:** data prep/ETL for GenAI; data quality checks in pipeline
</details>

## Dev tools

<details>
<summary><b>Amazon ECR</b></summary>

**GenAI relevance:** Stores AgentCore Runtime agent container images; created by agentcore configure.

**Pick it when:** agent container image registry
</details>

<details>
<summary><b>AWS CloudFormation</b></summary>

**GenAI relevance:** Infrastructure as code for GenAI CI/CD and MLOps. IaC deployment of AgentCore Runtime resources.

**Pick it when:** IaC for MLOps pipeline; infrastructure as code for agents
</details>

<details>
<summary><b>aws login (AWS CLI)</b></summary>

**GenAI relevance:** Browser-based CLI/SDK sign-in with console credentials and auto-expiring temporary tokens.

**Pick it when:** developer CLI without access keys
</details>

<details>
<summary><b>IAM Policy Autopilot</b></summary>

**GenAI relevance:** MCP server letting coding assistants author/troubleshoot valid least-privilege IAM policies via deterministic code analysis.

**Pick it when:** coding assistant writes IAM policy
</details>

<details>
<summary><b>Kiro</b></summary>

**GenAI relevance:** AI coding tool that also auto-generates unit and integration tests, including for legacy code. Ready-to-use agentic applications: fastest, lowest risk, highest ongoing cost.

**Pick it when:** AI-assisted coding, generate tests; ready-to-use agents, fast, low budget
</details>

---
[← Back to index](README.md)
