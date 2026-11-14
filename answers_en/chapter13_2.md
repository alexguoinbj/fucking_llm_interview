# 13.2 Retrieval Architectures and Loss Functions — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 13: Embedding Model Training · 28 questions

**1. What is the Difference Between Dual Encoders and Cross-Encoders? If You Need to Build a Large-Scale Semantic Search Engine, Which Architecture Would You Prioritize?**

<details>
<summary>💡 Show Answer</summary>

**Dual Encoder / Bi-Encoder**:

- Uses the same (or two separate) encoders to encode the query and document independently, obtaining a fixed-dimensional vector representation for each;
- Measures relevance by computing the similarity (e.g., cosine similarity, dot product) between the two vectors;
- The encoding processes of the query and document are **independent and mutually decoupled**—document vectors can be **pre-computed offline and indexed** (e.g., using Approximate Nearest Neighbor (ANN) indices such as FAISS, HNSW, etc.);
- During online retrieval, the query needs to be encoded only once, followed by a fast search via the ANN index to retrieve the top most similar document vectors. The retrieval speed is extremely fast (millisecond-level, scalable to millions or billions of documents);
- Disadvantages: Because the query and document are encoded independently, token-level cross-attention cannot be performed during encoding. As a result, its ability to model fine-grained semantic matching (such as multi-hop reasoning or exact wording matching) is relatively weak, and its relevance assessment accuracy is generally lower than that of Cross-Encoders.

**Cross-Encoder**:

- Concatenates the query and document into a single input sequence and feeds them together into the same Transformer model, allowing full token-level interaction between the query and document through the self-attention mechanism;
- Outputs a scalar score (e.g., via a classification head) that directly represents the relevance score for the query-document pair;
- Because each inference requires joint encoding of the query and a specific document, **document representations cannot be pre-computed or cached**. Every new (query, document) pair requires running a complete Transformer forward pass, making the computational cost vastly higher than that of a Dual Encoder;
- Advantages: Due to comprehensive token-level interaction, the accuracy of relevance assessment is typically significantly higher than that of Dual Encoders.

**Architecture Choice for Large-Scale Semantic Search Engines**: For scenarios requiring real-time retrieval across massive document collections (millions to billions), **a Dual Encoder should be prioritized as the first stage (retrieval stage)**. The reasons are:

- Dual Encoders can pre-compute vectors for all documents offline and build ANN indices, offering low latency and high scalability during online retrieval;
- Cross-Encoders cannot pre-compute representations; scoring all candidate documents individually for every query would incur an unacceptable computational burden ($O(N)$ full Transformer forward passes, where $N$ is the total number of documents).

In practice, a **two-stage architecture (Retrieve-and-Rerank)** is typically adopted:

1. **First Stage (Retrieval)**: Use a Dual Encoder to quickly retrieve top-$k$ (e.g., $k=100$) candidates from the massive document corpus;
2. **Second Stage (Reranking)**: Use a Cross-Encoder to compute more precise relevance scores for these $k$ candidates paired with the query, rerank them, and output the final result.

This combination leverages both the efficiency and scalability of Dual Encoders and the high accuracy of Cross-Encoders, making it the standard design pattern for large-scale semantic search engines.

</details>

**2. Besides Dual Encoders and Cross-Encoders, What Other Retrieval Architectures Exist? How Does a Multi-Vector Representation (e.g., ColBERT) Work, and What Are Its Advantages and Disadvantages Compared to Single-Vector Dual Encoders?**

<details>
<summary>💡 Show Answer</summary>

Besides single-vector Dual Encoders (Bi-Encoder) and Cross-Encoders requiring joint encoding, there also exists a **multi-vector (late interaction) architecture** positioned between the two, represented by work like ColBERT.

**1. Core Idea**

- Dual Encoders compress the entire query and document into **a single** fixed-length vector each, and then compute the similarity between the two vectors (e.g., cosine similarity). This compression process loses fine-grained token-level information;
- Multi-vector methods like ColBERT retain the encoded vector for **every token** in the query and document (rather than pooling them into a single vector). During retrieval, for each token vector in the query, the method finds the most similar token vector in the document (the MaxSim operation), and then sums the maximum similarities across all query tokens to form the overall relevance score for the query-document pair;
- This approach of "encoding separately first, then conducting fine-grained interaction" is known as **Late Interaction**, positioned between Dual Encoders (no interaction, compressed into single vectors early on) and Cross-Encoders (interaction starting from the input layer).

**2. Advantages Over Dual Encoders**

- Retains token-level matching information, resulting in higher retrieval precision than single-vector Dual Encoders in keyword-sensitive scenarios (e.g., queries containing proper nouns, code snippets, or rare entities), where a single vector often "averages out" these critical details during compression;
- Compared to Cross-Encoders, document-side token vectors can still be pre-computed offline and indexed. During the online phase, only the query needs to be encoded, followed by MaxSim calculations between vectors. Thus, it achieves a better balance between performance and efficiency than Cross-Encoders, while still supporting large-scale retrieval (when paired with specialized approximate search indices like PLAID).

**3. Costs Compared to Dual Encoders**

- Significantly higher storage costs: A vector must be stored for every token in a document (rather than storing a single vector per sentence), making the index footprint potentially tens of times larger than single-vector methods;
- Higher computational and engineering complexity during retrieval: Requires custom-designed Approximate Nearest Neighbor (ANN) search and post-processing workflows (e.g., first retrieving candidates with a lightweight method, then performing exact MaxSim reranking on candidates), preventing direct reuse of standard single-vector ANN retrieval libraries.

In summary, Dual Encoders, Multi-Vector models (ColBERT-style late interaction), and Cross-Encoders form an "efficiency vs. accuracy" continuum: Dual Encoders are the fastest but suffer the greatest information loss; Cross-Encoders are the most accurate but cannot be pre-computed offline and carry the highest cost; Multi-Vector approaches strike a trade-off by retaining token-level vectors, typically applied in scenarios demanding higher retrieval precision and able to afford higher storage costs. In practice, they are also often deployed as a second-stage reranking option following an initial Dual Encoder retrieval.

</details>

**3. What Are the Advantages and Disadvantages of Sparse Retrieval (e.g., BM25/SPLADE) vs. Dense Retrieval (Embedding-Based Retrieval)?**

<details>
<summary>💡 Show Answer</summary>

Sparse retrieval and dense retrieval represent two parallel technical paradigms in text retrieval. Understanding their fundamental differences is essential for designing retrieval systems:

- **Sparse Retrieval**: Represented by BM25, it essentially scores text based on exact lexical matching (combined with statistical signals such as term frequency and inverse document frequency). Vector representations are high-dimensional and sparse (dimensionality equals vocabulary size, with the vast majority of dimensions being zero). Its advantages include highly reliable matching for keywords, proper nouns, and precise terminology (without missing matches due to synonyms or differing vocabulary), strong interpretability, and zero training requirement, making it a longstanding foundation of industrial search engines. Its disadvantages lie in its inability to handle semantic synonyms, paraphrases, or cross-lingual matching (if user queries and documents differ in wording, matching fails);
- **Dense Retrieval**: Refers to semantic retrieval based on embedding models, which encodes queries and documents into low-dimensional dense vectors and retrieves via vector similarity. Its advantage lies in capturing semantic-level relevance, demonstrating strong generalization for paraphrases and cross-lingual scenarios. Its disadvantages include potential unreliability for exact keyword/term matching (e.g., product model numbers or proper nouns, where dense vectors sometimes "blur" key details due to an over-focus on global semantics), as well as requiring training data and compute resources;
- **Complementarity of Both**: Sparse retrieval excels at exact lexical matching, while dense retrieval excels at semantic generalization. They often exhibit complementary failure modes, which explains why hybrid retrieval schemes are widely adopted.

In summary, sparse retrieval and dense retrieval represent two distinct retrieval paradigms—"exact lexical matching" and "semantic similarity matching." Each has its applicable scenarios and limitations. Real-world production systems rarely rely on just one in isolation, preferring to combine them to leverage their respective strengths.

</details>

**4. How Are Learned Sparse Representation Methods Such as SPLADE Trained? What Improvements Do They Offer Over Traditional BM25?**

<details>
<summary>💡 Show Answer</summary>

SPLADE (Sparse Lexical and Expansion Model) represents a class of methods positioned between traditional sparse retrieval and dense retrieval: using sparse vector representations learned by neural networks rather than relying on fixed statistical formulas like BM25:

- **Core Idea**: Retains the form of sparse vectors (where vector dimension equals vocabulary size), but the weight of each dimension is no longer computed by a term-frequency formula. Instead, a neural network head (similar to a masked language model) predicts an "importance score" for each position in the input text across all vocabulary words, which is then pooled (e.g., via max pooling) into a sparse weight vector for the entire text;
- **Term Expansion Capability**: Unlike BM25, which can only match words **actually present** in the text, SPLADE's neural head can assign non-zero weights to words that do not appear in the text but are semantically related (e.g., for an input "automobile," the model might assign weights to related terms like "vehicle" or "car"). This implicitly performs query/document expansion, mitigating the traditional sparse retrieval limitation with synonyms;
- **Training Methodology**: Similar to dense retrieval, SPLADE is trained using contrastive learning (such as InfoNCE), except similarity is computed via the dot product of sparse vectors. Additionally, a regularization term (such as FLOPS regularization) is usually added to encourage sparsity and prevent the vectors from degenerating into dense vectors (which would lose sparse retrieval's advantages in storage and retrieval efficiency);
- **Improvements Over BM25**: By incorporating semantic information learned by neural networks, SPLADE retains the exact-matching advantages and efficient inverted-index retrieval capabilities of sparse retrieval while acquiring semantic generalization ability. Its performance across multiple retrieval benchmarks typically surpasses traditional BM25 and also outperforms purely dense retrieval in certain exact-matching scenarios, making it a prominent "third route" in both academia and industry in recent years.

In summary, SPLADE replaces traditional statistical formulas with neural networks to learn sparse vector weights, combining the efficient inverted-index capabilities of sparse retrieval with semantic generalization (term expansion). It represents a noteworthy technical path between sparse and dense retrieval, particularly suited for scenarios with high demands on both exact matching and semantic generalization, especially where reusing existing sparse retrieval infrastructure (inverted indices) is desired.

</details>

**5. What is Hybrid Search? How Are Results From Sparse and Dense Retrieval Fused and Ranked (e.g., via RRF)?**

<details>
<summary>💡 Show Answer</summary>

Hybrid search refers to a retrieval architecture that simultaneously retrieves candidates using sparse retrieval (e.g., BM25) and dense retrieval (embedding models), and then fuses the two streams of results into a final ranking. It is a very common practice in modern production retrieval systems:

- **Why Hybrid is Needed**: As mentioned earlier, sparse and dense retrieval exhibit complementary failure modes—sparse retrieval excels at exact matching (keywords, terminology, IDs, etc.), while dense retrieval excels at semantic generalization (synonym paraphrasing, cross-lingual, vague queries). Using them in combination ensures high retrieval quality across both dimensions;
- **Fusion Method 1: Score-Weighted Fusion**: The relevance scores of the two retrieval streams are weighted and summed to yield a final fused score (normalization is required beforehand, as BM25 scores and vector similarity scores have completely different scales/distributions). Weights can be fixed empirical values, tuned via a validation set, or learned using a lightweight model;
- **Fusion Method 2: Reciprocal Rank Fusion (RRF)**: Instead of using raw scores directly (avoiding scale mismatch issues across different retrieval methods), RRF relies solely on the **rank** of each document within its respective retrieval result list. The fused score is calculated as $\sum_i \frac{1}{k + rank_i(d)}$ (where $k$ is a smoothing constant, typically set to 60), assigning higher fusion scores to documents ranked near the top in each retrieval stream, and finally re-sorting by the fused score. RRF's advantages are that it requires no score normalization, is simple to implement, robust, and widely adopted in practice;
- **Fusion Method 3: Using a Reranking Model**: The deduplicated candidate set from merging both retrieval streams is passed to an independent reranking model (such as a Cross-Encoder) for unified scoring and ranking. This does not rely on explicit score fusion rules and generally yields better results, albeit at higher computational cost. It is typically used for the final reranking stage after candidates have been significantly narrowed down by the initial two retrieval streams.

In summary, hybrid search runs sparse and dense retrieval in parallel and merges their results, achieving both keyword exact matching and semantic generalization advantages. RRF is one of the most commonly used fusion methods due to its simplicity and lack of score scale dependency. In scenarios with higher retrieval quality requirements, a reranking model layer can be stacked after fusion to further boost precision.

</details>

**6. In a Complete Retrieval System, Why is a Multi-Stage Architecture ("Retrieval – Pre-Ranking – Reranking") Typically Adopted Instead of Using a Single Model for All Tasks?**

<details>
<summary>💡 Show Answer</summary>

Modern large-scale retrieval systems (especially in RAG and search engine scenarios) almost universally adopt a staged funnel-like architecture rather than solving the problem end-to-end with a single model, primarily due to trade-offs between performance and efficiency:

- **First Stage: Retrieval (Recall)**: From a massive pool of candidates (potentially millions to billions of documents), the computationally most efficient methods (typically vector retrieval using Dual Encoders, or sparse/hybrid retrieval) are used to quickly filter down to a significantly smaller candidate set (e.g., hundreds to thousands). This stage pursues **high recall** (minimizing false negatives for genuinely relevant documents) with relatively lower precision requirements, as subsequent stages will further refine the ranking;
- **Second Stage: Pre-ranking (Optional)**: For scenarios where the candidate set remains relatively large (e.g., e-commerce search with thousands of candidates), an intermediate pre-ranking stage is inserted. A model lighter than the reranker but finer than the retrieval stage (e.g., lightweight dual-tower models or simplified Cross-Encoders) is used to shrink the candidate pool further;
- **Third Stage: Reranking (Fine Ranking)**: For candidate pools already narrowed down significantly (e.g., tens to a hundred), the model with the highest computational cost and accuracy (typically a Cross-Encoder or a complex ranking model combining multiple features) performs final fine-grained sorting, directly determining the final output order presented to the user;
- **Why a Single Model Cannot Be Used**: If high-precision models like Cross-Encoders were applied directly to retrieve from the full candidate corpus, the computational cost would be prohibitive (requiring every candidate to be passed through the model together with the query, preventing pre-indexing and efficient approximate nearest neighbor algorithms). Conversely, relying solely on efficient Dual Encoders through to the final step results in degraded ranking quality, as Dual Encoders generally lack the fine-grained ranking precision of Cross-Encoders. A multi-stage architecture essentially manages trade-offs between efficiency and accuracy in tiers, allowing computational cost to scale up incrementally as candidate pool sizes shrink.

In summary, the multi-stage retrieval architecture employs a funnel design of "rapidly reducing the candidate pool using the most efficient methods, followed by layer-by-layer filtering using progressively finer but more expensive models." This maintains high final ranking accuracy while keeping overall retrieval latency and compute costs within acceptable bounds, making it the standard engineering practice for large-scale retrieval scenarios.

</details>

**7. When Training a Cross-Encoder as a Reranker Model, What Are the Characteristics of Pointwise, Pairwise, and Listwise Training Objectives?**

<details>
<summary>💡 Show Answer</summary>

Cross-Encoders are often used as reranking models in retrieval pipelines. The design of training objectives (i.e., how loss functions are defined) follows three mainstream paradigms inherited from the traditional Learning to Rank (LTR) field:

- **Pointwise**: Formulates the ranking problem as independent regression or classification tasks, assigning a relevance score to each (query, document) pair individually (e.g., binary classification: relevant/irrelevant, or multi-grade ratings: 0–4 relevance levels). During training, loss is calculated independently for each sample (e.g., cross-entropy or mean squared error), ignoring relative relationships between different candidates under the same query. Its advantages are simple implementation and stable training. Its disadvantage is that it does not explicitly model the objective of "ranking" itself—the model learns absolute relevance judgment, whereas actual business applications care about the **relative order** among candidates, which are not completely equivalent;
- **Pairwise**: Takes two candidate documents under the same query at a time (one with higher relevance, one with lower relevance) and trains the model to correctly determine "which candidate should rank higher." The loss function typically requires the score of the higher-relevance candidate to significantly exceed that of the lower-relevance candidate (similar to the Triplet Loss concept mentioned earlier, but comparing two candidates under the same query rather than positive and negative samples across instances). Compared to pointwise, pairwise models ranking relationships more directly and is one of the most common training choices among the three;
- **Listwise**: Directly optimizes the ranking quality of the entire candidate list. The loss function considers the relative ordering among **all** candidate documents for the same query (rather than pairwise comparisons), typically optimizing objectives closer to ranking evaluation metrics like nDCG (e.g., ListNet models the permutation probability distribution of the entire list via softmax, and LambdaRank directly uses nDCG variance as gradient weights). Theoretically, it has the highest alignment with final ranking evaluation metrics (nDCG, etc.), but its implementation and training are relatively more complex, requiring multiple candidates to be processed simultaneously for each query.

In summary, the three training objectives progress incrementally in "modeling complexity" and "alignment with final ranking goals": Pointwise is the simplest but least aligned with ranking goals; Pairwise offers a balanced and widely adopted choice between performance and implementation complexity; Listwise is theoretically optimal but requires more complex engineering and hyperparameter tuning. In practice, many reranking models (including open-source Cross-Encoder rerankers) adopt pairwise loss or hybrid training approaches combining multiple objectives.

</details>

**8. Why Is the Computational Cost of Cross-Encoders Extremely High, Preventing Their Direct Use in the Large-Scale Candidate Retrieval Stage?**

<details>
<summary>💡 Show Answer</summary>

Understanding the computational complexity bottleneck of Cross-Encoders is key to understanding why retrieval systems require multi-stage architectures and why Dual Encoders and Cross-Encoders must collaborate:

- **Fundamental Difference in Computation**: Dual Encoders encode queries and documents independently. Document vectors can be **pre-computed offline and indexed**. In the online phase, the query needs to be encoded only once, followed by searching the pre-built index using efficient Approximate Nearest Neighbor (ANN) search algorithms, where retrieval computational complexity is largely independent of the total candidate count (thanks to logarithmic or sub-linear complexity of ANN indices). In contrast, a Cross-Encoder requires concatenating the query and **each** candidate document into a single input sequence for joint encoding. This means document representations **cannot** be computed or cached independently of a specific query—for every new query, a complete forward pass must be re-run for every candidate document in the pool;
- **Complexity Comparison**: Assuming the total number of candidate documents is $N$, the online computational complexity of Dual Encoder retrieval is roughly $O(\log N)$ (with ANN indices). Conversely, a Cross-Encoder requires a full Transformer forward pass for every candidate, yielding a complexity of $O(N)$, where each individual forward pass itself (especially with longer concatenated sequence lengths) is much more expensive than a Dual Encoder encoding a short query;
- **Practical Impact**: If the candidate corpus reaches millions or more, scoring all candidates with a Cross-Encoder would result in completely unacceptable single-query latency and compute cost (potentially requiring minutes or longer, whereas production retrieval scenarios typically demand responses within milliseconds to hundreds of milliseconds). This is precisely why Cross-Encoders can only be used in the reranking stage after the candidate set has been drastically reduced (e.g., to tens or hundreds of candidates) and cannot perform full-corpus candidate retrieval.

In summary, because Cross-Encoders must jointly encode the query with each candidate document and cannot pre-compute candidate representations offline to build efficient indices like Dual Encoders, their computational complexity scales linearly with the candidate count. Consequently, they are suitable only for fine-grained ranking after candidates have been drastically compressed by prior stages, and cannot be directly applied to the retrieval stage facing massive candidate pools. This represents one of the core engineering reasons behind multi-stage retrieval architectures.

</details>

**9. Can Cross-Encoders and Dual Encoders Be Jointly Trained (e.g., Sharing Lower-Layer Parameters or Using Cross-Encoders to Guide Dual Encoder Training) to Improve Dual Encoder Performance?**

<details>
<summary>💡 Show Answer</summary>

Beyond the knowledge distillation approach mentioned earlier (using a Cross-Encoder to distill similarity/ranking relationships into a Dual Encoder), there are other potential joint training and co-optimization strategies between Cross-Encoders and Dual Encoders:

- **Sharing Lower-Layer Encoder Parameters**: Share the lower layers of the same pre-trained backbone model between Dual Encoder and Cross-Encoder (e.g., sharing the initial several Transformer layers, while branching into separate Dual Encoder and Cross-Encoder top layers). Under a multi-task framework, joint training optimizes losses for both branches. This allows basic language understanding capacity to be shared across both architectures, enabling the Dual Encoder to indirectly benefit from the finer semantic understanding signals of Cross-Encoder training;
- **Using Cross-Encoder Scores as an Auxiliary Tool for Hard Negative Mining**: As one of the hard negative mining techniques discussed earlier, a trained (or jointly training) Cross-Encoder can be used to mine more precise hard negatives for the Dual Encoder. This is an indirect "collaboration" where the Cross-Encoder itself does not directly participate in the Dual Encoder's gradient updates, but provides higher-quality training data for it;
- **Iterative Co-training (Self-Distillation Loop)**: First train a Cross-Encoder (typically on high-quality labeled data) and use it to distill/guide Dual Encoder training. The trained Dual Encoder is then used to mine a new round of hard negatives to re-train/fine-tune the Cross-Encoder. Through repeated iterations, both models mutually enhance each other. This "generator-discriminator" style iterative training concept is reflected in several retrieval-related works;
- **Cost and Engineering Complexity**: Compared to training two models independently, joint training introduces higher engineering complexity (requiring design of reasonable multi-task loss weights, and gradients from shared layers may interfere with each other, leading to training instability). In practice, deciding whether to adopt joint training requires weighing the extra performance gains against increased engineering complexity. In many scenarios, a simpler pipeline—"first train a strong Cross-Encoder, then perform unidirectional distillation to the Dual Encoder"—already achieves strong performance without needing more complex joint/iterative training.

In summary, Cross-Encoders and Dual Encoders can be connected and mutually enhanced through parameter sharing, distillation, hard negative mining, and iterative co-training. The core intuition is to transfer the "fine-grained discrimination ability" of the computationally expensive but highly accurate Cross-Encoder to the computationally efficient but less accurate Dual Encoder. The choice of joint training strategy depends on balancing engineering complexity against practical performance gains.

</details>

**10. When Converting Transformer Outputs into a Single Sentence Vector, What Are the Common Pooling Strategies? What Are Their Respective Advantages and Disadvantages?**

<details>
<summary>💡 Show Answer</summary>

A Transformer encoder outputs a vector for each token in the input. To obtain a fixed-length embedding representing the entire sentence, token-level vectors must be aggregated into a sentence vector via a pooling strategy. Common strategies include:

**1. [CLS] / Special Token Pooling**

- Directly takes the output vector corresponding to the special token at the beginning of the sequence (e.g., [CLS] in BERT) as the sentence representation;
- Advantages: Simple to implement. If the model has undergone specialized training for this token (e.g., Next Sentence Prediction or contrastive learning objectives directly supervising this position), the token can acquire strong sentence-level semantic aggregation capability;
- Disadvantages: If the model has not been specifically trained for this token (such as raw BERT pre-trained solely with Masked Language Modeling), the [CLS] vector is often of poor quality, yielding subpar results when directly used for similarity calculations.

**2. Mean Pooling**

- Takes the average of the output vectors across all (non-padding) tokens in the sequence as the sentence representation;
- Advantages: Does not rely on any single specific position and fully utilizes information from every token in the sequence. Empirically validated in work like Sentence-BERT to consistently outperform raw [CLS], making it one of the most widely used default strategies today;
- Disadvantages: Treats all tokens equally, which may dilute key information with unimportant words (such as function words or stop words). In long sentences, the few truly critical words may be averaged out.

**3. Last-token Pooling**

- Takes the output vector of the last token in the sequence (for autoregressive/causal attention models, such as embedding models based on GPT architectures) as the sentence representation, because under causal attention mechanisms, the last token is the only position that can attend to all preceding context;
- Advantages: Naturally aligns with causal language model architectures (mean pooling with bidirectional attention does not fit pure causal models well). Recent LLM-based embedding solutions (e.g., converting LLMs into embedding models) frequently adopt this strategy;
- Disadvantages: Relies solely on a single position, making it highly sensitive to the representation quality of that specific position. Additional contrastive learning fine-tuning is typically required for this position to learn high-quality sentence-level semantics (during pre-training, this position primarily predicts the next word, which is not inherently suited for semantic aggregation).

**4. Max Pooling**

- Takes the maximum value across all tokens for each dimension. In practice, its performance is generally less stable than mean pooling and it is rarely used in sentence embedding tasks today, being more common in early CNN/word-vector eras.

In summary, the choice of pooling strategy should be aligned with model architecture (bidirectional vs. causal) and training setup (whether specialized supervision signals exist for specific positions): For bidirectional encoders with contrastive fine-tuning, mean pooling is typically a robust default choice; LLM-based embedding schemes utilizing causal architectures mostly adopt last-token pooling; [CLS] pooling depends heavily on whether that position was explicitly optimized during pre-training/fine-tuning and should not be used directly from un-finetuned [CLS] vectors.

</details>

**11. Why Do Many Embedding Models Append a Linear Projection Layer (Projection Head) After the Transformer Encoder Output (Post-Pooling)? What Is the Function of This Layer?**

<details>
<summary>💡 Show Answer</summary>

Examining the architecture design of many embedding models (including SimCLR in computer vision and several text embedding models), one observes that after pooling the encoder backbone output into a sentence vector, a small projection head (typically one or two linear transformations, optionally with non-linear activations) is attached. This design serves specific considerations:

- **Decoupling "General Language Representation" from "Contrastive Learning Specific Representation"**: Deep parameters in the encoder backbone (such as BERT) contain rich general language understanding capabilities that may not be directly ideal for similarity calculation tasks. The projection head learns a mapping on top of the encoder backbone output specifically optimized for the contrastive learning objective, moderately decoupling the vector space used for similarity computation from the original representation space inside the encoder;
- **Insights from Computer Vision (SimCLR Findings)**: In the seminal vision contrastive learning work SimCLR, experiments revealed that computing contrastive loss directly on raw encoder outputs yielded lower performance than the "encoder backbone output $\rightarrow$ projection head $\rightarrow$ contrastive loss" architecture. Crucially, when applying vectors to downstream tasks, using the **encoder backbone output directly** (rather than post-projection head output) performed better. This indicates that the projection head absorbs transformations during training that are "necessary for optimizing contrastive loss but potentially detrimental to general representations," thereby shielding the quality of backbone output representations from excessive distortion by contrastive objectives;
- **Differences in Text Embedding Practices**: Unlike computer vision, many text embedding models (especially those following the Sentence-BERT line of work) directly use post-pooling vectors (potentially after projection) as final sentence representations. Whether to use a projection head and whether to take the post-projection output as final vectors varies across studies, requiring verification against specific model technical reports;
- **Additional Practical Value**: Projection heads are also commonly used to compress high-dimensional raw encoder outputs (e.g., 1024 dimensions or higher) down to a desired final embedding dimension, fulfilling dimensionality reduction and adapting to downstream storage/retrieval dimensionality requirements.

In summary, the core value of a projection head lies in moderately decoupling "transformations tailored for contrastive loss" from "general language representations learned by the encoder backbone," preventing contrastive learning objectives from overly distorting backbone representation quality. Concurrently, it serves practical dimensionality adaptation functions. Whether and how to use it depends on specific model architecture and downstream usage scenarios.

</details>

**12. In a dual-encoder architecture, should the query encoder and document encoder share the same set of parameters? Under what circumstances should an asymmetric (untied-parameter) dual-encoder be used?**

<details>
<summary>💡 Show Answer</summary>

Dual-encoder architectures offer two design choices: "symmetric" (where the query and document share the same set of encoder parameters) and "asymmetric" (where the query and document use independent encoders with separate parameters). The choice between them depends on the specific application scenario:

- **Symmetric Dual-Encoder (Siamese/Tied Weights)**: The query and document are encoded independently through the **same** encoder (sharing all parameters). The underlying assumption of this design is that the **text distribution, length, and style** of the query and document are relatively close (for instance, the two sentences compared in semantic text similarity tasks often have similar lengths and styles). Parameter sharing leads to fewer total parameters and easier training (as gradient updates from both sides mutually reinforce the optimization of the same parameter set), making it a common choice for many Semantic Textual Similarity (STS) tasks and paraphrase retrieval scenarios.
- **Asymmetric Dual-Encoder (Untied Weights)**: The query and document use two **independent** encoders (with untied parameters, potentially even using models of different scales or architectures). This design is suitable for scenarios where the query and document exhibit **pronounced differences in textual characteristics**. For example, in a typical retrieval scenario, the query is usually very short (ranging from a few words to a single sentence), while the document might be a long paragraph or an entire article. Significant differences exist in their language distributions and representation requirements. Encoding both with the same set of parameters makes it difficult to simultaneously handle the two distinct subtasks: "understanding short, vague query intents" and "compressing rich information from long documents."
- **Specific Implementations of Asymmetric Designs**: This can range from two completely independently trained models to a "partially shared" design where lower layers (such as word embeddings and the first few Transformer layers) are shared to capture general language features, while the top layers split into two independent branches—striking a compromise between parameter efficiency and adapting to input asymmetry.
- **Practical Selection Criteria**: If the texts on both sides are highly similar in length, style, and source (e.g., paraphrase detection, sentence similarity), a symmetric design is preferred (offering lower parameter counts and more stable training). If there is marked asymmetry between the two sides (e.g., typical query-document retrieval scenarios), an asymmetric design should be considered; however, the additional parameter footprint and training costs must be weighed. In practice, many real-world systems still opt for a parameter-shared symmetric design even in asymmetric scenarios for engineering simplicity, and performance differences should be verified empirically.

In summary, whether to share parameters between the query and document encoders fundamentally depends on the similarity of the text on both sides in terms of length, style, and semantic granularity. The more similar they are, the more suitable a parameter-shared (symmetric) design becomes; the larger the gap, the more necessary it is to consider an asymmetric design to better adapt to their respective representation needs. In practice, empirical validation on specific tasks is required to determine whether the performance gain of an asymmetric design justifies the extra parameter and training overhead.

</details>

**13. What is the relationship between the dual-encoder architecture and the classic Siamese Network?**

<details>
<summary>💡 Show Answer</summary>

The Siamese Network is an earlier, more general deep learning architectural concept. The dual-encoder can be viewed as a concrete application of the Siamese Network paradigm within the field of text embedding:

- **Core Idea of Siamese Networks**: Use the **same** encoder network (with shared weights) to separately process two (or more) inputs, mapping them into the same vector space, and then compare these vectors via some distance metric (such as Euclidean distance or cosine similarity). It was originally proposed for metric learning tasks like face verification (determining whether two face photos belong to the same person).
- **Relationship with Dual-Encoders**: The previously mentioned "symmetric dual-encoder" (where the query and document share the same encoder parameters) is essentially a direct application of the Siamese Network concept to the NLP domain—using the same Transformer encoder to independently encode two text segments before computing their similarity. The seminal paper for Sentence-BERT explicitly used the word "Siamese" in its title (*Sentence-BERT: Sentence Embeddings using Siamese BERT-Networks*), drawing directly upon this classic architectural framework.
- **Difference between "Asymmetric Dual-Encoder" and Classic Siamese Networks**: Strictly speaking, a Siamese Network requires weight sharing. The aforementioned asymmetric dual-encoder (untied parameters) aligns more closely with another related yet distinct concept—the **Pseudo-Siamese Network**, where two encoders with similar structures but independent parameters process different types of inputs. Although this distinction is sometimes highlighted in academic literature, in industrial practice and informal technical communication, the term "Bi-Encoder" is commonly used as an umbrella term for any architecture involving "two-way independent encoding followed by similarity comparison," regardless of whether weights are shared.
- **Historical Context**: Understanding this connection helps reveal that the dual-encoder architecture is not a brand-new invention of the text embedding era, but rather a natural continuation and transfer of classic metric learning concepts into the Transformer era.

In summary, the dual-encoder architecture conceptually inherits directly from the earlier and more general Siamese Network metric learning framework. A "symmetric dual-encoder" corresponds to a strict weight-sharing Siamese Network, whereas an "asymmetric dual-encoder" is closer to a weight-independent Pseudo-Siamese Network. Grasping this historical lineage aids in understanding text embedding model design within the broader context of deep metric learning research.

</details>

**14. In vector retrieval, how should one choose among cosine similarity, dot product (inner product), and Euclidean distance as similarity/distance metrics?**

<details>
<summary>💡 Show Answer</summary>

To measure "how similar" two embedding vectors are, common metrics include cosine similarity, dot product, and Euclidean distance. These three are interconnected yet distinct, and the following factors should be considered when choosing between them:

- **Cosine Similarity**: Measures the similarity in **direction** between two vectors, independent of their magnitudes, with values in the range $[-1, 1]$. It is the most widely used metric in text embeddings because semantic similarity is generally believed to depend solely on vector direction (representing semantic content) without being distorted by vector magnitudes (which may be influenced by extraneous factors during training).
- **Dot Product (Inner Product)**: As previously noted, if the vectors are L2-normalized, the dot product is numerically identical to cosine similarity. However, if vectors are unnormalized, the dot product is jointly affected by both direction and magnitude. In this case, dot product similarity implicitly assumes that "magnitude conveys a form of importance/confidence" (e.g., in certain recommendation systems, unnormalized magnitudes are deliberately retained to encode extra confidence or popularity signals).
- **Euclidean Distance**: Measures the **absolute distance** between two vectors in space, affected by both direction and magnitude. Smaller values indicate higher similarity (opposite to cosine similarity where larger values mean higher similarity; care must be taken not to invert this in implementation). If vectors are normalized to unit length, a simple monotonic transformation relates Euclidean distance and cosine similarity ($d^2 = 2 - 2\cdot cos_sim$), making them strictly equivalent in ranking order.
- **Practical Selection**: Under the prerequisite that vectors are L2-normalized (the standard practice in modern embedding models), all three metrics are practically equivalent in terms of ranking (dot product and cosine similarity produce identical numerical values, while Euclidean distance corresponds monotonically). The choice among them is largely driven by engineering efficiency: dot product is the simplest to compute and benefits from specialized low-level optimizations in many vector databases and ANN indexing libraries (such as FAISS's Inner Product index). Therefore, in normalized vector scenarios, dot product is usually preferred to directly leverage highly efficient index implementations.

In summary, under the standard practice of normalized vectors, cosine similarity, dot product, and Euclidean distance yield identical ranking results for retrieval. Practical selection is mostly determined by computational efficiency and underlying engineering infrastructure (native support in vector databases/ANN indexing libraries). Dot product is typically the default choice for normalized vectors due to its simple computation and mature indexing optimizations.

</details>

**15. How should the choice of similarity metric be aligned with the underlying Approximate Nearest Neighbor (ANN) index type?**

<details>
<summary>💡 Show Answer</summary>

As mentioned earlier, matching the choice of similarity metric with the ANN (Approximate Nearest Neighbor) index type used by the vector retrieval system is an easily overlooked engineering detail that directly affects retrieval accuracy and efficiency:

- **Native Metric Support in Index Libraries**: Different ANN indexing algorithms/libraries (such as FAISS, HNSWlib, Milvus, etc.) usually implement specialized low-level optimizations for specific similarity metrics (inner product, L2 Euclidean distance, cosine similarity). Choosing a metric aligned with the native support of the index ensures optimal retrieval performance. Mismatched metrics (e.g., an index optimized for L2 distance when the business logic requires ranking by cosine similarity) may demand additional conversion steps (such as the aforementioned equivalence between L2 distance and cosine similarity after normalization), introducing engineering complexity and potential implementation risks.
- **Coupling Between Vector Normalization and Index Selection**: If output vectors from the embedding model are already L2-normalized (standard practice in modern embedding models), one can directly select index types supporting inner (dot) product computation (since normalized dot product equals cosine similarity) without extra processing. If vectors are unnormalized for specific reasons (such as retaining magnitude to convey confidence), the choice of index and metric must align with the specific business logic. Furthermore, the optimization objective during training (e.g., whether loss is computed via cosine similarity or dot product) must match the metric used during inference retrieval; otherwise, "metric mismatch" between training and inference can severely impair retrieval performance.
- **Common Pitfalls**: A frequent practical engineering error is calculating cosine similarity loss on normalized vectors during training, but forgetting to normalize vectors when deploying to a vector database (or misconfiguring the index metric). This leads to retrieval results deviating from expectations. Such issues rarely throw explicit errors; instead, they silently degrade retrieval performance, highlighting the need to rigorously check metric consistency between training and inference before deployment.

In summary, selecting a similarity metric is not an isolated decision. It must be aligned with the native support of the underlying ANN index library, whether vectors are normalized, and the metric used during training. A mismatch at any stage can lead to subtle degradation in retrieval effectiveness, making it a critical detail to audit during vector retrieval system deployment.

</details>

**16. What are the principles behind common Approximate Nearest Neighbor (ANN) index algorithms (e.g., HNSW, IVF, LSH), and how should they be chosen in practice?**

<details>
<summary>💡 Show Answer</summary>

When a vector database reaches a large scale, exact similarity computation between a query and all candidate vectors (brute-force search) becomes prohibitively expensive. Approximate Nearest Neighbor (ANN) algorithms are used to trade a small loss in accuracy for massive speedups in retrieval. Common algorithms include:

- **HNSW (Hierarchical Navigable Small World)**: Constructs a multi-layer graph structure where each layer represents a nearest-neighbor graph connecting vectors. Upper layers contain sparse nodes for rapid global localization, while lower layers are dense for fine-grained searching. During retrieval, navigation starts from the top layer and moves downward layer by layer to approximate the target region. Advantages include high retrieval speed and strong recall performance (exhibiting an excellent trade-off between precision and speed), making it one of the most widely used ANN algorithms in industry today. Disadvantages are longer index build times and relatively high memory overhead (due to storing graph structures).
- **IVF (Inverted File Index)**: First partitions the vector space into a set of clusters ("buckets") using a clustering algorithm (such as K-means), assigning each vector to the bucket corresponding to its nearest centroid. During retrieval, the search quickly narrows down to a few candidate buckets most likely to contain the query vector (rather than searching the entire database), performing exact similarity calculations only within these buckets. Advantages include high memory efficiency and relatively fast index construction. Disadvantages are that accuracy heavily depends on clustering quality and requires tuning hyper-parameters like the number of centroids. It is frequently combined with quantization techniques (such as Product Quantization, PQ) as IVF-PQ to further compress memory footprint.
- **LSH (Locality-Sensitive Hashing)**: Utilizes specially designed hash functions to map similar vectors to the same hash bucket with high probability. During retrieval, candidate vectors are compared only within the same (or adjacent) hash buckets. Advantages include solid theoretical guarantees and suitability for ultra-high-dimensional sparse vector scenarios. Disadvantages are that its practical efficiency and recall in dense, medium-dimensional vector scenarios fall short of modern graph-based methods like HNSW, leading to relatively sparse usage in text embedding retrieval today.
- **Practical Selection Recommendations**: For moderate vector sizes (millions to tens of millions) with high precision requirements, HNSW is generally the best all-around default choice. For massive vector databases (billion-scale and above) operating under constrained memory where slightly lower recall is acceptable in exchange for significantly reduced memory footprint, IVF (especially IVF-PQ combined with quantization) offers distinct advantages. LSH is rarely used in mainstream text embedding retrieval and is largely confined to specific sparse or ultra-high-dimensional vector domains.

In summary, HNSW delivers the best overall balance between retrieval speed and accuracy via hierarchical graph structures, serving as the top choice for most medium-scale scenarios. IVF (combined with quantization) excels in ultra-large-scale, memory-constrained settings. LSH retains theoretical and applied value in niche scenarios. Practical selection requires balancing vector database scale, memory budget, and accuracy requirements.

</details>

**17. What accuracy loss is introduced by Approximate Nearest Neighbor (ANN) retrieval compared to exact brute-force search, and how can one balance effectiveness and efficiency?**

<details>
<summary>💡 Show Answer</summary>

The fundamental reason ANN algorithms achieve massive speed improvements is that they trade a marginal amount of retrieval accuracy (potentially missing a few of the true top-k results) for computational efficiency. Understanding and managing this trade-off is a vital aspect of vector retrieval system design:

- **Sources of Accuracy Loss**: ANN algorithms (whether graph-based like HNSW or cluster-based like IVF) essentially use heuristic strategies to rapidly restrict the search scope rather than exhaustively comparing all candidates. Consequently, some of the true most similar vectors might be missed simply because they fell outside the heuristic search path. This manifests as a **Recall** (the overlap between ANN retrieval results and the ground-truth top-$k$ results from brute-force search) of less than 100%.
- **Key Parameters Influencing Recall**: Almost all ANN algorithms expose tunable parameters governing the "breadth of search scope" to balance accuracy and speed. For example, `ef_search` in HNSW (the number of candidates explored during search; higher values increase recall but slow down speed) and `nprobe` in IVF (the number of cluster buckets searched during retrieval; higher values improve accuracy at the expense of speed). These parameters are typically tuned experimentally to strike an optimal balance between target recall (e.g., $>95%$ overlap with brute-force search) and latency budgets.
- **Evaluation and Monitoring**: Prior to deploying an ANN retrieval system to production, one should evaluate the recall gap (Recall@$k$ consistency) between ANN retrieval and brute-force exact search on a validation set to ensure accuracy loss stays within acceptable bounds. Post-deployment, periodic sampling and monitoring should be conducted, as changes in dataset size or distribution can degrade recall under static ANN hyper-parameters, requiring dynamic adjustments.
- **Mitigating Accuracy Loss via Two-Stage Pipelines**: Similar to vector quantization strategies, a two-stage architecture can be adopted: "Fast ANN Retrieval (prioritizing high recall over exact ranking) + Exact Re-ranking (performing exact similarity calculations or applying a stronger reranking model on a small candidate pool)." Even if the ANN phase introduces approximation error, as long as relevant documents have a high probability of appearing within the retrieved candidate pool, the subsequent reranking stage can correct ordering errors introduced by ANN.

In summary, ANN retrieval trades speed for accuracy by using heuristic search strategies, inevitably introducing some recall loss. In practice, algorithm search breadth parameters must be tuned to balance precision and performance. Combining an "ANN Retrieval + Exact Reranking" two-stage pipeline effectively keeps approximation errors within acceptable limits for downstream business outcomes.

</details>

**18. What are the pros and cons of Multiple Negatives Ranking Loss (MNR), Cosine Similarity Loss, and Softmax Loss when training embedding models?**

<details>
<summary>💡 Show Answer</summary>

**1. Multiple Negatives Ranking Loss (MNR, also known as InfoNCE)**

*Mechanism*: For each positive pair $(query, positive)$, the positives of other samples within the batch (or explicitly provided extra negative samples) serve as negative instances. The model computes the similarity between the query and all candidates ($1$ positive + $N-1$ negatives) and uses a Softmax + Cross-Entropy objective to maximize the fraction of positive similarity relative to all candidates.

*Pros*:
- Implicitly leverages a large number of in-batch negatives. The number of negatives grows with the batch size, yielding high training efficiency;
- Directly optimizes the ranking objective of "distinguishing positives from negatives," aligning closely with downstream retrieval/ranking tasks and typically yielding the best performance on semantic search;
- Does not require explicit numerical similarity scores—training requires only positive pairs.

*Cons*:
- Training efficacy depends on batch size (larger batch sizes supply more negatives, but incur higher GPU memory overhead);
- Vulnerable to false negatives (where an in-batch "negative" is actually semantically relevant to the query), introducing optimization noise;
- Requires carefully designed sampling strategies (such as hard negative mining) to further boost performance.

**2. Cosine Similarity Loss**

*Mechanism*: Given pairs of samples annotated with continuous similarity labels (e.g., continuous values between $0$ and $1$ representing semantic similarity), the model directly regresses the predicted cosine similarity against the label (e.g., using MSE).

*Pros*:
- Can utilize fine-grained similarity annotations (beyond binary "similar/dissimilar" labels to continuous similarity scores, such as STS datasets);
- Simple and intuitive to implement, directly aligning the loss function with the final evaluation metric (cosine similarity).

*Cons*:
- Requires high-quality labeled data with continuous similarity scores, which is expensive to acquire;
- Cosine similarity ranges in $[-1, 1]$, but practical semantic similarity labels often cluster within a narrow sub-range. The numerical distribution of regression targets may lack sufficient variance, providing weaker optimization signals than contrastive ranking objectives;
- Fails to directly exploit relative ranking information among negative pairs, making it less targeted for retrieval tasks compared to MNR.

**3. Softmax Loss (Classification Loss, e.g., Softmax Cross-Entropy over Classes)**

*Mechanism*: Treats the relationship between sample pairs (e.g., sentence pairs) as a classification problem (e.g., 3-way classification of "entailment/neutral/contradiction" in NLI tasks). Model representations pass through a classification head to compute class probabilities via Softmax, evaluated against ground-truth class labels using Cross-Entropy loss.

*Pros*:
- Can leverage off-the-shelf classification datasets (e.g., NLI datasets) with clear supervisory signals;
- Performs well on tasks with discrete categorical boundaries (rather than continuous similarities), capturing nuanced semantic relation types (e.g., distinguishing "entailment" from "contradiction", rather than mere "similarity" vs. "dissimilarity").

*Cons*:
- The learned representation space is optimized for classification, which may not directly align with the "vector distance = semantic similarity" paradigm, leading to sub-optimal transfer performance on retrieval tasks compared to ranking-loss objectives;
- Relies on datasets with explicit categorical labels, incurring high construction costs;
- Fixed number of categories makes it difficult to express fine-grained similarity gradients.

**Summary**:
- **Data Format Compatibility**: MNR is best suited for positive pair data (most readily accessible and common); Cosine Similarity Loss fits continuous similarity-annotated data; Softmax Loss fits discrete category-labeled data (such as NLI).
- **Empirical Efficacy**: For training embedding models geared toward retrieval and semantic search, **MNR (and its InfoNCE variants) represents the prevailing and most effective choice today**, often serving as the primary loss in pre-training. Cosine Similarity Loss and Softmax Loss (especially on NLI data) are frequently applied in early stages or as auxiliary/fine-tuning losses to help models capture fine-grained semantic relations.

</details>

**19. How are Pairwise and Listwise ranking losses applied in embedding model training, and what are their connections and differences compared to standard InfoNCE loss?**

<details>
<summary>💡 Show Answer</summary>

Earlier, we discussed the differences between Triplet Loss and InfoNCE in contrastive learning. Here, we examine the relationships between pairwise and listwise losses and common embedding loss functions from the classical perspective of Learning to Rank (LTR):

- **InfoNCE/MNR Inherently Possesses Listwise Characteristics**: A close inspection of the InfoNCE formulation reveals that it considers one positive sample alongside a set of negative samples simultaneously, normalizing over the entire candidate set ($1$ positive + $n$ negatives) via Softmax. This effectively models ranking across a "list" (the candidate set) rather than simple pairwise comparisons. Hence, InfoNCE/MNR loss can be viewed as a (simplified) listwise ranking loss, where the list contains only a single "ground-truth answer" (binary relevance: positive vs. negative) rather than multi-level relevance grades common in traditional listwise ranking losses.
- **Triplet Loss is a Classic Pairwise Formulation**: As previously noted, Triplet Loss compares the relative distance of a positive sample and a negative sample with respect to an anchor at each step, representing a classic pairwise comparison paradigm.
- **Strict Listwise Ranking Loss**: If training data provides fine-grained multi-level relevance grades (e.g., ratings from 0 to 4) for multiple candidates under the same query rather than simple binary labels, strict listwise ranking losses (such as ListNet or LambdaRank) can be applied. These losses directly optimize the overall list ranking quality, achieving tighter alignment with evaluation metrics like nDCG. Such objectives are more prevalent when training cross-encoder rerankers (which typically handle fine-grained relevance labels). In contrast, during the pre-training/fine-tuning stages of dual-encoders—where datasets are massive and annotations are mostly binary—the simplified listwise InfoNCE/MNR loss is predominantly used.
- **Selection Criteria**: If training data contains only binary relevance labels (the norm in large-scale weakly supervised scenarios), InfoNCE/MNR is the natural and efficient choice. If fine-grained, multi-level relevance annotations are available (typically smaller-scale, high-cost datasets), strict listwise ranking losses should be considered to fully exploit these fine-grained signals.

In summary, InfoNCE/MNR can be understood as a simplified listwise ranking loss (with binary labels), while Triplet Loss exemplifies the classic pairwise paradigm. Strict listwise losses (such as ListNet and LambdaRank) that incorporate multi-level relevance grades are primarily reserved for settings with fine-grained annotations (e.g., reranker training). These three approaches differ in whether they leverage list-wide context and the granularity of annotations; selection should match the annotation format of the training data.

</details>

**20. Can classic Learning to Rank (LTR) methods like ListNet and LambdaRank be directly applied to train embedding models?**

<details>
<summary>💡 Show Answer</summary>

Methods like ListNet and LambdaRank are classic learning-to-rank algorithms with a long history in search and retrieval. They were originally designed for **feature-based** ranking models (where inputs consist of hand-crafted relevance features and outputs are ranking scores). Adapting them to train deep learning-based embedding models requires addressing several alignment challenges:

- **Core Idea of ListNet**: Converts the ranking task into a probability distribution comparison problem. Given ground-truth relevance scores and model predicted scores for a candidate document set, both are converted via Softmax into top-1 probability distributions (representing the probability of each document being ranked first). The loss function is the Cross-Entropy (or KL-divergence) between these two distributions. This concept transfers cleanly to embedding model training: candidate similarity scores computed by the model pass through Softmax, and Cross-Entropy is computed against an ideal target distribution (e.g., where only the relevant document has probability 1 and others 0, degenerating into standard InfoNCE; or constructing a smoother target distribution if fine-grained relevance grades are available).
- **Core Idea of LambdaRank**: Bypasses defining an explicit loss function by defining gradients directly. For every pair of candidate documents, it checks whether their relative ranking order is correct; if incorrect, the magnitude of the gradient is scaled by the change in the nDCG metric ($\Delta nDCG$) resulting from swapping their ranks. This allows training to directly optimize non-differentiable ranking metrics like nDCG—a technique of using differentiable gradients to optimize non-differentiable metrics. When applied to embedding models, gradient weights for each sample pair during backpropagation are adjusted according to LambdaRank's weighting rules based on computed similarity scores from dual-encoders or cross-encoders.
- **Challenges in Transferring to Embeddings**: These classic methods were originally formulated for scenarios with a limited number of candidates per query (dozens of documents) and rich, fine-grained relevance labels (like traditional web search ranking datasets). When transferred to embedding model training where datasets are massive but rely mostly on binary weak supervision, these methods offer marginal gains over standard InfoNCE loss while incurring significantly higher engineering complexity (e.g., maintaining pairwise comparison and $\Delta nDCG$ computation logic).
- **Practical Role**: Consequently, these methods are more commonly deployed during the **fine-grained reranking / reranker** model training phase with high-quality annotated data (where candidate pools are limited and annotation quality is high, leveraging listwise strengths), rather than during large-scale weakly-supervised pre-training.

In summary, the core concepts of classic Learning-to-Rank methods like ListNet and LambdaRank can be transferred to train embedding models (especially rerankers), enabling direct optimization aligned with ranking metrics such as nDCG. However, due to their reliance on fine-grained relevance labels and manageable candidate sizes, they are better suited for high-precision reranking stages with controlled candidate counts, rather than large-scale pre-training driven by massive weakly-supervised binary data.

</details>

**21. What is the relationship and distinction between classification Softmax loss (treating retrieval as multi-class classification) and standard contrastive loss (InfoNCE) when training embedding models?**

<details>
<summary>💡 Show Answer</summary>

In some embedding and retrieval model training practices, the problem is modeled directly as a traditional multi-class classification task using standard Softmax Cross-Entropy loss. Understanding its relationship to contrastive learning loss helps provide a comprehensive perspective on loss function design space:

- **Formulation of Classification Softmax Loss**: If candidate categories (e.g., documents or labels) are **fixed and finite** (such as a fixed product taxonomy or a static knowledge base), retrieval/matching can be modeled directly as a standard multi-class classification task. A fixed output layer weight vector is learned for each candidate class. After encoding a query, the model computes similarity scores against each class weight vector and applies Softmax classification trained via standard Cross-Entropy loss.
- **Similarities to InfoNCE**: Mathematically, standard Softmax Cross-Entropy loss is strikingly similar to InfoNCE loss (both apply Softmax normalization over similarity scores followed by negative log-likelihood). InfoNCE can be viewed as a Softmax classification where the candidate set **dynamically changes at every training step** (defined by the current batch), whereas traditional classification Softmax uses a **fixed candidate set** (class weights) shared across all training steps.
- **Key Distinction: Fixed vs. Dynamic Candidate Sets**: This represents the fundamental difference. Classification loss requires the number and identity of candidate classes to be predefined and static (each class maps to a learnable weight vector), making it incapable of handling scenarios where the candidate corpus grows dynamically (e.g., in open-domain retrieval where new documents are continually indexed, making it impossible to train a fixed weight vector for every new document). In contrast, candidates in InfoNCE are generated on the fly via **real-time encoder forward passes** (i.e., another sample's embedding vector itself acts as a candidate, rather than an independently learned static class weight). Consequently, InfoNCE generalizes naturally to open-ended, growing candidate pools, explaining why contrastive loss—rather than traditional classification loss—has become the mainstream paradigm for training open-domain retrieval models.
- **Applicable Scenarios**: Classification Softmax loss is ideal for tasks with a finite and fixed set of candidate classes (e.g., fixed intent classification, product matching across closed categories), whereas InfoNCE / contrastive losses excel in open-text scenarios requiring generalization to unseen documents and queries.

In summary, classification Softmax loss and InfoNCE loss share the same mathematical foundation. The key distinction lies in the fact that classification loss relies on a fixed number of learnable class weights and cannot naturally scale to open candidate sets, whereas InfoNCE represents candidates using real-time encoder outputs, naturally supporting open-ended corpus expansion. This is the primary reason why embedding models for general retrieval almost universally adopt contrastive learning objectives over traditional classification losses.

</details>

**22. What are the differences between Encoder-only (e.g., BERT-like) and Decoder-only (e.g., LLM/GPT-derived) architectures when training embedding models?**

<details>
<summary>💡 Show Answer</summary>

Backbone architectures for embedding models primarily follow two main paradigms: Encoder-only (e.g., BERT, RoBERTa) and Decoder-only (derived from causal language models like the GPT or LLaMA series). Each has distinct characteristics:

- **Encoder-only Characteristics**: Employs bidirectional self-attention mechanisms (where each token simultaneously attends to both left and right context), making it naturally suited for tasks requiring complete contextual comprehension. Historically, it served as the dominant backbone for text embedding models (BERT, Sentence-BERT, and subsequent works). Coupled with pooling strategies like mean pooling, high-quality sentence-level representations can be directly extracted.
- **Decoder-only Characteristics**: Uses causal (unidirectional) self-attention (where each token only attends to preceding tokens and cannot see future context). Originally designed for text generation, the rapid advancement of Large Language Models (LLMs) has spurred extensive research into adapting these powerful LLM backbones into embedding models (e.g., using last-token pooling tailored for causal architectures). These models typically have significantly higher parameter counts than traditional Encoder-only models.
- **Performance Comparison**: Benefiting from vast pre-trained world knowledge and superior language comprehension/reasoning in LLM backbones, Decoder-only LLM-based embedding models have set new records across benchmarks like MTEB, particularly excelling in tasks demanding complex semantic reasoning and instruction following. However, the native causal attention mechanism poses an inherent limitation for sentence representation (earlier tokens cannot incorporate information from subsequent tokens), requiring appropriate pooling strategies (such as last-token pooling, since the final token attends to the entire prefix) or additional fine-tuning to overcome.
- **Inference Cost Trade-offs**: LLM-based embedding models possess much larger parameter sizes than BERT-like models, leading to noticeably higher inference costs (latency and VRAM footprint). In latency-critical or ultra-large-scale deployment scenarios, traditional lightweight Encoder-only models retain significant cost advantages. In practice, backbone selection requires carefully balancing performance ceilings against serving budgets.

In summary, Encoder-only architectures have long been the staple choice due to their native bidirectional attention, architectural fit for text embedding, and lower inference overhead. Decoder-only (LLM-adapted) architectures have gained leadership in benchmark accuracy due to superior pre-trained language understanding, albeit at much higher inference costs. Choosing between the two is fundamentally a trade-off between performance upper bounds and deployment budgets.

</details>

**23. What representational paradigms (e.g., GritLM) have emerged in recent years for training embedding models using Large Language Models (LLMs) as backbones?**

<details>
<summary>💡 Show Answer</summary>

With the rapid progression of LLMs, leveraging their advanced language understanding capabilities to train superior embedding models has become a prominent research direction, giving rise to several representative paradigms:

- **Directly Fine-Tuning LLMs as Dedicated Embedding Models**: Starting from a pre-trained LLM, fine-tuning is performed using contrastive frameworks (e.g., InfoNCE) alongside tailored pooling strategies (such as last-token pooling) to produce vector representations suitable for similarity computation and retrieval. This is the most direct approach; representative models built on open-source LLMs like Mistral and LLaMA frequently outperform traditional BERT-like embeddings by a wide margin on benchmarks.
- **Unified Models Combining Generation and Embedding (e.g., GritLM)**: A more advanced approach trains a unified model possessing **both** text generation and embedding representation capabilities simultaneously. By utilizing task instructions or formatting templates on a single model, it distinguishes whether the current query requires a "generation task" or an "embedding task" (similar to instruction-aware design). Joint training is performed using appropriate loss functions for each task (standard causal language modeling loss for generation and contrastive loss for embeddings). Such a unified model can handle both "document retrieval" (embedding capability) and "retrieval-augmented answer generation" (generation capability) in a RAG pipeline, eliminating the need to deploy and maintain two separate models.
- **Advantages of Unified Models**: In addition to simplifying deployment (one model serving dual capabilities), joint modeling theoretically enables positive knowledge transfer—deep semantic reasoning acquired during generation can enhance embedding representation quality, and vice versa.
- **Key Challenges**: Jointly training two distinct task objectives requires carefully balanced multi-task training strategies (e.g., loss weighting and prompt formatting for task discrimination). Furthermore, these unified models tend to be large in scale, incurring substantial inference overhead. For cost-sensitive applications requiring only embedding functionality, lightweight dedicated embedding models remain a more suitable choice.

In summary, moving from fine-tuning LLMs into dedicated embedding models to training unified generation-and-embedding architectures like GritLM represents a crucial direction in harnessing LLM intelligence for embeddings. While offering marked accuracy gains, these methods incur higher training and inference costs. Their adoption depends on balancing benchmark requirements, deployment simplicity, and operational budgets.

</details>

**24. How should output vector dimensions be selected when designing embedding models, and what issues arise when dimensions are too high or too low?**

<details>
<summary>💡 Show Answer</summary>

Embedding vector dimension is a critical design parameter that requires balancing retrieval accuracy, storage overhead, and search efficiency:

- **Issues with Excessively Low Dimensions**: Low vector dimensions (e.g., tens of dimensions) severely restrict information capacity, making it difficult to fully express complex text semantics. Especially in tasks requiring subtle semantic distinctions, low-dimensional vectors lead to poor retrieval precision due to insufficient representation capacity (forcing distinct semantics into a tight spatial representation, causing confusion).
- **Issues with Excessively High Dimensions**: While high dimensions (e.g., thousands of dimensions) theoretically increase representation capacity, they introduce practical challenges: storage costs scale linearly with dimensionality (imposing a heavy financial burden in large-scale vector databases, which quantization/dimensionality reduction techniques aim to relieve); retrieval computation (vector comparisons in both brute-force and ANN searches) also grows with dimension. Furthermore, excessively high dimensions increase over-fitting risks when training data is limited (a manifestation of the curse of dimensionality), meaning empirical performance does not strictly increase monotonically with dimension.
- **Industry Empirical Standards**: Current mainstream open-source embedding models predominantly use output dimensions around 384, 768, or 1024, representing well-validated balances between expressiveness and cost. High-accuracy commercial APIs may offer larger choices (e.g., 3072 dimensions) for applications that tolerate higher serving expenses.
- **Modern Practices via Matryoshka Representation Learning**: Technologies like Matryoshka Representation Learning (MRL) optimize representation quality across multiple truncated dimensions during training. This allows a single trained model to flexibly output lower dimensions (saving storage/compute) or higher dimensions (maximizing accuracy) at inference time. MRL alleviates the traditional constraint of fixing a single dimension prior to training, serving as a highly recommended design pattern for new embedding architectures.

In summary, selecting embedding vector dimensions is fundamentally a trade-off between representation capacity (higher theoretical upper bound with more dimensions) and computational/storage costs (alongside over-fitting risks). An empirical sweet spot of 384–1024 dimensions has emerged across the industry, while modern techniques like Matryoshka Representation Learning offer dynamic flexibility to trade off accuracy and overhead at runtime without locking into irreversible training-time dimensional choices.

</details>

**25. Transformer encoders typically have a fixed maximum input length limit (e.g., 512 tokens); how can embedding models be extended to handle longer documents?**

<details>
<summary>💡 Show Answer</summary>

Most embedding models built on architectures like BERT have fixed input length constraints (commonly 512 tokens). Inputs exceeding this limit must be truncated or specially processed. Common technical avenues for extending this boundary include:

- **Positional Encoding Extrapolation and Interpolation**: Input length limits in Transformers largely stem from positional encodings trained on bounded sequence lengths; performance drops sharply on longer unseen contexts. Recent positional encoding advances (such as Rotary Position Embedding [RoPE] and RoPE-based extrapolation techniques like NTK-aware interpolation and YaRN) apply mathematical transformations to positional encodings. This enables expanding effective context lengths by several folds without full retraining (or with minimal continued pre-training).
- **Continued Pre-Training/Fine-Tuning on Longer Sequences**: A more direct approach is conducting continued pre-training or fine-tuning on long-text corpora, allowing the model to observe extended inputs and learn long-range dependency modeling. While producing more reliable results, this requires extra compute and long-text training datasets.
- **Hierarchical and Segmented Encoding Strategies**: For documents far exceeding hard model limits, an engineering workaround splits the document into segments within token limits, encodes each separately, and aggregates segment vectors (e.g., via mean pooling or a lightweight aggregation model) to produce the document embedding. While conceptually aligned with chunking, this strategy operates at the **model architecture and inference pipeline level**, whereas chunking is a data preprocessing technique for retrieval granularity in RAG. Both can be combined effectively.
- **Recent Advances in Long-Context Embedding Models**: Recently released embedding models (particularly those adapted from long-context LLMs) natively support thousands or tens of thousands of tokens. This mitigates reliance on workaround strategies like segmented encoding, representing a prominent trend in embedding architecture evolution.

In summary, expanding an embedding model's ability to handle long documents can be achieved natively via positional encoding improvements (such as RoPE extrapolation) and targeted long-text continued training, or at the application level via segmented encoding and aggregation workarounds. Additionally, long-context LLM-based embeddings are pushing native input limits significantly higher, making long-context support a key criterion when evaluating embedding backbones.

</details>

**26. Do similarity metrics between embedding vectors strictly satisfy mathematical distance axioms (e.g., the triangle inequality), and what is the practical impact on retrieval system design?**

<details>
<summary>💡 Show Answer</summary>

From a theoretical standpoint, an important nuance regarding similarity metrics between embedding vectors is that not all popular metrics strictly satisfy formal mathematical distance axioms. This key property impacts retrieval algorithm designs that rely on geometric distance assumptions:

- **Summary of Distance Axioms**: A strict metric distance must satisfy non-negativity, identity of indiscernibles (distance to self is zero), symmetry, and the **triangle inequality** ($d(a,c) \le d(a,b) + d(b,c)$, meaning a path via an intermediate point cannot be shorter than direct distance).
- **Properties of Cosine Similarity and Cosine Distance**: Cosine **similarity** itself is not a distance metric (larger values imply higher similarity, contrasting with distance intuition where smaller values mean proximity). However, it can be converted into Cosine **distance** ($1 - cos_sim$). Under L2 normalization, cosine distance mathematically satisfies the triangle inequality, providing theoretical guarantees for cosine distance-based retrieval systems.
- **Why This Property Matters**: Many efficient ANN indexing algorithms (especially early tree-based methods or spatial partitioning algorithms reliant on distance pruning) rely on the triangle inequality to prune search paths (e.g., "if distance $A$-to-$B$ is large and $B$ is close to query $Q$, the triangle inequality can prune $A$ without explicitly computing distance $A$-to-$Q$"). If the chosen metric violates the triangle inequality, distance-based pruning might prune valid top-$k$ candidates incorrectly.
- **Practical Impact and Engineering Choices**: Fortunately, standard mainstream setups (using dot product / cosine distance on L2-normalized vectors) satisfy distance axioms. Furthermore, modern graph-based ANN algorithms like HNSW do not rely strictly on triangle inequality pruning, avoiding these issues. However, if custom scoring functions (e.g., combining business rules into weighted similarity scores) are introduced, system designers must audit whether these custom functions violate mathematical distance axioms, preventing subtle retrieval errors caused by invalid pruning assumptions.

In summary, mainstream cosine similarity/distance is mathematically well-behaved (satisfying distance axioms) and can be safely used alongside distance-dependent retrieval algorithms. However, when complex, non-standard custom scoring functions are introduced for domain-specific needs, designers must verify whether distance axioms (especially the triangle inequality) still hold to avoid subtle search errors from invalid algorithm pruning.

</details>

**27. When Training Embedding Models for Retrieval Scenarios, Should Candidates for Hard Negative Mining Come from ANN Index Retrieval Results or Random Sampling? What Are the Respective Issues with Both Approaches?**

<details>
<summary>💡 Show Answer</summary>

Various algorithms for hard negative mining (such as ANCE) have been discussed earlier. Here, we delve deeper into a specific engineering problem: the candidate source used for hard negative mining (ANN index retrieval vs. full-corpus random sampling) leads to different performance characteristics and engineering trade-offs:

- **Using ANN index retrieval results as the source of hard negatives**: Using the ANN index built by the current model (or its earlier version), top-ranked documents retrieved for each query that are annotated as irrelevant are selected as hard negatives. This approach can efficiently locate candidates that the model currently deems "similar" (without traversing the entire document corpus to calculate similarity scores), making it standard practice in methods such as ANCE;
- **Index staleness problem (Staleness)**: Because model parameters are continuously updated during training, while rebuilding the entire ANN index (re-encoding all documents in the corpus and constructing index structures) is typically computationally expensive and time-consuming, it is impractical to rebuild the index after every training step. Consequently, in practice, indices are usually updated "periodically" (e.g., rebuilt every few training epochs or fixed number of steps). This implies that the index version used for mining hard negatives may lag behind the **currently** training model parameters (the index is "stale"), and the mined "hard negatives" may not perfectly align with the model's true current decision boundary;
- **Full-corpus random sampling**: Directly sampling negative examples at random from the entire document corpus without relying on the model's current discrimination capability. This approach is simple to implement and avoids index staleness. However, as repeatedly mentioned earlier, most random negatives are "easy negatives," which offer limited training signal value for enhancing the fine-grained discriminative capability of the model;
- **Engineering trade-offs and compromise solutions**: In practice, a trade-off strategy of "periodic index rebuilding + mixing random negatives with index-mined hard negatives during training" is commonly adopted. This approach tracks the improvement of model capabilities through periodic index rebuilding (despite a certain staleness lag), while mitigating the negative impacts of staleness and ensuring training signal diversity through mixed random negatives. The index rebuilding frequency is an engineering hyperparameter that must be tuned through experimentation, considering the total number of training steps, index construction costs, and the model's sensitivity to hard negative quality.

In summary, mining hard negatives via ANN index retrieval provides targeted training signals that align closely with the model's current capability boundary. However, constrained by index rebuilding costs, it inevitably suffers from index staleness. Full-corpus random sampling is simple and reliable, but its signal value is limited. In practice, combining both approaches and carefully setting the index rebuilding frequency to manage the impact of staleness on training performance are crucial engineering details when deploying hard negative mining.

</details>

**28. In Production Environments, How Should a Decision Framework Be Designed to Select a Retrieval Architecture (Dual-Encoder, Multi-Vector, Cross-Encoder Combinations)?**

<details>
<summary>💡 Show Answer</summary>

Combining the previously discussed retrieval architectures (Dual-Encoders, ColBERT-style Multi-Vectors, and Cross-Encoders) along with their respective pros and cons, a systematic architecture selection framework can be constructed along the following dimensions for practical production deployment:

- **Candidate pool size**: The size of the candidate document pool is the primary factor dictating architecture selection. When the candidate pool reaches millions or more, a retrieval layer that supports efficient offline indexing and fast online search is mandatory (dual-encoders or sparse retrieval are essential choices; cross-encoders cannot serve this role). When the candidate size is small (e.g., thousands to tens of thousands), one can even consider applying cross-encoders directly to perform exhaustive scoring on all candidates, eliminating complex multi-stage architectures;
- **Latency budget**: Business tolerance for retrieval latency directly dictates how computationally heavy subsequent processing can be after retrieval. Scenarios with extremely strict latency requirements (e.g., real-time low-millisecond responses) may have to rely solely on pure dual-encoder retrieval without additional reranking stages. Scenarios with more relaxed latency constraints (e.g., accepting hundreds of milliseconds) can stack cross-encoder reranking on top of dual-encoder retrieval, or even introduce ColBERT-style multi-vector solutions as a transitional layer between retrieval and reranking;
- **Accuracy requirements and business value**: For domain scenarios extremely sensitive to retrieval precision where misrankings incur substantial business losses (such as medical or legal specialized retrieval), it is worth introducing a cross-encoder reranking stage to guarantee ranking quality despite added latency and cost. For scenarios with more relaxed precision requirements (such as auxiliary retrieval for internal knowledge bases), a pure dual-encoder solution may suffice;
- **Storage and cost budget**: As previously discussed, multi-vector (ColBERT) solutions incur significantly higher storage costs than single-vector dual-encoders. If vector storage and index maintenance constitute clear cost bottlenecks, one must carefully evaluate whether the accuracy gains justify these extra expenses, or consider replacing them with a combination of dual-encoders plus a stronger reranker model;
- **Data and iteration capability**: The ability to continuously accumulate high-quality relevance annotations, conduct hard negative mining, and iterate on models also influences architecture selection. Introducing a more complex multi-stage architecture (especially reranking stages) typically requires corresponding high-quality training data to unlock its full potential. If data accumulation and iteration capabilities are limited, a simple and reliable architecture is a more pragmatic starting point, launching a simple pipeline first to collect business data and feedback before gradually evolving the architecture.

In summary, there is no one-size-fits-all optimal answer for retrieval architecture selection. A systematic evaluation must revolve around candidate pool size, latency budget, accuracy requirements, cost constraints, and team data/iteration capabilities. In practice, a pragmatic path is typically to start with a simple architecture (e.g., pure dual-encoder) for rapid deployment, and then gradually evolve toward more complex multi-stage architectures (incorporating reranking, hybrid search, etc.) based on actual business feedback and accumulated data, rather than pursuing an all-inclusive complex architecture from the outset.

</details>

