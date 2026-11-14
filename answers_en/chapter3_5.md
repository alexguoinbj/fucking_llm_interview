# 3.5 Distributed Training: Parallel Strategies and Memory Optimization — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 58 questions

**1. What Is Point-to-Point Communication?**

<details>
<summary>💡 Show Answer</summary>

Point-to-Point Communication refers to a mode of data transmission directly between two communication nodes, typically used for simple data exchange between devices in a distributed system.

#### Basic Characteristics

- **Low Latency**: Data is transmitted directly between two nodes without routing through intermediate nodes or broadcasting, resulting in low latency.
- **High Bandwidth Utilization**: Communication involves only two endpoints, fully utilizing link bandwidth.
- **Flexibility**: Sender and receiver can independently control communication timing and data volume.
- **Limited Scalability**: As the number of nodes increases, if every pair of nodes requires point-to-point communication, the total communication volume scales as $O(N^2)$, making it unsuitable for large-scale data transfer.

#### Applications in Distributed Training

Point-to-point communication in distributed training is mainly used for:
- **Activation Passing in Pipeline Parallelism**: Devices between adjacent pipeline stages pass forward activations and backward gradients via point-to-point communication, serving as the core communication primitive for pipeline parallelism.
- **Targeted Transfer of Model Parameters**: In Parameter Server (PS) architectures, workers pushing gradients to and pulling parameters from specific PS nodes are all point-to-point communications.
- **Dynamic Routing and MoE**: In Mixture-of-Experts (MoE) models, tokens are routed to devices where specific experts reside, requiring point-to-point All-to-All communication.

#### Common Implementations

In NCCL (NVIDIA Collective Communications Library), point-to-point communication is implemented via `ncclSend` and `ncclRecv`; in PyTorch, it corresponds to `dist.send` and `dist.recv`, as well as the non-blocking versions `dist.isend` / `dist.irecv`.

% -------------------------------------------------------

</details>

**2. What Is Collective Communication?**

<details>
<summary>💡 Show Answer</summary>

![Collective communication primitives: AllReduce, AllGather, ReduceScatter, and AllToAll.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig31_collective_communication_primitives.png)




Collective Communication refers to a data transmission mode where multiple communication nodes transfer data simultaneously according to a specific pattern. Unlike point-to-point communication, collective communication operations involve all processes within a communication group, where each process can act as both sender and receiver.

#### Common Collective Communication Primitives

- **Broadcast**: Broadcasts data from a single root node (Root) to all nodes in the group. Commonly used during initialization to distribute model weights from the main process to all workers.
- **Reduce**: Reduces data across all nodes in the group according to a specific operation (e.g., sum, max) onto a single Root node.
- **AllReduce**: Reduces data across all nodes in the group and distributes the result to all nodes; it is the core primitive for synchronizing gradients in data parallelism. Taking Ring-AllReduce as an example, the communication volume is $2(N-1)/N \cdot M$ ($N$ being the number of nodes and $M$ being the data size), which is independent of the number of nodes and has excellent scalability.
- **AllGather**: Each node sends its data to all other nodes, resulting in all nodes having the concatenated results of all data. ZeRO-3 uses AllGather to gather full parameters during forward propagation.
- **ReduceScatter**: First performs a Reduce on the data from all nodes, and then distributes the reduced chunks to each respective node. ZeRO-3 uses ReduceScatter to distribute gradient shards during backpropagation.
- **All-to-All**: Each node sends different data to every other node, commonly used for token routing in MoE models and dimension transposition in tensor parallelism.
- **Scatter / Gather**: Scatter distributes chunks of data from Root to each node; Gather aggregates data from each node to Root; they are inverse operations of each other.

#### Applications in Distributed Training

Collective communication is widely used in various stages of distributed training:
- Gradient synchronization in data parallelism (AllReduce);
- Parameter/gradient shard aggregation in the ZeRO family (AllGather + ReduceScatter);
- Merging computation results across devices in tensor parallelism (AllReduce or AllGather);
- Expert routing in MoE models (All-to-All).

Mainstream implementation libraries include NCCL (GPU clusters), MPI, and Gloo (CPU/cross-platform), all of which are encapsulated by PyTorch's `torch.distributed`.

% -------------------------------------------------------

</details>

**3. What Is Data Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Data Parallelism (DP) is the earliest and most widely used distributed training strategy. Its core idea is to divide the training dataset into several non-overlapping shards. Each device (GPU/TPU) holds a **complete copy of the model** and processes one of the data shards. After each device independently completes forward and backward passes, gradients are synchronized via collective communication to uniformly update the model parameters.

#### Basic Workflow

1. **Initialization**: Broadcasts the model to all devices to ensure identical model parameters across devices.
2. **Data Distribution**: Evenly splits a global batch (Global Batch) into $N$ local batches (Local Batch) and distributes them to $N$ devices.
3. **Local Forward Pass**: Each device executes a forward pass using its local data to compute local loss.
4. **Local Backward Pass**: Each device independently computes local gradients.
5. **Gradient Synchronization**: Averages gradients across all devices via AllReduce (typically Ring-AllReduce) so each device obtains the global average gradient.
6. **Parameter Update**: Each device independently updates its local model parameters using the global average gradient. Because the gradients are identical, parameters remain consistent across all devices after the update.

#### Main Implementation Methods

- **Parameter Server (PS) Architecture**: Contains centralized parameter servers. Workers push gradients to the PS, which aggregates them and sends updated parameters back. Communication is point-to-point, making the PS prone to becoming a bottleneck.
- **AllReduce Architecture**: Decentralized, where all workers communicate peer-to-peer via Ring-AllReduce. PyTorch DDP and Horovod both adopt this approach, which offers higher communication efficiency and is currently the mainstream standard.

#### Limitations

The primary limitation of data parallelism is that **each device must store a full copy of the model**, including parameters, gradients, and optimizer states (approximately $16\times$ bytes of the parameter count in the case of Adam). When model parameter volume exceeds single-card GPU memory, pure data parallelism can no longer be used, requiring combination with model parallelism or techniques like ZeRO.

% -------------------------------------------------------

</details>

**4. How Does Data Parallelism Improve Efficiency?**

<details>
<summary>💡 Show Answer</summary>

Efficiency improvements from data parallelism are mainly reflected in the following dimensions:

#### Computational Efficiency

- **Linear Speedup (Ideal Case)**: $N$ devices process $N$ times the data; theoretically, training throughput (samples/second) scales linearly with the number of devices. In practice, affected by communication overhead, scaling efficiency is typically between $85%\sim95%$ (in high-speed interconnect environments).
- **Larger Effective Batch Size**: Data parallelism is equivalent to using a global batch size of $N \times LocalBatchSize$, which contributes to training stability (more accurate gradient estimation) and can be combined with the Linear Scaling Rule for learning rate to accelerate convergence.

#### Communication Optimization

- **Ring-AllReduce**: Devices are arranged in a logical ring, with each device communicating only with its neighbors. Total communication volume is $2(N-1)/N \cdot M$, which is independent of the device count $N$ ($M$ being total gradient volume), avoiding the bandwidth bottleneck of the central node in PS architectures.
- **Gradient Compression**: Reduces AllReduce communication data volume via techniques like gradient quantization (e.g., 1-bit SGD) and sparsification (Top-K gradients), trading minor precision loss for communication speedup.
- **Overlapping Computation and Communication**: During backpropagation, as soon as the gradient calculation for a parameter is complete, PyTorch DDP immediately initiates the AllReduce for that parameter, running in parallel with gradient calculations of subsequent layers to hide communication latency. This is implemented via the `autograd hook` mechanism and is one of the key factors behind DDP's efficiency.

#### Scalability and Engineering Practice

- **Gradient Accumulation**: When single-card GPU memory is limited, simulating large-batch training via gradient accumulation over multiple micro-batches is equivalent to increasing batch size without extra GPU memory overhead, which orthogonally combines with data parallelism.
- **Synergy with Mixed Precision**: Data parallelism is naturally compatible with FP16/BF16 mixed precision training. AllReduce can be performed in half precision (or half-precision computation + FP32 accumulation), further boosting communication and computation efficiency.

% -------------------------------------------------------

</details>

**5. What is Pipeline Parallelism?**

<details>
<summary>💡 Show Answer</summary>

![1F1B scheduling timeline in Pipeline Parallelism.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig32_pipeline_parallelism_schedule.png)




Pipeline Parallelism (PP) is a strategy that splits a model by **layers** into multiple sequential stages (Stages) and assigns them to different devices to execute in parallel. Each device is only responsible for a portion of the model's layers, and data flows between devices in a "relay" manner, thereby distributing a large model that exceeds single-card GPU memory across multiple cards or even multiple nodes for training.

#### Basic Principles

Taking 4 stages (4 devices) as an example:
- Stage 0 (GPU 0): Responsible for the Embedding layer and the first $L/4$ Transformer layers;
- Stage 1 (GPU 1): Responsible for the $L/4 \sim L/2$-th Transformer layers;
- Stage 2 (GPU 2): Responsible for the $L/2 \sim 3L/4$-th Transformer layers;
- Stage 3 (GPU 3): Responsible for the final $L/4$ Transformer layers and the output head.

During forward propagation, data flows from GPU 0 to GPU 3, and each GPU passes the activation values (outputs) of the current stage to the next GPU via point-to-point communication; during backward propagation, gradients flow backwards from GPU 3 to GPU 0.

#### Pipeline Bubble Problem

Naive pipeline parallelism (GPipe) suffers from severe device idle time, known as the **Pipeline Bubble**. Taking $p$ stages and $m$ micro-batches as an example, the bubble fraction is approximately:
\[
Bubble Fraction = \frac{p-1}{m + p - 1}
\]
When $m \gg p$, the bubble fraction approaches zero. Therefore, increasing the number of micro-batches is the primary means to reduce bubbles.

#### Scheduling Strategies

- **GPipe (F-then-B)**: Completes the forward propagation of all micro-batches first, and then performs backward propagation uniformly. The bubble fraction is approximately $(p-1)/m$, but it needs to cache activation values for all micro-batches, incurring high GPU memory overhead.
- **PipeDream (1F1B)**: Alternates between forward and backward propagation execution. In the steady-state phase, each device continuously has tasks executing. GPU memory usage is superior compared to GPipe with the same bubble fraction, making it the current mainstream solution (adopted by Megatron-LM).
- **Interleaved 1F1B (Virtual Pipeline)**: Each device is responsible for multiple non-contiguous layer chunks (Virtual Stages), reducing the bubble fraction to $(p-1)/(m \cdot v)$ (where $v$ is the number of virtual stages), further reducing bubbles at the cost of increased communication frequency.
- **Zero-Bubble Pipeline (ZB-H1/ZB-H2)**: By precisely scheduling the order of W (weight gradient) computation and B (activation gradient) computation, it theoretically reduces bubbles to near zero.

#### GPU Memory Analysis

The GPU memory advantage of pipeline parallelism lies in that each device only stores $\frac{1}{p}$ of the total model parameters, enabling the training of ultra-large models that originally could not fit on a single card. Its main memory overhead comes from the intermediate activation values passed along the pipeline, which can be further reduced by combining with activation recomputation.

% -------------------------------------------------------

</details>

**6. What is Tensor Parallelism (intra-layer)?**

<details>
<summary>💡 Show Answer</summary>

Tensor Parallelism (TP), also known as Intra-layer Parallelism, is a strategy that splits the weight tensors within a single layer of a model across multiple devices for parallel computation. Unlike pipeline parallelism, which splits by layers, tensor parallelism splits matrix operations **within a layer**, allowing computations to be distributed across multiple GPUs even if the parameter count of a single layer exceeds the single-card GPU memory.

#### Core Principle: Matrix Multiplication Splitting

Linear layers in Transformers are essentially matrix multiplication $Y = XW$. Tensor parallelism achieves parallelism by splitting $W$:

- **Column Parallel**: Splits $W \in \mathbb{R}^{d \times 4d}$ by columns into $[W_1, W_2, \ldots, W_N]$. Each device computes $Y_i = X W_i$, and the results are concatenated by columns (AllGather) to obtain the complete output.
- **Row Parallel**: Splits $W$ by rows. Each device holds the corresponding column slice of the input $X$, computes the local result, and sums them via AllReduce.

Megatron-LM applies column parallel and row parallel to the two linear layers in the MLP respectively, requiring only one AllReduce for the entire MLP block, resulting in extremely high communication efficiency.

#### Application in Attention

Multi-Head Attention is naturally suited for tensor parallelism: different attention heads are assigned to different devices, each device independently computes its assigned heads, and finally an AllReduce merges the outputs. If the model has $H$ heads and there are $N$ devices, each device is responsible for $H/N$ heads, and communication occurs only at the Projection layer of the Attention output.

#### Sequence Parallelism

Megatron-LM v3 introduces sequence parallelism on top of tensor parallelism: for operations that cannot be split by tensor parallelism (such as LayerNorm and Dropout), the sequence dimension is split across different devices. Combined with AllGather/ReduceScatter, it seamlessly integrates with tensor parallelism, further reducing the GPU memory footprint of activation values.

#### Scope of Application and Limitations

- **Suitable for Intra-node**: The AllReduce communication in tensor parallelism is frequent (at least once per layer), requiring high-bandwidth interconnects (such as NVLink, 600 GB/s intra-node), and is not suitable for inter-node communication (InfiniBand is around 400 Gb/s, far lower than NVLink).
- **Scalability Upper Limit**: The tensor parallel degree is usually set to the number of GPUs within a node (e.g., 8); increasing it further causes communication overhead to exceed computational gains.
- **Requires Divisible Model Dimensions**: The split dimensions must be divisible by the parallel degree, imposing constraints on the network architecture.

% -------------------------------------------------------

</details>

**7. What is 3D Parallelism?**

<details>
<summary>💡 Show Answer</summary>

3D Parallelism is a hybrid parallel training scheme that organically combines three parallel strategies: **Data Parallelism (DP), Tensor Parallelism (TP), and Pipeline Parallelism (PP)**. It was systematically proposed and implemented by NVIDIA in the Megatron-LM paper. The three dimensions address different bottlenecks respectively and work together synergistically.

#### Division of Labor Across Three Dimensions

- **Tensor Parallelism (TP)**: Splits single-layer weights across multiple cards within a node, leveraging the high bandwidth of NVLink (600 GB/s) to hide communication overhead. The TP degree is usually equal to the number of GPUs within a node (e.g., TP=8).
- **Pipeline Parallelism (PP)**: Splits the model by layers across nodes, with each node responsible for several layers, passing activation values between nodes via InfiniBand (where communication volume is far smaller than parameter size). The PP degree is usually equal to the number of nodes (e.g., PP=8, i.e., 8 nodes).
- **Data Parallelism (DP)**: Performs data parallelism across multiple TP$\times$PP groups, where each group processes different data batches and synchronizes gradients via AllReduce. ZeRO-1/2 can be stacked to further optimize GPU memory.

#### Device Allocation Illustration

Taking 64 GPUs (8 nodes $\times$ 8 cards), TP=8, PP=4, DP=2 as an example:
- 8 cards within a node perform tensor parallelism (TP=8);
- 4 nodes are connected sequentially to perform pipeline parallelism (PP=4), with each node responsible for $\frac{1}{4}$ of the model's layers;
- The remaining 4 nodes form the second data-parallel replica (DP=2), totaling $8 \times 4 \times 2 = 64$ GPUs.

#### Hierarchical Communication Design

The core design principle of 3D Parallelism is **communication localization**:
- High-frequency, high-bandwidth TP communication (AllReduce) is restricted to intra-node NVLink;
- Low-frequency, small-data-volume PP communication (point-to-point activation values) goes through inter-node InfiniBand;
- Low-frequency, large-data-volume DP gradient synchronization goes through inter-node InfiniBand, but at a low frequency (once per Global Batch).

#### GPU Memory and Computation Analysis

For a model with parameter count $\Phi$, under the configuration of TP=$t$, PP=$p$, DP=$d$ (total GPU count $N = t \times p \times d$):
- The parameter count per card is approximately $\Phi / (t \times p)$;
- The activation memory is further compressed through micro-batches and 1F1B scheduling;
- The effective global batch size is $d \times LocalBatch \times GradAccumSteps$.

% -------------------------------------------------------

</details>

**8. Can Data Parallelism, Tensor Parallelism, and Pipeline Parallelism Be Combined?**

<details>
<summary>💡 Show Answer</summary>

Yes, the three parallel approaches can not only be combined, but when training LLMs with over 100 billion parameters, combining them is **necessary** rather than optional.

#### Why Is Combination Necessary?

A single parallel strategy has its limitations:
- Pure Data Parallelism: Each card needs to store the entire model, making it helpless when single-layer parameter counts are too large.
- Pure Tensor Parallelism: Communication is frequent, bandwidth is insufficient across nodes, and scalability is limited (usually $\leq 8$).
- Pure Pipeline Parallelism: Suffers from bubble overhead and cannot handle cases where single-layer parameters are excessively large.

When combined, the three parallel methods complement each other, each handling the dimension it is best suited for.

#### Constraints of Combination

- Total GPUs $= TP \times PP \times DP$, where the product of the three equals the total number of available GPUs.
- TP degree is constrained by the number of GPUs per node and model dimensions (must be divisible); PP degree is constrained by the number of model layers (must be divisible); DP degree is determined by the remaining GPU count.
- During tuning, one must trade off the bubble overhead, communication overhead, and GPU memory footprint among the three to find the optimal combination.

#### Engineering Practice

Both Megatron-LM and DeepSpeed support combined configurations of 3D parallelism. Taking the training of GPT-3 (175B parameters) on 1024 A100 GPUs as an example, a typical configuration is TP=8, PP=16, DP=8, making full use of the bandwidth hierarchy of intra-node NVLink and inter-node InfiniBand.

% -------------------------------------------------------

</details>

**9. What Are 1D/2D/2.5D/3D Tensor Parallelisms in Colossal-AI?**

<details>
<summary>💡 Show Answer</summary>

The 1D/2D/2.5D/3D concepts in Colossal-AI specifically refer to different splitting methods for the **Tensor Parallelism** dimension. They differ in meaning from the commonly referenced "3D Parallelism" (TP+PP+DP) and represent a further subdivision of internal implementation schemes for tensor parallelism.

- **1D Tensor Parallelism**: The standard Megatron-LM style tensor parallelism, which splits the matrix along one dimension (rows or columns) across $P$ devices. It requires 1 AllReduce operation with a communication volume of $O(P)$. It is the simplest to implement and has high engineering maturity.

- **2D Tensor Parallelism**: Splits the weight matrix along both row and column dimensions across $P = \sqrt{P} \times \sqrt{P}$ devices based on the SUMMA matrix multiplication algorithm. Each device holds a $\frac{1}{P}$ fraction of the matrix, using Broadcast for communication with a communication volume of $O(\sqrt{P})$, superior to 1D's $O(P)$, but with higher implementation complexity.

- **2.5D Tensor Parallelism**: Introduces a depth dimension $d$ on top of 2D, arranging devices into a 3D grid of $d \times \sqrt{P/d} \times \sqrt{P/d}$. When $d=1$, it degenerates to 2D; as $d$ increases, the communication volume is further reduced to $O(\sqrt{P/d})$, but it requires more devices (totaling $d \times P$), representing a trade-off between communication volume and device count.

- **3D Tensor Parallelism**: Decomposes matrix multiplication into parallel computation over a 3D device grid of $\sqrt[3]{P} \times \sqrt[3]{P} \times \sqrt[3]{P}$, based on a 3D matrix multiplication algorithm. The communication volume drops to $O(P^{1/3})$, theoretically offering the best scalability, but it is the most complex to implement and requires the device count to be a perfect cube (e.g., 8, 27, 64).

#### Practical Recommendations

In engineering practice, 1D tensor parallelism (Megatron-LM style) remains the mainstream choice due to its simple implementation, ease of debugging, and excellent compatibility with NCCL's highly optimized AllReduce. 2D/2.5D/3D offer more pronounced theoretical advantages at ultra-large scales (thousands of cards or more) when communication bandwidth becomes a bottleneck, but their engineering complexity increases significantly.

% -------------------------------------------------------

</details>

**10. What is Micro-batch?**

<details>
<summary>💡 Show Answer</summary>

A micro-batch is a data processing strategy used in deep learning training and inference that further subdivides a larger batch into several smaller sub-batches for individual processing, which are then equivalent to large-batch training through mechanisms such as gradient accumulation.

#### Relationship Between Micro-batch and Related Concepts

- **Global Batch Size**: The total number of samples used in a complete training step, which determines the quality of gradient estimation.
- **Local Batch Size** (per device): The number of samples processed by each card in a single step, $= Global Batch / DP Degree$.
- **Micro-batch Size**: The smallest processing unit further split in pipeline parallelism, $Local Batch = Micro-batch \times Gradient Accumulation Steps$.

#### Role in Pipeline Parallelism

Micro-batches are the key to the efficient operation of pipeline parallelism. After splitting the Local Batch into $m$ micro-batches:
- Pipeline stages can **interleave the processing** of different micro-batches, reducing device idle time (lowering the bubble fraction from $(p-1)/1$ to $(p-1)/m$);
- The activation values for each micro-batch are smaller, allowing them to be passed individually without waiting for the entire batch, reducing pipeline latency;
- In 1F1B scheduling, while device $i$ processes the forward pass of the $k$-th micro-batch, it simultaneously processes the backward pass of the $(k-1)$-th micro-batch, achieving pipelining of forward and backward passes.

#### Main Characteristics

1. **Improve Device Utilization**: Reduces device idle time through pipelining; in the steady state, each device has tasks executing almost all the time.
2. **Optimize GPU Memory Usage**: Activation memory needs to be allocated for only one micro-batch at a time instead of the full batch, effectively relieving peak GPU memory pressure.
3. **Reduce Communication Overhead**: Only activation values of a single micro-batch are transferred between pipeline stages, keeping communication data volume small.
4. **Applicable to Large-Scale Models**: Micro-batch technology is the foundation of pipeline parallel training for ultra-large models; training models like LLaMA and GPT-3 relies on micro-batch mechanisms.

#### Selection of Micro-batch Size

If the micro-batch size is too large, the reduction in pipeline bubbles is insufficient; if it is too small, the computational granularity is inadequate, resulting in low GPU utilization (matrices are too small to fully utilize Tensor Cores). In practice, the micro-batch size is usually set to the minimum value that can fully utilize GPU compute resources (e.g., 1$\sim$4 samples), and bubbles are reduced by increasing the number of micro-batches $m$.

% -------------------------------------------------------

</details>

**11. What Are the Strategies to Reduce Pipeline Bubbles?**

<details>
<summary>💡 Show Answer</summary>

A Pipeline Bubble refers to the period in pipeline parallelism during which some devices are in a waiting state and unable to perform effective computation due to data dependencies between stages. The bubble fraction directly impacts GPU utilization and is a core optimization objective in pipeline parallelism.

1. **Increase the Number of Micro-batches $m$**: The bubble fraction is $(p-1)/(m+p-1)$; the larger $m$ is, the fewer bubbles there are. This is the most direct approach, with the trade-off being increased single-step latency and requiring more GPU memory to cache micro-batch activation values (which can be mitigated by combining with activation recomputation).

2. **Virtual Pipeline (Interleaved 1F1B)**: Each device is responsible for $v$ non-contiguous layer chunks (virtual stages), reducing the bubble fraction to $(p-1)/(m \cdot v)$, which is $\frac{1}{v}$ of basic 1F1B. The trade-off is a $v$-fold increase in communication frequency, suitable for scenarios with ample communication bandwidth. Megatron-LM adopts this strategy by default.

3. **Dual-directional Pipeline Parallelism (DualPipe, Proposed by DeepSeek-V3)**: Splits micro-batches into two directions for simultaneous pipelining. The communication phases of forward and backward passes completely overlap, theoretically reducing bubbles to near zero without increasing communication volume. It is suitable for MoE model scenarios combining expert parallelism with pipeline parallelism.

4. **Zero-Bubble Pipeline Parallelism (ZB-H1/ZB-H2)**: By decoupling weight gradient computation (W-pass) from activation gradient computation (B-pass) and finely scheduling the execution order across devices, the W-pass fills the original bubble periods. ZB-H2 can theoretically achieve zero bubbles, but requires more GPU memory to cache intermediate states.

5. **Overlap Computation and Communication**: While a device executes local computation, it asynchronously initiates activation transfer for the next micro-batch (using NCCL non-blocking communication + CUDA Stream parallelism), hiding communication latency within compute time.

6. **Optimize Task Scheduling and Load Balancing**: Ensure equal compute workload across pipeline stages (assigning the same number of layers to each stage) to prevent other stages from waiting due to a heavy workload in one stage. For MoE models, replicate high-load experts and dynamically assign tokens to reduce extra bubbles caused by expert imbalance.

7. **Speculative Execution**: Under certain conditions, allow devices to start computation in advance based on predicted inputs; if the prediction is correct, waiting time is saved (similar to CPU branch prediction, more commonly used in LLM inference).

</details>

**12. What are the methods to improve PP inference efficiency?**

<details>
<summary>💡 Show Answer</summary>

Pipeline parallelism faces different challenges during inference compared to training: during training, a large number of micro-batches can be used to fill the pipeline, but during inference, the latency requirements of single requests often limit the number of micro-batches, making the bubble problem more prominent.

1. **Reducing Pipeline Bubbles**:
    - Use continuous batching (Continuous Batching) to pack tokens from different requests into micro-batches, keeping the pipeline saturated even during the inference phase;
    - Adopt interleaved (Interleaved) scheduling to disperse bubbles and reduce single waiting times;
    - For latency-insensitive offline inference, increase the batch size to keep the pipeline full.

2. **Optimizing Communication Mechanisms**: Use efficient communication protocols such as RDMA (Remote Direct Memory Access) and InfiniBand to reduce the activation transfer latency between pipeline stages; combine with CUDA Streams to achieve overlap of communication and computation, hiding communication overhead.

3. **Micro-batching and Dynamic Batching**: In inference serving, dynamically organize micro-batches based on real-time request arrival (Dynamic Batching), maximizing the batch size within latency constraints to boost GPU utilization. Combining vLLM's continuous batching (Continuous Batching) and PagedAttention with pipeline parallelism can significantly increase inference throughput.

4. **Speculative Decoding**: Use a small model (Draft Model) to quickly generate multiple candidate tokens, and a large model (Target Model) to validate all candidates in parallel, where accepted tokens do not need to be regenerated. When combined with pipeline parallelism, the Draft Model and Target Model can be distributed across different pipeline stages, further enhancing throughput.

5. **KV Cache Optimization**:
    - In pipeline parallelism, the KV Cache is stored on the local GPUs of each stage, requiring proper planning of the KV Cache capacity at each stage;
    - Use PagedAttention (vLLM) to perform paged management of the KV Cache, reducing GPU memory fragmentation;
    - Quantize inactive KV Caches (INT8/INT4) or offload them to CPU memory to save GPU memory.

6. **Attention Mechanism Optimization**: Adopt FlashAttention to reduce GPU memory read/write overhead in Attention computation ($O(N^2) \to O(N)$), accelerating Attention computation within each pipeline stage; combine with GQA/MQA to reduce KV Cache size and lower communication volume between pipeline stages.

7. **Pipeline-Aware Request Scheduling**: Dynamically allocate requests according to the real-time load of each pipeline stage to prevent any single stage from becoming a bottleneck; for requests of varying lengths, reduce computational waste through padding or packing strategies.

% -------------------------------------------------------

</details>

**13. How to implement hybrid parallelism strategies (e.g., Data Parallelism + Model Parallelism)?**

<details>
<summary>💡 Show Answer</summary>

![Mapping hybrid 3D parallelism to GPU cluster interconnect hardware.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig33_hybrid_parallelism_3d.png)




The implementation of hybrid parallelism strategies requires coordination across multiple levels, including device grouping, communication topology design, and framework configuration.

#### Hierarchical Partitioning Design

1. **Determine Degrees of Parallelism**: Based on model size, GPU count, and network topology, determine the degrees of parallelism for the TP, PP, and DP dimensions, ensuring that $TP \times PP \times DP = Total GPU Count$.

2. **Process Group Partitioning**: Partition all GPU processes into multiple communication groups:
    - **Tensor Parallel Group**: Within the same node, a group consists of TP-degree GPUs connected via high-speed NVLink;
    - **Pipeline Parallel Group**: Across nodes, 1 GPU (or 1 TP group) from each of the PP-degree nodes is connected sequentially into a group;
    - **Data Parallel Group**: GPUs at the same pipeline position and same tensor parallel position across DP replicas form a group for gradient synchronization.

3. **Model Partitioning**: Partition the model by PP stages, and within each stage, further split by TP. Pay attention to the choice of TP split points (typically at GEMM operations in linear layers) and the shape of activation values at the interfaces between pipeline stages.

#### Communication Optimization

- Within tensor parallel groups, use AllReduce/AllGather over intra-node NVLink (high bandwidth);
- Between pipeline stages, use point-to-point communication (Send/Recv) over inter-node InfiniBand (low latency);
- Gradient synchronization between data parallel groups (AllReduce or ReduceScatter) goes over inter-node InfiniBand, which can run in parallel with pipeline backward propagation to hide latency.

#### Framework Support and Configuration Example

\begin{lstlisting}[style=promptstyle]
	# Megatron-LM configuration example
	--tensor-model-parallel-size 8   # TP=8 (intra-node)
	--pipeline-model-parallel-size 4  # PP=4 (inter-node)
	# DP is automatically determined by Total GPUs / (TP * PP)

	# DeepSpeed + Megatron configuration (ds_config.json)
	{
		"zero_optimization": { "stage": 1 },  # ZeRO-1 combined with DP
		"fp16": { "enabled": true }
	}
\end{lstlisting}

#### Important Considerations

- In pipeline parallelism, layer distribution between adjacent stages should be as balanced as possible to prevent load imbalance from introducing extra bubbles;
- When mixing tensor parallelism and pipeline parallelism, activations at pipeline boundaries are already sharded by TP, so the receiving end requires TP-degree GPUs to receive simultaneously;
- During debugging, it is recommended to verify workflow correctness using a small model on a single machine before gradually scaling up the degrees of parallelism.

% -------------------------------------------------------

</details>

**14. What are common distributed training frameworks and their features?**

<details>
<summary>💡 Show Answer</summary>

- **PyTorch DDP (Distributed Data Parallel)**:
    The official data parallelism implementation provided by PyTorch, which uses Ring-AllReduce to synchronize gradients and supports overlapping computation with communication (via the Bucket mechanism). It is currently the most widely used data parallelism solution. Suitable for multi-GPU/multi-node training of medium-sized models (that fit on a single GPU), offering a low learning curve and seamless integration with the PyTorch ecosystem.

- **PyTorch FSDP (Fully Sharded Data Parallel)**:
    The official full-parameter sharding implementation in PyTorch 2.x, with functionality similar to ZeRO-3. It supports full sharding of parameters, gradients, and optimizer states, as well as hybrid sharding strategies (full replica intra-node, sharded inter-node). Compared with DDP, it offers higher GPU memory efficiency, serves as the official training solution for Llama 2/3, and is becoming the new standard replacing DDP.

- **DeepSpeed**:
    An open-source deep learning optimization library developed by Microsoft, centered around the ZeRO family (ZeRO-1/2/3, ZeRO-Infinity, ZeRO++). It simultaneously supports pipeline parallelism, tensor parallelism (when combined with Megatron-LM), mixed precision, gradient compression, and other features. It provides a configuration-file-driven workflow and deep integration with HuggingFace Transformers, making it one of the most widely used frameworks in the current LLM training ecosystem.

- **Megatron-LM**:
    An open-source large language model training framework by NVIDIA, renowned for its highly efficient tensor parallelism and pipeline parallelism implementations, with sequence parallelism technology further reducing activation memory. The codebase is relatively low-level with strong customization capabilities, serving as an important reference implementation for training models like GPT-3 and LLaMA. It is often combined with DeepSpeed (Megatron-DeepSpeed) to simultaneously achieve 3D parallelism and ZeRO optimizations.

- **Colossal-AI**:
    Supports multiple parallel strategies (1D/2D/2.5D/3D tensor parallelism + pipeline parallelism + data parallelism) and provides high-level API encapsulations to lower the barrier to entry for complex parallel strategies. It has an active community and provides optimized training scripts for models like LLaMA and ChatGLM, making it suitable for scientific research and small-to-medium-scale engineering practices.

- **Horovod**:
    A lightweight distributed training framework open-sourced by Uber. It uses Ring-AllReduce to optimize data-parallel communication and supports multiple frameworks such as TensorFlow, PyTorch, and Keras (framework agnosticism being its primary advantage). Its support for model parallelism is weak, making it mainly suitable for rapid data-parallel deployment of medium-sized models. As PyTorch DDP and FSDP have matured, its usage scenarios have narrowed.

- **JAX + XLA**:
    Google's functional ML framework, which achieves distributed training via `jax.pmap` (inter-device parallelism) and `jax.sharding` (tensor sharding), with the XLA compiler automatically optimizing computation graphs. It is the preferred framework on TPUs, supporting extremely large-scale training (serving as the training platform for Google's internal models such as PaLM).

% -------------------------------------------------------

</details>

**15. Can you introduce the DeepSpeed library?**

<details>
<summary>💡 Show Answer</summary>

DeepSpeed is an open-source deep learning optimization library developed by Microsoft, focusing on enhancing the efficiency of large-scale model training and inference. Since its open-sourcing in 2020, it has become one of the most important tools in the LLM training ecosystem.

#### Core Technical Modules

- **ZeRO (Zero Redundancy Optimizer) Series**: DeepSpeed's most representative innovation, which eliminates memory redundancy in data parallelism by sharding optimizer states (ZeRO-1), gradients (ZeRO-2), and parameters (ZeRO-3). ZeRO-Infinity further offloads parameters to CPU memory and NVMe SSDs, theoretically supporting infinitely large models. ZeRO++ introduces quantized communication (qgZ) and hierarchical AllReduce, reducing inter-node communication volume by approximately $4\times$.

- **3D Parallelism Support**: Integrated with Megatron-LM (Megatron-DeepSpeed), it provides joint support for tensor parallelism + pipeline parallelism + ZeRO data parallelism, serving as the standard solution for training 100-billion-parameter models.

- **Mixed Precision and Communication Optimization**: Native support for FP16/BF16 mixed-precision training with dynamic loss scaling; supports gradient compression (1-bit Adam, 1-bit LAMB) to significantly reduce communication volume.

- **DeepSpeed Inference**: Optimized for inference, supporting Transformer operator fusion (combining multiple operators into a single CUDA Kernel), INT8 quantization inference, KV Cache optimization, etc., significantly improving inference throughput.

- **Pipeline Engine**: Built-in pipeline parallel engine supporting 1F1B scheduling and gradient accumulation, which can be combined with ZeRO.

#### Usage

DeepSpeed is driven by configuration files (JSON) and has minimal intrusion into training code:

\begin{lstlisting}[style=promptstyle]
	# ds_config.json
	{
		"train_batch_size": 256,
		"fp16": { "enabled": true },
		"zero_optimization": {
			"stage": 2,
			"allgather_partitions": true,
			"reduce_scatter": true,
			"overlap_comm": true
		}
	}
\end{lstlisting}

\begin{lstlisting}[style=promptstyle]
	# Minimal training code changes
	import deepspeed
	model_engine, optimizer, _, _ = deepspeed.initialize(
	model=model, optimizer=optimizer, config="ds_config.json"
	)
	model_engine(inputs).backward()
	model_engine.step()
\end{lstlisting}

#### Ecosystem and Integration

DeepSpeed is deeply integrated with HuggingFace `Trainer`, `Accelerate`, and `trl` (RLHF), supporting one-click enabling of ZeRO and mixed precision. It is currently the most commonly used acceleration tool for fine-tuning open-source models such as LLaMA and Mistral.

% -------------------------------------------------------

</details>

**16. What is the Selective Activation Recompute strategy in DeepSpeed?**

<details>
<summary>💡 Show Answer</summary>

Selective Activation Recompute is DeepSpeed's fine-grained improvement over standard Gradient Checkpointing technology. By **selectively** deciding which activations to retain and which to recompute, it achieves a better balance between memory savings and computational overhead.

#### Background: Trade-offs of Standard Activation Recomputation

Standard activation recomputation (full recomputation) recomputes all intermediate activations during backward propagation, reducing activation memory from $O(L \cdot B \cdot S)$ to $O(\sqrt{L} \cdot B \cdot S)$, but introduces approximately $33%$ extra computation (as the forward pass per token is executed twice).

#### Core Idea of Selective Activation Recompute

Not all activations have the same memory footprint and recomputation cost. DeepSpeed analyzes various activations in the Transformer layer:
- Activations with **high memory footprint and low recomputation cost**: For example, the output of Attention QKV projections (shape $[B, S, H]$, consuming large memory), but recomputation requires only a single matrix multiplication, which is cheap. These activations are **selected for recomputation** and not saved.
- Activations with **low memory footprint and high recomputation cost**: For example, the Softmax attention weight matrix (shape $[B, H, S, S]$, with memory proportional to $S^2$), where recomputation requires re-executing the entire Attention operation. These activations are **selected to be retained** to avoid recomputation.

#### Practical Impact

The typical impact of Selective Activation Recompute on Transformer models is that compared to using no recomputation, GPU memory is reduced by about $60%\sim70%$; compared to full recomputation, the additional computational overhead drops from about $33%$ to about $5%\sim10%$, achieving a superior balance between GPU memory and speed.

% -------------------------------------------------------

</details>

**17. How does FlashAttention-2 in DeepSpeed achieve memory optimization?**

<details>
<summary>💡 Show Answer</summary>

![FlashAttention-2 kernel integration and parallel warp scheduling in DeepSpeed.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig34_flashattention2_deepspeed_integration.png)




FlashAttention-2 is an efficient attention mechanism implementation proposed by Tri Dao et al. It fundamentally alters the memory access pattern of standard Attention through **IO-aware tiled computation**, reducing the memory complexity of Attention from $O(N^2)$ to $O(N)$ ($N$ is the sequence length). DeepSpeed integrates it as the `deepspeed.ops.flash_attention` module.

#### Memory Bottleneck of Standard Attention

The standard Scaled Dot-Product Attention is computed as:
\[
Attention(Q, K, V) = softmax\left(\frac{QK^T}{\sqrt{d}}\right) V
\]
where both $QK^T \in \mathbb{R}^{N \times N}$ and the Softmax output must be fully written to HBM (GPU High Bandwidth Memory), resulting in a memory footprint of $O(N^2)$. When the sequence length $N=128K$, a single attention matrix reaches tens of gigabytes.

#### Core Optimizations of FlashAttention-2

- **Tiled (Tiling) Computation**: Load blocks of $Q$, $K$, and $V$ into SRAM (on-chip high-speed cache, ~20 MB), perform tiled Attention computation inside SRAM, and only write final results back to HBM, avoiding HBM read/writes for $N \times N$ intermediate matrices. HBM access volume per operation is reduced from $O(N^2)$ to $O(N)$.
- **Online Softmax**: Since the complete $QK^T$ row cannot be seen all at once during block-wise computation, FlashAttention computes Softmax via streaming accumulation (maintaining running max values and normalization factors), obtaining numerically stable Softmax results without storing the full $N \times N$ attention matrix.
- **Backward Recomputation**: FlashAttention-2 does not store the forward attention matrix during backward propagation; instead, it recomputes attention weights from saved $Q$, $K$, and $V$ ($O(N)$ memory), trading a small amount of extra computation for massive memory savings.
- **Parallelism and Warp Optimization** (Improvements of FlashAttention-2 over v1): Optimizes task allocation within thread blocks (parallelizing along the $Q$ block dimension instead of $K$ and $V$), reducing inter-warp communication and improving measured FLOP utilization on A100 GPUs from ~25% in v1 to ~70%.

#### Practical Benefits

For a model with sequence length $N=2048$, compared to standard Attention, FlashAttention-2 provides:
- Memory: Attention memory drops from $O(N^2)$ to $O(N)$, saving approximately $5\sim10\times$;
- Speed: End-to-end training speed is boosted by $2\sim4\times$ (the primary bottleneck shifts from memory bandwidth to compute);
- Sequence Length Support: Supports much longer sequences under the same memory constraints (e.g., extending 2048 to 32K or even 128K).

% -------------------------------------------------------

</details>

**18. Comparing Distributed Training Features of DeepSpeed and Horovod**

<details>
<summary>💡 Show Answer</summary>

- **DeepSpeed**: Specially designed for LLM training (GPU memory + compute optimization), supporting ZeRO optimizations (ZeRO-1/2/3/Infinity), gradient accumulation, CPU/NVMe Offload, and comprehensive 3D parallelism (TP+PP+DP). It is deeply integrated with the HuggingFace ecosystem and is currently the mainstream choice for full training and fine-tuning of LLMs. Suitable for models ranging from billions to hundreds of billions of parameters.

- **Horovod**: Focused on general distributed training (data parallelism prioritized). Its core contribution was bringing Ring-AllReduce to deep learning with framework agnosticism (supporting TensorFlow, PyTorch, Keras) and minimal code intrusion. Its support for model parallelism is weak and lacks memory optimizations such as ZeRO, making it primarily suitable for multi-node data parallelism of medium-sized models (that fit on a single GPU).

#### Detailed Comparison

\begin{center}


| **Feature** | **DeepSpeed** | **Horovod** |
| --- | --- | --- |
| Core Optimization Goal | Memory Optimization + LLM Support | Communication Efficiency + Framework Agnosticism |
| Data Parallelism | Supported (ZeRO Enhanced) | Supported (Ring-AllReduce) |
| Model Parallelism | Comprehensive (TP+PP) | Barely Supported |
| Memory Optimization | ZeRO Series, Offload | None |
| Framework Support | Mainly PyTorch | TF/PyTorch/Keras |
| Configuration Complexity | Medium (JSON config file) | Low (A few lines of code) |
| Target Scale | Billions to Hundreds of Billions | Millions to Billions |
| Community Ecosystem | Deeply integrated with HuggingFace | Independent ecosystem, declining usage |


\end{center}

**Selection Advice**: Choose DeepSpeed if you need to train or fine-tune LLMs (that cannot fit on a single GPU); choose Horovod if you need quick cross-framework data parallelism for medium-sized models with high requirements for simplicity; if you are already in the PyTorch ecosystem, you can directly use DDP or FSDP to replace Horovod.

% -------------------------------------------------------

</details>

**19. How Does DDP Achieve Synchronization?**

<details>
<summary>💡 Show Answer</summary>

The synchronization mechanism of PyTorch DDP (DistributedDataParallel) is designed to be exquisite and efficient, using multiple levels of technical techniques to ensure that the model parameters across all processes remain consistent at all times.

#### Initialization Phase Synchronization

When DDP is initialized, it broadcasts the model parameters and buffers from the Rank 0 process to all other processes, ensuring that the initial states of all replicas are completely consistent when training begins:

\begin{lstlisting}[style=promptstyle]
	model = DDP(model, device_ids=[local_rank])
	# Automatically executed inside DDP:
	# dist.broadcast(param, src=0) for all params and buffers
\end{lstlisting}

#### Gradient Synchronization in the Backward Pass (Core Mechanism)

DDP utilizes `autograd hook` technology to embed gradient synchronization logic directly into the backward pass workflow:

1. **Register Gradient Hooks**: DDP registers an `AccumulateGrad` hook for each parameter, which is automatically triggered as soon as the gradient computation for that parameter completes.
2. **Bucket Mechanism**: DDP groups parameters in their registration order into several buckets (by default ~25 MB per bucket). Once all parameter gradients inside a bucket are ready, it triggers an AllReduce for that bucket. This merges a large number of small AllReduce operations into a few large AllReduce operations, reducing communication frequency.
3. **Ring-AllReduce**: Uses NCCL's Ring-AllReduce to sum and average gradients across all processes. The communication volume is $2(N-1)/N \cdot M$, which is independent of the number of processes.
4. **Overlap of Computation and Communication**: Since the backward pass computes layer-by-layer from the last layer backward, while AllReduce is initiated starting from the first ready bucket, the AllReduce of gradients from later layers can run in parallel with the gradient computation of earlier layers, effectively hiding communication latency.

#### Buffer Synchronization

In addition to parameter gradients, buffers such as the running mean/variance in layers like BatchNorm also need to be synchronized across processes. DDP synchronizes all buffers via Broadcast before the start of each forward pass (this can be turned off via `broadcast_buffers=False` to save overhead, in which case manual synchronization is required).

#### Why Do Parameters Remain Consistent After Update?

Since the gradients of all processes become completely identical after AllReduce, and each process uses the exact same optimizer and learning rate, the parameter update amount per step $\Delta \theta = -\eta \cdot g_{avg}$ is completely identical. Therefore, the parameters of all processes naturally stay synchronized after the update, without needing any additional communication after the update.

#### Comparison with FSDP

Each process in DDP maintains a complete replica of parameters. Gradient synchronization requires only one AllReduce, which makes communication simple but results in low memory utilization ($N$ replicas). FSDP goes beyond AllReduce by further sharding parameters (ReduceScatter + AllGather), achieving higher memory efficiency at the cost of a more complex communication pattern. Both of their gradient synchronizations are based on NCCL, sharing the same core communication primitives.

</details>

**20. What Is the Principle of Data Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Data Parallelism (DP) is the most fundamental distributed training approach: complete model parameters are replicated onto each computing device (GPU), and global batch data is split into multiple shards. Each device independently completes the forward and backward passes using its local data subset to obtain its own gradients. After obtaining the gradients, an AllReduce operation is performed to average (or sum and normalize) the gradients across all devices, and the synchronized gradients are then used to update the local model parameter replicas. Since each device shares identical initial parameters and identical gradients after synchronization, model parameters across all devices remain consistent at all times.  The advantage is simple implementation and non-intrusiveness to the model architecture; the disadvantage is that every device must store a complete copy of model parameters, gradients, and optimizer states, meaning memory overhead does not decrease as the number of devices increases (failing to break single-GPU memory limits), and the communication volume scales linearly with the model parameter count.

</details>

**21. What Is the Principle of Model Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Model Parallelism (MP) partitions the model itself (parameters and computation) across multiple devices, where each device only stores and computes a fraction of the model. This is common in scenarios where single-GPU memory cannot fit the entire model. Model parallelism is generally divided into two major categories: 

- **Tensor Parallelism**: Matrix operations within a single layer (such as the weight matrix of a linear layer) are split row-wise or column-wise across different devices, allowing multiple devices to collaboratively compute the same layer.
- **Pipeline Parallelism**: The model is partitioned along layers (the depth dimension) into multiple stages. Different stages are placed on different devices, and data passes through each stage sequentially like an assembly line.

 Compared to Data Parallelism, Model Parallelism enables a single device to store only a portion of the parameters, thereby supporting the training of large models that exceed single-GPU memory capacity. The trade-off is that intermediate activations or gradients must be passed frequently between devices, resulting in more complex communication patterns, a strong dependence on how the model structure is partitioned, and higher implementation difficulty.

</details>

**22. What Is the Principle of Tensor Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Tensor Parallelism (TP) splits tensor computations within a single neural network layer across multiple devices for parallel execution. Taking the most common Megatron-LM partitioning scheme in Transformer as an example: 

- **MLP Layer**: The weight matrix of the first linear layer is split column-wise across N devices, so each device computes a partial output along the hidden dimension. The activation function (such as GeLU) can be executed independently on each device for its own partial result. The weight matrix of the second linear layer is split row-wise, where each device performs matrix multiplication using its partial output and its partitioned weight matrix. Finally, an **AllReduce** operation sums up the partial results from each device to obtain the full output.
- **Attention Layer**: Partitioned along the attention head dimension, where each device is responsible for the Q, K, V computation and attention calculation of a subset of heads. Finally, an AllReduce operation is similarly used to aggregate the results of the output projection.

 During the entire forward pass, two AllReduce operations are typically required (one after Attention and one after MLP), with corresponding communications required in the backward pass as well. The advantage of Tensor Parallelism is that it can shard single-layer parameters across multiple GPUs, significantly reducing per-GPU memory consumption. The disadvantage is high communication frequency and extremely high demand for inter-device bandwidth (such as NVLink); thus, it is usually restricted to GPUs within the same node.

</details>

**23. What Is the Principle of Pipeline Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Pipeline Parallelism (PP) partitions the model by layer into several contiguous stages, with each stage assigned to one device (or a group of devices). Data from a mini-batch is further divided into multiple smaller micro-batches and fed into the pipeline sequentially: after the first device finishes the forward computation of the first micro-batch, it passes the activations to the second device while simultaneously starting computation on the second micro-batch, and so on. This enables multiple devices to overlap work in time, improving device utilization. Naive pipeline parallelism suffers from a severe "pipeline bubble" problem: during the initial filling and final draining phases of the pipeline, some devices remain idle and waiting. To reduce bubbles, several scheduling strategies have been proposed in the industry, such as: 

- **GPipe**: Completes the forward pass for all micro-batches first before performing the backward pass uniformly. The bubble ratio is approximately (number of devices - 1) / number of micro-batches.
- **1F1B (One-Forward-One-Backward)**: Alternates forward and backward computations to release activation memory earlier; the bubble ratio is the same as GPipe, but peak memory usage is lower.
- **Interleaved 1F1B**: Each device handles multiple non-contiguous layer chunks, further reducing bubbles at the cost of increased communication frequency.

 The communication volume of pipeline parallelism is relatively much smaller than tensor parallelism (transferring activations and gradients only at stage boundaries), making it suitable for cross-node deployment. However, it requires a larger number of micro-batches to amortize the bubble overhead.

</details>

**24. How Does ZeRO Save Memory?**

<details>
<summary>💡 Show Answer</summary>

![ZeRO Stage 1, 2, and 3 memory partitioning across GPU cluster.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig30_zero_stages_memory.png)




The core idea behind ZeRO's memory savings is **eliminating redundant storage in data parallelism**. In standard data parallelism, every GPU maintains a full copy of model parameters, gradients, and optimizer states (for the Adam optimizer, optimizer states are typically more than twice the parameter size). This content is completely identical across all devices, representing pure redundancy. ZeRO shards these states into N partitions according to the number of data parallel devices N, with each device persistently storing only 1/N of them: 

- When data for a specific shard is needed, it is temporarily collected from other devices via **AllGather**;
- Immediately released once computation is complete and no longer needed, avoiding long-term memory occupation;
- After gradient computation completes, the gradients for the shard managed by the local device are obtained directly via **Reduce-Scatter**, rather than performing AllReduce for full gradients first before sharding.

 Thus, theoretically, as the number of devices N increases, the memory footprint per device approaches (total memory requirement) / N. This enables training models far larger than single-GPU memory capacity on a fixed number of GPUs, while retaining the simple programming model of data parallelism (without needing to manually partition the model structure as in model parallelism). The cost is additional AllGather/Reduce-Scatter communication, with the communication volume gradually increasing across ZeRO stages (1/2/3).

</details>

**25. What Are the Main Communication Bottlenecks in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

Communication bottlenecks in distributed training mainly stem from the following aspects: 1. **Gradient/parameter synchronization communication volume is proportional to model scale**: In data parallelism, every step requires an AllReduce over the gradients of all parameters; larger models require larger communication volumes. ZeRO-3 additionally requires AllGather of parameters during forward and backward passes for every layer. 2. **Cross-node bandwidth limitations**: Intra-node GPUs are interconnected via NVLink/NVSwitch with bandwidths up to hundreds of GB/s, whereas inter-node connections typically rely on InfiniBand or Ethernet, where bandwidth (tens to over a hundred GB/s) and latency are far inferior to intra-node connections. Cross-node communication is often the primary bottleneck. 3. **High-frequency small-packet communication in tensor parallelism**: Tensor parallelism performs multiple AllReduce/AllGather operations per layer with extremely high communication frequency and sensitivity to latency, making it usually restricted to intra-node use (high-bandwidth interconnects). 4. **Point-to-point communication and bubbles in pipeline parallelism**: Transferring activations/gradients between stages involves a relatively small communication volume, but if computation and communication cannot be effectively overlapped, pipeline bubbles arise, leading to idle devices. 5. **All-to-All communication (MoE scenarios)**: In MoE models, routing tokens to different experts requires All-to-All communication, which easily becomes a bottleneck when communication patterns are irregular or loads are unbalanced. 6. **Difficulty in overlapping communication and computation**: When communication primitives (such as AllReduce) cannot be executed asynchronously in overlap with backward computation, communication time directly adds to the total training time. Overall, **communication bandwidth (especially cross-node bandwidth) and communication latency** are the core factors constraining the scaling efficiency (speedup) of large-scale distributed training. Therefore, one of the central goals of parallel strategy design is to restrict high-frequency communication (tensor parallelism) within high-bandwidth domains, while placing lower-frequency communication (data parallelism, pipeline parallelism) over lower-bandwidth cross-node links.

</details>

**26. What Are AllReduce, AllGather, and ReduceScatter?**

<details>
<summary>💡 Show Answer</summary>

These three are the most commonly used collective communication primitives in distributed training, all implemented by communication libraries such as NCCL. A common efficient implementation method is the Ring algorithm. 

- **AllReduce**: Each device holds a piece of data (such as locally computed gradients). AllReduce performs a reduction operation (typically summation) over the data on all devices and **broadcasts the reduced result to all devices**, so that every device ultimately holds identical, complete reduced results. Typical use case: Synchronizing gradients across devices in data parallelism (summing them up and dividing by the number of devices to obtain average gradients).

- **AllGather**: Each device holds a portion of data (a shard). AllGather **gathers and concatenates** shards from all devices so that every device ultimately possesses the full concatenated data of all shards. Typical use case: Gathering parameter shards from all devices in ZeRO-3 to reconstruct complete parameters for a specific layer.

- **ReduceScatter**: Each device holds a full set of data. ReduceScatter first performs a reduction (such as summation) over the data across all devices, and then **splits the reduced result into shards**, with each device retaining only one shard. It can be understood as the "first half" of AllReduce (reduction) plus "sharding" (without broadcasting the full result). Typical use case: In ZeRO-2/3, each device directly obtains the gradient shard it is responsible for, avoiding the redundancy of getting full gradients before sharding.

 The relationship among the three can be summarized as: **AllReduce $\approx$ ReduceScatter + AllGather** (first reduce into shards, then gather and broadcast each shard to all devices, yielding a result equivalent to direct AllReduce).

</details>

**27. What Hybrid Parallel Strategies Are Commonly Used for Large Models?**

<details>
<summary>💡 Show Answer</summary>

When training ultra-large-scale models, a single parallel strategy often cannot satisfy both memory and efficiency requirements simultaneously. Thus, "Hybrid Parallelism," which combines multiple parallel approaches, is commonly adopted: 

- **Data Parallelism (DP)**: Scales training throughput, but does not solve single-GPU memory issues.
- **Tensor Parallelism (TP)**: Partitions matrix operations within a single layer to reduce single-GPU memory usage, but requires frequent communication and is generally restricted to intra-node setups (e.g., 8 GPUs on a single node interconnected via NVLink).
- **Pipeline Parallelism (PP)**: Partitions the model layer-by-layer across different devices/nodes, involving smaller communication volume and making it suitable for cross-node deployment.
- **3D Parallelism (DP + TP + PP)**: A typical combination in frameworks like Megatron-LM and DeepSpeed, e.g., 8 GPUs intra-node performing tensor parallelism, pipeline parallelism applied across multiple nodes, and data parallelism at the outermost layer scaling to more pipeline groups, thereby balancing memory, communication efficiency, and scalability simultaneously.
- **ZeRO + Data Parallelism**: Uses ZeRO to shard optimizer states/gradients/parameters combined with data parallelism to further reduce memory overhead, often combined with TP/PP to form higher-dimensional parallelism (sometimes called 4D parallelism).
- **Expert Parallelism (EP)**: Designed for MoE models, distributing different experts across different devices and utilizing All-to-All communication to route tokens, often combined with DP/TP/PP.
- **Sequence Parallelism**: Partitions the sequence dimension across multiple devices in coordination with tensor parallelism, further reducing activation memory footprint for operations like LayerNorm and Dropout.

 In practice, ultra-large model training (such as GPT-3, LLaMA, DeepSeek, etc.) is typically a combination of the aforementioned strategies, where the dimension size of each parallelism is determined based on cluster topology (bandwidth differences intra-node vs. inter-node).

</details>

**28. What Are the Components of Memory Footprint in Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

The memory footprint during large model training mainly consists of the following components: 1. **Model Parameters**: The model weights themselves, taking 2 bytes per parameter during FP16/BF16 training. 2. **Gradients**: Same count as parameters, taking 2 bytes per gradient under FP16/BF16. 3. **Optimizer States**: Taking the most commonly used Adam/AdamW as an example, first moment (momentum) and second moment (variance) must be saved, typically in FP32 to ensure numerical stability, taking $2 \times 4 = 8$ bytes per parameter; if an FP32 master weight copy is also maintained (a common practice in mixed-precision training), an additional 4 bytes are required, totaling around 12 bytes per parameter for optimizer-related states. 4. **Activations**: Intermediate results retained during the forward pass for backward propagation, such as layer inputs, attention matrices, and activation function outputs. Activation memory is roughly proportional to batch size, sequence length, hidden dimension, and number of layers, making it one of the main sources of memory consumption in long-sequence, large-batch training, which can be significantly reduced via activation recomputation. 5. **Temporary Buffers**: Temporary buffers used in communication operations (AllReduce/AllGather), temporary workspaces for CUDA kernels, etc. 6. **Memory Fragmentation**: Fragmented space caused by non-contiguous memory allocation/deallocation that cannot be effectively utilized. A rough estimation formula (mixed precision + Adam, excluding activations): each parameter requires approximately (2 parameters + 2 gradients + 4 first moment + 4 second moment + 4 FP32 master weights) $\approx$ 16 bytes. For example, a 7B parameter model requires approximately 112 GB of memory just for parameters + gradients + optimizer states, which does not even include activations, making it a primary target for optimization by technologies like ZeRO.

</details>

**29. How Is Activation Checkpointing Implemented?**

<details>
<summary>💡 Show Answer</summary>

The specific implementation of activation checkpointing is usually based on the checkpointing mechanism provided by deep learning frameworks (such as PyTorch's `torch.utils.checkpoint`). The core steps include: 1. **Dividing Checkpoint Segments**: The model is divided into several segments at a certain granularity (e.g., treating each Transformer layer as a segment), with each segment acting as a checkpoint unit. 2. **Forward Pass Phase**:

- For each checkpoint segment, the forward computation is executed normally to obtain outputs;
- However, **intermediate activation values inside the segment are not saved** (such as intermediate results of Attention or intermediate hidden states of MLP); only the segment's **input** (serving as the starting point for recomputation) and **output** (passed to the next segment) are saved.
- To ensure consistent randomness during recomputation (such as Dropout), the random seed used during the forward pass is typically recorded and reused during backward recomputation.

3. **Backward Pass Phase**:

- When gradients for a checkpoint segment need to be computed, the saved segment input is first used to **re-execute the forward pass of that segment** (this time retaining intermediate activations) to build the complete computation graph;
- Then, based on the regenerated intermediate activations, backward propagation is executed normally to compute gradients for that segment;
- Once computed, temporary activations generated by this recomputation are immediately released.

4. **Inter-segment Gradient Propagation**: The input gradient computed for each segment serves as the output gradient for the previous segment, continuing backward propagation according to the chain rule. At the implementation level, PyTorch's `checkpoint` function uses a custom `autograd.Function` to execute computations with `torch.no_grad()` in `forward` and discard intermediate results, then re-enables gradient tracking in `backward`, recomputes the forward pass, and immediately runs backward propagation. This transparently implements the logic of "not saving activations during forward pass, recomputing during backward pass" for users. In engineering practice, a common strategy is to checkpoint each Transformer block as a whole (coarser granularity, simple implementation), or more finely checkpoint only the parts in Attention or MLP with the largest memory footprint ("selective recomputation"), achieving a finer balance between memory footprint and computational overhead.

</details>

**30. What Is the Role of Gradient Accumulation in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

Gradient Accumulation refers to performing forward and backward passes sequentially across multiple micro-batches without updating parameters immediately, **accumulating** the gradients calculated from each micro-batch. After accumulating for a set number of steps, a single parameter update is executed using the accumulated gradients (typically divided by the number of accumulation steps to obtain average gradients). Its main roles include: 1. **Simulating a larger batch size under memory constraints**: Single-GPU memory might only accommodate a smaller micro-batch, but through gradient accumulation, one can equivalently achieve the effect of training with a large batch where "global batch size = micro-batch size $\times$ accumulation steps $\times$ number of data-parallel devices", without increasing single-step peak memory usage. 2. **Reducing communication frequency**: In data parallelism, gradient synchronization (AllReduce) only needs to be executed once after accumulation is complete, right before updating parameters, rather than synchronizing per micro-batch. This reduces communication calls and improves communication efficiency (especially when cross-node bandwidth is limited). 3. **Working in tandem with pipeline parallelism**: Pipeline parallelism itself requires splitting a batch into multiple micro-batches to fill the pipeline. Gradient accumulation naturally aligns with this partitioning method, where the number of micro-batches simultaneously impacts the pipeline bubble ratio and the effective batch size of gradient accumulation. It should be noted that gradient accumulation merely changes "how much data is used per gradient update" without altering how each sample contributes to the final gradient (equivalent to summing first then averaging). Thus, mathematically it is equivalent to training directly with a large batch (ignoring differences in layers like BatchNorm that rely on batch statistics).

</details>

**31. How Does Batch Size Impact Training Stability and Performance?**

<details>
<summary>💡 Show Answer</summary>

Batch size is one of the key hyperparameters affecting training stability and final performance of large models, mainly reflected in the following aspects: 1. **Gradient Noise and Generalization**: Under smaller batch sizes, the gradient calculated at each step is a noisier estimate of the true gradient. This noise sometimes helps skip out of sharp local minima and improves generalization, but it may also cause training curve oscillations and slower convergence. Larger batch sizes yield more accurate gradient estimates and smoother training curves, but may converge to "sharper" minima, leading to relatively worse generalization (the "large batch generalization gap" issue). 2. **Matching Relationship with Learning Rate**: As batch size increases, statistical noise per step gradient decreases, usually requiring a corresponding increase in learning rate to maintain similar convergence speed (Linear Scaling Rule). However, an excessively large learning rate easily causes early training instability or divergence, so large-batch training typically requires warmup (learning rate warmup) to mitigate early instability. 3. **Training Stability (Loss Spikes)**: In large model training, overly large batch sizes sometimes increase the risk of loss spikes (sudden spikes in loss), especially when learning rates are high or anomalous samples exist in data. Conversely, overly small batch sizes can cause excessive gradient noise, indirectly inducing instability. 4. **Hardware Utilization and Training Efficiency**: From an engineering perspective, a larger batch size can more fully utilize the parallel computing capabilities of GPUs, increasing throughput (tokens/sec). However, an excessively large batch size increases memory footprint, requiring integration with technologies like ZeRO and activation recomputation. 5. **Critical Batch Size**: There exists a critical threshold beyond which continuing to increase batch size yields significantly diminishing returns in convergence speed per unit of computation (i.e., "spending more compute for diminishing returns"). The critical batch size is related to model scale, data distribution, and other factors, serving as an important reference for determining the global batch size in large model training. Overall, the choice of batch size requires balancing gradient estimation quality, learning rate tuning, hardware utilization, and training stability. In large model training, batch size is often dynamically adjusted as training progresses (e.g., batch size warmup, gradually increasing from a smaller value to the target value).

</details>

**32. How Should Micro-batches Be Designed?**

<details>
<summary>💡 Show Answer</summary>

A micro-batch is the fundamental data unit in pipeline parallelism and gradient accumulation. Its design requires comprehensive consideration of memory footprint, pipeline efficiency, and convergence behavior: 1. **Lower bound of micro-batch size**: Limited by hardware compute efficiency, micro-batch size cannot be too small; otherwise GPU execution units (especially Tensor Cores for matrix multiplication) cannot be fully utilized, leading to a drop in compute efficiency (MFU, Model FLOPs Utilization). Meanwhile, overly small batches cause unstable estimation of statistics in layers like LayerNorm. 2. **Upper bound of micro-batch size**: Limited by single-GPU memory capacity, a larger micro-batch results in a larger activation memory footprint retained during forward propagation. When using pipeline parallelism, each pipeline stage must retain activations for multiple in-flight micro-batches, so micro-batch size must be considered together with pipeline depth. 3. **Number of micro-batches and pipeline bubbles**: The bubble ratio in pipeline parallelism is approximately $(number of pipeline stages - 1) / number of micro-batches$. Therefore, more micro-batches lead to a smaller bubble ratio and higher pipeline efficiency. However, increasing the number of micro-batches implies that micro-batch size needs to be correspondingly reduced (given a fixed global batch size), requiring a balance between the two. In practice, the number of micro-batches is usually set to several times (e.g., 4x or more) the number of pipeline stages to keep bubbles within an acceptable range. 4. **Relationship with gradient accumulation**: $Global batch size = micro-batch size \times number of micro-batches (gradient accumulation steps) \times data parallel size$. Once the global batch size (determined by convergence requirements) and data parallel size (determined by cluster scale) are fixed, the product of micro-batch size and count is determined. Combining memory constraints and pipeline bubble considerations then determines the specific split. A common practice in industry is to first test and find the maximum micro-batch size runnable on a single GPU (ensuring no OOM after enabling techniques like activation recomputation), then back-calculate the required gradient accumulation steps (number of micro-batches) based on global batch size requirements and pipeline depth, fine-tuning the final configuration through throughput experiments.

</details>

**33. How Is a Multi-Node Multi-GPU Training Architecture Typically Constructed?**

<details>
<summary>💡 Show Answer</summary>

Constructing a multi-node multi-GPU distributed training architecture generally encompasses the following layers: **1. Hardware and Network Layer**

- Each machine (node) is equipped with multiple GPUs, interconnected intra-node via high-speed NVLink/NVSwitch (bandwidth of several hundred GB/s);
- Nodes are interconnected inter-node via InfiniBand or high-speed Ethernet (RoCE), configured with GPUDirect RDMA to allow GPUs to communicate directly over the network bypassing the CPU, reducing latency and boosting bandwidth;
- Node allocation is usually planned rationally according to network topology (such as rack and switch hierarchy), ensuring communication-intensive parallel dimensions (e.g., tensor parallelism) are assigned as much as possible to devices with closer network proximity.

 **2. Communication Library and Framework Layer**

- Uses NCCL as the underlying collective communication library among GPUs, responsible for implementing operations like AllReduce, AllGather, Broadcast, and automatically selecting optimal communication algorithms (e.g., Ring, Tree) based on hardware topology.
- On the framework side, PyTorch Distributed (in conjunction with DDP, FSDP), DeepSpeed, Megatron-LM, etc., are commonly used. These frameworks encapsulate parallel strategy implementations (data parallelism, tensor parallelism, pipeline parallelism, ZeRO, etc.) and communication scheduling.

 **3. Process and Device Mapping**

- Each GPU typically corresponds to one training process (rank), launched via `torchrun` or similar launchers (e.g., DeepSpeed launcher, Slurm) to start the corresponding number of processes on each machine;
- Sets a globally unique `world_size` (total process count), `rank` (global ID), `local_rank` (intra-node ID), as well as `master_addr`/`master_port` for process group initialization (`init_process_group`).
- Maps ranks into different parallel groups (data parallel group, tensor parallel group, pipeline parallel group) according to the parallel strategy (such as 3D parallelism), based on which the framework creates corresponding communication sub-groups (process groups).

 **4. Scheduling and Orchestration Layer**

- Cluster job scheduling typically relies on systems such as Slurm or Kubernetes (combined with Kubeflow/Volcano) for resource allocation, node scheduling, and fault-tolerant restarts;
- Works alongside distributed file systems (such as NFS, Lustre, object storage) to share datasets, model weights, and checkpoint files, ensuring consistent data access across all nodes.

 **5. Monitoring and Fault Tolerance**

- Deploys training monitoring systems (such as TensorBoard, Weights & Biases, Prometheus + Grafana) to observe metrics like loss, throughput, GPU utilization, memory footprint, and communication overhead in real time;
- Configures periodic checkpoint saving and auto-restart mechanisms to handle training interruptions caused by hardware failures (e.g., node disconnection).

 Overall, constructing a multi-node multi-GPU training architecture is a collaborative engineering effort combining "hardware interconnect topology + communication library + distributed training framework + job scheduling system", aiming to align the communication patterns of parallel strategies with hardware topologies to maximize training throughput and speedup ratios.

</details>

**34. What Role Does NCCL Play in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

NCCL (NVIDIA Collective Communications Library) is a collective communication library provided by NVIDIA and specifically optimized for multi-GPU/multi-node environments. It serves as the underlying communication engine relied upon by the vast majority of large model distributed training frameworks (PyTorch, DeepSpeed, Megatron-LM, etc.). Its main roles include: 1. **Implementing efficient collective communication primitives**: Provides high-performance implementations of communication operations such as AllReduce, AllGather, ReduceScatter, Broadcast, Reduce, All-to-All, and Send/Recv. These primitives form the foundation for gradient synchronization in data parallelism, parameter/gradient gathering in ZeRO, activation passing in pipeline parallelism, token routing communication in MoE, and more. 2. **Topology-aware communication optimization**: NCCL automatically detects hardware topology (NVLink/PCIe interconnections between GPUs, network connections between nodes), selects optimal communication algorithms based on topology (such as Ring AllReduce, Tree AllReduce, Double Binary Tree, etc.), and automatically divides communication tasks across multiple SMs (Streaming Multiprocessors) on GPUs to fully leverage bandwidth. 3. **Direct GPU communication**: Supports direct GPU-to-GPU communication via NVLink (without passing through CPU memory), and supports GPUDirect RDMA across nodes so GPU memory can directly transmit data over network interfaces, significantly reducing communication latency and boosting bandwidth utilization. 4. **Overlapping with computation**: NCCL communication operations can be executed asynchronously (such as overlapping gradient AllReduce with other gradient computations during backward pass in PyTorch DDP), thereby hiding part of the communication latency and improving overall training efficiency. 5. **Scalability**: NCCL is extensively optimized for communication efficiency across large-scale GPU clusters (thousands or even tens of thousands of GPUs), serving as one of the critical infrastructure components supporting the effective scaling of ultra-large model training on large clusters. In summary, NCCL acts as the bridge connecting "upper-level parallel strategies (data parallelism / tensor parallelism / pipeline parallelism / ZeRO, etc.)" with "underlying hardware interconnects (NVLink / InfiniBand)", and its performance directly dictates communication efficiency and overall speedup ratios in distributed training.

</details>

**35. How to Measure Training Speed and Speedup?**

<details>
<summary>💡 Show Answer</summary>

![Distributed training speedup and scaling efficiency curves.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig35_training_speedup_scaling.png)




Common metrics used to measure distributed training speed and speedup include the following categories:  **1. Throughput**

- **Tokens/second (tokens per second)**: The number of tokens processed per unit time. It is the most commonly used speed metric in LLM training, making it easy to compare across configurations with different batch sizes and sequence lengths.
- **Samples/second (samples per second)**: The number of samples processed per unit time.
- **Time per step / iteration time**: The time required to complete one training step (a full forward pass + backward pass + parameter update).

 **2. Speedup and Scaling Efficiency**

- **Speedup** = Throughput with N devices / Throughput with 1 device. Ideally, it should be close to N (linear speedup).
- **Scaling efficiency** = Speedup / N (i.e., the number of devices). The closer the scaling efficiency is to 100%, the smaller the additional overhead (communication, synchronization wait) introduced by distributed training.
- **Strong scaling**: Keep the global problem size fixed (e.g., fixed global batch size) and increase the number of devices to observe the speedup effect—ideally, the total training time should decrease almost linearly.
- **Weak scaling**: Increase the problem size proportionally while increasing the number of devices (e.g., global batch size grows linearly with the number of devices) to observe whether the amount of data processed per unit time grows linearly.

 **3. Compute Resource Utilization**

- **MFU (Model FLOPs Utilization)**: The ratio of the actual floating-point operation throughput achieved (FLOPs/s) to the hardware's theoretical peak compute capacity (Peak FLOPs/s). It reflects the utilization degree of hardware compute power by model computation itself (excluding overhead such as communication and data loading). It is one of the core metrics for evaluating LLM training efficiency, usually expressed as a percentage. Industry-leading LLM training MFU can reach around 40%--60%.
- **GPU Utilization**: Observed via tools like `nvidia-smi` to monitor the occupancy of GPU compute units, reflecting whether the GPU is in a "busy" state (however, high utilization does not mean compute power is effectively used for useful computation, as it may include communication wait times).

 **4. Communication Overhead Ratio**

- Analyze the ratio of compute time to communication time in a training step using profiling tools (such as PyTorch Profiler or Nsight Systems). The higher the communication ratio, the more severe the communication bottleneck, which usually leads to lower scaling efficiency.

 Overall, when evaluating the performance of distributed training, one typically needs to pay attention simultaneously to **absolute throughput (tokens/s)**, **relative speedup/scaling efficiency**, and **MFU**. The former reflects actual production efficiency, while the latter two reflect the "quality" of distributed strategies and hardware utilization, making it easy to diagnose where the bottleneck lies (compute bottleneck vs. communication bottleneck).

</details>

**36. What Are the Stability Issues in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

Common stability issues during LLM distributed training include:  1. **Loss Spikes**: The loss value suddenly increases significantly during training. This can be caused by abnormal data samples, numerical overflow (excessively large gradients/activations in mixed precision), excessively high learning rate, or abnormal optimizer states. In severe cases, it can cause training divergence, requiring mitigation measures such as gradient clipping, learning rate reduction, skipping abnormal batches, or rolling back to the latest checkpoint. 2. **Numerical Overflow/Underflow (NaN/Inf)**: In FP16 training, gradients or activations may exceed the representation range of FP16, resulting in NaN or Inf values that cause abnormal parameter updates. This is commonly mitigated through loss scaling in mixed precision training or switching to BF16 (which has a wider dynamic range). 3. **Hardware Failures and Node Failures**: In large-scale clusters, hardware issues such as GPU failures, network disruptions, and node crashes are almost inevitable during long-duration training. Reliability relies on checkpointing mechanisms and automatic restarts to ensure training can recover from failures (resuming from breakpoints). 4. **Communication Timeouts and Deadlocks**: Collective communication operations (such as AllReduce) require all participating processes to call them simultaneously. If a process fails to make the call in time due to unbalanced workload, abnormal exit, or hanging, it will cause other processes to wait for a long time or even raise timeout errors. This is common in scenarios with load imbalances or individual process anomalies. 5. **Straggler Problem**: Individual nodes in a cluster may run slowly due to hardware performance differences, network jitter, or shared resource contention. Since all devices must wait for the slowest one in synchronous training, stragglers slow down the overall training speed. 6. **Out of Memory (OOM)**: Due to fluctuations in sequence length (e.g., certain batches having exceptionally long sequences) or GPU memory fragmentation, training may suddenly encounter OOM at a particular step, causing process crashes. 7. **Numerical Consistency Issues in Optimizer States and Mixed Precision**: For example, when using techniques such as ZeRO, subtle implementation differences in parameter sharding, gradient accumulation, and mixed-precision conversion can cause numerical results to differ slightly from single-GPU training, and errors may accumulate and amplify over long training periods. 8. **Data-Related Issues**: If the data loading order or shuffle randomness is improperly handled in a distributed environment, different ranks may see duplicate or missing data, affecting training effectiveness and reproducibility.  Addressing these issues typically requires a comprehensive combination of: gradient clipping, loss scaling, periodic checkpointing, monitoring and alerting (loss, gradient norm, activation range, etc.), automatic restart mechanisms, and detailed training log recording for quick diagnosis and resolution of anomalies.

</details>

**37. What Impact Does Parameter Synchronization Latency Have?**

<details>
<summary>💡 Show Answer</summary>

Parameter synchronization latency refers to the time spent synchronizing gradients or parameters across devices during distributed training. It mainly brings the following impacts:  1. **Directly Slowing Down Training Speed**: In synchronous training (such as standard data parallelism), parameter updates must wait until all devices finish gradient synchronization (AllReduce). Synchronization latency adds directly to the total time of each training step, becoming one of the main bottlenecks of training throughput. 2. **Reducing Scaling Efficiency**: As the number of devices increases, the theoretical workload allocated to each device shrinks (shortening the per-step computation time), but communication volume (especially cross-node communication) often does not decrease proportionally. As a result, the proportion of communication latency in total time gradually rises, causing scaling efficiency (speedup / number of devices) to decrease with more devices. This is the fundamental reason why large-scale distributed training struggles to achieve perfectly linear speedup. 3. **Affecting Overlap Efficiency Between Computation and Communication**: If synchronization latency cannot be effectively overlapped with computation (e.g., waiting for AllReduce before the backward pass finishes computing the corresponding gradients), it leads to idle GPU waiting, reducing GPU utilization (MFU). 4. **Amplifying the Straggler Effect**: In synchronous training, if a particular device experiences exceptionally slow parameter synchronization due to network latency or hardware performance, all other devices must wait for it. This creates a "barrel effect" where overall training speed is dictated by the slowest device. 5. **Impact on Asynchronous Training (e.g., using asynchronous updates)**: Under the asynchronous training paradigm, parameter synchronization latency causes different devices to use inconsistent parameter versions ("stale gradients")—that is, the parameter version a device bases its gradient computation on has already been updated by other devices. Such latency can degrade convergence speed and final model quality; the larger the latency, the higher the degree of staleness, and typically the worse the negative impact on convergence.  To mitigate the impact of parameter synchronization latency, common practices include: using high-bandwidth, low-latency interconnects (NVLink, InfiniBand); adopting technologies such as ZeRO to reduce unnecessary communication volume; lowering synchronization frequency via gradient accumulation; overlapping communication with computation as much as possible (e.g., initiating AllReduce as soon as a portion of gradients is computed); and designing parallel strategies appropriately to constrain high-frequency communication within low-latency domains.

</details>

**38. How Are Checkpoints Saved and Restored During Training?**

<details>
<summary>💡 Show Answer</summary>

Saving and restoring training checkpoints is a key mechanism to ensure the reliability of long-duration LLM training, mainly including the following aspects:  **1. Saved Content** A complete training checkpoint typically contains:

- **Model parameters** (weights);
- **Optimizer states** (such as Adam's first and second moments; for LLMs, this portion is usually comparable to or even larger than the parameter count);
- **Learning rate scheduler state** (current learning rate, trained step count, etc.);
- **Training progress information**: current epoch/step, processed data position (used for data loader recovery), random seed states, etc.;
- **(Optional) Mixed-precision training states**: such as the current scale value of the loss scaler.

 **2. Saving Methods in Distributed Scenarios**

- **Sharded Checkpoint**: When using techniques like ZeRO, tensor parallelism, or pipeline parallelism, each device holds only a portion of the model/optimizer state. Typically, each rank independently saves its assigned shard to a separate file to avoid expensive collective communication during saving; upon restoration, each rank simply loads its corresponding shard.
- **Consolidated Checkpoint**: Sometimes, to facilitate subsequent inference deployment or migration to a different parallel configuration, sharded checkpoints need to be re-aggregated into a complete model weight file (merged via AllGather or offline scripts).
- **Asynchronous Checkpointing**: To reduce the blocking time of checkpoint saving on training, parameters can be copied to CPU memory or shared memory first and then asynchronously written to disk/object storage, allowing training to proceed without waiting for disk I/O to complete.

 **3. Saving Strategies**

- Periodically save at fixed step intervals or time intervals (e.g., save every N steps);
- Retain the most recent several checkpoints (to prevent unlimited disk space growth) while optionally retaining periodic "milestone" checkpoints (e.g., at the end of each epoch) for backtracking;
- Upon detecting an anomaly (such as a loss spike), choose to roll back to the most recent normal checkpoint and resume training.

 **4. Restoration Workflow**

- Check for available checkpoints when launching training (e.g., the latest checkpoint in a specified path);
- Each rank loads its corresponding parameter and optimizer state shard according to the parallel configuration consistent with saving time;
- Restore the learning rate scheduler, training step count, data loader positions, etc., ensuring training can seamlessly resume from the interruption point;
- If the parallel configuration during restoration (e.g., number of GPUs, parallel strategy) differs from that at saving time, the sharded checkpoints must first be consolidated or resharded (reshaped) to adapt to the new parallel configuration.

 A good checkpointing mechanism should balance **saving efficiency (not excessively blocking training)**, **storage cost (not indefinitely occupying disk space)**, and **restoration correctness (able to correctly restore under different parallel configurations)**.

</details>

**39. How Is Resuming Training from Checkpoints Implemented?**

<details>
<summary>💡 Show Answer</summary>

Resuming training from a checkpoint (Resume Training from Checkpoint) means that after training is interrupted due to failure or manual intervention, it can recover from the most recently saved checkpoint and continue training without starting from scratch. Key implementation points include:  1. **Complete Saving of Training State**: As described in Question 324, it is necessary to save model parameters, optimizer states, learning rate scheduler state, current step/epoch, data loading progress, random seed states, etc., to ensure the training state after restoration is as consistent as possible with the state before interruption. 2. **Restoration of Data Loading Position**:

- If using a dataset supporting random access (such as an index-based Dataset), one can record the processed sample indices or the offset within the current epoch, skipping processed data upon restoration or continuing from the recorded position;
- If using streaming data (such as large-scale pretraining corpora continuously read from a data stream), one typically records the data file name/shard ID and intra-file offset, positioning to the corresponding location to resume reading upon restoration;
- For scenarios using shuffle, random seeds and shuffle states must be restorable to minimize data duplication or omission (fully exact restoration is often costly, and a certain degree of approximation is acceptable in practice).

3. **Continuity of Learning Rate Scheduling**: Learning rate schedulers (such as cosine decay or warmup) usually compute the current learning rate based on the number of trained steps. Upon restoration, the trained step count must be correctly set to ensure the learning rate curve remains continuous without "jumping" or "re-starting warmup" due to restart. 4. **Restoration of Mixed-Precision States**: If dynamic loss scaling is used, the current scale value must be restored to avoid re-exploring from default values after restart, which could cause early numerical instability. 5. **Consistency in Distributed Environments**: In multi-node multi-GPU environments, all ranks must correctly restore from their corresponding checkpoint shards and re-establish distributed communication groups (re-executing `init_process_group`) to ensure states across all ranks are consistent and synchronized. 6. **Automated Restart Mechanism**: In production environments, task scheduling systems (such as Slurm or Kubernetes) are typically configured to automatically detect abnormal process exits and restart training, specifying the latest checkpoint path upon restart to achieve unattended breakpoint resumption. 7. **Idempotent Design of Training Scripts**: Training startup scripts should be able to determine whether to "start from scratch" or "resume from checkpoint", initializing the model, optimizer, and data loaders accordingly to prevent duplicate initialization or state conflicts.  Implementing a robust breakpoint resumption mechanism enables training to recover at minimal cost even when hardware failures, preemption (e.g., using preemptible instances), or manual pauses occur, making it an indispensable engineering capability for long-duration, large-scale training.

</details>

**40. How Should Data Loading Be Designed in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

![Distributed data loading, prefetching, and async host-to-device transfer pipeline.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig36_distributed_data_loader.png)




The design of data loading in distributed training needs to balance **correctness (each sample is properly assigned and used)**, **efficiency (not becoming a training bottleneck)**, and **recoverability (supporting resumption from breakpoints)**. The main considerations include:  1. **Data Sharding**: Divide the entire training dataset into non-overlapping shards according to the number of data parallel ranks. Each rank reads only its assigned shard to prevent different ranks from redundantly loading identical data (unless intentional for data augmentation). Common practices include file/shard-level partitioning (each rank is responsible for a subset of data files) or sample-level partitioning via `index % world_size == rank`. 2. **Shuffle Strategy**:

- Global shuffle is extremely expensive on large-scale datasets. A combination of "intra-shard shuffle + shard order shuffle" is usually adopted: first shuffle the order of data shards, and then perform local shuffle inside each shard;
- Use different random seeds for each epoch to prevent seeing the exact same data order in every round of training.

3. **Prefetching and Multi-process Loading (Prefetching & DataLoader Workers)**: Use multiple background processes/threads to read and preprocess data in advance (such as `num_workers` and `prefetch_factor` in PyTorch DataLoader), overlapping data preparation with GPU computation to prevent the GPU from idling while waiting for data. 4. **Streaming Data Loading**: For TB-scale or even larger pretraining corpora that cannot fit into memory at once, streaming reading is adopted (e.g., based on WebDataset or iteratively reading compressed shard files), processing and training while reading. 5. **Dynamic Batching and Length Bucketing**:

- For variable-length sequences (e.g., NLP data), group samples of similar lengths into a batch to reduce computational waste caused by padding;
- Dynamic batch sizing can be determined by token count (rather than sample count), keeping the total token count per batch nearly consistent to improve GPU memory utilization and compute efficiency.

6. **Cooperation Between Data Loading and Parallel Strategies**:

- In tensor parallelism or pipeline parallelism, all ranks within the same parallel group usually need to see the **same data** (since they jointly process different parts of the same batch of samples). Data sharding should be performed along the data parallel dimension rather than the tensor/pipeline parallel dimensions;
- Set the data parallel rank ID and total data parallel group count for each rank correctly to ensure the correctness of data sharding.

7. **Breakpoint Resumption Support**: As described in Question 325, position information of data reading (shard index, intra-file offset, consumed sample count, etc.) needs to be recorded, and reading should resume from the corresponding position upon restoration to minimize data duplication or omission. 8. **Data Format and I/O Optimization**: Preprocess raw data into efficient binary formats (such as TFRecord, Megatron's indexed dataset, Parquet, etc.) to reduce online parsing overhead during training; store data on distributed file systems or object stores, and use local caching (such as node-local SSD caching for frequently used data) to cut down repeated remote I/O.  A good data loading design should ensure that **data I/O does not become a training bottleneck** (i.e., data preparation speed equals or exceeds GPU compute speed) while ensuring **correctness of data sharding and reproducibility**.

</details>

**41. How Does Distributed Training Address Load Balancing?**

<details>
<summary>💡 Show Answer</summary>

The load balancing problem in distributed training refers to the imbalance of computational or communication workload across different devices/processes, leading to some devices idling and waiting, thus reducing overall efficiency. Common load imbalance scenarios and their solutions include:  **1. Data Length Imbalance (Variable-Length Sequences)**

- Problem: In tasks like NLP, sample lengths vary significantly. If batches are split simply by sample count, some ranks may receive far more effective tokens than others, causing inconsistent compute times.
- Solution: Adopt dynamic batching based on token count rather than sample count; perform finer-grained splitting or bucketing on long sequence samples so that the total token count processed by each rank per step is nearly equal.

 **2. Workload Imbalance Across Stages in Pipeline Parallelism**

- Problem: If a model is evenly divided across pipeline stages by layer count, but compute workloads of different layers (e.g., Embedding layer, final output layer vs. intermediate Transformer blocks) differ, certain stages will become bottlenecks, generating additional bubble overhead.
- Solution: Partition pipeline stages based on actual floating-point operations (FLOPs) per layer rather than simple layer counts, keeping computation times across stages as close as possible; heavy layers (e.g., output layers with large vocabularies) can also be handled independently or split across multiple devices.

 **3. Expert Load Imbalance in MoE Models**

- Problem: The number of tokens routed to different experts may vary greatly, causing experts on some devices to be "overloaded" while others remain "idle".
- Solution: Introduce **Load Balancing Loss**, adding an auxiliary loss term to the training objective that encourages uniform token assignment across experts; set capacity limits (capacity factor), dropping or overflowing tokens that exceed capacity to other experts; use expert parallelism combined with All-to-All communication for dynamic scheduling.

 **4. Heterogeneous Hardware / Straggler Problem**

- Problem: Some nodes in a cluster may have lower hardware performance or suffer interference from other tasks (resource contention), slowing down their compute speeds and dragging down overall synchronous training.
- Solution: Ensure assigned node performance is consistent at the task scheduling layer; monitor compute time per rank to identify and replace or isolate persistently slow "stragglers"; in certain scenarios, consider using asynchronous training or elastic parallel frameworks to reduce sensitivity to individual stragglers.

 **5. Imbalanced Partitioning in Tensor Parallelism**

- Problem: If certain model dimensions (e.g., number of attention heads, hidden dimension) are not divisible by the parallel degree, some devices may receive slightly more or less computation.
- Solution: When designing model architectures and selecting parallel degrees, ensure key dimensions (e.g., hidden size, head count) are divisible by the tensor parallel degree; for indivisible cases, apply appropriate padding or special non-uniform partitioning logic.

 The overall philosophy is to: **first identify the root cause of imbalance (data, model architecture, or hardware), and then systematically apply dynamic scheduling, auxiliary losses, reasonable partitioning, etc., to make workloads across compute units as close as possible, thereby reducing synchronization wait times and improving overall hardware utilization**.

</details>

**42. What Is the Principle of Mixed Precision Training?**

<details>
<summary>💡 Show Answer</summary>

Mixed Precision Training refers to **simultaneously using low-precision (such as FP16/BF16) and high-precision (FP32) numerical formats** during training to significantly reduce memory footprint and increase compute speed while maintaining model accuracy (convergence behavior) as much as possible. Its core principles include:  1. **Low-Precision Computation**: Computationally intensive operations like matrix multiplications and convolutions in forward and backward passes are performed using FP16 or BF16. These operations run much faster on GPUs supporting Tensor Cores compared to FP32 (typically 2 to 8 times faster), while halving memory footprint.  2. **FP32 Master Weights**: To prevent small gradient update values from being "swallowed" (underflow/precision loss) when added to weights due to FP16's limited precision (having only 10 mantissa bits), an **FP32 master weight copy** is maintained:

- Forward/backward passes use the FP16 version of weights;
- After gradients are computed, parameter updates are executed using the FP32 master weight copy and FP32 gradients (optimizer states are also maintained in FP32);
- The updated FP32 master weights are then cast back to FP16 for the next round of forward/backward computation.

 3. **Loss Scaling**: FP16 has a much smaller dynamic range than FP32. Gradient values in backward passes can be extremely small (close to or below the minimum representable positive number in FP16), and directly computing in FP16 leads to gradient underflow to 0. The approach of loss scaling is:

- Before backward propagation, multiply the loss value by a large scale factor so that gradients during backward propagation are scaled up accordingly to prevent underflow;
- Before using gradients to update parameters, divide the gradients by the same scale factor to restore them to their correct numerical range;
- **Dynamic Loss Scaling**: Automatically adjust the scale value. If gradients are detected containing Inf/NaN (indicating the scale is too large causing overflow), lower the scale and skip this step's update; if no overflow occurs for multiple consecutive steps, appropriately increase the scale to use as large a scale value as possible without overflowing.

 4. **Operator Precision Selection**: Not all operations are suitable for FP16/BF16 computation. Operations sensitive to numerical precision (such as reduction sums in LayerNorm, Softmax, loss computation, etc.) are typically still executed in FP32 to ensure numerical stability. Frameworks (such as PyTorch AMP) usually handle this "operator-wise precision selection" logic automatically.  Through these mechanisms, mixed precision training can significantly increase training speed and reduce GPU memory footprint with virtually no degradation in final model accuracy, making it a standard configuration in modern LLM training.

</details>

**43. How Is Gradient Overflow Handled in Mixed Precision Training?**

<details>
<summary>💡 Show Answer</summary>

In mixed precision (especially FP16) training, gradient overflow (gradient values becoming Inf or NaN) is a common issue, which is mainly detected and handled through the following methods:  **1. Detecting Overflow**

- After gradients are computed in each backward pass, check whether the gradient tensors contain Inf or NaN (typically determined efficiently through aggregated statistics of gradients, such as checking whether the maximum absolute value or sum of gradients is Inf/NaN, rather than element-wise inspection).

 **2. Dynamic Loss Scaling Response Strategy**

- If gradient overflow is detected:
- **Skip current parameter update**: Do not use this gradient update (containing Inf/NaN) to update model parameters, effectively discarding this step;
- **Reduce loss scale**: Lower the loss scale factor by a certain ratio (e.g., halved); the next backward pass will use a smaller scale, thereby reducing the probability of gradient overflow;
- **Reset overflow counter**: Record "the number of consecutive non-overflowing steps since the last scale reduction".
- If no overflow occurs for multiple consecutive steps (e.g., 2000 steps):
- **Appropriately increase loss scale** (e.g., doubled) to fully utilize FP16's representation range, avoid gradient underflow, and improve numerical precision utilization.
- This "reduce upon overflow, increase upon sustained stability" strategy enables the loss scale to adaptively remain within a reasonable range that avoids both overflow and underflow.

 **3. Gradient Clipping**

- Clip the global norm (or per-parameter norm) of gradients prior to parameter updates to cap the maximum gradient magnitude. This mitigates numerical issues caused by excessively large gradients and promotes training stability. It is usually combined with loss scaling (clipping should be performed after unscaling gradients—dividing by loss scale).

 **4. Using BF16 as a Replacement for FP16**

- Because BF16 shares the same exponent bits as FP32, it offers a wider dynamic range, making the probability of gradient overflow/underflow significantly lower than FP16. Many LLM training setups directly adopt BF16 and **do not use loss scaling**, fundamentally bypassing FP16's overflow problems (though trade-offs regarding BF16's lower mantissa precision must be weighed).

 **5. Monitoring and Alerting**

- Record metrics such as loss scale trends and update skip frequencies in training logs. If loss scale is persistently suppressed to very small values or the update skip frequency rises abnormally, it often signals deeper underlying issues in training (e.g., excessively high learning rate, data anomalies, or numerical instability in model architecture) that require further investigation, rather than relying solely on automated loss scaling mechanisms to "mask" the problem.

</details>

**44. How to Debug Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

![Distributed training failure diagnosis and debugging workflow.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig27_distributed_training_debugging.png)




Debugging distributed training is significantly more complex than single-GPU training. Common methods and tools include:  **1. Validate correctness on single-GPU/small scale first**

- Before introducing distributed training, ensure that the model, data processing, loss computation, etc., can converge normally on a single GPU (or small scale with a small amount of data) to eliminate bugs in the model itself;
- Use small-scale configurations (e.g., 2-GPU data parallelism) to quickly reproduce distributed-related issues, then gradually scale up to larger scales, avoiding direct debugging on large clusters which leads to high costs and difficulty in reproducing issues.

 **2. Numerical consistency check**

- Compare whether the loss, gradient norm, etc., of the first few steps in distributed training and single-GPU training (or between different parallel configurations) are consistent (within an acceptable numerical error range) under the same inputs and initialization. This is used to verify whether the parallel implementation (e.g., split logic in tensor parallelism, parameter synchronization in ZeRO) is correct.
- Common techniques: fix random seeds, disable sources of randomness such as Dropout, and compare intermediate results (e.g., output or gradient of a specific layer).

 **3. Logging and monitoring**

- Output logs with rank numbers from each rank to distinguish outputs from different processes;
- Monitor key metrics: loss, gradient norm (grad norm), learning rate, loss scale (mixed precision), throughput and GPU memory usage of each rank. Abnormal values (e.g., one rank having a significantly higher loss than other ranks) often suggest issues in data allocation or parallel implementation.
- Use tools like TensorBoard and Weights & Biases to visualize training curves for easy detection of abnormal fluctuations.

 **4. Troubleshooting communication and deadlock issues**

- When training hangs, a common reason is that a rank exits due to an exception (e.g., OOM, Python exception) while other ranks are still waiting for collective communication. You can set the NCCL timeout (e.g., `NCCL_TIMEOUT`) so that stuck communications throw errors instead of waiting indefinitely;
- Enable NCCL debug logging (`NCCL_DEBUG=INFO`) to view detailed information during communication initialization and execution, locating which communication operation and rank encountered issues;
- Check whether the calling order and frequency of collective communication operations are identical across all ranks (inconsistent calling order is a common cause of deadlock, e.g., a rank invoking AllReduce one fewer time due to a different conditional branch).

 **5. Performance bottleneck analysis (Profiling)**

- Use tools such as PyTorch Profiler and NVIDIA Nsight Systems to perform performance profiling on the training process, distinguishing the proportion of computation time, communication time, and data loading time;
- Compare throughput and MFU under different parallel configurations to pinpoint whether the bottleneck lies in computation, communication, or I/O.

 **6. GPU memory troubleshooting**

- Use GPU memory analysis tools (such as `torch.cuda.memory_summary()`) to inspect the breakdown of GPU memory usage (parameters, gradients, optimizer states, activations, cache, etc.) and locate where the peak GPU memory occurs;
- Pinpoint the specific cause of OOM by gradually enabling/disabling activation recomputation, adjusting micro-batch size, etc.

 **7. Small-scale reproduction + binary search isolation**

- When an issue is difficult to locate directly, reproduce it by gradually simplifying the configuration (reducing parallel dimensions, shrinking model size, reducing data volume), and then use binary search/elimination to step-by-step rule out modules like parallel strategies, data processing, and model architecture, narrowing down the scope of the problem.

 Overall, the core methodology for debugging distributed training is: **first ensure correctness on a single machine/small scale, then gradually expand the scale and verify consistency through meticulous logging, monitoring, and numerical comparison; when encountering deadlocks, OOM, or performance issues, leverage specialized debugging and profiling tools to locate the specific root cause**.

</details>

**45. How to Optimize Communication in Distributed Training?**

<details>
<summary>💡 Show Answer</summary>

Common methods for communication optimization in distributed training include:  **1. Communication and computation overlap**

- During backward propagation, as soon as the gradient calculation for certain parameters is completed, the corresponding AllReduce communication can start immediately, without waiting for the entire backward pass to finish (the bucket mechanism in PyTorch DDP is based on this idea: parameters are grouped into multiple buckets; once the gradients of a bucket are computed, the AllReduce for that bucket is triggered and executed in parallel with subsequent gradient calculations).
- In pipeline parallelism, forward/backward computation of different micro-batches can also overlap as much as possible with inter-stage activation/gradient transfers.

 **2. Gradient compression and quantization**

- Compress the gradients to be transmitted by using methods such as low-bit quantization (e.g., quantizing gradients to 8-bit or lower), sparsification (transmitting only larger gradient components and accumulating the rest into the next step), or low-rank decomposition to reduce communication volume. The precision loss caused by compression needs to be balanced.

 **3. Communication primitive selection and topology optimization**

- Select appropriate communication algorithms according to the hardware topology (such as Ring AllReduce vs Tree AllReduce). Libraries like NCCL usually select them automatically, but manual tuning may be needed in custom implementations or special topologies;
- Properly design parallel strategies so that parts with the highest communication frequency and data volume (e.g., AllReduce in tensor parallelism) are placed on the highest-bandwidth connections (intra-node NVLink), while parts with lower communication frequency (e.g., data parallelism, pipeline parallelism) are placed on inter-node connections.

 **4. Reducing communication calls (Bucketing/Fusion)**

- Merge multiple small communication operations into fewer large operations (e.g., concatenating gradients of multiple small tensors into a large tensor before performing a single AllReduce), because each communication call incurs fixed latency overhead. Merging communications amortizes this fixed overhead and improves effective bandwidth utilization.

 **5. Communication scheduling in memory optimization techniques such as ZeRO**

- In ZeRO-3, the AllGather for the next layer's parameters can be initiated in advance via a prefetch mechanism to overlap with current layer computation;
- Properly set the communication granularity of ZeRO (e.g., bucket size) to balance communication efficiency and GPU memory footprint.

 **6. Mixed-precision communication**

- Use FP16/BF16 instead of FP32 for communication during gradient synchronization (even if master weights and optimizer states are FP32), halving the transmitted data volume.

 **7. Reducing inter-node communication volume**

- For pipeline parallelism, place adjacent pipeline stages on nodes that are network-wise closer to each other as much as possible to reduce cross-rack or cross-switch communication;
- For MoE All-to-All communication, try to restrict expert parallel groups within a node or within an optimal network topology range.

 **8. Network hardware level**

- Use higher bandwidth, lower latency interconnects (such as InfiniBand HDR/NDR, NVLink/NVSwitch), and enable GPUDirect RDMA to reduce CPU involvement and data copying.

 Overall, the core methodology for communication optimization is: **reduce communication volume (compression, quantization, mixed precision), reduce communication frequency/calls (fusion, bucketing), overlap communication with computation, and align communication patterns with hardware topology**. Multiple techniques often need to be combined and their practical efficacy verified through profiling.

</details>

**46. What Common Evaluation Metrics Are Used for Training Efficiency?**

<details>
<summary>💡 Show Answer</summary>

![LLM training efficiency metrics: MFU, HFU, and GPU throughput.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig28_training_efficiency_metrics.png)




Common metrics used to evaluate the training efficiency of large models include:  1. **Throughput**:

- **Tokens/second**: The number of tokens processed per unit of time, which is the most commonly used core efficiency metric in LLM training;
- **Samples/second** or **Step time**.

 2. **MFU (Model FLOPs Utilization)**: The percentage of actual compute throughput (FLOPs/s) relative to the hardware theoretical peak compute performance. It reflects the effective utilization degree of compute resources and is an important metric measuring training "efficiency" rather than mere "speed", because it eliminates the impact of factors like model size and batch size, facilitating cross-configuration comparisons.  3. **GPU Utilization**: The occupancy rate of GPU compute units observed via `nvidia-smi` or monitoring systems. It reflects whether the GPU is busy, but does not directly equal "effective computation" (as it may include idling, waiting for communication, etc.).  4. **Memory Utilization**: The ratio of actual GPU memory occupancy to the total GPU memory during training. A ratio that is too low indicates underutilization of GPU memory (which can be improved by increasing batch size, etc.), while a ratio that is too high poses an OOM risk.  5. **Speedup and Scaling Efficiency**: The throughput speedup multiple achieved using N devices relative to 1 device, and the scaling efficiency obtained by dividing this multiple by N, reflecting the "cost-effectiveness" of distributed scaling.  6. **Communication Overhead Ratio**: The proportion of communication time to the total time in one training step, measured via profiling tools. A higher ratio indicates a more severe communication bottleneck.  7. **Metrics related to total training cost**:

- **Training cost per sample/token (e.g., USD or GPU-hours per million tokens)**: Comprehensively considers hardware cost and training time, used to evaluate the economic efficiency of training;
- **Total compute (FLOPs) or total time required to reach the target loss/performance**: Used to compare the overall efficiency of different training schemes (different model sizes, data volumes, hyperparameter configurations).

 8. **Training stability-related metrics (indirectly reflecting efficiency)**: Such as the frequency of loss spikes, the proportion of effective training time lost due to failure restarts, etc., which indirectly reduce actual effective training efficiency.  In practice, attention is usually focused comprehensively on three types of metrics: **tokens/second (absolute speed)**, **MFU (resource utilization efficiency)**, and **scaling efficiency (distributed scaling quality)**, combined with training costs for overall evaluation and optimization decision-making.

</details>

**47. Pros and Cons of BF16 and FP16 Half-Precision Training**

<details>
<summary>💡 Show Answer</summary>

**FP16 Half-Precision Training**  Advantages:

- Compared to FP32, memory footprint is halved, and computation speed (on hardware supporting Tensor Cores) is significantly improved;
- Has 10 mantissa bits, offering higher relative precision than BF16, allowing finer numerical representation when within the allowable numerical range.

 Disadvantages:

- Has only 5 exponent bits, so **its numerical dynamic range is far smaller than FP32** (maximum of approx. $6.5 \times 10^4$), making gradients or activations during training prone to exceeding this range, resulting in **overflow (becoming Inf)** or **underflow (becoming 0)**;
- Must be paired with additional mechanisms like **Loss Scaling** to mitigate dynamic range issues, increasing implementation complexity; furthermore, adjusting the loss scale itself can introduce training instability (such as frequently skipping step updates);
- In certain large model training scenarios sensitive to dynamic range (especially larger-scale models), FP16 is prone to training divergence, requiring more cautious hyperparameter tuning and numerical stability handling.

 **BF16 Half-Precision Training**  Advantages:

- Has the same number of exponent bits as FP32 (8 bits), so **its numerical dynamic range is identical to FP32**, making overflow/underflow almost non-existent and **typically requiring no loss scaling**, simplifying mixed-precision implementation and hyperparameter tuning;
- Also features a halved memory footprint compared to FP32 and faster computation speed (on supported hardware);
- More robust against situations common in LLM training where "values span multiple orders of magnitude" (e.g., certain activation values or gradients may be very large or small), offering generally superior training stability compared to FP16.

 Disadvantages:

- Has only 7 mantissa bits, so **its precision is lower than FP16** (fewer significant digits representable within the same numerical range), which may introduce larger rounding errors in operations requiring high precision (e.g., certain reduction sums);
- Requires native hardware support for BF16 execution to achieve speed benefits (newer GPUs like NVIDIA A100/H100, Google TPUs, etc. have good support, while some older hardware may not support it or have limited support).

 **Summary and Comparison**:

- FP16 is slightly superior to BF16 in precision, but has a narrow dynamic range, is prone to overflow, and requires an extra loss scaling mechanism;
- BF16 has a wide dynamic range, provides greater training stability, and simplifies implementation, though its precision is relatively lower;
- In current LLM training practice, **BF16 has become the preferred half-precision format for many large models (such as LLaMA, PaLM, etc.) due to its training stability and implementation simplicity**, while FP16 is still used in some precision-sensitive scenarios equipped with complete loss scaling mechanisms. Both yield significant memory and speed benefits compared to FP32 and are core components of mixed-precision training.

</details>

**48. What Are the Characteristics of Distributed Parallelism and GPU Memory Optimization Techniques?**

<details>
<summary>💡 Show Answer</summary>

In large language model training, distributed parallelism techniques and GPU memory optimization techniques are two complementary core approaches.

#### Distributed Parallelism Techniques
Distributed parallelism accelerates training through multi-device collaboration, mainly including the following paradigms:

- **Data Parallelism (DP)**: Replicates the same model across multiple devices, with each device processing different data batches and gradients synchronized/aggregated across devices. It is simple to implement, but each device still needs to store the complete model, resulting in high memory pressure.
- **Tensor Parallelism (TP)**: Splits the model weight matrices by rows or columns across multiple devices, so each device stores only a portion of the parameters, making it suitable for single layers with extremely large parameter counts. Communication overhead is concentrated within operators with higher latency, so it is usually used intra-node.
- **Pipeline Parallelism (PP)**: Divides the model by layers into multiple stages assigned to different devices, with data flowing in a pipelined fashion. It can significantly reduce per-device memory usage, but requires carefully designed micro-batch scheduling to minimize bubble time.
- **Hybrid Parallelism**: Combines multiple parallelism methods above (i.e., "3D Parallelism")---using tensor parallelism intra-node, pipeline parallelism inter-node, and data parallelism globally---to fully utilize bandwidth resources and control communication costs.

The core characteristic of distributed parallelism is **Scale-out**: the more hardware, the higher the theoretical throughput; however, it also introduces engineering complexity such as communication overhead, load balancing, and fault tolerance.

#### GPU Memory Optimization Techniques
GPU memory optimization focuses on **Scale-up**, i.e., training larger models under fixed hardware conditions. Key approaches include:

- **ZeRO Series**: Partition optimizer states, gradients, and parameters across multiple devices to completely eliminate redundant storage in data parallelism.
- **Activation Recomputation (Gradient Checkpointing)**: Retain only a subset of intermediate activations during the forward pass and recompute them on demand during the backward pass, trading time for space.
- **Mixed Precision Training**: Use FP16/BF16 for forward and backward passes and FP32 master weights for optimizer updates, balancing speed and numerical stability.
- **Memory Defragmentation and Operator Fusion**: Reduce memory fragmentation and temporary buffer overhead through memory pool management and fused operators.

The two types of techniques are typically **used synergistically**: first use GPU memory optimization techniques to lower single-device usage, and then use distributed parallelism techniques to scale out horizontally, thereby training ultra-large-scale models under limited resources.

% -------------------------------------------------------

</details>

**49. What Are the GPU Memory Optimization Techniques and Their Characteristics?**

<details>
<summary>💡 Show Answer</summary>

GPU VRAM is one of the core bottlenecks in large model training. During training, GPU memory is primarily occupied by the following categories of data: model parameters, gradients, optimizer states (such as Adam's first/second moments), intermediate activations from forward propagation, and runtime temporary buffers of the framework. Targeting these memory consumers, the industry has developed various GPU memory optimization techniques:

- **ZeRO (Zero Redundancy Optimizer)**:
    Proposed by DeepSpeed, divided into three stages:
    - **ZeRO-1**: Partitions optimizer states (accounting for approx. $\frac{12}{N}$ of memory) across N data-parallel processes.
    - **ZeRO-2**: Further partitions gradients on top of ZeRO-1.
    - **ZeRO-3**: Partitions parameters, gradients, and optimizer states entirely, theoretically reducing per-card memory to $\frac{1}{N}$, but at the cost of increased communication.
    Characteristics: Low invasiveness to user code, naturally compatible with data parallelism, but communication overhead increases as partitioning deepens.

- **Activation Recomputation (Activation Checkpointing / Gradient Checkpointing)**:
    Saves only the input of each Transformer layer (checkpoint) during forward propagation and discards intermediate activations; during backward propagation, re-executes forward computation from checkpoints to reconstruct activation values.
    Characteristics: Reduces activation memory from $O(L \cdot B \cdot S)$ to $O(\sqrt{L} \cdot B \cdot S)$ ($L$ is the number of layers, $B$ is batch size, $S$ is sequence length), at the cost of approximately $33%$ extra computation.

- **Mixed Precision Training**:
    Leverages Tensor Cores on modern GPUs to execute compute-intensive operations such as matrix multiplications in FP16 or BF16, while retaining FP32 master weights for optimizer updates, using loss scaling to prevent FP16 underflow.
    Characteristics: Reduces GPU memory footprint by approximately $50%$, increases training speed by $2\sim3\times$; BF16 offers a wider dynamic range and better numerical stability than FP16, making it the current mainstream choice.

- **CPU Offload**:
    Offloads optimizer states or even parameters to CPU memory, transferring them back to the GPU on demand during training. Both DeepSpeed ZeRO-Infinity and ZeRO-Offload support this feature.
    Characteristics: Tremendously expands trainably supported model scales, but PCIe bandwidth becomes a bottleneck, leading to a noticeable drop in training speed.

- **Operator Fusion and Memory Reuse**:
    Fuses multiple consecutive small operators (e.g., LayerNorm + Dropout + Add) into a single CUDA kernel, reducing memory read/write of intermediate results. FlashAttention is a prime example, reducing Attention memory complexity from $O(N^2)$ to $O(N)$.
    Characteristics: Saves memory while improving computational efficiency, but has higher invasiveness to code and is usually implemented at the framework level.

- **Quantization-Aware Training (QAT)**:
    Simulates quantization errors (INT8/INT4) during the training process to make the model adapt to low-precision inference.
    Characteristics: Has smaller precision loss compared to post-training quantization, but involves a more complex training workflow.

% -------------------------------------------------------

</details>

**50. Why Still Need 3D Parallelism When We Have the ZeRO Series?**

<details>
<summary>💡 Show Answer</summary>

ZeRO series and 3D parallelism overlap in the problems they solve, but they have different focuses, and both are often used in combination for ultra-large-scale training scenarios.

#### Limitations of the ZeRO Series

ZeRO is essentially a GPU memory optimization for **data parallelism**, where the core idea is to eliminate redundant storage. However, it has the following inherent limitations:

- **Communication volume increases as sharding deepens**: ZeRO-3 requires All-Gather for parameters during forward and backward passes and Reduce-Scatter for gradients, resulting in roughly $3\times$ the communication volume of ZeRO-1. When inter-node bandwidth (e.g., 100GbE) is far lower than intra-node bandwidth (e.g., NVLink 600GB/s), the cross-node communication of ZeRO-3 becomes a severe bottleneck.
- **Lower bound on single-layer parameter count**: If the parameter count of a single layer itself exceeds the GPU memory of a single card, ZeRO cannot solve this issue—it can only scatter "who stores what," but during the computation of a layer, the complete parameters of that layer still need to be gathered onto a single card.
- **Sequence length limitation**: Activation memory footprint is positively correlated with sequence length; ZeRO does not directly optimize activation memory (ZeRO-R has partial optimization, but its effectiveness is limited).

#### Complementarity of 3D Parallelism

3D parallelism (Tensor Parallelism + Pipeline Parallelism + Data Parallelism) provides structural solutions to the above problems:

- **Tensor Parallelism** splits single-layer parameters across multiple GPUs, fundamentally solving the issue of overly large single layers, and communication only occurs over intra-node high-speed interconnects (NVLink) with ample bandwidth.
- **Pipeline Parallelism** partitions the model layer-by-layer, reducing the parameter count per device, and inter-layer communication only involves activations (much smaller than parameters), making it friendly to cross-node bandwidth.
- **Data Parallelism** (which can be combined with ZeRO) is responsible for batch scaling.

#### Practical Recommendations

In trillion-parameter scale training (such as GPT-3/4 scale), the common strategy is:
1. Use Tensor Parallelism (TP=8) intra-node (8 GPUs) to fully utilize NVLink;
2. Use Pipeline Parallelism (PP=a certain number of nodes) inter-node, where the communication volume consists only of activations;
3. Use Data Parallelism across pipeline groups, combined with ZeRO-1 or ZeRO-2 to further optimize GPU memory.

This avoids the high cross-node communication volume of ZeRO-3 while making full use of intra-node high bandwidth.

% -------------------------------------------------------

</details>

**51. Are There Other Ways for Large-Scale Training Besides 3D Parallelism?**

<details>
<summary>💡 Show Answer</summary>

Besides 3D parallelism, the industry has also developed various large-scale training solutions, each with its own focus:

- **ZeRO + DeepSpeed**:
    Microsoft's DeepSpeed framework takes the ZeRO series as its core, accompanied by extensions such as ZeRO-Infinity (CPU/NVMe Offload) and ZeRO++, enabling efficient training of models from tens to hundreds of billions of parameters on communication-constrained clusters. ZeRO++ introduces quantized communication (qgZ) and hierarchical All-Reduce to further reduce cross-node communication volume.

- **FSDP (Fully Sharded Data Parallel)**:
    PyTorch's official implementation of fully sharded data parallelism, similar in functionality to ZeRO-3, has been deeply integrated into the PyTorch ecosystem (`torch.distributed.fsdp`). It supports hybrid sharding strategies (`HYBRID_SHARD`), performing full parameter synchronization intra-node and sharding inter-node, balancing GPU memory and communication efficiency. The official training scripts for the Llama series models use FSDP.

- **Megatron-LM**:
    NVIDIA's open-source large model training framework, famous for its efficient tensor parallelism implementation. Its Sequence Parallelism technology shards and stores activation values of LayerNorm and Dropout, further reducing memory usage. Megatron-LM is often used in combination with DeepSpeed (i.e., Megatron-DeepSpeed).

- **Sequence Parallelism (Sequence Parallelism / Ring Attention)**:
    Targeting training with ultra-long sequences (such as 128K tokens), the sequence dimension is sharded across multiple devices, and Attention computation passes Key/Value between devices via Ring communication patterns, making ultra-long context training possible. Representative implementations include LongLoRA and DeepSpeed-Ulysses.

- **Expert Parallelism**:
    Targeting Mixture of Experts (MoE) models, different experts are distributed across different devices, and token routing is performed via All-to-All communication. Compared to dense models, MoE can scale parameter size under the same computation budget, serving as the core architecture for models such as GPT-4 and Mixtral.

- **Parameter-Efficient Fine-Tuning (PEFT)**:
    For downstream task fine-tuning, methods like LoRA, Prefix Tuning, and Adapters train only a small number of additional parameters (typically less than $1%$ of the original parameters), allowing fine-tuning tasks that originally required dozens of GPUs to be completed on a single GPU or a few GPUs, significantly lowering the threshold for large-scale training.

- **Cloud Managed Training Services**:
    Cloud services such as AWS SageMaker, Google Cloud TPU Pods, and Azure ML provide out-of-the-box large-scale training infrastructure, enabling users to perform ultra-large-scale training without managing clusters themselves.

% -------------------------------------------------------

</details>

**52. Is 3D Parallelism Suitable for Everyday Users?**

<details>
<summary>💡 Show Answer</summary>

Short conclusion: **The threshold is high, and everyday users should evaluate carefully, but it is not completely infeasible.**

#### Resource and Technical Requirements of 3D Parallelism

- **Hardware requirements**: Tensor parallelism requires intra-node high-speed interconnects (NVLink); otherwise, communication becomes the bottleneck. Pipeline parallelism requires at least dozens of GPUs to be fully effective. A typical setup is $8\times$ A100/H100 nodes, which is extremely costly.
- **Debugging complexity**: Bugs under 3D parallelism are often difficult to reproduce and pinpoint, involving multiple dimensions such as multi-processing, multi-machine communication, and numerical synchronization.
- **Hyperparameter tuning**: The interactions among hyperparameters such as micro-batch size, pipeline stage count (PP degree), and gradient accumulation steps are complex, requiring extensive experimentation to find optimal configurations.
- **Framework learning curve**: Megatron-LM's codebase is relatively low-level, making customized modifications difficult; Colossal-AI provides friendlier wrappers, but its documentation and community support are relatively weaker.

#### Under What Circumstances Can Everyday Users Try?

- The goal is **learning and experimentation** rather than production training: basic 3D parallelism workflows can be verified with smaller models (e.g., 1B parameters) on $4\sim8$ consumer-grade GPUs (RTX 3090/4090).
- Using **high-level wrapper frameworks**: Frameworks like Colossal-AI and LLaMA-Factory lower the barrier to entry for 3D parallelism, allowing execution with just a few lines of configuration.
- Leveraging **cloud GPU clusters**: Renting multi-machine multi-GPU instances on the cloud (e.g., Lambda Labs, CoreWeave) allows experiencing 3D parallelism without purchasing hardware.

#### Recommended Alternatives for Everyday Users

For everyday users without large-scale clusters, the following solutions are often more practical:
1. Single-machine multi-GPU + FSDP or ZeRO-2: Covers full fine-tuning needs for models under 70B.
2. LoRA/QLoRA: 4-bit quantization + LoRA allows fine-tuning a 70B model on a single 24GB VRAM card.
3. Inference optimization (vLLM, llama.cpp): Deploy quantized models directly for inference without training.

% -------------------------------------------------------

</details>

**53. Is Multi-Machine Multi-GPU ZeRO-3 Suitable for Everyday Users?**

<details>
<summary>💡 Show Answer</summary>

Short conclusion: **Technically feasible, but the engineering challenges cannot be overlooked; a step-by-step approach is recommended.**

#### Advantages of Multi-Machine Multi-GPU ZeRO-3

- Theoretically, the GPU memory consumption per card can be reduced to $\frac{1}{N}$ of a single card ($N$ is the total number of GPUs), enabling ultra-large model training under limited single-card memory.
- DeepSpeed provides a complete ZeRO-3 implementation driven by configuration files with minimal code modifications.
- Deeply integrated with HuggingFace `Accelerate`, enabling everyday users to launch quickly via `accelerate config`.

#### Challenges of Multi-Machine Multi-GPU ZeRO-3

- **Network bandwidth bottleneck**: ZeRO-3's cross-machine communication volume is roughly $6\times$ the model parameter size (forward All-Gather + backward All-Gather + Reduce-Scatter). If inter-machine connectivity is limited to 10GbE, GPU utilization for large model training may fall below $30%$. Ideally, InfiniBand (200Gb/s+) or at least 100GbE RDMA is required.
- **Complex environment configuration**: Multi-machine training requires configuring passwordless SSH, NCCL environment variables (`NCCL_IB_HCA`, `NCCL_SOCKET_IFNAME`, etc.), making network failures difficult to troubleshoot.
- **Fault tolerance and checkpointing**: In multi-machine training, if any node fails, the entire task crashes. Proper checkpoint saving strategies (e.g., saving every N steps) and stable shared storage (NFS or distributed filesystem) must be configured.
- **Cost control**: Multi-machine multi-GPU cloud setups (e.g., 8×A100 × 2 nodes) can cost tens of dollars per hour, easily generating high bills if debugging is done carelessly.

#### Practical Recommendations for Everyday Users

\begin{lstlisting}[style=promptstyle]
	Recommended path:
	1. Single-machine single-GPU -> Verify training pipeline correctness (using small model, small dataset)
	2. Single-machine multi-GPU + ZeRO2 -> Familiarize with DeepSpeed config, troubleshoot local environment issues
	3. Single-machine multi-GPU + ZeRO3 -> Verify parameter sharding logic, measure communication overhead
	4. Multi-machine multi-GPU + ZeRO3 -> Confirm sufficient network bandwidth before scaling horizontally
\end{lstlisting}

Before officially moving to multi-machine setups, it is recommended to use the `nccl-tests` tool to benchmark actual inter-machine communication bandwidth, confirming that `all_reduce` bandwidth meets expectations before training models.

% -------------------------------------------------------

</details>

**54. What Are Memory-Efficient LLM Training / Fine-Tuning / Inference Methods?**

<details>
<summary>💡 Show Answer</summary>

1. **Quantization Techniques**
    - **Post-Training Quantization (PTQ)**: Quantizes the weights of an already trained model (INT8/INT4/NF4) without requiring retraining. Representative tools: `bitsandbytes` (LLM.int8()), GPTQ, AWQ. VRAM footprint can be reduced by $2\sim4\times$, with accuracy loss usually within an acceptable range.
    - **Quantization-Aware Training (QAT)**: Simulates quantization noise during training so the model adapts to low precision, yielding smaller accuracy loss but at higher training cost.
    - **Mixed-Precision Training (FP16/BF16)**: Computations use half-precision while parameter updates retain full precision; it is standard for almost all current LLM training, reducing VRAM by about $50%$ and speeding up training by $2\sim3\times$.

2. **Model Optimization and Parameter-Efficient Fine-Tuning (PEFT)**
    - **LoRA (Low-Rank Adaptation)**: Adds low-rank decomposition matrices ($\Delta W = AB$, $A \in \mathbb{R}^{d \times r}$, $B \in \mathbb{R}^{r \times k}$, $r \ll \min(d,k)$) alongside the original weight matrix, training only $A$ and $B$ while freezing the original weights. Reduces trainable parameters to $0.1%\sim1%$ of the original, significantly reducing GPU memory used by gradients and optimizer states.
    - **QLoRA**: Uses LoRA on top of a 4-bit quantized frozen base model, combined with NF4 quantization and Double Quantization, enabling fine-tuning of a 65B model on a single 24GB GPU.
    - **Prefix Tuning / Prompt Tuning**: Trains only a few learnable tokens at the input side while completely freezing the main model, suitable for extremely memory-constrained scenarios.
    - **Gradient Checkpointing (Activation Recomputation)**: Trades about $33%$ extra computation for a drastic reduction in activation memory. Almost all mainstream training frameworks support it, enabled with a single line of code: `model.gradient_checkpointing_enable()`.

3. **Distributed Training Optimization**
    - **ZeRO (Zero Redundancy Optimizer)**: Shards optimizer states, gradients, and parameters across multiple GPUs in three stages, reducing VRAM per card down to $\frac{1}{N}$ at maximum ($N$ is the number of GPUs).
    - **FSDP (Fully Sharded Data Parallel)**: PyTorch native implementation, similar in function to ZeRO-3, supporting hybrid sharding strategies and deeply integrated into the PyTorch ecosystem; it is the official training solution for Llama.
    - **CPU/NVMe Offload**: Offloads optimizer states or parameters to CPU memory or even NVMe drives (DeepSpeed ZeRO-Infinity), extending trainable model size to tens of times the single-machine GPU memory at the cost of reduced training speed.

4. **Inference Optimization**
    - **KV Cache Optimization**:
        - **Multi-Query Attention (MQA) / Grouped-Query Attention (GQA)**: Reduces the number of KV heads to cut down KV Cache memory usage; both Llama 3 and Mistral adopt GQA.
        - **PagedAttention**: Memory management technique proposed by vLLM that dynamically allocates KV Cache in units of pages, eliminating memory fragmentation and boosting GPU utilization from $\sim30%$ to $\sim90%$.
    - **Speculative Decoding**: Uses a small model (Draft Model) to quickly generate multiple candidate tokens, which are verified in parallel by the large model, increasing inference throughput by $2\sim3\times$ without accuracy loss.
    - **Continuous Batching**: Dynamically schedules requests of different lengths to maximize GPU utilization, avoiding GPU memory waste under fixed batching.
    - **Quantized Inference**: Uses quantization formats like GPTQ, AWQ, and GGUF (llama.cpp) to compress models to INT4/INT3, allowing 10B+ parameter models to run on consumer GPUs or even CPUs.
    - **FlashAttention**: Rewrites the Attention operator, leveraging IO-aware tiled computation to reduce Attention memory complexity from $O(N^2)$ to $O(N)$ while boosting computation speed; it has become standard in almost all mainstream frameworks.

</details>

**55. Communication Primitives Such as AllReduce / AllGather / All2All**

<details>
<summary>💡 Show Answer</summary>

Assume there are $N$ devices, with each device holding a block of data.

**AllReduce**:
- Each device's data is first **reduced (reduce, e.g., summation)**, and then the reduced result is **broadcast** to all devices.
- Result: All devices have the same complete, reduced result.
- Applications: Gradient synchronization in data parallelism, summation/merging of partial results in tensor parallelism.

**AllGather**:
- Each device contributes its own chunk of data, and all devices end up with the **concatenation of data from all devices** (no reduction operation).
- Result: Each device goes from "having only its own share" to "having $N$ complete shares of data."
- Applications: Gathering partial outputs computed by each GPU in tensor parallelism, gathering parameter/gradient shards in ZeRO.

**ReduceScatter**:
- First reduces the data across all devices, but the reduced result is **scattered** according to the number of devices, so each device only receives a portion of the result.
- Can be viewed as the "inverse operation" of AllGather; AllReduce = ReduceScatter + AllGather.
- Applications: Gradient scattering in ZeRO-2/3.

**All2All (AllToAll)**:
- Each device **splits its data into $N$ parts**, sending the $i$-th part to the $i$-th device; simultaneously, each device receives one part from each of the other $N-1$ devices and concatenates them.
- It is a "transpose-style" all-to-all exchange communication, where the data sent and received by each device are distinct (unlike AllReduce/AllGather where all devices get identical results).
- Applications: **Token routing to different experts in MoE models** (tokens on each GPU need to be sent to the GPU where the corresponding expert resides, and then the expert computation results are sent back); core communication primitive for Expert Parallelism (EP).

**Broadcast**: Data from a single device is sent to all devices (one-to-many).

**Scatter**: Data from a single device is split and distributed to different devices (one-to-many, but each device receives a different portion).

</details>

**56. Ring AllReduce vs Tree AllReduce**

<details>
<summary>💡 Show Answer</summary>

![Ring AllReduce communication topology for distributed GPU synchronization.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig37_ring_allreduce_topology.png)




**Ring AllReduce**:
- $N$ devices form a logical ring, and each device only communicates with its left and right neighbors.
- Divided into two phases:
1. **ReduceScatter Phase**: Data is split into $N$ blocks. Over $N-1$ steps, in each step, a device sends a data block it holds to the next device and accumulates it. Eventually, each device holds one complete block of reduced results.
2. **AllGather Phase**: Over another $N-1$ steps, the reduced result blocks held by each device are propagated along the ring. Eventually, every device possesses all reduced results.
- **Communication volume analysis**: The total communication volume for each device is $\frac{2(N-1)}{N} \times total data size$, **independent of the number of devices $N$ (approaching $2 \times data size$ when $N$ is large)**, making it a **bandwidth-optimal** algorithm.
- **Applicable scenarios**: Scenarios with a large number of devices, large data volume per communication, and where bandwidth is the bottleneck (such as gradient synchronization in large model training); the disadvantage is that latency grows linearly with $N$ (step count is $2(N-1)$).

**Tree AllReduce (Tree/Binary Tree)**:
- Devices are organized into a tree structure. In the reduction phase, data aggregates upward along the tree to the root node (reduce), and in the broadcast phase, data is distributed downward from the root node (broadcast).
- The number of communication steps is $O(\log N)$, offering **superior latency**.
- However, communication bandwidth utilization across layers is unbalanced (links near the root node become bottlenecks), so overall bandwidth efficiency is inferior to Ring.
- **Applicable scenarios**: Scenarios with fewer devices, smaller data volumes, and high sensitivity to latency (frequent synchronization of small data).

**Summary**: Ring offers optimal bandwidth utilization in large-scale, large-data-volume scenarios and is the mainstream choice (NCCL defaults to Ring for large data); Tree is superior for small data or scenarios requiring reduced latency ($\log N$ steps). NCCL also automatically switches between the two algorithms based on data volume (or uses variants like Double Binary Tree).

</details>

**57. Differences Between TP/EP/PP/SP Parallelism**

<details>
<summary>💡 Show Answer</summary>

![Comparative analysis of Tensor, Expert, Pipeline, and Sequence Parallelism.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig38_tp_ep_pp_sp_comparison.png)




- **TP (Tensor Parallelism)**: Splitting a single operator (e.g., the weight matrix of a Linear layer) by row/column across multiple devices, where each device computes partial results that are then combined via AllReduce/AllGather. It is mainly used to split large matrix operations in attention and FFN layers.
- **PP (Pipeline Parallelism)**: Vertically splitting the model along the layer dimension, where different devices are responsible for different sets of consecutive layers, and data is transmitted between devices in a micro-batch pipelined fashion (transmitting only activations/gradients).
- **EP (Expert Parallelism)**: Dedicated to MoE (Mixture-of-Experts) models. Different experts are distributed across different devices, and tokens are routed via All2All communication to the device where their corresponding experts reside for computation, and then sent back. Essentially, it is "model parallelism" for MoE layers; non-MoE layers (such as attention) usually still use TP/DP.
- **SP (Sequence Parallelism)**:
- In a broad sense: Splitting the sequence dimension across different devices, commonly used to reduce activation memory redundancy in operators like LayerNorm and Dropout in TP setups (SP in Megatron-LM is used alongside TP to split parts not split by TP along the sequence dimension, further lowering memory usage).
- It also refers to splitting the Attention computation for long sequences (such as early forms of Ring Attention / Context Parallelism), where very long sequences are partitioned across multiple devices to compute attention separately before merging.

**Brief Summary**: TP partitions the "weight matrix dimensions", PP partitions the "layers", EP partitions the "experts", and SP partitions the "sequence length dimension"—the four target different dimensions for partitioning and can be combined orthogonally (e.g., 3D/4D multi-dimensional parallelism such as TP+PP+EP+DP, a common configuration in Megatron+DeepSpeed/MoE frameworks).

</details>

**58. Model Weight Partitioning Methods Under TP/EP/PP Parallelism**

<details>
<summary>💡 Show Answer</summary>

**TP (Tensor Parallelism) Weight Partitioning**:
- **Attention Layer**: The QKV projection matrix is split **column-wise** across different GPUs (each GPU handles a subset of attention heads), and the output projection matrix is split **row-wise**, followed by an AllReduce to combine the outputs.
- **FFN Layer**: The first linear layer (dimension expansion, $d_{model} \rightarrow d_{ff}$) is split column-wise, while the second linear layer (dimension reduction, $d_{ff} \rightarrow d_{model}$) is split row-wise. The activation function in between is computed locally and independently on each GPU (no communication needed), with a final AllReduce to combine the outputs.
- **Partitioning Principle**: For two consecutive matrix multiplications, the former is split column-wise and the latter row-wise, allowing intermediate results to be computed locally and independently, requiring only a single AllReduce at the end of the layer.

**PP (Pipeline Parallelism) Weight Partitioning**:
- Partitioned as whole **Transformer layers**, where each GPU/stage holds a complete set of consecutive layers (including all weights of those layers: attention, FFN, LayerNorm, etc.), with no partitioning within single layers.
- The embedding layer and final output layer (LM head) are typically assigned to the first and last stages, respectively.

**EP (Expert Parallelism) Weight Partitioning**:
- Applies only to MoE layers: **Different experts** (each expert being a full FFN) are assigned to different GPUs, with each GPU holding the complete weights of several experts.
- Attention layers and router weights are generally not expert-partitioned; they are either replicated (DP) or partitioned using TP.
- Tokens are dynamically routed to the corresponding GPU holding their target experts via All2All communication for computation.

**Combined Strategies (e.g., 3D/4D Parallelism with TP+EP+PP)**:
- Within the same Transformer block: Weights of the attention/FFN parts (non-MoE layers) are split via TP; experts in MoE layers are distributed via EP, while the FFN matrices inside each expert can be further split via TP (two-level TP$\times$EP partitioning).
- Between different blocks: Partitioned across different stages via PP.
- Overall, this forms a hierarchical distributed weight layout of "inter-layer PP partitioning $\rightarrow$ intra-layer hybrid TP/EP partitioning", where the final weights held by each GPU = several layers of a PP stage $\times$ the slice assigned along the TP/EP dimensions for those layers.

</details>

