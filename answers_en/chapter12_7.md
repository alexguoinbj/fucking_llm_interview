# 12.7 Text-to-Image and Video Generation Models — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 16 questions

**1. What Is a Text-to-Image Model? Provide Examples to Explain Common Technical Routes**

<details>
<summary>💡 Show Answer</summary>

A text-to-image (T2I) model refers to a model that automatically generates image content semantically matching a natural language text description provided by a user. It is one of the most widely followed application directions in generative AI (well-known products such as Stable Diffusion, DALL-E, and Midjourney all belong to this category). Its core challenge lies in how to accurately translate discrete, symbolic text descriptions into continuous, rich visual content within pixel space, while simultaneously ensuring that the generated images satisfy the semantic requirements of the text description and possess high visual quality and aesthetic appeal.

Currently, there are two mainstream technical routes:

- **Diffusion Model-Based Route**: Conditioning on the text description, a clear final image is iteratively generated step-by-step from a random noise image via a "denoising" process. It is currently the technical route with the highest generation quality and widest application (e.g., Stable Diffusion, DALL-E 3, and Imagen all adopt this route);
- **Autoregressive-Based Route**: First converting images into discrete visual token sequences using methods like VQ-VAE, and then using a language-model-like autoregressive Transformer to predict and generate these visual tokens one by one conditioned on text (the original DALL-E adopted this idea, and recent works like VAR and LlamaGen have made further improvements on this foundation).
Currently, the diffusion model route generally holds advantages in generation quality and detail richness, serving as the dominant choice in the text-to-image field. The advantage of the autoregressive route lies in its easier unification with the autoregressive generation paradigm of language models, making it an important direction in exploring "unified image understanding and generation" models.

</details>

**2. Core Idea of Diffusion Models and Reasons Why They Are Suitable for Generation**

<details>
<summary>💡 Show Answer</summary>

**Core Idea**: Gradually adding noise to data until it becomes pure noise via a "forward noisy process," and then training the model to learn a "reverse denoising" process to recover data step-by-step from noise; during generation, starting from random noise, the target image is iteratively generated via the learned denoising process.

**Reasons Suitable for Generation**:
- Naturally suited for modeling data distributions in continuous space (e.g., pixel values).
- The iterative denoising process allows the model to make "minor corrections" at each step, progressively refining generation quality.
- Simple training objective (predicting noise), requiring no adversarial training (the discriminator-generator minimax game in GANs).
- Enables flexible control over the alignment degree between generated results and conditions via Classifier-Free Guidance.

</details>

**3. What Is Classifier-Free Guidance (CFG)? What Is Its Role in Text-to-Image Generation?**

<details>
<summary>💡 Show Answer</summary>

In early conditional diffusion model research, to make generation results adhere more closely to given conditions (such as text descriptions), one approach was to train an additional independent classifier and use the gradient information of that classifier to guide the generation direction during sampling (Classifier Guidance). However, this method required training and maintaining an additional classifier, making implementation complex and inflexible.

Classifier-Free Guidance (CFG) provides a sleeker alternative: During the training of the **same** diffusion model, the model is trained to learn generation capabilities under both "with text condition" and "without text condition (i.e., replacing text input with an empty string or a special null condition token)" settings by randomly dropping the text condition with a certain probability during training. In the actual generation (sampling) stage, the model computes the noise prediction result $\epsilon_{cond}$ given the text condition and the noise prediction result $\epsilon_{uncond}$ without the text condition separately, and then amplifies the difference between the two by a weighting coefficient $w$ (i.e., Guidance Scale) according to the following formula: $\epsilon_{final} = \epsilon_{uncond} + w \times (\epsilon_{cond} - \epsilon_{uncond})$. This difference $(\epsilon_{cond} - \epsilon_{uncond})$ essentially represents the "extra directional information brought by the text condition." By amplifying this direction, the final generated result can be pushed more strongly toward aligning with the text description.

The value of Guidance Scale ($w$) directly influences generation outcomes: A larger $w$ leads to stronger correlation and alignment between the generated result and the text description, but an excessively large $w$ easily causes side effects such as over-saturation, excessively high contrast, and detail distortion, degrading image quality; conversely, an overly small $w$ may result in insufficient matching with the text description. In practice, $w$ usually needs to be tuned to a suitable intermediate value (commonly ranging around 5--15) based on the specific model and task to strike a good balance between "text alignment" and "image quality."

</details>

**4. What Is Diffusion Transformer (DiT)? What Are Its Advantages Over Traditional U-Net-Based Diffusion Models?**

<details>
<summary>💡 Show Answer</summary>

Early diffusion models (such as original DDPM, Stable Diffusion 1.x/2.x) universally adopted U-Net (a convolutional neural network featuring encoder-decoder structures and skip connections) as the backbone network for learning the core task of "denoising." DiT (Diffusion Transformer) proposed a different approach: Dividing the image (in latent space) into a series of patches, just as ViT processes images, and feeding these patch tokens into a standard Transformer architecture for processing, completely replacing U-Net with a Transformer to act as the denoising network.

Compared to the traditional U-Net architecture, the advantages of DiT are mainly reflected in:

- **Better and More Explicit Scalability (Scaling Characteristics)**: The Transformer architecture has been extensively proven in the field of language models to follow very favorable and predictable scaling laws---that is, increases in model scale, data volume, and compute yield relatively stable and predictable performance gains. In contrast, the scaling behavior of convolutional architectures like U-Net is less clear and stable than that of Transformers. The introduction of DiT demonstrated that the denoising backbone of diffusion models can similarly enjoy the "larger is stronger" scaling dividend, providing a more reliable architectural foundation for continuously improving generation quality by expanding model scale;
- **More Natural Fusion of Conditional Control Information**: Transformers are naturally based on attention mechanisms, allowing conditional signals such as text conditions (embeddings of text tokens) and timestep information to interact flexibly and uniformly with image patch tokens via cross-attention or by concatenating conditional information as additional tokens into the input sequence. Compared to U-Net's reliance on specific convolutional layers to insert condition information, this fusion method is more unified and flexible, making it easier to scale up support for diverse conditional controls (e.g., simultaneously accepting multiple conditions like text and pose skeleton maps).
Precisely due to these advantages, the DiT architecture has become the mainstream choice for new-generation advanced text-to-image/text-to-video models such as Stable Diffusion 3 and FLUX, gradually superseding early U-Net-based diffusion model architectures.



![卷积型U-Net与通道块结构Diffusion Transformer (DiT) 架构对比图 (Architectural comparison: Convolutional U-Net versus Diffusion Transformer (DiT).)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig24_dit_vs_unet.png)

</details>

**5. What Is Latent Diffusion? Why Can It Drastically Reduce Computational Costs?**

<details>
<summary>💡 Show Answer</summary>

Early diffusion models performed the complete iterative process of adding noise and denoising directly in high-resolution original pixel space. For example, generating an image of $512 \times 512$ resolution required the diffusion process to repeatedly iterate in a high-dimensional space containing hundreds of thousands of pixel values. Computation and memory overheads were enormous, keeping training and inference costs prohibitively high.

Latent Diffusion (on which the Stable Diffusion series is based) proposed a more efficient solution: **Instead of performing diffusion directly in the original pixel space**, an autoencoder (typically a VAE, Variational Autoencoder) is pre-trained to compress and encode the original image into a **latent representation** whose dimensions are **far smaller** than those of original pixel space (for instance, compressing a $512 \times 512 \times 3$ pixel image into a $64 \times 64 \times 4$ latent feature map, compressing spatial dimensions by 8 times). Subsequently, the entire noise-adding and denoising process of the diffusion model takes place completely within this low-dimensional latent space---the model learns "how to recover meaningful latent representations from noise within latent space" rather than operating directly in pixel space. Upon completing generation, the VAE decoder part is used to decode and restore the final generated latent representation back into a full pixel image.

Because the dimensionality of latent space is far smaller than that of original pixel space (in the above example, data size is reduced by $8 \times 8 = 64$ times), the data volume, computation, and memory footprint required for each iteration step of the diffusion model are drastically reduced. This makes it feasible to train larger-scale diffusion models and generate higher-resolution images under limited compute resources. Meanwhile, because the compression and reconstruction process of VAE itself is specially trained, key visual information of images can be well preserved. This strategy of "compress first, diffuse next, and decompress at last" introduces relatively controllable quality loss, proving in practice to be a highly successful trade-off between computational efficiency and generation quality, as well as the key technological foundation enabling Stable Diffusion to run on consumer-grade GPUs.

</details>

**6. Similarities and Differences in Technical Routes Between Image Generation Models and Image Understanding Models**

<details>
<summary>💡 Show Answer</summary>

**Similarities**: Both rely on powerful visual feature extraction capabilities; both benefit from large-scale pre-training.

**Differences**:
- **Understanding Models**: Primarily discriminative, taking images as input to extract semantic features and outputting text descriptions/classifications.
- **Generative Models**: Primarily generative, starting from noise or text conditions to iteratively generate pixels/latent representations. Mainstream technical routes include diffusion models (Stable Diffusion) and autoregressive models (DALL-E).
- **Architectural Differences**: Understanding models typically employ ViT + LLM; generative models employ U-Net/DiT + text encoder.

</details>

**7. Difficulties in T2I, T2V, and I2V**

<details>
<summary>💡 Show Answer</summary>

1. **Difficulties in T2I (Text-to-Image)**: **Precision of Text-Image Semantic Alignment**---The model needs to accurately understand every detail requirement in the text description and reflect it in the image, being prone to omissions or confusion especially when prompts contain multiple objects or multiple qualifiers; **Fine-Grained Attribute Binding**---When descriptions involve multiple objects and their respective colors, quantities, and spatial relationships (e.g., "a red ball on the left and two blue cubes on the right"), the model easily suffers from attribute binding errors (attributing colors to incorrect objects); **Text Rendering**---Accurately rendering specified text content in generated images (such as words on signboards or book cover titles) has historically been a major weakness of text-to-image models (see details in the next question).
2. **Difficulties in T2V (Text-to-Video)**: **Temporal Consistency**---Ensuring that the same object maintains consistent appearance and shape across different frames in the generated video, avoiding "flickering" phenomena caused by abrupt shape changes between frames; **Action Plausibility**---The generated motion process must comply with physical laws and common sense (e.g., human walking gait, object falling trajectories), rather than performing arbitrary interpolations across frames; **Accumulated Error in Long Video Generation**---As generated video duration increases, minor errors in early frames are continuously amplified and accumulated during subsequent frame generation, leading to noticeable quality degradation or content drift in later portions of the video; **Extremely High Computational Cost**---Video data volume far exceeds that of single images, resulting in orders of magnitude growth in compute resources required for training and inference compared to text-to-image tasks.
3. **Difficulties in I2V (Image-to-Video)**: **Balancing First-Frame Consistency with Dynamic Motion Injection**---Requiring both strict preservation of the first frame matching the original user-provided input image completely, and reasonable injection of logical motion and changes upon it, which demands a delicate balance; **Control of Motion Magnitude**---If motion is too small, the video appears stiff and lacks dynamic appeal; if motion is too large, it easily degrades the structural integrity and identity consistency of the original image (such as figure distortion); **Coordination of Foreground and Background Motion**---Ensuring that motions of multiple dynamic subjects in the frame (foreground figures, background environmental elements) are visually coordinated and adhere to common-sense logic of a single physical scene, rather than executing independent, unreasonable motions.

</details>

**8. Why Is the Common "Text Rendering" Problem (Correctly Displaying Text in Generated Images) Particularly Difficult in Text-to-Image Models?**

<details>
<summary>💡 Show Answer</summary>

Enabling text-to-image models to accurately render specified text content in generated images (for example, "generate an image of a cake with the words 'Happy Birthday'") has long been a prominent weakness of such models. The fundamental reason for this difficulty lies in the intrinsic informational difference between text and ordinary visual content:

- **Text Consists of Discrete Symbols Requiring "Exact Reproduction," Whereas Diffusion Models Learn Statistical Distributions of Pixels**: Ordinary visual content (such as a leaf or a cloud), even if generated with minor subtle differences from reality, will usually not appear "incorrect" (after all, leaf shapes in nature are inherently diverse). Text, however, is entirely different---every stroke and every detailed structure of text must accurately correspond to the correct character shape; even a missing or extraneous stroke can render text unreadable or completely change its meaning. Yet the training objective of diffusion models is fundamentally to learn overall pixel statistical distributions of natural images, lacking an inherent capacity to model such "zero-tolerance" precise symbols;
- **Text Encoders Extract Semantic-Level Features Rather Than Character-Level Precise Shape Information**: Text-to-image models typically employ text encoders such as CLIP to convert input text descriptions into condition vectors. The training objective of such encoders is to capture the **overall semantics** of text (e.g., understanding the concept of "birthday cake"), rather than precisely recording the exact shape and layout of each character. Consequently, during generation, the model "knows text should be present" but lacks precise guidance regarding specific glyph shapes, easily yielding results that "look like text but are actually meaningless gibberish strokes."
Recent improvement schemes attempt to address this issue progressively by introducing specialized character-level text encoders (which retain per-character shape details more finely), significantly enlarging model scales, and constructing targeted training data containing numerous accurate text-rendering samples. However, even for state-of-the-art text-to-image models, the error rate remains noticeably higher when generating long text paragraphs or non-Latin characters (such as Chinese) compared to generating standard visual content.



![文生图模型中子词级文本嵌入引起的字符表征丢失结构示意图 (Character-level representation loss during subword text embedding in T2I models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig25_text_rendering.png)

</details>

**9. Why Do Diffusion Models Perform Excellently in Image Generation but Remain Non-Mainstream for Text Generation?**

<details>
<summary>💡 Show Answer</summary>

The root of this issue lies in the essential difference in **continuity** between image and text data modalities:

Images (especially at the level of pixels or VAE-encoded latent representations) are fundamentally **continuous** numerical signals---pixel values can vary smoothly within a continuous numerical range. The iterative generation process of diffusion models, starting from pure noise and gradually removing noise in small increments, is naturally suited for smoothly modeling and generating data in such continuous spaces, where every minor noise removal step corresponds to a reasonable, continuous change in data distribution.

Text, in contrast, is fundamentally a sequence of **discrete** symbols---each word or subword is an isolated, discontinuous discrete symbol within a vocabulary. There exists no "continuous transition" relationship between words akin to pixel values (for instance, while the words "cat" and "dog" may be semantically related, there is no valid word sitting "numerically midway" between them). Applying the continuous noise-adding and denoising framework of diffusion models directly onto such discrete symbol spaces is inherently unnatural and demands specialized handling, such as mapping discrete words into a continuous embedding space for diffusion (and mapping back to discrete words post-generation), or designing discrete diffusion processes specifically for discrete state transitions. These schemes carry higher implementation complexity than continuous diffusion for image generation.

Furthermore, from an engineering ecosystem perspective, current autoregressive Transformer language model architectures are mature in training efficiency, inference acceleration (especially key-value (KV) caching mechanisms enabling massive inference speedups), and supporting developer toolchains. By contrast, discrete diffusion text generation models lag far behind the autoregressive paradigm in terms of maturity and ecosystem accumulation. Weighing generation quality, training/inference efficiency, and compatibility with existing ecosystems, autoregressive Transformers remain the universally accepted mainstream paradigm in text generation. While exploring diffusion models for text generation remains an active research avenue, it is still far from replacing the autoregressive paradigm.

</details>

**10. What Is ControlNet? How Does It Achieve Fine-Grained Control Over Generated Images?**

<details>
<summary>💡 Show Answer</summary>

Standard text-to-image models accept only text as conditional input, making it difficult for users to precisely control specific object poses, composition layouts, or contour shapes in generated images using text alone (for instance, it is difficult to precisely describe spatial details such as "a figure raising their left hand at a 45-degree angle" in text). The core objective of ControlNet is to enable diffusion models to accept **additional structured conditional inputs** (such as Canny edge maps, depth maps, human pose skeleton maps, scribble sketches, semantic segmentation maps, etc.), thereby achieving fine-grained control over generated outcomes.

Its implementation approach is as follows: **Duplicating** the U-Net encoder portion of a pre-trained diffusion model (such as Stable Diffusion) to serve as a new, trainable "control branch," while keeping the original U-Net backbone weights frozen. The control branch accepts the additional conditional image as input, and its output features at each layer are injected into the feature maps of corresponding layers in the original U-Net via a **zero convolution layer** (Zero Convolution, a $1 \times 1$ convolution whose weights are initialized to all zeros). The cleverness of zero convolution lies in the fact that, because its weights are initialized to zero, whatever the control branch outputs at the start of training will be zeroed out after zero convolution, exerting zero impact on the behavior of the original U-Net. This guarantees that pre-trained generation capabilities are not disrupted at the onset of training. As training progresses, zero convolution weights gradually learn from zero, allowing control branch information to be smoothly injected into the backbone network. Thus, the model learns to make final generated results strictly adhere to the structural information of additional condition maps while preserving its original generation quality.

</details>

**11. What Architectural Improvements Do New-Generation Text-to-Image Models Like Stable Diffusion 3 and FLUX Feature Over Early SD?**

<details>
<summary>💡 Show Answer</summary>

Compared to early versions such as Stable Diffusion 1.x/2.x, new-generation models incorporate systematic improvements across multiple levels:

1. **Architectural Upgrade of the Denoising Backbone**: Early SD used convolutional U-Net as the denoising backbone. New-generation models (such as SD3's MM-DiT, Multimodal Diffusion Transformer) switch to a Transformer-based architecture, while feeding both latent space image patch tokens and text tokens into the same Transformer for joint processing. Image and text maintain separate sets of weight parameters but interact through shared attention layers. This design preserves modeling flexibility for each modality while achieving fuller image-text feature fusion, as well as superior scalability (scaling laws) over U-Net;
2. **Improvement of Training Objectives**: Early diffusion models universally adopted noise prediction objectives proposed by DDPM, whose noise-adding and denoising trajectories form curved paths in probability space. New-generation models (such as SD3 and FLUX) widely adopt objectives like **Rectified Flow / Flow Matching**. The core idea is to train the model to learn a transport path from noise to real data that is as close to a **straight line** as possible. A straighter path means accumulated errors are smaller when approximating the path with fewer discrete sampling steps, allowing high-quality generation results to be obtained in far fewer sampling steps (even single-digit steps) than traditional methods;
3. **Enhancement of Text Encoders**: Early SD relied primarily on a single CLIP text encoder to extract text condition features. The CLIP encoder was designed for vision-language contrastive learning and exhibits limited understanding of long text and complex logical relationships. New-generation models often combine multiple text encoders (such as using CLIP and T5-XXL simultaneously). Specialized language models like T5 excel in comprehending long prompts with complex multiple constraints and logical relations, thereby enhancing model responsiveness and precision to complex prompts.

</details>

**12. What Is a Consistency Model? How Does It Accelerate Sampling Speed in Diffusion Models?**

<details>
<summary>💡 Show Answer</summary>

The generation process of traditional diffusion models requires starting from pure noise and iteratively denoising backward along a pre-defined noise trajectory. Removing only a small portion of noise at each step typically requires tens to hundreds of iteration steps to obtain a high-quality final image, making inference speed a major bottleneck in practical diffusion model applications.

The core idea of Consistency Models is to directly train a function $f$ such that for an intermediate state at **any** timestep (i.e., any noise level) along the diffusion trajectory, feeding it into function $f$ yields an output that "consistently" maps back to the same origin point---namely, the original, noise-free clean data. In other words, rather than learning a local mapping such as "removing a small step of noise," the model directly learns a global mapping: "jumping directly back to clean data regardless of the current noise level on the trajectory" (self-consistency constraint: the mapped results of any two points on the trajectory under function $f$ must remain consistent).

Once trained, generating new samples no longer requires step-by-step, multi-iteration denoising along the trajectory. Instead, a random noise can be directly input into the trained consistency function to obtain high-quality generation in a **single step** (or with an additional 1--4 steps for minor quality refinement). This drastically compresses the required sampling steps from tens or hundreds down to single-digit steps, representing one of the most critical technological directions for diffusion model inference acceleration, particularly suited for real-time or interactive applications demanding fast generation speeds.



![一致性模型映射路径与普通扩散模型步进式求解路径对比曲线图 (Consistency mapping trajectories versus step-by-step diffusion ODE trajectories.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig26_consistency_sampling.png)

</details>

**13. How Is Image Editing (Inpainting/Outpainting) Implemented in Diffusion Models?**

<details>
<summary>💡 Show Answer</summary>

**Inpainting (Image Restoration / Local Redrawing)**: Given an original image and a mask specifying the region requiring regeneration, at each step of the diffusion model's reverse denoising process, for areas outside the mask (regions to retain), the content generated by the model itself is forcibly replaced with the result of "the original image's corresponding region after adding noise at the current timestep." For areas inside the mask (regions to regenerate), the model is allowed to freely generate content following the standard diffusion denoising pipeline. By performing this "replacement" operation at every step, the final output achieves the effect of "strictly preserving original image content outside the mask while freely generating new content inside the mask conditioned on text prompts or blank space." Meanwhile, because regions near mask boundaries naturally influence one another during diffusion (receptive fields of attention and convolutions cross mask boundaries), a natural transition usually forms between generated content and retained regions without awkward stitching seams.

**Outpainting (Image Expansion / Extension)**: The concept is similar to inpainting, but the application scenario is "expanding the canvas outward" rather than "modifying internal regions"---placing the original image at the center (or any target expansion direction) of a larger blank canvas. The area covered by the original image is treated as the region to "retain," while blank areas outside the original image on the canvas are treated as regions to "generate." Utilizing the exact same mechanism as inpainting (replacing retained regions at each denoising step and freely generating regions to be completed), the model naturally extends new visual content outward based on boundary content, lighting, and style cues from the original image, achieving the effect of expanding the original image's field of view.

</details>

**14. What Is Autoregressive Image Generation (e.g., VAR, LlamaGen)? What Are Its Pros and Cons Compared to Diffusion Models?**

<details>
<summary>💡 Show Answer</summary>

Autoregressive image generation continues the language model paradigm of "predicting the next token one by one" applied to image generation: First, a vector-quantized autoencoder similar to VQ-VAE is used to compress and encode an image into a discrete vocabulary-like visual token sequence (where each token corresponds to an index from a finite "visual codebook"). Subsequently, a GPT-like autoregressive Transformer is trained to predict these discrete visual tokens sequentially according to a pre-defined order (e.g., raster scan order). During generation, token sequences are similarly sampled token-by-token in that order, and finally decoded back into a pixel image via a decoder.

Recent works such as VAR (Visual Autoregressive Modeling) improve upon traditional patch-by-patch sequential generation by employing "coarse-to-fine multi-scale autoregression": First generating a very low-resolution (such as $1 \times 1$ or $2 \times 2$) rough visual token layout, and then progressively predicting tokens at higher-resolution scales (each step conditioned on the result from the previous coarser scale). This coarse-to-fine generation order aligns better with human perception of images ("grasping overall contours before inspecting details"), significantly accelerating generation speed and improving quality compared to traditional patch-by-patch ordering.

**Pros and Cons Compared to Diffusion Models**: The advantage of the autoregressive approach lies in its ability to **directly reuse** the highly mature training and inference infrastructure from the language model field (such as KV Cache for accelerating autoregressive decoding and mature parallel training strategies). Additionally, it makes unifying image generation capabilities and text generation capabilities under a single autoregressive framework easier (i.e., the "unified understanding + generation" model concept mentioned earlier). The disadvantage is that forcibly discretizing continuous image signals into a finite visual vocabulary introduces a degree of quantization loss. Moreover, token-by-token sequential generation (even multi-scale coarse-to-fine) currently lags behind state-of-the-art diffusion models in generation quality, detail richness, and diversity in most works, which is why diffusion models continue to hold a dominant position in image generation.

</details>

**15. How Are Unified "Image Understanding + Generation" Models (e.g., Chameleon, Emu, Janus) Designed?**

<details>
<summary>💡 Show Answer</summary>

The core design philosophy of such models is to break the long-standing separation between "understanding models" and "generative models," enabling the **same model** to perform both "image captioning/understanding" (taking images as input and outputting text) and "text-to-image generation" (taking text as input and outputting pixels), and even supporting interleaved image-text generation (text and images alternating within a single passage). In terms of specific implementation, images are typically encoded into discrete or continuous tokens (similar to the autoregressive route of text-to-image generation, often using VQ-like methods to quantize images into discrete tokens). These are incorporated alongside text tokens into a **single** autoregressive Transformer for joint modeling---during training, the model learns both "predicting text tokens given image tokens" (understanding tasks) and "predicting image tokens given text tokens" (generation tasks), with both capabilities sharing the same set of model parameters.

It is worth noting that some recent works (such as the Janus series) have identified an intrinsic conflict between the visual feature requirements of "understanding" and "generation" tasks: Understanding tasks demand high-level semantically abstract features (similar to features extracted by CLIP encoders, which facilitate language alignment), whereas generation tasks require features that preserve rich pixel details (facilitating precise image reconstruction). Consequently, models like Janus choose to **decouple vision encoders**: Using a semantically strong encoder (such as SigLIP) dedicated to understanding tasks, while employing another encoder with strong pixel reconstruction capabilities (such as VQ-based tokenizers) dedicated to generation tasks. Both independent visual processing paths ultimately connect to a single unified autoregressive Transformer backbone. This design of "decoupled encoders, unified generative backbone" has been proven to achieve superior overall performance compared to forcibly sharing the same set of visual features.

</details>

**16. What Is the Core Technical Approach of Video Generation Models Like Sora?**

<details>
<summary>💡 Show Answer</summary>

The core technical approach of advanced text-to-video models represented by Sora is built upon the framework of "combining diffusion models with Transformers and representing video uniformly as spacetime patch sequences":

1. **Spacetime Expansion of Visual Patches**: Unlike image diffusion models that process only 2D spatial patches, video generation models divide video simultaneously across spatial and temporal dimensions into a series of "spacetime patches." Each patch contains both spatial local information and temporal local information, thereby converting 4D video data (width, height, time, channels) into a 1D token sequence that can be directly fed into a Transformer.
2. **Operating in Compressed Latent Space**: Similar to Latent Diffusion, video data is first mapped into a lower-dimensional latent space via a video compression network (spacetime autoencoder). Spacetime patchification and diffusion processes are then conducted within this latent space, drastically reducing the computational cost of directly processing high-dimensional video data in raw pixel space.
3. **Uniform Handling of Variable Durations, Resolutions, and Aspect Ratios**: Because spacetime patchification naturally supports variable numbers of tokens, such models can be trained and can generate videos of differing durations, resolutions, and aspect ratios under the exact same architecture, without being constrained by fixed input sizes as early video generation models were.
4. **DiT as the Denoising Backbone**: Employing Transformer-based diffusion model (DiT) architectures as the denoising network backbone provides inherently strong scalability (scaling laws). This allows video generation quality to improve continuously with increases in model scale and training compute, aligning with the trend in image generation of transitioning from U-Net to DiT.

This combination of "spacetime patches + latent space diffusion + DiT" is regarded as a key technical route for achieving high-quality, long-duration, and high-resolution video generation, as well as an important technical foundation for simulating physical world dynamics via video generation in "World Model" research.



![类Sora视频生成模型中的统一时空分块去噪流程图 (Unified spatiotemporal patch denoising workflow in Sora-like video generators.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig27_sora_pipeline.png)

</details>

