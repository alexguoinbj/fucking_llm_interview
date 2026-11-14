# 12.2 Vision-Language Model (VLM) Architecture — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 17 questions

**1. What are the Typical Architectural Types of Multimodal Large Models?**

<details>
<summary>💡 Show Answer</summary>

The typical architectural types of multimodal large models mainly include the following:

1. **Dual-Encoder / Contrastive Architecture**: Representative models include CLIP and ALIGN. Images and text are mapped to the same semantic space via independent encoders and aligned through contrastive learning. Suitable for retrieval and zero-shot classification tasks with high inference efficiency, though cross-modal interaction is weak.
2. **Fusion Encoder Architecture**: Representative models include ViLBERT and ALBEF. Image and text features undergo cross-modal interaction (Cross-Attention) in intermediate layers, performing well on understanding tasks but with limited generative capabilities.
3. **Encoder-Decoder Architecture**: Representative models include Flamingo, BLIP-2, and CogVLM. After the vision encoder extracts visual features, they are injected into the language model decoder via an adapter layer (such as Q-Former or linear projection), supporting tasks like image-text generation, VQA, and multi-turn dialogue.
4. **Unified Transformer Architecture**: Representative models include GPT-4o, Gemini, and the LLaVA series. Image patch tokens and text tokens are concatenated and jointly fed into an autoregressive Transformer to achieve unified modeling of vision and text, offering the highest flexibility.
5. **Diffusion-based Architecture**: Representative models include Stable Diffusion and DALL-E 3. Primarily used for image generation, controlling the generation process conditioned on text.

</details>

**2. How is the Multimodal Transformer Structure Typically Designed? What are the Trade-offs Among Early Fusion, Late Fusion, and Cross-Attention Fusion?**

<details>
<summary>💡 Show Answer</summary>

The design focus of multimodal Transformers lies in how to handle token sequences from different modalities. There are three main fusion paradigms:

1. **Early Fusion**: Image patch tokens and text tokens are directly concatenated and fed together into a unified Transformer, allowing modalities to interact freely via Self-Attention (e.g., LLaVA, GPT-4V). The advantage is learning fine-grained cross-modal interactions from shallow layers; the disadvantage is that implementation and training are relatively complex, demanding higher modality alignment.
2. **Late Fusion**: Each modality is encoded by an independent Transformer before fusion, preserving modality specificity (e.g., the dual-stream structure of early ViLBERT). The advantage is being more modular and able to reuse pretrained unimodal models; the disadvantage is weaker fine-grained cross-modal interaction.
3. **Cross-Attention Fusion**: Cross-Attention is used at certain layers for cross-modal interaction, where text serves as Query and image as Key/Value (or vice versa), which is more computationally efficient than pure concatenation (e.g., Flamingo).

Additionally, two engineering details need to be considered:

- **Attention mask design**: Controlling the attention visibility between visual and textual tokens, e.g., image tokens are mutually visible to each other, while text tokens can only attend to preceding tokens (causal masking).
- **Positional encoding design**: Image tokens use 2D positional encodings while text tokens use 1D positional encodings, which are processed uniformly after concatenation.

Current mainstream VLMs (vision encoder + connector + LLM) concatenate visual tokens and text tokens at the input layer of the LLM before uniform processing, which is closer to an "intermediate fusion"—a compromise between efficiency and fusion depth.

</details>

**3. Comparison of VLM Architectural Paradigms: Connector-Based vs. Cross-Attention-Based**

<details>
<summary>💡 Show Answer</summary>

Current mainstream VLM architectural paradigms, categorized by their methods of fusing visual and language information, can be divided into two main categories:

\begin{table}[h]

\caption{Comparison of VLM Architectural Paradigms}


| **Dimension** | **Connector-Based** | **Cross-Attention-Based** |
| --- | --- | --- |
| Representative Models | LLaVA, MiniGPT-4 | Flamingo, BLIP-2 |
| Core Idea | Early alignment, late fusion | Fusion while generating |
| Advantages | High training efficiency, simple architecture | Deeper fusion, strong few-shot capability |
| Disadvantages | Information bottleneck, shallow fusion depth | Complex architecture, high training cost |


\end{table}

**Connector-Based**: Vision encoder extracts features $\to$ connector (linear layer / Q-Former) transforms visual features into fixed-length visual tokens $\to$ concatenated with text tokens and fed into the LLM. Only lightweight connectors need to be trained, allowing reuse of powerful pretrained vision and language models with low cost and simple implementation, demonstrating excellent performance on visual instruction-following tasks; however, visual information is compressed into a fixed number of visual tokens, creating an information bottleneck.

**Cross-Attention-Based**: After the vision encoder extracts features, at each step of text generation, the LLM uses internal Cross-Attention to allow textual Queries to dynamically query visual feature Keys/Values, achieving "fusion while generating." Fusion is deeper, exhibiting outstanding performance in few-shot in-context learning (as validated by Flamingo); however, it requires modifying the internal architecture of the LLM, leading to higher training and inference costs.

Summary: Connector-based architectures represent the current mainstream solution for cost-effective, high-performance VLMs, pursuing efficiency and simplicity; cross-attention-based architectures represent the direction of pursuing extreme fusion depth, but at higher costs.

</details>

**4. What is a Native Multimodal Model, and How Does It Differ from "Stitched" Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

The construction concept of **"Stitched" Multimodal Models** (such as mainstream open-source VLMs like LLaVA and Qwen-VL) is to take a vision encoder (such as CLIP ViT) and a large language model that have each been fully pretrained on massive unimodal data independently. The two components have never "seen" each other during their respective pretraining stages. Subsequently, an additional connector module (such as a linear projection layer or Q-Former) is trained to "stitch" them together, supplemented by a certain scale of multimodal fine-tuning to endow the entire system with cross-modal understanding capabilities. The advantage of this approach is that existing, mature, and powerful unimodal models can be reused, significantly reducing training costs; however, modality fusion occurs relatively late (performing post-hoc alignment between two models that are already "fixed"), and the depth of fusion is limited by the expressive power of the connector module.

**Native Multimodal Models** are completely different: from the very beginning of the pretraining phase, data from multiple modalities such as text, images (and even audio and video) are mixed together, and trained jointly using a single set of objective functions and model architecture. Modality fusion occurs at the deepest level of the model right from the start of training. Theoretically, because visual and linguistic representations are learned together "from scratch" in the same shared space rather than being forcibly "translated" and aligned post-hoc, native multimodal models can achieve tighter, more natural cross-modal representation capabilities (such as the more natural real-time speech interaction experience demonstrated by GPT-4o). However, the cost of this approach is that both training cost and data scale requirements are far higher than those of stitched architectures, which is the practical reason why most open-source models currently still choose the stitched route.



![“拼接式”与“原生”多模态架构的对比图 (Comparison between stitched and native multimodal architectures.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig05_native_vs_stitched.png)

</details>

**5. What is the "Unified Representation Space" in Unified Multimodal Models, and What Core Challenges are Faced in Building It?**

<details>
<summary>💡 Show Answer</summary>

The unified representation space refers to: raw data from different modalities (text, images, audio, video, etc.), after undergoing their respective specialized encoding/tokenization processing, being ultimately mapped into the **exact same** semantic vector space (this shared space is typically the token embedding space used internally by the language model). This enables the model to process, interact with, and even generate information originating from any modality without distinction under a single unified Transformer computational framework—ideally, the model cannot "tell" in this unified space whether a specific token originally came from an image or text, but simply performs unified reasoning computations based on its semantic content.

Building such a genuinely unified representation space faces the following core challenges:

1. **Huge disparity in information density**: For sequences of equal length (e.g., 100 tokens), if derived from text, almost every token carries independent, explicit semantic information; however, if derived from images or videos, the information density and redundancy are vastly lower. How to compress visual information into a limited number of tokens while preserving key content relevant to linguistic semantics as much as possible is a major challenge;
2. **Unified design of positional encodings/attention mechanisms**: Text sequences are naturally 1D sequential structures, images are 2D spatial structures, and videos additionally superimpose a temporal dimension. Designing a unified positional encoding scheme (such as the aforementioned M-RoPE) capable of rationally expressing these vastly different structural relationships simultaneously is itself a complex architectural design endeavor;
3. **Mutual interference of representation capabilities across modalities**: If tokens from different modalities share the same set of model parameters (especially feed-forward network parameters), the huge variance in statistical properties across modality data easily leads to a zero-sum phenomenon during model optimization—over-fitting the data distribution of one modality may degrade the representation capabilities already learned for another modality. How to balance this trade-off is also a key consideration when constructing a unified representation space.

</details>

**6. What is the Application of MoE (Mixture-of-Experts) Architecture in Multimodal Large Models?**

<details>
<summary>💡 Show Answer</summary>

The core idea of MoE (Mixture-of-Experts) is to place multiple "expert" subnetworks with identical structures but independent parameters in certain layers of the network (typically feed-forward network FFN layers), while introducing a lightweight router module. Based on the features of the current input token, the router dynamically decides which expert (or experts) should be activated to process this token, rather than routing all tokens through all experts. The advantage of this design is that the model's **total parameter count** can be scaled up significantly with the number of experts (meaning the model can store more knowledge and capability), but because each token actually activates only a few experts, the computational cost (activated parameters) consumed during inference does not scale proportionally with total parameters, achieving decoupling between "parameter capacity" and "inference cost."

In multimodal large models, the MoE architecture can be applied to the language model backbone—since different experts spontaneously develop preferences for different types of input during training (implicitly specializing in specific modalities, tasks, or semantic domains), this helps the model accommodate more diverse multimodal knowledge at a constrained inference cost. Some works further design "modality-aware" routing mechanisms, such as explicitly configuring different expert allocation tendencies for visual tokens and text tokens (or even specialized modality-exclusive experts), thereby expanding overall model capacity and better accommodating multimodal knowledge while controlling actual computational overhead during inference.

</details>

**7. What is the "Vision Expert" Module Proposed by CogVLM, and What Problem Does It Solve?**

<details>
<summary>💡 Show Answer</summary>

The researchers of CogVLM argued that the mainstream VLM approach (such as LLaVA) of "vision encoder output $\to$ simple linear projection $\to$ direct concatenation into LLM input" suffers from an inherent representation capability bottleneck: linear projection is essentially a relatively simple mapping operation that causes higher-dimensional and information-rich visual features to lose considerable detail when "dimensionally reduced" to adapt to the language model's representation space. Furthermore, this mapping **occurs only once at the input layer**; once visual information enters the interior of the language model, it is processed entirely according to the language model's original parameters, without further specialized processing tailored to the unique nature of visual information.

To address this issue, CogVLM proposed the "Vision Expert" module: at **every layer** of the language model, an independent set of feed-forward networks (FFN) and QKV projection matrices—whose parameters are completely unshared with the original language model—is additionally inserted specifically to process visual tokens. When visual tokens flow through each layer, they pass through these independent "Vision Expert" parameters, whereas text tokens continue to pass through the original language model parameters. The two types of tokens interact only during the calculation of Self-Attention (i.e., the attention calculation itself is shared and interactive), while their respective feed-forward network updates are completely independent and decoupled. This design of "deep fusion (interaction at every layer) yet parameter decoupling (independent feed-forward parameters)" enables visual features to maintain high, customized expressive capability for visual information across all layers of the language model, rather than performing a one-time, potentially lossy projection transformation solely at the input layer as in standard practice. Practical results demonstrate that this design significantly enhances the model's understanding of visual details, at the cost of significantly increasing total parameter count due to duplicating a set of vision expert parameters close to the scale of the language model.

</details>

**8. How are the parameters of multimodal large language models typically allocated among the visual encoder, connector, and language model?**

<details>
<summary>💡 Show Answer</summary>

In a typical concatenated VLM architecture, the parameter scales of the three components differ dramatically:

- **Language Model**: It typically accounts for the vast majority of the parameters of the entire model. Mainstream open-source VLMs use language model backbones ranging in scale from 7B to 70B. This is because the language model carries the core and most complex knowledge storage and reasoning/generation tasks of the entire system, requiring sufficient parameter capacity to support these capabilities;
- **Visual Encoder**: Its scale is relatively much smaller. Mainstream choices (such as CLIP ViT-L/14) typically have parameter counts ranging from a few hundred million (M) to around 1--2B (InternViT used by InternVL is an exception, reaching up to 6B). This is because the task undertaken by the visual encoder is relatively single-focused (extracting general visual semantic features) and does not require storing and applying vast amounts of world knowledge like the language model;
- **Connector**: It has the smallest number of parameters. Whether it is a simple linear projection layer or a relatively complex Q-Former, it usually has only a few M to tens of M parameters, because it merely assumes the relatively simple function of "spatial transformation/information compression" without undertaking complex reasoning tasks.
This "top-heavy" parameter allocation pattern essentially reflects the design philosophy of current mainstream architectures: "maximally reuse powerful, fully pre-trained single-modality models (especially language models) and only train an additional lightweight bridging module." It is also a practical consideration for rapidly constructing high-performance multimodal models at the lowest cost under limited training resources.



![模块化视觉语言模型中的典型参数占比结构图 (Typical parameter distribution in modular vision-language models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig06_parameter_allocation.png)

</details>

**9. What are the similarities and differences between Perceiver Resampler (e.g., used in Flamingo) and Q-Former?**

<details>
<summary>💡 Show Answer</summary>

**Similarities**: Perceiver Resampler and Q-Former are highly similar in their core concept---both introduce a fixed number of randomly initialized, learnable Query vectors, which "distill" information from a large (and resolution-dependent) quantity of raw visual features through a Cross-Attention mechanism. Ultimately, they output a compressed representation of fixed size that does not vary with input image resolution. This fundamentally solves the problem of "excessive and resolution-variable visual tokens," allowing the downstream language model (or Cross-Attention layer) to process visual inputs of arbitrary resolutions in a unified, fixed-length format.

**Differences**: Q-Former (proposed by BLIP-2) typically **jointly trains multiple additional pre-training objectives** during the pre-training stage (such as Image-Text Contrastive ITC, Image-Text Matching ITM, and Image-Text Generation LM), so that its learned compressed representation is not merely "information compression" but also embeds strong image-text semantic alignment capabilities. While its training pipeline is relatively complex, it usually yields better alignment quality. In contrast, Perceiver Resampler (proposed by Flamingo) features a simpler structural design that generally performs solely the single duty of "information compression" without combining additional pre-training objectives like contrastive learning. The compressed visual tokens need to be dynamically injected and utilized layer by layer inside the language model in conjunction with Flamingo's proprietary Gated Cross-Attention layers (rather than concatenating visual and text tokens together all at once into the language model after compression, as Q-Former does). The two exhibit architectural differences in "when and how" the compressed visual information is delivered to the language model for use.

</details>

**10. How is Flamingo's Gated Cross-Attention specifically designed?**

<details>
<summary>💡 Show Answer</summary>

Flamingo adopts the approach of "freezing the backbone network + inserting new modules": images/videos first pass through a frozen visual encoder to extract features, and are then compressed into a fixed number (e.g., 64) of visual tokens via a Perceiver Resampler. Unlike LLaVA, these visual tokens are not directly concatenated with text to feed into the language model; instead, they are injected by inserting a new **Gated Cross-Attention Dense (GATED XATTN-DENSE)** module every few layers (e.g., every 4 layers) of the pre-trained language model:

- **Cross-Attention Calculation**: Taking the text hidden states of the current layer as Query and the visual tokens as Key/Value, cross-attention of text over visual information is computed, allowing the text to "look back" at the image content at every step of the generation process;
- **Gating Mechanism**: The outputs of the Cross-Attention layer and the subsequent feed-forward layer are respectively multiplied by a learnable scalar initialized to 0 (processed via a $\tanh$ function to ensure the output range is in $[-1, 1]$), and then added to the output of that layer from the original language model. Since the initial gating value is 0, the newly inserted modules do not affect the behavior of the original language model at all when training starts (equivalent to an identity mapping), ensuring training stability; as training progresses, the gating value gradually deviates from 0, and the model incrementally learns to truly utilize visual information;
- **Freezing Original Language Model Parameters**: During Flamingo training, typically only the newly inserted Gated Cross-Attention layers and Perceiver Resampler are updated, while the parameters of the original language model remain frozen, thereby maximally retaining the language capabilities acquired during its pre-training phase.
This design allows Flamingo to seamlessly "transform" any powerful text-only language model into a multimodal model without requiring any fine-tuning of the language model itself.

</details>

**11. What is a decoder-only unified multimodal architecture? What advantages does it have over traditional encoder-decoder architectures?**

<details>
<summary>💡 Show Answer</summary>

A decoder-only unified multimodal architecture concatenates tokens from all modalities (image patch tokens, text tokens, and even audio/video tokens) into a single unified 1D sequence and handles them entirely with a single autoregressive Transformer decoder, no longer distinguishing separate "encoder" and "decoder" modules like encoder-decoder architectures (such as early T5-like models or dedicated encoder-decoder VLMs).

Its advantages are reflected in:

1. **Extremely Simple Architecture and Reuse of Mature Infrastructure**: Only a single set of Transformer Decoder structures is needed, directly reusing the highly mature training and inference infrastructure of text-only LLMs (KV Cache acceleration, tensor parallelism, FlashAttention, etc.);
2. **Ease of Scaling**: The unified architecture simplifies the engineering implementation of scaling up model parameters and training data, making empirical findings of Scaling Laws easier to transfer from the text-only domain;
3. **Native Support for Any-Position Modality Generation**: Since all modalities share the same autoregressive generation process, the model can theoretically generate content of any modality at any position in the sequence (e.g., generating interleaved image-text content such as "a paragraph of text + an image + another paragraph of text") without needing to design separate output heads for each modality as in encoder-decoder architectures.
This is also the architectural choice universally adopted by current multimodal large models pursuing "native unification," such as GPT-4o, Gemini, and Chameleon.

</details>

**12. Why are "native multimodal" models like GPT-4o and Gemini considered the next-generation architectural direction?**

<details>
<summary>💡 Show Answer</summary>

Concatenated architectures (pre-trained visual encoder + connector + pre-trained LLM, such as LLaVA and Qwen-VL) are essentially "late-stage stitching": the visual encoder and language model are pre-trained separately on their respective unimodal data, modality fusion occurs at a later stage of training, and the fusion depth is limited by the expressiveness of the connector (visual information must first be compressed into a limited number of tokens before being "translated" for the language model to understand). This compression bottleneck inevitably leads to the loss of detailed information.

In contrast, native multimodal models mix text, image (and even speech and video) data under the same target objective function for joint training from the earliest stages of pre-training, resulting in earlier and deeper modality fusion. The potential advantages this brings include:

- **Tighter Cross-Modal Reasoning**: Since visual and language representations are learned "from scratch" within the same shared space rather than aligned and concatenated post-hoc, they can theoretically support finer-grained and more natural cross-modal interactions (e.g., real-time voice conversations shown in GPT-4o demos that can be interrupted by users at any time and perceive paralinguistic cues such as tone and pauses);
- **Unified Multimodal Generation Capabilities**: Native multimodal models can more easily support bi-directional capabilities---both "image-to-text description" and "text-to-image/speech generation"---without needing to connect separate understanding models and generation models;
- **Avoiding Representation Space Mismatches Between Pre-trained Unimodal Models**: Concatenated architectures must forcefully "translate" two representation spaces that were originally trained independently via a connector, whereas native multimodality learns in the same representation space from the start, theoretically reducing information loss during the "translation" process.
Of course, the training cost and data requirements of native multimodal models are far higher than those of concatenated architectures, which is the practical reason why most current open-source models still adopt concatenated architectures.



![原生多模态模型中的统一表征与生成流程图 (Unified representation and generation flow in native multimodal models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig07_native_multimodal_flow.png)

</details>

**13. Is the Scaling Law for multimodal large models consistent with that of text-only LLMs? How do they differ?**

<details>
<summary>💡 Show Answer</summary>

The Scaling Law of text-only LLMs reveals that model loss exhibits a predictable power-law decay as parameter count, training data volume, and compute grow. This law also roughly holds in multimodal scenarios---larger visual encoders, more image-text training data, and larger language model backbones generally still yield better performance on multimodal tasks.

However, multimodal scaling involves additional complexity, making its patterns less clear and unified than in text-only scenarios:

1. **Mutual Coupling of Multiple Scalable Dimensions**: Performance improvements depend not only on the scale of the language model, but also rely heavily on multiple dimensions such as the scale of the visual encoder, image resolution (which determines the number of visual tokens), and the ratio of image-text data. Simply expanding the language model size without synchronously enhancing visual-side capabilities will quickly hit a bottleneck in performance gains;
2. **Modality Imbalance in Data Quantity and Quality**: The scale and quality of text-only data are far higher than those of high-quality image-text pair data. As a result, "visual-language alignment data" rather than "language model parameter count" often becomes the performance bottleneck first in multimodal training, forming the so-called "Data Bottleneck";
3. **Evaluation Complexity Brought by Task Diversity**: Scaling Laws for text-only LLMs are usually measured by a single, continuous metric such as perplexity. In contrast, multimodal tasks are highly diverse (classification, retrieval, generation, reasoning), and different tasks exhibit varying degrees of sensitivity to scaling, making it difficult to cleanly characterize the overall scaling curve with a single metric.

</details>

**14. What is the concept of Mixture-of-Transformers (MoT) in multimodal large models?**

<details>
<summary>💡 Show Answer</summary>

MoT can be understood as a specialized application of the Mixture-of-Experts (MoE) concept along the "modality" dimension: while standard MoE typically dynamically routes tokens to different experts based on token content, MoT more directly assigns completely independent Transformer sub-modules to different tokens **by modality type**---for instance, it equips text tokens and image tokens with their own independent Feed-Forward Network (FFN) parameters, and even independent partial attention projection parameters (Q/K/V weights), while retaining shared or interactive mechanisms only in the Attention calculation step itself (i.e., how tokens of different modalities interact).

The motivation behind this design lies in the fact that statistical properties of data from different modalities differ enormously; making them share the same FFN parameters may cause optimization goals of different modalities to interfere with each other (gradient updates from one modality might harm representations already learned by another modality). By adopting an approach of "parameter decoupling with attention interaction," MoT enables the processing path of each modality to specialize in optimizing its own characteristics while still achieving cross-modal information fusion via shared Attention layers. This represents an engineering trade-off between "specialization" and "fusion," highly similar in idea to CogVLM's visual expert module (which sets up separate FFN and QKV projections for visual tokens at each layer).

</details>

**15. What are the pros and cons of sharing parameters vs. separating parameters for visual tokens and text tokens within the same Transformer?**

<details>
<summary>💡 Show Answer</summary>

**Shared Parameters** (mainstream practices such as standard LLaVA and Qwen-VL: visual tokens, after connector mapping, are processed using identical Transformer parameters as text tokens):

- **Pros**: Simple implementation without requiring extra modality-specific network architecture designs; the model naturally learns cross-modal associations within a single unified representation space, and parameter counts do not expand unnecessarily as the number of modalities increases;
- **Cons**: Visual tokens (low information density, high local redundancy) and text tokens (high information density, abstract semantics) differ hugely in statistical properties. Sharing the same FFN parameters can lead to optimization conflicts between the two modalities, forcing the model to compromise between them, making it theoretically difficult for each modality to reach its respective upper bound of capability.

**Separated Parameters** (such as CogVLM's visual expert and MoT approaches: visual tokens and text tokens use their own independent FFN/QKV parameters):

- **Pros**: Each modality can utilize a parameter space better suited to its own characteristics, reducing cross-modal optimization conflicts and theoretically preventing the fine-grained accuracy of visual representations from being compromised by accommodating the parameter distribution of the language model;
- **Cons**: Significantly increases total parameter count and implementation complexity of the model (equivalent to duplicating a parameter set close to the size of the language model for the visual component), and still requires carefully designed cross-modal information interaction mechanisms within Attention layers; otherwise, it easily degenerates into a loose concatenation of "independent processing lacking genuine fusion."
In practice, most open-source VLMs still choose the shared parameter scheme for engineering simplicity and training cost efficiency, while separated parameter schemes appear more in models pursuing stronger retention of visual details (such as CogVLM).

</details>

**16. How is Qwen2-VL's M-RoPE (Multimodal Rotary Position Embedding) designed? What improvements does it make over Qwen-VL?**

<details>
<summary>💡 Show Answer</summary>

**Background**: Like most early VLMs, the first-generation Qwen-VL required pre-processing input images to fixed or limited resolutions, resulting in a relatively fixed number of patches output by the visual encoder and relatively simple position embedding designs. Qwen2-VL proposed "Naive Dynamic Resolution," allowing the model to directly process images of arbitrary resolutions and dynamically convert them into a variable number of visual tokens without forcibly resizing/cropping images to fixed dimensions.

**Design of M-RoPE**: To enable Rotary Position Embedding (RoPE) to simultaneously handle 1D text sequences and multi-dimensional (2D image, 3D video) visual signals, Qwen2-VL decomposes position embeddings into three independent components: **Temporal**, **Height**, and **Width**. For text-only tokens, the position indices of the three components remain identical (degenerating into standard 1D RoPE); for image tokens, the temporal component remains constant (as static images lack a time dimension), while the height and width components increase according to the patch's actual 2D coordinates in the image grid; for video tokens, the temporal component increases with the frame index, while height and width components increase according to the 2D grid within each frame. This design allows the same position embedding framework to naturally and uniformly handle sequence structures of text, images, and videos, with each component remaining relatively independent and non-interfering.

**Improvements Compared to Qwen-VL**: First, it supports arbitrary-resolution inputs (eliminating the need for fixed-size pre-processing), and during inference, images of different resolutions are packed into the same sequence with controlled packing length to limit GPU memory usage. Second, to further compress the number of visual tokens, an MLP follows the visual encoder to merge adjacent 2×2 visual tokens into 1 token (e.g., for a 224×224 image encoded by a ViT with patch size=14, the compressed sequence length can be reduced to around 64 tokens, totaling about 66 tokens with start/end markers), significantly reducing computational overhead caused by long sequences.



![多模态旋转位置编码（M-RoPE）的时空坐标建模向量示意图 (Temporal and spatial coordinate modeling in Multimodal Rotary Position Embedding (M-RoPE).)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig08_mrope_encoding.png)

</details>

**17. How is the "Dynamic High Resolution" strategy of the InternVL series specifically implemented?**

<details>
<summary>💡 Show Answer</summary>

To retain details while avoiding information loss caused by direct scaling, InternVL (such as InternVL 1.5/2.0/2.5) designed a high-resolution processing scheme of "dynamic cropping + thumbnail":

1. **Predefined Aspect Ratio Set**: A set of candidate aspect ratio combinations (e.g., 35 types) is defined in advance. During inference, the closest aspect ratio is selected based on the input image's original aspect ratio, and the image is resized to a target size matching that aspect ratio (for example, an 800×1300 image might be adjusted to 896×1344), minimizing distortion caused by aspect ratio mismatch.
2. **Cropping into Fixed-Size Tiles**: The adjusted image is evenly divided into multiple 448×448 pixel tiles. Each tile is independently encoded by a visual encoder (such as InternViT-6B). During training, a single image is cropped into at most around 12 tiles, while during inference, it can expand to over a hundred tiles, supporting inputs up to roughly 4K resolution.
3. **Retaining an Additional Global Thumbnail**: Beyond local tiles, the entire original image is additionally downscaled into a 448×448 thumbnail and encoded together, providing global contextual information that might be lost between tiles. The thumbnail feature is usually appended after all local tile features.
4. **Pixel Shuffle Token Compression**: Because the total number of visual tokens increases significantly after cropping into multiple tiles, InternVL introduces Pixel Shuffle (an operation borrowed from image super-resolution, here applied inversely for downsampling). By rearranging channel and spatial dimensions, it "compresses" multiple adjacent patch features into a smaller number of tokens, reducing final sequence length with almost no loss of information.

A progressive strategy is typically adopted during training: in the pre-training stage, the visual encoder (InternViT) and connector layer (MLP) participate in training while the language model is frozen; in the instruction fine-tuning stage, all parameters are unfrozen and trained together, gradually transitioning from lower to higher resolutions to balance training efficiency and final performance.

</details>

