# 8.4 Agent and RAG Evaluation Special Topic — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 41 questions

**1. Why is Evaluating an Agent More Difficult and Complex Than Evaluating a Base LLM? How Do Evaluation Dimensions Differ?**

<details>
<summary>💡 Show Answer</summary>

Evaluating an Agent is more difficult and complex than evaluating a base LLM because the evaluation object transforms from a static, single-turn "text generator" into a dynamic, multi-turn "decision-maker" interacting with an environment.

Roots of Difficulty and Complexity:

1. Interactivity and State Space: A base LLM is stateless, and its evaluation follows a simple "input -> output" pattern. An Agent, however, is stateful, performing multi-step interactions with the environment where each action changes both the environment and its own internal state. This leads to an astronomical number of possible execution trajectories, making full coverage virtually impossible.
2. Environmental Dynamism and Uncertainty: The evaluation environment for an LLM is deterministic (the same input typically expects the same range of outputs). In contrast, an Agent's evaluation environment (such as real web pages or APIs) is dynamically changing and unpredictable. An API that works today may fail tomorrow, and web page structures can change at any time, making evaluation results difficult to reproduce.
3. Non-determinism: Due to sampling randomness inherent in LLMs and environmental dynamism, the same Agent executing under identical initial task parameters may yield entirely different execution paths and final results across two runs.
4. Task Open-endedness: Tasks handled by Agents are often open-ended without a single correct answer (e.g., "Help me book the most cost-effective flight ticket to Singapore"), making it impossible to define a simple binary "correct/incorrect" metric.

Differences in Evaluation Dimensions:

| Evaluation Dimension | Base LLM | Agent |
| :----------------- | :----------------------------------------------------------------------------------------------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Core Evaluation Object | Quality of a single response | The entire task completion process |
| Primary Dimensions | - Accuracy<br>- Fluency<br>- Relevance<br>- Safety | - Task Success Rate: Can it ultimately achieve the goal?<br>- Efficiency: How many resources were spent completing the task? (See below)<br>- Robustness: Can it handle exceptions and errors?<br>- Autonomy: How far can it go without human intervention? |
| Added Process Dimensions | (None) | - Cost: Number of LLM calls, API fees, token consumption.<br>- Latency: Total time to complete the task.<br>- Number of Steps: Steps in task decomposition and execution.<br>- Error Recovery: Ability to recover from tool errors or faulty states. |
| Evaluation Methods | Benchmarks on static datasets (MMLU, HumanEval) | Benchmarks in interactive environments (WebArena, AgentBench) |

In summary, evaluating an LLM is more like a "product quality inspection," whereas evaluating an Agent is more like a "real-world driving test under complex road conditions"—evaluating not only whether the destination is reached, but also driving efficiency, safety, and the ability to handle unexpected emergencies.

</details>

**2. Why is the Evaluation of Agent Systems Particularly Difficult?**

<details>
<summary>💡 Show Answer</summary>

Agent tasks are often open-ended and multi-step, where a "correct answer" may not be unique. The rationality of the execution process is not entirely equivalent to the correctness of the final result, and external side effects such as tool calls are involved, making it difficult to evaluate using simple string matching.

</details>

**3. How Should the "Success Rate" of an Agent Be Defined?**

<details>
<summary>💡 Show Answer</summary>

It is typically defined based on clear task completion criteria (e.g., whether the final state meets expectations, and whether key artifacts are generated and correct). It can be binary (success/failure) or graded (partial completion scoring), requiring criteria tailored to specific task scenarios.

</details>

**4. What Are the Commonly Used Metrics for Agent Evaluation?**

<details>
<summary>💡 Show Answer</summary>

They include task success rate, average execution steps, token/cost consumption, execution latency, tool call accuracy, error recovery rate, as well as sub-metrics for specific capabilities (e.g., planning rationality, memory usage effectiveness).

</details>

**5. When Evaluating an Agent's Task Completion, What Process Metrics Deserve Attention Besides the Correctness of the Final Result? (e.g., Efficiency, Cost, Robustness)**

<details>
<summary>💡 Show Answer</summary>

When evaluating an Agent, solely focusing on the correctness of the final result (Task Success) is far from sufficient. An excellent Agent must not only "do things right," but also "do things smartly, efficiently, and reliably." Therefore, paying attention to process metrics is crucial, as they provide a more comprehensive reflection of the Agent's intelligence level.

Key process metrics worthy of attention include:

1. Efficiency:

- Definition: Measures the resources consumed by the Agent to complete a task. Efficiency is a key factor determining whether an Agent is practical in real-world applications.
- Specific Metrics:
- Cost:
- Token Consumption: The total number of tokens consumed by the Agent across all thinking and generation steps.
- API Call Fees: If paid LLM or tool APIs are used, the total cost to complete a task.
- Latency:
- Wall-clock Time: The actual elapsed real time from task start to completion.
- Computation Time (CPU/GPU Time): The computation time consumed by the Agent's own execution.
- Number of Steps / Turns: The total count of "thought-action" loops executed by the Agent. Typically, an Agent that can complete a task in fewer steps is considered to have stronger planning capabilities.

1. Robustness:

- Definition: Measures the Agent's performance when encountering non-ideal or unexpected situations.
- Specific Metrics:
- Error Handling Capability: When a tool returns an error, a webpage fails to load, or unexpected environmental states occur, whether the Agent can identify the issue and take corrective measures (e.g., trying different tools, correcting input parameters, re-planning).
- Disturbance Resistance: Adding noise or misleading information to the environment to evaluate how much the Agent's success rate drops.

1. Autonomy & Alignment:

- Definition: Measures the degree to which the Agent can complete tasks independently and whether its behavior aligns with human intent.
- Specific Metrics:
- Number of Human Interventions: In a system requiring human assistance, a more autonomous Agent requires fewer human interventions.
- Interpretability: Whether the Agent's "thought" process is clear and logical, enabling humans to understand the basis for its decisions.
- Plan Adherence: If the Agent generates a plan in advance, the extent to which it follows its own plan.

By comprehensively evaluating these process metrics, we can not only determine whether an Agent "can accomplish the task," but also gain deep insight into how well it performs, thereby identifying targeted directions for optimization.





![Execution metrics grid for evaluating AI Agent performance beyond success rates. (AI Agent 任务执行多维度过程指标评测体系结构图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig17_agent_process_metrics.png)

</details>

**6. How Should the Cost of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Calculate overall metrics such as "cost per successful task" by tallying the total token consumption, the number of model/tool calls and their corresponding fees for completing a single task, combined with the task success rate.

</details>

**7. How Should the Latency of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Measure the total end-to-end time required to complete a task and break down the proportion of time spent in each stage (model inference, tool execution, waiting) to pinpoint latency bottlenecks.

</details>

**8. How to Conduct Offline Evaluation for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Run the Agent in batches within an isolated environment using a pre-constructed standardized task set (including inputs and evaluation criteria), and evaluate results automatically or semi-automatically. This approach is suitable for regression verification prior to version iterations.

</details>

**9. How to Conduct Online Evaluation for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Collect real-world performance data of the Agent from live traffic via instrumentation (success rate, user feedback, manual sampling audit results) to discover real-scenario issues that offline evaluation failed to cover.

</details>

**10. How to Conduct A/B Testing for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Allocate live traffic proportionally to different versions of the Agent (with different prompt, model, or tool configurations) and compare differences in key business metrics (task completion rate, user satisfaction, cost) to validate the effectiveness of changes.

</details>

**11. How to Conduct Regression Testing for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Maintain a standard task set covering core scenarios and historical problem cases, and re-run this test set after each system modification to ensure that new changes do not introduce performance regressions to existing capabilities.

</details>

**12. How to Achieve Automated Agent Evaluation?**

<details>
<summary>💡 Show Answer</summary>

For tasks with explicit evaluation criteria (such as whether code passes tests or whether numerical calculations are correct), programs can be used for automatic judgment. For open-ended outputs, an "LLM-as-Judge" model can be introduced to score according to predefined standards, supplemented by manual sampling checks to calibrate the reliability of the judge model.

</details>

**13. How to Conduct Task-Level Evaluation for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Focus on whether the end-to-end final result achieves the task goal without considering intermediate processes; this is suitable for measuring "whether the problem was solved for the user."

</details>

**14. How to Conduct Step-Level Evaluation for an Agent?**

<details>
<summary>💡 Show Answer</summary>

Evaluate the rationality and correctness of each intermediate step individually (such as a specific tool call or planning decision); this is suitable for pinpointing which specific stage experienced an issue, serving as a detailed complement to task-level evaluation.

</details>

**15. How Should the Tool Calling Capability of an Agent Be Evaluated Separately?**

<details>
<summary>💡 Show Answer</summary>

Construct a dedicated tool calling test dataset to evaluate metrics such as "whether the correct tool was selected," "whether parameters were filled correctly," and "whether the calling timing was reasonable," decoupling the evaluation from the final task outcome.

</details>

**16. What Are "Tool Use Benchmarks" in Evaluation (e.g., ToolBench, Berkeley Function-Calling Leaderboard)? What Aspects Do They Focus On?**

<details>
<summary>💡 Show Answer</summary>

These benchmarks specifically evaluate the model's function calling and tool execution capabilities. Dimensions of focus include: whether the model can correctly select the tools to invoke based on the task (in single-tool or multi-tool scenarios), whether it can accurately generate invocation parameters adhering to the schema, whether it can properly process tool execution results to generate a final answer, and its selection accuracy when faced with a large number of available tools.

</details>

**17. What Are the "AST Evaluation" and "Execution Evaluation" Judgment Methods in the Berkeley Function-Calling Leaderboard?**

<details>
<summary>💡 Show Answer</summary>

AST (Abstract Syntax Tree) evaluation parses function calls generated by the model into structured representations and compares them with ground-truth answers at the structural and parameter levels (checking whether function names, parameter names, and parameter values match, allowing for certain equivalent forms); execution evaluation, on the other hand, actually executes the model-generated function calls (in a real or simulated environment) to check whether execution outputs meet expectations. AST evaluation is fast and does not require a real execution environment, but it may misjudge certain valid calls that are "syntactically different but semantically equivalent"; execution evaluation is closer to real-world scenarios but carries higher implementation costs and complexity.

</details>

**18. How to Evaluate the Discrepancy That May Arise Between "Tool Selection Correctness" and "Final Task Completion" in a Tool-Enabled Agent?**

<details>
<summary>💡 Show Answer</summary>

Two separate evaluation setups can be constructed: (1) Tool calling trajectory evaluation—independently checking whether a reasonable tool and parameters were selected at each step (even if the final task ultimately fails due to other reasons); (2) Task result evaluation—focusing solely on whether the final state achieves the goal (even if the intermediate process is not the "standard" path, provided it reaches the goal via another reasonable path). Discrepancies between the two (such as "non-standard" tool choices that nevertheless complete the task) are valuable diagnostic insights themselves, potentially reflecting that the standard path definition is overly narrow or that the model discovered a superior path.

</details>

**19. How Should the RAG Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluate the retrieval stage (whether correct documents/chunks were recalled) and the generation stage (whether accurate and faithful answers were provided based on the retrieved content) separately; reference can be made to the layered evaluation methods mentioned in RAG evaluation.

</details>

**20. How to Evaluate Whether "Retrieved Content Is Correctly Utilized by the Model" in a RAG System (i.e., Whether Generated Content Is Faithful to Retrieval Results)?**

<details>
<summary>💡 Show Answer</summary>

The following methods can be adopted: (1) NLI (Natural Language Inference) model-based evaluation, determining whether each generated statement is entailment-supported by the retrieved content; (2) LLM-as-Judge, which, given the retrieved content and generated response, assesses whether every factual statement in the answer has verifiable evidence; (3) using the "hallucination rate" (the proportion of statements in the generated text that lack evidence in the retrieved content) from manually annotated samples as a calibration baseline.





![Pipeline for evaluating faithfulness and citation accuracy in retrieval-augmented systems. (RAG 系统中生成内容对检索文档忠实度（Faithfulness）评估流程图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig18_rag_faithfulness_eval.png)

</details>

**21. How Should the Memory Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Design test scenarios requiring multi-turn or cross-session interaction to answer correctly, assessing whether the Agent correctly stores, retrieves, and utilizes relevant historical information, as well as whether it avoids interference from irrelevant memories.

</details>

**22. How Should the Planning Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluate whether the generated plan covers the key steps required for the task, whether the sequence of steps is logical, and whether there are redundant or missing steps, comparing against human expert solutions or reference benchmarks.

</details>

**23. What Are the Benchmark Design Difficulties in Evaluating "Long-Horizon Planning" Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Difficulties include: (1) The tasks themselves require long step sequences to reflect "long-horizon" characteristics, but more steps lead to higher evaluation execution costs (time and token consumption); (2) Clear, automatically verifiable "intermediate milestones" and "final success" criteria must be designed; otherwise, it is difficult to distinguish between "the model's overall direction is correct but a specific intermediate step failed" and "the model completely deviated from the goal"; (3) Due to dependencies between steps, a minor error in one step may be amplified in subsequent steps, making the final success rate highly sensitive to the correctness of early steps, which can result in large variance in evaluation results and require multiple runs to gather statistical averages.





![Compounding error propagation and planning failures in long-horizon agent tasks. (长程规划（Long-Horizon Planning）评测中误差累积导致任务失败的链条图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig19_long_horizon_planning.png)

</details>

**24. How Should the Safety of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Construct test sets containing scenarios such as privilege escalation attempts, induction of sensitive information leakage, and malicious prompt injection, verifying whether the Agent can correctly identify and refuse/evade these risky operations.

</details>

**25. How Should the Robustness of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Perform perturbation tests on inputs (such as typos, incomplete information, and unconventional expressions), observe whether the Agent's performance remains stable under these perturbations, and identify issues where the Agent is overly sensitive to specific input formats.

</details>

**26. How Should the Stability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Run the Agent multiple times on identical or similar inputs, compute the consistency/variance of the output results (or task success status), and evaluate the repeatability of the system across multiple runs.

</details>

**27. How Should the Interpretability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Check whether the Agent can provide clear rationale explanations for its decisions and actions that align with the actual execution process, which can be evaluated by manually reviewing the consistency between traces and explanations.

</details>

**28. Why Do Agents Often Score High on Benchmarks But Frequently Fail in Real Online Deployment?**

<details>
<summary>💡 Show Answer</summary>

Benchmark tasks typically feature relatively concentrated distributions and standardized input formats, whereas online deployment encounters broader, "dirtier" real-world distributions; additionally, benchmark evaluation criteria may differ from actual business success metrics (i.e., "gaming benchmark scores" and "solving real-world problems" are not entirely equivalent).

</details>

**29. Do Current Agents Truly Possess "Reasoning" Capabilities?**

<details>
<summary>💡 Show Answer</summary>

The reasoning capability demonstrated by current models is essentially pattern extension and probabilistic combination derived from large-scale data training. While it can produce reasoning-like effects across many tasks, its underlying mechanism differs from human reasoning based on causal understanding, showing limited generalization ability toward novel logical structures beyond the training distribution.

</details>

**30. Do Current Agents Truly Possess "Planning" Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Agents can generate structurally sound plan texts and perform close to "planning" in many scenarios, but their planning process lacks continuous modeling of real-world states and feedback loops. It resembles generating "plausible-looking" step sequences based on empirical patterns rather than strict state-space search (like classical AI planning algorithms).

</details>

**31. Do Current Agents Truly Possess "Learning" Capabilities?**

<details>
<summary>💡 Show Answer</summary>

"Learning" within a single conversation or task mainly manifests as utilizing new context information to adjust subsequent behavior (i.e., In-Context Learning), which does not alter the model's underlying parameters. Achieving persistent capability enhancement still relies on external memory mechanisms or model fine-tuning.

</details>

**32. How Should the Generalization Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Design test scenarios whose distributions differ from the training/fine-tuning data (different domains, different task structures) and observe the performance degradation of the Agent in these "out-of-distribution" scenarios.

</details>

**33. How Should the Long-Term Task Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Design tasks requiring continuous multi-step execution over extended time horizons (or across multiple sessions), evaluating whether the Agent maintains goal consistency and state correctness throughout the entire process.

</details>

**34. How to Design Evaluation Methods for "Long-Term Tasks" (e.g., Tasks Spanning Multiple Days or Sessions)?**

<details>
<summary>💡 Show Answer</summary>

It is necessary to build an evaluation environment supporting cross-session state persistence (simulating system capabilities to save and restore states during task execution intervals). In addition to final task completion, evaluation metrics should focus on: accurate tracking of task progress, whether goal forgetting or drift occurs, ability to recover properly after interruption, and whether resource consumption (cost, time) throughout the process remains within reasonable limits. Since such evaluations are time-consuming, automated evaluation frameworks typically need to support parallel execution of large numbers of long-term task instances.

</details>

**35. How Should the Complex Task Processing Capability of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Design comprehensive tasks involving multiple sub-goals, multiple tools, and inter-step dependencies, observing whether the Agent can correctly decompose and coordinate the execution of each part.

</details>

**36. How Should the Collaboration Capability of an Agent Be Evaluated (in Multi-Agent Scenarios)?**

<details>
<summary>💡 Show Answer</summary>

Evaluate whether communication among multiple Agents in collaborative tasks is efficient, whether redundant work or omissions occur, and whether the final integrated output outperforms the output of any single Agent.

</details>

**37. How Should the Autonomy of an Agent Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluate whether an Agent can autonomously perform reasonable task decomposition, decision-making, and execution when given only high-level goals without explicit step-by-step guidance, as well as its autonomous coping capability when encountering unexpected situations.

</details>

**38. What benchmarks do you know that are specifically designed for evaluating Agent capabilities? How do these benchmarks typically construct test environments and tasks?**

<details>
<summary>💡 Show Answer</summary>

Yes, with the rise of Agent research, a series of benchmarks specifically designed to evaluate Agent capabilities have been developed. Their core characteristic is providing controlled and reproducible interactive environments.

Several well-known Agent capability benchmarks include:

1. WebArena:

- **Focus Area**: Web browsing and operation.
- **Brief Introduction**: A highly realistic, standalone web environment simulator. It replicates the functionality of multiple real-world websites (such as e-commerce, forums, software development collaboration tools), allowing Agents to complete complex real-world tasks within them.
- **Task Examples**: Find a product meeting specific requirements (such as price and rating) on an e-commerce website and add it to the shopping cart; book a meeting room on a forum.
- **Evaluation Method**: Programmatic judgment based on the final web page state (for example, whether the shopping cart contains the correct item).

1. AgentBench:

- **Focus Area**: Comprehensive evaluation of general Agent capabilities.
- **Brief Introduction**: A comprehensive benchmark containing 8 distinct environments to evaluate Agent capabilities across different scenarios.
- **Task Examples**:
- Operating System Environment: Operate files and execute commands in a Linux terminal.
- Database Environment: Query a SQL database based on natural language questions.
- Knowledge Graph Environment: Perform multi-hop reasoning over a knowledge graph.
- Game Environment: Play simple text-based adventure games.

1. GAIA (General AI Assistants):

- **Focus Area**: Simulating humans using real tools to complete complex tasks.
- **Brief Introduction**: A highly challenging benchmark whose questions typically require Agents to perform multi-step reasoning and combine multiple tools (such as web browsers, code interpreters, file operations) to solve. These problems are designed to be simple for humans but difficult for AI.
- **Task Example**: "Among all papers that cite both Paper A and Paper B, who is the third author of the paper with the highest citation count?"

How do these benchmarks typically construct test environments and tasks?

1. Environment Construction -> Sandboxing & Reproducibility:

- For security and reproducibility, benchmarks typically do not grant Agents direct access to the live Internet; instead, they create a controlled, isolated environment.
- Methods:
- Use Docker containers to encapsulate an independent environment containing a browser, terminal, and file system.
- For web browsing, a static copy of a website is usually hosted locally, or a web backend simulator is used to respond to Agent requests.
- API calls are redirected to a mock API server.

1. Task Construction -> Goal-Oriented:

- Tasks are typically provided in the form of a high-level goal rather than specific step-by-step instructions.
- Task design attempts to cover a variety of capabilities that Agents need to demonstrate, such as information retrieval, tool use, reasoning and planning, memory, etc.
- Tasks usually come with explicit, programmatically verifiable success criteria.

1. Evaluation Construction -> Programmatic Validation:

- The core of evaluation is automatically judging whether a task succeeded.
- Methods: After the Agent completes the task, an evaluator script automatically checks whether the final state of the environment meets the success conditions.
- Examples:
- Check whether a file with the correct contents has been created on disk.
- Check whether the final state of the shopping cart contains the correct item and quantity.
- Check whether the final answer string submitted by the Agent matches the ground truth answer.





![Interactive agent benchmark architecture with sandboxed runtime environments. (AI Agent 评测基准的交互式沙盒运行与自动校验架构图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig20_agent_benchmark_env.png)

</details>

**39. What is an "Agentic Benchmark"? What are its fundamental differences from traditional single-turn QA Benchmarks?**

<details>
<summary>💡 Show Answer</summary>

Agentic Benchmarks (such as SWE-bench, WebArena, AgentBench, etc.) evaluate a model's ability to complete complex tasks through multi-step interactions with environments (e.g., code repositories, web pages, operating systems) and tool calls. They focus on end-to-end "task completion" rather than the accuracy of a single question-answering turn. Compared with traditional single-turn QA Benchmarks, their evaluation process is more complex (requiring the construction of interactive environments and definition of task success criteria), and evaluation costs and execution time are significantly higher, but they are much closer to actual application scenarios of Agents in the real world.

</details>

**40. Why is "environment reproducibility" a particularly prominent challenge when evaluating Agentic Benchmarks (such as SWE-bench)?**

<details>
<summary>💡 Show Answer</summary>

Agentic Benchmarks typically require evaluating a model's multi-step operations in a complete, interactive environment (such as a code repository with specific dependency versions or a runnable container). Minor differences in the environment itself (such as varying dependency library versions, non-determinism from network access, or randomness in container initial states) can affect whether a task succeeds. Compared to pure text input-output evaluation, environment-level uncertainties introduce extra "noise", making it harder to ensure the reproducibility of evaluation results and cross-team consistency. This necessitates strict environment imaging/containerization and version pinning to mitigate.

</details>

**41. What is the biggest difficulty in Agent evaluation?**

<details>
<summary>💡 Show Answer</summary>

It is the lack of a unified, scalable evaluation standard that truly reflects online performance—human evaluation is expensive and subjective, while automated metrics (such as LLM-as-a-judge scoring) carry their own biases and unreliability. Finding a balance among evaluation cost, scale, and accuracy is the core difficulty.

</details>

