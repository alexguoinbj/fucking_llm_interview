# 1.1 Basic Overview of Large Models — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 1: Basic Knowledge of Large Models · 33 questions

**1. Briefly Introduce Large Models**

<details>
<summary>💡 Show Answer</summary>

Large Models: Generally refer to models with over 100 million parameters, but this standard is constantly evolving, and currently models with over a trillion parameters also exist. Large Language Models (LLM) are large models tailored for language.

</details>

**2. What Do Terms Like 5B, 7B, 31B Following Large Models Refer To?**

<details>
<summary>💡 Show Answer</summary>

5B, 7B, 31B, etc.: These generally refer to the number of parameters, where B stands for Billion. For example, 15B means 15 billion parameters.

</details>

**3. What Are the Advantages of Large Models?**

<details>
<summary>💡 Show Answer</summary>

1. They can utilize large amounts of unlabeled data to train a general-purpose model, and then fine-tune it with a small amount of labeled data to adapt to specific tasks. This pre-training and fine-tuning approach reduces data annotation costs and time while improving model generalization capability.
2. They can leverage generative artificial intelligence technology to produce novel and valuable content, such as images, text, music, etc. This generative capability helps users achieve better experiences and outcomes in fields like creativity, entertainment, and education.
3. They can utilize emergent capabilities to accomplish tasks that were previously impossible or difficult to perform, such as math word problems, commonsense reasoning, and symbolic manipulation. These emergent capabilities reflect the model's level of intelligence and reasoning ability.





![Key Parameters and Architecture Pillars of Large Language Models](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig01_llm_key_parameters.png)

</details>

**4. What Are the Disadvantages of Large Models?**

<details>
<summary>💡 Show Answer</summary>

1. They require vast computational and storage resources to train and run, increasing economic and environmental burdens. It is estimated that training a GPT-3 model costs approximately $300,000 and produces about 284 tons of carbon dioxide emissions.
2. They face data quality and security issues, such as data bias, data leakage, and data misuse. These problems may lead to inaccurate or unethical outputs, affecting the interests of users and society.
3. They need to address challenges regarding interpretability, reliability, and sustainability, such as how to understand and control model behavior, ensure accuracy and stability, and balance benefits and risks. These challenges require multidisciplinary research and collaboration to ensure the healthy development of large models.

</details>

**5. What Are the Key Parameters of LLMs?**

<details>
<summary>💡 Show Answer</summary>

LLMs are complex AI models whose performance is influenced by several key parameters. Understanding these parameters is crucial for effectively developing, training, and utilizing LLMs.

Here is a breakdown of some of the most important parameters:

- **LLM Architecture**: This refers to the underlying design and structure of the neural network used for LLMs. It determines the efficiency of the model in learning and processing information, as well as its stability and versatility when handling different tasks. Common LLM architectures include the high-performing Transformer, which excels at processing long text sequences.

- **Model Size (Parameters)**: This refers to the total number of numerical values (weights and biases) in the LLM's neural network. Simply put, it reflects the complexity of the model. Generally, larger models with more parameters possess stronger capabilities in handling complex tasks and can produce more nuanced outputs. However, they also require more computational resources for training and inference.

- **Quality and Quantity of Training Data**: The training data used by an LLM significantly impacts the quality and relevance of its outputs. High-quality, diverse, and relevant data leads to more accurate and reliable models. The quantity of training data is also essential; typically, larger datasets lead to better model performance, but require more training time and resources.

- **Hyperparameters**: These are settings that control the learning process of the LLM. Unlike model parameters, they do not directly encode knowledge, but guide how the model learns from training data. Examples of hyperparameters include learning rate, batch size, and optimizer type. Tuning these settings can fine-tune the learning process and optimize model performance for specific tasks.

</details>

**6. How Do LLMs Work?**

<details>
<summary>💡 Show Answer</summary>

LLMs use word embeddings to process text, which is a multidimensional word representation method capable of capturing the meaning of words and their relationships with other words. This enables the Transformer model (a deep learning technology) to understand context and relationships within sentences via an encoder. Leveraging this knowledge, the decoder can generate human-like text based on prompts or context.





![Paradigm Shift from Traditional NLP to Large Language Models](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig02_nlp_vs_llm_paradigm.png)

</details>

**7. What Are Generative Models and Discriminative Models?**

<details>
<summary>💡 Show Answer</summary>

- **Generative Models**: These models learn the underlying probability distribution of the data to generate new, similar data samples. They model the input features $\mathbf{x}$ and their corresponding labels $y$, capable of answering questions such as “Given $\mathbf{x}$, what is the probability of $y$?” and “For $y$, what are the possible $\mathbf{x}$ values?”.
    - **Applications**: Text generation (GPT), image synthesis (GANs), speech generation.
    - **Examples**: GPT generates human-like text by predicting the next word.
- **Discriminative Models**: These models focus on learning decision boundaries between different classes. They directly model $P(y|\mathbf{x})$, which is the probability of label $y$ given input $\mathbf{x}$, without learning the data distribution.
    - **Applications**: Classification tasks (e.g., spam detection, sentiment analysis).
    - **Examples**: BERT classifies text into categories such as positive or negative sentiment.

</details>

**8. What Do You Think Is the Biggest Difference Between NLP and LLMs? What Are Their Similarities and Differences?**

<details>
<summary>💡 Show Answer</summary>

The relationship between NLP (Natural Language Processing) and LLMs (Large Language Models) is one of field vs. technology, general vs. specific. LLMs represent the most cutting-edge and impactful technological paradigm in the evolution of NLP to date, largely reshaping the field of NLP.

Similarities:

- Consistent ultimate goal: The fundamental goal of both is to achieve AI understanding, generation, and application of human language---the so-called “jewel in the crown of artificial intelligence”.
- Shared technical foundation: Both modern NLP and LLMs are built on deep learning, particularly neural networks. The Transformer architecture serves as a key bridge connecting the two; from BERT to GPT, all are extensions and developments of its core concepts.

Biggest difference and key distinctions: The most significant difference lies in a fundamental shift in research and application paradigms---moving from “training a model for each task” to “solving all tasks with one model”.

Specifically, this can be viewed from the following dimensions:

1. Task-Handling Paradigm:

- Traditional NLP: Adopts a “divide and conquer” strategy. Researchers design specific model architectures, loss functions, and training datasets for each individual NLP task (e.g., machine translation, sentiment analysis, named entity recognition), following a `Pre-train -> Fine-tune` workflow. Each model acts as a specialized “expert”.
- LLMs: Pursue a “unified” general-purpose model. By pre-training on massive amounts of data at scale, a foundational LLM gains the potential to solve a wide variety of tasks. Users guide the model by designing prompts (Prompt) or providing in-context examples (In-context Learning), drastically simplifying development workflows and even achieving zero-shot (Zero-shot) and few-shot (Few-shot) learning.

1. Model Capabilities & Emergence:

- Traditional NLP: The model's capabilities are clear and limited, usually directly related to its training objectives.

- LLMs: When model scale (parameters, data, compute) crosses a certain threshold, it exhibits “emergent abilities” (Emergent Abilities) that do not exist in small models. For example, complex logical reasoning (Chain-of-Thought, CoT), code generation, and following complex instructions. These capabilities are not directly trained, but spontaneously learned from massive amounts of data.

1. Scale:

- Traditional NLP: Model parameter counts are typically in the millions to hundreds of millions (e.g., BERT-base is about 110M).
- LLMs: Parameter counts start from billions (Billion) and scale to hundreds of billions or even trillions. Training data and required computational resources are also orders of magnitude higher than traditional NLP models.

1. Interaction & Application:

- Traditional NLP: Typically integrated into software as APIs, with relatively fixed input and output formats.
- LLMs: Have birthed a brand-new interaction paradigm centered around dialogue and instructions (such as ChatGPT), making AI much more accessible. Applications have also evolved from backend tools into user-facing products.

Summary: If traditional NLP is like building a toolbox composed of various “specialized tool experts”, LLMs are striving to create a “Swiss Army knife” style general intelligent tool. While it may not be as refined as specialized tools on certain specific tasks, its versatility, flexibility, and powerful emergent capabilities are unprecedented.

</details>

**9. Please Explain Emergent Capabilities**

<details>
<summary>💡 Show Answer</summary>

Understanding “Emergent Capabilities”: “Emergent Capabilities” (Emergent Abilities) refer to abilities that do not exist or perform poorly in smaller models, but suddenly appear and significantly surpass random levels when the model scale (including parameter count, training data, and compute) reaches a critical threshold.

Its core characteristics are non-linearity and unpredictability:

- Non-linear growth: The performance of such capabilities does not increase smoothly and linearly with model scale. Instead, a “phase transition”-like jump occurs within a certain scale range, where performance rapidly rises from near random guessing to a very high level.
- Non-direct training: These advanced capabilities are usually not directly trained via specific supervised learning objectives. For example, we did not directly teach the model how to “think step by step”, but when the model becomes large enough, it spontaneously acquires this capability by learning logical relationships in massive text.

Typical examples of emergent capabilities include:

1. Chain-of-Thought (CoT): When facing mathematical or logical problems requiring multi-step reasoning, by prompting the model to “think step by step”, a large model can generate a coherent reasoning process and arrive at the correct answer. Small models cannot utilize such prompts.
2. In-context Learning: Without updating model weights, by simply providing a few task examples (Few-shot) in the prompt, a large model can “learn” and execute this new task.
3. Executing complex instructions: Understanding and following complex human instructions containing multiple steps, constraints, and negative logic.

Model scale for emergence: There is no fixed numerical threshold for the specific scale at which emergent capabilities appear; it depends on the capability itself, model architecture, data quality, and the complexity of the evaluation task.

However, according to landmark studies by Google and other institutions, many notable emergent capabilities, such as Chain-of-Thought reasoning, typically begin to emerge when the model parameter scale reaches tens of billions to a hundred billion.

- For example, in experiments with Google's PaLM model, Chain-of-Thought reasoning capability began to manifest on the 62B parameter model, while remaining completely ineffective on the 8B and 16B models. This capability grew stronger and more stable as the model scaled to 540B.

In summary, “emergent capabilities” are a vivid manifestation of “quantitative change leading to qualitative change” in the domain of large models. It demonstrates that simply scaling up can unlock entirely new, higher-level cognitive abilities, which is one of the core drivers continuously pushing model scale growth in current LLM research.





![Scaling Curve and Phase Transition of Emergent Capabilities](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig03_emergent_abilities_scaling.png)

</details>

**10. What Causes Emergent Capabilities?**

<details>
<summary>💡 Show Answer</summary>

1. The evaluation metrics for tasks are not smooth enough;
2. Complex tasks vs. subtasks: For instance, assume a task $T$ consists of 5 subtasks $Sub-T$. As the model grows, the metric for each $Sub-T$ increases from 40% to 60%, but the overall metric for the final task only increases from 1.1% to 7%. In other words, an emergent phenomenon is observed macroscopically, but the performance on subtasks actually grows smoothly.

</details>

**11. What Is Hallucination, and How Can It Be Controlled via Prompt Engineering?**

<details>
<summary>💡 Show Answer</summary>

**Hallucination** refers to instances where large language models generate content that appears plausible but is actually inaccurate, untrue, or entirely fabricated. This is one of the core challenges currently facing large models.

**Causes**:
1. **Data level**: Training data itself contains a large amount of false, outdated, or contradictory content. During the pre-training phase, the model memorizes and fits to this noisy data, leading it to potentially reproduce or derive erroneous information during generation. In addition, uneven coverage of training data can cause the model to be “overconfident” in certain domain knowledge.

2. **Model level**: Large language models are inherently probabilistic generative systems whose outputs result from sampling conditional probability distributions over the next token. Models do not possess true “understanding” or “fact-checking” capabilities; instead, they tend to generate text that is statistically “plausible-sounding”, which fundamentally makes hallucination difficult to eliminate completely.

3. **Decoding strategy level**: Decoding parameters (such as temperature, top-p, top-k) significantly impact the occurrence of hallucinations. Higher temperature values introduce greater randomness, increasing the probability of the model deviating from facts; conversely, overly greedy decoding may amplify model bias. Selecting appropriate decoding strategies can suppress hallucinations to some extent.

4. **Knowledge boundary level**: Model knowledge has a cutoff date (knowledge cutoff), making it unaware of new events and knowledge occurring after the training data. When users ask about content outside the model's knowledge boundary, the model often does not refuse to answer, but tends to “fabricate” a plausible-sounding answer, making this a high-frequency scenario for hallucinations.

**Solutions**:
1. **Data level**: Introduce higher-quality, manually reviewed datasets during the pre-training and fine-tuning stages to reduce the proportion of noise and misinformation. Guide the model toward producing more accurate and reliable content through data deduplication, fact-checking pipelines, and Reinforcement Learning from Human Feedback (RLHF). Additionally, “I don't know” samples can be included during fine-tuning to explicitly train the model to acknowledge uncertainty at its knowledge boundaries.

2. **RAG (Retrieval-Augmented Generation) level**: Combine external knowledge bases with model inference. Before generating an answer, retrieve context snippets relevant to the question from trusted knowledge sources (such as enterprise documents, real-time databases, internet), and feed them into the model as part of the prompt. RAG effectively mitigates hallucinations caused by outdated knowledge and insufficient coverage, making it one of the most mainstream hallucination mitigation solutions in the industry today.

3. **Prompt engineering level**: Constrain model behavior through carefully crafted prompts. Key strategies include:
    - **Explicitly instruct uncertainty**: Require the model in the prompt to explicitly state when it is uncertain, e.g., “If you are uncertain, please directly answer 'I don't know'\,”.
    - **Guide step-by-step reasoning (Chain-of-Thought)**: Require the model to list reasoning steps before providing conclusions, which helps expose and reduce logical errors.
    - **Provide reference context**: Attach relevant background materials in the prompt to let the model answer based on given facts rather than relying on parametric memory.
    - **Output constraints and formatting**: Limit the model's output format (such as JSON or structured lists) to reduce room for “free improvisation”.
    - **Self-verification prompts**: Require the model to verify the logical consistency of its output after generating an answer, e.g., “Please re-check whether your answer contradicts the above materials.”

</details>

**12. How to Control LLM Hallucinations at Different Levels?**

<details>
<summary>💡 Show Answer</summary>

LLM hallucination refers to instances where the model generates content that is contrary to facts, fabricated out of thin air, or self-contradictory. It can be controlled across the following levels:

**Data Level:**
- **High-quality training data**: Use rigorously cleaned, deduplicated, and fact-checked datasets during pre-training and fine-tuning to reduce the introduction of noise and misinformation.
- **Knowledge cutoff labeling**: Label timestamps on training data to help the model distinguish between “known” and “unknown”, avoiding overconfidence in expired information.
- **Negative sample training**: Introduce negative samples containing hallucinated content, enabling the model to recognize and refuse to generate such content through contrastive learning.

**Model Training Level:**
- **RLHF (Reinforcement Learning from Human Feedback)**: Have human annotators penalize hallucinated outputs, using algorithms like PPO to guide the model toward factual answers.
- **Direct Preference Optimization (DPO)**: Compare “hallucinated answers” with “factual answers” in a more stable manner to optimize model preferences.
- **Uncertainty modeling**: Train models to output confidence estimates, explicitly expressing “I don't know” when uncertain about a question, rather than forcing an answer.
- **Self-Consistency training**: Generate multiple reasoning chains for the same question, selecting the answer with the highest consistency to filter out sporadic hallucinations.

**Inference and Decoding Level:**
- **Temperature adjustment**: Lower sampling temperature so model output concentrates on high-probability tokens, reducing random hallucinations.
- **Beam Search**: Compared to greedy decoding, beam search optimizes at the global sequence level, helping generate more coherent and factual outputs.
- **Chain-of-Thought Prompting (CoT)**: Guide the model through step-by-step reasoning, exposing intermediate steps for error checking and mitigating leap-of-logic hallucinations.
- **Retrieval-Augmented Generation (RAG)**: Retrieve external knowledge bases before generation and inject retrieved documents into the prompt as context, anchoring generation in real sources.

**System and Engineering Level:**
- **Tool Use**: Empower the model to call search engines, calculators, database queries, and other tools, outsourcing tasks requiring precision to reliable external systems.
- **Fact-checking module**: Integrate an independent fact verifier (e.g., NLI-based verification model) post-output to filter statements contradicting reference documents.
- **Output post-processing and filtering**: Use rule engines or classifiers to identify high-risk hallucination patterns (e.g., nonexistent citations, forged data) and trigger retries or warnings.
- **Human-in-the-loop audit**: Introduce human review steps in high-risk scenarios (medical, legal, financial) as a final line of defense.

**Prompt Level:**
- **Explicit constraint instructions**: Require the model in the System Prompt to “explicitly state if uncertain” and “answer based only on the provided context”.
- **Provide reference documents**: Place authoritative sources directly in context, requiring the model to answer accordingly rather than relying on parametric memory.
- **Step-by-step verification prompts**: Require the model to list evidence before drawing conclusions to form a self-review feedback loop.





![Multi-Level Defense Architecture for Mitigating LLM Hallucinations](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig04_hallucination_multilevel_control.png)

</details>

**13. How to Improve the Accuracy and Reliability of LLMs and Make Their Answers Verifiable?**

<details>
<summary>💡 Show Answer</summary>

Improving the accuracy and reliability of LLMs and granting verifiability to their answers requires collaborative efforts across multiple dimensions, including generation mechanisms, knowledge sources, output formats, and evaluation systems.

**Reliability of Knowledge Sources:**
- **Retrieval-Augmented Generation (RAG)**: Combine the model with real-time or domain-specific knowledge bases, citing specific document passages during generation so that every statement has a traceable basis.
- **Knowledge Graph Integration**: Incorporate structured knowledge graphs (e.g., Wikidata, enterprise knowledge bases) into the reasoning process to ensure accuracy of entity relationships.
- **Regular Knowledge Updates**: Perform continuous fine-tuning or knowledge injection on proprietary models to avoid errors caused by outdated knowledge.

**Traceability and Citability of Outputs:**
- **Source Citation**: Force the model to annotate information sources (document name, page number, URL) in answers, enabling users to independently verify.
- **Citation Consistency Verification**: Introduce post-processing steps to automatically check whether sources cited by the model actually exist and whether cited content aligns with the original text.
- **Structured Output (JSON/XML)**: Output answers in structured formats with fields such as “claim”, “evidence”, and “confidence”, facilitating automated verification by downstream systems.

**Transparency of Reasoning Processes:**
- **Chain-of-Thought (CoT) and Tree-of-Thoughts (ToT)**: Require the model to explicitly output reasoning steps, enabling humans or automated tools to review logical chains step by step.
- **Self-Reflection**: Prompt the model after generating an answer to critically review its own output, identifying and correcting potential errors.
- **Multi-Path Reasoning and Majority Voting**: Generate multiple independent reasoning paths in parallel for the same question, selecting the majority-consistent answer to filter out accidental errors.

**External Verification and Tool Enhancement:**
- **Tool Use**: Invoke calculators for numerical computations, search engines for real-time information, and interpreters for code execution verification, delegating precision tasks to deterministic systems.
- **Independent Verifier**: Deploy specialized verification models (Critic Models) or NLI-based checkers to perform secondary assessment on the main model's output.
- **Code Generation and Execution Verification**: For programmatically verifiable problems (math, logic, data analysis), have the model generate code and execute it to cross-validate textual answers against execution results.

**Calibration and Uncertainty Quantification:**
- **Confidence Score Output**: Train or prompt the model to output confidence scores for each statement, keeping users alert to highly uncertain answers.
- **Calibration Evaluation**: Periodically measure how model confidence aligns with actual accuracy on benchmark datasets, fine-tuning overconfident models for calibration.
- **Refusal Mechanism**: When the model determines its own knowledge is insufficient to support an accurate answer, explicitly output “Cannot determine” rather than generating low-quality guesses.

**Continuous Establishment of Evaluation Systems:**
- **Factual Benchmarks**: Continuously evaluate model factual accuracy using specialized benchmarks such as TruthfulQA, FActScore, and FEVER to track inter-version changes.
- **Domain Expert Annotation**: Engage domain experts in specialized fields like medical, legal, and financial sectors to score outputs, building high-quality evaluation sets.
- **Red Teaming**: Actively probe model hallucination boundaries via adversarial prompts to identify systemic weaknesses and make targeted improvements.
- **User Feedback Loop**: Collect user feedback on answer accuracy in production environments as data sources for continuous fine-tuning and monitoring.

</details>

**14. What are the Causes and Solutions for Large Language Models Becoming "Repeaters"?**

<details>
<summary>💡 Show Answer</summary>

**Phenomenon Description**: From the perspective of data, during attention, this part of the information is skipped while predicting after the previous context to obtain the prediction. However, the prediction contributes very little to attention, causing the model to still focus its attention on the previous content when predicting the next token after the prediction, which leads to cyclical/repetitive predictions.

**Causes**:
1. **Data Bias**: Large language models are typically trained on massive unlabeled data during the pre-training phase. If the training data contains a large amount of repetitive text or certain specific sentences or phrases appear with high frequency, the model may tend to copy these conventional patterns when generating text.
2. **Limitations of Training Objectives**: The training of large language models is usually based on self-supervised learning methods, learning language modeling by predicting the next word or masking words/phrases. Such training objectives may make the model more inclined to generate text similar to the input, leading to the repetition problem.
3. **Lack of Diverse Training Data**: Although large language models can process large-scale data, if the training data lacks diversity in linguistic expressions and contexts, the model may fail to learn sufficient diversity and creativity, leading to the repetition problem.
4. **Model Architecture and Parameter Settings**: The structure and parameters of large language models can also affect the repetition problem. For example, the model's attention mechanism and generation strategy may cause the model to be more inclined to copy the input text.

**Solutions**:

Currently, there are two main strategies for addressing the repetitive generation problem in LLMs: training-based strategies and decoding-based strategies.

1. **Training Strategy**: The overall idea is to construct synthetic data (i.e., phrase repetitions, sentence repetitions, etc., such as repeating a phrase or sentence $N$ times) and design a repetition penalty term to suppress the LLM from generating repetitive sentences. Relevant losses are introduced, such as the DITTO method. The repetition penalty is achieved by designing a loss function, where $\lambda$ is the penalty factor, which may take different values for different tasks.

2. **Decoding-based Strategy**: Decoding-based strategies encompass various methods, such as beam search, random search (top-$K$, top-$P$), temperature, n-gram, etc.
    1. **Beam Search**: An improvement over the greedy strategy, expanding the search scope slightly. At each time step, instead of keeping only the single output with the highest current score (greedy strategy), it maintains `num_beams` candidates. When `num_beams`=1, beam search degrades to greedy search.
    2. **Random Search (top-$K$, top-$P$)**: Top-$K$ filters from the $K$ tokens with the highest probabilities, allowing other high-scoring tokens a chance to be selected. Top-$P$ candidate lists tokens whose cumulative probability does not exceed a specified threshold; top-$P$ is usually set to a higher value to limit long-tail low-probability tokens from being sampled.
    3. **Temperature $T$**: Lower temperature means less randomness; a temperature of 0 always yields the exact same output, while higher temperature means higher randomness, helping the model produce more creative outputs.

\begin{quote}
**References**
1. <https://zhuanlan.zhihu.com/p/672261242>
2. <https://zhuanlan.zhihu.com/p/666918508>
3. Learning to Break the Loop: Analyzing and Mitigating Repetitions for Neural Text Generation
\end{quote}





![Training and Decoding Strategies for Mitigating Repetitive Text Loops](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig05_repeater_mitigation_strategies.png)

</details>

**15. How to Evaluate the Performance of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Evaluating LLM performance requires a comprehensive consideration of cost, user experience, and responsible AI practices. Traditional benchmark datasets have limitations for LLMs, necessitating evaluation through real-world user traffic. This helps measure human-like capabilities and ensures a safe and valuable user experience.

Key metrics include:
- GPU utilization (cost estimation).
- Responsible AI (detecting and mitigating risks).
- Performance (latency).
- Utility (user value).

A/B testing is also crucial for measuring the impact of LLM features. This includes pre-launch experiments (dark launches, controlled rollouts) and post-launch experiments (shadow experiments, standard A/B tests) to optimize these features.

</details>

**16. What Are Scaling Laws? What Relationship Do They Reveal Among Model Performance, Compute, and Data Volume? What Guidance Do They Offer for LLM R&D?**

<details>
<summary>💡 Show Answer</summary>

What are Scaling Laws? Scaling Laws are a set of empirical patterns discovered by institutions such as OpenAI and DeepMind through extensive experiments. They reveal that the performance of large language models (typically measured by cross-entropy Loss) exhibits a predictable power-law relationship with three key resource factors: model parameter scale ($N$), training dataset size ($D$), and compute used for training ($C$).

What relationship do they reveal?

1. Predictability of Performance: Scaling Laws show that the model's loss decreases smoothly and predictably as $N$, $D$, and $C$ increase. This relationship can be described by a power-law formula. For instance, when data and compute are sufficient, the relationship between model loss $L$ and parameter count $N$ is roughly: $L(N) \propto N^{-\alpha}$, where $\alpha$ is a small positive exponent. This means we can extrapolate (predict) the performance that a larger model might achieve based on experimental results from smaller models.
2. Bottleneck Effect: The final performance of a model is constrained by the most limited factor among $N$, $D$, and $C$. If one only increases model size without increasing data volume, performance improvements will quickly hit a bottleneck; and vice versa. To effectively improve model performance, all three factors must be scaled synergistically.
3. Optimal Resource Allocation: For a given compute budget (FLOPs), there exists an optimal combination of model size ($N$) and data volume ($D$). DeepMind's Chinchilla paper was a landmark discovery that corrected the earlier view prioritizing model scaling, demonstrating that to achieve compute-optimality, model parameters and training token count should scale at an approximate 1:20 ratio. For example, training a 70B parameter model requires approximately 1.4 trillion tokens of data.

Guidance for LLM R&D:

1. Scientific Guidance for Project Planning: Before investing millions or tens of millions of dollars in a large-scale training run, research institutions can first fit the Scaling Law for their dataset and model architecture through small-scale experiments. This allows them to scientifically predict the final model performance, evaluate project ROI, and reasonably request compute resources.
2. Optimizing Resource Allocation and Avoiding Waste: Scaling Laws, especially Chinchilla scaling, provide clear guidance on how to efficiently spend compute budgets. They show that rather than training an over-sized model with insufficient data (over-trained/undertrained relative to size), it is better to use the same compute to train a slightly smaller model on more tokens, which often yields better results. This has driven the industry from purely chasing "large parameters" to balancing "large parameters with large data."
3. Emphasizing the Importance of Data: The discovery of Scaling Laws made academia and industry realize that high-quality, large-scale training data is just as important as model parameter scale, and even more critical at certain stages. This has driven advancements in data engineering, data cleaning, and high-quality synthetic data generation.





![Scaling Laws and Chinchilla Compute-Optimal Frontier](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig06_scaling_laws_chinchilla.png)

</details>

**17. What Are the Differences Between Open-Source Base, Chat, and Instruction Models from Various Vendors?**

<details>
<summary>💡 Show Answer</summary>

Base, Chat, and Instruction are three different model variants, obtained through different training or fine-tuning approaches. Currently, LLM training is divided into three stages: pre-training, supervised fine-tuning (SFT), and reinforcement learning from human feedback (RLHF). Through these three stages, Base models and Chat models are produced respectively.

1. **Base Model (Foundation Model)**: The Base model is obtained through the first stage, pre-training. It is pre-trained on massive unlabeled data to learn broad language representations. Base models feature large parameter scales and excel in diverse tasks such as text generation, semantic understanding, and language translation. This model offers high generality without specific task preferences, but requires fine-tuning for specific tasks.

2. **Chat Model**: The Chat model is built on top of the Base model through the second stage (SFT) and the third stage (RLHF). After fine-tuning and reinforcement on general tasks, the Chat model acquires conversational capabilities, reasoning abilities, user preference alignment, and other natural language understanding (NLU) skills. Incorporating human preferences, it engages in fluent dialogue with appropriate emotional and politeness control, making it widely used in conversational and interactive scenarios.

3. **Instruction Model**: An Instruction model is fine-tuned on instruction datasets, enabling it to understand and execute complex natural language instructions. Many domain-specific models are built on pre-trained models by applying targeted instruction tuning to better adapt to target tasks and align with user preferences. During instruction tuning, instructions and corresponding responses are formatted into text pairs to fine-tune the model. This model primarily targets instruction-following in specific scenarios, performing clear tasks with structured outputs.

</details>

**18. What Are the Mainstream Open-Source Model Families Currently?**

<details>
<summary>💡 Show Answer</summary>

Currently, mainstream open-source model architectures fall into three main categories:

**Category 1: Prefix Decoder Family**
- Description: Bidirectional attention on input prefix, unidirectional attention on output generation.
- Representative Models: ChatGLM, ChatGLM2, U-PaLM

**Category 2: Causal Decoder Family**
- Description: Left-to-right unidirectional attention.
- Representative Models: LLaMA-7B, LLaMA derivatives

**Category 3: Encoder-Decoder Family**
- Description: Bidirectional attention on input, unidirectional attention on output generation.
- Representative Models: T5, Flan-T5, BART





![Multi-Stage Training Lifecycle: Base, Instruction, and Chat Models](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig07_base_chat_instruction_models.png)

</details>

**19. How Does GPT-4 Differ from GPT-3?**

<details>
<summary>💡 Show Answer</summary>

- **Multimodal Input**: GPT-4 can process both text and images, making it versatile in tasks such as visual question answering.
- **Larger Context Window**: GPT-4 can handle longer input sequences than GPT-3, improving coherence in long discussions or documents.
- **Accuracy**: Language understanding, logical reasoning, and factual accuracy are enhanced due to improved training and increased parameters.
- **Multilingual Capabilities**: Improved handling of a broader range of languages makes GPT-4 more accessible globally.

</details>

**20. What Are the Differences Between LLaMA 1, LLaMA 2, and LLaMA 3?**

<details>
<summary>💡 Show Answer</summary>

- **Parameter Sizes**: The first and second generations of LLaMA models both included four parameter scale versions: the smallest model at 7B, followed by 13B, 34B, and 70B (the largest in LLaMA 1 was 65B). For LLaMA 3, only the 8B and 70B versions were initially released. According to announcements, the largest version is a 400B parameter model, which was still in training.

- **Model Architecture**: LLaMA 3 fully adopts GQA to accelerate training and inference.

- **Context Length**: The context lengths of the three LLaMA generations are 2K, 4K, and 8K tokens, respectively. Although LLaMA 3 was trained on 8K context, current industry techniques allow further context extension.

- **Vocabulary Size**: In LLaMA 1 and LLaMA 2, Meta AI used a vocabulary size of 32K. The third-generation LLaMA 3 model expanded its vocabulary size to 128K, representing a major tokenizer update. A larger vocabulary enables stronger semantic representation and provides a foundation for multi-lingual support.

- **Training Data**: The training data for LLaMA 3 increased dramatically to 15 trillion tokens—several times more than LLaMA 1 and LLaMA 2 combined. The smaller LLaMA 1 models were trained on 1 trillion tokens, while the 65B model used 1.4 trillion tokens. For LLaMA 2, the training dataset grew to 2 trillion tokens across all sizes.

- **Training Duration**: When the LLaMA 1 paper was published, training LLaMA 1 cost millions of dollars, as the 65B model required 1.02 million GPU hours. With LLaMA 3, training compute grew dramatically; the training duration of LLaMA 3-8B alone exceeded that of the 65B LLaMA 1 model.

- **Evaluation Metrics**: Benchmark results on MMLU (general understanding), GSM8K (math reasoning), and HumanEval (coding ability) show that the performance jump from LLaMA 2 to LLaMA 3 is significantly larger than the jump from LLaMA 1 to LLaMA 2.

</details>

**21. Architectural Features of ChatGLM**

<details>
<summary>💡 Show Answer</summary>

- Early ChatGLM adopted the **Prefix LM** attention pattern—using bidirectional attention for the input prefix portion and unidirectional attention for the text to be generated.
- **Autoregressive Blank Infilling** Pre-training Objective: Randomly masking multiple continuous text spans in the sequence, requiring the model to fill in these blanks autoregressively.
- **2D Positional Encoding**: Separately encodes the token's position in the original sequence and its relative position within the filled span.
- Subsequent versions (ChatGLM2/3) gradually converged toward the standard Decoder-only + Causal LM architecture, introducing MQA, FlashAttention, and support for longer context windows.

</details>

**22. What Key Components Are Included in Current Mainstream Open-Source Large Model Architectures?**

<details>
<summary>💡 Show Answer</summary>

The LLaMA series adopts a **Decoder-only Transformer** architecture with the following main features:

- **Pre-normalization + RMSNorm**: Normalization is applied at the input of each sub-layer using RMSNorm instead of LayerNorm, eliminating mean computation for higher computational efficiency and training stability.
- **SwiGLU Activation Function**: The FFN layer replaces standard ReLU/GELU with SwiGLU, boosting model expressiveness for the same parameter count.
- **Rotary Position Embedding (RoPE)**: Incorporates positional information onto $Q$ and $K$ vectors via rotation matrices, offering superior relative position modeling and length extrapolation capabilities.
- **Causal Self-Attention**: Each token can only attend to itself and preceding tokens.
- **Bias-Free Design**: Most linear layers omit bias terms to simplify the model and improve training stability.
- **Tokenizer**: Uses Byte-Pair Encoding (BPE, based on SentencePiece).

Most subsequent open-source models (such as Qwen, Mistral, Baichuan, etc.) follow this fundamental paradigm, with variations mainly in specific attention variants (e.g., GQA/MQA), activation details, or normalization placement.





![Modern Decoder-Only LLM Architecture (RMSNorm, SwiGLU, RoPE)](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig08_llama_architecture_components.png)

</details>

**23. Which Open-Source Frameworks Are You Familiar With?**

<details>
<summary>💡 Show Answer</summary>

Open-Source Frameworks:

- Foundational Frameworks: PyTorch is the de facto standard for current LLM research and development, offering flexible tensor computation and automatic differentiation capabilities.
- Models and Ecosystem: Hugging Face Transformers is the most crucial model library and ecosystem, dramatically lowering the barrier to using and sharing models.
- Large-Scale Training: DeepSpeed (Microsoft) and Megatron-LM (NVIDIA) are core frameworks for large-scale distributed training, implementing key technologies such as 3D parallelism and ZeRO.
- Efficient Inference: Frameworks such as vLLM and TensorRT-LLM focus on optimizing LLM inference speed and throughput, addressing KV Cache VRAM bottlenecks through techniques like PagedAttention.

</details>

**24. What Are the Differences Between LLMs and Traditional Chatbots?**

<details>
<summary>💡 Show Answer</summary>

\begin{center}


| **Comparison Dimension** | **Traditional Chatbot** | **LLM** |
| --- | --- | --- |
| Knowledge Source | Manually written rules and templates | Trained on massive text data |
| Understanding | Keyword matching, limited intent recognition | Deep semantic understanding, context modeling |
| Coverage | Closed domain (preset scenarios) | Open domain (any topic) |
| Multi-turn Dialogue | Relies on explicit state machines | Implicit context modeling, natural flow |
| Output Method | Fixed template filling | Free-form text generation |
| Maintenance Cost | Requires frequent manual rule updates | Adaptable via fine-tuning or prompts |


\end{center}

Traditional chatbots rely on preset rules and fixed templates for responses, handling only limited scenarios and failing to address questions outside their defined scope (i.e., “out-of-scope” issues); once user intent deviates from predefined paths, the system cannot provide reasonable responses.

Through pre-training on massive text corpora, LLMs internalize rich linguistic and world knowledge. They can comprehend complex semantics, address open-domain questions, engage in multi-turn conversations, and produce more natural and fluent outputs unfettered by canned answers. After Instruction Tuning and RLHF alignment, LLMs align even better with user intent, significantly raising interaction quality.

</details>

**25. What Are the Typical Application Scenarios for LLMs?**

<details>
<summary>💡 Show Answer</summary>

- **Text Generation**: Article writing, code generation, summarization. LLMs can generate high-quality long text from brief prompts, condense long documents into summaries, or automatically generate code from natural language descriptions (e.g., GitHub Copilot).

- **Conversational Interaction**: Intelligent customer service, virtual assistants. Compared to traditional rule-based chatbots, LLMs handle open-domain questions and maintain multi-turn context, significantly enhancing user experience.

- **Information Extraction**: Named Entity Recognition (NER), relation extraction, event extraction. LLMs can extract structured information via prompts or few-shot examples, greatly reducing annotation costs.

- **Language Translation**: Multilingual translation. LLMs significantly improve translation quality for low-resource language pairs and effectively manage domain-specific terminology.

- **Question Answering Systems**: Knowledge Base Question Answering (KBQA), Machine Reading Comprehension (MRC). Combined with Retrieval-Augmented Generation (RAG), LLMs can answer time-sensitive queries using external knowledge bases, mitigating hallucination issues.

- **Sentiment Analysis**: Public opinion monitoring, review analysis, user feedback mining. LLMs demonstrate superior capabilities over traditional classification models in capturing fine-grained sentiment (e.g., sarcasm, subtle emotions).

- **Reasoning and Planning**: Agent task planning, tool invocation (Function Calling), multi-step reasoning. Combined with Chain-of-Thought (CoT) prompting or the ReAct framework, LLMs automate complex multi-step tasks.

- **Code Assistance**: Code completion, bug fixing, code explanation, and test case generation have become essential aids in software engineering.

- **Multimodal Understanding**: Combined with visual encoders (such as LLaVA, GPT-4V), LLMs process mixed text and image inputs, applied in image captioning, document understanding, and beyond.

</details>

**26. What Are the Limitations of LLMs?**

<details>
<summary>💡 Show Answer</summary>

- **Hallucination**: LLMs can generate content that appears plausible but is factually incorrect, particularly in factual knowledge, numerical calculations, and source citations. The root cause is that LLMs generate text based on statistical probabilities rather than querying structured knowledge bases, so factual consistency cannot be guaranteed.

- **Knowledge Cutoff**: Training data is bounded by a specific date, rendering the model unaware of real-time information (e.g., recent policies, current news). This can be mitigated via RAG or web search integration.

- **Limited Complex Reasoning Ability**: LLMs remain prone to errors in multi-step mathematical reasoning, formal logic, and symbolic computation. Chain-of-Thought (CoT) prompting provides partial improvements, but reliability still falls short of dedicated symbolic systems.

- **Context Length Constraints**: Information exceeding the context window is forgotten (“Lost in the Middle” phenomenon). Although modern context windows span hundreds of thousands of tokens, effectively utilizing ultra-long documents remains challenging.

- **Consistency Issues**: Asking the same question with varied phrasing can yield contradictory answers, as model outputs are highly sensitive to prompt wording.

- **Insufficient Domain Knowledge**: In highly specialized vertical domains (e.g., medicine, law, finance), general-purpose models lack deep expertise, requiring domain-specific fine-tuning or RAG enrichment.

- **High Compute Resource Consumption**: Training and inferencing large-scale LLMs require substantial GPU resources, resulting in high deployment costs and making edge deployment difficult.

- **Poor Interpretability**: The decision-making process of LLMs functions as a black box, making it hard to provide trustworthy explanations for outputs, which poses trust barriers in high-risk scenarios (such as healthcare and law).

</details>

**27. What Ethical and Safety Issues Must Be Considered When Using LLMs?**

<details>
<summary>💡 Show Answer</summary>

- **Bias and Discrimination**: Biases present in training data (e.g., gender, race, culture) can be learned and amplified by the model, leading to unfair outputs that disadvantage marginalized groups.

- **Privacy Protection**: Training data may contain sensitive user information, posing risks of memorization leakage. Studies indicate LLMs can regurgitate personal data (e.g., emails, ID numbers) under specific prompts.

- **Intellectual Property**: Generated content may infringe on copyright, and licensing for training data remains legally controversial (e.g., lawsuits between OpenAI and news publishers).

- **Misuse Risks**: LLMs can be exploited to generate misinformation (deepfake text), phishing content, malicious code, and automated scams, posing threats to societal security.

- **Environmental Impact**: Large model training consumes vast compute resources, resulting in significant carbon emissions. Training GPT-3 generated an estimated 500 tons of CO$_2$, equivalent to the annual emissions of dozens of gasoline cars.

- **Alignment Problem**: Model behavior may deviate from human values and intended goals, giving rise to specification gaming, reward hacking, and other issues—a central focus of AI safety research.

- **Over-reliance and Deskilling**: Over-dependence on LLMs can degrade user skills, causing debate in education, technical writing, and other fields.

- **Regulatory Compliance**: AI content regulation varies by region and jurisdiction (e.g., the EU AI Act), requiring enterprises to navigate complex compliance challenges.





![Attention Mask Patterns across Decoder-Only, Prefix LM, and Encoder-Decoder](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig09_causal_vs_prefix_vs_encdec.png)

</details>

**28. How Does Training Data Bias Affect LLMs, and How Can It Be Mitigated?**

<details>
<summary>💡 Show Answer</summary>

**Impact**: Models learn and replicate biases present in data—such as gender bias (association of occupations with specific genders), racial bias, political bias, and cultural bias. This can produce unfair outputs in downstream tasks and potentially cause systemic harm to specific groups. Furthermore, bias may be amplified as model scale increases (“scaling amplifies bias”).

**Mitigation Methods**:
- **Data Level**: Data cleaning and debiasing (filtering discriminatory content), filtering data diversity during pre-training, and constructing diverse, balanced annotated datasets for fine-tuning to ensure representative coverage across geographic, cultural, and demographic dimensions.

- **Training Level**: Applying alignment techniques such as RLHF (Reinforcement Learning from Human Feedback) to correct biased behaviors via human feedback; incorporating adversarial training to actively detect and penalize biased outputs.

- **Evaluation Level**: Establishing dedicated bias evaluation benchmarks (e.g., WinoBias, BBQ) to systematically evaluate bias prior to model deployment.

- **Deployment Level**: Implementing output filtering and content moderation mechanisms to intercept or flag high-risk outputs for human review.

- **Prompt Engineering Level**: Designing prompts with explicit constraints or persona definitions to guide the model toward generating fairer responses.

</details>

**29. When to use BERT models and when to use Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

\begin{table}[h]



| **Dimension** | **BERT** | **LLaMA / ChatGLM** |
| --- | --- | --- |
| Architecture | Encoder-only | Decoder-only |
| Suitable Tasks | Text classification, NER, similarity, extractive QA | Text generation, dialogue, reasoning, instruction following |
| Inference Mode | Bidirectional understanding, single-pass encoding | Autoregressive token-by-token generation |
| Resource Requirements | Lower, suitable for deployment | Higher, requires GPU |
| Typical Scenarios | Sentiment analysis, entity recognition, sentence pair matching | Intelligent customer service, code generation, content creation |
| Inference Speed | Fast (single forward pass) | Slow (token-by-token autoregression) |
| Finetuning Cost | Low (a few thousand samples suffice) | Relatively high (requires parameter-efficient finetuning like LoRA) |


\end{table}

**Selection Recommendations**:
- If the task is **understanding-based** (classification, matching, extraction), with fixed input and output formats, and sensitive to inference speed and deployment costs, prioritize BERT-series models.
- If the task is **generation-based** (dialogue, writing, reasoning, instruction following), or requires handling open-domain problems, prioritize LLMs.
- For **boundary scenarios** (such as generative QA, summarization), consider Encoder-Decoder architectures (such as T5, BART) to balance understanding and generation capabilities.

</details>

**30. Do specialized domain fields require their own dedicated large models to serve them?**

<details>
<summary>💡 Show Answer</summary>

Not necessarily; it depends on scenario requirements and cost constraints:
- **General LLM + RAG**: Can meet most knowledge retrieval requirements with low cost and fast updates, injecting up-to-date domain knowledge without retraining the model. Suitable for scenarios like knowledge queries and document QA.

- **Domain Finetuned Models (SFT/RLHF)**: Suitable for scenarios with strict requirements on output formats, technical terminology, and industry standards (such as medical diagnostic reports, legal documents, financial research reports), requiring the model to "speak the jargon" and adhere to domain-specific specifications.

- **Continue PreTrain (Domain Continued Pre-training)**: Suitable for cases where the general model lacks sufficient coverage on domain corpus and needs adaptation at the language distribution level (such as finance, code, Chinese law). Continuing pre-training on a large domain corpus allows the model to internalize domain knowledge.

- **Domain-Specific Models Trained from Scratch**: Considered only in extreme scenarios where data privacy is paramount, compliance requirements are strict (such as hospital internal data that cannot be sent externally), or the general model architecture is unsuitable. The cost is extremely high, so it is generally not recommended.

The general trend is: general models are becoming increasingly powerful (GPT-4, Claude 3.5, etc., are already close to expert levels on most professional tasks), and the value of domain models is increasingly reflected in **data privacy, compliance requirements, and extreme performance**.

</details>

**31. Are Encoder-only architectures suitable for generative tasks?**

<details>
<summary>💡 Show Answer</summary>

Currently, Encoder-only structures (such as BERT) are primarily used for encoding information and are not well-suited for generative tasks. Technically, however, generative tasks can be performed by masking the last token to simulate autoregressive decoding.

However, the following core issues exist:

1. **Train-Test Mismatch**: The model was never explicitly optimized during training for "masking only the last token" (BERT's MLM randomly masks 15% of tokens). Therefore, masking the last token during inference diverges from the training distribution, leading to poor generalization.

2. **Inability to Reuse KV Cache**: Encoder-only models use **bidirectional attention**, meaning each token can see all other tokens in the sequence (including those to its right). This causes later tokens to affect representations of earlier tokens, requiring all historical token representations to be recomputed after generating each new token. They cannot reuse KV Cache like Decoder-only models, resulting in extremely low inference efficiency.

3. **Lack of Autoregressive Training Objectives**: The pre-training objectives of Encoder-only models (MLM, NSP) do not match the objective required for generative tasks ("predicting the next token given a prefix"), so the model fails to learn the conditional generation probability distribution $P(x_t \mid x_1, \ldots, x_{t-1})$.

</details>

**32. Can BERT be used for generative tasks?**

<details>
<summary>💡 Show Answer</summary>

Technically yes, but the performance is usually inferior to specialized generative models. BERT's core training tasks include Masked Language Modeling (MLM) and Next Sentence Prediction (NSP). This bidirectional context understanding ability enables BERT to perform exceptionally well in natural language understanding tasks.

When used for generative tasks, BERT can achieve this through the following methods:
- **Text Infilling**: Given a text with missing parts, BERT can predict the missing portions based on bidirectional contextual information. This aligns closely with its MLM pre-training objective, yielding acceptable results.

- **Text Continuation**: This can be achieved using an iterative mask-then-predict technique. However, because BERT is not trained for autoregressive generation, the coherence of the generated text is usually weak.

- **UniLM Approach**: UniLM proposed by Microsoft designs different Attention Masks (unidirectional, bidirectional, Seq-to-Seq hybrid) on top of BERT to support both understanding and generation tasks, serving as a representative work in expanding generative capabilities within the BERT framework.





![Evolution Timeline of LLaMA Series (LLaMA 1 to LLaMA 3)](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig10_llama_evolution_timeline.png)

</details>

**33. Why is BERT less effective for generative tasks compared to modern large models?**

<details>
<summary>💡 Show Answer</summary>

Limitations of BERT in generative tasks:
1. **Insufficient Generation Coherence**: BERT's training objective is to independently predict each masked word, making different masked positions **conditionally independent**. The model does not learn to generate long texts that are coherent and logically consistent throughout. In contrast, autoregressive models like GPT explicitly model the sequential dependencies of text via the chain rule of conditional probability $P(x_1, \ldots, x_n) = \prod_{t=1}^n P(x_t \mid x_{<t})$.

2. **Restricted Generation Length**: BERT's positional encoding upper limit is usually 512 tokens, whereas modern generative LLMs (LLaMA, Qwen, etc.) have expanded context windows to tens or hundreds of thousands of tokens, enabling generation of much longer texts.

3. **Lack of Explicit Generation Objectives**: Specialized generative models (such as the GPT series) are pre-trained via **Causal Language Modeling** (CLM), directly aligning their training objectives with generation tasks; BERT's MLM objective suffers from a fundamental mismatch with autoregressive generation.

4. **Scale Gap**: Modern generative LLMs typically have parameter counts ranging from billions to hundreds of billions (BERT-Large has only about 340 million parameters). Larger parameter scales bring stronger knowledge memorization capacity and generation quality, illustrating the Scaling Law effect.

5. **Lack of Instruction Following Ability**: Current generative LLMs undergo instruction tuning and RLHF, enabling them to understand and follow complex user instructions. BERT has not undergone such alignment training, making it difficult to handle open-ended generation instructions.

</details>

