# 7.3 Embedding Models — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 14 questions

**1. What Is an Embedding? What Role Does It Play in RAG?**

<details>
<summary>💡 Show Answer</summary>

An embedding is the process of mapping text (or other modalities) into a high-dimensional dense vector space such that semantically similar texts are closer together in the vector space. Its roles in RAG include:
1. **Document Indexing**: In the offline phase, each chunk is converted into a vector and stored in a vector database to form a searchable index;
2. **Query Vectorization**: In the online phase, the user query is converted into a vector in the same space;
3. **Semantic Matching**: Find the most semantically relevant documents by computing the similarity between the query vector and document vectors;
4. **Language Independence**: A good multilingual embedding model enables cross-lingual retrieval (e.g., querying English documents with a Chinese query).
The quality of the embedding model directly determines the upper bound of retrieval performance.

</details>

**2. How Are Embedding Models Used in LLM Applications?**

<details>
<summary>💡 Show Answer</summary>

Embedding models in LLM applications act like a search engine, helping the LLM locate the most relevant information snippets from large volumes of data. This helps the LLM provide more accurate, informative, and helpful answers.

Imagine an LLM as a powerful language expert that needs a mechanism to understand the meaning behind user queries. The embedding model steps in to convert text inputs (queries, documents) into numerical representations called embeddings. These embeddings capture semantic relationships between words and concepts.

LLM applications commonly leverage Retrieval-Augmented Generation (RAG). In RAG, the embedding model transforms both the user query and relevant information in the knowledge base into embeddings. It then finds the information embeddings in the knowledge base that are most similar to the user query. This retrieved information, along with the user query, serves as context for the LLM. Finally, the LLM utilizes this context to generate answers that are both relevant and informative.

</details>

**3. How to Select an Embedding Model for a RAG System? What Are the Key Considerations?**

<details>
<summary>💡 Show Answer</summary>

Key considerations:
1. **Retrieval Quality**: Metrics such as NDCG and MRR on representative tasks, referencing the MTEB leaderboard;
2. **Domain Adaptability**: General-purpose models vs. domain-specific models (legal, medical, code, etc.);
3. **Language Support**: Whether target languages are supported (Chinese, multilingual);
4. **Vector Dimensions**: Higher dimensions generally yield higher precision but incur larger storage and computational costs (typically 768 or 1024);
5. **Maximum Input Length**: The maximum number of tokens the model can process, which affects the chunking strategy;
6. **Inference Speed**: In high QPS scenarios, embedding computation latency must be considered;
7. **Deployment Options**: On-premise deployment vs. API calls (privacy, cost, stability);
8. **Cost**: API pricing or hardware costs for local deployment.

</details>

**4. How to Choose a Suitable Embedding Model? What Metrics Are Used to Evaluate an Embedding Model?**

<details>
<summary>💡 Show Answer</summary>

**How to Choose a Suitable Embedding Model**:
1. **Consult Public Leaderboards**: MTEB (Massive Text Embedding Benchmark) is currently the most authoritative and comprehensive benchmark for evaluating embedding models. You can directly inspect the MTEB leaderboard to select models with high scores on Retrieval tasks. C-MTEB is a benchmark specifically tailored for Chinese.
2. **Consider Specific Application Scenarios**: Domain specificity (domain-specific vs. general-purpose models), language support (especially multilingual scenarios), model size and speed (trade-offs between performance and speed).
3. **Proprietary vs. Open-Source Models**: Proprietary models (such as OpenAI's Ada series) offer strong performance but pose privacy risks and higher costs; open-source models (such as BGE, M3E, Jina-embeddings, etc.) can be deployed locally with controllable data security, but require self-managed deployment and maintenance.

**Metrics for Evaluating Embedding Models** (mainly from the MTEB benchmark):
1. **Retrieval**: nDCG@$k$ (comprehensively measures relevance and ranking, most critical); Recall@$k$ (recall rate); MRR (Mean Reciprocal Rank, position of the first relevant document).
2. **Semantic Textual Similarity (STS)**: Spearman or Pearson correlation coefficient, measuring the consistency between calculated cosine similarity and human judgment.
3. **Classification**: Accuracy, obtained by training a classifier using embedding vectors as features.
4. **Clustering**: V-measure, measuring whether the model can naturally group semantically similar texts together.

</details>

**5. What Are the Main Differences Between Open-Source Embedding Models (e.g., BGE, E5) and Commercial Models (e.g., OpenAI ada)?**

<details>
<summary>💡 Show Answer</summary>

**Open-Source Models (BGE, E5, GTE)**:
1. On-premise deployment, keeping data within local domain, suitable for privacy-sensitive scenarios;
2. One-time hardware cost, offering lower per-query costs under heavy usage;
3. Fine-tunable to adapt to specific domains;
4. Strong Chinese support (e.g., the BGE series is optimized for Chinese);
5. Requires self-maintenance of inference services.

**Commercial Models (OpenAI text-embedding-3, Cohere embed)**:
1. API-based calls with no deployment overhead;
2. Pay-as-you-go pricing, making costs low for light usage;
3. Continuously updated models;
4. Typically superior performance in English;
5. Data is sent to third-party servers, posing privacy risks.
Selection advice: Privacy-sensitive $\rightarrow$ Open-source; Quick prototyping $\rightarrow$ Commercial APIs; Large-scale production with MLOps capabilities $\rightarrow$ Open-source.





![Feature comparison matrix of open-source and commercial embedding models. (开源与商用嵌入模型的特点对比矩阵)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig06_embedding_comparison.png)

</details>

**6. What Is Embedding Dimensionality? How Does Dimensionality Affect Retrieval Performance and Speed?**

<details>
<summary>💡 Show Answer</summary>

Embedding dimensionality refers to the number of elements in a vector (e.g., 768, 1024, 1536). Effects:
1. **Precision vs. Dimensionality**: Higher dimensions generally offer stronger expressiveness, capturing finer semantic nuances and achieving higher retrieval accuracy; however, returns diminish beyond a certain dimension;
2. **Storage Cost**: Doubling the dimension doubles the vector storage space; 1 million vectors $\times$ 1024-dimensional float32 $\approx$ 4\,GB;
3. **Retrieval Speed**: Higher dimensions increase ANN search computational complexity, leading to higher latency;
4. **Dimensionality Reduction**: Matryoshka representations allow truncating vector dimensions (e.g., from 1536 down to 256) with limited accuracy loss, useful for low-latency scenarios.

</details>

**7. What Is a Late Interaction Model (e.g., ColBERT)? How Does It Differ from Standard Dual-Tower Models?**

<details>
<summary>💡 Show Answer</summary>

ColBERT (Contextualized Late Interaction over BERT): Queries and documents are separately encoded by BERT into sequences of token-level vectors. During retrieval, token-level maximum similarity calculations (MaxSim) are performed instead of computing a single document-level vector similarity. Differences from standard dual-tower models:
1. **Granularity**: ColBERT preserves token-level vectors (interactions between every query token and document token), whereas dual-tower models produce only a single document-level vector;
2. **Accuracy**: ColBERT's accuracy lies between dual-tower models and Cross-Encoders, performing better than dual-tower models;
3. **Speed**: Faster than Cross-Encoders (document vectors can be precomputed) but slower than dual-tower models (requires MaxSim computation);
4. **Storage**: ColBERT stores token-level vectors, requiring significantly more storage than dual-tower models (hundreds of vectors per document vs. 1 vector).

</details>

**8. What Should Be Done When Embedding Models Fail in Certain Scenarios?**

<details>
<summary>💡 Show Answer</summary>

You can introduce domain-finetuned embedding models, combine keyword search (BM25) to perform hybrid retrieval as a supplement, or add rule-based/structured retrieval paths for specific query types. Comprehensive mitigation strategies:

1. **Diagnose Failure Causes**: First analyze the types of failing queries—is it a failure to understand domain jargon, special formats (code/formulas/acronyms), or cross-lingual issues? Different root causes correspond to different solutions.
2. **Hybrid Retrieval Remediation**: Combine with BM25 keyword search, which provides significant remediation for exact lexical match failures (where embedding models fail to distinguish proper nouns) at low implementation cost without altering the embedding model.
3. **Domain-Specific Embedding Models**: Replace general-purpose models with domain-specific embedding models for specialized domains (code, medical, legal), such as using CodeBERT for code retrieval or BiomedBERT for medical documents.
4. **Lightweight Adapter Fine-Tuning**: Add an Adapter to the output layer of the existing embedding model and fine-tune the Adapter using a small amount of domain data (question-document pairs), aligning the vector space closer to the target domain at a fraction of full fine-tuning costs.
5. **Query Rewriting/Expansion**: For specific types of failing queries, rewrite them before retrieval (expanding acronyms, converting colloquial expressions to formal terms) to indirectly improve embedding matching.
6. **Rule/Structured Retrieval Paths**: For known special query types (e.g., "search product ID X-1234"), route directly to exact matching paths, bypassing embedding retrieval.
7. **Introduce Reranker as Fallback**: Even if embedding recall is imprecise, a Reranker (Cross-Encoder) can evaluate relevance more accurately among top-$k$ candidates, compensating for recall limitations.

</details>

**9. What Is Fine-Tuned Embedding? In What Scenarios Is Fine-Tuning Needed?**

<details>
<summary>💡 Show Answer</summary>

Fine-tuned Embedding: Fine-tuning a pre-trained embedding model on domain- or task-specific data to better understand domain jargon and query patterns. Scenarios requiring fine-tuning:
1. **Strong Domain Specialization**: Domains such as medical, legal, and financial contain vast domain jargon that general models fail to understand correctly;
2. **Poor Retrieval Quality with General Models**: Low Hit Rate on domain evaluation datasets indicates that general models are unsuitable;
3. **Specialized Query Patterns**: The query style of the target system differs significantly from general training data (e.g., code search);
4. **Sufficient Training Data Available**: Requires question-document pairs or positive/negative sample pairs (typically thousands to tens of thousands of pairs);
5. If the performance of general models is already acceptable, the marginal gains of fine-tuning may not justify the extra cost.

</details>

**10. How to Evaluate the Retrieval Quality of an Embedding Model in a Specific Domain?**

<details>
<summary>💡 Show Answer</summary>

1. **Construct a Domain Evaluation Set**: Create question-answer pairs from target domain documents, annotating each question with the document/paragraph containing the correct answer (golden document);
2. **Hit Rate@$k$**: The proportion of top-$k$ retrieved results containing the golden document using the embedding model;
3. **MRR (Mean Reciprocal Rank)**: The mean of the reciprocal ranks of the golden documents, measuring precise ranking ability;
4. **NDCG@$k$**: A comprehensive ranking quality metric considering graded relevance;
5. **Baseline Comparison**: Compare against currently used models (or general baselines) to calculate relative performance gains;
6. **End-to-End Quality**: Fix the generation phase and compare the final answer quality under different embedding models (e.g., RAGAS scores);
7. Reference: MTEB (Massive Text Embedding Benchmark) is a widely recognized benchmark for multi-task evaluation.

</details>

**11. How to Design a Domain Adaptation Scheme for an Embedding Model (Without Full Fine-Tuning)?**

<details>
<summary>💡 Show Answer</summary>

Lightweight domain adaptation strategies:
1. **Prompt Engineering**: Some embedding models support instruction prefixes (e.g., the E5 series), guiding the embedding space by prepending a task description to the query;
2. **Domain Dictionary Enhancement**: Construct a domain synonym dictionary to perform synonym expansion on queries during retrieval, supplementing keyword search;
3. **Lightweight Adapter**: Add a small Adapter network (linear layer or small MLP) to the output layer of the embedding model, training only the Adapter while keeping the backbone frozen, resulting in extremely low training costs;
4. **Hybrid Retrieval**: Supplement with BM25 keyword search to offset vector retrieval limitations on domain-specific terminology;
5. **Knowledge Injection**: Prepend domain term explanations to each chunk during chunking so that embeddings encapsulate richer domain-specific information.

</details>

**12. What Are Common Multilingual Embedding Models? What Are the Challenges of Cross-Lingual Retrieval?**

<details>
<summary>💡 Show Answer</summary>

**Common Multilingual Embedding Models**: multilingual-e5-large (Microsoft, supporting 100+ languages), mGTE (Alibaba), BGE-M3 (supporting 100+ languages, BAAI), Cohere embed-multilingual (commercial API), LaBSE (Google, 109 languages).

**Challenges of Cross-Lingual Retrieval**:
1. **Language Misalignment**: Identical meanings in different languages may not be close in vector space, requiring strong cross-lingual alignment capabilities from the model;
2. **Low-Resource Languages**: Poor performance for languages with scarce training data, leading to unbalanced model capability;
3. **Cultural Nuances**: The same concept may have subtle differences in expression and meaning across languages and cultures;
4. **Evaluation Difficulty**: Evaluating cross-lingual retrieval requires multilingual annotated datasets, which are expensive to construct;
5. **Tokenization Differences**: Different languages employ distinct tokenization strategies, leading to inconsistent token consumption.

</details>

**13. What Is Matryoshka Representation Learning (MRL)? What Is Its Value to RAG?**

<details>
<summary>💡 Show Answer</summary>

MRL (Kusupati et al. 2022): A training methodology that structures model-generated vectors hierarchically—truncating to the first $d$ dimensions (where $d$ can be 64, 128, 256, 512, 1024, etc.) yields valid embeddings, with higher dimensions delivering higher precision. Value to RAG:
1. **Flexible Accuracy-Speed Trade-Off**: Use low-dimensional vectors for the initial retrieval phase (fast recall) and high-dimensional vectors for the reranking phase (precise ranking), supporting both stages with a single model;
2. **Storage Savings**: Low-dimensional vectors (e.g., 256 dimensions) can replace full dimensions (1536 dimensions) in certain scenarios, reducing storage by $6\times$ with acceptable accuracy loss;
3. **Mobile/Edge Deployment**: Low-dimensional vectors are better suited for resource-constrained devices;
4. The OpenAI text-embedding-3 series adopts MRL, allowing users to truncate dimensions as needed.

</details>

**14. How to Efficiently Rebuild Vector Indexes When an Embedding Model Is Updated?**

<details>
<summary>💡 Show Answer</summary>

1. **Evaluate Rebuilding Necessity**: Compare the retrieval quality of the new and old models on a test set first, rebuilding only when the new model demonstrates significant improvement;
2. **Incremental Rebuilding**: Rebuild only chunks with significant similarity divergence between the old and new models rather than performing full rebuilding;
3. **Blue-Green Deployment**: Construct the new index in parallel on a new service, switch traffic after quality verification, and retain the old index for a period to enable rollback;
4. **Prioritized Rebuilding**: Rebuild high-frequency access documents first, then gradually rebuild low-frequency documents;
5. **Batch Parallel Processing**: Utilize GPU parallelism for embedding computation with batching (batch_size=256+) to maximize GPU utilization, reducing rebuilding time for large-scale knowledge bases from days to hours;
6. **Vector Compatibility Check**: If the vector spaces of the old and new models are incompatible (e.g., different dimensions), a full rebuild is mandatory.

%% ============================================================
%% 4. Vector Databases and Indexes
%% ============================================================

</details>

