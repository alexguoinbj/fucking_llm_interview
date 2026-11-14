# 4.9 Fine-Tuning Evaluation and Model Selection — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 8 questions

**1. When Performing SFT, Should the Base Model Be Chat or Base?**

<details>
<summary>💡 Show Answer</summary>

When performing supervised fine-tuning (SFT), the choice of base model should be determined by task nature and dataset characteristics:
- If the task is related to dialogue generation, selecting a Chat model may be more appropriate because it has been specifically optimized for conversational interaction.
- If the task involves single-turn text generation or non-conversational tasks, selecting a Base model may be more appropriate because it performs well on single-turn text generation tasks.

</details>

**2. How to Evaluate the Performance of Fine-Tuned Models?**

<details>
<summary>💡 Show Answer</summary>

Evaluating fine-tuned model performance is an essential step to ensure that the model meets task requirements:
1. **Evaluation Using a Test Set**: Input the test set into the model, obtain prediction results, and compare them with ground-truth labels.
2. **Selecting Appropriate Evaluation Metrics**: Use Accuracy, Recall, F1 score, etc., for classification tasks; BLEU score, ROUGE score, etc., for generation tasks; for multi-turn dialogue tasks, dialogue coherence, consistency, and user satisfaction can also be evaluated.
3. **Conducting Comparative Analysis**: Compare the fine-tuned model with baseline models to analyze specific aspects of performance improvement.
4. **User Feedback and Real-World Testing**: Collect user feedback in practical application scenarios and conduct A/B testing.
5. **Error Analysis**: Perform detailed analysis on mispredicted samples to identify model weaknesses and deficiencies.

</details>

**3. What Common Evaluation Metrics Are Used for Fine-Tuned Models?**

<details>
<summary>💡 Show Answer</summary>

Evaluation metrics for fine-tuned model performance vary depending on task types, commonly including:

**1. Classification/Sequence Labeling Tasks**: Accuracy, Precision, Recall, F1 (as well as macro-average/micro-average); confusion matrix.

**2. Generation Tasks (translation, summarization, etc., with reference answers)**: BLEU, ROUGE, METEOR (based on n-gram overlap); BERTScore (semantic-level evaluation based on embedding similarity).

**3. Open-Ended Generation Tasks (dialogue, creative writing, etc., without fixed reference answers)**: LLM-as-Judge; human evaluation; Win Rate / preference win rate.

**4. Instruction-Following Capability**: Format/constraint adherence rate (automated checks); multi-step instruction completion rate.

**5. General Capability Benchmarks**: MMLU, C-Eval, etc., to monitor whether fine-tuning negatively impacts the model's general capabilities.

**6. Task-Specific Business Metrics**: "Issue resolution rate" in customer service scenarios, "code execution pass rate / unit test pass rate" in code generation scenarios, etc.

**7. Safety and Compliance Metrics**: Evaluate whether fine-tuning introduces safety risks using dedicated safety evaluation benchmarks.

</details>

**4. How to Construct a Domain-Specific Evaluation Benchmark?**

<details>
<summary>💡 Show Answer</summary>

The steps for constructing a domain-specific evaluation benchmark are as follows:
1. **Data Collection**: Obtain data relevant to the target domain from the internet, public datasets, or domain experts to ensure data representativeness and diversity.
2. **Data Annotation**: Annotate data according to task types, which can be done manually or with automated tools.
3. **Dataset Splitting**: Split the annotated data into training, validation, and test sets.
4. **Designing Evaluation Metrics**: Select appropriate metrics based on task types.
5. **Conducting Evaluation**: Use the evaluation benchmark to evaluate the fine-tuned model and calculate metrics.
6. **Analysis and Improvement**: Analyze model performance based on evaluation results and adjust hyperparameters or model architecture as needed.

</details>

**5. How to Choose an Appropriate Pre-Trained Model for Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Selecting an appropriate pre-trained model is key to successful fine-tuning:
1. **Task Relevance**: Choose the pre-trained model most relevant to the task based on the task type.
2. **Model Size and Resources**: Consider model size and computational resources. Larger models typically feature stronger performance, but also require more VRAM and computational resources.
3. **Pre-Training Data Quality and Diversity**: Select models pre-trained on large-scale, high-quality corpora.
4. **Community Support and Documentation**: Select pre-trained models with strong community support and detailed documentation.
5. **Experimental Verification**: If conditions permit, try fine-tuning multiple pre-trained models and compare their performance on specific tasks.

</details>

**6. How to Perform Efficient Fine-Tuning with Minimal Data?**

<details>
<summary>💡 Show Answer</summary>

When annotated data is extremely scarce (e.g., tens to hundreds of samples), common strategies for efficient fine-tuning include:
1. **Prioritize PEFT Methods**: Parameter-efficient fine-tuning methods such as LoRA are less prone to overfitting on small datasets and are the preferred choice for few-shot fine-tuning.
2. **SetFit-like Methods (for Classification Tasks)**: "Combine" a small number of annotated samples via contrastive learning to generate a large number of training pairs, and then train a simple classifier.
3. **Data Augmentation and Synthetic Data**: Use LLMs to rewrite, expand, or generate similar variants from a small number of seed samples.
4. **Fully Leverage Pre-Trained/General Model Capabilities**: Select a pre-trained model that already possesses strong baseline capabilities in related tasks/domains; consider using In-Context Learning as an alternative or supplement to fine-tuning.
5. **Transfer Learning**: Fine-tune first on a related task with a larger dataset, then perform a second-stage fine-tuning on the target small dataset.
6. **Fine-Grained Hyperparameters and Early Stopping Strategies**: Use a smaller learning rate, fewer training epochs, and aggressive early stopping strategies.
7. **Cross-Validation**: When data volume is extremely small, adopting $k$-fold cross-validation allows for a more robust evaluation of model performance.

</details>

**7. If Annotated Training Data Is Very Scarce, How to Augment the Amount of Training Data? (Hint: SetFit)**

<details>
<summary>💡 Show Answer</summary>

![SetFit Two-Stage Contrastive Learning Architecture for Ultra-Few-Shot Text Classification](https://img.pricenode.uk/llm_interview_questions/english/chapter4/setfit_contrastive_fewshot.png)




When annotated data is extremely scarce (e.g., only tens of samples per class), common data augmentation and efficient utilization strategies include:

**1. Data Augmentation**: Text-level perturbations (synonym replacement, EDA, back-translation); LLM-based generative augmentation; Mixup / embedding space interpolation.

**2. SetFit Method (Sentence Transformer Fine-Tuning)**

SetFit is an efficient method specifically designed for "few-shot text classification." Its core workflow consists of two steps:
- **Step 1: Contrastive Fine-Tuning of Embedding Model**: Use a very small number of annotated samples to construct a large number of sentence pairs (same-class positive pairs, different-class negative pairs), and fine-tune a Sentence Transformer with contrastive loss to better adapt its embedding space to the semantic boundaries of the target classification task.
- **Step 2: Training a Lightweight Classifier Head**: Extract embedding vectors for all annotated samples using the fine-tuned embedding model, and use these features to train a simple classifier (e.g., logistic regression).
The core value of SetFit lies in "combining" annotated samples to generate a large volume of sentence pairs, transforming the "few-shot classification" problem into a two-step process of "contrastive fine-tuning + simple classifier."

**3. Semi-Supervised / Weakly Supervised Methods**: Train an initial model with a small amount of annotated data, assign "pseudo-labels" to a large amount of unannotated data, and filter high-confidence pseudo-labeled samples to add to the training set (self-training).

</details>

**8. Someone Claims an Article Was Generated by DeepSeek-R1 and Provides You with the Complete Prompt Used for Generation. How Should You Verify or Refute This Claim? How Can You Quantitatively Calculate the Probability of This Prompt Generating the Article?**

<details>
<summary>💡 Show Answer</summary>

**Verification Strategy**:
1. **Direct Reproduction**: Call the DeepSeek-R1 model multiple times using the provided complete prompt to generate content, and compare the generated results with the target article in terms of content, structure, and wording style. Focus on whether core information points and argumentative structures align closely, as well as traces of the Chain-of-Thought (CoT) characteristic of the R1 model.
2. **Stylistic Feature Analysis**: Analyze whether the target article exhibits statistical features characteristic of DeepSeek-R1 (frequency of certain phrases, sentence length distribution, etc.).
3. **Using AI Text Detection Tools**: Can serve as auxiliary references but cannot be used as decisive evidence.
4. **Checking Timeliness and Knowledge Boundaries**: If the article contains information beyond the knowledge cutoff date of DeepSeek-R1's training data, it may indicate that the article was not entirely generated by the model.

**Quantitatively Calculating "the Probability of This Prompt Generating the Article"**:

From the perspective of language modeling, given prompt $P$ and model $M$, the probability of generating article $T$ can be expressed as:

$P(T | P, M) = \prod_{i} P(t_i | P, t_1, ..., t_{i-1}, M)$

If model API access is available to obtain token-level log probabilities, article $T$ can be fed into the model token-by-token to obtain the model's predicted probability for each ground-truth token. Summing the log probabilities of all tokens yields $\log P(T | P, M)$. Perplexity can serve as a proxy metric: $PPL = \exp(-\frac{1}{N} \sum \log P(t_i | ...))$, where a lower perplexity indicates a higher "likelihood" of generating this text under the given prompt.

**Important Limitations**: This numerical value alone cannot directly prove whether the article was generated by the model. It requires collecting statistics on the probability distributions of two categories of samples—a large volume of text samples known to be "generated by the model" versus "not generated by the model"—in order to determine a statistically significant decision threshold.

</details>

