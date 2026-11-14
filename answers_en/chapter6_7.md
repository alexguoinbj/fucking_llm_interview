# 6.7 Evaluation, Observability, and Deployment — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 6: Agent · 22 questions

**1. How to Evaluate the Task Completion Quality of an Agent System?**

<details>
<summary>💡 Show Answer</summary>

1. **End-to-End Success Rate**: Use a benchmark task set to compute the proportion of fully completed tasks (meeting predetermined success criteria);
2. **Partial Completion Rate**: For complex tasks, evaluate what percentage of sub-goals were accomplished;
3. **Result Correctness**: For tasks with ground-truth answers, evaluate consistency between outputs and ground-truth answers;
4. **LLM-as-a-Judge**: Use stronger models to evaluate output quality (accuracy, completeness, relevance);
5. **Human Evaluation**: Sample task results for human experts to score, serving as a calibration benchmark for automated evaluation;
6. **User Satisfaction**: Explicit upvote/downvote feedback from real users;
7. **Business Metrics**: Impact of task completion on actual business metrics (efficiency gains, error rate reductions).

</details>

**2. What Is Step Efficiency in Agents? How Can It Be Optimized?**

<details>
<summary>💡 Show Answer</summary>

**Step Efficiency**: The number of tool invocations required to complete a task; fewer steps indicate higher efficiency (given equivalent quality). Calculation: `Actual Steps / Theoretically Optimal Steps`, where a ratio closer to 1 indicates higher efficiency. Optimization methods: \begin{enumerate}
- **Parallelization**: Combine independent tool calls into parallel execution;
- **Stronger Planning**: Superior initial planning reduces trial-and-error and backtracking;
- **Tool Combination**: Merge tools that are frequently called sequentially into a composite tool;
- **Caching**: Cache results of duplicate queries to avoid redundant tool calls;
- **CoT Quality Enhancement**: Better reasoning reduces erroneous tool calls;
- **Historical Experience Reuse**: Retrieve optimal execution paths for similar tasks from memory to reuse directly.

</details>

**3. How to Establish Observability for an Agent System?**

<details>
<summary>💡 Show Answer</summary>

Three core pillars: \begin{enumerate}
- **Logging**: Record the inputs/outputs of each LLM inference, parameters and return values of each tool invocation, state changes, and error events; use structured logging (JSON) for easy querying and analysis;
- **Tracing**: Assign a unique trace ID to each task across multiple Agents and tool calls, linking all operations into a complete call chain; tools: LangSmith, Langfuse, Honeycomb;
- **Metrics**: Task completion rate, average steps, tool call success rate, token consumption, end-to-end latency; build Dashboards for continuous monitoring. Additional elements: Anomaly alerting (automatic notifications when key metrics breach thresholds), sampling (proportional sampling of logs and traces during high traffic).

</details>

**4. How to Systematically Debug When an Agent's Execution Result Does Not Meet Expectations?**

<details>
<summary>💡 Show Answer</summary>

Debugging steps: \begin{enumerate}
- **Confirm Problem Scope**: Do all tasks fail, or only specific types of tasks? Narrow down the scope;
- **Reproduce Problem**: Reproduce the failure with identical inputs to confirm whether it is a deterministic or random issue;
- **Trace Execution Path**: Inspect the complete Thought$\rightarrow$Action$\rightarrow$Observation chain to locate the first point of deviation;
- **Troubleshoot Tool Issues**: Check whether tool calls succeeded and whether return results met expectations;
- **Examine LLM Reasoning**: Check if the LLM's Thought was reasonable and whether erroneous deductions occurred;
- **Inspect State Issues**: Check whether the Agent state experienced inconsistency at a particular step;
- **Isolation Testing**: Test suspicious steps in isolation to pinpoint which step caused the failure;
- **Comparative Testing**: Compare against similar, normally functioning tasks to identify points of divergence.

</details>

**5. How to Evaluate the Token Consumption Efficiency of an Agent?**

<details>
<summary>💡 Show Answer</summary>

1. **Token Consumption Per Task**: Average token count (input + output) consumed to complete a task; lower is better;
2. **Effective Token Ratio**: Proportion of total tokens that make substantive contributions to the task (excluding repeated history and redundant context);
3. **Step Token Distribution**: Analyze tokens consumed at each step to identify steps with anomalous consumption;
4. **Redundancy Detection**: Check whether the context contains massive duplicate information (such as tool return results being included verbatim multiple times);
5. **Baseline Comparison**: Compare token consumption against an optimal baseline (human-designed shortest execution path) to calculate the efficiency ratio;
6. **Optimization ROI**: Quantify token savings for each optimization scheme (history compression, parallelization, caching), prioritizing implementations with the highest return.

</details>

**6. How to Design Automated End-to-End Testing for an Agent System?**

<details>
<summary>💡 Show Answer</summary>

1. **Test Task Suite**: Build a standard task suite covering varying complexities and types (simple single-step tasks, multi-step tasks, tasks requiring error recovery);
2. **Real Tools vs. Mock Tools**: Use Mock tools for core flow testing (controllable, fast, side-effect free) and real tools for integration testing (end-to-end verification);
3. **Success Criteria Definition**: Define explicit success criteria for each test task (output format, presence of key fields, business logic correctness);
4. **Automated Execution**: Integrate into CI/CD pipelines to automatically run the test suite upon every code change (Agent logic, tool implementation, prompts);
5. **Failure Reporting**: Output complete execution trajectories (every step's T-A-O) upon test failure to facilitate rapid diagnosis;
6. **Performance Baselines**: Log historical step counts and token consumption for each test case to compare during regression testing, preventing efficiency regression.




![Illustration of How to Design Automated End-to-End Testing for an Agent System?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig53_design_automated_testing_agent.png)

</details>

**7. How to Evaluate the Performance of Different Agent Frameworks Using Benchmarks?**

<details>
<summary>💡 Show Answer</summary>

Evaluation Dimensions: \begin{enumerate}
- **Task Suite Selection**: Use widely recognized benchmarks (WebArena, SWE-bench, GAIA, AgentBench) covering multiple task types;
- **Success Rate**: End-to-end success rate on standard task suites, serving as the primary quality metric;
- **Efficiency Metrics**: Average step count, average token consumption, average completion time;
- **Reliability**: Consistency (variance) of success rates across repeated runs of identical tasks;
- **Safety**: Proportion of instances breached by prompt injection testing;
- **Cost**: Average API cost per task;
- **Engineering Factors**: Framework usability, extensibility, and community activity. Combine multiple dimensions to draw radar charts for comparison and make selection decisions based on specific scenario weights.

</details>

**8. When Building a Complex Agent, What Do You Consider to Be the Primary Challenges?**

<details>
<summary>💡 Show Answer</summary>

1. **Robustness of Planning and Reasoning**: Agents easily get "lost" during long tasks—forgetting goals, falling into ineffective loops, or suffering complete task failure due to a single step error, lacking strong error-correction and dynamic re-planning capabilities.
2. **Reliable and Reproducible Evaluation**: Complex open-ended tasks lack unique correct answers, making evaluation metrics difficult to define, and execution results involving dynamic tools hard to reproduce; human evaluation remains the most reliable method currently, which is difficult to scale.
3. **Cost, Latency, and Scalability**: Complex tasks may require dozens of LLM calls; using strong models drives single-task costs up to several dollars with unacceptable latency, making large-scale deployment impractical.
4. **Safety and Controllability**: Permission management difficulties (how to prevent dangerous operations), prompt injection attacks, and behavioral unpredictability are core obstacles to whether Agents can be trusted and deployed at scale.

</details>

**9. During Agent Deployment, Which Is Harder to Master: Planning Capability or State Management Capability?**

<details>
<summary>💡 Show Answer</summary>

Comparing the difficulty of planning capabilities versus state management capabilities reveals two distinct types of engineering challenges in Agent systems. Understanding this distinction helps rationally prioritize engineering investment when resources are constrained.

1. **Characteristics of Planning Errors—Explicit and Easy to Detect**: Issues of poor planning quality typically manifest quickly—generated plans are infeasible, or execution visibly deviates from goals. Such problems are relatively easy to observe (whether in development testing or production monitoring), and mitigation paths are relatively clear (optimizing prompts, improving planning algorithms, introducing reflection mechanisms), representing an "explicit" engineering challenge.
2. **Characteristics of State Management Issues—Implicit and Hard to Pinpoint**: State management issues (such as state drift, duplicate execution, concurrency conflicts, checkpoint corruption) rarely surface as immediate explicit errors; instead, they accumulate silently until exploding in unexpected ways during later stages of long-task execution (task hangs, inconsistent results, executing an operation twice). The root cause of such issues often involves state changes several steps prior, making diagnosis and reproduction far harder than planning-level issues.
3. **Engineering Risk Underestimation Due to Covertness**: Precisely because state management issues are unlikely to fully expose themselves during development and testing (short tasks and simple test cases rarely trigger deep state consistency bugs), teams tend to underestimate the engineering investment needed. When issues erupt under long tasks and high concurrency in production, diagnosing and fixing them costs far more than anticipated.
4. **Mutual Influence Between the Two**: State management defects sometimes present under the guise of "planning errors"—for example, because state reading retrieved stale data, planning was based on false premises. In this case, the surface symptom is "poor planning quality," but the root cause lies in state management, further complicating problem diagnosis. This "disguise" characteristic is one reason state management issues are often underestimated.
5. **Practical Recommendations**: From early project stages, equal or even higher engineering investment priority should be given to state management compared to planning capability. In particular, design robust state consistency guarantees, checkpoint mechanisms, and observability means for long tasks and high concurrency scenarios, rather than reactively patching after production issues occur.

</details>

**10. Why Do Agents Frequently Get Stuck at Intermediate Steps When Executing Multi-Step Tasks?**

<details>
<summary>💡 Show Answer</summary>

Getting stuck at intermediate steps during multi-step task execution is an extremely common failure mode in Agent engineering practice, typically stemming from fragile inter-step handoff design rather than insufficient reasoning capability of the model itself.

1. **Incompatible Formats Between Steps**: The output format of a preceding step is inconsistent with the expected input format of a subsequent step (e.g., the former returns a JSON with extra fields while the latter's parsing logic strictly demands a specific Schema). This "interface mismatch" prevents subsequent steps from properly processing inputs, causing hangs or cascading errors.
2. **Lack of Handling Logic After Tool Call Failures**: When a tool call relied upon by a step fails (network timeout, API rate limit) and the system lacks retry or fallback logic designed for such scenarios, the Agent may enter a stalled state of "not knowing what to do next"—neither able to proceed (data not retrieved) nor equipped with clear failure handling to gracefully terminate or switch to backup plans.
3. **"Getting Lost" Due to Erroneous State Judgment**: In multi-step tasks with long contexts, lacking an explicit state tracking mechanism may lead the model to misjudge "which step it should currently be at" due to context compression, information forgetting, or comprehension bias. This causes duplicate execution of completed steps or skipping of required steps, making the task appear "stuck" or behave abnormally.
4. **Absence of Completion Criteria**: If a step lacks clear, objective "completion criteria" (relying on subjective model judgment like "this should be done now"), two opposite issues may arise: the model believing the step is incomplete and executing repeatedly, or believing it is complete and advancing prematurely resulting in incomplete data. Both manifest as symptoms of being "stuck in the middle."
5. **Solutions**: Define clear input/output Schema contracts for each step to ensure upstream and downstream interface compatibility; design explicit failure handling paths for key tool calls; maintain explicit state objects to track current progress rather than relying on implicit model understanding of history; set objective and verifiable completion criteria for each step.

</details>

**11. How Does an Agent Determine That a Task Is Completed and Execution Should Stop?**

<details>
<summary>💡 Show Answer</summary>

Determining when a task is completed is an easily underestimated yet vital aspect of Agent system design. Improper design leads to two opposite issues: premature task termination (incomplete results) or inability to terminate (resource waste, poor user experience).

1. **Structured Goal Validation (Most Reliable Method)**: Define explicit, objectively verifiable completion criteria at the start of a task (e.g., "output must contain fields X, Y, Z with values in a reasonable range," "tool A must be successfully invoked with a non-null return"). Continuously check during execution whether current states meet these predefined conditions, declaring completion once satisfied. This method does not rely on subjective model judgment and offers the highest reliability.
2. **Dedicated Completion Assessment Step**: For complex tasks where completion criteria are hard to define with simple rules, design a dedicated "Evaluator Agent" or assessment step independent of the main task execution flow, focusing exclusively on evaluating "whether current outputs sufficiently answer the user's original request." This separation of roles (executor does not judge its own completion) reduces self-evaluation bias.
3. **Progress Checklist Verification for Multi-Step Tasks**: Decompose the task into an explicit checklist of subtasks, each with an independent completion marker. Overall task completion judgment is based on the objective check of "whether all mandatory subtasks are marked as complete," rather than a holistic, vague judgment.
4. **Unreliable Method—Purely Relying on Model Self-Judgment**: Merely asking the model to output "task completed" when it deems itself done is purely subjective and prone to two failure modes: the model believing it is done prematurely (actually missing key steps) or being overly cautious and reluctant to declare completion (even when standards are met). Both are unreliable mechanisms.
5. **Necessity of Fallback Mechanisms**: Regardless of which primary judgment method is adopted, set maximum step limits and timeout mechanisms as a final safety net—even if primary judgment logic fails to correctly identify completion status, the system can forcefully terminate within reasonable resource consumption caps, avoiding resource waste caused by indefinite execution.




![Illustration of How Does an Agent Determine That a Task Is Completed and Execution Should Stop?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig54_agent_determine_that_task.png)

</details>

**12. Why Does an Agent Execute Wrong Operations Even When It Already "Knows" the Correct Answer?**

<details>
<summary>💡 Show Answer</summary>

The phenomenon where an Agent "knows the correct answer yet executes the wrong operation" reveals the fundamental gap between an LLM as a "language generation system" and an "execution system." This gap between knowledge and action is far more worthy of deep understanding than merely assuming "the model is not smart enough."

1. **Separation of Linguistic Expression and Structured Execution**: A model "knowing" the correct answer at the natural language level means it can accurately describe in text what ought to be done; however, converting this "knowing" into an actual tool call (specific function names, strictly formatted parameters) requires a translation process from "natural language to structured instructions." This translation process itself carries error potential—even with perfect semantic understanding, minor errors in parameter formats or field names cause execution results to diverge from the "known correct answer."
2. **Cumulative Risk Across Multi-Stage Execution Chains**: From "model generating intention" to "actual operation taking effect," the path passes through parameter construction, Schema validation, actual tool invocation, and result parsing. Each stage bears independent error probability. Even if initial model judgment is completely correct, an execution deviation in any link will cause final behavior to veer off the "known correct path."
3. **Inconsistency Between Contextual State and Runtime State**: A model makes decisions based on its "understood" current state (deeming a certain operation appropriate). However, if this understood state deviates from the actual system state at runtime (state drift), even sound decision logic based on stale state can result in "wrong operations" when executed in the actual environment.
4. **Solution Direction—Consistency Verification Mechanisms**: Introduce explicit consistency verification checkpoints between "knowing" and "executing"—prior to execution, require the model to restate the specific upcoming operation (forcing implicit "knowing" to become an explicit, verifiable execution plan), comparing the execution plan against the original correct judgment and triggering correction upon detecting inconsistencies rather than executing directly, bridging the gap between knowledge and action at the architectural level.

</details>

**13. Why Are Agents Prone to Omitting Critical Steps During Execution Flows?**

<details>
<summary>💡 Show Answer</summary>

Omitting critical steps is a common quality issue in long-task Agents, with root causes primarily originating from context management and instruction structure.

1. **Implicitly Unclear Instruction Priorities**: When the system prompt contains multiple instructions without clear priority annotations, the model's judgment on "which steps are mandatory" versus "which are optional suggestions" relies on implicit linguistic cues (word severity, placement). This implicit judgment is unreliable, easily leading to critical steps being skipped as optional content.
2. **Primacy and Recency Bias**: Model attention is naturally stronger at the beginning and end of context than in the middle (the "Lost in the Middle" phenomenon). If instructions for critical steps are located in the middle of a long prompt, they are more easily "ignored" or under-weighted compared to system settings at the start and recent user messages at the end.
3. **Lack of Structured Progress Tracking**: Without an explicit step list (Checklist) and completion status tracking, the model must rely on "impressions" of dialogue history to judge which steps are completed and which remain. Such implicit tracking easily causes omissions or duplicate execution in long tasks.
4. **Solution—Structured Step Checklists**: Explicitly enumerate critical steps as a numbered Checklist, with each step having a clear completion mark field. The model must update corresponding marks after executing each step, clearly displaying "Completed [x]" and "Pending [ ]" checklist status during the next invocation round, substantially reducing omission rates.
5. **Mandatory Language Annotation**: Use mandatory phrasing ("must execute," "cannot be skipped") for critical steps that must be executed and position them in high-weight areas of the prompt (beginning or end), distinguishing them from advisory optional content ("may consider") to help the model correctly identify priorities.

</details>

**14. Why Do Agents Suffer from the "Drifting Off-Topic the More They Reason" Phenomenon in Multi-Turn Reasoning?**

<details>
<summary>💡 Show Answer</summary>

In multi-round reasoning, "getting increasingly off-topic the more one thinks" is the combined result of the autoregressive generation mechanism and the cumulative effect of long-range reasoning. Understanding its mechanism helps in designing more robust multi-step reasoning workflows.

1. **Error Accumulation**: In multi-round reasoning, the output of each round becomes the input context for the next round. If a slight deviation occurs in a certain round (imprecise wording, logical jumps, misunderstanding of premises), this deviation is treated as a "confirmed fact" and passed to the next round. The next round continues reasoning on this basis, further amplifying the deviation, similar to "noise accumulation" in signal processing.
2. **Self-Reinforcing Erroneous Assumptions**: When the model forms a (possibly wrong) assumption in early reasoning steps, subsequent reasoning steps tend to continue thinking within the framework of this assumption rather than questioning whether the assumption itself holds. This "path dependency" makes it difficult for erroneous assumptions to self-correct; instead, they are continuously reinforced as reasoning deepens.
3. **Double-Edged Sword Effect of Reflection Mechanisms**: Self-reflection is intended to help correct errors, but if reflection lacks external objective verification grounds (relying solely on the model's own judgment of "where problems might lie"), the reflection process may merely "go in circles" within its own cognitive framework—questioning correct parts that shouldn't be questioned while ignoring the actual issue, drifting further from the original correct direction the more it reflects.
4. **Attention Dilution within Context Window**: As the reasoning rounds increase and the context continues to accumulate, the relative weight of early correct information (including the original task goal) decreases within the long context, making the model more susceptible to being dominated by content from recent rounds that may have already drifted.
5. **Mitigation Strategies**: Insert a "back-to-basics" check every few reasoning rounds (re-aligning with original task goals and established facts, rather than merely examining the previous round's reasoning); introduce external verification signals (tool execution results, data queries) to interrupt purely introspective continuous reasoning and anchor the reasoning direction with objective facts; limit the rounds of continuous self-reflection, switching to requesting external input or pausing for human intervention once a threshold is exceeded.

</details>

**15. Why Does an Agent Gradually "Forget" the Original Goal During Long-Task Execution?**

<details>
<summary>💡 Show Answer</summary>

An Agent gradually "forgetting" its original goal in long tasks is a phenomenon resulting from the superposition of inherent limitations of context management mechanisms and long-range task complexity. Understanding its mechanisms helps in designing effective preventive measures.

1. **Natural Decay of Context Position**: The original goal is usually established at the beginning of the task. As the task progresses, a large amount of intermediate process information (tool execution results, intermediate reasoning, state updates) accumulates in the context. Relative to the growing total context, the "position" of the original goal information shifts further toward the beginning and its proportion shrinks, causing the model's attention weight on this information to naturally decline (a manifestation of the "Lost in the Middle" phenomenon).
2. **Risk of Goal Loss in Context Compression**: When context length approaches the window limit and requires compression, if the compression algorithm (whether rule-based or LLM summarization) does not explicitly identify and protect core information like the "original goal", the goal description may be simplified, blurred, or even accidentally lost during compression, as compression often prioritizes retaining the "most recent" information over the "initial" information.
3. **Attention Crowding Out by Intermediate Results**: A large volume of intermediate results generated in long tasks (data returned by tools, milestone conclusions) continuously occupies context space and processing attention. Comparatively, the macro-level yet less frequently mentioned information of "why this is all being done" (the original goal) is easily overwhelmed by these dense intermediate details.
4. **Preventive Mechanism—Explicit Goal Anchoring**: The most effective solution is not to rely on the model "remembering" the original goal from lengthy history, but rather to explicitly re-inject the original goal into the current prompt before every key decision (even if it involves some token redundancy). This ensures that regardless of how the context evolves, goal information is consistently presented to the model with high weight. This "active anchoring" is far more reliable than "passive reliance on memory".
5. **Periodic Goal Alignment Checks**: Insert explicit "goal alignment check" steps at critical milestones of long tasks, requiring the model to explicitly state "whether current execution still serves the original goal, and if there is a deviation, where it lies". This proactive self-examination mechanism enables deviations to be discovered and corrected at an early stage, rather than allowing deviations to continuously accumulate undetected until task completion.




![Illustration of Why Does an Agent Gradually "Forget" the Original Goal During Long-Task Execution?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig55_agent_gradually_forget_original.png)

</details>

**16. What is the Most Difficult Stage for Troubleshooting in Agent Systems?**

<details>
<summary>💡 Show Answer</summary>

The invisibility of the "decision chain" in Agent systems is the single biggest difficulty in troubleshooting. This is fundamentally different from traditional software system debugging: traditional system execution logic follows deterministic code paths, whereas an Agent's decision-making process relies on the model's internal reasoning, which is naturally far less traceable than code logic.

1. **Implicit Nature of Decision Rationale**: When the model chooses to call a tool or take a specific branch, the underlying "why" exists in the model's internal activation states. Unless explicitly requested to output the reasoning process (such as the Thought in the ReAct framework), external observers can only see "what was done" but not "why it was done," presenting a fundamental difficulty for troubleshooting.
2. **Tracing Multi-Step Decision Chains**: When an error occurs after a task undergoes multiple steps, locating "exactly which decision at which step caused the problem" requires backtracking through the entire execution chain and incrementally inspecting the inputs, reasoning processes, and outputs at each decision point. This process is extremely time-consuming without a comprehensive logging system.
3. **Solution—Structured Trace System**: Require the model to output structured reasoning rationales at each key decision point (not just the final decision, but also "why this option was selected over others"), and persist the complete context of each step (inputs, intermediate reasoning, outputs, confidence scores) into a searchable tracing system to build a complete "decision audit trail".
4. **Visualized Decision Trees**: Visualize the multi-step task execution process as a decision tree (where each node represents a decision point containing key information of that decision), enabling engineers to quickly browse the entire execution path and pinpoint the exact location of abnormal decisions without reading lengthy raw logs word by word.
5. **Comparative Analysis**: When a specific decision point is suspected of having issues, similar inputs can be constructed for controlled testing to observe whether decisions remain consistent. Combining comparative analyses across multiple similar cases makes it easier to identify systemic problems in decision logic rather than analyzing single failure cases in isolation.

</details>

**17. Why Do Agents Consistently Fail on Edge Cases?**

<details>
<summary>💡 Show Answer</summary>

Failing on edge cases is the hardest hurdle to overcome in transitioning Agent systems from "demo-capable" to "production-reliable." The root cause lies in the inherent characteristic of edge cases: rare yet complex.

1. **Coverage Blind Spots in Training Data and Examples**: Models are mostly exposed to "typical scenarios" during training and in few-shot examples. Edge cases (empty inputs, extreme values, special characters, concurrency conflicts, rare user intent expressions) occur infrequently, leaving the model lacking explicit "learning bases" in these scenarios and making it more prone to unpredictable behavior.
2. **Combinatorial Explosion Effect**: Edge cases are often caused not by a single factor, but by a combination of multiple marginal conditions (e.g., "empty result + network timeout + ambiguous user input" occurring simultaneously). With each additional marginal factor, the combinatorial space grows exponentially, making exhaustive testing coverage of all combinations engineering-unrealistic.
3. **Rules and Examples Struggle to Generalize to Unseen Combinations**: Even if special handling rules are written for known edge cases, new and unforeseen edge combinations will continually emerge. Rule maintenance costs continually escalate as scenarios expand, leaving an endless supply of "yet-to-be-encountered" edge cases.
4. **Mitigation Strategies—Defensive Design Over Exhaustive Coverage**: Rather than attempting to exhaustively cover all edge cases, a more effective strategy is to design generic defensive mechanisms: perform sanity checks on all external inputs (rejecting blatantly abnormal inputs instead of trying to process them); perform schema verification on all tool call results (triggering standardized error-handling flows for abnormal results rather than letting the model "improvise"); and design unified fallback responses of "Uncertain, requesting clarification" for ambiguous cases, rather than forcing out potentially erroneous outputs.
5. **Continuous Collection and Regression Testing**: Systematically collect edge-case failure instances from production environments and convert them into regression test cases, progressively expanding test coverage. Concurrently, analyze common characteristics of edge cases to design generalized solutions targeting entire classes of issues (rather than isolated instances), thereby enhancing generalization capability.

</details>

**18. Why is Transitioning Agent Systems from Prototype to Production Particularly Difficult?**

<details>
<summary>💡 Show Answer</summary>

The "perilous leap" of Agent systems from prototype to production is essentially bridging the infrastructure chasm required to transition from "proof of concept" to a "production-grade engineering system." This chasm is often far larger than expected.

1. **Implicit Assumptions in Prototype Stage**: Prototype development typically focuses solely on whether core functionalities can run, implicitly assuming idealized conditions such as "network is always available," "tools always succeed," "user inputs are always reasonable," and "no concurrency." None of these assumptions hold in production environments, but the prototype stage fails to expose these deficiencies.
2. **Building Observability from Scratch**: During prototyping, developers can debug simply by inspecting console output and local logs. However, production environments require establishing a complete observability system (metrics collection, distributed tracing, alerting systems) to proactively detect anomalies before users report issues. This system often demands dedicated development investment, whose importance and workload are easily underestimated.
3. **Catching Up on Security and Permissions**: For rapid validation, prototypes often lack strict permission controls and input validations. Transitioning to production requires remedying these gaps—tool execution permission management, Prompt Injection defense, and operation audit logs—which are completely missing in prototypes yet indispensable security infrastructure for production.
4. **Systematization of Fault Tolerance Mechanisms**: In prototype stages, encountering an error can simply throw an exception and terminate. Production environments require designing explicit handling strategies for every potential failure (retry, fallback, circuit breaking, human intervention). The design and testing workload for this fault-tolerance system often exceeds that of the core business logic itself.
5. **Engineering for Cost and Performance**: Prototypes rarely pay attention to token consumption and latency. Once exposed to real-world production traffic, cost and performance bottlenecks rapidly manifest, necessitating optimization techniques such as caching, batching, and model routing. These optimizations often require substantial modifications to the original architecture.
6. **Mitigation Strategies**: Reserve extension points for observability and fault tolerance during the prototype design phase (even if full features are not implemented initially) to avoid starting from scratch during production rollout; treat the engineering infrastructure required for production as independent iteration milestones rather than glossing over them as "add-on features."




![Illustration of Why is Transitioning Agent Systems from Prototype to Production Particularly Difficult?](https://img.pricenode.uk/llm_interview_questions/english/chapter6/fig56_transitioning_agent_systems_from.png)

</details>

**19. What Do You Think is the Most Core Unsolved Problem in Current Agent Systems?**

<details>
<summary>💡 Show Answer</summary>

(Open question; below are representative sample answers) \begin{enumerate}
- **Long-Task Reliability**: In long tasks exceeding 50 steps, error propagation and state drift cause the success rate of Agents to drop sharply;
- **Error Recovery Capability**: Current autonomous recovery capabilities of Agents when encountering unexpected situations remain weak, making them prone to getting stuck or causing cascading errors;
- **Indirect Prompt Injection Defense**: Systematic solutions for injection defense when Agents read external content are still lacking;
- **Evaluation Difficulty**: Lack of universally recognized end-to-end Agent evaluation benchmarks makes objective comparison of different system capabilities difficult;
- **Cost Control**: Multi-step tasks consume a high volume of tokens, posing heavy cost pressures on commercial deployment. Selecting 1--2 points to elaborate in-depth and presenting your own improvement ideas during an interview is more persuasive.

</details>

**20. How to Migrate an Existing Manual Workflow to an Agent Automated Workflow?**

<details>
<summary>💡 Show Answer</summary>

Migration Steps: \begin{enumerate}
- **Workflow Mapping**: Document every step, input, output, and decision point of the existing manual process;
- **Automation Feasibility Analysis**: Assess the automation difficulty of each step (simple rule-based vs. judgment-required vs. strictly manual);
- **Tool Identification**: Identify or develop corresponding tools for each automatable step;
- **Progressive Migration**: First automate low-risk, highly repetitive steps while retaining Human-in-the-Loop (HITL) nodes for manual steps;
- **Parallel Execution**: Run the Agent and manual workflow in parallel for a period, comparing results to validate quality;
- **Retention of Manual Nodes**: Keep manual intervention for judgment-based decisions, high-risk operations, compliance audits, etc.;
- **Gradual Expansion of Automation**: After stability is verified, progressively replace select manual nodes with automation while monitoring quality;
- **Full Migration**: Once a comprehensive monitoring system is established, consider full automation while maintaining emergency manual intervention channels.

</details>

**21. Have You Fine-Tuned Agent Capabilities? How Did You Collect the Dataset?**

<details>
<summary>💡 Show Answer</summary>

The core of fine-tuning Agent capabilities is teaching the model how to better "think" and "use tools," which is essentially a form of Behavioral Cloning—training data consists not of simple (input, output) pairs, but of high-quality "decision-making trajectories."

Primary Methods of Data Collection:
1. **Teacher Model Synthetic Data Generation**: Use powerful closed-source models (such as GPT-4o) under the ReAct framework to execute tasks, record full "Thought $\rightarrow$ Action" trajectories, and filter out low-quality trajectories.
2. **Manual Composition or Correction of Trajectories**: Domain experts manually construct or correct high-quality reasoning trajectories.
3. **Collection from Real User Interactions**: Accumulate successful execution cases from production environments.

</details>

**22. Which Agent Frameworks Have You Used? How Did You Make the Technology Selection? What Were the Evaluation Metrics for Your Final Scenario?**

<details>
<summary>💡 Show Answer</summary>

Technology selection is typically based on core project requirements, evaluated from two perspectives: "logic-orchestration-driven" or "data-driven." When the project is centered around Agents requiring complex logic orchestration and multi-tool collaboration, LangChain is preferred; when the project is centered around data requiring strong knowledge base construction and Q&A capabilities, LlamaIndex is preferred.

Evaluation metrics are comprehensively assessed across three dimensions: Task Success Rate (completion ratio and accuracy of final tasks); Process Efficiency (cost / latency / number of steps); Robustness and Predictability (error-handling capability, consistency, security evaluation).

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

</details>

