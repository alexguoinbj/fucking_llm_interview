# 3.2 Pre-training Fundamentals — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 33 questions

**1. Does LLM Training Require Setting a Temperature Coefficient?**

<details>
<summary>💡 Show Answer</summary>

![Effect of Temperature coefficient on Softmax output probability distribution.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig05_temperature_softmax.png)




During the training phase of large language models (LLMs), the temperature coefficient (Temperature) is typically fixed at $T = 1$, which means no scaling is applied to logits, equivalent to not setting a temperature coefficient. The core role of the temperature coefficient is manifested in sampling control during the **inference phase**. Below is a detailed breakdown from the aspects of principles, training phase, and inference phase.

#### Mathematical Definition of Temperature Coefficient

In a language model, given the logits vector $\mathbf{z} \in \mathbb{R}^{|V|}$ at step $t$ ($|V|$ is the vocabulary size), the softmax function with a temperature coefficient is defined as:


$$
	P(w_i \mid \mathbf{z}, T) = \frac{\exp(z_i / T)}{\sum_{j=1}^{|V|} \exp(z_j / T)}
$$


where $T > 0$ is the temperature coefficient. Its behavioral characteristics are as follows:
- When $T = 1$, it reduces to standard softmax, and the probability distribution is unaffected by additional intervention;
- When $T \to 0^+$, the distribution approaches a one-hot vector, and the model tends toward **greedy decoding**;
- When $T \to +\infty$, the distribution approaches a uniform distribution, and the output **randomness increases**.

#### Training Phase: Temperature Coefficient Fixed at 1

In standard autoregressive language model training, the loss function is the cross-entropy loss:


$$
	\mathcal{L} = -\sum_{t=1}^{T} \log P(w_t \mid w_{<t})
$$


At this time, softmax takes $T = 1$ for the following reasons:

1. **Gradient Consistency**: If $T \neq 1$ is introduced during training, it is equivalent to proportionally scaling all logits. The model can "offset" the effect of temperature by adjusting weight magnitudes; thus, the temperature itself does not provide additional learning signals.
2. **Purity of Training Objective**: The objective of training is to let the model learn the true data distribution $P_{data}$. Using standard softmax with $T=1$ ensures that the loss function directly measures the KL divergence between the model distribution and the true distribution without introducing extra biases.
3. **Exception in Knowledge Distillation**: In **Knowledge Distillation** scenarios, the temperature coefficient is explicitly used during training. The softmax of both the teacher model and student model uses the same higher temperature $T > 1$, allowing soft labels to carry richer inter-class similarity information:

    $$
    \mathcal{L}_{KD} = T^2 \cdot KL\!\left(P^{(T)}_{teacher} \,\|\, P^{(T)}_{student}\right)
    $$

    However, this is a special training paradigm and not part of the standard pre-training or fine-tuning workflow.

#### Inference Phase: Temperature Coefficient for Sampling Control

The decoding strategy during the inference phase directly affects the quality and diversity of generated text, where the temperature coefficient plays a critical role:

- **Greedy Decoding** ($T \to 0$): Selects the token with the highest probability at each step; the generation result is deterministic but may get trapped in repetitions;
- **Temperature Sampling** ($0 < T < 1$): Compresses the distribution, increasing the relative weight of high-probability tokens, making generations more conservative and fluent;
- **High-Temperature Sampling** ($T > 1$): Flattens the distribution, increasing the chance of low-probability tokens being selected, making generations more diverse and creative, but potentially introducing noise;
- **Combining with Top-$k$ / Top-$p$**: In practical engineering, temperature sampling is often combined with Top-$k$ sampling or Nucleus (Top-$p$) sampling to achieve a balance between diversity and coherence.

Typical inference call example (taking OpenAI API style as an example):

\begin{lstlisting}[style=promptstyle]
	{
		"model": "gpt-4",
		"messages": [{"role": "user", "content": "Please write a poem about autumn"}],
		"temperature": 0.8,
		"top_p": 0.95,
		"max_tokens": 256
	}
\end{lstlisting}

#### Summary

\begin{table}[h]



| **Phase** | **Temperature Value** | **Role** |
| --- | --- | --- |
| Pre-training / Fine-tuning | $T = 1$ (Fixed) | No intervention; ensures correct loss function semantics |
| Knowledge Distillation | $T > 1$ (Explicitly set) | Softens labels; conveys inter-class relationship information |
| Inference Sampling | $T \in (0, +\infty)$ (Tunable) | Controls the balance between generation diversity and determinism |


	\caption{Usage of temperature coefficient in different phases}
\end{table}

In summary, **there is no need to set a temperature coefficient during the training phase** (default $T=1$). The temperature coefficient is a decoding hyperparameter in the inference phase, used to trade off between generation quality and diversity.

</details>

**2. What Are the Key Steps in the LLM Pre-training Phase?**

<details>
<summary>💡 Show Answer</summary>

![End-to-end LLM Pre-training lifecycle and engineering pipeline.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig06_pretraining_lifecycle.png)




LLM pre-training generally consists of the following key steps:

1. **Data Collection and Cleaning**: Collect massive raw text from multiple sources such as web pages (e.g., Common Crawl), books, code repositories, encyclopedias, and forum Q&As, and perform cleaning work including deduplication, filtering low-quality content, and removing harmful/private information (refer to Questions 463 and 464).

2. **Data Mixing and Ratio Design**: Mix data from different sources/domains in certain proportions to construct the final pre-training dataset (refer to Question 466).

3. **Tokenizer Training/Selection**: Train a tokenizer (such as BPE/SentencePiece) based on (or referring to) the pre-training corpus, determining vocabulary size and coverage scope (refer to Question 477).

4. **Model Architecture Design**: Determine the model architecture (number of layers, hidden dimension, number of attention heads, positional encoding schemes, activation functions, etc.) and parameter scale.

5. **Training Infrastructure Construction**: Set up a distributed training environment (parallelism strategies such as Data Parallelism/Tensor Parallelism/Pipeline Parallelism/ZeRO, refer to Chapter 13) and configure mixed-precision training, etc.

6. **Hyperparameter Setup**: Set learning rates (and scheduling strategies, such as warmup + cosine decay), batch size, optimizer (such as AdamW's beta, weight decay), gradient clipping, etc.

7. **Massive Training Execution**: Take language modeling loss (CLM or MLM, refer to Questions 460 and 461) as the objective to conduct long-term distributed training on data, while continuously monitoring metrics such as loss, gradient norm, and throughput, and handling stability issues during training (e.g., loss spikes).

8. **Checkpoint Management and Evaluation**: Save checkpoints periodically, evaluate the development trend of model capabilities using evaluation benchmarks (such as MMLU) in intermediate stages, and adjust data proportions/hyperparameters when necessary.

9. **Annealing/Learning Rate Decay Phase (Optional)**: Learning rate annealing is often performed in the final stage of training, sometimes combined with an "annealing dataset" of high-quality data to further boost model performance during the annealing phase.

**Summary**: The pre-training workflow as a whole is an end-to-end engineering pipeline of "**Data Preparation (Collection, Cleaning, Mixing) $\rightarrow$ Tokenizer & Architecture Design $\rightarrow$ Distributed Infrastructure & Hyperparameter Setup $\rightarrow$ Massive Training (including Continuous Monitoring & Stability Handling) $\rightarrow$ Checkpoint Evaluation & Necessary Iterative Adjustments**", where data-related work often occupies the core position in terms of workload and importance across the entire workflow.

</details>

**3. What Is the Core Objective of LLM Pre-training?**

<details>
<summary>💡 Show Answer</summary>

The core objective of LLM pre-training is: **to let the model learn the statistical patterns, grammatical structures, and semantic relationships of language, as well as the world knowledge, common sense, and reasoning patterns implicit in text, by conducting self-supervised learning (most commonly "next token prediction") on massive and diverse text (and other modal) data, thereby obtaining a "base model" with broad general capabilities and providing a solid starting point for subsequent fine-tuning and alignment**.

Specifically, the core objective can be understood from several dimensions:

1. **Learning Language Patterns Themselves**: Vocabulary, grammar, syntactic structures, context dependencies, etc.---this is the most fundamental capability of a language model.

2. **Learning World Knowledge**: Through factual statements, encyclopedic knowledge, and common sense contained in massive text, the model "compressively stores" a vast amount of information about the world within its parameters (refer to Question 481 "Why Next Token Prediction Can Learn World Knowledge").

3. **Learning Reasoning and Pattern Recognition Capabilities**: Text contains abundant patterns of logical reasoning, causal relationships, mathematical calculations, etc. By predicting these patterns, the model gradually develops a certain level of reasoning capability.

4. **Forming Transferable General Representations**: The pre-trained model parameters (especially intermediate layer representations) serve as a good initialization for various downstream tasks, significantly reducing the amount of annotated data and training cost required for downstream tasks (this is the core advantage of the "pre-training--fine-tuning" paradigm compared to "training each task model from scratch").

5. **Implicit "Compression" Perspective**: From an information theory perspective, the objective of "next token prediction" is essentially learning the probability distribution of data. A model that can better predict data is equivalent to achieving better "compression" of the data---and there is a profound theoretical link between "compression capability" and "intelligence/understanding capability" (better compression requires a deeper "understanding" of the data generation process).

**Summary**: The core objective is **to learn language patterns, world knowledge, and reasoning patterns from massive data through self-supervised language modeling tasks, forming a base model with powerful general capabilities and good transferability**, laying the foundation for subsequent fine-tuning and alignment targeted at specific tasks/scenarios.

</details>

**4. What Is the Loss Function for LLM Pre-training?**

<details>
<summary>💡 Show Answer</summary>

LLM pre-training typically adopts the training objective of an **autoregressive language model**, where the core loss function is the **Cross-Entropy Loss**, specifically formulated as the average negative log-likelihood over each token in the sequence:


$$
	\mathcal{L} = -\frac{1}{T} \sum_{t=1}^{T} \log P(x_t \mid x_1, x_2, \ldots, x_{t-1}; \theta)
$$


where:
- $T$ is the sequence length;
- $x_t$ is the $t$-th token;
- $\theta$ represents the model parameters;
- $P(x_t \mid x_1, \ldots, x_{t-1}; \theta)$ is the probability of the model predicting the $t$-th token given the prefix (provided by the softmax output layer).

**Intuitive Understanding:** This loss function requires the model, given the preceding context $x_1, \ldots, x_{t-1}$, to maximize the predicted probability of the true next token $x_t$. This is equivalent to minimizing the KL divergence between the model distribution and the true data distribution, as well as maximizing the log-likelihood (MLE) of the training corpus.

**Relationship with Perplexity (PPL):**


$$
	PPL = e^{\mathcal{L}}
$$


Perplexity is commonly used as an evaluation metric for language models and directly corresponds to the training loss.

</details>

**5. Why Was the Next Sentence Prediction (NSP) Task in BERT Deprecated Later?**

<details>
<summary>💡 Show Answer</summary>

**Next Sentence Prediction (NSP) Task**: In BERT pre-training, in addition to MLM (Masked Language Modeling), the NSP task was designed---given two sentences A and B, determine whether B is the next sentence after A in the original text (binary classification: IsNext/NotNext, where negative samples are usually sentences randomly drawn from the corpus).

**Main Reasons Why NSP Was Deprecated by Subsequent Works (e.g., RoBERTa)**:

1. **Task Difficulty Too Low / Low Signal Quality**:
- The negative samples of NSP are "sentences randomly selected from the corpus." Such negative samples and positive samples (true next sentences) often exhibit **very distinct topic/subject differences**---the model can easily perform the NSP task correctly relying merely on the **shallow topic consistency feature** of "whether the two sentences discuss the same topic," without needing to deeply understand finer-grained **coherence/logical relationships** between sentences (such as coreference, causality, etc.);
- In other words, the NSP task actually **mixes two subtasks of different difficulty: "topic prediction" and "coherence prediction"**, and the model primarily relies on the easier-to-learn "topic prediction" signal, resulting in limited help from the NSP task for the model to learn "fine-grained inter-sentence relationships."

2. **Experimental Findings from Works like RoBERTa**:
- The RoBERTa paper discovered through comparative experiments that **removing the NSP task (retaining only MLM) resulted in downstream performance comparable to or even better than keeping NSP**;
- Furthermore, RoBERTa explored different input construction methods such as "FULL-SENTENCES" (inputs composed of contiguous sentences concatenated together, which can cross document boundaries) and "DOC-SENTENCES" (inputs composed of contiguous sentences from a single document), finding that **even without the NSP loss, training via MLM on "longer contiguous text segments" alone yields sufficiently good or even superior performance**;

3. **Training Efficiency Considerations**:
- The NSP task requires training samples to be "paired sentences," imposing additional constraints on data organization (requiring positive and negative sample pair construction). Compared to directly splitting long text into contiguous segments for MLM training, **the data processing pipeline is more complex**, and NSP itself contributes limited effective signals, offering a low cost-effectiveness ratio.

4. **Proposals of Subsequent Improved Tasks**:
- Some subsequent works (such as ALBERT) proposed the **SOP (Sentence Order Prediction)** task as an improvement over NSP---the negative samples in SOP are formed by "swapping the order of two contiguous sentences" (rather than "randomly sampling irrelevant sentences"). Thus, the **topic remains consistent** between negative and positive samples, and the difference lies solely in **order/coherence**, thereby **focusing more on the valuable signal of "coherence"** and avoiding the "easy shortcut" of topic difference in NSP. This also corroborates from another angle the intrinsic flaw in NSP's task design---its negative sample construction made the task "too easy," failing to effectively drive the model to learn deeper inter-sentence relationships.

**Summary**: The core reason for deprecating NSP is that **its negative sample construction method (randomly selecting irrelevant sentences) allowed the task to be easily solved via "shallow topic consistency judgment," failing to effectively drive the model to learn deeper inter-sentence coherence/logical relationships, with limited actual contribution to pre-training effectiveness**; experiments in works like RoBERTa demonstrated that removing NSP and relying solely on MLM (with longer contiguous text inputs) yields comparable or better performance. Consequently, subsequent models generally deprecated NSP, while some works (such as ALBERT's SOP) attempted to refine the task design to retain the goal of "inter-sentence relationship modeling" while avoiding NSP's flaws.

</details>

**6. What Is the Training Objective of Causal Language Modeling (CLM)?**

<details>
<summary>💡 Show Answer</summary>

![Comparison of Causal Attention Masking (CLM) and Bidirectional Masking (MLM).](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig07_clm_vs_mlm_masking.png)




**CLM (Causal Language Modeling)**, also known as Autoregressive LM, is the pre-training objective adopted by current mainstream LLMs (such as the GPT series and LLaMA series).

**Training Objective**:

Given a sequence of tokens $x_1, x_2, \ldots, x_T$, the objective of CLM is to maximize the (conditional) joint probability of the entire sequence, decomposed via the chain rule as:

$P(x_1,\ldots,x_T) = \prod_{t=1}^{T} P(x_t \mid x_1,\ldots,x_{t-1})$

That is, the model needs to predict the probability distribution of the **next token** based on **all preceding appeared tokens (historical context)**. The training loss is the negative log-likelihood (NLL, i.e., cross-entropy loss):

$L_{CLM} = -\sum_{t=1}^{T} \log P_\theta(x_t \mid x_1,\ldots,x_{t-1})$

**Key Characteristics**:

1. **Unidirectional (Left-to-Right) Context Dependency**: When predicting the token at position $t$, the model **can only "see" information from positions $1$ to $t-1$, and cannot see content at position $t$ and beyond**---this is implemented by applying a **causal mask (Causal Mask / lower triangular mask)** in the Self-Attention calculation, ensuring that the attention calculation at each position can only attend to itself and preceding positions.

2. **Naturally Suited for Generation Tasks**: Since the training objective itself is "predicting the next token based on existing content," the model can **naturally generate new content autoregressively token-by-token** during inference---the training objective is highly consistent with the inference usage (generation), which is a key advantage of CLM over MLM (which requires special handling during inference to be used for generation).

3. **Parallel Loss Calculation for All Positions Within a Single Sequence**: Although autoregressive generation is performed sequentially token-by-token, during the **training phase**, because the entire sequence is known, a causal mask enables **simultaneous calculation of prediction losses for all positions in the sequence in a single forward pass** (the loss at each position $t$ depends only on information from $1$ to $t-1$, all of which are known during training); thus, training itself is highly parallelizable.

**Summary**: The training objective of CLM is **to maximize the probability of predicting the next token given the context (i.e., minimizing the negative log-likelihood / cross-entropy loss)**, ensuring that each position can only utilize left-sided context information via causal masking. This "unidirectional, autoregressive" modeling approach is naturally consistent with how generation tasks are performed, serving as the standard objective for current mainstream LLM pre-training.

</details>

**7. What Is the Training Objective of Masked Language Modeling (MLM)?**

<details>
<summary>💡 Show Answer</summary>

**MLM (Masked Language Modeling)** is a pre-training objective adopted by models such as BERT. Unlike the "unidirectional autoregressive" nature of CLM, MLM is a **bidirectional** pre-training objective.

**Training Objective**:

Given a sequence of tokens, a fraction of positions (e.g., 15%) is **randomly selected for "masking"** (replacing the original token with a special [MASK] token, a random token, or leaving it unchanged—specifically in BERT, 80% are replaced with [MASK], 10% with a random token, and 10% kept unchanged). The model's objective is to **predict the original tokens at the masked positions based on the bidirectional context (i.e., all unmasked tokens to the left and right of that position)**.

The training loss is computed only on the **masked positions**:

$L_{MLM} = -\Sigma_{i \in masked positions} \log P_\theta(x_i | x_{masked})$

where $x_{masked}$ represents all tokens except the masked positions (including the context to both the left and right of the masked positions).

**Key Characteristics**:

1. **Bidirectional Context Modeling**: Unlike CLM's causal mask (which can only look to the left), MLM's self-attention **does not require a causal mask**—the representation at each position (including masked positions) can fully leverage all context information from **both its left and right sides**. This gives the representations learned by MLM a typical advantage in tasks that require "understanding the context of the whole sentence" (e.g., classification, NER, sentence similarity evaluation).

2. **Train-Test Discrepancy**: During MLM training, the input contains [MASK] tokens, whereas **in actual inference/fine-tuning, the input typically does not contain [MASK]** (unless the task itself is "cloze filling"). This inconsistency of "seeing [MASK] during training but not during inference" is a theoretical limitation of MLM (partially mitigated by BERT's 80/10/10 strategy, but not completely eliminated).

3. **Not Directly Suitable for Autoregressive Generation**: Because MLM trains the model to "predict a position based on bidirectional context" rather than **"predict the next token based on left context"**, models pre-trained with MLM **cannot be directly and naturally used for autoregressive text generation like CLM models** (although special methods exist to use MLM models for generation, none are as natural as CLM models).

4. **Only the masked portion (e.g., 15%) in each sequence participates in loss computation**. Compared to CLM where almost all positions in the sequence (except the first token) contribute to the loss, **MLM has a relatively lower "information utilization efficiency" per training sample** (requiring more training samples/steps to reach the same total amount of "effective supervisory signal" as CLM).

**Summary**: The training objective of MLM is to **randomly mask a portion of tokens in the input sequence and predict these original masked tokens based on bidirectional context**. By removing the causal mask, it achieves bidirectional context modeling, making it better suited for discriminative tasks requiring "overall understanding" (such as classification, NER, etc.). However, it suffers from limitations such as "train-test input discrepancy" and "lack of direct adaptability for autoregressive generation," which is an important reason why current mainstream large generative LLMs generally adopt CLM rather than MLM as their pre-training objective.

</details>

**8. Under What Conditions Does the Scaling Law Fail? Are There Counterexamples?**

<details>
<summary>💡 Show Answer</summary>

![Chinchilla Scaling Laws for compute-optimal LLM pre-training.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig09_scaling_laws.png)




**Basic Content of Scaling Law**:

The Scaling Law describes the **power-law relationship** between model performance (typically measured by test loss or perplexity) and the number of model parameters $N$, dataset size $D$, and compute budget $C$. Within a certain range, increasing $N$, $D$, and $C$ leads to a continuous drop in model loss according to a power-law rule, and this relationship exhibits **good predictability across a broad range (allowing extrapolation from small-scale experiments to large scales)**.

**Conditions Where the Scaling Law May Fail or No Longer Apply**:

**1. Data Quality / Data Volume Bottleneck**

- The "data dimension" of the Scaling Law assumes that newly added data brings a certain amount of "information gain." If **available high-quality data is nearly "exhausted"** (i.e., continuing to increase the "data volume $D$" is mainly achieved by increasing epoch repetitions or lowering quality thresholds rather than adding truly "new, informative" data), then the performance improvement from increasing data volume will **significantly deviate from the original power-law curve, with the rate of improvement slowing down drastically or even saturating**. This is at the core of many current discussions around the "data wall" problem and is considered a practical bottleneck that the Scaling Law may encounter along the "data dimension."

**2. "Saturation" of Evaluation Metrics**

- The Scaling Law is typically observed based on continuous, smooth metrics such as **pre-training loss or perplexity**. However, for **discrete metrics of downstream tasks (such as accuracy)**, when the model's accuracy on a certain task approaches 100% ("saturation"), even if the pre-training loss continues to drop according to a power law, the downstream task accuracy **can no longer improve (having reached its upper limit)**. That is, the correspondence between "power-law reduction in loss" and "improvement in specific downstream metrics" breaks down in the **near-saturation region**.

**3. Discontinuity of "Emergent Abilities" (Controversial Counterexample)**

- Some studies (such as "Emergent Abilities of Large Language Models") report that certain capabilities (such as multi-step arithmetic or specific few-shot tasks) remain **near zero (close to random guessing) until the model scale crosses a certain threshold, after which they suddenly improve significantly**. This "emergence" appears to be a **deviation from the Scaling Law intuition that "performance improves smoothly with scale"** (the performance curve exhibits a "step function" rather than a "smooth power law").
- However, subsequent studies (such as "Are Emergent Abilities of Large Language Models a Mirage?") point out that **this "emergence" phenomenon is largely an "illusion" caused by the choice of evaluation metrics**. If continuous metrics (such as log-likelihood or metrics measuring "partial correctness") are used instead of discrete ones (such as accuracy where only "completely correct" counts), the underlying performance improvement is actually **still smooth and follows a power law**. "Emergence" is merely a visual effect produced when a smooth improvement crosses a discrete threshold. This point itself remains a subject of ongoing debate in academia without full consensus.

**4. Different "Recipes" (Architectures, Data Ratios, Training Methods) Lead to Different "Coefficients" in Scaling Laws**

- Research such as Chinchilla points out that the specific "coefficients" in Scaling Laws (such as the optimal $N/D$ ratio) **depend on the specific training recipe**. Switching to different architectures, data distributions, or optimizer settings might leave the power-law relationship itself intact, but the **specific curve parameters (such as the optimal ratio) will change**. Therefore, a "Scaling Law fitted under a specific recipe" **cannot be naively and indiscriminately generalized to a completely different recipe**. This also represents a form of "failure" (not that the power-law relationship itself is wrong, but that the extrapolation range of specific parameters is limited).

**5. Scaling Patterns in the Post-Training Phase (such as RL) May Differ**

- The Scaling Law was originally established primarily for the relationship between loss and compute during the **pre-training phase**. For the **RL training phase**, whether the relationship between "compute investment" and "model capability improvement" follows a similar power-law form is a relatively new and ongoing topic of exploration. While some recent work has begun studying "scaling" laws in the RL phase, whether its mathematical form and stability align with pre-training Scaling Laws remains inconclusive.

**Summary**: The Scaling Law may exhibit "failure" or "no longer directly apply" under conditions such as **"high-quality data nearing exhaustion (data wall), downstream discrete metrics approaching saturation limits, extrapolation across different training recipes, and whether post-training phases like RL follow similar patterns."** Whether the so-called "emergent abilities" constitute a "counterexample" to the Scaling Law remains controversial in academia (some studies argue that this is largely an "illusion" caused by metric discretization, while the underlying pattern remains smooth). Overall, the Scaling Law exhibits strong predictive power under conditions where "model scale, data volume, and compute budget are all within a 'reasonable range' and the training recipe is relatively fixed." However, when these conditions are broken (data bottlenecks, metric saturation, drastic recipe changes, or new training paradigms like large-scale RL), its applicability must be re-evaluated.

</details>

**9. Given a Fixed Budget, Should You Train a Larger Model or Use More Data? What Different Answers Do Chinchilla vs. LLaMA Provide?**

<details>
<summary>💡 Show Answer</summary>

**Chinchilla's Conclusion**:

DeepMind's Chinchilla paper ("Training Compute-Optimal Large Language Models") systematically investigated the question: "Under a fixed compute budget (FLOPs), how should the number of model parameters $N$ and the amount of training data (number of tokens) $D$ be allocated to minimize the final pre-training loss?" Its core conclusions are:

- Prior to this (e.g., models like GPT-3), the industry generally favored **"prioritizing scaling model parameters while keeping data volume relatively insufficient"**—model sizes grew rapidly while training token counts grew relatively slowly. As a result, many large models were actually **"under-trained"** (meaning that for a given parameter scale, training with more data would further reduce the loss).
- Through extensive experiments, Chinchilla fitted the scaling relationships for "compute-optimal" $N$ and $D$, discovering that **$N$ and $D$ should scale in roughly equal proportions with compute budget $C$** (i.e., "for every doubling of model parameters, the training data volume (tokens) should also double," with both growing at roughly equal rates).
- Based on this conclusion, the Chinchilla model (70B parameters, but trained on far more tokens than previous models of comparable scale) **significantly outperformed Gopher (280B parameters)**, which had a larger parameter count but relatively insufficient training data, under the exact same training compute budget. In other words, "using the same compute budget to train a 'smaller model trained more thoroughly on more data' yields better results than a 'larger model that is under-trained.'"

**LLaMA's Practice**:

An important observation/argument from the LLaMA paper series is that: **Chinchilla's "compute-optimal" analysis focuses on the goal of "achieving the lowest pre-training loss given a training compute budget," but in practical applications, a model's "inference cost" is also a crucial factor that must be considered**:

- A larger model trained "at the Chinchilla-optimal ratio" may be optimal in terms of "training compute efficiency," but its **inference cost (inference latency and VRAM footprint determined by parameter size) is correspondingly higher**.
- LLaMA's strategy is to **choose a relatively smaller model size (e.g., 7B, 13B, etc., with parameter counts smaller than what the strict Chinchilla-optimal ratio recommends for that compute budget) while training on far more tokens than suggested by the Chinchilla-optimal ratio**. In other words, **"while this configuration might not be optimal in the strict $N/D$ ratio sense of 'training compute optimality,' the LLaMA team argued that for a model intended to be widely deployed for massive inference across various hardware, pairing 'smaller parameter size with longer training (more data/tokens)' significantly reduces post-deployment inference costs while achieving comparable or superior capability levels—a trade-off of far greater practical value."**
- Experimental results demonstrated that LLaMA-13B performed comparably to or even better than GPT-3-175B (which has a vastly larger parameter count) across multiple benchmarks. This indicated that **"continuing to add training data (even beyond the $D/N$ ratio corresponding to 'Chinchilla optimal') still yields sustained capability gains for smaller models."**

**The Different "Answers" Provided by the Two—In Essence, a Difference in Optimization Goals**:

- **Chinchilla's Perspective**: Under a "fixed training compute budget," solve for the optimal allocation of $N$ and $D$ that minimizes "pre-training loss"—providing a scaling relationship where $N$ and $D$ "grow synchronously in proportion."
- **LLaMA's Perspective**: Considering the **overall trade-off between "training cost" and "inference cost"**, it argues that for models intended for widespread deployment, **modest "sacrifice" of training-stage "compute optimality" (i.e., not strictly expanding model size according to the Chinchilla ratio, but maintaining a smaller parameter count while investing more data/training time) can be exchanged for significant reductions in "inference cost," while model capabilities can still reach high levels through "training smaller models on more data" (within practical bounds)**.
- The two are not in contradiction; rather, **under the premise of whether inference cost is included in the optimization objective, they offer different practical recommendations for how to allocate the training compute budget**—Chinchilla focuses on "training efficiency," while LLaMA focuses on "overall optimality across both training efficiency and inference efficiency."

**Summary**: On the question of "training a larger model vs. using more data" under a fixed budget, **Chinchilla's answer is to "scale model size and data volume synchronously in a specific ratio to minimize pre-training loss under a given training compute budget" (correcting the previous imbalance of "models growing too fast relative to data"); LLaMA's practice builds on Chinchilla's findings by factoring in the additional dimension of "inference cost," favoring "maintaining a relatively smaller model size while investing far more training data than suggested by the Chinchilla ratio" to drastically lower post-deployment inference costs while achieving comparable capability levels**. The difference fundamentally reflects two different optimal strategies arising from two distinct optimization objectives: "optimizing training-stage compute efficiency alone" vs. "comprehensively considering overall costs across both training and deployment/inference."

</details>

**10. In Multi-Task Learning, How Should You Handle Significant Differences in Loss Across Tasks?**

<details>
<summary>💡 Show Answer</summary>

This can be resolved through the following methods:
- Dynamically adjust loss weights
- Use task-specific loss functions
- Modify model architecture
- Introduce regularization
The goal is to balance the contribution of each task to optimize the model's training effectiveness.

</details>

**11. How Does Hyperparameter Tuning Affect the Performance of LLMs?**

<details>
<summary>💡 Show Answer</summary>

Hyperparameter tuning is the process of optimizing model hyperparameters (such as learning rate, batch size, and dropout rate) to achieve optimal performance. For LLMs, this is particularly critical because:

- **Impact on Convergence**: For instance, the choice of learning rate determines the convergence speed of the model during training. An improperly tuned learning rate may lead to slow convergence or even prevent the model from learning effectively.
- **Preventing Overfitting/Underfitting**: Properly tuning regularization parameters (such as dropout) helps prevent the model from overfitting the training data while maintaining high performance on unseen data.
- **Maximizing Accuracy**: Fine-tuning hyperparameters can significantly boost model accuracy on specific tasks, particularly in domains such as language generation, translation, or summarization.
Effective hyperparameter tuning is a key step in optimizing LLM performance for specific tasks or datasets.

</details>

**12. When the Batch Size Increases, How Should the Learning Rate Change Accordingly?**

<details>
<summary>💡 Show Answer</summary>

Analysis from the perspective of variance:
- When batch size increases by a factor of $x$, the learning rate increases by a factor of $\sqrt{x}$ (which performs best in practice).
- Analysis from the perspective of loss: The learning rate increases monotonically with batch size, but has an upper bound.

OpenAI found that using a large batch size paired with a large learning rate achieves the same final effect as using a small batch size paired with a small learning rate.

</details>

**13. How to Adjust Batch Size and Learning Rate Under the Adam Optimizer?**

<details>
<summary>💡 Show Answer</summary>

When the batch size is expanded by a factor of $c$:
1. The learning rate should be scaled to $\sqrt{c}$ times the original value.
2. The momentum decay rates are adjusted to: $\beta_1' = \beta_1^{-1/c}$, $\beta_2' = \beta_2^{-1/c}$.

</details>

**14. Do Large Models Truly Possess Self-Correction Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Numerous studies indicate that in the absence of external feedback, large models possess virtually no self-correction capability. Self-correction capability cannot be achieved merely through simple supervised fine-tuning algorithms; even methods such as STaR or RFT struggle to be effective.

Current challenges faced:
1. How to retain the correct answers generated by the model to prevent them from being erroneously modified or overwritten.
2. How to correct the wrong answers generated by the model so that it can return to the correct result.
3. How to ensure the accuracy of the model's initial answer distribution during training.

</details>

**15. How Does the Domain Ratio of Pre-Training Corpora Affect the Model's Downstream Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Analysis from the perspective of data distribution:

The domain ratio of pre-training corpora directly determines the model's "knowledge structure," and the proportion of data from different domains profoundly impacts the model's performance in downstream tasks.

#### Core Mechanism of Ratio Impact
Language models are essentially fitting the distribution of training data. If data from a particular domain accounts for a high proportion, the model acquires stronger capabilities to model that domain's language patterns, knowledge structures, and reasoning methods. Conversely, domains with an extremely low proportion remain virtually in an "underfitted" state within the model.

#### Typical Domains and Their Roles
- **Web Text (e.g., Common Crawl)**: Broad coverage, aiding general language capability, but noisy with inconsistent quality.
- **Books**: Long text, coherent logic, aiding long-range dependency modeling and narrative reasoning capabilities.
- **Code**: Significantly boosts the model's logical reasoning, structured output, and mathematical capabilities, showing positive transfer effects even on non-code tasks (such as Chain-of-Thought capabilities).
- **Academic Papers (e.g., ArXiv)**: Enhances scientific reasoning and domain knowledge density.
- **Dialogue Data**: Strengthens the model's instruction-following and interaction capabilities.

#### Consequences of Imbalanced Ratios
- Insufficient code data: Model performance on programming tasks and logical reasoning is noticeably weaker than models of comparable scale.
- Excessive low-quality web data: Model is prone to producing repetitive, redundant, or biased outputs.
- Over-concentration in a single domain: Downstream generalization capability degrades, leading to domain overfitting.

#### Mixing Strategies in Practice
In practice, corpora are usually not mixed according to raw crawl proportions; instead, high-quality data sources are manually upsampled. For example, GPT-3 significantly upsampled Books and Wikipedia, while LLaMA introduced substantial amounts of code data to enhance reasoning capabilities. The optimal mixing ratio often needs to be determined through small-scale ablation studies, as there is currently no unified theoretical guidance.

</details>

**16. Why Do Large Models Train for Very Few Epochs?**

<details>
<summary>💡 Show Answer</summary>

Analysis from the perspective of data scale and overfitting:

#### Core Reason: Data Volume Far Exceeds Model Needs
The dataset scale used for pre-training large models typically reaches the trillion-token level. Even if a model's parameter count reaches hundreds of billions, it still cannot "consume" all the data relative to its size. Under these conditions, training for just 1 epoch or even less than 1 epoch exposes the model to a massive and diverse set of samples, making the marginal returns of repeated training extremely low.

#### Guidance from Chinchilla's Law
The Chinchilla Scaling Law proposed by Hoffmann et al. points out that under a fixed compute budget, the optimal strategy is to maintain a model parameter-to-training token ratio of approximately 1:20. This means for a 70B model, the optimal training volume is around 1.4T tokens, rather than repeatedly training on the same data multiple times.

#### Harms of Repeated Training
- **Overfitting**: The model begins memorizing specific samples from the training data, leading to degraded generalization capability.
- **Data Memorization and Privacy Risks**: Multiple training repetitions make the model more prone to verbatim memorization of training text, giving rise to privacy leakage risks.
- **Reduced Diversity**: Model output tends toward high-frequency patterns in the training set, diminishing creativity and diversity.

#### Comparison with Small Models
Traditional small models (such as BERT), due to limited data volume, typically require training over multiple epochs. In contrast, the sheer data scale of large models provides a natural regularization, eliminating the need to rely on multi-epoch iterations for thorough learning.

</details>

**17. What Are the Parameter Initialization Strategies for Pre-Trained Models? What Are the Key Considerations for Setting the Variance in Random Initialization?**

<details>
<summary>💡 Show Answer</summary>

Analysis from the perspective of numerical stability:

#### Common Initialization Strategies
- **Xavier Initialization (Glorot)**: Sets the weight variance to $\frac{2}{n_{in} + n_{out}}$, suitable for sigmoid/tanh activation functions, with the goal of keeping the variance of activation values consistent during forward and backward propagation.
- **He Initialization (Kaiming)**: Sets the weight variance to $\frac{2}{n_{in}}$, designed specifically for ReLU-like activation functions, accounting for the variance halving caused by ReLU setting the negative half-axis to zero.
- **Small-Variance Normal Distribution Initialization**: Commonly seen in the GPT series, using a normal distribution with mean 0 and standard deviation 0.02 to initialize all linear layer weights.
- **Residual Scaling Initialization**: Performs additional scaling down on the output projection layers within residual connections (such as $W_O$ in Attention and the second layer of FFN), scaling the standard deviation to $\frac{0.02}{\sqrt{2L}}$, where $L$ is the number of layers, to prevent variance explosion caused by residual accumulation in deep networks.

#### Key Considerations for Variance Setting
- **Preventing Vanishing/Exploding Gradients**: If the initialization variance is too large, activation values grow exponentially during forward propagation; if too small, the signal attenuates layer by layer, preventing gradients from propagating effectively.
- **Relation to Network Depth**: The deeper the model, the more severe the variance accumulation along residual paths, necessitating additional variance compression on specific layers.
- **Special Treatment for Embedding Layers**: Word embedding layers are typically initialized with a smaller standard deviation (such as 0.02) to prevent excessively large initial logits variance from causing softmax saturation.

</details>

**18. What is the Impact of Vocabulary Size Selection on Model Training and Inference?**

<details>
<summary>💡 Show Answer</summary>

![Trade-offs between Vocabulary Size, sequence length, and embedding parameters.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig17_vocab_size_tradeoff.png)



Analyzed from the perspectives of computational efficiency and expressiveness:

#### Basic Trade-offs of Vocabulary Size
Vocabulary size is a core hyperparameter in tokenizer design that directly affects the model's input representation, computational overhead, and language expressiveness.

#### Issues with an Overly Small Vocabulary
- The same word is split into more tokens, leading to increased sequence length and higher attention computational overhead.
- For low-frequency words, specialized terminology, and multilingual characters, the tokenization granularity becomes too fine, making it difficult for the model to learn complete semantic units.
- The context window is inefficiently occupied, reducing the effective information density.

#### Issues with an Overly Large Vocabulary
- The parameter counts of the embedding matrix and LM Head (output projection layer) grow linearly with vocabulary size, consuming a large amount of GPU memory.
- Rare tokens appear extremely infrequently during training, resulting in insufficiently trained embedding vectors of poor quality.
- Softmax computation overhead increases (although this can be mitigated using parallel softmax).

#### Choices of Mainstream Models
Currently, the vocabulary size of mainstream large models usually ranges from 32k to 128k: GPT-3 uses about 50k, LLaMA 2 uses 32k, and LLaMA 3 expands to 128k to enhance multilingual and coding capabilities. Expanding the vocabulary combined with a more efficient tokenizer (such as tiktoken) can improve expressiveness without significantly increasing inference overhead.

</details>

**19. Why is Dropout Usually Set to 0 During the Pre-training Phase?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspectives of data scale and regularization requirements:

#### Essential Role of Dropout
Dropout is a regularization technique that prevents model overfitting to training data by randomly dropping neurons during training. Its core assumption is that training data is relatively limited compared to model capacity, creating a risk of overfitting.

#### Why Dropout is Not Needed in LLM Pre-training
- **Extremely Large Data Volume**: Pre-training data reaches trillions of tokens. It is almost impossible for the model to overfit on such a massive dataset, providing a natural regularization effect.
- **Extremely Few Training Epochs**: Models are typically trained for less than 1 epoch, with each sample seen only once, making the risk of overfitting extremely low.
- **Dropout Causes Training Instability**: In large models, Dropout introduces additional randomness, interfering with the accuracy of gradient estimation, which may lead to unstable training or slower convergence.
- **Reduces Training Efficiency**: Dropout requires generating random masks during training, increasing computational and memory overhead, while its regularization benefits are practically negligible in large-scale pre-training scenarios.

#### Different Handling During the Fine-tuning Phase
During the fine-tuning phase on downstream tasks, because the data volume is usually small and the risk of overfitting rises, Dropout is re-introduced (typically set to 0.1) as a regularization technique.

</details>

**20. How to Determine Whether an LLM Has Sufficiently Converged During Pre-training? What are the Criteria?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspective of training dynamics monitoring:

#### Core Metrics: Training Loss and Validation Loss
- **Training Loss Stabilizing**: When the rate of decline in training loss approaches zero with no noticeable decreasing trend over a long period (e.g., tens of thousands of steps), the model can be considered close to convergence under the current data distribution.
- **Validation Loss Stopping Decreasing**: On a held-out validation set, perplexity or cross-entropy loss no longer improving is an important signal of convergence.
- **Gap Between Training and Validation Loss**: If the gap between the two continues to widen, it indicates overfitting; if both stabilize synchronously, the convergence state is good.

#### Supplementary Criteria
- **Gradient Norm**: Upon convergence, the gradient norm should stabilize at a small value; continuous severe fluctuations indicate that training is still unstable.
- **Downstream Task Evaluation**: Periodically evaluate on standard benchmarks (such as MMLU and HellaSwag); if performance no longer improves, the model can be considered converged.
- **End of Learning Rate Schedule**: Most large models adopt cosine decay learning rate scheduling; when training reaches the point where the learning rate decays near its minimum value, it typically also signifies that training is nearing its end.

#### Practical Challenges
Training large models is extremely expensive, making it impossible to wait indefinitely for convergence. In practice, the optimal number of training tokens is often pre-calculated based on Scaling Laws, and training is stopped after reaching that number rather than waiting for the loss to completely plateau.

</details>

**21. What Mutual Impacts Occur Between Low-Resource and High-Resource Languages During Multilingual Pre-training?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspective of cross-lingual transfer:

#### Positive Transfer: Low-Resource Languages Benefit from High-Resource Languages
- **Cross-lingual Knowledge Sharing**: Different languages exhibit alignment relationships in the semantic space. Grammatical structures and world knowledge learned from high-resource languages (such as English) can transfer to low-resource languages, allowing the latter to achieve good comprehension capabilities even with scarce training data.
- **Multilingual Alignment**: Shared subword vocabularies (such as SentencePiece BPE) cause partial overlaps in the token spaces of different languages, promoting cross-lingual representation alignment.
- **Zero-Shot Cross-lingual Transfer**: Reasoning capabilities trained on English can transfer zero-shot to other languages, as demonstrated by mBERT and XLM-R.

#### Negative Impacts: Curse and Competition
- **Curse of Multilinguality**: Under a fixed model capacity, an increasing number of languages causes the expressive capacity per parameter allocated to each language to decrease, which may lead to performance degradation in high-resource languages.
- **Data Imbalance Leading to Underfitting**: Due to the small data volume of low-resource languages, even with upsampling, the model's modeling of their linguistic characteristics remains insufficient.
- **Vocabulary Competition**: High-resource languages occupy a large number of token slots in the shared vocabulary, causing characters/morphemes of low-resource languages to be split into more tokens, thereby reducing token efficiency.

#### Mitigation Strategies
Upsample low-resource languages using temperature sampling to balance the training frequency across languages; increase model capacity to mitigate the curse of multilinguality; and design dedicated vocabulary extensions for low-resource languages.

</details>

**22. What Forms of Attention Masks Are Used in Pre-training? What Training Objectives Are They Suited For?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspective of attention mechanism design:

#### 1. Causal Mask (Autoregressive Mask)
- **Form**: A lower triangular matrix where each token can only attend to itself and tokens to its left, with future tokens masked.
- **Applicable Objective**: Causal Language Models (CLM), such as the GPT series and LLaMA. The training objective is Next Token Prediction, which naturally requires the model not to see future information.

#### 2. Bidirectional Mask (No Mask)
- **Form**: An all-ones matrix where all tokens can attend to each other without directional constraints.
- **Applicable Objective**: Masked Language Models (MLM), such as BERT. The model needs to utilize bidirectional contextual information to predict masked tokens, so no causal constraints can be imposed.

#### 3. Prefix Mask
- **Form**: Bidirectional attention is used within the prefix part (prompt), while causal attention is used in the generation part.
- **Applicable Objective**: PrefixLM architectures (such as T5's Decoder and GLM), balancing bidirectional comprehension capability for encoding the prefix with autoregressive properties for generation, suitable for seq2seq and conditional generation tasks.

#### 4. Padding Mask
- **Form**: Masks padding token positions in the sequence so that they do not participate in attention computation.
- **Applicable Objective**: A general auxiliary mask for handling variable-length sequences across all training scenarios, typically applied on top of the three mask types above.

#### 5. Pack Mask (In-Document Mask)
- **Form**: In the Pack training strategy, tokens from different documents within the same sequence are masked from each other, allowing only tokens within the same document to attend to one another.
- **Applicable Objective**: When concatenating multiple short documents for training, it prevents cross-document information contamination and preserves the semantic independence of each document.

</details>

**23. Why Is the Transformer Architecture Preferred Over LSTM in LLM Pre-training?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspectives of computational efficiency and modeling capacity:

#### Core Limitations of LSTM
- **Sequential Dependency, Inability to Parallelize**: The hidden state $h_t$ of LSTM depends on $h_{t-1}$, requiring serial computation step-by-step during training without sequence-dimension parallelization, resulting in extremely low GPU utilization.
- **Long-range Dependency Degradation**: Although LSTM mitigates the vanishing gradient problem through gating mechanisms, on ultra-long sequences (thousands of tokens), information is still gradually compressed and forgotten, making it difficult to model long-range dependencies.
- **Hidden State Information Bottleneck**: All historical information is compressed into a fixed-dimensional hidden state vector, placing an upper bound on expressiveness.

#### Core Advantages of Transformer
- **Fully Parallel Training**: Self-Attention computations across all token pairs in a sequence are mutually independent, enabling complete parallelization along the sequence dimension and fully leveraging the massive parallel computing power of modern GPUs/TPUs, far exceeding LSTM in training efficiency.
- **Direct Modeling of Dependencies at Any Distance**: The attention weight between any two tokens requires only a single matrix operation and is not limited by sequence length, making long-range dependency modeling fundamentally superior to RNNs.
- **Strong Scalability**: Transformer performance continuously improves as model scale (parameter count, depth, width) increases, conforming to Scaling Laws, whereas LSTM has relatively limited scalability.
- **Mature Engineering Ecosystem**: Optimization tools surrounding Transformers (such as Flash Attention, Tensor Parallelism, etc.) have reached a high level of maturity.

</details>

**24. What Is the Necessity of Pre-training Data Deduplication? What Specific Harms Do Duplicate Data Cause?**

<details>
<summary>💡 Show Answer</summary>

![MinHash and LSH document deduplication algorithmic flow.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig22_lsh_minhash_deduplication.png)




Analyzed from the perspectives of data quality and model behavior:

#### Necessity of Deduplication
Large-scale web-scraped data contains a massive amount of duplicate content, including exact duplicate pages, near-duplicate paragraphs, and cross-document paragraph-level repetitions. If trained directly without processing, the model will implicitly overlearn this content.

#### Specific Harms of Duplicate Data
- **Training Data Memorization**: Models memorize high-frequency duplicate content verbatim and may output raw training data verbatim during inference, causing privacy leakage and copyright risks. Research shows that data duplication count is positively correlated with the probability of memorization.
- **Distribution Shift**: Duplicate data causes the training distribution to deviate from the true natural language distribution, assigning excessively high probability to repeating text patterns and leading to repetitive and monotonous generated content.
- **Wasted Computational Resources**: Repeatedly training on identical or similar content yields no new knowledge while occupying precious training compute and token budget.
- **Evaluation Data Contamination**: If the training set contains content overlapping with the test set, the model's performance on benchmarks will be artificially inflated, failing to reflect true generalization capability.

#### Common Deduplication Methods
- **Exact Deduplication**: Deduplication based on exact matching of documents or paragraphs using MD5/SHA hashes.
- **Fuzzy Deduplication**: Uses MinHash + LSH algorithms to detect near-duplicate documents, which is currently the mainstream method for large-scale data deduplication (adopted by datasets such as C4 and The Pile).
- **n-gram Overlap Detection**: Calculates the n-gram overlap ratio between documents to filter out highly similar document pairs.

</details>

**25. What Are the Benefits of Using the Pack Strategy in Pre-training? Are There Any Potential Issues?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspectives of training efficiency and data processing:

#### Definition of the Pack Strategy
The Pack strategy (also known as Sequence Packing) refers to concatenating multiple training samples of varying lengths into fixed-length sequences for training, rather than applying padding to short sequences.

#### Core Advantages
- **Eliminating Padding Waste**: In traditional approaches, samples shorter than the maximum sequence length need to be filled with padding tokens. These padding positions carry no useful information yet consume computational resources. The Pack strategy completely eliminates padding through concatenation, devoting all GPU compute power to actual token processing.
- **Improving Hardware Utilization**: When using efficient attention implementations like Flash Attention, the Pack strategy ensures that the token count per batch is fixed and full, significantly boosting training throughput.
- **Significant Training Efficiency Gain**: In practice, the Pack strategy can increase training speed by 20%--50%, depending on the ratio of the average sequence length to the maximum sequence length in the raw dataset.

#### Potential Issues
- **Cross-document Attention Contamination**: Without special handling, in the concatenated sequence, tokens in a subsequent document can attend to content in a preceding document, causing semantic contamination and affecting the model's correct modeling of document boundaries.
- **Solution**: Introduce an in-document Attention Mask (Pack Mask) to ensure each token can only attend to tokens within the same document, completely isolating cross-document information. However, this introduces additional mask calculation overhead and is not easily handled efficiently in certain attention implementations.
- **Position Encoding Handling**: After concatenation, position encodings must be reset independently for each document (position IDs restart from 0); otherwise, the model will receive incorrect positional information.

</details>

**26. How to Evaluate the Capabilities of a Base Model After Pre-training? What Are the Limitations of Commonly Used Benchmarks?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspective of model evaluation systems:

#### Common Evaluation Methods
- **Perplexity**: Calculating the language model's perplexity on a held-out test set measures the quality of natural language modeling. A lower value indicates a better fit to the language distribution.
- **Standardized Benchmarks**: Performing zero-shot or few-shot evaluations across multiple public benchmarks under unified evaluation frameworks (such as lm-evaluation-harness).
- **Generation Quality Evaluation**: Scoring the fluency, accuracy, and usefulness of model-generated content via human evaluation or using GPT-4 as a Judge.

#### Commonly Used Benchmarks
- **MMLU**: Multiple-choice questions covering 57 subjects, testing the model's broad knowledge base.
- **HellaSwag**: Commonsense reasoning and sentence completion tasks.
- **ARC (Challenge/Easy)**: Science question answering, testing reasoning capability.
- **GSM8K / MATH**: Mathematical reasoning capability evaluation.
- **HumanEval / MBPP**: Code generation capability evaluation.

#### Core Limitations of Benchmarks
- **Data Contamination**: Pre-training data may contain test questions or answers from benchmarks, causing the model to essentially "memorize answers" during evaluation, resulting in inflated scores that fail to reflect true generalization capability.
- **Saturation Issue**: As model capabilities improve, many benchmarks (such as HellaSwag) approach human-level performance or are surpassed, leading to decreasing discriminative power.
- **Static and Narrow Scope**: Most benchmarks are based on fixed question banks with limited capability dimensions covered, making it hard to comprehensively reflect actual model performance in open-ended scenarios.
- **Disconnect Between Evaluation and Application**: A high benchmark score does not equate to a good real-world application experience; the complexity of real-world user scenarios far exceeds standardized testing.
- **Format Sensitivity**: Asking the same question with different prompt formats can lead to significantly different model scores, demonstrating that benchmark scoring is sensitive to prompt engineering and has questionable stability.

</details>

**27. What Is the Token Dropping Strategy in Pre-training? What Impacts Does It Have on Model Quality?**

<details>
<summary>💡 Show Answer</summary>

Analyzed from the perspective of computational efficiency optimization:

#### Definition of Token Dropping
Token Dropping is a training acceleration strategy whose core idea is to selectively drop certain "unimportant" tokens in intermediate layers during forward propagation so that they do not participate in subsequent layer computations, thereby reducing computational workload. The dropped tokens are typically re-inserted before the final layer (via a recovery mechanism) to ensure the completeness of the output sequence.

#### Representative Works
EfficientViT proposed by Google and related research in the LLM domain (such as TR-BERT and CoLT5) have explored various forms of token dropping mechanisms. Their core criteria for judgment are typically based on the magnitude of attention scores: tokens with low attention weights are considered to contribute little to the current context and are prioritized for dropping.

#### Impacts on Model Quality
- **Positive Impacts**: Under a moderate dropping rate (e.g., 10%--20%), the loss in model performance is minimal, while training speed can be increased by over 20%, making it a relatively practical acceleration approach.
- **Negative Impacts**:
    - When the dropping rate is too high, dropped tokens may carry important semantic information, leading to performance degradation on tasks requiring fine-grained semantic understanding.
    - The dropping strategy is based on intermediate-layer attention scores, which are unstable during early training stages, potentially leading to inaccurate dropping decisions in early training.
    - Learning of long-tail knowledge and low-frequency tokens may be impacted more severely because these tokens already appear infrequently, and additional dropping further reduces learning opportunities.

</details>

**28. What is the Essential Difference Between Absolute and Relative Position Embeddings in Long-Text Generalization?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of positional information modeling:

#### Absolute Position Embedding
- **Principle**: Assigns a fixed position vector to each position in the sequence, which is directly added to the token embedding. This includes the sinusoidal encoding from the original Transformer paper and the learnable position embeddings used in the BERT/GPT series.
- **Fundamental Flaw in Long-Text Generalization**: Absolute position embedding learns a representation for each absolute position. When the sequence length during inference exceeds the maximum length during training, the model encounters position indices it has never seen before, rendering it unable to extrapolate and causing a sharp decline in performance. Even though sinusoidal encoding can theoretically extrapolate, in practice performance still degrades significantly beyond the training length.

#### Relative Position Embedding
- **Principle**: Rather than encoding the absolute positions of tokens, it encodes the relative distance between tokens, introducing a relative position bias into the attention calculation. Representative methods include T5's relative bias, RoPE (Rotary Position Embedding), and ALiBi.
- **Advantages in Long-Text Generalization**:
    - **RoPE**: Encodes relative position information into the inner product of queries and keys using rotation matrices, theoretically allowing extrapolation to arbitrary lengths. However, in practice, performance still degrades beyond the training length, requiring interpolation methods like YaRN or LongRoPE for length extension.
    - **ALiBi**: Adds a negative bias linearly proportional to the relative distance on top of the attention scores, imposing larger penalties for longer distances. This design naturally inclines the model to attend to nearby tokens, giving it strong extrapolation capabilities beyond the training length.
- **Essential Advantage**: Relative position embeddings model the relative relationships between tokens rather than absolute positions. Positional relationships learned on short sequences can be more naturally transferred to long sequences, resulting in better length generalization.

</details>

**29. How Can Harmful Content in Pre-training Data Be Filtered and Mitigated During the Training Phase?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of data governance and model safety:

#### Main Types of Harmful Content
Common harmful content in pre-training data (especially web-crawled data) includes: hate speech, discriminatory content, sexually explicit content, violent content, misinformation, and personally identifiable information (PII, such as ID numbers and phone numbers).

#### Filtering Methods

**1. Rule-Based Filtering**
- Use keyword blacklists to filter documents containing sensitive terms.
- Filter personally identifiable information (PII) such as email addresses, phone numbers, and ID numbers based on regular expressions.
- Set text quality thresholds to filter out low-quality documents (e.g., garbled text, pure advertisement).

**2. Classifier-Based Filtering**
- Train dedicated toxicity classifiers (e.g., fastText- or BERT-based classifiers) to score documents and filter high-toxicity content.
- Use content safety APIs (such as the Perspective API) to score and filter batch data for toxicity.

**3. Data Source Whitelisting Strategy**
- Prioritize human-curated, high-quality data sources (e.g., Wikipedia, academic papers, books) to reduce reliance on raw web crawls.
- Assign different trust levels and sampling weights to data from different sources.

#### Limitations and Residual Risks
Filtering methods cannot achieve complete removal: keyword filtering can misclassify legitimate content (e.g., text related to historical research); classifiers themselves have false positives and false negatives; and implicit biases (e.g., gender or racial biases) are difficult to detect via rules alone. Therefore, pre-training data filtering is typically combined with downstream RLHF/safety fine-tuning to form multi-layered defenses.

</details>

**30. What Are the Advantages and Risks of Continual Pre-training Compared to Pre-training from Scratch?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of model reuse and knowledge retention:

#### Definition of Continual Pre-training
Continual pre-training refers to continuing the pre-training process on top of an already pre-trained base model using new data (such as domain-specific data, new language data, or updated corpora), rather than starting from scratch with random initialization.

#### Core Advantages
- **Significant Savings in Computational Resources**: There is no need to learn general language capabilities from scratch; the model can rapidly adapt to new domains on top of existing knowledge, incurring far lower training costs than pre-training from scratch.
- **Knowledge Transfer**: The general language understanding and reasoning abilities already possessed by the base model can be transferred to the new domain, providing strong baseline performance even when domain-specific data is limited.
- **Rapid Domain Adaptation**: For vertical domains such as healthcare, law, and finance, continual pre-training can swiftly boost domain expertise with a relatively small amount of domain data.

#### Major Risks
- **Catastrophic Forgetting**: This is the primary risk of continual pre-training. When there is a large distribution shift between the new data and the original pre-training data, the model rapidly forgets its original general capabilities while learning new knowledge, leading to a significant performance drop on original tasks.
- **Sensitivity to Learning Rate Settings**: Continual pre-training typically requires a learning rate much smaller than that used for pre-training from scratch; a learning rate that is too large will destroy the existing weight structure, whereas one that is too small results in insufficient learning.
- **Data Ratio Issues**: If a certain proportion of the original general data is not mixed into the new data (i.e., the replay mechanism), catastrophic forgetting will become even more severe.

#### Common Strategies to Mitigate Forgetting
- **Data Replay (Data Mixing)**: Mix a certain ratio of the original pre-training data into the new data to maintain memory of the original distribution.
- **Smaller Learning Rates**: Use warmup + cosine decay scheduling, where the peak learning rate is typically set to 1/5 to 1/10 of that used for training from scratch.

</details>

**31. How Do Model Depth and Width Differently Impact Pre-training Performance, and How to Trade Off Between Them?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of model architecture design:

#### Impact of Depth
- **Hierarchical Feature Extraction**: Deeper models can learn higher-level abstract features, with lower layers capturing syntax and local semantics, and higher layers capturing reasoning and global semantics.
- **Enhanced Expressive Power**: Theoretically, increasing depth enables the model to express more complex functions.
- **Increased Training Difficulty**: With too many layers, gradients must propagate through longer paths during backpropagation. Even with residual connections, training instability (e.g., gradient vanishing/explosion, loss spikes) is more likely to occur.
- **Increased Inference Latency**: Increasing depth means more layers executed sequentially, causing inference latency to grow linearly.

#### Impact of Width
- **Increased Representation Capacity**: Wider models (larger hidden dimension $d_{model}$) can store more information at each layer, which benefits knowledge-intensive tasks.
- **Parallel-Friendly Computation**: Increasing width mainly expands matrix multiplication dimensions, which can be efficiently parallelized via Tensor Parallelism, leading to relatively small increases in inference latency.
- **Parameter Efficiency**: For a fixed parameter count, increasing width generally has lower parameter efficiency than increasing depth, as quadratic scaling of matrix dimensions with width may introduce redundancy.

#### Trade-off Principles
- According to Scaling Law studies, given a fixed compute budget, depth and width should scale jointly, typically following empirical depth-to-width ratios (e.g., GPT-3's 96 layers and 12,288 dimensions, LLaMA's 32 layers and 4,096 dimensions).
- Latency-sensitive inference scenarios (e.g., online serving) tend to decrease depth and increase width to leverage Tensor Parallelism for latency reduction.
- When prioritizing training stability, depth should be appropriately controlled to avoid the training difficulties associated with extremely deep networks.

</details>

**32. What Is the Theoretical Basis for Batch Size Scheduling Strategies in Pre-training?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of optimization dynamics:

#### Common Strategy: Small-to-Large Batch Size Scheduling
Large language model pre-training often adopts a schedule that gradually increases batch size: using a smaller batch size during initial training and progressively scaling up to the target batch size as training proceeds.

#### Theoretical Basis

**1. Initial Training Requires "Noisy" Gradients**

Early in training, model parameters are far from the optimal solution and the loss landscape is rough. At this stage, gradient noise introduced by smaller batch sizes helps the model escape local minima and saddle points, explore a broader parameter space, and serve as an implicit regularizer.

**2. Late Training Requires Accurate Gradient Directions**

Later in training, the model approaches the convergence region and the loss landscape becomes smoother. At this stage, more accurate gradient estimations are needed to stabilize convergence. Larger batch sizes reduce gradient variance and make parameter update directions more precise, aiding fine adjustment.

**3. Coupling with Learning Rate**

According to the Linear Scaling Rule, when the batch size doubles, the learning rate should also double accordingly to maintain an effective step size for parameter updates. Therefore, batch size scheduling is typically co-designed with learning rate scheduling.

**4. Computational Efficiency Considerations**

Smaller batch sizes enable faster parameter updates early in training (less compute per step), rapidly moving to better regions in the parameter space; larger batch sizes boost hardware utilization later in training, accelerating training throughput.

</details>

**33. Does Training Data Order Affect Pre-training Results? What Are the Relevant Experimental Findings?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of curriculum learning and data scheduling:

#### Baseline Practice of Random Shuffling
Most large model pre-training approaches employ thorough random shuffling of training data to eliminate biases introduced by data ordering, ensure that the data distribution of each batch is as uniform as possible, and reduce distribution shift during training.

#### Experimental Findings on the Impact of Data Order

**1. Positive Effects of Curriculum Learning**

Inspired by human learning processes, curriculum learning organizes training data from "easy to hard". Some studies (e.g., experiments on mathematical reasoning and code tasks) show that training on simple samples first before introducing complex ones can accelerate convergence speed and yield slight performance gains on certain tasks.

**2. Domain Curriculum Scheduling**

In multi-stage pre-training, pre-training on general data first and then progressively introducing high-quality domain data (e.g., code, mathematics) has been proven superior to mixing all data from the beginning. Models such as LLaMA 3 increase the proportion of high-quality data in late-stage training to reinforce specific capabilities.

**3. Findings Showing Limited Impact of Data Order**

Other studies suggest that for sufficiently large models and dataset sizes, the impact of data order on final performance is relatively limited, and thorough random shuffling is sufficient. The effects of data quality and mixture ratio far outweigh those of data ordering.

#### Overall Conclusion
Data order does affect pre-training results, but the magnitude of its impact is small compared to data quality, mixture ratio, and scale. Curriculum learning offers practical value in specific tasks (math, code) and late-stage data scheduling, but it is not a universally applicable silver bullet.

</details>

