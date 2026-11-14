# 7.8 Knowledge Base Management and Engineering Practices — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 22 questions

**1. What is a Knowledge Base? What Role Does It Play in RAG?**

<details>
<summary>💡 Show Answer</summary>

A knowledge base is a collection of processed and indexed external information, serving as the core data source for RAG. Roles in RAG:
1. **Information Source**: Provides knowledge absent from LLM training data (proprietary documents, real-time information, domain-specific expertise);
2. **Retrieval Target**: What is stored in the vector database are precisely the embeddings of knowledge base documents;
3. **Accuracy Assurance**: LLM answers are anchored in knowledge base content, reducing hallucinations;
4. **Updatability**: The knowledge base can be updated independently of the model, keeping the system up to date;
5. **Source Traceability**: Each chunk is linked to original document information, enabling answers to cite sources.
The quality of the knowledge base represents the upper limit of RAG system quality.

</details>

**2. What Are the Incremental Update Strategies for RAG Knowledge Bases?**

<details>
<summary>💡 Show Answer</summary>

1. **Full Rebuild**: Periodically clear the vector database and re-index all documents. It is simple to implement and fragment-free, but time-consuming, making it suitable for scenarios with small data volumes or low update frequencies;
2. **Incremental Upsert**: When document addition/modification is detected, process only the changed documents---delete old chunks, re-embed, and insert new chunks; use document ID + hash to detect changes;
3. **Event-Driven Updates**: The document management system publishes change events, which the RAG system subscribes to and processes in real time;
4. **Soft Deletion**: Expired documents are not immediately deleted but marked as "archived," filtered out by default during retrieval, retaining rollback capabilities;
5. **Versioned Updates**: Retain vectors for each document version, supporting retrieval by version;
6. Key Challenge: Ensuring system retrieval consistency during updates, avoiding state confusion where "old documents are not deleted while new documents are already inserted."

</details>

**3. How Does RAG Support Real-Time Updated Data?**

<details>
<summary>💡 Show Answer</summary>

An incremental index update mechanism needs to be established to promptly update vector databases and metadata when data changes. For data with extremely high real-time requirements, real-time query interfaces (rather than pure retrieval) can also be combined as a supplement. The specific implementation schemes are as follows:

1. **Event-driven incremental updates**: Document management systems (CMS, databases) publish change events (via message queues such as Kafka, SQS, etc.) when documents are added, modified, or deleted. The RAG system subscribes to these events and automatically triggers the re-embedding of the corresponding chunks and vector database updates, with a target end-to-end latency of $<$ 1 minute.
2. **Scheduled batch synchronization**: For scenarios with less demanding real-time requirements (such as daily updated reports), set up scheduled tasks (e.g., hourly or daily) to scan changed documents in the data source and re-index them in batches.
3. **Real-time query interface fallback**: For data with extremely high real-time requirements (at the second level, such as stock prices or inventory levels), vector retrieval is unsuitable. Instead, direct calls to external APIs or structured database queries should be executed, passing the query results as context to the LLM and merging them with vector retrieval results.
4. **Soft deletion mechanism**: Deleted or expired documents are not immediately removed from the vector database. Instead, they are first marked as "archived", filtered out by default during retrieval, and cleaned up periodically to prevent data gaps during the update process.
5. **Version consistency guarantee**: During updates, adopt the order of "write new first, delete old later" to ensure that retrieval returns valid results at any moment, avoiding queries matching old chunks that have been deleted.

The core challenges lie in the tradeoff between high-frequency updates and the computational cost of index rebuilding, as well as maintaining retrieval consistency guarantees during updates.

</details>

**4. How to Implement Incremental Updates for RAG Indexes?**

<details>
<summary>💡 Show Answer</summary>

When adding/modifying documents, only the changed parts are re-chunked, re-embedded, and updated in the index, rather than performing a full rebuild. It is necessary to maintain the mapping between document versions and index entries to support accurate creation, reading, updating, and deletion (CRUD). The specific implementation steps are:

1. **Change detection**: Calculate the content hash (MD5/SHA256) of each document and compare it with the hash value from the last ingestion. Documents with changed hashes are regarded as modified and need to be reprocessed; documents with unchanged hashes are skipped to avoid unnecessary re-embedding.
2. **Document-chunk mapping maintenance**: Record all chunk IDs corresponding to each document ID in metadata. When a document changes, quickly find and delete all old chunks via `document_id`, then insert the newly generated chunks.
3. **Atomicity guarantee**: Adopt the sequence of "inserting new chunks first, then deleting old chunks" to ensure that valid chunks are always available for retrieval during the update process, avoiding data gaps during updates.
4. **Soft deletion mechanism**: Deleted chunks are first marked with `status=deleted` and not immediately physically deleted from the index. They are cleaned up periodically to prevent inconsistencies during concurrent updates.
5. **Version number management**: Assign version numbers to each batch update, allowing version-based rollbacks (restoring the status of chunks from `deleted` back to `active`).
6. **Batch processing optimization**: Combine updates of multiple documents into batch operations to reduce the number of writes to the vector database and increase throughput. Embedding computation uses GPU batch inference (`batch_size=128+`) to maximize utilization.
7. **Monitoring metrics**: Track the end-to-end latency of incremental updates (from document change to becoming effective in the vector database), success rate, and retry counts, and configure timeout alerts.

</details>

**5. How to Design a RAG System Supporting Real-Time Updates (Effective Immediately Upon Document Modification)?**

<details>
<summary>💡 Show Answer</summary>

1. **Event-driven architecture**: The document management system publishes change events (Kafka/SQS), which the RAG system subscribes to and processes in real time;
2. **Asynchronous processing pipeline**: Event received $\rightarrow$ chunking $\rightarrow$ embedding calculation $\rightarrow$ vector database update, fully asynchronous throughout, with a target latency of $<30$ seconds;
3. **Read-write separation**: Writing new vectors does not affect ongoing retrieval; the vector database supports concurrent read and write operations;
4. **Soft deletion + delayed cleanup**: Old chunks are first marked as "pending deletion" and are only deleted after new chunks are successfully written;
5. **Idempotency**: Update events for the same document may be delivered repeatedly; the processing logic must be idempotent (deduplicated via document ID + version number);
6. **Real-time SLA**: Monitor the end-to-end latency from document update to becoming effective in the vector database, and set alert thresholds.

</details>

**6. How to Handle Timeliness Issues in Knowledge Bases (Expired Documents)?**

<details>
<summary>💡 Show Answer</summary>

1. **Timestamp metadata**: Record creation and last modification times for each document, allowing time filter conditions to be attached during retrieval;
2. **Expiration tagging**: Tag expiration dates for content with explicit validity periods, automatically excluding them during retrieval;
3. **Periodic review mechanism**: Establish review cycles for different types of documents, verifying whether content is still valid manually or via automated tasks upon expiration;
4. **Source monitoring**: For documents sourced from external websites, monitor changes on source pages and automatically trigger re-crawling and updates;
5. **User feedback signals**: When users flag "expired information", trigger manual verification of the corresponding document;
6. **Timeliness discounting**: In retrieval ranking, apply time decay penalties (decay factor) to older documents, allowing newer documents to naturally rank higher.

</details>

**7. How to Handle Contradictory Information in Knowledge Bases (Different Documents Stating Different Answers to the Same Question)?**

<details>
<summary>💡 Show Answer</summary>

1. **Source authority ranking**: Assign authority scores to documents from different sources (official documentation $>$ blogs $>$ forums), ranking documents with higher authority higher during retrieval;
2. **Recency priority**: When authority is equal, prioritize the latest documents;
3. **Declare handling strategies in the prompt**: Instruct the LLM: "If retrieved documents contain contradictions, defer to [Source A] as authoritative and note that differing perspectives exist";
4. **Contradiction detection**: Have the LLM detect whether contradictions exist among input documents during answer generation and explicitly state them in the answer;
5. **Knowledge base quality audit**: Periodically identify and resolve contradictory content within the knowledge base to address the issue at the source;
6. **Version control**: When contradictions stem from different document versions, retain only the latest version.

</details>

**8. How to Design Document Access Control for RAG Systems (Different Users Accessing Different Documents)?**

<details>
<summary>💡 Show Answer</summary>

1. **Document permission metadata**: Attach permission tags to each chunk (`accessible_roles: ["admin", "sales"]`);
2. **Query-time permission injection**: When a user initiates a query, the system injects user roles/permissions into retrieval filter conditions, retrieving only authorized chunks;
3. **Vector database native filtering**: Most vector databases support metadata filtering, performing access control filtering concurrently with vector search;
4. **Row-Level Security**: Enforce access control at the vector database layer, similar to database Row-Level Security, preventing bypasses;
5. **User context isolation**: Cached retrieval results must be bound to user permissions, prohibiting users with different permissions from sharing retrieval caches;
6. **Permission change propagation**: When document permissions change, update the corresponding chunk permission metadata to ensure real-time enforcement.





![Document access control and permission segregation model in multi-tenant RAG. (多租户RAG系统中的文档访问控制与权限隔离模型)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig19_access_control.png)

</details>

**9. How Does a RAG System Implement Access Control?**

<details>
<summary>💡 Show Answer</summary>

Accessible documents/chunks should be filtered according to user permissions during the retrieval phase (rather than post-filtering after retrieval). Permission information should participate in retrieval filtering as part of the index metadata. The complete access control scheme is as follows:

1. **Permission metadata injection**: During document ingestion, store access permissions (accessible role lists, departments, user IDs, etc.) as structured metadata alongside chunk vectors, e.g., `accessible_roles: ["admin", "finance"]`.
2. **Query-time permission filtering**: When a user initiates a query, the system retrieves current user role/permission information from the authentication layer, converts it into filter conditions for the vector database, and executes permission filtering simultaneously with vector search, ensuring only chunks authorized for the user are returned.
3. **Importance of filtering timing**: Permission filtering must occur during the retrieval phase rather than after retrieval. Although post-filtering is easier to implement, it poses security risks—retrieved unauthorized content might be indirectly leaked through LLM generation.
4. **Row-Level Security (RLS)**: Enforce access control at the vector database layer so permission filtering cannot be bypassed by the application layer, similar to RLS mechanisms in relational databases.
5. **Cache isolation**: When caching retrieval or generation results, user permissions must be included as part of the cache key to prevent low-permission users from hitting cached results generated for high-permission users.
6. **Real-time synchronization of permission changes**: When document permissions are updated, permission metadata of corresponding chunks in the vector database must be updated synchronously to prevent security vulnerabilities such as "revoked permissions still being retrievable".
7. **Audit logging**: Record document access logs for each query to support compliance auditing.

Best practice: Access control should be treated as a first-class citizen in the architectural design phase of a RAG system, rather than an afterthought patch.

</details>

**10. How to Achieve Knowledge Base Isolation in Multi-Tenant RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

1. **Independent vector database**: Each tenant has an independent vector database instance, achieving complete physical isolation with highest security and highest cost;
2. **Shared database + namespace**: A shared vector database where each tenant is assigned an independent collection/namespace, providing logical isolation with lower costs but requiring strict guarantees that queries do not cross namespaces;
3. **Metadata filtering isolation**: All documents reside in the same collection, with each chunk tagged with tenant ID metadata, enforcing a `tenant_id = xxx` filter condition on queries;
4. **Dual permission verification**: Perform tenant verification at both the API layer and vector database layer to prevent single-point failures;
5. **Data encryption**: Vectors of different tenants are encrypted and stored with distinct keys;
6. **Audit logging**: Log every cross-tenant operation for security auditing.

</details>

**11. What Are the Best Practices for Document Version Management in a Knowledge Base?**

<details>
<summary>💡 Show Answer</summary>

1. **Version number tagging**: Each document has a version number and creation timestamp, and chunks inherit version information from their parent document;
2. **Coexisting versions**: When a new version is released, older versions are not deleted immediately but retained for a period (e.g., 30 days) for rollback or version comparison queries;
3. **Default to latest version**: Retrieval returns only the latest document version by default;
4. **On-demand historical version retrieval**: Allow querying historical versions by specifying version numbers (for audit or comparison purposes);
5. **Change logs**: Record content summaries for each version update to facilitate traceability;
6. **Automated cleanup**: Old versions exceeding the retention period are automatically archived or deleted to prevent index bloat;
7. **Atomic updates**: Insertion of new version chunks and deletion of old version chunks are performed within transactions to prevent inconsistent states.

</details>

**12. How to Detect and Clean Up Low-Quality Documents in Knowledge Bases?**

<details>
<summary>💡 Show Answer</summary>

Detection methods:
1. **Rule-based filtering**: Text is too short ($<50$ words), too long ($>100,000$ words), contains an excessively high proportion of special characters, or has abnormal stop-word ratios;
2. **Duplicate detection**: MinHash LSH to detect semantically duplicate documents ($>90%$ similarity);
3. **Information density scoring**: Use language models to evaluate document information density;
4. **Retrieval frequency analysis**: Documents that have not been retrieved by any query for a long time may be low quality or expired;
5. **User feedback**: Source documents flagged by users as "useless".
Cleanup strategies: Automatically clean up obvious low-quality documents triggered by rules; manually review edge cases; establish document quality scores to conduct periodic batch cleanup of low-scoring documents; create backups prior to cleanup to prevent accidental deletion.

</details>

**13. How to Evaluate Knowledge Base Coverage (What Proportion of Target Questions Can Be Answered)?**

<details>
<summary>💡 Show Answer</summary>

1. **Construct target question set**: Collect $N$ most frequently asked questions by target users (via user log analysis + business expert curation);
2. **Manual annotation**: Manually evaluate whether the knowledge base contains documents required to answer each question;
3. **Automated coverage evaluation**: Use the RAG system to answer each question and employ LLM-as-judge to determine if the answer has sufficient knowledge support;
4. **Coverage = Number of questions with sufficient knowledge support / Total number of questions**;
5. **Gap analysis**: Categorize uncovered questions to guide directions for knowledge base supplementation;
6. **Continuous monitoring**: Online systems track the proportion of "no relevant content found in knowledge base" events as a proxy metric for coverage;
7. Target coverage depends on the scenario (customer service Q&A typically requires $>90%$).





![Evaluation process for measuring the knowledge coverage rate of external repositories. (外部知识库覆盖率与问答支撑能力的评估流程)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig20_coverage_evaluation.png)

</details>

**14. How to Build a Multimodal Knowledge Base (Text + Images + Tables)?**

<details>
<summary>💡 Show Answer</summary>

1. **Text content**: Standard embedding pipeline, chunking and vectorizing text before storing in a vector database;
2. **Image processing**: Option A: Use multimodal models (GPT-4V, LLaVA) to generate detailed image descriptions and perform text embeddings on descriptions; Option B: Use multimodal embedding models (CLIP, ImageBind) to generate vectors directly for images; Option C: Hybrid, storing both image descriptions and image vectors;
3. **Table processing**: Convert to Markdown or CSV format before performing text embedding; generate natural language summaries for complex tables;
4. **Unified retrieval interface**: Use a unified retrieval API for contents of different modalities, routing internally to distinct embeddings and indexes;
5. **Metadata recording content type**: Tag each chunk with `content_type: image/table/text` to facilitate filtering by type;
6. **Multimodal reranker**: Use multimodal models to rerank mixed content after retrieval.

</details>

**15. How to Design Cache Strategies for RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

Retrieval results and final generated responses for frequent queries can be cached (with expiration strategies configured according to data update frequencies) to reduce redundant computation and costs. The complete multi-tiered caching scheme is as follows:

1. **Embedding cache**: Embedding computation results for identical text can be cached (using text content hash as key). During query preprocessing and document updates, avoid duplicate calls to the Embedding API for unchanged content, yielding significant cost savings, especially in batch scenarios with high repetition.
2. **Retrieval result cache (Semantic Cache)**: Cache vector retrieval results of queries using the query embedding vector as key (via approximate nearest neighbor matching, allowing semantically similar queries to hit the same cache entry). Suitable for high-frequency repetitive questions (e.g., FAQ scenarios), achieving hit rates of 30%--60%. Note: Corresponding caches must be invalidated when the knowledge base is updated.
3. **Generated result cache (Response Cache)**: Cache complete "query $\rightarrow$ final answer" pairs, suitable for identical or highly similar questions. Invalidation policy: Bind to knowledge base versions so that caches expire upon knowledge base updates; set TTLs (e.g., 24 hours).
4. **Tiered TTL strategy**: Set different cache durations based on data update frequencies: real-time data (no cache or TTL = 5 minutes); daily updated data (TTL = 1 hour); static knowledge base (TTL = 24 hours or version-based invalidation).
5. **Permission isolation**: Cache keys must include user permission identifiers to prevent data leaks caused by users of different permission levels sharing cached entries.
6. **Cache warming**: Identify high-frequency queries (extracted from historical logs) and pre-warm caches for these queries after system startup or knowledge base updates, avoiding high latencies during cold starts.
7. **Cost-benefit evaluation**: Embedding cache and retrieval cache offer high ROI (evident computational cost savings); generated response cache requires balancing cache consistency against cost savings and is unsuitable for highly personalized Q&A.

</details>

**16. How Does RAG Handle Long Documents?**

<details>
<summary>💡 Show Answer</summary>

Long documents must first be sliced appropriately while retaining hierarchical structural information (such as section ownership). During retrieval, a multi-level retrieval strategy of "retrieving section summaries first, then retrieving fine-grained chunks within sections" can be adopted. The complete scheme is as follows:

1. **Structure-aware chunking**: Perform semantic chunking using the inherent structure of long documents (table of contents, section titles, paragraph markers) rather than simply splitting by character count. Each chunk retains the hierarchical path "Document Name $\rightarrow$ Section Name $\rightarrow$ Paragraph" as a metadata prefix, ensuring independent chunks maintain context positioning.
2. **Multi-granularity indexing (hierarchical summaries)**: Generate multi-granularity indexes for long documents:
    - Full-text summary (1 vector): Used for macro-matching to determine if the document is relevant to the query;
    - Section summaries (several vectors): Used to identify the section containing the answer;
    - Fine-grained chunks (large number of vectors): Used for precise positioning of specific paragraphs.
3. **Two-stage retrieval**: Perform coarse-grained retrieval using section summary vectors first to narrow down candidates, followed by fine-grained chunk retrieval within candidate sections to locate relevant paragraphs precisely.
4. **Parent-child chunk strategy**: Use small chunks ($\sim 200$ tokens) for precise retrieval matching; upon hit, retrieve the corresponding parent chunk ($\sim 800$ tokens) to pass to the LLM, balancing retrieval accuracy and context completeness.
5. **Cross-page continuity processing**: Formats like PDF may break semantic continuity across page boundaries. Preprocessing detects cross-page paragraphs (judging by whether the page-end text ends with a complete sentence) and concatenates cross-page content prior to chunking.
6. **Map-Reduce generation**: When a question requires synthesizing an entire long document (e.g., "Summarize key points of the document"), generate local answers for each chunk individually (Map), and then aggregate all local answers into a final answer (Reduce), rather than attempting to stuff the entire document into a single LLM invocation.

</details>

**17. What Types of Data Sources Can RAG Use?**

<details>
<summary>💡 Show Answer</summary>

The knowledge base of a RAG system can connect to almost any type of data source, with different processing strategies for different data types:

**1. Unstructured text documents** (Most common)
- **Document files**: PDF, Word (.docx), PowerPoint (.pptx), Markdown, plain text (.txt);
- **Web content**: Scraped via crawlers or URL lists, processing HTML to strip irrelevant content (navigation bars, ads);
- **E-books**: EPUB, MOBI formats.

**2. Structured and semi-structured data**
- **Databases**: SQL databases (MySQL, PostgreSQL) support natural language queries via Text-to-SQL;
- **Spreadsheets**: CSV, Excel, where each row/column can be transformed into natural language descriptions before indexing;
- **JSON/XML**: Configuration files, API response data, requiring preprocessing to flatten into text;
- **Knowledge graphs**: Neo4j, RDF triple databases, supporting structured entity-relation retrieval.

**3. Collaboration and productivity tools**
- **Wikis and documentation platforms**: Confluence, Notion, SharePoint, MediaWiki;
- **Code repositories**: GitHub, GitLab, code files, READMEs, issues, PR descriptions;
- **Instant messaging**: Historical message logs from Slack, Teams (suitable for enterprise internal knowledge accumulation);
- **Email**: Historical email threads (requiring privacy filtering).

**4. Real-time and streaming data sources**
- **News APIs**: Real-time news feeds (NewsAPI, Reuters Data Services);
- **Search engine APIs**: Bing Search API, SerpAPI, providing real-time web search results;
- **Financial data APIs**: Stock prices, financial reports, macroeconomic data;
- **IoT data**: Sensor readings, device statuses (typically requiring streaming processing prior to indexing).

**5. Multimedia content** (Requires additional processing)
- **Images**: Indexed after generating descriptions via multimodal models (GPT-4V), or using multimodal embeddings like CLIP;
- **Audio/Video**: Indexed after generating subtitles/transcripts via speech-to-text (Whisper);
- **Scanned PDFs**: Indexed after extracting text via OCR.

</details>

**18. How Does RAG Process Structured Data (e.g., Tables, Database Records)?**

<details>
<summary>💡 Show Answer</summary>

Structured data is generally unsuitable for direct vector retrieval. It can be converted into natural language descriptions prior to indexing, or methods such as "Text-to-SQL" can be employed to allow the model to query structured data sources directly, followed by combining the results with unstructured retrieval results. The specific schemes are as follows:

1. **Vector indexing after natural language transformation**: Convert tables and database records into natural language descriptions (e.g., transforming a single record into "Product X costs Y dollars with Z units in stock"), then embed these descriptions into a vector database to support fuzzy semantic retrieval. Suitable for scenarios with moderate data volumes and relatively static content.
2. **Text-to-SQL**: For structured databases, have the LLM convert user natural language questions into SQL queries, execute queries directly, and return precise results. Suitable for questions requiring exact numeric values, aggregations, and filtering (e.g., "Top 5 products by sales volume last year").
3. **Embedding Markdown tables into context**: For small tables embedded in documents, convert them to Markdown format and include them directly in chunk text. LLMs can directly parse Markdown tables, suitable when table scale is small ($<$20 rows).
4. **Table summary generation**: For large tables, use an LLM to pre-generate textual summaries (describing table structure, key data distribution, important statistics) and store vectors of summaries. When a table is retrieved, pass both the summary and original table to the LLM.
5. **Hybrid routing**: Construct a routing layer to identify whether a query involves structured data; if so, take the Text-to-SQL path, otherwise take the vector retrieval path, and finally merge the results of both paths.
6. **Challenges**: Text-to-SQL accuracy remains limited in complex schemas and multi-table join scenarios; natural-language-transformed structured data may lose precise numeric information, requiring appropriate selection of schemes based on specific scenarios.





![Information flow for processing structured tables and databases in RAG pipeline. (RAG管道中处理结构化表格与数据库的信息流图)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig21_structured_data_rag.png)

</details>

**19. How Does RAG Support Graph-Structured Data?**

<details>
<summary>💡 Show Answer</summary>

A graph database can be built to support entity-relation-based graph retrieval (GraphRAG), finding query-related entities and their associated information via graph traversal, which compensates for the limitation of pure vector retrieval in expressing multi-hop relationships. The specific implementation is as follows:

1. **Knowledge Graph Construction**: Use NLP techniques (NER + Relation Extraction) to extract entities and relations from documents, building a knowledge graph stored in a graph database (such as Neo4j, Amazon Neptune, etc.). Nodes represent entities (people, places, concepts), while edges represent relationships (belongs to, created in, competitive relationship, etc.).
2. **Entity Linking**: Associate chunks in the vector database with entity nodes in the knowledge graph, giving each chunk both a semantic vector and a structured position in the graph.
3. **Graph Retrieval Path**:
    - Identify core entities from the query (e.g., "OpenAI");
    - Locate the entity node in the graph, perform graph traversal (breadth-first / depth-first) to find relevant nodes and relationships within N hops;
    - Use these relevant nodes as anchors to retrieve corresponding text chunks in the vector database;
    - Combine the structured graph information with the vector-retrieved text and pass them to the LLM.
4. **Community Detection and Global Search**: For questions requiring a global perspective, utilize the community structure of the graph (e.g., Louvain algorithm) to partition the graph into thematic communities, generate summaries for each community, and support macro QA such as "What are the main themes of these documents?".
5. **GraphRAG Framework**: Microsoft's open-source GraphRAG provides a complete pipeline from documents to graphs to QA, representing the most mature graph-structured RAG implementation currently available.
6. **Applicable Scenarios**: Highly interconnected knowledge (medical drug-disease-gene relations, corporate equity ownership, scientific literature citation networks); complex QA requiring multi-hop reasoning; scenarios requiring comprehension of entity relationships.

</details>

**20. How Is RAG Applied to Codebase Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Chunking strategies need to be designed specifically for code characteristics (chunking by function/class/module rather than fixed lengths), combining code semantic embeddings with symbol-level indices (such as call graphs and definition references) to support cross-file dependency retrieval. The complete solution is as follows:

1. **Code-Aware Chunking**: Use AST (Abstract Syntax Tree) to parse code and split it by function, class, or method units, rather than cutting by line count. Each chunk includes the complete function signature, function body, and docstring to ensure semantic integrity.
2. **Code-Specific Embedding Models**: Use embedding models pre-trained on code (e.g., CodeBERT, UniXcoder, StarEncoder), which possess significantly better code semantic understanding capabilities than general text embedding models.
3. **Metadata Enrichment**: Attach rich metadata to each chunk—file path, class name, function signature, programming language, imported dependencies, caller functions, etc.—to support precise symbol-based filtering.
4. **Symbol Index**: Establish an independent symbol index (function name $\rightarrow$ definition location, class name $\rightarrow$ method list) to support exact symbol queries ("find the definition of the DataLoader class"), complementing semantic retrieval.
5. **Dependency Graph Enhancement**: Build a code call graph so that when a function is retrieved, its directly dependent functions and classes are automatically associated and passed to the LLM together, enabling it to understand the full call context.
6. **Hybrid Natural Language + Code Retrieval**: Developers' queries are typically natural language ("how to initialize a database connection"), requiring mapping natural language queries into code semantic space using models that support cross-modal natural-language-to-code retrieval (e.g., CodeT5+).
7. **Typical Applications**: Code QA ("What is the function of this code?"), code generation assistance ("How to add feature X to the existing codebase"), bug localization ("Which piece of code might cause this error?").

</details>

**21. How Does RAG Combine with Knowledge Graphs?**

<details>
<summary>💡 Show Answer</summary>

Knowledge graphs can provide structured entity relationship information to enhance retrieval precision (expanding retrieval based on entity associations) and generation accuracy (providing structured facts to constrain model outputs). The specific integration methods are as follows:

1. **Entity-Recognition-Enhanced Retrieval**: Identify entities in the query (people names, place names, product names, etc.), first search for related nodes and relationships of the entity in the knowledge graph, and merge the graph query results (structured facts) with vector retrieval results (unstructured text) to serve together as context for the LLM.
2. **Relation-Expansion Recall**: When a query involves entity relationships ("Who are the competitors of Company A?"), expand along edges in the knowledge graph to find all associated entities, and then use these entities as anchors to retrieve relevant text snippets in the vector database, achieving "graph-guided RAG".
3. **Multi-Hop Reasoning Support**: For questions requiring multi-step association ("Which university did the CEO of Company X's parent company graduate from?"), the knowledge graph completes the first step (X $\rightarrow$ Parent Company $\rightarrow$ Who is the CEO), and vector retrieval completes the second step (CEO's educational background), concatenating the two to accomplish multi-hop reasoning.
4. **Fact Verification and Constraint**: After generating an answer, use structured facts in the knowledge graph to verify entity relationships in the answer, detecting obvious factual errors (e.g., "The CEO of A is B" contradicting records in the graph), thereby improving factual accuracy.
5. **GraphRAG Architecture**: Microsoft's GraphRAG extracts entities and relations from documents into a graph, builds community summaries on the graph, and supports answering global questions—a capability difficult for vector RAG to achieve.
6. **Challenges**: Knowledge graph construction and maintenance costs are high (requiring entity extraction, relation extraction, entity linking, etc.), and graph coverage and update frequency remain long-term challenges.

</details>

**22. How Does RAG Ensure Responses Are Based on the Latest Information?**

<details>
<summary>💡 Show Answer</summary>

Ensuring that RAG responses are based on the latest information requires collaborative design across three levels: knowledge base maintenance, retrieval strategy, and generation stage:

**1. Knowledge Base Level (Fundamental Guarantee)**:
1. **Incremental Real-Time Updates**: Establish an event-driven automated indexing pipeline. When source documents (web pages, databases, file systems) change, re-processing and index updates are automatically triggered, targeting an end-to-end latency of $<1$ minute;
2. **Timestamp Metadata**: Record precise creation time and last update time for each document and chunk, providing a basis for subsequent time filtering and timeliness judgment;
3. **Scheduled Full Audits**: For data sources lacking proactive change push capabilities (e.g., public web pages), periodically crawl and compare hashes with existing content, triggering update workflows upon detecting changes;
4. **Expired Document Cleanup**: Set validity periods for different document types (e.g., 30 days for press releases, 1 year for product manuals); expired documents are automatically marked as archived and filtered from retrieval results.

**2. Retrieval Level (Timeliness Filtering)**:
1. **Time-Priority Ranking**: Introduce a Time Decay factor during retrieval reranking to downweight older documents, placing newer documents higher under equal semantic relevance;
2. **Time Range Filtering**: Append time filtering conditions based on the timeliness requirements of the query ("retrieve documents from the past 6 months only"), preventing outdated information from entering the context;
3. **Deduplication of New and Old Versions**: When multiple versions of the same document exist simultaneously, retain only the latest version for retrieval (managed via document ID + version number).

**3. Generation Level (Timeliness Declaration)**:
1. **Information Date Annotation**: Instruct the LLM in the prompt to annotate document dates when citing information, allowing users to know as of when the answer is based;
2. **Uncertainty Declaration**: When the latest retrieved document still exceeds a certain age (e.g., more than 6 months), proactively remind the user in the response: "This information is from [Year/Month], verification for updates is recommended";
3. **Real-Time API Bypass**: For data with extremely high timeliness requirements (minute-level changes), bypass vector retrieval and directly call real-time APIs (weather, stock prices, exchange rates) to obtain the latest data.

%% ============================================================
%% 9. RAG Evaluation (RAGAS, Faithfulness, Relevancy)
%% ============================================================

</details>

