# 5.4 重要性采样与 KL 散度 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 5 章：强化学习与后训练对齐（RLHF / DPO） · 共 9 题

**1. 重要性采样的基本原理是什么？**

<details>
<summary>💡 查看答案</summary>

重要性采样（Importance Sampling, IS）是一种解决"从分布 $p$ 计算期望，但只能从分布 $q$ 采样"问题的统计技术，是 PPO 算法的数学基础。

**核心思想**：利用期望的定义，通过引入比值权重 $w(x) = p(x)/q(x)$，将原本需要从 $p$ 采样的期望转化为在 $q$ 上的加权期望：
\[\mathbb{E}_{x \sim p}[f(x)] = \mathbb{E}_{x \sim q}\left[\frac{p(x)}{q(x)} f(x)\right] = \mathbb{E}_{x \sim q}[w(x) \cdot f(x)]\]
其中 $w(x) = p(x)/q(x)$ 称为**重要性权重（Importance Weight）**，用于"校正"$q$ 和 $p$ 之间的分布差异。

**直观理解**：若 $q$ 在某区域的概率比 $p$ 更高，则从 $q$ 采样时该区域被过度采样，需要用小权重 $w < 1$ 向下校正；若 $q$ 在某区域概率比 $p$ 更低，则被欠采样，需要用大权重 $w > 1$ 向上校正。

**应用前提**：$q(x) > 0$ 的地方 $p(x)$ 也必须 $> 0$（支撑集包含条件），否则某些 $p$ 下高概率的区域在 $q$ 下从未被采样到，期望估计将出现严重误差。

**在 PPO 中的应用**：允许用旧策略 $\pi_{\theta_old}$ 采集的数据来更新新策略 $\pi_\theta$，重要性权重为 $\pi_\theta(a|s)/\pi_{\theta_old}(a|s)$。PPO 的 clip 机制正是为了约束此权重不偏离 1 太远，从而控制方差。


![重要性采样修正概率分布偏差原理](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter5/fig_importance_sampling.png)

</details>

**2. 重要性采样的作用是什么？**

<details>
<summary>💡 查看答案</summary>

在强化学习和 LLM RL 中，重要性采样解决了一个核心工程问题：**如何复用旧策略生成的数据来更新新策略**，使得 on-policy 方法在保证理论正确性的同时提高样本利用效率。

**1. 在 PPO 中的核心作用**

PPO 在每轮 rollout 后，会对同一批数据进行**多个 epoch 的小批量更新**（通常 2-4 epoch）。在第一次更新后，策略参数 $\theta$ 已经改变（$\theta \neq \theta_old$），但数据仍然是由旧策略 $\pi_{\theta_old}$ 生成的。重要性权重 $\frac{\pi_\theta(a|s)}{\pi_{\theta_old}(a|s)}$ 用于修正这种分布差异，使得"用旧数据更新新策略"在数学上是合法的：
\[\mathcal{L}^{IS} = \mathbb{E}_{(s,a) \sim \pi_{\theta_old}}\left[\frac{\pi_\theta(a|s)}{\pi_{\theta_old}(a|s)} \cdot A(s,a)\right]\]

**2. 提升样本效率**

没有重要性采样的纯 on-policy 方法（如 REINFORCE）每次更新后必须丢弃所有数据重新采样，极其浪费。重要性采样使得每批 rollout 数据可以被多次使用，显著降低了在线采样（即昂贵的 LLM 自回归生成）的频率。

**3. PPO 的 Clip 机制**

由于重要性权重方差与 $p$/$q$ 的差异正相关，PPO 通过裁剪权重：$clip(\pi_\theta/\pi_{\theta_old}, 1-\varepsilon, 1+\varepsilon)$，限制权重不超出安全范围，在保证复用旧数据的同时控制方差，防止策略跳跃式更新。

**4. 在 GRPO 中的体现**

GRPO 同样使用重要性采样比（ratio）来计算目标函数，保留了 PPO 的 clip 机制，确保多步更新时策略偏移不超出安全范围。

</details>

**3. 解释一下 PPO 算法重要性采样的原理**

<details>
<summary>💡 查看答案</summary>

PPO 算法中重要性采样的核心原理是利用旧策略的数据来估计新策略的期望回报，从而避免每次策略更新后都需要重新采样数据。


$$
	\mathbb{E}_{a \sim \pi_\theta} \left[ f(a) \right] = \mathbb{E}_{a \sim \pi_{\theta_{old}}} \left[ \frac{\pi_\theta (a|s)}{\pi_{\theta_{old}}(a|s)} f(a) \right]
$$


其中 $\frac{\pi_\theta}{\pi_{\theta_{old}}}$ 是重要性权重，用于修正动作分布差异。

</details>

**4. 为什么计算重要性参数用 log 相减后再 exp 而不是直接除？**

<details>
<summary>💡 查看答案</summary>

1. **数值稳定性**：概率值可能非常小，直接计算会导致数值下溢或溢出。
2. **计算效率**：log_softmax 是原生操作，比先计算概率再取对数更高效。
3. **梯度计算**：对数概率的梯度计算更数值稳定。
4. **数学一致性**：概率的乘法在对数空间变为加法。

</details>

**5. 离散 KL 散度的推导过程**

<details>
<summary>💡 查看答案</summary>

**1. KL 散度的定义**

KL 散度（Kullback-Leibler Divergence）的定义为：

$$
	KL[q, p] = \mathbb{E}_{x \sim q} \left[ \log \frac{q(x)}{p(x)} \right]
$$


**2. k1 估计器**

k1 是 KL 散度的朴素蒙特卡洛估计器，直接基于 KL 散度的定义：

$$
	k1 = \frac{1}{N} \sum_{i=1}^N \log \frac{q(x_i)}{p(x_i)}
$$


性质：无偏性，但高方差。

**3. k2 估计器**

k2 是基于二阶近似的估计器，利用了 KL 散度在 $p \approx q$ 时的局部性质：

$$
	k2 = \frac{1}{2} \left( \log \frac{p(x)}{q(x)} \right)^2
$$


性质：低方差，但有偏性。

**4. k3 估计器**

k3 结合了 k1 和控制变量（control variate）的思想：

$$
	k3 = (r - 1) - \log r, \quad r = \frac{p(x)}{q(x)}
$$


性质：无偏性，低方差，稳定性好。

</details>

**6. 近似 KL 散度的计算**

<details>
<summary>💡 查看答案</summary>

KL 散度的完整形式可能没有解析解，计算复杂度高。近似方法需要考虑偏差和方差。

**前向 KL 散度近似**：

$$
	Forward KL = \mathbb{E}_{x \sim q} \left[ \log \frac{q(x)}{p(x)} \right] \approx -\log x
$$


**反向 KL 散度近似**：

$$
	Reverse KL = \mathbb{E}_{x \sim p} \left[ \log \frac{p(x)}{q(x)} \right] \approx x \log x
$$


前向 KL 对小概率事件极其敏感（mode-covering），反向 KL 对大概率更敏感（mode-seeking）。在大语言模型和生成任务中，反向 KL 通常更受青睐。

</details>

**7. PPO 等 RL 算法中怎么算 KL 散度的？**

<details>
<summary>💡 查看答案</summary>

在 PPO、GRPO 等算法中，KL 散度的计算方式如下：

**PPO**：通常使用 $KL(\pi_\theta | \pi_{ref})$，即反向 KL。

**GRPO**：同样使用 $KL(\pi_\theta | \pi_{ref})$。

令 $x = \frac{\pi_\theta(a|s)}{\pi_{old}(a|s)}$，则：
- Forward KL: $-\log x$
- Reverse KL: $x \log x$

正向 KL 倾向于使模型分布覆盖目标分布的所有支持点（mode-covering），反向 KL 倾向于使模型分布在目标分布的高概率区域（mode-seeking）。

</details>

**8. 具体怎么计算 KL 散度，KL 散度和交叉熵有什么关系？**

<details>
<summary>💡 查看答案</summary>

**KL 散度的计算**：

对于两个概率分布 P 和 Q（定义在同一空间上），KL 散度（Kullback-Leibler Divergence）定义为：

\[
KL(P||Q) = \Sigma_{x} P(x) · log(P(x)/Q(x)) = E_{x~P}[log P(x) - log Q(x)]
\]

（连续分布则用积分代替求和）。

**在 LLM 场景中的计算**：对于策略 $\pi_{\theta}$ 和参考策略 $\pi_{ref}$，在给定上下文 s 下，对词表中所有可能的下一个 token：

\[
KL(\pi_{\theta}(·|s)||\pi_{ref}(·|s)) = \Sigma_{token v} \pi_{\theta}(v|s) · log(\pi_{\theta}(v|s)/\pi_{ref}(v|s))
\]

由于词表很大，直接对所有 token 求和计算成本较高，实践中常用**蒙特卡洛采样估计**：对于实际采样得到的 token $y_t$，用 log $\pi_{\theta}$($y_t$|$s_t$) - log $\pi_{ref}$($y_t$|$s_t$) 作为该位置 KL 散度的（有偏但简单的）单样本估计；为了得到方差更低、(近似)无偏的估计，常用如前述的 **k3 估计器**：

\[
KL_estimate = \pi_{ref}(y_t|s_t)/\pi_{\theta}(y_t|s_t) - log(\pi_{ref}(y_t|s_t)/\pi_{\theta}(y_t|s_t)) - 1
\]

这个表达式利用了 $e^x$ - x - 1 ≥ 0（当 x=log($\pi_{ref}$/$\pi_{\theta}$)时）的性质，保证估计值恒为非负（与KL散度本身非负的性质一致），且方差更低。

**KL 散度与交叉熵的关系**：

交叉熵（Cross-Entropy）定义为：

\[
H(P,Q) = -\Sigma_{x} P(x)·log Q(x) = E_{x~P}[-log Q(x)]
\]

而 P 的熵（Entropy）为：

\[
H(P) = -\Sigma_{x} P(x)·log P(x) = E_{x~P}[-log P(x)]
\]

KL 散度可以表示为交叉熵与熵的差：

\[
KL(P||Q) = \Sigma_{x} P(x)·log(P(x)/Q(x)) = \Sigma_{x} P(x)·log P(x) - \Sigma_{x} P(x)·log Q(x) = -H(P) + H(P,Q) = H(P,Q) - H(P)
\]

即：**KL(P||Q) = 交叉熵 H(P,Q) - 熵 H(P)**

**直观理解**：
- 交叉熵 H(P,Q) 表示"用分布 Q 来编码来自分布 P 的数据所需的平均编码长度（比特数，若以2为底）"；
- 熵 H(P) 是"用分布 P 自身（最优编码）来编码来自 P 的数据所需的平均编码长度"，是理论最小值；
- **KL 散度即为"用 Q 编码而非用 P 编码所产生的'额外'编码长度"**，衡量了 Q 与真实分布 P 之间的"差距"。

**在机器学习训练中的体现**：在分类/语言建模任务中，常见的交叉熵损失 -$\sum$ $y_true$·log $y_pred$，如果将 $y_true$ 视为真实分布 P（通常是one-hot分布，熵H(P)=0），$y_pred$ 视为模型预测分布 Q，那么此时**最小化交叉熵等价于最小化 KL(P||Q)**（因为 H(P)=0 是常数，不影响优化），这就是为什么交叉熵损失和KL散度在分类问题的优化中是"等价"的——但在一般情况下（P不是退化的one-hot分布时），两者相差一个常数项 H(P)。

**总结**：KL散度计算公式为 KL(P||Q)=$E_{x~P}$[log P(x)-log Q(x)]，在 LLM-RL 中常用 k3 估计器进行低方差估计；KL散度与交叉熵的关系是 **KL(P||Q) = H(P,Q) - H(P)**，即KL散度是交叉熵减去真实分布的熵，当真实分布是确定性分布（熵为0，如标准分类任务的one-hot标签）时，最小化交叉熵等价于最小化KL散度。



![KL 散度在策略约束中防止模型崩溃的作用](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter5/fig_kl_constraint_effect.png)

</details>

**9. RL 中的 KL 散度系数怎么设置？**

<details>
<summary>💡 查看答案</summary>

不同论文中的设置：
- OpenAI（InstructGPT/PPO-RLHF）：KL reward coefficient $\approx 0.01-0.02$
- DPO：默认 $\beta = 0.1$；TL;DR 实验 $\beta = 0.5$
- DeepSeek（DeepSeekMath/GRPO）：KL = 0.04
- Qwen2.5-Math：KL = 0.001

没有“万能”数值，各论文反复强调要 tune KL——太小会允许模型偏离 reference 导致灾难性退化，太大会抑制收益。

</details>

