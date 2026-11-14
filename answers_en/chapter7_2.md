# 7.2 Document Processing and Chunking Strategies — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 21 questions

**1. What is Document Chunking? Why Can't an Entire Document Be Used as a Single Unit?**

<details>
<summary>💡 Show Answer</summary>

Chunking is the process of splitting a document into smaller segments (chunks). Reasons why an entire document cannot be used directly:
1. **Exceeding context window**: Long documents exceed the LLM's context window and cannot be fed in directly;
2. **Poor retrieval precision**: The embedding of an entire document is an averaged representation of all its content, making the semantics too vague to precisely match specific questions;
3. **Token waste**: When passing an entire document, most content is irrelevant to the current question, wasting tokens and introducing noise;
4. **Lower relevance score**: In similarity calculations, the score of a long document containing a large amount of irrelevant content will be diluted.
A reasonable chunk size keeps the semantics of each chunk focused, resulting in higher matching precision for specific queries.

</details>

**2. What Are the Different Methods for Chunking Text Data?**

<details>
<summary>💡 Show Answer</summary>

Below are several different text data chunking methods:
- **Sentence-based parsing**: Decomposes text documents into complete sentence chunks. Simple to implement and low processing cost, but may fail to capture complete conceptual context.
- **Fixed-size parsing (with overlap)**: Decomposes documents into chunks based on a fixed number of characters or tokens, with a set overlap between chunks. Can capture semantic information spanning multiple sentences, but requires selecting an appropriate chunk size and overlap amount.
- **Custom code**: Uses custom code to create chunks based on patterns in the document structure. Suitable for documents with known or inferable structures.
- **LLM-augmented**: Utilizes large language models to create chunks, such as generating text representations of images or summaries of tables. Suitable for non-text data, but has higher processing costs.
- **Document layout analysis**: Uses libraries and services to extract document structure and text. Provides better semantic context for the content.





![Overview of different text data chunking methodologies and characteristics. (不同文本分块方法的原理与特点)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig03_chunking_methods.png)

</details>

**3. What Are the Pros and Cons of Fixed-Size Chunking vs. Semantic Chunking?**

<details>
<summary>💡 Show Answer</summary>

**Fixed-size chunking**: Splits text based on character or token counts, making it simple to implement and fast. Pros: No need to understand document structure, suitable for any format. Cons: May cut through the middle of sentences or paragraphs, destroying semantic integrity; poor contextual continuity between adjacent chunks.

**Semantic chunking**: Splits text along semantic boundaries (paragraphs, headings, sentences), preserving semantic integrity. Pros: Each chunk has complete content, resulting in higher retrieval quality. Cons: Requires understanding document structure (hard to generalize), inconsistent chunk sizes (hard to predict token consumption).

Practical recommendation: Prioritize semantic chunking (at paragraph level) supplemented by an upper size limit constraint; perform a secondary fixed-size split when the limit is exceeded.

</details>

**4. What is Recursive Character Splitting?**

<details>
<summary>💡 Show Answer</summary>

Recursive character splitting (such as LangChain's `RecursiveCharacterTextSplitter`): Tries different separators to split text in order of decreasing priority, resorting to finer-grained separators only when the resulting segments still exceed the maximum size limit. Separator priority (typically):
1. Double newline (paragraph boundary) →
2. Single newline →
3. Period / question mark / exclamation mark →
4. Comma →
5. Space →
6. Single character.
Pros: Tries to split at semantic boundaries to preserve semantic integrity; automatically adapts to different document formats; uses finer-grained splits only when necessary to avoid over-fragmentation. It is a general chunking solution that balances practicality and semantic integrity.

</details>

**5. When Building a Knowledge Base, How Should One Choose the Appropriate Chunk Size and Overlap Length? What Are the Underlying Trade-Offs?**

<details>
<summary>💡 Show Answer</summary>

Text chunking is one of the most critical and experience-driven steps in the RAG pipeline, directly affecting retrieval recall and precision.

**How to Choose an Appropriate Chunk Size**:
1. **Based on embedding model capabilities**: Embedding models have a maximum input token limit. The chunk size should be less than this limit, and many embedding models perform best when handling medium-length text (e.g., 256–512 tokens).
2. **Based on data type and structure**: Structured documents (such as papers and reports) can use semantic chunking, i.e., splitting by paragraphs and headings; unstructured long texts rely more on fixed-length chunking; code should be chunked by functions or classes, rather than simply by line count.
3. **Based on expected query types**: If user queries are typically specific and require precise targeting, smaller chunks are more effective; if user queries are broad and require synthesizing information across multiple paragraphs, larger chunks work better.

**How to Choose an Appropriate Overlap Length**: The role of overlap is to prevent semantic information from being abruptly severed at chunk boundaries. A common rule of thumb is to set the overlap length to 10%–20% of the chunk size. For example, for a chunk of 1024 tokens, an overlap of 128 or 256 tokens can be set. Larger overlap is not always better; excessively large overlap increases data redundancy and storage costs.

**Underlying Trade-offs**:
- Pros of large chunks: Contain richer context; Cons: Increased noise, reduced retrieval precision, higher cost, and easily triggering the Lost in the Middle problem.
- Pros of small chunks: High information density, strong relevance to specific queries; Cons: Insufficient context, requiring retrieval and concatenation of multiple small chunks; easily causing semantic fragmentation.

In practice, one usually starts from a reasonable baseline (such as `chunk_size=512`, `overlap=64`) and iteratively optimizes for specific document types and query scenarios by evaluating retrieval quality.

</details>

**6. How Does Chunk Size Affect Retrieval Performance? What Are the Issues with Being Too Large or Too Small?**

<details>
<summary>💡 Show Answer</summary>

**Too small (<100 tokens)**:
1. Incomplete semantics: A single chunk cannot express a complete thought;
2. Embedding vector semantics are too localized, leading to unstable similarity calculations;
3. Large index volume and high retrieval cost;
4. Fragmented information passed to the LLM, requiring more chunks to cover the answer.

**Too large (>1000 tokens)**:
1. Diluted semantics: A single chunk covers multiple topics, diluting relevance during retrieval;
2. Chunks relevant to specific queries contain a large amount of irrelevant content;
3. Consumes more tokens when passed into the LLM.

Empirical values: General text 512–1024 tokens; Q&A documents (FAQ) 200–500 tokens; code files chunked by function/class. The optimal chunk size depends on the specific document type and query patterns, and needs to be determined experimentally.

</details>

**7. Why Shouldn't Chunks Be Cut Too Finely?**

<details>
<summary>💡 Show Answer</summary>

Cutting too finely destroys semantic integrity, causing retrieved fragments to lack context, making it difficult for the model to understand their meaning; at the same time, an excessively large number of fine chunks increases indexing and retrieval costs. Detailed analysis is as follows:

1. **Destruction of semantic integrity**: When a complete semantic unit (such as an argument and its supporting sentences) is split into multiple chunks, each chunk viewed individually is semantically incomplete. Receiving such fragments makes it difficult for the LLM to understand their true meaning, and even harder to generate accurate answers based on them.
2. **Degradation of embedding quality**: Embedding models have limited capability in representing the semantics of excessively short text (e.g., 1--2 sentences); vectors cannot fully capture semantic information, leading to unstable similarity calculations and false matches where items are "semantically similar but topically irrelevant".
3. **Ambiguity caused by lack of context**: The exact same sentence can have completely different meanings in different contexts. When chunked too finely, chunks lose their contextual backing, and retrieval may confuse chunks with similar wording but entirely different contexts.
4. **Index bloat**: Finer chunking leads to a larger number of chunks, causing vector database storage costs and ANN retrieval computation costs to grow linearly, while also increasing the probability of retrieving noisy chunks.
5. **Need to pass more chunks**: Because each chunk lacks sufficient information density, more chunks must be passed to the LLM to cover a complete answer, leading to increased token consumption and a higher likelihood of triggering the Lost in the Middle problem.
6. **Empirical rules**: It is generally recommended that the minimum chunk be no less than 100 tokens, with 256--512 tokens recommended for general scenarios to ensure that each chunk contains at least one complete semantic paragraph.

</details>

**8. How Should Chunk Size Be Selected?**

<details>
<summary>💡 Show Answer</summary>

A trade-off is required between "semantic integrity" and "retrieval precision": being too large causes a single segment to contain multiple topics and reduces retrieval precision; being too small leads to loss of context. Usually, text is split naturally based on document structure (paragraphs/sections) combined with experimental tuning, with common ranges in the magnitude of several hundred tokens. Specific selection strategies:

1. **Reference document structure**:
    - Documents with clear paragraph structure (academic papers, product documentation) → split by paragraph; natural paragraphs are usually 200--500 tokens;
    - Long narrative documents → split by chapter/section, where each section may be 500--1500 tokens;
    - FAQ-type documents → split by Q&A pair, usually 100--300 tokens.
2. **Reference embedding model limitations**: Embedding models have maximum input length constraints (e.g., BERT family typically 512 tokens, BGE-M3 supports 8192 tokens). Chunk size should not exceed the model's optimal processing length, otherwise excess portions will be truncated or suffer degradation in semantic quality.
3. **Reference query types**: Precise factual queries ("What is the price of X?") → small chunks (100--300 tokens) to increase exact match precision; comprehensive analysis queries ("Summarize the pros and cons of X") → large chunks (500--1000 tokens) to preserve more context.
4. **Experimentation first**: Chunk size is a hyperparameter whose optimal value depends on the specific document types and query distribution. It is recommended to start with 512 tokens and compare Hit Rate@k across three candidate values (256/512/1024) on an evaluation set to select the optimal value.
5. **Dynamic chunking as an advanced solution**: Using different chunk sizes for different types of documents (chunking code by functions, papers by paragraphs, FAQs by Q&A pairs) yields better results than a single uniform fixed size.

</details>

**9. What is Overlap Chunking? How Should the Overlap Size Be Set?**

<details>
<summary>💡 Show Answer</summary>

Overlap chunking: There is a certain number of overlapping tokens between adjacent chunks to prevent key information from being truncated right at the boundary between two chunks. How it works: chunk_1 covers tokens 1–500, chunk_2 covers tokens 401–900 (an overlap of 100 tokens), thus ensuring that continuous information spanning boundaries appears in at least one complete chunk. Recommendations for setting overlap size:
1. Usually 10–20% of the chunk size (e.g., chunk=512 tokens, overlap=50–100 tokens);
2. Overlap too small: Boundary information may still be lost;
3. Overlap too large: Index volume increases, and retrieved redundant content increases;
4. Adjust by document type: Structured documents (code) can have smaller overlap, while narrative documents (papers) should have larger overlap.

</details>

**10. Is It Necessary to Set Overlap Between Chunks?**

<details>
<summary>💡 Show Answer</summary>

It is necessary to some extent, as it avoids key information being truncated right at a split point; however, excessively large overlap increases index redundancy and storage costs, requiring moderate settings. Detailed analysis is as follows:

**Necessity of Overlap**:
1. **Prevent key information from being truncated**: Without overlap, if an important passage happens to span across the boundary line of two chunks, splitting into a first half and a second half, neither chunk alone can fully express the information. Consequently, whichever chunk is matched during retrieval, the LLM receives incomplete content;
2. **Preserve contextual continuity**: Overlap between adjacent chunks gives each chunk greater semantic completeness, reducing semantic fragmentation caused by split locations;
3. **Increase retrieval probability for boundary information**: Key information right at boundaries will appear in both chunks simultaneously, meaning retrieving either chunk can provide that information.

**Cost of Overlap**:
1. **Index bloat**: Larger overlap leads to more chunks, causing vector database storage costs and retrieval latency to increase linearly;
2. **Redundant content**: top-k retrieval results may contain chunks with highly repetitive content (coming from overlapping sections of adjacent chunks), requiring deduplication processing;
3. **Increased embedding computation cost**: Overlapping content needs to be embedded repeatedly, increasing index construction time.

**Recommendations**: Usually set Overlap to 10%--20% of chunk size (e.g., chunk=512 tokens, overlap=50--100 tokens); for structured documents (code, tables) with clear boundaries, Overlap can be set to 0 or a very small value; for narrative documents (papers, reports), Overlap should be appropriately increased.





![Visual representation of chunk overlap mechanism ensuring context continuity. (确保上下文连续性的块重叠机制示意图)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig04_chunk_overlap.png)

</details>

**11. What is the Parent-Child Chunk Strategy? What Problem Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

Parent-Child Chunk: Splits a document into two granularities—small chunks (Child, approx. 100–200 tokens) for precise retrieval, and large chunks (Parent, approx. 500–1000 tokens) for feeding to the LLM for generation. Workflow:
1. During retrieval, perform similarity matching using small chunks (small chunks have focused semantics and higher matching precision);
2. After retrieving a small chunk, find its corresponding parent chunk and pass the parent chunk to the LLM;
3. The LLM receives richer contextual information, resulting in higher generation quality.
Problem solved: Small chunks offer high retrieval precision but incomplete context; large chunks offer complete context but poor retrieval precision (semantics are too scattered). Parent-Child combines the advantages of both to achieve precise retrieval + complete context.

</details>

**12. How Should Chunking Strategies Be Designed for Different Formats Such as PDF, HTML, and Markdown?**

<details>
<summary>💡 Show Answer</summary>

**PDF**:
1. Preserve page numbers and paragraph structures when extracting text using PyMuPDF/pdfplumber;
2. Split by paragraph and preserve page numbers as metadata;
3. Process tables separately (convert to Markdown table format);
4. Pay attention to text extraction ordering issues in multi-column PDFs.

**HTML**:
1. Parse DOM structure and split by semantic tags (`<p>`, `<section>`, `<article>`);
2. Strip irrelevant content such as navigation bars and advertisements;
3. Preserve heading hierarchy as context prefixes.

**Markdown**:
1. Split by heading hierarchy (`#`, `##`) to form hierarchical chunks;
2. Keep code blocks intact without splitting inside code blocks;
3. Keep tables intact.

General principle: Leveraging the document's own structural information for semantic splitting is superior to blindly splitting by character counts.

</details>

**13. How Should Documents Containing Tables, Code Blocks, and Images Be Chunked?**

<details>
<summary>💡 Show Answer</summary>

**Tables**:
1. Preserve table completeness without splitting inside tables;
2. Convert tables into Markdown format or CSV for easier LLM comprehension;
3. Add titles and descriptions as context prefixes for tables;
4. For complex tables, LLMs can be used to generate textual summaries replacing the original tables.

**Code blocks**:
1. Split by functions/classes/methods as units, without splitting inside code blocks;
2. Preserve language identifiers and function signatures;
3. Add metadata such as file paths and class names to each code block.

**Images**:
1. Cannot be directly vectorized (text embedding models cannot process images);
2. Option 1: Use multimodal models (GPT-4V, LLaVA) to generate image descriptions and embed the descriptions;
3. Option 2: Extract image titles/captions as proxies;
4. Option 3: Use multimodal embedding models (CLIP).

</details>

**14. How to Add Metadata to Chunks? What Are the Common Uses of Metadata?**

<details>
<summary>💡 Show Answer</summary>

**How metadata is added**: When storing chunks in a vector database, structured metadata fields are stored simultaneously and associated with vectors. **Common metadata fields**: Document source (URL/file path), document title, author, creation/update time, page number, chapter/heading hierarchy, document type, language, version number, permission tags.

**Uses**:
1. **Metadata filtering**: Attach filter conditions during retrieval ("only retrieve documents after 2024", "only retrieve marketing type") to improve precision;
2. **Answer attribution**: Cite original document sources when outputting answers;
3. **Access control**: Filter accessible chunks based on user permissions;
4. **Timeliness management**: Filter by update time to exclude expired content;
5. **Debugging and analysis**: Track which documents are frequently retrieved.

</details>

**15. How to Handle Semantic Continuity Issues Across Pages or Paragraphs?**

<details>
<summary>💡 Show Answer</summary>

1. **Overlap chunking**: Maintain an overlap of 50–100 tokens between adjacent chunks to ensure cross-boundary information appears in at least one complete chunk;
2. **Parent-Child strategy**: Pass larger parent chunks after retrieving small chunks, naturally incorporating cross-paragraph context;
3. **Heading prefix injection**: Prefix each chunk with its section/heading information so independent chunks retain document structure context;
4. **Cross-page stitching**: During PDF processing, detect cross-page paragraphs first (where the top of the next page lacks a new paragraph marker) and stitch them before chunking;
5. **Hierarchical summarization**: Generate multi-level summaries for long documents (paragraph → chapter → full text) and index summaries of different granularities for queries requiring cross-paragraph understanding.

</details>

**16. How Much Does Document Preprocessing (Cleaning, Denoising) Affect RAG Performance?**

<details>
<summary>💡 Show Answer</summary>

The impact is significant: garbage in, garbage out (GIGO). Main steps and impacts of preprocessing:
1. **Removing irrelevant content** (headers, footers, ads, navigation bars): Reduces noise chunks and improves retrieval precision;
2. **Fixing encoding issues** (garbled text, special characters): Prevents encoding failures in embedding models due to abnormal characters;
3. **Format standardization** (unifying punctuation, removing redundant whitespace): Ensures consistent style across similar documents;
4. **Removing duplicate content** (identical or highly similar content in the document store): Reduces index redundancy and prevents the same content from being retrieved multiple times;
5. **PII removal** (personally identifiable information): Compliance requirement to prevent the knowledge base from exposing sensitive information.
Empirical evidence shows that high-quality preprocessing can improve RAG retrieval precision by 15–30%.

</details>

**17. What Preprocessing Steps Are Required for the Retrieval Component?**

<details>
<summary>💡 Show Answer</summary>

1. **Document Loading and Format Parsing**: Load documents from various data sources (PDF, Word, HTML, Markdown, databases), extract plain text content, handle special formats (convert tables to Markdown, generate descriptions for images), and preserve document structural information (heading hierarchies, paragraph boundaries).
2. **Text Cleaning**: Remove irrelevant content (headers, footers, ad navigation bars, redundant disclaimers); fix encoding issues (garbled text, special characters); standardize text format (unify punctuation marks, remove extra blank lines, normalize number formats); detect and remove duplicate documents (MinHash LSH similarity deduplication).
3. **Chunking**: Split long documents into appropriate-sized chunks (typically 200--1000 tokens) using strategies tailored to the document type, preserving semantic integrity, setting appropriate overlap, and recording source information for each chunk (document ID, page number, section).
4. **Metadata Extraction and Annotation**: Annotate each chunk with structured metadata (source URL/file path, document title, creation/update time, author, document type, language, permission tags, etc.), which will later be used for metadata filtering during retrieval and answer provenance/attribution.
5. **Embedding**: Invoke an embedding model to convert each chunk into a high-dimensional vector, ensuring the use of the same model and parameters as in the online retrieval phase; use batch processing to improve efficiency (batch_size=128--256); truncate or segment overly long chunks.
6. **Index Construction and Ingestion**: Write vectors along with their corresponding metadata and raw text into a vector database, construct ANN indexes (such as HNSW, IVF), and synchronously build BM25 inverted indexes for hybrid retrieval.
7. **Quality Verification**: Perform sampling verification on the indexing results (randomly sample chunks to query and confirm they retrieve themselves), and conduct end-to-end quality evaluation on test datasets to ensure the preprocessing pipeline introduces no errors.

</details>

**18. How to Perform Hierarchical Indexing for Ultra-Long Documents (e.g., Reports with Hundreds of Pages)?**

<details>
<summary>💡 Show Answer</summary>

1. **Hierarchical Summary Indexing**: Generate multi-level summaries for the document (section summaries $\rightarrow$ full-text summary), and index both summaries and raw chunks; select the appropriate level based on question complexity (use the summary level for macro questions and raw chunk level for detailed questions);
2. **Sparse Indexing + Dense Retrieval**: Perform coarse-grained filtering using document-level summaries first to narrow down candidate documents, then perform fine-grained retrieval within the candidate documents;
3. **Table-of-Contents-Driven Chunking**: Leverage the document's table-of-contents structure to split along chapter/section boundaries, treating each section as an independent retrieval unit and performing fine-grained chunking within each section;
4. **Sliding Window Retrieval**: Maintain multi-granularity chunk versions (small/medium/large) for ultra-long documents, and select the appropriate granularity based on the query type;
5. **Graph Indexing**: Extract entities and relationships from the document to construct a knowledge graph, supporting cross-chapter associative retrieval.

</details>

**19. How to Design an Adaptive Chunking Strategy to Dynamically Adjust Chunk Size Based on Document Type?**

<details>
<summary>💡 Show Answer</summary>

1. **Document Type Classification**: Classify documents by type first (FAQ, academic papers, code, news, legal documents, etc.), using either rule-based heuristics or classifiers;
2. **Type-to-Strategy Mapping**: Pre-configure optimal chunking strategies for each document type: FAQ $\rightarrow$ split by Q&A pairs; Academic papers $\rightarrow$ split by section (abstract/introduction/methodology/conclusion kept independent); Code $\rightarrow$ split by function/class; Legal documents $\rightarrow$ split by clause/article;
3. **Dynamic Size Adjustment**: Dynamically adjust chunk sizes for documents of the same type based on information density (paragraphs with high information density receive smaller chunks);
4. **Evaluation and Feedback**: Collect retrieval performance data and adjust strategies for document types with poor retrieval quality;
5. **Metadata Recording**: Record the chunking strategy used for each chunk in its metadata for ease of tracing and debugging.

</details>

**20. How to Evaluate the Quality of Chunking Strategies? What Are the Key Metrics?**

<details>
<summary>💡 Show Answer</summary>

1. **Retrieval Accuracy Metrics**: On a test set annotated with the "ground-truth answer chunk", calculate the proportion of top-$k$ results containing the correct chunk (Hit Rate@$k$);
2. **MRR (Mean Reciprocal Rank)**: The average of the reciprocal ranks of the correct chunks in retrieval results, measuring precise ranking quality;
3. **End-to-End Answer Quality**: Fix the LLM generation phase and compare the final answer quality under different chunking strategies (e.g., RAGAS scores);
4. **Context Precision**: The proportion of truly useful chunks among those passed to the LLM;
5. **Chunk Size Distribution**: The proportion of overly large or small chunks; an ideal distribution should concentrate within the target size range;
6. **A/B Testing**: Compare two chunking strategies in a production environment to directly compare user satisfaction.





![Evaluation metrics framework for measuring text chunking strategies. (文本分块策略的质量评估指标体系)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig05_chunking_metrics.png)

</details>

**21. How to Handle Chunking for Multilingual Documents?**

<details>
<summary>💡 Show Answer</summary>

1. **Language Detection**: Detect the language of each document during preprocessing and record it as metadata;
2. **Chunking Strategy Adaptation**: Different languages may require different delimiters (e.g., Chinese has no explicit word boundaries and requires splitting by periods/newlines, while English can split on word boundaries);
3. **Multilingual Embedding Models**: Use multilingual-supported embedding models (such as mE5, multilingual-e5-large) to ensure chunks in different languages are comparable within the same vector space;
4. **Language Filtering**: Filter by language during retrieval, or allow cross-lingual retrieval (e.g., querying English documents with a Chinese query);
5. **Translation Strategy**: If the document volume is large and languages are diverse, consider translating all documents into a unified language prior to indexing; however, translation introduces errors, requiring trade-offs.

%% ============================================================
%% 3. Embedding Models
%% ============================================================

</details>

