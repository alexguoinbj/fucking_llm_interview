# 4.8 SFT Fine-Tuning Practice — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 12 questions

**1. What is SFT Packing?**

<details>
<summary>💡 Show Answer</summary>

![SFT Sequence Packing vs. Traditional Padding Token Efficiency and GPU VRAM Throughput](https://img.pricenode.uk/llm_interview_questions/english/chapter4/sft_packing_token_efficiency.png)




SFT Packing (Supervised Fine-Tuning Packing) is a technique for efficiently utilizing computational resources during the supervised fine-tuning stage. Its core idea is to concatenate (pack) multiple training samples (especially short texts) into a single continuous sequence, thereby reducing the computational waste caused by traditional padding.

**Direct Concatenation**: Directly connect multiple samples with a delimiter (such as `<eos>`) between them. For example, sample A (“Hello”) and sample B (“How are you?”) are packed into `Hello<eos>How are you?<eos>`.

**Advantages**: Zero computational waste, no padding, and all tokens are valid; consistency with pre-training, as models usually process continuous text during pre-training.

**Potential Noise**: The model might learn spurious correlations between samples, though the practical impact is generally minor.

</details>

**2. What is the Impact of SFT Packing on SFT Training?**

<details>
<summary>💡 Show Answer</summary>

Through packing technology, training efficiency is improved. This technique can integrate several short training samples into a single sequence, optimizing GPU utilization efficiency and reducing padding waste. However, it may also induce potential attention interaction issues, such as unexpected attention interactions occurring between different samples, which can lead to information leakage, especially when the attention mask is not set correctly.

Experimental findings: When the data volume is small or when facing specific complex tasks, SFT packing can negatively affect generalization ability; however, in large-scale data scenarios, it causes no harm.

</details>

**3. Why Does the LLM Seem to Become "Dumber" After SFT?**

<details>
<summary>💡 Show Answer</summary>

After supervised fine-tuning (SFT), it is sometimes observed that the language model's performance degrades or exhibits "dumb" behavior. Possible reasons include:
1. **Data Bias**: The dataset used for SFT may differ in distribution from the pre-training data. If the discrepancy is large, the model may perform poorly on new tasks.
2. **Annotation Errors**: Annotations in the fine-tuning dataset may contain errors or inaccuracies, which negatively affect model performance.
3. **Overfitting**: If the fine-tuning dataset is small or the model has too many parameters, the model may overfit, leading to poor performance on new inputs.
4. **Lack of Data Diversity**: The fine-tuning dataset may fail to cover the variety of input scenarios the model might encounter in new tasks.

</details>

**4. Which Hyperparameters Should Be Selected and Optimized When Fine-Tuning Large Models?**

<details>
<summary>💡 Show Answer</summary>

When tuning large models, substantial adjustments to hyperparameters are usually unnecessary:
- GPU VRAM capacity is the key factor determining the batch size, and the context length is generally recommended to be set to 2048 or higher;
- The learning rate is generally recommended to use a cosine annealing schedule and be appropriately adjusted according to the batch size;
- When performing gradient clipping, the threshold is typically set around 0.3, and the weight decay value can be chosen as 0.1 or 0;
- Apart from the above parameters, other parameters generally do not require adjustment.

</details>

**5. What Problems Occur If the Batch Size Is Set Too Small When Fine-Tuning Large Models?**

<details>
<summary>💡 Show Answer</summary>

When the batch size is small, the direction of gradient updates has high variance, causing updates to be primarily noise-driven. Although the variance cancels out over multiple updates to push the model in the correct direction, individual updates may be ineffective. In contrast, using a larger batch size enables more effective updates.

</details>

**6. What Problems Occur If the Batch Size Is Set Too Large When Fine-Tuning Large Models?**

<details>
<summary>💡 Show Answer</summary>

When the batch size is extremely large, any two batches of data drawn from the training set will be very similar because they both almost completely match the true gradient. In this scenario, increasing the batch size yields almost no performance improvement. Furthermore, although using a larger batch size reduces the number of training steps, overall training time barely improves while computational resource consumption increases significantly. Research by Google shows that under a fixed epoch budget, when the batch size exceeds a certain critical threshold, model performance degrades as the batch size increases.

</details>

**7. How to Choose Optimizers When Fine-Tuning Large Models?**

<details>
<summary>💡 Show Answer</summary>

Besides the commonly used Adam and AdamW, other optimizers such as Sophia are also worth studying. Sophia uses gradient curvature instead of variance for normalization, which may improve training efficiency and model performance.

</details>

**8. Some Experimental Settings for Fine-Tuning Large Models?**

<details>
<summary>💡 Show Answer</summary>

- **Model Architecture**: Adopt Causal Decoder + LM, which possesses strong zero-shot and few-shot capabilities and can demonstrate emergent abilities.
- **Layer Normalization**: Use Pre-RMSNorm.
- **Activation Function**: GeGLU or SwiGLU is recommended.
- **Embedding Layer**: Do not add Layer Normalization after the embedding layer to avoid negatively affecting LLM performance.
- **Positional Encoding**: Use RoPE or ALiBi, with RoPE being more widely adopted.

</details>

**9. How to Resolve Abnormally High Loss Spikes During Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

During pre-training, abnormally high loss spikes are often difficult to diagnose directly and may require adjusting the learning rate or introducing regularization terms to resolve. If an abnormal loss spike is encountered during fine-tuning, it may be caused by insufficient data volume. It is recommended to use mature frameworks and make appropriate adjustments to the learning rate and gradient clipping values.

</details>

**10. How to Prevent or Mitigate Overfitting During Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

To prevent or mitigate overfitting, it is recommended to select large models after data deduplication and train for only one epoch. For large models, overfitting is usually not a major concern; if overfitting occurs, one can try using different data formats or adding more high-quality data.

</details>

**11. What Issues Need Attention in Long-Context Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Long-context fine-tuning requires attention to the following issues:

**1. VRAM and Computational Overhead**: Attention computational complexity grows quadratically with sequence length, requiring integration with efficient Attention implementations such as activation recomputation (gradient checkpointing) and FlashAttention.

**2. Positional Encoding Extrapolation Issue**: Models are typically trained with a fixed maximum length during pre-training. If sequences exceeding the pre-training length need to be processed during fine-tuning, positional encoding performance may degrade. Common solutions include Position Interpolation (PI) and adjusting RoPE base frequency parameters (NTK-aware scaling, YaRN, etc.).

**3. Length Distribution of Training Data**: Fine-tuning data needs to include a sufficient quantity and proportion of long-text samples; care should be taken to avoid computational waste caused by extensive padding.

**4. "Needle in a Haystack" Capabilities**: In fine-tuning data design, samples where "key information is distributed across different positions in the text" can be consciously constructed to improve the model's balanced utilization of information across all positions in long texts.

</details>

**12. Based on the Open-Source Llama-3 70B Model, How to Fine-Tune the Model to Make Its Output Style Conciser, More Like WeChat Chatting, and Ensure the Content Complies with Chinese LLM Safety Requirements?**

<details>
<summary>💡 Show Answer</summary>

This is a dual-objective fine-tuning task of "style transfer + safety compliance":

**1. Data Construction**:

*Style Data*: Collect dialogue samples conforming to the "WeChat chat" style (short sentences, colloquial language, segmented messaging), and use LLMs to rewrite existing long-response styles into concise, colloquial versions.

*Safety & Compliance Data*: Construct refusal/proper guidance samples covering various sensitive topics (political sensitivity, violence, pornography, illegal acts, etc.), covering both direct requests and "disguised/bypass" requests.

**2. Training Strategy**: Mix "stylized general task data" and "safety compliance data" in a certain ratio for SFT; use methods like QLoRA to complete fine-tuning under acceptable hardware resources; take care not to weaken the model's inclination to reject sensitive requests due to a large amount of "casual" chat content in the style data.

**3. Evaluation**: Style evaluation (average response length, sentence count distribution, LLM-as-Judge); safety evaluation (Chinese safety evaluation benchmark + jailbreak testing).

% ============================================================

</details>

