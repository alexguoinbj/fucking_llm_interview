# 6.2 Reasoning Frameworks, Planning, and Decision-Making — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 6: Agent · 43 questions

**1. What is the ReAct Framework? How Does the Thought-Action-Observation Loop Work?**

<details>
<summary>💡 Show Answer</summary>

ReAct (Reasoning + Acting): Allows the LLM to alternately output Thought and Action, observe environmental feedback (Observation), and then continue reasoning. Workflow:
1. **Thought**: The model writes out its reasoning process in natural language ("I need to query user information first, then check orders");
2. **Action**: Specifies the tool to call and its parameters (`search_user(id=123)`);
3. **Observation**: Tool return results are written into the context (`\{"name": "Zhang San", "email": "..."\}`);
4. The model reads the Observation and continues thinking to decide on the next step. Advantages: Reasoning process is visible and easy to debug; intermediate steps are verifiable, reducing error propagation.

</details>

**2. Please Explain the ReAct Framework in Detail. How Does It Combine Chain-of-Thought and Action to Complete Complex Tasks?**

<details>
<summary>💡 Show Answer</summary>

The core idea of ReAct is that when humans solve complex problems, they tightly intertwine "thinking" and "acting"—first thinking, then taking action, observing results, and thinking again based on those results. ReAct mimics this loop pattern of "Thinking $\rightarrow$ Action $\rightarrow$ Observation $\rightarrow$ Thinking...".

Traditional CoT generates all thinking steps at once and then produces an answer, whereas ReAct's "thinking" is a dynamic chain of thought prior to each action, based on the latest observation results. This combination enables the Agent to handle dynamic environments (adjusting strategies in real time based on new information), perform error correction (analyzing failure reasons and attempting different actions), and complete complex tasks requiring multi-step reasoning and tool interactions.




![Illustration of Please Explain the ReAct Framework in Detail. How Does It Combine Chain-of-Thought and Action to Complete Complex Tasks?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig05_please_explain_react_framework.png)

</details>

**3. What is Model Self-Reflection in LLM Agents?**

<details>
<summary>💡 Show Answer</summary>

It typically refers to the process where a model evaluates and adjusts its own outputs or behaviors. This self-reflection may include the following aspects:
- **Performance Evaluation**: The model may periodically evaluate its own performance on specific tasks, such as language generation, text classification, or dialogue systems.
- **Error Analysis**: The model analyzes errors it produces, including misunderstanding inputs, generating non-fluent text, or producing outputs inconsistent with context.
- **Domain Adaptation**: If the model performs poorly in a specific domain, it may attempt to adapt itself to that domain.
- **User Feedback Processing**: If the model receives user feedback, it may leverage this feedback to improve itself.

</details>

**4. What Methods Exist for Model Self-Reflection in LLM Agents?**

<details>
<summary>💡 Show Answer</summary>

Methods for self-reflection in LLM Agents can be categorized from the perspective of mechanism implementation into the following classes, each suitable for different depths of reflection and scenario requirements:

1. **Implicit Reflection in the ReAct Paradigm**: In the "Think-Act-Observe" loop, after observing the tool return results each time, the model naturally evaluates previous actions in the next round of "Thinking"—whether expected effects were achieved and whether strategies need adjustment. This reflection is embedded within the normal reasoning flow without requiring extra explicit steps.
2. **Explicit Self-Critique**: After producing an initial answer, the model plays the role of a "critic" to re-examine its output, explicitly pointing out potential issues (logical loopholes, factual errors, omitted considerations), and then generates a revised version based on the critique. This method uses role-switching prompt design ("Now please act as a reviewer to inspect the above answer") to trigger deeper reflection.
3. **Reflection Based on External Feedback**: The model's reflection does not rely solely on self-judgment, but incorporates external signals (code execution error messages, tool invocation failure reasons, user correction feedback) as the basis for reflection. This type of reflection is generally more reliable than pure introspective reflection due to support from objective error evidence.
4. **Multi-Candidate Comparative Reflection**: After generating multiple candidate answers, the model compares and analyzes the pros and cons of each candidate, discovering issues in a single answer that are hard to detect by mutual comparison. Essentially, it leverages "diversity" as an auxiliary signal for reflection.
5. **Structured Checklist Reflection**: Predefines a checklist of inspection items for specific task types (e.g., "whether code handles edge cases", "whether concurrency safety is considered"), guiding the model to self-check item by item according to the checklist. This is more systematic and less prone to omissions compared to open-ended requests like "please check your answer."

</details>

**5. Why is CoT (Chain-of-Thought) Effective?**

<details>
<summary>💡 Show Answer</summary>

CoT (Chain-of-Thought) can significantly enhance a model's performance on complex reasoning tasks. Its effectiveness stems from two interconnected core mechanisms.

1. **Increasing Effective Computational Depth**: The computational depth of the Transformer architecture in a single forward pass is fixed (determined by the number of layers), consuming the same amount of "thinking" regardless of problem difficulty. By prompting the model to generate intermediate reasoning steps, CoT decomposes complex computation—originally compressed into a single forward pass—into a continuous generation process across multiple tokens. Generating each reasoning token is equivalent to executing an additional full forward pass, dynamically increasing the model's effective computational depth for the problem, thereby allocating computational resources proportional to the problem's complexity.
2. **Explicating Intermediate States and Enabling Self-Correction**: Without CoT, the model's "thinking process" is entirely hidden within internal activation states, inaccessible to external observation or to the model's own "retrospection" during subsequent generation. CoT transforms this implicit process into explicit natural language text, allowing every subsequent token to "see" and reference preceding reasoning steps. If an error occurs in an intermediate step, subsequent steps have the opportunity to detect contradictions based on the explicit text and correct them—a capability for self-reference and self-correction that does not exist in implicit reasoning.
3. **Synergy of the Two Mechanisms**: The increase in computational depth provides "sufficient thinking space" for complex problems, while explication offers a "verifiable and correctable" medium for this space. Together, these two mechanisms explain why simply prompting the model to "think step by step" yields substantial accuracy improvements on complex tasks such as mathematical reasoning and logic puzzles.

</details>

**6. Does Chain-of-Thought (CoT) Truly Enhance Model Reasoning Ability?**

<details>
<summary>💡 Show Answer</summary>

This is a subject of ongoing debate in academia. Understanding its nuances helps form a proper perspective on the true role of CoT, rather than blindly assuming that "CoT is a silver bullet."

1. **Evidence supporting "CoT indeed enhances reasoning ability"**: On tasks requiring step-by-step decomposition—such as mathematical word problems, multi-step logical reasoning, and code generation—CoT achieves significant and consistent accuracy gains compared to direct answer generation (improving performance by up to 20--30 percentage points on certain benchmarks). The magnitude and consistency of this gain are difficult to explain entirely by "better utilization of existing capabilities."
2. **Evidence supporting "CoT merely elicits rather than creates capability"**: Research shows that the effectiveness of CoT heavily depends on whether the model was exposed to abundant step-by-step reasoning text during pre-training. Rather than "learning new reasoning paradigms," the CoT format activates implicit capabilities already present in the model that were underutilized in direct Q&A mode. For novel problem types lacking corresponding reasoning patterns in the training data, the performance boost from CoT drops significantly.
3. **A More Accurate Conceptual Framework**: CoT is neither "creating new capabilities out of thin air" nor a "simple formatting trick." Instead, by explicating intermediate steps, it provides the model with **additional computational resources** (more tokens mean more forward passes) and **opportunities for self-correction** (intermediate steps can be examined and revised). This effect is more accurately described as "unlocking and optimizing the utilization efficiency of existing capabilities," rather than "endowing completely new reasoning abilities."
4. **Practical Implications**: The practical takeaway of this debate is that one should not expect CoT to enable models to solve problems that lie beyond their knowledge boundaries or fundamental reasoning limits. The optimal use cases for CoT are tasks where the model "possesses the capability but is prone to errors during direct output," yielding the highest cost-effectiveness.

</details>

**7. Why Does Using CoT Sometimes Degrade Performance?**

<details>
<summary>💡 Show Answer</summary>

CoT is not beneficial in all scenarios. Understanding the mechanisms behind its negative effects is crucial for using CoT effectively.

1. **The "Overthinking" Trap in Simple Tasks**: For simple tasks where the model can directly yield correct answers based on intuition or memory (e.g., commonsense Q&A, simple classification), forcing the model to generate a reasoning process is equivalent to demanding "justifications" rather than "answers." In attempting to construct a plausible reasoning chain, the model may introduce irrelevant or contradictory intermediate arguments, leading itself astray to an incorrect final conclusion.
2. **Error Accumulation in Reasoning Paths**: In a long reasoning chain, each step carries a probability of introducing slight deviations, which accumulate over sequential generation. Even if a correct path to the final answer exists, the model may derail at an intermediate step, resulting in a final output inconsistent with the ground truth. The more reasoning steps required, the larger the risk window for error accumulation.
3. **Noise Injection and Attention Dilution**: Overly long reasoning processes introduce substantial exploratory text unrelated to the final conclusion. When generating the final answer, this noise consumes part of the model's attention capacity, potentially diluting focus on critical information.
4. **Negative Cases in Specific Task Types**: Studies have shown that on certain highly perceptive or intuition-dependent tasks (e.g., visual commonsense reasoning, specific sentiment analysis scenarios), CoT degrades accuracy. The optimal solutions for such tasks typically do not involve explicit step-by-step reasoning; the "rationalization" process forcibly introduced by CoT disrupts the accuracy of intuitive judgment.
5. **Practical Advice**: Whether to deploy CoT should be determined based on task complexity: for straightforward tasks, request concise answers directly; for complex tasks requiring multi-step reasoning, enable CoT. Use A/B testing to verify the empirical effect of CoT on specific tasks rather than assuming CoT is universally beneficial.




![Illustration of Why Does Using CoT Sometimes Degrade Performance?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig06_using_cot_sometimes_degrade.png)

</details>

**8. What is the Role of Chain-of-Thought in Agent Planning?**

<details>
<summary>💡 Show Answer</summary>

The role of CoT in Agent planning:
1. **Assisting Task Decomposition**: Prompts the model to think step by step about "what prerequisites and steps are needed to achieve the goal";
2. **Tool Selection Reasoning**: Evaluates "which tool is best suited for the current step and what the parameters should be";
3. **Error Anticipation**: Reasons prior to execution about "why this step might fail and how to prevent it";
4. **Result Verification**: Uses CoT self-scrutiny before producing the final answer to check "whether this answer truly addresses the user's question";
5. **Thought Field**: The Thought field in the ReAct framework is fundamentally CoT, rendering the reasoning process visible and facilitating debugging and auditing.

</details>

**9. Why is "Long Thinking" Effective on Complex Tasks?**

<details>
<summary>💡 Show Answer</summary>

"Long thinking" essentially allows the model to allocate more computation (tokens) to "think" before producing the final answer, which is equivalent to converting the reasoning process from implicit to explicit.

Reasons for effectiveness:
1. Overcoming Working Memory Limitations: Explicit CoT provides a "scratchpad."
2. Self-Correction of Errors: Reflection steps allow the model to verify its intermediate conclusions.
3. Computational Sufficiency: Solving certain problems requires multiple steps, which cannot be computed within a single forward pass.

</details>

**10. What is Test-Time Scaling?**

<details>
<summary>💡 Show Answer</summary>

Test-Time Scaling (scaling at inference time) is an important trend in the recent LLM field. Its core idea is to move beyond relying solely on scaling during the training phase, and instead invest more compute during the inference (generation) phase to trade for better performance. This paradigm has been validated as an effective path by OpenAI's o1/o3 series and Claude's Extended Thinking.

1. **Best-of-N (BoN)**: Generates $N$ independent candidate answers for the same prompt (via sample diversity) and uses a Verifier or Reward Model to score each candidate, selecting the highest-scoring one as the final output. It is simple and effective, but compute cost grows linearly with $N$.
2. **Beam Search**: Rather than screening after full answers are generated, Beam Search maintains multiple candidate reasoning paths (beams) during inference. At each step, next-token candidates across all paths are scored, retaining the top-$K$ scoring paths to expand further, and ultimately selecting the optimal answer from the surviving paths. It is more efficient than BoN because low-quality paths can be pruned early.
3. **Monte Carlo Tree Search (MCTS)**: Models the reasoning process as a tree search problem, incrementally constructing a search tree through the cycle of "Selection--Expansion--Simulation--Backpropagation" to balance exploration (trying new paths) and exploitation (deepening known superior paths). It is better suited than Beam Search for complex reasoning tasks requiring long-horizon planning and multi-step verification.
4. **Extended Thinking**: Directly increases the budget of "thinking tokens" available to the model before generating the final output, giving the model more scope for chain reasoning, self-checking, and correction without requiring external search or sampling mechanisms. This is currently the most widely deployed approach in industry due to its low implementation complexity and cost.
5. **Core Trade-off**: Test-Time Scaling essentially trades "inference compute cost" for "answer quality." It requires dynamically allocating the inference compute budget based on task difficulty—over-allocating on simple questions is wasteful, while under-allocating on complex ones yields subpar quality. Adaptive compute budget allocation is an active research hotspot.

</details>

**11. How is the Reflection (Self-Correction) Mechanism Implemented in Engineering?**

<details>
<summary>💡 Show Answer</summary>

Engineering implementations of reflection mechanisms can be divided into three levels, ranging from simple to complex, depending on the accuracy requirements and cost budget of the application scenario.

1. **Prompt-based Reflection (Simplest, zero extra training cost)**: Instructs the model in the system prompt or single API call to follow the workflow: "generate initial answer $\rightarrow$ adopt a reviewer role to inspect the answer $\rightarrow$ produce a revised version based on inspection results." This can be completed within a single call (the model completes the full cycle in one generation pass) or split into two independent LLM calls (where the second call specifically reviews the first output, allowing the reviewer prompt to be more focused and strict).
2. **External Verifier-Triggered Reflection (Moderate complexity, relies on external signals)**: Introduces an independent verification mechanism—such as deterministic rule checkers (checking if code execution throws errors or if output complies with a schema), specially trained Reward Models/Verifiers, or a stronger LLM serving as judge. The Verifier provides concrete, actionable feedback (rather than vague assertions like "something is wrong"), which is injected as structured input into the next prompt round to trigger targeted revision. This approach is more reliable than purely prompt-based reflection because reflection relies on objective external verification rather than the model's potentially biased self-judgment.
3. **Internalizing Reflection Capabilities During Training (Most complex, best performance, but highest cost)**: Constructs training samples during SFT or RLHF that explicitly contain the full "generation $\rightarrow$ reflection $\rightarrow$ correction" trajectory, training the model to autonomously perform effective self-reflection without needing explicit external prompts. This is the core training paradigm adopted by cutting-edge models (e.g., OpenAI o1, Claude's Extended Thinking), where the model naturally produces internalized reflection behaviors such as "Wait, let me double-check this" during inference.
4. **Practical Selection Strategy**: Use prompt-based reflection in prototyping to quickly validate feasibility. In production environments with high accuracy requirements, prioritize implementing external Verifier mechanisms (controllable cost with significant performance gains). Consider internalizing reflection capabilities during training only when self-reflection is a core product differentiator and sufficient training compute is available.




![Illustration of How is the Reflection (Self-Correction) Mechanism Implemented in Engineering?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig07_reflection_mechanism_implemented_engineering.png)

</details>

**12. Is the Reflection (Self-Correction) Mechanism Truly Effective?**

<details>
<summary>💡 Show Answer</summary>

The effectiveness of reflection mechanisms depends heavily on the presence of objective external verification signals. This serves as the primary criterion for determining when to employ reflection and when to use it with caution.

1. **Effective Scenarios—Objectively Verifiable Feedback**: When reflection is grounded in concrete, objective, and verifiable information (e.g., error tracebacks from code execution, failing unit test cases, structured database query errors, factual contradictions returned by search engines), the model explicitly knows "what error occurred and where." The direction for reflection and revision is unambiguous, and the efficacy of reflection mechanisms in such settings is widely validated (e.g., the substantial success rate improvement in "execute--error--fix" loops in code generation).
2. **Constrained Scenarios—Pure Introspective Reflection**: When reflection relies entirely on the model's subjective judgment of "whether this answer is correct" (without external verification signals, simply prompting "please check if your response is correct"), the model is evaluating the answer using the same cognitive framework that produced it. If the initial error stemmed from a knowledge blind spot or systematic bias, introspective reflection is unlikely to uncover the real issue—the model "does not know what it does not know."
3. **Mechanisms of Reflection Introducing New Errors**: In the absence of external verification, the model may engage in over-correction simply because it was "instructed to find mistakes." Even if the original answer was correct, the model might invent "issues" to satisfy the reflection prompt, making unnecessary or incorrect modifications. This phenomenon of "reflecting for the sake of reflecting" is common in practice.
4. **Practical Recommendations**: When designing reflection mechanisms, prioritize finding or constructing objective verification signals (even indirect signals, such as using an independent LLM instance to evaluate from a different perspective, are more reliable than asking the same model to review its own output). For subjective tasks that inherently lack external verification, evaluate the net gains of reflection cautiously and run A/B tests to verify whether extra reflection steps genuinely improve output quality, rather than assuming that "thinking one step further is always better."

</details>

**13. Why is the Performance of Tree-of-Thought (ToT) Unstable?**

<details>
<summary>💡 Show Answer</summary>

Tree-of-Thought (ToT) aims to enhance problem-solving for complex tasks by constructing a multi-branch reasoning tree and performing search algorithms. However, its practical performance remains unstable, primarily due to inherent limitations in evaluation mechanisms and search costs.

1. **Reliability Bottleneck of the Evaluator**: The core of ToT lies in evaluating the value of each reasoning branch to decide which paths to expand and which to prune. This evaluation is typically performed by the LLM itself (evaluating the quality of its own generated branches). However, the model's meta-cognitive ability to judge "which reasoning path is more likely to lead to the correct answer" is inherently limited—a task often harder than answer generation itself. Inaccurate evaluations directly undermine the search effectiveness of ToT.
2. **Cascading Impact of Evaluation Errors**: If the evaluator mistakenly assigns a low score to a correct branch (causing it to be pruned) or a high score to a flawed branch (wasting computational resources), the efficiency and correctness of the entire search process are compromised. Unlike single-path CoT, an evaluation error in ToT affects the entire search tree's structure and resource allocation rather than just a single reasoning chain.
3. **Cost Escalation from Search Space Explosion**: As reasoning depth increases, the number of branches at each layer grows exponentially. Even with pruning strategies, thorough exploration demands computational resources far exceeding single-path CoT (token consumption and latency are typically several to tens of times higher than CoT). For cost-sensitive production applications, this overhead is often prohibitive, constraining ToT's practical deployment.
4. **Hyperparameter Sensitivity**: ToT's performance is sensitive to hyperparameters such as branch width (breadth), search depth, and pruning thresholds. Optimal hyperparameter configurations vary significantly across different task types, and the lack of universal auto-tuning methods necessitates extensive empirical tuning for specific tasks, contributing to engineering instability.
5. **Limitations in Applicable Scenarios**: ToT performs well on tasks with clear intermediate state evaluation criteria (e.g., mathematical puzzles, the Game of 24) because branch evaluation is relatively objective. However, on open-ended or highly subjective tasks (e.g., creative writing, complex decision-making), the absence of clear evaluation standards prevents ToT from leveraging its advantages, leading to further instability.

</details>

**14. In Agent design, "planning capability" is crucial. What are the mainstream methods used to endow LLMs with planning capabilities?**

<details>
<summary>💡 Show Answer</summary>

Mainstream methods for endowing LLMs with planning capabilities, ordered from simple to complex:

1. **Prompt-based Implicit Planning**: Chain of Thought (CoT) guides linear reasoning via "Let's think step by step"; ReAct combines CoT with action, making planning a dynamic process.
2. **Search-based Explicit Planning**: Tree of Thoughts (ToT) constructs a "thought tree" and explores multiple parallel paths via BFS/DFS, allowing evaluation and backtracking; Graph of Thoughts (GoT) further permits path merging and looping to support more complex iterative refinement.
3. **Task Decomposition-based Planning**: Trains or prompts the LLM to act as a "Planner," explicitly decomposing the main task into a dependency graph or step list, which is then executed step by step by an "Executor."




![Illustration of In Agent design, "planning capability" is crucial. What are the mainstream methods used to endow LLMs with planning capabilities?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig08_agent_design_planning_capability.png)

</details>

**15. What are the Deep Design Challenges of Search-Augmented Reasoning?**

<details>
<summary>💡 Show Answer</summary>

1. **Search Trigger Timing**: When should search be initiated? Searching too frequently wastes tokens, while searching too infrequently leads to insufficient information.
2. **Noise in Search Results**: Retrieved documents may be irrelevant or mutually contradictory.
3. **Interleaving Search and Reasoning**: In the ReAct framework, loops may extend indefinitely.
4. **Latency Issues**: Each search call introduces hundreds of milliseconds of latency.
5. **Context Length Management**: Context length grows significantly after multiple search rounds.

</details>

**16. How Can Reasoning Capabilities Be Distilled from Large Models to Small Models?**

<details>
<summary>💡 Show Answer</summary>

Reasoning distillation transfers the reasoning process of a Teacher model (e.g., GPT-4, Claude) to a smaller Student model. The core approaches include:

1. **Process Supervision Distillation**: Rather than distilling only final answers, this distills the reasoning trajectories (Chain of Thought). The Teacher generates trajectories with CoT, and the Student undergoes SFT on these trajectories.
2. **Step-level KD**: Computes token distributions for each reasoning step of the Teacher, training the Student to match the Teacher's step distributions.
3. **On-policy Data Generation**: Allows the Student to generate its own reasoning trajectories, which are scored by the Teacher, and the Student then performs RL on its own rollouts.
4. **Rejection Sampling Fine-tuning (RFT)**: The Teacher or Verifier filters correct reasoning trajectories generated by the Student, and the Student is fine-tuned strictly on these correct trajectories.

</details>

**17. How to Systematically Diagnose and Locate Issues When Reasoning Fails?**

<details>
<summary>💡 Show Answer</summary>

Diagnostic framework for classifying reasoning failures:

**Step 1: Categorize Failure Types**
- Formatting Failure: Output format fails to comply with requirements.
- Process Failure: Errors occur in reasoning steps, distinct from knowledge deficits.
- Knowledge Failure: Lack of necessary factual or conceptual knowledge.
- Comprehension Failure: Misinterpretation of the problem statement.

**Step 2: Targeted Analysis**
- Formatting Failure: Inspect formatting instructions in the prompt; strengthen formatting SFT data.
- Process Failure: Audit CoT trajectories step by step; score steps using a Process Reward Model.
- Knowledge Failure: Check if supplementing knowledge via RAG resolves the issue.
- Comprehension Failure: Test using alternative prompt rephrasings.

</details>

**18. What Are Common Strategies for Task Decomposition?**

<details>
<summary>💡 Show Answer</summary>

1. **Sequential Decomposition**: Divides the task into ordered steps where preceding outputs serve as inputs to subsequent steps; suitable for tasks with clear dependencies;
2. **Parallel Decomposition**: Assigns independent subtasks concurrently to multiple agents or tools and aggregates results; suitable for parallelizable tasks;
3. **Hierarchical Decomposition**: Recursively splits main tasks $\rightarrow$ subtasks $\rightarrow$ sub-subtasks; suitable for highly complex tasks;
4. **Conditional Decomposition**: Dynamically determines subsequent steps based on intermediate results; suitable for exploratory tasks with uncertain outcomes;
5. **Map-Reduce**: Batch-processes homogeneous subtasks (Map) and aggregates the results (Reduce); suitable for large-scale data processing.

</details>

**19. What Scenarios Are Suitable for Sequential Decomposition vs. Parallel Decomposition?**

<details>
<summary>💡 Show Answer</summary>

**Sequential Decomposition is suitable for**: Strong dependencies between steps (step B requires output from step A); task outputs requiring iterative refinement (generate $\rightarrow$ review $\rightarrow$ edit); resource constraints prohibiting parallel execution; operations strictly requiring ordered execution (e.g., read before write).

**Parallel Decomposition is suitable for**: Subtasks that are mutually independent with no data dependencies; scenarios requiring reduced overall execution latency; homogeneous subtasks (e.g., batch processing multiple files); multi-perspective verification (where multiple agents analyze independently before aggregation). In practical systems, the two patterns are often combined: completing prerequisite steps sequentially before executing independent subtasks in parallel.

</details>

**20. What is Hierarchical Planning? What Are Its Advantages and Disadvantages?**

<details>
<summary>💡 Show Answer</summary>

Hierarchical Planning recursively decomposes a complex task into multiple levels: a high-level Orchestrator creates the global plan, mid-level Agents handle sub-plans, and low-level Agents execute concrete operations. Advantages: divide-and-conquer design, where each level manages its own complexity; easy parallelization, allowing different subtrees to execute concurrently; modularity, enabling subtask reuse. Disadvantages: high inter-level communication overhead with latency increasing alongside layer depth; complex coordination where errors can propagate across levels; challenging debugging requiring layer-by-layer troubleshooting; over-hierarchical structure introduces unnecessary complexity.

</details>

**21. What is the Plan-and-Execute Pattern? How Does It Differ from ReAct?**

<details>
<summary>💡 Show Answer</summary>

**Plan-and-Execute**: First, a Planning Agent generates a complete plan (a list of all steps), which is then executed step-by-step by an Execution Agent, keeping the plan fixed (or revised only when necessary) during execution. **ReAct**: Planning and execution alternate step-by-step; after each step, the next action is determined based on observations, without a predefined full plan. Differences:
1. ReAct is more flexible and can respond to unexpected results in real time; Plan-and-Execute is more structured and better suited for tasks with predictable steps;
2. In ReAct, the LLM must "reason about the next step" at every turn, leading to high token consumption; Plan-and-Execute plans once, resulting in lower token consumption during execution;
3. Complex tasks typically favor Plan-and-Execute, as ReAct tends to "lose direction" when step counts are large.




![Illustration of What is the Plan-and-Execute Pattern? How Does It Differ from ReAct?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig09_pattern_it_differ_from.png)

</details>

**22. What is ReWOO (Reasoning WithOut Observation)? What Problems Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

ReWOO is an agent framework that decouples planning from execution: a Planner LLM first generates a complete execution plan (including all tool call sequences), tool calls are then executed in batch, and finally a Solver LLM synthesizes all outputs into the final answer. Problems solved:
1. **Token Waste in ReAct**: ReAct injects full execution history and observations into the context at each step, causing token consumption to scale quadratically with the number of steps;
2. **Latency Issues**: Tool calls in ReWOO can be executed in parallel, significantly reducing overall latency;
3. **Efficiency**: On multi-step tasks, ReWOO reduces token consumption by 3--4$\times$ while maintaining comparable accuracy. It is well-suited for tasks where step dependencies can be determined beforehand.

</details>

**23. How to Design a High-Quality Planner Module?**

<details>
<summary>💡 Show Answer</summary>

Core components of Planner system design:

Input Layer: User goal parsing and disambiguation, context state, and descriptions of available tools/actions.

Core Capabilities:
1. **Task Decomposition**: Decomposes high-level goals into sequences of executable subtasks and identifies inter-task dependencies.
2. **Plan Representation**: Represents plans using structured formats (JSON/YAML), supporting DAG structures to express parallel tasks.
3. **Feasibility Verification**: Checks whether each step in the plan corresponds to available tools or capabilities.
4. **Dynamic Replanning**: Triggers local replanning when execution failures occur or environment state changes.

</details>

**24. How to Implement DAG Planning to Enable Agents to Execute Independent Subtasks in Parallel?**

<details>
<summary>💡 Show Answer</summary>

Core principles of Directed Acyclic Graph (DAG) planning:
1. **Dependency Analysis**: Analyzes data dependencies between subtasks, builds a dependency graph, and applies topological sorting to establish execution order.
2. **DAG Representation**: Represents task dependency relationships in a structured format.
3. **Parallel Execution Engine**: Uses asynchronous execution to manage concurrent tasks.
4. **Failure Handling**: When a node fails, cancels all downstream tasks dependent on that node and triggers local replanning.

</details>

**25. How to Mitigate Planning Drift in Agents?**

<details>
<summary>💡 Show Answer</summary>

Planning drift refers to the phenomenon where an Agent gradually strays from its original goal during execution.

Preventing Drift:
1. **Goal Anchoring**: Reiterate the original goal in the system prompt of every LLM call.
2. **Step Verification**: Prior to executing each step, require the model to verify whether the step directly contributes to achieving the goal.
3. **Milestone Inspection**: Establish key milestones and perform global goal alignment checks upon reaching each milestone.
4. **Maximum Step Limits**: Enforce a maximum step count limit for execution.

</details>

**26. How Does an Agent Handle Unforeseen Intermediate Results During the Planning Phase?**

<details>
<summary>💡 Show Answer</summary>

1. **Dynamic Replanning**: When encountering unexpected results during execution, the LLM re-evaluates the remaining plan and modifies subsequent steps as necessary;
2. **Conditional Branching**: Pre-defines multiple execution paths during planning and selects the appropriate branch based on intermediate outcomes;
3. **Human-in-the-Loop Triggers**: Pauses execution and requests user guidance when intermediate results fall outside expected boundaries;
4. **Partial Execution Rollback**: Reverts to the decision bifurcation point and replans if assumptions from earlier steps are invalidated;
5. **Fault-Tolerant Design**: Incorporates "worst-case handling" into each step's plan to mitigate the blast radius of unexpected events;
6. **Key Principle**: An Agent's planning must be dynamic and adaptable to execution feedback, rather than a rigid, predefined script.

</details>

**27. How to Enable an Agent to Dynamically Revise Its Original Plan During Execution?**

<details>
<summary>💡 Show Answer</summary>

1. **Checkpoint Evaluation**: After completing each major step, have the LLM evaluate whether "current progress aligns with the plan and whether downstream steps require adjustment";
2. **Exception-Triggered Replanning**: Automatically trigger partial replanning when tool calls fail or return unexpected outputs;
3. **Goal Alignment Auditing**: Periodically prompt the LLM to compare the current state with the original goal to check for drift;
4. **Maintaining Revision History**: Record reasons for each plan revision to prevent the Agent from getting stuck in endless replanning loops;
5. **Restricting Replanning Scope**: Revise only remaining downstream steps, keeping completed steps intact (unless fundamental errors are uncovered);
6. **User Confirmation**: Notify the user and obtain explicit confirmation before executing major plan modifications.

</details>

**28. How Does an Agent Determine Whether a Subtask Has Been Completed?**

<details>
<summary>💡 Show Answer</summary>

Determination Methods:
1. **Explicit Completion Criteria**: Define clear completion criteria for each subtask during planning (e.g., "when the query returns non-empty results" or "when the file is written successfully");
2. **Tool Return Codes**: Use success/failure status codes returned by tools as completion indicators;
3. **LLM Evaluation**: Have the LLM inspect tool outputs to judge whether expected objectives are met;
4. **Output Validation**: Perform schema validation on structured outputs, considering the task complete upon passing;
5. **Important Principle**: Completion determination should rely on objective signals (tool outputs) rather than subjective LLM judgments to minimize hallucination risks.

</details>

**29. How to Avoid "Analysis Paralysis" Caused by Over-Refinement in Agent Planning?**

<details>
<summary>💡 Show Answer</summary>

1. **Time/Token Budget**: Set a token limit during the planning stage (e.g., maximum 500 tokens); execute the current best plan if the limit is exceeded;
2. **Maximum Step Cap**: Limit the plan to at most $N$ steps to enforce high-level abstraction;
3. **Eager Execution**: Begin execution as soon as a step becomes executable, rather than waiting for a perfect global plan;
4. **Minimum Viable Plan**: Formulate a "minimum viable plan to reach the goal" first, augmenting it dynamically during execution;
5. Emphasize in system prompts: "Prioritize action and learn from execution outcomes rather than spending excessive compute on planning."

</details>

**30. How Should an Agent Handle Goal Conflicts During Planning?**

<details>
<summary>💡 Show Answer</summary>

Goal conflict is a central problem in multi-objective planning. Handling mechanisms depend on conflict types and application contexts.

Categorization of Conflict Types: Resource conflicts, temporal conflicts, and value conflicts.

Resolution Strategies:
1. **Priority Ranking**: Assign explicit priorities to each goal, resolving conflicts by trading off according to priority.
2. **Human-in-the-Loop**: Pause planning and request user clarification on priorities when critical goal conflicts are detected.
3. **Pareto Optimal Search**: Search for Pareto-optimal trade-offs among competing objectives.
4. **Constraint Satisfaction**: Convert lower-priority goals into hard constraints.

</details>

**31. What is Multi-Path Reasoning and What is Its Purpose?**

<details>
<summary>💡 Show Answer</summary>

Multi-Path Reasoning (also known as Self-Consistency) is a technique that enhances reasoning reliability via "multiple sampling + voting." Its core idea is to counteract the randomness and uncertainty of single-pass inference through statistical aggregation.

1. **Basic Operating Principle**: For the same question, high-temperature sampling is used to independently generate complete reasoning chains multiple times (typically 5--20 times). Although generated paths may vary due to sampling randomness, if the model possesses strong confidence in the correct solution, multiple independent runs tend to converge to the same final answer, even across distinct intermediate paths.
2. **Voting Mechanism (Self-Consistency)**: Final answers are extracted from all generated reasoning paths, answer frequencies are tabulated, and the most frequent answer is selected as output. This majority voting scheme achieves notable improvements on tasks with deterministic ground truths (e.g., math reasoning, logic puzzles) because erroneous paths tend to be diverse and dispersed, whereas correct paths consistently converge.
3. **Underlying Logic of Consistency Verification**: If multiple logically independent reasoning paths generated by the model arrive at the same conclusion, it provides indirect evidence supporting the answer's correctness—akin to how agreement among multiple independent observers is more trustworthy than a single report. Conversely, highly dispersed answers across paths indicate low model confidence, providing a valuable uncertainty signal.
4. **Evaluation Models Beyond Simple Voting**: For open-ended tasks lacking definitive ground truth (where frequency counting is inapplicable), an independent evaluation model (Verifier/Reward Model) can be trained or leveraged to score candidate reasoning paths, selecting the highest-scoring path rather than relying on frequency statistics.
5. **Cost-Benefit Trade-off**: The compute cost of multi-path reasoning scales linearly with sampling iterations ($5\times$ sampling implies $5\times$ token consumption). The iteration count should be determined by task criticality and accuracy demands: high-risk decision-making can justify higher compute costs for maximum reliability, while standard tasks should limit sampling to 3--5 passes to balance cost and performance.

</details>

**32. Why Are Agents Prone to "Overthinking"?**

<details>
<summary>💡 Show Answer</summary>

An Agent's "overthinking" problem is a negative side effect caused when planning capabilities and deliberative designs are pushed to extremes. Understanding its underlying causes helps find a balance between an Agent's "cautiousness" and "decisiveness."

1. **Overly Deliberate Inclination in System Prompts**: When a system prompt contains numerous caution-emphasizing instructions such as "please consider carefully before executing" or "ensure full understanding of the task before acting," the model tends to apply this deliberative mode across all tasks. Even for tasks simple enough to not require complex analysis, unnecessary multi-turn self-confirmations and re-evaluations will be triggered.
2. **Blurred Boundaries Between Planning and Execution**: If the Agent lacks a clear mechanism to distinguish between "complex tasks requiring detailed planning" and "simple tasks that can be directly executed," all tasks will follow the exact same process (containing extensive review and confirmation steps), resulting in simple tasks being overprocessed under this "one-size-fits-all" workflow.
3. **Abuse of Self-Reflection Mechanisms**: Reflection/self-checking mechanisms introduced to improve task quality may be triggered indiscriminately on all tasks if reasonable trigger conditions are not configured (such as enabling them only in complex or high-risk tasks). Over-auditing simple tasks not only wastes resources but may also introduce new uncertainties due to over-deliberation (the previously mentioned phenomenon where "reflection introduces new errors").
4. **Specific Manifestations of Decision Paralysis**: Typical symptoms of overthinking include: repeatedly confirming already explicit requirements, requiring multiple validations even for low-risk operations, repeatedly weighing options among multiple equally feasible solutions without executing, and generating massive analysis text irrelevant to the final decision. These behaviors not only waste tokens and time but also make the Agent appear verbose and indecisive in terms of user experience.
5. **Solutions**: Establish a fast classification mechanism for task complexity where simple tasks follow a streamlined process (without triggering deep planning and multi-turn reflection), and only genuinely complex or high-risk tasks enable the full deliberative process. Clearly define evaluation criteria in the system design regarding "what degree of uncertainty requires further analysis versus what degree allows direct action based on current information," thereby avoiding the indiscriminate application of deliberation.




![Illustration of Why Are Agents Prone to "Overthinking"?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig10_agents_prone_overthinking.png)

</details>

**33. Why Are Agents Prone to Making Faulty Plans?**

<details>
<summary>💡 Show Answer</summary>

Planning relies on the model's understanding of the task and environment. If there are biases in understanding (such as not understanding the actual limitations of a tool or misunderstanding task constraints), the generated plan is built on false assumptions right from the start. The core reason is an inaccurate "world model" upon which planning is based—there are discrepancies in the understanding of tool capabilities, environmental constraints, and actual task requirements, combined with a lack of verification mechanisms for these assumptions during planning.

</details>

**34. How Should an Agent Plan Complex Tasks?**

<details>
<summary>💡 Show Answer</summary>

Complex task planning is the part of Agent capabilities that best demonstrates "intelligence." Hierarchical Planning is the core strategy for handling complexity, avoiding falling into "analysis paralysis" or producing unrealistically detailed plans during the planning phase.

1. **Top-Down Hierarchical Structure**: Complex task planning should start at a high level—first identifying several major phases (milestones) required to complete the task, without delving into the specific execution details of each phase. This level of planning should be relatively stable because high-level task decomposition is usually easier to predict accurately than detailed execution steps.
2. **Lazy Refinement**: Specific steps for each phase are not fully planned at the start of the task; instead, they are refined and expanded only when about to enter that phase. The benefit of this is having more accurate contextual information during refinement (the actual execution results of previous phases), leading to higher quality step plans and avoiding the waste of detailed plans for distant phases being invalidated by changing circumstances.
3. **Recursive Decomposition of Complexity**: When a specific phase remains overly complex (the number of steps after refinement is still huge), recursively apply the same hierarchical planning method to that phase—decomposing sub-milestones within it until the step granularity can be directly executed by a single tool call or a single Agent.
4. **Alternation of Planning and Execution**: Planning complex tasks is not completed all at once before execution; rather, it is an alternating cycle of "Plan $\rightarrow$ Execute first phase $\rightarrow$ Refine next phase plan based on results $\rightarrow$ Execute $\rightarrow \dots$". This alternating pattern ensures that planning is always built upon the latest factual information, substantially improving the success rate of complex tasks.
5. **Balancing Planning Depth and Time Budget**: Set a reasonable time/token budget for planning to avoid consuming excessive resources during the planning stage—high-level plans should be concise (a few milestones described in a few sentences), leaving detailed thinking for immediate upcoming steps, which aligns with the concept of "rolling horizon planning."

</details>

**35. How Should an Agent Plan Long-Term Tasks?**

<details>
<summary>💡 Show Answer</summary>

The core challenge facing long-term task planning is "future uncertainty"—the longer the task, the harder it is for the initial plan to accurately predict later steps. Forcing detailed long-term plans will instead lead to frequent failures during execution; rolling horizon planning is the best practice to resolve this contradiction.

1. **Core Concept of Rolling Horizon Planning**: Analogous to the "rolling forecast" financial model—always maintain a detailed execution plan for the near term (e.g., the next 3--5 steps), while keeping only high-level milestones and directions for the long term (beyond 5 steps). As the task progresses and a step is completed, refine the next milestone into concrete steps, keeping the "detailed plan window" continuously covering the immediate 3--5 steps, achieving a planning granularity stratification of "more detailed for nearer steps, more macro for distant steps."
2. **Milestone + Step Stratification**: Planning is split into two layers—the **Milestone Layer** (high-level, defining key outputs of task phases such as "Complete data collection," "Complete model training," and "Complete deployment verification," which are fully determined from the task's start) and the **Step Layer** (low-level, specific execution steps for each milestone, which are detailed only prior to entering that milestone). Separating the two layers keeps distant directions stable while keeping near-term execution flexible.
3. **Context-Aware Dynamic Updates**: After completing each step, adjust the subsequent "near-term step plan" based on the latest actual execution results and current system state, rather than mechanically executing the initial plan. If a milestone is found to no longer be feasible due to changing circumstances, promptly notify the user to re-confirm goals, instead of waiting until after several steps of execution to discover the direction was wrong.
4. **Milestone Completion Confirmation**: Upon completing each milestone, perform a "global alignment check"—whether current task progress remains aligned with the final goal, and whether subsequent milestone definitions need adjustment. These checkpoints serve as the most natural HITL (Human-in-the-Loop) nodes, enabling users to confirm directions at critical phases.

</details>

**36. How Should an Agent Handle Planning for Multi-Objective Tasks?**

<details>
<summary>💡 Show Answer</summary>

Multi-objective task planning requires the Agent to handle dependencies, conflicts, and priority relationships among multiple goals while pursuing them simultaneously, making it a much more complex problem than single-objective planning.

1. **Goal Parsing and Priority Ranking**: First, explicitly articulate the user's multi-objective requirements—list all sub-goals that need to be achieved and confirm (or infer from context) the priority of each goal with the user. Priority determines trade-off directions when goals conflict: for instance, when "speed" conflicts with "quality," which one takes precedence is dictated by explicit priority rather than subjective judgment by the Agent.
2. **Goal Dependency Graph Modeling**: Identify prerequisite dependencies between goals (Goal B must start only after Goal A is completed) and parallel relationships (Goal C and Goal D can run concurrently). Construct a goal dependency graph and plan execution paths in topological order to maximize parallelism and shorten total time.
3. **Goal Conflict Detection and Resolution**: Actively detect mutual exclusivity among goals—resource conflicts (two goals both requiring exclusive use of the same resource), value conflicts (inherent contradictions such as "reducing costs" vs. "improving service quality"), and temporal conflicts (two goals requiring the same operation to be completed at different times). When conflicts are detected, resolve them based on priority; if autonomous resolution is impossible, pause and consult the user.
4. **Pareto Optimal Search**: For situations where trade-offs exist between goals rather than complete mutual exclusivity, search for "Pareto optimal" solutions where no further improvement can be made on any goal without degrading another. Present multiple Pareto optimal options to the user for selection.
5. **Milestone Setup and Stage Inspections**: Execute multi-objective tasks in stages, evaluating the current achievement status of each goal at every milestone to promptly identify if any goal is lagging behind or needs adjustment, ensuring that all core objectives are ultimately addressed.

</details>

**37. How Does an Agent Perform Replanning?**

<details>
<summary>💡 Show Answer</summary>

Replanning is the Agent's "course-correction" mechanism when encountering execution deviations. Proper replanning not only corrects deviations but also retains valid completed work, avoiding the high cost of starting completely over from scratch.

1. **Conditions Triggering Replanning**: Tool call results differ significantly from expectations (e.g., expected 100 records returned, but only 3 were returned); intermediate steps reveal constraints unconsidered in the initial plan (e.g., discovering rate limits on the target API); environmental state changes (e.g., the target database switched versions); the current step fails and retries are ineffective; or the Agent's self-evaluation indicates the current path cannot reach the goal.
2. **Local vs. Global Replanning**: Not all situations require replanning from scratch. **Local Replanning**: Re-plans only the affected subsequent steps while retaining completed and still-valid steps (e.g., if the first 3 steps succeeded and the 4th failed, only replan the 4th step onwards). **Global Replanning**: Perform full replanning only when foundational assumptions (such as the task goal itself) are overturned, notifying the user to confirm the direction of the new plan.
3. **Inputs for Replanning**: Instead of merely looking at "current failure," comprehensive context is provided to the planning LLM: Original Goal + Completed steps and their actual results + Current true state + Reason for failure (why the current plan failed) + Previously attempted solutions (to prevent choosing the same failed solution after replanning). "Previously attempted solutions" is key to ensuring that replanning does not repeat mistakes.
4. **Limit on Replanning Attempts**: Set a maximum number of replanning attempts (e.g., at most 2 replanning attempts for the same subtask). Once exceeded, stop automatic replanning and submit the full failure history and current state to the user for them to decide whether and how to proceed, preventing the Agent from falling into an endless replanning loop.
5. **Saving Replanning History**: Record the reason, old plan, and new plan for each replanning event to form a "planning decision log." This serves both audit tracking purposes and as valuable material for improving future initial plan quality (analyzing which types of tasks trigger frequent replanning and targeting optimizations accordingly).




![Illustration of How Does an Agent Perform Replanning?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig11_agent_perform_replanning.png)

</details>

**38. How Does an Agent Dynamically Adjust Plans Based on Execution Feedback?**

<details>
<summary>💡 Show Answer</summary>

Dynamically adjusting plans based on execution feedback is the core manifestation of intelligence distinguishing an Agent from a "rigid script." It requires establishing a closed-loop mechanism of "Execution $\rightarrow$ Feedback $\rightarrow$ Evaluation $\rightarrow$ Adjustment."

1. **Structured Comparison of Expected vs. Actual**: Define a structured description of "expected results" for each step in the plan (not just a vague "should succeed," but specifically "expected to return N records with status active"). After step execution, perform a field-level comparison between actual results and expectations, quantifying the degree of discrepancy rather than making a simple binary "right/wrong" judgment.
2. **Discrepancy Thresholds and Response Levels**: Set tiered responses—**Minor Discrepancies** (e.g., returned record count is slightly fewer than expected but within a reasonable range): Log but do not adjust the plan, continue execution; **Moderate Discrepancies** (e.g., key fields are missing from returned results): Trigger local step adjustments and re-evaluate the next 1--2 steps; **Major Discrepancies** (e.g., no results returned at all, or opposite to expected direction): Trigger a whole-plan re-evaluation, possibly requiring a return to the previous decision point.
3. **Adjustment Principle Based on Latest State**: When adjusting subsequent plans, regenerate steps based on the "current actual state" rather than making local "patchwork" modifications on top of the original plan. Patchwork adjustments easily create new inconsistencies due to residual dependence on assumptions from the original plan; replanning from the latest true state, while slightly more costly, is far more reliable.
4. **Gradual Nature of Adjustments**: Dynamic adjustment does not mean drastically modifying plans upon every piece of feedback—distinguish between "noise fluctuations" (normal execution variances) and "signal changes" (situations truly requiring a direction shift). An overly sensitive adjustment strategy leads to frequent plan changes, which degrades execution efficiency and stability.
5. **Traceability of Adjustment History**: Record the triggering reason and before-and-after comparison for every plan adjustment. This serves post-hoc analysis (determining which feedback types most frequently trigger adjustments and whether initial plan quality needs improvement) and prevents the Agent from re-triggering adjustments for similar reasons after an adjustment, avoiding oscillations.

</details>

**39. How Does an Agent Make Decisions Among Multiple Candidate Solutions?**

<details>
<summary>💡 Show Answer</summary>

An Agent making decisions among multiple candidate solutions is essentially a multi-dimensional optimization problem. It requires establishing a systematic evaluation framework rather than relying on the model's "intuition" for a single-pass judgment.

1. **Aligning Evaluation Criteria with Objective Functions**: Evaluation criteria should not be abstractly "which option is better," but should be closely tied to the task's actual objective function—if the task goal is "minimizing completion time," evaluation should focus on each option's expected latency; if the goal is "maximizing accuracy," evaluation should focus on expected quality. Subjective evaluations detached from the objective function easily produce decisions incompatible with actual needs.
2. **Multi-Dimensional Quantitative Scoring**: Deconstruct abstract "good/bad" judgments into quantifiable specific dimensions—expected return (quality/value of task completion), implementation cost (time, tokens, API expenses), risk level (error probability and severity of error consequences), and implementation complexity (number of execution steps and uncertainty). Score each dimension independently to avoid bias brought by a single blanket judgment.
3. **Weight Assignments Reflecting Task Priorities**: Different dimensions hold varying importance for different tasks, requiring weight settings based on specific task scenarios—time-sensitive tasks assign higher weights to the "cost/speed" dimension, while high-risk scenarios assign higher weights to the "risk" dimension. Weight settings should be made explicit during the planning stage rather than decided ad hoc subjectively.
4. **Constraint Satisfaction Prior to Optimization Ranking**: Before scoring and ranking, first use hard constraints to filter out infeasible options (e.g., directly eliminating options that exceed budget or violate safety rules). This prevents improperly set weights from picking an option that violates constraints despite earning a high overall score.
5. **Interpretable Logs of Decisions**: Record the scoring details and final selection rationale for each candidate solution, making the decision process transparent and auditable. This facilitates post-mortem retrospective analysis of "why this option was selected at the time" when issues arise and accumulates experience for future similar decisions.

</details>

**40. How Does an Agent Conduct Exploration in Uncertain Environments?**

<details>
<summary>💡 Show Answer</summary>

Exploration in uncertain environments is the core ability of an Agent to make decisions under incomplete information. Drawing on the classic concept of "exploration vs. exploitation" in reinforcement learning, it effectively balances the cost of information acquisition with the improvement of decision quality.

1. **Incremental Probing Principle**: Before executing high-cost or irreversible operations, prioritize low-cost, reversible exploratory actions to gather more information—use read-only queries instead of direct writes, small-scale testing instead of full execution, and simulation/previews instead of real operations. This "look before you leap" strategy drastically reduces the risk of acting rashly under insufficient information.
2. **Value of Information Assessment**: Before deciding whether to conduct additional exploration, evaluate whether "acquiring more information can significantly improve decision quality"—if current information is already sufficient to support a high-confidence decision, excessive exploration wastes resources; if uncertainty is high and subsequent operational risk is huge, the value of exploration clearly outweighs its cost, and acquiring information should be prioritized.
3. **Dynamic Exploration-Exploitation Balance**: The exploration phase (exploratory queries, gathering feedback) and the exploitation phase (executing decisive actions based on existing information) should dynamically shift according to task progress—leaning towards exploration early in the task when information is insufficient, and gradually shifting toward deterministic execution as information accumulates, avoiding over-probing (delaying execution) when sufficient information already exists.
4. **Feedback-Driven Strategy Adjustment**: Results of each exploratory action should substantially affect subsequent strategies—if exploration results align with expectations, confidence in the current direction can be increased to accelerate progress; if results are unexpected, re-evaluate the overall plan based on new information rather than ignoring exploration results and mechanically continuing according to the original plan.
5. **Budget Control for Exploration Costs**: Set resource caps for the exploration phase (e.g., at most 3 exploratory queries) to prevent the Agent from getting bogged down during exploration and failing to enter the practical execution phase, which also serves as a relevant mechanism to avoid "overthinking."

</details>

**41. How Does an Agent Model Uncertainty?**

<details>
<summary>💡 Show Answer</summary>

Uncertainty modeling enables an Agent to distinguish between "high-confidence judgments" and "doubtful guesses," serving as a foundational capability for achieving reliable decision-making and appropriately triggering human intervention.

1. **Explicit Confidence Output**: Design prompts to require the model, while outputting conclusions, to append a confidence evaluation for that conclusion (e.g., "I am 80% confident that the answer is X") and explain specific factors causing uncertainty (incomplete information, multiple reasonable interpretations, reliance on unverified assumptions). This explicit expression is more reliable than the model's implicit "tone of confidence," though it still relies on the model's self-awareness capability and retains inherent limitations.
2. **Consistency-Based Indirect Estimation (Self-Consistency Method)**: Generate answers via multiple independent samplings (the aforementioned multi-path reasoning method) and observe the degree of consistency in the results—if multiple samples highly converge on the same conclusion, it indicates high confidence (low uncertainty) for that problem; if results are highly dispersed, it shows that the model itself possesses high uncertainty. This approach does not rely on self-reporting by the model; instead, it objectively infers uncertainty from behavioral performance and is usually more reliable than directly asking the model "how certain are you."
3. **Uncertainty Assessment Based on External Verification Signals**: If the task possesses objective verification methods (whether code executes correctly, whether query results align across multiple independent sources), the consistency of verification results can be used as an indirect quantitative metric for uncertainty, rather than relying solely on the model's internal judgment.
4. **Tiered Response to Uncertainty**: Establish clear threshold mechanisms—conclusions with high confidence (e.g., $> 90%$) can be directly adopted to proceed with execution; moderate confidence conclusions trigger extra validation steps (searching for more information, calling other tools for cross-verification); low confidence conclusions pause autonomous decision-making and transition to requesting user clarification or confirmation, rather than proceeding with operations that might cause negative consequences based on unreliable guesses.
5. **Uncertainty Propagation Management**: In multi-step tasks, uncertainty from predecessor steps propagates and accumulates into subsequent steps. It is necessary to track and update the overall task's cumulative uncertainty at every step, triggering human intervention promptly when cumulative uncertainty crosses a threshold, rather than letting uncertainty compound unchecked until issues are discovered at the end of the task.




![Illustration of How Does an Agent Model Uncertainty?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig12_agent_model_uncertainty.png)

</details>

**42. What Is the Biggest Problem Currently Facing Planning?**

<details>
<summary>💡 Show Answer</summary>

The core problem currently facing planning can be summarized as the "gap between planning assumptions and execution realities." This issue spans the entire closed loop from planning design to execution feedback, serving as one of the hardest bottlenecks to break through in current Agent systems.

1. **Implicitness and Fragility of Assumptions**: Plans generated during the planning phase implicitly embed numerous assumptions about the environment, tools, and data states ("assuming database availability," "assuming user-provided information is accurate and complete"). These assumptions are rarely explicitly listed and verified; once an assumption proves invalid during execution, the validity of the entire subsequent plan is compromised, and the model often fails to promptly perceive that its assumptions have broken down.
2. **Incompleteness of the Feedback Closed Loop**: An ideal planning system should be a continuous closed loop of "Plan $\rightarrow$ Execute $\rightarrow$ Obtain real feedback $\rightarrow$ Update environmental understanding $\rightarrow$ Revise planning." However, in many current Agent systems, feedback obtained during execution is often insufficiently utilized to update the model's "mental model" of the environment, causing subsequent planning to proceed based on initial (and potentially outdated) assumptions even when execution results deviate from expectations.
3. **Misalignment Between Planning and Execution Capabilities**: The model responsible for planning (typically the same LLM under a different role) derives its "understanding" of the real world intrinsically from training data and current context. This understanding is inherently lagged and incomplete, whereas the true state of the execution environment changes dynamically; it is extremely difficult for the planning model to synchronize with this dynamism in real time.
4. **Compounding Errors of Long-Horizon Planning**: The more steps and longer span a plan has, the probability and magnitude of initial assumptions deviating from reality compound and amplify, causing long-horizon planning reliability to drop significantly with increased plan length. This is precisely why progressive strategies such as "rolling horizon planning" are more widely endorsed in industry than "one-shot generation of complete long-term plans."
5. **Future Directions**: Current research and engineering practices are all striving toward strengthening the "plan-execute-feedback" closed loop—verifying and updating planning assumptions more frequently with real environmental states, explicitly tracking key assumptions dependent on planning and setting verification checkpoints, adopting rolling horizon planning to reduce long-horizon forecasting uncertainty, and introducing stronger environmental perception capabilities (real-time querying rather than relying on historical memory). Fundamentally resolving this problem may depend on the model itself possessing stronger metacognitive abilities to "know what it does not know."

</details>

**43. Will Planning as a Capability Be Replaced by Model Intelligence Enhancements in the Future?**

<details>
<summary>💡 Show Answer</summary>

The model's native reasoning and planning capabilities will continue to improve, potentially reducing reliance on explicit external planning frameworks. However, as long as tasks involve interactions with the real world (tools, environments), the external closed-loop mechanism of "generate plan $\rightarrow$ execute $\rightarrow$ obtain feedback $\rightarrow$ adjust" will remain an essential engineering structure, which will not be completely replaced by pure model capability enhancements alone.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

</details>

