# 7.7 Advanced RAG (GraphRAG, Self-RAG, CRAG, Agentic RAG) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 14 questions

**1. What Key Improvements Does Advanced RAG Make Compared to Naive RAG?**

<details>
<summary>💡 Show Answer</summary>

**Pre-retrieval Improvements**: Query Rewriting (rewriting queries to improve retrieval quality), Query Decomposition (decomposing complex queries), HyDE (generating hypothetical documents).

**Retrieval Improvements**: Hybrid retrieval (vector + keyword), multi-way recall (multiple sources, multiple models), metadata filtering.

**Post-retrieval Improvements**: Reranking (fine-grained ranking to improve top-k quality), Contextual Compression (compression to reduce noise), MMR (Maximal Marginal Relevance for diversity optimization).

**Overall Architectural Improvements**: Parent-Child Chunk (retrieving with small chunks and passing large chunks), hierarchical indexing, iterative retrieval.

These improvements target different failure modes of Naive RAG. In practical systems, 3--5 improvements are usually combined rather than stacking all of them (too many improvements increase complexity and latency).

</details>

**2. What is Modular RAG? What is Its Core Idea?**

<details>
<summary>💡 Show Answer</summary>

Modular RAG (Gao et al. 2023): Deconstructs the RAG pipeline into independent functional modules, where each module can be independently replaced, combined, and configured, breaking the fixed "retrieval-generation" pipeline. Core modules: Routing (selecting retrieval strategy based on query type), Retrieval (vector/keyword/hybrid), Reranking, Compression, Fusion, Generation. Core ideas:
1. Modularization allows each stage to be optimized independently;
2. Different query types can follow different combinations of modules;
3. Supports dynamic workflows: adjusting subsequent module selection based on intermediate results;
4. Facilitates A/B testing: replacing individual modules to compare performance, rather than evaluating the system as a whole.
Compared with fixed-pipeline RAG, it is more flexible and scalable, representing the current development direction of Advanced RAG.

</details>

**3. What is GraphRAG? How Does It Differ from Traditional Vector RAG?**

<details>
<summary>💡 Show Answer</summary>

GraphRAG (Microsoft Research 2024): Extracts entities and relationships from documents to construct a knowledge graph. During retrieval, it combines vector similarity and graph traversal to support complex Q&A requiring cross-entity associations. Differences from traditional vector RAG:
1. **Data Structure**: Vector RAG stores vectors of text chunks; GraphRAG additionally stores a knowledge graph of entities (nodes) and relations (edges);
2. **Retrieval Method**: Vector RAG performs similarity retrieval; GraphRAG performs graph traversal on top of similarity retrieval (expanding along relation edges after finding relevant entities);
3. **Suitable Question Types**: Vector RAG is suitable for direct Q&A; GraphRAG is suitable for multi-hop reasoning and global scope questions;
4. **Construction Cost**: GraphRAG requires additional entity extraction and relation extraction with higher costs, making it suitable for knowledge-dense static knowledge bases.

</details>

**4. What is the Difference Between Global Search and Local Search in GraphRAG?**

<details>
<summary>💡 Show Answer</summary>

**Local Search**: Starting from the entities most relevant to the query, it expands along the relation edges of the knowledge graph to related entities and text, acting like graph-guided vector retrieval. Suitable for: Q&A targeting specific entities or concrete facts ("Who is the CEO of Company X").

**Global Search**: Utilizing the community structure of the graph, it generates hierarchical summaries for the entire graph or large subgraphs to answer questions requiring a global perspective. Suitable for: Comprehensive questions requiring synthesis across multiple entities/documents ("What are the main themes of these documents").

The two are complementary. In practical systems, they are automatically selected or combined based on the query type (specific vs. comprehensive).





![Schematic differences between Global Search and Local Search in GraphRAG framework. (GraphRAG架构中全局搜索与局部搜索的原理差异对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig16_graphrag_search.png)

</details>

**5. In What Scenarios Would You Choose to Use a Graph Database or Knowledge Graph to Enhance or Replace Traditional Vector Database Retrieval?**

<details>
<summary>💡 Show Answer</summary>

I would choose to use a graph database or knowledge graph primarily when handling highly interconnected, structured data and in scenarios requiring complex relational reasoning.

Core application scenarios:
1. **Complex questions requiring multi-hop reasoning**: For example, "Who is the CEO of the company where the author of `Llama 2` works?" requires traversing multiple hops along the relation chain, which vector retrieval can hardly accomplish, but for a knowledge graph, it is just a few simple graph traversal queries.
2. **When the data itself has strong structure and connectivity**: Corporate shareholding structures in finance, fund flow networks in fraud detection, and drug-gene-disease relationship networks in healthcare.
3. **When highly interpretable answers need to be provided**: The query path in a knowledge graph is itself a very intuitive, interpretable evidence chain, such as "Why was this transaction flagged as high risk?"

Enhancements vs. Replacement: In most cases, knowledge graphs and vector databases complement and enhance each other. A common advanced RAG pattern is: if a query involves complex relationships, first query the knowledge graph to find core entities and facts, then use this structured information as context to retrieve relevant unstructured text in the vector database, and finally summarize everything for the LLM to generate an answer.

</details>

**6. What is Self-RAG? How Does It Enable Models to Autonomously Decide Whether Retrieval is Needed?**

<details>
<summary>💡 Show Answer</summary>

Self-RAG (Asai et al. 2023): Trains a specialized LLM capable of autonomously outputting "Reflection Tokens" during the generation process to decide whether retrieval is needed, and to evaluate the relevance and faithfulness of retrieval results. Reflection token types:
1. `[Retrieve]`: Decides whether to trigger retrieval;
2. `[Relevant]`: Evaluates whether the retrieved document is relevant to the query;
3. `[Supported]`: Evaluates whether the generated sentence is supported by the retrieved content;
4. `[Utility]`: Evaluates the overall usefulness of the answer.
Advantages: Retrieval on demand (reducing unnecessary retrieval calls), more precise relevance judgment, self-evaluation of faithfulness during generation. Disadvantages: Requires dedicated training; cannot be directly applied using existing off-the-shelf general LLMs.

</details>

**7. What is Corrective RAG (CRAG)? How Does It Handle Low-Quality Retrieval Results?**

<details>
<summary>💡 Show Answer</summary>

CRAG (Yan et al. 2024): Adds a retrieval quality evaluation and correction mechanism on top of standard RAG. Workflow:
1. After standard retrieval, an evaluator (lightweight LLM) scores the relevance of the retrieval results;
2. If the relevance score is high $\rightarrow$ generate normally using retrieval results;
3. If the relevance score is medium $\rightarrow$ refine knowledge from retrieval results (retaining only relevant parts);
4. If the relevance score is low $\rightarrow$ judge as retrieval failure, triggering web search (Bing/Google) as a supplementary source.
Advantages: Automatically identifies and corrects poor retrieval quality without relying solely on the original knowledge base to provide fallback answers; suitable for scenarios where knowledge base coverage is incomplete or user queries are diverse.

</details>

**8. What is Agentic RAG? What Capability Extensions Does It Offer Compared to Standard RAG?**

<details>
<summary>💡 Show Answer</summary>

Agentic RAG: Combines the planning, tool calling, and autonomous decision-making capabilities of Agents with the knowledge retrieval capabilities of RAG, enabling the system to proactively and flexibly complete complex information acquisition and Q&A tasks. Capability extensions compared to standard RAG:
1. **Retrieval on Demand**: Does not retrieve every time; the Agent determines whether retrieval is needed;
2. **Multi-Round Iterative Retrieval**: Can retrieve multiple times within a single task to incrementally supplement information;
3. **Cross-Knowledge-Base Collaboration**: Simultaneously uses multiple tools (Knowledge Base A + Knowledge Base B + Web Search);
4. **Intertwined Reasoning and Retrieval**: First reasons about what information is needed, then retrieves, then reasons further, forming a chain of thought;
5. **Task Planning**: Decomposes complex questions into multiple sub-questions, retrieves for each, and then synthesizes;
6. **Result Verification**: Autonomously verifies the accuracy of retrieved information (multi-source cross-verification);
7. Suitable for: Complex multi-step information synthesis tasks where a single retrieval in standard RAG is insufficient.





![Interaction pattern comparison between standard static RAG and dynamic Agentic RAG. (标准静态RAG与动态智能体RAG(Agentic RAG)的交互模式对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig17_agentic_rag.png)

</details>

**9. How to Combine RAG with Agents to Build a System Capable of Proactively Deciding Retrieval Strategies?**

<details>
<summary>💡 Show Answer</summary>

The core of Agentic RAG: Allowing the Agent to autonomously decide when to retrieve, what to retrieve, and how many times to retrieve. Implementation methods:
1. **Toolifying Retrieval**: Encapsulate RAG retrieval as one of the Agent's tools (`search_knowledge_base(query)`), allowing the Agent to autonomously decide whether to call it;
2. **Multiple Retrieval Tools**: Provide multiple retrieval tools (vector retrieval / keyword retrieval / graph retrieval / web search), with the Agent selecting based on the scenario;
3. **Iterative Retrieval**: After obtaining initial retrieval results, the Agent analyzes whether they are sufficient; if insufficient, it retrieves again (using different queries or different knowledge bases);
4. **Retrieval Strategy Reasoning**: Have the Agent explicitly state in Thought "I need to retrieve X to obtain Y information";
5. **Retrieval Result Evaluation**: The Agent makes relevance judgments on retrieval results, triggering re-retrieval or alternative solutions for low-quality results;
6. **Context Management**: Multi-round retrieval results are logically integrated into the context to avoid repetition and redundancy.

</details>

**10. How to Design a RAG System Supporting "Dynamic Routing Across Multiple Knowledge Bases"?**

<details>
<summary>💡 Show Answer</summary>

1. **Query Classification Routing**: Use a classifier (LLM or lightweight classification model) to determine which knowledge base domain the query belongs to, routing it to the corresponding knowledge base for retrieval;
2. **Metadata Filtering Routing**: Implement logical routing via metadata filtering within a unified vector database;
3. **Parallel Multi-Base Retrieval**: When uncertain which knowledge base a query belongs to, issue retrieval requests to multiple knowledge bases simultaneously, merging results before reranking;
4. **Routing Confidence Score**: The router outputs a confidence score; if it falls below a threshold, parallel multi-base retrieval is triggered;
5. **Dynamic Scalability**: When adding a new knowledge base, only register the metadata and description of the new base with zero changes to the routing layer;
6. **Routing Performance Monitoring**: Track the final retrieval quality of each routing decision to identify routing error patterns and optimize the classifier.

</details>

**11. Traditional RAG Pipelines Follow "Retrieve-Then-Generate." Are You Familiar with Some More Complex RAG Paradigms?**

<details>
<summary>💡 Show Answer</summary>

Yes. Although the traditional "retrieve-then-generate" paradigm is classic, it is relatively rigid. To address more complex questions, the research community has proposed several more dynamic and intelligent RAG paradigms:

**1. Iterative Retrieval --- e.g., Self-RAG, Corrective-RAG**: Transforms RAG from a unidirectional pipeline into a looping, self-correcting process. After initial retrieval and generation, the LLM "reflects" on the tentatively generated answer and retrieved context to evaluate whether the current information is sufficient. If deemed insufficient, the LLM proactively generates a new, more targeted query to conduct a new round of retrieval. Finally, it integrates all information retrieved across rounds to generate a more comprehensive final answer.

**2. Adaptive Retrieval --- e.g., FLARE, Self-Ask**: Rather than retrieving all information at once prior to generation, retrieval occurs "on demand" during the generation process, realizing "just-in-time" information acquisition. The LLM predicts uncertainty while generating; when it anticipates generating an uncertain factual statement, it pauses generation, inserts a special placeholder, proactively formulates a query, retrieves the information, and then resumes generation.

**3. Multi-Source Data RAG**: Enables the Agent to intelligently retrieve and synthesize from diverse types of data sources. The Agent first decomposes the problem, potentially retrieving unstructured documents from a vector database, querying structured entity relationships from a knowledge graph, executing SQL database queries for precise statistics, or even calling search engine APIs for real-time information, finally synthesizing all information into a comprehensive answer.

</details>

**12. What is a Hybrid RAG System?**

<details>
<summary>💡 Show Answer</summary>

Hybrid RAG systems combine traditional retrieval methods (e.g., sparse retrieval such as BM25/TF-IDF) with neural vector retrieval models (dense retrieval) to balance speed and accuracy. Roles of components and fusion mechanisms:

1. **Sparse Retrieval (BM25/TF-IDF)**: Based on keyword term frequency statistics, it is highly effective for exact lexical matching (proper nouns, product IDs, person names), fast, requires no GPU or training, and serves as an engineering-mature, reliable baseline.
2. **Dense Retrieval (Vector Search)**: Based on semantic vector similarity from Embedding models, it understands synonyms, paraphrases, and semantic associations, making up for BM25's insensitivity to lexical variations.
3. **Fusion Methods**:
    - **RRF (Reciprocal Rank Fusion)**: Re-ranks by taking the weighted sum of reciprocal ranks from both retrieval paths. It requires no hyperparameter tuning, is highly robust, and is the most commonly used fusion method;
    - **Weighted Linear Fusion**: $score = \alpha \times dense_score + (1-\alpha) \times sparse_score$, requiring tuning of $\alpha$;
    - **Cascaded Fusion**: Uses one path for coarse filtering first (e.g., BM25 recalling 1000 documents) and another path for fine reranking (vector reranking).
4. **Practical Gains**: Compared to standalone vector retrieval, hybrid retrieval achieves an average quality improvement of 5--15% on comprehensive benchmarks, with even more significant gains in enterprise document scenarios containing domain-specific terminology.
5. **Engineering Implementation**: Elasticsearch (BM25 + vector search), Weaviate, Qdrant, etc., all natively support hybrid retrieval, eliminating the need to deploy two separate standalone systems.

</details>

**13. How Does RAG Adapt to Dynamic Data Sources?**

<details>
<summary>💡 Show Answer</summary>

Dynamic data sources refer to data whose content changes frequently, including news, real-time databases, and frequently revised documents. RAG adapts to dynamic data sources through the following mechanisms:

1. **Event-Driven Incremental Indexing**: Data sources (databases, CMS, file systems) publish events when content changes. The RAG system subscribes to these events, automatically re-embeds changed content, and updates the vector database in near real-time, rather than waiting for scheduled batch synchronization.
2. **Scheduled Batch Synchronization**: For data sources unable to provide change events, scheduled crawling tasks (e.g., hourly) scan the data source and perform diff comparisons against existing indices, re-indexing only newly added or modified content to reduce computational costs.
3. **Real-Time API Query Bypass**: For extremely volatile data (stock prices, inventory, weather), where vector retrieval freshness is insufficient, vector retrieval can be bypassed to directly call real-time APIs during generation, passing API results into the LLM as context.
4. **Document Versioning and Expiration Handling**: Each indexed document carries an update timestamp and an optional expiration time. Retrieval automatically filters out expired documents to prevent outdated content from affecting answer quality.
5. **Periodic Retriever Retraining**: For learning-based retrieval models (such as fine-tuned Embedding models), periodic fine-tuning on the latest data ensures adaptation to new domain terminology and expression changes, preventing retrieval degradation over time.
6. **Data Source Health Monitoring**: Monitors update frequencies and quality across data sources, issuing alerts for sources un-updated for long periods to prevent knowledge bases from silently becoming "outdated" without user awareness.

</details>

**14. Which Open-Source RAG Frameworks (e.g., RAGFlow) Are You Familiar With or Have Used? How Do You Choose for Different Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Beyond the most widely known LangChain and LlamaIndex, a batch of platforms focused on providing end-to-end RAG solutions has emerged, among which RAGFlow is a representative example. Other similar frameworks include Haystack, DSPy, etc.

**Understanding of RAGFlow**: Unlike "code library" style frameworks such as LangChain/LlamaIndex, RAGFlow behaves more like an "out-of-the-box," business-user-friendly RAG application platform. Its key features include: Automation and Visualization (offering deep-learning-based "intelligent" text chunking methods); End-to-End Integration (from data ingestion, processing, and indexing to final application APIs); Designed for Non-Experts (target users include business analysts or product managers, not just developers).

**How to Choose for Appropriate Scenarios**:

Scenarios to choose LangChain / LlamaIndex: Highly customized requirements (requiring deep control and customization over every stage of the RAG pipeline); integrating as a low-level library into a larger software system; developer-centric teams. In a nutshell: choose them for "flexibility" and "control."

Scenarios to choose platforms like RAGFlow / Haystack: Rapid prototyping and validation (building a RAG prototype within a few days); aiming for best practices (directly leveraging proven industry best practices); limited technical team size or business-driven teams. In a nutshell: choose them for "efficiency" and "ease of use."

My selection strategy: In the early stage of a project, use a platform like RAGFlow to build a baseline. After validating the business value, if the standardized workflows of the platform fail to satisfy deeper performance optimizations or custom business logic requirements, then consider using LangChain or LlamaIndex for fine-grained refactoring and implementation.

%% ============================================================
%% 8. Knowledge Base Management and Engineering Practices
%% ============================================================




![Architecture overview of popular open-source RAG implementation frameworks. (常用开源RAG落地实现框架的架构图)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig18_framework_comparison.png)

</details>

