# 3.9 Inference Frameworks and Deployment (vLLM, TensorRT) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 7 questions

**1. Explain Continuous Batching**

<details>
<summary>💡 Show Answer</summary>

![vLLM Continuous Batching iteration-level scheduling compared to Static Batching.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig50_vllm_continuous_batching.png)




**Background Issue**: Traditional static batching requires all requests within a batch to start and finish simultaneously (padded to the longest sequence length), leading to:
- Large variances in generation length across requests, leaving GPU resources idle after short requests finish while waiting for long ones.
- New requests must wait until the current batch completely finishes before joining, negatively impacting both throughput and latency.

**Continuous Batching (also known as iteration-level scheduling)**:
- Refines the scheduling granularity from the "entire batch" down to "each decode step (iteration)".
- After generating each token, the scheduler checks status: completed requests are immediately removed from the batch and return results, while new requests from the waiting queue are dynamically inserted into the batch.
- As a result, the batch composition can change at every iteration, maintaining high GPU utilization at all times.

**Problems Solved**:
1. Eliminates GPU idle time caused by inconsistent request lengths ("head-of-line blocking").
2. Significantly boosts throughput (empirically several-fold improvement in systems like vLLM).
3. Reduces average latency, as new requests do not need to wait for the entire batch to complete.

Implementation-wise, it relies on technologies like PagedAttention to manage dynamically changing KV Caches (where KV Cache lengths and counts vary over time across different requests).

</details>

**2. GPTQ vs AWQ vs GGUF**

<details>
<summary>💡 Show Answer</summary>

**GPTQ (GPT Quantization)**:
- Based on the Optimal Brain Quantization (OBQ) concept, it quantizes weights layer-by-layer and column-by-column, using Hessian matrix information to progressively compensate for quantization errors on the remaining unquantized weights
- Requires a calibration dataset to compute the Hessian
- Primarily **weight-only quantization** (W4A16 is common); activations remain FP16 during inference, requiring a dequantization kernel

**AWQ (Activation-aware Weight Quantization)**:
- Core observation: The small fraction of channels in the weights corresponding to "salient activation values" has a greater impact on model accuracy
- Instead of applying special treatment to these important weights (avoiding the complexity of mixed precision), it uses **per-channel scaling**: enlarging channels corresponding to important weights and shrinking channels corresponding to activations to minimize overall quantization error, with scaling factors obtained via search
- Also weight-only; the calibration process is lighter (no Hessian computation required), typically achieving superior accuracy and faster quantization speed compared to GPTQ

**GGUF (GPT-Generated Unified Format, llama.cpp ecosystem)**:
- Not a quantization algorithm, but a **model file format** that defines how to store quantized weights, model metadata, tokenizers, etc., supporting multiple quantization schemes (various bit-widths and block/group strategies such as Q4_0, Q4_K_M, Q5_K, Q8_0)
- Primarily targeted at CPU inference and edge/on-device deployment (llama.cpp), emphasizing cross-platform compatibility and ease of use
- The quantization algorithm itself is relatively simple (block-based linear quantization + partial mixed precision); accuracy is usually slightly lower than GPTQ/AWQ, but it possesses the most complete deployment ecosystem

**Summary**: GPTQ and AWQ are quantization algorithms (targeted at GPU inference frameworks such as vLLM/TensorRT-LLM), while GGUF is a file format + quantization scheme (targeted at llama.cpp/CPU scenarios); AWQ typically outperforms GPTQ in accuracy and speed, whereas GGUF excels in cross-platform deployment convenience.

</details>

**3. LLM Inference Serving System Design**

<details>
<summary>💡 Show Answer</summary>

**Objectives**: Multi-user concurrency, low latency (TTFT, TPOT), high throughput (QPS, tokens/s).

**Overall Architectural Layers**:

1. **Request Ingestion Layer**: API Gateway receives requests, handles authentication, rate limiting, and request queuing
2. **Scheduling Layer**:
- Uses a **Continuous Batching** scheduler (such as vLLM's scheduler) to dynamically assemble batches at the iteration level
- Optional **PD Disaggregation**: Prefill and decode are deployed on separate GPUs/clusters; the prefill side pursues high throughput (compute-bound, suitable for large batches), while the decode side pursues low latency (memory-bound, batch size affects throughput but latency must be controlled)
- Request priority management: Mixed scheduling of long and short requests to prevent long requests from blocking short requests (chunked prefill)

1. **GPU Memory Management**:
- **PagedAttention**: Partition KV Cache into fixed-size blocks allocated on demand, similar to OS virtual memory paging, avoiding GPU memory fragmentation and over-reservation
- Support **Prefix Caching** (RadixAttention): Reuse KV Cache for identical prefixes (e.g., system prompts, multi-turn conversation history) to reduce redundant computation

1. **Parallelism Strategies**: Select TP (intra-node) / PP (inter-node) / EP (Expert Parallelism for MoE models) based on model size, balancing actual model size and latency SLA
2. **Computation Optimizations**:
- Use fused kernels such as FlashAttention
- Quantization (AWQ/GPTQ/FP8) to reduce weight memory access and GPU memory footprint
- Speculative decoding to reduce single-user per-token latency

1. **Multi-replica and Load Balancing**:
- Scale multiple inference instances horizontally, with front-end load balancing (intelligent routing considering each instance's current KV Cache utilization and queue length)
- Auto-scaling to handle traffic fluctuations

1. **Observability**: Monitor core metrics such as TTFT (Time to First Token), TPOT (Time per Output Token), GPU utilization, GPU memory footprint, and queueing duration to dynamically adjust scheduling strategies

**Core Trade-offs**: Throughput and latency are trade-offs (larger batches increase throughput but increase latency for individual requests), requiring dynamic adjustments of batch size caps and scheduling priorities based on SLAs.

</details>

**4. Chunked Prefill**

<details>
<summary>💡 Show Answer</summary>

**Principle**: Divide the prefill computation of a long prompt into **multiple chunks**, where each chunk individually participates in the batch scheduling of a decode iteration, rather than completing the entire prefill all at once.

Specific approach: Assuming a prefill length of 4096 and chunk_size=512, the prefill for this request is split into 8 chunks. Each chunk forms part of a single forward pass, mixed into a batch with decode tokens from other requests, and gradually completed over 8 iterations.

**Pain Points Addressed**:

1. **Long prefill blocking decode (Head-of-Line Blocking)**: In traditional scheduling, a prefill for an ultra-long prompt requires exclusive execution of a complete forward pass (which may take far longer than a single decode step). During this time, all requests currently decoding must wait, causing severe jitter in their TPOT (per-token latency). Chunked Prefill splits long prefills so they can be interleaved and executed together with decode requests, smoothing latency spikes.
2. **Improving GPU utilization / batch efficiency**: By mixing compute-bound prefill chunks and memory-bound decode tokens in the same batch (i.e., "piggybacking"), the high compute density of prefill can "fill in" the idle GPU compute units during the decode phase, improving overall hardware utilization (this is also the motivation for combining Chunked Prefill with Continuous Batching in systems like vLLM).
3. **More controllable scheduling**: By setting an upper limit on chunk_size, the scheduler can more precisely control the total computation per iteration, preventing large fluctuations in single-iteration execution time and making latency more predictable.

</details>

**5. RadixAttention Applicable Scenarios**

<details>
<summary>💡 Show Answer</summary>

The core of **RadixAttention** (proposed by SGLang) is using a **Radix Tree (compact prefix tree)** to manage the KV Cache of multiple requests, automatically identifying and reusing KV Caches with **identical prefixes** to avoid redundant computation.

**Scenarios with Significant Efficiency Gains**:

1. **Multi-turn Conversation**: The history of previous turns in the same session serves as a shared prefix for subsequent turns. A new turn's request only needs to compute the KV Cache for the newly added tokens, directly reusing the historical portion from the radix tree
2. **Shared System Prompt Scenarios**: A large number of concurrent requests use the same system prompt/instruction template (e.g., customer service chatbots, Agent frameworks); the KV Cache for this prefix is computed only once and shared across all requests
3. **Few-shot Prompting / ICL**: Identical few-shot examples are reused across multiple different queries, allowing the KV Cache of the example portion to be cached and shared
4. **Parallel Sampling / Best-of-N / Beam Search**: Generating multiple distinct outputs for the same prompt (sampling multiple candidates); the prompt portion has identical KV Cache while only the generated portions are independent, significantly reducing redundant prefills
5. **Tree-of-Thought / Agent Branch Exploration**: Different branches in the search tree share the KV Cache along their prefix path

**Core Benefits**: Reduces redundant prefill computation (saving compute) and decreases GPU memory footprint of duplicate KV Caches (managing cache eviction in the radix tree via strategies like LRU). In scenarios with high prefix repetition rates, throughput can be increased multi-fold; if there are virtually no shared prefixes among requests (e.g., completely independent requests with drastically different content), the benefits are negligible.

</details>

**6. Load Balancing Strategies in PD Disaggregation**

<details>
<summary>💡 Show Answer</summary>

**Background**: Under the PD disaggregation architecture, the prefill cluster and decode cluster scale independently, requiring separate load balancing while also considering KV Cache transfer and request assignment coordination between them.

**Common Strategies**:

1. **Queue Length / Load-Based Routing**:
- Upon arrival, requests are routed to the prefill instance with the **shortest waiting queue** or **earliest estimated completion time**
- Decode instances receive new requests based on their current **KV Cache utilization / remaining GPU memory** and **number of active requests**

1. **Prefix Cache Affinity-Based Routing (Cache-Aware Routing)**:
- Prioritize routing requests to instances that already cache the KV Cache for the same prefix (combined with RadixAttention / Prefix Caching) to reduce redundant computation
- Conflicts with pure load balancing: May lead to high load on instances with cache hits, requiring a trade-off between "cache hit benefits" and "load balancing"

1. **Predictive Scheduling (Estimating Load Based on Request Features)**:
- Estimate a request's prefill/decode resource consumption based on prompt length and expected generation length (estimated via request parameters or historical statistics), making more rational allocations in advance

1. **Dynamic Resource Pool Partitioning**:
- Dynamically adjust the allocation of GPU counts between the two pools based on the current load ratio between the prefill and decode sides (e.g., temporarily adding prefill instances during traffic peaks with heavy prefill requests)

**Means to Prevent Load Imbalance**:
- **Consistent Hashing + Virtual Nodes**: Introduce consistent hashing on top of cache-aware routing to prevent hotspot prefixes from concentrating on a single instance
- **Periodic Load Rebalancing**: Monitor actual load on each instance (GPU utilization, GPU memory usage, queue depth) and periodically trigger request migration or routing weight adjustments
- **Capacity Upper Bounds + Overflow Routing**: Once single-instance load exceeds a threshold, route new requests to sub-optimal instances (even if cache misses occur), sacrificing partial cache hit rate for overall balance
- **Reserving Buffer Resources**: Reserve a small pool of "general-purpose" instances that do not participate in cache affinity binding, specifically handling overflow requests during traffic spikes

</details>

**7. KV Cache Transfer Optimization in PD Disaggregation**

<details>
<summary>💡 Show Answer</summary>

**Background**: Under PD disaggregation, the KV Cache computed during the prefill phase needs to be transferred from the prefill instance to the decode instance. The latency of this transfer directly adds to the Time to First Token (TTFT), making it a key performance bottleneck in the PD disaggregation architecture.

**Optimization Techniques**:

1. **High-Speed Interconnect Transfer (RDMA/NVLink)**:
- Use RDMA (such as InfiniBand, RoCE) to enable direct memory access between GPUs, avoiding multiple copies of data through CPU memory (GPUDirect RDMA) and significantly reducing transfer latency
- Within a node, NVLink can be leveraged for higher bandwidth transfers

1. **Overlapping Transfer and Computation (Pipelining)**:
- Transfer KV Cache asynchronously as soon as it is generated layer-by-layer, without waiting for the prefill of all layers to complete. The decode side can begin preparing upon receiving the first few layers of KV, running in parallel with the transfer of subsequent layers
- Similarly, transfers can be chunked in segments (combined with Chunked Prefill)

1. **Compressed Transfer**:
- Quantize and compress the KV Cache (such as INT8/FP8) prior to transfer to reduce transmitted data volume; the decode side dequantizes upon reception (or directly stores and uses it in low precision)

1. **KV Cache-Aware Scheduling / Pre-allocation**:
- Decode instances pre-reserve GPU memory space in advance (based on seq_len and other information returned by the prefill side), avoiding secondary waiting caused by memory allocation delays after transfer arrival
- Combined with Prefix Cache mechanisms, if the decode side already holds the KV Cache for identical prefixes (from prior requests), transfer of that portion can be skipped, transmitting only the incremental portion

1. **Dedicated Transfer Engines / Libraries**:
- Utilize specially optimized transfer frameworks (such as Mooncake's Transfer Engine, NIXL, etc.) to uniformly manage KV Cache migration across various interconnect media (NVLink/RDMA/TCP), automatically selecting optimal paths and performing multi-channel concurrent transfers

**Objective**: Conceal the KV Cache transfer time as much as possible behind prefill computation or execution of subsequent steps, avoiding it becoming an additive overhead to TTFT.

</details>

