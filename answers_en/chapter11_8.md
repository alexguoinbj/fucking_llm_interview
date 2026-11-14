# 11.8 World Models — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. What Is a "World Model"? What Is Its Essential Difference from Traditional Generative Models?**

<details>
<summary>💡 Show Answer</summary>

A World Model refers to a model capable of learning and simulating the physical laws, causal relationships, and state evolution processes of the real world. Its core capability is: given the current state and an action, predicting how future states will evolve.

**Differences from Traditional Generative Models:**
- Traditional image/video generative models (such as early text-to-image models) optimize for generating content that is visually realistic and conforms to the training data distribution, without requiring internal explicit understanding of physical laws—as long as the output "looks right";
- The core demand of a World Model is **modeling the causality and consistency of state evolution**—the generated next state must not only be visually plausible, but also satisfy physical constraints (e.g., objects cannot vanish into thin air, motion trajectories must obey inertia), and must respond to the interactive input of an "action" (given different actions/interventions, producing distinct subsequent states conforming to causal logic);
- This is also the meaning behind the industry's paradigm shift from "Next-Token Prediction to Next-State Prediction": language models predict statistical continuations of symbol sequences, whereas World Models predict the **causal evolution of physical world states under the influence of actions**. There are fundamental differences in their modeling targets and evaluation standards.

</details>

**2. What Is the Core Concept of the JEPA Architecture? How Does It Differ from Autoregressive Generative Modeling?**

<details>
<summary>💡 Show Answer</summary>

JEPA (Joint Embedding Predictive Architecture) is the World Model path advocated by Yann LeCun, whose core assertion is that "prediction should occur in an abstract representation space, rather than in the raw pixel/signal space."

**Core Concepts:**
- Traditional autoregressive generative models (including video generation models that generate pixel-by-pixel or token-by-token) need to precisely reconstruct every single detail of the original input (every pixel, every token), forcing the model to waste significant capacity on predicting unpredictable, irrelevant detailed noise (such as the random swaying texture of leaves in a frame);
- The JEPA approach is: mapping inputs (such as past frames of a video) via an encoder to an abstract **latent space representation**. The predictor only needs to predict "what the representation at a future moment should be" in this latent space, without reconstructing the full raw pixel frame of the future moment;
- The training objective is to make the predicted future representation as close as possible to the encoded representation of the actual future input, while employing mechanisms (such as asymmetric encoder designs and regularization constraints) to prevent the model from learning a trivial yet meaningless solution of "encoding all inputs into the exact same constant vector" (the representation collapse problem).
**Differences from Autoregressive Generative Modeling:** Autoregressive models must "generate"—that is, explicitly produce complete, detail-level future content; JEPA only needs to "predict abstract representations." Theoretically, it is computationally more efficient and focuses more on semantic/physical information truly critical to the prediction task, without dwelling on unpredictable detailed noise. This is also LeCun's core argument for why this route is closer to "genuinely understanding the world" than pure generative language models.



![生成式自回归架构与联合嵌入预测架构 (JEPA) 的预测空间对比 (Comparison of prediction spaces between generative autoregressive architectures and Joint Embedding Predictive Architectures (JEPA).)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig107_jepa_vs_autoregressive.png)

</details>

**3. What Are the Main Technical Routes in Current World Model Research? What Are Their Respective Representatives and Focuses?**

<details>
<summary>💡 Show Answer</summary>

They can be roughly summarized into three technical routes developing in parallel with distinct focuses:

1. **Pixel Generation Faction**: Based on video generation models (such as the Sora series and Genie series), directly generating realistic, interactive sequences of future frames through training on massive video data. The advantage is that it can directly produce visual outcomes perceivable by humans and usable for downstream content production, as well as serving as a visual simulation environment for embodied AI; the limitation is the high computational cost of pixel-by-pixel generation, and visual realism does not equal physical correctness;
2. **JEPA Abstraction Faction**: Represented by the latent-space predictive architecture advocated by LeCun, asserting that neither language models nor pixel-by-pixel generation models are the path to true intelligence, and advocating learning the causal structure of the world within an abstract representation space. Theoretically, this better aligns with the goal of "efficient learning focused on essential laws," though its progress in large-scale application and concrete productization is currently at an earlier stage;
3. **Generative Spatial Intelligence Faction**: Represented by Fei-Fei Li's World Labs, advocating that World Models should be "generative, multimodal, interactive" 3D spatial models capable of generating explorable and editable 3D scenes from text or images. The focus is more heavily leaning toward spatial understanding and 3D content generation, closely related to the concept of "spatial intelligence" discussed earlier, with application scenarios leaning toward games, film/TV, and robotics simulation environment generation.
There are still debatable overlaps among these three routes in academic definitions (video generation models can also be argued to have implicitly learned a form of world model). The actual choice of route depends heavily on the relative weights given to dimensions such as "realism," "computational efficiency," "interactivity," and "3D consistency" in specific application scenarios.

</details>

**4. How Can World Models Help Address the Training Data Bottleneck in Embodied AI?**

<details>
<summary>💡 Show Answer</summary>

As discussed earlier, embodied AI faces bottlenecks of high acquisition costs and limited scale for real interactive data. World Models are considered an important technical means to alleviate this bottleneck.

- **Batch generating synthetic interaction data with action annotations**: A well-trained World Model can quickly "imagine" how subsequent states evolve given the current state and candidate actions, thereby batch-generating large-scale, diverse "state-action-consequence" training data at a cost far lower than real robot trial-and-error;
- **Making data without action annotations usable**: Massive human activity videos exist on the internet without explicit robot action annotations, making them difficult for traditional imitation learning to leverage directly. A World Model can learn the implicit state transition laws within videos, converting such data into valuable training signals for state evolution modeling and expanding the scope of usable data sources;
- **Low-cost simulation of high-risk/long-tail scenarios**: Dangerous, rare, and costly real-world scenarios (such as collisions and extreme weather driving scenarios) are difficult to acquire in large quantities through physical collection. World Models can batch-generate such long-tail scenario data in simulation to fill gaps in real data distributions;
- **Limitations**: All of this hinges on the premise that the World Model's own physical law modeling is sufficiently accurate. If the World Model's "imagination" deviates from actual physics, training downstream policies on its generated data may instead introduce new systematic biases. This is why current practices typically adopt a hybrid approach of "real data + World Model synthetic data" rather than relying entirely on synthetic data.

</details>

**5. What Are the Major Inference Cost Challenges Facing Current World Model Deployment?**

<details>
<summary>💡 Show Answer</summary>

Compared with traditional large text models, running a high-quality, real-time interactive World Model (especially real-time interactive models of the pixel generation faction) consumes one to two orders of magnitude higher compute during the inference stage.

**Main Causes:**
- **Continuous high-resolution, high-frame-rate generation**: To ensure visual realism and interactive smoothness, the model must continuously generate high-resolution frames at a high frame rate. Compared to language models that only need to generate discrete, low-dimensional token sequences, the single-step compute and output data volume are significantly larger;
- **Additional overhead for maintaining long-horizon consistency**: To ensure that generated content maintains scene, object, and physical law consistency over long interactions (without "memory loss" visual collapses), models typically need to maintain and process longer historical context, further increasing the computational burden;
- **Stringent latency requirements for real-time interaction**: A World Model with real-time interactive capabilities needs to rapidly generate response frames after a user executes an action. This imposes high demands on single-step inference latency, often requiring more hardware resources in parallel to meet latency metrics rather than amortizing costs via batch processing like non-real-time tasks.
**Industry Status**: According to some reports, the inference cost of running certain real-time interactive World Models can reach the scale of consuming an entire high-end GPU per user per hour—around a hundred times the inference cost of an equivalent-sized text model. This is one of the core reasons why World Models currently still face a significant gap before achieving large-scale consumer-grade real-time applications (such as real-time playable AI-generated games).

</details>

**6. Why Is Evaluating World Models More Difficult Than Evaluating Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

Many capabilities of language models (knowledge QA, code correctness, mathematical reasoning) possess relatively clear, automatically verifiable standard answers, whereas evaluating World Models faces more fundamental difficulties:

- **More ambiguous definition of "correctness"**: Language model answers usually have a unique or limited set of correct responses, but given the same initial state and action, a World Model may have multiple physically reasonable future evolutions (e.g., after a ball is lightly pushed, there is inherent randomness regarding the exact final rolling position). Defining "whether the generated result is correct" itself lacks a clear benchmark;
- **Difficulty in automatically verifying physical consistency**: Judging whether a generated video violates physical laws (e.g., clipping, objects vanishing into thin air, trajectories failing to obey inertia) currently relies heavily on human subjective assessment. It lacks fully automated, unambiguous verification tools like code execution results or math answer verification. Although specialized physical understanding benchmarks (such as PhysBench mentioned earlier) are attempting to quantify such capabilities, their coverage and reliability remain at an early stage;
- **High evaluation costs for long-horizon consistency**: Assessing whether a model maintains scene and object state coherence across long interactive sequences often requires manual frame-by-frame and segment-by-segment inspection, making it difficult to execute in large-scale automated parallel fashion like language model evaluation;
- **Lack of consensus on unified benchmarks**: Because the three technical routes mentioned earlier disagree on the very definition and goals of a "World Model," academia and industry have not yet established a widely recognized standardized evaluation framework similar to MMLU or HumanEval for language models. Different teams often use custom evaluation methods, making cross-comparison difficult.

</details>

**7. How Are World Models Combined with Autonomous Driving Simulation? What Significance Does This Hold for Autonomous Driving R&D?**

<details>
<summary>💡 Show Answer</summary>

Autonomous driving R&D has long relied on two data sources: real road test data (costly and difficult to cover long-tail scenarios) and traditional rule/physics engine simulation data (limited scenario diversity and realism). World Models provide a third path for this purpose.

- **Generating high-fidelity, interactive driving scenarios**: World Models trained on real road test data can generate simulation scenarios that are visually highly realistic and conform to the behavioral patterns of real traffic participants. These scenarios are used to train and test autonomous driving policies, adhering much closer to real-world distributions than scenarios rendered by traditional simulation engines;
- **Batch generating dangerous and long-tail scenarios**: Real-world data for actual traffic accidents or extreme weather driving scenarios is extremely scarce and bears exceedingly high acquisition costs (including safety risks). World Models can "imagine" and batch-generate such scenarios in simulation, augmenting training and evaluation coverage for autonomous driving systems on long-tail scenarios;
- **Closed-loop simulation testing**: Allowing autonomous driving policies to undergo closed-loop testing in simulation environments generated by World Models (where policy actions influence the subsequent evolution of the simulation environment in real time, rather than merely replaying fixed historical data) enables more realistic evaluation of policy performance in dynamic interaction scenarios—something difficult to achieve with traditional "open-loop replay" evaluation methods;
- **Significance**: This combination is believed to promise significant reductions in the real-world road test mileage required for autonomous driving systems to achieve an acceptable safety level, representing one of the few application directions for World Models that has formed a clear commercialization path.

</details>

**8. Are World Models an Essential Path to AGI? What Are the Viewpoints of Different Camps?**

<details>
<summary>💡 Show Answer</summary>

This is one of the most controversial issues in the field of AI today, with representative figures and institutions holding distinct positions:

- **"Essential Path" Camp**: Represented by Yann LeCun, the core argument is that pure language models can only learn knowledge expressed in human language that is highly compressed and abstract. In contrast, most capabilities of human and animal intelligence (physical intuition, common-sense reasoning, planning capability) stem from direct perception and interaction experience with the physical world. Such experiences are difficult to fully convey through text corpora. Therefore, proponents argue that pure language model routes lacking physical world modeling capabilities face fundamental limitations, even using terms like "dead end" to describe the pure LLM route;
- **"Important Complement but Not the Only Path" Camp**: Many researchers acknowledge the value of world models for specific capabilities such as embodied intelligence and physical reasoning, but do not view them as the sole or mandatory path to general intelligence. They argue that Large Language Models (LLMs), through continuous scaling, reinforcement learning, multimodal fusion, and other means, may also gradually acquire sufficiently strong physical common sense and reasoning capabilities without an explicit physical world model (partial evidence comes from the fact that LLM performance on physics-related Q&A tasks indeed improves with scale and multimodal training, although benchmarks like the aforementioned PhysBench also show that this improvement is unstable and exhibits noticeable shortcomings);
- **Pragmatic Middle Ground**: An increasing number of voices in industry tend to view this more as an engineering problem of "what modeling paradigms are required for different capabilities," rather than an either-or paradigm conflict—language/symbolic reasoning capabilities rely more on large-scale text corpus training, while physical intuition and spatial manipulation capabilities rely more on world-model-style training. Future general intelligence systems are likely to be hybrids integrating multiple modeling paradigms, rather than the victory of a single route.

</details>

**9. What Are the Key Technical Points of "Real-Time Interactive World Models" Represented by Genie 3?**

<details>
<summary>💡 Show Answer</summary>

Real-time interactive world models represented by Genie 3, compared to early video generation models, achieve their core breakthrough in **supporting users in intervening in the generation process in real time using action commands while receiving low-latency frame responses**, upgrading the paradigm from "generating a video segment offline" to an "explorable virtual environment in real time."

**Key Technical Points:**
- **Frame-by-Frame Autoregressive Generation with Action Conditioning**: When generating each frame, the model must simultaneously accept the action currently input by the user (such as movement direction) as a condition to adjust the generation direction of subsequent frames in real time, rather than generating an entire fixed content segment in advance;
- **Long-Horizon Scene Consistency Maintenance**: Because interaction is continuous rather than fixed length, the model needs the capability to "remember" previously generated scene layouts and object positions (e.g., if a user turns away and turns back, the scene should remain consistent rather than being regenerated into a completely different appearance), which imposes high requirements on the model's memory mechanism and long-context modeling capability;
- **Low-Latency Inference Architecture**: To ensure a smooth real-time interaction experience, the model architecture and inference system must be specifically optimized for low latency (such as the aforementioned efficient decoding and hardware-level parallel optimizations), which is also the direct cause of high inference costs for such models;
- **Significance**: Such real-time interactive world models are considered the most direct technical embodiment of the application direction "world models as explorable simulation environments," offering direct value to fields such as embodied AI training environment generation and game content generation.



![交互式世界模型中动作条件驱动的动力学与帧预测循环 (Action-conditioned dynamics and frame prediction loop in interactive world models.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig114_interactive_world_model.png)

</details>

**10. How Is the View That "Breakthroughs in Training Data Paradigms Are More Important Than Model Architecture Innovations" Demonstrated in the Field of World Models?**

<details>
<summary>💡 Show Answer</summary>

The core assertion of this view is that when facing tasks requiring an understanding of complex physical causal relationships like world models, the gains brought purely by improving network architectures (e.g., adopting better attention mechanisms or deeper networks) may be less significant than those brought by **altering how training data is organized and utilized**.

**Specific Manifestations in the Field of World Models:**
- **Shift from Passive Observation Data to Active Interactive Data**: Early video generation models were primarily trained on passively recorded videos (human activity logs, film/TV materials), which lack explicit "action-consequence" causal annotations. Recently, there has been a stronger emphasis on collecting or generating interactive data with explicit action conditions and state transition annotations (such as the aforementioned teleoperation data, Ego-View data, and synthetic simulation data). This shift in data paradigm is believed to enhance the model's ability to model causal relationships more effectively than merely scaling up passive video data;
- **Joint Utilization of Cross-Modal and Cross-Source Data**: Data from a single source (pure videos or pure robot trajectories) has inherent coverage limitations. Jointly utilizing data from different sources and modalities via unified representations and training objectives (rather than training separately and concatenating) brings more significant capability improvements than architectural tweaks;
- **Analogy to Language Model History**: This perspective echoes the widely acknowledged empirical rule from the history of language models that "improvements in pre-training data scale and quality often contribute more than contemporaneous architectural modifications." The field of world models is currently considered to be in a similar early stage where data paradigms still offer huge room for improvement.

</details>

**11. Does a Language Model Implicitly Contain a "World Model"? How Does This Perspective Differ from the World Models Discussed Earlier in This Chapter?**

<details>
<summary>💡 Show Answer</summary>

The question of "whether a world model exists inside LLMs" was discussed in the section "Model Capability Boundaries and Interpretability." That perspective leaned more toward mechanistic interpretability—i.e., verifying through probe analysis and other methods whether the internal activations of a language model encode implicit state representations (such as the current state of a chess game). In contrast, the world models discussed in this section represent an independent system of models **explicitly designed with state prediction as their direct training objective**. The two perspectives and evaluation standards differ noticeably:

- **Implicit vs. Explicit**: The "potential" world model within a language model is a byproduct of training—to better accomplish the goal of "next-token prediction," the model may incidentally acquire internal state representations that aid prediction. Such representations are implicit, unsupervised, and have uncertain capability boundaries. In contrast, the world models discussed in this section are specifically designed and trained with state prediction itself as an explicit and direct training objective;
- **Differences in Capability Reliability**: Evaluations such as the aforementioned PhysBench show that the physical understanding implicitly embedded in language/vision-language models does not scale stably with model size, indicating clear limits to the capability boundary and reliability of such "implicit world models." On the other hand, explicitly trained world models (especially those trained on domain-specific data) typically outperform implicitly emergent capabilities in terms of physical consistency and state prediction accuracy within their target scenarios;
- **Complementary Relationship Between the Two Perspectives**: Mechanistic interpretability research focuses on the scientific question of "whether it can be proven/localized that a state representation indeed exists inside the model," whereas the world models discussed in this section are more concerned with the engineering question of "how to design a practical system dedicated to state prediction and simulation." Although both share the term "world model," their research goals and evaluation methods are not identical, requiring careful distinction of what "world model" refers to in specific contexts when reading relevant literature.

</details>

**12. How Might the Three Major Technical Routes of World Models Differentiate in Future Application Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Considering the respective technical characteristics and cost structures of the three routes, a reasonable expectation is that they will form relatively differentiated divisions of labor across different application scenarios rather than completely replacing one another:

- **The Pixel Generation School** is better suited for scenarios with high demands on **visual presentation quality** that can tolerate higher inference costs, such as film/game content generation, simulation demonstrations requiring direct visual presentation to human users, and short-duration, high-fidelity embodied AI training clip generation;
- **The JEPA Abstraction School** is better suited for scenarios with high requirements for **computational efficiency** where visual content directly viewable by humans is unnecessary, and prediction results are instead used for downstream decision-making (such as robot planning modules and autonomous driving behavior prediction modules), because these scenarios truly require accurate and efficient state prediction rather than realistic pixel images;
- **The Generative Spatial Intelligence School** is better suited for scenarios requiring the construction of **persistent, editable, and reusable 3D environments**, such as virtual world/metaverse content creation, 3D prototyping for architectural and product design, and robot simulation platforms that require repeated training and testing within the same environment.
**Overall Judgment:** In the short term, the three routes are more likely to develop in parallel and prove their respective value in different niche scenarios, rather than rapidly converging into a single "winning" route. Whether they will move toward some form of technical convergence in the long term (such as hybrid architectures combining pixel generation with latent space prediction) remains an open question.

</details>

**13. What Role Do World Foundation Models (e.g., NVIDIA Cosmos) Play in Robot Training?**

<details>
<summary>💡 Show Answer</summary>

A World Foundation Model refers to a foundation model that undergoes large-scale pre-training and possesses general physical world modeling capabilities. Like language foundation models, it can be fine-tuned or directly invoked by various downstream robotic tasks, rather than training a dedicated simulation model from scratch for every task.

**Role in Robot Training:**
- **As Infrastructure for General-Purpose Simulation Environments**: Providing a unified simulation foundation with strong understanding of physical laws across different robot platforms and task scenarios. Downstream teams can rapidly build simulation training environments targeted at specific tasks on top of it, without needing to construct physical simulation engines and scene assets from scratch;
- **As a "Data Factory" Generating Training Data**: Combining the aforementioned idea of "synthetic data alleviating real-world data bottlenecks," World Foundation Models can generate customized synthetic training data in batches (including corresponding action annotations) according to specific robot task requirements, serving as one of the data sources for training embodied AI policies;
- **Strategic Positioning for Hardware Vendors**: Represented by NVIDIA Cosmos, hardware vendors launching World Foundation Model platforms to some extent extends their positioning from "providing compute hardware" to "providing the software infrastructure required for physical AI training." By integrating hardware and software, they enhance their ecosystem position in the emerging market of robotics and embodied AI. This also reflects that world models are accelerating their evolution from pure research topics into infrastructure products with clear commercialization pathways.

</details>

**14. What Is the Relationship Between World Models and Vision-Language-Action (VLA) Models? How Are Tasks Divided Between the Two in Embodied AI Systems?**

<details>
<summary>💡 Show Answer</summary>

VLA models were discussed earlier in the section "Multimodal and Embodied AI Applications"—they are models that end-to-end predict specific action outputs directly from visual observations and language instructions, essentially serving as a **policy**. In contrast, world models do not directly output actions; instead, they predict "how the world will evolve given a state and an action," essentially serving as an **environment simulator**.

**Division of Labor and Integration Modes in Embodied AI Systems:**
- **World Models as Training Environments for VLA**: As previously discussed, training VLA policies requires massive amounts of state-action-consequence data. World models can serve as low-cost simulation environments where VLA policies undergo large-scale trial-and-error training, bridging the shortage of real robot training data, followed by Sim-to-Real fine-tuning with a small amount of real-world data;
- **World Models Assisting VLA in Planning**: A more advanced integration mode is allowing the VLA policy to "rehearse" potential consequences of multiple candidate actions using the world model before actually executing an action, selecting the action with the best expected outcome for execution (similar to planning concepts in model-based reinforcement learning). Theoretically, this "imagine first, act later" paradigm can handle complex tasks requiring multi-step planning far better than relying solely on single-step mapping by policy networks;
- **Potential Trend of Capability Fusion**: Currently, VLAs and world models are mostly trained separately and combined during usage as two independent models. However, research directions are also exploring the integration of both capabilities into a unified model—capable of directly outputting actions like a VLA while retaining state prediction capabilities similar to a world model. This follows a logic similar to the trajectory in language models of "moving from multiple specialized models toward unified foundation models."



![具身智能中世界模型模拟与 VLA 策略执行之间的功能划分 (Functional division between World Model simulation and VLA policy execution in embodied AI.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig119_world_model_vla.png)

</details>

**15. What Is the Relationship Between World Models and Model-Based Reinforcement Learning (Model-Based RL) in Reinforcement Learning?**

<details>
<summary>💡 Show Answer</summary>

Model-Based Reinforcement Learning (MBRL) is a long-standing branch within reinforcement learning. Its core idea is to have an agent first learn a dynamics model of the environment (i.e., a model predicting "given state $s$ and executing action $a$, what state $s'$ it will transition to and what reward it will receive"), and then leverage this learned model for planning or generating synthetic experiences to accelerate policy learning. This differs from "Model-Free Reinforcement Learning" (Model-Free RL), which learns policies directly from real interactions without constructing an environment model.

**Relationship Between the Two:** Methodologically, the currently popular concept of "world models" can be seen as the modern extension and scaling of the "environment dynamics model" component from MBRL in the deep learning era, tailored for high-dimensional visual observations and large-scale pre-training scenarios:
- **Shared Value Proposition**: Both aim to reduce downstream policy learning's dependence on real environment interactions by "first learning to predict how the environment changes." The world model's approach of generating synthetic training data and assisting planning shares the exact same heritage as classic MBRL's approach of "using learned models to generate virtual rollouts to reduce real environment sampling";
- **Differences in Scale and Paradigm**: Environment models researched in classic MBRL are typically small in scale and trained for a single task/environment. In contrast, modern world models pursue large-scale pre-training on massive, diverse real-world data (videos, robot trajectories), aiming to acquire general environment understanding capabilities across tasks and scenarios. This is closer to the projection of the "foundation model" paradigm onto MBRL;
- **Insights**: Understanding long-standing core challenges in classic MBRL—such as model error accumulating and compounding over long horizons (the "model bias" problem) and how policy learning appropriately accounts for model uncertainty—provides direct reference value for understanding current difficulties faced by world models in long-horizon consistency and reliability. It demonstrates that many challenges facing world models are not entirely new issues, but rather classic reinforcement learning problems re-emerging under new model and data scales.

</details>

