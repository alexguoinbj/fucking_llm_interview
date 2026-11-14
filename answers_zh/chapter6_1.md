# 6.1 Agent 基础概念与 Agentic Loop — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 6 章：Agent · 共 29 题

**1. 什么是 Agent?**

<details>
<summary>💡 查看答案</summary>

LLM Agent 是一种人工智能系统，它利用大型语言模型（LLM）作为其核心计算引擎，展示文本生成之外的功能，包括进行对话、完成任务、推理，并可以展示一定程度的自主行为。LLM Agent 根据设计阶段授予的功能，从纯粹的被动到高度主动的自主行为。同时利用大模型的推理能力，让 Agent 可以在人工监督下管理相对独立的工作流程：分析目标，项目规划，执行，回顾过去的工作，迭代细化。

</details>

**2. 什么是 AI Agent？它与普通 LLM 对话有什么本质区别？**

<details>
<summary>💡 查看答案</summary>

AI Agent 是以 LLM 为核心，具备工具调用、规划和记忆能力，能够感知环境、制定计划、执行动作、观察反馈，循环完成复杂多步骤任务的系统。与普通对话的本质区别：\begin{enumerate}
- 有主动行动能力（调用 API、执行代码、浏览网页）；\item  存在反馈循环（观察→思考→行动），而非单轮响应；\item  有状态记忆，跨步骤保持上下文；\item  目标驱动，而非消息驱动；\item  可以自主决策执行路径，不需要用户逐步指导。

</details>

**3. Agent 和 Chatbot 有什么区别？**

<details>
<summary>💡 查看答案</summary>

**Chatbot**：主要做对话交互，回答用户问题，输出是文本，无法直接操作外部系统，每轮独立响应。**Agent**：以完成目标为导向，能调用工具执行真实操作（写文件、发邮件、执行代码），多步骤自主执行，有状态记忆，结果影响真实世界。核心区别：Chatbot 是"说"，Agent 是"做"。一个高质量的 Chatbot 也可以是 Agent 的一部分（负责用户交互层），但 Agent 的能力远超 Chatbot。

</details>

**4. LLM Agent 有什么关键能力？**

<details>
<summary>💡 查看答案</summary>

- 利用 LLM 的语言能力理解指令、上下文和目标，可以根据人类提示自主和半自主操作。
- 可以利用工具套件（计算器、API、搜索引擎）来收集信息并采取行动来完成分配的任务。
- 可以做逻辑推理类型的任务，例如 chain-of-thought、tree-of-thought。
- 可以量身定制文本，例如邮件、报告、市场材料。
- 可以自动或半自动的响应用户的需求。
- Agent 可以和不同类型的 AI 系统对接，例如 LLM + image generators。

</details>

**5. LLM Agents 有哪些类型？**

<details>
<summary>💡 查看答案</summary>

一般来说 LLM Agents 分为会话型 Agents 和任务型 Agents：
- **Conversational Agents**：模拟人类对话，能够在讨论中反映人类的倾向。允许细致入微的上下文交互，会考虑语气、说话风格、领域知识、观点和个性怪癖等因素。
- **Task-Oriented Agents**：实现目标驱动，利用模型的能力分析 prompt、提取关键参数、指定计划、调用 API、通过集成 tools 执行操作，并生成结果回复。

</details>

**6. 怎样构建基于 LLM 的 Agents？**

<details>
<summary>💡 查看答案</summary>

Agent = LLM + Prompt Recipe + Tools + Interface + Knowledge + Memory

- **Prompt Recipe**：特定的内容要求、目标受众、所需的语气、输出长度、创造力水平等。
- **Tools**：工具集成允许通过 API 和外部服务完成任务。
- **Knowledge**：知识适用于所有用户的一般专业知识，分为专业知识、常识知识和程序知识。
- **Memory**：单个用户或单个任务的上下文和记录细节，分为短期记忆和长期记忆。

</details>

**7. 你如何定义一个基于 LLM 的智能体（Agent）？它通常由哪些核心组件构成？**

<details>
<summary>💡 查看答案</summary>

一个基于 LLM 的智能体（Agent）是一个能够自主理解环境、进行规划决策、并执行行动以达成特定目标的计算系统。其核心特征是利用一个大型语言模型（LLM）作为其"大脑"或"中央处理器"，来进行复杂的推理和决策。与传统的调用LLM进行问答或文本生成不同，Agent具有自主性和循环执行的特点，它能主动地、持续地与环境或工具交互，直到完成任务。

一个典型的LLM Agent通常由以下四个核心组件构成：

1. **大脑/核心引擎**：一个强大的大型语言模型（LLM），负责理解用户目标、感知环境信息、进行常识推理、制定计划、并决定下一步的行动。
2. **规划模块**：可以是LLM的内置能力（如通过CoT、ReAct等提示策略激发），负责将复杂目标分解为可执行子任务，并根据反馈动态调整计划。
3. **记忆模块**：弥补LLM有限的上下文窗口，分为短期记忆（当前对话历史、推理 scratchpad）和长期记忆（向量数据库存储的过去经验和知识）。
4. **工具使用模块**：一系列外部API、函数库或硬件接口，扩展Agent的能力边界，使其能获取实时信息、执行计算或与物理世界交互。

![基于大语言模型的智能体 (Agent) 核心组件架构](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter6/fig_agent_components.png)

</details>

**8. 什么是 Agentic Loop？描述其基本运行机制。**

<details>
<summary>💡 查看答案</summary>

Agentic Loop 是 Agent 的核心执行模式，基本机制为：\begin{enumerate}
- **感知**：接收当前状态（用户目标 + 工具返回结果 + 历史上下文）；\item  **思考**：LLM 分析当前状态，决定下一步行动；\item  **行动**：调用工具、请求用户输入或直接输出结论；\item  **观察**：获取行动结果，更新状态；\item  重复循环，直到满足终止条件（目标达成、步骤上限、用户中断）。每次循环都是一次完整的 LLM 推理，循环次数与任务复杂度成正比。

![Agentic Loop (感知-思考-行动-观察) 闭环运行机制](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter6/fig_agentic_loop.png)

</details>

**9. 到底什么让 Agent 有了自治的能力？**

<details>
<summary>💡 查看答案</summary>

通常有自制能力的系统，至少有两类 agent 组成。一个用于生成的 agent，一个用于监督的 agent。生成 agent 根据提示生成回复。监督 agent 在必要时审查和重新提示或指示生成 agent 继续工作，同时提供交互反馈。自主技能是通过持续提示培养出来的。专门的监督 agent 提供方向、纠正和不断提高挑战，持续的提示释放了推理、效能和自主决策能力的增长。

</details>

**10. Agent 的"感知-思考-行动-观察"四个阶段分别做什么？**

<details>
<summary>💡 查看答案</summary>

**感知**：收集当前可用信息，包括用户原始目标、已完成步骤的结果、工具返回值、外部环境状态。**思考**：LLM 基于当前信息推理，判断任务是否完成、下一步应该做什么、需要调用哪个工具、参数是什么。**行动**：执行决策——调用工具（搜索、代码执行、API 请求）、生成中间结果或最终输出，或向用户请求补充信息。**观察**：接收行动结果，将结果写入上下文，作为下一轮感知的输入，形成闭环。

</details>

**11. 什么是工具（Tool）？Agent 常见的工具类型有哪些？**

<details>
<summary>💡 查看答案</summary>

工具是 Agent 可以调用的外部能力接口，让 LLM 能够执行超出纯文本生成范围的操作。常见类型：\begin{enumerate}
- **信息检索类**：搜索引擎、向量数据库查询、知识图谱；\item  **代码执行类**：Python sandbox、Shell 命令执行；\item  **文件操作类**：读写文件、格式转换、图片处理；\item  **API 调用类**：日历、邮件、数据库、第三方 SaaS 服务；\item  **浏览器操作类**：网页抓取、表单填写、点击交互；\item  **通信类**：发送消息、通知、邮件；\item  **子 Agent 调用**：将复杂子任务委托给专门的 subagent。

</details>

**12. Agent 的"目标"（Goal）和"任务"（Task）有什么区别？**

<details>
<summary>💡 查看答案</summary>

**目标（Goal）**：用户期望达成的最终结果，通常是高层次、模糊的描述（"帮我整理本月的销售报告"）。**任务（Task）**：为实现目标而分解出的具体可执行步骤（"查询数据库获取本月销售数据"、"计算各产品销售额"、"生成图表"、"写摘要"）。Agent 的规划层负责将 Goal 分解为一系列 Task，每个 Task 可以由工具调用或子 Agent 执行。区别的重要性：Goal 由用户定义，Task 由 Agent 自主生成，Task 的质量决定 Goal 能否被高效实现。

</details>

**13. 什么是工具调用（Tool Call / Function Calling）？它的基本执行流程是什么？**

<details>
<summary>💡 查看答案</summary>

工具调用是 LLM 输出结构化指令，由宿主程序执行外部操作并将结果返回给 LLM 的机制。基本流程：\begin{enumerate}
- LLM 接收工具的 schema（名称、参数描述、返回值说明）；\item  LLM 决定调用某工具，输出结构化调用指令（JSON 格式，含工具名和参数）；\item  宿主程序解析调用指令，执行对应函数或 API；\item  执行结果以 tool_result 形式写回上下文；\item  LLM 读取结果后继续推理。LLM 本身不执行代码，只负责"决策调用什么"，实际执行由外部程序完成。

</details>

**14. 什么是 Orchestrator Agent 和 Subagent？各自的职责是什么？**

<details>
<summary>💡 查看答案</summary>

**Orchestrator Agent**（编排者）：接收用户的最终目标，负责任务分解、子任务分配、进度跟踪、结果汇总和错误处理，是整个 Agent 系统的"大脑"，不直接执行具体操作。**Subagent**（执行者）：接收 Orchestrator 分配的具体子任务，专注执行（代码运行、数据检索、文件处理等），将结果返回给 Orchestrator，通常职责单一、能力专精。两者分离的好处：关注点分离，便于独立优化和测试，Orchestrator 可以并行调度多个 Subagent。

</details>

**15. 什么是单 Agent 系统和 Multi-Agent 系统？各自适合什么场景？**

<details>
<summary>💡 查看答案</summary>

**单 Agent 系统**：一个 LLM 实例负责全部规划和执行，工具调用直接由该实例驱动。适合：任务相对简单、步骤数量有限（<20 步）、无需并行处理的场景。**Multi-Agent 系统**：多个 Agent 协作，Orchestrator 负责规划和协调，多个 Subagent 并行或串行执行子任务。适合：\begin{enumerate}
- 任务复杂、需要并行处理多个子任务；\item  不同子任务需要不同专业知识或工具集；\item  单个 Agent 的上下文窗口不足以容纳全部任务状态；\item  需要相互验证（一个 Agent 生成，另一个审查）。

</details>

**16. 什么是 Human-in-the-Loop（HITL）？哪些场景下必须有人工介入？**

<details>
<summary>💡 查看答案</summary>

HITL 是在 Agent 执行流程的关键节点插入人工审核/确认步骤，保持人类对高风险行为的控制权。必须介入的场景：\begin{enumerate}
- **不可逆操作**：删除文件、发送邮件、提交代码到生产环境；\item  **高金额财务操作**：转账、支付、合同签署；\item  **不确定性高**：Agent 置信度低时主动请示；\item  **超出授权范围**：任务要求访问未预先授权的资源；\item  **异常检测**：执行路径与预期差异大，Agent 自我评估可能出错时；\item  **法律/合规敏感**：需要人工对结果承担法律责任的决策。

</details>

**17. 什么情况下不适合用 Agent，直接用单次 LLM 调用更合适？**

<details>
<summary>💡 查看答案</summary>

以下情况直接用 LLM 调用更合适：\begin{enumerate}
- **任务简单**：单次推理即可完成，不需要多步骤；\item  **无需外部信息**：模型内置知识已经足够回答；\item  **无需执行操作**：只需生成文本，不需要调用工具；\item  **延迟敏感**：Agent 的多步循环增加延迟，实时性要求高时不适合；\item  **成本敏感**：Agent 的多次 LLM 调用成本远高于单次；\item  **确定性要求高**：Agent 的自主决策引入不确定性，需要精确可复现输出时不适合。经验法则：能用 prompt 解决的，不用 Agent。

</details>

**18. Agent 中的"状态"（State）是什么？它存储在哪里？**

<details>
<summary>💡 查看答案</summary>

Agent 状态是记录当前任务执行情况的所有信息，包括：用户原始目标、已完成的步骤及其结果、当前步骤、中间变量、工具调用历史、错误记录等。存储位置：\begin{enumerate}
- **短期**：LLM 的上下文窗口，每次推理直接可用，但有容量上限；\item  **结构化状态对象**：JSON/字典格式，明确记录关键状态字段，比原始对话历史更紧凑；\item  **外部存储**：数据库或文件系统，用于长任务的持久化，支持断点续跑；\item  **向量数据库**：语义化的长期记忆，按需检索历史信息。

</details>

**19. Agent 任务失败后的基本处理策略有哪些？**

<details>
<summary>💡 查看答案</summary>

1. **重试（Retry）**：对可恢复错误（网络超时、API 限流）自动重试，设置最大重试次数和指数退避间隔；\item  **回退（Fallback）**：切换到备用工具或降级方案（用搜索替代 API 调用）；\item  **人工介入**：不可恢复错误时暂停执行，通知用户并等待指令；\item  **状态回滚**：对不可逆操作前保存检查点，出错后恢复到已知好状态；\item  **错误上报**：记录完整错误上下文（工具名、参数、错误信息），便于事后诊断；\item  **优雅终止**：超过最大重试次数后，输出已完成的部分结果和失败原因，而不是静默退出。

</details>

**20. 什么是 Agent 的"终止条件"？如果没有终止条件会发生什么？**

<details>
<summary>💡 查看答案</summary>

终止条件是 Agent 停止 Agentic Loop 的判断标准，常见类型：\begin{enumerate}
- **目标达成**：LLM 判断任务已完成；\item  **步骤上限**：达到最大允许的工具调用次数；\item  **时间上限**：执行时间超过预设阈值；\item  **Token 预算耗尽**：上下文 token 达到上限；\item  **用户中断**：用户主动终止。若没有终止条件：Agent 可能陷入无限循环（反复调用工具但不收敛），导致 token 和 API 费用无限增长，甚至触发外部系统的速率限制或造成重复操作（重复发邮件、重复写入数据库）。

</details>

**21. Agent 如何获取上下文对话信息？**

<details>
<summary>💡 查看答案</summary>

- 获取全量历史对话
- 滑动窗口获取最近部分对话内容
- 获取历史对话中实体信息
- 利用知识图谱获取历史对话中的实体及其联系
- 对历史对话进行阶段性总结摘要
- 需要获取最新对话，又要兼顾较早历史对话
- 回溯最近和最关键的对话信息
- 基于向量检索对话信息

</details>

**22. 什么是 LangChain，为什么它很有用？**

<details>
<summary>💡 查看答案</summary>

LangChain 是一个用于开发由大型语言模型（LLMs）驱动的应用程序的开源框架，通过标准化的抽象和丰富的集成组件，大幅降低了构建复杂 LLM 应用的工程门槛。

1. **解决的核心问题**：LLM 应用通常需要将模型调用、外部工具、数据检索、状态记忆、多步骤流程等多个复杂组件组合在一起，从零实现需要大量重复的"胶水代码"。LangChain 提供了统一的抽象层和大量开箱即用的集成，让开发者专注于业务逻辑而非底层拼接。
2. **核心价值主张**：**统一 LLM 接口**——同一套代码接入 OpenAI、Anthropic、Hugging Face 等不同模型，一行代码切换；**RAG 全链路**——文档加载、分块、向量化、存储、检索的完整工具链；**Agent 框架**——ReAct 等 Agent 范式的开箱即用实现；**LCEL 流水线**——声明式的链式组合语法，大幅减少代码量。
3. **生态系统优势**：超过 600 个第三方集成（向量数据库、文档格式、API 服务）、活跃的开源社区（GitHub 90k+ stars）、配套的 LangSmith 可观测性平台和 LangGraph 多 Agent 编排框架，形成完整的 LLM 应用开发工具链。
4. **局限性**：抽象层带来一定的调试复杂度（出错时需要穿透抽象层定位）；版本迭代快（早期 API 不稳定）；对于简单应用，框架本身的复杂度反而增加了学习成本；在极致性能场景下，框架抽象层的额外开销不可忽视。

</details>

**23. 解释 LangChain 的关键组件**

<details>
<summary>💡 查看答案</summary>

LangChain 的架构由五个核心组件构成，理解它们之间的关系是掌握框架设计哲学的关键。

1. **Models（模型层）**：统一封装各类 LLM（OpenAI、Anthropic、本地模型）、Chat Model（对话格式接口）和 Embedding Model（向量化模型），通过统一接口屏蔽底层 API 差异，是整个框架的"算力底座"。
2. **Prompts（提示层）**：包括 PromptTemplate（变量化提示模板）、Example Selector（动态示例选择）、Output Parser（结构化输出解析），负责构建高质量、可复用的提示工程组件。
3. **Chains（链）**：将多个组件（Prompt、Model、Parser）组合成可执行的处理流水线，是 LangChain 早期的核心抽象，现代开发中逐渐被更灵活的 LCEL 取代。
4. **Memory（记忆）**：维护对话或任务的历史状态，从简单的全量历史存储（ConversationBufferMemory）到智能摘要（ConversationSummaryMemory），解决多轮交互中的上下文连续性问题。
5. **Agents & Tools（代理与工具）**：Agent 赋予应用动态决策能力（根据情况选择行动），Tools 是 Agent 可调用的外部能力集合（搜索、计算、API 调用），两者结合构成了 LangChain 处理复杂、开放性任务的核心机制。

这五个组件可以灵活组合——简单任务用 Prompt + Model 即可，复杂任务通过 LCEL 将 Memory、Tools、Agent 串联成完整的应用。

</details>

**24. LangChain 中的代理是什么？它们与链有何不同？**

<details>
<summary>💡 查看答案</summary>

代理（Agent）和链（Chain）是 LangChain 中两种不同的执行范式，核心区别在于决策的动态性。

1. **链（Chain）——固定流程**：链是预先定义好的步骤序列，每次执行都走相同的路径（A→B→C），步骤之间的连接是硬编码的，不会根据中间结果改变方向。优点是行为可预测、易于测试；缺点是无法处理需要动态判断的情况。典型例子：RAG 问答链（检索→格式化→LLM 生成→解析）。
2. **代理（Agent）——动态决策**：代理使用 LLM 作为"大脑"，在每个步骤实时决定"下一步做什么"——是调用某个工具、等待用户输入，还是直接给出最终答案。代理可以根据工具返回的中间结果改变执行路径，处理预先无法确定步骤数的任务。典型例子：能联网搜索的问答代理、能执行代码并修复错误的编程代理。
3. **选型原则**：任务步骤固定、逻辑可预测 → 用 Chain（更快、更稳定、成本低）；任务需要根据中间结果动态判断、使用多种工具 → 用 Agent（更灵活，但成本高、不确定性大）。实践中，Chain 常作为 Agent 的工具被调用（Agent 决定何时调用哪条 Chain）。
4. **混合使用**：许多生产系统采用"外部 Agent + 内部 Chain"的架构——Agent 负责高层规划和工具选择，具体的工具实现用确定性的 Chain，兼顾灵活性和可靠性。

</details>

**25. LangChain 中的链是什么，它们是如何工作的？**

<details>
<summary>💡 查看答案</summary>

Chain（链）是 LangChain 最基础的抽象概念，将多个处理步骤组合成一个可重复执行的处理流水线，前一步的输出自动成为后一步的输入。

**工作原理**：每个 Chain 实现统一的 `.invoke()/.run()` 接口，内部封装了"接收输入→处理→产生输出"的逻辑。多个 Chain 可以组合形成更复杂的 Chain，类似函数组合（Function Composition）。

**常见 Chain 类型**：
1. **LLMChain**：最基础的链，将 PromptTemplate 和 LLM 组合，接收变量字典，输出模型生成的文本。这是构建其他复杂链的基本单元。
2. **SequentialChain**：将多个 Chain 按顺序串联，前一个 Chain 的输出自动映射为下一个 Chain 的输入变量，适合"摘要→翻译→格式化"这类固定流程的任务。
3. **RouterChain**：根据输入内容动态路由到不同的处理分支——先用一个分类 LLMChain 判断查询类型，再根据分类结果路由到对应的专用 Chain（如"技术问题"路由到技术支持 Chain，"账单问题"路由到财务 Chain）。
4. **ConversationalChain**：内置 Memory 组件，自动维护对话历史，每次调用时将历史对话和当前输入一起传给模型，保持多轮对话的上下文连贯性。
5. **RetrievalQAChain**：结合 Retriever 和 LLM，是构建 RAG 问答系统最基础的 Chain 类型。

**现代实践**：LangChain 推荐使用 LCEL（`prompt | llm | parser`）替代传统 Chain 类继承体系，语法更简洁、原生支持流式和并行，但底层概念（"步骤串联，输出即输入"）与 Chain 是一致的。

</details>

**26. OpenAI 函数和 LangChain 函数有什么区别？**

<details>
<summary>💡 查看答案</summary>

在将函数与大型语言模型集成时，OpenAI 函数和 LangChain 函数提供了不同的方法。OpenAI 函数提供更手动的体验。你可以定义自定义函数，并对 AI 调用哪些函数以及如何使用它们拥有更大的控制权。然而，这需要更多的开发工作，因为你需要编写代码来解析 AI 的响应并管理参数。

另一方面，LangChain 函数优先考虑易用性。使用 LangChain 工具包可以帮助你避免从头开始编写函数定义和实现。然而，这也有局限性。你将受限于所选代理类型或工具包提供的函数。此外，告知模型可用函数依赖于在 LangChain 中选择合适的代理类型。

</details>

**27. Agent 框架（LangChain、AutoGen、CrewAI 等）的选型依据是什么？**

<details>
<summary>💡 查看答案</summary>

选型考量维度：\begin{enumerate}
- **任务类型**：简单顺序任务→LangChain；Multi-Agent 对话→AutoGen；角色分工明确的团队→CrewAI；\item  **生产成熟度**：LangChain 最成熟、社区最大；AutoGen 微软出品、企业支持好；\item  **定制化需求**：需要深度定制 Agent 行为时，轻量框架（如直接使用 LLM API）比重量框架更灵活；\item  **可观测性**：框架对 LangSmith/Langfuse 等追踪工具的支持程度；\item  **团队熟悉度**：团队已有的技术栈和学习成本；\item  **性能要求**：框架本身的抽象层是否引入不可接受的延迟；\item  **开发效率**：框架提供的内置组件能否减少重复开发量。建议：先用 LangChain 快速原型，性能敏感时考虑裸写框架。

</details>

**28. 请比较一下 LangChain 和 LlamaIndex，它们的核心应用场景有何不同？**

<details>
<summary>💡 查看答案</summary>

**LangChain**：一个通用的LLM应用"编排"框架，目标是将LLM与各种组件（工具、记忆、数据源）"链接"在一起，构建复杂的应用程序，更关注"工作流"的构建。核心抽象：Chains、Agents、Memory、Callbacks。最适合：构建复杂的多步骤Agent、需要多工具协作的自动化流程。

**LlamaIndex**：专注于外部数据的"数据"框架，解决LLM与私有或外部数据连接的核心问题（RAG）。更关注"数据流"的管理。核心抽象：Data Connectors、Indexes、Retrievers、Query Engines。最适合：构建高性能知识库问答系统、处理复杂非结构化数据。

实践中两者常结合使用：LlamaIndex构建强大的知识库检索工具，接入LangChain Agent中。

</details>

**29. 了解 A2A 框架吗？它和普通 Agent 框架的区别在哪？**

<details>
<summary>💡 查看答案</summary>

A2A（Agent-to-Agent）框架关注多个异构Agent之间的通信和协作，定义使由不同开发者、不同技术栈构建的Agent能够相互发现、理解和交互的通用标准与协议。

最关键的不同点：普通Agent框架关注的是"个体的实现"（Implementation of an individual），而A2A框架关注的是"群体的交互标准"（Interaction standard for a collective）。普通框架（如LangChain）在"应用层"，致力于构建能干活的个体；A2A框架在"协议层"，致力于构建所有个体互相交流的"社会规则"——类似AI领域的HTTP协议。


%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

</details>

