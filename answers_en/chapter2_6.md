# 2.6 Chain-of-Thought (CoT) and Reasoning Enhancement — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 18 questions

**1. How does Chain-of-Thought (CoT) prompting improve the reasoning abilities of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Chain-of-Thought (CoT) prompting is a technique that guides Large Language Models (LLMs) to incrementally generate intermediate reasoning steps before outputting the final answer. Its core idea is to simulate the human thought process when solving complex problems—breaking the problem down into a series of ordered sub-steps, thereby reducing the cognitive difficulty of each step.

#### Basic Principles

Traditional prompting methods directly ask the model for an answer, whereas CoT prompting explicitly demonstrates the reasoning chain in examples, guiding the model to generate similar intermediate steps during the inference phase. Its typical form is as follows:

\begin{lstlisting}[style=promptstyle]
	Question: Xiao Ming has 5 apples. He gave Xiao Hong 2, and bought 3 more. How many does he have now?
	Let's think step by step:
	1. Xiao Ming initially has 5 apples.
	2. He gave Xiao Hong 2, leaving 5 - 2 = 3 apples.
	3. He bought 3 more, totaling 3 + 3 = 6 apples.
	Answer: Xiao Ming now has 6 apples.
\end{lstlisting}

#### Mechanisms for Enhancing Reasoning Abilities

CoT prompting improves the reasoning abilities of LLMs from the following dimensions:

- **Decomposing Complex Problems**: Breaking down multi-step problems into several sub-problems allows the model to handle only local logic at each step, reducing the probability of errors.
- **Activating Implicit Knowledge**: An explicit reasoning chain forces the model to invoke and organize the knowledge acquired during pre-training, rather than relying solely on superficial pattern matching.
- **Improving Interpretability**: Intermediate steps make the reasoning process transparent, facilitating manual inspection and error localization.
- **Generalizing to New Tasks**: Through a few examples (few-shot CoT) or zero-shot instructions (zero-shot CoT, such as "Let's think step by step"), the model can transfer reasoning patterns to unseen task types.

#### Applicable Scenarios

CoT is particularly effective in tasks such as math word problems, logical reasoning, commonsense question answering, multi-step planning, and code debugging, which require systematic logical deduction. Research indicates that the benefits of CoT are more pronounced when the model parameters are large enough (usually exceeding 100B), whereas smaller models benefit relatively little.

</details>

**2. If CoT prompting fails, how do you improve the LLM's reasoning abilities?**

<details>
<summary>💡 Show Answer</summary>

When an initial CoT prompt fails to achieve the expected results, systematic improvements can be made from prompt design, technical methods, and the data level.

#### Refining Prompt Design

- **Eliminating Ambiguity**: Review the prompt for vague expressions and clarify the boundary conditions and expected output format of the problem.
- **Supplementing Missing Steps**: If the model skips certain key reasoning steps, explicitly complete these steps in the few-shot examples.
- **Increasing Example Diversity**: Provide examples covering different difficulties and structures to help the model better understand the generalization of reasoning patterns.

#### Exploring Advanced Prompting Techniques

- **Self-Consistency**: Sample multiple reasoning paths for the same problem and select the most consistent answer through majority voting, effectively reducing the random error of a single inference.
- **Tree of Thoughts (ToT)**: Model the reasoning process as a tree structure, allowing the model to search and backtrack among multiple candidate branches, suitable for tasks requiring exploratory reasoning.
- **Providing Background Knowledge and Assumptions**: Inject relevant domain knowledge into the prompt or explicitly list premise assumptions to reduce reasoning biases caused by the model's lack of knowledge.
- **Encouraging Multi-perspective Analysis**: Instruct the model to examine the problem from different perspectives or counterexamples to enhance the robustness of reasoning.
- **Step-by-step Verification**: Require the model to perform self-verification of each reasoning step after giving the final answer, proactively identifying and correcting errors.

An improved prompt example combining the above strategies:

\begin{lstlisting}[style=promptstyle]
	Please answer the following question. When answering:
	1. List all known conditions and assumptions.
	2. Decompose the problem into several sub-steps and solve them one by one.
	3. After completion, review each step to check for logical flaws.
	4. Provide the final answer and state your confidence level.

	Question: [Question content]
\end{lstlisting}

#### Introducing Human Feedback

Through step-by-step annotation and feedback by humans on the model's reasoning process (such as Reinforcement Learning from Human Feedback, RLHF), the model's reasoning preferences can be systematically corrected to better align with human logical expectations.

#### Improving Training Data

Ensure the fine-tuning dataset contains diverse, high-quality reasoning examples covering various reasoning patterns of the target task. If necessary, construct contrastive data specifically for failed cases (correct reasoning vs. incorrect reasoning), and strengthen the model's preference for the correct reasoning path through contrastive learning.

</details>

**3. How to use stop sequences in LLMs?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to use stop sequences in LLMs?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig42_how_to_use_stop_sequences_in_l.png)



A Stop Sequence is one or more specific strings specified in advance by the user. When the LLM encounters these strings during the generation process, it will immediately stop text generation. This mechanism provides a precisely controllable termination boundary for model output.

#### Working Principle

During the inference phase, the LLM generates text token by token in an autoregressive manner. When the generated token sequence perfectly matches the preset stop sequence, the generation process terminates. The stop sequence itself is usually not included in the final output. Most mainstream APIs (like OpenAI, Anthropic) directly support this feature through parameters:

\begin{lstlisting}[style=promptstyle]
	# Example: Setting stop sequences using Anthropic Python SDK
	import anthropic

	client = anthropic.Anthropic()

	response = client.messages.create(
	model="claude-sonnet-4-6",
	max_tokens=1024,
	stop_sequences=["Human:", "###", "\n\n"],
	messages=[
	{"role": "user", "content": "Please list three fruits, one per line."}
	]
	)

	print(response.content)
	print("Stop Reason:", response.stop_reason)  # Output: stop_sequence or end_turn
\end{lstlisting}

#### Common Use Cases

- **Multi-turn Dialogue Control**: In dialogue systems, use `"Human:"` or `"User:"` as a stop sequence to prevent the model from continuing to write the user's input itself, ensuring correct segmentation of dialogue turns.

    \begin{lstlisting}[style=promptstyle]
    stop_sequences=["Human:", "User:"]
    \end{lstlisting}

- **Structured Output Truncation**: When requiring the model to generate JSON, code blocks, or lists, use specific delimiters (such as \verb|"“`"| or `"</output>"`) to mark the end of the output, preventing the model from generating extraneous content.

    \begin{lstlisting}[style=promptstyle]
    Prompt: Please summarize the following content in JSON format, and write </output> after the output.
    stop_sequences=["</output>"]
    \end{lstlisting}

- **Step-by-step Reasoning Control**: In scenarios requiring step-by-step review of the reasoning process, use markers like `"[STEP END]"` to let the model pause after completing each step, letting an external program decide whether to continue.

- **Preventing Content Overflow**: Stop after generating a specific number of list items, for example, using empty lines as stop sequences to limit the number of output paragraphs.
#### Notes

- Stop sequences should be distinctly different from normal output content to avoid false triggers that cause premature truncation of the output.
- Multiple stop sequences can be set simultaneously; if any one is matched, it triggers a stop.
- Stop sequences and the `max_tokens` parameter are independent of each other. Both can serve as generation termination conditions, and the actual output is determined by whichever triggers first.
- The `stop_reason` field returned via the API can distinguish whether generation was terminated by a stop sequence or by reaching the token limit, facilitating downstream logic processing.

</details>

**4. What is Chain-of-Thought (CoT) Prompting? How does it improve reasoning capabilities?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Chain-of-Thought (CoT) Prompting? How does](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig43_what_is_chain_of_thought_cot_p.png)



CoT is providing examples containing step-by-step reasoning processes in the prompt (or directly adding "Let's think step by step"), guiding the model to explicitly output intermediate reasoning steps before giving the answer. Reasons for improving reasoning capabilities:
1. **Sub-problem decomposition**: Breaking down complex problems into smaller steps, reducing the difficulty of each step;
2. **Working memory expansion**: Intermediate reasoning tokens act as "working memory", allowing the model to pass information across multiple steps;
3. **Error visualization**: Incorrect reasoning steps are exposed in intermediate tokens, helping subsequent steps to correct them;
4. **Test-time compute scaling**: Generating more tokens = more computation, directly helping reasoning tasks (Test-Time Compute Scaling).

</details>

**5. What is the difference between Zero-shot CoT and Few-shot CoT? What scenarios are they suitable for?**

<details>
<summary>💡 Show Answer</summary>

**Zero-shot CoT**: Only adds "Let's think step by step" (or similar trigger words) without providing examples with reasoning steps. Suitable for:
1. No high-quality annotated examples of reasoning steps;
2. Quick experimentation, low cost;
3. Diverse task types, difficult to design CoT examples for each task separately.
**Few-shot CoT**: Provides 3-8 examples containing complete reasoning chains (input $\rightarrow$ reasoning steps $\rightarrow$ answer). Suitable for:
1. Having manually annotated reasoning steps;
2. Tasks with a specific reasoning structure (like math problem-solving formats);
3. High performance requirements, willing to pay the cost of example design.
Effect comparison: Few-shot CoT > Zero-shot CoT, but Zero-shot CoT is more convenient to use, and the effects are similar on large models.

</details>

**6. For which types of tasks does CoT improve most significantly? For which tasks is the improvement limited?**

<details>
<summary>💡 Show Answer</summary>

**Tasks with the most significant improvement**:
1. Mathematical reasoning (arithmetic, algebra, geometry);
2. Logical reasoning (propositional logic, causal inference);
3. Commonsense reasoning (requiring multi-step thinking);
4. Code generation and debugging;
5. Complex planning (tasks requiring sequential steps).
**Tasks with limited improvement**:
1. Simple factual Q&A (can be answered with a single-step lookup);
2. Text classification (pattern matching type, no reasoning chain needed);
3. Translation (usually does not require intermediate steps);
4. Models with very small parameters (<7B): CoT does not change the underlying capability, it merely activates it; when the model has no reasoning capability, CoT cannot help;
5. Purely memorization tasks.

</details>

**7. What is Auto-CoT? How does it automatically generate CoT examples?**

<details>
<summary>💡 Show Answer</summary>

Auto-CoT (Zhang et al. 2022): Automatically generates reasoning steps for few-shot CoT, reducing manual annotation. Workflow:
1. Use K-Means to cluster test questions, selecting one representative question for each cluster;
2. For each representative question, use Zero-shot CoT ("Let's think step by step") to let the model automatically generate reasoning steps;
3. Use the generated "question + reasoning steps + answer" triplets as few-shot examples;
4. Use these automatically generated examples to perform few-shot CoT reasoning on the test set.
Effect: On most mathematical reasoning benchmarks, the performance is comparable to manually annotated CoT, significantly reducing labor costs.
Limitations: Automatically generated reasoning steps may contain errors, affecting overall performance.

</details>

**8. What is Self-Consistency? How does it cooperate with CoT to improve reasoning quality?**

<details>
<summary>💡 Show Answer</summary>

Self-Consistency (Wang et al. 2022): For the same problem, sample multiple different reasoning paths using CoT (with a high Temperature), perform a majority vote on the final answers, and take the most frequent answer as the final output. Mechanism of cooperating with CoT:
1. CoT generates reasoning paths (intermediate steps);
2. Self-Consistency solves the problem that a single CoT path might go wrong;
3. Correct reasoning paths tend towards the same answer under different expressions;
4. Answers from wrong paths are more scattered and eliminated by majority voting.
Effect: On mathematical reasoning benchmarks like GSM8K, Self-Consistency + CoT improves by 5-15% over single CoT, with more obvious improvements on difficult problems.
Cost: Requires 20-40 times of sampling, increasing reasoning costs by 20-40 times.

</details>

**9. How does the sampling count of Self-Consistency affect performance and cost?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How does the sampling count of Self-Consistency af](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig44_how_does_the_sampling_count_of.png)



**Relationship between sampling count and performance**:
1. 1 time (single CoT): baseline, no Self-Consistency;
2. 5 times: significant improvement, covering main correct reasoning paths;
3. 20 times: substantial improvement, most tasks show obvious benefits here;
4. 40 times: close to the performance upper limit, marginal returns diminish;
5. >40 times: almost no further improvement, cost continues to increase.
**Cost-performance tradeoff**:
1. Low-cost scenarios (simple tasks): 5 samples;
2. General scenarios: 10-20 times;
3. High precision requirements (math competition problems): 40 times;
4. High real-time requirements: not suitable for massive sampling, consider alternatives like Beam Search.
Practical advice: Start from 10 times, and determine the optimal sampling count based on the performance-cost curve.

</details>

**10. What is Tree-of-Thoughts (ToT)? How does it differ from CoT?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Tree-of-Thoughts (ToT)? How does it differ](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig45_what_is_tree_of_thoughts_tot_h.png)



ToT (Yao et al. 2023): Organizes the reasoning process into a tree structure, generates multiple candidate thoughts at each decision point, scores candidate thoughts through an evaluation function (LLM self-evaluation), and only retains promising paths to continue expanding, supporting backtracking. Differences from CoT:
1. **Structure**: CoT is a single linear reasoning chain; ToT is a multi-path tree search;
2. **Backtracking**: CoT cannot be corrected once it goes wrong; ToT can backtrack to better branching points;
3. **Evaluation mechanism**: ToT introduces explicit evaluation of intermediate states; CoT does not;
4. **Applicable tasks**: ToT is suitable for planning and search problems requiring exploration of multiple possibilities (Game of 24, mazes); CoT is suitable for tasks with clear reasoning steps;
5. **Cost**: ToT requires multiple LLM calls, and the cost is much higher than CoT.

</details>

**11. What is Program-of-Thoughts (PoT)? In what scenarios is it better than CoT?**

<details>
<summary>💡 Show Answer</summary>

PoT (Chen et al. 2022): Writes reasoning steps as executable code (Python) rather than natural language, obtaining the answer by actually executing the code. Differences from CoT: CoT asks the LLM to "calculate" in natural language, which is prone to numerical errors; PoT asks the LLM to generate code, and the calculation is executed precisely by Python. Scenarios better than CoT:
1. **Numerical calculation**: Addition, subtraction, multiplication, division, statistics, mathematical formulas. CoT is prone to miscalculation, code execution does not err;
2. **Data manipulation**: Operations on data structures like lists and dictionaries;
3. **Step-by-step algorithms**: Algorithms requiring precise steps like sorting and searching;
4. **Complex mathematics**: Calculus, matrix operations, etc., which are difficult for LLMs to reason precisely.
Limitations: Requires a safe code execution environment; has no advantage in natural language reasoning (logical relationship judgment).

</details>

**12. What is Least-to-Most Prompting? How does it decompose complex reasoning problems?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Least-to-Most Prompting? How does it decom](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig46_what_is_least_to_most_promptin.png)



Least-to-Most Prompting (Zhou et al. 2022): Decomposes a complex problem into a series of simpler sub-problems first, solves them sequentially from simple to complex, and uses the answer of each sub-problem as the context for solving the next. Two stages:
1. **Decomposition stage**: Prompts the LLM "To solve this problem, what sub-problems do I need to solve first?", generating a list of sub-problems;
2. **Step-by-step solving**: Starting from the simplest sub-problem, appending previous answers into the prompt at each step to gradually solve more complex sub-problems.
Advantage: Particularly suitable for reasoning tasks that require "learning basics before solving hard problems"; compared to standard CoT, it shows significant improvement on compositional generalization tasks.
Suitable scenarios: Math word problems, programming task decomposition, multi-step Q&A.

</details>

**13. What is Analogical Reasoning Prompting? What is its core idea?**

<details>
<summary>💡 Show Answer</summary>

Analogical Reasoning Prompting (PS+, Wang et al. 2023): Asks the LLM to autonomously recall solving methods of similar problems before answering the question, and then applies these analogical methods to the current problem. Core idea: When humans face a new problem, they naturally associate it with similar problems solved before to draw on existing experience. Implementation:
1. Add an instruction in the prompt "Recall a problem similar to the current one and its solution";
2. The LLM generates an analogical example and reasoning process;
3. Solves the current problem based on analogical reasoning.
Advantage: No need to manually design CoT examples, the LLM autonomously generates relevant examples; performance on mathematical reasoning tasks is close to manually designed CoT, and more flexible.

</details>

**14. Are CoT reasoning steps the longer the better? What are the research findings?**

<details>
<summary>💡 Show Answer</summary>

Not the longer the better. Research findings:
1. **Appropriate length is beneficial**: Reasoning steps too short (<3 steps) yield poor results, while appropriate expansion (5-10 steps) works best;
2. **Too long can be harmful**: When there are too many steps, the model easily introduces errors in the middle, and errors propagate to the final answer;
3. **"Misleading CoT" phenomenon**: If the reasoning steps themselves contain errors, these errors will guide the model to draw wrong conclusions, worse than no CoT;
4. **Task dependence**: Complex math problems need more steps; for simple reasoning problems, too many steps are noise;
5. **Quality > Length**: Precise, concise reasoning steps are better than lengthy but vague steps;
6. **Practical advice**: Reasoning steps only need to cover all necessary logical leaps, no need to fill in redundant steps.

</details>

**15. How to evaluate the quality of CoT reasoning chains (not just looking at the final answer)?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to evaluate the quality of CoT reasoning chain](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig47_how_to_evaluate_the_quality_of.png)



1. **Step accuracy**: Score each intermediate step of the reasoning chain separately (correct/incorrect), counting the error distribution of each step;
2. **Logical coherence**: Evaluate whether the logical relationship between adjacent steps is valid (LLM-as-judge scoring);
3. **Consistency between reasoning chain and answer**: Check whether the conclusion supported by the reasoning chain is consistent with the final answer (if reasoning chain is correct but answer is wrong $\rightarrow$ answer extraction issue);
4. **Error propagation analysis**: Where does the first error step appear, and how is the accuracy of subsequent steps (analyzing error propagation);
5. **Faithfulness**: Is the final answer truly derived from the reasoning chain, or did the model "put aside" the reasoning chain to give an answer unrelated to the reasoning (Turpin et al. 2023 found CoT is sometimes post-hoc rationalization);
6. **Tools**: Process Reward Model (PRM), scoring each reasoning step.

</details>

**16. Why is the CoT capability an "emergent ability"? Why does it fail on small models?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of Why is the CoT capability an "emergent ability"? W](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig48_why_is_the_cot_capability_an_e.png)



Reason for CoT as an emergent ability: When the model scale exceeds about 100B parameters, CoT capabilities suddenly manifest (CoT hardly helps or is even harmful on small models), reflecting the non-linear emergence of capabilities. Reasons for failure on small models:
1. **Insufficient capacity**: Small models have limited ability to simultaneously maintain multi-step reasoning, intermediate steps will "confuse" rather than "help";
2. **Weak reasoning foundation**: CoT only activates existing reasoning capabilities, small models inherently lack reasoning capability, CoT cannot create capabilities that do not exist;
3. **Weak instruction following**: Small models might not understand the "step-by-step reasoning" instruction itself;
4. **Insufficient attention**: Maintaining a long reasoning chain requires the model to keep precise attention on multiple intermediate steps, small models lack attention quality;
5. **Verification**: The same CoT prompt shows almost no improvement on a 7B model, obvious improvement on 70B, and the most significant improvement on 175B+.

</details>

**17. What is a Process Reward Model (PRM)? How does it improve the quality of CoT?**

<details>
<summary>💡 Show Answer</summary>

PRM (Lightman et al. 2023, OpenAI Let's Verify Step by Step): Scores each intermediate step of the CoT reasoning chain (instead of only scoring the final answer), used to evaluate and filter high-quality reasoning paths. Ways it improves CoT:
1. **Best-of-N Sampling**: Generate N CoT reasoning chains for the same problem, PRM scores the step quality of each chain, and the chain with the highest total score is selected as the final answer;
2. **Beam Search Guidance**: When generating reasoning steps step-by-step, use PRM scores to guide the search direction, pruning poor-quality steps;
3. **Data Filtering**: Use PRM to score reasoning chains in training data, keeping only high-quality reasoning chains for Fine-tuning;
4. Compared to Outcome Reward Model (only looking at whether the final answer is right or wrong), PRM provides a finer-grained supervision signal, particularly helpful in reducing the issue of "fluke correctness" (reasoning wrong but answer happens to be right).

</details>

**18. What is Verification-guided CoT? How does it allow the model to self-verify reasoning steps?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is Verification-guided CoT? How does it allow](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig49_what_is_verification_guided_co.png)



Verification-guided CoT (or Self-Verification CoT): After generating CoT reasoning steps, let the model verify its own reasoning process, discover errors, correct them, and then output the final answer. Implementations:
1. **Backward Verification**: After generating the answer, let the model deduce backwards from the answer back to the problem conditions to verify consistency;
2. **Step-level Verification**: Insert "check if this step is correct" after each reasoning step, and backtrack when an error is found;
3. **Dual Prompt**: One prompt generates the answer, another prompt acts as a "critic" to check the reasoning, both dialoguing until a consensus is reached;
4. **Self-Refine**: After completing the initial draft, let the model provide improvement suggestions for its own reasoning, and rewrite based on suggestions.
Effect: On mathematical and logical reasoning tasks, self-verification can reduce errors by about 10-20%, especially simple calculation errors.

</details>

