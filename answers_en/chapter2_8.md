# 2.8 Context Loading and Deduplication — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 20 questions

**1. What is the role of Context in LLMs?**

<details>
<summary>💡 Show Answer</summary>

The context is all the text content attached to each request sent to the LLM. It is the model's only source of information for "perceiving the world." Specifically, the context usually consists of the following parts:

- **System Prompt**: Defines the model's role, behavioral guidelines, output format, and other global instructions.
- **Conversation History**: The sequence of user/assistant messages that have occurred in a multi-turn dialogue.
- **Retrieved Documents**: External knowledge fragments dynamically injected via mechanisms like RAG.
- **Tool Definitions**: Describes the schema of tools the model can call (function names, parameters, return value formats).
- **Structured State**: Task states, intermediate results, etc., in Agent scenarios.

When generating each token, the model calculates attention based on the entire context, so the quality of the context directly determines the output quality. The core goal of Context Engineering is: **To maximize the information density and relevance of the context within a limited token budget**.

A common misconception is "the longer the context, the better." In reality, redundant and irrelevant context not only wastes the token budget but also distracts the model's attention, causing key information to be "diluted" and reducing output quality. Therefore, how to carefully design and dynamically trim the context is a core engineering challenge for building high-quality LLM applications.

</details>

**2. What is JIT (Just-In-Time) Context Loading?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is JIT (Just-In-Time) Context Loading?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig57_what_is_jit_just_in_time_conte.png)



JIT (Just-In-Time) Context Loading is a strategy for injecting information into the context "on demand and instantly", rather than preloading all potentially useful information into the prompt before the request starts.

This concept is analogous to **Lazy Loading** in software engineering: rather than loading all resources at initialization, you only introduce a piece of resource when it is truly needed. In the context of LLMs:

- **Traditional Preloading**: Shoving the entire product manual, user history, and knowledge base into the prompt, regardless of whether the current query needs them.
- **JIT Loading**: First understand the user's intent, then precisely retrieve the few segments of content required for the current query, and inject only those contents.

The core idea of JIT is: **Information appears in the context only at the moment it is needed**. This saves tokens and avoids interference from irrelevant information, making it an important engineering paradigm for building large-scale, efficient LLM applications.

</details>

**3. What are the pros and cons of JIT Context Loading vs. Preloading (Eager Loading)?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]



| **Method** | **Pros** | **Cons** \\ \hline |
| --- | --- | --- |
| Preloading | Simple to implement, low latency | Wastes tokens, irrelevant info distracts model \\ \hline |
| JIT Loading | Token efficient, reduces interference | Complex to implement, may introduce retrieval latency \\ \hline |


\end{table}

Detailed breakdown:

#### Applicability and Limitations of Preloading
Preloading is suitable for scenarios with a small knowledge base and fixed content (e.g., only dozens of FAQs). The cost of injecting all content into the prompt is acceptable, and it avoids the risk of errors in the retrieval phase. However, when the knowledge base expands to tens of thousands or millions of items, preloading becomes unfeasible—not only does the token cost explode, but the model's ability to extract key information from a massive amount of irrelevant content drops significantly (the "Lost in the Middle" problem).

#### Core Benefits of JIT Loading
JIT has obvious advantages in large-scale scenarios: each request only injects a small amount of highly relevant fragments (usually Top-3 to Top-10), reducing token consumption and highly focusing the model's attention on relevant content. The cost is the need to introduce a retrieval module (vector database, BM25 index, etc.) and bear additional retrieval latency (usually 50-300ms).

For scenarios with a huge knowledge base where only a small amount of information is needed each time (like enterprise knowledge QA or customer service bots), JIT is clearly superior to preloading.

</details>

**4. What are common implementations of JIT Context Loading?**

<details>
<summary>💡 Show Answer</summary>

1. **RAG (Retrieval-Augmented Generation)**: The most mainstream JIT implementation. Perform vector retrieval (or hybrid retrieval) on the user's query and take the Top-K relevant fragments from the knowledge base to inject into the context. Retrieval quality directly determines JIT effectiveness, so the choice of embedding model and index building is crucial.

2. **On-Demand Pulling via Tool Calls**: Allow the model to actively request information via tool calls (e.g., querying databases, reading files, calling APIs). The model decides for itself what information it needs and when to fetch it, providing the highest flexibility, but relying on the model's tool-calling capability and introducing network/computational latency for each call.

3. **Intent Routing**: Dynamically decide which system prompt module to inject based on user intent classification. For example, only inject the refund policy module if the user's question is identified as a "refund issue"; if it's "logistics tracking", inject the logistics module. This avoids the waste of fully injecting all business rules.

4. **Lazy Document Loading**: Split long documents into fixed-size chunks and build indexes. During retrieval, only the matched chunks are recalled, rather than injecting the entire document. This is the standard practice for handling ultra-long documents (e.g., technical manuals, contracts).

These methods can be combined. For example, a mature Agent system may use RAG for knowledge retrieval, intent routing for dynamic system prompt assembly, and tool calls for real-time data pulling simultaneously.

</details>

**5. What is Context Deduplication? Why is it needed?**

<details>
<summary>💡 Show Answer</summary>

Context deduplication refers to detecting and removing duplicate or highly similar content fragments when constructing a prompt, ensuring that every piece of content injected into the model provides incremental information value.

#### Why is deduplication needed?

- **Multiple Hits in RAG Scenarios**: The same text fragment might be retrieved by multiple different queries (e.g., a core policy description is highly relevant to multiple questions). Without deduplication, this fragment will appear repeatedly in the context.

- **Content Accumulation in Multi-turn Dialogue**: The assistant's reply from the previous turn is sometimes re-injected in the next turn (e.g., some frameworks serialize all historical messages and re-pass them), causing the same info to repeatedly appear.

- **Token Waste**: Duplicate content directly consumes precious token budgets without providing any additional info. In a 128k token context window, 10% duplication means wasting ~12k tokens.

- **Model Attention Bias**: The Transformer's attention mechanism assigns higher weight to frequently appearing content. Duplicate content artificially magnifies the influence of that part, skewing the model's output towards the repeated content and breaking generative balance.

</details>

**6. What are common strategies for Context Deduplication?**

<details>
<summary>💡 Show Answer</summary>

1. **Exact Deduplication (Hash Matching)**: Calculate the MD5 or SHA-256 hash for each text fragment and maintain a set of existing fragment hashes. Check the set before adding a new fragment; discard it if there is a match. Computational cost is extremely low (O(1) lookup), suitable for handling perfectly identical duplicate fragments (e.g., the same document retrieved twice).

2. **Semantic Deduplication (Vector Similarity)**: Calculate embedding vectors for fragments and judge semantic similarity via cosine similarity. Fragments with a similarity exceeding a threshold (e.g., 0.95) are considered semantic duplicates, and only the one with higher relevance is kept. It can handle situations of "different phrasing but same meaning," but the computational cost is higher than hash deduplication.

3. **Paragraph Merging**: For adjacent fragments from the same document, check if they are contiguous in the original text (via position index). If they are, merge them into a complete paragraph to avoid fragmented repetition and loss of boundary information.

4. **Dialogue-Level Deduplication**: In multi-turn dialogue scenarios, detect whether the content injected in the current turn highly overlaps with historical messages. For example, if a user quotes the assistant's full previous reply in their new message, there is no need to reinject that historical message.

In practice, exact deduplication is usually used as the first pass of filtering (low cost, good effect), followed by semantic deduplication on the remaining fragments, forming a two-stage deduplication pipeline.

</details>

**7. How to determine if two retrieved fragments are "semantically duplicate" in RAG?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to determine if two retrieved fragments are "s](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig58_how_to_determine_if_two_retrie.png)



The core of semantic duplication judgment is vector similarity calculation. In practice, a **greedy deduplication** flow is adopted:

\begin{lstlisting}[style=promptstyle]
	from sklearn.metrics.pairwise import cosine_similarity

	def is_duplicate(emb1, emb2, threshold=0.95):
	sim = cosine_similarity([emb1], [emb2])[0][0]
	return sim > threshold

	def greedy_dedup(chunks, embeddings, threshold=0.95):
	selected = []
	selected_embs = []
	for chunk, emb in zip(chunks, embeddings):
	if all(not is_duplicate(emb, s_emb, threshold)
	for s_emb in selected_embs):
	selected.append(chunk)
	selected_embs.append(emb)
	return selected
\end{lstlisting}

Detailed process: Sort the retrieval results in descending order by relevance score. Iterate through each fragment, checking its maximum cosine similarity against all fragments in the already-selected set before adding. Skip it if the threshold is exceeded. This ensures that in the final fragment set selected for the context, the semantic similarity between any two fragments is below the threshold.

The choice of threshold needs to be adjusted based on the scenario:
- **Above 0.95**: Only removes almost identical fragments, preserving subtle differences.
- **0.85-0.95**: Removes highly similar fragments, suitable for general RAG scenarios.
- **Below 0.85**: Aggressive deduplication, may lose valuable supplementary information, use with caution.

</details>

**8. What is MMR (Maximal Marginal Relevance)? How does it solve the deduplication problem?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is MMR (Maximal Marginal Relevance)? How does](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig59_what_is_mmr_maximal_marginal_r.png)



MMR is a classic algorithm that balances "relevance" and "diversity" in retrieval results. In each step, it selects the fragment with the highest score from the candidate set to add to the result set:

\begin{lstlisting}[style=promptstyle]
	MMR(d) = \lambda * sim(d, query) - (1-\lambda) * max sim(d, already_selected)
\end{lstlisting}

- **First term** $\lambda \cdot sim(d, query)$: Relevance of the fragment to the original query; higher is better.
- **Second term** $(1-\lambda) \cdot \max sim(d, already_selected)$: Similarity of the fragment to the most similar fragment in the selected set. Acts as a penalty term: the higher it is, the more duplicate it is, lowering the score.
- **$\lambda$ parameter**: A control knob to balance the weights of the two. $\lambda=1$ degrades to pure relevance ranking (no deduplication); $\lambda=0$ degrades to pure diversity selection (ignores relevance); in practice, $\lambda \in [0.5, 0.7]$ is usually chosen.

Compared to simple greedy deduplication, MMR's advantage is that it preserves diversity while deduplicating—it does not simply discard similar fragments, but weighs scores to maximize content diversity with minimal loss of relevance. For example, for two articles on the same topic, if one focuses on pros and the other on cons, MMR will tend to retain both, rather than discarding one just because of high similarity.

</details>

**9. In JIT Loading, how to decide "when" to trigger context loading?**

<details>
<summary>💡 Show Answer</summary>

The choice of trigger timing directly affects system latency and retrieval quality. Common strategies:

- **Trigger before every request**: The simplest and most straightforward strategy. Perform a retrieval at the start of every dialogue turn and inject the results into the current context. Stateless and easy to maintain, suitable for scenarios with frequently updated info or vastly changing queries. Disadvantage: overhead for every turn.

- **Trigger upon intent change**: Through intent classifiers or topic similarity detection, judge whether the topic of the user's current query has changed significantly relative to the previous turn. If the topic is coherent (e.g., continuous follow-ups on the same issue), reuse the previous retrieval results; if the topic switches, re-trigger retrieval. Can effectively reduce repeated retrieval overhead.

- **Active triggering by the model (Tool Call)**: Encapsulate the retrieval capability as a tool, allowing the model to independently decide when supplementary info is needed during generation and pull it actively via tool calls. Highest flexibility, but requires strong tool-calling capabilities from the model and may introduce cumulative latency from multi-turn tool calls.

- **Confidence-based triggering**: Evaluate the confidence of the model's output (e.g., self-scoring or uncertainty detection). If the model determines its own knowledge is insufficient to answer the question, trigger supplementary retrieval and regenerate. Suitable for scenarios requiring extreme accuracy (like medical or legal), but complex to implement and increases overall response time.

The most common in production is "RAG retrieval before every request", combined with **query caching** (caching retrieval results for identical or similar queries for a certain period) to avoid repeated retrieval, balancing real-time performance and efficiency.

</details>

**10. After context deduplication, how to handle potentially unique information in the removed fragments?**

<details>
<summary>💡 Show Answer</summary>

Deduplication is not simply "discarding similar content"; we must prevent the loss of valuable unique information:

1. **Information Difference Extraction**: After determining two fragments are duplicates, do not discard them directly. Compare their contents, extract the differences (e.g., different numbers, different dates, supplementary details), and append the differences in a concise format to the retained fragment.

2. **Independent Retention of Metadata**: Even if the body text of two fragments is highly similar, their metadata (source, author, publish time, version number) may differ and be valuable. Retain metadata references from all sources when deduplicating, allowing the model to perceive the multi-source nature of the information.

3. **Scenario-Adaptive Thresholds**: For high-risk scenarios (medical diagnosis, legal clauses, financial compliance), raise the similarity threshold to 0.99 to only remove almost identical content, maximizing the retention of fragments with subtle differences. For low-risk scenarios (general QA), use lower thresholds (0.85-0.90) to save tokens.

4. **Soft Deduplication (Down-weighting instead of discarding)**: Do not completely discard high-similarity fragments, but lower their weight in ranking so they can still be included in the context if the budget allows, and only be trimmed first if the budget is tight. This is a milder deduplication strategy balancing information completeness and token efficiency.

</details>

**11. In Agent scenarios, how does JIT Context Loading combine with tool calling?**

<details>
<summary>💡 Show Answer</summary>

In Agent architectures, tool calling is naturally the best carrier for JIT loading—the model dynamically decides what information is needed during the reasoning process and proactively initiates tool calls to fetch it:

\begin{lstlisting}[style=promptstyle]
	User: Help me analyze the sales data for the past three months and find the category with the fastest decline.

	Agent Thought: I need to get the sales data first, then perform statistical analysis.
	Action: query_database(table="sales", date_range="last_3_months", group_by="category")
	Observation: [Returns sales data by category]

	Agent Thought: I need to calculate the month-over-month growth rate for each category.
	Action: calculate_trend(data=..., metric="mom_growth_rate")
	Observation: [Returns trend analysis results]

	Agent Final Answer: Based on the data, the category with the fastest decline is...
\end{lstlisting}

The core advantages of this pattern are:
- **Precision On-Demand**: The model only pulls the data genuinely needed for the current reasoning step, rather than preloading the entire database.
- **Dynamic Context Construction**: The return result of each tool call is dynamically appended to the context, forming a reasoning chain that supports multi-step complex tasks.
- **External System Integration**: Through tool calls, the sources of JIT loaded information can be extended to any external system (databases, APIs, file systems), not limited to pre-built vector indexes.

When implementing, note: the cumulative latency of multiple tool calls can be significant. Local caching should be applied to frequently accessed tool results, and a maximum limit on tool calls should be set to prevent reasoning from falling into an infinite loop.

</details>

**12. How to measure the effectiveness of JIT Loading?**

<details>
<summary>💡 Show Answer</summary>

A complete evaluation system for JIT effectiveness should cover three dimensions: efficiency, quality, and reliability:

- **Token Saving Rate**: $\frac{Preloaded Tokens - JIT Tokens}{Preloaded Tokens}$, directly measures the resource efficiency of JIT. It is usually expected to reach 50%-80%.

- **Retrieval Latency**: The additional retrieval time introduced by JIT. Focus on P50 (median latency) and P95 (tail latency). An overly high P95 latency will affect user experience; usually, P95 < 500ms is expected.

- **Task Accuracy**: The accuracy of responses on the same test set compared to the preloading approach. The accuracy of the JIT approach should not be significantly lower than preloading (usually allowing a fluctuation within ±2%).

- **Recall**: Whether the JIT approach missed critical information required for the task. This can be tested by constructing cases where "critical information must come from a specific document" to verify if the retrieval module successfully recalled it.

- **Context Utilization Rate**: The proportion of the injected context that is actually referenced by the model (analyzable via citation tracking or attention weights). A low utilization rate indicates insufficient retrieval precision and the injection of too much irrelevant content.

</details>

**13. What causes a low context window utilization rate? How can JIT and deduplication solve it?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What causes a low context window utilization rate?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig60_what_causes_a_low_context_wind.png)



#### Root Causes of Low Utilization
- **Injection of Irrelevant Content**: Insufficient retrieval precision leading to the recall of a large number of document fragments with low relevance to the query.
- **Accumulation of Duplicate Content**: The same fragment is hit multiple times by retrieval, or identical information repeatedly appears in the dialogue history.
- **Bloated System Prompt**: Writing all business rules and edge cases into a static system prompt, while most rules are not needed for the current query.
- **Coarse-Grained Document Chunking**: The retrieval unit (chunk) is too large. A chunk is hit, but only a small section is relevant to the query, while the rest is "free-riding" redundant content.

#### Solutions
- **Precise JIT Retrieval**: Optimize the embedding model and retrieval strategy to improve the relevance of recalled fragments, reducing irrelevant content injection at the source.

- **Deduplication Pipeline**: Introduce a two-stage filter of hash deduplication + semantic deduplication to clear out redundant content, ensuring every token carries unique information.

- **Modular System Prompt**: Split the system prompt into multiple functional modules, dynamically assembling them based on intent routing, injecting only the modules needed for the current query.

- **Content Compression**: Summarize long documents first, injecting the summary instead of the full original text. If the model finds the summary information insufficient, it can pull specific paragraphs of the original text via tool calls.

- **Fine-Grained Chunking**: Reduce the document chunking size (e.g., 256 tokens per chunk) combined with sentence-level embeddings to improve retrieval precision and reduce "free-riding" content.

</details>

**14. How to prevent JIT-retrieved context from contradicting the dialogue history?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to prevent JIT-retrieved context from contradi](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig61_how_to_prevent_jit_retrieved_c.png)



JIT retrieval might bring in outdated documents or those inconsistent with the user's statements, requiring systematic contradiction prevention and handling mechanisms:

1. **Document Timeliness Filtering**: Add time-dimension filtering conditions in the retrieval stage, prioritizing the recall of the latest versions of documents (e.g., "only retrieve content updated within 30 days") to prevent outdated info from entering the context at the source.

2. **Priority Declaration**: Explicitly declare information priority rules in the system prompt. For example: "If retrieved reference documents conflict with facts explicitly stated by the user in this conversation, the user's statements shall prevail; if there are version conflicts within the documents, the latest published time shall prevail."

3. **Explicit Source Attribution**: Attach source information and update time (e.g., "[Source: Product Manual v2.3, Updated: 2024-11]") when injecting each retrieved fragment, allowing the model to perceive the timeliness of the info during generation and inform the user if the document might be outdated.

4. **Pre-Injection Consistency Check**: Before injecting retrieval results into the context, use rules or a lightweight model to quickly check for obvious contradictions (like numerical conflicts or opposing facts) with the current dialogue history. If a contradiction is detected, prioritize prompting the user for confirmation rather than directly injecting conflicting info that might mislead the model.

</details>

**15. In large-scale RAG systems, where is the performance bottleneck for context deduplication? How to optimize it?**

<details>
<summary>💡 Show Answer</summary>

#### Performance Bottleneck
The core bottleneck of semantic deduplication lies in the complexity of similarity calculation: comparing $n$ candidate fragments pairwise has a complexity of $O(n^2)$. This is negligible when the number of candidate fragments per request is small (e.g., Top-20), but when deduplicating an entire corpus (millions of fragments) offline, $O(n^2)$ complexity is completely unfeasible. Additionally, calculating embeddings for each fragment itself has a certain inference cost.

#### Optimization Solutions

- **Locality Sensitive Hashing (LSH)**: Map high-dimensional embedding vectors into low-dimensional buckets via hash functions, where similar vectors have a high probability of falling into the same bucket. Only perform exact similarity calculations on fragments within the same bucket, reducing the overall complexity from $O(n^2)$ to near $O(n)$, trading a small loss of precision for a magnitude of performance boost.

- **Offline Pre-Clustering Deduplication**: Perform clustering on the full corpus during knowledge base construction (offline phase), grouping semantically similar documents into the same cluster, and completing deduplication within the clusters in advance. During online retrieval, directly recall results from the deduplicated corpus, eliminating online deduplication overhead entirely.

- **Two-Stage Filtering Pipeline**: First use hash exact deduplication ($O(1)$ lookup, extremely low cost) to handle perfectly identical fragments, then perform semantic deduplication on the subset that passes exact deduplication. Because exact duplicates usually account for a high percentage, the candidate set size for the second-stage semantic deduplication is massively reduced, significantly lowering overall processing time.

- **Limiting Candidate Set for Online Deduplication**: In real-time RAG requests, only perform semantic deduplication on the Top-K retrieval results (e.g., Top-20), not the full index. Pairwise comparison of Top-20 requires only 190 operations, which can be completed in milliseconds and won't become a performance bottleneck.

</details>

**16. When JIT Loading fails (retrieves irrelevant content), how should the model behave? How to design fallback mechanisms?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of When JIT Loading fails (retrieves irrelevant conte](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig62_when_jit_loading_fails_retriev.png)



#### Identifying Failure Scenarios
Retrieval failure usually manifests as: the relevance scores of the Top-K results returned by vector retrieval are all below a preset threshold (e.g., cosine similarity < 0.6), indicating that no content matching the current query exists in the knowledge base.

#### Fallback Mechanism Design
1. **Relevance Threshold Filtering**: Set a minimum relevance entry threshold for each retrieved fragment. Fragments below the threshold are absolutely not injected into the context, adhering to the principle of "better nothing than garbage." This avoids "injecting irrelevant content that misleads the model," which is better than forcefully injecting low-quality content.

2. **Explicit Declaration of Model Behavior**: Pre-declare the behavioral guidelines for retrieval failures in the system prompt, e.g., "If the provided reference materials are insufficient to support the answer, directly tell the user 'No relevant materials found,' and do not fabricate answers based on content outside the reference materials." This internalizes the fallback behavior into the model's instruction following, rather than relying on its spontaneous judgment.

3. **Multi-Path Retrieval to Boost Recall**: Run **Keyword Retrieval (BM25)** and **Vector Semantic Retrieval** channels in parallel, taking the union of their results before deduplication and relevance filtering. The two are complementary: BM25 excels at exact keyword matching (for proper nouns, code, etc.), while vector retrieval excels at semantic understanding (for synonyms, generalized expressions). Hybrid recall can significantly reduce the retrieval failure rate.

4. **Graceful Degradation Strategy**: When retrieval indeed fails and no valid content can be recalled, the system falls back to answering based on the model's own parametric knowledge, and explicitly annotates in the response: "No relevant documents found. The following answer is based on the model's general knowledge; please verify." This ensures the system still provides a valuable response in edge cases of knowledge base absence, rather than just throwing an error or returning empty.

</details>

**17. How to design a "Context Budget Allocation" system to reasonably distribute token quotas for JIT Loading?**

<details>
<summary>💡 Show Answer</summary>

Context budget allocation is an engineering system for fine-grained management of a limited token window based on priority and usage. Taking a 128k token window as an example:

\begin{lstlisting}[style=promptstyle]
	Total Budget: 128k tokens
	|-- System Prompt: Fixed 2k (Role definition, guidelines, format specs)
	|-- Dialogue History: Dynamic, up to 20k (Rolling summary management, early history compressed when exceeded)
	|-- Retrieved Documents: Dynamic, up to 60k (JIT allocation)
	|   |-- Core Relevant Fragments: 40k (High priority, needed to directly answer query)
	|   +-- Supplementary Context: 20k (Low priority, background knowledge, discarded if budget is tight)
	|-- Structured State: Fixed 1k (Agent task state, variables, etc.)
	+-- Reserved for Output: 45k (Leaving ample space for model generation)
\end{lstlisting}

#### Implementation Key Points

- **Priority Sequential Filling**: Fill sequentially in the order of System Prompt $\to$ Structured State $\to$ Dialogue History $\to$ Core Retrieved Fragments $\to$ Supplementary Context. If the total budget is exceeded, start trimming from the lowest priority.

- **Dialogue History Rolling Compression**: When historical messages exceed their quota, apply Progressive Summarization to early history: first compress the earliest N turns into a summary paragraph, freeing up token space for new messages.

- **Dynamic Budget Adjustment**: Dynamically adjust quotas for each module based on the task type. For example, analytical tasks need more space for retrieved documents; creative tasks produce longer outputs and need more tokens reserved for generation.

- **Budget-Aware Retrieval**: Be aware of the remaining budget during the JIT retrieval stage and dynamically adjust the number of recalled fragments (e.g., if 30k tokens are currently available, only recall content amounting to ~30k tokens, rather than fixing a Top-K and truncating later).

</details>

**18. How to balance context deduplication with information diversity?**

<details>
<summary>💡 Show Answer</summary>

Overly aggressive deduplication loses multi-perspective information, while insufficient deduplication wastes tokens. Balancing the two requires fine-grained strategy design:

- **Tiered Deduplication Strategy**: Use a strict threshold (e.g., 0.90) for content from the **same source** to strictly avoid injecting the same document multiple times; use a relaxed threshold (e.g., 0.97) for content from **different sources** to retain divergent perspectives from different authors or institutions on the same topic.

- **Protection of Opinion Diversity**: For opinion and evaluation content, introduce sentiment/stance analysis in addition to similarity calculations. Even if two articles have high text similarity (e.g., both discussing a policy), if their sentiment leanings or core stances differ (one supports, one opposes), forcefully retain both and do not trigger deduplication.

- **Topic Coverage Completeness Check**: After deduplication, perform a topic coverage check on the final selected fragments—extract the main topic clusters of the original Top-K results and verify if the deduplicated result set still covers all topic clusters. If content for a specific topic cluster is entirely removed by deduplication, forcefully retain the highest relevance fragment from that cluster.

- **User-Controllable Deduplication Intensity**: Expose a `dedup_threshold` parameter, allowing callers to adjust the deduplication aggressiveness based on specific business scenarios. High-accuracy scenarios (medical, legal) can set higher thresholds (retaining more content), while high-efficiency scenarios (general QA) can set lower thresholds (aggressive deduplication).

</details>

**19. How is JIT context loading handled in Streaming scenarios?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How is JIT context loading handled in Streaming sc](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig63_how_is_jit_context_loading_han.png)



Streaming requires the first token to appear as quickly as possible (low Time to First Byte, TTFB), whereas JIT retrieval must complete before generation, creating a natural latency conflict. Main solutions:

- **Parallel Pre-Retrieval**: When the user is typing (at the moment they pause), immediately trigger a pre-retrieval based on the current input, executing in parallel with the user's "thinking time before submission." When the user clicks send, the retrieval results might already be ready, hiding the retrieval latency within the user interaction latency.

- **Speculative Loading**: Based on dialogue history, predict the user's most likely next topic or query, pre-trigger retrieval of relevant content, and cache the results. If the prediction hits, the cached retrieval results can be used directly when the user asks, resulting in zero retrieval latency.

- **Multi-Stage Generation**: Split the generation process into two stages—the first stage rapidly generates a preliminary answer based on the model's parametric knowledge (extremely low latency, streaming starts immediately); after retrieval completes in the second stage, generate citation annotations and source references, incrementally appending them to the first stage's output. Suitable for scenarios extremely sensitive to TTFB.

- **Streaming Retrieval and Incremental Generation**: Start generation as soon as the retrieval system returns the first high-confidence fragment. Subsequently retrieved fragments are dynamically appended to the context, influencing the generation of later tokens. This requires special inference frameworks to support incremental context updates, which is highly complex to implement but achieves both low latency and high quality.

</details>

**20. In multi-modal scenarios (text and image mixed), what are the special challenges of JIT context loading and deduplication?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of In multi-modal scenarios (text and image mixed), w](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig64_in_multi_modal_scenarios_text_.png)



Multi-modal scenarios face special difficulties in JIT loading and deduplication that text-only scenarios do not:

#### Main Challenges
- **High Token Consumption for Images**: A medium-resolution image might consume 1000-2000 tokens (depending on the model's visual encoding strategy), which is several to tens of times that of a text fragment. If all relevant images are preloaded, the token budget is rapidly exhausted. JIT loading is especially crucial in multi-modal scenarios.

- **Difficulty in Cross-Modal Semantic Deduplication**: A product image and a text description of that product highly overlap semantically, but this overlap cannot be detected via text hashes or pure text embeddings. Cross-modal embedding models (like CLIP) are needed to map images and text into a unified vector space for cross-modal similarity calculations.

- **Asymmetrical Retrieval Granularity**: Text can be finely chunked by sentence or paragraph, but an image is an indivisible whole. You can inject just one paragraph of text, but an image must be injected fully, leading to fundamental asymmetry in retrieval granularity.

- **Complex Multi-Modal Relevance Judgment**: When the user's query itself is a mix of text and image (e.g., uploading an image with a text description), relevance judgment requires understanding the combined semantics of the image and text. Single-modal retrieval models are inadequate; multi-modal retrieval models are required.

#### Solution Concepts
- **Textualized Surrogate Retrieval for Images**: Generate high-quality text descriptions (captions or alt text) for each image and use these descriptions to build a retrieval index. During JIT retrieval, first locate relevant images via the text index, and after confirming relevance, inject the original image into the context. This transforms the multi-modal retrieval problem into a text retrieval problem, reusing mature text JIT infrastructure.

- **Unified Multi-Modal Vector Space**: Use CLIP or similar multi-modal embedding models to map both images and text into the same vector space, supporting "text query retrieving images" and "cross-modal deduplication" (detecting semantic overlap between an image and the text describing it), fundamentally solving the cross-modal similarity calculation problem.

</details>

