# 3.1 数学基础 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 3 章：预训练、推理与工程优化 · 共 14 题

**1. 请解释一下 KL 散度是什么？**

<details>
<summary>💡 查看答案</summary>

KL 散度（Kullback-Leibler Divergence）是一种衡量两个概率分布差异的非对称性度量工具，由 Solomon Kullback 和 Richard Leibler 于 1951 年提出。其定义如下：


$$
	D_{KL}(P \| Q) = \sum_{x} P(x) \log \frac{P(x)}{Q(x)}
$$


对于连续型随机变量，KL 散度的积分形式为：


$$
	D_{KL}(P \| Q) = \int_{-\infty}^{+\infty} p(x) \log \frac{p(x)}{q(x)} \, dx
$$


其中 $P$ 表示目标分布（真实分布），$Q$ 表示参考分布（近似分布）。KL 散度反映了从 $Q$ 转换到 $P$ 时的信息损失程度，可以理解为：若用分布 $Q$ 来近似描述实际服从分布 $P$ 的数据时，所额外需要的平均编码位数。

**重要性质：**
- **非负性**：$D_{KL}(P \| Q) \geq 0$，当且仅当 $P = Q$ 时等号成立（由 Gibbs 不等式保证）。
- **非对称性**：一般而言 $D_{KL}(P \| Q) \neq D_{KL}(Q \| P)$，因此 KL 散度不是严格意义上的距离度量。
- **不满足三角不等式**：这也是它不构成度量空间中"距离"的原因。
- **信息论解释**：KL 散度等于在真实分布 $P$ 下，使用最优编码方案编码 $P$ 和使用针对 $Q$ 设计的最优编码方案编码 $P$ 之间的平均比特数之差。

**与信息熵的关系：**

设 $H(P)$ 为分布 $P$ 的信息熵，$H(P, Q)$ 为 $P$ 关于 $Q$ 的交叉熵，则：


$$
	D_{KL}(P \| Q) = H(P, Q) - H(P)
$$


即 KL 散度等于交叉熵减去信息熵，体现了用 $Q$ 替代 $P$ 带来的额外信息代价。

</details>

**2. KL 散度与交叉熵的区别是什么？**

<details>
<summary>💡 查看答案</summary>

KL 散度与交叉熵都是信息论中衡量概率分布差异的工具，但二者有本质区别。

**数学关系：**

设真实分布为 $P$，预测分布为 $Q$，则：

$$
	信息熵： & \quad H(P) = -\sum_{x} P(x) \log P(x) \\
	交叉熵： & \quad H(P, Q) = -\sum_{x} P(x) \log Q(x) \\
	KL 散度： & \quad D_{KL}(P \| Q) = \sum_{x} P(x) \log \frac{P(x)}{Q(x)} = H(P, Q) - H(P)
$$


因此，KL 散度与交叉熵相差一个常数项 $H(P)$（真实分布的熵），在优化过程中最小化交叉熵等价于最小化 KL 散度。

**主要区别对比：**

\begin{table}[h]



| **特性** | **KL 散度** | **交叉熵** |
| --- | --- | --- |
| 非对称性 | 是（$D_{KL}(P\|Q) \neq D_{KL}(Q\|P)$） | 是 |
| 最小值 | 0（当 $P = Q$ 时） | $H(P)$（当 $P = Q$ 时） |
| 包含真实分布熵 | 是 | 否 |
| 机器学习中应用 | 变分推断、知识蒸馏等 | 分类损失函数 |
| 可作为距离度量 | 否 | 否 |


	\caption{KL 散度与交叉熵的对比}
\end{table}

**在机器学习中的应用差异：**
- **交叉熵**：更常用于分类任务的损失函数，计算简洁，当真实分布 $P$ 固定时，$H(P)$ 为常数，最小化交叉熵即最小化 KL 散度。
- **KL 散度**：常用于变分自编码器（VAE）的正则项、知识蒸馏（Knowledge Distillation）、强化学习中策略更新约束（如 PPO、TRPO）等场景，需要显式衡量两个可变分布之间的差异。

![信息熵、交叉熵与 KL 散度的数学关系示意图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_entropy_relation.png)

</details>

**3. 交叉熵损失函数是什么？它的物理意义是什么？**

<details>
<summary>💡 查看答案</summary>

交叉熵损失函数（Cross-Entropy Loss）来源于信息论，用于衡量两个概率分布之间的差异：


$$
	H(p, q) = -\sum_{i} p_i \log q_i
$$


其中 $p_i$ 是真实标签分布，$q_i$ 是模型预测的概率分布。

**二分类交叉熵（Binary Cross-Entropy）：**

当问题为二分类时（$y \in \{0, 1\}$），交叉熵退化为：


$$
	L = -\left[ y \log \hat{y} + (1 - y) \log (1 - \hat{y}) \right]
$$


其中 $\hat{y}$ 为模型输出的正类概率。

**物理与信息论意义：**

- 交叉熵 $H(p, q)$ 表示：在真实分布 $p$ 下，使用基于 $q$ 设计的最优编码方案所需的**平均编码位数**。
- 当预测分布 $q$ 与真实分布 $p$ 完全一致时，$H(p, q) = H(p)$（即真实分布的熵），损失取得最小值。
- 当两个分布差异越大，交叉熵越大，表示模型需要更多的"额外信息"来描述真实标签。
- 在监督学习中，真实标签通常是 one-hot 编码，即 $p_i = 1$ 对于正确类别，$p_j = 0$ 对于其他类别，此时交叉熵退化为：

    $$
    L = -\log q_{y}
    $$

    其中 $y$ 是真实类别索引，即仅惩罚模型对正确类别分配的概率的对数。

**梯度性质：**

交叉熵损失配合 Sigmoid/Softmax 激活函数时，梯度形式简洁：


$$
	\frac{\partial L}{\partial z} = \hat{y} - y
$$


其中 $z$ 为 logit（激活函数前的输出），该梯度不存在梯度消失问题，这也是交叉熵优于 MSE 用于分类的重要原因之一。

</details>

**4. 分类问题中，为什么使用交叉熵损失函数而不是均方误差（MSE）？**

<details>
<summary>💡 查看答案</summary>

在分类问题中，交叉熵损失函数相较于均方误差（MSE）具有显著优势，原因如下：

**1. 概率分布的适配性**

分类问题的输出通过 Softmax/Sigmoid 映射为概率值，交叉熵天然衡量两个概率分布之间的差异，与任务目标一致；而 MSE 衡量数值差异，与概率的语义不匹配。

**2. 梯度消失问题**

以二分类为例，使用 Sigmoid 激活函数 $\sigma(z) = \frac{1}{1+e^{-z}}$ 时：

- 使用 **MSE** 的梯度为：

    $$
    \frac{\partial L_{MSE}}{\partial z} = (\hat{y} - y) \cdot \sigma(z)(1 - \sigma(z))
    $$

    当预测值 $\hat{y}$ 接近 0 或 1 时，$\sigma(z)(1 - \sigma(z)) \approx 0$，导致梯度几乎为零，训练极慢。
- 使用**交叉熵**的梯度为：

    $$
    \frac{\partial L_{CE}}{\partial z} = \hat{y} - y
    $$

    梯度仅与预测误差相关，不存在梯度消失问题，训练更稳定高效。

**3. 对错误预测的惩罚力度**

交叉熵含有对数函数，当模型对错误类别给出极高置信度时（如预测概率接近 1 但实际为 0），$-\log(\epsilon) \to +\infty$，惩罚极大；而 MSE 的最大惩罚有界（$(1-0)^2 = 1$），无法有效纠正高置信度的错误预测。

**4. 统计视角：最大似然估计**

对于分类问题，假设标签服从类别分布，交叉熵损失等价于对模型参数做**最大似然估计（MLE）**，具有良好的统计理论支撑；而 MSE 对应的是高斯噪声假设，与分类任务的数据生成过程不符。

**总结对比：**

\begin{table}[h]



| **特性** | **交叉熵** | **MSE** |
| --- | --- | --- |
| 适用任务 | 分类 | 回归 |
| 梯度消失 | 无（配合 Softmax/Sigmoid） | 有（在饱和区域） |
| 对高置信度错误的惩罚 | 极大（对数惩罚） | 有界（最大为 1） |
| 统计理论支撑 | 最大似然估计 | 高斯噪声假设 |


	\caption{交叉熵与 MSE 的对比}
\end{table}

![交叉熵与 MSE 在二分类任务中的损失函数与梯度对比图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter3/fig_loss_comparison.png)

</details>

**5. 多分类问题中的分类损失函数（Softmax）是什么样的？**

<details>
<summary>💡 查看答案</summary>

在多分类问题中，通常将神经网络最后一层的输出 $\mathbf{z} = (z_1, z_2, \ldots, z_n)$（称为 logits）通过 Softmax 函数转化为概率分布：


$$
	p_i = softmax(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{n} e^{z_j}}, \quad i = 1, 2, \ldots, n
$$


随后，使用交叉熵计算损失（称为 Softmax 交叉熵损失）：


$$
	L = -\sum_{i=1}^{n} y_i \log p_i
$$


其中 $n$ 是类别数，$y_i$ 是第 $i$ 类的真实标签（通常为 one-hot 编码），$p_i$ 是第 $i$ 类的预测概率。

**One-hot 标签下的简化形式：**

由于 $y$ 为 one-hot 向量，设正确类别为 $c$（即 $y_c = 1$，其余为 0），损失简化为：


$$
	L = -\log p_c = -\log \frac{e^{z_c}}{\sum_{j=1}^{n} e^{z_j}}
$$


**对 logits 的梯度：**

Softmax 交叉熵损失对 logit $z_i$ 的梯度具有非常简洁的形式：


$$
	\frac{\partial L}{\partial z_i} = p_i - y_i
$$


即预测概率与真实标签之差，这使得反向传播计算非常高效。

**标签平滑（Label Smoothing）：**

在实际应用中，为防止模型对训练集过拟合、输出过于极端的概率，常对 one-hot 标签进行平滑处理：


$$
	y_i^{smooth} = (1 - \varepsilon) \cdot y_i + \frac{\varepsilon}{n}
$$


其中 $\varepsilon$ 为平滑系数（通常取 0.1），从而使模型输出更具有校准性（calibration）。

</details>

**6. 如果 Softmax 的 $e$ 次方超过浮点数范围怎么办？**

<details>
<summary>💡 查看答案</summary>

当 logit 向量中存在较大数值时，直接计算 $e^{z_i}$ 可能导致浮点数上溢（overflow）；当数值较小时可能导致下溢（underflow），进而出现除以零的情况。

**数值稳定的 Softmax：**

利用 Softmax 的平移不变性，将分子分母同时乘以 $e^{-\max(\mathbf{z})}$：


$$
	softmax(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}} = \frac{e^{z_i - M}}{\sum_j e^{z_j - M}}, \quad M = \max_j(z_j)
$$


减去最大值后，所有指数项满足 $z_i - M \leq 0$，从而 $e^{z_i - M} \in (0, 1]$，彻底避免上溢，同时最大项为 1，避免所有项同时下溢至 0。

**数值稳定的 Log-Softmax：**

在实际训练中，通常需要计算 $\log p_i$（用于交叉熵损失）。若先计算 Softmax 再取对数，可能引入额外的数值误差。更稳定的做法是直接计算 Log-Softmax：


$$
	\log softmax(z_i) = z_i - M - \log \sum_j e^{z_j - M}
$$


其中 $\log \sum_j e^{z_j - M}$ 即 log-sum-exp 操作，可以高效且稳定地计算。

**代码示例（NumPy 实现）：**

\begin{lstlisting}[style=promptstyle]
	import numpy as np

	def stable_softmax(z):
	# 减去最大值以保证数值稳定性
	z_shifted = z - np.max(z)
	exp_z = np.exp(z_shifted)
	return exp_z / np.sum(exp_z)

	def stable_log_softmax(z):
	# 数值稳定的 log-softmax
	M = np.max(z)
	log_sum_exp = M + np.log(np.sum(np.exp(z - M)))
	return z - log_sum_exp

	# 示例
	z = np.array([1000.0, 1001.0, 1002.0])  # 大数值场景
	print(stable_softmax(z))       # [0.0900, 0.2447, 0.6652]
	print(stable_log_softmax(z))   # [-2.4076, -1.4076, -0.4076]
\end{lstlisting}

**为什么平移不变性成立？**

设 $M$ 为任意常数：


$$
	\frac{e^{z_i - M}}{\sum_j e^{z_j - M}} = \frac{e^{z_i} \cdot e^{-M}}{\sum_j e^{z_j} \cdot e^{-M}} = \frac{e^{z_i}}{\sum_j e^{z_j}}
$$


公分母 $e^{-M}$ 可以消去，因此平移操作不改变 Softmax 的输出值，仅改善数值稳定性。

**深度学习框架的实现：**

主流框架（如 PyTorch、TensorFlow）在实现 `nn.CrossEntropyLoss` 或 `tf.nn.softmax_cross_entropy_with_logits` 时，均内置了上述数值稳定处理，直接接受 logits 作为输入，无需用户手动计算 Softmax，从而同时保证数值稳定性与计算效率。A

</details>

**7. 矩阵乘法的计算复杂度是多少？在 Transformer 中哪里最耗时？**

<details>
<summary>💡 查看答案</summary>

两个矩阵 $A \in \mathbb{R}^{m \times k}$ 与 $B \in \mathbb{R}^{k \times n}$ 相乘，标准算法的计算复杂度为 $O(mkn)$。

在 Transformer 中，主要的矩阵乘法开销集中在以下几处：

1. **Self-Attention 的 QK$^\top$ 计算**：形状为 $(B, H, L, d_k) \times (B, H, d_k, L)$，复杂度为 $O(L^2 d_k)$，随序列长度 $L$ 平方增长，是长序列场景的主要瓶颈。
2. **Attention Score 与 V 的加权求和**：复杂度同为 $O(L^2 d_k)$。
3. **FFN 层的线性变换**：形状通常为 $(B, L, d) \times (d, 4d)$，复杂度为 $O(L \cdot d \cdot 4d)$。对于长序列，FFN 的总计算量往往超过 Attention。
4. **输入/输出投影（QKV Projection、Output Projection）**：复杂度为 $O(L \cdot d^2)$。

**总结**：短序列时 FFN 是主要开销；长序列时 $O(L^2)$ 的 Attention 计算成为瓶颈，这也是 FlashAttention、线性 Attention 等优化方案的核心动机。

</details>

**8. 什么是 SVD 分解？它在大模型中有哪些应用？**

<details>
<summary>💡 查看答案</summary>

**奇异值分解（SVD）** 将任意矩阵 $W \in \mathbb{R}^{m \times n}$ 分解为：

$$W = U \Sigma V^\top$$

其中 $U \in \mathbb{R}^{m \times m}$、$V \in \mathbb{R}^{n \times n}$ 为正交矩阵，$\Sigma \in \mathbb{R}^{m \times n}$ 为对角矩阵，对角元素 $\sigma_1 \geq \sigma_2 \geq \cdots \geq 0$ 称为奇异值。

**低秩近似**：取前 $r$ 个最大奇异值，得到 $\hat{W} = U_r \Sigma_r V_r^\top$，是秩为 $r$ 的最优近似（Eckart–Young 定理）。

**在大模型中的应用**：

1. **LoRA**：冻结预训练权重 $W_0$，用低秩矩阵 $\Delta W = BA$（$B \in \mathbb{R}^{m \times r}$，$A \in \mathbb{R}^{r \times n}$，$r \ll \min(m,n)$）近似权重更新，其理论基础正是低秩假设——模型微调时的参数变化量本质上是低秩的。
2. **模型压缩**：对权重矩阵做 SVD 后截断小奇异值，减少参数量。
3. **分析模型内部结构**：用奇异值分布诊断权重矩阵的"有效秩"，判断是否存在冗余。

</details>

**9. 什么是最大似然估计（MLE）？语言模型训练与它的关系是什么？**

<details>
<summary>💡 查看答案</summary>

**最大似然估计（Maximum Likelihood Estimation, MLE）** 是一种参数估计方法：给定观测数据 $\mathcal{D} = \{x_1, x_2, \ldots, x_N\}$，寻找参数 $\theta$ 使数据的联合概率（似然）最大：

$$\hat{\theta}_{MLE} = \arg\max_\theta \prod_{i=1}^N p_\theta(x_i) = \arg\max_\theta \sum_{i=1}^N \log p_\theta(x_i)$$

取对数后转化为对数似然求和，便于计算且数值稳定。

**与语言模型训练的关系**：

自回归语言模型的训练目标是最小化交叉熵损失：

$$\mathcal{L} = -\frac{1}{T} \sum_{t=1}^T \log p_\theta(x_t \mid x_{<t})$$

这等价于对训练语料做最大似然估计——最大化模型对真实 token 序列的对数概率。因此，**语言模型的预训练本质上就是在做 MLE**，交叉熵损失就是负对数似然。

</details>

**10. 什么是链式法则？反向传播的数学本质是什么？**

<details>
<summary>💡 查看答案</summary>

**链式法则**是微积分中复合函数求导的基本规则：若 $y = f(g(x))$，则：

$$\frac{dy}{dx} = \frac{dy}{dg} \cdot \frac{dg}{dx}$$

推广到多层复合（如神经网络）：设 $L = f_n \circ f_{n-1} \circ \cdots \circ f_1(x)$，则损失对第 $k$ 层参数 $\theta_k$ 的梯度为：

$$\frac{\partial L}{\partial \theta_k} = \frac{\partial L}{\partial f_n} \cdot \frac{\partial f_n}{\partial f_{n-1}} \cdots \frac{\partial f_{k+1}}{\partial f_k} \cdot \frac{\partial f_k}{\partial \theta_k}$$

**反向传播（Backpropagation）的本质**：就是对上述链式法则的高效实现，通过**从输出层向输入层逐层传递梯度**，复用中间计算结果，避免重复计算，将复杂度从指数级降为线性。

- **前向传播**：计算并缓存各层的激活值。
- **反向传播**：利用缓存的激活值，按链式法则逐层计算梯度。

梯度消失问题（深层网络中靠近输入层的梯度趋近于零）本质上是链式法则中大量小于 1 的梯度连乘导致的指数衰减。

</details>

**11. 什么是梯度消失和梯度爆炸？从数学角度如何理解？**

<details>
<summary>💡 查看答案</summary>

在深度网络中，损失 $L$ 对第 $l$ 层参数的梯度需要经过 $(n-l)$ 层的链式相乘：

$$\frac{\partial L}{\partial \theta_l} = \frac{\partial L}{\partial h_n} \cdot \prod_{k=l+1}^{n} \frac{\partial h_k}{\partial h_{k-1}} \cdot \frac{\partial h_l}{\partial \theta_l}$$

其中 $\frac{\partial h_k}{\partial h_{k-1}} = W_k^\top \cdot diag(\sigma'(h_{k-1}))$。

- **梯度消失**：若每层的 Jacobian 矩阵的谱范数（最大奇异值）$< 1$，连乘后梯度指数衰减趋于 0，深层参数几乎无法更新。Sigmoid/Tanh 激活函数在饱和区导数接近 0，是经典原因。
- **梯度爆炸**：若谱范数 $> 1$，连乘后梯度指数增大，导致训练不稳定（loss spike、NaN）。

**大模型中的应对措施**：

1. **残差连接**：$h_l = f(h_{l-1}) + h_{l-1}$，梯度可直接通过恒等路径回流，缓解消失。
2. **Layer Normalization**：稳定各层激活分布，避免饱和。
3. **梯度裁剪（Gradient Clipping）**：限制梯度范数上界，防止爆炸。
4. **合理的权重初始化**（如 Xavier、He 初始化）：使初始状态下各层 Jacobian 谱范数接近 1。

</details>

**12. 什么是熵？语言模型的 Perplexity 与熵有什么关系？**

<details>
<summary>💡 查看答案</summary>

**熵（Entropy）** 衡量随机变量的不确定性，对于离散分布 $p$：

$$H(p) = -\sum_{x} p(x) \log p(x)$$

熵越大，分布越均匀，不确定性越高；熵越小，分布越集中，越容易预测。

**Perplexity（困惑度）** 是语言模型的标准评估指标，定义为测试集上每个 token 平均负对数似然的指数：

$$PPL = \exp\left(-\frac{1}{T} \sum_{t=1}^T \log p_\theta(x_t \mid x_{<t})\right)$$

**两者关系**：

- $-\frac{1}{T} \sum_t \log p_\theta(x_t \mid x_{<t})$ 正是模型分布相对于真实数据分布的**交叉熵**的经验估计。
- 因此 $PPL = 2^{H(p, q)}$（以 2 为底）或 $e^{H(p,q)}$（以 $e$ 为底），即**困惑度是交叉熵的指数**。
- 当模型完美拟合真实分布时，交叉熵等于数据的真实熵，PPL 取得理论下界。
- 直觉上，PPL 可理解为模型在每个位置平均需要从多少个等可能选项中猜测下一个 token。PPL = 10 意味着模型平均在 10 个选项中选择。

</details>

**13. Adam 优化器的核心思想是什么？为什么比 SGD 更适合大模型训练？**

<details>
<summary>💡 查看答案</summary>

**Adam（Adaptive Moment Estimation）** 结合了动量（Momentum）和自适应学习率（RMSProp）两种思想：

1. **一阶矩估计**（动量）：对梯度做指数移动平均，平滑更新方向：
    $$m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$$
2. **二阶矩估计**（自适应学习率）：对梯度平方做指数移动平均，估计梯度的方差：
    $$v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2$$
3. **偏差校正**：修正初始阶段的偏差：$\hat{m}_t = m_t / (1-\beta_1^t)$，$\hat{v}_t = v_t / (1-\beta_2^t)$
4. **参数更新**：
    $$\theta_t = \theta_{t-1} - \alpha \cdot \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

**为什么比 SGD 更适合大模型**：

- **自适应学习率**：不同参数的梯度尺度差异巨大（如 Embedding 层 vs FFN 层），Adam 为每个参数独立调整有效学习率，SGD 用统一学习率难以兼顾。
- **对超参数不敏感**：Adam 对学习率的初始设置更宽容，调参成本低。
- **稀疏梯度友好**：在 Embedding 等参数更新稀疏的场景，自适应学习率能为更新频率低的参数保持较大步长。
- **训练稳定性**：动量的平滑效果减少梯度噪声，在大 batch 训练中尤为重要。

实际使用中通常采用 **AdamW**（将权重衰减从梯度更新中解耦），是当前 LLM 训练的事实标准。

</details>

**14. 什么是贝叶斯定理？在语言模型中有哪些体现？**

<details>
<summary>💡 查看答案</summary>

**贝叶斯定理**描述了在获得新证据后如何更新对事件概率的信念：

$$P(\theta \mid \mathcal{D}) = \frac{P(\mathcal{D} \mid \theta) \cdot P(\theta)}{P(\mathcal{D})}$$

其中：
- $P(\theta)$：先验概率（Prior），对参数的初始信念。
- $P(\mathcal{D} \mid \theta)$：似然（Likelihood），给定参数时观测数据的概率。
- $P(\theta \mid \mathcal{D})$：后验概率（Posterior），观测数据后对参数的更新信念。
- $P(\mathcal{D})$：证据（Evidence），归一化常数。

**在语言模型中的体现**：

1. **MLE vs MAP**：标准预训练做的是 MLE（最大化似然），等价于无先验的贝叶斯估计；加入 L2 正则（Weight Decay）等价于对参数加高斯先验的 MAP（最大后验估计）。
2. **In-Context Learning 的贝叶斯解释**：Few-shot 示例可理解为给模型提供"证据"，更新模型对任务的"后验理解"，本质是隐式的贝叶斯推断。
3. **RAG 的贝叶斯视角**：检索到的文档作为先验知识，结合用户 query（似然），生成更准确的后验回答。
4. **RLHF 中的奖励建模**：对人类偏好数据建模，本质是在估计人类偏好分布的后验。

</details>

