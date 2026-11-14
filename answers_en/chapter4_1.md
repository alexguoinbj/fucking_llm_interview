# 4.1 Basic Concepts of Fine-Tuning and Method Selection — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 8 questions

**1. Suppose I Have an LLM Model, How Can I Customize It Using Data Based on My Requirements?**

<details>
<summary>💡 Show Answer</summary>

![LLM Customization Strategy Selection Matrix (Prompt Engineering vs. RAG vs. Fine-Tuning vs. Pre-Training)](https://img.pricenode.uk/llm_interview_questions/english/chapter4/llm_customization_matrix.png)




The four primary architectural patterns for customizing Large Language Models (LLMs) to meet specific needs include: Prompt Engineering, Retrieval-Augmented Generation (RAG), Fine-tuning, and Pre-training. These methods are not mutually exclusive; rather, they complement each other and can often be combined to achieve optimal results.

- If you have limited data and insufficient compute resources, Prompt Engineering is a good starting point.
- If you need more comprehensive responses and have some relevant data, consider using RAG.
- If accuracy is paramount and you possess a large dataset, Fine-tuning is a powerful choice.
- If your application scenario is highly specialized, and you have massive domain-specific data along with substantial compute resources, Pre-training is the most ideal choice.

</details>

**2. When Should Fine-Tuning Be Used Instead of RAG?**

<details>
<summary>💡 Show Answer</summary>

Fine-tuning is the perfect choice when you need to focus on a specific task. Here are some scenarios where fine-tuning is superior to RAG:
- Is your task in a niche domain such as medicine or law? Fine-tuning allows you to customize the model to the terminology and complexity of that specific domain.
- Do you only have a small amount of task-specific data? Fine-tuning leverages the knowledge of pre-trained LLMs as a starting point, improving efficiency and preventing overfitting.
- Does your task require precise output, such as sentiment analysis or classification? Fine-tuning excels at identifying specific patterns within a specific domain.

</details>

**3. How Does Task-Specific Fine-Tuning Differ from Zero-Shot Learning in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Task-specific fine-tuning adapts an LLM to a specific task by further training it on a dataset dedicated to that task. This enables the model to:
- **Improve Accuracy**: Fine-tuning optimizes model performance for narrow tasks (such as sentiment analysis or named entity recognition), achieving higher accuracy compared to zero-shot learning.
- **Specialization**: By focusing on task-specific data, the model becomes highly proficient in the target task, though it may lose some generalization ability.
In contrast, zero-shot learning leverages the model's broad pre-trained knowledge without requiring additional task-specific training, making it more versatile, but potentially less precise on specific tasks.

</details>

**4. What Is Zero-Shot Learning, and How Is It Applied to LLMs?**

<details>
<summary>💡 Show Answer</summary>

Zero-shot learning enables LLMs to perform tasks they were not explicitly trained on by leveraging their extensive knowledge of language and general concepts. Rather than relying on task-specific fine-tuning, it allows the model to understand and generate relevant responses based on the context and instructions in the prompt. For example:

- **Text Classification**: A model trained on general language data can classify text into different categories when provided with appropriate prompts, without being specifically trained on that task.
- **Translation or Summarization**: LLMs can perform translation or summarization without fine-tuning for these tasks, using only the instructions provided in the input.
Zero-shot learning demonstrates the ability of LLMs to generalize across tasks, allowing them to adapt to a wide range of application scenarios.

</details>

**5. To Teach a Model Domain-Specific Knowledge, Should You Choose Pre-Training or Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

- **Choose Continued Pre-Training (Continue PreTrain)**: Suitable when the target domain's language distribution differs significantly from general corpora, and the general model has weak language modeling capabilities in that domain (e.g., code, Chinese law, medical literature, etc.).
- **Choose Supervised Fine-Tuning (SFT)**: Suitable when the general model already possesses the domain knowledge and only needs adjustment in output formatting and instruction-following behavior.
- **Choose Retrieval-Augmented Generation (RAG)**: Suitable when knowledge changes frequently or private data should not be trained into the model directly, offering the lowest cost.
Recommended general pipeline: General Base Model $\rightarrow$ Continued Pre-Training (Optional) $\rightarrow$ SFT $\rightarrow$ RLHF/DPO

</details>

**6. How Do You Estimate the Infrastructure Requirements for Fine-Tuning an LLM?**

<details>
<summary>💡 Show Answer</summary>

Accurately estimating the infrastructure required to fine-tune an LLM requires careful consideration of multiple factors. The primary driving factors include the size and complexity of the model you are working with, as well as the desired training speed.

Large models with billions of parameters naturally require greater computational power and memory. This means you will need powerful Graphics Processing Units (GPUs) or Tensor Processing Units (TPUs), potentially in multi-unit configurations. On the other hand, if achieving faster training times is critical, you will need to scale your resources further. This may involve using multiple GPUs or TPUs working in parallel, or leveraging cloud solutions that offer on-demand scalability. It is important to keep in mind that all these choices come with budget constraints. Consider exploring quantization techniques, which can reduce model size and thereby lower computational requirements.

</details>

**7. How Much GPU Memory Is Required for Full Fine-Tuning? How Can It Be Estimated?**

<details>
<summary>💡 Show Answer</summary>

![GPU VRAM Allocation Breakdown for FP16 Full Fine-Tuning (Model Weights, Gradients, Adam Optimizer States, and Activations)](https://img.pricenode.uk/llm_interview_questions/english/chapter4/gpu_memory_ft_breakdown.png)




Rough estimation formula (mixed-precision training):
**GPU Memory $\approx$ Number of Parameters $\times$ Bytes per Parameter $\times$ Factor**
- Model Weights (FP16): Number of Parameters $\times$ 2 bytes
- Gradients (FP16): Number of Parameters $\times$ 2 bytes
- Optimizer States (Adam, FP32): Number of Parameters $\times$ 8 bytes (4 bytes each for 1st momentum and 2nd momentum)
- Total approximately: Number of Parameters $\times$ 16 bytes
**Example**: Full fine-tuning of a 7B parameter model $\approx 7 \times 10^9 \times 16 \approx 112$~GB, which requires multiple A100 GPUs (80GB).
In practice, activation memory must also be added; Gradient Checkpointing is commonly used to trade computation time for memory space.

</details>

**8. In Fine-Tuning Tasks, Which Layer Weights Should Be Frozen? What Are the Differences Between Fine-Tuning Lower Encoder Layers, Higher Encoder Layers, and Feed-Forward Neural Network Layers?**

<details>
<summary>💡 Show Answer</summary>

![Hierarchical Transformer Layer Freezing Strategy (Lower Layers vs. Higher Layers vs. Feed-Forward Neural Networks)](https://img.pricenode.uk/llm_interview_questions/english/chapter4/transformer_layer_freezing.png)




The core rationale for freezing strategies is that features learned by different Transformer layers exhibit a hierarchical structure ranging from "general to specific":

- **Lower Layers (First Few Layers)**: Typically learn lower-level, general features (such as lexical items, local syntax, and basic semantic patterns). These features exhibit strong transferability across different tasks and domains and are usually frozen first to avoid damaging the general representations learned during pre-training, while reducing trainable parameters and mitigating overfitting risks (especially when data volume is small).
- **Higher Layers (Last Few Layers)**: Closer to the output, learning high-level semantic/task-specific features closely tied to the specific task. They play the largest role in downstream task adaptation and are therefore typically the focus of fine-tuning, requiring unfreezing and training.
- **Feed-Forward Neural Network Layers (FFN)**: FFN layers account for a large proportion of parameters in a Transformer block (often far exceeding the Attention component) and act more as "feature transformers / memory storage". Studies show that FFNs store a vast amount of specific knowledge; fine-tuning FFN layers usually has a greater impact on adapting the model to new knowledge/tasks, but the large parameter size also implies higher overfitting risk and computational overhead.

**Common Strategies in Practice**:
- Very small data volume: Freeze most lower layers, fine-tune only the top 1--2 layers + task head (e.g., classification head), sometimes combined with PEFT methods (such as LoRA) to adjust very few parameters;
- Medium data volume: Unfreeze several top layers (e.g., the upper 1/3), while keeping the lower layers frozen;
- Sufficient data volume and large domain gap: Full parameter fine-tune, but typically apply a smaller learning rate to lower layers (layer-wise learning rate decay) and a larger learning rate to top layers, reflecting the intuition that "lower layer changes should be small while top layer changes can be larger."

The overall principle is: **The closer the frozen layers are to the input, the better general capabilities are preserved; the closer the unfrozen layers are to the output/FFN, the stronger the task adaptation capability**. A comprehensive balance must be struck based on dataset size, domain gap, and overfitting risk.

</details>

