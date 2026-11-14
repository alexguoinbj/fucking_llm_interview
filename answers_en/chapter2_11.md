# 2.11 Prompt Evaluation and A/B Testing — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 37 questions

**1. Why is it necessary to evaluate Prompts?**

<details>
<summary>💡 Show Answer</summary>

Prompts are the core configuration of LLM applications, directly determining the quality boundaries of the model's output. Unlike traditional software, where code logic is verified through unit testing, the effect of a Prompt has **high context dependence and statistical uncertainty**, making systematic evaluation an engineering necessity rather than an option.

#### The Necessity of Evaluation

- **Intuitive judgment is unreliable**: The effects of Prompt modifications are often counterintuitive—what seems like a clearer expression may lead to performance degradation, while seemingly redundant constraints might be crucial for a certain type of input. There is a fundamental cognitive gap between human subjective perception of natural language and the LLM's token-level processing mechanism. The correct rate of modification decisions made solely by developer intuition is far lower than expected, so data must serve as the basis for decisions.

- **LLM output randomness requires statistical significance**: An LLM's output is not fixed under the same input (influenced by temperature parameters and sampling strategies). Conclusions drawn from a single observation are not statistically representative. For example, if a new Prompt yields better output quality in one test, it might just be the result of sampling randomness rather than a genuine systemic improvement. Reliable conclusions can only be drawn by multiple samplings over a sufficiently large test set and conducting statistical significance tests.

- **The standard for "good" varies by task and needs explicit definition**: "Good" in a summarization task is information completeness and conciseness; "good" in a code generation task is functional correctness and code style; "good" in customer service dialogue is problem resolution rate and user satisfaction. The evaluation metrics corresponding to different tasks are completely different. If evaluation standards are not explicitly defined in advance, the "optimization" direction might deviate completely from actual business goals.

- **Regression Prevention**: In iterative development, Prompt optimization for one type of problem might inadvertently degrade performance on other types of inputs. Without a systematic evaluation framework, regressions are often only discovered when they trigger user complaints after deployment, resulting in extremely high fixing costs. A perfect evaluation framework is a safety net for continuous iteration.

- **Multi-model/multi-version comparative decision making**: In the LLM ecosystem, models iterate rapidly (e.g., from GPT-4o to GPT-4o-mini, from Claude 3 to Claude 3.5). The performance difference of the same Prompt across different models can be significant. When facing decisions like "whether to switch foundation models" or "whether a new model version is worth migrating to," systematic evaluation is the only reliable basis for judgment.

- **Quantitative trade-offs between cost and quality**: Prompt modifications are often accompanied by changes in token usage, which in turn affect API call costs. The evaluation system should be able to simultaneously quantify quality improvements and cost changes, providing data support for decisions like "whether using more tokens for higher quality is worth it," rather than solely focusing on the quality dimension.

#### Typical Risks of Lacking Evaluation

Here are common engineering failure modes when lacking systematic Prompt evaluation:

1. **Cherry-picking Trap**: Developers only test a new Prompt on a few examples that "look good," only to find it fails massively on long-tail inputs after deployment;
2. **Silent Regression**: A certain "harmless format tweak" quietly lowers the accuracy of a specific sub-category of inputs. Because there is no evaluation baseline, the issue is not noticed until weeks later;
3. **Overfitting Fixes**: Modifying the Prompt for a specific case reported in user feedback, which improves that case but introduces systemic interference for other cases;
4. **Out-of-Control Model Upgrades**: When migrating an application from an old model to a new one, the lack of benchmark testing makes it impossible to judge whether the migration brings overall benefits, nor can it identify which scenarios have experienced degradation.

#### The Position of Evaluation in the Prompt Engineering Lifecycle

Systematic evaluation should permeate the **entire lifecycle** of Prompt engineering, not just be conducted before deployment:

1. **Baseline Establishment Phase**: Immediately after the first version of the Prompt is completed, build an evaluation test set and record baseline metrics as a comparison baseline for all subsequent iterations;
2. **Iterative Development Phase**: After every Prompt modification, run a rapid evaluation on the local evaluation set to confirm the modification direction is correct before proceeding;
3. **Pre-deployment Verification Phase**: Run full evaluation on the complete test set to confirm through statistical testing that the new version is significantly better than the baseline and has no sub-category regressions;
4. **Continuous Online Monitoring Phase**: Continuously collect quality signals (user feedback, auto-scoring) of real requests in the production environment, monitoring whether evaluation metrics drift over time (e.g., performance degradation due to shifts in user behavior distribution).

</details>

**2. Why do Prompts perform unstably under different inputs?**

<details>
<summary>💡 Show Answer</summary>

The model might produce vastly different outputs given minor changes in the input (phrasing, order), especially when ambiguous instructions or multiple possible interpretations exist in the Prompt, amplifying this sensitivity.

#### Root Cause
Large language models are inherently probabilistic models, and their outputs are statistical approximations of the training data distribution. Even for semantically similar inputs, different attention patterns might be activated at the Token sequence level, leading to branching in the decoding path.

#### Primary Triggers

1. **Word Order Sensitivity**: Models possess a natural sensitivity to Token positions. For instance, placing constraints at the beginning versus the end of a Prompt can result in vastly different levels of attention from the model.
2. **Phrasing Ambiguity**: Terms like "briefly answer" might be interpreted as "one sentence" or "one paragraph" depending on the input; vague qualifiers force the model to fill in semantic gaps on its own.
3. **Multiple Interpretation Paths**: When a Prompt contains multiple potential goals simultaneously (e.g., "be concise, yet comprehensive"), the model leans towards different trade-off strategies depending on the input.
4. **Context Pollution**: When the distribution of Few-shot examples does not match the actual input, the examples "anchor" the model to incorrect output patterns.
5. **Temperature and Sampling Randomness**: Even with exactly identical Prompts and inputs, a non-zero temperature parameter introduces randomness in the output, which is particularly evident in edge cases.

#### Typical Manifestation

Below is an example of an unstable Prompt where the exact same instruction yields inconsistent format outputs given slightly differently phrased inputs:

\begin{lstlisting}[style=promptstyle]
	# Prompt (Original)
	Extract the key information from the following text and output it as a list.

	# Input A (Clearly structured)
	Zhang San, 30 years old, software engineer, Beijing.
	# Output A: Outputs as a Markdown list, neatly formatted

	# Input B (Natural expression)
	Zhang San is thirty years old this year and works in software development in Beijing.
	# Output B: Outputs as a natural language paragraph, failing to follow the list requirement
\end{lstlisting}

#### Improvement Strategies

- **Explicit Constraints**: Change implicit requirements into clear formatting specifications, such as replacing "output as a list" with "Output in JSON format with fields name/age/city".
- **Add Output Anchors**: Append an output-guiding prefix at the end of the Prompt (like \texttt{Output: \{} or ““json`) to limit the model's room for freeform generation.
- **Cover Diverse Few-shots**: Ensure examples cover both structured and unstructured input forms to reduce the model's sensitivity to input style.
- **Set Low Temperature**: For scenarios requiring stable outputs, set `temperature` to 0 or close to 0.

</details>

**3. Why are Prompts fine-tuned for specific Cases prone to "overfitting"?**

<details>
<summary>💡 Show Answer</summary>

Adding special rules or phrasing tailored specifically to individual examples makes the Prompt perform well on those examples, but degrades its generalization to other similar yet non-identical inputs; essentially, the rules become too specific.

#### Analogy to Overfitting in Machine Learning

In traditional machine learning, overfitting refers to a model performing excellently on the training set but poorly on the test set. Prompt overfitting is highly similar: developers repeatedly observe a few failed examples and patch them specifically, which equates to overfitting on the "training set" (a small number of observed cases) while ignoring the broader input distribution.

#### Typical Causes of Overfitting

1. **Appending Excessive Special Rules**: Every time a failed case is encountered, a new rule is appended to the Prompt. These rules might conflict with each other and are hard to scale to cover all variants.
2. **Lack of Diversity in Few-shot Examples**: The selected examples are overly concentrated on a certain input pattern, causing the model to adapt highly to that pattern while deviating on others.
3. **Phrasing Coupled with Specific Inputs**: The Prompt contains keywords or sentence structures highly related to specific cases, so the model learns to "pattern-match" rather than understand the intent.
4. **Lack of Systematic Test Sets**: The tuning process relies entirely on subjective observation and a few examples, without verifying generalization effects on a diverse test set.

#### Overfitting Example

\begin{lstlisting}[style=promptstyle]
	# Prompt before overfitting (Generic)
	Classify the user input as "Complaint" or "Inquiry".

	# Observed Failed Case:
	User input "My order has been delayed for a long time" was misclassified as "Inquiry".

	# Prompt after overfitting (Piling up special rules)
	Classify the user input as "Complaint" or "Inquiry".
	Note: If it mentions "order" and includes "delayed" or "for a long time", it must be classified as "Complaint".
	Note: If it mentions "express delivery" and includes "not received", it must be classified as "Complaint".
	Note: If it mentions "waited for a long time", it must be classified as "Complaint".
	# Problem: The new input "It seems my package was sent to the wrong address" still cannot be covered.
\end{lstlisting}

#### Correct Tuning Approach

You should extract **abstract rules** from specific failed cases, rather than adding concrete rules:

\begin{lstlisting}[style=promptstyle]
	# Correct improvement method: Distill intent rather than enumerating scenarios
	Classify the user input as "Complaint" or "Inquiry".

	Classification Criteria:
	- Complaint: The user expresses dissatisfaction with an issue that has already occurred or seeks a resolution, usually containing negative emotions or describing a service failure.
	- Inquiry: The user seeks information, wants to understand rules, or asks how to operate, usually without negative emotions.
\end{lstlisting}

#### Methods to Prevent Overfitting

- **Build Diverse Test Sets**: Construct a test set covering multiple input patterns before tuning, and run regression verification on the entire test set after each modification.
- **Rule Abstraction**: When encountering a failed case, first think about "what common rule does this case represent?" instead of "how do I handle this case?".
- **Control Few-shot Example Diversity**: Ensure examples are representative in length, style, and domain to prevent the example distribution from being overly concentrated.
- **Peer Review**: Have personnel not involved in tuning test the Prompt with new inputs to discover generalization blind spots.

</details>

**4. What are the common evaluation dimensions for Prompts?**

<details>
<summary>💡 Show Answer</summary>

- **Accuracy**: Whether the output matches facts/expected answers.
- **Format Compliance**: Whether it outputs in a specific format as requested (JSON, Markdown, etc.).
- **Completeness**: Whether it covers all necessary information points.
- **Conciseness**: Whether there is no redundant content.
- **Safety**: Whether it triggers harmful outputs.
- **Consistency**: Whether multiple run results are stable.
- **Task Success Rate**: In task-oriented scenarios, whether the goal is successfully completed.

These dimensions are not independent; trade-offs often exist in actual evaluation. Below are detailed explanations of the evaluation methods and considerations for each dimension.

#### Accuracy

Accuracy is the core evaluation dimension, measuring how well the output matches the correct answer.

- **Objective Questions Scenario**: Can be automatically calculated via Exact Match or F1 score.
- **Subjective Questions Scenario**: Requires introducing human evaluation or the LLM-as-Judge method, where another model scores the output.
- **Considerations**: For generative tasks, the "correct answer" itself may not be unique, necessitating a pre-defined grading standard (Rubric).

#### Format Compliance

Measures whether the output strictly follows the specified structural requirements; can be fully automated for evaluation.

\begin{lstlisting}[style=promptstyle]
	# Format compliance check example (Python pseudocode)
	import json

	def check_format(output: str) -> bool:
	try:
	parsed = json.loads(output)
	required_keys = {"name", "age", "city"}
	return required_keys.issubset(parsed.keys())
	except json.JSONDecodeError:
	return False
\end{lstlisting}

#### Completeness

Measures whether the output covers all information points required by the task, usually quantified by **information point coverage**:

\[
Completeness Score = \frac{Number of necessary info points covered in output}{Total number of necessary info points required by task}
\]

#### Conciseness

Measures the proportion of redundant content in the output. Overly lengthy outputs not only increase Token costs but may also dilute key information. Evaluation methods include:
- Ratio of output length to reference answer length
- Human judgment on whether invalid repetitions or irrelevant content exist

Note that a natural tension exists between completeness and conciseness, so their respective weights must be determined based on task characteristics.

#### Safety

Evaluates whether the Prompt easily induces harmful, violating, or sensitive outputs from the model, especially its robustness against adversarial inputs. Evaluation methods include:
- Constructing Jailbreak and injection attack test cases
- Using safety classifiers to automatically scan outputs
- Manually reviewing output samples in high-risk scenarios

#### Consistency

Sampling the same input multiple times (usually 5-20 times) to measure the stability of the output. Common metrics:
- **Semantic Consistency**: Uses text similarity (e.g., BERTScore, cosine similarity) to measure the semantic distance between multiple outputs.
- **Structural Consistency**: In format-based tasks, counts the proportion of sampled outputs that comply with the format.
- **Conclusion Consistency**: In classification or judgment tasks, counts the percentage of the majority answer.

#### Task Success Rate

In goal-oriented task scenarios (e.g., information extraction, code generation, tool invocation), using **whether the goal is achieved end-to-end** as a binary evaluation standard is the most direct business-oriented metric.

\[
Task Success Rate = \frac{Number of samples that successfully completed the goal}{Total number of test samples} \times 100%
\]

#### Scenario-Based Configuration of Dimension Weights

Different task scenarios place different emphasis on these dimensions; it is recommended to explicitly configure weights based on actual business needs:

\begin{center}


| **Scenario** | **Accuracy** | **Format** | **Completeness** | **Conciseness** | **Safety** |
| --- | --- | --- | --- | --- | --- |
| Knowledge QA | High | Low | Medium | Medium | Medium |
| Structured Extraction | Medium | High | High | Low | Low |
| Content Generation | Medium | Low | High | High | High |
| Code Generation | High | High | High | Medium | Medium |
| Customer Service QA | Medium | Low | Medium | Medium | High |


\end{center}

</details>

**5. What is LLM-as-Judge? What is its basic principle?**

<details>
<summary>💡 Show Answer</summary>

LLM-as-Judge involves using one (usually stronger) LLM to score the output of another LLM, serving as a replacement for human evaluation.

Basic Process:
\begin{lstlisting}[style=promptstyle]

	Evaluation Prompt = "Please score the following answer from 1 to 5 based on [Accuracy/Completeness/Format],
	and provide reasons:
	Question: {question}
	Answer: {answer}
	Rubric: {rubric}"

	Judge LLM -> Outputs score and reasons
\end{lstlisting}

Pros: Automatable, scalable, cheaper than human evaluators. Cons: The Judge itself has biases, and scoring consistency is not as good as human evaluators.

</details>

**6. When evaluating Prompts, how should the test set be constructed?**

<details>
<summary>💡 Show Answer</summary>

The test set is the cornerstone of the Prompt evaluation system, and its quality directly determines the credibility of the evaluation conclusions. A well-designed test set must satisfy requirements in three dimensions simultaneously: **representativeness, diversity, and difficulty distribution**.

#### Construction Principles

- **Representativeness**: The input distribution of the test set should be as close as possible to the distribution of real user requests. The most reliable source is real request logs from the production environment (which must be desensitized). If production data is unavailable, the real distribution should be simulated through user research or business analysis to avoid coverage bias caused by test sets constructed from a developer's perspective;

- **Diversity**: Cover different input lengths (short sentences, medium paragraphs, long texts), expression styles (formal/colloquial/jargon), topic domains, and types of user intent. A test set lacking diversity is prone to yielding false-positive results like "performing well on specific patterns but failing massively after actual deployment";

- **Edge Cases**: Specifically construct pressure test samples, including:
    - Empty or extremely short inputs (single characters, punctuation marks);
    - Extremely long inputs (approaching or exceeding the context window limit);
    - Ambiguous inputs (where multiple reasonable interpretations hold);
    - Unexpected formats (users not following the preset input template);
    - Mixed language inputs (e.g., mixing Chinese and English).

- **Adversarial Cases**: Collect or construct historically known failure modes, semantic traps, and input types prone to triggering hallucinations, specifically testing the Prompt's robustness on difficult samples;

- **Ground Truth**: Every test sample should be equipped with a clear expected output or evaluation standard. Annotation quality directly determines evaluation effectiveness—it is recommended to have two annotators annotate independently and calculate Cohen's Kappa consistency coefficient. $\kappa > 0.8$ is considered reliable annotation; below this threshold, annotation guidelines need to be re-discussed.

#### Scale Recommendations

The scale of the test set should match the evaluation purpose:
- **Rapid Validation** (Directional judgment during development): 50--100 items, focusing on core scenario coverage;
- **Formal Evaluation** (Version iteration comparison): 200--500 items, needing to cover main sub-categories;
- **Pre-production Deployment** (Final quality gate): 1000+ items, including ample edge and adversarial cases;
- **Multilingual/Multi-scenario Products**: On top of the above, prepare a dedicated subset for each target language or scenario.

#### Source Strategy

High-quality test sets usually fuse multiple sources:
1. **Real User Logs**: Most representative, reflecting true distributions, must be desensitized before use;
2. **Manually Constructed**: Specifically targeting edge cases and known hard examples, compensating for insufficient coverage of rare scenarios in logs;
3. **Historical Failure Cases**: Solidify newly discovered failure modes into test cases to achieve permanent regression protection;
4. **Adversarial Generation**: Use LLMs to assist in generating variants that are semantically similar to existing samples but differ greatly in expression, expanding diversity.

</details>

**7. What scenarios are suitable for automated evaluation and human evaluation respectively?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]

	\caption{Comparison of Automated Evaluation vs. Human Evaluation}


| **Evaluation Method** | **Suitable Scenarios** | **Unsuitable Scenarios** |
| --- | --- | --- |
| Automated Evaluation | Clear answers (classification, extraction) | Subjective quality judgments |
|  | Format compliance checks | Creative content evaluation |
|  | Large-scale rapid iteration | Detailed semantic understanding |
|  | Regression testing (CI/CD integration) | Initial evaluation standard definition |
| Human Evaluation | Subjective tasks (writing, dialogue quality) | Large sample batches (too costly) |
|  | Final pre-deployment verification | High-frequency iteration (response too slow) |
|  | Calibration and annotation of methods | Pure format checking tasks |
|  | First evaluation of new task types |  |


\end{table}

In practice, they are usually **used in a layered combination**: automated evaluation handles high-frequency, large-batch rapid screening (triggered immediately after every Prompt modification), while human evaluation handles low-frequency, high-value final confirmation (before version deployment or when calibrating evaluation methods). The division of labor can be summarized as "automated evaluation improves efficiency, human evaluation guarantees the upper limit."

Furthermore, **LLM-as-Judge** acts as a compromise between the two, superior to pure human or pure rule evaluation in cost and accuracy, suitable for large-batch scoring of subjective tasks, but it requires periodic human spot checks to calibrate its consistency with real human judgments.

</details>

**8. What is A/B testing? How is it applied in Prompt optimization?**

<details>
<summary>💡 Show Answer</summary>

A/B testing is dividing test samples randomly into two groups, applying Prompt A (control group, usually the current production version) and Prompt B (experimental group, the new version) respectively, and comparing the core metric differences between the two groups via statistical methods to drive the decision of which version is better with data.

#### Application Steps

1. **Define Primary Metric**: Clarify the single primary decision metric (e.g., task success rate, average LLM-Judge score) before testing begins, to avoid picking favorable results from multiple metrics afterward (p-hacking);

2. **Prepare Representative Test Set**: Use a test set that covers the true distribution. Both A/B groups must use **the exact same test set** (the same sample pair tested on both versions) to eliminate the interference of sample differences on conclusions;

3. **Fix All Irrelevant Variables**: Model version, temperature, top_p, max_tokens, system time, and all non-test factors must remain completely consistent during A/B testing;

4. **Parallel Inference**: Call both versions A and B in parallel on the same batch of test samples, recording their respective output results and evaluation scores;

5. **Statistical Significance Testing**: Choose the appropriate test method based on the metric type (Chi-square or Z-test for ratio metrics, t-test or Bootstrap for continuous scores), with $p < 0.05$ considered statistically significant;

6. **Multi-metric Comprehensive Decision**: A significant improvement in the primary metric is a necessary condition, but it must also be confirmed that secondary metrics (format compliance rate, safety, cost) have not significantly degraded;

7. **Logging and Archiving**: Fully record test configurations, sample sets, results, and decision reasons as reference bases for future iterations.

</details>

**9. In A/B testing, how do you ensure the fairness of the comparison between the two groups?**

<details>
<summary>💡 Show Answer</summary>

The validity of A/B testing conclusions relies entirely on the **strictness of controlled variables**; any uncontrolled differences could become confounding factors leading to erroneous conclusions.

- **Consistent Model Version**: Strictly prohibit switching foundation models or model versions during A/B testing. Even silent model updates by the platform can affect results, so it's recommended to specify exact model version numbers (e.g., `claude-sonnet-4-6` instead of `claude-latest`);

- **Completely Consistent Inference Parameters**: `temperature`, `top_p`, `top_k`, `max_tokens`, and random seed (if fixing is supported) must all remain identical;

- **Exactly Same Test Set**: A/B must use **the exact same batch of samples from the same test set** to avoid introducing bias from sample discrepancies. If the test set is too large and requires sampling, the same random seed must be used to ensure A/B draw identical subsets;

- **Temporal Parallelism**: In online A/B tests, groups A and B should run in parallel within the **same time window** to avoid confounding from temporal factors (e.g., holiday traffic pattern differences, model service quality fluctuations);

- **Single Variable Principle**: Each A/B test should only change one dimension of the Prompt (e.g., only modifying example content, or only tweaking task description phrasing) to ensure observed effects can be explicitly attributed to that variable's change rather than the compounding of multiple edits.

</details>

**10. How much sample size is meaningful for A/B testing?**

<details>
<summary>💡 Show Answer</summary>

Insufficient sample size is the most common methodological error in Prompt A/B testing, leading to unreliable conclusions (insufficient statistical power to detect real differences, or misjudging random fluctuations as significant differences).

#### Determining Factors of Sample Size

Sample size is determined collectively by the following three parameters:
- **Effect Size**: The magnitude of the expected difference between A/B to be detected. Smaller differences require more samples;
- **Significance Level** ($\alpha$): Usually set to 0.05, meaning a 5% probability of mistakenly judging "no difference as a difference" is allowed;
- **Statistical Power** (Power = $1 - \beta$): Usually set to 0.8, meaning there is an 80% probability of detecting a real difference when it exists.

#### Empirical Reference Values

\begin{table}[h]

	\caption{Minimum sample size per group for different effect sizes ($\alpha=0.05$, Power=0.8)}


| **Expected Absolute Lift** | **Min. Sample Size/Group** | **Applicable Scenarios** |
| --- | --- | --- |
| 10% or more | $\approx$ 100 items | Early rapid validation, large errors tolerated |
| 5% | $\approx$ 400 items | Routine iterative evaluation |
| 2% | $\approx$ 2500 items | Refined optimization stage |
| 1% | $\approx$ 10000 items | Large-scale production fine comparison |


\end{table}

Precise calculations can use the Python `statsmodels` library:
\begin{verbatim}
	from statsmodels.stats.power import NormalIndPower
	analysis = NormalIndPower()
	n = analysis.solve_power(effect_size=0.2, alpha=0.05, power=0.8)
\end{verbatim}

</details>

**11. What are ROUGE and BLEU metrics? What type of tasks are they suitable for evaluating?**

<details>
<summary>💡 Show Answer</summary>

- **BLEU (Bilingual Evaluation Understudy)**: Calculates the n-gram exact match rate between model output and a reference answer. Originally designed for machine translation evaluation, BLEU focuses on **precision**—what proportion of words in the model output appear in the reference answer. It also introduces a Brevity Penalty to prevent models from "cheating" by outputting extremely short text. The formula is:

    $$
    BLEU = BP \cdot \exp\left(\sum_{n=1}^{N} w_n \log p_n\right)
    $$

    Where $p_n$ is the n-gram precision, $w_n$ is the weight, and BP is the brevity penalty factor.

- **ROUGE (Recall-Oriented Understudy for Gisting Evaluation)**: Focuses more on **recall**—what proportion of words in the reference answer are covered by the model output. It is commonly used for text summarization evaluation. Common variants include ROUGE-1 (unigram), ROUGE-2 (bigram), and ROUGE-L (Longest Common Subsequence).

#### Applicability and Limitations

\begin{table}[h]

	\caption{BLEU/ROUGE Applicability Scope}


| **Task Type** | **Applicability** | **Reason** |
| --- | --- | --- |
| Machine Translation | BLEU suitable | Reference answers relatively unique, word overlap is meaningful |
| Text Summarization | ROUGE suitable | Evaluates key info coverage |
| Info Extraction | Both suitable | Clear target vocabulary/phrases |
| Open-ended QA | Unsuitable | Semantic equivalents diverse, word overlap low |
| Creative Writing | Unsuitable | Reference not unique, evaluation dimension is quality |
| Dialogue Systems | Unsuitable | Natural reply diversity unrelated to n-gram match rate |


\end{table}

For open-ended generative tasks, it's recommended to switch to **BERTScore** (similarity based on semantic embeddings) or **LLM-as-Judge** (semantic-level quality evaluation) to compensate for the lack of semantic generalization in metrics based on word overlap.

</details>

**12. How to evaluate format compliance (e.g., requiring the model to output JSON)?**

<details>
<summary>💡 Show Answer</summary>

Format compliance evaluation is one of the most suitable evaluation types for automation, allowing for high-frequency testing with zero manual cost.

#### Three-Tier Check System

1. **Hard Parsing Check**: Attempt to parse the output using the target format's official parser. If parsing fails, it is directly judged as non-compliant:
    \begin{verbatim}
    import json
    def check_json_valid(output: str) -> bool:
    try:
    json.loads(output)
    return True
    except json.JSONDecodeError:
    return False
    \end{verbatim}

2. **Schema Structure Validation**: After successful parsing, verify whether the data structure conforms to a pre-defined Schema (using `jsonschema` or Pydantic):
    \begin{verbatim}
    from pydantic import BaseModel
    class OutputSchema(BaseModel):
    title: str
    score: float
    tags: list[str]

    def check_schema(output: str) -> bool:
    try:
    OutputSchema.model_validate_json(output)
    return True
    except Exception:
    return False
    \end{verbatim}

3. **Semantic Soft Checking**: After Schema compliance, further verify the business rationality of field values (whether enum values are within the preset range, numeric values are within reasonable intervals, string lengths meet constraints, etc.).

#### Core Evaluation Metrics

- **Format Compliance Rate**: $Compliance Rate = \dfrac{Number of samples passing all three tiers}{Total number of samples}$. This is the most crucial single metric;
- **Field Completeness Rate**: The proportion of samples where mandatory fields exist, used to pinpoint partial compliance issues like "output parseable but missing key fields";
- **Value Legality Rate**: The proportion of field values passing type/range validation;
- **Failure Mode Distribution**: Classifying non-compliant outputs by failure type (parsing failure/missing fields/illegal values/extra prefix text, etc.) to guide targeted optimization.

</details>

**13. What are the common biases of LLM-as-Judge? How to mitigate them?**

<details>
<summary>💡 Show Answer</summary>

LLM-as-Judge is an important method for automated evaluation of subjective tasks, but its judgments are influenced by multiple systemic biases. Left uncontrolled, these biases can distort evaluation conclusions.

#### Common Biases

- **Position Bias**: In pairwise comparison evaluations, Judges tend to favor the answer presented **first** (primacy effect), or on some models, exhibit a recency preference;
- **Length Bias**: Judges tend to give higher scores to **longer** responses, even if the longer response contains redundant or irrelevant content and has lower actual information density;
- **Self-Enhancement Bias**: When the Judge model is identical or similar in style to the model being evaluated, the Judge tends to score answers that mimic its own output style higher;
- **Language Bias**: Judges typically score English responses higher than equivalent responses in other languages, even if the content quality is exactly the same;
- **Authority Signal Bias**: If an answer contains "authoritative-looking" elements like citations, numbers, or jargon, Judges tend to score it high, even if these elements are inaccurate.

#### Mitigation Strategies

- **Position Debiasing (Swapping Order)**: Evaluate the same pair (A vs B) twice: first presented as A-B, then as B-A. Only judgments consistent across both runs (both choosing A or both B) are accepted. Contradictory results are marked as a "tie", effectively eliminating position bias;

- **Multi-Judge Ensembling**: Use multiple models from different families (e.g., GPT-4o, Claude, Gemini) to judge separately and take a majority vote. Different models' bias directions vary, and ensembling can cancel them out;

- **Structured Scoring Scale (Rubric)**: Provide the Judge with detailed scoring dimensions and specific descriptions corresponding to each score to reduce the Judge's discretionary space:
    \begin{verbatim}
    5 Points: Completely correct, info complete, no redundancy
    4 Points: Mostly correct, minor omissions but doesn't affect use
    3 Points: Partially correct, obvious omissions or minor errors
    2 Points: Major info errors, but direction is right
    1 Point: Completely wrong or irrelevant
    \end{verbatim}

- **Human Calibration**: Periodically sample 5-10% of the Judge's evaluation results for human verification, calculating the human-machine agreement rate (Cohen's Kappa). If $\kappa < 0.6$, the Judge Prompt needs redesign or the Judge model needs replacing.

</details>

**14. What is the difference between offline evaluation and online A/B testing? What are their respective values?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]

	\caption{Comparison of Offline Evaluation and Online A/B Testing}


| **Dimension** | **Offline Evaluation** | **Online A/B Testing** |
| --- | --- | --- |
| Environment | Simulated, historical/constructed test set | Real user traffic |
| Iteration Speed | Fast, results in minutes | Slow, requires traffic accumulation |
| Cost | Low, only API call fees | High, carries experimental quality risks |
| Authenticity | May deviate from true distribution | Fully authentic, no distribution bias |
| Eval Metrics | Automated/human eval metrics | Business metrics (retention, conversion, satisfaction) |
| Applicable Stage | Development iteration stage | Final deployment validation stage |
| Controllability | Fully controllable, reproducible | Influenced by real traffic and user behavior |


\end{table}

**Complementary Value Relationship**: These two evaluation methods form a serialized quality assurance pipeline—offline evaluation is used for **rapid screening** of candidate Prompts, narrowing down from dozens to 1-2 candidates; online A/B testing performs **final business value validation** on the best version selected by offline evaluation, confirming its business metric gains under real traffic.

It is important to note that offline evaluation metrics and online business metrics are **not always positively correlated**—a 5% increase in offline accuracy does not necessarily translate to a 5% increase in online user satisfaction. Therefore, online A/B testing as the final decision basis cannot be skipped.

</details>

**15. How to establish a regression testing system for Prompts to prevent quality degradation during iterations?**

<details>
<summary>💡 Show Answer</summary>

1. **Golden Test Set**: Maintain a **fixed** test set (100--500 items) covering core functional scenarios, and full evaluation must be run after every Prompt modification. The test set itself must be under version control (Git), and modifying it requires separate review to prevent the test set from being "polluted" to accommodate new versions;

2. **CI/CD Automated Integration**: Integrate Prompt evaluation into the deployment pipeline, automatically triggering evaluation tasks after every Prompt change commit. Define quality gate thresholds (e.g., block deployment if main metrics regress by over 2%) to intercept issues before they reach production;

3. **Baseline Locking**: Treat the evaluation score of the current production version as a **non-negotiable lower bound baseline**. New versions must score no lower than the baseline on all evaluation dimensions to be deployed;

4. **Dimensional Monitoring**: Set monitoring and thresholds **separately** for accuracy, format compliance rate, safety, cost, etc. This prevents significant regression in one dimension from being masked by improvements in others (e.g., accuracy increases but format compliance rate drops);

5. **Permanent Failure Cases**: New failure modes discovered in production must be solidified as test cases in the golden test set to ensure the same issue will not reappear in future versions—this is the core mechanism for continuously strengthening the regression testing system.

</details>

**16. How to evaluate the generalization ability of a Prompt across different linguistic/cultural scenarios?**

<details>
<summary>💡 Show Answer</summary>

1. **Language-Specific Test Sets**: Prepare **independently constructed** test sets for each target language, rather than simply machine-translating an English test set. Language carries cultural background, and direct translation loses contextual differences (e.g., the expression habits of Chinese users fundamentally differ from English users);

2. **Involvement of Native Evaluators**: Cross-cultural subjective evaluations (such as the naturalness, politeness, and cultural appropriateness of a response) must be conducted by **native speakers** of the target language. Judgments by non-native speakers have systemic biases;

3. **Localization Format Validation**: Automatically detect format localization correctness in outputs, including date formats (`2025/08/01` vs `01.08.2025`), currency symbols, number separators, punctuation habits (Chinese full-width vs English half-width), etc.;

4. **Cultural Sensitivity Detection**: Run on test sets from different cultural backgrounds to detect whether outputs contain culturally inappropriate expressions (such as handling of specific holidays, customs, or taboo topics);

5. **Cross-Language Consistency**: For identical semantic content with inputs in Chinese and English (e.g., "今天天气怎么样？" vs "What's the weather today?"), detect whether outputs are consistent on core content to quantify the degree of cross-language semantic drift.

</details>

**17. How to handle output randomness (non-determinism) in Prompt evaluation?**

<details>
<summary>💡 Show Answer</summary>

LLM sampling randomness introduces statistical noise into evaluation results, and mishandling it leads to the systemic error of "misjudging Prompt superiority due to random fluctuations."

1. **Use temperature=0 for Deterministic Tasks**: For tasks with clear correct answers like classification, information extraction, and format conversion, set temperature to 0 to eliminate sampling randomness, making evaluation results completely reproducible and yielding identical conclusions every time the same Prompt is evaluated;

2. **Average Over Multiple Samples for Subjective Tasks**: For subjective tasks like creative writing and dialogue quality, run each test sample 3--5 times and average the scores, reducing the variance of a single evaluation by increasing sampling times;

3. **Consistency as an Independent Metric**: Incorporate "output consistency over multiple samplings" itself into the Prompt quality evaluation system, calculating the mean semantic similarity or standard deviation of format compliance rate across N outputs—Prompts with low consistency bring unpredictable user experience fluctuations in production;

4. **Bootstrap Confidence Intervals**: Evaluation metric reports should not only provide point estimates but also 95% confidence intervals. Obtain metric distributions via Bootstrap resampling (sampling with replacement from the test set 1000 times and calculating metric values each time). Two versions with overlapping confidence intervals should not rashly be deemed to have a "significant difference."

</details>

**18. How to design a specialized evaluation framework for RAG scenarios?**

<details>
<summary>💡 Show Answer</summary>

RAG system evaluation must simultaneously cover both the **retrieval quality** and **generation quality** stages independently. A failure in either stage makes the final output unusable, so targeted metrics must be established separately.

#### Retrieval Quality Evaluation

- **Recall@K**: The proportion of relevant documents covered within the Top-K retrieved results, measuring "were the correct documents retrieved";
- **Precision@K**: The proportion of relevant documents in the Top-K results, measuring "the noise ratio of the retrieved results";
- **MRR (Mean Reciprocal Rank)**: $MRR = \dfrac{1}{|Q|}\sum_{i=1}^{|Q|}\dfrac{1}{rank_i}$, measuring the average rank of the most relevant document, reflecting retrieval ranking quality;
- **NDCG (Normalized Discounted Cumulative Gain)**: A ranking quality metric considering relevance gradients, suitable for multi-level relevance annotation scenarios.

#### Generation Quality Evaluation (RAG Specific)

- **Faithfulness**: Whether every statement in the answer is clearly supported by retrieved content, detecting if the model introduced hallucinatory info outside the retrieved context;
- **Answer Relevance**: Whether the generated answer directly responds to the user's original query, rather than answering a different question;
- **Context Relevance**: Whether retrieved content is truly relevant to the user query, evaluating the precision of the retrieval strategy;
- **Citation Accuracy**: If the answer contains source citations, whether the citation actually points to the document supporting that answer.

These metrics can be evaluated automatically through the **RAGAS** framework. RAGAS uses the LLM-as-Judge method to calculate scores across all dimensions and can be run in large batches without manual annotation.

</details>

**19. How to evaluate the quality of a Chain-of-Thought Prompt?**

<details>
<summary>💡 Show Answer</summary>

Evaluating CoT Prompts is more complex than evaluating direct outputs because it requires simultaneously assessing both the **reasoning process** and the **final conclusion** on two levels, and the quality of both is not always consistent.

- **Final Answer Accuracy**: Whether the final answer after reasoning is complete is correct. This is the most basic evaluation dimension but insufficient—correct answers might stem from incorrect reasoning (Lucky Guess);

- **Reasoning Step Validity**: Step-by-step evaluation of the logical correctness of intermediate reasoning steps. Use LLM-as-Judge to score each step (correct/flawed/incorrect), and tally the proportion of "wrong reasoning but correct answer" (Lucky Guess Rate). A high proportion means CoT isn't truly providing reasoning gains;

- **Quantifying Net CoT Gain**: Compare "Accuracy with CoT" vs "Accuracy of direct output without CoT" to quantify true lift from CoT. If the net gain is near zero, it means the task is unsuitable for CoT or the CoT design needs improvement;

- **Reasoning Efficiency**: Given similar accuracy, compare the length of the reasoning chain (token count) of different CoT designs. A more concise reasoning chain generally means lower cost and faster response;

- **Error Propagation Analysis**: Count the distribution of "first step correct but subsequent steps wrong" vs "wrong at first step" to locate weak links in the CoT chain and guide targeted Few-shot example optimization.

</details>

**20. How to quickly evaluate multiple candidate Prompt versions when resources are limited?**

<details>
<summary>💡 Show Answer</summary>

1. **Funnel Screening**: Don't do full evaluations on all candidate versions. Instead, establish a multi-round screening funnel—first round uses 20--30 samples to quickly eliminate obviously poor versions (usually weeding out 50--70%); second round does medium-scale evaluation with 100--200 samples on remaining candidates; third round only does full evaluation on the final 1--2 candidates;

2. **Stratified Sampling**: Do stratified sampling by input category from the test set, covering an approximately complete input distribution with far fewer samples than the full set. For example, if a test set has 10 subcategories with 100 items each, drawing 10 from each to get 100 provides better distribution coverage than randomly picking 100;

3. **Focus on Key Metrics**: During resource-constrained phases, calculate only 1--2 core decision metrics, skipping full-dimensional evaluation. Complete multi-dimensional evaluation is reserved for finalized versions;

4. **Pairwise Evaluation**: Directly have the Judge (LLM or human) compare whether A or B is better, rather than assigning absolute scores to each. Pairwise comparisons have lower noise (evaluators are better at relative judgment than absolute scoring) and yield more reliable conclusions with the same sample size;

5. **Parallel Batch Inference**: Utilize Batch APIs and concurrent requests to execute inference for all candidates on the test set in parallel, compressing what would be serial evaluation time down to the evaluation time of a single version.

</details>

**21. How should Prompt evaluation results guide the next round of optimization?**

<details>
<summary>💡 Show Answer</summary>

The value of evaluation results lies not in "what score was achieved," but in **locating failure modes and guiding targeted improvements**. Systematic Error Analysis is the critical link connecting evaluation and optimization.

#### Mapping Metrics to Optimization Directions

- **Low Accuracy**: Cluster failed samples and determine whether it's due to unclear task descriptions (lacking clear success criteria definition), poor example quality (Few-shots not covering failure types), or the task itself exceeding the model's capabilities;

- **High Format Error Rate**: Usually implies formatting instructions are unclear or examples are insufficient. Optimization directions include: strengthening format constraint wording ("Output only JSON, do not include any other text"), adding correct format examples, introducing negative examples to show incorrect formats, or switching to Structured Outputs APIs for fundamental format guarantees;

- **Output Length Not Meeting Expectations**: Explicitly add length constraints in the Prompt and provide positive and negative examples demonstrating target length;

- **Edge Case Failures**: Add explicit handling instructions for such boundary cases in the Prompt, or introduce Few-shot examples specifically covering those situations;

- **Low Consistency**: Lower temperature, add constraints to reduce output space, or explicitly specify output formats to reduce ambiguity;

- **Safety or Compliance Issues**: Add defensive declarations, detail descriptions of prohibited behaviors, or introduce a content safety detection layer.

#### Error Analysis Process

1. Collect all failed samples scoring below a threshold;
2. Manually or with LLM assistance, classify and tag the reasons for failure (Task Misunderstanding / Format Error / Factual Error / Safety Issue / Other);
3. Count frequency distributions of failure types, prioritizing the most frequent failure modes;
4. Modify Prompt for main failure modes, rapidly verify the fix effect on subsets containing those failure samples;
5. Submit full evaluation after confirming the fix doesn't introduce other types of regressions.

</details>

**22. How to establish an end-to-end MLOps process for Prompt Engineering?**

<details>
<summary>💡 Show Answer</summary>

\begin{lstlisting}[style=promptstyle]
	Req Definition -> Prompt Dev -> Offline Eval -> Human Review -> Online A/B Test -> Full Rollout -> Monitoring
	^                                                                              |
	|-------------------------- Issue Feedback and Iteration ----------------------|
\end{lstlisting}

#### Detailed Process Steps

1. **Requirement Definition**: Clarify task goals, success criteria, core evaluation metrics, and non-negotiable safety red lines. Establish the evaluation framework before writing the Prompt;

2. **Prompt Development**: Manage Prompt templates in version control (Git), ensuring every modification has clear change logs and reasons, supporting rapid rollback;

3. **Offline Evaluation**: CI/CD triggers automatic evaluation, results written to experiment tracking systems (LangSmith, PromptFlow, MLflow). Ensure evaluation results, configs, and test set versions of every iteration are logged for reproducibility;

4. **Human Review**: Human spot-checking of automated evaluation results to calibrate evaluation method accuracy, make human judgments on edge cases, and provide final deployment recommendations;

5. **Online A/B Testing**: Shunt traffic via Request ID hashing. A/B versions run parallel in real traffic, tracking business metrics (task success rate, user satisfaction score, session completion rate) in real time;

6. **Full Rollout and Continuous Monitoring**: Continuously track production metrics post-deployment, set anomaly alert thresholds, and automatically trigger alerts allowing one-click rollback to the last stable version if metrics deteriorate.

</details>

**23. How to evaluate the quality of a Prompt? What are the quantitative metrics?**

<details>
<summary>💡 Show Answer</summary>

Quantifying Prompt quality requires establishing a metric system across four dimensions: **Functionality, Quality, Robustness, and Cost**. A single metric cannot comprehensively reflect a Prompt's actual performance.

#### Functionality Metrics
- **Accuracy/Pass Rate**: In tasks with standard answers (classification, QA, code generation), the proportion of outputs that completely match expected answers or pass test cases;
- **Format Compliance Rate**: The proportion of outputs that fully conform to specified format requirements (JSON parseable, complete fields, valid values);
- **Task Completion Rate**: In multi-step tasks, the proportion of samples successfully completing all required steps.

#### Quality Metrics
- **LLM-Judge Score**: Using a strong model to score outputs on dimensions like relevance, accuracy, completeness, and fluency, providing a fine-grained quality portrait;
- **Human Scoring**: Sampling for human review (100--200 items) serves as a calibration baseline for LLM-Judge and a reference for the final quality upper bound.

#### Robustness Metrics
- **Sampling Consistency**: With temperature > 0, the mean semantic similarity of outputs from multiple runs on the same input;
- **OOD Generalization Rate**: The magnitude of performance degradation on Out-of-Distribution inputs (input types beyond test set coverage);
- **Edge Case Handling Rate**: The proportion of boundary inputs (empty inputs, ultra-long inputs, ambiguous inputs) handled correctly.

#### Cost Metrics
- **Average Input/Output Tokens**: Directly determines API call costs and must be considered alongside quality metrics (token efficiency = quality lift / token increment);
- **Average End-to-End Latency**: Time from request sent to full response received. Impacts user experience, and increased tokens linearly affect latency.

</details>

**24. What is Prompt "Consistency"? How is it measured?**

<details>
<summary>💡 Show Answer</summary>

Prompt Consistency refers to the degree to which a Prompt induces stable model outputs in format, core content, and quality under identical or semantically equivalent inputs. Consistency is a core metric of Prompt reliability; low consistency implies unpredictable user experience.

#### Measurement Methods

1. **Deterministic Consistency** (temperature=0): Multiple runs on the same input should theoretically yield completely identical outputs. Differences usually imply ambiguity in the Prompt causing the model to randomly wander among equally probable interpretations;

2. **Sampling Consistency** (temperature > 0): Sample the same input N times (usually 5--10 times) and use the following to quantify output stability:
    - ROUGE/BLEU: Calculate n-gram overlap between each output and the mean output;
    - Embedding Similarity: Calculate the mean cosine similarity of embedding vectors of N outputs;
    - LLM-Judge Consistency Score: Have a Judge model determine if core content across N outputs is consistent.

3. **Format Consistency**: Count the proportion of fully format-compliant runs out of N runs. Format consistency is generally easier to quantify than content consistency;

4. **Key Information Stability**: Extract key info elements from each output (numerical conclusions, enumerated items, yes/no judgments) and tally the proportion where these key elements remain consistent across N runs.

#### Common Causes of Low Consistency

Ambiguity in the Prompt (multiple reasonable interpretations of task descriptions), contradictions among Few-shot examples, unclear constraints (like unclear definition of "brief"), or temperature set too high for the task.

</details>

**25. What is the "Format Compliance Rate" of a Prompt? How to optimize it?**

<details>
<summary>💡 Show Answer</summary>

Format Compliance Rate is defined as: out of N runs, the proportion of samples where outputs **completely conform** to specified format requirements (JSON parseable, mandatory fields present, field value types valid, no extra prefix/suffix text).

#### Optimization Methods (Ranked from strongest to weakest impact)

1. **API-Layer Hard Constraints**: Use Structured Outputs or JSON Mode API provided by models to constrain outputs to legal formats at the sampling algorithm level. This is currently the most reliable format assurance method (compliance rates can reach 99%+);

2. **Few-shot Positive Reinforcement**: Provide 2--3 output examples that **completely conform to the format** in the Prompt, letting the model learn the correct format via pattern matching, which is significantly better than pure text descriptions;

3. **Negative Example Comparison**: Alongside positive examples, provide 1--2 format examples labeled "incorrect" to help the model understand boundaries:
    \begin{verbatim}
    Incorrect example (do not output like this):
    "Okay, here is the JSON: “`json\n{...}\n“`"
    Correct example (should output like this):
    {"key": "value"}
    \end{verbatim}

4. **Explicit Mandatory Instructions**: Use unambiguous mandatory phrasing: "Output only a valid JSON object, without adding any explanations, preambles, code block markers, or line breaks";

5. **App-Layer Format Validation & Retry**: The application automatically validates the format after calling the LLM; if non-compliant, it retries automatically (attaching a format error message). A max of 2 retries keeps costs controllable and significantly boosts final compliance rate;

6. **Two-Step Generation**: Decouple "generating content" and "formatting" into two steps—first let the model generate content freely, then use a separate formatting Prompt to convert the first step's output to the target format. Separating concerns yields higher compliance in both steps than doing it all at once.

</details>

**26. How to compare the effects of two Prompt versions using A/B testing?**

<details>
<summary>💡 Show Answer</summary>

1. **Single Variable Control**: **Modify only one dimension** per A/B test (like only changing example count, or task phrasing) to ensure observed differences can clearly be attributed to that variable, rather than compounding changes;

2. **Unified Test Set**: A/B must use completely identical test samples. Feed the same sample simultaneously to A and B versions to eliminate confusion from sample disparities;

3. **Pre-calculated Sample Size**: Calculate the minimum sample size based on expected effect size and significance requirements **before** testing, rather than collecting until "significant" (which causes p-hacking);

4. **Metric Pre-registration**: Explicitly log primary and secondary metrics before testing to avoid picking favorable results post-hoc;

5. **Statistical Significance Testing**: After reaching the preset sample size, choose tests based on metric types (Z-test for ratios, t-test or Bootstrap for continuous). Declare version B superior to A only if $p < 0.05$ and the effect size has practical business meaning;

6. **Comprehensive Multi-Metric Judgment**: A significant lift in primary metrics is required, while simultaneously verifying that secondary metrics like safety, format compliance, and cost haven't shown significant regression, before holistically deciding whether to adopt B;

7. **Complete Record Archiving**: Record and archive the complete testing configuration (test set version, model version, inference parameters, sample size), result data, and reasoning behind the decision, serving as a reference point for future similar decisions.

</details>

**27. How to build a Golden Test Set for Prompts?**

<details>
<summary>💡 Show Answer</summary>

1. **Diverse Sources**: A Golden Test Set should blend three types of sample sources:
    - *Real user logs* (anonymized): The most representative, reflecting real input distributions, should account for 50--60%;
    - *Manually constructed samples*: specifically covering edge cases, known difficult examples, and low-frequency but critical scenarios, accounting for 30--40%;
    - *Historical failure cases*: Extracted from production incidents and user complaints, each failure case serves as a permanent regression test case, accounting for 10--20%.

2. **Annotation Standards and Quality Control**: Establish a clear scoring standard document (criteria for completely correct / partially correct / incorrect), with at least two independent annotators. Calculate the Cohen's Kappa agreement coefficient; $\kappa \geq 0.8$ is considered reliable. If below this threshold, annotation standards must be re-discussed and revised;

3. **Balanced Dimensional Coverage**: Ensure the test set has sufficient samples across the following dimensions: normal inputs / boundary inputs / adversarial inputs / different difficulty levels / varying input lengths. This avoids skewed evaluation conclusions caused by biased sample distributions;

4. **Strict Separation from Development Set**: Only use the Development Set for quick validations during development. The Golden Test Set is reserved exclusively for the final evaluation stage to prevent overfitting to the test set, which renders the evaluation meaningless;

5. **Continuous Incremental Updates**: Every time a new failure mode is discovered in production, add that case to the Golden Test Set immediately after fixing it, forming a continuous enhancement mechanism of "discover $\rightarrow$ fix $\rightarrow$ solidify into test case";

6. **Version Management**: The Golden Test Set itself is put under Git version control. Any modifications (additions, deletions, annotation changes) require review and logging to prevent arbitrary changes made to accommodate new versions, ensuring historical evaluation results remain comparable.

</details>

**28. How to systematically troubleshoot the root cause of poor Prompt output quality?**

<details>
<summary>💡 Show Answer</summary>

Troubleshooting framework (drilling down layer by layer):
1. **Confirm the Scope of the Problem**: Is it poor across all inputs, or specific types? Specific types $\rightarrow$ constraints for that type are missing;
2. **Format vs. Content**: Is it a format error or content error? Format error $\rightarrow$ format constraint issue; Content error $\rightarrow$ knowledge or reasoning issue;
3. **Task Understanding**: Did the model understand the task? Test with Zero-shot. If understood correctly $\rightarrow$ format/example issue; If misunderstood $\rightarrow$ unclear task description;
4. **Knowledge vs. Instruction Following**: The model has the knowledge but didn't follow instructions $\rightarrow$ instruction issue; The model lacks knowledge $\rightarrow$ requires RAG or Fine-tuning;
5. **Example Impact**: Effect improves after removing examples $\rightarrow$ example quality issue; Effect unchanged $\rightarrow$ problem lies in the task description;
6. **Model Capability Ceiling**: Switching to a stronger model yields significant improvement $\rightarrow$ current model capabilities are insufficient, the marginal benefit of prompt optimization is limited.

</details>

**29. How to design Prompt regression testing to prevent performance regressions in optimized versions?**

<details>
<summary>💡 Show Answer</summary>

1. **Establish a Golden Test Set**: Include historically encountered failure cases (added after being fixed) alongside representative normal cases;
2. **Automated Execution**: After every prompt change, CI/CD automatically runs the test set and outputs a pass-rate report;
3. **Multi-metric Monitoring**: Do not just look at primary metrics; simultaneously monitor format compliance rates, response length distributions, and edge case handling rates;
4. **Delta Analysis**: Automatically contrast performance differences between new and old versions on the test set, highlighting test cases that regressed;
5. **Blocking Mechanism**: If any metric drops beyond a threshold (e.g., 2%), automatically block the release;
6. **Permanent Retention of Failure Cases**: After every bug fix, the corresponding test case is permanently retained in the test set to prevent recurrence;
7. **Baseline Version Locking**: Record the baseline score on the test set when releasing a new version. Future versions must surpass this baseline to be released.

</details>

**30. What is "Prompt Fragility"? How to identify and fix it?**

<details>
<summary>💡 Show Answer</summary>

Prompt Fragility: The prompt is highly sensitive to minor input changes (phrasing, order, formatting), resulting in significantly different or even opposite outputs. Identification methods:
1. **Synonymous Paraphrase Testing**: Test with different phrasings that mean the same thing. Output differences >10% indicate fragility;
2. **Noise Testing**: Introduce slight noise (extra spaces, minor typos) into the input and observe output changes;
3. **Example Order Testing**: Shuffle the order of few-shot examples. Output changes >5% indicate fragility;
4. **Variance Analysis**: Use output variance from multiple runs on the same input as a fragility metric. Fix methods:
5. Add more diverse examples, covering a broader range of input variants;
6. Simplify and clarify task descriptions to reduce room for ambiguity;
7. Self-Consistency (multi-sample voting) reduces the impact of randomness;
8. Calibration to correct label biases;
9. Structured Outputs (JSON Schema) to reduce format fragility.

</details>

**31. What is Automatic Prompt Engineer (APE)? What is its core idea?**

<details>
<summary>💡 Show Answer</summary>

APE (Zhou et al. 2022): Uses an LLM to automatically generate a large number of candidate prompts, and selects the best-performing one by evaluating these candidates on a validation set. Core idea:
1. **Generation Phase**: Provide the APE system with a task description and a few input-output examples, letting the LLM generate multiple candidate prompts (usually 50–100);
2. **Evaluation Phase**: Run all candidate prompts on the validation set and compute performance metrics for each;
3. **Selection Phase**: Select the highest-scoring prompt as the final output;
4. **Optional Iteration**: Mutate the best prompt to generate the next generation of candidates, repeating evaluation and selection. Advantages: Reduces manual trial-and-error, capable of discovering highly effective prompts humans wouldn't naturally conceive. Limitations: High computational costs (N candidates $\times$ M validation samples); the optimal prompt found may be unreadable; limited out-of-distribution generalization.

</details>

**32. What is DSPy? How does it automate Prompt optimization?**

<details>
<summary>💡 Show Answer</summary>

DSPy (Khattab et al. 2023): A Python framework that automates prompt optimization for LLM pipelines, shifting "writing prompts" to "declarative programming + automatic optimization." Core mechanisms:
1. **Declarative Modules**: Use `dspy.Signature` to declare input/output semantics (`"question -> answer"`) instead of hand-writing prompts;
2. **Teleprompter (Optimizer)**: Automatically searches for the optimal combination of few-shot examples and instruction phrasing, aiming to optimize validation set performance;
3. **Module Composition**: Complex pipelines are built from multiple DSPy modules. The optimizer can perform end-to-end optimization of the entire pipeline;
4. **Decoupled from LLMs**: When switching the underlying LLM, prompts are automatically re-optimized without needing manual rewrites. Advantages: Extremely suitable for tasks with annotated validation sets; reduces reliance on LLM experts for prompt engineering; supports automated CoT example generation.

</details>

**33. How to design a Prompt template management system for large-scale, multi-scenario applications?**

<details>
<summary>💡 Show Answer</summary>

Core design:
1. **Tiered Template Architecture**: Global Base Layer (brand tone, safety rules) + Scenario Layer (task-specific instructions) + Dynamic Layer (user inputs, search results), combined together;
2. **Template Engine**: Use Jinja2 or similar template languages supporting variable substitution (`{{user_query}}`) and conditional rendering (`{% if has_context %}`);
3. **Version Control**: Templates are tracked in Git with full change history, supporting rollbacks;
4. **Registry Center**: Unified registration of all templates (name, version, description, use case), supporting search by LLM version and scenario;
5. **Automated Evaluation Integration**: Template updates trigger the execution of corresponding scenario evaluation suites;
6. **Hot Reload**: Supports updating templates without restarting services (paired with canary releases);
7. **Multilingual Support**: Unified management of different language versions for the same scenario;
8. **Audit Logging**: Record every template change for easy issue tracing.

</details>

**34. How to quantitatively assess the impact of Prompt changes on business metrics?**

<details>
<summary>💡 Show Answer</summary>

1. **Define Business Metrics**: First, clarify business metrics directly tied to the prompt (user satisfaction, task completion rate, manual handover rate, repeat question rate);
2. **A/B Test Design**: Traffic splitting. The experimental group uses the new prompt, the control uses the old prompt, tracking both technical and business metrics simultaneously;
3. **Conversion Funnel Analysis**: Prompt quality change $\rightarrow$ User behavior change $\rightarrow$ Business metric change. Establish a complete causal chain;
4. **Statistical Significance**: Only draw conclusions after gathering a sufficient sample size (calculated using statistical power analysis);
5. **Causal Attribution**: Rule out other concurrent variables (product feature updates, seasonal fluctuations) to confirm the impact is due to the prompt change;
6. **Long-term Tracking**: Some business metric changes lag (building user trust takes time), requiring 4–8 weeks of long-term observation;
7. **Cost-Benefit**: Quantify prompt optimization development costs versus the value brought by lifted business metrics to calculate ROI.

</details>

**35. How to establish a continuous optimization flywheel for Prompts (mining improvement directions automatically from production data)?**

<details>
<summary>💡 Show Answer</summary>

Flywheel Design:
1. **Data Collection**: Sample production traffic + Auto-tag user negative feedback (downvotes/complaints);
2. **Failure Classification**: Auto-classify collected low-quality outputs (format errors/content deviation/false refusals/hallucinations);
3. **Root Cause Localization**: For each failure type, automatically analyze which prompt section likely caused it (insufficient example coverage/unclear constraints/unspecified formatting);
4. **Improvement Suggestion Generation**: Use LLMs to generate prompt improvement suggestions based on failure cases;
5. **Automated Testing**: New suggestions are automatically verified against the test set. If they pass, they enter a human review queue;
6. **Canary Release**: After human confirmation, canary test on a small slice of traffic;
7. **Full Release & Monitoring**: Continuously monitor metrics post full-release. Rollback instantly if regressions are found;
8. **Cycle Frequency**: Aim to complete the flywheel loop every 1–2 weeks; the more data accumulates, the faster it goes.

</details>

**36. How to improve Prompt maintainability?**

<details>
<summary>💡 Show Answer</summary>

The core idea for improving Prompt maintainability is **modular design** and **standardized management**, avoiding piling everything into one giant monolithic Prompt, which makes pinpointing problems and making localized changes difficult.

#### 1. Modular Breakdown

Split a single Prompt into several independent, reusable functional segments:
- **Role Setting Module**: Describes the identity and professional background the model should assume.
- **Task Description Module**: Clarifies the core goal and scope of this specific invocation.
- **Constraints Module**: Lists prohibited behaviors, style preferences, length limits, etc.
- **Output Format Module**: Specifies the structure of the returned results (e.g., JSON, Markdown, numbered lists).

At the code level, these modules can be managed separately using string variables, configuration files, or template functions, eventually stitching them into a complete Prompt at runtime. Example structure below:

\begin{lstlisting}[style=promptstyle]
	# [Role Setting Module]
	You are a senior technical documentation writing expert, adept at turning complex concepts into clear and understandable explanations.

	# [Task Description Module]
	Based on the provided API interface description below, please write a usage guide intended for developers.

	# [Constraints Module]
	- Do not use English vocabulary exceeding the B2 level (if bilingual mixing is required)
	- Subjective evaluative words are prohibited (e.g., "very easy to use", "perfect")
	- Keep total word count under 300 words

	# [Output Format Module]
	Please output according to the following structure:
	1. Interface Overview (1-2 sentences)
	2. Parameter Descriptions (Table format)
	3. Request Example (Code block)
	4. Notes (Unordered list)
\end{lstlisting}

#### 2. Unified Naming and Versioning Standards

Establish naming conventions and version numbers for each module and full Prompt to easily track change history:

\begin{lstlisting}[style=promptstyle]
	prompts/
	├── modules/
	│   ├── role_tech_writer_v1.2.txt
	│   ├── constraint_length_300_v1.0.txt
	│   └── format_api_doc_v2.1.txt
	└── composed/
	└── api_doc_generator_v3.0.txt   <- Stitched from the modules above
\end{lstlisting}

Version numbers follow semantic versioning standards (e.g., `MAJOR.MINOR`):
- **MAJOR**: Significant changes to the overall logic or structure of the module;
- **MINOR**: Backwards-compatible changes like phrasing tweaks or minor constraint adjustments.

#### 3. Avoiding the Common Pitfalls of Monolithic Prompts

- **Muddled Responsibilities**: Writing roles, tasks, and formatting all within one natural language paragraph. Modifying one part easily affects the comprehension of the others.
- **Lack of Comments**: Long Prompts lacking comments make it impossible for team members to quickly pinpoint the intent of any specific text segment during collaboration.
- **Hardcoding Business Data**: Writing specific client names, dates, or product models directly into the Prompt requires manual full-text replacement every single time it is used.

Good maintainability allows team members to maintain Prompts just like they maintain code—local modifications, independent testing, and version rollbacks—dramatically lowering iteration costs.

</details>

**37. How to improve Prompt reusability across different tasks?**

<details>
<summary>💡 Show Answer</summary>

The key to improving reusability lies in **abstracting common structures** and **parameterizing task-specific content**, allowing the same template skeleton to drive multiple similar but distinct scenarios.

#### 1. Establish Generic Structural Templates

Most task-oriented Prompts can be abstracted into the following four-part framework:

\begin{lstlisting}[style=promptstyle]
	[ROLE]        # Role setting: Identity and capabilities the model should possess
	[TASK]        # Task description: Core objective of this invocation
	[CONSTRAINTS] # Constraints: Style, length, prohibitions, etc.
	[OUTPUT]      # Output format: Structure, language, examples, etc.
\end{lstlisting}

This framework itself isn't bound to any specific business logic and can serve as a "Prompt skeleton" to be used uniformly across projects.

#### 2. Parameterized Injection of Task-Specific Content

Replace the parts of the skeleton that vary by task with **placeholder variables**, injecting them dynamically at runtime:

\begin{lstlisting}[style=promptstyle]
	# Generic Template (Pseudocode representation)

	[ROLE]
	You are an experienced {{domain}} expert.

	[TASK]
	Please complete the "{{task_type}}" task for the following content:
	{{input_content}}

	[CONSTRAINTS]
	- Output language: {{language}}
	- Target audience: {{audience}}
	- Word count limit: under {{max_words}} words

	[OUTPUT]
	Strictly follow this format for the returned result:
	{{output_schema}}
\end{lstlisting}

When utilizing it, simply fill in different parameters to reuse the same template:

\begin{lstlisting}[style=promptstyle]
	# Scenario A: Generating Technical Blog Summaries
	domain        = "Technical Writing"
	task_type     = "Summary Extraction"
	input_content = "<Original Text>"
	language      = "English"
	audience      = "Developers with basic programming knowledge"
	max_words     = 150
	output_schema = "Three-sentence summary, each under 50 words"

	# Scenario B: Legal Clause Risk Identification
	domain        = "Legal Compliance"
	task_type     = "Risk Identification"
	input_content = "<Original Contract>"
	language      = "English"
	audience      = "Corporate Legal Staff"
	max_words     = 300
	output_schema = "JSON format containing fields: risk_level, clause, suggestion"
\end{lstlisting}

#### 3. Layered Reusability Strategy

Templates can be divided into three tiers based on reusability granularity:

- **Atomic Module Layer**: Smallest functional units, like "JSON output format instructions" or "Polite refusal dialogue", which can be referenced in any template.
- **Domain Template Layer**: Combines atomic modules into semi-finished templates aimed at specific domains (like customer service, code review, document generation).
- **Task Instance Layer**: Injects specific parameters into domain templates to generate the final executable Prompt.

\begin{lstlisting}[style=promptstyle]
	# Hierarchy Illustration

	Atomic Modules: format_json_v1  +  role_analyst_v2  +  constraint_no_hallucination_v1
	|
	V
	Domain Template: data_analysis_template_v1 (Combines the modules above)
	|
	V
	Task Instance: Analyze Q3 2024 sales data, output growth rate rankings by category (JSON)
\end{lstlisting}

#### 4. Establish a Prompt Asset Library

Centrally manage verified templates and atomic modules to form a Prompt asset library shared by the team:
- Every template should include **applicable scenario descriptions** and **parameter filling examples** to lower the barrier to entry;
- Track changes using version control tools like Git, supporting multi-person collaboration on new templates;
- Regularly review templates with low reusability rates and merge homogeneous content to keep the library clean.
Through these mechanisms, a polished generic template can cover dozens of similar task scenarios, significantly reducing the repetitive labor of writing Prompts from scratch. Additionally, because the template is validated across multiple scenarios, its stability and output quality often surpass those of hastily assembled, one-off Prompts.

</details>

