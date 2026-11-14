# 8.1 Basic Evaluation Metrics and Methodology — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 49 questions

**1. What is the primary purpose of evaluating Large Language Models (LLMs)?**

<details>
<summary>💡 Show Answer</summary>

The purpose of evaluation is to measure a model's performance across specific capability dimensions (such as knowledge, reasoning, and safety). It is used for model selection, version iteration comparisons, identifying capability shortcomings, and guiding subsequent data and training optimization directions.

</details>

**2. What dimensions are usually included in LLM evaluation?**

<details>
<summary>💡 Show Answer</summary>

It usually includes: general capabilities (knowledge, reasoning, language understanding), specialized capabilities (code, mathematics, multilingualism), alignment capabilities (instruction following, safety, values), generation quality (fluency, relevance, factuality), efficiency (inference speed, VRAM footprint), robustness (adversarial examples, noisy inputs), long-context capability, multi-turn interaction capability, etc.

</details>

**3. What is a Benchmark?**

<details>
<summary>💡 Show Answer</summary>

A benchmark is a collection of tasks with standardized inputs and reference answers/scoring criteria. It is used to measure and compare the capabilities of different models in a unified, reproducible manner, facilitating cross-model comparisons.

</details>

**4. What are Automatic Evaluation and Human Evaluation?**

<details>
<summary>💡 Show Answer</summary>

Automatic evaluation scores model outputs programmatically (e.g., exact matching, rule-based scoring, automated metric calculation). It is fast, low-cost, and scalable/reproducible on a large scale. Human evaluation relies on human annotators to judge and score model outputs according to guidelines, which aligns more closely with real user experience, but is costly, slow, and subject to subjective variation.

</details>

**5. What are the differences between human evaluation and automatic evaluation?**

<details>
<summary>💡 Show Answer</summary>

Human evaluation is closer to real user experience and can capture subtle dimensions such as semantics, tone, and creativity, but it suffers from high costs, long cycles, and subjective bias. Automatic evaluation (based on rules or judge models) is highly efficient and scalable/reproducible, but may face issues such as metrics inconsistent with real quality and preference biases of the judge model itself. The two are usually combined to validate each other.

</details>

**6. What are Exact Match (EM) and F1 Score? In which scenarios are they commonly used?**

<details>
<summary>💡 Show Answer</summary>

Exact Match measures whether the model output is completely identical to the reference answer. F1 Score measures the word/token-level overlap between the model output and the reference answer (the harmonic mean of precision and recall). Both are commonly used in tasks with clear gold-standard answers, such as extractive question answering (e.g., reading comprehension).

</details>

**7. What is Perplexity (PPL)? What does it reflect?**

<details>
<summary>💡 Show Answer</summary>

Perplexity reflects the probability assigned by a model to a given text; a lower value indicates that the model considers the text to be more "expected" (stronger language modeling capability). It primarily reflects the model's capability to model language fluency, but cannot directly reflect the factual correctness or usefulness of the generated content.

</details>

**8. What are BLEU and ROUGE? What tasks are they respectively suitable for?**

<details>
<summary>💡 Show Answer</summary>

BLEU evaluates generation quality by calculating the $N$-gram overlap between generated text and reference text (primarily precision-oriented), commonly used in machine translation. ROUGE focuses more on recall (how much content in the reference text is covered by the generated text), commonly used to evaluate text summarization tasks.

</details>

**9. What scenarios are ROUGE, BLEU, and BERTScore respectively suitable for?**

<details>
<summary>💡 Show Answer</summary>

- ROUGE: A recall-oriented metric based on $n$-gram overlap, commonly used for evaluating text summarization tasks.
- BLEU: Based on $n$-gram precision with a brevity penalty, commonly used for evaluating machine translation tasks.
- BERTScore: Computes semantic similarity based on pre-trained language models, capable of capturing paraphrases. It is suitable for evaluating generation tasks with high requirements for semantic consistency and large differences in surface vocabulary.

</details>

**10. Why do traditional NLP evaluation metrics (such as BLEU and ROUGE) have significant limitations in evaluating the generation quality of modern LLMs?**

<details>
<summary>💡 Show Answer</summary>

The core idea of traditional NLP evaluation metrics, such as BLEU (commonly used in machine translation) and ROUGE (commonly used in text summarization), is to compare the surface vocabulary ($n$-gram) overlap between model-generated text and one or more "reference answers." This approach has significant limitations when evaluating the generation quality of modern LLMs for the following reasons:

1. Lack of Semantic Understanding:

- These metrics care only about surface-level lexical matching and have no understanding of the underlying semantics. For example, "The weather is great today" and "The sunshine is very bright today" mean essentially the same thing to humans, but their BLEU/ROUGE scores would be very low due to minimal lexical overlap. Conversely, a sentence with high lexical overlap with the reference answer but poor grammar or confused logic might still receive a high score.

1. Cannot Evaluate Factual Accuracy:

- One of the core challenges of LLMs is hallucination. A generated answer might be linguistically fluent and even similar in style to the reference answer, yet contain completely incorrect facts. BLEU/ROUGE cannot detect such factual errors.

1. Ignores Diversity and Creativity:

- For open-ended generation tasks (such as dialogue, creative writing, or brainstorming), there is simply no unique "gold standard answer." A good LLM should be able to generate diverse, creative, and reasonable responses. Evaluation methods based on fixed reference answers penalize any response that differs from the reference answer despite being equally excellent, suppressing creativity.

1. Poor Evaluation of Long-Form Content:

- These metrics are virtually incapable of evaluating the coherence, logic, and structure of long-form text (such as articles or reports). They can only perform local, fragmented lexical matching.

1. Ignores the Reasoning Process:

- For questions requiring reasoning (such as math or logic problems), the value of an LLM lies not only in the final answer but even more in its "Chain-of-Thought." BLEU/ROUGE can only compare the text strings of the final answers, completely failing to evaluate whether the reasoning steps are correct.

In summary, modern LLM evaluation needs to transcend surface lexical matching and delve into higher-dimensional capabilities such as semantic understanding, factuality, logical reasoning, safety, and instruction following, which are precisely the blind spots of traditional metrics like BLEU and ROUGE.





![Comparison of traditional lexical matching (BLEU/ROUGE) and semantic evaluation for LLMs. (传统词汇匹配（BLEU/ROUGE）与现代大语言模型语义评估方法的对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig01_traditional_metrics_limits.png)

</details>

**11. How is generation quality evaluated?**

<details>
<summary>💡 Show Answer</summary>

It can be evaluated across dimensions such as fluency, coherence, relevance, information richness, and style consistency. Common methods include automatic metrics (ROUGE, BLEU, BERTScore, etc.), human scoring (Likert scales), and using judge models (e.g., GPT-4 as a judge) for pairwise comparison or scoring.

</details>

**12. What is Multiple-Choice Evaluation? Why do many knowledge benchmarks adopt this format?**

<details>
<summary>💡 Show Answer</summary>

Multiple-choice evaluation provides a question alongside several candidate options and determines whether the model can select the correct option. This format features a unique answer and easy automated scoring, making it suitable for large-scale, low-cost evaluation of a model's knowledge mastery. Thus, it is widely adopted in knowledge-based benchmarks such as MMLU.

</details>

**13. Why does "shuffling option order" in multiple-choice evaluation affect model scores? What does this indicate?**

<details>
<summary>💡 Show Answer</summary>

Some models may exhibit position bias toward option placement (e.g., a tendency to select "A" or the last option) rather than deciding purely based on content. If the score changes significantly after shuffling option orders, it indicates that the model's "accuracy" is confounded by this position bias rather than purely reflecting its knowledge or reasoning capability.

</details>

**14. What are Zero-shot and Few-shot evaluations? What is the difference between them?**

<details>
<summary>💡 Show Answer</summary>

Zero-shot evaluation tests the model directly with instructions without providing any task examples. Few-shot evaluation provides a small number (e.g., 1--5) of task examples in the prompt before requesting an answer. Few-shot evaluation typically improves model performance on specific formats or tasks. Both settings are often reported separately to comprehensively reflect model capabilities.

</details>

**15. What is the "Pass@k" metric in evaluation? In what scenarios is it commonly used?**

<details>
<summary>💡 Show Answer</summary>

Pass@$k$ refers to the probability that at least one out of $k$ generated candidate answers passes verification (e.g., code passing unit tests). It is commonly used to evaluate tasks such as code generation where correctness can be automatically verified and model outputs involve randomness.

</details>

**16. How to evaluate the coding capabilities of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Using benchmarks such as HumanEval, MBPP, LiveCodeBench, and CodeContests, code correctness is measured via unit test pass rates ($pass@k$). Sub-capabilities such as code completion, debugging, explanation, multilingual code translation, and code comprehension can also be evaluated, combined with human review of code style and maintainability.

</details>

**17. How to evaluate the mathematical capabilities of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Benchmarks such as GSM8K (grade-school math word problems), MATH (competition problems), AIME, and MathBench are used to measure final answer accuracy. The correctness of the reasoning process (whether intermediate steps are sound), performance across difficulty gradients, and generalization to multi-step structured problems can also be evaluated.

</details>

**18. How to evaluate the logical reasoning capabilities of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Benchmarks such as BBH (Big-Bench Hard), LogiQA, ARC-Challenge, and ReClor are used to test deductive, inductive, and analogical reasoning. Chain-of-reasoning tasks can also be designed to check the logical consistency of intermediate steps and performance under counterfactual or paradoxical scenarios.

</details>

**19. How to evaluate the long-context comprehension capability of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Using benchmarks like LongBench, L-Eval, InfiniteBench, and Needle-in-a-Haystack testing, tasks such as information retrieval, cross-paragraph reasoning, summarization, and question answering are evaluated under various context lengths, paying close attention to whether performance degrades as context length increases.

</details>

**20. How is long-context capability evaluated?**

<details>
<summary>💡 Show Answer</summary>

Through "Needle-in-a-Haystack" testing, key information is inserted at different context lengths and depth positions to test the model's retrieval accuracy. Multi-task long-context benchmarks such as LongBench are also used to evaluate performance curves for summarization, Q&A, and multi-document reasoning across different lengths, focusing on whether a distinct "lost in the middle" phenomenon occurs.

</details>

**21. What are the design challenges in "long-context" evaluation? What are the common evaluation paradigms?**

<details>
<summary>💡 Show Answer</summary>

The main challenges are: constructing tasks that genuinely require utilizing information scattered across long contexts (rather than relying on local information), and preventing evaluation tasks from being bypassed via shortcuts like "keyword matching" without truly understanding the long text. Common paradigms include: Needle-in-a-Haystack (inserting specific facts into long text to test whether the model can accurately retrieve them at designated locations), multi-document comprehensive Q&A (where answers require integrating information scattered across different positions in the long context), and faithfulness evaluation of long-document summarization.

</details>

**22. What are the limitations of the "Needle in a Haystack" test itself?**

<details>
<summary>💡 Show Answer</summary>

The test primarily measures a model's retrieval capability for an "inserted isolated fact," whereas real-world long-context scenarios often require the model to perform comprehensive understanding, reasoning, and information integration over the entire context rather than simple retrieval and localization. Excellent performance on this test does not guarantee that the model will perform well in more complex scenarios requiring multi-needle synthesis reasoning or handling contradictory/distracting information in context.





![Comparison of single-needle retrieval vs multi-needle reasoning in long-context evaluation. (长文本评估中单针检索与多针推理的区别对比示意图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig02_needle_haystack_limits.png)

</details>

**23. How to evaluate the multi-turn dialogue capability of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Using benchmarks like MT-Bench and MultiWOZ, capabilities such as context memory retention, anaphora resolution, topic switching adaptability, multi-turn consistency, and coherence are evaluated. Multi-turn dialogue quality can also be scored by human evaluators or judge models.

</details>

**24. In multi-turn dialogue scenarios, what dimensions should evaluation focus on that single-turn evaluation cannot cover?**

<details>
<summary>💡 Show Answer</summary>

These include: context memory capability (whether information from previous turns is correctly utilized), dialogue consistency (whether answers contradict previous ones), error correction capability (whether the model can correctly adjust after the user points out an error), topic switching handling (whether the model can correctly identify a user's shift in topic without forcibly linking to previous content), and whether performance/quality degrades as turns increase in long conversations.

</details>

**25. How to evaluate the factual accuracy of model-generated content?**

<details>
<summary>💡 Show Answer</summary>

Using benchmarks/metrics such as TruthfulQA and FActScore, factual claims in generated content are verified via retrieval augmentation or external knowledge bases. Human annotation or judge models can also be used to verify the truthfulness of individual claims line by line to calculate hallucination rates.

</details>

**26. How to evaluate the "Factuality" of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Usually, a test set containing explicit factual questions (with verifiable answers) is constructed to evaluate whether the factual statements in the model's answers are correct. For systems combined with retrieval, it is also necessary to evaluate whether the generated content is faithful to the retrieved source content (avoiding hallucinations even when correct references are provided).

</details>

**27. What are the reasons for LLMs generating hallucinations?**

<details>
<summary>💡 Show Answer</summary>

The main reasons include: erroneous or outdated information in training data; the training objective (next-token prediction) encouraging fluent but not necessarily true content; the model's lack of awareness of its own knowledge boundaries; "fabrication" caused by missing retrieval/knowledge; decoding strategies (e.g., high-temperature sampling) increasing randomness; error accumulation in long chain reasoning; and alignment training leading models to tend toward providing answers that "appear confident."

</details>

**28. What methods can be used to mitigate model hallucinations?**

<details>
<summary>💡 Show Answer</summary>

Retrieval-Augmented Generation (RAG), introducing external tools for verification, prompting models to express uncertainty, incorporating factual preference data during the SFT/RLHF stages, Chain-of-Thought and self-consistency verification, source citation labeling, knowledge editing and post-hoc correction, and using judge models or rule-based output filtering.

</details>

**29. How to design evaluations to measure model "Calibration"?**

<details>
<summary>💡 Show Answer</summary>

Calibration measures whether a model's expressed confidence matches its actual accuracy (e.g., when the model claims "I am 90% confident," whether the actual accuracy is close to 90%). The evaluation method typically involves asking the model to provide answers alongside corresponding confidence levels for a set of questions (via logprobs or explicitly outputting numerical confidence scores). Questions are binned by confidence level to compute the actual accuracy within each bin, plotting a Reliability Diagram and calculating metrics such as Expected Calibration Error (ECE).





![Reliability diagram showing model calibration curves: perfect calibration, overconfidence, and underconfidence. (模型校准曲线置信度与准确率的对照图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig03_model_calibration_curve.png)

</details>

**30. How to evaluate the instruction-following capability of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Using benchmarks such as IFEval and FollowBench, instructions containing multi-constraint conditions (format, word count, keywords, structural requirements, etc.) are designed to test whether the output satisfies all constraints. Human evaluators or judge models can also score instruction comprehension accuracy and execution completeness.

</details>

**31. How to design evaluations specifically targeting "instruction-following capability"?**

<details>
<summary>💡 Show Answer</summary>

Typically, instructions containing multiple concrete constraints are constructed (e.g., "Answer in no more than 50 words and list three points in bullet format"). The model's output is checked to see if it satisfies all explicit constraint conditions simultaneously. Rule-based automatic verification (checking whether word count and format meet requirements) can be combined with LLM-as-Judge to evaluate content quality.

</details>

**32. What considerations should be kept in mind when comparing different LLMs?**

<details>
<summary>💡 Show Answer</summary>

It is essential to ensure consistent evaluation conditions, including prompt templates, few-shot settings, decoding parameters (temperature, sampling strategies), and evaluation script versions; pay attention to differences in model scale and training data cutoff dates; avoid one-sided conclusions from a single benchmark by synthesizing multi-dimensional results; and stay vigilant regarding test set contamination or target-specific optimization.

</details>

**33. What considerations are important when evaluating small models?**

<details>
<summary>💡 Show Answer</summary>

Small models generally perform weakly on tasks requiring complex reasoning or long context. Evaluations should focus more on their practical utility in specific scenarios (on-device deployment, domain-specific tasks). Prompt complexity should be controlled to prevent distorted results caused by weak instruction-following capability. Attention should also be given to trade-offs between accuracy and efficiency metrics such as inference speed and VRAM footprint.

</details>

**34. What evaluation metrics are commonly used for domain-specific models?**

<details>
<summary>💡 Show Answer</summary>

The medical domain commonly uses professional licensing examination questions (such as medical practitioner test banks) and diagnostic accuracy; the legal domain uses bar examination questions and case analysis accuracy; the financial domain uses financial report comprehension and risk analysis accuracy. Additionally, general metrics include domain terminology understanding, professional Q&A accuracy, compliance, and safety.

</details>

**35. How to evaluate multilingual capabilities?**

<details>
<summary>💡 Show Answer</summary>

Multilingual benchmarks such as XTREME, Flores, and mMMLU are used to evaluate translation quality, cross-lingual knowledge Q&A, and multilingual instruction-following capabilities. Attention should also be paid to performance gaps in low-resource languages, as well as cross-lingual consistency (whether answers to the same question across different languages are consistent).

</details>

**36. What is "multilingual consistency" evaluation?**

<details>
<summary>💡 Show Answer</summary>

It evaluates whether a model's performance capability (accuracy) and response content (such as viewpoints and factual statements) remain consistent when answering the same or corresponding questions across different languages, used to identify capability gaps in low-resource languages or cross-lingual inconsistencies.

</details>

**37. How to evaluate model bias and ethical issues?**

<details>
<summary>💡 Show Answer</summary>

Benchmarks such as BBQ, StereoSet, and CrowS-Pairs are used to detect stereotypes and bias tendencies across dimensions like gender, race, and religion. Bias severity is quantified by comparing differences in model outputs under descriptions of different demographic groups, combined with human auditing of responses to sensitive topics for ethical compliance.





![Evaluation pipeline for quantifying model bias and stereotypes across demographic groups. (用于量化人口统计学维度下模型偏差与刻板印象的系统评估流程图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig04_bias_evaluation_pipeline.png)

</details>

**38. How to analyze the capability boundaries of LLMs?**

<details>
<summary>💡 Show Answer</summary>

By testing performance curves across task difficulty gradients, the critical points where performance drops sharply can be identified. The decay patterns of model performance under out-of-distribution (OOD) data, long-tail knowledge, and increasing multi-step reasoning depth are analyzed. Probing experiments can also be conducted to analyze how internal representations support specific capabilities.

</details>

**39. How to evaluate model alignment efficacy?**

<details>
<summary>💡 Show Answer</summary>

Via human preference comparisons (A/B testing, win rate statistics), reward model scoring, safety and toxicity testing, instruction-following evaluations, and assessing the model's overall performance on value alignment (e.g., helpfulness, harmlessness, honesty).

</details>

**40. How to evaluate preference alignment effectiveness?**

<details>
<summary>💡 Show Answer</summary>

A common method is constructing paired responses for annotators or judge models to select the superior answer, calculating the aligned model's win rate relative to a baseline model. Changes in reward model score distributions can also be calculated, KL divergence used to measure deviation from the base model, and separate evaluations conducted across multiple preference dimensions (helpfulness, harmlessness, honesty).

</details>

**41. How do evaluation results guide subsequent model optimizations?**

<details>
<summary>💡 Show Answer</summary>

Based on shortcomings exposed during evaluation (e.g., weak math reasoning, long-context performance degradation, poor specific language skills), training data can be supplemented in a targeted manner or dedicated fine-tuning tasks designed. Error analysis on failure samples helps locate whether the root cause is knowledge deficiency, insufficient reasoning capability, or instruction understanding failure. Finally, evaluation metrics are incorporated into regression testing during iterative development to prevent performance regression on existing capabilities in new versions.

</details>

**42. What is the "disconnect between metrics and user experience" problem in evaluation? Why does it occur?**

<details>
<summary>💡 Show Answer</summary>

It refers to the phenomenon where a model achieves outstanding performance on automated metrics (such as benchmark scores), yet provides poor real-world user experience. Reasons include: automated metrics are often designed for specific task formats and fail to comprehensively cover dimensions like diversity, coherence, and safety in real interactions; moreover, models may undergo "teaching-to-the-test" optimizations tailored to specific benchmark formats, which do not necessarily generalize to real-world scenarios.

</details>

**43. What is the "format sensitivity" problem in evaluation?**

<details>
<summary>💡 Show Answer</summary>

It refers to substantial fluctuations in a model's score caused by minor variations in prompt phrasing, option ordering, or output format requirements. These minor changes should not fundamentally alter the model's "true capability," yet they significantly impact actual scoring, reflecting instability in evaluation results.

</details>

**44. What is the "reference answer dependence" problem in evaluation? How can it be handled for open-ended generation tasks?**

<details>
<summary>💡 Show Answer</summary>

Many evaluation methods (e.g., Exact Match, BLEU) rely on literal matching with standard reference answers. However, open-ended generation tasks (e.g., creative writing, explanations) may have multiple equally correct but differently phrased answers, leading these metrics to underestimate actual quality. Approaches to handle this include using semantic similarity evaluation, employing LLM-as-Judge to score based on rubrics (rather than comparing against a single reference answer), or providing multiple reference answers.

</details>

**45. What is the "Chain-of-Thought Leakage" problem in evaluation, and how does it affect evaluation results?**

<details>
<summary>💡 Show Answer</summary>

It refers to situations in certain evaluation setups where the model's intermediate reasoning process (Chain-of-Thought) itself may leak hints about the final answer, or where the evaluation's answer extraction logic fails to properly handle the Chain-of-Thought format (e.g., erroneously extracting non-final answer fragments from a lengthy reasoning chain), leading to errors in scoring results. This requires that answer extraction/parsing logic in evaluation must strictly match the model's actual output format (including whether it contains Chain-of-Thought) and be thoroughly tested.

</details>

**46. What is Capability-Specific Evaluation? Why is it necessary to break down evaluation rather than relying solely on overall scores?**

<details>
<summary>💡 Show Answer</summary>

Capability-specific evaluation designs tests and reports scores separately for different capability dimensions of a model (such as math reasoning, code generation, knowledge Q&A, safety, instruction following). An overall score may mask severe shortcomings in specific dimensions (e.g., a high overall score despite poor safety performance). Capability-specific evaluation helps pinpoint model strengths and weaknesses with finer granularity.

</details>

**47. What is "Task Completion Rate" in evaluation? What types of systems is it commonly used to evaluate?**

<details>
<summary>💡 Show Answer</summary>

Task Completion Rate refers to the proportion of given tasks in which a system (especially Agent-based systems) successfully achieves the task goal. It is the most direct metric for end-to-end evaluation of Agent/tool-use systems, typically requiring clearly defined, automatically or semi-automatically verifiable "completion" criteria for each task.

</details>

**48. What is "Consistency" evaluation in LLMs, and how is it designed?**

<details>
<summary>💡 Show Answer</summary>

Consistency evaluation measures whether a model produces consistent answers when presented with inputs that have identical semantics but different phrasings (avoiding self-contradictions). A common design method is constructing multiple paraphrased versions of the same question (synonym paraphrasing, sentence restructuring) and observing whether the model's responses across these versions remain consistent in facts and conclusions.

</details>

**49. What are End-to-End Evaluation and Component-Level Evaluation? How do they work together?**

<details>
<summary>💡 Show Answer</summary>

End-to-end evaluation focuses on the value of the overall system's (e.g., RAG/Agent systems) final output to the user. Component-level evaluation separately assesses the performance of individual modules within the system (e.g., retrieval module, generation module, tool-calling module). Used together, component-level evaluation can locate which specific module is causing issues after an end-to-end evaluation reveals a problem, enabling targeted optimizations.

</details>

