# 9.2 Code Retrieval and Repository-Level Code Understanding — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 9: AI Coding · 22 questions

**1. How Does AI Understand the Meaning and Function of a Snippet of Code?**

<details>
<summary>💡 Show Answer</summary>

By analyzing the code's syntactic structure, naming, comments, file context, and its position in the overall call graph (who calls it and what it calls), AI comprehensively infers its functionality and purpose.

</details>

**2. What Role Do AST/CST Play in Code Understanding?**

<details>
<summary>💡 Show Answer</summary>

Abstract Syntax Trees (AST) and Concrete Syntax Trees (CST) convert code from plain text into structured syntactic representations, allowing precise extraction of symbol definitions, scopes, and invocation relationships. Compared to plain text matching, they locate structural boundaries of code more accurately, serving as essential infrastructure for building indexing and retrieval systems.



![Code parsing process from source text to Abstract Syntax Tree (AST) structure.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig05_ast_cst_structure.png)

</details>

**3. How Is a Call Graph Constructed and Used for Code Understanding?**

<details>
<summary>💡 Show Answer</summary>

Static analysis scans all function/method definitions and call sites across the codebase to establish a directed graph of "who calls whom." When understanding a code snippet, AI can trace upward along the call graph to caller usage scenarios and downward to the specific behavior of called functions, obtaining complete context beyond a single-file view.



![Dependency tracing and upstream/downstream navigation in a static Call Graph.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig06_call_graph.png)

</details>

**4. What Are the Respective Roles of Static Analysis and Dynamic Analysis in Code Understanding?**

<details>
<summary>💡 Show Answer</summary>

Static analysis extracts structural information (types, dependencies, control flows) without running code, offering low cost and broad coverage, but struggles to capture runtime-determined behavior (such as reflection or dynamic typing). Dynamic analysis captures real runtime data (call stacks, variable values) via execution, providing higher accuracy at greater cost and limited scenario coverage. The two are typically combined.

</details>

**5. How Does AI Compensate for Gaps in Understanding When Code Comments and Documentation Quality Are Poor?**

<details>
<summary>💡 Show Answer</summary>

AI can deduce true semantics by analyzing actual invocation patterns and test cases, make analogical inferences from modules with similar naming and structures within the same codebase, or supplement missing context using commit messages and PR descriptions from revision history. Overall, however, poor documentation quality still significantly degrades understanding accuracy.

</details>

**6. What Information Can Repository Commit History (Git Blame/Log) Provide for Code Understanding?**

<details>
<summary>💡 Show Answer</summary>

It reconstructs code evolution and the underlying reasons behind modifications (combined with commit messages), identifies "hotspots" of frequent recent changes (which may indicate instability or ongoing iterations), and locates the original design intent and author via Git Blame—information unobtainable from current code snapshots alone.

</details>

**7. How Are Design Patterns Identified in Code and Used to Aid Understanding?**

<details>
<summary>💡 Show Answer</summary>

Based on structural characteristics (class organization, invocation patterns), typical structures of common design patterns (such as Factory, Observer, Strategy) can be matched. Identifying design patterns helps AI rapidly establish expectations regarding overall module responsibility boundaries, reducing line-by-line analysis costs.

</details>

**8. How Should Code Retrieval Be Conducted in AI Coding?**

<details>
<summary>💡 Show Answer</summary>

By combining keyword retrieval (exact matching of function/variable names), semantic retrieval (understanding functional intent), and code structure-based retrieval (call graphs, inheritance relationships) to complement one another.

</details>

**9. How Is Ambiguity Handled for Functions/Variables with the Same Name in Code Retrieval?**

<details>
<summary>💡 Show Answer</summary>

Disambiguation requires combining scope information (file, module, class ownership) and call context rather than relying solely on name matching. When ambiguity persists, multiple candidate results along with their respective contexts can be provided to the model for further determination based on the specific task scenario.

</details>

**10. How Are Recall and Precision Balanced in Code Retrieval?**

<details>
<summary>💡 Show Answer</summary>

Low recall leads to missing critical context, causing obvious errors in generated output; low precision introduces substantial irrelevant information, diluting model attention and increasing costs. In practice, systems usually ensure high recall first (broad retrieval) and then refine precision during a re-ranking (Rerank) stage, placing the most relevant snippets at the front of the final context.



![Two-stage code retrieval pipeline: Hybrid retrieval followed by Reranking.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig07_two_stage_retrieval.png)

</details>

**11. How Can "Implicit Dependencies" (e.g., Behaviors Determined by Config Files or Environment Variables) Be Retrieved?**

<details>
<summary>💡 Show Answer</summary>

Dedicated indexes must be established for configuration files, environment variable definitions, and deployment scripts. During retrieval, code locations reading configurations or environment variables should be actively linked to corresponding configuration sources; indexing code text alone easily misses implicit dependencies outside function call chains.

</details>

**12. How Should Relevance Ranking and Trimming Be Handled When Retrieved Context Is Excessive?**

<details>
<summary>💡 Show Answer</summary>

Comprehensive scoring and ranking can be performed based on semantic relevance to the current task, "distance" in the call graph from the target location, and recent modification timestamps. Trimming is then applied according to the context window budget, prioritizing the completeness of highly relevant snippets over uniform allocation across all candidates.

</details>

**13. How Should RAG Be Applied in AI Coding Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Establish function/class/module-level indexes over the codebase, and combine code semantic retrieval to identify code snippets, documentation, and historical Issues/PRs relevant to the current task, providing them to the model as context.



![Repository-level Retrieval-Augmented Generation (RAG) system architecture.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig08_repo_rag_workflow.png)

</details>

**14. How Do Code-Oriented Embedding Models Differ from General Text Embedding Models?**

<details>
<summary>💡 Show Answer</summary>

Code embedding models require specialized training on code corpora to capture code-specific semantic similarity (e.g., functionally equivalent code with different syntaxes should be judged as similar). General text models lack targeted training on programming language syntax and call semantics, yielding significantly poorer recall quality when directly applied to code retrieval.

</details>

**15. What Role Do Vector Databases Play in Code Retrieval?**

<details>
<summary>💡 Show Answer</summary>

As infrastructure for storing and retrieving code snippet/documentation embedding vectors, they support approximate nearest neighbor (ANN) search across large-scale codebases to rapidly retrieve candidate snippets semantically closest to the current query. They serve as a critical component for enabling semantic retrieval (rather than keyword-only matching).

</details>

**16. How Does AI Achieve Repository-Level Code Understanding?**

<details>
<summary>💡 Show Answer</summary>

Typically, it indexes the entire codebase (file structure, symbol definitions and reference relationships, documentation) and combines retrieval augmentation to provide the model with repository-wide relevant context during task execution, rather than relying solely on a single-file view.

</details>

**17. How Should Indexes Be Incrementally Updated After Codebase Changes?**

<details>
<summary>💡 Show Answer</summary>

File change events must be monitored to perform local re-indexing strictly on modified files and their directly related symbols/references. Rebuilding the entire index on every edit creates severe indexing latencies on large codebases that degrade real-time usability.

</details>

**18. How Is Retrieval Efficiency Ensured in Ultra-Large Monorepo Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Indexes should be organized hierarchically by module/package. During retrieval, coarse-grained module-level filtering narrows down candidate scope before fine-grained retrieval is executed within that reduced scope, avoiding full-index searches across the entire monorepo for every query.

</details>

**19. What Are the Difficulties in Understanding Cross-Language, Multi-Module Codebases?**

<details>
<summary>💡 Show Answer</summary>

Syntax structures and dependency management vary greatly across languages. Cross-language calls (e.g., frontend calling backend APIs, calling low-level libraries via FFI) often lack unified static analysis toolchains for tracking. Extra cross-language interface contract mappings (e.g., API definitions, Schemas) must be established to correlate and understand code across languages.

</details>

**20. How Is Code Understanding Handled Across Multiple Repositories (Microservices)?**

<details>
<summary>💡 Show Answer</summary>

Service-to-service interface contract indexes (such as API docs, Protobuf/OpenAPI definitions) must be established to chain together services dispersed across repositories but connected via network calls. Combining service registries and call-tracing telemetry compensates for static analysis limitations across repository boundaries.

</details>

**21. How Are Trade-offs Made for Repo-Level Understanding When LLM Context Windows Are Limited?**

<details>
<summary>💡 Show Answer</summary>

Priority is given to maintaining the full integrity of code directly related to the current task and its 1-hop dependencies. Indirectly related components retain only summaries or interface signatures rather than full implementations—essentially compressing the "entire repository" into a "task-relevant minimal sufficient context" via retrieval and summarization mechanisms.

</details>

**22. How Can the Effectiveness of a Code Retrieval System Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluation datasets annotated with "ground-truth relevant snippets" can be constructed to measure retrieval recall, precision, and other metrics. A method closer to real-world impact is end-to-end evaluation: feeding retrieved context to the model to complete specific coding tasks and assessing whether task completion quality improves, rather than relying solely on retrieval metrics.

</details>

