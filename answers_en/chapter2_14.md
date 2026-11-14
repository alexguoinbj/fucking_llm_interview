# 2.14 Advanced Prompt Engineering Techniques — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 22 questions

**1. What is Prompt Decomposition? How is it different from Prompt Chaining?**

<details>
<summary>💡 Show Answer</summary>

**Prompt Decomposition**: Splitting a complex prompt into parts with clearer structure at the design stage (role setting, context, instructions, format requirements), where each part has a single responsibility, and the whole is still a single LLM call. **Prompt Chaining**: Breaking a complex task into multiple sequential LLM calls, where each call is an independent step, and the output of the previous step is passed to the next. Differences: \begin{enumerate}
- Decomposition is a structural optimization of a prompt, still a single call; Chaining is the decomposition of the execution flow, multiple calls; \item Decomposition solves the problem of "a prompt being too complex"; Chaining solves the problem of "cannot be completed in a single call"; \item The two can be combined: the prompt inside each chain node can be a structured prompt designed through decomposition.

</details>

**2. What is "Counterfactual Prompting"? What are its application scenarios?**

<details>
<summary>💡 Show Answer</summary>

Counterfactual Prompting: Requires the model to analyze "how the result would change if a certain condition were different," activating deep reasoning through hypothetical scenarios. Application scenarios:
1. **Causal Analysis**: "What would have happened if this company didn't pivot in 2020";
2. **Plan Evaluation**: "If Plan A is not adopted, what are alternative paths";
3. **Risk Prediction**: "If this assumption is wrong, what are the consequences";
4. **Code Debugging**: "If this line of code is removed, how would program behavior change";
5. **Historical Analysis**: "If this technology appeared 10 years earlier, what impact would it have on the industry";
6. **Decision Support**: "Under [different conditions], how would the optimal decision change".
Advantage: Helps the model step out of superficial answers and explore deep logical relationships.

</details>

**3. What is Emotional Prompting? Does it affect model output?**

<details>
<summary>💡 Show Answer</summary>

Emotional Prompting involves adding emotional language in the prompt, such as "this is very important to me," "please do everything you can to help," "this is an urgent task." Research finds (Li et al. 2023): Emotional prompting does improve LLM output quality in some tasks (about 5–10%), likely because such phrases correlate highly with high-quality, earnest answers in training data. Practical applications:
1. **Positive Emotion**: "This analysis is critical for my thesis, please think carefully"—may improve answer quality;
2. **Deadline Pressure**: "I need this report by tomorrow"—may improve response completeness;
3. Note: Effects are unstable, vary greatly across models; should not be a primary optimization method, just a lightweight auxiliary trick;
4. Works more stably when combined with role prompting.

</details>

**4. How to Design Prompts to Guide the Model in Self-Critique?**

<details>
<summary>💡 Show Answer</summary>

1. **Two-step method**: Step one generates a first draft, step two lets the model review its own output (in the same prompt or chain);
2. **Critic role**: "Review the above answer, point out any problems, omissions, or areas for improvement";
3. **Critique against standards**: "Check your answer against the following standards: [Standard 1], [Standard 2]...";
4. **Revision iteration**: "Based on your critique, generate an improved version";
5. **Three-step Refine**: First draft -> Critique -> Improved version, can be iterated multiple times;
6. **Specific critique dimensions**: "Check: Are there hallucinations? Is the logic coherent? Is any important information missing?";
7. Application scenarios: Code review, article proofreading, plan evaluation, and other tasks with high quality requirements.

</details>

**5. What is Generated Knowledge Prompting? How does it improve the quality of factual answers?**

<details>
<summary>💡 Show Answer</summary>

Generated Knowledge Prompting (Liu et al. 2021): Before answering a question, first ask the LLM to generate background knowledge related to the question, and then answer the question based on this self-generated knowledge. Two-step implementation:
1. Step 1: "Regarding [Topic], list the relevant knowledge you know";
2. Step 2: "Based on the above knowledge, answer: [Original question]".
Reasons for quality improvement:
1. The first step forces the model to "activate" relevant knowledge into the context, reducing omissions;
2. The generated knowledge acts as "working memory," making the answer more grounded;
3. The knowledge generation step can be inspected, and if errors are found, they can be corrected in the second step.
Suitable scenarios: Complex Q&A requiring the integration of multiple knowledge points; knowledge-intensive tasks. Limitations: If the knowledge generated in the first step is incorrect, it will contaminate the answer in the second step.

</details>

**6. What is Directional Stimulus Prompting? How does it finely control the output direction?**

<details>
<summary>💡 Show Answer</summary>

Directional Stimulus Prompting (DSP, Li et al. 2023): Adds "hints" or "directional signals" in the prompt to guide the model's generation direction, rather than directly giving the answer or dictating the format. How it works: Hints are directional instructions more abstract than answers, such as keywords, direction descriptions, and theme words. Example: For the task "Write a short essay on climate change," add the hint "Please cover in the essay: scientific evidence, economic impacts, and solutions." Application scenarios:
1. **Summary Generation**: Use keyword hints to guide summary content;
2. **Creative Writing**: Use theme words and emotional directions to guide writing;
3. **Dialogue Systems**: Guide the conversation to develop in a specific direction;
4. Combination with ICL: Hints can be dynamically retrieved from a candidate pool.

</details>

**7. What is Maieutic Prompting (Socratic Prompting)? What scenarios is it suitable for?**

<details>
<summary>💡 Show Answer</summary>

Maieutic Prompting (Jung et al. 2022): By continuously asking "why," it asks the model to break down its own reasoning layer by layer until a fundamental truth is found or a contradiction in reasoning is discovered. Implementation method:
1. Let the model give an initial answer;
2. For each key argument, ask "Why do you think this is correct?";
3. Continue to ask until the reasoning becomes self-consistent or a contradiction is found;
4. Verify or correct the initial answer based on the breakdown results.
Suitable scenarios:
1. **Complex Reasoning Verification**: Test whether the reasoning chain truly holds;
2. **Hypothesis Testing**: Discover inferences that "sound reasonable but are actually wrong";
3. **Educational Scenarios**: Guide students to think deeply about problems;
4. **Decision Analysis**: Break down the assumptions behind decisions to identify weak links.

</details>

**8. How to Achieve "Multi-Perspective Analysis" through Prompts?**

<details>
<summary>💡 Show Answer</summary>

1. **Perspective enumeration**: "Analyze this issue from the following three perspectives: [User Perspective], [Developer Perspective], [Business Perspective]";
2. **Stakeholder analysis**: "List all relevant parties, and analyze the impact of this decision on each party";
3. **Pro and con debate**: "First give the strongest arguments supporting this view, then give the strongest arguments against it, and finally synthesize an evaluation";
4. **Time dimension**: "Analyze the short-term (within 1 year), medium-term (3 years), and long-term (5+ years) effects of this plan";
5. **SWOT framework**: "Analyze from the four perspectives of Strengths, Weaknesses, Opportunities, and Threats";
6. **Expert role rotation**: "Provide opinions from the perspectives of a data scientist, business manager, and compliance officer, respectively";
7. Formatted output: Assign a structure to each perspective to facilitate comparison.

</details>

**9. What is Skeleton-of-Thought Prompting? How does it improve generation speed?**

<details>
<summary>💡 Show Answer</summary>

Skeleton-of-Thought (Ning et al. 2023): Divides the generation process into two steps:
1. First, ask the LLM to quickly generate the "skeleton" (outline/list of key points) of the answer;
2. Then expand on the detailed content of each point in parallel.
Principle of speed improvement: Traditional auto-regressive generation is serial (token by token). The Skeleton method parallelizes the expansion step—the expansion of multiple points can happen simultaneously (via parallel API calls). Speed improvement: For long text generation, the speed can theoretically increase proportionally to the number of key points (N points -> N parallel calls). Cost: Requires multiple API calls; increased complexity in coordinating parallel calls; final integration requires an additional step. Suitable scenarios: Fast generation of long articles, detailed reports, and multi-chapter content.

</details>

**10. How to Design Prompts to Make the Model Distinguish between "Facts" and "Opinions"?**

<details>
<summary>💡 Show Answer</summary>

1. **Explicit classification requirements**: "In your answer, tag statements supported by evidence with [Fact], and tag subjective judgments or inferences with [Opinion]";
2. **Formatted output**: `{"facts": ["..."], "opinions": ["..."]}`;
3. **Source requirements**: "For content tagged as facts, state its basis (statistical data/research conclusions/general consensus)";
4. **Example guidance**: Provide an example that correctly distinguishes facts and opinions;
5. **Opposing views**: "If this is a controversial topic, list the views of different positions, do not just present one view";
6. **Certainty language**: "Use 'research shows' to express facts, use 'I think/perhaps/it is estimated' to express opinions";
7. Application scenarios: News analysis, research reports, legal documents, and other fields requiring strict distinction between facts and opinions.

</details>

**11. What is a Dual Prompt? What are its application scenarios?**

<details>
<summary>💡 Show Answer</summary>

Dual Prompt uses two opposing prompts to generate pros and cons separately, and then synthesizes the two to generate a balanced final output. Implementation method:
1. Positive prompt: Generates arguments supporting a position;
2. Negative prompt: Generates arguments opposing the position;
3. Synthesis prompt: Generates a comprehensive analysis based on both sides' content.
Application scenarios:
1. **Debate Preparation**: Generate arguments for and against a debate topic;
2. **Product Evaluation**: Generate advantages and disadvantages respectively;
3. **Decision Analysis**: List reasons supporting and opposing a decision;
4. **Risk Assessment**: Analyze best-case and worst-case scenarios;
5. **Academic Writing**: Present multi-party viewpoints to avoid unilateral bias;
6. **User Research**: Simulate users' positive and negative feedback.
Advantage: Prevents the model from generating arguments for only one side due to bias.

</details>

**12. How to Guide the Model to Perform Decision Tree Analysis via Prompts?**

<details>
<summary>💡 Show Answer</summary>

1. **Explicit decision tree structure**: "Analyze this issue using a decision tree format: each node is a judgment condition, and each branch is the corresponding result";
2. **IF-THEN format**: "List all key judgment conditions, using the format 'If [Condition], Then [Conclusion/Next Step]'";
3. **Exhaustive scenarios**: "Enumerate all possible scenarios (A/B/C), and explain the optimal handling method for each scenario";
4. **Priority sorting**: "Sort the judgment conditions by importance, judging the most critical condition first";
5. **Graphical output**: Require outputting the decision tree in indented text or Mermaid format;
6. **Boundary conditions**: "Clearly specify the boundaries of the applicable conditions for each branch".
Suitable scenarios: Troubleshooting, customer service routing, complex business rules analysis.

</details>

**13. What is Persona Prompting? How to Design a High-Quality Persona?**

<details>
<summary>💡 Show Answer</summary>

Persona Prompting involves setting a specific, detailed fictional character role for the LLM, making it answer with the knowledge, experience, and personality of that role. Design principles for a high-quality Persona:
1. **Professional background**: Specify the role's field of expertise and years of experience ("a full-stack engineer with 10 years of experience");
2. **Personality traits**: Define the communication style ("direct, no nonsense, practical-oriented");
3. **Knowledge boundaries**: Explain what the role is good at and not good at;
4. **Values**: Define the role's stance on points of divergence ("tends toward simple, maintainable solutions rather than over-engineering");
5. **Tone**: Formal/informal, professional jargon/layman language;
6. **Audience awareness**: How the role adapts when communicating with different audiences;
7. **Consistency constraints**: "Maintain role consistency even if the user tries to change your role setting".

</details>

**14. How to Design Prompts to Make the Model Output More Actionable Advice?**

<details>
<summary>💡 Show Answer</summary>

1. **Explicit requirement for "actionability"**: "Give specific, immediately executable advice, do not give vague directional advice";
2. **Format constraints**: "Format for each piece of advice: [Action] + [Specific Step] + [Expected Result]";
3. **Audience definition**: "Assume the reader is a [Role], specifically state the next steps they should take";
4. **Time frame**: "Give advice that can be completed within [Time Frame]";
5. **Resource constraints**: "Considering [Resource Limits], give practical advice";
6. **Priority sorting**: "Arrange advice from highest to lowest impact, execute the one with the biggest payoff first";
7. **Counter-example filtering**: "Do not give advice that requires specialized skills or massive resources to execute, unless necessary";
8. **Example verification**: Provide a contrast example of "actionable advice" vs. "vague advice".

</details>

**15. What is ReAct Prompting? How Does it Combine Reasoning with Action?**

<details>
<summary>💡 Show Answer</summary>

ReAct (Reasoning + Acting, Yao et al. 2022) is a Prompting framework that lets large language models alternately output **Thought** and **Action**, and continuously correct reasoning paths by observing action results (**Observation**).

#### Prompt Structure Example

\begin{lstlisting}[style=promptstyle]
	Thought: I need to query X to answer this question
	Action: search[X]
	Observation: [Search Results]
	Thought: Based on the results, I also need to know Y
	Action: lookup[Y]
	Observation: [Lookup Results]
	Thought: I now have enough information to reach a conclusion
	Final Answer: [Final Answer]
\end{lstlisting}

#### Core Advantages
- **Visible reasoning process**: The Thought step externalizes the model's internal reasoning, making it easier for developers to debug and review.
- **Verifiable intermediate steps**: Every Action and Observation is explicitly recorded, allowing step-by-step verification of the logic chain's correctness.
- **Think before acting**: Thought prompts the model to pre-judge before taking action, reducing blind tool calls.
- **Adjustments based on real feedback**: Observation introduces real information from the external world, allowing reasoning to be dynamically corrected rather than relying on the model's static knowledge.

#### Essential Difference from Pure Chain-of-Thought (CoT)

Pure CoT only unfolds the reasoning chain within the model, without any interaction with the external environment; whereas ReAct introduces real external tool calls (such as search engines, databases, APIs) and observation feedback from the environment, enabling the model to obtain real-time, accurate external information to assist reasoning. ReAct thus becomes the foundational Prompting framework for building **LLM Agents**—it defines the basic cycle of Agent perception, thought, and action.

</details>

**16. How Does Prompt Design Support Multi-Agent Systems?**

<details>
<summary>💡 Show Answer</summary>

In multi-agent systems, Prompt design needs to solve the two core issues of **role division** and **collaborative communication**.

#### Define Clear Responsibility Boundaries for Each Agent

Each Agent's System Prompt should precisely describe its role positioning, capability scope, and prohibited out-of-bounds behaviors to avoid overlap and conflict among multiple Agents in the same responsibility domain. For example:

\begin{lstlisting}[style=promptstyle]
	# Role: Data Analysis Agent
	You are only responsible for performing statistical analysis on structured data and outputting analysis reports.
	You are not responsible for data collection, visual rendering, or final decision-making.
	When visualization is needed, transfer the task to the Visualization Agent.
\end{lstlisting}

#### Standardize Inter-Agent Communication Formats

The Prompts for passing messages between Agents should specify a uniform structure (such as JSON Schema or a fixed Markdown template) to ensure the receiving end can reliably parse them. For example:

\begin{lstlisting}[style=promptstyle]
	{
		"from": "ResearchAgent",
		"to": "SummaryAgent",
		"task": "summarize",
		"payload": {
			"content": "...",
			"max_length": 200,
			"language": "en"
		}
	}
\end{lstlisting}

#### Key Design Principles
- **Single Responsibility**: Each Agent's Prompt focuses on one type of task, reducing ambiguity.
- **Explicit Handover Protocols**: Define in the Prompt when and how to transfer tasks to other Agents.
- **Error Handling Conventions**: Reserve error codes and exception description fields in the message format, allowing Agents to degrade gracefully.

</details>

**17. How Does Prompt Design Support Workflow Systems?**

<details>
<summary>💡 Show Answer</summary>

Workflow systems break down complex tasks into ordered sequences of nodes (or Directed Acyclic Graphs), and the Prompt design for each node must strictly align with the state machine structure of the overall process.

#### Node-Level Prompt Design Specifications

The Prompt for each Workflow node should explicitly declare three elements:

\begin{lstlisting}[style=promptstyle]
	## Node: Information Extraction Node

	### Input Sources
	- Upstream Node: Document Preprocessing Node
	- Input Variable: {{raw_text}} (cleaned plain text)

	### Processing Logic
	Extract the following fields from the input text:
	- Entity names (names of people, places, organizations)
	- Time information
	- Core events

	### Output Format
	Strictly output JSON, structured as follows:
	{"entities": [...], "time": "...", "event": "..."}
	Do not output any additional text or Markdown tags.
\end{lstlisting}
#### Design Points Aligning with Workflow State Machines
- **Explicit Input Declaration**: Clearly mark which information comes from upstream nodes using variable placeholders (e.g., `\{\{variable\}\}`) in the Prompt, avoiding implicit dependencies.
- **Strict Output Format Constraints**: The node output must be in a format that downstream nodes can directly consume, reducing the extra processing of parsing layers.
- **Conditional Branch Support**: For nodes requiring conditional jumps, define branch identifier outputs in the Prompt for routing decisions by the Workflow engine.
- **Idempotency Design**: Node Prompts should be designed to produce stable outputs for the same inputs, facilitating Workflow retries and breakpoint resumption.

</details>

**18. How Should an Enterprise-Level Prompt System Be Built?**

<details>
<summary>💡 Show Answer</summary>

An enterprise-level Prompt system needs to be built systematically from four dimensions: **standardization, version management, reusability**, and **quality assurance**.

#### Standardized Prompt Template Library

Categorize and archive Prompts by business scenarios, establishing a unified template directory structure:

\begin{lstlisting}[style=promptstyle]
	prompt-library/
	customer-service/     # Customer service scenario
	intent-detection.txt
	reply-generation.txt
	content-creation/     # Content creation scenario
	blog-writing.txt
	summary.txt
	data-analysis/        # Data analysis scenario
	sql-generation.txt
	report-writing.txt
\end{lstlisting}

#### Version Management Mechanism

Referencing code version management ideas, host Prompts via Git, recording for each change: change content, change reason, and effect comparison data. The Prompt header should contain metadata:

\begin{lstlisting}[style=promptstyle]
	---
	prompt_id: customer-service/intent-detection
	version: 2.3.1
	author: prompt-engineering-team
	last_updated: 2025-06-01
	model_target: claude-sonnet-4-6
	---
\end{lstlisting}

#### Unified Variable Injection Specification

Formulate enterprise-wide unified variable naming and injection specifications to avoid different teams using incompatible placeholder formats:

\begin{lstlisting}[style=promptstyle]
	# Specification: Use double curly braces, snake_case naming
	User Name: {{user_name}}
	Chat History: {{chat_history}}
	Current Time: {{current_datetime}}
\end{lstlisting}

#### Evaluation and Monitoring System
- **Automated Evaluation Sets**: Establish golden test case sets for each type of Prompt, automatically regressing after each version update.
- **Online Effect Monitoring**: Track key metrics (task completion rate, user satisfaction, output compliance rate) and set anomaly alerts.
- **A/B Testing Mechanism**: Support parallel canary releases for multi-version Prompts, driving version promotion decisions based on data.

</details>

**19. What is the Real Bottleneck of Prompt Optimization?**

<details>
<summary>💡 Show Answer</summary>

The bottleneck of Prompt optimization **is not the wording skills themselves**, but is concentrated in the following three deeper dimensions:

#### 1. Depth of Understanding the Task Essence

Many Prompts perform poorly because the designer lacks a thorough understanding of the task to be solved—unclear about the boundary conditions, failure modes, and success criteria of the task. The linguistic expression of the Prompt is the last step; the prerequisite is a clear modeling of the task.

#### 2. Organization Quality of Context Information

The model's output quality is highly dependent on the completeness and structured degree of the input information. Missing key information, chaotic information order, and mixing relevant with irrelevant information in the context will severely impact output quality; this is a more fundamental problem than "changing a word."

#### 3. Maturity of the Evaluation System

**Prompt tuning without evaluation is blind.** If there is a lack of quantitative evaluation metrics and standard test sets, developers cannot distinguish between "this change really works" and "this output just happens to be better." The real bottlenecks lie in:
- Whether a test set covering typical scenarios and edge cases has been established;
- Whether there is an objective automated scoring mechanism (rather than relying on manual feeling);
- Whether the impact of each modification on the overall effect distribution can be tracked, rather than just looking at a few samples.

</details>

**20. Should We Choose Prompt Tuning or Model Fine-Tuning to Solve Problems?**

<details>
<summary>💡 Show Answer</summary>

#### Decision Principle: Prompt First, Fine-Tune as Needed

- **First Choice Prompt Tuning**: Extremely low cost (no GPU resources needed), extremely fast iteration (minutes), suitable for initial exploration and continuous optimization of the vast majority of tasks.
- **Conditions for Considering Fine-Tuning**:
    - The task involves a fundamental lack of the model's mastery of **specific domain knowledge** (e.g., highly specialized legal terminology systems, rare language dialects);
    - The task has extremely strict requirements on **output formatting** that are difficult to stably constrain by Prompts (e.g., complex nested JSON Schema);
    - Prompt tuning has been repeatedly tried and reached a **clear bottleneck**, with extremely limited room for effect improvement;
    - There is **enough high-quality annotated data** (hundreds to thousands) to support fine-tuning.

#### Practical Advice

The two schemes are not mutually exclusive—a fine-tuned model still needs well-designed Prompts to achieve optimal results. The correct path is: first use Prompt tuning to clarify task boundaries and failure modes, accumulate high-quality samples, then drive fine-tuning with these samples, and finally continue Prompt optimization on the fine-tuned model.

</details>

**21. Should We Choose Prompt or RAG to Solve Knowledge-Related Problems?**

<details>
<summary>💡 Show Answer</summary>

#### Selection Basis: Scale and Dynamism of Knowledge

- **Directly placing into Prompt (In-Context Knowledge) Applicable Scenarios**:
    - The required knowledge scale is small and can fit within the context window;
    - The knowledge is relatively stable and does not require frequent updates;
    - Requires the model to holistically understand and comprehensively apply this knowledge during reasoning.
- **RAG (Retrieval-Augmented Generation) Applicable Scenarios**:
    - The knowledge base volume is massive (tens of thousands to millions of documents) and cannot be fully placed in context;
    - The knowledge updates frequently, requiring real-time reflection of the latest content (e.g., product docs, regulations);
    - Only needs on-demand retrieval of fragments highly relevant to the current question, rather than full knowledge.

#### Combining the Two

In practice, the two schemes often synergize: RAG is responsible for retrieving the most relevant document fragments from a large-scale knowledge base, and then injecting these fragments into a carefully designed Prompt, allowing the model to comprehensively reason and generate the final answer. The quality of the Prompt determines how the model utilizes the retrieval results, while the retrieval quality of RAG determines what information the model can obtain; both jointly determine the final effect.

</details>

**22. Should We Choose Prompt or Memory to Manage User-Related Information?**

<details>
<summary>💡 Show Answer</summary>

#### Core Distinction Dimension: Timeliness and Persistence

- **Directly placing into Prompt is suitable for**:
    - **Temporary preferences** within the current conversation (e.g., "use a formal tone for this reply");
    - Contextual information explicitly stated by the user at the beginning of this session;
    - One-off instructions that do not need to be retained across sessions.
- **Memory Systems are suitable for**:
    - User information requiring **cross-session persistence** (e.g., professional background, language preferences, historical decision habits);
    - User profiles requiring **long-term accumulation and updating** (enriching continuously with each conversation);
    - Information volume that has exceeded the reasonable carrying capacity of a single Prompt.

#### Collaboration Mode

Memory systems and Prompts are not competing but upstream-downstream relationships: The Memory system handles persistent storage and intelligent retrieval, and at the start of each conversation, it **dynamically injects** the user memory fragments most relevant to the current task into the Prompt, achieving personalized experiences while controlling context length. The design key lies in: the retrieval strategy determines which memories are injected, and the Prompt template determines how these memories are presented; the two together determine the upper limit of the personalized effect.

</details>

