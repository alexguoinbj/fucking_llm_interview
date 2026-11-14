# 1.2 大模型架构与原理 — 题目与答案
> [⬅ 返回目录](../README_ZH.md) · 第 1 章：大模型基础知识点 · 共 107 题

**1. 为什么大模型都是 Decoder-only 结构？**

<details>
<summary>💡 查看答案</summary>

在探讨当前大型语言模型（LLM）普遍采用 Decoder-only 架构的现象时，可以从以下几个角度进行分析：

1. **注意力机制的满秩特性**：Decoder-only 架构采用的因果注意力机制（causal attention）形成了一个下三角矩阵，这种结构保证了矩阵的满秩性（即行列式非零），从而赋予模型更强的建模能力。相比之下，Encoder 架构中的双向注意力机制可能会导致矩阵的秩降低，进而影响模型的表达能力。

2. **预训练任务的难度**：在 Decoder-only 架构中，模型在预训练阶段每个位置只能接触到有限的信息，这增加了预测下一个 token 的难度。然而，当模型规模足够大且数据量充足时，Decoder-only 模型在学习和泛化方面表现出更高的上限。

3. **上下文学习能力**：Decoder-only 架构在上下文学习方面表现更为优异，这是因为 prompt 和 demonstration 信息可以被视为对模型参数的一种隐式微调。这种特性使得 Decoder-only 架构在 few-shot 学习场景中具有明显优势。

4. **效率问题**：Decoder-only 架构支持 KV-Cache 的复用，这对于多轮对话任务更为友好。在这种架构中，每个 token 的表示仅依赖于之前的输入，而 Encoder-Decoder 架构则难以实现这一点。

5. **Zero-Shot 性能**：Decoder-only 模型在没有额外 tuning 数据的情况下，展现出更好的 zero-shot 性能。相比之下，Encoder-Decoder 架构则需要在一定量的标注数据上进行多任务微调（multitask finetuning），以激发其最佳性能。

综上所述，Decoder-only 架构因其在训练效率、工程实现以及理论上的优势，成为了当前 LLM 设计的主流选择。

</details>

**2. Prefix LM 和 Causal LM 的区别是啥？**

<details>
<summary>💡 查看答案</summary>

就是 attention mask 不一样。它们定义如下：

- **Prefix LM（前缀语言模型）**：在输入序列的开头添加一个可学习的任务相关的前缀，然后使用这个前缀和输入序列一起生成输出。这种方法可以引导模型生成适应特定任务的输出。
- **Causal LM（因果语言模型）**：也称为自回归语言模型，它根据之前生成的 token 预测下一个 token。在生成文本时，模型只能根据已经生成的部分生成后续部分，不能访问未来的信息。

</details>

**3. Prefix LM、Causal LM、Encoder-Decoder 的区别及各自优缺点**

<details>
<summary>💡 查看答案</summary>

**Prefix LM**：解码器可以访问整个输入序列（包括前缀和之前生成的输出），从而更好地理解上下文。代表模型：T5、UniLM。优点是可以减少对预训练模型参数的修改，降低过拟合风险；缺点是可能受到前缀表示长度的限制。

**Causal LM**：使用掩码确保在生成每个词时只能考虑它之前（包括当前）的词。代表模型：GPT 系列。优点是可以生成灵活的文本，适应各种生成任务；缺点是无法访问未来的信息，可能生成不一致或有误的内容。

**Encoder-Decoder**：编码器使用双向注意力，解码器使用单向注意力。代表模型：Transformer、Flan-T5、BART。优点是可以处理输入和输出序列不同长度的任务；缺点是模型结构较为复杂，训练和推理计算量较大。

</details>

**4. 简单介绍 Transformer 网络？**

<details>
<summary>💡 查看答案</summary>

大多数现代 LLM 的核心是 Transformer 架构。这种神经网络摒弃了传统的循环神经网络（RNN），擅长理解序列中的长距离依赖关系，使其特别适合语言处理任务。Transformer 由两个子组件组成：

- **编码器**：这一部分处理输入文本，将其分解为一系列编码表示，捕捉词之间的关系。
- **解码器**：在这里，模型利用编码器提供的编码信息逐词生成输出文本。



![Transformer Encoder 结构](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/transformer_encoder.png)



**自注意力机制**：Transformer 中的这一巧妙机制使模型能够专注于输入序列中与给定词或短语最相关的部分。它根据对当前预测的重要性，不同程度地关注输入文本的不同部分。这种能力对于 LLM 理解语言和上下文的细微差别至关重要。

**输入嵌入和输出解码**：
- **输入嵌入**：在将文本数据输入 LLM 之前，词嵌入将其转换为数值表示。这一过程将词转换为向量，捕捉它们的语义相似性和关系。
- **输出解码**：LLM 处理完编码输入后，通过解码将内部表示转换回人类可读的文本。

**模型大小和参数数量**：LLM 中的参数数量（权重和偏差）对其能力有重大影响。大规模 LLM 通常拥有数十亿甚至数千亿参数，使其能够学习语言数据中的复杂模式和关系。然而，这也需要大量的计算资源来训练和运行模型。

</details>

**5. Transformer 架构如何克服 Seq2Seq 模型的挑战？**

<details>
<summary>💡 查看答案</summary>

Transformer 架构通过用以下方式替代递归机制，彻底改变了序列到序列任务：
- **自注意力**：并行处理所有标记，高效捕获全局依赖关系。
- **位置编码**：添加标记顺序信息。
- **可扩展性**：处理长序列而不会出现 RNN 固有的梯度消失问题。
Transformer 更快，更适合需要对长输入序列进行上下文感知预测的任务。

</details>

**6. Transformer 模型的基本结构包括哪些部分？**

<details>
<summary>💡 查看答案</summary>

Transformer 整体采用编码器-解码器（Encoder-Decoder）架构，主要由以下部分组成：

- **输入嵌入层（Input Embedding）**：将输入的离散 token（词或子词）映射为连续的向量表示。
- **位置编码（Positional Encoding）**：由于模型本身不具备顺序信息，需要将位置信息以向量形式叠加到词嵌入上。
- **编码器（Encoder）**：由 N 个结构相同的层堆叠而成，每层包含一个多头自注意力子层和一个前馈神经网络子层，每个子层后都有残差连接和层归一化（Add & Norm）。编码器负责将输入序列转换为一组富含上下文信息的表示。
- **解码器（Decoder）**：同样由 N 个结构相同的层堆叠而成，但每层包含三个子层：带掩码的多头自注意力（防止看到未来信息）、编码器-解码器注意力（融合编码器的输出信息）、前馈神经网络，每个子层后同样有 Add & Norm。
- **输出层**：解码器最后接一个线性层，将隐藏状态映射到词表大小的维度，再经过 Softmax 得到下一个 token 的概率分布。

整体来看，编码器负责"理解"输入，解码器负责"生成"输出，二者通过编码器-解码器注意力相连接。

</details>

**7. Transformer 模型中的"注意力遮蔽"（Attention Masking）是如何工作的？**

<details>
<summary>💡 查看答案</summary>

注意力遮蔽的核心思想是：在计算 Softmax 之前，将不希望被关注的位置的注意力分数设置为一个非常大的负数（如 -1e9 或 -$\infty$），这样经过 Softmax 后，这些位置对应的权重会趋近于 0，从而在加权求和时几乎不产生任何影响，相当于"屏蔽"了这些位置。

Transformer 中常见的两种掩码：

1. **填充掩码（Padding Mask）**：由于一个批次内的序列长度不同，需要通过 padding 补齐到相同长度。填充掩码用于标记哪些位置是真实 token、哪些是填充的占位符，使模型在计算注意力时忽略填充部分，避免无意义的信息干扰。
2. **序列掩码 / 因果掩码（Look-ahead Mask / Causal Mask）**：仅在解码器的自注意力中使用，作用是确保位置 i 的 token 在计算注意力时，只能看到位置 1 到 i 的信息，而不能看到位置 i+1 及之后的信息。这通常通过构造一个上三角矩阵（对角线以上为 -$\infty$）来实现，保证了模型在训练和推理时行为一致，符合自回归生成的因果关系。

在实际实现中，这两种掩码可以叠加使用，共同构成最终的注意力掩码矩阵。

</details>

**8. 为何 Transformer 选择使用 Softmax 函数来计算注意力权重？**

<details>
<summary>💡 查看答案</summary>

Transformer 在计算注意力时，需要将 Query 与各个 Key 的点积结果（一组实数分数）转换为可用于加权求和的权重，Softmax 函数在这里非常合适，原因主要包括：

1. **归一化为概率分布**：Softmax 将任意一组实数映射为非负数且和为 1 的分布，这使得这些权重可以直接理解为"注意力强度"或"重要性占比"，并能够直接用于对 Value 向量做凸组合（加权平均）。
2. **平滑且可微**：Softmax 是一个光滑的、处处可导的函数，便于在反向传播中计算梯度，使整个注意力机制可以端到端训练。
3. **突出相对差异**：由于 Softmax 中含有指数运算，分数较大的项会被指数级放大，分数较小的项会被相对压缩，这使得模型能够更明确地将注意力集中在少数最相关的位置上，而不是均匀地分散注意力。
4. **与缩放点积配合**：Transformer 在 Softmax 之前会将点积结果除以 $\sqrt{}$$d_k$（缩放因子），这是为了避免当向量维度较大时点积值过大，导致 Softmax 的梯度过小（梯度消失），从而保证训练的稳定性。

综合来看，Softmax 既满足了概率解释的需要，又兼顾了训练的可行性和稳定性，是计算注意力权重的自然选择。

</details>

**9. Transformer 模型如何处理不同长度的输入序列？**

<details>
<summary>💡 查看答案</summary>

在实际应用中，一个批次（batch）内的输入序列长度往往不同，Transformer 主要通过以下方式处理这一问题：

1. **填充（Padding）**：将批次内所有序列填充（通常用特殊的 `<pad>` token）到相同的长度，使其可以组成一个规则的张量进行并行计算。
2. **填充掩码（Padding Mask）**：在注意力计算中，通过掩码将填充位置的注意力分数设为负无穷，使这些位置在 Softmax 后权重为 0，从而不会对真实 token 的表示产生影响，也不会让模型"学习"到填充符号的语义。
3. **位置编码的灵活性**：位置编码（无论是三角函数式还是可学习式）通常是按位置索引生成的，理论上可以适用于任意长度的序列。
4. **架构本身对长度的适应性**：自注意力机制对序列长度没有固定限制，不像 RNN 那样依赖逐步递归，因此从结构上可以处理任意长度的输入。

但需要注意的是，Transformer 的自注意力计算复杂度是 $O(n^2)$（n 为序列长度），因此实际可处理的最大长度会受到显存和计算资源的限制；同时模型在训练时通常会设置一个最大序列长度（如 512、1024、2048 等），对超出该长度的序列需要截断或采用滑动窗口、分块等策略；对于需要处理超长序列的场景，业界也提出了 Longformer、Reformer、Transformer-XL 等改进架构来降低计算复杂度并增强长程依赖建模能力。

</details>

**10. Transformer 中如何实现序列到序列的映射？**

<details>
<summary>💡 查看答案</summary>

Transformer 实现序列到序列（Seq2Seq）映射的整体流程如下：

1. **编码阶段**：输入序列（如源语言句子）经过嵌入层和位置编码后，输入到编码器中。编码器通过多层自注意力和前馈网络，将输入序列转换为一组与输入等长的上下文表示（每个位置的向量都融合了整个输入序列的信息）。
2. **解码阶段（自回归生成）**：解码器在训练时以"teacher forcing"方式接收目标序列（右移一位，并加上起始符），在推理时则从一个起始符 `<bos>` 开始，逐步生成输出：
- 解码器对已生成的部分目标序列进行掩码自注意力计算，得到当前生成位置的表示；
- 通过编码器-解码器注意力，将编码器的输出信息融合进来；
- 经过前馈网络后，再通过输出线性层和 Softmax，得到当前位置在整个词表上的概率分布；
- 选择概率最高（或通过采样、束搜索等策略选择）的 token 作为当前位置的输出，并将其拼接到已生成序列的末尾，作为下一步解码的输入。

1. **终止条件**：当生成了结束符 `<eos>` 或达到预设的最大长度时，生成过程结束，最终得到完整的输出序列。

通过这种"编码器一次性编码、解码器逐步自回归生成"的方式，Transformer 实现了从任意长度输入序列到任意长度输出序列的映射，这也是它在机器翻译、文本摘要、对话生成等任务中广泛应用的基础。

</details>

**11. Transformer 模型中的参数共享策略具体是指什么？**

<details>
<summary>💡 查看答案</summary>

Transformer 及其变体中常见的参数共享策略主要包括以下几种：

1. **输入嵌入与输出投影共享（Tied Embeddings / Weight Tying）**：解码器最后输出层的线性变换矩阵（将隐藏状态映射到词表维度）与词嵌入矩阵共享同一组参数（或为其转置）。这样做既减少了参数量，又因为嵌入空间和输出空间被约束在同一语义空间中，往往有助于提升模型性能，在 GPT、T5 等模型中较为常见。
2. **编码器与解码器之间的嵌入共享**：在某些机器翻译或多语言模型中，编码器和解码器可以共享同一套词嵌入参数（尤其当源语言和目标语言共享词表时）。
3. **跨层参数共享（Layer-wise Sharing）**：标准 Transformer 的各层（Layer）虽然结构相同，但参数是独立的，即每一层都有自己的一套权重。而像 **ALBERT**、**Universal Transformer** 等模型则会让不同层之间共享同一组参数（例如所有 Transformer block 共用一套权重，循环执行），从而大幅减少模型的总参数量，同时让模型在一定程度上具有"递归"计算的特性。
4. **多头注意力内部的共享**：在标准实现中，多头注意力的每个头都有独立的线性投影矩阵；但在一些效率优化的变体（如 Multi-Query Attention、Grouped-Query Attention）中，会让多个 Query 头共享同一组 Key 和 Value 的投影参数，以减少推理时的显存占用和计算量。

总体来看，参数共享的核心目的是在尽量不损失模型表达能力的前提下，减少参数数量、降低过拟合风险、提升训练和推理效率。

</details>

**12. 如何理解 Transformer 中的自回归属性？**

<details>
<summary>💡 查看答案</summary>

自回归（Autoregressive）是指模型在生成序列时，当前时刻的输出依赖于之前所有时刻已经生成的输出，即：

\begin{verbatim}
P(y_1, y_2, ..., y_T) = P(y_1) * P(y_2|y_1) * P(y_3|y_1,y_2) * ... * P(y_T|y_1,...,y_{T-1})
\end{verbatim}

在 Transformer 解码器中，这一属性是通过**掩码自注意力（Masked Self-Attention）**来实现的：在计算第 t 个位置的注意力时，通过因果掩码（Causal Mask）强制将第 t+1 及之后位置的注意力分数置为 -$\infty$，使得位置 t 在计算自身表示时只能"看到"位置 1 到 t 的信息，无法获取未来信息。

自回归属性带来的影响包括：

1. **训练与推理的一致性**：训练时通常采用"teacher forcing"，即把真实的目标序列（右移一位）整体输入解码器并通过掩码并行计算所有位置的损失；推理时则必须严格按照自回归方式逐步生成，每生成一个 token 后才能预测下一个。
2. **推理效率问题**：由于推理时必须逐 token 生成，无法像训练时那样并行计算，因此自回归模型在推理阶段的速度通常较慢，这也是后续出现非自回归（Non-Autoregressive）生成模型、并行解码、KV Cache 缓存等优化技术的重要动机。
3. **保证生成的因果性和连贯性**：自回归属性使得模型生成的每个 token 都建立在前文已生成内容的基础上，是语言模型能够生成连贯、符合语法和语义的文本的关键机制之一。

</details>

**13. Transformer 模型的解码器部分是如何生成序列的？**

<details>
<summary>💡 查看答案</summary>

Transformer 解码器生成序列的过程是一个**自回归（Autoregressive）**的迭代过程，具体步骤如下：

1. **初始化**：在推理开始时，解码器的输入只有一个特殊的起始符（如 `<bos>`）。
2. **掩码自注意力**：解码器对当前已生成的序列（包括起始符）进行自注意力计算，并通过因果掩码（Causal Mask）确保每个位置只能关注它自己以及之前的位置，不能"看到"尚未生成的未来 token。
3. **编码器-解码器注意力（Cross-Attention）**：将上一步得到的表示作为 Query，与编码器输出的 Key、Value 进行注意力计算，融合输入序列（如源语言句子）的相关信息。
4. **前馈网络与归一化**：经过前馈神经网络（FFN）以及残差连接和层归一化后，得到当前层的输出，多层解码器依次重复上述 1-4 步。
5. **输出投影与概率计算**：解码器最后一层的输出经过一个线性层映射到词表大小的维度，再经过 Softmax 函数，得到下一个 token 在整个词表上的概率分布。
6. **采样/选择下一个 token**：根据该概率分布，使用某种解码策略选择下一个 token，常见策略包括：
- **贪心搜索（Greedy Search）**：每次直接选择概率最高的 token；
- **束搜索（Beam Search）**：同时维护多个候选序列，保留累计概率最高的若干条路径；
- **采样方法**：如 Top-k 采样、Top-p（核采样）、温度采样等，引入一定随机性以增加生成多样性。

1. **迭代生成**：将新生成的 token 拼接到已生成序列的末尾，作为下一步解码器的输入，重复步骤 2-6，直到生成结束符 `<eos>` 或达到预设的最大生成长度为止。

在实际部署中，为了避免每一步都重新计算之前所有位置的 Key 和 Value，通常会使用 **KV Cache** 缓存历史的 Key、Value 向量，从而显著提升自回归生成的推理效率。

</details>

**14. Transformer 模型在哪些 NLP 任务中展现了显著的性能提升？**

<details>
<summary>💡 查看答案</summary>

自 2017 年提出以来，Transformer 在几乎所有主流 NLP 任务上都带来了显著的性能提升，主要体现在：

1. **机器翻译**：Transformer 最初就是为机器翻译任务设计的，在 WMT 等翻译基准上大幅超越了基于 RNN/LSTM 的 Seq2Seq 模型，同时训练速度也因为可并行计算而大幅提升。
2. **语言建模与文本生成**：以 GPT 系列为代表的自回归 Transformer 模型，在语言建模困惑度、长文本生成的连贯性和创造性方面取得突破，能够生成高质量的文章、对话、代码等内容。
3. **文本理解类任务**：以 BERT 为代表的双向编码器 Transformer，通过预训练+微调的范式，在文本分类、情感分析、自然语言推理（NLI）、命名实体识别（NER）、问答系统（如 SQuAD）等任务上刷新了大量榜单记录。
4. **文本摘要**：无论是抽取式还是生成式摘要，Transformer 及其变体（如 BART、PEGASUS）都能更好地捕捉长文档中的关键信息并生成流畅的摘要。
5. **长距离依赖建模**：相比 RNN/LSTM 受限于序列递归带来的梯度消失/爆炸问题，Transformer 的自注意力可以直接建立任意两个位置之间的联系，在处理长文本、跨句子甚至跨段落的依赖关系上表现明显更好。
6. **多任务与迁移学习**：通过"预训练-微调"或"预训练-提示（Prompt）"的范式，同一个 Transformer 预训练模型可以迁移到几乎所有下游 NLP 任务上，大幅降低了针对每个任务单独设计模型架构的成本。

</details>

**15. Transformer 模型是如何被应用到非 NLP 任务中的？**

<details>
<summary>💡 查看答案</summary>

Transformer 最初是为自然语言处理设计的，但其核心思想——将输入表示为一个 token 序列，并通过自注意力建模序列内各元素之间的关系——具有很强的通用性，因此被广泛迁移到其他领域：

1. **计算机视觉（Vision Transformer, ViT）**：将一张图像切分成固定大小的图像块（patch），每个 patch 经过线性投影后被当作一个"token"，加上位置编码后输入标准的 Transformer 编码器，用于图像分类、目标检测（如 DETR）、图像分割等任务。后续的 Swin Transformer 等还引入了层级化结构和局部窗口注意力以适应视觉任务的特点。
2. **语音处理**：在语音识别（ASR）和语音合成（TTS）中，音频信号被处理为帧序列或频谱特征序列，输入 Transformer 编码器-解码器结构，如 Conformer（结合卷积和 Transformer）、Speech-Transformer 等。
3. **推荐系统**：将用户的历史行为序列（点击、购买、浏览记录）视为一个 token 序列，使用自注意力机制建模用户兴趣的动态变化，捕捉行为之间的长程依赖关系，用于点击率预测、序列推荐等。
4. **生物信息学**：在蛋白质结构预测领域，AlphaFold2 等模型利用类似 Transformer 的注意力机制（Evoformer）来建模氨基酸序列之间以及序列与结构信息之间的关系，取得了突破性进展。
5. **时间序列预测**：将时间序列的历史观测值作为输入序列，利用自注意力建模不同时间步之间的依赖关系，用于金融预测、销量预测、气象预测等场景（如 Informer、Autoformer 等改进模型）。
6. **强化学习**：Decision Transformer 等工作将强化学习中的"状态-动作-回报"轨迹建模为序列数据，把策略学习问题转化为序列建模问题，用 Transformer 直接预测动作序列。
7. **多模态学习**：通过将图像、文本、音频等不同模态的数据统一编码为 token 序列，Transformer 可以作为统一的骨干网络处理多模态输入，如 CLIP、Flamingo、GPT-4V 等多模态大模型。

可以说，Transformer 已经从一种 NLP 专用架构演变为一种通用的"序列建模"基础架构。

</details>

**16. Transformer 的哪些变体或改进模型在工业界得到了广泛应用？**

<details>
<summary>💡 查看答案</summary>

经过多年发展，基于原始 Transformer 衍生出了大量变体和改进模型，许多已经在工业界得到大规模应用，主要包括：

1. **BERT 及其家族**：BERT 仅使用 Transformer 编码器部分，通过掩码语言模型（MLM）和下一句预测（NSP）进行预训练，广泛应用于搜索引擎的语义理解、文本分类、信息抽取等场景；其后续变体如 RoBERTa、ALBERT、ERNIE 等也在工业界被大量采用。
2. **GPT 系列**：仅使用 Transformer 解码器部分，通过自回归语言建模进行预训练，是当前大语言模型（LLM）的主流架构，被应用于对话系统、内容生成、代码生成等几乎所有生成式 AI 产品中。
3. **T5 / BART**：采用完整的编码器-解码器结构，将各种 NLP 任务统一建模为"文本到文本"的形式，广泛用于摘要、翻译、问答等生成式任务。
4. **Vision Transformer (ViT) 及其变体**：包括 Swin Transformer、DeiT 等，已成为图像分类、检测、分割等视觉任务的主流骨干网络之一，被集成到许多视觉 AI 产品中。
5. **高效注意力变体**：如 Longformer、BigBird、Reformer 等通过稀疏注意力或局部+全局注意力的组合，降低了长序列处理的计算复杂度，应用于需要处理长文档的场景（如法律、医疗文本分析）。
6. **大规模预训练语言模型**：如 LLaMA、PaLM、Qwen、ChatGLM 等开源或商用大模型，普遍采用经过优化的 Decoder-only Transformer 架构（如使用 RoPE 位置编码、RMSNorm、Grouped-Query Attention 等改进），被广泛部署在搜索、客服、办公自动化、代码辅助等众多产品中。
7. **混合专家模型（MoE）结合 Transformer**：如 Switch Transformer、Mixtral 等，在 Transformer 的前馈层引入混合专家结构，在不显著增加推理成本的情况下大幅扩展模型容量，是当前超大规模模型的重要技术路线之一。

</details>

**17. 未来 Transformer 模型的发展趋势是什么？**

<details>
<summary>💡 查看答案</summary>

结合当前研究和工业界的发展方向，Transformer 模型未来可能呈现以下几个趋势：

1. **更高效的注意力机制**：标准自注意力的计算和内存复杂度为 $O(n^2)$，限制了模型处理超长上下文的能力。未来会持续探索稀疏注意力、线性注意力（如 Linear Attention、Performer）、低秩近似等方法，在尽量保持效果的前提下降低复杂度，支持更长的上下文窗口（从几千 token 扩展到数十万甚至上百万 token）。
2. **架构上的混合与替代**：以 Mamba 等为代表的状态空间模型（State Space Model, SSM）在长序列建模上展现出与 Transformer 相当甚至更优的效果，且具有线性复杂度。未来可能出现更多 Transformer 与 SSM、卷积、RNN 等架构相结合的混合模型，兼顾效果与效率。
3. **混合专家（MoE）与稀疏化**：通过 MoE 等稀疏激活机制，在推理时只激活部分参数，从而在大幅扩展模型总参数量、提升模型容量的同时，控制实际计算成本，是通向更大规模模型的重要路径。
4. **多模态统一建模**：Transformer 作为通用序列建模架构，将进一步成为文本、图像、音频、视频、3D 等多种模态数据的统一骨干网络，推动多模态大模型（如 GPT-4V、Gemini 等）的发展。
5. **推理效率优化**：包括模型量化（INT8/INT4 等）、知识蒸馏、模型剪枝、KV Cache 优化、推测解码（Speculative Decoding）等技术，将持续被研究和应用，以降低大模型的部署和推理成本，使其能够运行在边缘设备或资源受限的环境中。
6. **与外部工具和系统的结合**：检索增强生成（RAG）、工具调用、智能体（Agent）框架等会与 Transformer 模型进一步融合，使模型不仅依赖自身参数中的知识，还能动态获取外部信息、调用外部工具完成更复杂的任务。
7. **训练范式的演进**：包括更高效的预训练目标设计、强化学习（如 RLHF、DPO）与人类偏好对齐技术的进一步发展，以及针对特定领域（如代码、数学、科学计算）的专用预训练和微调策略，将持续提升模型在复杂推理和专业任务上的表现。

</details>

**18. Transformer 模型中的"负面注意力"（Negative Attention）是什么意思？**

<details>
<summary>💡 查看答案</summary>

需要说明的是，"负面注意力（Negative Attention）"并不是 Transformer 原始论文及主流文献中的标准术语，更多是面试或讨论中对注意力机制某些现象的一种通俗说法，常见的几种理解角度包括：

1. **低权重 / 接近零的注意力**：在 Softmax 输出的注意力分布中，大部分位置的权重可能非常接近 0，这意味着模型判断这些位置与当前 Query 的相关性很低，几乎不贡献信息，可以理解为一种"被忽略"或"负面"的关注。
2. **被掩码屏蔽的注意力**：在因果掩码或填充掩码中，被设置为 -$\infty$ 的注意力分数，经过 Softmax 后权重为 0，这部分也可以被通俗地理解为"负面（不允许）的注意力"，即结构上被强制禁止关注的位置。
3. **抑制性 / 负相关性**：从更广义的角度，如果将注意力分数本身视为一种"相关性得分"，那么较大的负分数在数学上代表 Query 与 Key 的向量方向相反（点积为负），暗示两者在语义上可能存在某种"负相关"或"对立"关系，但经过 Softmax 后这种负相关只会体现为极低的权重，而不会出现"负的注意力权重"（因为 Softmax 输出始终非负）。

总的来说，Transformer 的注意力权重在数学上始终是非负的（因为经过了 Softmax），不存在真正意义上的"负权重"，"负面注意力"更多是对低权重或被掩码部分的一种形象化描述。

</details>

**19. 如何理解 Transformer 模型中的"编码器-解码器注意力"机制？**

<details>
<summary>💡 查看答案</summary>

编码器-解码器注意力（Encoder-Decoder Attention，也称为 Cross-Attention）是解码器中第二个注意力子层，其核心作用是让解码器在生成每个输出 token 时，能够动态地"查阅"输入序列中的相关信息。

具体来说：
- **Query（查询）** 来自解码器当前层前一个子层（掩码自注意力）的输出，代表"当前需要生成什么内容"。
- **Key（键）和 Value（值）** 都来自编码器最后一层的输出，代表"输入序列中包含哪些信息"。

通过计算 Query 与所有 Key 的相似度（点积后经过 Softmax 归一化），得到一组注意力权重，再用这些权重对 Value 进行加权求和，得到融合了输入序列相关信息的上下文向量。

这种机制本质上实现了源序列和目标序列之间的"软对齐"（soft alignment），类似于传统 Seq2Seq 模型中的注意力机制，例如在机器翻译中，生成目标语言的某个词时，模型会自动关注源语言句子中与之对应的词或短语。

</details>

**20. 什么是序列到序列模型？**

<details>
<summary>💡 查看答案</summary>

序列到序列模型将输入序列转换为相应的输出序列，常用于：
- 机器翻译：英语 $\rightarrow$ 法语。
- 文本摘要：长文本 $\rightarrow$ 摘要。
- 问答：上下文 $\rightarrow$ 答案。

**组件**：
- 编码器：处理输入并生成表示。
- 解码器：将表示转换为所需的输出。

**示例**：基于 Transformer 的模型如 T5 和基于 RNN 的模型如带注意力机制的 Seq2Seq。

</details>

**21. 什么是掩码语言建模（MLM）？**

<details>
<summary>💡 查看答案</summary>

MLM 涉及随机掩码输入文本中的一些标记，并让模型根据周围上下文进行预测。由 BERT（基于 Transformer 的双向编码器表示）推广。

**目标**：通过利用双向上下文学习语言中的双向依赖关系。

**示例**：
- 输入：“The [MASK] is blue.”
- 预测：“The sky is blue.”

</details>

**22. 什么是下一句预测（NSP）？**

<details>
<summary>💡 查看答案</summary>

NSP 是一种预训练目标，用于教模型理解句子之间的关系。模型预测给定句子 B 是否自然地跟在句子 A 之后。

**示例（BERT）**：
- 输入：
    - 句子 A：“I love books.”
    - 句子 B：“They expand my knowledge.”
- 模型输出：True.

</details>

**23. 自回归模型和掩码模型有何不同？**

<details>
<summary>💡 查看答案</summary>

- **自回归模型（例如 GPT）**：逐个生成文本，使用之前生成的标记作为上下文。
    - **示例**：预测序列中的下一个词。
    - **优势**：适合文本生成。
- **掩码模型（例如 BERT）**：使用双向上下文预测序列中的掩码标记。
    - **优势**：适合理解任务如分类或问答。

</details>

**24. 请详细解释一下 Transformer 模型中的自注意力机制是如何工作的？它为什么比 RNN 更适合处理长序列？**

<details>
<summary>💡 查看答案</summary>

自注意力（Self-Attention）机制是Transformer模型的核心，它使得模型能够动态地衡量输入序列中不同单词之间的重要性，并据此生成每个单词的上下文感知表示。


工作原理如下：


1. 生成Q, K, V向量： 对于输入序列中的每一个词元（token）的嵌入向量，我们通过乘以三个可学习的权重矩阵 $W^Q, W^K, W^V$ ，分别生成三个向量：查询向量（Query, Q）、键向量（Key, K）和值向量（Value, V）。

- Query (Q): 代表当前词元为了更好地理解自己，需要去“查询”序列中其他词元的信息。
- Key (K): 代表序列中每个词元所“携带”的，可以被查询的信息标签。
- Value (V): 代表序列中每个词元实际包含的深层含义。

1. 计算注意力分数： 为了确定当前词元（由Q代表）应该对其他所有词元（由K代表）投入多少关注，我们计算当前词元的Q与其他所有词元的K的点积。这个分数衡量了两者之间的相关性。

\[Score(Q_i, K_j) = Q_i \cdot K_j\]


1. 缩放（Scaling）： 将计算出的分数除以一个缩放因子 $\sqrt{d_k}$（ $d_k$ 是K向量的维度）。这一步是为了在反向传播时获得更稳定的梯度，防止点积结果过大导致Softmax函数进入饱和区。

\[\frac{Q \cdot K^T}{\sqrt{d_k}}\]


1. Softmax归一化： 将缩放后的分数通过一个Softmax函数，使其转换为一组总和为1的概率分布。这些概率就是“注意力权重”，表示在当前位置，每个输入词元所占的重要性。

\[AttentionWeights = softmax\left(\frac{Q K^T}{\sqrt{d_k}}\right)\]


1. 加权求和： 最后，将得到的注意力权重与每个词元对应的V向量相乘并求和，得到最终的自注意力层输出。这个输出向量融合了整个序列的上下文信息，且权重由模型动态学习得到。

\[Output = AttentionWeights \cdot V\]


为什么比RNN更适合处理长序列？


1. 并行计算能力： 自注意力机制在计算时，可以一次性处理整个序列，计算所有位置之间的关联，是高度并行的。而RNN（包括LSTM、GRU）必须按照时间顺序依次处理每个词元，无法并行化，导致处理长序列时速度非常慢。
2. 解决长距离依赖问题： 在自注意力中，任意两个位置之间的交互路径长度都是O(1)，因为可以直接计算它们的注意力分数。而在RNN中，序列首尾两个词元的信息传递需要经过整个序列的长度，路径为O(N)，这极易导致梯度消失或梯度爆炸，使得模型难以捕捉长距离的依赖关系。

</details>

**25. 你知道MHA，MQA，GQA的区别吗？详细解释一下。**

<details>
<summary>💡 查看答案</summary>

MHA、MQA和GQA是Transformer模型中三种不同的注意力机制变体，它们的主要区别在于如何组织和共享Query、Key和Value的“头”（Head），核心目标是在模型效果和推理效率（特别是显存占用）之间做出不同的权衡。


	extbf{1. MHA (Multi-Head Attention)}

这是原始Transformer论文中提出的标准注意力机制。

- 工作原理：

1. 将输入的Q、K、V向量分别通过 $N$ 个独立的线性变换，得到 $N$ 组不同的 $Q_i, K_i, V_i$ 头（ $i=1, ..., N$ ）。
2. 这 $N$ 组头在各自的子空间中并行地计算注意力（Scaled Dot-Product Attention）。
3. 将 $N$ 个头计算得到的输出向量拼接（Concatenate）起来。
4. 最后通过一个线性变换将拼接后的向量映射回原始维度。

- 结构： $N$ 个Query头， $N$ 个Key头， $N$ 个Value头。
- 优点： 效果最好，模型能力最强。每个头可以在不同的表示子空间中学习到不同的信息。
- 缺点： 推理成本高。在自回归生成任务中，需要缓存每一层的Key和Value（即KV Cache），MHA的KV Cache大小与头的数量$N$成正比，显存占用非常大，限制了长序列的生成。

	extbf{2. MQA (Multi-Query Attention)}

为了解决MHA在推理时的显存瓶颈而被提出。

- 工作原理：

1. 与MHA一样，有 $N$ 个独立的Query头。
2. 核心区别： 所有的 $N$ 个Query头共享同一个Key头和同一个Value头。

- 结构： $N$ 个Query头，1个Key头，1个Value头。
- 优点： 极大地降低了推理成本。KV Cache的大小不再依赖于头的数量 $N$ ，相比MHA减小了 $N$ 倍，显著降低了显存占用，并加快了推理速度。
- 缺点： 可能会导致模型性能的下降。因为所有Query头被迫从同样的一组Key和Value中提取信息，模型的表达能力受到了一定的限制。

	extbf{3. GQA (Grouped-Query Attention)}

GQA是MHA和MQA之间的一个折中方案，旨在平衡性能和效率。

- 工作原理：

1. 将 $N$ 个Query头分成 $G$ 组。
2. 核心区别： 每组内的Query头共享一个Key头和一个Value头。总共有 $G$ 个Key头和 $G$ 个Value头。

- 结构： $N$ 个Query头，G个Key头，G个Value头。（通常 $1 < G < N$ ）。
- 说明：
- 当 $G=N$ 时，GQA等价于MHA。
- 当 $G=1$ 时，GQA等价于MQA。
- 优点： 在推理效率上远超MHA，同时在模型性能上优于MQA。它提供了一个灵活的旋钮，可以根据具体需求在效率和效果之间进行调整。Llama 2等模型就采用了GQA。

总结： | 特性         | MHA (Multi-Head Attention) | MQA (Multi-Query Attention) | GQA (Grouped-Query Attention) | | :----------- | :------------------------- | :-------------------------- | :---------------------------- | | 结构     | N个Q头, N个K头, N个V头     | N个Q头, 1个K头, 1个V头      | N个Q头, G个K头, G个V头        | | 模型质量 | 最高                       | 可能下降                    | 接近MHA，优于MQA              | | 推理效率 | 最低 (KV Cache大)          | 最高 (KV Cache小)           | 居中，远好于MHA               | | 应用     | BERT, GPT-3                | PaLM                        | Llama 2, Mixtral              |

</details>

**26. MHA 后续的计算优化工作**

<details>
<summary>💡 查看答案</summary>

针对标准 MHA 在推理阶段 KV Cache 显存占用过大的问题：

1. **MQA（Multi-Query Attention）**：所有 Query head 共享同一组 Key 和 Value head，大幅减少 KV Cache，但可能轻微影响模型质量。
2. **GQA（Grouped-Query Attention）**：MHA 和 MQA 的折中——将 Query head 分成若干组，每组共享一组 K、V head，被 LLaMA 2/3、Mistral 等广泛采用。
3. **MLA（Multi-head Latent Attention）**：DeepSeek 提出，通过对 K、V 进行低秩联合压缩，在大幅压缩 KV Cache 显存的同时保持模型表达能力。
4. **Sliding Window Attention**：每个 token 只 attend 到固定窗口大小内的历史 token，将复杂度从 O($n^2$) 降为 O($n \cdot w$)。
5. **Flash Attention**：不改变注意力的数学定义，通过分块计算 + 在线 softmax + 减少 HBM 读写的算子级优化，大幅降低显存占用并提升计算速度。

</details>

**27. Attention 几种常见的计算方式**

<details>
<summary>💡 查看答案</summary>

1. **标准自注意力**：Attention(Q,K,V) = softmax(QK$^T$/$\sqrt{d_k}$) V，复杂度 O($n^2d$)。
2. **多头注意力（MHA）**：将 Q、K、V 投影到多个低维子空间分别计算注意力后拼接。
3. **MQA / GQA**：通过共享 K、V head 减少推理时显存/计算开销。
4. **因果（Masked）自注意力**：对未来位置的 attention score 加上 $-\infty$ 掩码。
5. **交叉注意力（Cross-Attention）**：Q 来自一个序列，K、V 来自另一个序列。
6. **稀疏注意力（Sparse Attention）**：只计算部分位置对的注意力，降低复杂度。
7. **线性注意力（Linear Attention）**：通过核函数近似或改变计算顺序，将复杂度降为 O($n$)。
8. **FlashAttention**：精确计算标准注意力，但通过分块和算子融合优化 IO。
9. **MLA**：通过低秩压缩 KV 来优化显存的注意力变体。

</details>

**28. 推导一下 softmax 中为啥要除以根号 $d$**

<details>
<summary>💡 查看答案</summary>

**定义与假设**：$Q$ 和 $K$ 是两个独立的随机矩阵，维度均为 $n \times d_k$，每个元素独立同分布，均值为 0，方差为 1。

计算 $(QK^T)_{ij} = \sum_{l=1}^{d_k} q_{il} k_{jl}$：
- 均值：$E[(QK^T)_{ij}] = 0$
- 方差：$D[(QK^T)_{ij}] = d_k$

除以 $\sqrt{d_k}$ 后：

$$
    D\left[\frac{(QK^T)_{ij}}{\sqrt{d_k}}\right] = \frac{d_k}{d_k} = 1
$$


**直观解释**：未缩放时 $QK^T$ 的方差随 $d_k$ 线性增长，导致 Softmax 输入值过大，输出接近 one-hot 分布，梯度消失。缩放后方差稳定为 1，Softmax 输入分布更平滑，梯度传播更稳定。

</details>

**29. 大模型中加入 dropout 的吗？**

<details>
<summary>💡 查看答案</summary>

观察 LLaMA3 源码发现没有 dropout，观察 Qwen 发现有但 dropout 设置为 0。

这一现象并非偶然，而是大模型训练实践不断演进的结果。本节从理论动机、实践经验和源码层面对这一问题进行详细分析。

#### Dropout 的基本原理与初衷

Dropout 由 Srivastava 等人于 2014 年提出，其核心思想是在训练过程中以概率 $p$ 随机将神经元输出置零，从而：

- 防止神经元之间产生过度的共适应（co-adaptation），起到正则化作用；
- 可以理解为对指数级数量的子网络进行集成学习；
- 在中小规模模型和训练数据有限的场景下，显著缓解过拟合问题。

在早期的 Transformer 模型（如原始 *Attention is All You Need* 中的实现）中，Dropout 被广泛应用于注意力权重、残差连接输出、前馈层等位置，典型值为 $p = 0.1$。

#### 大模型训练中 Dropout 逐渐被弃用的原因

随着模型规模和数据量的急剧扩大，dropout 的必要性大幅下降，主要原因如下：

1. **海量数据本身提供了足够的正则化**\\
    大模型通常在数千亿乃至数万亿 token 上进行训练，数据的多样性和规模使得模型很难在训练集上过拟合。数据增强与多样性本身扮演了正则化的角色，dropout 的额外正则化效果变得微乎其微。

2. **Dropout 损害训练稳定性与收敛速度**\\
    在大规模训练中，dropout 引入的随机噪声会干扰梯度估计，导致：
    - 损失曲线震荡加剧，收敛变慢；
    - 与学习率调度、梯度裁剪等技术产生复杂的相互影响；
    - 在混合精度训练（FP16/BF16）下，随机置零可能加剧数值不稳定性。

3. **训练与推理的不一致性**\\
    Dropout 在训练阶段激活、推理阶段关闭，这种不一致性在大模型场景下会引入额外的工程复杂度，尤其是在分布式训练和模型并行场景中，随机数状态的同步本身就是一个挑战。

4. **其他正则化手段已足够**\\
    现代大模型通常综合采用以下技术替代 dropout 的正则化作用：
    - **Weight Decay**：通过 L2 正则化约束参数量级，是目前最主流的正则化手段；
    - **梯度裁剪（Gradient Clipping）**：防止梯度爆炸，提升训练稳定性；
    - **数据多样性**：大规模、高质量、多领域的训练语料本身即为强正则；
    - **学习率调度**：Warmup + Cosine Decay 等策略有助于平稳收敛。

#### LLaMA3 源码分析：完全移除 Dropout

在 LLaMA3 的官方实现中，模型结构的各个位置均未使用 dropout。以其核心模块为例：

\begin{lstlisting}[style=promptstyle]
	# LLaMA3 Attention 模块（简化）
	class Attention(nn.Module):
	def forward(self, x, ...):
	...
	# 注意力权重直接 softmax，无 dropout
	scores = torch.softmax(scores, dim=-1)
	output = torch.matmul(scores, values)
	return output

	# LLaMA3 FeedForward 模块（简化）
	class FeedForward(nn.Module):
	def forward(self, x):
	# SwiGLU 激活，无 dropout
	return self.w2(F.silu(self.w1(x)) * self.w3(x))
\end{lstlisting}

Meta 官方的训练报告（LLaMA 2 Tech Report）也明确指出，在预训练阶段不使用 dropout，仅依赖 weight decay 和 AdamW 优化器进行正则化。

#### Qwen 源码分析：保留接口但设置为 0

Qwen 系列模型的实现中保留了 dropout 的相关参数接口，但在默认配置下将其设置为 0，即实际上等价于不使用 dropout：

\begin{lstlisting}[style=promptstyle]
	# Qwen 配置文件示例（config.json）
	{
		"attention_dropout": 0.0,
		"hidden_dropout": 0.0,
		"emb_dropout_prob": 0.0,
		...
	}

	# Qwen Attention 模块（简化）
	class QwenAttention(nn.Module):
	def __init__(self, config):
	self.attn_dropout = nn.Dropout(config.attention_dropout)  # 实际为 Dropout(0.0)

	def forward(self, ...):
	attn_weights = self.attn_dropout(attn_weights)  # p=0，等价于恒等映射
	...
\end{lstlisting}

保留 dropout 接口的好处在于：

- **灵活性**：下游微调（Fine-tuning）时可按需启用 dropout 以防止过拟合，无需修改模型代码；
- **实验对比**：研究人员可以方便地通过配置文件开启 dropout 进行消融实验；
- **继承性**：基于早期 Transformer 代码库演进而来，保留接口有助于向后兼容。

#### 微调阶段是否需要 Dropout？

与预训练不同，**在微调（Fine-tuning）阶段适当引入 dropout 往往是有益的**，尤其是在以下场景：

- **数据量较少的 SFT（Supervised Fine-Tuning）**：训练样本较少时，模型容易过拟合，此时 $p = 0.05 \sim 0.1$ 的 dropout 可提供一定的正则化保护；
- **LoRA 等参数高效微调**：LoRA 适配器参数量少，引入少量 dropout 有助于防止适配器过拟合；
- **指令微调（Instruction Tuning）**：当指令数据分布较窄时，dropout 有助于保留模型的泛化能力。

#### 小结

- **预训练阶段**：现代大模型普遍不使用 dropout（如 LLaMA3），或将其设为 0（如 Qwen）。根本原因在于：海量训练数据已提供足够正则化，而 dropout 反而会损害训练效率与稳定性。
- **接口保留的意义**：Qwen 等模型保留 dropout 接口，主要是为了微调阶段的灵活性和实验便利性，并非预训练中实际使用。
- **微调阶段**：视数据规模和任务特性，可适当启用较小的 dropout 率（$p \leq 0.1$）以防止过拟合。

</details>

**30. 大模型中不用 dropout 的主要原因**

<details>
<summary>💡 查看答案</summary>

在上一节中我们观察到 LLaMA3 完全移除了 dropout、Qwen 将其设置为 0 的现象。本节从更深层次归纳大模型弃用 dropout 的核心原因。

1. **引入噪声导致训练不稳定**

    Dropout 在前向传播中以概率 $p$ 随机屏蔽神经元，这等价于在梯度流中引入随机乘性噪声。对于参数量在数十亿乃至数千亿量级的大模型而言，其网络层数极深（如 LLaMA3-70B 有 80 层 Transformer），随机噪声会在反向传播过程中逐层累积放大，具体表现为：
    - 损失曲线出现明显抖动，难以平稳下降；
    - 梯度方差增大，对学习率的选取更加敏感；
    - 与梯度裁剪、学习率 Warmup 等稳定性手段产生负向干扰，整体训练鲁棒性下降。
    相比之下，去掉 dropout 后梯度信号更加干净，配合 AdamW + 梯度裁剪可以实现更稳定的收敛。

2. **计算资源开销增加，训练效率降低**

    Dropout 看似只是"置零"操作，但在工程实现层面引入了不可忽视的额外开销：
    - **随机数生成**：每个训练步骤需要为每一层生成与激活值等形状的随机掩码（mask），在大批量、大序列长度下，随机数生成本身会消耗显著的计算时间；
    - **内存占用**：随机掩码需要在前向传播中保存，以供反向传播使用，增加了显存压力；
    - **分布式同步复杂度**：在张量并行（Tensor Parallelism）和流水线并行（Pipeline Parallelism）场景下，各设备上的随机数状态需要保持一致，否则会引入额外的不确定性，增加了工程复杂度。
    在数万块 GPU 长达数月的预训练场景下，上述开销的累积效应不可忽视。

3. **去掉 Dropout 不会导致效果变差**

    现代大模型已集成多种在结构和数据层面的正则化手段，使得 dropout 的正则化贡献变得可有可无：
    - **Pre-Norm（RMSNorm）**：将层归一化置于残差连接之前，显著提升了深层网络的训练稳定性，本身就具备一定的正则化效果；
    - **残差连接（Residual Connection）**：保证梯度直通路径，缓解深层网络的退化问题；
    - **多头注意力（MHA）与分组查询注意力（GQA/MQA）**：多头结构本身的参数共享和信息整合具有隐式正则化效果；
    - **Weight Decay**：通过 L2 惩罚约束参数量级，是最主流的显式正则化手段；
    - **大规模多领域数据**：在数万亿 token 的多源异构数据上训练，数据本身的多样性是最强的正则化来源，模型几乎不会对训练数据过拟合。
    上述因素的综合作用，使得去掉 dropout 后模型的泛化能力不减反升（因为少了随机噪声对有效特征学习的干扰）。

4. **大量实验证明 Dropout 在大模型中效果不显著甚至有害**

    学术界和工业界的大量消融实验表明，dropout 的收益与模型规模呈负相关：
    - 在小模型（如参数量在百万级的 LSTM、小型 Transformer）中，dropout 能显著提升验证集精度，防止过拟合；
    - 随着模型规模增大，dropout 带来的收益逐渐递减，当参数量超过数亿时，dropout 几乎不带来精度提升；
    - 在超大规模模型（如 GPT-3、LLaMA 系列）的训练中，使用 dropout 反而会拖慢收敛、降低最终性能；
    - PaLM、Chinchilla 等技术报告均明确指出预训练阶段不使用 dropout，这一结论已在业界形成共识。
    其根本原因在于：dropout 的设计初衷是在"参数多、数据少"的过拟合场景下发挥作用，而大模型的训练场景恰恰相反——数据极为充裕，模型反而处于欠拟合状态，此时引入噪声只会适得其反。





![大模型在预训练与微调阶段启用 Dropout 的策略对比](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/fig_dropout_finetune.png)

</details>

**31. BERT 的三个 Embedding 为什么可以进行相加？**

<details>
<summary>💡 查看答案</summary>

BERT 的输入表示由三个部分叠加而成：Token Embedding（词嵌入）、Segment Embedding（句段嵌入）和 Position Embedding（位置嵌入）。三者相加而非拼接（Concat），这一设计选择看似反直觉，实则有多重角度的合理性支撑。

1. **信号处理角度**

    从信号处理的视角来看，一个向量可以被理解为携带特定信息的信号。多个信号叠加（superposition）是信号处理领域的基本操作——例如，AM/FM 广播中多个频道的信号可以在同一介质中叠加传播，接收端的解码器能够从混合信号中分离出各自的信息。

    类比到 BERT 中：Token、Segment、Position 三个 Embedding 叠加后形成一个混合信号，后续的多层 Transformer（本质上是一个极强的非线性函数近似器）有能力从这个混合信号中解耦出各维度的信息，并分别利用。邱锡鹏教授在《神经网络与深度学习》中也持类似观点：只要后续网络足够强大，相加的信号就可以被有效地分解和利用。

2. **One-hot 等价角度**

    苏剑林从代数角度给出了一个更为严格的等价性证明，说明"三个 Embedding 相加"与"三个 One-hot 向量拼接后过线性层"在数学上是等价的。

    具体推导如下：设词表大小为 $V$，句段类别数为 $S$，最大序列长度为 $L$，Embedding 维度为 $d$。对于第 $i$ 个位置的 token，定义：
    - Token 的 One-hot 向量 $\mathbf{t} \in \mathbb{R}^V$；
    - Segment 的 One-hot 向量 $\mathbf{s} \in \mathbb{R}^S$；
    - Position 的 One-hot 向量 $\mathbf{p} \in \mathbb{R}^L$。
    将三者拼接得到 $\mathbf{x} = [\mathbf{t}; \mathbf{s}; \mathbf{p}] \in \mathbb{R}^{V+S+L}$，过一个线性层 $W \in \mathbb{R}^{d \times (V+S+L)}$（无偏置）：
    \[
    W\mathbf{x} = [W_t \mid W_s \mid W_p] \begin{bmatrix} \mathbf{t} \\ \mathbf{s} \\ \mathbf{p} \end{bmatrix} = W_t \mathbf{t} + W_s \mathbf{s} + W_p \mathbf{p}
    \]
    其中 $W_t \mathbf{t}$、$W_s \mathbf{s}$、$W_p \mathbf{p}$ 分别就是查表得到的 Token Embedding、Segment Embedding 和 Position Embedding。因此：
    \[
    Concat(\mathbf{t}, \mathbf{s}, \mathbf{p}) \xrightarrow{Linear} Emb_token + Emb_seg + Emb_pos
    \]
    两种操作在参数等价的条件下完全等价，说明"相加"并不是一种信息损失的操作，而是 Concat + Linear 的一种紧凑实现。

3. **信息融合角度**

    从表示学习的角度理解，三个 Embedding 分别生活在三个不同的语义子空间中：
    - Token Embedding 编码的是词汇的语义信息（词义、词性等）；
    - Segment Embedding 编码的是句子归属信息（属于句子 A 还是句子 B）；
    - Position Embedding 编码的是序列位置信息（token 在序列中的绝对位置）。
    三者相加，等价于将三个子空间中的向量通过各自的线性嵌入矩阵映射到同一个 $d$ 维统一特征空间，再在该空间中进行逐元素叠加融合。这种融合方式要求三类信息在同一空间中能够以互补而非冲突的方式共存，而 BERT 的训练过程（MLM + NSP）会自动驱动模型学习到这样一种协调的表示。

    此外，相较于 Concat 操作，相加操作不会增加 Embedding 的维度，在保持表达能力的同时显著降低了参数量和计算量，是一种在表达能力与计算效率之间取得良好平衡的设计选择。

#### 三种角度的对比与联系

- **信号处理角度**强调的是网络的解耦能力，是一种直觉性的解释；
- **One-hot 等价角度**提供了严格的代数证明，说明相加不损失信息表达能力；
- **信息融合角度**从表示空间的几何意义出发，解释了相加操作的内在含义。

三种角度相互补充，共同说明"Embedding 相加"是一种数学上合理、工程上高效、模型上可学习的设计。





![BERT 的三个输入 Embedding 相加机制示意图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/fig_bert_embeddings.png)

</details>

**32. 位置编码是如何使模型具有顺序感知能力的？**

<details>
<summary>💡 查看答案</summary>

自注意力机制本质上是对序列中所有 token 进行加权求和，这种操作对输入顺序是不敏感的（即打乱输入顺序，输出结果的集合不变，只是顺序也随之打乱），这意味着模型本身无法区分"猫追狗"和"狗追猫"这种顺序差异。

为了解决这个问题，原始 Transformer 论文采用了**正余弦位置编码**：对每个位置 pos 和每个维度 i，使用不同频率的 sin 和 cos 函数计算出一个固定的向量，公式大致为：

\begin{verbatim}
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
\end{verbatim}

这个位置向量与词嵌入向量直接相加，作为模型的实际输入。由于不同位置生成的编码向量不同，且具有一定的相对位置关系（可以通过线性变换相互转换），模型就能够通过这些附加信息感知 token 在序列中的相对和绝对位置。

除了固定的三角函数编码，后续也出现了**可学习的位置编码**（如 BERT）以及**相对位置编码**（如 Transformer-XL、T5、RoPE 旋转位置编码等），它们在长序列建模和外推能力上通常表现更好。

</details>

**33. 什么是位置编码？在 Transformer 中，为什么它是必需的？请列举至少两种实现方式。**

<details>
<summary>💡 查看答案</summary>

什么是位置编码？ 位置编码（Positional Encoding, PE）是一个与词嵌入维度相同的向量，其目的是向模型注入关于词元在输入序列中绝对或相对位置的信息。它会与词元的词嵌入（Token Embedding）相加，然后一同输入到Transformer的底层。


为什么它是必需的？ Transformer的核心机制——自注意力，在计算时处理的是一个集合（Set）而非序列（Sequence）。它本身不包含任何关于词元顺序的信息，是 置换不变（Permutation-invariant） 的。这意味着，如果打乱输入序列中词元的顺序，自注意力层的输出也会相应地被打乱，但每个词元自身的输出向量（在不考虑softmax归一化的情况下）是相同的。这显然不符合自然语言的特性，因为语序至关重要（例如“我打你”和“你打我”含义完全相反）。因此，必须通过一种外部机制，将位置信息显式地提供给模型，这就是位置编码的作用。


至少两种实现方式：


1. 正弦/余弦位置编码（Sinusoidal Positional Encoding）：

这是原始Transformer论文《Attention Is All You Need》中使用的方法。它使用不同频率的正弦和余弦函数来生成位置编码，其公式如下：

\[PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{model}})\]


\[PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{model}})\]


其中， $pos$ 是词元在序列中的位置， $i$ 是编码向量中的维度索引， $d_{model}$ 是嵌入维度。

- 优点：
- 可外推性： 能够处理比训练中最长序列还要长的序列。
- 相对位置信息： 模型可以轻易地学习到相对位置关系，因为对于任何固定的偏移量 $k$ ， $PE_{pos+k}$ 都可以表示为 $PE_{pos}$ 的一个线性函数，这使得模型更容易捕捉相对位置的依赖。

1. 可学习的绝对位置编码（Learned Absolute Positional Encoding）：

这种方法将位置编码视为模型参数的一部分，通过训练学习得到。具体来说，会创建一个形状为 `(max_sequence_length, embedding_dimension)` 的位置编码矩阵。在处理序列时，根据每个词元的位置索引，从这个矩阵中查找对应的编码向量，并加到词嵌入上。BERT和GPT-2等模型采用了这种方式。

- 优点： 模式更加灵活，可以让模型自己学习出最适合数据的位置表示。
- 缺点： 无法泛化到超过预设 `max_sequence_length` 的长度。如果需要处理更长的序列，就需要对位置编码进行微调或采用其他策略。

</details>

**34. 请你详细介绍ROPE，对比绝对位置编码它的优劣势分别是什么？**

<details>
<summary>💡 查看答案</summary>

RoPE (Rotary Position Embedding) 介绍 RoPE，全称旋转位置编码，是目前大语言模型（如Llama系列、Qwen等）中最主流的位置编码方案之一。它是一种将位置信息融入自注意力机制的创新方法。


其核心思想是：通过向量旋转的方式，将绝对位置信息编码到Query和Key向量中，从而使得模型在计算注意力分数时，能够自然地利用相对位置信息。


工作原理： RoPE不再像传统位置编码那样直接将位置向量加到词嵌入上。它的操作发生在生成Q和K向量之后、计算注意力分数之前：

1. 维度分组： 将Q和K向量的 $d$ 维特征两两一组，视为 $d/2$ 个二维向量。
2. 构造旋转矩阵： 对于序列中的位置 $m$，构造一个与位置相关的旋转矩阵 $R_m$。这个矩阵在二维空间中表示一个旋转操作。
3. 旋转Q和K： 将每个二维向量组通过对应的旋转矩阵 $R_m$ 进行旋转。

数学上，这个过程等价于将每个二维向量 $(x_m, x_{m+1})$ 看作一个复数，然后乘以一个复数 $e^{im\theta}$，其中 $m$ 是位置， $\theta$ 是一个预设的、与维度相关的常数。这个操作只会改变向量的相位（方向），而不改变其模（长度）。


关键特性： RoPE的巧妙之处在于，经过旋转后的两个位置 $m$ 和 $n$ 的Query向量 $q_m$ 和Key向量 $k_n$ 进行点积运算时，其结果只与它们的相对位置 $(m-n)$ 有关，而与它们的绝对位置 $m$ 和 $n$ 无关。这使得自注意力机制天然地具备了对相对位置的感知能力。


对比绝对位置编码的优劣势：


RoPE的优势：

1. 内置相对位置建模： 这是其最大的优势。RoPE使得注意力分数直接依赖于词元间的相对距离，这更符合自然语言中语法和语义依赖通常是相对的这一特性。
2. 良好的外推能力： 由于其数学性质，RoPE在处理比训练时更长的序列时表现出色，具有很强的长度泛化能力，这也是长序列LLM偏爱它的重要原因。
3. 不引入额外可训练参数： RoPE是一种函数式的、固定的编码方式，不需要像可学习位置编码那样占用模型参数。
4. 随着距离增加，依赖性衰减： 旋转的性质使得距离越远的词元，其内积关系会呈现周期性的衰减，符合语言中距离越远相关性越弱的直觉。

RoPE的劣势：

1. 理论理解相对复杂： 其背后的数学原理（复数、欧拉公式、旋转矩阵）比直接相加的绝对位置编码更抽象。
2. 对绝对位置信息的表征可能较弱： 虽然RoPE从绝对位置导出，但其在注意力机制中的核心作用是体现相对位置。对于那些强依赖绝对位置信息的特定任务（例如，判断一个词是否在句子开头），它的效果可能不如直接使用绝对位置编码直观。

</details>

**35. LLM 中的位置编码是什么？**

<details>
<summary>💡 查看答案</summary>

Transformer 缺乏固有的序列感，因为它们并行处理输入标记。位置编码通过引入序列信息来解决这一问题：
- **工作原理**：位置编码使用数学函数（例如正弦和余弦）根据其在序列中的位置为每个标记分配一个独特的编码。
- **为什么重要**：它帮助模型区分“The cat chased the dog”和“The dog chased the cat”。

</details>

**36. 目前大模型中的位置编码有哪些？**

<details>
<summary>💡 查看答案</summary>

在大模型中，位置编码主要分为两大派：绝对位置编码和相对位置编码。主流的几种位置编码如下：

**正弦曲线（Sinusoidal）位置编码**：这是 Transformer 原始论文中提出的位置编码方式。它通过正弦和余弦函数的组合为每个位置创建编码，波长呈几何级数排列，使得每个位置的编码都是独特的，并且能够捕获位置之间的相对关系。公式如下：

$$
    PE_{(pos,2i)} = \sin(pos/10000^{2i/d_{model}}), \quad PE_{(pos,2i+1)} = \cos(pos/10000^{2i/d_{model}})
$$

其中 $pos$ 表示位置，$d_{model}$ 代表 embedding 的维度，$2i, 2i+1$ 代表 embedding 不同位置的索引。

优点：实现简单，效率高，易用性好，有一定的理论支撑。缺点：绝对位置编码带来的灵活性差；外推性较差，难以对超出预训练长度的文本进行很好的外推；难以表示有效的相对位置间的关系。

**旋转位置编码（RoPE）**：这是一种相对位置编码，通过旋转向量的方式来编码位置信息。RoPE 在特定任务中表现出独特的优势，尤其是在处理具有对称性或周期性特征的数据时。

$$
    \langle f_q(x_m,m), f_k(x_n,n) \rangle = g(x_m, x_n, m-n)
$$


优点：能保证相对位置关系；良好的外推性；可以和线性注意力机制兼容。缺点：实现上稍微复杂；外推到超长序列也存在性能下降问题（超过 20% 的预训练长度）。

**相对位置编码（ALiBi）**：这是一种微调 Attention 结构的方法，使其能够分辨不同位置的 Token。ALiBi 并没有显式的在网络中加入编码，唯一的修改是在 qk 点积之后，加入了一个静态的不可学习的 bias：

$$
    softmax(q_i K^T + m \cdot [-(i-1), \ldots, -2, -1, 0])
$$

$m$ 是一个 head-specific 的超参，对于 $n$ 个 head 而言，步长一般为 $2^{-8/n}$。



![ALiBi 位置编码示意图](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/alibi_diagram.png)



优点：实现简单，减少了模型 embedding 参数，效率高，具有良好外推性。缺点：外推到超长序列也存在性能下降问题。

上面介绍了 3 个主流的位置编码方式，分别从 attention 前（正弦编码）、attention 中（将编码融入到 attention 计算中）以及 attention 后（算完 qk 乘积后加上 bias）的三种方式。

</details>

**37. 手写一下正弦编码和旋转位置编码的代码？**

<details>
<summary>💡 查看答案</summary>

**正弦位置编码**核心代码：
\begin{lstlisting}[language=Python]
position = torch.arange(0, max_len, dtype=torch.float32).unsqueeze(1)
two_i = torch.arange(0, d_model, 2, dtype=torch.float32)
div_term = torch.exp(two_i * -(math.log(10000.0) / d_model))
encodings = torch.zeros(max_len, d_model)
encodings[:, 0::2] = torch.sin(position * div_term)
encodings[:, 1::2] = torch.cos(position * div_term)
\end{lstlisting}

**旋转位置编码**核心代码：
\begin{lstlisting}[language=Python]
# 生成旋转矩阵
def precompute_freqs_cis(dim: int, seq_len: int, theta: float = 10000.0):
    freqs = 1.0 / (theta ** (torch.arange(0, dim, 2)[: (dim // 2)].float() / dim))
    t = torch.arange(seq_len, device=freqs.device)
    freqs = torch.outer(t, freqs).float()  # 计算m * theta
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)  # 复数向量
    return freqs_cis

# 旋转位置编码计算
def apply_rotary_emb(xq, xk, freqs_cis):
    xq_ = xq.float().reshape(*xq.shape[:-1], -1, 2)
    xk_ = xk.float().reshape(*xk.shape[:-1], -1, 2)
    xq_ = torch.view_as_complex(xq_)
    xk_ = torch.view_as_complex(xk_)
    xq_out = torch.view_as_real(xq_ * freqs_cis).flatten(2)
    xk_out = torch.view_as_real(xk_ * freqs_cis).flatten(2)
    return xq_out.type_as(xq), xk_out.type_as(xk)
\end{lstlisting}

</details>

**38. ALiBi 真的有长序列依赖建模的能力吗？**

<details>
<summary>💡 查看答案</summary>

ALiBi 的线性 bias 决定了在一层 attention 中不可能看到太远的距离。之所以 ALiBi 有外推能力，其实是因为它本身是一个近似的局部滑动注意力，依靠多层叠加的 attention 来感知到大于一层 attention 长度的信息，这种长度依赖的扩展是与网络层数成正比的，理论长度是有限的。其实 RoPE 也可以做局部滑动 mask 来实现类似的长度扩展。所以，就这一点来看的话，ALiBi 不太像是一种位置编码，更像一种 attention 后处理的 trick，ALiBi 所施加的局部注意力，起到的是增强局域性的作用，贴合语言模型任务本身的特性。



![ALiBi 在算完 attention 后减去的矩阵（局部 attention 平滑）](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/alibi_attention_matrix.png)

</details>

**39. 大模型推理中超出训练长度的外推方式有哪些？**

<details>
<summary>💡 查看答案</summary>

主要的几个做法为 ALiBi、内插法、NTK-Aware 方法、Yarn 方法。

长度外推（Length Generalization）研究的是如何预训练的时候使用较小的长度，但在外推的时候能够泛化到更大的长度上去。好的外推性能表现就是在泛化到超长序列的时候，相关指标不会出现较大的下降，模型表现依然很稳健。

目前研究长度泛化的经典思路如下：

**1. ALiBi 直接外推**：ALiBi 主要是在计算 attention score 后添加上一个不可学习的 bias，公式如下：



![ALiBi 公式（一）](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/alibi_formula1.png)





![ALiBi 公式（二）](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/alibi_formula2.png)



ALiBi 由于线性偏差，无法在单层注意力中捕捉远距离信息。ALiBi 之所以能外推，是因为它类似局部滑动注意力，通过多层 attention 捕捉超过单层长度的信息，但这种信息感知能力与网络层数成正比，理论上是有限的。

**2. PI 内插法**：将预测的长文本的位置编码乘上因子 $L_{train} / L_{test}$，缩放到训练长度范围内。尽管位置内插避免了远处的位置越界问题，但这同时压缩了邻近 Token 的距离，严重扰乱了模型的局部分辨率，会导致 PPL 爆炸，但研究显示，PI 经过长文本微调后，效果也是可以的。

**3. NTK-aware 系列**：NTK-aware Scaled RoPE 的思路是修改 $\sin(m/base^{-2i/d})$ 中的 base，变成 $\sin(m/(base \cdot \alpha)^{-2i/d})$。从苏剑林等人的分析来看，这种方式类似于进制转换。理论上分析，这种方法可以实现高频外推、低频内插的目的。

Dynamic NTK 则是随着序列长度的增加动态缩放超参数，其 base 缩放公式为：

$$
    base' = base \cdot \left(\frac{\alpha \cdot seq_len}{max_position_embeddings} - (\alpha - 1)\right)^{d/(d-2)}
$$


**4. ReRoPE 和 Leaky ReRoPE**：苏剑林提出的方法，结合外推和内插的方法，先设定一个窗口大小 $w$，在窗口内使用大小为 1 的位置间隔，在窗口外使用大小为 $1/k$ 的位置间隔。不管输入长度是多少，它的位置编码范围都不超过 $w$，这种方式可以支持任意长度的 content。此外，苏剑林还提出了 $\log n$ 的缩放因子，在测试阶段每个 $q$ 都乘上 $\log n$ 以稳定注意力分数的分布。

**5. YaRN**：有两部分工作。第一部分是对灵活进制的设计，不同的位置用不同的进制：对于低维度，表示数字高位，外推，角度不变；对于中间维度，表示数字的中间位，需要逐步内插；对于高维度，表示数字的低位，彻底内插。计算公式如下：



![YaRN 的位置编码缩放公式](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/yarn_formula.png)



第二部分工作是在对 logits 进行 softmax 操作之前引入温度 $t$，可以统一地影响困惑度，注意力权重的计算修改为：



![YaRN 的注意力权重计算公式](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/yarn_attention.jpg)



YaRN 方法在微调和非微调场景中均超过以前所有方法，且在对不到 0.1% 的原始预训练数据进行微调后，YaRN 在上下文窗口扩展中达到了最先进的性能。

</details>

**40. 请比较一下几种常见的 LLM 架构，例如 Encoder-Only, Decoder-Only, 和 Encoder-Decoder，并说明它们各自最擅长的任务类型。**

<details>
<summary>💡 查看答案</summary>

LLM的架构主要可以分为三类，它们的核心区别在于使用了Transformer的哪些部分以及注意力机制的类型，这直接决定了它们各自擅长的任务。


**1. Encoder-Only 架构**

- 结构： 由多个Transformer Encoder层堆叠而成。
- 核心机制： 双向自注意力机制。在处理序列中的任何一个词元时，模型都可以同时关注到它左边和右边的所有词元。这使得模型能够获得非常丰富的上下文表示。
- 最擅长的任务类型：自然语言理解 (NLU)。
- 具体任务：
- 分类任务： 情感分析、文本分类。
- 序列标注： 命名实体识别 (NER)。
- 句子关系判断： 自然语言推断 (NLI)。
- 完形填空： 像BERT的Masked Language Model (MLM) 预训练任务本身。
- 原因： 这些任务的核心是理解输入文本的深层含义，而双向上下文对于准确理解至关重要。这类模型的输出通常是固定的标签或类别，而非自由生成的长文本。

**2. Decoder-Only 架构**

- 结构： 由多个Transformer Decoder层堆叠而成，但移除了其中的Encoder-Decoder交叉注意力部分。
- 核心机制： 单向（因果）自注意力机制 (Causal Self-Attention)。在预测第 `t` 个词元时，模型只能关注到位置 `1` 到 `t-1` 的词元，不能看到未来的信息。这种自回归的特性天然适合生成任务。
- 最擅长的任务类型：自然语言生成 (NLG)。
- 具体任务：
- 开放式文本生成： 写文章、故事、诗歌。
- 对话系统/聊天机器人： 如ChatGPT。
- 代码生成： 如Copilot。
- 上下文续写 (In-context Learning)。
- 原因： 语言的生成过程是顺序的、从左到右的，Decoder-Only架构的单向注意力完美地模拟了这一过程。目前绝大多数的通用大语言模型都采用此架构。

**3. Encoder-Decoder 架构**

- 结构： 包含一个完整的Encoder栈和一个完整的Decoder栈。
- 核心机制： Encoder部分使用双向注意力来编码整个输入序列，形成一个全面的上下文表示。Decoder部分在生成输出时，一方面使用单向注意力处理已生成的序列，另一方面通过交叉注意力 (Cross-Attention)机制来关注Encoder的输出，确保生成内容与输入相关。
- 最擅长的任务类型：序列到序列 (Seq2Seq)。
- 具体任务：
- 机器翻译： 将一种语言（输入序列）翻译成另一种语言（输出序列）。
- 文本摘要： 将一篇长文章（输入序列）概括成几句话（输出序列）。
- 问答： 将问题（输入序列）转换为答案（输出序列）。
- 原因： 这类任务需要首先对源序列有一个完整的、全局的理解（由Encoder完成），然后基于这个理解有条件地生成一个目标序列（由Decoder完成）。

</details>

**41. prefix Decoder、causal Decoder 和 Encoder-Decoder 的区别是什么？**

<details>
<summary>💡 查看答案</summary>

prefix Decoder、causal Decoder 和 Encoder-Decoder 的区别在于 **attention mask** 不同：

**Encoder-Decoder**：
- **介绍**：输入双向注意力，输出单向注意力
- **代表模型**：T5、Flan-T5、BART
- **特点**：在输入上采用双向注意力，对问题的编码理解更充分
- **适用任务**：在偏理解的 NLP 任务上效果好
- **缺点**：在长文本生成任务上效果差，训练效率低

**causal Decoder**：
- **特点**：自回归语言模型，预训练和下游应用是完全一致的，严格遵守只有后面的 token 才能看到前面的 token 的规则
- **适用任务**：文本生成任务效果好
- **优点**：训练效率高，zero-shot 能力更强，具有涌现能力

**prefix Decoder**：
- **特点**：prefix 部分的 token 互相能看到，是 causal Decoder 和 Encoder-Decoder 的折中
- **缺点**：训练效率低



![prefix Decoder、causal Decoder 和 Encoder-Decoder 的 attention mask 对比](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/attention_masks.png)

</details>

**42. 什么是 Prefix LM 和 Causal LM？**

<details>
<summary>💡 查看答案</summary>

**Causal LM（因果语言模型）**：
- 每个 token 只能 attend 到它自己以及它之前的所有 token（下三角掩码）。
- 训练目标是标准的"预测下一个 token"。
- 代表：GPT 系列、LLaMA。
- 优点：天然适合自回归生成，推理时可直接逐 token 生成并复用 KV Cache。

**Prefix LM（前缀语言模型）**：
- 对前缀部分的 token 允许双向注意力，对生成部分仍采用因果注意力。
- 代表：UniLM、GLM（ChatGLM 早期版本）。
- 优点：前缀部分可获得类似 BERT 的双向理解能力，同时生成部分保持自回归特性。

**核心区别**：注意力掩码设计不同——Causal LM 全程单向；Prefix LM 对输入前缀双向、对输出部分单向。目前主流大模型几乎全部采用纯 Causal LM。

</details>

**43. 为什么现在的 LLM 都是 Decoder-only 架构？**

<details>
<summary>💡 查看答案</summary>

1. **统一的预训练目标，简单且可规模化**：使用单一的"预测下一个 token"自回归目标，数据获取和训练流程更简单统一。
2. **天然适合生成任务和 In-Context Learning**：Decoder-only 在零样本/少样本学习方面表现出更强的能力。
3. **架构和实现更简单、效率更高**：单一的因果 Transformer 堆叠，配合 KV Cache，推理时效率更高。
4. **实验/规模化经验的支持**：大量实验表明 Decoder-only + 大规模预训练 + 指令微调的范式展现出更好的可扩展性和泛化能力。
5. **理论上的表达能力**：有研究指出，Decoder-only 模型理论上可以模拟双向注意力的某些效果。

</details>

**44. 激活函数有了解吗，你知道哪些LLM常用的激活函数？为什么选用它？**

<details>
<summary>💡 查看答案</summary>

是的，我了解激活函数。激活函数是神经网络中至关重要的一环，它的主要作用是为网络引入非线性（non-linearity）。如果没有激活函数，多层神经网络本质上等同于一个单层的线性模型，无法学习和拟合复杂的数据模式。


在现代大型语言模型（Transformer架构）中，最常用的激活函数主要有两个：GeLU 和 SwiGLU。


1. GeLU (Gaussian Error Linear Unit):

- 简介： GeLU曾是Transformer模型中的主流激活函数，被BERT、GPT-2等经典模型采用。它的数学形式是 $x \cdot \Phi(x)$，其中 $\Phi(x)$ 是高斯分布的累积分布函数。
- 为什么选用它？
- 平滑性： GeLU是ReLU的一个平滑近似。相比于ReLU在0点的突变，GeLU的平滑特性使其在优化过程中梯度更稳定，更有利于模型收敛。
- 随机正则化思想： GeLU可以看作是综合了Dropout和ReLU的思想。它根据输入的数值大小，对其进行随机的“归零”或“保留”，但这个过程是确定性的。输入越小，其输出被“归零”的概率越高。

1. SwiGLU (Swish-Gated Linear Unit):

- 简介： SwiGLU是目前最先进、最主流的选择，被Llama、PaLM、Mixtral、Gemma等一系列现代LLM广泛采用。它属于门控线性单元（Gated Linear Unit, GLU） 家族的变体。
- 工作原理： 它将前馈网络（FFN）的第一个线性层的输出 $X$ 分成两部分， $A$ 和 $B$ 。然后通过公式 $Swish(A) \otimes B$ 计算输出，其中 $Swish(x) = x \cdot \sigma(x)$ ， $\sigma$ 是Sigmoid函数， $\otimes$ 是逐元素相乘。
- 为什么选用它？
- 门控机制（Gating Mechanism）： SwiGLU的核心优势在于其“门控”设计。 $B$ 部分可以被看作一个动态的“门”，它可以根据输入内容，控制 $Swish(A)$ 中的信息哪些可以通过、哪些需要被抑制。这种机制显著增强了模型的表达能力，使得FFN层可以更灵活地处理信息。
- 实证效果优越： Google在PaLM论文中的实验发现，使用SwiGLU替换标准的GeLU或ReLU，可以显著提升模型的性能（降低困惑度）。尽管SwiGLU会增加FFN层的参数量（因为需要两个矩阵而不是一个），但其带来的性能增益被证明是值得的。

</details>

**45. GLU 与 SwiGLU 激活函数**

<details>
<summary>💡 查看答案</summary>

**GLU（Gated Linear Unit）**：
- 基本形式：GLU(x) = (xW + b) $\otimes$ $\sigma$(xV + c)，通过门控机制动态控制信息的流通量。

**SwiGLU**：
- 是 GLU 的一个变体，将门控函数中的激活函数换成 **Swish/SiLU**（SiLU(x) = x $\cdot$ $\sigma$(x)）。
- 在 FFN 层中的具体应用形式（如 LLaMA）：FFN(x) = (SiLU(xW$_1$) $\otimes$ (xW$_3$)) W$_2$，即用三个线性变换矩阵替代标准 FFN 中的两个矩阵。
- **优势**：实验表明使用 SwiGLU 的 Transformer 在多个下游任务上的表现优于使用 ReLU/GeLU 的标准 FFN，代价是引入了第三个权重矩阵，需要适当缩小隐藏层维度以保持总参数量大致不变。

</details>

**46. LLAMA 中的 FFN 层作用是什么？**

<details>
<summary>💡 查看答案</summary>

- **模型结构**：对于单纯的 self-attention 而不考虑残差和 MLP 的话，堆叠 $n$ 层后整个模型的秩会很快坍缩，所有表征趋于一个 vector，加上 MLP 和残差会好很多。
- **增加表达能力**：升维之后的线性变换通常伴随着非线性激活函数，打破了线性模型的限制。
- **存储知识**：FFN 本身占据了 Transformer 很大的参数量，第一层线性变换是 Key Memory，第二层线性变换是 Value Memory。

</details>

**47. FFN 的计算公式是什么样的？**

<details>
<summary>💡 查看答案</summary>

前馈网络（Feed-Forward Network，FFN）是 Transformer 中每个编码器/解码器层的重要组成部分，紧跟在注意力子层之后。标准 FFN 由两个线性变换和一个非线性激活函数组成：


$$
	FFN(x) = ReLU(xW_1 + b_1)W_2 + b_2
$$


其中 $W_1 \in \mathbb{R}^{d \times 4d}$ 是升维矩阵（将隐层维度从 $d$ 扩展到 $4d$），$W_2 \in \mathbb{R}^{4d \times d}$ 是降维矩阵（将维度从 $4d$ 压缩回 $d$），$b_1 \in \mathbb{R}^{4d}$、$b_2 \in \mathbb{R}^{d}$ 为偏置项。

**关于中间维度 $4d$ 的选择**

原始 Transformer 论文将中间维度设置为 $4d$，这一选择在实践中被证明是合理的经验值：
- 升维到 $4d$ 为模型提供了足够的"宽度"来学习复杂的非线性特征变换，可以理解为在高维空间中对 token 的语义表示进行丰富和重组。
- FFN 模块的参数量为 $2 \times d \times 4d = 8d^2$（忽略偏置），约占 Transformer 总参数量的 $2/3$，是模型存储知识的主要容量来源。研究表明 FFN 层可以近似看作键值记忆（Key-Value Memory），隐式存储了训练数据中的事实知识。
- 对于现代大模型（如去除偏置、采用 Pre-Norm 的变体），FFN 的基本结构保持不变，但激活函数和维度比例有所调整（见后续小节）。

FFN 在每个位置独立计算（position-wise），即对序列中每个 token 的表示向量单独应用相同的 FFN，参数在所有位置间共享，与注意力机制的跨位置交互形成互补。

</details>

**48. 介绍一下 GeLU 计算公式**

<details>
<summary>💡 查看答案</summary>

GELU（Gaussian Error Linear Unit）是一种激活函数，由 Hendrycks & Gimpel 于 2016 年提出，其精确定义为：


$$
	GELU(x) = x \cdot \Phi(x) = x \cdot \frac{1}{2}\left[1 + erf\!\left(\frac{x}{\sqrt{2}}\right)\right]
$$


其中 $\Phi(x)$ 是标准正态分布 $\mathcal{N}(0,1)$ 的累积分布函数（CDF），$erf(\cdot)$ 为高斯误差函数。由于精确计算涉及误差函数，实践中常采用以下两种近似：


$$
	GELU(x) \approx 0.5x\left(1 + \tanh\!\left[\sqrt{\frac{2}{\pi}}\left(x + 0.044715x^3\right)\right]\right)
$$


或更简洁的 Sigmoid 近似：

$$
	GELU(x) \approx x \cdot \sigma(1.702x)
$$


**GELU 的主要特点**：
- **平滑性**：在全定义域内无限可微，梯度连续，有助于优化器稳定收敛，避免了 ReLU 在原点处的不可微问题。
- **非单调性**：在负值区域存在一个极小值（约 $x \approx -0.17$ 处），之后随 $x$ 减小趋近于零，与 Swish 类似，这种非单调性被认为有助于提升表达能力。
- **概率解释**：$\Phi(x)$ 可理解为输入 $x$ 被"保留"的概率——输入值越大，通过的概率越接近 1；输入值越小，越接近 0。GELU 可视为对输入的随机正则化（类似 Dropout）的期望。
- **与 ReLU 的关系**：当输入较大时，$\Phi(x) \to 1$，GELU 近似线性；当输入极小时，$\Phi(x) \to 0$，输出趋近于零，整体行为与 ReLU 相似但更平滑。

</details>

**49. 介绍一下 Swish 计算公式**

<details>
<summary>💡 查看答案</summary>

Swish（也称 SiLU，Sigmoid Linear Unit）由 Google Brain 于 2017 年提出，其定义为：


$$
	Swish(x) = x \cdot \sigma(\beta x) = \frac{x}{1 + e^{-\beta x}}
$$


其中 $\sigma(\cdot)$ 为 Sigmoid 函数，$\beta$ 为可调节的超参数（或可学习参数）。在大多数实际应用中取 $\beta = 1$，此时 Swish 退化为 SiLU：


$$
	SiLU(x) = x \cdot \sigma(x) = \frac{x}{1 + e^{-x}}
$$


**Swish 的主要特点**：
- **平滑性**：与 GeLU 同样在全域无限可微，梯度流动更稳定。
- **非单调性**：在 $x \approx -1.28$ 处取得极小值约 $-0.278$，之后单调递增。负值区域的非零输出允许梯度反向传播，避免了 ReLU 的"神经元死亡"问题。
- **无上界、有下界**：输出无上界（随 $x \to +\infty$ 线性增长），但有下界（约 $-0.278$），兼具线性函数的梯度传播能力和非线性函数的表达能力。
- **可学习性**：当 $\beta$ 作为可学习参数时，Swish 可以在训练过程中自适应地调整激活形状；$\beta \to 0$ 时趋近于线性函数 $x/2$；$\beta \to \infty$ 时趋近于 ReLU。
- **与 GeLU 的关系**：两者形状非常接近，$GELU(x) \approx Swish_{1.702}(x)$，在实践中性能差异通常可以忽略。

</details>

**50. 使用 GLU 线性门控单元的 FFN 块计算公式**

<details>
<summary>💡 查看答案</summary>

GLU（Gated Linear Unit）由 Dauphin 等人于 2017 年提出，通过引入门控机制动态调控信息流：


$$
	GLU(x, W, V, b, c) = \sigma(xW + b) \otimes (xV + c)
$$


其中 $\sigma(\cdot)$ 为 Sigmoid 函数，$\otimes$ 为逐元素（Hadamard）乘积。Sigmoid 分支充当"门"，决定每个维度上信息通过的比例；线性分支提供待调控的内容表示。

将 GLU 集成到 FFN 块中，并将激活函数泛化为任意函数 $Act(\cdot)$，得到：


$$
	FFN_{GLU}(x) = \bigl(Act(xW_1) \otimes xV\bigr) W_2
$$


**参数量对比**：
- **传统 FFN**：两个权重矩阵，$W_1 \in \mathbb{R}^{d \times 4d}$、$W_2 \in \mathbb{R}^{4d \times d}$，参数量为 $8d^2$。
- **GLU FFN**：三个权重矩阵 $W_1, V \in \mathbb{R}^{d \times d_{ff}}$、$W_2 \in \mathbb{R}^{d_{ff} \times d}$，若保持 $d_{ff} = 4d$ 则参数量增加 50%。为维持与传统 FFN 相同的参数量，通常将 $d_{ff}$ 调整为 $\frac{2}{3} \times 4d = \frac{8}{3}d$（取最近的合适倍数）。

根据门控激活函数的选择，GLU 有以下常见变体：
- $Act = \sigma$：原始 GLU
- $Act = ReLU$：ReGLU
- $Act = GELU$：GeGLU（Gemma 采用）
- $Act = Swish$：SwiGLU（LLaMA、PaLM 等采用）

</details>

**51. LLaMA 中的 FFN 层有哪些魔改的做法？**

<details>
<summary>💡 查看答案</summary>

LLaMA 系列对标准 FFN 进行了多项改进，以提升参数效率、训练稳定性和模型性能：

1. **SwiGLU 激活函数**：将标准 FFN 的 ReLU 替换为 Swish 门控变体（取 $\beta = 1$）：

    $$
    FFN_{SwiGLU}(x) = \bigl(Swish_1(xW_1) \otimes (xW_3)\bigr) W_2
    $$

    LLaMA 的 FFN 层共有三个权重矩阵需要训练：$W_1$（门控分支的升维投影）、$W_3$（内容分支的升维投影）、$W_2$（降维输出投影）。为与原始双矩阵 FFN 保持参数量大致相当，LLaMA 将中间维度设为 $\frac{2}{3} \times 4d$ 并取128的最近倍数（例如，$d=4096$ 时中间维度为 11008）。

    相比 GeLU，SwiGLU 在 PaLM 等模型的实验中显示出更好的困惑度和下游任务性能，被认为是 LLaMA 性能竞争力的重要来源之一。

2. **去除偏置项**：LLaMA 的所有线性层（包括 FFN 中的 $W_1$、$W_2$、$W_3$）均不使用偏置项。这一设计简化了模型结构，减少了参数量，并在大规模训练中被证明不影响模型性能，同时有助于张量并行实现的简洁性。

3. **RMSNorm 前置归一化（Pre-Norm）**：与原始 Transformer 的 Post-Norm 不同，LLaMA 在 FFN 子层的输入端使用 RMSNorm 进行归一化（Pre-Norm），并去掉了归一化中的均值中心化步骤，仅保留缩放操作：

    $$
    RMSNorm(x) = \frac{x}{RMS(x)} \cdot \gamma, \quad RMS(x) = \sqrt{\frac{1}{d}\sum_{i=1}^d x_i^2}
    $$

    Pre-Norm 结合 RMSNorm 显著提升了深层模型的训练稳定性，并降低了约 7%--64% 的归一化计算开销（相比 LayerNorm）。

4. **MoE 的思路（LLaMA 的延伸方向）**：混合专家模型（Mixture of Experts，MoE）将单一的 FFN 替换为 $N$ 个并行的"专家" FFN，并通过一个可学习的路由网络（Router）为每个 token 动态选择 Top-$K$ 个专家进行计算：

    $$
    MoE-FFN(x) = \sum_{i=1}^{N} g_i(x) \cdot FFN_i(x), \quad g(x) = TopK\!\left(softmax(xW_g),\ K\right)
    $$

    MoE 的核心优势在于以较低的推理计算量（每个 token 仅激活 $K$ 个专家）换取极大的模型总参数量，实现"稀疏激活"下的参数扩展。Mixtral 8$\times$7B 即采用此架构，总参数量约 47B，但每个 token 的激活参数仅约 13B，推理成本与 13B 稠密模型相当，性能却接近 70B 级别。DeepSeek-V2/V3 进一步将 MoE 与细粒度专家划分和共享专家机制结合，将参数效率推向新的极限。

</details>

**52. 大模型 FFN 中用 SwiGLU 为啥设置 FFN 的映射为 $\frac{8{3}d$？}**

<details>
<summary>💡 查看答案</summary>

这是为了在 SwiGLU 结构下保持参数总量和计算量与原来普通的 ReLU-FFN 基本相等的换算方式。

普通 ReLU-FFN 的参数量：$Params_{ReLU} = 2 \cdot d \cdot f_{ffn}$，其中 $f_{ffn} = 4d$。

SwiGLU-FFN 的参数量：$Params_{SwiGLU} = 3 \cdot d \cdot m$（$W$, $V$, $W_{out}$ 三个矩阵）。

令两者相等：$3 \cdot d \cdot m = 2 \cdot d \cdot 4d$，解得 $m = \frac{8}{3}d$。

**一句话总结**：$\frac{8}{3}$ 只是一个让 SwiGLU 的参数/计算量 $\approx$ 原来 ReLU-FFN 的换算系数。

</details>

**53. 各 LLMs 都使用哪种激活函数？**

<details>
<summary>💡 查看答案</summary>

激活函数是前馈网络（FFN）模块的核心组件，直接影响模型的表达能力与训练稳定性。现代大语言模型普遍从早期的 ReLU 迁移到更平滑、信息保留更充分的门控激活函数。以下是主流模型的激活函数选型及其背后的设计考量：

- **GPT 系列（OpenAI）：GeLU**

    GPT-1/2/3 及后续 GPT 系列均采用 **GeLU**（Gaussian Error Linear Unit）。其定义为：
    \[
    GeLU(x) = x \cdot \Phi(x) = x \cdot \frac{1}{2}\left[1 + erf\!\left(\frac{x}{\sqrt{2}}\right)\right]
    \]
    其中 $\Phi(x)$ 为标准正态分布的累积分布函数。实践中常用近似公式：
    \[
    GeLU(x) \approx 0.5x\left(1 + \tanh\!\left[\sqrt{\frac{2}{\pi}}\left(x + 0.044715x^3\right)\right]\right)
    \]
    GeLU 相比 ReLU 在负值区域有平滑的非零输出，梯度更稳定，在 NLP 任务中表现优于 ReLU，是 BERT、GPT 系列的标配激活函数。

- **LLaMA 系列（Meta）：SwiGLU**

    LLaMA 1/2/3 全系列均采用 **SwiGLU**，这是 GLU（Gated Linear Unit）的 Swish 变体，由 Noam Shazeer 在 2020 年提出。其定义为：
    \[
    SwiGLU(\mathbf{x}, \mathbf{W}, \mathbf{V}, \mathbf{b}, \mathbf{c}) = Swish_1(\mathbf{x}\mathbf{W} + \mathbf{b}) \otimes (\mathbf{x}\mathbf{V} + \mathbf{c})
    \]
    其中 $Swish_1(x) = x \cdot \sigma(x)$，$\otimes$ 为逐元素乘法。简化形式下，LLaMA 的 FFN 模块结构为：
    \[
    FFN_{SwiGLU}(\mathbf{x}) = \left(Swish(\mathbf{x} W_1) \otimes \mathbf{x} W_3\right) W_2
    \]
    与标准 FFN 相比，SwiGLU 引入了第三个权重矩阵 $W_3$，使得参数量略有增加。为保持总参数量不变，LLaMA 将隐层维度从 $4d$ 调整为 $\frac{8}{3}d$（取最近的128的倍数）。SwiGLU 在多项 NLP 基准上被证明优于 GeLU 和 ReLU，是目前开源大模型的主流选择。

- **ChatGLM2（清华 KEG & 智谱 AI）：SwiGLU（从 GeLU 升级）**

    ChatGLM 第一代使用 GeLU，从 **ChatGLM2** 起升级为 **SwiGLU**，同时结合了 RMSNorm、RoPE 等一系列现代架构改进，整体向 LLaMA 架构对齐。这一升级带来了训练稳定性和下游任务性能的双重提升。

- **PaLM（Google）：SwiGLU**

    Google 的 PaLM（Pathways Language Model）及其后续版本 PaLM-2 同样采用 **SwiGLU**，并结合并行注意力与 FFN 层（Parallel Transformer Block）的设计，进一步提升了训练效率。

- **其他主流模型的激活函数选型**

    - **Mistral / Mixtral（Mistral AI）**：SwiGLU
    - **Qwen 系列（阿里巴巴）**：SwiGLU
    - **DeepSeek 系列**：SwiGLU（部分版本在 MoE 专家层中使用）
    - **Gemma（Google DeepMind）**：GeGLU（GeLU 的门控版本）
    - **BERT（Google）**：GeLU
    - **T5（Google）**：ReLU（原始版本），后续 T5 变体改用 GeLU 或 GeGLU

**激活函数演进小结**

- **ReLU**：早期主流，计算简单，但存在"神经元死亡"问题（负值梯度为零），在大模型中逐渐被淘汰。
- **GeLU**：平滑近似 ReLU，负值区域有非零输出，训练更稳定，是 BERT、GPT 系列的选择。
- **Swish**（即 SiLU）：$Swish(x) = x \cdot \sigma(x)$，无上界、有下界、非单调，在多项任务上优于 GeLU。
- **SwiGLU / GeGLU**：在 Swish 或 GeLU 基础上引入门控机制，通过元素级乘法实现对信息流的动态调控，是当前最先进大模型的首选激活函数。

从趋势上看，现代大语言模型几乎一边倒地选择了 **SwiGLU**，其在参数效率、训练稳定性和下游性能之间取得了良好平衡，已成为事实上的行业标准。

</details>

**54. 混合专家模型（MoE）是如何在不显著增加推理成本的情况下，有效扩大模型参数规模的？请简述其工作原理。**

<details>
<summary>💡 查看答案</summary>

混合专家模型（Mixture of Experts, MoE）是一种模型架构，它的核心思想是通过 “稀疏激活”（Sparse Activation） 的策略，来解决模型规模与计算成本之间的矛盾。它允许模型拥有巨大的总参数量，但在处理任何一个输入时，只动用其中一小部分参数，从而在不显著增加推理成本（FLOPs）的情况下，大幅提升模型容量。


工作原理如下：


1. 用“专家”替换FFN层：

- 在标准的Transformer架构中，计算量最大的部分之一是前馈网络（Feed-Forward Network, FFN）层。
- MoE架构将模型中的部分或全部FFN层替换为MoE层。一个MoE层由两部分组成：
- N个“专家”（Experts）： 每个专家本身就是一个独立的、规模较小的FFN。
- 1个“门控网络”或“路由器”（Gating Network / Router）： 这是一个小型的神经网络，通常是一个简单的线性层。

1. 动态路由决策：

- 当一个词元（token）的向量表示来到MoE层时，它首先被送入路由器。
- 路由器的作用是 “决策” ，判断这个token应该由哪些专家来处理最合适。它会输出一个包含N个分数的向量，代表该token与N个专家的“匹配度”。

1. Top-K稀疏激活：

- 路由器输出的分数经过Softmax归一化后，系统并不会激活所有的专家。相反，它只选择分数最高的Top-K个专家（K通常很小，比如1或2）。
- 这就是“稀疏激活”的关键：对于每一个token，只有极少数（K个）专家被激活并进行计算，其余的（N-K个）专家则完全不参与，不产生任何计算成本。

1. 加权输出：

- 被选中的K个专家分别对输入的token向量进行处理，得到K个输出向量。
- 最终的输出是这K个输出向量的加权和，权重同样由路由器的输出分数决定。

如何实现“参数大但成本低”？

- 假设一个模型有8个专家（N=8），并且每次只激活2个（K=2），如Mixtral-8x7B模型。
- 总参数量： 模型的总参数量是所有共享部分（如注意力层）的参数量，加上所有8个专家的参数量之和。这使得模型的总参数规模可以非常大（例如达到47B）。
- 推理成本： 但在进行一次前向传播（推理）时，对于任意一个token，实际参与计算的只有共享部分和被激活的2个专家。因此，其计算量（FLOPs）约等于一个规模小得多的“稠密”模型（例如一个13B的模型）。
- 结论： MoE成功地将总参数量（代表模型的知识容量）和单次推理的计算量（代表模型的速度和成本）解耦，从而实现了“用小模型的成本，获得大模型的知识”。

</details>

**55. 什么是 MoE？**

<details>
<summary>💡 查看答案</summary>

MoE（Mixture of Experts，混合专家模型）是一种先进的神经网络架构，其核心思想是将多个专门化的子模型（称为“专家”）与一个动态路由机制结合，根据输入数据的特点选择性地激活部分专家进行计算。

核心组件：
1. **专家（Experts）**：每个专家是一个独立的子模型，通常是一个小型神经网络，专注于处理输入数据的特定子集或特定任务。
2. **门控网络（Gating Network）**：负责根据输入数据的特征，动态选择最适合的专家或专家组合，并分配权重。
3. **稀疏激活机制（Sparsity）**：每次输入只激活部分专家，而不是整个模型，从而显著降低计算成本。

工作原理：当输入数据进入模型时，门控网络会根据输入特征计算每个专家的权重，并选择权重最高的专家（或前 K 个专家）进行处理。最终，专家的输出结果会按照权重进行汇总，形成模型的最终输出。

</details>

**56. MoE 的特点？**

<details>
<summary>💡 查看答案</summary>

1. **稀疏性**：MoE 的稀疏性主要体现在两个方面：一是模型架构的稀疏性，即不是所有专家都会同时被激活；二是数据处理的稀疏性，即只有与任务最相关的数据才会被选定的专家处理。
2. **高效性**：与稠密模型相比，MoE 能够在相同的计算资源条件下，更快地达到相同的性能水平。这主要得益于其稀疏的门控机制，使得模型能够集中处理关键信息，减少不必要的计算开销。
3. **扩展性**：MoE 架构的扩展性非常好，能够在不增加计算负担的情况下，显著增加模型容量和数据集规模。

</details>

**57. MoE 是替代了原先大模型中的哪一层？**

<details>
<summary>💡 查看答案</summary>

MoE（混合专家模型，Mixture of Experts）在大模型中主要替代了传统 Transformer 架构中的前馈网络（FFN）层。本节将从 FFN 层的功能定位、MoE 的替代动机、具体结构对比以及主流实现方案等角度进行详细阐述。

#### 为什么是 FFN 层而不是注意力层？

在标准 Transformer 中，每个块（Block）由两个核心子层构成：

- **多头自注意力层（Multi-Head Self-Attention, MHA）**：负责建模序列中不同位置之间的依赖关系，参数量相对较小，计算具有强结构性（Query-Key-Value 三路投影）；
- **前馈网络层（Feed-Forward Network, FFN）**：对每个位置的表示独立进行非线性变换，通常由两层线性变换和一个激活函数构成，参数量占整个 Transformer 块的约 $\frac{2}{3}$。

FFN 层天然适合被 MoE 替代，原因如下：

1. **FFN 是参数的主要载体**：在标准 Transformer 中，若隐藏维度为 $d$，FFN 中间层维度通常为 $4d$，FFN 层参数量为 $2 \times d \times 4d = 8d^2$，而 MHA 层参数量约为 $4d^2$，FFN 占据了更大比例的参数量，是模型容量的核心瓶颈；
2. **FFN 各位置独立计算**：FFN 对序列中每个位置的 token 独立处理（position-wise），不同 token 之间没有交互，这使得"不同 token 走不同专家"的路由机制天然可行；
3. **注意力层难以稀疏化**：MHA 层的 Query、Key、Value 之间存在全局交互，若对其进行专家化改造，路由机制的设计和训练稳定性将面临极大挑战，目前学界对此的探索仍较有限。

#### 标准 FFN 层的结构

标准 Transformer 的 FFN 层结构如下（以 SwiGLU 激活为例，LLaMA 系列采用此设计）：

\begin{lstlisting}[style=promptstyle]
	# 标准 FFN 层（SwiGLU 变体）
	class FeedForward(nn.Module):
	def __init__(self, d_model, d_ff):
	self.w1 = nn.Linear(d_model, d_ff)   # 门控路径
	self.w2 = nn.Linear(d_ff, d_model)   # 输出投影
	self.w3 = nn.Linear(d_model, d_ff)   # 激活路径

	def forward(self, x):
	# 所有 token 经过同一组参数
	return self.w2(F.silu(self.w1(x)) * self.w3(x))
\end{lstlisting}

关键特点：每个 token 无论其语义内容如何，都经过**完全相同的一组参数**进行变换，这是一种"一刀切"的设计，缺乏对不同类型输入的专门化处理能力。

#### MoE 对 FFN 层的替代方式

MoE 将单一 FFN 替换为 $N$ 个并行的 FFN 专家（Expert），并引入一个轻量级的**门控网络（Gating Network / Router）**，由其决定每个 token 应该被送往哪些专家进行处理：

\[
MoE(\mathbf{x}) = \sum_{i=1}^{N} G(\mathbf{x})_i \cdot Expert_i(\mathbf{x})
\]

其中 $G(\mathbf{x}) \in \mathbb{R}^N$ 为门控网络的输出权重，通常只有 Top-$k$（$k \ll N$）个专家的权重非零，其余专家不参与计算。以 Top-2 路由为例：

\begin{lstlisting}[style=promptstyle]
	# MoE 层（简化示意）
	class MoELayer(nn.Module):
	def __init__(self, d_model, d_ff, num_experts=8, top_k=2):
	self.experts = nn.ModuleList(
	[FeedForward(d_model, d_ff) for _ in range(num_experts)]
	)
	self.gate = nn.Linear(d_model, num_experts)  # 门控网络

	def forward(self, x):
	# 计算每个 token 对各专家的路由分数
	gate_scores = F.softmax(self.gate(x), dim=-1)

	# 只激活 Top-2 专家
	topk_scores, topk_indices = gate_scores.topk(2, dim=-1)
	topk_scores = topk_scores / topk_scores.sum(dim=-1, keepdim=True)

	# 加权求和各激活专家的输出
	output = sum(
	topk_scores[..., i:i+1] * self.experts[topk_indices[..., i]](x)
	for i in range(2)
	)
	return output
\end{lstlisting}

#### MoE 替代 FFN 的核心优势

1. **参数量与计算量解耦**：MoE 模型的总参数量随专家数量线性增长，但每个 token 的实际计算量只与激活的 Top-$k$ 个专家相关，保持不变。例如，拥有 8 个专家、每次激活 2 个的 MoE 层，参数量是稠密 FFN 的 8 倍，但计算量（FLOPs）仅为稠密 FFN 的 $2/8 = 25%$（稀疏激活）或约 2 倍（若 $k=2$）；

2. **专家专门化**：不同专家在训练过程中会自发地学习处理不同类型的输入，例如某些专家专注于代码、某些专注于自然语言、某些专注于数学推理，从而实现隐式的功能分工；

3. **更高的模型容量上限**：在相同计算预算下，MoE 可以部署远多于稠密模型的总参数，从而突破参数规模的天花板。

#### 主流大模型中的 MoE 实现

当前主流的 MoE 大模型均遵循"注意力层保持不变、FFN 层替换为 MoE"的设计范式：

- **Mixtral 8×7B（Mistral AI）**：将每个 Transformer 块的 FFN 替换为 8 个专家，每次 Top-2 激活，实际激活参数约 13B，总参数约 47B；
- **DeepSeek-V2 / V3**：引入细粒度专家分割（Fine-Grained Expert Segmentation）和共享专家（Shared Expert）机制，在 MoE 基础上进一步优化负载均衡；
- **Qwen-MoE 系列**：同样以 FFN 替换为核心，引入专家数量更多（如 64 个专家，Top-4 激活）的设计；
- **GPT-4（推测）**：据多方报道采用了 MoE 架构，OpenAI 未官方确认，但业界普遍认为其采用了类似设计。

#### 小结

MoE 替代的是 Transformer 中**前馈网络（FFN）层**，而非注意力层。这一选择并非随意，而是由 FFN 的高参数占比、位置独立性以及路由可行性共同决定的。通过将单一 FFN 替换为多个并行专家加门控路由的结构，MoE 在几乎不增加单 token 推理计算量的前提下，大幅提升了模型的总参数容量和专门化处理能力，成为当前超大规模语言模型突破参数规模瓶颈的核心技术路径。





![标准 Transformer 模块与 MoE Transformer 模块结构对比](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/fig_moe_replace_ffn.png)

</details>

**58. MoE 和 FFN 相比的优势？**

<details>
<summary>💡 查看答案</summary>

- MoE 通过门控网络动态选择部分专家进行计算，而不是像 FFN 那样对所有输入进行密集计算，这种稀疏性可以大幅减少计算量。
- FFN 层主要负责特征变换，而 MoE 的专家网络可以针对不同输入数据动态选择不同的特征变换路径，从而更好地适配多样化的任务。

</details>

**59. MoE 中如何解决负载均衡问题？**

<details>
<summary>💡 查看答案</summary>

**1. 辅助损失函数（Auxiliary Loss）**

为了实现负载均衡，通常会在 MoE 模型的训练过程中引入辅助损失函数：
- **负载均衡损失**：通过计算每个专家实际处理的输入数据量与理想处理数据量之间的差异，并将其作为模型总损失的一部分。
- **变异系数（CV）**：通过计算专家的重要性分数的变异系数，衡量专家之间的重要性分数的差异程度。通过优化辅助损失，降低 CV 分数，从而实现负载均衡。

**2. 动态调整专家选择概率**

- **KeepTopK 策略**：通过引入可训练的高斯噪声，打破对特定专家的偏好，使选择更加随机化。同时，通过设置权重为负无穷，确保只有顶部 K 个专家被激活。
- **无辅助损失策略**：DeepSeek 提出了一种无辅助损失的负载均衡策略，通过动态调整专家选择概率的 Bias 项，实现序列级负载均衡，避免引入额外损失函数对模型收敛的干扰。

**3. 专家容量限制与调整**

为每个专家设定容量上限，限制其一次能处理的样本数量。如果专家达到容量上限，可以暂存待处理的任务，或者通过动态调整专家容量来优化负载。

**4. 分布式训练与负载均衡**

- **冗余专家策略**：当某些专家负载过高时，动态复制高负载专家，并将冗余副本分配到不同的计算节点（如 GPU），避免单一节点成为性能瓶颈。
- **分层负载均衡**：将专家组均匀打包到节点上，确保不同节点的负载平衡，然后在每个节点内复制专家，最后将复制的专家分配到各个 GPU。
- **全局负载均衡**：在专家并行规模较大时，对专家进行全局复制，并将复制的专家分配到各个 GPU。

**5. 简化路由策略**

例如，Switch Transformer 采用了一种简化的 Top-1 路由策略，每次只将输入路由到一个专家，从而减少计算量和通信量，同时通过负载均衡损失确保专家的均匀使用。

</details>

**60. 使用 MoE 的好处是什么？**

<details>
<summary>💡 查看答案</summary>

**MoE（Mixture of Experts）基本结构**

MoE 层由两部分组成：若干个**专家网络**（Expert Networks）$\{E_1, E_2, \ldots, E_n\}$ 和一个**门控网络**（Gating Network）$G$。对于输入 $x$，MoE 层的输出为：
\[
y = \sum_{i=1}^{n} G(x)_i \cdot E_i(x)
\]
在稀疏 MoE 中，$G(x)$ 仅对 Top-$k$ 个专家返回非零权重，其余专家不参与计算。

**优势**：
1. **高效性**：通过稀疏激活机制，每个 token 仅激活 Top-$k$ 个专家（通常 $k=1$ 或 $k=2$），而非全部 $n$ 个专家。设每个专家的 FFN 计算量为 $C$，则稀疏 MoE 每步的计算量为 $k \cdot C$，而密集模型（等效于 $n$ 个专家全部激活）的计算量为 $n \cdot C$，计算量降低至 $k/n$。

2. **可扩展性**：可以在**不显著增加计算量**的情况下，通过增加专家数量来扩展模型的参数规模（即“参数量与计算量解耦”）。例如，将专家数从 8 增加到 64，模型参数量增加 8 倍，但每个 token 的计算量不变（仍激活 $k$ 个专家）。这一特性使得 MoE 非常适合超大规模模型的构建，如 Mixtral 8×7B、DeepSeek-V2/V3 等。

3. **灵活性与专业化**：不同专家在训练过程中自然地倾向于处理不同类型的输入（如不同语言、不同领域的知识），实现了**隐式的功能分工**，使模型对多样化任务具有更强的适应能力。

4. **训练效率**：在相同计算预算下，MoE 模型通常能达到比同计算量稠密模型更好的性能，即在 FLOPs 受限场景下具有更高的参数利用率。

MoE 模型广泛应用于自然语言处理、图像识别等领域，尤其适合处理大规模数据和复杂任务。

</details>

**61. MoE 在训练中主要有哪几种并行策略？**

<details>
<summary>💡 查看答案</summary>

MoE 的训练并行策略主要围绕如何高效地分配和利用计算资源展开，常见的策略包括：

**1. 数据并行（Data Parallelism, DP）**：将训练数据分割成多个小批次，每个计算单元（如 GPU）处理一部分数据。在 MoE 架构中，门控网络和专家网络都会被复制到每个计算单元上，各设备独立前向传播后通过 AllReduce 同步梯度。DP 实现简单，但每个设备需存储完整的模型参数，显存压力较大。

**2. 张量并行（Tensor Parallelism, TP）**：将模型中单个算子（如矩阵乘法）的计算切分到多个设备上并行执行。在 MoE 中，门控网络通常会被复制到每个计算单元，而专家网络中的权重矩阵则按行或列切分到不同设备。TP 可以有效降低单设备显存占用，但引入较多设备间通信（AllReduce）。

**3. 专家并行（Expert Parallelism, EP）**：MoE 特有的并行策略，将不同的专家分配到不同的计算单元上，每个设备只存储和计算自己负责的专家。token 经过门控网络路由后，通过 All-to-All 通信发送到对应专家所在设备进行计算，再将结果返回原设备。EP 可以显著减少单个设备的显存负担，同时允许模型扩展到更多的专家，但 All-to-All 通信在大规模集群中可能成为瓶颈。

**4. 流水线并行（Pipeline Parallelism, PP）**：将模型的不同层（Transformer Block）分配到不同设备，数据以流水线方式依次经过各设备。在 MoE 模型中，由于各层计算量可能不均匀（MoE 层与普通层交替），需要仔细设计流水线划分以避免设备空闲（Pipeline Bubble）。

**5. 混合并行策略**：在实际大规模训练中，通常会结合多种并行策略以达到最佳的显存与计算效率，例如：
- **EP + DP**：专家并行处理参数分布，数据并行提升吞吐量，是 MoE 训练中最常见的组合。
- **EP + DP + TP**：在上述基础上进一步对专家内部的大矩阵进行张量切分，适用于单专家参数量仍然较大的场景。
- **EP + PP + TP**：超大规模 MoE 训练（如千亿参数级别）通常采用三维并行，DeepSeek-V3 等模型的训练即采用类似策略。

</details>

**62. MoE 和集成学习方法有什么异同？**

<details>
<summary>💡 查看答案</summary>

MoE 和集成学习的思想异曲同工，都是集成了多个模型的方法，但两者在目标、结构和推理方式上存在明显差异：

\begin{center}


| **对比维度** | **MoE** | **集成学习** |
| --- | --- | --- |
| 核心目标 | 条件计算，提升模型容量 | 降低方差/偏差，提升泛化性 |
| 专家/基模型激活 | 稀疏激活（仅部分专家） | 所有基模型均参与（稠密） |
| 路由方式 | 可学习的门控网络（动态路由） | 固定规则（平均、投票、Stacking） |
| 训练方式 | 端到端联合训练 | 可独立训练后融合 |
| 推理开销 | 低（稀疏激活） | 高（所有模型均推理） |


\end{center}

MoE 主要是将预测任务分为多个子任务，通过路由网络决定对于当前数据该使用哪个专家，在现有的 MoE 实现中通常只激活部分专家，模型是**稀疏**的；集成学习对于当前预测任务使用多个基模型进行学习，通过 Bagging 或 Boosting 方法融合结果，其模型结构是**稠密**的。

</details>

**63. MoE 中的路由网络是怎么设计的？**

<details>
<summary>💡 查看答案</summary>

**基本结构**

路由网络（Gating Network / Router）通常是一个**轻量级的线性层**，参数量远小于专家网络。对于输入 token 的隐状态 $x \in \mathbb{R}^d$，路由网络计算各专家的 logit 分数：
\[
h(x) = x \cdot W_g, \quad W_g \in \mathbb{R}^{d \times n}
\]
其中 $n$ 为专家总数。

**Top-$k$ 稀疏路由**

主流实现中，取 Top-$k$ 个最高分数的专家参与计算，其余专家的权重置为 $-\infty$（经 softmax 后为 0）：
\[
G(x) = softmax(TopK(h(x), k))
\]
最终输出为被选中专家输出的加权和：
\[
y = \sum_{i \in TopK} G(x)_i \cdot E_i(x)
\]

**负载均衡设计**

朴素的 Top-$k$ 路由容易出现**专家塌缩**（Expert Collapse）问题，即门控网络总是选择少数几个专家，导致其他专家得不到训练。常见的解决方案包括：

- **辅助负载均衡损失**（Auxiliary Load Balancing Loss）：在训练损失中加入惩罚项，鼓励每个专家被均匀地分配 token：
    \[
    \mathcal{L}_{aux} = \alpha \cdot n \cdot \sum_{i=1}^{n} f_i \cdot P_i
    \]
    其中 $f_i$ 为第 $i$ 个专家实际处理的 token 比例，$P_i$ 为门控网络分配给第 $i$ 个专家的平均概率，$\alpha$ 为平衡系数。Switch Transformer、Mixtral 等均采用此方法。

- **加噪声路由**（Noisy Top-$k$ Gating）：在 logits 上添加可调噪声以增加路由随机性（详见下节）。

- **专家容量限制**（Expert Capacity）：为每个专家设定最大可处理 token 数（Capacity Factor），超出容量的 token 将被丢弃或路由至其他专家，强制实现负载均衡。

</details>

**64. 稀疏与密集 MoE 的区别？**

<details>
<summary>💡 查看答案</summary>

- **稀疏 MoE（Sparse MoE）**：每个 token 只路由到 Top-$k$（$k \ll n$）个专家，其余专家不参与计算，门控权重为稀疏向量。计算量与 $k$ 成正比，与专家总数 $n$ 无关，适合大规模模型扩展。当前主流大模型（Mixtral、DeepSeek-V2/V3、Qwen-MoE 等）均采用稀疏 MoE。代价是需要 All-to-All 通信进行专家路由，且存在负载不均衡问题。

- **密集 MoE（Dense MoE）**：选择所有 $n$ 个专家参与计算，但根据门控权重 $G(x)_i$ 对各专家输出进行加权融合。等价于一个参数量更大的标准 FFN（所有专家权重均被激活），计算量与 $n$ 成正比。密集 MoE 无需路由通信，训练更稳定，但失去了稀疏激活带来的计算效率优势，实际应用较少。

</details>

**65. Switch Transformer 的路由策略是什么？**

<details>
<summary>💡 查看答案</summary>

Switch Transformer（Google，2021）采用了一种极简的 **Top-1 路由策略**，即每个 token 只路由到**一个**专家（$k=1$）：
\[
G(x) = softmax(x \cdot W_g), \quad 选择 \arg\max_i G(x)_i
\]

**设计动机**：
- 将每 token 的专家通信量从 Top-$k$（$k>1$）降至 1 次，大幅减少 All-to-All 通信开销。
- 简化路由机制，降低实现复杂度，同时实验表明 Top-1 路由在多数任务上与 Top-2 效果相当。

**负载均衡**：Switch Transformer 引入**辅助负载均衡损失**，并设置**专家容量因子**（Capacity Factor $\geq 1.0$），超出容量的 token 直接跳过当前 MoE 层（通过残差连接传递），以此强制保证专家间的负载均匀分配。

</details>

**66. 路由网络加噪声的初衷是什么？**

<details>
<summary>💡 查看答案</summary>

**问题背景**：在训练初期，门控网络的参数接近随机初始化，若某个专家偶然获得较高的 logit 分数并被频繁选中，则该专家获得更多梯度更新，进而在下一轮中获得更高分数，形成**正反馈循环**，最终导致绝大多数 token 只路由到少数几个专家（专家塌缩）。

**噪声路由的做法**：在路由 logits 上添加标准正态噪声，经缩放后再取 Top-$k$：
\[
h(x)_i = (x \cdot W_g)_i + \epsilon \cdot softplus\!\left((x \cdot W_n)_i\right), \quad \epsilon \sim \mathcal{N}(0, 1)
\]
其中 $W_n$ 为可学习的噪声缩放矩阵，$softplus$ 保证噪声幅度非负。

**噪声的作用**：
1. **打破对称性**：训练初期各专家 logit 相近，噪声使不同 token 随机路由到不同专家，防止早期的“赢家通吃”现象。
2. **增加探索性**：类似强化学习中的 $\epsilon$-greedy 策略，噪声使得原本分数略低的专家也有机会被选中，从而获得梯度更新，所有专家得到充分训练。
3. **促进负载均衡**：噪声使路由决策更加分散，与辅助负载均衡损失配合，进一步缓解专家负载不均问题。

随着训练的进行，门控网络逐渐学会合理的路由策略，噪声的相对影响减小，路由决策趋于稳定。

</details>

**67. 传统 MoE 和现在主流 MoE 是如何选择 expert 的？**

<details>
<summary>💡 查看答案</summary>

**传统 MoE 选择方法**：学习一个 gate 网络，令其为 $W_g$，$W_g$ 和输入 $x$ 乘积的 softmax 记为 $G$，则 $G$ 就为每个专家的权重，多个专家聚合的时候按照**稠密**的方式进行聚合（所有专家均参与）：

$$
	y = \sum_{i=1}^{n} G(x)_i \cdot E_i(x), \quad G(x) = softmax(x \cdot W_g)
$$


**主流 MoE 选择方法**：使用 **Noisy Top-$k$ Gating** 来计算，通过可学习的权重矩阵 $W_g$ 和噪音矩阵 $W_n$ 得到加噪后的 logit 值，取 Top-$k$ 后仅对这 $k$ 个位置计算 softmax，其余专家权重置零，然后和上面的专家聚合方式一样：

$$
	h(x)_i = (x \cdot W_g)_i + \epsilon \cdot softplus((x \cdot W_n)_i)
$$


$$
	G(x) = softmax(TopK(h(x),\ k))
$$


$$
	y = \sum_{i \in TopK} G(x)_i \cdot E_i(x)
$$


**两种方法的对比**：

\begin{center}


| **对比维度** | **传统 MoE** | **主流 MoE（Noisy Top-$k$）** |
| --- | --- | --- |
| 专家激活数量 | 全部 $n$ 个（稠密） | Top-$k$ 个（稀疏） |
| 计算量 | $O(n \cdot C)$ | $O(k \cdot C)$，$k \ll n$ |
| 负载均衡 | 自然均匀 | 需辅助损失+噪声 |
| 可扩展性 | 差（计算随 $n$ 线性增长） | 好（计算与 $n$ 无关） |


\end{center}



![传统 MoE 与主流 MoE 门控机制对比](https://img.pricenode.uk/llm_interview_questions/zhongw/chapter1/moe_gating_comparison.jpg)

</details>

**68. 如何解决 MoE 中训练不均衡现象？**

<details>
<summary>💡 查看答案</summary>

**方法一：在门控网络后应用 dropout**

MoE 的稀疏性与 dropout 的原理类似，MoE 是根据任务的具体情况选择激活一定数量的专家模型来完成这个任务，而 dropout 则是对神经网络中的神经元进行随机性失活，每次训练的时候只保留一定的参数，这不但让网络具备了稀疏性特征，减轻了整个网络的参数压力，还会降低模型发生过拟合的概率，提高模型的泛化能力。

**方法二：引入软约束**

引入额外的损失函数：对于一个批次的训练样本，定义每个专家的重要性度量 $Importance(X)$，即该专家在该批次中门控输出值的总和。同时，定义损失项 $L_{importance}(X)$，并将其加入到模型的总体损失中：

$$
    Importance(X) = \sum_{x \in X} G(x)
$$


$$
    L_{importance}(X) = w_{importance} \cdot CV(Importance(X))^2
$$


其中 $CV(X)$ 表示变异系数，它反映了分布的离散程度。如果出现极端分布，CV 值将会较高。

此外，ST-MoE 引入的 Router z-loss 在保持了模型性能的同时显著提升了训练的稳定性：

$$
    L_z(x) = \frac{1}{B} \sum_{i=1}^{B} \left( \log \sum_{j=1}^{N} e^{x_j^{(i)}} \right)^2
$$


其中 $B$ 是 token 的数量，$N$ 是专家的数量，$x \in \mathbb{R}^{B \times N}$ 是输入到路由器的 logits。这会惩罚进入门控网络的较大 logits，在路由器之前使用 z-loss 是有用的。

</details>

**69. MoE 算法为何训练时候容易不稳定？**

<details>
<summary>💡 查看答案</summary>

1. 要设计好门控来控制进入的 expert，不然容易导致模型坍塌到一个 expert 上去。
2. 稀疏模型的输出具有高方差，其对随机种子敏感（确定初始参数值，训练数据改组，要剔除的值等），因此有不同训练会导致不同的表现。
3. 现在都是混合精度训练，一些低精度的格式会影响梯度不稳定，从而导致训练不稳定。
4. 在训练过程中，由于每个输入样本仅触发部分专家，导致不同专家的梯度更新频率不同，这种不均衡的梯度更新可能会导致模型参数更新不稳定，影响整体模型的收敛性。

</details>

**70. MoE 算法优缺点？**

<details>
<summary>💡 查看答案</summary>

**MoE 的优点**：
- **提高预训练速度**：MoE 模型能够在比稠密模型更少的计算资源下进行有效的预训练。
- **更快的推理速度**：由于在推理过程中只激活部分专家，MoE 模型的推理速度通常比具有相同参数数量的稠密模型更快。
- **提升模型扩展性**：MoE 架构允许模型在保持计算成本不变的情况下增加参数数量。
- **多任务学习能力**：MoE 在多任务学习中表现出色，例如 Switch Transformer 在所有 101 种语言上都显示出了性能提升。

**MoE 面临的挑战**：
- **训练不稳定性**：不同专家的梯度更新频率不同，可能导致模型参数更新不稳定。
- **负载均衡问题**：路由机制的随机性和输入数据的特性可能导致某些专家承载的计算负载远高于其他专家。
- **微调挑战**：MoE 模型在微调时可能会面临迁移学习效果不确定的问题。
- **推理效率**：在推理阶段，MoE 模型需要根据输入选择合适的专家，这种选择过程增加了计算复杂度。
- **部署复杂性**：MoE 模型通常比传统的稠密模型更加复杂，在实际部署中可能需要针对不同硬件环境进行优化。

</details>

**71. 为什么 MoE 有效从原理上分析一下？**

<details>
<summary>💡 查看答案</summary>

论文“Towards Understanding Mixture of Experts in Deep Learning”对 MoE 的有效性进行了分析，其结论是：底层问题的聚类结构以及专家的非线性是 MoE 成功的关键。论文为了论证这个观点做了一个分类实验，发现使用单一的专家难以很好的将多个类分类，而使用了多个非线性的专家模块，就可以很好的解决这个问题；理论证明了路由网络可以学习聚类中心特征，从而将输入的复杂问题分解为多个简单的子问题，而这个子问题用单个专家就可以解决。

总结：MoE 路由网络的设计有助于发现底层问题的聚类结构，从而有效将问题拆解，以便于各个专家解决。此外，MoE 可以视作为多个模型的集成，而从集成学习的角度来看，多模型集成的结果，在泛化性上是要优于单模型的。

</details>

**72. DeepSeek-MoE 中的细粒度专家是什么意思？**

<details>
<summary>💡 查看答案</summary>

“细粒度”在 DeepSeek-MoE 里的含义可以一句话概括为：**把原来“大而全”的专家拆成很多“小而专”的微专家，让模型在计算量不变的前提下，能用更灵活、更精准的组合去处理每一种输入。**

具体做法：
1. 把每个传统 FFN 专家沿隐藏维度切成 $m$ 份，中间维度缩为原来的 $1/m$，于是得到 $m$ 倍数量的“微专家”。
2. 为了维持相同的计算预算，同时把每次激活的专家数也扩大到 $mK$ 个（原来选 $K$ 个，现在选 $mK$ 个）。
3. 组合空间因此爆炸式增加：例如 64 个微专家里选 8 个，组合数可达 4.4 亿种，而传统 16 个专家选 2 个只有 120 种。
4. 每个微专家专注极窄的知识子域，从而提升专业度和整体精度。
5. 再配上一组始终激活的共享专家，负责通用知识，避免重复存储，减少冗余。

一句话总结：细粒度 = **专家更小、数量更多、组合更灵活、知识更专精**，在参数与算力都不增加的情况下，让 MoE 的“分而治之”发挥到极致。

</details>

**73. DeepSeek-MoE 中的细粒度专家代码是如何实现的？**

<details>
<summary>💡 查看答案</summary>

**1. 微专家权重切分**

原始大专家权重 $\mathbf{W} \in \mathbb{R}^{d_{model} \times d_{ff}}$，沿隐藏维度切成 $m$ 份，得到微专家权重：

$$
    \mathbf{W} = [\mathbf{W}_1, \mathbf{W}_2, \dots, \mathbf{W}_m], \quad \mathbf{W}_i \in \mathbb{R}^{d_{model} \times (d_{ff}/m)}
$$


每个微专家的中间维度 $d_{ff}^{micro} = \frac{d_{ff}}{m}$。

**2. 计算量守恒**

传统 MoE 每 token 激活 $K$ 个专家，总计算量 $FLOP_{old} = K \cdot (2 d_{model} d_{ff})$。

细粒度后激活 $mK$ 个微专家，每个维度 $d_{ff}/m$，总计算量：

$$
    FLOP_{new} = mK \cdot \left(2 d_{model} \frac{d_{ff}}{m}\right) = K \cdot (2 d_{model} d_{ff}) = FLOP_{old}
$$


**3. 路由与输出（微专家部分）**

对输入 token $\mathbf{x} \in \mathbb{R}^{d_{model}}$：

$$
    logits(\mathbf{x}) = \mathbf{x}^{\top} \mathbf{E}, \quad \mathbf{E} \in \mathbb{R}^{d_{model} \times N_{micro}}, \quad N_{micro} = m N_{expert}
$$


$$
    g_i = Softmax_i(Top-mK(logits(\mathbf{x})))
$$


微专家输出：

$$
    \mathbf{y}_{micro} = \sum_{i \in \mathcal{T}_{mK}} g_i \mathbf{W}_i \mathbf{x}
$$


**4. 共享专家（始终激活）**

最终输出：

$$
    \mathbf{y}_{final} = \sum_{i \in \mathcal{T}_{mK}} g_i \mathbf{W}_i \mathbf{x} + \sum_{j=1}^{s} \mathbf{W}^{shared}_j \mathbf{x}
$$


**将大矩阵 $\mathbf{W}$ 沿隐藏维度切成 $m$ 份，同时把激活专家数从 $K$ 提升到 $mK$，计算量保持不变，但路由组合空间被指数级放大。**

</details>

**74. DeepSeek 的 MoE 为何把门控从 softmax 变成 sigmoid？**

<details>
<summary>💡 查看答案</summary>

DeepSeek-V3 把 MoE 门控从 softmax 变成 sigmoid 的核心原因：

1. **专家维度爆炸**：V3 路由专家从 160 增加到 256，维度越高，softmax 的归一化越会把分数压向 $1/N$，Top-K 区分度骤降。softmax 先把所有 logit 减去最大值防止溢出，再取 exp，最后除以总和。当专家数 $N$ 很大时，logit 的方差往往不大，exp 后的值会非常接近，于是归一化结果 $\approx 1/N$。
2. **数值稳定性**：softmax 需要整体求和，FP32 下小数位误差被放大；sigmoid 每个专家独立计算，避免累加误差。
3. **梯度更稳**：softmax 饱和区梯度小、易出现“赢者通吃”；sigmoid 梯度平滑，训练更稳定。
4. **“多标签”视角**：激活 8 个专家本质是多标签问题，sigmoid 给每个专家独立 0--1 分，比 softmax 的“互斥”假设更自然。

</details>

**75. DeepSeek 中如何计算 sequence 的负载均衡损失？**

<details>
<summary>💡 查看答案</summary>

在 DeepSeek-V3 的 MoE 中，“sequence 粒度的负载均衡损失”不再像早期工作那样对每个 token 独立计算，而是把一个完整序列当成一个统计单元，用序列内所有 token 的“路由计数”来做均衡。

**1. 序列级统计量**

对整个序列做一次汇总：
- 专家 $i$ 被选中的总次数：$f_i = \sum_{t=1}^{T} \mathbb{1}_{i,t}$
- 专家 $i$ 的累计 gate 值：$P_i = \sum_{t=1}^{T} g_{i,t}$

**2. 序列级负载均衡损失**

希望 $f_i$ 尽可能接近“平均负载” $\frac{T \cdot K}{N}$：

$$
    \mathcal{L}_{bal} = \sum_{i=1}^{N} \left(f_i - \frac{T \cdot K}{N}\right)^2
$$


当所有专家负载完全相等时，损失为 0。

**3. 训练中的用法**

该损失只在训练阶段加入总 loss，权重系数通常很小（如 1e-2）。推理阶段完全去掉，不影响生成质量。

</details>

**76. DeepSeek 设备级 Token 丢弃策略导致丢失的 Token 怎么办？**

<details>
<summary>💡 查看答案</summary>

虽然多种负载均衡损失能在训练时引导模型趋向负载均衡，但它们无法严格保证计算负载完全均衡。为此，DeepSeek-MoE 在训练与推理阶段均引入了一种设备级 Token 丢弃策略：

1. **计算设备容量**：对每个批次的输入 token，先计算每张设备的平均容量 $C = \frac{Batch 总 token 数}{设备数}$。
2. **排序并准备丢弃**：对当前设备实际分到的 token 数 $T_a$，按路由得分降序排列。若 $T_a > C$，则从尾部开始丢弃超出的 token，使其总量恰好等于 $C$。
3. **丢弃后的行为**：被丢弃的 token 仍保留残差路径。在当前 MoE 层不再进入任何专家网络，但残差连接继续生效，token 的 hidden state 仍传递到下一层 Transformer。由于不同层的专家路由是解耦的，这些 token 在后续层仍有机会被路由并参与专家计算。
4. **训练-推理一致性**：训练阶段，每批次有 10% 的样本不执行 token 丢弃，模拟推理阶段“无丢弃”场景，防止分布漂移。推理阶段，默认始终启用 token 丢弃，确保计算量严格受控。

一句话总结：通过“按路由分排序 + 尾部 token 丢弃”，在不破坏残差流的前提下，实现严格的设备级计算负载均衡，并在训练中保留少量无丢弃样本以稳定分布。

</details>

**77. 参数共享是什么，它如何提高 LLM 的效率？**

<details>
<summary>💡 查看答案</summary>

参数共享是一种技术，模型的多个层或部分共享相同的参数集。这减少了模型的总参数数量，而不会牺牲性能。它提供了几个优势：

- **内存效率**：共享参数减少了模型的内存占用，使其在存储和计算方面都更加高效。
- **改进的泛化能力**：通过重用参数，模型不太可能过拟合特定任务或数据集，从而提高其对新任务的泛化能力。
- **更快的训练和推理**：较少的参数意味着模型可以更快地进行训练和部署，使其更适合资源受限的环境。
参数共享通常用于 BERT 和 GPT 等模型，以增强可扩展性，而不会显著增加参数数量。

</details>

**78. 嵌入在 LLM 中扮演什么角色？**

<details>
<summary>💡 查看答案</summary>

嵌入是标记的向量表示，编码：
- 语义信息（意义）。
- 语法信息（结构）。
它们将离散标记转换为连续数值，以便模型输入。

**类型**：
- 单词嵌入：表示整个单词。
- 子词嵌入：处理未知/罕见单词。

**示例**：Word2Vec、GloVe 和 Transformer 中的学习嵌入。

</details>

**79. 上下文嵌入在 LLM 中的作用是什么，它与静态嵌入有何不同？**

<details>
<summary>💡 查看答案</summary>

上下文嵌入是由 Transformer 等模型生成的动态词表示，它会根据句子中周围的词而变化。相比之下，静态嵌入（如 Word2Vec 或 GloVe）为每个词分配一个固定的表示，无论其上下文如何。上下文嵌入的关键优势包括：

- **捕捉多义性**：上下文嵌入可以根据上下文表示一个词的多种含义，而静态嵌入在处理不同句子中具有不同含义的词时会遇到困难。
- **性能更好**：由于它们考虑了单词在句子中的完整上下文，上下文嵌入在情感分析、机器翻译和问答等任务中表现更好。
- **灵活性更强**：上下文嵌入能够适应输入，更有效地捕捉单词和句子之间的细微关系。
像 BERT 和 GPT 这样的 LLM 依靠上下文嵌入在 NLP 任务中实现最先进的性能。

</details>

**80. 嵌入短内容和长内容有什么区别？**

<details>
<summary>💡 查看答案</summary>

嵌入短内容和长内容有一些关键区别。对于短内容，捕捉完整含义可能具有挑战性。由于单词较少，嵌入模型难以把握文本的细微差别。这可能导致类似短文本仅因为其简短而被认为更相似，而不是基于实际的语义相似性。

另一方面，长内容为嵌入模型提供了更丰富的信息池。它可以分析整个文本中单词之间的关系，考虑到句子结构和主题发展等因素。这可以创建更准确、更全面的嵌入，反映内容的完整含义。然而，过长的内容也可能带来挑战。某些模型可能难以处理大量信息，可能会丢失一些细节。

</details>

**81. 在 LLM 应用中如何使用嵌入模型？**

<details>
<summary>💡 查看答案</summary>

嵌入模型在 LLM 应用中就像一个搜索引擎，帮助 LLM 从大量数据中找到最相关的信息片段。这有助于 LLM 提供更准确、更信息丰富且更有帮助的回答。以下是一个很好的例子：

假设 LLM 是一位强大的语言专家，但它需要一种方法来理解用户话语背后的含义。嵌入模型介入并将文本输入（查询、文档）转换为称为嵌入的数值表示。这些嵌入捕捉单词和概念之间的语义关系。

LLM 应用通常利用检索增强生成（RAG）。在 RAG 中，嵌入模型将用户查询和知识库中的相关信息都转换为嵌入。然后，它找到知识库中与用户查询最相似的嵌入信息。这些检索到的信息连同用户的查询一起成为 LLM 的上下文。最后，LLM 利用这个上下文生成既相关又信息丰富的回答。

</details>

**82. 什么是上下文窗口大小，它如何影响 LLM 的性能？**

<details>
<summary>💡 查看答案</summary>

上下文窗口大小是指模型一次能够处理的最大 token 数量。它对 LLM 的性能有显著影响，因为：

- **更长的上下文理解**：更大的上下文窗口允许模型考虑输入序列的更大部分，提高其捕捉标记之间长距离依赖关系和关系的能力。
- **对长文本的性能**：在文档总结或长篇问答等任务中，更大的上下文窗口使模型能够处理更长的输入，而不会截断重要信息。
- **与计算的权衡**：增加上下文窗口大小需要更多的计算资源，因为自注意力机制的复杂性与序列长度呈二次方增长。
选择适当的上下文窗口大小对于根据手头的任务平衡模型性能和计算效率至关重要。

</details>

**83. 什么是词元化？请比较一下 BPE 和 WordPiece 这两种主流的子词切分算法。**

<details>
<summary>💡 查看答案</summary>

什么是词元化（Tokenization）？ 词元化是将原始的文本字符串分解成一个个独立的单元（称为“词元”或“token”），并将这些词元映射到唯一的整数ID的过程。这是自然语言处理模型处理文本的第一步，因为模型只能处理数字输入。


现代大型语言模型普遍采用 子词（Subword） 词元化算法，它介于按词切分和按字符切分之间。这样做的好处是：

1. 有效处理未登录词（OOV）： 任何罕见词或新词都可以被拆解成已知的子词组合，避免了“未知”标记。
2. 平衡词表大小与序列长度： 相比于词级别，词表规模大大减小；相比于字符级别，生成的序列长度又不会过长，兼顾了效率。
3. 保留形态信息： 像 "running", "runner" 这样的词可以共享 "run" 这个子词，使得模型能够理解词根和词缀的关系。

BPE vs. WordPiece


BPE和WordPiece是两种最主流的子词切分算法，它们构建词表的过程相似，但在合并子词的决策标准上有所不同。


	extbf{BPE (Byte Pair Encoding)}

- 工作原理：

1. 初始化： 词汇表由语料库中出现的所有基本字符组成。
2. 迭代合并： 重复以下步骤直到达到预设的词表大小：

a.  在整个语料库中，统计所有相邻词元对的出现频率。 b.  找出频率最高的那个词元对（例如 `('e', 's')`）。 c.  将这个词元对合并成一个新的、更长的词元（`'es'`），并将其加入词汇表。 d.  在语料库中，用新词元替换所有出现的该词元对。

- 应用模型： GPT系列、Llama等。
- 特点： 算法思想简单直观，完全基于数据中符号对的出现频率。

	extbf{WordPiece}

- 工作原理：

1. 初始化： 与BPE一样，词汇表也从所有基本字符开始。
2. 迭代合并（核心区别）： WordPiece在选择合并哪两个子词时，不是基于频率，而是基于语言模型的似然（Likelihood）。它会尝试所有可能的合并，并选择那个能够最大程度提升训练数据似然值的合并操作。

- 可以通俗地理解为：如果把语料库看作一个语言模型，每次合并都应该让这个语言模型产生当前语料库的概率变得最大。它倾向于合并那些内部凝聚力更强的字符组合。
- 应用模型： BERT, DistilBERT, Electra。
- 特点： WordPiece在切分时，通常会在单词的非起始部分子词前加上特殊符号（如`##`），例如 "tokenization" 可能会被切分为 `("token", "##ization")`。

主要区别总结： | 特性             | BPE (Byte Pair Encoding)                     | WordPiece                                                  | | :--------------- | :------------------------------------------- | :--------------------------------------------------------- | | 合并决策标准 | 频率驱动：合并出现次数最多的相邻子词对。 | 似然驱动：合并能最大化提升语料库语言模型似然的子词对。 | | 理论基础     | 数据压缩算法，简单高效。                     | 概率语言模型，理论上更优。                                 | | 应用代表     | GPT, Llama, RoBERTa                          | BERT, T5                                                   |

</details>

**84. LLM 如何处理词汇表外（OOV）单词？**

<details>
<summary>💡 查看答案</summary>

LLM 通过子词分词技术处理 OOV 单词，例如：
- **字节对编码（BPE）**：将单词分解为较小的单元，如前缀、后缀或字符对。
- **WordPiece**：类似于 BPE，但合并标准略有不同。
- **单词语言模型**：选择最可能的子词序列。
这确保即使是未见过的单词也可以表示为已知子词的组合。

</details>

**85. 1 个 token 对应多少汉字或英文？**

<details>
<summary>💡 查看答案</summary>

Token 是大语言模型处理文本的基本单位。在模型内部，所有输入和输出都以 token 序列的形式存在，而非原始字符。不同语言、不同字符的 token 化效率存在显著差异，理解这一点有助于更好地估算上下文长度与费用。

#### 英文的 token 对应关系

对于英文文本，主流模型（如 GPT 系列、Claude 系列）通常采用 BPE（Byte Pair Encoding）或类似的子词分词算法。其一般规律如下：

- **常见英文单词**：大多数常见单词（如 `hello`、`world`、`language`）对应 **1 个 token**。
- **较长或生僻单词**：会被拆分为多个子词 token，例如 `unbelievable` 可能被切分为 `un`、`believ`、`able` 三个 token。
- **经验法则**：平均而言，**1 个 token $\approx$ 4 个英文字符**，或大约 **0.75 个英文单词**。换言之，100 个 token 大致对应 75 个英文单词。
- **空格与标点**：空格通常会与紧随其后的单词合并为一个 token（如 `_hello`），标点符号一般单独占 1 个 token。

#### 中文的 token 对应关系

中文属于表意文字体系，每个汉字在语义上相对独立，但其 token 化方式与英文有所不同：

- **常见汉字**：在多数分词器中，**1 个汉字通常对应 1 至 2 个 token**。这是因为汉字的 UTF-8 编码通常占 3 个字节，而 BPE 词表中对常见汉字可能有直接映射，也可能回退到字节级编码。
- **经验法则**：粗略估算，**1 个 token $\approx$ 0.5 至 1 个汉字**，即平均 **1 个汉字约需 1 至 2 个 token**。
- **与英文的对比**：相同语义内容的中文文本，其 token 数通常多于对应的英文文本，这意味着处理中文时上下文窗口的"利用效率"相对较低，费用也相应更高。

#### 不同模型分词器的差异

不同模型使用的分词器（Tokenizer）不同，token 对应关系也有所差异：

- **OpenAI（GPT 系列）**：使用 `tiktoken` 库，采用 cl100k_base 等 BPE 词表，中文汉字平均约 1.5 至 2 个 token/字。
- **Anthropic（Claude 系列）**：使用自研分词器，优化了多语言支持，中文 token 效率与 GPT 系列相近。
- **Meta（LLaMA 系列）**：早期版本对中文支持有限，token 效率较低；LLaMA 3 改进了多语言词表。
- **阿里（Qwen 系列）**、**百度（ERNIE 系列）**等中文优化模型：针对中文扩充了词表，1 个汉字往往能直接映射为 1 个 token，token 效率显著更高。

#### 示例对比

以下通过具体示例说明 token 数量的差异：

\begin{lstlisting}[style=promptstyle]
	示例 1（英文）：
	文本：Hello, how are you?
	Token 数（约）：6 tokens
	分析：Hello / , / how / are / you / ?

	示例 2（中文）：
	文本：你好，你最近怎么样？
	Token 数（约）：9~14 tokens
	分析：每个汉字和标点各占 1~2 个 token

	示例 3（混合）：
	文本：我使用 GPT-4 来处理 NLP 任务。
	Token 数（约）：15~20 tokens
	分析：中文字符 token 效率低于英文字母组成的词汇
\end{lstlisting}

#### 实际估算建议

在实际使用中，可参考以下经验值进行 token 数量的快速估算：

- 英文：$token 数 \approx 单词数 \times 1.3$
- 中文：$token 数 \approx 汉字数 \times 1.5$（使用通用英文为主的分词器时）
- 中文（中文优化模型）：$token 数 \approx 汉字数 \times 1.0$

如需精确计算，建议直接使用对应模型的官方分词工具，例如 OpenAI 提供的在线 Tokenizer 工具（<https://platform.openai.com/tokenizer>）或 Hugging Face 上各模型的 `AutoTokenizer`。。

</details>

**86. 现有的大模型如何实现灵活填空生成文本？**

<details>
<summary>💡 查看答案</summary>

Fill-in-the-Middle（FIM，中间填充）是一种让语言模型在文本的任意位置动态生成缺失内容的技术，突破了传统自回归模型只能从左到右生成的限制。

核心思想是**双向上下文建模**和**动态插入生成**。通过特殊标记（如 `<PRE>`、`<SUF>`、`<MID>`）将输入文本划分为前缀、后缀和中间部分，模型根据前缀和后缀预测中间内容。

应用案例：
- **代码补全**（如 GitHub Copilot）
- **指令生成**（Reverse Mode Generation）
- **文本编辑**（句子改写）

</details>

**87. 什么是分词？**

<details>
<summary>💡 查看答案</summary>

分词（Tokenization）是自然语言处理（NLP）中的基础步骤，指将原始文本分解为更小的语义或结构单元（称为“标记”，即 token）的过程。这些单元可以是单词、子词、字符，甚至是有意义的短语或标点符号。

**常见的分词粒度**

根据应用场景的不同，分词可以在不同粒度上进行：
- **词级别（Word-level）**：以空格或标点为边界切分，例如 “I love NLP” $\rightarrow$ [“I”, “love”, “NLP”]。简单直观，但词表规模大，且无法处理未登录词（OOV）。
- **子词级别（Subword-level）**：将词拆分为更小的有意义片段，是现代大模型的主流方案。常见算法包括 BPE（Byte Pair Encoding）、WordPiece 和 SentencePiece。
- **字符级别（Character-level）**：以单个字符为 token，词表最小，但序列较长，模型难以捕捉长距离依赖。
- **字节级别（Byte-level）**：以 UTF-8 字节为单位，GPT-2/GPT-4 等模型采用此方案，可以无损地表示任意语言的文本。

**为什么分词重要？**
- **数字化表示**：语言模型将文本作为数字序列进行处理，每个 token 对应词汇表（vocabulary）中的一个唯一索引，模型通过该索引查找对应的 embedding 向量。
- **计算效率**：通过将文本转换为固定词表上的离散符号序列，显著降低了计算复杂度，使模型能够高效地进行矩阵运算。
- **处理未登录词（OOV）**：通过将罕见词或新词拆分为子词来处理，例如 “unbelievable” $\rightarrow$ [“un”, “believ”, “able”]，从而避免了纯词级别分词中大量 `<UNK>` 标记的问题。
- **多语言灵活性**：子词和字节级别分词能够跨语言共享词表，在多语言任务中无需为每种语言单独维护词表，同时有效控制词表规模（通常为 32K--100K）。
- **影响上下文长度**：同样的文本内容，分词粒度越细，序列越长，消耗的上下文窗口（context window）越多；因此分词效率直接影响模型能够处理的有效信息量。

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%


% -------------------------------------------------------

</details>

**88. 大模型中的 token embedding 和输出的 embedding 是同一个吗？**

<details>
<summary>💡 查看答案</summary>

不是，从 LLaMA 的源码来看不是同一个东西。输入使用的是 `VocabParallelEmbedding`，而输出使用的是 `ColumnParallelLinear`，两者是独立的参数。

**详细说明**

在 Transformer 架构中，涉及两处与词汇表相关的线性映射：

- **输入端（Token Embedding）**：将输入 token 的索引映射为稠密向量，即一个形状为 $[V, d]$ 的嵌入矩阵 $\mathbf{E}$，其中 $V$ 为词表大小，$d$ 为隐层维度。在 LLaMA 中，这由 `VocabParallelEmbedding` 实现，其权重按词表维度切分到多个 GPU 上。
- **输出端（LM Head）**：将最后一层 Transformer 的隐层状态 $\mathbf{h} \in \mathbb{R}^d$ 映射回词表空间，得到 logits $\mathbf{z} = \mathbf{h} \mathbf{W}^\top \in \mathbb{R}^V$，再经 softmax 得到下一个 token 的概率分布。在 LLaMA 中，这由 `ColumnParallelLinear` 实现，其权重按输出维度（词表维度）切分。

**权重共享（Weight Tying）的概念**

尽管 LLaMA 中两者是独立参数，但在一些较早的模型（如原始 Transformer、GPT-2）中，会将输入嵌入矩阵 $\mathbf{E}$ 与输出投影矩阵 $\mathbf{W}$ **共享权重**（Weight Tying），即令 $\mathbf{W} = \mathbf{E}$。其理论依据在于：
- 输入嵌入捕捉了 token 的语义相似性，输出投影也应利用相同的语义空间来计算 token 的相关性得分，两者的目标一致。
- 权重共享可以减少约 $V \times d$ 个参数，在词表较大时（如 $V=50257$，$d=768$）节省显著的显存。
- 实验表明权重共享在中小规模模型上通常不会损失性能，甚至有正则化效果。

**LLaMA 选择不共享的原因**

LLaMA 采用独立参数的原因主要有以下几点：
- **模型规模扩大**：在数十亿参数规模下，输入嵌入与输出投影分别承担不同职责，解耦有助于模型更充分地学习各自的表示。
- **张量并行（Tensor Parallelism）的需求**：LLaMA 针对多 GPU 分布式训练做了深度优化。输入嵌入按行切分（`VocabParallelEmbedding`），输出投影按列切分（`ColumnParallelLinear`），两者的并行策略不同，独立参数便于分别优化通信模式。
- **词表扩展**：LLaMA 系列在不同版本中对词表进行了扩展（如加入代码、多语言 token），独立参数在微调时更灵活。

**LLaMA 源码对应关系（简化示意）**

\begin{lstlisting}[style=promptstyle]
	# 输入端：Token Embedding
	self.embed_tokens = VocabParallelEmbedding(
	config.vocab_size,
	config.hidden_size,
	...
	)

	# 输出端：LM Head（线性层，无偏置）
	self.lm_head = ColumnParallelLinear(
	config.hidden_size,
	config.vocab_size,
	bias=False,
	...
	)

	# 前向传播
	hidden_states = self.embed_tokens(input_ids)   # [B, T] -> [B, T, d]
	# ... Transformer layers ...
	logits = self.lm_head(hidden_states)            # [B, T, d] -> [B, T, V]
\end{lstlisting}

可以看到，`embed_tokens.weight` 与 `lm_head.weight` 是两个独立的张量，并不共享，这与 GPT-2 等早期模型显式执行 `lm_head.weight = embed_tokens.weight` 的做法有所不同。

</details>

**89. 大模型中的 tokenize 方法有哪些？**

<details>
<summary>💡 查看答案</summary>

**主流分词算法简介**

- **BPE（Byte Pair Encoding）**：从字符级别出发，反复合并语料中出现频率最高的相邻符号对，直到词表达到预设大小。GPT 系列模型广泛采用此方法。
- **WordPiece**：与 BPE 类似，但合并策略基于最大化语言模型的似然值而非频率。BERT 采用此方法。
- **SentencePiece**：语言无关的分词框架，将空格也作为特殊字符处理，支持 BPE 和 Unigram 两种子算法，LLaMA、T5 等模型采用此框架。
- **Unigram Language Model**：从一个较大的初始词表出发，通过最小化整体损失逐步剔除低贡献的 token，最终得到紧凑词表。

**示例：BPE 分词过程**

以下是一个简化的 BPE 迭代示例，初始语料为 “low lower lowest”：

\begin{lstlisting}[style=promptstyle]
	初始词表（字符级）：
	l o w </w>  ->  "low"
	l o w e r </w>  ->  "lower"
	l o w e s t </w>  ->  "lowest"

	第1次合并：频率最高的符号对为 (l, o)，合并为 "lo"
	第2次合并：频率最高的符号对为 (lo, w)，合并为 "low"
	第3次合并：频率最高的符号对为 (e, r)，合并为 "er"
	...

	最终分词结果：
	"lower"  ->  ["low", "er", "</w>"]
	"newest" ->  ["new", "est", "</w>"]  （未见词也可分解）
\end{lstlisting}

</details>

**90. 说下 BPE 算法的流程？**

<details>
<summary>💡 查看答案</summary>

BPE（字节对编码）是一种简单的数据压缩算法，其中最常出现的连续字节对被替换为不出现在该数据中的一个字节。它最初在 1994 年发表的文章“一种新的数据压缩算法”中被描述。

假设我们有数据 **aaabdaaabac** 需要被编码（压缩）。字节对 **aa** 出现最频繁，因此我们将用 **Z** 替换它，因为 **Z** 不出现在我们的数据中。所以我们现在有 **ZabdZabac**，其中 **Z = aa**。下一个常见的字节对是 **ab**，所以让我们用 **Y** 替换它。我们现在有 **ZYdZYac**，其中 **Z = aa** 和 **Y = ab**。唯一剩下的字节对是 **ac**，它只出现一次，所以我们不会对其进行编码。我们可以使用递归字节对编码将 **ZY** 编码为 **X**。我们的数据现在已经变成了 **XdXac**，其中 **X = ZY, Y = ab, Z = aa**。它不能再被进一步压缩，因为没有字节对出现超过一次。

BPE 确保最常见的单词在词汇表中以单个标记的形式表示，而罕见单词被分解成两个或更多的 subword 标记，这与基于 subword 的 tokenize 算法所做的一致。

</details>

**91. FlashAttention 的原理是什么？它解决了什么问题？**

<details>
<summary>💡 查看答案</summary>

**问题背景**：标准 Attention 计算需要将 Q、K、V 的注意力矩阵（大小为 N×N）全部写入 HBM（显存），当序列长度 N 较大时，显存读写成为瓶颈，速度慢且显存占用大。
**FlashAttention 核心思路**：
- 利用 GPU 的 SRAM（片上高速缓存）替代 HBM 进行中间计算
- 将 Q、K、V 分块（Tiling）依次载入 SRAM，分块计算 Softmax 和加权求和，避免完整 N×N 矩阵的显存读写
- 通过重计算（Recomputation）在反向传播时重新计算注意力权重，而非存储，节省显存
**效果**：速度提升 2~4 倍，显存从 O(N²) 降至 O(N)，且数值结果与标准 Attention 完全等价。

</details>

**92. ALiBi 位置编码的原理是什么？和 RoPE 相比有什么异同？**

<details>
<summary>💡 查看答案</summary>

**ALiBi（Attention with Linear Biases）原理**：
不向输入 embedding 添加位置信息，而是在 Attention 计算时，直接对注意力得分矩阵加上一个与相对距离成正比的负偏置：距离越远，偏置越负，注意力权重越低。每个注意力头使用不同的斜率系数。
**与 RoPE 对比**：
\begin{table}[h]



| **维度** | **ALiBi** | **RoPE** |
| --- | --- | --- |
| 实现方式 | 在注意力分数上加线性偏置 | 对 Q/K 向量做旋转变换 |
| 外推能力 | 强，可直接推广到更长序列 | 需要额外手段（如 YaRN）才能外推 |
| 训练开销 | 无额外参数，轻量 | 无额外参数，轻量 |
| 性能 | 长序列外推好，短序列略弱于 RoPE | 短序列性能优秀 |
| 代表模型 | BLOOM | LLaMA、Qwen、Mistral |


\end{table}

</details>

**93. 传统 Attention 存在哪些问题？**

<details>
<summary>💡 查看答案</summary>

传统的缩放点积注意力机制（Scaled Dot-Product Attention）在处理长序列时暴露出若干根本性瓶颈，主要体现在以下几个方面：

1. **上下文长度约束与二次方复杂度**：计算复杂度与序列长度 $L$ 呈平方关系 $O(L^2)$。对于长度为 $L$ 的序列，注意力机制需要计算所有 token 对之间的相关性，产生 $L \times L$ 的注意力矩阵。当 $L$ 从 512 增长到 8192 时，计算量增长约 256 倍。这使得在受限的硬件条件下，模型无法高效处理长文档、长对话或长代码等场景，训练和推理的时间开销均显著上升。

2. **长距离依赖捕捉困难**：在长序列中，注意力权重经过 softmax 归一化后分散在大量 token 上，导致任意单个远距离 token 所获得的权重极小。这种稀释效应使得模型难以在跨越数百甚至数千个位置的 token 之间建立有效的语义关联，信息传递路径随距离增加而退化，长距离依赖的捕捉能力受到限制。

3. **内存占用高**：标准注意力实现需要在前向传播中显式存储完整的 $L \times L$ 注意力矩阵以供反向传播使用。以 $L=4096$、`float32` 精度、32 个注意力头为例，仅注意力矩阵本身即需约 $4096^2 \times 32 \times 4~bytes \approx 2~GB$ 的显存，严重制约了批大小和序列长度的选择空间。

4. **推理阶段的 KV Cache 开销**：在自回归生成阶段，为避免重复计算，需要缓存每一层所有历史 token 的 Key 和 Value 向量（即 KV Cache）。其大小随序列长度线性增长：
    \[
    KV Cache 大小 = 2 \times L \times n_{heads} \times d_{head} \times n_{layers} \times 精度字节数
    \]
    对于大型模型（如 70B 参数级别），长序列下 KV Cache 可轻易占用数十 GB 显存，成为推理吞吐量的主要瓶颈。

5. **位置编码的泛化局限**：传统绝对位置编码（如正弦编码或可学习位置嵌入）在训练时固定了最大序列长度，模型在推理时若遇到超出训练长度的序列，性能会急剧下降，缺乏对序列长度的外推能力。

</details>

**94. 传统 Attention 相关问题的解决方案**

<details>
<summary>💡 查看答案</summary>

针对上述问题，学术界和工业界提出了多个方向的改进方案：

1. **稀疏注意力机制（Sparse Attention）**：不再让每个 token 关注序列中所有其他 token，而是通过预设的稀疏模式限制注意力范围，从而将复杂度从 $O(L^2)$ 降低到 $O(L \sqrt{L})$ 或 $O(L \log L)$。常见模式包括：
    - **局部窗口注意力**：每个 token 仅关注其周围固定窗口内的 token，如 Longformer 采用的滑动窗口机制。
    - **跨步注意力（Strided Attention）**：以固定步长采样远距离 token，与局部注意力结合实现全局信息传递，如 Sparse Transformer。
    - **全局 token**：引入少量全局 token（如 `[CLS]`），这些 token 与所有位置双向交互，充当信息汇聚节点。

2. **分块注意力（Chunked / Block Attention）**：将序列划分为若干不重叠或部分重叠的块，块内执行完整注意力，块间通过摘要向量或跨块注意力传递信息。此方案在实现上简单，易于与现有系统集成，TransformerXL 的段级循环机制即属于此类思路的扩展。

3. **线性注意力（Linear Attention）**：通过核函数近似将 softmax 注意力重写为线性形式，利用矩阵乘法的结合律将复杂度降低到 $O(L)$：
    \[
    Attention(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \frac{\phi(\mathbf{Q})(\phi(\mathbf{K})^\top \mathbf{V})}{\phi(\mathbf{Q})\phi(\mathbf{K})^\top \mathbf{1}}
    \]
    其中 $\phi(\cdot)$ 为特征映射函数（如 ELU+1）。代表工作包括 Linear Transformer、Performer（使用随机特征近似 softmax）等。线性注意力的缺点是近似误差会导致性能略有损失，且在实践中加速效果因实现方式而存在差异。

4. **多尺度 / 层次化注意力**：结合局部精细注意力与全局粗粒度注意力，在不同分辨率上捕捉序列信息。典型做法是先在局部窗口内计算细粒度注意力，再对窗口摘要执行全局注意力，从而以较低代价获得全局感受野，如 Longformer、BigBird 的混合注意力设计。

5. **FlashAttention**：从 IO 感知的角度重新设计注意力的计算流程，通过分块计算（tiling）和重计算（recomputation）避免将完整的 $L \times L$ 注意力矩阵写入 HBM（显存），将显存复杂度从 $O(L^2)$ 降低到 $O(L)$，同时保持数值上与标准注意力完全等价，在实际硬件上实现了 2--4 倍的训练加速，是目前主流大模型训练的标准实现。

6. **相对位置编码与长度外推**：采用 RoPE（旋转位置编码）、ALiBi 等相对位置编码方案，使注意力分数的位置偏置仅依赖于 token 间的相对距离，从而提升模型在推理时对超长序列的外推能力，缓解传统绝对位置编码的长度泛化问题。

</details>

**95. Attention 变体有哪些？**

<details>
<summary>💡 查看答案</summary>

注意力机制的变体可以按照改进目标的不同大致归为以下几类：

- **稀疏 Attention**：通过限制每个 token 的注意力范围降低复杂度，代表工作包括 Sparse Transformer、Longformer、BigBird、Sliding Window Attention（Mistral 采用）等。

- **线性化 Attention**：利用核函数近似将二次复杂度降为线性，代表工作包括 Linear Transformer、Performer、cosFormer、RetNet（使用衰减机制的线性递推形式）等。

- **原型与记忆压缩（Prototype & Memory Compression）**：将 Key/Value 压缩为少量原型向量，代表工作包括 Set Transformer（使用 Inducing Points）、Perceiver、Longformer 的全局 token 机制等。

- **低秩 Self-Attention**：假设注意力矩阵具有低秩结构，通过矩阵分解或随机投影降低计算量，代表工作包括 Linformer（对 K、V 做低维投影，将 $L \times L$ 矩阵压缩为 $L \times k$）等。

- **结合先验的 Attention**：在注意力权重中引入结构先验（如局部性偏置、位置衰减），代表工作包括 ALiBi（在注意力 logit 上加线性位置惩罚项）、T5 的相对位置偏置等。

- **改进多头机制**：对标准 Multi-Head Attention（MHA）的头数、KV 维度或头间交互方式进行改进，代表工作包括 Multi-Query Attention（MQA，所有头共享同一组 KV）、Grouped-Query Attention（GQA，头分组共享 KV，LLaMA 2/3 采用）、Multi-Head Latent Attention（MLA，DeepSeek 采用的低秩 KV 压缩方案）等。

- **IO 感知实现优化**：在不改变数学等价性的前提下重新设计计算图，代表工作为 FlashAttention 系列（v1/v2/v3），通过分块计算规避 HBM 瓶颈，是目前工业界最广泛采用的注意力实现方式。

</details>

**96. Multi-head Attention 存在什么问题？**

<details>
<summary>💡 查看答案</summary>

Multi-Head Attention（MHA）为每个注意力头维护独立的 Query、Key、Value 投影，在表达能力上具有优势，但也带来了显著的工程代价，尤其体现在推理阶段：

- **训练过程**：MHA 对训练速度的影响相对有限。由于训练阶段采用全序列并行计算，注意力矩阵可以一次性计算完毕，多头带来的参数量增加（相比后续变体）可通过批处理和张量并行缓解。但头数过多时，每个头的维度 $d_{head} = d_{model} / n_{heads}$ 随之减小，单头表达能力下降，在某些任务上可能带来微小的性能损失。

- **推理过程（KV Cache 瓶颈）**：MHA 是推理阶段性能问题的核心来源。在自回归解码时，需要为每一层的每个头单独缓存历史 token 的 Key 和 Value 向量，KV Cache 的总大小为：
    \[
    KV Cache = 2 \times n_{layers} \times n_{heads} \times d_{head} \times L_{seq} \times bytes
    \]
    以 LLaMA-2 70B（80层，64头，$d_{head}=128$，`float16`）、序列长度 4096 为例：
    \[
    2 \times 80 \times 64 \times 128 \times 4096 \times 2~bytes \approx 10.7~GB
    \]
    这意味着仅 KV Cache 就几乎消耗了一张 A100（80GB）约 13% 的显存，严重限制了并发请求数（batch size）和可支持的最大序列长度。

- **内存带宽瓶颈**：推理时每生成一个 token，需要从 HBM 中读取全部 KV Cache。由于读取的数据量远大于实际计算量，推理过程是典型的 **内存带宽受限**（memory-bound）操作而非计算受限（compute-bound）。这导致 GPU 的算力利用率极低，批处理规模受限，推理吞吐量难以提升。

- **改进方向**：为解决上述问题，业界提出了以下替代方案：
    - **Multi-Query Attention（MQA）**：所有查询头共享同一组 Key 和 Value，将 KV Cache 缩小为原来的 $1/n_{heads}$，推理速度大幅提升，但可能略微损失模型精度（Falcon 采用）。
    - **Grouped-Query Attention（GQA）**：将查询头分为 $G$ 组，每组共享一对 KV，在 MHA 与 MQA 之间取得平衡，是目前主流大模型的首选方案（LLaMA-2/3、Mistral、Qwen 均采用）。
    - **Multi-Head Latent Attention（MLA）**：DeepSeek-V2/V3 提出，通过低秩联合压缩 KV 向量，将 KV Cache 压缩至极低维度（如仅缓存 512 维的潜在向量而非完整的 KV），在保持接近 MHA 表达能力的同时大幅降低显存占用。

</details>

**97. 解释下 MiniMax 中 Lightning Attention 的由来**

<details>
<summary>💡 查看答案</summary>

1. **背景**
    - **传统注意力**（Softmax Attention）复杂度为 $O(n^2)$，难以处理长序列。
    - **线性注意力**通过替换 Softmax 为其他激活函数（如 Swish），将计算拆分为：

        $$
        (\phi(Q)\phi(K)^T)V = \phi(Q)(\phi(K)^TV)
        $$

        复杂度降至 $O(nd^2)$，实现线性增长。

2. **因果模型的问题**
    - **单向注意力限制**：生成模型（如 GPT）需使用掩码矩阵 $M$，导致无法直接右乘优化：$O = (QK^T \odot M)V$
    - **累加求和瓶颈**：需逐步计算历史 KV（如 $o_t = \sum_{j=1}^t q_i^T k_j v_j$），无法并行化。

3. **Lightning Attention 的演进**

    **Lightning Attention-1**（来自论文：TransNormerLLM: A Faster and Better Large Language Model with Improved TransNormer）
    - **方法**：取消 Softmax，采用 Swish 激活函数，结合 Flash Attention 的分块计算（SRAM 优化）。
    - **局限**：复杂度仍为 $O(n^2)$，未解决因果模型根本问题。

    **Lightning Attention-2**（MiniMax 的工作：MiniMax-01: Scaling Foundation Models with Lightning Attention）
    - **核心创新**：分块处理序列，分为块内（intra-block）和块间（inter-block）注意力：
        - **块内**：传统左乘 + 掩码，保证单向性。
        - **块间**：缓存历史 KV 并右乘，实现 $O(nd^2)$ 复杂度。
    - **公式示例**：

        $$
        O_i = O_{intra} + O_{inter} = [(Q_i K_i^T) \odot M]V_i + \Lambda Q_i (KV)
        $$

        其中 $\Lambda$ 为位置编码对角矩阵。

4. **关键算法流程**

    前向传播：
    1. 分块加载 Q/K/V 到 SRAM。
    2. 计算块内注意力（带掩码）和块间注意力（利用缓存 KV）。
    3. 更新 KV 缓存并输出结果。

    反向传播：
    1. 分块计算梯度时，分别处理块内和块间贡献。
    2. 通过缓存机制高效传递梯度。

5. **总结**
    - **Lightning Attention-2** 首次实现因果模型的线性复杂度注意力，支持超长序列训练。
    - **核心贡献**：分块策略 + 缓存机制，平衡计算效率与模型性能。

</details>

**98. MoE 中的负载均衡问题是如何解决的？**

<details>
<summary>💡 查看答案</summary>

**问题**：MoE 中 Router 可能将大量 token 路由到少数几个专家，导致专家负载不均，部分专家过载而其他专家闲置，训练不稳定。
**常见解决方案**：
- **辅助损失（Auxiliary Loss）**：在训练损失中加入负载均衡损失项，鼓励各专家被均匀选择（如 GShard、Switch Transformer）
- **Expert Capacity**：设置每个专家的 token 处理上限，超出部分丢弃或传给下一专家
- **随机路由扰动**：训练时在路由得分上加少量噪声，防止模型过早锁死到固定专家
- **DeepSeek-V2 方案**：引入共享专家（Shared Expert）处理所有 token 的公共知识，路由专家只处理差异化内容，缓解单专家过载

</details>

**99. Layer Normalization 和 Batch Normalization 有什么区别？LLM 中为什么用 LayerNorm？**

<details>
<summary>💡 查看答案</summary>

**BatchNorm**：在 batch 维度上对每个特征做归一化，依赖 batch 内多个样本的统计量（均值、方差）。
**LayerNorm**：在单个样本的特征维度上做归一化，不依赖 batch 中其他样本。
**LLM 选择 LayerNorm 的原因**：
- LLM 处理变长序列，batch size 往往很小甚至为 1，BatchNorm 的统计量不稳定
- 自回归推理时每次只生成一个 token，无法计算跨样本的 batch 统计
- LayerNorm 对序列长度无要求，计算稳定，更适合 NLP 场景

</details>

**100. Pre-Norm 和 Post-Norm 有什么区别？各自的优缺点是什么？**

<details>
<summary>💡 查看答案</summary>

**Post-Norm**（原始 Transformer）：先做残差连接，再做 LayerNorm：`output = LayerNorm(x + sublayer(x))`
**Pre-Norm**（现代 LLM 主流）：先做 LayerNorm，再做残差连接：`output = x + sublayer(LayerNorm(x))`
\begin{table}[h]



| **维度** | **Post-Norm** | **Pre-Norm** |
| --- | --- | --- |
| 训练稳定性 | 较差，深层网络梯度容易消失/爆炸 | 好，梯度流更稳定 |
| 模型性能 | 收敛后效果略好 | 略弱于 Post-Norm（理论上） |
| 适用场景 | 层数较浅的模型 | 深层大模型（LLaMA、GPT 等均采用） |


\end{table}
主流 LLM 几乎都使用 Pre-Norm，因为其训练稳定性更重要。

</details>

**101. Layer Norm 的计算公式**

<details>
<summary>💡 查看答案</summary>

$$
    LayerNorm(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
$$

其中 $\mu$ 和 $\sigma^2$ 分别是沿特征维度计算的均值和方差，$\gamma$ 和 $\beta$ 是可学习的缩放和偏移参数。

</details>

**102. RMS Norm 的计算公式**

<details>
<summary>💡 查看答案</summary>

$$
    RMSNorm(x) = \gamma \cdot \frac{x}{\sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2 + \epsilon}}
$$

</details>

**103. RMS Norm 相比于 Layer Norm 有什么优点？**

<details>
<summary>💡 查看答案</summary>

**Layer Norm 的基本形式**如下：

对于输入向量 $\mathbf{x} \in \mathbb{R}^d$，Layer Norm 计算：
\[
\mu = \frac{1}{d}\sum_{i=1}^{d} x_i, \quad \sigma^2 = \frac{1}{d}\sum_{i=1}^{d}(x_i - \mu)^2
\]
\[
\hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}}, \quad LN(\mathbf{x}) = \gamma \odot \hat{\mathbf{x}} + \beta
\]
其中 $\gamma, \beta \in \mathbb{R}^d$ 为可学习的缩放和偏移参数。

**RMS Norm 的基本形式**如下：

RMS Norm 简化了 Layer Norm，去除了计算均值进行平移的部分，仅保留均方根（Root Mean Square）归一化：
\[
RMS(\mathbf{x}) = \sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2 + \epsilon}
\]
\[
RMSNorm(\mathbf{x}) = \frac{\mathbf{x}}{RMS(\mathbf{x})} \odot \gamma
\]
其中 $\gamma \in \mathbb{R}^d$ 为可学习的缩放参数，省去了 Layer Norm 中的偏移参数 $\beta$。

**RMS Norm 相比 Layer Norm 的优点**主要体现在以下几个方面：

1. **计算效率更高**：Layer Norm 需要计算均值 $\mu$ 和方差 $\sigma^2$ 两个统计量，而 RMS Norm 只需计算均方根一个统计量，省去了均值的计算和中心化操作，减少了约一半的统计量计算开销，在大规模模型中可以带来明显的训练与推理加速。

2. **参数量更少**：RMS Norm 去掉了偏移参数 $\beta$，参数量减少了一半（仅保留 $\gamma$），在超大规模模型中可节省一定的参数存储。

3. **效果基本相当甚至略有提升**：实验表明，去掉均值中心化操作对模型性能影响很小。RMS Norm 的论文（Zhang & Sennrich, 2019）在多个 NLP 任务上验证了 RMS Norm 与 Layer Norm 效果相当，且由于训练更稳定，部分场景下略有提升。

4. **实现简洁**：RMS Norm 的实现更加简洁，易于在定制硬件（如 TPU、NPU）上进行高效算子融合（Kernel Fusion）优化。

**直观理解**：Layer Norm 的设计初衷是同时消除激活值的均值偏移（re-centering）和方差偏移（re-scaling），但研究发现 re-centering 操作对最终性能的贡献有限，re-scaling 才是归一化效果的核心来源，因此 RMS Norm 去掉 re-centering 后仍然能保持良好性能。

</details>

**104. 为何理论上 Post-Norm 比 Pre-Norm 好？**

<details>
<summary>💡 查看答案</summary>

首先回顾 **Pre-Norm** 与 **Post-Norm** 的结构差异：

- **Post-Norm**（原始 Transformer 结构）：
    \[
    \mathbf{x}_{l+1} = Norm(\mathbf{x}_l + F_l(\mathbf{x}_l))
    \]
- **Pre-Norm**（现代大模型常用结构）：
    \[
    \mathbf{x}_{l+1} = \mathbf{x}_l + F_l(Norm(\mathbf{x}_l))
    \]

其中 $F_l(\cdot)$ 表示第 $l$ 层的子模块（如 Self-Attention 或 FFN）。

**Pre-Norm 的问题：退化为“浅而宽”的模型**

Pre-Norm 结构中，残差连接的恒等分支（identity branch）未经过任何归一化，因此随着网络层数加深，恒等映射的权重在梯度传播中始终保持较大的比例。具体地：

- 在 Pre-Norm 结构中，$\mathbf{x}_{l+1} = \mathbf{x}_l + F_l(Norm(\mathbf{x}_l))$，
    输出对输入的梯度为：
    \[
    \frac{\partial \mathbf{x}_{l+1}}{\partial \mathbf{x}_l} = \mathbf{I} + \frac{\partial F_l(Norm(\mathbf{x}_l))}{\partial \mathbf{x}_l}
    \]
    恒等矩阵 $\mathbf{I}$ 始终存在，梯度流可以直接绕过残差分支"跳过"多层，从而使模型过度依赖浅层的直连通路。
- 随着层数 $L$ 增大，底层（bottom layers）的梯度贡献相对减小，网络倾向于退化为一个等效深度较浅的模型，深层网络的表达能力未能充分发挥。

**Post-Norm 的优势：层数“足秤”**

Post-Norm 在每个子层的输出处进行归一化：
\[
\mathbf{x}_{l+1} = Norm(\mathbf{x}_l + F_l(\mathbf{x}_l))
\]

- Norm 操作对恒等分支和残差分支的输出之和进行归一化，每经过一次 Norm，恒等分支的相对权重就被压缩一次，从而**迫使残差分支**（即各层的非线性变换 $F_l$）发挥更大的作用。
- 这意味着 Post-Norm 中每一层都被“充分利用”，层数越深，模型的有效表达深度越接近实际层数，最终能达到更高的性能上限。

**训练难度的权衡**

- **Pre-Norm**：梯度可以通过恒等分支畅通无阻地回流，训练初期梯度稳定，收敛容易，适合直接训练极深网络，是目前大多数大语言模型（LLaMA、Qwen 等）的默认选择。
- **Post-Norm**：梯度必须经过 Norm 层，在训练初期容易出现梯度消失或爆炸问题，需要精细的学习率调度（如 Warmup）和初始化策略（如 T-Fixup、DeepNorm 等）才能稳定训练，但一旦收敛，效果通常优于同参数量的 Pre-Norm 模型。

**总结**：Pre-Norm 在同样参数配置下难以达到 Post-Norm 的性能上界，但训练简单稳定；Post-Norm 理论上可达到更好的性能上限，但训练过程中需要仔细监控梯度，收敛较为困难，需要配合专门的初始化或归一化策略（如 DeepNorm）。

</details>

**105. Deep Norm 思路是什么？**

<details>
<summary>💡 查看答案</summary>

**背景与动机**

Post-Norm 虽然效果好，但训练极深（如百层以上）的 Transformer 时极易出现梯度爆炸/消失，难以稳定收敛。Pre-Norm 虽然稳定，但如前所述存在性能上限较低的问题。DeepNorm（由微软研究院提出，ICML 2023）旨在融合两者优点：**兼具 Pre-Norm 的训练稳定性和 Post-Norm 的性能上限**。

**DeepNorm 的核心做法**

DeepNorm 对标准的 Post-Norm 结构进行了两处关键修改：

1. **残差连接的 Up-scale（$\alpha > 1$）**：在执行 Layer Norm 之前，对残差连接的恒等分支乘以一个大于 1 的系数 $\alpha$：
    \[
    \mathbf{x}_{l+1} = Norm(\alpha \cdot \mathbf{x}_l + F_l(\mathbf{x}_l))
    \]
    通过放大恒等分支，使得 Norm 层的输入在训练初期主要由 $\alpha \cdot \mathbf{x}_l$ 主导，从而稳定梯度传播，避免梯度爆炸。

2. **模型参数初始化的 Down-scale（$\beta < 1$）**：在初始化阶段，将模型中各子层（如 FFN、Attention 的输出投影矩阵等）的参数按系数 $\beta < 1$ 进行缩放初始化，使得训练初期残差分支 $F_l(\mathbf{x}_l)$ 的输出幅度较小，进一步稳定训练初期的更新。

**$\alpha$ 和 $\beta$ 的理论推导**

DeepNorm 论文中通过分析模型更新量（model update）的上界，给出了 $\alpha$ 和 $\beta$ 关于网络深度 $N$（Encoder 层数）和 $M$（Decoder 层数）的理论取值建议，例如对于仅含 Encoder 的网络：
\[
\alpha = (2N)^{1/4}, \quad \beta = (8N)^{-1/4}
\]
对于 Encoder-Decoder 结构，$\alpha$ 和 $\beta$ 的取值则同时依赖 $N$ 和 $M$。这些取值通过理论分析保证了训练过程中每一层的模型更新量（相对于初始化）被约束在一个常数范围内，从而实现稳定训练。

**DeepNorm 的优势**

- **训练稳定性**：通过 $\alpha$-scale 和 $\beta$-init，DeepNorm 可以稳定地训练**千层**（1000 层）的 Transformer，而标准 Post-Norm 在数十层时就可能出现训练不稳定问题。
- **性能接近 Post-Norm**：由于 DeepNorm 本质上仍是 Post-Norm 的变体，其性能上限接近标准 Post-Norm，优于 Pre-Norm。
- **实现简单**：DeepNorm 仅需修改残差系数和初始化策略，无需改变模型架构，易于工程实现。

**与其他方法的对比**

\begin{center}


| **方法** | **训练稳定性** | **性能上限** | **可扩展深度** |
| --- | --- | --- | --- |
| Post-Norm | 差 | 高 | 数十层 |
| Pre-Norm | 好 | 中 | 数百层 |
| DeepNorm | 好 | 高 | 千层以上 |


\end{center}

</details>

**106. 主流的大模型分别用了哪种 Layer Normalization？**

<details>
<summary>💡 查看答案</summary>

下表总结了主流大语言模型所采用的归一化方式：

\begin{center}


| **模型** | **Norm 位置** | **Norm 类型** | **备注** |
| --- | --- | --- | --- |
| LLaMA 系列 | Pre-Norm | RMSNorm | LLaMA 1/2/3 均采用此配置 |
| Qwen 系列 | Pre-Norm | RMSNorm | Qwen1/2/2.5 均采用此配置 |
| Mistral | Pre-Norm | RMSNorm | 与 LLaMA 结构相近 |
| GPT 系列 | Pre-Norm | LayerNorm | GPT-2 起改为 Pre-Norm |
| BERT | Post-Norm | LayerNorm | 原始 Transformer 结构 |
| 原始 Transformer | Post-Norm | LayerNorm | Vaswani et al., 2017 |


\end{center}

- **LLaMA**（Meta AI）：采用 **Pre-Norm**，归一化方式为 **RMSNorm**。
    LLaMA 系列（1/2/3）放弃了原始 LayerNorm 中的均值中心化操作，使用 RMSNorm 以提升训练速度；
    Pre-Norm 位置保证了极深网络的训练稳定性。

- **Qwen**（阿里巴巴）：采用 **Pre-Norm**，归一化方式为 **RMSNorm**。
    Qwen 系列（1/1.5/2/2.5）在整体架构上与 LLaMA 类似，均选择 Pre-Norm + RMSNorm 的组合，兼顾训练稳定性与计算效率。

- **Mistral**：采用 **Pre-Norm**，归一化方式为 **RMSNorm**。
    Mistral 架构高度参考 LLaMA，同样使用 Pre-Norm + RMSNorm。

- **GPT 系列**（OpenAI）：采用 **Pre-Norm**，归一化方式为标准 **LayerNorm**。
    GPT-1 原始论文使用 Post-Norm，从 GPT-2 开始改为 Pre-Norm 以稳定大规模训练，但仍使用完整的 LayerNorm（包含均值和方差）。

- **BERT**（Google）：采用 **Post-Norm**，归一化方式为标准 **LayerNorm**。
    BERT 沿用原始 Transformer 的 Post-Norm 结构，在每个子层（Self-Attention、FFN）输出后执行 LayerNorm。

**趋势分析**：目前主流大语言模型（特别是生成式 Decoder-only 模型）普遍采用 **Pre-Norm + RMSNorm** 的组合，原因在于：
1. Pre-Norm 训练稳定，适合数十亿乃至数千亿参数规模的超大模型训练；
2. RMSNorm 计算效率更高，省去均值计算，在大规模训练中可节省可观的计算资源；
3. 两者结合在工程实践中已被 LLaMA、Qwen、Mistral 等广泛验证，成为事实上的行业标准。

</details>

**107. KV Cache 是什么？它是如何加速推理的？**

<details>
<summary>💡 查看答案</summary>

**背景：自回归生成的重复计算问题**

LLM 在推理阶段采用自回归（Auto-Regressive）方式逐 token 生成，即每次生成一个新 token，再将其拼接到输入序列中，作为下一步的输入。标准 Self-Attention 的计算公式为：
\[
Attention(Q, K, V) = softmax\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
\]
其中 $Q, K, V \in \mathbb{R}^{N \times d_k}$，$N$ 为当前序列长度。

在第 $t$ 步生成时，序列长度为 $t$，需要对所有 $t$ 个 token 重新计算 $K$ 和 $V$ 矩阵。而第 $t-1$ 步已经计算过前 $t-1$ 个 token 的 $K$、$V$，这部分计算被完全丢弃，造成大量**重复计算**。若序列总长度为 $N$，朴素实现下整个生成过程的总计算量为：
\[
\sum_{t=1}^{N} O(t \cdot d_k) = O(N^2 \cdot d_k)
\]

**KV Cache 的原理**

KV Cache 的核心思想是：**将每一层 Attention 中已计算过的历史 token 的 $K$、$V$ 向量缓存下来，生成新 token 时直接复用，避免重复计算**。

具体流程如下：

1. **Prefill 阶段（预填充）**：输入 Prompt 中所有 token 一次性并行计算各层的 $K$、$V$，将结果存入 KV Cache。
2. **Decode 阶段（逐步解码）**：每生成一个新 token $x_t$，仅计算该 token 的 $Q_t$、$K_t$、$V_t$；将 $K_t$、$V_t$ **追加**到 KV Cache 中；然后用 $Q_t$ 与 Cache 中所有历史 $K$、$V$ 做 Attention：
    \[
    Attention(Q_t,\ K_{1:t},\ V_{1:t}) = softmax\!\left(\frac{Q_t K_{1:t}^\top}{\sqrt{d_k}}\right)V_{1:t}
    \]

**加速推理的效果**

使用 KV Cache 后，每一步 Decode 只需计算当前新 token 的 $Q$、$K$、$V$（计算量为 $O(d_k)$），以及 $Q_t$ 与长度为 $t$ 的 Cache 做向量-矩阵乘法（计算量为 $O(t \cdot d_k)$）。整个生成过程的总计算量变为：
\[
\sum_{t=1}^{N} O(t \cdot d_k) \;\longrightarrow\; 每步仅增量计算，避免对历史 token 的重复计算
\]
即**每步生成的计算量从 $O(N^2)$ 降至 $O(N)$**（对于单步而言），显著加速推理过程。KV Cache 已成为 LLM 推理的标配优化，几乎所有主流推理框架（vLLM、TensorRT-LLM、HuggingFace TGI 等）均默认启用。

**KV Cache 的显存开销分析**

KV Cache 的代价是显存占用随序列长度线性增长。设模型共有 $L$ 层，每层 Attention 的隐藏维度为 $d$，数据类型为 FP16（每个元素 2 字节），则缓存一条长度为 $N$ 的序列所需显存为：
\[
KV Cache 大小 = 2 \times L \times N \times d \times 2  字节
\]
（系数 2 来自 $K$ 和 $V$ 各一份）。

以 LLaMA-2 70B 为例（$L=80$，$d=8192$，FP16），缓存长度为 4096 的序列约需：
\[
2 \times 80 \times 4096 \times 8192 \times 2 \approx 8.6  GB
\]
在长序列或大 batch 推理时，KV Cache 的显存压力非常显著。

**KV Cache 压缩方案**

为缓解 KV Cache 的显存压力，学界和工业界提出了多种压缩方案：

- **MQA（Multi-Query Attention）**：所有 Query head 共享同一组 $K$、$V$，将 KV Cache 压缩至 $1/h$（$h$ 为 head 数），代价是略微损失模型精度。Falcon、PaLM 等模型采用此方案。

- **GQA（Grouped-Query Attention）**：将 $h$ 个 Query head 分为 $g$ 组，每组共享一对 $K$、$V$，KV Cache 压缩至 $g/h$，在 MQA 与 MHA 之间取得平衡。LLaMA-2 70B、Qwen 系列等采用此方案。

- **PagedAttention（vLLM）**：借鉴操作系统虚拟内存的分页思想，将 KV Cache 以固定大小的“页”（Block）为单位进行非连续存储和动态分配，避免显存碎片，大幅提升 GPU 显存利用率和并发吞吐量。

- **滑动窗口 / 稀疏 Attention**：仅缓存最近 $W$ 个 token 的 KV（如 Mistral 的 Sliding Window Attention），以有限的上下文窗口换取固定大小的 KV Cache。

- **KV Cache 量化**：将 KV Cache 从 FP16 量化为 INT8 或 INT4，直接减少一半乃至四分之三的显存占用，如 KVQuant、KIVI 等方法。

</details>

