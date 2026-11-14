# 12.1 Multimodal Fundamentals and Core Concepts — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 17 questions

**1. What is a Multimodal Large Model?**

<details>
<summary>💡 Show Answer</summary>

A Multimodal Large Model refers to a large-scale neural network model capable of simultaneously understanding and/or generating information across multiple modalities (such as text, images, audio, video, etc.). Unlike traditional large language models that can only process a single modality (such as pure text), the inputs or outputs of multimodal large models can span different forms of information representation: for example, a typical vision-language model (VLM) can simultaneously receive an image and a text instruction as input, understand the combined semantics of both, and generate a corresponding textual response (such as describing the image content or answering questions about the image); further advanced models (such as text-to-image models and native multimodal models) can also directly generate non-textual content such as images, speech, or even videos. The core value of multimodal large models lies in endowing models with capabilities closer to humans in comprehensively utilizing multiple sensory modalities of information for understanding and communication, rather than being limited to a single textual information channel.

</details>

**2. What are Common Multimodal Tasks?**

<details>
<summary>💡 Show Answer</summary>

Multimodal large models can handle a wide variety of tasks, which can be broadly categorized according to combinations of input and output modalities into the following types:

- **Image Understanding**: Image captioning (generating a textual description given an image), visual question answering (VQA, generating an answer given an image and a question), image-text retrieval (retrieving the most relevant content from an image or text database for a given query), and visual grounding (locating specific objects or regions in an image);
- **Generation**: Text-to-image (generating corresponding images based on textual descriptions), text-to-video, and image editing (modifying parts of an existing image based on instructions);
- **Speech/Audio**: Speech recognition (converting speech to text), speech synthesis (converting text to speech), and spoken dialogue (direct multi-turn interaction in speech format);
- **Document and Structured Visual Information Understanding**: Document/table/chart understanding (combining OCR with layout structure to understand document content) and scene text recognition;
- **Video Understanding**: Video captioning, video question answering, and temporal grounding (locating the time interval where a specific event occurs in a video).
In practical applications, a powerful multimodal large model can often perform multiple tasks listed above simultaneously through a unified natural language interaction interface, without needing to train separate specialized models for each task.

</details>

**3. What are the Core Components of a Standard Vision-Language Model (VLM) Architecture?**

<details>
<summary>💡 Show Answer</summary>

The basic architecture of mainstream VLMs typically consists of three core components:

1. **Vision Encoder**: Usually based on the ViT (Vision Transformer) architecture, it is responsible for splitting the input image into patches and encoding them into a series of feature vectors to extract visual semantic information from the image, serving as the perceptual entry point for the model to "see and understand" images;
2. **Connector / Projector**: Responsible for "translating"/mapping the features output by the vision encoder from the visual representation space into a representation space understandable by the language model. Common implementations include simple linear projection layers, multilayer perceptrons (MLPs), or more complex modules such as Q-Former and Perceiver Resampler, acting as the bridge connecting the visual and language modalities;
3. **Language Model (LLM)**: Receives the complete sequence formed by concatenating the visual tokens output by the connector with the user input text tokens, utilizing its powerful language understanding and generation capabilities to reason in combination with visual information and generate the final textual answer.
Among these three components, the vision encoder and language model are usually mature models fully pretrained on their respective unimodal data, whereas the connector module is often an added part that needs to be specifically trained for multimodal tasks (or even learned from random initialization). This is also the core focus of the "Modality Alignment and Fusion Mechanisms" discussed later in this chapter.

</details>

**4. What is ViT (Vision Transformer)?**

<details>
<summary>💡 Show Answer</summary>

ViT (Vision Transformer) is a pioneering work that directly applies the Transformer architecture to image processing tasks. Its core idea is to transform an image into a sequence format that a Transformer can process: first, the input image is divided into a sequence of fixed-size image patches (e.g., 16×16 pixels per patch). Each patch is flattened into a one-dimensional vector and then mapped through a linear projection layer into a vector representation that matches the hidden dimension of the Transformer (conceptually similar to the word embedding process in NLP, which is why each patch is also commonly called a "visual token"); next, positional embeddings are added to each patch token (since self-attention itself lacks position awareness and requires explicit positional information); finally, all patch tokens (sometimes with an additional learnable CLS token concatenated) are fed into a standard Transformer encoder architecture (consisting of stacked self-attention and feed-forward networks) for processing. The introduction of ViT proved that the Transformer architecture, originally designed for processing one-dimensional text sequences, can be directly transferred and applied to two-dimensional image data, achieving performance superior to traditional Convolutional Neural Networks (CNNs) under large-scale data pretraining, laying the foundation for the architectural design of almost all mainstream vision encoders that followed.

</details>

**5. What Preprocessing Steps are Typically Required Before Images are Fed into a Multimodal Large Model?**

<details>
<summary>💡 Show Answer</summary>

Before image data is fed into a vision encoder, it typically needs to undergo the following preprocessing steps:

1. **Rescaling/Cropping to a fixed (or candidate) resolution**: Since vision encoders are usually pretrained on a specific resolution or a limited set of resolutions, the input image must first be resized (aspect-ratio scaled) or cropped to match the target size. Some models that support dynamic resolutions select the most suitable target resolution based on a predefined set of candidate aspect ratios;
2. **Normalization**: Standardizing the image pixel values from their original integer range of 0–255 using the mean and variance used during vision encoder pretraining, converting them into floating-point values with a mean close to 0 and a variance close to 1, which helps stabilize neural network training and inference;
3. **Patch extraction**: Splitting the preprocessed image into a sequence of image patches according to the patch size set by the vision encoder, preparing for subsequent patch-wise encoding;
4. **Data augmentation (training phase only)**: During training, data augmentation operations such as random cropping, flipping, and color jittering are typically applied to improve model robustness and generalization (such augmentations are usually omitted during inference);
5. **Tiling processing for ultra-large images**: For ultra-large images whose resolutions far exceed the standard processing capacity of the vision encoder, an additional dynamic tiling strategy is required to divide the image into multiple sub-patches to be processed separately (see the related discussion in the section "Visual Encoding and Input Processing" of this chapter).



![视觉语言模型中的标准图像预处理流程图 (Standard image preprocessing pipeline for Vision-Language Models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig01_image_preprocessing.png)

</details>

**6. What is Image Tokenization?**

<details>
<summary>💡 Show Answer</summary>

Image Tokenization refers to converting continuous image pixel signals into a sequence of discrete or continuous "token" representations, allowing image data to be processed uniformly by Transformer architectures in a manner similar to text token sequences. There are two main technical routes:

- **Continuous Representation** (Adopted by mainstream VLMs): After dividing the image into patches, each patch is mapped into a continuous high-dimensional vector via linear projection. This representation does not correspond to any discrete symbol in a "vocabulary", but is essentially a feature vector in a continuous space. This is the approach used by current mainstream VLMs such as LLaVA and Qwen-VL;
- **Discrete Representation** (Common in image generation models): Leveraging vector quantization methods such as VQ-VAE (Vector Quantized Variational Autoencoder), a finite-sized "visual codebook" (analogous to a text vocabulary) is first learned, and each local region of the image is mapped to the nearest discrete code index in the codebook. In this way, the image is converted into a truly discrete, text-token-like sequence representation. This method is widely used in autoregressive image generation models (such as DALL-E and VAR), because discrete tokens facilitate reusing the autoregressive generation paradigm of language models.
Both approaches have their strengths and weaknesses: continuous representation preserves information more completely with higher precision, making it more suitable for understanding tasks that require fine-grained analysis of image content; discrete representation, on the other hand, is easier to unify with the autoregressive generation mechanism of language models, although the quantization process itself inherently introduces some information loss.

</details>

**7. Why Can ViT Not Assign a Unique Discrete ID to Each Image Patch Like Text Tokens?**

<details>
<summary>💡 Show Answer</summary>

A text vocabulary is essentially a **finite discrete set**—whether complete words or subwords, every element in the vocabulary corresponds to a unique integer ID, and the model can obtain its vector representation through a simple lookup operation (Embedding Lookup). This is because although the basic units of language are numerous, they are ultimately finite (for instance, common BPE vocabulary sizes typically range from tens of thousands to over one hundred thousand).

In contrast, every image patch is essentially a **continuous high-dimensional pixel signal**—even for a small 16×16 pixel block, the theoretically possible combinations of pixel values constitute an almost infinite continuous space. There is no natural, finite "image patch vocabulary" that can exhaust all possible patch contents and assign a unique discrete ID to each. Therefore, ViT adopts the approach of using a learnable **linear projection layer** to map the raw pixel values of each patch directly into a continuous vector representation (rather than looking up a vector corresponding to a discrete ID from a finite vocabulary). This fundamental difference in processing method represents the most essential distinction between visual and language modalities at the tokenization level—language tokenization is "discrete symbol lookup," whereas visual tokenization is fundamentally "continuous signal encoding and projection." (Some works, such as VQ-VAE, achieve discretization by training an additional "visual codebook", but this itself requires a specialized training process to learn a finite approximate codebook, rather than an intrinsic discrete property naturally possessed by visual data.)

</details>

**8. What is Image-Text Retrieval, and How Does It Differ from VQA?**

<details>
<summary>💡 Show Answer</summary>

**Image-Text Retrieval** refers to retrieving several items whose semantics are most relevant to a given query (which can be either an image or a piece of text) from a large candidate pool (correspondingly, a text database or an image database) and returning them ranked by relevance. Its core technology involves calculating the **cross-modal similarity** between the query and candidates (typically using models trained with contrastive learning, such as CLIP, to encode images and text into vectors separately and calculate cosine similarity). The output of the task is a **ranked list**, and evaluation metrics are typically retrieval metrics such as Recall@K.

**VQA (Visual Question Answering)** is given a specific image and a natural language question about the content of that image; the model needs to understand the image content, perform reasoning in conjunction with the question, and ultimately **generate a piece of text as the answer** (which can be short words or full sentences). Its core is the combination of the model's image understanding and language generation capabilities. The output is **concrete open-ended or multiple-choice answer text**, and evaluation metrics are typically answer accuracy or matching degree with reference answers.

The fundamental difference between the two is: image-text retrieval focuses on "finding the most relevant items among a candidate pool", which is essentially a **ranking/matching** problem that does not involve reasoning and generation for a specific question; VQA, on the other hand, focuses on "providing the correct answer to a specific question", which is essentially a problem requiring **reasoning and generation** combined with visual understanding. The key required model capabilities and evaluation methodologies for the two differ significantly.

</details>

**9. What are the Fundamental Differences Between Multimodal Large Models and Traditional Computer Vision Models (e.g., ResNet Classifiers)?**

<details>
<summary>💡 Show Answer</summary>

Traditional computer vision models (such as ResNet for image classification or Faster R-CNN for object detection) are typically trained for a **single, predefined task**: for example, the output space of an image classification model is a fixed, finite set of categories (such as the 1000 categories in ImageNet). The model can only determine which of these predefined categories an input image belongs to, completely lacking the ability to understand and generate natural language, nor can it handle new categories or tasks that were not defined during training.

Multimodal large models, on the other hand, combine powerful visual understanding capabilities with the reasoning and generation capabilities of language models: their output is **open-ended natural language**, which in theory can describe visual content at any arbitrary level of richness and detail. They can also handle entirely new categories never explicitly seen during training (performing zero-shot reasoning by leveraging the vast common sense and world knowledge stored in language models), execute complex multi-step instructions, and conduct multi-turn interactive conversations. This generality and open-endedness are core advantages that traditional computer vision models entirely lack. Conversely, precisely because multimodal large models pursue generality rather than extreme optimization for a single task, their accuracy and inference speed on certain specialized tasks with clear boundaries (such as high-precision object detection in specific scenarios) still often lag behind specially optimized traditional vision models. This is also why some current practical applications still adopt a hybrid architecture of "multimodal large model for understanding and decision-making + traditional vision model for specific perception."



![传统计算机视觉模型与多模态大模型的对比图 (Comparison between traditional computer vision models and multimodal large models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig02_mllm_vs_traditional_cv.png)

</details>

**10. Why is "Language" Considered the Core Hub for Achieving Unified Representation in Current Multimodal Large Models?**

<details>
<summary>💡 Show Answer</summary>

Current mainstream multimodal large model architectures generally choose the **language model** as the core backbone of the entire system. There are several important reasons behind this design choice:

- **Language possesses high abstraction and compositionality**: Natural language can describe almost any information content that humans can perceive and understand—whether objects, scenes, and actions in images, event developments in videos, or semantic content in audio, all can be re-stated and expressed relatively completely using language. This "universal expressive power" makes language naturally suited as an intermediate medium for converging and interacting information from different modalities;
- **Pretrained language models already possess powerful general capabilities**: Language models pretrained on massive text data have already internalized vast amounts of world knowledge, logical reasoning capabilities, and instruction-following abilities. If information from other modalities can be successfully "translated" into representations understandable by the language model, these pretrained powerful capabilities can be directly reused without needing to train a brand-new backbone network that treats multiple modalities equally from scratch;
- **Convenience in engineering implementation**: Centering around the language model means being able to directly reuse highly mature model architectures, training methods, and inference acceleration techniques (such as KV Cache) from the NLP field. One only needs to train an additional relatively lightweight "translation" module (connector) to quickly construct a system with multimodal capabilities. This is far more economical in engineering and training costs than designing from scratch a truly symmetric architecture that treats all modalities equally.
For these reasons, almost all mainstream multimodal large model architectures today are essentially different implementations of the paradigm: "using the language model as the backbone, and finding ways to 'translate' other modalities into a representation space understandable by the language model," rather than architectures where all modalities genuinely hold completely equal status.

</details>

**11. What is the Difference Between Multimodal Large Models and Multimodal Agents?**

<details>
<summary>💡 Show Answer</summary>

A multimodal large model itself is a foundation model capable of image-text understanding and/or generation, with its capabilities bounded within a "single invocation"—the user inputs images and/or text, and the model outputs text (or images). The interaction format is relatively static single-turn Q&A or multi-turn dialogue, and the model itself lacks the capacity to actively perceive environmental changes, continuously execute tasks, or invoke external tools.

A Multimodal Agent, on the other hand, uses the multimodal large model as its "brain" and wraps it in a complete engineering system supporting autonomous action. This typically includes: **Perception Module** (continuously obtaining environmental state, such as screenshots or sensor data); **Planning Module** (decomposing complex tasks into executable sequences of sub-steps); **Tool Use / Execution Module** (translating model decisions into concrete operations on external systems, such as clicking buttons, calling APIs, or controlling robotic arms); and **Memory Module** (retaining historical information across multiple turns or even sessions to avoid starting from scratch each time). Therefore, one can say: a multimodal large model is a static capability provider that "can see and understand", while a multimodal agent superimposes a "perception-decision-action" closed loop on top of it, emphasizing autonomy and task completion rate rather than the correctness of a single Q&A turn. For instance, the same VLM can be used as a foundation model for "image-based Q&A" or wrapped into an agent system that can autonomously browse web pages and complete shopping orders.

</details>

**12. What is a "Modality"? How are the Boundaries of Modalities Defined?**

<details>
<summary>💡 Show Answer</summary>

A modality typically refers to the way information manifests and is perceived in the physical world, such as vision (image/video), audition (speech/music/ambient sound), text (natural language), touch (haptic feedback), etc. The fundamental differences between different modalities are reflected across multiple dimensions, including data structure (images are two-dimensional/three-dimensional continuous numerical matrices, text is a one-dimensional discrete symbol sequence, audio is a one-dimensional continuous waveform), collection methods (cameras, microphones, keyboard inputs), and statistical properties (local image pixels are highly correlated, whereas long-range semantic dependencies exist between words in text).

The boundaries of modalities are not absolutely distinct, but rather represent a practical judgment of "whether specialized encoding methods and modeling assumptions are required":
- Although code is written in pure text, its strict syntactic structure and executability lead it to be viewed in some works as a special sub-modality between "natural language text" and a "symbolic formal system," often requiring specialized tokenizers or structure-aware modeling approaches;
- Structured data such as tables and charts contain both visual rendering forms (as image inputs) and underlying numerical/textual contents; thus, they are often regarded as hybrid text-vision modalities requiring both pixel-level perceptual capabilities and structured numerical understanding capabilities;
- Sensor signals (such as LiDAR point clouds and haptic feedback signals) are increasingly viewed as independent modalities in embodied AI scenarios, requiring specially designed encoders.
In summary, whether a class of data should be treated as an independent modality hinges on whether its data distribution characteristics differ sufficiently from existing modalities (such as text and images) to warrant specialized representation learning methods.

</details>

**13. What are the Common Open-Source VLMs? Briefly Summarize Their Characteristics.**

<details>
<summary>💡 Show Answer</summary>

- **LLaVA Series**: Features the simplest architecture (pretrained ViT + linear projection layer + pretrained LLM) and uses two-stage training (alignment pretraining + visual instruction tuning). It serves as the open-source baseline with the lowest replication cost and widest secondary development in academia. Subsequent versions LLaVA-1.5 and LLaVA-NeXT (supporting AnyRes dynamic high resolution) continuously iterate to enhance performance.
- **Qwen-VL Series**: Qwen-VL supported fine-grained visual grounding early on (capable of directly outputting bounding box coordinates of objects); Qwen2-VL introduced naive dynamic resolution and M-RoPE positional encoding, supporting arbitrary resolution images and long video understanding, with outstanding performance in Chinese language scenarios.
- **InternVL Series**: Employs a large-scale vision encoder (InternViT, with parameters up to 6B) far exceeding standard ViT-L, combined with a dynamic high-resolution tiling strategy, achieving leading visual representation capabilities across multiple benchmarks.
- **MiniCPM-V**: Optimized for edge deployment (resource-constrained devices like mobile phones), achieving competitive multimodal understanding performance despite having significantly fewer parameters than mainstream large models.
- **CogVLM**: Introduces a "Vision Expert" module, maintaining an independent visual parameter path in every layer of the language model, emphasizing deeper visual-language fusion than simple linear projection.
- **DeepSeek-VL / Janus Series**: The Janus series further explores a unified multimodal architectural concept of using different vision encoders for "understanding" and "generation" tasks.



![主流开源视觉语言模型的分类与架构特点结构图 (Classification and characteristics of mainstream open-source VLMs.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig03_open_source_vlms.png)

</details>

**14. In What Aspects is the "Large" in Multimodal Large Models Reflected?**

<details>
<summary>💡 Show Answer</summary>

In addition to the most intuitive aspect of massive parameter scale (vision encoders reaching billions of parameters and language models reaching tens of billions of parameters), "large" is also reflected in the following dimensions:

1. **Massive scale of training data**: The pretraining phase typically requires covering billions or more image-text pairs, as well as vast amounts of video and audio data, far exceeding the data scale requirements of traditional computer vision models (which usually only require millions of annotated samples).
2. **Strong task generality**: The same model does not need specialized model heads trained separately for each task (such as "image classification", "image captioning", "visual question answering", or "visual reasoning"). Instead, it handles almost any form of vision-language task with the same set of parameters through a unified natural language interface.
3. **Emergent abilities**: When model scale and data scale cross a certain threshold, complex capabilities completely absent in smaller models emerge, such as multi-step visual reasoning, cross-modal analogy, and generalized understanding of instruction combinations not explicitly taught during training.
4. **Vast knowledge capacity**: The language portion of the model implicitly stores massive amounts of world knowledge (history, general science, domain-specific knowledge) during pretraining. This knowledge can be naturally invoked to assist visual understanding and reasoning (such as identifying landmark buildings in an image and answering related questions by combining historical knowledge), a capability completely absent in traditional vision models.

</details>

**15. What are the Fundamental Differences in Information Density and Processing Difficulty Among Image, Video, and Audio Modalities?**

<details>
<summary>💡 Show Answer</summary>

From the perspective of **information density** (the effective semantic information carried per unit of data): text has the highest density, where almost every token corresponds to an explicit semantic unit with very little "redundancy"; images have intermediate density, where despite a huge total number of pixels, neighboring pixels are highly correlated (continuous local textures), and truly independent semantic information is often concentrated in limited areas such as object outlines and key regions; videos and audio have relatively the lowest information density—video superimposes temporal redundancy on top of the spatial redundancy of images (adjacent frames often vary minimally), while audio is a continuous 1D sampled waveform signal that requires a relatively long time window to carry meaningful linguistic or acoustic information.

From the perspective of **processing difficulty**, modalities with lower information density tend to have larger raw data volumes, posing greater challenges in tokenization and sequence length control: an image might be encoded into a few hundred tokens; a few-minute video, if uncompressed, might correspond to tens of thousands or more tokens; similarly, a speech signal requires specialized acoustic feature extraction and compression to be converted into a token sequence of reasonable length. This is precisely why subsequent sections of this chapter will separately discuss patch-based image compression, keyframe sampling and token explosion in video, and audio tokenization.

</details>

**16. What are the Meanings of Zero-shot and Few-shot in Multimodal Scenarios?**

<details>
<summary>💡 Show Answer</summary>

**Zero-shot** means that the model can complete a task directly based solely on task instructions or natural language descriptions of category names, without having seen any training examples for that task or category. The most typical example is CLIP's zero-shot image classification: one only needs to wrap candidate category names into a text template (such as "a photo of a \{class\}") and calculate similarity with image features, enabling the model to make reasonable classifications for categories never seen during training without training a specialized classification head for those categories.

**Few-shot** means providing a small number of image-text demonstrations (e.g., 2–8) of the task in the inference prompt. The model rapidly understands the input-output pattern of the task via **In-Context Learning** (without updating any parameters) and generalizes it to new samples. For example, Flamingo supports inserting multiple pairs of "image-description" examples in a single conversation context; after seeing these examples, even when encountering images of an entirely new style, the model can imitate the description style and level of detail demonstrated in the examples to provide answers. Few-shot typically achieves significant accuracy improvements over zero-shot, but at the cost of consuming longer context lengths.

</details>

**17. What are the Core Challenges of Multimodal Large Models (VLMs)?**

<details>
<summary>💡 Show Answer</summary>

The core challenge of multimodal large models can be summarized as "how to achieve effective alignment and fusion between information from different modalities (especially vision and language)," which can be specifically broken down into the following levels:

1. **Heterogeneity of representation spaces**: Text is a discrete symbol sequence with high semantic density, whereas images are continuous pixel signals with high redundancy. Their statistical properties, information densities, and structural formats are completely different. How to map two heterogeneous signals into a representation space that the model can process uniformly is the most fundamental challenge.
2. **Misalignment of semantic granularity**: A textual description (such as "a cat sitting on a chair") usually corresponds to a visual scene in an image that is far richer than the text (cat's breed, fur color, posture, chair material, etc.). How to preserve key details relevant to linguistic semantics as much as possible while compressing visual information (controlling token count) is a trade-off that must be navigated when designing connector modules.
3. **Imbalance in data scale and quality**: High-quality, finely annotated image-text pair data is far scarcer than pure text data. Web-crawled image-text pairs suffer from widespread noise (inaccurate or incomplete descriptions), directly limiting the upper bound of modality alignment quality.
4. **Trade-off between computational efficiency and information completeness**: Visual signals are naturally more "verbose" than text. If all pixel information is converted to tokens without compression, the sequence length and computational burden expand rapidly (especially for high-resolution images and videos). Consequently, almost all architectural designs (patchification, token compression, Q-Former, etc.) are compromises aimed at balancing efficiency and information completeness.
5. **Hallucination and reliability**: Because the language model component often possesses strong language priors, the model is prone to relying on language habits to "fabricate" content that appears reasonable but does not actually exist when visual evidence is insufficient or compression loss occurs. Ensuring that the model generates answers genuinely grounded in visual inputs rather than relying on language shortcuts is a key challenge for model reliability.
These challenges are interwoven and serve as the starting point for discussions throughout subsequent sections of this chapter, including "Architectural Design", "Modality Alignment", "Training and Data", and "Evaluation".



![视觉语言模型（VLM）面临的核心挑战信息图 (Core challenges in development and deployment of Vision-Language Models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig04_vlm_challenges.png)

</details>

