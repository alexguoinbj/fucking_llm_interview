# 13.3 Unsupervised Pre-training (TSDAE) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 13: Embedding Model Training · 27 questions

**1. What Is the Specific Training Mechanism of TSDAE? How Does It Learn Sentence Representations Through "Noise Addition and Reconstruction"?**

<details>
<summary>💡 Show Answer</summary>

TSDAE (Transformer-based Sequential Denoising Auto-Encoder) is an unsupervised sentence embedding training method. Its core training mechanism borrows the concept of denoising autoencoders but applies it at the sentence level:

- **Noise addition process**: For each sentence in the training corpus, a certain percentage of words is randomly deleted (or subjected to other forms of corruption such as word order shuffling) to produce a "noisy" corrupted version as the input to the encoder;
- **Encoding**: The noisy sentence is fed into an encoder (Transformer), and through pooling (the TSDAE paper demonstrates that using the output of a special token position yields better results, as discussed earlier), a fixed-length sentence vector is obtained. This vector is expected to carry all the information required to "reconstruct the original complete sentence";
- **Decoding/Reconstruction**: The encoded sentence vector is fed into a decoder (typically also a Transformer structure). The decoder's task is to reconstruct the **original, uncorrupted** complete sentence as accurately as possible. The training objective is the discrepancy between the reconstructed sentence and the original sentence (e.g., word-by-word cross-entropy loss);
- **Core assumption**: If the encoded vector successfully enables the decoder to recover the complete original sentence from corrupted inputs, it demonstrates that the vector effectively compresses and preserves the core semantic information of the original sentence. Consequently, this vector can be considered a high-quality sentence representation. This process requires zero human annotations (no similar sentence pairs or positive/negative pairs are needed), relying solely on large amounts of raw unannotated text for training.

In summary, TSDAE forces the encoder to compress a sentence's core semantics into a fixed-length vector representation through a self-supervised task: "deliberately corrupting the sentence and then requiring the model to reconstruct the original sentence relying solely on a single compressed vector." It is a sentence embedding pre-training method that relies entirely on unannotated data and is driven by reconstruction loss (rather than contrastive loss).

</details>

**2. Why Does TSDAE Choose Special Tokens Rather Than Mean Pooling for Sentence Representation?**

<details>
<summary>💡 Show Answer</summary>

TSDAE (Transformer-based Sequential Denoising Auto-Encoder) is an unsupervised sentence embedding pre-training method. Its core concept is to add noise to input sentences (e.g., randomly deleting words), allow the encoder to compress the noisy sentence into a fixed-length sentence representation, and then have the decoder reconstruct the original (un-corrupted) sentence based solely on this representation. The primary reasons for choosing **special tokens (such as the output at the [CLS] position or a specially added token) over mean pooling across all token outputs** include:

1. **Enforcing an Information Bottleneck**: TSDAE's training objective requires the decoder to reconstruct the entire sentence **relying solely on this single fixed vector**. If mean pooling were used, each token output from the encoder would still retain significant local information, and the averaging operation itself would not force the model to compress "global information" into a specific position. Conversely, when a special token's output is designated as the sole representation passed to the decoder, the encoder **must learn under training pressure to aggregate the entire sentence's semantic information into this single vector**, as it is the only channel through which the decoder receives information. This acts as a true "information bottleneck," forcing the vector to learn high-quality sentence-level semantic representations.

1. **Alignment with decoder architecture**: TSDAE typically adopts an Encoder-Decoder-like architecture. When generating (reconstructing) sentences, the decoder requires a fixed-length "context vector" as a conditioning signal (analogous to the final hidden state of an encoder in traditional Seq2Seq models). Using special token outputs naturally fulfills the role of this "context vector," corresponding directly to the decoder's input interface. Although vectors obtained via mean pooling are also fixed-length, they represent an "averaged mixture" of all token information mixed with local token-level noise, making them less "pure" as compressed representations compared to dedicated special tokens.

1. **Avoiding dilution of key information by mean pooling**: Mean pooling averages all tokens indiscriminately (including noisy words, padding tokens, and unimportant function words), which risks diluting genuinely critical semantic information in the sentence. By contrast, allowing the model to autonomously learn how to aggregate information into a special token enables it to selectively aggregate the most essential information via attention mechanisms.

In summary, TSDAE's selection of special tokens for sentence representations is fundamentally aimed at constructing a **strict information bottleneck**. Paired with the self-supervised objective of "reconstructing the original sentence based on this representation," it forces the model to learn high-quality compressed representations capable of summarizing whole-sentence semantics. This differs from contrastive learning settings where mean pooling is commonly used (where mean pooling achieves strong representations given supervised contrastive signals), reflecting the distinct training paradigms of the two approaches.

</details>

**3. What Is the Relationship Between TSDAE and Traditional Denoising Autoencoders? How Do Their Application Levels Differ?**

<details>
<summary>💡 Show Answer</summary>

Denoising Autoencoders (DAEs) represent a classic, long-standing self-supervised learning paradigm in deep learning. TSDAE can be viewed as a specific application of this classic idea tailored to Transformer architectures and sentence-level representation learning:

- **Concept of classic DAEs**: Originally proposed in image processing and early neural network research, the core idea is to artificially add noise (such as masking pixels or injecting Gaussian noise) to input data (e.g., image pixels, word vectors) and train an encoder-decoder structure to reconstruct clean original data from noisy inputs, thereby learning robust and meaningful low-dimensional representations of the data;
- **Specialization in TSDAE**: TSDAE applies this concept specifically to the **sentence** level (rather than pixels or individual words). The noise takes the form of deleting words from the sentence (rather than adding continuous noise), both encoder and decoder are based on Transformer architectures (rather than early MLPs or CNNs), and the framework is explicitly designed around the goal of "obtaining high-quality sentence-level embedding representations";
- **Difference from BERT Masked Language Modeling (MLM)**: Although MLM can also be interpreted as a "denoising" task (masking certain words and predicting them), MLM's reconstruction target is at the **word level** (independent predictions at each masked position), and during prediction, the model can still observe the unmasked tokens in the sentence via bidirectional attention, leading to multiple information leakage paths. In contrast, TSDAE requires the decoder to reconstruct the entire sentence **relying completely** on the **single sentence vector** compressed by the encoder without "peeking" at other contextual information from the encoder input during decoding. This stronger information bottleneck constraint is precisely the critical design allowing TSDAE to learn high-quality **sentence-level** (rather than word-level) representations.

In summary, TSDAE inherits the classic denoising autoencoder philosophy of "learning robust representations via reconstruction tasks." However, by framing the noise and reconstruction targets at the sentence level and forcing the decoder to reconstruct solely from a single compressed vector (unlike MLM, which leverages bidirectional contextual tokens), TSDAE produces representations that are naturally better suited as sentence-level semantic embeddings. This represents its core design distinction from word-level denoising tasks like MLM.

</details>

**4. Besides TSDAE, What Are Other Representative Unsupervised Sentence Embedding Pre-training Methods? (e.g., ICT, Contriever)**

<details>
<summary>💡 Show Answer</summary>

Unsupervised sentence/document embedding pre-training is an evolving research direction. Beyond TSDAE and SimCSE, several other representative methods are worth noting:

- **ICT (Inverse Cloze Task)**: The core idea is to randomly extract a sentence from a longer document/passage as a "pseudo-query" and treat the remaining text (after removing the sentence) as a "pseudo-document." These two form a natural positive pair (since the sentence originates from the document and is highly semantically related). Training contrastive learning models on these automatically constructed positive pairs eliminates the need for human annotation, serving as a primary data construction concept for subsequent works like Contriever;
- **Contriever**: Building upon automatically mined pseudo-positive pairs from ICT and combining techniques such as momentum contrast (Memory Bank/MoCo-style cross-batch negative queues), Contriever conducts large-scale unsupervised contrastive pre-training. It proves that relying entirely on automatically constructed unsupervised data can yield embedding models competitive across multiple retrieval benchmarks, providing crucial practical validation for the subsequent paradigm of "massive unsupervised pre-training + light fine-tuning";
- **Skip-thought Vectors**: An earlier (pre-Transformer era, RNN/LSTM-based) unsupervised sentence representation method. Inspired by Word2Vec's Skip-gram objective of "predicting context words given a target word," it extends this concept to the sentence level: given a sentence, the model is trained to predict its **preceding and following adjacent** sentences, thereby learning distributed sentence representations. While superseded by modern Transformer-based approaches, its core philosophy of "leveraging naturally occurring context adjacency in text to construct self-supervised signals" directly aligns with subsequent methods like ICT and TSDAE.

In summary, from early Skip-thought Vectors utilizing adjacent sentence relationships, to ICT utilizing sentence-to-document membership relationships, to TSDAE utilizing sentence noise-reconstruction relationships, the common thread among unsupervised sentence embedding pre-training methods is automatically constructing some form of "self-supervised signal" from unannotated text (whether adjacency, membership, or reconstruction). Works like Contriever further combine these automatically generated signals with large-scale contrastive learning, serving as vital reference points for understanding the evolutionary trajectory of unsupervised embedding pre-training.

</details>

**5. How Does ICT (Inverse Cloze Task) Automatically Mine Positive Pairs from Documents? How Does Its Positive Pair Construction Differ from TSDAE?**

<details>
<summary>💡 Show Answer</summary>

ICT's approach to positive pair construction differs fundamentally from TSDAE. Understanding this distinction helps in selecting the appropriate unsupervised pre-training method for specific scenarios:

- **Specific mining process in ICT**: Given a long document (containing multiple sentences/passages), a sentence is randomly selected as the query, and the remaining text after **removing** the sentence serves as the corresponding document (a small probability of retaining the sentence is sometimes kept so the model observes cases where "the query is inside the document," enhancing robustness). The (query, document) pair constitutes a positive sample, while other irrelevant document passages (or sentences extracted from other documents within the batch) serve as negative examples, trained using standard contrastive learning frameworks;
- **Core difference from TSDAE positive pair construction**: In TSDAE, the positive pair relationship is between the "complete version" and the "randomly word-deleted corrupted version" of the **same sentence** (essentially two different "views" of the exact same semantic content). In contrast, ICT's positive pair relationship represents a **membership/relevance** relationship between **a single sentence** and its **surrounding context document** (although the sentence and document are not identical in content, they share semantic relevance because the sentence originates from that document). This closely mirrors the "relevant yet non-identical" relationship between queries and documents in real retrieval scenarios. Consequently, data constructed by ICT is **distributionally closer to downstream retrieval tasks**, whereas TSDAE aligns more closely with **sentence-level semantic similarity** or paraphrase detection tasks;
- **Differences in applicable scenarios**: This explains why, in practice, if the downstream target task leans toward retrieval (where queries are significantly shorter than documents and their contents do not completely overlap), the pre-training data distribution of ICT and its derivative methods (such as Contriever) tends to be more suitable. Conversely, if the target task focuses on sentence-level semantic similarity comparison (where the two texts being compared are similar in length and characteristics), TSDAE's positive pair construction method is typically a better fit.

In summary, ICT automatically constructs positive pairs reflecting the "query-document relevant yet non-identical" relationship typical of real-world retrieval by "extracting a sentence from a document, removing it, and pairing them." This forms a sharp contrast with TSDAE's concept of constructing positive pairs from "the complete vs. corrupted versions of the same sentence." The two methods differ fundamentally in the **type of semantic relationship** leveraged during unsupervised data construction. In practice, the choice of unsupervised pre-training method should depend on whether the downstream task resembles retrieval or semantic similarity comparison.

</details>

**6. What Are the Core Differences Between SimCSE and TSDAE as Unsupervised Sentence Embedding Methods? Which Scenarios Are They Suited For?**

<details>
<summary>💡 Show Answer</summary>

SimCSE (which constructs positive pairs using dropout noise and trains via contrastive learning) and TSDAE (which constructs corrupted inputs via word deletion noise and trains via reconstruction loss) were introduced earlier. Although both fall under the "unsupervised" umbrella, they differ fundamentally in training mechanisms and data requirements:

- **Fundamental difference in training objectives**: SimCSE utilizes a **contrastive learning** loss (InfoNCE), centered on teaching the model to discriminate "which sample is positive vs. which are negative." TSDAE utilizes a **reconstruction** loss (similar to word-by-word prediction cross-entropy in autoencoders), focused on teaching the model "how to restore complete information from a compressed vector." The two differ completely in mathematical form and optimization targets;
- **Difference in positive sample construction**: SimCSE relies on the model's internal dropout randomness to create "two slightly noisy views of the same sentence," requiring no explicit textual modifications. TSDAE explicitly corrupts sentences via word deletion to create "complete-corrupted" pairs;
- **Difference in domain adaptability for training data**: As discussed earlier (in TSDAE's pros and cons), because TSDAE does not rely on the contrastive learning mechanism of "using other samples within the batch as negatives," it is theoretically less susceptible to the issue of "whether in-domain negatives are reliable" during cross-domain adaptation. Thus, it is better suited for handling **out-of-domain data** or scenarios requiring **domain adaptation**. SimCSE relies on in-batch negatives, which may suffer from domain-specific false negatives during domain transfer;
- **Selection criteria in practice**: If the target scenario aligns well with the pre-training data domain and primarily focuses on sentence-level semantic similarity tasks, SimCSE is usually simple to implement, fast to train, and yields reliable performance. If the target scenario requires adapting to a specialized vertical domain that deviates significantly from general pre-training corpora, TSDAE—by avoiding reliance on contrastive negative mechanisms and concentrating on sentence information compression—serves as a safer choice, particularly as a domain-adaptive pre-training step prior to supervised fine-tuning.

In summary, SimCSE and TSDAE represent two distinct technological routes in unsupervised sentence embedding learning: "contrastive learning" and "reconstruction learning." The former is simple to implement and computationally efficient, making it suitable for scenarios that closely match the pre-training corpus domain. The latter does not rely on negative pair mechanisms and exhibits greater robustness in domain adaptation scenarios. The two can also be combined (e.g., performing domain-adaptive pre-training with TSDAE first, followed by further fine-tuning with SimCSE or supervised contrastive learning).

</details>

**7. Compared with Supervised Methods, What Are the Advantages and Disadvantages of Unsupervised Pre-training Methods Like TSDAE When Handling Out-of-Domain Data or Performing Domain Adaptation?**

<details>
<summary>💡 Show Answer</summary>

**Advantages**: 

1. **No labeled data required**: TSDAE requires only a large volume of unlabeled domain text (collections of sentences) for training. This is exceptionally friendly to vertical domains where annotation costs are prohibitive or labeled data is scarce (such as legal, medical, or specialized industry terms)—as long as textual corpora for that domain can be gathered, domain-adaptive pre-training can be performed.

1. **Capability to learn domain-specific vocabulary and linguistic patterns**: By training on reconstruction tasks over domain corpora, the model can learn "surface-level" and "shallow semantic" features unique to that domain, such as vocabulary distributions, sentence structures, and terminology collocations. This assists subsequent downstream tasks in that domain (classification, retrieval, etc.), functioning as a form of "Domain-Adaptive Pre-training (DAPT)."

1. **Serving as a "warm start" before supervised fine-tuning**: A common practical strategy is to conduct unsupervised pre-training with TSDAE on domain corpora first, followed by supervised contrastive fine-tuning using a small amount of labeled data (similar to SetFit's philosophy). Compared to fine-tuning a general model directly, this two-stage approach of "unsupervised domain adaptation followed by supervised fine-tuning" typically yields superior performance when annotated data is scarce.

 **Disadvantages**: 

1. **Lower upper bound on representation quality compared to supervised contrastive learning**: An "objective mismatch" exists between TSDAE's training target (sentence reconstruction) and final application goals (semantic similarity, retrieval ranking)—a representation capable of reconstructing the original sentence is not necessarily a "good" representation for semantic similarity calculations. Consequently, representations learned by TSDAE usually require further supervised fine-tuning to perform well on downstream tasks; relying solely on TSDAE pre-trained representations directly for semantic search typically underperforms models trained via contrastive learning with positive and negative pairs.

1. **Weaker modeling of sentence-level semantic relationships**: TSDAE performs "noise addition and reconstruction" within a single sentence. During training, the model receives no cross-sentence relational signals such as "whether two different sentences are semantically similar." Thus, for tasks like "judging whether two text passages convey the same meaning," TSDAE pre-training offers limited direct signals, functioning primarily to "familiarize the model with domain language."

1. **Training cost and data volume requirements**: Although human labels are not required, achieving significant performance gains from unsupervised pre-training still requires a considerable domain corpus (on the scale of tens to hundreds of thousands of sentences). If available text in the target domain is itself scarce, the gains from TSDAE will be constrained.

1. **Trade-off between domain drift and generalization**: Over-training on domain corpora may cause the model to "overfit" to domain-specific language patterns, sacrificing a portion of its generalization capability across general domains (analogous to catastrophic forgetting during continued pre-training). Controlling the extent of training (e.g., training steps, learning rate) is necessary.

 **Summary**: Unsupervised methods like TSDAE hold significant advantages in "data availability," making them especially suitable for domain adaptation scenarios where annotations are scarce. However, the intrinsic "discriminative power" of their learned representations is generally inferior to supervised contrastive learning. Therefore, in practice, they are best positioned as a **pre-training/warm-start stage prior to supervised fine-tuning**, combined with supervised fine-tuning on a small amount of labeled data to form a two-stage pipeline of "unsupervised domain adaptation + supervised task alignment," rather than being deployed alone as a final solution.

</details>

**8. When Training TSDAE, How Should the Word Deletion Ratio (Noise Intensity) Be Set? What Issues Arise If the Ratio Is Too High or Too Low?**

<details>
<summary>💡 Show Answer</summary>

The primary hyperparameter controlling "noise intensity" in TSDAE is the word deletion ratio. Setting this parameter directly influences the difficulty of the training task and the quality of the resulting representations:

- **Issues with a deletion ratio that is too low**: If only a small percentage of words is deleted (e.g., under 5%), the encoder receives inputs with minimal corruption. The decoder can easily reconstruct the original sentence without needing to truly "understand" its core semantics (since most information remains intact in the input). Such a training task is overly simple, preventing the model from acquiring sentence representations with sufficient discriminative power or information compression capability, resulting in limited training gains;
- **Issues with a deletion ratio that is too high**: If the deletion ratio is excessively high (e.g., above 60%), the input becomes heavily corrupted. Even humans would struggle to accurately reconstruct the original sentence from the remaining sparse words. Under such conditions, the model may struggle to learn meaningful reconstruction patterns, potentially causing training instability (since many training instances effectively become "impossible tasks," leading to slow or erratic loss convergence);
- **Empirical value from the TSDAE paper**: Experiments in the original TSDAE paper revealed that a deletion ratio around 60% (a relatively high ratio) unexpectedly yielded strong performance. While somewhat counterintuitive (as high corruption ratios are typically thought to make the task excessively difficult), this finding highlights that a "sufficiently challenging task" is necessary to force the encoder to learn representations with genuine information compression value. Overly simple reconstruction tasks fail to provide sufficient learning signals;
- **Tuning recommendations in practice**: The precise optimal ratio varies depending on dataset characteristics (average sentence length, vocabulary richness, etc.) and specific tasks. In practice, it is recommended to use published values (around 60%) as a starting baseline and perform small-scale hyperparameter search experiments on a validation set using target domain data. By observing performance trends on downstream tasks (such as semantic similarity or retrieval) across different deletion ratios, one can select the best-performing configuration rather than blindly applying a static number.

In summary, TSDAE's word deletion ratio must strike a balance between "tasks that are too simple and lack effective learning signals" and "tasks that are overly difficult, nearly impossible to solve, and lead to unstable training." Empirical results from the original paper suggesting a relatively high deletion ratio (~60%) indicate that a "moderately high difficulty level" fosters the learning of valuable representations. Nevertheless, the exact optimal value should be determined empirically based on target dataset characteristics.

</details>

**9. What Are the Specific Structural Designs of the Encoder and Decoder in TSDAE Training? Why Can the Decoder Be Discarded During Inference?**

<details>
<summary>💡 Show Answer</summary>

Understanding the architectural design details of TSDAE's encoder and decoder helps clarify why it can produce high-quality sentence embeddings while maintaining lightweight efficiency during inference:

- **Encoder**: Typically uses a standard Transformer encoder (e.g., BERT), taking noisy corrupted sentences as input. Through the aforementioned pooling strategies (the TSDAE paper validates that using the output at a special token position yields superior results), it outputs a fixed-length sentence vector;
- **Decoder**: Typically employs another Transformer structure (either a decoder-only architecture or an encoder architecture augmented with cross-attention). Its critical design constraint is that when reconstructing the original sentence, the decoder **can only obtain information through this single fixed-length vector output by the encoder** (e.g., using this vector as the sole key/value source for cross-attention mechanisms at every decoder layer, or as the initial hidden state). It cannot access the full sequence of encoder outputs across all positions like standard Transformer encoder-decoder models (e.g., machine translation models). This "information bottleneck" design is key to TSDAE learning high-quality single-sentence vector representations;
- **Why the decoder can be discarded during inference**: Once training is complete, the decoder's sole purpose was to supply a "reconstruction loss" signal during training, backpropagating gradients to guide the encoder in compressing the sentence's core semantics into that fixed-length vector. Once training concludes, the encoder has already mastered generating high-quality sentence vectors. The decoder itself does not participate in any computation required for "generating embedding vectors" during actual deployment. Consequently, only the encoder portion is retained during inference; new sentences are simply processed through encoder forward passes and pooling to obtain sentence vectors. The decoder can be discarded entirely, imposing zero extra computational or storage overhead during inference—a practical advantage compared to self-supervised methods requiring auxiliary structures at inference time.

In summary, within TSDAE's encoder-decoder framework, the decoder serves as a "source of supervision signal during training." By forcing the decoder to reconstruct the original sentence relying exclusively on the single vector output by the encoder, it compels the encoder to compress core sentence semantics. Once training is completed, the decoder can be discarded, leaving only the lightweight encoder for deployment. This represents an engineering-friendly characteristic of autoencoder-style pre-training methods like TSDAE.

</details>

**10. After Unsupervised Pre-training with Methods Like TSDAE, Is Further Supervised Fine-Tuning Still Necessary? How Should the Two Stages Be Connected?**

<details>
<summary>💡 Show Answer</summary>

Although unsupervised pre-training methods like TSDAE can learn meaningful sentence representations in the complete absence of labeled data, in the vast majority of practical scenarios, unsupervised pre-training is usually **not the end goal**, but rather serves as a superior starting point for subsequent supervised fine-tuning:

- **Why supervised fine-tuning is usually still necessary**: Although representations learned via unsupervised pre-training surpass those of raw pre-trained language models (such as BERT without sentence-level training), the training signal originates from "reconstruction"—a relatively indirect proxy task—rather than directly optimizing "how semantic similarity should manifest in vector space." As a result, its performance on specific downstream tasks (such as semantic similarity scoring or retrieval ranking) still generally lags behind models fine-tuned with supervised contrastive learning (even with only a modest amount of high-quality data);
- **Typical two-stage transition pipeline**: First, perform unsupervised pre-training with TSDAE on large-scale unlabeled target domain text to adapt the model's representation space to domain-specific language distributions and characteristics (serving as "domain adaptation," particularly when the domain deviates significantly from general corpora). Building upon this, use high-quality in-domain labeled/weakly-supervised data (e.g., a small set of human-annotated similar sentence pairs or Q&A pairs) to conduct standard supervised contrastive fine-tuning, further calibrating the representation space to align with specific downstream task requirements;
- **Value of this two-stage pipeline**: Particularly in domains where labeled data is extremely scarce (such as non-natural language domains, amino acid sequences, circuit designs, etc.), conducting TSDAE pre-training on abundant unlabeled domain text first, followed by fine-tuning on scarce labeled data, significantly reduces reliance on large labeled datasets. This constitutes a vital practical strategy in data-constrained scenarios;
- **When unsupervised pre-training alone is sufficient**: If the target scenario genuinely lacks any usable labeled or weakly-supervised data (where even small high-quality datasets are unattainable) and accuracy demands are relaxed, using the unsupervised pre-trained model directly is acceptable. In such cases, performance remains noticeably superior to raw pre-trained models lacking sentence-level training. However, whenever even a small amount of high-quality data is obtainable, further supervised fine-tuning almost invariably yields meaningful performance gains.

In summary, unsupervised pre-training methods like TSDAE are primarily positioned as providing a "superior starting point for further supervised fine-tuning." They play a critical role in "domain adaptation," particularly when target domains differ significantly from general corpora and labeled data is scarce. The typical production deployment pipeline follows a two-stage combination: "large-scale unlabeled pre-training + small-scale high-quality supervised fine-tuning," rather than treating unsupervised pre-training as a standalone final solution.

</details>

**11. What Is Generative Pseudo Labeling (GPL)? How Does It Achieve Domain Adaptation by Combining Query Generation and Pseudo-Labeling?**

<details>
<summary>💡 Show Answer</summary>

GPL (Generative Pseudo Labeling) is a domain adaptation method specifically designed to address the issue of "lacking labeled data in target domains." It cleverly combines query generation and pseudo-labeling techniques:

- **Step 1: Query Generation**: A specially trained query generation model (typically a sequence-to-sequence model such as T5, trained on general Q&A/retrieval data to acquire the capability of "generating a relevant query given a passage") is used to batch-generate pseudo-queries for unlabeled target domain documents. This automatically constructs a large volume of candidate positive pairs of (pseudo-query, target-domain document) without human annotation;
- **Step 2: Negative Sampling**: For each generated pseudo-query, an initial retrieval model (such as an off-the-shelf general embedding model not fine-tuned on the target domain, or BM25) retrieves a batch of candidate negative documents;
- **Step 3: Pseudo Labeling**: Because query generation models may yield variable-quality pseudo-queries and imprecise positive/negative relationships, GPL introduces a powerful cross-encoder (typically a high-precision reranker pre-trained on general data) to rescore each (pseudo-query, document) pair, yielding a "soft label" (a continuous relevance score rather than a hard binary 0/1 label) as the ultimate training signal;
- **Step 4: Training the target domain embedding model with soft labels**: The soft labels provided by the cross-encoder (rather than simple binary positive/negative labels) are used to train the target domain dual-encoder. Typically using MSE or distillation-style losses, the dual-encoder's similarity scores are trained to approximate the soft label scores given by the cross-encoder.

In summary, GPL achieves effective domain adaptation for embedding models without human annotations in the target domain by pairing "automatic query generation to solve the absence of target domain queries" with "pseudo-labeling via a stronger cross-encoder to resolve variable query generation quality." It represents a clever automated pipeline for addressing vertical domain data scarcity, essentially combining knowledge distillation (using a cross-encoder to guide a dual-encoder) with generative data augmentation.

</details>

**12. When Facing a Completely Unlabeled New Vertical Domain, How Should a Complete Unsupervised Embedding Model Training Pipeline Be Designed?**

<details>
<summary>💡 Show Answer</summary>

Combining the unsupervised and weakly-supervised techniques discussed earlier, a systematic pipeline can be outlined to address the common practical challenge of a target domain completely lacking labeled data:

- **Step 1: Evaluate zero-shot transfer performance of existing general models**: First, evaluate how an off-the-shelf embedding model pre-trained on general corpora performs directly (zero-shot, without any fine-tuning) on the target domain data. If the distribution discrepancy between the target domain and general pre-training corpora is small (e.g., domain text vocabulary and sentence structures closely resemble general text), using an off-the-shelf model may already achieve acceptable performance without requiring additional training investment;
- **Step 2: Perform unsupervised domain-adaptive pre-training if discrepancies are large**: If significant differences exist, use unsupervised methods like TSDAE to perform further pre-training on large-scale unlabeled text in the target domain, adapting the model's representation space to domain-specific vocabulary distributions and language patterns;
- **Step 3: Automatically construct pseudo-labeled data for further fine-tuning**: Use methods like GPL discussed earlier (query generation + cross-encoder pseudo-labeling) or leverage LLMs to generate hard negatives/positives to automatically construct pseudo-supervised training data for the target domain, which is then used for further contrastive learning fine-tuning;
- **Step 4: Supplement with small-scale, high-quality human annotations (if obtainable)**: Even if large-scale annotation is impossible, attempt to acquire a small batch (e.g., tens to hundreds of samples) of high-quality human-annotated data (even if domain experts spend a short time verifying model-high-confidence pairs). Although small in scale, this dataset acts as an "anchor," validating the quality of automatically generated pseudo-labels or providing a final high-precision micro-fine-tuning step;
- **Step 5: Iterative verification and evaluation**: Because the pipeline relies heavily on automatically constructed data (lacking large-scale human gold standards), special attention must be paid to validating the training process and final performance via qualitative spot-checks (manually auditing whether sample pairs deemed similar/dissimilar by the model align with intuition), preventing systematic biases in the automated pipeline from going unnoticed.

In summary, when faced with a new vertical domain lacking labeled data, a pragmatic systematic strategy follows this sequence: "Evaluate zero-shot transfer of off-the-shelf models $\rightarrow$ Perform unsupervised domain-adaptive pre-training if necessary $\rightarrow$ Fine-tune with automatically generated pseudo-labeled data $\rightarrow$ Supplement with a small set of high-quality human annotations for validation and polishing $\rightarrow$ Continuously verify validity through qualitative spot-checks." This pipeline comprehensively integrates multiple unsupervised/weakly-supervised techniques discussed in this chapter, representing a benchmark practical workflow for training embedding models under data constraints.

</details>

**13. What Role Does Query Generation (e.g., doc2query) Play in Unsupervised/Weakly-Supervised Embedding Model Training?**

<details>
<summary>💡 Show Answer</summary>

Query generation refers to training a dedicated model that takes a document/passage as input and generates user queries that the passage could address. This technique is widely applied in constructing unsupervised/weakly-supervised retrieval training data:

- **Basic principle**: Typically based on sequence-to-sequence models (such as T5), the generator is trained on existing labeled (query, relevant document) datasets (e.g., public Q&A datasets or retrieval benchmarks like MS MARCO) to acquire the capability of "generating queries that a given passage can answer or satisfy";
- **Application in weakly-supervised data construction**: The trained query generation model can be applied to any new document collection lacking native query annotations (e.g., a target vertical domain document corpus) to automatically generate one or more pseudo-queries per document. This rapidly creates large-scale (pseudo-query, document) positive pairs at low cost for subsequent contrastive learning, serving as the concrete implementation of the query generation step in GPL discussed earlier;
- **Application of doc2query in traditional sparse retrieval**: Notably, doc2query was not originally created for embedding models, but was proposed to **enhance traditional sparse retrieval (such as BM25)**. By **appending** generated pseudo-query vocabulary to the original document (expanding the document's vocabulary coverage), queries originally unmatchable by BM25 due to vocabulary mismatch gain a higher probability of matching via the newly added words. This concept was naturally transferred to weakly-supervised data construction for embedding models;
- **Quality control**: The quality of generated pseudo-queries directly impacts subsequent training data quality. In practice, filtering techniques via guide models or cross-encoder pseudo-labeling (as discussed earlier) are typically incorporated to verify the relevance between generated queries and original passages. Low-quality pseudo-queries (e.g., those inconsistent with passage content or overly vague) are filtered out to prevent low-quality synthetic data from undermining training effectiveness.

In summary, query generation technology trains a "document-to-query" generation model to automatically synthesize large-scale pseudo-queries for document collections lacking native queries. It stands as one of the core technologies for constructing weakly-supervised retrieval training data. Its application has expanded from document expansion in traditional sparse retrieval to weakly-supervised/unsupervised training data construction for modern embedding models, typically requiring quality verification mechanisms to ensure data usability.

</details>

**14. Comparing Three Unsupervised Methods (TSDAE, SimCSE, and ICT): What Are Their Requirements for Training Data? What Types of Application Scenarios Are They Best Suited For?**

<details>
<summary>💡 Show Answer</summary>

Comparing the three representative unsupervised sentence/document embedding pre-training methods discussed earlier side-by-side helps make reasonable choices in practice based on data characteristics:

- **Comparison of Data Requirements**: Unsupervised SimCSE has the most lenient data requirements, needing only an arbitrary collection of independent sentences without requiring any structural association between them, because its positive samples rely entirely on internal dropout noise construction. TSDAE similarly requires only a collection of independent sentences or short texts, because its positive samples are "complete--damaged" pairs of the same sentence and do not rely on external associations between texts. In contrast, ICT requires the training corpus to consist of **longer documents with a certain internal structure** (capable of partitioning a "sentence" and the "context to which the sentence belongs"). If the training corpus itself consists only of scattered short sentences (lacking longer document context), ICT can hardly be applied directly.
- **Differences in Learned Representation Relationships**: The core capability learned by SimCSE and TSDAE leans more toward "identifying the consistency of the same semantics under different expressions" (paraphrase and semantic similarity relationships). The core capability learned by ICT leans more toward "identifying the relevance between a concise snippet and its broader context" (closer to the relationship between queries and documents in retrieval scenarios).
- **Summary of Applicable Scenarios**: If the target application scenario involves tasks where "the lengths and properties of the two texts are similar"---such as semantic similarity comparison, paraphrase detection, or sentence clustering---SimCSE or TSDAE is a more suitable choice. If the target application scenario is a typical retrieval task (where the query is usually much shorter than the document and their contents do not completely overlap), ICT (and its derivative works such as Contriever) aligns better with downstream tasks in terms of training data distribution. If the target domain lacks structured long documents (e.g., containing only scattered short texts), TSDAE or SimCSE offers greater advantages in data availability.
- **Possibility of Combined Usage**: In practice, these methods are not mutually exclusive. Many pre-training pipelines for modern embedding models comprehensively utilize multiple unsupervised data construction approaches (such as simultaneously using TSDAE-style noisy reconstruction data and ICT-style document--sentence mining data) to cover a broader range of semantic relationship types and improve the generalization capability of the final model.

In summary, the three methods---SimCSE, TSDAE, and ICT---exhibit clear differences in structural requirements for training data and the types of learned semantic relationships. Choices in practice should be made based on the specific format of available data (whether long document structures exist) and whether downstream applications lean closer to similarity comparison or retrieval tasks. Multiple methods can also be combined to achieve a more comprehensive representation learning effect.

</details>

**15. In Unsupervised Contrastive Learning, How Can Models Avoid Learning "Shortcut Solutions" (Shortcut Learning)---That Is, Relying on Surface Features Rather Than Genuine Semantic Information?**

<details>
<summary>💡 Show Answer</summary>

The shortcut learning issue is a widespread risk in self-supervised/unsupervised learning---models may find certain "shortcuts" that are superficially correlated with the training task but do not reflect genuine semantic understanding to decrease training loss, rather than learning the desired deep semantic representations:

- **Typical Manifestations of Shortcut Solutions**: Taking SimCSE as an example, if the noise introduced by dropout is too small, or if the model finds a simple rule (such as relying solely on sentence length or the presence of certain high-frequency words) to determine whether two vectors originate from two passes of the same sentence without genuinely understanding its semantic content, a shortcut solution may occur. In such cases, the model performs well on training loss, but the quality of the learned representations on real downstream tasks remains unsatisfactory. Similarly, in TSDAE, if the decoder finds simple heuristic rules (such as part-of-speech distribution patterns of the remaining words) rather than truly understanding semantics to reconstruct sentences, similar issues can arise.
- **Proper Design of Data Augmentation/Noise Intensity**: As discussed previously (e.g., TSDAE deletion ratios and choice of data augmentation methods), ensuring that the intensity of noise or augmentation is neither too simple (easily exploited by shortcuts) nor too extreme (rendering the task infeasible) is a fundamental approach to mitigating shortcut learning.
- **Introducing Richer and More Diverse Data Augmentation Methods**: A single augmentation method (such as using only dropout or only a fixed ratio of random word deletion) makes it easier for the model to find specific shortcuts. Combining multiple data augmentation methods discussed earlier (e.g., back-translation, paraphrase generation, random deletion at varying intensities) in randomized combinations increases the difficulty of forming shortcuts, forcing the model to learn more fundamental semantic features.
- **Validation via Downstream Task Performance Rather Than Relying Solely on Training Loss**: A key characteristic of shortcut learning is that training loss decreases normally while downstream task performance remains poor (a phenomenon mentioned in the earlier training diagnostics discussion). Therefore, continuously monitoring performance on multiple distinct types of downstream tasks during training (rather than focusing solely on the training loss curve) is a critical strategy for detecting shortcut solutions promptly.
- **Auxiliary Diagnostics from a Theoretical Perspective**: The Alignment/Uniformity metrics mentioned earlier can also assist in diagnosing shortcut learning to some extent. If the Alignment metric (closeness of positive sample pairs) shows abnormal behavior as training loss decreases (such as converging prematurely while the Uniformity metric simultaneously becomes abnormal), it may indicate that a simplified shortcut is being exploited by the model.

In summary, the core risk of shortcut learning lies in the model finding surface patterns unrelated to true semantic understanding to "tackle" the training task. Mitigation strategies include properly designing augmentation/noise intensity, combining diverse data augmentation methods, and consistently using downstream task performance (rather than pure training loss) as the benchmark for determining genuine training effectiveness. It represents a class of issues requiring continuous vigilance in unsupervised representation learning.

</details>

**16. Can Denoising Reconstruction Objectives (e.g., TSDAE's Reconstruction Loss) Be Combined with Contrastive Learning Objectives for Joint Training to Further Improve Embedding Model Performance?**

<details>
<summary>💡 Show Answer</summary>

Having previously discussed two distinct unsupervised training routes---reconstruction-based (TSDAE) and contrastive learning-based (SimCSE, etc.)---a natural question arises: can the two be combined to leverage their complementary strengths?

- **Basic Idea of Combination**: Within a single training framework, two loss terms are configured simultaneously---a reconstruction loss (such as TSDAE-style noisy reconstruction, requiring the model to learn to restore the original text from damaged inputs) and a contrastive learning loss (such as InfoNCE, requiring the model to learn to distinguish positive from negative samples). A weighted sum of the two losses serves as the final training objective, sharing the same encoder backbone.
- **Theoretical Complementarity**: The reconstruction loss focuses more on requiring representations to **retain sufficient information** to support precise content restoration (information integrity-oriented). In contrast, the contrastive learning loss focuses more on requiring representations to **possess good discriminability in the semantic space** (discriminative capability-oriented). Their optimization targets are complementary: a pure reconstruction task may cause the model to overemphasize surface lexical details (to accurately restore the original text) while insufficiently focusing on semantic-level discrimination; whereas pure contrastive learning may yield representations with limitations when negative sample coverage is inadequate (such as false negatives and batch size constraints discussed earlier). Jointly training both objectives theoretically helps the model balance both "information fidelity" and "semantic discriminability."
- **Practical Challenges**: Joint training requires careful design of weight balancing between the two loss terms (similar to the multi-task loss weighting problem discussed previously). Improper weight settings may lead to one objective dominating training while the other becomes virtually ineffective. In addition, the reconstruction task typically requires an additional decoder architecture during training (increasing computational overhead during the training stage, even though the decoder can still be discarded during inference), which correspondingly increases engineering complexity.
- **Empirical Results and Current Research Status**: This joint training idea has been explored in several research studies. Experimental results show that in specific scenarios (especially when target domain data is scarce and limited unlabeled data must be fully exploited), joint training may yield performance gains over single-objective training. However, the magnitude of improvement varies across specific tasks and datasets, and it does not yield significant, consistent gains in all scenarios. Whether introducing this additional complexity is worthwhile must be empirically validated based on the specific application scenario.

In summary, the joint training approach combining reconstruction loss and contrastive learning loss can theoretically balance both information integrity and semantic discriminability---two optimization directions with different emphasis---showing clear complementary potential. In practice, however, loss weight balancing and additional engineering complexity must be handled carefully, and performance gains vary by scenario. It remains a noteworthy technical direction in unsupervised embedding model training, though it has not yet become a universally adopted standard practice.

</details>

**17. What Unique Value Do Unsupervised Pre-Training Methods (e.g., TSDAE) Offer in Low-Resource Languages or Low-Resource Domains Compared to Supervised Methods?**

<details>
<summary>💡 Show Answer</summary>

Low-resource scenarios (whether resource-poor minor languages or data-scarce vertical domains) represent important application settings where unsupervised pre-training methods demonstrate unique value:

- **Fundamental Dilemma of Supervised Methods in Low-Resource Scenarios**: Supervised contrastive learning (whether relying on manual annotations or the various weak-supervision data mining techniques discussed earlier) fundamentally requires a certain scale of **structured association data** (similar sentence pairs, query--document pairs, etc.). Low-resource languages/domains often severely lack even these basic sources of structured data (such as Wikipedia, Q&A communities, or click logs), leaving supervised/weakly supervised methods helpless due to the lack of raw materials ("making bricks without straw").
- **Lenient Data Requirements of Unsupervised Pre-Training**: Taking TSDAE as an example, its training requires only **arbitrary raw unlabeled text** (without requiring any paired relationships between sentences). The accessibility of this data format is far higher than that of structured paired data---even low-resource languages can usually yield a certain volume of raw text such as news, books, or government documents (even if limited in scale). This allows methods like TSDAE to remain functional even in extreme scenarios with virtually zero structured data.
- **Combining with Multilingual Pre-Trained Backbones**: For low-resource language scenarios, it is common to first start with a backbone model pre-trained on multilingual corpora (such as XLM-R mentioned in the cross-lingual training discussion). Further TSDAE-style pre-training using a small amount of raw text in the target low-resource language can then adapt the representation space specifically to that language, leveraging the existing cross-lingual knowledge in the backbone.
- **Realistic Performance Expectations**: It is important to objectively recognize that while unsupervised pre-training in extreme low-resource scenarios is "always better than no training at all," its performance is still usually significantly weaker than in scenarios with abundant high-quality annotated data. In practice, it should be positioned as the "optimal feasible solution under constrained data conditions," rather than a "panacea" capable of fully making up for data scarcity. If even a small amount of high-quality annotated/weakly supervised data can subsequently be obtained, further supervised fine-tuning on top of unsupervised pre-training is still strongly recommended (such as the two-stage bridging strategy discussed earlier).

In summary, unsupervised pre-training methods---by virtue of their lenient data requirements (needing only raw text without any structured paired relationships)---possess irreplaceable, unique value in low-resource language or domain scenarios where supervised/weakly supervised data is practically unavailable. They represent a priority technical path worth considering under extreme data constraints, though realistic expectations regarding their performance ceiling must be maintained.

</details>

**18. How Do the Convergence Speed and Required Training Duration of Unsupervised Pre-Training Methods (e.g., TSDAE) Differ from Supervised Contrastive Learning?**

<details>
<summary>💡 Show Answer</summary>

From an engineering practice perspective, understanding the characteristics of different training methods regarding computational costs and training duration helps properly plan training resources and project schedules:

- **Relationship Between Task Difficulty and Convergence Speed**: The reconstruction task in TSDAE (especially under the higher deletion ratio settings discussed earlier) is inherently quite challenging, as the model must learn to recover full information from severely damaged inputs. Such tasks typically converge relatively slowly, requiring more training epochs/steps for the model to learn stable and effective representations. In contrast, when the training data itself is of high quality (such as positive and negative pairs constructed from supervised NLI data), contrastive learning under the relatively more direct goal of "identifying positive versus negative pairs" often shows faster improvements in training loss and downstream validation metrics.
- **Additional Computational Overhead from Decoder Training**: As discussed earlier, the training stage of TSDAE requires running both encoder and decoder networks simultaneously (even though the decoder is discarded during inference). This means that for the same batch size, the single-step computational load of TSDAE is typically higher than that of pure dual-encoder contrastive learning (which usually only runs the encoder itself). This extra overhead must be factored into total training time planning.
- **Typically Larger Scale of Unsupervised Pre-Training Data**: Since the cost of obtaining unsupervised pre-training data is far lower than that of annotated/weakly supervised data (as highlighted in the low-resource scenario advantages), practice often favors using much larger unlabeled corpora for TSDAE training. This implies that even if single-step training is slower, the total training duration ($number of epochs \times time per epoch$) will usually be correspondingly longer due to the goal of traversing larger data volumes.
- **Resource Planning Recommendations in Practice**: During project planning, if unsupervised pre-training such as TSDAE is chosen as part of the pipeline, adequate computational resources and time budgets should be reserved for this stage (rather than planning with the same resource expectations as fast, short-cycle supervised fine-tuning). Combined with the training diagnostics metrics discussed earlier, performance curves should be continuously monitored during training to avoid under-training or wasting resources caused by blindly following fixed epoch settings.

In summary, because TSDAE involves high task difficulty, requires running an additional decoder network, and is typically paired with larger unlabeled corpora, its convergence speed and total training duration overhead differ significantly from relatively more "direct" supervised contrastive learning. In practice, project scheduling and resource planning must take these differences into account to avoid underestimating the time and compute investments required for the unsupervised pre-training phase.

</details>

**19. Why Does TSDAE Choose Reconstruction Loss Over Contrastive Loss for Training? What Are the Respective Advantages of These Two Losses?**

<details>
<summary>💡 Show Answer</summary>

Understanding why the designers of TSDAE chose the reconstruction loss route rather than directly adopting contrastive learning loss like SimCSE helps gain deeper insight into the theoretical characteristics and application boundaries of both loss types:

- **Core Advantage of Reconstruction Loss: No Reliance on Negative Samples**: The effectiveness of contrastive learning loss (such as InfoNCE) highly depends on the quality and quantity of negative samples (as repeatedly discussed regarding false negatives, hard negative mining, batch size impacts, and complex engineering considerations). In contrast, reconstruction loss requires no negative sample construction at all---it only concerns the self-contained task of "whether the original input can be restored from the compressed representation." Its training mechanism is simpler and immune to negative sample quality issues, which is a major reason why TSDAE exhibits robust performance in cross-domain adaptation scenarios.
- **Limitations of Reconstruction Loss**: The optimization target of reconstruction loss is "whether information can be restored," which is not fully equivalent to "whether semantic representations possess good discriminability." Even if a representation supports the reconstruction task well, it does not guarantee superior performance on downstream tasks such as semantic similarity comparison or retrieval ranking. There exists an inherent risk of "objective misalignment" between reconstruction loss and downstream task goals, which explains why further supervised contrastive fine-tuning on top of TSDAE pre-training is typically recommended.
- **Core Advantage of Contrastive Loss: Direct Optimization of Downstream Objectives**: Contrastive learning loss directly optimizes the goal that "similar samples should be close in vector space, while dissimilar samples should be far apart"---an objective highly aligned with semantic similarity and retrieval tasks. Therefore, when supported by reliable training data (whether supervised or high-quality weakly supervised), it generally improves downstream task performance more directly and efficiently.
- **Essence of Design Trade-Offs**: TSDAE's choice of reconstruction loss can essentially be viewed as a trade-off between "training mechanism robustness / non-reliance on negative sample quality" and "direct alignment with downstream task objectives." In scenarios where negative sample quality cannot be guaranteed (such as new vertical domains lacking reliable negative sampling sources), the robustness advantage of reconstruction loss becomes particularly prominent. Conversely, in scenarios where negative sample quality is assured (supported by sufficient annotations or high-quality weak supervision), the direct alignment advantage of contrastive loss becomes more crucial.

In summary, reconstruction loss and contrastive loss represent two distinct technical trade-offs: "no reliance on negative samples with a more robust training mechanism but indirect alignment with downstream goals" versus "direct optimization of semantic discriminative targets with a higher performance ceiling but heavy reliance on negative sample quality." TSDAE's selection of reconstruction loss is a targeted design choice made with full consideration of its target application scenarios (cross-domain adaptation where negative sample quality is difficult to guarantee). The two losses are not mutually exclusive and can be combined or deployed in stages as previously discussed.

</details>

**20. In the Complete Absence of Annotated Data, How Can One Evaluate Unsupervised Pre-Training (e.g., TSDAE) and Determine Whether Training Is Effective?**

<details>
<summary>💡 Show Answer</summary>

A realistic challenge in unsupervised pre-training scenarios is: if no annotated data is available at all, how can one assess whether the trained model's performance is good or bad? Common coping strategies include:

- **Using Public General Benchmarks for Proxy Evaluation**: Even if the target domain itself lacks annotated data, public general semantic similarity/retrieval benchmarks (such as subtasks in STSB or MTEB discussed earlier) can still be periodically evaluated during training. Although the data distribution of these benchmarks may differ from the target domain, they provide at least a **relative** reference signal to judge whether training is progressing toward "improving sentence representation quality" (e.g., if training loss decreases while benchmark performance continuously deteriorates, it may signal issues such as shortcut learning or representation collapse).
- **Constructing a Small-Scale In-Domain Human Validation Set**: Even if large-scale annotation costs cannot be afforded, efforts should be made to have domain experts dedicate a small amount of time to annotating tens to a hundred sample pairs, forming a small yet high-quality in-domain validation set. This serves to monitor performance during training in a way closer to real application scenarios, aligning with the "anchor stone" role of small-scale high-quality data in new vertical domain training pipelines.
- **Qualitative Spot-Checking and Manual Review**: Randomly sample pairs of instances predicted by the model as highly similar or highly dissimilar on a regular basis, and have personnel familiar with the domain conduct qualitative reviews to judge whether these similarity assessments align with intuition and domain common sense. Although such qualitative "spot checks" do not provide quantitative metrics, they can quickly uncover obvious model flaws (such as systematic misjudgments of specific phrasing patterns).
- **Leveraging Unsupervised Training Diagnostics**: Even without annotated downstream data, label-free diagnostic metrics discussed earlier (such as positive vs. negative similarity gap, Alignment/Uniformity metrics, vector distribution health, etc.) can be monitored to assess whether the training process is normal or experiencing collapse. While these metrics do not directly reflect downstream task performance, they help rule out explicit training anomalies.

In summary, evaluating unsupervised pre-training in unannotated data scenarios requires a combination of leveraging public general benchmarks as relative references, building small-scale high-quality domain validation sets, performing qualitative spot reviews, and monitoring label-free training diagnostics. Although no single method can replace the reliability provided by a large-scale annotated evaluation benchmark, combining them offers a sufficiently solid basis for judging training effectiveness under data-constrained conditions.

</details>

**21. When Adapting a Pre-Trained General-Domain Embedding Model to a Specific Vertical Domain, What Other Common Domain Adaptation Approaches Exist Besides TSDAE?**

<details>
<summary>💡 Show Answer</summary>

In addition to TSDAE and related technologies (e.g., GPL) discussed in detail previously, several other common technical approaches exist for domain adaptation in embedding models, which are often used in combination according to practical conditions:

- **Continued Pre-Training**: Performing further self-supervised pre-training such as Masked Language Modeling (MLM) on large-scale unlabeled text in the target domain using the general pre-trained backbone (rather than necessarily sentence-level TSDAE). This first adapts the vocabulary and linguistic patterns of the backbone to the target domain (e.g., medical or legal fields containing specialized terminology rare in general corpora), after which sentence-level contrastive learning is stacked on top. This shares similarities with TSDAE but operates at a different level (token-level vs. sentence-level), and the two can also be used in combination.
- **Domain Vocabulary Expansion**: If the target domain contains numerous specialized terms that general tokenizers struggle to split effectively (such as chemical formulas or code identifiers), expanding or retraining the tokenizer vocabulary on target domain corpora can be considered. This enables the model to represent domain-specific lexical units more efficiently, a step that typically needs to precede continued pre-training.
- **Adversarial Domain Adaptation**: Borrowing adversarial training concepts from transfer learning, a "domain discriminator" is introduced. The encoder is trained so that the discriminator **cannot** distinguish whether representations originate from the source domain (general corpus) or the target domain. Through this adversarial process, the encoder is encouraged to learn domain-invariant and more generalizable semantic representations. This approach is relatively more complex and less widely applied in embedding model domain adaptation than the aforementioned methods.
- **Lightweight Domain Adaptation via PEFT/LoRA**: As discussed previously, parameter-efficient fine-tuning methods such as LoRA can be applied to perform lightweight adaptation on small target domain datasets. This preserves the general pre-trained model's language capabilities while tailoring it to the target domain at low cost, making it particularly suitable for scenarios requiring separate adapted versions across multiple vertical domains.

In summary, beyond TSDAE---which centers on sentence-level reconstruction tasks---token-level continued pre-training, domain vocabulary expansion, adversarial domain adaptation, and lightweight adaptation with PEFT are all common technical means for domain adaptation in embedding models. In practice, a comprehensive adaptation scheme is often constructed by combining multiple techniques based on the disparity between target domain and general corpora, available compute resources, and whether multiple domains need to be adapted simultaneously.

</details>

**22. Are All Types of Domain Texts Suitable for Direct Use in TSDAE Training? What Data Characteristics Affect TSDAE Training Performance?**

<details>
<summary>💡 Show Answer</summary>

Although TSDAE has relatively lenient requirements regarding training data format (needing only raw unlabeled text), not all types of text data can be equally and effectively converted into high-quality training signals. The following data characteristics warrant attention:

- **Information Density and Sentence Length**: TSDAE's core task is to reconstruct full sentences from damaged inputs. If sentences in the training corpus are too short (such as multi-word phrases or titles), deleting a certain proportion of words may leave too little remaining information, rendering reconstruction virtually impossible and producing low-quality training signals. Conversely, if sentences are excessively long and contain heavy redundancy or highly predictable content (such as templated text), the model might complete reconstruction through simple pattern matching rather than genuine semantic understanding (risking shortcut learning as discussed earlier). Therefore, sentence length distribution and information density must match hyperparameters such as the deletion ratio.
- **Topic Diversity and Consistency of the Corpus**: If the target domain corpus is overly narrow or highly repetitive in topic (such as large volumes of highly structured, templated text), the model may struggle to learn sufficiently rich and discriminative representations because the range of semantic variation provided by the training data is limited.
- **Text Quality and Noise Level**: If the raw corpus itself contains severe noise (such as garbled text, formatting errors, or meaningless fragments in web-scraped data), this noise mixes with the artificial noise introduced by TSDAE, potentially interfering with the model's ability to learn meaningful semantic reconstruction patterns. In practice, raw corpora must undergo necessary cleaning (removing obvious noise and formatting errors) before being used for TSDAE training.
- **Adaptability to Structured/Semi-Structured Text**: For highly structured text (such as text converted from tabular data or source code), linguistic characteristics differ significantly from natural language. Directly applying word-deletion strategies designed for natural language (such as the ratio settings discussed earlier) may be inappropriate, requiring specialized noise injection strategies (such as deleting along code syntax structures rather than simple word deletion).

In summary, although TSDAE has much more relaxed data format requirements than structured paired data, training corpus sentence length distribution, topic diversity, text quality, and structural specificity significantly impact final training results. In practice, before applying TSDAE directly, a preliminary analysis of these target corpus characteristics should be conducted to adjust data cleaning strategies and noise injection hyperparameters accordingly, rather than assuming "any unlabeled text can be equally and effectively used for TSDAE training."

</details>

**23. How Feasible Is Applying TSDAE to Train Embedding Models for Low-Resource or Minor Languages? What Additional Considerations Require Attention?**

<details>
<summary>💡 Show Answer</summary>

Building on the application value in low-resource scenarios discussed earlier, this section explores specific considerations when applying TSDAE to train low-resource or minor language models:

- **Availability of Tokenization/Segmentation Tools**: TSDAE's noise injection process (random word deletion) relies on correctly segmenting sentences into words or subword units. If the target minor language lacks mature tokenizers, or if its word formation rules differ vastly from languages used to train the tokenizer (e.g., unsegmented scripts or morphologically complex agglutinative languages), extra care must be taken regarding whether the tokenizer produces reasonable segmentations. Poor tokenization quality directly undermines the validity of word deletion operations and the effectiveness of the reconstruction task.
- **Language Coverage of Pre-Trained Backbones**: As discussed earlier, continuing TSDAE training from a backbone model pre-trained on multilingual corpora (such as multilingual BERT or XLM-R) is recommended over training from scratch. However, it is essential to verify whether the target language is included in the backbone's pre-training language list. If completely uncovered ("unknown language" to the model), the backbone will lack fundamental linguistic knowledge of that language, severely capping TSDAE training performance and potentially necessitating preliminary steps like vocabulary expansion.
- **Uneven Scale and Quality of Available Corpora**: Raw text corpora for low-resource languages are often limited in size and restricted in domain source (e.g., predominantly religious texts or government documents). Topic diversity issues discussed earlier become more pronounced in low-resource settings, requiring effort to broaden corpus sourcing (e.g., social media, news sites) and perform appropriate filtering on variable-quality data.
- **Exacerbated Evaluation Challenges**: The evaluation difficulties in unannotated settings discussed earlier intensify in low-resource language scenarios. Many public general benchmarks (such as MTEB) offer very limited coverage for minor languages, making it a challenge to find even small-scale validation datasets. In practice, evaluation relies more heavily on qualitative spot-checking and human review by native speakers or language experts.

In summary, applying TSDAE to low-resource/minor language training is theoretically a feasible and important technical path. However, compared to high-resource settings, additional attention must be paid to tokenizer availability, backbone language coverage, corpus size/diversity limitations, and inherent evaluation difficulties. These factors collectively dictate the achievable performance ceiling, requiring realistic expectations during project planning.

</details>

**24. When Only a Small Amount of High-Quality Annotated Data Is Available, How Should One Design a Semi-Supervised / Few-Shot Domain Adaptation Pipeline Combining "Unsupervised Pre-Training + Small-Scale Annotated Fine-Tuning"?**

<details>
<summary>💡 Show Answer</summary>

Building on the two-stage concept mentioned previously, this section details a semi-supervised domain adaptation pipeline designed for the common practical constraint of "small-scale annotated data":

- **Reasonable Expectations for Data Scale**: "Small amount of annotated data" typically refers to scales ranging from dozens to a few thousand pairs (compared to millions or more in large-scale weakly supervised pre-training). Training from scratch at this scale yields limited performance and easily overfits. Therefore, the key to making full use of this valuable small dataset lies in **not requiring it to carry the entire training burden alone**, but rather treating it as a "refinement" step on top of large-scale unsupervised pre-training.
- **Technical Choice for the Fine-Tuning Stage**: Combined with PEFT/LoRA technologies discussed earlier, parameter-efficient fine-tuning methods are far less prone to overfitting than full-parameter fine-tuning when data is scarce (since trainable parameters are drastically reduced), making PEFT the preferred technical route for this stage.
- **Amplifying the Value of Small Annotated Data via Data Augmentation**: Data augmentation methods discussed earlier (such as paraphrase generation and back-translation) can be applied to moderately expand small annotated datasets. This increases surface diversity without altering core semantic relationships, mitigating overfitting risks stemming from data scarcity.
- **Boosting Data Efficiency with Hard Negative Mining**: Even if the number of positive samples is limited, hard negative mining techniques (such as using the current model or BM25 to mine hard negatives) can be leveraged to pair these limited positive pairs with high-quality hard negatives mined from a larger pool of unlabeled domain documents. This increases the density of supervision signals provided per training sample without incurring additional positive annotation costs.
- **Cross-Validation and Early Stopping Mechanisms**: Because the validation set itself may be small in size, cross-validation strategies (such as k-fold cross-validation) must be designed cautiously to evaluate model performance reliably, alongside setting appropriate early stopping mechanisms to prevent overfitting on small datasets. This differs from hyperparameter tuning in conventional large-scale training, requiring a more conservative and cautious stance.

In summary, facing the practical constraint of scarce high-quality annotated data, the core strategy is to "let unsupervised pre-training handle the primary representation learning work, while the small annotated dataset focuses only on final fine-grained calibration." Combining PEFT, data augmentation, hard negative mining, and cautious cross-validation maximizes the training value of precious yet limited annotations. This represents the canonical technical path adopted for deploying domain-specific embedding models today.

</details>

**25. What Are the Connections and Differences Between the GenQ (Generative Querying) Method and the GPL Method?**

<details>
<summary>💡 Show Answer</summary>

Both GenQ and GPL involve the core idea of leveraging generative models to automatically construct retrieval training data, but they differ in specific workflow design. Understanding their relationship helps master this technical direction more comprehensively:

- **Core Workflow of GenQ**: Consistent with the query generation approach discussed earlier, GenQ utilizes a pre-trained query generation model (such as T5) to batch-generate pseudo-queries for target domain documents, constructing $(pseudo-query, document)$ positive sample pairs. Standard contrastive learning (such as MNR/InfoNCE) is then applied directly to train the embedding model on these synthetic pairs. The overall process is relatively straightforward.
- **GPL's Improvements Over GenQ**: As detailed previously, GPL adds a critical step on top of GenQ's basic pseudo-query generation step---namely, **Cross-Encoder pseudo-labeling**. Rather than treating generated pseudo-queries directly as hard positive instances, GPL employs a stronger Cross-Encoder to re-score generated $(pseudo-query, document)$ pairs, yielding finer-grained soft labels as training signals. Simultaneously, negative samples are mined for each pseudo-query and scored similarly. This additional step effectively mitigates data quality issues inherent in purely generative models (where generated pseudo-queries might drift from document topics or be overly broad).
- **Performance Differences**: Because GPL incorporates quality verification and fine-grained soft-label supervision, it generally achieves superior domain adaptation performance compared to straightforward GenQ-style pseudo-labeling workflows. This represented the core improvement highlighted when GPL was introduced relative to earlier GenQ-like methods.
- **Trade-Offs in Engineering Costs**: Compared to GenQ, GPL introduces an additional Cross-Encoder scoring phase, entailing higher computational costs during data construction (requiring Cross-Encoder inference for every candidate pair). In scenarios with massive data scale and limited compute resources, one might first validate feasibility using a simpler GenQ-style workflow before deciding whether to invest in GPL's fine-grained pipeline based on performance demands.

In summary, both GenQ and GPL fall under the broader paradigm of "using generative models to automatically construct pseudo-queries." GPL can be viewed as an upgraded version of GenQ that incorporates Cross-Encoder pseudo-labeling, substituting rigid hard pseudo-labels with fine-grained soft-label supervision signals. While GPL generally yields superior performance, it introduces extra engineering and computational overhead. Selection in practice requires balancing specific resource constraints against performance requirements.

</details>

**26. Can Unsupervised Pre-Training Completely Replace Supervised Contrastive Learning? Where Are Their Respective Performance Ceilings?**

<details>
<summary>💡 Show Answer</summary>

This is a common practical question when designing embedding model training workflows. Combining our previous discussions, a comprehensive analysis can be provided:

- **Performance Ceiling of Unsupervised Pre-Training**: Unsupervised methods (TSDAE, unsupervised SimCSE, etc.) derive training signals entirely from automatically constructed proxy tasks (reconstruction, dropout noise, etc.) rather than direct human/high-quality annotated supervision targeting "semantic relevance." Consequently, the quality of learned representations faces a clear upper bound. Extensive empirical studies (including comparative experiments in papers like SimCSE) show that even the best-performing unsupervised methods usually lag significantly behind models trained with moderate amounts of high-quality annotated data (e.g., supervised SimCSE leveraging NLI data).
- **Performance Ceiling of Supervised Contrastive Learning**: The performance ceiling of supervised methods is primarily constrained by the **scale, quality, and diversity of semantic relationships covered by annotated data**. When data conditions permit, supervised contrastive learning (especially when combined with hard negative mining and multi-stage training paradigms) remains the primary path to top-tier performance. Almost all state-of-the-art embedding models on public benchmarks rely to some degree on supervised or high-quality weakly supervised signals rather than pure unsupervised training.
- **Why Unsupervised Pre-Training Cannot Fully Replace Supervised Learning**: Fundamentally, the concept of "semantic relevance" carries rich and context-dependent definitions across different application scenarios (as discussed with instruction-aware models, where different tasks define "similarity" differently). Such nuanced semantic relationship judgment can hardly be implicitly learned purely through automatically constructed proxy tasks (reconstruction, noise invariance, etc.). Direct human annotation or high-quality weak supervision remains the most direct and reliable medium for imparting fine-grained semantic relationship knowledge.
- **Accurate Positioning of Their Relationship**: Based on the above analysis, unsupervised pre-training is more accurately positioned as "the optimal feasible solution when annotated data is lacking" and "infrastructure offering a better starting point for subsequent supervised fine-tuning," rather than a complete substitute for supervised learning. Their relationship is one of **complementarity and progression** rather than mutual exclusion---which is why this chapter repeatedly emphasizes the two-stage pipeline of "unsupervised pre-training + supervised fine-tuning."

In summary, unsupervised pre-training cannot completely replace supervised contrastive learning, as a clear performance gap exists between their ceilings. Empirical evidence indicates that methods incorporating annotated or high-quality weakly supervised signals still hold a distinct advantage in maximum performance potential. Unsupervised pre-training is more accurately characterized as a viable solution in data-constrained scenarios and a foundational stepping stone prior to supervised fine-tuning. The two should be understood as complementary and progressive rather than mutually exclusive choices, explaining why mainstream embedding model training pipelines across the industry universally adopt the combined paradigm of "unsupervised/weakly-supervised pre-training + supervised fine-tuning."

</details>

**27. In Practice, How Should One Select Suitable Unsupervised Pre-Training Methods and Technical Combinations Based on Data Characteristics, Target Domain, and Available Compute Resources?**

<details>
<summary>💡 Show Answer</summary>

Synthesizing the unsupervised pre-training techniques discussed in this section (TSDAE, SimCSE, ICT, GPL, etc.), below is a reference decision-making framework for practical use:

- **Step 1: Evaluate Data Availability**: Determine what raw data can be obtained in the target domain---whether large-scale unlabeled text exists (deciding if unsupervised pre-training is feasible), whether natural long-document structure exists (deciding if ICT-like methods are suitable), and whether even small amounts of annotated/weakly supervised data can be acquired (deciding subsequent fine-tuning strategies).
- **Step 2: Assess Disparity Between Target Domain and General Corpora**: If target domain text exhibits minor vocabulary or syntactic differences from general pre-training corpora (e.g., general commercial text), using off-the-shelf general embedding models or lightweight fine-tuning first is recommended. If disparities are substantial (e.g., vertical domains with heavy specialized jargon), investing in domain-adaptation pre-training such as TSDAE becomes essential, alongside lower-level adaptation like vocabulary expansion if needed.
- **Step 3: Select Specific Methods Based on Data Structure Characteristics**: If the corpus predominantly comprises scattered short texts, prioritize TSDAE or SimCSE. If the corpus contains longer, structurally complete documents, ICT-style positive pair construction can be additionally incorporated, or both can be combined to cover a broader array of semantic relationship types.
- **Step 4: Evaluate Compute Resources and Time Budgets**: As discussed, TSDAE training involves additional decoder computational overhead and typically requires longer training times. If compute resources or project schedules are strictly limited, lighter alternatives (such as off-the-shelf multilingual/multi-domain pre-trained models + PEFT fine-tuning on small data) can be considered instead of a full unsupervised pre-training workflow.
- **Step 5: Plan the Transition to Subsequent Supervised Fine-Tuning**: Regardless of the chosen unsupervised pre-training method, project planning should clarify early on whether annotated/weakly supervised data can be obtained for further fine-tuning. Automated pseudo-labeling schemes like GPL should be planned in advance as contingencies, rather than treating unsupervised pre-training as the end of the line.

In summary, choosing the right combination of unsupervised pre-training techniques has no one-size-fits-all solution. Instead, it requires a comprehensive assessment across multiple dimensions: data availability, domain disparity from general corpora, corpus structural traits, and actual compute/time budgets. A tailored technical path should be selected accordingly, with unsupervised pre-training positioned as a bridge connecting stages within the overall training pipeline, rather than an isolated ultimate goal.

</details>

