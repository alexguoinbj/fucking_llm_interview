# 12.9 Multimodal Evaluation and Benchmarks — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 15 questions

**1. How Are Multimodal Models Evaluated?**

<details>
<summary>💡 Show Answer</summary>

**Understanding-oriented Evaluation:** VQA series (VQA v2, GQA, OK-VQA): visual question answering accuracy; POPE: specialized evaluation of hallucination severity (object existence judgment); MMBench, MMStar: comprehensive multi-dimensional understanding benchmarks; SEED-Bench, MM-Vet: multi-type visual understanding and reasoning; ScienceQA: multimodal scientific reasoning.

**Generation-oriented Evaluation:** NoCaps, COCO Caption (image captioning quality, measured via metrics like CIDEr, BLEU, METEOR); FID (image generation quality evaluation).

**OCR and Document Understanding:** TextVQA, DocVQA, ChartQA: text recognition and document understanding capabilities.

**Comprehensive Benchmarks:** MMMU (college-level multi-discipline multimodal understanding); MathVista (mathematical visual reasoning).

**Human Evaluation:** Human scoring of fluency, accuracy, and hallucination degree in generated content serves as a vital complement to automated evaluations.

</details>

**2. How Is Multimodal Generation Quality Evaluated?**

<details>
<summary>💡 Show Answer</summary>

**Automated Metrics (Text-Image Generation):** BLEU, METEOR, ROUGE (text quality metrics based on n-gram overlap); CIDEr (TF-IDF weighted image caption evaluation metric, highly sensitive to human descriptions); SPICE (scene graph-based semantic evaluation focusing on the accuracy of objects, attributes, and relations).

**Image Generation Evaluation:** FID (Fréchet Inception Distance, measuring the distance between generated and real image distributions, lower is better); CLIP Score (computing the CLIP similarity between generated images and text descriptions to measure image-text consistency); Aesthetic Score (using aesthetic scoring models to evaluate visual quality).

**Multimodal Understanding Evaluation:** VQA accuracy, MMBench overall score; POPE, HallusionBench (specifically evaluating hallucination levels).

**Human Evaluation:** Manual scoring of fluency, accuracy, and relevance; comparative evaluation (A/B testing); considered the gold standard for hallucination evaluation despite high costs.

</details>

**3. What Causes Hallucinations in Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

Multimodal hallucination refers to a model generating content that does not exist in the visual input. The main causes include:

1. **Strong Language Priors**: Language models learn strong linguistic statistical patterns during pre-training, which may cause them to ignore visual content during generation and rely on linguistic inertia to generate plausible yet inaccurate descriptions.
2. **Insufficient Image-Text Alignment**: Visual features are not fully mapped into the semantic space of the language model, preventing precise understanding of image details and prompting generation based on "imagination."
3. **Training Data Bias**: Training image-text pairs contain captions that do not fully correspond to the images, leading the model to learn inaccurate vision-language associations.
4. **Loss of Visual Information**: When images are encoded into a fixed number of tokens, detailed information (small objects, text, spatial relations) may be lost, preventing accurate descriptions.
5. **Decoding Strategy Issues**: During autoregressive decoding, early erroneous tokens steer subsequent tokens in incorrect directions (error accumulation).
6. **Lack of Rejection Capability**: Models are not trained to express uncertainty when uncertain, tending instead to output deterministic answers.

**Mitigation Methods:** RLHF/DPO alignment, contrastive decoding (e.g., ICD), visual information enhancement (higher resolutions, more patches), specialized hallucination evaluation combined with data augmentation.

</details>

**4. How Do the Manifestations of "Hallucination" Differ Between VLMs and Pure Text LLMs?**

<details>
<summary>💡 Show Answer</summary>

Both VLMs and pure text LLMs exhibit hallucinations by generating non-factual or fabricated content; however, VLM hallucinations are anchored to visual inputs, manifesting primarily as forcibly "inserting" incorrect or non-existent visual facts into descriptions:

1. **Object Hallucination**: Describing objects completely absent from the image (pure text LLMs fabricate facts out of thin air, such as inventing a non-existent book title; VLMs incorrectly "see" things that are not there). Example: The image contains only a cat, but the VLM describes "a cat and a small dog."
2. **Attribute Hallucination**: Correctly identifying objects while misdescribing attributes such as color, shape, or count (pure text LLMs misremember facts, whereas VLMs misread visual details). Example: Describing a blue shirt as red, or two apples as three.
3. **Relationship Hallucination**: Correctly identifying multiple objects while incorrectly describing their spatial or interaction relationships (pure text LLMs confuse conceptual relationships, whereas VLMs confuse physical spatial relationships). Example: Describing "a book beside a cup" as "a book inside a cup."

</details>

**5. Please Explain the Meaning of Grounding in VLMs. How Do We Evaluate a VLM's Grounding Capability?**

<details>
<summary>💡 Show Answer</summary>

Grounding (localization / referring expression grounding) refers to the capability to establish precise correspondences between specific concepts or phrases in text and specific pixel regions in an image—the model must know not only "what" is in the picture, but also "where" it is. This is crucial for tasks such as coreference resolution, visual grounding, and interpretability, typically requiring the inclusion of localization data annotated with bounding boxes during training.

Evaluating grounding capability typically relies on datasets with spatial annotations (e.g., RefCOCO, Visual Genome), using the following primary methods:

1. **Referring Expression Grounding**: Given an image and a descriptive phrase (e.g., "the woman in the red dress"), the model outputs a bounding box. The Intersection over Union $IoU = Overlap Area / Union Area$ is computed between the predicted bounding box and the human-annotated Ground Truth BBox. A threshold (e.g., 0.5) is set to determine localization correctness and calculate Accuracy@IoU.
2. **Visual Grounding Dialogue**: The model outputs corresponding coordinates/bounding boxes simultaneously while generating text referencing objects.
3. **Attention Map Visualization**: Visualizing the model's attention activation regions when generating descriptions of specific objects to help diagnose whether it truly "saw" the corresponding content.



![评估视觉定位能力的交并比（IoU）计算向量示意图 (Intersection-over-Union (IoU) calculation for evaluating visual grounding localization.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig31_visual_grounding_eval.png)

</details>

**6. How Do We Evaluate a Multimodal Model's Capability in Long Video Understanding Tasks, and What Unique Challenges Are Faced?**

<details>
<summary>💡 Show Answer</summary>

Evaluating long video understanding capabilities typically requires constructing video QA or video summarization benchmark suites that explicitly cover **longer time spans** (e.g., several minutes to tens of minutes, rather than the typical few seconds to tens of seconds in conventional short video benchmarks). The designed questions deliberately require the model to comprehensively understand the progression of events **spanning extended temporal spans**, causal relationships between different segments, or to integrate multiple pieces of information scattered across different positions in the video to answer correctly (rather than answering based solely on observing a single local segment). This specifically tests whether the model possesses genuine long-temporal understanding abilities, rather than merely "guessing the correct answer after taking a few random samples."

Unique challenges faced in such evaluations include:

- **Tradeoff Between Frame Sampling Coverage and Computational Cost**: Ultra-long videos cannot have every single frame processed entirely. Designing a sampling strategy that ensures sufficient temporal coverage (without missing key event timestamps) while avoiding excessive computational and GPU memory costs caused by over-dense sampling is itself a delicate tradeoff. The choice of sampling strategy can even directly impact the fairness and comparability of evaluation results;
- **Limitations of Model Long-Temporal Modeling Capabilities**: Even if sampling is resolved, whether the model's internal long-range temporal dependency modeling (similar to long-context text challenges) is sufficiently strong directly determines if it can truly "remember" and synthesize information appearing early in the video to answer questions regarding later parts or the overall content;
- **Scarcity of High-Quality Long Video Annotated Data**: Constructing high-quality QA pairs for long videos that necessitate comprehensive understanding of global information is significantly more difficult and costly to annotate than for short videos or static images. Annotators must watch and thoroughly comprehend entire long video clips, which explains the current relative scarcity and inconsistent quality of long video evaluation datasets.

</details>

**7. How Is the "Instruction Following" Capability of Multimodal Models Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluating instruction-following capability focuses on the core question: can the model accurately understand and strictly execute specific user instructions that may contain complex constraints, rather than merely giving a "broadly relevant answer"? Evaluation methodologies typically include the following key aspects:

- **Test Suite Design**: Specially constructing test samples pairing images/videos with complex natural language instructions. Instruction content deliberately spans scenarios demanding precise execution, such as requiring exact counting ("How many birds are in the image in total?"), requiring spatial relationship reasoning ("What object is located to the left of the cup?"), specifying strict output formatting ("Answer with a phrase of no more than 10 words"), or requesting multi-step compound instructions ("First describe the weather in the picture, then judge whether it is suitable for outdoor sports");
- **Automated Evaluation**: For instructions with clear, objective correct answers (such as counting tasks), correctness can be automatically evaluated via rule-based string matching or numeric comparisons. For more open-ended instruction compliance (e.g., format compliance, missing specific constraints), LLM-as-Judge approaches (using another powerful model as an evaluator) are frequently employed to provide flexible scoring;
- **Human Evaluation**: For subjective evaluations requiring comprehensive judgment on "whether the model truly and fully fulfilled all requirements of the user instruction," human evaluation remains an indispensable complementary tool, especially when automated evaluation reliability is uncertain.
This form of evaluation goes beyond simple "accuracy of content understanding" to specifically examine the model's **execution precision and completeness** regarding instruction details, representing a key dimension for measuring real-world usability.

</details>

**8. What Is the Application of LLM-as-Judge in Multimodal Evaluation, and What Are Its Limitations?**

<details>
<summary>💡 Show Answer</summary>

LLM-as-Judge refers to using a powerful (multimodal) large model as a "judge" to score candidate model responses or compare them against reference answers or other models' outputs, replacing portions of human evaluation to lower costs and accelerate evaluation speed. In multimodal scenarios, the judge model must inspect both the visual input and candidate responses to determine whether a response accurately describes the image content. Its limitations include: the judge model itself may exhibit hallucinations or visual comprehension biases, making scores not entirely reliable; the judge model may favor superficial characteristics such as length or style (e.g., preferring longer, more detailed responses over concise ones) rather than true accuracy; furthermore, systematic biases may exist between judge model standards and human preferences, usually requiring small-scale human evaluations to calibrate consistency between automated judgments and human assessments.

</details>

**9. How to Evaluate the "Safety" of Multimodal Models (e.g., Refusal Capability for Harmful Images/Instructions)?**

<details>
<summary>💡 Show Answer</summary>

Evaluating the safety of multimodal models typically requires constructing dedicated safety evaluation benchmarks to systematically test the model's safety boundaries from multiple perspectives:

- **Inappropriate Image Recognition**: Test whether the model can correctly identify non-compliant image content (such as pornography, violence, or gore) and refuse to perform detailed analysis or description, rather than giving normal responses indiscriminately;
- **Defense Against Malicious Instructions**: Test the model's capability to handle malicious requests combined with normal images—for example, providing a photo of a person and asking the model to infer private information such as their identity or home address, or asking the model to generate discriminatory or misleading content based on an image. The model should be able to recognize the malicious nature of such requests and reject them;
- **Jailbreak Attack Testing**: Test whether the model can be "tricked" by carefully constructed adversarial inputs to bypass safety guardrails, such as embedding misleading textual prompts within images (attempting to inject malicious instructions via the visual channel that would otherwise be intercepted in pure text) or inducing the model via specially constructed visual content to give responses it should have refused. This type of testing specifically checks whether the model's safety mechanisms have vulnerabilities in multimodal scenarios that are not covered by text-only safety alignment.

The core evaluation dimensions typically include: **Refusal Rate** (the proportion of harmful requests that actually ought to be refused for which the model yields a refusal response; higher values indicate more effective safety protection); **False Refusal Rate** (the proportion of completely normal and reasonable requests to which the model erroneously responds with a refusal—this metric is equally important because if a model is "overly safe" and misjudges a large number of normal requests as harmful and rejects them, it will severely damage actual model usability and user experience; thus, safety evaluation needs to strike a balance between "effective protection" and "not falsely penalizing normal requests"); and **Appropriateness of Refusal Responses** (whether the model expresses its refusal politely and appropriately explains the reason for refusal, rather than abruptly terminating the conversation or returning unpleasant responses).



![多模态大模型安全评估与越狱防御审计流程图 (Audit process flow for multimodal model safety and jailbreak evaluation.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig32_safety_audit_flow.png)

</details>

**10. What is the "Multiple-Choice Bias" Issue in Evaluation?**

<details>
<summary>💡 Show Answer</summary>

To facilitate automated scoring (avoiding complex open-ended text answer matching problems), many multimodal benchmarks (such as MMBench, ScienceQA, etc.) rely heavily on multiple-choice questions (single-choice from multiple options) for evaluation—given an image, a question, and several candidate options, the model only needs to output the label of the correct option. Although this evaluation format is simple and objective to score, it introduces a systematic issue known as the "language shortcut": the model may completely **not need to truly understand the image content** and can guess the correct answer relying solely on several "shortcuts"—

- **Prior Knowledge of Language Models**: Relying solely on common sense embedded in the question text itself (e.g., "what is the most reasonable answer under normal circumstances") without actually looking at the image;
- **Linguistic Cues in Options**: A particular option's wording may naturally appear more "standard" or "complete," or align better grammatically with the question's phrasing. Models may learn to exploit such superficial cues rather than visual evidence to select the answer;
- **Process of Elimination**: Even when uncertain about the correct answer, eliminating obviously unreasonable distractors increases the probability of guessing correctly.
This issue causes evaluation scores under the multiple-choice format to be potentially "inflated," failing to accurately reflect the model's true visual understanding capabilities. This is also an important reason why researchers in recent years have attached increasing importance to open-ended QA evaluations or designing specialized "counterfactual" samples (as described in the next question) to test whether models genuinely rely on visual input.

</details>

**11. How to Design "Counterfactual" Evaluation Samples to Test Whether Models Truly Rely on Visual Input?**

<details>
<summary>💡 Show Answer</summary>

The core design concept of counterfactual evaluation samples is to deliberately construct image content that "contradicts language common sense/priors." Only models that truly "look at the image" carefully can give the correct answer, whereas models relying on language shortcuts will be "misled" by their own prior knowledge, resulting in incorrect yet seemingly plausible answers. Common construction methods include:

- **Commonsense-Contradicting Counterfactual Images**: For example, digitally editing a photo of a cat to add a pair of glasses and then asking the model "Is the cat in the image wearing glasses?"—if the model relies solely on the language prior that "cats usually do not wear glasses," it will incorrectly answer "No." Only a model that answers based on visual evidence can correctly identify this unusual detail;
- **Triple Control Test**: For the same question, test the model's responses under three separate settings: (1) providing the correct image genuinely relevant to the question, (2) providing an irrelevant or incorrect image, and (3) providing no image at all (question text only). Then compare the **consistency and accuracy differences** across these three settings. If the model's answers are highly similar across all three settings (indicating the model "hardly looks at the image" and relies primarily on language priors from the question text), it reveals a severe dependence on language shortcuts. Conversely, if accuracy improves significantly when the correct image is provided and degrades noticeably under the irrelevant-image or no-image settings, it demonstrates that the model indeed effectively leverages visual information.

</details>

**12. Does "Benchmark Gaming" Exist in Multimodal Models? How to Address It?**

<details>
<summary>💡 Show Answer</summary>

Yes, "benchmark gaming" is a realistic issue in the multimodal field as well. Because certain well-known benchmarks (such as classic VQA datasets) have been widely used for years, their evaluation images, question styles, and even specific sample contents have circulated and been discussed extensively on the internet. Model developers, when collecting large-scale pre-training data, may (intentionally or unintentionally) mix in data highly similar to or even duplicated from these benchmarks (i.e., the aforementioned "data contamination" problem). Alternatively, they may overly optimize for specific benchmark answer formats or scoring rules (e.g., specifically learning the concise answer style preferred by a particular benchmark). This ultimately leads to "inflated" scores on that benchmark, but such high scores do not genuinely reflect the model's generalization capabilities in novel, unseen real-world scenarios.

Common methods to address this problem include: **periodically releasing brand-new, previously undisclosed evaluation benchmarks** (ensuring evaluation data could not have appeared in the training data of any existing models); **adopting dynamically generated or continuously updated evaluation samples** (e.g., replacing test images and questions periodically rather than relying long-term on a static evaluation set); and **cross-validating across multiple benchmarks with diverse styles covering different capability dimensions**, rather than relying solely on a single leaderboard score to judge the model's true capability level, thereby avoiding "over-specialization" caused by over-optimizing for a single evaluation target.

</details>

**13. What is the Difference Between "Fine-Grained Evaluation" and "Holistic Evaluation"? Why is Fine-Grained Evaluation More Valuable?**

<details>
<summary>💡 Show Answer</summary>

**Holistic evaluation** typically yields only a single comprehensive overall score (e.g., "a model achieves an overall accuracy of 75% on a certain benchmark"). While such scores facilitate high-level comparisons across different models, they suffer from significant information loss—two models with identical overall scores may possess vastly different capability profiles (e.g., one might be strong at object recognition but weak at spatial reasoning, while the other is the exact opposite). Relying solely on overall scores conceals these differences and offers little guidance on which specific capabilities should be improved next.

**Fine-grained evaluation**, on the other hand, pre-categorizes all test samples according to specific capability dimensions (such as object recognition, attribute judgment, spatial relationship reasoning, counting capability, OCR text recognition, commonsense reasoning, causal reasoning, etc.) and calculates the model's accuracy on each specific dimension individually. The final result is presented as a multi-dimensional capability "radar chart" or itemized scorecard rather than a single vague number.

The reason fine-grained evaluation holds greater value during model R&D is that it provides clearer, more **actionable** diagnostic insights: R&D teams can explicitly identify which specific capabilities are key weaknesses of the model (e.g., discovering that the model's score in the "counting" dimension is significantly lower than in other dimensions). Consequently, they can targetedly augment training data or design dedicated fine-tuning strategies for that capability, rather than blindly expanding overall training data volume or tuning hyperparameters without understanding where the actual bottlenecks lie.



![多维度细粒度子技能测试与整体性聊天指标评估对比图 (Evaluation paradigm differences: Multi-dimensional sub-skills versus overall chat metrics.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig33_fine_vs_holistic_eval.png)

</details>

**14. How to Evaluate the Cross-Lingual Performance Consistency of Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

The core idea for evaluating cross-lingual performance consistency is to construct **parallel evaluation benchmarks**—that is, for the exact same set of images and questions, multiple language versions are created (typically using English as the baseline and expanding it into French, Chinese, Arabic, Hindi, etc., via human translation or high-quality machine translation combined with human verification). This ensures that, aside from the language itself, the image content, the evaluated capability dimensions, and the question difficulty remain strictly identical. Consequently, any performance disparity can be attributed solely to the variable of "language" rather than differences in evaluation content.

On this basis, the model's accuracy and internal consistency across language versions for the **same image and identical semantic question** are compared (e.g., whether the model yields substantively identical answers to the same question in English and Chinese). If performance in certain languages (especially low-resource languages that constitute a small fraction of training data, such as Swahili or Vietnamese) is noticeably and systematically lower than in high-resource languages like English, it indicates a clear cross-lingual capability imbalance. This usually suggests that future training dataset construction should proactively enhance data coverage for these weaker languages (whether by collecting more native data or improving the quality and ratio of machine-translated data) to gradually bridge the cross-lingual capability gap.

</details>

**15. What is POPE? How is It Designed Specifically to Evaluate Object Hallucination in Multimodal Models?**

<details>
<summary>💡 Show Answer</summary>

POPE (Polling-based Object Probing Evaluation) is a benchmark specifically designed to evaluate "object hallucination" (i.e., models describing objects that do not exist in the image) in vision-language models. Its core design principle is to transform open-ended "image description" tasks into a series of easily verifiable binary "Yes/No" judgment questions, thereby obtaining more stable and quantifiable evaluation results than free-form text generation.

**Construction Method**: Given an image and its ground-truth object annotations, POPE constructs a series of triplet questions (Image, Queried Object, Ground-truth Yes/No) of the form "Is there a \{object\} in the image?". Half of the queried objects are ground-truth objects present in the image (ground-truth answer: Yes), while the other half are absent objects (ground-truth answer: No), typically in a 1:1 ratio.

**Three Negative Sampling Strategies**: To comprehensively examine hallucination problems across different difficulty levels, POPE designs three ways to select "absent objects":
- **Random (Random Sampling)**: Randomly selects objects absent from the image from all object categories in the dataset, presenting the lowest difficulty;
- **Popular (High-Frequency Sampling)**: Prioritizes object categories with the highest frequency across the entire dataset (but absent in the current image) to test whether the model erroneously assumes an object is present simply because "it is very common";
- **Adversarial (Adversarial Sampling)**: Prioritizes object categories that frequently co-occur with real objects present in the image (but are absent in the current image). This has the highest difficulty and specifically tests whether the model hallucinated due to statistical co-occurrence patterns among objects (e.g., "seeing a bed usually implies a pillow").
Experiments demonstrate that model performance difficulty increases progressively across the three strategies (Adversarial > Popular > Random), validating the phenomenon that "frequently co-occurring objects in training data are more likely to trigger hallucinations."

**Evaluation Metrics**: Treating questions as a standard binary classification task, POPE uses four metrics—Accuracy, Precision, Recall, and F1 Score—to measure overall performance. It also records the proportion of "Yes" responses from the model to diagnose whether a systematic bias exists where the model tends to answer "Yes" regardless of whether it actually perceives the object.

**Significance and Limitations**: Compared to earlier text-matching-based metrics like CHAIR, POPE yields more stable evaluation results via Yes/No questions (showing better robustness to prompt templates) and incurs lower construction and evaluation costs. However, its limitation lies in only covering the basic hallucination type of "object existence," failing to assess finer-grained attribute hallucinations (color, count) or relationship hallucinations (spatial/interactive relationships). Subsequent benchmarks like MMHal-Bench targetedly expanded into these fine-grained dimensions.

</details>

