# 2.3 Few-shot Prompting and In-Context Learning (ICL) Basics — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 16 questions

**1. What aspects need attention when using few-shot prompting?**

<details>
<summary>💡 Show Answer</summary>

Few-shot prompting is a powerful method for guiding large language models to complete specific tasks. Here are some suggestions for effectively using few-shot prompting:

- Although it is called "few-shot", pay attention to the quality of the examples. Choose clear, concise examples that accurately reflect the desired output format and style.
- Do not just provide isolated examples. Include context in the prompt to connect the examples to the desired task. This helps the model understand the relationship between the input and output.
- If your task involves multiple categories, make sure your examples proportionally represent each category. An uneven distribution may cause the model to be biased toward certain outputs.
- Clearly define the task you want the model to perform in the prompt. This helps the model focus on a specific goal and avoid irrelevant information.

</details>

**2. For a few-shot binary classification task, what is the difference between in-context learning and supervised learning?**

<details>
<summary>💡 Show Answer</summary>

In a few-shot binary classification task, the core differences between In-Context Learning (ICL) and Supervised Fine-tuning (SFT) are mainly reflected in the learning method and the task adaptation mechanism.

In-Context Learning relies on directly providing a small number of examples through a prompt during the inference phase, and the model performs real-time reasoning based on these examples without updating its parameters. The core of this method is that the model utilizes the general capabilities learned during the pre-training phase, combined with the examples in the context, for rapid adaptation. The advantage of ICL is that it requires no additional training and is suitable for dynamic task scenarios, but its performance highly depends on the selection of examples and the design of the prompt.

In contrast, Supervised Fine-tuning fine-tunes the parameters of a pre-trained model on a small amount of labeled data, making the model more focused on the current task. SFT adjusts the model weights through gradient descent, thereby learning the task features more directly. Although it requires additional training time, the fine-tuned model generally performs more stably than ICL, especially when the task distribution differs significantly from the pre-training data.

In summary, ICL emphasizes real-time in-context reasoning capabilities, while SFT achieves task adaptation through parameter optimization. The former is flexible but depends on the quality of examples, whereas the latter is more robust but requires computational resources. Choosing which method to use depends on the trade-offs between task requirements, data availability, and computational conditions.

</details>

**3. What is In-Context Learning (ICL)? What are its core characteristics?**

<details>
<summary>💡 Show Answer</summary>

In-Context Learning (ICL) is a reasoning paradigm for large language models: without updating any model parameters, and simply by providing a few task demonstrations in the input prompt, the model is enabled to infer the task pattern from these examples and apply it to a new query. The essence of ICL is to utilize the meta-learning capability accumulated by the model during the pre-training phase to dynamically "learn" new tasks during inference.

**Core Characteristic 1: Parameter Freezing (No Gradient Update)**
ICL does not modify the model parameters at all during the entire process. Regardless of how many examples are provided, the model weights always remain in their post-pre-training state. This is fundamentally different from the traditional machine learning "learning" paradigm—ICL is a purely Inference-time capability, not a Training-time capability. This characteristic gives ICL extremely high flexibility: the same model can instantly switch to completely different tasks across different requests.

**Core Characteristic 2: Pattern Induction from Demonstrations**
By analyzing the input-output pairs provided in the prompt, the model infers the mapping rules of the task—including task type, output format, reasoning logic, domain style, etc. This induction process occurs during the model's Forward Pass, using the attention mechanism to establish associations between context tokens, rather than updating parameters via backpropagation.

**Core Characteristic 3: Emergent Generalization**
ICL is an ability that emerges as the model size increases. Studies have shown that the ICL capability does not exist linearly in small models, but suddenly and significantly appears when the model parameter count exceeds a certain threshold (generally considered to be around the 10B parameter scale). This indicates that ICL is not merely simple pattern matching, but relies on the deep semantic understanding and meta-learning capabilities accumulated during large-scale pre-training.

**Core Characteristic 4: Sensitivity to Demonstration Design**
Compared to the robustness of supervised learning to training data, ICL is highly sensitive to the design of examples. The order, quantity, format, label correctness, and even the diversity among examples can all have a significant impact on the effectiveness of ICL. This sensitivity is both a source of ICL's flexibility and its main engineering challenge.

**Core Characteristic 5: No Task-specific Dataset Required**
ICL requires only a small number (typically 1--32) of hand-crafted or randomly sampled examples, without the need to build large-scale labeled datasets or design specialized training pipelines. This makes ICL particularly suitable for emerging tasks where data is scarce or for rapid prototyping scenarios.

</details>

**4. What is the essential difference between ICL and Fine-tuning?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the essential difference between ICL and F](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig12_what_is_the_essential_differen.png)



ICL and Fine-tuning represent two fundamentally different paradigms for utilizing pre-trained large models. There are essential differences between the two across multiple dimensions, including where knowledge is stored, computation methods, data requirements, and applicable scenarios.

**Fundamental Difference in Knowledge Updating Mechanisms:**
Fine-tuning updates model parameters through Backpropagation and Gradient Descent, "writing" task-specific knowledge into the model weights and forming permanent parameter modifications. ICL, on the other hand, does not modify any parameters; task information exists only in the context window of the current request and completely disappears after the session ends, leaving no trace. Essentially, Fine-tuning changes "what the model is," whereas ICL only changes "what the model does in the current task."

**Magnitude Difference in Data Requirements:**
Fine-tuning typically requires hundreds to tens of thousands of high-quality labeled samples (in the SFT phase), as well as specialized training pipelines (data cleaning, hyperparameter tuning, training monitoring). ICL usually requires only 1--32 examples, or even zero (Zero-shot). This difference is critical for data-scarce scenarios (new languages, new domains, new task types).

**Difference in Computational Resource Requirements:**
Fine-tuning requires a complete training infrastructure: multi-GPU clusters, devices with large VRAM, longer training times (from hours to days), and the memory overhead of gradient computation and backpropagation. ICL only requires standard inference resources, with the same computational load as a normal inference request and no need for extra infrastructure.

**Difference in Flexibility of Task Switching:**
A fine-tuned model is "locked" onto a specific task; switching to a different task requires retraining or maintaining multiple model copies, posing a risk of Catastrophic Forgetting. ICL allows the same model to switch tasks instantly by modifying the Prompt; one model instance can simultaneously serve countless different tasks without the need for multi-model management.

**Difference in Performance Ceilings:**
In scenarios with abundant data, Fine-tuning can usually achieve a higher task performance ceiling because it allows the model to undergo deep specialization for the target task. ICL's performance is limited by the number of examples that can be provided in the Prompt and the context window size, and it often falls short of Fine-tuning on highly specialized tasks or tasks requiring deep domain knowledge.

**Difference in Interpretability and Controllability:**
ICL's behavior stems from explicitly visible examples in the context and is relatively transparent—engineers can intuitively analyze "why the model did this." Fine-tuning scatters knowledge across billions of parameters, making the source of the model's behavior hard to trace and rendering debugging and explanation more difficult.

**Complementarity in Applicable Scenarios:**
The two are not mutually exclusive, but rather complementary. Fine-tuning is suitable for: fixed tasks, abundant data, extremely high performance requirements, and scenarios requiring deep domain specialization. ICL is suitable for: variable tasks, scarce data, the need for rapid iteration, and scenarios demanding high flexibility. In practice, the optimal solution is often to combine the two: use Fine-tuning to build strong foundational capabilities, and then use ICL for task specialization during inference.

</details>

**5. What are Zero-shot, One-shot, and Few-shot? What are the differences between them?**

<details>
<summary>💡 Show Answer</summary>

Zero-shot, One-shot, and Few-shot are three standard paradigms for classifying ICL based on the number of task demonstrations provided in the Prompt. They differ in information density, applicable scenarios, and performance characteristics.

**Zero-shot Learning:**
Zero-shot refers to not providing any task examples in the Prompt at all. By only providing the task description and query, the model is asked to generate the answer directly. The model relies entirely on the general knowledge and task understanding accumulated during the pre-training phase to complete the reasoning.

Typical Prompt Structure:
\begin{lstlisting}[style=promptstyle]

	Translate the following English sentence into Chinese:
	"The quick brown fox jumps over the lazy dog."
\end{lstlisting}

The core advantage of Zero-shot is that the Prompt is concise and no examples need to be prepared. It is suitable for general tasks (translation, summarization, Q&A) and scenarios where the model already has sufficient pre-training coverage of the task type. Its limitation is a weaker ability to handle tasks with special formatting, vague definitions, or rare tasks, because the model must rely entirely on its understanding of the task description and lacks format anchors.

**One-shot Learning:**
One-shot refers to providing exactly one complete input-output example pair in the Prompt before proposing the new query. A carefully chosen example can clearly anchor the task format, output style, and reasoning pattern, significantly reducing the model's ambiguity space.

Typical Prompt Structure:
\begin{lstlisting}[style=promptstyle]

	Example:
	Input: Apple -> Output: Fruit
	Please classify:
	Input: Rose -> Output:
\end{lstlisting}

One-shot strikes a balance between Zero-shot and Few-shot: providing critical format guidance to the model at a minimal example cost (a single example). It is suitable for scenarios where token budgets are limited but format guidance is needed, and is also commonly used for tasks with highly specific output formats (such as specific JSON structures or naming conventions).

**Few-shot Learning:**
Few-shot refers to providing more than 2 (usually 3--32) example pairs in the Prompt, allowing the model to induce the task pattern from multiple examples. More examples provide richer pattern signals, helping the model understand task boundaries more precisely, handle edge cases, and learn domain-specific reasoning logic.

The effectiveness of Few-shot is generally superior to Zero-shot and One-shot, but the marginal return diminishes as the number of examples increases (usually the improvement is no longer significant after exceeding 8--16 examples). Furthermore, an increased number of examples will consume more token budget and may introduce the Lost in the Middle problem.

**Core Comparison of the Three:**

\begin{center}


| **Paradigm** | **Number of Examples** | **Token Cost** | **Most Suitable Scenarios** |
| --- | --- | --- | --- |
| Zero-shot | 0 | Lowest | General tasks, simple formatting |
| One-shot | 1 | Low | Specific formatting, limited tokens |
| Few-shot | 2--32+ | Medium--High | Complex tasks, high precision requirements |


\end{center}
**Selection Advice:** Start testing with Zero-shot; if the output format does not meet expectations, upgrade to One-shot to provide format examples; if the task logic is complex or accuracy is still insufficient, upgrade to Few-shot and gradually increase the number of examples until the marginal benefit disappears.

</details>

**6. What is the role of "demonstration" in ICL?**

<details>
<summary>💡 Show Answer</summary>

In ICL, a demonstration (example) is a complete input-output pair provided in the Prompt, serving as the core source of information for the model to infer the task pattern. Deeply understanding the multi-level roles of demonstration is crucial for designing efficient ICL Prompts.

**Role 1: Format Anchoring**
The most direct role of demonstration is to provide a specific reference for the output format of the model. No matter how the output format is explained in text in the task description, a concrete example is often more effective than hundreds of words of format description. By observing the structure of the output in the example (JSON structure, list format, punctuation usage, number format, etc.), the model can precisely reproduce the same formatting specifications.

**Role 2: Task Type Identification**
Through the semantic relationship between the input and output in the demonstration, the model can infer which type of task it is (classification, translation, summarization, sentiment analysis, entity extraction, etc.), even if there is no explicit task name in the Prompt. Research shows that in some cases, the correct input-output format can even make the model understand the task intent without any task description.

**Role 3: Label Space Definition**
For classification tasks, demonstration conveys the output space boundaries of the task to the model by showing the actually used labels (such as "positive", "negative", "neutral", or "A", "B", "C", "D"). Even if the labels are random symbols, as long as they are used consistently in the examples, the model can follow the same label system during prediction.

**Role 4: Reasoning Style Transfer**
In Chain-of-Thought (CoT) tasks, the reasoning process shown in the demonstration will guide the model to adopt a similar reasoning style. For example, if the example shows a reasoning pattern of "first list the premises, then derive step-by-step, and finally give a conclusion," the model will tend to reproduce a similar reasoning structure when answering new queries.

**Role 5: Implicit Domain Knowledge Transfer**
Professional terminology, industry conventions, and domain-specific expressions used in the demonstration implicitly guide the model to invoke knowledge paths related to that domain, making the output closer to domain professional standards, even without explicit domain instructions in the Prompt.

**Role 6: Difficulty Calibration**
Through the complexity of the demonstration, the model can infer the expected depth of response for the current task. Simple examples guide the model to give concise and direct answers; examples containing complex reasoning chains guide the model to conduct in-depth analysis.

**Role 7: Capability Activation**
Certain model capabilities are in a "dormant" state without appropriate example guidance, and providing relevant demonstrations can activate these capabilities. For example, providing examples containing code generation and test cases can activate the model's Test-Driven Development (TDD) capability, which is not actively exhibited in ordinary conversations.

</details>

**7. Why do large models have ICL capabilities while small models usually do not?**

<details>
<summary>💡 Show Answer</summary>

The relationship between ICL capability and model size is one of the core discoveries in large language model research, revealing the important phenomenon of "Emergent Abilities". Understanding this relationship requires an in-depth analysis from three levels: pre-training, model architecture, and meta-learning.

**The Nature of Emergent Abilities:**
ICL capability does not grow linearly with model size, but suddenly appears after the number of model parameters exceeds a certain threshold. This phenomenon is called "Emergent Abilities". The ICL capability of GPT-3 (175B parameters) is much stronger than that of GPT-2 (1.5B parameters), while the latter has almost no meaningful ICL capability. This indicates that ICL is not just a "larger language model", but a qualitative change that requires a specific capability threshold to activate.

**The Decisive Role of Pre-training Data Volume and Diversity:**
The fundamental reason why large models can perform ICL is the scale and diversity of their pre-training corpus. Large models have been exposed to trillions of tokens of text during training, covering almost all task types written by humans—translation, summarization, question answering, classification, reasoning, code generation, etc. These tasks often naturally appear in the pre-training corpus in the form of "example-answer" (such as textbooks, tutorials, Q&A websites, code comments), allowing the model to implicitly learn the Meta-learning Ability of "inferring task rules from examples". Small models cannot fully acquire this meta-ability due to limited pre-training data volume.

**Model Capacity and Knowledge Storage:**
The effective operation of ICL requires the model to store a large amount of prior knowledge in its parameters, so that related knowledge can be quickly activated when seeing examples. Large models have sufficient parameter capacity to store rich cross-domain knowledge. When the pattern signal provided by the example triggers related knowledge, the model can combine existing knowledge with new patterns to generate accurate outputs. The parameter capacity of small models is limited, and the amount of stored knowledge is insufficient to support this rapid knowledge activation and combination.

**The Expressiveness of Attention Mechanism:**
ICL relies on the attention mechanism to establish precise semantic associations between demonstration tokens and query tokens. Large models have more Attention Heads, more Layers, and a larger Hidden Dimension, enabling them to establish more precise and finer-grained token associations in a higher-dimensional semantic space. Small models have insufficient attention capacity and struggle to accurately capture task patterns in complex demonstrations.

**Implicit Gradient Descent Hypothesis:**
Researchers have proposed an influential theoretical hypothesis: the working principle of ICL is equivalent to gradient descent in some sense—the forward computation process of a Transformer can simulate the effect of executing a few steps of gradient updates on the examples, "temporarily encoding" the task information in the examples into the attention activations. This process requires the model to have sufficient depth and width to achieve, and small models cannot complete this "implicit optimization".

**Prerequisite for Instruction Following Capability:**
ICL also relies on the model's ability to understand and follow natural language instructions. This capability also increases with model size and is further strengthened through alignment training such as RLHF. Even if small models receive the same Prompt formally, they often cannot correctly parse the structural signals in the examples, leading to ICL failure.

</details>

**8. What is the role of "label" in ICL? What does it do in the demonstration?**

<details>
<summary>💡 Show Answer</summary>

In ICL, a "label" specifically refers to the output part of each example in the demonstration—that is, the correct answer or expected output corresponding to a given input. Deeply understanding the role of labels is crucial for explaining the working mechanism of ICL and designing effective examples.

**Core Functions of Labels:**

**1. Output Space Definition:**
The most direct role of a label is to inform the model of what constitutes a valid output format for the current task. In classification tasks, the label set (e.g., \{"positive", "negative", "neutral"\}) defines which candidate set the model should choose from during inference. This limitation of the output space is particularly important for tasks that require precise category labels, preventing the model from generating free text with non-compliant formatting.

**2. Task Mapping:**
The label and corresponding input together form the functional mapping relationship of the task $f: input \rightarrow output$. By observing the input-label pairs in multiple examples, the model induces this mapping rule and applies it to new query inputs.

**3. Format Anchoring:**
The writing format of the label (capitalization, punctuation, line breaks, whether there are prefix words, etc.) will be precisely copied by the model into the output. For example, if the label in the example is written as "Category: Positive", the model output will also be presented in the same format, rather than just outputting "Positive".

**Surprising Findings About Label Correctness:**
Researchers (Min et al., 2022) discovered a counter-intuitive conclusion through systematic experiments: in Few-shot ICL, using random wrong labels (i.e., deliberately replacing the labels of examples with random incorrect answers) has a much smaller impact on model performance than expected—on some tasks, the performance of random labels only dropped by 10%--15% compared to correct labels.

This finding suggests that the true value of labels does not entirely lie in the correctness of their content, but more in:
- **Format Signal**: The existence of the label tells the model "there should be an output here, and the output has a specific format".
- **Input Distribution Perception**: The input part of the example helps the model perceive the task-relevant input data distribution.
- **Task Type Identification**: The combination structure of input-label pairs helps the model identify the task type, which is more important than the correctness of the label itself.

**Practical Advice for Label Design:**
Although random labels do not completely break ICL, using correct labels can still bring performance improvements, especially in the following situations: semantically complex classification tasks (the semantic content of the label itself provides important clues); tasks requiring precise values or generative outputs (the concept of "random labels" is not applicable); and production scenarios with extremely high accuracy requirements. Therefore, in practice, correct labels should always be used, while understanding that format consistency is more critical than the label itself.

</details>

**9. How does the number of Few-shot examples affect the results? Is more always better?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How does the number of Few-shot examples affect th](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig13_how_does_the_number_of_few_sho.png)



The number of examples (shots) is one of the most direct hyperparameters in Few-shot ICL design, but its relationship with performance is not simply monotonically increasing, but exhibits complex non-linear characteristics.

**Benefits of Increasing the Number of Examples:**

Increasing the number of examples improves ICL performance from the following dimensions:
- **Pattern Signal Enhancement**: More examples provide more sufficient inductive evidence, making the model's inference of task rules more accurate, especially for tasks with complex rules or many edge cases.
- **Covering More Input Types**: In tasks with high input diversity, more examples can cover a wider range of input types (different language styles, different difficulty levels, different topic domains), enhancing the model's generalization capability to out-of-distribution inputs.
- **Reducing Format Ambiguity**: For complex output formats (such as multi-field JSON, nested lists), multiple examples can reinforce format specifications from multiple angles, reducing the probability of the model deviating from the format.
- **Sufficient Display of Label Space**: In multi-class classification tasks, having each category appear at least once in the examples helps the model correctly understand the criteria for each category.

**Diminishing Marginal Returns of Increasing Examples:**

Experimental studies (Brown et al., GPT-3 paper) systematically measured the performance changes under different numbers of shots and found that: the improvement from 0-shot to 1-shot is usually the largest; there is still a significant improvement from 1-shot to 4-shot; the improvement from 4-shot to 8-shot starts to decrease; after exceeding 16-shot, the performance improvement for most tasks tends to flatten or even slightly decrease. This phenomenon of diminishing marginal returns means that blindly increasing the number of examples is not an efficient strategy to improve ICL effects.

**Negative Effects Introduced by Too Many Examples:**
- **Token Budget Consumption**: Each example occupies the token space of the context window. Too many examples will squeeze the space left for retrieved content, dialogue history, or user queries, constituting a direct trade-off when the token budget is limited.
- **Increased Risk of "Lost in the Middle"**: As the number of examples increases, earlier examples are pushed to the middle of the context, weakening their format and pattern guidance roles, which may lead to a decline in overall ICL performance.
- **Introduction of Contradictions Between Examples**: When the number of examples increases, improper example selection may introduce contradictory pattern signals (e.g., two examples giving different formatted outputs for the same type of input), confusing the model's pattern induction.
- **Context Pollution Risk**: A large number of low-quality or low-relevance examples is more likely to introduce noise than a small number of high-quality examples, leading to a performance drop.

**Method for Determining the Optimal Number of Examples:**

Theoretically, the optimal number of examples varies by task type, model capability, and token budget. Practical advice is as follows: use 4--8 examples as a starting point; test the performance of 1, 2, 4, 8, 16 examples on a validation set and plot a learning curve; find the "inflection point" where performance improvement tends to flatten, and use that number as the production configuration; when the token budget is limited, prioritize example quality (strong representativeness, clear format) over quantity.

</details>

**10. What is the "format" of a Prompt? How much does it affect ICL?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the "format" of a Prompt? How much does it](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig14_what_is_the_format_of_a_prompt.png)



The format of a Prompt refers to the overall design of how examples are structurally presented when constructing an ICL Prompt, including the arrangement structure of examples, separation methods, writing style of input and output markers, use of whitespace characters and line breaks, and the visual and semantic organizational relationships between various elements. Format is a design dimension that is often underestimated in ICL but actually has an extremely significant impact.

**Main Components of Format:**
- **Separator Design**: How to separate examples from each other and inputs from outputs (blank lines, triple hyphens, XML tags, specific keywords like "Input:"/"Output:").
- **Role Prefix**: Whether to use role markers (such as "User:"/"Assistant:", "Q:"/"A:") to explicitly identify the dialogue structure.
- **Input/Output Markers**: Whether to use explicit keys (such as "Text:", "Label:", "Answer:") to mark the boundaries of inputs and outputs.
- **Arrangement Order**: Are examples arranged by increasing difficulty, randomly, or grouped by category.
- **Indentation and Line Breaks**: Whether to use unified indentation and line break specifications to assist the model's semantic parsing with visual structure.

**Dimensions of Format's Impact on ICL Effects:**

**1. The Criticality of Format Consistency:** Whether the format is highly consistent across examples is the most important formatting factor affecting ICL performance. Research shows that inconsistent formats (e.g., some examples using an "Answer:" prefix, while others output the answer directly) will cause the model to have ambiguity about the output structure, and the error rate will rise significantly. Format consistency is sometimes more important than content correctness.

**2. Compatibility between Format and Model:** Due to differences in pre-training data and alignment training, different models have different "preferences" for specific formats. For example, instruction-tuned models usually respond better to the "role-content" format (system/user/assistant structure), while Base LMs perform more stably on pure text formats. Choosing a Prompt format that is compatible with the format used during the target model's training can significantly improve ICL performance.

**3. Matching Format Complexity with Model Capability:** For weaker models, overly complex formats (such as deeply nested XML structures) may cause parsing burdens, actually degrading performance. For powerful models, explicit structural formats can usually improve performance because they reduce ambiguity space.

**4. Format's Influence on Output Format:** Models tend to precisely copy the formatting features (capitalization, punctuation, line breaks, prefix words) of the output part in the examples, so Prompt format design is a direct means of output format control. If a specific output format is required (such as a pure label without any prefixes), the output part in the examples must strictly follow that format.

**Practical Advice:** Before formal use, conduct systematic testing (A/B testing) on different format designs to quantify the impact of different formats on the accuracy of the target task. Once the optimal format is determined, it should be kept strictly consistent across all examples, and the same format should be applied to the input part of new queries to ensure format alignment between examples and queries.

</details>

**11. Does ICL require updating model parameters?**

<details>
<summary>💡 Show Answer</summary>

By definition, ICL explicitly does not require updating any model parameters, which is one of its most fundamental characteristics and its core difference from Fine-tuning. But behind this seemingly simple answer, there are rich mechanistic details worth understanding deeply.

**ICL's Parameter Freezing Mechanism:**
In the ICL process, the entire computational flow is pure Forward Inference: all tokens in the Prompt (including examples and queries) are encoded into input vectors, pass through the forward computation of the various Transformer layers of the model, and finally generate the probability distribution of the output tokens. There is no loss function calculation, no backpropagation, and no parameter updating by the optimizer throughout the process. The model weights are exactly the same before and after inference.

**Where Does "Learning" Happen?**
Although parameters are not updated, some form of "learning" does occur in ICL—this learning happens in the Attention Activations, not in the model weights. The information from the examples is encoded into the Key-Value pairs of the various layers through the attention mechanism, and is dynamically retrieved and utilized when processing the query tokens. This "Activation-level Learning" is transient and intra-request, and will not persist across requests.

**Theoretical Equivalence Between ICL and Gradient Descent:**
Research by Aky\"urek et al. (2022) and Von Oswald et al. (2023) mathematically proved that the self-attention mechanism of the Transformer can implement operations similar to one step of gradient descent—that is, the forward computation of ICL is equivalent to executing an implicit parameter optimization on the examples, but the result of this optimization is stored in the activation states rather than being written to the model weights. This theory explains why ICL, despite not updating parameters, can exhibit effects similar to fine-tuning with a small number of gradient steps.

**Practical Implications:**
Parameters not updating means that ICL does not have any persistent learning effects: what is "taught" to the model through examples in the current request completely disappears in the next request, and examples need to be provided in the Prompt again. This is both an advantage of ICL's flexibility (the same model can instantly switch to different tasks through different examples) and its limitation (unable to accumulate cross-request task knowledge, requiring token costs for providing examples every time).

</details>

**12. What are the main advantages of ICL? In which scenarios is it suitable?**

<details>
<summary>💡 Show Answer</summary>

The advantages of ICL are reflected in multiple dimensions, making it the most flexible and lowest-barrier way to leverage capabilities in large language model applications.

**Main Advantages:**
- **Zero Training Cost**: ICL completely skips machine learning engineering steps such as data collection, cleaning, annotation, training monitoring, and hyperparameter tuning, achieving task adaptation directly during inference. This is a huge efficiency advantage for small teams or quick validation phases.
- **Extremely High Task Switching Flexibility**: The same model instance can switch from sentiment analysis to machine translation, and then to code generation within milliseconds by modifying the Prompt, without model switching or reloading. A single model serves infinite task types.
- **Low Data Requirements**: Task adaptation can be achieved with only 1--32 examples, suitable for emerging tasks, long-tail tasks, or highly personalized task requirements where annotated data is scarce.
- **Strong Interpretability**: The behavior of ICL is directly determined by the visible examples in the Prompt. When output problems occur, engineers can directly locate the cause and fix it by inspecting the examples, which is much more transparent than the black-box debugging of Fine-tuning.
- **Rapid Iteration**: Modifying ICL examples only requires editing text without retraining, compressing the iteration cycle from days (Fine-tuning) to minutes, making it very suitable for agile development and A/B testing.
- **No Risk of Catastrophic Forgetting**: Since parameters are not updated, ICL does not have the problem of forgetting the ability to do old tasks after learning new ones. The model's general capabilities remain intact at all times.

**Most Suitable Usage Scenarios:**
- **Rapid Prototype Validation**: Quickly validate task feasibility and baseline performance with ICL before deciding whether it's worth investing resources into Fine-tuning.
- **Data-Scarce Tasks**: For tasks with extremely sparse annotated data (fewer than 100 entries), ICL is often the only feasible adaptation method.
- **Multi-Task Concurrent Scenarios**: When the same system needs to simultaneously handle various task types (such as a unified assistant supporting Q&A, translation, and summarization at the same time), ICL allows switching tasks through dynamic Prompts without maintaining multiple Fine-tuned models.
- **Scenarios with Frequently Changing Task Requirements**: When business rules, output formats, or task definitions are updated frequently, ICL can keep up with the changes just by modifying examples, whereas Fine-tuning requires retraining for every change.
- **Format Conversion and Normalization Tasks**: For tasks with clear input-output formats and strong regularity (such as data format conversion, standardizing addresses, generating code comments), ICL can achieve high accuracy with a few examples.
- **Low-Latency Insensitive, Cost-Controllable Inference Scenarios**: When the cost and latency per inference are within an acceptable range, ICL is the optimal solution for general tasks.

</details>

**13. What are the main limitations of ICL?**

<details>
<summary>💡 Show Answer</summary>

Although ICL has significant advantages, its limitations are equally obvious. Understanding these limitations is critical for making correct technical selection decisions.

**1. Performance Ceiling is Lower than Fine-tuning:**
In scenarios with sufficient data, the task performance of ICL is usually lower than that of specifically Fine-tuned models. Fine-tuning allows the model to undergo deep parameter optimization for the target task, while ICL can only provide guidance within a limited context space and cannot achieve the same depth of task specialization. For production tasks requiring high accuracy (medical diagnostic assistance, legal document review, financial risk classification), the performance gap of ICL might be unacceptable.

**2. Hard Limit of Context Window:**
Each example in ICL occupies tokens. When a task requires a large number of examples (such as covering complex rules or multi-class tasks), the token budget available for queries and other content is heavily consumed. For complex tasks requiring over 100 examples to achieve ideal performance, ICL is practically unfeasible, while Fine-tuning can leverage thousands of samples for training without being constrained by the context window.

**3. Highly Sensitive to Example Design:**
The performance of ICL is highly sensitive to the selection, order, and format of examples. Different example sets can lead to performance differences exceeding 20% (Zhao et al., 2021). This means that the effect of ICL largely depends on the engineer's Prompt design skills, and it is difficult to transfer design experience across different tasks, leading to a high reliance on "manual craftsmanship".

**4. Inference Cost Increases Linearly with Example Quantity:**
For every additional example, the number of tokens processed per inference increases, along with API call costs and inference latency. In high-concurrency production environments, a Prompt containing 16 examples may cost several times more to infer than a Zero-shot Prompt, resulting in significant long-term operational costs.

**5. No Persistent Learning Effect:**
The "learning" effect of ICL is only valid during a single request and cannot accumulate knowledge across requests. For scenarios requiring the model to continuously accumulate knowledge or style preferences in a certain domain, ICL needs to provide examples anew every time, unable to achieve progressive learning, which is inefficient.

**6. Almost Ineffective on Small Models:**
ICL is a capability unique to large models (usually $\geq$10B parameters) and has almost no effect on small language models ($\leq$1B parameters). In scenarios requiring the deployment of lightweight models (edge devices, low-power scenarios), ICL is unavailable, and Fine-tuning must be relied upon.

**7. Lower Reproducibility and Stability:**
Even with fixed examples and queries, the output of ICL between different runs may vary due to the model's sampling strategy (Temperature > 0). Furthermore, the same set of examples may perform significantly differently on different versions of a model, requiring a re-evaluation of the effectiveness of all ICL Prompts during system upgrades.

</details>

**14. What is the "diversity" of demonstration? Why is it important?**

<details>
<summary>💡 Show Answer</summary>

The diversity of a demonstration in a Few-shot Prompt refers to the selected example set presenting sufficient coverage and differentiation across multiple dimensions, rather than being repeatedly sampled from a single type or a single distribution. Diversity is one of the most important design principles in example selection strategies, having a direct and significant impact on the generalization ability of ICL.

**Multiple Dimensions of Diversity:**
- **Label Diversity**: In classification tasks, all categories should appear in the examples, avoiding having too many examples of a certain category in the example set (class imbalance), which would cause the model to produce output bias. Ideally, the number of examples for each category should be equal.
- **Length Diversity**: The input lengths in the examples should cover the typical length range of the target task, avoiding having all examples as short sentences or long paragraphs, enabling the model to learn to operate stably under different input sizes.
- **Topic Diversity**: Examples should cover different topics, domains, and content types that the target task might involve, preventing the model from binding the task to specific topics (e.g., if all examples are tech-related, it leads to insufficient generalization when handling sports topics).
- **Difficulty Diversity**: Simultaneously including simple (typical cases) and difficult (edge cases, ambiguous cases, rare patterns) examples helps the model establish a clear understanding of the task boundaries.
- **Style Diversity**: In generation tasks, examples should cover different expression styles, syntactic structures, and vocabulary habits, preventing the model from learning only a single style.

**Reasons for the Importance of Diversity:**

**1. Preventing Distribution Shift:** If the example set represents a specific subset of the input distribution, the model will implicitly bind the task rules to the characteristics of that subset, resulting in a sharp decline in performance when handling out-of-distribution inputs. Diversity ensures that the example set has broader coverage over the target input distribution, enhancing the model's robustness to various inputs.

**2. Suppressing Label Bias:** When examples of a certain category are in the majority, the model will develop statistical class bias, tending to predict the majority class in situations of uncertainty. A class-balanced example set can effectively suppress this bias and maintain the model's prediction calibration across categories.

**3. Revealing the True Complexity of the Task:** Simple examples of a single type may make the model mistakenly believe that the task is always simple and straightforward, whereas diverse examples (including hard cases and edge cases) can show the model the true complexity of the task, forcing it to learn more general reasoning rules rather than just memorizing simple patterns.

**Methods to Achieve Diversity:** When selecting examples, one can use Clustering-based Sampling—first performing semantic clustering on candidate examples, then extracting representative examples from each cluster to ensure coverage of different semantic subspaces; or using the Maximum Marginal Relevance (MMR) algorithm to maximize the diversity between examples while ensuring they are relevant to the query.

</details>

**15. How to design a Prompt to make the model confirm its understanding of the question before answering?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to design a Prompt to make the model confirm i](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig15_how_to_design_a_prompt_to_make.png)



When designing a Prompt, you can use the following methods to have the model confirm its understanding of the question before answering.

#### Strategy 1: Explicitly Requesting Restatement Before Answering

The most direct way is to explicitly instruct the model in the Prompt to restate the question in its own words first, and then give the answer:

\begin{lstlisting}[style=promptstyle]
	Before answering, please restate your understanding of the question in one sentence,
	and then provide the formal answer. Format as follows:
	[My Understanding] <restate question>
	[Answer] <your answer>
\end{lstlisting}

This method forces the model to output intermediate steps, enabling the user to discover potential misunderstandings and promptly correct them before the model gives a lengthy response.

#### Strategy 2: Requesting Identification of Ambiguity and Proactive Clarification

For highly ambiguous tasks, the model can be instructed to ask questions proactively when it detects uncertainty:

\begin{lstlisting}[style=promptstyle]

	You are a rigorous question-answering assistant. When a question is ambiguous or lacks critical information, please first list the points of ambiguity you identified and ask the user; if the question is clear, answer it directly.
\end{lstlisting}

This strategy is applicable to scenarios with a high incidence of ambiguity, such as customer service, legal consultation, and technical support. The model no longer guesses the user's intent but actively confirms it.

#### Strategy 3: Step-by-Step Structured Confirmation

Require the model to explicitly break down "Understanding-Assumption-Answering" into three steps:

\begin{lstlisting}[style=promptstyle]

	Please handle the user's question according to the following steps:
	Step 1 [Understanding] Summarize the core intent of the question in one sentence.
	Step 2 [Assumption] List the critical assumptions made when answering (if any).
	Step 3 [Answer] Provide a formal answer based on the above.
\end{lstlisting}

Making assumptions transparent allows the user to verify the model's reasoning starting point, making it more interpretable than mere repetition, and is suitable for complex reasoning or specialized domain tasks.

#### Strategy 4: Few-shot Examples to Guide Confirmation Behavior

By providing examples that include confirmation behavior, implicitly guide the model to imitate this style:

\begin{lstlisting}[style=promptstyle]

	Example:
	User: How is Apple?
	Assistant: [Understanding] Do you mean Apple Inc. or the fruit apple?
	Which aspect would you like to know about?

	Now please process:
	User: {user_input}
\end{lstlisting}

The Few-shot approach does not require modifying the underlying logic of the model's behavior, making the output style more natural, and is suitable for scenarios in dialogue systems with high consistency requirements.

#### Strategy 5: Persistent Settings at the System Prompt Level

Embed the confirmation mechanism in the `system` field of the API call to make it continuously effective for all user inputs:

\begin{lstlisting}[style=promptstyle]

	You are a helpful assistant. Before answering any question:
	1. Restate your understanding in one concise sentence.
	2. If critical information is ambiguous, ask exactly ONE
	clarifying question before proceeding.
	3. Only answer after confirming your understanding.
\end{lstlisting}

#### Considerations

Excessively demanding confirmation will lead to interaction redundancy and reduce user experience. It is recommended to dynamically enable it based on task complexity, for example, triggering the clarification process only when the user's input has a small number of words or contains ambiguous pronoun references. For clear and simple questions, the model should answer directly.

</details>

**16. What Kinds of Tasks Are Most Suitable for Prompt Engineering (Including ICL) Rather Than Fine-tuning?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What Kinds of Tasks Are Most Suitable for Prompt E](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig16_what_kinds_of_tasks_are_most_s.png)



Prompt Engineering (including In-Context Learning, ICL) and Fine-tuning are essentially two different model adaptation paradigms: the former guides the model's behavior by modifying the input text without changing the model weights; the latter solidifies specific knowledge or styles into model parameters through gradient updates. The following analyzes the task types most suitable for Prompt Engineering from multiple dimensions.

#### 1. Tasks with Scarce Annotated Data or Extremely High Acquisition Costs

Fine-tuning has high requirements for data volume, typically requiring hundreds to tens of thousands of high-quality annotated samples to achieve stable gains. Prompt Engineering is a more practical choice when any of the following conditions are met: annotated data is less than a few hundred items; annotation requires scarce domain experts (e.g., medical image reports, contract review), resulting in high costs; the task is a one-off or low-frequency requirement, not worth investing training resources. ICL requires only zero to a few dozen examples to guide the model to complete high-quality reasoning in context, demonstrating a significant advantage in data-scarce scenarios.

#### 2. Tasks Requiring Rapid Iteration or Frequently Changing Requirements

Requirements in the early stages of a product are often unstable, with task definitions, output formats, or evaluation criteria potentially changing weekly. The training-evaluation-deployment cycle of Fine-tuning usually takes days or even weeks, whereas modifications to Prompts can take effect in seconds and be validated immediately. Furthermore, in multi-tenant scenarios, different clients may require different behavioral styles, which can be achieved through Prompt parameterization without the need to maintain independent fine-tuned models for each client.

#### 3. Tasks That Are Essentially Compositional Reasoning Rather Than Pattern Memorization

Fine-tuning excels at mapping specific input patterns to specific outputs, making it suitable for memorization tasks, such as translating industry-specific jargon or fixed classification label systems. In contrast, Prompt Engineering has more advantages in the following reasoning tasks: mathematical proofs and logical inference (guiding the model to unfold the chain of thought step-by-step through Chain-of-Thought Prompts); multi-step instruction execution (e.g., "First summarize the original text, then translate it into English, and finally extract three keywords"); creative writing or analysis tasks that require flexibly invoking multi-domain knowledge. The core capabilities for these tasks already exist in the pre-trained model, and the role of the Prompt is to activate and guide, not to relearn.

#### 4. Output Format Control and Structured Generation Tasks

When the main challenge of a task is "outputting in a specific format" rather than "acquiring new knowledge," Prompt Engineering is more efficient. For example, requiring the model to output a strict JSON structure, follow a specific report template, control tone and length, or use specified heading levels in Markdown. These requirements can be achieved by providing format examples or explicit constraints in the Prompt without needing to adjust the model weights.

#### 5. Tasks Requiring Full Utilization of Pre-trained General Knowledge

Fine-tuning carries the risk of Catastrophic Forgetting: after over-training on a specific domain, the model may lose its general reasoning capabilities on other tasks. For open-domain question answering, multi-language mixed processing, and tasks requiring broad common-sense reasoning, retaining the knowledge breadth of the base model is crucial. In this case, using Prompt Engineering can adapt to specific requirements without losing general capabilities. A practical judgment criterion is: if the task requires the model to "use existing knowledge more smartly," choose Prompt Engineering; if it requires the model to "learn an entirely new private knowledge system or proprietary terminology," then consider Fine-tuning.

#### 6. Scenarios with Limited Budgets or Simple Deployment Architectures

Fine-tuning requires dedicated GPU resources for training, and the fine-tuned model also needs independent deployment or incurs higher hosting fees, along with the need to establish version management and rollback mechanisms. Prompt Engineering directly calls the API of a general foundation model, resulting in a simple cost structure without the need to maintain additional model instances, giving it a clear advantage in situations with limited budgets or teams lacking MLOps capabilities.

#### Comprehensive Decision Suggestions

In practice, it is recommended to prioritize establishing a baseline with Prompt Engineering (including ICL). Fine-tuning should only be introduced when the space for Prompt optimization has been largely exhausted and sufficient data and computing resources are available. The two can also be combined: first enhancing the model's base capabilities in a specific domain through Fine-tuning, and then fine-tuning the output format and behavioral style through Prompts, which often achieves better results than using either method alone.

</details>

