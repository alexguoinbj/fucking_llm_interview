# 11.4 Multimodal and Embodied AI Applications — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. How Might Multimodal Unified Modeling Evolve in the Future?**

<details>
<summary>💡 Show Answer</summary>

Directions of development for multimodal unified modeling:

1. **Unified Tokenizer for Any Modality**: Tokenize all modalities such as images, audio, video, and point clouds into discrete tokens, sharing a unified vocabulary with text tokens to achieve true modality-agnostic modeling (e.g., AnyGPT);
2. **Unified Generation and Understanding**: A single model simultaneously supports both understanding (analysis) and generation (creation) across modalities, such as omni-modality unification in GPT-4o (Transfusion, Show-o);
3. **World Model Direction**: Multimodal models evolve into "world models" equipped with physical world understanding, capable of simulating environmental state changes to support robotics planning and autonomous driving (e.g., Genie, JEPA);
4. **Hybrid Modeling of Continuous and Discrete Spaces**: Mixing diffusion models (continuous space) and language models (discrete space) to balance high-quality image generation and language understanding;
5. **Native Multimodal Pre-training**: Shifting away from "language model + vision adapter" to unified pre-training from scratch on multimodal data, avoiding a stitched-together feel between modalities;
6. **Embodied AI Integration**: Deep integration of multimodal models with robot sensors and actuators to form a closed loop of perception, reasoning, and action.

</details>

**2. How Can Large Models Be Combined with Robotics?**

<details>
<summary>💡 Show Answer</summary>

Core directions of empowering robotics with large models:

1. **Natural Language Instruction Understanding**: Robots understand complex natural language instructions via LLMs ("Place the red cup to the right of the tray") and decompose them into executable subtask sequences.
2. **Vision-Language-Action (VLA) Models**: Representative models like RT-2, OpenVLA, and $\pi$0 uniformly model visual observations, language instructions, and robot actions, generating robot control signals directly from images and text.
3. **Task Planning**: LLMs serve as high-level Task Planners, decomposing high-level goals into low-level Motion Primitives, which are then executed by low-level controllers (SayCan, Code as Policies).
4. **Knowledge Transfer**: Leveraging commonsense knowledge accumulated by LLMs on internet data to enable robots to perform reasonable inferences in unseen scenarios ("Drawers are usually under tables").
5. **Multi-Robot Collaboration**: LLMs coordinate division of labor among multiple robots, carrying out task allocation and communication at the natural language level.
6. **Embodied AI Research**: Combining concrete physical interactions with abstract language reasoning to explore embodied cognition where "understanding is action."



![将高层规划与底层控制分离的机器人基座模型分层架构 (Hierarchical architecture of robot foundation models separating high-level planning from low-level control.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig47_robotics_foundation.png)

</details>

**3. What Role Can Large Models Play in Autonomous Driving?**

<details>
<summary>💡 Show Answer</summary>

Roles of large models in autonomous driving:

1. **Scene Understanding and Description**: Multimodal large models understand camera and LiDAR inputs to generate natural language descriptions of scenes ("A pedestrian is crossing the road ahead"), assisting decision-making;
2. **Decision-Making Reasoning**: After converting driving scenes into structured descriptions, LLMs perform high-level decision reasoning (whether to overtake, how to handle anomalous situations) and output driving intentions;
3. **Long-Tail Scenario Handling**: A core difficulty of autonomous driving lies in long-tail rare scenarios; the commonsense reasoning capability of LLMs helps address special cases not covered by training data;
4. **Driving Data Generation**: Using large models to generate diverse driving scene descriptions or simulation scene parameters to expand training data (e.g., CARLA simulation + LLM);
5. **Human-Machine Interaction**: In-cabin natural language interaction, driver intention understanding, voice navigation, etc.;
6. **End-to-End Autonomous Driving Enhancement**: Models like DriveVLM and UniAD combine VLMs with end-to-end driving models, utilizing intermediate language representations to improve interpretability and generalization capabilities.

</details>

**4. How Are Large Models Applied to Scientific Computing?**

<details>
<summary>💡 Show Answer</summary>

Applications of large models in scientific computing:

1. **Protein Structure Prediction**: AlphaFold 2/3 combines Transformers with biological sequence modeling, achieving revolutionary breakthroughs in protein folding; large models are further applied to protein design (ESM series).
2. **Materials Science**: Using large models to predict crystal structures and material properties; GNoME (Google DeepMind) discovered over 2 million new materials.
3. **Mathematical Theorem Proving**: AlphaProof, Lean Copilot, etc., combine LLMs with formal proof systems to assist in machine verification of mathematical theorems.
4. **Climate and Weather Forecasting**: Pangu-Weather, GraphCast, etc., replace traditional numerical weather prediction with Transformers, boosting speeds by a thousand times.
5. **Drug Discovery**: Molecular generation, drug-target affinity prediction; LLMs comprehend literature and propose hypotheses.
6. **Physics Simulation**: Using neural networks to learn the evolution laws of physical systems, replacing expensive numerical simulations (Physics-Informed Neural Networks).
7. **Automated Experiment Design**: Large models act as "scientific assistants" to interpret experimental results, formulate hypotheses, and design subsequent experiments (AI Scientist).

</details>

**5. What Is the Core Design Philosophy of Vision-Language-Action (VLA) Models?**

<details>
<summary>💡 Show Answer</summary>

VLA (Vision-Language-Action) models aim to enable robots to directly output executable action sequences from visual observations and language instructions. Their core design typically includes:

1. **Unified Tokenization**: Images, language, and robot actions (after discretizing continuous quantities such as joint angles/end-effector poses) are all encoded into unified token sequences, reusing the autoregressive framework of large models for modeling;
2. **Pre-training + Fine-tuning**: Perform vision-language pre-training on internet-scale image-text data first to acquire general perception and reasoning capabilities, then fine-tune action output heads on relatively limited real/simulated robot trajectory data;
3. **Hierarchical Structure**: The high level uses large models for task planning and commonsense reasoning (deciding "what to do"), while the low level uses lightweight, high-frequency controllers to execute concrete actions (deciding "how to do it"), balancing reasoning capability with real-time control requirements.

**Representative Works:** RT-2, OpenVLA, etc. The core challenge is that the scale of robot action data is far smaller than image-text data; how to fully activate the world knowledge and commonsense reasoning capabilities already present in large models using limited embodied data remains a critical difficulty in current research.



![视觉-语言-动作 (VLA) 模型架构中的端到端标记化与融合管线 (End-to-end tokenization and fusion pipeline in Vision-Language-Action (VLA) model architectures.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig50_vla_architecture.png)

</details>

**6. What Challenges Does the Application of Large Models Face in the Medical Domain?**

<details>
<summary>💡 Show Answer</summary>

1. **Accuracy and Hallucination Risk**: Medical decision-making has an extremely low error tolerance; model hallucinations could lead to severe consequences, necessitating strict fact-checking and traceable citation mechanisms;
2. **Data Privacy and Compliance**: Patient data involves strict privacy regulations (e.g., HIPAA). Both model training and inference must meet compliance requirements such as keeping data within local domains and anonymization;
3. **Timeliness of Professional Knowledge**: Medical guidelines and drug information are continuously updated. Knowledge in statically trained models easily becomes outdated, requiring real-time integration with the latest authoritative knowledge bases via mechanisms like RAG;
4. **Attribution of Responsibility**: Legal responsibility boundaries when model recommendations lead to misdiagnosis or adverse outcomes remain unclear. Currently, large models are mostly positioned as "assisting tools" rather than "diagnostic entities," with final decisions still requiring physician oversight;
5. **Difficulty of Multimodal Fusion**: Real-world medical scenarios involve highly heterogeneous multimodal information such as medical imaging (CT/MRI), laboratory data, and structured/unstructured medical records, making multimodal fusion modeling significantly more challenging than in general multimodal tasks.

</details>

**7. What Is the Technical Route of Sora-like Video Generation Models?**

<details>
<summary>💡 Show Answer</summary>

The core architecture is typically **Diffusion Model + Transformer (Diffusion Transformer, DiT)**: compressing videos into spatiotemporal patches (similar to image patches in ViT, but with an added temporal dimension) and performing diffusion denoising on these patches.

**Key Designs:**
1. **Joint Spatiotemporal Modeling**: Instead of simply treating video as a sequence of frame-by-frame images, spatiotemporal patches are uniformly encoded, enabling the model to capture motion consistency and long-term dependencies across frames;
2. **Variable Resolution/Duration Training**: Training directly on videos with native aspect ratios and durations (rather than uniformly cropping and scaling to a fixed size), enhancing model generalization capabilities and generation flexibility;
3. **Latent Space Compression**: First using a video VAE to compress the pixel space into a low-dimensional latent space, and then performing diffusion within the latent space, significantly reducing computational overhead.

**Challenges:** Temporal consistency in long videos (avoiding object "flickering" or incoherent motion logic), accurate modeling of physical laws (such as collisions, gravity, and fluid dynamics), and extremely high computational costs for training and inference remain the core difficulties in the current video generation field.



![用于视频生成的时空补丁压缩与去噪扩散 Transformer (DiT) 管线 (Pipeline of spatiotemporal patch compression and Denoising Diffusion Transformer (DiT) for video generation.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig52_sora_video_generation.png)

</details>

**8. How Are Tactile/Force Feedback Signals Integrated into Multimodal Large Models? What Are the Main Difficulties?**

<details>
<summary>💡 Show Answer</summary>

A vast amount of aligned image-text data exists between vision and language, but tactile/force feedback signals lack paired "tactile-language" data of comparable scale, which forms the most fundamental bottleneck in this direction.

**Integration Approaches:**
- **Dedicated Tactile Encoders**: Collect data via tactile sensors (such as the GelSight series of vision-based tactile sensors, which convert contact deformation into image signals), and then map tactile signals into a semantic space shared with vision/language using encoder architectures similar to vision encoders;
- **Cross-Modal Self-Supervised Alignment**: Utilize temporally aligned vision, force feedback, and action trajectory data simultaneously collected during the same manipulation process to perform self-supervised pre-training, enabling the model to learn causal relationships between "observed actions" and "felt force feedback";
- **Simulation Data Augmentation**: Physical simulation engines can generate large quantities of synthetic data with precise force annotations at low cost, mitigating the high cost and small scale of real tactile data collection; however, a domain discrepancy between simulation and reality (sim-to-real gap) persists.

**Core Difficulties:** Tactile signals are high-frequency, local, and strongly correlated with specific materials and geometries, making them difficult to standardize using unified image patch approaches like vision. Currently, this research direction remains in its early stages and has yet to establish a universal tactile-language alignment paradigm comparable to CLIP.

</details>

**9. What Core Problems Does Sim-to-Real Transfer Face in Embodied AI? What Are the Mitigation Approaches?**

<details>
<summary>💡 Show Answer</summary>

Training robot policies in simulation environments is low-cost, fast, and massively parallelizable. However, there is a systematic gap between simulation environments and the real physical world (the sim-to-real gap), causing direct transfer to real robots to often suffer a significant performance drop.

**Core Problems:**
- **Visual Domain Gap**: Simulation-rendered images have systematic differences from real camera-captured images in lighting, texture/material, and noise;
- **Physical Parameter Errors**: Physics engines often use simplified or inaccurate modeling for physical phenomena such as friction, contact collisions, and deformable object deformations, causing dynamic responses in the real world to significantly deviate from simulation predictions;
- **Sensor Noise and Latency**: Real sensors exhibit noise and latency characteristics that are not adequately modeled in simulation.
**Mitigation Approaches:**
- **Domain Randomization**: Randomize factors such as lighting, textures, and physical parameters over a wide range during simulation training, forcing the policy to learn features robust to these variations rather than overfitting to a fixed set of simulation parameters;
- **Domain Adaptation**: Fine-tune the simulation-trained policy using a small amount of real-world data, or train a discriminator to make simulation images and real images indistinguishable in feature space;
- **World Model / Video Generation Model Assistance**: As mentioned earlier, use world models that understand physical laws to generate synthetic training data closer to real-world distributions, closing the gap between simulation and reality at the source.

</details>

**10. What Are the Primary Causes of Hallucination in Multimodal Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

Multimodal hallucination refers to information generated in model descriptions or reasoning that contradicts the actual content of the input image/video (such as describing non-existent objects in the image or incorrect spatial relationships).

**Primary Causes:**
- **Overly Strong Language Priors**: The co-occurrence statistical patterns learned by the model on massive pure text datasets may override the visual evidence itself. For example, even if a "kitchen" image does not contain a refrigerator, the model might still "hallucinate" a refrigerator due to the language prior that "kitchens usually have refrigerators";
- **Information Loss in Visual Encoding**: After an image is encoded into a finite number of visual tokens, fine-grained spatial location, counting, and detailed texture information are easily lost during compression, causing the model to make errors on tasks requiring precise counting or spatial reasoning;
- **Imperfect Alignment of Text and Images in Training Data**: In web-scraped image-text pairs, text descriptions themselves may not be completely accurate or may contain information unrelated to the image, leading the model to learn unreliable image-text association patterns from such data;
- **Modality Gap in Modular/Stitched Architectures**: In traditional "independent visual encoder + language model" architectures, visual information must first be "translated" into representations resembling text tokens before being processed by the language model. This translation process itself serves as a potential bottleneck for information distortion, which is one reason why native multimodal architectures are considered promising for mitigating this issue.

</details>

**11. What Are the Advantages of Speech "Native Multimodal" Fusion Compared to the Traditional ASR+LLM+TTS Cascaded Scheme?**

<details>
<summary>💡 Show Answer</summary>

Traditional speech interaction solutions are cascaded: Automatic Speech Recognition (ASR) converts speech to text, the language model processes the text to generate a text response, and Text-to-Speech (TTS) synthesizes the text response into speech. These three modules are relatively independent and executed sequentially in series.

**Limitations of the Cascaded Scheme:**
- Paralinguistic information in speech such as intonation, pauses, emotions, and stress is almost entirely lost during the ASR text-transcription stage, rendering the language model completely incapable of perceiving it;
- The three-stage pipeline leads to accumulated end-to-end latency, making it difficult to support natural, real-time interactions such as interruptions and interjections;
- Errors accumulate and propagate down the pipeline (ASR recognition errors directly pollute all subsequent stages).
**Idea of Native Speech Multimodality:** Allow the model to directly perform comprehension and generation on continuous or discretized speech features (rather than transcribed text). Speech understanding, language reasoning, and speech synthesis are completed within a unified representation space inside a single model. The model can directly "understand" intonation variations and preserve or control these paralinguistic features when generating spoken responses, while supporting lower-latency streaming interaction (such as responding promptly when interrupted mid-speech). The trade-off is significantly increased training data construction and modeling complexity, requiring massive speech-text aligned datasets as well as decoding architectures supporting streaming speech generation.



![传统级联语音系统与原生语音对语音架构的结构对比 (Structural comparison between traditional cascaded speech systems and native speech-to-speech architectures.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig56_speech_native_vs_cascade.png)

</details>

**12. What Is the Primary Bottleneck in Data Acquisition for Embodied AI? What Are the Current Solution Paths?**

<details>
<summary>💡 Show Answer</summary>

Training embodied AI policies ideally requires massive real interaction data in the form of "environment state-action-outcome". However, collecting real robot data is extremely expensive (requiring physical hardware, human operation, or lengthy autonomous trial-and-error, while carrying a high risk of hardware damage).

**Evolution of Solution Paths:**
- **Teleoperation Collection**: Human operators remotely control robots to complete tasks and record trajectories. The data quality is high, but collection is slow and expensive;
- **Cross-Embodiment Data Aggregation**: Projects like RT-X aggregate datasets collected from multiple institutions and across diverse robot configurations for unified training, offsetting the scarcity of single-source data through scale;
- **Embodiment-Free Collection Devices**: Portable collection tools like UMI collect data closely matching the robot's manipulation viewpoint and actions without relying on a full robot hardware setup, significantly lowering the barrier to data collection and boosting data volume by one to two orders of magnitude;
- **Egocentric (Ego-View) Human Data**: Treat humans as "another type of robot" and use wearable sensors to record egocentric data of humans interacting with objects in real environments. The potential scale of this data far exceeds specialized robot data collection, making it one of the most promising avenues for scaling up, though effectively mapping human body actions into the robot's action space remains a key challenge;
- **World Model-Generated Synthetic Data**: As mentioned earlier, use physics-informed world models to generate synthetic interaction data with action annotations in bulk, particularly suited for populating long-tail scenarios (e.g., anomalous or hazardous situations) that are rare in real-world collection.

</details>

**13. What Common Characteristics Do the Design Paradigms of Robot Foundation Models (e.g., RT-X, $\pi_0$) Share?**

<details>
<summary>💡 Show Answer</summary>

A wave of "robot foundation models" has emerged in recent years, attempting to replicate the successful language model paradigm of "large-scale pre-training + small-scale fine-tuning for downstream generalization" in the field of robotic manipulation.

**Common Design Paradigms:**
- **Unified Action Space Across Embodiments and Tasks**: Different robot configurations vary in degrees of freedom and action representations. These models typically design a relatively universal action encoding scheme (such as discretized action tokens) so that data from diverse robot platforms can be learned by a single model;
- **Joint Vision-Language-Action (VLA) Modeling**: Model inputs usually contain both visual observations and language instructions, and outputs are directly executable action sequences, leveraging the semantic understanding capabilities of pre-trained vision-language models as a foundation for action generation;
- **Large-Scale Multi-Institutional Data Aggregation Training**: The competitiveness of these models largely stems from aggregating data across multiple laboratories and robot platforms. The scaling effect itself enhances cross-task and cross-scenario generalization, validating the thesis that "data scale is a primary driver of progress in embodied AI";
- **Few-Shot/Zero-Shot Adaptation to New Tasks**: After pre-training, the model usually requires only a small amount of fine-tuning data on target tasks (or even no fine-tuning, relying solely on language prompt guidance) to adapt to new manipulation tasks or new robot platforms. This represents the core value proposition of the "foundation model" paradigm over previous task-specific policy training paradigms.

</details>

**14. What Unique Challenges Does Multimodal Retrieval-Augmented Generation (Multimodal RAG) Face Compared to Text-Only RAG?**

<details>
<summary>💡 Show Answer</summary>

Multimodal RAG refers to retrieving from databases that include non-text content such as images, charts, and videos, requiring the model to generate answers based on evidence retrieved across modalities.

**Unique Challenges:**
- **Semantic Alignment in Cross-Modal Retrieval**: It requires constructing retrieval models capable of mapping text queries and image/chart content into a shared semantic space (similar to CLIP's cross-modal embeddings). Retrieval quality is directly constrained by cross-modal alignment precision—especially for charts containing dense numerical and structured information (e.g., financial report screenshots), where simple image-text similarity retrieval easily misses critical details;
- **Interpretable Presentation of Evidence**: In text-only RAG, citing source material is relatively straightforward (e.g., a text passage). In contrast, multimodal RAG must consider how to present to the user "which image or region within an image supports this conclusion," involving fine-grained grounding;
- **Modality Fusion During Generation**: When generating answers, the model needs to synthesize retrieved image and text evidence simultaneously for reasoning—such as combining a chart with explanatory text to answer jointly. This places higher demands on the model's cross-modal joint reasoning capability;
- **More Complex Chunking/Indexing Strategies**: Unlike text, images and videos cannot be directly chunked by paragraph for indexing. Dedicated chunking and indexing strategies must be designed for image regions, video keyframes, chart structures, etc. The engineering complexity of this pipeline is far higher than that of text document chunking pipelines.



![包含多向量数据库检索与跨模态上下文融合的多模态检索增强生成流程 (Workflow of Multimodal Retrieval-Augmented Generation involving multi-vector database search and cross-modal context fusion.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig59_multimodal_rag.png)

</details>

**15. What New Requirements Do 3D Scene Understanding and "Spatial Intelligence" Pose for Multimodal LLMs?**

<details>
<summary>💡 Show Answer</summary>

Traditional multimodal LLMs primarily process 2D images/videos, whereas "spatial intelligence" requires models to understand 3D spatial structures, geometric relationships between objects, affordance (how objects can be operated), and other information—capabilities that 2D visual understanding cannot fully cover.

**New Requirements:**
- **Modeling 3D Representations**: The model needs to process 3D data representations such as point clouds, depth maps, and 3D Gaussian Splatting, aligning them with language/2D visual information into a unified space;
- **Viewpoint-Invariant Understanding**: Observing the same 3D scene from different viewpoints yields completely different 2D images. The model must possess the ability to infer true 3D object relationships across viewpoints, rather than relying solely on single-viewpoint 2D appearances;
- **Affordance and Physical Interaction Reasoning**: Beyond identifying "what this object is," the model must understand "how this object can be grasped, pushed, or opened." Such reasoning directly serves manipulation planning in embodied AI;
- **Integration with Generative 3D Content**: As discussed earlier regarding spatial intelligence world models (such as Marble-like tools), generating editable, interactive 3D scenes from text/images can in turn serve as a source of training data for embodied AI, as well as a tool for content production in gaming/film.

</details>

