# 5.1 RL Fundamental Concepts and Frameworks — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 5: Reinforcement Learning and Post-Training Alignment (RLHF / DPO) · 22 questions

**1. Explain the Basic Concepts of Reinforcement Learning**

<details>
<summary>💡 Show Answer</summary>

![Reinforcement Learning Agent-Environment Interaction Loop](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig01_explain_the_basic_concepts_of.png)




Reinforcement Learning (RL) is a subfield of machine learning aimed at learning an optimal policy that maximizes cumulative reward through **continuous interaction between an Agent and an Environment**. Its core relies on a "trial-and-error" and "feedback" mechanism: the agent continuously attempts different actions, the environment provides reward signals, and the agent adjusts its behavior accordingly.

**Core Elements**:
- **State ($S$)**: Describes the complete information of the current environment, such as a chessboard configuration or a dialogue context;
- **Action ($A$)**: Operations that the agent can execute, corresponding to each generated token in LLM scenarios;
- **Reward ($R$)**: The immediate feedback signal provided by the environment for the agent's action, which can be positive or negative;
- **Policy ($\pi$)**: A mapping function from states to actions, serving as the agent's "decision rule";
- **Value Function ($V/Q$)**: Evaluates the long-term expected return of a certain state (or state-action pair).

**Optimization Objective**: Maximize the cumulative discounted reward $G_t = \sum_{k=0}^{\infty} \gamma^k r_{t+k}$, where the discount factor $\gamma \in [0,1]$ controls the emphasis placed on future rewards.

**Core Differences from Supervised Learning**: Supervised learning has explicit "ground-truth answers" as labels; reinforcement learning has no standard answers, only sparse reward signals, and current actions affect future state distributions (temporal dependence). This gives RL a unique advantage in scenarios where all high-quality behaviors cannot be exhaustively enumerated in advance (such as dialogue and reasoning).

**Mapping to LLMs**: The token generation process of LLMs naturally fits the RL framework—choosing a token (action) at each timestep, receiving a human preference reward (reward) after generating a complete response, and updating model parameters via algorithms such as PPO/GRPO (policy optimization).

</details>

**2. What Key Elements Are Included in Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

Reinforcement learning problems are typically modeled as a Markov Decision Process (MDP), which includes the following key elements:
1. **State ($S$)**: The environmental state in which the agent is located.
2. **Action ($A$)**: The operations that the agent can execute under a specific state.
3. **Transition Dynamics ($P$)**: The probability of transitioning to the next state $s'$ after executing action $a$ from the current state $s$.
4. **Reward ($R$)**: The immediate return received by the agent at each timestep.
5. **Discount Factor ($\gamma$)**: Used to balance the relative importance between immediate rewards and future rewards, usually ranging within $(0, 1]$.

</details>

**3. What Are Two Key Characteristics of Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

First, it relies on an ongoing trial-and-error process to discover optimal behaviors; second, it must handle the issue of delayed rewards. This leads to significant differences between RL and supervised learning in how models are evaluated:
- **Supervised Learning**: The model receives immediate feedback based on known ground-truth answers, and the learning process directly targets reducing prediction errors.
- **Reinforcement Learning**: The agent evaluates its actions through interactions with the environment, and the effectiveness of the policy can only be determined through the evaluation of cumulative rewards.

</details>

**4. What Properties Does a Markov Process Possess?**

<details>
<summary>💡 Show Answer</summary>

![Markov Property and Memorylessness State Transition](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig02_what_properties_does_a_markov.png)




The Markov property is a core assumption in the theoretical framework of reinforcement learning, also referred to as **"memorylessness" or "one-step dependency"**.

**Core Definition**: Given the current state $S_t$, the conditional probability of the future state $S_{t+1}$ is independent of the history sequence of states $S_0, S_1, ..., S_{t-1}$, i.e.:
\[P(S_{t+1} | S_t, A_t) = P(S_{t+1} | S_0, S_1, ..., S_t, A_0, ..., A_t)\]
The current state fully summarizes all historical information and completely determines the future evolution of the system.

**Key Properties**:
- **State Sufficiency**: The current state $S_t$ is a sufficient statistic required to make optimal decisions without looking back at history;
- **Time Homogeneity (Additional assumption for Homogeneous MDP)**: The state transition probability $P(s'|s,a)$ does not change over time;
- **Foundation of the Bellman Equation**: Processes satisfying the Markov property constitute a Markov Decision Process (MDP), where the long-term value can be decomposed into a recursive relationship using the Bellman equation: $V^*(s) = \max_a [R(s,a) + \gamma \sum_{s'} P(s'|s,a) V^*(s')]$

**Application and Limitations in LLMs**: In the token generation process of an LLM, treating "the full sequence generated so far (all context recorded in KV Cache)" as the state satisfies the Markov property. In practice, however, restricted by context window length, LLMs may not be able to observe the entire history, which to some extent violates the strict Markov assumption.

</details>

**5. How to Understand RL in LLMs?**

<details>
<summary>💡 Show Answer</summary>

![Mapping Reinforcement Learning Elements to LLMs](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig03_how_to_understand_rl_in_llms.png)




The integration of reinforcement learning and large language models can be understood as follows:

- **State ($S$)**: The state is the input prompt (instruction or question), which is the text input provided by the user. The state is the information that the agent (LLM) needs to process, determining its subsequent decision-making.
- **Action ($A$)**: The action is the output response of the LLM, i.e., the next token (or a segment of text) generated by the model. The action is the decision made by the model in the current state, dictating the quality and direction of the generated content.
- **Reward ($R$)**: The reward is a score given by the Reward Model (RM) based on the combination of prompt + response. The reward signal measures the quality of the generated content, guiding the model to adjust its policy to generate better outputs.
- **Overall Goal**: Given a prompt, adjust the policy to generate responses that align with human preferences.

</details>

**6. What Are the State, Action Space, and Action when Applying RL to Large Models?**

<details>
<summary>💡 Show Answer</summary>

When modeling LLM training as a reinforcement learning problem (taking RLHF as an example), the elements correspond as follows:

- **State**: The current "context", i.e., the entire sequence of tokens generated so far, including the prompt and the partial response generated so far. Specifically, when generating the $t$-th token, state $s_t = (prompt, y_1, y_2, \dots, y_{t-1})$—the state is updated with every newly generated token (as the new token is appended to the context).

- **Action Space**: All possible tokens in the vocabulary, i.e., the set of all tokens that the model can choose to output at each step; the size of the action space equals the vocabulary size (typically tens of thousands to hundreds of thousands).

- **Action**: Under state $s_t$, the specific token sampled/chosen and output by the policy model (LLM), i.e., $a_t = y_t$.

- **Policy**: The LLM itself, where $\pi_{\theta}(a_t \mid s_t)$ represents the output probability distribution over each token in the vocabulary given the context (state).

- **Trajectory/Episode**: The complete generation process starting from a given prompt and generating token-by-token until producing EOS (or reaching maximum length) constitutes an "episode", corresponding to a full response.

- **Reward**: Typically sparse—given as a scalar reward by the reward model for the entire (prompt, response) pair only upon completion of generation (when the whole response is done); the "immediate reward" for each intermediate step is usually 0 (in implementations with KL penalty, each step also receives a small penalty signal based on KL divergence).

**Summary**: The state is the "currently generated context", the action space is the "vocabulary", the action is the "token chosen for output at the current step", and the entire generation process forms a trajectory. Rewards are typically provided only at the end of the trajectory, making it a classic "sequential decision-making, sparse reward" problem.

</details>

**7. How Are Classic RL Algorithms Categorized?**

<details>
<summary>💡 Show Answer</summary>

![Taxonomy of Reinforcement Learning Algorithms](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig04_how_are_classic_rl_algorithms.png)




Reinforcement learning algorithms can be categorized according to different criteria:

1. **Model-Based vs. Model-Free**:
    - Model-Based: Relies on a model of the environment to make decisions, such as AlphaGo/AlphaZero and MuZero.
    - Model-Free: Does not rely on an environment model, learning directly from interactions with the environment.
2. **Value-Based vs. Policy-Based**:
    - Value-Based: Makes decisions by learning state-value functions or action-value functions, such as Q-Learning and DQN.
    - Policy-Based: Directly learns a policy function, such as Policy Gradient and TRPO/PPO.
3. **On-Policy vs. Off-Policy**:
    - On-Policy: Updates the policy using data generated by the current policy, such as SARSA.
    - Off-Policy: Can update the target policy using data generated by other policies, such as Q-Learning and DQN.

</details>

**8. What Are the Key Differences Between General RL and LLM RL?**

<details>
<summary>💡 Show Answer</summary>

General reinforcement learning (e.g., games, robotics control) and large model RL (LLM post-training) exhibit several key differences in their core components and challenges:

**1. Action Space Dimension**
- **General RL**: The action space is usually small and limited (e.g., 361 placement positions in Go, or continuous joint angles in robotic control);
- **LLM RL**: The action space is the size of the vocabulary (30,000–100,000+ tokens), selecting an action in this ultra-high-dimensional space at every timestep, making value-based methods (like Q-learning) virtually infeasible.

**2. Determinism of State Transitions**
- **General RL**: State transitions are typically dictated by the real physical world or deterministic game rules;
- **LLM RL**: State transitions are completely deterministic—given the generated token sequence, the next state is simply the current sequence + newly generated token, with no stochasticity in state transitions (stochasticity enters only during token sampling).

**3. Episode Structure**
- **General RL**: An episode can span thousands to millions of timesteps (e.g., training game AIs);
- **LLM RL**: An episode usually spans only dozens to a few thousand tokens, being relatively short overall, but the model forward computation per token is extremely expensive.

**4. Sparsity of Reward Signals**
- **General RL**: Depending on the task, rewards can be dense (per-step) or sparse (only at termination);
- **LLM RL**: Rewards are almost always **given only after the entire response generation is completed** (sequence-level sparse rewards), which is the fundamental source of the credit assignment challenge in RLHF.

**5. Policy Parameter Scale**

- **General RL**: Policy networks are usually small (millions of parameters);
- **LLM RL**: The policy model itself is an LLM (billions to hundreds of billions of parameters); a single forward pass consumes massive compute, imposing extremely high requirements on training efficiency.

</details>

**9. What Is the Difference Between On-Policy and Off-Policy in Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

![On-Policy vs. Off-Policy Training Loop Comparison](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig05_what_is_the_difference_between.png)




On-policy and off-policy are core concepts in reinforcement learning describing the **data sources used to update policies**. The fundamental distinction lies in whether the "data-generating policy" matches the "target policy being optimized".

**Definition Comparison**:
- **On-Policy**: The sample data used to calculate gradients and update parameters **must be generated by the current policy $\pi_\theta$ being optimized**. After each policy update, old data becomes "stale" and must be re-collected;
- **Off-Policy**: Can use data generated by **any other policy (behavior policy $\mu$, such as historical versions or human annotations)** to update the target policy $\pi_\theta$, and the two can be distinct.

**Core Technical Difference**:

Off-policy methods typically require **Importance Sampling** to correct differences in data distribution:
\[\mathbb{E}_{a \sim \mu}[f(a)] = \mathbb{E}_{a \sim \pi}\left[\frac{\mu(a|s)}{\pi(a|s)} f(a)\right]\]
Introducing the importance weight $w = \pi(a|s)/\mu(a|s)$ "re-weights" data from the old policy, making it equivalent to data collected under the new policy. PPO leverages this exact principle to achieve "proximal on-policy" learning (reusing old data within a constrained range).

**Mapping to LLM Scenarios**:
- **PPO/GRPO**: Inherently on-policy methods, where each rollout round generates responses using the current policy, and data is discarded and re-sampled after updates;
- **DPO**: Uses pre-collected human preference pairs (generated by SFT models or other policies), serving as a typical off-policy method;
- **Iterative DPO**: Re-samples data with the latest policy in each round before running DPO, acting as a compromise between on-policy and off-policy.

**Core Trade-off**: On-policy gradient estimation is unbiased but has low sample efficiency; off-policy has high sample efficiency but faces distribution shift risks. PPO balances both through a clip mechanism, which is the fundamental reason why PPO is widely adopted.

</details>

**10. Why Distinguish Between On-Policy and Off-Policy?**

<details>
<summary>💡 Show Answer</summary>

1. **Policy Distribution Consistency**: If data originates from a distribution different from the target policy, additional techniques (such as importance sampling) are required to ensure learning correctness.
2. **Data Efficiency**: On-policy methods typically follow a “sample-train-discard” paradigm where data cannot be reused; off-policy methods can reuse data repeatedly via mechanisms like experience replay.
3. **Safety and Controllability**: Off-policy methods can collect data using a conservative behavior policy while learning a more aggressive target policy.

</details>

**11. What Are Representative Algorithms for On-Policy and Off-Policy?**

<details>
<summary>💡 Show Answer</summary>

**Representative On-Policy Algorithms**:
- **SARSA**: A classic on-policy TD algorithm that uses the action actually executed by the current policy during updates;
- **A2C/A3C**: Actor-Critic methods that update based on data collected by the current policy;
- **PPO**: Allows multiple mini-batch updates via importance sampling and controls policy update magnitude using a clip mechanism, serving as a core algorithm for LLM RL;
- **GRPO**: A variant of PPO that removes the Critic and uses group normalization to estimate advantages.

**Representative Off-Policy Algorithms**:
- **Q-Learning**: The most classic off-policy algorithm, which uses $\max_{a'} Q(s',a')$ (a greedy policy) during updates, independent of the actual behavior policy;
- **DQN (Deep Q-Network)**: Combines Q-Learning with deep networks and introduces an experience replay buffer, representing a typical off-policy method;
- **SAC (Soft Actor-Critic)**: A maximum entropy reinforcement learning framework that leverages experience replay to efficiently handle continuous action spaces;
- **DPO**: A typical off-policy method in LLM alignment that trains on pre-collected preference data without online sampling.

In LLM post-training scenarios, on-policy PPO/GRPO and off-policy DPO are two dominant alignment algorithms—the former provides more stable training at a higher cost, while the latter is simple and efficient but relies heavily on data quality.

</details>

**12. What Are the Advantages and Disadvantages of On-Policy and Off-Policy?**

<details>
<summary>💡 Show Answer</summary>

**Advantages and Disadvantages of On-Policy**:

**Advantages**:
- **Strong Theoretical Guarantees**: Training data matches the current policy distribution completely, avoiding distribution shift problems and yielding unbiased gradient estimations;
- **Stable Training**: Avoids the instability caused by discrepancies between data and current policy distributions in off-policy methods;
- **Effective Exploration**: Collects new data based on the latest policy in each round, enabling continuous exploration in the action space near the current policy.

**Disadvantages**:
- **Low Sample Efficiency**: Each batch of data can only be used for one (or a limited number of) update(s), becoming "stale" immediately after the update and making historical experience non-reusable;
- **High Computational Cost**: In LLM scenarios, every update round requires expensive online rollouts (autoregressive generation), acting as a training bottleneck;
- **Difficult to Parallelize**: Data collection and parameter updates share a strong dependency, making complete asynchronous execution difficult.

**Advantages and Disadvantages of Off-Policy**:

**Advantages**:
- **High Sample Efficiency**: Can reuse historical experience (experience replay) by leveraging the same data repeatedly, significantly reducing required interactions;
- **Low Training Cost**: Methods like DPO do not require online sampling, operating similarly to supervised learning with simpler implementation and debugging;
- **Support for Offline Datasets**: Can directly utilize historically collected datasets (such as human-annotated preference pairs) without needing real-time interaction with the environment.

**Disadvantages**:
- **Distribution Shift Risk**: Inconsistency between training data distribution and the current policy distribution may cause value estimation bias or even training divergence;
- **Overfitting Risk**: Methods like DPO tend to overfit to fixed preference datasets, limiting generalization capability;
- **Weak Exploration Capability**: Unable to discover superior policies outside the dataset through online exploration.

</details>

**13. What Is the Difference Between Online and Offline in Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

![Online vs. Offline RL Data collection Flow](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig06_what_is_the_difference_between.png)




Online/Offline and On-policy/Off-policy represent classifications across two distinct dimensions, describing different aspects of RL training data sources and update mechanisms.

**Online Learning**:

The agent interacts with the environment **in real time** during training, updating policies while collecting new data.
- **Characteristics**: Data is generated dynamically and collected adaptively according to the exploration needs of the current policy;
- **Advantages**: Continuously obtains fresh data under the current policy distribution, adapting to changes in policy;
- **Disadvantages**: Requires real-time environmental interaction, which in LLM scenarios implies online rollout sampling with high computational costs;
- **Typical Algorithms**: PPO, GRPO, SAC, DQN (with experience replay).

**Offline Learning**:

The agent learns using a **pre-collected static dataset** without interacting with the environment during training.
- **Characteristics**: The dataset is fixed, and the policy can only learn from existing data;
- **Advantages**: Requires no real-time environment, fully utilizes historical data, and features low training costs;
- **Challenges**: Severe Out-Of-Distribution (OOD) issues—the policy may learn erroneous actions outside the scope covered by the dataset;
- **Typical Algorithms**: DPO (using pre-collected preference pair datasets), CQL, IQL.

**Key Differences from On/Off-Policy**:
- On/Off-policy describes **whether the data used for updates comes from the current policy** (consistency between data-generation policy vs. optimized policy);
- Online/Offline describes **whether there is real-time interaction with the environment during training** (whether data is dynamically collected or statically used).
- Note: Online algorithms are not necessarily On-policy (e.g., DQN is Online + Off-policy); Offline algorithms are usually Off-policy, but the converse is not necessarily true.

In LLM alignment: PPO/GRPO is Online + On-policy; DPO is Offline + Off-policy; Iterative DPO serves as an Online + Off-policy compromise.

</details>

**14. What Is the Difference Between On/Off-Policy and Online/Offline?**

<details>
<summary>💡 Show Answer</summary>

1. **Different Dimensions**: On/off-policy addresses the degree of consistency between the “sampling policy” and “target policy”; Online/offline addresses “whether continuous interaction to collect new data is possible”.
2. **Common Combinations**: Offline reinforcement learning is almost always off-policy, whereas online reinforcement learning can be either on-policy or off-policy.

</details>

**15. Differences and Connections Between On/Off-Line and On/Off-Policy in RL**

<details>
<summary>💡 Show Answer</summary>

- **Online + On-policy**: Real-time interaction and updated only using data from the current policy (e.g., PPO).
- **Online + Off-policy**: Real-time interaction but updated using historical data (e.g., DQN using experience replay).
- **Offline + Off-policy**: The most common combination, updating policies using static datasets (e.g., BCQ, CQL).
- **Offline + On-policy**: Theoretically possible, but virtually non-existent in practice.

</details>

**16. What is the Relationship Between the Advantage Function, Value Function, and Action-Value Function?**

<details>
<summary>💡 Show Answer</summary>

![Relationship between Advantage (A), Value (V), and Action-Value (Q) Functions](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig07_what_is_the_relationship_betwe.png)




The relationship between the advantage function (A), value function (V), and action-value function (Q) is that the advantage function is the difference between the Q-function and the V-function.
- **Value Function (V)**: Measures the value of a state, reflecting the total expected return the agent can obtain from that state under the current policy.
- **Action-Value Function (Q)**: Measures the total expected return the agent can obtain after taking a specific action in a given state.
- **Advantage Function (A)**: Used to evaluate the return gain of a specific action relative to the average policy, serving as a metric for comparing actions.

</details>

**17. What are the Three Common Methods for Estimating Value Functions?**

<details>
<summary>💡 Show Answer</summary>

In reinforcement learning, estimating value functions is a key step. Three common methods include:
1. **Monte Carlo Method (MC)**: Estimates the value function through complete trajectory sampling.
2. **Temporal Difference Method (TD)**: Estimates using partial trajectory information.
3. **Generalized Advantage Estimation (GAE)**: Combines the advantages of Monte Carlo and Temporal Difference methods, improving estimation stability and efficiency by trading off bias and variance.

</details>

**18. Summarize the Three Value Function Estimation Methods from the Perspective of Bias and Variance**

<details>
<summary>💡 Show Answer</summary>

![Bias-Variance Trade-off in Value Function Estimation](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig08_summarize_the_three_value_func.png)




1. **MC Method**: High variance, because it relies on the complete episode return, meaning updated values can be significantly affected by stochastic factors; unbiased, as it updates using the actual cumulative return directly.
2. **TD Method**: Low variance, as it updates using immediate step-by-step rewards; high bias, as it relies on current estimates for bootstrapping, easily introducing bias.
3. **GAE Method**: Compromise/trade-off, allowing flexible balancing between bias and variance by adjusting the parameter $\lambda$. When $\lambda$ is close to 0, it behaves closer to the TD method; when $\lambda$ is close to 1, it behaves closer to the MC method.

</details>

**19. What Are the Advantages and Disadvantages of the Monte Carlo Method?**

<details>
<summary>💡 Show Answer</summary>

The Monte Carlo (MC) method estimates the value function using the **actual return of complete trajectories**, without using bootstrapping.

**Advantages**:
- **Unbiased estimation**: Directly uses the actual return $G_t = r_t + \gamma r_{t+1} + \gamma^2 r_{t+2} + ...$, independent of other estimators; it is unbiased in expectation without introducing systematic error;
- **No environment model required**: Does not rely on transition probabilities, making it a fully model-free method;
- **More robust to non-Markovian properties**: Uses actual returns rather than recursive estimates, rendering it insensitive to violations of the Markov assumption;
- **Suitable for episodic tasks**: Simple to implement in tasks with clear starts and ends (such as a single LLM response).

**Disadvantages**:
- **High variance**: Randomness at each step of the complete trajectory accumulates into the final return, leading to high estimation variance and unstable training;
- **Must wait until episode ends to update**: Cannot update online, making it unsuitable for continuous tasks or scenarios with very long episodes;
- **Low efficiency for long trajectories**: In LLM generation scenarios, if the response is very long (e.g., CoT reasoning with thousands of tokens), waiting for complete generation before updating results in poor real-time responsiveness;
- **Difficult credit assignment**: In long sequences with sparse rewards, it is hard to discern from the final return which intermediate steps actually contributed.

**Manifestation in LLM RL**: The REINFORCE algorithm is essentially an MC method; GRPO uses "group normalization" to reduce MC variance; while PPO introduces a Critic (TD estimation) to replace MC returns, achieving a balance between bias and variance.

</details>

**20. What Are the Advantages and Disadvantages of Temporal Difference Methods?**

<details>
<summary>💡 Show Answer</summary>

Temporal Difference (TD) methods combine ideas from Monte Carlo and dynamic programming, updating value estimates at every step through **bootstrapping**: $V(S_t) \leftarrow V(S_t) + \alpha[r_{t+1} + \gamma V(S_{t+1}) - V(S_t)]$.

**Advantages**:
- **Low variance**: Uses only a single immediate reward $r_{t+1}$ plus the next state's value estimate per step, introducing fewer sources of randomness and far lower variance than MC;
- **Supports online/incremental updates**: Does not need to wait until the end of an episode, updating at each step, which is suitable for continuous tasks and long sequences;
- **Higher sample efficiency**: Utilizes current experience at every timestep without wasting intermediate steps;
- **Usually converges faster in practice**: Although biased, low variance typically makes the actual learning speed superior to MC.

**Disadvantages**:
- **Biased estimation**: TD bootstraps using the current value estimate $V(S_{t+1})$, which itself is inaccurate, introducing systematic bias; although bias decreases as the value function converges, it cannot be completely eliminated;
- **Sensitive to the Markov assumption**: The validity of TD bootstrapping depends on the current state sufficiently summarizing historical information; if violated, value estimation may suffer systematic errors;
- **Instability with function approximation**: When combined with non-linear function approximation (such as neural networks), TD learning may encounter the "deadly triad" problem, leading to divergence.

**Manifestation in LLM RL**: The Critic in PPO is essentially TD learning—the Critic utilizes GAE (a variant of TD($\lambda$)) to estimate the advantage function at each token position, balancing low variance (bootstrapping) and low bias (multi-step rollout).

</details>

**21. What is Generalized Advantage Estimation (GAE)?**

<details>
<summary>💡 Show Answer</summary>

Generalized Advantage Estimation (GAE), proposed by Schulman et al., is an **advantage function estimation method** that combines ideas from Monte Carlo and Temporal Difference learning, with the core goal of balancing bias and variance.

**Core Idea**:

Single-step TD advantage estimation ($\delta_t = r_t + \gamma V(S_{t+1}) - V(S_t)$) has low variance but high bias, while MC returns have low bias but high variance. GAE performs an exponentially weighted average of TD errors across different timesteps by introducing the hyperparameter $\lambda$:
\[A_t^{GAE(\gamma,\lambda)} = \sum_{l=0}^{\infty} (\gamma\lambda)^l \delta_{t+l}\]
where $\delta_{t+l} = r_{t+l} + \gamma V(S_{t+l+1}) - V(S_{t+l})$ is the TD error at step $t+l$.

**Role of Hyperparameter $\lambda$**:
- $\lambda = 0$: Degenerates to single-step TD error, having maximum bias but minimum variance;
- $\lambda = 1$: Degenerates to MC return (minus baseline), having maximum variance but minimum bias;
- $0 < \lambda < 1$ (typically 0.9--0.99): Flexibly trades off between bias and variance, performing best in practice.

**Application in PPO**: PPO uses GAE to estimate the advantage value $A_t$ at each token position, then uses $A_t$ to weight policy gradients for updating the Actor, while simultaneously training the Critic to fit the true value function $V(s)$. GAE is one of the key techniques enabling PPO to achieve more stable training compared to pure REINFORCE.

</details>

**22. What Are the Advantages and Disadvantages of Generalized Advantage Estimation (GAE)?**

<details>
<summary>💡 Show Answer</summary>

**Advantages of GAE**:
- **Controllable bias-variance**: Continuously interpolates between TD (low variance, high bias) and MC (high variance, low bias) via the $\lambda$ parameter, allowing flexible adjustments based on task characteristics; it is the most practical advantage estimation method in current RL training;
- **Reduced dependency on Critic accuracy**: When $\lambda > 0$, GAE incorporates components of actual returns, mitigating the impact of Critic estimation bias on advantage estimation;
- **More reasonable credit assignment**: Compared to single-step TD, GAE accounts for multi-step future rewards, enabling more accurate attribution of final rewards to early critical actions, which is especially important in sparse reward scenarios;
- **Naturally compatible with PPO**: The advantage function estimated by GAE has moderate variance, and training is extremely stable when combined with PPO's clipping mechanism, making it a standard choice for large-scale industrial RL training.

**Disadvantages of GAE**:
- **Requires a Critic model**: GAE relies on estimating the value function $V(s)$, requiring simultaneous maintenance and training of a Critic, which increases GPU memory and computational overhead—precisely the disadvantage that GRPO avoids by removing the Critic in favor of group normalization;
- **Hyperparameter sensitivity**: The $\lambda$ parameter requires careful tuning, and optimal values vary significantly across different tasks;
- **Higher computational cost**: Requires computing weighted accumulations for each token position and waiting until the sequence ends to calculate complete GAE estimates backward from the end;
- **Challenges from simplified methods like GRPO**: GRPO completely bypasses GAE and Critic via in-group relative reward normalization; although accuracy might be slightly lower, it drastically reduces engineering complexity and often achieves comparable or even superior results on reasoning tasks.

</details>

