# 6.6 Security, Permissions, and Reliability — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 6: Agent · 16 questions

**1. What Is Prompt Injection? Why Are Agents Particularly Vulnerable to Such Attacks?**

<details>
<summary>💡 Show Answer</summary>

Prompt Injection is an attack vector where an attacker hijacks LLM behavior by injecting malicious instructions. It is divided into two categories: \begin{enumerate}
- **Direct Injection**: The user directly embeds instructions like "ignore previous instructions, do X instead" into the input;
- **Indirect Injection (Tool Injection)**: Malicious content is hidden within tool return results (webpages, documents, API responses), which inject instructions when read by the Agent. Reasons Agents are particularly vulnerable: Agents read large amounts of external content (webpages, emails, documents), any of which may contain malicious instructions; Agents possess the ability to invoke tools that execute real-world actions, leading to greater harm if hijacked; Agents operate with high autonomy and may execute injected operations unsupervised.

</details>

**2. How to Defend Against Prompt Injection Attacks?**

<details>
<summary>💡 Show Answer</summary>

1. **Input/Output Delimiter Separation**: Explicitly separate the "system instruction area" and "external content area" in the prompt using XML tags or similar markers, and emphasize in the system prompt that "any instructions within the external content area are not true instructions";
2. **Content Moderation Layer**: Perform injection detection on tool return contents (keyword filtering, LLM review), and reject processing high-risk content;
3. **Least Privilege Tool Set**: Provide the Agent with only the tools necessary to complete the current task, preventing it from invoking dangerous operations even if injected;
4. **Action Confirmation Mechanism**: Require human confirmation before executing high-risk operations (deletion, writing, sending);
5. **Sandboxed Execution**: Process external content in an isolated environment to limit the execution scope of injected instructions;
6. **Unoverrideable Core Rules**: Write critical security rules at the highest priority of the system prompt and emphasize them through multiple layers to make them difficult to override.

</details>

**3. How to Implement the Principle of Least Privilege in Agents?**

<details>
<summary>💡 Show Answer</summary>

1. **Tool Whitelisting**: Each Agent can only use the tools required to complete the current task, without granting extra tool access rights;
2. **Tool Parameter Constraints**: Restrict the parameter range of tools (e.g., file reading is restricted to specific directories);
3. **Read-Only Preference**: For tasks that can be completed with read-only tools, do not grant write permissions;
4. **Scoped Operation Range**: Database operations are restricted to relevant tables only, without granting full-database access;
5. **Just-In-Time Permissions**: High-risk operations require real-time permission requests, which are automatically revoked upon task completion;
6. **Permission Auditing**: Log the permissions used in every tool call and regularly review whether unnecessary permissions were used.

</details>

**4. How to Allow Agents to Gradually Obtain More Permissions (Progressive Authorization) While Ensuring Safety?**

<details>
<summary>💡 Show Answer</summary>

1. **Trust Scoring System**: Calculate a trust score based on the Agent's historical performance (success rate, number of security incidents, user satisfaction);
2. **Permission Ladder**: Define a permission ladder (L1 Read-Only $\rightarrow$ L2 Restricted Write $\rightarrow$ L3 Full Write $\rightarrow$ L4 High-Risk Operations), where upgrading requires reaching a trust score threshold;
3. **Sandbox Verification**: Test new permissions in a sandbox environment first, granting production permissions only after passing verification;
4. **Human Approval Gate**: Upgrading from L2 to L3 must undergo human approval; fully automatic escalation is not allowed;
5. **Permission Usage Monitoring**: Continuously monitor the usage of each permission tier, revoking permissions or downgrading immediately upon detecting anomalies;
6. **Maximum Permission Ceiling**: Even with an extremely high trust score, certain high-risk operations (bulk deletion, fund transfer) always require human confirmation and cannot be automatically authorized.

</details>

**5. How to Evaluate the Overall Security Risk of an Agent System?**

<details>
<summary>💡 Show Answer</summary>

Evaluation Framework:
1. **Attack Surface Analysis**: List all possible attack entry points (user input, external content, tool return values);
2. **Tool Risk Rating**: Grade each tool by severity of harm and reversibility (low/medium/high/critical);
3. **Prompt Injection Testing**: Conduct injection testing on all external content reading paths;
4. **Privilege Escalation/Bypass Testing**: Test whether the Agent can bypass permission limits through combinations of tool calls;
5. **Worst-Case Analysis**: Assume the Agent is completely hijacked and analyze the maximum potential damage an attacker could cause;
6. **Compliance Auditing**: Audit whether data handling complies with regulatory requirements such as GDPR and SOC2;
7. **Red Teaming**: Have security experts act as attackers to proactively discover vulnerabilities;
8. **Risk Matrix**: Plot a risk matrix based on Probability $\times$ Impact, prioritizing fixes for high-risk items.




![Illustration of How to Evaluate the Overall Security Risk of an Agent System?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig49_evaluate_overall_security_risk.png)

</details>

**6. How to Ensure an Agent's Behavior is Safe, Controllable, and Aligned with Human Intent? What Are the Alignment Guarantee Methods?**

<details>
<summary>💡 Show Answer</summary>

The main alignment guarantee methods include:

1. **Core Model Alignment**: Fine-tune the foundation LLM using techniques such as RLHF, DPO, and Constitutional AI so that it adheres to the "Helpful, Honest, and Harmless" principles.
2. **Least-Privilege Tool Management**: Grant the Agent only the minimum set of tools and permissions required to complete its task, define tool whitelists, and restrict access to the file system, database, etc.
3. **Human-in-the-Loop (HITL)**: Require explicit human approval prior to executing high-risk or irreversible operations (file deletion, financial transactions); allow humans to monitor in real time and intervene at any moment.
4. **Execution Environment Sandboxing**: Execute code within controlled sandboxes such as Docker containers or virtual machines to limit the blast radius of potential harm.
5. **Explicit Rules and Guardrails**: Incorporate hardcoded rules or output filters into the Agent's control logic to block specific dangerous operations.
6. **Continuous Red Teaming and Auditing**: Deploy dedicated teams to attack the Agent from various angles; record detailed logs of all action trajectories for post-hoc auditing.

</details>

**7. How to Strike a Balance Between "Agent Autonomy" and "Human Control"?**

<details>
<summary>💡 Show Answer</summary>

**Core Contradiction**: Higher autonomy leads to higher efficiency but greater risk; stronger controllability ensures better safety but degrades user experience. **Balancing Strategies:** \begin{enumerate}
- Tiering by Action Impact: Read-only operations are fully autonomous $\rightarrow$ Reversible write operations notify after execution $\rightarrow$ Irreversible operations require confirmation before execution $\rightarrow$ High-risk operations always require human approval;
- Progressive Authorization: New Agents start with restricted permissions and gradually expand permissions after building up a trusted track record;
- Trading Transparency for Autonomy: Allow users to inspect every decision step of the Agent, leveraging transparency to gain user acceptance of autonomy;
- Circuit Breaker Mechanism: Automatically downgrade to a conservative mode upon detecting anomalous behavior;
- User Preference Settings: Allow users to select autonomy levels (Conservative/Balanced/Aggressive) based on their risk appetite;
- Learning-Based Permissions: If an Agent consistently performs well on a specific category of operations, the confirmation requirement for that category is automatically lowered.

</details>

**8. In Which Scenarios Are Agent "Hallucinations" Most Harmful? How to Mitigate Them at the Architectural Level?**

<details>
<summary>💡 Show Answer</summary>

**Most Harmful Scenarios:** \begin{enumerate}
- Tool Parameter Fabrication (fabricating non-existent API parameter values, leading to invocation failure or unintended results);
- Misinterpretation of Tool Return Results (making erroneous factual interpretations of tool outputs, causing subsequent steps to be based on false assumptions);
- State Hallucination (mistakenly assuming a step has been completed and skipping its actual execution);
- Permission Hallucination (mistakenly assuming possessing a certain permission and attempting to execute unauthorized operations).
	**Architectural Mitigations:** 
- Strict Tool Parameter Validation: Schema verification to reject invalid parameters;
- Result Grounding: Subsequent steps must explicitly reference content returned by tools, disallowing ungrounded "knowledge";
- Explicit State Management: Use structured state objects to track progress rather than relying on LLM memory;
- Critical Step Verification: Require the LLM to restate its execution plan and confirm before executing irreversible operations;
- External State Verification: Periodically query actual status using tools instead of relying on LLM memory of history.

</details>

**9. When an Agent Needs to Execute Tasks in a Real or Simulated Environment (e.g., Robotics, Games), What Is Its Fundamental Difference from a Purely Software-Tool-Based Agent?**

<details>
<summary>💡 Show Answer</summary>

For Embodied Agents entering real or simulated physical environments, the fundamental differences lie in: \begin{enumerate}
- **Perception**: Software Agents receive structured JSON data; Embodied Agents receive high-dimensional, noisy sensor data (pixel streams, point clouds) and must solve the symbol grounding problem;
- **State Observability**: Software Agents are typically fully observable; Embodied Agents are only partially observable and must infer state from incomplete histories;
- **Action Uncertainty**: Software Agent operations are deterministic; Embodied Agent physical executions exhibit stochasticity, requiring sensor feedback to confirm each action;
- **Real-Time Requirement**: Software Agents can perform asynchronous reasoning; Embodied Agents must respond in real time to dynamic environments;
- **Safety and Irreversibility**: Software Agent errors are typically reversible; Embodied Agent errors may cause physical damage, making safety the primary consideration.




![Illustration of When an Agent Needs to Execute Tasks in a Real or Simulated Environment (e.g., Robotics, Games), What Is Its Fundamental Difference from a Purely Software-Tool-Based Agent?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig50_when_agent_needs_execute.png)

</details>

**10. Why Are Agents Prone to Falling into a "Repeated Try-Fail" Infinite Loop?**

<details>
<summary>💡 Show Answer</summary>

The repeated try-fail infinite loop is one of the most frustrating failure modes in Agent systems. The root cause lies in the fact that the "failure" event is not effectively converted into actionable information to guide subsequent decision-making.

1. **Absence of Failure Memory**: If the system does not explicitly log "what plans were previously attempted and why they failed," the model effectively remakes decisions in an "amnesic" state during each retry—it may arrive at the same (erroneous) action plan again based on the same (erroneous) reasoning logic, because the root cause of the previous failure was never incorporated into the scope of decision-making.
2. **Injection of Explicit Failure History**: The core of the solution is to maintain a structured list of "attempted plans and failure causes." Prior to each new decision, this list is explicitly injected into the context ("Attempted Plan A: Failed due to parameter format error; Attempted Plan B: Failed due to insufficient permissions"), forcing the model to take these historical lessons into account when generating new plans rather than recreating a potentially duplicate plan out of thin air.
3. **Introduction of Differentiation Requirements**: Explicitly require in the prompt that "the new trial plan must differ substantially from previous failed plans," guiding the model to proactively search for alternative resolution paths rather than homogeneous repeated attempts with minor adjustments (such as merely altering parameter wording while keeping the essential logic unchanged).
4. **Categorization and Escalation of Failure Modes**: When the failure reasons of multiple consecutive attempts belong to the same category (e.g., repeatedly failing due to permission issues), it indicates that the problem may not lie in the "specific plan attempted" but in a more fundamental obstacle (such as genuinely lacking required permissions). At this point, the system should cease blindly trying new plans and instead handle the root obstacle or escalate to human intervention, rather than endlessly varying execution approaches on the same class of issue.
5. **Hard Loop Detection**: As a final safety net, the system should be able to detect the pattern of "repeating semantically similar attempts" (even if specific parameters slightly differ). Upon detecting such a loop tendency, it forcibly interrupts the execution and triggers multi-level handling strategies, rather than relying on the unreliable judgment of the model "realizing it is stuck in a loop."

</details>

**11. How Should the Exception Handling Flow Be Designed for Agents?**

<details>
<summary>💡 Show Answer</summary>

Designing the exception handling workflow for an Agent requires layered management of exceptions of varying sources and severity levels, forming a comprehensive fault-tolerance system rather than a simple try-catch wrapper.

1. **Tool Layer—Error Standardization**: The implementation of each tool should catch raw exceptions (network errors, API errors, data format errors) and convert them into a unified, structured error format (e.g., `\{"error_type": "timeout", "message": "...", "retryable": true\}`), enabling the LLM to comprehend the cause of the error and make rational decisions, rather than throwing hard-to-parse raw stack traces.
2. **Agent Layer—Decision Response**: After receiving a structured error, the Agent selects a coping strategy based on the error type—errors with `retryable: true` (network timeout, rate limiting) trigger automatic retries; errors with `retryable: false` (parameter errors, insufficient permissions) prompt attempts to correct parameters or switch tools; consecutive failures trigger fallback solutions or escalation to the user.
3. **System Layer—Fallback Mechanism**: Above the Agent decision layer, the system layer provides the ultimate safety net—single-step execution timeouts trigger mandatory interruption; total task execution time exceeding limits triggers mandatory termination; consecutive exception counts exceeding thresholds trigger a circuit breaker to pause the Agent instance and issue alerts, preventing exceptions from propagating endlessly and consuming resources.
4. **Graduated Exception Response**: Take responses of varying intensity based on the severity of the exception—minor exceptions (such as a single tool call failure with retry opportunities) are handled silently, requiring only log recording; moderate exceptions (multiple retries failed, requiring plan switching) log detailed context and proceed with fallback plans; severe exceptions (task cannot proceed) pause execution and proactively notify the user, accompanied by a clear problem description and partially completed results.
5. **Exception Logging and Observability**: All exceptions, whether successfully handled or not, should record complete context (Task ID, step, exception type, handling method, handling result) to provide a data foundation for subsequent problem analysis, exception pattern identification, and system optimization.




![Illustration of How Should the Exception Handling Flow Be Designed for Agents?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig51_should_exception_handling_flow.png)

</details>

**12. Why Do Agent Demos Perform Well, Yet Experience Frequent Failures After Deployment?**

<details>
<summary>💡 Show Answer</summary>

This is the most prevalent phenomenon in Agent engineering deployment. The root cause is an order-of-magnitude gap in complexity between Demo environments and production environments.

1. **Happy Path Bias**: Demos typically utilize carefully prepared "standard inputs"—clean data, complete parameters, and fully available tools. In contrast, real online user inputs are infinitely varied: ambiguous requirements, incomplete information, typos, and mixed languages. An Agent's performance under non-standard inputs is vastly different from its performance in a demo.
2. **Tool Reliability Assumptions**: Tools in demos are 100% available without timeouts, rate limits, or API version changes. In production environments, tools can suffer transient failures at any time (P99 latency spikes, occasional 5xx errors). The Agent must possess robust error handling and retry mechanisms, which are completely untested during the demo stage.
3. **Insufficient Coverage of Long-Tail Inputs**: Demos only cover the most common 5--10 task scenarios, whereas real users in production raise countless long-tail requests. Long-tail scenarios often represent blind spots in model training and prompt design, yielding failure rates far higher than core demo scenarios.
4. **Concurrency and State Contamination**: Demos are usually tested in single-user, single-threaded setups. In production, multiple Agent instances run concurrently, leading to race conditions and state contamination issues when sharing state (such as memory, databases, tool quotas).
5. **Context Accumulation Effect**: Demos typically involve short tasks (5--10 steps), while long tasks in production (50+ steps) result in context window overflow, plan drift, and state loss—issues that never surface during brief demos.
6. **Mitigation Approaches**: Conduct large-scale "stress testing" (simulating online traffic distribution), "chaos testing" (randomly injecting tool failures), and "red teaming" (manually crafting edge inputs to attempt breaking the Agent) prior to deployment, alongside building a complete observability framework to quickly discover and locate production issues.

</details>

**13. How to Enhance the Overall Stability of an Agent System?**

<details>
<summary>💡 Show Answer</summary>

Enhancing the overall stability of an Agent system is a systematic engineering effort that requires simultaneous action across multiple layers; there is no single "silver bullet" to solve all stability issues.

1. **Resource Boundary Control**: Set clear upper limits on steps, token budgets, and execution time for each task to prevent individual anomalous tasks from consuming resources indefinitely or falling into dead loops, safeguarding overall system resource availability.
2. **Tool-Layer Fault-Tolerant Design**: Configure reasonable timeout thresholds and retry strategies for all external tool calls (distinguishing between retryable and non-retryable error types), and introduce circuit breaker mechanisms to prevent an unstable dependency tool from bringing down the entire system.
3. **Structured Output Validation**: Pass all critical LLM-generated outputs (tool invocation parameters, final results) through Schema validation to reject non-compliant outputs and trigger correction, rather than allowing malformed outputs to flow downstream and cause cascading failures.
4. **Tiered Human Review Mechanism**: Establish mandatory human confirmation checkpoints for high-risk or irreversible operations as a last line of defense when automated decisions fail, balancing automation efficiency with security.
5. **Comprehensive Observability System**: Establish thorough monitoring metrics (success rate, latency, error distribution) and alerting mechanisms to continuously track system health, enabling proactive detection and intervention before problem impacts expand, rather than reactively responding to user complaints.
6. **Canary Release and Rollback Capabilities**: Validate any modification to Agent logic, prompts, or tools via canary releases; enable rapid rollback upon detecting quality degradation to avoid a single problematic change affecting all users.

</details>

**14. Why Does Agent Behavior Exhibit Randomness Under the Same Input?**

<details>
<summary>💡 Show Answer</summary>

The randomness of Agent behavior under identical inputs is an inherent characteristic arising from the probabilistic generation nature of LLMs. Understanding its sources helps implement targeted controls in scenarios requiring determinism.

1. **Core Impact of the Temperature Parameter**: When `temperature` is greater than 0, the model does not deterministically select the candidate with the highest probability at each token position, but samples according to the probability distribution. This means that even with identical inputs, outputs generated across multiple runs will vary due to sampling stochasticity. This is the primary and most easily controllable source of randomness—setting temperature to 0 (or close to 0) significantly reduces (though cannot completely eliminate) randomness.
2. **Hidden Contextual Differences**: Even if the user input text is identical, the actual complete prompt sent to the model may contain implicit differences—current timestamps, session IDs, minor order variations in dialogue history, or system-injected random elements (such as nonce values). These "seemingly irrelevant" differences also affect the model's generated output.
3. **Concurrency and System-Level Non-Determinism**: In distributed inference systems, batching strategies, numerical precision variations in GPU computations, and request routing differences caused by load balancers can introduce additional output non-determinism under extreme cases. This stems from system engineering rather than the model itself.
4. **Engineering Techniques to Reduce Randomness**: Set `temperature` to 0 to obtain near-deterministic outputs ("near" because floating-point computation non-determinism may still cause tiny variations); use the `seed` parameter to pin random seeds (supported by some APIs, though cross-hardware/version consistency is not guaranteed); leverage structured outputs (Structured Outputs/JSON Schema constraints) to constrain the variation space of free generation; apply post-processing standardization to outputs at key decision points to eliminate trivial expression differences.
5. **When Randomness Should Be Intentionally Preserved**: Not all scenarios demand total determinism—creative generation tasks may require moderate randomness to ensure diversity, and multi-path reasoning (Self-Consistency) actively utilizes multiple sampling runs to enhance overall reliability. The key is to consciously control the level of randomness based on scenario requirements, rather than strictly enforcing or ignoring it across the board.

</details>

**15. What Are the Common Methods for Improving the Task Execution Success Rate of Agents?**

<details>
<summary>💡 Show Answer</summary>

Improving an Agent's task execution success rate is a systematic engineering effort that requires comprehensive measures across task design, execution mechanisms, and continuous optimization; no single method alone can significantly boost overall success rates.

1. **Fining Single-Step Task Granularity**: Break down complex tasks into finer-grained, more explicit sub-steps, where each step has clear and simple success criteria. The finer the granularity, the higher the probability of success for each step, which correspondingly improves the overall task's "chained success rate" (the cumulative probability of all steps succeeding).
2. **Structured Output and Strict Validation**: Enforce structured formats and strict validation for all key decision points (tool invocation parameters, final outputs) to detect and correct format or logic errors early, preventing errors from accumulating and amplifying in downstream steps.
3. **Self-Checking and Reflection Mechanisms**: Introduce self-review steps at critical nodes—especially reflection combined with external verifiable signals (tool execution results, test cases)—which effectively discovers and corrects errors during execution, yielding significantly higher success rates than unchecked "one-shot generation."
4. **Human Confirmation as a Safety Net**: Introduce human confirmation checkpoints for high-risk or highly uncertain key decisions; although sacrificing a degree of automation efficiency, it effectively prevents a single automated decision error from causing total task failure.
5. **Failure-Case-Driven Continuous Optimization**: Systematically collect failure cases and analyze common characteristics of failure modes (whether tool issues, prompt design issues, or task comprehension issues), targeting the most frequent failure causes for specialized optimization, which is far more direct and effective than vaguely "improving model capability."
6. **Improving Actionability of Error Feedback**: When tool calls or intermediate steps fail, provide specific, structured, and actionable error messages so that the model can adjust strategies based on clear feedback rather than retrying blindly; this is key to enhancing the ability to "recover from failure."

</details>

**16. Where Is the Real Bottleneck of Current Agent Systems?**

<details>
<summary>💡 Show Answer</summary>

The answer to where the real bottleneck of current Agent systems lies reflects an important shift in industry perception regarding Agent technology development—gradually transitioning from "model capability-centricity" to "system engineering-centricity."

1. **Model Capability Is No Longer the Primary Bottleneck**: With continuous enhancements in the reasoning capabilities of frontier models (GPT-4, Claude, etc.), whether the model itself "can understand the task and formulate a reasonable plan" is no longer the main constraint on Agent deployment in most actual business scenarios. Model reasoning and planning capabilities are, to a considerable extent, "sufficient."
2. **The Real Bottleneck Lies in Engineering Infrastructure**: Reliability of state management (ensuring state consistency and recoverability during long-task execution), standardization of the tool ecosystem (lack of unified standards for tool interfaces, error handling, and permission models), observability frameworks (continuously monitoring and debugging Agent behavior in production), and evaluation systems (objectively measuring the true effectiveness of Agent systems)—the maturity of these engineering infrastructures severely lags behind the pace of model capability development.
3. **Implicit Costs of Massive "Dirty Work"**: Building a truly reliable production-grade Agent system requires the vast majority of effort not in "crafting clever prompts" or "selecting advanced models," but in tackling numerous tedious yet essential engineering details—error handling coverage, fallback logic for edge cases, multi-layered security defenses, and fine-grained performance tuning. This "heavy lifting" often accounts for over 70% of overall development effort and is key to determining whether the system can run stably.
4. **Industry Significance of the Bottleneck Shift**: This shift in perception signifies that competitive advantage in the Agent domain is transitioning from "whose model is smarter" to "whose engineering system is more solid and reliable." This holds important implications for team capability building and resource allocation—simply chasing the latest and strongest models while neglecting engineering infrastructure makes it difficult to construct Agent systems that operate stably in production environments.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%



![Illustration of Where Is the Real Bottleneck of Current Agent Systems?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig52_where_real_bottleneck_current.png)

</details>

