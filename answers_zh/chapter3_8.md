# 3.8 推理优化与加速 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 3 章：预训练、推理与工程优化 · 共 39 题

**1. attention 是怎么计算出来的？**

<details>
<summary>💡 查看答案</summary>

注意力机制（Attention Mechanism）是 Transformer 的核心组件，其标准形式为**缩放点积注意力**（Scaled Dot-Product Attention）。

#### 基本概念

注意力机制将输入映射为三个矩阵：
- $Q \in \mathbb{R}^{n \times d_k}$：Query 矩阵，表示当前位置"想要查询什么信息"；
- $K \in \mathbb{R}^{m \times d_k}$：Key 矩阵，表示各位置"提供什么信息的索引"；
- $V \in \mathbb{R}^{m \times d_v}$：Value 矩阵，表示各位置"实际携带的内容"。

其中 $n$ 为 Query 序列长度，$m$ 为 Key/Value 序列长度，$d_k$ 为 Key 的维度，$d_v$ 为 Value 的维度。

#### 计算公式


$$
	Attention(Q, K, V) = softmax\!\left(\frac{QK^\top}{\sqrt{d_k}}\right) V
$$


#### 逐步推导

1. **计算相似度得分**：通过点积衡量 Query 与每个 Key 的相关性：

    $$
    S = QK^\top \in \mathbb{R}^{n \times m}
    $$

    其中 $S_{ij}$ 表示第 $i$ 个 Query 与第 $j$ 个 Key 之间的相似度。

2. **缩放（Scaling）**：将得分除以 $\sqrt{d_k}$，得到缩放后的得分 $\hat{S} = S / \sqrt{d_k}$。

    缩放的必要性：当 $d_k$ 较大时，点积的方差约为 $d_k$（假设 $Q, K$ 各分量独立同分布于均值为 0、方差为 1 的分布），即：

    $$
    Var(Q_i \cdot K_j) = d_k
    $$

    未缩放的大值会使 softmax 进入梯度极小的饱和区，导致训练不稳定。除以 $\sqrt{d_k}$ 后方差归一化为 1，缓解此问题。

3. **Softmax 归一化**：对每行做 softmax，将相似度转换为概率分布（注意力权重）：

    $$
    A_{ij} = \frac{\exp(\hat{S}_{ij})}{\sum_{k=1}^{m} \exp(\hat{S}_{ik})}, \quad A \in \mathbb{R}^{n \times m}
    $$

    $A_{ij}$ 表示第 $i$ 个 Query 对第 $j$ 个 Value 的注意力权重，满足 $\sum_j A_{ij} = 1$。

4. **加权求和**：以注意力权重对 Value 矩阵加权求和，得到最终输出：

    $$
    O = AV \in \mathbb{R}^{n \times d_v}
    $$

    输出的第 $i$ 行 $O_i = \sum_j A_{ij} V_j$，是各 Value 向量的加权组合。

#### 多头注意力（Multi-Head Attention）

实际中通常使用多头注意力，将 $Q, K, V$ 分别投影到 $h$ 个子空间，并行计算注意力后拼接：

$$
	MultiHead(Q, K, V) = Concat(head_1, \ldots, head_h) W^O
$$


$$
	head_i = Attention(QW_i^Q,\; KW_i^K,\; VW_i^V)
$$

其中 $W_i^Q \in \mathbb{R}^{d_{model} \times d_k}$，$W_i^K \in \mathbb{R}^{d_{model} \times d_k}$，$W_i^V \in \mathbb{R}^{d_{model} \times d_v}$，$W^O \in \mathbb{R}^{hd_v \times d_{model}}$。多头机制使模型能够在不同子空间中同时关注不同类型的信息。

#### 计算复杂度

\begin{table}[h]



| **操作** | **时间复杂度** | **空间复杂度** |
| --- | --- | --- |
| $QK^\top$ 矩阵乘 | $O(n^2 d_k)$ | $O(n^2)$ |
| Softmax | $O(n^2)$ | $O(n^2)$ |
| $AV$ 矩阵乘 | $O(n^2 d_v)$ | $O(nd_v)$ |


	\caption{Scaled Dot-Product Attention 的计算与空间复杂度（$n$ 为序列长度）}
\end{table}

注意力矩阵 $A \in \mathbb{R}^{n \times n}$ 的存储开销随序列长度**平方增长**，这是长序列场景下内存瓶颈的根本来源，也是 FlashAttention 等优化工作的出发点。

</details>

**2. 原生的 softmax 在工程上有哪些问题？**

<details>
<summary>💡 查看答案</summary>

原生 Softmax 在工程实现中存在**数值溢出**与**计算效率**两类核心问题。

#### 数值上溢（Overflow）

标准 Softmax 的定义为：

$$
	softmax(x_i) = \frac{e^{x_i}}{\sum_{j=1}^{n} e^{x_j}}
$$


当输入 $x_i$ 较大时，$e^{x_i}$ 会超出浮点数的表示范围，产生 `inf`（无穷大），导致分子分母均为 `inf`，最终结果为 `NaN`。

以常用的混合精度训练格式 FP16 为例：
- FP16 的最大可表示值约为 $65504$；
- 当 $x > \ln(65504) \approx 11.09$ 时，$e^x > 65504$，即发生上溢出；
- 而 Attention 的 logits $QK^\top / \sqrt{d_k}$ 在未归一化之前数值范围难以保证，极易触发此问题。

#### 数值下溢（Underflow）

当输入 $x_i$ 极小（如大负数）时，$e^{x_i} \approx 0$，若所有输入均为大负数，分母趋近于零，导致除零错误或精度损失。

#### 三次遍历带来的内存带宽瓶颈

原生 Softmax 需要对输入数据进行**三次遍历**（Pass）：
1. **第一次**：遍历所有输入，求最大值 $m = \max_i x_i$（用于数值稳定）；
2. **第二次**：遍历所有输入，计算 $\sum_j e^{x_j - m}$（分母）；
3. **第三次**：遍历所有输入，计算每个 $e^{x_i - m} / \sum_j e^{x_j - m}$（归一化）。

每次遍历都涉及从 GPU HBM（高带宽内存）读取完整数据，内存带宽成为瓶颈，在长序列场景下尤为突出。

#### 与注意力计算结合时的中间矩阵显存问题

在标准注意力中，Softmax 需要先将完整的注意力分数矩阵 $S \in \mathbb{R}^{n \times n}$ 写入 HBM，再读取进行归一化，产生 $O(n^2)$ 的额外显存占用，对长序列（如 $n = 100K$）来说几乎不可承受。

</details>

**3. 怎么解决原生 softmax 的问题？**

<details>
<summary>💡 查看答案</summary>

针对数值溢出问题，标准解决方案是**减去最大值的数值稳定 Softmax**；针对效率问题，则采用**在线（Online）流式计算**策略。

#### 数值稳定 Softmax

利用 Softmax 的平移不变性：

$$
	softmax(x_i) = \frac{e^{x_i - m}}{\sum_j e^{x_j - m}}, \quad m = \max_k x_k
$$


减去最大值 $m$ 后，指数的最大值为 $e^0 = 1$，彻底避免上溢出。同时，由于 $x_j - m \leq 0$，所有指数值均在 $(0, 1]$ 范围内，分母至少为 1，也不会发生下溢导致的除零问题。

#### Safe Softmax 的完整实现

\begin{lstlisting}[style=promptstyle]
	def safe_softmax(x):
	m = x.max(dim=-1, keepdim=True).values   # 求最大值
	x_shifted = x - m                          # 平移，防溢出
	exp_x = torch.exp(x_shifted)              # 指数
	return exp_x / exp_x.sum(dim=-1, keepdim=True)  # 归一化
\end{lstlisting}

#### Online Softmax（流式两遍计算）

进一步地，可以将三遍遍历压缩为两遍，在第二遍遍历中同步更新最大值与累加和。具体地，维护两个统计量 $m$（当前最大值）和 $d$（归一化分母），按如下递推更新：


$$
	m_j = \max(m_{j-1},\; x_j)
$$


$$
	d_j = d_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j}
$$


当所有元素遍历完毕后，$m_n = \max_k x_k$，$d_n = \sum_k e^{x_k - m_n}$，即可一次性完成归一化。这一思想是 FlashAttention 实现 one-pass 注意力计算的数学基础。

</details>

**4. 怎么加快 softmax 计算过程？**

<details>
<summary>💡 查看答案</summary>

加速 Softmax 的核心思路是减少对全局内存（HBM）的访问次数，将多次遍历合并，利用**流式（Streaming）在线计算**在单次或两次遍历中完成所有统计量的计算。

#### Online Softmax 的形式化推导

设输入序列 $\mathbf{x} = (x_1, x_2, \ldots, x_n)$，定义前 $j$ 步的统计量：

$$
	m_j &= \max(x_1, \ldots, x_j) \\
	d_j &= \sum_{i=1}^{j} e^{x_i - m_j}
$$


递推关系：

$$
	m_j &= \max(m_{j-1},\; x_j) \\
	d_j &= d_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j}
$$


当遍历完所有元素后，第 $i$ 个 token 的 Softmax 值为：

$$
	p_i = \frac{e^{x_i - m_n}}{d_n}
$$


#### 与 Value 矩阵的联合流式计算

进一步地，可以将 Softmax 与后续的 $AV$ 乘法**合并**为一次遍历。维护累加统计量 $\mathbf{o}_j$（Value 的加权累加）：

$$
	\mathbf{o}_j = \mathbf{o}_{j-1} \cdot e^{m_{j-1} - m_j} + e^{x_j - m_j} \cdot V_j
$$


最终输出为 $\mathbf{o}_n / d_n$，实现了在**一次数据读取**中完成 Softmax 与加权求和的全部计算，将三次 HBM 访问压缩为一次，这正是 FlashAttention 的核心数学基础。

#### 硬件层面的加速

- **SRAM 复用**：将数据分块加载到 GPU SRAM（片上缓存），在 SRAM 中完成流式计算，避免中间结果反复写回 HBM；
- **Warp-level Reduction**：利用 GPU 的 warp 内规约指令（如 `__shfl_down_sync`）高效完成跨线程的最大值和累加操作；
- **向量化访存**：使用 128-bit 向量化内存指令（`float4`）提升内存带宽利用率。

</details>

**5. Flash Attention 的基本思想是什么？**

<details>
<summary>💡 查看答案</summary>

FlashAttention 是一种**IO 感知**（IO-Aware）的精确注意力算法，其核心目标不是减少 FLOPs，而是通过最小化 HBM 访问次数来提升实际运行速度并降低显存占用。

#### 动机：内存带宽是瓶颈而非算力

标准注意力的瓶颈在于内存访问而非计算：
- 注意力矩阵 $S, A \in \mathbb{R}^{n \times n}$ 需要写入/读取 HBM，产生 $O(n^2)$ 的 IO 开销；
- GPU 的 HBM 带宽（约 2 TB/s）远低于 SRAM 带宽（约 20 TB/s），HBM 读写成为主要延迟来源；
- 对于 $n = 2048$、$d = 64$，注意力矩阵占用约 512 MB，远超 SRAM 容量（约 20 MB）。

#### 核心思想：分块计算 + 在线 Softmax

FlashAttention 将 $Q, K, V$ 矩阵分割成大小为 $B_r \times d$ 和 $B_c \times d$ 的小块（Tile），使得每个 Tile 能够完整装入 SRAM。对于每个 $Q$ 块，遍历所有 $K, V$ 块，利用 Online Softmax 维护数值稳定的统计量，在 SRAM 内完成注意力计算，**无需将中间注意力矩阵写回 HBM**：


$$
	HBM IO（标准注意力） = O(nd + n^2)
$$


$$
	HBM IO（FlashAttention） = O(n^2 d^2 / M)
$$


其中 $M$ 为 SRAM 大小。当 $M \gg d^2$ 时，FlashAttention 的 IO 复杂度远小于标准注意力。

#### 算法流程概述

1. 将 $Q$ 分为 $T_r = \lceil n / B_r \rceil$ 个块，$K, V$ 分为 $T_c = \lceil n / B_c \rceil$ 个块；
2. 外层循环遍历 $K, V$ 块，将其加载到 SRAM；
3. 内层循环遍历 $Q$ 块，在 SRAM 中计算局部注意力分数，并用 Online Softmax 更新全局统计量 $(m, d, \mathbf{o})$；
4. 所有块遍历完毕后，将最终输出 $\mathbf{o}/d$ 写回 HBM。

#### 反向传播的 Recomputation 策略

标准反向传播需要存储前向计算的注意力矩阵 $A$（$O(n^2)$ 显存）。FlashAttention 在反向传播时**重新计算**注意力矩阵（只需存储 $m, d$ 统计量，$O(n)$ 显存），以额外的计算换取显存的大幅节省。

#### 性能收益

- **速度**：相比 PyTorch 标准实现，FlashAttention 在 A100 GPU 上速度提升 2--4$\times$；
- **显存**：注意力层显存从 $O(n^2)$ 降至 $O(n)$，支持更长序列训练；
- **精度**：数学上与标准注意力完全等价，无近似误差。

![标准 Attention 与 Flash Attention 显存频繁读写瓶颈及 SRAM 分块缓存机制对比图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_flash_attention.png)

</details>

**6. 为什么 Flash Attention 可以实现 one-pass 呢？**

<details>
<summary>💡 查看答案</summary>

FlashAttention 能够实现 one-pass 计算的根本原因在于：**Self-Attention 的最终输出可以表示为若干局部统计量的递推累加**，而这些统计量在每步读入新数据时可以被在线校正，无需预先得知全局信息。

#### 数学推导

设 Query $q \in \mathbb{R}^{d}$，Keys $K = [k_1, \ldots, k_n]^\top$，Values $V = [v_1, \ldots, v_n]^\top$，注意力输出为：

$$
	o = \sum_{i=1}^{n} \frac{e^{q \cdot k_i / \sqrt{d}}}{\sum_j e^{q \cdot k_j / \sqrt{d}}} v_i
$$


定义 $s_i = q \cdot k_i / \sqrt{d}$，则：

$$
	o = \frac{\sum_{i=1}^n e^{s_i} v_i}{\sum_{i=1}^n e^{s_i}}
$$


**关键观察**：分子 $\sum e^{s_i} v_i$ 和分母 $\sum e^{s_i}$ 均为**可分解的累加和**，支持流式计算。

维护三个统计量：

$$
	m_j &= \max(s_1, \ldots, s_j) & &（当前最大值，用于数值稳定）\\
	d_j &= \sum_{i=1}^j e^{s_i - m_j} & &（归一化分母）\\
	\mathbf{o}_j &= \sum_{i=1}^j e^{s_i - m_j} v_i & &（加权 Value 累加）
$$


当第 $j+1$ 步读入新数据 $(s_{j+1}, v_{j+1})$ 时，更新规则为：

$$
	m_{j+1} &= \max(m_j,\; s_{j+1}) \\
	d_{j+1} &= d_j \cdot e^{m_j - m_{j+1}} + e^{s_{j+1} - m_{j+1}} \\
	\mathbf{o}_{j+1} &= \mathbf{o}_j \cdot e^{m_j - m_{j+1}} + e^{s_{j+1} - m_{j+1}} \cdot v_{j+1}
$$


校正因子 $e^{m_j - m_{j+1}}$ 将旧统计量从旧最大值基准校正到新最大值基准，保证数值一致性。

#### 最终输出

遍历完所有 $n$ 个 Key-Value 对后，输出为：

$$
	o = \frac{\mathbf{o}_n}{d_n}
$$


整个过程**仅需一次数据遍历**，每步只需 $O(1)$ 的额外存储（$m, d, \mathbf{o}$），无需将完整注意力矩阵 $A \in \mathbb{R}^{n \times n}$ 实例化到内存中。

#### 分块并行的推广

在实际 GPU 实现中，$K, V$ 被分为多个块，每个块在 SRAM 内完成局部计算，块间通过上述递推公式合并统计量。这正是 FlashAttention Tiling 策略的数学基础。

</details>

**7. 解释下 Flash Attention 中的 tiling 策略？**

<details>
<summary>💡 查看答案</summary>

Tiling（分块）是 FlashAttention 将大型注意力矩阵计算适配到有限 SRAM 容量的核心工程策略。

#### 动机

GPU 的存储层次结构如下：
- **HBM**（高带宽内存）：容量大（40--80 GB），带宽约 2 TB/s，延迟高；
- **SRAM**（L1 Cache / Shared Memory）：容量小（约 192 KB/SM），带宽约 19 TB/s，延迟低。

标准注意力需要在 HBM 中存储 $n \times n$ 的注意力矩阵，而 FlashAttention 的目标是**将所有中间计算限定在 SRAM 内**，从而将 HBM 访问次数从 $O(n^2)$ 降至 $O(n)$。

#### 块大小的选择

设 SRAM 容量为 $M$，Key/Value 块大小 $B_c$ 和 Query 块大小 $B_r$ 的选取需满足：

$$
	B_c \cdot d + B_r \cdot d + B_r \cdot B_c \leq M
$$


即一个 $Q$ 块、一个 $K$ 块、一个 $V$ 块以及对应的注意力分数块能同时装入 SRAM。通常取：

$$
	B_c = B_r = \left\lfloor \frac{M}{4d} \right\rfloor
$$


#### 算法细节

外层循环（$K, V$ 方向，顺序遍历）：
1. 从 HBM 加载第 $j$ 块 $K_j \in \mathbb{R}^{B_c \times d}$ 和 $V_j \in \mathbb{R}^{B_c \times d}$ 到 SRAM；
2. 内层循环（$Q$ 方向，可并行）：
    1. 从 HBM 加载第 $i$ 块 $Q_i \in \mathbb{R}^{B_r \times d}$ 及其当前统计量 $m_i, d_i, \mathbf{o}_i$；
    2. 在 SRAM 内计算局部注意力分数 $S_{ij} = Q_i K_j^\top / \sqrt{d} \in \mathbb{R}^{B_r \times B_c}$；
    3. 用 Online Softmax 更新 $m_i, d_i, \mathbf{o}_i$；
    4. 将更新后的统计量写回 HBM（仅写 $O(B_r \cdot d)$ 大小，远小于 $O(n \cdot n)$）。

#### Tiling 的 IO 分析

- 每个 $K, V$ 块被读取 $T_r = n / B_r$ 次，共读取 $T_c \times T_r \times 2 B_c d$ 个元素；
- 总 HBM 读写量为 $O(n^2 d / M)$，与标准注意力的 $O(nd + n^2)$ 相比，在 $M \gg d$ 时有显著优势；
- **注意力矩阵从不写入 HBM**，彻底消除了 $O(n^2)$ 的显存瓶颈。

#### 因果掩码（Causal Mask）的处理

对于解码器自注意力（Causal Self-Attention），Tiling 只需在计算 $S_{ij}$ 时对块内的上三角部分置为 $-\infty$（即令对应位置的注意力权重为 0），无需修改整体 Tiling 框架，实现简洁高效。

</details>

**8. 在 Flash Attention 中对 MQA/GQA 是如何处理呢？**

<details>
<summary>💡 查看答案</summary>

MQA（Multi-Query Attention）和 GQA（Grouped-Query Attention）是减少 KV 缓存、降低推理内存带宽消耗的主流技术，FlashAttention 通过**索引（Indexing）机制**对其提供原生支持。

#### MQA 与 GQA 回顾

- **MHA**（Multi-Head Attention）：每个 Query Head 对应独立的 K、V Head，共 $h$ 组；
- **MQA**（Multi-Query Attention）：所有 Query Head 共享同一组 K、V，KV 参数量减少至 $1/h$；
- **GQA**（Grouped-Query Attention）：$h$ 个 Query Head 分为 $g$ 组，每组共享一个 KV Head（$1 \leq g \leq h$），是 MHA 与 MQA 的折中方案。

#### FlashAttention 中的索引处理

在 FlashAttention 的 CUDA Kernel 实现中，通过向 Kernel 传入 **KV Head 索引**来处理 MQA/GQA，无需复制 KV 数据：

\begin{lstlisting}[style=promptstyle]
	# GQA 的 KV Head 索引计算（伪代码）
	num_q_heads = 32
	num_kv_heads = 8         # GQA: 每 4 个 Q Head 共享 1 个 KV Head
	group_size = num_q_heads // num_kv_heads  # = 4

	# 对第 q_head_idx 个 Query Head，找到对应的 KV Head
	kv_head_idx = q_head_idx // group_size

	# 在 Kernel 中，根据 kv_head_idx 计算 K、V 的内存偏移地址
	k_ptr = K_base + kv_head_idx * kv_head_stride
	v_ptr = V_base + kv_head_idx * kv_head_stride
\end{lstlisting}

#### 具体实现原理

1. **启动配置**：Kernel 按 Query Head 数量 $h_q$ 启动，每个线程块负责一个 Q Head 的注意力计算；

2. **KV Head 映射**：在 Kernel 内部，通过整除运算 `kv_head_idx = q_head_idx // group_size` 确定当前 Q Head 对应的 KV Head；

3. **内存地址计算**：根据 KV Head 索引直接计算 $K, V$ 张量的内存偏移，**无需在内存中复制或广播 KV 数据**，节省内存带宽与显存；

4. **Tiling 兼容性**：KV 的分块加载按 KV Head 对应的实际数据进行，多个 Q Head 可以**并发读取同一 KV 块**（通过 L2 Cache 命中），进一步提升缓存效率。

#### 性能优势

- MQA/GQA 使 KV Cache 大小从 $O(n \cdot h \cdot d)$ 降至 $O(n \cdot g \cdot d)$，显著减少推理时的内存带宽消耗；
- FlashAttention 的索引机制避免了 KV 数据的冗余存储与复制，相比朴素的"先 Broadcast 再计算"方式节省 $(h/g - 1) \times$ 的 KV 内存；
- GQA 是 LLaMA 3、Mistral、Gemma 等主流模型的标准配置，FlashAttention 对其的高效支持是实际部署中的重要工程基础。

</details>

**9. 为什么只有 KV cache 而没有 Q cache 呢？**

<details>
<summary>💡 查看答案</summary>

KV Cache 与 Q Cache 的不对称性源于自回归解码的**时序特性**：Query 是动态变化的，每步都不同，而历史 Key、Value 在后续步骤中可以复用。

#### 自回归解码的工作机制

自回归语言模型按如下方式逐 token 解码：

$$
	x_{t+1} \sim P(\cdot \mid x_1, x_2, \ldots, x_t)
$$


在第 $t$ 步，模型需要计算：

$$
	Attention_t = softmax\!\left(\frac{q_t [K_1, K_2, \ldots, K_t]^\top}{\sqrt{d_k}}\right) [V_1, V_2, \ldots, V_t]^\top
$$


#### 为什么需要 KV Cache？

若没有 KV Cache，每一步解码都需要对**所有历史 token** 重新计算 $K, V$，计算复杂度为 $O(t \cdot d^2)$（QKV 投影），总解码复杂度为 $O(n^2 d^2)$，随序列增长呈平方级增加。

注意到在第 $t$ 步，历史 token $x_1, \ldots, x_{t-1}$ 对应的 $K, V$ 与上一步**完全相同**（因为它们的输入 token 没有改变），因此可以缓存：

$$
	K_i = x_i W^K, \quad V_i = x_i W^V \quad (i < t,  不变)
$$


KV Cache 将每步解码的复杂度从 $O(t \cdot d^2)$ 降至 $O(d^2)$（仅计算新 token 的 $K, V$），总解码复杂度从 $O(n^2 d^2)$ 降至 $O(n d^2)$。

#### 为什么不需要 Q Cache？

Query 向量在每步解码中仅由**当前新生成的 token** 产生：

$$
	q_t = x_t W^Q
$$


在自回归解码中，每一步只有一个新 token，因此 $q_t$ 是一个形状为 $1 \times d_k$ 的向量，**计算成本极低**（$O(d^2)$），缓存意义不大。

此外，$q_t$ 仅在第 $t$ 步使用，在下一步（$t+1$ 步）中，新的 Query $q_{t+1}$ 由 $x_{t+1}$ 产生，与 $q_t$ 完全无关，缓存 $q_t$ 不能为后续步骤提供任何复用价值。

#### 对比总结

\begin{table}[h]



| **属性** | **KV（Key/Value）** | **Q（Query）** |
| --- | --- | --- |
| 来源 | 历史 token | 当前新生成 token |
| 跨步复用性 | 完全可复用（不变） | 不可复用（每步不同） |
| 缓存收益 | 高（避免 $O(n)$ 重算） | 无（每步仅 $O(1)$ 计算） |
| 显存开销 | $O(n \cdot h \cdot d)$ | $O(h \cdot d)$（当步） |


	\caption{KV Cache 与 Q Cache 的对比分析}
\end{table}

#### KV Cache 的工程权衡

KV Cache 虽然大幅加速解码，但也带来显著的显存压力。以 LLaMA-2 70B 为例，在 FP16 精度下，KV Cache 大小约为：

$$
	KV Cache = 2 \times n \times L \times h \times d_k \times 2  bytes
$$

其中 $n$ 为序列长度，$L$ 为层数，$h$ 为头数，$d_k$ 为每头维度。这正是 MQA/GQA、量化 KV Cache、PagedAttention 等技术的设计动机。

![自回归解码中 KV Cache 动态拼接缓存机制与 QKV 计算原理图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_kv_cache.png)

</details>

**10. 1B 的模型和 1T 的数据大概要训练多久？**

<details>
<summary>💡 查看答案</summary>

训练时间的估算依赖于模型规模、数据量、硬件算力以及系统效率等多个因素，可以通过以下公式系统推导。

#### 训练时间估算公式


$$
	T_{train} = \frac{C_{total}}{N_{GPU} \times F_{GPU} \times \eta}
$$


其中：
- $C_{total}$：总计算量（FLOPs）；
- $N_{GPU}$：GPU 数量；
- $F_{GPU}$：单卡理论峰值算力（FLOPs/s）；
- $\eta$：GPU 利用率（MFU，Model FLOPs Utilization），工程实践中通常为 0.3--0.5。

#### 总计算量 $C_{total$ 的推导}

训练一个参数量为 $N$ 的模型，处理 $D$ 个 token 的总 FLOPs 为：

$$
	C_{total} = 6 N D
$$


系数 6 的来源：
- 前向传播：每个参数参与 1 次乘法 + 1 次加法，约 $2N$ FLOPs/token；
- 反向传播：梯度计算约为前向的 2 倍，即 $4N$ FLOPs/token；
- 合计：$2N + 4N = 6N$ FLOPs/token。

若启用**激活重计算**（Gradient Checkpointing），前向传播执行两次，总 FLOPs 变为 $8ND$，以额外计算换取显存节省。

#### 数值代入（单 GPU 场景）

取 $N = 10^9$（1B 参数），$D = 10^{12}$（1T token），使用 NVIDIA A100 80GB（BF16 峰值算力约 $312 \times 10^{12}$ FLOPs/s），$\eta = 0.3$，启用激活重计算（系数取 8）：


$$
	C_{total} &= 8 \times 10^9 \times 10^{12} = 8 \times 10^{21}  FLOPs \\
	T &= \frac{8 \times 10^{21}}{1 \times 312 \times 10^{12} \times 0.3} \\
	&= \frac{8 \times 10^{21}}{9.36 \times 10^{13}} \approx 8.55 \times 10^7  秒 \approx 990  天
$$


#### 多 GPU 集群场景

实际训练通常使用多卡并行。以 64 张 A100 为例（$\eta = 0.4$）：

$$
	T &= \frac{8 \times 10^{21}}{64 \times 312 \times 10^{12} \times 0.4} \approx \frac{8 \times 10^{21}}{7.99 \times 10^{15}} \approx 10^6  秒 \approx 11.6  天
$$


#### Chinchilla 最优性检验

根据 Chinchilla 法则，1B 参数模型的最优训练 token 数约为 $20N = 20 \times 10^9 = 20B$ tokens，而非 1T tokens。1T token 相对于 1B 模型是严重**过训练**（over-training）的，从计算效率角度并不最优；但若追求推理部署效率（使小模型在推理时性能更强），适当过训练是合理的（如 LLaMA 系列的策略）。

#### 小结

\begin{table}[h]



| **场景** | **GPU 数量** | **MFU** | **估算时间** |
| --- | --- | --- | --- |
| 单卡 A100 | 1 | 0.30 | $\approx$ 990 天 |
| 8卡 A100 | 8 | 0.35 | $\approx$ 108 天 |
| 64卡 A100 | 64 | 0.40 | $\approx$ 12 天 |
| 512卡 A100 | 512 | 0.45 | $\approx$ 1.3 天 |


	\caption{不同集群规模下 1B 模型 + 1T token 的训练时间估算}
\end{table}

</details>

**11. 大模型中为啥使用 RMSNorm 而不是 LayerNorm?**

<details>
<summary>💡 查看答案</summary>

RMSNorm（Root Mean Square Layer Normalization）是 LayerNorm 的简化变体，在现代大语言模型（LLaMA、Mistral、Gemma、Qwen 等）中几乎已成为标配，其优势体现在计算效率、训练稳定性和理论合理性三个层面。

#### LayerNorm 回顾

LayerNorm 对每个样本的特征维度进行归一化，计算公式为：

$$
	LayerNorm(x_i) = \gamma \cdot \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
$$

其中：

$$
	\mu = \frac{1}{d} \sum_{i=1}^d x_i, \quad \sigma^2 = \frac{1}{d} \sum_{i=1}^d (x_i - \mu)^2
$$

涉及**均值**和**方差**两步统计量计算，以及可学习的平移参数 $\beta$（bias）。

#### RMSNorm 的定义

RMSNorm 去除了均值中心化步骤，仅利用均方根（RMS）进行缩放：

$$
	RMSNorm(x_i) = \gamma \cdot \frac{x_i}{RMS(\mathbf{x})}, \quad RMS(\mathbf{x}) = \sqrt{\frac{1}{d} \sum_{i=1}^d x_i^2 + \epsilon}
$$


相比 LayerNorm，RMSNorm **去除了均值计算**和**平移参数** $\beta$，仅保留缩放参数 $\gamma$。

#### 计算效率优势

- LayerNorm 需要两次遍历：第一次计算均值 $\mu$，第二次计算方差 $\sigma^2$（需要已知 $\mu$）；
- RMSNorm 仅需**一次遍历**：直接累加 $x_i^2$，计算 $RMS$ 后归一化；
- 计算量约减少 $\mathbf{7%--15%}$，在深层网络中（如 80 层的 LLaMA-2 70B）累积效果显著；
- 减少了 $d$ 个可学习参数（$\beta$ 向量），轻微降低模型参数量。

#### 理论依据：重缩放不变性

Zhang & Sennrich（2019）的研究表明，LayerNorm 的性能提升主要来源于**重缩放不变性**（rescaling invariance），而非均值中心化。具体地，对于输入 $\mathbf{x}$ 和缩放因子 $\alpha > 0$：

$$
	RMSNorm(\alpha \mathbf{x}) = \alpha \cdot RMSNorm(\mathbf{x}) \cdot \frac{1}{\alpha} = RMSNorm(\mathbf{x})
$$

即输出对输入的缩放不敏感，具有**尺度不变性**，有助于稳定深层网络的训练。均值中心化对最终性能的贡献相对有限，去除后实验表现差异不显著。

#### 训练稳定性

- **梯度稳定**：RMSNorm 的梯度形式更简洁，在深层网络（如 100+ 层）中有助于缓解梯度消失/爆炸问题；
- **隐式学习率自适应**：通过对激活值的幅度归一化，RMSNorm 使得不同层的学习率需求趋于一致，减少对精细学习率调节的依赖；
- **Pre-Norm 结构下的优势**：现代大模型普遍采用 Pre-Norm（在残差连接之前归一化），RMSNorm 在此结构下与 LayerNorm 性能相当，但计算更轻量。

#### 实证对比

\begin{table}[h]



| **属性** | **LayerNorm** | **RMSNorm** |
| --- | --- | --- |
| 计算步骤 | 均值 + 方差 | 仅均方根 |
| 可学习参数 | $\gamma, \beta$ | 仅 $\gamma$ |
| 遍历次数 | 2 次 | 1 次 |
| 计算开销 | 基准 | 约减少 10% |
| 下游性能 | 相当 | 相当（略优于深层） |
| 代表模型 | BERT、GPT-2 | LLaMA、Mistral、Qwen |


	\caption{LayerNorm 与 RMSNorm 的对比}
\end{table}

综上，RMSNorm 在**保持与 LayerNorm 相近性能的前提下**，提供了更低的计算开销和更简洁的实现，是大规模训练场景下的工程优选。

</details>

**12. 大模型训练过程中如何估计显卡利用率？**

<details>
<summary>💡 查看答案</summary>

显卡利用率（GPU Utilization / MFU，Model FLOPs Utilization）是衡量训练系统效率的关键指标，直接反映了硬件算力的实际使用效率。

1. **GPU 计算效率评估（FLOPs 基准法）**

    最直接的方法：通过统计模型实际执行的 FLOPs 与硬件理论峰值的比值来衡量利用率。


    $$
    MFU = \frac{实测 FLOPs/s}{F_{GPU,peak}}
    $$


    实测 FLOPs/s 可通过以下方式获取：

    $$
    实测 FLOPs/s = \frac{6ND}{T_{actual} \times N_{GPU}}
    $$

    其中 $N$ 为参数量，$D$ 为已处理 token 数，$T_{actual}$ 为实际耗时，$N_{GPU}$ 为 GPU 数量。

    实践中，A100 集群的 MFU 通常在 0.35--0.50 之间；低于 0.3 则意味着存在明显的通信、数据加载或 IO 瓶颈。

2. **吞吐量对比评估法**

    通过将实际训练吞吐量（tokens/s）与文献中相同配置的标称吞吐量对比，间接估算利用率：

    $$
    相对利用率 = \frac{实际吞吐量（tokens/s）}{文献标称吞吐量（tokens/s）}
    $$


    优点是无需精确统计 FLOPs，实现简单；缺点是受文献测试条件差异影响，参考价值有限。实践中可使用 `tokens/s/GPU` 作为归一化指标，便于跨硬件比较。

3. **运行时性能剖析法（Profiling）**

    使用专业工具进行细粒度性能分析，定位具体瓶颈：

    - **PyTorch Profiler**：记录每个算子的 CPU/GPU 执行时间、内存占用、CUDA kernel 调用情况：
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

    - **NVIDIA Nsight Systems / Nsight Compute**：提供 GPU kernel 级别的时间线分析，可识别计算空泡（bubble）、通信等待、内存带宽瓶颈等问题；

    - **nvidia-smi**：实时监控 GPU 利用率（SM 占用率）、显存使用量、功耗等硬件指标（注意：`nvidia-smi` 报告的利用率是 SM 活跃时间比例，与 MFU 并不等价）；

    - **DCGM（Data Center GPU Manager）**：适用于大规模集群的 GPU 健康状态监控与指标采集。

</details>

**13. 大模型训练中如何优化显卡利用率？**

<details>
<summary>💡 查看答案</summary>

提升 GPU 利用率需要从数据流、模型计算、通信开销等多个维度系统优化，常见瓶颈与对应解决方案如下。

1. **降低日志输出频率**

    频繁的日志打印（如每步打印 loss、梯度范数）会触发 CPU-GPU 同步（`torch.cuda.synchronize()`），阻塞 GPU 计算流水线。建议：
    - 将日志频率从每步降低至每 10--100 步；
    - 使用异步日志（`loss.item()` 会触发同步，可延迟调用）；
    - 将监控指标通过 TensorBoard 异步写入，避免主训练循环阻塞。

2. **使用性能分析工具定位瓶颈**

    在优化前，必须先通过 Profiler 确认瓶颈位置（计算、通信、IO 还是 CPU），避免盲目优化。常见瓶颈模式：
    - GPU 计算时间占比低 → 数据加载或通信瓶颈；
    - `all-reduce` 占比高 → 通信带宽不足，考虑梯度压缩或重叠通信；
    - 大量 `cudaMemcpyAsync` → Host-Device 数据传输过多。

3. **提升数据加载效率**

    数据加载是常见瓶颈，尤其在高 GPU 算力场景下。优化措施：
    - **增加 DataLoader 线程数**：`num_workers` 设为 CPU 核数的 1/4--1/2；
    - **启用内存映射（Memory-mapped）数据集**：使用 `.bin` 或 Arrow 格式，避免重复磁盘 IO；
    - **数据预缓存与预取**：将下一批次数据提前加载到 CPU 内存，与 GPU 计算重叠；
    - **使用更快的存储设备**：NVMe SSD 替代 HDD，或将数据集存放于本地 SSD 而非网络文件系统（NFS）；
    - **pin_memory=True**：启用固定内存，加速 CPU → GPU 数据传输（`cudaMemcpyHostToDevice`）。

4. **优化模型架构与算子**

    - 使用 **FlashAttention** 替换标准注意力，减少 HBM 带宽消耗；
    - 使用 **torch.compile**（PyTorch 2.0+）对模型进行图编译优化，减少 kernel 启动开销和内存分配；
    - 使用 **Fused Kernels**（如 fused layernorm、fused softmax、fused Adam）减少独立 kernel 启动次数；
    - 合理设置 `d_ff`、`d_model` 等维度为 128 的倍数，充分利用 Tensor Core。

5. **调整 Batch Size**

    - 增大 batch size 可提升 GPU 计算密度（arithmetic intensity），减少相对通信开销，但受显存限制；
    - 使用**梯度累积**（Gradient Accumulation）在有限显存下模拟大 batch：

        $$
        有效 batch size = micro batch size \times 梯度累积步数 \times N_{GPU}
        $$

    - 过大的 batch size 可能影响模型收敛质量（泛化性下降），需配合学习率线性缩放（Linear Scaling Rule）调整。

6. **调整学习率和优化器**

    - 使用 **混合精度训练**（BF16/FP16 + FP32 master weights）：在不损失精度的前提下将显存减半，允许更大 batch size，同时 Tensor Core 在低精度下算力更高；
    - 使用 **8-bit Adam**（如 bitsandbytes 库）：将优化器状态量化为 8-bit，将优化器显存从 $12N$ bytes（FP32 Adam）降至约 $6N$ bytes，释放更多显存用于增大 batch；
    - 启用**学习率预热（Warmup）+ 余弦退火**调度，稳定大 batch 训练初期的梯度更新；
    - 对于多机训练，合理设置 **ZeRO 优化级别**（DeepSpeed ZeRO-1/2/3）以平衡通信量与显存节省。

</details>

**14. 大模型的显存和什么有影响？**

<details>
<summary>💡 查看答案</summary>

大模型训练的显存消耗由多个部分构成，可以分为**模型状态**（Model States）和**激活值**（Activations）两大类，其中模型状态是主要占用来源。

#### 显存消耗的构成分析

以混合精度训练（FP16 + FP32 master weights）+ Adam 优化器为例，参数量为 $N$ 的模型：

\begin{table}[h]



| **组成部分** | **精度** | **显存大小** |
| --- | --- | --- |
| 模型参数（前向/反向） | FP16 | $2N$ bytes |
| 梯度 | FP16 | $2N$ bytes |
| Master weights（优化器） | FP32 | $4N$ bytes |
| Adam 一阶矩（$m$） | FP32 | $4N$ bytes |
| Adam 二阶矩（$v$） | FP32 | $4N$ bytes |
| **模型状态合计** |  | $\mathbf{16N}$ bytes |
| 激活值（与 batch/序列长度相关） | FP16/BF16 | 依配置而定 |


	\caption{混合精度训练 + Adam 的显存构成（$N$ 为参数量）}
\end{table}

例如，70B 参数模型的模型状态显存约为 $16 \times 70 \times 10^9 \approx 1120$ GB，需要至少 14 张 80GB A100。

#### 各影响因素详细分析

1. **模型参数量**：最基础的显存来源，直接决定模型状态的下界。参数量与显存成**线性关系**，是显存规划的首要考虑因素。

2. **Batch Size**：
    - 激活值（中间层输出）的显存随 batch size **线性增长**；
    - 模型状态（参数、梯度、优化器状态）与 batch size 无关；
    - 可通过梯度累积在不增加单步显存的前提下增大有效 batch size。

3. **序列长度**：
    - 激活值显存随序列长度**线性增长**（FFN 部分）；
    - 注意力矩阵显存随序列长度**平方增长**（$O(n^2)$），是长序列训练的主要瓶颈；
    - FlashAttention 将注意力层显存从 $O(n^2)$ 降至 $O(n)$，显著缓解此问题。

4. **模型结构**：
    - **层数**：更深的网络激活值显存更大（需存储每层的前向输出用于反向传播）；
    - **隐藏维度**：$d_{model}$ 增大使注意力和 FFN 的中间张量更大；
    - **MHA vs MQA/GQA**：MQA 将 KV 头数降至 1，GQA 降至 $g < h$，KV Cache 显存减少 $h/g$ 倍；
    - **MoE 架构**：参数量大但每次前向只激活少数专家，激活值显存相对较小。

5. **优化器选择**：
    - **SGD**：仅需存储梯度，模型状态 = $2N$（FP16 参数）$+ 4N$（FP32 master）$+ 4N$（梯度）= $10N$ bytes；
    - **Adam**：额外存储一阶矩和二阶矩，模型状态 = $16N$ bytes；
    - **8-bit Adam**：量化优化器状态，模型状态约 $\approx 6N$ bytes；
    - **Adafactor**：不存储二阶矩矩阵，仅存储行/列因子分解，显存接近 SGD。

6. **混合精度训练**：
    - 使用 FP16/BF16 训练，计算时的参数和梯度各占 $2N$ bytes；
    - 相比纯 FP32 训练（$4N + 4N = 8N$ bytes）节省约 25% 的参数/梯度显存；
    - BF16 相比 FP16 动态范围更大，训练更稳定（现代 GPU 如 A100、H100 推荐使用 BF16）。

7. **激活重计算（Gradient Checkpointing）**：
    - 不使用激活重计算：需存储所有层的激活值，显存为 $O(L \cdot n \cdot d)$；
    - 使用激活重计算：仅存储检查点层的激活（如每隔 $k$ 层保存一次），显存降至 $O(\sqrt{L} \cdot n \cdot d)$，代价是增加约 1/3 的计算量。

#### 显存优化策略汇总

\begin{table}[h]



| **优化手段** | **显存节省来源** | **代价** |
| --- | --- | --- |
| 混合精度（BF16） | 参数/梯度减半 | 精度稍降（可接受） |
| 8-bit Adam | 优化器状态减半 | 略增计算量 |
| 激活重计算 | 激活值大幅减少 | 约增加 33% FLOPs |
| ZeRO-1/2/3 | 模型状态跨卡分片 | 通信量增加 |
| GQA/MQA | KV Cache 显存减少 | 轻微性能损失 |
| FlashAttention | 注意力层 $O(n^2) \to O(n)$ | 无精度损失 |


	\caption{常见显存优化手段对比}
\end{table}

</details>

**15. 什么是束搜索，它与贪婪解码有何不同？**

<details>
<summary>💡 查看答案</summary>

束搜索（Beam Search）和贪婪解码（Greedy Decoding）是序列生成中最常用的两类解码算法，两者在搜索空间探索策略和输出质量上存在本质差异。

#### 贪婪解码

贪婪解码在每一步选择**当前条件概率最大**的 token：

$$
	x_t^* = \arg\max_{w} P(w \mid x_1, \ldots, x_{t-1})
$$


- **优点**：实现简单，时间复杂度为 $O(n \cdot |V|)$（$|V|$ 为词表大小），速度极快；
- **缺点**：每步局部最优不等于全局最优，可能陷入次优解。例如，某步选择了概率稍低的 token，可能为后续步骤打开更高概率的路径，但贪婪解码无法回溯；
- **典型问题**：生成重复序列（repetition）、语句截断过早、语义不连贯。

#### 束搜索

束搜索在每一步维护 $k$（束宽，beam size）个**最优候选序列**，同时探索多条路径：

1. 初始化：从 $k$ 个得分最高的初始 token 出发，得到 $k$ 个候选序列；
2. 扩展：对每个候选序列，计算所有可能的下一个 token 的概率，得到 $k \times |V|$ 个扩展序列；
3. 剪枝：按**累积对数概率**排序，保留得分最高的 $k$ 个序列：

    $$
    score(x_1, \ldots, x_t) = \sum_{i=1}^{t} \log P(x_i \mid x_1, \ldots, x_{i-1})
    $$

4. 终止：当所有束均生成 `<EOS>` 或达到最大长度时停止，输出得分最高的序列。

- **时间复杂度**：$O(k \cdot n \cdot |V|)$，约为贪婪解码的 $k$ 倍；
- **空间复杂度**：$O(k \cdot n)$，需存储 $k$ 条候选序列；
- **优点**：相比贪婪解码探索更广的搜索空间，通常能找到更高累积概率的序列；
- **缺点**：仍非全局最优（完全搜索复杂度为 $O(|V|^n)$）；在生成多样性要求高的任务（如开放域对话）中，束搜索倾向于生成"安全"但单调的输出。

#### 长度惩罚

束搜索中，累积对数概率对较短序列有利（较少相乘），通常引入**长度惩罚**（Length Penalty）进行校正：

$$
	score_normalized(x_1, \ldots, x_t) = \frac{\sum_{i=1}^t \log P(x_i \mid x_{<i})}{t^\alpha}
$$

其中 $\alpha \in [0, 1]$ 为长度惩罚系数，$\alpha = 0$ 退化为无惩罚，$\alpha = 1$ 为完全归一化。

#### 对比总结

\begin{table}[h]



| **属性** | **贪婪解码** | **束搜索**（beam=$k$） |
| --- | --- | --- |
| 每步候选数 | 1 | $k$ |
| 搜索空间 | 最小 | 中等 |
| 输出质量 | 较低 | 较高 |
| 多样性 | 低 | 低（确定性） |
| 计算开销 | $O(n|V|)$ | $O(kn|V|)$ |
| 适用场景 | 实时推理 | 机器翻译、摘要 |


	\caption{贪婪解码与束搜索对比}
\end{table}

</details>

**16. Top-k 和核采样之间有什么区别？**

<details>
<summary>💡 查看答案</summary>

Top-$k$ 采样和核采样（Nucleus Sampling / Top-$p$ 采样）均是在语言模型解码中引入**随机性**、提升生成多样性的截断采样策略，两者的核心区别在于**截断集合的定义方式**。

#### Top-$k$ 采样

Top-$k$ 采样将候选集固定为概率最高的前 $k$ 个 token，对其重新归一化后采样：

$$
	P'(x_t = w) = \begin{cases} \dfrac{P(w \mid x_{<t})}{\sum_{w' \in \mathcal{V}_k} P(w' \mid x_{<t})} & w \in \mathcal{V}_k \\ 0 & otherwise \end{cases}
$$

其中 $\mathcal{V}_k$ 为概率最高的前 $k$ 个 token 的集合。

- **优点**：实现简单，计算高效（只需排序取 Top-$k$）；
- **缺点**：$k$ 为**固定值**，无法适应分布的动态变化。当概率分布**尖锐**（熵低）时，$k$ 个候选中可能包含大量低概率噪声 token；当分布**平坦**（熵高）时，$k$ 个候选可能遗漏大量合理选项。

#### 核采样（Top-$p$ / Nucleus Sampling）

核采样由 Holtzman 等人（2020）提出，动态选择满足累积概率不超过 $p$ 的最小候选集 $\mathcal{V}_p$：

$$
	\mathcal{V}_p = \arg\min_{\mathcal{V} \subseteq \mathcal{V}_{sorted}} \left\{ |\mathcal{V}| : \sum_{w \in \mathcal{V}} P(w \mid x_{<t}) \geq p \right\}
$$

即按概率从高到低排序，累积到概率之和首次超过 $p$ 时截断，对 $\mathcal{V}_p$ 重新归一化后采样。

- **优点**：候选集大小**随分布动态调整**。分布尖锐时 $|\mathcal{V}_p|$ 较小（聚焦于高概率词），分布平坦时 $|\mathcal{V}_p|$ 较大（覆盖更多合理词），有效避免了 Top-$k$ 的固定截断问题；
- **缺点**：计算需要对概率排序（$O(|V| \log |V|)$），略慢于 Top-$k$；$p$ 的选取（通常 0.9--0.95）仍需调参。

#### 直观对比示例

设词表大小为 5，两种情形下的概率分布：

\begin{table}[h]



|  | **token A** | **token B** | **token C** | **token D/E** |
| --- | --- | --- | --- | --- |
| 尖锐分布 | 0.90 | 0.07 | 0.02 | 0.005 |
| 平坦分布 | 0.25 | 0.22 | 0.20 | 0.17/0.16 |


	\caption{两种概率分布示例}
\end{table}

- **Top-$k$（$k=3$）**：两种情形均选 \{A, B, C\}，尖锐时包含低质量的 C（概率仅 0.02），平坦时遗漏了合理的 D、E；
- **Top-$p$（$p=0.95$）**：尖锐时 \{A, B\} 即已超过 0.95，仅 2 个候选；平坦时需 \{A, B, C, D, E\} 才超过 0.95，包含全部 5 个候选，适应性更好。

#### 实践中的组合使用

Top-$k$ 与 Top-$p$ 通常**联合使用**，先截断 Top-$k$，再在其中做 Top-$p$ 筛选，同时配合温度系数控制整体分布的锐利程度：

\begin{lstlisting}[style=promptstyle]
	# 典型推理参数配置
	{
		"temperature": 0.8,   # 控制分布锐利度
		"top_k": 50,          # 先限制最大候选数
		"top_p": 0.95,        # 再做累积概率截断
		"repetition_penalty": 1.1  # 抑制重复生成
	}
\end{lstlisting}

#### 对比总结

\begin{table}[h]



| **属性** | **Top-$k$ 采样** | **核采样（Top-$p$）** |
| --- | --- | --- |
| 截断方式 | 固定数量前 $k$ 个 | 动态累积概率阈值 $p$ |
| 候选集大小 | 固定（$= k$） | 动态（随分布变化） |
| 分布适应性 | 弱 | 强 |
| 实现复杂度 | 低（Top-$k$ 排序） | 中（全排序 + 累积） |
| 推荐超参 | $k = 40\sim100$ | $p = 0.9\sim0.95$ |


	\caption{Top-$k$ 与核采样（Top-$p$）的对比}
\end{table}

</details>

**17. 一次前向传播的 FLOPs 计算**

<details>
<summary>💡 查看答案</summary>

前向传播的 FLOPs 分析是评估模型计算效率、进行 Roofline 分析和训练资源规划的基础。以标准 Transformer（$L$ 层，隐藏维度 $d_{model}$，注意力头数 $h$，每头维度 $d_k = d_{model} / h$，FFN 维度 $d_{ff} = 4 d_{model}$，序列长度 $n$）为例进行逐层分析。

#### 基本计数规则

矩阵乘法 $A \in \mathbb{R}^{m \times k}$ 与 $B \in \mathbb{R}^{k \times n}$ 的 FLOPs 为 $2mkn$（每个输出元素需 $k$ 次乘法和 $k-1$ 次加法，近似为 $2k$）。

#### 单层 Transformer 的 FLOPs 拆解

**（1）QKV 投影**

将输入 $X \in \mathbb{R}^{n \times d_{model}}$ 分别投影为 $Q, K, V \in \mathbb{R}^{n \times d_{model}}$：

$$
	FLOPs_{QKV} = 3 \times 2 \times n \times d_{model} \times d_{model} = 6n d_{model}^2
$$


**（2）注意力计算**

- $QK^\top$ 矩阵乘：$2 \times n \times n \times d_{model} = 2n^2 d_{model}$ FLOPs
- Softmax 归一化：$O(n^2)$（远小于矩阵乘，通常忽略）
- $AV$ 矩阵乘：$2 \times n \times n \times d_{model} = 2n^2 d_{model}$ FLOPs

$$
	FLOPs_{Attn} = 4n^2 d_{model}
$$


**（3）输出投影**

将注意力输出 $\in \mathbb{R}^{n \times d_{model}}$ 投影回 $d_{model}$：

$$
	FLOPs_{out} = 2 \times n \times d_{model} \times d_{model} = 2n d_{model}^2
$$


**（4）FFN（两层 MLP）**


$$
	FLOPs_{FFN} = 2 \times 2 \times n \times d_{model} \times d_{ff} = 4n \cdot d_{model} \cdot d_{ff} = 16n d_{model}^2
$$


（取 $d_{ff} = 4 d_{model}$）

#### 单层总 FLOPs


$$
	FLOPs_{layer} = 6nd_{model}^2 + 4n^2 d_{model} + 2nd_{model}^2 + 16nd_{model}^2 = 24nd_{model}^2 + 4n^2 d_{model}
$$


#### 全模型前向 FLOPs

$L$ 层 Transformer 加上最终的 LM Head（$d_{model} \times |V|$）：

$$
	FLOPs_{forward} = L \left(24nd_{model}^2 + 4n^2 d_{model}\right) + 2n d_{model} |V|
$$


当 $n \ll d_{model}$（短序列）时，注意力项 $4n^2 d_{model}$ 可忽略，前向 FLOPs $\approx 2nN$（$N \approx 12 L d_{model}^2$ 为参数量）。

#### 训练总 FLOPs

反向传播 FLOPs 约为前向的 2 倍，训练总 FLOPs：

$$
	FLOPs_{train} \approx 6nN \times D_{total}
$$

其中 $D_{total}$ 为训练 token 总数。若启用激活重计算，前向执行两次，系数变为 8。

#### Roofline 分析：计算密集 vs 访存密集

对于每个算子，计算**算术强度**（Arithmetic Intensity）：

$$
	I = \frac{FLOPs}{访存字节数} \quad （FLOPs/Byte）
$$


- 若 $I > I_{ridge}$（硬件的脊点，A100 约为 200 FLOPs/Byte），则为**计算密集型**，性能受算力限制；
- 若 $I < I_{ridge}$，则为**访存密集型**，性能受内存带宽限制。

以 $n=2048, d_{model}=4096$ 为例：
\begin{table}[h]



| **算子** | **FLOPs** | **访存（Bytes）** | **算术强度** |
| --- | --- | --- | --- |
| QKV 投影（矩阵乘） | $6nd^2 \approx 2.06T$ | $\approx 0.10T$ | $\approx 206$（计算密集） |
| $QK^\top$（注意力） | $2n^2d \approx 34.4G$ | $\approx 0.27T$ | $\approx 128$（访存密集） |
| FFN（矩阵乘） | $16nd^2 \approx 5.5T$ | $\approx 0.27T$ | $\approx 204$（计算密集） |
| Softmax | $O(n^2)$ | $O(n^2 \cdot sizeof)$ | 低（访存密集） |


	\caption{各算子 Roofline 分析示例（A100，BF16）}
\end{table}

由此可见，注意力计算（$QK^\top$ 和 $AV$）在长序列下是访存密集型算子，这正是 FlashAttention 通过减少 HBM 访问量来显著加速的原因；而 FFN 和 QKV 投影是计算密集型算子，受算力限制，优化空间有限。

</details>

**18. Roofline Model是什么理论**

<details>
<summary>💡 查看答案</summary>

Roofline Model 用于描述硬件的理论性能上限，横轴是计算强度（Arithmetic Intensity, AI = FLOPs / Bytes），纵轴是吞吐性能（FLOPS）。

两条线：
- **水平线**：硬件峰值算力（Peak FLOPS）
- **斜线**：硬件带宽上限决定的性能（Bandwidth × AI）

判断方法：
- 计算任务的 AI = 该任务总FLOPs / 总访存字节数
- 若 AI < 拐点（峰值算力/带宽），则任务是 **memory-bound**（性能受带宽限制）
- 若 AI > 拐点，则任务是 **compute-bound**（性能受算力限制）

例如 A100 峰值约 312 TFLOPS(FP16)，带宽约 2TB/s，拐点 AI ≈ 156。GEMM类操作（大矩阵乘）通常 AI 高，是 compute-bound；elementwise操作（如LayerNorm、softmax、激活函数）AI 低，是 memory-bound。

</details>

**19. Llama-2-70B 在 A100 推理延迟估算**

<details>
<summary>💡 查看答案</summary>

Batch size=1 的 decode 阶段是典型的 **memory-bound** 场景，每生成一个 token 需要把全部参数从显存读取一次。

估算公式：
\begin{verbatim}
单token延迟 ≈ 模型参数量(字节) / 显存带宽
\end{verbatim}

具体计算：
- 70B 参数，FP16存储 = 70×$10^{9}$ × 2 bytes = 140 GB
- A100 显存带宽 ≈ 2 TB/s (A100 80GB SXM约2039GB/s)
- 单token延迟 ≈ 140GB / 2TB/s ≈ **70ms**

但70B模型本身超过单卡80GB显存（140GB > 80GB），实际需要多卡（如TP=2），此时带宽汇总但通信开销也需考虑。实际测试值通常在 30-50ms/token（多卡TP并行后单卡读取量减少，带宽利用提升）。

KV Cache的读写也会贡献一部分访存量，但参数读取占主导。

</details>

**20. Transformer中的计算瓶颈与访存瓶颈**

<details>
<summary>💡 查看答案</summary>

**Compute-bound（算力瓶颈）**：
- 大矩阵乘法 GEMM：QKV投影、输出投影、FFN的两个线性层
- Prefill阶段的Attention计算（长序列时$QK^{\top}$、AV的FLOPs很高）
- 这些操作的 AI 高，可以充分利用Tensor Core算力

**Memory-bound（访存瓶颈）**：
- LayerNorm/RMSNorm：每个元素只做少量运算但需读写整个张量
- Softmax：同样是elementwise+reduction，访存占主导
- 激活函数（GELU、SiLU等）
- Decode阶段的所有GEMV操作（batch=1时矩阵退化为向量，AI骤降）
- KV Cache的读取（decode阶段每步都要读取全部历史KV）
- Embedding lookup

判断关键：**decode阶段几乎所有操作都是memory-bound**，因为batch维度小，矩阵运算退化为矩阵向量乘（GEMV），算力利用率低，瓶颈在于把权重从HBM搬到计算单元。

</details>

**21. FlashAttention：FLOPs更高但更快**

<details>
<summary>💡 查看答案</summary>

标准Attention需要计算并存储完整的 N×N 注意力矩阵（S=$QK^{\top}$, P=softmax(S), O=PV），存在大量中间结果的写入和读取HBM操作。

FlashAttention通过**分块计算+在线softmax（online softmax）**，将Q、K、V分块加载到SRAM（片上高速缓存）中计算，避免将完整的 N×N 矩阵写回HBM。虽然为了实现online softmax需要额外的重计算（增加了部分FLOPs，比如需要重复计算某些中间项的rescale），导致总FLOPs略高于标准实现。

但因为：
- 标准Attention是 **memory-bound**（瓶颈在HBM读写N²大小的中间矩阵）
- FlashAttention大幅减少HBM访问次数（IO复杂度从 O(N²) 降到 O(N²/M)，M为SRAM大小相关项）

根据Roofline模型，当瓶颈是带宽而非算力时，减少访存量比减少FLOPs更重要。FlashAttention用"多算"换"少存取"，整体延迟反而更低，这是经典的**算法重新设计以适配硬件存储层级**的案例。

</details>

**22. 算子融合（Kernel Fusion）**

<details>
<summary>💡 查看答案</summary>

**原理**：将多个连续的、原本各自需要读写HBM的小算子（如 LayerNorm → Linear → GELU → Linear），合并成一个CUDA kernel，在GPU片上（寄存器/shared memory）完成多步计算，只在最开始和最后访问一次HBM。

**收益**：
1. **减少HBM访存次数**：避免中间结果的写入和读取，对memory-bound的elementwise算子（如LayerNorm、激活函数、bias-add、dropout）效果显著
2. **减少kernel launch开销**：每次kernel启动有固定的调度延迟，融合后减少调度次数
3. **提高数据局部性**：数据在SRAM/寄存器中复用，减少跨层数据搬运

典型例子：FlashAttention本身就是attention算子的融合；FusedLayerNorm、FusedAdam、bias+GELU融合等。代价是融合kernel的实现复杂度高，通用性降低，需要针对特定shape/硬件手写或用编译器（如Triton、TorchInductor）自动生成。

</details>

**23. CUDA中grid/block/thread/warp关系**

<details>
<summary>💡 查看答案</summary>

层级结构（从大到小）：
- **Grid**：一次kernel启动包含的所有block集合，对应整个计算任务
- **Block（线程块）**：grid中的一个执行单元，包含多个thread，block内线程可通过shared memory通信，可以同步（`__syncthreads()`）
- **Thread（线程）**：最小执行单元，每个线程执行相同的kernel代码，处理不同数据（SIMT模型）
- **Warp**：硬件调度的最小单位，固定为**32个线程**，同一warp内线程执行相同指令（SIMD lockstep）。block会被划分为多个warp调度执行

硬件映射关系：
- Block被分配到一个SM（Streaming Multiprocessor）上执行，SM内的warp scheduler负责调度warp
- 一个SM可同时驻留多个block/warp，通过warp间切换隐藏访存延迟（latency hiding）
- Grid维度可以是1D/2D/3D，用于映射数据的多维结构（如图像的行列、矩阵的tile）

设计原则：block大小通常设为32的倍数（避免warp内线程浪费），常见128/256/512线程/block。

</details>

**24. Prefill与Decode的区别**

<details>
<summary>💡 查看答案</summary>

| | Prefill | Decode |
|---|---|---|
| **输入** | 完整prompt（多个token） | 单个token（上一步生成的） |
| **矩阵形状** | (seq_len, hidden) 的GEMM | (1, hidden) 的GEMV |
| **计算特性** | compute-bound | memory-bound |
| **并行度** | 序列维度可并行处理 | 序列维度=1，并行度低 |
| **KV Cache** | 一次性写入所有token的KV | 每步读取全部历史KV+写入当前token的KV |
| **耗时主导因素** | 算力（FLOPS利用率高） | 显存带宽（权重和KV Cache的读取） |

**性能特征差异**：Prefill阶段GPU利用率高，吞吐量大但单次耗时较长（与prompt长度正相关）；Decode阶段逐token生成，每步都要重新加载全部模型权重，GPU计算单元大量空闲，是推理延迟的主要来源（自回归生成的串行特性导致decode阶段总耗时通常远超prefill）。

这也是PD分离架构（prefill/decode disaggregation）的motivation：两阶段资源需求特征完全不同，分开调度可以分别优化。

</details>

**25. 投机解码（Speculative Decoding）**

<details>
<summary>💡 查看答案</summary>

**核心原理**：用一个小而快的"draft模型"先自回归生成多个（如K个）候选token，再用大的"目标模型"**一次forward**并行验证这K个token是否与目标模型的输出分布一致。

**验证机制**：
- 对每个候选token，按照拒绝采样（rejection sampling）规则判断是否接受：若大模型在该位置给出该token的概率 ≥ draft模型给出的概率，则接受；否则按一定概率拒绝并从修正分布中重新采样
- 一旦某个位置被拒绝，后续候选全部丢弃，从该位置用大模型采样的token继续

**为什么能加速**：
- 关键洞察：decode阶段是memory-bound，大模型一次forward处理1个token和处理K+1个token的**延迟几乎相同**（都是一次权重读取，计算量增加但不是瓶颈）
- 因此用一次大模型forward验证K个token，平均接受率为$\alpha$时，相当于用1次forward的延迟生成了 (1 + $\alpha$ + $\alpha$² + ... )期望个token
- 数学期望上的加速比约为 (1-$\alpha^{K+1}$)/((1-$\alpha$)×(单步开销比))，接受率越高，draft模型越快越准，加速效果越好

输出分布严格等价于直接用大模型采样（无损加速），代价是需要额外维护一个draft模型，且draft与target模型分布差异大时接受率低，加速不明显。

</details>

**26. 量化的两个收益维度 & Prefill/Decode加速差异**

<details>
<summary>💡 查看答案</summary>

**两个收益维度**：
1. **显存占用减少**：权重从FP16/BF16（2字节）量化到INT8（1字节）或INT4（0.5字节），显存占用减少2-4倍，可以放更大batch或更长上下文
2. **访存带宽节省**：从HBM读取权重的数据量同比减少，对memory-bound场景直接降低延迟

**为什么decode加速明显，prefill加速不明显**：

Decode阶段是memory-bound（GEMV，权重读取主导耗时），量化后权重体积变小，读取时间直接按比例减少，延迟显著下降（例如INT4相比FP16理论上可以接近4倍加速）。

Prefill阶段是compute-bound（GEMM，矩阵乘法的计算占主导），瓶颈在Tensor Core的算力而非权重读取。虽然量化后权重读取量减少，但：
- 如果硬件不支持低精度算子的原生加速（如INT4 GEMM kernel），反而需要先反量化（dequantize）为FP16再计算，引入额外开销
- 即使支持低精度Tensor Core，计算本身的FLOPs（在reduce到fp运算后）并未减少，加速比远低于decode

因此实践中量化技术（GPTQ/AWQ等）主要针对memory-bound的decode/小batch场景优化，对大batch的prefill收益有限甚至可能因反量化开销而变慢。

</details>

**27. Shared Memory优化原理与使用场景**

<details>
<summary>💡 查看答案</summary>

**原理**：Shared Memory是GPU上每个SM内的片上高速存储（容量通常几十到上百KB），访问延迟远低于全局内存（HBM），同一block内所有线程可共享访问。

**优化原理**：
- 将数据从HBM一次性加载到shared memory，block内线程重复使用该数据时直接从shared memory读取，避免每个线程都单独访问HBM造成的大量重复全局内存访问
- 减少全局内存访问次数，提升数据复用率（提高arithmetic intensity，对memory-bound kernel尤其有效）

**使用场景**：
1. **矩阵乘法（GEMM）**：tile-based算法，将矩阵分块load到shared memory，每个tile内的数据被多个线程复用（经典CUDA GEMM优化）
2. **卷积运算**：滑动窗口的输入数据在邻近输出位置间存在重叠，可缓存到shared memory
3. **归约操作（Reduction）**：如softmax、layernorm的求和/求最大值，先在shared memory内做block内归约，再做block间归约
4. **FlashAttention**：将Q、K、V的block加载到shared memory，在片上完成attention计算，避免中间矩阵写回HBM

**注意**：shared memory容量有限，分配过多会限制每个SM能驻留的block数量（影响occupancy）；同时要注意bank conflict问题（见731）。

</details>

**28. Thread/Block/Grid关系**

<details>
<summary>💡 查看答案</summary>

（与719重复，补充说明）

- 一个Kernel启动对应一个**Grid**
- Grid由多个**Block**组成（可以是1D/2D/3D排列）
- 每个Block由多个**Thread**组成（也可以是1D/2D/3D排列）
- 每个Thread执行相同的kernel函数代码，通过`threadIdx`、`blockIdx`、`blockDim`、`gridDim`这些内置变量计算出自己负责处理的数据索引

\begin{verbatim}
int idx = blockIdx.x * blockDim.x + threadIdx.x;
\end{verbatim}

- Block是资源分配和调度的基本单位：一个block会被分配到一个SM上执行，block内的线程可以同步（`__syncthreads`）和共享shared memory
- Block内部进一步被划分为**Warp**（32线程）进行硬件调度执行

</details>

**29. Bank冲突是什么以及如何解决？**

<details>
<summary>💡 查看答案</summary>

**Bank冲突（Bank Conflict）**：Shared Memory被划分为多个（通常32个）等宽的存储体（bank），可以并行访问。如果同一个warp中的多个线程同时访问**同一个bank**的不同地址，这些访问会被**串行化**处理，导致性能下降。

**典型场景**：访问shared memory数组时，如果数组按stride访问且stride与bank数量（32）有公约数关系，容易导致同一warp内多个线程落在同一bank。

例如：32个bank，若线程i访问 `shared_mem[i * 32]`，所有线程都访问bank 0，产生32-way冲突。

**优化方法**：
1. **Padding（填充）**：在数组的行/列之间插入额外的元素，打破stride与bank数量的整除关系。例如二维数组声明为 `shared float tile[32][33]` 而非 `[32][32]`，使每行多一个元素，让相邻行的同一列错开bank
2. **改变访问模式**：调整线程对数据的访问顺序，使同一warp内线程访问连续地址（每个线程访问不同bank）
3. **广播机制利用**：如果所有线程访问同一地址（不是同一bank的不同地址），硬件支持广播，不算冲突

特殊情况：当所有线程访问完全相同的地址时不会产生冲突（broadcast），只有"同bank、不同地址"才是冲突。

</details>

**30. MoE主流程与Router按Token路由的原因**

<details>
<summary>💡 查看答案</summary>

**MoE主要流程**：
1. 输入的每个token的隐藏状态 h 送入**Router（门控网络）**，输出每个expert的得分（通常经过softmax）
2. 根据得分选择**Top-K个expert**（如Top-2）
3. 该token的隐藏状态被发送（dispatch）到选中的K个expert
4. 每个expert独立对接收到的token做FFN计算
5. 各expert的输出按照Router给出的权重做**加权求和（combine）**，得到该token最终的输出
6. 分布式场景下（EP并行），dispatch和combine通过**All2All通信**实现跨GPU的token转发

**为什么以token为单位路由**：
- MoE的核心思想是**条件计算（conditional computation）**：不同的输入应该激活不同的"专家知识"，而每个token代表一个独立的语义单元（一个词/子词），其语义内容、所需的处理方式可能差异很大
- 以token为粒度路由，使模型能针对每个token的具体语义特征（而非整个序列的统一特征）动态选择最相关的专家组合，实现细粒度的专业化分工
- 若以序列（sequence）为单位路由，会强制一个序列内所有token共享同一组expert，丧失了token级别的灵活性，且容易导致expert利用不均衡（某些expert永远不会被某类序列选中）
- token粒度路由也带来挑战：**负载不均衡**（某些expert被过多token选中导致显存/计算压力大，需要capacity factor限制和aux loss辅助平衡）、batch中不同token路由到不同expert导致**通信模式动态化**（All2All的通信量随路由结果变化）

</details>

**31. RoPE计算逻辑 & 长文本扩展方案**

<details>
<summary>💡 查看答案</summary>

**RoPE（Rotary Position Embedding）计算逻辑**：
- 核心思想：通过对Query和Key向量按位置施加**旋转变换**来注入位置信息，使得两个向量的内积（attention score）只依赖于它们的**相对位置**
- 将d维向量两两分组（d/2组），每组视为一个2D平面上的向量
- 对位置m的向量，第i组(2D子空间)按角度 m×$\theta_{i}$ 进行旋转，其中 $\theta_{i}$ = $base^{-2i/d}$，base通常取10000
- 旋转矩阵作用于(x_{2i}, x_{2i+1})：
    \begin{verbatim}
    x'_{2i}   = x_{2i}*cos(mθ_i) - x_{2i+1}*sin(mθ_i)
    x'_{2i+1} = x_{2i}*sin(mθ_i) + x_{2i+1}*cos(mθ_i)
    \end{verbatim}
- 由于旋转变换的性质，Q_m·K_n（内积）只与相对位置(m-n)和$\theta$有关，天然编码了相对位置信息

**长文本扩展方案对比**：

| 方法 | 思路 | 特点 |
|---|---|---|
| **Sliding Window** | 限制attention只在固定窗口大小内计算（如Mistral的局部attention） | 不改变RoPE本身，通过限制感受野控制计算量，远距离信息通过多层堆叠间接传递（类似CNN的receptive field扩大）；无法直接扩展模型对绝对长距离位置的理解能力 |
| **DynamicNTK** | 推理时根据当前实际序列长度**动态调整RoPE的base值**，使得超出训练长度时频率分布平滑外推 | 不需要微调，base随序列长度按比例放大（NTK-aware scaling），训练长度内行为不变，超出后逐渐改变频率，缓解了直接外推时高频维度"周期重复混淆"的问题 |
| **YaRN** | 结合NTK插值思想，**对不同频率维度采取不同的插值策略**：高频维度（局部位置敏感）保持外推/小幅调整，低频维度（全局位置）做线性插值（interpolation），并引入attention温度缩放校正 | 通常需要少量微调以达到最佳效果，相比单纯NTK或PI（Position Interpolation）在更长上下文上保持更好的精度，是目前长文本扩展的主流方案之一 |

核心矛盾：训练时模型只见过有限长度的位置编码，直接外推到更长序列会使高频旋转角度超出训练分布（"未见过的旋转角度"导致attention pattern混乱）；各方案本质都是在"改变频率分布"和"保持训练时学到的局部attention pattern"之间做权衡。

</details>

**32. 说下采样算法有哪些**

<details>
<summary>💡 查看答案</summary>

**Argmax（贪心解码）**：每步选择概率最高的token，结果确定性强但容易陷入重复、缺乏多样性。

**温度（Temperature, T）**：
- 对logits做缩放：`logits / T`，再做softmax
- T < 1：分布更尖锐（更确定，偏向高概率token）
- T > 1：分布更平坦（更随机，增加多样性）
- T → 0 等价于argmax；T=1为原始分布

**Top-K采样**：
- 只保留概率最高的K个token，重新归一化后从中采样
- 避免从概率极低的"长尾"token中采样产生不合理输出；K越小越保守

**Top-P（Nucleus）采样**：
- 按概率从高到低累加，直到累计概率超过阈值P，保留这个最小集合，从中重新采样
- 相比Top-K更动态：分布尖锐时集合小（接近贪心），分布平坦时集合大（更多样），自适应性更好

**Beam Search**：
- 维护多个（beam size = B个）候选序列，每步对每个候选扩展所有可能的下一token，保留累积得分（log概率和）最高的B个序列
- 用于追求"整体序列概率最大化"的任务（如翻译、摘要），开放式生成任务中容易产生重复、缺乏多样性的输出，且计算开销随beam size增加

**组合使用**：实际生成常用 `temperature + top-K + top-P` 组合采样，先用top-K/top-P裁剪候选集合，再用temperature调整分布形状后采样。

</details>

**33. KV Cache大小计算 & TP/PP下的分布**

<details>
<summary>💡 查看答案</summary>

**KV Cache大小公式**：
\begin{verbatim}
KV Cache size = 2 (K和V) × batch_size × seq_len × num_layers × num_kv_heads × head_dim × dtype_bytes
\end{verbatim}

例如：Llama-2-7B（32层，32个KV head，head_dim=128，FP16），batch=1，seq_len=4096：
\begin{verbatim}
2 × 1 × 4096 × 32 × 32 × 128 × 2 bytes ≈ 2.1 GB
\end{verbatim}

若使用**GQA/MQA**（KV head数量少于Query head数量），num_kv_heads会远小于num_attention_heads，大幅减少KV Cache（如num_kv_heads=8时，KV Cache缩小4倍）。

**TP下的分布**：
- TP将attention的head维度切分到不同GPU（每个GPU负责 num_kv_heads / TP_size 个KV head）
- 因此每个GPU上的KV Cache大小为总量的 **1/TP_size**
- 注意：若num_kv_heads < TP_size（如MQA的num_kv_heads=1，TP_size=8），则KV head无法继续切分，部分GPU上会出现KV head的**冗余复制**（每个GPU仍需完整存一份该KV head）

**PP下的分布**：
- PP将layer维度切分到不同stage
- 每个stage（GPU）只需存储自己负责的那部分layer对应的KV Cache，因此每个GPU的KV Cache大小为总量的 **1/PP_size**（按层数比例切分）
- 不同stage上KV Cache互不重叠，序列处理过程中KV Cache随着层间流水线传递（实际是激活值传递，KV Cache就地保存在各自stage）

**综合**：TP和PP同时使用时，单GPU的KV Cache ≈ 总KV Cache / (TP_size × PP_size)（在num_kv_heads足够大、可被TP整除的前提下）。

</details>

**34. 模型轻量化方法思想对比**

<details>
<summary>💡 查看答案</summary>

**AWQ（Activation-aware Weight Quantization）**：
- 核心：识别对激活值影响大的"显著权重通道"(salient channels)，通过**per-channel缩放**保护这些重要权重的精度，而非用混合精度
- 缩放因子搜索：在校准集上搜索使量化误差最小的scale，将"重要性"从权重转移到激活（数学上等价变换：W·s 和 X/s）
- 特点：weight-only量化，无需反向传播/训练，硬件友好（标准INT4 kernel即可）

**SmoothQuant**：
- 针对**weight和activation都量化**（W8A8）场景
- 核心问题：LLM的激活值存在大量**outlier**（离群值），导致per-tensor量化误差很大
- 解决：引入平滑因子s，将激活的"尖锐分布"转移一部分到权重上（X/s, W·s），使激活变得平滑易量化，同时权重的分布相对平坦，量化误差易于容忍
- 本质是在weight和activation之间做量化难度的**再平衡（迁移）**

**GPTQ（GPT Quantization）**：
- 基于OBQ（Optimal Brain Quantization）的逐层量化框架，利用二阶信息（Hessian矩阵的逆）逐列量化权重，并将量化误差实时补偿到尚未量化的权重上，最小化整体输出误差
- 是一种**误差补偿式**的逐层贪心量化算法，需要少量校准数据计算Hessian近似

**SpinQuant**：
- 核心思想：通过引入**可学习的旋转矩阵（rotation matrix）**对权重和激活做正交变换
- 旋转变换不改变数学计算结果（正交矩阵），但可以重新分布激活值/权重中的outlier，使变换后的分布更适合量化（类似于将outlier"旋转扩散"到各个维度，降低单维度的极值）
- 通过在校准数据上优化旋转矩阵（如Cayley参数化+优化算法），比SmoothQuant的简单缩放更灵活，对4-bit量化效果更好

**总结脉络**：AWQ/GPTQ聚焦weight-only量化（保护重要权重/误差补偿）；SmoothQuant/SpinQuant聚焦weight+activation联合量化（通过数学等价变换重塑分布以适配量化），SpinQuant是SmoothQuant思路的进一步泛化（缩放→旋转）。

</details>

**35. 数据并行 / 张量并行 / 流水线并行**

<details>
<summary>💡 查看答案</summary>

**数据并行（Data Parallelism, DP）**：
- 每个GPU持有完整模型副本，输入数据切分到不同GPU
- 各GPU独立前向+反向，梯度通过AllReduce同步
- **适用场景**：模型能放入单卡显存，主要瓶颈是数据吞吐
- **Trade-off**：通信量大（每步AllReduce全部梯度），显存冗余高（每卡都存完整模型+优化器状态）

**张量并行（Tensor Parallelism, TP）**：
- 将单层内部的矩阵运算（如Linear层的权重矩阵）按行或列切分到不同GPU，每个GPU计算部分结果，再通过AllReduce/AllGather合并
- **适用场景**：单层参数过大无法放入单卡，需要节点内多卡协作；对通信延迟极其敏感（每层前向都需要一次同步）
- **Trade-off**：通信频率最高（每个Transformer block内都有多次集合通信），要求极高带宽（NVLink），跨节点效率骤降

**流水线并行（Pipeline Parallelism, PP）**：
- 按layer将模型纵向切分到不同GPU（不同GPU负责不同的层），数据像流水线一样依次流过各阶段
- **适用场景**：模型层数很深、总参数量巨大，需要跨节点扩展
- **Trade-off**：存在流水线气泡（pipeline bubble），即GPU等待上下游数据的空闲时间；需要micro-batching和合理的调度策略（如1F1B）来减少bubble；通信量相对较小（只传递层间激活值），对带宽要求低于TP

实际大模型训练通常**三者混合使用**（3D并行）：节点内TP，节点间PP，多组之间DP。

</details>

**36. 为什么TP适合节点内、PP适合节点间**

<details>
<summary>💡 查看答案</summary>

**TP适合节点内**：
- TP在每个Transformer层内部都需要执行AllReduce/AllGather通信（比如attention和FFN各一次），通信频率极高（每层2次以上）
- 这些通信涉及的数据量较大（激活值张量），且对延迟极度敏感——通信无法被计算掩盖，是严格的同步阻塞点
- 节点内GPU通过**NVLink/NVSwitch**互联，带宽可达数百GB/s甚至TB/s级别，延迟极低，能够支撑这种高频大数据量通信
- 跨节点用的是InfiniBand/RoCE等网络，带宽（通常几十到上百GB/s）和延迟远不如NVLink，若TP跨节点，每层的通信延迟会成为严重瓶颈

**PP适合节点间**：
- PP只需在stage之间传递**激活值**（forward）和**梯度**（backward），数据量相对TP小很多（只是某一层的输出张量，而非全部权重相关的中间结果）
- 通信发生在stage边界，频率远低于TP（每个micro-batch一次，而不是每层多次）
- 这种"低频率、可流水线掩盖"的通信模式，对网络带宽和延迟的要求宽松很多，适合跨节点的网络条件
- 流水线的bubble本身可以通过增加micro-batch数量来缓解，跨节点的额外延迟可以被流水线的并行执行部分隐藏

简言之：**TP通信频繁且对带宽/延迟敏感 → 必须用最快的互联（节点内NVLink）；PP通信稀疏且可流水线掩盖 → 可以容忍较慢的互联（节点间网络）**。

</details>

**37. ZeRO三个Stage是什么**

<details>
<summary>💡 查看答案</summary>

ZeRO（Zero Redundancy Optimizer）通过切分各类训练状态来消除数据并行中的显存冗余。三个状态：优化器状态(Optimizer States, O)、梯度(Gradients, G)、参数(Parameters, P)。

- **Stage 1 (ZeRO-1)**：切分**优化器状态**（如Adam的momentum、variance）。每个GPU只保存1/N的优化器状态，更新参数时需要先AllGather获得完整梯度对应的更新量
- **Stage 2 (ZeRO-2)**：在Stage 1基础上，进一步切分**梯度**。每个GPU只保存自己负责更新部分参数所需的梯度，反向传播后梯度通过Reduce-Scatter分发到对应GPU
- **Stage 3 (ZeRO-3)**：在Stage 2基础上，进一步切分**模型参数**本身。每个GPU只持久化存储1/N的参数，前向/反向计算时需要临时AllGather所需层的完整参数，用完即释放

**显存与代价的Trade-off**：
- 显存节省：Stage1→Stage2→Stage3，显存占用持续降低，理论上Stage3可将模型状态显存降至 O(1/N)，N为GPU数
- 代价：
- Stage1/2：通信量与标准DP的AllReduce相近（甚至更优），代价较小
- Stage3：每层前向和反向都需要额外的AllGather获取参数，通信量显著增加（约为标准DP的1.5倍），且引入额外的延迟（需要等待参数到达才能计算），对网络带宽要求更高
- 切分越细，显存换得越多，但通信开销和实现复杂度（需要频繁的gather/release时机管理）也随之增加

实践中，ZeRO-3常配合offload（CPU/NVMe）进一步突破单机显存限制，但会引入额外的PCIe传输延迟。

</details>

**38. 大模型外推中存在的问题有哪些？**

<details>
<summary>💡 查看答案</summary>

大模型的"外推"（Extrapolation）是指在推理阶段处理长度超过训练时最大序列长度的输入。这一场景下主要存在以下两类核心问题：

1. **位置编码的分布外（Out-of-Distribution）问题**：无论是绝对位置编码（如原始 Transformer 的正弦编码、BERT 的可学习位置嵌入）还是相对位置编码（如 RoPE、ALiBi），模型在训练时只见过长度范围内的位置索引。推理时一旦出现训练中从未出现过的位置编码值，模型对这些位置的表示没有任何学习保证，导致注意力权重计算混乱，输出质量急剧下降。以 RoPE 为例，训练时旋转角度 $\theta$ 对应的频率范围有限，超出范围的位置对应的旋转矩阵在高频分量上会产生严重震荡，破坏位置信息的相对一致性。

2. **注意力分布稀释（Attention Dilution）问题**：Transformer 的注意力机制在计算 softmax 时，是对当前 token 与序列中**所有** token 的相似度做归一化。当序列长度大幅超过训练长度时，参与归一化的 token 数量急剧增加，导致注意力权重被"摊薄"，每个 token 分配到的注意力概率值趋近于均匀分布（$1/n$）。这意味着模型难以聚焦于真正相关的上下文，有效信息被大量无关 token 淹没，模型的上下文理解能力随序列变长而退化。这一现象也被称为"注意力崩塌"（Attention Collapse）。

两类问题相互叠加，使得直接对预训练 LLM 进行长上下文推理时，模型性能会在超出训练长度后迅速恶化，困惑度（perplexity）急剧上升。

</details>

**39. 如何解决大模型外推中的不一致和超长预测长度泛化性问题？**

<details>
<summary>💡 查看答案</summary>

**核心思路**：通过 Attention Mask 操作，将推理阶段每个 token 实际"看到"的上下文窗口长度限制在训练时的范围内，从而同时消除位置编码 OOD 和注意力稀释两个问题。

**具体方法**：

- **滑动窗口注意力（Sliding Window Attention）**：为每个 token 设置一个固定大小的局部注意力窗口（等于训练最大长度），仅关注最近的 $w$ 个 token，通过 Attention Mask 屏蔽窗口外的所有位置。这样无论输入序列多长，每个 token 参与计算的上下文长度始终与训练时一致，位置编码始终在已训练的范围内。Mistral 7B 采用了这一策略。

- **StreamingLLM**：在滑动窗口的基础上，额外保留序列开头的若干"Attention Sink" token（通常是前 4 个），因为研究发现 LLM 倾向于将大量注意力集中在初始 token 上。StreamingLLM 的 Attention Mask 同时保留"初始锚点 + 局部窗口"，在无限流式输入场景下实现稳定推理。

- **位置编码内插（Position Interpolation）**：通过对位置索引进行线性缩放（如将 $[0, L_{max_ext}]$ 的位置压缩映射回 $[0, L_{max_train}]$），使得超长序列的位置编码值始终落在训练分布内，再配合少量微调（Fine-tuning）恢复压缩带来的精度损失。LLaMA 的长上下文扩展即采用此方案。

- **RoPE 缩放变体（YaRN、LongRoPE 等）**：针对 RoPE 位置编码，在不同频率分量上使用不同的缩放因子（高频分量更激进地压缩，低频分量保持不变），在保持短距离位置精度的同时实现长上下文外推，结合 Attention Mask 和少量长文本微调，可将上下文窗口扩展至 128K 甚至更长。

**统一视角**：上述方法的本质都是通过 Attention Mask 或位置编码变换，确保推理时每个 token 的有效注意力计算等价于在训练窗口长度内进行，从根本上消除了训练与推理之间的分布不一致，同时将注意力分布的有效 token 数量控制在模型已充分学习的范围内。

</details>

