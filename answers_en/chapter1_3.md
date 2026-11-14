# 1.3 Training and Inference — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 1: Basic Knowledge of Large Models · 18 questions

**1. What is the Training Objective of Large Language Models (LLMs)?**

<details>
<summary>💡 Show Answer</summary>

Predicting the next word based on preceding words, where the training objective is maximum likelihood:

**Training Efficiency**:
- Prefix Decoder $<$ Causal Decoder
- Causal Decoder architectures calculate loss on all tokens, whereas Prefix Decoder only calculates loss on output tokens.

Randomly replacing certain text spans and training the language model to reconstruct the corrupted text spans. The objective function is:

$$
    \mathcal{L} = -\sum_{i} \log P(w_i | w_{<i})
$$


**Denoising Autoencoder**:
- Higher implementation difficulty
- Tasks/Models adopting denoising autoencoders as their training objective include GLM-130B and T5

</details>

**2. How Are LLMs Typically Trained?**

<details>
<summary>💡 Show Answer</summary>

The core of LLM training is a Transformer-based neural network with billions of parameters. These parameters connect nodes across layers, enabling the model to learn complex relationships. LLMs are trained on massive datasets of high-quality text and code. This data provides the model with the raw material for learning language patterns. During training, the model predicts the next word in a sequence based on preceding words. It then adjusts its internal parameters to improve predictions, essentially self-learning through vast amounts of examples. After training is complete, the LLM can be fine-tuned for specific tasks.

</details>

**3. What Are L1 and L2 Regularization, and Which Scenarios Are They Suitable For?**

<details>
<summary>💡 Show Answer</summary>

Both L1 and L2 regularization are common techniques used in machine learning and deep learning to prevent model overfitting. They achieve this goal by adding a penalty term representing model complexity to the loss function of the model.

	**L1 Regularization**

- Definition: The penalty term added by L1 regularization is the sum of the absolute values of all weight parameters $w_i$ of the model, multiplied by a regularization coefficient $\lambda$.

\[Loss_{L1} = Original Loss + \lambda \sum_{i} |w_i|\]

- Core Role: Inducing Sparsity.

During gradient descent optimization, the L1 penalty term drives the weights of features that contribute little to the model to become exactly 0. This is equivalent to completely removing these features from the model.

- Applicable Scenarios: Feature Selection.

When your dataset contains a large number of features but you suspect many of them are redundant or useless, L1 regularization is extremely useful. It can automatically "screen out" the most important features, simplifying the model and improving interpretability.

	**L2 Regularization**

- Definition: The penalty term added by L2 regularization is the sum of the squares of all weight parameters $w_i$ of the model, multiplied by a regularization coefficient $\lambda$.

\[Loss_{L2} = Original Loss + \lambda \sum_{i} w_i^2\]

- Core Role: Weight Decay.

L2 regularization penalizes large weight values, encouraging model weight parameters to be as small as possible—approaching 0 but generally not becoming exactly 0. This makes the model's weight distribution smoother and more diffused, preventing the model from excessively relying on a few high-weight features.

- Applicable Scenarios: General Overfitting Prevention.

L2 is a more common and general regularization method. When features may be correlated (collinear) or when you believe most features contribute to the prediction to some degree, L2 is the first choice. It effectively enhances the model's generalization ability, performing better on unseen data. In deep learning, "weight decay" usually refers to L2 regularization.

Summary Comparison: | Comparison Item | L1 Regularization | L2 Regularization | | :----------- | :------------------------------------- | :----------------------- | | Penalty Term | Sum of absolute values of weights (L1 norm) | Sum of squares of weights (L2 norm) | | Effect | Weight sparsity, some weights become 0 | Weight smoothing, weights approach 0 | | Main Use | Feature selection, model simplification | Prevent overfitting, enhance generalization ability | | Solution Property | Unstable, small data changes may cause feature set changes | Stable, solution is unique |

</details>

**4. What Major Engineering and Algorithmic Challenges Will You Face When Training an LLM with Tens or Hundreds of Billions of Parameters? (e.g., VRAM, Communication, Training Instability, etc.)**

<details>
<summary>💡 Show Answer</summary>

Training an LLM with tens or hundreds of billions of parameters is a massive systems engineering endeavor involving deep co-design across hardware, software, and algorithms. Its main challenges manifest in the following three aspects:

1. Memory Wall Challenge:

- Problem: A 100-billion-parameter model requires several TBs of memory in total for its parameters, gradients, and optimizer states (such as momentum and variance in Adam), which far exceeds the memory capacity of any single GPU (even the advanced H100 has only 80GB).
- Solution (3D Parallelism):
- Data Parallelism (DP): The most fundamental form of parallelism. A complete copy of the model is retained on each GPU, but the data is split into multiple batches, with each GPU processing one batch. Gradients are synchronized via an All-Reduce operation after computation. This approach cannot solve the problem of insufficient single-card VRAM.
- Pipeline Parallelism (PP): Vertically partitions the model's layers so that different GPUs are responsible for different portions of the model (e.g., GPU-1 handles layers 1–16, GPU-2 handles layers 17–32). This effectively reduces per-card VRAM usage but introduces "pipeline bubbles," where some GPUs remain idle while waiting for upstream or downstream data.
- Tensor Parallelism (TP): Horizontally splits individual large operators within the model (such as large weight matrices) across multiple GPUs for collaborative computation. For example, decomposing a large matrix multiplication across multiple cards. This also reduces per-card VRAM usage, but introduces very high communication overhead.
- ZeRO (Zero Redundancy Optimizer): A memory optimization technology proposed by Microsoft DeepSpeed. Building on Data Parallelism, it partitions optimizer states, gradients, and even model parameters across all GPUs. Each GPU only retains the portion it needs for computation, dramatically reducing single-card memory redundancy. It is now standard for large-scale training.

1. Communication Bottleneck Challenge:

- Problem: All the aforementioned parallel strategies introduce heavy inter-GPU communication. For example, DP needs to synchronize gradients, PP needs to pass activation values, and TP needs to exchange intermediate results during every forward and backward pass. When the number of GPUs is massive, communication time may exceed computation time, becoming the bottleneck of the entire training process.
- Solutions:
- Hardware Level: Use high-speed interconnect technologies, such as intra-node NVLink and inter-node InfiniBand networks.
- Software Level: Develop efficient communication algorithms (such as Ring All-Reduce) and design scheduling strategies to overlap computation with communication operations, thereby hiding communication latency.

1. Training Instability Challenge:

- Problem: Training such a massive model is numerically very fragile. Due to extremely deep architectures and massive data volume, exploding or vanishing gradients can easily occur during training, causing the loss to suddenly spike to NaN (Not a Number), ruining hours or even days of training progress.
- Solutions:
- Numerical Precision: BF16 (BFloat16) mixed-precision training is widely adopted. Compared to FP16, BF16 has a dynamic range as wide as FP32, effectively preventing gradient underflow while maintaining stability. Meanwhile, critical components (such as the optimizer's master weights) are still kept in FP32 to ensure precision.
- Stable Model Architectures: Adopt more stable architectural designs, such as Pre-LayerNorm (performing layer normalization before Self-Attention and FFN) and smoother activation functions like GeLU/SwiGLU.
- Gradient Clipping: Set a threshold for the gradient norm. If the calculated gradient exceeds this threshold, it is scaled down within the limit. This is the most direct and effective method to prevent gradient explosion.
- Learning Rate Scheduling & Warmup: Use carefully designed learning rate schedules, such as a "warmup" phase where a smaller learning rate is used initially and gradually increased, helping the model stabilize in early training stages.

</details>

**5. Can the Input Sentence of a LLaMA Model Theoretically Be Infinitely Long? Why?**

<details>
<summary>💡 Show Answer</summary>

**Theoretically, it cannot be infinitely long**, mainly restricted by the following limitations:

1. **Limited Extrapolation Capability of Positional Encoding**: When the sequence length during inference far exceeds what was seen during training, performance degrades on positional encoding combinations outside the training distribution, unless coupled with length extrapolation techniques.
2. **Self-Attention Computational/Memory Complexity of $O(n^2)$**: As sequence length $n$ increases, the computation and memory footprint of the attention matrix grow quadratically, and the KV cache grows linearly, making actual hardware resources a hard constraint.
3. **Fixed Context Window During Training**: The model has only seen sequences up to a specific maximum length during the training phase, making positional combinations beyond that length "out-of-distribution."

The length that can actually be processed is jointly constrained by three aspects: **architecture (positional encoding extrapolability), hardware resources (VRAM/computation), and the context window set during training**.

</details>

**6. How Can Large Language Models Be Made to Process Longer Texts?**

<details>
<summary>💡 Show Answer</summary>

1. **Improve Positional Encoding/Extrapolation Methods**: Position Interpolation (PI), NTK-aware scaling, YaRN, ALiBi, etc.
2. **Continue Training/Fine-Tuning for Long Context**: Perform continued pre-training or fine-tuning on longer sequence data.
3. **Optimize Attention Computation**: FlashAttention reduces memory overhead; sparse attention/sliding window attention reduces complexity from $O(n^2)$ to nearly $O(n)$.
4. **KV Cache Optimization**: Use architectures like MQA, GQA, and MLA to reduce KV cache memory consumption.
5. **Chunked Processing + Retrieval/Summarization**: Chunk ultra-long text and combine with RAG or rolling summarization.
6. **Architectural Innovations**: Architectures with theoretically linear complexity, such as State Space Models (Mamba) and Linear Attention.

</details>

**7. What Are the Common Decoding Strategies During LLM Inference?**

<details>
<summary>💡 Show Answer</summary>

During the inference (or decoding) phase of an LLM, the model generates a probability distribution over tokens, and the decoding strategy determines how the next token is selected from this distribution. Common strategies can be categorized into deterministic and stochastic methods.

**1. Greedy Search**

- Principle: At each time step, always select the token with the highest probability from the current probability distribution as the output.
- Advantages:
- Fast: Minimal computational overhead and simplest implementation.
- Disadvantages:
- Local Optimum: The "greedy" selection at each step may lead to a sequence that is not globally optimal. A high-probability word might be followed by a series of low-probability words, resulting in a lower overall sequence probability.
- Lack of Diversity: Output is completely deterministic; for the same input, the generated result is identical every time, and the content tends to be rigid and repetitive.

**2. Beam Search**

- Principle: An improvement over Greedy Search. At each time step, it retains $k$ ($k$ is called the "beam width" or "beam size") most likely candidate sequences. In the next step, starting from these $k$ candidates, it generates all possible next tokens and then selects the $k$ sequences with the highest cumulative probability from all expanded new sequences. Finally, it picks the best sequence from the final $k$ complete sequences.
- Advantages:
- Higher Quality: By exploring a wider search space, it typically finds sequences with higher probability and better quality than Greedy Search.
- Disadvantages:
- High Computational Cost: Requires maintaining $k$ candidate sequences, resulting in $k$ times the computational and memory overhead of Greedy Search.
- Still Favors Safe and High-Frequency Output: The optimization target is global probability, which still makes it tend to generate common and safe sentences, potentially lacking creativity and susceptible to repetition in long-text generation.

**3. Top-K Sampling**

- Principle: A stochastic sampling strategy. At each time step, instead of selecting the optimal token, it performs the following:

1. Filter out the $K$ tokens with the highest probabilities from the vocabulary's probability distribution.
2. Normalize the probabilities of these $K$ tokens (so their sum equals 1).
3. Randomly sample from these $K$ tokens according to the new probability distribution.

- Advantages:
- Increases Diversity: Introduces randomness, making the generated content richer, more interesting, and unpredictable.
- Avoids Low-Probability Tokens: By restricting sampling to the Top-$K$ tokens, it filters out extremely low-probability tokens that might be disfluent or odd.
- Disadvantages:
- Fixed $K$ Value: $K$ is a fixed hyperparameter. When the probability distribution is sharp (the model is very confident about the next word), a large $K$ may introduce irrelevant tokens; when the distribution is flat (the model is uncertain), a small $K$ may overly restrict choices.

**4. Nucleus Sampling / Top-P Sampling**

- Principle: An improvement over Top-$K$ sampling that uses a dynamic candidate token set.

1. Sort all tokens in descending order of their probability.
2. Starting from the highest-probability token, accumulate their probabilities until the cumulative sum exceeds a predefined threshold $p$ (e.g., $p=0.95$).
3. All tokens included in this cumulative process form the "Nucleus" candidate set.
4. Then, within this dynamically sized candidate set, normalize their original probabilities and perform random sampling.

- Advantages:
- Adaptive Candidate Set: The size of the candidate set dynamically changes based on the context. When the model is very confident, the distribution is sharp, and only one or two tokens may exceed $p$, resulting in a small candidate set for precise generation. When uncertain, the distribution is flat, requiring more tokens to reach $p$, expanding the candidate set to allow more exploration.
- Balances Quality and Diversity: Compared to Top-$K$, it is a more principled and robust method, serving as the default sampling strategy for most LLM applications today.

</details>

**8. Explain the Concept of Temperature in LLM Text Generation**

<details>
<summary>💡 Show Answer</summary>

Temperature is a parameter that controls the randomness of language model outputs by adjusting the probability distribution of tokens.
- **Low Temperature ($<1$)**: Makes output more deterministic and focused. The model prioritizes high-probability tokens, reducing diversity.
- **High Temperature ($>1$)**: Increases variability by making token probabilities more uniform. Suitable for creative and diverse outputs.

</details>

**9. Briefly Explain the Calculation Logic of the PPL Metric**

<details>
<summary>💡 Show Answer</summary>

**PPL (Perplexity)** is a metric that measures a language model's predictive ability on test data, representing "how perplexed the model is by the sample." A lower value indicates more accurate model predictions.

The formula is as follows:

$$
    PPL = \exp\left(-\frac{1}{N}\sum_{i=1}^N \log P(w_i|w_{j})\right)
$$

Where:
- $N$: Total number of tokens in the test set
- $P(w_i|w_j)$: Where $j<i$, the predicted probability of the $i$-th token by the model

The calculation process is as follows:
1. Compute the log probability of prediction for each token
2. Average across all tokens
3. Take the exponential of the result

An intuitive understanding is shown below:
\begin{table}[htbp]



| **PPL Value** | **Model Performance** |
| --- | --- |
| 1 | Perfect prediction (100% probability) |
| 10 | Correctly guesses 1 out of 10 candidate words on average |
| 100 | Correctly guesses 1 out of 100 candidate words on average |


    \caption{Correspondence between PPL values and model performance}

\end{table}

Typical application scenarios include:
- **Language Model Pre-training**: Monitoring the PPL decay curve during training
- **Model Comparison**: Comparing the performance of models with different architectures/scales
- **Domain Adaptation Evaluation**: Testing changes in model PPL within specific domains

**Advantages**:
- Computationally simple and differentiable
- Highly correlated with human perception of language fluency
- Suitable for evaluating foundational capabilities in generative tasks

**Disadvantages**:
- Cannot directly reflect the factualness or logical coherence of generated content
- Exhibits high variance when evaluating short texts
- May be influenced by the tokenization method

GPT-3 PPL on different benchmark test sets:
\begin{table}[htbp]



| **Test Set** | **PPL** |
| --- | --- |
| WikiText-103 | 18.4 |
| Penn Treebank | 35.6 |


    \caption{GPT-3 PPL on different test sets}

\end{table}

**Key Takeaways**:
1. PPL is essentially the **exponential of cross-entropy**, reflecting the uncertainty of model predictions
2. In autoregressive models, it is usually calculated using a **sliding window** approach
3. Modern large language models (175B+ parameters) typically have a PPL of $<20$ on standard test sets
4. Relationship with Loss: $PPL = e^{Loss}$, but Loss focuses more on the training process while PPL emphasizes inference quality

</details>

**10. What Are the Commonly Used Parallel Strategies in LLM Training?**

<details>
<summary>💡 Show Answer</summary>

![3D Parallelism (DP, TP, PP) and ZeRO Memory Sharding Architecture in Distributed Training](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig21_3d_parallelism_zero.png)



\begin{table}[h]



| **Parallel Method** | **Principle** | **Applicable Scenario** |
| --- | --- | --- |
| **Data Parallelism (DP)** | Each GPU holds the complete model; data is partitioned for independent forward/backward passes, followed by gradient aggregation and synchronization | The model can fit into a single GPU's VRAM |
| **Model Parallelism (Tensor Parallelism, TP)** | Splits the model's weight matrices row-wise/column-wise across GPUs; each GPU holds only a portion of the weights | Single-layer parameters are too large for a single GPU |
| **Pipeline Parallelism (PP)** | Assigns different layers of the model to different GPUs, forming a pipeline | The model has many layers and is too deep vertically |
| **Sequence Parallelism (SP)** | Partitions the sequence length dimension across multiple GPUs | Training on ultra-long sequences |
| **ZeRO (Zero Redundancy Optimizer)** | Shards and stores optimizer states, gradients, and parameters across different GPUs | Running out of VRAM during Data Parallelism |


\end{table}
In practice, training typically uses combinations of these, such as DP + TP + PP (3D Parallelism).

</details>

**11. What Is Mixed-Precision Training (FP16/BF16)? What Are Its Advantages and Disadvantages?**

<details>
<summary>💡 Show Answer</summary>

**Principle**: Performs most operations in the model's forward/backward passes using half-precision (FP16 or BF16), while maintaining a set of FP32 master weights in the optimizer for parameter updates, balancing speed and precision.
**Advantages**:
- Memory footprint is reduced by approximately half
- Faster half-precision computation on GPUs (accelerated by Tensor Cores)
- Reduced communication volume, making distributed training more efficient
**Disadvantages**:
- FP16 has a narrow dynamic range and is prone to gradient underflow/overflow, requiring Loss Scaling
- BF16 has a wide dynamic range (same as FP32) and is more stable, but with slightly lower precision; modern large models tend to prefer BF16

</details>

**12. What Are Gradient Accumulation and Gradient Checkpointing, and What Problems Do They Solve?**

<details>
<summary>💡 Show Answer</summary>

**Gradient Accumulation**:
- Problem: Inability to use large batch sizes when VRAM is insufficient
- Method: Accumulate gradients over $N$ small batches before performing a single parameter update, equivalent to a larger effective batch size
- Cost: Increases training time without increasing VRAM usage
**Gradient Checkpointing / Activation Recomputation**:
- Problem: Intermediate activation values from forward passes must be stored for backward passes, consuming significant VRAM when the model has many layers
- Method: Save activation values only at key checkpoint nodes, and recompute the rest during backward propagation
- Cost: Increases computation by about 30%--40%, but significantly reduces VRAM consumption

</details>

**13. Why Does Memory Usage Persist and Grow During LLM Inference?**

<details>
<summary>💡 Show Answer</summary>

Inference memory consists of the following components:
- **Model Weights**: Fixed overhead, not released once loaded
- **KV Cache**: Grows linearly with the generated sequence length and is the primary cause of memory growth during inference. For every new token generated, Key and Value vectors across all layers are appended to the Cache
- **Activations**: Intermediate results of forward passes for the current token, produced at each step but relatively small
- **Batching**: Larger batch sizes increase both KV Cache and activation memory usage
Therefore, the persistent growth of memory during long-sequence inference is primarily caused by KV Cache expansion.

</details>

**14. How Does LLM Inference Speed Differ Between GPU and CPU?**

<details>
<summary>💡 Show Answer</summary>

- **GPU Inference**: GPUs possess thousands of parallel compute units and excel at large-scale matrix multiplications (Attention, FFN), yielding fast inference speeds. A 7B model can reach 50--100 tokens/s on an A100
- **CPU Inference**: CPUs have fewer cores and lack advantages in parallel matrix computation, resulting in inference speeds that are dozens of times slower. A quantized 7B model reaches roughly 5--20 tokens/s on modern CPUs
- **Applicable Scenarios**: GPUs are suitable for high-concurrency online inference; CPU inference combined with quantization (e.g., llama.cpp) is suitable for local lightweight deployment as a fallback when GPU resources are unavailable

</details>

**15. What is Model Quantization? What are Common Quantization Methods?**

<details>
<summary>💡 Show Answer</summary>

![Model Quantization Mapping from FP16/BF16 to INT8/INT4 Bins with Outlier Clipping](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig22_llm_quantization_int8_int4.png)



**Model Quantization**: Converts model weights or activation values from high-precision floating-point numbers (FP32/FP16) to low-bit integers (INT8, INT4, etc.) to reduce VRAM usage and accelerate inference.
**Common Methods**:
\begin{table}[h]



| **Method** | **Quantization Target** | **Characteristics** |
| --- | --- | --- |
| **GPTQ** | Weights (INT4) | Layer-by-layer quantization, small accuracy loss, suitable for large models |
| **AWQ** | Weights (INT4) | Protects important weights by observing activation magnitudes, superior accuracy compared to GPTQ |
| **SmoothQuant** | Weights + Activations (INT8) | Transfers the quantization difficulty of activation values to weights |
| **BitsAndBytes** | Weights (INT8/NF4) | Deeply integrated with Transformers, easy to use |
| **GGUF/llama.cpp** | Weights (2--8 bit) | Specifically designed for CPU inference, multiple precision options available |


\end{table}

</details>

**16. What Memory-Saving Inference Optimization Methods Exist for Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

- **KV Cache Optimization**: MQA and GQA reduce the number of KV heads; PagedAttention (vLLM) dynamically manages KV Cache VRAM
- **Model Quantization**: INT8/INT4 quantization, reducing VRAM usage by 50%--75%
- **FlashAttention**: Reduces HBM read/write operations during Attention computation
- **Continuous Batching**: Dynamically schedules requests of varying lengths to improve GPU utilization
- **Speculative Decoding**: Uses a draft small model to predict multiple tokens and a large model to verify them in parallel, reducing sequential generation steps
- **Model Parallel Inference**: Distributes the model across multiple GPUs to bypass single-GPU VRAM limitations

</details>

**17. How to Make Large Language Model Outputs More Compliant and Controllable?**

<details>
<summary>💡 Show Answer</summary>

- **Prompt Constraints**: Explicitly define output formats, scope of content, and forbidden behaviors within the system prompt
- **Output Filtering**: Integrate content moderation modules after model generation to filter out non-compliant content
- **RLHF / DPO Alignment**: Guide the model during the training stage to reject harmful requests
- **Format Control**: Enforce specific output formats using structured output (such as JSON Schema constraints) or logits constraints
- **Temperature and Sampling Parameters**: Lower the temperature to make outputs more deterministic; use Top-P to control vocabulary diversity
- **RAG Factual Constraints**: Combine with retrieval results to constrain the model to answer based only on provided documents, reducing hallucinations
- **Human-in-the-Loop Backup**: Introduce human moderation workflows in high-risk scenarios

</details>

**18. How to Use Prompt Engineering to Improve LLM Output Quality?**

<details>
<summary>💡 Show Answer</summary>

- **Clear and Explicit Instructions**: Describe specific tasks, output formats, and constraints to avoid ambiguous expressions
- **Few-Shot Examples**: Provide 2--5 input-output examples in the prompt to help the model understand task patterns
- **Chain-of-Thought (CoT)**: Instruct the model to "think step by step" to improve accuracy in complex reasoning
- **Role Prompting**: Assign a persona to the model (e.g., "You are a senior physician") to enhance answer quality in domain-specific fields
- **Output Format Constraints**: Specify outputs as JSON, Markdown tables, or specific custom structures
- **Negative Prompting**: Explicitly state "what not to do" to minimize unwanted outputs
- **Iterative Optimization**: Continually adjust prompts by experimenting and comparing output quality across different prompt designs

</details>

