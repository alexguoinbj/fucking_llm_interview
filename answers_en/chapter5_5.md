# 5.5 Detailed Explanation of the PPO Algorithm — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 5: Reinforcement Learning and Post-Training Alignment (RLHF / DPO) · 34 questions

**1. Why Use PPO Instead of REINFORCE / Q-learning?**

<details>
<summary>💡 Show Answer</summary>

**Compared to REINFORCE**:
- REINFORCE is a purely on-policy policy gradient method with no baseline or only a simple baseline, resulting in **extremely high variance**, which makes training highly unstable, especially in high-dimensional discrete action spaces like LLMs.
- PPO introduces a **value function (critic) as a baseline** to significantly reduce variance; simultaneously, it limits the update step size at each step via a **clipping mechanism**, making training much more stable.

**Compared to Q-learning Family Algorithms (such as DQN)**:
- The action space of an LLM is the entire vocabulary (extremely large and discrete), and the states at each step rarely repeat, making it difficult for the Q-function to generalize and converge.
- Q-learning relies on off-policy experience replay and max operations, which incur prohibitive computational costs and large estimation biases in such a massive action space.
- PPO is an on-policy algorithm that directly optimizes the policy itself. Combined with KL constraints, it is better suited for making small policy improvements under the setup where "the language model already serves as a strong prior".

Summary: PPO strikes a good balance among **stability (clip + KL)**, **sample efficiency (variance reduction via actor-critic)**, and **adaptability to large-scale discrete action spaces**.

</details>

**2. What Are the Differences and Connections Between REINFORCE and PPO?**

<details>
<summary>💡 Show Answer</summary>

1. **Update Mechanism**:
    - REINFORCE: Performs a one-time update using the data sampled after the complete trajectory is finished.
    - PPO: Reuses the same batch of sampled data for multiple updates, constraining the magnitude of each update via clipping or KL divergence penalties.
2. **Stability and Efficiency**:
    - REINFORCE: Simple to implement, but suffers from high variance and low sample efficiency.
    - PPO: Significantly improves stability and convergence speed by limiting the magnitude of policy updates.
3. **Application Scenarios**:
    - REINFORCE: Often regarded as the foundational policy gradient method, suitable for teaching the principles of reinforcement learning.
    - PPO: More common in practice, as it maintains convergence and stability while utilizing data more efficiently.

</details>

**3. In Stage 3 of RLHF, PPO is the Most Mainstream Reinforcement Learning Algorithm. Why Choose PPO Instead of Other Simpler Policy Gradient Algorithms (such as REINFORCE) or Q-learning Family Algorithms? What Key Role Does the KL Divergence Penalty Term Play in PPO?**

<details>
<summary>💡 Show Answer</summary>

Choosing PPO (Proximal Policy Optimization) as the primary algorithm in the third stage of RLHF is based on its well-balanced tradeoff among training stability, sample efficiency, and implementation simplicity in complex environments like large language models.

Why not choose other algorithms?

1. vs. REINFORCE (Simple Policy Gradient):

- The REINFORCE algorithm is notorious for its high variance. It directly uses rewards from whole sequences obtained via Monte Carlo sampling to update the policy, leading to highly unstable gradient estimates, especially in environments like LLMs with massive action spaces and sparse reward signals. The training process oscillates severely and struggles to converge. PPO significantly reduces variance and stabilizes training by introducing a value function as a baseline and employing an advantage function.

1. vs. Q-learning Family Algorithms (e.g., DQN):

- Value-based algorithms like DQN are primarily designed for discrete and low-dimensional action spaces. They require computing a Q-value for every possible action under each state. For an LLM, the action space at each time step is the entire vocabulary, resulting in an exceptionally massive combinatorial space. Directly applying Q-learning to compute a Q-value for each word is computationally infeasible. As a policy gradient method, PPO directly optimizes in the policy space, making it naturally suitable for such continuous or massive action spaces.

Key Role of the KL Divergence Penalty Term in PPO:

The objective function of PPO contains a highly critical KL divergence penalty term:

\[Objective( \pi_{RL} ) = \mathbb{E} [ Reward ] - \beta \cdot \mathbb{KL}(\pi_{RL} || \pi_{SFT})\]

where $\pi_{RL}$ is the policy currently being optimized, $\pi_{SFT}$ is the initial SFT policy trained in the first stage, and $\beta$ is a hyperparameter. This KL divergence term acts as a "trust region" or "regularization", serving two key purposes:

1. Preventing Policy Collapse: The reward model (RM) is imperfect and inevitably contains exploits/loopholes. Without the KL penalty term, the RL policy would aggressively exploit RM vulnerabilities ("reward hacking") to gain the highest scores, often resulting in gibberish, highly repetitive, or offensive text—known as "mode collapse". By constraining the new policy from drifting too far from the initial, reasonably performing SFT policy, the KL penalty keeps optimization within a "safe" region, preserving the strong language capabilities of the SFT model.
2. Ensuring Exploration Efficiency and Diversity: Maintaining proximity to the SFT model prevents the model from prematurely converging to a local optimum that yields high rewards but poor overall text quality. It encourages the model to explore near the learned, meaningful language distribution rather than jumping into a completely unfamiliar space where the reward model may fail. This helps preserve the diversity and readability of the generated text.

</details>

**4. Please Briefly Describe the PPO Algorithm Workflow and Its Differences from TRPO.**

<details>
<summary>💡 Show Answer</summary>

![Iterative PPO Policy Optimization Loop](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig20_please_briefly_describe_the_pp.png)




**PPO (Proximal Policy Optimization) Algorithm Workflow**:

1. **Sampling (Rollout)**: The current policy $\pi_\theta$ (actor) interacts with the environment (in RLHF scenarios, the "environment" is the given prompt, and the "action" is the generated token sequence) to collect a batch of trajectories (prompt, response, reward);
2. **Advantage Estimation**: The value function (critic) is used to estimate the value of each state $V(s)$, combined with the actual received rewards to calculate the advantage $A(s,a)$ (measuring how much better the action is relative to the "average level"), commonly computed via the GAE (Generalized Advantage Estimation) method;
3. **Constructing PPO Objective Function**: Calculate the ratio of probabilities of taking that action between the new and old policies $ratio = \pi_\theta(a|s) / \pi_{\theta_{old}}(a|s)$, with the objective function given by:

\[
L = \mathbb{E}[\min(ratio \cdot A, clip(ratio, 1-\varepsilon, 1+\varepsilon) \cdot A)]
\]

which takes the minimum between "probability ratio $\times$ advantage" and "clipped probability ratio $\times$ advantage", thereby constraining the single-step update magnitude;

1. **Updating Policy and Value Networks**: Maximize the above objective function via gradient ascent (typically including value function regression loss and entropy regularization), performing multi-epoch mini-batch gradient updates on actor and critic parameters;
2. **Iteration**: Resample using the updated policy and enter the next iteration.

**Differences from TRPO**:

TRPO (Trust Region Policy Optimization) is the predecessor of PPO. Both share the core goal—**limiting the magnitude of policy updates to prevent instability caused by large discrepancies between old and new policies**—but differ in implementation:

1. **Constraint Implementation Method**:
- **TRPO** relies on explicit **constrained optimization**: "maximize the objective function subject to the constraint that the KL divergence between old and new policies does not exceed a threshold $\delta$". This constrained optimization problem requires computing second-order information (such as the Fisher Information Matrix) and solving via conjugate gradient methods, making it **complex to implement and computationally heavy**.
- **PPO** approximates the same effect using a simple first-order method via a **clipped objective function**: "clipping the probability ratio within $[1-\varepsilon, 1+\varepsilon]$", preventing drastic probability changes for any action in a single update step. It thus **replaces TRPO's complex constrained optimization with a simple surrogate objective**, allowing direct optimization via standard stochastic gradient descent (SGD/Adam), which is **much simpler to implement**.

1. **Computational Efficiency**:
- TRPO requires computing second-order derivative information (Hessian matrix or its approximation), incurring high computational cost;
- PPO requires only first-order gradient information, leveraging automatic differentiation in modern deep learning frameworks efficiently, supporting mini-batch multi-epoch updates, and achieving far higher computational efficiency than TRPO.

1. **Performance and Empirical Results**:
- PPO achieves comparable (or superior) stability and sample efficiency to TRPO while being simpler to implement and far more computationally efficient. Consequently, in practical applications (including RLHF), **PPO has almost completely replaced TRPO** as the de facto standard choice.

**Summary**: Both algorithms share the same objective (limiting policy update step size to guarantee stability). TRPO achieves this via complex constrained optimization and second-order methods, whereas PPO approximates this effect via simple objective clipping (a first-order method). PPO has become the vastly more practical choice due to its simplicity and computational efficiency.

</details>

**5. What Is the Workflow of PPO?**

<details>
<summary>💡 Show Answer</summary>

The implementation workflow of PPO includes:
1. **Environment Sampling**: The policy model generates a series of outputs, which are then scored by the reward model.
2. **Advantage Estimation**: The critic model predicts the future cumulative reward of the generated outputs, and the advantage function is estimated via the Generalized Advantage Estimation (GAE) algorithm.
3. **Optimization and Adjustment**: The policy model is optimized based on the advantage function, while the reference model is used to maintain policy update stability.

</details>

**6. What Are the Four Models in PPO and Their Respective Roles?**

<details>
<summary>💡 Show Answer</summary>

PPO involves the cooperative training and inference of four models:
1. **Policy Model (Actor Model)**: Generates model outputs and serves as the final target model of RLHF training. Its gradients are updated during PPO training.
2. **Reward Model**: Outputs reward scores to evaluate the quality of generated content. Its gradients are not updated during PPO training.
3. **Critic Model**: Predicts the value of generated content to assist in optimal action selection. Its gradients are updated during PPO training.
4. **Reference Model**: Provides a baseline/backup model to prevent overly aggressive policy updates. Its gradients are not updated during PPO training.

</details>

**7. Are You Familiar with RLHF-PPO? How Many Models Need to Be Trained and How Many Need to Be Loaded?**

<details>
<summary>💡 Show Answer</summary>

**Models that need to be trained**: **2**—

1. **Actor (Policy Model)**: Parameters are updated during training; this is the final desired artifact;
2. **Critic (Value Model)**: Parameters are updated during training; used to estimate the value function to assist in computing advantages.

**Models that need to be loaded (frozen parameters, not trained)**: **2**—

1. **Reward Model**: Parameters are frozen; scores the generated (prompt, response) pairs;
2. **Reference Model**: Parameters are frozen; used to compute the KL penalty.

**Total number of models loaded into VRAM: 4** (2 need training + maintaining gradient and optimizer states, 2 only require forward inference).

**Supplementary Notes**:
- In practice, to save GPU memory, the Critic sometimes shares the underlying architecture with the Reward Model (or is even initialized from the same model), and the Reference Model is sometimes identical to the Actor's initial checkpoint (i.e., Actor = Reference prior to RL training, with Actor gradually diverging from Reference during training);
- Some engineering implementations share base backbone weights between the Actor and Reference Model, adding only a small number of trainable parameters to the Actor via LoRA to further reduce VRAM consumption (where the Reference model is simply the base model "without LoRA");
- The Critic and Reward Model can also share a backbone network (differing only in their output heads), though in standard implementations they are typically treated as two independent model instances.

**Summary**: In RLHF-PPO, **2 models need to be trained (Actor, Critic)**, and **2 additional models need to be loaded (frozen: Reward Model, Reference Model)**, resulting in a total of **4 models** residing concurrently in VRAM. This is one of the core reasons why PPO has a significantly larger memory footprint compared to DPO.

</details>

**8. What Are the Respective Roles of the Actor Model, Critic Model, Reward Model, and Reference Model in PPO?**

<details>
<summary>💡 Show Answer</summary>

![Interaction of the Four Models in PPO RLHF](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig21_what_are_the_respective_roles.png)




When PPO is used for RLHF, it typically involves four models:

**1. Actor / Policy Model**

- The policy model currently being optimized, which generates responses given input prompts;
- During training, actor parameters are continuously updated with the objective of obtaining higher rewards for generated responses (while constrained by KL divergence penalty from drifting too far from the reference model);
- After training, the actor is the final model deployed for downstream use.

**2. Critic / Value Model**

- Estimates the **value function $V(s)$** for a given state (typically the context at a certain generation step), representing the "expected cumulative reward achievable from the current state";
- Used to calculate the **advantage function $A(s,a) = R - V(s)$** (or computed more precisely via GAE). The advantage function measures "how much better the actual taken action is compared to the average level", serving as a crucial term in the PPO objective function to reduce variance in gradient estimates and improve training efficiency;
- The Critic typically shares part of its underlying structure with the Actor (such as sharing the Transformer backbone with separate output heads), though it can also be an independent model; the critic itself is trained using a regression loss (the error between predicted values and actual returns).

**3. Reward Model (RM)**

- A **pre-trained model that remains frozen during RLHF**, taking (prompt, response) as input and outputting a scalar reward value representing response quality / human preference alignment;
- The reward model itself is typically trained on human-annotated preference data (pairwise comparisons) using formulations like the Bradley-Terry model;
- In PPO training, the reward model scores each response generated by the actor, and this score (after adjustment by the KL penalty) serves as the primary target optimization signal for the actor.

**4. Reference Model**

- Typically the pre-RLHF SFT model (with parameters **frozen, not involved in training**);
- Used to calculate the **KL divergence** between the actor (policy model) and reference model as a "regularization constraint" on the actor, preventing the actor from deviating too far from the behavioral distribution of the original SFT model while optimizing for rewards (preventing catastrophic forgetting and reward hacking);
- The probability distribution output of the reference model is essential for computing the KL penalty term.

**Interplay and Cooperation Among the Four Models**:

In a typical PPO training iteration:

1. The Actor generates a response based on the prompt;
2. The Reward Model scores the (prompt, response) pair to produce a raw reward $R$;
3. The Reference Model computes the probability of the response under the reference policy, which is compared with the Actor's computed probability to derive the KL penalty term, subtracted from $R$ (yielding the adjusted reward $R'$);
4. The Critic estimates state values $V(s)$ at each token position, combining them with the adjusted reward $R'$ to calculate the advantage function $A$;
5. Using PPO's clipped objective function combined with advantage $A$, parameters of both Actor and Critic are updated.

**Summary**: The Actor is the policy being optimized (the final output); the Critic assists in value estimation to reduce training variance and improve efficiency; the Reward Model provides the core optimization signal of "quality scoring"; the Reference Model supplies the KL constraint to preserve training stability and generalization ability. The four models have distinct duties and work synergistically to constitute the complete PPO-RLHF training loop.

</details>

**9. What is the Role of the Critic in the PPO Algorithm?**

<details>
<summary>💡 Show Answer</summary>

The role of the Critic model (value network) is to provide value estimation signals for the Actor (policy network), thereby stabilizing and accelerating policy updates.

- **Compute Advantage**: $A_t = R_t - V_\phi(s_t)$, where the $V_\phi(s)$ output by the Critic estimates the state value.
- **Reduce Variance, Stabilize Training**: Using $V_\phi(s_t)$ as a baseline significantly reduces the variance of the policy gradient.
- **Auxiliary Optimization Objective (Value Loss)**: $L_{value} = (V_\phi(s_t) - R_t)^2$

The total loss of PPO consists of three terms:

$$
	L = L_{clip}^{policy} + c_1 L_{value} - c_2 H[\pi_\theta]
$$

</details>

**10. Why do the Actor and Critic in the PPO Algorithm Have Different Learning Rates?**

<details>
<summary>💡 Show Answer</summary>

The Actor and Critic play different roles in PPO, and setting different learning rates reflects the differences in their training objectives and stability requirements.

**The Actor uses a smaller learning rate (typically 1e-6 to 1e-5)**:
- **Higher Risk**: Each update of the Actor directly alters the policy's action distribution. If an update is too large, the policy may jump into a region with very low rewards (policy collapse). Moreover, since rollout data generated by the new policy serves as the foundation for subsequent training, erroneous updates can cause a chain reaction;
- **PPO Already Has a Clip Mechanism for Safeguarding**: The Clip mechanism limits the step-wise policy shift at the algorithmic level, but if the learning rate is too large, updates even within the clip range can lead to excessively large parameter updates;
- **Difference from SFT**: Learning rates in the SFT stage are typically between 1e-5 and 1e-4, whereas the RL stage requires a smaller learning rate because RL updates fine-tune on an already aligned baseline, where any large deviation might destroy previously acquired language capabilities.

**The Critic can use a relatively larger learning rate**:
- **Simpler Task**: The Critic's objective is to fit a regression function (value function), similar to supervised learning, yielding more stable gradients;
- **No Direct Impact on Action Execution**: Critic updates do not change the policy's behavior; even if the Critic occasionally experiences large fluctuations, it will not immediately cause the policy to collapse;
- **Needs to Catch Up Quickly with Policy Changes**: After each Actor update, the true value of the value function also changes. The Critic needs to quickly refit the value distribution under the new policy, and a relatively larger learning rate facilitates rapid adaptation.

In practice, the Actor's learning rate is typically about 1/3 to 1/10 of the Critic's, with the exact ratio adjusted based on training stability.

</details>

**11. Is the PPO Algorithm On-Policy or Off-Policy?**

<details>
<summary>💡 Show Answer</summary>

PPO is **essentially an on-policy algorithm**, but through techniques such as importance sampling and experience replay, it **partially borrows ideas from off-policy methods**.

- **PPO's original design is on-policy**: It requires updating using data collected by the current policy interacting with the environment.
- **Introduces off-policy characteristics via importance sampling**: It allows performing multiple gradient updates using data from an older policy, while constraining the update magnitude via clipping or KL penalties.

Therefore, PPO can be regarded as a hybrid method that is **"mostly on-policy, slightly off-policy"**.

</details>

**12. What is the Relationship Between the Old and New Policies in the PPO Algorithm?**

<details>
<summary>💡 Show Answer</summary>

In PPO, the relationship between the new policy and the old policy is established through alternating between data collection and policy updates:

1. **Data Collection Phase**: The old policy ($\pi_{old}$) interacts with the environment to collect a batch of trajectories and records action probabilities.
2. **Policy Update Phase**: On the collected data, the policy parameters are updated by optimizing the objective function to obtain $\pi_{new}$, adjusted using importance sampling weights.
3. **Policy Replacement**: After the update completes, the parameters of $\pi_{new}$ are copied to $\pi_{old}$.

</details>

**13. What Problems Arise from Completely Reusing Old Data in PPO?**

<details>
<summary>💡 Show Answer</summary>

Completely reusing old data leads to training instability or even failure, with the core reasons being policy distribution shift and variance explosion of importance weights:
1. **Policy Distribution Shift**: When the policy updates from $\pi_{old}$ to $\pi_{new}$, action selection probabilities may differ significantly.
2. **Mathematical Risks of Importance Weights**: The ratio $w_t = \frac{\pi_{new}(a_t|s_t)}{\pi_{old}(a_t|s_t)}$ may take extreme values.
3. **Consequences of Variance Explosion**: Unstable gradient updates, leading to policy collapse.

</details>

**14. How to Address High Variance and Unstable Training in the PPO Algorithm?**

<details>
<summary>💡 Show Answer</summary>

The main sources of instability in PPO training include: high gradient variance, overly aggressive policy updates, and noisy reward signals. The corresponding solutions are:

**1. Use GAE to Reduce Advantage Estimation Variance**

Replace pure Monte Carlo (MC) returns with Generalized Advantage Estimation (GAE, $\lambda=0.95$) to achieve a balance between bias and variance. A reasonable value of $\lambda$ can reduce the gradient estimation variance several-fold.

**2. Importance Weight Clipping (Clip Ratio)**

The core design of PPO: Clip $\pi_\theta/\pi_{\theta_old}$ to $[1-\varepsilon, 1+\varepsilon]$ to prevent a single update from causing an excessively large policy change, keeping the training within a "trust region." Typically, $\varepsilon=0.2$.

**3. Reward Normalization and Clipping**
- Apply z-score normalization to rewards across all responses within a batch to eliminate the scale effect of absolute rewards;
- Clip reward values (e.g., restricted to $[-5, 5]$) to prevent extreme reward values from dominating the gradient update direction.

**4. KL Divergence Penalty**

Subtract the KL divergence between the policy and reference model, $\beta \cdot KL(\pi_\theta || \pi_ref)$, from the reward to prevent the policy from deviating excessively from the SFT initialization point, retaining the foundational capabilities of the language model.

**5. Entropy Regularization**

Add a policy entropy bonus term to the loss function to prevent the policy from prematurely collapsing into a deterministic policy, maintaining exploration capability and mitigating entropy collapse.

**6. Advantage Function Normalization**

Apply z-score normalization (mean 0, standard deviation 1) to advantage values within a mini-batch to render gradient scales across different prompts comparable and stabilize update magnitudes.

**7. Lower Learning Rate + Early Stopping**

The learning rate during the RL phase should be far smaller than in SFT (typically on the order of 1e-6). Combining this with Early Stopping (terminating the current update iteration early if the KL divergence exceeds a threshold) prevents over-reusing old data and comprehensively controls training stability.

</details>

**15. What Does "Proximal" Mean in PPO? How to Prevent the Model's Generalization Capability from Degrading Beyond Fine-Tuning Datasets? How to Prevent the Model from Converging to a Single Type of High-Reward Answer?**

<details>
<summary>💡 Show Answer</summary>

![KL Divergence Penalty Behavior over Training Steps](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig22_what_does_proximal_mean_in_ppo.png)




**Meaning of "Proximal"**:

"Proximal" in PPO refers to **constraining the difference between the new policy and the old policy to remain within a "proximal" (small-magnitude) range after each policy update**, meaning that parameter updates in each step should not cause drastic shifts in policy. This is achieved via the clipping mechanism $clip(ratio, 1-\varepsilon, 1+\varepsilon)$ in the objective function—restricting the probability ratio between the new and old policies to an interval close to 1; updates beyond this range yield no additional objective gains, thereby suppressing excessively large policy updates. The core motivation of "proximal" is that policy gradient methods are extremely sensitive to step size: overly large update steps can cause a drastic drop in policy performance that is difficult to recover from (since subsequent sampled data originates from this "degraded" new policy, compounding the issue). Keeping updates within the "proximal" range guarantees training stability and monotonic improvement (or at least avoids severe degradation).

**How to Prevent Generalization Capability Degradation Beyond Fine-Tuning Datasets**:

1. **KL Divergence Penalty Term**: Add a KL divergence penalty between the policy model and the reference model (typically the SFT model) into PPO's reward function, such that the actually optimized reward is:

\[
R'(s,a) = R(s,a) - \beta \cdot KL(\pi_\theta(\cdot|s) || \pi_{ref}(\cdot|s))
\]

This penalty term constrains the policy model from deviating too far from the reference model, **preserving the general capabilities of the reference model (learned on broader pre-training/SFT data)**, preventing the model from over-specializing and losing performance on other types of problems merely to earn higher rewards on the RLHF training distribution.

1. **Coverage of Training Data**: Ensure that the prompt dataset used for RLHF training inherently possesses high diversity and wide coverage, narrowing the gap between the "training distribution" and the "practical application distribution."

2. **Continuous Monitoring on General Evaluation Sets**: Periodically evaluate model performance during training on general capability evaluation benchmarks (tasks independent of RLHF training data), detecting generalization drop trends early to adjust the KL coefficient $\beta$ or stop training early.

**How to Prevent the Model from Converging to a Single Type of High-Reward Answer (i.e., "Reward Model Hacking" / "Mode Collapse")**:

The model might discover that a specific response pattern (such as a fixed opening phrase, an overly sycophantic tone, or a specific format) consistently receives high scores from the reward model, thereby inclining to generate this "high-reward yet repetitive/low-quality" response across all inputs. This is a manifestation of reward hacking, and countermeasures include:

1. **Entropy Regularization**: Include the entropy of the policy distribution as a regularization term in PPO's objective function to encourage maintaining output diversity/randomness, preventing the policy from shrinking too quickly into a single, deterministic output mode.

2. **KL Penalty (as above)**: Constraining the policy from deviating too far from the reference model also indirectly limits policy collapse into "exotic" output patterns, since the output distribution of the reference model itself is diverse.

3. **Improve Reward Model Robustness**:
- Use an **ensemble of multiple reward models**, taking an aggregated score (such as minimum or average) across multiple reward models for the same output, reducing the risk of exploiting vulnerabilities in a single reward model;
- Include negative examples targeting known "reward hacking patterns" in the reward model training data to enhance the reward model's ability to discern such "superficially pleasing yet hollow" outputs;
- Periodically update/retrain the reward model with newly sampled data (generated by the policy model), enabling the reward model to keep pace with the policy model's "evolution" and detect newly emerging hacking patterns.

1. **Limit Training Steps / Early Stopping**: Reward hacking often manifests progressively as training steps increase (as the policy gradually finds and exploits loopholes in the reward model). Appropriately controlling the total number of training steps and closely monitoring generation diversity and quality allows timely stopping when signs of mode collapse appear.

2. **Human Sampling Inspection**: Periodically inspect samples generated by the policy model manually to check for obvious "stereotyped/repetitive" output tendencies, supplementing automated metrics.

**Summary**: "Proximal" in PPO refers to constraining the magnitude of policy updates to ensure training stability; the core to preventing generalization degradation is **using KL divergence penalties to constrain the policy from deviating too far from the reference model**; the core to preventing convergence to a single high-reward pattern is **entropy regularization to maintain exploration + KL penalty + improving reward model robustness (ensembling, targeted negative examples, continuous updates) + controlling training steps and human inspection**, with multiple mechanisms working together to balance "optimizing reward" and "maintaining diversity/generalization."

</details>

**16. In RLHF (Referring to the OpenAI InstructGPT Paper), What is the Loss Function for Training the LLM?**

<details>
<summary>💡 Show Answer</summary>

In the InstructGPT paper, the objective function optimized during the PPO phase (i.e., the "reward", which is the quantity to be maximized in PPO) is:

\[
objective(\phi) = E_{(x,y)~\pi_{\phi}^RL}[r_\theta(x,y) - \beta·log(\pi_{\phi}^RL(y|x) / \pi^SFT(y|x))] + \gamma·E_{x~D_pretrain}[log(\pi_{\phi}^RL(x))]
\]

Explanation of each term:

1. **$r_\theta$(x,y)**: The scalar reward given by the reward model for (prompt x, response y), serving as the core optimization signal;

2. **-$\beta$·log($\pi_{\phi}^{RL}$(y|x) / $\pi^{SFT}$(y|x))**: The **KL penalty term** (negated because the entire objective is to be maximized; a larger KL divergence indicates a greater deviation from the SFT model, making a negative contribution to the objective). This term constrains the policy model $\pi_{\phi}^{RL}$ from deviating too far from the SFT model ($\pi^{SFT}$, i.e., the reference model), where $\beta$ is the KL penalty coefficient;

3. **$\gamma$·$E_{x \sim D_{pretrain}}$[log($\pi_{\phi}^{RL}$(x))]**: A unique term in the InstructGPT paper—**pretraining loss mixing**. While performing RL training, it additionally computes the log-likelihood of language modeling on the original pretraining data distribution $D_{pretrain}$ (this term is effectively the negative of the pretraining loss, so maximizing it here is equivalent to minimizing standard language modeling loss during RL training). The role of this term is to **further mitigate the model's performance degradation on general NLP tasks during RLHF ("alignment tax")**, where $\gamma$ is the coefficient for this mixing term.

The entire objective function is optimized via the **PPO algorithm** (i.e., the actual "loss function" is PPO's clipped objective function, where the "reward" signal is the $r_\theta(x,y) - \beta \cdot KL$ term in the objective above, and both the value function and advantage estimation are computed based on this adjusted reward).

**Summary**: The target optimized during the RL phase in InstructGPT = **Reward given by reward model - $\beta$ × (KL divergence between policy model and SFT model) + $\gamma$ × (Language modeling log-likelihood on pretraining data distribution)**, optimized via the PPO algorithm; the KL penalty term constrains the model from deviating too far from SFT, while the pretraining loss mixing term mitigates general capability degradation (a relatively unique design in the InstructGPT paper, not necessarily included in all subsequent RLHF implementations).

</details>

**17. In RLHF-PPO, What is the Reward Design, and What is Generalized Advantage Estimation?**

<details>
<summary>💡 Show Answer</summary>

**Reward Design**:

The actual "reward" signal used for PPO updates in RLHF-PPO is not the raw output of the reward model, but is adjusted as follows:

- **Token-level Reward Allocation**: For a generated response sequence $y_1, \dots, y_T$, typically the **scalar score $r_\theta(x,y)$ from the reward model is assigned only to the last token (sequence end position)**, while the "environmental reward" for all preceding tokens is 0;
- **KL Penalty as a Per-Token Reward Term**: At each token position $t$, an additional penalty term $-\beta \cdot (\log \pi_{\theta}(y_t|s_t) - \log \pi_{ref}(y_t|s_t))$ is added, which is the log probability difference between the current policy and the reference model at that token multiplied by coefficient $\beta$; this term exists **per token** (unlike the reward model score which appears only at the end);
- **Final Reward Used to Compute Advantage Function** = KL penalty at token position $t$ + (if last token) reward model score.

This design ensures that: the "overall quality score" provided by the reward model operates sparse-wise at the end of the sequence, whereas the KL penalty provides dense, per-token signals related to "whether it deviates from the reference model", together forming the reward sequence upon which PPO optimization relies.

**"Absolute Advantage Estimation" (Here Understood as Generalized Advantage Estimation, GAE)**:

The term "absolute advantage estimation" in the question is another phrasing/translation for **GAE (Generalized Advantage Estimation)**. GAE is used to estimate the advantage function $A(s_t, a_t)$, defined as:

\[
A_t^{GAE} = \sum_{l=0}^{\infty} (\gamma\lambda)^l \cdot \delta_{t+l}
\]

where $\delta_{t} = r_t + \gamma \cdot V(s_{t+1}) - V(s_t)$ is the TD error (temporal difference error), $\gamma$ is the discount factor, and $\lambda$ is the smoothing coefficient of GAE (between 0 and 1, controlling the bias-variance tradeoff):
- When $\lambda=0$, $A_t = \delta_{t}$ (using only a one-step TD error, low variance but high bias);
- When $\lambda=1$, $A_t$ degenerates to a Monte Carlo estimate ($\sum \gamma^l r_{t+l} - V(s_t)$, low bias but high variance);
- When $\lambda$ takes an intermediate value, it achieves a balance between bias and variance, which is the commonly used setting in practice.

The core value of GAE is to **leverage the estimate of the value function $V(s)$ to gradually "propagate" and allocate the "final sparse reward" to each position in the sequence**, providing effective, low-variance advantage estimation signals for each PPO update step.

**Summary**: Reward in RLHF-PPO = "reward model score at the end of the sequence (sparse) + per-token KL penalty (dense)"; "Absolute/Generalized Advantage Estimation (GAE)" is a method combining value function estimation and multi-step TD errors to convert this (partially sparse) reward signal into advantage estimates with low variance and controllable bias at each position in the sequence for PPO policy updates.

</details>

**18. Loss Function Formula for RLHF-PPO Training**

<details>
<summary>💡 Show Answer</summary>

![Clipped Surrogate Objective in PPO](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig23_loss_function_formula_for_rlhf.png)




The total loss function of RLHF-PPO is typically a combination of the following three components:

**1. PPO Clipped Policy Loss (Clipped Surrogate Objective)**

\[
L^{CLIP}(\theta) = -E_t[\min(ratio_t \cdot A_t, clip(ratio_t, 1-\varepsilon, 1+\varepsilon) \cdot A_t)]
\]

where $ratio_t = \pi_{\theta}(a_t|s_t) / \pi_{\theta_{old}}(a_t|s_t)$, and $A_t$ is the advantage function estimate (GAE). Note that a negative sign is added in front because the original PPO objective is to be "maximized", which requires negation when converted to a "loss" (to be minimized).

**2. Value Function Loss**

\[
L^{VF}(\theta) = E_t[(V_\theta(s_t) - V_t^{target})^2]
\]

Or adopting a clipped version:

\[
L^{VF_clip}(\theta) = E_t[\max((V_\theta(s_t) - V_t^{target})^2, (clip(V_\theta(s_t), V_{old}(s_t)-\varepsilon, V_{old}(s_t)+\varepsilon) - V_t^{target})^2)]
\]

where $V_t^{target}$ is typically the return estimate used in GAE computation ($A_t + V_{old}(s_t)$).

**3. Entropy Bonus**

\[
S(\pi_{\theta})(s_t) = -\sum_{a} \pi_{\theta}(a|s_t) \cdot \log \pi_{\theta}(a|s_t)
\]

**Total Loss (Objective to Minimize)**:

\[
L^{total}(\theta) = L^{CLIP}(\theta) + c_1 \cdot L^{VF}(\theta) - c_2 \cdot S(\pi_{\theta})
\]

where $c_1$ and $c_2$ are coefficient weights for the value loss and entropy bonus ($c_2$ has a minus sign in front because higher entropy is better, equivalent to "subtracting" entropy in the loss).

**Supplementary—Where KL Penalty Is Manifested**: In most LLM-PPO implementations, the KL penalty is not added directly to $L^{total}$ as an independent term in the loss function; rather, it is **already incorporated into the reward signal $r_t$ (as per-token KL penalty being part of the reward)**, which subsequently affects the calculation of $A_t$. Therefore, the KL penalty "implicitly" affects $L^{CLIP}$ through modifying the advantage function $A_t$ calculation, rather than appearing as an explicit extra term in the loss formula (though some implementation choices do include KL penalty as an explicit additional term in the loss function, and both implementation strategies exist).

**Summary**: Loss function of RLHF-PPO = **Clipped policy loss (based on advantage function $A_t$ and new-to-old policy probability ratio $ratio_t$) + $c_1$ × value function regression loss − $c_2$ × policy entropy**, where the calculation of advantage function $A_t$ (via GAE) already implicitly incorporates KL penalty consideration (KL penalty participates in GAE computation as a per-token reward signal).

</details>

**19. In PPO, What is Generalized Advantage Estimation?**

<details>
<summary>💡 Show Answer</summary>

![Generalized Advantage Estimation (GAE) Data Flow](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig24_in_ppo_what_is_generalized_adv.png)




(This question overlaps with the second half of the previous question; "absolute advantage estimation" refers to GAE, Generalized Advantage Estimation, and is further explained here from the perspective of the "role" of the advantage function.)

**Core Role of the Advantage Function $A(s,a)$**:

The advantage function measures "how much more return action $a$ achieves in state $s$ compared to the 'average level' in that state (i.e., the value function $V(s)$)":

\[
A(s,a) = Q(s,a) - V(s)
\]

where $Q(s,a)$ is the state-action value function (the expected return after taking action $a$), and $V(s)$ is the state-value function (the expected return of all possible actions under that state, i.e., the "baseline").

**Why is an advantage function needed instead of directly using return $Q(s,a)$ or reward $r$**:

- Directly using raw returns as the "weights" for policy gradients leads to high variance (the range of return values under different states can vary dramatically);
- After subtracting a baseline $V(s)$, $A(s,a)$ represents the "extra gain" relative to the average level of that state—if $A>0$, it indicates that this action is "better than average" and its probability should be increased; if $A<0$, its probability should be decreased. This "relative" metric **significantly reduces the variance of the policy gradient estimate** without introducing bias (subtracting a baseline does not change the gradient direction in expectation).

**Calculation of GAE (Generalized Advantage Estimation)**:

\[
A_t^GAE(\gamma,\lambda) = \Sigma_{l=0}^{T-t-1} (\gamma\lambda)^l · \delta_{t+l}，where \delta_{t} = r_t + \gamma V(s_{t+1}) - V(s_t)
\]

By interpolating between "1-step TD error (low variance, high bias)" and "full Monte Carlo return (high variance, low bias)" via the parameter $\lambda$, GAE is currently the standard method for estimating advantage functions in PPO implementations.

**Summary**: The advantage function $A(s,a) = Q(s,a) - V(s)$ measures how good a certain action is relative to the "average level" of that state, serving as a core tool for variance reduction in policy gradient methods. GAE is an advantage function estimation method that utilizes value functions and multi-step TD errors to balance bias and variance through the parameter $\lambda$, representing the standard practice for computing $A_t$ in PPO.

</details>

**20. What is the loss function of the critic model in PPO?**

<details>
<summary>💡 Show Answer</summary>

The loss function of the Critic (value model) is a **regression loss (Mean Squared Error, MSE)**, whose objective is to make the Critic's predicted value $V_\theta(s_t)$ as close as possible to the "true" (estimated) return $V_t^target$:

\[
L^VF(\theta) = E_t[(V_\theta(s_t) - V_t^target)^2]
\]

where **$V_t^target$** is typically computed as follows:

\[
V_t^target = A_t^GAE + V_old(s_t)
\]

That is, "the $V_{old}(s_t)$ estimated by the old value function plus the advantage $A_t$ calculated by GAE." This sum approximates the estimate of "the actual (multi-step discounted) return received at that state" (which is a bootstrap-form target combining TD principles).

**Clipped Version (Value Clipping, see Question 401)**:

To prevent excessive update magnitudes of the Critic (consistent with the motivation of policy clipping), a clipped version is often adopted in practice:

\[
V_clip(s_t) = clip(V_\theta(s_t), V_old(s_t) - \varepsilon, V_old(s_t) + \varepsilon)
\]

\[
L^VF_clip(\theta) = E_t[max((V_\theta(s_t) - V_t^target)^2, (V_clip(s_t) - V_t^target)^2)]
\]

That is, taking the larger of the "unclipped MSE" and the "clipped MSE" as the loss, ensuring that the Critic's predictions do not deviate too far from the old predictions in a single update while still updating in the direction of the target value.

**Summary**: The loss function of the Critic is essentially the **mean squared error (MSE) between the predicted value and the target return value (calculated via GAE + old value estimate)**. In practice, it is often paired with a clipping mechanism (limiting single-step update magnitudes) to improve training stability, which shares the same design philosophy as the policy network's clipping objective.

</details>

**21. What functions do key techniques such as advantage normalization, value function clipping, and entropy regularization play in PPO?**

<details>
<summary>💡 Show Answer</summary>

**1. Advantage Normalization**

- Normalize the advantage values $A$ calculated within a batch by subtracting their mean and dividing by their standard deviation (making the mean 0 and variance 1);
- Role/Effects:
- **Reduce gradient variance and stabilize training**: Unnormalized advantage functions can have vastly different value ranges across different batches and training stages (depending on the scale of rewards), leading to unstable gradient updates; after normalization, the range of advantage values is relatively consistent, making the magnitude of gradient updates much more controllable;
- **Make hyperparameters like learning rate more transferable across tasks/stages**: Because advantage functions are standardized to a unified scale, the same learning rate produces more consistent update magnitudes across different scenarios.

**2. Value Function Clipping**

- When updating the value function (Critic), similar to clipping in the policy objective, clipping is applied to the value function update: restricting the new value prediction $V_{new}(s)$ within a neighborhood around the old value prediction $V_{old}(s)$ (such as $[V_{old} - \varepsilon, V_{old} + \varepsilon]$). The value function loss takes the maximum of the "unclipped loss" and the "clipped loss" (taking the maximum serves to continue penalizing excessively large deviations);
- Role/Effects:
- Similar to the motivation for policy clipping, **preventing the value function from changing too drastically in a single update**. Drastic fluctuations in the value function lead to unstable advantage estimation, which in turn adversely affects the quality of policy updates, forming an unstable feedback loop. Clipping the value function helps make the entire training process (joint update of policy and value function) smoother.

**3. Entropy Regularization (Entropy Regularization / Bonus)**

- Add the negative entropy of the policy distribution as an extra loss term to the total PPO loss function (i.e., encouraging higher entropy). Higher entropy indicates a more "dispersed" output distribution for the policy (higher uncertainty, favoring exploration);
- Role/Effects:
- **Encourage exploration and prevent the policy from prematurely converging to deterministic, single output modes** (avoiding the "mode collapse" issue described in Question 398);
- In RLHF scenarios, moderate entropy helps maintain the diversity of generated content, preventing the model from producing highly similar, formulaic responses for all inputs;
- The coefficient for entropy regularization usually needs careful tuning—too large will cause the policy to be overly random and fail to converge to high-quality outputs, while too small will fail to prevent mode collapse.

**4. Other Common Techniques (Extended)**

- **Reward Normalization/Clipping**: Normalize raw reward values output by the reward model (e.g., subtract mean, divide by standard deviation) or clip them within a certain range to prevent abnormally large reward values from destabilizing training (refer to Question 400);
- **Gradient Clipping**: Clip the gradient norm of policy and value networks, serving as a general technique for training stability;
- **Learning Rate Scheduling**: Typically adopt a small learning rate combined with warmup/decay.

**Summary**: The shared goal of these techniques is to **improve the stability of the PPO training process, reduce the negative impacts of various numerical fluctuations (advantage function scale, value function estimation, reward scale, etc.) on training, and achieve a balance between "optimizing reward" and "maintaining exploration/diversity" through entropy regularization**—advantage normalization and value function clipping primarily serve "stability", while entropy regularization primarily serves "exploration/preventing mode collapse". The combination of these three forms an important engineering foundation for PPO to train stably in complex scenarios such as RLHF.

</details>

**22. How does PPO solve the classic sparse reward and reward hacking problems in RL?**

<details>
<summary>💡 Show Answer</summary>

![PPO Critic Loss Convergence and Value Clipping](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig25_how_does_ppo_solve_the_classic.png)




**Sparse Reward Problem and PPO's Coping Strategy**:

In RLHF scenarios, "sparse reward" manifests as: the reward model usually gives a single scalar reward only for the **complete generated sequence** (the entire response), rather than providing independent reward signals for each token in the sequence—that is, the reward signal appears only once at the "end of the sequence", and every intermediate step (generation of each token) has no direct reward feedback. This is a classic sparse reward / delayed reward problem.

PPO (in specific RLHF implementations, such as InstructGPT) mitigates this problem through the following ways:

1. **Introduction of Value Function (Critic) and Time Difference (TD) / GAE**: The Critic learns to estimate the value $V(s)$ of each intermediate state (i.e., when generated up to a certain token), which represents "how much (discounted) reward is expected from this position onward"; through GAE (Generalized Advantage Estimation), even though the final reward appears only at the end of the sequence, training the value function (regression based on TD error) can gradually "propagate" this "delayed reward signal" back to earlier positions in the sequence, allowing each position to obtain a meaningful advantage estimate $A(s,a)$, thereby providing effective gradient signals for policy updates at every step and mitigating the sparsity issue where "only the last step has a signal".

2. **Reward Discounting and Allocation Mechanism**: Although the original reward is given only at the end of the sequence, in implementation it is typically treated as being "obtained at the last token", with the "immediate reward" for all preceding tokens set to 0 (some implementations also add a KL penalty at each step as a "per-token" signal, ensuring each step has at least a signal related to the degree of deviation from the reference model caused by the KL penalty); combined with the bootstrap estimation of the value function, overall this ensures every position in the sequence receives a non-zero training signal.

**Reward Hacking Problem and PPO's Coping Strategy**:

Reward hacking refers to the phenomenon where the policy model finds "exploit" output patterns that allow an (imperfect) reward model to give high scores, but which do not actually align with true human preferences. PPO's own training mechanism (clipped objective function) primarily addresses the problem of "training stability" and cannot directly "solve" reward hacking caused by inaccuracies in the reward model itself. However, the overall RLHF framework (including auxiliary mechanisms commonly used in PPO training) mitigates this problem from multiple levels:

1. **KL Divergence Penalty**: Restricts the policy from deviating too far from the reference model. Reward hacking often manifests as the policy straying from the normal distribution toward some "extreme" pattern, and the KL penalty mechanically suppresses such large deviations;

2. **Entropy Regularization**: Maintains output diversity, preventing the policy from prematurely collapsing into a fixed pattern that "cheats" the reward model;

3. **Reward Normalization and Clipping (see Question 401)**: Normalizes reward values to prevent abnormally high reward values (which may be the exact signal of "being hacked") from exerting an excessively large impact on training;

4. **Early Stopping and Continuous Monitoring**: Reward hacking is usually a process that gradually "worsens" as training progresses. Through continuous manual sample inspection and monitoring on independent evaluation sets, training can be stopped in a timely manner when detected;

5. **Improving the Reward Model Itself**: This is the more fundamental solution (such as ensemble reward models, adversarial data augmentation, etc., see Question 398)—the PPO algorithm itself merely "faithfully" optimizes the given reward signal, and the quality of the reward signal itself determines the potential space for "hacking".

**Summary**: PPO mainly mitigates the sparse reward problem by **introducing a value function + GAE to "propagate" the sparse reward signal from the end of the sequence to every position in the sequence**, providing effective gradients for step-by-step updates. Regarding reward hacking, the PPO algorithm itself cannot fundamentally "solve" it (as this depends on the quality of the reward model), but mechanisms commonly used in the RLHF training framework, such as **KL penalty, entropy regularization, reward normalization, continuous monitoring, and early stopping**, can suppress and delay reward hacking to a certain extent. A fundamental resolution still requires improving the accuracy and robustness of the reward model itself.

</details>

**23. Is PPO MC or TD? What about GRPO?**

<details>
<summary>💡 Show Answer</summary>

**PPO**: The update of the value function in PPO (as well as the calculation of the advantage function GAE) is essentially a **combination of TD (Temporal Difference) methods and MC (Monte Carlo) methods (interpolated via the $\lambda$ parameter of GAE)**:
- When $\lambda=0$ in GAE, it degenerates to pure **TD(0)** (single-step TD error);
- When $\lambda=1$, it degenerates to pure **Monte Carlo** (using cumulative returns of full trajectories);
- In practice, $\lambda$ often takes an intermediate value (e.g., 0.95), taking the form of **TD($\lambda$)**, which is a hybrid of TD and MC.

Because PPO explicitly trains a value function $V(s)$ and computes TD errors via bootstrapping (i.e., using $V(s_{t+1})$ to estimate future returns), **PPO essentially belongs to TD methods (more accurately, methods under the TD($\lambda$) framework)**, though its behavior can be adjusted between MC and TD(0) depending on $\lambda$.

**GRPO**: GRPO **removes the value function (Critic)**, and its "advantage" is obtained by **sampling a group of complete responses for the same prompt and directly performing group normalization using the (final) rewards of these complete trajectories**—i.e., $A_i$ = ($r_i$ - mean($group_rewards$)) / std($group_rewards$), where $r_i$ is the **final reward of the complete response (entire trajectory)**, without using any value function bootstrap estimation.

Therefore, **GRPO is essentially closer to MC (Monte Carlo) methods**—it relies on actual returns (rewards) of complete trajectories rather than value-function-based "estimates/bootstrapping" of future returns.

**Summary**: **PPO is TD (more accurately TD($\lambda$), combining characteristics of both TD and MC, adjustable via GAE's $\lambda$ parameter)**; **GRPO, having no value function and directly using final rewards of complete trajectories for relative group normalization, is closer to MC methods**.

</details>

**24. What are the pain points of PPO?**

<details>
<summary>💡 Show Answer</summary>

Although PPO performs excellently in LLM alignment, it suffers from significant engineering and algorithmic pain points:

**Algorithmic Pain Points**:
- **Hyperparameter Sensitivity**: Multiple key hyperparameters, such as KL coefficient $\beta$, clip range $\varepsilon$, GAE $\lambda$, and entropy coefficient, are coupled together, making tuning difficult and requiring readjustments across different tasks;
- **Training Instability**: PPO is an on-policy RL method where minor changes in the policy affect subsequent rollout distributions, and erroneous updates can trigger chain-reaction instabilities; flaws in the reward model itself can be exploited by the policy (reward hacking);
- **Credit Assignment Difficulty**: Rewards are given only at the end of the sequence; how to reasonably allocate sequence-level rewards to each token remains an open question in both theory and practice.

**Engineering Implementation Pain Points**:
- **Four-Model Architecture**: Concurrently requires four models: Actor, Critic, Reward Model, and Reference Model, demanding over 4$\times$ the VRAM of SFT;
- **Rollout as a Bottleneck**: Autoregressive generation (rollout) of LLMs is a purely sequential operation that cannot be highly parallelized, and data quickly becomes "stale" after each rollout round, leading to low overall GPU utilization (30–50%);
- **Complexity of Training-Inference Separation**: Parallel strategies differ between training and inference (rollout) for the Actor. Weight synchronization and precision alignment between the two frameworks (train-eval mismatch issue) add system complexity;
- **Multi-Process Coordination**: The four models may run on different GPUs, requiring complex multi-node multi-GPU communication and scheduling mechanisms;
- **Debugging Difficulty**: During training, multiple metrics such as reward, KL divergence, entropy, and clip ratio must be monitored simultaneously; any anomaly can stem from multiple possible causes.

It is precisely these pain points that drove the emergence of simplified schemes like GRPO (eliminating Critic) and DPO (eliminating RM and online sampling).

</details>

**25. What does Rollout mean?**

<details>
<summary>💡 Show Answer</summary>

Rollout is a core operation in reinforcement learning with specific connotations in LLM scenarios. Understanding it is crucial for comprehending PPO/GRPO.

**General RL Definition**:

Rollout refers to starting from an initial state $s_0$, where the agent continuously interacts with the environment according to the current policy $\pi$, step-by-step generating a complete state-action-reward trajectory: $s_0 \xrightarrow{a_0} s_1 \xrightarrow{a_1} s_2 \xrightarrow{a_2} \cdots \xrightarrow{a_{T-1}} s_T$.

**Mapping in LLM RL**:
- **Initial State $s_0$**: Given prompt (input instruction);
- **Action $a_t$**: Generating one token at each timestep;
- **State Transition**: New state $s_{t+1}$ = prefix + newly generated token;
- **Trajectory Termination**: Generating an EOS token or reaching the maximum length;
- **Reward**: Typically, after the entire response generation is completed, a sequence-level reward is provided at once by the reward model (sparse reward).

**Position of Rollout in the Training Pipeline**:

Every PPO/GRPO iteration begins with Rollout—using the current policy model to perform inference (autoregressive sampling) on a batch of prompts, generating responses, collecting (prompt, response, reward, per-step log-prob) data, and then proceeding to the training phase. For each prompt in GRPO, $G$ responses need to be sampled, making the rollout compute volume $G$ times that of PPO.

**Rollout as the Training Bottleneck**:

LLM autoregressive generation is a sequential operation (each step depends on previous steps), making the compute efficiency of the rollout phase far lower than the parallel training phase, typically accounting for 60–80% of the entire training loop time. This is also the core reason why asynchronous RL frameworks (such as AReaL) and efficient inference engines (vLLM, SGLang) are introduced into RL training.

</details>

**26. Explain what the rollout--generation gap is.**

<details>
<summary>💡 Show Answer</summary>

The so-called rollout--generation gap essentially refers to the systematic discrepancy between the policy used to generate trajectories (rollout) and the policy that is actually optimized and updated. Once this discrepancy becomes non-negligible, a nominally on-policy algorithm quietly degrades to off-policy in terms of training dynamics.

The performance degradation does not originate from the PPO update formula itself, but rather from the decoupling of the rollout distribution and the optimization distribution. Solution strategies include: explicitly modeling the behavior policy, controlling rollout staleness, and introducing terms into the objective function to suppress policy drift.

</details>

**27. What problems exist when using rewards in PPO?**

<details>
<summary>💡 Show Answer</summary>

1. **Reward Model Bias**: Inconsistency between the training data distribution and the actual policy-generated data distribution.
2. **Reward Sparsity and Credit Assignment Issues**: Reward signals are sparse or delayed.
3. **Reward Hacking**: The policy obtains high rewards by "deceiving" the reward model.
4. **Non-stationarity of Reward Models**: Continuous shift in the policy distribution causes the evaluation criteria of the reward model to become invalid.
5. **Reward Scale Issues**: The output range of the reward model is unstable.
6. **Multi-Objective Conflict**: When the reward model needs to balance multiple objectives, it may be difficult to allocate weights reasonably.

</details>

**28. What does clip ratio = 1 signify during LLM RL?**

<details>
<summary>💡 Show Answer</summary>

In PPO/GRPO, an importance ratio is calculated at each step:

$$
	r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)}
$$


When clipped_ratio=1.0, it indicates that $r_t$ for all tokens fell outside $[1-\epsilon, 1+\epsilon]$ and were forcibly clipped. That is to say, the model update direction is completely untrustworthy, rendering the training virtually invalid.

</details>

**29. What causes clip ratio = 1 in RL, and what are the specific actions to take?**

<details>
<summary>💡 Show Answer</summary>

When the clip ratio (the proportion of clipped tokens) approaches 1, meaning that importance weights for almost all tokens fall outside the clip range, it indicates that the difference between the new and old policies is excessively large, signaling severe training issues.

**Common Causes**:
- **Learning Rate Too Large**: Excessive update magnitude per step causes the policy to deviate sharply from the old policy within just a few steps;
- **Too Many Update Epochs**: Each batch of rollout data is used for too many epochs of updates, accumulating drift;
- **Gradient Explosion**: Numerical issues in the loss function (e.g., NaN/Inf) lead to abnormally large parameter updates;
- **Reward Scale Too Large**: Reward normalization is absent, with extreme reward values generating extreme gradients;
- **Unstable Advantage Estimation**: Numerical issues occur in GAE or group normalization (e.g., division by zero due to zero variance);
- **Weights Not Properly Synchronized**: An older version of weights was used during rollout while updates had already completed in the training framework.

**Specific Actionable Solutions**:
1. **Lower the Learning Rate**: Reduce Actor learning rate from 1e-5 to 5e-6 or 1e-6 to decrease single-step update magnitude;
2. **Reduce PPO Epochs**: Decrease from 4 epochs to 2 or even 1 epoch;
3. **Set Gradient Clipping**: Use `max_grad_norm` (e.g., 1.0) to prevent gradient explosion;
4. **Reward Normalization**: Perform z-score normalization on all rewards within the batch to eliminate abnormally large reward values;
5. **Strengthen KL Constraint**: Increase $\beta$ to limit the drift velocity of the policy algorithmically;
6. **Check Weight Synchronization Logic**: Confirm that weights in the rollout framework (e.g., vLLM) are correctly synchronized with the latest training parameters;
7. **Add Early Stopping**: If KL divergence exceeds a threshold (e.g., 0.3) during an update round, immediately terminate the current update round and start a new rollout.

</details>

**30. What causes the importance ratio between new and old policies to remain 1 during PPO training?**

<details>
<summary>💡 Show Answer</summary>

If you observe that it concentrates around 1 or is numerically equal to 1, the sole mathematical implication is that the new policy's probability distribution on these state-action pairs is virtually identical to the old policy's. Reasons include:
1. Update magnitude is near zero (advantage close to 0, learning rate too small)
2. KL/clip constraints are too strong, directly "locking" the policy
3. The new and old policies are actually the "same model"
4. Data reuse count is set to 1

</details>

**31. Why does RLHF need to concatenate prompt and response together when calculating log_prob?**

<details>
<summary>💡 Show Answer</summary>

1. **Generation Process vs. Training Objective**: The generation phase involves forward sampling without backpropagating gradients; the training phase requires computing loss based on log-prob and backpropagating.
2. **Loss Cannot Be Computed "While Decoding"**: During decoding, the model samples the next token, and sampling is a non-differentiable operation.
3. **From a Mathematical Perspective**: Training requires computing log-prob and backpropagating across the entire response sequence, thus requiring a fresh forward pass on the concatenated prompt + response after decoding.

</details>

**32. If the Available Human Preference Dataset Has High Quality but Limited Quantity, Should One Use PPO or DPO?**

<details>
<summary>💡 Show Answer</summary>

![DPO Loss and Reference Model Constraint](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig26_if_the_available_human_prefere.png)




In the scenario of "high-quality but limited-quantity preference data", **DPO is generally more recommended** for the following reasons:

**1. PPO relies on training a reward model with good generalization capability, whereas the quality of the reward model depends heavily on the quantity of preference data**

- In the PPO pipeline, preference data is first used to train a **reward model**, which needs to accurately score **various new, unseen outputs** generated by the policy model during training (i.e., the reward model must possess strong generalization capability to assign reasonable scores to out-of-distribution samples);
- If the quantity of preference data is limited, the trained reward model may have **insufficient generalization capability**, yielding inaccurate scores for new samples produced during PPO iterations, which leads to **reward hacking** (where the policy model finds "loopholes" in the reward model, generating content that the reward model misjudges as high-scoring but is actually of low quality)—a problem that is amplified when data quantity is scarce.

**2. DPO trains directly on the given preference data without requiring an additional reward model or online generation of new samples to "test" the reward model's generalization**

- DPO's training objective is directly optimized based on (prompt, chosen, rejected) triplets, with the model learning to "increase the relative probability of chosen over rejected";
- This approach utilizes the information in existing preference data more directly, **avoiding the additional source of uncertainty regarding "whether the reward model is accurate on out-of-distribution samples"**, making it relatively more reliable when data volume is limited.

**3. PPO's online sampling and multi-model training pipeline imposes higher resource and engineering demands; when data volume is insufficient to fully train a reward model, the "cost-effectiveness" of this added complexity and instability is low**

- When data volume is limited, rather than using the scarce data to train a potentially inaccurate reward model and then optimizing through a complex and unstable PPO process, it is better to **directly and efficiently leverage DPO to learn from this high-quality data**.

**4. High-quality but limited-quantity data is better suited for "offline" methods**

- Offline preference optimization methods such as DPO (and its variants like IPO, KTO, etc.) are specifically designed for scenarios where "large-scale online sampling is infeasible / high-quality reward models cannot be trained", allowing them to more fully squeeze information out of limited high-quality data.

**Exceptions**:

- If, in addition to this high-quality preference data, there are sufficient **extra resources** (such as a large pool of unlabeled prompts available for online sampling, and high training/engineering budgets can be tolerated), one may consider using this high-quality data to first train a relatively reliable reward model (which may remain accurate within its covered distribution), and then combine it with PPO for further optimization; however, in this case, close attention is usually paid to the reward model's behavior on OOD samples (e.g., by increasing KL penalty constraints to prevent the policy from straying too far).

**Summary**: In the typical scenario where preference data is of "high quality but limited quantity", **DPO is the more suitable choice because it does not require training an independent reward model, does not rely on out-of-distribution generalization of the reward model, and is more stable and efficient to train**; PPO is better suited for scenarios with abundant preference data (sufficient to train a reward model with good generalization) and plentiful online sampling resources.

</details>

**33. What Happens If the KL Penalty Coefficient $\beta$ Is Too Large or Too Small?**

<details>
<summary>💡 Show Answer</summary>

The KL penalty term $\beta \cdot KL(\pi_{RL} || \pi_{SFT})$ is used to constrain the RL policy from deviating too far from the initial SFT policy.

**$\beta$ is too large**:
- Policy updates are severely suppressed, making it almost impossible for the model to learn new behaviors from the RM's reward signal. The RL phase exists in name only, leaving the final model very close to the SFT model with **poor alignment effect**.

**$\beta$ is too small**:
- The policy can freely stray far from the SFT distribution, easily leading to:
- **Reward Hacking**: The model finds vulnerabilities in the RM, generating content that receives high RM scores but is actually low-quality or unnatural.
- **Language Capability Degradation / Mode Collapse**: Deviating too far from the original language distribution, resulting in syntactically weird text and a sharp drop in diversity.
- **Catastrophic Forgetting**: Knowledge and capabilities acquired during pre-training are destroyed.

In practice, $\beta$ usually requires **adaptive adjustment** (e.g., dynamically tuned according to the current KL divergence magnitude) to strike a balance between "learning new preferences" and "maintaining base capabilities".

</details>

**34. What Are the Differences Between PPO and DPO in Computational Efficiency, Implementation Complexity, and Training Stability?**

<details>
<summary>💡 Show Answer</summary>

![System Architecture Comparison: PPO vs. DPO](https://img.pricenode.uk/llm_interview_questions/english/chapter5/fig27_what_are_the_differences_betwe.png)




**Computational Efficiency**:

- **PPO**: During training, it needs to simultaneously maintain and run **4 models** (actor/policy model, critic/value model, reward model, and reference model). Each training step involves: (1) generating responses with the actor (sampling, high inference overhead); (2) scoring generated content with the reward model; (3) computing KL penalty with the reference model; and (4) estimating values with the critic. The overall computational overhead and GPU memory consumption are large, making training relatively slow.
- **DPO**: Trains directly on "preference pair" (chosen, rejected) data without requiring online sampling or separate reward and value models. It **only requires the policy model and a reference model (typically the initial SFT model, whose output probabilities can even be pre-cached to further reduce online computation)**. In essence, it is a pairwise data-based supervised learning process similar to SFT, yielding much higher computational efficiency than PPO.

**Implementation Complexity**:

- **PPO**: Highly complex to implement, involving multi-model orchestration, online sampling and generation (handling engineering details such as KV caching and variable-length batching), Generalized Advantage Estimation (GAE), value function training, reward normalization, and hyperparameter tuning (clip range, KL coefficient, entropy coefficient, etc.). The overall engineering complexity and debugging difficulty are high, and the requirements for distributed training systems are more demanding.
- **DPO**: Relatively simple to implement; it is essentially a **contrastive loss function** (derived from the Bradley-Terry preference model). The training workflow closely resembles standard supervised fine-tuning (on preference datasets), involving no online generation and requiring no separate reward or value models, which drastically reduces engineering and debugging complexity.

**Training Stability**:

- **PPO**: Because it involves online reinforcement learning (where the continuously changing policy alters the sampling distribution), interactions among multiple models, and potential inaccuracies in the reward model itself (risk of reward hacking), the training process is **relatively unstable**. Problems like reward hacking, KL divergence explosion, and training collapse frequently arise, requiring meticulous hyperparameter tuning and extensive training tricks (e.g., normalization, clipping, entropy regularization, etc., as described in Question 401) to stabilize training.
- **DPO**: As an offline contrastive learning method, its training dynamics closely resemble standard supervised learning, making it **generally more stable**. It is less sensitive to hyperparameters, produces smoother training curves, and avoids the common training collapse issues of PPO. However, DPO has its own stability considerations (e.g., choice of the $\beta$ parameter; if too small, the model may overfit to preference data or deviate too far from the reference model).

**Summary**:

\begin{table}[h]



| Dimension | PPO | DPO |
| --- | --- | --- |
| Computational Efficiency | Low (4 models, online sampling) | High (2 models, offline training) |
| Implementation Complexity | High (full RL pipeline) | Low (similar to contrastive supervised learning) |
| Training Stability | Lower (multiple instability factors) | Higher (closer to supervised learning) |


\end{table}

Overall, **DPO significantly outperforms PPO in computational efficiency, implementation complexity, and training stability**, which is a key reason DPO has been widely adopted since its introduction. However, as an online reinforcement learning method, PPO theoretically possesses the capability to "continuously explore and use a reward model to score arbitrary generated content." In certain scenarios (especially when the reward model is of high quality and one desires the model to explore superior policies beyond the training data distribution), PPO still retains unique advantages. The choice between the two requires trade-offs based on the specific scenario (refer to Question 397).

</details>

