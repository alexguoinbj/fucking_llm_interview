# 2.6 链式思考（CoT）与推理增强 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 2 章：提示与上下文工程（Prompt & Context Engineering） · 共 18 题

**1. 链式思考（CoT）提示如何提高 LLM 的推理能力？**

<details>
<summary>💡 查看答案</summary>

链式思考（Chain-of-Thought，CoT）提示是一种通过引导大型语言模型（LLM）在输出最终答案之前，逐步生成中间推理步骤的技术。其核心思想是模拟人类解决复杂问题时的思维过程——将问题分解为一系列有序的子步骤，从而降低每一步的认知难度。

#### 基本原理

传统提示方式直接要求模型给出答案，而 CoT 提示则在示例中明确展示推理链条，引导模型在推断阶段同样生成类似的中间步骤。其典型形式如下：

\begin{lstlisting}[style=promptstyle]
	问题：小明有 5 个苹果，他给了小红 2 个，又买了 3 个，现在他有几个？
	让我们一步步思考：
	1. 小明最初有 5 个苹果。
	2. 他给了小红 2 个，剩下 5 - 2 = 3 个。
	3. 他又买了 3 个，共有 3 + 3 = 6 个。
	答案：小明现在有 6 个苹果。
\end{lstlisting}

#### 提升推理能力的机制

CoT 提示从以下几个维度提升 LLM 的推理能力：

- **分解复杂问题**：将多步骤问题拆解为若干子问题，使模型在每一步只需处理局部逻辑，降低出错概率。
- **激活隐式知识**：显式的推理链迫使模型调用并组织其预训练中习得的知识，而非仅依赖表面的模式匹配。
- **提高可解释性**：中间步骤使推理过程透明，便于人工检查和定位错误。
- **泛化至新任务**：通过少量示例（few-shot CoT）或零样本指令（zero-shot CoT，如"Let's think step by step"），模型可将推理模式迁移至未见过的任务类型。

#### 适用场景

CoT 在以下任务中效果尤为显著：数学文字题、逻辑推理、常识问答、多步骤规划以及代码调试等需要系统性逻辑推导的场景。研究表明，CoT 的收益在模型参数量足够大（通常超过 100B）时更为明显，小模型受益相对有限。

</details>

**2. 如果 CoT 提示失败了，你怎么改进 LLM 的推理能力？**

<details>
<summary>💡 查看答案</summary>

当初始的 CoT 提示未能达到预期效果时，可以从提示设计、技术手段和数据层面进行系统性改进。

#### 细化提示设计

- **消除歧义**：审查提示中是否存在模糊表达，明确问题的边界条件与预期输出格式。
- **补充缺失步骤**：若模型跳过了某些关键推理步骤，在 few-shot 示例中显式补全这些步骤。
- **增加示例多样性**：提供覆盖不同难度和结构的示例，帮助模型更好地理解推理模式的泛化方式。

#### 探索进阶提示技术

- **自洽性（Self-Consistency）**：对同一问题采样多条推理路径，通过多数投票选取最一致的答案，有效降低单次推理的随机误差。
- **思维树（Tree of Thoughts, ToT）**：将推理过程建模为树状结构，允许模型在多个候选分支中进行搜索与回溯，适合需要探索式推理的任务。
- **提供背景知识与假设**：在提示中注入相关领域知识或明确列出前提假设，减少模型因知识缺失导致的推理偏差。
- **鼓励多角度分析**：指示模型从不同视角或反例出发审视问题，增强推理的鲁棒性。
- **分步验证**：要求模型在给出最终答案后，对每个推理步骤进行自我检验，主动识别并纠正错误。

一个结合上述策略的改进提示示例：

\begin{lstlisting}[style=promptstyle]
	请解答以下问题。在作答时：
	1. 列出所有已知条件和假设。
	2. 将问题分解为若干子步骤，逐一求解。
	3. 完成后，回顾每个步骤，检查是否存在逻辑漏洞。
	4. 给出最终答案，并说明你的置信度。

	问题：[问题内容]
\end{lstlisting}

#### 引入人类反馈

通过人类对模型推理过程的逐步标注与反馈（如基于人类反馈的强化学习，RLHF），可以系统性地纠正模型的推理偏好，使其更符合人类的逻辑期望。

#### 改善训练数据

确保微调数据集中包含多样化、高质量的推理示例，覆盖目标任务的各类推理模式。必要时可构造专门针对失败案例的对比数据（正确推理 vs. 错误推理），通过对比学习强化模型对正确推理路径的偏好。

</details>

**3. 如何在 LLM 中使用停止序列？**

<details>
<summary>💡 查看答案</summary>

停止序列（Stop Sequence）是用户预先指定的一个或多个特定字符串，当 LLM 在生成过程中遇到这些字符串时，将立即停止文本生成。这一机制为模型输出提供了精确可控的终止边界。

#### 工作原理

在推理阶段，LLM 以自回归方式逐 token 生成文本。当生成的 token 序列与预设的停止序列完全匹配时，生成过程终止，停止序列本身通常不包含在最终输出中。大多数主流 API（如 OpenAI、Anthropic）均通过参数直接支持此功能：

\begin{lstlisting}[style=promptstyle]
	# 示例：使用 Anthropic Python SDK 设置停止序列
	import anthropic

	client = anthropic.Anthropic()

	response = client.messages.create(
	model="claude-sonnet-4-6",
	max_tokens=1024,
	stop_sequences=["Human:", "###", "\n\n"],
	messages=[
	{"role": "user", "content": "请列举三种水果，每行一个。"}
	]
	)

	print(response.content)
	print("停止原因:", response.stop_reason)  # 输出: stop_sequence 或 end_turn
\end{lstlisting}

#### 常见使用场景

- **多轮对话控制**：在对话系统中，使用 `"Human:"` 或 `"User:"` 作为停止序列，防止模型自行续写用户输入，确保对话轮次的正确切分。

    \begin{lstlisting}[style=promptstyle]
    stop_sequences=["Human:", "User:"]
    \end{lstlisting}

- **结构化输出截断**：当要求模型生成 JSON、代码块或列表时，使用特定定界符（如 \verb|"“`"| 或 `"</output>"`）标记输出结束，避免模型生成多余内容。

    \begin{lstlisting}[style=promptstyle]
    提示：请将以下内容总结为 JSON 格式，输出后写 </output>。
    stop_sequences=["</output>"]
    \end{lstlisting}

- **分步推理控制**：在需要逐步审核推理过程的场景中，使用 `"[STEP END]"` 等标记让模型每完成一步即暂停，由外部程序决定是否继续。

- **防止内容溢出**：在生成特定数量的列表项后停止，例如以空行作为停止序列来限制输出段落数。

#### 注意事项

- 停止序列应与正常输出内容有明显区别，避免误触发导致输出提前截断。
- 可同时设置多个停止序列，任意一个被匹配即触发停止。
- 停止序列与 `max_tokens` 参数相互独立，两者均可作为生成终止条件，实际输出以先触发者为准。
- 通过 API 返回的 `stop_reason` 字段可区分生成是由停止序列终止还是达到 token 上限终止，便于下游逻辑处理。

</details>

**4. 什么是 Chain-of-Thought（CoT）Prompting？它如何提升推理能力？**

<details>
<summary>💡 查看答案</summary>

CoT 是在 prompt 中提供包含逐步推理过程的示例（或直接加"Let's think step by step"），引导模型在给出答案前显式输出中间推理步骤。提升推理能力的原因：\begin{enumerate}
- **子问题分解**：将复杂问题拆解为更小的步骤，每步难度降低；\item  **工作记忆扩展**：中间推理 token 充当"工作记忆"，让模型可以在多步骤中传递信息；\item  **错误可见化**：错误的推理步骤在中间 token 中暴露，有助于后续步骤修正；\item  **测试时计算增加**：生成更多 token = 更多计算，对推理类任务有直接帮助（Test-Time Compute Scaling）。

</details>

**5. Zero-shot CoT 和 Few-shot CoT 有什么区别？各自适合什么场景？**

<details>
<summary>💡 查看答案</summary>

**Zero-shot CoT**：只添加"Let's think step by step"（或类似触发词），不提供带推理步骤的示例。适合：\begin{enumerate}
- 没有高质量标注推理步骤的示例；\item  快速尝试，成本低；\item  任务类型多样，难以为每种任务单独设计 CoT 示例。**Few-shot CoT**：提供 3–8 个包含完整推理链的示例（输入→推理步骤→答案）。适合：\item  有人工标注的推理步骤；\item  任务有特定的推理结构（如数学解题格式）；\item  效果要求高，愿意付出示例设计成本。效果对比：Few-shot CoT > Zero-shot CoT，但 Zero-shot CoT 使用更便捷，在大模型上效果也接近。

</details>

**6. CoT 对哪些类型的任务提升最明显？对哪些任务提升有限？**

<details>
<summary>💡 查看答案</summary>

**提升最明显的任务**：\begin{enumerate}
- 数学推理（算术、代数、几何）；\item  逻辑推理（命题逻辑、因果推断）；\item  常识推理（需要多步骤思考）；\item  代码生成和调试；\item  复杂规划（需要顺序步骤的任务）。**提升有限的任务**：
- 简单的事实性问答（单步查找即可回答）；\item  文本分类（模式匹配类，不需要推理链）；\item  翻译（通常不需要中间步骤）；\item  参数很小的模型（<7B）：CoT 不改变底层能力，只是激活能力，模型没有推理能力时 CoT 无法帮助；\item  纯记忆性任务。

</details>

**7. 什么是 Auto-CoT？它如何自动生成 CoT 示例？**

<details>
<summary>💡 查看答案</summary>

Auto-CoT（Zhang et al. 2022）：自动为 few-shot CoT 生成推理步骤，减少人工标注。工作流程：\begin{enumerate}
- 用 K-Means 将测试问题聚类，每个聚类选一个代表性问题；\item  对每个代表性问题，用 Zero-shot CoT（"Let's think step by step"）让模型自动生成推理步骤；\item  将生成的"问题+推理步骤+答案"三元组作为 few-shot 示例；\item  用这些自动生成的示例在测试集上做 few-shot CoT 推理。效果：在多数数学推理基准上与人工标注 CoT 效果相当，显著减少人工成本。局限：自动生成的推理步骤可能包含错误，影响整体效果。

</details>

**8. 什么是 Self-Consistency？它如何与 CoT 配合提升推理质量？**

<details>
<summary>💡 查看答案</summary>

Self-Consistency（Wang et al. 2022）：对同一问题用 CoT 采样多条不同的推理路径（使用高 Temperature），对最终答案进行多数投票，取出现频率最高的答案作为最终输出。与 CoT 配合的机制：\begin{enumerate}
- CoT 生成推理路径（中间步骤）；\item  Self-Consistency 解决了单条 CoT 可能走错路径的问题；\item  正确的推理路径在不同表达方式下趋向相同答案；\item  错误路径的答案更分散，被多数投票排除。效果：在 GSM8K 等数学推理基准上，Self-Consistency + CoT 比单次 CoT 提升 5–15%，在困难题目上提升更明显。代价：需要采样 20–40 次，推理成本增加 20–40 倍。

</details>

**9. Self-Consistency 的采样次数如何影响效果和成本？**

<details>
<summary>💡 查看答案</summary>

**采样次数与效果关系**：\begin{enumerate}
- 1次（单次 CoT）：baseline，无 Self-Consistency；\item  5次：显著提升，覆盖主要的正确推理路径；\item  20次：效果大幅提升，大多数任务在此收益明显；\item  40次：接近效果上限，边际收益递减；\item  >40次：几乎不再提升，成本持续增加。**成本-效果权衡**：
- 低成本场景（简单任务）：5次采样；\item  一般场景：10–20次；\item  高精度要求（数学竞赛题）：40次；\item  实时性要求高：不适合大量采样，考虑 Beam Search 等替代方案。实践建议：从 10次开始，根据效果-成本曲线确定最优采样数。

</details>

**10. 什么是 Tree-of-Thoughts（ToT）？它与 CoT 有什么区别？**

<details>
<summary>💡 查看答案</summary>

![CoT、ToT 与 GoT 推理拓扑结构对比图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter2/fig_cot_tot_got_comparison.png)




ToT（Yao et al. 2023）：将推理过程组织为树状结构，在每个决策点生成多个候选思维（thoughts），通过评估函数（LLM 自我评估）对候选思维打分，只保留有前途的路径继续扩展，支持回溯。与 CoT 的区别：\begin{enumerate}
- **结构**：CoT 是单条线性推理链；ToT 是多路径树状搜索；\item  **回溯**：CoT 一旦走错无法纠正；ToT 可以回溯到更好的分叉点；\item  **评估机制**：ToT 引入了对中间状态的显式评估；CoT 没有；\item  **适用任务**：ToT 适合需要探索多种可能性的规划和搜索问题（24点游戏、迷宫）；CoT 适合有明确推理步骤的任务；\item  **成本**：ToT 需要多次 LLM 调用，成本远高于 CoT。

</details>

**11. 什么是 Program-of-Thoughts（PoT）？它在什么场景下优于 CoT？**

<details>
<summary>💡 查看答案</summary>

PoT（Chen et al. 2022）：将推理步骤写成可执行代码（Python），而不是自然语言，通过实际执行代码获得答案。与 CoT 的区别：CoT 让 LLM 用自然语言"计算"，容易出现数值错误；PoT 让 LLM 生成代码，计算由 Python 精确执行。优于 CoT 的场景：\begin{enumerate}
- **数值计算**：加减乘除、统计、数学公式，CoT 容易算错，代码执行不出错；\item  **数据操作**：对列表、字典等数据结构的操作；\item  **逐步算法**：排序、搜索等需要精确步骤的算法；\item  **复杂数学**：微积分、矩阵运算等 LLM 难以精确推理的内容。局限：需要安全的代码执行环境；对自然语言推理（逻辑关系判断）没有优势。

</details>

**12. 什么是 Least-to-Most Prompting？它如何拆解复杂推理问题？**

<details>
<summary>💡 查看答案</summary>

Least-to-Most Prompting（Zhou et al. 2022）：将复杂问题先分解为一系列更简单的子问题，按从简单到复杂的顺序依次解答，每个子问题的答案作为解答下一个子问题的上下文。两个阶段：\begin{enumerate}
- **分解阶段**：提示 LLM "要解决这个问题，我需要先解决哪些子问题？"，生成子问题列表；\item  **逐步解答**：从最简单的子问题开始，每步将已有答案拼入 prompt，逐步解决更复杂的子问题。优势：特别适合需要"先学基础再解难题"的推理任务；相比标准 CoT，在组合泛化（compositional generalization）任务上提升明显。适合场景：数学应用题、编程任务分解、多步骤问答。

</details>

**13. 什么是 Analogical Reasoning Prompting？它的核心思路是什么？**

<details>
<summary>💡 查看答案</summary>

Analogical Reasoning Prompting（PS+，Wang et al. 2023）：让 LLM 在回答问题之前，先自主回忆类似问题的解题方法，再将这些类比方法应用于当前问题。核心思路：人类在面对新问题时，会自然联想到之前解决过的类似问题，借鉴已有经验。实现方式：\begin{enumerate}
- 在 prompt 中加入指令"回忆一个与当前问题类似的问题及其解法"；\item  LLM 生成一个类比示例及推理过程；\item  基于类比推理解决当前问题。优势：不需要手工设计 CoT 示例，LLM 自主生成相关示例；在数学推理任务上效果接近手工设计的 CoT，且更灵活。

</details>

**14. CoT 的推理步骤是否越长越好？有哪些研究发现？**

<details>
<summary>💡 查看答案</summary>

不是越长越好。研究发现：\begin{enumerate}
- **适当长度有益**：推理步骤太短（<3步）效果差，适当展开（5–10步）效果最好；\item  **过长反而有害**：步骤过多时，模型容易在中间引入错误，错误会传播到最终答案；\item  **"误导性CoT"现象**：如果推理步骤本身包含错误，这些错误会引导模型得出错误结论，比没有 CoT 更差；\item  **任务依赖**：复杂数学题需要更多步骤；简单推理题过多步骤是噪声；\item  **质量 > 长度**：精确、简洁的推理步骤优于冗长但模糊的步骤；\item  实践建议：推理步骤覆盖所有必要的逻辑跳跃即可，不需要填充冗余步骤。

</details>

**15. 如何评估 CoT 推理链的质量（不只看最终答案）？**

<details>
<summary>💡 查看答案</summary>

1. **步骤正确率**：对推理链的每个中间步骤单独打分（正确/错误），统计各步骤的错误分布；\item  **逻辑连贯性**：评估相邻步骤之间的逻辑关系是否有效（LLM-as-judge 打分）；\item  **推理链与答案一致性**：推理链支持的结论与最终答案是否一致（推理链对但答案错 → 答案提取问题）；\item  **错误传播分析**：第一个错误步骤出现在哪一步，后续步骤的准确率如何（分析错误传播）；\item  **忠实度（Faithfulness）**：最终答案是否真正来源于推理链，还是模型"抛开"推理链给出了与推理无关的答案（Turpin et al. 2023 发现 CoT 有时是事后合理化）；\item  工具：Process Reward Model（PRM），为每个推理步骤打分。

</details>

**16. CoT 能力为什么是一种"涌现能力"？它在小模型上为什么失效？**

<details>
<summary>💡 查看答案</summary>

CoT 作为涌现能力的原因：在模型规模超过约 100B 参数时，CoT 能力突然显现（小模型上 CoT 几乎没有帮助甚至有害），体现了能力的非线性涌现。小模型失效的原因：\begin{enumerate}
- **容量不足**：小模型同时维护多步推理的能力有限，中间步骤会"混淆"而非"帮助"；\item  **推理能力基础薄弱**：CoT 只是激活已有推理能力，小模型本身推理能力不足，CoT 无法创造没有的能力；\item  **指令跟随能力弱**：小模型可能无法理解"逐步推理"的指令本身；\item  **注意力不足**：维护长推理链需要模型对多个中间步骤保持精确注意力，小模型的注意力质量不足；\item  验证：同样的 CoT prompt 在 7B 模型上几乎没有提升，在 70B 上有明显提升，在 175B+ 上提升最显著。

</details>

**17. 什么是 Process Reward Model（PRM）？它如何改进 CoT 的质量？**

<details>
<summary>💡 查看答案</summary>

PRM（Lightman et al. 2023，OpenAI Let's Verify Step by Step）：对 CoT 推理链的每个中间步骤打分（而不是只对最终答案打分），用于评估和筛选高质量推理路径。改进 CoT 的方式：\begin{enumerate}
- **Best-of-N 采样**：对同一问题生成 N 条 CoT 推理链，PRM 对每条链的步骤质量打分，选择总分最高的链作为最终答案；\item  **Beam Search 引导**：在逐步生成推理步骤时，用 PRM 分数指导搜索方向，剪枝质量差的步骤；\item  **数据过滤**：用 PRM 对训练数据的推理链打分，只保留高质量推理链用于 Fine-tuning；\item  相比 Outcome Reward Model（只看最终答案对错），PRM 提供更精细的监督信号，特别有助于减少"侥幸正确"（推理错误但答案碰对）的问题。

</details>

**18. 什么是 Verification-guided CoT？它如何让模型自我验证推理步骤？**

<details>
<summary>💡 查看答案</summary>

Verification-guided CoT（或 Self-Verification CoT）：在生成 CoT 推理步骤后，让模型对自己的推理过程进行验证，发现错误后修正再输出最终答案。实现方式：\begin{enumerate}
- **Backward Verification**：生成答案后，让模型从答案反向推导回问题条件，验证一致性；\item  **Step-level Verification**：在每个推理步骤后插入"检查这一步是否正确"，发现错误时回退；\item  **Dual Prompt**：一个 prompt 生成答案，另一个 prompt 扮演"批评者"检查推理，两者对话直到达成共识；\item  **Self-Refine**：完成初稿后，让模型给自己的推理提供改进建议，再基于建议重写。效果：在数学和逻辑推理任务上，自我验证可减少约 10–20% 的错误，特别是简单计算错误。

</details>

