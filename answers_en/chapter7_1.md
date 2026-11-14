# 7.1 RAG Basics and Principles — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 13 questions

**1. What is Retrieval-Augmented Generation (RAG)?**

<details>
<summary>💡 Show Answer</summary>

RAG is a hybrid approach that combines a retrieval system with a generative language model. It operates in two steps:
- **Retrieval Component**: Searches for relevant information in a large external corpus or dataset based on the input query.
- **Generative Model**: Generates responses using the retrieved information.

This approach enhances traditional natural language processing models by accessing external knowledge, enabling them to provide more precise, context-aware, and information-rich responses.

</details>

**2. How Does RAG Differ from Traditional Language Models?**

<details>
<summary>💡 Show Answer</summary>

Traditional models (such as GPT-3) generate text solely based on what was learned during training, relying on statistical patterns within the data without access to external knowledge sources. RAG differs by introducing a retrieval step that first retrieves relevant data from an external source before generating text.

Comparison of Core Differences:

\begin{center}


| **Feature** | **Traditional Language Model** | **RAG** |
| --- | --- | --- |
| Knowledge Source | Training data only (parametric knowledge) | Training data + external knowledge base (non-parametric knowledge) |
| Knowledge Timeliness | Restricted by training cutoff date | Accessible to real-time updated knowledge bases |
| Hallucination Tendency | Higher (tends to fabricate uncertain content) | Lower (answers anchored in retrieved documents) |
| Private Knowledge Support | Not supported | Supported (connects to private knowledge base) |
| Answer Traceability | Non-traceable | Source documents can be cited |
| Knowledge Update Cost | High (requires retraining) | Low (only requires updating the knowledge base) |
| Inference Latency | Lower (direct generation) | Higher (two steps: retrieval + generation) |


\end{center}

**Essence of the Key Difference**: The "knowledge" of traditional LLMs is stored in the model weights (parameters), which is "parametric knowledge" frozen at the time of training; the core innovation of RAG is introducing "non-parametric knowledge"—an external knowledge base—allowing knowledge to be dynamically updated and expanded independently of model parameters, fundamentally solving the core limitation of LLM knowledge staticity.

**RAG is an Extension of LLM, Not a Replacement**: RAG still relies on the language understanding and generation capabilities of the LLM (understanding queries, synthesizing multi-document information, generating fluent answers), but uses external retrieval to make up for the LLM's deficiencies in knowledge coverage and timeliness on top of that.

</details>

**3. Please Explain the Working Principle of RAG. Compared with Directly Fine-Tuning an LLM, What Main Problems Does RAG Solve? What Are Its Advantages?**

<details>
<summary>💡 Show Answer</summary>

RAG (Retrieval-Augmented Generation) works on a "retrieve-then-generate" paradigm, combining Information Retrieval (IR) with Text Generation to enhance the capabilities of Large Language Models (LLMs).

The workflow is as follows:
1. **Retrieve**: When a user asks a question, the RAG system does not immediately send the question directly to the LLM. Instead, it treats the user's question as a Query to search within an external knowledge base (typically a vector database) for the most relevant pieces of information (documents/chunks).
2. **Augment**: The system concatenates the retrieved relevant information with the user's original question to construct a richer, more informative Augmented Prompt.
3. **Generate**: Finally, this augmented prompt is fed to the LLM. The LLM generates a more accurate and factual response based on both its parametric knowledge and the provided context.

RAG primarily addresses the following core problems of LLMs:
1. **Static and Outdated Knowledge**: The knowledge of an LLM is "frozen" at the cutoff point of its training data. By connecting to an external knowledge base that can be updated at any time, RAG enables LLMs to access and utilize the latest information.
2. **Hallucination**: LLMs tend to fabricate facts when answering questions outside their knowledge scope or when uncertain. RAG "anchors" the LLM's responses to concrete, relevant context, significantly reducing hallucinations.
3. **Lack of Specialized Domain Knowledge and Private Knowledge**: Fine-tuning LLMs to inject domain-specific knowledge is costly and often yields limited effectiveness. RAG can easily connect the model to any private dataset (such as internal company documents), making it a domain expert.

Advantages of RAG compared to Fine-tuning:
- **Low Knowledge Update Cost**: Updating knowledge only requires adding or modifying documents in the database, without retraining expensive LLMs.
- **Traceability and Interpretability**: RAG can clearly present which source documents the answer is based on, allowing users to inspect sources for fact-checking.
- **Reduced Hallucination**: RAG grounds answers in factual evidence, making them verifiable.
- **Cost-Effectiveness**: For scenarios requiring factual knowledge injection, RAG's development and maintenance costs are far lower than fine-tuning.
- **Personalization**: Different knowledge sources can be dynamically connected for each user or request.

</details>

**4. What is RAG? What Core Problems of LLMs Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

RAG (Retrieval-Augmented Generation) is a framework that combines information retrieval with language generation: during inference, relevant documents are retrieved from an external knowledge base and concatenated into the prompt, enabling the LLM to generate answers based on the retrieved content. The core problems it solves are:
1. **Knowledge Cutoff**: LLM training data is time-sensitive; RAG can connect to real-time updated knowledge bases;
2. **Hallucination Issues**: Anchors generation in actual documents to reduce fabrication out of thin air;
3. **Private Knowledge Access**: Internal corporate documents and proprietary databases are not in the training data; RAG can bridge them;
4. **Traceability**: Can cite which document the answer originates from, enhancing credibility;
5. **Cost-Effectiveness**: Cheaper than full fine-tuning; knowledge updates only require updating the knowledge base.

</details>

**5. What Are the Two Stages of RAG's Standard Pipeline? What Does Each Stage Do?**

<details>
<summary>💡 Show Answer</summary>

**Offline Stage (Indexing Construction)**:
1. Load documents;
2. Text cleaning and preprocessing;
3. Split into chunks according to a strategy;
4. Call an Embedding model to vectorize each chunk;
5. Write vectors and original text into the vector database.

**Online Stage (Retrieval + Generation)**:
1. Receive user query;
2. Vectorize the query;
3. Perform similarity search in the vector database to recall top-k chunks;
4. Optional: Reranking fine-grained scoring;
5. Concatenate retrieved chunks into prompt;
6. LLM generates final answer based on prompt;
7. Optional: Citation and source attribution.

</details>

**6. What Key Steps Are Included in a Complete RAG Pipeline? Please Describe the Entire Process in Detail from Data Preparation to Final Generation.**

<details>
<summary>💡 Show Answer</summary>

A complete RAG pipeline can be divided into two main stages: the offline data preparation (indexing) stage and the online query (inference) stage.

**Stage 1: Data Preparation / Indexing Pipeline (Offline / Indexing Pipeline)**, the goal of this stage is to build a searchable knowledge base, which is usually executed once or periodically:
1. **Data Loading (Load)**: Load raw documents from various data sources, which can include PDF files, Word documents, web pages, Notion databases, Confluence pages, database tables, etc.
2. **Text Chunking (Split / Chunk)**: Split the loaded long documents into smaller, semantically complete text chunks. This step is crucial because subsequent retrieval and generation are conducted at the unit of these small chunks.
3. **Embedding (Embed)**: Use a pre-trained text embedding model (such as BERT, BGE, M3E, etc.) to convert each text chunk into a high-dimensional numerical vector. This vector captures the semantic information of the text chunk.
4. **Storage (Store)**: Store the content of each text chunk and its corresponding embedding vector into a vector database (such as FAISS, ChromaDB, Pinecone, etc.). The database indexes these vectors for efficient similarity search.

**Stage 2: Query / Inference Pipeline (Online / Inference Pipeline)**, this stage is executed in real-time when a user asks a question:
1. **User Query**: The system receives the natural language question input by the user.
2. **Query Embedding (Embed Query)**: Convert the user query into a query vector using the exact same embedding model as in the indexing stage.
3. **Vector Retrieval (Retrieve)**: Compute similarity (usually cosine similarity or dot product) between this query vector and all text chunk vectors stored in the vector database, retrieving the Top-K most similar text chunks to the query vector.
4. **(Optional) Re-ranking**: To further improve retrieval quality, a reranking model is used to score and rank the preliminary Top-K text chunks more finely, selecting the final Top-N chunks (N < K).
5. **Augment & Generate**: Combine the contents of the top N re-ranked text chunks with the user's original question into an augmented prompt, and send it to the LLM, which generates the final grounded answer based on the provided context.





![Standard RAG pipeline workflow from offline indexing to online generation. (标准RAG管道工作流：从离线索引到在线生成)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig01_rag_pipeline.png)

</details>

**7. What Are the Main Application Scenarios of RAG?**

<details>
<summary>💡 Show Answer</summary>

Common use cases for RAG include:

1. **Enterprise Knowledge Base QA**: Enterprise employees query internal documents, HR policies, technical specifications, etc. via natural language. RAG retrieves relevant content from the enterprise knowledge base and generates accurate answers, replacing inefficient keyword searches and manual browsing. This is currently the largest commercial application scenario for RAG.
2. **Intelligent Customer Service**: Connects to knowledge bases such as product manuals, FAQs, and after-sales policies to automatically answer common user questions, reducing manual customer service workload while ensuring answer accuracy (reducing hallucinations compared to pure LLMs).
3. **Domain-Specific QA Systems**: In specialized domains like law (contract review, regulation lookup), medicine (clinical guidelines, drug information), and finance (research report analysis, regulatory compliance), RAG connects to specialized knowledge bases to provide verifiable professional advice.
4. **Code and Technical Documentation Assistant**: Connects to codebases and technical documentation to help developers quickly find API usage, debug errors, and understand code logic; products like GitHub Copilot Chat utilize similar technologies.
5. **Content Summarization and Report Generation**: Retrieves relevant content from multiple documents to synthesize summaries, research reports, or briefings, suitable for analysts and researchers who need to process large amounts of text.
6. **Conversational Search Engines**: Replaces the traditional "Search $\rightarrow$ Click $\rightarrow$ Read" paradigm by directly providing comprehensive natural language answers; Perplexity AI is a representative product in this scenario.
7. **Personalized Recommendation Assistance**: Based on user history and current query, retrieves the most relevant recommendation content from the content library and generates personalized recommendation rationales, enhancing recommendation explainability.

</details>

**8. What Is the Primary Goal of Using RAG in NLP Tasks?**

<details>
<summary>💡 Show Answer</summary>

The primary goal of RAG in NLP tasks is to combine the retrieval of relevant external knowledge with generative capabilities to enhance accuracy, relevance, and context-awareness in NLP tasks. Specifically, RAG is designed to address the following core objectives:

1. **Improving Factual Accuracy**: Pure generative models are prone to hallucinations; retrieving real documents as evidence grounds the answers in facts, reducing the likelihood of generating erroneous information.
2. **Overcoming Knowledge Timeliness Limits (Knowledge Currency)**: An LLM's knowledge is frozen at its training cutoff; RAG connects to dynamic knowledge bases, enabling NLP systems to acquire and utilize up-to-date knowledge to answer questions about events and updated information after the training cutoff.
3. **Supporting Domain Knowledge Integration**: Enables general-purpose NLP models to access specialized domain knowledge (medical, legal, financial) or enterprise private data without undergoing expensive fine-tuning for each domain.
4. **Enhancing Answer Traceability**: RAG allows NLP systems to provide specific source citations for their outputs, supporting fact verification by users and regulatory bodies—vital in highly regulated industries (such as medical diagnosis and legal advice).
5. **Reducing Model Deployment and Maintenance Costs (Cost Efficiency)**: By having external knowledge bases hold most factual knowledge, NLP systems can use smaller base models; knowledge updates only require updating the knowledge base rather than retraining the model, significantly lowering total cost of ownership.
6. **Improving Contextual Relevance**: RAG enables NLP systems to dynamically retrieve the most relevant context for each query rather than relying on fixed parametric knowledge, ensuring each answer is customized to the specific context of the current query.

</details>

**9. How Does RAG Balance Retrieval and Generation?**

<details>
<summary>💡 Show Answer</summary>

RAG balances retrieval and generation by first gathering relevant data via a retriever and then having a generative model process that data. The retriever ensures factual accuracy, while the generator provides natural, human-like text. Specific balancing mechanisms and engineering practices include:

1. **Retrieval Quality Sets the Upper Bound for Generation**: The responsibility of the retrieval stage is to "provide correct and sufficient factual information," while the generation stage is tasked with "generating fluent and accurate responses based on those facts." The two are in a serial dependency relationship—if retrieval fails, generation will inevitably fail. Thus, retrieval quality is a prerequisite for system quality.
2. **Trade-off in Retrieval Quantity**: The number of chunks passed to the generator (the $k$ value) needs to balance between "covering complete information (large $k$)" and "reducing noise interference (small $k$)." In practice, top-3 to top-10 chunks after reranking are selected to strike a balance between quality and quantity.
3. **Contextual Compression**: When retrieved documents are too long, Contextual Compression is used to extract sentences most relevant to the query from each chunk, reducing the volume of content passed to the generator and lowering distraction without losing key information.
4. **Generation Constraint Prompts**: Constrain the generator to "stay faithful to retrieved content" through prompt design, preventing the generator from over-relying on its own parametric knowledge and disrupting the balance between retrieval and generation (where generation overrides retrieval).
5. **Adaptive Retrieval (Advanced)**: Frameworks like FLARE and Self-RAG allow the generator to actively decide when supplementary retrieval is needed, achieving dynamic interleaving of retrieval and generation rather than a fixed serial "retrieve-then-generate" process, better accommodating complex query requirements.

</details>

**10. How is RAG Integrated into Machine Learning Pipelines?**

<details>
<summary>💡 Show Answer</summary>

1. **Retrieval Component Integration**: The retrieval component of RAG (vector database + Embedding model) is deployed as an independent microservice, communicating with other components in the machine learning pipeline via REST API or gRPC. The retrieval component receives the vectorized query and returns relevant documents along with their metadata, decoupled from model inference services in the pipeline.
2. **Generative Component Integration**: The generative model (LLM) is likewise deployed as an independent service, receiving the constructed prompt (containing retrieval results and user query) and returning the generated text response. Multiple LLM instances can be managed via load balancers to support high concurrency.
3. **Data Pipeline Integration**: Offline document processing and indexing pipelines (ETL Pipelines) are integrated into existing data engineering infrastructure (Airflow, Prefect, etc.), processing and writing new documents into the vector database on a scheduled or event-driven basis, completely decoupled from the online inference pipeline.
4. **Feature Engineering Pipeline Integration**: Documents retrieved by RAG can serve as structured features (such as relevance scores of retrieved documents, document type distribution) passed into downstream machine learning models (e.g., classifiers evaluating answer quality), enabling integration between RAG and traditional ML models.
5. **Monitoring and Feedback Loop**: Output quality metrics of the RAG system (Faithfulness, user feedback) are sent back to the machine learning monitoring platform to monitor model performance drift and trigger retraining or parameter tuning workflows, forming a complete MLOps closed loop.
6. **A/B Testing Framework Integration**: The RAG system integrates with A/B testing platforms via feature flags or traffic splitting, supporting controlled experiments across different RAG configurations (chunking strategies, Embedding models, $k$ values), with data funneled into a unified experiment management platform for analysis.

</details>

**11. Are You Familiar with Search Systems? What Is the Difference Between a Search System and RAG?**

<details>
<summary>💡 Show Answer</summary>

Search systems and RAG systems are closely related, but their goals and final outputs differ fundamentally. It can be said that a RAG system is a higher-level application built on top of a search system.

**Search System (e.g., Google Search, Elasticsearch)**:
- **Core Goal**: Information Retrieval (IR). Based on the user's query, find and return a ranked list of documents from a large document collection.
- **Final Output**: "Sources". It provides "raw materials that may contain the answer," requiring users to click links, read documents, and synthesize answers themselves.
- **Core Technologies**: Indexing techniques (such as inverted index), ranking algorithms (such as BM25, PageRank, TF-IDF), query understanding and expansion.

**RAG System (Retrieval-Augmented Generation System)**:
- **Core Goal**: Question Answering (QA). Directly provide a precise, conversational, and comprehensive natural language answer.
- **Final Output**: "Answers". It uses retrieved "sources" as factual grounding, ultimately delivering a synthesized, refined, and summarized finished product.
- **Core Technologies**: It includes a search system as its "retrieval" module, and more critically adds a Large Language Model (LLM) as its "generation/synthesis" module.

The most key differences:

\begin{center}


| **Feature** | **Search System** | **RAG System** |
| --- | --- | --- |
| Task | Find documents | Provide answers |
| Output | Document list | Natural language answer |
| User Role | User needs to read and summarize | Directly receives finished answer |
| Core Components | Indexer + Ranker | [Indexer + Ranker] + Generator (LLM) |


\end{center}

A simple analogy: A search system is like a library librarian—if you ask about "the history of Singapore", they will tell you which books are useful and leave you to read them yourself; a RAG system is like a history expert—they will consult those books in the library and then directly tell you the answer while citing the references.





![Architectural comparison between traditional Search Systems and Retrieval-Augmented Generation (RAG) Systems. (传统搜索系统与RAG系统的架构对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig02_search_vs_rag.png)

</details>

**12. Can a RAG System Work Without Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Yes, a RAG system can function using pre-trained generative models and retrieval components, but fine-tuning on specific datasets can often enhance performance for specific applications. The detailed analysis is as follows:

**How Non-Fine-Tuned RAG Works**: The core value of RAG lies in "injecting external knowledge into the prompt," a mechanism independent of whether the model is fine-tuned. Using pre-trained models such as GPT-4 or Claude alongside high-quality knowledge bases allows building practical RAG systems without any fine-tuning, enabling rapid deployment.

**Limitations of Non-Fine-Tuned RAG**:
1. **Insufficient Understanding of Domain Terminology**: Pre-trained models may inaccurately understand highly specialized domain jargon (medical, legal, specific industry slang), leading to potential misinterpretations even when correct documents are retrieved;
2. **Inconsistent Output Formats**: Without fine-tuning, model output formats might not meet business requirements (such as requiring a fixed JSON format or specific citation formats);
3. **Varying Instruction-Following Capabilities**: The ability to adhere strictly to "answer based solely on documents without fabrication" depends on the model's RLHF training quality in different scenarios; weaker models might fail to adhere strictly.

**When Fine-Tuning is Needed**: When there is a large volume of high-quality domain data; when the model needs to master specific output styles and formats; when general-purpose models show clear deficiencies in faithfulness or relevance on evaluation sets; or when latency-sensitive applications can replace large general models with smaller fine-tuned models.

**Best Practice**: Start with pre-trained models + engineering optimizations (Prompting, Reranking, chunking strategies) for rapid validation, and consider fine-tuning only when the return on investment is sufficiently large, rather than fine-tuning from the outset.

</details>

**13. What is Naive RAG? What Are Its Main Limitations?**

<details>
<summary>💡 Show Answer</summary>

**Naive RAG**: The most basic RAG implementation, following a fixed pipeline: query $\rightarrow$ vector retrieval $\rightarrow$ concatenate top-k chunks $\rightarrow$ LLM generation, without any optimization. Main limitations:
1. **Poor Retrieval Quality**: Single vector retrieval precision is limited, suffering from semantic gaps;
2. **Coarse Chunking**: Fixed-size chunking may truncate crucial information;
3. **Failure in Multi-Hop Reasoning**: A single retrieval pass cannot gather information requiring multi-step association;
4. **Noise Interference**: Recalled chunks contain irrelevant content, degrading generation quality;
5. **Poor Faithfulness**: LLMs may "add" hallucinations beyond the retrieved content;
6. **Lack of Query Optimization**: Poorly phrased original user queries lead to poor retrieval performance.

%% ============================================================
%% 2. Document Processing and Chunking Strategies
%% ============================================================

</details>

