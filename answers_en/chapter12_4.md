# 12.4 Vision Encoding and Input Processing — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 17 questions

**1. Why does image input need to undergo patchification?**

<details>
<summary>💡 Show Answer</summary>

The computational complexity of Transformer's Self-Attention is quadratic in sequence length. Directly treating pixel-level images (e.g., 224×224=50176 pixels) as token sequences results in an enormous amount of computation.

**Core reasons for patchification:**
1. **Reduce sequence length**: Dividing an image into N×N patches (e.g., 16×16 pixels) turns a 224×224 image into 196 patch tokens, making computation feasible;
2. **Construct local features**: Each patch captures local visual semantics, forming an analogy with words (tokens) in text, which makes it suitable for Transformer modeling;
3. **Unify input formats**: After linear projection, the dimension of patch embeddings aligns with text embeddings, facilitating unified multimodal modeling;
4. **Flexibly handle resolution**: The trade-off between sequence length and detail accuracy can be controlled by adjusting patch size or image resolution.

</details>

**2. How is positional encoding applied in the image modality?**

<details>
<summary>💡 Show Answer</summary>

Image patches are arranged in a two-dimensional grid, and positional encoding needs to reflect this spatial structure:

1. **1D Learnable Positional Encoding** (Original ViT scheme): All patches are arranged in raster order, with a learnable vector assigned to each position. Practice shows that performance is comparable to 2D encoding, and the implementation is simple.
2. **2D Sinusoidal Positional Encoding**: Uses sine/cosine encodings separately for row and column coordinates, which are then concatenated or added together to express spatial positional relationships more explicitly.
3. **Relative Positional Encoding** (2D extension of RoPE): Extends RoPE to two dimensions to directly encode relative positions between two patches during Attention computation, making it more robust to changes in image resolution.
4. **Dynamic Resolution Positional Interpolation**: When the inference resolution differs from the training resolution, bilinear interpolation is applied to the pre-trained positional encodings, enabling the model to adapt to higher-resolution inputs (such as positional interpolation fine-tuning in ViT).
5. **No Positional Encoding Scheme**: Some models (such as certain variants of MAE) explore removing explicit positional encodings, relying on the attention mechanism to adaptively learn positional relationships.

</details>

**3. Why is the "choice of visual encoder" said to have a major impact on the upper bound of the final multimodal model's capabilities?**

<details>
<summary>💡 Show Answer</summary>

The visual encoder serves as the **sole gateway** for the entire multimodal system to acquire visual information. It determines the "ceiling" of the information granularity and semantic hierarchy that the model can extract from images. If the visual encoder itself has not adequately learned good representation capabilities for certain visual information during pre-training (such as fine-grained textures, tiny text, or precise spatial positional relationships), then no matter how exquisitely the subsequent connection module is designed or how strong the language model's reasoning capacity is, it cannot "generate" out of thin air information that was absent from the visual features in the first place—this is a classic issue of "downstream being unable to compensate for upstream bottlenecks," akin to "making bricks without straw." Therefore, factors such as the scale and diversity of the visual encoder's pre-training data (whether it has seen sufficiently rich scenes and objects), the supported input resolution (which determines how fine details can be captured), and specific pre-training objectives (contrastive learning leaning toward semantics vs. masked reconstruction leaning toward details, as mentioned earlier) directly dictate and constrain the upper bound of capabilities that the entire multimodal large model can achieve in various downstream visual understanding tasks. This is also a major reason why an increasing number of recent advanced VLMs (such as InternVL) choose to train dedicated visual encoders of larger scale and stronger capabilities rather than directly reusing off-the-shelf CLIP models.

</details>

**4. What methods exist for multimodal processing of high-resolution/multi-resolution inputs? What challenges do they introduce?**

<details>
<summary>💡 Show Answer</summary>

Standard visual encoders (such as ViT) are typically designed to handle fixed-size inputs of low resolution (e.g., 224×224 or 336×336). Processing high-resolution images introduces challenges in three aspects:

1. **Explosion in computational complexity**: The number of patches is proportional to the square of the side length, while Self-Attention complexity is proportional to the square of the number of patches. Doubling the resolution leads to a 16-fold increase in computational cost.
2. **Failure of positional encoding**: Pre-trained ViT positional encodings are learned for a specific number of patches; an increase in the number of patches will exceed the range of existing positional encodings.
3. **Surge in GPU memory usage**: Longer sequences mean that every layer of the Transformer must store larger activation values.

Currently, the primary coping strategies include:

1. **Tiling-Encoding-Concatenation**: Defines a baseline resolution ("tiling unit"), cuts the image into multiple sub-blocks matching the shape of the tiling unit, independently encodes them, and concatenates their features, while retaining a low-resolution global view of the original image. The advantage is that pre-trained low-resolution models can be directly reused, whereas the disadvantage is that global structures of objects spanning across tiles may be disrupted.
2. **Packing methods**: Concatenates patchified features from images of different resolutions directly along the sequence dimension and introduces block diagonal masks to avoid mutual attention computation between tokens of different images, thereby enabling batch-parallel processing of inputs with different resolutions.
3. **Variable-resolution patches**: Keeps the number of patches constant while dynamically adjusting the size of each patch based on input resolution to avoid computational explosion, though larger patches lead to the loss of local details.
4. **Multi-scale feature fusion**: Uses a visual backbone capable of handling high resolutions (such as Swin Transformer) to extract multi-scale features from different levels, which are fused via a Feature Pyramid Network (FPN) before being fed into adapters (such as Fuyu-8B, Monkey).

</details>

**5. After increasing the visual encoder resolution, how can more information be preserved without significantly increasing the number of tokens?**

<details>
<summary>💡 Show Answer</summary>

The core contradiction of this question lies in: increasing resolution inherently implies more image details, but unconstrained proportional growth in the number of visual tokens quickly makes computational and memory overhead unsustainable. Therefore, an optimal balance must be sought between "information gain from increased resolution" and "cost from token count growth." Common approaches include:

- **Dynamic tiling strategy**: Crops the high-resolution image into multiple sub-images, independently processes each with a relatively lower-resolution encoder (rather than using a single encoder to directly process an ultra-large resolution), and then reorganizes and concatenates the features of each sub-image via positional markers. This captures high-resolution details without requiring the visual encoder itself to support ultra-long sequence lengths;
- **Hierarchical feature fusion**: Simultaneously extracts low-resolution global features (providing overall scene context) and high-resolution local features (providing detailed information), fusing the two before feeding them into the language model, rather than crudely preserving all tokens from every resolution level as-is;
- **Efficient visual token compression modules**: Technologies such as the aforementioned Q-Former (distilling key information via learnable queries), Token Pooling (pooling and merging spatially adjacent tokens), and Pixel Shuffle (folding spatial dimensions into channel dimensions) essentially aim to retain as much valid information as possible under the constraint of an "unchanged or slightly increased total token budget." This represents the most mainstream technical direction for tackling this issue today.



![高分辨率视觉输入下的视觉Token压缩流程图 (Token compression pipeline for processing high-resolution visual inputs.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig12_token_compression.png)

</details>

**6. How to handle images with extreme aspect ratios (tall/wide images)?**

<details>
<summary>💡 Show Answer</summary>

For images with extreme aspect ratios (such as a very narrow tall image or a very wide panoramic image), crudely cropping or squishing them into standard square inputs will result in cutting off large amounts of original content or producing severe deformation distortion. Therefore, specialized processing strategies are required:

1. **Adaptive tiling**: Dynamically determines the way sub-image blocks are split according to the actual aspect ratio of the image. For example, for a very wide horizontal image, more columns of sub-blocks should be split horizontally, while for a vertical tall image, more rows should be split vertically. This ensures that each sub-block itself remains close to a square (aligning with the pre-training assumptions of visual encoders) while avoiding the loss of significant original image information caused by forcibly uniform cropping into a square;
2. **Aspect-ratio-preserving scaling with padding**: Proportionally scales the image within the target size range according to its original aspect ratio and fills (pads) the shorter side with blank pixels up to the target square or standard ratio. This completely preserves the relative spatial relationships of all content in the original image without introducing any deformation, at the cost of introducing some "invalid" padded areas;
3. **Multi-scale encoding**: Applies different encoding resolutions to regions of varying importance in the image. For instance, using higher-resolution encoding for the main subject or user-focused areas and lower-resolution encoding for secondary regions like backgrounds. Thus, under a limited total token budget, the "resolution budget" is allocated more rationally to content that truly matters.

</details>

**7. What is the AnyRes (Any-Resolution) dynamic tiling strategy? (e.g., implementations in LLaVA-NeXT, InternVL)**

<details>
<summary>💡 Show Answer</summary>

The core idea of AnyRes (also known as dynamic high-resolution strategy) is to predefine a set of **candidate grid partitioning schemes** (e.g., various row-column combinations such as 1×1, 1×2, 2×1, 2×2, 1×3, 3×1). During inference, the system automatically selects from these predefined candidate grids the scheme that is **closest** to the actual aspect ratio of the input image. It then splits the image into the corresponding number of sub-image blocks according to the selected grid, with each sub-block encoded independently by the visual encoder. Meanwhile, to avoid the loss of global context information caused by partitioning (for example, a large object spanning across the boundaries of multiple sub-blocks might be fragmented after splitting), the system also **retains an additional low-resolution thumbnail of the entire image** to be encoded alongside them. Finally, the features of all sub-image blocks and the features of this global thumbnail are concatenated together and jointly fed into the language model. Compared with completely unconstrained dynamic resolution processing, this design of "predefined candidate grids + thumbnail supplementing global context" makes engineering implementation and the training process much more controllable (due to the finite number of candidate grids), while being far better adapted to diverse real-world image inputs with vast aspect ratio differences than fixed single-resolution schemes. It has become a widely adopted practical solution in current mainstream high-resolution VLMs such as LLaVA-NeXT and InternVL.

</details>

**8. How do the visual encoder's pre-training objectives (contrastive learning vs. masked reconstruction) affect downstream multimodal task performance?**

<details>
<summary>💡 Show Answer</summary>

Different pre-training objectives lead visual encoders to learn feature representations of distinctly different styles, which in turn influences their performance characteristics in downstream multimodal tasks:

**Encoders trained on contrastive learning** (such as CLIP and SigLIP, trained via image-text contrastive objectives): Because their training objectives directly demand that image features and corresponding text descriptions draw close to each other in semantic space, the features learned by these encoders naturally possess strong **semantic abstraction** properties and achieve high-quality alignment with natural language. Consequently, they excel at tasks requiring "understanding what the image as a whole is depicting," such as image-text retrieval and zero-shot classification. However, because the training objective focuses on global semantic matching rather than pixel-level detail reconstruction, these encoders have relatively weaker retention capabilities for local details (such as precise textures, boundary shapes, and tiny text).

**Encoders trained on masked image modeling** (such as MAE and BEiT, trained by randomly masking out image patches and requiring the model to reconstruct the masked content): Because their training objective is the precise reconstruction of original pixels or low-level visual features, these encoders excel at capturing **local texture and fine-grained structural** information in images. However, since the training process involves no textual supervision signals whatsoever, the alignment capability between their learned features and language semantics is relatively weak, rendering their direct application to image-text alignment tasks generally less effective than CLIP-style encoders.

Precisely because both types of encoders have their own strengths and complement each other, many works striving for ultimate performance attempt to simultaneously utilize encoders trained with two (or even more) different pre-training objectives (for example, using both CLIP and DINOv2 to extract semantic features and detail features, respectively). Their output features are then fused before being fed into the connection module, aiming to strike a balance between semantic alignment capability and detail fidelity.

</details>

**9. What is 2D-RoPE? Why is it increasingly adopted in multimodal large models?**

<details>
<summary>💡 Show Answer</summary>

RoPE (Rotary Position Embedding) was originally designed as a relative positional encoding scheme for 1D text sequences. Its core idea is to apply a position-dependent rotation transformation to the Query and Key vectors, allowing the Attention computation to naturally encapsulate the relative positional information between two tokens, while endowing the encoding scheme with favorable length extrapolation properties.

2D-RoPE extends this idea from 1D sequences to 2D grid structures: for each image patch, instead of using a single positional index to represent its location, its **row coordinate** and **column coordinate** in the image grid are computed separately, with rotary positional encodings applied to both coordinate components. Consequently, during Attention computation, the model can implicitly perceive the relative positional relationships between any two patches along the row and column dimensions independently.

Compared with traditional learnable absolute positional encodings that must be learned for a fixed number of patches, the advantage of 2D-RoPE lies in the fact that its encoding method is based on mathematical transformations of relative positional relationships (rather than fixed lookup table vectors). Unlike absolute positional encoding, when encountering a resolution during inference that differs from training (corresponding to a different number of patches), 2D-RoPE does not require additional interpolation to "piece together" a new set of positional encodings. 2D-RoPE naturally and smoothly generalizes to combinations of resolutions and aspect ratios never seen during training. This characteristic makes it exceptionally suited to current multimodal large model architectures where high-resolution and dynamic-resolution processing have become mainstream requirements. As a result, it has been widely adopted by new-generation models such as Qwen2-VL (and further extended into M-RoPE to simultaneously support the temporal dimension in videos).



![应用于图像分块坐标网格的2D旋转位置编码（2D-RoPE）示意图 (2D Rotary Position Embeddings applied to image patch coordinate grid.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig13_2d_rope.png)

</details>

**10. How to handle "multi-image input" scenarios (where a single conversation contains multiple images)?**

<details>
<summary>💡 Show Answer</summary>

Handling multi-image input scenarios requires specialized designs across multiple levels:

- **Independent encoding**: Each image independently extracts features through the visual encoder to generate its own sequence of visual tokens, ensuring images do not interfere with each other during the encoding phase;
- **Image separator tokens**: In the final concatenated complete token sequence, explicit separator tokens (such as special tokens like `<image_1>` and `<image_2>`) need to be inserted between the visual token sequences corresponding to different images. This helps the model clearly identify the boundaries of "which image this section of tokens belongs to," avoiding the confusion of visual information across multiple images in the sequence;
- **Independent positional encodings**: When designing positional encodings, mutually independent spatial positional indices must be assigned to the patch tokens of different images (for instance, row and column coordinates within each image are recalculated starting from zero). This avoids situations where "row 5 of the first image" and "row 5 of the second image" are encoded with identical positional information, preventing location confusion;
- **Specialized training data**: Supporting multi-image input at the architectural level alone is insufficient. Samples involving multi-image comparison and multi-image relational reasoning (e.g., "compare the differences between these two images" or "how should these three images be arranged in chronological order") must be specifically constructed in the instruction fine-tuning data. Only then can the model be truly trained to acquire **cross-image reasoning** capabilities, rather than treating each image as an isolated, unrelated island of information.

</details>

**11. What is Token Merging / Token Pruning? How can the number of visual tokens be reduced during inference?**

<details>
<summary>💡 Show Answer</summary>

**Token Merging**: The core idea is to compute pairwise similarities between all visual tokens between certain layers of the Transformer (usually based on Key vectors rather than original features, which lowers computation costs), identify candidate token pairs with the highest similarity, and merge them via weighted averaging into a new representative token (the merging weights can consider the original "size" represented by the merged tokens to prevent information dilution). Because large background areas or solid color regions in images inherently contain numerous highly similar patches, merging these redundant tokens typically incurs minimal information loss while significantly reducing sequence length.

**Token Pruning**: Unlike merging, pruning directly ranks visual tokens based on an importance score (most commonly the sum of attention weights received by the token during Attention computation, or whether it receives significant focus from CLS Tokens/text Queries) and directly discards the lowest-scoring portion of tokens without any information preservation processing.

What they have in common is that both can serve as "plug-and-play" techniques applied during the **inference phase** (some methods do not even require model retraining, only inserting merging/pruning operations at intermediate layers during inference) to dynamically determine the compression ratio based on actual latency or GPU memory budgets. Hence, they are widely used in deployment scenarios sensitive to inference costs (such as online services handling high concurrent requests, or scenarios involving multi-image/video processing). The difference is that Merging generally preserves information better (at the cost of extra similarity computation overhead), whereas Pruning is simpler and more straightforward to implement (though directly discarding information risks losing key details if the compression ratio is too aggressive).

</details>

**12. How to choose the Patch Size in ViT? What impact does it have on model performance and efficiency?**

<details>
<summary>💡 Show Answer</summary>

Patch Size determines how many "visual tokens" an image is divided into: for an image of fixed resolution (such as 224×224), a smaller Patch Size results in a larger number of patches (for example, Patch Size = 14 produces about 256 patches, whereas Patch Size = 32 produces only about 49 patches).

**Impact on performance**: A smaller Patch Size means each patch covers a smaller area of original pixels, allowing the model to capture finer local visual details (such as edges of small objects or strokes of characters). This is particularly important for tasks requiring fine-grained perception (OCR, fine-grained classification, chart understanding). Conversely, a larger Patch Size "packs" a broader range of pixels into the same patch, causing details to be averaged out or blurred, making it easy to lose critical information when handling small objects or dense text.

**Impact on efficiency**: Since the computational complexity of Self-Attention is proportional to the square of sequence length, doubling the number of patches increases computational overhead by 4 times. Therefore, while a smaller Patch Size enhances detail capture capabilities, it significantly increases computational load and memory footprint, as well as prolonging inference latency.

**Practical trade-offs**: Mainstream pre-trained visual encoders usually offer multiple Patch Size versions (e.g., CLIP offers ViT-B/32, ViT-B/16, ViT-L/14, etc.). Applications favoring general understanding or sensitive to latency can opt for larger Patch Sizes (such as /32, /16); applications leaning toward fine-grained understanding (documents, charts, dense small object detection) tend to choose smaller Patch Sizes (such as /14), or directly increase input resolution rather than decreasing Patch Size to indirectly boost detailed information (both increase the number of patches, but increasing resolution is typically easier to adapt with pre-trained weights via positional encoding interpolation).

</details>

**13. Why do some models (such as Fuyu-8B) attempt to remove the independent visual encoder and directly feed image patches into the language model?**

<details>
<summary>💡 Show Answer</summary>

The architecture of the vast majority of mainstream VLMs follows "pre-trained visual encoder (such as CLIP ViT) + connector + language model." Although this design reuses powerful pre-trained visual representations, it also introduces inherent limitations: visual encoders are usually pre-trained at fixed, relatively low resolutions, and processing images beyond this resolution range requires additional engineering work such as tiling or interpolation. Furthermore, the pre-training objective of visual encoders (such as CLIP's image-text contrastive objective) may not be completely aligned with the ultimate multimodal task objective, potentially leading to a "representation mismatch" issue.

Models such as Fuyu-8B adopt a different route: **completely removing the independent visual encoder** and directly feeding raw pixel patches cut from images (after simple linear projection) alongside text tokens as-is into the language model (which itself is a pure decoder-only Transformer), allowing the language model to learn how to understand these pixel signals from scratch. The motivations behind this design include:

- **Ultimate architectural simplification**: Eliminates the need to separately train or maintain a visual encoder or design dedicated connector modules; the entire model is a unified Transformer;
- **Native support for arbitrary resolutions**: Unconstrained by the inherent input size limitations of pre-trained visual encoders, it can theoretically process images of arbitrary resolutions and aspect ratios directly without resizing or complex dynamic tiling strategies;
- **More thorough end-to-end unified modeling**: Visual understanding capabilities "emerge" entirely during the language model's autoregressive training process, rather than relying on external pre-trained modules, adhering more closely to the unified modeling philosophy of "one model learning everything."

However, the trade-off of this approach is losing the general visual priors already learned by pre-trained visual encoders and validated on massive datasets. The model must rely on much larger scale multimodal training data to learn basic visual pattern recognition capabilities from scratch, resulting in significantly higher training difficulty and data requirements.



![基于视觉编码器与无编码器（如Fuyu）VLM架构对比图 (Comparison between visual-encoder-based and encoder-free VLM architectures.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig14_encoderless_vlm.png)

</details>

**14. What roles do the "CLS Token" and "Patch Tokens" in the visual encoder play in multimodal tasks?**

<details>
<summary>💡 Show Answer</summary>

In ViT's input sequence, in addition to the tokens corresponding to each patch cut from the image (Patch Tokens), a special, randomly initialized learnable vector is typically prepended: the **CLS Token** (Class Token). This design inherits from text Transformer models such as BERT.

**Role of CLS Token**: Because it undergoes multi-layer Self-Attention computations alongside all Patch Tokens, it continuously "absorbs" information from every patch. Therefore, at the final layer, the output vector corresponding to the CLS Token is usually regarded as a **global semantic summary** of the entire image. It is suitable for direct application in tasks requiring "overall judgment," such as zero-shot image classification (determining the category of an entire image) and global similarity computation in image-text retrieval (CLIP uses the CLS Token output as the representation vector for the whole image).

**Role of Patch Tokens**: Each Patch Token primarily retains fine-grained features corresponding to its local image region during encoding (while also integrating certain global context via Self-Attention). It is suitable for tasks requiring **spatial localization and local details**, such as visual grounding (needing to know where an object is located in the image) and fine-grained visual question answering (needing to focus on details of a specific local region in the image).

In multimodal large model practice, most VLMs (such as LLaVA) choose to retain **all (or most) Patch Tokens** rather than using only a single CLS Token vector when feeding visual features into the language model. This is because the language model needs rich local details of the image to answer various specific questions (rather than merely making a vague global assessment of the image). Discarding Patch Tokens to use only the CLS Token would result in significant loss of detail.

</details>

**15. How to handle the mismatch between input image resolution and pre-set training resolution through positional encoding interpolation?**

<details>
<summary>💡 Show Answer</summary>

During the pre-training phase, ViT is typically fixed at a specific resolution (such as 224×224), where the image is cut into a fixed number of patches (e.g., 16×16). Correspondingly, the model learns a set of positional encodings that perfectly matches this number of patches (whether learnable absolute positional encoding matrices or certain relative positional encoding schemes). When higher-resolution images need to be processed during inference (corresponding to a larger number of patches, e.g., 32×32 patches), the "size" of the pre-trained positional encoding matrix (corresponding to a 16×16 grid) cannot directly cover the new, larger patch grid (32×32).

A common solution is to perform **interpolation** on the positional encoding matrix: conceptually restore the original positional encoding matrix into a 2D grid (e.g., 16×16 positional vectors), and then use standard image processing interpolation algorithms (Bilinear or Bicubic interpolation) to "stretch" or "compress" this 2D grid to the target larger (or smaller) grid size (e.g., 32×32). Each interpolated position receives a reasonable new position vector smoothly transitioned from surrounding original positional encodings. This technique was first proposed in the ViT paper for resolution adaptation in downstream tasks and has since been broadly adopted in various VLMs supporting dynamic/high-resolution inputs, allowing models to generalize relatively smoothly to unseen resolutions without retraining positional encodings.

</details>

**16. How to design positional encodings to distinguish both spatial and temporal dimensions for multi-frame image (video) inputs?**

<details>
<summary>💡 Show Answer</summary>

When processing video data that possesses both spatial (row and column positions within the same frame) and temporal (sequential order across different frames) dimensions, pure 2D spatial positional encodings are insufficient for the model to distinguish between "same position in different frames" and "different positions in the same frame." Thus, positional encoding schemes capable of expressing both dimensions simultaneously must be specifically designed. Common approaches include:

- **Decoupled additive/concatenation scheme**: In addition to a 2D spatial positional encoding describing its row and column location within the current frame, each visual Patch Token is attached with a temporal positional encoding specifically indicating its frame index. The two are combined via addition or concatenation into the token's final complete positional representation, enabling the model to simultaneously perceive "spatial structure" (which patches are adjacent within the same frame) and "temporal structure" (the sequential order across different frames);
- **Unified multi-dimensional rotary positional encoding**: Such as the M-RoPE scheme in Qwen2-VL, which explicitly decouples positional encodings into three independent components: time, height, and width. For each patch in a video, values are assigned to the three components based on its frame index, row coordinate, and column coordinate within the frame. A single rotary positional encoding framework naturally and uniformly handles images (where the temporal component is constant), videos (where all three components vary), and even pure text (where all three components degenerate into equivalence, identical to 1D RoPE).

The core goal of such designs is to enable the model to apply reasonable relative positional biases based on the "spatial distance" and "temporal distance" between two tokens during Attention computation, thereby accurately modeling the spatial relationships and temporal dynamic changes of objects in videos.

</details>

**17. What is the Pixel Shuffle operation? How is it used to compress visual tokens in models like Qwen2-VL / InternVL?**

<details>
<summary>💡 Show Answer</summary>

Pixel Shuffle was originally an upsampling operation in image super-resolution tasks: it reshapes a feature map with many channels and lower spatial resolution by transferring information from the "channel dimension" to the "spatial dimension," thereby increasing the spatial resolution of the feature map (i.e., reducing channel count while expanding height and width) without introducing extra parameters or performing interpolation.

In visual token compression for multimodal large models, the exact opposite is done—the Pixel Shuffle operation is used **in reverse**: visual features with high spatial resolution (a large number of patches) and fewer channels are reshaped to transfer information from the "spatial dimension" into the "channel dimension," thereby reducing spatial resolution (decreasing patch/token count) while correspondingly expanding each token's feature dimension. For example, folding 2×2 spatially adjacent Patch Tokens into 1 token expands that token's feature dimension by 4 times, directly compressing the number of visual tokens to one-fourth. Unlike simple average pooling, this does not directly discard information—theoretically, all original feature information is preserved, merely rearranged into fewer tokens of higher dimensions, after which a subsequent linear layer projects the dimension back to the size required by the language model.

This operation is widely adopted by models such as Qwen2-VL and InternVL. It represents an efficient, virtually lossless compression technique that requires no additional attention computation overhead to address the "visual token explosion caused by high-resolution or multi-image inputs." Compared to compression methods like Q-Former that require extra cross-attention module training, it is considerably lighter.



![使用空间到通道像素重组（Pixel Shuffle）压缩视觉Token的矩阵图 (Downsampling visual tokens using the spatial-to-channel Pixel Shuffle operation.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig15_pixel_shuffle.png)

</details>

