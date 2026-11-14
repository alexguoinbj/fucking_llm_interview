# 12.11 Multimodal Deployment and Engineering Practice — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 18 questions

**1. What Considerations Are Important for Multimodal Model Deployment?**

<details>
<summary>💡 Show Answer</summary>

1. **Image Preprocessing Performance**: Image decoding, resizing, and normalization should be performed asynchronously on the CPU to prevent GPU idle waiting; during batch inference, ensure image size alignment (via padding or dynamic bucketing).
2. **KV Cache Management**: Visual tokens are numerous (e.g., 256--2048), leading to high KV Cache memory consumption; use Prefix Caching to reuse KV Cache for the same image (in multi-turn conversation scenarios over the same image).
3. **VRAM Optimization**: Apply INT4/INT8 quantization (AWQ, GPTQ) and FlashAttention to reduce VRAM usage; vision encoders and language models can be distributed across different GPUs.
4. **Dynamic Resolution Handling**: Deployment must support inputs of varying resolutions, ensuring padding does not introduce extraneous computation, or leveraging dynamic batch grouping.
5. **Latency and Throughput**: Image encoding and language generation represent the two main time-consuming stages; disaggregate prefill-decode deployment can be used; during batching, schedule requests of different lengths reasonably.
6. **Safety Filtering**: Perform NSFW detection on input images and conduct content safety auditing on outputs.
7. **Multimodal Input Validation**: Rigorously validate image formats, size limits, and file sizes to prevent abnormal inputs from crashing the service.

</details>

**2. Where Are the Capability Boundaries of Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

1. **Fine-Grained Visual Understanding**: Understanding small targets, dense text, and complex charts remains limited; when an image contains numerous details, the information loss caused by patchification is significant.
2. **Spatial Reasoning**: The model's understanding of 3D spatial relationships, directions, and distances is relatively weak, often generating incorrect spatial descriptions.
3. **Counting Ability**: Accurately counting the number of objects in an image remains a difficult challenge, especially with significant errors when the quantity is large.
4. **Cross-Frame Long Video Understanding**: Constrained by context length and computational cost, fine-grained understanding of videos lasting more than several minutes remains difficult.
5. **Hallucination Control**: It cannot completely avoid "fabricating out of thin air" non-existent content, making it unreliable in high-precision scenario requirements (such as medical or legal domains).
6. **Multimodal Chain-of-Thought Reasoning**: Complex multi-step image-text reasoning (such as mathematical proof problems and complex flowchart reasoning) still has substantial room for improvement.
7. **Knowledge Timeliness**: Model knowledge is frozen at the training cutoff date, making it impossible to update in a timely manner regarding the latest events and newly emerging visual concepts.
8. **Cross-Modal Consistency**: Maintaining consistency in descriptions of the same image details across multi-turn conversations remains challenging.

</details>

**3. How to Trade Off Between a "Unified Large Model" and a "Pipeline of Multiple Specialized Small Models" Architecture in Industrial Applications of Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

In practical industrial applications, when facing complex multimodal application requirements, there are usually two distinctly different technical route choices: one is using a **single unified multimodal large model** to complete the entire task end-to-end; the other is decomposing the task into multiple sub-steps, using a pipeline composed of **multiple specialized small models** to complete them (for example, first using a dedicated object detection model to locate objects, then a dedicated OCR model to recognize text, and finally a small classification model to determine the outcome). Both routes have their pros and cons, requiring a comprehensive tradeoff based on the specific scenario:

**Advantages of Unified Large Models**: Being trained and inferred end-to-end, they avoid the problem of **error accumulation and propagation** common in pipeline architectures (for instance, if the OCR module in a pipeline misrecognizes text, this error directly affects all subsequent steps and is difficult for downstream stages to perceive and correct); additionally, deployment and maintenance are relatively simple, requiring only the management and iteration of a **single** model service without needing to coordinate and maintain complex interfaces and data flow logic across multiple independent modules.

**Advantages of Specialized Small Model Pipelines**: Each sub-module can be **independently optimized and rapidly iterated for its specific sub-task**. For example, when a certain stage underperforms, only that single small module needs to be improved and retrained, without needing to retrain the entire massive unified model. The overall computational cost may also be lower, as many sub-tasks are inherently simple and can be efficiently solved by a lightweight specialized small model, making it completely unnecessary to invoke a massive multimodal large model with huge parameter sizes and expensive inference costs. Furthermore, pipeline architectures make it **easier to locate issues**—when the final output contains an error, one can inspect each stage of the pipeline step-by-step to identify which specific component failed, whereas a unified large model acts as an integrated "black box," making it often difficult to pin down which internal step caused the error.

**Considerations for Practical Tradeoffs**: Task complexity (the more a complex task requires synthesizing multifaceted information for deep reasoning, the more it favors the end-to-end understanding capability of unified large models); sensitivity to latency and cost (scenarios demanding ultra-high real-time performance and massive request volumes may prefer lightweight specialized model pipelines to control costs); the team's own technical iteration capabilities (whether they have the capability to continuously maintain and optimize a unified large model or are better at maintaining multiple independent traditional model modules); and the ease of data acquisition (end-to-end training of a unified large model often requires annotated data covering the entire task chain, whereas annotated data for each independent module in a pipeline is relatively easier to collect separately).

</details>

**4. What is an OCR-Free Document Understanding Model? What Advantages Does it Have Compared to the Traditional OCR+NLP Pipeline?**

<details>
<summary>💡 Show Answer</summary>

Traditional document understanding workflows are typically divided into two independent stages: first, a dedicated OCR (Optical Character Recognition) module detects and recognizes the text content in the document image, converting the image into plain text; then, this text is handed over to a downstream Natural Language Processing (NLP) model for tasks such as comprehension, question answering, or summarization.

OCR-Free models take a completely different approach: **bypassing the independent OCR recognition stage**, they directly feed the complete document image (including all visual information such as text, tables, figures, and layout) as a whole into the vision encoder and subsequent language model. The model directly understands document content and completes corresponding tasks end-to-end, without an explicit intermediate step of generating "plain text."

The advantages of this end-to-end approach over the traditional OCR+NLP two-stage workflow are mainly reflected in:

- **Avoiding the Propagation of OCR Recognition Errors**: In the traditional workflow, if the OCR module itself makes recognition errors (e.g., misreading blurry fonts or omitting certain text), these errors are directly passed to the downstream NLP model. Downstream models are typically unable to perceive or correct such upstream-introduced errors, ultimately leading to incorrect comprehension results. OCR-Free models lack such an independent intermediate conversion step that could introduce errors, theoretically avoiding this issue of **error accumulation and propagation**;
- **Better Preserving and Utilizing Document Visual Layout Information**: Besides textual content itself, documents contain rich semantic information implicit in their **visual layout**—such as the row-column structural relationships in tables, the importance hierarchy implied by larger font sizes or bold styles in headings, and the spatial correspondence between text and surrounding images/charts. Traditional OCR pipelines usually extract only a linear sequence of text, losing these vital visual layout cues. In contrast, OCR-Free models directly process full image inputs, theoretically leveraging these visual layout signals more effectively to aid understanding. This advantage is particularly prominent for document types where layout structure itself carries crucial semantics, such as tables, invoices, and resumes.



![传统“文字提取+NLP解析”双层流水线与原生免OCR直接视觉理解对比图 (Traditional OCR-plus-NLP text parsing compared to direct OCR-free visual reasoning.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig39_ocr_free_vs_pipeline.png)

</details>

**5. How Do Multimodal Models Support "Scene Text" Understanding in Images?**

<details>
<summary>💡 Show Answer</summary>

Scene Text refers to textual content appearing in natural scene photographs, such as place names on street signs, store names on shop boards, and explanatory text on product packaging. Such text often features variable shooting angles, diverse font styles, and complex background environments, making its recognition and understanding significantly more difficult than neatly structured, clear document scans.

The capability of multimodal models to support scene text understanding mainly relies on the following aspects:

- **Sufficient Resolution and Fine-Grained Feature Extraction Capability**: The vision encoder needs to possess the capability to capture the shapes of small, dense text strokes. This typically requires higher input resolution (or smaller patch sizes, as mentioned earlier); otherwise, text regions tend to become blurry and hard to distinguish during encoding due to insufficient resolution;
- **Abundant Targeted Training Data**: Training datasets need to include a large number of image-text pairs containing scene text in real-world environments (e.g., photos containing street signs, billboards, product packaging, accompanied by annotations accurately describing the text content in the image), allowing the model to thoroughly learn visual presentation patterns of text under various complex scenarios during training;
- **Specialized Text Detection/Recognition Auxiliary Training Objectives**: Some models introduce dedicated text detection (locating specific positions where text appears in images) or text recognition (identifying exact textual content) as auxiliary training tasks during multimodal pre-training or fine-tuning stages. Reinforcing the model's perception and understanding of scene text through such targeted supervisory signals avoids relying entirely on generic image-text alignment objectives to learn this capability "by the way."

</details>

**6. How Do Multimodal Large Models Process Structured Visual Information Such as "Charts/Tables"?**

<details>
<summary>💡 Show Answer</summary>

Comprehending content like charts (e.g., bar charts, line graphs, pie charts) and tables imposes unique requirements on models that differ from ordinary natural image understanding: the model must not only recognize the **visual elements** themselves within the chart (such as the relative height of a bar, the trend of a line, or the relative size of slices in a pie chart), but more crucially, it needs to understand the **precise numerical meanings** and **axis/legend information** behind these visual elements (for instance, the exact numerical value represented by a specific bar, or what the x- and y-axes denote). This requires the model to establish an accurate mapping between "visual presentation forms" and "underlying structured data," rather than merely staying at a vague perceptual level of overall chart visual styles.

To endow the model with this capability, training datasets typically need to incorporate large volumes of specialized chart/table-related image-text or question-answering pairs. One particularly effective data construction method is **programmatic synthesis**: automatically plotting various types of charts (bar charts, line graphs, etc.) via code. Because they are programmatically generated, the underlying true values corresponding to each visual element can be **precisely recorded** alongside the generated chart images, resulting in training data with absolutely accurate annotations that can be mass-produced at scale. This synthetic data construction approach of "naturally obtaining fully accurate annotations while generating images" is a unique advantage of chart/table understanding tasks compared to natural image understanding tasks (where natural image annotations usually cannot achieve complete programmatic precision), and serves as an important data source for mainstream approaches training models to learn the "visual pattern–underlying numerical value" mapping.

</details>

**7. Besides Image Captioning and Visual Question Answering (VQA), What Other Cutting-Edge or Promising Application Directions Exist for VLMs?**

<details>
<summary>💡 Show Answer</summary>

1. **Multimodal Dialogue Systems and Personal Assistants**: Users can send images or screenshots and engage in multi-turn, in-depth conversations with the assistant around these visual inputs (e.g., "Look at this photo of my fridge, what can I cook for dinner tonight?").
2. **Visual Grounding and Instruction Execution (Visual Grounding & Grounded Agents)**: VLMs not only understand image content but can also locate and operate on objects within images, used for UI automation (e.g., "Click that blue button labeled 'Submit'") and Embodied AI (acting as a robot's "brain" to plan and execute actions based on camera feeds and instructions).
3. **Specialized Domain Visual Analysis Assistants**: Medical imaging analysis (assisting in interpreting X-rays, CT scans), industrial quality inspection (analyzing product images in real-time to detect defects), and insurance loss assessment (evaluating damage severity based on accident photos).
4. **Content Creation and Code Generation**: WYSIWYG webpage/app generation (uploading design sketches to generate front-end code), chart and data visualization interpretation ("reading" complex charts to extract key information).
5. **Education and Accessibility Assistance**: Real-time scene description (describing surrounding environments for visually impaired individuals) and interactive learning (taking photos of problem sets to provide step-by-step explanations).

</details>

**8. Looking to the Future, What Unique Challenges Do Multimodal Large Models Face in "Embodied AI" Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Embodied AI refers to enabling AI systems (typically deployed on physical entities such as robots) to perceive real physical environments and take actions to interact with them. Compared to current mainstream multimodal large model applications centered around dialogue and content comprehension/generation, Embodied AI scenarios present several unique and far more severe challenges:

- **Extreme Requirements for Inference Latency**: Embodied AI systems must continuously process incoming streams of sensory inputs (visual feeds, possibly alongside tactile and force-feedback signals) and generate real-time action decisions—for example, a robotic arm during object manipulation must output control commands continuously at frequencies far higher than conversational response speeds (potentially at the millisecond level). Any noticeable inference latency could lead to action failure or safety hazards, imposing stringent demands on model inference efficiency far beyond typical application scenarios;
- **Need to Truly Understand Physical Causal Laws and Spatial Relationships**: Conversational multimodal applications rely primarily on **semantic associations** between images and text (e.g., identifying objects in an image, describing scene contents). In contrast, Embodied AI tasks require models to truly comprehend the physical world's **causal laws** (e.g., how pushing an object will cause it to move, how an object's weight and friction affect manipulation difficulty) and precise **spatial relationships** (millimeter-level position and pose estimation). This deep understanding of underlying physical principles is far more difficult than semantic image-text associations and represents a key capability gap that current "world model" research aims to bridge;
- **Dramatic Shift in Training Data Modality and Acquisition Cost**: Current multimodal large models rely heavily on static, massive internet image-text pairs for training. In contrast, training data required for Embodied AI tasks consists fundamentally of "observation-action" paired **interaction trajectory data**—data that must be generated by letting agents (whether physical robots or virtual agents in simulation environments) actually interact with environments. Such data cannot be scraped at scale from the internet like image-text data, making data collection costs extremely prohibitive;
- **The Sim-to-Real Gap Between Simulated Environments and the Real World**: Because real-world data collection is cost-prohibitive, much research chooses to generate large-scale interaction data in simulation environments for training. However, no matter how refined a simulation physics engine is, it struggles to fully replicate all the complexities of real-world physical laws and sensor noise. As a result, models trained in simulation often suffer significant performance degradation when transferred to perform tasks in the real physical world. How to narrow this sim-to-real gap remains one of the long-standing core challenges in Embodied AI.



![虚拟大模型智能体与实体物理环境交互的核心技术障碍结构图 (Interaction barriers between virtual cognitive models and physical dynamic environments.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig40_embodied_challenges.png)

</details>

**9. Have You Conducted Fine-Tuning Related to VLMs? What Models Did You Use?**

<details>
<summary>💡 Show Answer</summary>

(This question assesses practical experience; answers should be framed around specific projects. If experience is limited, one can clearly outline a complete conceptual workflow. Below is an example response.)

**Project Background and Objectives**: Utilizing the LLaVA-1.5 model to address a visual defect detection and classification task in a specific industrial domain, with the goal of building an intelligent assistant capable of conversing with quality control inspectors—after uploading product images, obtaining accurate answers through natural language queries (e.g., "What defect is in this image?", "Where is the defect located?").

**Model Selection**: Selecting LLaVA-1.5 (7B) as the base model for reasons including: a mature architecture ("ViT + linear projection + Vicuna" is the mainstream architecture for open-source VLMs, easy to understand and modify); strong foundational general visual dialogue capabilities requiring only domain knowledge injection; and a thriving open-source ecosystem with abundant turn-key fine-tuning scripts and community support.

**Fine-Tuning Strategy**:
1. **Data Preparation**: Constructing a small-scale, high-quality visual instruction dataset where each sample contains a defective industrial product image, diverse instruction templates mimicking inspector queries, and meticulously written standard answers.
2. **Fine-Tuning Strategy**: Employing LoRA for parameter-efficient fine-tuning on the LLM component, while keeping the vision encoder (CLIP ViT) and connector (MLP) frozen, since base visual representation capabilities are already sufficient and the primary goal is teaching the LLM to describe visual features using domain terminology.
3. **Training and Evaluation**: Training for a few epochs on a single A100 GPU; beyond text similarity metrics, evaluation places greater emphasis on manual assessment of answer professionalism, accuracy, and grounding capability.

**Challenges and Insights**: The main challenge lies in the high acquisition cost of high-quality annotated data. However, even with only a few hundred high-quality domain instruction samples, the model's performance on specific tasks improves significantly, demonstrating the critical role of visual instruction tuning in VLM domain adaptation.

</details>

**10. What Unique Challenges Does On-Device (Mobile/Edge Devices) Deployment of Multimodal Models Face?**

<details>
<summary>💡 Show Answer</summary>

- **Severely Constrained Compute and VRAM**: Chip compute power and available memory on mobile and edge devices lag orders of magnitude behind cloud GPU servers. They typically cannot directly execute full models with billions of parameters, necessitating heavy quantization compression (e.g., compressing model weights from FP16 to INT4/INT8) and knowledge distillation (training much smaller student models to imitate large model behavior) to barely meet deployment conditions;
- **Hardware Operator Adaptation Issues**: Many operators used inside vision encoders and language models (especially Attention calculations and certain special normalization layers) require operator-level optimization and adaptation for mobile NPUs (Neural Processing Units). Implementations that run efficiently on cloud GPUs might not be equally efficient on NPUs, or certain operators might not be supported at all, requiring additional engineering adaptation work;
- **Power Consumption and Thermal Restrictions**: Mobile devices have limited battery capacity and thermal dissipation capabilities, making them unable to support long-duration, high-intensity continuous inference computation like data centers. This restricts the trade-off space for on-device models between inference speed and battery life;
- **Dedicated Lightweight Training Rather Than Simple Compression**: Practice shows that applying "post-hoc compression" (quantization, pruning) directly to a large model designed for the cloud often performs worse than **specifically designing and training** a smaller-scale model architecture from scratch tailored to on-device compute budgets (such as MiniCPM-V mentioned earlier). Dedicated training allows better reallocation and optimization of capability distributions under limited parameter budgets, rather than merely compressing a model "designed for higher compute capacity."

</details>

**11. How to Design Multimodal Model APIs/Service Interfaces to Support Efficient Transmission of Large Files Such as Images and Videos?**

<details>
<summary>💡 Show Answer</summary>

- **Support URL References Instead of Mandatory Uploads**: Allowing clients to directly supply remote URLs for images/videos for the server to download independently, rather than forcing clients to encode full file contents into Base64 strings embedded in the request body—Base64 encoding typically bloats data size by ~33%, which significantly increases request transfer latency and bandwidth consumption for larger image or video files;
- **Chunked Uploads and Resumable Transfers**: For large files that indeed require direct client uploads (especially videos), employing chunked upload mechanisms (slicing files into smaller data blocks for individual upload) and supporting resumable transfers (allowing resume from the last successfully uploaded chunk after network interruptions without re-uploading the entire file), thereby enhancing transfer reliability in unstable network environments;
- **Server-Side Preprocessing and Feature Caching**: Servers can perform unified preprocessing (decoding, resizing, format conversion) on input images/videos and cache previously computed visual features—for example, when the same image is repeatedly referenced across multi-turn dialogues, the visual encoder computation pipeline does not need to be re-executed every time; cached visual tokens can be directly reused, significantly reducing computational overhead and response latency for repeated requests;
- **Asynchronous Task Interfaces**: For time-consuming video processing requests, designing asynchronous task modes is appropriate—clients receive a task ID immediately upon submission while the server processes the task in the background. Clients retrieve results upon completion via polling task status or registering webhooks, avoiding long synchronous blocking while waiting for responses. This design is far more friendly for improving overall system concurrency and user experience.

</details>

**12. What is Special About Model Compression (Distillation, Pruning) for Multimodal Large Models Compared to Text-Only LLMs?**

<details>
<summary>💡 Show Answer</summary>

In addition to reusing general techniques mature in text-only LLMs—such as Quantization, structured/unstructured Pruning, and Knowledge Distillation (applied to their language model components)—multimodal model compression requires extra consideration of compressing vision encoders and connector modules, as well as performance bottleneck sources unique to multimodal scenarios:

- **Special Considerations for Vision Encoder Compression**: When distilling or pruning vision encoders, the compressed model must still extract sufficiently fine-grained, semantically rich visual features. Because the vision encoder serves as the origin of the entire multimodal system's "perceptual capability," any degradation in its representation capability directly and irreversibly impairs performance across all downstream tasks relying on visual information. Consequently, vision encoder compression usually demands more conservative and cautious validation of performance loss than language model components;
- **Dual Sources of Performance Bottlenecks**: Inference performance bottlenecks for text-only LLMs stem primarily from model parameter counts (which dictate compute load and VRAM footprint). In contrast, multimodal model performance bottlenecks exist in two dimensions: **parameter count** and **number of visual tokens**. Because self-attention computational complexity grows quadratically with sequence length, processing high-resolution images or multi-image/video inputs leads to a surge in visual tokens; even if model parameter counts remain constant, inference compute overhead spikes dramatically;
- **Token-Level Compression as a Unique Acceleration Method for Multimodal Models**: Due to the above characteristics, besides traditional parameter-focused compression techniques, **reducing the number of visual tokens itself** (such as the aforementioned Token Merging/Pruning, Pixel Shuffle, etc.) often yields more direct and significant inference acceleration in multimodal scenarios than solely compressing model parameters. This represents an optimization dimension unique to the multimodal field that is absent in text-only LLM compression work.



![保持跨模态表征对齐条件下的VLM蒸馏与剪枝压缩路径图 (Aligned distillation constraints for compressing vision-language models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig41_multimodal_compression.png)

</details>

**13. What Are Typical Deployment Modes for Multimodal Large Models in Customer Service and E-Commerce Scenarios?**

<details>
<summary>💡 Show Answer</summary>

- **Automated Generation of Product Content**: Automatically generating marketing copy and product detail page descriptions aligned with platform styles and SEO requirements based on merchant-uploaded product images, significantly reducing labor costs for merchants writing copy;
- **Image-Based Search for Identical/Similar Items**: Users upload an image of a product of interest (e.g., an outfit seen on social media), and the system uses multimodal models to extract visual features to search the catalog for visually similar or identical items on sale—a vital feature for boosting purchase conversion rates in e-commerce;
- **After-Sales Quality Inspection and Return/Exchange Assistance**: Users upload photos of physical items, and the system automatically identifies whether defects or damages exist, or determines whether item appearance meets platform return/exchange policy requirements, assisting (or partially replacing) human customer service in initial after-sales evaluations to increase processing efficiency;
- **Intelligent Customer Service Multimodal Q&A**: Combining user chat logs with uploaded screenshots/photos (e.g., a user sending an App error prompt screenshot), intelligent customer service systems can comprehend specific error details within screenshots and provide targeted solutions in context, sparing users the effort of describing screenshot contents in text;
- **Batch Generation of Marketing Assets**: Leveraging existing product image libraries combined with text-to-image/image-editing models to automatically generate marketing asset images across various styles and scene backgrounds (e.g., automatically placing product images into different ambient backgrounds), lowering asset creation costs for marketing teams.
These scenarios usually do not require training entirely new multimodal large models from scratch. Instead, deployment adopts the approach of "general open-source or commercial multimodal large models + fine-tuning with a small volume of high-quality instruction data tailored to specific business scenarios (typically combined with parameter-efficient methods like LoRA)," enabling effective adaptation to specific business requirements while controlling development costs.

</details>

**14. How Does Cost Accounting (Token Pricing) for Multimodal Large Models Differ From Text-Only Models?**

<details>
<summary>💡 Show Answer</summary>

Pricing for text-only large models is relatively straightforward—charging separately based on the number of input and output text tokens, allowing users to estimate API call costs easily based on text length.

Billing logic for multimodal large models is more complex: non-text inputs like images and videos must first be converted into an equivalent **"number of visual tokens"** during billing, and then billed into total costs based on the same (or different) unit price as text tokens. This conversion process involves several special aspects:

- **The Cost May Vary for "a Single Image"**: Because the number of visual tokens depends on image resolution (higher resolution generally splits into more patches/tokens), even for the identical action of "uploading an image," a high-resolution image and a low-resolution image can differ by several times in visual token count billed, resulting in noticeable cost differences;
- **Video Costs Are Typically Significantly Higher**: Since video is inherently a sequence of multi-frame images, processing requires feature extraction for each (sampled) frame separately. The equivalent token count is often tens of times or more compared to text content of equivalent duration. Consequently, processing video requests is generally far more expensive than processing a single static image;
- **Simplified "Tiered" Pricing**: To reduce complexity for users estimating costs, many service providers simplify billing by assigning images to several predefined resolution "tiers" (e.g., "low resolution" corresponding to a fixed smaller token count, "high resolution" to a fixed larger token count). Users only need to roughly estimate costs based on their selected tier without worrying about low-level implementation details such as how many visual tokens an image is split into.

</details>

**15. What is a VLA (Vision-Language-Action) Model? What Are its Essential Differences From Ordinary VLMs?**

<details>
<summary>💡 Show Answer</summary>

A VLA (Vision-Language-Action) model extends the output space of a VLM from "natural language text" to "robot-executable control actions" (such as robotic arm joint angles, end-effector poses, mobile chassis velocity, etc.). This enables the model not only to "see and understand" images and "comprehend" language instructions, but also to directly "generate" action sequences in the physical world, serving as a core technical route in Embodied AI.

Essential differences are mainly reflected in:

1. **Different Output Modalities**: VLM outputs are discrete language tokens (text), ultimately serving cognitive tasks like dialogue, captioning, and Q&A; VLA outputs are continuous or discretized action signals that directly drive physical actuators, imposing higher requirements on accuracy and real-time capability.
2. **Different Evaluation Standards**: VLM performance is typically evaluated by language-level accuracy and fluency; VLA performance is ultimately tested by task **success rate** in real (or simulated) physical environments—even if language comprehension and visual perception are correct, if action execution accuracy is insufficient (e.g., a grasp position off by a few millimeters), the task will still fail.
3. **Different Training Data**: VLMs rely primarily on web image-text pairs and human-annotated instruction data; VLAs, in addition to similar image/video-language data, must rely on "trajectory data" (observation-action pairs) collected when robots actually execute tasks. The collection cost of such data is vastly higher than purely web-scraped image-text data, forming one of the main bottlenecks in current VLA development.
4. **Pre-Training Requirements**: Some researchers argue that only models fully pre-trained on large-scale visual-language data before transfer-learning action generation capabilities can be strictly called VLAs; if a model directly trains joint visual-language-action mappings from scratch on limited robot trajectory data without large-scale vision-language pre-training, it is better categorized as a "Multimodal Policy" rather than a VLA.

</details>

**16. How Does RT-2 Transform a VLM into a VLA Model? What is its Core Concept?**

<details>
<summary>💡 Show Answer</summary>

The core concept of RT-2 (proposed by Google DeepMind) is "re-representing robot actions as special language tokens, thereby directly reusing off-the-shelf VLM architectures and training paradigms" without designing brand-new network structures for action output. Specific approaches include:

1. **Action Discretization**: Discretizing continuous control signals of the robot's end-effector (such as x/y/z position offsets, rotation angles, gripper open/close states) into a finite number of bins (intervals), with each bin assigned an integer ID. In this way, the continuous action space is converted into a finite set of discrete symbols similar to a vocabulary.
2. **Reusing the Language Model Vocabulary**: Directly mapping these discretized action IDs to rarely used tokens in the language model's existing vocabulary (or extending the vocabulary with dedicated action tokens), making "action generation" formally equivalent to "generating special text"—the model simply predicts these action tokens autoregressively one by one, just like normal text generation.
3. **Co-Fine-Tuning**: On top of a VLM already pre-trained on large-scale web image-text data, co-fine-tuning using two types of data simultaneously—one part being original web image-text data (preserving the model's general visual-language understanding capability to prevent catastrophic forgetting), and the other being trajectory data of "image + instruction $\rightarrow$ action token sequence" collected during robot task execution (learning specific action generation capabilities).
4. **Emergent Generalization Capabilities**: Because the underlying VLM has learned rich semantic and commonsense knowledge (e.g., recognizing various objects, understanding abstract instructions) from massive internet image-text data, RT-2 demonstrates generalization capabilities beyond the training distribution—such as correctly executing instructions like "move the object about to fall off the table edge to the middle of the table," which never appeared in robot training trajectories but require commonsense reasoning to comprehend.

This approach of RT-2 proves that without specifically designing a new "action decoder," simply through the clever representation transformation of "turning actions into a special language," powerful VLMs can be directly "upgraded" into VLA models capable of physical-world action.



![RT-2视觉-语言-动作（VLA）模型中的控制动作Token输出生成流程图 (Action token output generation pipeline in RT-2 Vision-Language-Action model.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig42_rt2_concept.png)

</details>

**17. What is the Common "Hierarchical" Architecture Design Concept in VLA Models? What is the FiLM Fusion Method?**

<details>
<summary>💡 Show Answer</summary>

**Hierarchical Architecture**: Facing the two distinctly different demands of "long-horizon task planning" and "fine-grained action control," many VLA models adopt a hierarchical design, splitting the system into:

- **High-level Planner**: Typically undertaken by a larger VLM with strong language reasoning capabilities, responsible for understanding complex, potentially multi-step task instructions (such as "clean up the plates on the table and put them into the dishwasher") and decomposing them into a sequence of simpler semantic sub-tasks (such as "pick up plate" $\rightarrow$ "open dishwasher" $\rightarrow$ "place plate inside").
- **Low-level Control Policy**: Receives current sub-tasks output by the high-level planner and, combined with real-time visual observations, generates concrete, high-frequency updated low-level control commands (such as joint angle increments). It typically operates at a frequency far higher than the high-level planner to satisfy response speed requirements for real-time control.
The benefit of this hierarchical design is allowing "semantic understanding and task decomposition" and "fine-grained motor control" to be implemented using model scales and update frequencies best suited to each, avoiding using a single massive and slow model to simultaneously undertake low-level control tasks requiring high-frequency response.

**FiLM (Feature-wise Linear Modulation)**: A lightweight vision-language fusion method adopted in some VLA control policies (such as RT-1). Its core idea is: rather than fusing the two modalities through complex Cross-Attention mechanisms, the output of the language encoder passes through a small network to generate a set of scale ($\gamma$) and shift ($\beta$) coefficients, directly performing element-wise linear transformations on each channel of the visual feature map (i.e., $output = \gamma \cdot visual_feature + \beta$, where $\gamma, \beta$ are generated from the language instruction). This approach is equivalent to using language instructions to "modulate" the response intensity of visual features, with computational overhead far lower than Cross-Attention, making it suitable for real-time robot control scenarios demanding low inference latency.

</details>

**18. What is a "World Model"? What Are its Essential Differences From Current Multimodal Large Language Models (MLLMs)?**

<details>
<summary>💡 Show Answer</summary>

A World Model refers to a model capable of internally building simulations/representations of physical world dynamics and, based on current state and given actions, **predicting future state changes** (formally represented as a state transition function: "current state + action $\rightarrow$ future state"). Its design intention is to allow agents to mentally simulate potential consequences of different decisions in "their heads" before making more rational action choices, without needing trial and error in the real world.

Compared with current Multimodal Large Language Models, their essential differences are mainly reflected in:

1. **Different Modeling Targets**: MLLMs essentially process highly abstract symbolic information (text) or semantic features compressed by pre-trained vision encoders, which have lost vast raw details of the physical world (such as lighting variations, material friction, and real object weight intuition). World models emphasize modeling raw data rich in physical signals directly (especially videos), attempting to capture true physical dynamics, kinematic laws, and causal relationships.
2. **Different Core Capabilities**: The core capability of MLLMs is understanding and generation—providing appropriate descriptions or answers based on existing information. The core capability of world models is **prediction and rollout**—given current state and a hypothetical action, predicting how the environment will change, thereby supporting planning and decision-making based on an "imagined future" (often combined with reinforcement learning).
3. **Whether Relying on Language as the Core Hub**: MLLMs (especially mainstream concatenated VLMs) typically use language models as their core backbones, "translating" other modal information into representations digestible by language models. In contrast, many world model research directions (such as JEPA-based self-supervised representation learning methods) deliberately do **not** rely on language as an intermediary, learning predictive representations directly in continuous sensory signal (video frame) spaces, positing that language abstraction and compression of physical world details inherently causes loss of significant valid information.
4. **Different Typical Application Scenarios**: MLLMs are mostly oriented toward cognitive tasks like dialogue and content understanding/generation. Typical application scenarios for world models center on autonomous driving (predicting future trajectories of other vehicles/pedestrians), robot simulation training (generating training data via low-cost trial and error in simulated environments), and game engine simulation (e.g., directly using neural networks to "imagine" and render next frames instead of traditional graphics computation)—domains requiring deep interaction with real physical dynamics.

It should be noted that "world models" and "multimodal large models" are not mutually exclusive, opposing technical routes, but rather complementary relationships: future Embodied AI systems will very likely rely simultaneously on language understanding and commonsense reasoning provided by MLLMs, alongside physical dynamic prediction capabilities provided by world models, working in tandem to support agent perception, decision-making, and action in the real physical world.



![静态图文推理大模型与主动式物理环境世界模拟器对比图 (Cognitive processing differences: Static QA models versus active world physics simulators.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig43_world_model_vs_mllm.png)

</details>

