# 4.6 Multi-Task and Continual Learning Fine-Tuning — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 6 questions

**1. How to Mitigate Catastrophic Forgetting in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Catastrophic forgetting refers to an LLM completely forgetting or significantly weakening its mastery of previously learned tasks when learning new information. Several techniques can address this challenge:
- **Rehearsal-Based Methods**: Allow the LLM to review subsets of both old and new data simultaneously during training.
- **Elastic Weight Consolidation (EWC)**: Assign importance scores to different weights in the LLM network and penalize changes to parameters critical to old tasks.
- **Modular Approaches**: Introduce independent modules for new tasks, allowing the LLM to expand its knowledge base without completely rewriting existing connections.

</details>

**2. Model Capabilities Degrade After Fine-Tuning: What is Catastrophic Forgetting?**

<details>
<summary>💡 Show Answer</summary>

Catastrophic Forgetting refers to the phenomenon where, during fine-tuning, a model trained on a new task may forget previously learned knowledge, leading to a performance drop on old tasks. Its causes include:
1. **Data Distribution Shift**: The data distribution of the new task differs significantly from the pre-training data or old task data, causing the model to over-adjust and forget old task knowledge.
2. **Parameter Update Conflict**: Gradient updates from the new task may conflict with gradient updates from old tasks, leading to the overwriting of old task knowledge.

Methods to mitigate catastrophic forgetting include: replay buffers, elastic weight sharing, continual/incremental learning, multi-task learning, knowledge distillation, regularization techniques, etc.

</details>

**3. How to Avoid Catastrophic Forgetting During Incremental Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Methods to avoid catastrophic forgetting during incremental fine-tuning (continuing to fine-tune on a previously fine-tuned model for new tasks/data) include:

**1. Data Rehearsal / Replay**: Mixing a portion of old task data into the training data of the new task is one of the most direct and effective ways to mitigate forgetting.

**2. Parameter-Efficient Fine-Tuning (PEFT) and Modularization**: Use methods like LoRA to train independent add-on modules for new tasks while keeping original model parameters frozen and unaffected; independent LoRA modules can be maintained for different tasks and dynamically loaded/swapped during deployment.

**3. Regularization to Constrain Parameter Changes**: EWC (Elastic Weight Consolidation) imposes penalties on important parameters, limiting them from deviating too much from original values during new task training; L2 regularization / parameter distance constraints.

**4. Knowledge Distillation Constraints**: Ensure that the output distribution of the incrementally fine-tuned model on inputs related to old tasks remains as close as possible to that of the model before incremental fine-tuning.

**5. Smaller Learning Rates and Fewer Training Steps**: Reduce the magnitude of parameter updates to minimize damage to existing capabilities.

**6. Multi-Task Joint Training**: Conduct multi-task training with new and old task data together, fundamentally eliminating the forgetting problem caused by sequential training order.

**7. Evaluation and Monitoring**: Periodically evaluate on the old task test set during incremental fine-tuning, and promptly adjust strategies if a noticeable degradation is detected.

</details>

**4. How to Balance Gradients Across Different Tasks in Multi-Task Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

![Multi-Task Gradient Conflict Resolution via PCGrad Orthogonal Projection and GradNorm Dynamic Balancing](https://img.pricenode.uk/llm_interview_questions/english/chapter4/multitask_gradient_balancing.png)




In Multi-Task Fine-tuning, loss scales, difficulty, and data volume across different tasks can vary significantly. Common balancing strategies include:

**1. Loss Weighting**: Fixed weighting (manually set based on task importance); uncertainty weighting (dynamically learning the relative weight of each task).

**2. Controlling Data Sampling Ratios**: Adjust sampling ratios using a "temperature" based on task data volume (e.g., sampling proportionally to the square root of data volume), allowing tasks with smaller data volumes to receive sufficient training signals.

**3. Gradient Normalization / Gradient Balancing Methods**: GradNorm (dynamically adjusts weights of each task's loss so that task gradient norms tend to align); PCGrad (Projected Conflicting Gradients, projects conflicting gradients onto directions orthogonal to each other to eliminate direct conflicts).

**4. Staged / Curriculum Training**: Gradually introduce tasks in a certain order, or adopt a two-stage strategy of "single-task warmup first, then multi-task joint training".

**5. Design of Shared and Task-Specific Parameters**: Differentiate between "shared parameters" and "task-specific parameters" in the model architecture to reduce direct gradient conflicts between tasks.

**6. Monitoring Independent Metrics for Each Task**: Separately monitor changes in loss and metrics for each task on its validation set, and adjust strategies promptly if task interference is observed.

</details>

**5. What Should Be Paid Attention to When Mixing Instruction Data and Dialogue Data for Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

When mixing "single-turn instruction data" and "multi-turn dialogue data" for fine-tuning, the following issues need attention:

**1. Format Unification**: Convert all data into a unified format (typically the multi-turn dialogue `messages` format) and apply a uniform chat template.

**2. Consistency of System Prompt**: Add a uniform default system prompt to all samples to avoid conflicts between different system prompts.

**3. Context Dependency in Multi-Turn Dialogue**: Ensure during mixed training that the model can handle both "single-turn instructions without historical context" and "dialogues relying on multi-turn context history".

**4. Consistency of Loss Calculation**: Compute loss only on content from the `assistant` role for all data types to guarantee training objective consistency.

**5. Balancing Data Ratios and Task Capabilities**: Determine the mixing ratio of the two data types according to the practical requirements of target application scenarios.

**6. Disparity in Length Distributions**: The sequence length of multi-turn dialogue data is typically significantly longer than single-turn instruction data; pay attention to batching strategies to avoid padding waste caused by excessive length differences.

</details>

**6. What are the Techniques for Multi-Turn Dialogue Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Practical techniques for multi-turn dialogue fine-tuning include:

1. **All-Turn Loss Calculation**: Calculate loss on the assistant's response in every turn of multi-turn dialogue, significantly improving data utilization efficiency.
2. **Context Window and Truncation Strategy**: Prioritize retaining the system prompt and the most recent dialogue turns; for exceptionally long dialogues, truncate contiguous segments of several turns as independent training samples.
3. **Role Consistency and Persona Handling**: Ensure that the response style and persona/pronoun usage of the `assistant` role remain consistent throughout the training data.
4. **Covering Coreference Resolution and Context Dependencies**: Deliberately include responses that require preceding context to be correctly understood, training the model's context-tracking capability.
5. **Topic Switching and Error Recovery**: Include sample scenarios such as users switching topics, users correcting model errors, or the model proactively asking for clarification.
6. **Training-Inference Alignment**: Ensure that the chat template used during training is strictly aligned with the one used during inference deployment.

% ============================================================

</details>

