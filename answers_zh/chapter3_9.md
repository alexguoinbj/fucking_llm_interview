# 3.9 推理框架与部署（vLLM、TensorRT） — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 3 章：预训练、推理与工程优化 · 共 7 题

**1. Continuous Batching请解释下**

<details>
<summary>💡 查看答案</summary>

**背景问题**：传统static batching要求一个batch内所有请求必须同时开始、同时结束（按最长序列pad），导致：
- 不同请求生成长度差异大，短请求完成后GPU资源闲置等待长请求
- 新请求必须等当前batch全部结束才能加入，吞吐和延迟都受影响

**Continuous Batching（连续批处理，也叫iteration-level scheduling）**：
- 调度粒度从"整个batch"细化到"每个decode step（iteration）"
- 每完成一个token的生成，调度器检查：已完成的请求立即移出batch并返回结果，同时将等待队列中的新请求动态插入batch
- 因此batch组成在每个iteration都可能变化，始终保持GPU的高利用率

**解决的问题**：
1. 消除了因请求长度不一致导致的GPU空闲（"队头阻塞"）
2. 大幅提升吞吐量（vLLM等系统实测可提升数倍）
3. 降低平均延迟，新请求不需等待整批结束

实现上依赖PagedAttention等技术管理动态变化的KV Cache（不同请求的KV Cache长度和数量随时间变化）。

![vLLM PagedAttention 虚拟内存块映射及非连续物理内存管理原理图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_paged_attention.png)

</details>

**2. GPTQ vs AWQ vs GGUF**

<details>
<summary>💡 查看答案</summary>

**GPTQ（GPT Quantization）**：
- 基于Optimal Brain Quantization (OBQ) 思想，逐层、逐列地量化权重，并利用Hessian矩阵信息逐步补偿量化误差到剩余未量化的权重上
- 需要校准数据集（calibration dataset）来计算Hessian
- 主要是**weight-only量化**（W4A16常见），推理时激活仍为FP16，需要反量化kernel

**AWQ（Activation-aware Weight Quantization）**：
- 核心观察：权重中对应"激活值较大（salient）"的那一小部分channel对模型精度影响更大
- 不对这些重要权重做特殊处理（避免混合精度的复杂性），而是通过**逐channel缩放（scaling）**：放大重要权重对应的channel、缩小对应激活的channel，使整体量化误差最小化，缩放因子通过搜索得到
- 同样是weight-only，校准过程更轻量（不需要计算Hessian），通常精度优于GPTQ且量化速度快

**GGUF（GPT-Generated Unified Format，llama.cpp生态）**：
- 不是一种量化算法，而是一种**模型文件格式**，定义了如何存储量化后的权重、模型元数据、tokenizer等，支持多种量化方案（Q4_0, Q4_K_M, Q5_K, Q8_0等多种位宽和分组策略）
- 主要面向CPU推理和端侧部署（llama.cpp），强调跨平台兼容性和易用性
- 量化算法本身相对简单（基于分组的线性量化+部分混合精度），精度通常略低于GPTQ/AWQ，但部署生态最完善

**总结**：GPTQ和AWQ是量化算法（针对GPU推理框架如vLLM/TensorRT-LLM），GGUF是文件格式+量化方案（针对llama.cpp/CPU场景）；AWQ通常精度和速度优于GPTQ，GGUF胜在跨平台部署便利性。

</details>

**3. 大模型推理服务系统设计**

<details>
<summary>💡 查看答案</summary>

**目标**：多用户并发、低延迟（TTFT、TPOT）、高吞吐（QPS、tokens/s）。

**整体架构层次**：

1. **请求接入层**：API Gateway接收请求，做认证、限流、请求排队
2. **调度层**：
- 使用**Continuous Batching**调度器（如vLLM的scheduler），按iteration粒度动态组batch
- 可选**PD分离**：prefill和decode部署在不同GPU/集群，prefill侧追求高吞吐（compute-bound适合大batch），decode侧追求低延迟（memory-bound，batch大小影响吞吐但需控制延迟）
- 请求优先级管理：长短请求混合调度，避免长请求阻塞短请求（chunked prefill）

1. **显存管理**：
- **PagedAttention**：将KV Cache划分为固定大小的block，按需分配，类似OS的虚拟内存分页，避免显存碎片和过度预留
- 支持**Prefix Caching**（RadixAttention）：复用相同前缀（如系统prompt、多轮对话历史）的KV Cache，减少重复计算

1. **并行策略**：根据模型大小选择TP（节点内）/PP（跨节点）/EP（MoE模型的专家并行），结合模型实际大小和延迟SLA权衡
2. **计算优化**：
- 使用FlashAttention等融合kernel
- 量化（AWQ/GPTQ/FP8）降低权重访存和显存占用
- 投机解码降低单用户的逐token延迟

1. **多副本与负载均衡**：
- 多个推理实例水平扩展，前端做负载均衡（考虑各实例当前KV Cache占用、队列长度做智能路由）
- 自动扩缩容应对流量波动

1. **可观测性**：监控TTFT（首token延迟）、TPOT（每token时间）、GPU利用率、显存占用、排队时长等核心指标，用于动态调整调度策略

**核心矛盾与权衡**：吞吐与延迟此消彼长（更大batch提升吞吐但增加单请求延迟），需要根据SLA设定动态调整batch大小上限和调度优先级。

</details>

**4. Chunked Prefill**

<details>
<summary>💡 查看答案</summary>

**原理**：将一个长prompt的prefill计算**切分成多个chunk**，每个chunk分别参与一次decode迭代的batch调度，而不是一次性完成整个prefill。

具体做法：假设prefill长度为4096，chunk_size=512，则该请求的prefill被拆成8个chunk，每个chunk作为一次forward的一部分，与其他请求的decode token混合组成batch，分8次迭代逐步完成。

**解决的痛点**：

1. **长prefill阻塞decode（队头阻塞）**：传统调度中，一个超长prompt的prefill需要独占一次完整forward（耗时可能远超一次decode），期间所有正在decode的请求都要等待，导致这些请求的TPOT（单token延迟）剧烈抖动。Chunked Prefill将长prefill拆分，使其能与decode请求"插空"混合执行，平滑了延迟尖峰。
2. **提升GPU利用率/batch效率**：通过将compute-bound的prefill chunk和memory-bound的decode token混合在同一个batch中（即"piggybacking"），可以让prefill的高计算密度"填补"decode阶段GPU计算单元的空闲，提高整体硬件利用率（这也是vLLM等系统Chunked Prefill+Continuous Batching结合的motivation）。
3. **更可控的调度**：通过设置chunk_size上限，调度器可以更精确地控制每次迭代的总计算量，避免单次迭代耗时波动过大，使延迟更可预测。

</details>

**5. RadixAttention适用场景**

<details>
<summary>💡 查看答案</summary>

**RadixAttention**（SGLang提出）的核心是用**Radix Tree（基数树/压缩前缀树）**管理多个请求的KV Cache，自动识别和复用具有**相同前缀**的KV Cache，避免重复计算。

**大幅提升效率的场景**：

1. **多轮对话（Multi-turn Conversation）**：同一会话的历史轮次内容是后续轮次的公共前缀，新一轮请求只需计算新增部分的KV，历史部分直接从radix tree中复用
2. **共享System Prompt场景**：大量并发请求使用相同的system prompt/instruction模板（如客服机器人、Agent框架），这部分前缀的KV Cache只需计算一次，所有请求共享
3. **Few-shot Prompting / ICL**：相同的few-shot示例被多个不同query复用，示例部分的KV可缓存共享
4. **并行采样/Best-of-N/Beam Search**：对同一个prompt生成多个不同的输出（采样多个候选），prompt部分的KV完全相同，只有生成部分各自独立，可大幅减少重复prefill
5. **Tree-of-Thought / Agent的分支探索**：搜索树中不同分支共享前缀路径上的KV Cache

**核心收益**：减少重复的prefill计算（节省compute）、减少重复KV Cache的显存占用（通过LRU等策略管理radix tree的缓存淘汰）。在前缀重复率高的场景下，可实现数倍的吞吐提升；若请求间几乎没有共同前缀（如完全独立、内容差异巨大的请求），收益则不明显。

</details>

**6. PD分离中的负载均衡策略**

<details>
<summary>💡 查看答案</summary>

**背景**：PD分离架构下，prefill集群和decode集群分别独立扩展，需要分别做负载均衡，同时还需考虑两者之间的KV Cache传输和请求分配协同。

**常见策略**：

1. **基于队列长度/负载的路由**：
- 请求到达后，路由到当前**等待队列最短**或**预估完成时间最早**的prefill实例
- decode实例根据当前**KV Cache占用率/显存余量**和**正在处理的请求数**接收新请求

1. **基于Prefix Cache亲和性的路由（cache-aware routing）**：
- 优先将请求路由到已经缓存了相同前缀KV Cache的实例（结合RadixAttention/Prefix Caching），减少重复计算
- 与纯负载均衡存在冲突：可能导致cache命中的实例负载过高，需要权衡"cache命中收益"与"负载均衡"

1. **预测式调度（基于请求特征预估负载）**：
- 根据prompt长度、预期生成长度（可由请求参数或历史统计估计）预估该请求对prefill/decode资源的占用，提前做更合理的分配

1. **动态资源池划分**：
- 根据当前prefill/decode两侧的负载比例，动态调整两个池子的GPU数量分配（如流量高峰时prefill请求多则临时增加prefill实例）

**避免负载倾斜的手段**：
- **一致性哈希+虚拟节点**：在cache-aware路由基础上引入一致性哈希，避免热点前缀集中到单一实例
- **定期负载rebalance**：监控各实例的实际负载（GPU利用率、显存占用、队列深度），周期性触发请求迁移或调整路由权重
- **设置容量上限+溢出路由（overflow routing）**：单实例负载超过阈值后，新请求路由到次优（即使cache不命中）的实例，牺牲部分cache命中率换取整体均衡
- **预留缓冲资源**：保留少量"通用"实例不参与cache亲和绑定，专门处理负载尖峰时的溢出请求

</details>

**7. PD分离中KV Cache传输优化**

<details>
<summary>💡 查看答案</summary>

**背景**：PD分离下，prefill阶段计算出的KV Cache需要从prefill实例传输到decode实例，这个传输过程的延迟会直接叠加到首token延迟（TTFT）中，是PD分离架构的关键性能瓶颈。

**优化手段**：

1. **高速互联传输（RDMA/NVLink）**：
- 使用RDMA（如InfiniBand、RoCE）实现GPU间直接内存访问，避免数据经过CPU内存的多次拷贝（GPU Direct RDMA），大幅降低传输延迟
- 节点内可利用NVLink实现更高带宽传输

1. **传输与计算overlap（流水线化）**：
- KV Cache按层（layer-by-layer）生成后立即异步传输，无需等待全部层的prefill完成。decode侧可以在收到前几层KV后就开始准备，与后续层的传输并行
- 类似地，可以按chunk（结合Chunked Prefill）分段传输

1. **压缩传输**：
- 对KV Cache进行量化压缩（如INT8/FP8）后传输，减少传输数据量，decode侧接收后再反量化（或直接以低精度存储使用）

1. **KV Cache感知的调度/预先分配**：
- decode实例提前预留显存空间（基于prefill侧返回的seq_len等信息），避免传输到达后因显存分配延迟造成的二次等待
- 结合Prefix Cache机制，如果decode侧已经持有相同前缀的KV Cache（来自之前的请求），可以跳过该部分的传输，只传输增量部分

1. **专用传输引擎/库**：
- 使用专门优化的传输框架（如Mooncake的Transfer Engine、NIXL等），统一管理多种互联介质（NVLink/RDMA/TCP）的KV Cache搬运，自动选择最优路径并做多路并发传输

**目标**：使KV Cache的传输时间尽可能被prefill计算或后续步骤的执行时间掩盖，避免成为TTFT的额外开销叠加项。

</details>

