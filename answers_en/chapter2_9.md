# 2.9 Prompt Caching and Context Engineering — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 20 questions

**1. What is KV Cache? What role does it play in LLM inference?**

<details>
<summary>💡 Show Answer</summary>

KV Cache (Key-Value Cache) is the core optimization mechanism in Transformer inference. Understanding its role requires starting from the computational principles of the self-attention mechanism.

#### The Problem of Repeated Computation in Self-Attention

In the self-attention layer of a Transformer, each token's representation is determined by three matrices: Query ($Q$), Key ($K$), and Value ($V$). The attention output calculation formula is:


$$
	Attention(Q, K, V) = softmax\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$


During autoregressive generation, the model generates only one new token at a time, but to calculate the attention weight of that token, it needs to perform dot products with the $K$ and $V$ matrices of **all preceding tokens**. If not optimized, at generation step $t$, the $K$ and $V$ for the previous $t-1$ tokens must be recalculated, resulting in a total computational complexity of $O(n^2)$, growing quadratically with sequence length.

#### How KV Cache Works

The core idea of KV Cache is: **the $K$ and $V$ matrices of already generated tokens will not change in subsequent steps** (because those tokens are fixed), so they can be cached in VRAM (Video RAM) and directly read in subsequent steps without recalculation.

The specific flow is:
1. **Prefill Phase**: The model processes the entire input prompt at once, computes the $K$ and $V$ matrices for all tokens, and stores them in the KV Cache;
2. **Decode Phase**: For each new token generated, compute only the $Q$, $K$, and $V$ for that token, append the new $K$ and $V$ to the Cache, and perform attention calculation with the historical Cache;
3. Loop this process, reducing the computational complexity per step from $O(n)$ to near $O(1)$ (only computing for the one new token).

#### Practical Benefits

- **Latency Reduction**: The computation per step in the Decode phase is massively reduced, significantly increasing token generation speed (throughput tokens/s);
- **Long Inputs Benefit More**: The longer the input, the larger the redundant computation without Cache, meaning a higher acceleration ratio from the Cache;
- **Cost**: KV Cache consumes a massive amount of GPU VRAM. For instance, LLaMA-3 70B can take tens of GBs for a 128k token KV Cache. Memory pressure is the core bottleneck limiting concurrent long-context inference.

</details>

**2. What is Prompt Caching? How does it relate to KV Cache?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Prompt Caching? How does it relate to KV C](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig65_what_is_prompt_caching_how_doe.png)



Prompt Caching is a feature where API providers **expose KV Cache capabilities across requests** to users. When the prefix of a prompt remains unchanged across multiple requests, the API backend caches the KV state of that prefix. Subsequent requests directly reuse it, skipping the Prefill computation for the prefix.

#### Relationship with KV Cache

The essence of both is the same; the difference lies in their scope:

- **KV Cache**: Valid **within a single generation**, reusing the KV results from the Prefill phase during the Decode phase, and freed immediately after the request ends;
- **Prompt Caching**: Extends the lifespan of KV Cache **across multiple API calls**. The backend persists the KV state of specific prefixes (usually a 5-minute TTL). If different requests share the same prefix, they can all reuse the same KV Cache.

It can be understood as: Prompt Caching = Cross-request persisted KV Cache. It is the engineering extension of KV Cache on the server side.

#### Trigger Mechanism

Prompt Caching is typically triggered by users explicitly marking cache breakpoints in API requests (e.g., Anthropic's `cache_control` field). During the first processing, the server writes the KV state before the breakpoint to cache storage. When subsequent requests are detected with matching prefixes, it loads directly, skipping the Prefill computation.

</details>

**3. What specific benefits does Prompt Caching bring?**

<details>
<summary>💡 Show Answer</summary>

- **Latency Reduction**: When there's a cache hit, the prefix tokens skip the Prefill calculation, significantly lowering the Time To First Token (TTFT). For a long document prefix of 80k tokens, a cache hit can compress the TTFT from tens of seconds to under a second;

- **Cost Reduction**: Platforms like Anthropic charge about 10% of the original price for cache-hit tokens, which means a ~90% saving on prefix calculation costs. For example, with a fixed 10k token prefix per request and 10,000 requests per day, monthly token costs can be reduced by thousands of dollars;

- **Throughput Increase**: The saved GPU computational resources can be used to serve more concurrent requests, improving overall system throughput under the same hardware budget.

The specific benefits depend on two core variables: **Prefix Length** (the longer it is, the more saved per hit) and **Cache Hit Rate** (the higher it is, the greater the overall benefit). When both are high, the ROI of Prompt Caching is extremely significant.

</details>

**4. What kind of content is best suited to be placed in a cacheable prefix?**

<details>
<summary>💡 Show Answer</summary>

Content suitable for caching has the following characteristics: **static, fixed, and shared by multiple requests**. Typical contents include:

- **System Prompt**: Role definitions, behavioral guidelines, output formatting rules, and other content that rarely changes throughout the product's lifecycle;
- **Tool/Function Definitions**: Tool schemas in Agent systems, usually containing JSON descriptions for dozens of tools, which have a large and fixed token footprint;
- **Long Documents**: Knowledge base docs, codebases, legal contracts, technical manuals, etc., injected as static text for background knowledge;
- **Few-shot Example Sets**: Fixed examples in the In-Context Learning (ICL) example pool, especially CoT examples with long reasoning chains.

**Content NOT suited for caching**: User IDs, current timestamps, real-time account statuses, personalized variables, and other dynamic content—placing this content in the prefix causes the prefix to change every request, resulting in continuous cache misses. This not only yields no benefits but also incurs additional cache writing overhead.

</details>

**5. What are the design principles of Context Engineering when using Prompt Caching?**

<details>
<summary>💡 Show Answer</summary>

Core principle: **Stable content upfront, dynamic content at the back**.

\begin{lstlisting}[style=promptstyle]
	[Cacheable Prefix]
	|-- System Prompt (Role, rules)         <- Most stable, highest hit rate
	|-- Tool Definitions (Tool Schema)
	|-- Background Docs (Knowledge base, codebase)
	+-- Few-shot Examples

	[Dynamic Part, Not Cached]
	|-- Dialogue History (New every turn)
	+-- User's Current Message
\end{lstlisting}

As long as the prefix part does not experience any token-level changes, the cache will hit, saving the entire Prefill calculation overhead of the prefix.

In addition, follow these auxiliary principles:
- **Minimize Breakpoints**: The number of cache breakpoints shouldn't be too many (usually 1-2); too many breakpoints increase management complexity;
- **Place Breakpoint at the End of the Longest Stable Prefix**: The further back the breakpoint, the more tokens are saved per hit;
- **Avoid Implicitly Dynamic Content in the Prefix**: E.g., current dates or random seeds that seem fixed but might change with the deployment environment.

</details>

**6. How to enable Prompt Caching in the Anthropic API?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to enable Prompt Caching in the Anthropic API?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig66_how_to_enable_prompt_caching_i.png)



By adding a `cache_control` field inside the content block to mark cache breakpoints:

\begin{lstlisting}[language=Python, style=promptstyle]
	messages = [
	{
		"role": "user",
		"content": [
		{
			"type": "text",
			"text": "Here is a long document content (80k tokens)...",
			"cache_control": {"type": "ephemeral"}  # Cache breakpoint
		},
		{
			"type": "text",
			"text": "User's specific question"                 # Not cached
		}
		]
	}
	]
\end{lstlisting}

The position marked by `cache_control` is the cache breakpoint; all content **before** this position will be cached. The API returns a `usage` field containing the following fields for monitoring cache status:
- `cache_creation_input_tokens`: Number of tokens newly written to the cache for this request (first time, full price);
- `cache_read_input_tokens`: Number of tokens read from the cache for this request (hit, approx. 10% price).

</details>

**7. What are the trigger conditions and limitations of Prompt Caching?**

<details>
<summary>💡 Show Answer</summary>

**Trigger Conditions (using Anthropic as an example):**
- Cache prefix length $\geq$ 1024 tokens (2048 tokens for claude-haiku); caching is not triggered below this threshold;
- The interval between requests must be within the cache TTL (about 5 minutes); after timeout, the cache automatically expires;
- Prefix content must be **exactly identical** to when it was cached (any single token change invalidates the cache, including spaces, punctuation, and encoding differences).

**Main Limitations:**
- **First Write Billed at Full Price**: When the cache is established (`cache_creation`), it's still billed at the normal rate; savings start from the second hit;
- **Cache Warm-up Problem**: In high-concurrency scenarios, multiple requests might simultaneously trigger a cache write (before the cache is established), causing concurrent cold starts;
- **Security Isolation**: Caches are strictly isolated between different API Keys. Requests from different users/tenants cannot share caches (even if prefixes are identical);
- **Short TTL**: The 5-minute TTL is practically ineffective for low-frequency request scenarios (e.g., hourly batch jobs).

</details>

**8. How do Prompt Caching and RAG work together?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How do Prompt Caching and RAG work together?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig67_how_do_prompt_caching_and_rag_.png)



The two are not mutually exclusive, but rather complementary. Depending on knowledge base size and access patterns, there are two main collaboration models:

**Model 1: Document Preloading + Caching (Cache-First)**

Inject entire knowledge documents into the prompt prefix and mark cache breakpoints. Suitable for scenarios with few documents (single document $\leq$ 200k tokens) that are queried at high frequency (e.g., product manuals, API docs). Each time a user asks a different question, the prefix remains unchanged, ensuring continuous cache hits. Yields the best results.

**Model 2: RAG Retrieval + Caching (RAG-First)**

Dynamically retrieve the most relevant document fragments and inject them into the prompt. Suitable for large knowledge bases (cannot be fully injected). Optimization point: if the same document fragment is retrieved and hit multiple times, cache breakpoints can also be marked on that fragment to reuse the KV Cache at the fragment level. The effect is weaker than Model 1, but it has broader applicability.

**Selection Criteria:** Use Model 1 if knowledge base documents are few and fixed; use Model 2 if document volume is large and dynamically updated frequently; both models can be combined (preload high-frequency docs, RAG-retrieve low-frequency docs).

</details>

**9. What problems arise if the Cache TTL is too short? How to deal with it?**

<details>
<summary>💡 Show Answer</summary>

An overly short TTL (e.g., 5 minutes) causes continuous cache cold starts for low-frequency request scenarios. Every request will have to recalculate the prefix, missing all cache benefits entirely, and cache writing itself brings slight extra overhead.

**Mitigation Strategies:**
- **Active Cache Warming**: Deploy a scheduled task (e.g., every 4 minutes) to send a minimal request containing the cache prefix to keep the cache alive; suitable for predictable low-frequency scenarios;
- **Request Batching**: Aggregate scattered low-frequency requests over time, sending them in bulk within short time windows to increase request density and hit rates;
- **Architecture Adjustment**: For scenarios where cache benefits are marginal (extremely low frequency, short prefixes), don't rely on Prompt Caching. Instead, use other inference optimization methods like model quantization or KV Cache compression;
- **Platform Selection**: Some platforms support longer cache TTLs or manual lifecycle control; choose a provider that better matches your business needs.

</details>

**10. How to measure the actual benefits of Prompt Caching?**

<details>
<summary>💡 Show Answer</summary>

**Core Monitoring Metrics:**
- **Cache Hit Rate**: $Hit Rate = \dfrac{Requests with non-zero cache_read_input_tokens}{Total Requests}$, reflecting the effectiveness of the caching strategy;

- **Token Savings**: $Savings = \sum cache_read_input_tokens \times (P_{original_price} - P_{cache_price})$, directly quantifying cost savings;

- **Latency Improvement**: Compare the TTFT distributions (P50/P95) of cache hits vs. misses to quantify experience improvements;

- **Cost Saving Rate**: $Saving Rate = \dfrac{Total Cost without Cache - Total Cost with Cache}{Total Cost without Cache}$, used for ROI reporting.

Data source is the `usage` field in API responses. It's recommended to ingest `cache_read_input_tokens` and `cache_creation_input_tokens` into a monitoring system, aggregating by hour/day to quickly detect abnormal drops in cache hit rate (which usually implies the prefix was accidentally modified).

</details>

**11. In multi-tenant systems, how to safely share Prompt Caching?**

<details>
<summary>💡 Show Answer</summary>

In multi-tenant scenarios, cache sharing requires fine-grained balancing between **efficiency** (share as much as possible) and **security isolation** (preventing data leaks). A **Tiered Caching Architecture** is recommended:

- **Tier 1 — Global Public Layer** (Shared by all tenants): Generic System Prompts, platform-level tool definitions, etc., containing no tenant info. Can be safely shared across all requests under the same API Key;

- **Tier 2 — Tenant-Specific Layer** (Isolated per tenant): Tenant custom knowledge bases, tenant-level behavior rules, etc., containing tenant-specific info. Cache isolation is achieved by assigning independent API Keys to each tenant (API platforms usually partition cache namespaces by API Key);

- **Tier 3 — User Dynamic Layer** (Not cached): User messages, session variables, real-time states, and content unique to each request. Bypasses caching entirely.

This design allows the Tier 1 cache to be shared by all tenants for maximum hit rates; Tier 2 is independently maintained for security; and Tier 3 bypasses the cache entirely with zero security risks.

</details>

**12. How does Prompt Caching affect Prompt version management strategies?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How does Prompt Caching affect Prompt version mana](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig68_how_does_prompt_caching_affect.png)



Prompt Caching turns prefix stability into an engineering constraint with direct economic costs, profoundly affecting how Prompts are version-managed:

- **Version Change means Cache Invalidation**: Modifying a single character of the System Prompt (including spaces, punctuation) instantly invalidates all established caches, generating full cold-start costs. In high-traffic systems, this could mean extra fees for hundreds of requests;

- **Canary Releases + Cache Warming**: Before rolling out a new Prompt version, use a small fraction of traffic (e.g., 1%) to trigger cache creation. Gradually switch traffic only after cache warming completes, avoiding mass cold starts at the exact moment of version switch;

- **Batch Accumulation of Changes**: High-frequency Prompt iteration causes continuous cache invalidation. Small modifications should be accumulated and released together in batches, rather than deploying each optimization immediately;

- **Tiered Freezing Strategy**: Split the Prompt into a "Stable Core Layer" (cached, cautiously modified) and an "Iterative Expansion Layer" (uncached, freely modified), decoupling their iteration tempos;

- **A/B Testing Costs**: When two Prompt versions run simultaneously, their caches must be warmed up independently. Actual costs are higher than a single version, requiring a trade-off between testing benefits and additional expenses.

</details>

**13. In long Agent conversations, how can Prompt Caching be used to optimize multi-turn reasoning costs?**

<details>
<summary>💡 Show Answer</summary>

Multi-turn reasoning by Agents involves massive repetitive content per request (System Prompt, tool definitions, accumulated dialogue history), making it one of the scenarios where Prompt Caching yields the highest returns.

**Recommended Context Structure:**

\begin{lstlisting}[style=promptstyle]
	Round 1: [System + Tools (Cache Breakpoint) | UserMsg1]
	-> Written to Cache

	Round 2: [System + Tools (Cache Hit) | History1 | UserMsg2]
	-> System+Tools hits cache, only History1+UserMsg2 require new computation

	Round 3: [System + Tools (Cache Hit) | History1+2 | UserMsg3]
	-> System+Tools continues hitting cache, cumulative savings grow with turns
\end{lstlisting}

**Key Design Points:**
- Fix the System Prompt and tool definitions as an unchanging prefix, setting a cache breakpoint at the end of them;
- Dialogue history is appended after the cache breakpoint and is not cached (because it grows every turn);
- If tool definitions are long (e.g., 50+ tools), the savings are exceptionally noticeable—tool definitions might consume thousands of tokens, which are saved every turn;
- For ultra-long conversations, an additional cache breakpoint can be set on the stable early part of the dialogue history (like the Assistant replies from the first $N$ turns) to further expand the cache hit scope.

</details>

**14. Does Prompt Caching affect model output quality?**

<details>
<summary>💡 Show Answer</summary>

Theoretically, **no impact**. The essence of Prompt Caching is skipping the repetitive Prefill computation of prefix tokens and directly loading precomputed KV states. The context semantics "seen" by the model is entirely identical to the uncached state, and the output distribution should theoretically remain identical.

**Potential Risks (Extremely Rare):**
- **Platform Implementation Defects**: If a bug exists in the server's cache storage, corrupted KV states might cause anomalous outputs (extremely rare in practice; mainstream platforms have robust validation mechanisms);
- **Model Hot-Update Conflicts**: If the platform updates model weights online within the cache TTL, there might be incompatibilities between the old KV Cache and new model weights (legitimate platforms usually flush all caches proactively during model updates to avoid this).

**Engineering Validation Recommendation:** After enabling Prompt Caching, periodically run comparison tests on the same inputs between "cache hit" and "forced cold start" to track output consistency. If systemic discrepancies are found, report them to the platform for investigation immediately.

</details>

**15. How to design a Prompt template system to maximize Prompt Caching returns?**

<details>
<summary>💡 Show Answer</summary>

**Tiered Template Architecture:**

1. **Base Layer (Globally Shared, Highest Hit Rate)**: Content universally applied across all business scenarios, such as platform-level role definitions, safety guidelines, and universal output formats. This layer has the lowest change frequency and should have the first cache breakpoint;

2. **Scenario Layer (By Business Type, Medium Hit Rate)**: Tool definitions, domain knowledge, and scenario-specific examples for particular business scenarios. This layer is fixed within the same business line and takes the second cache breakpoint;

3. **Request Layer (Dynamic, Not Cached)**: User messages, personalized variables, real-time context. Unique for every request, placed after all cache breakpoints.

\begin{lstlisting}[style=promptstyle]
	[Layer 1: Global System Prompt]  <- cache_control breakpoint 1 (Highest Hit Rate)
	[Layer 2: Business Scenario Tools] <- cache_control breakpoint 2
	[Layer 3: User History + Current Msg] <- Not cached, dynamically appended
\end{lstlisting}

**Engineering Implementation Points:**
- Use template rendering engines (like Jinja2) to ensure the tokenization result of static layers is exactly consistent (pay attention to whitespace normalization);
- Maintain "token fingerprints" for each template version. Proactively trigger cache warm-ups when a fingerprint change is detected;
- **Strictly limit** dynamic variables to appear after cache breakpoints, prohibiting any dynamic content from infiltrating cacheable layers.

</details>

**16. What are the differences and connections between Prompt Caching and Batch API?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]

	\caption{Comparison of Prompt Caching and Batch API}


| **Dimension** | **Prompt Caching** | **Batch API** |
| --- | --- | --- |
| Optimization Goal | Reduce repeated compute (Latency+Cost) | Batch process to lower cost (Throughput) |
| Latency | Drastically lower when hit | Higher (Asynchronous, minute-level) |
| Applicability | Real-time interaction, high prefix repeat | Offline batch processing, latency-insensitive |
| Cost Savings | ~90% discount for cached tokens | ~50% discount for batches |
| Trigger Method | User tags cache breakpoints | Submit to batch task queue |


\end{table}

**Can be used together:** Enable Prompt Caching concurrently with batch requests. Reusing the KV Cache of common prefixes in batch tasks allows you to enjoy both discounts, reaching an overall cost discount of around 5% of the original price ($10% \times 50%$). Suitable for large-scale offline data processing scenarios, like batch analyzing tens of thousands of documents under the same System Prompt.

</details>

**17. How to still achieve Prompt Caching in dynamic personalization scenarios?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to still achieve Prompt Caching in dynamic per](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig69_how_to_still_achieve_prompt_ca.png)



If dynamic personalized content (like user names, account balances, real-time statuses) is injected into the prefix, the cache will invalidate every time. Architecture design must decouple personalization from caching.

**Option 1: Strictly Append Personalized Content at the End**

Strip all personalized variables from the prefix and move them after the cache breakpoint:

\begin{lstlisting}[style=promptstyle]
	[Cacheable] You are a smart customer service assistant, responsible for helping users solve account issues... (General rules)
	<- cache_control breakpoint
	[Not Cached] Current user: {user_name}, Account status: {user_state}
	User question: {query}
\end{lstlisting}

**Option 2: Dynamically Fetch Personalized Info via Tool Calls**

Change the acquisition of personalized info from "injecting into prefix" to "model actively calling tools to query". The prefix only declares the existence of the tool, not its return value:

\begin{lstlisting}[style=promptstyle]
	[Cacheable] You have a get_user_info(user_id) tool.
	Please call it when you need to fetch user info.
	<- cache_control breakpoint
	[Not Cached] User ID: {user_id}
	User question: {query}
\end{lstlisting}

During inference, the model actively calls the tool to fetch personalized data. The prefix stays constant, and the cache continues to hit. An additional benefit of this approach is more precise timing for information retrieval (on-demand querying) and easier permission control/auditing of tool call results.

</details>

**18. How to quantitatively evaluate "whether it is worth enabling Prompt Caching for a certain scenario"?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to quantitatively evaluate "whether it is wort](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig70_how_to_quantitatively_evaluate.png)



**ROI Decision Framework:**


$$
	Single Saving &= N_{prefix} \times (P_{original} - P_{cache}) \\
	Write Cost &= N_{prefix} \times P_{original} \times (1 + \delta) \\
	Breakeven Hits &= \left\lceil \frac{Write Cost}{Single Saving} \right\rceil \approx \frac{1 + \delta}{1 - P_{cache} / P_{original}} \approx 1.1  hits
$$


Where $N_{prefix}$ is the number of prefix tokens, $\delta$ is the cache writing markup rate (about 25% for Anthropic), and $P_{cache} / P_{original} \approx 0.1$.

In practice, **about 2 hits are enough to break even**. Therefore, as long as the same prefix is called more than twice within the TTL, enabling caching yields net benefits.

**Conditions for Enabling (recommend enabling if any two are met):**
- Prefix token count $\geq$ 1024 (platform minimum requirement);
- Expected hits for the same prefix within the TTL (5 minutes) $\geq$ 3;
- High prefix content stability (no frequent modification plans recently);
- System has clear requirements for TTFT latency (real-time interaction scenarios).

</details>

**19. When there is a conflict between Prompt Caching and Prompt optimization iterations, how to choose?**

<details>
<summary>💡 Show Answer</summary>

Nature of the conflict: Prompt optimization requires frequent modification of the prefix, while Prompt Caching benefits rely on the prefix remaining unchanged—the two have diametrically opposed needs regarding the prefix.

**Trade-off Strategies by Product Stage:**
- **Development/Exploration Stage**: Do not rely on Prompt Caching at all. Iterate Prompts freely, prioritizing effect optimization. Low cache hit rates are expected results, not problems;

- **Effect Stabilization Stage**: Freeze the core System Prompt, introduce caching, and shift the iteration focus to the dynamic layer that bypasses caching (like dynamic selection strategies for Few-shot examples);

- **Production Maturity Stage**: Implement a dual-track system of "Stable Core Layer + Iterative Expansion Layer." Cautiously modify the core layer and pass it through a canary warm-up process, while freely iterating the expansion layer without cache constraints;

- **Emergency Hotfix Scenario**: If the core prefix must be immediately modified (e.g., security vulnerability, critical error), accept the one-time cold start cost. Post-fix, warm up the cache as soon as possible, and notify downstream systems in advance of expected latency spikes if necessary.

</details>

**20. In ultra-long context (e.g., 100k+ tokens) scenarios, what are the best practices for Prompt Caching?**

<details>
<summary>💡 Show Answer</summary>

**Scenario Example:** Injecting an entire technical manual (80k tokens) into the prompt, and the user queries it repeatedly.

**Best Practices:**
1. **Single Cache Breakpoint**: Set one cache breakpoint at the end of the document, treating the entire document as a single caching unit, avoiding management complexities of multiple breakpoints;

2. **Ensure Document Tokenization Consistency**: The document content processed by the tokenizer must be perfectly identical every time. Note: file encoding (UTF-8/GBK), line endings (`\textbackslash n` vs `\textbackslash r\textbackslash n`), leading/trailing whitespaces, etc., can all cause token sequence discrepancies. It is recommended to store the tokenized result of the standardized document;

3. **Strictly Append User Questions at the End**: All user questions and dialogue histories are placed after the cache breakpoint. Never insert any dynamic variables inside the document content;

4. **Proactive Warm-up on Service Startup**: Send a "warm-up request" when the service instance starts to establish the cache, ensuring the first batch of real user requests hits the cache and avoiding cold-start impacts on user experience;

5. **Monitor Cache Effectiveness**: Continuously monitor `cache_read_input_tokens`, which should be close to the document token count ($\approx$ 80k). If this value remains abnormally low, it usually means the document content or prefix structure has accidentally changed and requires immediate troubleshooting.

**Cost Calculation (80k token document, 1000 user queries):**

\begin{table}[h]

	\caption{Prompt Caching Cost Comparison (80k token document)}


| **Solution** | **Equivalent Billed Tokens** | **Relative Cost** |
| --- | --- | --- |
| No Cache (Full calculation every time) | $80k \times 1000 = 80,000,000$ | 100% |
| With Cache (First write + 999 hits) | $80k + 80k \times 10% \times 999 \approx 8,800,000$ | $\approx 11%$ |
| **Savings** | $\approx 71,200,000 tokens$ | **$\approx$ 89%** |


\end{table}

In this scenario, Prompt Caching is currently the most cost-effective single-point optimization tool, achieving nearly 90% cost savings with almost zero architecture transformation cost.

</details>

