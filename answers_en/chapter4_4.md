# 4.4 Pre-Training vs. Fine-Tuning vs. Continued Pre-Training — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 6 questions

**1. To Have a Model Learn Domain Knowledge, Should One Choose Pre-Training or Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

- **Choose Continued Pre-Training**: Suitable when the target domain language distribution differs significantly from general corpora, and general models have weak language modeling capabilities in that domain (e.g., code, Chinese law, medical literature);
- **Choose SFT (Instruction Fine-Tuning)**: Suitable when the general model already possesses domain knowledge and only needs adjustment of output format and instruction-following behavior;
- **Choose RAG (Retrieval-Augmented Generation)**: Suitable when knowledge updates frequently or private data is unsuitable to be trained into the model, offering the lowest cost.
Generally recommended path: General Base Model $\rightarrow$ Continued Pre-Training (optional) $\rightarrow$ SFT $\rightarrow$ RLHF/DPO

</details>

**2. At Which Stage Is Knowledge Injected: Pre-Training or Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

- **Pre-training phase**: The model learns general capabilities such as grammar, semantics, world knowledge, and commonsense reasoning by performing self-supervised learning on large-scale unlabeled corpora. Knowledge is primarily injected during this stage.
- **Fine-tuning phase (SFT/RLHF)**: Rather than injecting new knowledge, this stage activates and aligns knowledge already present from pre-training, enabling the model to output according to human-expected formats and styles while enhancing instruction-following capabilities.

</details>

**3. Comparison of Pros and Cons Between SFT and RLHF**

<details>
<summary>💡 Show Answer</summary>

**SFT**:
- Advantages: Simple implementation, stable training, relatively low computational cost, predictable performance.
- Disadvantages: Relies on large volumes of high-quality annotated data; essentially imitation rather than preference optimization, making it hard to express "relative quality"; prone to poor performance in out-of-distribution scenarios; cannot directly optimize higher-level goals such as "human satisfaction."

**RLHF**:
- Advantages: Models relative preferences via reward models, better aligning with "what humans truly want"; explores wider output spaces to find superior strategies; more effective in improving overall quality such as "helpfulness and harmlessness."
- Disadvantages: Complex training pipeline, sensitive to hyperparameters, unstable training, high computational cost; risk of Reward Hacking.

**Relationship in Practice**: Usually, SFT is used first to lay a solid foundation, followed by RLHF (or lighter preference optimization methods such as DPO) for further alignment; the two are complementary rather than mutually exclusive.

</details>

**4. During Continued Pre-Training, How Can One Ensure the Model Acquires Specific Domain Knowledge While Preserving Its General Capabilities to the Maximum Extent?**

<details>
<summary>💡 Show Answer</summary>

![Data Replay Strategy and Corpus Mixture Ratio in Continued Domain Pre-Training](https://img.pricenode.uk/llm_interview_questions/english/chapter4/continued_pretrain_data_replay.png)




The core contradiction faced in continued pre-training (Continued Pretraining / Domain-Adaptive Pretraining) is the trade-off between "learning new domain knowledge" and "forgetting existing general capabilities." Commonly used mitigation strategies include:

**1. Data Level**: Mix a certain proportion of general corpora (such as a subset of original pre-training data) into the domain data, allowing the model to continuously "review" the general distribution while learning domain knowledge.

**2. Training Hyperparameters Level**: Use a smaller learning rate (e.g., 1/10 to 1/100 of the original learning rate) and shorter training steps, combined with warmup and decay scheduling strategies.

**3. Parameter Level**: Use PEFT methods such as LoRA to train only a small number of newly added parameters while keeping original model parameters frozen, structurally guaranteeing that general capabilities are not damaged.

**4. Regularization Methods**: Continual learning regularization methods like Elastic Weight Consolidation (EWC); knowledge distillation constraints to keep the output distribution of the continued pre-trained model on general data as close as possible to the original model.

**5. Evaluation and Monitoring**: Periodically evaluate on both general capability benchmark suites and domain evaluation sets simultaneously; once a significant forgetting trend appears, promptly adjust the data mixing ratio or stop training early.

</details>

**5. Please Compare the Pros and Cons of the Following Three Approaches for Text Classification Tasks in Vertical Domains: (a) Fine-Tuning a General BERT Model Directly; (b) Fine-Tuning BERT After Continued Pre-Training on Medical Text; (c) Pre-Training a Model from Scratch on Medical Text and Then Fine-Tuning.**

<details>
<summary>💡 Show Answer</summary>

**(a) Fine-Tuning General BERT Directly**

Advantages: Simple implementation, lowest training cost; general BERT has already learned rich general language knowledge, which remains effective for medical texts with minor differences.

Disadvantages: May insufficiently understand terms, abbreviations, and professional expressions unique to the medical domain; when labeled data is scarce, it is difficult for the model to make up for the lack of domain knowledge solely through the fine-tuning phase.

**(b) Fine-Tuning BERT After Continued Pre-Training on Medical Text**

Advantages: Allows the model to learn medical vocabulary distributions and term semantics via continued pre-training while retaining most general language capabilities; outperforms approach (a) and costs significantly less than approach (c).

Disadvantages: Requires a certain scale of unlabeled medical corpora and extra computational resources; carries some risk of catastrophic forgetting.

**(c) Pre-Training from Scratch on Medical Text and Then Fine-Tuning**

Advantages: All model parameters are learned from scratch based on medical corpora, theoretically allowing maximum optimization for the medical domain.

Disadvantages: Extremely high training cost; loss of general language capabilities; cost-effectiveness is usually very low.

**Overall Recommendation**: The optimal sweet spot for most practical scenarios is (b); only consider (c) when possessing vast volumes of domain-exclusive corpora and a sufficient compute budget.

</details>

**6. In Continued Pre-Training Based on Masked Language Modeling, How Should the Mask Positions and Probabilities Be Designed?**

<details>
<summary>💡 Show Answer</summary>

- **Mask Ratio**: Usually follows the original BERT setting (about 15%); when domain term density is high, the ratio can be appropriately lowered;
- **Entity/Term-Aware Masking**: Prioritize or additionally increase the masking probability for domain-specific terms and entities, forcing the model to focus on learning to predict these terms;
- **Whole Word Masking (WWM)**: For terms split by subwords, all subwords forming the same word should be masked together to prevent the task from being overly simple;
- **Span Masking**: Randomly select continuous token spans to mask, encouraging the model to learn longer-range dependencies.

% ============================================================

</details>

