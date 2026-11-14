# 11.12 AI for Science and Artificial General Intelligence for Science (AGI4S) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 11: Cutting-Edge Topics and Comprehensive Design · 15 questions

**1. What Is the Paradigm Upgrade from AI4S to AGI4S? What Are the Core Differences Between the Two?**

<details>
<summary>💡 Show Answer</summary>

AI4S (AI for Science) refers to using artificial intelligence techniques to **assist** one or several specific stages in the scientific research workflow (such as accelerating a specific computational task or assisting literature retrieval). AI acts primarily as a "tool," while the overall workflow design, hypothesis generation, and result interpretation remain predominantly led by human scientists.

**Core Differences of AGI4S (AGI-Driven Scientific Discovery):**
- **From "Tool-Based Empowerment" to "Driven Paradigm Transformation"**: AGI4S envisions AI no longer merely assisting individual stages, but running through the entire scientific discovery lifecycle—autonomously generating new scientific hypotheses, invoking experimental tools and applications to control real or simulated experiments, and collaborating with humans and other AIs to interpret results, acting as a "research colleague" rather than a "tool";
- **Requirement for Generalist-Specialist Integration**: AGI4S demands that models possess both interdisciplinary general reasoning abilities and deep domain specialization in specific scientific fields. This is an extension of the previously discussed debate between "general-purpose LLMs vs. domain-specialized models" within the specific scenario of scientific discovery;
- **Infrastructure Upgrades**: Supporting AGI4S requires the synergy of two infrastructure pillars: "scientific multimodal large models" (capable of understanding diverse scientific data modalities like molecular structures, experimental images, and scientific literature) and "scientific discovery platforms" (capable of actually scheduling and executing scientific experimental workflows), rather than merely connecting a general conversational model to scientific literature Q&A.



![从单一领域科学 AI 模型 (AI4S) 到通用科学智能体 (AGI4S) 的范式转变 (Paradigm shift from narrow scientific AI models (AI4S) to general-purpose scientific agents (AGI4S).)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig171_ai4s_to_agi4s.png)

</details>

**2. What Does the "Cannikin Law" (Bucket Effect) Exhibited by Current Frontier Models on Scientific Tasks Specifically Refer To?**

<details>
<summary>💡 Show Answer</summary>

Studies collaborating with scientists across multiple scientific domains to design evaluation systems revealed that frontier models achieve a moderate score of around 50 points (out of 100) on **general scientific reasoning** tasks, but their scores plummet to 15--30 points on specific tasks requiring **specialized depth** (such as domain-specific literature retrieval or detailed experimental protocol design).

**Meaning of the "Bucket Effect":** Scientific discovery is a full chain composed of multiple stages (hypothesis generation $\rightarrow$ experiment design $\rightarrow$ execution/simulation $\rightarrow$ result analysis $\rightarrow$ hypothesis iteration). Overall efficacy is constrained by the **weakest link** in the chain. Even if a model performs acceptably in general scientific reasoning, as long as it falls short in a specific practical domain step (such as designing an actually executable experimental protocol satisfying real laboratory constraints), it will bottleneck the actual output of the entire scientific discovery process—consistent with the principle that "the shortest slat determines the capacity of the bucket."

**Implications for R&D Directions**: Purely increasing a model's general scientific knowledge level (akin to enhancing "breadth") yields diminishing returns. What truly determines whether AGI4S can be successfully deployed is whether the "short-board" capabilities in specialized domain steps can be shored up. This is why "integrating general reasoning with specialized capabilities" is considered an imperative choice in moving from AI4S to AGI4S, rather than a problem that can be automatically solved simply by scaling up model size.

</details>

**3. Which Categories of Data Are Required to Support an AI-Ready Scientific Data System? How Do They Convert into and Promote One Another?**

<details>
<summary>💡 Show Answer</summary>

**Four Core Data Categories:**
- **Literature Data**: Scientific literature itself carries humanity's accumulated scientific knowledge and serves as the most fundamental data source;
- **Experimental Data**: Raw observational data produced by real experiments, serving as first-hand authentic signals for verifying hypotheses and training models;
- **Simulation Data**: Data generated through computational simulations (such as molecular dynamics simulations and physical simulations), which can supplement data at scale and low cost across parameter spaces difficult to cover via real experiments;
- **Model Data**: Data generated or contained by trained scientific models themselves, which condenses scientific knowledge patterns extracted by models from massive raw data.
**Mechanisms of Mutual Conversion and Promotion:** Parsing content from literature data extracts and accumulates experimental data suitable for model training. Model data itself contains extensive scientific knowledge, which can in turn be used to generate richer simulation data. Through deeper cross-linking and integrated fusion, these data categories form a structured, interpretable, and inferable global knowledge base to support scientific discovery. This cycle is aligned with the specific manifestations of concepts discussed earlier—such as "synthetic data training" and "world model training data generation"—in the scientific realm: feeding the model's own outputs back into the training data system to form a data flywheel. However, one must remain vigilant regarding the risk of error accumulation caused by over-reliance on self-generated model data.

</details>

**4. How Are Agentic and Embodied AI Technologies Integrated into the Scientific Research Workflow?**

<details>
<summary>💡 Show Answer</summary>

Recent observations show that the core capabilities of agentic and embodied AI technologies in planning, decision-making, task scheduling, and multi-agent collaboration are accelerating in maturity and rapidly integrating into scientific research workflows. This is highly related to the concrete applications of Agent Harness and multi-agent collaboration architectures discussed earlier in scientific research contexts:

- **Automated Orchestration of Research Tasks**: Decomposing a complex research project into multiple subtasks—such as literature review, hypothesis generation, experiment design, and data analysis—with a primary Agent responsible for overall planning and scheduling, while multiple sub-Agents specializing in different stages complete specific tasks in parallel or sequentially. This represents a direct transfer of the multi-agent collaboration model discussed in the "Agent Product Engineering Architecture" section to scientific research scenarios;
- **Embodied AI Undertaking Physical Experimental Operations**: For experimental stages requiring hands-on manipulation (such as chemical synthesis and material preparation), embodied AI technology enables robots to autonomously execute certain experimental steps in laboratory environments. This relies on advancements in robotics foundation models, VLA (Vision-Language-Action), and other embodied AI technologies discussed earlier;
- **Shift in Human-AI Collaboration Patterns**: The role of researchers is gradually shifting from "personally executing every single step" to "designing research directions, auditing hypotheses and protocols proposed by Agents, and overseeing critical decision nodes." This mirrors the division of labor discussed earlier in Agentic Coding scenarios, where "humans audit and oversee, while Agents undertake specific execution."

</details>

**5. What Role Do Open-Source Projects Play in the Field of AI for Science? What Hierarchical Structure Does Their Development Exhibit?**

<details>
<summary>💡 Show Answer</summary>

Open-source projects are regarded as the core vehicle for knowledge sharing and collaborative innovation in AI for Science. Their development currently exhibits a clear layered structure from the infrastructure level up to the application layer:

- **Framework Layer**: Provides underlying software frameworks and foundational tool libraries required for scientific computing and scientific AI model training, serving as the bedrock of the entire tech stack;
- **Toolkit Layer**: Builds domain-specific toolkits atop the framework layer targeting particular categories of scientific tasks (such as molecular property prediction and protein structure analysis), encapsulating common workflows and standard components to lower development barriers for researchers;
- **Model Layer**: Pre-trained scientific models directly addressing specific scientific tasks—usable out of the box or with minimal fine-tuning (such as the scientific multimodal large models mentioned earlier)—representing the most direct deliverables encountered by researchers.
**Value:** This layered open-source ecosystem allows research teams of varying capability levels and resource scales to participate and contribute at appropriate levels. Teams with strong infrastructure capabilities can focus on framework-level innovation, whereas application-oriented teams can directly reuse achievements at the framework and toolkit layers to concentrate on solving specific scientific problems at the model layer. This division of labor and collaborative model accelerates the diffusion speed of innovation across the field.

</details>

**6. Through What Concrete Mechanisms Does AI Facilitate Interdisciplinary Knowledge Flow and Fusion Transfer?**

<details>
<summary>💡 Show Answer</summary>

Interdisciplinary domains—such as the intersections of chemistry, physics, and materials science—often generate numerous critical new scientific questions. Traditionally, however, interdisciplinary collaboration has been constrained by knowledge barriers and communication costs between scientists of different fields.

**Mechanisms by Which AI Promotes Interdisciplinary Integration:**
- **Unified Knowledge Representations Lower Interdisciplinary Comprehension Barriers**: Jointly trained on multidisciplinary literature and data, general scientific large models can act to a certain extent as a "translation layer" between different disciplinary knowledge systems. This helps scientists in one domain quickly comprehend and draw inspiration from concepts and methods in another, lowering comprehension barriers that traditionally required prolonged specialized training;
- **Transfer Learning-Style Methodological Borrowing**: Modeling methodologies proven effective in one discipline (such as neural network architectural designs incorporating specific physical constraints) can be rapidly transferred via AI models to structurally similar problems in other disciplines. Such "methodological transfer," which traditionally depended on individual scientists' cross-domain vision and accumulated experience, can theoretically be discovered and executed by AI systems in a far more systematic and large-scale manner;
- **Computation and Data as a Lingua Franca**: Regardless of specific disciplinary backgrounds, an increasing number of scientific problems can ultimately be formulated into computationally tractable, data-driven representations. This trend of "computation as a lingua franca" objectively fosters convergence and mutual inspiration among research paradigms across disciplines, with AI technology serving as the primary driver behind this trend.

</details>

**7. Compared with General Multimodal Large Models, What Special Architectural Considerations Are Involved in Designing Scientific Multimodal Large Models (such as Intern-S1)?**

<details>
<summary>💡 Show Answer</summary>

Scientific multimodal large models must process data modalities far more complex and specialized than general multimodal large models (which primarily process natural images and text). Their design involves several special considerations:

- **Support for Science-Specific Data Modalities**: The model must be able to understand highly specialized data modalities—such as chemical structural formulas, crystal structures, protein folding conformations, microscopic/telescopic observational images, spectral data, and experimental instrument reading curves. The statistical properties and semantic structures of these data modalities differ fundamentally from natural images and everyday text. Directly reusing visual encoders from general multimodal models yields limited effectiveness, necessitating the targeted design or training of specialized encoders;
- **Need for Explicit Modeling of Physical/Chemical Constraints**: Generated or predicted outputs in scientific contexts (such as molecular structures) must often satisfy strict physical and chemical laws (such as chemical bond plausibility). Techniques discussed earlier, such as neuro-symbolic integration and constrained decoding, hold direct application value in scientific contexts—purely data-driven generation is prone to producing "hallucinated" results that violate basic physical and chemical principles;
- **Prioritizing Depth of Domain Expertise Over Breadth of Coverage**: Unlike general multimodal models that pursue covering as broad a range of daily scenarios as possible, scientific multimodal models often require deeper specialized training within specific scientific disciplines. This echoes the aforementioned "generalist-specialist integration" design philosophy—retaining cross-disciplinary general understanding while achieving depth approaching professional scientists in specific subjects.

</details>

**8. What Capabilities Do AI-Driven Scientific Discovery Platforms (such as Intern-Discovery) Typically Require in Their Architecture?**

<details>
<summary>💡 Show Answer</summary>

Distinct from standalone scientific models (which handle "thinking"), scientific discovery platforms represent the concrete realization of the Agent product engineering architecture discussed earlier in scientific research contexts, requiring complete capabilities to "execute" research workflows:

- **Interface Integration with Experimental Tools and Instruments**: Connecting via standardized protocols (similar to the MCP discussed earlier) to actual laboratory instruments, simulation software, database retrieval systems, and other external tools to convert model-generated experimental protocols into executable operational instructions;
- **Closed-Loop Hypothesis-Verification Scheduling**: The platform must support the automated closed-loop scheduling of "hypothesis generation $\rightarrow$ verification protocol design $\rightarrow$ execution (experiment or simulation) $\rightarrow$ result analysis $\rightarrow$ hypothesis refinement." This applies the Agent Loop execution model to scientific research, requiring specialized rigor-guarantee mechanisms during verification to prevent outputting an unverified hypothesis as a "confirmed" conclusion;
- **Inter-Institutional and Cross-Team Collaborative Support**: Real scientific research often involves collaboration across multiple institutions and disciplines. The discovery platform must support human researchers stepping in at key nodes to review and adjust AI-proposed research directions, while maintaining complete logs of the research process to enable credibility reviews by the scientific community (extending the observability and auditability requirements discussed earlier to scientific integrity scenarios);
- **Safety and Ethical Review Mechanisms**: Scientific discovery platforms involved in fields like biology and chemistry require built-in safety review mechanisms to prevent being used to generate potentially hazardous research outcomes (such as molecular design schemes that could be misused to manufacture dangerous substances). This aligns with the requirement that model safety alignment mechanisms be specially reinforced for scientific discovery contexts.



![AI 驱动的科学发现平台集成系统架构 (Integrated system architecture of AI-driven scientific discovery platforms.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig178_scientific_discovery_platform.png)

</details>

**9. What Capability Characteristics Are Demonstrated When Applying AI to Specialized Fields Such as Drug Molecule Design and Quantum Circuit Optimization?**

<details>
<summary>💡 Show Answer</summary>

In specialized domain tasks requiring "multi-step viable solutions"—such as drug molecule design and quantum circuit optimization—frontier models are reported to demonstrate coherent multi-step reasoning and planning capabilities approaching those of human experts, going beyond simply reproducing patterns present in training data.

**Demonstrated Capability Characteristics:**
- **Combinatorial Optimization Capability**: Drug molecule design is essentially a combinatorial optimization problem under multiple constraints, such as pharmacological activity, safety, and synthetic accessibility. Quantum circuit optimization similarly requires balancing circuit depth, fidelity, hardware constraints, and other dimensions. Models need the capability to solve multi-constraint combinatorial optimization problems rather than relying on simple pattern matching or retrieval;
- **Integration of Domain Expertise and General Reasoning**: Such tasks demand that models possess adequate domain-specific expertise (such as fundamental principles of medicinal chemistry or foundational quantum computing theory) alongside the reasoning transfer capability to apply this knowledge to novel problems. This is precisely the domain-depth capability highlighted as a common bottleneck in the prior "bucket effect" discussion;
- **Limitations to Guard Against**: Although reports suggest breakthroughs by models in these domains, as repeatedly emphasized regarding evaluation reliability, actual domain outputs (such as designed molecular structures) must ultimately be validated for efficacy and safety through real wet-lab experiments or rigorous simulations. A gap remains between a model's "plausible-looking multi-step scheme" and a "rigorously verified, practical scheme," necessitating oversight from human experts and experimental validation rather than blindly trusting seemingly coherent reasoning outputs.

</details>

**10. How Can AI-Native Research Workflows Be Designed in the AI for Science Field to Serve Research and Education Needs in Resource-Constrained Regions?**

<details>
<summary>💡 Show Answer</summary>

Beyond frontier scientific discovery capabilities targeted at top-tier research institutions, AI for Science is also recognized as carrying the mission of bridging research and education resource divides, requiring workflows specially designed for resource-constrained scenarios.

- **Reducing Reliance on Specialized Equipment and Data**: Applications serving resource-constrained regions (such as rural education in India, agriculture in Kenya, and primary healthcare in Brazil) must minimize reliance on expensive specialized equipment and massive localized data during design. Instead, they should rely more on general knowledge embedded within models and lightweight local adaptations, providing effective research and production assistance via mobile phones or low-cost terminals;
- **Adaptation to Localized Contexts**: The value of such applications hinges significantly on their ability to understand and adapt to specific local contexts (local languages, local crop varieties, local common diseases, and healthcare resource constraints). This requires dedicated local data collection and fine-tuning, rather than directly applying general-purpose models trained for developed-region scenarios;
- **"AI-Native" Rather Than "AI-Patch" Workflow Design Philosophy**: Rather than rigidly plugging AI capabilities into existing workflows that may already be ill-suited to local resource conditions, a more effective design approach is to re-architect brand-new workflows centered on AI capabilities and adapted to local realities (such as unstable network connectivity or scarcity of professional staff). This design philosophy aligns with the earlier concept of "re-architecting engineering systems around model capabilities rather than naive concatenation," extending the application context from enterprise software engineering to broader social impact domains.

</details>

**11. Will AI's Involvement in Scientific Discovery Diminish the Core Role of Human Scientists in Asking "Profound Questions"? How Should We View This Concern?**

<details>
<summary>💡 Show Answer</summary>

Some argue that every major scientific revolution that transformed human life began with humans asking profound questions (such as inquiries into the fundamental composition of the universe or the origin of life). This raises a question worth exploring: Will AI's deep involvement in the scientific discovery workflow diminish this core human capability and role of asking "profound questions"?

**Perspectives from Different Angles:**
- **The view supporting "AI potentially assuming part of the questioning role"**: As models' capabilities to autonomously generate hypotheses improve (such as "AI generating new hypotheses" in the aforementioned AGI4S vision), AI is indeed starting to take on the "asking questions" stage—traditionally led entirely by human scientists—to some extent. Particularly at the level of data-driven pattern discovery (identifying anomalous patterns or underlying associations from massive datasets that are hard for humans to detect), AI may be better suited than humans at discovering new problems worth pursuing within existing knowledge frameworks;
- **The view emphasizing human irreplaceability**: Truly "profound" scientific questions often stem from questioning and breaking through existing paradigms themselves (such as the theory of relativity challenging the Newtonian mechanics paradigm). This fundamental level of paradigm questioning requires a deep understanding of the history of science, philosophy, and human values and meaning—capabilities considered difficult for current AI systems (which essentially still perform pattern recognition and reasoning extensions based on existing knowledge and data distributions) to truly possess. Thus, the higher-level role of humans in setting "directions worth pursuing" is difficult to replace in the short term;
- **A more realistic vision of division of labor**: A pragmatic expectation is to form a collaborative model where "humans are responsible for posing directional, profound questions, while AI is responsible for efficiently generating specific, verifiable sub-hypotheses and executing verification within the given direction," rather than simply discussing "who replaces whom." This aligns with the division of labor philosophy repeatedly emphasized earlier—"humans serve as the gatekeeping mechanism, while AI handles specific execution"—as manifested at the macro level of scientific discovery.

</details>

**12. What Does the Concept of "System Intelligence" Refer to in the Context of Scientific AI Infrastructure?**

<details>
<summary>💡 Show Answer</summary>

Some argue that the future of computing lies not only in faster speeds, but even more in intelligence; "System Intelligence" enables AI to evolve from merely generating code to designing, optimizing, and managing the entire computing/scientific research system itself.

**Implications in the context of scientific AI infrastructure:**
- **From "using tools" to "designing and optimizing tools themselves"**: Early AI-assisted scientific research primarily involved using established computational tools and workflows to complete specific tasks. The vision of system intelligence is to enable AI to further participate in the design and optimization of the underlying computing systems relied upon by scientific research (such as automatically optimizing parallelization strategies for scientific computing tasks or auto-tuning parameter configurations of simulation software to improve efficiency). This represents an analogous extension of the previously discussed "Meta-Harness" (AI optimizing the Agent execution environment itself) in scientific computing infrastructure scenarios;
- **Intelligent scheduling of scientific research infrastructure resources**: Similar to the idea in inference economics discussed earlier regarding "peak-valley pricing guiding staggered compute usage," system intelligence can also be manifested as the intelligent scheduling and optimized allocation of scientific computing resources (such as supercomputing clusters and experimental equipment scheduling), enabling limited scientific infrastructure resources to achieve greater overall efficacy;
- **Significance**: This concept reflects the industry's further expansion of AI's role positioning—moving beyond being a mere "participant" completing specific research tasks to potentially becoming a "system engineer" responsible for designing and maintaining the efficacy of the entire scientific research infrastructure system. This is a natural extension of the positioning of "AI as a full-process scientific collaborator" within the AGI4S vision.



![科学计算中构成系统智能的分层基础设施 (Hierarchical infrastructure layers composing system intelligence in scientific computing.)](https://img.pricenode.uk/llm_interview_questions/english/chapter11/fig182_system_intelligence.png)

</details>

**13. Compared to General Capability Evaluation, What Special Design Requirements Does the Evaluation System in the AI for Science Domain Have?**

<details>
<summary>💡 Show Answer</summary>

Combining the reliability issues of general capability evaluation discussed earlier (benchmark contamination, overfitting, etc.), evaluation in the domain of AI for Science also faces additional challenges that are more domain-specific:

- **Requirement for real scientists to participate in evaluation design and result judgment**: The "correctness" of scientific tasks often lacks a uniquely determined standard answer unlike conventional benchmarks (especially for tasks involving experimental protocol design and hypothesis plausibility assessment). It requires inviting domain-expert scientists to participate in designing evaluation prompts and grading results. This is also the reason behind the aforementioned approach of "collaborating with 100 scientists across 10 different scientific fields to design an evaluation system"; however, the scalability and standardization level of this evaluation approach are naturally lower than pure automated evaluation;
- **Evaluation needs to cover the full workflow of scientific discovery rather than isolated stages**: Due to the aforementioned "barrel effect," evaluating performance in only an isolated stage (such as pure knowledge Q&A) fails to reflect the model's actual effectiveness across the full, real-world scientific discovery process. The evaluation system design needs to cover multiple stages, including hypothesis generation, experimental design, and result interpretation, while paying special attention to whether transitions between stages are smooth;
- **High cost and long cycle of real-world verification**: The "ground truth" for many scientific tasks often requires real experiments or rigorous simulations to obtain, and this verification process may take months or even years (such as verifying the real effectiveness of a drug molecule design). This causes AI for Science evaluations to naturally suffer from a structural contradiction where the "evaluation feedback cycle is far longer than the model iteration cycle." How to design faster proxy metrics to approximate model evaluation on such long-cycle tasks remains a core methodological challenge in this field.

</details>

**14. What New Requirements Does the Development of AI for Science Pose for Traditional Scientific Talent Cultivation and Research Organizational Models?**

<details>
<summary>💡 Show Answer</summary>

- **Rising demand for interdisciplinary talent**: Interdisciplinary scientific talent—who possess knowledge in specific scientific fields while also understanding how to collaborate effectively with AI systems (designing suitable research workflows and evaluating the boundaries of credibility of AI outputs)—is increasingly becoming a key cultivation focus for research institutions. This directly relates to the "shift in human-AI collaboration paradigms" discussed earlier;
- **Potential flattening of research organizational structures**: Traditional scientific research often relies on division of labor within relatively large teams to execute various stages from literature review to experimental execution. If AI Agents can undertake a significant portion of this workload, it could theoretically enable smaller teams—or even individual researchers—to complete complex research projects that previously required large teams. This may impact traditional, scale-oriented laboratory organizational models while fostering more flexible, rapidly iterating small scientific research team formats;
- **New challenges in research integrity and attribution of responsibility**: When AI is deeply involved in hypothesis generation, experimental design, and even result interpretation, how to delineate the respective contributions of humans and AI in research outcomes, and how to maintain research integrity when AI might make errors or produce "hallucinated" fabricated results, become new issues that scientific ethics and evaluation systems must re-examine. This creates new cross-cutting requirements with technical topics previously discussed, such as model safety alignment and interpretability, within the specific application context of research integrity.

</details>

**15. How Should We View the Challenges Faced by AI-Generated Scientific Discoveries in Reproducibility and the Peer Review System?**

<details>
<summary>💡 Show Answer</summary>

The scientific community has long relied on peer review and independent replication as core mechanisms to guarantee the credibility of research findings. The deep involvement of AI in the scientific discovery workflow presents new challenges to this traditional framework:

- **Opacity of the generation process increases replication difficulty**: If the hypothesis generation and scheme design process of a scientific discovery rely heavily on the internal reasoning of a specific AI model (and this reasoning process itself may be difficult to fully replicate, especially involving factors such as random sampling and model version iterations), independent researchers attempting to replicate the same discovery path may face difficulties not traditionally encountered. This directly relates to model interpretability discussed earlier, as well as the relatively niche yet important topic of "reproducibility of model outputs";
- **Peer review systems require new evaluation capabilities**: In addition to evaluating the scientific validity of research conclusions themselves, reviewers may also need the ability to assess "what role AI played in this research and whether its outputs underwent sufficient independent verification." This imposes new demands on the knowledge structure of reviewers. The academic community also needs to gradually establish more granular disclosure and review norms for research with deep AI involvement (similar to the disclosure requirements currently expected by academic publishers for AI-assisted writing, but extended to cover deeper stages such as study design and data analysis);
- **Suggested directions for response**: A pragmatic direction is to require researchers to transparently disclose the specific roles and contributions undertaken by AI in the research workflow, while upholding the baseline requirement that "hypotheses and schemes produced by AI must ultimately be validated through independent, reproducible real-world experiments or rigorous simulations," which should not be relaxed due to AI involvement. That is, AI can dramatically increase the efficiency of hypothesis generation and scheme design, but cannot replace the core principle of "independent verifiability" in scientific methodology itself.

</details>

