# 6.8 Agent System Design Comprehensive Questions — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 6: Agent · 7 questions

**1. Please Design an Agent Capable of Automatically Completing the Full End-to-End Workflow of "Reading Requirements Document $\rightarrow$ Generating Code $\rightarrow$ Running Tests $\rightarrow$ Fixing Bugs." Describe the Architecture and Key Design Decisions.**

<details>
<summary>💡 Show Answer</summary>

**Architecture Design:**
- **Orchestrator Agent**: Responsible for overall planning and coordination, receiving requirements documents, allocating subtasks, and monitoring progress
- **Requirements Analysis Agent**: Parses requirements documents, generating structured requirements lists and acceptance criteria
- **Code Generation Agent**: Generates code according to requirements, invoking code execution tools to verify syntax
- **Test Execution Agent**: Runs test suites, collecting failed test cases and error messages
- **Debug Agent**: Analyzes errors, generates fix plans, and invokes code execution to verify solutions

**Key Design Decisions:**
1. Set a maximum number of repair loops (e.g., 5 iterations) to prevent infinite debug loops;
2. Perform all code execution within sandboxes to prevent test code from damaging the environment;
3. Record changes with `git diff` after each code generation and fix, enabling rollbacks;
4. Pause and request human review when test failures exceed threshold bounds;
5. Track mapping relationships from requirements documents to test cases throughout the pipeline, ensuring all requirements are covered by corresponding tests.




![Illustration of Please Design an Agent Capable of Automatically Completing the Full End-to-End Workflow of "Reading Requirements Document $\rightarrow$ Generating Code $\rightarrow$ Running Tests $\rightarrow$ Fixing Bugs." Describe the Architecture and Key Design Decisions.](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig57_please_design_agent_capable.png)

</details>

**2. If You Were to Build an Enterprise-Grade Agent Platform from Scratch, What Would be the Core Modules?**

<details>
<summary>💡 Show Answer</summary>

Core Modules: \begin{enumerate}
- **Agent Orchestration Engine**: Core runtime supporting Agentic Loop, Multi-Agent collaboration, and parallel execution;
- **Tool Registration and Management**: Tool registration, versioning, permission control, and monitoring;
- **Memory System**: Unified interface combining short-term context management + long-term vector storage + episodic memory;
- **State Management**: Task state persistence, checkpointing, and breakpoint resuming;
- **Security Layer**: Prompt Injection detection, authorization checks, and operational audit logging;
- **Observability**: Distributed tracing, metrics collection, log management, and real-time dashboards;
- **Testing Framework**: Mocking tools, end-to-end test suites, and regression testing CI/CD pipelines;
- **Release Management**: Agent logic version control, canary deployments, and fast rollbacks;
- **Billing and Quota**: Token consumption billing and user quota management;
- **Developer Portal**: Agent configuration GUI, tool marketplace, documentation, and code examples.

</details>

**3. How to Design an Agent Capable of Reliably Executing Tasks Lasting Several Hours?**

<details>
<summary>💡 Show Answer</summary>

1. **State Persistence**: Serialize state to external storage after each step, supporting breakpoint resuming at any arbitrary moment;
2. **Checkpointing Mechanism**: Save checkpoints upon completing major subtasks rather than only at every individual step;
3. **Idempotent Operation Design**: Ensure all tool invocations are idempotent so that retries produce no side effects;
4. **Context Management**: Combine rolling summarization with external memory to prevent context window overflow;
5. **Human-in-the-Loop Checkpoints**: Report progress to the user and request confirmation to proceed after completing each major milestone;
6. **Resource Caps**: Establish hard limits for maximum steps, maximum execution time, and maximum token budgets;
7. **Monitoring and Alerting**: Trigger alerts when no progress is detected over extended periods;
8. **Error Budgeting**: Allow a designated percentage of step failures and retries while enforcing an overall upper bound on failure budgets.

</details>

**4. How Can Agent "Hallucinations" be Mitigated in Comprehensive System Design?**

<details>
<summary>💡 Show Answer</summary>

Comprehensively mitigating hallucinations from a system design perspective: \begin{enumerate}
- Strict tool parameter validation (schema validation, rejecting illegal parameters);
- Result grounding (subsequent steps must explicitly reference content returned by tools, disallowing ungrounded "knowledge");
- Explicit state representation (tracking progress with structured state objects rather than relying on LLM memory);
- Critical step verification (requiring the LLM to restate the execution plan and confirm before executing irreversible operations);
- External state auditing (periodically querying actual state via tools rather than relying on LLM memory of execution history).

</details>

**5. How Should Task Decomposition be Handled for Complex Tasks?**

<details>
<summary>💡 Show Answer</summary>

Granularity design in task decomposition represents the core trade-off in Agent planning quality: if decomposition is too coarse, the complexity of individual steps remains excessively high, making reliable single-step execution difficult for the model; if decomposition is too fine, step counts explode exponentially, causing coordination and management overheads to surge dramatically.

1. **Two-Stage Decomposition Strategy**: Stage one generates high-level milestone divisions (the primary phases of the task), which do not require excessive precision as long as the logical structure is sound; stage two refines and expands a milestone into concrete executable subtasks only when that milestone is about to be executed. This "coarse-first, fine-later" approach avoids attempting overly precise full-scale decomposition under uncertain information at the very beginning.
2. **Aligning Granularity with Execution Capability**: The minimal granularity of decomposition should precisely correspond to a single tool invocation or a distinct LLM inference step. One should neither split "calling an API" further into finer sub-steps (over-decomposition) nor bundle a complex operation requiring multi-tool coordination into a single step (under-decomposition leading to unreliable execution).
3. **Explicit Expression of Dependency Relations**: Decomposed subtasks should clearly indicate mutual dependencies (which must execute sequentially, and which can run in parallel). This dependency structure itself reflects decomposition quality—overly fragmented decomposition is often accompanied by complex and chaotic dependency networks that impede efficient scheduling.
4. **Dynamically Adjusting Decomposition Granularity**: Decomposition granularity should not remain static; it should adjust dynamically based on actual complexity observed during execution. If a subtask turns out to be more complex than expected during execution, it can be split further; if several subtasks can be merged for simplification, they should be flexibly combined rather than mechanically adhering to the initial decomposition scheme.
5. **Explicit Consideration of Coordination Costs**: Evaluating a decomposition scheme must account for coordination overheads (complexity of inter-step data passing, state synchronization, and error handling) rather than solely considering "whether logical sub-division is possible." The coordination costs introduced by over-decomposition sometimes outweigh the clarity benefits gained from decomposition itself.

</details>

**6. How Does an Agent Set Execution Priorities for Multiple Pending Tasks?**

<details>
<summary>💡 Show Answer</summary>

Task prioritization in Multi-Agent systems directly impacts overall throughput and response speed for critical tasks, requiring a rational scheduling strategy designed in conjunction with multi-dimensional task attributes.

1. **Dependency Constraint Layer (Baseline Constraint for Prioritization)**: The primary principle of prioritization is not violating dependencies—tasks whose prerequisite tasks remain incomplete cannot be executed regardless of how high their priority is. First perform a topological sort to identify the "currently executable task set" (all prerequisite tasks completed), and then sort by priority within this set to ensure sequence validity.
2. **Urgency (Deadline-Driven)**: Tasks with deadlines take precedence using the Earliest Deadline First (EDF) strategy—within the executable task set, the task with the nearest deadline is prioritized for Agent resource allocation. For tasks lacking explicit deadlines, preset "implicit SLAs" based on task types (e.g., an implicit deadline of 5 seconds for real-time user interaction tasks and 1 hour for batch processing tasks).
3. **Critical Path Priority**: Identify the critical path in the overall task graph (the path where delays impact final completion time). Tasks on the critical path take higher priority over non-critical path tasks, because delays in critical path tasks directly postpone completion of overall system goals, whereas non-critical tasks have "slack time" permitting reasonable delays.
4. **Resource-Aware Scheduling**: Do not sort purely by priority; also consider current resource availability. Prioritize tasks whose resource demands best match currently available resources to avoid resource starvation, where lower-resource-demand medium-priority tasks wait idly because high-priority tasks temporarily lack sufficient resources.
5. **Anti-Starvation Mechanism**: Pure priority scheduling can lead to low-priority tasks being starved of execution over long periods. Introduce "waiting time weighting"—the longer a task waits, the higher its effective priority automatically scales, ensuring all tasks are eventually executed and preventing low-priority tasks from being indefinitely deferred.

</details>

**7. If You Were to Design a Skill Management Platform from Scratch, What Would be the Core Functional Modules?**

<details>
<summary>💡 Show Answer</summary>

Core Functional Modules: \begin{enumerate}
- **Skill Editor**: Online editing of `SKILL.md`, `description`, and `references` files with syntax highlighting and format validation;
- **Trigger Testing Tool**: Input test messages and view in real time which skill is triggered to facilitate debugging the `description`;
- **Version Management**: Git integration for Skills, version history viewing, and one-click rollback;
- **Test Suite Management**: Maintain positive and negative test cases for each skill, supporting one-click regression testing execution;
- **Token Analysis Dashboard**: Visualize token consumption per skill to identify optimization opportunities;
- **A/B Testing Framework**: Compare performance across different skill versions;
- **Release Pipeline**: Automated release workflow: Testing $\rightarrow$ Canary $\rightarrow$ Full Rollout;
- **Monitoring and Alerting**: Automated alerts when trigger rates become abnormal or quality degrades.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

</details>

