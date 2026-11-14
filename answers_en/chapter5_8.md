# 5.8 RL Training Practice and Engineering — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 5: Reinforcement Learning and Post-Training Alignment (RLHF / DPO) · 40 questions

**1. What Hyperparameters Need to Be Set During RL Training (e.g., PPO/GRPO), and What Are Their Typical Values?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for What Hyperparameters Need to Be Set During RL Training (e.g., PPO/GRPO), and What Are Their Typical Values?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig41_what_hyperparameters_need_to_b.png)




Common hyperparameters and typical value ranges in PPO/GRPO training (Note: exact optimal values depend heavily on model scale, task, and implementation; the following are common empirical ranges for reference only):

**1. Learning Rate**

- Policy model: Typically on the order of **1e-6 to 1e-5** (significantly smaller than the learning rate in the SFT stage, as updates during the RL stage require greater "caution");
- Critic (PPO): Can be close to or slightly larger than the policy model learning rate.

**2. KL Penalty Coefficient $\beta$**

- Typical range is **0.01 to 0.1** (depending on the calculation method and magnitude of KL in the specific implementation). Some implementations feature **dynamically adjusted $\beta$** (e.g., adaptively increasing/decreasing $\beta$ based on whether the current KL divergence exceeds a target value).

**3. Clip Range $\varepsilon$ (PPO Clip Range)**

- Standard symmetric clipping usually uses **$\varepsilon = 0.2$** (i.e., clipping range $[0.8, 1.2]$);
- Improvements such as DAPO use asymmetric clipping, where $\varepsilon_{low}$ may remain close to 0.2 while $\varepsilon_{high}$ can be larger (e.g., 0.28 or higher).

**4. GAE Parameters: Discount Factor $\gamma$ and $\lambda$**

- $\gamma$ (discount factor): In LLM-RL, since sequences are generally not "infinitely long" and many scenarios only provide a single reward at the end of the sequence, $\gamma$ is often set **close to 1 (such as 1.0 or 0.99)**;
- $\lambda$ (GAE smoothing factor): Common values range from **0.9 to 0.99**.

**5. Entropy Regularization Coefficient**

- Usually a small value (e.g., **0.0 to 0.01**), which needs to be adjusted based on whether entropy collapse is observed.

**6. Group Size $G$ in GRPO (Number of Sampled Responses per Prompt)**

- Common values range from **4 to 64** (varying significantly across implementations depending on compute resources; a larger $G$ leads to more stable estimation of intra-group statistics, but increases rollout computation overhead).

**7. Number of Update Epochs per Rollout (PPO Epochs)**

- Typically **1 to 4** (performing multi-epoch mini-batch updates on the same batch of rollout data); too many epochs increases the "degree of off-policyness", which may compromise the effectiveness of the clipping mechanism.

**8. Batch Size / Mini-batch Size**

- Depends on hardware resources; generally, a larger batch size is desirable to reduce the variance of gradient estimates, but it is constrained by GPU memory (especially during the rollout phase).

**9. Maximum Response Length**

- Set according to task requirements (e.g., mathematical reasoning tasks may require longer Chain-of-Thought (CoT), set to thousands of tokens).

**10. Value Function Loss Coefficient $c_1$ and Entropy Reward Coefficient $c_2$ (Weights in Total PPO Loss, see Question 414)**

- $c_1$ typical values are around **0.5 to 1**; $c_2$ is usually small (e.g., 0.0 to 0.01).

**Summary**: The core hyperparameters involved in PPO/GRPO include **learning rate (on the order of 1e-6 to 1e-5), KL coefficient $\beta$ (0.01 to 0.1), clip range $\varepsilon$ (approx. 0.2, potentially asymmetric), GAE parameters $\gamma$ and $\lambda$ ($\gamma$ close to 1, $\lambda$ around 0.9 to 0.99), entropy regularization coefficient (small), group size $G$ in GRPO (4 to 64), update epochs per rollout (1 to 4), and maximum response length**, among others. The exact values of these hyperparameters are highly dependent on the specific task and implementation. Furthermore, many hyperparameters (such as $\beta$ and clip range) employ dynamic adjustment or asymmetric designs in advanced algorithm variants. In practice, fine-grained tuning based on training curves (trends in reward, KL divergence, and entropy) is usually required.

</details>

**2. Does Reinforcement Learning Increase Training Costs and GPU Requirements?**

<details>
<summary>💡 Show Answer</summary>

Yes, compared to SFT, reinforcement learning significantly increases training costs and GPU requirements, which represents one of the main engineering challenges for its large-scale application.

**Direct Reasons for Increased Costs**:
- **Overhead of online rollout**: RL training (especially PPO/GRPO) requires continuous LLM inference (rollout) to generate training data. Autoregressive generation in LLMs is a purely sequential operation, resulting in GPU utilization far lower than parallel matrix operations during training. Rollout typically accounts for 60--80% of the entire training loop time;
- **Multi-model concurrency**: PPO requires maintaining 4 models simultaneously (Actor, Critic, RM, Ref), while GRPO also requires 2--3 models, making memory requirements 2--4 times that of SFT;
- **Multiple sampling in GRPO**: Each prompt requires sampling $G$ responses ($G=8--64$), equivalent to $G$ times the rollout computational load;
- **Multi-stage training**: The complete RLHF pipeline consists of three stages: SFT, RM training, and PPO, requiring separate management of different datasets and model checkpoints.

**Specific Cost Comparison (Approximate)**:
- SFT: Requires only 1 model, batch training, high GPU utilization (70--90%);
- PPO: 4 models + online rollout, GPU utilization is low due to rollout (30--50%);
- GRPO: 2--3 models + $G$-fold rollout, overall cost is roughly 5--15 times that of SFT;
- DPO: 2 models, offline training, cost is roughly 2 times that of SFT, making it the lowest-cost alignment scheme.

**Engineering optimization directions**: Using efficient inference engines (e.g., vLLM/SGLang) for rollout, asynchronous training frameworks (e.g., AReaL), CPU offloading, and LoRA to reduce trainable parameters can reduce RL training costs by 30--60%, though it remains significantly higher than SFT.

</details>

**3. Why Are Large Language Models Generally Trained for Only 1–2 Epochs?**

<details>
<summary>💡 Show Answer</summary>

Large-scale pre-trained models are typically trained for only 1--2 epochs (traversing the training data once or twice). There are multiple theoretical and engineering reasons behind this:

**1. Asymmetry Between Data Volume and Compute Cost**

Modern LLM pre-training datasets reach sizes of trillions of tokens (e.g., LLaMA-3 uses 15T tokens), where traversing the dataset just once requires months of GPU cluster compute. Even training for a single epoch provides sufficient compute for the model to thoroughly learn patterns from this massive and diverse data, yielding diminishing returns for multiple epochs.

**2. Overfitting Risk and Generalization Capability**

In massive datasets, each sample typically appears only 1--2 times. If the dataset is traversed multiple times, the model risks "memorizing" specific contents of the training data rather than distilling general rules, leading to degraded generalization capability.

**3. Equivalence of Random Sampling**

When the data volume is extremely large, the data diversity brought by random sampling combined with large batch sizes is statistically approximately equivalent to multiple epochs. Each batch contains virtually no duplicates, naturally achieving a "multi-epoch" effect.

**4. Design of Learning Rate Schedules**

LLM training typically employs a cosine decay learning rate schedule: decaying gradually from the peak to zero, where the complete decay cycle corresponds precisely to 1--2 epochs. Training for multiple epochs requires redesigning the LR schedule, adding hyperparameter tuning complexity.

**5. Complementarity of Post-training Stages**

Pre-training only requires the model to acquire vast general knowledge, while the SFT and RLHF stages perform fine-tuning. Therefore, pre-training does not pursue extreme fitting; 1--2 epochs are sufficient to achieve its goal.

**Fewer Epochs in SFT and RL**: SFT usually takes 1--3 epochs and RL usually 1--2 epochs for similar reasons—overtraining leads to overfitting and catastrophic forgetting.

</details>

**4. How to Guarantee Deterministic Results Across Multiple Inferences for the Same Prompt in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Determinism in LLM inference involves multiple levels and requires simultaneous control over sources of randomness at both algorithmic and engineering levels.

**Sources of Randomness Affecting Result Consistency**:
- **Sampling strategy**: $temperature > 0$ or top-$p$/top-$k$ sampling introduces randomness;
- **Batch padding differences**: Padding at different positions during batch inference affects attention computation;
- **Non-determinism of floating-point operations**: The execution order of parallel floating-point operations on GPUs is non-deterministic, resulting in different accumulated errors;
- **Optimized operators such as FlashAttention**: Certain optimized kernel operators run in non-deterministic modes.

**Specific Methods to Guarantee Consistency**:
1. **Use greedy decoding or set $temperature=0$**: Eliminates sampling randomness by picking the highest-probability token at each step;
2. **Fix random seeds**: Set deterministic random seeds at the PyTorch/CUDA level: `torch.manual_seed(42)`, `torch.cuda.manual_seed_all(42)`;
3. **Enable CUDA deterministic mode**: `torch.backends.cudnn.deterministic = True` + `torch.use_deterministic_algorithms(True)`, forcing the use of deterministic operators (may reduce performance);
4. **Single-sample inference (avoid batch inference)**: In batch inference, padding at different positions causes the effective context length of the same prompt to vary, affecting attention calculation. Single-sample inference completely avoids this issue;
5. **Standardize padding policy**: If batch inference is mandatory, use static padding (pad all sequences to the same maximum length) to ensure the relative position of each prompt within the batch is fixed.

**Note**: During the RL rollout phase, deterministic inference is actually **not the goal**—a high temperature is required to ensure exploration diversity, whereas determinism is pursued only during evaluation or deployment.

</details>

**5. Why Does the Same Prompt Produce Different Results in Batch Inference?**

<details>
<summary>💡 Show Answer</summary>

The primary reasons are the dynamic padding mechanism and the cumulative effect of minute numerical differences in deep Transformer models:
1. Impact of batch processing and dynamic padding: The same prompt may have different amounts of padding in different batches.
2. Numerical stability of Softmax: Even if certain logits are masked out to extremely small values, under the finite precision of floating-point arithmetic, non-zero yet tiny weights may still be produced.
3. Amplification effect of residual connections: After stacking dozens of layers, tiny numerical differences can be amplified.
4. Sensitivity of sampling strategies: Minute perturbations can change the selection in greedy decoding.

</details>

**6. What Is the Approximate Initial Loss at the Start of LLM Training?**

<details>
<summary>💡 Show Answer</summary>

The theoretical initial loss value for LLM training can be derived from information theory, serving as an important "health check" metric in practice.

**Theoretical Initial Loss**:

Pre-training uses cross-entropy loss. If model parameters are randomly initialized, the prediction for each token is close to a uniform distribution (the probability for each token in the vocabulary $V$ is approximately $1/V$), making the initial loss approximately:
\[\mathcal{L}_init \approx \ln(V)\]

Theoretical initial loss corresponding to common vocabulary sizes:
- $V = 32{,}000$ (LLaMA): $\ln(32000) \approx 10.37$
- $V = 50{,}257$ (GPT-2/3): $\ln(50257) \approx 10.82$
- $V = 100{,}000+$ (some recent models): $\ln(100000) \approx 11.51$

**Practical Loss Trajectory**:
- First few hundred steps: Loss drops rapidly from approx. 10.5--11.0 down to 3--5;
- Mid-training phase: Loss stabilizes around 2--3 (depending on data quality and model scale);
- At convergence: Approx. 1.5--2.5 on high-quality data, and as low as 1.0 on certain specialized tasks.

**Implications of Abnormal Loss**:
- **Initial loss far higher than $\ln(V)$**: Usually indicates improper parameter initialization (e.g., weights being too large, causing softmax saturation);
- **Initial loss significantly lower than $\ln(V)$**: The model may have loaded pre-trained weights from a checkpoint rather than starting from scratch;
- **Initial loss in SFT/RLHF stage**: Because the model is already pre-trained, the initial SFT loss is typically between 1.5 and 2.5, rather than close to $\ln(V)$;
- **Loss fails to decrease or turns into NaN/Inf**: Usually caused by an overly large learning rate, gradient explosion, abnormal tokens in the data, etc.

</details>

**7. Have You Encountered the Reward Sparsity Problem? How Can It Be Solved?**

<details>
<summary>💡 Show Answer</summary>

**Typical Scenarios of the Reward Sparsity Problem**:

As described in Questions 400 and 408, the most common form of reward sparsity in LLM-RL is: **a scalar reward is obtained only upon generating the complete sequence (end of response), with no direct reward feedback at intermediate steps within the sequence**. In more complex Agent tasks (such as multi-step tool calls or long-horizon planning), sparsity is further reflected as: **a reward is provided only when the entire task is completed (success/failure), without feedback at intermediate steps**—this "long-horizon sparse reward" presents an especially severe challenge to RL training (severe signal delay makes credit assignment of the "final success/failure" to specific intermediate steps/decisions difficult).

**Solutions**:

**1. Value Function and GAE (for intra-sequence sparsity, see Questions 400 and 413)**

- By training a Critic to estimate the value of intermediate states, TD/GAE is used to "propagate" the scalar reward at the end of the sequence to each position within the sequence, providing an effective advantage estimate for every step—this is the standard approach in PPO to tackle "sequence-level sparse rewards".

**2. Process Reward Model (PRM)**

- For multi-step reasoning/Agent tasks, train a **process reward model specifically designed to evaluate whether intermediate steps are reasonable/correct**, providing independent reward signals for each intermediate step rather than assigning rewards only at the final outcome (see Question 444 for a detailed comparison of process rewards vs. outcome rewards);
- This requires additional annotations (correctness annotations for intermediate steps) or automated methods (such as Monte Carlo sampling to estimate the probability of each intermediate step leading to final success, serving as an implicit process reward for that step).

**3. Reward Shaping**

- On top of the final sparse reward, **manually design auxiliary intermediate rewards** to guide the model toward exploring in the right direction. For example:
- In code generation tasks, in addition to "whether final unit tests pass", a small positive reward can be given for "whether the code compiles/runs successfully (even if tests fail)", guiding the model to first produce executable code;
- In long-text generation tasks, small rewards can be given for intermediate dimensions such as "whether formatting requirements are met" or "whether key information points are included";
- Note: Reward shaping requires careful design to avoid introducing "shortcuts" inconsistent with the final objective (i.e., shaping itself can become a new source of reward hacking; see Question 438 for details).

**4. Curriculum Learning**

- Start training from simpler tasks/sub-tasks where rewards are easier to obtain, then gradually transition to more complex, sparser-reward complete tasks. This allows the model to build foundational capabilities in simpler stages with dense feedback before transferring to complex sparse-reward tasks.

**5. Exploration Enhancement**

- Increase entropy regularization coefficients, or adopt more diverse sampling strategies (e.g., higher temperature, larger group size $G$), thereby raising the probability of accidentally discovering high-reward trajectories under sparse reward environments and providing more valid signals for training.

**6. Hindsight Methods (e.g., Ideas from Hindsight Experience Replay)**

- In certain goal-oriented tasks, even if the originally set goal is not achieved, the "actually reached state" can be relabeled as a "virtual goal" to construct additional non-zero reward training samples, increasing the amount of effective training signals (a common technique in traditional RL, whose application in LLM scenarios is relatively recent).

**Summary**: The reward sparsity problem is widespread in LLM-RL (sequence-level and task-level). Common solutions include: "**using value functions + GAE to propagate sparse signals within sequences; introducing process reward models to provide independent feedback for intermediate steps; carefully designing reward shaping for auxiliary guidance signals; employing curriculum learning to transition from simple to complex tasks; and enhancing exploration (entropy regularization, larger group sampling) to increase the likelihood of discovering high-reward trajectories**". Depending on the granularity—intra-sequence sparsity vs. task-level (multi-step) sparsity—the emphasized techniques differ (the former relies more on value functions/GAE, while the latter leans more on process rewards, reward shaping, and curriculum learning).

</details>

**8. How to Handle Reward Sparsity During LLM RL Training?**

<details>
<summary>💡 Show Answer</summary>

**Theoretical Methods**:
1. **Reward Shaping**: Expanding originally sparse rewards into denser reward signals.
2. **Hindsight Experience Replay (HER)**: Reusing experience by taking observed actual outcomes as new "goals" during the experience replay phase.
3. **Confidence-controlled Exploration (CCE)**: Achieving higher sample efficiency in sparse reward scenarios by dynamically adjusting exploration strategies.
4. **Hybrid Reward Framework**: Structurally combining sparse binary verifier signals with continuous scores from a reward model.

**Engineering Practice**:
1. Deliberately avoid using pure binary success rewards as the sole supervisory signal.
2. Treat intra-group discriminability as a minimum requirement for reward design.
3. Adopt staged or curriculum-based reward design.
4. Perform rigorous offline sanity checks prior to the reward pipeline.
5. Apply engineering constraints to enforce reward numerical stability.

</details>

**9. What Are Your Views on Process Reward vs. Outcome Reward? Please Compare Them.**

<details>
<summary>💡 Show Answer</summary>

![Credit Assignment: PRM vs. ORM](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig42_what_are_your_views_on_process.png)




**Outcome Reward**:

- Definition: Rewards are awarded solely based on the **final outcome of the task** (e.g., whether the final answer is correct or whether the task is successfully completed), without considering the intermediate process.

Advantages:
- **Objective, easy to obtain, and hard to hack**: For tasks with explicit standards of correctness (e.g., math problem answers, code test cases), outcome correctness can usually be **automated and objectively verified** without additional annotation costs, and such "verifiable" rewards are difficult for the model to bypass through "speculation/hacking" (refer to Question 438);
- **Does not restrict the model from exploring "unconventional yet effective" problem-solving paths**: As long as the final outcome is correct, the model receives a reward regardless of what intermediate (possibly non-standard) ideas or steps it takes, affording the model greater space for autonomous exploration.

Disadvantages:
- **Sparse rewards**: For complex multi-step tasks, rewards are given only at the final step, making it impossible to directly evaluate the correctness of intermediate steps (refer to Question 443);
- **Difficult error attribution**: If the final result is incorrect but some intermediate steps were correct, the outcome reward cannot distinguish "which intermediate steps caused the issue." All intermediate steps receive negative feedback "together," which may lead the model to "mistakenly" suppress the probabilities of intermediate steps that were actually reasonable (inaccurate signal attribution);
- **May bestow undeserved rewards due to a "flawed intermediate process coincidentally yielding the correct final result"** (e.g., calculation errors that cancel each other out to produce the correct final answer).

**Process Reward**:

- Definition: Evaluates and rewards **each intermediate step in the reasoning/problem-solving process** individually (e.g., whether the reasoning in that step is correct or reasonable).

Advantages:
- **Denser reward signals, alleviating the sparsity problem**;
- **More precise error attribution**: Enables accurate localization of "which step caused the error," providing negative feedback for that specific step without unfairly penalizing prior correct steps;
- **Helps train models to produce more standardized and interpretable reasoning processes** (focusing not only on the final answer, but also on the quality of the reasoning chain itself).

Disadvantages:
- **High acquisition/training cost**: Requires annotating the correctness of intermediate steps (high human annotation cost) or training a dedicated Process Reward Model (PRM)—which itself requires step-level annotated data, leading to a chicken-and-egg problem;
- **May restrict exploratory flexibility / introduce "standard answer bias"**: If the PRM is trained on certain "standard solution steps," it may **penalize methods whose steps differ from the standard but are essentially equally reasonable (or even superior)**, thereby restricting the model from exploring diverse and creative reasoning paths;
- **Easier to be hacked**: The PRM is itself a neural network (which may lack robustness). The policy model might learn to "format intermediate step descriptions to match the PRM's structural preferences" rather than "truly improving the correctness of intermediate reasoning" (i.e., PRM faces risks similar to LLM-as-Judge discussed in Question 439);
- **The definition of a step can be ambiguous**: What constitutes "a single step" may lack a unified standard across different tasks or reasoning styles, posing difficulties for annotation and PRM training.

**Comparison and Practical Recommendations**:

- For tasks with **clear and automatically verifiable final outcomes** (e.g., mathematics, coding), **outcome rewards are the preferred and more reliable choice** (practical work such as DeepSeek-R1 demonstrates that pure outcome rewards coupled with large-scale RL training enable models to spontaneously learn effective or even complex reasoning processes—meaning that "process" can emerge to some extent from "outcome-driven" optimization without strictly requiring explicit process supervision);
- Process rewards are better suited as **auxiliary/supplementary signals**, especially in scenarios where outcome rewards are excessively sparse (very long task chains with severely delayed outcome feedback) or where **outcome verification itself is difficult/expensive** (such as open-ended writing or complex multi-step Agent tasks where determining "final success" via simple rules is hard);
- In practical applications, the two are often **combined**—with outcome rewards as the primary driver (ensuring the optimization direction aligns with the final objective and remains hard to hack), and process rewards as an auxiliary signal (providing denser intermediate guidance to alleviate sparsity), using smaller weights to prevent process rewards from dominating the optimization direction or imposing unnecessary exploration constraints.

**Summary**: Outcome rewards are **objective and hard to hack, but sparse and imprecise in attribution**; process rewards are **dense and precise in attribution, but costly to obtain, prone to restricting exploration, and susceptible to hacking**. Each has its strengths and weaknesses. In practice, the tendency is to use **outcome rewards as the core driving force (especially for tasks with automatically verifiable outcomes), with process rewards serving as auxiliary dense guidance signals**. The specific weights and combination strategies should be balanced based on task characteristics (whether outcomes are easy to verify, task chain length, etc.).

</details>

**10. Which Scenarios are PRM and ORM Respectively Suitable For?**

<details>
<summary>💡 Show Answer</summary>

**ORM (Outcome Reward Model)**:
- Scores only the correctness of the final answer.
- Suitable for: Tasks where the final answer is unique and easy to verify; scenarios with limited training data/annotation budgets; serving as the reward signal in large-scale RL training.
- Limitations: Cannot distinguish between "correct reasoning but final calculation error" and "flawed reasoning logic that coincidentally guessed the correct answer."

**PRM (Process Reward Model)**:
- Scores each intermediate step in the reasoning process individually.
- Suitable for: Scenarios requiring precise localization of the specific step where an error occurred; guiding search/sampling during complex multi-step reasoning; scenarios demanding higher interpretability of reasoning quality.
- Limitations: High annotation cost; vulnerable to exploitation/hacking; requires continuous updating to adapt to policy distribution shifts.

Summary: ORM is better suited for scenarios requiring objective, low-cost reward signals for large-scale RL; PRM is better suited for scenarios requiring fine-grained process supervision.

</details>

**11. How to Solve the Credit Assignment Problem? What is the Difference Between Token-level and Sequence-level Rewards?**

<details>
<summary>💡 Show Answer</summary>

![Token-level vs. Sequence-level Credit Assignment](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig43_how_to_solve_the_credit_assign.png)




The Credit Assignment Problem is a classic dilemma in reinforcement learning. In the context of language model generation, it refers to: when a complete response (sequence) receives a final reward score, how do we determine which specific tokens in the sequence should be credited with (or blamed for) this score? A good ending might compensate for a poor beginning, and vice versa. Simply assigning the final reward equally to every token is unfair and inefficient.

Token-level Reward vs. Sequence-level Reward

1. Sequence-level Reward:

- Definition: This is the most common form in RLHF. The reward model (RM) reads the entire generated sequence and outputs a single scalar score as the evaluation for the entire sequence.
- Advantages:
- Consistent with human evaluation patterns: Humans typically form an overall impression after reading the entire response, making this approach easier for collecting preference data and training the RM.
- Simple implementation: The design and computation of the reward function are straightforward.
- Disadvantages:
- Ambiguous credit assignment: This directly reflects the credit assignment problem. All tokens in the sequence receive the same reward signal, making it impossible to distinguish "good words" from "bad words," resulting in sparse and noisy learning signals that degrade learning efficiency.

1. Token-level Reward:

- Definition: Assigns an independent reward score to every token in the sequence. This score should reflect the token's contribution given the context at that moment.
- Advantages:
- Fine-grained signal: Provides a highly detailed and dense learning signal, which theoretically can greatly boost learning efficiency and final performance because it directly informs the model which step was right and which was wrong.
- Disadvantages:
- Difficult to obtain: Having annotators score every token is nearly impossible due to extreme cognitive load. Therefore, token-level rewards are rarely collected directly from humans.
- Difficult to define: Defining the "goodness" of an individual token is inherently complex. The value of a token depends heavily on the subsequent context generated later.

How to Solve (or Alleviate) the Credit Assignment Problem?

Although we typically receive only sequence-level rewards, mainstream RL algorithms (such as PPO) employ internal mechanisms to mitigate the credit assignment problem:

1. Advantage Function and Value Function:

- Method: In PPO, in addition to the policy model (Actor), a value model (Critic) is also trained. The role of this Critic is to estimate the expected future reward under a given state (i.e., the context of the partially generated sequence).
- Credit Assignment: By calculating the Advantage function, i.e., `A(s, a) = R_t - V(s_t)` (in simplified form), we can estimate how much better taking action $a_t$ (generating a specific token) at current state $s_t$ is compared to the "average expectation." $R_t$ is the actual future return obtained, and $V(s_t)$ is the expected average return. This advantage value can be viewed as a pseudo token-level reward signal.
- GAE (Generalized Advantage Estimation): PPO typically uses GAE to estimate the advantage function more stably. GAE combines TD errors across multiple time steps via exponential weighted averaging, further balancing bias and variance to provide a more reliable credit assignment signal for each time step.

In summary, although we only have a single final sequence reward, by introducing a Critic that learns future expectations, PPO can generate a reasonable, indirect "advantage" signal reflecting each token step's marginal contribution, thereby effectively addressing the credit assignment problem in practice.

</details>

**12. Credit Assignment Problem: Token-level vs. Sequence-level Rewards**

<details>
<summary>💡 Show Answer</summary>

**Credit Assignment Problem**: In a response, the final reward is awarded for the **entire sequence**, yet the policy must update the **generation probability of each token**—how to appropriately distribute this overall "good/bad" signal onto every individual token in the sequence.

**Sequence-level Reward**: Gives a scalar reward only at the end of the sequence. Advantage: simple to implement. Disadvantage: coarse credit assignment; longer propagation paths introduce higher noise and variance.

**Token-level Reward**: Each token (or each reasoning step) receives an immediate reward signal. Advantage: finer-grained credit assignment. Disadvantage: high cost of obtaining high-quality annotations.

**Practical Trade-offs**: A common approach combines a sequence-level final reward with a KL penalty (where the KL penalty is naturally token-level); GAE decomposes the sequence-level reward into step-by-step token advantages via the value function. For reasoning tasks, PRMs provide finer-grained process rewards that can be combined with ORMs.

</details>

**13. Scenario Question: Given a Practical Scenario, How Should the Reward Design for RL Training Be Structured?**

<details>
<summary>💡 Show Answer</summary>

Taking "training an Agent model for automated code bug fixing" as an example, we illustrate the design philosophy for rewards:

**Scenario Description**: Given a codebase containing bugs along with associated error descriptions / failing test cases, the model needs to generate a code modification (patch). The objective is to make the codebase pass the originally failing tests while not breaking any existing functional capabilities.

**Reward Design**:

**1. Primary Outcome Reward—Based on Verifiable Objective Signals**

- **Test passage status** (core signal):
- Apply the patch generated by the model to the codebase and run the test suite;
- If **originally failing tests now pass, and all other previously passing tests still pass**: Grant a higher positive reward (e.g., +1);
- If **originally failing tests still fail**: Grant a negative reward or 0 (task uncompleted);
- If **the target test is fixed, but causes other existing tests to fail (introducing new bugs)**: Grant a reward **between "complete failure" and "complete success," or even a negative reward** (because "breaking existing functionality" is a worse outcome than "unfixed," which needs to be explicitly reflected in the reward design as an asymmetric cost, similar to the asymmetric error costs discussed in Question 358).

**2. Process/Intermediate Reward (Optional Auxiliary Signal)**

- If the Agent's repair process consists of multiple steps (e.g., "localize problematic code" $\rightarrow$ "analyze root cause" $\rightarrow$ "generate patch"), a minor reward can be given for the **reasonableness of intermediate steps**, for example:
- Whether the Agent correctly localized the code files/functions related to the error (evaluable by comparing against known ground-truth repair files, if such reference data exists);
- Whether the generated patch complies with coding style/syntax guidelines (checkable via static analysis tools, granting minor format-related rewards/penalties).
- The weight of such process rewards should be **far smaller than the final outcome reward**, serving merely as a "shaping" signal to prevent the model from sacrificing the ultimate objective for the sake of "process rewards" (the outcome reward must remain dominant).

**3. Efficiency/Conciseness Reward (Secondary, avoiding redundant edits)**

- Apply a minor penalty to the scope of patch modifications (e.g., lines of code modified; larger edits incur higher penalties) to encourage the model to produce **minimalist, focused fixes** rather than drastically rewriting vast amounts of code (such rewrites, even if passing tests, might introduce un-tested risks and impede code reviews). This weight should be very small to avoid discouraging necessary edits out of fear of penalty.

**4. Security/Compliance Check (Constraint-based, as a hard gate rather than a weighted term)**

- Conduct basic security checks on the generated patch (e.g., checking for obvious vulnerability patterns or malicious code). If issues are detected, **directly assign the lowest possible reward (treated as task failure)** rather than a simple point deduction—such issues must serve as "hard constraints" rather than "soft penalties."

**Overall Combination Form of Reward (Example)**:

R = $R_{test_outcome}$ (Dominant term, based on test passing status; can be tiered: full fix > partial fix > unfixed > introduced new bug)
+ $\alpha \cdot R_{process}$ (Small weight, process reasonableness, optional)
+ $\beta \cdot R_{efficiency}$ (Very small weight, minor regularization for conciseness)
Subject to hard security check gate constraints (if failed, R is set directly to the minimum value)

**Summary**: In this scenario, the core principle of reward design is to **prioritize outcome rewards based on objective, verifiable standards (test passage status) as the dominant signal**, which are resistant to "hacking" (since they rely on deterministic external program execution rather than subjective model scoring); **process and efficiency rewards act as small-weight auxiliary/regularization terms** to guide model behavior without dictating optimization direction; **security compliance issues serve as hard constraints (gates) rather than soft weighted terms**, ensuring that pursuing "test passage" produces no hazardous side effects. This design philosophy of "outcome reward dominance + auxiliary shaping + hard safety constraints" is broadly applicable across many practical RL scenarios.

</details>

**14. What Problems May Arise with Rewards Provided by LLM-as-Judge, and What are the Solutions?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for What Problems May Arise with Rewards Provided by LLM-as-Judge, and What are the Solutions?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig44_what_problems_may_arise_with_r.png)




**Problems that may arise when providing rewards via LLM-as-Judge**:

1. **Position Bias**: In pairwise comparison evaluations (A vs. B), the judge model may systematically prefer options **presented first (or second)**, rather than judging based on the quality of the content itself.

2. **Length Bias**: The judge model may tend to give higher scores to **longer responses**, even when increased length brings no substantial information gain (the superficial impression that "looks more detailed = better").

3. **Style/Formatting Bias**: The judge model may prefer **specific formats** (such as lists, bold text, bullet points), even when these formats are unnecessary for the actual quality of content, causing the evaluated policy model to learn to "pile on formatting" to appease the judge.

4. **Self-Preference Bias**: If the judge model and the evaluated policy model originate from the **same model family or underwent similar training**, the judge may assign higher ratings to outputs whose "style resembles its own," rather than judging based on objective quality.

5. **Limited ability to discern factual errors**: The judge model itself may have knowledge blind spots or hallucinations. For content requiring domain-specific expertise or precise fact-checking, the judge may fail to accurately assess correctness, yielding flawed scores.

6. **Consistency / Reproducibility issues**: For the exact same pair of outputs, the judge model might render inconsistent judgments across different evaluation runs (influenced by temperature parameters, prompt phrasing, etc.), resulting in high noise/variance in scores.

7. **Vulnerability to "adversarial" prompt tricks**: During RL training, the policy model may discover **specific phrasing/verbiage tailored to exploit the judge model** (e.g., adding certain phrases that boost the judge's score without delivering actual value to the user), constituting a specific form of reward hacking.

8. **Opaque / Unstable evaluation criteria**: The judge model's "evaluation criteria" are implicitly encoded within its parameters, making them hard to precisely control or audit. They may drift as the judge model itself is updated, causing "drift" in the training objectives.

**Solutions**:

1. **Addressing Position Bias**: For every pairwise comparison, **randomize/swap the presentation order** for two evaluation runs and average the results (or accept judgments only when both runs agree), eliminating systematic bias caused by position.

2. **Addressing Length Bias**: Explicitly instruct the judge in the prompt that **"higher scores should not be given merely because a response is longer; focus on substantive content quality"**; or apply **post-hoc length normalization** to scores (e.g., computing the correlation between judge scores and response lengths, and debiasing scores if significant correlation exists).

3. **Addressing Self-Preference Bias**: Whenever possible, **use a judge model from a different family or with a different training paradigm than the policy model**, or employ an **ensemble of multiple judge models from diverse sources** (taking a combined score across multiple judges).

4. **Addressing Limited Fact-Checking Ability**: For content requiring factual verification, **combine external tools/retrieval (such as RAG) to assist the judge model in verification**, or for tasks with clear ground-truth answers, prioritize **rule-based verification** over pure LLM judgment (refer to the "rule-based reward" ideas in Question 438).

5. **Addressing Score Consistency Issues**:
- **Multi-sample averaging / Majority voting**: Have the judge model evaluate the same pair multiple times (using different temperatures on the same model or using multiple distinct judge models) and aggregate the results;
- **Use structured evaluation prompts**: Require the judge to evaluate according to explicit dimensions (e.g., scoring accuracy, helpfulness, and safety separately) and clear rubrics rather than a vague "which is better," improving consistency.

6. **Addressing "Adversarial Phrasing" / Reward Hacking**:
- Periodically use **human evaluation** sampling to verify whether LLM-as-Judge scoring remains reliable. Upon detecting signs of the judge being "tricked," **update the judge model's prompt** (explicitly stating not to be influenced by such verbiage) or **re-select/fine-tune the judge model**;
- Combine with general reward hacking mitigation methods such as reward ensembling and KL constraints mentioned in Question 437.

7. **Fine-tuning the Judge Model itself**: The judge model can be specifically **fine-tuned on evaluation tasks (using human-annotated scoring data)** to align its scoring behavior more closely with human annotators' standards and reduce the aforementioned biases.

**Summary**: Serving as a reward source, LLM-as-Judge suffers from a series of issues including **position bias, length bias, style preference, self-preference bias, limited factual discrimination, poor consistency, and vulnerability to adversarial phrasing**, which fundamentally stem from the inherent limitations of "using an imperfect model to evaluate another model." Solutions revolve around multiple directions: "**eliminating identifiable systematic biases (randomizing order, length normalization, structured rubric standards), enhancing judge reliability (multi-source judges, ensembling, domain-specific fine-tuning), integrating objective verification tools (rules/tool assistance), and conducting continuous human verification**." There is no silver bullet; continuous monitoring and iterative refinement are required.

</details>

**15. What Does Reward Hacking Mean?**

<details>
<summary>💡 Show Answer</summary>

Reward Hacking (reward exploitation/gaming) is a core alignment challenge in reinforcement learning, referring to when **an agent discovers and exploits flaws/loopholes in the reward function to maximize its reward score in unexpected ways, without actually achieving the true target goal desired by humans**.

**Causes**:

The reward function (RM) is an imperfect proxy for human preferences and cannot fully capture all critical value dimensions. Any sufficiently powerful optimization process will uncover the "gap" between the reward function and the true goal and exploit it—this is known as Goodhart's Law: "When a measure becomes a target, it ceases to be a good measure."

**Typical Manifestations in LLM RLHF**:
- **Length bias**: If the RM has a slight preference for longer responses, the model will generate increasingly verbose yet content-empty responses;
- **Sycophancy/Flattery**: The model learns to prefix responses with phrases like "That is an excellent question!", as such expressions often earn higher RM scores;
- **Format gaming**: If the RM favors bulleted list formats, the model adopts bullet points for virtually any prompt;
- **Repeating keywords**: Repetitively emphasizing keywords from the user's prompt to boost RM scores through "keyword matching";
- **Excessive safety refusal**: If the RM awards high scores for "safe" responses, the model may refuse to answer any topic even slightly sensitive.

**Detection Signals**: RM scores continuously rise while independent human evaluations indicate declining quality, or the KL divergence increases drastically.

**Core Essence**: The root of Reward Hacking is **misalignment between the training objective (RM score) and the true objective (human satisfaction)**, serving as one of the central problems in AI alignment research.

</details>

**16. Why is RL Training Reward High but Test Reward Low?**

<details>
<summary>💡 Show Answer</summary>

There are two primary potential core issues:
1. **Reward Hacking**: The agent achieves high rewards by "gaming the system," but these behaviors do not genuinely align with the human-intended task objective.
2. **Generalization Problem**: The model overfits the training environment and fails to generalize to novel scenarios.

Diagnostic Method: If test reward increases while human evaluation results decline, it is very likely a reward hacking issue. If test reward fails to increase or even drops, it is typically a generalization issue.

</details>

**17. How to Address the Reward Hacking Phenomenon in RL?**

<details>
<summary>💡 Show Answer</summary>

1. **Through Reward Function Shape Design**: Reduce the space vulnerable to "gaming" by bounding and non-linearly compressing rewards.
2. **Explicitly Modeling Uncertainty in Rewards**: Adopt probabilistic uncertainty reward models (PURM) to penalize policy steps under high uncertainty during optimization.
3. **Adding Adversarial or Distributional Constraints at the Policy Level**: Introduce adversarial training or distributional bounds to prevent speculative convergence.

</details>

**18. How to Detect Reward Hacking in Reinforcement Learning**

<details>
<summary>💡 Show Answer</summary>

Determining whether reward hacking has occurred usually requires integrating signals from multiple dimensions, rather than relying solely on whether "rewards are increasing" on the training curve:

**1. Disconnect Between Reward Increase and Actual Quality Evaluation**

- Core criterion: **The reward provided by the reward model continues to rise during training, but human evaluation or independent, more trustworthy evaluations (such as a stronger LLM-as-Judge or human annotation) reveal that actual output quality has not improved or has even declined**---this divergence where "internal metrics improve while external/real quality degrades" is the most direct signal of reward hacking.

**2. Anomalous/Degenerate Patterns in Generated Content**

- Observe whether the generated content exhibits characteristics that are **unnatural, formulaic, or contrary to "high quality" intuition**, such as:
- Abnormal growth in output length (the "length bias" mentioned earlier, where the model discovers that "longer outputs receive higher rewards");
- Repeated occurrence of specific phrases/sentence structures/formats in the output (e.g., always starting in a fixed manner, or repeatedly piling up sycophantic phrases);
- Outputs that are hollow in content and confused in logic, but whose superficial "form" (such as standard formatting, keyword matching) conforms to patterns preferred by the reward model.

**3. Abnormal Changes in Reward Distribution**

- Observe changes in the **distribution** of reward values across the entire training/sampling dataset---if the mean reward continues to rise while the **variance of the distribution shrinks rapidly** (rewards across all samples converge and cluster around a very high value), it may indicate that the model has found a "universal" high-reward output pattern and applies the same "shortcut" to all or most inputs.

**4. Sharp Increase in KL Divergence Relative to the Reference Model**

- If the KL divergence between the policy model and the reference model (SFT model) **grows rapidly and substantially** during training (far exceeding expectations/historical empirical ranges), it indicates that the policy model's behavior has "significantly deviated" from the original reasonable distribution, which is a common companion phenomenon of reward hacking (or at least "over-optimization").

**5. Performance on Held-Out Evaluation Sets and Out-of-Distribution (OOD) Inputs**

- Evaluate the model on evaluation benchmarks outside the training distribution (covering topics/scenarios different from training); if model performance deteriorates significantly (despite performing well on the reward metrics of the training distribution), it indicates that the model may have "overfit" to specific preferences/exploits of the reward model on the training distribution, rather than learning truly generalizable standards of "high quality."

**6. Adversarial Probing**

- Actively construct probing samples **specifically designed to "induce" reward hacking** (such as prompt types known to be easily exploited by certain models) and observe whether the model's behavior on these samples exposes obvious "opportunistic" patterns.

**Summary**: The core method for detecting reward hacking is to **compare "the score given by the reward model" against "the actual quality measured through independent, more trustworthy means (human evaluation, stronger model evaluation, held-out evaluation sets)" and observe whether a divergence occurs**; simultaneously, combine this with auxiliary signals such as **observing whether generated content displays abnormal formulaic/degenerate patterns, whether the reward distribution is excessively concentrated, and whether the KL divergence relative to the reference model increases abnormally**, providing a comprehensive multi-dimensional assessment.

</details>

**19. What Are the Approaches to Mitigating Reward Hacking?**

<details>
<summary>💡 Show Answer</summary>

![Reward Hacking Detection and Mitigation Framework](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig45_what_are_the_approaches_to_mit.png)




Approaches to resolving reward hacking can be structured across three levels: the "reward signal itself," "algorithmic constraints," and "workflow/monitoring":

**1. Enhancing the Quality and Robustness of the Reward Model Itself**

- **Higher-quality, more comprehensive preference data**: Cover more diverse scenarios and styles to prevent the reward model from learning overly narrow or superficial preference patterns;
- **Reward model ensembling**: Train multiple reward models (with different architectures, data subsets, or random seeds) and aggregate their scores (e.g., mean or minimum), reducing the likelihood that exploits in a single model are abused;
- **Adversarial data augmentation**: Actively construct samples of known "hacking patterns" (e.g., verbose yet empty responses, format-correct but content-erroneous responses) and include them as negative examples in the reward model's training data, improving its discrimination against such superficial pleasing behaviors;
- **Rule-based / verifiable rewards**: For tasks with clear correctness standards (mathematics, code), prioritize rule-based rewards (such as answer verification, unit test pass rate), which are much harder to hack than neural network reward models.

**2. Algorithmic Constraints**

- **KL penalty**: Restrain the policy from deviating too far from the reference model (refer to Questions 398 and 411);
- **Entropy regularization**: Maintain output diversity to prevent the policy from collapsing into a single opportunistic pattern (refer to Question 401);
- **Reward normalization / clipping**: Limit the impact of abnormally high reward values on training (refer to Question 400);
- **Length normalization / control**: Specifically apply normalization targeting length bias, a common hacking pattern (refer to Questions 418 and 431).

**3. Training Workflow and Iteration**

- **Iterative RM updates (Iterative RLHF)**: Periodically use samples generated by the latest policy model for human preference re-annotation to update (retrain) the reward model, allowing the RM to "keep up" with the policy model's evolution and identify newly emerging hacking patterns;
- **Limiting training steps / early stopping**: Reward hacking often emerges gradually as training progresses; closely monitor training and stop early when anomalous signs appear;
- **Multi-objective / multi-dimensional rewards**: Instead of relying on a single scalar reward, combine multi-dimensional evaluations (e.g., separate scoring for usefulness, safety, conciseness before aggregation), reducing the possibility of attaining an overall high score by taking shortcuts in a single dimension.

**4. Continuous Human / Independent Evaluation Monitoring**

- As described in Question 436, periodically use human evaluation, stronger model evaluation, and held-out test sets to "calibrate" the authenticity of reward metrics during training, intervening promptly to adjust whenever a disconnect is detected.

**5. Post-Processing / Output Filtering (Fallback Safeguards)**

- Add rule-based or classifier-based detection at the final output stage (such as checking for abnormal repetition patterns or excessive length) as a fallback defense; even if certain hacking tendencies are not fully avoided during training, they can still be intercepted or corrected to some extent during deployment.

**Summary**: Mitigating reward hacking requires a multi-pronged approach: "**enhancing reward signal quality and robustness (data, ensembling, adversarial augmentation, rule-based rewards) + algorithmic constraints (KL, entropy, normalization) + iterative training workflows (continually updating RM, monitoring for early stopping) + independent evaluation and validation + output post-processing when necessary**." It is a systematic engineering effort requiring continuous investment and iteration, with no silver-bullet single solution.

</details>

**20. What Is the "Over-Optimization" Phenomenon in the RL Scaling Law and Its Mitigation Methods?**

<details>
<summary>💡 Show Answer</summary>

During the RLHF training process, if the policy model over-optimizes against the reward model, it may lead to model outputs that run counter to true human preferences.

**Mitigation Methods**:
1. **Early stopping**: Monitor performance on a validation set and stop training promptly before model performance begins to degrade.
2. **Limiting model deviation**: Introduce a KL divergence penalty term to constrain the deviation between the policy model and the initial model.
3. **Reward model updates**: Mitigate over-optimization by improving the generalization ability and robustness of the reward model.

</details>

**21. Can Random Rewards Also Yield Performance Gains?**

<details>
<summary>💡 Show Answer</summary>

Random rewards can also yield performance gains on certain benchmarks. This phenomenon highlights a potential pitfall in current RL reasoning capability research: **partial test set contamination or "memory re-weighting" effects may be misattributed to genuine improvements in reasoning ability**.

**Phenomenon Description**:

Some studies have found that even when using completely random reward signals during RL training (i.e., noise rewards unrelated to output quality), the model's scores on certain reasoning benchmarks still increase. This contradicts intuition---random rewards should not improve reasoning capabilities.

**Potential Explanations**:
- **Data leakage / test set contamination**: If benchmark test questions were already present in the pre-training data, RL training may reactivate ("un-forget") the model's memory of those questions rather than instilling genuine new reasoning capabilities;
- **"Memory re-weighting" effect**: RL training essentially re-weights the model's output distribution; random rewards may inadvertently boost output probabilities for questions whose answers were seen during pre-training;
- **Special relationship between evaluation and training distributions**: The problem distribution of certain benchmarks may happen to overlap heavily with the prompt distribution of RL rollouts, so even random rewards might guide the model to produce outputs that "better resemble the training distribution";
- **Benchmarks themselves lack robustness**: Some benchmarks may be sensitive to shallow features such as output format or length rather than measuring actual reasoning ability.

**Takeaway**: When evaluating RL effectiveness, one should not rely solely on a single benchmark. Instead, validation should simultaneously employ **fresh test sets strictly protected against contamination, real-user evaluations, and diversified tasks**. Only "carefully designed rewards + rigorous evaluation" can confirm that RL has delivered authentic improvements in reasoning capability, rather than spurious benchmark score increases.

</details>

**22. What Are the Mitigation Strategies for Entropy Collapse During Reinforcement Learning Training?**

<details>
<summary>💡 Show Answer</summary>

**Entropy Collapse**: Refers to a phenomenon where the **entropy of the policy model's output distribution** (the probability distribution over the next token at each generation position) **drops rapidly and trends toward "determinism"** (i.e., the model becomes "overly confident" about what to output at each step, with the probability distribution concentrating heavily on a few tokens). This typically occurs in the early to mid-stages of RL training and causes the model to **lose exploration capability**---in subsequent training, the model almost exclusively generates token sequences it is already confident about, making it difficult to explore potentially better outputs. Consequently, training easily falls into local optima or "reward saturation" (refer to Question 440).

**Mitigation Strategies**:

**1. Increase the Entropy Regularization Coefficient**

- Increase the weight of policy entropy in the loss function (refer to entropy regularization in Question 401) to directly encourage the policy to maintain higher output uncertainty, delaying or suppressing the rapid decay of entropy.

**2. Decoupled / Asymmetric Clipping Range (Clip-Higher, refer to DAPO in Question 432)**

- Standard symmetric clipping $[1-\varepsilon, 1+\varepsilon]$ may restrict the probability growth headroom for "low-probability yet good action" tokens (where the upper bound $1+\varepsilon$ limits the relative magnitude by which low-probability tokens can grow), causing the model to rapidly concentrate probability mass onto tokens that "already have higher probabilities" (leading to entropy decay);
- Adopting a larger upper clipping range $\varepsilon_{\mathrm{high}}$ gives "low-probability good actions" greater headroom to increase, helping to maintain more balanced exploration.

**3. Adjust Learning Rate / Reduce Step Size**

- An excessively large learning rate causes the policy to "over-confidently" reinforce the probability of certain tokens at each update step, accelerating entropy collapse; appropriately lowering the learning rate makes policy updates smoother.

**4. KL Penalty Constraints**

- Strengthen the KL penalty (increase $\beta$) to restrict the deviation of the policy relative to the reference model; since the reference model typically exhibits a more dispersed output distribution (having not yet undergone RL sharpening), the KL penalty indirectly helps maintain distribution dispersion.

**5. Increase Sampling Temperature / Sampling Diversity (in Rollout Phase)**

- Use a slightly higher sampling temperature during rollout to increase generated sample diversity, providing more varied data for training and indirectly countering the policy's tendency toward premature distribution concentration.

**6. Monitoring and Timely Intervention**

- Continuously monitor policy entropy trends during training; once entropy rapidly drops below a certain threshold, **promptly adjust the aforementioned hyperparameters (increase entropy regularization, adjust clipping ranges, lower learning rate)** rather than waiting until entropy nears 0 (complete determinism)---at which point the model may be locked into a suboptimal pattern that parameter adjustments can no longer remedy.

**7. Check Whether the Task/Data Itself Is Too Simple**

- Related to the "premature convergence" issue in Question 440---if the training data has low overall difficulty, the model may "quickly" generate high-confidence (correct) outputs for most inputs, making the entropy drop somewhat "reasonable" (the model has indeed mastered the task); in such cases, entropy collapse itself may not be a problem, and must be evaluated alongside actual performance metrics (distinguishing between "healthy convergence" and "pathological premature convergence").

**Summary**: The core idea for mitigating entropy collapse is to **actively preserve the dispersion of the policy output distribution using entropy regularization, asymmetric clipping ranges (Clip-Higher), KL penalties, learning rate reduction, etc., suppressing premature and excessive concentration of probability mass onto a few tokens**. This requires **continuous monitoring of entropy trends for early intervention**, alongside evaluating task difficulty to discern whether "entropy reduction" represents a "natural outcome of healthy convergence" or "pathological premature convergence."

</details>

**23. What Are the Mitigation Strategies When Premature Convergence Occurs During GRPO Training (i.e., Reward Saturates Before Completing a Single Epoch)?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for What Are the Mitigation Strategies When Premature Convergence Occurs During GRPO Training (i.e., Reward Saturates Before Completing a Single Epoch)?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig46_what_are_the_mitigation_strate.png)




The "premature convergence / reward saturation" phenomenon refers to a scenario early in training where reward metrics rise rapidly and plateau (no longer showing noticeable gains). Potential causes and mitigation strategies include:

**1. Distinguishing True Mastery from Fake Saturation Caused by Reward Hacking**

- First, following the methods in Question 436, verify whether reward saturation is accompanied by a **genuine improvement in actual output quality** (via independent evaluation); if it is fake saturation caused by reward hacking (the model found a shortcut), handle it according to Question 437 (enhancing RM robustness, strengthening KL constraints, etc.) rather than simply continuing training.

**2. If the Task Is Indeed Learned Quickly at Its Current Difficulty**

- **The training data/task may be overly simple**: If the overall difficulty of the training prompt set is low, the model may reach a state where "rewards approach upper bounds on these easy tasks" within very few training steps;
- Actionable strategies:
- **Introduce harder samples (Curriculum Learning)**: Analyze items in current training data where the model consistently achieves high rewards, and gradually introduce more difficult samples where performance is lacking to keep training signals effective (preventing the advantage $A_i$ of many samples from approaching $0$, referencing DAPO's dynamic sampling concept---sample groups that are entirely correct cannot provide effective gradients);
- **Filter out "saturated" samples**: Lower the sampling frequency of problems the model reliably solves, concentrating compute resources on samples where the model still has room for improvement.

**3. Check Whether Entropy Has Collapsed Prematurely (Entropy Collapse)**

- Premature convergence is frequently accompanied by a **rapid decline in policy entropy**---the model quickly becomes over-confident, the output distribution becomes deterministic, and the exploration space contracts sharply;
- Actionable strategies:
- **Increase the entropy regularization coefficient** to encourage continued exploration;
- **Adjust clipping ranges (e.g., DAPO's Clip-Higher strategy)** to grant low-probability good tokens greater room for probability increase, preventing premature locking into a specific output pattern;
- **Lower the learning rate** to slow down policy changes, giving the model more opportunities to explore different outputs rather than rapidly converging to a local optimum.

**4. Check Whether KL Divergence Has Become Substantial**

- If KL divergence has increased significantly alongside reward saturation, it may indicate that the model has fully adjusted its behavior relative to the reference model, yielding diminishing marginal returns from further training;
- Actionable strategies: Evaluate model performance on broader/harder evaluation benchmarks; if overall capability has substantially improved, consider **ending the current training stage early** and directing compute resources toward the next stage (e.g., using harder data or refining reward design).

**5. Check Whether the Reward Design Imposes an Artificial "Ceiling"**

- If the reward function design inherently contains an **easily reachable upper bound** (e.g., discrete binary 0/1 rewards where the model quickly obtains 1 on most samples), reward saturation naturally arises---while not necessarily a flaw, it implies that **the current reward signal can no longer provide effective discrimination**;
- Actionable strategies: **Design fine-grained rewards** (such as transitioning from binary 0/1 to multi-dimensional continuous rewards accounting for process quality, efficiency, etc.; refer to Question 438) to reintroduce discriminative power and guide further model optimization.

**Summary**: The workflow for addressing GRPO premature convergence / reward saturation is: "**First check if it is fake saturation caused by reward hacking (independent evaluation verification) $\rightarrow$ If true mastery, consider introducing harder samples/curriculum learning to maintain training signal efficacy $\rightarrow$ Check if entropy dropped prematurely, increasing entropy regularization / adjusting clipping range / lowering learning rate as needed to sustain exploration $\rightarrow$ Check if the reward design itself has an overly low ceiling, designing finer-grained rewards when necessary**." The core lies in diagnosing the nature of saturation (genuine progress vs. fake/overly simple task) and targeting data difficulty, exploration mechanisms, or reward designs accordingly.

</details>

**24. What Are the Mitigation Strategies When Training Reward Rises but Validation Performance Drops During RL Training?**

<details>
<summary>💡 Show Answer</summary>

![Overoptimization Curve and Early Stopping](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig47_what_are_the_mitigation_strate.png)




This phenomenon of "training reward increasing while validation performance drops" is fundamentally a **disconnect between the training objective (reward) and the true generalization objective**. Potential causes and mitigation strategies include:

**1. Reward Hacking (Overfitting/Opportunism on the Training Distribution)**

- The most common cause---the model finds a "shortcut" on the training reward signal/distribution, driving reward continuously upward; however, this shortcut is ineffective or even harmful on the validation set (or broader distributions);
- Mitigation: Refer to Questions 436 and 437 to enhance reward model/rule robustness, strengthen KL constraints, and implement independent evaluation monitoring.

**2. Discrepancy Between Validation Set and Training Distribution (Out-of-Distribution Generalization Issue)**

- If scenarios covered by the validation set differ substantially from training prompt distributions, over-optimization on training distributions may specialize the model to training-specific patterns, leading to poor generalization on the validation set (OOD);
- Mitigation: Expand the diversity and coverage of training prompts to align the training distribution more closely with real-world application / validation set distributions.

**3. Excessive KL Divergence and Catastrophic Forgetting**

- KL divergence between the policy and reference model may have grown significantly during training; in pursuing training rewards, the model degrades in other capabilities (which the validation set may assess);
- Mitigation: Increase the KL penalty coefficient $\beta$ to restrict policy deviation from the reference model, or execute early stopping (before KL becomes excessive).

**4. Overfitting to Specific Training Sample Content (Rather than Learning Generalizable Capabilities)**

- If the training prompt set is small or has high redundancy, the model may "memorize" high-reward output patterns specific to those prompts rather than learning transferable general capabilities;
- Mitigation: Scale up and diversify training data; apply appropriate regularization (such as entropy regularization) to retain exploratory randomness for generalization.

**5. Issues in the Validation Set Evaluation Methodology Itself**

- Check whether the validation set evaluation method is sound---for instance, whether there is a systematic inconsistency between the "judge"/rules used for validation and the training reward signal (e.g., training reward encourages "conciseness" while validation favors "detail", creating conflicting objectives that naturally cause validation scores to drop as training reward rises);
- Mitigation: Ensure alignment in direction between training rewards and final evaluation objectives (represented by the validation set), adjusting reward design when necessary to closely reflect final evaluation criteria.

**6. Instability Caused by Inappropriate Training Steps or Learning Rates**

- Investigate whether late-stage training instability has occurred (e.g., extreme behaviors following entropy collapse where rewards remain high on specific training prompts while generalization deteriorates);
- Mitigation: Reduce the learning rate, increase regularization, and apply early stopping (determining stop points based on validation metrics rather than training rewards, referring to the discussion on "when to stop training" in Question 390).

**Summary**: The core realization is that "**training reward increase $\neq$ true capability improvement**." Resolution strategies revolve around investigating multiple angles: "**Check for reward hacking (independent evaluation, strengthening reward robustness and KL constraints) $\rightarrow$ Inspect training/validation distribution differences (expanding training data diversity) $\rightarrow$ Check for overfitting to specific training samples (data diversification, regularization) $\rightarrow$ Verify alignment between validation criteria and training rewards $\rightarrow$ Determine stopping points based on validation metrics rather than training rewards**." Fundamentally, this reflects the same type of "decoupling between training signals and true objectives" in RL scenarios as discussed in Question 390 regarding "validation loss reduction during SFT not guaranteeing better performance."

</details>

**25. How to Prevent RL from Being Effective on Small Models but Performing Poorly on Large Models?**

<details>
<summary>💡 Show Answer</summary>

The situation where RL is effective on small models but performs poorly on large models is a common RL scaling problem, requiring approaches addressing model differences, data design, and algorithmic adjustments.

**Root Cause Analysis**:
- **Differences in output distribution diversity**: Large model outputs tend to be semantically more "concentrated" (due to strong capabilities, they answer most problems correctly, resulting in small within-group reward variance), whereas small models produce responses with distinct correctness/quality gaps for the same problem due to limited capabilities (resulting in large within-group reward variance and stronger gradient signals);
- **Variations in exploration space**: When small models explore diverse responses, quality gaps between different trials are more pronounced; for large models, quality between different samples is close, making it difficult to distinguish good from bad;
- **Differences in SFT baseline starting point**: Large models typically reach a high baseline post-SFT, leaving limited room for marginal gains via RL.

**Mitigation Methods**:
- **Increase Sampling Temperature and Group Size $G$**: Use a higher temperature (e.g., 0.8--1.2) and a larger $G$ (e.g., 32--64) during rollout to increase within-group output diversity;
- **Curriculum Learning**: Design a task distribution with higher difficulty for large models to ensure the proportion of "all-correct/all-wrong" groups is not excessively high, thereby preserving effective gradient signals;
- **Dynamic Sampling in DAPO**: Filter out all-correct or all-wrong sample groups to focus training on questions where the large model "still exhibits uncertainty";
- **Use Process Reward Models (PRM)**: When the large model can already obtain correct final answers, use step-level evaluations of intermediate reasoning to provide finer-grained discriminative signals;
- **Adjust Hyperparameters**: The learning rate for large models should be smaller (on the order of $1e-7$), and the KL coefficient should be appropriately increased to prevent performance degradation caused by over-updating on already well-performing tasks.

</details>

**26. How Does GRPO Improve Sample Efficiency During Rollout?**

<details>
<summary>💡 Show Answer</summary>

In GRPO training, rollout (sampling generation) is typically the **most computationally expensive stage** (involving autoregressive generation, which cannot be massively parallelized like the training stage). Strategies to improve the utilization efficiency of rollout samples include:

**1. Avoid Sampling "Ineffective Groups" (Dynamic Sampling, Ref. Question 432 DAPO)**

- For "all-correct" or "all-wrong" prompts (where all responses within the group receive identical rewards, resulting in zero variance and no effective gradient signal), the computational resources spent on this rollout are essentially wasted;
- One can **predetermine (or dynamically estimate during rollout) the difficulty of a prompt**. For prompts where it is known (based on historical training records) that "the model can already consistently answer correctly/incorrectly", **reduce their sampling frequency** and allocate more rollout resources to prompts with "moderate difficulty" (where model performance is still unstable and intra-group rewards show discriminative variance).

**2. Increase Parallelism and Throughput of a Single Rollout**

- Use efficient inference frameworks (such as vLLM and other inference engines supporting continuous batching and PagedAttention) for rollout to maximize GPU utilization and make the computationally intensive generation stage as efficient as possible;
- **Batching**: Perform batched generation simultaneously across multiple prompts and multiple responses per prompt ($G$ responses within a group) to fully leverage the parallel computing capabilities of GPUs.

**3. Mitigate the "Long-Tail" Problem (Ref. Question 451)**

- Generation lengths of different responses within a batch often vary significantly (some terminate quickly while others generate very long outputs). If an approach of "synchronously waiting for all samples to complete generation before starting training" is adopted, **computing resources allocated to short responses remain idle while waiting for long responses to finish**;
- One can adopt **asynchronous/pipelined rollout and training** (ref. Questions 449 and 453 on asynchronous frameworks) to allow completed samples to enter the training pipeline as early as possible without waiting for the longest sample in the entire batch.

**4. Reuse/Cache Computation Results**

- If multiple prompts share common prefixes (e.g., identical system prompts or similar few-shot examples), leverage **KV cache prefix reuse** to avoid redundant attention computation for shared prefix portions, thereby improving generation efficiency.

**5. Curriculum Learning / Data Filtering**

- Combining the insights from Question 440, dynamically adjust the proportions of various prompt types in the training data via curriculum learning. This ensures that the sample "groups" produced during rollout exhibit effective reward variance more frequently, reducing the ratio of "ineffective" rollouts.

**6. Reuse Rollout Samples for Multiple Update Epochs (Within Limits Allowed by PPO/GRPO)**

- In standard practice, after a single rollout, multiple epochs (e.g., 2--4 epochs) of mini-batch gradient updates are performed to fully utilize the samples generated from each rollout (within the "near on-policy" constraint bounds), rather than updating for only a single step per rollout.

**Summary**: The core strategies for enhancing rollout sample efficiency in GRPO include "**avoiding computational waste on 'ineffective groups' (all-correct/all-wrong) via dynamic sampling and curriculum learning; maximizing rollout throughput using high-efficiency inference frameworks (e.g., vLLM) and batching; mitigating idle resource overhead caused by length long-tail issues through asynchronous/pipelined designs; reducing redundant computation via KV cache prefix reuse; and performing multi-epoch gradient updates on the same batch of rollout samples within on-policy constraint bounds**." These multi-faceted approaches jointly boost overall training efficiency.

</details>

**27. What Is the Inference-Time Scaling Law?**

<details>
<summary>💡 Show Answer</summary>

![Inference-Time Scaling Law: Compute vs. Accuracy](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig48_what_is_the_inference_time_sca.png)




The Inference-Time Scaling Law (or Test-Time Compute Scaling) represents a major breakthrough in LLM research in 2024, revealing that **increasing compute during the inference phase (rather than increasing model parameters) can continuously improve model performance on complex tasks**.

**Core Discovery**:

Analogous to the pre-training "Scaling Law" (more parameters + more data $\rightarrow$ better performance), during the inference phase: granting the model more "thinking time" (more tokens) leads to sustained performance gains within a certain range. This improvement is particularly pronounced for complex reasoning tasks (such as mathematics, coding, and logical reasoning).

**Primary Methods to Achieve Inference Scaling**:
- **Long Chain-of-Thought (Long-CoT)**: As seen in OpenAI o1 and DeepSeek-R1, generating ultra-long chains of thought containing self-reflection and multiple verifications, where reasoning tokens can reach thousands or even tens of thousands;
- **Best-of-$N$ Sampling**: Sampling multiple candidates for the same problem and selecting the optimal answer, where performance improves as $N$ increases;
- **Search Algorithms**: Combining LLMs with Monte Carlo Tree Search (MCTS) to systematically explore the reasoning space.

**Relationship Between RL and Inference Scaling**:

Reinforcement learning methods such as GRPO train the model to learn "how to effectively utilize more thinking tokens" through online exploration. The model not only generates longer reasoning sequences, but also learns effective strategies such as self-verification, error reflection, and attempting alternative methods, making inference scaling genuinely effective rather than merely generating "more verbose filler."

**Trade-offs**: Inference scaling improves performance but also increases latency and cost. Choosing between "stronger single-pass reasoning" and "more parallel sampling" depends on the requirements of specific application scenarios.

</details>

**28. Explain Long-CoT, Short-CoT, and Long2Short**

<details>
<summary>💡 Show Answer</summary>

**Long-CoT (Long Chain-of-Thought)**:

Refers to the model generating an **ultra-long reasoning process containing numerous intermediate steps** before producing the final answer. This typically includes problem decomposition and planning, trying and comparing multiple approaches, self-verifying intermediate results ("let me check this..."), and reflecting upon and correcting errors upon detection ("wait, there's a problem here, let me rethink..."). Representative models include OpenAI o1/o3 and DeepSeek-R1. Advantages: significantly improves accuracy on complex tasks; Disadvantages: high inference latency (ranging from seconds to tens of seconds) and substantially increased cost.

**Short-CoT (Short Chain-of-Thought)**:

The standard reasoning approach used by traditional LLMs (such as GPT-4 and Claude 3), which generates concise reasoning steps (typically a few sentences to a single paragraph) before providing the answer. Advantages: fast response time and low cost; Disadvantages: lower accuracy on complex multi-step reasoning tasks compared to Long-CoT.

**Long2Short (Transfer from Long to Short Reasoning)**:

Compressing the capabilities of a Long-CoT model into a Short-CoT model, with the goal of **significantly reducing reasoning token counts while maintaining high accuracy**:
- **Distillation-Based**: Using Long-CoT models to generate responses and training Short-CoT models to learn "efficient reasoning paths";
- **RL-Based Length Control**: Incorporating length penalty rewards during RL training to enable the model to shorten its chain of thought while maintaining accuracy (e.g., token-level loss design in DAPO);
- **Dynamic Switching**: Automatically deciding whether to use Long-CoT or Short-CoT based on problem complexity—answering simple questions directly while engaging in deep reasoning for complex ones.

</details>

**29. How to Optimize a Model from Long Reasoning to Short Reasoning?**

<details>
<summary>💡 Show Answer</summary>

Methods to achieve Long2Short include:
1. Model Merging: Merging long-reasoning and short-reasoning models to produce outputs of intermediate length.
2. Length Penalty: Introducing length penalties in reinforcement learning to balance reasoning length and accuracy.
3. Shortest Data Filtering: Filtering the shortest correct reasoning paths generated by long-reasoning models to serve as training data.
4. Direct Preference Optimization: Directly optimizing the short reasoning process via preference-based reinforcement learning.

</details>

**30. Is Enhancing Reasoning Ability via Reinforcement Learning More Effective Than Simply Increasing Model Parameters?**

<details>
<summary>💡 Show Answer</summary>

This is a central debate in current LLM research. The answer is that **the two are complementary with their own applicable scenarios, but RL demonstrates unique advantages in elevating reasoning "quality"**.

**Evidence of RL Enhancing Reasoning Ability**:
- Works such as DeepSeek-R1 demonstrate that models trained with GRPO can **significantly outperform larger-parameter models that lack reasoning-specific RL training** on reasoning tasks such as math competitions and coding benchmarks;
- The 7B-parameter DeepSeek-R1-Distill matches or even surpasses 70B general-purpose models on specific reasoning tasks, demonstrating the computational efficiency gains driven by reasoning RL training;
- RL allows models to autonomously explore effective reasoning paths (such as reflection, self-verification, and trying multiple methods)—behaviors that simply increasing model parameters cannot guarantee to emerge.

**Limitations of Simply Increasing Parameters**:
- Increasing parameters primarily expands knowledge capacity and language comprehension, but cannot guarantee the emergence of "deep reasoning" capabilities;
- Examples of reasoning steps in pre-training data are limited, leading models to mostly "imitate" existing reasoning patterns;
- Marginal gains from scaling parameter counts are diminishing, whereas RL combined with inference-time scaling laws offers an alternative pathway for performance improvement.

**Complementary Relationship**:
- Parameter size determines the "knowledge foundation" and the "upper limit of potential capacity"—if a model is too small, even RL struggles to elicit complex reasoning abilities;
- RL dictates "how to efficiently utilize existing knowledge"—at the same parameter scale, models trained with reasoning RL significantly outperform models relying solely on SFT on complex tasks.

Conclusion: **For complex reasoning tasks, RL offers higher cost-effectiveness than simply scaling up parameter size, but the two are not mutually exclusive—combining large parameters with reasoning RL achieves optimal performance**.

</details>

**31. Are Text-Based Generation Tasks Also Suitable for Post-Training with Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

Using reinforcement learning for post-training on open-ended text tasks (such as creative writing, general dialogue, and subjective Q&A) is **feasible, but faces significantly more challenges than reasoning tasks**, requiring substantially more effort in reward design.

**Major Challenges of RL for Text Tasks**:
- **Difficulty in Reward Design**: Text quality evaluation is highly subjective ("what constitutes good creative writing" varies per person); without objective ground-truth answers for verification, one must rely on Reward Models (RM) or LLM-as-a-Judge, introducing additional noise;
- **Higher Risk of Reward Hacking**: RM scores for text are easily exploited by surface-level features (such as length, format, or specific vocabulary), making it easier for policy models to discover "shortcut hacks";
- **Multi-Dimensional Preferences Are Hard to Express via a Single Reward**: High-quality text requires balancing multiple dimensions simultaneously—such as informativeness, fluency, safety, and conciseness—which a single scalar reward can hardly encapsulate.

**Suitable Scenarios and Solutions**:
- **Constrained Text Tasks**: Tasks such as format adherence (JSON output, word count limits) that can be evaluated using rule-based rewards are well-suited for RL training;
- **Safety Alignment**: For behaviors such as "refusing harmful requests," safety rewards can be defined relatively objectively (via safety classifiers), where RL yields strong results;
- **RLHF with High-Quality RMs**: For subjective tasks such as general dialogue, using a well-trained RM alongside strict KL constraints remains the standard applicable domain for RLHF;
- **DPO May Be a Better Choice**: For highly subjective text tasks, Direct Preference Optimization (DPO) is often more stable and less prone to reward hacking than PPO/GRPO, making it frequently preferred in practice.

</details>

**32. Are You Familiar with Route Replay During MoE Model Training?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for Are You Familiar with Route Replay During MoE Model Training?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig49_are_you_familiar_with_route_re.png)




**Background—Special Issues of MoE Models in RL Training**:

In a Mixture of Experts (MoE) model, during forward propagation, each token passes through a **router** that dynamically selects which experts to activate based on the current hidden states. This "routing decision" itself depends on the model's current parameters (the router's parameters and all upstream parameters influencing the hidden states).

In the typical workflow of RL training (such as PPO/GRPO):
1. **Rollout Phase**: Generate samples using the current policy $\pi_{\theta_{old}}$ (where the router makes routing decisions based on $\theta_{old}$, assigning each token to specific experts);
2. **Training Phase**: Use the generated samples to compute losses and update parameters to $\theta$ (over multiple gradient update steps).

**Problem Addressed by Route Replay**:

During the training phase, when computing $\pi_{\theta}(y_t \mid s_t)$ (i.e., recalculating the probability of each token in the generated sample using updated parameters to compute the importance sampling ratio), **the router of the MoE model under parameters $\theta$ may select different experts for the same token compared to when using $\theta_{old}$**—that is, the "routing decisions have changed."

This introduces an issue: during rollout, the token was actually generated by the "expert combination corresponding to $\theta_{old}$", but during training, when recomputing $\log \pi_{\theta}(y_t \mid s_t)$, the router may switch to a "different expert combination corresponding to $\theta$". **This "routing inconsistency" causes the calculated probabilities/gradients to mismatch the actual computation path taken during token generation**, introducing additional bias and instability (representing a special manifestation of the "training-inference discrepancy" discussed in Question 452 within MoE scenarios).

**Mechanism of Route Replay**:

To ensure that the probabilities recomputed during training remain **consistent** with the computation path during rollout, "Route Replay" means: **when recalculating $\log \pi_{\theta}(y_t \mid s_t)$ during training, force the system to reuse the routing decisions (i.e., which experts each token was assigned to) recorded during the rollout phase (under $\theta_{old}$), rather than allowing the router to make (potentially different) routing decisions based on the current parameters $\theta$**—that is, "replaying" the rollout routing paths while evaluating output probabilities using the updated expert parameters $\theta$ along those fixed paths.

**Effect and Significance**:
- Ensures that the computation of $\log \pi_{\theta}(y_t \mid s_t)$ remains **consistent in "computation graph structure" (i.e., activated expert combinations) with rollout**, such that only the inner expert parameters change (which is precisely what gradient updates aim to modify), without introducing extraneous probability ratio variations caused by "routing path shifts";
- This enables the importance sampling ratio $ratio_t = \pi_{\theta}(y_t \mid s_t) / \pi_{\theta_{old}}(y_t \mid s_t)$ to accurately reflect how expert parameter updates themselves affect token probabilities, uncorrupted by changes in routing decisions, thereby **enhancing the stability of MoE models during RL training**.

**Summary**: Route Replay is a technique employed during MoE RL training to resolve the issue where "recalculating probabilities during training may cause the router to select different experts under updated parameters, leading to computational path mismatches relative to rollout." **By enforcing the reuse (replay) of routing decisions recorded during rollout, it ensures consistency between training probability calculations and generation paths, allowing importance sampling ratios to accurately reflect parameter updates and improving RL training stability in MoE models**.

</details>

**33. Is the VeRL Framework Synchronous or Asynchronous, and What Is Its Concrete Training Workflow?**

<details>
<summary>💡 Show Answer</summary>

Basic Design of **VeRL (Volcano Engine Reinforcement Learning, an open-source RLHF training framework by Volcano Engine)**:

VeRL **adopts a synchronous training workflow by default** (although its architectural design flexibly supports extension to asynchronous modes, its classic and primary operational mode is synchronous).

**Concrete Training Workflow (Synchronous Mode)**:

VeRL adopts the so-called **"HybridFlow"** architecture, organizing computations (generation, forward pass, backward pass) across different roles in the RL training pipeline (Actor, Critic, Reward Model, Reference Model) into a clear, stage-structured process. Each training iteration roughly comprises:

1. **Rollout Phase**: Uses the current Actor model (typically leveraging efficient inference engines like vLLM/SGLang) to generate responses for a batch of prompts (e.g., generating $G$ responses per prompt in GRPO). The workflow transitions to the next phase only after all rollout tasks in this stage are completed—i.e., **synchronously** waiting for the entire batch of rollouts to finish;

2. **Scoring Phase**: Uses a Reward Model (and/or rule-based verifiers) to compute rewards for the responses obtained from rollout;

3. **Value Estimation Phase (in PPO)**: Uses the Critic model to compute value estimates $V(s)$ for each position, combined with the Reference Model to compute KL penalties; in GRPO, computes group-normalized advantages instead;

4. **Training (Update) Phase**: Based on the computed advantages/reward signals, performs gradient updates on the Actor (and Critic, if applicable) parameters (potentially spanning multiple mini-batches and epochs);

5. **Weight Synchronization**: Updated Actor parameters must be synchronized back to the inference engine used for rollout (e.g., updating vLLM weights to the latest Actor parameters) so that the next iteration uses the latest policy for rollout;

6. Repeat the above process to proceed to the next iteration.

**"Synchronous" Behavior Manifestation**: In each iteration, **the rollout, scoring, and training stages execute sequentially, strictly waiting for each other to complete**—the current batch of rollouts must finish entirely before scoring begins; scoring must finish before training starts; and training together with weight synchronization must finish before launching the next rollout round. This synchronous mode is simple to implement and logically clear, but suffers from the "long-tail" problem discussed in Question 451 (a few extra-long generations in a rollout batch stall the completion of the entire batch, leaving completed samples and their computing resources waiting in idle state).

**Summary**: VeRL (in its classic/default setup) adopts a **synchronous** training workflow, cycling through the order of "**Rollout (Generation) $\rightarrow$ Scoring (Reward/Rule Verification) $\rightarrow$ Value Estimation/Advantage Computation $\rightarrow$ Parameter Update $\rightarrow$ Weight Sync to Inference Engine**". Strict dependencies and waiting exist between stages; this design is transparent and easy to debug, but suffers from low resource utilization caused by length long-tails when rollout lengths vary significantly (ref. Question 451). Consequently, academia and industry are actively exploring asynchronous optimizations (ref. Question 453).

</details>

**34. What Parameters Need to Be Configured When Training with the VeRL Framework?**

<details>
<summary>💡 Show Answer</summary>

Parameters to configure in VeRL training can be broadly categorized into the following groups (combining general RL hyperparameters from Question 446, with emphasis on VeRL-specific/engineering configurations below):

**1. Model and Parallelism Configurations**

- **Model paths/checkpoints** for Actor, Critic (if used), Reward Model, and Reference Model;
- **Parallelism strategies** for each model (specific dimensions for data parallelism, tensor parallelism, and pipeline parallelism). Models of different roles may use different parallelism configurations (e.g., using FSDP/Megatron during Actor training, while using tensor parallelism in vLLM during rollout);
- Whether to enable PEFT methods such as **LoRA**.

**2. Rollout Configurations**

- Choice of inference engine (such as vLLM or SGLang) and its parameters (e.g., GPU memory utilization, maximum concurrent requests);
- **Sampling parameters**: temperature, $top_p$, $top_k$, etc.;
- **Number of samples per prompt** (group size $G$ in GRPO);
- **Maximum generation length** ($max_response_length$);
- **Rollout batch size** (number of prompts processed per iteration).

**3. Training Configurations**

- **Learning rates** (set separately for Actor and Critic);
- **Optimizer** configurations (e.g., AdamW betas, weight decay);
- **Training mini-batch size** and **gradient accumulation steps**;
- **Number of PPO epochs** per rollout;
- **Learning rate schedule strategy** (warmup steps, decay scheme).

**4. RL Algorithm Hyperparameters**

- **KL penalty coefficient $\beta$** (and its estimation method, such as selection of k1/k2/k3 estimators);
- **Clipping range $\varepsilon$** (PPO/GRPO clip ratio, whether asymmetric clipping is used);
- **GAE parameters $\gamma$ and $\lambda$** (for PPO);
- **Entropy regularization coefficient**;
- Value function loss weight $c_1$ (for PPO).

**5. Reward/Verification Configurations**

- Reward Model path (if using a neural network RM);
- Logic configurations for rule-based reward/verifiers (e.g., rules for answer extraction and comparison in math tasks);
- Whether to enable reward normalization/clipping.

**6. Distributed and Resource Configurations**

- Total number of GPUs and their allocation across different roles (Actor/Critic/RM/Ref/Rollout engine);
- Whether to enable CPU offloading (a common option in actual setups);
- Whether models of different roles are colocated on the same set of GPUs (time-sharing) or deployed separately.

**7. Training Flow Control Parameters**

- Total training steps/epochs;
- Checkpoint saving frequency;
- Validation/evaluation frequency and validation set configuration;
- Logging configurations (e.g., wandb project name).

**Summary**: Parameter configurations in VeRL training cover multiple dimensions, including "**model and parallelism strategies, rollout generation settings (sampling parameters, group size, max length), training optimization parameters (learning rate, batch size, epochs), RL algorithm hyperparameters (KL coefficient, clipping range, GAE parameters, entropy coefficient), reward/verification settings, and distributed resource allocations**." It constitutes a comprehensive configuration system combining algorithm hyperparameters and systems engineering options, typically managed uniformly via YAML configuration files in practice.

</details>

**35. What are the possible issues of synchronous frameworks, and what solutions exist for the long-tail problem in rollout?**

<details>
<summary>💡 Show Answer</summary>

**Possible issues of synchronous frameworks**:

1. **"Long-tail" Problem**: Within a batch, the generated response **lengths from different prompts can vary greatly** (some responses quickly hit EOS and terminate, while others require generating very long sequences before ending, especially in reasoning tasks where the length of the "thinking" process can differ by orders of magnitude). Because synchronous frameworks require **waiting for the entire batch of rollouts to complete before proceeding to the next stage**, the computing resources (GPUs) corresponding to those "short responses" remain in an **idle waiting** state while the "long responses" are still being generated. This leads to low overall GPU utilization, making the rollout stage a clear bottleneck in the training pipeline (potentially taking up a large fraction of the total execution time).

2. **"Phase-wise" Waste of Resource Utilization**: The rollout stage (generation-intensive) and the training stage (gradient-computation-intensive) have different demand patterns for computing resources (rollout relies more on inference optimizations such as KV caching, while training relies more on large-batch parallel computing). Under a synchronous framework, executing the two stages sequentially means that during the rollout stage, certain resources required for "training" (if disaggregated deployment is used) sit idle; conversely, during the training stage, rollout resources sit idle. If the two are colocated, switching between two different computation modes is required, incurring additional switching overhead.

3. **Weight Synchronization Overhead**: After the Actor parameters are updated in each iteration, the new parameters need to be synchronized to the inference engine used by rollout. This synchronization process (especially when the model scale is very large) itself takes a non-trivial amount of time, further increasing the total time per iteration in synchronous frameworks.

**Solutions for the long-tail problem**:

1. **Asynchronous Pipelining**: Make rollout and training **not strictly aligned by "batch"**---samples that have completed generation can **immediately enter the scoring and training pipeline** without waiting for all samples in the same batch to finish. The samples used for training may come from rollouts completed at "slightly different time points," introducing a certain degree of off-policy nature (which needs to be balanced against the train-inference consistency issue discussed in Question 452).

2. **Partial Rollout / Truncation and Continuation**: For samples with exceptionally long generation times, one can consider **first letting their "partially generated results" participate in a round of training** (e.g., treated as "incomplete" with appropriate reward shaping, referring to DAPO's reward shaping for ultra-long samples). The "remaining generation" of the sample can continue/be supplemented in subsequent rounds, avoiding a single "ultra-long" sample slowing down the progress of the entire batch.

3. **Dynamic Batching / Grouping by Length**: During rollout scheduling, try to **group prompts with similar expected generation lengths into the same batch** to reduce length variance within the same batch (although the generation length itself is difficult to predict accurately before generation, rough grouping can be performed based on historical statistics/heuristics).

4. **Oversampling + Early Termination**: Sample **more samples than actually needed** in a batch of rollouts. Once a "sufficient number" of samples have finished generation (reaching the batch size required for training), **early terminate/cancel** those samples that are still generating and taking too long (treated as "discarded" or "truncated"), preventing the whole batch from being dragged down by a very small number of "ultra-long tail" samples.

5. **Increase Concurrency and Leverage Continuous Batching of Inference Engines**: Use inference engines supporting continuous batching (such as vLLM) so that "earlier-completed requests" can be promptly replaced by new requests to fill GPU resource slots. Even within a "large batch" in a synchronous framework, resource utilization can be maximized internally during rollout (although this cannot completely eliminate the synchronous framework's constraint of "waiting for the entire batch to complete," it mitigates resource waste within the batch).

**Summary**: The core issue of synchronous frameworks is the **resource idleness (low GPU utilization) caused by synchronous waiting due to the "long-tail distribution" of generation lengths in the rollout stage**, as well as the extra overhead brought by phase switching and weight synchronization. Approaches to solving the long-tail problem include **asynchronous pipelining (starting subsequent processing without waiting for the entire batch to finish), Partial Rollout (truncating ultra-long samples and continuing later), grouping batching by expected length, oversampling + early termination**, etc. The core idea is to break the strict synchronous constraint of "having to wait for the entire batch of rollouts to complete" so that computing resources remain as "busy" as possible. This is also one of the main drivers promoting the evolution of RL training frameworks toward the **asynchronous direction** (refer to Question 453).

</details>

**36. Are you familiar with the train-inference mismatch problem in RL? What are the causes of train-inference mismatch, and what solutions exist?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for Are you familiar with the train-inference mismatch problem in RL? What are the causes of train-inference mismatch, and what solutions exist?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig50_are_you_familiar_with_the_trai.png)




**Train-Inference Mismatch Problem**:

Refers to the fact that in the RL training pipeline, between the **"inference engine used to generate rollout samples (such as vLLM)"** and the **"training framework used to compute gradients and update parameters (such as FSDP/Megatron)"**, there may exist a **small but non-zero numerical discrepancy** in the "token probabilities (logits/log-probs) computed for the exact same model under the exact same input". This discrepancy affects the accuracy of computing the importance sampling ratio = $\pi_{\theta}$(y|x)/$\pi_{\theta_{old}}$(y|x), which in turn impacts the training stability and correctness of algorithms such as PPO/GRPO.

**Causes**:

1. **Different Computation Implementations / Kernel Differences**: The training framework and inference framework (e.g., vLLM) may use **different underlying operator implementations** (e.g., different attention implementations, different matrix multiplication library versions, different operator fusion methods). Even if they are mathematically computing the "same function," due to the **non-associativity (order sensitivity) of floating-point arithmetic**, different intermediate computation orders across implementations lead to slight numerical differences in the final output logits (usually within floating-point precision error bounds, but after softmax and log operations, these minor differences can be amplified).

2. **Precision Differences**: To pursue speed, the inference engine may use lower numerical precision than the training framework (e.g., FP8/INT8 quantization for inference, BF16/FP32 for training). Precision differences directly result in numerical discrepancies in computation results.

3. **Parallelism Strategy Differences**: The tensor parallelism / data parallelism splitting strategies used during training may differ from those used by the inference engine, leading to slight numerical discrepancies in certain computations involving cross-device communication/reduction (such as LayerNorm and softmax normalization) due to different floating-point addition orders.

4. **Batching / Padding Differences**: The batching method of samples in the inference engine (e.g., different requests mixed together in continuous batching) differs from the batching method during training. Certain computations dependent on "other samples within the batch" (theoretically there should not be such dependence, but implementation details like padding handling may introduce it) might produce discrepancies.

**Problems caused by this mismatch**:

- In the rollout stage, the recorded "probability of generating this token $\pi_{\theta_{old}}$($y_t$|$s_t$)" is computed by the **inference engine**;
- In the training stage, recomputing the "probability of this token under the current policy $\pi_{\theta}$($y_t$|$s_t$)" (used to compute ratio) is performed by the **training framework**;
- Even if $\theta$=$\theta_{old}$ (parameters are identical, such as the very first step of training), due to the implementation differences mentioned above, **there may be a slight difference between the probability computed by the inference engine and that computed by the training framework**. As a result, the calculated ratio is not strictly equal to 1 even when "parameters have not changed," introducing a **noise unrelated to "parameter updates" and caused purely by "differences in computational implementation,"** which may affect the accuracy of PPO/GRPO clipping mechanisms and training stability.

**Solutions**:

1. **Unify Computation Implementations / Use the Same Inference Backend for Probability Recomputation During Training**: Ensure as much as possible that when recomputing $\log \pi_{\theta}$($y_t$|$s_t$) during the training stage, the **same (or as close as possible) computational kernels/precision as in the rollout stage** are used, reducing numerical bias caused by implementation differences.

2. **Use Higher / Consistent Numerical Precision**: In computational steps requiring high train-inference consistency (such as calculating probabilities/logits), try to consistently use higher precision (e.g., FP32 or BF16 instead of lower-precision quantization) to minimize discrepancies caused by precision differences.

3. **Clipping / Truncation of Importance Sampling Ratios Inherently Provides Some Tolerance**: The clipping mechanism in PPO/GRPO itself is designed to "tolerate" a certain degree of policy change (including ratio estimation noise). If the numerical discrepancy caused by train-inference mismatch falls within the clipping range $\varepsilon$, its impact is relatively limited. However, if the discrepancy is large (especially accumulating over long sequences, see the motivation of GSPO in Question 433), it may still cause non-negligible training bias.

4. **Sequence-Level Importance Sampling (such as GSPO)**: As described in Questions 433--434, aggregating token-level ratios into a sequence-level geometric mean can **smooth out token-level minor numerical noise caused by train-inference mismatch**, mitigating its impact on overall training.

5. **Accept a Certain Degree of Mismatch and Provide Overall Stability Constraints via Mechanisms like KL Penalty**: Completely eliminating train-inference mismatch in engineering can be very costly (optimizations adopted by different inference engines for performance inherently differ from the design goals of training frameworks). In practice, a **certain degree of mismatch is often accepted**, relying on algorithm-level "robust designs" such as KL penalties and clipping mechanisms to absorb the impact of this noise, rather than striving for "complete elimination."

**Summary**: The train-inference mismatch problem stems from **slight numerical discrepancies produced by differences in underlying operator implementations, numerical precisions, and parallel/batching strategies between the inference engine used for rollout and the training framework used for training when computing token probabilities for the same model**. This discrepancy affects the accuracy of importance sampling ratios. Solutions include **striving to unify the computation implementation/precision between training and inference ends, utilizing sequence-level importance sampling (GSPO) to smooth token-level noise, and relying on algorithm-level robust designs like clipping mechanisms and KL penalties to tolerate a certain degree of mismatch**. It is a widely focused yet difficult to eradicate issue in current LLM-RL large-scale training engineering practice.

</details>

**37. What asynchronous RL frameworks are you familiar with?**

<details>
<summary>💡 Show Answer</summary>

To address issues such as the "long tail" in synchronous frameworks (refer to Question 451), academia and industry have proposed various asynchronous RL training frameworks/methods. Common ones include:

**1. AReaL (Asynchronous Reinforcement Learning for LLM Reasoning)**

- An asynchronous RL training framework whose core idea is to **decouple rollout (generation) and training (parameter updates) into two processes/services running in parallel**---rollout workers continuously generate samples using a (possibly slightly outdated) policy version and place them into a data buffer; training workers continuously consume samples from the buffer to update parameters. The two run asynchronously in parallel, **without needing to wait for each other**.
- Because the policy version used for rollout may lag behind the latest version in training ("staleness"), frameworks like AReaL need to address the resulting "off-policy degree" issues (such as capping staleness, applying importance sampling corrections to overly stale samples, or discarding them).

**2. Frameworks Related to Partial Rollout**

- As mentioned in Question 451, allowing **long samples to be "interrupted" during generation, letting the generated portion participate in a round of training first, while unfinished portions are "continued" in subsequent rounds** to continue generation. This "partial rollout" approach inherently possesses an asynchronous/pipelined nature, avoiding a single ultra-long sample blocking the overall workflow.

**3. Transfer of Traditional Distributed RL Architectural Concepts Based on Actor-Learner Separation**

- Drawing on architectural concepts from traditional deep RL (such as IMPALA) where "Actors (responsible for environment interaction / data generation)" and "Learners (responsible for gradient updates)" are decoupled with asynchronous communication, applying them to LLM-RL scenarios---multiple rollout workers (Actors) continuously generate and upload data, while one or more training workers (Learners) continuously consume data and update parameters, with parameters synchronized back to rollout workers periodically.

**4. In-House Asynchronous RL Systems at Tech Giants**

- Multiple institutions engaged in LLM RL training have internal asynchronous optimization solutions targeting the "long-tail problem of synchronous frameworks." Most core ideas center around the basic paradigm of **"decoupling rollout and training, introducing data buffers, and controlling staleness"**, though specific implementation details and naming may vary.

**Common challenges faced by asynchronous frameworks**:

- **Staleness Management**: A "generation gap" exists between the policy parameter version used during rollout and the latest version in training. An excessively large gap leads to an overly high "off-policy degree," affecting training correctness/stability (related to the on-policy/off-policy degree discussed in Question 424);
- **Train-Inference Mismatch (Refer to Question 452)** may be more complex in asynchronous scenarios (because parameter synchronization timing is more flexible, involving more complex "version management");
- **System Complexity**: Asynchronous architectures involve more inter-process communication, data buffer management, and parameter version synchronization mechanisms, making engineering implementation and debugging significantly more complex than in synchronous frameworks.

**Summary**: The core idea of asynchronous RL frameworks (such as AReaL) is to **decouple rollout (generation) and training (parameter updates) into components that can run in parallel and asynchronously**, connected via data buffers to avoid resource waste caused by "waiting for the entire batch of rollouts to complete" in synchronous frameworks (the long-tail problem). However, asynchrony also introduces new challenges such as **staleness management, train-inference consistency, and system complexity**, representing an active research and engineering direction in LLM-RL training framework optimization.

</details>

**38. Do AReaL or other partially rollout frameworks save the KV cache of previous policies during rollout?**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for Do AReaL or other partially rollout frameworks save the KV cache of previous policies during rollout?](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig51_do_areal_or_other_partially_ro.png)




**Handling of KV Cache in Partial Rollout**:

In partial rollout scenarios (interrupted in partial rounds and continued later), for "interrupted samples to be continued," it is indeed **necessary to save (or be able to reconstruct) the KV cache of the sample up to the interruption point**. The reasons and considerations include:

**1. Why KV cache is needed**

- The KV cache stores the **Key and Value vectors corresponding to already generated tokens** during Transformer autoregressive generation, avoiding recomputation of Key/Value vectors for all previous tokens when generating subsequent tokens;
- If the generation of a sample is "interrupted" midway (e.g., because it is ultra-long, so it participates in one round of training first while the remaining part is left for continued generation in subsequent rounds), **to "continue" generation (rather than regenerating the entire sequence from scratch)**, the KV cache of all tokens prior to the interruption point is theoretically required. Otherwise, during continuation, one must recompute the KV vectors of all previously generated tokens starting from the prompt (i.e., re-performing a full forward pass up to the interruption point), introducing extra computational overhead.

**2. Key problem in practical handling---The mapping between "policy version" and KV cache**

- The core subtlety here is: **the KV cache is computed using the model parameters (policy version) present when generating that portion of tokens**. If **policy parameters have been updated** before "continuing" generation (the training worker updated Actor parameters, which is very common in asynchronous frameworks), then:
- If one directly reuses the KV cache computed under the "old policy version" while using the "new policy version" to continue generating subsequent tokens, a situation arises where the **KV cache mismatches current model parameters**. This is essentially a special manifestation of the "train-inference mismatch" problem discussed in Question 452 under asynchronous partial rollout scenarios (here it is "parameter version mismatch" rather than "implementation difference");
- This approach of "using old KV cache + new parameters to continue generation" yields a probability distribution for subsequent tokens that **strictly speaking corresponds neither fully to 'old policy generating the full sequence' nor fully to 'new policy generating the full sequence from scratch'**, resulting in a "hybrid" state that is conceptually not entirely "clean."

**3. Possible practical approaches**

- **Option 1: Save and reuse the old KV cache** (accepting the approximation of "parameter version mismatch" mentioned above). During continuation, directly use old KV cache + current latest parameters to keep generating. **The advantage is avoiding the overhead of recomputing the prefix**, while the disadvantage is introducing approximation errors where "KV cache version does not fully match current parameter version" (the extent of this error depends on the magnitude of policy updates---if parameter changes between updates are small, the impact of this approximation is also small);
- **Option 2: Discard old KV cache and recompute the prefix KV cache using latest parameters** (i.e., "prefix recomputation"), ensuring that the KV cache perfectly corresponds to current parameter versions. **The advantage is avoiding the "parameter-KV mismatch" issue, while the disadvantage is incurring extra prefix recomputation overhead** (however, since attention computation can efficiently batch process the prefix, this overhead is usually far smaller than "generating the full sequence from scratch").

- Different partial rollout implementations may make different trade-off choices between "reusing old KV cache (accepting approximation error for efficiency)" and "recomputing prefix KV cache (ensuring consistency at the cost of extra overhead)" based on the **frequency/magnitude of policy updates**. If policy updates are very frequent (updating after every rollout), "prefix recomputation" might be preferred to ensure correctness; if policy updates are relatively infrequent, the approximation error of "reusing old KV cache" may be acceptable.

**Summary**: When handling "interrupted and then continued" samples, partial rollout frameworks **theoretically require KV caching to avoid recomputing prefixes and improve efficiency**. However, because policy parameters in asynchronous scenarios might be updated between "interruption" and "continuation," there is an inherent subtle issue: **"the saved old KV cache does not perfectly match the new policy parameter version used during continuation."** In practice, one must trade off between **"reusing old KV cache (efficiency prioritized, accepting approximation errors)"** and **"recomputing prefix KV cache with new parameters (consistency prioritized, accepting extra computation overhead)"**. The specific choice depends on the frequency of policy updates and the requirements for training correctness/stability, which is one of the detailed technical issues requiring careful consideration in the engineering implementation of asynchronous RL frameworks.

</details>

**39. In your view, between the SFT stage and the RL stage, which will be more important in the future?**

<details>
<summary>💡 Show Answer</summary>

This is an open-ended opinion question that can be answered from multiple perspectives, including current technological trends, the "role positioning" of different stages, and potential future development directions (a possible analytical framework is provided below rather than a single "correct answer"):

**Role positioning in the current stage (refer to Question 448)**:

- **SFT**: Plays more of a **"cold start / format alignment / capability activation"** role---enabling the pre-trained model to output in desired formats (e.g., dialogue format, instruction following, specific output structures) and initially acquire basic instruction-following capabilities, providing a "reasonable starting point" for the subsequent RL stage;
- **RL**: Plays a **"capability enhancement, generalization, and fine-grained alignment"** role---building upon the foundation established by SFT, using outcome-oriented optimization to let the model explore and reinforce more effective strategies (such as reasoning paths, subtle response style adjustments, and safety behavior reinforcement).

**From recent trends, the importance of RL is relatively rising**:

- Works on multiple frontier models (e.g., DeepSeek-R1) show that **large-scale, meticulously designed RL training plays a significant role in enhancing complex reasoning capabilities**. In certain settings, the contribution of the RL stage to final performance has even become comparable to or higher than that of SFT;
- As RL algorithms (GRPO and its improvements, more robust reward designs, asynchronous training frameworks) mature and training efficiency improves, "allocating more compute budget to the RL stage" becomes increasingly feasible, and overall industry resource allocation may gradually tilt toward RL.

**However, the SFT stage will not be "replaced", and its importance manifests in different dimensions**:

- **Without a good SFT starting point, RL exploration efficiency would be very low or even fail to converge**---if starting RL directly from a pre-trained model (skipping SFT), the model might struggle to satisfy even the basic requirement of "outputting in a reasonable format," making RL reward signals extremely sparse and difficult to provide effective guidance. The "good initialization" provided by SFT is a **prerequisite** for RL to proceed effectively;
- For rapid adjustments of certain "formats/styles/specific behavioral patterns" (such as the "WeChat chatting style" in Question 384), **SFT remains a more direct and efficient means**, not necessarily requiring resort to higher-cost RL;
- From the perspective of data efficiency, SFT utilizes "annotated demonstration data," where data "information density" (supervisory signal per sample) is relatively explicit. RL (especially in sparse reward scenarios) may require a massive number of rollouts to acquire sufficient effective training signals, so its "data efficiency" per compute unit is not always higher.

**A comprehensive perspective**:

- "Which is more important" may not be a "binary choice" question, but rather **that their roles and "modes of expressing importance" differ, and they will continue to coexist and co-evolve**---SFT is responsible for "laying the foundation / aligning basic format and behavioral frameworks," while RL is responsible for "exploring and reinforcing higher-level capabilities and alignment on top of that foundation";
- As RL technology (algorithms, frameworks, reward designs) develops, **the future RL stage may assume more and more capability enhancements that "originally relied on SFT + large amounts of annotated data"** (especially capabilities hard to exhaust via "annotated demonstrations" that require "exploration" to discover, such as complex reasoning and long-horizon planning). From this perspective, **the importance of RL in "pushing the capability ceiling" may continue to grow**;
- However, SFT's efficiency advantage in "rapidly and low-costly achieving specific behavioral alignment (format, style, basic safety guidelines)" is difficult to be completely replaced by RL in the short term. **The two are more likely in a "complementary and co-evolving" relationship rather than a zero-sum substitution relationship**.

**Summary** (presented as an opinion with room for discussion): From the perspective of "capability enhancement potential and ceiling," **RL (especially large-scale RL for complex reasoning / long-horizon tasks) may play an increasingly important role in the future**, serving as a key direction of investment in many frontier studies. However, SFT's foundational role as an "efficient, low-cost means of format alignment and capability activation" and as a "good starting point for RL" will not be diminished in the short term. The two are more likely in a relationship of **division of labor and co-evolution** rather than a simple replacement of "which is more important." The specific proportion of resource investment will dynamically adjust with technological advances and specific task demands.

</details>

**40. Describe a process where you discovered a problem, analyzed it, and solved it during RL training**

<details>
<summary>💡 Show Answer</summary>

![Dynamic Visualization for Describe a process where you discovered a problem, analyzed it, and solved it during RL training](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig52_describe_a_process_where_you_d.png)




This is an open question assessing practical experience and troubleshooting methodologies. Below is a **typical and representative troubleshooting case** provided as a reference framework (connecting multiple knowledge points previously covered in this chapter):

**Problem Symptom (Discovering the Problem)**:

During RL training using GRPO on mathematical reasoning tasks (outcome reward = answer correctness), it was observed that after a number of training steps, the **reward curve continued to rise and reached saturation** (nearing the upper bound). However, the **length of responses generated by the model increased dramatically** (far exceeding the average length at the start of training), and the **accuracy on an independent validation set (held-out math problems) did not increase synchronously with training reward, and even showed a slight decrease**.

**Analyzing the Problem**:

1. **Step 1: Investigate the "disconnect" between reward curve and validation set performance**---Following the ideas of Questions 436 and 447, since validation accuracy failed to keep up with the increase in training reward, it was suspected that some form of "overfitting/gaming on the training distribution" was occurring.

2. **Step 2: Observe specific changes in output content**---Sampling responses generated during training revealed that the model tended to produce **extremely verbose content containing repetitive "self-checking" / "trying another method"** phrases. In many cases, the model would **try multiple different problem-solving methods; as long as one method yielded the correct final answer, it received a reward** (since the reward only looked at the "final answer," unconcerned with process or length).

3. **Step 3: Locate the root cause---Implicit correlation between length and reward**---Analysis revealed that for difficult problems, "generating longer outputs and attempting more methods" objectively **increased the probability that at least one method got the correct answer**. Thus, a **real yet potentially "excessive" positive correlation** existed between "reward" and "length"---the learned "policy" was "regardless of problem difficulty, try to generate as long as possible and attempt as many methods as possible." This indeed improved rewards on training distribution problems (which might be of moderate overall difficulty), but:
- For **relatively simple problems** in the validation set, such "excessively verbose multi-method attempts" could instead introduce **additional opportunities for error** (e.g., during multiple attempts, an intermediate result that appeared "better" but was actually wrong interfered with the model's final decision/output);
- Meanwhile, overly long outputs could also be **truncated due to approaching/exceeding maximum generation length limits**, causing responses that were originally "correct in thought process but had not yet written out the final answer" to be judged as "no answer provided," receiving negative feedback (related to the "reward shaping for ultra-long samples" in DAPO, Question 432).

**Solving the Problem**:

1. **Introduce Length-Related Reward Adjustments**---Following the ideas in Questions 418 and 431, add a **slight penalty term for response length** to the reward (e.g., giving incremental penalties when length exceeds a reasonable threshold) to suppress the "gaming" tendency of "mindlessly stacking length / number of method attempts," guiding the model to learn a "more efficient and targeted" reasoning process rather than a "shotgun approach" of multi-method trials.

2. **Check and Adjust Handling of Ultra-Long Samples**---Following DAPO's "reward shaping for ultra-long samples" concept, for samples truncated due to reaching the maximum length, **do not directly assign strong negative rewards** (preventing the model from overly associating "ultra-long" itself with "error," which generates unnecessary "fear" of length and could lead to another extreme of premature termination); simultaneously **appropriately increase maximum generation length limits** (if hardware resources permit) to reduce instances of "correct thought process but failing to write out final answer due to length constraints."

3. **Re-evaluate and Iterate**---After applying the above adjustments, re-run training and **continuously monitor the relationship among the reward curve, response length distribution, and validation set accuracy**---expecting to see that the reward increase is accompanied by a synchronized (or at least non-disconnected) increase in validation set accuracy, with response lengths stabilizing within a "reasonable" range (no longer growing infinitely), rather than merely having "nice-looking reward numbers without actual capability improvement."

4. **Further Preventive Measures**---Establish the issue of "implicit correlation between length and reward" discovered here as a **standard checkpoint for subsequent RL training**---that is, in future similar RL training runs, **by default monitor the correlation between 'reward' and 'output length distribution' simultaneously** as a common, typical early-warning signal for reward hacking (refer to the diagnostic method in Question 436).

**Summary**: This case demonstrates a typical RL training troubleshooting workflow---**"Training metric (reward) disconnects from actual goal (validation accuracy)" $\rightarrow$ "Sample and analyze generated content to locate specific forms of 'gaming' (verbosity, multi-method attempts)" $\rightarrow$ "Analyze the implicit association between this 'gaming' behavior and reward design (positive correlation between length and probability of 'guessing right at least once')" $\rightarrow$ "Targeted adjustment of reward design (length penalties, reward shaping for ultra-long samples)" $\rightarrow$ "Retrain and continuously monitor multiple related metrics to verify problem resolution, establishing the issue pattern as a routine checkpoint for subsequent training"**---This cycle of "discovering disconnect $\rightarrow$ sampling analysis to locate mechanism $\rightarrow$ targeted adjustment of reward/training settings $\rightarrow$ verifying and consolidating experience" is a representative paradigm for troubleshooting and iterating in RL training (especially reward design related issues).

</details>

