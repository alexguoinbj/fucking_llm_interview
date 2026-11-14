# 3.7 推理过程 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 3 章：预训练、推理与工程优化 · 共 13 题

**1. 缓存命中和缓存未命中是什么意思？**

<details>
<summary>💡 查看答案</summary>

在大模型接口里，“缓存命中（cache hit）”与“缓存未命中（cache miss）”描述的是：当前这次请求的可计算部分，是否已经在系统中以可复用的形式被计算过并保存下来。

**请求级缓存**：系统对 prompt 做规范化与哈希，如果完全相同的请求曾被执行过且输出仍有效，直接返回历史结果（零前向计算）。

**前缀级缓存**：系统缓存某个前缀对应的 KV cache 中间状态。“命中”的含义是当前请求的前缀中有一段已在历史请求中出现过，系统可直接复用。一个请求可能前 2000 个 token 命中缓存，而后续 token 未命中。

缓存命中直接关联三个指标：响应延迟、算力成本和吞吐稳定性。

</details>

**2. 现在大模型在推理阶段都是左 padding？**

<details>
<summary>💡 查看答案</summary>

在推理阶段，单解码器架构的大模型通常倾向于使用左填充（left-padding）。这是因为模型在生成文本时，会依赖序列中最后一个有效 token 的 logits 来决定下一步输出。如果采用右填充，最后一个 token 很可能是 `<pad>`，导致错误结果。左填充确保序列的最后一个 token 始终是真实的、有意义的 token。

此外，左填充使得不同长度的序列在右侧对齐，KV-cache 可以被复用，提高推理效率。

</details>

**3. 请问温度为 0 一定可以得到确定的输出吗？**

<details>
<summary>💡 查看答案</summary>

虽然温度为 0 理论上应消除随机性，但在实践中 LLM 的输出仍可能出现轻微变化，原因包括：
- **MoE 路由效应**：批处理竞争可能导致路由决策不同。
- **浮点舍入误差**：浮点运算精度有限。
- **并行性与竞争条件**：GPU 的并行操作可能引入非确定性。

</details>

**4. 为什么 G4-turbo 在 topp=0.0 的情况下还不能保证预测稳定性？**

<details>
<summary>💡 查看答案</summary>

即使设置 `top_p=0.0`，实际采样仍受以下因素干扰：
- 温度参数的残留影响
- 浮点数精度限制
- 并行计算的数值误差
- Transformer 架构的累积误差

</details>

**5. 同一个 prompt 复制 5 次进 LLM 为啥得到的输出都不一样？**

<details>
<summary>💡 查看答案</summary>

采用贪婪解码理论上结果应一致。不一致的原因：
1. **误差累积**：模型内部的随机性操作（如层归一化）可能影响结果。
2. **浮点数计算不精确性**：大规模矩阵运算的精度误差会逐渐累积。
3. **padding 位置影响**：padding 位置的极小值在深层模型中可能被放大。

</details>

**6. 如何处理语言模型中的长期依赖关系？**

<details>
<summary>💡 查看答案</summary>

语言模型中的长期依赖关系指的是捕捉序列中相隔较远的单词之间的关系。标准的神经网络通常在这方面表现不佳，因为信息会随着时间的推移而衰减。以下是一些解决长期依赖关系的标准技术：

- **循环神经网络（RNN）**：这些网络内部有循环，允许它们存储来自先前输入的信息，并用它来影响当前输入的处理。这种形式的“记忆”可以弥合相隔较远的单词之间的差距。
- **长短期记忆网络（LSTM）**：这是一种专门设计用于解决梯度消失问题的 RNN 架构。LSTM 内部有控制信息流动的门，允许它们保留相关信息更长时间，并处理更远距离的依赖关系。
- **Transformer-XL**：这是 Transformer 架构的一种改进，另一种用于语言任务的强大神经网络。Transformer-XL 通过在编码新输入片段时保留和利用先前处理步骤中的隐藏状态来解决长期依赖问题。这使它能够在更长的序列中保持上下文。
- **注意力机制**：这些技术允许模型专注于当前预测最相关的输入序列的特定部分。即使这些部分相隔较远，关注这些信息部分也有助于模型学习长期依赖关系。

</details>

**7. 为什么大模型推理时显存涨得那么多还一直占着？**

<details>
<summary>💡 查看答案</summary>

大模型推理时显存占用高且持续占用，是多种因素叠加的结果，并非单一原因。

#### 显存占用的主要来源

1. **模型参数权重**：大模型参数众多，权重本身占据了显存的基础底线。以 LLaMA-3 70B 模型为例，FP16 精度下参数本身即占用约 $70 \times 2 = 140$ GB 显存。这部分显存在整个推理过程中**始终占用**，无法释放，因为每次前向传播都需要访问全部权重。

2. **KV Cache（键值缓存）**：这是推理阶段显存“涨起来”的核心原因。在自回归解码（Autoregressive Decoding）中，每生成一个新 Token，都需要与之前所有 Token 的 Key 和 Value 做 Attention 计算。为避免重复计算，框架会将历史 Token 的 $K$、$V$ 矩阵缓存起来。

    KV Cache 的显存占用约为：
    \[
    KV Cache = 2 \times L \times H \times d_h \times S \times B \times bytes
    \]
    其中 $L$ 为层数，$H$ 为注意力头数，$d_h$ 为每头维度，$S$ 为已生成序列长度，$B$ 为批大小。以 LLaMA-3 70B、FP16、batch=1、序列长度 4096 为例，KV Cache 约占 **8 GB**，且随序列长度线性增长——这正是推理过程中显存持续上涨的主要原因。

3. **中间激活值**：前向传播过程中，每一层的输入输出张量（形状为 $[B, S, d_{model}]$）需要暂存于显存以完成当前层计算。推理时无需反向传播，故激活值在每层计算完成后即可释放，但在峰值时刻（同时存在多层激活）仍会带来额外显存压力。

4. **框架运行时开销**：CUDA 上下文（约 300\~{}500 MB）、NCCL 通信缓冲区（多卡推理时）、cuDNN/cuBLAS 工作区、PyTorch 内存池的预分配碎片等，均会占用一部分显存，且通常在进程退出前不会释放。

5. **显存释放延迟**：推理结束后，PyTorch 的显存分配器（caching allocator）并不立即将显存归还给 CUDA 驱动，而是将其保留在内存池中以备复用。因此在 `nvidia-smi` 中观察到的显存占用可能远高于当前实际使用量，形成“一直占着”的假象。只有调用 `torch.cuda.empty_cache()` 或进程退出后，显存才会真正归还。

#### 显存占用随时间变化的典型曲线

\begin{lstlisting}[style=promptstyle]
	时间轴 →
	[模型加载] → 显存跳升至参数占用基线（如 140 GB）
	[Prefill 阶段] → 处理输入 prompt，KV Cache 初始化，显存小幅上升
	[Decode 阶段，逐 Token 生成] → KV Cache 随序列长度线性增长，显存持续上涨
	[生成结束] → 显存维持高位（框架缓存，未释放）
	[empty_cache()] → 显存回落至参数基线
\end{lstlisting}

% -------------------------------------------------------

</details>

**8. 大模型推理时显存占用高，有没有解决办法？**

<details>
<summary>💡 查看答案</summary>

针对不同来源的显存占用，有对应的优化手段：

1. **禁用梯度计算**：推理时使用 `torch.no_grad()` 或 `torch.inference_mode()` 上下文管理器，阻止 PyTorch 构建计算图和存储梯度信息。`inference_mode` 比 `no_grad` 更彻底，还会禁用版本追踪，显存节省更多，是推理场景的首选。

2. **主动释放中间变量**：对不再需要的大张量显式 `del`，并调用 `torch.cuda.empty_cache()` 将 PyTorch 缓存池中的空闲显存归还给 CUDA 驱动。注意：`empty_cache()` 不能释放仍被引用的张量，需先确保变量已无引用。

3. **将输出移动到 CPU**：在流式生成场景中，将已生成的 Token 的隐藏状态或 logits 及时移至 CPU（`.cpu()`），避免 GPU 显存因累积输出而持续增长。

4. **KV Cache 优化**：
    - **PagedAttention（vLLM）**：将 KV Cache 以固定大小的"页"（Page）为单位动态分配，避免为最大序列长度预分配全量显存，显存利用率从约 $30%$ 提升至 $90%$ 以上。
    - **KV Cache 量化**：将 KV Cache 以 INT8 或 INT4 存储（如 `transformers` 的 `cache_implementation="quantized"`），显存降低 $2\sim4\times$，精度损失通常可接受。
    - **滑动窗口注意力（Sliding Window Attention）**：仅缓存最近 $W$ 个 Token 的 KV，超出窗口的历史 KV 被丢弃，KV Cache 大小从 $O(S)$ 降为 $O(W)$，Mistral 系列采用此方案。
    - **Multi-Query Attention（MQA）/ GQA**：减少 KV 头数量（所有 Query 头共享少数 KV 头），从源头降低 KV Cache 大小，LLaMA-3、Qwen2 等均采用 GQA。

5. **模型参数卸载（Offloading）**：
    - 使用 **ZeRO-Inference**（DeepSpeed）或 **Accelerate** 的 `device_map` 功能，将部分层的权重放在 CPU 内存或 NVMe 磁盘，按需搬运到 GPU 计算，以牺牲推理速度换取更低的 GPU 显存占用。
    - **llama.cpp** 支持将部分层卸载到 GPU（`--n-gpu-layers`），其余层在 CPU 上计算，实现 GPU/CPU 混合推理。

6. **混合精度推理**：使用 BF16 或 FP16 等低精度格式存储和计算，相比 FP32 显存降低 $50%$。BF16 是目前推荐的默认推理精度（动态范围与 FP32 相同，无需损失缩放）。对于更激进的场景，可使用 FP8（H100 原生支持）。

7. **模型量化压缩**：
    - **GPTQ / AWQ**：训练后权重量化（INT4/INT3），在精度损失极小的情况下将模型大小压缩 $4\sim8\times$，如 70B 模型量化后可在单张 A100 80G 上运行。
    - **GGUF（llama.cpp）**：支持 Q2_K 到 Q8_0 等多种量化级别，适合 CPU 和混合推理场景。
    - **bitsandbytes**：提供 `load_in_8bit` 和 `load_in_4bit` 选项，与 HuggingFace `transformers` 无缝集成，一行代码即可启用量化加载。

8. **批处理与请求调度优化**：
    - 使用**连续批处理（Continuous Batching）**动态调度请求，避免为等待中的请求预占 KV Cache 显存；
    - 设置合理的 `max_model_len` 上限，防止单个超长请求占用过多 KV Cache。

% -------------------------------------------------------

</details>

**9. 大模型在 GPU 和 CPU 上推理速度如何？**

<details>
<summary>💡 查看答案</summary>

#### GPU 推理

GPU 的大规模并行架构（数千个 CUDA Core + Tensor Core）使其非常适合 Transformer 中大量的矩阵乘法（GEMM）操作。

- **Prefill 阶段（计算密集）**：处理输入 Prompt 时，所有 Token 的 QKV 计算可以并行执行，GPU 的并行优势充分发挥，Tensor Core 利用率高，算术强度（FLOP/Byte）高，通常可达 GPU 峰值算力的 $40%\sim80%$。
- **Decode 阶段（访存密集）**：每次只生成一个 Token，矩阵乘法退化为矩阵-向量乘法（GEMV），算术强度极低（约 $1\sim2$ FLOP/Byte），GPU 计算单元大量空闲，瓶颈在 HBM 带宽。A100 的 HBM 带宽为 2 TB/s，H100 为 3.35 TB/s，这也是为何大批量推理（大 batch）能显著提升 Decode 吞吐量（多个请求共享一次权重加载）。
- **典型速度**：A100 80G 上，LLaMA-3 70B（BF16）Decode 速度约 $20\sim30$ tokens/s（batch=1），FP8 量化后可达 $40\sim60$ tokens/s。

#### CPU 推理

CPU 的核心数少（通常 $16\sim128$ 核），但每核的单线程性能强，适合小模型或量化后的低精度推理。

- **SIMD 加速**：现代 CPU（Intel Xeon、AMD EPYC）支持 AVX-512 指令集，可一次处理 16 个 FP32 或 32 个 FP16/BF16 数据，对量化模型（INT8/INT4）加速效果尤为显著。
- **内存带宽**：CPU 的 DDR5 内存带宽约 $200\sim400$ GB/s，远低于 A100 的 HBM 带宽（2 TB/s），是 CPU 推理 Decode 速度慢的根本原因。
- **典型速度**：双路 Xeon（96 核）上，llama.cpp Q4_K_M 量化的 LLaMA-3 8B 约 $10\sim20$ tokens/s，70B 模型约 $1\sim3$ tokens/s。
- **优化工具**：llama.cpp（高度优化的 CPU 推理引擎，支持 AVX2/AVX-512/AMX）、OpenVINO（Intel 专用）、ONNX Runtime（跨平台）。

#### GPU vs CPU 速度对比（以 LLaMA-3 8B 为例）

\begin{center}


| **环境** | **精度** | **Decode 速度（tokens/s）** |
| --- | --- | --- |
| A100 80G | BF16 | $\sim$120 |
| RTX 4090 | FP16 | $\sim$80 |
| A100 80G | INT8 | $\sim$200 |
| 双路 Xeon 96C | Q4_K_M | $\sim$15 |
| Apple M3 Max | Q4_K_M | $\sim$50（统一内存带宽优势） |


\end{center}

Apple Silicon（M 系列芯片）因其统一内存架构（CPU 和 GPU 共享高带宽内存，约 400 GB/s），在小模型 CPU/GPU 混合推理上表现出色，是消费级本地推理的热门选择。

% -------------------------------------------------------

</details>

**10. 如何具体实施 GPU-CPU 混合推理？**

<details>
<summary>💡 查看答案</summary>

GPU-CPU 混合推理的核心思想是：将计算密集的操作留在 GPU，将显存溢出的部分卸载到 CPU 内存，利用 PCIe/NVLink 带宽在两者间传输数据，在显存受限的情况下实现更大规模模型的推理。

#### 异构计算负载分配

1. **按层划分（Layer-wise Offloading）**：将模型的前 $k$ 层放在 GPU，后 $L-k$ 层放在 CPU（或 NVMe），推理时按顺序搬运层权重执行计算。`Accelerate` 的 `device_map="auto"` 会根据可用显存自动确定 $k$。代价是 PCIe 传输（约 16\~{}32 GB/s）成为瓶颈，解码速度可能下降 $5\sim20\times$。

2. **按操作类型划分**：
    - Prefill 阶段（批量 Token，矩阵并行度高）→ 全部在 GPU 执行，充分利用 Tensor Core；
    - Decode 阶段（单 Token，GEMV 操作）→ 部分层或 KV Cache 计算在 CPU 执行，减少 GPU 显存占用；
    - 数据预处理（Tokenization）和后处理（Sampling、Beam Search）→ CPU 执行，节省 GPU 算力。

3. **KV Cache 分级存储**：
    - 当前活跃请求的 KV Cache → GPU HBM（最快，延迟最低）；
    - 非活跃或历史 KV Cache → CPU 内存（带宽约 200 GB/s，延迟约 $10\times$ GPU）；
    - 超长历史上下文 → NVMe SSD（带宽约 7 GB/s，仅用于极端长上下文场景）。
        LLM.int8() 和 FlexGen 均采用类似的 KV Cache 分级策略。

#### 内存层级优化

- **NUMA 感知**：在多路 CPU 服务器上，确保模型权重和 KV Cache 分配在与 GPU 直连的 NUMA 节点内存上，避免跨 NUMA 访问带来的额外延迟（跨 NUMA 带宽约为本地的 $50%$）。
- **Pinned Memory（锁页内存）**：将 CPU 侧的权重和 KV Cache 分配在锁页内存（`torch.cuda.HostAllocator` 或 `pin_memory=True`）中，使 PCIe DMA 传输绕过操作系统 Page 机制，带宽提升约 $2\times$。
- **预取（Prefetching）**：在 GPU 执行第 $k$ 层计算时，异步将第 $k+1$ 层的权重从 CPU 搬运到 GPU（通过独立 CUDA Stream），将传输延迟隐藏在计算时间内，是混合推理性能优化的关键。

#### 软件栈优化

- **FlexGen**（学术框架）：专门为单 GPU + 大 CPU 内存 + NVMe 的混合推理设计，通过线性规划自动求解最优的层分配和调度策略，支持 OPT-175B 在单张 RTX 3090 上推理。
- **llama.cpp**：通过 `--n-gpu-layers N` 指定 GPU 层数，其余层在 CPU 执行，实现灵活的 GPU/CPU 混合，支持 GGUF 量化格式，是本地部署最成熟的工具。
- **DeepSpeed ZeRO-Inference**：支持将权重按需从 CPU/NVMe 流式传输到 GPU，并行执行计算与传输，对 HuggingFace 模型兼容性好。

#### 硬件配置优化

- 优先将 GPU 插在直连 CPU 的 PCIe 插槽（PCIe 5.0 x16，带宽约 64 GB/s），避免通过 PCIe Switch 中转导致带宽折半；
- 使用 8 通道 DDR5-4800 内存（理论带宽约 307 GB/s），为 CPU 侧计算和数据预取提供充足带宽；
- 若预算允许，使用 NVLink（NVIDIA NVLink Switch）将多张 GPU 连接为统一显存池（NVSwitch，双向带宽 900 GB/s），从根本上缓解单卡显存限制。

% -------------------------------------------------------

</details>

**11. FP16 在哪些硬件上支持最好？**

<details>
<summary>💡 查看答案</summary>

FP16（IEEE 754 半精度浮点）的硬件支持质量差异显著，主要体现在专用计算单元、峰值吞吐量和数值处理能力三个维度。

#### NVIDIA GPU（支持最完善）

- **H100/H800**：Hopper 架构，FP16 Tensor Core 峰值达 **989 TFLOPS**（SXM5 版本），同时原生支持 FP8（Transformer Engine），是目前 FP16 推理性能最强的商用 GPU。支持 Transformer Engine 在 FP16 和 FP8 之间动态切换，保证精度的同时最大化吞吐量。
- **A100/A800**：Ampere 架构，FP16 Tensor Core 峰值 **312 TFLOPS**，HBM2e 带宽 2 TB/s，是目前大模型推理的主流数据中心 GPU。支持 sparsity（稀疏加速），稀疏 FP16 峰值可达 624 TFLOPS。
- **RTX 4090**：Ada Lovelace 架构，FP16 峰值约 **165 TFLOPS**，24 GB GDDR6X（带宽 1 TB/s），适合中等规模模型（7B\~{}13B）的本地 FP16 推理，性价比极高。
- **RTX 3090**：Ampere 架构，FP16 峰值约 **71 TFLOPS**，24 GB GDDR6X，可运行 7B FP16 模型或 30B INT4 量化模型。
- **注意**：消费级 GPU（RTX 系列）的 FP64 性能被刻意限制（约为 FP32 的 $\frac{1}{64}$），但 FP16 性能不受限，非常适合推理场景。

#### Intel CPU/GPU

- **Intel Xeon Scalable（第四代+）**：通过 **AVX-512 BF16** 指令集支持 BF16（注意：原生 FP16 支持较弱，主要优化方向是 BF16），以及 **AMX（Advanced Matrix Extensions）** 加速 INT8 和 BF16 矩阵运算，峰值可达约 **7 TFLOPS**（BF16），适合大规模 CPU 集群推理。
- **Intel Arc GPU / Gaudi2**：Gaudi2 的 FP16 峰值约 **432 TFLOPS**，是 Intel 面向 AI 推理的主要产品，支持 FP16/BF16/FP8，但软件生态相比 NVIDIA 仍有差距。

#### AMD GPU

- **MI300X**：CDNA3 架构，FP16 峰值 **1307 TFLOPS**，HBM3 带宽 **5.3 TB/s**，192 GB HBM3，是目前单卡带宽最高的 AI 加速卡，在 Decode 带宽受限阶段理论优势明显。支持 ROCm 生态，与 PyTorch 兼容性日益改善。

#### 边缘/移动芯片

- **NVIDIA Orin-X（Jetson AGX Orin）**：支持 INT8 和 FP16 推理，FP16 峰值约 **275 TOPS**（INT8 等效），TDP 仅 60W，适合自动驾驶、机器人等低功耗场景的端侧推理。
- **Apple Silicon（M3/M4 系列）**：统一内存架构（CPU+GPU 共享内存，带宽 300\~{}400 GB/s），支持 FP16 和 Metal Performance Shaders，通过 llama.cpp 的 Metal 后端可高效运行量化模型，在消费级本地推理上竞争力强。
- **高通 Snapdragon 8 Gen 3**：NPU 支持 FP16 和 INT8，峰值约 **45 TOPS**，适合手机端 7B 以下小模型的量化推理。

% -------------------------------------------------------

</details>

**12. 大模型生成时的参数怎么设置？**

<details>
<summary>💡 查看答案</summary>

大模型的生成参数控制着解码策略（Decoding Strategy），直接影响输出的质量、多样性和速度。以下是各参数的详细说明与调参建议：

1. **温度（Temperature）**：
    在 Softmax 之前将 Logits 除以温度 $T$，控制概率分布的平滑程度：
    \[
    p_i = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}
    \]
    - $T \to 0$：输出趋近确定性（等同于 Greedy Decoding），始终选最高概率词，适合代码生成、数学推导等需要精确性的场景。
    - $T = 1.0$：保持模型原始分布，平衡多样性与准确性。
    - $T > 1.0$：分布更均匀，创意性更强但容易产生无意义输出，适合头脑风暴、创意写作。
    - 实践推荐：对话/问答取 $0.6\sim0.8$，创意生成取 $0.9\sim1.2$，代码生成取 $0.1\sim0.4$。

2. **Top-k 采样**：
    每步仅从概率最高的 $k$ 个 Token 中采样，舍弃长尾低概率词汇，防止模型生成低质量词。
    - $k = 1$：等同于贪心解码；$k$ 越大，多样性越强但质量风险越高。
    - 常用值：$k = 40\sim50$（GPT-2 默认）或 $k = 20\sim30$（质量优先）。
    - 缺点：固定 $k$ 无法适应分布的动态变化——当模型高度确定时（概率集中），$k=50$ 仍可能引入噪音；当模型不确定时，$k=50$ 可能范围不够。

3. **Top-p 采样（Nucleus Sampling）**：
    动态选取概率累积超过 $p$ 的最小词汇集合进行采样，自适应地调整候选集大小。
    \[
    V_p = \arg\min_{V'} \left\{ \sum_{i \in V'} p_i \geq p \right\}
    \]
    - $p = 0.9$：选取累积概率达 $90%$ 的 Token 集合，高确定性时集合小（可能只有 2\~{}3 个词），低确定性时集合大，比 Top-k 更灵活。
    - $p = 1.0$：不做任何过滤；$p = 0.0$：等同贪心解码。
    - 实践推荐：$p = 0.85\sim0.95$，通常与 Temperature 联用（先 Temperature 缩放，再 Top-p 过滤）。

4. **Repetition Penalty（重复惩罚）**：
    对已生成过的 Token 的 Logits 施加惩罚系数 $r > 1$，降低重复出现的概率，缓解模型的"复读机"问题。
    - 典型值：$r = 1.1\sim1.3$；过大（$> 1.5$）可能导致模型刻意回避合理的词汇。
    - 适用场景：长文本生成、对话场景，代码生成中有时不适用（关键字本来就需要重复）。

5. **最大生成长度（`max_new_tokens`）**：
    限制模型最多生成的新 Token 数量，防止无限循环或超出上下文窗口。
    - 需结合任务类型设置：对话回复 $64\sim256$，摘要 $128\sim512$，代码生成 $256\sim2048$。
    - 注意与 `max_length`（输入+输出总长度）的区别，避免 prompt 过长时截断输出。

6. **Beam Search（束搜索）**：
    同时维护 $B$ 条候选序列（Beam），每步扩展后保留累积概率最高的 $B$ 条，最终输出全局概率最高的序列。
    - 适合需要高质量单一输出的场景（机器翻译、摘要），不适合多样性要求高的场景。
    - $B = 4\sim8$ 为常用值；$B$ 越大质量越好但速度越慢（推理时间约为 $B\times$ 贪心解码）。
    - 大模型推理中，Beam Search 因 KV Cache 需维护 $B$ 份副本而显存开销大，实际部署中更多使用采样策略。

7. **KV Cache**：
    并非严格的生成"参数"，而是影响推理效率的关键机制——缓存历史 Token 的 Key/Value 矩阵，避免在每个 Decode 步重新计算整个序列的 Attention，将 Decode 时间复杂度从 $O(S^2)$ 降至 $O(S)$（$S$ 为当前序列长度）。推理框架（vLLM、TGI、TensorRT-LLM）默认开启，无需手动配置，但需要为其预留足够显存。

#### 常见场景推荐配置

\begin{lstlisting}[style=promptstyle]
	# 创意写作（多样性优先）
	temperature=1.1, top_p=0.92, top_k=50, repetition_penalty=1.1

	# 对话/问答（平衡）
	temperature=0.7, top_p=0.9, top_k=40, repetition_penalty=1.05

	# 代码生成（准确性优先）
	temperature=0.2, top_p=0.95, top_k=10, repetition_penalty=1.0

	# 摘要/翻译（确定性优先）
	num_beams=4, temperature=1.0, do_sample=False
\end{lstlisting}

% -------------------------------------------------------

</details>

**13. LLM 推理相比普通 Transformer 推理有什么特殊性？**

<details>
<summary>💡 查看答案</summary>

LLM 推理与普通 Transformer 推理（如 BERT 式编码器推理）在架构、计算模式和工程挑战上存在本质差异。

#### 计算模式的根本差异：自回归解码

普通 Transformer 推理（如 BERT 做分类/NER）是**一次性前向传播**：将整个输入序列并行处理，输出结果。而 LLM 推理（GPT 系列、LLaMA 等 Decoder-only 架构）是**自回归逐 Token 生成**：每次前向传播只生成一个 Token，该 Token 又作为下一步的输入，循环 $S_{out}$ 次才能完成一次推理。

这一差异带来了以下特殊性：

1. **两阶段推理模式（Prefill + Decode）**：
    - **Prefill（预填充）阶段**：一次性处理完整 Prompt，计算所有输入 Token 的 KV 并缓存，生成第一个输出 Token。此阶段为批量矩阵乘法，**计算密集（Compute-bound）**，GPU 利用率高。
    - **Decode（解码）阶段**：逐 Token 生成，每步只有一个新 Token 需要计算 Attention（与所有历史 Token 的 KV 做内积），退化为矩阵-向量乘（GEMV），**访存密集（Memory-bound）**，GPU 计算单元大量闲置，瓶颈在 HBM 带宽。
    这导致 LLM 推理的优化策略与普通 Transformer 截然不同：Prefill 优化算法复杂度，Decode 优化内存访问效率。

2. **KV Cache 的引入与管理**：
    普通 Transformer 推理无需 KV Cache（一次性前向，无历史状态）。LLM 推理必须维护 KV Cache 以避免每步重复计算历史序列，带来大量额外显存占用（随序列长度线性增长）和显存管理工程复杂度（PagedAttention、KV 量化、KV 卸载等）。

3. **模型规模与层数的量级差异**：
    BERT-Large 约 340M 参数，24 层；而当前主流 LLM 从 7B 到 405B 参数，层数从 32 层到 126 层，单模型推理的显存需求从 MB 级跃升至百 GB 级，必须引入量化、分布式推理、Offloading 等工程手段。

4. **算子实现的演化与特化**：
    LLM 中的 Attention、FFN 等算子语义与 BERT 相同，但实现层面发生了大量特化：
    - **FlashAttention**：针对 Decode 阶段的单 Token Attention 进行 IO 优化，避免写入 $N \times N$ 注意力矩阵；
    - **Paged KV Cache**：将 KV Cache 以页为单位管理（vLLM），避免显存碎片；
    - **GEMV 专用优化**：Decode 阶段矩阵乘法退化为 GEMV，需专门的 Kernel（如 `CUTLASS` 的 GEMV 优化或 `marlin` 的 INT4 GEMV）；
    - **Continuous Batching**：动态调度不同长度的请求，避免固定批处理的显存浪费和等待延迟；
    - **算子融合（Kernel Fusion）**：将 RMSNorm + Linear、Attention + Softmax 等连续算子融合为单个 CUDA Kernel，减少显存读写次数（TensorRT-LLM、vLLM 的 Fused Kernels）。

5. **服务化与延迟/吞吐量的双重优化目标**：
    普通 Transformer 推理通常是离线批处理，优化目标单一（最大化吞吐量）。LLM 推理服务需同时满足：
    - **Time to First Token（TTFT）**：用户提交请求到收到第一个回复 Token 的时间，主要受 Prefill 速度影响；
    - **Time Per Output Token（TPOT）**：每生成一个 Token 的平均时间，主要受 Decode 速度（HBM 带宽）影响；
    - **吞吐量（Throughput）**：单位时间内服务的总 Token 数，通过大 batch 和连续批处理提升。
    延迟与吞吐量之间存在权衡（大 batch 提升吞吐但增加延迟），需要根据业务场景进行调优。

6. **长上下文的特殊挑战**：
    随着 LLM 上下文窗口从 4K 扩展到 128K 乃至 1M Token，标准 Attention 的 $O(N^2)$ 计算和 $O(N)$ KV Cache 开销成为主要瓶颈，催生了 FlashAttention、RingAttention（序列并行）、稀疏注意力、线性注意力等一系列专门针对长上下文的推理优化技术，这在普通 Transformer 推理中几乎不存在。

![大模型推理 Prefill (预填充) 与 Decode (解码) 双阶段的硬件瓶颈与特征对比图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_prefill_vs_decode.png)

</details>

