# 8.2 Detailed Explanation of Core and Advanced Benchmarks — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 49 questions

**1. What are the common benchmarks for general capability evaluation?**

<details>
<summary>💡 Show Answer</summary>

MMLU, C-Eval, CMMLU, AGIEval, BIG-Bench, HellaSwag, ARC, WinoGrande, TruthfulQA, SuperGLUE, Open LLM Leaderboard series, etc.

</details>

**2. What do MMLU, C-Eval, GSM8K, MATH, and HellaSwag evaluate respectively?**

<details>
<summary>💡 Show Answer</summary>

- MMLU: Multiple-choice questions covering 57 subjects, evaluating multi-domain knowledge and comprehension capabilities.
- C-Eval: Chinese comprehensive knowledge and reasoning evaluation, covering 52 subjects.
- GSM8K: Primary school math word problems, evaluating multi-step arithmetic reasoning capability.
- MATH: Competition-level math problems, evaluating complex mathematical reasoning and symbolic operation capabilities.
- HellaSwag: Sentence completion multiple-choice questions, evaluating commonsense reasoning and linguistic coherence judgment.

</details>

**3. Please introduce several comprehensive LLM benchmarks widely used in the industry and explain their respective focus areas. (e.g., MMLU, Big-Bench, HumanEval)**

<details>
<summary>💡 Show Answer</summary>

To evaluate the capabilities of LLMs more comprehensively, academia and industry have developed many comprehensive benchmarks. Among them, MMLU, Big-Bench, and HumanEval are several of the most representative ones, each with a different focus:

1. MMLU (Massive Multitask Language Understanding)

- Focus: Knowledge breadth and subject problem-solving capabilities.
- Overview: MMLU is a large-scale multitask benchmark designed to measure a model's knowledge level across various academic and professional domains. It contains 57 different subjects, ranging from elementary mathematics, US history, and computer science to professional-level law, marketing, and medicine.
- Format: All questions are single-choice questions with four options.
- Evaluation Purpose: To test whether a model possesses extensive, cross-disciplinary knowledge reserves and the ability to apply this knowledge to solve problems. A model with a high score on MMLU is generally considered a "knowledgeable" model.

1. Big-Bench (Beyond the Imitation Game Benchmark)

- Focus: Exploring the capability boundaries and future potential of LLMs.
- Overview: Big-Bench is an extremely diverse benchmark created through community collaboration, containing over 200 tasks. These tasks are designed to be highly challenging, aiming to test capabilities that current LLMs find difficult to solve, such as commonsense reasoning, logic, physical intuition, and creative tasks.
- Format: Task formats are highly diverse, including multiple-choice questions, generation tasks, comparison tasks, etc.
- Evaluation Purpose: The goal of Big-Bench is to "predict the future." It attempts to identify new capabilities that might "emerge" once model scale or technology reaches a certain critical threshold. It measures the general intelligence level and frontier capabilities of models.

1. HumanEval (Human-Labeled Evaluation)

- Focus: Code generation and programming capabilities.
- Overview: HumanEval is a benchmark created by OpenAI specifically for evaluating code generation capabilities. It contains 164 hand-written programming problems, each providing a function signature, docstring, and several unit tests.
- Format: Models are required to generate a complete Python function body based on the function signature and docstring.
- Evaluation Method: Employs the pass@k metric. That is, the model generates $k$ code samples, and as long as at least one of them passes all unit tests, it is considered a success. This measures the model's ability to write correct and usable code.

Other important benchmarks:

- GSM8K: Focuses on evaluating reasoning capabilities on primary school-level math word problems, requiring models to perform multi-step Chain-of-Thought reasoning.
- ARC (AI2 Reasoning Challenge): Focuses on evaluating challenging multiple-choice questions that require scientific commonsense and reasoning.
- HellaSwag: Focuses on evaluating commonsense reasoning, with the task of choosing the most plausible sentence to continue a given scenario.





![Industrial LLM benchmarks and their core evaluation focus areas. (工业界主流大语言模型综合评测基准及其核心侧重点示意图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig05_comprehensive_benchmarks.png)

</details>

**4. What is MMLU? What capabilities of a model does it primarily evaluate?**

<details>
<summary>💡 Show Answer</summary>

MMLU (Massive Multitask Language Understanding) is a multiple-choice benchmark covering dozens of subject domains (such as mathematics, history, law, medicine, etc.). It is primarily used to evaluate a model's mastery of broad subject knowledge and is one of the representative test sets for measuring a model's "encyclopedic knowledge" level.

</details>

**5. What are the known limitations or issues of MMLU in evaluation?**

<details>
<summary>💡 Show Answer</summary>

These include: some questions contain annotation errors or controversial answers; the multiple-choice format may be affected by formatting factors such as option order and phrasing; as model capabilities improve, scores of many models on MMLU have approached saturation, decreasing discriminative power; and parts of the dataset may already exist in the pre-training corpora of newer models (data contamination risk).

</details>

**6. What is MMLU-Pro? What improvements does it make compared to the original MMLU?**

<details>
<summary>💡 Show Answer</summary>

MMLU-Pro makes several improvements over the original MMLU: it increases the number of options (from 4 to 10, reducing the probability of guessing correctly), filters question difficulty (removing overly simple or ambiguous questions and increasing the proportion of questions requiring reasoning), and conducts stricter human verification on certain questions, aiming to alleviate the saturation issue of the original MMLU and provide more discriminative evaluation.

</details>

**7. What is GSM8K? What capability does it primarily evaluate?**

<details>
<summary>💡 Show Answer</summary>

GSM8K is a dataset consisting of primary school-level math word problems, primarily used to evaluate a model's multi-step arithmetic reasoning capability. Models need to derive correct numerical answers through step-by-step reasoning (usually accompanied by CoT).

</details>

**8. Why do mathematical benchmarks like GSM8K typically report both Zero-shot CoT and Few-shot results?**

<details>
<summary>💡 Show Answer</summary>

Different prompt settings (whether examples are provided and whether the model is guided to output the reasoning process) significantly affect a model's performance on mathematical reasoning tasks. Reporting results under multiple settings simultaneously provides a more comprehensive reflection of the model's true capabilities under different usage conditions. It also helps assess the extent to which the model relies on example guidance to complete tasks correctly (i.e., its sensitivity to prompt settings).

</details>

**9. What is the MATH dataset? How does its difficulty and focus differ from GSM8K?**

<details>
<summary>💡 Show Answer</summary>

The MATH dataset contains problems from mathematical competitions covering multiple fields such as algebra, geometry, and number theory. Compared to GSM8K (arithmetic reasoning at the level of primary school word problems), MATH features higher problem difficulty, requiring more complex mathematical concepts and multi-step derivations, making it more suitable for evaluating a model's capability in high-level mathematical reasoning.

</details>

**10. What is HumanEval? What is it primarily used to evaluate?**

<details>
<summary>💡 Show Answer</summary>

HumanEval is a code generation benchmark containing a series of programming problems with function signatures and docstrings. Models need to generate complete function implementations, which are evaluated for pass/fail by running predefined unit tests, primarily assessing the correctness of model code generation.

</details>

**11. What is the difference between "Pass@1" and "Pass@k" in HumanEval?**

<details>
<summary>💡 Show Answer</summary>

Pass@1 refers to the probability (or proportion) of passing tests when the model generates code only once; Pass@k refers to the probability of success when the model generates $k$ candidate code samples (typically introducing diversity using a certain sampling temperature), as long as at least one passes the tests. Pass@k is commonly used to evaluate model performance in scenarios that "allow multiple attempts," where a larger $k$ value generally leads to a higher pass rate.

</details>

**12. What limitations does HumanEval have? Why were subsequent improved/supplementary versions like MBPP and HumanEval+ introduced?**

<details>
<summary>💡 Show Answer</summary>

Limitations of HumanEval include: relatively small problem scale, limited unit test coverage (faulty code might happen to pass insufficient test cases, resulting in "false positive" passes), and problems being relatively simple and potentially widely incorporated into training data. HumanEval+ reduces false positive passes by adding more comprehensive test cases; MBPP (Mostly Basic Python Problems) provides a larger and more diverse Python programming problem set with a rich difficulty gradient as a supplement.

</details>

**13. What is HellaSwag? What type of capability does it evaluate?**

<details>
<summary>💡 Show Answer</summary>

HellaSwag is a commonsense reasoning benchmark. Given a context description, it requires the model to select the most plausible completion from multiple candidates, primarily evaluating the model's understanding of everyday commonsense and situations.

</details>

**14. What is TruthfulQA? What is its design purpose?**

<details>
<summary>💡 Show Answer</summary>

TruthfulQA is a benchmark specifically designed to test whether a model mimics common "misconceptions" or "false rumors" found in training data to produce inaccurate answers. Its questions often target widespread but factually incorrect beliefs held by humans, aiming to evaluate the truthfulness of models (as opposed to answers that "sound plausible but are wrong").





![Informativeness vs Truthfulness matrix mapping model answers in TruthfulQA. (TruthfulQA 评估框架中真实性与信息量维度的象限分布图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig06_truthfulqa_framework.png)

</details>

**15. What is ARC (AI2 Reasoning Challenge)?**

<details>
<summary>💡 Show Answer</summary>

ARC is a question-answering benchmark composed of grade-school science exam questions, consisting of two difficulty levels: the "Easy Set" and the "Challenge Set." It is primarily used to evaluate a model's scientific knowledge and basic reasoning capabilities.

</details>

**16. What is BBH (BIG-Bench Hard)?**

<details>
<summary>💡 Show Answer</summary>

BBH is a subset of tasks filtered from the larger BIG-Bench benchmark that were particularly difficult for models at the time (i.e., performance under standard prompts was close to random or below human level). It covers various high-difficulty tasks including logical reasoning, algorithms, and language understanding.

</details>

**17. What is GPQA? What are its design characteristics, and why is it considered a "high-quality" knowledge benchmark?**

<details>
<summary>💡 Show Answer</summary>

GPQA (Graduate-Level Google-Proof Q&A) consists of graduate-level difficult questions authored by domain experts at the PhD level. It is specially designed such that non-experts in the field find it difficult to quickly find correct answers even when web search is allowed (i.e., "Google-Proof"). Its key characteristics are high question quality and strong domain expertise, aiming to evaluate model capabilities in expert-level knowledge and reasoning. Due to the special question design, it is relatively difficult to "take shortcuts" through simple retrieval or memorization.

</details>

**18. What is ARC-AGI (Abstraction and Reasoning Corpus)? How does its design philosophy differ from traditional knowledge benchmarks?**

<details>
<summary>💡 Show Answer</summary>

ARC-AGI consists of a series of abstract reasoning puzzles based on grid pattern transformations. Each task provides a few "input-output" example pairs, requiring the model to induce the implicit transformation rules and apply them to new inputs. Its design philosophy deliberately avoids reliance on any domain-specific knowledge (such as mathematical formulas or linguistic knowledge), focusing on evaluating a model's "few-shot inductive / abstract reasoning" capability. Such tasks are relatively easy for humans, but historically highly challenging for models (especially because models cannot "cheat" by memorizing similar problems from training data), and are regarded by some researchers as a reference dimension for measuring "artificial general intelligence."





![Abstraction and Reasoning Corpus (ARC-AGI) grid puzzle illustrating few-shot visual reasoning. (ARC-AGI 图形转换谜题示意图（展示少样本抽象视觉推理）)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig07_arc_agi_concept.png)

</details>

**19. What is IFEval? What capability does it evaluate?**

<details>
<summary>💡 Show Answer</summary>

IFEval (Instruction-Following Evaluation) focuses on evaluating a model's capability to follow "verifiable instructions" (such as "the response must contain a specific keyword," "word count must not exceed XX," "respond in all capital letters," and other formatting/constraint instructions). It objectively measures whether a model strictly complies with specific formatting requirements in the prompt via rule-based automated verification (rather than relying on human or LLM judgment).

</details>

**20. What is MGSM (Multilingual GSM8K)? What issue of models does it reveal?**

<details>
<summary>💡 Show Answer</summary>

MGSM is a multilingual mathematical reasoning benchmark constructed by translating GSM8K problems into multiple languages, used to evaluate a model's capability to complete equivalent mathematical reasoning tasks across different languages. It frequently reveals a significant gap in a model's reasoning capabilities in low-resource languages compared to high-resource languages (especially English), reflecting the imbalance of model capabilities in cross-lingual scenarios.

</details>

**21. What is MT-Bench? How does it differ from traditional benchmarks?**

<details>
<summary>💡 Show Answer</summary>

MT-Bench is a benchmark specifically designed to evaluate the multi-turn conversation capabilities of chat models. It contains a series of open-ended multi-turn questions and uses the LLM-as-Judge method to score model responses. Unlike traditional multiple-choice or exact-match benchmarks, it focuses more on evaluating a model's comprehensive performance in open-ended, multi-turn interaction scenarios.

</details>

**22. What is Chatbot Arena (e.g., LMSYS Chatbot Arena)? What evaluation method does it adopt?**

<details>
<summary>💡 Show Answer</summary>

Chatbot Arena is an evaluation platform where real users converse with two anonymous models simultaneously, compare their response quality, and vote for the better one. Based on a large volume of user votes, it uses an Elo-like rating system to compute relative rankings of models, reflecting real users' preference rankings for models in natural interaction scenarios.

</details>

**23. What are the advantages and limitations of "Crowdsourced Evaluation" (e.g., Chatbot Arena) compared to traditional expert-annotated benchmarks?**

<details>
<summary>💡 Show Answer</summary>

Advantages: The evaluation scenarios are closer to the real and diverse user usage distribution (covering real questions across various topics and styles); evaluation scale can expand naturally with user participation, with relatively low costs.

Limitations: Ordinary users' judgments may be more easily influenced by "response style" (e.g., longer answers or flashier formatting) rather than "actual content correctness/helpfulness" (especially for questions involving domain expertise, where ordinary users may be unable to accurately judge factual correctness); furthermore, the distribution of user-submitted questions itself may carry bias (e.g., leaning toward general chat scenarios rather than specific professional/task scenarios).

</details>

**24. What is SWE-bench? What scenario's capabilities does it evaluate?**

<details>
<summary>💡 Show Answer</summary>

SWE-bench is constructed based on actual issues and corresponding human fix PRs from real open-source code repositories. It requires models to generate code modifications based on issue descriptions within a given repository environment, and determines successful problem resolution by running test cases corresponding to the PR. It primarily evaluates a model's code understanding and bug-fixing capabilities in real-world software engineering scenarios.

</details>

**25. What is the "Multi-Document QA" task in "long-context benchmarks"? What is its core evaluated capability?**

<details>
<summary>💡 Show Answer</summary>

In this type of task, multiple documents are provided as context, and answering the question correctly requires synthesizing information from different documents (rather than relying solely on local information from a single document). Its core evaluated capability is the model's ability to retrieve, integrate, and reason across multiple information sources within long contexts, which is closer to real-world long-document comprehension scenarios than simple "finding an isolated piece of information in long text" (Needle-in-a-Haystack) tasks.

</details>

**26. What are "context window utilization" benchmarks (such as RULER) in evaluation? What dimensions do they focus on?**

<details>
<summary>💡 Show Answer</summary>

Building upon Needle-in-a-Haystack, such benchmarks design more diverse variants of long-context tasks (such as multiple "needles," tasks requiring aggregate statistics, and variable tracking tasks). They focus on whether a model's performance under different types of long-context utilization demands (retrieving single-point information vs. aggregating multi-point information vs. tracking state changes) exhibits significant degradation as context length increases, thereby evaluating more comprehensively the effective actual utilization of the context length the model "claims to support."

</details>

**27. Why do many frontier models emphasize performance on competition math problems like "AIME" (American Invitational Mathematics Examination) upon release? What do such benchmarks reflect?**

<details>
<summary>💡 Show Answer</summary>

Math competition problems such as AIME are far more difficult than traditional math benchmarks like GSM8K/MATH, and questions typically require long chains of creative reasoning rather than applying fixed formulas, posing an extreme challenge for models. Emphasizing performance on such benchmarks is usually intended to showcase significant improvements in "deep reasoning" capabilities compared to previous generations of models. In particular, models adopting new technical paradigms such as long Chain-of-Thought / Test-Time Compute often focus on demonstrating breakthrough progress on these high-difficulty reasoning benchmarks.

</details>

**28. Why do "competition-type benchmarks" (such as AIME and Codeforces) often experience a rapid "score surge" as new models are released? Does this imply a qualitative leap in model reasoning capability?**

<details>
<summary>💡 Show Answer</summary>

On one hand, this may indeed reflect genuine improvements in reasoning capability (especially when combined with new training paradigms such as long Chain-of-Thought / Test-Time Compute). On the other hand, caution is required: (1) competition problems (especially historical ones) carry a high risk of being incorporated into training data (since competition problems and detailed solutions circulate widely on the Internet); (2) the specific causes of a "score surge" may be the result of multiple stacked factors. Cross-validation combining the model's performance on undisclosed or newly released problems (such as continuously updated benchmarks like LiveCodeBench), as well as observing the model's robustness against "variations/perturbations" of problems, is necessary to judge more accurately to what extent score increases reflect genuine reasoning capability improvements rather than "memorization or fitting of specific problem types/patterns."





![Comparison of public benchmark score surge vs actual out-of-distribution reasoning capability. (公开评测集分数暴涨与模型真实分布外推理能力演进的对比曲线图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig08_score_surge_vs_capability.png)

</details>

**29. What is the difference in benchmark performance between "instruction-tuned models" and "base models" in evaluation? Why is it necessary to evaluate them separately?**

<details>
<summary>💡 Show Answer</summary>

Base models are trained solely through pre-training without instruction fine-tuning and usually require specific Few-shot prompt formats to perform tasks well; instruction-tuned models have undergone alignment training such as SFT/RLHF, making them better at understanding and following natural language instructions, typically performing better under Zero-shot settings. Evaluating them separately is meaningful because a base model's benchmark performance reflects the "raw capabilities" acquired during pre-training, serving as the foundation for subsequent fine-tuning; meanwhile, an instruction-tuned model's performance is closer to end users' actual experience. Together, they reflect performance levels across different links in the model capability chain.

</details>

**30. What is the "Human Baseline" of a benchmark? What is its role?**

<details>
<summary>💡 Show Answer</summary>

The human baseline refers to the performance level achieved when humans (typically laypeople or domain experts) complete the same benchmark tasks, serving as a frame of reference for measuring model performance. Its roles include: helping determine whether model performance has "exceeded human level" or "still has a significant gap," and helping assess whether a benchmark is "too simple" (if average humans can easily achieve near-perfect scores, the benchmark may lack discriminative power for evaluating advanced capabilities).

</details>

**31. What are "adversarial benchmarks" (such as the construction of ANLI)? How are they constructed?**

<details>
<summary>💡 Show Answer</summary>

Adversarial benchmarks typically adopt an iterative "Model-in-the-Loop" construction approach: annotators attempt to craft examples that can "fool" the current strongest models (causing them to give wrong answers); these successfully "fooling" samples are incorporated into new evaluation/training sets. After the model is retrained on the new data, annotators again seek new "adversarial" samples targeting the updated model, iterating continuously. This construction method inherently targets current model weaknesses, resulting in higher difficulty and stronger discriminative power compared to statically collected datasets, though it may also cause the benchmark's difficulty distribution to deviate from "natural" real-world data distributions.

</details>

**32. What is the difference between "Process Reward" and "Outcome Reward" when evaluating reasoning benchmarks? Why is "process evaluation" more difficult but potentially more valuable?**

<details>
<summary>💡 Show Answer</summary>

Outcome reward/evaluation only focuses on whether the final answer is correct; process evaluation focuses on whether each step in the reasoning process is correct and logically sound (even if the final answer happens to be correct, intermediate steps may contain errors, and vice versa). Process evaluation is more difficult because it requires fine-grained correctness judgment for every step in the reasoning chain, which itself demands substantial domain expertise and evaluation standards (typically relying on human annotation or specially trained process reward models). Its value lies in more accurately reflecting whether the model has "truly mastered" the problem-solving methodology (rather than "guessing the right answer" or arriving at the correct answer through flawed reasoning), making it more meaningful for diagnosing the true state of a model's reasoning capabilities and guiding improvement directions.





![Process Reward Model (PRM) vs Outcome Reward Model (ORM) in step-by-step reasoning evaluation. (过程奖励模型（PRM）与结果奖励模型（ORM）在推理步骤评估中的对比)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig09_prm_vs_orm.png)

</details>

**33. What is a "cost-normalized" benchmark evaluation approach? Why is this approach receiving increasing attention?**

<details>
<summary>💡 Show Answer</summary>

Cost-normalized evaluation refers to reporting benchmark scores alongside the computational resources consumed by the model to complete the task (such as token counts, API call costs, wall-clock time), using "the highest score achievable under equal cost budgets" or "the cost required to reach a target score" as comparison dimensions, rather than merely comparing "the highest score regardless of cost." This approach is receiving increasing attention because as Test-Time Compute (such as generating longer Chains-of-Thought or performing multi-sample voting) becomes a common mean to boost benchmark scores, comparing purely "highest scores" may evolve into a competition of "who is willing/able to spend more inference compute." In practical applications, cost is always an unavoidable constraint; thus, comparing model performance under identical cost constraints carries greater practical reference value.

</details>

**34. What is the impact of "multiple sampling + majority voting" (Self-Consistency) on benchmark scores in evaluation? What needs to be noted when reporting results under this evaluation setting?**

<details>
<summary>💡 Show Answer</summary>

Self-Consistency refers to having the model generate multiple independent responses to the same prompt (e.g., with sampling temperature > 0) and selecting the final answer via majority voting. This generally improves scores on certain reasoning tasks (since a single sample may fail due to randomness, whereas the "majority opinion" of multiple samples is more likely to be correct). When reporting results, it is necessary to explicitly state whether evaluation results are based on "single sampling (Pass@1, Temperature=0 or low temperature)" or "multiple sampling + voting." Scores under these two settings cannot be directly compared, and the latter actually corresponds to several times the compute cost of single inference. Configurations such as the number of sample iterations should be clearly annotated in reports to avoid misleading cross-model comparisons.

</details>

**35. What is the relationship between a model's "benchmark leaderboard score" and its "actual deployment cost" in evaluation? Why is it often necessary to consider both on leaderboards?**

<details>
<summary>💡 Show Answer</summary>

Different models may achieve similar scores on a given benchmark, but their parameter sizes, inference speeds, and API call costs can differ by orders of magnitude. Many leaderboards (such as Chatbot Arena and certain benchmark sites) display model capability scores alongside cost/speed metrics simultaneously (even plotting cost-performance scatter diagrams), helping users strike a comprehensive balance between "capability" and "cost" rather than making model selection decisions based solely on a single score ranking.

</details>

**36. Why is a "single composite leaderboard score" of limited reference value for model selection? How should benchmark leaderboard information be used more rationally?**

<details>
<summary>💡 Show Answer</summary>

A single composite score is typically a weighted average of scores across multiple different task/capability dimensions. However, different application scenarios weigh each capability dimension very differently (e.g., code assistants prioritize programming scores, while QA products prioritize knowledge scores). A composite score may mask a model's significant shortcomings or strengths in specific critical dimensions. A more rational approach is to focus on sub-item benchmark performance most relevant to one's own business scenario, cross-reference multiple benchmarks from different sources/types (to avoid single-leaderboard bias), and ultimately rely on internal evaluations closely tied to one's business as the primary basis for decision-making. Public leaderboards are better suited for initial screening of candidate models.

</details>

**37. What is the impact of "implementation differences in scoring criteria / answer extraction logic (parsers)" on model rankings in benchmarks?**

<details>
<summary>💡 Show Answer</summary>

For benchmarks requiring the extraction of final answers from free-form model outputs (such as extracting numerical answers from long reasoning text or code from code blocks), the robustness of "answer extraction" logic (regular expressions, parsing rules) implemented across different evaluation frameworks can vary. For instance, if a framework's parser fails to handle a specific output format unique to a certain model (e.g., using different delimiter symbols), that model's score may be erroneously depressed even if its actual answer is correct. This means that reported benchmark scores for the exact same model across different evaluation frameworks or reproductions may vary; subtle score gaps may not reflect true differences in model capabilities, but rather stem from differences in evaluation implementation details.

</details>

**38. What is the risk of "overfitting leaderboards" in evaluation, and how does the community typically address it?**

<details>
<summary>💡 Show Answer</summary>

This refers to model development teams continuously monitoring scores on public leaderboards and adjusting training strategies or data ratios specifically to boost leaderboard scores (optimizing for the leaderboard itself rather than true capability). This phenomenon is analogous to "overfitting the validation set" in machine learning, except the validation set has become a public leaderboard that any team can repeatedly game. Common community countermeasures include: periodically replacing or updating evaluation problems (such as the aforementioned "dynamic benchmarks"), introducing "hidden test set" mechanisms with submission limits or costs (where developers cannot directly see specific questions and can only submit models to obtain scores), and emphasizing cross-validation across multiple independent benchmarks from different sources rather than relying on a single leaderboard ranking.

</details>

**39. What is the "saturation" problem in evaluation?**

<details>
<summary>💡 Show Answer</summary>

It refers to the phenomenon where, as model capabilities improve, more and more models achieve scores near perfection on a given benchmark, causing the benchmark to lose its ability to discriminate capability differences among different models. Consequently, new and more challenging benchmarks must be designed to continue measuring advancements in model capabilities.

</details>

**40. Why do some researchers believe that "existing static benchmark systems will eventually be 'solved' (saturated)"? What insights does this offer for benchmark design paradigms?**

<details>
<summary>💡 Show Answer</summary>

As model training data scale and quality continuously increase, coupled with advances in model capabilities themselves, any fixed set of problems will eventually be "solved" by models (either due to genuine mastery of the underlying capability or because problem contents and similar patterns have leaked into training data). This suggests that benchmark design should evolve toward a "dynamic" and "continuously updated" paradigm—moving away from relying on fixed problem sets as long-term standards, and instead establishing a system of "continuously generating new problems, periodically evaluating, and tracking relative rates of progress." The focus shifts from a model's "absolute score on a fixed benchmark" to "relative trends of capability growth over time and performance on the latest, undisclosed problems."

</details>

**41. What is a sliding-window (or dynamically updated) evaluation set, and why is it needed?**

<details>
<summary>💡 Show Answer</summary>

It refers to an evaluation set that is periodically updated and replaced with new, undisclosed problems, rather than using a fixed set of problems over a long period. This is designed to tackle data contamination—as fixed problem sets are increasingly likely to be incorporated into the training data of new models over time, scores gradually lose discriminative power, necessitating the continuous introduction of new problems to maintain evaluation validity.

</details>

**42. What is the design motivation behind "real-time updating" benchmarks like LiveBench and LiveCodeBench?**

<details>
<summary>💡 Show Answer</summary>

The design motivation is to mitigate the issues of traditional static benchmarks gradually being absorbed into new models' training data (data contamination) over time and suffering from "difficulty saturation." By continuously collecting new problems released after known model training cutoff dates (such as newly published programming contest problems) and regularly updating the test set, the evaluation results are more likely to reflect a new model's "true capabilities" rather than "memorization of existing problems."

</details>

**43. What is the phenomenon of "Benchmark Hacking" (optimization/cheating targeted at evaluations)? What are its typical manifestations?**

<details>
<summary>💡 Show Answer</summary>

Benchmark Hacking refers to model developers (intentionally or unintentionally) performing "targeted optimization" for a well-known benchmark to inflate the model's score on that benchmark, though such gains may not generalize to real capabilities outside the benchmark. Typical manifestations include: mixing samples highly similar to (or rewritten from) benchmark questions into training data (an active form of data contamination); performing format-level "adaptation" training targeted at a benchmark's specific output format or parser logic; and excessively adjusting training data ratios for a few widely watched benchmarks, leading to models that "significantly lead" on those benchmarks but perform mediocrely on unmonitored capability dimensions or newly released benchmarks of similar types.

</details>

**44. What is the issue of "benchmark metadata / construction process transparency"? Why is it important for the credibility of evaluation results?**

<details>
<summary>💡 Show Answer</summary>

This refers to whether benchmark creators publicly disclose information such as problem sources and construction methodology, data contamination detection procedures, annotation quality control pipelines, and specific prompt templates and scoring logic used during evaluation. Low-transparency benchmarks (e.g., with unknown problem sources or lacking construction documentation) make it difficult for users to assess whether data contamination risks exist, whether scoring is fair and consistent, and to reproduce or verify reported results. Therefore, construction transparency is a crucial factor in judging the overall credibility of a benchmark.





![Standard transparent pipeline for benchmark dataset curation and audit trail. (数据评测集构建的透明化流程与元数据审计链路图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig10_benchmark_transparency.png)

</details>

**45. What is the importance of the "difficulty gradient" in benchmark design? How should a well-designed benchmark cover the difficulty gradient?**

<details>
<summary>💡 Show Answer</summary>

If benchmark problem difficulties are overly concentrated (where either most models pass easily or most fail completely), the benchmark will have very low discriminative power for distinguishing models of different capability levels. A well-designed benchmark should cover a continuous difficulty gradient ranging from tasks that "current models can handle well" to those that "even the strongest current models find difficult." This ensures that scores of models at different capability levels are meaningfully spread out and enables the benchmark to retain discriminative power over a longer period as overall model capabilities advance (rather than saturating rapidly).

</details>

**46. What is the impact of "task definition ambiguity" on benchmark fairness in evaluation? Give an example.**

<details>
<summary>💡 Show Answer</summary>

If the "correct answer" or "success criterion" of certain tasks in a benchmark involves subjectivity or permits multiple reasonable interpretations (such as open-ended creative writing tasks, or programming problems with multiple valid solutions where the evaluation accepts only one specific implementation as "correct"), different models may be judged as "incorrect" simply because they adopt equally valid approaches that do not match the exact "gold standard answer." This prevents benchmark scores from accurately reflecting true model capabilities and is particularly unfair to models inclined toward non-standard yet valid solutions. This requires benchmark designs to clarify task definitions as much as possible and adopt evaluation standards capable of accommodating multiple reasonable solutions (such as using "execution and passing tests" rather than "verbatim matching with reference code" for coding tasks).

</details>

**47. Why does a "high benchmark score" not fully represent "good performance in real business scenarios"?**

<details>
<summary>💡 Show Answer</summary>

Benchmarks typically focus on specific, relatively narrow, and format-standardized task distributions, whereas real business inputs are often more diverse, "messier," and involve more complex multi-turn interactions and domain-specific knowledge. Furthermore, models may have undergone targeted optimization for public benchmark task distributions (whether intentionally or because training data contains similarly distributed content), causing the correlation between benchmark performance and real-world scenarios to diminish. This is why many teams build internal "business-customized evaluation sets."

</details>

**48. Looking ahead, how might the development trends of "general capability benchmarks" and "vertical domain-specific benchmarks" diverge?**

<details>
<summary>💡 Show Answer</summary>

As general large language models converge on mainstream general capability benchmarks (with multiple top models scoring similarly), the "discriminative power" and marginal reference value of general benchmarks for model selection may gradually decline. Simultaneously, as LLMs deepen their application in more vertical domains (such as healthcare, law, finance, and scientific research), professional and in-depth domain-specific benchmarks (which typically require domain experts' participation in design and annotation) will gain increasing importance. The focus of evaluation may gradually shift from "how smart the model is overall" to "whether the model reaches a practical, trustworthy, and deployable level in specific domain-specific areas." The construction and maintenance of vertical domain benchmarks may themselves become critical components of industry ecosystems.

</details>

**49. Overall, what critical thinking framework should be adopted when using any benchmark results for decision-making?**

<details>
<summary>💡 Show Answer</summary>

It is recommended to examine from the following perspectives: (1) whether the specific capabilities measured by the benchmark are relevant to your actual application scenario; (2) whether the benchmark suffers from known issues such as data contamination, saturation, or shortcut learning, and whether the scores retain discriminative power and reference value; (3) under what evaluation settings the reported scores were obtained (Zero-shot/Few-shot, single/multiple sampling, whether Chain-of-Thought was used, etc.), and whether scores across different settings are comparable; (4) whether the scores factor in practical deployment constraints such as cost and latency; (5) whether conclusions from a single benchmark can be corroborated by evaluations from other independent sources or internal testing. Benchmark scores should be viewed as "a reference signal" of model capability rather than a "final verdict," and ultimate decisions should be grounded in comprehensive evaluations closely aligned with one's specific business scenarios.





![A critical verification framework for model selection based on benchmark evaluation. (基于评测结果进行模型选型决策的批判性验证流程图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig11_critical_evaluation_framework.png)

</details>

