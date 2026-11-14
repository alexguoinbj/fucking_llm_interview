# 3.4 Pre-training Engineering — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 47 questions

**1. What Problems Exist in Training Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

Training Large Language Models (LLMs) is a highly challenging engineering and scientific task that involves complex problems across multiple dimensions, including computation, data, algorithms, and engineering. The following sections provide a detailed elaboration from multiple perspectives.

#### 1. Prohibitive Computational Resource Requirements

Training modern large language models requires thousands of high-performance accelerator chips (GPUs or TPUs) running continuously for weeks or even months. Taking GPT-3 (175B parameters) as an example, its training is estimated to consume approximately $3.14 \times 10^{23}$ floating-point operations (FLOPs), translating to compute costs of millions of dollars. This brings the following specific issues:

- **Extremely High Hardware Threshold:** A single high-end GPU (such as the NVIDIA A100/H100) typically has 80GB of VRAM, whereas a 100-billion-parameter model requires over 200GB of VRAM just to store weights (in FP16 precision). Consequently, model parallelism, tensor parallelism, and other techniques must be used to distribute storage across multiple cards.
- **Huge Energy Consumption:** Training GPT-3 is estimated to produce about 552 metric tons of $\mathrm{CO_2}$ equivalent, imposing a significant environmental burden and sparking widespread academic discussion on "Green AI."
- **Unaffordable Financial Costs:** For most academic institutions and small-to-medium enterprises, independently training frontier LLMs is virtually infeasible, intensifying the centralization trend of AI capabilities.

#### 2. Data Storage, Cleaning, and Processing

The scale of training corpora for LLMs typically reaches hundreds of billions or even trillions of tokens, with raw data sizes routinely measured in terabytes (TB) or petabytes (PB), giving rise to a series of data engineering challenges:

- **Data Storage and I/O Bottlenecks:** Massive volumes of data require high-speed distributed storage systems (e.g., HDFS, object storage) for support. If data loading speeds during training cannot keep up with compute speeds, GPUs will idle, reducing overall efficiency.
- **Data Quality and Deduplication:** Raw web-scraped corpora contain substantial noise, duplicated content, low-quality text, and even harmful content, requiring significant human labor and compute power for filtering, deduplication (such as MinHash LSH algorithms), and quality assessment.
- **Data Mixture and Distribution:** The proportion in which data from different sources, languages, and domains should be mixed significantly impacts final model performance. However, mature theoretical guidance is currently lacking, often relying on extensive ablation experiments.
- **Data Copyright and Privacy Risks:** Training data may contain copyrighted content or user privacy information, introducing legal compliance risks and potentially leading to models "memorizing" and leaking sensitive data.

#### 3. Slow Model Convergence and Training Instability

The training process of ultra-large-scale models is extremely prone to instability. Common issues include:

- **Loss Spikes:** During training, the loss value may suddenly spike drastically, causing model performance to degrade severely. This requires restarting training from an earlier checkpoint, resulting in significant waste of compute resources.
- **Vanishing and Exploding Gradients:** In extremely deep Transformer networks, backpropagated gradients may approach zero (vanish) or explode exponentially (explode), preventing normal training. Common mitigation techniques include gradient clipping, weight initialization strategies (such as $\mu$P), and residual connections.
- **Complex Learning Rate Scheduling:** Learning rate warmup and decay strategies (such as cosine annealing) must be meticulously designed, and optimal hyperparameters do not easily transfer across models of different scales.
- **Slow Convergence Speed:** Even with abundant hardware resources, training a model with hundreds of billions of parameters to convergence consumes significant wall-clock time, limiting the efficiency of iterative experimentation.

#### 4. Communication Overhead and Synchronization Issues in Distributed Training

To address insufficient single-card VRAM, LLM training typically adopts multiple parallelism strategies, which introduce additional engineering complexity:

- **Data Parallelism:** Each card holds a complete copy of the model and processes different data batches. At the end of each iteration, gradients must be synchronized via AllReduce operations. When the card count is extremely high, communication bandwidth becomes a bottleneck.
- **Model Parallelism:** Distributing different layers of the model across different devices requires frequent transmission of activations between devices. Although Pipeline Parallelism can alleviate part of the problem, it introduces "bubble" idle time.
- **Tensor Parallelism:** Splitting single-layer weight matrices across multiple cards demands extremely high inter-device communication bandwidth (typically requiring high-speed interconnects such as NVLink) and places strict requirements on cluster network topology.
- **Scheduling Complexity of Hybrid Parallelism:** Practical training often simultaneously employs data parallelism, pipeline parallelism, and tensor parallelism (3D Parallelism). The complexity of scheduling and load balancing is extremely high, requiring specialized frameworks (such as Megatron-LM and DeepSpeed) for support.

#### 5. Difficulties in Model Optimization and Debugging

- **Extremely High Cost of Hyperparameter Tuning:** Hyperparameters such as learning rate, batch size, weight decay, and dropout rate have a significant impact on model performance. However, performing grid search at ultra-large scales is virtually impossible, typically relying on small-scale proxy experiments or empirical heuristics.
- **Poor Interpretability:** Anomalous behaviors occurring during training (such as sudden performance drops on specific tasks) are difficult to trace back to root causes, as internal model representations are virtually opaque to humans.
- **Lagging Evaluation:** During training, monitoring is usually restricted to proxy metrics such as perplexity, while actual downstream task performance (e.g., reasoning, code generation) can only be evaluated after training completes, leading to severe lag in issue detection.
- **Catastrophic Forgetting:** During fine-tuning, the model may forget general capabilities learned during the pre-training phase when learning new tasks, requiring mitigation through regularization or data mixing strategies.

#### 6. Memory Efficiency and VRAM Optimization

- **VRAM Footprint of Activations:** Intermediate activation values generated during forward propagation must be retained for backpropagation, and their VRAM usage is often several times that of the model parameters themselves. Gradient Checkpointing trades recomputation for memory, but increases computational overhead by approximately 33%.
- **Numerical Stability in Mixed-Precision Training:** Training with FP16 or BF16 significantly reduces VRAM footprint and increases compute speed, but issues such as overflow and precision loss resulting from reduced dynamic range require careful handling through techniques like Loss Scaling.
- **Optimizer State Overhead:** Adaptive optimizers such as Adam must maintain first-order and second-order moment estimates for each parameter. This VRAM overhead is comparable to the parameter volume itself, further compounding VRAM pressure. Techniques like ZeRO (Zero Redundancy Optimizer) mitigate this issue by partitioning optimizer states.

#### 7. Alignment and Safety Issues

Even if a model successfully completes pre-training, its behavior does not necessarily align with human intent:

- **Harmful Content Generation:** Models may learn biases, discriminatory speech, or harmful knowledge from training data, reproducing or even amplifying them during inference.
- **Insufficient Instruction Following:** Pre-trained models are generally "completion engines" that cannot be directly used for dialogue or instruction execution. They require alignment through Supervised Fine-Tuning (SFT) and Reinforcement Learning from Human Feedback (RLHF), phases that themselves suffer from high data costs and reward model bias.
- **Hallucination Issues:** Models tend to generate factually incorrect content in a fluent and confident tone; how to fundamentally suppress hallucinations during the training phase remains an open challenge.

In summary, training large language models is a comprehensive challenge spanning algorithms, systems, data engineering, and safety ethics. A flaw in any single component can lead to training failure or model quality falling short of expectations. This is precisely why frontier LLM training currently remains concentrated in the hands of a few institutions with top-tier engineering capabilities and deep financial resources.

</details>

**2. What Causes Loss Spikes During Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

![LLM pre-training loss spike anomaly and checkpoint recovery workflow.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig23_loss_spike_mitigation.png)




In a single sentence, the cause of a loss spike is: because the Adam optimizer is used, although optimization efficiency is fast, in later training stages, the large shift in data distribution between different batches causes shallow-layer gradient changes to become severe and turn into a bimodal non-steady state, destroying the steady state established in earlier training and resulting in a sudden surge in loss.

A loss spike refers to a phenomenon during pre-training, especially in ultra-large models (parameter count exceeding 100B), where the loss value suddenly skyrockets. A loss spike triggers a series of issues; for instance, the model may require a long time to recover to the pre-spike state, or in more severe cases, the loss fails to decrease again, meaning the model can no longer converge.

Specific process:
1. During healthy training, the model exhibits a unimodal distribution close to a normal distribution, with low correlation between gradient estimates.
2. As training continues, gradient changes in shallow layers (e.g., embedding layer) become less noticeable, while gradient changes in deep layers remain larger.
3. Shallow-layer gradients collapse: with gradient $g \ll \epsilon$, it leads to $m \ll \epsilon, v \ll \epsilon$, meaning shallow layers essentially cease updating.
4. As the model continues learning and deep networks keep optimizing and updating, the probability increases that a batch will require “reconsidering” feature maps learned in early layers of the model, causing gradients to become a bimodal non-steady state.

</details>

**3. How to Handle Loss Spikes Encountered During Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

1. **Replacing the training batch**: As mentioned in PaLM and GLM-130B, once a loss spike occurs, resume from a checkpoint approximately 100 steps before the spike began, skipping about 200--500 data batches.
2. **Reducing the learning rate**: Reduce the learning rate when gradients are large; however, this does not fundamentally help the stability of model training.
3. **Reducing the size of $\epsilon$**.
4. **Adjusting the optimizer when distribution shifts occur**.

</details>

**4. What Are Common Sources of NaN in Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

- **Division by zero or logarithm of zero**: Extremely large softmax inputs cause denominator overflow; $\log(0)$ occurs in cross-entropy calculation;
- **FP16 overflow**: Activation values exceed the FP16 maximum and become `Inf`, producing NaN in subsequent operations;
- **Exploding gradients**: Gradient values exceed the floating-point representation range;
- **Corrupted data**: Training corpus contains samples with NaN values or extreme token IDs;
- **Unstable initialization**: Excessively large variance in weight initialization leads to overflow in the early stages of forward propagation.

**Troubleshooting workflow:** Register forward/backward hooks to print statistics layer by layer (mean, variance, maximum value), locate the first layer where NaN appears, and apply targeted fixes.

</details>

**5. What Is the Role of Learning Rate Warmup in Large Model Training?**

<details>
<summary>💡 Show Answer</summary>

In the early stage of training, model parameters are randomly initialized and gradient variance is extremely large. Directly applying the target learning rate can easily lead to severe parameter oscillations or even loss spikes. The Warmup strategy linearly increases the learning rate from $0$ (or an extremely small value) to the target value $\eta_{\max}$ during the first $T_w$ steps:


$$
	\eta_t = \eta_{\max} \cdot \frac{t}{T_w}, \quad t \leq T_w
$$


Afterward, it is usually combined with cosine annealing or polynomial decay:


$$
	\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\frac{\pi (t - T_w)}{T - T_w}\right)
$$


Typical warmup steps account for $1%\sim2%$ of total training steps (e.g., GPT-3 uses a 375M token warmup).

</details>

**6. How Do Z-loss and Auxiliary Loss Improve Training Stability?**

<details>
<summary>💡 Show Answer</summary>

**Z-loss** (introduced by works like PaLM) penalizes the magnitude of logits input to the softmax to prevent numerical instability caused by excessively large logits:


$$
	\mathcal{L}_z = \alpha \cdot \frac{1}{B} \sum_{i=1}^{B} \left(\log \sum_j e^{z_{ij}}\right)^2
$$


where $z_{ij}$ is the logit vector of the $i$-th sample, and $\alpha$ is a hyperparameter (typical value $10^{-4}$).

**MoE Auxiliary Loss**: In Mixture-of-Experts (MoE) models, to prevent routing collapse (where all tokens rush into a few experts), a load balancing auxiliary loss is introduced:


$$
	\mathcal{L}_{aux} = \alpha \cdot E \sum_{i=1}^{E} f_i \cdot P_i
$$


where $f_i$ is the fraction of tokens actually processed by expert $i$, $P_i$ is the average probability assigned to expert $i$ by the router, and $E$ is the number of experts.

</details>

**7. Briefly Explain the Principle of Mixed-Precision Training?**

<details>
<summary>💡 Show Answer</summary>

The core idea of Mixed Precision Training is to simultaneously use low precision (FP16 or BF16) and high precision (FP32) during the training process, thereby leveraging the speed and VRAM advantages of low-precision operations while ensuring numerical stability.

#### Why Is Mixed Precision Needed?

- **Issues with FP32:** Although single-precision floating point (32-bit) is numerically stable, each parameter consumes 4 bytes, creating immense VRAM pressure during large model training. Furthermore, GPU FP32 compute throughput is lower than FP16.
- **Issues with Pure FP16:** Half-precision floating point (16-bit) has a narrow dynamic range (maximum around 65504, minimum normalized number around $6\times10^{-5}$). Directly using it for gradient updates easily leads to numerical overflow or precision loss, causing non-convergence.
- **The Compromise of Mixed Precision:** Using FP16 in forward/backward propagation accelerates computation, while using FP32 during parameter updates guarantees precision, balancing efficiency and stability.

#### Core Mechanism

Mixed precision training refers to using FP16 to store weights, activations, and compute gradients, while maintaining an FP32 weight backup (Master Weight), and using FP32 when performing gradient updates.

The process is as follows:
1. Convert neural network weight parameters from initialized FP32 to FP16;
2. Perform forward and backward computation with FP16, and compute gradients;
3. Convert FP16 gradients to FP32;
4. Multiply FP32 gradients by the learning rate;
5. Update network weights using FP32, yielding updated FP32 weights.

#### Regarding BF16

In practical engineering, BF16 (Brain Float 16) is gradually replacing FP16. BF16 shares the same exponent bit width (8 bits) as FP32, giving it an identical dynamic range to FP32. Overflow issues are drastically reduced, eliminating the need for loss scaling. New-generation GPUs such as A100/H100 as well as TPUs offer native hardware support for BF16, making it the mainstream precision choice for current large model training.

In models, if FP16 is used instead of FP32, values exceeding this dynamic range may occur, causing overflow or underflow. In mixed-precision training, “loss scaling” technology is employed to prevent precision loss caused by excessively small numerical values.

% -------------------------------------------------------

</details>

**8. Basic Workflow of Mixed-Precision Training**

<details>
<summary>💡 Show Answer</summary>

1. **Initialize Model and Optimizer**: Define the neural network model and initialize its weights in FP32 format. Meanwhile, maintain FP32 copies of first-order/second-order momentum for the optimizer (e.g., Adam).

2. **Convert Weights to FP16**: Copy and convert the FP32 master weights to FP16 for subsequent calculations. Execute this step at the beginning of each iteration.

3. **Forward Pass (FP16)**: Perform forward pass computations using weights in FP16 format, storing intermediate activation values in FP16 as well, thereby saving VRAM and increasing compute throughput. The theoretical throughput of Tensor Core units under FP16 is $8\times$ that of FP32 (approx. 312 TFLOPS vs. 19.5 TFLOPS on A100).

4. **Loss Computation (FP32)**: When calculating the loss function, final Logits or Loss are typically accumulated in FP32 format to ensure numerical stability and avoid precision loss caused by large-scale summation.

5. **Loss Scaling**: Before backpropagation, multiply the FP32 loss value by a scaling factor $s$ (e.g., $s=2^{15}$) so that gradient values fall within the representable range of FP16, preventing gradient underflow.

6. **Backward Pass (FP16)**: Compute gradients using FP16 format. At this point, the gradients have been scaled to larger values and are less likely to underflow.

7. **Gradient Unscaling and Check**: Divide the FP16 gradients by the scaling factor $s$ to unscale them back to true gradients and convert them to FP32. Simultaneously check whether `Inf` or `NaN` exists in the gradients—if present, skip parameter updates for this step and decrease the scaling factor; if no anomalies occur for several consecutive steps, appropriately increase the scaling factor (i.e., dynamic loss scaling strategy).

8. **Update Weights (FP32)**: Use the unscaled FP32 gradients to update the FP32 master weights via the optimizer. Adam's momentum states are also maintained in FP32 throughout the process to guarantee numerical precision.

9. **Repeat Training Process**: Repeat the above steps until the model converges.

#### VRAM Analysis

Taking a model with parameter count $\Phi$ as an example, the VRAM consumption of mixed-precision training is approximately:
- FP16 parameters: $2\Phi$ bytes
- FP32 master weight backup: $4\Phi$ bytes
- FP32 gradients: $4\Phi$ bytes
- FP32 optimizer states (Adam): $8\Phi$ bytes (first-order $4\Phi$ + second-order $4\Phi$)
- **Total**: approx. $18\Phi$ bytes (compared to approx. $16\Phi$ bytes for pure FP32)

It can be seen that mixed precision does not save VRAM compared to pure FP32; its main benefit lies in storing intermediate activations as FP16 during forward/backward passes, thereby significantly reducing activation VRAM footprint (especially effective with long sequences and large batch sizes) while fully leveraging Tensor Core compute acceleration.

% -------------------------------------------------------

</details>

**9. What are the Advantages of Mixed-Precision Training?**

<details>
<summary>💡 Show Answer</summary>

- **Reduce Activation VRAM Footprint**: Storing intermediate activations in FP16 during forward pass can reduce activation VRAM by approximately $50%$ in long-sequence and large-batch training scenarios, enabling larger models to be trained on limited hardware or allowing larger batch sizes to improve training efficiency.

- **Significantly Accelerate Training Process**: Modern GPUs (such as A100, H100) and TPUs are equipped with dedicated Tensor Core units that provide hardware acceleration for FP16/BF16 matrix multiplication, yielding theoretical compute throughput several to over ten times that of FP32, which significantly shortens per-step training time.

- **Maintain Model Accuracy via Dynamic Loss Scaling**: Dynamic loss scaling automatically adjusts the scale factor, preventing gradient underflow while avoiding gradient overflow, so that the final accuracy of the model is comparable to pure FP32 training.

- **Good Compatibility and Easy Adoption**: Mainstream deep learning frameworks (PyTorch, TensorFlow, JAX) natively support mixed-precision training. PyTorch provides the `torch.cuda.amp` module, enabling one-click activation via just two APIs (`GradScaler` and `autocast`), with minimal intrusiveness to training code.

- **Orthogonal to Other Optimization Techniques**: Mixed precision can be combined with techniques such as ZeRO, gradient accumulation, and activation recomputation to further enhance training efficiency.

% -------------------------------------------------------

</details>

**10. What are the Disadvantages of Mixed-Precision Training?**

<details>
<summary>💡 Show Answer</summary>

- **Higher Implementation Complexity**: Compared to pure FP32 training, mixed precision requires maintaining extra FP32 master weight copies, implementing loss scaling logic, and handling gradient checking steps, increasing debugging difficulty. Although framework abstractions lower the barrier, manual specification of which operations use FP16 versus FP32 is still required in custom operators or specialized network architectures.

- **Requires Manual Intervention for Loss Scaling Parameters in Certain Cases**: Dynamic loss scaling works automatically in most scenarios, but for models with highly unstable gradient distributions (such as certain GANs or reinforcement learning policy networks), manually setting initial scaling factors or tuning growth/decrease strategies may be required.

- **Numerical Stability Risks**: Certain operations (such as Softmax, variance computation in LayerNorm and Batch Normalization) are prone to numerical instability under FP16. These operations must be explicitly marked for FP32 execution; otherwise, training divergence may occur.

- **Overflow Risk Specific to FP16**: The maximum representable number in FP16 is approximately 65504. Overflow easily occurs when gradient values are large (e.g., early in training or with excessively high learning rates), generating `Inf` or `NaN` and requiring additional overflow detection mechanisms. BF16 mitigates this issue to a certain extent.

- **Hardware Dependency**: Tensor Core acceleration requires matrix dimensions to satisfy specific alignment requirements (e.g., A100 requires FP16 matrix dimensions to be multiples of 16). If network structures fail to meet these conditions, the actual speedup ratio may fall far short of theoretical values. In addition, some older GPU models (e.g., GTX 10 series) have limited FP16 support, resulting in poor acceleration performance.

- **Debugging Difficulties**: Due to precision loss, numerical issues in mixed-precision training (such as gradient vanishing and NaN propagation) are often harder to locate and reproduce than in FP32 training.

% -------------------------------------------------------

</details>

**11. What are the Causes of Overflow Issues in Mixed-Precision Training?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Loss Scaling workflow in FP16 mixed-precision training.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig24_fp16_mixed_precision_overflow.png)




1. **Numerical Range Limitations**: The numerical range of FP16 is much smaller than that of FP32, making it far more susceptible to overflow and underflow. Specifically, the maximum positive number for FP16 is approximately $6.55\times10^4$, and the minimum normalized number is approximately $6.10\times10^{-5}$; whereas the maximum positive number for FP32 is approximately $3.4\times10^{38}$, and the minimum normalized number is approximately $1.18\times10^{-38}$. The difference in dynamic range reaches tens of orders of magnitude, making FP16 hit boundaries very easily during training.

2. **Activation Function and Gradient Issues**: Certain activation functions (such as `exp`, `softmax`) generate extremely large or small intermediate values during calculation, easily causing FP16 numerical overflow; gradients in late-stage training are typically small (especially in deep networks) and tend to underflow to zero, causing parameters to fail to update effectively—meaning the “gradient vanishing” problem is even more severe under FP16.

3. **Gradient Scaling Issues**: When the loss scale factor is too large, scaled gradient values may exceed the upper limit of FP16 (approx. 65504), producing `Inf`, which then propagates as `NaN` in backpropagation and corrupts the entire gradient. Conversely, a scale factor that is too small cannot effectively prevent underflow.

4. **Accumulation Errors**: Large-scale addition accumulation (such as mean/variance computation in LayerNorm, normalization denominators in Softmax) leads to rounding errors in FP16 that amplify as the number of addends increases, causing calculation results to deviate severely from FP32 reference values.

5. **Incomplete Hardware and Software Support**: FP16 compute units in some hardware (e.g., early GPUs) do not support `subnormal` (subnormal number) handling and directly flush tiny numbers to zero, exacerbating underflow issues; custom operators in some frameworks have not undergone FP16 adaptation, which may also introduce numerical problems.

6. **Model and Data Type Conversions**: When converting data types between FP32 and FP16, FP32 values exceeding the FP16 range are clipped to `Inf`, which requires particular attention when converting large weight matrices.

% -------------------------------------------------------

</details>

**12. How to Solve Overflow Issues?**

<details>
<summary>💡 Show Answer</summary>

1. **Loss Scaling**: Before backpropagation, multiply the loss value by a large scaling factor $s$ so that the scaled gradient values fall within the representable range of FP16; after backpropagation is completed, divide by $s$ to restore the true gradients. **Dynamic Loss Scaling** strategy:
    - If `Inf`/`NaN` exists in the current step's gradients, skip parameter updates and multiply $s$ by a reduction factor (e.g., $s \leftarrow s / 2$);
    - If no overflow occurs for $N$ (e.g., 2000) consecutive steps, multiply $s$ by a growth factor (e.g., $s \leftarrow s \times 2$) to maintain as large a scaling factor as possible.

2. **Use FP32 Weight Copies**: Maintain an FP32 master weight copy for gradient accumulation and parameter updates, while using FP16 weights only for forward/backward computations, fundamentally guaranteeing the precision of parameter updates.

3. **Explicitly Use FP32 for Critical Operations**: Explicitly perform calculations in FP32 for operations prone to overflow. Common operations requiring FP32 include:
    - Softmax and its cross-entropy loss
    - Variance computation in LayerNorm and BatchNorm
    - Accumulation of large-scale vector dot products
    In PyTorch `autocast`, the framework automatically keeps these operations in FP32, requiring no manual user intervention.

4. **Switch to BF16**: BF16 shares the same exponent width (8 bits) as FP32, giving it an identical dynamic range to FP32 (up to approx. $3.4\times10^{38}$), fundamentally eliminating FP16 overflow risks. It is currently the preferred precision format for training large models. On hardware supporting BF16 (A100, H100, TPU), FP16 can be directly replaced with BF16 without requiring loss scaling.

5. **Gradient Clipping**: Clip the L2 norm of gradients before parameter updates (e.g., `max_norm=1.0`) to prevent overflow caused by gradient explosion, while also contributing to training stability.

6. **Hardware and Software Optimization**: Ensure the use of hardware supporting FP16/BF16 acceleration (A100, H100, RTX 4090, etc.), and use mature deep learning frameworks (PyTorch `amp`, TensorFlow `tf.keras.mixed_precision`) to obtain fully tested, numerically stable implementations.

% -------------------------------------------------------

</details>

**13. What is the Workflow of Loss Scaling?**

<details>
<summary>💡 Show Answer</summary>

Loss Scaling is a core mechanism for ensuring numerical stability of gradients during mixed-precision training. The fundamental reason is that the underflow threshold of FP16 is around $6\times10^{-5}$, whereas gradient values in backpropagation of deep networks are often far smaller than this. Directly computing in FP16 causes gradients to be flushed to zero, preventing parameters from updating.

#### Static Loss Scaling

Uses a fixed scaling factor $s$ (e.g., $s=2^{15}=32768$) that remains constant throughout training. The drawback is that manually selecting an appropriate $s$ is required—excessively large values easily cause overflow, while excessively small values provide poor anti-underflow protection.

#### Dynamic Loss Scaling (Mainstream Approach)

The workflow is as follows:
1. **Compute Loss Value**: Compute the loss value $\mathcal{L}$ in FP32 during the forward pass.
2. **Scale Loss Value**: Multiply the loss value by the scale factor $s$ to obtain the scaled loss $\hat{\mathcal{L}} = s \cdot \mathcal{L}$, ensuring it falls within the representable range of FP16.
3. **Backward Pass**: Backpropagate the scaled loss $\hat{\mathcal{L}}$ to compute scaled gradients $\hat{g} = s \cdot g$ in FP16 format (by the chain rule, loss scaling is equivalent to gradient scaling).
4. **Overflow Detection**: Check whether `Inf` or `NaN` exists in the gradients of all parameters. If present:
    - Skip parameter updates for this step (do not call `optimizer.step()`);
    - Halve the scale factor: $s \leftarrow s / 2$;
    - Proceed to the next training step.
5. **Unscale Weight Gradients**: If no overflow occurs, divide the scaled gradients by $s$ before gradient clipping or other gradient-related calculations to restore true gradients: $g = \hat{g} / s$, and convert them to FP32.
6. **Gradient Clipping** (Optional): Perform L2 norm clipping on the restored FP32 gradients.
7. **Parameter Update**: Update the FP32 master weights using the FP32 gradients.
8. **Dynamically Adjust Scale Factor**: If no overflow occurs for $N$ consecutive steps (e.g., 2000 steps), double the scale factor: $s \leftarrow s \times 2$, maintaining as large a scale factor as possible to maximize anti-underflow performance.

#### PyTorch Implementation Illustration

\begin{lstlisting}[style=promptstyle]
	scaler = torch.cuda.amp.GradScaler()  # Automatically manage dynamic scale factor

	for data, target in dataloader:
	optimizer.zero_grad()
	with torch.cuda.amp.autocast():   # Automatically cast operations to FP16/BF16
	output = model(data)
	loss = criterion(output, target)

	scaler.scale(loss).backward()     # Scale loss and backpropagate
	scaler.unscale_(optimizer)        # Unscale gradients (for gradient clipping)
	torch.nn.utils.clip_grad_norm_(model.parameters(), max_norm=1.0)
	scaler.step(optimizer)            # Check for overflow; update parameters if none
	scaler.update()                   # Dynamically adjust scale factor
\end{lstlisting}

% -------------------------------------------------------

</details>

**14. What is the Dynamic Range of Floating-Point Numbers?**

<details>
<summary>💡 Show Answer</summary>

The dynamic range of a floating-point number refers to the range of numerical values it can represent, including very small numbers (close to zero) and very large numbers. The IEEE 754 standard defines common floating-point formats, whose core components are as follows:

#### Structure of Floating-Point Numbers

A floating-point number consists of three parts: Sign, Exponent, and Mantissa/Fraction. Its value is:
\[
(-1)^{S} \times 2^{E - bias} \times (1 + M)
\]

Comparison among precision formats is shown below:

\begin{center}


| **Format** | **Total Bits** | **Sign Bits** | **Exponent Bits** | **Mantissa Bits** |
| --- | --- | --- | --- | --- |
| FP64 (Double Precision) | 64 | 1 | 11 | 52 |
| FP32 (Single Precision) | 32 | 1 | 8 | 23 |
| BF16 | 16 | 1 | 8 | 7 |
| FP16 (Half Precision) | 16 | 1 | 5 | 10 |


\end{center}

#### Dynamic Range vs. Precision Comparison

- **FP32**: Maximum positive number approx. $3.4\times10^{38}$, minimum normalized number approx. $1.18\times10^{-38}$, effective decimal digits approx. 7.
- **FP16**: Maximum positive number approx. $6.55\times10^{4}$ (65504), minimum normalized number approx. $6.10\times10^{-5}$, effective decimal digits approx. 3. The dynamic range is far smaller than that of FP32, which is the root cause of overflow issues in mixed-precision training.
- **BF16**: Exponent width is identical to FP32 (8 bits), so its dynamic range is consistent with FP32; however, it has only 7 mantissa bits, giving it precision (approx. 2$\sim$3 effective decimal digits) lower than FP32 but higher than FP16.

#### Overflow and Underflow

- **Overflow**: When a value exceeds the maximum representable number of a floating-point format, the result is represented as $\pm\infty$ (`Inf`). In subsequent computations, `Inf` propagates as `NaN`, corrupting gradients.
- **Underflow**: When the absolute value of a number is too small—smaller than the minimum normalized number—it is represented as a subnormal number or directly flushed to zero. Gradient underflow to zero causes parameters to fail to update.

% -------------------------------------------------------

</details>

**15. Why Does Deep Learning Typically Default to Single-Precision FP32 for Training Instead of Double-Precision FP64?**

<details>
<summary>💡 Show Answer</summary>

The main reason is the huge gap between the actual numerical precision requirements of deep learning and the theoretical precision of FP64, while the cost of FP64 is far higher than FP32:

- **Precision Demand Matching**: For deep learning, FP32 precision (approx. 7 effective decimal digits) is already sufficient. Neural network training is essentially a stochastic optimization process (SGD/Adam), where noise itself is an integral component of training. Extremely high numerical precision not only fails to improve final model quality, but may also cause the model to overfit to numerical noise in training batches. In contrast, scientific computing requiring ultra-high precision (e.g., finite element analysis, weather forecasting) or financial modeling (e.g., high-precision interest rate modeling) truly requires FP64.

- **VRAM and Bandwidth Overhead**: FP64 occupies 8 bytes per parameter, twice that of FP32. For the same VRAM capacity, trainable model size is halved, and VRAM bandwidth utilization is reduced.

- **Vastly Different Compute Throughput**: FP32 throughput on modern GPUs is far higher than FP64. Taking NVIDIA A100 as an example, FP32 peak performance is approx. 19.5 TFLOPS, while FP64 is only approx. 9.7 TFLOPS (non-Tensor Core), a gap of approx. $2\times$; for consumer GPUs (such as RTX 4090), FP64 performance is even as low as $\frac{1}{64}$ of FP32, making it completely unsuitable as the default training precision.

- **Hardware Ecosystem Optimization Focus**: GPU vendors (NVIDIA, AMD) and AI accelerators (TPU, Ascend) all prioritize FP32/FP16/BF16 as their primary optimization targets for deep learning. Tensor Cores do not support FP64 matrix acceleration, and the ecosystem defaults to FP32.

% -------------------------------------------------------

</details>

**16. Why is FP16 Used for Multiplication while FP32 is Used for Addition Accumulation?**

<details>
<summary>💡 Show Answer</summary>

This design principle stems from the differences in the **error propagation patterns** of floating-point numbers:

#### Characteristics of Multiplication Error

The relative error of floating-point multiplication is bounded, approximately equal to the machine epsilon $\epsilon_{mach}$ (around $9.8\times10^{-4}$ for FP16). The error introduced by each multiplication is a relative error and does not accumulate linearly with the number of multiplications (because the error of subsequent multiplications is a relative error applied to the new result). Therefore, the numerical error of individual multiplication operations for each element in matrix multiplication remains within an acceptable range.

#### Characteristics of Addition Accumulation Error

In addition accumulation (such as vector inner product $\sum_{i=1}^{N} a_i b_i$), the absolute error of each step is carried over into the next addition step, forming a **linear accumulation** of error:

\[
Accumulated Error \approx N \cdot \epsilon_{mach} \cdot \max|a_i b_i|
\]

When $N$ is very large (such as the inner product with $d_{model} = 4096$ in Transformers), the accumulated error of FP16 may reach the order of magnitude of the result itself, severely affecting numerical stability. Accumulating in FP32 limits the error to the magnitude of $\epsilon_{mach}^{FP32} \approx 1.2\times10^{-7}$, which is far lower than FP16's $\epsilon_{mach}^{FP16} \approx 9.8\times10^{-4}$.

#### Engineering Implementation

Tensor Cores in modern GPUs are designed based on this exact principle: in general matrix multiplication (GEMM) operations, multiplications are executed in FP16, but the accumulation registers (Accumulators) are FP32, referred to as "FP16 multiply, FP32 accumulate," which maintains numerical precision while guaranteeing computation speed.

% -------------------------------------------------------

</details>

**17. Can All Operations Be Computed Using FP16?**

<details>
<summary>💡 Show Answer</summary>

![Bit layout and dynamic range comparison across FP32, FP16, and BF16 formats.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig25_fp16_vs_bf16_range.png)




Not all operations are suitable for FP16 computation. Based on their numerical characteristics, operations can be categorized into the following groups:

1. **Operations Safe to Use FP16**:
    - Matrix Multiplication (GEMM): Multiplication is executed in FP16 while accumulation uses FP32 (default Tensor Core behavior). This is the primary source of speedup for FP16 acceleration and the most computationally intensive operation in Transformers.
    - Element-wise Operations: Such as `ReLU`, `GELU`, addition, etc., where input and output ranges are limited, and FP16 precision is usually sufficient.
    - Convolution Operations: Similar to matrix multiplication, Tensor Cores support FP16 convolution acceleration with controllable numerical risks.

2. **Operations Requiring FP32**:
    - **Vector Dot Products and Large-Scale Sums**: As mentioned earlier, accumulation should be performed in FP32 to ensure numerical accuracy (Tensor Cores implement FP32 accumulation at the hardware level).
    - **Softmax**: Involves the `exp` operation; if input values are large (e.g., attention scores early in training), it easily causes FP16 overflow. Summation and accumulation in the denominator also require FP32. PyTorch `autocast` keeps Softmax in FP32 by default.
    - **LayerNorm / BatchNorm**: Mean and variance calculations involve large-scale accumulation where FP16 precision is insufficient; additionally, square root operations easily underflow when variance is very small.
    - **Loss Function Computation**: Such as cross-entropy loss, which typically requires FP32 to ensure numerical stability (especially the `log-softmax` part).
    - **Optimizer State Updates**: Adam's momentum, variance estimates, and parameter updates use FP32 throughout to guarantee the precision of long-term accumulated parameter updates.
    - **Gradient Accumulation**: The summation of gradients across multiple micro-batches should be performed in FP32 to avoid error accumulation from repeated FP16 additions.

3. **Context-Dependent Operations**:
    - **Dropout**: The operation itself (generating random masks and setting values to zero) carries no numerical risk, so FP16 is sufficient; however, attention must be paid to whether subsequent operations require FP32.
    - **Embedding Lookup**: Outputs are usually fed directly into subsequent layers; storing in FP16 saves VRAM, but conversion to FP32 is needed if subsequent accumulation operations follow.
    - **Position Encoding**: Numerical ranges are limited, so FP16 is usually safe, though attention must be paid to precision during trigonometric calculations as sequence length increases.

#### Practical Recommendations

When using PyTorch `autocast`, the framework automatically determines the precision of each operation based on predefined whitelists and blacklists, requiring no manual user intervention under normal circumstances. For custom operators or special requirements, you can temporarily disable it within the `autocast` context using `torch.amp.autocast(enabled=False)` to force specific operations to execute in FP32:

\begin{lstlisting}[style=promptstyle]
	with torch.cuda.amp.autocast():
	output = model(input)                # Automatically uses FP16/FP32
	with torch.cuda.amp.autocast(enabled=False):
	sensitive_output = sensitive_op(output.float())  # Forces FP32
\end{lstlisting}

</details>

**18. What Are the Memory Optimization Techniques in LLM Training?**

<details>
<summary>💡 Show Answer</summary>

- **Mixed-Precision Training**: Parameters are stored in BF16/FP16, halving memory usage;
- **Gradient Checkpointing**: Intermediate activations are not saved during the forward pass and are recomputed during the backward pass, trading compute for memory, reducing activation memory from $O(L)$ to $O(\sqrt{L})$;
- **ZeRO Optimizer** (DeepSpeed): Optimizer states, gradients, and parameters are sharded across multiple GPUs; ZeRO-3 can compress memory usage down to $1/N$ ($N$ is the number of GPUs);
- **FlashAttention**: Rewrites the attention computation kernel to avoid the $O(n^2)$ memory footprint of the attention matrix;
- **CPU Offload**: Offloads optimizer states to CPU memory;
- **Reducing Batch Size + Gradient Accumulation**: Simulates large batch sizes using multiple small micro-batches, lowering the peak memory footprint per step.

</details>

**19. What Are the Differences Between Data Parallelism, Model Parallelism, and Pipeline Parallelism in LLM Training?**

<details>
<summary>💡 Show Answer</summary>

![Taxonomy of 3D Distributed Parallelism: Data, Tensor, and Pipeline Parallelism.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig29_3d_parallelism_taxonomy.png)




- **Data Parallelism (DP)**: Each GPU holds a complete replica of the model and processes different subsets of data, synchronizing gradients via all-reduce after the backward pass. Simple and efficient, but requires a single GPU to accommodate the full model;
- **Tensor Parallelism (TP)**: Weight matrices of a single layer are partitioned row-wise or column-wise across multiple GPUs, where each GPU computes a fraction of the matrix multiplication, suitable for extremely large single layers (such as FFN, Attention);
- **Pipeline Parallelism (PP)**: The model is partitioned by layer, with different GPUs responsible for different layers, and data flows in a pipelined fashion as micro-batches; the key to reducing inter-GPU waiting time (bubble) lies in partitioning stages reasonably;
- **3D Parallelism**: Combines DP + TP + PP simultaneously, serving as the mainstream strategy for training models with hundreds of billions of parameters (e.g., Megatron-LM).

</details>

**20. What Are the Learning Rate Scheduling Strategies in LLM Training, and How to Choose Them?**

<details>
<summary>💡 Show Answer</summary>

Common learning rate scheduling strategies:

- **Linear Warmup + Cosine Annealing**: Currently the most mainstream approach; after warmup, the learning rate smoothly decays to $\eta_{\min}$ following a cosine curve:

    $$
    \eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max}-\eta_{\min})\left(1+\cos\frac{\pi t}{T}\right)
    $$

- **Linear Warmup + Linear Decay**: Simple to implement, with relatively aggressive decay;
- **Constant with Warmup**: Maintained constant after warmup, suitable for continual pre-training scenarios;
- **WSD (Warmup-Stable-Decay)**: Adopted by works like MiniMax; begins with warmup, followed by long-term stable training, and finally a rapid decay, facilitating training termination at any checkpoint.

Selection recommendations: Cosine annealing is preferred for ultra-large-scale training from scratch; constant or WSD can be used for continual pre-training or fine-tuning.

</details>

**21. What Optimizer Is Typically Chosen in LLM Training, and Why Not Use SGD?**

<details>
<summary>💡 Show Answer</summary>

LLM pre-training almost universally uses the **AdamW** optimizer:


$$
	m_t = \beta_1 m_{t-1} + (1-\beta_1) g_t, \quad
	v_t = \beta_2 v_{t-1} + (1-\beta_2) g_t^2
$$


$$
	\theta_t = \theta_{t-1} - \eta \cdot \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon} - \eta \lambda \theta_{t-1}
$$


Advantages of AdamW compared to SGD:
- Adaptive learning rate: Independently adjusts step sizes for each parameter, enabling stable convergence even when gradient magnitudes differ significantly across layers;
- Momentum mechanism: Accelerates convergence along sparse gradient directions;
- Weight decay decoupling (AdamW vs Adam): Separates weight decay from gradient updates, resulting in a cleaner regularization effect.

SGD requires extensive hyperparameter tuning and is sensitive to the learning rate, making it virtually unviable at the scale of hundreds of millions or billions of parameters.

</details>

**22. How Is Batch Size Set in LLM Training, and What Is Its Relationship with Learning Rate?**

<details>
<summary>💡 Show Answer</summary>

LLM training typically uses an extremely large global batch size (e.g., GPT-3 uses 3.2M tokens/batch) for the following reasons:
- Fully utilizes cluster computational efficiency (reducing communication overhead proportion);
- Provides more accurate gradient estimation and more stable training.

**Linear Scaling Rule**: When the batch size is multiplied by $k$, the learning rate is scaled by $k$ simultaneously to maintain equivalent expected parameter updates:


$$
	\eta' = k \cdot \eta
$$


However, this rule breaks down when the batch size is excessively large and must be paired with warmup. In practice, **square root scaling** is also commonly used: $\eta' = \sqrt{k} \cdot \eta$.

</details>

**23. How to Design Checkpoint Strategies for LLM Training?**

<details>
<summary>💡 Show Answer</summary>

- **Saving Frequency**: Usually saved every fixed number of steps (e.g., 1000 steps) or tokens, balancing storage overhead against fault tolerance granularity;
- **Asynchronous Saving**: Writing checkpoints to disk runs in parallel with training to avoid blocking GPUs;
- **Saved Content**: Model parameters, optimizer states ($m_t, v_t$ of AdamW), learning rate scheduler state, random seeds, current training step, and dataset position (for exact breakpoint resuming);
- **Disaster Recovery Rollback**: When NaN or loss spikes are detected, automatically roll back to the previous healthy checkpoint and skip or filter problematic data batches;
- **Distributed Checkpoint**: Under ZeRO-3, each GPU holds only a parameter shard, requiring coordinated saving/loading (e.g., DeepSpeed's `save_checkpoint`).

</details>

**24. What Is Overfitting and How to Prevent It?**

<details>
<summary>💡 Show Answer</summary>

Overfitting occurs when a model memorizes patterns in the training data rather than generalizing to unseen data. Preventive measures include:
- **Regularization**: Penalizes large weights (e.g., L2 regularization).
- **Dropout**: Randomly disables neurons during training to promote robustness.
- **Data Augmentation**: Expands the diversity of training data.
- **Early Stopping**: Stops training when validation performance plateaus.
- **Simpler Models**: Reduces complexity to prevent overfitting.

</details>

**25. How to Handle Overfitting in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Overfitting can cause an LLM to produce nonsensical outputs or excessively mimic the training data. We can leverage techniques used in other machine learning models for handling overfitting. One approach is data augmentation, which artificially expands the training data by creating variations of existing examples. This forces the LLM to learn underlying patterns rather than memorizing specific phrases. In addition, regularization techniques such as Dropout can be employed. Dropout randomly deactivates neurons during training, preventing the model from overly relying on any single feature, thereby promoting better generalization capabilities.

</details>

**26. How Does Scaling Law Analysis Help Optimize LLM Design?**

<details>
<summary>💡 Show Answer</summary>

Scaling law analysis provides insights into how model performance changes with parameters, dataset size, and computational resources. By studying these relationships, developers can:

- **Optimize Model Size**: Determine the point of diminishing returns for increasing parameter count, helping strike a balance between model complexity and performance.
- **Improve Computational Efficiency**: Scaling laws guide the allocation of compute resources, ensuring the model is trained on an optimal volume of data corresponding to its size.
- **Maximize Performance**: Understanding scaling laws enables developers to fine-tune architecture, training data, and compute allocation to achieve optimal performance within resource constraints.
Scaling laws are crucial for developing LLMs that are both high-performing and computationally efficient.

</details>

**27. Which Key Metrics Need to Be Monitored During LLM Training?**

<details>
<summary>💡 Show Answer</summary>

**1. Loss Curves**
- **Training Loss**: Should decrease smoothly; spikes require immediate investigation;
- **Validation Loss**: The gap between validation loss and training loss reflects the degree of overfitting;
- **Perplexity**: $PPL = e^{\mathcal{L}}$, intuitively measuring the quality of the language model.

**2. Gradient Norm**

$$
	\|\mathbf{g}\|_2 = \sqrt{\sum_i g_i^2}
$$

The gradient norm should fluctuate within a reasonable range; continuous increases indicate gradient explosion, while continuously approaching zero indicates gradient vanishing or learning stagnation.

**3. Activation Distribution**
- Mean and variance of activations across layers;
- A large amount of saturation (activations tending toward extreme values) or all-zeros (Dead Neurons) indicates problems with initialization or normalization;
- Pay attention to the distribution of Attention weights to check for attention collapse (certain heads degenerating into uniform distributions).

**4. Learning Rate and Optimizer States**
- Whether the current learning rate changes normally according to the schedule curve;
- An abnormal increase in Adam's second moment $v_t$ indicates extremely high gradient variance for a specific parameter.

**5. Hardware and Efficiency Metrics**
- GPU utilization (MFU, Model FLOPs Utilization);
- VRAM usage, communication time (all-reduce time proportion);
- Whether throughput (tokens/sec) remains stable.

</details>

**28. How to Determine if the Model is Training Effectively?**

<details>
<summary>💡 Show Answer</summary>

**Signals of Effective Training:**
- Training loss decreases steadily without prolonged plateaus;
- Validation loss decreases synchronously, maintaining a controllable gap with training loss;
- Gradient norms fluctuate within a reasonable range (e.g., $[0.1, 5.0]$);
- Downstream sampled task accuracy improves with training steps.

**Identifying Signals of Underfitting:**
- Both training loss and validation loss remain high and stop decreasing;
- Model-generated content is repetitive or semantically incoherent;
- Gradient norm is extremely small ($< 10^{-3}$), and parameters are barely updating;
- Initial stagnation caused by an excessively small learning rate or unfinished warmup;
- Poor data quality (large amount of noise or duplicate data) leads to insufficient effective signals.

**Troubleshooting Steps:** Check learning rate schedule $\rightarrow$ Verify whether the data pipeline is normal $\rightarrow$ Validate model initialization $\rightarrow$ Small-scale overfitting experiment (check whether loss can be reduced to an extremely low level on a small dataset).

</details>

**29. How to Perform Evaluation During Training?**

<details>
<summary>💡 Show Answer</summary>

**1. Held-out Perplexity**

Set aside a validation set from the training set that does not participate in training, and periodically calculate perplexity:

$$
	PPL_{val} = \exp\left(-\frac{1}{N}\sum_{i=1}^{N} \log P(x_i \mid x_{<i})\right)
$$

The validation set should cover multiple domains (code, natural language, mathematics, etc.). Reporting PPL by domain can help diagnose data mixture issues.

**2. Sampled Downstream Task Evaluation**
- Evaluate on representative benchmarks (such as MMLU, HellaSwag, ARC, etc.) at regular step intervals (e.g., every 5,000 steps);
- Adopt a few-shot setup (0-shot / 5-shot) to avoid introducing fine-tuning bias;
- Sample only a subset of data to control evaluation time cost (e.g., sample 200 instances per task).

**3. Human/Model Sampling of Generation Quality**

Periodically perform generation on a fixed set of prompts and score the outputs manually or with a strong model to capture degradation phenomena (e.g., repetition, hallucination) that PPL cannot reflect.

</details>

**30. How to Diagnose Training Issues Using Logs and Visualization Tools?**

<details>
<summary>💡 Show Answer</summary>

**Common Tools:** TensorBoard, Weights & Biases (WandB), MLflow.

**Key Practices:**
- **Scalar Dashboard**: Compare train loss, val loss, grad norm, and lr on the same screen to spot anomalies at a glance;
- **Histogram Dashboard**: Record distributions of weights, gradients, and activation values for each layer every $N$ steps to observe distribution shifts;
- **Custom Alerts**: Set alerts in WandB triggered when loss $>$ threshold or grad norm $>$ threshold;
- **Dual Axes for Steps and Tokens**: Display both training steps and consumed tokens on the horizontal axis to facilitate cross-experiment comparisons;
- **System Metrics**: Report GPU utilization, VRAM, and network bandwidth synchronously to perform joint analysis of training efficiency and model metrics;
- **Experiment Version Control**: Record hyperparameters and code commit hashes for each experiment to facilitate reproducibility and comparison.

% =============================================

</details>

**31. How to Handle Hardware Failures in Large-Scale Training?**

<details>
<summary>💡 Show Answer</summary>

In large-scale cluster training (hundreds to thousands of GPUs), hardware failures (GPU crashes, network outages, storage errors) are the norm rather than the exception.

**Main Strategies:**
- **Periodic Checkpointing**: Save the full training state at fixed step intervals, rolling back to the nearest healthy checkpoint after a failure. Typical intervals range from 500$\sim$2000 steps;
- **Asynchronous Checkpointing**: Parallelize disk writing with training to avoid blocking GPUs (e.g., PyTorch's `async_save`);
- **Elastic Training**: Frameworks (e.g., PyTorch Elastic, Horovod) support dynamic joining/leaving of nodes, allowing remaining nodes to continue training without restarting the entire job;
- **Automatic Restart**: Configure health checks and automatic resubmission using scheduling systems (SLURM, Kubernetes), automatically spinning up processes and resuming from the latest checkpoint upon detecting abnormal exits;
- **Redundant Communication Topologies**: Use fault-tolerant all-reduce libraries (e.g., NCCL's `ncclCommAbort` + reconstruction mechanism).

</details>

**32. What Consistency Issues Should Be Noted When Resuming Training from a Checkpoint?**

<details>
<summary>💡 Show Answer</summary>

If state inconsistency occurs when resuming training, it can cause unexpected loss spikes at best, or irreproducible results at worst. Key considerations include:

- **RNG State (Random Number Generator)**: The random seed states across CPU, CUDA, numpy, and Python layers must be restored; otherwise, the sequence of random operations like dropout and data augmentation will drift, harming reproducibility;
- **Data Offset**: Record consumed dataset positions (file index + line offset) and continue reading from the breakpoint upon resuming, avoiding duplicate consumption or skipping data;
- **Optimizer State**: The first-moment estimate $m_t$, second-moment estimate $v_t$, and current step count $t$ of AdamW must be fully restored. Missing them causes adaptive learning rates to reset, leading to loss spikes;
- **Learning Rate Schedule State**: The current step count of the scheduler must be strictly aligned with the model step count;
- **Distributed Consistency**: Under ZeRO partitioning, each GPU saves its own parameter/optimizer partition. When resuming, ensure that the GPU count and partitioning strategy remain unchanged, or use tools supporting re-partitioning (e.g., DeepSpeed's `zero_to_fp32.py`).

% =============================================

</details>

**33. Differences and Considerations Between Continual Pretraining and Training from Scratch**

<details>
<summary>💡 Show Answer</summary>

**Differences:**
- **Training from Scratch**: Parameters are initialized randomly and trained on a full mixed dataset, consuming trillions of tokens;
- **Continual Pretraining (CPT)**: Training continues on top of an existing pre-trained model for specific domains (e.g., medical, code, Chinese) or updated knowledge. The starting point already possesses general capabilities, requiring far fewer tokens (typically billions to hundreds of billions).

**CPT Considerations:**
- **Catastrophic Forgetting**: An excessively high proportion of new data will overwrite existing general capabilities; a certain proportion of general corpus (e.g., 10%$\sim$20%) needs to be mixed in for "replay";
- **Learning Rate Setup**: Should be far smaller than the peak learning rate used when training from scratch, typically set to $1/10$ of the original peak rate, accompanied by a short warmup;
- **Data Distribution Matching**: Target domain data must maintain a consistent format with original training data (same Tokenization) to avoid distribution shifts;
- **Evaluation Baselines**: Dual-directional evaluations on both general benchmarks and target-domain benchmarks are required before and after CPT to confirm trade-offs between "gains" and "losses".

</details>

**34. What Are the Engineering Challenges of Long-Context Pre-training?**

<details>
<summary>💡 Show Answer</summary>

![Long-context pre-training distributed memory and sequence parallel architecture.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig26_long_context_pretraining.png)




Extending the context window from 4K to 128K or even longer presents the following challenges:

**1. Computation and VRAM**

The computational complexity of standard attention is $O(n^2)$; doubling the sequence length quadruples the attention VRAM usage. Solutions include:
- **Flash Attention**: IO-aware tiled attention computation, reducing VRAM footprint from $O(n^2)$ to $O(n)$;
- **Sequence Parallelism**: Splitting long sequences across multiple GPUs and leveraging Ring Attention to achieve inter-GPU attention computation.

**2. Position Encoding Extrapolation**

RoPE has limited generalization capability beyond its trained length. Common extension schemes include:
- **Position Interpolation**: Compresses position indices into the trained context range: $pos' = pos \times \frac{L_{train}}{L_{target}}$;
- **YaRN**: Applies different scaling strategies to different frequency components, outperforming linear interpolation;
- **LongRoPE**: Searches non-uniform scaling factors to maximize the retention of short-distance position information.

**3. Training Data**

Long-text corpora are scarce; books, long documents, code repositories, etc. must be specially collected, and reasonable sequence concatenation and truncation strategies need to be designed.

</details>

**35. Data Mixture Adjustment Strategies in the Annealing Phase**

<details>
<summary>💡 Show Answer</summary>

The annealing phase refers to the final stage of training where the learning rate decays rapidly (typically covering the last 5%$\sim$10% of tokens). Data mixture adjustment during this stage is crucial:

- **Increase High-Quality Data Ratio**: Reduce web corpus and increase weights of books, papers, and high-quality QA data, allowing the model to "refine" knowledge prior to convergence;
- **Incorporate Target Domain Data**: If there is a specific downstream application, blending relevant data during annealing significantly boosts domain capabilities (at a cost far lower than CPT);
- **Reduce Noisy Data**: Filter out sources with low quality or high duplicate ratios;
- **WSD Strategy**: The Warmup-Stable-Decay training paradigm designs the annealing phase independently, making it convenient to attach different annealing data recipes to checkpoints at any stable stage, enhancing experimental flexibility.

</details>

**36. What Are the Quality Filtering Strategies for Pre-training Data?**

<details>
<summary>💡 Show Answer</summary>

- **Heuristic Rule Filtering**:
    - Language detection (removing non-target languages);
    - Length filtering (overly short texts, texts containing only punctuation/digits);
    - Character-level quality metrics (alphabet ratio, special character ratio, average line length);
    - Perplexity filtering: Use small language models to score text and filter out text with extremely high PPL (garbled code) or extremely low PPL (template repetitions).
- **Model Scoring**: Train classifiers (e.g., based on fastText or small BERT) to distinguish high-quality text (e.g., Wikipedia) from low-quality text, filtering out low-scoring samples (adopted by both CCNet and FineWeb);
- **Content Safety Filtering**: Remove harmful content such as pornography, violence, and hate speech;
- **Personally Identifiable Information (PII) Filtering**: Apply regular expressions to remove PII such as phone numbers, emails, and ID numbers.

</details>

**37. How to Perform Data Deduplication?**

<details>
<summary>💡 Show Answer</summary>

Duplicate data causes models to memorize specific content, reduces data diversity, and inflates reported token consumption.

**1. Exact Deduplication**
- Compute MD5/SHA hashes of text and remove exact duplicates;
- Can only handle exact matches and cannot capture near-duplicates.

**2. MinHash Near-Deduplication**
- Represent text as a set of $n$-grams and use MinHash signatures to approximate Jaccard similarity:

    $$
    J(A, B) = \frac{|A \cap B|}{|A \cup B|} \approx \Pr[\min h(A) = \min h(B)]
    $$

- Pair with LSH (Locality-Sensitive Hashing) to efficiently discover similar document pairs, with a time complexity far lower than pairwise comparison;
- Typical threshold: Jaccard $> 0.8$ is treated as duplicate, keeping only one instance.

**3. SimHash / Semantic Deduplication**
- SimHash applies random projection to word frequency vectors to generate fixed-length binary signatures; smaller Hamming distances indicate similarity;
- Semantic Deduplication: Use embedding models to calculate text vectors and deduplicate when cosine similarity exceeds a threshold. This approach has the highest computational cost but achieves the best results.

</details>

**38. How to Design Data Mixture?**

<details>
<summary>💡 Show Answer</summary>

Data mixture directly dictates the capability boundary of the model. Design principles:

- **Based on Downstream Targets**: If the goal is a general-purpose model, natural language, code, mathematics, and multilingual data must be balanced; if the target is a code model, the proportion of code data can be increased to 50%+;
- **Empirical Mixture Reference**: LLaMA 3 utilizes approximately 50% general web text + 25% code + math, multilingual data, etc.;
- **Scaling Law Guidance**: Exhaustively evaluate mixture options on small models, predict the optimal mixture using validation PPL or downstream task performance, and transfer to large models (methods such as DoReMi, RegMix, etc.);
- **Dynamic Mixture**: Dynamically adjust weights during training based on the loss decay speed of each domain, reducing sampling for domains with diminishing returns.

</details>

**39. How to Choose Tokenization Strategies for Training Data?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]



| |p{3.5cm}|p{3cm}|} |
| --- |
| **Method** |
| BPE |
| WordPiece |
| SentencePiece |
| Byte-level BPE |


\end{table}

**Selection Recommendations:**
- For multilingual LLMs, **SentencePiece + BPE** is preferred (e.g., LLaMA, Qwen);
- For English-dominated models, **Byte-level BPE** can be selected (e.g., GPT-2/3/4);
- Vocabulary size is typically between 32K$\sim$128K; a larger vocabulary improves compression rate but increases Embedding parameter count.

</details>

**40. How Do Data Ordering and Curriculum Learning Affect Training?**

<details>
<summary>💡 Show Answer</summary>

![Curriculum learning data scheduling across pre-training stages.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig21_curriculum_learning_schedule.png)




**Impact of Data Ordering:**
- Random shuffling is a fundamental operation that prevents the model from overfitting to the sequential patterns of a particular data source;
- Order is maintained within the same document, while different documents are shuffled randomly;
- Different data epoch orderings will affect the convergence path, but the final performance difference is usually small.

**Curriculum Learning:** Simulates the human learning process from easy to difficult by arranging the training sequence in order of increasing sample difficulty.

- **Difficulty Metrics**: Perplexity (PPL, scored by a smaller model), text length, vocabulary complexity, etc.;
- **Potential Benefits**: Early exposure to simple samples helps stabilize initial training and accelerate convergence;
- **Limitations**: Gains are unstable in large-scale pre-training, with high implementation complexity; currently, it is used more in the fine-tuning stage than in the pre-training stage;
- **Implicit Curriculum**: Increasing data quality during the annealing phase is essentially a curriculum strategy from "coarse" to "fine".

</details>

**41. LLM Training OOM Issues: Performance & Memory Optimization Tricks**

<details>
<summary>💡 Show Answer</summary>

Common tricks to resolve/mitigate OOM:

1. **Mixed Precision Training (FP16/BF16)**: Reduces GPU memory usage for activations and gradients.
2. **Gradient Accumulation**: Splits a large batch size into multiple small micro-batches, sequentially computes gradients, and accumulates them to simulate large-batch training.
3. **Gradient Checkpointing / Activation Recomputation**: Does not save all intermediate activation values during training, but recomputes them during backpropagation, trading computation time for GPU memory space.
4. **ZeRO Series Optimizations (DeepSpeed)**: Shards optimizer states, gradients, and parameters across multiple GPUs.
5. **Model Parallelism / Tensor Parallelism / Pipeline Parallelism**: Splits the model itself across multiple GPUs.
6. **CPU/NVMe Offloading**: Offloads optimizer states and part of the parameters to CPU memory or disk.
7. **Using More Efficient Attention Implementations (e.g., FlashAttention)**: Reduces GPU memory overhead of intermediate matrices.
8. **Reducing Sequence Length / Batch Size**, or using more efficient optimizers (e.g., 8-bit Adam).
9. **PEFT Methods like LoRA/QLoRA**: Drastically reduces the number of parameters requiring gradient and optimizer state storage.

</details>

**42. During LLM Inference, What Components Occupy GPU Memory?**

<details>
<summary>💡 Show Answer</summary>

1. **Model Weights**: The primary component occupying GPU memory, with size depending on the parameter count and precision.
2. **KV Cache**: Caches the Key and Value vectors for each token in every layer during autoregressive generation. Its size is directly proportional to the number of layers, batch_size, sequence length, number of heads, and head_dim, making it the fastest-growing component during the inference phase.
3. **Activations**: Intermediate calculation results currently being processed. During inference, typically only the activations of the current token need to be retained.
4. **Temporary Buffers / Intermediate Computational Results**: Such as attention score matrices, softmax intermediate results, etc.
5. **Scheduling / Fragmentation Overhead**: Such as PagedAttention block management metadata in vLLM.

The GPU memory requirement during inference is much smaller than that during training, with the primary memory bottleneck concentrated in **Model Weights + KV Cache**.

</details>

**43. How Much GPU Memory Is Occupied During Model Training and Inference?**

<details>
<summary>💡 Show Answer</summary>

**Training Phase (taking full fine-tuning with Adam optimizer and FP16 mixed precision as an example)**:
- **Model Weights**: FP16 requires approximately 2 bytes $\times$ parameter count; FP32 master weight copy adds another 4 bytes $\times$ parameter count.
- **Gradients**: Approximately 2 bytes $\times$ parameter count (FP16).
- **Optimizer States (Adam)**: First-order momentum + second-order momentum (FP32), approximately 8 bytes $\times$ parameter count.
- **Activations**: Proportional to batch size, sequence length, and number of layers; can be compressed via gradient checkpointing.

Rough Estimation: Each parameter requires approximately 16--18 bytes, meaning a 7B model requires about 110--130 GB of GPU memory (excluding activations).

**Inference Phase**: Requires only model weights (e.g., FP16 approx. 2 bytes $\times$ parameter count) + KV Cache + a small amount of activations; baseline FP16 inference for a 7B model requires approx. 14 GB + KV Cache.

</details>

**44. What Is Mixed Precision Training?**

<details>
<summary>💡 Show Answer</summary>

**Mixed precision training** refers to the technique of **simultaneously using low-precision (FP16/BF16) and high-precision (FP32)** data types during training to significantly reduce GPU memory usage and accelerate computation while maintaining training numerical stability.

Typical implementation workflow:
- **Forward and Backward Pass**: Computed using FP16/BF16.
- **Master Weight Copy**: Retains an FP32 precision parameter copy for weight updates.
- **Loss Scaling**: Multiplies the loss value by a large scale factor so that gradient values fall within the representable range of FP16.

**FP16 vs. BF16**: FP16 offers higher precision but has a smaller dynamic range, requiring loss scaling; BF16 has the same exponent bit-width as FP32, making its numerical range more stable and generally eliminating the need for loss scaling, which makes it the preferred choice in modern LLM training.

</details>

**45. Overview of Common Optimizers**

<details>
<summary>💡 Show Answer</summary>

1. **SGD / SGD with Momentum**: The most fundamental optimizer, rarely used alone in LLM training.
2. **Adagrad**: Maintains the sum of historical squared gradients for each parameter to adaptively adjust the learning rate, but the learning rate decays towards 0 prematurely.
3. **RMSProp**: Uses the exponential moving average of squared gradients instead of accumulated sums to prevent the learning rate from decaying too fast.
4. **Adam**: Combines Momentum and RMSProp, currently the most commonly used optimizer, typically with hyperparameters $\beta_1=0.9, \beta_2=0.95$.
5. **AdamW**: Decouples weight decay from gradient updates, currently serving as the de facto standard optimizer for LLM training.
6. **Lion**: Only maintains a single momentum state and updates directions based on the sign of momentum, resulting in lower GPU memory usage.
7. **Adafactor**: Reduces memory footprint by performing low-rank matrix factorization on second-order momentum matrices, often used in ultra-large-scale model training.

</details>

**46. GPU Memory Footprint of Common Optimizers During Training**

<details>
<summary>💡 Show Answer</summary>

Taking $N$ parameters and FP32 as the baseline, the GPU memory consumption of optimizer states (excluding parameters themselves and gradients):

- **SGD (without Momentum)**: 0 (no extra state required).
- **SGD with Momentum**: Approx. 4N bytes.
- **Adam / AdamW**: First-order momentum + second-order momentum, approx. 8N bytes, meaning an additional 8 bytes per parameter.
- **Adafactor**: By factorizing second-order momentum into row and column sums, memory consumption is far smaller than Adam.
- **Lion**: Only requires 1 momentum state, approx. 4N bytes, half that of Adam.

When using AdamW + mixed precision, the total GPU memory cost per parameter is $\approx$ 2 (FP16 params) + 2 (FP16 grads) + 4 (FP32 master weight copy) + 8 (Adam states) = 16 bytes.

</details>

**47. What Are the Main Workflows and Differences Between LLM Training Mode and Inference Mode?**

<details>
<summary>💡 Show Answer</summary>

**Training Mode**: Employs Teacher Forcing, where the previous token of the ground-truth target sequence serves as the input to the decoder at the current timestep. A masking mechanism is used to ensure each timestep can only attend to previous tokens.

**Inference Mode**: Autoregressive generation, where the decoder generates one token at each step and feeds it back as the input for the next timestep until an end-of-sequence token is generated or the maximum length is reached.

</details>

