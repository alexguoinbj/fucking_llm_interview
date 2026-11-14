# 3.1 Mathematical Foundations — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 14 questions

**1. What is KL Divergence?**

<details>
<summary>💡 Show Answer</summary>

![Mathematical illustration of KL Divergence showing distribution discrepancy and information loss.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig01_kl_divergence.png)




Kullback-Leibler Divergence (KL Divergence) is an asymmetric measure of the difference between two probability distributions, proposed by Solomon Kullback and Richard Leibler in 1951. Its definition is as follows:


$$
	D_{KL}(P \| Q) = \sum_{x} P(x) \log \frac{P(x)}{Q(x)}
$$


For continuous random variables, the integral form of KL divergence is:


$$
	D_{KL}(P \| Q) = \int_{-\infty}^{+\infty} p(x) \log \frac{p(x)}{q(x)} \, dx
$$


where $P$ denotes the target distribution (true distribution) and $Q$ denotes the reference distribution (approximate distribution). KL divergence reflects the degree of information loss when transforming from $Q$ to $P$. It can be understood as: the extra average number of bits required to encode data that actually follows distribution $P$ when using distribution $Q$ for approximation.

**Important Properties:**
- **Non-negativity**: $D_{KL}(P \| Q) \geq 0$, with equality holding if and only if $P = Q$ (guaranteed by Gibbs' inequality).
- **Asymmetry**: In general, $D_{KL}(P \| Q) \neq D_{KL}(Q \| P)$, so KL divergence is not a true distance metric.
- **Violation of Triangle Inequality**: This is another reason why it does not constitute a "distance" in a metric space.
- **Information Theory Interpretation**: KL divergence equals the difference in average bits between encoding under the true distribution $P$ using an optimal code for $P$ versus using an optimal code designed for $Q$.

**Relationship with Information Entropy:**

Let $H(P)$ be the information entropy of distribution $P$, and $H(P, Q)$ be the cross-entropy of $P$ relative to $Q$. Then:


$$
	D_{KL}(P \| Q) = H(P, Q) - H(P)
$$


That is, KL divergence equals cross-entropy minus information entropy, reflecting the extra information cost incurred by using $Q$ to substitute $P$.

</details>

**2. What is the Difference Between KL Divergence and Cross-Entropy?**

<details>
<summary>💡 Show Answer</summary>

Both KL divergence and cross-entropy are tools in information theory used to measure differences between probability distributions, but they have fundamental differences.

**Mathematical Relationship:**

Let the true distribution be $P$ and the predicted distribution be $Q$. Then:

$$
	Entropy: & \quad H(P) = -\sum_{x} P(x) \log P(x) \\
	Cross-Entropy: & \quad H(P, Q) = -\sum_{x} P(x) \log Q(x) \\
	KL Divergence: & \quad D_{KL}(P \| Q) = \sum_{x} P(x) \log \frac{P(x)}{Q(x)} = H(P, Q) - H(P)
$$


Therefore, KL divergence and cross-entropy differ by a constant term $H(P)$ (the entropy of the true distribution). In the optimization process, minimizing cross-entropy is equivalent to minimizing KL divergence.

**Comparison of Main Differences:**

\begin{table}[h]



| **Property** | **KL Divergence** | **Cross-Entropy** |
| --- | --- | --- |
| Asymmetry | Yes ($D_{KL}(P\|Q) \neq D_{KL}(Q\|P)$) | Yes |
| Minimum Value | 0 (when $P = Q$) | $H(P)$ (when $P = Q$) |
| Includes True Distribution Entropy | Yes | No |
| Applications in Machine Learning | Variational inference, Knowledge Distillation, etc. | Classification loss function |
| Valid Distance Metric | No | No |


	\caption{Comparison between KL Divergence and Cross-Entropy}
\end{table}

**Differences in Machine Learning Applications:**
- **Cross-Entropy**: More commonly used as loss functions in classification tasks due to concise computation. When the true distribution $P$ is fixed, $H(P)$ is a constant, so minimizing cross-entropy is equivalent to minimizing KL divergence.
- **KL Divergence**: Commonly used in regularization terms of Variational Autoencoders (VAEs), Knowledge Distillation, policy update constraints in reinforcement learning (e.g., PPO, TRPO), and other scenarios where explicit measurement of the difference between two variable distributions is required.

</details>

**3. What is Cross-Entropy Loss? What is Its Physical Meaning?**

<details>
<summary>💡 Show Answer</summary>

![Comparison of Cross-Entropy Loss and MSE Loss gradient behavior.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig02_ce_vs_mse.png)




The Cross-Entropy Loss function originates from information theory and is used to measure the difference between two probability distributions:


$$
	H(p, q) = -\sum_{i} p_i \log q_i
$$


where $p_i$ is the true label distribution, and $q_i$ is the model's predicted probability distribution.

**Binary Cross-Entropy:**

When the problem is binary classification ($y \in \{0, 1\}$), cross-entropy reduces to:


$$
	L = -\left[ y \log \hat{y} + (1 - y) \log (1 - \hat{y}) \right]
$$


where $\hat{y}$ is the model's output probability for the positive class.

**Physical and Information-Theoretic Meaning:**

- Cross-entropy $H(p, q)$ represents: under the true distribution $p$, the **average number of encoding bits** required when using an optimal code designed for $q$.
- When the predicted distribution $q$ is perfectly aligned with the true distribution $p$, $H(p, q) = H(p)$ (the entropy of the true distribution), achieving the minimum loss.
- The greater the difference between the two distributions, the larger the cross-entropy, indicating that the model requires more "extra information" to describe the true labels.
- In supervised learning, true labels are typically one-hot encoded, i.e., $p_i = 1$ for the correct class and $p_j = 0$ for other classes. At this point, cross-entropy reduces to:

    $$
    L = -\log q_{y}
    $$

    where $y$ is the index of the true class, which penalizes only the logarithm of the probability assigned by the model to the correct class.

**Gradient Properties:**

When combined with Sigmoid/Softmax activation functions, the gradient of the cross-entropy loss takes a concise form:


$$
	\frac{\partial L}{\partial z} = \hat{y} - y
$$


where $z$ is the logit (output prior to activation). This gradient does not suffer from the vanishing gradient problem, which is one of the primary reasons why cross-entropy outperforms MSE for classification tasks.

</details>

**4. Why Use Cross-Entropy Loss Instead of Mean Squared Error (MSE) in Classification Problems?**

<details>
<summary>💡 Show Answer</summary>

In classification problems, the cross-entropy loss function offers significant advantages over Mean Squared Error (MSE) for the following reasons:

**1. Suitability for Probability Distributions**

The output of a classification problem is mapped to probability values via Softmax/Sigmoid. Cross-entropy naturally measures the difference between two probability distributions, aligning directly with the task goal; in contrast, MSE measures numerical differences, which does not match the semantics of probabilities.

**2. Vanishing Gradient Problem**

Taking binary classification as an example, when using the Sigmoid activation function $\sigma(z) = \frac{1}{1+e^{-z}}$:

- The gradient using **MSE** is:

    $$
    \frac{\partial L_{MSE}}{\partial z} = (\hat{y} - y) \cdot \sigma(z)(1 - \sigma(z))
    $$

    When the predicted value $\hat{y}$ approaches 0 or 1, $\sigma(z)(1 - \sigma(z)) \approx 0$, causing the gradient to vanish near zero and leading to extremely slow training.
- The gradient using **Cross-Entropy** is:

    $$
    \frac{\partial L_{CE}}{\partial z} = \hat{y} - y
    $$

    The gradient depends solely on the prediction error and does not suffer from vanishing gradients, resulting in more stable and efficient training.

**3. Severity of Penalty for Confident Errors**

Cross-entropy contains a logarithmic function. When the model outputs high confidence for an incorrect class (e.g., predicted probability approaching 1 when the ground truth is 0), $-\log(\epsilon) \to +\infty$, resulting in an immense penalty; conversely, the maximum penalty of MSE is bounded ($(1-0)^2 = 1$), failing to effectively correct highly confident erroneous predictions.

**4. Statistical Perspective: Maximum Likelihood Estimation**

For classification problems, assuming labels follow a categorical distribution, cross-entropy loss is equivalent to performing **Maximum Likelihood Estimation (MLE)** on the model parameters, backed by solid statistical theory; MSE, on the other hand, corresponds to a Gaussian noise assumption, which does not match the data generation process of classification tasks.

**Summary Comparison:**

\begin{table}[h]



| **Property** | **Cross-Entropy** | **MSE** |
| --- | --- | --- |
| Applicable Task | Classification | Regression |
| Gradient Vanishing | None (with Softmax/Sigmoid) | Present (in saturation regions) |
| Penalty for Confident Errors | Immense (logarithmic penalty) | Bounded (maximum 1) |
| Statistical Foundation | Maximum Likelihood Estimation | Gaussian Noise Assumption |


	\caption{Comparison between Cross-Entropy and MSE}
\end{table}

</details>

**5. What Does the Classification Loss Function (Softmax) Look Like in Multi-Class Classification Problems?**

<details>
<summary>💡 Show Answer</summary>

In multi-class classification problems, the output of the neural network's final layer, $\mathbf{z} = (z_1, z_2, \ldots, z_n)$ (referred to as logits), is typically transformed into a probability distribution via the Softmax function:


$$
	p_i = softmax(z_i) = \frac{e^{z_i}}{\sum_{j=1}^{n} e^{z_j}}, \quad i = 1, 2, \ldots, n
$$


Subsequently, cross-entropy is used to calculate the loss (referred to as Softmax Cross-Entropy Loss):


$$
	L = -\sum_{i=1}^{n} y_i \log p_i
$$


where $n$ is the number of classes, $y_i$ is the ground-truth label for the $i$-th class (typically one-hot encoded), and $p_i$ is the predicted probability for the $i$-th class.

**Simplified Form Under One-Hot Labels:**

Since $y$ is a one-hot vector, assuming the correct class is $c$ (i.e., $y_c = 1$, and all others are 0), the loss simplifies to:


$$
	L = -\log p_c = -\log \frac{e^{z_c}}{\sum_{j=1}^{n} e^{z_j}}
$$


**Gradients with Respect to Logits:**

The gradient of the Softmax cross-entropy loss with respect to logit $z_i$ takes a very concise form:


$$
	\frac{\partial L}{\partial z_i} = p_i - y_i
$$


which is simply the difference between the predicted probability and the ground-truth label, making backpropagation computations highly efficient.

**Label Smoothing:**

In practical applications, to prevent the model from overfitting to the training set and outputting overly extreme probabilities, one-hot labels are often smoothed:


$$
	y_i^{smooth} = (1 - \varepsilon) \cdot y_i + \frac{\varepsilon}{n}
$$


where $\varepsilon$ is the smoothing factor (typically set to 0.1), thereby improving the calibration of the model's outputs.

</details>

**6. What If the Exponentiation in Softmax Exceeds the Floating-Point Range?**

<details>
<summary>💡 Show Answer</summary>

![Numerically Stable Softmax computation pipeline preventing overflow.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig03_stable_softmax.png)




When there are large values in the logit vector, directly computing $e^{z_i}$ may lead to floating-point overflow; when the values are small, it may cause underflow, which in turn leads to division by zero.

**Numerically Stable Softmax:**

By leveraging the shift invariance of Softmax, multiply both the numerator and denominator by $e^{-\max(\mathbf{z})}$:


$$
	softmax(z_i) = \frac{e^{z_i}}{\sum_j e^{z_j}} = \frac{e^{z_i - M}}{\sum_j e^{z_j - M}}, \quad M = \max_j(z_j)
$$


After subtracting the maximum value, all exponent terms satisfy $z_i - M \leq 0$, so that $e^{z_i - M} \in (0, 1]$, completely avoiding overflow. Meanwhile, the maximum term becomes 1, preventing all terms from simultaneously underflowing to 0.

**Numerically Stable Log-Softmax:**

In practical training, we usually need to compute $\log p_i$ (for cross-entropy loss). If Softmax is computed first and then the logarithm is taken, additional numerical errors may be introduced. A more stable approach is to directly compute Log-Softmax:


$$
	\log softmax(z_i) = z_i - M - \log \sum_j e^{z_j - M}
$$


where $\log \sum_j e^{z_j - M}$ is the log-sum-exp operation, which can be computed efficiently and stably.

**Code Example (NumPy Implementation):**

\begin{lstlisting}[style=promptstyle]
	import numpy as np

	def stable_softmax(z):
	# Subtract max value for numerical stability
	z_shifted = z - np.max(z)
	exp_z = np.exp(z_shifted)
	return exp_z / np.sum(exp_z)

	def stable_log_softmax(z):
	# Numerically stable log-softmax
	M = np.max(z)
	log_sum_exp = M + np.log(np.sum(np.exp(z - M)))
	return z - log_sum_exp

	# Example
	z = np.array([1000.0, 1001.0, 1002.0])  # Large values scenario
	print(stable_softmax(z))       # [0.0900, 0.2447, 0.6652]
	print(stable_log_softmax(z))   # [-2.4076, -1.4076, -0.4076]
\end{lstlisting}

**Why Does Shift Invariance Hold?**

Let $M$ be an arbitrary constant:


$$
	\frac{e^{z_i - M}}{\sum_j e^{z_j - M}} = \frac{e^{z_i} \cdot e^{-M}}{\sum_j e^{z_j} \cdot e^{-M}} = \frac{e^{z_i}}{\sum_j e^{z_j}}
$$


The common factor $e^{-M}$ cancels out; therefore, the shift operation does not change the output values of Softmax, but merely improves numerical stability.

**Implementation in Deep Learning Frameworks:**

Mainstream frameworks (such as PyTorch and TensorFlow) incorporate the aforementioned numerical stability handling internally when implementing `nn.CrossEntropyLoss` or `tf.nn.softmax_cross_entropy_with_logits`. They directly accept logits as input without requiring users to manually compute Softmax, thereby guaranteeing both numerical stability and computational efficiency.

</details>

**7. What Is the Computational Complexity of Matrix Multiplication? Where Is It Most Time-Consuming in Transformer?**

<details>
<summary>💡 Show Answer</summary>

When multiplying two matrices $A \in \mathbb{R}^{m \times k}$ and $B \in \mathbb{R}^{k \times n}$, the computational complexity of the standard algorithm is $O(mkn)$.

In Transformer, the primary matrix multiplication overhead is concentrated in the following parts:

1. **$QK^\top$ Computation in Self-Attention**: The shapes are $(B, H, L, d_k) \times (B, H, d_k, L)$, with a complexity of $O(L^2 d_k)$, which grows quadratically with sequence length $L$ and serves as the main bottleneck in long-sequence scenarios.
2. **Weighted Sum of Attention Scores and $V$**: The complexity is also $O(L^2 d_k)$.
3. **Linear Transformation in FFN Layer**: The shapes are typically $(B, L, d) \times (d, 4d)$, with a complexity of $O(L \cdot d \cdot 4d)$. For long sequences, the total computation of FFN often exceeds Attention.
4. **Input/Output Projections (QKV Projection, Output Projection)**: The complexity is $O(L \cdot d^2)$.

**Summary**: For short sequences, FFN is the primary overhead; for long sequences, the $O(L^2)$ Attention computation becomes the bottleneck, which is also the core motivation behind optimization schemes such as FlashAttention and linear Attention.

</details>

**8. What Is SVD Decomposition? What Are Its Applications in Large Models?**

<details>
<summary>💡 Show Answer</summary>

**Singular Value Decomposition (SVD)** decomposes any matrix $W \in \mathbb{R}^{m \times n}$ into:

$$W = U \Sigma V^\top$$

where $U \in \mathbb{R}^{m \times m}$ and $V \in \mathbb{R}^{n \times n}$ are orthogonal matrices, $\Sigma \in \mathbb{R}^{m \times n}$ is a diagonal matrix, and the diagonal elements $\sigma_1 \geq \sigma_2 \geq \cdots \geq 0$ are called singular values.

**Low-Rank Approximation**: Taking the top $r$ largest singular values yields $\hat{W} = U_r \Sigma_r V_r^\top$, which is the optimal rank-$r$ approximation (Eckart--Young theorem).

**Applications in Large Models:**

1. **LoRA**: Freezes the pre-trained weights $W_0$ and uses low-rank matrices $\Delta W = BA$ ($B \in \mathbb{R}^{m \times r}$, $A \in \mathbb{R}^{r \times n}$, $r \ll \min(m,n)$) to approximate weight updates. Its theoretical foundation is precisely the low-rank hypothesis—the parameter change during model fine-tuning is essentially low-rank.
2. **Model Compression**: Performs SVD on the weight matrix and truncates small singular values to reduce parameter count.
3. **Analyzing Internal Model Structure**: Uses singular value distribution to diagnose the "effective rank" of the weight matrix and determine whether redundancy exists.

</details>

**9. What Is Maximum Likelihood Estimation (MLE)? What Is Its Relationship with Language Model Training?**

<details>
<summary>💡 Show Answer</summary>

**Maximum Likelihood Estimation (MLE)** is a parameter estimation method: given observed data $\mathcal{D} = \{x_1, x_2, \ldots, x_N\}$, it searches for parameters $\theta$ that maximize the joint probability (likelihood) of the data:

$$\hat{\theta}_{MLE} = \arg\max_\theta \prod_{i=1}^N p_\theta(x_i) = \arg\max_\theta \sum_{i=1}^N \log p_\theta(x_i)$$

Taking the logarithm converts it into a sum of log-likelihoods, which facilitates calculation and ensures numerical stability.

**Relationship with Language Model Training:**

The training objective of autoregressive language models is to minimize cross-entropy loss:

$$\mathcal{L} = -\frac{1}{T} \sum_{t=1}^T \log p_\theta(x_t \mid x_{<t})$$

This is equivalent to performing maximum likelihood estimation on the training corpus—maximizing the model's log-probability of the true token sequence. Therefore, **language model pre-training is essentially performing MLE**, where the cross-entropy loss is the negative log-likelihood.

</details>

**10. What Is the Chain Rule? What Is the Mathematical Essence of Backpropagation?**

<details>
<summary>💡 Show Answer</summary>

**The chain rule** is a basic rule in calculus for differentiating composite functions: if $y = f(g(x))$, then:

$$\frac{dy}{dx} = \frac{dy}{dg} \cdot \frac{dg}{dx}$$

Generalized to multi-layer compositions (such as neural networks): let $L = f_n \circ f_{n-1} \circ \cdots \circ f_1(x)$, then the gradient of the loss with respect to the $k$-th layer parameter $\theta_k$ is:

$$\frac{\partial L}{\partial \theta_k} = \frac{\partial L}{\partial f_n} \cdot \frac{\partial f_n}{\partial f_{n-1}} \cdots \frac{\partial f_{k+1}}{\partial f_k} \cdot \frac{\partial f_k}{\partial \theta_k}$$

**The Essence of Backpropagation**: It is an efficient implementation of the aforementioned chain rule. By **propagating gradients layer by layer from the output layer to the input layer**, it reuses intermediate computation results and avoids redundant calculations, reducing the complexity from exponential to linear.

- **Forward Propagation**: Computes and caches the activation values of each layer.
- **Backward Propagation**: Uses the cached activation values to compute gradients layer by layer according to the chain rule.

The vanishing gradient problem (where gradients near the input layer in deep networks approach zero) is essentially an exponential decay caused by repeated multiplication of numerous gradients less than 1 in the chain rule.

</details>

**11. What Are Vanishing and Exploding Gradients? How to Understand Them from a Mathematical Perspective?**

<details>
<summary>💡 Show Answer</summary>

In deep networks, the gradient of the loss $L$ with respect to the parameters of the $l$-th layer requires chain multiplication across $(n-l)$ layers:

$$\frac{\partial L}{\partial \theta_l} = \frac{\partial L}{\partial h_n} \cdot \prod_{k=l+1}^{n} \frac{\partial h_k}{\partial h_{k-1}} \cdot \frac{\partial h_l}{\partial \theta_l}$$

where $\frac{\partial h_k}{\partial h_{k-1}} = W_k^\top \cdot diag(\sigma'(h_{k-1}))$.

- **Vanishing Gradient**: If the spectral norm (largest singular value) of the Jacobian matrix for each layer is $< 1$, the gradient exponentially decays towards 0 after repeated multiplication, making it almost impossible to update parameters in deep layers. Derivatives of Sigmoid/Tanh activation functions approaching 0 in saturation regions are classic causes.
- **Exploding Gradient**: If the spectral norm is $> 1$, the gradient grows exponentially after repeated multiplication, leading to unstable training (loss spikes, NaN).

**Countermeasures in Large Models:**

1. **Residual Connections**: $h_l = f(h_{l-1}) + h_{l-1}$, allowing gradients to flow back directly through identity paths, mitigating vanishing gradients.
2. **Layer Normalization**: Stabilizes activation distributions across layers and prevents saturation.
3. **Gradient Clipping**: Restricts the upper bound of the gradient norm to prevent explosion.
4. **Proper Weight Initialization** (such as Xavier and He initialization): Ensures that the spectral norm of the Jacobian matrix for each layer is close to 1 in the initial state.

</details>

**12. What Is Entropy? What Is the Relationship Between Perplexity and Entropy in Language Models?**

<details>
<summary>💡 Show Answer</summary>

**Entropy** measures the uncertainty of a random variable. For a discrete distribution $p$:

$$H(p) = -\sum_{x} p(x) \log p(x)$$

The higher the entropy, the more uniform the distribution and the higher the uncertainty; the lower the entropy, the more concentrated the distribution and the easier it is to predict.

**Perplexity (PPL)** is a standard evaluation metric for language models, defined as the exponent of the average negative log-likelihood per token on the test set:

$$PPL = \exp\left(-\frac{1}{T} \sum_{t=1}^T \log p_\theta(x_t \mid x_{<t})\right)$$

**Relationship Between the Two:**

- $-\frac{1}{T} \sum_t \log p_\theta(x_t \mid x_{<t})$ is precisely the empirical estimate of the **cross-entropy** of the model distribution relative to the true data distribution.
- Therefore, $PPL = 2^{H(p, q)}$ (base 2) or $e^{H(p,q)}$ (base $e$), meaning **perplexity is the exponent of cross-entropy**.
- When the model perfectly fits the true distribution, the cross-entropy equals the true entropy of the data, and PPL achieves its theoretical lower bound.
- Intuitively, PPL can be understood as the average number of equally likely options from which the model needs to guess the next token at each position. PPL = 10 means that the model is choosing among 10 options on average.

</details>

**13. What Is the Core Idea of the Adam Optimizer? Why Is It More Suitable for Large Model Training Than SGD?**

<details>
<summary>💡 Show Answer</summary>

![Adam Optimizer update pipeline integrating momentum and adaptive learning rates.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig04_adam_optimizer_pipeline.png)




**Adam (Adaptive Moment Estimation)** combines the ideas of Momentum and Adaptive Learning Rate (RMSProp):

1. **First-Moment Estimation** (Momentum): Computes the exponential moving average of gradients to smooth the update direction:
    $$m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t$$
2. **Second-Moment Estimation** (Adaptive Learning Rate): Computes the exponential moving average of squared gradients to estimate the variance of gradients:
    $$v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2$$
3. **Bias Correction**: Corrects the bias in the initial stage: $\hat{m}_t = m_t / (1-\beta_1^t)$, $\hat{v}_t = v_t / (1-\beta_2^t)$
4. **Parameter Update**:
    $$\theta_t = \theta_{t-1} - \alpha \cdot \frac{\hat{m}_t}{\sqrt{\hat{v}_t} + \epsilon}$$

**Why Is It More Suitable for Large Models Than SGD?**

- **Adaptive Learning Rate**: Gradient scales across different parameters vary dramatically (e.g., Embedding layer vs. FFN layer). Adam independently adjusts the effective learning rate for each parameter, whereas SGD struggles to accommodate this with a uniform learning rate.
- **Insensitivity to Hyperparameters**: Adam is more forgiving regarding the initial setting of learning rates, lowering hyperparameter tuning costs.
- **Sparse Gradient Friendly**: In scenarios with sparse parameter updates such as Embeddings, adaptive learning rates maintain larger step sizes for parameters that are updated infrequently.
- **Training Stability**: The smoothing effect of momentum reduces gradient noise, which is particularly important in large-batch training.

In practical applications, **AdamW** (which decouples weight decay from gradient updates) is commonly used and has become the de facto standard for LLM training.

</details>

**14. What Is Bayes' Theorem? How Is It Manifested in Language Models?**

<details>
<summary>💡 Show Answer</summary>

**Bayes' Theorem** describes how to update the belief in the probability of an event after obtaining new evidence:

$$P(\theta \mid \mathcal{D}) = \frac{P(\mathcal{D} \mid \theta) \cdot P(\theta)}{P(\mathcal{D})}$$

where:
- $P(\theta)$: Prior probability (Prior), the initial belief about the parameters.
- $P(\mathcal{D} \mid \theta)$: Likelihood (Likelihood), the probability of observed data given the parameters.
- $P(\theta \mid \mathcal{D})$: Posterior probability (Posterior), the updated belief about the parameters after observing data.
- $P(\mathcal{D})$: Evidence (Evidence), a normalizing constant.

**Manifestations in Language Models:**

1. **MLE vs. MAP**: Standard pre-training performs MLE (maximum likelihood estimation), which is equivalent to Bayesian estimation with no prior; adding L2 regularization (Weight Decay) is equivalent to MAP (maximum a posteriori estimation) with a Gaussian prior on parameters.
2. **Bayesian Interpretation of In-Context Learning**: Few-shot examples can be understood as providing "evidence" to the model, updating its "posterior understanding" of the task, which is essentially implicit Bayesian inference.
3. **Bayesian Perspective of RAG**: Retrieved documents serve as prior knowledge, which, combined with the user query (likelihood), generates a more accurate posterior response.
4. **Reward Modeling in RLHF**: Modeling human preference data is essentially estimating the posterior of the human preference distribution.

</details>

