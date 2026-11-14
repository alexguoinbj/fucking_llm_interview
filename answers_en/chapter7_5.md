# 7.5 Retrieval Strategies (Hybrid Search, Rerank, MMR) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 22 questions

**1. What Are the Commonly Used Retrieval Technologies in RAG?**

<details>
<summary>💡 Show Answer</summary>

RAG retrieval technologies have evolved from sparse to dense, and from single-method to hybrid approaches. Modern production systems typically combine multiple technologies:

**1. Sparse Retrieval**
- **BM25**: A classic algorithm based on term frequency (TF) and inverse document frequency (IDF). It works exceptionally well for exact keyword matching, is fast, requires no training, and is the default ranking algorithm for Elasticsearch;
- **TF-IDF**: The predecessor to BM25, also based on term frequency statistics, but its document length normalization handling is less refined than BM25's. It has largely been replaced by BM25 in modern systems;
- **SPLADE**: Learns sparse embeddings through neural networks, combining the efficiency of sparse retrieval with the semantic understanding capabilities of neural retrieval.

**2. Dense Retrieval**
- **Bi-Encoder**: Queries and documents are encoded independently into dense vectors, with relevance calculated via cosine similarity or inner product. It supports offline pre-computation of document vectors and offers fast retrieval speed. Representative models: DPR, BGE, E5;
- **Late Interaction (ColBERT)**: Retains token-level vector interaction, offering higher precision than single-vector Bi-Encoders, though at a higher storage cost;
- **ANN Acceleration**: Uses Approximate Nearest Neighbor (ANN) algorithms such as FAISS (IVF-PQ, HNSW) and ScaNN to accelerate dense vector retrieval, reducing search times across millions of vectors to the millisecond scale.

**3. Hybrid Retrieval**
- Uses both BM25 and dense retrieval simultaneously, fusing results from both paths using Reciprocal Rank Fusion (RRF);
- Combines the exact keyword matching of sparse retrieval with the semantic understanding of dense retrieval, making it currently the most stable retrieval solution in terms of performance.

**4. Fine-grained Ranking Models (Reranker)**
- **Cross-Encoder**: Concatenates the query and candidate document for joint encoding. It offers high accuracy but cannot be pre-computed offline, making it suitable for the post-recall re-ranking stage (top-50 $\rightarrow$ top-5);
- **Representative models**: BGE-Reranker, ms-marco-MiniLM, Cohere Rerank API.

**5. Graph-based Retrieval**
- Traverses entity relationships based on knowledge graphs, making it suitable for scenarios requiring multi-hop reasoning and entity association queries, such as GraphRAG.

</details>

**2. Why Is the Retrieval Model Important in a RAG System?**

<details>
<summary>💡 Show Answer</summary>

The retrieval model is a critical bottleneck of a RAG system—the generation model can only generate answers based on retrieved content, and information that is not retrieved can never appear in the answer. The importance of the retrieval model is reflected in the following dimensions:

1. **Determines the upper bound of system performance**: The core assumption of RAG is that "the correct answer exists in the knowledge base and can be retrieved." The quality of the retrieval model determines the extent to which this assumption holds true. For every 10% increase in retrieval recall rate, end-to-end answer quality typically improves by 5--8%.
2. **Gatekeeper of information admission**: The retrieval model determines which information is allowed to enter the LLM's context window. It acts both as a "guide" for information (recalling relevant content) and as a "filter" for noise (excluding irrelevant content). This dual role is crucial to the quality of the final answer.
3. **Bridge for knowledge updating**: The core reason RAG holds more practical value than fine-tuning is that updating knowledge requires only updating the knowledge base, while the retrieval model is responsible for accurately "fetching" the updated knowledge for LLM use. The stability of the retrieval model ensures reliable transmission of knowledge updates.
4. **Core of semantic understanding**: A good retrieval model understands that "Apple phone" and "iPhone" are semantically identical, supports cross-lingual retrieval (e.g., retrieving English documents with a Chinese query), and understands domain-specific terminology. These capabilities go far beyond simple keyword matching and are key to differentiating RAG systems from traditional search.
5. **Cornerstone of system scalability**: As the scale of the knowledge base grows (from tens of thousands to hundreds of millions of documents), the efficiency of the retrieval model (ANN algorithms, index structures) directly determines whether the system can complete retrieval within an acceptable latency, impacting the scalability of the entire RAG system.

In short: **Whatever the retrieval model finds, the LLM can only generate based on that**; without high-quality retrieval, even the most powerful LLM cannot deliver correct answers.

</details>

**3. What Is Hybrid Search? How Are Vector Retrieval and Keyword Retrieval Fused?**

<details>
<summary>💡 Show Answer</summary>

Hybrid search: Uses vector retrieval (capturing semantic similarity) and keyword retrieval (BM25, capturing exact lexical matching) simultaneously, fusing the results of both paths to complement each other's strengths. Fusion methods include:
1. **RRF (Reciprocal Rank Fusion)**: Weighted sum of reciprocal ranks from both retrieval paths to re-rank results; simple, effective, and parameter-free;
2. **Weighted Fusion**: Vector score $\times \alpha$ + BM25 score $\times (1-\alpha)$, where $\alpha$ is a hyperparameter;
3. **Cascading**: Uses one retrieval path for coarse filtering first, followed by fine-grained re-ranking using the other path.
Usage scenarios: Documents containing domain terminology, product part numbers, or other items requiring exact matching; when user queries contain exact keywords ("iPhone 15 Pro"), BM25 is more accurate than vector retrieval. Hybrid search typically improves retrieval quality by 5--15% compared to a single method.

</details>

**4. Why Is Hybrid Search Becoming Increasingly Popular?**

<details>
<summary>💡 Show Answer</summary>

It combines the advantages of semantic retrieval (handling synonymous/paraphrased expressions) and keyword retrieval (handling exact matches like proper nouns and product codes), compensating for the shortcomings of single retrieval methods and delivering more stable overall performance in most scenarios. Specific reasons include:

1. **Mutual complementarity of the two retrieval methods**:
    - Dense Retrieval: Excel at handling synonyms, paraphrased expressions, and semantically similar questions, but weak at matching exact keywords, special product codes, and domain-specific terminology;
    - BM25 (Sparse Retrieval): Highly sensitive to exact keyword matching, but incapable of semantic understanding, treating "car" and "vehicle" as entirely different words;
    - Hybrid search combines the strengths of both, avoiding their respective drawbacks.
2. **Practical demands of enterprise knowledge bases**: Enterprise documents often contain a mixture of numerous technical terms, product models, and internal codes requiring exact matching (suitable for BM25), while also requiring semantic understanding (suitable for vector retrieval). Single vector retrieval performs inconsistently in such hybrid scenarios.
3. **Significant empirical improvements**: Extensive research and engineering practices show that hybrid search improves average retrieval quality by 5--15% on general benchmarks compared to pure vector retrieval, and rarely performs worse than a single method in almost any scenario.
4. **RRF fusion is simple and effective**: The Reciprocal Rank Fusion (RRF) algorithm can fuse results from two retrieval paths without parameters. Its engineering implementation is simple, requiring no additional training or hyperparameter tuning, making hybrid search very cost-effective to deploy.
5. **Native support by vector databases**: Mainstream vector databases such as Weaviate, Qdrant, and Elasticsearch natively support hybrid search, further lowering the barrier to adoption.
6. **Surge of proper nouns in the LLM era**: As RAG is applied to an increasing number of vertical domains, queries containing exact keywords such as model names (e.g., "GPT-4o"), version numbers ("v2.1.3"), and pharmaceutical names have become increasingly common, leading to a resurgence in the recognized value of BM25.

</details>

**5. What Is BM25? What Are Its Advantages and Disadvantages Compared to Vector Retrieval?**

<details>
<summary>💡 Show Answer</summary>

**BM25** (Best Match 25): A classic keyword retrieval algorithm that computes relevance based on term frequency (TF) and inverse document frequency (IDF), with normalization for long documents. It serves as the default ranking algorithm in Elasticsearch.

**Advantages**:
1. Exceptional performance on exact lexical matching (product codes, names of people, proper nouns);
2. Fast speed, no GPU required, low resource consumption;
3. No training required, works out of the box;
4. High retrieval sensitivity to rare vocabulary (low-frequency terms).

**Disadvantages**:
1. Unable to handle synonyms (treating "automobile" and "car" as different words);
2. Lacks semantic understanding; similar questions with different phrasing may fail to match;
3. Poor multilingual support (requires tokenizer adaptation).
The advantages of vector retrieval precisely compensate for BM25's disadvantages, making the two highly complementary.





![Performance trade-off comparison between keyword-based BM25 and semantic-based Vector retrieval. (基于关键词的BM25与基于语义的向量检索的性能对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig11_bm25_vs_vector.png)

</details>

**6. What Is Reranking (Fine-grained Ranking)? What Are the Commonly Used Reranking Models?**

<details>
<summary>💡 Show Answer</summary>

Reranking is a second round of fine-grained sorting performed on the top-$k$ results initially retrieved by vector search, ensuring that the most relevant documents appear first. Working principle: A Cross-Encoder is used to concatenate the query and each candidate document together as input into the model to produce precise relevance scores. Commonly used models include:
1. **Open-source**: BGE-Reranker (bilingual Chinese/English, with bge-reranker-large yielding the best performance), ms-marco-MiniLM (English, fast speed);
2. **Commercial APIs**: Cohere Rerank (no local deployment required, simple API calls);
3. **ColBERT**: Late interaction model, balancing precision and speed between bi-encoders and cross-encoders.
Typical pipeline: Vector retrieval recalls top-50 $\rightarrow$ Reranker fine-grained re-ranking $\rightarrow$ extract top-5 to pass to the LLM.

</details>

**7. Is the Rerank Step Mandatory in the RAG Pipeline?**

<details>
<summary>💡 Show Answer</summary>

It is not strictly mandatory, but in scenarios where the quality of recalled results varies widely or Top $K$ is large, Rerank can significantly enhance the context quality ultimately fed into the model. It is generally considered a standard component of high-quality RAG systems. Detailed analysis:

**Scenarios where Rerank is unnecessary**:
- Small knowledge base scale ($<10,000$ chunks), where vector retrieval precision is inherently high;
- Small $k$ value (top-3), where recalled result quality is already sufficient;
- Extreme latency sensitivity, where the 50--200ms latency of the Reranker is unacceptable;
- Rapid prototyping phase, where fine-grained optimization is not yet required.

**Scenarios where Rerank is strongly recommended**:
- Large knowledge base scale ($>100,000$ chunks), where relevance among the recalled top-$k$ from vector search varies significantly;
- Diverse user queries containing a mixture of domain terminology and colloquial expressions;
- Highly relevant document content with fine-grained detail differences requiring precise ordering;
- Production systems with strict answer quality requirements (e.g., legal, medical, and financial scenarios);
- Using hybrid search (vector + BM25), where score scales from both paths are inconsistent, requiring a Reranker for unified re-ranking.

**Benefits of Rerank**: Empirical data indicates that in large-scale knowledge base scenarios, adding BGE-Reranker yields an average 10%--20% improvement in end-to-end answer quality (Faithfulness + Relevancy), making it one of the single improvements with the highest ROI in RAG optimization.

**Lightweight solutions**: If a full Reranker incurs too high a latency, a MiniLM-based lightweight Reranker can be used (latency $\approx 20 ms$), striking a balance between speed and quality.

</details>

**8. What Roles Do Cross-Encoder and Bi-Encoder Play in Retrieval?**

<details>
<summary>💡 Show Answer</summary>

**Bi-Encoder (Dual-Tower Encoder)**: Queries and documents are encoded into independent vectors, with similarity computed via vector dot product or cosine similarity. It is extremely fast and supports pre-computing document vectors, making it suitable for the large-scale recall stage. Disadvantage: Queries and documents are encoded independently, preventing fine-grained token-level interactions, placing a ceiling on accuracy.

**Cross-Encoder**: Concatenates the query and document for joint encoding, allowing the model to perform full attention internally and capture fine-grained interactions with high precision. Disadvantage: Cannot be pre-computed, requiring fresh inference for every candidate pair each time; slow speed makes it unsuitable for large-scale recall.

**Collaborative Relationship**: Bi-Encoder is responsible for fast recall (top-100), while Cross-Encoder is responsible for precise re-ranking (top-5). The two-stage combination balances both efficiency and accuracy.

</details>

**9. What Is Maximal Marginal Relevance (MMR)? What Problem Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

MMR (Carbonell & Goldstein 1998): A retrieval ranking algorithm that simultaneously considers relevance and diversity. Calculation formula: `MMR = argmax[λ·Sim(doc, query) - (1-λ)·max Sim(doc, selected)]`. Each selected document must be relevant to the query while being as distinct as possible from already selected documents. Problem solved: The top-$k$ documents returned by standard vector search are often highly similar (originating from adjacent chunks in the same paragraph), introducing redundant information while genuinely relevant documents with different phrasing are ranked lower. MMR ensures that selected chunks cover different aspects to provide more comprehensive information. The $\lambda$ parameter controls the weighting between relevance and diversity ($\lambda=1$ degrades to pure relevance ranking).

</details>

**10. How to Handle Duplicate Content in Retrieval Results?**

<details>
<summary>💡 Show Answer</summary>

1. **Deduplication based on chunk_id**: When merging multi-path retrieval results, for documents sharing the same chunk_id, keep only the highest-ranked one;
2. **Semantic deduplication**: Compute cross-similarity among returned chunks; for chunks with similarity exceeding a threshold, keep only the one with the highest relevance (using the MMR algorithm);
3. **Document-level deduplication**: For multiple highly similar chunks from the same document, keep the most relevant one and merge the rest into a single entry;
4. **Templated content filtering**: Remove redundant boilerplate content present in the knowledge base (disclaimers, headers, footers) during the preprocessing stage;
5. **Diversity constraints**: Use MMR to maximize result diversity while preserving relevance, naturally reducing repetition.

</details>

**11. How to Use Metadata Filtering to Improve Retrieval Accuracy?**

<details>
<summary>💡 Show Answer</summary>

Metadata filtering: Attaching structured filter conditions during vector retrieval to perform similarity search only within the subset of chunks that satisfy the criteria. Application scenarios:
1. **Time filtering**: Retrieve documents from only the last 6 months;
2. **Type filtering**: Retrieve documents of only the FAQ type;
3. **Permission filtering**: Filter accessible documents based on user roles;
4. **Source filtering**: Retrieve documents from only specific product lines;
5. **Language filtering**: Retrieve documents matching the user's language.
Metadata filtering partitions a large vector database into multiple subsets, simultaneously improving precision and reducing candidate set size to boost retrieval speed.





![Flow diagram comparison between pre-filtering and post-filtering schemes in metadata search. (元数据过滤中前置过滤与后置过滤的流程对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig12_metadata_filtering.png)

</details>

**12. What Is Contextual Compression? What Is Its Role?**

<details>
<summary>💡 Show Answer</summary>

Contextual Compression: Before passing retrieved chunks to the LLM, a lightweight model or LLM is used to extract sentences from each chunk that are most relevant to the current query. Only the extracted essence is passed rather than the full chunk. Roles:
1. **Reduce token consumption**: Passing only relevant sentences instead of full chunks saves LLM inference costs;
2. **Reduce noise**: Filter out content within chunks irrelevant to the question to improve generation quality;
3. **Allow higher $k$ values**: Chunks are smaller after compression, allowing more candidate chunks to be passed to increase recall;
4. **Mitigate Lost in the Middle**: Less total content means the LLM can more easily focus on all relevant information.
Cost: One additional LLM call, which increases latency.

</details>

**13. What Is Multi-Source Retrieval? How Are Multi-Source Results Merged?**

<details>
<summary>💡 Show Answer</summary>

Multi-Source Retrieval: Simultaneously recalling candidate documents from multiple data sources or using multiple retrieval methods, followed by merging and re-ranking. Sources:
1. Different vector databases (storing different knowledge bases separately);
2. Different retrieval methods (vector search + BM25 + graph search);
3. Different Embedding models (different models recall different documents for the same query).
Merging methods:
- **RRF (Reciprocal Rank Fusion)**: Weighted sum of reciprocal ranks across paths for re-ranking; parameter-free and highly robust;
- **Weighted merging**: Assign weights based on the historical performance of each retrieval path;
- **Deduplicated merging**: Remove duplicate results with identical chunk_ids prior to merging;
- **Unified Reranker fine-grained re-ranking**: Use a Cross-Encoder for unified fine-grained re-ranking after merging, resolving score scale inconsistencies across different paths.

</details>

**14. Why Does Increasing Recall Sometimes Cause Final Answer Quality to Decrease?**

<details>
<summary>💡 Show Answer</summary>

Recalling more content simultaneously introduces more irrelevant or weakly relevant information, increasing noise and interference during the model's generation stage. The model can be misled by distracting content, leading to a situation where "information increases, but the signal-to-noise ratio drops." Specific mechanisms include:

1. **Decrease in signal-to-noise ratio**: Documents ranked lower in Top-$k$ have progressively lower relevance. As $k$ increases, newly added documents are mostly weakly relevant or irrelevant. When passed to the LLM alongside genuinely useful content, this noise dilutes the weight of high-quality information.
2. **Exacerbation of the Lost in the Middle effect**: The longer the context, the weaker the LLM's attention to content in the middle positions. Increasing $k$ from 5 to 20 makes it very likely that truly important chunks will be buried in a vast amount of marginal content, making them easier to ignore.
3. **Increase in contradictory information**: Expanding the retrieval scope raises the probability of encountering contradictory statements within recalled content, making the LLM more prone to generating erroneous or inconsistent answers when faced with conflicting information.
4. **LLM context window saturation**: An excessively large $k$ may exceed the LLM's context window limit, causing truncation of content or causing the LLM to degrade its detailed processing quality for each chunk in an effort to "digest" a massive volume of content.
5. **Counterintuitive tradeoff between cost and quality**: A higher $k$ incurs higher token costs while quality decreases, resulting in a double loss.

**Correct approach**: Rather than simply increasing $k$ to boost recall rate, one should enhance top-$k$ quality through Reranking fine-grained re-ranking, or use Contextual Compression to condense redundant content in each chunk, thereby increasing information density while keeping total content fed to the LLM under control.

</details>

**15. Is "Recall" Really the Only Core Problem in RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

No. Recall is only the first step. How the generation stage utilizes retrieved content (filtering noise, determining relevance, integrating multiple chunks) is equally critical. In fact, many failure cases occur when retrieval is correct, but the generation stage fails to utilize it properly. The core problems of RAG systems should be understood at three levels:

**Level 1: Recall Quality**
- Can documents containing the correct answer be retrieved? (Recall)
- Are the retrieved documents all relevant? (Precision)
- This is a necessary condition for RAG, but not a sufficient one.

**Level 2: Context Quality** (Often Overlooked)
- Is the signal-to-noise ratio of the chunks passed to the LLM high enough? Noisy chunks will interfere with LLM judgment;
- Is the order of chunks reasonable? (The "Lost in the Middle" problem)
- Is there contradictory information among multiple chunks? How can the LLM handle contradictions correctly?
- Is the context of an individual chunk complete? Can the LLM understand isolated fragments?

**Level 3: Generation Faithfulness** (Equally Critical)
- Does the LLM truly answer based on the retrieved content, or does it ignore the retrieved content?
- Does the LLM "add" hallucinated content outside of the retrieved content?
- Has the LLM correctly integrated information from multiple chunks?

Practical experience shows that about 70% of RAG failure cases are not due to "failing to retrieve", but rather "retrieved but not utilized effectively". Therefore, post-processing techniques such as context compression, reranking, and faithfulness prompt design are just as important as recall quality.

</details>

**16. Is the Truly Difficult Part of RAG Retrieval or the Subsequent Filtering/Integration?**

<details>
<summary>💡 Show Answer</summary>

As retrieval technologies (vector databases, hybrid search) become increasingly mature, "how to filter, denoise, and integrate context from recall results to make it most useful for generation" has gradually become a more prominent bottleneck. The detailed analysis is as follows:

**Advances on the Retrieval Side**: Technologies such as vector databases, hybrid search, and rerankers are relatively mature, and recall rates in standard scenarios are already quite high. However, retrieval still faces inherent difficulties: synonym and terminology variation issues, multi-hop reasoning requiring multi-round retrieval, and semantic understanding of rare domain queries.

**Difficulties on the Filtering/Integration Side (Increasingly Prominent)**:
1. **Noise Filtering**: Even with high-recall retrieval, top-k results are often mixed with relevant yet imprecise or even misleading chunks. Enabling the LLM to accurately identify and ignore this noise is a core challenge;
2. **Handling Contradictory Information**: Conflicting information may exist across multiple chunks, requiring the LLM to judge which takes precedence. However, current LLMs have limited capabilities in processing contradictory information;
3. **Multi-Chunk Information Integration**: When an answer requires synthesizing information from 3--5 chunks, the LLM must perform cross-paragraph reasoning and integration, placing high demands on the model's long-context understanding and reasoning capabilities;
4. **Faithfulness Control**: Forcing the LLM to answer strictly based on retrieved content without "supplementing" with its internal knowledge remains an ongoing engineering challenge;
5. **Lost in the Middle**: The correct content is retrieved, but ignored by the LLM because it is located in the middle of the context.

**Conclusion**: In practice, about 30% of RAG failure cases stem from failing to retrieve the correct content, while about 70% of failure cases stem from retrieving relevant content that the LLM fails to utilize properly. Therefore, subsequent filtering and integration are becoming the more critical bottleneck.

</details>

**17. Why Is RAG Prone to Introducing Noise into the Generation Process?**

<details>
<summary>💡 Show Answer</summary>

Retrieval is based on similarity rather than "correctness." Content that is similar but irrelevant or only partially relevant will be retrieved as well, interfering with the model's judgment—especially when Top K is large. The sources and impact mechanisms of noise are as follows:

1. **Similarity $\neq$ Relevance**: Vector retrieval is based on semantic similarity, so chunks that are "topically relevant but cannot answer the current question" may score very high. For example, asking about "the price of iPhones" might retrieve content like "the history of Apple Inc.", which is related but useless.
2. **Noise Propagation from Knowledge Base**: Low-quality documents in the knowledge base (disorganized formatting, outdated content, duplication and redundancy) are also indexed after embedding and will be retrieved for relevant queries, propagating source data quality issues into the generation stage.
3. **Degradation of Tail Content in Top-k**: Chunks ranked lower in similarity have weaker relevance to the query; when k is large, a large number of weakly relevant chunks enter the context, significantly degrading the signal-to-noise ratio.
4. **Cross-Chunk Contradictions**: Chunks from different documents or versions may contain conflicting information, making it difficult for the LLM to judge which to follow and leading to mixed or incorrect answers.
5. **LLM Sensitivity to Noise**: Studies show that even if only 1--2 highly relevant yet incorrect chunks are mixed into the context, the probability of the LLM generating an incorrect answer increases significantly; "contaminating" noise is far more harmful than purely irrelevant noise.

**Denoising Techniques**: Reranking (filtering out low-relevance chunks), Contextual Compression (extracting sentences most relevant to the query from chunks), knowledge base quality management (removing low-quality documents during preprocessing), setting similarity thresholds (chunks below the threshold are not passed to the LLM even if within the top-k).

</details>

**18. Please Explain the "Lost in the Middle" Problem. What Phenomenon in RAG Does It Describe? What Methods Can Mitigate This Issue?**

<details>
<summary>💡 Show Answer</summary>

"Lost in the Middle" refers to a phenomenon where LLMs tend to recall and utilize information located at the beginning and end of a long context better, while ignoring or forgetting information located in the middle.

In RAG, we typically concatenate the retrieved Top-K document chunks with the user's original question to form a long prompt. If the LLM suffers from the "Lost in the Middle" problem, Document 1 and Document K will receive full attention from the LLM, whereas documents located in the middle—even if they contain key information—have a high probability of being ignored.

Mitigation methods:
1. **Document Re-ordering**: Place the most relevant documents at the beginning and end of the context, while placing secondary relevant documents in the middle, ensuring key information resides in the LLM's "attention sweet spot."
2. **Reduce the Number of Retrieved Documents**: Strictly control the Top-K K value, such as taking only Top-3 or Top-5, to ensure high quality of retrieved documents.
3. **Instructional Prompting**: Explicitly instruct the model in the prompt to attend to all provided context, e.g., "Please ensure your answer is based entirely on all context information provided above and do not ignore any document."
4. **Fine-tuning the LLM**: Construct specific fine-tuning datasets with tasks requiring the model to utilize information in the middle of the context to answer correctly; this is the most fundamental but also most costly solution.





![Curve diagram showing the performance degradation of LLMs in 'Lost in the Middle' scenarios. (大模型在“Lost in the Middle”场景中的检索召回性能衰减曲线图)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig13_lost_in_the_middle.png)

</details>

**19. Besides Basic Vector Retrieval, What Other Techniques Can Improve RAG Retrieval Quality?**

<details>
<summary>💡 Show Answer</summary>

Basic vector retrieval often encounters bottlenecks when dealing with complex queries and diverse documents. Improvements can be made from three directions:

**1. Improving the Retriever**:
1. **Hybrid Search**: Combine sparse retrieval (e.g., BM25) and dense retrieval (vector search) to balance exact keyword matching and fuzzy semantic matching.
2. **Re-ranking**: Adopt a two-stage pipeline. First, use vector search or hybrid search to retrieve a larger candidate set (e.g., Top-50) from massive documents, and then use a Cross-Encoder to perform fine-grained re-ranking on the small candidate set to select the final Top-N (e.g., Top-5).

**2. Improving the Query**:
1. **Query Expansion and Transformation**: Instead of directly using the user's original query for retrieval, first use an LLM to "process" the query. This includes multi-query retrieval (generating multiple queries from different perspectives), HyDE (generating a hypothetical answer before retrieval), and sub-question queries (decomposing into multiple simple sub-questions).

**3. Improving the Index Structure**:
1. **Small-to-Large Chunking**: Match queries against small chunks during retrieval for high precision, but send parent chunks containing richer context to the LLM, balancing the precision of small chunk retrieval with the completeness of large chunk context.
2. **Graph Indexing**: In addition to vector indexing, use LLMs to extract entities and relationships from documents to build a knowledge graph, which is highly effective for queries requiring multi-hop reasoning and understanding entity relationships.

</details>

**20. How Does RAG Handle Ambiguous Queries?**

<details>
<summary>💡 Show Answer</summary>

RAG handles ambiguous queries by retrieving multiple documents relevant to different interpretations of the query. The generative model synthesizes this information into a coherent response. The specific processing strategies are as follows:

1. **Multi-Interpretation Retrieval**: For ambiguous queries (e.g., "How is Apple?"), the system can identify multiple possible interpretations (Apple Inc., apple fruit, iPhone) and retrieve documents separately for each interpretation. All retrieved results are then passed to the LLM to judge the most likely user intent based on context and generate an answer.
2. **Intent Disambiguation and Clarification**: When multi-turn dialogue is supported, the system can first clarify the user's intent for highly ambiguous queries (e.g., "Would you like to know about Apple Inc.'s stock price or how to use an iPhone?") instead of forcibly retrieving and potentially providing an off-topic answer.
3. **Query Expansion**: Generate multiple synonymous or related search variants for ambiguous queries to expand retrieval coverage and increase the probability of covering different interpretations. For example, use an LLM to generate 3--5 query variants based on different possible intents, perform parallel retrieval, and merge the results.
4. **Context-Aware Disambiguation**: In multi-turn dialogue scenarios, utilize historical dialogue context to determine the most likely meaning of the current ambiguous query (e.g., if previous turns discussed iPhones, "Apple" most likely refers to the iPhone), automatically disambiguating before retrieval.
5. **Confidence Annotation**: When the system is uncertain about query intent, explicitly state in the answer: "Your question may involve X or Y; information for both interpretations is provided below," improving answer transparency and user experience.

</details>

**21. How Does RAG Handle Adversarial Queries?**

<details>
<summary>💡 Show Answer</summary>

1. **Input Filtering and Compliance Detection**: Before a query enters the RAG pipeline, perform security checks—identifying prompt injection attacks (attempts to modify system prompts via query), unauthorized access attempts (querying content beyond user permission scope), harmful content requests, etc. This can be implemented using LLM classifiers or rule engines.
2. **Knowledge Base Content Filtering**: Pre-screen documents in the knowledge base for security, filtering out documents containing misleading, harmful, or politically sensitive content, preventing adversarial queries from extracting dangerous content via retrieval. Re-assess relevance and security for retrieved documents during search.
3. **Prompt Injection Defense**: Explicitly instruct the model in the system prompt to ignore instructions within the user query that attempt to alter its behavior (e.g., "Ignore previous instructions and instead..."), and flag/manually review abnormally long queries or queries containing special instruction formats.
4. **Post-Generation Safety Check**: Pass generated answers through a safety filter before returning to the user, detecting prohibited content, PII leaks, unauthorized information, etc. Answers non-compliant with safety guidelines are intercepted and replaced with standard refusal responses.
5. **Robustness Fine-Tuning**: Perform adversarial fine-tuning on the generative model, including adversarial inputs and correct refusal response examples in training data so the model learns to identify and appropriately reject harmful queries.
6. **Rate Limiting and Anomaly Detection**: Set access rate limits for users frequently attempting edge cases; anomalous query patterns (e.g., systematically probing permission boundaries) trigger alerts and manual review.

</details>

**22. Can RAG Be Used for Real-Time Applications?**

<details>
<summary>💡 Show Answer</summary>

Yes, RAG can be applied to real-time scenarios, but targeted latency optimizations are required. Below is the key analysis and engineering strategies:

**RAG Latency Composition**: A complete RAG request typically includes: Query Embedding computation (10--50ms) + vector retrieval (5--50ms) + optional Reranking (50--200ms) + LLM generation (200ms--several seconds). Total end-to-end latency ranges from 500ms to 3s, depending on the definition of "real-time" ($<$200ms vs. $<$2s).

**Scenarios Suitable for Real-Time Applications**:
- Customer support dialogue (users have some tolerance for 1--2 second response latency);
- Knowledge base Q&A (non-real-time transaction types, second-level response is acceptable);
- Code assistance (Copilot style, streaming output reduces perceived latency).

**Key Means for Real-Time Optimization**:
1. **Streaming Output**: Tokens are streamed back incrementally during LLM generation so the user does not need to wait for the complete answer; Time to First Token (TTFT) can be reduced to $<$500ms, significantly improving the perceived real-time experience;
2. **Semantic Caching**: Cache results for high-frequency repeated queries, yielding near-zero latency ($<$10ms) upon cache hit;
3. **Lightweight Embedding Models**: MiniLM-L6 (22M parameters) vs. BGE-Large (335M parameters), where the former is 5--10$\times$ faster in embedding computation;
4. **In-Memory Vector Indexing**: Load vector indices fully into memory (HNSW) for retrieval latency $<$10ms;
5. **Skip Reranking**: For extremely latency-sensitive scenarios, the reranking step can be omitted to trade a minor loss in precision for a 50--200ms latency reduction;
6. **Choose Faster LLMs**: Use small models with fast inference speeds, such as GPT-4o-mini or Claude Haiku, suitable for real-time scenarios with latency sensitivity and non-extreme quality requirements.

**Scenarios Unsuitable for Real-Time**: High-frequency trading (millisecond-level latency requirements), real-time multiplayer gaming ($<$100ms requirements). In these scenarios, the inherent latency of RAG cannot meet requirements, necessitating pre-computation or entirely different architectures.

%% ============================================================
%% 6. Query Optimization (Rewriting, HyDE, Decomposition)
%% ============================================================

</details>

