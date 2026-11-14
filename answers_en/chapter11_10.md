# 11.10 Native Multimodal — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. What is the essential design difference between native multimodal architectures and "encoder-spliced" multimodal architectures?**

<details>
<summary>💡 Show Answer</summary>

"Encoder-spliced" architectures (such as early typical approaches like LLaVA) usually employ an independently pretrained visual encoder (such as CLIP) to encode an image into a set of feature vectors, and then use a projection layer to "translate" and map these features into the word embedding space of a language model, prepending or appending them to the text token sequence. During training/inference, the main body of the language model essentially treats these visual features as "special text tokens".

**Differences in native multimodality:**
- **Joint training from scratch rather than splicing and reuse**: Native multimodal models are jointly trained on unified network architectures using text, image (and sometimes audio/video) data right from the pre-training phase. There is no sequential order of "train separately first, then splice together." Visual and language representations are co-shaped within the same optimization process;
- **Shared rather than isolated attention computation**: Information from different modalities shares the same set of attention mechanisms for computation across most layers inside the network, unlike spliced architectures where self-attention inside the visual encoder and self-attention inside the language model are two relatively independent computational processes that interact only at a "narrow interface" (the projection layer);
- **Core significance**: In spliced architectures, visual information must first be compressed and translated into a format that the language model can "understand." This translation process itself acts as an information bottleneck and a potential source of distortion (which is also the "modality gap" issue mentioned in earlier discussions on multimodal hallucinations). Native multimodality attempts to eliminate this artificial translation bottleneck from the architectural design, allowing the model to directly learn cross-modal semantic associations within a shared space.



![编码器拼接与原生多模态架构的结构差异对比 (Structural difference between encoder-spliced and native multimodal architectures.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig136_native_multimodal.png)

</details>

**2. How do native multimodal models achieve "unified semantic space" training?**

<details>
<summary>💡 Show Answer</summary>

The core challenge in achieving a unified semantic space lies in the fact that text is a discrete symbolic sequence, while raw signals such as images and audio are continuous high-dimensional data. Suitable approaches must be designed so that both can be processed under the same network architecture and the same tokenization mechanism.

**Common implementation paths:**
- **Unified discretizing tokenizers**: Train specialized visual/audio tokenizers to split images into patches or audio into frames, and map them via discrete encoding methods similar to VQ-VAE into discrete "visual tokens"/"audio tokens" analogous to text vocabularies. This enables images and text to be represented in the same data structure (discrete token sequences), thereby reusing a unified Transformer architecture for joint processing;
- **Shared positional embeddings and attention space**: Tokens from different modalities are interleaved in the same sequence. Through unified (or adapted/extended) positional encoding mechanisms, the model can freely establish cross-modal associations between text tokens and visual/audio tokens within a single attention computation process;
- **Joint multi-task pre-training objectives**: Training objectives typically mix various tasks, such as text-only language modeling, image-text alignment, and image/audio generation or reconstruction. Through multi-task joint optimization, the model is compelled to learn truly cross-modally unified semantic representations in a shared parameter space that satisfy these diverse goals simultaneously, rather than learning independent, mutually unrelated representation subspaces for each modality.

</details>

**3. What unique challenges are faced in constructing training data for native multimodal models?**

<details>
<summary>💡 Show Answer</summary>

- **Scarcity of high-quality image-text/audio-text alignment data**: In web-scraped image-text pairs, text descriptions are often incomplete, inaccurate, or weakly correlated with the image content (e.g., web text surrounding an image may be irrelevant ads or navigation links). Constructing high-quality, fine-grained aligned image-text data is costly, typically requiring a combination of automated quality filtering and fine-grained annotation assisted by humans or AI models;
- **Scale imbalance across multimodal data**: Compared to text data, high-quality image, audio, and video data still lag behind significantly in overall scale and diversity. How to properly balance the proportions of different modalities during training—avoiding insufficient training due to a lack of one modality or squeezing out the learning space of other modalities due to an excess of another—is a core difficulty in data engineering;
- **Training and data dependence of discretizing tokenizers**: As mentioned earlier, building a unified visual/audio tokenizer itself requires pre-training on large-scale, high-quality data of the corresponding modality. The quality of the tokenizer directly sets the upper limit for downstream joint training performance. If the tokenizer suffers from information compression loss or bias, all subsequent training based on this tokenizer will inherit this limitation;
- **Difficulty of automated cross-modal consistency verification**: Bulk-constructed multimodal training data cannot easily undergo quality control via simple rules or classifiers as text-only data can. Determining whether "this text accurately describes this image" is itself a complex multimodal understanding task, making it easy to fall into the circular dependency problem of "what tool to use to verify multimodal data quality."

</details>

**4. What common design patterns exist for cross-modal attention sharing mechanisms in concrete implementations?**

<details>
<summary>💡 Show Answer</summary>

- **Full sharing**: Tokens from all modalities are mixed into the same sequence, sharing the exact same set of attention and feed-forward network parameters. The model does not differentiate which modality a token originally belongs to when processing it. This design maximizes parameter sharing and cross-modal information flow, but places higher demands on the model's ability to learn how to properly handle distinct modality characteristics;
- **Modality-specific adapter layers + shared backbone**: While retaining shared parameters for the majority of the network, a small number of dedicated adapter components are designed for different modalities (such as modality-specific normalization layer parameters or dedicated feed-forward network branches). This maintains the capacity to learn a shared global semantic space while preserving flexibility to handle modality-specific statistical characteristics, offering a trade-off between "full sharing" and "completely independent encoders";
- **Cross-modal attention routing (Mixture-of-Modality-Experts approach)**: Drawing on the ideas of Mixture-of-Experts (MoE), tokens of different modalities are routed more inclined toward expert subnetworks "specialized" in handling modality-specific features, while retaining shared experts to process general cross-modal semantic associations. This aims to strike a finer balance between parameter efficiency and modality adaptability.

</details>

**5. In which specific tasks do native multimodal architectures demonstrate the most pronounced advantages compared to spliced architectures?**

<details>
<summary>💡 Show Answer</summary>

- **Fine-grained visual grounding and counting tasks**: Tasks requiring precise spatial information and detailed counting (such as "How many birds are in the image?" or "Locate all red objects in the image") show superior accuracy theoretically and practically in native multimodality due to reduced information compression loss compared to spliced architectures;
- **Complex cross-modal reasoning tasks**: Tasks requiring deep reasoning by integrating both visual details and linguistic commonsense simultaneously (e.g., "Infer the likely season based on the weather conditions and clothing in the image, and provide a rational justification"). Because visual and linguistic representations co-evolve throughout training in native multimodal models, they generally display more natural and tightly integrated cross-modal reasoning capabilities, rather than a fragmented processing pattern of "first roughly understanding the image, then applying language model commonsense for secondary processing";
- **Understanding text/structured content in images**: When handling "mixed text-image" content such as text and structural information in documents, charts, or UI screenshots, native multimodal architectures often exhibit better consistency than spliced architectures. This is because the visual encoders in spliced architectures are typically not specially optimized during pre-training for tasks like "precisely recognizing text content in images."

</details>

**6. What are the detection methods for multimodal hallucinations? Can native multimodal architectures fundamentally reduce hallucination rates?**

<details>
<summary>💡 Show Answer</summary>

**Detection methods:**
- **Consistency cross-verification**: Querying the model multiple times about the same image using different angles or phrasing to check whether its answers contradict each other. Contradictory answers often imply the presence of hallucinations;
- **Visual evidence grounding**: Requiring the model to explicitly point out the specific regions in the image corresponding to its descriptive conclusions (such as via bounding box annotations). If the model fails to provide reasonable visual grounding, the credibility of the conclusion is discounted;
- **Combining representation engineering techniques**: Analyzing whether the model's internal attention truly focuses on relevant regions of the image when generating a specific description, or relies more on language-level statistical priors. This shares the same methodology as "analyzing internal activation patterns" in text-only hallucination detection.
**Impact of native multimodality on hallucination rates:** Because native multimodality eliminates the information loss in the step of "translating and compressing visual information into a form understandable by the language model" present in spliced architectures, theoretical arguments and some empirical results support that native multimodal architectures produce relatively fewer hallucinations on fine-grained visual tasks. However, it must be noted that another major cause of hallucination—an overly strong language prior suppressing visual evidence itself—is an issue at the level of training data distribution and optimization objectives. It does not automatically vanish simply by switching from a spliced architecture to a native multimodal one. Thus, native multimodality primarily **mitigates** a class of architectural causes of hallucinations, rather than fundamentally and comprehensively eliminating the hallucination problem.

</details>

**7. What specific advantages does native speech multimodality have in preserving paralinguistic information compared to the cascaded "ASR + LLM + TTS" approach?**

<details>
<summary>💡 Show Answer</summary>

The overall advantages of native speech multimodality were discussed earlier in the Multimodal and Embodied AI section. Here, we focus specifically on the dimension of "preserving paralinguistic information":

- **Intonation and emotional information**: A speaker's pitch inflections and emotional state (e.g., anger, hesitation, excitement) are carried in the pitch and energy variation patterns of the audio signal. Once ASR transcribes this into plain text, these signals are completely lost. Native speech models directly process continuous or discretized audio features, enabling such information to be preserved and passed through to subsequent understanding and generation stages—for instance, appropriately matching or responding to the user's current emotional state when generating spoken responses;
- **Semantic functions of pauses and stress**: The exact same text sentence can convey subtle yet crucial differences in meaning depending on which words are stressed and where pauses occur (such as contextual nuances implied by emphasizing "it is *me*" versus "*it* is me"). Such details are likewise lost when transcribed into plain text, whereas native speech understanding can capture these nuances and integrate them into the comprehension process;
- **Controllability on the generation side**: When generating spoken responses, native speech multimodal models can directly control paralinguistic features such as intonation, speaking rate, and emotion within the audio space. In contrast, TTS modules in cascaded schemes can typically generate speech only based on text content and a limited set of preset style tags, making it difficult to dynamically and finely adjust speech expressiveness according to actual conversational context as native solutions do.



![原生语音模型中的信息保留路径与级联管道中的信息损失对比 (Information preservation path in native speech models compared to information loss in cascaded pipelines.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig142_native_speech.png)

</details>

**8. How is temporal modeling in video understanding typically handled in native multimodal architectures?**

<details>
<summary>💡 Show Answer</summary>

Compared to static images, video introduces an additional time dimension. How to efficiently model the temporal information of long videos within a native multimodal framework presents a distinct technical challenge.

- **Keyframe sampling and compression**: Converting every video frame directly into a complete sequence of visual tokens causes the token count to scale linearly or even faster with video duration, quickly exceeding the context length manageable by the model. Therefore, reasonable sampling strategies (such as dynamic keyframe extraction or scene change detection) or compression strategies (redundancy compression between adjacent frames) are typically needed to retain crucial temporal information while controlling the total token count;
- **Joint spatiotemporal tokenization**: Unlike separately processing the "spatial dimension" (image patches within a single frame) and the "temporal dimension" (relationships across frames), some approaches design joint spatiotemporal tokenizers that treat a short segment of consecutive frames (rather than a single frame) as the basic encoding unit. This enables local short-term motion details to be compactly encoded into a single token, reducing the total token count while retaining motion information;
- **Hierarchical temporal modeling**: Borrowing ideas from long-text processing ("local attention + global summarization"), video understanding can adopt a similar hierarchical design—fine-grained temporal modeling is performed only within local time windows (ensuring the accuracy of motion details), while long-range associations are established globally using coarser, compressed representations, balancing computational efficiency and long video comprehension capability.

</details>

**9. What difficulties are faced in designing evaluation benchmarks for native multimodal models?**

<details>
<summary>💡 Show Answer</summary>

- **Decoupled evaluation of fine-grained capabilities**: A failure in a multimodal task may stem from inaccurate visual perception, linguistic misunderstanding, or errors in cross-modal reasoning. A good evaluation benchmark needs to disentangle these potential failure modes (e.g., by simultaneously providing controlled tests for text-only subtasks, vision-only subtasks, and cross-modal fusion subtasks) rather than offering only a vague final task accuracy; otherwise, it is difficult to pinpoint the model's specific shortcomings;
- **Avoiding "cheating" via language priors**: Many questions that seemingly require visual understanding can actually be answered with high probability using only linguistic commonsense (e.g., "What color is the sky in the image?"—answering "blue" without looking at the image yields a high hit rate). Benchmark design must deliberately avoid question types that allow language priors to serve as "shortcuts," thereby truly measuring the model's actual reliance on visual evidence;
- **Objectivity in evaluating open-ended generation tasks**: Open-ended responses in image captioning or visual question answering lack a single standard ground truth. Evaluation often relies on human scoring or employing another LLM as a judge. Both methods suffer from limitations such as high cost, subjectivity, and potential biases in judge models affecting the objectivity of results;
- **Persistence of benchmark contamination**: Similar to text-only benchmarks, as multimodal training data scales up, public multimodal evaluation benchmarks face the risk of being inadvertently or intentionally included in training sets, causing artificially inflated evaluation scores that fail to reflect true generalization capabilities.

</details>

**10. Across which dimensions must trade-offs be made in the design of multimodal tokenizers?**

<details>
<summary>💡 Show Answer</summary>

- **Compression rate vs. information fidelity**: Tokenizers compress continuous visual/audio signals into a finite set of discrete tokens. Higher compression rates (representing the same content with fewer tokens) yield higher processing efficiency and support longer context windows, but lead to greater information loss—potentially losing fine-grained textures, exact spatial positions, and other details, directly impacting performance on fine-grained visual tasks as discussed earlier;
- **Vocabulary size vs. semantic expressiveness**: The size of the "vocabulary" for discretized visual/audio tokens resembles that of text tokenizers. A larger vocabulary theoretically allows a single token to carry richer semantic information, but an excessively large vocabulary increases embedding layer parameter count and training difficulty (more categories mean relatively sparser training signals per category);
- **Generality vs. task specialization**: Visual tokenizers specifically trained for a particular downstream task (such as medical imaging analysis) may perform better on that task, but offer poorer generality and cross-task reusability. Tokenizers pursuing generality need to be trained on broader, more diverse data, which may render them less fine-grained in certain specialized niche scenarios than dedicated solutions;
- **Compatibility with generation tasks**: If a model must not only understand visual inputs but also generate image/audio outputs (such as native multimodal image generation), the tokenizer design must simultaneously satisfy the additional constraint of "whether the post-encoding representation facilitates autoregressive generation." This often imposes extra requirements on vocabulary structure and discretization methods that do not perfectly align with the optimal choices for pure comprehension tasks, requiring further trade-offs.

</details>

**11. In what aspects is the cross-modal generalization/transfer capability of native multimodal models reflected?**

<details>
<summary>💡 Show Answer</summary>

Cross-modal generalization refers to a model's ability to effectively transfer and apply knowledge or capabilities acquired from one modality to tasks involving another modality. This serves as an important indicator of whether "unified semantic space" training is truly successful.

- **Explaining visual anomalies using textual knowledge**: For example, when a model observes an abnormal shadow on an X-ray, it can leverage disease knowledge learned from vast text corpora to provide a reasonable medical explanation for the visual observation. This capability reflects effective association between text-only knowledge and visual perception within a shared semantic space, rather than treating the two capacities as isolated;
- **Visual evidence assisting in correcting erroneous language priors**: As noted earlier, language priors can sometimes lead models to give "taken-for-granted" answers that contradict actual visual content. A model with strong cross-modal generalization should exhibit a stronger tendency to "rely on visual evidence to correct flawed language priors," rather than letting language priors dominate the output;
- **Few-shot cross-modal analogical reasoning**: Presenting the model with a few novel image-text paired analogical examples (such as a previously unseen chart type and its interpretation) to observe whether it can rapidly transfer this newly learned image-text correlation pattern to new samples with similar structures but different content. This few-shot cross-modal analogical capability is considered a crucial metric for evaluating whether a model has established genuine, general cross-modal reasoning capabilities (rather than relying on memorization of specific training distributions).

</details>

**12. What additional computational costs does native multimodal training entail compared to text-only pre-training? How can they be optimized?**

<details>
<summary>💡 Show Answer</summary>

- **Significant increase in visual/audio token sequence length**: Even after compression and sampling optimization, the number of tokens converted from images and videos is usually much higher than that of text with equivalent information content. This directly leads to a drastic increase in the effective sequence length the model must process. Because the computational complexity of Transformer self-attention scales quadratically with sequence length, this is a core reason for the elevated cost of native multimodal training;
- **Storage and throughput bottlenecks for multimodal data**: The storage and reading bandwidth demands for raw image, audio, and video data are vastly higher than those for text. Data loading pipelines during training require specialized optimization; otherwise, GPU compute resources can easily stall while waiting for data I/O;
- **Optimization directions**: Combining the efficiency optimization techniques discussed earlier—such as native sparse attention and FlashAttention-like I/O-aware optimizations—becomes even more valuable when dealing with long visual/audio token sequences. Additionally, targeted video/audio compression strategies like keyframe sampling and joint spatiotemporal tokenization are key methods for keeping native multimodal training costs manageable. Furthermore, MoE architectures are frequently employed in native multimodal LLMs because they expand total model capacity without significantly increasing active computation, thereby balancing multimodal capabilities with training/inference costs.

</details>

**13. In what aspects are native multimodality and embodied AI combined?**

<details>
<summary>💡 Show Answer</summary>

- **Directly providing a unified representation foundation for VLA models**: In the Vision-Language-Action (VLA) models discussed earlier, if the visual and language comprehension modules are built upon native multimodal architectures, they achieve more precise and consistent cross-modal representations than spliced architectures. This directly benefits action decision accuracy, especially in manipulation tasks requiring fine spatial localization (e.g., "pick up the object at this specific position");
- **Native multimodal models naturally support incorporating new modalities such as tactile and force sensing**: If a model architecture is inherently designed to "process multiple modalities in a unified manner," compared to spliced architectures that require separate encoder designs and custom integration for each new modality, native multimodal architectures theoretically possess superior architectural consistency and scalability when extending to support new modalities required by embodied AI, such as tactile and force signals;
- **Representation sharing potential with world models**: In embodied AI tasks, visual observations, linguistic instructions, and state predictions (the core capability of world models) fundamentally rely on a shared semantic understanding of the physical world. If native multimodality can be further integrated with the state prediction capabilities of world models, it promises an integrated embodied AI foundation model capable of "understanding instructions, perceiving environments, predicting consequences, and executing actions"—representing one of the most promising yet engineering-challenging integration directions in current embodied AI research.

</details>

**14. What are the design ideas of cross-modal alignment frameworks for defending against multimodal jailbreak attacks?**

<details>
<summary>💡 Show Answer</summary>

Multimodal models introduce a new attack surface not present in text-only models—attackers can hide harmful instructions inside images (such as embedding text into an image or utilizing visual content to mislead the model), thereby bypassing safety filters designed for plain text.

- **Cross-modal consistency verification**: Designing specialized verification mechanisms to check for suspicious semantic inconsistencies between image content and user-provided text instructions (e.g., the text claims "this is an ordinary landscape photo," but the actual image content or embedded text implies a completely different, potentially harmful intent). Such inconsistencies often signal a disguised attack;
- **Detection of hidden text in images**: Because attackers may embed harmful instructions as visual text inside images rather than typing them into text boxes, safety mechanisms must cover the recognition and auditing of text within images. This text must be subjected to safety filtering workflows just as rigorous as those for text inputs, rather than auditing only the user-provided text portion;
- **Unified, all-modality safety representation monitoring**: Combining the representation engineering concepts mentioned earlier, within native multimodal models, key safety-related semantic directions should ideally be shared and consistent across modalities (whether a harmful intent is expressed via text or image, it should activate similar internal "alertness" representations). Such safety mechanism designs are more thorough than building separate filters for each modality and are harder to bypass by simply "switching modalities of expression."



![用于检测和阻断多模态越狱输入的跨模态对齐防御框架 (Cross-modal alignment framework for detecting and blocking multimodal jailbreak inputs.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig149_cross_modal_alignment.png)

</details>

**15. What additional requirements does the image/audio generation capability (rather than comprehension alone) of native multimodal models impose on architectural design?**

<details>
<summary>💡 Show Answer</summary>

If a native multimodal model must not only comprehend visual/audio inputs but also generate outputs in corresponding modalities (such as generating images from text descriptions or generating voice replies from conversational context), the architectural design needs to additionally account for the unique characteristics of generation tasks:

- **Choice between autoregressive and parallel generation**: Continuing the language model paradigm of token-by-token autoregressive generation to synthesize visual/audio tokens allows the reuse of a unified architecture and training objective, but results in slower generation speeds (especially given the large number of tokens required for image generation). Other approaches introduce the parallel denoising concepts of diffusion models into the generation phase, compromising slightly on architectural unity to gain generation efficiency, resulting in a hybrid architecture of "autoregressive comprehension + diffusion generation";
- **Dual optimization objectives of generation quality and semantic consistency**: Generation tasks require not only high quality of the output content itself (photorealistic images, natural audio) but also consistency between the generated content and the input language instructions or conversational context. This requires training objectives to simultaneously balance content quality and cross-modal consistency, making the optimization objective more complex than that of pure understanding tasks;
- **Trade-offs between inference latency and user experience**: The inference latency of generative multimodal outputs (especially images and video) is typically significantly higher than that of text-only generation. Product deployment requires designing appropriate interactive experiences to handle this latency (such as streaming progressive generation progress or offering previews of intermediate results), rather than naively copying the "real-time streaming output" interaction paradigm of text-only chat.

</details>

