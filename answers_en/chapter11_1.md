# 11.1 Frontiers in Model Architecture — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. Is It Possible for Mamba-like Models to Replace Transformer in the Future?**

<details>
<summary>💡 Show Answer</summary>

Mamba is a new architecture based on State Space Models (SSMs), featuring advantages such as linear time complexity and fixed-size state representation.

**Advantages of Mamba:**
- High inference efficiency: Linear complexity with respect to sequence length (quadratic for Transformer), making it significantly faster when processing long sequences;
- Memory-friendly: No need to store full KV Cache, maintaining constant GPU memory usage during inference;
- Long sequence modeling: Theoretically capable of processing sequences of arbitrary length without increasing computational overhead.

**Limitations of Mamba:**
- Parallel training efficiency: The recurrent structure of SSMs yields lower training parallelism than Transformer, though hardware-friendly Parallel Scan algorithms can mitigate this;
- In-context learning (ICL) capability: Current experiments indicate that Mamba performs worse than Transformer on ICL and few-shot tasks;
- Expressive capacity: Whether the inductive bias of Selective SSM is sufficient to match full attention mechanisms remains controversial;
- Insufficient scaling verification: Currently, the scale of Mamba models is still smaller than mainstream LLMs, and its performance under large-scale training remains to be verified.

**Conclusion:** Mamba will not replace Transformer in the short term, but **hybrid architectures** (Mamba layers + Attention layers) represent a highly promising direction that can combine the strengths of both. In the long term, if breakthroughs are achieved in ICL and reasoning capabilities, Mamba-like models are expected to become strong alternatives to Transformer in certain scenarios (such as long sequences and edge deployment).



![Transformer 与 Mamba 架构信息流对比，突出内存与计算复杂度差异 (Comparison of information flow between Transformer and Mamba architectures, highlighting memory and computational complexity differences.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig01_mamba_vs_transformer.png)

</details>

**2. What Are the Engineering Advantages and Limitations of RWKV?**

<details>
<summary>💡 Show Answer</summary>

RWKV is a model combining ideas from both RNNs and Transformers; it trains like a Transformer and infers like an RNN.

**Engineering Advantages:**
1. **Extremely High Inference Efficiency**: During inference, the state is of fixed size, with time and space complexity both being $O(1)$/token, making it suitable for resource-constrained devices;
2. **No KV Cache Overhead**: It does not require a KV Cache that grows with context length, keeping long-text inference memory constant;
3. **Streaming-Friendly**: The RNN structure naturally supports real-time streaming input;
4. **Parallel Training**: Parallel computation during the training phase is achieved via the WKV operator, free from the serial restriction of RNNs.

**Limitations:**
1. **Limited Recall Capability**: The RNN state compresses historical information, making it weaker than full attention mechanisms when precise recall of early context (such as key details in ultra-long documents) is required;
2. **Upper Bound on Expressive Capacity**: Studies show that RWKV is weaker than Transformers of equivalent scale on complex reasoning and in-context learning tasks;
3. **Positional Sensitivity**: RWKV's time-decay mechanism is highly sensitive to token order, resulting in a significant performance drop under out-of-order inputs;
4. **Relatively Immature Ecosystem**: Tooling, fine-tuning frameworks, and community support still lag behind those of Transformer.

</details>

**3. Where Is the Limit of Long-Context Extrapolation in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Long-context extrapolation faces multiple constraints:

**Technical Bottlenecks:**
1. **Attention Complexity**: Standard Self-Attention computational complexity is $O(n^2)$; a sequence of 128K tokens already consumes vast amounts of computation, making million-token levels prohibitively expensive;
2. **Positional Encoding Extrapolation**: Positional encodings such as RoPE suffer performance degradation when exceeding the training length; although mitigation solutions like YaRN and LongRoPE exist, their effectiveness has an upper bound;
3. **Perplexity Paradox**: Evaluations indicate that even when extrapolating to 1M tokens, the model's actual "effectively utilized" context window may be far smaller than this (the "Lost in the Middle" phenomenon);
4. **Memory Constraints**: KV Cache grows linearly with context length; a 100K-token KV Cache may require tens of gigabytes of GPU memory.

**Practical Limits:**
- Current commercial models (Gemini 1.5 Pro 200K, Claude 3 series 200K, GPT-4 128K) perform quite well at the scale of hundreds of thousands of tokens;
- Research-level work has explored up to 1M tokens (Gemini experiments, MegaByte, etc.), though actual comprehension capacity degrades;
- The limit is constrained more by **effective information utilization efficiency** than by a purely technical upper bound on length.

</details>

**4. What Are the Future Development Trends of MoE Models?**

<details>
<summary>💡 Show Answer</summary>

Future trends for Mixture-of-Experts (MoE) models include:

1. **Finer-Grained Expert Partitioning**: Evolving from token-level routing toward finer-grained (sub-token, feature-dimension) routing to improve expert utilization;
2. **Expert Specialization**: Training experts with true domain division of labor (e.g., code experts, math experts, multilingual experts), rather than random allocations;
3. **Dynamic Number of Experts**: Adaptively activating different numbers of experts based on task complexity (Adaptive Compute);
4. **MoE + Multimodality**: Using different experts for different modalities, routing image tokens and text tokens to different expert groups (e.g., MoE-LLaVA);
5. **Communication Optimization**: Resolving All-to-All communication overhead in distributed MoE represents a core engineering challenge; expert localization (combining Expert Parallelism and Data Parallelism);
6. **Training Stability**: Expert Collapse (over-utilization of certain experts) is the central issue in training stability; technologies like load balancing loss and Expert Dropout continue to evolve;
7. **MoE-ification of Small Models**: Applying MoE concepts to small models (e.g., mobile device models) to achieve high performance with a small active parameter footprint via sparse activation.

</details>

**5. What Trends May Emerge in LLM Architectures Over the Next 3 Years?**

<details>
<summary>💡 Show Answer</summary>

Predictions based on current research directions:

1. **Mainstreaming of Hybrid Architectures**: Hybrid architectures combining Transformer + SSM (Mamba) or Attention + Linear Attention will become mainstream, balancing long-sequence efficiency with attentional expressive capacity;
2. **Native Multimodal Architectures**: Moving away from stitching "LLM + visual adapter," toward native multimodal Transformers that uniformly model image, audio, and video tokens alongside text;
3. **Ultra-Long Context as Standard**: Million-token context windows will gradually become standard, with efficient mechanisms such as sparse attention and linear attention becoming standard features;
4. **Inference-Efficiency-Oriented Design**: Model architecture design will place greater emphasis on inference-time KV Cache efficiency and Memory Bandwidth utilization (further deepening directions like GQA and MLA);
5. **Refinement of MoE**: Transitioning from coarse-grained experts to fine-grained routing, with higher expert specialization and mature Expert Parallelism engineering;
6. **Architecture Optimization for On-Device Deployment**: Small-model architectures specially designed for mobile/edge devices (sub-billion parameters) will become an independent track, incorporating quantization, distillation, and architecture search;
7. **Architectural Support for Test-Time Compute**: Designing friendly architectures for expanding computation during the inference phase (Chain-of-Thought, MCTS, Self-Consistency), such as Diffusion LMs and parallel decoding.



![未来三年大语言模型架构路线图与演进趋势 (Architectural roadmap and evolution trends of Large Language Models over the next three years.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig05_llm_architectures_3years.png)

</details>

**6. How to Choose Between Ultra-Large Models and Efficient Small Models?**

<details>
<summary>💡 Show Answer</summary>

Both paths have their respective application scenarios and are not mutually exclusive:

**Advantages of Ultra-Large Models:**
- High capability ceiling: Large models hold a clear advantage in tasks involving complex reasoning, creative generation, and multi-step planning;
- Strong few-shot / zero-shot generalization;
- Emergent capabilities: Certain abilities emerge only in large models;
- Suitable for building general-purpose intelligence infrastructure.

**Advantages of Efficient Small Models:**
- Low deployment cost: Capable of running on consumer GPUs, mobile phones, and edge devices;
- Low inference latency: Satisfies real-time performance requirements;
- Data security: Local deployment poses no privacy leak risks;
- Performance on specific tasks approaches or even surpasses large models (after thorough fine-tuning).

**Selection Principles:**
- **Task Complexity**: Complex, open-ended tasks (writing, planning, multi-step reasoning) $\rightarrow$ Large models; Simple, structured, domain-specific tasks $\rightarrow$ Small models;
- **Deployment Constraints**: Cloud services $\rightarrow$ Large models are feasible; Edge/Local/Low-latency $\rightarrow$ Small models;
- **Cost Budget**: Training and inference costs dictate the choice of path;
- **Future Trends**: Large models drive the exploration of capability frontiers, while small models drive large-scale practical deployment. The two paths will coexist long-term and mutually reinforce each other through distillation.

</details>

**7. What Is the Core Idea of Linear Attention?**

<details>
<summary>💡 Show Answer</summary>

Standard Softmax Attention has a computational complexity of $O(n^2)$ (where $n$ is the sequence length). Linear attention approximates $softmax(QK^\top)V$ with $\phi(Q)(\phi(K)^\top V)$ (where $\phi$ is a kernel function / feature map), leveraging the associative property of matrix multiplication to reduce complexity to $O(n)$.

**Core Trick:** Rearranging attention computation to "first compute the cumulative state of $K^\top V$, and then multiply by $Q$," allowing it to recurse with a fixed-size state like an RNN, naturally supporting streaming inference without needing to store a full KV Cache that grows with sequence length during inference.

**Representative Works:**
- Linear Transformer: First proposed using kernel functions to approximate softmax;
- Performer: Uses random features (FAVOR+) to approximate the softmax kernel;
- RetNet: Introduces an explicit decay mechanism on top of linear attention, accommodating both parallel training and recurrent inference representations.

**Limitations:** Linearization sacrifices part of the "sharp" selectivity of softmax, usually performing weaker than standard attention on tasks requiring precise retrieval/copying. In practice, it is often blended with local window attention or a few full-attention layers to compensate for its shortcomings.

</details>

**8. What Are the Advantages of Hybrid Architectures Combining State Space Models and Transformers (e.g., Jamba)?**

<details>
<summary>💡 Show Answer</summary>

Pure SSMs (such as Mamba) are relatively weak in in-context learning and precise retrieval tasks; conversely, pure Transformers suffer from high memory and computational costs during long-sequence inference. A hybrid architecture alternately stacks SSM layers and a small number of Attention layers within the same model, attempting to combine the strengths of both.

**Advantages:**
1. The majority of layers use SSM, significantly reducing KV Cache memory footprint and long-sequence inference cost;
2. A small number of Attention layers retain precise long-distance retrieval and in-context learning capabilities, covering the shortcomings of pure SSMs on these tasks;
3. Existing practices (e.g., Jamba, Zamba) demonstrate that under an equivalent parameter count, hybrid architectures can achieve performance close to pure Transformers, while inference throughput and memory efficiency are significantly superior to pure Transformers.

**Future Directions:** The optimal ratio of SSM to Attention layers and their inter-layer arrangement (whether uniformly alternating or stage-stacked) remain active research directions, and different task distributions impose varying requirements on the optimal ratio.

</details>

**9. Can Diffusion Models Be Used for Text Generation? What Are Their Pros and Cons Compared to Autoregressive Models?**

<details>
<summary>💡 Show Answer</summary>

Text diffusion models iteratively restore a complete sequence all at once from pure noise by defining forward noise-addition and reverse denoising processes over discrete tokens (or their continuous embeddings), rather than generating token-by-token like autoregressive models.

**Advantages:**
1. Generating the entire sequence in parallel theoretically drastically reduces generation latency, yielding noticeable benefits especially for long-text scenarios;
2. Naturally supporting bidirectional context modeling and "infilling" generation (such as modifying intermediate content), making them better suited for controllable generation and local editing tasks.

**Disadvantages:**
1. Diffusion modeling for discrete data is more difficult than for continuous images, requiring specialized designs (e.g., discrete diffusion, embedding-space diffusion);
2. At present, the generation quality and training scalability of text diffusion models still lag behind mainstream autoregressive LLMs;
3. There is a trade-off between sampling steps and generation quality; reducing denoising steps significantly sacrifices generation quality.

**Conclusion:** Autoregression remains the mainstream paradigm for text generation in the short term, but diffusion models have shown potential in niche scenarios such as low latency and controllable editing, representing an exploratory direction worth following.

</details>

**10. What Is the IO-Aware Optimization Idea of the FlashAttention Series?**

<details>
<summary>💡 Show Answer</summary>

Standard attention implementations require materializing an $O(n^2)$-sized attention matrix into GPU memory, repeatedly reading and writing between HBM (High-Bandwidth Memory) and SRAM (on-chip cache), which becomes the main bottleneck in long-sequence scenarios.

**Core Idea of FlashAttention:** Utilizing tiling techniques, Q/K/V are divided into small blocks suitable for fitting into SRAM. Local softmax and weighted summation are performed inside SRAM, avoiding the materialization of the full attention matrix via online softmax (which maintains running maximums and normalization terms for incremental updates), thereby reducing HBM memory access from $O(n^2)$ down to $O(n)$.

- FlashAttention-1 primarily solves the memory wall issue during training, trading recomputation for memory savings;
- FlashAttention-2 further optimizes parallelization strategies and warp-level work partitioning, reducing the proportion of non-matrix-multiplication operations and achieving noticeably higher measured throughput;
- FlashAttention-3 specializes optimizations for Hopper architecture's asynchronous execution units and low-precision (FP8) computation, further squeezing hardware utilization.

**Significance:** FlashAttention is a classic example of "keeping the algorithm unchanged while optimizing memory access patterns," demonstrating that inference/training efficiency bottlenecks often stem not from computation itself, but from the memory wall—serving as the conceptual origin for a series of subsequent works including KV Cache compression and PagedAttention.



![FlashAttention 系列的分块机制与 IO 感知内存访问模式 (Tiling mechanism and IO-aware memory access pattern in FlashAttention series.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig10_flashattention_io.png)

</details>

**11. What Is Native Sparse Attention (NSA)? How Does It Differ from Traditional Sparse Attention?**

<details>
<summary>💡 Show Answer</summary>

Traditional sparse attention (such as sliding window, local+global hybrid, Longformer-style patterns) mostly consists of sparsification schemes applied via **post-processing during inference** or **post-training fine-tuning adaptation**, which easily causes inconsistencies between training and inference behaviors. Moreover, sparse patterns are often manually designed fixed rules, making them difficult to adapt to information distributions across different tasks.

**Core Idea of NSA:** Designing the sparsification mechanism as a **natively trainable** module, allowing the model to learn which token blocks to "selectively attend to" as early as the pre-training stage. It typically combines three parallel information pathways: compressed coarse-grained global information, dynamically selected important token blocks (Top-$K$ block selection), and a local sliding window, with the outputs of all three being fused.

- Advantages: Because the sparse pattern is learned end-to-end, it fits actual attention distributions better than manual rules. Meanwhile, hardware-friendly tiled computation patterns are used right from the training stage, ensuring consistent acceleration across training and inference without behavior drift like "dense in training, sparse only in inference";
- Costs: It requires redesigning training kernels, resulting in significantly higher engineering implementation complexity than standard attention, and the sparse selection mechanism itself introduces extra routing/scoring overhead.

</details>

**12. What Are the Design Trade-Offs of Grouped-Query Attention (GQA) Compared to Multi-Head Latent Attention (MLA)?**

<details>
<summary>💡 Show Answer</summary>

**Idea of GQA:** Allowing multiple Query heads to share the same set of Key/Value heads (rather than each Query head having an independent KV as in MHA, or all Query heads sharing a single set of KV as in MQA), striking a compromise between KV Cache size and model expressive capacity.

**Idea of MLA:** Instead of simply reducing the number of KV heads, KV is first projected into a low-rank latent vector. During inference, only this low-dimensional latent representation needs to be cached, and it is dynamically uncompressed into full K/V during actual Attention computation, balancing memory footprint and expressive capacity.

**Comparison:**
- GQA is a direct compression of "reducing the number of heads." It is simple to implement, but the loss in expressive capacity directly correlates with the reduction in heads;
- MLA is "low-rank compression," theoretically retaining more information under the same KV Cache size. DeepSeek's practice demonstrates that MLA can outperform GQA/MQA at equivalent compression rates;
- The cost of MLA lies in a more complex computation graph during training and inference (requiring the maintenance of two equivalent forms: "uncompressed MHA in training state" and "compressed MQA in decoding state"), making engineering implementation and debugging significantly more challenging.



![多头注意力、分组查询注意力与多头潜变量注意力中的 KV 缓存结构对比 (Comparison of Key-Value (KV) cache structures in Multi-Head, Grouped-Query, and Multi-Head Latent Attention.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig12_gqa_vs_mla.png)

</details>

**13. What Are the Recent Advances in Positional Encoding Extrapolation Schemes for Long-Context Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Beyond traditional absolute positional encodings and learnable positional encodings, several major lines of work have emerged in recent years surrounding RoPE extrapolation:

1. **Interpolation-Based Methods**: Such as Position Interpolation and YaRN. By applying non-linear scaling/interpolation to RoPE rotation frequencies, the model "perceives" during inference that the sequence does not exceed the training length. YaRN further applies differentiated scaling strategies across frequency dimensions (less scaling for high-frequency dimensions, more scaling for low-frequency dimensions), outperforming linear interpolation in extrapolation performance;
2. **Segmented Extension Methods**: Such as LongRoPE. It uses evolutionary search to find optimal scaling factors for each dimension and combines progressive short-to-long training strategies to achieve extrapolation to million-token levels;
3. **No/Weak Positional Encoding**: Research on NoPE (No Positional Embedding) reveals that pure Decoder-only causal attention itself can implicitly encode positional information through causal masking; completely removing explicit positional encodings actually leads to more robust performance in certain extrapolation scenarios;
4. **Attention-Constraint Methods**: Such as ALiBi. By adding a linear bias proportional to relative distance onto attention scores, it naturally possesses length extrapolation capability without needing to observe long sequences during training.

**Common Limitation:** Regardless of the extrapolation scheme, a gap remains between "the extrapolated length" and "the length the model can effectively utilize" (i.e., the "Lost in the Middle" phenomenon). Extrapolation solves the issue of "not collapsing" rather than "fully utilizing."

</details>

**14. What Is the Attention Residuals Architecture? What Problem Does It Attempt to Solve?**

<details>
<summary>💡 Show Answer</summary>

The residual connection in a standard Transformer adds the input itself to the output of each sublayer (Attention / FFN), i.e., $x_{l+1} = x_l + Sublayer(x_l)$. As network depth increases, this design encounters an **information dilution** problem—as the residual stream continuously accumulates, the proportion of information written by earlier layers becomes smaller, leading to diminishing marginal growth in the effective capacity of deeper models.

**Idea of Attention Residuals:** Entrusting the residual connection itself to be learned by an attention mechanism—rather than simply adding back the previous layer's output as-is, the model uses an attention-like mechanism to "selectively" decide how much and which information to read from historical layers to add to the current layer, thereby alleviating the attenuation of information with depth in deep networks.

**Effects and Costs:** Related practices show that such architectures can lower training loss and boost training efficiency under equivalent compute budgets, with only limited increases in inference latency. However, the cost is that the residual path is no longer an identity mapping, requiring extra parameters and computation to maintain the cross-layer reading mechanism, which elevates the complexity of engineering implementation and numerical stability tuning.

</details>

**15. What Are the Recent Advances in Expert Routing Mechanisms for MoE Models?**

<details>
<summary>💡 Show Answer</summary>

On top of foundational paradigms such as top-$K$ routing and load balancing loss, recent improvements in MoE routing mainly concentrate on the following directions:

- **Fine-Grained Experts**: Splitting experts into smaller and more numerous units and activating more but smaller experts each time. Theoretically, this allows combining a richer subset of capabilities at a finer granularity. Practices in models like DeepSeek have demonstrated that this design can achieve better results with the same number of activated parameters;
- **Shared Experts**: Establishing several shared experts that always participate in computation to specifically absorb general and high-frequency knowledge, leaving sparsely routed experts for more specialized capabilities and alleviating redundancy where different experts learn duplicate content;
- **Auxiliary-Loss-Free Load Balancing**: Traditional aux loss interferes to some extent with the optimization direction of the main task. Recent solutions achieve load balancing by dynamically adjusting routing biases for each expert (rather than directly penalizing the loss), reducing interference with the main task gradients;
- **Cross-Layer Expert Sharing**: Exploring the reuse of partial expert parameters across adjacent layers to further compress the total parameter count, though its impact on model expressive capacity is still under verification.

</details>

