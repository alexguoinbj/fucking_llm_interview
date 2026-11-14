# 4.5 Domain Model Construction and Vertical Domain Fine-Tuning — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 4: Fine-Tuning · 6 questions

**1. What are the Data Selection Methods for Domain Model Continued Pre-Training?**

<details>
<summary>💡 Show Answer</summary>

In the process of domain model continued pre-training, data selection is crucial. Here are some common methods:
1. **Domain-Relevant Data Collection**: Collect data related to the target domain from the internet, domain documents, or internal company data.
2. **Domain Expert Annotation**: If domain experts are available, they can annotate the data to provide supervised data.
3. **Pseudo-Label Generation**: Use a pre-trained model to make predictions on domain data, and use the prediction results as pseudo-labels for training.
4. **Data Balancing**: Pay attention to data balance; for categories with fewer samples, data augmentation or oversampling techniques can be used.
5. **Data Quality Control**: Use quality evaluation metrics to filter and refine data, ensuring data accuracy and consistency.

</details>

**2. Is Vocabulary Expansion Necessary for Domain Models?**

<details>
<summary>💡 Show Answer</summary>

Whether vocabulary expansion is necessary for a domain model depends on the specific situation:
1. **Domain-Specific Vocabulary**: If the target domain contains specific vocabulary not covered by the general vocabulary, vocabulary expansion is necessary to help the model better understand and process these terms.
2. **Domain-Specific Context**: If the domain context differs significantly from the context of the general pre-trained model, vocabulary expansion can help the model better understand domain-specific contexts.
3. **Data Scarcity**: If the training data in the target domain is limited, vocabulary expansion can help the model better utilize pre-trained knowledge and improve performance.
It should be noted that vocabulary expansion may increase computational and storage costs. A comprehensive trade-off is required between the importance of domain-specific vocabulary, data scarcity, and computational resource constraints.

</details>

**3. How to Mitigate the Loss of General Capabilities After Training on Domain Data?**

<details>
<summary>💡 Show Answer</summary>

After training on domain data, the model may forget its general capabilities. Here are some mitigation methods:
1. **Retaining General Data**: Keep a portion of general data during domain data training to maintain the model's general language capabilities.
2. **Continual Learning**: Alternately use domain data and general data for training to help the model maintain memory of general knowledge while learning the new domain.
3. **Combining Pre-training and Fine-Tuning**: First perform pre-training on large-scale general data, and then fine-tune on domain data to balance general capabilities and domain performance.
4. **Data Resampling**: Increase the sampling frequency of general data during training to mitigate the issue of forgetting general capabilities.

</details>

**4. What are the Strategies for Vertical Domain Fine-Tuning?**

<details>
<summary>💡 Show Answer</summary>

![Multi-Stage Vertical Domain Fine-Tuning Roadmap (Pre-Training, SFT, RAG, and DPO Safety Guardrails)](https://img.pricenode.uk/llm_interview_questions/english/chapter4/vertical_domain_multistage_ft.png)




Common strategies for vertical domain (e.g., medical, legal, financial) fine-tuning include:

**1. Data Strategy**: Construct high-quality domain instruction datasets; supplement domain knowledge through continued pre-training; mix general data to prevent degradation of general conversational ability.

**2. Training Strategy**: Multi-stage training (domain continued pre-training $\rightarrow$ general instruction fine-tuning $\rightarrow$ domain instruction fine-tuning); use PEFT methods like LoRA for domain adaptation.

**3. Combining Knowledge with Retrieval (RAG)**: For vertical domains where knowledge updates frequently, fine-tuning is responsible for "how to say it", while RAG is responsible for "what to say (specific facts)".

**4. Safety and Compliance Constraints**: Vertical domains have higher safety and compliance requirements for output; fine-tuning data should include training samples that refuse to answer questions outside the scope of capability.

**5. Evaluation System and Human Expert Involvement**: Establish a specialized evaluation benchmark for the vertical domain, and involve domain experts in stages such as data annotation, quality auditing, and evaluation.

</details>

**5. For a Classification Task in a Critical Scenario (e.g., Misclassifying "Severe Adverse Reaction" as "Mild Adverse Reaction" is More Dangerous than the Reverse Error), How to Select Evaluation Metrics, Address Dataset Class Imbalance, and Modify the Loss Function?**

<details>
<summary>💡 Show Answer</summary>

![Cost-Sensitive Asymmetric Loss Function and Decision Threshold Adjustment for Critical Classification Scenarios](https://img.pricenode.uk/llm_interview_questions/english/chapter4/cost_sensitive_asymmetric_loss.png)




This is a classification problem involving **asymmetric error costs + class imbalance**, which requires a comprehensive approach across three levels: evaluation metrics, data processing, and loss functions:

**1. Selection of Evaluation Metrics**: Do not rely solely on overall accuracy. Focus primarily on the Recall of the minority class (severe adverse reactions), as missed detections correspond to the "more dangerous" error described. Simultaneously, use the F-beta score ($\beta > 1$, such as $F_2$) to weight the balance between Recall and Precision according to business requirements. Under class imbalance scenarios, PR (Precision-Recall) curves reflect the model's performance on the minority class better than ROC curves.

**2. Addressing Class Imbalance**: Oversample the minority class (e.g., SMOTE); undersample the majority class; use data augmentation to increase the number of minority class samples; apply stratified sampling to ensure consistent class proportions.

**3. Modifying the Loss Function**: Class-weighted loss, assigning higher weight to the "severe adverse reaction" class; cost-sensitive learning / asymmetric loss, imposing additional penalties for specific error directions (severe $\rightarrow$ mild); Focal Loss, forcing the model to focus more on hard-to-classify samples when classes are extremely imbalanced.

**4. Decision Threshold Adjustment**: Set a lower decision threshold for the "severe adverse reaction" class, preferring to accept more "false positives" (false alarms) in order to minimize "false negatives" (missed detections).

</details>

**6. Which Datasets are Recommended for Domain Large Language Model Pre-Training?**

<details>
<summary>💡 Show Answer</summary>

The selection of domain pre-training data should follow these principles:

- **Domain Relevance**: Prioritize data highly relevant to the target domain;
- **Data Quality**: Select high-quality, accurate domain data to avoid noise and erroneous information;
- **Data Diversity**: Ensure the data covers different sub-domains and task types within the target domain;
- **Data Scale**: A sufficiently large volume of data is fundamental to ensuring pre-training effectiveness;
- **Data Legality**: Ensure that the data used complies with relevant laws, regulations, and ethical standards;
- **General Data Mixing**: Mix in a certain proportion of general corpora to prevent performance degradation on general capabilities caused by over-focusing on domain data.

% ============================================================

</details>

