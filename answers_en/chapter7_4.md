# 7.4 Vector Databases and Indexes — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 14 questions

**1. What Is a Vector Database? What Are Common Vector Databases?**

<details>
<summary>💡 Show Answer</summary>

A vector database is a database specifically designed to store, index, and retrieve high-dimensional vectors. It supports Approximate Nearest Neighbor (ANN) search, capable of returning the most similar results among millions of vectors in milliseconds. Common vector databases:
1. **Cloud-Managed**: Pinecone (fully managed, user-friendly), Weaviate (supports hybrid retrieval), Qdrant (high performance, open source);
2. **Open-Source Self-Hosted**: Milvus (large-scale production-grade), Chroma (lightweight, easy to integrate), FAISS (Facebook open-source library, highly efficient underlying implementation);
3. **Traditional Database Extensions**: pgvector (PostgreSQL extension), Elasticsearch vector search.
Selection criteria: Data scale, latency requirements, MLOps/operational capabilities, and hybrid retrieval needs.

</details>

**2. What Are the Differences Among Vector Indexes, Vector Databases, and Vector Plugins?**

<details>
<summary>💡 Show Answer</summary>

The three represent a progressive hierarchical relationship, serving different usage scenarios and scale requirements:

**Vector Index**:
- The lowest-level data structure, focusing on vector storage and Approximate Nearest Neighbor (ANN) search algorithms themselves;
- Representative implementations: FAISS (Facebook AI Similarity Search), ScaNN, Annoy;
- Features: No persistent storage, no metadata management, no access control—purely a vector computation library;
- Applicable scenarios: Embedded into applications as a low-level computation module, or small-scale in-memory vector search.

**Vector Database**:
- A complete database system built on top of vector indexes, providing enterprise-grade capabilities such as persistent storage, CRUD operations, metadata filtering, access control, and high availability;
- Representative products: Pinecone, Qdrant, Weaviate, Milvus, Chroma;
- Features: Supports joint queries of vectors and structured metadata, horizontal scaling, backup and recovery, monitoring and alerting;
- Applicable scenarios: Production-grade RAG systems, scenarios requiring persistence, high concurrency, and operations/maintenance capabilities.

**Vector Plugin**:
- Plugins that extend vector search capabilities on existing traditional databases (PostgreSQL, Elasticsearch, Redis);
- Representative implementations: pgvector (PostgreSQL), Elasticsearch kNN, Redis VSS;
- Features: Reuses existing infrastructure, stores vectors and business data in the same database, with low maintenance costs;
- Limitations: Vector retrieval performance is usually inferior to dedicated vector databases, with limited scalability in large-scale scenarios;
- Applicable scenarios: Teams with existing PostgreSQL/Elasticsearch infrastructure, medium data volumes ($<$ one million vectors), and a desire to quickly integrate vector search.

**Selection Advice**: Rapid prototyping → Chroma or FAISS; small-to-medium production → pgvector or Qdrant; large-scale production with dedicated O&M capabilities → Milvus or Pinecone.





![Key architectural differences between vector indexes, databases, and plugins. (向量索引、向量数据库与向量插件的架构差异对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig07_vector_storage.png)

</details>

**3. What Are the Types of Filtering and Their Challenges in Vector Databases?**

<details>
<summary>💡 Show Answer</summary>

Filtering in vector databases mainly consists of two fundamental methods, each with its own pros, cons, and applicable scenarios:

**1. Pre-filtering**: Before executing vector similarity search, the candidate set is first narrowed down based on metadata constraints, performing ANN search only on the subset that meets the criteria.
- **Pros**: Reduces the candidate scale for vector search and lowers computational costs; precisely satisfies filtering criteria without returning non-matching results;
- **Cons**: When filtering conditions are extremely strict (the candidate set is extremely small), the performance advantage of the ANN index vanishes, potentially degrading into a brute-force scan; when filtering conditions are complex, building and querying metadata indexes also incur additional overhead.

**2. Post-filtering**: Vector search is executed first to obtain top-$k$ results, and metadata filtering conditions are then applied to the results, keeping only those that meet the criteria.
- **Pros**: Fully utilizes the efficiency of the ANN index with no additional cost during the vector search phase;
- **Cons**: If filtering conditions are relatively strict, the number of filtered results may be far fewer than $k$, requiring multiple rounds of expanding $k$ and re-retrieving; in the worst case, all top-$k$ results are filtered out, necessitating a full scan, which is extremely inefficient.

**3. Hybrid Filtering (In-line Filtering)**: Metadata filtering is executed concurrently alongside vector search, dynamically excluding non-compliant vectors during ANN traversal via SIMD instructions or bitmap indexes. This is the mainstream implementation in modern vector databases such as Qdrant and Weaviate, balancing accuracy and performance.

**Core Challenges**:
1. **Conflict between filtering selectivity and ANN precision**: The stricter the filtering conditions, the smaller the candidate set, and the larger the approximation error of ANN;
2. **Index maintenance for dynamic filtering conditions**: Frequently changing metadata fields such as permissions and status require efficient incremental index updating mechanisms;
3. **Performance of multi-field composite filtering**: Combinations of multiple filtering conditions (AND/OR) significantly increase index optimization complexity.

</details>

**4. What Are "Retrieval (Recall)" and "Reranking" in RAG?**

<details>
<summary>💡 Show Answer</summary>

**Retrieval (Recall)**: The first-stage retrieval that quickly finds top-$k$ candidate documents (typically $k=20--100$) from the full vector database using approximate nearest neighbor algorithms. It is fast but has slightly lower precision, with the goal of minimizing missing relevant documents (high recall rate).

**Reranking**: The second stage re-ranks the retrieved results using a more precise but slower model (Cross-Encoder), calculating the exact relevance score between the query and each candidate document, and then selecting top-$n$ ($n < k$) after re-sorting.

Two-stage design: Retrieval trades precision for speed, while reranking trades speed for precision. Combined, they balance efficiency and quality.





![Two-stage retrieval process: coarse dense passage retrieval followed by fine reranking. (双阶段检索流程：粗筛密集检索与精筛重排)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig08_retrieval_and_reranking.png)

</details>

**5. What Is Top-$k$ Retrieval, and How Does the Value of $k$ Affect RAG Performance?**

<details>
<summary>💡 Show Answer</summary>

Top-$k$ retrieval: Returning the $k$ document chunks most similar to the query vector from the vector database. The impact of the $k$ value:
1. **$k$ is too small**: May miss truly relevant documents (low recall), leading to incomplete answers;
2. **$k$ is too large**: Introduces more irrelevant documents, increases noise, exceeds the LLM context window, and worsens the "lost in the middle" effect;
3. **Typical values**: $k=20--50$ in the retrieval stage, and $n=3--10$ passed to the LLM after reranking;
4. **Dynamic $k$**: Adjusted according to the task type: small $k$ for precise Q&A, large $k$ for comprehensive analysis;
5. **Threshold filtering**: Documents with similarity below a threshold are filtered out even if they fall within top-$k$, ensuring a minimum relevance.

</details>

**6. Why Does Increasing Top-$K$ Sometimes Degrade Overall Performance?**

<details>
<summary>💡 Show Answer</summary>

A larger Top-$K$ introduces more low-relevance or even contradictory content, increasing the model's burden in filtering information and making it easier to trigger "Lost in the Middle", thereby diluting the weight of highly relevant content. Detailed mechanism analysis:

1. **Decline in signal-to-noise ratio**: Retrieval results from vector search are ranked by similarity. Top-ranked chunks have high relevance, but as $K$ increases, newly added chunks have progressively lower relevance. When $K$ increases from 5 to 20, the last 15 chunks are mostly marginally relevant or even irrelevant, introducing a large amount of noise into the context.
2. **Lost in the Middle effect**: Studies show that LLMs pay significantly less attention to content placed in the middle of the context than to content at the beginning or end. As $K$ grows larger and the context gets longer, the probability of ignoring truly useful chunks increases, even if they rank highly in top-$k$.
3. **Increased contradictory information**: When $K$ increases, documents from different time versions or sources are retrieved, which may contain mutually contradictory statements (e.g., old version behavior vs. new version behavior of a feature). LLMs are more prone to generating incorrect or mixed answers when facing contradictory content.
4. **Increased LLM reasoning burden**: Synthesizing reasoning from 20 chunks is far more difficult than from 5 chunks. LLMs are more likely to "get lost" in complex contexts, generating answers that are superficially fluent but actually taken out of context.
5. **Context window pressure**: An excessively large $K$ may hit the LLM's context window limit, causing chunks at the end to be truncated and resulting in partial information loss.

**Best Practice**: Increase $K$ to expand the retrieval candidate pool, but do not pass all of them directly to the LLM; combine with Reranking to select only the top-3 to top-5 most relevant chunks to pass to the generation stage, maintaining high information density.

</details>

**7. What Is Semantic Similarity, and How Should One Choose Between Cosine Similarity and Dot Product in Vector Retrieval?**

<details>
<summary>💡 Show Answer</summary>

**Semantic Similarity**: Measures the closeness of two text segments at the semantic level, represented by the distance/angle between their corresponding embedding vectors.

**Cosine Similarity**: Calculates the cosine of the angle between two vectors, considering only direction and not magnitude, with a range of $[-1, 1]$. It is suitable for cases where vector lengths are inconsistent (e.g., embeddings of texts with different lengths).

**Dot Product**: Considers both direction and magnitude, and is faster (one step less normalization). It is suitable for vectors normalized via $L_2$ (when magnitudes are identical, dot product equals cosine similarity).

Selection Advice: Most Embedding models output normalized vectors, making dot product more efficient; if vectors are not normalized, use cosine similarity; FAISS natively supports Inner Product (IP), which is equivalent to cosine similarity after normalization.

</details>

**8. What Problems Are Caused by "Context Window" Limitations in RAG?**

<details>
<summary>💡 Show Answer</summary>

1. **Limited number of retrieved contents**: The number of chunks that can be passed to the LLM has an upper limit, so the $k$ value cannot be infinitely large;
2. **Lost in the Middle**: When too many chunks are passed in, the LLM's attention to content in the middle position decreases, potentially ignoring important information;
3. **Increased costs**: The more chunks passed in, the higher the token consumption per request;
4. **Conflict between quality and recall rate**: Aiming for a high recall rate requires a large $k$, but a large $k$ exceeds context limits;
5. Mitigation strategies: Pass only a small number of the most relevant chunks after Reranking, use Contextual Compression to compress each chunk, or process in chunks and summarize (Map-Reduce).

</details>

**9. How Is Semantic Search Implemented in RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

Semantic search is the core technology of the RAG retrieval component, enabling the system to understand the semantic meaning of queries and documents rather than relying solely on keyword matching. The complete implementation workflow of semantic search in RAG is as follows:

**1. Offline Indexing Construction (Document Side)**:
1. Use pre-trained semantic Embedding models (e.g., BERT, Sentence-BERT, BGE, E5) to encode document chunks into high-dimensional dense vectors (typically 768--1536 dimensions);
2. Each vector captures the semantic information of the corresponding text, where semantically similar texts are closer together in the vector space;
3. Store all document vectors into a vector database (FAISS, Qdrant, Milvus) and build ANN indexes (e.g., HNSW) to support efficient retrieval.

**2. Online Query Processing (Query Side)**:
1. Use the **exact same** Embedding model as in the indexing stage to encode the user query into a query vector;
2. Execute Approximate Nearest Neighbor search (ANN Search) in the vector database to compute the similarity between the query vector and all document vectors (typically using cosine similarity or inner product);
3. Return the top-$k$ document chunks with the highest similarity, which are the most semantically relevant to the query.

**3. Key Advantages of Semantic Search**:
- **Synonym understanding**: "Car" and "automobile" have close vector distances, eliminating the need for exact keyword matching;
- **Cross-lingual retrieval**: Multilingual Embedding models allow Chinese queries to retrieve English documents;
- **Intent understanding**: Capable of understanding the underlying intent behind the query rather than just surface-level words;
- **Handling spelling variations**: Displays tolerance to minor typos and abbreviations.

**4. Common Tools and Frameworks**: LangChain and LlamaIndex encapsulate complete semantic search pipelines; FAISS provides efficient low-level ANN computation; the Sentence-Transformers library offers a rich selection of pre-trained Embedding models.

</details>

**10. What Is Dense Passage Retrieval (DPR), and How Is It Used in RAG?**

<details>
<summary>💡 Show Answer</summary>

DPR (Dense Passage Retrieval, Karpukhin et al. 2020) is an open-domain question answering retrieval method proposed by Facebook AI Research, serving as one of the foundational works in modern vector retrieval technology.

**Technical Principles**: DPR adopts a Bi-encoder architecture:
- **Question Encoder**: Encodes the user's question into a dense vector;
- **Passage Encoder**: Encodes document passages into dense vectors;
- Both encoders are pre-trained based on BERT and jointly fine-tuned via contrastive learning (positive/negative sample pairs), making question vectors closer to relevant passage vectors in the vector space;
- During retrieval, Maximum Inner Product Search (MIPS) is used to quickly find the passages most similar to the question.

**Role of DPR in RAG**:
1. DPR was the retrieval component used in the original RAG paper (Lewis et al. 2020), demonstrating the feasibility of dense retrieval + generation and laying the technical foundation for modern RAG;
2. Compared to traditional BM25, DPR understands semantic similarity (e.g., "car" and "automobile" are treated as having the same meaning), achieving significantly better Recall@$k$ in open-domain Q&A than BM25;
3. DPR is the predecessor of modern Embedding models (BGE, E5, GTE). Current RAG systems generally use stronger pre-trained Embedding models in place of original DPR, but the core Bi-encoder paradigm has been fully preserved.

**Modern Developments**: Following DPR, stronger Embedding models such as BGE, E5, GTE, and Cohere Embed have emerged sequentially, significantly outperforming DPR on most benchmarks. In actual production, original DPR models are rarely used directly today, but their technical philosophy remains the foundation of all vector retrieval.

</details>

**11. What Role Do Embeddings Play in RAG?**

<details>
<summary>💡 Show Answer</summary>

Embeddings serve as a core bridge in RAG systems, converting discrete text into continuous high-dimensional vectors so that computers can perform similarity comparisons at the semantic level. Its role in RAG is reflected in the following dimensions:

1. **Semantic representation**: Embeddings represent text data as points in a high-dimensional vector space, where semantically close texts are nearer to each other, thereby supporting cross-expression retrieval with "same meaning, different phrasing."
2. **Offline indexing**: During the knowledge base construction phase, each document chunk is encoded by the embedding model, and its vector is stored into a vector database to form an efficiently searchable index structure.
3. **Online querying**: When a user asks a question, the same embedding model encodes the query into a vector and performs approximate nearest neighbor search in the vector database to find the semantically closest chunks.
4. **Cross-lingual bridging**: Multilingual embedding models (e.g., mE5, BGE-M3) can map texts in different languages into a shared vector space, supporting cross-lingual scenarios such as retrieving English documents using Chinese queries.
5. **Determining the upper quality bound**: The quality of the embedding model directly determines the semantic matching capabilities of retrieval, serving as one of the performance ceilings of the entire RAG system—if the embedding model is poor, all subsequent optimizations are merely remediations built on a flawed foundation.

In short, embeddings are the core technology in RAG for "understanding semantics and bridging retrieval and generation." Without high-quality embeddings, RAG degrades into a simple keyword-matching system.





![Role of embeddings in semantic vector similarity mapping between queries and documents. (嵌入表示在Query与文档语义向量相似度匹配中的作用)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig09_embedding_similarity.png)

</details>

**12. What Types of Generative Models Are Used in RAG?**

<details>
<summary>💡 Show Answer</summary>

The generative component of RAG is responsible for receiving the retrieved context and user query to generate the final natural language answer. Common generative model types and their applicable scenarios include:

**1. Large Pre-trained Language Models (LLMs) — The Mainstream Solution**
- **GPT Series** (GPT-4o, GPT-4 Turbo): OpenAI's closed-source models with strong overall capabilities and convenient API access, suitable for rapid deployment, though data must be sent to third-party servers;
- **Claude Series** (Claude 3.5 Sonnet/Haiku): Anthropic's closed-source models, excelling in instruction following and long-context comprehension, suitable for RAG scenarios requiring precise source citation;
- **Gemini Series** (Google): Ultra-long context support (1M tokens), suitable for scenarios needing integration of vast amounts of retrieved content.

**2. Open-Source Locally Deployable LLMs — The Preferred Choice for Privacy Scenarios**
- **LLaMA Series** (Meta LLaMA 3): The most widely used open-source base models, deployable locally with data remaining on-premise, ideal for enterprises with strict data privacy requirements;
- **Mistral/Mixtral**: European open-source LLMs with high inference efficiency, suitable for resource-constrained deployment environments;
- **Qwen Series** (Alibaba), **GLM Series** (Zhipu): Strong Chinese comprehension capabilities, suitable for Chinese-centric RAG systems;
- **Phi-3** (Microsoft): Small parameter count (3.8B) yet excellent performance, suitable for edge deployment or low-latency scenarios.

**3. Sequence-to-Sequence Models (Seq2Seq) — Lightweight Task-Specific Scenarios**
- **T5/FLAN-T5**: Based on the Encoder-Decoder architecture, suitable for specific NLP tasks such as summarization and question answering. They feature small parameter counts and low inference costs, but their overall capabilities are weaker than those of large LLMs;
- **BART**: Also an Encoder-Decoder model, specializing in text summarization tasks and usable as a context compression component in RAG.

**4. Domain-Specific Models — Vertical Scenarios**
- Medical domain: MedPaLM, BioMedLM;
- Legal domain: LegalBERT series;
- Code generation: CodeLlama, DeepSeek-Coder.

**Selection Advice**: Prioritize GPT-4o or Claude for production environments (rapid and high quality); use locally deployed LLaMA-3 or Qwen for privacy-sensitive scenarios; consider small models like Phi-3 or Mistral for resource-constrained or high-QPS scenarios.

</details>

**13. How Does RAG Manage Retrieval Across Extremely Large Datasets?**

<details>
<summary>💡 Show Answer</summary>

- **Sharding**: Partitioning the corpus into smaller, more manageable subsets, where each shard builds its index independently, and queries are executed in parallel across all shards and merged during retrieval.
- **Approximate Nearest Neighbor (ANN)**: Using ANN algorithms such as FAISS (IVF-PQ) and HNSW for faster similarity search, trading a minor loss of precision for order-of-magnitude speedups, reducing complexity from linear scanning $O(n)$ to approximately $O(\log n)$.
- **Distributed Indexing**: Leveraging vector databases that support distributed deployment (e.g., Milvus, Weaviate) to distribute index and retrieval calculations across multiple nodes, horizontally scaling storage and query throughput.

**Advanced Strategies for Extremely Large Datasets**:
1. **Hierarchical Retrieval**: First use a coarse-grained index (document-level summary vectors) to quickly narrow down the candidate range, and then perform fine-grained retrieval within the candidate subset, reducing the full-database retrieval scope from millions to thousands, significantly lowering computational costs;
2. **Vector Compression (Product Quantization)**: Compress float32 vectors (4 bytes/dimension) into smaller quantized codes (1--2 bytes/dimension), reducing memory footprint by $4--8\times$, enabling a single node to support larger-scale in-memory indexes;
3. **Hot/Cold Tiered Storage**: Vectors of frequently accessed documents are kept in in-memory indexes (millisecond-level retrieval), while vectors of low-frequency documents are stored in disk indexes (hundreds of milliseconds retrieval), dynamically adjusted based on access frequency to balance cost and performance;
4. **Metadata Pre-filtering**: Utilize metadata (document type, time range, department) prior to retrieval to narrow down the candidate set, avoiding vector searches over the entire dataset.





![Hierarchical retrieval architecture for extremely large document collections. (海量文档集的层级化检索存储架构)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig10_large_scale_retrieval.png)

</details>

**14. How Can Retrieval and Generation in RAG Be Parallelized?**

<details>
<summary>💡 Show Answer</summary>

**1. Parallelization within Retrieval**:
1. **Parallel execution of multi-path retrieval**: Multiple retrieval methods such as vector retrieval, BM25 keyword retrieval, and graph retrieval can be initiated simultaneously, rather than waiting sequentially for each retrieval method to complete before starting the next, finally merging all parallel retrieval results;
2. **Parallel querying of multiple knowledge bases**: When the RAG system connects to multiple knowledge bases, retrieval requests can be issued to all knowledge bases simultaneously, setting the response time ceiling to the slowest knowledge base rather than the sum of all knowledge base response times;
3. **Parallel retrieval after query rewriting**: After Multi-Query RAG generates multiple query variants, it initiates multi-path retrieval in parallel rather than processing them sequentially one by one.

**2. Parallelization of Retrieval with Other Steps**:
1. **Parallelizing query processing and retrieval**: Preprocessing steps such as query rewriting and intent classification can be executed in parallel while initiating vector retrieval (provided they are not mutually dependent);
2. **Retrieval and context compression pipelining**: Context compression can be initiated immediately after retrieval is complete, while chunks whose relevance has already been determined can start Reranking scoring early, forming a parallel pipeline;
3. **Prefetching**: For predictable subsequent queries (such as pagination scenarios), content that might be needed for the next page can be pre-fetched while the user views current results.

**3. Batch Processing Parallelization (For High-QPS Scenarios)**:
1. **Batch Embedding computation**: Combine query embeddings from multiple concurrent users into a batch and process multiple queries in a single GPU inference pass, achieving a 5--10$\times$ efficiency improvement over processing them individually;
2. **Batch Reranking**: Similarly, candidate documents from multiple users are combined for batch scoring;
3. **Batch LLM inference**: For offline or asynchronous scenarios, group multiple generation requests into a batch to send to the LLM, leveraging KV Cache sharing of the model to increase throughput.

**Engineering Implementation**: In Python, `asyncio` + `aiohttp` are used to implement asynchronous concurrent retrieval; multi-path retrieval results are aggregated using `asyncio.gather()` to achieve genuine parallelism rather than mere concurrency.

%% ============================================================
%% 5. 检索策略（混合检索、Rerank、MMR）
%% ============================================================

</details>

