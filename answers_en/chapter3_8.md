# 3.8 Inference Optimization and Acceleration — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 39 questions

**1. How is Attention Calculated?**

<details>
<summary>💡 Show Answer</summary>

The attention mechanism is the core component of Transformer, and its standard form is **Scaled Dot-Product Attention**.

#### Basic Concepts

The attention mechanism maps the input into three matrices:
- $Q \in \mathbb{R}^{n \times d_k}$: Query matrix, representing "what information the current position wants to query";
- $K \in \mathbb{R}^{m \times d_k}$: Key matrix, representing "the indices of what information each position provides";
- $V \in \mathbb{R}^{m \times d_v}$: Value matrix, representing "the content actually carried by each position".

where $n$ is the Query sequence length, $m$ is the Key/Value sequence length, $d_k$ is the dimension of Key, and $d_v$ is the dimension of Value.

#### Calculation Formula


$$
	Attention(Q, K, V) = softmax\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$


#### Step-by-Step Derivation

1. **Calculate similarity scores**: Measure the relevance between Query and each Key via dot product:

    $$
    S = QK^\top \in \mathbb{R}^{n \times m}
    $$

    where $S_{ij}$ represents the similarity between the $i$-th Query and the $j$-th Key.

2. **Scaling**: Divide the score by $\sqrt{d_k}$ to obtain the scaled score $\hat{S} = S / \sqrt{d_k}$.

    Necessity of scaling: When $d_k$ is large, the variance of the dot product is approximately $d_k$ (assuming the components of $Q$ and $K$ are independent and identically distributed with mean 0 and variance 1), i.e.:

    $$
    Var(Q_i \cdot K_j) = d_k
    $$

    Unscaled large values push the softmax into regions with extremely small gradients (saturation region), leading to unstable training. Dividing by $\sqrt{d_k}$ normalizes the variance to 1, alleviating this issue.

3. **Softmax Normalization**: Perform softmax on each row to convert similarities into probability distributions (attention weights):

    $$
    A_{ij} = \frac{\exp(\hat{S}_{ij})}{\sum_{k=1}^{m} \exp(\hat{S}_{ik})}, \quad A \in \mathbb{R}^{n \times m}
    $$

    $A_{ij}$ represents the attention weight of the $i$-th Query on the $j$-th Value, satisfying $\sum_j A_{ij} = 1$.

4. **Weighted Summation**: Compute the weighted sum of the Value matrix using attention weights to obtain the final output:

    $$
    O = AV \in \mathbb{R}^{n \times d_v}
    $$

    The $i$-th row of the output, $O_i = \sum_j A_{ij} V_j$, is a weighted combination of the individual Value vectors.

#### Multi-Head Attention

In practice, multi-head attention is usually used, projecting $Q, K, V$ into $h$ subspaces respectively, computing attention in parallel, and then concatenating:

$$
	MultiHead(Q, K, V) = Concat(head_1, \ldots, head_h) W^O
$$


$$
	head_i = Attention(QW_i^Q,\; KW_i^K,\; VW_i^V)
$$

where $W_i^Q \in \mathbb{R}^{d_{model} \times d_k}$, $W_i^K \in \mathbb{R}^{d_{model} \times d_k}$, $W_i^V \in \mathbb{R}^{d_{model} \times d_v}$, and $W^O \in \mathbb{R}^{hd_v \times d_{model}}$. The multi-head mechanism allows the model to jointly attend to information from different representation subspaces simultaneously.

#### Computational Complexity

\begin{table}[h]



| **Operation** | **Time Complexity** | **Space Complexity** |
| --- | --- | --- |
| $QK^\top$ Matrix Mult | $O(n^2 d_k)$ | $O(n^2)$ |
| Softmax | $O(n^2)$ | $O(n^2)$ |
| $AV$ Matrix Mult | $O(n^2 d_v)$ | $O(nd_v)$ |


	\caption{Computational and space complexity of Scaled Dot-Product Attention ($n$ is sequence length)}
\end{table}

The storage overhead of the attention matrix $A \in \mathbb{R}^{n \times n}$ grows **quadratically** with sequence length, which is the root cause of the memory bottleneck in long-sequence scenarios and the starting point for optimization works like FlashAttention.

</details>

**2. What Problems Does Naive Softmax Have in Engineering?**

<details>
<summary>💡 Show Answer</summary>

Naive Softmax has two core issues in engineering implementation: **numerical overflow** and **computational efficiency**.

#### Numerical Overflow

Standard Softmax is defined as:

$$
	softmax(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{n} e^{x_j}}
$$


When the input $x_i$ is large, $e^{x_i}$ exceeds the representable range of floating-point numbers, producing `inf` (infinity), causing both numerator and denominator to become `inf`, resulting in `NaN`.

Taking FP16, a commonly used mixed-precision training format, as an example:
- The maximum representable value of FP16 is approximately $65504$;
- When $x > \ln(65504) \approx 11.09$, $e^x > 65504$, causing an overflow;
- The attention logits $QK^\top / \sqrt{d_k}$ before normalization have unconstrained value ranges, making it very easy to trigger this issue.

#### Numerical Underflow

When the input $x_i$ is extremely small (such as a large negative number), $e^{x_i} \approx 0$. If all inputs are large negative numbers, the denominator approaches zero, leading to division-by-zero errors or loss of precision.

#### Memory Bandwidth Bottleneck Brought by Three Passes

Naive Softmax requires **three passes** over the input data:
1. **First pass**: Iterate over all inputs to find the maximum value $m = \max_i x_i$ (for numerical stability);
2. **Second pass**: Iterate over all inputs to compute $\sum_j e^{x_j - m}$ (denominator);
3. **Third pass**: Iterate over all inputs to compute each $e^{x_i - m} / \sum_j e^{x_j - m}$ (normalization).

Each pass involves reading full data from GPU HBM (High Bandwidth Memory), making memory bandwidth a bottleneck, which is particularly prominent in long-sequence scenarios.

#### VRAM Issue of Intermediate Matrices When Combined with Attention Calculation

In standard attention, Softmax must first write the complete attention score matrix $S \in \mathbb{R}^{n \times n}$ into HBM and then read it back for normalization, generating $O(n^2)$ extra VRAM footprint, which is almost unbearable for long sequences (e.g., $n = 100K$).

</details>

**3. How to Solve the Problems of Naive Softmax?**

<details>
<summary>💡 Show Answer</summary>

To address the numerical overflow problem, the standard solution is **numerically stable Softmax by subtracting the maximum value**; to address efficiency issues, the strategy of **online streaming computation** is adopted.

#### Numerically Stable Softmax

Utilizing the shift-invariance property of Softmax:

$$
	softmax(x_i) = \frac{e^{x_i - m}}{\sum_j e^{x_j - m}}, \quad m = \max_k x_k
$$


After subtracting the maximum value $m$, the maximum value of the exponent is $e^0 = 1$, completely avoiding overflow. At the same time, since $x_j - m \leq 0$, all exponential values lie within $(0, 1]$, and the denominator is at least 1, so division-by-zero problems caused by underflow will not occur.

#### Complete Implementation of Safe Softmax

\begin{lstlisting}[style=promptstyle]
	def safe_softmax(x):
	m = x.max(dim=-1, keepdim=True).values   # Compute maximum value
	x_shifted = x - m                          # Shift to prevent overflow
	exp_x = torch.exp(x_shifted)              # Exponentiate
	return exp_x / exp_x.sum(dim=-1, keepdim=True)  # Normalize
\end{lstlisting}

#### Online Softmax (Streaming Two-Pass Computation)

Furthermore, the three passes can be compressed into two passes, synchronously updating the maximum value and the sum during the second pass. Specifically, two statistics $m$ (current maximum value) and $d$ (normalization denominator) are maintained and updated recursively as follows:


$$
	m_j = \max(m_{j-1},\; x_j)
$$


$$
	d_j = d_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j}
$$


After all elements are traversed, $m_n = \max_k x_k$ and $d_n = \sum_k e^{x_k - m_n}$, allowing normalization to be completed in one step. This idea serves as the mathematical foundation for FlashAttention to achieve one-pass attention computation.

</details>

**4. How to Accelerate the Softmax Computation Process?**

<details>
<summary>💡 Show Answer</summary>

The core idea of accelerating Softmax is to reduce memory access to global memory (HBM), merge multiple passes, and utilize **streaming online computation** to complete the calculation of all statistics in a single or two passes.

#### Formal Derivation of Online Softmax

Given an input sequence $\mathbf{x} = (x_1, x_2, \ldots, x_n)$, define the statistics up to the first $j$ steps:

$$
	m_j &= \max(x_1, \ldots, x_j) \\
	d_j &= \sum_{i=1}^{j} e^{x_i - m_j}
$$


Recurrence relation:

$$
	m_j &= \max(m_{j-1},\; x_j) \\
	d_j &= d_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j}
$$


After traversing all elements, the Softmax value of the $i$-th token is:

$$
	p_i = \frac{e^{x_i - m_n}}{d_n}
$$


#### Joint Streaming Computation with the Value Matrix

Furthermore, Softmax and the subsequent $AV$ multiplication can be **fused** into a single pass. Maintain the accumulated statistic $\mathbf{o}_j$ (weighted accumulation of Value):

$$
	\mathbf{o}_j = \mathbf{o}_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j} \cdot V_j
$$


The final output is $\mathbf{o}_n / d_n$, completing all calculations of Softmax and weighted summation within a **single data read**, compressing three HBM accesses into one. This is precisely the core mathematical foundation of FlashAttention.

#### Hardware-Level Acceleration

- **SRAM Reuse**: Load data in tiles into GPU SRAM (on-chip cache) to perform streaming computation within SRAM, avoiding writing intermediate results back to HBM repeatedly;
- **Warp-Level Reduction**: Utilize GPU intra-warp reduction instructions (such as `__shfl_down_sync`) to efficiently perform maximum and accumulation operations across threads;
- **Vectorized Memory Access**: Use 128-bit vectorized memory instructions (`float4`) to improve memory bandwidth utilization.

</details>

**5. What is the Basic Idea of FlashAttention?**

<details>
<summary>💡 Show Answer</summary>

![FlashAttention tiling strategy reducing GPU HBM memory read/write I/O bottleneck.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig45_flashattention_tiling.png)




FlashAttention is an **IO-aware** exact attention algorithm whose core goal is not to reduce FLOPs, but to increase actual execution speed and reduce VRAM footprint by minimizing HBM access times.

#### Motivation: Memory Bandwidth Is the Bottleneck, Not Compute

The bottleneck of standard attention lies in memory access rather than computation:
- Attention matrices $S, A \in \mathbb{R}^{n \times n}$ need to be written to/read from HBM, incurring $O(n^2)$ IO overhead;
- GPU HBM bandwidth (approx. 2 TB/s) is far lower than SRAM bandwidth (approx. 20 TB/s), making HBM read/write the primary source of latency;
- For $n = 2048$ and $d = 64$, the attention matrix occupies about 512 MB, far exceeding the SRAM capacity (approx. 20 MB).

#### Core Idea: Tiling + Online Softmax

FlashAttention splits the $Q, K, V$ matrices into small tiles of size $B_r \times d$ and $B_c \times d$, enabling each tile to fit completely into SRAM. For each $Q$ block, it iterates through all $K, V$ blocks, utilizes Online Softmax to maintain numerically stable statistics, and completes attention computation inside SRAM **without writing intermediate attention matrices back to HBM**:


$$
	HBM IO (Standard Attention) = O(nd + n^2)
$$


$$
	HBM IO (FlashAttention) = O(n^2 d^2 / M)
$$


where $M$ is the SRAM size. When $M \gg d^2$, the IO complexity of FlashAttention is significantly smaller than that of standard attention.

#### Algorithm Flow Overview

1. Divide $Q$ into $T_r = \lceil n / B_r \rceil$ blocks, and $K, V$ into $T_c = \lceil n / B_c \rceil$ blocks;
2. The outer loop iterates over $K, V$ blocks, loading them into SRAM;
3. The inner loop iterates over $Q$ blocks, computing local attention scores in SRAM and updating global statistics $(m, d, \mathbf{o})$ using Online Softmax;
4. After all blocks are traversed, write the final output $\mathbf{o}/d$ back to HBM.

#### Recomputation Strategy in Backward Pass

Standard backward pass requires storing the attention matrix $A$ computed during forward pass ($O(n^2)$ VRAM). FlashAttention **recomputes** the attention matrix during backward pass (requiring only storing $m, d$ statistics, taking $O(n)$ VRAM), trading additional computation for significant VRAM savings.

#### Performance Gains

- **Speed**: Compared to standard PyTorch implementation, FlashAttention achieves 2--4$\times$ speedup on A100 GPUs;
- **VRAM**: Attention layer VRAM footprint is reduced from $O(n^2)$ to $O(n)$, supporting training with longer sequences;
- **Precision**: Mathematically exact and equivalent to standard attention, with zero approximation error.

</details>

**6. Why Can FlashAttention Achieve One-Pass Computation?**

<details>
<summary>💡 Show Answer</summary>

The fundamental reason FlashAttention can achieve one-pass computation lies in the fact that: **the final output of Self-Attention can be expressed as a recursive accumulation of several local statistics**, which can be corrected online as new data is read in at each step, without needing prior knowledge of global information.

#### Mathematical Derivation

Let Query $q \in \mathbb{R}^{d}$, Keys $K = [k_1, \ldots, k_n]^\top$, and Values $V = [v_1, \ldots, v_n]^\top$. The attention output is:

$$
	o = \sum_{i=1}^{n} \frac{e^{q \cdot k_i / \sqrt{d}}}{\sum_j e^{q \cdot k_j / \sqrt{d}}} v_i
$$


Define $s_i = q \cdot k_i / \sqrt{d}$, then:

$$
	o = \frac{\sum_{i=1}^n e^{s_i} v_i}{\sum_{i=1}^n e^{s_i}}
$$


**Key Observation**: Both the numerator $\sum e^{s_i} v_i$ and the denominator $\sum e^{s_i}$ are **decomposable summations**, which support streaming computation.

Maintain three statistics:

$$
	m_j &= \max(s_1, \ldots, s_j) & &(Current maximum, for numerical stability)\\
	d_j &= \sum_{i=1}^j e^{s_i - m_j} & &(Normalization denominator)\\
	\mathbf{o}_j &= \sum_{i=1}^j e^{s_i - m_j} v_i & &(Weighted Value accumulation)
$$


When new data $(s_{j+1}, v_{j+1})$ is read at step $j+1$, the update rules are:

$$
	m_{j+1} &= \max(m_j,\; s_{j+1}) \\
	d_{j+1} &= d_j \cdot e^{m_j - m_{j+1}} + e^{s_{j+1} - m_{j+1}} \\
	\mathbf{o}_{j+1} &= \mathbf{o}_j \cdot e^{m_j - m_{j+1}} + e^{s_{j+1} - m_{j+1}} \cdot v_{j+1}
$$


The correction factor $e^{m_j - m_{j+1}}$ rescales the old statistics from the old maximum baseline to the new maximum baseline, ensuring numerical consistency.

#### Final Output

After iterating over all $n$ Key-Value pairs, the output is:

$$
	o = \frac{\mathbf{o}_n}{d_n}
$$


The entire process requires **only a single pass over the data**, with only $O(1)$ auxiliary storage per step ($m, d, \mathbf{o}$), without needing to instantiate the full attention matrix $A \in \mathbb{R}^{n \times n}$ in memory.

#### Extension to Block-Wise Parallelism

In actual GPU implementations, $K$ and $V$ are split into multiple blocks. Each block completes its local computation within SRAM, and statistics are merged across blocks using the recursive formula above. This forms the exact mathematical foundation of FlashAttention's Tiling strategy.

</details>

**7. Explain the Tiling Strategy in FlashAttention**

<details>
<summary>💡 Show Answer</summary>

Tiling (block-wise division) is the core engineering strategy used by FlashAttention to adapt large attention matrix computations to limited SRAM capacity.

#### Motivation

The storage hierarchy of GPUs is as follows:
- **HBM** (High Bandwidth Memory): Large capacity (40--80 GB), high bandwidth (~2 TB/s), high latency;
- **SRAM** (L1 Cache / Shared Memory): Small capacity (~192 KB/SM), high bandwidth (~19 TB/s), low latency.

Standard attention requires storing the $n \times n$ attention matrix in HBM, whereas FlashAttention aims to **confine all intermediate computations within SRAM**, thereby reducing HBM access frequency from $O(n^2)$ to $O(n)$.

#### Choice of Block Size

Let the SRAM capacity be $M$. The selection of Key/Value block size $B_c$ and Query block size $B_r$ must satisfy:

$$
	B_c \cdot d + B_r \cdot d + B_r \cdot B_c \leq M
$$


That is, one $Q$ block, one $K$ block, one $V$ block, and the corresponding attention score block can fit into SRAM simultaneously. Typically, they are chosen as:

$$
	B_c = B_r = \left\lfloor \frac{M}{4d} \right\rfloor
$$


#### Algorithm Details

Outer loop (along the $K, V$ dimension, sequential traversal):
1. Load the $j$-th block $K_j \in \mathbb{R}^{B_c \times d}$ and $V_j \in \mathbb{R}^{B_c \times d}$ from HBM into SRAM;
2. Inner loop (along the $Q$ dimension, parallelizable):
    1. Load the $i$-th block $Q_i \in \mathbb{R}^{B_r \times d}$ and its current statistics $m_i, d_i, \mathbf{o}_i$ from HBM;
    2. Compute local attention scores $S_{ij} = Q_i K_j^\top / \sqrt{d} \in \mathbb{R}^{B_r \times B_c}$ within SRAM;
    3. Update $m_i, d_i, \mathbf{o}_i$ using Online Softmax;
    4. Write the updated statistics back to HBM (writing only $O(B_r \cdot d)$ size, far smaller than $O(n \cdot n)$).

#### IO Analysis of Tiling

- Each $K, V$ block is read $T_r = n / B_r$ times, reading a total of $T_c \times T_r \times 2 B_c d$ elements;
- The total HBM memory access (read/write) volume is $O(n^2 d / M)$, which offers significant advantages over standard attention's $O(nd + n^2)$ when $M \gg d$;
- **The attention matrix is never written to HBM**, completely eliminating the $O(n^2)$ VRAM footprint bottleneck.

#### Handling Causal Masking

For decoder self-attention (Causal Self-Attention), Tiling simply sets the upper triangular part within the block to $-\infty$ when computing $S_{ij}$ (i.e., making the attention weights at corresponding positions zero), without needing to modify the overall Tiling framework, making the implementation concise and efficient.

</details>

**8. How are MQA/GQA Handled in FlashAttention?**

<details>
<summary>💡 Show Answer</summary>

![Structural comparison of MHA, GQA, and MQA Key-Value head sharing strategies.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig48_mha_gqa_mqa_comparison.png)




MQA (Multi-Query Attention) and GQA (Grouped-Query Attention) are mainstream techniques to reduce KV cache size and lower inference memory bandwidth consumption. FlashAttention provides native support for them via an **indexing mechanism**.

#### Review of MQA and GQA

- **MHA** (Multi-Head Attention): Each Query Head corresponds to independent K and V Heads, totaling $h$ groups;
- **MQA** (Multi-Query Attention): All Query Heads share the same single group of K and V, reducing KV parameters to $1/h$;
- **GQA** (Grouped-Query Attention): $h$ Query Heads are divided into $g$ groups, with each group sharing one KV Head ($1 \leq g \leq h$), serving as a tradeoff between MHA and MQA.

#### Index Processing in FlashAttention

In FlashAttention's CUDA Kernel implementation, MQA/GQA is handled by passing **KV Head indices** into the kernel, without copying KV data:

\begin{lstlisting}[style=promptstyle]
	# KV Head index calculation for GQA (pseudocode)
	num_q_heads = 32
	num_kv_heads = 8         # GQA: Every 4 Q Heads share 1 KV Head
	group_size = num_q_heads // num_kv_heads  # = 4

	# For the q_head_idx-th Query Head, find the corresponding KV Head
	kv_head_idx = q_head_idx // group_size

	# Inside the Kernel, compute memory offset addresses for K and V based on kv_head_idx
	k_ptr = K_base + kv_head_idx * kv_head_stride
	v_ptr = V_base + kv_head_idx * kv_head_stride
\end{lstlisting}

#### Implementation Details

1. **Launch Configuration**: The Kernel is launched with grid size based on the number of Query Heads $h_q$, where each thread block handles the attention computation for one Q Head;

2. **KV Head Mapping**: Inside the Kernel, integer division `kv_head_idx = q_head_idx // group_size` determines the KV Head corresponding to the current Q Head;

3. **Memory Address Calculation**: The memory offsets of $K$ and $V$ tensors are computed directly based on the KV Head index, **without replicating or broadcasting KV data in memory**, saving memory bandwidth and VRAM;

4. **Tiling Compatibility**: Block-wise loading of KV is performed using the actual data corresponding to the KV Head. Multiple Q Heads can **concurrently read the same KV block** (hit via L2 Cache), further enhancing cache efficiency.

#### Performance Advantages

- MQA/GQA reduces the KV Cache size from $O(n \cdot h \cdot d)$ to $O(n \cdot g \cdot d)$, significantly decreasing memory bandwidth consumption during inference;
- FlashAttention's indexing mechanism avoids redundant storage and copying of KV data, saving $(h/g - 1) \times$ of KV memory compared to the naive "broadcast first, then compute" approach;
- GQA is the standard configuration in mainstream models such as LLaMA 3, Mistral, and Gemma; FlashAttention's efficient support for GQA is a crucial engineering foundation in practical deployments.

</details>

**9. Why Is There Only a KV Cache and No Q Cache?**

<details>
<summary>💡 Show Answer</summary>

The asymmetry between KV Cache and Q Cache stems from the **temporal characteristics** of autoregressive decoding: Query is dynamically changing and different at each step, whereas historical Key and Value vectors can be reused in subsequent steps.

#### Working Mechanism of Autoregressive Decoding

An autoregressive language model decodes token by token as follows:

$$
	x_{t+1} \sim P(\cdot \mid x_1, x_2, \ldots, x_t)
$$


At step $t$, the model needs to compute:

$$
	Attention_t = softmax\!\left(\frac{q_t [K_1, K_2, \ldots, K_t]^\top}{\sqrt{d_k}}\right) [V_1, V_2, \ldots, V_t]^\top
$$


#### Why Is KV Cache Needed?

Without KV Cache, every decoding step would require recomputing $K$ and $V$ for **all historical tokens**, resulting in a computational complexity of $O(t \cdot d^2)$ (QKV projections) per step, and a total decoding complexity of $O(n^2 d^2)$, which grows quadratically with sequence length.

Notice that at step $t$, the $K$ and $V$ corresponding to historical tokens $x_1, \ldots, x_{t-1}$ are **identical** to those in the previous step (because their input tokens have not changed), so they can be cached:

$$
	K_i = x_i W^K, \quad V_i = x_i W^V \quad (i < t,  unchanged)
$$


KV Cache reduces the complexity per decoding step from $O(t \cdot d^2)$ to $O(d^2)$ (computing $K, V$ for only the new token), reducing the total decoding complexity from $O(n^2 d^2)$ to $O(n d^2)$.

#### Why Is Q Cache Not Needed?

The Query vector in each decoding step is produced solely by the **currently newly generated token**:

$$
	q_t = x_t W^Q
$$


In autoregressive decoding, there is only one new token per step, so $q_t$ is a vector of shape $1 \times d_k$, with an **extremely low computational cost** ($O(d^2)$), making caching practically useless.

Furthermore, $q_t$ is only used at step $t$. In the next step (step $t+1$), the new Query $q_{t+1}$ is generated from $x_{t+1}$ and is completely independent of $q_t$. Caching $q_t$ provides no reusability for subsequent steps.

#### Comparative Summary

\begin{table}[h]



| **Property** | **KV (Key/Value)** | **Q (Query)** |
| --- | --- | --- |
| Source | Historical tokens | Currently newly generated token |
| Cross-step reusability | Fully reusable (unchanged) | Not reusable (different per step) |
| Cache benefit | High (avoids $O(n)$ recomputation) | None (only $O(1)$ computation per step) |
| VRAM overhead | $O(n \cdot h \cdot d)$ | $O(h \cdot d)$ (current step) |


	\caption{Comparative analysis between KV Cache and Q Cache}
\end{table}

#### Engineering Tradeoffs of KV Cache

Although KV Cache significantly accelerates decoding, it also incurs substantial VRAM pressure. Taking LLaMA-2 70B as an example, under FP16 precision, the KV Cache size is approximately:

$$
	KV Cache = 2 \times n \times L \times h \times d_k \times 2  bytes
$$

where $n$ is the sequence length, $L$ is the number of layers, $h$ is the number of heads, and $d_k$ is the dimension per head. This is precisely the design motivation behind techniques such as MQA/GQA, quantized KV Cache, and PagedAttention.

</details>

**10. How Long Does It Take to Train a 1B Model on 1T Tokens?**

<details>
<summary>💡 Show Answer</summary>

Estimating training time depends on multiple factors, including model scale, dataset size, hardware compute capacity, and system efficiency. It can be systematically derived using the following formula.

#### Training Time Estimation Formula


$$
	T_{train} = \frac{C_{total}}{N_{GPU} \times F_{GPU} \times \eta}
$$


where:
- $C_{total}$: Total compute (FLOPs);
- $N_{GPU}$: Number of GPUs;
- $F_{GPU}$: Theoretical peak compute per GPU (FLOPs/s);
- $\eta$: GPU utilization (MFU, Model FLOPs Utilization), typically between 0.3--0.5 in engineering practice.

#### Derivation of Total Compute $C_{total$}

For a model with parameter count $N$, processing $D$ tokens requires a total compute of:

$$
	C_{total} = 6 N D
$$


Origin of the factor 6:
- Forward pass: Each parameter involves 1 multiplication + 1 addition, requiring approx. $2N$ FLOPs/token;
- Backward pass: Gradient computation requires approx. twice as much as the forward pass, i.e., $4N$ FLOPs/token;
- Total: $2N + 4N = 6N$ FLOPs/token.

If **activation recomputation** (Gradient Checkpointing) is enabled, the forward pass is executed twice, increasing total FLOPs to $8ND$ to trade extra compute for VRAM savings.

#### Numerical Example (Single-GPU Scenario)

Taking $N = 10^9$ (1B parameters), $D = 10^{12}$ (1T tokens), using an NVIDIA A100 80GB (BF16 peak compute approx. $312 \times 10^{12}$ FLOPs/s), $\eta = 0.3$, and enabling activation recomputation (factor of 8):


$$
	C_{total} &= 8 \times 10^9 \times 10^{12} = 8 \times 10^{21}  FLOPs \\
	T &= \frac{8 \times 10^{21}}{1 \times 312 \times 10^{12} \times 0.3} \\
	&= \frac{8 \times 10^{21}}{9.36 \times 10^{13}} \approx 8.55 \times 10^7  seconds \approx 990  days
$$


#### Multi-GPU Cluster Scenario

Actual training typically employs multi-GPU parallelism. Taking 64 A100 GPUs as an example ($\eta = 0.4$):

$$
	T &= \frac{8 \times 10^{21}}{64 \times 312 \times 10^{12} \times 0.4} \approx \frac{8 \times 10^{21}}{7.99 \times 10^{15}} \approx 10^6  seconds \approx 11.6  days
$$


#### Chinchilla Optimality Verification

According to Chinchilla's law, the compute-optimal number of training tokens for a 1B parameter model is approx. $20N = 20 \times 10^9 = 20B$ tokens, rather than 1T tokens. Training on 1T tokens constitutes severe **over-training** for a 1B model, which is non-optimal from a compute efficiency standpoint; however, if the goal is inference deployment efficiency (making a smaller model stronger during inference), moderate over-training is reasonable (e.g., the strategy used in the LLaMA series).

#### Summary

\begin{table}[h]



| **Scenario** | **GPU Count** | **MFU** | **Estimated Time** |
| --- | --- | --- | --- |
| Single A100 GPU | 1 | 0.30 | $\approx$ 990 days |
| 8x A100 GPUs | 8 | 0.35 | $\approx$ 108 days |
| 64x A100 GPUs | 64 | 0.40 | $\approx$ 12 days |
| 512x A100 GPUs | 512 | 0.45 | $\approx$ 1.3 days |


	\caption{Training time estimates for a 1B model + 1T tokens across different cluster sizes}
\end{table}

</details>

**11. Why Use RMSNorm Instead of LayerNorm in LLMs?**

<details>
<summary>💡 Show Answer</summary>

RMSNorm (Root Mean Square Layer Normalization) is a simplified variant of LayerNorm that has become a standard choice in modern large language models (LLaMA, Mistral, Gemma, Qwen, etc.). Its advantages span three dimensions: computational efficiency, training stability, and theoretical validity.

#### LayerNorm Review

LayerNorm normalizes across the feature dimension for each sample, with the formula:

$$
	LayerNorm(x_i) = \gamma \cdot \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
$$

where:

$$
	\mu = \frac{1}{d} \sum_{i=1}^d x_i, \quad \sigma^2 = \frac{1}{d} \sum_{i=1}^d (x_i - \mu)^2
$$

It involves two-step statistic computations for the **mean** and **variance**, along with a learnable shift parameter $\beta$ (bias).

#### RMSNorm Definition

RMSNorm discards the mean-centering step and scales inputs using only the Root Mean Square (RMS):

$$
	RMSNorm(x_i) = \gamma \cdot \frac{x_i}{RMS(\mathbf{x})}, \quad RMS(\mathbf{x}) = \sqrt{\frac{1}{d} \sum_{i=1}^d x_i^2 + \epsilon}
$$


Compared to LayerNorm, RMSNorm **removes mean calculation** and the **shift parameter** $\beta$, retaining only the scaling parameter $\gamma$.

#### Computational Efficiency Advantages

- LayerNorm requires two passes: the first to compute the mean $\mu$, and the second to compute the variance $\sigma^2$ (which requires knowing $\mu$);
- RMSNorm requires only **a single pass**: directly accumulating $x_i^2$, computing $RMS$, and normalizing;
- Reduces computation cost by approx. $\mathbf{7%--15%}$, accumulating significant savings in deep networks (e.g., 80-layer LLaMA-2 70B);
- Reduces $d$ learnable parameters (the $\beta$ vector), slightly reducing model parameter size.

#### Theoretical Basis: Rescaling Invariance

Research by Zhang & Sennrich (2019) demonstrated that the performance gain of LayerNorm primarily stems from **rescaling invariance** rather than mean-centering. Specifically, for input $\mathbf{x}$ and scaling factor $\alpha > 0$:

$$
	RMSNorm(\alpha \mathbf{x}) = \alpha \cdot RMSNorm(\mathbf{x}) \cdot \frac{1}{\alpha} = RMSNorm(\mathbf{x})
$$

That is, the output is insensitive to scaling of the input, possessing **scale invariance**, which helps stabilize the training of deep networks. Mean-centering contributes relatively little to final performance, and removing it yields no significant difference in experimental results.

#### Training Stability

- **Gradient stability**: RMSNorm features a simpler gradient formulation, which helps mitigate vanishing/exploding gradients in deep networks (e.g., 100+ layers);
- **Implicit learning rate adaptation**: By normalizing activation magnitudes, RMSNorm aligns learning rate requirements across different layers, reducing reliance on fine-grained learning rate tuning;
- **Advantages under Pre-Norm architecture**: Modern LLMs widely adopt Pre-Norm (normalizing prior to residual connections), where RMSNorm performs comparably to LayerNorm while being computationally lighter.

#### Empirical Comparison

\begin{table}[h]



| **Attribute** | **LayerNorm** | **RMSNorm** |
| --- | --- | --- |
| Computation Steps | Mean + Variance | RMS only |
| Learnable Parameters | $\gamma, \beta$ | $\gamma$ only |
| Number of Passes | 2 passes | 1 pass |
| Compute Overhead | Baseline | Approx. 10% reduction |
| Downstream Performance | Comparable | Comparable (Slightly better in deep networks) |
| Representative Models | BERT, GPT-2 | LLaMA, Mistral, Qwen |


	\caption{Comparison between LayerNorm and RMSNorm}
\end{table}

In summary, **while maintaining performance comparable to LayerNorm**, RMSNorm offers lower computational overhead and a simpler implementation, making it an engineering preference for large-scale training scenarios.

</details>

**12. How to Estimate GPU Utilization During LLM Training?**

<details>
<summary>💡 Show Answer</summary>

GPU utilization (GPU Utilization / MFU, Model FLOPs Utilization) is a key metric for measuring training system efficiency, directly reflecting the actual usage efficiency of hardware compute capacity.

1. **GPU Compute Efficiency Evaluation (FLOPs Benchmark Method)**

    The most direct method: measure utilization by calculating the ratio of the model's actually executed FLOPs to the hardware's theoretical peak.


    $$
    MFU = \frac{Measured FLOPs/s}{F_{GPU,peak}}
    $$


    Measured FLOPs/s can be obtained as follows:

    $$
    Measured FLOPs/s = \frac{6ND}{T_{actual} \times N_{GPU}}
    $$

    where $N$ is the parameter count, $D$ is the number of processed tokens, $T_{actual}$ is the actual elapsed time, and $N_{GPU}$ is the number of GPUs.

    In practice, the MFU of an A100 cluster typically ranges between 0.35--0.50; an MFU below 0.3 indicates significant communication, data loading, or I/O bottlenecks.

2. **Throughput Comparison Evaluation Method**

    Indirectly estimate utilization by comparing the actual training throughput (tokens/s) with the nominal throughput for the same configuration reported in the literature:

    $$
    Relative Utilization = \frac{Actual Throughput (tokens/s)}{Literature Nominal Throughput (tokens/s)}
    $$


    The advantage is that it does not require precise FLOPs accounting and is simple to implement; the disadvantage is that it is affected by differences in test conditions across literature, limiting its reference value. In practice, `tokens/s/GPU` can be used as a normalized metric for convenient cross-hardware comparison.

3. **Runtime Performance Profiling Method (Profiling)**

    Use specialized tools to conduct fine-grained performance analysis to locate specific bottlenecks:

    - **PyTorch Profiler**: Records execution time on CPU/GPU, memory usage, and CUDA kernel calls for each operator:
        \begin{lstlisting}[style=promptstyle]
        import torch
        from torch.profiler import profile, ProfilerActivity, schedule

        with profile(
        activities=[ProfilerActivity.CPU, ProfilerActivity.CUDA],
        schedule=schedule(wait=1, warmup=1, active=3),
        on_trace_ready=torch.profiler.tensorboard_trace_handler('./log'),
        record_shapes=True,
        with_stack=True
        ) as prof:
        for step, batch in enumerate(dataloader):
        train_step(batch)
        prof.step()
        \end{lstlisting}

    - **NVIDIA Nsight Systems / Nsight Compute**: Provides GPU kernel-level timeline analysis to identify issues such as computation bubbles, communication waits, and memory bandwidth bottlenecks;

    - **nvidia-smi**: Real-time monitoring of hardware metrics such as GPU utilization (SM occupancy), VRAM usage, and power consumption (Note: the utilization reported by `nvidia-smi` represents the proportion of SM active time, which is not equivalent to MFU);

    - **DCGM (Data Center GPU Manager)**: Suitable for GPU health monitoring and metric collection in large-scale clusters.

</details>

**13. How to Optimize GPU Utilization in Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

Improving GPU utilization requires systematic optimization across multiple dimensions, including data flow, model computation, and communication overhead. Common bottlenecks and their corresponding solutions are as follows.

1. **Reduce Logging Frequency**

    Frequent log printing (e.g., printing loss and gradient norm every step) triggers CPU-GPU synchronization (`torch.cuda.synchronize()`), blocking the GPU execution pipeline. Recommendations:
    - Reduce logging frequency from every step to every 10--100 steps;
    - Use asynchronous logging (`loss.item()` triggers synchronization and can be deferred);
    - Asynchronously write monitoring metrics via TensorBoard to avoid blocking the main training loop.

2. **Use Performance Profiling Tools to Locate Bottlenecks**

    Before optimizing, a Profiler must be used to locate the bottleneck (computation, communication, I/O, or CPU) to avoid blind optimization. Common bottleneck patterns:
    - Low GPU computation time percentage $\rightarrow$ data loading or communication bottleneck;
    - High `all-reduce` ratio $\rightarrow$ insufficient communication bandwidth; consider gradient compression or overlapping communication;
    - Large amount of `cudaMemcpyAsync` $\rightarrow$ excessive Host-Device data transfer.

3. **Improve Data Loading Efficiency**

    Data loading is a common bottleneck, especially in high GPU compute capability scenarios. Optimization measures:
    - **Increase DataLoader Workers**: Set `num_workers` to 1/4--1/2 of the number of CPU cores;
    - **Enable Memory-Mapped Datasets**: Use `.bin` or Arrow formats to avoid repeated disk I/O;
    - **Data Pre-caching and Prefetching**: Load the next batch of data into CPU memory in advance to overlap with GPU computation;
    - **Use Faster Storage Devices**: Replace HDDs with NVMe SSDs, or store datasets on local SSDs instead of Network File Systems (NFS);
    - **`pin_memory=True`**: Enable pinned memory to accelerate CPU $\rightarrow$ GPU data transfer (`cudaMemcpyHostToDevice`).

4. **Optimize Model Architecture and Operators**

    - Replace standard attention with **FlashAttention** to reduce HBM bandwidth consumption;
    - Use **torch.compile** (PyTorch 2.0+) for model graph compilation optimization to reduce kernel launch overhead and memory allocation;
    - Use **Fused Kernels** (e.g., fused LayerNorm, fused Softmax, fused Adam) to reduce the number of individual kernel launches;
    - Set dimensions such as `d_ff` and `d_model` to multiples of 128 to fully utilize Tensor Cores.

5. **Adjust Batch Size**

    - Increasing batch size improves GPU arithmetic intensity and reduces relative communication overhead, but is constrained by GPU memory;
    - Use **Gradient Accumulation** to simulate large batches under limited GPU memory:

        $$
        Effective batch size = micro batch size \times gradient accumulation steps \times N_{GPU}
        $$

    - An excessively large batch size may affect model convergence quality (degrading generalization), requiring adjustment in conjunction with the Linear Scaling Rule for learning rate.

6. **Adjust Learning Rate and Optimizer**

    - Use **Mixed-Precision Training** (BF16/FP16 + FP32 master weights): Halve GPU memory usage without loss of precision, enabling larger batch sizes while Tensor Cores provide higher compute performance at lower precision;
    - Use **8-bit Adam** (e.g., the `bitsandbytes` library): Quantize optimizer states to 8-bit, reducing optimizer memory from $12N$ bytes (FP32 Adam) to approximately $6N$ bytes, freeing up memory to increase batch size;
    - Enable **Learning Rate Warmup + Cosine Annealing** scheduling to stabilize gradient updates in the early stages of large-batch training;
    - For multi-node training, appropriately configure **ZeRO Optimization Levels** (DeepSpeed ZeRO-1/2/3) to balance communication volume and GPU memory savings.

</details>

**14. What Factors Affect GPU Memory Consumption in Large Models?**

<details>
<summary>💡 Show Answer</summary>

GPU memory consumption in large model training consists of multiple components, which can be broadly categorized into **Model States** and **Activations**. Model states represent the primary source of memory occupation.

#### Analysis of GPU Memory Components

Taking mixed-precision training (FP16 + FP32 master weights) with the Adam optimizer as an example, for a model with $N$ parameters:

\begin{table}[h]



| **Component** | **Precision** | **GPU Memory Size** |
| --- | --- | --- |
| Model Parameters (Forward/Backward) | FP16 | $2N$ bytes |
| Gradients | FP16 | $2N$ bytes |
| Master Weights (Optimizer) | FP32 | $4N$ bytes |
| Adam 1st Moment ($m$) | FP32 | $4N$ bytes |
| Adam 2nd Moment ($v$) | FP32 | $4N$ bytes |
| **Total Model States** |  | $\mathbf{16N}$ bytes |
| Activations (batch / sequence length dependent) | FP16/BF16 | Depends on configuration |


	\caption{GPU memory breakdown for mixed-precision training + Adam ($N$ represents parameter count)}
\end{table}

For example, the model state memory for a 70B-parameter model is approximately $16 \times 70 \times 10^9 \approx 1120$~GB, requiring at least 14 GPUs with 80GB A100s.

#### Detailed Analysis of Influencing Factors

1. **Model Parameter Count**: The most fundamental source of memory usage, directly determining the lower bound of model states. The parameter count has a **linear relationship** with memory and is the primary consideration in memory planning.

2. **Batch Size**:
    - Memory for activations (intermediate layer outputs) grows **linearly** with batch size;
    - Model states (parameters, gradients, optimizer states) are independent of batch size;
    - Gradient accumulation can be used to increase the effective batch size without increasing single-step memory usage.

3. **Sequence Length**:
    - Activation memory grows **linearly** with sequence length for the FFN component;
    - Attention matrix memory grows **quadratically** ($O(n^2)$) with sequence length, serving as the main bottleneck in long-sequence training;
    - FlashAttention reduces attention layer memory from $O(n^2)$ to $O(n)$, significantly mitigating this issue.

4. **Model Architecture**:
    - **Number of Layers**: Deeper networks require more activation memory (forward outputs of each layer must be stored for backpropagation);
    - **Hidden Dimension**: Increasing $d_{model}$ results in larger intermediate tensors in attention and FFN layers;
    - **MHA vs. MQA/GQA**: MQA reduces the number of KV heads to 1, while GQA reduces it to $g < h$, reducing KV Cache memory by a factor of $h/g$;
    - **MoE Architecture**: Possesses a large parameter count, but only a few experts are activated per forward pass, resulting in relatively smaller activation memory.

5. **Optimizer Choice**:
    - **SGD**: Requires storing only gradients; model states $= 2N$ (FP16 parameters) $+ 4N$ (FP32 master) $+ 4N$ (gradients) $= 10N$ bytes;
    - **Adam**: Stores additional first and second moments; model states $= 16N$ bytes;
    - **8-bit Adam**: Quantizes optimizer states; model states $\approx 6N$ bytes;
    - **Adafactor**: Does not store full second-moment matrices, storing only row/column factorizations; memory usage approaches SGD.

6. **Mixed-Precision Training**:
    - Training with FP16/BF16 consumes $2N$ bytes each for parameters and gradients during computation;
    - Compared to pure FP32 training ($4N + 4N = 8N$ bytes), it saves about 25% of parameter/gradient memory;
    - BF16 offers a wider dynamic range than FP16, leading to more stable training (recommended for modern GPUs like A100 and H100).

7. **Activation Recomputation (Gradient Checkpointing)**:
    - Without activation recomputation: Activations of all layers must be stored, with memory scaling as $O(L \cdot n \cdot d)$;
    - With activation recomputation: Only checkpointed layer activations are stored (e.g., saved every $k$ layers), reducing memory to $O(\sqrt{L} \cdot n \cdot d)$ at the cost of increasing compute by roughly 1/3.

#### Summary of GPU Memory Optimization Strategies

\begin{table}[h]



| **Optimization Technique** | **Source of Memory Savings** | **Cost / Trade-off** |
| --- | --- | --- |
| Mixed-Precision (BF16) | Parameters/gradients halved | Slight precision drop (acceptable) |
| 8-bit Adam | Optimizer states halved | Slightly increased computation |
| Activation Recomputation | Significant reduction in activations | ~33% FLOPs increase |
| ZeRO-1/2/3 | Model states sharded across GPUs | Increased communication volume |
| GQA/MQA | KV Cache memory reduced | Slight performance degradation |
| FlashAttention | Attention layer $O(n^2) \rightarrow O(n)$ | No precision loss |


	\caption{Comparison of common GPU memory optimization techniques}
\end{table}

</details>

**15. What is Beam Search and How Does It Differ from Greedy Decoding?**

<details>
<summary>💡 Show Answer</summary>

Beam Search and Greedy Decoding are the two most commonly used decoding algorithms in sequence generation, differing fundamentally in their search space exploration strategies and output quality.

#### Greedy Decoding

Greedy decoding selects the token with the **highest conditional probability at the current step**:

$$
	x_t^* = \arg\max_{w} P(w \mid x_1, \ldots, x_{t-1})
$$


- **Pros**: Simple to implement, with a time complexity of $O(n \cdot |V|)$ (where $|V|$ is the vocabulary size), and extremely fast;
- **Cons**: Local optima at each step do not guarantee global optimality, potentially falling into sub-optimal solutions. For example, selecting a token with a slightly lower probability at a certain step might unlock paths with higher probabilities for subsequent steps, but greedy decoding cannot backtrack;
- **Typical Issues**: Generating repetitive sequences (repetition), premature sequence truncation, and semantic incoherence.

#### Beam Search

Beam search maintains $k$ (beam size) **top candidate sequences** at each step, exploring multiple paths simultaneously:

1. Initialization: Start with the $k$ initial tokens with the highest scores to obtain $k$ candidate sequences;
2. Expansion: For each candidate sequence, compute the probabilities for all possible next tokens, yielding $k \times |V|$ expanded sequences;
3. Pruning: Rank by **cumulative log probability** and retain the top $k$ highest-scoring sequences:

    $$
    score(x_1, \ldots, x_t) = \sum_{i=1}^{t} \log P(x_i \mid x_1, \ldots, x_{i-1})
    $$

4. Termination: Stop when all beams generate `<EOS>` or reach the maximum length, and output the sequence with the highest score.

- **Time Complexity**: $O(k \cdot n \cdot |V|)$, approximately $k$ times that of greedy decoding;
- **Space Complexity**: $O(k \cdot n)$, requiring storage for $k$ candidate sequences;
- **Pros**: Explores a broader search space compared to greedy decoding, typically finding sequences with higher cumulative probability;
- **Cons**: Still not globally optimal (exhaustive search complexity is $O(|V|^n)$); in tasks requiring high generation diversity (e.g., open-domain dialogue), beam search tends to produce "safe" but repetitive or dull outputs.

#### Length Penalty

In beam search, cumulative log probability favors shorter sequences (fewer log probability additions). A **Length Penalty** is typically introduced for normalization:

$$
	score_normalized(x_1, \ldots, x_t) = \frac{\sum_{i=1}^t \log P(x_i \mid x_{<i})}{t^\alpha}
$$

where $\alpha \in [0, 1]$ is the length penalty coefficient, where $\alpha = 0$ degrades to no penalty and $\alpha = 1$ corresponds to full normalization.

#### Comparison Summary

\begin{table}[h]



| **Attribute** | **Greedy Decoding** | **Beam Search** (beam=$k$) |
| --- | --- | --- |
| Candidates per step | 1 | $k$ |
| Search space | Minimal | Moderate |
| Output quality | Lower | Higher |
| Diversity | Low | Low (Deterministic) |
| Computational overhead | $O(n|V|)$ | $O(kn|V|)$ |
| Use cases | Real-time inference | Machine translation, summarization |


	\caption{Comparison between Greedy Decoding and Beam Search}
\end{table}

</details>

**16. What is the Difference Between Top-$k$ Sampling and Nucleus Sampling?**

<details>
<summary>💡 Show Answer</summary>

Top-$k$ sampling and nucleus sampling (Top-$p$ sampling) are both truncation sampling strategies used to introduce **randomness** and enhance generation diversity in language model decoding. The core difference between them lies in **how the candidate truncation set is defined**.

#### Top-$k$ Sampling

Top-$k$ sampling fixes the candidate set to the top-$k$ tokens with the highest probabilities, renormalizes their probabilities, and then samples:

$$
	P'(x_t = w) = \begin{cases} \dfrac{P(w \mid x_{<t})}{\sum_{w' \in \mathcal{V}_k} P(w' \mid x_{<t})} & w \in \mathcal{V}_k \\ 0 & otherwise \end{cases}
$$

where $\mathcal{V}_k$ is the set of the top-$k$ tokens with the highest probabilities.

- **Pros**: Simple to implement and computationally efficient (requires only sorting to pick Top-$k$);
- **Cons**: $k$ is a **fixed value** and cannot adapt to dynamic changes in the distribution. When the probability distribution is **peaked** (low entropy), the $k$ candidates may contain a large number of low-probability noise tokens; when the distribution is **flat** (high entropy), $k$ candidates may omit many plausible options.

#### Nucleus Sampling (Top-$p$ / Nucleus Sampling)

Nucleus sampling was proposed by Holtzman et al. (2020). It dynamically selects the smallest candidate set $\mathcal{V}_p$ whose cumulative probability exceeds a threshold $p$:

$$
	\mathcal{V}_p = \arg\min_{\mathcal{V} \subseteq \mathcal{V}_{sorted}} \left\{ |\mathcal{V}| : \sum_{w \in \mathcal{V}} P(w \mid x_{<t}) \geq p \right\}
$$

That is, tokens are sorted in descending order of probability and accumulated until the sum of probabilities first exceeds $p$, at which point the candidate set is truncated, renormalized, and sampled from.

- **Pros**: The candidate set size **dynamically adjusts with the distribution**. When the distribution is peaked, $|\mathcal{V}_p|$ is smaller (focusing on high-probability tokens); when the distribution is flat, $|\mathcal{V}_p|$ is larger (covering more plausible tokens), effectively avoiding the fixed truncation issue of Top-$k$;
- **Cons**: Computationally requires sorting probabilities ($O(|V| \log |V|)$), which is slightly slower than Top-$k$; hyperparameter tuning is still required for choosing $p$ (typically 0.9--0.95).

#### Intuitive Comparison Example

Assume a vocabulary size of 5 and consider the probability distributions under two scenarios:

\begin{table}[h]



|  | **token A** | **token B** | **token C** | **token D/E** |
| --- | --- | --- | --- | --- |
| Peaked Distribution | 0.90 | 0.07 | 0.02 | 0.005 |
| Flat Distribution | 0.25 | 0.22 | 0.20 | 0.17/0.16 |


	\caption{Examples of two probability distributions}
\end{table}

- **Top-$k$ ($k=3$)**: Both scenarios select \{A, B, C\}. In the peaked distribution, it includes low-quality C (probability only 0.02); in the flat distribution, it misses plausible options D and E.
- **Top-$p$ ($p=0.95$)**: In the peaked distribution, \{A, B\} already exceeds 0.95, resulting in only 2 candidates; in the flat distribution, \{A, B, C, D, E\} is required to exceed 0.95, including all 5 candidates, demonstrating better adaptability.

#### Combined Usage in Practice

Top-$k$ and Top-$p$ are often **used together**: Top-$k$ truncation is applied first, followed by Top-$p$ filtering within the remaining candidates, alongside a temperature coefficient to control the overall sharpness of the distribution:

\begin{lstlisting}[style=promptstyle]
	# Typical inference parameter configuration
	{
		"temperature": 0.8,   # Control distribution sharpness
		"top_k": 50,          # First limit the maximum number of candidates
		"top_p": 0.95,        # Then apply cumulative probability truncation
		"repetition_penalty": 1.1  # Suppress repetitive generation
	}
\end{lstlisting}

#### Comparison Summary

\begin{table}[h]



| **Attribute** | **Top-$k$ Sampling** | **Nucleus Sampling (Top-$p$)** |
| --- | --- | --- |
| Truncation Method | Fixed number of top $k$ tokens | Dynamic cumulative probability threshold $p$ |
| Candidate Set Size | Fixed ($= k$) | Dynamic (varies with distribution) |
| Distribution Adaptability | Weak | Strong |
| Implementation Complexity | Low (Top-$k$ sorting) | Medium (Full sorting + accumulation) |
| Recommended Hyperparameters | $k = 40\sim100$ | $p = 0.9\sim0.95$ |


	\caption{Comparison between Top-$k$ and Nucleus Sampling (Top-$p$)}
\end{table}

</details>

**17. FLOPs Calculation for a Single Forward Pass**

<details>
<summary>💡 Show Answer</summary>

FLOPs analysis of a forward pass serves as the foundation for evaluating model computational efficiency, conducting Roofline analysis, and planning training resources. Taking a standard Transformer ($L$ layers, hidden dimension $d_{model}$, number of attention heads $h$, per-head dimension $d_k = d_{model} / h$, FFN dimension $d_{ff} = 4 d_{model}$, and sequence length $n$) as an example, we perform a layer-by-layer analysis.

#### Basic Counting Rules

The FLOPs for matrix multiplication of $A \in \mathbb{R}^{m \times k}$ and $B \in \mathbb{R}^{k \times n}$ is $2mkn$ (each output element requires $k$ multiplications and $k-1$ additions, approximated as $2k$).

#### FLOPs Breakdown for a Single Transformer Layer

**(1) QKV Projection**

Projecting input $X \in \mathbb{R}^{n \times d_{model}}$ into $Q, K, V \in \mathbb{R}^{n \times d_{model}}$ respectively:

$$
	FLOPs_{QKV} = 3 \times 2 \times n \times d_{model} \times d_{model} = 6n d_{model}^2
$$


**(2) Attention Calculation**

- $QK^\top$ matrix multiplication: $2 \times n \times n \times d_{model} = 2n^2 d_{model}$ FLOPs
- Softmax normalization: $O(n^2)$ (much smaller than matrix multiplication, typically ignored)
- $AV$ matrix multiplication: $2 \times n \times n \times d_{model} = 2n^2 d_{model}$ FLOPs

$$
	FLOPs_{Attn} = 4n^2 d_{model}
$$


**(3) Output Projection**

Projecting attention output $\in \mathbb{R}^{n \times d_{model}}$ back to $d_{model}$:

$$
	FLOPs_{out} = 2 \times n \times d_{model} \times d_{model} = 2n d_{model}^2
$$


**(4) FFN (Two-layer MLP)**


$$
	FLOPs_{FFN} = 2 \times 2 \times n \times d_{model} \times d_{ff} = 4n \cdot d_{model} \cdot d_{ff} = 16n d_{model}^2
$$


(assuming $d_{ff} = 4 d_{model}$)

#### Total FLOPs for a Single Layer


$$
	FLOPs_{layer} = 6nd_{model}^2 + 4n^2 d_{model} + 2nd_{model}^2 + 16nd_{model}^2 = 24nd_{model}^2 + 4n^2 d_{model}
$$


#### Full Model Forward FLOPs

An $L$-layer Transformer plus the final LM Head ($d_{model} \times |V|$):

$$
	FLOPs_{forward} = L \left(24nd_{model}^2 + 4n^2 d_{model}\right) + 2n d_{model} |V|
$$


When $n \ll d_{model}$ (short sequences), the attention term $4n^2 d_{model}$ can be neglected, yielding forward FLOPs $\approx 2nN$ (where $N \approx 12 L d_{model}^2$ is the parameter count).

#### Total Training FLOPs

Backward pass FLOPs is roughly twice that of the forward pass, so total training FLOPs:

$$
	FLOPs_{train} \approx 6nN \times D_{total}
$$

where $D_{total}$ is the total number of training tokens. If activation recomputation is enabled, the forward pass runs twice, and the coefficient becomes 8.

#### Roofline Analysis: Compute-Bound vs. Memory-Bound

For each operator, we calculate its **Arithmetic Intensity**:

$$
	I = \frac{FLOPs}{Memory Access Bytes} \quad (FLOPs/Byte)
$$


- If $I > I_{ridge}$ (the hardware ridge point, approximately 200 FLOPs/Byte for A100), it is **compute-bound**, and performance is limited by compute capability;
- If $I < I_{ridge}$, it is **memory-bound**, and performance is limited by memory bandwidth.

Taking $n=2048, d_{model}=4096$ as an example:
\begin{table}[h]



| **Operator** | **FLOPs** | **Memory Access (Bytes)** | **Arithmetic Intensity** |
| --- | --- | --- | --- |
| QKV Projection (GEMM) | $6nd^2 \approx 2.06T$ | $\approx 0.10T$ | $\approx 206$ (Compute-bound) |
| $QK^\top$ (Attention) | $2n^2d \approx 34.4G$ | $\approx 0.27T$ | $\approx 128$ (Memory-bound) |
| FFN (GEMM) | $16nd^2 \approx 5.5T$ | $\approx 0.27T$ | $\approx 204$ (Compute-bound) |
| Softmax | $O(n^2)$ | $O(n^2 \cdot sizeof)$ | Low (Memory-bound) |


	\caption{Roofline analysis example for operators (A100, BF16)}
\end{table}

Thus, attention computation ($QK^\top$ and $AV$) becomes memory-bound under long sequence lengths, which is precisely why FlashAttention achieves significant speedup by reducing HBM access; whereas FFN and QKV projections are compute-bound, limited by compute capacity, and have limited room for speedup optimization.

</details>

**18. What is the Roofline Model?**

<details>
<summary>💡 Show Answer</summary>

![Roofline Model illustrating Memory-Bound and Compute-Bound regions in LLM inference.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig46_roofline_model.png)




The Roofline Model is used to describe the theoretical upper bound of hardware performance. The horizontal axis represents arithmetic intensity (Arithmetic Intensity, $AI = FLOPs / Bytes$), and the vertical axis represents throughput performance (FLOPS).

Two lines:
- **Horizontal Line**: Hardware peak compute capacity (Peak FLOPS)
- **Slanted Line**: Performance determined by hardware memory bandwidth upper bound ($Bandwidth \times AI$)

Decision methodology:
- Compute task $AI = Total FLOPs of the task / Total Memory Access Bytes$
- If $AI < Ridge Point$ ($Peak FLOPS / Bandwidth$), the task is **memory-bound** (performance is limited by bandwidth)
- If $AI > Ridge Point$, the task is **compute-bound** (performance is limited by compute capacity)

For example, the A100 peak performance is approximately 312 TFLOPS (FP16) with a bandwidth of around 2 TB/s, yielding a ridge point $AI \approx 156$. GEMM-type operations (large matrix multiplication) typically have high AI and are compute-bound; elementwise operations (such as LayerNorm, Softmax, activation functions) have low AI and are memory-bound.

</details>

**19. Inference Latency Estimation for Llama-2-70B on A100**

<details>
<summary>💡 Show Answer</summary>

The decode phase with batch size = 1 is a typical **memory-bound** scenario, where generating each token requires reading all parameters from GPU memory once.

Estimation formula:
\begin{verbatim}
Per-token latency ≈ Model Parameter Size (Bytes) / Memory Bandwidth
\end{verbatim}

Specific calculation:
- 70B parameters stored in FP16 = 70×$10^{9}$ × 2 bytes = 140 GB
- A100 GPU memory bandwidth ≈ 2 TB/s (A100 80GB SXM is approx. 2039 GB/s)
- Per-token latency ≈ 140GB / 2TB/s ≈ **70ms**

However, the 70B model itself exceeds the 80GB VRAM of a single GPU (140GB > 80GB), requiring multiple GPUs in practice (e.g., TP=2). In this case, bandwidth is aggregated, but communication overhead must also be considered. Empirical benchmark values typically range between 30--50ms/token (after multi-GPU tensor parallelism, the per-GPU memory read volume decreases and overall bandwidth utilization improves).

KV Cache read/write operations also contribute to memory access traffic, but weight loading dominates.

</details>

**20. Compute Bottlenecks vs. Memory Bottlenecks in Transformer**

<details>
<summary>💡 Show Answer</summary>

**Compute-bound (Compute Bottleneck)**:
- Large matrix multiplications (GEMM): QKV projection, output projection, two linear layers in FFN
- Attention computation in the Prefill phase (under long sequences, FLOPs for $QK^{\top}$ and $AV$ are very high)
- These operations have high AI and can fully utilize Tensor Core compute capacity

**Memory-bound (Memory Bandwidth Bottleneck)**:
- LayerNorm/RMSNorm: performs only a small number of operations per element but requires reading/writing the entire tensor
- Softmax: also elementwise + reduction, where memory access dominates
- Activation functions (GELU, SiLU, etc.)
- All GEMV operations in the Decode phase (when batch = 1, matrix multiplication degenerates into vector multiplication, causing AI to drop sharply)
- KV Cache reads (every step in the decode phase requires reading all historical KV values)
- Embedding lookup

Key takeaway: **Almost all operations in the decode phase are memory-bound** because the batch dimension is small, matrix operations degenerate into matrix-vector multiplications (GEMV), resulting in low compute utilization, where the bottleneck lies in moving weights from HBM to compute units.

</details>

**21. FlashAttention: Higher FLOPs but Faster**

<details>
<summary>💡 Show Answer</summary>

Standard Attention needs to compute and store the full $N \times N$ attention matrix ($S=QK^{\top}$, $P=softmax(S)$, $O=PV$), involving a large volume of HBM write and read operations for intermediate results.

FlashAttention loads blocks of Q, K, and V into SRAM (on-chip high-speed cache) for computation through **tiling + online softmax**, avoiding writing the full $N \times N$ matrix back to HBM. Although additional recomputation is required to realize online softmax (which adds some FLOPs, such as repeatedly computing rescalings for certain intermediate terms), resulting in total FLOPs slightly higher than the standard implementation,

because:
- Standard Attention is **memory-bound** (the bottleneck lies in HBM read/write of $N^2$-sized intermediate matrices)
- FlashAttention drastically reduces the number of HBM accesses (reducing IO complexity from $O(N^2)$ to $O(N^2/M)$, where $M$ is a term related to the SRAM size)

According to the Roofline model, when the bottleneck is memory bandwidth rather than compute capacity, reducing memory access is more critical than reducing FLOPs. FlashAttention trades "more computation" for "fewer memory accesses", leading to lower overall latency. This is a classic case of **algorithm redesign to fit hardware memory hierarchy**.

</details>

**22. Kernel Fusion**

<details>
<summary>💡 Show Answer</summary>

**Principle**: Merge multiple consecutive small operators that originally required reading and writing HBM individually (e.g., LayerNorm $\rightarrow$ Linear $\rightarrow$ GELU $\rightarrow$ Linear) into a single CUDA kernel. Multi-step computation is completed on-chip (registers / shared memory), accessing HBM only at the very beginning and end.

**Benefits**:
1. **Reduce HBM memory accesses**: Avoid writing and reading intermediate results, yielding significant benefits for memory-bound elementwise operators (such as LayerNorm, activation functions, bias-add, dropout).
2. **Reduce kernel launch overhead**: Each kernel launch incurs a fixed scheduling latency; fusion reduces the number of launches.
3. **Improve data locality**: Data is reused within SRAM/registers, reducing cross-layer data movement.

Typical examples: FlashAttention itself is a fusion of attention operators; FusedLayerNorm, FusedAdam, bias+GELU fusion, etc. The tradeoff is that fused kernels have higher implementation complexity and lower generality, requiring hand-writing for specific shapes/hardware or automatic code generation via compilers (such as Triton, TorchInductor).

</details>

**23. Grid / Block / Thread / Warp Relationships in CUDA**

<details>
<summary>💡 Show Answer</summary>

Hierarchical Structure (from largest to smallest):
- **Grid**: The set of all blocks included in a single kernel launch, corresponding to the entire compute task.
- **Block (Thread Block)**: An execution unit within a grid containing multiple threads. Threads within a block can communicate via shared memory and synchronize (`__syncthreads()`).
- **Thread**: The smallest execution unit. Each thread executes the same kernel code, processing different data (SIMT model).
- **Warp**: The smallest unit of hardware scheduling, fixed at **32 threads**. Threads within the same warp execute the same instruction (SIMD lockstep). A block is divided into multiple warps for scheduling and execution.

Hardware Mapping Relationships:
- A block is assigned to a Streaming Multiprocessor (SM) for execution; the warp scheduler inside the SM is responsible for scheduling warps.
- An SM can concurrently reside multiple blocks/warps, hiding memory latency by switching between warps (latency hiding).
- Grid dimensions can be 1D/2D/3D, used to map multidimensional data structures (such as image rows/columns, matrix tiles).

Design Principle: Block size is typically set to a multiple of 32 (to avoid thread idling within a warp), commonly 128/256/512 threads/block.

</details>

**24. Differences Between Prefill and Decode**

<details>
<summary>💡 Show Answer</summary>

| | Prefill | Decode |
|---|---|---|
| **Input** | Full prompt (multiple tokens) | Single token (generated from previous step) |
| **Matrix Shape** | (seq_len, hidden) GEMM | (1, hidden) GEMV |
| **Compute Property** | compute-bound | memory-bound |
| **Parallelism** | Parallelizable along sequence dimension | Sequence dimension = 1, low parallelism |
| **KV Cache** | Writes KV for all tokens at once | Reads all historical KV + writes current token's KV each step |
| **Dominant Time Factor** | Compute power (high FLOPS utilization) | VRAM bandwidth (reading weights and KV Cache) |

**Performance Characteristic Differences**: In the Prefill phase, GPU utilization is high and throughput is large, but a single step takes longer (positively correlated with prompt length); in the Decode phase, generation occurs token-by-token, requiring all model weights to be reloaded at every step while GPU compute units remain largely idle. Decode is the primary source of inference latency (the sequential nature of autoregressive generation makes the total decode time usually far exceed prefill time).

This is also the motivation behind Prefill/Decode (PD) disaggregation architecture: the two phases have completely different resource requirement characteristics, and disaggregated scheduling allows independent optimization for each.

</details>

**25. Speculative Decoding**

<details>
<summary>💡 Show Answer</summary>

![Speculative Decoding architecture with fast draft generation and parallel target verification.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig47_speculative_decoding_pipeline.png)




**Core Principle**: Use a small and fast "draft model" to first generate multiple (e.g., $K$) candidate tokens autoregressively, and then use the large "target model" in a **single forward pass** to verify in parallel whether these $K$ tokens align with the output distribution of the target model.

**Verification Mechanism**:
- For each candidate token, decide whether to accept according to rejection sampling rules: if the probability of the token given by the target model at that position is $\ge$ the probability given by the draft model, it is accepted; otherwise, it is rejected with a certain probability and resampled from a modified distribution.
- Once a position is rejected, all subsequent candidates are discarded, and generation continues from that position using the token sampled by the target model.

**Why It Accelerates**:
- Key Insight: The decode phase is memory-bound; the **latency of a single forward pass** in the large model processing 1 token versus $K+1$ tokens is **almost identical** (both involve a single memory read of model weights; the increased computation is not the bottleneck).
- Therefore, using one forward pass of the target model to verify $K$ tokens with an average acceptance rate of $\alpha$ is equivalent to generating an expected $(1 + \alpha + \alpha^2 + \dots)$ tokens within the latency of 1 forward pass.
- The expected speedup ratio is approximately $(1-\alpha^{K+1})/((1-\alpha) \times (single-step overhead ratio))$. The higher the acceptance rate, and the faster and more accurate the draft model, the better the speedup.

The output distribution is mathematically identical to direct sampling with the target model (lossless acceleration). The trade-off is the need to maintain an additional draft model; furthermore, when there is a large divergence between the draft and target model distributions, the acceptance rate drops and acceleration becomes marginal.

</details>

**26. Two Benefit Dimensions of Quantization & Acceleration Differences Between Prefill and Decode**

<details>
<summary>💡 Show Answer</summary>

**Two Benefit Dimensions**:
1. **VRAM Footprint Reduction**: Quantizing weights from FP16/BF16 (2 bytes) to INT8 (1 byte) or INT4 (0.5 bytes) reduces VRAM consumption by 2--4$\times$, allowing larger batch sizes or longer context lengths.
2. **Memory Bandwidth Savings**: The amount of weight data read from HBM decreases proportionally, directly reducing latency in memory-bound scenarios.

**Why Decode Accelerates Significantly While Prefill Does Not**:

The Decode phase is memory-bound (GEMV, where weight retrieval dominates latency). Quantization shrinks the weight size, proportionally reducing read time and significantly decreasing latency (e.g., INT4 theoretically offers nearly $4\times$ speedup compared to FP16).

The Prefill phase is compute-bound (GEMM, where matrix multiplication computation dominates). The bottleneck lies in Tensor Core compute capacity rather than weight retrieval. Although weight read volume is reduced after quantization:
- If the hardware does not support native acceleration for low-precision operators (e.g., INT4 GEMM kernels), weights must first be dequantized back to FP16 before computation, introducing additional overhead.
- Even if low-precision Tensor Cores are supported, the compute FLOPs (after reduction to floating-point operations) are not reduced, resulting in speedup far lower than in decode.

Therefore, in practice, quantization techniques (such as GPTQ/AWQ) primarily target memory-bound decode / small-batch scenarios, offering limited benefits or even slowdowns for large-batch prefill due to dequantization overhead.

</details>

**27. Shared Memory Optimization Principles and Use Cases**

<details>
<summary>💡 Show Answer</summary>

**Principle**: Shared Memory is on-chip high-speed storage within each SM on the GPU (typically tens to hundreds of KB in capacity), with access latency far lower than Global Memory (HBM). All threads within the same block can share access to it.

**Optimization Principle**:
- Load data from HBM into shared memory at once; when threads within a block reuse the data, they read directly from shared memory, avoiding redundant global memory accesses caused by each thread independently accessing HBM.
- Reduce global memory accesses and improve data reuse rate (increasing arithmetic intensity, which is particularly effective for memory-bound kernels).

**Use Cases**:
1. **Matrix Multiplication (GEMM)**: Tile-based algorithms load matrix blocks into shared memory so data within each tile can be reused by multiple threads (classic CUDA GEMM optimization).
2. **Convolution Operations**: Input data for sliding windows overlaps between neighboring output positions and can be cached in shared memory.
3. **Reduction Operations**: Such as sum/max calculations in softmax or layernorm—perform intra-block reduction in shared memory first, followed by inter-block reduction.
4. **FlashAttention**: Load Q, K, and V blocks into shared memory to complete attention computation on-chip, avoiding writing intermediate matrices back to HBM.

**Note**: Shared memory capacity is limited. Allocating too much will restrict the number of blocks residing on each SM (affecting occupancy). Care must also be taken regarding bank conflict issues (see 731).

</details>

**28. Thread / Block / Grid Relationships**

<details>
<summary>💡 Show Answer</summary>

(Duplicate of 719, supplementary notes)

- A single Kernel launch corresponds to a **Grid**.
- A Grid consists of multiple **Blocks** (can be arranged in 1D/2D/3D).
- Each Block consists of multiple **Threads** (can also be arranged in 1D/2D/3D).
- Each Thread executes the same kernel function code, computing the index of the data it is responsible for via built-in variables like `threadIdx`, `blockIdx`, `blockDim`, and `gridDim`.

\begin{verbatim}
int idx = blockIdx.x * blockDim.x + threadIdx.x;
\end{verbatim}

- A Block is the basic unit of resource allocation and scheduling: a block is assigned to an SM for execution; threads within a block can synchronize (`__syncthreads`) and share shared memory.
- Internally, a Block is further divided into **Warps** (32 threads) for hardware scheduling and execution.

</details>

**29. What is a Bank Conflict and How to Resolve It?**

<details>
<summary>💡 Show Answer</summary>

![CUDA execution model hierarchy and Shared Memory Bank organization.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig49_cuda_execution_model.png)




**Bank Conflict**: Shared Memory is divided into multiple (typically 32) memory banks of equal width that can be accessed in parallel. If multiple threads within the same warp simultaneously access different addresses in the **same bank**, these accesses are **serialized**, resulting in performance degradation.

**Typical Scenario**: When accessing a shared memory array, if the array is accessed with a stride that shares a common divisor with the number of banks (32), it easily causes multiple threads within the same warp to land on the same bank.

For example: With 32 banks, if thread $i$ accesses `shared_mem[i * 32]`, all threads access bank 0, producing a 32-way conflict.

**Optimization Methods**:
1. **Padding**: Insert extra elements between array rows/columns to break the divisibility relation between stride and bank count. For instance, declaring a 2D array as `shared float tile[32][33]` instead of `[32][32]` adds one extra element per row, shifting the bank assignment for the same column in adjacent rows.
2. **Change Access Pattern**: Adjust thread access order so that threads within the same warp access contiguous addresses (each thread accesses a different bank).
3. **Leverage Broadcast Mechanism**: If all threads access the exact same address (rather than different addresses in the same bank), hardware broadcast is supported, which does not count as a conflict.

Special case: When all threads access the exact same address, no conflict occurs (broadcast); a conflict occurs only when accessing "the same bank, different addresses".

</details>

**30. MoE Main Workflow and Reasons for Token-Level Routing**

<details>
<summary>💡 Show Answer</summary>

**Main MoE Workflow**:
1. The hidden state $h$ of each input token is fed into the **Router (gating network)**, which outputs scores for each expert (typically via softmax).
2. Select the **Top-$K$ experts** based on scores (e.g., Top-2).
3. The hidden state of the token is dispatched to the selected $K$ experts.
4. Each expert independently performs FFN computation on the received token.
5. Outputs from each expert are **weighted and summed (combined)** according to the weights provided by the Router to produce the token's final output.
6. In distributed scenarios (Expert Parallelism, EP), dispatch and combine are implemented via **All2All communication** for cross-GPU token routing.

**Why Routing is Done at Token Granularity**:
- The core idea of MoE is **conditional computation**: different inputs should activate different "expert knowledge", and each token represents an independent semantic unit (a word/subword) whose semantic content and required processing may vary significantly.
- Routing at token granularity allows the model to dynamically select the most relevant combination of experts based on the specific semantic features of each token (rather than uniform sequence-wide features), achieving fine-grained specialization.
- Routing at sequence granularity would force all tokens in a sequence to share the same set of experts, losing token-level flexibility and easily leading to expert load imbalance (certain experts might never be selected by certain sequence types).
- Token-level routing also brings challenges: **load imbalance** (some experts selected by too many tokens experience heavy VRAM/compute pressure, requiring capacity factor limits and auxiliary loss for balance) and **dynamic communication patterns** caused by different tokens in a batch routing to different experts (All2All communication volume varies with routing outcomes).

</details>

**31. RoPE Computation Logic & Long Context Extension Schemes**

<details>
<summary>💡 Show Answer</summary>

![Rotary Position Embedding (RoPE) 2D rotation geometry.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig08_rope_rotary_embedding.png)




**RoPE (Rotary Position Embedding) Computation Logic**:
- Core Idea: Inject position information by applying a **rotary transformation** to Query and Key vectors based on position, making the inner product of two vectors (attention score) depend solely on their **relative position**.
- Group the $d$-dimensional vector into pairs ($d/2$ groups), treating each pair as a vector in a 2D plane.
- For a vector at position $m$, the $i$-th group (2D subspace) is rotated by an angle $m \times \theta_i$, where $\theta_i = base^{-2i/d}$, with base usually set to 10000.
- The rotation matrix acts on ($x_{2i}, x_{2i+1}$):
    \begin{verbatim}
    x'_{2i}   = x_{2i}*cos(mθ_i) - x_{2i+1}*sin(mθ_i)
    x'_{2i+1} = x_{2i}*sin(mθ_i) + x_{2i+1}*cos(mθ_i)
    \end{verbatim}
- Due to properties of rotary transformations, $Q_m \cdot K_n$ (inner product) depends only on relative position $(m-n)$ and $\theta$, naturally encoding relative position information.

**Comparison of Long Context Extension Schemes**:

| Method | Approach | Characteristics |
|---|---|---|
| **Sliding Window** | Restrict attention computation to a fixed window size (e.g., Mistral's local attention) | Does not modify RoPE itself; controls compute volume by limiting receptive field; long-distance information is passed indirectly through multi-layer stacking (similar to CNN receptive field expansion); cannot directly extend the model's ability to comprehend absolute long-distance positions |
| **DynamicNTK** | Dynamically adjust the base value of RoPE during inference based on actual sequence length, enabling smooth frequency distribution extrapolation when exceeding training length | Requires no fine-tuning; base scales proportionally with sequence length (NTK-aware scaling); behavior within training length remains unchanged; frequency gradually alters beyond training length, mitigating the "periodic repetition confusion" in high-frequency dimensions during direct extrapolation |
| **YaRN** | Combines NTK interpolation concepts by applying different interpolation strategies to different frequency dimensions: high-frequency dimensions (sensitive to local positions) undergo extrapolation/minor adjustments, while low-frequency dimensions (global positions) undergo linear interpolation, alongside introducing attention temperature scaling correction | Typically requires minimal fine-tuning for optimal performance; maintains higher precision over longer contexts compared to pure NTK or PI (Position Interpolation), serving as one of the mainstream long context extension schemes currently

|

Core Contradiction: During training, the model only sees position embeddings up to a limited length. Direct extrapolation to longer sequences pushes high-frequency rotation angles outside the training distribution ("unseen rotation angles" causing chaotic attention patterns). All schemes essentially trade off between "altering the frequency distribution" and "preserving local attention patterns learned during training".

</details>

**32. Common Sampling Algorithms**

<details>
<summary>💡 Show Answer</summary>

**Argmax (Greedy Decoding)**: Selects the token with the highest probability at each step. The result is highly deterministic, but easily falls into repetition and lacks diversity.

**Temperature ($T$)**:
- Scales the logits: `logits / T`, followed by softmax.
- $T < 1$: The distribution becomes sharper (more deterministic, biased towards high-probability tokens).
- $T > 1$: The distribution becomes flatter (more random, increases diversity).
- $T \to 0$ is equivalent to argmax; $T=1$ corresponds to the original distribution.

**Top-$K$ Sampling**:
- Retains only the $K$ tokens with the highest probabilities, renormalizes them, and samples from this set.
- Prevents sampling from extremely low-probability "long-tail" tokens that produce unreasonable output; a smaller $K$ leads to more conservative generation.

**Top-$P$ (Nucleus) Sampling**:
- Sorts tokens by probability in descending order and accumulates them until the cumulative probability exceeds the threshold $P$, preserving this smallest set to resample from.
- More dynamic than Top-$K$: the set is smaller when the distribution is sharp (close to greedy decoding) and larger when the distribution is flat (more diverse), offering better adaptivity.

**Beam Search**:
- Maintains multiple candidate sequences ($beam size = B$). At each step, it expands all possible next tokens for each candidate and retains the $B$ sequences with the highest cumulative scores (sum of log probabilities).
- Used for tasks seeking "overall sequence probability maximization" (e.g., translation, summarization). In open-ended generation tasks, it tends to generate repetitive and non-diverse outputs, and its computational overhead grows with the beam size.

**Combined Usage**: In practical generation, a combination of `temperature + top-K + top-P` sampling is commonly used. Top-$K$/Top-$P$ is applied first to truncate the candidate set, followed by temperature adjustment of the distribution shape before sampling.

</details>

**33. KV Cache Size Calculation & Distribution under TP/PP**

<details>
<summary>💡 Show Answer</summary>

**KV Cache Size Formula**:
\begin{verbatim}
KV Cache size = 2 (K and V) × batch_size × seq_len × num_layers × num_kv_heads × head_dim × dtype_bytes
\end{verbatim}

For example: Llama-2-7B (32 layers, 32 KV heads, head_dim=128, FP16), batch=1, seq_len=4096:
\begin{verbatim}
2 × 1 × 4096 × 32 × 32 × 128 × 2 bytes ≈ 2.1 GB
\end{verbatim}

When using **GQA/MQA** (where the number of KV heads is smaller than the number of Query heads), `num_kv_heads` will be much smaller than `num_attention_heads`, significantly reducing the KV Cache size (e.g., when `num_kv_heads`=8, the KV Cache is reduced by $4\times$).

**Distribution under TP**:
- TP splits the attention head dimension across different GPUs (each GPU handles `num_kv_heads` / `TP_size` KV heads).
- Therefore, the KV Cache size on each GPU is **1/TP_size** of the total amount.
- Note: If `num_kv_heads` < `TP_size` (e.g., in MQA where `num_kv_heads`=1 and `TP_size`=8), the KV heads cannot be split further, resulting in **redundant replication** of KV heads across GPUs (each GPU still needs to store a full copy of the KV head).

**Distribution under PP**:
- PP splits the layer dimension across different stages.
- Each stage (GPU) only needs to store the KV Cache corresponding to the layers assigned to it; thus, the KV Cache size on each GPU is **1/PP_size** of the total amount (split proportionally by the number of layers).
- KV Caches across different stages do not overlap. During sequence processing, while activations are passed along the inter-layer pipeline, the KV Cache is stored locally in its respective stage.

**Combined**: When TP and PP are used simultaneously, the KV Cache per GPU $\approx Total KV Cache / (TP_size \times PP_size)$ (assuming `num_kv_heads` is sufficiently large and divisible by `TP_size`).

</details>

**34. Comparison of Model Lightweighting Methodologies**

<details>
<summary>💡 Show Answer</summary>

**AWQ (Activation-aware Weight Quantization)**:
- Core Idea: Identifies "salient weight channels" that have a large impact on activations, protecting the precision of these important weights via **per-channel scaling** instead of using mixed precision.
- Scaling Factor Search: Searches for scales on a calibration set to minimize quantization error, transferring "salience" from weights to activations (via mathematically equivalent transformations: $W \cdot s$ and $X / s$).
- Features: Weight-only quantization, requires no backpropagation/training, hardware-friendly (compatible with standard INT4 kernels).

**SmoothQuant**:
- Target Scenario: Designed for joint **weight and activation quantization** (W8A8).
- Core Issue: Activations in LLMs exhibit prominent **outliers**, leading to large errors in per-tensor quantization.
- Solution: Introduces a smoothing factor $s$ to transfer part of the "sharp distribution" of activations to weights ($X / s, W \cdot s$), making activations smooth and easy to quantize, while weight distributions remain relatively flat so quantization errors are easily tolerated.
- Essence: Performs a **re-balancing (migration)** of quantization difficulty between weights and activations.

**GPTQ (GPT Quantization)**:
- A layer-wise quantization framework based on OBQ (Optimal Brain Quantization), which uses second-order information (the inverse of the Hessian matrix) to quantize weights column by column, compensating quantization errors onto unquantized weights in real-time to minimize overall output error.
- It is an **error-compensating** layer-wise greedy quantization algorithm that requires a small amount of calibration data to compute the Hessian approximation.

**SpinQuant**:
- Core Idea: Applies orthogonal transformations to weights and activations by introducing **learnable rotation matrices**.
- Rotation transformations preserve mathematical computation results (due to orthogonal matrices) while redistributing outliers in activations/weights to make the transformed distribution more suitable for quantization (analogous to "rotating and diffusing" outliers across dimensions to lower single-dimension extreme values).
- By optimizing rotation matrices on calibration data (e.g., Cayley parameterization + optimization algorithms), it is more flexible than SmoothQuant's simple scaling and achieves better results for 4-bit quantization.

**Summary of Evolutionary Path**: AWQ and GPTQ focus on weight-only quantization (protecting salient weights / error compensation); SmoothQuant and SpinQuant focus on joint weight + activation quantization (reshaping distributions via mathematical equivalence transformations to fit quantization). SpinQuant is a further generalization of SmoothQuant's philosophy (scaling $\to$ rotation).

</details>

**35. Data Parallelism / Tensor Parallelism / Pipeline Parallelism**

<details>
<summary>💡 Show Answer</summary>

**Data Parallelism (DP)**:
- Each GPU holds a complete replica of the model, and input data is split across different GPUs.
- Each GPU performs forward and backward passes independently, and gradients are synchronized via AllReduce.
- **Applicable Scenarios**: The model fits into single-card memory, and data throughput is the primary bottleneck.
- **Trade-off**: Large communication volume (AllReduce of all gradients at every step), high memory redundancy (each card stores the full model + optimizer states).

**Tensor Parallelism (TP)**:
- Splits matrix operations within a single layer (such as the weight matrix of a Linear layer) row-wise or column-wise across different GPUs. Each GPU computes a partial result, which is then combined via AllReduce/AllGather.
- **Applicable Scenarios**: Single-layer parameters are too large to fit on a single card, requiring intra-node multi-card collaboration; extremely sensitive to communication latency (requires synchronization at every layer's forward pass).
- **Trade-off**: Highest communication frequency (multiple collective communications per Transformer block), requires extremely high bandwidth (NVLink), and efficiency drops drastically across nodes.

**Pipeline Parallelism (PP)**:
- Splits the model vertically by layers across different GPUs (different GPUs handle different layers), and data flows through each stage sequentially like an assembly pipeline.
- **Applicable Scenarios**: The model depth is very deep with massive total parameter counts, requiring cross-node scaling.
- **Trade-off**: Suffers from pipeline bubbles (idle GPU time while waiting for upstream/downstream data); requires micro-batching and reasonable scheduling strategies (such as 1F1B) to reduce bubbles; communication volume is relatively small (passing only inter-layer activations), demanding lower bandwidth than TP.

In practice, large model training typically uses a **hybrid of all three** (3D Parallelism): intra-node TP, inter-node PP, and DP across multiple groups.

</details>

**36. Why TP Is Suited for Intra-Node and PP for Inter-Node**

<details>
<summary>💡 Show Answer</summary>

**TP Is Suited for Intra-Node**:
- TP requires AllReduce/AllGather communications within every Transformer layer (e.g., at least once in attention and once in FFN), resulting in extremely high communication frequency (2+ times per layer).
- These communications involve large data volumes (activation tensors) and are extremely sensitive to latency---communication cannot be overlapped with computation, making it a strict synchronous blocking point.
- Intra-node GPUs are interconnected via **NVLink/NVSwitch**, offering bandwidths up to hundreds of GB/s or even TB/s with ultra-low latency, capable of supporting high-frequency, high-volume data transfers.
- Inter-node networks use InfiniBand/RoCE, where bandwidth (typically tens to hundreds of GB/s) and latency are far inferior to NVLink. If TP spans across nodes, per-layer communication latency becomes a severe bottleneck.

**PP Is Suited for Inter-Node**:
- PP only transfers **activations** (forward) and **gradients** (backward) between stages. The data volume is significantly smaller than TP (only the output tensor of a specific layer, rather than all weight-related intermediate results).
- Communication occurs at stage boundaries, with a frequency far lower than TP (once per micro-batch, rather than multiple times per layer).
- This "low-frequency, pipeline-overlapped" communication pattern has much relaxed requirements on network bandwidth and latency, making it well-suited for inter-node network conditions.
- The pipeline bubble itself can be mitigated by increasing the number of micro-batches, and extra inter-node latency can be hidden by the parallel execution of the pipeline.

In short: **TP communication is frequent and sensitive to bandwidth/latency $\to$ must use the fastest interconnect (intra-node NVLink); PP communication is sparse and pipeline-overlapped $\to$ can tolerate slower interconnects (inter-node networking)**.

</details>

**37. What Are the Three Stages of ZeRO?**

<details>
<summary>💡 Show Answer</summary>

ZeRO (Zero Redundancy Optimizer) eliminates memory redundancy in data parallelism by partitioning various training states. The three states are: Optimizer States ($O$), Gradients ($G$), and Parameters ($P$).

- **Stage 1 (ZeRO-1)**: Partitions **Optimizer States** (such as Adam's momentum and variance). Each GPU stores only $1/N$ of the optimizer states. When updating parameters, an AllGather is performed first to obtain the updates corresponding to the full gradients.
- **Stage 2 (ZeRO-2)**: On top of Stage 1, further partitions **Gradients**. Each GPU stores only the gradients required for updating its assigned parameters; after backpropagation, gradients are distributed to the corresponding GPUs via Reduce-Scatter.
- **Stage 3 (ZeRO-3)**: On top of Stage 2, further partitions the **Model Parameters** themselves. Each GPU persistently stores only $1/N$ of the parameters. During forward and backward passes, the full parameters for the required layer are temporarily gathered via AllGather and released immediately after use.

**Memory and Overhead Trade-off**:
- Memory Savings: Stage 1 $\to$ Stage 2 $\to$ Stage 3 continuously decreases memory footprint. Theoretically, Stage 3 can reduce model state memory consumption to $O(1/N)$, where $N$ is the number of GPUs.
- Overhead:
- Stage 1/2: Communication volume is comparable to (or even better than) standard DP's AllReduce, resulting in low overhead.
- Stage 3: Extra AllGathers are required for every layer during both forward and backward passes to fetch parameters. Communication volume increases significantly ($\approx 1.5\times$ of standard DP) and introduces additional latency (must wait for parameters to arrive before computation), placing higher demands on network bandwidth.
- Finer partitioning yields greater memory savings, but communication overhead and implementation complexity (managing frequent gather/release timings) increase accordingly.

In practice, ZeRO-3 is often combined with offloading (CPU/NVMe) to further break single-node memory limits, though this introduces additional PCIe transfer latency.

</details>

**38. What Issues Exist in Large Model Extrapolation?**

<details>
<summary>💡 Show Answer</summary>

Large model "extrapolation" refers to processing inputs during inference whose length exceeds the maximum sequence length used during training. In this scenario, two core issues primarily arise:

1. **Out-of-Distribution (OOD) Issue of Position Embeddings**: Whether using absolute position embeddings (such as sinusoidal embeddings in the original Transformer, or learnable position embeddings in BERT) or relative position embeddings (such as RoPE, ALiBi), the model has only seen position indices within the trained length range during training. During inference, once position encoding values outside the training range appear, the model has no learned guarantees for representing these positions, leading to erratic attention weight calculations and a sharp drop in output quality. Taking RoPE as an example, the frequency range corresponding to rotation angle $\theta$ during training is limited; positions exceeding this range cause the rotation matrices to severely oscillate in high-frequency components, breaking the relative consistency of positional information.

2. **Attention Dilution Issue**: When computing softmax, the Transformer's attention mechanism normalizes the similarity scores between the current token and **all** tokens in the sequence. When the sequence length greatly exceeds the training length, the number of tokens participating in normalization increases sharply, diluting attention weights so that the attention probability assigned to each token approaches a uniform distribution ($1/n$). This makes it difficult for the model to focus on truly relevant context, causing effective information to be drowned out by numerous irrelevant tokens, and degrading the model's context understanding capability as sequence length grows. This phenomenon is also referred to as "Attention Collapse."

The superposition of these two issues causes the performance of pre-trained LLMs to deteriorate rapidly when directly performing long-context inference beyond the training length, with perplexity surging drastically.

</details>

**39. How to Solve Inconsistency and Long-Sequence Generalization Issues in LLM Extrapolation?**

<details>
<summary>💡 Show Answer</summary>

**Core Idea**: By using Attention Mask operations, restrict the context window length that each token actually "sees" during inference to the range used during training, thereby simultaneously eliminating both position embedding OOD and attention dilution issues.

**Specific Methods**:

- **Sliding Window Attention**: Sets a fixed-size local attention window for each token (equal to the maximum training length), attending only to the most recent $w$ tokens while masking out all positions outside the window via Attention Mask. In this way, regardless of input sequence length, the context length for each token remains consistent with training, and position embeddings stay within the trained range. Mistral 7B adopts this strategy.

- **StreamingLLM**: On top of sliding window attention, it retains several "Attention Sink" tokens at the beginning of the sequence (typically the first 4 tokens), based on findings that LLMs tend to concentrate significant attention on initial tokens. StreamingLLM's Attention Mask maintains both "initial anchors + local window", achieving stable inference in infinite streaming scenarios.

- **Position Interpolation (PI)**: Linearly scales position indices (e.g., compressing and mapping positions from $[0, L_{max_ext}]$ back to $[0, L_{max_train}]$), ensuring position embedding values for ultra-long sequences always fall within the training distribution. Combined with light fine-tuning, it restores precision lost to compression. LLaMA's long-context extension adopted this scheme.

- **RoPE Scaling Variants (YaRN, LongRoPE, etc.)**: Specifically targeting RoPE position embeddings, these methods apply different scale factors across frequency components (compressing high-frequency components more aggressively while preserving low-frequency components). This achieves long-context extrapolation while maintaining short-distance positional accuracy. Combined with Attention Mask and minor long-text fine-tuning, the context window can be expanded to 128K or even longer.

**Unified Perspective**: The essence of the above methods is to ensure, via Attention Mask or position embedding transformations, that the effective attention calculation for each token during inference is equivalent to operating within the training window length. This fundamentally eliminates distribution inconsistency between training and inference while keeping the effective number of tokens in the attention distribution within the model's well-learned range.

</details>

