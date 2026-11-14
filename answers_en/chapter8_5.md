# 8.5 Data Contamination, Robustness, and Adversarial Evaluation — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 9 questions

**1. How to determine if an evaluation set is contaminated?**

<details>
<summary>💡 Show Answer</summary>

Check the n-gram overlap rate between the training data and the evaluation set, or use methods such as Min-K%, perplexity anomaly detection to determine whether the model has "memorized" test samples; one can also compare model performance between the public version and a rewritten/perturbed version of the evaluation set—a significant performance drop after rewriting indicates overfitting or contamination.

</details>

**2. What is Data Contamination in evaluation? How to detect it?**

<details>
<summary>💡 Show Answer</summary>

Data contamination refers to the situation where test items (or highly similar content) from the evaluation set are already present in the model's training data. As a result, the model may have "memorized" the answers rather than truly possessing the corresponding capabilities, leading to inflated evaluation scores. Detection methods include: searching the training corpus for exact or approximate text snippets of test items (N-gram overlap detection), and comparing whether the model's performance on the public test items is significantly higher than on structurally similar but unreleased new items of the same type.





![Methodologies for detecting test dataset leakage and training contamination. (大模型训练数据污染（评测集泄漏）的常用检测方法链路图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig21_contamination_detection.png)

</details>

**3. How to design evaluations to distinguish whether "a model truly possesses a certain capability" versus "a model obtains correct answers through surface pattern matching"?**

<details>
<summary>💡 Show Answer</summary>

This can be achieved by constructing "counterfactual" or "perturbed" versions of questions—keeping the core reasoning logic of the task unchanged while altering specific numerical values, entity names, or surface phrasing (avoiding complete identity with original questions that might have been seen in the training data)—and observing whether the model's performance on these variants is consistent with the original questions. If the model's performance drops significantly on variants, it indicates that its original "correctness" likely relied on memorizing specific surface patterns rather than a true mastery of the underlying logic.

</details>

**4. What is the "Shortcut Learning" phenomenon in Benchmarks? Give an example.**

<details>
<summary>💡 Show Answer</summary>

Shortcut learning refers to the phenomenon where a model learns to leverage statistical shortcuts present in a dataset—which are unrelated to the true capability required by the task—to achieve high scores, rather than learning the actual underlying logic of the task. For example, in a Natural Language Inference (NLI) dataset, there might be a statistical bias such as "when the hypothesis sentence contains negative words, the label is more likely to be 'contradiction'". The model might achieve a decent score on this dataset purely based on the surface feature of "whether it contains negative words," without truly understanding the logical relationship between the premise and the hypothesis. When the test set no longer exhibits this bias, the model's performance drops significantly.

</details>

**5. What are "Compositional Generalization" Benchmarks? What do they attempt to test?**

<details>
<summary>💡 Show Answer</summary>

Such Benchmarks test whether a model can combine mastered "basic components" (such as known concepts or operations) in novel ways unseen during training to solve new problems. For example, if a model has separately learned the application of "addition" and "multiplication" operations, can it correctly solve a complex word problem that requires combining both in a novel manner? They attempt to distinguish whether a model solves problems by "memorizing composite patterns seen in the training distribution" or if it truly possesses a deeper generalization/reasoning capability to "flexibly combine known basic capabilities for novel compositions"—the latter of which is generally considered a hallmark closer to "general intelligence".

</details>

**6. How to perform adversarial evaluation?**

<details>
<summary>💡 Show Answer</summary>

Construct adversarial samples (such as prompt injections, jailbreak attacks, noise perturbations, and adversarial suffixes) to test model performance under stress scenarios; discover model vulnerabilities through manual red-teaming or automated adversarial generation tools (such as GCG attacks), and calculate attack success rates and defense effectiveness.

</details>

**7. What is "Adversarial Testing" in evaluation?**

<details>
<summary>💡 Show Answer</summary>

Adversarial testing refers to specifically constructing inputs designed to "induce" model errors or expose flaws (such as ambiguous questions, premises with misleading information, or deliberately crafted edge cases) to discover weaknesses that are not easily exposed in standard evaluations.

</details>

**8. How to evaluate model robustness?**

<details>
<summary>💡 Show Answer</summary>

Test the performance stability of the model against input perturbations (synonym replacement, typos, word order adjustments, noise injection), prompt rewrites, and out-of-distribution (OOD) samples; measure the output consistency and accuracy variation before and after perturbations—smaller variation indicates better robustness.

</details>

**9. What is "Out-of-Distribution" (OOD) generalization evaluation? How to design it?**

<details>
<summary>💡 Show Answer</summary>

OOD evaluation focuses on model performance on inputs that have systematic differences from the common distribution of the training/evaluation set, such as using different writing styles, new events/concepts after the model's training knowledge cutoff date, or expressions from different cultural backgrounds. In terms of design, it requires deliberately collecting or constructing samples with noticeable distribution shifts from standard evaluations to specifically assess the model's generalization robustness rather than merely its performance on "in-distribution" test sets.

</details>

