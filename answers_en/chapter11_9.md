# 11.9 Agent Product Engineering Architecture (Taking Coding Agents as an Example) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. What Core Modules Are Typically Included in a Production-Grade Coding Agent Runtime?**

<details>
<summary>💡 Show Answer</summary>

Taking publicly analyzed Coding Agent architectures (such as OpenAI Codex class products) as a reference, a production-grade Agent runtime is far more complex than "sending a single request and printing a single response," typically containing the following core modules:

- **Session State Management**: Maintaining the context, historical interactions, and progress of completed subtasks for the current task;
- **Permission System**: Controlling which categories of operations the Agent is authorized to execute in the current session (read-only, writable files, executable network requests, etc.);
- **Execution Sandbox**: An isolated code execution environment preventing Agent operations from affecting the host system;
- **Tool Registry**: Managing the set of tools callable by the Agent along with their interface definitions;
- **Event/Task Queue**: Handling the scheduling of multiple concurrent requests or multi-step tasks;
- **Context Manager**: Responsible for long-horizon management logic such as context compression and history summarization discussed earlier;
- **Protocol Layer**: Interfacing with various client entry points like CLI, desktop apps, IDE plugins, and MCP Servers, unifying underlying interaction protocols.
**Core Conclusion:** The architectural complexity of such systems is closer to a small operating system or runtime environment than to a simple "chatbot backend," which is also a direct reflection of the judgment that "the engineering barrier of Agent products does not lie in the model itself."



![生产级代码智能体运行时的核心组件与模块化架构 (Core components and modular architecture of a production-grade coding agent runtime.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig121_coding_agent_runtime.png)

</details>

**2. How Are Task Decomposition and Result Aggregation Typically Designed in Multi-Agent Parallel Collaboration Architectures?**

<details>
<summary>💡 Show Answer</summary>

When dealing with tasks of high complexity that can be decomposed, an increasing number of Agent products adopt an architecture of "Main Agent (Orchestrator) + multiple Subagents executing in parallel," rather than a single Agent processing sequentially from start to finish.

**Typical Design:**
- **Main Agent Responsible for Task Decomposition**: Upon receiving a complex task, the Main Agent first performs overall understanding and planning, breaking the task into several relatively independent and parallelizable subtasks, and assigning clear goals, scopes, and acceptance criteria for each subtask;
- **Subagents Executing Independently**: Each Subagent works within a relatively isolated context (typically with independent tool permissions and context windows), processing its subtask in parallel without mutual interference. This both improves overall processing efficiency (parallelism instead of serialization) and enables each Subagent to focus on a smaller problem scope, reducing the risk of single-context overload;
- **Result Aggregation and Conflict Resolution**: After all Subagents complete their work, the Main Agent (or a dedicated aggregation step) consolidates the outputs of each Subagent, detects and resolves potential conflicts (such as merge conflicts resulting from multiple Subagents modifying different parts of the same code file), and initiates a new refinement loop if necessary;
- **Core Value**: The core pain point addressed by this architecture is the limited context window and low serial processing efficiency of a single Agent. Through reasonable task partitioning and parallelization, the system can complete larger-scale complex tasks in shorter actual execution time. However, the tradeoff is that the quality of task decomposition directly determines the overall effect—unreasonable decomposition (vague subtask boundaries, excessive interdependencies) will instead yield worse results than serial processing.

</details>

**3. How Should the Protocol Layer Be Designed When Interfacing Agents with Multiple Entry Points Such as IDEs, Terminals, and the Cloud?**

<details>
<summary>💡 Show Answer</summary>

Modern Agent products often need to support multiple client formats simultaneously (command-line tools, desktop applications, IDE plugins, mobile remote access, and cloud asynchronous execution). Protocol layer design must achieve a balance between **uniformity** and the **differentiated requirements of each client**.

- **Decoupling Core Engine from Clients**: Implementing the Agent's core logic, such as inference loops and tool execution, as an engine service independent of specific clients. Various clients are only responsible for UI rendering and user interaction, communicating with the core engine via a unified protocol (such as an event-stream-based communication protocol). In this way, adding a new client format does not require re-implementing the core logic;
- **Session Portability**: A user may initiate a long-running task in a terminal and later wish to remotely check progress or intervene in decisions via a mobile app. Protocol design needs to support session state persistence and cross-client continuation, rather than tightly coupling the session to a specific client process;
- **Hierarchical Interaction Granularity**: Different entry points have varying demands for interaction granularity—terminal scenarios may prefer streaming, fine-grained text output, while IDE plugin scenarios require structured protocol messages that directly map to editor actions (such as highlighting specific code lines or displaying diffs). The protocol design must support such multi-tiered information expression, rather than delivering raw text streams of identical granularity to all clients.

</details>

**4. How Should the Responsibility Boundaries of Subagents Be Partitioned?**

<details>
<summary>💡 Show Answer</summary>

Improper Subagent partitioning is a common reason for poor performance in multi-Agent systems. Rational responsibility boundary design typically follows these principles:

- **Partitioning by Specialized Domain Rather Than by Steps**: A more effective division of labor is usually based on professional capabilities/domains (such as "Code Implementation Subagent," "Testing & Verification Subagent," and "Documentation Subagent"), rather than mechanically splitting steps in chronological order. Domain-partitioned Subagents possess a more focused toolset and context tailored to that domain, yielding higher output quality;
- **Minimizing Inter-Subagent Dependencies and Shared State**: Ideally, each Subagent should be able to complete work independently based on clear task descriptions assigned by the Main Agent, minimizing the need to frequently exchange intermediate states or wait on one another. Otherwise, efficiency gains from parallelization will be offset by coordination overhead;
- **Granting Verification/Review Subagents an Independent Perspective**: If a Subagent responsible for generating content is also tasked with reviewing its own output, a "self-confirmation" bias easily occurs (making it hard to catch its own mistakes). Therefore, a common practice is to let verification Subagents use a different context or prompt strategy than the generation Subagent, uncovering issues from a "fresher" perspective closer to third-party review. This design principle aligns with the logic of "independent configuration for the verification stage" in the reasoning sandwich tiered scheduling discussed earlier.

</details>

**5. What Is the Role of Project-Level Configuration Files Like AGENTS.md?**

<details>
<summary>💡 Show Answer</summary>

AGENTS.md (or similarly named configuration files) is an instruction document placed in the root directory of a repository, written specifically for Agents rather than human developers, used to convey project-specific conventions and context information to Agents.

**Typical Content and Roles:**
- **Project-Specific Build/Test Commands**: Explicitly informing the Agent of commands to use for compilation, running tests, and code style checking, preventing the Agent from wasting time trial-and-error searching for correct operations and directly saving many trial steps;
- **Code Style and Architectural Conventions**: Specifying code styles followed by the project, directory structure conventions, and restricted modules (which should not be modified arbitrarily), helping Agent-generated code better conform to existing project standards and reducing rework during code review;
- **Known Pitfalls and Caveats**: Documenting common pitfalls in the project (such as known issues with third-party dependencies or associated files requiring synchronized updates when a module changes), essentially crystallizing implicit team knowledge into explicit assets for Agent reference;
- **Relationship with Previously Mentioned Agent Skills**: AGENTS.md can be viewed as a project-level, more focused Skill document. Both reflect the common engineering philosophy of "solidifying experience into reusable external document assets rather than relying on on-the-spot exploration by the model." As the proportion of Agents participating in actual software development grows, such configuration files are becoming a de facto standard convention in software projects.

</details>

**6. How Is the Auto-Review Mechanism Typically Designed in Agent Products?**

<details>
<summary>💡 Show Answer</summary>

Automated code review is a key step to ensure the quality of Agent-generated code, usually designed independently from code generation itself:

- **Independent Review Context**: As mentioned earlier, the review phase best uses a different context/perspective from the generation phase to avoid the self-confirmation bias of "acting as both player and referee";
- **Multi-Level Review Standards**: Ranging from syntax/compilation correctness and unit test passing status, to code style consistency, and up to higher-level design rationality (whether unnecessary complexity is introduced, whether a simpler implementation exists). These form multiple levels transitioning from high automation to lower automation with progressively deeper review depth. The earlier levels can be fully automated, while levels involving design trade-offs may still require human review or judgment from stronger models;
- **Combining Real Execution Results**: Compared to relying solely on a model "reading code to judge right or wrong," incorporating outputs from actual test runs and static analysis tools (linters, type checkers) as review criteria provides more objective and reliable signals than subjective model judgment. This aligns with the previously discussed idea of "using executable verification signals rather than model self-judgment for quality assurance";
- **Review-Result-Driven Refinement Loop**: Upon uncovering issues during review, the system must design a proper feedback mechanism to relay specific issue descriptions (rather than generic "there are errors") back to the modifying Agent, forming a "generate-review-refine" iterative loop until review passes or the retry limit is reached.

</details>

**7. How Are Agent Execution Sandboxes and Security Isolation Typically Designed?**

<details>
<summary>💡 Show Answer</summary>

Coding Agents frequently need to execute code, modify file systems, and run terminal commands. Such high-risk operations must be conducted in isolated environments:

- **Containerized Isolation**: Each Agent session/task runs in an independent container or virtual machine, restricting access to the file system scope assigned to that task. Even if the Agent performs destructive actions, the impact is confined within the sandbox without affecting the host system or other parallel tasks;
- **Network Access Control**: Restricting by default or finely controlling the network access scope of the sandbox environment (e.g., only allowing access to specified code hosting platforms or package manager mirror sources) to prevent Agents, manipulated by malicious prompt injection, from exfiltrating sensitive data to external servers or downloading and executing unvetted malicious code;
- **Resource Quota Limits**: Setting resource caps on CPU, memory, disk, and execution duration for the sandbox environment to prevent Agents from consuming resources indefinitely due to entering infinite loops or flawed logic;
- **Controlled Merging of Artifacts**: Modifications completed by Agents inside the sandbox (such as code changes) are generally not directly applied to production environments or main branches without review. Instead, they are submitted in reviewable, rollback-capable forms like diffs or Pull Requests, preserving a final human approval step. This echoes the safety principle of "designing explicit approval mechanisms for irreversible operations" discussed earlier.

</details>

**8. What Is the Relationship Between Agent Systems and the Model Context Protocol (MCP)?**

<details>
<summary>💡 Show Answer</summary>

MCP is an open protocol recently widely adopted for standardizing "how models/Agents communicate with external tools and data sources," which can be analogized to the "USB interface standard" of the Agent era.

**Relationship with Agent Systems:**
- **Decoupling Agent Core from Tool Implementations**: Prior to a unified protocol, whenever integrating a new tool/data source, Agent products often had to develop custom adapter code specifically for that tool. With MCP defining standardized interface specifications, any MCP Server implemented following the protocol can be directly accessed by any MCP-compliant Agent client, eliminating the need to customize adaptation logic for each tool individually;
- **Ecosystem-Driven Tool Expansion**: This standardization enables third-party developers to independently build and release MCP Servers (such as various services in connector directories discussed earlier). Consequently, the capability boundaries of Agent products can rapidly expand by connecting to existing MCP Servers in the ecosystem, rather than relying solely on the product team to develop all integrations;
- **Relationship with the Tool Registry**: The tool registry in an Agent Harness is typically the concrete implementation layer managing and scheduling various tools integrated via MCP (and other means). The MCP protocol resolves the interoperability problem of "how tools are standardizedly described and invoked," whereas the tool registry resolves the scheduling problem of "which integrated tools should be exposed to the model during a specific task." The two are complementary rather than overlapping.



![连接智能体客户端与外部工具和上下文服务器的 MCP 客户端-服务器架构 (MCP client-server architecture bridging agent clients with external tool and context servers.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig128_agent_mcp.png)

</details>

**9. How Are Checkpoint and Recovery Mechanisms Designed for Long-Horizon Agent Tasks?**

<details>
<summary>💡 Show Answer</summary>

For long-horizon tasks that may run for tens of minutes or even hours, the system must cope with interruptions (network failures, process restarts, manual user pauses) and support resuming from the point of interruption, rather than restarting from scratch every time.

- **Phase-Based State Persistence**: Persisting the current session state, executed action history, and intermediate artifacts at key nodes during task execution (such as after completing each subtask or at fixed time intervals), rather than maintaining state purely in memory;
- **Idempotency Design**: Because resuming execution may re-trigger operations that were already executed, tool calling and state update logic should be designed to be idempotent where possible (repeated executions produce no side effects or erroneous results), or possess the capability to detect whether "this step has already been completed" upon resumption, preventing data inconsistency caused by repeated execution;
- **Context Reconstruction Upon Resumption**: When resuming from a checkpoint, persisted historical states need to be reloaded as a context comprehensible to the model (possibly combining the context compression mechanism discussed earlier to load summaries rather than complete raw histories), ensuring the model can "continue" previous progress without cognitive disconnection;
- **Integration with Observability**: The checkpoint mechanism also provides natural data anchors for the observability framework—each checkpoint can serve as a reference node for auditing and troubleshooting, facilitating analysis of which phase a task encountered issues in.

</details>

**10. What Do Harness Engineering Experiments (e.g., Using Agent Teams for Large-Scale Code Development) Demonstrate?**

<details>
<summary>💡 Show Answer</summary>

Recently, several industry practical reports and experiments have emerged regarding "using Agents (in conjunction with meticulously designed harnesses and workflows) to complete large-scale, long-cycle software engineering tasks." Such practices reveal several key takeaways:

- **Scalable Collaboration Requires Engineered Processes Rather Than Relying Solely on Model Autonomy**: Completing large-scale code development tasks relies heavily on meticulously designed engineering processes such as task decomposition, code review, and integration testing (i.e., the aforementioned harness and multi-Agent collaboration architectures), rather than simply handing off a massive task to a single Agent for unguided execution;
- **Continuous Human Oversight and Milestone Acceptance Remain Essential**: Even if Agents perform most of the concrete coding work, human oversight and gatekeeping at critical architectural decision points and milestone acceptance steps remain essential for ensuring output quality and correct direction. "Completely unsupervised autonomous development" remains unrealistic at the current stage;
- **Demonstrated That Harness/Process Design Is Equally Decisive for Ultra-Large-Scale Tasks**: Such experiments further substantiate the core point repeatedly emphasized earlier—when facing complex, long-cycle tasks, the engineering system design surrounding the model (how to decompose, how to verify, and how to coordinate multiple Agents) is equally important as, or even more critical than, underlying model capabilities in determining the final result.

</details>

**11. What Is the Workflow Paradigm of an "Issue-Tracker-Triggered Agent"?**

<details>
<summary>💡 Show Answer</summary>

This is a typical application scenario for event-driven Agent workflows, seamlessly embedding Agents into existing software engineering processes.

**Typical Workflow:**
1. A team member creates a new issue in an issue tracking system (such as GitHub Issues or Jira) or labels an issue with a specific tag (such as "agent-ready");
2. This event triggers the pre-configured Agent to automatically launch and read the full issue description, relevant codebase context, and project conventions such as the aforementioned AGENTS.md;
3. The Agent independently completes the entire workflow of problem analysis, code modification, and testing verification, submitting the results as a Pull Request. The PR description typically includes the Agent's understanding of the issue, adopted modification plans, and test results;
4. Human developers review the submitted PR and can merge it directly or append comments under the PR requesting further modifications by the Agent (the Agent automatically re-engages upon listening to new comments).
**Value:** This paradigm seamlessly embeds Agents into a team's existing collaboration tools and workflows without requiring developers to learn new interaction interfaces, while naturally preserving the human gatekeeping step of PR reviews. It represents one of the mature and widely accepted patterns for landing Agentic Coding into real-world team collaboration scenarios.



![由代码库问题跟踪器触发的自动化智能体端到端工作流 (End-to-end workflow of an automated agent triggered by repository issue trackers.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig131_issue_tracker_agent.png)

</details>

**12. In Agent Product Design, When Should Workflows Be Hardcoded Rather Than Fully Delegating Decision-Making Authority to the Model?**

<details>
<summary>💡 Show Answer</summary>

This is a direct reflection of the Loop Engineering concept at the product design level. The core evaluation criterion is the **structural determinism** of the task itself:

- **When Task Steps Are Enumerable and Workflows Are Relatively Fixed, They Should Be Hardcoded into Deterministic Code**: For example, established workflows such as "tests and code linters must be run before committing code." Enforcing these directly via code is more reliable, faster, and less expensive than relying on the model to "remember" calling tools in the correct order every time. Model uncertainty in such scenarios brings only risks without additional benefits;
- **When Tasks Require Open-Ended Exploration and Paths Cannot Be Known in Advance, Decision-Making Should Be Delegated to the Model**: For instance, "diagnosing an online failure of unknown root cause." Exhaustively enumerating which log to check first or which module to suspect into a fixed procedure beforehand is difficult. Such tasks are better suited for letting the model autonomously decide the next investigation direction based on currently observed information;
- **A Compromise Solution: Local Autonomy Within a Structured Framework**: A more common practice combines both—using state machines or flowcharts to define macro phases of a task (such as the graph/flow orchestration patterns discussed earlier), while leaving concrete execution details within each phase to the model's autonomous handling. This ensures both controllability and predictability of the overall workflow while retaining the flexibility required to handle local uncertainty, representing the actual design philosophy adopted by most mature Agent products today.

</details>

**13. What Are the Common Strategies for Cost and Latency Optimization in Agent Products?**

<details>
<summary>💡 Show Answer</summary>

As the number of steps involved in Agent tasks increases, cumulative token consumption and latency scale significantly. Cost and latency optimization remain an ongoing topic in Agent product engineering:

- **Model Routing**: Similar to the reasoning sandwich idea discussed earlier, dynamically selecting model/compute tiers based on the complexity and importance of each step, rather than using the strongest (and most expensive, slowest) model throughout;
- **Parallel Execution**: As in the multi-Agent parallel collaboration architecture mentioned earlier, executing parallelizable subtasks concurrently rather than sequentially waiting, trading concurrency for reduced end-to-end latency;
- **Context Management and Compression**: Utilizing context compression mechanisms discussed earlier to reduce token volume processed per inference step, lowering both single-step latency and single-step cost;
- **Result Caching and Reuse**: For recurring sub-problems or tool invocations (such as exploration results of the same repository structure), caching existing results to avoid redundant computations;
- **Early Failure and Fast Feedback**: Combining loop detection and circuit breaker mechanisms discussed earlier to identify and terminate task branches doomed to failure or stymied in deadlocks early, avoiding continuous resource consumption on low-value paths. This is an easily overlooked yet significantly impactful aspect of cost optimization—in many cases, cost waste stems not from "normal task processing," but from abnormal loops that were not timely terminated.

</details>

**14. How Do Agent Products Design Cross-Session Persistent Memory Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Different from context management within a single session, cross-session memory refers to an Agent's ability to "remember" user preferences, project backgrounds, and historical decisions revealed across multiple prior, independent interactions, automatically leveraging this information in new sessions without requiring users to explain them repeatedly.

- **Memory Extraction and Consolidation**: The system needs to identify which information from historical session logs is worth consolidating into long-term memory (such as user tech stack preferences and project architectural conventions). This typically requires an independent summarization step running asynchronously after session completion, rather than being performed synchronously during real-time dialogue, avoiding negative impacts on interaction response speeds;
- **Memory Retrieval and Injection**: At the start of a new session, relevant historical memory snippets must be retrieved based on current context relevance and injected into system prompts or context, rather than indiscriminately stuffing all historical memories into context (which causes the context bloat problem discussed earlier). This retrieval process is inherently analogous to the RAG retrieval mechanism discussed earlier;
- **Memory Update and Conflict Resolution**: User preferences and project conditions evolve over time. The system must be capable of identifying and updating outdated memories (e.g., a user previously preferred a particular tech stack but later explicitly indicated a switch), preventing the Agent from relying long-term on invalidated information when making decisions. This is a relatively tricky and easily overlooked aspect of memory system design;
- **Balancing User Control**: A well-designed memory system typically provides users with the ability to view, edit, and delete stored memories, rather than managing them entirely as a black box. This both respects user privacy and autonomy and helps users proactively correct information that the system may have misremembered or that has become outdated.

</details>

**15. What issues need to be considered in Agent product session state management when supporting simultaneous access from multiple devices/clients?**

<details>
<summary>💡 Show Answer</summary>

Modern Agent products often allow users to access the same ongoing task via multiple clients such as web browsers, desktop apps, and mobile devices. Session state management needs to handle the resulting concurrency and consistency issues.

- **Single source of truth for state**: The authoritative state of a session should be centrally maintained on the server side (rather than scattered across local clients). Each client serves merely as a presentation and interaction entry point for the state, avoiding situations where task progress appears inconsistent across different devices;
- **Conflict resolution for concurrent operations**: When a user pauses a task on one device while attempting to resume or continue operations on another, the system must have explicit arbitration rules (such as taking the latest operation as authoritative, or prompting the user to confirm a conflict), preventing unexpected state corruption;
- **Real-time synchronization mechanism**: Intermediate progress generated during Agent execution (such as completing a subtask or generating a block of code) needs to be pushed and synchronized to all currently connected clients in near real-time. This ensures that users see the latest state regardless of which device they check, typically relying on the event stream push mechanism in the protocol layer mentioned earlier;
- **Decoupling long-running tasks from client disconnections**: A user closing a client (such as closing a browser tab) should not cause an active long-running task in the background to be interrupted. The execution lifecycle of a task should be independent of the persistence of any single client connection. This is also an important scenario that the aforementioned checkpointing and recovery mechanism needs to support—tasks continue to run in the background without active client connections, and users can reopen any client later to seamlessly resume viewing and interacting with them.

</details>

