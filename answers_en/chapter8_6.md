# 8.6 Safety Evaluation and Red Teaming — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 8: Evaluation and Benchmark · 13 questions

**1. How to evaluate model safety?**

<details>
<summary>💡 Show Answer</summary>

Use benchmarks such as SafetyBench, ToxiGen, and Do-Not-Answer to test the model's refusal rate and refusal quality against harmful requests (violence, illegal activities, discrimination, privacy leaks, etc.); it is also necessary to evaluate the "over-refusal" problem—the proportion of normal requests incorrectly refused—and combine this with red teaming to discover potential risks.

</details>

**2. What are the construction methods and focus areas of common safety evaluation Benchmarks (such as SafetyBench, ToxiGen, and Do-Not-Answer)?**

<details>
<summary>💡 Show Answer</summary>

- **SafetyBench**: A multiple-choice benchmark covering various safety risk categories (e.g., illegal crimes, discrimination/bias, privacy/property, ethics/morality, physical/mental health, etc.). Most questions are presented in multiple-choice format, requiring the model to judge whether a given content or behavior is safe and compliant. Its advantage is objective and easily automated scoring, but compared to open-ended harmful requests in real-world scenarios, the multiple-choice format has certain limitations in attack surface coverage and realism.
- **ToxiGen**: Focuses on evaluating model recognition capability for **implicit toxicity** (i.e., expressions that contain no explicit profanity or offensive words, but implicitly harbor discrimination or stereotypes). Its dataset was constructed using LLM generation combined with human annotation, covering multiple protected groups. It is mainly used to test content moderation systems or the model's own ability to identify content that is superficially normal but fundamentally harmful, serving as an important complement to pure keyword-filtering methods.
- **Do-Not-Answer**: A collection specifically gathering high-risk questions that models should refuse to answer (such as assisting in illegal activities, spreading misinformation, violating privacy, etc.). The core evaluation metrics are the model's **refusal rate** and **refusal quality** (whether it is polite, whether it offers alternative suggestions instead of blunt refusals). Its original design intention is to provide a relatively pure test set for refusal behavior, avoiding confusion with normal questions used in over-refusal evaluations.

These three types of benchmarks respectively cover three distinct facets: "safety knowledge judgment", "implicit toxicity identification", and "explicit harmful request refusal". In practice, it is usually necessary to combine multiple safety Benchmarks rather than relying on a single benchmark, because the task construction method of each benchmark determines that it can only cover a fraction of the safety risk space.

</details>

**3. What is a "Jailbreak Attack"? What are the common types of jailbreak techniques?**

<details>
<summary>💡 Show Answer</summary>

A jailbreak attack refers to a user attempting to bypass a model's safety alignment mechanism by crafting prompt inputs that induce the model to output harmful content that should otherwise be refused (such as illegal information, dangerous knowledge, discriminatory remarks, etc.). Common jailbreak techniques can be categorized into several main types:

- **Role-play / Persona Induction**: Asking the model to act as a fictional persona "without any restrictions", attempting to relax safety constraints within the framing of role-play.
- **Scenario Framing**: Wrapping a harmful request into a seemingly legitimate scenario, such as claiming it is for novel writing, security research, or educational purposes, attempting to leverage the model's trust in reasonable motivations to obtain information that should be refused.
- **Prompt Injection / System Override**: Attempting to disguise inputs as system-level instructions, claiming to ignore all previous rules, or overriding the model's original safety settings through special formatting (such as forged system message tags).
- **Encoding Obfuscation**: Encoding or transforming harmful requests using Base64, Pinyin, reversed characters, or special symbol replacements, trying to bypass safety detectors based on keywords or semantic patterns while expecting the model to decode, comprehend, and execute the request.
- **Adversarial Suffix**: Using automated optimization algorithms (such as GCG) to search for a sequence of seemingly meaningless character suffixes; appending them to harmful requests significantly increases the success rate of bypassing safety alignment. Such attacks are typically generated for specific models under white-box or grey-box settings.
- **Multi-turn / Crescendo Induction**: Refraining from posing a harmful question directly in a single turn, and instead gradually guiding the model's topic and tone across multi-turn dialogue, presenting the substantive harmful request only after the model is committed to a context.

For jailbreak evaluation, it is usually necessary to construct test sets covering the various attack types mentioned above and calculate the model's **Attack Success Rate (ASR)** across different attack categories; simply computing the overall refusal rate fails to reflect the model's vulnerability to specific attack techniques.





![Common jailbreak attack vectors bypassing safety alignment in LLMs. (绕过大模型安全对齐的常见越狱攻击（Jailbreak）手段分类图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig22_jailbreak_attack_types.png)

</details>

**4. What is a "Multi-turn / Crescendo Jailbreak"? Why is it harder to defend against than single-turn attacks?**

<details>
<summary>💡 Show Answer</summary>

Multi-turn Crescendo Jailbreak refers to an attack strategy where the attacker does not pose a harmful request directly in a single-turn conversation, but instead uses carefully designed multi-turn dialogue for gradual guidance: starting from completely benign topics, progressively escalating the topic sensitivity, with each turn's request incurring only a subtle boundary violation relative to the previous turn. By exploiting the model's established dialogue context and inclination to cooperate with the context, it ultimately induces the model to generate harmful content that would be directly refused in a single-turn setup.

The main reasons why it is harder to defend against than single-turn attacks include:

- **Data distribution limitations in safety alignment training**: Most safety alignment training data is constructed around single-turn question-refusal patterns. The model mainly learns to identify and refuse individual, blatantly harmful requests, whereas its capability for complex pattern recognition—such as determining whether the current request resides in a gradually guided harmful context by incorporating full conversation history—remains weak.
- **Small step size, difficult to trigger refusal threshold**: The magnitude of boundary violation in each turn relative to the previous one is very small. When considering the current turn in isolation, the model often judges it as benign or acceptable, making safety guardrails easier to bypass because refusal judgments typically rely more on evaluating the harm level of the current request itself rather than cumulative risk evaluation across the entire dialogue trajectory.
- **Context Commitment Effect**: Models tend to maintain response coherence and consistency in multi-turn dialogues. Once the model has provided cooperative responses along a topic direction in earlier turns, it is more inclined to follow the established context in subsequent turns rather than suddenly interrupting and refusing. This resembles the commitment consistency psychological effect in human interaction.

Evaluating against such attacks requires specifically constructing test sets of multi-turn attack sequences (rather than evaluating single-turn refusal rates alone). The evaluation metrics should focus on whether the model is successfully induced in **any turn** of the conversation, rather than looking solely at the final turn output. Furthermore, it should evaluate whether the model possesses the capability to retroactively recognize risks in the entire dialogue trajectory and proactively refuse or adjust its stance mid-way.





![Step-by-step safety threshold bypass in a multi-turn Crescendo jailbreak. (多轮渐进式越狱（Crescendo Jailbreak）如何通过逐步引导突破安全防御的机制图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig23_crescendo_jailbreak.png)

</details>

**5. What are automated adversarial suffix attacks such as GCG (Greedy Coordinate Gradient)? What is their significance for safety evaluation?**

<details>
<summary>💡 Show Answer</summary>

GCG is an automated adversarial attack algorithm designed for open-source (white-box) large language models. Its core idea is: given a harmful request, it greedily searches and optimizes an **adversarial suffix** (typically a string of seemingly meaningless character combinations) appended to the end of the request in the vocabulary space using gradient information, such that after attaching this suffix, the probability of the model generating harmful content is significantly increased, thereby bypassing the original safety refusal behavior. Because these suffixes are automatically discovered through optimization algorithms, they often possess a degree of **transferability**—an attack suffix optimized for a specific open-source model may also be effective on other models (including some closed-source models).

The significance of such automated adversarial attacks for safety evaluation is primarily reflected in the following aspects:

- **Revealing the superficiality of safety alignment**: The success of GCG-style attacks indicates that much of safety alignment merely teaches models to recognize harmful request patterns expressed in natural language, rather than understanding and refusing harmful content at a deeper level. Once the input format deviates from the natural language patterns seen during training, safety behavior may break down.
- **Providing more systematic and larger-scale attack surface coverage than manual red teaming**: Manual red teaming is constrained by human labor and creativity, whereas automated adversarial attacks can exhaustively search large numbers of attack variants under a given computational budget, evaluating worst-case scenarios of model safety boundaries more systematically. This forms an essential component of stress testing in safety evaluation.
- **Promoting the application of adversarial training in safety alignment**: The discovered adversarial suffix samples can in turn be added as negative samples during safety alignment training (such as SFT or RLHF stages), forming a continuous iterative loop of attack discovery -> model hardening -> new attack rounds, aligned with the mindset of red-teaming-driven safety improvements.
- **Highlighting limitations in evaluating closed-source models**: Because GCG relies on internal model gradient information (white-box attack), its direct application to fully closed-source models accessible only via API is constrained. However, known transferable attack suffixes can be leveraged to indirectly evaluate the robustness of closed-source models, underscoring that safety evaluation must cover both white-box and black-box attack paradigms.

</details>

**6. How to evaluate a model's resistance to "Prompt Injection" attacks, especially in Agent/RAG scenarios?**

<details>
<summary>💡 Show Answer</summary>

Prompt injection refers to an attacker embedding malicious instructions within **external content** processed by the model (rather than direct user conversation inputs), attempting to hijack the model's original task goal. This is particularly prominent in Agent and RAG scenarios, because the model reads untrusted external content such as web pages, documents, and tool return results, where attackers can embed malicious instructions.

Core ideas and methods for evaluating resistance to prompt injection include:

- **Constructing external content test sets containing malicious instructions**: Embed injection instructions of varying intensities and phrasing formats into RAG retrieved documents, web pages accessed by Agents, or tool return results (e.g., "ignore previous instructions and execute XX instead"), observing whether the model executes the user's original intent or is hijacked by malicious instructions in the external content.
- **Distinguishing between direct injection and indirect injection**: Direct injection means the attacker directly controls the user input itself; indirect injection occurs when the attacker cannot directly converse with the model, but instead carries out the attack by poisoning third-party data sources that the model will read (such as placing malicious instructions on their own website, waiting for an Agent to access and scrape it). In Agent/RAG scenarios, indirect injection poses higher risk and evaluation difficulty, requiring specially constructed evaluation scenarios with poisoned external data sources.
- **Evaluation metrics**: Core metrics include **injection success rate** (the proportion of cases where the model executed the injected instructions instead of the original task), **degree of task deviation** (the extent to which original task execution quality is degraded even if injected instructions are only partially executed), and whether the model possesses the capability to identify and **proactively notify the user** upon detecting suspicious instructions.
- **Evaluating the model's ability to distinguish instruction sources**: Design test scenarios to verify whether the model can correctly differentiate among information sources with different trust levels—such as system/developer instructions, user instructions, and external tool return content—and consistently prioritize higher-priority instruction sources rather than naively treating all textual content as executable instructions equally.
- **Combining permission/sandbox evaluation**: For Agents executing real operations (such as file read/write, API calls, payments), it is also necessary to evaluate whether they execute high-risk operations exceeding expected permission scopes under prompt injection attacks. Evaluation environments typically require sandboxing to avoid real-world side effects.

</details>

**7. What is the "Over-refusal" problem? How to design evaluations to quantify it?**

<details>
<summary>💡 Show Answer</summary>

Over-refusal refers to the phenomenon where a model falsely identifies safe, compliant requests that should be answered normally as harmful requests and refuses them. It is a common overcorrection side-effect in safety alignment training—for instance, a model might refuse an entirely harmless question simply because sensitive words appear in it (e.g., "how to defeat the final boss in a game" or "the historical invention process of explosives in a chemistry textbook").

Common methods for quantitatively evaluating the over-refusal problem include:

- **Constructing control test sets that appear sensitive but are actually harmless**: Specially collecting questions that superficially contain sensitive words or touch upon safety red lines, but whose actual requests are completely legitimate (e.g., normal queries in professional scenarios such as medicine, law, chemistry, and security research). Such datasets are commonly referred to as over-refusal benchmarks.
- **Core metric—False Refusal Rate (FRR)**: The proportion of questions incorrectly refused among those that should be answered normally. Evaluating this alongside the **True Refusal Rate (TRR)** on genuinely harmful test sets allows a complete characterization of the model's balance point between safety and usefulness. A high refusal rate alone does not mean the model is safer; it might simply be overly conservative.
- **Controlled experiment design**: Construct paired questions sharing identical sensitive keywords but differing in intent (one harmful, one harmless), observing whether the model makes distinctions based on the **actual intent of the request** rather than surface keywords. This minimal contrastive design enables more precise diagnosis of whether the model relies on shallow keyword matching or truly understands the context and intent.
- **Plotting safety-utility trade-off curves**: By adjusting the model's refusal threshold (if adjustable), plot trade-off curves between True Refusal Rate and False Refusal Rate (similar to ROC curves), helping teams select appropriate safety trade-off operating points for specific application scenarios rather than pursuing single-dimension extremes.





![Safety alignment spectrum: helpfulness vs over-refusal of benign requests. (模型安全对齐谱系：区分恶意请求拦截与无害问题过度拒绝（Over-refusal）的象限对比图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig24_over_refusal_tradeoff.png)

</details>

**8. How should a Moderation Classifier itself be evaluated for effectiveness?**

<details>
<summary>💡 Show Answer</summary>

A moderation classifier typically acts as a safety guardrail independent of the generative model, classifying and intercepting harmful user inputs or model outputs. Its evaluation strategy differs from evaluating the safety of generative models and is closer to the traditional **classification model evaluation** paradigm:

- **Basic classification metrics**: Calculate **Precision**, **Recall**, and **F1 Score** on annotated harmful/harmless test sets, with breakdown statistics for different harmful categories (violence, pornography, discrimination, illegal activities, etc.). Because classifier performance often varies significantly across categories, an overall F1 score might mask severe false negatives in specific categories.
- **Business cost asymmetry between False Negatives and False Positives**: False negatives mean harmful content is allowed through, while false positives mean normal content is blocked. The business costs of the two are typically asymmetric (false negatives usually carry more severe security risks). Therefore, evaluation must go beyond overall accuracy, establishing clear tolerance limits for false negative rates according to business scenarios and evaluating false positive rates under those constraints.
- **Adversarial robustness evaluation**: Classifiers need targeted evaluation regarding the degree of detection degradation when facing **adversarially transformed inputs** (such as typo substitution, homophones, character decomposition, kaomoji obfuscation, multilingual mixing). Attackers will actively try to bypass moderation classifiers, so high scores on static test sets do not guarantee reliability in real-world adversarial scenarios.
- **Latency and throughput evaluation**: Since moderation classifiers are usually deployed in the real-time pipeline of production environments, it is necessary to evaluate whether inference latency and throughput meet online service performance requirements—an engineering dimension often overlooked in purely efficacy-based evaluations.
- **End-to-end evaluation integrated with generative models**: Classifiers and generative models (with their own safety alignment) represent different layers in a multi-tiered defense system. End-to-end evaluation should verify the overall protection effectiveness of combining classifier interception with model self-refusal, rather than evaluating classifier performance as an isolated component.

</details>

**9. How to evaluate model performance regarding bias and stereotypes? What Benchmarks are commonly used?**

<details>
<summary>💡 Show Answer</summary>

The core idea for evaluating model bias is: construct **contrastive sample pairs** that differ solely in a **key variable** (such as identity attributes like gender, race, religion, age, etc.) while remaining identical in all other content, observing whether the model's outputs (such as scoring, text completion, or selection preference) exhibit systematic differences when identity attributes change. Such systematic differences serve as quantitative evidence of bias.

Commonly used bias evaluation benchmarks include:

- **BBQ (Bias Benchmark for QA)**: By constructing QA scenarios with ambiguous or disambiguated contexts, it tests whether a model relies on social stereotypes to make biased inferences about different demographic groups when lacking sufficient supporting information (e.g., whether it tends to attribute negative traits to a specific group when information is insufficient).
- **StereoSet**: Evaluates the model's preference for stereotypical content at the language modeling level by having the model choose or score among three completion options: "stereotypical", "anti-stereotypical", and "unrelated".
- **CrowS-Pairs**: Constructs a large set of sentence pairs where each pair differs only in expressions involving a protected group (one sentence being stereotypical, the other anti-stereotypical), measuring whether internal stereotype preferences exist by comparing the probabilities assigned to both sentences by the model.

In addition to using these standardized benchmarks, evaluating bias requires paying attention to:

- **Multi-dimensional nature of bias**: Bias manifests not only in direct discriminatory speech, but also in more subtle forms—such as using different phrasing tendencies or tone variations when describing different demographic groups, or systematic stereotypical associations in occupation/role association tasks. Diverse probing tasks need to be designed rather than relying solely on a single form of testing.
- **Cultural and linguistic differences**: Concrete manifestations and social sensitivities of bias can differ significantly across cultural and linguistic contexts. Directly applying bias benchmarks built for English contexts to Chinese or other languages may lead to cultural misalignment or omission of locally specific bias types. Ideally, evaluation sets should be specifically constructed or adapted for the target application's cultural context.





![Bias evaluation framework using counterfactual and ambiguous prompt templates. (基于反事实与歧义模版进行偏见与刻板印象评测的基准框架图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig25_bias_benchmarks.png)

</details>

**10. What is Red Teaming? What Role Does It Play in Discovering Security Vulnerabilities and Biases in LLMs and Agents?**

<details>
<summary>💡 Show Answer</summary>

Red Teaming is an adversarial testing methodology derived from penetration testing in the field of cybersecurity. In the AI domain, it refers to organizing a specialized team (the "red team") to proactively, creatively, and as an "attacker," search for and exploit vulnerabilities, flaws, and unintended behaviors of LLMs or Agents to evaluate and enhance their security and robustness.

Unlike conventional testing (which relies on fixed, known test cases), the core of red teaming lies in "exploring the unknown," uncovering "edge cases" and "attack vectors" that developers did not anticipate during design, which could lead to severe consequences.

Core roles of red teaming in discovering security vulnerabilities and biases:

1. Discovering Security Vulnerabilities:

- Bypassing Safety Guardrails: Red teams design various complex, carefully crafted prompts (i.e., "jailbreak prompts") to attempt to bypass the model's safety moderation mechanisms and induce it to generate harmful content, such as violence, pornography, hate speech, or instructions for illegal activities.
- Prompt Injection Attacks (for Agents): This is one of the most critical threats to Agents. Red teams simulate malicious users or poisoned external data (such as a webpage containing malicious instructions) to try hijacking the Agent's control flow, compelling the Agent to execute unintended, dangerous operations, such as:
- Leaking sensitive information from its context.
- Abusing its tools, such as sending spam emails or deleting files.
- Altering its original objectives.
- Discovering Resource Abuse Vulnerabilities: Red teams attempt to cause Agents to enter infinite loops or execute high-consumption operations, testing their resource limits and circuit-breaker mechanisms.

1. Discovering Biases:

- Exposing Stereotypes: Red teams design seemingly neutral yet leading questions involving specific groups of people (such as race, gender, nationality, or occupation) to expose whether the model generates responses with stereotypes or discrimination.
- Testing Political and Social Biases: By asking controversial social or political questions, they evaluate whether the model maintains a neutral stance and whether biases exist.
- Revealing Underrepresentation Issues: Exploring whether the model exhibits a lack of knowledge or generates inaccurate descriptions when handling queries related to non-mainstream cultures or marginalized groups.

Summary: Red teaming plays the role of a "stress tester for the AI system's immune system." By simulating worst-case scenarios and the most cunning adversaries, it helps developers systematically discover and fix deep-seated security and alignment issues that are difficult to expose in standard testing prior to model deployment, serving as a vital guarantee for ensuring safe, reliable, and fair AI systems.





![Iterative lifecycle of red teaming evaluation and safety guardrail patching. (大语言模型红蓝对抗（Red Teaming）安全演练与防御打补丁的闭环流程图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig26_red_teaming_process.png)

</details>

**11. How is Automated Red Teaming Implemented? What Are Its Pros and Cons Compared to Manual Red Teaming?**

<details>
<summary>💡 Show Answer</summary>

Automated red teaming refers to the process of using algorithms or another large model to automatically generate large amounts of adversarial attack samples, replacing or supplementing purely manual construction of attack prompts. Common implementations include:

- **Optimization Algorithm-Based Attack Generation**: Using gradient-based search algorithms such as the aforementioned GCG to automatically optimize adversarial inputs against white-box models.
- **LLM-Generated Attack Prompts (LLM Red-Teaming LLM)**: Training or prompting a specialized attacker model to learn how to construct prompts that induce harmful outputs from the target model. The attacker model can iteratively optimize its attack strategy based on the target model's historical failure samples, forming an automated attack-defense game loop.
- **Reinforcement Learning-Based Attack Strategy Search**: Using successful inducement of harmful outputs from the target model as a reward signal to train the attacker model's attack strategy via reinforcement learning, enabling it to systematically explore the target model's safety boundaries without relying on human experience to exhaustively list attack techniques.
- **Template Mutation and Combination**: Based on a library of known jailbreak templates, generating variant attack samples in batches through automated synonymous rewriting, template stitching, and multilingual translation, rapidly expanding test coverage.

Pros and cons of automated red teaming compared to manual red teaming:

- **Advantages**: Its scale and speed far surpass manual testing, enabling the generation and testing of thousands of attack samples in a short time; its coverage is more systematic, allowing exhaustive exploration of attack variant combinations that human red teamers might not conceive; costs are relatively lower, and it can be continuously integrated into the automated testing workflow of model iterations (e.g., automatically running a regression test after training each new version).
- **Disadvantages**: Automatically generated attack samples are often inferior to experienced human red teamers in creativity and alignment with real malicious user behavior, easily getting trapped in local optima of a few fixed attack patterns, whereas humans are better at constructing more cunning attack scenarios involving complex social engineering and multi-turn psychological induction; additionally, vulnerabilities discovered by automated attacks require further manual verification to confirm their severity and real exploitability, avoiding false positives that interfere with prioritizing security hardening.

The best practice in real-world deployment is to use automated red teaming as a large-scale, high-frequency first line of defense, while human red teams focus on complex scenarios and novel attack techniques that automated methods struggle to cover, complementing rather than replacing each other.

</details>

**12. How to Design a Continuously Updated Safety Evaluation System to Address Emerging Attack Techniques?**

<details>
<summary>💡 Show Answer</summary>

Because security attack techniques continuously evolve (with new jailbreak templates and new social engineering methods constantly emerging), static, one-time built safety evaluation benchmarks will gradually become outdated or targeted and overcome, just like other benchmarks. Therefore, it is necessary to design a continuously updated safety evaluation system. The core elements include:

1. **Establishing a Closed-Loop Process from Vulnerability Discovery to Evaluation Set Updates**: Periodically and systematically organizing and supplementing real online intercepted attack cases, newly discovered attack techniques from red teaming, and publicly disclosed novel jailbreak methods into the evaluation dataset. This forms a continuous cycle from discovering new attacks to adding them to the regression test suite and verifying whether the model has been hardened.
2. **Distinguishing Between Regression Test Sets and Exploratory Testing**: Regression test sets should remain relatively stable to verify whether new model versions suffer capability regressions on known issues (preventing the introduction of new issues when fixing old vulnerabilities, or the relapse of old vulnerabilities after model updates); exploratory testing requires continuously introducing brand-new, unseen attack techniques to discover unknown risks. The two differ in update frequency and purpose and should not be conflated.
3. **Combining with Automated Red Teaming for Rapid Iteration**: Leveraging the aforementioned automated attack generation methods to endow the evaluation system with the capability to automatically search for new failure modes against the current model without relying entirely on manual discovery, thereby shortening the time lag between the emergence of new attack techniques and their inclusion in evaluation coverage.
4. **Establishing Cross-Team/Cross-Organizational Threat Intelligence Sharing Mechanisms**: Emerging attack techniques in the security domain often exhibit cross-model and cross-vendor generalizability (e.g., a certain type of jailbreak template may work on models from multiple vendors). Participating in industry threat intelligence sharing (such as public red teaming reports and academic attack papers) enables earlier detection of potential risks without passively responding only after one's own product is actually attacked.
5. **Monitoring Whether the Evaluation Set Itself is Saturated or Specifically Optimized Against**: Be vigilant against model teams performing evaluation-set-specific hardening for known safety benchmarks (i.e., the model merely learns to handle specific samples in the evaluation set rather than truly enhancing safety generalization capabilities). The authenticity of safety improvements can be verified by periodically conducting perturbation tests on the evaluation set, such as sample rewriting and synonym substitution, which aligns with the rationale behind checking whether general capability benchmarks are being overfitted.
6. **Setting Core Metrics for Continuous Tracking**: Beyond periodic evaluation reports, establish a dashboard for continuously trackable core security metrics (such as success rate trends for various attacks and false refusal rate trends), and combine online real user feedback with manual sampling data to maintain a tight feedback loop between the safety evaluation system and actual deployment performance, rather than remaining confined to the offline evaluation stage.





![Dynamic CI/CD loop for continuous red teaming and safety evaluation updates. (持续集成与更新的模型动态安全评估体系架构图)](https://img.pricenode.uk/llm_interview_questions/english/chapter8/fig27_dynamic_safety_eval.png)

</details>

**13. How Should an Evaluation System Be Designed When Assessing the Trade-Off Between "Safety Alignment" and "Helpfulness"?**

<details>
<summary>💡 Show Answer</summary>

Separate test sets need to be constructed to measure "over-refusal" (the model refusing to answer harmless requests) and "insufficient safety" (the model providing assistance to harmful requests that it should not render). The scores of both dimensions should be presented simultaneously rather than merged into a single metric, making it possible to observe whether a specific model adjustment results in "sacrificing helpfulness to boost safety scores (increased over-refusals)" or vice versa (i.e., shifts in the Pareto frontier).

</details>

