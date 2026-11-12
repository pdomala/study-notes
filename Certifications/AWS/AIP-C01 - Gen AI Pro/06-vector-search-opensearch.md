[← Back to index](README.md)

# 🔎 06 · Vector search with OpenSearch

> Exam domain: D1 / D4

## Key points
- ANN: **HNSW** = graph, **no training**, low latency/high recall · **IVF** = buckets, **needs training**, supports **product quantization**. Engines **FAISS / Lucene**.
- **Byte quantization** = 4 bytes → 1 byte per dimension (4x memory cut).
- **Neural sparse** = inverted index, keyword-speed semantic search · **hybrid** = normalised BM25 + k-NN scores (search pipeline) · **conversational search** = RAG + memory.
- KB on OpenSearch Serverless: metadata filtering needs **FAISS**; a BYO index needs **FAISS + matching dims + text + metadata fields**.
- Neural plugin + Bedrock connector embeds at ingest and query time. Precomputed vectors in S3 → **OpenSearch Ingestion**. Scale with sharding and multi-index per domain.

## Notes
### OpenSearch in the RAG data flow
- OpenSearch Service / Serverless = AWS vector DB option; KB chunks stored as text + **dense or sparse vectors**
- Flow: **ML connector** to Bedrock / 3rd-party model host → **neural search pipeline** ingests docs (embeds at ingest) → user query → pipeline returns **semantically similar** results → LLM prompt built with context → response
- Generate embeddings with Bedrock models through the OpenSearch **neural search pipeline** (ingest + query time)
- Alt path: generate embeddings externally → store in **S3** → load with **Amazon OpenSearch Ingestion**
- LangChain can use OpenSearch / OpenSearch Serverless as the vector store (`OpenSearchVectorSearch`) → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (LangChain)

### Vector search key terms — engines, methods, space types
- **Vector model** → distance between vectors = relevance
- **Vector embedding** → array of numbers per chunk; **embedding model** → dense multidimensional representation
- **Engines** (approximate k-NN libraries): **FAISS**, **Apache Lucene**
- **Methods**: **HNSW** (hierarchical proximity graph) · **IVF** (cluster buckets, search a subset)
- **Space types**: **Euclidean**, **Manhattan**, **cosine**
- Choose **exact k-NN** or **approximate (ANN)**

### HNSW vs IVF
| Method | How it works | Training | Notes |
| --- | --- | --- | --- |
| **HNSW** | Layered graphs; start at a random top-layer vector, descend, track k nearest at the bottom layer | **None** | Most common ANN; good latency-to-recall (low latency, high recall) |
| **IVF** | Representative vector per bucket; index into nearest bucket; search only a subset | **Requires a training step** | Supports **product quantization** (less memory) |

### Quantization & vector memory
- Embeddings are usually **4-byte floats** per dimension
- **Byte quantization** → **1 byte per dimension** (4x compression)
- **Product quantization** → reduces IVF index memory

### Search types in OpenSearch
| Type | Algorithm | Notes |
| --- | --- | --- |
| **Dense retrieval** | k-NN / ANN | Vector search (text/image/video models); **multimodal** = shared image+text embedding space |
| **Sparse retrieval** | **BM25** / TF-IDF | **Keyword** (term overlap) · **neural sparse** (expand query/docs with contextual tokens, `amazon/neural-sparse` models) |
| **Hybrid** | BM25 + k-NN | **Normalises & combines** keyword + vector scores |
| **RAG search** | BM25 or k-NN + LLM | Retrieve → prompt LLM; **conversational search = RAG + memory** |

- **Neural sparse search** → **inverted index**, as efficient as keyword search, uses **sparse embedding models**
- Search choice depends on **data nature, volume, velocity**

### Bedrock Knowledge Bases on OpenSearch Serverless
- **OpenSearch Serverless** → petabyte scale, billions of vectors, **millisecond latency**, no cluster management (S3 Vectors vs OpenSearch Serverless choice → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (Vector store selection))
- **Vector index**: Bedrock can quick-create/auto-create it, or create a **vector search collection** first, then an index with:
  - **vector field**, engine **FAISS**
  - **dimensions = the embedding model's**
  - **text field** + **metadata field**
- KB search types (Default / Hybrid / Semantic; hybrid needs a **filterable text field**) → See [05 · RAG](05-rag-knowledge-bases-chunking-embeddings.md) (RetrieveAndGenerate vs Retrieve; Advanced retrieval)
- **Metadata filtering** on an OpenSearch Serverless KB requires the **FAISS engine**

## Exam triggers
- "ANN with no training, low latency, high recall" → **HNSW** · "bucketed search, reduce memory with PQ" → **IVF** (needs training)
- "Compress vectors 4× in OpenSearch" → **byte quantization** · "reduce IVF memory" → **product quantization**
- "Keyword-efficient semantic search via inverted index" → **neural sparse search**
- "Chat over docs with memory" → **conversational search** (RAG + memory)
- "Generate embeddings inside OpenSearch via Bedrock" → **neural search pipeline + ML connector**
- "Pre-computed embeddings in S3 → OpenSearch" → **OpenSearch Ingestion**
- "Bring-your-own OSS index for KB" → **FAISS**, dimensions match model, text + metadata fields
- "KB hybrid search falls back to semantic" → OSS index needs a **filterable text field**
- "Metadata filtering on an OpenSearch Serverless KB" → **FAISS engine**
- "Billions of vectors, millisecond latency, no cluster management" → **OpenSearch Serverless**

## Flashcards
<details>
<summary><b>Q:</b> What must a pre-created OpenSearch Serverless index contain to back a Bedrock KB?</summary>

A vector field using the FAISS engine with dimensions matching the embedding model, plus a text field and a metadata field.

*D1 · BYO OSS index for KB*
</details>

<details>
<summary><b>Q:</b> In OpenSearch terms, what is conversational search?</summary>

RAG search plus memory — retrieve, prompt the LLM, and retain conversation history.

*D2 · Conversational search*
</details>

<details>
<summary><b>Q:</b> How can OpenSearch generate embeddings with a Bedrock model at ingest and query time?</summary>

Configure an ML connector to Bedrock and a neural search pipeline that ingests docs and returns semantically similar results.

*D1 · Embeddings inside OpenSearch*
</details>

<details>
<summary><b>Q:</b> An OpenSearch vector index needs ANN with no training step, low latency and high recall. Which method?</summary>

HNSW (hierarchical proximity graph). IVF uses buckets and requires a training step.

*D1 · HNSW vs IVF*
</details>

<details>
<summary><b>Q:</b> How does OpenSearch hybrid search combine results?</summary>

It normalises and combines BM25 keyword scores with k-NN vector scores.

*D1 · Hybrid search mechanics*
</details>

<details>
<summary><b>Q:</b> What does hybrid search in a Bedrock KB require, and what happens otherwise?</summary>

OpenSearch Serverless with a filterable text field; otherwise it falls back to semantic search.

*D1 · Hybrid search requirement*
</details>

<details>
<summary><b>Q:</b> Embeddings were generated externally and stored in S3. Which service loads them into OpenSearch?</summary>

Amazon OpenSearch Ingestion.

*D1 · Loading precomputed vectors*
</details>

<details>
<summary><b>Q:</b> A team wants semantic-aware search as efficient as keyword search using an inverted index. What should they use?</summary>

Neural sparse search, which uses sparse embedding models (e.g., `amazon/neural-sparse`) that expand queries/docs with contextual tokens.

*D1 · Neural sparse search*
</details>

<details>
<summary><b>Q:</b> Name the approximate k-NN engines and space types available in OpenSearch.</summary>

Engines: FAISS and Apache Lucene. Space types: Euclidean, Manhattan, cosine.

*D1 · OpenSearch engines*
</details>

<details>
<summary><b>Q:</b> How do you reduce memory for an IVF index vs compress embeddings 4x in OpenSearch?</summary>

IVF supports product quantization to reduce memory; byte quantization stores 1 byte per dimension instead of 4-byte floats (4x compression).

*D4 · Vector memory reduction*
</details>

---
[← Back to index](README.md)
