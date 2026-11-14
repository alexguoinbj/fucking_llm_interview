# 2.9 Prompt Caching 与上下文工程 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 2 章：提示与上下文工程（Prompt & Context Engineering） · 共 20 题

**1. 什么是 KV Cache？它在 LLM 推理中起什么作用？**

<details>
<summary>💡 查看答案</summary>

KV Cache（Key-Value Cache）是 Transformer 推理中的核心优化机制。理解其作用需要从自注意力机制的计算原理出发。

#### 自注意力中的重复计算问题

在 Transformer 的自注意力层中，每个 token 的表示由三个矩阵决定：Query（$Q$）、Key（$K$）和 Value（$V$）。注意力输出的计算公式为：


$$
	Attention(Q, K, V) = softmax\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
$$


在自回归生成过程中，模型每次只生成一个新 token，但为了计算该 token 的注意力权重，需要与**所有前序 token**的 $K$ 和 $V$ 矩阵做点积。若不做任何优化，第 $t$ 步生成时需要对前 $t-1$ 个 token 重新计算 $K$ 和 $V$，导致总计算量为 $O(n^2)$，随序列长度呈平方增长。

#### KV Cache 的工作原理

KV Cache 的核心思想是：**已生成 token 的 $K$ 和 $V$ 矩阵在后续步骤中不会改变**（因为这些 token 是固定的），因此可以将其缓存在显存中，后续步骤直接读取，无需重新计算。

具体流程为：
1. **Prefill 阶段**：模型一次性处理整个输入 prompt，计算所有 token 的 $K$、$V$ 矩阵并存入 KV Cache；
2. **Decode 阶段**：每生成一个新 token，仅计算该 token 的 $Q$、$K$、$V$，将新的 $K$、$V$ 追加到 Cache 中，并与历史 Cache 做注意力计算；
3. 如此循环，将每步的计算复杂度从 $O(n)$ 降低为接近 $O(1)$（仅新增一个 token 的计算）。

#### 实际收益

- **延迟降低**：Decode 阶段每步计算量大幅减少，token 生成速度（吞吐 token/s）显著提升；
- **长输入受益更大**：输入越长，无 Cache 时的重复计算量越大，Cache 带来的加速比越高；
- **代价**：KV Cache 占用大量 GPU 显存。以 LLaMA-3 70B 为例，处理 128k token 的 KV Cache 可达数十 GB，显存压力是限制长上下文推理并发量的核心瓶颈。

</details>

**2. 什么是 Prompt Caching？它和 KV Cache 有什么关系？**

<details>
<summary>💡 查看答案</summary>

Prompt Caching 是 API 服务商将 KV Cache 能力**跨请求暴露给用户**的功能。当 prompt 的前缀部分在多次请求中保持不变时，API 服务端缓存该前缀的 KV 状态，后续请求直接复用，跳过前缀的 Prefill 计算。

#### 与 KV Cache 的关系

两者的本质相同，区别在于作用范围：

- **KV Cache**：在**单次生成内**有效，Decode 阶段复用 Prefill 阶段的 KV 结果，请求结束后即释放；
- **Prompt Caching**：将 KV Cache 的生命周期**延伸到多次 API 调用之间**，服务端将特定前缀的 KV 状态持久化（通常 5 分钟 TTL），不同请求若共享相同前缀，均可复用同一份 KV Cache。

可以理解为：Prompt Caching = 跨请求持久化的 KV Cache，是 KV Cache 在服务端的工程化延伸。



![Prompt Caching 提示词缓存与首字延迟优化对比图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter2/fig_prompt_caching.png)




#### 触发机制

Prompt Caching 通常由用户在 API 请求中显式标记缓存断点（如 Anthropic 的 `cache_control` 字段），服务端在首次处理时将断点之前的 KV 状态写入缓存存储，后续请求检测到前缀一致时直接加载，跳过 Prefill 计算。

</details>

**3. Prompt Caching 能带来哪些具体收益？**

<details>
<summary>💡 查看答案</summary>

- **延迟降低**：命中缓存时，前缀 token 跳过 Prefill 计算，首 token 延迟（TTFT, Time To First Token）显著降低。对于 80k token 的长文档前缀，命中缓存可将 TTFT 从数十秒压缩至秒级以内；

- **成本降低**：Anthropic 等平台对缓存命中的 token 收费约为原价的 10%，即节省约 90% 的前缀计算费用。以每次请求包含 10k token 固定前缀、日均 10,000 次请求为例，月度 token 费用可节省数千美元；

- **吞吐量提升**：省下的 GPU 计算资源可用于服务更多并发请求，在相同硬件预算下提升系统整体吞吐量。

具体收益取决于两个核心变量：**前缀长度**（越长单次节省越多）和**缓存命中率**（越高总体收益越大）。两者均高时，Prompt Caching 的 ROI 极为显著。

</details>

**4. 哪类内容最适合放在可缓存的前缀中？**

<details>
<summary>💡 查看答案</summary>

适合缓存的内容具备以下特征：**静态、固定、被多个请求共享**。典型内容包括：

- **系统 Prompt**：角色定义、行为准则、输出格式要求等在整个产品生命周期内几乎不变的内容；
- **工具/函数定义**：Agent 系统中的 tool schema，通常包含数十个工具的 JSON 描述，token 量大且固定；
- **长文档**：知识库文档、代码库、法律合同、技术手册等作为背景知识注入的静态文本；
- **Few-shot 示例集合**：ICL 示例池中的固定示例，尤其是包含长推理链的 CoT 示例。

**不适合缓存的内容**：用户 ID、当前时间戳、实时账户状态、个性化变量等动态内容——将此类内容置于前缀会导致每次请求前缀不同，缓存持续失效，不仅无法获得收益，还会额外产生缓存写入开销。

</details>

**5. Prompt Caching 在上下文工程中有什么设计原则？**

<details>
<summary>💡 查看答案</summary>

核心原则：**稳定内容前置，动态内容后置**。

\begin{lstlisting}[style=promptstyle]
	[可缓存前缀]
	├── System Prompt（角色、规则）         ← 最稳定，命中率最高
	├── 工具定义（Tool Schema）
	├── 背景文档（知识库、代码库）
	└── Few-shot 示例

	[动态部分，不缓存]
	├── 对话历史（每轮新增）
	└── 用户当前消息
\end{lstlisting}

只要前缀部分不发生任何 token 级别的变化，缓存就能命中，节省前缀部分的全部 Prefill 计算开销。

此外，还需遵循以下辅助原则：
- **断点最小化**：缓存断点数量不宜过多（通常 1--2 个），过多断点增加管理复杂度；
- **断点位置选择最长稳定前缀末尾**：断点越靠后，单次命中节省的 token 越多；
- **避免前缀中的隐式动态内容**：如当前日期、随机种子等即使看似固定也可能随部署环境变化的内容。

</details>

**6. 如何在 Anthropic API 中启用 Prompt Caching？**

<details>
<summary>💡 查看答案</summary>

通过在 content block 中添加 `cache_control` 字段标记缓存断点：

\begin{lstlisting}[language=Python, style=promptstyle]
	messages = [
	{
		"role": "user",
		"content": [
		{
			"type": "text",
			"text": "这里是长文档内容（80k tokens）...",
			"cache_control": {"type": "ephemeral"}  # 缓存断点
		},
		{
			"type": "text",
			"text": "用户的具体问题"                 # 不缓存
		}
		]
	}
	]
\end{lstlisting}

`cache_control` 标记的位置即为缓存断点，该位置**之前**的所有内容会被缓存。API 返回的 `usage` 字段中包含以下字段用于监控缓存状态：
- `cache_creation_input_tokens`：本次请求新写入缓存的 token 数（首次，全额计费）；
- `cache_read_input_tokens`：本次请求从缓存读取的 token 数（命中，约 10% 计费）。

</details>

**7. Prompt Caching 有哪些触发条件和限制？**

<details>
<summary>💡 查看答案</summary>

**触发条件（以 Anthropic 为例）：**
- 缓存前缀长度 $\geq$ 1024 tokens（claude-haiku 为 2048 tokens），低于此阈值不触发缓存；
- 请求间隔在缓存 TTL 内（约 5 分钟），超时后缓存自动失效；
- 前缀内容与缓存时**完全一致**（任何单个 token 的变化都会导致缓存失效，包括空格、标点、编码差异）。

**主要限制：**
- **首次写入全额计费**：缓存建立时（`cache_creation`）仍按正常价格计费，节省从第二次命中开始；
- **缓存预热问题**：高并发场景下，多个请求可能同时触发缓存写入（缓存尚未建立），导致并发冷启动；
- **安全隔离**：不同 API Key 之间的缓存严格隔离，不同用户/租户的请求无法共享缓存（即使前缀完全相同）；
- **TTL 较短**：5 分钟的 TTL 对低频请求场景（如每小时一次的批处理任务）几乎无效。

</details>

**8. Prompt Caching 和 RAG 如何配合使用？**

<details>
<summary>💡 查看答案</summary>

两者并非互斥，而是互补关系。根据知识库规模和访问模式，有两种主要配合模式：

**模式一：文档预加载 + 缓存（Cache-First）**

将整个知识文档注入 prompt 前缀并标记缓存断点，适合文档数量少（单文档 $\leq$ 200k token）且被高频查询的场景（如产品说明书、API 文档）。用户每次提出不同问题，前缀不变，缓存持续命中，效果最佳。

**模式二：RAG 检索 + 缓存（RAG-First）**

动态检索最相关的文档片段注入 prompt，适合大型知识库（无法全量注入）。优化点在于：若同一文档片段被多次检索命中，也可对该片段标记缓存断点，在片段级别复用 KV Cache。效果弱于模式一，但适用范围更广。

**选择依据：**知识库文档数量少且固定用模式一；文档量大、动态更新频繁用模式二；两种模式可结合使用（高频文档预加载，低频文档 RAG 检索）。

</details>

**9. 缓存 TTL 过短会带来什么问题？如何应对？**

<details>
<summary>💡 查看答案</summary>

TTL 过短（如 5 分钟）导致低频请求场景下缓存持续冷启动，每次请求均需重新计算前缀，完全无法享受缓存收益，且缓存写入本身还会带来轻微的额外开销。

**应对策略：**
- **主动预热（Cache Warming）**：部署定时任务（如每 4 分钟一次），向 API 发送包含缓存前缀的最小化请求，维持缓存活跃状态，适合可预期的低频场景；
- **请求批处理**：将散落的低频请求在时间上聚合，集中在短时间窗口内批量发送，提高请求密度从而提升缓存命中率；
- **架构调整**：对缓存收益不明显的场景（极低频、短前缀），不依赖 Prompt Caching，转而采用模型量化、KV Cache 压缩等其他推理优化手段；
- **平台选择**：部分平台支持更长的缓存 TTL 或手动控制缓存生命周期，可根据业务需求选择更匹配的服务商。

</details>

**10. 如何衡量 Prompt Caching 的实际收益？**

<details>
<summary>💡 查看答案</summary>

**核心监控指标：**
- **缓存命中率**：$命中率 = \dfrac{cache_read_input_tokens 非零的请求数}{总请求数}$，反映缓存策略的有效性；

- **Token 节省量**：$节省 = \sum cache_read_input_tokens \times (P_{原价} - P_{缓存价})$，直接量化成本收益；

- **延迟改善**：对比缓存命中与未命中请求的 TTFT 分布（P50/P95），量化体验提升；

- **成本节省率**：$节省率 = \dfrac{无缓存总成本 - 有缓存总成本}{无缓存总成本}$，用于 ROI 汇报。

数据来源为 API 返回的 `usage` 字段，建议将 `cache_read_input_tokens` 和 `cache_creation_input_tokens` 接入监控系统，按小时/天聚合分析，及时发现缓存命中率异常下降（通常意味着前缀被意外修改）。

</details>

**11. 多租户系统中，如何安全地共享 Prompt Caching？**

<details>
<summary>💡 查看答案</summary>

多租户场景下，缓存共享需要在**效率**（尽可能共享）与**安全隔离**（防止数据泄露）之间精细权衡。推荐采用**分层缓存架构**：

- **第一层——全局公共层**（所有租户共享）：通用 System Prompt、平台级工具定义等不含任何租户信息的内容，可在同一 API Key 下的所有请求间安全共享缓存；

- **第二层——租户专属层**（按租户隔离）：租户自定义知识库、租户级别的行为规则等含有租户专属信息的内容，通过为每个租户分配独立 API Key 实现缓存隔离（API 平台通常按 API Key 划分缓存命名空间）；

- **第三层——用户动态层**（不缓存）：用户消息、会话变量、实时状态等每次请求唯一的内容，不参与缓存。

此设计使第一层缓存由所有租户共享，命中率最高；第二层缓存按租户独立维护，安全性有保障；第三层完全绕过缓存，无安全风险。

</details>

**12. Prompt Caching 如何影响 Prompt 的版本管理策略？**

<details>
<summary>💡 查看答案</summary>

Prompt Caching 使前缀稳定性成为一个具有直接经济成本的工程约束，从而深刻影响 Prompt 的版本管理方式：

- **版本变更即缓存失效**：修改 System Prompt 的任何一个字符（包括空格、标点）都会使所有已建立的缓存立即失效，产生一次全量冷启动成本，在高流量系统中可能意味着数百次请求的额外费用；

- **灰度发布 + 缓存预热**：新版 Prompt 上线前，应先用少量流量（如 1%）触发缓存建立，待缓存预热完成后再逐步切流，避免版本切换瞬间的大规模冷启动；

- **批量积累修改**：高频迭代 Prompt 会持续触发缓存失效，应将多个小修改积累后集中发布，而非每次优化立即上线；

- **分层冻结策略**：将 Prompt 拆分为"稳定核心层"（缓存，谨慎修改）和"可迭代扩展层"（不缓存，自由修改），使两者的迭代节奏相互解耦；

- **A/B 测试成本**：两个版本的 Prompt 同时运行时，各自的缓存需独立预热，实际成本高于单版本，需在测试收益与额外成本间权衡。

</details>

**13. 在 Agent 长对话中，如何利用 Prompt Caching 优化多轮推理成本？**

<details>
<summary>💡 查看答案</summary>

Agent 多轮推理中，每轮请求都包含大量重复内容（System Prompt、工具定义、历史对话累积），是 Prompt Caching 收益最显著的场景之一。

**推荐的上下文结构：**

\begin{lstlisting}[style=promptstyle]
	Round 1: [System + Tools（缓存断点）| UserMsg1]
	→ 写入缓存

	Round 2: [System + Tools（命中缓存）| History1 | UserMsg2]
	→ System+Tools 命中，仅 History1+UserMsg2 新增计算

	Round 3: [System + Tools（命中缓存）| History1+2 | UserMsg3]
	→ System+Tools 持续命中，累积节省随轮次增长
\end{lstlisting}

**关键设计要点：**
- 将 System Prompt 和工具定义固定为不变前缀，并在其末尾设置缓存断点；
- 对话历史追加在缓存断点之后，不参与缓存（因每轮都在增长）；
- 若工具定义较长（如 50+ 个工具），节省效果尤为显著——工具定义 token 数可能达到数千，每轮均可节省这部分计算；
- 对于超长对话，可对历史对话中稳定的早期部分（如前 $N$ 轮的 Assistant 回复）也设置额外缓存断点，进一步扩大命中范围。

</details>

**14. Prompt Caching 对模型输出质量有影响吗？**

<details>
<summary>💡 查看答案</summary>

理论上**无影响**。Prompt Caching 的本质是跳过前缀 token 的重复 Prefill 计算，直接加载已计算好的 KV 状态，模型"看到"的上下文语义与无缓存时完全相同，输出分布理论上应保持一致。

**潜在风险（极少见）：**
- **平台实现缺陷**：若服务端缓存存储存在 bug，损坏的 KV 状态可能导致异常输出（实践中极为罕见，主流平台有完善的校验机制）；
- **模型热更新冲突**：若平台在缓存 TTL 期间对模型权重进行在线更新，旧版本 KV Cache 与新版本模型权重之间可能存在不兼容（正规平台通常在模型更新时主动清空所有缓存以避免此问题）。

**工程验证建议：**在启用 Prompt Caching 后，定期对同一输入进行"缓存命中"与"强制冷启动"的对比测试，统计输出一致性，若发现系统性差异应及时上报平台排查。

</details>

**15. 如何设计一个 Prompt 模板系统，最大化 Prompt Caching 收益？**

<details>
<summary>💡 查看答案</summary>

**分层模板架构：**

1. **基础层（全局共享，最高命中率）**：跨所有业务场景通用的内容，如平台级角色定义、安全准则、通用输出格式。此层变更频率最低，应设置第一个缓存断点；

2. **场景层（按业务类型，中等命中率）**：特定业务场景的工具定义、领域知识、场景专属示例。此层在同一业务线内固定，设置第二个缓存断点；

3. **请求层（动态，不缓存）**：用户消息、个性化变量、实时上下文，每次请求均不同，放在所有缓存断点之后。

\begin{lstlisting}[style=promptstyle]
	[Layer 1: 全局 System Prompt]  ← cache_control 断点1（最高命中率）
	[Layer 2: 业务场景工具定义]    ← cache_control 断点2
	[Layer 3: 用户历史 + 当前消息] ← 不缓存，动态追加
\end{lstlisting}

**工程实现要点：**
- 使用模板渲染引擎（如 Jinja2）确保静态层的 token 化结果完全一致（注意空白字符规范化）；
- 为每个模板版本维护"token 指纹"，检测到指纹变化时主动触发缓存预热流程；
- 将动态变量**严格限制**在缓存断点之后，禁止任何动态内容渗透到可缓存层。

</details>

**16. Prompt Caching 和 Batch API 有什么区别和联系？**

<details>
<summary>💡 查看答案</summary>

\begin{table}[h]

	\caption{Prompt Caching 与 Batch API 对比}


| **维度** | **Prompt Caching** | **Batch API** |
| --- | --- | --- |
| 优化目标 | 减少重复计算（延迟 + 成本） | 批量处理降低成本（吞吐量） |
| 延迟表现 | 命中时延迟大幅降低 | 延迟更高（异步处理，分钟级） |
| 适用场景 | 实时交互，前缀重复率高 | 离线批处理，对延迟不敏感 |
| 成本节省 | 缓存 token 约 90% 折扣 | 批量约 50% 折扣 |
| 触发方式 | 用户标记缓存断点 | 提交批处理任务队列 |


\end{table}

**两者可叠加使用：**批量请求同时启用 Prompt Caching，在批处理任务中复用公共前缀的 KV Cache，可同时享受两种优惠，总体成本折扣可达原价的 5% 左右（$10% \times 50%$）。适合大规模离线数据处理场景，如对数万条文档做同一 System Prompt 下的批量分析。

</details>

**17. 如何在动态个性化场景中仍然实现 Prompt Caching？**

<details>
<summary>💡 查看答案</summary>

动态个性化内容（如用户姓名、账户余额、实时状态）若注入前缀会使缓存每次失效，需通过架构设计将个性化与缓存解耦。

**方案一：个性化内容严格后置**

将所有个性化变量从前缀中剥离，移至缓存断点之后：

\begin{lstlisting}[style=promptstyle]
	[可缓存] 你是一个智能客服助手，职责是帮助用户解决账户问题...（通用规则）
	← cache_control 断点
	[不缓存] 当前用户：{user_name}，账户状态：{user_state}
	用户问题：{query}
\end{lstlisting}

**方案二：工具调用动态获取个性化信息**

将个性化信息的获取从"注入前缀"改为"模型主动调用工具查询"，前缀只声明工具的存在而非其返回值：

\begin{lstlisting}[style=promptstyle]
	[可缓存] 你有 get_user_info(user_id) 工具，
	需要获取用户信息时请调用它。
	← cache_control 断点
	[不缓存] 用户 ID：{user_id}
	用户问题：{query}
\end{lstlisting}

模型在推理时主动调用工具获取个性化数据，前缀始终保持稳定，缓存持续命中。此方案额外带来的好处是信息获取时机更精准（按需查询），且便于对工具调用结果做权限控制和审计。

</details>

**18. 如何量化评估"是否值得为某个场景启用 Prompt Caching"？**

<details>
<summary>💡 查看答案</summary>

**ROI 决策框架：**


$$
	单次节省 &= N_{prefix} \times (P_{原价} - P_{缓存价}) \\
	写入成本 &= N_{prefix} \times P_{原价} \times (1 + \delta) \\
	盈亏平衡命中次数 &= \left\lceil \frac{写入成本}{单次节省} \right\rceil \approx \frac{1 + \delta}{1 - P_{缓存价} / P_{原价}} \approx 1.1  次
$$


其中 $N_{prefix}$ 为前缀 token 数，$\delta$ 为缓存写入附加费率（Anthropic 约为 25%），$P_{缓存价} / P_{原价} \approx 0.1$。

实践中，**约 2 次命中即可回本**，因此只要同一前缀在 TTL 内被调用超过 2 次，启用缓存即有净收益。

**值得启用的判断条件（满足任意两条即建议启用）：**
- 前缀 token 数 $\geq$ 1024（平台最低要求）；
- 同一前缀在 TTL（5 分钟）内预期命中次数 $\geq$ 3；
- 前缀内容稳定性高（近期无频繁修改计划）；
- 系统对 TTFT 延迟有明确要求（实时交互场景）。

</details>

**19. 当 Prompt Caching 和 Prompt 优化迭代存在冲突时，如何取舍？**

<details>
<summary>💡 查看答案</summary>

冲突本质：Prompt 优化需要频繁修改前缀，而 Prompt Caching 的收益依赖前缀稳定不变——两者对前缀的诉求在方向上正好相反。

**按产品阶段的取舍策略：**
- **开发探索阶段**：完全不依赖 Prompt Caching，自由迭代 Prompt，以效果优化为第一优先级，缓存命中率低是预期结果而非问题；

- **效果稳定阶段**：冻结核心 System Prompt，引入缓存，将迭代重心转移至不参与缓存的动态层（如 Few-shot 示例的动态选择策略）；

- **生产成熟阶段**：实行"稳定核心层 + 可迭代扩展层"双轨制，核心层谨慎修改并走灰度预热流程，扩展层自由迭代不受缓存约束；

- **紧急修复场景**：若需要立即修改核心前缀（如安全漏洞、重大错误），接受一次性冷启动成本，修复后尽快重新预热缓存，必要时提前通知下游系统预期延迟上升。

</details>

**20. 在超长上下文（如 100k+ token）场景中，Prompt Caching 的最佳实践是什么？**

<details>
<summary>💡 查看答案</summary>

**场景示例：**将整本技术手册（80k tokens）注入 prompt，用户对其反复提问。

**最佳实践：**
1. **单一缓存断点**：在文档末尾设置一个缓存断点，将整个文档作为一个整体缓存单元，避免多断点带来的管理复杂度；

2. **确保文档 token 化一致性**：文档内容经 tokenizer 处理后必须每次完全相同。注意：文件编码（UTF-8/GBK）、行尾符（`\textbackslash n` vs `\textbackslash r\textbackslash n`）、首尾空白字符等细节均可能导致 token 序列差异，建议将标准化后的文档 token 化结果固化存储；

3. **用户问题严格后置**：所有用户问题、对话历史均放在缓存断点之后，绝不在文档内容中插入任何动态变量；

4. **服务启动时主动预热**：在服务实例启动时发送一个"预热请求"建立缓存，确保首批真实用户请求均能命中缓存，避免冷启动影响用户体验；

5. **监控缓存有效性**：持续监控 `cache_read_input_tokens`，其值应接近文档 token 数（$\approx$ 80k）。若该值持续偏低，通常意味着文档内容或前缀结构发生了意外变动，需立即排查。

**成本测算（80k token 文档，1000 次用户查询）：**

\begin{table}[h]

	\caption{Prompt Caching 成本对比（80k token 文档）}


| **方案** | **等效计费 Token 数** | **相对成本** |
| --- | --- | --- |
| 无缓存（每次全量计算） | $80k \times 1000 = 8000万$ | 100% |
| 有缓存（首次写入 + 999 次命中） | $80k + 80k \times 10% \times 999 \approx 880万$ | $\approx 11%$ |
| **节省** | $\approx 7120万 token$ | **$\approx$ 89%** |


\end{table}

在此场景下，Prompt Caching 是目前性价比最高的单点优化手段，几乎无任何架构改造成本即可实现接近 90% 的成本节省。

</details>

