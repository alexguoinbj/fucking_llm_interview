# 8.7 Evaluation Engineering Practice: System Design, Cost, and Online Monitoring — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 11 questions

**1. What is "Cost-Effectiveness" Trade-Off Analysis in Evaluation? Why is It Important in Model Selection?**

<details>
<summary>💡 Show Answer</summary>

It refers to evaluating the overall "cost-performance ratio" when comparing different models by taking into account not only their capability scores but also their inference costs (per-token unit price, latency). In practical product decision-making, a model with slightly lower capability but significantly lower cost may offer higher practical value in many scenarios than a top-tier large model; thus, cost-effectiveness trade-off is an indispensable dimension in model selection evaluation.

</details>

**2. How to Design an "Internal Evaluation Dataset" Tailored to a Specific Business Scenario (e.g., Customer Service Dialogue)?**

<details>
<summary>💡 Show Answer</summary>

First, sample representative instances from real historical interaction data to cover different user intent distributions and difficulty levels (including common scenarios and known edge cases); define clear evaluation criteria for each sample (which can be rules, reference answers, or scoring dimensions for open-ended responses); the evaluation dataset should be updated periodically to reflect business evolution (new scenarios, new question types) and calibrated with human evaluation to verify the effectiveness of automated evaluation methods.

</details>

**3. What is "Meta-Evaluation" in Evaluation Systems? Why is It Needed?**

<details>
<summary>💡 Show Answer</summary>

Meta-evaluation refers to evaluating the validity of evaluation methods themselves—for example, assessing the correlation between scoring results from an automated metric/LLM-as-a-Judge and the "true quality" (typically proxied by thorough human evaluation). Meta-evaluation is needed because automated evaluation methods themselves may exhibit systematic biases; if the evaluation method itself is not validated, model optimization decisions based on biased evaluation results may also be biased.

</details>

**4. What is the Perspective of "Eval as a Moat"? Why Do Some Believe a High-Quality Evaluation System Itself Possesses Competitive Barrier Value?**

<details>
<summary>💡 Show Answer</summary>

This perspective posits that as model capabilities across providers converge, an evaluation system capable of accurately reflecting a "model's real performance under specific business scenarios" (comprising business-aligned evaluation data, rubric criteria, and continuous update mechanisms) is itself an asset difficult to replicate. It enables teams to determine more quickly and accurately which model or iteration truly brings business value improvements, thereby establishing a relative advantage in model selection and optimization iteration speed. This "evaluation capability" itself holds more sustainable competitive barrier value than merely utilizing any single model.

</details>

**5. When Conducting Human Evaluation, How to Design Reasonable Evaluation Criteria and Workflows to Ensure Objectivity and Consistency of Results?**

<details>
<summary>💡 Show Answer</summary>

In human evaluation, ensuring the objectivity and consistency of results is the greatest challenge because human judgment is inherently subjective. Designing reasonable evaluation criteria (rubrics) and workflows is key to overcoming this challenge.

1. Designing Reasonable Evaluation Criteria (Rubrics):

1. Clear and Atomic Dimensions:

- Avoid using vague words like "good" or "bad." Decompose "quality" into multiple mutually independent, concrete dimensions. For example:
- Accuracy: Does the answer contain factual errors?
- Completeness: Does the answer comprehensively address all aspects of the question?
- Conciseness: Is there redundant information?
- Harmlessness: Does it contain harmful content?

1. Quantitative Rating Scale:

- Use quantitative scales such as Likert scales (1–5 points) or binary judgements (Yes/No).
- Provide clear, unambiguous definitions for each score tier. For instance, for the accuracy dimension: 5 points = Completely accurate; 4 points = Basically accurate with minor flaws; 3 points = Contains noticeable but non-core errors...; 1 point = Completely wrong.

1. Providing Abundant Examples:

- Provide typical golden examples and counter-examples for each score level across every dimension. This significantly helps annotators calibrate their rating standards.

2. Designing Reasonable Evaluation Workflows:

1. Rater Training and Calibration:

- Before evaluation starts, systematically train all annotators to ensure they fully understand the evaluation rubrics and all definitions.
- Hold calibration sessions where all annotators score the same batch of samples, followed by open discussion and alignment of scoring discrepancies until everyone's understanding converges.

1. Blind Evaluation:

- Annotators should not know which model (Model A, Model B, or human) produced the response they are evaluating. This eliminates brand bias or preconceived notions.

1. Multiple Independent Ratings & Consistency Check:

- Each sample should be independently evaluated by at least 2–3 annotators.
- Use statistical metrics to measure Inter-Annotator Agreement (IAA), such as Cohen's Kappa or Fleiss' Kappa.
- If the IAA is too low, it indicates ambiguity in the evaluation rubrics, requiring a return to step 1 for revisions.

1. Adopting Pairwise Comparison Rather Than Absolute Rating:

- For scenarios comparing two models (A vs. B), asking human evaluators to judge "which is better" (A is better / B is better / Tie) is usually easier and more reliable than having them assign absolute scores to A and B individually. This method effectively reduces variance in individual scoring scales.

1. Establishing an Adjudication Mechanism:

- For "difficult cases" with significant disagreement among annotators, a higher-level expert or committee is needed to perform final adjudication to ensure the authority of the final results.





![Standardized double-blind human annotation workflow with agreement auditing. (确保客观性与一致性的人工双盲评测质量控制标准流程图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig28_human_eval_workflow.png)

</details>

**6. How to Design an Evaluation Scheme to Measure Specific Capabilities of LLMs, Such as "Factuality/Hallucination Level," "Reasoning Capability," or "Safety"?**

<details>
<summary>💡 Show Answer</summary>

Designing an evaluation scheme to measure specific capabilities of an LLM requires following the workflow: "Define Capability $\rightarrow$ Construct Dataset $\rightarrow$ Determine Evaluation Method".

1. Measuring "Factuality/Hallucination Level":

- Capability Definition: Whether the model's generated responses are grounded in verifiable facts rather than fabricated information.
- Dataset Construction:
- Knowledge-Base-Based QA: Construct a question set where the answer to each question can be found in a deterministic knowledge source (e.g., Wikipedia, internal company documents, databases).
- Adversarial Questions: Design questions that induce the model to hallucinate, such as asking about non-existent figures or events.
- Evaluation Methods:
- Exact Match / Keyword Match: For factually simple questions (e.g., "Who is the current President of Singapore?"), entities in the generated answer can be directly compared against standard answers.
- LLM-as-a-Judge: Use a stronger LLM to judge whether the generated response aligns with or contradicts the provided ground-truth knowledge.
- Automated Frameworks: Use metrics like FaithScore or Faithfulness in RAGAS, which automatically cross-verify each claim in the generated answer against the context.

1. Measuring "Reasoning Capability":

- Capability Definition: Whether the model can derive correct conclusions through multi-step deductions using logic, mathematics, or common sense in the absence of direct knowledge.
- Dataset Construction:
- Use dedicated reasoning benchmarks such as GSM8K (math word problems), LogiQA (logical reasoning), or selected tasks from Big-Bench Hard.
- Custom-design tasks requiring specific reasoning pathways—for example, providing a series of premises and asking the model to infer the conclusion.
- Evaluation Methods:
- Outcome-Based Evaluation: Only judge whether the final answer is correct. This is the most direct method.
- Process-Based Evaluation: For models using Chain-of-Thought (CoT), evaluate not only the final answer but also whether the reasoning steps are logical and correct via humans or another LLM. This provides deeper insight into the model's reasoning process.

1. Measuring "Safety":

- Capability Definition: Whether the model can refuse to answer harmful, unethical, dangerous, or illegal user requests.
- Dataset Construction:
- Use public adversarial prompt datasets such as AdvBench (Adversarial Benchmarks) or SafetyBench, which contain a large number of engineered "dangerous questions" attempting to bypass safety guardrails.
- Build new offensive prompts proactively and creatively through human expert red teaming.
- Evaluation Methods:
- Classifier Evaluation: Input the model's response into a pre-trained safety classifier (typically another LLM or a specialized classification model) to determine whether it falls into categories like "harmful," "refusal," or others.
- Core Metrics:
- Refusal Rate: The proportion of harmful questions the model successfully refuses to answer.
- False Refusal Rate: The proportion of normal, safe questions the model incorrectly refuses to answer.
- Human Evaluation: For ambiguous or novel cases, human review remains the ultimate gold standard.





![Methodological framework for designing target capability evaluation schemes. (针对特定能力进行评测方案设计的通用方法论框架图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig29_capability_eval_design.png)

</details>

**7. What is the "Reproducibility" Challenge in Evaluation? What Factors Affect the Reproducibility of Evaluation Results?**

<details>
<summary>💡 Show Answer</summary>

Influencing factors include: randomness in model inference (sampling temperature greater than 0 without a fixed seed), subtle differences in prompt templates (including spaces and line breaks), variations in evaluation framework versions or answer extraction logic, and version updates of the model service itself (the actual weights or service implementation behind the same model ID may change over time). To guarantee reproducibility, all these variables must be fixed and evaluation configurations meticulously documented.

</details>

**8. How to continuously monitor and evaluate the performance of a deployed LLM application or Agent service to deal with potential performance degradation or behavior drift?**

<details>
<summary>💡 Show Answer</summary>

Continuously monitoring and evaluating a deployed LLM application or Agent service is a proactive and iterative process aimed at addressing Model Drift and Data Drift to ensure consistent service quality.

Data drift refers to changes in the distribution of input data in the production environment (e.g., users start asking novel types of questions), whereas model drift refers to the decline in a model's predictive capabilities caused by data drift.

A comprehensive monitoring and evaluation system should encompass the following dimensions:

1. Collection and Logging:

- Full Logging: Record complete interaction data for every request, including user inputs, intermediate generation steps (such as the Agent's Chain-of-Thought), final outputs, invoked tools, latency, Token consumption, etc.
- User Feedback: Embed explicit user feedback mechanisms within the product UI, such as "thumbs up/thumbs down" buttons, ratings, or one-click issue reporting. This provides the most direct performance signals.

1. Automated Monitoring:

- Monitoring Proxy Metrics: Track automatically computable metrics that are highly correlated with performance. Abnormal fluctuations in these metrics often serve as early warning signs of issues.
- Input Metrics: Query length, topic distribution, language of questions, etc.
- Output Metrics: Response length, code block ratio, JSON format error rate, refusal rate, etc.
- Process Metrics (for Agents): Average number of execution steps, tool invocation frequency, tool invocation failure rate.
- Automated Quality Evaluation:
- Periodic Sampling: Randomly sample a small portion of production traffic.
- LLM-as-a-Judge: Use a powerful "judge LLM" to automatically score sampled cases based on a fixed set of evaluation criteria (e.g., toxicity/harmfulness, off-topic relevance).
- Comparison against Golden Set: Compare sampled responses against an internally maintained, high-quality "golden evaluation set" to verify whether model performance remains stable on key benchmarks.

1. Human Review and Analysis:

- Regular Human Audits: Periodically organize operations or evaluation teams to perform deep-dive human analyses on random production samples, negative feedback cases (bad cases), and abnormal cases flagged by automated monitoring.
- Root Cause Analysis: For identified issues, perform in-depth analysis to pinpoint which component failed: Has the LLM's capability degraded? Is the Agent's planning logic faulty? Or has a tool API changed?

1. Feedback Loop and Model Iteration:

- Continuous Data Management: Clean, annotate, and continuously add valuable cases discovered in production (especially failure cases and disliked responses) into evaluation sets and fine-tuning datasets.
- Periodic Retraining/Fine-tuning: Periodically fine-tune or retrain models based on accumulated new data to adapt to shifting data distributions and user requirements.
- A/B Testing: When deploying new model versions or Agent logic, use an A/B testing framework with small-scale traffic to verify whether the new version outperforms the old one, ensuring every iteration brings positive improvements.

By establishing such a closed loop of "Collection -> Monitoring -> Analysis -> Iteration", we can proactively manage and maintain online service quality rather than reactively waiting for user complaints.

</details>

**9. What are the challenges brought by "Model Drift" in evaluation?**

<details>
<summary>💡 Show Answer</summary>

For closed-source models accessed via APIs, even if the model version number remains unchanged, service providers may update models in the backend (such as adjusting safety policies or optimizing inference performance). Consequently, the behavior of the same model on identical inputs can vary across different points in time. This poses significant challenges for long-term evaluation comparison and monitoring—requiring periodic re-runs of core evaluation benchmarks, as well as considering whether observed performance trend changes stem from changes in the model itself or shifts in the benchmark/data.





![Impact of continuous version updates and RLHF alignment on model capability drift. (大模型持续迭代与 RLHF 过程中能力偏移（Model Drift）对评估的影响图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig30_model_drift_evaluation.png)

</details>

**10. When evaluating multiple candidate models for production replacement, what factors should be comprehensively considered besides capability metrics?**

<details>
<summary>💡 Show Answer</summary>

Factors include: cost (price per Token, overall invocation costs), latency and throughput stability, API stability and service availability SLAs, level of support and reliability for specific output formats (such as Function Calling, structured outputs), safety and compliance (whether data will be used for training, compliance with industry requirements), and the compatibility impact of switching models on existing prompts and system infrastructure (how much adaptation effort is required).

</details>

**11. Looking forward, in what directions might LLM evaluation systems evolve?**

<details>
<summary>💡 Show Answer</summary>

Potential trends include: evolving from static benchmark datasets toward dynamically generated and continuously updated evaluation content (mitigating data contamination); evolving from single metrics toward multi-dimensional, scenario-specific "evaluation profiles" (combining specific business evaluation configurations); tighter integration between evaluation and real-world production monitoring (forming a closed loop between online A/B testing and offline evaluation); and, with the widespread adoption of Agent systems, expanding the evaluation subject from "a single model output" to "the overall performance of the system during long-term, multi-step interactions with real environments."

</details>

