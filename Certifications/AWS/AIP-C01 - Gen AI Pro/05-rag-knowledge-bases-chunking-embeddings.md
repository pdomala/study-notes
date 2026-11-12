[← Back to index](README.md)

# 📚 05 · RAG, Knowledge Bases, chunking & embeddings

> Exam domain: D1

## Key points
- **RetrieveAndGenerate** = fully managed RAG with citations and session ID · **Retrieve** = chunks + scores only, you own generation (custom rerank, LangChain).
- KB returns **5** results by default, max **100**. Data sources can be S3 (files **under 50 MB**), web crawler, Confluence, SharePoint, Salesforce.
- **Fixed after KB creation**: parser, chunking strategy, embedding model, vector store. Embedding dimension ≠ index dimension → **ingestion fails**.
- Chunking: default **~300 tokens** · fixed (+ overlap) · **semantic** (unclear boundaries) · **hierarchical** (nested docs: match child, return parent) · none. Tables/scans → **FM parsing** (or Bedrock Data Automation). Bespoke split logic → **Lambda custom chunking**.
- Metadata filters = sidecar `doc.pdf.metadata.json` → **re-sync** after adding. Keep data fresh with sync jobs (EventBridge/Lambda on S3 changes).
- **S3 Vectors** = cheapest store, subsecond, **semantic only**; filterable metadata **2 KB**, total **40 KB** per vector → put `AMAZON_BEDROCK_TEXT` in non-filterable. Hybrid search or ms latency → **OpenSearch Serverless**.
- Titan Embeddings V2: 1024 → 512 dims ≈ **99%**, → 256 ≈ **97%** accuracy (cheaper storage). Binary embeddings cut storage further, with some accuracy loss.
- **Reranker** = better relevance + fewer chunks (text only) · multi-part question → **query decomposition** · repeated questions → **semantic cache** · entity/relationship multi-hop → **GraphRAG (Neptune Analytics)**.

## Notes

### Why RAG — problems fixed, benefits, use cases & challenges
- Pre-trained FM limits (hallucinations, stuck in time, no enterprise context, lower domain accuracy, low interpretability) → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (FM & LLM fundamentals)
- RAG adds external knowledge **before generation, without retraining** → cost-effective domain tailoring; also fixes **unreliable sources** and **terminology confusion** (same term, different meanings)
- Gives orgs control: answers grounded in **pre-determined, authoritative sources**; retrieved results as context → **reduces hallucinations**
- RAG = **retrieval system** (sparse/dense retrieval over a corpus) + **generative LLM** (synthesises the answer from retrieved context); traditional RAG = vector similarity search
- Benefits: **cost-effective** (lower cost + compute vs custom model/retraining) · **current information** (news, social feeds, frequently updated sources) · **enhanced user trust** (source attribution/citations) · **more developer control** (swap info sources, test faster, **restrict sensitive info by authorisation level**)
- Use cases: **question answering** (customer service, tech support, research) · **content generation** · **conversational AI** · **data analysis & insights** · **fact-checking & verification** · **personalised content** (product descriptions, marketing, recommendations) · ❌ not RAG: video translation, scheduling social posts, ordering from a general menu
- Business apps: **intelligent Q&A**, **expanding/enriching knowledge bases**, **high-quality content generation**; KB examples: customer-service chatbot (product KB), legal research, healthcare Q&A
- QA patterns (generic base FM vs personalised RAG) and the too-much-context trade-off → See [01 · Fundamentals](01-fundamentals-genai-strategy.md) (Anatomy of a GenAI application)
- RAG limits: only knows what's embedded at retrieval time, **model stays static**, large context → latency
- Challenges:
  - **Updating external data** → async refresh of docs + embeddings: **real-time** or **periodic batch** (better for large datasets)
  - **Scalability** → bigger corpus = slower/costlier retrieval → efficient indexing
  - **Relevance & accuracy** of retrieved info for complex queries
  - **Bias & fairness** → KB biases flow into output
  - **Evaluation** must cover retrieval **and** generation
  - Multimodal / structured data → KBs now support **native multimodal retrieval**

### RAG architecture & pipeline
- 4-step architecture:
  1. **Data ingestion** → external data (APIs, DBs, doc repos) → **chunk** → **embeddings model** → **vector DB** · ⚠️ check the embedding model's **supported languages**
  2. **Retrieve** → query → vector → **semantic search** (vector maths) for relevant chunks
  3. **Augment** → inject retrieved chunks into the prompt (prompt engineering)
  4. **Generate** → augmented prompt → FM → response
- 6-step pipeline view: identify **clean, domain-specific** data → **chunk** into meaningful pieces → **create vectors** (store understands **intent**) → **store** in vector store → **semantic similarity search** (meaning, not just keywords) → **include similar context in prompt**
- Components: **knowledge base** (chunked sources stored as text + **dense or sparse vectors**) · **retriever model** (narrows the search space, must understand **user intent**; retrieved docs + question → prompt → LLM)
- Query is embedded with the **same embedding model** used at ingestion
- **Embedding** = numerical representation of text/image/audio in vector space; similar meaning → nearby points; vectors + **metadata** indexed in a purpose-built vector DB; similarity via **k-NN** or **cosine similarity**
- Purpose of vectorising chunks → capture the **intent behind searches** (semantic similarity), not just keyword matching
- No vector DB = no RAG

### Chunking strategies & chunk sizing
- Chunk size governed by **retrieval goals** (doc-level vs concept/sentence-level similarity) + **LLM context window**; ask "what format helps the FM do the task?"
- Trade-off: **too small** → faster retrieval but chunks not meaningful enough for semantic search, **LLM lacks context** · **too large** → more context but **irrelevant data/noise**, more compute → **iterate/test**
- Bedrock KB built-in options (**automatic chunking** is a KB capability):
  - **Default** → ~**300 tokens** (≤300), keeps sentence boundaries
  - **Fixed-size** → max tokens default **300** (range **20–8192**) + **overlap %** default **20%** (typical **10–20%**); max chunk size depends on embedding model: **Titan Text Embeddings = 8192 tokens**, **Cohere Embed = 512 tokens**; predictable token sizes, simple
  - **Hierarchical** → **parent + child** chunks (settings: max parent / max child tokens + overlap tokens); **match on small child, return large parent**; nested docs (legal, technical manuals, academic, nested tables); pairs well with **FM parsing**
  - **Semantic** → boundaries by embedding similarity (settings: **buffer size**, max tokens, **breakpoint threshold**); costlier; good when boundaries are unclear (legal, manuals) and for complex technical queries needing context
  - **No chunking** → whole doc/file = one chunk → use for pre-split files
- General methods:
  - **Fixed** → equal words/chars (e.g. 1,000 words / 3,000 chars); fast but **not context-aware** → improve with **recursive character splitting** (line breaks) + **overlap**
  - **Structural** → split on HTML, Markdown headers, code blocks; needs a **well-defined schema**
  - **Semantic** → compare adjacent sentence embeddings, split when difference crosses a **percentile threshold** · e.g. threshold 90 + 101 sentences → 100 pairs → 10 least-similar split → **11 chunks** (further split if > max tokens)
  - Other content-aware: sentence-level, recursive, Markdown/LaTeX; **hybrid** chunking approaches
- **Custom chunking** → your own code as a **Lambda function**; can use LangChain / LlamaIndex components
- ⚠️ Chunking strategy **cannot be changed after creation** → recreate the data source/KB

### Parsing & ingestion transformation
- **Bedrock default parser** → standard text, text-only docs, **no extra cost**
- **FM parsing** (FM as parser) → complex structure: PDFs, scanned images, **tables**; alternative **Bedrock Data Automation** for complex docs with tables/images/visuals
- **Custom** parsing option
- Advanced: custom **Lambda transformation** before ingestion
- ⚠️ Parsing strategy **cannot be changed after creation**

### Embedding models & embedding concepts
- Selection factors: **dimensionality** (1024 = richer, costlier; smaller = cheaper) · **multilingual support** · **performance metrics** (retrieval accuracy, similarity) · **compatibility/integration** · doc type, search method, volume (text-only → text model; images + text → **multimodal** model)
- Use the **same model at ingest and query**
- **Titan Text Embeddings V2** → text; dims **256 / 512 / 1024**; 1024→512 keeps **~99%** accuracy, →256 keeps **~97%** → cut storage cost; **100+ languages**; **unit vector normalisation**
- KB-listed models also include **Titan Embeddings G1 – Text** and **Cohere Embed English / Multilingual**
- **Cohere Embed v3** → English + multilingual, multimodal
- **Cohere Embed v4** → **multimodal** (text + images, interleaved); complex business docs with **tables, graphs, diagrams**; **100+ languages**; float/int8/uint8/binary/ubinary; dims **256–1536**
- **Amazon Nova Multimodal Embeddings** → text, docs, images, video, audio in **one vector space** → **cross-modal retrieval** (text query → images); semantic search, agentic RAG
- **Titan Multimodal Embeddings G1** → images + text (e.g. Bynder: search over 175M assets, −75% search time)
- **Embeddings type**: **float32** = more precise, more storage cost · **binary** = cheaper storage, less precise, not supported by all models/stores
- **Vector dimensions** (e.g. Titan V2 = 1024): higher = better accuracy but more storage/latency
- Embeddings capture the semantics of the **phrase, not the string**: synonyms close; reordered sentence still close; German translation closer than an unrelated sentence → cross-lingual · distance score **0 = most similar**, 1 = least
- ⚠️ Embedding model **can't be changed after KB creation**

### Vector store selection
- Selection factors: **data volume/scale**, **query performance/latency**, **metadata filtering**, **hybrid search**, integration, deployment model, cost, security/compliance, availability/durability
- **KB-supported stores**: Aurora PostgreSQL (pgvector), OpenSearch Serverless, OpenSearch managed clusters, S3 Vectors, MongoDB Atlas, **Neptune Analytics (GraphRAG)**, Pinecone, Redis Enterprise Cloud
- KB vector store: let Bedrock **quick-create** (OpenSearch Serverless / S3 Vectors) or bring your own
- Wider AWS vector DB options: **OpenSearch Service/Serverless**, **Aurora PostgreSQL & RDS for PostgreSQL (pgvector)**, **Neptune Analytics / Neptune ML**, **DocumentDB**, **DynamoDB**, **MemoryDB**, **ElastiCache (Redis OSS)**, **Kendra**, **Bedrock Knowledge Bases**; also **Pinecone** (AWS Marketplace); open-source in-memory → **FAISS**, **Chroma**
- **S3 Vectors** = cheapest, subsecond, **semantic search only** · need hybrid/keyword search, high QPS or millisecond latency → **OpenSearch Serverless**
- ⚠️ Vector store (type) **can't be changed after KB creation**
- KB-on-OpenSearch-Serverless index setup (FAISS, dims, text + metadata fields) → see [06 · Vector search & OpenSearch](06-vector-search-opensearch.md)

### Bedrock Knowledge Bases — overview & setup
- **Fully managed** RAG workflow (ingestion → retrieval → prompt augmentation), no custom data-source integrations; **built-in session context** → multi-turn/conversational
- Capabilities: **traditional RAG** (vector similarity) · **GraphRAG** (GA) · **multimodal** (text, images, docs) · **automatic chunking** · **real-time updates** without downtime
- Handles content comparison, **failure handling, throughput control, encryption**; all answers come with **citations / source attribution** → transparency, fewer hallucinations
- KBs can be attached to **Bedrock Agents**
- Create: name + **runtime (service) role** → data source → **embedding model** → **vector store**; console path Bedrock → Knowledge Bases → Create → **"Knowledge Base with vector store"**; 5 steps: **Create KB → Configure data source → Configure storage & processing → Sync → Test**
- Prerequisites: IAM role with Bedrock + S3 access; **model access enabled before creating the KB** for the embedding model (e.g. Titan Text Embeddings V2) and generation model (e.g. Nova Pro)
- **Data sources**: **S3** (same account **or cross-account**), **web crawler**, Confluence, Salesforce, SharePoint; up to **5 data sources** per KB (1 + 4 more)
- S3 file types: .txt, .md, .pdf, .html, .doc/.docx, .csv, .xls/.xlsx; files **under 50 MB**
- Optional: tags, log delivery to **S3 or CloudWatch**, **KMS CMK** encryption, **customer-managed KMS key for the source S3 data** (so Bedrock can decrypt it)
- ⚠️ **Fixed after creation**: parser, chunking strategy, embedding model, vector store
- Data source **data deletion policy**: default **Delete** (vectors removed when the data source is deleted) or **Retain**
- SDK: `create_knowledge_base` → `knowledgeBaseConfiguration.type = VECTOR`, set `embeddingModelArn` + `dimensions` (must match index) + `embeddingDataType = FLOAT32`; `storageConfiguration.type = S3_VECTORS` + `s3VectorsConfiguration.indexArn`

### Amazon S3 Vectors as the KB vector store
- Cost problem: traditional vector DBs run on **SSD / in-memory** storage → expensive as KBs and embedding granularity grow → teams **shrink RAG scope** or use **coarser embeddings**; archival/historical data that must stay searchable doesn't need high-QPS pricing
- **S3 Vectors** = first cloud object storage with **native vector store + query**; **up to 90% lower** cost for vector **upload, storage and query**; **subsecond** latency (not millisecond); pay-as-you-go, no infra; still needs an embedding model
- Supports **metadata filtering** (dates, categories, sources); **semantic search ONLY** (no hybrid/keyword)
- Handles vectors from massive **unstructured data: text, images, audio, video**
- Ideal use cases: **extensive document repositories**, **historical archives**, **granular vector representations** (fine-grained chunks without cost blow-out)
- KB storage type `S3_VECTORS`
- Vector store options: **Quick create** (recommended) → auto-creates **vector bucket + vector index** · **Use existing** → provide **vector bucket ARN + vector index ARN**
- Index params: `dimension`, `distanceMetric` (**cosine** / euclidean), `dataType=float32`
- ⚠️ **Embedding model dimensions MUST match the vector index dimension** → mismatch = **ingestion failure**
- Encryption: default **SSE-S3**; tick **"Add AWS KMS key for S3 vector bucket"** in Quick create → **SSE-KMS**
- Metadata = key-value pairs per vector (string, number, Boolean):
  - **Total metadata: up to 40 KB per vector** · **filterable metadata: max 2 KB per vector**
  - All metadata is **filterable by default** unless listed in `nonFilterableMetadataKeys`
  - Add `AMAZON_BEDROCK_TEXT` (the chunk text) to `nonFilterableMetadataKeys` → large chunks don't eat the 2 KB filterable budget
  - ⚠️ Non-filterable keys **can't be used in filters** and **can't be changed after index creation**
- ARN format: `arn:aws:s3vectors:{region}:{acct}:bucket/{bucket}/index/{index}`

### KB service role permissions & cleanup (S3 Vectors)
- KB service role needs:
  - `bedrock:InvokeModel` on embedding (+ generation/rerank) models
  - `s3:ListBucket` + `s3:GetObject` on the source bucket (`GetObject` scoped to `{SOURCE_BUCKET}/{PREFIX}/*`)
  - `s3vectors:GetIndex, QueryVectors, PutVectors, GetVectors, DeleteVectors` on the index ARN
  - `kms:GenerateDataKey` + `kms:Decrypt` **only if using a customer managed key**
  - Use `aws:ResourceAccount` condition to scope access
- **Before deleting the KB, note the IAM service role name + S3 vector index ARN** → deleting the KB does NOT delete the vector bucket/index or role (separate cleanup to stop charges)
- Delete order: **KB → vector index → vector bucket → IAM role → source S3 data**
- CLI: `aws s3vectors delete-index` then `delete-vector-bucket` (index must go first)

### Sync & keeping data fresh
- Add / modify / delete docs → run an **ingestion job (sync)** = **incremental** sync
- Sync flow: pull latest from S3 → parse → chunk → embed → store/index (write to vector index)
- Automate via **Lambda** on a **schedule or event**; for events add an **event queue** (bursts)
- KB is ready for Retrieve / RetrieveAndGenerate after the job completes
- Monitor sync via **CloudWatch Logs** (if configured)
- Data source console actions: **Sync**, **Stop sync**, **Add documents from S3** (ingest specific docs without full re-sync)

### RetrieveAndGenerate vs Retrieve APIs
- **RetrieveAndGenerate** (fully managed RAG): query → embed → search/retrieve → augment → **FM generates** → response + **citations** + retrieved chunks
  - **Required**: query text, **knowledge base ID**, **model ARN** · **Optional**: prompt template, numberOfResults, search type, **session ID (+ KMS key)** for multi-turn
  - Client: `bedrock-agent-runtime` → `retrieve_and_generate(type='KNOWLEDGE_BASE')`
  - **Custom prompt template** → tone/format; placeholders `$search_results$`, `$query$`, `$output_format_instructions$`
  - **overrideSearchType** → `SEMANTIC` (vectors only) or `HYBRID` (vectors + raw text); Default = Bedrock decides
  - **numberOfResults** → default **5**, max **100**
  - Metadata filter via `retrievalConfiguration` · reranking model · query decomposition (`orchestrationConfiguration`) · inference params · Guardrails (`guardrailId` + version)
- **Retrieve** (customised RAG): KB only does retrieval → returns **contexts** (text, S3 location, metadata, relevance **score**) to your app
  - Required: **query text** · Optional: next token, numberOfResults, search type, filters
  - You own generation: **InvokeModel** with your own prompt, or **LangChain** (`AmazonKnowledgeBasesRetriever` + `RetrievalQA`, chain\_type `stuff`)
  - Use when you need control after search: custom prompts, 3rd-party orchestration, **custom re-ranking**
- Query-time options are the same across vector stores: **metadata filters, Guardrails, reranking, query modification**
- Testing: Retrieve → raw chunks + relevance scores + metadata; RetrieveAndGenerate → chunks passed to an FM for an answer; console Test KB → configure icon → search type, max source chunks, filters; validate source chunks + scores + metadata before production

### Metadata & metadata filtering
- Per-doc sidecar JSON next to the source file in the bucket: `doc.pdf.metadata.json` (`<file>.metadata.json`) → `{"metadataAttributes": {"year": 2019}}`
- ⚠️ **Re-sync** the data source after adding metadata files
- Filter at retrieval via `retrievalConfiguration`, e.g. `{'equals': {'key': 'year', 'value': 2019}}` → "Who is CEO?" → 2019 = Bezos, 2022/23 = Jassy
- Metadata filters **narrow the search space** → more accurate, better relevance
- Metadata filtering on OpenSearch Serverless (FAISS requirement) → See [06 · OpenSearch](06-vector-search-opensearch.md) (KBs on OpenSearch Serverless)
- Store **metadata with chunks** → references/traceability + **ACL** requirements
- S3 Vectors metadata limits (2 KB filterable / 40 KB total) → see the S3 Vectors section above

### Advanced retrieval — hybrid, reranking, query decomposition, semantic cache, GraphRAG
- **Hybrid search** → keyword (facts, exact terms) + semantic (nuance); Bedrock picks by default; `HYBRID` only with **OpenSearch Serverless** (with a filterable text field, else falls back to semantic); S3 Vectors = semantic only
- **Re-ranking** → initial results miss nuance or return too many docs
  - Options: **custom rerank function** · **FM re-ranks** via prompt · **Bedrock Reranker models**
  - Reranker models are trained for relevance → better accuracy; **fewer chunks to FM = lower cost + latency**
  - Use via **Rerank API** (query + docs) or inside Retrieve / RetrieveAndGenerate (`rerankingConfiguration`, e.g. retrieve 10 → keep 5); overrides default ranking
  - ⚠️ Reranking = **text only**
- **Query reformulation** at retrieval for complex queries; **query decomposition** (`orchestrationConfiguration`) avoids **semantic dilution** of multi-part queries → subqueries retrieved separately, pooled + ranked
- **Semantic cache** → reuse retrieval + FM output for similar repeated queries → lower latency/cost
- **GraphRAG** (GA in KB) → knowledge graph of entities + relationships/hierarchies → **multi-hop reasoning**; best for **root-cause analysis, compliance tracing, enterprise knowledge discovery**; store = **Neptune Analytics**
- Best practices: **define domain + curated data** (support bot → manuals/FAQs; medical QA → medical DBs/papers) · chunking fit to task · metadata filters · prompt engineering + templates · **continuous evaluation & refinement** (Guardrails → [10 · Guardrails](10-guardrails-hallucination.md); RAG evaluation → [14 · Evaluation](14-evaluation-testing-troubleshooting.md))

### LangChain retrieval components (indexes)
- **Document loaders** → load docs from sources (DB, online, local) **to generate embeddings**; HTML, PDF, code; e.g. `S3FileLoader(bucket, key)`
- **Text splitters** chunk loaded docs
- Text embedding models (`BedrockEmbeddings`) and packages → See [02 · FMs](02-foundation-models-bedrock-apis-inference.md) (Calling Bedrock models from LangChain)
- **Retriever** → searches the index for the most relevant docs for a query → sends to app
  - **Amazon Kendra** → fully managed **semantic search**, connectors (S3, SharePoint, Confluence, web), HTML/Word/PPT/PDF/Excel → `AmazonKendraRetriever`
  - Pair with `ConversationalRetrievalChain.from_llm(llm, retriever, return_source_documents=True)`
  - KB retriever: `AmazonKnowledgeBasesRetriever` + `RetrievalQA`
- **Vector stores** → docs → embeddings → store in vector DB → retrieve relevant → pass to LLM as context
  - Supported: **OpenSearch / OpenSearch Serverless vector engine** (`OpenSearchVectorSearch`), **Aurora PostgreSQL pgvector**; `.as_retriever()`

## Exam triggers
- "Answer questions from our docs", "up-to-date info", "personalised/domain-specific QA" → **RAG** (not a base FM)
- "Managed RAG, minimal code, citations" → **Knowledge Bases + RetrieveAndGenerate**
- "Custom generation / own prompt / LangChain / custom rerank" → **Retrieve** + InvokeModel
- "Multi-turn KB chat" → **session ID** (built-in session context)
- "Answers mix years/departments/tenants" → **metadata filter** (sidecar `.metadata.json` + re-sync)
- "Traceable answers / ACL on chunks" → store **metadata** with chunks
- "Exact keywords/product codes missed by KB semantic search" / "keyword match + meaning" → **hybrid search** (BM25 + k-NN scores, normalised) → **OpenSearch Serverless**
- "Retrieved chunks relevant but poorly ordered / too many to FM" → **reranker model**
- "Complex multi-part question gets weak retrieval" → **query decomposition/reformulation**
- "Same questions asked repeatedly, cut latency/cost" → **semantic cache**
- "PDFs with tables / scanned docs" → **FM parsing** (or Bedrock Data Automation)
- "Nested legal/manual docs", "search small, return big context" → **hierarchical chunking**
- "Unclear section boundaries" → **semantic chunking**
- "Bespoke split logic" → **custom chunking Lambda**
- "Docs with HTML/Markdown/code structure" → **structural chunking**
- "Fixed chunks losing context" → add **overlap** / **recursive character splitting**
- "Need to change chunking" → **recreate the data source/KB**
- "Reduce vector storage cost, keep accuracy" → **Titan V2 at 512/256 dims** (~99% / ~97%)
- "Search images with text", "one space across text, images, video, audio" → **Nova Multimodal Embeddings** (Cohere Embed v4 for text + images)
- "Docs full of tables/charts, 100+ languages" → **Cohere Embed v4**
- "Vectors in existing Postgres" → **Aurora/RDS + pgvector** · "in-memory open source" → **FAISS/Chroma**
- "Relationship-heavy data", "multi-hop root-cause / compliance tracing" → **GraphRAG on Neptune Analytics**
- "Keep KB fresh automatically" → **Lambda-triggered ingestion job** (schedule/event + queue)
- "Lowest cost", "millions of docs", "archival but searchable", "can tolerate subsecond latency" → **S3 Vectors**
- "High QPS", "millisecond latency", "hybrid / keyword search" → **OpenSearch Serverless** (not S3 Vectors)
- "Ingestion fails after connecting existing index" → **embedding dimension mismatch**
- "Filtering fails / metadata too large" → move `AMAZON_BEDROCK_TEXT` to **non-filterable**
- "KB file rejected" → check **over 50 MB** / unsupported type
- "Fetch relevant docs for a query" → **retriever** · "load docs from S3/DB" → **document loader** · "query embeddings" → **vector store**
- "Enterprise semantic search with SharePoint/Confluence connectors as RAG retriever" → **Amazon Kendra**

## Flashcards
<details>
<summary><b>Q:</b> What two problems do too-small chunks cause?</summary>

Chunks aren't meaningful enough for semantic search, and the LLM doesn't get enough context. Too-large chunks pass irrelevant data.

*D1 · Chunk size trade-offs*
</details>

<details>
<summary><b>Q:</b> Which KB chunking strategy suits nested documents such as legal texts and technical manuals, and which parsing option pairs with it?</summary>

Hierarchical chunking (parent + child chunks), paired well with FM parsing.

*D1 · Chunking strategy choice*
</details>

<details>
<summary><b>Q:</b> A single embedding space is needed across text, documents, images, video and audio. Which model?</summary>

Amazon Nova Multimodal Embeddings.

*D1 · Cross-modal retrieval*
</details>

<details>
<summary><b>Q:</b> A team needs bespoke split logic (e.g., using LlamaIndex components) during KB ingestion. How do they implement it?</summary>

Custom chunking implemented as a Lambda function, which can use LangChain or LlamaIndex components.

*D1 · Custom chunking*
</details>

<details>
<summary><b>Q:</b> What does the default Bedrock KB chunking strategy do?</summary>

Splits into chunks of roughly 300 tokens while keeping sentence boundaries.

*D1 · Default chunking*
</details>

<details>
<summary><b>Q:</b> After connecting an existing S3 vector index to a new knowledge base, ingestion fails. What is the most likely cause?</summary>

The embedding model's output dimension does not match the vector index dimension; they must match.

*D5 · Dimension mismatch*
</details>

<details>
<summary><b>Q:</b> How do you attach per-document metadata for filtering in a Bedrock KB S3 data source?</summary>

Add a `<file>.metadata.json` sidecar file next to each document in the source bucket.

*D1 · Document metadata*
</details>

<details>
<summary><b>Q:</b> What is the trade-off between float32 and binary embeddings?</summary>

Float32 is more precise but costs more storage; binary is cheaper to store but less precise and not supported by all models/stores. Higher dimensions also increase accuracy but storage/latency.

*D4 · Embedding data type*
</details>

<details>
<summary><b>Q:</b> How can a team cut vector storage cost with Titan Text Embeddings V2 while keeping most accuracy?</summary>

Reduce dimensions from 1024 to 512 (~99% accuracy retained) or 256 (~97% retained).

*D4 · Embedding dimensions cost*
</details>

<details>
<summary><b>Q:</b> In LangChain embedding comparisons, does a distance score of 0 mean most or least similar?</summary>

0 = most similar, 1 = least. Embeddings capture semantics, so translations and reordered sentences stay close.

*D1 · Embedding distance*
</details>

<details>
<summary><b>Q:</b> A company needs embeddings for business documents full of tables and charts in 100+ languages. Which model?</summary>

Cohere Embed v4.

*D1 · Embedding model choice*
</details>

<details>
<summary><b>Q:</b> S3 Vectors ingestion/filtering fails because chunk text exceeds the filterable metadata budget. What is the fix?</summary>

Add `AMAZON_BEDROCK_TEXT` to `nonFilterableMetadataKeys` so chunk text doesn't consume the 2 KB filterable limit (note: non-filterable keys can't be changed after index creation).

*D5 · Filterable metadata overflow*
</details>

<details>
<summary><b>Q:</b> Fixed-size chunks are losing context at boundaries. How can this be improved?</summary>

Add overlap and use recursive character splitting (e.g., on line breaks).

*D1 · Fixed chunk context loss*
</details>

<details>
<summary><b>Q:</b> What are the default and allowed values for fixed-size chunking in Bedrock KB?</summary>

Max tokens default 300 (range 20–8192); overlap default 20% (typical 10–20%). Max chunk size depends on embedding model: Titan Text Embeddings 8192 tokens, Cohere Embed 512.

*D1 · Fixed-size chunking*
</details>

<details>
<summary><b>Q:</b> Which KB-supported vector store enables GraphRAG for relationship-heavy data?</summary>

Amazon Neptune Analytics.

*D1 · GraphRAG store*
</details>

<details>
<summary><b>Q:</b> Which Knowledge Bases retrieval approach supports multi-hop reasoning over entities and relationships, e.g. for root-cause analysis or compliance tracing?</summary>

GraphRAG (GA in Bedrock Knowledge Bases), which uses a knowledge graph of entities and relationships rather than pure vector similarity.

*D1 · GraphRAG use cases*
</details>

<details>
<summary><b>Q:</b> Users report exact product codes are missed by KB semantic search. What search type helps and which vector store supports it?</summary>

HYBRID search (vectors + raw text) via overrideSearchType; supported with OpenSearch Serverless.

*D1 · Hybrid search store*
</details>

<details>
<summary><b>Q:</b> After deleting a Bedrock KB backed by S3 Vectors, charges continue. Why?</summary>

Deleting the KB does not delete the vector bucket/index or IAM role. Delete KB → vector index → vector bucket → IAM role → source data (index must be deleted before bucket).

*D4 · KB cleanup*
</details>

<details>
<summary><b>Q:</b> How do you keep a Knowledge Base automatically up to date when documents change in S3?</summary>

Run incremental ingestion (sync) jobs triggered by Lambda on a schedule or event, adding an event queue to absorb bursts.

*D1 · KB data freshness*
</details>

<details>
<summary><b>Q:</b> How many data sources can a single Bedrock knowledge base have?</summary>

Up to 5 data sources.

*D1 · KB data sources*
</details>

<details>
<summary><b>Q:</b> Which knowledge base settings cannot be changed after creation?</summary>

Parsing strategy, chunking strategy, embedding model and vector store (type). Changing chunking requires recreating the data source/KB.

*D1 · KB immutable settings*
</details>

<details>
<summary><b>Q:</b> Which permission does the KB service role need to use the embedding model?</summary>

`bedrock:InvokeModel` (not `s3:GetObject`, `s3vectors:QueryVectors` or `kms:Decrypt`).

*D3 · KB role permissions*
</details>

<details>
<summary><b>Q:</b> A team wants a fully managed RAG pipeline with minimal code that returns FM answers with citations. Which Bedrock Knowledge Bases API should they call?</summary>

RetrieveAndGenerate — it embeds the query, retrieves, augments the prompt, generates with the FM and returns citations plus retrieved chunks.

*D1 · Managed RAG API*
</details>

<details>
<summary><b>Q:</b> Metadata filtering on a KB backed by OpenSearch Serverless requires which vector engine?</summary>

FAISS.

*D1 · Metadata filter engine*
</details>

<details>
<summary><b>Q:</b> KB answers mix information from different years. How do you restrict retrieval to 2019 documents?</summary>

Add a sidecar `doc.pdf.metadata.json` with `{"metadataAttributes": {"year": 2019}}`, re-sync the data source, then apply a metadata filter (e.g., equals year 2019) in `retrievalConfiguration`.

*D1 · Metadata filtering*
</details>

<details>
<summary><b>Q:</b> A KB chatbot must remember earlier turns of the conversation. What should the app pass to RetrieveAndGenerate?</summary>

A session ID (optionally with a KMS key) — Knowledge Bases provide built-in session context for multi-turn conversations.

*D2 · Multi-turn KB chat*
</details>

<details>
<summary><b>Q:</b> What are the default and maximum numberOfResults for Bedrock KB retrieval?</summary>

Default 5, maximum 100.

*D1 · numberOfResults limits*
</details>

<details>
<summary><b>Q:</b> A KB must ingest scanned PDFs containing tables. Which parsing option should be used?</summary>

FM parsing (foundation model parser) for complex structure such as PDFs, scanned images and tables.

*D1 · Parsing complex docs*
</details>

<details>
<summary><b>Q:</b> Complex multi-part questions get weak KB retrieval. What feature addresses semantic dilution?</summary>

Query decomposition (`orchestrationConfiguration`) — subqueries are retrieved separately, then pooled and ranked.

*D1 · Query decomposition*
</details>

<details>
<summary><b>Q:</b> Is a pre-existing S3 vector bucket a prerequisite for creating a KB with S3 Vectors?</summary>

No – Quick create makes the vector bucket and index. Prerequisites are an IAM role with Bedrock + S3 access and model access enabled for the embedding (and generation) model before creating the KB.

*D1 · Quick create prerequisites*
</details>

<details>
<summary><b>Q:</b> Employees need answers grounded in an internal knowledge base. What is the right approach?</summary>

RAG – build an intelligent Q&A system grounded in company knowledge (e.g. Bedrock Knowledge Bases), not training an LLM on the KB.

*D1 · RAG business use*
</details>

<details>
<summary><b>Q:</b> Retrieved chunks are relevant but poorly ordered and too many are sent to the FM. What should be added, and what is its limitation?</summary>

A Bedrock reranker model (Rerank API or `rerankingConfiguration` in Retrieve/RetrieveAndGenerate, e.g., retrieve 10 keep 5); it lowers cost and latency but supports text only.

*D1 · Reranking*
</details>

<details>
<summary><b>Q:</b> A developer needs custom re-ranking and their own prompt/LangChain chain after search. Which KB API fits and what does it return?</summary>

Retrieve — it returns contexts only (text, S3 location, metadata, score) and the app owns generation (e.g., InvokeModel or LangChain `AmazonKnowledgeBasesRetriever`).

*D1 · Retrieve vs RetrieveAndGenerate*
</details>

<details>
<summary><b>Q:</b> Which parameters are required for a RetrieveAndGenerate call?</summary>

Query text, knowledge base ID and model ARN. Optional: prompt template, numberOfResults, search type, session ID (+ KMS key) for multi-turn.

*D1 · RetrieveAndGenerate parameters*
</details>

<details>
<summary><b>Q:</b> What is the primary benefit of using Amazon S3 Vectors as a Bedrock Knowledge Base vector store?</summary>

Up to 90% lower cost for vector upload, storage and query while maintaining subsecond query performance. It is not millisecond latency and still requires an embedding model.

*D4 · S3 Vectors benefit*
</details>

<details>
<summary><b>Q:</b> What is the default encryption for S3 Vectors, and when does the KB role need KMS permissions?</summary>

Default is SSE-S3; SSE-KMS is optional. `kms:GenerateDataKey` + `kms:Decrypt` are needed only when using a customer managed key.

*D3 · S3 Vectors encryption*
</details>

<details>
<summary><b>Q:</b> What are the per-vector metadata limits in S3 Vectors?</summary>

Total metadata up to 40 KB per vector; filterable metadata max 2 KB per vector. All metadata is filterable by default unless listed in `nonFilterableMetadataKeys`.

*D1 · S3 Vectors metadata limits*
</details>

<details>
<summary><b>Q:</b> A company must keep millions of historical archive documents searchable at the lowest cost and can tolerate subsecond latency. Which KB vector store?</summary>

Amazon S3 Vectors.

*D1 · S3 Vectors use case*
</details>

<details>
<summary><b>Q:</b> A RAG app needs hybrid (keyword + semantic) search with millisecond latency at high QPS. Should it use S3 Vectors?</summary>

No – S3 Vectors supports semantic search only with subsecond latency. Use OpenSearch Serverless for hybrid search, high QPS or millisecond latency.

*D1 · S3 Vectors vs OpenSearch*
</details>

<details>
<summary><b>Q:</b> With semantic chunking at a 90th-percentile threshold over 101 sentences, how many chunks result (before max-token splits)?</summary>

11 chunks: 100 adjacent pairs, the 10 least-similar pairs become split points.

*D1 · Semantic chunking math*
</details>

<details>
<summary><b>Q:</b> Complex technical queries need enough context in each chunk. Which chunking strategy is recommended?</summary>

Semantic chunking.

*D1 · Semantic chunking use*
</details>

<details>
<summary><b>Q:</b> Documents have clear HTML, Markdown header and code-block structure. Which chunking method fits?</summary>

Structural chunking — splits on those markers; requires a well-defined schema.

*D1 · Structural chunking*
</details>

<details>
<summary><b>Q:</b> A team already runs PostgreSQL and wants to store vectors there. What should it use?</summary>

Aurora PostgreSQL or RDS for PostgreSQL with pgvector. For open-source in-memory, use FAISS or Chroma.

*D1 · Vector store choice*
</details>

<details>
<summary><b>Q:</b> What is the purpose of vectorising chunks in a RAG pipeline?</summary>

To capture the intent behind searches via semantic similarity, not just keyword matching.

*D1 · Vectorisation purpose*
</details>

---
[← Back to index](README.md)
