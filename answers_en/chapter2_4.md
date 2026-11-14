# 2.4 ICL Example Selection, Order, and Format — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 24 questions

**1. What is the "Format" of a Prompt? How Big is Its Impact on ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the "Format" of a Prompt? How Big is Its I](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig18_what_is_the_format_of_a_prompt.png)



Format refers to the structured presentation of the prompt, including the layout of examples, separators, the representation of labels (`Label: X` vs `Answer: X`), the alignment of inputs and outputs, etc. The impact is very large:\begin{enumerate}
- Inconsistent formatting causes the model to misunderstand the task structure, leading to disordered output formatting;\item  Under different formats for the same task, accuracy differences can reach 10–20%;\item  The format directly affects the model's understanding of "where to read the label from";\item  Good format: Clear separators (e.g., `###`, `<input>`/`<output>` tags), consistent alignment, and identical structure for each example.The importance of the format is often even higher than the quality of the example content itself.

</details>

**2. Do the Formats of the Query and Demonstration Need to be Consistent in ICL?**

<details>
<summary>💡 Show Answer</summary>

They must be completely consistent. The query and the demonstration should use the exact same formatting template, except the query is missing the output part (the part the model needs to fill in). If the formats are inconsistent:\begin{enumerate}
- The model may misjudge the task structure, not knowing "where the output should appear";\item  Format differences may be interpreted by the model as a different task, leading to output deviation;\item  Separators and label names (e.g., "Input:", "Question:") need to be absolutely identical across all examples and the query.Best practice: Use a unified template to generate all examples and the query, avoiding formatting inconsistencies caused by manual assembly.

</details>

**3. What is a "Task Description"? Is it Necessary in ICL?**

<details>
<summary>💡 Show Answer</summary>

A task description is a natural language explanation of the task preceding the examples (e.g., "Please translate the following English sentences into Chinese:", "Determine the sentiment of the following reviews, outputting positive/negative:"). Is it necessary:\begin{enumerate}
- **Zero-shot**: Without examples, the task description is the critical and sole signal, having a major impact on performance;\item  **Few-shot**: With sufficient examples, the importance of the task description decreases, as the model can infer the task from the examples;\item  **Complex tasks**: When a task has special constraints, the task description is used to supplement details that examples cannot demonstrate;\item  General recommendation: Always provide a task description; even with examples, a task description can eliminate ambiguity and reduce output instability.

</details>

**4. What Types of NLP Tasks is ICL Suitable For?**

<details>
<summary>💡 Show Answer</summary>

**Suitable for**:\begin{enumerate}
- Text classification (sentiment analysis, topic classification);\item  Named Entity Recognition (NER);\item  Text summarization;\item  Machine translation;\item  Question answering (knowledge already in the model);\item  Text generation (style transfer, continuation);\item  Code generation (tasks with fixed patterns);\item  Format conversion (JSON structured extraction).**Tasks with limited effectiveness**:
- Question answering requiring real-time knowledge (knowledge not in the model);\item  Complex numerical calculations (need to be combined with code execution);\item  Deep reasoning requiring vast amounts of specialized knowledge;\item  Extreme personalization (requiring massive user data adaptation).

</details>

**5. How Much Does the Selection of Few-shot Demonstrations Affect ICL Performance? What Are the Selection Principles?**

<details>
<summary>💡 Show Answer</summary>

The impact is extremely significant; different sets of examples can lead to a 5–20% difference in accuracy. Selection principles:

1. **Diversity**: Cover different input patterns, types, and difficulties, avoiding highly similar examples;
2. **Representativeness**: Examples should come from the real data distribution of the target task, not deliberately cherry-picked extreme cases;
3. **Quality**: The inputs and outputs of the examples must be correct and well-formatted;
4. **Relevance**: Dynamically selecting examples that are semantically similar to the current query outperforms fixed random examples by about 5%–10%;
5. **Difficulty gradient**: Examples should include a hierarchy from easy to hard to help the model understand task boundaries;
6. **Label balancing**: In classification tasks, the number of examples for each category should be kept as balanced as possible.

</details>

**6. What Are Static and Dynamic Demonstrations? What Are Their Respective Pros and Cons?**

<details>
<summary>💡 Show Answer</summary>

**Static Demonstrations**: Using a fixed, identical set of examples for all queries. Pros: Simple to implement, stable inference, no extra computational overhead. Cons: Performs poorly on queries that differ significantly from the example distribution, relevance of examples is not guaranteed.**Dynamic Demonstrations**: Real-time retrieval of the most relevant examples from an example pool based on the current query (typically using Embedding similarity). Pros: Examples are highly relevant to the query, leading to better performance; highly adaptable. Cons: Requires an additional retrieval step (increased latency, implementation complexity); retrieval quality affects the final result.Experience: Static examples are sufficient when the data distribution is relatively uniform; dynamic examples are used when distribution differences are large or when performance requirements are high, typically providing a 5–10% improvement.

</details>

**7. What is KATE (KNN-Augmented In-Context Example Selection)? What is Its Core Idea?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is KATE (KNN-Augmented In-Context Example Sel](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig20_what_is_kate_knn_augmented_in_.png)



KATE (Liu et al. 2021): Uses the K-Nearest Neighbors algorithm to select the training samples most similar to the test query as ICL examples, rather than choosing randomly. Core idea:\begin{enumerate}
- Vectorize all candidate examples using Sentence Embeddings;\item  Perform embedding on the current query;\item  Calculate the cosine similarity between the query and all candidate examples;\item  Select the top-k most similar samples as examples;\item  Sort them in descending order of similarity (placing the most relevant ones at the end, closer to the query, to leverage the recency effect).Effect: Compared to random example selection, KATE improves performance by 3–10% on various NLP tasks. Limitations: Requires pre-computing embeddings for the candidate pool, incurring certain computational and storage overhead.

</details>

**8. How to Build a High-Quality Demonstration Candidate Pool for ICL?**

<details>
<summary>💡 Show Answer</summary>

1. **Diversify sources**: Collect samples from different data sources, users, and scenarios to ensure comprehensive coverage;\item  **Quality annotation**: Manually or using a strong LLM, review the quality of the input-output pairs of each candidate example, removing low-quality samples;\item  **Deduplication**: Remove highly similar examples (similarity > 0.9) to ensure diversity within the pool;\item  **Balanced sampling**: Ensure each category has a sufficient number of high-quality samples for classification tasks;\item  **Difficulty stratification**: Maintain examples across three levels of difficulty—easy, medium, and hard—for easy retrieval as needed;\item  **Continuous updating**: As business data accumulates, periodically update the example pool by adding new patterns and removing outdated examples;\item  **Scale recommendation**: At least 20–50 high-quality samples per category, with a total pool size of 200–1000.

</details>

**9. How to Balance Diversity and Relevance of Demonstrations?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to Balance Diversity and Relevance of Demonstr](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig21_how_to_balance_diversity_and_r.png)



Both are important, but their focuses differ: **Relevance** (KATE's direction): Ensures examples match the current query, making it easier for the model to transfer; **Diversity** (Coverage direction): Ensures examples cover different input patterns of the task, helping the model understand the full scope. Balancing strategies:\begin{enumerate}
- **When task distribution is uniform**: Prioritize relevance, select the most similar examples for each query;\item  **When task distribution is non-uniform**: First, select a base set of examples for diversity, then filter by relevance;\item  **MMR Algorithm**: Maximal Marginal Relevance, balancing the relevance to the query with the differences from already selected examples during selection;\item  Experience: Typically, within 4 examples, 2 highly relevant + 2 covering diversity perform better than 4 purely relevant or purely diverse examples.

</details>

**10. How to Select the Most Relevant Demonstrations for the Current Query (Selective ICL)?**

<details>
<summary>💡 Show Answer</summary>

Mainstream methods:\begin{enumerate}
- **Embedding similarity**: Perform embedding on the query and candidate examples, selecting top-k by cosine similarity (KATE);\item  **BM25 retrieval**: Select similar examples via keyword matching, which is fast and suitable for tasks with many precise terminologies;\item  **Hybrid retrieval**: Fusion of Embedding similarity + BM25, balancing semantic and lexical matching;\item  **LLM scoring**: Ask an LLM to score the relevance of the query-example pairs, offering the highest precision but at a high cost;\item  **EPR (Efficient Prompt Retrieval)**: Specifically trains an example retrieval model using the final ICL performance as the training signal;\item  Practical advice: For most scenarios, Embedding similarity + simple BM25 is sufficient to meet requirements.

</details>

**11. What impact does the difficulty gradient in examples (from easy to hard) have on ICL performance?**

<details>
<summary>💡 Show Answer</summary>

Arranging examples from easy to hard is usually better than random order:
1. **Learning Curve Simulation**: First presenting simple cases allows the model to understand the basic pattern, and then presenting complex cases expands the boundaries;
2. **Difficulty Perception**: The model can perceive the range of task complexity, and will not underestimate the task complexity because it only sees simple examples;
3. **Recency Effect Superposition**: Placing the most complex (most similar to the hard query) examples at the end satisfies both the difficulty gradient and takes advantage of the recency effect;
4. Note: If all examples are simple cases, the model's generalization performance on complex queries will be poor; conversely, all complex examples may make the model overcomplicate simple questions.

</details>

**12. How to handle duplicate or highly similar samples in the candidate example pool?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to handle duplicate or highly similar samples](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig22_how_to_handle_duplicate_or_hig.png)



1. **Preprocessing Deduplication**: When building the example pool, calculate the Embedding similarity of all sample pairs, and only keep one (the higher quality one) for samples with similarity > 0.9;
2. **MinHash LSH**: For large-scale example pools, use MinHash Locality-Sensitive Hashing to quickly detect near-duplicates, which is more efficient than full pairwise comparison;
3. **Clustering Deduplication**: Perform clustering (K-Means) on the example pool, and only keep the representative sample closest to the center for each cluster;
4. **Online Deduplication**: During dynamic example selection, the already selected examples no longer participate in subsequent selection, avoiding selecting multiple highly similar examples;
5. **Diversity Constraints**: Add MMR (Maximal Marginal Relevance) constraints when selecting examples to naturally reduce the selection of similar examples.

</details>

**13. When task data is very scarce (e.g., 3–5 samples), how to maximize the ICL effect?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of When task data is very scarce (e.g., 3–5 samples),](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig23_when_task_data_is_very_scarce_.png)



1. **Carefully Design the Format**: The impact of formatting is greater when data is extremely scarce, so spend time optimizing the prompt structure;
2. **Optimize Task Description**: Supplement detailed task instructions to compensate for the insufficient number of examples;
3. **Data Augmentation**: Use LLM to generate more pseudo-labeled examples to expand the example pool (let GPT-4 generate more examples with similar formats);
4. **Select the Most Representative Examples**: Manually pick examples covering different input patterns to maximize diversity;
5. **CoT Examples**: If the task involves reasoning, adding reasoning steps to the few examples often yields better results than pure input-output examples;
6. **Self-Consistency**: Sample multiple times and take a majority vote to reduce the instability caused by a small number of examples;
7. **Combine with Zero-shot CoT**: When data is too scarce, Zero-shot CoT ("Let's think step by step") sometimes performs better than a few low-quality examples.

</details>

**14. How to automatically evaluate the quality of example selection strategies?**

<details>
<summary>💡 Show Answer</summary>

1. **Build an Evaluation Set**: Carve out a validation set from the target task data (not used in the example pool), and annotate the gold standard answers;
2. **Baseline Comparison**: Run random example selection and target selection strategies separately, and calculate Accuracy/F1 on the validation set;
3. **Leave-one-out Evaluation**: For each sample in the validation set, select examples from the remaining samples and evaluate the prediction accuracy;
4. **Retrieval Quality Metrics**: If it is retrieval-based selection, additionally evaluate the distribution of relevance scores between retrieved examples and the query;
5. **Stability Testing**: Run evaluations multiple times (different random seeds) and calculate the variance of accuracy; a small variance indicates a stable strategy;
6. **Cost-Effectiveness Curve**: Compare the performance and token consumption under different k values to find the optimal k.

</details>

**15. Does the order of examples affect ICL output results? What are the research conclusions?**

<details>
<summary>💡 Show Answer</summary>

It has a significant impact, which is one of the known vulnerabilities of ICL. Main research conclusions:
1. **Recency Bias**: The last few examples have the greatest impact on the model, so the most important examples should be placed close to the query (Lu et al., 2021);
2. **Order Sensitivity**: The accuracy difference caused by different orders can reach 10–20% (small models are more sensitive, large models are relatively stable);
3. **Optimal Order is Hard to Predict**: There is no universal optimal order; different tasks and models have different optimal orders;
4. Mitigation Methods: Self-Consistency (sampling multiple times with different orders and voting); specialized example ordering optimization algorithms (GlobalE, METAICL).

</details>

**16. What is the "recency bias"? How to deal with it in ICL?**

<details>
<summary>💡 Show Answer</summary>

Recency bias: The model assigns higher weights to the examples at the end of the prompt (closest to the query) while ignoring the examples at the beginning, causing the final prediction to lean toward the labels of the last few examples. Coping methods:
1. **Strategic Ordering**: Place the most relevant or important examples for the current query at the end;
2. **Diversity Balancing**: If the label distribution is uneven, place examples of minority classes at the end to compensate;
3. **Self-Consistency**: Run prompts with different orders multiple times and take the majority vote to dilute the recency bias;
4. **Calibration**: Use PMI or a content-free prompt baseline to correct the prior biases of each label;
5. **Larger Models**: Large language models are relatively less sensitive to recency bias.

</details>

**17. What impact does the label distribution of examples (e.g., positive-negative sample ratio) have on ICL?**

<details>
<summary>💡 Show Answer</summary>

The label distribution has a significant impact on ICL performance, known as "majority label bias":
1. **More Positive Samples**: The model will lean toward predicting the positive label, even if the input looks more like the negative class;
2. **Imbalanced Labels Can Be Exploited**: Intentionally constructing an imbalanced example distribution can guide the model toward a specific label (but not recommended for production);
3. **Ideal State**: Each label is evenly distributed among the examples (50% each for binary classification tasks, equal proportions for multi-class tasks);
4. **Experimental Findings**: Even if labels are randomly assigned (incorrect labels), the model's label distribution bias still exists, indicating this is not just a learning effect, but also a prior bias issue;
5. **Calibration Correction**: Using Calibration (PMI normalization) can correct the majority label bias to some extent.

</details>

**18. How to mitigate the "majority label bias" in ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to mitigate the "majority label bias" in ICL?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig24_how_to_mitigate_the_majority_l.png)



1. **Balance Examples**: Build a uniformly distributed example set from the source to avoid bias;
2. **Calibration**: Use a content-free null input to measure the model's prior probabilities for each label, and subtract this prior during prediction (PMI normalization);
3. **Contextual Calibration (CC)**: Proposed by Zhao et al., 2021, replace the input with "N/A" or a space to obtain the contextual prior of each label, and then use this prior for correction during prediction;
4. **Domain-Context Calibration (DC)**: Use unlabeled samples similar to the test distribution to estimate the prior, which is more accurate than CC;
5. **Self-Consistency**: Randomness in multiple samplings naturally disperses the bias;
6. **Explicit Task Description**: Emphasize in the task description "Please judge based on the input content and do not bias toward any specific label".

</details>

**19. Does the design of input-output separators affect ICL performance? What are the best practices?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of Does the design of input-output separators affect](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig25_does_the_design_of_input_outpu.png)



It has a noticeable impact. Clear separators help the model accurately parse the prompt structure. Best practices:
1. **Explicit Separators**: Use clear tags (`Input:` / `Output:`, `Question:` / `Answer:`, `###`) rather than pure space separators;
2. **Consistency**: All examples and the query must use exactly the same separators;
3. **XML/Markdown Tags**: For complex prompts, using structured tags (e.g., `<input>`/`</input>`) is clearer;
4. **Separation Between Examples**: Use obvious separators between different examples (e.g., `---` or double newlines);
5. **Avoid Ambiguity**: The separator itself should not appear in the input content (otherwise the model may misunderstand the structure);
6. Experiments show that switching to different styles of separators (e.g., `Q:` vs. `Question:`) can result in a 2–5% difference in accuracy.

</details>

**20. What is the impact of example length on ICL? What are the risks of excessively long examples?**

<details>
<summary>💡 Show Answer</summary>

**Examples are too short**: They do not provide enough context, and the model may not accurately understand the complexity of the task and the expected level of detail in the output. **Examples are too long**:
1. They occupy too much of the context window, reducing the available number of examples;
2. Key information is diluted, distracting the model's attention;
3. Generation speed slows down, and costs increase;
4. The model might overfit to the specific format of the examples rather than the task itself. **Rules of Thumb**: For classification tasks, an example length of 50–200 tokens is usually sufficient; examples for generation tasks can be longer (200–500 tokens), but try to keep the total token count for examples within 1000–2000;
5. If a single example is very long, consider reducing the number of examples to ensure the completeness of key examples.

</details>

**21. What is the "calibration" (alignment noise) problem? How to mitigate it using PMI and other methods?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the "calibration" (alignment noise) proble](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig26_what_is_the_calibration_alignm.png)



Alignment noise (Calibration problem): LLMs have built-in prior preferences for different labels (e.g., tending to output "positive" rather than "negative"). This preference superimposes on the learning effect during ICL, causing prediction deviations. PMI (Pointwise Mutual Information) mitigation method:
1. Use a meaningless input (e.g., "N/A") to measure the model's prior probability of outputting each label, P(label | null);
2. During actual prediction, calculate `P(label | input) / P(label | null)` (PMI normalization);
3. This eliminates the prior bias and preserves only the judgment related to the input. Other methods: Contextual Calibration (CC), Batch Calibration (BC), Domain-Calibration. Practical Significance: On tasks with significant prior bias in labels (e.g., sentiment analysis), calibration can improve performance by 3–8%.

</details>

**22. How to design the output format in ICL (structured vs. free text)?**

<details>
<summary>💡 Show Answer</summary>

Selection principles:
1. **Whether Downstream Parsing is Needed**: If the answer needs to be processed by a program (extracting fields, values), a structured format (JSON/YAML/fixed template) must be used;
2. **Task Complexity**: Use word/phrase outputs for simple classifications; free text for explanatory tasks; and CoT structures for tasks requiring intermediate steps;
3. **Structured Output Design**: Show complete JSON templates in the examples, including all fields; instruct in the task description to "output JSON only, without any explanation";
4. **Free Text Output**: Suitable for summarization, translation, and creative writing, but needs to demonstrate the desired level of detail and tone in the examples;
5. **Mixed Format**: Output reasoning steps first (free text), followed by structured conclusions (e.g., CoT + final answer label);
6. Unified format: The query and all examples must use exactly the same output format.

</details>

**23. Placing the query before vs. after the examples: which way is better?**

<details>
<summary>💡 Show Answer</summary>

Usually, **placing the query after the examples** works better. Reasons:
1. **Recency Effect**: When the query is placed at the end, the model pays the most attention to it when answering;
2. **Example Guidance**: Reading examples before the query leads to higher consistency in answering according to the example format;
3. **Few Opposite Cases**: When the query is extremely long or complex, showing the query first allows the model to read examples "with questions in mind," which sometimes helps it understand the task better;
4. **Standard Position**: The vast majority of few-shot prompt designs place the query at the end of the examples, which is also the more common format in pre-training data. Conclusion: By default, place the query after all examples, and experimentally adjust it when necessary based on task characteristics.

</details>

**24. How to test the robustness of ICL prompts to changes in example order?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to test the robustness of ICL prompts to chang](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig27_how_to_test_the_robustness_of_.png)



1. **Random Order Testing**: Generate multiple random permutations (e.g., 20 different orders) for the same set of examples, run them separately, and calculate the mean and variance of accuracy;
2. **Max/Min Difference**: Identify the best and worst orders and analyze the example characteristics that caused the difference;
3. **Recency Effect Testing**: Keep the example set unchanged, but swap only the last example (with one from a different class), and observe the degree of impact on the prediction;
4. **Robustness Score**: Smaller variance = higher robustness to order;
5. **Improvement Strategy**: If the variance is large, use Self-Consistency (multi-order sampling + voting) to improve robustness;
6. **Target**: Under 20 order configurations, an accuracy variance < 3% is considered acceptable robustness; variance > 5% requires redesigning examples or using Self-Consistency.

</details>

