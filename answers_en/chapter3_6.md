# 3.6 Model Compression — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 12 questions

**1. What is the Role of Quantization in Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

Quantization reduces resource consumption across multiple dimensions by converting model parameters from floating-point formats (such as FP32 or FP16) to low-precision formats (such as INT8 or even INT4):

- **Reduced GPU Memory Footprint**: FP32 requires 4 bytes per parameter while INT8 requires only 1 byte, theoretically compressing memory requirements to 1/4 of the original, making it possible to run large models that previously could not fit on consumer-grade GPUs.
- **Increased Inference Speed**: Low-precision integer operations are accelerated by dedicated instruction sets on modern hardware (such as Tensor Cores in NVIDIA GPUs), enabling INT8 inference speeds to reach 2--4 times that of FP32.
- **Decreased Bandwidth Pressure**: The bottleneck in model inference is often memory bandwidth rather than computation; halving the parameter bit-width means halving the bandwidth required for each weight load.
- **Reduced Deployment Costs**: Smaller models can be deployed on edge devices, mobile devices, or resource-constrained cloud instances, significantly reducing hardware costs for inference services.

The core idea of quantization is to trade precision for resources: mapping continuous floating-point values to a limited set of discrete integer values and reconstructing the original value distribution as closely as possible using a scale factor and a zero point.

</details>

**2. What is the Difference Between Post-Training Quantization (PTQ) and Quantization-Aware Training (QAT)?**

<details>
<summary>💡 Show Answer</summary>

![Workflow comparison between PTQ and QAT model quantization.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig39_ptq_vs_qat.png)




- **PTQ (Post-Training Quantization)**: Directly quantizes weights after model training is complete, without requiring retraining. Its advantages are simple implementation, high speed, and non-invasiveness to existing models; its disadvantage is that the model has never "seen" quantization errors, leading to relatively larger accuracy degradation, especially in low-bit (e.g., INT4) quantization. PTQ usually requires a small amount of calibration data to compute the distribution range of activations in order to determine quantization parameters.
- **QAT (Quantization-Aware Training)**: Introduces "fake quantization" nodes during training to simulate precision loss caused by quantization, enabling the model to actively adapt to quantization errors during backpropagation and maintain higher precision after quantization. The drawback of QAT is the need for retraining, which incurs high costs, but it offers significant accuracy advantages in extremely low-bit scenarios (such as INT4 or even INT2).

The core difference between the two can be summarized as: PTQ is "post-quantization remediation", while QAT is "prevention during training". For LLMs, due to the extremely high training costs, PTQ (combined with advanced algorithms such as AWQ and GPTQ) is currently the mainstream choice.

</details>

**3. What is the Difference Between Weight Quantization and Activation Quantization in LLMs?**

<details>
<summary>💡 Show Answer</summary>

- **Weight Quantization**: Quantizes the static parameters (weight matrices) of the model. Weights are quantized prior to inference and stored in a low-precision format. Since weight distributions are relatively stable, quantization is less difficult, making it the most common form of LLM quantization (such as Q4_K_M in GGUF format). Weight quantization mainly reduces storage footprint and loading bandwidth, but during actual matrix multiplication, weights usually still need to be dequantized to FP16 for computation (referred to as W4A16).
- **Activation Quantization**: Quantizes the intermediate activations dynamically generated during inference. Activation distributions vary significantly across different inputs (especially given the "outlier" problem in LLMs), making quantization more challenging. However, if activations can also be quantized to INT8, matrix multiplication can be executed entirely in the integer domain (W8A8), leading to more significant hardware throughput gains. Methods like LLM.int8() and SmoothQuant specifically propose solutions for the outlier problem in activation quantization.

In short, weight quantization primarily addresses the issue of "fitting the model into memory", while activation quantization primarily addresses "computing inference quickly". Combining both (W8A8) fully leverages the potential of hardware integer compute units.

</details>

**4. Does INT8 Quantization Significantly Impact Model Accuracy?**

<details>
<summary>💡 Show Answer</summary>

The impact of INT8 quantization on model accuracy is generally manageable, though specific effects vary depending on model scale, quantization strategy, and task type:

- **Accuracy Loss**: For weight quantization (W8A16), accuracy loss is usually minimal, remaining virtually on par with FP16 across most benchmarks ($<1%$ gap). If activations are also quantized (W8A8), methods like SmoothQuant are needed to handle activation outliers; otherwise, the loss will be more significant.
- **Impact of Model Scale**: The larger the parameter count of the model, the better accuracy is preserved after quantization. This is because larger models possess greater redundancy and higher tolerance for quantization errors. Models with fewer than 1B parameters often show more noticeable accuracy loss after INT8 quantization.
- **GPU Memory Optimization**: INT8 quantization compresses each parameter from 2 bytes in FP16 to 1 byte, reducing GPU memory footprint by about 50%. For example, a 70B-parameter model requires about 140GB of GPU memory in FP16, which drops to about 70GB after INT8 quantization.
- **Inference Speed**: On GPUs supporting INT8 Tensor Cores (such as A100, H100), W8A8 inference throughput can reach 1.5--2 times that of FP16. However, since W8A16 still requires dequantization, its acceleration is mainly reflected in bandwidth savings, offering limited speedup for compute-bound scenarios.
- **Applicable Scenarios**: INT8 quantization is the top choice for resource-constrained deployment environments, especially suitable for scenarios where inference services require high concurrency (large batch sizes).

</details>

**5. How Does INT8 Compare to FP16 in Terms of Inference Speed?**

<details>
<summary>💡 Show Answer</summary>

- **INT8**:
    - GPU memory footprint is about 50% of FP16, allowing larger models to be run or larger batch sizes to be processed on the same hardware.
    - On hardware supporting INT8 Tensor Cores (such as NVIDIA A100/H100), the theoretical peak compute throughput is 2 times that of FP16 (A100 INT8 is 2000 TOPS vs. FP16 at 312 TFLOPS).
    - Actual end-to-end inference speedup usually ranges between 1.5--3$\times$, depending on the model architecture and optimization level of the inference framework (such as TensorRT, vLLM).
    - Suitable for scenarios with extremely high latency and throughput requirements and relatively tolerant accuracy expectations.
- **FP16**:
    - GPU memory footprint is 50% of FP32 with accuracy close to FP32, making it the mainstream precision format for current LLM inference.
    - On modern GPUs, FP16 mixed-precision computation has full hardware support (Tensor Cores), delivering a 2--8$\times$ speedup compared to FP32.
    - Higher precision, suitable for scenarios with strict output quality requirements (such as code generation, mathematical reasoning).
    - As a quantization baseline, FP16 is typically the starting format for PTQ methods.

Overall, INT8 is a good trade-off between speed and accuracy, whereas FP16 is a solid choice for accuracy-priority scenarios. In practical deployment, an increasing number of systems adopt a mixed-precision strategy: keeping accuracy-sensitive layers (such as LayerNorm and attention score computation) in FP16 while using INT8 for the remaining layers.

</details>

**6. How to Choose Quantization Precision (e.g., INT8 vs. FP16)?**

<details>
<summary>💡 Show Answer</summary>

Selecting quantization precision requires a comprehensive consideration of hardware conditions, accuracy requirements, latency targets, and deployment scenarios:

- **Scenarios Prioritizing FP16**:
    - Generation quality requirements are extremely high (such as high-risk fields like healthcare and legal).
    - The model itself is small ($<7$B), where quantization gains are limited while accuracy degradation is relatively noticeable.
    - Hardware resources are abundant, eliminating the need to reduce GPU memory requirements via quantization.
- **Scenarios Prioritizing INT8**:
    - GPU memory is limited, requiring larger parameter-count models to run within restricted GPU memory.
    - High-concurrency throughput is pursued, requiring larger batch sizes to be processed on the same GPU.
    - Hardware supporting INT8 acceleration is used (NVIDIA A100/H100, Intel Sapphire Rapids).
    - Tasks have higher accuracy tolerance (such as text summarization, translation, and other generation tasks).
- **More Aggressive Quantization (INT4)**:
    - Need to run 70B-scale models on consumer GPUs (such as RTX 4090 24GB) or CPUs.
    - Recommended to use INT4 schemes optimized for LLMs such as GPTQ, AWQ, and GGUF Q4_K_M, where accuracy loss remains generally acceptable.

Practical Advice: Prioritize FP16 when resources allow; choose INT8 (with bitsandbytes or TensorRT) when GPU memory is insufficient; and consider INT4 (GGUF/llama.cpp) when deploying on local edge devices.

</details>

**7. What are the Steps of AWQ Quantization?**

<details>
<summary>💡 Show Answer</summary>

![AWQ activation-aware salient weight protection and channel scaling mechanism.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig40_awq_activation_aware_quantization.png)




AWQ (Activation-aware Weight Quantization) is a PTQ method designed specifically for LLMs. Its core insight is that only about 1% of "salient weights" in the parameters contribute most to model accuracy, and the channels corresponding to these weights exhibit larger activation magnitudes. The complete steps of AWQ are as follows:

1. **Collect Calibration Data**: Prepare a small set of representative texts (typically several hundred samples) to calculate activation distributions across layers. No labels are needed—only a forward pass is required.
2. **Identify Salient Channels**: Analyze the per-channel mean or maximum of activation values to identify weight channels that have the greatest impact on model outputs (weights corresponding to channels with larger activation magnitudes are more "important").
3. **Calculate Per-channel Scale**: For each input channel of a linear layer, compute a scale factor $s$ based on activation statistics. Multiply the weights by $s$ per channel (protecting/scaling salient weights) while dividing the corresponding activations by $s$ (an equivalent transformation that preserves output).
4. **Perform Weight Quantization**: Perform group-wise quantization on the scaled weights (typically with group size = 128), mapping FP16 weights to INT4 or INT8 integers and saving the quantization parameters (scale and zero point) for each group.
5. **Evaluate Precision**: Evaluate the quantized model on downstream tasks or perplexity metrics to confirm that accuracy loss remains within an acceptable range.

The key advantage of AWQ is that it does not require gradient computation or backpropagation (compared to QAT), enabling fast quantization. At the same time, through its activation-aware scaling strategy, its performance under INT4 precision is significantly superior to naive PTQ methods (such as GPTQ). Currently, AWQ is one of the most popular quantization schemes in LLM production deployment, supported by major frameworks (vLLM, TGI, llm-compressor).

</details>

**8. Why Does Quantization Not Significantly Reduce LLM Accuracy?**

<details>
<summary>💡 Show Answer</summary>

Strictly speaking, quantization **does** reduce LLM accuracy, but this loss is usually minimal and acceptable in practical applications. A more accurate statement is that quantization offers an attractive trade-off between accuracy and efficiency for the following reasons:

- **Over-parameterization of LLMs**: Large language models are often highly over-parameterized with substantial weight redundancy. Small errors in individual parameters do not significantly affect overall output, providing natural fault tolerance for quantization.
- **Scale Effect**: Research shows that larger model parameter counts lead to stronger robustness to quantization. For example, LLaMA-3 70B after INT4 quantization often outperforms unquantized LLaMA-3 8B, as capability gains from scaling far outweigh the loss caused by quantization.
- **Advanced Quantization Algorithms**: Modern quantization methods like GPTQ, AWQ, and SmoothQuant control quantization errors at extremely low levels through activation-aware scaling, layer-wise error compensation, and mixed-precision techniques, allowing INT4 quantized models to perform virtually on par with FP16 across most benchmarks.
- **Reduced Model Size**: Quantized models can be deployed on memory-constrained devices, making inference tasks possible that were previously infeasible.
- **Faster Inference Speed**: Low-precision computation executes faster on dedicated hardware, lowering end-to-end latency.
- **Lower Power Consumption**: Smaller data bit-widths mean fewer memory accesses and computational operations, thereby reducing power consumption—a crucial factor for edge devices and large-scale data center deployments.

The key is that accuracy loss caused by quantization can be continuously reduced through algorithmic innovation. Researchers are constantly developing new techniques, such as mixed-precision quantization (using different precisions for different layers) and protective quantization for specific sensitive layers of the model, making quantization virtually imperceptible in practical applications.

</details>

**9. What is Model Distillation, and How is it Applied to LLMs?**

<details>
<summary>💡 Show Answer</summary>

Model Distillation is a model compression technique whose core idea is to transfer knowledge from a large, high-performance "Teacher Model" to a lightweight "Student Model", enabling the student model to approximate the behavior of the teacher model at a dramatically lower cost.

**Basic Principles of Distillation**:

Unlike training the student model directly with hard labels (one-hot vectors), distillation trains the student model to learn the "soft labels" of the teacher model—that is, the probability distribution of the teacher model's outputs (logits processed by softmax with temperature $T$). Soft labels contain dark knowledge such as inter-class similarities (e.g., the teacher considers a token to have a 30% probability of being the answer and another to have a 20% probability), which carries far richer information than hard labels.

The **distillation loss function** typically consists of two parts:

- **Distillation Loss**: KL divergence between the student's output and the teacher's soft labels (with temperature $T > 1$ to soften the distribution).
- **Task Loss**: Cross-entropy loss between the student's output and the ground-truth labels (with temperature $T = 1$).

**Applications in LLMs**:

- **Output Distillation**: The student model learns the token-level probability distribution of the teacher model, which is the most common form. For example, DistilBERT compresses BERT's parameter count to 40% while retaining about 97% of its performance.
- **Intermediate-Layer Distillation**: The student model also learns intermediate representations of the teacher model (such as attention matrices and hidden states), further improving knowledge transfer efficiency (e.g., TinyBERT).
- **Data Generation Distillation**: Leveraging large models to generate synthetic training data (such as Chain-of-Thought reasoning data), which is then used to fine-tune smaller models so they acquire the reasoning capabilities of large models (e.g., DeepSeek-R1 distilled series).
- **Typical Examples**: GPT-4's capabilities are approximated into smaller models (such as GPT-4o mini) via distillation; Meta trained the instruction-following capabilities of the LLaMA series using synthetic data distillation.

</details>

**10. What is Knowledge Distillation, and How is it Used in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Knowledge Distillation (KD) is the academic term for model distillation, formalized by Hinton et al. in 2015 as a general framework for transferring "knowledge" from large models to small models.

**Core Concepts**:

- **Teacher Model**: A pre-trained large, high-performance model serving as the knowledge source.
- **Student Model**: A small, lightweight model to be trained, aiming to approximate the teacher's capability at low cost.
- **Forms of Knowledge**: Includes response-based knowledge (the teacher's output distribution), feature-based knowledge (the teacher's intermediate representations), and relation-based knowledge (mutual relationships between samples).

**Specific Use Cases in LLMs**:

- **SFT Data Generation**: Using powerful teacher LLMs (such as GPT-4, DeepSeek-R1) to generate high-quality instruction-response pairs as fine-tuning data for student models. This is currently the most mainstream form of "distillation", requiring no access to teacher model logits.
- **Logit Distillation**: When teacher model APIs return logprobs, student models can be directly trained on the teacher's token probability distribution, leading to more thorough knowledge transfer.
- **Chain-of-Thought (CoT) Distillation**: Prompting teacher models to generate Chain-of-Thought data with explicit reasoning steps; student models gain complex reasoning capabilities by learning these step-by-step processes (e.g., DeepSeek-R1 distilled into Qwen/LLaMA series).
- **Domain-Specific Distillation**: Generating specialized data from large models in specific domains (such as healthcare, law, code) to train domain-specialized small models, balancing deployment efficiency and domain performance.

Knowledge distillation is one of the core drivers behind the rapid capability improvements of small models in the current LLM ecosystem, enabling 7B--14B parameter models to approach or even surpass the performance of earlier, larger-scale models.

</details>

**11. How Does Model Pruning Improve LLM Efficiency Without Sacrificing Performance?**

<details>
<summary>💡 Show Answer</summary>

Model Pruning improves efficiency by identifying and removing redundant or unimportant parameters in a neural network, reducing model size and computational complexity while preserving the model's original performance as much as possible.

**Main Types of Pruning**:

- **Unstructured Pruning**: Directly zeros out weights with small absolute values in weight matrices, forming sparse matrices. It offers high flexibility and minimal accuracy loss, but requires dedicated sparse compute kernels to achieve hardware acceleration.
- **Structured Pruning**: Removes entire neurons, attention heads, or Transformer layers to generate more regular, smaller dense models that achieve direct speedup on standard hardware. Examples include:
    - **Attention Head Pruning**: Removes attention heads that contribute minimally to the final output; studies show significant head redundancy in LLMs.
    - **Layer Pruning**: Removes entire Transformer layers, typically pruned from middle or shallow layers with minimal impact on performance.
    - **FFN Neuron Pruning**: Compresses the intermediate dimension of FFN layers to reduce the parameter count of feed-forward networks.
- **Weight Pruning**: Prunes specific rows or columns of weight matrices, bridging unstructured and structured pruning.

**Key Techniques for Minimizing Performance Loss**:

- **Importance Scoring**: Estimates parameter importance using gradient information, activation statistics, or Hessian matrices, prioritizing the retention of important parameters.
- **Pruning + Fine-tuning**: Fine-tunes remaining parameters with a small amount of data after pruning to restore performance lost during pruning—a critical step for maintaining precision.
- **Iterative Pruning**: Gradually prunes in small steps alternating with fine-tuning, rather than performing a single large prune, to better preserve accuracy.
- **LLM-Specific Methods (e.g., LLM-Pruner)**: Pruning strategies tailored to LLM dependency structures (such as multi-head attention and inter-layer residual connections) that achieve efficient compression while preserving structural integrity.

In practice, pruning and quantization are often combined (pruning followed by quantization) to achieve even higher compression ratios while keeping accuracy loss within acceptable bounds.

</details>

**12. Why Distill DeepSeek Instead of Using DeepSeek Directly?**

<details>
<summary>💡 Show Answer</summary>

![DeepSeek reasoning trajectory distillation into lightweight student models.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig41_distillation_deepseek_student.png)




Directly using the original DeepSeek large model is indeed feasible in many scenarios, but the following core issues motivate us to choose distillation:

1. **Cost Issues**: Flagship models like DeepSeek-R1 have parameter counts up to 671B (MoE architecture). API call costs cannot be ignored, and limited by network latency and provider quotas, they cannot support high-concurrency real-time scenarios. Distilled small models ranging from 1.5B to 14B can run locally on consumer-grade GPUs (or even CPUs), reducing marginal inference costs to near zero, making them suitable for high-frequency conversational business scenarios.

2. **Data Privacy and Security**: In enterprise internal use cases (such as private codebase analysis, internal document Q&A, and customer data processing), sending sensitive data to external APIs poses compliance risks (GDPR, industry regulations, etc.). Distilled small models deployed on an enterprise intranet or private cloud ensure data never leaves the domain, fundamentally eliminating leakage risks.

3. **Domain Specialization and Customization**: General-purpose large models lack knowledge of enterprise-private domain information (such as specific database schemas, internal terminology, and business rules). By generating distillation data on proprietary enterprise data and fine-tuning, small models can master this domain-specific knowledge, potentially outperforming general-purpose large models on specific tasks.

4. **Controllability and Optimization**: Owning model weights grants complete technical autonomy, enabling:
    - **Continued Fine-Tuning (SFT/LoRA)**: Continuously iterate for new tasks or new data.
    - **Reinforcement Learning Alignment (RLHF/GRPO)**: Optimize model behavior based on business feedback.
    - **Structured Decoding**: Force model outputs to conform to specific formats (such as JSON Schema), ensuring downstream system stability.
    - **Inference Acceleration and Optimization**: Combine techniques such as quantization and speculative decoding to further boost inference efficiency.

5. **Service Stability**: Relying on external APIs carries risks of service outages, rate limiting, price changes, etc. Self-hosted distilled models ensure high availability and SLA for services.

Summary: Using the DeepSeek API directly is suitable for rapid prototyping and low-frequency call scenarios. However, for production systems requiring high concurrency, strong privacy protection, deep customization, and long-term operations, distilling and self-hosting a small model is a more cost-effective and sustainable choice.

</details>

