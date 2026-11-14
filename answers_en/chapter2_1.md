# 2.1 High-Quality Prompt Design Fundamentals — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 16 questions

**1. How to write high-quality prompts?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to write high-quality prompts?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig01_how_to_write_high_quality_prom.png)



High-quality prompts are the core of efficient collaboration with large language models. Their essence is to transform the vague intent of humans into structured instructions that the model can accurately execute. The following systematically expounds on the methodology of writing high-quality prompts from multiple dimensions.

#### 1. Provide specific and clear instructions

Vague instructions are the primary cause of low-quality outputs. The model cannot guess the user's true intent, so instructions must be unambiguous and actionable.

- **Avoid abstract descriptions, switch to actionable instructions**:
    - *Low quality*: "Write an article about climate change."
    - *High quality*: "Write an 800-word explanatory article aimed at high school students, introducing the three main causes of climate change, with a real-world example for each cause, and ending with two action suggestions that individuals can take."
- **Quantify all quantifiable elements**: word count, number of paragraphs, number of list items, time range, precision requirements, etc., should all be clearly given as numbers.
- **Distinguish between mandatory and optional**: indicate in the instructions which requirements are hard constraints and which are preferences, avoiding errors when the model makes trade-offs.

#### 2. Provide sufficient context

Context is the basis for the model to accurately understand the task intent. Instructions lacking context will force the model to make a large number of assumptions, thereby increasing the risk of bias.

\subparagraph{2.1 Background Information (Background)}

Explain the situation of the task to help the model calibrate its knowledge scope and expression style:

- Task usage scenario: internal reporting, customer communication, academic writing, product copywriting, etc.
- Relevant domain or industry background: e.g., "This product is for the medical device regulatory field."
- Existing work foundation: e.g., "The following is a completed first draft, please revise based on this."

\subparagraph{2.2 Audience Information (Audience)}

Defining the target readers determines the language difficulty, terminology density, and expression style of the output:

- *Example*: "The readers are corporate executives with no technical background; please avoid using programming jargon, and all technical concepts must be explained using business language analogies."

\subparagraph{2.3 Persona Setting (Persona)}

Assigning a specific persona to the model can activate knowledge patterns in relevant domains and stabilize the output style:

- *Example*: "You are a senior product manager with 15 years of experience, skilled at translating technical requirements into user stories."
- *Example*: "You are a Socratic philosophy mentor, accustomed to guiding students to think independently through rhetorical questions, rather than giving answers directly."

Persona setting affects not only the content but also the model's reasoning path and response attitude.

#### 3. Use strong verbs to guide the response

The starting verb of the instruction directly determines the model's behavior pattern. Precise verbs should be selected according to the task type:

\begin{table}[h]



| **Task Type** | **Recommended Verbs** | **Vague Verbs to Avoid** |
| --- | --- | --- |
| Content Generation | Write, draft, generate, create | Do, make, write a bit |
| Information Extraction | List, extract, identify, locate | Look for, see if there is |
| Analysis and Evaluation | Analyze, evaluate, compare, diagnose | Talk about, chat about |
| Transformation and Rewriting | Translate, rewrite, compress, expand, convert | Process, change a bit |
| Structural Organization | Classify, summarize, sort, aggregate | Organize a bit |
| Reasoning and Decision Making | Infer, demonstrate, suggest, predict | Think about, guess |


	\caption{Task Type and Recommended Verbs Reference Table}
\end{table}

Additionally, when requiring the model to demonstrate the reasoning process, adverbs such as "step-by-step" or "in stages" can be used to modify verbs, such as "analyze step-by-step" or "plan in stages".

#### 4. Keep it concise and avoid information overload

Lengthy or repetitive prompts will dilute the weight of key instructions, making it difficult for the model to judge priorities. The core principles of conciseness are:

- **Do one thing at a time**: If the task is complex, break it down into multi-turn conversations or multiple independent prompts, rather than piling all requirements into a single request.
- **Remove redundant modifiers**: Avoid softening language like "please as much as possible" or "if convenient", as such expressions reduce the execution force of the instructions.
- **Avoid repeated declarations**: The same constraint only needs to be declared once; repeated declarations will not reinforce the effect, but rather increase interference.
- **Structure is better than stacking**: Use numbered lists, paragraphs, or tags (such as `[Task]`, `[Constraint]`, `[Example]`) to organize information, rather than writing all the content as a continuous paragraph.

#### 5. Make good use of examples (Few-Shot Prompting)

Examples are one of the most efficient ways to convey intent, especially suitable for tasks with complex formats or special styles:

- **Positive examples**: demonstrate the desired output format and content style.
- **Negative examples**: explicitly point out output forms that are not desired, helping the model eliminate ambiguity.
- **Number of examples**: usually 1 to 3 examples are sufficient; too many examples will compress the model's processing space for the actual task.

*Example structure reference*:
\begin{quote}
	**[Task]**: Rewrite the following product description into a one-sentence slogan, with a concise and powerful style, highlighting the core selling points.\\
	**[Example Input]**: This earphone uses active noise cancellation technology, has a 40-hour battery life, and weighs only 180 grams.\\
	**[Example Output]**: 40 hours of silence, light as nothing—focus, never interrupted.\\
	**[Actual Input]**: This air purifier covers an area of 120 square meters, has a triple filtration system, and a noise level below 30 decibels.
\end{quote}

#### 6. Guide the model to demonstrate the reasoning process

For complex tasks requiring accuracy, instructing the model to "think before outputting" can significantly improve result quality:

- *Example*: "Before giving the final answer, please list your reasoning steps first."
- *Example*: "Please analyze each sub-problem of this issue first, answer them one by one, and finally give a comprehensive conclusion."

This technique (namely Chain-of-Thought Prompting) is particularly effective in tasks involving mathematical reasoning, logical judgment, and multi-step planning.

#### 7. Iterative optimization rather than one-time completion

High-quality prompts are usually refined through multiple rounds. It is recommended to follow the iterative process below:

1. **First draft**: Write a basic version according to the core requirements of the task.
2. **Testing**: Verify whether the output meets expectations with diversified inputs.
3. **Attribution**: Analyze the root cause of outputs that do not meet expectations (instruction ambiguity, missing constraints, insufficient examples, etc.).
4. **Revision**: Supplement or modify the corresponding modules specifically.
5. **Regression**: Retest all historical use cases after modification to ensure no new issues are introduced.

Archiving the modification records of each iteration helps to accumulate a library of prompt engineering experience within the organization.

</details>

**2. How does prompt engineering affect LLM output?**

<details>
<summary>💡 Show Answer</summary>

Prompt engineering is the process of designing clear, specific, and goal-oriented input prompts to guide Large Language Models (LLMs) to achieve desired results. It is crucial in maximizing LLM performance, especially in the following scenarios:
- **Zero-shot learning**: The model performs a task without precedents, relying on the clarity of the prompt to infer intent.
- **Few-shot learning**: Providing a small number of examples in the prompt to guide the model.

**Effective prompt engineering**:
- Provides context to reduce ambiguity.
- Uses structured instructions to emphasize task requirements.
- Utilizes strategies such as Chain-of-Thought (CoT) prompting to elicit logical reasoning.

**Example**: Rewording “Summarize this text” to “Provide a 3-sentence summary focusing on the main theme” can improve the precision of the output.

</details>

**3. What is a System Prompt? How is it different from a User Message?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is a System Prompt? How is it different from](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig02_what_is_a_system_prompt_how_is.png)



A System Prompt is a global instruction set by the developer or system before the conversation starts, used to define the model's identity, behavioral guidelines, output format, knowledge scope, and constraints. It remains effective throughout the entire lifecycle of the conversation, acting as a constraint on all subsequent interactions. The User Message is the specific request or question input by the user turn by turn during the conversation, belonging to dynamic, real-time interactive content.

The main differences between the two are reflected in the following dimensions:

**Action level:** The System Prompt is at a higher priority, defining the "meta-behavior" of the model; the User Message is processed within this framework, subject to the constraints of the System Prompt.

**Persistence:** The System Prompt remains unchanged throughout the session; the User Message updates with each turn of the conversation, having timeliness.

**Author:** The System Prompt is usually written by product developers or operators and is invisible to the user; the User Message is input in real-time by the end-user.

**Typical content:** The System Prompt often includes persona setting ("You are a professional legal advisor"), output format requirements, prohibitions, language style, etc.; the User Message includes specific tasks, questions, or context supplements.

**Security boundary:** The System Prompt is the primary barrier for implementing security policies and preventing prompt injection attacks; its authority is higher than that of the User Message.

</details>

**4. What is Role Prompting? What impact does it have on output quality?**

<details>
<summary>💡 Show Answer</summary>

Role Prompting refers to assigning a clear identity or persona to the model in the prompt (e.g., "You are a senior data scientist", "You are a Socratic philosophy teacher"), thereby guiding the model to generate responses using the knowledge base, way of thinking, and expression style of that persona.

**Impact on output quality:**
- **Enhances professional depth**: After being assigned an expert persona, the model tends to use domain terminology and reference industry conventions, making the output more professional and authoritative.
- **Unifies tone and style**: Persona setting anchors the model's expression tone (rigorous, humorous, concise, etc.), maintaining consistency in multi-turn conversation styles.
- **Activates relevant knowledge**: The persona's cognitive framework guides the model to prioritize calling knowledge paths related to that persona, improving the relevance of the response.
- **Potential risks**: If the persona setting is too broad or mismatched with the task, it may introduce unnecessary bias or limit the model's flexibility; over-reinforcing a certain persona may also cause the model to reject reasonable requests outside the scope of that persona.

Best practice is to combine persona setting with specific task goals, for example: "You are a Python instructor for beginners, please explain the following concepts using analogies and simple examples."

</details>

**5. What are positive instructions and negative instructions? In what scenarios is each more effective?**

<details>
<summary>💡 Show Answer</summary>

**Positive Instructions** explicitly tell the model "what it should do", such as "answer in concise Chinese", "list three key points". **Negative Instructions** explicitly tell the model "what it should not do", such as "do not use jargon", "do not repeat the question".

**Scenarios where positive instructions are more effective:**
- When needing to guide the model to output in a specific direction, positive instructions provide a clear behavioral goal, have low cognitive load, and result in higher execution consistency.
- When defining structural requirements such as output format, length, and language style, positive instructions are more direct.
- Generally, positive instructions should be the first choice, as the model's understanding of "what to do" is better than its understanding of "what not to do".

**Scenarios where negative instructions are more effective:**
- When needing to exclude known high-frequency error behaviors, for example, if the model habitually adds disclaimers, "do not add a disclaimer" can be used to directly suppress it.
- Security and compliance constraint scenarios, where explicitly prohibiting specific content (e.g., "must not disclose user privacy data") is clearer than positive descriptions.
- Best used in conjunction with positive instructions: first use positive instructions to define the desired behavior, then use negative instructions to exclude interfering behaviors outside the boundaries.

</details>

**6. Why is "showing" (show) more effective than "describing" (tell)?**

<details>
<summary>💡 Show Answer</summary>

"Describing" (tell) means using abstract language to explain desired output features, such as "please write concisely", "please keep it professional"; "showing" (show) is directly providing concrete examples, allowing the model to infer the desired pattern through the examples.

The reasons why "showing" is more effective are reflected in the following aspects:

- **Eliminates ambiguity**: Abstract descriptions vary from person to person; "concise" means different word counts to different users. Concrete examples clearly anchor the expected output form, eliminating room for subjective interpretation.
- **Reduces instruction complexity**: Sometimes accurately describing an output format with language requires a large amount of text, whereas a single example can immediately convey the same amount of information.
- **Activates pattern matching capability**: LLMs have accumulated powerful pattern recognition capabilities during the pre-training phase; Few-Shot examples directly trigger this capability, enabling the model to quickly summarize and transfer the hidden rules in the examples.
- **Exact reproduction of format and style**: For specific output structures (such as tables, code comment styles, JSON templates), examples can ensure the model reproduces them accurately better than text descriptions.

Practical advice: Use "describing" to establish principles and "showing" to provide anchors in the prompt simultaneously; combining the two can achieve the best results.

</details>

**7. What is the relationship between the length and quality of a prompt? Is longer always better?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the relationship between the length and qu](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig03_what_is_the_relationship_betwe.png)



There is no positive correlation between the length and quality of a prompt; the effective information density is the core factor determining output quality.

**Situations where moderately increasing length is beneficial:**
- Providing necessary background information, constraints, and examples helps the model accurately understand the task intent.
- When complex tasks require step-by-step instructions, detailed step descriptions can prevent the model from skipping steps or omitting things.
- When providing reference documents or context data, the more complete the content, the higher the generation quality.

**Problems caused by overly long prompts:**
- **"Lost in the Middle" phenomenon**: Research shows that the model's attention to the middle part of a very long context drops significantly, and key instructions placed in the middle may be ignored.
- **Risk of instruction conflict**: The more content, the higher the probability of contradictions, and the model may exhibit uncertain behavior between conflicting instructions.
- **Increased token cost**: In API call scenarios, lengthy prompts directly increase inference costs and latency.
- **Attention dilution**: When core instructions are surrounded by a large amount of irrelevant content, the model's compliance rate with key requirements decreases.

Best practice is: design prompts with the "principle of minimum sufficiency" — include all information necessary to complete the task, remove all redundant descriptions, and place the most critical instructions at the beginning or end.

</details>

**8. What is the "ambiguity" problem of Prompts? How to avoid it?**

<details>
<summary>💡 Show Answer</summary>

Prompt ambiguity refers to the phenomenon where the formulation of an instruction has multiple reasonable interpretations, causing the model's output to deviate from the user's true intent. Ambiguity can be divided into three categories: semantic ambiguity (unclear word meanings), scope ambiguity (unclear task boundaries), and format ambiguity (undefined output structure).

**Common examples of ambiguity:** "Help me summarize this article" — the model cannot determine the target length, audience, granularity of the summary, and whether to preserve the original structure.

**Strategies to avoid ambiguity:**
- **Clarify task goals and audience**: Explain the usage scenario and target readers of the output, for example, "Write an executive summary of under 200 words for executives with no technical background."
- **Define output format**: Use structural language or examples to specify the expected output form, such as "Output in Markdown list format, with each item not exceeding 20 words."
- **Limit scope and constraints**: Specify the boundary conditions of the task, such as "Answer solely based on the following document content, without introducing external information."
- **Use delimiters to isolate content**: Clearly separate instructions from data using XML tags, triple quotes, etc., to avoid the model confusing task descriptions with the content to be processed.
- **Ask the model to restate the task**: In complex scenarios, first ask the model to restate its understanding of the task in its own words before execution, in order to discover ambiguities early.

</details>

**9. How to control the tone and style of the model's output via Prompts?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to control the tone and style of the model's o](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig04_how_to_control_the_tone_and_st.png)



Controlling tone and style is an important dimension of Prompt engineering, which can be achieved through the following methods:

- **Directly describe style attributes**: Explicitly specify the target style in the Prompt, such as "use a formal, objective academic tone," "use a relaxed and humorous conversational style," or "avoid using passive voice."
- **Anchor style through role-playing**: Indirectly define style via Role Prompting, such as "You are a columnist for The Economist," and the model will automatically invoke the writing conventions associated with that role.
- **Provide style examples (Few-Shot)**: Give typical paragraphs of the target style, allowing the model to reproduce the style by imitating the examples, which is more precise than abstract descriptions.
- **Specify audience**: Audience definition naturally constrains vocabulary choices and sentence complexity; for example, "explain to a 10-year-old child" and "explain to a PhD student" will produce drastically different outputs.
- **Refine language parameters**: You can specifically require sentence length ("use short sentences, no more than 15 words per sentence"), vocabulary level ("avoid jargon, use everyday language"), emotional tone ("remain neutral, without emotional coloring"), etc.
- **Iterative correction**: Add instructions on top of the first draft for fine-tuning, such as "the above answer is too formal, please rewrite it to be more colloquial," gradually approaching the target style.

</details>

**10. What are Delimiters? What role do they play in Prompts?**

<details>
<summary>💡 Show Answer</summary>

Delimiters are markup symbols used in Prompts to distinguish different content blocks. Common forms include triple quotes (`"""`), XML tags (`<text></text>`), Markdown code blocks (““`), square brackets (`[]`), dash separators (`---`), etc.

**Main roles of delimiters:**
- **Prevent Prompt injection**: Clearly isolate user-input data from system instructions to prevent malicious users from hijacking model behavior by embedding forged instructions within the data. For example, placing text submitted by the user within `<user_input></user_input>` tags allows the model to clearly identify it as "content to be processed" rather than "instructions to execute."
- **Clarify content boundaries**: In Prompts where multiple segments of content coexist (instructions, examples, reference documents, text to be processed), delimiters help the model accurately identify the semantic role of each block.
- **Improve parsing accuracy**: Structured separation makes it easier for the model to locate key information, reducing misunderstandings caused by content confusion.
- **Support complex task orchestration**: In multi-step tasks, delimiters can be used to mark the inputs and outputs of different stages, guiding the model to process each subtask in order.

</details>

**11. How to effectively use XML tags or Markdown structures in Prompts?**

<details>
<summary>💡 Show Answer</summary>

XML tags and Markdown structures are two main categories of tools for organizing complex content in Prompts, each with its applicable scenarios and usage specifications.

**Effective use of XML tags:**
- **Semantic naming**: Tag names should clearly express the semantic role of their content, such as `<instructions>`, `<context>`, `<examples>`, `<output_format>`, making it easy for the model to understand the function of each block.
- **Nested structures express hierarchical relationships**: Nested tags can be used to express primary-secondary relationships, such as nesting multiple `<example>` tags within `<examples>`.
- **Isolate user input**: Wrap dynamically inserted user data in dedicated tags to prevent it from being misinterpreted as instructions by the model.
- **Guide structured output**: You can require the model to output answers within specific tags, such as "Please put your reasoning process inside `<reasoning>` tags, and the final answer inside `<answer>` tags."

**Effective use of Markdown structures:**
- **Hierarchical headings for organizing long Prompts**: For complex Prompts containing multiple modules, use headings like `#`, `##`, etc., to divide blocks, improving readability and the model's structural awareness.
- **Code blocks for isolating code content**: Wrap code snippets or formatted data in backtick code blocks to prevent special characters from being incorrectly parsed.
- **Lists for clear enumeration requirements**: Use ordered or unordered lists to list multiple parallel instructions, which is easier for the model to follow point by point than continuous paragraphs.
- **Environment adaptation**: Note that not all model APIs have the same rendering support for Markdown; in plain text environments, XML tags should be preferred.

</details>

**12. How to control the length of the model's output via Prompts?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to control the length of the model's output vi](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig05_how_to_control_the_length_of_t.png)



- **Directly specify word or character counts**: Such as "Please answer within 150 words," "Please provide a summary of no more than 500 words," providing a clear quantitative upper limit.
- **Specify the number of output units**: Such as "List 5 key points," "Provide 3 examples," indirectly controlling length by limiting the number of items.
- **Define output structure**: Require the model to output a fixed format (e.g., a single-paragraph summary, a one-sentence conclusion, a three-column table); the structure itself constrains the length range.
- **Explicitly forbid redundant content**: Such as "Do not repeat the question," "Do not add a summary paragraph," "Do not explain your reasoning process, only give the conclusion," compressing output through exclusion.
- **Hierarchically control detail level**: Set different levels of detail for different parts, such as "Summarize the background in one sentence, and elaborate on the plan in three paragraphs."
- **Be aware of the model's inherent tendencies**: Current mainstream LLMs have a tendency to over-generate. Length control instructions need to be placed in prominent positions in the Prompt (beginning or end) and can be globally set in the System Prompt to improve compliance rates.

</details>

**13. How to design Prompts so the model does not skip steps when processing multi-step tasks?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to design Prompts so the model does not skip s](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig06_how_to_design_prompts_so_the_m.png)



- **Explicitly enumerate steps**: Break the task down into ordered, numbered steps and explicitly require the model to "execute sequentially according to the following steps, do not skip any step."
- **Require output for each step's result**: After each step, require the model to output the intermediate result of that step, making skipping behavior visible in the output, which facilitates checking and error correction.
- **Set inter-step dependencies**: Explicitly declare that subsequent steps take the output of the previous step as input, such as "Step 3: Based on the conclusion reached in Step 2, further analyze...", strengthening the logical chain between steps.
- **Use Chain-of-Thought (CoT) prompting**: Add "Let's think step by step" in the Prompt or provide Few-Shot examples containing complete reasoning chains, activating the model's step-by-step reasoning mode.
- **Execute complex tasks in multiple rounds**: For tasks with strong inter-step dependencies, they can be split into multiple dialogue rounds, executing only one step per round, taking the previous round's output as the next round's input, fundamentally eliminating the space for skipping steps.
- **Verification and backtracking instructions**: Require the model to self-check whether every step has been completed after finishing all steps, and to perform supplementary execution if omissions are found.

</details>

**14. What is the basic workflow for iterative optimization of Prompts?**

<details>
<summary>💡 Show Answer</summary>

The iterative optimization of Prompts is a systematic engineering process, and its basic workflow is as follows:

- **Step 1: Define goals and evaluation criteria**: Before starting optimization, clearly define the evaluation dimensions for a "good output" (accuracy, format, style, length, etc.), and prepare a batch of test cases covering typical scenarios and edge cases.
- **Step 2: Draft a baseline Prompt**: Based on a preliminary understanding of the task, write the simplest and most direct version to serve as a baseline for subsequent iterations.
- **Step 3: Execute testing and collect outputs**: Run the baseline Prompt on the test set, systematically collecting the model's outputs, avoiding reliance on single subjective judgments.
- **Step 4: Diagnose failure modes**: Analyze outputs that do not meet expectations, and deduce the root causes of failure (instruction ambiguity, lack of examples, undefined formats, insufficient context, etc.).
- **Step 5: Targeted modification**: Only modify one variable per iteration (e.g., adding an example, changing the role setting, adjusting the order of instructions) in order to precisely pinpoint the effect of the change.
- **Step 6: Comparative evaluation**: Compare the modified outputs with the previous version on the same test set, using quantitative metrics (like pass rate, manual scoring) to determine if there is an improvement.
- **Step 7: Solidification and documentation**: Incorporate validated effective changes into the formal Prompt, record the reasons and effects of each iteration's modifications, and build a Prompt engineering knowledge base.
- **Continuous monitoring**: Continuously collect real user cases in the production environment, periodically triggering a new round of optimization iteration.

</details>

**15. What is "Over-specification"? What problems does it cause?**

<details>
<summary>💡 Show Answer</summary>

Over-specification refers to the phenomenon where a Prompt contains redundant constraints, repetitive instructions, or contradictory detailed regulations that go beyond what is actually needed to complete the task, leading to excessively high instruction density.

**Main problems caused by over-specification:**
- **Attention dilution**: The model's effective attention is limited. Too many instructions will cause key constraints to be drowned out, and the compliance rate with core requirements will actually decrease.
- **Instruction conflict**: The more constraints there are, the higher the probability of inconsistencies. When facing conflicts, the model will produce uncertain behavior, resulting in unstable outputs.
- **Constrained creativity**: In tasks requiring the model to exercise flexibility or creativity, overly detailed constraints will compress the model's solution space, leading to rigid and lifeless outputs.
- **Increased maintenance costs**: Lengthy and complex Prompts are difficult to maintain and debug. Modifying one constraint may trigger chain conflicts, resulting in low iteration efficiency.
- **Token waste**: Redundant content directly increases inference costs, which is especially noticeable in high-concurrency API scenarios.

The coping strategy is to periodically review Prompts, identify and delete redundant instructions that do not substantially contribute to output quality, and adhere to the "Minimum Sufficient Principle".

</details>

**16. What is "Under-specification"? What problems does it cause?**

<details>
<summary>💡 Show Answer</summary>

Under-specification refers to the phenomenon where a Prompt fails to provide key information, constraints, or examples necessary to complete a task, causing the model to have to rely on its own default assumptions to fill in the gaps.

**Main problems caused by under-specification:**
- **Unpredictable outputs**: When information is missing, the model will rely on statistical preferences from training data for completion; outputs vary significantly between different runs, and consistency is poor.
- **Default assumption bias**: The model's default assumptions may deviate from the user's true intent. For example, the default length for a "summary" or the default definition of "professional" varies by task and user.
- **Increased hallucination risk**: When a task requires specific facts or formats but the Prompt does not provide them, the model is more likely to generate seemingly plausible but actually incorrect content out of thin air.
- **Errors are hard to locate**: Output deviations caused by under-specification are often difficult to trace back because the model "followed instructions," but the instructions themselves were simply incomplete.
- **Degraded user experience**: In product scenarios, unstable outputs decrease user trust and increase the burden of manual review.

The coping strategy is to actively anticipate stages where the model might generate ambiguity or need assumptions when designing Prompts, and to supplement explicit constraints or examples at these key nodes.

</details>

