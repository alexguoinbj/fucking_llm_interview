# 1.2 LLM Architectures and Principles — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 1: Basic Knowledge of Large Models · 107 questions

**1. Why are Large Language Models predominantly Decoder-only architectures?**

<details>
<summary>💡 Show Answer</summary>

When exploring why current Large Language Models (LLMs) universally adopt Decoder-only architectures, analysis can be conducted from several perspectives:

1. **Full-Rank Property of the Attention Mechanism**: The causal attention mechanism used in Decoder-only architectures forms a lower triangular matrix. This structure guarantees full rank of the matrix (i.e., non-zero determinant), thereby conferring stronger modeling capability on the model. In contrast, the bidirectional attention mechanism in Encoder architectures may lead to rank deficiency of the matrix, affecting the model's expressiveness.

2. **Difficulty of Pre-training Tasks**: In Decoder-only architectures, the model accesses limited information at each position during pre-training, increasing the difficulty of predicting the next token. However, when model scale is sufficiently large and data is abundant, Decoder-only models exhibit a higher upper bound in learning and generalization.

3. **In-Context Learning Ability**: Decoder-only architectures perform superiorly in in-context learning because prompt and demonstration information can be viewed as an implicit fine-tuning of model parameters. This characteristic gives Decoder-only architectures a distinct advantage in few-shot learning scenarios.

4. **Efficiency Considerations**: Decoder-only architectures support the reuse of KV-Cache, which is more friendly to multi-turn dialogue tasks. In this architecture, the representation of each token depends only on previous inputs, which is difficult to achieve in Encoder-Decoder architectures.

5. **Zero-Shot Performance**: Decoder-only models demonstrate better zero-shot performance without additional tuning data. In contrast, Encoder-Decoder architectures require multi-task fine-tuning on a certain amount of labeled data to unleash their best performance.

In summary, Decoder-only architectures have become the mainstream choice for current LLM design due to their advantages in training efficiency, engineering implementation, and theoretical foundations.

</details>

**2. What is the difference between Prefix LM and Causal LM?**

<details>
<summary>💡 Show Answer</summary>

The key difference lies in the attention mask. They are defined as follows:

- **Prefix LM (Prefix Language Model)**: A learnable task-related prefix is added to the beginning of the input sequence, and output is generated using both this prefix and the input sequence. This approach guides the model to generate outputs tailored to specific tasks.
- **Causal LM (Causal Language Model)**: Also known as an autoregressive language model, it predicts the next token based on previously generated tokens. When generating text, the model can only generate subsequent parts based on what has already been generated and cannot access future information.

</details>

**3. Differences and Pros/Cons of Prefix LM, Causal LM, and Encoder-Decoder**

<details>
<summary>💡 Show Answer</summary>

**Prefix LM**: The decoder can access the entire input sequence (including the prefix and previously generated output), thereby achieving better context understanding. Representative models: T5, UniLM. Advantages: Reduces modifications to pre-trained model parameters and lowers overfitting risk. Disadvantages: May be constrained by the prefix representation length.

**Causal LM**: Employs masking to ensure that when generating each word, only preceding words (including the current one) are considered. Representative models: GPT series. Advantages: Can generate flexible text adaptable to various generation tasks. Disadvantages: Cannot access future information, which may lead to inconsistent or erroneous content.

**Encoder-Decoder**: The encoder uses bidirectional attention while the decoder uses unidirectional attention. Representative models: Transformer, Flan-T5, BART. Advantages: Handles tasks with different input and output sequence lengths. Disadvantages: More complex model architecture with higher training and inference computational overhead.

</details>

**4. Briefly introduce the Transformer network?**

<details>
<summary>💡 Show Answer</summary>

At the heart of most modern LLMs is the Transformer architecture. Departing from traditional Recurrent Neural Networks (RNNs), this neural network excels at understanding long-range dependencies in sequences, making it particularly well-suited for language processing tasks. The Transformer consists of two sub-components:

- **Encoder**: This part processes input text, breaking it down into a series of encoded representations that capture relationships between words.
- **Decoder**: Here, the model generates output text word-by-word utilizing the encoded information provided by the encoder.

**Self-Attention Mechanism**: This ingenious mechanism in Transformer allows the model to focus on the parts of the input sequence most relevant to a given word or phrase. It pays varying degrees of attention to different parts of the input text based on their importance to the current prediction. This capability is crucial for LLMs to comprehend language and contextual nuances.

**Input Embeddings and Output Decoding**:
- **Input Embeddings**: Before text data is fed into an LLM, word embeddings convert it into numerical representations. This process transforms words into vectors, capturing their semantic similarities and relationships.
- **Output Decoding**: After the LLM processes the encoded input, decoding converts internal representations back into human-readable text.

**Model Size and Parameter Count**: The number of parameters (weights and biases) in an LLM significantly impacts its capabilities. Large-scale LLMs typically possess billions or even hundreds of billions of parameters, enabling them to learn complex patterns and relationships in language data. However, this also demands substantial computational resources to train and run the models.





![Mathematical Computation Flow of Scaled Dot-Product Self-Attention](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig11_self_attention_calculation.png)

</details>

**5. How does the Transformer architecture overcome challenges of Seq2Seq models?**

<details>
<summary>💡 Show Answer</summary>

The Transformer architecture completely revolutionized sequence-to-sequence tasks by replacing recurrence mechanisms with:
- **Self-Attention**: Processes all tokens in parallel, efficiently capturing global dependencies.
- **Positional Encoding**: Adds token ordering information.
- **Scalability**: Handles long sequences without the vanishing gradient problems inherent in RNNs.
Transformer is faster and better suited for tasks requiring context-aware predictions over long input sequences.

</details>

**6. What basic components constitute the Transformer model architecture?**

<details>
<summary>💡 Show Answer</summary>

Overall, the Transformer adopts an Encoder-Decoder architecture, primarily composed of the following parts:

- **Input Embedding Layer**: Maps discrete input tokens (words or subwords) into continuous vector representations.
- **Positional Encoding**: Since the model itself lacks inherent sequential order information, positional information must be added to word embeddings as vectors.
- **Encoder**: Stacked by $N$ identical layers, each containing a multi-head self-attention sublayer and a feed-forward neural network sublayer, followed by residual connections and Layer Normalization (Add & Norm) after each sublayer. The encoder converts the input sequence into a set of context-rich representations.
- **Decoder**: Likewise stacked by $N$ identical layers, but each layer contains three sublayers: masked multi-head self-attention (preventing look-ahead into future information), encoder-decoder attention (fusing output information from the encoder), and a feed-forward neural network, with Add & Norm following each sublayer.
- **Output Layer**: The decoder ends with a linear layer that maps hidden states to the vocabulary size dimension, followed by Softmax to obtain the probability distribution over the next token.

Overall, the encoder is responsible for "understanding" the input, and the decoder is responsible for "generating" the output, connected together via encoder-decoder attention.

</details>

**7. How does Attention Masking work in the Transformer model?**

<details>
<summary>💡 Show Answer</summary>

The core idea of attention masking is: before computing Softmax, set the attention scores of positions that should not be attended to a large negative number (such as -1e9 or $-\infty$). Thus, after Softmax, the weights corresponding to these positions approach 0, yielding virtually no effect during weighted summation, effectively "masking" these positions.

Two common types of masks in Transformer:

1. **Padding Mask**: Since sequence lengths within a batch vary, padding is required to pad them to equal length. The padding mask marks which positions are real tokens and which are padding placeholders, allowing the model to ignore padding during attention computation and avoid interference from meaningless information.
2. **Sequence Mask / Causal Mask (Look-ahead Mask)**: Used exclusively in decoder self-attention to ensure that token at position $i$ can only see information from positions $1$ through $i$, and cannot look ahead to position $i+1$ and beyond. This is typically implemented by constructing an upper triangular matrix (with values above the diagonal set to $-\infty$), guaranteeing consistent behavior during training and inference aligned with autoregressive causal properties.

In practical implementations, these two masks can be combined additively to form the final attention mask matrix.





![Causal Attention Masking Matrix for Autoregressive Token Generation](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig12_attention_masking_causal.png)

</details>

**8. Why does Transformer choose the Softmax function to compute attention weights?**

<details>
<summary>💡 Show Answer</summary>

When computing attention, Transformer needs to convert the dot product results between Query and each Key (a set of real-valued scores) into weights suitable for weighted summation. The Softmax function is exceptionally well-suited here for the following primary reasons:

1. **Normalization into Probability Distribution**: Softmax maps any set of real numbers to a distribution of non-negative values that sum to 1. This allows these weights to be directly interpreted as "attention intensity" or "importance proportion", and enables them to be directly used for convex combinations (weighted averages) of Value vectors.
2. **Smooth and Differentiable**: Softmax is a smooth, everywhere-differentiable function, facilitating gradient computation during backpropagation so that the entire attention mechanism can be trained end-to-end.
3. **Highlighting Relative Differences**: Because Softmax involves exponentiation, larger scores are exponentially amplified while smaller scores are relatively suppressed. This enables the model to focus attention more decisively on a few most relevant positions rather than spreading attention uniformly.
4. **Synergy with Scaled Dot-Product**: Transformer divides dot-product results by $\sqrt{d_k}$ (scaling factor) before Softmax to prevent overly large dot-product values in high-dimensional vector spaces, which could otherwise lead to vanishingly small Softmax gradients and destabilize training.

In summary, Softmax fulfills both the necessity of probabilistic interpretation and the feasibility and stability of training, making it the natural choice for computing attention weights.

</details>

**9. How does the Transformer model handle input sequences of different lengths?**

<details>
<summary>💡 Show Answer</summary>

In practical applications, sequence lengths within a batch are often different. Transformer primarily handles this issue through the following mechanisms:

1. **Padding**: Pads all sequences within a batch (typically using a special `<pad>` token) to the same length so they can form a regular tensor for parallel computation.
2. **Padding Mask**: In attention computation, padding masks set attention scores of padding positions to negative infinity ($-\infty$). Consequently, these positions receive zero weight after Softmax, preventing them from affecting real token representations or causing the model to "learn" semantics of padding tokens.
3. **Flexibility of Positional Encoding**: Positional encodings (whether sinusoidal or learnable) are generated based on position indices, making them theoretically applicable to sequences of arbitrary lengths.
4. **Inherent Architectural Adaptability to Length**: The self-attention mechanism places no fixed constraint on sequence length and does not rely on step-by-step recurrence like RNNs, allowing it structurally to handle inputs of arbitrary lengths.

However, it is important to note that the computational complexity of Transformer self-attention is $O(n^2)$ (where $n$ is sequence length). Therefore, the maximum practical processing length is bounded by GPU memory and computational resources. Additionally, models are typically trained with a maximum sequence length (such as 512, 1024, 2048, etc.), requiring truncation, sliding windows, or chunking strategies for sequences exceeding this limit. For scenarios requiring ultra-long sequence processing, improved architectures like Longformer, Reformer, and Transformer-XL have been proposed to reduce computational complexity and enhance long-range dependency modeling.

</details>

**10. How is sequence-to-sequence mapping implemented in Transformer?**

<details>
<summary>💡 Show Answer</summary>

The overall process of Transformer implementing sequence-to-sequence (Seq2Seq) mapping is as follows:

1. **Encoding Stage**: The input sequence (such as a source language sentence), after passing through embedding layers and positional encoding, is fed into the encoder. The encoder transforms the input sequence into a set of context representations of equal length to the input through multiple layers of self-attention and feed-forward networks (where each position vector incorporates information from the entire input sequence).
2. **Decoding Stage (Autoregressive Generation)**: During training, the decoder receives the target sequence shifted right by one position with a start token added ("teacher forcing"). During inference, it starts with a start token `<bos>` and generates output step-by-step:
- The decoder performs masked self-attention over the already generated target sequence prefix to obtain the representation for the current generation position;
- It incorporates output information from the encoder via encoder-decoder attention;
- After passing through feed-forward networks, output linear layer, and Softmax, it obtains a probability distribution over the entire vocabulary for the current position;
- It selects the token with the highest probability (or via sampling, beam search, etc.) as the output for the current position and appends it to the end of the generated sequence as input for the next decoding step.

1. **Termination Condition**: When the end token `<eos>` is generated or the maximum preset length is reached, the generation process terminates, yielding the final output sequence.

Through this approach of "encoder single-pass encoding, decoder step-by-step autoregressive generation", Transformer achieves mapping from arbitrary-length input sequences to arbitrary-length output sequences, forming the foundation for its widespread application in machine translation, text summarization, dialogue generation, and other tasks.

</details>

**11. What specifically are parameter sharing strategies in Transformer models?**

<details>
<summary>💡 Show Answer</summary>

Common parameter sharing strategies in Transformer and its variants primarily include:

1. **Input Embedding and Output Projection Sharing (Tied Embeddings / Weight Tying)**: The linear transformation matrix of the decoder's final output layer (mapping hidden states to vocabulary dimension) shares parameters with (or is the transpose of) the word embedding matrix. This reduces parameter counts and, by constraining input embedding and output spaces within the same semantic space, often enhances model performance, commonly seen in models like GPT and T5.
2. **Embedding Sharing Between Encoder and Decoder**: In certain machine translation or multilingual models, the encoder and decoder share the same set of word embedding parameters (especially when source and target languages share a vocabulary).
3. **Cross-Layer Parameter Sharing (Layer-wise Sharing)**: While standard Transformer layers share identical structures, their parameters are independent (each layer has its own weights). Models like **ALBERT** and **Universal Transformer** share the same set of parameters across layers (e.g., all Transformer blocks share a single set of weights executed recurrently), drastically reducing parameter count while imparting a degree of "recurrent" computation.
4. **Sharing Within Multi-Head Attention**: In standard implementations, each head of multi-head attention has independent linear projection matrices. In efficiency-optimized variants (like Multi-Query Attention and Grouped-Query Attention), multiple Query heads share the same set of Key and Value projection parameters to reduce GPU memory footprint and computational cost during inference.

Overall, the core objective of parameter sharing is to reduce parameter count, lower overfitting risks, and improve training and inference efficiency while minimizing loss of model expressiveness.

</details>

**12. How to understand the autoregressive property in Transformer?**

<details>
<summary>💡 Show Answer</summary>

Autoregressive refers to the property that when generating a sequence, the output at the current time step depends on the outputs generated at all previous time steps, i.e.:

\begin{verbatim}
P(y_1, y_2, ..., y_T) = P(y_1) * P(y_2|y_1) * P(y_3|y_1,y_2) * ... * P(y_T|y_1,...,y_{T-1})
\end{verbatim}

In the Transformer decoder, this property is realized via **Masked Self-Attention**: when calculating attention for the $t$-th position, a Causal Mask forces attention scores for positions $t+1$ and beyond to $-\infty$. Consequently, position $t$ can only "see" information from positions $1$ to $t$ when computing its representation, preventing access to future information.

Implications of the autoregressive property include:

1. **Consistency Between Training and Inference**: Training typically employs "teacher forcing", feeding the full ground-truth target sequence (shifted right by one position) into the decoder to compute losses across all positions in parallel. Inference, however, must strictly proceed step-by-step autoregressively, predicting the next token only after generating the current one.
2. **Inference Efficiency Issues**: Because inference requires token-by-token generation without the parallel computation available during training, autoregressive models are generally slow during inference. This serves as a primary motivation for non-autoregressive (Non-Autoregressive) generative models, parallel decoding, KV Cache caching, and other optimization techniques.
3. **Ensuring Causality and Coherence in Generation**: The autoregressive property ensures that every token generated by the model builds upon previously generated context, serving as a critical mechanism enabling language models to produce coherent, grammatically and semantically sound text.

</details>

**13. How does the decoder portion of the Transformer model generate sequences?**

<details>
<summary>💡 Show Answer</summary>

The process of sequence generation by the Transformer decoder is an **autoregressive** iterative process, with the following detailed steps:

1. **Initialization**: At the start of inference, the decoder input contains only a special start token (such as `<bos>`).
2. **Masked Self-Attention**: The decoder computes self-attention over the sequence generated so far (including the start token) and enforces via Causal Mask that each position attends only to itself and preceding positions, unable to "see" ungenerated future tokens.
3. **Encoder-Decoder Attention (Cross-Attention)**: Using representations from the previous step as Queries, it performs attention computation with Keys and Values from the encoder output to integrate relevant information from the input sequence (such as a source language sentence).
4. **Feed-Forward Network and Normalization**: After passing through feed-forward neural networks (FFN), residual connections, and Layer Normalization, the current layer output is obtained. Multi-layer decoders repeat steps 1–4 sequentially.
5. **Output Projection and Probability Computation**: The output of the final decoder layer passes through a linear layer mapping to vocabulary dimension, followed by a Softmax function to produce the probability distribution over the next token across the entire vocabulary.
6. **Sampling/Selecting the Next Token**: Based on this probability distribution, a decoding strategy selects the next token. Common strategies include:
- **Greedy Search**: Directly picks the highest-probability token at each step;
- **Beam Search**: Maintains multiple candidate sequences simultaneously, preserving a fixed number of paths with highest cumulative probabilities;
- **Sampling Methods**: Such as Top-$k$ sampling, Top-$p$ (nucleus) sampling, temperature sampling, etc., introducing randomness to enhance generation diversity.

1. **Iterative Generation**: Appends the newly generated token to the end of the generated sequence to serve as the decoder input for the next step, repeating steps 2–6 until an end token `<eos>` is produced or the maximum generation length is reached.

In practical deployment, to avoid recomputing Key and Value vectors for all past positions at every step, **KV Cache** is typically employed to cache historical Key and Value vectors, significantly enhancing autoregressive inference efficiency.

</details>

**14. In which NLP tasks has the Transformer model demonstrated significant performance improvements?**

<details>
<summary>💡 Show Answer</summary>

Since its introduction in 2017, the Transformer has driven remarkable performance gains across virtually all mainstream NLP tasks, primarily demonstrated in:

1. **Machine Translation**: The Transformer was originally designed for machine translation, significantly outperforming RNN/LSTM-based Seq2Seq models on translation benchmarks like WMT while dramatically speeding up training through parallel computation.
2. **Language Modeling and Text Generation**: Autoregressive Transformer models represented by the GPT series achieved breakthroughs in language modeling perplexity, coherence, and creativity in long-text generation, capable of producing high-quality articles, dialogues, code, and more.
3. **Text Understanding Tasks**: Bidirectional encoder Transformers represented by BERT, through the pre-training + fine-tuning paradigm, set new benchmark records across text classification, sentiment analysis, Natural Language Inference (NLI), Named Entity Recognition (NER), Question Answering systems (such as SQuAD), and more.
4. **Text Summarization**: For both extractive and abstractive summarization, Transformer and its variants (such as BART, PEGASUS) better capture key information in long documents and generate fluent summaries.
5. **Long-Range Dependency Modeling**: Compared to RNNs/LSTMs constrained by vanishing/exploding gradient issues from sequential recurrence, Transformer self-attention directly establishes connections between any two positions, performing markedly better in handling dependencies across long texts, sentences, and paragraphs.
6. **Multi-Task and Transfer Learning**: Through "pre-training + fine-tuning" or "pre-training + prompting" paradigms, a single pre-trained Transformer model can be transferred to almost any downstream NLP task, drastically reducing the cost of designing task-specific model architectures.

</details>

**15. How is the Transformer Model Applied to Non-NLP Tasks?**

<details>
<summary>💡 Show Answer</summary>

Transformer was originally designed for Natural Language Processing (NLP), but its core idea—representing inputs as a sequence of tokens and modeling the relationships between elements within the sequence via self-attention—has high versatility and has thus been widely adapted to other fields:

1. **Computer Vision (Vision Transformer, ViT)**: An image is split into fixed-size patches. Each patch, after a linear projection, is treated as a "token." Added with positional embeddings, it is fed into a standard Transformer encoder for tasks such as image classification, object detection (e.g., DETR), and image segmentation. Subsequent models like Swin Transformer introduced hierarchical structures and local window attention to better adapt to the characteristics of visual tasks.
2. **Speech Processing**: In Automatic Speech Recognition (ASR) and Text-to-Speech (TTS), audio signals are processed into frame sequences or spectrogram feature sequences, which are then fed into Transformer encoder-decoder architectures, such as Conformer (combining convolution and Transformer) and Speech-Transformer.
3. **Recommender Systems**: User historical behavior sequences (clicks, purchases, browsing history) are treated as token sequences. Self-attention mechanisms are used to model the dynamic evolution of user interests and capture long-range dependencies between behaviors for tasks like click-through rate (CTR) prediction and sequential recommendation.
4. **Bioinformatics**: In protein structure prediction, models like AlphaFold2 utilize Transformer-like attention mechanisms (Evoformer) to model relationships between amino acid sequences as well as between sequence and structural information, achieving breakthrough progress.
5. **Time Series Forecasting**: Historical observations of time series are used as input sequences, leveraging self-attention to model dependencies across different time steps for scenarios such as financial forecasting, sales prediction, and weather forecasting (e.g., enhanced models like Informer and Autoformer).
6. **Reinforcement Learning**: Works such as Decision Transformer model "state-action-return" trajectories in reinforcement learning as sequential data, reformulating policy learning into a sequence modeling problem to directly predict action sequences using Transformers.
7. **Multimodal Learning**: By uniformly encoding data from different modalities—such as images, text, and audio—into token sequences, the Transformer acts as a unified backbone network to process multimodal inputs, as seen in multimodal foundation models like CLIP, Flamingo, and GPT-4V.

In short, the Transformer has evolved from an NLP-specific architecture into a general-purpose "sequence modeling" foundational infrastructure.

</details>

**16. Which Transformer Variants or Improved Models Have Been Widely Applied in Industry?**

<details>
<summary>💡 Show Answer</summary>

Over years of development, numerous variants and improved models derived from the original Transformer have emerged, many of which have been deployed at scale in industry. They primarily include:

1. **BERT and Its Family**: BERT uses only the Transformer encoder portion, pre-trained via Masked Language Modeling (MLM) and Next Sentence Prediction (NSP). It is widely applied in scenarios such as semantic understanding in search engines, text classification, and information extraction. Its subsequent variants, such as RoBERTa, ALBERT, and ERNIE, are also extensively adopted in industry.
2. **GPT Series**: Uses only the Transformer decoder portion, pre-trained via autoregressive language modeling. It is the mainstream architecture for current Large Language Models (LLMs) and is applied in almost all generative AI products, including conversational systems, content generation, and code generation.
3. **T5 / BART**: Employs the full encoder-decoder structure, uniformly modeling various NLP tasks into a "text-to-text" format. They are widely used for generative tasks such as summarization, translation, and question answering.
4. **Vision Transformer (ViT) and Its Variants**: Includes Swin Transformer, DeiT, etc., which have become one of the mainstream backbone networks for computer vision tasks such as image classification, detection, and segmentation, integrated into many visual AI products.
5. **Efficient Attention Variants**: Models like Longformer, BigBird, and Reformer reduce computational complexity for long-sequence processing through sparse attention or combinations of local + global attention, applied in scenarios requiring long-document processing (e.g., legal and medical text analysis).
6. **Large-Scale Pre-trained Language Models**: Open-source or commercial LLMs such as LLaMA, PaLM, Qwen, and ChatGLM generally adopt optimized Decoder-only Transformer architectures (e.g., incorporating improvements like RoPE positional encoding, RMSNorm, and Grouped-Query Attention). They are widely deployed across numerous products, including search, customer service, office automation, and coding assistance.
7. **Mixture-of-Experts (MoE) Combined with Transformer**: Models like Switch Transformer and Mixtral introduce MoE structures into the feed-forward layers of the Transformer. This significantly expands model capacity without substantially increasing inference cost, serving as an important technical path for current ultra-large-scale models.

</details>

**17. What Are the Future Development Trends of Transformer Models?**

<details>
<summary>💡 Show Answer</summary>

Combining current research and industrial development directions, Transformer models are likely to exhibit the following trends in the future:

1. **More Efficient Attention Mechanisms**: Standard self-attention has a computational and memory complexity of $O(n^2)$, limiting the model's capacity to process ultra-long contexts. Future efforts will continue to explore methods like sparse attention, linear attention (e.g., Linear Attention, Performer), and low-rank approximation to reduce complexity while maintaining performance as much as possible, supporting longer context windows (expanding from a few thousand tokens to hundreds of thousands or even millions of tokens).
2. **Architectural Hybridization and Alternatives**: State Space Models (SSMs), represented by models like Mamba, demonstrate performance comparable to or even better than Transformers in long-sequence modeling with linear complexity. In the future, more hybrid models combining Transformers with SSMs, convolutions, RNNs, and other architectures may emerge to balance effectiveness and efficiency.
3. **Mixture-of-Experts (MoE) and Sparsification**: Through sparse activation mechanisms such as MoE, only a subset of parameters is activated during inference. This allows for significantly scaling up total parameter count and expanding model capacity while keeping actual compute costs under control, serving as a critical pathway toward larger-scale models.
4. **Unified Multimodal Modeling**: As a general sequence modeling architecture, the Transformer will further become a unified backbone network for data across multiple modalities, including text, image, audio, video, and 3D, driving the development of multimodal foundation models (such as GPT-4V, Gemini, etc.).
5. **Inference Efficiency Optimization**: Techniques including model quantization (INT8/INT4, etc.), knowledge distillation, model pruning, KV Cache optimization, and Speculative Decoding will continue to be researched and applied. This aims to reduce the deployment and inference costs of LLMs, enabling them to run on edge devices or resource-constrained environments.
6. **Integration with External Tools and Systems**: Retrieval-Augmented Generation (RAG), tool calling, and Agent frameworks will further integrate with Transformer models. This allows models to rely not only on knowledge stored within their parameters, but also to dynamically acquire external information and invoke external tools to accomplish more complex tasks.
7. **Evolution of Training Paradigms**: Further developments in more efficient pre-training objective designs, alignment with human preferences via reinforcement learning (e.g., RLHF, DPO), as well as domain-specific pre-training and fine-tuning strategies (e.g., for code, mathematics, and scientific computing), will continuously improve model performance on complex reasoning and specialized tasks.

</details>

**18. What Does "Negative Attention" Mean in Transformer Models?**

<details>
<summary>💡 Show Answer</summary>

It should be noted that "Negative Attention" is not a standard term in the original Transformer paper or mainstream literature. It is more of an informal term used in interviews or discussions to describe certain phenomena in attention mechanisms. Common perspectives of understanding include:

1. **Low-Weight / Near-Zero Attention**: In the attention distribution output by Softmax, weights at most positions may be very close to 0. This means the model deems these positions to have very low relevance to the current Query, contributing almost no information, which can be informally understood as a form of "ignored" or "negative" attention.
2. **Masked-Out Attention**: In causal masking or padding masking, attention scores set to $-\infty$ yield weights of 0 after Softmax. This part can also be colloquially understood as "negative (disallowed) attention," i.e., positions structurally prohibited from being attended to.
3. **Inhibitory / Negative Correlation**: From a broader perspective, if the attention score itself is viewed as a "relevance score," then a large negative score mathematically represents opposite vector directions between Query and Key (negative dot product), suggesting a semantic "negative correlation" or "opposing" relationship. However, after Softmax, this negative correlation only manifests as an extremely low weight rather than a "negative attention weight" (since Softmax outputs are always non-negative).

Overall, Transformer attention weights are mathematically non-negative (due to the Softmax function), and true "negative weights" do not exist. "Negative attention" is largely a descriptive metaphor for low-weight or masked-out portions.

</details>

**19. How to Understand the "Encoder-Decoder Attention" Mechanism in Transformer Models?**

<details>
<summary>💡 Show Answer</summary>

Encoder-Decoder Attention (also known as Cross-Attention) is the second attention sub-layer in the decoder. Its core function is to enable the decoder to dynamically "consult" relevant information from the input sequence when generating each output token.

Specifically:
- **Query (Q)** comes from the output of the preceding sub-layer (masked self-attention) in the current decoder layer, representing "what needs to be generated currently."
- **Key (K) and Value (V)** both come from the output of the last encoder layer, representing "what information is contained in the input sequence."

By computing the similarity between Query and all Keys (via dot product followed by Softmax normalization), a set of attention weights is obtained. These weights are then used to take a weighted sum of the Values, yielding a context vector that incorporates relevant information from the input sequence.

This mechanism essentially achieves a "soft alignment" between the source and target sequences, similar to attention mechanisms in traditional Seq2Seq models. For example, in machine translation, when generating a word in the target language, the model automatically attends to the corresponding word or phrase in the source language sentence.

</details>

**20. What is a Sequence-to-Sequence Model?**

<details>
<summary>💡 Show Answer</summary>

Sequence-to-sequence (Seq2Seq) models convert an input sequence into a corresponding output sequence, commonly used for:
- Machine Translation: English $\rightarrow$ French.
- Text Summarization: Long text $\rightarrow$ Summary.
- Question Answering: Context $\rightarrow$ Answer.

**Components**:
- Encoder: Processes the input and generates representations.
- Decoder: Transforms representations into the desired output.

**Examples**: Transformer-based models such as T5 and RNN-based models such as Seq2Seq with attention mechanisms.

</details>

**21. What is Masked Language Modeling (MLM)?**

<details>
<summary>💡 Show Answer</summary>

MLM involves randomly masking some tokens in the input text and training the model to predict them based on the surrounding context. It was popularized by BERT (Bidirectional Encoder Representations from Transformers).

**Objective**: Learn bidirectional dependencies in language by leveraging bidirectional context.

**Example**:
- Input: “The [MASK] is blue.”
- Prediction: “The sky is blue.”

</details>

**22. What is Next Sentence Prediction (NSP)?**

<details>
<summary>💡 Show Answer</summary>

NSP is a pre-training objective used to teach the model to understand relationships between sentences. The model predicts whether a given Sentence B naturally follows Sentence A.

**Example (BERT)**:
- Input:
    - Sentence A: “I love books.”
    - Sentence B: “They expand my knowledge.”
- Model Output: True.

</details>

**23. How Do Autoregressive Models Differ from Masked Models?**

<details>
<summary>💡 Show Answer</summary>

- **Autoregressive Models (e.g., GPT)**: Generate text token by token, using previously generated tokens as context.
    - **Example**: Predict the next word in the sequence.
    - **Advantage**: Suitable for text generation.
- **Masked Models (e.g., BERT)**: Predict masked tokens in the sequence using bidirectional context.
    - **Advantage**: Suitable for understanding tasks such as classification or question answering.

</details>

**24. Please Explain in Detail How the Self-Attention Mechanism Works in the Transformer Model. Why is it Better Suited for Processing Long Sequences Than RNNs?**

<details>
<summary>💡 Show Answer</summary>

The Self-Attention mechanism is the core of the Transformer model. It enables the model to dynamically weigh the importance of different words in an input sequence and generate context-aware representations for each word accordingly.

The working principle is as follows:

1. Generate Q, K, V Vectors: For the embedding vector of each token in the input sequence, we multiply it by three learnable weight matrices $W^Q, W^K, W^V$ to generate three vectors: Query (Q), Key (K), and Value (V).

- Query (Q): Represents the information that the current token needs to "query" from other tokens in the sequence to better understand itself.
- Key (K): Represents the information label "carried" by each token in the sequence that can be queried.
- Value (V): Represents the actual deep semantic content contained in each token in the sequence.

1. Compute Attention Scores: To determine how much focus the current token (represented by Q) should place on all other tokens (represented by K), we compute the dot product of the current token's Q with the K vectors of all other tokens. This score measures the relevance between the two.

\[Score(Q_i, K_j) = Q_i \cdot K_j\]

1. Scaling: Divide the calculated scores by a scaling factor $\sqrt{d_k}$ (where $d_k$ is the dimension of the Key vector). This step ensures more stable gradients during backpropagation, preventing large dot products from causing the Softmax function to enter regions with extremely small gradients (saturation).

\[\frac{Q \cdot K^T}{\sqrt{d_k}}\]

1. Softmax Normalization: Pass the scaled scores through a Softmax function to convert them into a probability distribution summing to 1. These probabilities are the "attention weights," indicating the importance of each input token at the current position.

\[AttentionWeights = softmax\left(\frac{Q K^T}{\sqrt{d_k}}\right)\]

1. Weighted Sum: Finally, multiply the resulting attention weights by the corresponding V vector of each token and sum them up to obtain the output of the self-attention layer. This output vector integrates contextual information across the entire sequence, with weights learned dynamically by the model.

\[Output = AttentionWeights \cdot V\]

Why is it better suited for processing long sequences than RNNs?

1. Parallel Computation Capability: Self-attention processes the entire sequence simultaneously in computation, calculating associations between all positions at once, making it highly parallelizable. In contrast, RNNs (including LSTMs and GRUs) must process each token sequentially over time steps and cannot be parallelized, leading to very slow processing for long sequences.
2. Resolving Long-Distance Dependency Issues: In self-attention, the path length for interaction between any two positions is O(1) because their attention score can be directly calculated. In RNNs, information transmission between the head and tail tokens of a sequence must traverse the entire length of the sequence with a path length of O(N), which easily causes vanishing or exploding gradients, making it difficult for the model to capture long-range dependencies.

</details>

**25. Do You Know the Differences Among MHA, MQA, and GQA? Please Explain in Detail.**

<details>
<summary>💡 Show Answer</summary>

![Structural Comparison among Multi-Head Attention (MHA), Multi-Query Attention (MQA), and Grouped-Query Attention (GQA)](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig13_mha_mqa_gqa_comparison.png)



MHA, MQA, and GQA are three different attention mechanism variants in Transformer models. Their main difference lies in how they organize and share the "heads" of Query, Key, and Value. The core goal is to make different trade-offs between model effectiveness and inference efficiency (especially VRAM consumption).

**1. MHA (Multi-Head Attention)**

This is the standard attention mechanism proposed in the original Transformer paper.

- Working Principle:

1. Project input Q, K, V vectors through $N$ independent linear transformations to obtain $N$ sets of distinct $Q_i, K_i, V_i$ heads ($i=1, ..., N$).
2. These $N$ sets of heads compute attention (Scaled Dot-Product Attention) in parallel in their respective subspaces.
3. Concatenate the output vectors calculated from all $N$ heads.
4. Finally, map the concatenated vector back to the original dimension using a linear transformation.

- Structure: $N$ Query heads, $N$ Key heads, $N$ Value heads.
- Advantages: Best performance and strongest model capability. Each head can learn different information in different representation subspaces.
- Disadvantages: High inference cost. In autoregressive generation tasks, Key and Value for each layer need to be cached (i.e., KV Cache). The KV Cache size in MHA is proportional to the number of heads $N$, causing very high VRAM consumption and restricting long-sequence generation.

**2. MQA (Multi-Query Attention)**

Proposed to resolve the VRAM bottleneck of MHA during inference.

- Working Principle:

1. Like MHA, there are $N$ independent Query heads.
2. Key Difference: All $N$ Query heads share a single Key head and a single Value head.

- Structure: $N$ Query heads, 1 Key head, 1 Value head.
- Advantages: Drastically reduces inference cost. The size of the KV Cache no longer depends on the number of heads $N$, reducing by a factor of $N$ compared to MHA, significantly lowering VRAM consumption and speeding up inference.
- Disadvantages: May lead to degraded model performance. Because all Query heads are forced to extract information from the exact same set of Key and Value, the expressiveness of the model is somewhat constrained.

**3. GQA (Grouped-Query Attention)**

GQA is a trade-off solution between MHA and MQA, aimed at balancing performance and efficiency.

- Working Principle:

1. Divide the $N$ Query heads into $G$ groups.
2. Key Difference: Query heads within each group share one Key head and one Value head. In total, there are $G$ Key heads and $G$ Value heads.

- Structure: $N$ Query heads, $G$ Key heads, $G$ Value heads (usually $1 < G < N$).
- Notes:
- When $G=N$, GQA is equivalent to MHA.
- When $G=1$, GQA is equivalent to MQA.
- Advantages: Far superior to MHA in inference efficiency, while outperforming MQA in model performance. It provides a flexible knob to adjust between efficiency and performance according to specific needs. Models like Llama 2 adopt GQA.

Summary: | Feature | MHA (Multi-Head Attention) | MQA (Multi-Query Attention) | GQA (Grouped-Query Attention) | | :----------- | :------------------------- | :-------------------------- | :---------------------------- | | Structure | N Q heads, N K heads, N V heads | N Q heads, 1 K head, 1 V head | N Q heads, G K heads, G V heads | | Model Quality | Highest | May decrease | Close to MHA, superior to MQA | | Inference Efficiency | Lowest (Large KV Cache) | Highest (Small KV Cache) | Moderate, far better than MHA | | Applications | BERT, GPT-3 | PaLM | Llama 2, Mixtral |

</details>

**26. Subsequent Computational Optimization Work for MHA**

<details>
<summary>💡 Show Answer</summary>

To address the issue of excessive VRAM consumption by KV Cache in standard MHA during inference:

1. **MQA (Multi-Query Attention)**: All Query heads share the same set of Key and Value heads, drastically reducing KV Cache, though it may slightly impact model quality.
2. **GQA (Grouped-Query Attention)**: A compromise between MHA and MQA—Query heads are divided into several groups, each sharing one set of K and V heads. Widely adopted by models like LLaMA 2/3 and Mistral.
3. **MLA (Multi-head Latent Attention)**: Proposed by DeepSeek, it performs joint low-rank compression on K and V, significantly reducing KV Cache VRAM while maintaining model expressiveness.
4. **Sliding Window Attention**: Each token only attends to historical tokens within a fixed window size, reducing complexity from O($n^2$) to O($n \cdot w$).
5. **FlashAttention**: Without modifying the mathematical definition of attention, it significantly reduces VRAM usage and accelerates computation through operator-level optimizations including tiling, online softmax, and reduced HBM reads/writes.

</details>

**27. Common Computation Methods for Attention**

<details>
<summary>💡 Show Answer</summary>

1. **Standard Self-Attention**: Attention(Q,K,V) = softmax(QK$^T$/$\sqrt{d_k}$) V, with complexity O($n^2d$).
2. **Multi-Head Attention (MHA)**: Projects Q, K, V into multiple low-dimensional subspaces to compute attention separately, then concatenates the outputs.
3. **MQA / GQA**: Reduces VRAM and computational overhead during inference by sharing K and V heads.
4. **Causal (Masked) Self-Attention**: Adds a $-\infty$ mask to the attention scores of future positions.
5. **Cross-Attention**: Q comes from one sequence, while K and V come from another sequence.
6. **Sparse Attention**: Computes attention only for select pairs of positions to reduce complexity.
7. **Linear Attention**: Reduces complexity to O($n$) by using kernel function approximations or altering computation order.
8. **FlashAttention**: Computes standard attention exactly, but optimizes IO through tiling and operator fusion.
9. **MLA**: An attention variant that optimizes memory footprint through low-rank compression of KV.

</details>

**28. Derivation of Why Division by $\sqrt{d$ is Needed in Softmax}**

<details>
<summary>💡 Show Answer</summary>

**Definition and Assumptions**: $Q$ and $K$ are two independent random matrices, both with dimension $n \times d_k$. Each element is independent and identically distributed (i.i.d.) with a mean of 0 and a variance of 1.

Calculating $(QK^T)_{ij} = \sum_{l=1}^{d_k} q_{il} k_{jl}$:
- Mean: $E[(QK^T)_{ij}] = 0$
- Variance: $D[(QK^T)_{ij}] = d_k$

After dividing by $\sqrt{d_k}$:

$$
    D\left[\frac{(QK^T)_{ij}}{\sqrt{d_k}}\right] = \frac{d_k}{d_k} = 1
$$


**Intuitive Explanation**: Without scaling, the variance of $QK^T$ grows linearly with $d_k$, leading to excessively large inputs to Softmax. This pushes the output close to a one-hot distribution, causing vanishing gradients. After scaling, the variance is stabilized at 1, yielding a smoother Softmax input distribution and more stable gradient propagation.

</details>

**29. Is Dropout Used in Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

Observing the LLaMA3 source code reveals that dropout is absent, while observing Qwen shows that dropout parameters exist but are set to 0.

This phenomenon is not accidental, but rather the result of continuous evolution in LLM pre-training practices. This section provides a detailed analysis of this issue from the perspectives of theoretical motivation, practical experience, and source code.

#### Basic Principles and Original Purpose of Dropout

Dropout was proposed by Srivastava et al. in 2014. Its core idea is to randomly set neuron outputs to zero with probability $p$ during training, thereby:

- Preventing excessive co-adaptation between neurons, serving as a regularizer;
- Being interpreted as ensemble learning over an exponential number of sub-networks;
- Significantly alleviating overfitting issues in small-to-medium scale models and scenarios with limited training data.

In early Transformer models (such as the implementation in the original *Attention is All You Need*), Dropout was widely applied to attention weights, residual connection outputs, feed-forward layers, etc., with a typical value of $p = 0.1$.

#### Reasons for Gradually Phasing Out Dropout in LLM Training

With the dramatic scale expansion of model size and data volume, the necessity of dropout has declined significantly. The main reasons are as follows:

1. **Massive Data Itself Provides Sufficient Regularization**\\
    LLMs are typically trained on hundreds of billions or even trillions of tokens. The diversity and scale of data make it difficult for the model to overfit on the training set. Data diversity itself plays the role of regularization, rendering the additional regularization effect of dropout minimal.

2. **Dropout Impairs Training Stability and Convergence Speed**\\
    In large-scale training, the random noise introduced by dropout interferes with gradient estimation, leading to:
    - Increased loss curve oscillations and slower convergence;
    - Complex interaction effects with learning rate scheduling, gradient clipping, and other techniques;
    - Exacerbated numerical instability under mixed-precision training (FP16/BF16) due to random zeroing out.

3. **Discrepancy Between Training and Inference**\\
    Dropout is active during the training phase and turned off during inference. This discrepancy introduces additional engineering complexity in LLM scenarios, particularly in distributed training and model parallelism, where synchronizing random number states across devices is challenging.

4. **Other Regularization Techniques Are Sufficient**\\
    Modern LLMs typically use a combination of the following techniques to replace the regularization effect of dropout:
    - **Weight Decay**: Constrains parameter magnitudes via L2 regularization, serving as the most mainstream regularization technique;
    - **Gradient Clipping**: Prevents gradient explosion and improves training stability;
    - **Data Diversity**: Large-scale, high-quality, multi-domain training corpora inherently act as strong regularization;
    - **Learning Rate Scheduling**: Strategies like Warmup + Cosine Decay facilitate smooth convergence.

#### LLaMA3 Source Code Analysis: Completely Removing Dropout

In the official implementation of LLaMA3, dropout is not used anywhere in the model structure. Taking its core modules as an example:

\begin{lstlisting}[style=promptstyle]
	# LLaMA3 Attention module (simplified)
	class Attention(nn.Module):
	def forward(self, x, ...):
	...
	# Attention weights directly softmaxed, no dropout
	scores = torch.softmax(scores, dim=-1)
	output = torch.matmul(scores, values)
	return output

	# LLaMA3 FeedForward module (simplified)
	class FeedForward(nn.Module):
	def forward(self, x):
	# SwiGLU activation, no dropout
	return self.w2(F.silu(self.w1(x)) * self.w3(x))
\end{lstlisting}

Meta's official technical report (LLaMA 2 Tech Report) also explicitly points out that dropout is not used during the pre-training phase, relying solely on weight decay and the AdamW optimizer for regularization.

#### Qwen Source Code Analysis: Retaining Interfaces but Setting to 0

The implementation of the Qwen series models retains parameter interfaces related to dropout, but sets them to 0 in default configurations, which is practically equivalent to not using dropout:

\begin{lstlisting}[style=promptstyle]
	# Qwen configuration file example (config.json)
	{
		"attention_dropout": 0.0,
		"hidden_dropout": 0.0,
		"emb_dropout_prob": 0.0,
		...
	}

	# Qwen Attention module (simplified)
	class QwenAttention(nn.Module):
	def __init__(self, config):
	self.attn_dropout = nn.Dropout(config.attention_dropout)  # Actually Dropout(0.0)

	def forward(self, ...):
	attn_weights = self.attn_dropout(attn_weights)  # p=0, equivalent to identity mapping
	...
\end{lstlisting}

The benefits of retaining the dropout interface include:

- **Flexibility**: During downstream fine-tuning, dropout can be enabled as needed to prevent overfitting without modifying the model code;
- **Experimental Comparison**: Researchers can easily enable dropout via configuration files for ablation studies;
- **Inheritance**: Evolved from earlier Transformer codebases, retaining interfaces helps maintain backward compatibility.

#### Is Dropout Needed During Fine-tuning?

Unlike pre-training, **appropriately introducing dropout during fine-tuning is often beneficial**, especially in the following scenarios:

- **Supervised Fine-Tuning (SFT) with Limited Data**: When training samples are scarce, the model is prone to overfitting; dropout with $p = 0.05 \sim 0.1$ can provide a degree of regularization protection;
- **Parameter-Efficient Fine-Tuning like LoRA**: LoRA adapters have few parameters, and introducing a small amount of dropout helps prevent adapter overfitting;
- **Instruction Tuning**: When the instruction data distribution is narrow, dropout helps preserve the model's generalization capabilities.

#### Summary

- **Pre-training Phase**: Modern LLMs generally do not use dropout (e.g., LLaMA3) or set it to 0 (e.g., Qwen). The fundamental reason is that massive training data already provides sufficient regularization, whereas dropout degrades training efficiency and stability.
- **Significance of Interface Retention**: Models like Qwen retain dropout interfaces primarily for flexibility and experimental convenience during fine-tuning, rather than for actual use in pre-training.
- **Fine-tuning Phase**: Depending on data scale and task characteristics, a small dropout rate ($p \leq 0.1$) can be appropriately enabled to prevent overfitting.

</details>

**30. Main Reasons for Not Using Dropout in Large Language Models**

<details>
<summary>💡 Show Answer</summary>

In the previous section, we observed that LLaMA3 completely removed dropout while Qwen set it to 0. This section summarizes the core reasons for discarding dropout in LLMs from a deeper perspective.

1. **Introducing Noise Leads to Training Instability**

    Dropout randomly zeroes out neurons during forward propagation with probability $p$, which is equivalent to introducing random multiplicative noise into the gradient flow. For LLMs with billions to hundreds of billions of parameters, their network depth is extremely deep (e.g., LLaMA3-70B has 80 Transformer layers), and random noise accumulates and amplifies layer by layer during backpropagation, specifically manifesting as:
    - Pronounced loss curve fluctuations, making steady decay difficult;
    - Increased gradient variance, making learning rate selection more sensitive;
    - Negative interference with stability mechanisms like gradient clipping and learning rate Warmup, reducing overall training robustness.
    In contrast, removing dropout results in cleaner gradient signals, which, combined with AdamW and gradient clipping, achieves more stable convergence.

2. **Increased Computational Overhead and Decreased Training Efficiency**

    Although dropout appears to be a simple "zeroing" operation, it introduces non-negligible extra overhead at the engineering implementation level:
    - **Random Number Generation**: Each training step requires generating random masks matching the shape of activation values for every layer; under large batch sizes and sequence lengths, random number generation itself consumes significant compute time;
    - **Memory Consumption**: Random masks must be saved during forward propagation for use in backpropagation, increasing GPU memory pressure;
    - **Distributed Synchronization Complexity**: In Tensor Parallelism and Pipeline Parallelism scenarios, random number states across devices must be synchronized; otherwise, additional uncertainty is introduced, increasing engineering complexity.
    In pre-training scenarios spanning months across tens of thousands of GPUs, the cumulative effect of the above overhead cannot be ignored.

3. **Removing Dropout Does Not Degrade Performance**

    Modern LLMs have integrated multiple structural and data-level regularization techniques, rendering dropout's regularization contribution optional:
    - **Pre-Norm (RMSNorm)**: Placing layer normalization before residual connections significantly improves deep network training stability and inherently provides a certain degree of regularization;
    - **Residual Connections**: Guarantee gradient highway paths, mitigating degradation in deep networks;
    - **Multi-Head Attention (MHA) and Grouped-Query Attention (GQA/MQA)**: Parameter sharing and information integration in multi-head structures exert implicit regularization effects;
    - **Weight Decay**: Constrains parameter magnitudes via L2 penalty, acting as the most mainstream explicit regularization technique;
    - **Large-scale Multi-domain Data**: Training on trillions of tokens of multi-source heterogeneous data makes data diversity the strongest source of regularization, leaving the model almost immune to overfitting on training data.
    The combined effect of these factors means that model generalization ability often improves rather than declines after removing dropout (due to eliminating random noise interference on effective feature learning).

4. **Extensive Experiments Prove Dropout Is Ineffective or Even Harmful in LLMs**

    Numerous ablation studies in academia and industry demonstrate that the gain from dropout negatively correlates with model scale:
    - In small models (e.g., LSTMs or small Transformers with millions of parameters), dropout significantly boosts validation accuracy and prevents overfitting;
    - As model scale increases, the gains brought by dropout gradually diminish; when parameters exceed hundreds of millions, dropout yields almost no accuracy gains;
    - In pre-training ultra-large models (e.g., GPT-3, LLaMA series), using dropout slows down convergence and reduces final performance;
    - Technical reports for models such as PaLM and Chinchilla explicitly note that dropout is not used during pre-training, a conclusion that has reached consensus across the industry.
    The fundamental reason is that dropout was originally designed for "high parameter, low data" overfitting scenarios, whereas the training scenario for LLMs is the exact opposite—data is extremely abundant, and the model is actually in an underfitting regime, where introducing noise is counterproductive.

</details>

**31. Why Can the Three Embeddings in BERT Be Added Together?**

<details>
<summary>💡 Show Answer</summary>

The input representation of BERT is formed by adding three components: Token Embedding, Segment Embedding, and Position Embedding. Adding these three together rather than concatenating (Concat) them seems counterintuitive at first glance, but is supported by multiple perspectives of rationality.

1. **Signal Processing Perspective**

    From the perspective of signal processing, a vector can be understood as a signal carrying specific information. The superposition of multiple signals is a basic operation in signal processing—for instance, signals from multiple channels in AM/FM radio broadcasting can be superimposed and transmitted through the same medium, while the decoder at the receiving end can disentangle the respective information from the mixed signal.

    Analogously, in BERT: the superposition of Token, Segment, and Position Embeddings forms a mixed signal, and the subsequent multi-layer Transformer (essentially a powerful non-linear function approximator) has the capability to disentangle information along each dimension from this mixed signal and utilize them separately. Prof. Xipeng Qiu holds a similar view in *Neural Networks and Deep Learning*: as long as the downstream network is sufficiently powerful, superimposed signals can be effectively decomposed and utilized.

2. **One-Hot Equivalence Perspective**

    Jianlin Su provided a rigorous algebraic proof of equivalence, demonstrating that "adding three Embeddings" is mathematically equivalent to "concatenating three One-hot vectors and passing them through a linear layer."

    The detailed derivation is as follows: Let vocabulary size be $V$, number of segment types be $S$, maximum sequence length be $L$, and Embedding dimension be $d$. For the token at position $i$, define:
    - Token One-hot vector $\mathbf{t} \in \mathbb{R}^V$;
    - Segment One-hot vector $\mathbf{s} \in \mathbb{R}^S$;
    - Position One-hot vector $\mathbf{p} \in \mathbb{R}^L$.
    Concatenating the three yields $\mathbf{x} = [\mathbf{t}; \mathbf{s}; \mathbf{p}] \in \mathbb{R}^{V+S+L}$, passing through a linear layer $W \in \mathbb{R}^{d \times (V+S+L)}$ (without bias):
    \[
    W\mathbf{x} = [W_t \mid W_s \mid W_p] \begin{bmatrix} \mathbf{t} \\ \mathbf{s} \\ \mathbf{p} \end{bmatrix} = W_t \mathbf{t} + W_s \mathbf{s} + W_p \mathbf{p}
    \]
    where $W_t \mathbf{t}$, $W_s \mathbf{s}$, and $W_p \mathbf{p}$ correspond exactly to the table-lookup Token Embedding, Segment Embedding, and Position Embedding, respectively. Thus:
    \[
    Concat(\mathbf{t}, \mathbf{s}, \mathbf{p}) \xrightarrow{Linear} Emb_token + Emb_seg + Emb_pos
    \]
    The two operations are completely equivalent under equivalent parameters, indicating that "addition" is not an information-lossy operation, but rather a compact implementation of Concat + Linear.

3. **Information Fusion Perspective**

    Understood from the representation learning standpoint, the three Embeddings reside in three distinct semantic subspaces:
    - Token Embedding encodes lexical semantic information (word meaning, part-of-speech, etc.);
    - Segment Embedding encodes sentence attribution information (belonging to sentence A or sentence B);
    - Position Embedding encodes sequence positional information (absolute position of the token in the sequence).
    Adding the three is equivalent to mapping vectors from the three subspaces via their respective linear embedding matrices into a unified $d$-dimensional feature space, and then performing element-wise additive fusion in that space. This fusion mode requires that the three types of information coexist in a complementary rather than conflicting manner in the same space, and BERT's training process (MLM + NSP) automatically drives the model to learn such a coordinated representation.

    Furthermore, compared to the Concat operation, addition does not increase the Embedding dimension. While maintaining representational capacity, it significantly reduces the number of parameters and computation cost, serving as a design choice that achieves a fine balance between expressiveness and computational efficiency.

#### Comparison and Connections Among the Three Perspectives

- **Signal Processing Perspective**: Emphasizes the decoupling capability of the network, offering an intuitive explanation;
- **One-Hot Equivalence Perspective**: Provides a rigorous algebraic proof, demonstrating that addition does not compromise information expression capacity;
- **Information Fusion Perspective**: Starts from the geometric meaning of representation spaces to explain the intrinsic meaning of the addition operation.

The three perspectives complement each other, collectively demonstrating that "Embedding Addition" is a design that is mathematically sound, engineering-efficient, and model-learnable.

</details>

**32. How Does Positional Encoding Grant the Model Order-Awareness Capabilities?**

<details>
<summary>💡 Show Answer</summary>

The self-attention mechanism is essentially a weighted sum over all tokens in a sequence, an operation that is insensitive to input order (i.e., shuffling the input order yields the same set of outputs, merely shuffled accordingly). This means the model itself cannot distinguish order differences such as "cat chases dog" versus "dog chases cat".

To address this issue, the original Transformer paper adopted **Sinusoidal Positional Encoding**: for each position pos and each dimension i, a fixed vector is computed using sin and cos functions of different frequencies, with formulas roughly as follows:

\begin{verbatim}
PE(pos, 2i)   = sin(pos / 10000^(2i/d_model))
PE(pos, 2i+1) = cos(pos / 10000^(2i/d_model))
\end{verbatim}

This positional vector is added directly to the word embedding vector to serve as the actual input to the model. Because encoding vectors generated for different positions are distinct and exhibit clear relative position relationships (translatable via linear transformations), the model is able to perceive the relative and absolute positions of tokens in the sequence through this additional information.

In addition to fixed trigonometric encodings, subsequent works introduced **learnable positional encodings** (such as BERT) and **relative positional encodings** (such as Transformer-XL, T5, RoPE rotary positional embeddings, etc.), which typically perform better in long-sequence modeling and extrapolation capacity.

</details>

**33. What Is Positional Encoding? Why Is It Necessary in Transformer? Please List at Least Two Implementation Approaches.**

<details>
<summary>💡 Show Answer</summary>

What is positional encoding? Positional Encoding (PE) is a vector of the same dimension as the token embedding, intended to inject information about the absolute or relative position of tokens in the input sequence into the model. It is added to the token embedding and then jointly fed into the bottom layers of the Transformer.

Why is it necessary? The core mechanism of the Transformer—self-attention—processes data as a set rather than a sequence during computation. It inherently contains no information regarding token order and is permutation-invariant. This means that if the order of tokens in the input sequence is shuffled, the output of the self-attention layer will be shuffled accordingly, but each token's individual output vector (ignoring softmax normalization) remains identical. This clearly contradicts the characteristics of natural language, where word order is crucial (e.g., "I hit you" and "You hit me" have completely opposite meanings). Therefore, position information must be explicitly provided to the model through an external mechanism, which is precisely the role of positional encoding.

At least two implementation approaches:

1. Sinusoidal Positional Encoding:

This is the method used in the original Transformer paper *Attention Is All You Need*. It uses sine and cosine functions of different frequencies to generate positional encodings, according to the following formulas:

\[PE_{(pos, 2i)} = \sin(pos / 10000^{2i/d_{model}})\]

\[PE_{(pos, 2i+1)} = \cos(pos / 10000^{2i/d_{model}})\]

where $pos$ is the position of the token in the sequence, $i$ is the dimension index in the encoding vector, and $d_{model}$ is the embedding dimension.

- Advantages:
- Extrapolatability: Able to handle sequences longer than the maximum sequence length encountered during training.
- Relative position information: The model can easily learn relative positional relationships because for any fixed offset $k$, $PE_{pos+k}$ can be represented as a linear function of $PE_{pos}$, making it easier for the model to capture relative position dependencies.

1. Learned Absolute Positional Encoding:

This approach treats positional encoding as part of the model parameters, learned through training. Specifically, a positional encoding matrix of shape `(max_sequence_length, embedding_dimension)` is created. When processing a sequence, the corresponding encoding vector is looked up from this matrix based on each token's position index and added to the token embedding. Models such as BERT and GPT-2 adopt this approach.

- Advantages: The pattern is more flexible, allowing the model to learn the position representation best suited for the data.
- Disadvantages: Cannot generalize to lengths exceeding the preset `max_sequence_length`. If longer sequences need to be processed, fine-tuning the positional encodings or adopting other strategies is required.

</details>

**34. Could you introduce RoPE in detail, and compare its advantages and disadvantages with absolute position encoding?**

<details>
<summary>💡 Show Answer</summary>

![Rotary Position Embedding (RoPE) 2D Complex Plane Rotation Mechanism](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig14_rope_rotation_mechanism.png)



Introduction to RoPE (Rotary Position Embedding): RoPE, full name Rotary Position Embedding, is currently one of the most mainstream position encoding schemes in Large Language Models (such as the LLaMA series, Qwen, etc.). It is an innovative method that incorporates position information into the self-attention mechanism.

Its core idea is: by rotating vectors, absolute position information is encoded into the Query and Key vectors, allowing the model to naturally utilize relative position information when calculating attention scores.

Working principle: RoPE no longer directly adds position vectors to word embeddings like traditional position encodings do. Its operation occurs after generating Q and K vectors and before calculating attention scores:

1. Dimension grouping: Group the $d$-dimensional features of Q and K vectors in pairs, treating them as $d/2$ two-dimensional vectors.
2. Construct rotation matrix: For position $m$ in the sequence, construct a position-dependent rotation matrix $R_m$. This matrix represents a rotation operation in two-dimensional space.
3. Rotate Q and K: Rotate each two-dimensional vector group using the corresponding rotation matrix $R_m$.

Mathematically, this process is equivalent to treating each two-dimensional vector $(x_m, x_{m+1})$ as a complex number and then multiplying it by a complex number $e^{im\theta}$, where $m$ is the position and $\theta$ is a preset constant related to the dimension. This operation only changes the phase (direction) of the vector, without changing its magnitude (length).

Key features: The cleverness of RoPE lies in the fact that when dot product operations are performed on the Query vector $q_m$ and Key vector $k_n$ of two rotated positions $m$ and $n$, the result depends only on their relative position $(m-n)$, and is independent of their absolute positions $m$ and $n$. This allows the self-attention mechanism to naturally possess awareness of relative position.

Advantages and disadvantages compared to absolute position encoding:

Advantages of RoPE:

1. Built-in relative position modeling: This is its biggest advantage. RoPE makes attention scores directly dependent on the relative distance between tokens, which aligns better with the property that syntactic and semantic dependencies in natural language are usually relative.
2. Good extrapolation capability: Due to its mathematical properties, RoPE performs exceptionally well when processing sequences longer than those seen during training, demonstrating strong length generalization capability, which is a key reason long-sequence LLMs favor it.
3. No extra trainable parameters introduced: RoPE is a functional, fixed encoding method that does not occupy model parameters like learnable position encodings do.
4. Dependency decay with increasing distance: The property of rotation causes tokens farther apart to exhibit periodic decay in their inner product relationship, matching the intuition that greater distance implies weaker correlation in language.

Disadvantages of RoPE:

1. Relatively complex theoretical understanding: The underlying mathematical principles (complex numbers, Euler's formula, rotation matrices) are more abstract than absolute position encodings based on direct addition.
2. Representation of absolute position information may be weaker: Although RoPE is derived from absolute positions, its core role in the attention mechanism is to reflect relative positions. For specific tasks that strongly rely on absolute position information (e.g., determining whether a word is at the beginning of a sentence), its effectiveness may not be as intuitive as using absolute position encodings directly.

</details>

**35. What is position encoding in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Transformers lack an inherent sense of sequence because they process input tokens in parallel. Position encoding addresses this issue by introducing sequence information:
- **Working principle**: Position encoding uses mathematical functions (such as sine and cosine) to assign a unique encoding to each token based on its position in the sequence.
- **Why it is important**: It helps the model distinguish between “The cat chased the dog” and “The dog chased the cat”.

</details>

**36. What position encodings are currently used in LLMs?**

<details>
<summary>💡 Show Answer</summary>

In LLMs, position encodings are mainly divided into two major categories: absolute position encoding and relative position encoding. The mainstream position encodings are as follows:

**Sinusoidal position encoding**: This is the position encoding method proposed in the original Transformer paper. It creates an encoding for each position through a combination of sine and cosine functions, with wavelengths forming a geometric progression, making each position's encoding unique and capable of capturing relative relationships between positions. The formula is as follows:

$$
    PE_{(pos,2i)} = \sin(pos/10000^{2i/d_{model}}), \quad PE_{(pos,2i+1)} = \cos(pos/10000^{2i/d_{model}})
$$

where $pos$ represents the position, $d_{model}$ represents the dimension of the embedding, and $2i, 2i+1$ represent the indices of different dimensions in the embedding.

Advantages: Simple implementation, high efficiency, good usability, and solid theoretical foundation. Disadvantages: Poor flexibility caused by absolute position encoding; poor extrapolation, making it difficult to extrapolate well to texts exceeding the pre-training length; difficult to represent effective relative position relationships.

**Rotary Position Embedding (RoPE)**: This is a relative position encoding method that encodes position information by rotating vectors. RoPE demonstrates unique advantages in specific tasks, especially when processing data with symmetric or periodic features.

$$
    \langle f_q(x_m,m), f_k(x_n,n) \rangle = g(x_m, x_n, m-n)
$$


Advantages: Preserves relative position relationships; good extrapolation capability; compatible with linear attention mechanisms. Disadvantages: Slightly more complex implementation; performance degradation still exists when extrapolating to ultra-long sequences (exceeding 20% of the pre-training length).

**Relative Position Encoding (ALiBi)**: This is a method that modifies the Attention structure so it can distinguish Tokens at different positions. ALiBi does not explicitly add encodings into the network; the only modification is adding a static, non-learnable bias after the QK dot product:

$$
    softmax(q_i K^T + m \cdot [-(i-1), \ldots, -2, -1, 0])
$$

$m$ is a head-specific hyperparameter. For $n$ heads, the step size ratio is generally $2^{-8/n}$.

Advantages: Simple implementation, reduces model embedding parameters, high efficiency, and good extrapolation capability. Disadvantages: Performance degradation still exists when extrapolating to ultra-long sequences.

The above introduces three mainstream position encoding methods, which act respectively before attention (sinusoidal encoding), during attention (integrating encoding into attention calculation), and after attention (adding a bias after computing the QK product).

</details>

**37. Write code for sinusoidal encoding and rotary position embedding by hand?**

<details>
<summary>💡 Show Answer</summary>

Core code for **sinusoidal position encoding**:
\begin{lstlisting}[language=Python]
position = torch.arange(0, max_len, dtype=torch.float32).unsqueeze(1)
two_i = torch.arange(0, d_model, 2, dtype=torch.float32)
div_term = torch.exp(two_i * -(math.log(10000.0) / d_model))
encodings = torch.zeros(max_len, d_model)
encodings[:, 0::2] = torch.sin(position * div_term)
encodings[:, 1::2] = torch.cos(position * div_term)
\end{lstlisting}

Core code for **rotary position embedding**:
\begin{lstlisting}[language=Python]
# Generate rotation matrix
def precompute_freqs_cis(dim: int, seq_len: int, theta: float = 10000.0):
    freqs = 1.0 / (theta ** (torch.arange(0, dim, 2)[: (dim // 2)].float() / dim))
    t = torch.arange(seq_len, device=freqs.device)
    freqs = torch.outer(t, freqs).float()  # Compute m * theta
    freqs_cis = torch.polar(torch.ones_like(freqs), freqs)  # Complex vector
    return freqs_cis

# Rotary position embedding computation
def apply_rotary_emb(xq, xk, freqs_cis):
    xq_ = xq.float().reshape(*xq.shape[:-1], -1, 2)
    xk_ = xk.float().reshape(*xk.shape[:-1], -1, 2)
    xq_ = torch.view_as_complex(xq_)
    xk_ = torch.view_as_complex(xk_)
    xq_out = torch.view_as_real(xq_ * freqs_cis).flatten(2)
    xk_out = torch.view_as_real(xk_ * freqs_cis).flatten(2)
    return xq_out.type_as(xq), xk_out.type_as(xk)
\end{lstlisting}

</details>

**38. Does ALiBi really have long-sequence dependency modeling capabilities?**

<details>
<summary>💡 Show Answer</summary>

The linear bias of ALiBi dictates that it cannot see very far within a single attention layer. The reason ALiBi possesses extrapolation ability is actually because it is essentially an approximate local sliding attention, relying on stacked multi-layer attention to perceive information exceeding the length of a single attention layer. This expansion of length dependency is proportional to the number of network layers, so its theoretical length is limited. In fact, RoPE can also apply a local sliding mask to achieve similar length expansion. Therefore, from this perspective, ALiBi is less like a position encoding and more like an attention post-processing trick; the local attention imposed by ALiBi serves to enhance locality, which aligns with the inherent characteristics of language modeling tasks.

</details>

**39. What extrapolation methods are used for inference beyond training length in LLMs?**

<details>
<summary>💡 Show Answer</summary>

The main approaches include ALiBi, Position Interpolation (PI), NTK-Aware methods, and the YaRN method.

Length extrapolation (Length Generalization) studies how to use shorter lengths during pre-training while generalizing to larger lengths during extrapolation. Good extrapolation performance means that when generalizing to ultra-long sequences, relevant metrics do not experience a significant drop, and the model performance remains robust.

The classic approaches currently studying length generalization are as follows:

**1. ALiBi Direct Extrapolation**: ALiBi mainly adds a non-learnable bias after computing the attention score, with the formula as follows:

Due to its linear bias, ALiBi cannot capture long-distance information within a single layer of attention. The reason ALiBi can extrapolate is that it resembles local sliding attention, capturing information beyond single-layer length through multi-layer attention; however, this information perception capacity is proportional to the number of network layers, which is theoretically limited.

**2. PI Interpolation**: Multiplies the position encodings of the predicted long text by a factor $L_{train} / L_{test}$, scaling them back into the training length range. Although position interpolation avoids out-of-bounds positions at long distances, it simultaneously compresses distances between neighboring tokens, severely disrupting the model's local resolution and causing PPL explosion. However, studies show that after long-text fine-tuning, PI can also yield acceptable results.

**3. NTK-aware Series**: The idea of NTK-aware Scaled RoPE is to modify the base in $\sin(m/base^{-2i/d})$ to $\sin(m/(base \cdot \alpha)^{-2i/d})$. According to analyses by Su Jianlin et al., this approach is analogous to a change of radix (base conversion). Theoretical analysis shows that this method achieves high-frequency extrapolation and low-frequency interpolation.

Dynamic NTK dynamically scales the hyperparameter as the sequence length increases, with its base scaling formula being:

$$
    base' = base \cdot \left(\frac{\alpha \cdot seq_len}{max_position_embeddings} - (\alpha - 1)\right)^{d/(d-2)}
$$


**4. ReRoPE and Leaky ReRoPE**: Proposed by Su Jianlin, combining extrapolation and interpolation methods. It first sets a window size $w$, using a position interval of size 1 within the window and a position interval of size $1/k$ outside the window. Regardless of input length, its position encoding range does not exceed $w$, allowing support for content of arbitrary length. In addition, Su Jianlin proposed a $\log n$ scaling factor, multiplying each $q$ by $\log n$ during testing to stabilize the distribution of attention scores.

**5. YaRN**: Consists of two parts of work. The first part is the design of flexible bases, using different bases for different dimensions: low dimensions correspond to high-order digits of numbers, extrapolated with constant angle; intermediate dimensions correspond to middle digits, interpolated gradually; high dimensions correspond to low-order digits, fully interpolated. The calculation formula is as follows:

The second part of the work introduces a temperature $t$ before performing softmax on logits, which uniformly affects perplexity. The calculation of attention weights is modified as follows:

The YaRN method surpasses all previous methods in both fine-tuned and non-fine-tuned scenarios. After fine-tuning on less than 0.1% of the original pre-training data, YaRN achieves state-of-the-art performance in context window extension.

</details>

**40. Please compare several common LLM architectures, such as Encoder-Only, Decoder-Only, and Encoder-Decoder, and explain the types of tasks each excels at.**

<details>
<summary>💡 Show Answer</summary>

LLM architectures can mainly be divided into three categories. Their core differences lie in which parts of the Transformer are used and the types of attention mechanisms employed, which directly determine the tasks they each excel at.

**1. Encoder-Only Architecture**

- Structure: Formed by stacking multiple Transformer Encoder layers.
- Core mechanism: Bidirectional self-attention mechanism. When processing any token in a sequence, the model can simultaneously attend to all tokens to its left and right. This enables the model to obtain very rich contextual representations.
- Tasks best suited for: Natural Language Understanding (NLU).
- Specific tasks:
- Classification tasks: Sentiment analysis, text classification.
- Sequence labeling: Named Entity Recognition (NER).
- Sentence relationship judgment: Natural Language Inference (NLI).
- Cloze completion: Pre-training tasks like BERT's Masked Language Model (MLM) itself.
- Reasons: The core of these tasks is to understand the deep meaning of the input text, where bidirectional context is crucial for accurate understanding. The outputs of such models are usually fixed labels or categories, rather than freely generated long text.

**2. Decoder-Only Architecture**

- Structure: Formed by stacking multiple Transformer Decoder layers, but removing the Encoder-Decoder cross-attention parts.
- Core mechanism: Unidirectional (causal) self-attention mechanism (Causal Self-Attention). When predicting the `t`-th token, the model can only attend to tokens from position `1` to `t-1`, and cannot see future information. This autoregressive property naturally suits generation tasks.
- Tasks best suited for: Natural Language Generation (NLG).
- Specific tasks:
- Open-ended text generation: Writing articles, stories, poems.
- Dialogue systems/chatbots: Such as ChatGPT.
- Code generation: Such as Copilot.
- In-context continuation (In-Context Learning).
- Reasons: The language generation process is sequential and left-to-right; the unidirectional attention of the Decoder-Only architecture perfectly simulates this process. Currently, the vast majority of general-purpose large language models adopt this architecture.

**3. Encoder-Decoder Architecture**

- Structure: Contains a full Encoder stack and a full Decoder stack.
- Core mechanism: The Encoder part uses bidirectional attention to encode the entire input sequence, forming a comprehensive contextual representation. When generating output, the Decoder part uses unidirectional attention on the already generated sequence on one hand, and attends to the Encoder's output via a Cross-Attention mechanism on the other hand, ensuring generated content is relevant to the input.
- Tasks best suited for: Sequence-to-Sequence (Seq2Seq).
- Specific tasks:
- Machine Translation: Translating one language (input sequence) into another language (output sequence).
- Text Summarization: Summarizing a long article (input sequence) into a few sentences (output sequence).
- Question Answering: Converting questions (input sequence) into answers (output sequence).
- Reasons: Such tasks require first gaining a complete, global understanding of the source sequence (completed by the Encoder), and then conditionally generating a target sequence based on this understanding (completed by the Decoder).

</details>

**41. What are the differences between prefix Decoder, causal Decoder, and Encoder-Decoder?**

<details>
<summary>💡 Show Answer</summary>

The differences between prefix Decoder, causal Decoder, and Encoder-Decoder lie in their different **attention masks**:

**Encoder-Decoder**:
- **Introduction**: Bidirectional attention on input, unidirectional attention on output
- **Representative models**: T5, Flan-T5, BART
- **Characteristics**: Employs bidirectional attention on input, leading to fuller encoding and understanding of questions
- **Applicable tasks**: Performs well on understanding-heavy NLP tasks
- **Disadvantages**: Performs poorly on long text generation tasks, low training efficiency

**causal Decoder**:
- **Characteristics**: Autoregressive language model; pre-training and downstream applications are fully consistent, strictly adhering to the rule that subsequent tokens can only see previous tokens
- **Applicable tasks**: Performs well on text generation tasks
- **Advantages**: High training efficiency, stronger zero-shot capabilities, possesses emergent abilities

**prefix Decoder**:
- **Characteristics**: Tokens in the prefix part can see each other; it is a compromise between causal Decoder and Encoder-Decoder
- **Disadvantages**: Low training efficiency

</details>

**42. What are Prefix LM and Causal LM?**

<details>
<summary>💡 Show Answer</summary>

**Causal LM (Causal Language Model)**:
- Each token can only attend to itself and all tokens before it (lower triangular mask).
- Training objective is standard "next token prediction".
- Representative models: GPT series, LLaMA.
- Advantages: Naturally suited for autoregressive generation; during inference, tokens can be generated step-by-step with KV Cache reuse.

**Prefix LM (Prefix Language Model)**:
- Allows bidirectional attention for tokens in the prefix section, while still employing causal attention for the generated section.
- Representative models: UniLM, GLM (early versions of ChatGLM).
- Advantages: The prefix section gains BERT-like bidirectional understanding capabilities while the generated section retains autoregressive properties.

**Core difference**: Attention mask design differs—Causal LM is unidirectional throughout; Prefix LM is bidirectional on the input prefix and unidirectional on the output section. Currently, almost all mainstream large models adopt pure Causal LM.

</details>

**43. Why are current LLMs overwhelmingly Decoder-only architectures?**

<details>
<summary>💡 Show Answer</summary>

1. **Unified pre-training objective, simple and scalable**: Uses a single "next token prediction" autoregressive objective, making data acquisition and training pipelines simpler and more unified.
2. **Naturally suited for generation tasks and In-Context Learning**: Decoder-only demonstrates stronger capabilities in zero-shot/few-shot learning.
3. **Simpler architecture and implementation, higher efficiency**: Stacking a single causal Transformer, combined with KV Cache, leads to higher inference efficiency.
4. **Supported by empirical/scaling experience**: Extensive experiments show that the paradigm of Decoder-only + large-scale pre-training + instruction tuning exhibits superior scalability and generalization capability.
5. **Theoretical expressiveness**: Studies indicate that Decoder-only models can theoretically simulate certain effects of bidirectional attention.

</details>

**44. Are you familiar with activation functions? Which activation functions are commonly used in LLMs, and why are they chosen?**

<details>
<summary>💡 Show Answer</summary>

Yes, I am familiar with activation functions. Activation functions are a vital component of neural networks, whose primary role is to introduce non-linearity into the network. Without activation functions, a multi-layer neural network is essentially equivalent to a single-layer linear model, unable to learn and fit complex data patterns.

In modern large language models (Transformer architecture), the two most commonly used activation functions are GeLU and SwiGLU.

1. GeLU (Gaussian Error Linear Unit):

- Introduction: GeLU was once the mainstream activation function in Transformer models, adopted by classic models such as BERT and GPT-2. Its mathematical form is $x \cdot \Phi(x)$, where $\Phi(x)$ is the cumulative distribution function of the Gaussian distribution.
- Why choose it?
- Smoothness: GeLU is a smooth approximation of ReLU. Compared to the non-differentiable jump of ReLU at 0, GeLU's smooth characteristic provides more stable gradients during optimization, benefiting model convergence.
- Stochastic regularization idea: GeLU can be viewed as combining the ideas of Dropout and ReLU. It deterministically zero-out or retains inputs based on their values: smaller inputs have a higher probability of being "zeroed out".

1. SwiGLU (Swish-Gated Linear Unit):

- Introduction: SwiGLU is currently the state-of-the-art and most mainstream choice, widely adopted by a series of modern LLMs such as LLaMA, PaLM, Mixtral, and Gemma. It belongs to the Gated Linear Unit (GLU) family of variants.
- Working principle: It splits the output $X$ of the first linear layer in the feed-forward network (FFN) into two parts, $A$ and $B$. It then calculates the output using the formula $Swish(A) \otimes B$, where $Swish(x) = x \cdot \sigma(x)$, $\sigma$ is the Sigmoid function, and $\otimes$ is element-wise multiplication.
- Why choose it?
- Gating Mechanism: The core advantage of SwiGLU lies in its "gating" design. Part $B$ can be viewed as a dynamic "gate" that controls which information in $Swish(A)$ can pass through and which needs to be suppressed based on the input content. This mechanism significantly enhances the model's expressive power, allowing FFN layers to process information more flexibly.
- Superior empirical results: Google's experiments in the PaLM paper showed that replacing standard GeLU or ReLU with SwiGLU significantly improves model performance (lowers perplexity). Although SwiGLU increases the parameter count of FFN layers (requiring two matrices instead of one), the resulting performance gains have proven to be well worth it.

</details>

**45. GLU and SwiGLU Activation Functions**

<details>
<summary>💡 Show Answer</summary>

![SwiGLU Gated Feed-Forward Network (FFN) Architecture and Computational Flow](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig15_swiglu_ffn_architecture.png)



**GLU (Gated Linear Unit)**:
- Basic form: $GLU(x) = (xW + b) \otimes \sigma(xV + c)$, which dynamically controls the flow of information through a gating mechanism.

**SwiGLU**:
- A variant of GLU that replaces the activation function in the gate with **Swish/SiLU** ($SiLU(x) = x \cdot \sigma(x)$).
- Specific application in the FFN layer (e.g., LLaMA): $FFN(x) = (SiLU(xW_1) \otimes (xW_3)) W_2$, which replaces the two linear transformation matrices in standard FFN with three matrices.
- **Advantages**: Experiments demonstrate that Transformers using SwiGLU outperform standard FFNs using ReLU/GELU on multiple downstream tasks. The tradeoff is the introduction of a third weight matrix, requiring a moderate reduction in the hidden layer dimension to keep the total parameter count roughly constant.

</details>

**46. What is the Role of the FFN Layer in LLaMA?**

<details>
<summary>💡 Show Answer</summary>

- **Model Structure**: For pure self-attention without residual connections and MLPs, stacking $n$ layers causes the overall rank of the model to collapse quickly, with all representations converging to a single vector. Adding MLPs and residual connections significantly mitigates this.
- **Increasing Expressive Power**: The linear transformation after dimension expansion is usually accompanied by a non-linear activation function, breaking the limitations of linear models.
- **Storing Knowledge**: FFN accounts for a large portion of the parameters in Transformer. The first linear transformation acts as Key Memory, and the second linear transformation acts as Value Memory.

</details>

**47. What is the Calculation Formula for FFN?**

<details>
<summary>💡 Show Answer</summary>

The Feed-Forward Network (FFN) is an important component in each encoder/decoder layer of the Transformer, following immediately after the attention sub-layer. A standard FFN consists of two linear transformations and a non-linear activation function:


$$
	FFN(x) = ReLU(xW_1 + b_1)W_2 + b_2
$$


where $W_1 \in \mathbb{R}^{d \times 4d}$ is the up-projection matrix (expanding the hidden dimension from $d$ to $4d$), $W_2 \in \mathbb{R}^{4d \times d}$ is the down-projection matrix (compressing the dimension back from $4d$ to $d$), and $b_1 \in \mathbb{R}^{4d}, b_2 \in \mathbb{R}^{d}$ are bias terms.

**On the Choice of the Intermediate Dimension $4d$**

The original Transformer paper set the intermediate dimension to $4d$, a choice proven in practice to be a reasonable empirical value:
- Expanding the dimension to $4d$ provides sufficient "width" for the model to learn complex non-linear feature transformations, which can be understood as enriching and recombining the semantic representations of tokens in a higher-dimensional space.
- The parameter count of the FFN module is $2 \times d \times 4d = 8d^2$ (ignoring bias), accounting for roughly $2/3$ of the total parameters in a Transformer, making it the primary capacity source for storing knowledge in the model. Studies show that FFN layers can be approximately viewed as Key-Value Memory, implicitly storing factual knowledge from the training data.
- For modern LLMs (such as variants removing bias and using Pre-Norm), the basic structure of FFN remains unchanged, but the activation functions and dimension ratios have been adjusted (see subsequent sections).

The FFN is computed independently at each position (position-wise), meaning the same FFN is applied separately to each token's representation vector in the sequence, with parameters shared across all positions, complementing the cross-position interactions of the attention mechanism.

</details>

**48. Introduce the Calculation Formula for GELU**

<details>
<summary>💡 Show Answer</summary>

GELU (Gaussian Error Linear Unit) is an activation function proposed by Hendrycks & Gimpel in 2016, with its exact definition given by:


$$
	GELU(x) = x \cdot \Phi(x) = x \cdot \frac{1}{2}\left[1 + erf\!\left(\frac{x}{\sqrt{2}}\right)\right]
$$


where $\Phi(x)$ is the cumulative distribution function (CDF) of the standard normal distribution $\mathcal{N}(0,1)$, and $erf(\cdot)$ is the Gaussian error function. Since exact calculation involves the error function, two common approximations are frequently used in practice:


$$
	GELU(x) \approx 0.5x\left(1 + \tanh\!\left[\sqrt{\frac{2}{\pi}}\left(x + 0.044715x^3\right)\right]\right)
$$


or the simpler Sigmoid approximation:

$$
	GELU(x) \approx x \cdot \sigma(1.702x)
$$


**Main Characteristics of GELU**:
- **Smoothness**: Infinitely differentiable over the entire domain with continuous gradients, helping the optimizer converge stably and avoiding the non-differentiability problem of ReLU at the origin.
- **Non-monotonicity**: It exhibits a global minimum in the negative region (at approximately $x \approx -0.17$), after which it approaches zero as $x$ decreases. Similar to Swish, this non-monotonicity is believed to help enhance expressive capacity.
- **Probabilistic Interpretation**: $\Phi(x)$ can be understood as the probability that input $x$ is "retained"—the larger the input value, the closer the pass probability is to 1; the smaller the input value, the closer to 0. GELU can be viewed as the expectation of stochastic regularization (similar to Dropout) applied to the input.
- **Relationship with ReLU**: When the input is large, $\Phi(x) \to 1$, making GELU approximately linear; when the input is extremely small, $\Phi(x) \to 0$, making the output approach zero. Its overall behavior is similar to ReLU, but smoother.

</details>

**49. Introduce the Calculation Formula for Swish**

<details>
<summary>💡 Show Answer</summary>

Swish (also known as SiLU, Sigmoid Linear Unit) was proposed by Google Brain in 2017 and is defined as:


$$
	Swish(x) = x \cdot \sigma(\beta x) = \frac{x}{1 + e^{-\beta x}}
$$


where $\sigma(\cdot)$ is the Sigmoid function, and $\beta$ is a tunable hyperparameter (or learnable parameter). In most practical applications, $\beta = 1$ is used, at which point Swish reduces to SiLU:


$$
	SiLU(x) = x \cdot \sigma(x) = \frac{x}{1 + e^{-x}}
$$


**Main Characteristics of Swish**:
- **Smoothness**: Like GELU, it is infinitely differentiable across the entire domain, leading to more stable gradient flow.
- **Non-monotonicity**: It reaches a minimum of approximately $-0.278$ at $x \approx -1.28$, and monotonically increases thereafter. Non-zero outputs in the negative region allow gradients to backpropagate, avoiding the "dying ReLU" problem.
- **Unbounded Above, Bounded Below**: The output is unbounded above (growing linearly as $x \to +\infty$) but bounded below (at approximately $-0.278$), combining the gradient propagation capability of linear functions with the expressive capacity of non-linear functions.
- **Learnability**: When $\beta$ is treated as a learnable parameter, Swish can adaptively adjust the activation shape during training; as $\beta \to 0$, it approaches the linear function $x/2$; as $\beta \to \infty$, it approaches ReLU.
- **Relationship with GELU**: The shapes of both are very close, $GELU(x) \approx Swish_{1.702}(x)$, and performance differences in practice are usually negligible.

</details>

**50. Calculation Formula for FFN Block Using GLU (Gated Linear Unit)**

<details>
<summary>💡 Show Answer</summary>

GLU (Gated Linear Unit) was proposed by Dauphin et al. in 2017, dynamically controlling information flow by introducing a gating mechanism:


$$
	GLU(x, W, V, b, c) = \sigma(xW + b) \otimes (xV + c)
$$


where $\sigma(\cdot)$ is the Sigmoid function, and $\otimes$ denotes element-wise (Hadamard) product. The Sigmoid branch acts as a "gate" that determines the proportion of information passing through each dimension, while the linear branch provides the content representation to be controlled.

Integrating GLU into the FFN block and generalizing the activation function to an arbitrary function $Act(\cdot)$ yields:


$$
	FFN_{GLU}(x) = \bigl(Act(xW_1) \otimes xV\bigr) W_2
$$


**Comparison of Parameter Counts**:
- **Traditional FFN**: Two weight matrices, $W_1 \in \mathbb{R}^{d \times 4d}$ and $W_2 \in \mathbb{R}^{4d \times d}$, with a parameter count of $8d^2$.
- **GLU FFN**: Three weight matrices, $W_1, V \in \mathbb{R}^{d \times d_{ff}}$ and $W_2 \in \mathbb{R}^{d_{ff} \times d}$. If $d_{ff} = 4d$ is maintained, the parameter count increases by 50%. To maintain the same parameter count as traditional FFN, $d_{ff}$ is usually adjusted to $\frac{2}{3} \times 4d = \frac{8}{3}d$ (rounded to the nearest suitable multiple).

Depending on the choice of the gated activation function, GLU has the following common variants:
- $Act = \sigma$: Original GLU
- $Act = ReLU$: ReGLU
- $Act = GELU$: GeGLU (adopted by Gemma)
- $Act = Swish$: SwiGLU (adopted by LLaMA, PaLM, etc.)

</details>

**51. What Custom Modifications Are Made to the FFN Layer in LLaMA?**

<details>
<summary>💡 Show Answer</summary>

The LLaMA series introduces several improvements to the standard FFN to enhance parameter efficiency, training stability, and model performance:

1. **SwiGLU Activation Function**: Replaces ReLU in standard FFN with a Swish gated variant (setting $\beta = 1$):

    $$
    FFN_{SwiGLU}(x) = \bigl(Swish_1(xW_1) \otimes (xW_3)\bigr) W_2
    $$

    The FFN layer of LLaMA requires training three weight matrices in total: $W_1$ (up-projection of the gating branch), $W_3$ (up-projection of the content branch), and $W_2$ (down-projection output). To keep the parameter count roughly equivalent to the original two-matrix FFN, LLaMA sets the intermediate dimension to $\frac{2}{3} \times 4d$ rounded to the nearest multiple of 128 (e.g., when $d=4096$, the intermediate dimension is 11008).

    Compared to GELU, SwiGLU shows better perplexity and downstream task performance in experiments with models like PaLM, and is considered one of the key sources of LLaMA's competitive performance.

2. **Removal of Bias Terms**: All linear layers in LLaMA (including $W_1$, $W_2$, and $W_3$ in FFN) do not use bias terms. This design simplifies the model structure, reduces the parameter count, and has been proven not to affect model performance during large-scale training, while also contributing to cleaner tensor-parallel implementations.

3. **RMSNorm Pre-Normalization (Pre-Norm)**: Unlike Post-Norm in the original Transformer, LLaMA uses RMSNorm for normalization at the input of the FFN sub-layer (Pre-Norm), removing the mean-centering step in normalization and retaining only scaling:

    $$
    RMSNorm(x) = \frac{x}{RMS(x)} \cdot \gamma, \quad RMS(x) = \sqrt{\frac{1}{d}\sum_{i=1}^d x_i^2}
    $$

    Combining Pre-Norm with RMSNorm significantly improves training stability for deep models and reduces normalization computation overhead by about 7%--64% (compared to LayerNorm).

4. **MoE Concepts (An Extension Direction of LLaMA)**: Mixture of Experts (MoE) replaces a single FFN with $N$ parallel "expert" FFNs, dynamically selecting Top-$K$ experts for computation per token via a learnable routing network (Router):

    $$
    MoE-FFN(x) = \sum_{i=1}^{N} g_i(x) \cdot FFN_i(x), \quad g(x) = TopK\!\left(softmax(xW_g),\ K\right)
    $$

    The core advantage of MoE lies in trading low inference computational cost (activating only $K$ experts per token) for an enormous total parameter count, achieving parameter expansion under "sparse activation". Mixtral 8$\times$7B adopts this architecture with a total parameter count of approximately 47B, but only activated parameters of about 13B per token, resulting in inference costs comparable to a 13B dense model while achieving performance close to the 70B scale. DeepSeek-V2/V3 further combines MoE with fine-grained expert division and shared expert mechanisms, pushing parameter efficiency to new limits.

</details>

**52. Why Is the FFN Mapping Set to $\frac{8{3}d$ When Using SwiGLU in LLM FFNs?}**

<details>
<summary>💡 Show Answer</summary>

This conversion factor maintains the total parameter count and computation amount under the SwiGLU structure to be roughly equal to the original standard ReLU-FFN.

Parameter count of standard ReLU-FFN: $Params_{ReLU} = 2 \cdot d \cdot f_{ffn}$, where $f_{ffn} = 4d$.

Parameter count of SwiGLU-FFN: $Params_{SwiGLU} = 3 \cdot d \cdot m$ (three matrices: $W$, $V$, and $W_{out}$).

Setting them equal: $3 \cdot d \cdot m = 2 \cdot d \cdot 4d$, solving yields $m = \frac{8}{3}d$.

**One-sentence summary**: $\frac{8}{3}$ is simply a conversion scaling factor that keeps the parameter/computation amount of SwiGLU $\approx$ the original ReLU-FFN.

</details>

**53. Which Activation Functions Are Used by Various LLMs?**

<details>
<summary>💡 Show Answer</summary>

Activation functions are core components of Feed-Forward Network (FFN) modules, directly affecting the model's expressive capacity and training stability. Modern large language models have generally migrated from early ReLU to smoother gated activation functions that preserve information better. The following summarizes the activation function choices of mainstream models and their underlying design considerations:

- **GPT Series (OpenAI): GELU**

    GPT-1/2/3 and subsequent GPT series all adopt **GELU** (Gaussian Error Linear Unit). It is defined as:
    \[
    GELU(x) = x \cdot \Phi(x) = x \cdot \frac{1}{2}\left[1 + erf\!\left(\frac{x}{\sqrt{2}}\right)\right]
    \]
    where $\Phi(x)$ is the cumulative distribution function of the standard normal distribution. In practice, a commonly used approximation formula is:
    \[
    GeLU(x) \approx 0.5x\left(1 + \tanh\!\left[\sqrt{\frac{2}{\pi}}\left(x + 0.044715x^3\right)\right]\right)
    \]
    Compared to ReLU, GELU has a smooth, non-zero output in the negative region, leading to more stable gradients and superior performance in NLP tasks compared to ReLU. It serves as the standard activation function for BERT and the GPT series.

- **LLaMA Series (Meta): SwiGLU**

    The entire LLaMA 1/2/3 series adopts **SwiGLU**, which is a Swish variant of GLU (Gated Linear Unit) proposed by Noam Shazeer in 2020. It is defined as:
    \[
    SwiGLU(\mathbf{x}, \mathbf{W}, \mathbf{V}, \mathbf{b}, \mathbf{c}) = Swish_1(\mathbf{x}\mathbf{W} + \mathbf{b}) \otimes (\mathbf{x}\mathbf{V} + \mathbf{c})
    \]
    where $Swish_1(x) = x \cdot \sigma(x)$, and $\otimes$ denotes element-wise multiplication. In simplified form, the FFN module structure of LLaMA is:
    \[
    FFN_{SwiGLU}(\mathbf{x}) = \left(Swish(\mathbf{x} W_1) \otimes \mathbf{x} W_3\right) W_2
    \]
    Compared to standard FFN, SwiGLU introduces a third weight matrix $W_3$, slightly increasing the parameter count. To maintain the total parameter count, LLaMA adjusts the hidden dimension from $4d$ to $\frac{8}{3}d$ (rounded to the nearest multiple of 128). SwiGLU has been proven to outperform GELU and ReLU on multiple NLP benchmarks, making it the mainstream choice for current open-source large models.

- **ChatGLM2 (Tsinghua KEG & Zhipu AI): SwiGLU (Upgraded from GELU)**

    The first generation of ChatGLM used GELU; starting from **ChatGLM2**, it was upgraded to **SwiGLU**, while combining a series of modern architectural improvements such as RMSNorm and RoPE, aligning overall with the LLaMA architecture. This upgrade brought dual improvements in training stability and downstream task performance.

- **PaLM (Google): SwiGLU**

    Google's PaLM (Pathways Language Model) and its subsequent version PaLM-2 also adopt **SwiGLU**, combined with a Parallel Transformer Block design (parallel attention and FFN layers) to further boost training efficiency.

- **Activation Function Selection of Other Mainstream Models**

    - **Mistral / Mixtral (Mistral AI)**: SwiGLU
    - **Qwen Series (Alibaba)**: SwiGLU
    - **DeepSeek Series**: SwiGLU (used in MoE expert layers in some versions)
    - **Gemma (Google DeepMind)**: GeGLU (gated version of GELU)
    - **BERT (Google)**: GELU
    - **T5 (Google)**: ReLU (original version), with subsequent T5 variants adopting GELU or GeGLU

**Summary of Activation Function Evolution**

- **ReLU**: Mainstream in early models, simple to compute, but suffers from the "dying ReLU" problem (zero gradient for negative values); gradually phased out in large models.
- **GELU**: Smooth approximation of ReLU with non-zero output in the negative region, providing more stable training; the choice for BERT and the GPT series.
- **Swish** (i.e., SiLU): $Swish(x) = x \cdot \sigma(x)$, unbounded above, bounded below, non-monotonic, outperforming GELU on multiple tasks.
- **SwiGLU / GeGLU**: Introduces a gating mechanism on top of Swish or GELU, dynamically controlling information flow through element-wise multiplication; currently the top choice for state-of-the-art large models.

From a trend perspective, modern large language models have almost overwhelmingly chosen **SwiGLU**, which strikes a good balance among parameter efficiency, training stability, and downstream performance, becoming the de facto industry standard.

</details>

**54. How Does the Mixture of Experts (MoE) Model Effectively Expand Model Parameter Scale Without Significantly Increasing Inference Cost? Please Briefly Describe Its Working Principle.**

<details>
<summary>💡 Show Answer</summary>

![Mixture of Experts (MoE) Top-k Router Gating Network and Expert Dispatch Architecture](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig16_moe_routing_architecture.png)



Mixture of Experts (MoE) is a model architecture whose core idea is to resolve the contradiction between model scale and computational cost through a "sparse activation" strategy. It allows the model to have an enormous total parameter count while activating only a small subset of parameters when processing any given input, thereby significantly increasing model capacity without substantially increasing inference cost (FLOPs).

The working principle is as follows:

1. Replacing FFN layers with "Experts":

- In standard Transformer architectures, one of the most computationally expensive parts is the Feed-Forward Network (FFN) layer.
- The MoE architecture replaces part or all of the FFN layers in the model with MoE layers. An MoE layer consists of two components:
- $N$ "Experts": Each expert is itself an independent, smaller-scale FFN.
- 1 "Gating Network" or "Router": This is a small neural network, typically a simple linear layer.

1. Dynamic Routing Decisions:

- When the vector representation of a token arrives at the MoE layer, it is first sent to the router.
- The role of the router is to make a "decision" on which experts are most suitable to process this token. It outputs a vector containing $N$ scores, representing the "matching degree" between the token and the $N$ experts.

1. Top-$K$ Sparse Activation:

- After the scores output by the router are normalized via Softmax, the system does not activate all experts. Instead, it selects only the Top-$K$ experts with the highest scores ($K$ is typically very small, such as 1 or 2).
- This is the key to "sparse activation": for each token, only a very small number ($K$) of experts are activated for computation, while the remaining ($N-K$) experts do not participate at all, incurring no computational cost.

1. Weighted Output:

- The selected $K$ experts process the input token vector separately, producing $K$ output vectors.
- The final output is the weighted sum of these $K$ output vectors, where the weights are also determined by the router's output scores.

How to Achieve "Large Parameter Count with Low Cost"?

- Suppose a model has 8 experts ($N=8$), and activates only 2 at a time ($K=2$), such as the Mixtral-8$\times$7B model.
- Total Parameter Count: The total parameter count of the model is the sum of the parameters in all shared parts (such as attention layers) plus the parameters of all 8 experts. This allows the total parameter scale of the model to be very large (e.g., reaching 47B).
- Inference Cost: However, during a single forward pass (inference), for any given token, only the shared parts and the 2 activated experts actually participate in the computation. Therefore, its computational complexity (FLOPs) is roughly equal to a much smaller "dense" model (such as a 13B model).
- Conclusion: MoE successfully decouples the total parameter count (representing the model's knowledge capacity) from the computational complexity per inference (representing the model's speed and cost), thereby achieving "obtaining the knowledge of a large model at the cost of a small model."

</details>

**55. What is MoE?**

<details>
<summary>💡 Show Answer</summary>

MoE (Mixture of Experts) is an advanced neural network architecture whose core idea is to combine multiple specialized sub-models (called "experts") with a dynamic routing mechanism to selectively activate a subset of experts for computation based on the characteristics of the input data.

Core Components:
1. **Experts**: Each expert is an independent sub-model, typically a small neural network, focused on processing a specific subset of input data or a specific task.
2. **Gating Network**: Responsible for dynamically selecting the most suitable expert or combination of experts based on input features and assigning weights.
3. **Sparsity Mechanism**: Only activates a subset of experts per input rather than the entire model, thereby significantly reducing computational costs.

Working Principle: When input data enters the model, the gating network computes weights for each expert based on input features and selects the expert(s) with the highest weights (or Top-$K$ experts) for processing. Finally, the outputs of the experts are aggregated according to their weights to form the final output of the model.

</details>

**56. What Are the Characteristics of MoE?**

<details>
<summary>💡 Show Answer</summary>

1. **Sparsity**: The sparsity of MoE is mainly reflected in two aspects: first, the sparsity of model architecture, meaning that not all experts are activated simultaneously; second, the sparsity of data processing, meaning that only the data most relevant to the task is processed by the selected experts.
2. **Efficiency**: Compared with dense models, MoE can achieve the same level of performance faster under the same computational resource constraints. This is mainly attributed to its sparse gating mechanism, which enables the model to concentrate on processing critical information and reduce unnecessary computational overhead.
3. **Scalability**: The MoE architecture boasts excellent scalability, allowing a significant increase in model capacity and dataset size without increasing the computational burden.

</details>

**57. Which Layer in Original LLMs Does MoE Replace?**

<details>
<summary>💡 Show Answer</summary>

MoE (Mixture of Experts) primarily replaces the Feed-Forward Network (FFN) layer in traditional Transformer architectures within large language models. This section elaborates on this from the perspectives of the functional role of FFN layers, the motivation behind MoE replacement, concrete structural comparisons, and mainstream implementation schemes.

#### Why the FFN Layer Instead of the Attention Layer?

In a standard Transformer, each Block consists of two core sub-layers:

- **Multi-Head Self-Attention (MHA) Layer**: Responsible for modeling dependencies between different positions in a sequence. It has a relatively small parameter count and highly structured computation (three-way Query-Key-Value projections);
- **Feed-Forward Network (FFN) Layer**: Performs position-wise independent non-linear transformations on representations. It is usually composed of two linear transformations and an activation function, accounting for roughly $\frac{2}{3}$ of the total parameters in a Transformer block.

FFN layers are naturally suited to be replaced by MoE for the following reasons:

1. **FFN is the primary carrier of parameters**: In a standard Transformer, if the hidden dimension is $d$, the intermediate layer dimension of the FFN is typically $4d$, yielding an FFN parameter count of $2 \times d \times 4d = 8d^2$, whereas the MHA layer has roughly $4d^2$ parameters. FFN accounts for a larger proportion of parameters and is the core bottleneck of model capacity;
2. **Position-wise independent computation in FFN**: FFN processes each token position independently (position-wise) without interactions across different tokens, making the routing mechanism of "different tokens dispatched to different experts" naturally viable;
3. **Attention layers are difficult to sparsify**: Global interactions exist among Query, Key, and Value in the MHA layer. If modified into experts, the design of the routing mechanism and training stability face immense challenges, and academic exploration in this direction remains relatively limited.

#### Structure of Standard FFN Layers

The FFN layer structure of a standard Transformer is as follows (taking SwiGLU activation as an example, which is used in the LLaMA series):

\begin{lstlisting}[style=promptstyle]
	# Standard FFN layer (SwiGLU variant)
	class FeedForward(nn.Module):
	def __init__(self, d_model, d_ff):
	self.w1 = nn.Linear(d_model, d_ff)   # Gate path
	self.w2 = nn.Linear(d_ff, d_model)   # Output projection
	self.w3 = nn.Linear(d_model, d_ff)   # Activation path

	def forward(self, x):
	# All tokens pass through the exact same set of parameters
	return self.w2(F.silu(self.w1(x)) * self.w3(x))
\end{lstlisting}

Key characteristic: Regardless of semantic content, every token undergoes transformation via **the exact same set of parameters**. This is a "one-size-fits-all" design that lacks specialized processing capabilities for different types of input.

#### How MoE Replaces the FFN Layer

MoE replaces a single FFN with $N$ parallel FFN Experts and introduces a lightweight **Gating Network / Router**, which determines which experts each token should be routed to for processing:

\[
MoE(\mathbf{x}) = \sum_{i=1}^{N} G(\mathbf{x})_i \cdot Expert_i(\mathbf{x})
\]

where $G(\mathbf{x}) \in \mathbb{R}^N$ is the output weight vector of the gating network. Typically, only the Top-$k$ ($k \ll N$) experts have non-zero weights, while the remaining experts do not participate in computation. Taking Top-2 routing as an example:

\begin{lstlisting}[style=promptstyle]
	# MoE layer (simplified schematic)
	class MoELayer(nn.Module):
	def __init__(self, d_model, d_ff, num_experts=8, top_k=2):
	self.experts = nn.ModuleList(
	[FeedForward(d_model, d_ff) for _ in range(num_experts)]
	)
	self.gate = nn.Linear(d_model, num_experts)  # Gating network

	def forward(self, x):
	# Compute routing scores for each token across experts
	gate_scores = F.softmax(self.gate(x), dim=-1)

	# Only activate Top-2 experts
	topk_scores, topk_indices = gate_scores.topk(2, dim=-1)
	topk_scores = topk_scores / topk_scores.sum(dim=-1, keepdim=True)

	# Weighted sum of outputs from activated experts
	output = sum(
	topk_scores[..., i:i+1] * self.experts[topk_indices[..., i]](x)
	for i in range(2)
	)
	return output
\end{lstlisting}

#### Core Advantages of MoE Replacing FFN

1. **Decoupling Parameter Count from Computation**: The total parameter count of an MoE model grows linearly with the number of experts, but the actual computation per token depends only on the activated Top-$k$ experts, remaining constant. For example, an MoE layer with 8 experts activating 2 per token has 8 times the parameters of a dense FFN, but its computational cost (FLOPs) is only $2/8 = 25%$ of its total potential parameters (sparse activation) or about 2 times that of a single dense FFN (when $k=2$);

2. **Expert Specialization**: Different experts spontaneously learn to process different types of inputs during training—for instance, some experts specialize in code, some in natural language, and some in mathematical reasoning—achieving an implicit division of labor;

3. **Higher Model Capacity Ceiling**: Under the same computational budget, MoE can deploy far more total parameters than dense models, thereby breaking through the parameter scale ceiling.

#### MoE Implementations in Mainstream LLMs

Current mainstream MoE LLMs all follow the design paradigm of "keeping the attention layers unchanged and replacing FFN layers with MoE":

- **Mixtral 8$\times$7B (Mistral AI)**: Replaces the FFN in each Transformer block with 8 experts, activating Top-2 per token, with approximately 13B active parameters and 47B total parameters;
- **DeepSeek-V2 / V3**: Introduces Fine-Grained Expert Segmentation and Shared Expert mechanisms to further optimize load balancing on top of MoE;
- **Qwen-MoE Series**: Also uses FFN replacement as its core, adopting designs with a higher number of experts (e.g., 64 experts with Top-4 activation);
- **GPT-4 (Speculated)**: Reportedly adopts an MoE architecture according to multiple sources; although not officially confirmed by OpenAI, the industry generally considers it to employ a similar design.

#### Summary

MoE replaces the **Feed-Forward Network (FFN) layer** in Transformers, rather than the attention layer. This choice is by no means arbitrary; it is jointly determined by the high parameter proportion, position-wise independence, and routing feasibility of FFNs. By replacing a single FFN with a structure of multiple parallel experts plus gating routing, MoE dramatically increases the total parameter capacity and specialized processing capabilities of the model with virtually no increase in per-token inference FLOPs, becoming the core technical route for current ultra-large language models to break through parameter scale bottlenecks.

</details>

**58. What Are the Advantages of MoE Compared to FFN?**

<details>
<summary>💡 Show Answer</summary>

- MoE dynamically selects a subset of experts for computation via a gating network, rather than performing dense computation on all inputs like FFN does. This sparsity significantly reduces computational overhead.
- FFN layers are primarily responsible for feature transformations, whereas MoE's expert networks can dynamically choose different feature transformation paths for different input data, thereby better adapting to diverse tasks.

</details>

**59. How to Solve the Load Balancing Problem in MoE?**

<details>
<summary>💡 Show Answer</summary>

**1. Auxiliary Loss**

To achieve load balancing, auxiliary loss functions are typically introduced during the training of MoE models:
- **Load Balancing Loss**: Calculates the discrepancy between the actual amount of input data processed by each expert and the ideal processing amount, incorporating it as part of the total model loss.
- **Coefficient of Variation (CV)**: Measures the variance in importance scores among experts by calculating the coefficient of variation of their importance scores. Optimizing the auxiliary loss reduces the CV score, thereby achieving load balancing.

**2. Dynamically Adjusting Expert Selection Probabilities**

- **KeepTopK Strategy**: Breaks preference for specific experts by introducing trainable Gaussian noise to make selection more randomized. Simultaneously, setting non-top weights to negative infinity ensures that only the Top-$K$ experts are activated.
- **Auxiliary-Loss-Free Strategy**: DeepSeek proposed an auxiliary-loss-free load balancing strategy that achieves sequence-level load balancing by dynamically adjusting the bias terms in expert selection probabilities, avoiding interference with model convergence caused by additional loss terms.

**3. Expert Capacity Limits and Adjustments**

Sets a capacity upper bound for each expert, limiting the number of samples it can process at one time. If an expert reaches its capacity limit, pending tasks can be temporarily buffered, or load can be optimized by dynamically adjusting expert capacity.

**4. Distributed Training and Load Balancing**

- **Redundant Expert Strategy**: Dynamically replicates heavily loaded experts when certain experts experience excessive loads, distributing redundant copies across different compute nodes (e.g., GPUs) to prevent a single node from becoming a performance bottleneck.
- **Hierarchical Load Balancing**: Packages expert groups evenly onto nodes to ensure load balance across nodes, then replicates experts within each node, and finally assigns the replicated experts to individual GPUs.
- **Global Load Balancing**: Globally replicates experts when the expert parallelism scale is large, assigning the replicated experts across GPUs.

**5. Simplified Routing Strategies**

For instance, Switch Transformer adopts a simplified Top-1 routing strategy, routing input to only one expert at a time to reduce computation and communication overhead while ensuring uniform expert utilization through a load balancing loss.

</details>

**60. What Are the Benefits of Using MoE?**

<details>
<summary>💡 Show Answer</summary>

**Basic Structure of MoE (Mixture of Experts)**

An MoE layer consists of two components: several **Expert Networks** $\{E_1, E_2, \ldots, E_n\}$ and a **Gating Network** $G$. For an input $x$, the output of the MoE layer is:
\[
y = \sum_{i=1}^{n} G(x)_i \cdot E_i(x)
\]
In a sparse MoE, $G(x)$ returns non-zero weights only for the Top-$k$ experts, while the remaining experts do not participate in computation.

**Advantages**:
1. **Efficiency**: Through a sparse activation mechanism, each token activates only the Top-$k$ experts (typically $k=1$ or $k=2$) rather than all $n$ experts. Assuming the FFN computational cost per expert is $C$, the computational cost per step for a sparse MoE is $k \cdot C$, whereas for a dense model (equivalent to activating all $n$ experts) it is $n \cdot C$, reducing the computational load to $k/n$.

2. **Scalability**: Model parameter scale can be expanded by increasing the number of experts **without significantly increasing computation** (i.e., “decoupling parameter count from computation”). For example, increasing the number of experts from 8 to 64 multiplies total model parameters by 8, while computation per token remains unchanged (still activating $k$ experts). This property makes MoE highly suitable for constructing ultra-large models, such as Mixtral 8$\times$7B, DeepSeek-V2/V3, etc.

3. **Flexibility and Specialization**: Different experts naturally tend to process different types of inputs (such as different languages or domain-specific knowledge) during training, achieving an **implicit division of labor** that grants the model stronger adaptability across diverse tasks.

4. **Training Efficiency**: Under the same computational budget, MoE models typically achieve superior performance compared to dense models of equal computational cost, exhibiting higher parameter efficiency in FLOP-constrained scenarios.

MoE models are widely applied in fields such as natural language processing and image recognition, and are particularly well suited for handling large-scale data and complex tasks.

</details>

**61. What Are the Primary Parallelism Strategies for MoE Training?**

<details>
<summary>💡 Show Answer</summary>

Parallelism strategies in MoE training mainly revolve around efficiently allocating and utilizing compute resources. Common strategies include:

**1. Data Parallelism (DP)**: Splits training data into mini-batches, where each compute unit (e.g., GPU) processes a subset of the data. In MoE architectures, both gating and expert networks are replicated on every compute unit. Devices independently perform forward passes and synchronize gradients via AllReduce. DP is simple to implement, but each device must store complete model parameters, resulting in high GPU memory pressure.

**2. Tensor Parallelism (TP)**: Splits the computation of individual operators (e.g., matrix multiplication) across multiple devices to execute in parallel. In MoE, gating networks are usually replicated on each compute unit, whereas weight matrices within expert networks are split row-wise or column-wise across devices. TP effectively reduces per-device memory consumption, but introduces considerable inter-device communication (AllReduce).

**3. Expert Parallelism (EP)**: A parallel strategy unique to MoE, which distributes different experts across different compute units, such that each device only stores and computes its assigned experts. After tokens are routed by the gating network, they are transmitted via All-to-All communication to the device hosting the corresponding expert for computation, and results are sent back to the original device. EP significantly reduces per-device GPU memory burden while allowing the model to scale to more experts; however, All-to-All communication can become a bottleneck in large-scale clusters.

**4. Pipeline Parallelism (PP)**: Assigns different layers (Transformer Blocks) of the model to different devices, passing data sequentially through devices in a pipeline fashion. In MoE models, because computational workloads across layers may be uneven (alternating between MoE and dense layers), pipeline partitioning must be carefully designed to avoid device idling (Pipeline Bubbles).

**5. Hybrid Parallelism Strategies**: In practical large-scale training, multiple parallel strategies are typically combined to achieve optimal memory and computational efficiency, such as:
- **EP + DP**: Expert parallelism manages parameter distribution while data parallelism increases throughput; this is the most common combination in MoE training.
- **EP + DP + TP**: Further applies tensor splitting to large matrices within experts on top of EP + DP, suitable for scenarios where individual experts still possess large parameter sizes.
- **EP + PP + TP**: Ultra-large-scale MoE training (e.g., hundreds of billions of parameters) typically employs 3D parallelism; models such as DeepSeek-V3 utilize similar strategies during training.

</details>

**62. What Are the Similarities and Differences Between MoE and Ensemble Learning?**

<details>
<summary>💡 Show Answer</summary>

MoE and ensemble learning share similar underlying ideas—both integrate multiple models—yet significant differences exist between them regarding objectives, architecture, and inference mechanisms:

\begin{center}


| **Comparison Dimension** | **MoE** | **Ensemble Learning** |
| --- | --- | --- |
| Core Objective | Conditional computation, increase capacity | Reduce variance/bias, improve generalization |
| Expert/Base Model Activation | Sparse activation (only subset of experts) | All base models participate (dense) |
| Routing Mechanism | Learnable gating network (dynamic routing) | Fixed rules (averaging, voting, Stacking) |
| Training Method | End-to-end joint training | Can be trained independently then ensembled |
| Inference Overhead | Low (sparse activation) | High (all models perform inference) |


\end{center}

MoE primarily divides prediction tasks into multiple subtasks and uses a routing network to decide which experts to invoke for current data. In existing MoE implementations, usually only a subset of experts is activated, rendering the model **sparse**. Ensemble learning uses multiple base models for the prediction task and merges their outputs using Bagging or Boosting methods, rendering its overall model structure **dense**.

</details>

**63. How Is the Routing Network Designed in MoE?**

<details>
<summary>💡 Show Answer</summary>

**Basic Structure**

The routing network (Gating Network / Router) is typically a **lightweight linear layer** with significantly fewer parameters than the expert networks. For the hidden state $x \in \mathbb{R}^d$ of an input token, the routing network computes the logit scores for each expert:
\[
h(x) = x \cdot W_g, \quad W_g \in \mathbb{R}^{d \times n}
\]
where $n$ is the total number of experts.

**Top-$k$ Sparse Routing**

In mainstream implementations, the Top-$k$ experts with the highest scores are selected for computation, while the remaining expert weights are set to $-\infty$ (which evaluate to 0 after softmax):
\[
G(x) = softmax(TopK(h(x), k))
\]
The final output is the weighted sum of outputs from the selected experts:
\[
y = \sum_{i \in TopK} G(x)_i \cdot E_i(x)
\]

**Load Balancing Design**

Naive Top-$k$ routing is prone to **Expert Collapse**, where the gating network repeatedly selects a small subset of experts, leaving other experts untrained. Common solutions include:

- **Auxiliary Load Balancing Loss**: Adds a penalty term to the training loss to encourage uniform assignment of tokens to each expert:
    \[
    \mathcal{L}_{aux} = \alpha \cdot n \cdot \sum_{i=1}^{n} f_i \cdot P_i
    \]
    where $f_i$ is the proportion of tokens actually processed by the $i$-th expert, $P_i$ is the average probability assigned to the $i$-th expert by the gating network, and $\alpha$ is a balancing coefficient. Switch Transformer, Mixtral, etc., all adopt this method.

- **Noisy Top-$k$ Gating**: Adds tunable noise to the logits to increase routing randomness (see next section for details).

- **Expert Capacity**: Sets a maximum token capacity (Capacity Factor) for each expert; tokens exceeding capacity are either dropped or routed to other experts, forcing load balance.

</details>

**64. What Is the Difference Between Sparse and Dense MoE?**

<details>
<summary>💡 Show Answer</summary>

- **Sparse MoE**: Each token is routed only to the Top-$k$ ($k \ll n$) experts, while remaining experts do not participate in computation, resulting in a sparse gating weight vector. Computational FLOPs scale with $k$ and are independent of the total expert count $n$, making it suitable for large-scale model scaling. Current mainstream LLMs (Mixtral, DeepSeek-V2/V3, Qwen-MoE, etc.) all adopt Sparse MoE. The trade-off is the requirement of All-to-All communication for expert routing and susceptibility to load imbalance.

- **Dense MoE**: Selects all $n$ experts to participate in computation, combining expert outputs via weighted fusion according to gating weights $G(x)_i$. This is equivalent to a standard FFN with larger parameters (all expert weights are activated), where computation scales proportionally with $n$. Dense MoE requires no routing communication and exhibits more stable training, but sacrifices the computational efficiency advantages of sparse activation, seeing fewer practical applications.

</details>

**65. What Is the Routing Strategy of Switch Transformer?**

<details>
<summary>💡 Show Answer</summary>

Switch Transformer (Google, 2021) adopts a minimal **Top-1 routing strategy**, where each token is routed to only **one** expert ($k=1$):
\[
G(x) = softmax(x \cdot W_g), \quad select  \arg\max_i G(x)_i
\]

**Design Motivation**:
- Reduces per-token expert communication from Top-$k$ ($k>1$) down to 1 time, dramatically decreasing All-to-All communication overhead.
- Simplifies the routing mechanism and lowers implementation complexity, while empirical results show Top-1 routing performs comparably to Top-2 on most tasks.

**Load Balancing**: Switch Transformer introduces an **auxiliary load balancing loss** and sets a **Capacity Factor** ($\geq 1.0$). Tokens exceeding capacity skip the current MoE layer directly (passed via residual connection), thereby forcefully enforcing uniform load distribution across experts.

</details>

**66. What Is the Original Motivation Behind Adding Noise to the Routing Network?**

<details>
<summary>💡 Show Answer</summary>

**Background**: In early training, gating network parameters are close to random initialization. If an expert happens to receive a higher logit score and is frequently selected, it obtains more gradient updates, leading to even higher scores in subsequent iterations. This creates a **positive feedback loop**, ultimately causing the vast majority of tokens to be routed to only a few experts (expert collapse).

**Noisy Routing Approach**: Adds standard normal noise to the routing logits, scaled before taking the Top-$k$:
\[
h(x)_i = (x \cdot W_g)_i + \epsilon \cdot softplus\!\left((x \cdot W_n)_i\right), \quad \epsilon \sim \mathcal{N}(0, 1)
\]
where $W_n$ is a learnable noise scale matrix, and $softplus$ guarantees non-negative noise magnitudes.

**Role of Noise**:
1. **Breaking Symmetry**: At early stages of training, expert logits are close to each other; noise randomly dispatches different tokens to different experts, preventing early “winner-takes-all” phenomena.
2. **Increasing Exploration**: Similar to the $\epsilon$-greedy strategy in reinforcement learning, noise gives slightly lower-scoring experts a chance to be selected, thereby receiving gradient updates and ensuring all experts are adequately trained.
3. **Promoting Load Balance**: Noise scatters routing decisions, working in tandem with the auxiliary load balancing loss to further mitigate expert load imbalance.

As training progresses, the gating network gradually learns sensible routing strategies; the relative impact of noise decreases, and routing decisions stabilize.

</details>

**67. How do Traditional MoE and Current Mainstream MoE Select Experts?**

<details>
<summary>💡 Show Answer</summary>

**Traditional MoE Selection Method**: Learn a gating network denoted as $W_g$. The softmax of the product of $W_g$ and input $x$ is denoted as $G$, where $G$ represents the weight of each expert. When multiple experts are aggregated, they are aggregated in a **dense** manner (all experts participate):

$$
	y = \sum_{i=1}^{n} G(x)_i \cdot E_i(x), \quad G(x) = softmax(x \cdot W_g)
$$


**Mainstream MoE Selection Method**: Computed using **Noisy Top-$k$ Gating**. The noisy logit values are obtained through a learnable weight matrix $W_g$ and a noise matrix $W_n$. After taking the Top-$k$, softmax is computed only for these $k$ positions, while the weights of remaining experts are set to zero. Then, aggregation is performed in the same way as above:

$$
	h(x)_i = (x \cdot W_g)_i + \epsilon \cdot softplus((x \cdot W_n)_i)
$$


$$
	G(x) = softmax(TopK(h(x),\ k))
$$


$$
	y = \sum_{i \in TopK} G(x)_i \cdot E_i(x)
$$


**Comparison between the two methods**:

\begin{center}


| **Comparison Dimension** | **Traditional MoE** | **Mainstream MoE (Noisy Top-$k$)** |
| --- | --- | --- |
| Number of Activated Experts | All $n$ (Dense) | Top-$k$ (Sparse) |
| Computational Complexity | $O(n \cdot C)$ | $O(k \cdot C)$, $k \ll n$ |
| Load Balancing | Naturally balanced | Requires auxiliary loss + noise |
| Scalability | Poor (computation grows linearly with $n$) | Good (computation is independent of $n$) |


\end{center}

</details>

**68. How to Resolve the Load Imbalance Phenomenon in MoE Training?**

<details>
<summary>💡 Show Answer</summary>

**Method 1: Apply dropout after the gating network**

The sparsity of MoE is similar in principle to dropout. MoE chooses to activate a certain number of expert models according to the specific context of the task to complete it, whereas dropout randomly deactivates neurons in the neural network, retaining only a subset of parameters during each training iteration. This not only endows the network with sparse characteristics and reduces parameter overhead across the entire network, but also lowers the probability of model overfitting and improves generalization ability.

**Method 2: Introduce soft constraints**

Introduce an additional loss function: For a batch of training samples, define an importance metric $Importance(X)$ for each expert, which is the sum of the gating output values for that expert within the batch. At the same time, define the loss term $L_{importance}(X)$ and add it to the overall loss of the model:

$$
    Importance(X) = \sum_{x \in X} G(x)
$$


$$
    L_{importance}(X) = w_{importance} \cdot CV(Importance(X))^2
$$


where $CV(X)$ denotes the coefficient of variation, which reflects the dispersion of the distribution. If an extreme distribution occurs, the CV value will be high.

In addition, the Router z-loss introduced by ST-MoE significantly improves training stability while maintaining model performance:

$$
    L_z(x) = \frac{1}{B} \sum_{i=1}^{B} \left( \log \sum_{j=1}^{N} e^{x_j^{(i)}} \right)^2
$$


where $B$ is the number of tokens, $N$ is the number of experts, and $x \in \mathbb{R}^{B \times N}$ represents the logits input to the router. This penalizes large logits entering the gating network; using z-loss before the router is beneficial.

</details>

**69. Why is MoE Training Prone to Instability?**

<details>
<summary>💡 Show Answer</summary>

1. The gating mechanism must be properly designed to control routing to experts; otherwise, the model can easily collapse onto a single expert.
2. Outputs of sparse models exhibit high variance and are sensitive to random seeds (determining initial parameter values, training data shuffling, values to drop, etc.), so different training runs can yield different performance.
3. Modern training widely employs mixed precision, where low-precision formats can cause gradient instability, leading to unstable training.
4. During training, because each input sample only triggers a subset of experts, different experts experience different gradient update frequencies. This unbalanced gradient update can lead to unstable parameter updates and affect the convergence of the overall model.

</details>

**70. Advantages and Disadvantages of the MoE Algorithm?**

<details>
<summary>💡 Show Answer</summary>

**Advantages of MoE**:
- **Accelerated Pre-training Speed**: MoE models can undergo efficient pre-training using fewer computational resources compared to dense models.
- **Faster Inference Speed**: Because only a subset of experts is activated during inference, the inference speed of MoE models is typically faster than dense models with the same total parameter count.
- **Enhanced Model Scalability**: The MoE architecture allows models to increase parameter count while keeping computational cost constant.
- **Multi-task Learning Capabilities**: MoE excels in multi-task learning; for instance, Switch Transformer demonstrated performance gains across all 101 languages.

**Challenges Faced by MoE**:
- **Training Instability**: Varying gradient update frequencies across different experts can cause unstable model parameter updates.
- **Load Balancing Issues**: The randomness of the routing mechanism and characteristics of input data can cause certain experts to carry significantly higher computational loads than others.
- **Fine-tuning Challenges**: MoE models may face uncertain transfer learning performance during fine-tuning.
- **Inference Efficiency**: During the inference phase, the MoE model needs to select suitable experts based on inputs, which increases routing computational complexity.
- **Deployment Complexity**: MoE models are generally more complex than traditional dense models, potentially requiring hardware-specific optimization in practical deployment.

</details>

**71. Theoretical Analysis of Why MoE is Effective**

<details>
<summary>💡 Show Answer</summary>

The paper “Towards Understanding Mixture of Experts in Deep Learning” analyzes the effectiveness of MoE and concludes that the cluster structure of the underlying problem and the non-linearity of experts are the keys to MoE's success. To demonstrate this viewpoint, the paper conducted a classification experiment and found that using a single expert made it difficult to classify multiple classes effectively, whereas using multiple non-linear expert modules successfully solved this issue. The paper theoretically proved that the routing network can learn cluster center features, thereby decomposing complex input problems into multiple simple sub-problems, which can each be solved by a single expert.

Summary: The design of the MoE routing network helps discover the cluster structure of underlying problems, thereby effectively breaking down the problem for individual experts to solve. In addition, MoE can be viewed as an ensemble of multiple models; from an ensemble learning perspective, the result of a multi-model ensemble yields better generalization than a single model.

</details>

**72. What Does Fine-Grained Experts Mean in DeepSeek-MoE?**

<details>
<summary>💡 Show Answer</summary>

The meaning of “fine-grained” in DeepSeek-MoE can be summarized in one sentence: **Splitting original “large and comprehensive” experts into many “small and specialized” micro-experts, enabling the model to use more flexible and precise combinations to process each input without increasing computational complexity.**

Specific approach:
1. Split each traditional FFN expert along the hidden dimension into $m$ pieces, reducing the intermediate dimension to $1/m$ of its original size, thereby obtaining $m$ times as many “micro-experts”.
2. To maintain the same computational budget, scale the number of activated experts per token to $mK$ (previously selecting $K$, now selecting $mK$).
3. The combinatorial space explodes exponentially as a result: for example, selecting 8 out of 64 micro-experts yields up to 440 million combinations, whereas selecting 2 out of 16 traditional experts yields only 120 combinations.
4. Each micro-expert focuses on an extremely narrow sub-domain of knowledge, thereby enhancing specialization and overall accuracy.
5. Paired with a set of shared experts that are always activated to handle general knowledge, avoiding redundant storage and reducing redundancy.

In a word: Fine-grained = **smaller experts, larger quantity, more flexible combinations, and more specialized knowledge**, pushing the “divide and conquer” strategy of MoE to the extreme without increasing parameters or compute.

</details>

**73. How is the Fine-Grained Expert Code Implemented in DeepSeek-MoE?**

<details>
<summary>💡 Show Answer</summary>

**1. Micro-Expert Weight Splitting**

The original large expert weight $\mathbf{W} \in \mathbb{R}^{d_{model} \times d_{ff}}$ is split along the hidden dimension into $m$ parts, yielding micro-expert weights:

$$
    \mathbf{W} = [\mathbf{W}_1, \mathbf{W}_2, \dots, \mathbf{W}_m], \quad \mathbf{W}_i \in \mathbb{R}^{d_{model} \times (d_{ff}/m)}
$$


The intermediate dimension of each micro-expert is $d_{ff}^{micro} = \frac{d_{ff}}{m}$.

**2. Conservation of Computation**

Traditional MoE activates $K$ experts per token, with total FLOPs $FLOP_{old} = K \cdot (2 d_{model} d_{ff})$.

With fine-grained experts, $mK$ micro-experts are activated, each with dimension $d_{ff}/m$. The total computation is:

$$
    FLOP_{new} = mK \cdot \left(2 d_{model} \frac{d_{ff}}{m}\right) = K \cdot (2 d_{model} d_{ff}) = FLOP_{old}
$$


**3. Routing and Output (Micro-Expert Part)**

For input token $\mathbf{x} \in \mathbb{R}^{d_{model}}$:

$$
    logits(\mathbf{x}) = \mathbf{x}^{\top} \mathbf{E}, \quad \mathbf{E} \in \mathbb{R}^{d_{model} \times N_{micro}}, \quad N_{micro} = m N_{expert}
$$


$$
    g_i = Softmax_i(Top-mK(logits(\mathbf{x})))
$$


Micro-expert output:

$$
    \mathbf{y}_{micro} = \sum_{i \in \mathcal{T}_{mK}} g_i \mathbf{W}_i \mathbf{x}
$$


**4. Shared Experts (Always Activated)**

Final output:

$$
    \mathbf{y}_{final} = \sum_{i \in \mathcal{T}_{mK}} g_i \mathbf{W}_i \mathbf{x} + \sum_{j=1}^{s} \mathbf{W}^{shared}_j \mathbf{x}
$$


**By splitting the large matrix $\mathbf{W}$ along the hidden dimension into $m$ parts while increasing the activated expert count from $K$ to $mK$, the computational complexity remains unchanged while the routing combination space is exponentially expanded.**

</details>

**74. Why Did DeepSeek's MoE Switch Gating from Softmax to Sigmoid?**

<details>
<summary>💡 Show Answer</summary>

The core reasons why DeepSeek-V3 switched MoE gating from softmax to sigmoid:

1. **Expert Dimension Explosion**: V3 routed experts increased from 160 to 256. The higher the dimension, the more softmax normalization squashes scores toward $1/N$, drastically reducing Top-$K$ discriminability. Softmax subtracts the maximum value from all logits to prevent overflow, takes exp, and divides by the sum. When expert count $N$ is large, logit variance is often small, making post-exp values extremely close, resulting in normalized outputs $\approx 1/N$.
2. **Numerical Stability**: Softmax requires a global summation where small floating-point errors in FP32 are amplified; sigmoid computes each expert independently, avoiding accumulation errors.
3. **More Stable Gradients**: Softmax has small gradients in saturation regions and is prone to “winner-take-all”; sigmoid features smooth gradients, rendering training more stable.
4. **“Multi-label” Perspective**: Activating 8 experts is essentially a multi-label problem. Sigmoid assigns an independent score between 0 and 1 to each expert, which is more natural than softmax's “mutually exclusive” assumption.

</details>

**75. How is the Sequence-Level Load Balancing Loss Computed in DeepSeek?**

<details>
<summary>💡 Show Answer</summary>

In DeepSeek-V3's MoE, the “sequence-level load balancing loss” is no longer computed independently for each token as in earlier work. Instead, an entire sequence is treated as a statistical unit, balancing using the “routing count” across all tokens in the sequence.

**1. Sequence-Level Statistics**

Summarize across the entire sequence:
- Total times expert $i$ is selected: $f_i = \sum_{t=1}^{T} \mathbb{1}_{i,t}$
- Cumulative gate value of expert $i$: $P_i = \sum_{t=1}^{T} g_{i,t}$

**2. Sequence-Level Load Balancing Loss**

We want $f_i$ to be as close to the “average load” $\frac{T \cdot K}{N}$ as possible:

$$
    \mathcal{L}_{bal} = \sum_{i=1}^{N} \left(f_i - \frac{T \cdot K}{N}\right)^2
$$


When the load across all experts is perfectly equal, the loss is 0.

**3. Usage During Training**

This loss is added to the total loss only during the training phase, with a small loss weight coefficient (e.g., 1e-2). It is completely removed during inference, without affecting generation quality.

</details>

**76. How to Handle Dropped Tokens in DeepSeek's Device-Level Token Dropping Strategy?**

<details>
<summary>💡 Show Answer</summary>

Although various load balancing losses can guide the model toward load balance during training, they cannot strictly guarantee complete load balance. Therefore, DeepSeek-MoE introduces a device-level token dropping strategy during both training and inference:

1. **Calculate Device Capacity**: For input tokens in each batch, first calculate the average capacity per device $C = \frac{Total Batch Tokens}{Number of Devices}$.
2. **Sort and Prepare for Dropping**: For the actual number of tokens allocated to the current device $T_a$, sort them in descending order by routing score. If $T_a > C$, drop the excess tokens from the tail so that the total count equals $C$ exactly.
3. **Behavior After Dropping**: Dropped tokens still retain their residual paths. They do not enter any expert network in the current MoE layer, but the residual connection remains active, passing the token's hidden state to the next Transformer layer. Since expert routing across different layers is decoupled, these tokens still have opportunities to be routed and participate in expert computation in subsequent layers.
4. **Training-Inference Consistency**: During training, 10% of samples per batch do not execute token dropping to simulate the “no-drop” scenario of inference and prevent distribution shift. During inference, token dropping is enabled by default to ensure strict control over compute load.

In a word: Through “sorting by routing score + dropping tail tokens”, strict device-level compute load balancing is achieved without disrupting the residual stream, while preserving a small fraction of non-dropped samples during training to stabilize distribution.

</details>

**77. What is Parameter Sharing, and How Does It Improve LLM Efficiency?**

<details>
<summary>💡 Show Answer</summary>

Parameter sharing is a technique where multiple layers or parts of a model share the same set of parameters. This reduces the total parameter count of the model without sacrificing performance. It offers several advantages:

- **Memory Efficiency**: Sharing parameters reduces the memory footprint of the model, making it more efficient in both storage and computation.
- **Improved Generalization Ability**: By reusing parameters, the model is less prone to overfitting on specific tasks or datasets, thereby improving its generalization ability on new tasks.
- **Faster Training and Inference**: Fewer parameters mean the model can be trained and deployed faster, making it better suited for resource-constrained environments.
Parameter sharing is commonly used in models such as BERT and GPT to enhance scalability without significantly increasing parameter counts.

</details>

**78. What Role Do Embeddings Play in LLMs?**

<details>
<summary>💡 Show Answer</summary>

Embeddings are vector representations of tokens that encode:
- Semantic information (meaning).
- Syntactic information (structure).
They convert discrete tokens into continuous values for model input.

**Types**:
- Word Embeddings: Represent entire words.
- Subword Embeddings: Handle unknown/rare words.

**Examples**: Word2Vec, GloVe, and learned embeddings in Transformers.

</details>

**79. What is the Role of Contextual Embeddings in LLMs, and How Do They Differ from Static Embeddings?**

<details>
<summary>💡 Show Answer</summary>

Contextual embeddings are dynamic word representations generated by models like Transformers that change according to surrounding words in a sentence. In contrast, static embeddings (such as Word2Vec or GloVe) assign a fixed representation to each word regardless of its context. Key advantages of contextual embeddings include:

- **Capturing Polysemy**: Contextual embeddings can represent multiple meanings of a word based on context, whereas static embeddings struggle when dealing with words that have different meanings across different sentences.
- **Better Performance**: Because they consider the full context of a word within a sentence, contextual embeddings perform better in tasks such as sentiment analysis, machine translation, and question answering.
- **Greater Flexibility**: Contextual embeddings adapt to the input, capturing subtle relationships between words and sentences more effectively.
LLMs like BERT and GPT rely on contextual embeddings to achieve state-of-the-art performance across NLP tasks.

</details>

**80. What is the Difference Between Embedding Short and Long Content?**

<details>
<summary>💡 Show Answer</summary>

There are several key differences between embedding short and long content. For short content, capturing the complete meaning can be challenging. Because there are fewer words, embedding models struggle to grasp the subtle nuances of the text. This may lead similar short texts to be deemed more similar simply due to their brevity rather than actual semantic similarity.

On the other hand, long content provides a richer information pool for embedding models. It allows analyzing relationships among words throughout the whole text, taking into account factors like sentence structure and topical development. This can create more accurate and comprehensive embeddings reflecting the full meaning of the content. However, overly long content may also present challenges, as certain models may struggle to process massive amounts of information and might lose some fine details.

</details>

**81. How Are Embedding Models Used in LLM Applications?**

<details>
<summary>💡 Show Answer</summary>

Embedding models in LLM applications act like a search engine, helping the LLM find the most relevant information snippets from massive datasets. This helps the LLM provide more accurate, informative, and helpful responses. Here is a good example:

Suppose the LLM is a powerful language expert, but it needs a method to understand the underlying meaning behind a user's words. The embedding model steps in and converts text inputs (queries, documents) into numerical representations called embeddings. These embeddings capture semantic relationships between words and concepts.

LLM applications frequently utilize Retrieval-Augmented Generation (RAG). In RAG, the embedding model converts both the user query and relevant information from a knowledge base into embeddings. Then, it finds the embedded information in the knowledge base that is most similar to the user query. This retrieved information, along with the user query, becomes the context for the LLM. Finally, the LLM utilizes this context to generate responses that are both relevant and informative.

</details>

**82. What is Context Window Size, and How Does It Affect LLM Performance?**

<details>
<summary>💡 Show Answer</summary>

Context window size refers to the maximum number of tokens a model can process at one time. It significantly impacts LLM performance because:

- **Longer Context Understanding**: A larger context window allows the model to consider a larger portion of the input sequence, improving its ability to capture long-range dependencies and relationships between tokens.
- **Performance on Long Text**: In tasks such as document summarization or long-form question answering, a larger context window enables the model to process longer inputs without truncating critical information.
- **Trade-off with Compute**: Increasing context window size requires more computational resources, as the complexity of the self-attention mechanism grows quadratically with sequence length.
Selecting an appropriate context window size is crucial for balancing model performance and computational efficiency based on the task at hand.

</details>

**83. What is Tokenization? Compare BPE and WordPiece, Two Mainstream Subword Tokenization Algorithms**

<details>
<summary>💡 Show Answer</summary>

What is tokenization (Tokenization)? Tokenization is the process of breaking raw text strings into individual units (called "tokens"), and mapping these tokens to unique integer IDs. This is the first step for natural language processing models to process text, because models can only handle numerical input.

Modern large language models generally adopt subword (Subword) tokenization algorithms, which fall between word-level and character-level segmentation. The benefits of doing so are:

1. Effectively handling out-of-vocabulary (OOV) words: Any rare or new word can be decomposed into combinations of known subwords, avoiding "unknown" tags.
2. Balancing vocabulary size and sequence length: Compared to word-level, the vocabulary size is greatly reduced; compared to character-level, the generated sequence length is not excessively long, balancing efficiency.
3. Preserving morphological information: Words like "running" and "runner" can share the subword "run", allowing the model to understand relationships between word roots and affixes.

BPE vs. WordPiece

BPE and WordPiece are two of the most mainstream subword tokenization algorithms. Their vocabulary construction processes are similar, but they differ in decision criteria for merging subwords.

	**BPE (Byte Pair Encoding)**

- How it works:

1. Initialization: The vocabulary consists of all basic characters appearing in the corpus.
2. Iterative merging: Repeat the following steps until reaching the preset vocabulary size:

a.  Count the occurrence frequency of all adjacent token pairs across the entire corpus. b.  Find the token pair with the highest frequency (e.g., `('e', 's')`). c.  Merge this token pair into a new, longer token (`'es'`), and add it to the vocabulary. d.  In the corpus, replace all occurrences of this token pair with the new token.

- Applied models: GPT series, Llama, etc.
- Characteristics: The algorithm idea is simple and intuitive, entirely based on the occurrence frequency of symbol pairs in data.

	**WordPiece**

- How it works:

1. Initialization: As with BPE, the vocabulary also starts from all basic characters.
2. Iterative merging (core difference): When choosing which two subwords to merge, WordPiece is not based on frequency, but on the likelihood of the language model. It tests all possible merges and selects the merge operation that maximizes the likelihood of the training data.

- Intuitively understood as: if considering the corpus as a language model, each merge should maximize the probability of this language model generating the current corpus. It tends to merge character combinations with stronger internal cohesion.
- Applied models: BERT, DistilBERT, Electra.
- Characteristics: During segmentation, WordPiece usually adds special symbols (such as `##`) before subwords that are not at the beginning of a word; for example, "tokenization" might be segmented as `("token", "##ization")`.

Summary of key differences: | Feature | BPE (Byte Pair Encoding) | WordPiece | | :--------------- | :------------------------------------------- | :--------------------------------------------------------- | | Merge decision criteria | Frequency-driven: Merges adjacent subword pairs that appear most frequently. | Likelihood-driven: Merges subword pairs that maximize the corpus language model likelihood. | | Theoretical basis | Data compression algorithm, simple and efficient. | Probabilistic language model, theoretically superior. | | Representative models | GPT, Llama, RoBERTa | BERT, T5 |

</details>

**84. How Do LLMs Handle Out-of-Vocabulary (OOV) Words?**

<details>
<summary>💡 Show Answer</summary>

LLMs handle OOV words using subword tokenization techniques, such as:
- **Byte Pair Encoding (BPE)**: Decomposes words into smaller units, such as prefixes, suffixes, or character pairs.
- **WordPiece**: Similar to BPE, but with slightly different merge criteria.
- **Unigram Language Model**: Selects the most probable subword sequence.
This ensures that even unseen words can be represented as combinations of known subwords.

</details>

**85. How Many Chinese Characters or English Words Correspond to 1 Token?**

<details>
<summary>💡 Show Answer</summary>

Token is the basic unit for large language models to process text. Internally, all inputs and outputs exist in the form of token sequences rather than raw characters. Tokenization efficiency varies significantly across different languages and characters; understanding this helps to better estimate context length and costs.

#### Token Correspondence in English

For English text, mainstream models (such as the GPT series and Claude series) usually adopt BPE (Byte Pair Encoding) or similar subword tokenization algorithms. General patterns are as follows:

- **Common English words**: Most common words (such as `hello`, `world`, `language`) correspond to **1 token**.
- **Longer or rare words**: Split into multiple subword tokens; for example, `unbelievable` might be segmented into three tokens: `un`, `believ`, and `able`.
- **Rule of thumb**: On average, **1 token $\approx$ 4 English characters**, or roughly **0.75 English words**. In other words, 100 tokens roughly correspond to 75 English words.
- **Spaces and punctuation**: Spaces are usually merged with the immediately following word into a single token (such as `_hello`), while punctuation marks generally occupy 1 token individually.

#### Token Correspondence in Chinese

Chinese is an ideographic writing system where each Chinese character is relatively independent semantically, but its tokenization method differs from English:

- **Common Chinese characters**: In most tokenizers, **1 Chinese character usually corresponds to 1 to 2 tokens**. This is because UTF-8 encoding of Chinese characters usually takes 3 bytes, and BPE vocabularies may either directly map common Chinese characters or fallback to byte-level encoding.
- **Rule of thumb**: Rough estimation: **1 token $\approx$ 0.5 to 1 Chinese character**, i.e., on average **1 Chinese character requires about 1 to 2 tokens**.
- **Comparison with English**: For Chinese text with identical semantic content, the token count is usually higher than the corresponding English text. This means when processing Chinese, the "utilization efficiency" of the context window is relatively lower, and costs are correspondingly higher.

#### Differences Across Model Tokenizers

Different models use different tokenizers, resulting in variations in token correspondence:

- **OpenAI (GPT series)**: Uses the `tiktoken` library with BPE vocabularies such as cl100k_base, averaging about 1.5 to 2 tokens per Chinese character.
- **Anthropic (Claude series)**: Uses a proprietary tokenizer optimized for multilingual support, with Chinese token efficiency similar to the GPT series.
- **Meta (LLaMA series)**: Early versions had limited Chinese support and lower token efficiency; LLaMA 3 improved the multilingual vocabulary.
- **Alibaba (Qwen series)**, **Baidu (ERNIE series)**, and other Chinese-optimized models: Expanded vocabularies for Chinese, where 1 Chinese character can often directly map to 1 token, resulting in significantly higher token efficiency.

#### Example Comparison

The following concrete examples illustrate differences in token counts:

\begin{lstlisting}[style=promptstyle]
	Example 1 (English):
	Text: Hello, how are you?
	Token count (approx.): 6 tokens
	Analysis: Hello / , / how / are / you / ?

	Example 2 (Chinese):
	Text: 你好，你最近怎么样？ (Hello, how have you been recently?)
	Token count (approx.): 9~14 tokens
	Analysis: Each Chinese character and punctuation mark takes 1~2 tokens

	Example 3 (Mixed):
	Text: 我使用 GPT-4 来处理 NLP 任务。 (I use GPT-4 to process NLP tasks.)
	Token count (approx.): 15~20 tokens
	Analysis: Chinese character token efficiency is lower than words composed of English letters
\end{lstlisting}

#### Practical Estimation Advice

In practical applications, the following empirical rules can be used for quick estimation of token counts:

- English: $Token count \approx Word count \times 1.3$
- Chinese: $Token count \approx Chinese character count \times 1.5$ (when using English-dominated general tokenizers)
- Chinese (Chinese-optimized models): $Token count \approx Chinese character count \times 1.0$

For exact calculations, it is recommended to directly use the official tokenization tools of the corresponding models, such as OpenAI's online Tokenizer tool (<https://platform.openai.com/tokenizer>) or each model's `AutoTokenizer` on Hugging Face.

</details>

**86. How Do Existing LLMs Achieve Flexible Infilling Text Generation?**

<details>
<summary>💡 Show Answer</summary>

Fill-in-the-Middle (FIM) is a technique that enables language models to dynamically generate missing content at any location within text, breaking through the limitation of traditional autoregressive models that can only generate from left to right.

The core idea is **bidirectional context modeling** and **dynamic insertion generation**. Input text is divided into prefix, suffix, and middle parts using special tokens (such as `<PRE>`, `<SUF>`, `<MID>`), and the model predicts the middle content conditioned on both prefix and suffix.

Application cases:
- **Code completion** (e.g., GitHub Copilot)
- **Instruction generation** (Reverse Mode Generation)
- **Text editing** (sentence rewriting)

</details>

**87. What is Tokenization?**

<details>
<summary>💡 Show Answer</summary>

Tokenization is a fundamental step in Natural Language Processing (NLP), referring to the process of breaking raw text down into smaller semantic or structural units (called "tokens"). These units can be words, subwords, characters, or even meaningful phrases or punctuation marks.

**Common Tokenization Granularities**

Depending on the application scenario, tokenization can be performed at different granularities:
- **Word-level**: Segments based on spaces or punctuation boundaries, e.g., “I love NLP” $\rightarrow$ [“I”, “love”, “NLP”]. Simple and intuitive, but results in a large vocabulary size and cannot handle Out-of-Vocabulary (OOV) words.
- **Subword-level**: Splits words into smaller meaningful fragments, which is the mainstream approach in modern large models. Common algorithms include BPE (Byte Pair Encoding), WordPiece, and SentencePiece.
- **Character-level**: Uses individual characters as tokens, with the smallest vocabulary size, but sequences are longer, making it difficult for models to capture long-distance dependencies.
- **Byte-level**: Uses UTF-8 bytes as units; models like GPT-2/GPT-4 adopt this scheme, which can losslessly represent text in any language.

**Why is Tokenization Important?**
- **Numerical representation**: Language models process text as numerical sequences where each token corresponds to a unique index in the vocabulary, used by the model to look up the corresponding embedding vector.
- **Computational efficiency**: By converting text into discrete symbol sequences over a fixed vocabulary, computational complexity is significantly reduced, enabling models to perform matrix operations efficiently.
- **Handling Out-of-Vocabulary (OOV) words**: Handled by breaking rare or new words into subwords, e.g., “unbelievable” $\rightarrow$ [“un”, “believ”, “able”], thereby avoiding the issue of large numbers of `<UNK>` tokens in pure word-level tokenization.
- **Multilingual flexibility**: Subword and byte-level tokenization allow sharing vocabularies across languages without maintaining separate vocabularies for each language in multilingual tasks, while effectively controlling vocabulary size (typically 32K--100K).
- **Impact on context length**: For the same text content, finer tokenization granularity yields longer sequences, consuming more of the context window; thus, tokenization efficiency directly affects the amount of effective information a model can process.

%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%%

% -------------------------------------------------------

</details>

**88. Are the Token Embedding and Output Embedding in LLMs the Same?**

<details>
<summary>💡 Show Answer</summary>

No. Judging from the source code of LLaMA, they are not the same thing. The input uses `VocabParallelEmbedding`, while the output uses `ColumnParallelLinear`, making them independent parameters.

**Detailed Explanation**

In the Transformer architecture, two linear mappings related to the vocabulary are involved:

- **Input side (Token Embedding)**: Maps input token indices to dense vectors, resulting in an embedding matrix $\mathbf{E}$ of shape $[V, d]$, where $V$ is the vocabulary size and $d$ is the hidden dimension. In LLaMA, this is implemented by `VocabParallelEmbedding`, whose weights are partitioned across multiple GPUs along the vocabulary dimension.
- **Output side (LM Head)**: Maps the hidden state $\mathbf{h} \in \mathbb{R}^d$ from the final Transformer layer back into vocabulary space to obtain logits $\mathbf{z} = \mathbf{h} \mathbf{W}^\top \in \mathbb{R}^V$, which are then passed through softmax to obtain the probability distribution for the next token. In LLaMA, this is implemented by `ColumnParallelLinear`, whose weights are partitioned along the output dimension (vocabulary dimension).

**Concept of Weight Tying**

Although both are independent parameters in LLaMA, some earlier models (such as the original Transformer and GPT-2) **tied weights** between the input embedding matrix $\mathbf{E}$ and the output projection matrix $\mathbf{W}$ (Weight Tying), setting $\mathbf{W} = \mathbf{E}$. The theoretical basis for this lies in:
- Input embeddings capture semantic similarities between tokens, and output projections should also utilize the same semantic space to compute token relevance scores; their objectives are aligned.
- Weight tying reduces parameters by approximately $V \times d$, saving significant GPU memory when the vocabulary is large (e.g., $V=50257$, $d=768$).
- Experiments show that weight tying usually does not sacrifice performance in small-to-medium scale models, and even has a regularization effect.

**Reasons Why LLaMA Chooses Not to Tie Weights**

The main reasons LLaMA adopts independent parameters are:
- **Increased model scale**: At the multi-billion parameter scale, input embeddings and output projections serve different roles; decoupling helps the model learn their respective representations more thoroughly.
- **Tensor Parallelism requirements**: LLaMA is deeply optimized for multi-GPU distributed training. Input embeddings are split row-wise (`VocabParallelEmbedding`), while output projections are split column-wise (`ColumnParallelLinear`). Their parallelization strategies differ, so independent parameters facilitate optimizing communication patterns separately.
- **Vocabulary expansion**: The LLaMA series expanded its vocabulary across different versions (e.g., adding code and multilingual tokens); independent parameters provide greater flexibility during fine-tuning.

**LLaMA Source Code Correspondence (Simplified Illustration)**

\begin{lstlisting}[style=promptstyle]
	# Input side: Token Embedding
	self.embed_tokens = VocabParallelEmbedding(
	config.vocab_size,
	config.hidden_size,
	...
	)

	# Output side: LM Head (linear layer, without bias)
	self.lm_head = ColumnParallelLinear(
	config.hidden_size,
	config.vocab_size,
	bias=False,
	...
	)

	# Forward pass
	hidden_states = self.embed_tokens(input_ids)   # [B, T] -> [B, T, d]
	# ... Transformer layers ...
	logits = self.lm_head(hidden_states)            # [B, T, d] -> [B, T, V]
\end{lstlisting}

As can be seen, `embed_tokens.weight` and `lm_head.weight` are two independent tensors that are not shared, which differs from the practice in earlier models such as GPT-2 that explicitly executed `lm_head.weight = embed_tokens.weight`.

</details>

**89. What Are the Tokenization Methods in Large Models?**

<details>
<summary>💡 Show Answer</summary>

**Introduction to Mainstream Tokenization Algorithms**

- **BPE (Byte Pair Encoding)**: Starting from the character level, iteratively merges the most frequently occurring adjacent symbol pairs in the corpus until the vocabulary reaches the preset size. Widely adopted by the GPT series models.
- **WordPiece**: Similar to BPE, but its merge strategy is based on maximizing the likelihood of the language model rather than frequency. Adopted by BERT.
- **SentencePiece**: A language-independent tokenization framework that treats spaces as regular characters, supporting both BPE and Unigram sub-algorithms. Adopted by models such as LLaMA and T5.
- **Unigram Language Model**: Starts with a large initial vocabulary and iteratively removes low-contribution tokens by minimizing the overall loss, obtaining a compact vocabulary.

**Example: BPE Tokenization Process**

Below is a simplified example of BPE iteration, with the initial corpus being “low lower lowest”:

\begin{lstlisting}[style=promptstyle]
	Initial vocabulary (character-level):
	l o w </w>  ->  "low"
	l o w e r </w>  ->  "lower"
	l o w e s t </w>  ->  "lowest"

	1st merge: The symbol pair with highest frequency is (l, o), merged into "lo"
	2nd merge: The symbol pair with highest frequency is (lo, w), merged into "low"
	3rd merge: The symbol pair with highest frequency is (e, r), merged into "er"
	...

	Final tokenization result:
	"lower"  ->  ["low", "er", "</w>"]
	"newest" ->  ["new", "est", "</w>"]  (Unseen words can also be decomposed)
\end{lstlisting}

</details>

**90. Can You Explain the Workflow of the BPE Algorithm?**

<details>
<summary>💡 Show Answer</summary>

![Workflow and Subword Pair Merging Pipeline of the Byte-Pair Encoding (BPE) Algorithm](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig17_bpe_tokenization_flow.png)



BPE (Byte Pair Encoding) is a simple data compression algorithm in which the most frequently occurring adjacent byte pairs are replaced with a byte that does not occur in that data. It was originally described in the 1994 article "A New Algorithm for Data Compression".

Suppose we have data **aaabdaaabac** that needs to be encoded (compressed). The byte pair **aa** occurs most frequently, so we replace it with **Z**, since **Z** does not appear in our data. So we now have **ZabdZabac**, where **Z = aa**. The next common byte pair is **ab**, so let's replace it with **Y**. We now have **ZYdZYac**, where **Z = aa** and **Y = ab**. The only remaining byte pair is **ac**, which occurs only once, so we will not encode it. We can use recursive byte pair encoding to encode **ZY** into **X**. Our data has now become **XdXac**, where **X = ZY, Y = ab, Z = aa**. It can no longer be further compressed because no byte pair occurs more than once.

BPE ensures that the most common words are represented as single tokens in the vocabulary, while rare words are broken down into two or more subword tokens, consistent with what subword-based tokenization algorithms do.

</details>

**91. What is the Principle of FlashAttention? What Problem Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

![FlashAttention HBM and SRAM Tiling Memory Optimization Mechanism](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig18_flashattention_tiling_hbm_sram.png)



**Problem Background**: Standard Attention computation requires writing the entire attention matrix of size $N \times N$ for Q, K, V to HBM (high bandwidth memory). When sequence length $N$ is large, HBM memory I/O becomes the bottleneck, resulting in slow speed and high memory footprint.
**Core Idea of FlashAttention**:
- Utilizes GPU SRAM (on-chip high-speed cache) instead of HBM for intermediate calculations.
- Tiles Q, K, V and loads them block-by-block into SRAM, computing Softmax and weighted summation in tiles to avoid memory read/write of the full $N \times N$ matrix.
- Uses recomputation in backward pass to recompute attention weights on-the-fly rather than storing them, saving GPU memory.
**Effect**: Speedup of $2\times$ to $4\times$, GPU memory complexity reduced from $\mathcal{O}(N^2)$ to $\mathcal{O}(N)$, with numerical results mathematically equivalent to standard Attention.

</details>

**92. What is the principle of ALiBi position encoding? What are its similarities and differences compared to RoPE?**

<details>
<summary>💡 Show Answer</summary>

**Principle of ALiBi (Attention with Linear Biases)**:
Instead of adding positional information to input embeddings, ALiBi directly adds a negative bias proportional to the relative distance to the attention score matrix during Attention computation: the further the distance, the more negative the bias, and the lower the attention weight. Each attention head uses a different slope coefficient.

**Comparison with RoPE**:
\begin{table}[h]



| **Dimension** | **ALiBi** | **RoPE** |
| --- | --- | --- |
| Implementation | Adds linear bias to attention scores | Applies rotary transformation to Q/K vectors |
| Extrapolation Ability | Strong, directly extendable to longer sequences | Requires additional techniques (e.g., YaRN) for extrapolation |
| Training Overhead | No extra parameters, lightweight | No extra parameters, lightweight |
| Performance | Excellent long-sequence extrapolation, slightly weaker than RoPE on short sequences | Superior short-sequence performance |
| Representative Models | BLOOM | LLaMA, Qwen, Mistral |


\end{table}

</details>

**93. What problems exist in traditional Attention?**

<details>
<summary>💡 Show Answer</summary>

Traditional Scaled Dot-Product Attention exposes several fundamental bottlenecks when processing long sequences, mainly manifested in the following aspects:

1. **Context length constraints and quadratic complexity**: The computational complexity has a quadratic relationship $O(L^2)$ with the sequence length $L$. For a sequence of length $L$, the attention mechanism needs to compute pairwise correlations between all tokens, producing an $L \times L$ attention matrix. When $L$ grows from 512 to 8192, the compute load increases by approximately 256 times. Under constrained hardware conditions, this prevents models from efficiently handling scenarios such as long documents, long conversations, or long code, significantly increasing both training and inference time overheads.

2. **Difficulty in capturing long-distance dependencies**: In long sequences, attention weights are dispersed across a large number of tokens after softmax normalization, resulting in extremely small weights assigned to any individual distant token. This dilution effect makes it difficult for the model to establish effective semantic connections between tokens spanning hundreds or even thousands of positions; the information propagation path degrades with distance, constraining the ability to capture long-range dependencies.

3. **High memory footprint**: Standard attention implementations need to explicitly store the complete $L \times L$ attention matrix during the forward pass for use in backpropagation. Taking $L=4096$, `float32` precision, and 32 attention heads as an example, the attention matrix alone requires approximately $4096^2 \times 32 \times 4~bytes \approx 2~GB$ of VRAM, severely restricting the choice of batch size and sequence length.

4. **KV Cache overhead during inference**: During the autoregressive generation stage, to avoid redundant computations, Key and Value vectors of all historical tokens across every layer need to be cached (i.e., KV Cache). Its size grows linearly with sequence length:
    \[
    KV Cache Size = 2 \times L \times n_{heads} \times d_{head} \times n_{layers} \times Bytes per Precision
    \]
    For large models (such as at the 70B parameter scale), the KV Cache under long sequences can easily consume dozens of GB of VRAM, becoming the main bottleneck for inference throughput.

5. **Generalization limitations of position encodings**: Traditional absolute position encodings (such as sinusoidal encodings or learnable position embeddings) fix the maximum sequence length during training. If the model encounters sequences exceeding the training length during inference, performance drops sharply, lacking the capability for sequence length extrapolation.

</details>

**94. Solutions to Traditional Attention Issues**

<details>
<summary>💡 Show Answer</summary>

To address the aforementioned issues, academia and industry have proposed improvements across several directions:

1. **Sparse Attention Mechanism**: Rather than having each token attend to all other tokens in the sequence, attention scope is restricted through preset sparse patterns, reducing complexity from $O(L^2)$ to $O(L \sqrt{L})$ or $O(L \log L)$. Common patterns include:
    - **Local Window Attention**: Each token only attends to tokens within a fixed window around it, such as the sliding window mechanism adopted by Longformer.
    - **Strided Attention**: Samples distant tokens at fixed strides, combining with local attention to achieve global information propagation, as in Sparse Transformer.
    - **Global Tokens**: Introduces a small number of global tokens (such as `[CLS]`), which interact bidirectionally with all positions and act as information aggregation nodes.

2. **Chunked / Block Attention**: Divides the sequence into several non-overlapping or partially overlapping blocks. Full attention is executed within blocks, while information is passed between blocks via summary vectors or cross-block attention. This approach is simple to implement and easy to integrate into existing systems; the segment-level recurrence mechanism in Transformer-XL represents an extension of this concept.

3. **Linear Attention**: Rewrites softmax attention into linear form using kernel function approximations, leveraging the associativity of matrix multiplication to lower complexity to $O(L)$:
    \[
    Attention(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \frac{\phi(\mathbf{Q})(\phi(\mathbf{K})^\top \mathbf{V})}{\phi(\mathbf{Q})\phi(\mathbf{K})^\top \mathbf{1}}
    \]
    where $\phi(\cdot)$ is a feature mapping function (e.g., ELU+1). Representative works include Linear Transformer, Performer (using random features to approximate softmax), etc. The drawback of linear attention is that approximation errors lead to slight performance degradation, and in practice, speedup gains vary depending on implementation.

4. **Multi-scale / Hierarchical Attention**: Combines local fine-grained attention with global coarse-grained attention to capture sequence information across different resolutions. A typical approach is to first compute fine-grained attention within local windows, and then execute global attention on window summaries, achieving a global receptive field at low cost, such as the hybrid attention designs in Longformer and BigBird.

5. **FlashAttention**: Redesigns the attention calculation workflow from an IO-aware perspective. By employing tiling and recomputation, it avoids writing the full $L \times L$ attention matrix to High Bandwidth Memory (HBM/VRAM), reducing memory complexity from $O(L^2)$ to $O(L)$ while maintaining exact mathematical equivalence to standard attention. It achieves 2--4$\times$ training speedup on actual hardware and has become the standard implementation for training modern large language models.

6. **Relative Position Encodings and Length Extrapolation**: Adopts relative position encoding schemes such as RoPE (Rotary Position Embedding) and ALiBi, making the positional bias in attention scores depend solely on relative distance between tokens. This improves the model's extrapolation capability for ultra-long sequences during inference, alleviating the length generalization issues of traditional absolute position encodings.

</details>

**95. What Are the Variants of Attention?**

<details>
<summary>💡 Show Answer</summary>

Variants of the attention mechanism can be roughly categorized into the following types based on their improvement goals:

- **Sparse Attention**: Lowers complexity by restricting each token's attention range. Representative works include Sparse Transformer, Longformer, BigBird, Sliding Window Attention (adopted by Mistral), etc.

- **Linearized Attention**: Reduces quadratic complexity to linear using kernel approximations. Representative works include Linear Transformer, Performer, cosFormer, RetNet (using linear recurrent formulation with decay mechanism), etc.

- **Prototype & Memory Compression**: Compresses Key/Value into a small set of prototype vectors. Representative works include Set Transformer (using Inducing Points), Perceiver, Longformer's global token mechanism, etc.

- **Low-Rank Self-Attention**: Assumes the attention matrix exhibits low-rank structure, using matrix decomposition or random projection to reduce computation. Representative works include Linformer (projecting K and V to lower dimensions, compressing the $L \times L$ matrix to $L \times k$), etc.

- **Prior-Guided Attention**: Incorporates structural priors (such as locality bias or positional decay) into attention weights. Representative works include ALiBi (adding a linear position penalty term to attention logits), T5's relative position bias, etc.

- **Improved Multi-Head Mechanisms**: Enhances standard Multi-Head Attention (MHA) by modifying head count, KV dimensions, or inter-head interaction. Representative works include Multi-Query Attention (MQA, all heads share the same KV set), Grouped-Query Attention (GQA, heads are grouped to share KV, adopted by LLaMA 2/3), Multi-Head Latent Attention (MLA, DeepSeek's low-rank KV compression scheme), etc.

- **IO-Aware Implementation Optimization**: Redesigns the computation graph without altering mathematical equivalence. Representative work is the FlashAttention series (v1/v2/v3), which circumvents HBM bottlenecks through tiling, serving as the most widely adopted attention implementation in industry today.

</details>

**96. What Problems Exist in Multi-Head Attention?**

<details>
<summary>💡 Show Answer</summary>

Multi-Head Attention (MHA) maintains independent Query, Key, and Value projections for each attention head, which offers expressiveness advantages but also imposes significant engineering costs, particularly during inference:

- **Training phase**: MHA has a relatively limited impact on training speed. Because full-sequence parallel computation is used during training, the attention matrix can be computed all at once. The parameter increase brought by multiple heads (compared to subsequent variants) can be mitigated via batching and tensor parallelism. However, when there are too many heads, the dimension per head $d_{head} = d_{model} / n_{heads}$ decreases accordingly, weakening single-head expressiveness and potentially leading to minor performance losses on certain tasks.

- **Inference phase (KV Cache bottleneck)**: MHA is the core source of performance issues during inference. During autoregressive decoding, Key and Value vectors of past tokens must be independently cached for each head at every layer. The total size of KV Cache is:
    \[
    KV Cache = 2 \times n_{layers} \times n_{heads} \times d_{head} \times L_{seq} \times bytes
    \]
    Taking LLaMA-2 70B (80 layers, 64 heads, $d_{head}=128$, `float16`) and sequence length 4096 as an example:
    \[
    2 \times 80 \times 64 \times 128 \times 4096 \times 2~bytes \approx 10.7~GB
    \]
    This means KV Cache alone consumes nearly 13% of an A100 (80GB) GPU's VRAM, severely limiting concurrent requests (batch size) and maximum supported sequence length.

- **Memory bandwidth bottleneck**: During inference, generating each token requires reading the entire KV Cache from HBM. Since the volume of data read far exceeds actual computation, inference is a classic **memory-bound** operation rather than compute-bound. This results in very low GPU compute utilization, restricted batch sizes, and difficulty in increasing inference throughput.

- **Directions for Improvement**: To address the above issues, the industry has proposed the following alternatives:
    - **Multi-Query Attention (MQA)**: All query heads share a single set of Key and Value, shrinking KV Cache to $1/n_{heads}$ of its original size. Inference speed is significantly improved, though with potential minor accuracy loss (adopted by Falcon).
    - **Grouped-Query Attention (GQA)**: Groups query heads into $G$ groups, with each group sharing a pair of KV vectors. It strikes a balance between MHA and MQA and is currently the primary choice for mainstream LLMs (adopted by LLaMA-2/3, Mistral, and Qwen).
    - **Multi-Head Latent Attention (MLA)**: Proposed by DeepSeek-V2/V3, it compresses KV vectors via low-rank joint compression, reducing KV Cache to extremely low dimensions (e.g., caching only a 512-dimensional latent vector instead of full KV). This preserves expressiveness close to MHA while dramatically reducing VRAM footprint.

</details>

**97. Explain the Origin of Lightning Attention in MiniMax**

<details>
<summary>💡 Show Answer</summary>

1. **Background**
    - **Traditional Attention** (Softmax Attention) has $O(n^2)$ complexity, making it difficult to process long sequences.
    - **Linear Attention** replaces Softmax with other activation functions (such as Swish), decomposing computation into:

        $$
        (\phi(Q)\phi(K)^T)V = \phi(Q)(\phi(K)^TV)
        $$

        Complexity drops to $O(nd^2)$, achieving linear growth.

2. **Issues with Causal Models**
    - **Unidirectional attention constraint**: Generative models (e.g., GPT) must use a mask matrix $M$, preventing direct right-multiplication optimization: $O = (QK^T \odot M)V$
    - **Accumulation summation bottleneck**: Historical KV must be computed step-by-step (e.g., $o_t = \sum_{j=1}^t q_i^T k_j v_j$), preventing parallelization.

3. **Evolution of Lightning Attention**

    **Lightning Attention-1** (From paper: TransNormerLLM: A Faster and Better Large Language Model with Improved TransNormer)
    - **Method**: Removes Softmax, adopts Swish activation function, and incorporates tiling computation from Flash Attention (SRAM optimization).
    - **Limitation**: Complexity remains $O(n^2)$, failing to resolve the fundamental bottleneck of causal models.

    **Lightning Attention-2** (MiniMax's work: MiniMax-01: Scaling Foundation Models with Lightning Attention)
    - **Core Innovation**: Tiled sequence processing, split into intra-block and inter-block attention:
        - **Intra-block**: Traditional left-multiplication + mask, maintaining causality.
        - **Inter-block**: Caches historical KV and uses right-multiplication, achieving $O(nd^2)$ complexity.
    - **Formula Example**:

        $$
        O_i = O_{intra} + O_{inter} = [(Q_i K_i^T) \odot M]V_i + \Lambda Q_i (KV)
        $$

        where $\Lambda$ is a diagonal positional encoding matrix.

4. **Key Algorithm Workflow**

    Forward Pass:
    1. Load Q/K/V into SRAM in tiles.
    2. Compute intra-block attention (masked) and inter-block attention (using cached KV).
    3. Update KV cache and output results.

    Backward Pass:
    1. When computing gradients in tiles, process intra-block and inter-block contributions separately.
    2. Efficiently propagate gradients via caching mechanisms.

5. **Summary**
    - **Lightning Attention-2** achieves linear complexity attention for causal models for the first time, supporting ultra-long sequence training.
    - **Core Contribution**: Tiling strategy + caching mechanism, balancing computational efficiency and model performance.

</details>

**98. How Is the Load Balancing Problem Solved in MoE?**

<details>
<summary>💡 Show Answer</summary>

**Problem**: In MoE, the Router may route a large number of tokens to a small number of experts, resulting in unbalanced expert load where some experts are overloaded while others remain idle, leading to training instability.
**Common Solutions**:
- **Auxiliary Loss**: Adds a load balancing loss term to the training loss, encouraging experts to be selected uniformly (e.g., GShard, Switch Transformer).
- **Expert Capacity**: Sets a token processing capacity limit for each expert; tokens exceeding the limit are dropped or passed to the next expert.
- **Random Routing Noise**: Adds a small amount of noise to routing scores during training to prevent the model from prematurely locking into fixed experts.
- **DeepSeek-V2 Scheme**: Introduces Shared Experts to handle common knowledge across all tokens, while routed experts only process specialized content, alleviating single-expert overload.

</details>

**99. What Is the Difference Between Layer Normalization and Batch Normalization? Why Is LayerNorm Used in LLMs?**

<details>
<summary>💡 Show Answer</summary>

**BatchNorm**: Normalizes each feature across the batch dimension, relying on statistics (mean and variance) across multiple samples within the batch.
**LayerNorm**: Normalizes across feature dimensions within a single sample, independent of other samples in the batch.
**Reasons LLMs Choose LayerNorm**:
- LLMs process variable-length sequences, and batch sizes are often small or even 1, making BatchNorm statistics unstable.
- During autoregressive inference, only one token is generated at a time, making it impossible to compute cross-sample batch statistics.
- LayerNorm places no requirements on sequence length, maintains stable computation, and is better suited for NLP scenarios.

</details>

**100. What Is the Difference Between Pre-Norm and Post-Norm? What Are Their Respective Pros and Cons?**

<details>
<summary>💡 Show Answer</summary>

![Architecture Comparison between Pre-Norm and Post-Norm Residual Normalization](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig19_prenorm_vs_postnorm.png)



**Post-Norm** (Original Transformer): Performs residual connection first, then LayerNorm: `output = LayerNorm(x + sublayer(x))`
**Pre-Norm** (Mainstream modern LLMs): Performs LayerNorm first, then residual connection: `output = x + sublayer(LayerNorm(x))`
\begin{table}[h]



| **Dimension** | **Post-Norm** | **Pre-Norm** |
| --- | --- | --- |
| Training Stability | Poor, gradients prone to vanishing/exploding in deep networks | Good, more stable gradient flow |
| Model Performance | Slightly better performance after convergence | Slightly weaker than Post-Norm (theoretically) |
| Applicable Scenarios | Shallow networks | Deep large models (adopted by LLaMA, GPT, etc.) |


\end{table}
Mainstream LLMs almost exclusively use Pre-Norm because its training stability is far more critical.

</details>

**101. Formula of Layer Norm**

<details>
<summary>💡 Show Answer</summary>

$$
    LayerNorm(x) = \gamma \cdot \frac{x - \mu}{\sqrt{\sigma^2 + \epsilon}} + \beta
$$

where $\mu$ and $\sigma^2$ are the mean and variance computed along the feature dimension, respectively, and $\gamma$ and $\beta$ are learnable scale and shift parameters.

</details>

**102. Formula of RMS Norm**

<details>
<summary>💡 Show Answer</summary>

$$
    RMSNorm(x) = \gamma \cdot \frac{x}{\sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2 + \epsilon}}
$$

</details>

**103. What Are the Advantages of RMS Norm Compared to Layer Norm?**

<details>
<summary>💡 Show Answer</summary>

The basic form of **Layer Norm** is as follows:

For an input vector $\mathbf{x} \in \mathbb{R}^d$, Layer Norm computes:
\[
\mu = \frac{1}{d}\sum_{i=1}^{d} x_i, \quad \sigma^2 = \frac{1}{d}\sum_{i=1}^{d}(x_i - \mu)^2
\]
\[
\hat{x}_i = \frac{x_i - \mu}{\sqrt{\sigma^2 + \epsilon}}, \quad LN(\mathbf{x}) = \gamma \odot \hat{\mathbf{x}} + \beta
\]
where $\gamma, \beta \in \mathbb{R}^d$ are learnable scale and shift parameters.

The basic form of **RMS Norm** is as follows:

RMS Norm simplifies Layer Norm by removing the mean-centering step and retaining only the Root Mean Square normalization:
\[
RMS(\mathbf{x}) = \sqrt{\frac{1}{d}\sum_{i=1}^{d} x_i^2 + \epsilon}
\]
\[
RMSNorm(\mathbf{x}) = \frac{\mathbf{x}}{RMS(\mathbf{x})} \odot \gamma
\]
where $\gamma \in \mathbb{R}^d$ is a learnable scale parameter, omitting the shift parameter $\beta$ from Layer Norm.

The **advantages of RMS Norm compared to Layer Norm** are mainly reflected in the following aspects:

1. **Higher computational efficiency**: Layer Norm requires computing two statistics, mean $\mu$ and variance $\sigma^2$, whereas RMS Norm only computes a single statistic, the root mean square. It eliminates mean calculation and centering operations, reducing statistical computation overhead by about half, which yields noticeable training and inference speedups in large-scale models.

2. **Fewer parameters**: RMS Norm removes the shift parameter $\beta$, cutting the parameter count in half (retaining only $\gamma$), saving parameter storage in ultra-large models.

3. **Comparable performance or slight improvements**: Experiments show that removing the mean-centering operation has very little impact on model performance. The RMS Norm paper (Zhang & Sennrich, 2019) demonstrated across multiple NLP tasks that RMS Norm achieves performance comparable to Layer Norm, and slightly improves results in certain scenarios due to enhanced training stability.

4. **Cleaner implementation**: RMS Norm has a cleaner implementation, making it easier to perform efficient Kernel Fusion optimizations on custom hardware (such as TPUs and NPUs).

**Intuitive Understanding**: Layer Norm was originally designed to simultaneously eliminate both mean shift (re-centering) and variance shift (re-scaling) of activation values. However, research revealed that the re-centering operation contributes minimally to final performance, whereas re-scaling is the core driver of normalization efficacy. Thus, RMS Norm retains good performance even after removing re-centering.

</details>

**104. Why is Post-Norm Theoretically Better Than Pre-Norm?**

<details>
<summary>💡 Show Answer</summary>

First, let us review the structural differences between **Pre-Norm** and **Post-Norm**:

- **Post-Norm** (Original Transformer architecture):
    \[
    \mathbf{x}_{l+1} = Norm(\mathbf{x}_l + F_l(\mathbf{x}_l))
    \]
- **Pre-Norm** (Commonly used architecture in modern LLMs):
    \[
    \mathbf{x}_{l+1} = \mathbf{x}_l + F_l(Norm(\mathbf{x}_l))
    \]

where $F_l(\cdot)$ denotes the sub-module of the $l$-th layer (such as Self-Attention or FFN).

**The Problem with Pre-Norm: Degeneration into a "Shallow and Wide" Model**

In the Pre-Norm architecture, the identity branch of the residual connection does not undergo any normalization. Therefore, as the network deepens, the proportion of the identity mapping weight in gradient propagation always remains large. Specifically:

- In the Pre-Norm architecture, $\mathbf{x}_{l+1} = \mathbf{x}_l + F_l(Norm(\mathbf{x}_l))$, and the gradient of the output with respect to the input is:
    \[
    \frac{\partial \mathbf{x}_{l+1}}{\partial \mathbf{x}_l} = \mathbf{I} + \frac{\partial F_l(Norm(\mathbf{x}_l))}{\partial \mathbf{x}_l}
    \]
    The identity matrix $\mathbf{I}$ is always present, allowing the gradient flow to directly bypass the residual branch and "skip" multiple layers, which causes the model to rely excessively on shallow direct paths.
- As the number of layers $L$ increases, the relative gradient contribution of the bottom layers decreases, and the network tends to degenerate into a model with a shallower effective depth, failing to fully leverage the expressiveness of deep networks.

**The Advantage of Post-Norm: "Full Depth" Expressiveness**

Post-Norm performs normalization at the output of each sub-layer:
\[
\mathbf{x}_{l+1} = Norm(\mathbf{x}_l + F_l(\mathbf{x}_l))
\]

- The Norm operation normalizes the sum of outputs from the identity branch and the residual branch. Every time it passes through Norm, the relative weight of the identity branch is scaled down, thereby **forcing the residual branch** (i.e., the non-linear transformation $F_l$ at each layer) to play a larger role.
- This means that every layer in Post-Norm is "fully utilized". The deeper the model, the closer its effective expressive depth is to its actual number of layers, ultimately achieving a higher performance upper bound.

**Trade-off in Training Difficulty**

- **Pre-Norm**: Gradients can flow back unimpeded through the identity branch, leading to stable gradients during early training and easy convergence. It is suitable for directly training extremely deep networks and is currently the default choice for most Large Language Models (such as LLaMA, Qwen, etc.).
- **Post-Norm**: Gradients must pass through Norm layers, making it prone to vanishing or exploding gradients in early training. It requires delicate learning rate scheduling (such as Warmup) and initialization strategies (such as T-Fixup, DeepNorm, etc.) for stable training, but once converged, its performance is typically superior to a Pre-Norm model with the same parameter count.

**Summary**: Under the same parameter configuration, Pre-Norm struggles to reach the performance upper bound of Post-Norm, but it is simple and stable to train; Post-Norm can theoretically reach a better performance upper bound, but requires careful monitoring of gradients during training and suffers from difficult convergence, necessitating specialized initialization or normalization strategies (such as DeepNorm).

</details>

**105. What is the Idea Behind DeepNorm?**

<details>
<summary>💡 Show Answer</summary>

**Background and Motivation**

Although Post-Norm performs well, it easily suffers from exploding/vanishing gradients when training extremely deep Transformers (e.g., over 100 layers), making stable convergence difficult. Although Pre-Norm is stable, it suffers from a lower performance upper bound as mentioned earlier. DeepNorm (proposed by Microsoft Research, ICML 2023) aims to combine the strengths of both: **possessing both the training stability of Pre-Norm and the high performance upper bound of Post-Norm**.

**Core Approach of DeepNorm**

DeepNorm makes two key modifications to the standard Post-Norm architecture:

1. **Up-scaling of the Residual Connection ($\alpha > 1$)**: Before performing Layer Norm, the identity branch of the residual connection is multiplied by a scaling factor $\alpha > 1$:
    \[
    \mathbf{x}_{l+1} = Norm(\alpha \cdot \mathbf{x}_l + F_l(\mathbf{x}_l))
    \]
    By scaling up the identity branch, the input to the Norm layer is primarily dominated by $\alpha \cdot \mathbf{x}_l$ in the early stage of training, thereby stabilizing gradient propagation and avoiding exploding gradients.

2. **Down-scaling of Model Parameter Initialization ($\beta < 1$)**: During the initialization phase, the parameters of each sub-layer in the model (such as the output projection matrices of FFN and Attention) are scaled down by a factor $\beta < 1$, ensuring that the magnitude of the residual branch output $F_l(\mathbf{x}_l)$ remains small during early training, further stabilizing updates in early training.

**Theoretical Derivation of $\alpha$ and $\beta$**

By analyzing the upper bound of the model update, the DeepNorm paper provides theoretical recommendations for values of $\alpha$ and $\beta$ with respect to the network depth $N$ (number of Encoder layers) and $M$ (number of Decoder layers). For example, for an Encoder-only network:
\[
\alpha = (2N)^{1/4}, \quad \beta = (8N)^{-1/4}
\]
For an Encoder-Decoder architecture, the values of $\alpha$ and $\beta$ depend on both $N$ and $M$. These values guarantee through theoretical analysis that the model update at each layer during training (relative to initialization) is bounded within a constant range, thereby achieving stable training.

**Advantages of DeepNorm**

- **Training Stability**: Through $\alpha$-scale and $\beta$-init, DeepNorm can stably train a **thousand-layer** (1,000 layers) Transformer, whereas standard Post-Norm may suffer from training instability at just tens of layers.
- **Performance Close to Post-Norm**: Since DeepNorm is essentially a variant of Post-Norm, its performance upper bound is close to standard Post-Norm and superior to Pre-Norm.
- **Simple Implementation**: DeepNorm only requires modifying the residual coefficient and initialization strategy without changing the model architecture, making it easy to implement in practice.

**Comparison with Other Methods**

\begin{center}


| **Method** | **Training Stability** | **Performance Bound** | **Scalable Depth** |
| --- | --- | --- | --- |
| Post-Norm | Poor | High | Tens of layers |
| Pre-Norm | Good | Medium | Hundreds of layers |
| DeepNorm | Good | High | Over 1,000 layers |


\end{center}

</details>

**106. Which Layer Normalization Methods Do Mainstream LLMs Use?**

<details>
<summary>💡 Show Answer</summary>

The following table summarizes the normalization methods adopted by mainstream Large Language Models:

\begin{center}


| **Model** | **Norm Position** | **Norm Type** | **Remarks** |
| --- | --- | --- | --- |
| LLaMA Series | Pre-Norm | RMSNorm | Used in LLaMA 1/2/3 |
| Qwen Series | Pre-Norm | RMSNorm | Used in Qwen1/2/2.5 |
| Mistral | Pre-Norm | RMSNorm | Architecture similar to LLaMA |
| GPT Series | Pre-Norm | LayerNorm | Changed to Pre-Norm starting from GPT-2 |
| BERT | Post-Norm | LayerNorm | Original Transformer architecture |
| Original Transformer | Post-Norm | LayerNorm | Vaswani et al., 2017 |


\end{center}

- **LLaMA** (Meta AI): Adopts **Pre-Norm** with **RMSNorm** as the normalization method.
    The LLaMA series (1/2/3) discards the mean-centering operation of original LayerNorm and uses RMSNorm to boost training speed; the Pre-Norm position ensures training stability in extremely deep networks.

- **Qwen** (Alibaba): Adopts **Pre-Norm** with **RMSNorm** as the normalization method.
    The Qwen series (1/1.5/2/2.5) is similar to LLaMA in overall architecture, opting for the combination of Pre-Norm + RMSNorm to balance training stability and computational efficiency.

- **Mistral**: Adopts **Pre-Norm** with **RMSNorm** as the normalization method.
    The Mistral architecture closely follows LLaMA, also utilizing Pre-Norm + RMSNorm.

- **GPT Series** (OpenAI): Adopts **Pre-Norm** with standard **LayerNorm** as the normalization method.
    The original GPT-1 paper used Post-Norm, but starting from GPT-2, it switched to Pre-Norm to stabilize large-scale training while retaining full LayerNorm (including mean and variance).

- **BERT** (Google): Adopts **Post-Norm** with standard **LayerNorm** as the normalization method.
    BERT follows the Post-Norm architecture of the original Transformer, executing LayerNorm after the output of each sub-layer (Self-Attention, FFN).

**Trend Analysis**: Currently, mainstream Large Language Models (especially generative Decoder-only models) universally adopt the combination of **Pre-Norm + RMSNorm** because:
1. Pre-Norm provides training stability, making it suitable for training ultra-large models with billions to hundreds of billions of parameters;
2. RMSNorm is computationally more efficient as it eliminates mean computation, saving considerable compute resources during large-scale training;
3. The combination of both has been extensively validated in engineering practice by LLaMA, Qwen, Mistral, etc., becoming the de facto industry standard.

</details>

**107. What is KV Cache? How Does It Accelerate Inference?**

<details>
<summary>💡 Show Answer</summary>

![KV Cache Mechanism in Prefill (Prompt) and Autoregressive Decoding Phases](https://img.pricenode.uk/llm_interview_questions/english/chapter1/fig20_kv_cache_prefill_decoding.png)



**Background: The Problem of Redundant Computation in Auto-Regressive Generation**

During the inference phase, LLMs generate text token by token in an auto-regressive manner: each step generates a new token, which is concatenated to the input sequence as the input for the next step. The standard Self-Attention formula is:
\[
Attention(Q, K, V) = softmax\!\left(\frac{QK^\top}{\sqrt{d_k}}\right)V
\]
where $Q, K, V \in \mathbb{R}^{N \times d_k}$, and $N$ is the current sequence length.

When generating at step $t$, the sequence length is $t$, requiring re-computing the $K$ and $V$ matrices for all $t$ tokens. However, step $t-1$ had already computed $K$ and $V$ for the first $t-1$ tokens; discarding this work results in massive **redundant computation**. If the total sequence length is $N$, the overall computational complexity of the generation process under a naive implementation is:
\[
\sum_{t=1}^{N} O(t \cdot d_k) = O(N^2 \cdot d_k)
\]

**Principle of KV Cache**

The core idea of KV Cache is: **cache the already computed $K$ and $V$ vectors of historical tokens in each Attention layer, and directly reuse them when generating new tokens to avoid redundant computation**.

The specific workflow is as follows:

1. **Prefill Phase (Pre-filling)**: Compute $K$ and $V$ for all tokens in the input prompt in parallel at once, and store the results into the KV Cache.
2. **Decode Phase (Step-by-step decoding)**: For each newly generated token $x_t$, compute only $Q_t$, $K_t$, and $V_t$ for this token; **append** $K_t$ and $V_t$ to the KV Cache; then perform Attention using $Q_t$ against all historical $K$ and $V$ in the Cache:
    \[
    Attention(Q_t,\ K_{1:t},\ V_{1:t}) = softmax\!\left(\frac{Q_t K_{1:t}^\top}{\sqrt{d_k}}\right)V_{1:t}
    \]

**Inference Acceleration Effect**

With KV Cache, each Decode step only needs to compute $Q$, $K$, and $V$ for the current new token (computational complexity $O(d_k)$), along with a vector-matrix multiplication of $Q_t$ and the Cache of length $t$ (computational complexity $O(t \cdot d_k)$). The overall calculation workflow becomes:
\[
\sum_{t=1}^{N} O(t \cdot d_k) \;\longrightarrow\; Incremental computation per step, avoiding redundant calculation for historical tokens
\]
That is, the **computational complexity per generated step drops from $O(N^2)$ to $O(N)$** (for single step), significantly speeding up the inference process. KV Cache has become a standard optimization for LLM inference, enabled by default in almost all mainstream inference frameworks (vLLM, TensorRT-LLM, HuggingFace TGI, etc.).

**Memory Overhead Analysis of KV Cache**

The cost of KV Cache is that GPU VRAM consumption grows linearly with sequence length. Assuming the model has $L$ layers, each layer's Attention hidden dimension is $d$, and the data type is FP16 (2 bytes per element), the VRAM required to cache a sequence of length $N$ is:
\[
KV Cache Size = 2 \times L \times N \times d \times 2  bytes
\]
(The factor 2 comes from storing both $K$ and $V$).

Taking LLaMA-2 70B as an example ($L=80$, $d=8192$, FP16), caching a sequence of length 4096 requires approximately:
\[
2 \times 80 \times 4096 \times 8192 \times 2 \approx 8.6  GB
\]
For long sequences or large batch inference, the VRAM pressure from KV Cache becomes very significant.

**KV Cache Compression Schemes**

To alleviate the memory pressure of KV Cache, academia and industry have proposed various compression schemes:

- **MQA (Multi-Query Attention)**: All Query heads share the same single set of $K$ and $V$, compressing KV Cache to $1/h$ ($h$ is the number of heads) at the cost of a slight loss in model accuracy. Models like Falcon and PaLM adopt this scheme.

- **GQA (Grouped-Query Attention)**: Divides the $h$ Query heads into $g$ groups, with each group sharing a pair of $K$ and $V$, compressing KV Cache to $g/h$ to strike a balance between MQA and MHA. LLaMA-2 70B, Qwen series, etc., adopt this scheme.

- **PagedAttention (vLLM)**: Borrowing the concept of virtual memory paging from operating systems, PagedAttention stores and dynamically allocates KV Cache in non-contiguous fixed-size "pages" (Blocks), avoiding memory fragmentation and significantly improving GPU VRAM utilization and concurrent throughput.

- **Sliding Window / Sparse Attention**: Caches only the most recent $W$ tokens' KV (such as Mistral's Sliding Window Attention), trading a limited context window for a fixed-size KV Cache.

- **KV Cache Quantization**: Quantizes KV Cache from FP16 to INT8 or INT4, directly reducing memory consumption by half or three-quarters, such as methods like KVQuant and KIVI.

</details>

