# 5.7 GRPO and Other RL Variants — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 5: Reinforcement Learning and Post-Training Alignment (RLHF / DPO) · 25 questions

**1. Briefly explain the GRPO algorithm?**

<details>
<summary>💡 Show Answer</summary>

![GRPO (Group Relative Policy Optimization) Workflow](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig33_briefly_explain_the_grpo_algor.png)




GRPO (Group Relative Policy Optimization) is a simplified and improved version of PPO proposed by DeepSeek. Its core innovation is **replacing the Critic model with group relative reward normalization** to estimate the advantage function, maintaining policy optimization effectiveness while reducing training complexity.

**Core Idea**:

For each prompt $x$, sample $G$ responses $\{o_1, o_2, ..., o_G\}$ using the current policy, and obtain reward scores $\{r_1, r_2, ..., r_G\}$ for each response. Normalize the intra-group rewards as the advantage estimate for each response:
\[A_i = \frac{r_i - mean(r_1,...,r_G)}{std(r_1,...,r_G)}\]
The intra-group mean plays the role of the Critic (baseline) in PPO, avoiding the need to maintain and train an extra Critic model.

**Key Differences Compared to PPO**:
- **Elimination of Critic**: Memory usage is reduced from 4 models to 2--3 models, simplifying the training pipeline;
- **Simpler Advantage Estimation**: Eliminates GAE, using intra-group relative rewards directly;
- **Requires Multiple Sampling per Prompt**: Each prompt requires sampling $G$ responses (typically $G=8--64$), increasing rollout costs;
- **Particularly Suited for Reasoning Tasks**: When objective, verifiable rewards (math, code) are available, intra-group relative comparison signals are clear and effective.

The KL constraint in GRPO is implemented by directly adding a token-level KL divergence penalty term to the loss function, rather than subtracting the KL penalty from the reward as in PPO; both approaches yield similar empirical results in practice.

</details>

**2. What is the Objective Function of the GRPO Algorithm?**

<details>
<summary>💡 Show Answer</summary>

The objective function of GRPO inherits the clipping mechanism of PPO while integrating intra-group normalized advantage estimation and KL divergence penalty:

**Full Objective Function**:

For a prompt $q$ and $G$ outputs $\{o_i\}_{i=1}^G$ sampled from the current policy:

\[
\mathcal{L}_{GRPO}(\theta) = -\mathbb{E}_{q, \{o_i\}} \left[\frac{1}{G}\sum_{i=1}^G \frac{1}{|o_i|}\sum_{t=1}^{|o_i|} \left(\min\left(ratio_{i,t} A_i,\ clip(ratio_{i,t}, 1-\varepsilon, 1+\varepsilon) A_i\right) - \beta \cdot KL_{i,t}\right)\right]
\]

**Symbol Meanings**:
- $ratio_{i,t} = \pi_\theta(o_{i,t}|q, o_{i,<t}) / \pi_{\theta_old}(o_{i,t}|q, o_{i,<t})$: Importance sampling ratio for the $t$-th token of the $i$-th response;
- $A_i = (r_i - \mu_r) / \sigma_r$: Intra-group normalized advantage estimate ($\mu_r$ and $\sigma_r$ are the mean and standard deviation of the rewards for the $G$ responses, respectively);
- $KL_{i,t}$: Token-level KL divergence between the current policy and the reference policy, commonly estimated in practice using the k3 estimator: $\pi_ref/\pi_\theta - \log(\pi_ref/\pi_\theta) - 1$ (ensuring non-negativity and numerical stability);
- $\varepsilon$: Clip range, typically 0.2;
- $\beta$: KL penalty coefficient, typically 0.01--0.1.

**Design Intent of the Objective Function**:
1. **Clipping Mechanism**: Prevents excessively aggressive policy updates to ensure training stability;
2. **KL Penalty**: Prevents the policy from deviating too far from the reference model, guarding against catastrophic forgetting;
3. **Intra-group Normalization**: Replaces absolute advantages with intra-group relative rewards, eliminating differences in reward scales across different prompts and reducing training variance.

</details>

**3. Explain the basic principle of GRPO?**

<details>
<summary>💡 Show Answer</summary>

The basic principle of GRPO is to replace the value estimation model in PPO through intra-group normalization, thereby reducing computational resource demands. The specific operational steps are as follows:
1. **Sample Grouping**: For each question, generate $G$ different outputs, then calculate the mean and standard deviation of the reward values of these outputs.
2. **Advantage Evaluation**: For each output, calculate its advantage value relative to the group average reward: $A_i = \frac{r_i - mean(r)}{std(r)}$.
3. **Policy Adjustment**: Perform policy optimization by maximizing the product of the clipped policy ratio and the advantage value.

The main differences between GRPO and PPO are: no value model is required, reducing the parameter count and memory consumption needed during training; intra-group competition, where advantage calculation is based on relative performance within the group, making it better suited for sparse rewards.

</details>

**4. What are the differences among GRPO, PPO, and DPO?**

<details>
<summary>💡 Show Answer</summary>

![Comparison of GRPO, PPO, and DPO Structurally](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig34_what_are_the_differences_among.png)




**GRPO (Group Relative Policy Optimization)** is an RL algorithm proposed by DeepSeek, which can be understood as a simplified improvement over PPO. The relationships and differences among the three are:

**1. Whether a Critic (Value Model) is Required**

- **PPO**: Requires training an independent Critic (value model) to estimate $V(s)$ for computing the advantage function (GAE);
- **GRPO**: **Eliminates the Critic**. For the same prompt, it samples **a group of multiple responses** (e.g., $G$ responses), normalizes the reward of each response using the **mean and standard deviation of rewards** in this group, and uses it directly as the response's "advantage"—i.e.:

\[
A_i = (r_i - mean({r_1,...,r_G})) / std({r_1,...,r_G})
\]

This "intra-group relative comparison" replaces the "baseline of value function estimation", thereby eliminating the Critic model and significantly reducing GPU memory and computational overhead;
- **DPO**: Completely avoids value models, online sampling, and reward models, performing offline contrastive learning.

**2. Whether Online Sampling (On-policy) is Required**

- **PPO and GRPO**: Both are **online (on-policy) reinforcement learning** methods—requiring real-time sample generation (rollouts) using the current policy model, followed by scoring with a reward model/rules for iterative optimization;
- **DPO**: Is an **offline** method—training directly on pre-collected preference datasets ($x$, $y_w$, $y_l$), without requiring model generation during training.

**3. Sources and Forms of Reward**

- **PPO**: Typically relies on a learned reward model (neural network) to score generated content with a scalar value;
- **GRPO**: Rewards can come from a reward model or be **rule-based rewards** (such as whether a math answer is correct or code passes tests). In the DeepSeek model series, GRPO is often combined with rule-based rewards to train reasoning capabilities;
- **DPO**: Requires no explicit reward; the reward is "implicitly" reflected in the log probability ratio between the policy and reference models (refer to Question 418).

**4. Number of Models Involved in Training**

- **PPO**: 4 models (Actor, Critic, Reward Model, Reference Model);
- **GRPO**: 3 models (Actor, Reward Model [if using a neural network RM; not needed if using rule-based rewards], Reference Model)—eliminating the Critic;
- **DPO**: 2 models (Policy Model, Reference Model).

**5. Training Stability and Sample Efficiency**

- As online methods, PPO and GRPO have more complex training dynamics, but they are capable of "exploring" superior outputs beyond the current policy distribution;
- DPO training is more stable (close to supervised learning), but it is limited to the "preference directions" reflected in the pre-collected preference data and cannot explore new output patterns during training that were not covered in the dataset.

**Summary**:
- PPO is a classic online RL method requiring a Critic to estimate the value function;
- GRPO simplifies PPO by replacing the Critic with "intra-group relative reward normalization", while remaining an online method that is often paired with rule-based rewards for reasoning training;
- DPO is a fully offline contrastive learning method that requires no online sampling, value models, or reward models (trained on existing preference data).
The three are progressively simplified along dimensions such as "whether a Critic is needed", "online vs. offline", "reward sources", and "number of models involved".

</details>

**5. Are you familiar with DeepSeek's GRPO? What are its main differences from PPO, and what are its pros and cons?**

<details>
<summary>💡 Show Answer</summary>

GRPO (Group Relative Policy Optimization) is the core RL algorithm adopted by DeepSeek when training DeepSeek-R1, serving as an important improved variant of PPO.

**Core Difference: Alternative Way of Replacing the Critic**



| **Dimension** | **PPO** | **GRPO** |
| --- | --- | --- |
| Number of Models | 4 (Actor+Critic+RM+Ref) | 2--3 (Actor+Ref+Optional RM) |
| Advantage Estimation | Critic Network + GAE | Intra-group Reward Normalization |
| Samples per Prompt | 1 (typically) | G (G=8~64) |
| Baseline | Critic Prediction Value | Intra-group Reward Mean |
| KL Constraint | Subtract KL Penalty from Reward | Add KL Penalty Term to Loss |



**Advantages of GRPO**:
- **GPU Memory Friendly**: Eliminates the Critic, with each model comparable in size to the Actor, saving approximately 1/4 of total GPU memory;
- **Simpler Engineering**: No need to maintain and train a Critic network, eliminating the issue of the Critic lagging behind Actor learning;
- **More Intuitive Advantage Estimation**: Intra-group relative reward normalization directly reflects "which response is better under the same prompt", which is especially effective for tasks with objective verification rewards;
- **Well-Suited for Reasoning Tasks**: DeepSeek-R1 practices demonstrate that GRPO outperforms PPO in mathematical reasoning.

**Disadvantages of GRPO**:
- **Higher Rollout Cost**: Each prompt requires sampling $G$ responses (typically 8--64), resulting in total rollout computation that is $G$ times higher than PPO;
- **All-Right / All-Wrong Issue**: If all $G$ responses for the same prompt receive identical rewards (e.g., all correct or all wrong), the intra-group standard deviation is 0, making it impossible to calculate a meaningful advantage estimate and wasting that batch of data (the dynamic sampling mechanism of DAPO is specifically designed for this);
- **Weaker Adaptability to Open-Ended Tasks**: For open-ended tasks without objective reward signals, the quality of intra-group relative comparison signals relies on reward model accuracy, making its performance less stable than PPO.

</details>

**6. Differences Between GRPO and PPO; How Advantage Normalization Solves Value Function Estimation Issues**

<details>
<summary>💡 Show Answer</summary>

**Core Differences Between GRPO and PPO**:
- **Whether a Critic is Needed**: PPO requires training a value network of comparable scale to the policy model; GRPO completely eliminates the Critic.
- **Advantage Computation Method**: PPO uses $Advantage = Reward - V(s)$; GRPO samples a group of responses for the same prompt and normalizes using the mean and standard deviation of rewards within the group: $Advantage_i = (R_i - mean(R_{group})) / std(R_{group})$.

**How Advantage Normalization Solves the Value Function Estimation Issue**:

In PPO, the training objective of the value network depends on the policy itself (non-stationary target), and the estimation error of the value network directly contaminates the computation of the advantage. GRPO completely bypasses the difficulty of value function estimation through "intra-group relative comparison"—without needing to predict "how many points this response is expected to get", but only needing to know "how it performs relatively within this group of responses", making it inherently more robust.

</details>

**7. Are GRPO and PPO On-Policy or Off-Policy? What About DPO?**

<details>
<summary>💡 Show Answer</summary>

![Process Reward Model (PRM) Step-by-Step Evaluation](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig35_are_grpo_and_ppo_on_policy_or.png)




**Definition of On-Policy vs. Off-Policy**:
- **On-policy**: The data used to update the policy must be generated by the **current (or near-current) policy being optimized**;
- **Off-policy**: Data generated by **other policies (such as historical policy versions or other models)** can be used to update the current policy.

**PPO**: **On-policy (strictly speaking, "near-on-policy")**

- The standard pipeline of PPO is: use the current policy $\pi_{\theta_{old}}$ to generate a batch of samples (rollout), and then use this batch of samples to update the policy for **several mini-batch passes (typically 2--4 epochs)**;
- Since the policy changes during these updates (a divergence arises between $\pi_{\theta}$ and $\pi_{\theta_{old}}$), strictly speaking, the data used in subsequent updates is no longer "completely on-policy" (data was generated by $\pi_{\theta_{old}}$, but the current policy has become $\pi_{\theta}$);
- However, the clipping mechanism in PPO is specifically designed to **limit this "degree of off-policy"** to ensure that the difference between $\pi_{\theta}$ and $\pi_{\theta_{old}}$ does not grow too large, making the update to $\pi_{\theta}$ using data generated by $\pi_{\theta_{old}}$ remain a "reasonable approximation"---a property sometimes referred to as **"near-on-policy"**.

**GRPO**: Similar to PPO, **On-policy (near-on-policy)**

- GRPO similarly samples a set of responses (rollouts) using the current policy, calculates advantages based on rewards from these responses, and updates the policy. The overall "online" nature of the pipeline is consistent with PPO;
- Similarly, because multiple update steps may be performed after a single rollout, it also exhibits a "near-on-policy" property similar to PPO, where KL penalty and clipping mechanisms (if used) play a similar constraining role.

**DPO**: **Off-policy**

- The preference data ($x, y_w, y_l$) used for DPO training is typically **pre-collected**---these responses may be written by an initial SFT model, other models, or even humans, and have no direct relationship with the "current policy $\pi_{\theta}$ being optimized";
- **No online sampling is performed** during training, and optimization is carried out entirely on this fixed, pre-prepared dataset. Thus, it is a classic **off-policy (or even completely offline) method**.

**Supplementary---Spectrum of "Degree of On-Policy"**:
- Fully on-policy: Update the policy immediately after generating each single sample, and then generate the next sample using the new policy (theoretically purest, but extremely inefficient);
- PPO/GRPO: Batch sampling + multiple rounds of mini-batch updates, serving as "near-on-policy", striking a balance between efficiency and on-policy behavior;
- DPO: Entirely based on historical/pre-prepared data, with the highest degree of off-policy behavior (the policy at data generation time may differ significantly from the current policy).

**Summary**: **Both PPO and GRPO are (near-)on-policy methods**---they rely on samples generated by the current policy (or the policy right before updates) for optimization, and control the degree of "approximate on-policy" via clipping/KL constraints; **DPO is an off-policy (offline) method**, directly training on pre-collected preference data that may be independent of the current policy distribution.

</details>

**8. Why Does GRPO Add KL Divergence, and Is the KL Divergence Used Forward or Reverse?**

<details>
<summary>💡 Show Answer</summary>

**Why GRPO adds KL divergence**:

Aligned with the motivation of KL penalty in PPO (refer to Questions 398 and 411), GRPO adds the KL divergence between the policy model $\pi_{\theta}$ and the reference model $\pi_{ref}$ to the optimization objective, serving the following roles:
- **Constrain the policy model from drifting too far from the reference model (typically the SFT model)**, preventing catastrophic forgetting and maintaining the model's general capabilities;
- **Prevent reward hacking / over-optimization**: In the absence of a Critic to provide a baseline constraint (as GRPO removes the Critic), the KL penalty becomes an essential mechanism for constraining policy update magnitudes and preventing the policy from "going to extremes."

**Forward KL vs. Reverse KL**:

GRPO (as well as common implementations of PPO in LLM scenarios) uses **KL($\pi_{\theta}$ || $\pi_{ref}$)**, i.e.:

\[
KL(\pi_{\theta} || \pi_{ref}) = E_{y~\pi_{\theta}}[log(\pi_{\theta}(y|x)/\pi_{ref}(y|x))]
\]

This is the KL divergence with the **current policy $\pi_{\theta}$ as the sampling distribution**, which is commonly referred to in statistical learning as the **"reverse KL" relative to $\pi_{ref}$ (since if $\pi_{ref}$ is regarded as the "true distribution" and $\pi_{\theta}$ as the "approximate distribution", $KL(approximate \parallel true)$ is typically termed reverse KL, corresponding to "mode-seeking" behavior)**.

(Note: The naming convention for forward/reverse KL may vary across literature; here we adopt the more common convention where taking the "distribution being optimized" as the first argument in $KL(\cdot \parallel \cdot)$ designates this direction of KL.)

In the GRPO paper and most LLM-RL implementations, the specific form of the KL penalty term typically adopts the following **unbiased estimator (k3 estimator)**:

\[
KL_term = \pi_{ref}(y|x)/\pi_{\theta}(y|x) - log(\pi_{ref}(y|x)/\pi_{\theta}(y|x)) - 1
\]

This is an implementation of $KL(\pi_{\theta} \parallel \pi_{ref})$, which exhibits better numerical stability in practice (always non-negative with lower variance).

**Characteristics of Reverse KL (mode-seeking)**: When optimizing using $KL(\pi_{\theta} \parallel \pi_{ref})$, since the expectation is taken under $\pi_{\theta}$, the optimization process tends to concentrate $\pi_{\theta}$ in regions where $\pi_{ref}$ has high probability (i.e., "mode-seeking"). This aligns with the requirement in RLHF scenarios: "ensuring the policy model does not drift too far from the reference model's main behavioral modes, while still allowing adjustments within its support to improve rewards."

**Summary**: The purpose of adding KL divergence in GRPO is consistent with PPO---**constraining the policy from drifting too far from the reference model, preventing catastrophic forgetting and over-optimization (reward hacking)**; it uses **KL($\pi_{\theta}$||$\pi_{ref}$)** (the "reverse KL" with the current policy as the sampling distribution, possessing mode-seeking properties), and in practice, unbiased and low-variance formulations such as the k3 estimator are commonly adopted.

</details>

**9. Role of the KL Penalty Term in GRPO; Impact of Being Too Large or Too Small**

<details>
<summary>💡 Show Answer</summary>

GRPO adds a KL divergence penalty term ($\beta \cdot KL(\pi_\theta || \pi_ref)$) to the objective function, consistent with the KL penalty mechanism in PPO, though with a slightly different implementation method.

**Role of the KL penalty term**:
- **Prevent the policy from drifting too far from the reference model**: Restrict the KL divergence between the current policy $\pi_\theta$ and reference policy $\pi_ref$ (typically the SFT model), preventing the model from "forgetting" the general language capabilities learned during pre-training and SFT stages;
- **Prevent Reward Hacking**: GRPO removes the Critic, so the only constraint on advantage estimation comes from group normalization. Without the KL penalty, the policy could quickly converge to exploiting flaws in the reward model;
- **Information Regularization**: From an information-theoretic perspective, the KL penalty limits the "amount of information" introduced per update, making policy updates more robust.

**$\beta$ being too large**:
- The policy is overly constrained and cannot learn new reasoning patterns from reward signals;
- The KL penalty dominates the loss, reducing policy updates to near zero, which is equivalent to not performing RL;
- Training curves show almost no improvement in reward, with KL divergence remaining close to 0 long-term.

**$\beta$ being too small**:
- The policy can drift freely from the reference model; short-term rewards rise rapidly, but issues such as degraded output readability, increased hallucinations, and language degeneration may occur;
- KL divergence increases drastically, potentially triggering catastrophic forgetting;
- The risk of Reward Hacking increases significantly, as the model finds "shortcuts" in the reward model rather than genuinely learning reasoning.

**Practical Recommendations**: Typically set $\beta \in [0.01, 0.1]$, while simultaneously monitoring the absolute value of KL divergence (recommended to control within 0.05--0.3 nats) and dynamically adjusting based on training curves.

</details>

**10. Disregarding CPU Offloading, How Many Models Are in VRAM During GRPO Training, and What Are They?**

<details>
<summary>💡 Show Answer</summary>

![VRAM Allocation Comparison: GRPO vs. PPO](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig36_disregarding_cpu_offloading_ho.png)




Compared with PPO, GRPO removes the Critic (value model). The number of models required in VRAM is **3** (if using a neural network reward model) or **2** (if using rule-based rewards, requiring no neural network reward model):

1. **Actor (Policy Model)**: Requires training, maintaining parameters, gradients, and optimizer states;
2. **Reference Model**: Parameters are frozen, used only for forward inference to calculate KL divergence;
3. **Reward Model (if applicable)**: Parameters are frozen, used only for forward inference to score generated outputs; if rewards are entirely rule-based (e.g., math answer correctness checks, code execution results), **this model is not needed**.

**Compared to PPO (4 models: Actor, Critic, Reward Model, Reference Model), GRPO removes the Critic, reducing the model count by 1**---this is one of the primary sources of VRAM efficiency gain in GRPO compared to PPO (the Critic is typically similar in scale to the Actor, so eliminating it saves VRAM close to "one full model + its optimizer states").

**Summary**: GRPO VRAM typically contains **Actor (training) + Reference Model (frozen) + Reward Model (frozen, optional if using rule-based rewards)**. Having one fewer model (Critic) compared to PPO is one of the core manifestations of GRPO being "lighter weight."

</details>

**11. Same as Above, What About PPO? What About DPO?**

<details>
<summary>💡 Show Answer</summary>

**PPO**: Requires **4 models** in VRAM:

1. **Actor (Policy Model)**: Requires training (parameters + gradients + optimizer states);
2. **Critic (Value Model)**: Requires training (parameters + gradients + optimizer states);
3. **Reward Model**: Parameters are frozen, forward inference only;
4. **Reference Model**: Parameters are frozen, forward inference only.

Among them, Actor and Critic need to maintain full training states (gradients + optimizer states, which for Adam is typically several times the parameter count), serving as the primary source of VRAM overhead; Reward Model and Reference Model only require forward inference, incurring relatively smaller overhead (though still needing complete model parameters loaded).

**DPO**: Requires **2 models** in VRAM:

1. **Policy Model $\pi_{\theta}$**: Requires training (parameters + gradients + optimizer states);
2. **Reference Model $\pi_{ref}$**: Parameters are frozen, forward inference only (used to compute log probabilities of the $\pi_{ref}$ component in DPO loss).

DPO does not require a Critic or Reward Model, making its VRAM requirement the lowest among the options. **Further Optimization**: Since the parameters of $\pi_{ref}$ remain unchanged throughout training, its output $\log \pi_{ref}(y|x)$ on the training data (a fixed preference dataset) can theoretically be **pre-computed and cached**. During training, cached values are read directly, entirely eliminating the need to load the $\pi_{ref}$ model into VRAM during the training phase (further reducing VRAM demands). This is an additional engineering advantage of DPO over online RL methods.

**Summary**: Model count comparison---**PPO: 4 (Actor training + Critic training + RM frozen + Ref frozen); GRPO: 2--3 (Actor training + Ref frozen + optional RM frozen); DPO: 2 (Policy training + Ref frozen, with Ref further subtractable via pre-computation caching)**, with VRAM requirements decreasing in order. This is a key reason why GRPO and DPO are more "engineering-friendly" and resource-efficient compared to PPO.

</details>

**12. How Gradients Backpropagate During GRPO Training**

<details>
<summary>💡 Show Answer</summary>

The general form of the GRPO loss function (for a group of $G$ responses to a single prompt) is approximately:

\[
L_GRPO(\theta) = -E_{q,{o_i}}[ (1/G) \Sigma_{i=1}^G (1/|o_i|) \Sigma_{t=1}^{|o_i|} ( min(ratio_{i,t}·A_i, clip(ratio_{i,t},1-\varepsilon,1+\varepsilon)·A_i) - \beta·KL_{i,t} ) ]
\]

The specific process of gradient backpropagation:

**1. Forward Computation**

- For each token $y_{i,t}$ in each sampled response $o_i$, compute its log probability $\log \pi_{\theta}(y_{i,t}|s_{i,t})$ and $\log \pi_{ref}(y_{i,t}|s_{i,t})$ using the current policy $\pi_{\theta}$ and reference model $\pi_{ref}$ (as well as the old policy $\pi_{\theta_{old}}$ used during generation);
- Compute $ratio_{i,t} = \pi_{\theta}(y_{i,t}|s_{i,t}) / \pi_{\theta_{old}}(y_{i,t}|s_{i,t})$ (the probabilities of the old policy are recorded during sampling and treated as constants, not participating in gradient computation);
- Compute the group-normalized advantage $A_i$ (a scalar shared across all tokens in that response, calculated based on the final reward of the entire trajectory);
- Compute $KL_{i,t}$ (based on log probabilities of $\pi_{\theta}$ and $\pi_{ref}$; refer to Questions 422 and 423).

**2. Loss Computation**

- For each token position, compute the term $\min(ratio \cdot A, clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A) - \beta \cdot KL$;
- Average across all tokens within a response (normalized by token count), and then average across a group of $G$ responses to obtain the loss for that prompt (further averaged/summed over multiple prompts in a batch).

**3. Backpropagation**

- **Gradients backpropagate exclusively through the forward computation path of $\pi_{\theta}$ (current policy model)**---$\pi_{\theta_{old}}$ (policy used during rollout generation, treated as a constant) and $\pi_{ref}$ (reference model, parameters frozen) do not participate in gradient updates, and their outputs are treated as constants during backpropagation;
- Specifically:
- In **$ratio_{i,t} = \pi_{\theta}(y_{i,t}|s_{i,t}) / \pi_{\theta_{old}}(y_{i,t}|s_{i,t})$**, the numerator $\pi_{\theta}(\cdot)$ is a function of $\theta$ and requires gradient computation; the denominator $\pi_{\theta_{old}}(\cdot)$ is a constant;
- **Clipping operation**: When $ratio$ falls within $[1-\varepsilon, 1+\varepsilon]$, $clip(ratio, \cdot, \cdot) = ratio$, and gradients pass through normally; when $ratio$ exceeds this interval and gets clipped, the output of $clip(\cdot)$ becomes a constant independent of $\theta$ ($1-\varepsilon$ or $1+\varepsilon$). At this point, **the gradient of this term with respect to $\theta$ at that token position is 0** (if the $\min$ operation selects the clipped term; refer to Questions 428 and 429);
- In **$KL_{i,t}$**, the $\pi_{\theta}(\cdot)$ component requires gradient computation, whereas $\pi_{ref}(\cdot)$ is a constant;
- **$A_i$** is a scalar computed from the "final reward of the entire trajectory" (after group normalization), treated during backpropagation as a **constant weight identical across all token positions in that trajectory** (no gradients are calculated for $A_i$ itself, as rewards are not differentiable functions of $\theta$---rewards come from reward model/rule outputs and do not participate in end-to-end gradient propagation).

- Ultimately, the gradient of the loss function with respect to $\theta$ is essentially a weighted sum of the **"gradient of $\log \pi_{\theta}(y_{i,t}|s_{i,t})$ with respect to $\theta$"** (a standard language model token-level gradient, computable via standard backpropagation/automatic differentiation), multiplied by the corresponding ($\theta$-independent) "weight coefficient" (determined jointly by $ratio$, $A_i$, clipping state, and KL term).

**Summary**: GRPO gradient backpropagation is essentially: **for each generated token position, taking the gradient of $\log \pi_{\theta}(y_t|s_t)$ (standard language model token-level log probability, computable via standard Transformer forward pass + automatic differentiation)**, multiplying by a scalar weight determined jointly by the "group-normalized advantage $A_i$, PPO-style clipped ratio gradient factor, and gradient of the KL penalty term with respect to $\theta$," and then summing/averaging over all tokens and all responses. Outputs of $\pi_{\theta_{old}}$ and $\pi_{ref}$ serve throughout as **constants that do not participate in gradient computation**.

</details>

**13. Why Do Both GRPO and PPO Take a Min over rA and clip(1-e, 1+e, r)A, Instead of Directly Using clip(1-e, 1+e, r)A?**

<details>
<summary>💡 Show Answer</summary>

![Pesticide Function of Min in Clipped Surrogate Loss](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig37_why_do_both_grpo_and_ppo_take.png)




**The problem with directly using $clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A$**:

If only the clipped term $clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A$ is used as the objective, the following problems arise:

Consider the case where $A > 0$ (the action is a "good" action, so its probability should be increased, i.e., we want $ratio$ to increase):
- If $ratio$ has already exceeded $1+\varepsilon$ (i.e., relative to the old policy, the probability of this action has increased by more than proportion $\varepsilon$), after clipping $clip(ratio, \cdot, \cdot) = 1+\varepsilon$ (constant);
- In this scenario, if **only the clipped value is used**, the gradient of the objective function with respect to $\theta$ becomes 0 (since the clipped value is constant)---this seems "reasonable" because we indeed want to limit update magnitude;
- However, consider another case: if current $ratio < 1$ (i.e., the update direction is **decreasing** the probability of this "good action", which is **opposite** to the objective of "$A > 0$ should increase probability!"), and after clipping $clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A$ in certain situations may still yield a gradient that **continues updating in the wrong direction** (if looking only at the clipped value without comparing to the unclipped value, it may fail to correctly identify whether the current update direction is correct).

**Role of the min operation---constructing a "pessimistic lower bound"**:

The objective function $\min(ratio \cdot A, clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A)$ in PPO/GRPO is intended to construct a **pessimistic lower bound** of the unclipped objective $ratio \cdot A$. Its specific behavior is as follows:

- **When $A > 0$** (good action, wishing to increase probability, i.e., hoping $ratio \uparrow$):
- If $ratio$ is within $[1-\varepsilon, 1+\varepsilon]$: $\min(ratio \cdot A, clip \cdot A) = ratio \cdot A$ (the two are equal, normal update);
- If $ratio > 1+\varepsilon$ (update has "gone too far", probability increase exceeds the limit): $clip \cdot A = (1+\varepsilon) \cdot A < ratio \cdot A$, $\min$ takes $clip \cdot A$, **where the gradient becomes 0 (no longer continuing to increase the action's probability)**---correctly "applying the brakes";
- If $ratio < 1-\varepsilon$ (i.e., despite $A > 0$, the current update is **decreasing** the action's probability, a "reverse" update that shouldn't happen): $clip \cdot A = (1-\varepsilon) \cdot A$, whereas $ratio \cdot A < (1-\varepsilon) \cdot A$ (since $ratio < 1-\varepsilon$ and $A > 0$), so $\min$ takes **$ratio \cdot A$ (unclipped value)**, gradient is non-zero, and **the objective function continues to penalize this "reverse" update**, prompting $ratio$ to recover---this is the critical value of the $\min$ operation: **for cases of "moving too far in the wrong direction," clipping is not applied, allowing gradients to keep acting to correct updates in the wrong direction**.

- **When $A < 0$** (bad action, wishing to decrease probability, i.e., hoping $ratio \downarrow$), symmetrical analysis yields the same conclusion: the $\min$ operation ensures that "if current update moves too far in the wrong direction (increasing bad action probability)", gradients remain effective and can correct it back; only when "moving in the correct direction and reaching the limit" is the gradient zeroed out.

**Intuitive Summary**: **$clip$ itself merely "limits the range of change", but does not distinguish "whether the direction of change is correct"**; **the role of the $\min$ operation is: updates are stopped (clipping takes effect) only when "the update magnitude in the correct direction exceeds the limit", whereas "updates in the wrong direction" are never shielded by clipping regardless of magnitude (i.e., correction signals are always retained)**. This ensures that the objective function $\min(\dots)$ remains a **lower bound** (pessimistic estimate) of the original objective $ratio \cdot A$, rendering the optimization process "conservative" without "leaving wrong-direction updates uncorrected", thereby achieving the more reasonable behavior of "bounding good update magnitudes while unbounding corrections for bad update directions."

**Summary**: Using $clip(ratio, \cdot, \cdot) \cdot A$ alone cannot distinguish "whether the current update direction is correct", potentially zeroing out gradients mistakenly even when "the update direction is wrong" (allowing errors to persist); whereas **$\min(ratio \cdot A, clip(ratio, \cdot, \cdot) \cdot A)$ constructs a pessimistic lower bound of the original objective, ensuring that "excessive updates in the right direction" are restricted (clipping takes effect, gradient zeroed out), while "updates in the wrong direction" always retain corrective gradients**. This is the core subtlety in the design of the PPO/GRPO clipping mechanism.

</details>

**14. During GRPO Training, Do Clipped Tokens Contribute to the Loss and Gradients?**

<details>
<summary>💡 Show Answer</summary>

Combining the analysis from Question 428:

**Precise meaning of "being clipped"**: Refers to when in $\min(ratio \cdot A, clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A)$, **the $\min$ operation selects the clipped term $clip(ratio, \cdot, \cdot) \cdot A$** (rather than the unclipped $ratio \cdot A$). This occurs when "the update magnitude in the correct direction has exceeded the limit" (as analyzed in Question 428: $A > 0$ and $ratio > 1+\varepsilon$, or $A < 0$ and $ratio < 1-\varepsilon$).

**Contribution to loss**:

- In this scenario, $\min(\dots) = clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A$, where the value of $clip(ratio, 1-\varepsilon, 1+\varepsilon)$ is a **constant ($1+\varepsilon$ or $1-\varepsilon$, independent of the specific current value of $\theta$ because it has been clipped to boundary values)**, and $A$ is also a constant (independent of $\theta$, being the pre-calculated advantage);
- Therefore, the term corresponding to this token **remains a specific numerical value (non-zero)**, which **contributes to the value of the loss (i.e., the magnitude of total loss)**---that is, clipped tokens contribute to the "numerical magnitude of the loss" (the loss function does not ignore these tokens, and their "clipped values" are still included in total loss computation).

**Contribution to gradients**:

- However, because the term $clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A$ has a **gradient of 0 with respect to $\theta$** (since it is a constant independent of $\theta$---the output value of the clipping operation is fixed at boundary value $1+\varepsilon$ or $1-\varepsilon$, multiplied by constant $A$, yielding a constant overall);
- Thus, **clipped tokens contribute 0 gradient to that update step**---meaning these tokens exert no influence on parameter updates (will not push $\theta$ in any direction).

**Summary**: Clipped tokens:
- **Contribute to the loss value** (the clipped value remains a specific number included in total loss results, affecting values observed on the "loss curve");
- **Do not contribute to gradients (parameter updates)** (gradient is 0; these tokens will not affect the direction or magnitude of parameter changes in the current update step).

This design is precisely the objective of the clipping mechanism---**when a token's update has "reached the limit" (moved sufficiently far in the correct direction), it is no longer pushed further (gradient zeroed out, stopping updates for this part), yet its corresponding "objective function value" is still faithfully recorded in the total loss**. This allows us to monitor training status by observing loss changes and statistics such as "the proportion of tokens clipped" (if a large fraction of tokens is clipped, it may indicate overly fast policy updates or a learning rate that is too large, requiring adjustment).

</details>

**15. Can GRPO Be Used to Train Non-Reasoning Tasks?**

<details>
<summary>💡 Show Answer</summary>

GRPO can theoretically be applied to non-reasoning tasks, but actual performance needs to be evaluated based on specific task characteristics.

**Prerequisites for GRPO to work**:

The GRPO mechanism relies on: (1) sampling $G$ responses for the same prompt; (2) computing rewards for these $G$ responses; (3) estimating advantages using the relative magnitudes of intra-group rewards. These elements are equally applicable in non-reasoning tasks, provided suitable reward signals can be designed.

**Non-reasoning scenarios suitable for GRPO**:
- **Format-based tasks with objective evaluation criteria**: Such as "generating JSON output complying with specified formatting" (format compliance can be rule-checked to yield binary 0/1 rewards);
- **Dialogue safety training**: Sampling multiple times within a group, scoring with a safety classifier, and optimizing safe behaviors via GRPO;
- **Instruction-following tasks with quantifiable metrics**: Such as "generating a summary between 100--200 words" (length compliance rate can be precisely calculated);
- **Tool-use tasks**: Utilizing task completion rates (program execution results) as objective rewards, where GRPO fits naturally.

**Challenges of using GRPO in non-reasoning tasks**:
- **Difficulty in reward design**: Open-ended writing, creative generation, and similar tasks lack objective rewards, relying on LLM-as-a-Judge which introduces noise;
- **More severe all-correct / all-wrong issues**: For relatively simple format-following tasks, the model may satisfy all requirements with high probability early in training, causing intra-group reward variance to approach 0 and losing effective gradient signals;
- **Less efficient than DPO**: For general dialogue alignment tasks, DPO is simpler and often yields comparable performance.

Conclusion: **GRPO can be used for non-reasoning tasks, but is best suited for scenarios where reward signals are explicit and objectively verifiable**; for highly subjective tasks, DPO or PPO+RM is recommended.

</details>

**16. Can GRPO Use Off-Policy Data to Assist Reward Computation?**

<details>
<summary>💡 Show Answer</summary>

![Entropy Decay and Exploration Collapse](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig38_can_grpo_use_off_policy_data_t.png)




**On-policy nature of the standard GRPO pipeline**:

As described in Question 424, GRPO is standardly (near-)on-policy---the calculation of advantage $A_i$ relies on performing intra-group normalization over the rewards of "sampling a group of $G$ responses for the same prompt using the current policy".

**Whether off-policy data can be introduced**---Yes, but the following methods and caveats must be considered:

**1. Used to augment the sample sources within a "group"**

- Theoretically, intra-group normalization only requires "a set of responses with rewards for the same prompt." These responses **do not necessarily have to come entirely from real-time sampling of the current policy**—for example, responses generated by **historical policy versions or responses from other sources (such as human-written or generated by stronger models)** can also be included in this "group." They can be used alongside responses sampled by the current policy to calculate the intra-group mean and standard deviation for normalization;
- **Potential value** of doing so: If the $G$ responses generated by the current policy for a prompt have **similar rewards (e.g., all correct or all wrong, i.e., the "all-correct/all-wrong" problem, refer to Problem 432)**, incorporating off-policy responses with "known significant quality differences" (such as a high-quality reference answer or a historical response known to be of poor quality) into this group can **artificially introduce discrimination**, avoiding invalid gradients caused by zero intra-group variance.

**2. Importance Sampling Correction**

- If off-policy responses are also used to **compute gradients** (rather than merely "expanding the group to calculate normalization statistics"), strictly speaking, **importance sampling correction** must be considered—because these responses are not generated by the current policy $\pi_{\theta}$. Directly using them to compute $\log \pi_{\theta}(y_t|s_t)$ and participating in gradient updates means their "importance sampling ratio" $\pi_{\theta}(y|x)/\pi_{behavior}(y|x)$ (where $\pi_{behavior}$ is the policy used when generating the off-policy sample) may differ significantly from the ratio distribution of "on-policy samples generated by $\pi_{\theta_{old}}$", which could introduce additional variance/bias;
- A relatively simple and common approach is: **off-policy samples are used only to "compute intra-group statistics (mean, standard deviation)", but do not directly participate in "gradient computation"**—that is, only responses sampled by the current policy participate in gradient updates, while the normalization "baseline" (mean/standard deviation) takes off-policy samples into account. This enriches the normalization statistical baseline using off-policy data without introducing complex importance sampling corrections.

**3. Replay Buffer Idea**

- Similar to experience replay in traditional RL, a **replay buffer** can be maintained to store (prompt, response, reward) samples generated over recent training steps (by slightly older policy versions). When calculating the current "group" for a prompt, **historically sampled instances of the same prompt (or similar prompts) from the buffer can be added** to increase group diversity and sample size. However, attention must still be paid to the distributional mismatch between these "slightly older" samples and the current policy ("near-proximal off-policy", similar to the degree of "on-policy approximation" allowed by PPO, where the mismatch cannot be too large).

**Summary**: GRPO is standardly on-policy, but **can incorporate off-policy data to a certain extent**. Its main application is to **assist in group construction to increase the discriminativeness of intra-group rewards (alleviating the zero-variance problem caused by all-correct or all-wrong responses)**. If off-policy samples also directly participate in gradient calculation, **importance sampling correction** must be considered to avoid extra bias. A relatively safe trade-off is to **have off-policy samples participate only in calculating normalization statistics, while gradient updates remain primarily based on samples collected on-policy (by the current policy)**. This approach maintains training stability while improving sample efficiency and mitigating the issue of all-correct/all-wrong groups to some extent.

</details>

**17. What is the Main Idea Behind REINFORCE++?**

<details>
<summary>💡 Show Answer</summary>

REINFORCE++ is an enhanced version of the REINFORCE algorithm that incorporates key optimization techniques from PPO to improve training efficiency and stability, while **retaining the simplicity of not requiring a Critic network**.

**Limitations of REINFORCE**:

Standard REINFORCE uses the actual return of the entire trajectory as the policy gradient weight, resulting in high variance, the necessity of waiting for a complete episode, and low sample efficiency.

**Core Improvements of REINFORCE++**:
- **Introducing PPO's clipping mechanism**: By clipping the importance sampling ratio, it allows multiple mini-batch updates per batch of data, improving sample efficiency and preventing overly aggressive policy updates;
- **Reward normalization**: Performs z-score normalization on reward values within a batch to eliminate gradient variance caused by reward scale differences;
- **Introducing an optimized baseline**: Uses the mean reward within the batch as the baseline, subtracting the baseline from the return to reduce variance: $\hat{G}_t = G_t - b$;
- **KL divergence penalty**: Adds a KL penalty for policy deviation from a reference model to prevent over-optimization.

**Differences from GRPO**:
- REINFORCE++ typically samples 1--2 responses per prompt and performs batch-level normalization, resulting in lower computational overhead;
- GRPO samples $G$ responses per prompt ($G = 8--64$) and performs intra-group normalization, achieving a higher signal-to-noise ratio;
- Neither requires a Critic, and both have comparable implementation complexity;
- REINFORCE++ serves as a "lightweight version" of GRPO and is a more practical choice when resources are limited.

REINFORCE++ has been validated in multiple papers, achieving alignment performance close to PPO with lower computational overhead on small- to medium-scale models.

</details>

**18. Are You Familiar with REINFORCE / ReMax / Dr.GRPO / $\lambda$-GRPO / etc.?**

<details>
<summary>💡 Show Answer</summary>

**REINFORCE**:

The most classic policy gradient algorithm, whose gradient form is:

\[
\nabla J(\theta) = \mathbb{E}[\nabla \log \pi_{\theta}(a|s) \cdot R]
\]

That is, using the "total return $R$ of the entire trajectory" to directly weight the "gradient of log probabilities", without using a value function/baseline, resulting in **very high variance** (which served as the motivation for introducing baselines/value functions in subsequent methods like Actor-Critic and PPO). In LLM scenarios, REINFORCE corresponds to: for each sampled response, directly multiplying its reward by the sum of log-probability gradients of all tokens in that response.

**ReMax**:

Introduces a simple yet effective **baseline** on top of REINFORCE: for the same prompt, in addition to the sampled response (used for computing gradients), an **extra "baseline response" is generated using greedy decoding**. The reward of this baseline response serves as the baseline, which is subtracted from the reward of the sampled response to act as the "adjusted return" for REINFORCE:

\[
A = R(sampled response) - R(greedy response)
\]

Introducing this baseline significantly reduces the variance of REINFORCE while **requiring no additional value function (Critic) to be trained**. It is simpler compared to PPO, and the paper demonstrates its effectiveness and efficiency advantages in RLHF tasks (eliminating the VRAM and training overhead brought by Critic).

**Dr.GRPO ("Done Right" GRPO)**:

As described in Question 430, it **identifies and corrects two normalization operations in GRPO that may introduce bias**:
1. GRPO uses **response length to normalize** per-token loss (i.e., dividing loss by $|o_i|$). Dr.GRPO points out that this causes **responses of different lengths to have inconsistent "contribution weights" to the overall loss**, which may trigger a systematic preference for response length in the model (e.g., tending to generate shorter or longer content, depending on the correlation between reward and length);
2. GRPO normalizes advantages using the **standard deviation of intra-group rewards** ($A_i = (r_i - mean) / std$). Dr.GRPO points out that when the intra-group reward variance is very small (e.g., all responses have similar rewards), dividing by a tiny standard deviation will **amplify the value of advantages, potentially causing training instability or introducing gradient magnitudes disproportionate to "true quality differences"**.
- The "fix" of Dr.GRPO is to **remove these two normalization operations** (or adopt a more reasonable normalization scheme), allowing the optimization objective to more directly reflect the "relative magnitude of rewards themselves" and reducing extra bias introduced by normalization.

**$\lambda$-GRPO (Conceptual explanation, as this is a relatively new/niche variant that may refer to different specific modifications)**:

Usually refers to introducing an additional hyperparameter $\lambda$ into the advantage calculation or loss function of GRPO to **interpolate/blend between different advantage estimation methods** (similar to the role of $\lambda$ in GAE, balancing between "relying entirely on intra-group normalization" and "other forms of baseline estimation"), or to dynamically adjust hyperparameters such as the KL penalty coefficient or clip range across the training process. The goal is to adopt different optimization behaviors at different training stages (e.g., requiring more exploration in early stages and more stable convergence later).

**Summary**: These algorithms form an evolutionary trajectory of "policy gradient methods in LLM-RL scenarios continuously simplifying Critic dependency, correcting normalization biases, and improving training stability"—REINFORCE is the foundational high-variance method; ReMax uses the "reward of greedy decoding" as a simple baseline to reduce variance without a Critic; GRPO replaces Critic with "intra-group relative normalization"; Dr.GRPO further identifies and corrects potential length bias and variance amplification issues in GRPO's normalization operations; and variants like $\lambda$-GRPO provide more flexible advantage estimation/regularization scheduling strategies by introducing extra hyperparameters.

</details>

**19. What Are the Improvement Ideas for GRPO, and Can You Detail Several Specific Algorithms?**

<details>
<summary>💡 Show Answer</summary>

After GRPO was proposed, addressing some of its existing issues (such as length bias, training stability, sampling efficiency, etc.), academia and industry proposed various improved algorithms, mainly including:

**1. DAPO (Decoupled Clip and Dynamic Sampling Policy Optimization)**:
- Addressing issues in GRPO such as "entropy collapse," training instability, and gradients in long sequences, DAPO proposed several improvements (elaborated in Problem 432), such as decoupled clipping range (Clip-Higher), dynamic sampling to filter out "all-correct/all-wrong" sample groups, token-level loss computation (rather than per-sample averaging), and reward shaping for overlong sequences.

**2. Dr.GRPO (Done Right GRPO)**:
- Points out that two design choices in GRPO—"normalizing loss by response length" and "normalizing advantage by standard deviation"—may introduce **length bias**. Shorter or longer responses receive inconsistent "weights" in loss calculation, which could cause the model to tend toward generating outputs of specific lengths rather than based on content quality; Dr.GRPO proposes removing these bias-inducing normalization operations to make the optimization objective reflect the reward signal itself more "purely."

**3. GSPO (Group Sequence Policy Optimization)**:
- Changes the **importance sampling ratio from token-level to sequence-level** in GRPO, i.e., replacing token-wise ratios with the probability ratio of the entire response sequence (usually computed via geometric mean), mitigating training instability caused by excessive variance and cumulative errors of token-level ratios in long sequences (refer to Problems 433 and 434).

**4. ReMax**:
- A simpler policy gradient method that uses the "reward under greedy decoding" as the baseline (rather than GAE or intra-group normalization). Combining the ideas of REINFORCE to eliminate the Critic, the paper shows it to be simpler and effective compared to PPO for LLM alignment tasks.

**5. $\lambda$-GRPO and Other Variants (Conceptual Supplement)**:
- Introduces an extra hyperparameter (such as $\lambda$) to adjust the blending ratio between "intra-group normalization" and other baseline methods in advantage estimation, or to adjust dynamic scheduling strategies for key hyperparameters like KL penalty/clip range to adapt to requirements across different training stages.

**6. VAPO and Other Industry Improvements**:
- Major industrial players have combined components from PPO on top of GRPO (such as reintroducing value functions with more efficient estimation methods), improved advantage estimation methods, special reward shaping for reasoning tasks (e.g., process rewards for intermediate reasoning steps), forming their own "GRPO+" variants.

**Summary**: The main improvement directions of GRPO focus on several dimensions: **eliminating length bias (Dr.GRPO), enhancing training stability and sample efficiency (DAPO's multiple improvements, dynamic sampling), reducing variance of long-sequence ratio estimation (GSPO's sequence-level importance sampling), and further simplifying algorithmic structure (ReMax's alternative implementation without a Critic)**. Most of these improved algorithms are targeted optimizations addressing specific issues exposed during the practice of training large-scale reasoning models (such as DeepSeek-R1-style work) using GRPO.

</details>

**20. What Are DAPO's Improvements over GRPO?**

<details>
<summary>💡 Show Answer</summary>

![Inference Compute Budget Allocation (MCTS vs Best-of-N)](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig39_what_are_dapos_improvements_ov.png)




DAPO (Decoupled Clip and Dynamic sAmpling Policy Optimization) proposes several key improvements targeting issues revealed by GRPO in large-scale reasoning model training:

**1. Clip-Higher (Decoupled Clipping Range)**

- Standard PPO/GRPO uses a symmetric clip range $[1-\varepsilon, 1+\varepsilon]$. DAPO observed that symmetric clipping can make it **"difficult to effectively increase the probability of low-probability tokens"**—when a token's current probability is very low (the model is "unconfident," but it is actually a good action with $A > 0$), the symmetric upper bound $1+\varepsilon$ limits how much the ratio can grow, thereby constraining the speed of increasing that token's probability. This can lead to **premature entropy collapse**, where the model becomes deterministic too early and loses exploration capabilities;
- DAPO proposes **decoupling the upper and lower clip bounds**, using a larger upper bound $\varepsilon_{high}$ (allowing greater headroom for increasing the probability of "good but currently low-probability" tokens) while keeping the lower bound $\varepsilon_{low}$ small, i.e., $clip(ratio, 1-\varepsilon_{low}, 1+\varepsilon_{high})$. This mitigates entropy collapse and maintains exploration during training.

**2. Dynamic Sampling / Filtering "All-Correct" or "All-Wrong" Sample Groups**

- For a prompt, if a sampled group of $G$ responses is **all correct (rewards are identical, e.g., all 1s) or all wrong (rewards are identical, e.g., all 0s)**, then in the intra-group normalized advantage $A_i = (r_i - mean)/std$, **$std = 0$**. This causes numerical issues (division by zero) or results in all $A_i = 0$ (no effective gradient signal);
- DAPO proposes **dynamic sampling**: detecting and **filtering out such "all-correct/all-wrong" sample groups** (as they do not provide effective training signals), and **resampling** new prompt/response groups to fill the batch. This ensures every sample group used for training contains "discriminative" reward differences, improving sample utilization efficiency and training signal validity.

**3. Token-Level Policy Gradient Loss**

- Standard GRPO first averages token losses within each response (normalizing by response length) and then averages across multiple responses in the group. DAPO points out that this "normalize then average" approach causes **tokens in responses of different lengths to have different "weights" on the final loss** (each token's weight is diluted in longer responses);
- DAPO changes to **computing loss uniformly at the level of all (response, token) pairs directly** (without first normalizing by response length), so that every token contributes equally to the loss regardless of whether it comes from a long or short response, mitigating length-related bias.

**4. Overlong Reward Shaping**

- For reasoning tasks, model-generated responses sometimes **exceed preset maximum length limits and get truncated**. If such truncated (incomplete) responses are directly treated as "incorrect" with negative rewards, it may convey a false signal to the model that "it shouldn't produce long outputs" (when in fact the output might be "currently reasoning but not yet finished", rather than "content itself being wrong");
- DAPO proposes a special **reward shaping** strategy for overlong truncated samples (such as not directly assigning strong negative rewards, or giving neutral/soft penalties depending on truncation specifics), reducing noise and bias caused by length truncation in reward signals.

**Summary**: DAPO's four core improvements—**Clip-Higher (decoupled clipping range to mitigate entropy collapse), Dynamic Sampling (filtering invalid sample groups to increase sample utilization), Token-level loss computation (mitigating length-weight bias), and Overlong reward shaping (reducing reward noise from truncation)**—target GRPO's issues across "exploration maintenance," "training signal validity," "length bias," and "reward signal noise," making it an important improved version of GRPO in large-scale reasoning model training practice.

</details>

**21. What Are GSPO's Improvements over GRPO?**

<details>
<summary>💡 Show Answer</summary>

GSPO (Group Sequence Policy Optimization) proposes core improvements targeting potential issues brought by the **token-level importance sampling ratio** in GRPO (and PPO):

**Background**:

In GRPO/PPO, the importance sampling ratio $ratio_t = \pi_{\theta}(y_t|s_t) / \pi_{\theta_{old}}(y_t|s_t)$ is **computed per token**. In long-sequence generation scenarios (such as complex reasoning tasks where outputs may contain thousands of tokens):
- Each token's ratio carries a certain amount of estimation noise/variance;
- When these token-level ratios influence gradients through clipping or min operations, **errors can accumulate along the sequence**, leading to increased variance and instability in overall training—which is especially pronounced in models using MoE architectures or when token-level probabilities fluctuate significantly.

**Core Improvement of GSPO—Sequence-Level Importance Sampling**:

GSPO proposes changing the calculation of the importance sampling ratio from "per-token" to **"entire sequence (response) level"**. Specifically, the sequence-level importance ratio is defined as:

\[
s(\theta) = \left(\frac{\pi_{\theta}(y|x)}{\pi_{\theta_{old}}(y|x)}\right)^{1/|y|}
\]

That is, **taking the geometric mean of the probability ratio of the entire sequence (taking the $n$-th root based on sequence length, equivalent to exponentiating the average of log-probability differences)**. This ratio is a **scalar** representing the "geometric mean level of the response's probability ratio under the new versus old policy," rather than maintaining a separate ratio for each token.

Based on this sequence-level ratio, GSPO's objective function applies clipping and advantage weighting to the **entire sequence** (rather than per token):

\[
L_{GSPO} \propto \min(s(\theta) \cdot A, clip(s(\theta), 1-\varepsilon, 1+\varepsilon) \cdot A)
\]

**Effects of the Improvement**:
- **Variance Reduction**: The sequence-level ratio is the "geometric mean" of probability ratios across all tokens in the sequence, effectively "smoothing/averaging" per-token noise and significantly reducing the variance of the importance sampling ratio;
- **Avoiding Error Accumulation**: There is no longer an issue of "per-token errors accumulating and amplifying within the sequence," because both the optimization objective and clipping operate on the aggregated quantity of the "entire sequence";
- **Better Suited for Long Sequences / Complex Reasoning Tasks**: In scenarios with very long output sequences (such as CoTs with extensive reasoning steps), GSPO displays a more stable training curve compared to GRPO/PPO, especially on MoE models (where token-level probabilities may experience additional fluctuations due to routing changes, and sequence-level aggregation smooths such fluctuations better).

**Summary**: The core improvement of GSPO is changing the **token-level importance sampling ratio in GRPO (and PPO) to a sequence-level ratio (aggregated via geometric mean)**, performing clipping and advantage weighting based on this sequence-level ratio. This **significantly reduces the variance of importance sampling and prevents per-token error accumulation in long sequences**, making training more stable and particularly suitable for RL training scenarios with long outputs/complex reasoning tasks and MoE models.

</details>

**22. Why Does GSPO Use the Geometric Mean Instead of the Arithmetic Mean?**

<details>
<summary>💡 Show Answer</summary>

![LLM Security Alignment and Prompt Injection Defense](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig40_why_does_gspo_use_the_geometri.png)




In GSPO, the sequence-level importance ratio is defined as:

\[
s(\theta) = \left(\frac{\pi_{\theta}(y|x)}{\pi_{\theta_{old}}(y|x)}\right)^{1/|y|} = \exp\left(\frac{1}{|y|} \sum_{t} \left[\log \pi_{\theta}(y_t|s_t) - \log \pi_{\theta_{old}}(y_t|s_t)\right]\right)
\]

That is, **taking the arithmetic mean of each token's log-probability ratio ($\log \pi_{\theta}(y_t|s_t) - \log \pi_{\theta_{old}}(y_t|s_t)$), and then taking the exponential**—which is equivalent to **taking the geometric mean of each token's probability ratio $\pi_{\theta}(y_t|s_t)/\pi_{\theta_{old}}(y_t|s_t)$**. The reasons for choosing geometric mean (rather than directly taking the arithmetic mean of token probability ratios) are:

**1. Natural Mathematical Correspondence to the "Joint Probability Ratio of the Entire Sequence"**

- The joint probability ratio of the entire sequence is $\pi_{\theta}(y|x)/\pi_{\theta_{old}}(y|x) = \prod_{t} [\pi_{\theta}(y_t|s_t)/\pi_{\theta_{old}}(y_t|s_t)]$ (the **product** of token probability ratios, because sequence probability is the product of token conditional probabilities);
- Geometric mean is precisely "taking the $n$-th root of the product of a set of values," i.e., $(\prod_{t} a_t)^{1/n}$, which is **naturally aligned** with the product structure of the "sequence joint probability ratio"—the geometric mean can be understood as "normalizing the joint probability ratio of the whole sequence back to the same 'scale' as a single token's probability ratio" (taking the $n$-th root cancels out the scale inflation/reduction caused by multiplying $n$ ratios);
- If using the arithmetic mean (directly averaging token probability ratios $\frac{1}{n}\sum_{t} [\pi_{\theta}(y_t|s_t)/\pi_{\theta_{old}}(y_t|s_t)]$), this quantity **lacks a direct mathematical correspondence to the "sequence joint probability"**, making its numerical meaning less clear than that of the geometric mean.

**2. Numerical Stability—Avoiding Numerical Explosion/Underflow of the "Joint Probability Ratio" for Long Sequences**

- If directly using the joint probability ratio of the entire sequence $\prod_{t} ratio_t$ (without any average/root), when sequence length $n$ is very large, even if each token's $ratio_t$ deviates only slightly from 1 (such as 1.01 or 0.99), multiplying $n$ such values will result in **exponential growth or decay** ($1.01^{1000}$ is huge, while $0.99^{1000}$ is close to 0), causing numerical overflow/underflow and extreme training instability;
- After taking the geometric mean ($n$-th root), the numerical range of $s(\theta)$ is "compressed" back to a scale close to a single token's ratio (near 1), **avoiding numerical explosion issues caused by long sequences**. This is a crucial advantage of geometric mean regarding numerical stability.

**3. "Average Log-Probability Ratio" in Log Space Has More Direct Statistical Significance**

- The geometric mean corresponds to "exponentiating the arithmetic mean of log-probability ratios"—and the "arithmetic mean of log-probability ratios" itself is a statistically meaningful quantity (similar to "average degree of probability shift per token"). Using this quantity to represent the "overall probability shift trend of the whole sequence" is more robust than directly taking the arithmetic mean of raw probability ratios (which can be easily dominated by extreme values of a single token).

**4. Potential Issues with Arithmetic Mean**

- If taking the arithmetic mean of raw token probability ratios $ratio_t$, **if a single token's $ratio_t$ is abnormally large** (e.g., a token's probability suddenly spikes under the new policy), **it will dominate the entire average**, causing the sequence-level ratio to fail to reflect the "overall trend of most tokens." The geometric mean (averaging in log space) is far less sensitive to such "individual extreme values" than the arithmetic mean (since logarithms compress the dynamic range of numbers).

**Summary**: The core reasons for choosing geometric mean are: **(1) Geometric mean naturally corresponds in mathematical structure to the "sequence joint probability ratio" (product of token probability ratios), where taking the $n$-th root cancels scale changes brought by multiplication; (2) Geometric mean (arithmetic mean in log space followed by exponentiation) compresses joint probability ratios that could otherwise "exponentially explode/decay" in long sequences back into a reasonable scale close to single-token ratios, guaranteeing numerical stability; (3) Averaging in log space is insensitive to extreme single-token values, making it more robust than directly arithmetic-averaging raw ratios**. These properties make geometric mean a more natural and stable choice for sequence-level importance sampling ratios.

</details>

**23. Have You Heard of GSPO and DAPO? What Are Their Differences from GRPO?**

<details>
<summary>💡 Show Answer</summary>

Both DAPO and GSPO are improved algorithms addressing problems exposed during practical training of GRPO, developed by research teams from ByteDance/Alibaba and Tencent, respectively.

**DAPO (Decoupled Clip and Dynamic Sampling Policy Optimization)**:

**Issues in GRPO**: (1) Symmetric clip range limits the headroom for increasing low-probability good actions, leading to entropy collapse; (2) "All-correct/all-wrong" sample groups provide zero-variance gradients, wasting computational resources.

**DAPO's Improvements**:
- **Clip-Higher**: Decouples the upper and lower clip bounds, using a larger upper bound $\varepsilon_{high} > \varepsilon_{low}$ to allow low-probability good tokens larger headroom for improvement, mitigating entropy collapse;
- **Dynamic Sampling**: Filters out all-correct/all-wrong sample groups and resamples to ensure every group has effective gradient signals;
- **Token-Level Loss**: Switches to uniform weighting across all tokens without normalizing by response length;
- **Overlong Reward Shaping**: Applies special reward treatment for truncated overlong responses.

**GSPO (Group Sequence Policy Optimization, Tencent)**:

**Issues in GRPO**: Token-level importance sampling ratios accumulate variance in long sequences, leading to training instability, which is particularly evident in MoE models.

**GSPO's Improvements**: Aggregates token-level ratios into a **sequence-level geometric mean ratio**: $s(\theta) = (\pi_\theta(y|x)/\pi_{\theta_{old}}(y|x))^{1/|y|}$, performing clipping and advantage weighting based on sequence-level ratio, significantly reducing variance accumulation in long sequences.

**Comparison of the Three**: GRPO is the foundational version; DAPO primarily addresses exploration diversity and sampling efficiency issues; GSPO primarily addresses long-sequence training stability issues. In practice, their respective improvement ideas can be combined.

</details>

**24. How Should One View the Phenomenon of Numerous Algorithms Claiming to Outperform PPO?**

<details>
<summary>💡 Show Answer</summary>

In recent years, a large number of RL alignment algorithms claiming to "outperform PPO" have continuously emerged. This phenomenon needs to be understood critically.

**Why Does This Phenomenon Occur?**

- **Limitations of evaluation benchmarks**: Many claims of "outperforming PPO" are based on specific evaluation settings favorable to the proposed algorithm (specific tasks, specific prompt distributions, specific evaluation metrics). Switching to a different benchmark may cause the advantage to disappear;
- **Untuned PPO baselines**: PPO's hyperparameters (KL coefficient, clip range, GAE $\lambda$, learning rate, etc.) are extremely tedious to tune. The PPO baseline in paper comparisons is often inadequately tuned, leaving room for proposed methods to easily "surpass" it;
- **Unfair comparison of training costs**: Some "simpler" methods (such as DPO) do indeed outperform PPO on certain metrics, but have restrictions on data requirements and application scenarios, rather than comprehensive superiority;
- **Publication bias**: Papers claiming to surpass SOTA are easier to get accepted, even if the improvements are minor or fragile.

**How to View It Rationally?**

- **Distinguish "outperforming in specific settings" from "comprehensive superiority"**: Currently, no single method outperforms PPO across all tasks, all model scales, and all resource constraints;
- **Focus on genuine dimensions of improvement**: Algorithms like GRPO (reducing VRAM), DPO (lowering engineering complexity), and DAPO (improving exploration) indeed bring substantial improvements in **specific dimensions** and deserve attention;
- **PPO's core value is robustness**: Polished through years of engineering practice, PPO provides stable and reliable results across diverse tasks and scales. This "generalization robustness" is difficult for many new algorithms to match;
- **The value of new algorithms lies in lowering barriers**: The true value of methods like GRPO and DPO is not being "stronger," but rather being "more accessible"—enabling more teams to train alignment models under resource-constrained settings.

Conclusion: Treat claims of "outperforming PPO" with prudence, focusing on which dimensions and under what conditions an algorithm improves, rather than blindly believing absolute comparative claims.

</details>

**25. Analyzing the Limitations of Alternative RLHF Algorithms**

<details>
<summary>💡 Show Answer</summary>

Alternative RLHF algorithms that have emerged in recent years each have their own applicable scenarios, but they also possess non-negligible limitations:

**1. Limitations of the DPO Family**

- **Off-policy problem**: The training data is fixed. As the policy evolves, the shift between the data distribution and the current policy distribution becomes larger, impairing learning quality;
- **Lack of exploration capability**: DPO can only learn "good responses" that already exist in the dataset, unable to discover better strategies beyond the training data distribution through online exploration like PPO. For reasoning tasks, this limitation is particularly pronounced;
- **High sensitivity to data quality**: Preference data is highly dependent on construction quality. If the difference between chosen and rejected responses is too small, or the data distribution is sub-optimal, training performance suffers greatly;
- **Theoretical upper bound restricted by reference model**: DPO's optimization space is constrained by the reference model distribution. If the reference model (SFT model) itself has limited quality, the upper bound of DPO's performance is also capped.

**2. Limitations of GRPO**

- **High rollout cost**: Each prompt requires sampling $G$ responses ($G=8--64$), making the overall computational cost significantly higher than PPO;
- **All-correct / all-incorrect degradation**: If all responses for the same prompt receive identical rewards, the intra-group variance becomes zero, resulting in a wasted batch of data;
- **Poor adaptability to open-ended tasks**: It relies on objectively verifiable rewards, making its effectiveness on highly subjective tasks (e.g., creative writing, general dialogue) inferior to the PPO+RM combination.

**3. Limitations of Best-of-N**

- **Inference cost scales linearly with $N$**: $N=64$ implies a 64-fold increase in inference cost, making large-scale deployment difficult;
- **Does not alter intrinsic model capabilities**: BoN at inference time merely "taps into" existing capabilities of the model better, rather than enabling the model to learn new skills.

**Irreplaceability of PPO**:

In scenarios requiring **online exploration**, handling **subjective preference alignment**, or pursuing an **absolute performance ceiling**, PPO remains the most reliable choice. The core value of alternative algorithms lies in **lowering engineering barriers and resource requirements**, rather than surpassing PPO across all scenarios.

</details>

