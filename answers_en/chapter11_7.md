# 11.7 Agent Harness and Loop Engineering — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. What Is an Agent Harness? How Are Its Responsibilities Divided from the Model Itself?**

<details>
<summary>💡 Show Answer</summary>

An Agent Harness refers to a layer of **deterministic runtime** wrapped around an LLM, responsible for validating, authorizing, executing, and logging every action proposed by the model.

**Division of Responsibilities:**
- **Model Responsible for "Proposing"**: Reasoning based on current context, deciding which tool to invoke next, what parameters to pass, and determining whether the task is complete;
- **Harness Responsible for "Executing and Guarding"**: Performing schema validation, permission checks, budget controls (limits on tokens/costs/steps), and security rule interceptions on actions proposed by the model. Only actions passing validation are actually executed, with execution results, logs, and observability data fed back to the model.
**Significance**: An Agent without a qualified harness easily degrades into a "black box"—suffering token leaks, infinite loop calls, or executing dangerous commands unnoticed. With a harness, the Agent gains predictability, security, and observability, which is the core motivation behind the widely accepted agent engineering term ("Agent Harness") in 2026. Industry consensus holds that harness design often has an impact on final Agent performance equal to or even greater than that of the underlying model capabilities.

</details>

**2. What is Loop Engineering? What Problem Does It Attempt to Solve?**

<details>
<summary>💡 Show Answer</summary>

Loop Engineering is a concept explicitly named only in 2026. Its core proposition is: **The bottleneck of agent engineering has shifted from "how to write prompts" to "how to design control loops"**.

**Core Elements:**
1. **Trigger**: Under what conditions an Agent loop is initiated (user messages, scheduled tasks, external events, calls from other Agents);
2. **Topology**: Whether the execution structure of the loop is a simple single-threaded ReAct-style loop or a multi-Agent collaborative graph structure;
3. **Verifier**: How to judge after each step or each round whether the current result is reliable and whether a retry is needed;
4. **Stop Rule**: When an Agent should proactively terminate—whether upon task completion, reaching a budget ceiling, or making no progress over multiple consecutive steps.
**Problems to Solve:** The most common failure mode in practice is not that the model is "not smart enough," but rather that the loop itself is improperly designed—for example, repeatedly calling the same failing tool, claiming task completion without genuine verification of results, or retrying infinitely upon encountering exceptions (such as authentication failure or errors from dependent services). Loop Engineering treats these "control flow" issues as objects requiring dedicated engineering design, independent of model capabilities.

</details>

**3. Among Agent Execution Patterns, What Scenarios Are Agent Loop, Event-driven, State Machine, and Graph Orchestration Patterns Respectively Suitable For?**

<details>
<summary>💡 Show Answer</summary>

Based on a survey of a large number of open-source Agent projects, the current mainstream execution patterns can be categorized into several types, which exhibit clear trade-offs in **controllability, expressiveness, and implementation difficulty**:

- **Agent Loop (Reasoning-Action Loop)**: The model autonomously decides what to do at each step within a loop. It has the simplest implementation and strongest expressiveness (handling open-ended tasks where steps cannot be enumerated in advance), but has the weakest controllability and is prone to the aforementioned issue of loops getting out of control. Approximately 60% of mainstream open-source Agent projects adopt this pattern;
- **Event-driven**: Agent behaviors are triggered and driven by external events, suitable for passively reactive scenarios requiring decoupled collaboration with multiple external systems (e.g., "automatically summoning an Agent when a new issue appears in an issue tracker");
- **State Machine**: Finite states and state transition conditions that a task may undergo are defined in advance. It offers the strongest controllability and is suitable for structured tasks with relatively fixed flows and enumerable steps (e.g., customer service ticket handling), but its expressiveness is limited, making it difficult to handle open-ended exploratory tasks;
- **Graph/Flow Orchestration**: Positioned between Agent Loop and State Machine, it uses an explicit directed graph to define the macro structure of a task (e.g., the three stages of "planning $\rightarrow$ execution $\rightarrow$ verification"), while inside each node there can still be an autonomous Agent Loop. It balances structured control with local flexibility and is a compromise solution currently adopted by many production systems.
**Selection Principles:** When the task structure is known and the steps are enumerable, the structure should be written as program logic (state machine/graph) rather than handing over control entirely to the model; only in genuinely open-ended, exploratory tasks (brainstorming or open research without established workflows) is it more appropriate to let the model lead the execution loop.



![自主智能体系统中的结构化编排模式对比 (Comparison of structural orchestration patterns in autonomous agent systems.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig93_agent_loop_patterns.png)

</details>

**4. Why Is It Said That the Impact of "Harness Design" on Agent Performance May Be Greater Than Switching to a Stronger Base Model?**

<details>
<summary>💡 Show Answer</summary>

Research reports indicate that the task accuracy gap of the same base model under different harnesses can reach up to approximately sixfold. This finding subverts the intuition that "as long as the model is strong enough, the Agent will naturally perform well."

**Reason Analysis:**
- **The harness determines what the model can "see"**: Including the toolset exposed to the model, the quality of each tool's description, and what auxiliary information is included in each step's observation. These directly determine whether the model can obtain sufficient and appropriate context to make correct decisions—no matter how capable the model is, if critical information is not properly organized into the context, it cannot "reason" out the correct answer;
- **The harness determines whether errors can be corrected in a timely manner**: A good harness provides structured feedback at each step (such as the specific cause of a tool call failure or a structured summary of execution results), enabling the model to quickly locate problems and adjust strategies; a crude harness merely hands the model a pile of raw logs, forcing the model to extract key information from noise by itself, wasting substantial context and reasoning steps;
- **The harness determines whether failure modes can be fallbacked/safeguarded**: For example, in situations such as authentication failure, downstream service errors, or the model getting stuck in repeated calls, if the harness lacks corresponding detection and handling mechanisms, the model is likely to continue wasting resources on an incorrect path without noticing.
**Conclusion:** This is also why work like "Meta-Harness" (using AI to automatically search for optimal harness configurations) has emerged recently—given model capabilities, the harness itself has become an independent variable worthy of systematic optimization.

</details>

**5. What Is the "Reasoning Sandwich" Hierarchical Scheduling Strategy?**

<details>
<summary>💡 Show Answer</summary>

The "reasoning sandwich" is a scheduling strategy that dynamically allocates different compute/model tiers according to stage characteristics in multi-stage Agent tasks, figuratively comparing the entire process to a sandwich structure:

- **Top Layer (Planning Stage)**: Uses the model with the largest reasoning budget and highest capability (or highest thinking tier), because errors in decisions at this stage (how to decompose tasks, what overall strategy to choose) carry the highest cost and will be amplified across all subsequent steps;
- **Middle Layer (Execution Stage)**: Uses moderate-compute/faster models to complete concrete, relatively mechanical execution steps (such as invoking tools according to established plans or generating code), because this stage involves a large number of steps where single-step tasks are relatively simple and clear, making the strongest model cost-ineffective;
- **Bottom Layer (Verification Stage)**: Likewise uses models with higher reasoning budgets to strictly verify execution results, because missed detections during verification will similarly lead to error amplification and propagation downstream.
**Value:** Compared to simple schemes that "uniformly use the same model/compute tier from beginning to end," this hierarchical scheduling significantly reduces overall latency and cost while ensuring the quality of key decisions. It is currently one of the relatively mature model routing practices in production-grade Agent systems.

</details>

**6. What Design Principles Should Permission Systems and Execution Sandboxes in Agent Harnesses Follow?**

<details>
<summary>💡 Show Answer</summary>

- **Principle of Least Privilege**: When executing a specific task, an Agent should only be granted the minimum set of permissions necessary to complete that task, rather than possessing system-level full access by default, thereby preventing excessive destruction caused by erroneous model decisions or manipulation via malicious prompt injection;
- **Handling Authentication and Credential Isolation at Tool Boundaries**: Authentication logic for sensitive credentials (such as API keys and database passwords) should be handled at the tool invocation boundaries of the harness. The Agent's reasoning process itself should not directly touch raw credentials, avoiding accidental credential leaks through model outputs and reducing the risk of credentials being stolen by prompt injection attacks;
- **Sandbox-Isolated Execution Environments**: High-risk actions involving code execution, file system operations, etc., should be executed in isolated sandbox environments (containers, virtual machines). Even if the Agent executes harmful instructions, the scope of impact is confined within the sandbox without affecting the host system or other tasks;
- **Approval and Human-in-the-Loop Mechanisms**: For high-impact, irreversible operations (such as deleting data, sending external emails, or financial transactions), explicit approval stages should be designed (requiring human confirmation, or at least logging and permitting post-hoc auditing/rollback) rather than letting the Agent execute fully autonomously.

</details>

**7. What Is Context Compaction in Agents? Why Is It Needed in Long-Horizon Tasks?**

<details>
<summary>💡 Show Answer</summary>

Long-horizon Agent tasks (spanning tens or even hundreds of tool invocation steps) continuously accumulate conversation history, tool invocation logs, and intermediate results. These quickly exceed the model's context window capacity, or even if they do not, excessively long and redundant context dilutes the model's attention on key information, increasing reasoning costs and latency.

**Common Strategies for Context Compaction:**
- **Summary Compaction**: Periodically summarizes earlier historical interactions into short summaries to replace the raw detailed records, retaining only conclusive information at the cost of potentially losing details that may later prove important;
- **Hierarchical Storage**: Analogizes context to "working memory" and designs a paging mechanism similar to virtual memory, swapping infrequently accessed historical information out to external storage and swapping it back in when needed, maintaining only the most relevant active information within the current working memory window;
- **Structured Persistent Notes**: Allows the Agent to actively maintain a structured progress record (similar to a task log). After the context is compacted, this persistent note can still provide the key context of the task, supporting continuity across sessions.
**Core Trade-off:** The more aggressive the compaction, the more concise the context and the higher the reasoning efficiency, but the greater the risk of information loss; how to design a "minimum-fidelity invariant" (ensuring that key information required for correct task execution is still preserved after compaction) is the core difficulty in designing such mechanisms.



![面向长程规划将原始历史过滤为关键状态摘要的上下文压缩管线 (Context compaction pipeline filtering raw history into key state summaries for long-horizon planning.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig97_context_compaction.png)

</details>

**8. How to Detect and Prevent Agents from Getting Stuck in Infinite Loops (Loop Detection)?**

<details>
<summary>💡 Show Answer</summary>

A common failure mode for Agents in production environments is getting "stuck" locally—repeatedly invoking the same failing tool, oscillating back and forth between two states, or retrying infinitely due to persistent errors from downstream services.

**Detection Mechanisms:**
- **Behavioral Fingerprint Comparison**: Records action sequences over the recent several steps (hash digests of tool names + key parameters). If a highly repetitive pattern is detected (e.g., repeatedly calling the same tool multiple consecutive times with nearly identical parameters), it is judged as a suspected loop;
- **Progress Metrics**: Defines a quantifiable "task progress" metric (such as the number of completed subtasks or stage advancement in a state machine). If this metric remains unchanged across multiple consecutive iterations, an alert should be triggered even if the actions themselves do not completely repeat;
- **Circuit Breaker Linkage**: The loop detector should be linked with downstream "circuit breaker" mechanisms—if a tool is in an "open-circuit" (continuously failing) state and the Agent still repeatedly attempts to invoke it, it should be directly judged as a loop and trigger termination or fallback processing, rather than relying on the model to "realize" the issue by itself.
**Handling Strategies:** Upon detecting a loop, the harness layer typically forcibly interrupts the current loop. Options include directly terminating the task and reporting it, or switching to an alternative strategy (such as switching to another tool or falling back to human intervention), rather than leaving the decision-making authority to the model indefinitely.

</details>

**9. What Is the Core Concept of Meta-Harness (Automated Search for Optimal Harness)?**

<details>
<summary>💡 Show Answer</summary>

Meta-Harness treats the act of "designing a good harness" itself as a problem that can be automatically searched and optimized by AI, rather than relying entirely on manual empirical design.

**Core Concepts:**
- Treating specific harness configurations (which tools to expose, how tool descriptions are written, what auxiliary observation information is provided at each step) as a searchable design space;
- Using a highly capable LLM Agent to read performance data of candidate harness configurations on a suite of benchmark tasks, analyze their failure modes, and subsequently propose improved new harness configurations;
- Step-by-step approaching harness designs with superior performance under a given task distribution through multiple iterations of "propose configuration $\rightarrow$ evaluate $\rightarrow$ analyze failure $\rightarrow$ improve configuration." This process shares methodological similarities with model self-improvement (STaR-like methods) mentioned earlier, except that the object of optimization changes from "model parameters" to "harness configuration."
**Limitations:** Even with automated search, the gains from merely optimizing the harness itself have an upper bound—when the base model capability is inherently insufficient to complete a certain class of tasks, no matter how well designed the harness is, it cannot compensate for it. This is also why relevant research points out that "gains from harness design are not always satisfactory," indicating that harness optimization and model capability enhancement (post-training) need to be considered jointly rather than independently.

</details>

**10. Why Is It Said That There Is a Mutual Influence Between Harness Design and Model Post-training?**

<details>
<summary>💡 Show Answer</summary>

Early research on harnesses mostly assumed that the underlying model was fixed and only optimized harness configurations; however, recent studies have begun to focus on: **what kind of interaction occurs when the model itself is also being optimized via post-training for a specific harness**.

- **The harness determines the training data distribution**: If the model's reinforcement learning training environment uses a specific harness (providing specific observation formats and specific toolsets), the strategy learned by the model will naturally develop a dependency on this harness. Switching to a different harness configuration may lead to a significant performance drop, i.e., "overfitting" to specific harness assumptions;
- **The amount of auxiliary information affects the policy types learned by the model**: Research shows that models trained under a harness providing richer auxiliary information (detailed action descriptions, structured observations) and models trained under a harness with more concise information develop different decision-making patterns, and these differences demonstrate distinct robustness when the task or tool environment changes (out-of-distribution generalization);
- **Practical Implications**: This indicates that "designing a harness" should not be viewed as an engineering detail for the post-training deployment stage, but rather as a variable that needs to be co-designed with training algorithms right during the post-training stage, especially for Agent products planned for long-term iteration that may switch execution environments.

</details>

**11. What Is an Agent-Computer Interface (ACI)? Why Does Its Design Significantly Affect Agent Performance on Benchmarks?**

<details>
<summary>💡 Show Answer</summary>

Agent-Computer Interface refers to the specific interface format used when an Agent interacts with computer systems (file systems, terminals, code repositories, etc.)—not referring to underlying operating system calls themselves, but rather **encapsulated and simplified operational interfaces specifically designed for the LLM interaction mode**.

**Design Points:**
- **Adapting operation granularity to the cognitive characteristics of the model**: For example, encapsulating complex multi-step file editing operations into a single, semantically clear "edit" instruction (specifying file, line range, and new content), rather than requiring the model to directly generate low-level diff patches or line-by-line shell commands, thereby reducing the probability of model errors in low-level execution details;
- **Readability optimization for result feedback**: Information returned after command execution should be specifically optimized for model comprehension (e.g., code search results sorted by relevance and providing context snippets, rather than raw, verbose terminal outputs), reducing the burden on the model to extract effective information from noise by itself;
- **Empirical Value**: Related work (such as SWE-agent) demonstrates that under fixed model capabilities, merely redesigning the ACI (rather than improving the model itself) can bring significant task resolution rate improvements on real-world software engineering benchmarks. This is one of the important empirical sources supporting the industry consensus that "harness/interface design is just as important as model capability."

</details>

**12. What Content Should Observability Cover in an Agent Harness?**

<details>
<summary>💡 Show Answer</summary>

In production environments, an Agent's behavioral chain is much harder to predict than traditional software systems. Comprehensive observability is the foundation for discovering problems and continuous improvement.

**Content to Cover:**
- **Full Execution Traces**: Recording model inputs, model outputs (including reasoning processes, if available), actual tools and parameters invoked, and tool return results at each step, forming a replayable and auditable complete chain;
- **Traceability of Decision Points**: Being able to pinpoint at which step the Agent made an incorrect decision, whether due to insufficient context information, ambiguous tool descriptions, or the model's own reasoning errors, categorizing failure causes into specific stages rather than vaguely attributing them to "the model is not good enough";
- **Resource Consumption Metrics**: Token consumption, number of invocation steps, latency, and costs, broken down by task type and user dimension for cost control and anomaly detection;
- **Linkage with Loop Detection/Circuit Breaker Mechanisms**: Observability data should not only be used for post-hoc analysis, but should also be fed back in real time to control mechanisms such as the aforementioned loop detection and circuit breakers, forming an "observation-decision-intervention" closed loop;
- **Cross-Session Behavioral Pattern Analysis**: Aggregating large volumes of task traces to identify systemic, recurring failure modes (rather than focusing solely on the success or failure of a single task), providing data support for the next iteration of harness or model improvements. This is also the foundational data source relied upon by automated optimization methods such as Meta-Harness.

</details>

**13. What Are Agent Skills (e.g., the SKILL.md Specification)? What Problems Do They Solve?**

<details>
<summary>💡 Show Answer</summary>

Agent Skills refer to encapsulating domain-specific best practices, operational workflows, and constraint rules into structured, model-readable documentation (e.g., using SKILL.md as an entry file accompanied by a detailed references directory) for Agents to read and follow on demand when executing relevant tasks.

**Problems Solved:**
- **Avoiding "reinventing" best practices from scratch every time**: Many domain tasks have empirical rules summarized after encountering countless pitfalls (e.g., "what typesetting standards to follow when generating Word documents"). Solidifying these experiences into reusable Skill documentation is more reliable and consistent than relying on ad-hoc model performance every time;
- **On-demand loading to prevent context bloat**: Skills are typically designed in a hierarchical structure—entry files are concise and brief, and detailed reference materials are loaded further only when a task genuinely involves that domain, avoiding stuffing all potentially needed knowledge into system prompts at once and causing context waste;
- **Maintainability and Iterability**: Skills exist in document format and can be updated and iterated independently of the model itself (modifying documents directly upon discovering new best practices or correcting flawed understandings) without retraining the model. This serves as "another layer of iterable asset outside the harness"—solidifying experience into external editable natural language assets rather than model parameters.



![智能体技能包的结构组成及其与运行时引擎的集成 (Structural components of an Agent Skill package and its integration with the runtime engine.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig103_agent_skills_spec.png)

</details>

**14. What Commonalities Exist Between Harness Engineering and "Design by Contract" in Traditional Software Engineering?**

<details>
<summary>💡 Show Answer</summary>

Design by Contract is a traditional software engineering methodology whose core idea is to explicitly define **preconditions (what the caller must guarantee), postconditions (what the module promises to return), and invariants (properties that must be maintained throughout)** for each module/function, ensuring reliability of collaboration between system components through such explicit contracts.

**Commonalities with Harness Engineering:**
- **Explicitly separating uncertain parts from deterministic parts**: Design by Contract separates "how it is implemented" (subject to change, potentially buggy) from "what guarantees are promised" (a relatively stable contract); Harness Engineering similarly separates the "uncertain reasoning of the model" from the "deterministic execution guarantees of the harness." Both manage uncertainty in complex systems through explicit boundary partitioning;
- **Validation as a first-class citizen**: Design by Contract emphasizes proactively validating whether preconditions/postconditions are met at runtime, raising errors immediately upon violation rather than failing silently; a harness performs schema validation and permission checks on every action proposed by the model, sharing the exact same approach of "proactively validating at runtime and rejecting non-compliant inputs";
- **Defensive design against untrusted sources**: Design by Contract assumes callers might not follow agreements, requiring modules to perform defensive checks; a harness takes this a step further—treating the model akin to an **inherently untrusted caller** (the model may hallucinate, be manipulated by prompt injections, or make logical errors), thereby necessitating independent security checks at the harness layer independent of model self-discipline. This is why Harness Engineering is regarded as a natural extension and evolution of Design by Contract principles in the Agent era.

</details>

**15. How Should the Tool Registry in an Agent Harness Be Designed?**

<details>
<summary>💡 Show Answer</summary>

The Tool Registry is the core component in a harness for managing "which tools the model can invoke." As the number of tools connected to the Agent grows (especially third-party tools connected via protocols like MCP), its design quality directly impacts the reliability and efficiency of the Agent.

**Key Design Points:**
- **Clear, unambiguous tool descriptions**: The name, parameter explanations, and usage scenario descriptions for each tool must be sufficiently clear and non-ambiguous relative to one another. Ambiguous or overlapping tool descriptions are common causes of the model selecting wrong tools or passing wrong parameters, serving as one of the major sources behind the aforementioned "harness design affecting accuracy by several fold";
- **On-demand dynamic registration and retrieval**: When the number of available tools is large (e.g., connecting dozens of MCP Servers in enterprise scenarios), tool descriptions should not be stuffed into context all at once. Instead, dynamic retrieval and registration of the subset of tools actually needed for the current task based on task relevance should be supported, reducing context bloat and interference during model selection;
- **Version management and compatibility**: Tool interfaces (parameters, return formats) may evolve over time. The registry needs to maintain version information to prevent the model from generating incompatible calls based on outdated tool descriptions;
- **Invocation auditing and rate limiting**: The registry should record the invocation frequency, success rate, and average latency of each tool, while supporting invocation quotas set per tool or per task, acting as one of the data sources for the aforementioned observability and cost control systems.

</details>

