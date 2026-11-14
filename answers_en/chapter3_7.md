# 3.7 Inference Process — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 13 questions

**1. What Do Cache Hit and Cache Miss Mean?**

<details>
<summary>💡 Show Answer</summary>

In LLM APIs, “cache hit” and “cache miss” describe whether the computable portion of the current request has already been computed and saved in a reusable form within the system.

**Request-level caching**: The system normalizes and hashes the prompt; if an identical request was previously executed and its output is still valid, historical results are returned directly (zero forward computation).

**Prefix-level caching**: The system caches intermediate states of the KV cache corresponding to a certain prefix. A “hit” means that a segment of the current request's prefix appeared in a prior request, allowing the system to reuse it directly. A request might hit the cache for its first 2,000 tokens, while subsequent tokens suffer a miss.

Cache hits are directly linked to three metrics: response latency, compute cost, and throughput stability.

</details>

**2. Do Large Models Currently Use Left-Padding During the Inference Phase?**

<details>
<summary>💡 Show Answer</summary>

During the inference phase, decoder-only large language models typically prefer left-padding. This is because when generating text, the model relies on the logits of the last valid token in the sequence to determine the next step's output. If right-padding is used, the last token is likely to be `<pad>`, leading to incorrect results. Left-padding ensures that the final token of the sequence is always a real, meaningful token.

Additionally, left-padding aligns sequences of varying lengths to the right, enabling KV-cache reuse and improving inference efficiency.

</details>

**3. Does Setting Temperature to 0 Guarantee Deterministic Output?**

<details>
<summary>💡 Show Answer</summary>

Although setting the temperature to 0 theoretically eliminates randomness, in practice, LLM outputs may still exhibit minor variations due to several factors:
- **MoE Routing Effects**: Batching contention may cause different routing decisions.
- **Floating-Point Rounding Errors**: Limited precision in floating-point operations.
- **Parallelism and Race Conditions**: Parallel operations on GPUs may introduce non-determinism.

</details>

**4. Why Can't G4-turbo Guarantee Prediction Stability Even with topp=0.0?**

<details>
<summary>💡 Show Answer</summary>

Even when setting `top_p=0.0`, actual sampling is still subject to interference from the following factors:
- Residual effects of the temperature parameter
- Floating-point precision limitations
- Numerical errors in parallel computing
- Cumulative errors in Transformer architectures

</details>

**5. Why Are Outputs Different When the Same Prompt Is Copied 5 Times into an LLM?**

<details>
<summary>💡 Show Answer</summary>

Using greedy decoding should theoretically yield consistent results. Reasons for inconsistency include:
1. **Error Accumulation**: Stochastic or floating-point sensitivity in operations inside the model (such as layer normalization) can affect results.
2. **Floating-Point Inaccuracy**: Precision errors in large-scale matrix operations gradually accumulate.
3. **Impact of Padding Position**: Extremely small values at padding positions may be amplified in deep models.

</details>

**6. How to Handle Long-Term Dependencies in Language Models?**

<details>
<summary>💡 Show Answer</summary>

Long-term dependencies in language models refer to capturing relationships between words that are far apart in a sequence. Standard neural networks typically perform poorly in this regard because information decays over time. The following are standard techniques for addressing long-term dependencies:

- **Recurrent Neural Networks (RNNs)**: These networks contain internal loops, allowing them to store information from previous inputs and use it to influence the processing of current inputs. This form of “memory” can bridge the gap between words that are far apart.
- **Long Short-Term Memory Networks (LSTMs)**: This is an RNN architecture specifically designed to solve the vanishing gradient problem. LSTMs contain internal gates that control the flow of information, enabling them to retain relevant information for longer periods and process longer-distance dependencies.
- **Transformer-XL**: This is an enhancement of the Transformer architecture, another powerful neural network for language tasks. Transformer-XL addresses long-term dependencies by retaining and reusing hidden states from previous processing steps when encoding new input segments. This enables it to maintain context across much longer sequences.
- **Attention Mechanisms**: These techniques allow the model to focus on specific parts of the input sequence that are most relevant to the current prediction. Even when these parts are far apart, attending to these information segments helps the model learn long-term dependencies.

</details>

**7. Why Does GPU Memory Surge Significantly and Remain Occupied During LLM Inference?**

<details>
<summary>💡 Show Answer</summary>

![KV Cache memory allocation dynamics during Prefill and Decode inference phases.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig42_kv_cache_memory_layout.png)




High GPU memory consumption that remains occupied during LLM inference is the result of multiple compounding factors, rather than a single cause.

#### Primary Sources of GPU Memory Consumption

1. **Model Parameter Weights**: Large models have massive parameter counts, and the weights themselves form the baseline memory footprint. Taking the LLaMA-3 70B model as an example, under FP16 precision, the parameters alone occupy approximately $70 \times 2 = 140$ GB of GPU memory. This portion of memory remains **permanently occupied** throughout the entire inference process and cannot be released, because every forward pass requires accessing all weights.

2. **KV Cache (Key-Value Cache)**: This is the core reason GPU memory “surges” during the inference phase. In autoregressive decoding, generating each new token requires computing attention with the Key and Value matrices of all previous tokens. To avoid redundant computation, frameworks cache the $K$ and $V$ matrices of historical tokens.

    The GPU memory consumption of the KV Cache is approximately:
    \[
    KV Cache = 2 \times L \times H \times d_h \times S \times B \times bytes
    \]
    where $L$ is the number of layers, $H$ is the number of attention heads, $d_h$ is the dimension per head, $S$ is the generated sequence length, $B$ is the batch size, and $bytes$ is the bytes per element. Taking LLaMA-3 70B, FP16, batch=1, and sequence length 4096 as an example, the KV Cache occupies approximately **8 GB**, growing linearly with sequence length—which is the main reason GPU memory continuously surges during inference.

3. **Intermediate Activations**: During forward propagation, the input and output tensors of each layer (with shape $[B, S, d_{model}]$) must be temporarily stored in GPU memory to compute the current layer. Since backpropagation is not required during inference, activation values can be freed after each layer completes its computation; however, peak memory moments (when multiple layers have active tensors) still introduce additional GPU memory pressure.

4. **Framework Runtime Overhead**: CUDA context (approx. 300\~{}500 MB), NCCL communication buffers (during multi-GPU inference), cuDNN/cuBLAS workspaces, and pre-allocated memory fragments in PyTorch's memory pool all occupy GPU memory, which typically is not released until the process exits.

5. **GPU Memory Release Latency**: After inference completes, PyTorch's caching allocator does not immediately return GPU memory to the CUDA driver; instead, it retains it in the memory pool for reuse. As a result, the GPU memory usage observed in `nvidia-smi` may be far higher than the actual current usage, creating the illusion that it is “permanently occupied.” Only when calling `torch.cuda.empty_cache()` or exiting the process is GPU memory truly returned.

#### Typical Curve of GPU Memory Consumption Over Time

\begin{lstlisting}[style=promptstyle]
	Timeline →
	[Model Loading] → GPU memory jumps to parameter baseline (e.g., 140 GB)
	[Prefill Phase] → Processing input prompt, KV Cache initialized, GPU memory increases slightly
	[Decode Phase, Token-by-Token Generation] → KV Cache grows linearly with sequence length, GPU memory continuously surges
	[Generation Complete] → GPU memory remains high (framework cache, unreleased)
	[empty_cache()] → GPU memory drops back to parameter baseline
\end{lstlisting}

% -------------------------------------------------------

</details>

**8. High GPU Memory Usage During LLM Inference: Are There Solutions?**

<details>
<summary>💡 Show Answer</summary>

Corresponding optimization techniques exist for different sources of GPU memory usage:

1. **Disable Gradient Computation**: During inference, use context managers like `torch.no_grad()` or `torch.inference_mode()` to prevent PyTorch from constructing computation graphs and storing gradient information. `inference_mode` is more thorough than `no_grad` because it also disables version tracking, saving more GPU memory, making it the preferred choice for inference scenarios.

2. **Explicitly Release Intermediate Variables**: Explicitly `del` large tensors that are no longer needed, and call `torch.cuda.empty_cache()` to return unallocated memory from PyTorch's cache pool to the CUDA driver. Note: `empty_cache()` cannot free tensors that are still referenced, so ensure variables have no active references first.

3. **Move Outputs to CPU**: In streaming generation scenarios, promptly move hidden states or logits of generated tokens to CPU (`.cpu()`) to prevent GPU memory from continuously expanding due to accumulated outputs.

4. **KV Cache Optimizations**:
    - **PagedAttention (vLLM)**: Dynamically allocate KV Cache in fixed-size “pages,” avoiding full memory pre-allocation for maximum sequence lengths and boosting memory utilization from $\sim 30%$ to over $90%$.
    - **KV Cache Quantization**: Store KV Cache in INT8 or INT4 format (e.g., using `transformers`' `cache_implementation="quantized"`), reducing memory usage by $2\sim4\times$ with usually acceptable precision loss.
    - **Sliding Window Attention**: Cache only the KV for the most recent $W$ tokens and discard historical KV outside the window, reducing KV Cache size from $O(S)$ to $O(W)$. The Mistral series adopts this approach.
    - **Multi-Query Attention (MQA) / Grouped-Query Attention (GQA)**: Reduce the number of KV heads (where all Query heads share a few KV heads) to fundamentally decrease KV Cache size. LLaMA-3, Qwen2, and others adopt GQA.

5. **Model Parameter Offloading**:
    - Use **ZeRO-Inference** (DeepSpeed) or **Accelerate**'s `device_map` functionality to place weights of certain layers in CPU memory or NVMe disk and transfer them to GPU on demand for computation, trading inference speed for lower GPU memory consumption.
    - **llama.cpp** supports offloading a portion of layers to the GPU (`--n-gpu-layers`) while computing remaining layers on the CPU, achieving hybrid GPU/CPU inference.

6. **Mixed-Precision Inference**: Use lower-precision formats like BF16 or FP16 for storage and computation, reducing memory usage by $50%$ compared to FP32. BF16 is currently the recommended default inference precision (sharing the dynamic range of FP32 without needing loss scaling). For more aggressive scenarios, FP8 (natively supported on H100) can be used.

7. **Model Quantization Compression**:
    - **GPTQ / AWQ**: Post-Training Quantization (INT4/INT3) compresses model size by $4\sim8\times$ with minimal loss in precision; for example, a quantized 70B model can run on a single A100 80G GPU.
    - **GGUF (llama.cpp)**: Supports multiple quantization levels ranging from Q2_K to Q8_0, suitable for CPU and hybrid inference scenarios.
    - **bitsandbytes**: Provides `load_in_8bit` and `load_in_4bit` options, seamlessly integrating with HuggingFace `transformers` to enable quantized loading with a single line of code.

8. **Batching and Request Scheduling Optimization**:
    - Use **Continuous Batching** to dynamically schedule requests, avoiding pre-allocating KV Cache memory for waiting requests;
    - Set a reasonable upper limit for `max_model_len` to prevent a single ultra-long request from consuming excessive KV Cache.

% -------------------------------------------------------

</details>

**9. How Does LLM Inference Speed Compare Between GPUs and CPUs?**

<details>
<summary>💡 Show Answer</summary>

#### GPU Inference

The massively parallel architecture of GPUs (thousands of CUDA Cores + Tensor Cores) makes them ideally suited for the massive volume of matrix multiplication (GEMM) operations in Transformers.

- **Prefill Phase (Compute-Bound)**: When processing the input prompt, QKV computations for all tokens can be executed in parallel. The GPU's parallel advantage is fully leveraged, Tensor Core utilization is high, and arithmetic intensity (FLOP/Byte) is high, typically reaching $40%\sim80%$ of the GPU's peak compute capacity.
- **Decode Phase (Memory-Bound)**: Only one token is generated at a time, degrading matrix multiplication into matrix-vector multiplication (GEMV). Arithmetic intensity is extremely low ($\sim 1\sim2$ FLOP/Byte), leaving GPU compute units largely idle, with the primary bottleneck being HBM bandwidth. The HBM bandwidth of the A100 is 2~TB/s, and the H100 is 3.35~TB/s. This is why large-batch inference significantly boosts Decode throughput (multiple requests sharing a single weight load).
- **Typical Speed**: On an A100 80G GPU, the Decode speed of LLaMA-3 70B (BF16) is approx. $20\sim30$ tokens/s (batch=1), reaching $40\sim60$ tokens/s after FP8 quantization.

#### CPU Inference

CPUs have fewer cores (typically $16\sim128$ cores), but strong single-thread performance per core, making them suitable for small models or low-precision quantized inference.

- **SIMD Acceleration**: Modern CPUs (Intel Xeon, AMD EPYC) support instruction sets like AVX-512, which can process 16 FP32 or 32 FP16/BF16 data elements simultaneously, providing particularly prominent acceleration for quantized models (INT8/INT4).
- **Memory Bandwidth**: CPU DDR5 memory bandwidth is approx. $200\sim400$~GB/s, far below the A100's HBM bandwidth (2~TB/s), which is the fundamental reason for slow CPU Decode speed.
- **Typical Speed**: On a dual-socket Xeon (96 cores), llama.cpp with Q4_K_M quantized LLaMA-3 8B achieves approx. $10\sim20$ tokens/s, while the 70B model reaches approx. $1\sim3$ tokens/s.
- **Optimization Tools**: llama.cpp (highly optimized CPU inference engine supporting AVX2/AVX-512/AMX), OpenVINO (Intel-specific), ONNX Runtime (cross-platform).

#### GPU vs. CPU Speed Comparison (Taking LLaMA-3 8B as an Example)

\begin{center}


| **Environment** | **Precision** | **Decode Speed (tokens/s)** |
| --- | --- | --- |
| A100 80G | BF16 | $\sim$120 |
| RTX 4090 | FP16 | $\sim$80 |
| A100 80G | INT8 | $\sim$200 |
| Dual Xeon 96C | Q4_K_M | $\sim$15 |
| Apple M3 Max | Q4_K_M | $\sim$50 (Unified memory bandwidth advantage) |


\end{center}

Apple Silicon (M-series chips), due to its unified memory architecture (CPU and GPU sharing high-bandwidth memory, approx. 400~GB/s), performs exceptionally well in small-model CPU/GPU hybrid inference, making it a popular choice for consumer-level local inference.

% -------------------------------------------------------

</details>

**10. How to Specifically Implement GPU-CPU Hybrid Inference?**

<details>
<summary>💡 Show Answer</summary>

![GPU-CPU hybrid inference memory offloading and PCIe transfer architecture.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig43_gpu_cpu_hybrid_inference.png)




The core idea of GPU-CPU hybrid inference is to keep compute-intensive operations on the GPU while offloading the portion exceeding VRAM capacity to CPU memory. By utilizing PCIe/NVLink bandwidth to transfer data between the two, it enables the inference of larger-scale models under VRAM-constrained conditions.

#### Heterogeneous Compute Workload Allocation

1. **Layer-wise Offloading**: Place the first $k$ layers of the model on the GPU and the remaining $L-k$ layers on the CPU (or NVMe), sequentially transferring layer weights to execute computation during inference. `device_map="auto"` in `Accelerate` automatically determines $k$ based on available VRAM. The cost is that PCIe transfer (approx. 16\~{}32 GB/s) becomes a bottleneck, which may cause decoding speed to drop by $5\sim20\times$.

2. **Partitioning by Operation Type**:
    - Prefill phase (batched tokens, high matrix parallelism) $\to$ Executed entirely on GPU to fully utilize Tensor Cores;
    - Decode phase (single token, GEMV operation) $\to$ Computation for certain layers or KV Cache executed on CPU to reduce GPU VRAM footprint;
    - Data preprocessing (Tokenization) and postprocessing (Sampling, Beam Search) $\to$ Executed on CPU to save GPU compute resources.

3. **Hierarchical Storage for KV Cache**:
    - KV Cache of currently active requests $\to$ GPU HBM (fastest, lowest latency);
    - Inactive or historical KV Cache $\to$ CPU memory (bandwidth approx. 200 GB/s, latency approx. $10\times$ that of GPU);
    - Ultra-long historical context $\to$ NVMe SSD (bandwidth approx. 7 GB/s, only used for extreme long-context scenarios).
        Both LLM.int8() and FlexGen adopt similar KV Cache tiering strategies.

#### Memory Hierarchy Optimization

- **NUMA Awareness**: On multi-socket CPU servers, ensure that model weights and KV Cache are allocated on the memory of NUMA nodes directly connected to the GPU, avoiding additional latency caused by cross-NUMA accesses (cross-NUMA bandwidth is approximately $50%$ of local bandwidth).
- **Pinned Memory**: Allocate CPU-side weights and KV Cache in pinned memory (`torch.cuda.HostAllocator` or `pin_memory=True`), allowing PCIe DMA transfers to bypass the operating system paging mechanism and increasing bandwidth by about $2\times$.
- **Prefetching**: While the GPU is executing the computation for layer $k$, asynchronously transfer the weights of layer $k+1$ from CPU to GPU (via a separate CUDA Stream). Hiding transfer latency behind computation time is key to performance optimization in hybrid inference.

#### Software Stack Optimization

- **FlexGen** (academic framework): Designed specifically for hybrid inference on a single GPU + large CPU memory + NVMe. It uses linear programming to automatically solve for the optimal layer allocation and scheduling policy, supporting OPT-175B inference on a single RTX 3090.
- **llama.cpp**: Specifies the number of GPU layers via `--n-gpu-layers N`, executing remaining layers on CPU to achieve flexible GPU/CPU hybridization. Supporting the GGUF quantization format, it is the most mature tool for local deployment.
- **DeepSpeed ZeRO-Inference**: Supports streaming weights from CPU/NVMe to GPU on demand, executing computation and transfer in parallel, with good compatibility for HuggingFace models.

#### Hardware Configuration Optimization

- Prioritize plugging the GPU into PCIe slots directly connected to the CPU (PCIe 5.0 x16, bandwidth approx. 64 GB/s) to avoid bandwidth halving caused by intermediate routing through PCIe switches;
- Use 8-channel DDR5-4800 memory (theoretical bandwidth approx. 307 GB/s) to provide ample bandwidth for CPU-side computation and data prefetching;
- If budget permits, use NVLink (NVIDIA NVLink Switch) to connect multiple GPUs into a unified VRAM pool (NVSwitch, bidirectional bandwidth 900 GB/s) to fundamentally alleviate single-card VRAM limits.

% -------------------------------------------------------

</details>

**11. Which Hardware Has the Best Support for FP16?**

<details>
<summary>💡 Show Answer</summary>

Hardware support quality for FP16 (IEEE 754 half-precision floating-point) varies significantly, mainly reflected across three dimensions: dedicated compute units, peak throughput, and numerical handling capacity.

#### NVIDIA GPUs (Most Comprehensive Support)

- **H100/H800**: Hopper architecture, peak FP16 Tensor Core performance reaches **989 TFLOPS** (SXM5 version), with native support for FP8 (Transformer Engine). It is currently the commercial GPU with the strongest FP16 inference performance. It supports dynamic switching between FP16 and FP8 via Transformer Engine to maximize throughput while guaranteeing precision.
- **A100/A800**: Ampere architecture, peak FP16 Tensor Core performance of **312 TFLOPS**, HBM2e bandwidth of 2 TB/s. It is currently the mainstream data center GPU for LLM inference. Supports sparsity acceleration, with peak sparse FP16 reaching up to 624 TFLOPS.
- **RTX 4090**: Ada Lovelace architecture, peak FP16 performance around **165 TFLOPS**, 24 GB GDDR6X (bandwidth 1 TB/s), suitable for local FP16 inference of medium-scale models (7B\~{}13B) with extremely high cost-performance ratio.
- **RTX 3090**: Ampere architecture, peak FP16 performance around **71 TFLOPS**, 24 GB GDDR6X, capable of running 7B FP16 models or 30B INT4 quantized models.
- **Note**: FP64 performance on consumer GPUs (RTX series) is deliberately restricted (approx. $\frac{1}{64}$ of FP32), but FP16 performance is unrestricted, making them highly suitable for inference scenarios.

#### Intel CPUs/GPUs

- **Intel Xeon Scalable (4th Gen+)**: Supports BF16 via the **AVX-512 BF16** instruction set (Note: native FP16 support is weak; the primary optimization target is BF16), as well as **AMX (Advanced Matrix Extensions)** to accelerate INT8 and BF16 matrix operations, with peak performance reaching approx. **7 TFLOPS** (BF16), making it suitable for large-scale CPU cluster inference.
- **Intel Arc GPU / Gaudi2**: Gaudi2's peak FP16 performance is approx. **432 TFLOPS**. It is Intel's main product targeting AI inference, supporting FP16/BF16/FP8, though its software ecosystem still lags behind NVIDIA.

#### AMD GPUs

- **MI300X**: CDNA3 architecture, peak FP16 performance of **1307 TFLOPS**, HBM3 bandwidth of **5.3 TB/s**, 192 GB HBM3. It is currently the AI accelerator card with the highest single-card bandwidth, holding a clear theoretical advantage during the bandwidth-bound Decode phase. Supports ROCm ecosystem, with PyTorch compatibility steadily improving.

#### Edge/Mobile Chips

- **NVIDIA Orin-X (Jetson AGX Orin)**: Supports INT8 and FP16 inference, with peak FP16 performance around **275 TOPS** (INT8 equivalent) and TDP of only 60W, suitable for edge inference in low-power scenarios such as autonomous driving and robotics.
- **Apple Silicon (M3/M4 series)**: Unified memory architecture (CPU+GPU shared memory, bandwidth 300\~{}400 GB/s), supports FP16 and Metal Performance Shaders. Efficiently runs quantized models via llama.cpp's Metal backend, highly competitive for consumer-grade local inference.
- **Qualcomm Snapdragon 8 Gen 3**: NPU supports FP16 and INT8, with peak performance around **45 TOPS**, suitable for quantized inference of small models under 7B on mobile devices.

% -------------------------------------------------------

</details>

**12. How to Set Generation Parameters for Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

The generation parameters of large language models control the decoding strategy, directly affecting the output quality, diversity, and speed. Below are detailed descriptions and tuning recommendations for each parameter:

1. **Temperature**:
    Divides logits by temperature $T$ prior to Softmax to control the smoothness of the probability distribution:
    \[
    p_i = \frac{\exp(z_i / T)}{\sum_j \exp(z_j / T)}
    \]
    - $T \to 0$: Output approaches determinism (equivalent to Greedy Decoding), always choosing the token with the highest probability. Suitable for scenarios requiring precision, such as code generation and mathematical reasoning.
    - $T = 1.0$: Maintains the model's original distribution, balancing diversity and accuracy.
    - $T > 1.0$: Makes the distribution more uniform, boosting creativity but prone to generating nonsensical outputs. Suitable for brainstorming and creative writing.
    - Recommended values in practice: $0.6\sim0.8$ for dialogue/QA, $0.9\sim1.2$ for creative generation, $0.1\sim0.4$ for code generation.

2. **Top-k Sampling**:
    Samples only from the top $k$ highest-probability tokens at each step, discarding long-tail low-probability vocabulary to prevent generating low-quality words.
    - $k = 1$: Equivalent to greedy decoding; larger $k$ yields higher diversity but carries greater quality risks.
    - Common values: $k = 40\sim50$ (GPT-2 default) or $k = 20\sim30$ (quality prioritized).
    - Disadvantage: A fixed $k$ cannot adapt to dynamic changes in distribution—when the model is highly confident (probabilities concentrated), $k=50$ may still introduce noise; when the model is uncertain, $k=50$ may offer insufficient coverage.

3. **Top-p Sampling (Nucleus Sampling)**:
    Dynamically selects the smallest set of tokens whose cumulative probability exceeds $p$ for sampling, adaptively adjusting candidate set size.
    \[
    V_p = \arg\min_{V'} \left\{ \sum_{i \in V'} p_i \geq p \right\}
    \]
    - $p = 0.9$: Selects the token set reaching $90%$ cumulative probability. When confidence is high, the candidate set is small (possibly only 2\~{}3 words); when confidence is low, the set is large, making it more flexible than Top-k.
    - $p = 1.0$: No filtering applied; $p = 0.0$: Equivalent to greedy decoding.
    - Recommended values in practice: $p = 0.85\sim0.95$, typically combined with Temperature (apply Temperature scaling first, then Top-p filtering).

4. **Repetition Penalty**:
    Applies a penalty coefficient $r > 1$ to the logits of previously generated tokens to reduce their probability of recurrence, alleviating the model's "parrot / repetition" issue.
    - Typical values: $r = 1.1\sim1.3$; excessively large values ($> 1.5$) may cause the model to deliberately avoid valid words.
    - Applicable scenarios: Long-text generation and dialogue scenarios; sometimes inapplicable in code generation (where keywords naturally recur).

5. **Maximum Generation Length (`max_new_tokens`)**:
    Limits the maximum number of new tokens generated by the model to prevent infinite loops or exceeding context windows.
    - Should be set according to task type: $64\sim256$ for dialogue responses, $128\sim512$ for summarization, $256\sim2048$ for code generation.
    - Pay attention to the distinction from `max_length` (total length of input + output) to avoid truncating output when the prompt is excessively long.

6. **Beam Search**:
    Maintains $B$ candidate sequences (beams) concurrently, keeping the $B$ sequences with the highest cumulative probability after each expansion step, and finally outputs the sequence with the highest overall probability.
    - Suitable for scenarios requiring a single high-quality output (machine translation, summarization), but not suitable for scenarios requiring high diversity.
    - $B = 4\sim8$ are commonly used values; larger $B$ yields better quality but slower speed (inference time is roughly $B\times$ greedy decoding).
    - In LLM inference, Beam Search incurs heavy VRAM overhead because KV Cache must maintain $B$ copies, so sampling strategies are more widely used in actual deployment.

7. **KV Cache**:
    Not strictly a generation "parameter", but a critical mechanism affecting inference efficiency—it caches the Key/Value matrices of historical tokens to avoid recomputing Attention for the entire sequence at every Decode step, reducing Decode time complexity from $O(S^2)$ to $O(S)$ ($S$ being the current sequence length). Inference frameworks (vLLM, TGI, TensorRT-LLM) enable it by default without requiring manual configuration, though sufficient VRAM must be reserved for it.

#### Recommended Configurations for Common Scenarios

\begin{lstlisting}[style=promptstyle]
	# Creative writing (Diversity prioritized)
	temperature=1.1, top_p=0.92, top_k=50, repetition_penalty=1.1

	# Dialogue/QA (Balanced)
	temperature=0.7, top_p=0.9, top_k=40, repetition_penalty=1.05

	# Code generation (Accuracy prioritized)
	temperature=0.2, top_p=0.95, top_k=10, repetition_penalty=1.0

	# Summarization/Translation (Determinism prioritized)
	num_beams=4, temperature=1.0, do_sample=False
\end{lstlisting}

% -------------------------------------------------------

</details>

**13. What Are the Peculiarities of LLM Inference Compared to Standard Transformer Inference?**

<details>
<summary>💡 Show Answer</summary>

![Comparison of Prefill phase (compute-bound) and Decode phase (memory-bound) in LLM inference.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig44_prefill_vs_decode_phases.png)




LLM inference differs fundamentally from standard Transformer inference (such as BERT-style encoder inference) in architecture, computation patterns, and engineering challenges.

#### Fundamental Difference in Compute Pattern: Autoregressive Decoding

Standard Transformer inference (e.g., BERT for classification/NER) relies on a **one-shot forward pass**: it processes the entire input sequence in parallel and outputs the result. In contrast, LLM inference (Decoder-only architectures such as GPT series, LLaMA, etc.) relies on **autoregressive token-by-token generation**: each forward pass generates only a single token, which is then fed back as input for the next step, looping $S_{out}$ times to complete a single inference.

This difference leads to the following peculiarities:

1. **Two-Stage Inference Pattern (Prefill + Decode)**:
    - **Prefill Phase**: Processes the complete prompt in a single shot, computes and caches Key/Value states for all input tokens, and generates the first output token. This phase consists of batched matrix multiplications, making it **compute-bound** with high GPU utilization.
    - **Decode Phase**: Generates tokens one by one. In each step, only a single new token requires Attention computation (dot product with KV of all historical tokens), degrading to matrix-vector multiplication (GEMV). It is **memory-bound**, leaving GPU compute units largely idle with the bottleneck lying in HBM bandwidth.
    As a result, optimization strategies for LLM inference differ drastically from standard Transformers: Prefill optimizes algorithmic complexity, while Decode optimizes memory access efficiency.

2. **Introduction and Management of KV Cache**:
    Standard Transformer inference requires no KV Cache (one-shot forward pass without historical states). LLM inference must maintain KV Cache to avoid recomputing historical sequences at each step, bringing significant additional VRAM footprint (growing linearly with sequence length) and engineering complexity for VRAM management (PagedAttention, KV quantization, KV offloading, etc.).

3. **Orders of Magnitude Difference in Model Scale and Layer Count**:
    BERT-Large has approx. 340M parameters and 24 layers; current mainstream LLMs range from 7B to 405B parameters with layer counts from 32 to 126 layers. The VRAM requirement for single-model inference leaps from megabyte scale to hundreds of gigabytes, making engineering techniques such as quantization, distributed inference, and offloading indispensable.

4. **Evolution and Specialization of Operator Implementations**:
    While operators such as Attention and FFN in LLMs share identical semantics with BERT, extensive specialization has occurred at the implementation level:
    - **FlashAttention**: IO-optimized for single-token Attention in the Decode phase to avoid writing the $N \times N$ attention matrix;
    - **Paged KV Cache**: Manages KV Cache in page units (vLLM) to prevent memory fragmentation;
    - **GEMV Dedicated Optimization**: Matrix multiplication in the Decode phase degrades to GEMV, requiring specialized kernels (such as `CUTLASS` GEMV optimizations or `marlin` INT4 GEMV);
    - **Continuous Batching**: Dynamically schedules requests of varying lengths to avoid VRAM waste and waiting latency inherent to fixed batching;
    - **Kernel Fusion**: Fuses consecutive operators such as RMSNorm + Linear or Attention + Softmax into a single CUDA kernel, reducing memory read/write operations (TensorRT-LLM, vLLM Fused Kernels).

5. **Serving and Dual Optimization Objectives of Latency vs. Throughput**:
    Standard Transformer inference is usually offline batch processing with a single optimization goal (maximizing throughput). LLM inference serving must concurrently satisfy:
    - **Time to First Token (TTFT)**: The duration from request submission to receiving the first response token, primarily governed by Prefill speed;
    - **Time Per Output Token (TPOT)**: The average duration to generate each token, primarily governed by Decode speed (HBM bandwidth);
    - **Throughput**: The total number of tokens served per unit time, improved via large batch sizes and continuous batching.
    A tradeoff exists between latency and throughput (large batch sizes increase throughput but add latency), requiring fine-tuning based on business scenarios.

6. **Special Challenges of Long Contexts**:
    As LLM context windows expand from 4K to 128K or even 1M tokens, the $O(N^2)$ computation and $O(N)$ KV Cache overhead of standard Attention become the primary bottleneck. This has spawned a series of inference optimization techniques specifically targeting long context, such as FlashAttention, RingAttention (sequence parallelism), sparse attention, and linear attention, which are virtually non-existent in standard Transformer inference.

</details>

