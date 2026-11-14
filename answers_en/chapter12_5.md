# 12.5 Multimodal Training and Alignment — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 20 questions

**1. What training objectives are commonly used in multimodal pre-training?**

<details>
<summary>💡 Show Answer</summary>

Common training objectives in multimodal pre-training include:

1. **Image-Text Contrastive Learning (ITC)**: Pulls paired image-text representations closer while pushing unpaired ones apart to learn global semantic alignment (CLIP, ALIGN).
2. **Image-Text Matching (ITM)**: A binary classification task determining whether a given image-text pair matches, learning fine-grained image-text consistency (BLIP, ALBEF).
3. **Image-Text Generation (ITG / LM, Language Modeling)**: Given an image, autoregressively generates the corresponding textual description to train generative capabilities (BLIP, Flamingo).
4. **Masked Language Modeling (MLM)**: Masks certain words in text and predicts them in conjunction with image context, forcing the model to leverage image information to understand text.
5. **Masked Image Modeling (MIM)**: Masks certain image patches to predict pixels or features of the masked regions (MAE, BEiT), learning visual representations.
6. **Image Captioning**: A standard seq2seq objective mapping images to text, serving as the core objective of generative multimodal models.
7. **Visual Question Answering (VQA) Objective**: Given an image and a question, generates the answer, serving either as a supervised fine-tuning objective or one of the pre-training objectives.

</details>

**2. What is the objective of image-text contrastive learning?**

<details>
<summary>💡 Show Answer</summary>

The objective of image-text contrastive learning is to **pull semantically corresponding image-text pairs closer in the embedding space while pushing non-corresponding pairs apart**, thereby learning cross-modal semantic alignment.

**Formal Description:**
For $N$ image-text pairs $\{(v_i, t_i)\}$ within a batch, the image-text similarity matrix $S_{ij} = sim(v_i, t_j) / \tau$ is computed ($\tau$ is the temperature parameter), using a symmetric InfoNCE loss:

$$\mathcal{L} = -\frac{1}{2N}\sum_{i=1}^{N}\left[\log\frac{e^{S_{ii}}}{\sum_j e^{S_{ij}}} + \log\frac{e^{S_{ii}}}{\sum_j e^{S_{ji}}}\right]$$

**Key Elements:**
- **Large Batch Size**: The greater the number of negative samples, the better the performance; CLIP uses an ultra-large batch size of 32768;
- **Temperature parameter $\tau$**: A learnable parameter controlling the sharpness of the distribution;
- **Hard Negative Samples**: Negative samples within the same batch may be semantically close (the false negative problem), requiring denoising treatment (e.g., ALBEF uses a Momentum Encoder to filter them);
- **Momentum Encoder**: Uses a momentum-updated encoder to maintain a larger feature queue, providing more negative samples (MoCo style).

</details>

**3. What is the basic principle of image-text generative models?**

<details>
<summary>💡 Show Answer</summary>

Image-text generative models are divided into two categories:

**Text Generation (generating text given an image):**
- The image passes through an encoder to produce a visual token sequence;
- Visual tokens are concatenated with text tokens and fed into an autoregressive language model (or injected via Cross-Attention);
- The language model step-by-step generates textual output via Next Token Prediction;
- Representative models: LLaVA, InstructBLIP, Flamingo, GPT-4V.

**Image Generation (generating images given text):**
- Text passes through a text encoder (such as CLIP text encoder) to obtain condition vectors;
- A diffusion model (U-Net or DiT) iteratively denoises a noisy image, guided by text conditions (Classifier-Free Guidance);
- Representative models: Stable Diffusion, DALL-E 3, Imagen.
- There are also autoregressive image token generation approaches (such as VQVAE + GPT, represented by DALL-E 1).

</details>

**4. What stages are typically involved in the training of multimodal large models?**

<details>
<summary>💡 Show Answer</summary>

Taking the classic two-stage paradigm as an example, the training of multimodal large models is typically divided into:

1. **Modality Alignment Pre-training Stage**: In this stage, the two fully pre-trained modules—the visual encoder and the language model—are completely frozen, and only the randomly initialized connector module (such as a linear projection layer or Q-Former) is trained. Using a large-scale (potentially reaching hundreds of millions or more) but relatively noisy image-text pair dataset (such as web-scraped images with simple accompanying descriptions), the goal is to enable the connector module to learn a basic, rough mapping relationship so that visual features can align into a representation space understandable by the language model. This stage can be viewed as teaching the model to first "learn to translate" visual signals.
2. **Multimodal Instruction Tuning Stage**: In this stage, the model is fine-tuned on a much smaller but far higher-quality dataset in instruction format (each entry containing an image, specific instructions, and expected responses)—usually unfreezing the language model (possibly paired with parameter-efficient methods like LoRA to control training costs), and sometimes unfreezing the visual encoder as well. The goal of this stage is to enable the model to understand and follow complex natural language instructions involving visual information, acquiring capabilities to execute specific tasks such as description, question answering, and reasoning, which can be viewed as teaching the model to "learn to perform tasks."
More advanced models (such as InternVL, Qwen-VL) often insert an intermediate stage of "continued pre-training on high-quality data" between these two stages, forming the aforementioned three-stage training paradigm to achieve superior final performance by progressively elevating data quality.

</details>

**5. Why is it often necessary to "freeze" certain modules (such as the visual encoder) during multimodal model training?**

<details>
<summary>💡 Show Answer</summary>

Freezing certain modules (especially the thoroughly pre-trained visual encoder and language model) is mainly driven by the following considerations:

- **Protect existing capabilities and avoid catastrophic forgetting**: The visual encoder and language model have both acquired high-quality representation capabilities through long-term, compute-intensive training on massive single-modality datasets. In contrast, the data volume used in the multimodal alignment phase is relatively limited. If the parameters of these pre-trained modules are updated freely, the model can easily destroy the well-learned single-modality capabilities before acquiring new multimodal capabilities (i.e., catastrophic forgetting);
- **Reduce training costs**: Freezing means these parameters do not require gradient calculations or storage of corresponding optimizer states during training (e.g., the Adam optimizer requires storing additional first- and second-order momentum for each trainable parameter). This significantly reduces the number of parameters actively participating in updates, thereby substantially lowering GPU memory usage and computational overhead required for training, enabling multimodal alignment training within a limited compute budget;
- **Allow newly added modules to focus on learning the "translation" task**: Freezing both ends while training only the intermediate connector module clarifies the training objective—allowing the connector module to focus on learning the focused task of "how to correctly map visual features into a space understandable by the language model," making the training objective clearer and easier to converge.
Of course, freezing is not kept unchanged throughout the entire training process. As mentioned earlier in the progressive unfreezing strategy, practice typically involves gradually unfreezing more parameters in later training stages based on data scale and performance requirements to further enhance model capabilities.



![多模态预训练阶段中的模块冻结与激活状态结构图 (Module freezing configuration during two-stage VLM pretraining.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig16_module_freezing.png)

</details>

**6. How is Multimodal Instruction Tuning (Visual Instruction Tuning) Performed? Why is it a Critical Step?**

<details>
<summary>💡 Show Answer</summary>

Multimodal Instruction Tuning (Visual Instruction Tuning) aims to enable models to understand and follow natural language instructions containing images, serving as a bridge connecting the foundational capabilities of VLMs with their application capabilities.

**Data Construction:**
- Each data item is formatted as `<image> + <instruction> → <response>`;
- Existing image annotation data (object bounding boxes, captions) are usually combined with powerful language models (such as GPT-4) to automatically generate diverse instruction-response pairs, supplemented by a certain proportion of human annotation/verification data to ensure quality (e.g., LLaVA-Instruct);
- Data sources also include: human annotations, multi-task mixtures (VQA, description, reasoning, etc.), and rewrites of academic datasets.

**Training Flow (Taking LLaVA as an Example):**
1. Pre-training Alignment Stage: Freeze the image encoder and language model, train only the projection layer, and perform caption pre-training using large-scale image-text pairs;
2. Instruction Tuning Stage: Unfreeze the language model (the image encoder can optionally remain frozen), and perform full or LoRA fine-tuning on instruction data.

**Key Techniques:** Dialogue format alignment (reusing templates such as Vicuna, LLaMA-Chat, etc.); including samples in training data that refuse to answer inappropriate requests; mixing multi-type task data such as VQA, OCR, dialogue, and reasoning to enhance generalization capabilities.

**Why It Is Critical:** Traditional VQA/captioning models can only perform specific tasks they were trained on, whereas instruction tuning grants models the generalization capability to understand arbitrary instruction intents; it genuinely teaches LLMs how to utilize visual information to complete reasoning, follow complex instructions, and engage in multi-turn dialogues, combining LLMs' inherent abilities such as common-sense reasoning and creative writing with visual inputs; at the same time, it aligns the model's output format and interaction mode more closely with human expectations, marking a decisive step for the model from "usable" to "useful".

</details>

**7. How to Design a Multimodal Dialogue Model?**

<details>
<summary>💡 Show Answer</summary>

Multimodal dialogue models need to support multi-turn interleaved image-text dialogues:

**Architecture Design:**
- The base architecture is an autoregressive language model with image inputs (such as LLaVA, InstructBLIP, Qwen-VL);
- When an image appears in a particular turn of dialogue, all subsequent turns can reuse the visual tokens cached in the KV Cache without re-encoding;
- For multi-image inputs, each image is encoded separately and inserted sequentially into the corresponding position of the token sequence.

**Dialogue State Management:**
- Standard multi-turn dialogue templates are used, organized in an alternating `[USER]: ... [ASSISTANT]: ...` format;
- All historical dialogues are concatenated into the context window, allowing the model to naturally achieve multi-turn memory;
- Truncation strategies are required when exceeding context length (prioritizing the retention of image tokens and recent turns).

**Instruction Alignment:**
- Preference alignment is performed on multimodal outputs via RLHF or DPO to reduce hallucinations and inappropriate responses;
- Training data needs to cover various dialogue scenarios: image descriptions, detail questioning, reasoning, comparative analysis, etc.

</details>

**8. What is LoRA? How is it Applied in Multimodal Model Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

LoRA (Low-Rank Adaptation) is a parameter-efficient fine-tuning technique. Its core idea is not to directly update the model's original large weight matrix $W$ (keeping it completely frozen), but to add a pair of low-rank matrices $A$ and $B$ alongside it that are far smaller in size than the original matrix (the shape of $W$ is $d \times k$, while $A$ is $d \times r$ and $B$ is $r \times k$, where the rank $r$ is much smaller than $d$ and $k$). During training, only these two newly added low-rank matrices are updated, and the actual calculation of the final model becomes $W + BA$, i.e., superimposing a low-rank "correction term" on top of the original output. Because $r$ is usually very small (e.g., 8, 16, 64), the number of trainable parameters added can be reduced by several orders of magnitude compared to the original weight matrix, thereby drastically reducing GPU memory required for fine-tuning (especially optimizer state memory consumption) as well as training costs.

In multimodal model fine-tuning, LoRA can be flexibly applied to different components: the most common approach is to insert LoRA only into the Attention layers (Query, Key, Value projection matrices) of the **language model** component, which is also the most mainstream parameter-efficient scheme during the instruction tuning stage; if there is a need to enhance the model's adaptability to images in specific domains, LoRA can also be selectively inserted into the **visual encoder** component; LoRA can even be applied to the connector module. This flexibility of "inserting wherever adaptation is needed" makes LoRA an important tool for multimodal models to perform task/domain adaptation under resource-constrained scenarios.

</details>

**9. How to Design Reinforcement Learning Alignment for Multimodal Models (e.g., Preference Optimization based on Human Feedback)? What is Unique Compared to Pure Text Scenarios?**

<details>
<summary>💡 Show Answer</summary>

The basic workflow of reinforcement learning alignment for multimodal models is similar to that of pure text scenarios: first, multiple candidate responses are collected for the same input (in multimodal scenarios, an "image + question"), and human annotators (or a stronger model) annotate which response is better, forming preference comparison data; subsequently, one can either train an independent reward model based on this preference data and combine it with reinforcement learning algorithms (such as PPO) to optimize the policy model, or adopt the simpler Direct Preference Optimization (DPO) method to skip training an explicit reward model and directly optimize model parameters based on the preference comparison data.

Compared to pure text scenarios, multimodal reinforcement learning alignment exhibits the following unique aspects:

- **Preference judgment requires considering image-text consistency simultaneously**: When determining "which response is better," annotators cannot judge solely based on text-only dimensions such as language fluency and logic; they must also carefully verify the actual content of the image to determine whether the specific details mentioned in the response (objects, colors, quantities, positions, etc.) truly correspond to the content in the image, significantly increasing annotation complexity and cost;
- **Hallucination is a crucial dimension for preference judgment**: Due to the prominent issue of hallucinations in multimodal models, the annotation standards for preference data usually need to explicitly include the dimension of "whether hallucinated content exists." A response that is linguistically fluent but fabricates details not present in the image should be explicitly annotated as inferior to a response that is plainly expressed but completely accurate;
- **Special evaluation is required for "whether visual information is truly utilized"**: Preference annotation sometimes requires additional investigation into whether the model actually performed reasoning and answered based on image content, rather than relying on language priors to guess the correct answer (avoiding the aforementioned "language shortcut" problem from being further reinforced by reinforcement learning).



![多模态场景下的直接偏好优化（DPO）训练流程图 (Direct Preference Optimization (DPO) training pipeline in multimodal settings.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig17_multimodal_rlhf.png)

</details>

**10. What is "Cross-Modal Catastrophic Forgetting" in Multimodal Models? How to Mitigate It in Multi-Stage Training?**

<details>
<summary>💡 Show Answer</summary>

Cross-modal catastrophic forgetting refers to the phenomenon where, if the training data in a certain training stage is overly concentrated on a single modality or a single task type (for example, if fine-tuning data in a stage consists almost entirely of pure text dialogue data, or entirely of a specific category of visual task data), as training continues on this portion of data, the model's capabilities in **other** modalities or tasks that were not heavily trained during this stage may undergo significant and unexpected degradation—for instance, after a round of pure text instruction tuning, the model suddenly performs noticeably worse on image understanding tasks where it previously performed well. The root cause of this phenomenon is consistent with the general catastrophic forgetting issue in neural network training: the gradient update direction of the current stage is primarily aligned with fitting the current stage's data distribution, and this update direction may not necessarily align with—or may even conflict with—the directions of capabilities learned in previous stages.

Common methods to mitigate this issue include:

- **Data Replay**: Actively retain a certain proportion of data from previous stages (or other modalities/tasks) within the training data of each stage to participate in training together, rather than completely switching to an entirely new single data distribution, helping the model "review" existing capabilities while learning new ones to prevent forgetting;
- **Parameter-Efficient Fine-Tuning**: Adopting fine-tuning methods such as LoRA that only update a small number of newly added parameters causes less overall perturbation to the original pre-trained weights compared to full fine-tuning, naturally possessing a degree of anti-forgetting characteristics;
- **Continuous Monitoring of Multi-Dimensional Evaluation Metrics**: During training, one should not solely focus on target metrics prioritized for optimization in the current stage, but also continuously monitor model performance on fine-grained evaluation benchmarks covering different modalities and tasks. Once a noticeable decline is detected in capabilities that previously performed well, data ratios or training strategies need to be adjusted in a timely manner.

</details>

**11. What Are the Methods for Fine-Tuning Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

1. **Full Fine-Tuning**: Updates all parameters, achieving the best performance, but incurs high computational costs; suitable for scenarios with abundant data.
2. **LoRA / QLoRA**: Inserts low-rank matrices into the attention layers of the language model, updating only a small number of parameters; optionally adds LoRA to the image encoder as well; memory-friendly, suitable for consumer-grade GPUs.
3. **Projection Layer Fine-Tuning (Adapter Tuning)**: Fine-tunes only the image-text projection layer (such as MLP, Q-Former), while freezing the image encoder and language model; suitable for fast domain adaptation, but has a lower performance ceiling.
4. **Image Encoder Fine-Tuning**: Unfreezes the image encoder to participate in training, enhancing feature extraction capabilities for domain-specific images (such as medical images, remote sensing images); usually requires a smaller learning rate.
5. **Instruction Tuning**: Performs supervised fine-tuning on instruction-formatted data to improve the model's ability to follow instructions; currently the most mainstream multimodal fine-tuning method.
6. **RLHF / DPO**: Performs reinforcement learning or direct preference optimization based on human preference data to reduce hallucinations and improve response quality and safety.
7. **Comparison of Parameter-Efficient Methods**: LoRA < Adapter < Full Fine-Tuning (increasing trainable parameter count), but as data volume increases, full fine-tuning performance gradually surpasses parameter-efficient methods.

</details>

**12. How is Distributed Training Performed for Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

**Data Parallelism:** Each GPU holds a complete replica of the model and processes different data batches, implemented using DDP or FSDP; suitable for cases where the model fits into a single GPU.

**Model Parallelism:** Tensor Parallelism (splitting matrix operations across multiple GPUs, e.g., Megatron-LM); Pipeline Parallelism (assigning different model layers to different GPUs).

**ZeRO Optimization (DeepSpeed):** ZeRO-1 shards optimizer states; ZeRO-2 additionally shards gradients; ZeRO-3 shards model parameters; combined with CPU Offload, GPU memory requirements can be further reduced.

**Multimodal Special Considerations:** The image encoder and language model can be placed on different GPUs for heterogeneous parallelism; image preprocessing (decoding, resizing) is recommended to be completed asynchronously on the CPU side to avoid becoming a bottleneck; pay attention to CPU-GPU data transfer bandwidth during large batch training.

**Training Framework Selection:** DeepSpeed + HuggingFace Accelerate (most mainstream in industry); Megatron-LM (optimal efficiency for large-scale training but complex to use); PyTorch FSDP (official native support).

</details>

**13. How to Optimize GPU Memory in Multimodal Training?**

<details>
<summary>💡 Show Answer</summary>

1. **Freezing Partial Modules**: Freeze the image encoder and language model during the pre-training alignment stage, training only the projection layer.
2. **Gradient Checkpointing**: Avoid caching intermediate activation values and recompute them during backward passes, trading computation time for memory space, reducing memory usage by approximately 40-50%.
3. **LoRA Fine-Tuning**: Train only a small number of parameters, drastically reducing optimizer state memory consumption.
4. **Mixed-Precision Training (BF16/FP16)**: Reduces GPU memory by half compared to FP32, paired with Loss Scaling to maintain numerical stability.
5. **Image Token Compression**: Reduce the number of tokens per image (Pooling, Q-Former), directly decreasing Attention computation and KV Cache memory footprint.
6. **ZeRO Distributed Optimization (DeepSpeed)**: Shard optimizer states, gradients, and model parameters across multiple GPUs/nodes.
7. **FlashAttention**: IO-aware efficient Attention implementation, significantly reducing peak Attention GPU memory consumption.

</details>

**14. What is the Difference Between "Two-Stage" and "Three-Stage" Paradigms in Multimodal Training?**

<details>
<summary>💡 Show Answer</summary>

**Two-Stage Paradigm** (represented by classic LLaVA):
1. **Stage 1 — Modality Alignment Pre-training**: Freeze the visual encoder and language model, train only the randomly initialized connector (linear projection layer), using large-scale but relatively coarse image-text pair data (such as web-scraped caption data), allowing the connector to learn a basic and rough mapping from visual to language representation space;
2. **Stage 2 — Visual Instruction Tuning**: Unfreeze the language model (usually combined with LoRA), using smaller-scale but higher-quality instruction-formatted data (image + instruction + response), training the model to understand and follow complex instructions containing visual information.

**Three-Stage Paradigm** (commonly adopted by newer models such as InternVL and Qwen-VL) inserts an intermediate "deep pre-training / continued pre-training" stage between the two stages. Specifically, the three stages become: (1) Large-scale coarse data alignment (similar to Stage 1 of the two-stage paradigm, pursuing data scale); (2) **Continued pre-training with medium-scale high-quality data**—unfreezing more parameters (such as the visual encoder itself), using cleaned, higher-quality image-text data to further strengthen alignment performance and visual detail representation capabilities; (3) Small-scale, highly meticulously annotated instruction tuning data for final capability alignment. This design of "gradually increasing data quality stage by stage, and progressively expanding model unfreezing scope stage by stage" is essentially a curriculum learning approach. Compared to the two-stage paradigm, it obtains a more solid visual representation foundation and serves as the mainstream training paradigm for advanced VLMs currently pursuing higher performance.



![多模态训练的两阶段与三阶段范式对比图 (Step-by-step contrast between two-stage and three-stage multimodal training paradigms.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig18_training_stages.png)

</details>

**15. What Are the Special Considerations for Warm-up and Learning Rate Scheduling in Multimodal Training?**

<details>
<summary>💡 Show Answer</summary>

Warm-up refers to the gradual linear (or otherwise) growth of the learning rate from a very small value to the target learning rate during the initial training steps. The purpose is to avoid gradient explosion or training divergence caused by an excessively large learning rate at the start of training when model parameters and optimizer states are unstable.

In multimodal training, because the model naturally consists of multiple modules with vastly different degrees of pre-training, Warm-up and learning rate scheduling require more refined group-wise designs:

- **Connector (Randomly Initialized)**: Since its parameters learn from scratch, it can usually use a relatively larger peak learning rate, but still requires a certain number of Warm-up steps to help it stably find the roughly correct optimization direction during early training;
- **Language Model / Visual Encoder (Fully Pre-trained)**: To avoid damaging their already powerful learned capabilities, a learning rate much smaller than that of the connector is typically used (potentially differing by an order of magnitude), and in some practices, a longer Warm-up period is set for these modules, allowing the optimizer to adjust these "fragile" pre-trained parameters more slowly and cautiously;
- **Re-Warm-up at Stage Transitions**: When training transitions from one stage (such as alignment pre-training) to the next (such as instruction tuning), because the data distribution, training objective, and even the scope of unfrozen parameters change, the statistical properties of gradients differ significantly from the end of the previous stage. Therefore, it is usually necessary to reset a shorter Warm-up period at the beginning of the new stage to help the optimizer (especially momentum-based optimizers like AdamW) smoothly adapt to the new stage's gradient distribution, avoiding drastic loss oscillations immediately after stage switching.

</details>

**16. How to Choose Batch Size in Multimodal Model Training? What is its Impact on Training Stability?**

<details>
<summary>💡 Show Answer</summary>

The choice of Batch Size requires comprehensive consideration of both the **nature of training objectives** and **hardware resource constraints**:

**Requirements of Training Objectives on Batch Size**: For contrastive learning objectives represented by CLIP (ITC), negative samples in the loss function directly originate from other samples within the same batch. A larger Batch Size means that more and richer negative samples can be provided in each step, generally leading to better contrastive learning performance—this is also why the original CLIP paper used an ultra-large Batch Size of up to 32768. In contrast, training objectives during the instruction tuning stage are closer to standard supervised learning (next-token prediction) and lack a similar reliance on the quantity of negative samples, allowing the use of relatively smaller Batch Sizes (ranging from tens to hundreds).

**Hardware Resource Constraints**: Compared to pure text, multimodal data (especially samples containing images, multiple images, or videos) consumes significantly more GPU memory per sample (a single image may correspond to hundreds or even thousands of visual tokens), directly restricting the number of samples a single GPU can process simultaneously. A common coping method in practice is **Gradient Accumulation**: computing and accumulating gradients across multiple mini-batches, and executing a unified parameter update once the target equivalent Batch Size is accumulated. This enables maintaining the convergence stability advantages of large Batch Size training under GPU memory constraints (especially for contrastive learning objectives requiring large batches).

**Impact on Training Stability**: An overly small Batch Size leads to higher gradient variance (larger gradient noise) calculated at each step, potentially causing significant fluctuations in the training loss curve and slowing down convergence. Although an overly large Batch Size yields more stable gradients, it usually requires a corresponding increase in learning rate (otherwise convergence slows down), and an inappropriately scaled large learning rate may trigger new instabilities instead. Therefore, in practice, Batch Size and learning rate often need to be jointly tuned.

</details>

**17. What is Progressive Resolution Training?**

<details>
<summary>💡 Show Answer</summary>

Progressive Resolution Training refers to training by gradually increasing the resolution of input images stage by stage, rather than using the same (usually higher) target resolution throughout the entire training process. A typical staged approach is as follows:

1. **Early Stage (Low Resolution)**: Uses a smaller image resolution (such as 224×224 or smaller). At this point, the number of visual tokens is small and single-step computational cost is low, allowing a large number of training steps to be completed much faster, helping the model quickly learn basic visual concept recognition and coarse-grained visual-language associations;
2. **Later Stage (Progressively Increasing Resolution)**: As training progresses, the input resolution is gradually increased (such as up to 448×448, 672×672, or higher), combined with techniques such as positional embedding interpolation to adapt the model to a larger number of visual tokens. The primary goal of this stage is to enhance the model's perception of image details (small objects, text, fine textures).

The core benefit of this strategy lies in the **amortization of training costs**: since the vast majority of training steps (especially the initial training phase where model convergence is slower) are completed in the lower-resolution stage with lower computational costs, only a relatively small number of training steps need to bear the higher computational overhead of the high-resolution stage. Therefore, compared to using a fixed high resolution throughout, progressive resolution training can significantly reduce overall computational resources required for training while achieving comparable (or even superior, as staged learning itself serves as a beneficial curriculum learning) final results.

</details>

**18. Should the Learning Rates of the Visual Encoder and Language Model Be Set Differently? Why?**

<details>
<summary>💡 Show Answer</summary>

The answer is: It is generally recommended to set different learning rates, a strategy known as Layer-wise / Group-wise Learning Rate. Specific considerations are as follows:

- **Visual Encoder (If Participating in Training)**: Since it mostly originates from models like CLIP ViT that have been fully pre-trained on massive data with very high quality, its parameters are already in an "optimal" state. If the same (larger) learning rate as other modules is used, large gradient updates in early training can easily "destroy" its learned generic visual representation capabilities (i.e., causing catastrophic forgetting). Therefore, a much smaller learning rate than that of the language model or connector is typically set for the visual encoder (and even a smaller weight decay can be set additionally), allowing it to undergo only "gentle" fine-tuning during training;
- **Connector (Randomly Initialized)**: As a module initialized from scratch that needs to learn from zero, it usually requires a relatively larger learning rate to quickly converge to an effective visual-language mapping within a limited number of training steps;
- **Language Model**: Its learning rate setting usually lies between the visual encoder and the connector, with specific values depending on whether full fine-tuning or parameter-efficient methods like LoRA are adopted—the low-rank matrix parameters added during LoRA fine-tuning are likewise randomly initialized and also require a relatively larger learning rate.

This practice of setting group-wise learning rates essentially dynamically adjusts optimization intensity based on each module's "degree of pre-training" and "degree of trust in the module." It is an important engineering detail widely adopted in multimodal training. If this is ignored and a uniform learning rate is used across all modules, it easily leads to issues where the visual encoder's capabilities are destroyed or the connector converges too slowly.



![多模态联合训练中不同子模块的学率（LR）调度曲线图 (Learning rate schedules for different VLM submodules during joint training.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig19_learning_rates.png)

</details>

**19. In What Aspects Do the "Emergent Abilities" of Multimodal Large Models Usually Manifest?**

<details>
<summary>💡 Show Answer</summary>

Emergent Abilities refer to complex capabilities suddenly exhibited by a model after its parameter scale, data scale, or training compute crosses a certain threshold, which are completely unobservable (or perform extremely poorly) on smaller-scale models. Such capabilities are not acquired through explicit, targeted supervised training, but are the result of the model spontaneously learning and integrating existing knowledge during large-scale pre-training. In multimodal large models, emergent abilities typically manifest in the following aspects:

1. **Cross-Modal Common-Sense Reasoning Capability**: The model can combine visual cues observed in images with massive world knowledge acquired during pre-training to answer open-ended questions that require synthesizing visual perception and common-sense/domain knowledge (e.g., identifying a specific plant species in an image and answering questions about its growth habits based on botanical knowledge). This capability is not obtained through explicit training on the task of "combining vision and knowledge to answer questions," but is a natural result brought by large-scale pre-training;
2. **In-Context Learning Capability**: After providing a few image-text exemplars in the prompt, the model can "grasp" the pattern of the task without any parameter updates and generalize to samples with similar structures but entirely new content. This capability of "learning tasks from exemplars" is usually very weak in small-scale models, but significantly enhanced in large-scale models;
3. **Compositional Generalization Capability**: The model can flexibly combine visual concepts and language instructions that appeared independently in training data into novel combinations, correctly handling specific "instruction-image" pairings never seen during training (e.g., having separately seen "identifying cats" and "writing text in first-person perspective" during training, the model can combine both to accomplish a novel task such as "writing a first-person description from the perspective of a cat");
4. **Preliminary Chain-of-Thought Reasoning**: Without large-scale instruction tuning specifically targeted at multi-step visual reasoning, sufficiently large-scale models can spontaneously exhibit a degree of behavioral pattern involving "first decomposing the problem, then reasoning step-by-step."

These capabilities usually exhibit a "non-linear" pattern—they are almost unobservable when model scale is small, but emerge relatively suddenly once a certain scale threshold is crossed. This serves as one of the important foundations supporting the R&D trajectory of "continuously scaling up model and data sizes."

</details>

**20. What is the Progressive Unfreezing Strategy? How is it Applied in Multimodal Training?**

<details>
<summary>💡 Show Answer</summary>

Progressive unfreezing refers to not unfreezing all parameters right from the start (or conversely, keeping all parameters frozen initially), but rather gradually unfreezing more modules to participate in updates in a specific order as training stages progress. The typical application in multimodal training is as follows:

1. **Stage 1 (Alignment Pre-training)**: Completely freeze the visual encoder and language model, training only the connector (such as a linear projection layer or Q-Former), allowing the connector to first learn a rough but reasonable mapping from vision to language, avoiding huge gradients from the randomly initialized connector perturbing the pre-trained parameters at both ends.
2. **Stage 2 (Preliminary Instruction Tuning)**: Unfreeze the language model (usually combined with parameter-efficient methods like LoRA) while keeping the visual encoder frozen, enabling the language model to learn how to complete instruction tasks by incorporating visual information, built upon the foundational mapping learned by the connector.
3. **Stage 3 (Deep Fine-tuning, Optional)**: If data volume and compute allow, further unfreeze the visual encoder (typically using a smaller learning rate than the language model), allowing the visual representation itself to make adaptive adjustments for specific visual distributions of downstream tasks (e.g., medical imaging, industrial inspection images).

This order of "training newly added modules first, then gradually releasing old modules" is essentially aimed at controlling optimization stability during training—modules unfrozen later are typically those with higher degrees of pre-training and higher "fragility" (susceptible to capability degradation). Progressive unfreezing can protect existing capabilities while gradually expanding the model's adjustable degrees of freedom, serving as a common engineering technique to mitigate "cross-modal catastrophic forgetting."

</details>

