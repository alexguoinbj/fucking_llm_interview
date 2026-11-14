# 4.7 Fine-Tuning Hyperparameters, VRAM, and Overfitting Management — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 7 questions

**1. Why is the Model Usually More Sensitive to Hyperparameters Like Learning Rate During Fine-Tuning Than During Pre-Training?**

<details>
<summary>💡 Show Answer</summary>

The main reasons why the model is more sensitive to hyperparameters during fine-tuning include:
1. **The Model is Already in a "Favorable" Parameter Region**: Pre-trained model parameters reside in a low-loss "basin" formed over large-scale general data. During fine-tuning, if the learning rate is too high, parameter updates may "jump out" of this favorable region, causing the model to rapidly deviate from useful pre-trained representations (i.e., catastrophic forgetting).
2. **Fine-Tuning Data Volume is Typically Much Smaller Than Pre-Training Data**: Gradient estimates for each batch are noisier. If the learning rate is too large, substantial gradient updates driven by a small number of samples can cause parameters to oscillate violently or overfit.
3. **Fine-Tuning Usually Involves Very Few Training Steps**: If inappropriate initial learning rates cause parameters to drift far off track, there are not enough subsequent steps to "correct" them, which severely impacts final performance.
4. **Higher Risk of Overfitting**: With smaller data volumes and fewer training steps, the "safe operating window" for learning rates is typically narrower than in pre-training, requiring more precise tuning.

</details>

**2. How Should the Learning Rate Be Set During the Fine-Tuning Stage?**

<details>
<summary>💡 Show Answer</summary>

- Fine-tuning learning rates are typically 1–2 orders of magnitude smaller than during pre-training;
- For full-parameter fine-tuning, common learning rate ranges are between **1e-5 and 5e-5**;
- For PEFT methods (such as LoRA), common ranges are between **1e-4 and 5e-4**;
- Combined with warmup (the first 3%–10% of training steps) and decay schedules (linear or cosine decay);
- The larger the model scale, the smaller the suitable learning rate;
- Typically determined through small-scale grid search, combining loss curves on the validation set with downstream task metrics.

</details>

**3. When Fine-Tuning Models, How Should Hyperparameters Such as Learning Rate, LoRA Alpha, and LoRA Rank Be Set? How to Decide When to Stop Training, and Is a Lower Validation Loss Always Better?**

<details>
<summary>💡 Show Answer</summary>

**Hyperparameter Settings**:
- **Learning Rate**: Full-parameter fine-tuning commonly uses 1e-5~ 5e-5, while LoRA fine-tuning commonly uses 1e-4~ 5e-4, combined with warmup and decay schedulers;
- **LoRA Rank (r)**: Commonly ranges between 4~ 64; 8~ 16 is sufficient for most tasks;
- **LoRA Alpha**: A common empirical setting is $alpha = 2 \times r$ (i.e., scaling factor of 2) or $alpha = r$ (scaling factor of 1).

**When to Stop Training—Is Lower Validation Loss Always Better?**

**No**, achieving the lowest validation loss does not strictly equate to optimal performance on downstream tasks:
1. Loss measures the "accuracy of predicting the next token," whereas downstream task performance measures the "overall quality / usefulness / correctness of generated content"—the two are not strictly monotonic;
2. The model's loss on token-level predictions may continuously decrease, but it may overfit to specific patterns in the training data, resulting in degraded actual generation quality on broader unseen inputs;
3. "Soft" objectives like instruction following and generation quality are difficult to fully capture via loss alone.

**Practical Approaches to Deciding When to Stop Training**: Simultaneously monitor validation loss and downstream task metrics, selecting the checkpoint with optimal downstream metrics; periodically save checkpoints for comprehensive comparative evaluation; observe "inflection points" in loss curves; combine the gap between training and validation losses to assess overfitting risks.

</details>

**4. How to Detect Fine-Tuning Overfitting? How to Resolve Overfitting When It Occurs?**

<details>
<summary>💡 Show Answer</summary>

![Training vs. Validation Loss Divergence Curve and Optimal Early Stopping Inflection Point](https://img.pricenode.uk/llm_interview_questions/english/chapter4/overfitting_detection_early_stopping.png)




**Methods for Detecting Fine-Tuning Overfitting**:
- Training loss continues to drop while validation loss stops decreasing or starts rising, with the gap between them constantly widening;
- Downstream task metrics on the validation set stall or deteriorate;
- Diversity of generated content drops, or specific patterns from training data are repeatedly generated;
- Over-sensitivity to specific phrasing/formatting in training data, exhibiting poor generalization.

**Methods for Resolving Overfitting**:
- Early stopping;
- Reduce the number of training epochs / lower the learning rate;
- Increase data volume and diversity, and deduplicate training data;
- Regularization techniques (Dropout, weight decay, or reducing LoRA rank $r$);
- Mix in general data to prevent the model from overfitting to narrow data distributions;
- Capitalize on the regularization effect of PEFT methods themselves (fewer trainable parameters provide implicit regularization).

</details>

**5. How to Achieve Optimal Performance and Capabilities in LLM Systems While Controlling Costs?**

<details>
<summary>💡 Show Answer</summary>

- Since LLMs charge per token (words, punctuation, etc.), focus on clear, specific prompts and eliminate unnecessary information;
- When using LLM APIs, consider batching similar requests to reduce the total number of API calls, thereby cutting costs;
- Use techniques like LLMLingua to remove redundant words from inputs and outputs;
- For chatbots or interactive systems, limit dialogue history stored in memory;
- Different LLMs vary in cost and capability; select LLMs specifically tailored to your task to achieve optimal performance within budget.

</details>

**6. How Much VRAM Does Full-Parameter Fine-Tuning Require? How to Estimate It?**

<details>
<summary>💡 Show Answer</summary>

Rough estimation formula (mixed-precision training):
- Model weights (FP16): $Number of parameters \times 2 bytes$
- Gradients (FP16): $Number of parameters \times 2 bytes$
- Optimizer states (Adam, FP32): $Number of parameters \times 8 bytes$ (first and second moments, 4 bytes each)
- Total approx.: $Number of parameters \times 16 bytes$
**Example**: Full-parameter fine-tuning of a 7B model $\approx 7 \times 10^9 \times 16  bytes \approx 112 GB$, requiring multiple A100 GPUs (80GB). In practice, activation memory must also be added, where gradient checkpointing is commonly used to trade time for space.

</details>

**7. What Are Practical Data Augmentation Methods in Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Commonly used data augmentation methods in fine-tuning include:

**1. Text-Level Perturbation Methods**: Synonym substitution; random insertion/deletion/swap (EDA); back-translation.

**2. LLM-Based Generative Augmentation**: Paraphrasing/expansion; instruction expansion (generating more variants based on a small set of seed instructions); hard sample generation (adding distractor information, altering question phrasing).

**3. Template/Format-Level Augmentation**: Instruction phrasing diversification (using multiple distinct phrasing styles for the same task); random combinations of few-shot examples.

**4. Precautions**: Augmented samples must undergo quality checks to avoid introducing semantic drift; the ratio of augmented data should be controlled appropriately, as excessive augmented data may introduce "synthetic-data-specific artifacts." It is generally recommended to mix augmented data with original real data in a balanced ratio.

% ============================================================

</details>

