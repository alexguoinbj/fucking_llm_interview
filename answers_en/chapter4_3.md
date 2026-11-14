# 4.3 Instruction Fine-Tuning (SFT) and Data Construction — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 13 questions

**1. What is the Difference Between Pre-Training and SFT Operations?**

<details>
<summary>💡 Show Answer</summary>

![Pre-Training vs. Supervised Fine-Tuning (SFT) Operational and Loss Masking Pipeline](https://img.pricenode.uk/llm_interview_questions/english/chapter4/pretrain_vs_sft_pipeline.png)




- **Goal**: Pre-training learns general language representations and linguistic knowledge from large-scale text corpora through unsupervised learning; SFT further optimizes model performance on specific tasks.
- **Data**: Pre-training uses large-scale unlabeled text data; SFT uses labeled task-relevant data.
- **Training Approach**: Pre-training adopts an unsupervised learning approach; SFT adopts a supervised learning approach.
- **Model Adaptation**: Pre-training starts training from randomly initialized parameters; SFT usually takes the parameters of a pre-trained model as initialization.

</details>

**2. What Does a Large Language Model (LLM) Learn During SFT?**

<details>
<summary>💡 Show Answer</summary>

During supervised fine-tuning (SFT) of large language models, the model mainly learns the following content:
1. **Task-Specific Label Prediction**: The model learns to predict correct labels or target outputs based on input text.
2. **Contextual Understanding and Language Patterns**: The model leverages language knowledge learned during pre-training to further understand task-relevant contextual information.
3. **Feature Extraction and Representation Learning**: The model learns more effective feature representations through task data to better distinguish different categories or achieve task goals.
4. **Task-Relevant Optimization**: The model adjusts parameters via backpropagation and optimization algorithms to minimize task-relevant loss functions.
5. **Adaptation to Specific Domains or Styles**: If the task data originates from a specific domain or possesses a specific style, the model learns to adapt to linguistic expression styles of those domains or styles.

</details>

**3. Can New Knowledge Be Learned During the SFT Phase?**

<details>
<summary>💡 Show Answer</summary>

During the SFT phase, the model can learn new knowledge to a certain extent, but this learning is an optimization and extension based on pre-trained knowledge rather than starting completely from scratch.

**Cases where new knowledge can be learned**: If the SFT data contains patterns, regularities, or features different from those in the pre-training phase, the model can acquire new knowledge by learning from this data. For example, through domain-specific data fine-tuning, the model can learn specialized terminology and logical relationships within that domain.

**Cases with limited learning capacity**: If the new knowledge is completely unrelated to the knowledge from the pre-training phase or exceeds the model's comprehension capacity, it is difficult for the model to truly master this new knowledge. Training in the SFT phase fine-tunes parameters based on the pre-trained model, so the model's learning capacity is constrained by the pre-trained parameters.

</details>

**4. What Are the Benefits of Instruction Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

The benefits of instruction fine-tuning in large model training include:
1. **Personalized Adaptation**: Through instruction fine-tuning, the model can be optimized using data from specific tasks or domains to better adapt to task requirements.
2. **Performance Enhancement**: Through instruction fine-tuning, model parameters and structures can be adjusted to boost task performance.
3. **Controlling Model Behavior**: Through instruction fine-tuning, specific instructions or constraints can be introduced to align model behavior with task needs.
4. **Data Efficiency**: Through instruction fine-tuning, pre-trained knowledge from general data can be leveraged alongside a small amount of task-specific data for fine-tuning, saving data resources.
5. **Improving Training Efficiency**: Through instruction fine-tuning, fine-tuning can be conducted on top of a pre-trained model, avoiding training from scratch and saving time and resources.

</details>

**5. How to Construct Fine-Tuning Data for SFT?**

<details>
<summary>💡 Show Answer</summary>

![End-to-End 7-Stage Pipeline for Supervised Fine-Tuning (SFT) Data Construction and Quality Audit](https://img.pricenode.uk/llm_interview_questions/english/chapter4/sft_data_construction_pipeline.png)




Constructing a supervised fine-tuning (SFT) dataset involves the following steps:
1. **Collect Raw Data**: First, gather data relevant to the target task, such as dialogue data or classification data, ensuring representative and diverse data.
2. **Annotate Data**: Provide correct labels or target outputs for each sample, ensuring annotation accuracy and consistency.
3. **Split Dataset**: Partition the data into training, validation, and test sets for model training, hyperparameter tuning, and final evaluation, respectively.
4. **Data Preprocessing**: Clean, tokenize, remove stop words, or perform other processing according to task needs to make the data suitable for model input.
5. **Format Conversion**: Convert the data into formats suitable for model training, such as text files or JSON format.
6. **Model Fine-Tuning**: Fine-tune the base model using the processed data, selecting appropriate fine-tuning methods and hyperparameters.
7. **Model Evaluation**: Evaluate the fine-tuned model performance using the test set and further optimize the model based on results.

</details>

**6. How Should a Fine-Tuning Dataset Be Constructed?**

<details>
<summary>💡 Show Answer</summary>

Constructing a high-quality fine-tuning dataset requires attention to multiple aspects, including data sources, data quality, data diversity, and data formatting:

**1. Data Sources**

- **Human Annotation**: Controllable quality but higher cost, suitable for critical scenarios or seed data;
- **Existing Public Datasets**: Utilize open-source datasets relevant to the task;
- **Business Logs / Historical Data**: Real user inputs and ideal outputs collected from production systems, which closely match the real-world distribution;
- **LLM-Generated Data (Synthetic Data)**: Use stronger models to generate large volumes of training samples, requiring subsequent quality filtering;
- **Data Augmentation**: Rewrite or expand existing data.

**2. Data Quality Control**

- **Deduplication**: Avoid having large amounts of near-duplicate samples in the dataset;
- **Denoising / Filtering Low-Quality Samples**: Filter out samples with formatting errors, empty content, or blatantly incorrect answers;
- **Removal of Sensitive / Harmful Content**;
- **Answer Accuracy Verification**: For factual tasks, ensure that the annotated answers themselves are correct.

**3. Data Diversity**

- Cover various sub-scenarios and edge cases of the task;
- Diversity in instruction phrasing to improve the model's generalization ability across different expressions;
- Include samples ranging from simple to complex difficulty levels.

**Summary**: Data quality and diversity often exert a greater impact on the final outcome than sheer data quantity; "few but refined" is usually superior to "many but cluttered".

</details>

**7. How Is Instruction Fine-Tuning Data Formatted in General?**

<details>
<summary>💡 Show Answer</summary>

**1. Basic Triplet Format**

The most classic format (such as the Alpaca format) contains three fields: `instruction` (task instruction), `input` (optional specific input), and `output` (expected model response).

**2. Dialogue Format (Multi-turn / Chat Format)**

For conversational models, a "message list" format is typically adopted, where each message contains a role (`role`) and content (`content`), encompassing three roles: `system`, `user`, and `assistant`.

**3. Loss Calculation Scope (Loss Masking)**

In instruction fine-tuning, **loss is usually calculated only for the tokens in the assistant (model reply) portion**, while loss is not calculated for tokens in the `instruction`/`input`/`system`/`user` portions (labels are set to -100).

**4. Special Tokens and End-of-Sequence Symbols**

Dialogue turns are typically separated by special role-marker tokens and end-of-sequence symbols, ensuring that during inference the model can correctly determine "when to end generation for the current turn."

</details>

**8. What Impact Does It Have If the Fine-Tuning Data Template Lacks the End-of-Sequence Token `</s>?`**

<details>
<summary>💡 Show Answer</summary>

The End-of-Sequence token (EOS token) is used to mark the end of generated content. If this token is missing from the fine-tuning data template, it will lead to the following impacts:

**1. The model cannot learn "when to stop generating"**

The model does not learn the signal "should output EOS" at that position, causing it to fail to stop generating naturally and correctly during inference; instead, it continues generating until reaching the maximum length limit, where it is forcibly truncated.

**2. Generated content "cannot stop" / redundant content appears**

After providing what should be a complete answer, the model continues generating additional, irrelevant, or even repetitive content, exhibiting a phenomenon of "continuing to ramble on."

**3. Confused turn boundaries in multi-turn dialogue scenarios**

The model may fail to properly distinguish turn boundaries during multi-turn dialogue inference, causing confusion in the conversation flow.

**Solution**: Inspect and fix the training data template to ensure that the EOS token is correctly appended to the end of the assistant response in every training sample, and that the EOS token at this position is included in the loss calculation.

</details>

**9. During Fine-Tuning, Should the Loss Function Be Calculated Only on the Output Portion, or on Both Input and Output Portions?**

<details>
<summary>💡 Show Answer</summary>

![Sequence Loss Masking (Label -100) Mechanism in Supervised Fine-Tuning](https://img.pricenode.uk/llm_interview_questions/english/chapter4/loss_masking_mechanism.png)




**Option 1: Calculate loss only on the output (assistant response) portion (loss masking)**

This is currently the mainstream standard practice for instruction fine-tuning.

Advantages:
- The training objective is aligned with actual usage scenarios;
- Prevents the model from learning the unnecessary tendency to "repeat the input";
- It is the default/recommended practice for the vast majority of open-source instruction fine-tuning datasets and training frameworks.

**Option 2: Calculate loss on both input and output portions**

That is, performing language modeling loss calculation over the entire sequence, similar to pure "continued pre-training."

Disadvantages:
- The training objective is inconsistent with inference tasks;
- If the input portion is significantly longer than the output portion, the vast majority of gradients in the loss stem from "predicting the input," thereby suppressing the proportion of contribution toward optimizing the truly desired "output generation capability."

**Summary**: Calculating loss only on the output portion (loss masking) is the correct practice for the vast majority of instruction fine-tuning scenarios.

</details>

**10. How to Improve the Model's Instruction Following Capability?**

<details>
<summary>💡 Show Answer</summary>

Common methods to improve a model's instruction following capability:

1. **High-Quality, Diverse Instruction Fine-Tuning Data**: Cover a wide range of task types, instruction phrasing styles, and complexity levels.
2. **Specialized Data for Strict Format/Constraint Instructions**: Construct a large amount of data featuring "instructions with explicit constraint conditions and responses fully satisfying the constraints" to strengthen model sensitivity to constraints.
3. **Complex Instruction Decomposition and Multi-Step Reasoning Training**: Training data should include multi-subtask combined instructions and corresponding responses that fully cover all sub-requirements.
4. **Preference Optimization Such as RLHF / DPO**: Utilize human preference annotations on model outputs to further optimize performance on the dimension of "whether instructions are precisely satisfied."
5. **Evaluation and Iteration**: Establish evaluation datasets specifically targeting "instruction compliance" to supplement corresponding training data for weak points, forming an iterative loop.

</details>

**11. Given an Enterprise Knowledge Base Composed of Several Articles, How Can It Be Converted into a Dataset Suitable for SFT to Make the Model Memorize It? How Can the Required SFT Dataset Size Be Determined?**

<details>
<summary>💡 Show Answer</summary>

**Methods for converting a knowledge base into an SFT dataset**:

**1. QA Pair Generation**
- For each article in the knowledge base, use an LLM to automatically generate multiple relevant questions and their corresponding standard answers, forming SFT samples in `(question, answer)` format;
- Questions should be diverse; the same knowledge point can be queried from different angles and phrasing styles to increase the model's "memory redundancy" for that knowledge point.

**2. Article Summarization and Paraphrasing**: Train the model to summarize, paraphrase, and restate article content in different styles.

**3. Cloze / Completion Tasks**: Construct training samples of "given a snippet of an article, complete/continue the remaining text."

**4. Multi-Turn Dialogue Format**: Package knowledge into multi-turn dialogue scenarios of "user inquiry -- model answer."

**Important Trade-off Considerations**: SFT (parametric memory) is not the ideal way to store and update knowledge base content. In practice, **RAG (Retrieval-Augmented Generation) is usually a more suitable solution than pure SFT for "enabling the model to answer questions using an enterprise knowledge base"** -- "SFT handles capability, while RAG handles knowledge."

**Factors in Determining SFT Dataset Size**: The size of the knowledge base and the number of knowledge points, the "exposure" frequency of each knowledge point, the scale of the base model, and the balance between overfitting and catastrophic forgetting.

</details>

**12. When a Fine-Tuned Model Deployed Online Exhibits Repeatedly Erroneous Cases, How Should the SFT Dataset Be Modified?**

<details>
<summary>💡 Show Answer</summary>

![Closed-Loop Online Error Remediation and Incremental SFT Dataset Iteration Workflow](https://img.pricenode.uk/llm_interview_questions/english/chapter4/sft_error_feedback_loop.png)




To address repeatedly erroneous cases identified post-deployment, the modification workflow usually includes:

1. **Collection and Categorization of Error Cases**: Systematically collect error cases reported from production, categorize and attribute errors, and identify common patterns in error modes.
2. **Targeted Supplementation of Training Samples**: Construct corresponding "correct demonstration" training samples, and create a set of generalized variation samples around the error pattern so that the model learns "the correct way to handle that class of problems" rather than "a specific answer to a specific input."
3. **Negative Examples / Contrastive Samples (Optional, for Preference Optimization Phase)**: Use "the model's current erroneous output" as a negative sample and "the corrected output" as a positive sample to construct preference pairs for DPO/RLHF.
4. **Avoid Introducing New Imbalance or Forgetting**: Newly added data should be mixed with the original dataset for incremental fine-tuning, and metrics on original benchmark sets should be monitored for stability.
5. **Construction of a Regression Test Suite**: Organize identified error cases into a regression test suite and validate on this test suite after every model iteration.

</details>

**13. When a Model Exhibits a "Parrot" (Repetition) Issue after Long Multi-Turn Dialogues (e.g., Repeating User Questions or Previous Responses), How Can It Be Solved via Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

Fine-tuning solutions for the "parrot" (repetition) issue:

1. **Inspect and Fix Multi-Turn Dialogue Samples in Training Data**: Audit whether training data itself contains samples such as "assistant replies containing paraphrases/repeats of user questions," and clean, filter, or rewrite such samples.
2. **Increase "Long Dialogue, Low Repetition" Positive Samples**: Specifically construct high-quality multi-turn dialogue samples that are long (multi-turn) where assistant responses remain concise and never repeat historical content.
3. **Construct Targeted Negative Examples / Preference Pairs (DPO)**: Collect "repetitive" outputs generated by the current model as negative examples, manually write concise, non-repetitive versions as positive examples, and directly train the model via DPO to "avoid repetition."
4. **Check Context Length and Positional Encoding Issues**: If repetition mainly occurs during long multi-turn dialogues, it may be related to positional encoding extrapolation issues; verify whether training data related to long text / long dialogues is sufficient.
5. **Reinforce System Prompts**: Add explicit instructions such as "Do not repeat the user's question" into the system prompt of training data, and ensure that assistant responses in the training data strictly comply with this instruction.

% ============================================================

</details>

