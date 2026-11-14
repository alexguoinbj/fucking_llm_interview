# 9.3 Agent Design for AI Coding — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 9: AI Coding · 25 questions

**1. How Should Agents Be Designed for AI Coding Scenarios?**

<details>
<summary>💡 Show Answer</summary>

The core design typically revolves around the loop of "understanding task $\rightarrow$ exploring codebase $\rightarrow$ formulating modification plan $\rightarrow$ executing edits $\rightarrow$ running tests/verifications $\rightarrow$ iterating based on feedback." The key lies in leveraging real execution feedback (compilation/test results) as the basis for course correction.



![Core loop of Plan-Explore-Edit-Test-Verify-Iterate in coding agent design.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig09_agent_control_loop.png)

</details>

**2. How Should Tool Use Interfaces for Agents Be Designed?**

<details>
<summary>💡 Show Answer</summary>

Tool interfaces should have single responsibilities and clear semantics (e.g., keeping "read file," "write file," "run command," and "search code" distinct). They should return structured results (rather than raw text) for reliable parsing by the model, while incorporating well-designed error feedback formats so the model can adjust subsequent actions based on failure causes.

</details>

**3. How Does an Agent Determine That a Task Is Completed?**

<details>
<summary>💡 Show Answer</summary>

It typically relies on objectively verifiable signals (e.g., all relevant tests passing, zero compilation errors) rather than subjective model judgment ("feeling finished"). For tasks lacking automated verification, the model can be instructed to explicitly list acceptance criteria for self-auditing, or human confirmation can be integrated as the final verdict step.

</details>

**4. How Does an Agent Avoid Falling into an "Infinite Loop" (Repeated Edits and Repeated Errors)?**

<details>
<summary>💡 Show Answer</summary>

Hard limits must be set for maximum iteration counts or time budgets, alongside state checks after each modification to verify meaningful improvement over the previous round (e.g., reduction in failing test counts). If no improvement occurs over consecutive rounds or if oscillating changes occur (editing back and forth), the Agent should proactively abort and request human intervention rather than retrying indefinitely.

</details>

**5. How Can Reasonable "Stopping Conditions" Be Designed for Agents to Avoid Over-Editing?**

<details>
<summary>💡 Show Answer</summary>

Stopping conditions must incorporate both "positive completion signals" (task goals achieved, tests passed) and "negative guardrail signals" (modification scope exceeding expected file counts, no progress over consecutive rounds). Neither can be omitted: relying solely on completion signals causes infinite retries when goals are unattainable, while relying solely on guardrails risks prematurely aborting doable tasks.

</details>

**6. How Should an Agent Retry or Fall Back After Execution Failures?**

<details>
<summary>💡 Show Answer</summary>

Failure causes should first be analyzed (environmental issues, misaligned understanding, task exceeding capabilities) before determining a strategy: environmental issues can be retried directly; misalignments require re-planning rather than repeating prior actions; tasks confirmed beyond current capabilities should gracefully fall back to "providing suggestions without automated execution," delegating decision power back to humans.



![Exception fallback path and human-in-the-loop confirmation flow.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig10_fallback_confirmation.png)

</details>

**7. How Should Error Handling and Exception Catching Mechanisms Be Designed for Agents?**

<details>
<summary>💡 Show Answer</summary>

A distinction must be made between "expected failures" (e.g., failing tests, which serve as normal iterative feedback) and "unexpected systemic errors" (e.g., tool crash, insufficient permissions). The former should continue driving Agent iteration as feedback, whereas the latter should immediately halt the task and report clearly to the user, preventing the Agent from performing unpredictable actions in an abnormal state.

</details>

**8. How Should Memory Be Designed in AI Coding Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Memory should include: project architectural conventions and coding standards, history of modifications and their rationales, and previously encountered pitfalls and solutions, facilitating the reuse of project-specific knowledge in future tasks.

</details>

**9. How Does the Agent's Memory Mechanism Handle Updates and Staleness?**

<details>
<summary>💡 Show Answer</summary>

Project standards and best practices evolve over time. Outdated memories, if uncorrected, can mislead subsequent task execution. Therefore, memory entries should be tagged with timestamps or version markers. When changes in relevant code or standards are detected, old memories should be proactively invalidated or re-summarized rather than allowing memories to accumulate indefinitely.

</details>

**10. Should Agent Memory Be Divided into Short-Term and Long-Term Memory, and How Should Each Be Managed?**

<details>
<summary>💡 Show Answer</summary>

Short-term memory corresponds to intermediate states during single-task execution (explored files, attempted approaches) and can be cleared or archived after task completion. Long-term memory corresponds to project knowledge persisted across tasks (architecture conventions, historical pitfall experience), requiring separate storage and on-demand retrieval upon starting new tasks. Their update frequencies and lifecycles are entirely distinct and must not be conflated.



![Division and lifecycles of Short-Term (task-specific) and Long-Term (repository-wide) agent memories.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig11_memory_hierarchy.png)

</details>

**11. How Are Intermediate States/Context Managed and Compressed in Long-Running Tasks?**

<details>
<summary>💡 Show Answer</summary>

Detailed processes of completed steps can be periodically summarized and compressed, retaining only conclusive information (what changes were made, verification results) while discarding procedural details. This prevents context from expanding indefinitely and exceeding window limits as the task progresses. Meanwhile, the summarization mechanism must ensure key information needed by subsequent steps is not lost.

</details>

**12. What Risks Are Introduced when an Agent Runs Autonomously for Too Long?**

<details>
<summary>💡 Show Answer</summary>

Risks include error compounding and amplification (a minor early misunderstanding being magnified in subsequent steps), uncontrolled scope creep (deviating from the original task intent), and excessive resource and cost consumption. This is why long-running autonomous tasks usually require periodic checkpoints rather than complete non-intervention.

</details>

**13. How Should Planning Be Conducted in AI Coding?**

<details>
<summary>💡 Show Answer</summary>

Planning should be grounded in exploration results of the current codebase state (rather than unsubstantiated assumptions), clearly specifying which files need modification/addition, the general content and sequence of modifications, and potential ripple effects on other modules.

</details>

**14. How Does an Agent Balance Exploration Cost (Reading Too Many Files) with Execution Efficiency?**

<details>
<summary>💡 Show Answer</summary>

A "progressive exploration" strategy can be adopted: first make coarse-grained relevance judgments based on task descriptions and only deeply read highly relevant files; if information proves insufficient during execution, supplement exploration on demand rather than reading the entire codebase at once. Concurrently, prior codebase structural knowledge from previous tasks can be reused to reduce redundant exploration overhead.

</details>

**15. How Should the Granularity of Sub-agent/Sub-task Decomposition Be Managed?**

<details>
<summary>💡 Show Answer</summary>

Decomposition granularity should be benchmarked by "independent verifiability"—each sub-task should ideally correspond to a minimal unit whose output can be tested or inspected independently. Overly coarse granularity leaves complex multi-step decisions unmanaged within sub-tasks, while overly fine granularity introduces excessive coordination overhead and context transfer costs.

</details>

**16. How Should Responsibilities Be Divided in Multi-Agent Collaborative Patterns (Orchestrator + Worker) for Coding Scenarios?**

<details>
<summary>💡 Show Answer</summary>

The Orchestrator is responsible for understanding the overall task, decomposing sub-tasks, and aggregating execution results from workers. Workers focus on executing specific sub-tasks with clear boundaries (e.g., "implementing a function," "fixing a failing test"). This division allows planning capabilities and execution capabilities for complex tasks to be optimized independently, though it necessitates carefully designed sub-task dependency ordering and result integration mechanisms.



![Coordination and task delegation architecture between Orchestrator and Worker agents.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig12_multi_agent_collaboration.png)

</details>

**17. How Does AI Handle Cross-File Code Modifications?**

<details>
<summary>💡 Show Answer</summary>

It must first identify all relevant files and their dependency relationships involved in the modification, plan the execution order (e.g., modifying interface definitions before updating call sites), and perform consistency checks across all modified files after execution.

</details>

**18. How Do Agents in AI Coding Understand and Adhere to Project-Specific Coding Standards (e.g., Linter Rules)?**

<details>
<summary>💡 Show Answer</summary>

Prior to task execution, the agent can proactively inspect configuration files (such as ESLint or Prettier configs) and extract stylistic characteristics from existing code to serve as explicit constraints. After generating modifications, it invokes the corresponding static analysis tools for validation, integrating compliance checking into an execution feedback loop rather than relying solely on the model "memorizing" coding standards.

</details>

**19. How Is Version Control Managed in AI Coding?**

<details>
<summary>💡 Show Answer</summary>

AI-generated modifications should adhere to standard version control workflows (branches, commits, pull requests), with each change accompanied by clear commit histories and descriptions to facilitate traceability and rollback when necessary.

</details>

**20. How Is Rollback of Modifications Implemented in AI Coding?**

<details>
<summary>💡 Show Answer</summary>

It relies on version control systems (such as Git) as a natural rollback mechanism—ensuring clear version snapshots exist before each AI modification so that if issues arise, changes can easily be reverted to a previous commit.

</details>

**21. How Do Agents Perform Checkpoint Recovery in Multi-Step Tasks?**

<details>
<summary>💡 Show Answer</summary>

The agent can save snapshots of the current code state and task progress upon completing each critical step. If subsequent steps encounter errors or require resumption after interruption, execution can restart from the most recent stable checkpoint rather than re-running the entire task from scratch. This reduces computational cost and mitigates the risk of state corruption caused by mid-process failures.

</details>

**22. Which Operations Require Human-in-the-Loop Confirmation?**

<details>
<summary>💡 Show Answer</summary>

Typically, operations that are irreversible or have a broad blast radius—such as deleting files, altering database schemas, pushing directly to the main branch, or modifying permission and security configurations—require human confirmation. Even if the agent is capable of executing them autonomously, it should pause prior to execution and wait for explicit human approval.

</details>

**23. How Should Sandbox Isolation and Permission Control Be Designed for Agent Execution?**

<details>
<summary>💡 Show Answer</summary>

Agents should execute commands and run tests within sandboxes or containers isolated from the production environment. Their network access and filesystem permissions must be strictly scoped (e.g., restricted to reading and writing within designated project directories) to prevent unintended model behavior—such as accidental file deletion or execution of hazardous commands—from causing irreversible impacts on live environments.

</details>

**24. How Should Protection Mechanisms Be Designed Before Agents Execute Destructive Operations (e.g., File Deletion)?**

<details>
<summary>💡 Show Answer</summary>

In addition to upfront human confirmation, protection mechanisms can include automatically creating version control snapshots, establishing whitelist/blacklist rules for high-risk operations, and requiring the agent to explicitly explain the blast radius of an action prior to execution. These multi-layered defenses collectively minimize the probability of irreversible loss caused by destructive actions.

</details>

**25. How Do Agents Collaborate with Humans to Fulfill Complex Requirements (Rather Than Operating Fully Autonomously)?**

<details>
<summary>💡 Show Answer</summary>

Tasks can be decomposed into an iterative loop of "Agent Proposal $\rightarrow$ Human Review & Adjustment $\rightarrow$ Agent Execution $\rightarrow$ Human Acceptance." Human judgment is preserved at critical decision nodes (e.g., architectural selection, wide-reaching modifications), while mechanical, verifiable execution tasks are delegated to the agent, balancing efficiency with controllability.

</details>

