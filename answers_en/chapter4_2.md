# 4.2 Overview of LoRA / QLoRA and PEFT Methods — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 34 questions

**1. What Are the Common Fine-Tuning Methods for Large Models?**

<details>
<summary>💡 Show Answer</summary>

![Architectural Taxonomy and Structural Paradigms of Common Parameter-Efficient Fine-Tuning (PEFT) Methods](https://img.pricenode.uk/llm_interview_questions/english/chapter4/peft_methods_taxonomy.png)




Parameter fine-tuning methods mainly include the following:

**Adapter**: Adds Adapter modules inside each layer (or certain layers) of a pre-trained model. During fine-tuning, the main body of the pre-trained model is frozen, and the Adapter modules learn the knowledge for specific downstream tasks. Each Adapter module consists of two feed-forward sub-layers.

**Prefix-tuning**: Prefix-tuning prepends a sequence of continuous, task-specific vectors to the input, referred to as the prefix. Unlike discrete prompts, the prefix consists entirely of free parameters that do not correspond to real tokens. Compared to traditional fine-tuning, prefix-tuning only optimizes the prefix, requiring only a single copy of the large Transformer alongside the task-specific prefixes to be stored.

**P-tuning/P-tuning V2**: Directly optimizing the embedding layer in P-tuning V1 faced two challenges: the embedding layer was already pre-trained, so directly randomly initializing and training the input prompt embeddings made it easy to get trapped in local optima; additionally, it could not capture dependencies among prompt embeddings. P-Tuning V2 is an upgraded version that adds prompts across all layers with residual connections, allowing features to be represented more fully.

**LoRA**: Adopts the idea of low-rank matrix approximation by freezing the weight matrices of a pre-trained model and representing updates using matrices $A$ and $B$, updating only $A$ and $B$ during downstream tasks.

</details>

**2. What Are the Different Types of SFT Methods?**

<details>
<summary>💡 Show Answer</summary>

1. **Full Fine-Tuning**: Updates all parameters of the model, yielding the best performance in general but incurring the highest compute and storage costs, while being susceptible to overfitting on small datasets or catastrophic forgetting.

2. **Parameter-Efficient Fine-Tuning (PEFT)**: Trains only a small subset of parameters, drastically reducing costs:
    - **LoRA**: Inserts trainable low-rank decomposition matrices $A$ and $B$ alongside weight matrices while keeping original weights frozen, which can be merged during inference.
    - **QLoRA**: Performs LoRA fine-tuning on top of a quantized base model to further lower GPU memory requirements.
    - **Prefix/Prompt Tuning**: Prepends a trainable "soft prompt" vector sequence to the input while freezing the main model.
    - **Adapter**: Inserts small trainable modules between Transformer layers.
    - Ultra-lightweight methods like **(IA)$^3$** and **BitFit** that train only scaling factors for activations or bias terms.

3. **Instruction Tuning**: Uses a large volume of "instruction-response" pairs to train the model to follow diverse natural language instructions.

4. **Multi-Task / Multi-Turn Dialogue Fine-Tuning**: Designs training samples with role markers (system/user/assistant) tailored to multi-turn conversation scenarios, enabling the model to adapt to dialogue formats.

</details>

**3. What Is LoRA? What Is Its Core Concept?**

<details>
<summary>💡 Show Answer</summary>

![LoRA (Low-Rank Adaptation) Matrix Decomposition Mechanism and Forward Pass Computation](https://img.pricenode.uk/llm_interview_questions/english/chapter4/lora_matrix_decomposition.png)




**LoRA (Low-Rank Adaptation)** is a parameter-efficient fine-tuning (PEFT) method. Its core concept is:

The update weight matrix $\Delta W$ of pre-trained models during fine-tuning exhibits low-rank properties; thus, $\Delta W$ can be decomposed into the product of two small matrices: $\Delta W = A \times B$, where $A \in R^{d \times r}$, $B \in R^{r \times k}$, and $r$ is much smaller than $d$ and $k$.

During training, original weights are frozen, and only $A$ and $B$ are trained, dramatically reducing trainable parameters (typically by over 90%). During inference, $\Delta W$ can be merged back into the original weights, introducing zero additional inference latency.

</details>

**4. What Is a Low-Rank Matrix? What Is Low-Rank Decomposition? Why Can the Weight Update of Large Models Be Assumed to Be Low-Rank?**

<details>
<summary>💡 Show Answer</summary>

**Low-Rank Matrix**: An $m \times n$ matrix whose rank $r \ll \min(m, n)$, meaning most columns (or rows) are linearly dependent, and only a few directions carry the main information of the matrix.

**Low-Rank Decomposition**: Approximating the original matrix $W$ as the product of two small matrices, such as $W \approx A \times B$, where $A \in \mathbb{R}^{m \times r}$ and $B \in \mathbb{R}^{r \times n}$, reducing storage and compute costs from $\mathcal{O}(mn)$ to $\mathcal{O}((m+n)r)$.

**Why Are Weight Updates Low-Rank?** Aghajanyan et al. (2020) ("Intrinsic Dimensionality") discovered that large pre-trained models have an extremely low "intrinsic dimension" during fine-tuning---meaning fine-tuning essentially modifies only a very low-dimensional subspace in the parameter space. The original LoRA paper also empirically verified that the effective rank of the weight difference $\Delta W$ before and after fine-tuning is very low, and a matrix with a rank of 4 or 8 can approximate the vast majority of updates.

Intuitive understanding: Pre-training has already established rich representations; adaptation to downstream tasks only requires "fine-tuning" along a few key directions without large-scale reshaping of the entire weight space.

</details>

**5. Please Explain the Principle and Implementation of LoRA.**

<details>
<summary>💡 Show Answer</summary>

**LoRA (Low-Rank Adaptation)** is a parameter-efficient fine-tuning (PEFT) method whose core idea is: **During fine-tuning, the update quantity $\Delta W$ of the weight matrix typically exhibits low-rank characteristics**; thus, $\Delta W$ can be approximated by the product of two low-rank matrices without directly training the full $\Delta W$.

**Principle**:

For a weight matrix $W_0$ (with dimensions $d \times k$) in a pre-trained model, full parameter fine-tuning is equivalent to learning an update $\Delta W$, making the new weight $W = W_0 + \Delta W$. LoRA assumes that $\Delta W$ can be decomposed into the product of two low-rank matrices:

$\Delta W = B \cdot A$

where the dimensions of $A$ are $r \times k$, the dimensions of $B$ are $d \times r$, $r$ is a predefined rank, and $r \ll \min(d, k)$. In this way, the parameter count of $\Delta W$ is reduced from $d \times k$ to $r \times (d+k)$. When $r$ is very small (e.g., 4, 8, 16), the parameter count drops dramatically.

**Forward Computation**:

For an original linear transformation $h = W_0 \cdot x$, LoRA modifies it to:

$h = W_0 \cdot x + B \cdot A \cdot x = W_0 \cdot x + (B \cdot A) \cdot x$

That is, the original weight $W_0$ **remains frozen**, while only the newly added low-rank matrices $A$ and $B$ are trained.

**Initialization Strategy**:
- $A$ is typically randomly initialized using a Gaussian distribution;
- $B$ is typically initialized to a zero matrix;
- Thus, at the start of training, $\Delta W = B \cdot A = 0$, ensuring the initial model behavior is identical to the original pre-trained model, guaranteeing a stable starting point for training.

**Implementation Details**:

1. Select the weight matrices to apply LoRA to (typically the $Q$ and $V$ projection matrices in Attention modules);
2. Add a pair of low-rank matrices $A$ and $B$ (as new trainable parameter modules) for each selected weight matrix, setting the original weight matrices to non-trainable (frozen);
3. During training, only $A$ and $B$ participate in gradient updates;
4. Upon completing training:
- **Merge during inference**: Compute $\Delta W = B \cdot A$ and directly add it to the original weights $W_0$ to obtain new weights $W = W_0 + B \cdot A$. After merging, the model structure is identical to the original model, introducing no additional compute overhead or latency during inference;
- **Keep unmerged**: Retain the original weights and LoRA modules separately to allow flexible switching among multiple LoRA modules.

</details>

**6. Why Is $B$ Initialized to 0 and $A$ Initialized to a Gaussian Distribution in LoRA?**

<details>
<summary>💡 Show Answer</summary>

First, $A$ and $B$ cannot both be initialized to 0, because if both are 0, there will be no gradients. Therefore, one part needs to be initialized to 0 while the other part is normally initialized, ensuring that at initialization the model produces the exact same output as the original model. If neither $A$ nor $B$ is 0, the model output at initialization would be perturbed relative to the original model, introducing noise that harms training convergence compared to an initialization where $\Delta W = 0$.

Theoretical proofs show that initializing $A$ to a Gaussian distribution and $B$ to 0 yields better performance, for the following reasons:
1. Init$[A]$ allows the use of a larger learning rate than Init$[B]$.
2. Init$[A]$ can cause an "internal instability" where for certain inputs $z$, the feature $Az$ is large, but the LoRA output $BAz$ is small. This instability enables more efficient feature learning.
3. Init$[B]$ does not cause any instability, but training in this scenario is sub-optimal (matrix $B$ is under-trained).
4. Empirical results confirm the theory and demonstrate that Init$[A]$ generally leads to better performance than Init$[B]$.

</details>

**7. What Is the Purpose of the Scaling Factor ($\alpha/r$) in the LoRA Formula?**

<details>
<summary>💡 Show Answer</summary>

The forward computation of LoRA is: **$h = W_0 x + (\alpha/r) \cdot B A x$**

Purpose of the scaling factor $\alpha/r$:

- **Magnitude Control**: Directly determines the proportion of contribution that the LoRA adaptation increment $\Delta W = BA$ makes to the output, preventing overly large or small update magnitudes in low-rank matrices from hurting training stability;
- **Decoupling from Learning Rate**: After fixing $\alpha$, changing $r$ does not require adjusting the learning rate simultaneously---as $r$ increases, $\alpha/r$ decreases, automatically scaling the update magnitude;
- **Practical Significance**: Usually set to $\alpha = r$ (scaling factor of 1) or $\alpha = 2r$ (scaling factor of 2). The original paper found that after setting $\alpha$ to a constant multiple of $r$, tuning $r$ no longer required re-tuning the learning rate, simplifying hyperparameter search;
- **Essence**: Similar to the gain parameter mechanism in LayerNorm, used to stabilize training dynamics.

</details>

**8. How Should the Rank $r$ Be Selected in LoRA?**

<details>
<summary>💡 Show Answer</summary>

The rank $r$ is the most critical hyperparameter in LoRA, determining the expressiveness of the low-rank update matrix $\Delta W = B \cdot A$. The following factors should be considered when selecting $r$:

**1. The Essential Meaning of $r$**

$r$ determines the upper bound on the rank of the "update" matrix $\Delta W$, which represents the upper limit on the complexity of "new information" the model can learn during fine-tuning. A larger $r$ allows $\Delta W$ to express richer patterns of change, but at the same time, the number of trainable parameters increases linearly (parameter count $\propto r \times (d+k)$).

**2. Impact of Task Complexity and Data Volume**

- **Small difference between task and pre-training objectives, small data volume**: Typically, a smaller $r$ (e.g., 4, 8) is sufficient because the model only needs minor adjustments; an excessively large $r$ is prone to overfitting on small datasets and wastes computational resources.
- **Large difference between task and pre-training objectives (e.g., significant domain shift, needing to learn substantial new knowledge), large data volume**: A larger $r$ (e.g., 32, 64, or even higher) may be required to provide sufficient expressive power to capture more complex update patterns.

**3. Empirical Range of Values**

- In practice, common values for $r$ range from **4 to 64**. For most downstream tasks, **$r=8$ or $r=16$** often achieves performance close to full parameter fine-tuning and serves as a commonly used default starting point;
- For complex generative tasks or scenarios requiring injection of significant new knowledge, larger values of $r$ (e.g., 32, 64) can be attempted, though diminishing returns are typically observed.

**4. Coordination with $\alpha$ (Scaling Factor)**

LoRA usually has a scaling factor $\alpha$, where the final update is $(\alpha/r) \cdot B \cdot A$. A common practice is to fix the ratio $\alpha/r$ (e.g., $\alpha = 2r$ or $\alpha = r$), maintaining this ratio while tuning $r$.

**5. Experimental Tuning Suggestions**

It is recommended to perform small-scale experiments (e.g., $r \in \{4, 8, 16, 32\}$) on a validation set to compare performance and select a value that achieves a good balance between efficacy and training cost.

**Summary**: Selecting $r$ is essentially a trade-off between "model expressive capacity" and "parameter efficiency / overfitting risk", with **$r=8$--$16$ being a reasonable starting point for most tasks**.

</details>

**9. Where Is LoRA Typically Applied in the Model, and Why?**

<details>
<summary>💡 Show Answer</summary>

**1. Most Common Location: $Q$ and $V$ Projection Matrices in Attention Modules**

Experiments in the original LoRA paper found that **applying LoRA only to $W_q$ (Query projection) and $W_v$ (Value projection) in Attention modules** achieves performance close to full parameter fine-tuning on most tasks. The reasons may include:
- Query projection affects "what information the model attends to" (the attention distribution pattern), while Value projection affects "what information the model extracts from attended positions." Adjusting these two has a relatively direct and significant impact on model behavior;
- In contrast, Key projection ($W_k$) and output projection ($W_o$) exhibit relatively smaller responses to LoRA increments.

**2. Extending to More Locations**

In practice, applying LoRA to the following locations is also common:
- **$W_q$, $W_k$, $W_v$, $W_o$ (All Attention Projection Matrices)**: Provides more comprehensive attention pattern adjustment capabilities;
- **FFN Layers (e.g., `up_proj`, `down_proj`, `gate_proj`)**: FFN layers account for a larger proportion of parameters and contain most of the model's "knowledge storage". For tasks requiring injection of substantial new knowledge, applying LoRA to FFN layers often yields more noticeable performance improvements;
- **Embedding Layer and Output Layer (`lm_head`)**: If the task involves vocabulary changes or requires substantial adjustments to the output distribution, applying LoRA to these components can also be considered.

**3. Considerations for Selecting Locations**

- Tasks leaning toward "behavior/style adaptation" (e.g., conversational style, instruction-following format): Attention-related LoRA is usually sufficient;
- Tasks leaning toward "knowledge injection" (e.g., domain-specific knowledge, new facts): Adding LoRA to FFN layers often brings greater gains, as FFNs are believed to carry more "factual knowledge".

</details>

**10. Which modules of Transformer does LoRA usually act on? What is the difference between fine-tuning only Attention and fine-tuning both Attention and MLP?**

<details>
<summary>💡 Show Answer</summary>

**Common target_modules choices**:

- Attention only: q_proj, v_proj (default in the original paper), or adding k_proj, o_proj;
- Attention + MLP: further adding gate_proj, up_proj, down_proj;
- All linear layers: including embed_tokens, lm_head (less common, high cost).

**Difference Analysis**:

\begin{table}[h]

	\caption{Comparison of LoRA Target Modules}


| **Strategy** | **Parameters** | **Applicable Scenarios** | **Description** |
| --- | --- | --- | --- |
| Attention Only | Small | Semantic understanding, dialogue SFT | Attention controls inter-word relationships and information routing, which is sufficient for most tasks |
| Adding MLP | Large (approx. 2--3$\times$) | Domain knowledge injection, factual output modification | MLP is key to knowledge storage; injecting new knowledge yields more noticeable effects |


\end{table}

Practical Recommendation: In Supervised Fine-Tuning (SFT), tuning only q/v is usually sufficient; in domain knowledge injection tasks, simultaneously fine-tuning MLP yields better results, but memory usage and overfitting risks need to be balanced.

</details>

**11. When training LoRA, which hyperparameters do you usually tune? What are their respective roles?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]

	\caption{Common Hyperparameters for LoRA}


| **Hyperparameter** | **Role** | **Common Values** |
| --- | --- | --- |
| r (Rank) | Rank of low-rank matrices, determining the number of trainable parameters; larger r increases expressiveness, but increases memory usage and overfitting risk | 4, 8, 16, 32, 64 |
| lora_alpha | Scaling factor numerator, with actual scaling factor as alpha/r, controlling the magnitude of LoRA updates on original weights | Typically 1--2$\times$ of r |
| lora_dropout | Adds dropout in LoRA layers to prevent overfitting | 0.05--0.1 (small datasets) |
| target_modules | Specifies modules to apply LoRA (e.g., q_proj/v_proj), affecting performance and GPU memory | Depends on model architecture |
| Learning Rate (lr) | LoRA parameters are initialized small, usually requiring a higher learning rate than full fine-tuning | 1e-4 ~{} 5e-4 |
| Training Epochs | LoRA overfits relatively quickly; should not be too large when data volume is small | 1--3 epochs |


\end{table}

</details>

**12. Does LoRA increase latency during inference? Why?**

<details>
<summary>💡 Show Answer</summary>

**Standard LoRA inference does not increase latency**---provided that "weight merging" is performed.

**Principle**: Before inference, A and B are merged into the original weights: **W = W$_0$ + $\alpha$/r $\cdot$ BA**. After merging, the model architecture is identical to the original model, incurring zero additional computational overhead during inference.

**Under what circumstances will it increase latency?**

- **Unmerged (Dynamic Loading)**: Multi-LoRA switching scenarios (such as vLLM's multi-LoRA serving), where A/B matrices are retained and BAx is computed dynamically during inference, introducing an additional latency of approximately 5--15%;
- **Quantized Scenarios**: In QLoRA where original weights are quantized (4-bit NF4), merging requires dequantization before merging, which may cause a slight loss in precision;
- **Ultra-large Rank**: When r is extremely large, computation of BAx increases, though the impact is negligible when r $\leq$ 64.

**Conclusion**: In production deployment, weights are typically merged first before export, resulting in zero inference overhead.

</details>

**13. How to merge weights after PEFT training is completed?**

<details>
<summary>💡 Show Answer</summary>

![PEFT Weight Merging and Zero-Latency Production Deployment Pipeline](https://img.pricenode.uk/llm_interview_questions/english/chapter4/peft_weight_merging_pipeline.png)




After PEFT training is completed, the weight merging process differs depending on the structural characteristics of each method:

**1. Merging for LoRA / QLoRA**

The update for LoRA is ΔW = (alpha/r)·B·A, which has an **additive relationship** with the original frozen weights W₀ and shares the exact same dimensions. Therefore, it can be directly computed as:

W_merged = W₀ + (alpha/r)·B·A

Merging steps:
- Load the original base model weights W₀ (if using QLoRA, quantized weights must first be dequantized back to floating-point precision);
- Load the trained LoRA weights A and B;
- For each weight matrix with LoRA applied, calculate ΔW = (alpha/r)·B·A and add it to W₀ to obtain W_merged;
- Replace the corresponding original weights in the model with W_merged, yielding a standard model weight file whose **architecture is identical to the original model**;
- Mainstream frameworks (such as HuggingFace PEFT library) typically provide convenient interfaces like `merge_and_unload()` to perform this process.

Note: In QLoRA, if the base model is 4-bit quantized, merging usually requires first dequantizing the base weights to higher precision (such as FP16/BF16) before adding them to the LoRA weights.

**2. "Merging" for Prompt Tuning / Prefix Tuning / P-Tuning v2**

These methods essentially introduce **newly added continuous vector parameters** rather than additive updates to original weight matrices. Thus, **there is no operation of "merging into original weight matrices"**. During deployment, the trained prompt vectors must be saved and loaded alongside original model weights.

**3. Merging for Adapters**

Adapter modules are **serially inserted** sub-modules in the model architecture and do not share a simple additive relationship with original model weights. Therefore, they usually **cannot** be merged into a single weight file with identical architecture to the original model like LoRA does. During deployment, the structure and weights of Adapter modules must be retained.

</details>

**14. Do you know other matrix factorization methods? Why doesn't LoRA directly use SVD for decomposition?**

<details>
<summary>💡 Show Answer</summary>

**Common Matrix Factorization Methods**:

- **SVD** (Singular Value Decomposition): W = U $\Sigma$ V$^T$, exact decomposition, widely used in dimensionality reduction and recommendation systems;
- **PCA** (Principal Component Analysis): Essentially performs SVD on the covariance matrix, used for data dimensionality reduction;
- **NMF** (Non-negative Matrix Factorization): Constrains elements to be non-negative, commonly used in text topic modeling;
- **Tucker Decomposition / CP Decomposition**: Tensor decomposition methods used for multidimensional data;
- **Cholesky Decomposition**: Applicable to positive-definite matrices, used in numerical linear algebra.

**Why doesn't LoRA directly use SVD?**

1. **SVD is an "analytical" tool rather than a "learning" tool**: SVD solves for fixed matrices and cannot be updated via backpropagation during training; whereas A and B in LoRA are learnable parameters that continuously optimize with gradients.
2. **Different initialization strategies**: LoRA initializes A randomly (Gaussian distribution) and B to 0, ensuring $\Delta$W = BA = 0 at the beginning of training to avoid interfering with pre-trained weights; initial values obtained from SVD decomposition carry original weight information.
3. **Computational cost**: Performing SVD on large-scale weight matrices (e.g., 4096$\times$4096) is extremely expensive, whereas LoRA has a very small parameter scale and low update cost.

\begin{quote}
	Note: AdaLoRA introduces an SVD-style parameterization (P $\Lambda$ Q$^T$) and dynamically adjusts the rank of each layer, serving as a practical application of SVD principles within the LoRA framework.
\end{quote}

</details>

**15. In what scenarios have you used LoRA? Which specific models did you fine-tune?**

<details>
<summary>💡 Show Answer</summary>

LoRA is widely used in the following scenarios:

- **Vertical Domain Adaptation**: Such as legal, medical, and financial text QA, where base models are typically LLaMA-2/3, Qwen, ChatGLM, Baichuan, etc.;
- **Instruction-Following Fine-Tuning**: Performing SFT on 7B/13B models based on general instruction data (Alpaca, BELLE, etc.) to grant them better dialogue capabilities;
- **Style Transfer and Writing**: Enabling models to learn specific writing styles or brand tones;
- **Multimodal Scenarios**: Injecting LoRA weights into U-Net/attention layers of image generation models like Stable Diffusion and FLUX to generate specific characters or styles.

**Practical Example**: Using LoRA (r=16, alpha=32) on LLaMA-3-8B to fine-tune on a domain knowledge QA dataset, training only Query/Value projection matrices; converged in about 3 hours using 4 A100 GPUs, achieving performance close to full fine-tuning.

</details>

**16. What are the differences between LoRA and full fine-tuning in terms of GPU memory consumption, training speed, and final performance?**

<details>
<summary>💡 Show Answer</summary>

![Trade-off Matrix Comparing Full Fine-Tuning, Standard LoRA, and QLoRA across Memory, Throughput, and Performance](https://img.pricenode.uk/llm_interview_questions/english/chapter4/memory_speed_peft_tradeoff.png)




**① GPU Memory Consumption**: Full fine-tuning needs to store gradients and optimizer states for all parameters (approx. 3$\times$ parameter size under Adam), whereas LoRA only calculates gradients for low-rank matrices A/B while freezing original weights, significantly reducing memory consumption (typically by 60%--80%). Taking a 7B model as an example, full fine-tuning in bf16 requires approx. 80GB+, whereas LoRA can drop it to 16--24GB.

**② Training Speed**: Trainable parameters in LoRA account for only 0.1%--1% of full parameters, leading to smaller backpropagation computation and shorter per-step execution time when combined with gradient checkpointing; if gradient_checkpointing is used, forward recomputation in full fine-tuning significantly increases runtime, making LoRA's speed advantage even more pronounced.

**③ Final Performance**: When sufficient data is available, full fine-tuning has a higher performance ceiling; LoRA offers better generalization with limited data, is less prone to overfitting, and preserves pre-trained knowledge better. For most vertical domain tasks, LoRA's performance differs very little from full fine-tuning (< 1--2%).

</details>

**17. What is the difference in performance between LoRA fine-tuning and full parameter fine-tuning?**

<details>
<summary>💡 Show Answer</summary>

The performance differences between LoRA fine-tuning and Full Fine-Tuning are mainly reflected in the following aspects:

**1. Upper Bound of Task Adaptation**

- For most common downstream tasks (classification, routine instruction tuning, style adjustment, moderate domain adaptation), **the performance of LoRA (with appropriate rank r and target locations) can be very close to full parameter fine-tuning**, with gaps usually within a few percentage points;
- For tasks requiring **drastic changes in model behavior or injecting large amounts of new knowledge**, full parameter fine-tuning theoretically has a higher expressive capacity ceiling and may achieve better results than LoRA.

**2. Risk of Overfitting**

- Due to its small number of trainable parameters, LoRA is generally **less prone to overfitting** than full parameter fine-tuning in **small-data** scenarios, and its performance may even surpass full parameter fine-tuning;
- In scenarios with sufficient data volume, this difference diminishes.

**3. Catastrophic Forgetting**

- Because original model parameters are completely frozen, LoRA causes **less damage to the model's original capabilities**, making forgetting relatively mild in multi-task/continual learning scenarios;
- Full parameter fine-tuning is more likely to cause degradation in model capabilities on unaddressed tasks/domains.

**4. Training Costs and Resource Requirements**

- LoRA significantly reduces GPU memory requirements, speeds up training, and enables fine-tuning in resource-constrained environments;
- Full parameter fine-tuning requires maintaining gradients and optimizer states for all parameters, resulting in far higher resource demands.

**5. Deployment and Maintenance**

- LoRA training artifacts are very small in size, making it easy to store and manage adapters for multiple tasks/versions and switch flexibly;
- Full parameter fine-tuning requires saving a full copy of model weights for each task/version, incurring high storage and management costs.

**Summary**: In **most mainstream scenarios**, LoRA has become the de facto default choice due to its comprehensive advantages of "comparable performance, low resource requirements, lower susceptibility to overfitting/forgetting, and flexible deployment"; only in scenarios where **the task distribution deviates hugely from pre-training, massive new knowledge needs to be injected, and sufficient data and compute support are available** will full parameter fine-tuning potentially demonstrate significantly superior results over LoRA.

</details>

**18. Calculate the number of trainable parameters when using LoRA with a rank of 8 on a model that has 96 Transformer blocks, with each block containing 12288 × 12288 weight matrices?**

<details>
<summary>💡 Show Answer</summary>

**Parameter count formula for a single LoRA module**:

For a d×k weight matrix (here d=k=12288), LoRA with rank r introduces two matrices:
- A: Dimensions r×k = 8×12288
- B: Dimensions d×r = 12288×8

The parameter count for a single LoRA module is:

$r \times k + d \times r = r \times (d+k) = 8 \times (12288+12288) = 8 \times 24576 = **196,608**$

**Case 1: Only 1 such weight matrix per Transformer block applies LoRA**

Total parameters = 196,608 × 96 = **18,874,368 ≈ 18.87 million**

**Case 2: LoRA applied to Wq and Wv (most common default setting in the LoRA paper)**

LoRA parameters per block = 196,608 × 2 = 393,216

Total parameters = 393,216 × 96 = **37,748,736 ≈ 37.75 million**

**Case 3: LoRA applied to Wq, Wk, Wv, Wo all together**

LoRA parameters per block = 196,608 × 4 = 786,432

Total parameters = 786,432 × 96 = **75,497,472 ≈ 75.50 million**

**Summary**: The core calculation formula is **Total parameters = r × (d + k) × (number of matrices applying LoRA per block) × block count**, demonstrating LoRA's massive advantage in parameter efficiency.

</details>

**19. If you discover severe "catastrophic forgetting" during LoRA fine-tuning, how would you troubleshoot and resolve it?**

<details>
<summary>💡 Show Answer</summary>

![Systematic Troubleshooting and Mitigation Framework for Catastrophic Forgetting in LoRA Fine-Tuning](https://img.pricenode.uk/llm_interview_questions/english/chapter4/catastrophic_forgetting_mitigation.png)




**Troubleshooting Directions**:

- Whether the training data is too homogeneous and deviates excessively from the pre-training data distribution;
- Learning rate is too high, making LoRA parameter update step sizes too large;
- Rank r is set too high, causing LoRA to degenerate into something close to full fine-tuning;
- Training epochs are too high, leading to severe overfitting on fine-tuning data;
- target_modules coverage is too broad; the more layers fine-tuned, the more severe the forgetting.

**Solutions**:

\begin{table}[h]

	\caption{Solutions for Catastrophic Forgetting}


| **Method** | **Description** |
| --- | --- |
| Data Replay | Mix in general domain data (e.g., SlimPajama), training with domain data at a 1:3~1:5 ratio |
| Reduce Learning Rate | Decrease from 2e-4 to 5e-5, combined with warmup schedule |
| Reduce r | Reduce from 32 down to 8 or 4, shrinking the modifiable parameter space |
| EWC Regularization | Add penalty terms in the loss function to constrain modification magnitude of important parameters |
| Narrow target_modules | Fine-tune only q_proj/v_proj, keeping MLP weights unchanged |
| Early Stopping | Stop training when general evaluation metrics on validation set start to drop |


\end{table}

</details>

**20. What are LoRA and QLoRA?**

<details>
<summary>💡 Show Answer</summary>

**LoRA (Low-Rank Adaptation)** is a fine-tuning method that adapts pre-trained language models to new tasks by adding a small number of extra parameters. Instead of updating all parameters of the model, it reduces memory and computational costs during fine-tuning by introducing low-rank matrices. Its advantage lies in adapting efficiently without needing to retrain the entire model.

**QLoRA (Quantized LoRA)** is a variant of LoRA that further reduces memory footprint and computational requirements by quantizing model weights (for example, using 4-bit quantization). It is particularly suitable for running large models on resource-constrained hardware.

</details>

**21. Are you familiar with QLoRA? How does it differ from standard LoRA?**

<details>
<summary>💡 Show Answer</summary>

![QLoRA Architectural Innovations: NF4 Quantization, Double Quantization, Paged Optimizers, and BF16 Adapters](https://img.pricenode.uk/llm_interview_questions/english/chapter4/qlora_architecture_components.png)




QLoRA (Quantized LoRA, Dettmers et al. 2023) is a quantization-enhanced version of LoRA, with three core contributions:

1. **NF4 Quantization (4-bit NormalFloat)**: Quantizes the frozen base model weights into 4-bit using an information-theoretically optimal quantization data type for normally distributed data, incurring smaller precision loss compared to INT4;
2. **Double Quantization**: Quantizes the quantization constants (scale factors) a second time, saving an additional ~0.37 bits per parameter;
3. **Paged Optimizers**: Utilizes NVIDIA Unified Memory to page optimizer states to CPU memory during GPU memory spikes, avoiding OOM (Out Of Memory) errors.

**Differences from Standard LoRA**:

\begin{table}[h]

	\caption{Comparison between QLoRA and Standard LoRA}


| **Dimension** | **Standard LoRA** | **QLoRA** |
| --- | --- | --- |
| Base Model Precision | bf16/fp16 | 4-bit NF4 |
| GPU Memory Demand | Relatively high (e.g., 65B requires multi-GPU) | Extremely low (65B fine-tunable on a single 24GB GPU) |
| Training Speed | Fast | ~20--30% slower (dequantization overhead) |
| Precision Loss | None | Minimal, performance matches full-precision LoRA |
| Inference Deployment | Zero latency overhead after merging | Requires dequantization to merge or using a quantized inference engine |


\end{table}

</details>

**22. What improvements does QLoRA make compared to LoRA?**

<details>
<summary>💡 Show Answer</summary>

**QLoRA** introduces quantization techniques on top of LoRA to further reduce GPU memory footprint:
- Quantizes the base model into **4-bit NormalFloat (NF4)** precision for storage, reducing GPU memory by ~4$\times$
- Introduces **Double Quantization**, which quantizes the quantization constants a second time to save additional memory
- Uses **Paged Optimizers** to prevent GPU memory spikes from causing memory overflow
- LoRA adapters are still trained in BF16 precision to ensure training accuracy
Effect: Makes fine-tuning 65B models on a single consumer GPU (such as RTX 3090) possible.

</details>

**23. What is the principle behind combining quantization and LoRA in QLoRA?**

<details>
<summary>💡 Show Answer</summary>

The core idea of QLoRA (Quantized LoRA) is: **quantize the pre-trained model weights into extremely low bits (such as 4-bit) to drastically reduce GPU memory footprint, while adding LoRA adapters on top of the quantized model for fine-tuning**, allowing ultra-large-scale models to be fine-tuned on a single card with limited GPU memory. Its key technical components include:

**1. 4-bit NormalFloat (NF4) Quantization**

- QLoRA proposes NF4, a 4-bit data type specifically designed for neural network weights (which typically follow a zero-mean normal distribution). Compared to ordinary 4-bit integer quantization, the distribution of NF4 quantization bins aligns better with the actual value distribution of normally distributed weights, preserving more information and reducing quantization errors at the same bit depth.

**2. Double Quantization**

- The quantization process itself requires storing a scale factor for each quantization block, typically stored in FP32; QLoRA performs **a second quantization on these scale factors** (e.g., quantizing them to 8-bit), further compressing memory overhead.

**3. Paged Optimizers**

- Leverages NVIDIA Unified Memory to automatically **page** optimizer states and other data to CPU memory when GPU memory is insufficient, preventing OOM caused by momentary GPU memory spikes during training.

**4. Quantized Model + LoRA Training Workflow**

- **Base model weights**: Quantized into 4-bit NF4 format and frozen;
- **LoRA adapters**: Added on top of the quantized weights in a higher precision (such as BF16), where only LoRA's A and B matrices are trainable parameters;
- **Forward pass**: The 4-bit quantized weights are "dequantized" back to higher precision for matrix multiplication, while the LoRA computation proceeds normally in BF16;
- **Backward pass**: Gradients are backpropagated only to LoRA's A and B matrices (quantized base weights do not require gradient computation), which is the core source of GPU memory savings.

</details>

**24. How does block-wise quantization in QLoRA resolve the information loss problem caused by ordinary quantization?**

<details>
<summary>💡 Show Answer</summary>

**The Information Loss Problem of Ordinary (Global) Quantization**:

Naive quantization methods usually apply a **unified scale factor** across the entire weight tensor. The problem with this "global quantization" is that if there are **a small number of "outliers" with exceptionally large magnitudes** in the weight tensor, the global scale factor will be stretched by these outliers. As a result, the majority of weights (which have smaller magnitudes) are mapped into very dense, low-resolution regions of quantization levels after quantization, leading to a significant increase in quantization error.

**Solution of Block-wise Quantization**:

QLoRA adopts **block-wise quantization**: splitting the weight tensor into **multiple smaller blocks (e.g., every 64 elements form a block)**, where **each block independently computes its own scale factor** for quantization.

The benefits of this approach are:
- **Restricts the influence of outliers to their own block**, without degrading the quantization precision of other blocks in the tensor;
- The value distribution within most blocks is more "concentrated" (smaller dynamic range), allowing quantization with relatively finer scale factors, thereby significantly reducing overall quantization error;
- Effectively replaces "coarse-grained global uniform scaling" with "finer-grained local adaptive scaling," better accommodating the non-uniform value distributions within weight tensors.

</details>

**25. Many LoRA variants have been proposed by the community since then. Can you pick one and explain what pain point it resolves?**

<details>
<summary>💡 Show Answer</summary>

Take **DoRA (Weight-Decomposed Low-Rank Adaptation, Liu et al. 2024)** as an example:

**Pain Point of LoRA**: Low-rank matrices with a fixed rank can only perform a joint adjustment of "direction + magnitude," which tends to favor "magnitude changes" while neglecting "directional adjustments," underperforming full fine-tuning on certain tasks.

**Core Idea of DoRA**: Decomposes the weight matrix W into two components: a "magnitude" vector m and a "direction" matrix V:

\begin{verbatim}
	W = m · (V / ||V||)
\end{verbatim}

Then uses LoRA to update the direction matrix V: V = W$_0$ + BA

**Advantages**:
- Decouples direction and magnitude, allowing the model to independently adjust "weight magnitude" and "weight direction," leading to stronger expressiveness;
- Significantly outperforms LoRA on multiple tasks, with trainable parameter counts only slightly higher than LoRA;
- Compatible with QLoRA, making it usable in quantized scenarios.

\begin{quote}
	**AdaLoRA Supplement**: AdaLoRA parameterizes LoRA as P $\Sigma$ Q$^T$ (an SVD-like formulation) and dynamically prunes the ranks of different layers based on singular value importance. This addresses the limitation of "using the same rank across all layers," enabling adaptive allocation of parameter budgets.
\end{quote}

</details>

**26. Besides the LoRA family, what other PEFT methods are you familiar with?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]

	\caption{Comparison of PEFT Methods}


| **Method** | **Core Idea** | **Advantages** | **Disadvantages** |
| --- | --- | --- | --- |
| Adapter Tuning | Inserts small bottleneck modules after FFN/Attention | Few parameters, flexible | Cannot be merged during inference, increases latency |
| Prefix Tuning | Prepends trainable prefix vectors before K/V of each Attention layer | Suitable for generation tasks | Poor training stability, requires reparameterization |
| Prompt Tuning | Prepends trainable soft prompt vectors only before the input layer | Extremely simple, performance improves with model size | Poor performance on small models |
| P-Tuning v2 | Extends trainable prompts to all layers | Suitable for NLU, outperforms LoRA on small models | Relatively complex implementation |
| IA$^3$ | Element-wise multiplies K/V and FFN activations with trainable scaling vectors | Extremely small parameter count (0.01%), suitable for few-shot | Limited expressiveness |
| BitFit | Fine-tunes only the bias parameters of the model | Extremely few parameters | Performance cap, narrow applicability to tasks |


\end{table}

**Summary**: The LoRA family has become the most mainstream PEFT solution due to zero additional inference overhead (after merging) and excellent performance; Prefix/Prompt Tuning still hold a place in extremely resource-constrained scenarios or specific generation tasks.

</details>

**27. What is the basic principle of the Adapter architecture?**

<details>
<summary>💡 Show Answer</summary>

Adapter is a class of parameter-efficient fine-tuning methods proposed prior to LoRA. Its core idea is: **insert small, newly added trainable modules (Adapter modules) into each (or selected) Transformer layers of a pre-trained model while keeping original model parameters frozen, training only these newly added modules**.

**Architectural Design**:

A typical Adapter module is usually a **small feed-forward network with a bottleneck structure**:
- First passes through a **down-projection linear layer** (projecting hidden dimension d to a smaller bottleneck dimension m, m << d);
- Passes through a non-linear activation function (such as ReLU/GELU);
- Then passes through an **up-projection linear layer** (projecting dimension from m back to d);
- Usually adds a **residual connection**: Adapter Output = Input + Adapter(Input), allowing the Adapter to be initialized close to an identity mapping.

**Comparison with LoRA**:

- **Structural Differences**: Adapters are new modules **inserted in series** (adding extra layers to the forward computation path), which increases computation path length during inference; LoRA consists of **parallel low-rank branches** that can be merged with the original weights during inference without altering the original model structure or adding inference latency.
- **Inference Efficiency**: LoRA incurs zero extra inference overhead after merging; Adapters retain their serial structure during inference, introducing additional latency.

</details>

**28. What is the principle of Prefix Tuning?**

<details>
<summary>💡 Show Answer</summary>

Prefix Tuning is a parameter-efficient fine-tuning method designed for generation tasks (especially Transformer-based autoregressive or Encoder-Decoder models). Its core idea is: **in the Attention computation of every Transformer layer, prepend a segment of trainable "virtual prefixes" (prefix) to the Key and Value matrices, while keeping original model parameters frozen and training only these prefix vectors**.

**Detailed Principles**:

1. **Instead of prepending real tokens to the input text**, a segment of **trainable continuous vectors (prefix vectors) with a fixed length (e.g., p positions)** is prepended before the Key and Value matrices in each Attention layer. These vectors do not correspond to any real vocabulary tokens, but are "virtual token representations" learned directly in the continuous vector space.
2. After this modification, Attention computation becomes: query vectors compute attention over "prefix Key vectors + real input Key vectors," allowing every layer's output to be influenced by these trainable prefix vectors. This is equivalent to injecting a "task-specific context/instruction" into each layer of the model via prefix vectors.
3. **Each layer has an independent set of prefix vectors**, enabling prefixes to influence intermediate representations and attention patterns at different model levels in distinct ways, providing stronger steering capability than "adding prompts only at the input layer."
4. **Reparameterization**: Directly optimizing prefix vectors can sometimes be unstable during training; the original paper proposed using a small MLP network to generate the final prefix vectors from a set of lower-dimensional trainable parameters.

**Difference from Traditional Prompts (Discrete Text Prompts)**:

The "prefix" in Prefix Tuning consists of **trainable parameters in a continuous vector space**, rather than discrete tokens existing in the vocabulary, hence termed a "soft prompt." This allows optimization to be performed directly via gradient descent.

</details>

**29. What is the principle of Prompt Tuning?**

<details>
<summary>💡 Show Answer</summary>

Prompt Tuning is a simplified soft prompting method compared to Prefix Tuning. Its core idea is: **prepend a segment of trainable continuous vectors (soft prompt) only before the model's input embedding layer, while keeping all other model parameters (including all Transformer layers) completely frozen**.

**Detailed Principles**:

1. Given input text, first convert it into an embedding sequence;
2. At the very beginning of this embedding sequence, prepend a segment of **trainable embedding vectors** of length p, forming a new input sequence: [soft prompt embeddings] + [original input embeddings];
3. Feed the prepended sequence normally into the (completely frozen) Transformer model for forward computation;
4. During training, gradients backpropagate only to these soft prompt vectors, and none of the model's parameters are updated.

**Key Differences from Prefix Tuning**:

- **Application Location**: Prompt Tuning adds trainable vectors only at the **input layer (embedding layer)**; Prefix Tuning adds independent trainable prefixes to the **K and V of Attention in every layer**, exerting a direct and independent influence on intermediate computations at every layer.
- **Expressive Capacity**: Prompt Tuning's performance **nears full fine-tuning only when the model scale is sufficiently large (e.g., billions of parameters or more)**; on small and medium models, Prompt Tuning often underperforms Prefix Tuning or full fine-tuning significantly.

</details>

**30. What are the differences among P-Tuning, Prompt Tuning, and Prefix Tuning?**

<details>
<summary>💡 Show Answer</summary>

- **Prefix-Tuning vs. P-Tuning**: Prefix-Tuning prepends additional embeddings at the beginning, resembling instructions, whereas P-Tuning allows flexible positioning; Prefix-Tuning adds trainable parameters to every layer reparameterized by an MLP, whereas P-Tuning adds embeddings only at the input layer reparameterized by an LSTM or MLP.
- **Prefix-Tuning vs. Prompt Tuning**: Prompt Tuning can be viewed as a simplified version of Prefix Tuning, adding prompt tokens only at the input layer without requiring MLP reparameterization to address training instability.
- **P-Tuning vs. Prompt Tuning**: P-Tuning adds embeddings at the input layer reparameterized via LSTM or MLP with flexible prompt placement; Prompt Tuning adds virtual tokens at the input layer directly for learning without complex parameterization architectures.

</details>

**31. What is the principle of P-Tuning v2?**

<details>
<summary>💡 Show Answer</summary>

P-Tuning v2 can be seen as an improved method that **systematizes the idea of Prefix Tuning and applies it to a broader range of tasks (especially NLU tasks) and smaller-scale models**, aiming to solve the issue where Prompt Tuning performs poorly on small/medium models and complex NLU tasks.

**Core Improvements**:

1. **Adds trainable continuous prompts to every layer (similar to Prefix Tuning)**: P-Tuning v2 adds independent trainable continuous vectors to the **inputs (or K, V) of every Transformer layer**, allowing every model layer to be directly controlled by task-specific signals. This significantly enhances expressive capacity, enabling it to achieve performance close to full fine-tuning even on small and medium models.
2. **Removes reparameterization encoders**: P-Tuning v2 found that directly optimizing continuous prompt vectors yields good performance, simplifying the structure.
3. **Adopts task-specific output layer designs for different tasks**: P-Tuning v2 returns to a more traditional approach: **adding corresponding task-specific output layers (such as linear classification heads) for different tasks (classification, sequence labeling, etc.)**, enabling P-Tuning v2 to naturally adapt to various NLU task formats (including token-level tasks like sequence labeling).

**Relationship with Prefix Tuning**:

P-Tuning v2 is structurally very similar to Prefix Tuning and can be considered a systematic validation and engineering enhancement of the Prefix Tuning concept on NLU tasks. The paper's main contribution lies in demonstrating that the "deep prompt" paradigm (prompts in every layer) consistently achieves performance near full fine-tuning across **models of all scales and various NLU tasks**.

</details>

**32. How do common PEFT methods compare in terms of pros and cons?**

<details>
<summary>💡 Show Answer</summary>

| Method | Core Idea | Parameter Count | Inference Overhead | Advantages | Disadvantages |
| --- | --- | --- | --- | --- | --- |
| **LoRA** | Approximates weight update ΔW with low-rank matrices B·A, added in parallel alongside original weights | Relatively small (depends on rank r and number of applied target modules) | Mergeable, **zero extra inference overhead** | Performance close to full fine-tuning; no inference latency; flexible multi-task switching | Requires selecting r and target modules; may still have a performance upper bound on extremely complex capability transfers |
| **QLoRA** | LoRA + base model 4-bit quantization | Similar to LoRA (trainable parameters) | Requires dequantization during inference, slightly increasing computation while drastically reducing memory | Extremely low GPU memory requirement, allows fine-tuning ultra-large models on a single GPU | Quantization introduces minor precision loss; relatively complex implementation |
| **Adapter** | Inserts small bottleneck FFN modules inside Transformer layers | Relatively small (depends on bottleneck dimension and number of insertion locations) | **Serial structure, increases inference latency** | Flexible insertion, good multi-task isolation | Extra inference latency, cannot be seamlessly merged like LoRA |
| **Prefix Tuning** | Prepends trainable prefix vectors before K and V of Attention in each layer | Moderate (depends on layer count and prefix length) | Occupies part of effective context length, adds minor computation | Effective for generation tasks; independent control per layer | Prefix length requires tuning; consumes context window space |
| **Prompt Tuning** | Prepends trainable vectors only at input embedding layer; model is completely frozen | **Smallest** (depends only on prompt length and hidden size) | Virtually no extra overhead (only increases sequence length) | Extremely small parameter count, lightweight multi-task switching | Performance is sensitive to model scale, poor on small models |
| **P-Tuning v2** | Adds trainable continuous prompts to every layer (deep prompt) + task-specific output layers | Moderate | Similar to Prefix Tuning | Stable performance across model scales and NLU tasks, close to full fine-tuning | Parameter count larger than Prompt Tuning; relatively complex structure |



**Comprehensive Comparison Dimensions**:

1. **Performance Upper Bound**: Full Fine-Tuning > LoRA (close to full fine-tuning under appropriate r and module locations) $\approx$ P-Tuning v2 $\approx$ Prefix Tuning > Adapter $\approx$ Prompt Tuning (on small models);
2. **Inference Efficiency**: LoRA (zero overhead after merging) > QLoRA (some overhead due to quantization/dequantization) > Prompt Tuning (only adds a few tokens) > Prefix Tuning/P-Tuning v2 (occupies context + minor computation) > Adapter (serial structure, most noticeable latency);
3. **GPU Memory Efficiency (Training)**: QLoRA (best, base model quantized) > LoRA > Prompt Tuning/Prefix Tuning > Adapter (depends on bottleneck size);
4. **Implementation Complexity & Ecosystem Support**: LoRA, due to its simple structure and mergeability, has the most comprehensive support in mainstream frameworks, serving as the current de facto "default choice."

**Summary**: **LoRA (and its quantized variant QLoRA), by combining the dual advantages of "performance close to full fine-tuning" and "zero extra inference overhead," alongside mature toolchain support, stands as the most mainstream and practical PEFT method today**.

</details>

**33. What empirical insights exist regarding lightweight fine-tuning (e.g., LoRA)?**

<details>
<summary>💡 Show Answer</summary>

Lightweight fine-tuning methods are generally not recommended; renting more compute resources for full parameter fine-tuning is preferred. Although LoRA can be effective in specific domains, it may cap the model's potential performance ceiling and introduce new issues.

</details>

**34. What factors affect memory usage?**

<details>
<summary>💡 Show Answer</summary>

Memory usage is influenced by model size, batch size, number of LoRA parameters, and dataset characteristics. For example, using shorter training sequences can save memory.

% ============================================================

</details>

