# 2.5 ICL Mechanisms, Theories, and Optimization Strategies — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 33 questions

**1. What is the core idea of the "implicit gradient descent" hypothesis (Akyürek et al.) for ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the core idea of the "implicit gradient de](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig28_what_is_the_core_idea_of_the_i.png)



Akyürek et al., 2022 proposed that during the forward pass, the Transformer's attention mechanism implements a gradient descent process similar to linear regression. Core ideas:
1. For linear prediction tasks, certain layers of a Transformer are mathematically equivalent to performing one step of gradient descent in the activations space;
2. Few-shot examples play the role of "training data" in this framework, and attention heads perform "implicit weight updates" on the examples;
3. This update only occurs in the activations layer (forward pass) and does not change actual model parameters;
4. Thus, ICL can be understood as "rapid learning happening in the activation space at inference time." Impact: This hypothesis provides a mathematical explanation for the ICL mechanism, but is currently only strictly proven on simple linear tasks; its generalization to complex tasks is still under study.

</details>

**2. What is the difference between the "task recognition" hypothesis and the "task learning" hypothesis in ICL?**

<details>
<summary>💡 Show Answer</summary>

**Task Recognition Hypothesis**: The model does not "learn" new input-output mappings from the examples during ICL, but merely uses the examples to identify what task type it is, and then activates existing capabilities from pre-training. Examples act as "keys," not "textbooks." **Task Learning Hypothesis**: The model does indeed learn input-output mapping relationships from the examples to some extent, and this learning occurs at the activation level of attention. Verification of the difference: Min et al., 2022 found that random labels have little impact on ICL performance, supporting the "task recognition" hypothesis (labels are not important; what matters is the format and input distribution); however, Wei et al., 2023 found that on sufficiently large models, labels do have an impact, supporting the "task learning" hypothesis. Current consensus: Both exist; task recognition is the foundation, and task learning provides additional improvement.

</details>

**3. What did Min et al., 2022 discover regarding "label noise does not affect ICL"? What does this indicate?**

<details>
<summary>💡 Show Answer</summary>

Experimental findings: When randomly replacing the labels in few-shot examples with incorrect labels (e.g., marking positive as negative in sentiment analysis), the drop in the model's ICL performance is much smaller than expected (usually only drops 0–5%), and sometimes it does not drop at all. This indicates:
1. **Format > Labels**: The most critical information ICL extracts from examples is the input-output format, not the label mapping relationship;
2. **Input Distribution**: Examples demonstrate "what the inputs for this kind of task look like," which is more helpful to the model;
3. **Pre-trained Knowledge Dominates**: The model primarily relies on knowledge acquired during pre-training to complete the task, and examples are merely activation signals;
4. Practical Takeaways:
    - The standardization of the example format is more important than whether the labels are completely correct;
    - However, this conclusion is debated on large models and complex tasks (Wei et al. found that labels are still effective);
    - Careful annotation still holds value, especially for non-standard tasks.

</details>

**4. Does the model in ICL rely more on the "input-output format" or the "label correctness"?**

<details>
<summary>💡 Show Answer</summary>

Synthesizing current research, it **relies more on the input-output format**, but both have an impact:
1. **Role of Format** (Stronger): The format dictates "how the model organizes the output," and inconsistent formatting will significantly degrade performance; correct formatting is a foundational condition for ICL to work effectively;
2. **Role of Labels** (Weaker but existent): On standard tasks, the impact of random labels is small (Min et al.); on non-standard tasks (label spaces the model has never seen), the impact of labels is larger (Wei et al.);
3. **Impact of Model Size**: Larger models utilize labels more fully;
4. **Impact of Task Complexity**: For simple tasks, format is sufficient; for complex tasks, the impact of labels is more significant. Practical conclusion: First ensure complete correctness and consistency of formatting, then optimize label quality; both are important, but format is the prerequisite.

</details>

**5. What is the "Bayesian inference" perspective on ICL?**

<details>
<summary>💡 Show Answer</summary>

Bayesian perspective (Xie et al., 2021): ICL can be understood as the model performing Bayesian posterior inference on the "task concept" that generated the current prompt. Formalization:
1. In pre-training data, there is a large number of structures containing "multiple examples under the same concept" (e.g., textbook exercises);
2. The model implicitly learns to infer the underlying task concept (posterior) by observing examples (evidence);
3. Few-shot examples are observed data, and the task concept is the latent variable;
4. The model uses Bayesian inference to update its belief about the task concept and generates answers accordingly. Significance: This perspective explains why ICL performance improves as examples increase (more evidence $\rightarrow$ more accurate posterior), and why the example distribution is important (prior + likelihood $\rightarrow$ posterior).

</details>

**6. How does ICL capability scale with model size? Is there a scale threshold?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How does ICL capability scale with model size? Is](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig29_how_does_icl_capability_scale_.png)



Relationship between ICL capability and scale:
1. **Emergent Threshold**: Basic ICL capability (few-shot for simple tasks) begins to emerge at around 7B–13B parameters;
2. **Continuous Improvement**: As scale goes from 13B to 70B to 175B+, ICL performance consistently improves, with more noticeable gains on complex tasks;
3. **Higher Threshold for CoT**: The effectiveness of CoT Prompting requires roughly 100B+ parameters;
4. **Emergent Discontinuity**: Certain capabilities (like multi-step mathematical reasoning) suddenly appear at specific scale points, rather than growing linearly;
5. **Task Difficulty Impact**: Simple classification tasks are usable on 7B models; complex reasoning tasks only achieve satisfactory results on 70B+ models. Practical significance: When choosing model size, it's necessary to consider the target task complexity and the corresponding ICL capability threshold.

</details>

**7. How does the distribution of pre-training data affect ICL capabilities?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How does the distribution of pre-training data aff](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig30_how_does_the_distribution_of_p.png)



1. **Data Diversity**: The more diverse the pre-training data (covering more task types, formats, and domains), the stronger the generalization capability of ICL;
2. **Task Format Coverage**: Pre-training data containing a massive amount of "task description + example + answer" structures (like Stack Overflow Q&As, textbook examples) is the source of ICL capability;
3. **Code Data**: Code training data, due to its highly structured nature and explicit input-output formats, significantly enhances the format-following capability in ICL;
4. **Diverse Instruction Data**: Models like FLAN and T0, fine-tuned with a large volume of diverse instructions, show significantly enhanced ICL capabilities (especially Zero-shot);
5. **Domain-Specific Data**: If pre-training data has little coverage of a certain domain, ICL performance in that domain is usually poor and requires domain-relevant examples to compensate.

</details>

**8. Why is ICL considered a form of "meta-learning"? What are its similarities and differences with traditional meta-learning?**

<details>
<summary>💡 Show Answer</summary>

**Reasons for being considered meta-learning**: ICL achieves the capability to "learn how to learn from examples" — pre-training is equivalent to training a learning algorithm on a vast array of tasks, and during testing (ICL), this algorithm only needs a few examples to quickly adapt to new tasks. Comparison with traditional meta-learning (e.g., MAML):
1. **Similarities**: Both aim to generalize rapidly from a few samples; both rely on extensive pre-training across diverse tasks;
2. **Differences**: Traditional meta-learning explicitly optimizes the "ability to adapt quickly to new tasks"; ICL is a byproduct of pre-training without an explicit meta-learning objective; traditional meta-learning still performs gradient updates during inference; ICL completely avoids parameter updates; the concept of "task" is explicit in traditional meta-learning; task recognition in ICL is implicit.

</details>

**9. Is "in-context learning" genuine learning or merely pattern matching? How to differentiate them experimentally?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of Is "in-context learning" genuine learning or merel](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig31_is_in_context_learning_genuine.png)



This is a core debate in the research community. "Pattern matching" perspective: ICL merely recognizes the prompt structure and activates the corresponding task knowledge from pre-training, without genuinely "learning" a new mapping. "Genuine learning" perspective: Akyürek et al. demonstrated that Transformers can achieve gradient descent at the activation level, suggesting ICL indeed updates some "implicit state." **Differentiating experimental designs**:
1. Test ICL performance on label spaces the model has never seen before (e.g., using "flurp" instead of "positive") — if it can adapt to new labels, it indicates genuine learning;
2. Compare the use of true labels vs. random labels vs. semantically opposite labels to analyze the degree of label utilization;
3. Test whether ICL can learn function mappings that never appeared in the pre-training data;
4. Current consensus: Both pattern matching and genuine learning exist, and their ratio varies with task complexity and model size.

</details>

**10. How does the attention mechanism achieve "implicit parameter update" in ICL? Please explain from a mechanistic level.**

<details>
<summary>💡 Show Answer</summary>

Based on the theory by Akyürek et al., 2022: For a linear prediction task, a Transformer's single-layer attention mechanism `Attn(Q, K, V) = softmax(QKᵀ/√d)V` can be proven to be equivalent to performing a one-step gradient descent on a linear model. Specific derivation:
1. Treat K (keys) as "training inputs" and V (values) as "training labels";
2. Q (query) is the test input;
3. The attention weight `softmax(QKᵀ)` is equivalent to calculating the similarity between the test input and training inputs;
4. The final output `Σ weight_i * V_i` acts like a weighted prediction based on similarity, which under certain settings is equivalent to a prediction after one step of gradient descent. Olsson et al., 2022 further proposed that "induction heads," which specialize in "completing repetitive patterns in sequences," are the key attention heads realizing ICL and serve as its underlying mechanism.

</details>

**11. How to systematically optimize the performance of an ICL task? Provide a complete tuning workflow.**

<details>
<summary>💡 Show Answer</summary>

**Complete Tuning Workflow**:
1. **Establish Baseline**: First test with Zero-shot and record current performance;
2. **Example Quantity Experimentation**: Gradually increase from 1-shot to 8-shot to find the point of diminishing returns;
3. **Example Quality Optimization**: Manually screen for high-quality, diverse examples and remove low-quality ones;
4. **Example Selection Strategy**: Upgrade from static random examples to dynamic retrieval like KATE, and evaluate the improvement;
5. **Format Optimization**: Test different separators, label names, and arrangement styles;
6. **Reasoning Strategy**: Add CoT prompting for tasks involving reasoning;
7. **Order Optimization**: Place the most relevant examples closest to the query;
8. **Self-Consistency**: For complex tasks, sample multiple times and vote;
9. **Calibration**: If label bias exists, use Calibration to correct it;
10. **Comparison with Fine-tuning**: If ICL reaches a performance bottleneck, use data generated by ICL for Fine-tuning.

</details>

**12. What is Automatic Prompt Optimization (APO)? What are the representative methods?**

<details>
<summary>💡 Show Answer</summary>

APO: Automatically optimizing prompts (including ICL examples and task descriptions) to reduce manual trial and error. Representative methods:
1. **APE (Automatic Prompt Engineer)**: Uses an LLM to generate numerous candidate prompts, evaluates them on a validation set, and selects the best one (Zhou et al., 2022);
2. **OPRO (Optimization by PROmpting)**: Treats prompt optimization as an optimization problem and uses an LLM as the optimizer (Yang et al., 2023), generating better prompts iteratively based on historical performance;
3. **TextGrad**: Analogizes textual feedback (LLM's analysis of failed cases for the current prompt) to "gradients" to iteratively optimize prompts;
4. **DSPy**: Declaratively defines modules of a prompt and uses a Teleprompter to automatically optimize ICL examples and instructions;
5. **PromptBreeder**: Uses an evolutionary algorithm to optimize prompts, iteratively improving them via mutation and selection.

</details>

**13. What is Prompt Tuning? What are its connections and differences with ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Prompt Tuning? What are its connections an](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig32_what_is_prompt_tuning_what_are.png)



Prompt Tuning (Lester et al., 2021): Adds a continuous vector of learnable parameters (soft prompt) before the model input. These vectors are trained on downstream task data via gradient descent while the rest of the model parameters remain frozen. Connection with ICL: Both adapt to downstream tasks by modifying the input side rather than updating all model parameters. Differences:
1. **Parameter Updates**: ICL updates absolutely no parameters; Prompt Tuning updates the soft prompt vectors;
2. **Data Requirements**: ICL requires only few-shot examples; Prompt Tuning requires an annotated training set;
3. **Performance**: Prompt Tuning generally outperforms ICL (especially with large amounts of data);
4. **Flexibility**: ICL can switch instantly; Prompt Tuning requires separate training for each task;
5. **Interpretability**: ICL examples are human-readable; Prompt Tuning soft prompt vectors are non-readable.

</details>

**14. What is Instruction Induction? How to let the model automatically deduce task instructions from examples?**

<details>
<summary>💡 Show Answer</summary>

Instruction Induction (Honovich et al., 2022): Providing the model with several input-output examples (without a task description) and asking the model to automatically infer and generate the task instructions ("What task do these examples represent? Please provide the instructions for this task"). Application scenarios:
1. **Reverse Task Discovery**: Inferring task descriptions from existing datasets, used to automatically construct instruction fine-tuning data;
2. **Task Description Optimization**: Having the model generate multiple candidate task descriptions and selecting the optimal one;
3. **Zero-Annotation Scenarios**: When there are no task descriptions, first use a few examples for the model to infer one, then use the inferred description for ICL. Effect: On simple tasks, models can accurately deduce descriptions like "This is a sentiment analysis task"; inference quality drops on complex tasks. Relationship with APE: APE generates prompt candidates $\rightarrow$ selects the best; Instruction Induction infers descriptions from examples.

</details>

**15. How to use ICL data to generate high-quality fine-tuning training sets (Self-Play approach)?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to use ICL data to generate high-quality fine-](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig33_how_to_use_icl_data_to_generat.png)



Self-Play approach (used in Alpaca, WizardLM, etc.):
1. **Seed Data**: Start with a small number of high-quality, human-written examples;
2. **Data Augmentation**: Use a powerful LLM (like GPT-4) with the seed examples as ICL examples to generate large amounts of new instructions and responses in a similar format;
3. **Quality Filtering**: Use an LLM-as-judge or rules to filter out low-quality generated data (too short, repetitive, hallucinatory);
4. **Diversity Enhancement**: Actively increase the diversity of topics, difficulty, and style;
5. **Iterative Refinement**: Fine-tune a model with the generated data, and use the fine-tuned model to generate the next round of higher-quality data (iterative distillation). Note: This route depends on the capabilities of the original strong LLM; if the seed model's quality is poor, the generated data will also be poor. Model Collapse requires continuously mixing in real data to prevent.

</details>

**16. What is Calibration? How to improve ICL classification accuracy using calibration?**

<details>
<summary>💡 Show Answer</summary>

Calibration is a technique to correct an LLM's prior bias toward different labels. Implementation steps:
1. **Estimate Priors**: Construct a prompt with a meaningless input (N/A, space, or random word) and measure the model's output probabilities (priors) for each label when there is no effective input;
2. **Calibrate Predictions**: During actual prediction, divide the model's output probability by the prior probability (or calculate PMI) to obtain the "net probability";
3. **Re-rank**: Select the label with the highest net probability as the prediction. Effect: In classification tasks like sentiment analysis and natural language inference, Calibration can improve accuracy by 2–8%. Variants:
4. Contextual Calibration (CC): Uses unlabeled samples similar to the test distribution to estimate priors (more accurate than N/A);
5. Batch Calibration (BC): An efficient calibration method for batch calculations.

</details>

**17. How to address the performance degradation of ICL on long-tail tasks (tasks rarely seen in training data)?**

<details>
<summary>💡 Show Answer</summary>

1. **Enhance Task Descriptions**: Provide more detailed task descriptions for long-tail tasks to compensate for a lack of examples;
2. **Diversify Examples**: With limited examples, try to cover different input patterns and edge cases;
3. **Cross-Task Transfer**: Use examples from related tasks as support, and explain the relationship between the two in the task description;
4. **CoT Enhancement**: Include detailed reasoning steps in the examples, which conveys more information than direct input-output examples;
5. **Data Augmentation**: Use an LLM to generate synthetic examples for long-tail tasks to expand the example pool;
6. **Prompt Chaining**: Break down long-tail tasks into multiple sub-tasks that the model is familiar with;
7. **Fine-tuning as Fallback**: If ICL performance remains poor, using a small amount of data for PEFT (like LoRA) can often significantly improve long-tail task performance.

</details>

**18. How to use ICL for Data Augmentation?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to use ICL for Data Augmentation?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig34_how_to_use_icl_for_data_augmen.png)



Using the ICL capabilities of LLMs to generate extra training data:
1. **Synonymous Rewriting**: Provide original samples as examples and have the model generate multiple versions with different phrasing but identical semantics;
2. **Negative Example Generation**: Provide positive examples and let the model generate corresponding negative examples (generating wrong answers in classification tasks as hard negatives);
3. **Conditional Generation**: Given a specific label, ask the model to generate the corresponding input (reverse data augmentation);
4. **Style Transfer**: Rewrite original samples into different styles (formal/informal, long/short) to expand data diversity;
5. **Cross-lingual Augmentation**: Generate translated versions in target languages;
6. **Quality Control**: After generation, use rules (length, format) and an LLM-judge to filter out low-quality data;
7. Note: The diversity of ICL-generated data is constrained by the original examples and cannot cover distributions outside the model and examples.

</details>

**19. What is RLPROMPT? How does it use reinforcement learning to optimize ICL prompts?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is RLPROMPT? How does it use reinforcement le](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig35_what_is_rlprompt_how_does_it_u.png)



RLPROMPT (Deng et al., 2022): Treats prompt optimization as a sequential decision-making problem, utilizing reinforcement learning to train a "prompt generation policy network." It uses the ICL performance on the target task as the reward signal to optimize generating the most effective prompts. Workflow:
1. The policy network (a small LM) generates a sequence of prompt tokens;
2. The generated prompt is inserted into the target task, and a large LM performs ICL inference;
3. Calculate the reward (performance improvement on the target task);
4. Use PPO to update the policy network parameters;
5. Iterate until convergence. Characteristics: The generated prompt may not be human-readable (it could have strange word orders) but is effective for the model; it is complementary to APE (which generates readable prompts). Limitations: High training cost; poor interpretability of generated prompts; limited transferability across different models.

</details>

**20. What is Gradient-based Prompt Search (like AutoPrompt)? How does it relate to ICL?**

<details>
<summary>💡 Show Answer</summary>

AutoPrompt (Shin et al., 2020): Uses gradient search methods (like Hotflip/GCG) to search the vocabulary for trigger tokens that maximize target task performance, acting as a discrete approximation of a soft prompt. Workflow:
1. Initialize a random sequence of trigger tokens;
2. Use gradients to compute which vocabulary word should replace each trigger token to maximize loss reduction;
3. Iteratively search to find the optimal discrete trigger sequence. Relationship with ICL:
4. Both adapt to tasks by modifying prompts rather than model parameters;
5. AutoPrompt-generated prompts can be used to replace manually designed ICL examples, yielding better results on specific tasks;
6. However, AutoPrompt-generated triggers are unreadable (e.g., "atmosphere depicted realize mathematics") and cannot be understood and adjusted by humans like ICL examples;
7. ICL is more practical (requires no gradient calculations), whereas AutoPrompt is more automated but opaque.

</details>

**21. What problems does the "context length limit" of ICL cause? How to deal with them?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What problems does the "context length limit" of I](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig36_what_problems_does_the_context.png)



Problems:
1. Limited number of examples (context window is finite, more examples mean less space for other content);
2. Overly long examples get truncated, compromising completeness;
3. Early examples in long contexts are ignored (lost in the middle);
4. Token costs scale linearly with context length. Coping methods:
5. **Compress Examples**: Simplify the length of each example, retaining key information and removing redundancy;
6. **Reduce Example Quantity**: 3–4 high-quality examples are better than 8 low-quality ones;
7. **Combine RAG with ICL**: Use retrieval techniques to dynamically select the most relevant examples, reducing irrelevant examples occupying the context;
8. **Summary-based**: Generate summaries for long examples for display instead of the original text;
9. **Long Context Models**: Use models that support 128K+ context;
10. **Substitute with Fine-tuning**: When the data amount exceeds context limits, shift to fine-tuning.

</details>

**22. On which task types does ICL perform poorly? Why?**

<details>
<summary>💡 Show Answer</summary>

1. **Complex Numerical Calculation**: LLMs are inherently not good at precise calculation; CoT can only mitigate, not completely solve (requires code execution);
2. **Highly Specialized Knowledge**: In domains with low coverage in pre-training data, ICL cannot compensate for the lack of knowledge (such as the latest regulatory details);
3. **High-Precision Long Text Generation**: Such as contracts and reports with precise formats, ICL examples are difficult to fully constrain complex formats;
4. **Tasks Requiring Continuous Memory**: Multi-turn interactions exceeding the context window, ICL cannot persist memory;
5. **Low-Resource Languages**: The language appears rarely in pre-training data, resulting in poor ICL performance, and fine-tuning yields greater benefits;
6. **Adversarial Robustness Tasks**: ICL is sensitive to changes in example distribution and its performance drops significantly on out-of-distribution samples.

</details>

**23. Compared to Fine-tuning, when must you choose Fine-tuning over ICL?**

<details>
<summary>💡 Show Answer</summary>

Situations where Fine-tuning is mandatory:
1. **Abundant Data**: With 1000+ high-quality labeled samples, Fine-tuning performance will significantly surpass ICL;
2. **Extremely High Consistency Requirements**: Output format and style require extremely high consistency, which the randomness of ICL is hard to meet;
3. **Cost Sensitivity**: When reused massively, the ICL cost of passing examples for every inference (token cost) is much higher than a one-time fine-tuning cost;
4. **Latency Sensitivity**: Long prompts of ICL increase TTFT (Time To First Token), whereas inference after Fine-tuning is faster;
5. **Private Knowledge Injection**: When a large amount of private knowledge needs to be "internalized" into the model, which RAG+ICL cannot cover;
6. **Security and Alignment**: When robust security filtering is required, RLHF/DPO fine-tuning is a more reliable solution.

</details>

**24. How to handle "language bias" (the model's preference for a certain language style output) in ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to handle "language bias" (the model's prefere](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig37_how_to_handle_language_bias_th.png)



1. **Explicit Format Instructions**: Explicitly specify the output language and style in the prompt ("Please answer in concise English, no more than 100 words");
2. **Consistent Example Language**: Ensure that the output language and style of all examples are consistent with expectations;
3. **Negative Examples**: Provide examples of undesired output styles in the prompt and label them as "Incorrect Example";
4. **Post-processing Filtering**: Use rules to detect whether the output deviates from the expected language/style, and resample if not met;
5. **Temperature Adjustment**: Lower the Temperature (0.1–0.3) to reduce output diversity and improve format consistency;
6. **Evaluate Diversity**: Count the output language distribution on the test set to quantify the degree of bias, and then optimize accordingly;
7. **Thoroughly Resolved by Fine-tuning**: If language bias severely affects the product, consider DPO fine-tuning on preference data.

</details>

**25. What is the "error propagation" problem of ICL in reasoning tasks? How to mitigate it?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the "error propagation" problem of ICL in](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig38_what_is_the_error_propagation_.png)



Error propagation: In multi-step reasoning (CoT), if an intermediate step produces an error, this error will act as the input for subsequent steps, causing the error to propagate and amplify in the reasoning chain, ultimately leading to an incorrect answer. Mitigation methods:
1. **Self-Consistency**: Sample multiple times; the randomness of incorrect steps will dissipate in different paths, and take the majority vote;
2. **Verification Step Insertion**: Insert a sub-prompt like "verify whether this step is correct" after key reasoning steps;
3. **Fine-grained Step Design**: Break down reasoning steps finer to reduce the complexity of each step and lower the probability of single-step errors;
4. **Code Execution Substitution**: Replace numerical calculation steps with code execution to eliminate such errors;
5. **PRM Guidance**: Use Process Reward Model scoring to guide search and prune incorrect paths;
6. **Backtracking**: The ToT (Tree of Thoughts) architecture allows backtracking to earlier correct nodes after discovering an error.

</details>

**26. What is the "overfitting" phenomenon in ICL (oversensitivity to the example distribution)?**

<details>
<summary>💡 Show Answer</summary>

ICL overfitting: The model develops a reliance on the specific style, phrasing, and structure of the example set, and its performance drops significantly on inputs with a large distribution difference from the examples, rather than truly understanding the task intent. Manifestations:
1. Changing to a set of examples with the same format but different phrasing leads to a significant drop in accuracy;
2. When the language style of the test input differs from the examples, the model's output leans towards the example style rather than the correct answer;
3. When examples are all from a certain dataset, the generalization to other datasets is poor. Mitigation methods:
4. Diversify examples to cover different styles;
5. Test the variance across different example sets; high variance indicates overfitting;
6. Explicitly state the core rules of the task in the examples, rather than solely relying on input-output pairs;
7. Data augmentation, increasing example diversity through paraphrasing.

</details>

**27. How to evaluate the robustness of an ICL prompt to Out-of-Distribution (OOD) inputs?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to evaluate the robustness of an ICL prompt to](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig39_how_to_evaluate_the_robustness.png)



1. **Construct an OOD Test Set**: Collect or construct test samples that have large distributional differences from the examples (different domains, different language styles, uncommon input patterns);
2. **In-Distribution vs. OOD Comparison**: Evaluate accuracy on the ID and OOD test sets respectively, and calculate the "performance degradation rate";
3. **Distribution Transformation Testing**: Systematically change a certain dimension of the input (length, proportion of specialized vocabulary, language style) and observe how accuracy changes;
4. **Synonymous Paraphrasing Testing**: Express the same meaning with different phrasing to test output consistency;
5. **Noise Robustness**: Introduce spelling errors, missing words, and scrambled inputs to test the degree of performance degradation;
6. Robustness target: OOD accuracy should not be lower than 80% of ID accuracy, otherwise example diversity or format design needs improvement.

</details>

**28. ICL results have poor interpretability (not knowing why the model inferred this way). What are the methods to improve interpretability?**

<details>
<summary>💡 Show Answer</summary>

1. **CoT Forced Explanation**: Require the model to write out the reasoning process before giving the answer, making the reasoning visible;
2. **Rationale Generation**: Require the model to generate a natural language explanation for the answer ("Because... therefore...");
3. **Attribution Analysis**: Calculate attention weights to analyze which example tokens have the greatest impact on the output;
4. **Counterfactual Analysis**: Replace a single example or input word, observe output changes, and understand key influencing factors;
5. **Probing**: Insert classifiers in the model's intermediate layers to probe what internal representations ICL activated;
6. **Influence Functions**: Calculate the impact of each example on the prediction to identify the most critical examples;
7. **Contrastive Analysis**: Compare the example differences between successful and failed cases to summarize trigger patterns. Limitation: Even with CoT, Turpin et al. 2023 found that CoT is sometimes post-hoc rationalization and does not represent the true internal reasoning process.

</details>

**29. Why is ICL's performance on low-resource languages usually worse than English? What are the improvement directions?**

<details>
<summary>💡 Show Answer</summary>

Reasons for being worse than English:
1. **Imbalanced Training Data**: English accounts for the vast majority (70%+) of LLM pre-training data, with very little coverage of low-resource languages;
2. **Unfriendly Vocabulary**: BPE/SPE vocabularies are inefficient for low-resource languages (e.g., one Chinese character is one token, but a rare language might require multiple tokens to represent one word);
3. **Weaker ICL Format Understanding**: The model's understanding of prompt formats in low-resource languages is not accurate enough;
4. **Difficulty in Quality Evaluation**: Low-resource languages lack human-annotated data, making evaluation itself challenging. Improvement directions:
5. **Translation Bridging**: Translate the low-resource language input into English, use English examples for ICL, and then translate the output back;
6. **Cross-lingual ICL**: Provide English examples (which the model understands better), while the input remains the target language;
7. **Multilingual Fine-tuning**: Perform PEFT on target language data;
8. **Multilingual Embedding Optimization**: Improve tokenization schemes for low-resource languages;
9. **Synthetic Data**: Use LLMs to generate high-quality training data for low-resource languages.

</details>

**30. As context length increases (e.g., 128K tokens), does ICL performance continuously improve? What are the research findings?**

<details>
<summary>💡 Show Answer</summary>

It does not continuously improve; there is an obvious non-linear relationship. Key research findings:
1. **"Lost in the Middle" Effect** (Liu et al. 2023): LLMs pay significantly less attention to the middle part of a long context compared to the beginning and end. After the number of examples exceeds a certain threshold, the middle examples are almost ignored;
2. **Quality > Quantity**: The effect of more low-quality examples is not as good as a few high-quality examples;
3. **Actual Utilization of 128K**: Most models claim to support 128K, but at positions beyond 32K, performance has already significantly degraded (the "effective context" is much smaller than the maximum context);
4. **Example Saturation Point**: For most tasks, 8–16 high-quality examples are close to the ICL performance upper bound;
5. **Position Importance**: No matter how long the context is, examples closer to the query yield the best results.

</details>

**31. Please design an ICL prompt for "extracting sentiment and key issues from unstructured customer feedback", and explain the reasons for example selection and format design.**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of Please design an ICL prompt for "extracting sentim](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig40_please_design_an_icl_prompt_fo.png)



**Design Logic**:
Task Description: `Analyze customer feedback and extract: 1) overall sentiment (Positive/Negative/Neutral), 2) list of main issues (if any).`
Example Format (3 examples, covering three sentiment types):
\begin{lstlisting}[style=promptstyle]

	Feedback: The product quality is very good, delivery is also fast, will buy again next time.
	Analysis: {"Sentiment": "Positive", "Issues": []}
	Feedback: Waited for two weeks and the goods haven't arrived, customer service isn't replying either, so disappointed.
	Analysis: {"Sentiment": "Negative", "Issues": ["Delivery delay", "Poor customer service response"]}
	Feedback: The item basically matches the description, packaging is average.
	Analysis: {"Sentiment": "Neutral", "Issues": ["Average packaging experience"]}
\end{lstlisting}

**Design Reasons**:
1. 3 examples cover three sentiments (balanced distribution to avoid majority label bias);
2. Structured JSON output facilitates programmatic parsing;
3. Example length is moderate (50–100 words), without excessively occupying context;
4. The positive example contains an empty issues list, clearly stating the format when there are no issues.

</details>

**32. How can ICL and RAG be combined? What is each responsible for?**

<details>
<summary>💡 Show Answer</summary>

Combination strategies:
1. **RAG Provides Knowledge + ICL Provides Format**: RAG retrieves relevant knowledge documents and passes them into the context, while ICL examples demonstrate how to generate correctly formatted answers based on the retrieved content;
2. **Dynamic Example Retrieval** (RAG-style ICL): Use similarity retrieval to dynamically select the most relevant examples from an example pool, rather than fixed examples, retrieving both knowledge and examples simultaneously;
3. **Division of Labor**: RAG is responsible for "what to know" (factual knowledge), and ICL is responsible for "how to output" (format and reasoning method);
4. **Examples as Knowledge**: When the examples themselves contain knowledge (such as professional Q&A pairs), RAG selecting examples also passes on knowledge;
5. Typical prompt structure: `[System] [RAG Retrieval Results] [ICL Examples] [Current Query]`. Benefits: Reduced number of ICL examples (making room for knowledge documents), but examples are more relevant to the query (selected by RAG).

</details>

**33. How to design an automatic update mechanism for ICL examples (automatically optimizing the example pool as business data changes)?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to design an automatic update mechanism for IC](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig41_how_to_design_an_automatic_upd.png)



1. **Production Data Collection**: Collect real input-output pairs, as well as user satisfaction feedback (upvotes/downvotes) and downstream task metrics (conversion rate, accuracy, etc.), to build a candidate example corpus with annotations.

2. **Quality Filtering**: Filter candidate examples using high-quality standards. Specifically includes:
    - *Automated Scoring*: Use LLM-as-Judge to score output quality and filter out low-scoring samples;
    - *Consistency Check*: Compare outputs after multiple samplings to screen out high-variance (unstable) samples;
    - *Manual Spot Check*: Manually review high-uncertainty samples to ensure the quality of edge cases.

3. **Diversity Assurance**: Prevent the example pool from degrading into homogeneous samples.
    - Use **K-Means** or **Maximal Marginal Relevance (MMR)** to cluster candidate examples in the embedding space, and sample with quotas for each cluster;
    - Monitor the semantic coverage of the example pool (Coverage Score); when embeddings of new business scenarios fall into blank areas, trigger active supplementation;
    - Maintain a "hard example pool" dedicated to storing sample types where the model historically performed poorly.

4. **Incremental Update Strategy**:
    - *Sliding Window*: Phase out outdated examples according to a time window (e.g., business scenarios older than 30 days) and introduce new data;
    - *Importance Weighting*: Assign higher weights to recent data to prevent old distributions from dominating the example pool;
    - *Triggered Update*: When online performance metrics (like accuracy) drop below a threshold, automatically trigger the re-optimization process (similar to **Data Drift Detection**).

5. **Effect Verification Closed Loop**:
    - Conduct **A/B Testing** on the new and old example pools, taking online business metrics as the primary evaluation basis;
    - Maintain version history of the example pool to support fast rollback;
    - Record the "contribution" (task gain when selected) of each example, and regularly phase out examples with consistently negative contributions.

</details>

