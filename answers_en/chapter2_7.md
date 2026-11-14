# 2.7 Rolling Summary and Structured State — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 2: Prompt & Context Engineering · 20 questions

**1. What are the limitations of LLM context windows? Why do long conversations encounter problems?**

<details>
<summary>💡 Show Answer</summary>

LLM context windows have a maximum token count limit (like GPT-4's 128k, Claude's 200k). In long conversations, historical messages continuously accumulate and eventually exceed the window limit. Once exceeded, it either reports an error or is forcibly truncated, causing the model to "forget" early information and affecting reply quality. Specific problems manifest at the following levels:

#### Hard Limits
- **Token Limit**: The total input + output tokens per API call cannot exceed the model context window size. For example, with GPT-4o (128k), a 100-turn customer service dialogue can easily exceed this limit;
- **Billing and Latency**: Even if not exceeded, an ultra-long context will cause inference costs to grow linearly (or even super-linearly), and Time to First Token (TTFT) will rise significantly, affecting user experience.

#### Quality Degradation Issues
- **Lost in the Middle**: Research indicates that LLMs' ability to extract information from the **middle** of the context is significantly weaker than from the beginning and end, causing early key information to be "diluted" even if the context is not exceeded;
- **Attention Drift**: As context grows, model attention weights tend to scatter, and response accuracy to the latest instructions declines;
- **Increased Hallucination Rate**: When there is a large amount of redundant or noisy text in the context, the model is more prone to producing hallucinated outputs contradicting the context.

#### Engineering Challenges
- **KV Cache VRAM Pressure**: Long contexts consume massive GPU VRAM to store KV Cache, limiting concurrent serving capabilities;
- **Truncation Strategy Dilemma**: Although forcibly truncating early content solves token overflow, it causes irreversible information loss, especially severely impacting scenarios relying on early agreements (like user preferences, task goals).

</details>

**2. What is dialogue history management? What are common strategies?**

<details>
<summary>💡 Show Answer</summary>

Dialogue history management refers to how to choose which historical content to pass to the model in multi-turn conversations. The core goal is to strike a balance between **information completeness**, **token efficiency**, and **response quality**. Common strategies include:

- **Full History**: Pass all historical turns entirely into the context. Suitable for short conversations or scenarios with abundant token budgets; information is lossless but costs grow linearly with turns, facing window overflow risks.

- **Sliding Window**: Only retain the latest $N$ turns of conversation (e.g., last 10 turns), directly discarding the excess. Simple to implement, controllable latency, but it hard-drops early information, performing poorly for scenarios requiring cross-turn references (e.g., a need mentioned by the user in turn 1 is brought up again in turn 20).

- **Rolling Summary**: Periodically compress early history into semantic summaries, replacing original records with summaries. It massively compresses token usage while retaining semantic continuity, making it a mainstream strategy for long conversation scenarios (details in the next section).

- **Structured State Management**: Does not retain original dialogue text, but stores key information (like user name, order number, preference settings) into structured state objects (JSON/Database) via NLP techniques like entity extraction and slot filling, dynamically rendering it as a prompt snippet in each request. Suitable for Task-Oriented Dialogue, with high information density, but requires designing a complex state Schema.

- **RAG-based History**: Store historical dialogues into a vector database, retrieving the most relevant historical snippets based on the current user input to inject into the context per turn, instead of full passing. Suitable for ultra-long conversations (hundreds of turns), balancing relevance and token efficiency, but introduces retrieval latency and recall accuracy issues.

- **Hybrid Strategy**: Combines multiple aforementioned methods, e.g., "latest $N$ turns full history + distant history rolling summary + key entity structured storage", which is a common architecture for production-grade dialogue systems.

The comparison of various strategies is shown in Table~\ref{tab:history_strategies}.

\begin{table}[h]

	\caption{Comparison of Dialogue History Management Strategies}



| **Strategy** | **Info Completeness** | **Token Efficiency** | **Impl Complexity** | **Applicable Scenarios** |
| --- | --- | --- | --- | --- |
| Full History | High | Low | Low | Short dialogues |
| Sliding Window | Low | High | Low | Dialogues without long dependencies |
| Rolling Summary | Medium | High | Medium | General long dialogues |
| Structured State | Medium | High | High | Task-oriented dialogues |
| RAG History Retrieval | High | High | High | Ultra-long dialogues |
| Hybrid Strategy | High | High | Very High | Production-grade systems |


\end{table}

</details>

**3. What is a Rolling Summary?**

<details>
<summary>💡 Show Answer</summary>

A Rolling Summary refers to periodically compressing early dialogue history into a text summary as the conversation proceeds, replacing original records with the summary to free up token space. During each request, the summary is concatenated at the beginning of the context, allowing the model to still perceive historical semantics.

#### Workflow

1. **Trigger Condition Checking**: When the current context token count exceeds a preset threshold (e.g., 70% of the window limit), trigger the summary process;
2. **Defining Summary Scope**: Select historical content "before the most recent $M$ turns" as the area to be compressed, preserving the original content of the latest $M$ turns to maintain short-term continuity;
3. **Calling LLM to Generate Summary**: Send the content to be compressed to an LLM, using a dedicated summary prompt (e.g., "Please compress the following dialogue into a summary of no more than 200 words, retaining all key information and user preferences") to generate the compressed summary;
4. **Replacement and Concatenation**: Replace original history with the generated summary, combining it with the recent $M$ turns of original records and the new user input to form the context for the next request:

    $$
    Context_{t} = [\underbrace{Summary_{1:t-M}}_{Compressed History} \| \underbrace{H_{t-M:t-1}}_{Recent Original History} \| \underbrace{Q_t}_{Current Input}]
    $$

5. **Iterative Updating**: As the dialogue continues to advance, newly generated content is continuously added to recent history. When the threshold is triggered again, the previous summary is combined with the newly added history and compressed again (i.e., "summary of a summary"), forming a rolling update mechanism.

#### Key Design Decisions

- **Summary Granularity**: Summaries too short lose details, while those too long offer limited token savings. The target summary length is usually set to 10%--20% of the original content;
- **Information Weighting**: Explicitly specify the types of information that need to be strongly retained in the summary prompt, such as "the user's personal information, explicit preference settings, and unfinished task goals";
- **Asynchronous Summarization**: In production systems, summary tasks can be triggered asynchronously immediately after user input to avoid summary generation latency blocking the main dialogue flow;
- **Summary Version Management**: Save each generated version of the summary, supporting backtracking to the original history for re-compression if summary quality is poor.

#### Limitations and Countermeasures

- **Lossy Information**: Compression is inherently lossy; details mentioned early by the user (like specific numbers, special agreements) may be omitted in the summary. The countermeasure is to use **Structured Patches**: Maintain a key entity dictionary alongside the summary to ensure high-priority information is not discarded;
- **Summary Drift**: Multiple rounds of "summary of summaries" may cause semantic deviation to gradually accumulate. **Anchor Summaries** (high-quality summaries generated based on complete original history) can be periodically inserted to correct drift;
- **Additional Latency and Cost**: Each summary requires an extra LLM call. The overhead can be reduced by controlling summary trigger frequency (e.g., triggering every 20 turns instead of every turn).

</details>

**4. What is the difference between rolling summary and direct truncation?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of What is the difference between rolling summary and](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig50_what_is_the_difference_between.png)



\begin{table}[h]



| **Method** | **Information Retention** | **Implementation Complexity** | **Applicable Scenarios** \\ \hline |
| --- | --- | --- | --- |
| Direct Truncation | Loses truncated part | Very Low | Scenarios where history is unimportant \\ \hline |
| Rolling Summary | Retains semantic compression | Medium | Dialogues requiring long-term memory \\ \hline |


\end{table}

Truncation makes the model completely forget the truncated content; rolling summary retains the semantic continuity of key information through compression.

</details>

**5. What is the basic workflow of rolling summary?**

<details>
<summary>💡 Show Answer</summary>

1. Maintain a `summary` (initially empty) and `history` (dialogue history list)
2. Check if the `history` token count exceeds the threshold after each dialogue turn
3. If exceeded, take the oldest N turns of history, call the LLM to generate a summary, and append it to `summary`
4. Delete the compressed turns from `history`
5. Construct during each request: `[summary] + [remaining history] + [new message]`

</details>

**6. What is Structured State? What problems does it solve?**

<details>
<summary>💡 Show Answer</summary>

Structured state records key variables in a dialogue using structured formats like JSON/YAML, rather than saving complete dialogue text. The problem it solves: In task-oriented dialogues, what truly holds value are often a few core fields (goal, confirmed items, pending decision items), rather than massive redundant chat text.

#### Core Idea
Traditional dialogue history management is essentially "saving raw text," whereas structured state management views the dialogue as a **State Machine**: each turn of dialogue is a state transition, and the model's task is to read the current state, incorporate the user input, output a response, and update the state. This is consistent with the software engineering concept of "separating data from view"—the structured state is the "data layer," while natural language dialogue is the "view layer."

#### Specific Problems It Solves

1. **Token Redundancy Problem**: Natural language dialogues contain a lot of modal particles, repeated confirmations, and chitchat. These contribute almost nothing to task completion but consume a large amount of tokens. Structured state only retains core fields after semantic extraction, resulting in an information density far higher than raw text.

    For example, the following three turns of dialogue:
    \begin{quote}
    User: "I want to book a flight from Beijing to Shanghai for tomorrow." \\
    Assistant: "Okay, what cabin class do you prefer?" \\
    User: "Economy is fine, the cheaper the better."
    \end{quote}
    Can be compressed into the following structured state, reducing token usage by approximately 80%:
    \begin{verbatim}
    {
    "task": "book_flight",
    "departure": "Beijing",
    "destination": "Shanghai",
    "date": "tomorrow",
    "cabin": "economy",
    "preference": "lowest price",
    "status": "pending flight confirmation"
    }
    \end{verbatim}

2. **Information Forgetting Problem**: In plain text history management, key information confirmed early on (e.g., user name, task goal) may be lost due to context truncation. Structured state solidifies this information into fields; as long as the state object exists, the information will not be lost.

3. **Multi-turn Consistency Problem**: In long dialogues, the model may "forget" previous agreements due to distracted attention, generating contradictory responses. Structured state explicitly injects the current state into the prompt in each request turn, forcing the model to generate responses based on the state, fundamentally eliminating such inconsistencies.

4. **Debuggability and Maintainability Problem**: Plain text history is opaque to developers, making it difficult to trace "why the model made a certain judgment." Structured state provides a clear intermediate representation, facilitating logging, troubleshooting, and business auditing.

#### Typical Schema Design for Structured State

The state object of a production-level task-oriented dialogue system typically contains the following layers:

\begin{verbatim}
	{
		"session_id": "uuid-xxxx",
		"task": {
			"type": "hotel_booking",
			"goal": "Book a hotel",
			"status": "in_progress"       // pending / in_progress / completed / failed
		},
		"slots": {
			"confirmed": {                 // Slots explicitly confirmed by the user
				"city": "Shanghai",
				"check_in": "2025-08-01",
				"check_out": "2025-08-03",
				"guests": 2
			},
			"pending": ["hotel_grade"],    // Slots to be collected
			"optional": {
				"breakfast": true
			}
		},
		"user_profile": {
			"name": "Zhang San",
			"membership": "Gold member",
			"preferences": ["high floor", "non-smoking room"]
		},
		"constraints": {
			"budget_max": 800,
			"currency": "CNY"
		},
		"history_summary": "User has confirmed check-in dates and guest count, currently selecting hotel grade.",
		"last_action": "ask_hotel_grade",
		"turn_count": 5
	}
\end{verbatim}

#### Integration with LLM

Structured states typically work collaboratively with LLMs through the following process:

1. **State Rendering**: Before each request turn, the current state object is serialized and inserted into the system message or user message of the prompt, informing the model of currently known information;
2. **State Extraction**: The LLM is required to output a State Patch in JSON format while generating the response, for example:
    \begin{verbatim}
    {
    "slots.confirmed.hotel_grade": "5-star",
    "slots.pending": [],
    "last_action": "confirm_all_slots",
    "status": "ready_to_book"
    }
    \end{verbatim}
3. **State Merge**: The backend merges the state patch returned by the LLM with the current state, persists it to the database, and uses it as input for the next turn's request;
4. **State Validation**: Schema validation (e.g., field types, enum value ranges) is performed on the merged state, rejecting illegal updates to prevent LLM hallucinations from contaminating the state.

The overall data flow can be represented as:


$$
	S_{t+1} = Merge(S_t,\ Patch_t), \quad Patch_t = LLM(Render(S_t) \| Q_t)
$$


Where $S_t$ is the state object at turn $t$, $Q_t$ is the user input, $Render(\cdot)$ is the state serialization function, and $Merge(\cdot)$ is the state merge function.

#### Limitations and Applicability Boundaries

- **High Schema Design Cost**: Business stakeholders need to define the complete state Schema in advance, making it difficult to pre-exhaust all fields for open-domain dialogues or scenarios with ambiguous requirements;
- **Loss of Unstructured Information**: Information that is hard to structure, such as the user's emotions, phrasing style, and implicit preferences, cannot be captured, which may result in a lack of personalization in responses;
- **Applicable Scenarios**: Structured state is most suitable for scenarios with **clear task boundaries**, such as e-commerce ordering, flight booking, form filling, and IT ticket processing; for open-domain chitchat, creative writing, and similar scenarios, rolling summarization or full history are usually more appropriate choices.

</details>

**7. Give an example of a structured state and explain how it is used in a dialogue.**

<details>
<summary>💡 Show Answer</summary>

Taking restaurant booking as an example:
\begin{lstlisting}[style=promptstyle]
	{
		"restaurant": "Some Restaurant",
		"date": "2025-06-20",
		"time": "19:00",
		"guests": 4,
		"special_requests": "Window seat",
		"confirmed": false
	}
\end{lstlisting}

After each turn of dialogue, have the model update this JSON based on the user's input. In the next turn's request, inject the state into the system prompt: "The current booking state is as follows: \{state\}", allowing the model to continue the task without relying on the raw dialogue.

</details>

**8. Which scenarios is structured state suitable for? Which is it unsuitable for?**

<details>
<summary>💡 Show Answer</summary>

Structured state is not a silver bullet; its applicability highly depends on the characteristics of the business scenario. The core criteria are: **whether the dialogue goal is clear, whether key information is enumerable, and whether task completion criteria are verifiable**.

#### Suitable Scenarios

- **Task-oriented Dialogues (booking, form filling, configuration wizards)**: The set of slots in these scenarios is limited and predefined, and the sole goal of the dialogue is to collect all required fields and trigger downstream actions. For example, flight booking requires collecting fixed fields such as "origin, destination, date, cabin class, passenger information". Structured state naturally fits these needs and can quantify task completion as "number of filled slots / total number of slots".

- **Multi-step Workflows (clear states, limited fields)**: In scenarios like IT operations automation, approval processes, and software installation wizards, the inputs and outputs of each step are clearly defined, and the results of previous steps need to be referenced by subsequent steps. Structured state can precisely record "which step it is currently on, which steps are completed, and what conditions are met", avoiding workflow interruptions caused by context truncation.

- **Multi-Agent Collaboration (Shared Structured State)**: In multi-agent systems, different agents may take on different subtasks (e.g., one for information collection, another for decision-making, and a third for execution). Structured state serves as **Shared Memory** among agents, naturally solving the problem of cross-agent information transfer and avoiding the overhead of redundantly transmitting the full dialogue history across multiple agents. Each agent only needs to read state fields related to its responsibilities and write fields it produces, achieving loosely coupled collaboration.

- **Long-term User Profile Management**: For products that need to remember user preferences across sessions (across days or weeks), such as personal assistants or tutoring systems, persisting users' stable attributes (style preferences, historical goals, learned concepts) into a structured profile and injecting it in each new dialogue is far more efficient than extracting them anew from the dialogue history.

- **Compliance and Auditable Scenarios**: Heavily regulated industries like finance, healthcare, and law require retaining auditable records for key decision fields. Structured state provides clear field-level operation logs, making it easy to retrospectively trace "which field was confirmed in which dialogue turn, and what the value was", whereas plain text history struggles to support precise field-level auditing.

#### Unsuitable Scenarios

- **Open-ended Chitchat (emotions, random topics, hard to structure)**: Topics in chitchat dialogues are highly dynamic, and dimensions such as emotional expression, metaphors, and humor can hardly be captured by limited fields. Forcing structurization not only makes Schema design difficult but also causes a massive loss of valuable contextual information, making responses lack warmth and personalization. These scenarios are better suited for **rolling summarization** or **full history** strategies.

- **Scenarios Requiring Exact Verbatim Reproduction (e.g., legal records, verbatim analysis)**: Scenarios like court testimony analysis, clause-by-clause contract review, and line-by-line code review require preserving original wording. Any summarization or structured extraction carries the risk of information distortion; original text is irreplaceable.

- **Exploratory Dialogues with Hard-to-predefine State Fields**: In scenarios like product requirement exploration, creative brainstorming, and philosophical discussions, the direction of the dialogue may undergo fundamental shifts at any time, and dimensions of key information cannot be exhausted in advance. If a Schema is forced, either the fields will be too broad and lose meaning, or frequent Schema expansions will bring high maintenance costs.

- **Scenarios Where Original Wording is the Subject of Analysis**: In tasks like sentiment analysis, style transfer, and linguistic research, the user's word choices, sentence structures, and tone changes are themselves core analysis dimensions, and structured extraction would destroy this information.

- **Early-stage Products with Frequent Schema Evolution**: If a product is in a rapid iteration phase and business requirements have not stabilized, modifying the Schema frequently will lead to compatibility issues with historical state data, introducing additional data migration engineering costs.

#### Decision Framework

In actual engineering, the following questions can be used to quickly determine whether to adopt structured state:

1. Does the dialogue have a clear **end state** (task completion sign)? *Yes $\to$ Lean towards structured*
2. Can key information be enumerated by **limited fields**? *Yes $\to$ Lean towards structured*
3. Is there a need to share information **cross-session** or **cross-Agent**? *Yes $\to$ Strongly recommend structured*
4. Is the original wording **irreplaceable**? *Yes $\to$ Not suitable for structured, retain raw text*
5. Is the business Schema **stable** enough? *No $\to$ Postpone introduction, use summarization strategy to transition first*

In most production systems, the optimal solution is usually a **hybrid strategy**: use structured state for enumerable key fields, and retain recent raw history or rolling summaries for unstructured context; the two are complementary rather than mutually exclusive.

</details>

**9. How should the summarization Prompt be written in rolling summarization?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How should the summarization Prompt be written in](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig51_how_should_the_summarization_p.png)



The design quality of the summarization Prompt directly determines the usability of the compressed information. A good summarization Prompt needs to clarify three core elements: **Compression Goal** (how many words to compress to, what type of information to retain), **Retention Key Points** (what content absolutely cannot be lost), and **Output Format** (plain text, structured paragraphs, or bullet points).

#### Design Principles
- **Clarify Information Priority**: Tell the model which information "must be retained" and which "can be omitted," preventing the model from making its own judgments that lead to the loss of key information.
- **Limit Output Length**: Give a clear character or token limit to prevent the summary from inflating and losing the point of compression.
- **Stipulate Narrative Perspective**: It is recommended to use third-person objective description to avoid mixing the model's subjective judgments into the summary.
- **Conflict Handling Rules**: Pre-specify which one to keep when the user's earlier and later statements contradict each other.

#### Basic Example
\begin{lstlisting}[style=promptstyle]
	Please compress the following dialogue history into a concise summary (no more than 200 words).
	Requirements:
	- Retain all needs, preferences, and decisions explicitly expressed by the user.
	- Retain important numbers, dates, names, and locations.
	- Omit greetings, repeated confirmations, the model's explanatory remarks, and other redundant content.
	- Use third-person perspective to describe user behavior (e.g., "The user wants..." "The user confirmed...").
	- If the user has expressed multiple statements on the same matter, only retain the latest statement.

	Dialogue History:
	{history}
\end{lstlisting}

#### Enhanced Example (with Structured Output)

When downstream systems need to parse the summary, the model can be required to output a structured summary:

\begin{lstlisting}[style=promptstyle]
	Please compress the following dialogue history into a structured summary.

	Output format:
	## User Core Needs
	(Summarize the user's main goal in one sentence)

	## Key Information
	- Important numbers/dates/names: ...
	- User's explicit preferences or constraints: ...
	- Decisions reached: ...

	## Pending Items
	(Content that the user has not clarified or has ambiguity)

	Requirements:
	- Total word count not exceeding 300 words.
	- Only record information that appears in the dialogue, do not make inferences.
	- For matters where the user changes their mind, only keep the latest version.

	Dialogue History:
	{history}
\end{lstlisting}

#### Few-shot Enhancement

For complex scenarios, adding examples to the Prompt yields better results:

\begin{lstlisting}[style=promptstyle]
	[Example Input Snippet]
	User: I want to book a flight to Shanghai, preferably tomorrow morning.
	Assistant: Okay, which city are you departing from?
	User: Beijing, with a budget not exceeding 800 RMB.
	Assistant: Understood, let me check for you.
	User: By the way, change it to the day after tomorrow, I have something else tomorrow.

	[Example Output Summary]
	User plans to take a flight from Beijing to Shanghai.
	Departure date has been updated from tomorrow to the day after tomorrow (updated).
	Ticket price budget limit is 800 RMB.
\end{lstlisting}

</details>

**10. How to decide the trigger timing for rolling summarization?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to decide the trigger timing for rolling summa](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig52_how_to_decide_the_trigger_timi.png)



Choosing the trigger timing requires balancing **information completeness** and **computational cost**. Triggering too early will waste compression calls and lose recent details; triggering too late may cause context overflow and force history truncation.

#### Common Trigger Strategies
- **Fixed-turn Triggering**: Trigger compression every $N$ dialogue turns (e.g., every 10 turns). Simple to implement, but inflexible—sometimes 10 turns are very short and don't need compression at all, and sometimes 3 turns have already filled the window.
- **Token Threshold Triggering**: Triggered when the cumulative history token count exceeds a certain proportion of the context window (usually 50%–70%). This is the most direct way because it is directly linked to the context window pressure, and is recommended as the primary strategy.
- **Time Triggering**: Trigger compression after a certain time interval has elapsed (e.g., 30 minutes with no new messages). Suitable for asynchronous dialogue scenarios (like ticketing systems), utilizing idle time to pre-compress without occupying the response link.
- **Semantic Triggering**: Triggered when a topic switch is detected, compressing and archiving the previous topic paragraph. Complex to implement, but summary quality is high, and information boundaries are clear.
- **Hybrid Strategy**: Use Token threshold as the primary trigger condition, overlaid with minimum turn protection—even if the threshold is reached, at least the raw dialogue of the most recent 3–5 turns is retained uncompressed, ensuring the model can see complete immediate context.
#### Recommended Implementation Logic

\begin{lstlisting}[style=promptstyle]
	def should_trigger_summary(history_tokens, window_size, recent_turns):
	"""
	history_tokens : Token count of the current history
	window_size    : Model context window size
	recent_turns   : Number of recent dialogue turns
	"""
	threshold = window_size * 0.6       # 60% as the trigger threshold
	min_turns_before_compress = 3       # Keep at least the 3 most recent turns uncompressed

	if history_tokens > threshold and recent_turns >= min_turns_before_compress:
	return True
	return False
\end{lstlisting}

#### Strategy Comparison
\begin{table}[h]



| **Strategy** | **Implementation Complexity** | **Flexibility** | **Recommended Scenario** \\ \hline |
| --- | --- | --- | --- |
| Fixed Turns | Low | Low | Rapid Prototyping \\ \hline |
| Token Threshold | Low | High | Most Production Scenarios \\ \hline |
| Time-based Trigger | Low | Medium | Asynchronous/Customer Service Systems \\ \hline |
| Semantic Trigger | High | High | Long-term Complex Tasks \\ \hline |
| Hybrid Strategy | Medium | High | Production-grade Recommended Solution \\ \hline |


\end{table}

</details>

**11. How to Handle Structured State Update Failures?**

<details>
<summary>💡 Show Answer</summary>

Structured states rely on the model outputting valid JSON or a specific format, but model outputs are inherently probabilistic, making update failures inevitable. Common failure types include: **Field Omission** (the model fails to output certain fields), **Format Error** (invalid JSON, type mismatch), **Value Hallucination** (the model generates non-existent values out of thin air), and **Field Contamination** (fields not discussed in the current turn are incorrectly modified).

#### Handling Strategies
1. **Schema Validation (First Line of Defense)**: Use Pydantic (Python) or JSON Schema to strictly validate the model output. When validation fails, re-request the model with the error message attached, retrying a maximum of 2-3 times.

    \begin{lstlisting}[style=promptstyle]
    from pydantic import BaseModel, ValidationError

    class TaskState(BaseModel):
    destination: str
    budget: int
    departure_date: str

    def update_state(model_output: str, retries=2):
    for attempt in range(retries):
    try:
    state = TaskState.model_validate_json(model_output)
    return state
    except ValidationError as e:
    model_output = call_model_with_error(model_output, str(e))
    raise RuntimeError("State update failed multiple times, triggering rollback")
    \end{lstlisting}

2. **Delta Updates (Reducing Error Surface)**: Do not require the model to return the complete state object; only let it output the **fields that changed** in the current turn. This keeps the single output small, lowering the error probability, and also clearly distinguishes between "the model did not mention it" and "the model considers it unchanged."

    \begin{lstlisting}[style=promptstyle]
    # Delta Update Prompt Example
    Based on the latest round of conversation, output the fields in the state that need to be updated.
    Only output the fields that have changed; do not output unchanged fields.
    The output format is JSON, for example:
    {"destination": "Osaka", "budget": 5000}

    If the current turn does not involve any state changes, output an empty object: {}
    \end{lstlisting}

3. **Version Rollback (Final Safety Net)**: After every successful state update, store the previous valid state in a rollback slot. When the current update ultimately fails, restore to the previous valid state and log the reason for failure, ensuring the system does not crash due to state corruption.

4. **Field-Level Protection**: Set write protection for critical fields (such as user ID, order number, amount)—modification is only allowed when specific conditions are met (like explicit user confirmation), preventing the model from arbitrarily overwriting critical data.

5. **Human Fallback**: When critical fields change, or consecutive update failures occur, trigger a manual review process where a human confirms and manually updates the state. This is suitable for high-value, high-risk business scenarios.

#### Failure Handling Decision Tree
\begin{lstlisting}[style=promptstyle]
	Model Outputs State
	│
	├─ Schema Validation Passed?
	│       ├─ Yes → Delta merge into current state → Save new version
	│       └─ No → Retry with error (Max 2 times)
	│                   ├─ Retry Successful → Merge and save
	│                   └─ Retry Failed → Rollback to previous valid version
	│                                   └─ Log error + Trigger alert
\end{lstlisting}

</details>

**12. How to Evaluate the Quality of Rolling Summaries?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to Evaluate the Quality of Rolling Summaries?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig53_how_to_evaluate_the_quality_of.png)



Evaluating summary quality requires comprehensive measurement across multiple dimensions; a single metric easily leads to one-sided optimization.

#### Core Evaluation Dimensions
- **Information Retention Rate**: Extract a set of key entities (names, places, numbers, decision items) from the original dialogue and check if these entities appear in the summary. This can be quantified using the Entity F1 score: $F1 = \frac{2 \cdot precision \cdot recall}{precision + recall}$.

- **Compression Ratio**: $Compression Ratio = \frac{Summary Token Count}{Original History Token Count}$. A reasonable range is 10%-30%. Too high a compression ratio means the summary isn't compressing; too low may cause significant information loss.

- **Downstream Task Accuracy (Most Important Metric)**: Build a standard Q&A test set, answer the same questions using the original history and the summarized history separately, and compare the answer consistency rate. If the summary-based answers highly align with the original-based answers, the summary quality is good.

- **Consistency (Stability)**: For the same segment of history, generate summaries multiple times with temperature=0, and check if the results are stable. Unstable summaries lead to unpredictable system behavior.

- **Hallucination Rate**: The proportion of information appearing in the summary that **does not exist** in the original dialogue. Hallucinated information can mislead subsequent turns and requires close monitoring.

#### Automated Evaluation Pipeline Design

\begin{lstlisting}[style=promptstyle]
	Evaluation Process:

	1. Collect Test Set
	Original Dialogue History D → Human-annotated Key Entity Set E
	Design 10-20 standard questions Q and reference answers A_ref for D

	2. Generate Summary
	Summary Model(D) → Summary S

	3. Metric Calculation
	a. Entity Retention Rate = |E ∩ entities(S)| / |E|
	b. Compression Ratio     = tokens(S) / tokens(D)
	c. Downstream Accuracy = Consistency between Answer(Q, context=S) and A_ref
	d. Hallucination Rate     = |entities(S) - entities(D)| / |entities(S)|

	4. Summary Report
	Output scores for each dimension, flag summaries below thresholds for manual review
\end{lstlisting}

#### Evaluation Threshold Reference
\begin{table}[h]



| **Metric** | **Pass Mark** | **Excellent Mark** \\ \hline |
| --- | --- | --- |
| Entity Retention Rate | $\geq 85%$ | $\geq 95%$ \\ \hline |
| Compression Ratio | $10%\sim30%$ | $15%\sim25%$ \\ \hline |
| Downstream Accuracy | $\geq 80%$ | $\geq 90%$ \\ \hline |
| Hallucination Rate | $\leq 5%$ | $\leq 1%$ \\ \hline |


\end{table}

</details>

**13. What is the Essential Difference between Rolling Summaries and RAG Memory Solutions?**

<details>
<summary>💡 Show Answer</summary>

The two have a fundamental difference in design philosophy: rolling summaries involve **lossy compression followed by full injection**, while RAG involves **lossless storage followed by on-demand retrieval**. Understanding this essential difference helps make the correct technical choices in different scenarios.

#### Core Comparison
\begin{table}[h]



| **Dimension** | **Rolling Summary** | **RAG Memory** \\ \hline |
| --- | --- | --- |
| Storage Format | Compressed text, injected into context | Vector database, retrieved on demand \\ \hline |
| Retrieval Method | Full injection (no retrieval needed) | Semantic similarity retrieval \\ \hline |
| Suitable Scale | History within a single session | Cross-session, massive history \\ \hline |
| Information Integrity | Possible compression loss | Retains original fragments \\ \hline |
| Implementation Complexity | Low | High (requires vector DB, Embedding) \\ \hline |
| Retrieval Latency | No extra latency | Requires retrieval time (tens to hundreds of ms) \\ \hline |
| Recall Controllability | Fully controllable (full injection) | Relies on similarity, possible missed recall \\ \hline |
| Storage Cost | Low (text only) | High (vector index) \\ \hline |


\end{table}

#### Respective Applicable Scenarios
- **Rolling Summaries are better suited for**: Continuous conversations within a single session, limited dialogue turns (within dozens), scenarios where the model needs to see all historical information, and when engineering resources are limited.
- **RAG Memory is better suited for**: Long-term user memory across sessions, massive historical data (hundreds of sessions), scenarios where only a portion of history is relevant to the current question, and when precise recall of specific historical fragments is required.

#### Combined Architecture

The two are not mutually exclusive; production systems usually combine them:

\begin{lstlisting}[style=promptstyle]
	Context Construction Logic:

	[Current Session]
	└─ Recent 5 turns: Original dialogue (Full)
	└─ Earlier history: Rolling summary (Compressed and injected)

	[Cross-Session Long-Term Memory]
	└─ RAG Retrieval: Recall relevant fragments from historical sessions based on current Query

	Final Injected Context = Long-Term Memory Recalled Fragments + Session Summary + Recent Original Dialogue
\end{lstlisting}

This layered design balances **the integrity of immediate context** with **the scalability of long-term memory**.

</details>

**14. How Do Structured State and Rolling Summaries Collaborate in Multi-Turn Dialogues?**

<details>
<summary>💡 Show Answer</summary>

Structured states and rolling summaries are not competing, but rather **two complementary memory layers**, responsible for different types of information. Using them in isolation often leads to states being either too rigid (only structured state) or the context being too ambiguous (only summary).

#### Responsibility Division
- **Structured State**: Responsible for storing **enumerable, verifiable** core task variables. For example: target city, budget limit, completed steps, current phase. This type of information needs to be read and written accurately by programs, so it must be structured.
- **Rolling Summary**: Responsible for storing **hard-to-structure** narrative backgrounds. For example: user sentiment, discussed options and reasons for rejection, soft preferences ("the user doesn't like places that are too commercialized"). This type of information is best described in natural language.

#### Typical Collaboration Scenario

Taking a travel planning assistant as an example:
- Structured state record: `\{"destination": "Osaka", "budget": 8000, "days": 5, "confirmed": true\}`
- Rolling summary record: "The user initially considered Tokyo, then changed to Osaka due to pricing. The user prefers an artsy vibe, explicitly stating they don't want to go to too many shopping malls. Prioritizes culinary experiences over accommodation class."

Combined, the model can obtain precise constraint parameters while understanding the user's deeper preferences, thereby providing more appropriate suggestions.

#### Context Injection Template
\begin{lstlisting}[style=promptstyle]
	[System]: You are a travel planning assistant.

	## Current Task State (Structured)
	{structured_state}

	## Dialogue Background Summary
	{rolling_summary}

	## Recent Dialogue Records (Recent 3 turns, Original)
	{recent_history}

	[User]: {new_message}
\end{lstlisting}

#### Update Timing

Updates to both should be **triggered independently** and not coupled together:
- Structured State: Attempt delta update after **every turn** (low cost).
- Rolling Summary: Trigger compression when **token threshold is reached** (higher cost, unsuitable for per-turn calls).

</details>

**15. How to Handle Information Conflicts in Summaries? For Example, the User Changes Their Mind Later.**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to Handle Information Conflicts in Summaries?](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig54_how_to_handle_information_conf.png)



Information conflicts are high-frequency issues in multi-turn dialogues. When users change their minds, if the summary fails to handle it correctly, the coexistence of old and new information will cause confusion in model behavior.

#### Define Conflict Rules at the Prompt Level

The summary Prompt must explicitly specify conflict handling strategies:

\begin{lstlisting}[style=promptstyle]
	Conflict Handling Rules:
	- If the user expresses opinions multiple times on the same matter during the dialogue,
	only keep the latest opinion, and mark it with "(Updated)" in parentheses.
	- If the user explicitly revokes a previous decision,
	note in the summary that "Original plan X has been cancelled."
	- Do not retain both the old and new versions in the summary simultaneously,
	to prevent ambiguity when the model uses the summary.
\end{lstlisting}
#### Preserving Change History in Structured State

Merely labeling "updated" with text is not enough. It is recommended to also record the change trajectory in the state object, allowing both the model and the program to perceive the history:

\begin{lstlisting}[style=promptstyle]
	{
		"destination": "Osaka",
		"destination_updated_at": "Turn 5",
		"destination_history": ["Tokyo (Turn 1)", "Osaka (Turn 5, updated)"],
		"budget": 8000,
		"budget_updated_at": "Turn 3"
	}
\end{lstlisting}

#### Change-Aware Summary Generation

When a change in a state field is detected, a change prompt can be injected into the summary prompt to guide the model to specifically highlight the change in the summary:

\begin{lstlisting}[style=promptstyle]
	Note: The following fields changed during this turn,
	please explicitly reflect these changes in the summary:
	- destination: Tokyo -> Osaka
	Please use expressions like "The user changed the destination from Tokyo to Osaka (updated)".
\end{lstlisting}

#### Conflict Detection Mechanism

At the system level, an automatic conflict detection mechanism can be added: compare the old and new state objects. If a core field changes, an automatic user confirmation prompt is triggered to prevent accidental updates:

\begin{lstlisting}[style=promptstyle]
	Critical information change detected:
	Destination: Tokyo -> Osaka
	Are you sure you want to change the destination to Osaka? (Yes/No)
\end{lstlisting}

</details>

**16. How is the storage and architecture of rolling summaries designed in large-scale production systems?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How is the storage and architecture of rolling sum](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig55_how_is_the_storage_and_archite.png)



The core challenges faced by production-grade rolling summary systems include: high concurrency performance requirements, summary generation not blocking the response chain, traceability of historical versions, and state consistency across services.

#### Storage Layer Design
- **Hot Data Layer (Redis)**: Stores the `\{session_id, summary, recent_history, state\}` of current active sessions. Low read/write latency ($<$1ms), suitable for high-frequency access. Set a TTL (e.g., 24 hours), which automatically expires after the session ends.
- **Persistence Layer (Relational/Document Database)**: Stores all historical versions of summaries and states, supporting backtracking audits. A combination of PostgreSQL (structured state) + MongoDB (unstructured summary text) can be used.
- **Vector Layer (Optional)**: Vectorizes historical summaries and stores them in vector databases like Pinecone/Weaviate, supporting cross-session semantic retrieval.

#### Asynchronous Summarization

Summary generation is a high-cost operation (requires calling the LLM) and should not block user responses:

\begin{lstlisting}[style=promptstyle]
	Request processing flow:

	User sends a message
	|
	+- Synchronous: Call LLM to generate response -> return to user immediately
	|
	+- Asynchronous (Message Queue):
	Check if the summary threshold is triggered
	+- Yes -> Push compression task to the queue (Celery/Kafka)
	Worker asynchronously calls LLM to generate a summary
	Writes to Redis + DB upon completion
\end{lstlisting}

#### Version Management

Each summary update does not overwrite the old version, but appends a new version:

\begin{lstlisting}[style=promptstyle]
	summary_versions table schema:
	{
		session_id   : "sess_abc123",
		version      : 3,
		summary_text : "...",
		covered_turns: [1, 15],   // Range of dialogue turns covered by this summary
		created_at   : "2025-06-01T10:23:00Z",
		token_count  : 186
	}
\end{lstlisting}

#### Hierarchical Summary Architecture

For ultra-long sessions, the summaries themselves can be summarized again, forming a tree structure:

\begin{lstlisting}[style=promptstyle]
	Level 0: Original dialogue turns (the text of the last 5 turns is retained)
	Level 1: Single compression summary (compressed every 15 turns, covering the current window)
	Level 2: Session-level summary (re-compressing multiple Level-1 summaries)
	Level 3: User-level long-term summary (cross-session user profiles and long-term preferences)
\end{lstlisting}

#### Monitoring Metrics
- Summary trigger frequency (too high indicates history is growing too fast)
- Summary generation latency (P50/P99)
- Compression ratio distribution
- Summary generation failure rate (triggers alert threshold)
- Downstream QA accuracy rate (offline periodic evaluation)

</details>

**17. How does structured state support multi-Agent collaboration scenarios?**

<details>
<summary>💡 Show Answer</summary>

In multi-Agent systems, structured state acts as a **shared Blackboard**—all Agents coordinate actions by reading from and writing to the same state object, avoiding duplicate work and information silos.

#### Core Design Principles
- **Shared Reading**: All Agents can read the global state to get current task progress, constraints, and existing conclusions without repeatedly asking the user.
- **Atomic Writing**: State writing must be atomic to prevent data races caused by concurrent writes from multiple Agents. Database transactions, Redis distributed locks, or message queues (only one Agent holds write permission at a time) can be used.
- **Field Partitioning**: Different Agents are responsible for different fields of the state to reduce write conflicts. The Planner Agent only writes the `plan` field, the Executor Agent only writes the `execution_log`, and the Evaluator Agent only writes the `quality_score`.
- **Event-Driven Notifications**: After a state change, the Agents subscribing to that field are notified via an event bus (e.g., Redis Pub/Sub, Kafka) to trigger their subsequent actions.

#### State Structure Example

\begin{lstlisting}[style=promptstyle]
	{
		// Maintained by Orchestrator Agent
		"task_id": "task_001",
		"current_phase": "execution",
		"overall_goal": "Plan a 5-day Osaka itinerary for the user",

		// Written by Planner Agent
		"plan": {
			"days": [...],
			"confirmed": true,
			"planned_at": "turn_3"
		},

		// Written by Executor Agent
		"execution_log": [
		{"action": "Query hotel", "status": "done", "result": "..."},
		{"action": "Book restaurant", "status": "in_progress"}
		],

		// Written by Reviewer Agent
		"quality_score": null,
		"review_notes": []
	}
\end{lstlisting}

#### Typical Collaboration Workflow

\begin{lstlisting}[style=promptstyle]
	1. Orchestrator reads global state to determine the current phase
	2. Orchestrator distributes subtasks to corresponding SubAgents
	3. SubAgent executes and writes results to its responsible fields
	4. State change event triggers Orchestrator to re-read the state
	5. Orchestrator decides the next step: continue distribution / request user confirmation / end task
\end{lstlisting}

#### Concurrent Conflict Prevention

\begin{lstlisting}[style=promptstyle]
	# Using Redis distributed lock to protect state writes
	def update_state_safe(session_id, field, value):
	lock_key = f"lock:state:{session_id}"
	with redis.lock(lock_key, timeout=5):
	current = redis.get(f"state:{session_id}")
	current[field] = value
	redis.set(f"state:{session_id}", current)
\end{lstlisting}

</details>

**18. How to prevent "State Drift" of structured state in long conversations?**

<details>
<summary>💡 Show Answer</summary>

![Technical visualization of How to prevent "State Drift" of structured state i](https://img.pricenode.uk/llm_interview_questions/english/chapter2/fig56_how_to_prevent_state_drift_of_.png)



State drift refers to the gradual accumulation of tiny updating errors by the model on the state as dialogue turns increase, ultimately causing the state to significantly deviate from the user's true intent. Drift is often progressive and hard to spot in a single turn, but can severely impact system performance later on.

#### Causes of Drift
- When updating the delta, the model slightly tweaks the phrasing or values of unchanged fields.
- Summary compression loses critical constraints, causing subsequent state updates to lose their reference.
- The model's understanding of field semantics drifts as the context changes.
- State versions become chaotic after multiple retries and rollbacks.

#### Preventive Measures
1. **Strong Type Constraints**: Define strict types and enumeration value ranges for each field. The model cannot write invalid values, limiting the space for drift at the source.

    \begin{lstlisting}[style=promptstyle]
    class TaskState(BaseModel):
    destination: Literal["Tokyo", "Osaka", "Kyoto", "Other"]
    budget: int  # Must be integer, unit: RMB
    travel_style: Literal["Arts", "Food", "Nature", "Shopping"]
    status: Literal["Planning", "Confirmed", "Executing", "Completed"]
    \end{lstlisting}

2. **Periodic Verification**: Every $N$ turns (recommended 10-15 turns), ask the model to regenerate the values of key fields based on the **recent raw conversation**, and compare them with the current state. If the difference exceeds a threshold, a correction is triggered.

    \begin{lstlisting}[style=promptstyle]
    Verification Prompt:
    Based solely on the following recent 5 turns of raw conversation,
    please reconfirm the current values of the following fields:
    - destination
    - budget
    - travel_style

    Do not refer to the current state provided by the system.
    Judge entirely based on the raw conversation text.

    Recent 5 turns of conversation: {recent_raw_history}
    \end{lstlisting}

3. **User Confirmation Mechanism**: When a key field changes, actively display the current state to the user and request confirmation. The user's approval serves as a state "anchor", preventing irreversible drift.

4. **Change Log Auditing**: Record the triggering turn, changed content, and trigger reason for each state change, facilitating retrospective analysis of the starting point and propagation path of drift.

    \begin{lstlisting}[style=promptstyle]
    state_changelog record format:
    {
    "turn": 12,
    "field": "budget",
    "old_value": 8000,
    "new_value": 10000,
    "trigger": "User said 'budget can be relaxed a little'",
    "updated_by": "model_diff_update"
    }
    \end{lstlisting}

5. **Immutable Field Protection**: For fields explicitly set by the user at the beginning of the session and not mentioned later for modification, set a write-protect flag. They can only be changed if explicitly reset by the user.

</details>

**19. For ultra-long conversations (such as hundreds of turns in customer service systems), what are the optimization strategies for rolling summaries?**

<details>
<summary>💡 Show Answer</summary>

Standard rolling summaries face two core challenges in ultra-long conversations: **summary bloat** (the summary accumulates and eventually fills the window) and **information confusion** (early information is mixed with recent information, making it difficult for the model to distinguish priority). Addressing these two issues requires multi-level optimization strategies.

#### Detailed Optimization Strategies
- **Hierarchical Summarization**: Summarize the summary itself again to form a multi-level compression tree. Recent history is kept detailed, while distant history is progressively compressed into more abstract overviews. The higher the level, the fewer the details, but the core narrative thread is preserved.

    \begin{lstlisting}[style=promptstyle]
    Level 1 Summary (Turns 1-20):
    The user reported a delay issue with order #12345, provided the delivery address and
    contact info, and customer service promised to reply within 48 hours.

    Level 2 Summary (Level 1 + Re-compression of summaries from Turns 21-50):
    The user contacted customer service regarding order delay, the issue has been escalated.
    The user is quite agitated and has high expectations for the compensation plan.
    \end{lstlisting}

- **Topic-Aware Compression**: Detect topic switching points in the dialogue (e.g., switching from "delay complaint" to "refund request"), and summarize in segments by topic. The summary of each topic segment is independent, preventing information from different topics from contaminating each other.

- **Hybrid Extractive and Abstractive Summarization**: For highly important content (such as explicit user complaints, customer service promises), extract the original sentences directly to avoid hallucinations introduced by generative summaries; for background content, use generative summaries for compression.

- **Importance Annotation and Tiered Retention**: During summary generation, have the model simultaneously tag the importance level (High/Medium/Low) for each piece of information. When further compression is needed, prioritize removing low-importance content while retaining high-importance content.

    \begin{lstlisting}[style=promptstyle]
    Summary Generation Prompt (with importance tags):
    Please generate a summary and tag the importance for each piece of information:
    [High] User order ID: #12345, Issue: Logistics delayed by 7 days
    [High] Customer service promise: Arrange a replacement shipment within 3 days
    [Medium] User emotion: Quite agitated, urged multiple times
    [Low] User mentioned buying 3 times before, is a returning customer
    \end{lstlisting}
- **External Memory Offloading**: When the session length exceeds a certain threshold (e.g., 100 turns), offload early summaries to a vector database as RAG memory, remove them from the context, and retrieve them on demand. This ensures that only recent summaries and raw history remain in the context window, preventing infinite bloat.

    \begin{lstlisting}[style=promptstyle]
    Ultra-long session context structure:

    [RAG Retrieval Results] (Early summary fragments related to the current query)
    [Level-2 Session Summary] (Highly compressed overview of turns 1-80)
    [Level-1 Recent Summary] (Detailed summary of turns 81-95)
    [Raw Dialogue] (Last 5 turns, fully retained)
    [Current User Message]
    \end{lstlisting}

</details>

**20. What engineering trade-offs need to be considered when designing a complete conversational memory system?**

<details>
<summary>💡 Show Answer</summary>

There is no silver bullet for a complete conversational memory system. Every design decision involves trade-offs across multiple dimensions. Understanding these trade-off axes is essential to making reasonable choices based on specific business scenarios.

#### Core Trade-off Dimensions
\begin{table}[h]



| **Trade-off Axis** | **Option A** | **Option B** \\ \hline |
| --- | --- | --- |
| Information Completeness | Full History (High fidelity, high tokens) | Summary/State (Low tokens, lossy) \\ \hline |
| Response Latency | Synchronous Summary (Blocks response) | Asynchronous Summary (Non-blocking, slight delay) \\ \hline |
| Storage Medium | Memory/Redis (Fast, volatile) | Database (Persistent, slower) \\ \hline |
| Recall Method | Full Injection (No risk of missed recall) | Vector Retrieval RAG (Saves tokens, risk of missed recall) \\ \hline |
| Update Granularity | Per-turn update (Real-time, high cost) | Batch-triggered (Low cost, lag) \\ \hline |
| Implementation Complexity | Simple truncation (Easy maintenance) | Hierarchical Summary+RAG (Powerful, complex) \\ \hline |
| Interpretability | Raw dialogue is auditable | Hard to trace summary origins \\ \hline |


\end{table}

#### Recommended Tech Stacks by Scenario
- **Fast Prototyping / Low Complexity Scenarios**: Simple history truncation (retain last $N$ turns), no summary, easiest to implement. Suitable for Demos and internal tools.
- **Medium Complexity Production Scenarios**: Rolling summaries (triggered by token thresholds) + Structured state (delta updates per turn) + Redis caching.
- **High Complexity Production Scenarios (Recommended Architecture)**: Hierarchical memory system, asynchronous writes at each layer.

#### Recommended Production-Grade Architecture (Hierarchical Memory)
- **Layer 0 — Immediate Context**: Last 5-10 turns of raw dialogue, full injection, lossless, auto-sliding window update per turn.
- **Layer 1 — Session Summary**: Rolling summary, token threshold triggered, asynchronously generated, injected into context.
- **Layer 2 — Structured State**: Core task variables, delta updates per turn, Schema validation, supports version rollback.
- **Layer 3 — Long-term Memory**: Cross-session history vectorized and stored in RAG, semantically retrieved by current Query, injected on demand.
- **Layer 4 — User Profile**: Cross-session user preferences and behavioral patterns, periodically updated, acts as part of the System Prompt.

#### Architecture Diagram
\begin{lstlisting}[style=promptstyle]
	User message arrives
	|
	+- [Read] Redis: Recent history + Session summary + Structured state
	+- [Retrieve] Vector DB: Long-term memory fragments related to current message
	|
	+- Construct Context: Long-term memory + User profile + State + Summary + Recent history
	|
	+- Call LLM -> Generate response -> Return to user immediately
	|
	+- Asynchronous write queue:
	+- Update structured state (delta)
	+- Check summary threshold, trigger compression if met
	+- Periodically write summary to vector DB (long-term memory)
\end{lstlisting}

#### Key Engineering Principles
- **Zero Blocking of Response Chain**: All memory update operations (summary generation, state writing, vectorization) execute asynchronously. The user only waits for the LLM to generate a response.
- **Degradation Strategy**: If any memory layer fails, the system should be able to run in a degraded mode (e.g., use truncated history directly if the summary service fails) rather than crashing completely.
- **Observability**: Add monitoring metrics for each memory layer operation to facilitate locating performance bottlenecks and quality issues.
- **Progressive Evolution**: Start with the simplest history truncation and gradually introduce summaries, states, and RAG based on actual business pain points to avoid over-engineering.

</details>

