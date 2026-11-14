# 12.6 Multimodal Training Data — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 12: Multimodal Large Models · 14 questions

**1. How is Multimodal Training Data Constructed? Sources, Ratios, and Quality Control**

<details>
<summary>💡 Show Answer</summary>

**Large-Scale Pre-training Data:**
- Web-crawling image-text pairs from the internet (e.g., LAION-5B, CC3M, CC12M, YFCC100M);
- Using CLIP similarity to filter out low-quality image-text pairs; performing deduplication and safety filtering (NSFW detection) on images;
- Supplementing textual information using tools such as OCR and ASR.

**High-Quality Instruction Data:**
- Human annotation: High cost but good quality;
- Model generation (Self-Instruct / GPT-4 generation): Inputting metadata such as image bounding boxes and captions into GPT-4 to generate multi-turn dialogue samples (the LLaVA approach);
- Academic dataset conversion: Rewriting supervised datasets such as VQA, Captioning, NLI, and OCR into instruction formats.

**Data Sources and Ratios:** Pre-training data usually mixes internet image-text pairs (large scale but noisy), high-quality human-annotated data (small scale but high quality), synthetic data (model-generated descriptions/dialogues), pure text data (maintaining language capabilities), and specific task data (OCR, chart understanding, video captioning). Data ratios directly affect model performance across language capabilities vs. visual understanding capabilities vs. specific task capabilities; unbalanced ratios may lead to catastrophic forgetting or underfitting of certain capabilities.

**Data Quality Control:** Diversity (covering different task types, image types, and language styles); Filtering (removing truncated, misaligned, and low-resolution images); Balancing (preventing excessive single-task data from causing forgetting of other capabilities).

</details>

**2. What is the "Noise" Problem in Image-Text Alignment Data? How to Address It?**

<details>
<summary>💡 Show Answer</summary>

Image-text pair data crawled on a large scale from the internet (such as alt text accompanying images on web pages or surrounding descriptive text) universally suffers from the "noise" problem where text descriptions do not fully match the actual content of images. Specific manifestations include: **inaccurate** descriptions (content mentioned in text deviates from what is actually presented in the image), **incomplete** descriptions (describing only part of the image content or being overly generic and lacking details), or even descriptions **completely unrelated** to image content (e.g., merely text occurring adjacent in web layout with zero semantic connection to the image itself). If such noisy data is directly used for training without processing, it will mislead the model into learning incorrect image-text correspondences, serving as one of the data-level root causes leading to model hallucinations and related issues.

Common methods to address this issue include:

- **Relevance Score Filtering**: Using pre-trained image-text alignment models (such as CLIP) to compute image-text similarity scores for each pair, filtering out samples with excessively low scores (which are very likely misaligned);
- **Recaptioning (Regenerating Descriptions)**: Using specially trained high-quality caption generation models to directly regenerate a more accurate and detailed descriptive text for the image, replacing the original web-crawled description that may contain noise with this "regenerated" high-quality description (this is also one of the core ideas of the CapFilt mechanism proposed by BLIP);
- **Constructing High-Quality Human-Annotated Data**: For the most critical training stages (such as the instruction tuning stage), using data that is limited in scale but rigorously audited by humans, ensuring that this portion of data with the greatest impact on shaping model capabilities is free of noise issues.



![网页爬取的图文对数据集清洗与语义对齐过滤流程图 (Data cleaning and semantic alignment pipeline for web-scraped image-text datasets.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig20_noise_filtering.png)

</details>

**3. What is the "Data Contamination" Problem in Multimodal Model Evaluation? What is Unique About It in Multimodal Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Data contamination refers to the situation where samples from an evaluation benchmark used to assess model performance (or samples highly similar to them) have actually already appeared in the model's pre-training or fine-tuning data. As a result, during "evaluation," the model is actually "recalling" content seen during training rather than genuinely demonstrating its capability to generalize to unseen data. This leads to artificially inflated evaluation scores that fail to truthfully reflect the model's actual level.

In multimodal scenarios, data contamination exhibits certain unique characteristics and stronger stealthiness: because almost all current mainstream multimodal large models rely heavily on massive web-crawled image-text pairs for training, and images used by many well-known evaluation datasets (such as classic VQA and Image Captioning datasets) may have long circulated widely on the internet, these evaluation images themselves **are highly likely to have already been indexed by crawlers and used in the model's pre-training**—even if the specific questions and ground-truth answer texts in the evaluation benchmark never directly appeared in training data. Simply because the model has "seen this image" (and potentially learned certain descriptive texts related to it during pre-training), it may achieve abnormally good scores during evaluation that do not represent true generalization ability. Compared to pure text scenarios where contamination detection is relatively easy via text string matching (comparing whether large blocks of repeated text exist between training and evaluation texts), this **"image-level"** contamination is far more covert and difficult to detect. This is because images cannot be easily compared via simple string matching; specialized image similarity detection methods such as perceptual hashing are often required to reliably screen out such contamination.

</details>

**4. How to Handle "Harmful / Unsafe" Content in Training Data?**

<details>
<summary>💡 Show Answer</summary>

Safety handling of multimodal training data needs to cover multiple stages of the data pipeline:

- **Safety Filtering During Data Collection Stage**: Using specially trained NSFW (Not Safe For Work) detection models to automatically scan massive images, identifying and removing non-compliant image content such as pornography, violence, and gore; simultaneously conducting sensitive word filtering and harmful content detection on accompanying text descriptions to prevent bad text content from entering training data;
- **Refusal Capability Training During Instruction Tuning Stage**: Merely filtering out harmful content in training data is insufficient; it is also necessary to **specifically construct "refusal-type" samples** within instruction tuning data—such as constructing scenarios where users upload inappropriate images and make malicious requests (e.g., asking to identify private personal information or assist in generating harmful content), paired with appropriate and polite refusal responses that the model should provide, teaching the model proper ways to respond when facing such requests via supervised learning;
- **Red Teaming After Training Completion**: After model training is completed, **Red Teaming** is also required—actively organizing personnel to try constructing various edge cases and "jailbreak" techniques (e.g., embedding misleading text inside images, carefully designed adversarial prompts, etc.) to test whether vulnerabilities exist in the model's safety boundaries. Based on issues discovered during testing, data filtering rules and refusal strategies are continuously iterated and improved, forming a continuous iterative loop of "testing to find issues — improving data/training strategies — re-testing for verification."

</details>

**5. What is the "Curriculum Learning" Idea in Data Ratios?**

<details>
<summary>💡 Show Answer</summary>

The core concept of curriculum learning is borrowed from the human learning process—usually learning step-by-step in a reasonable order **from easy to difficult, and from general foundations to specialized subdivisions**, rather than directly facing the most complex and difficult content from the very beginning. Applying this idea to the organization of training data for multimodal large models typically manifests as:

1. **Step 1: Large-Scale Foundational Alignment**—First use massive but relatively coarse, broad-coverage web image-text pair data, enabling the model to establish basic, coarse-grained associations between visual concepts and language (equivalent to "first recognizing common things in the world");
2. **Step 2: High-Quality Capability Enhancement**—Gradually introduce smaller-scale but higher-quality instruction data with higher task complexity (e.g., tasks requiring multi-turn dialogues or complex reasoning), allowing the model to further enhance its ability to solve complex problems on top of existing foundations;
3. **Step 3: Targeted Capability Reinforcement**—Finally, targeting specific capabilities where the model still exhibits noticeable shortcomings after the first two steps of training (such as fine-grained capabilities like precise counting or complex chart understanding), supplement small amounts of highly focused specialized training data to perform final capability gap filling.

Compared to mixing data of all difficulty levels together right from the start, this progressive, coarse-to-fine organization method is usually proven in practice to make the training process more stable. The model can also more easily build a step-by-step capability system from simple to complex, without easily getting "discouraged" by overly difficult data in early training (which would cause unstable training or convergence difficulties).

</details>

**6. How to Build Multimodal Large Language Models When Image-Text Pair Data in Vertical Domains (e.g., Medicine) Is Extremely Limited?**

<details>
<summary>💡 Show Answer</summary>

In vertical domains such as medical imaging, remote sensing, and industrial quality inspection, the cost of collecting and annotating domain-specific data is extremely high (often requiring qualified doctors or domain experts to participate in annotation). Consequently, the scale of high-quality image-text pair data is usually much smaller than in general domain scenarios, making it difficult to train a domain-specific multimodal large model from scratch with sufficient data support. Common strategies include:

1. **Domain Vision Encoder Pre-training**: Utilizing a large volume of relatively easily obtainable **unlabeled** domain images (e.g., massive amounts of unlabeled medical imaging scans), a domain-specific vision encoder is pre-trained using self-supervised learning methods (such as the masked reconstruction objective of MAE). This enables it to learn the unique visual patterns and feature distributions of domain images without relying on expensive human annotations;
2. **Transfer Learning from General VLMs**: Building upon a powerful VLM already pre-trained on general-domain data, targeted fine-tuning is performed using only a limited scale of high-quality domain image-text data (usually in conjunction with parameter-efficient methods such as LoRA). This allows the model to acquire domain-specific knowledge and expressions while retaining its general vision-language understanding capabilities;
3. **Synthetic Data Expansion**: Leveraging existing powerful multimodal models (such as GPT-4V) to generate preliminary descriptions or Q&A pair drafts for domain images, which are then reviewed and corrected by domain experts. Compared with pure manual annotation from scratch, this "model drafting + expert review" paradigm significantly reduces annotation costs while guaranteeing the professional accuracy of the final data;
4. **Multi-stage Hierarchical Training**: Completing foundational vision-language alignment training on general image-text data first (learning general image-text association abilities), and then performing instruction tuning using a relatively small amount of high-quality domain data. Through this phased approach of "laying a foundation with general capabilities and fine-tuning with domain data," domain knowledge can be effectively injected into the model even under limited data conditions.



![垂直领域图文数据极度匮乏下的领域适应策略结构图 (Domain adaptation strategies under severe vertical data constraints.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig21_domain_adaptation.png)

</details>

**7. How to Construct Multimodal Preference Data (for RLHF/DPO)?**

<details>
<summary>💡 Show Answer</summary>

The typical workflow for constructing multimodal preference data is as follows: First, given the same multimodal input (image + question/instruction), the model to be optimized (or multiple model versions with slightly different parameter configurations) is used to generate multiple candidate responses; subsequently, human annotators or a more capable model (such as GPT-4V) serve as a "judge" to perform pairwise comparisons between these candidate responses, evaluating which response is **more accurate** (the content aligns better with the actual image scenario), **less hallucinatory** (contains no fabricated content absent from the image), and **better adheres to instructions** (better completes the user's specific instruction), and labeling the comparison result as a "chosen--rejected" preference pair.

There are several key points during the annotation process: Annotators must **carefully verify the actual content of the image itself**, rather than judging quality solely based on the fluency or linguistic quality of the response text, because a response with ornate language but factual inaccuracies should be explicitly judged as inferior to a plain but accurate response; moreover, beyond relying on human/model pairwise evaluations, there is a cleverer and lower-cost automated negative sample construction method---prompting the model to generate responses under two settings: "providing the correct image" and "providing an irrelevant image (or providing no image at all)". The response corresponding to the former can usually serve as a "chosen" sample, while the latter, lacking proper visual grounding, often relies on language priors to generate lower-quality or hallucinatory responses that can directly serve as "rejected" samples, thereby automatically constructing a large volume of preference comparison data without requiring extra human annotation.

</details>

**8. What Role Does Synthetic Data Play in Multimodal Training? What Are the Generation Methods?**

<details>
<summary>💡 Show Answer</summary>

Synthetic data plays an increasingly vital role in multimodal LLM training, serving primarily to alleviate the practical bottleneck of limited scale and high costs associated with high-quality human-annotated data. Common generation methods include:

- **Generating Descriptions/Dialogues via Powerful Models**: Utilizing highly capable closed-source multimodal models (such as GPT-4V) to generate detailed descriptive text, multi-turn dialogue samples, or even chain-of-thought data containing intermediate reasoning steps for existing image materials. This data "taught" by strong models can be used to train relatively weaker open-source models (an idea derived from knowledge distillation);
- **Programmatically Generating Data with Precise Annotations**: For certain specific capabilities (such as chart understanding and precise counting), image content can be automatically generated programmatically while simultaneously recording **absolutely accurate** annotation information---for instance, using code to automatically draw bar charts while recording the true value corresponding to each bar. Annotations generated in this manner are perfectly accurate without any annotation error, which is particularly valuable for training models to precisely master structured visual information understanding;
- **Reverse-Constructing Image-Text Pairs via Text-to-Image Models**: Leveraging text-to-image models to generate corresponding images based on pre-defined text descriptions. Since the text descriptions are known and fully controllable, this method enables the fast and low-cost construction of a massive number of training pairs with perfectly matched image-text semantics.
The core advantages of synthetic data lie in its **low cost, fast generation speed, and high controllable accuracy of annotations** (especially in programmatically generated scenarios); however, a potential risk to note is that the image styles and linguistic expression styles of synthetic data often differ to some extent from real-world data distributions. If the proportion of synthetic data in the training set is excessively high, the model may "overfit" to the unique stylistic patterns of synthetic data, degrading its performance when dealing with more diverse and "messy" real-world data. Therefore, synthetic data typically needs to be used in conjunction with high-quality real data rather than replacing it entirely.

</details>

**9. How to Deduplicate Large-Scale Web-Scraped Image-Text Data?**

<details>
<summary>💡 Show Answer</summary>

In large-scale web-scraped image-text data, duplicate or highly similar samples (such as the same image republished by different websites, or classic memes/watermarked images appearing repeatedly), if left untreated, will cause the model to "overfit" to these high-frequency samples during training, wasting training resources and potentially distorting the data distribution. Common deduplication methods include:

- **Image Perceptual Hashing (Perceptual Hash)**: Mapping each image to a hash fingerprint that is insensitive to minor pixel perturbations (scaling, slight cropping, compression distortion), and judging whether two images are "perceptually identical" by comparing the Hamming distance of their hash values. This efficiently detects exact duplicates or duplicate images that have undergone simple edits (watermarking, slight cropping);
- **Text MinHash/SimHash**: For the accompanying descriptive text, locality-sensitive hashing (LSH) techniques such as MinHash or SimHash are used to rapidly identify samples with highly similar or duplicate text content, preventing massive amounts of nearly identical descriptions from flooding the training data;
- **Semantic Deduplication Based on CLIP Embeddings**: Encoding all images (or image-text pairs) into CLIP feature vectors, and identifying sample clusters with very close semantic embedding distances through clustering or nearest neighbor search, retaining only representative samples within each cluster. This approach goes a step further than pixel-level or text-level deduplication, enabling the discovery of semantic duplicates that are "similar in content but different in visual expression" (e.g., photos of the same scene taken from different angles).
The goal of deduplication is not to achieve "zero duplicates," but to control the proportion of high-frequency duplicate samples in the data distribution, preventing them from excessively dominating the direction of training gradients, while also helping to reduce the risk of subsequent "data contamination" on known evaluation benchmark datasets.

</details>

**10. What Is a Data Recipe? How Is It Typically Disclosed in Open-Source Model Technical Reports?**

<details>
<summary>💡 Show Answer</summary>

Data Recipe refers collectively to the overall composition scheme of data used in model training. Specifically, it encompasses the following aspects: **Data Source Proportions** (percentages occupied respectively by web-scraped image-text pairs, human-annotated high-quality data, model-synthesized data, retained pure text data, and task-specific data such as OCR/chart understanding); **Multi-stage Data Switching Strategies** (which data sources are used in the pre-training, continued pre-training, and instruction fine-tuning stages, and whether proportions change across stages); and **Data Cleaning and Filtering Rules** (which deduplication, quality filtering, and safety filtering techniques were adopted).

Technical reports of open-source models (such as papers from the LLaVA, Qwen-VL, and InternVL series) typically disclose, via tables or charts, the approximate total data volume used in different training stages (e.g., "around X billion image-text pairs in the pre-training stage") as well as the relative proportions of each data source, helping the research community understand the overall philosophy of model training. However, due to considerations such as data licensing and commercial competition, the specific raw datasets themselves (especially private annotated data or generation prompt details of synthetic data) are often not fully disclosed. This reflects a reality where "data recipes" generally exhibit lower transparency compared to model architectures.



![综合性视觉语言模型训练数据集混合配比柱状图 (Typical mixture of datasets in a comprehensive VLM training data recipe.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig22_data_recipe.png)

</details>

**11. Where Is Video-Text Pair Data Obtained? What Are the Special Challenges Compared to Image-Text Data?**

<details>
<summary>💡 Show Answer</summary>

Common sources of video-text pair data include: **Web Videos with Subtitles/Voiceovers** (utilizing auto-generated subtitles provided by video platforms or user-uploaded subtitle files as weakly annotated text); **Professionally Annotated Video Captioning Datasets** (such as MSR-VTT and ActivityNet Captions, where human annotators write descriptions for video clips, yielding high annotation quality but limited scale); and **Automatically Generating Video Summaries via Existing Image/Video Captioning Models** (first using existing models to extract keyframes or generate segmented descriptions for videos, and then summarizing them into overall video text descriptions, which is currently an important means of scaling up video-text data).

Compared with image-text data, video-text data faces special challenges that mainly include:

- **High Storage and Processing Costs**: The file size and decoding computational load of videos far exceed those of single images, so constructing large-scale video datasets inherently requires greater investment in storage and compute infrastructure;
- **Granularity Mismatch Between Text Descriptions and Video Content**: A video that may be several minutes long is typically accompanied by only one or a few summarizing overall sentences, whereas the temporal details actually contained within the video (who did what action at what moment) are far richer than these few sentences. This leads to a severe mismatch between the granularity of supervision signals and the information content of video media;
- **Extreme Scarcity of Fine-Grained Temporal Annotations**: Annotations precise to specific time intervals, such as "what specific event occurred from second 3 to second 5," require annotators to watch the video repeatedly and precisely log timestamps. The annotation cost is far higher than that for static image captions, which is the root cause for the relative scarcity of training data in tasks such as temporal grounding.

</details>

**12. How to Construct Multilingual Multimodal Data? How to Prevent Models from Developing Imbalanced Language Capabilities?**

<details>
<summary>💡 Show Answer</summary>

Building multilingual multimodal training data relies primarily on two complementary approaches:

1. **Directly Collecting Native Data**: Collecting image-text pairs and instruction data in local languages directly from native websites, social media, and other channels in non-English regions. This type of data can more genuinely reflect the expression habits and cultural contexts of native speakers, but collection costs are high, and the covered languages and data scales are typically limited;
2. **Machine Translation Expansion**: Batch-translating pre-constructed high-quality English instruction data (which is often larger in scale and higher in maturity) into target languages via machine translation systems. This approach is low-cost and yields fast results, rapidly "padding" data scales for low-resource languages. However, translation quality varies, and deviations easily occur particularly when handling expressions that require visual context for accurate translation (such as certain culturally specific object names). This usually requires supplementation via human sample verification or higher-quality translation models.
To prevent imbalanced capability development across different languages, it is necessary to **actively control the proportion of data for each language** during training (rather than letting the natural distribution of data dominate, as web data is naturally dominated by English). Low-resource languages should be appropriately **oversampled**, meaning that data in these languages is reused more frequently within a training epoch to artificially increase its "exposure." Concurrently, parallel benchmarks covering multiple languages (the same image and question rendered in different languages) must be specially constructed during the evaluation phase to continuously monitor performance gaps across languages. Once a significant capability gap is discovered, targeted adjustments to the training data ratio for that language or the addition of more high-quality data in that language become necessary.

</details>

**13. How to Evaluate the Quality of a Multimodal Training Dataset?**

<details>
<summary>💡 Show Answer</summary>

Evaluating training dataset quality usually requires comprehensive consideration across multiple dimensions rather than relying on a single metric:

- **Image-Text Correlation**: Whether descriptive text accurately corresponds to the actual content of the image, and whether there are samples where text fails to match the image or where descriptions are clearly irrelevant to the image. Automated metrics such as CLIP Score can be leveraged for preliminary coarse filtering of full datasets (samples with excessively low similarity are highly likely to suffer from image-text mismatch), though automated metrics themselves carry potential for misjudgment and usually require supplementary human sample verification;
- **Diversity**: Whether the dataset covers a sufficiently rich spectrum of scene types, object categories, task formats, and linguistic expression styles, avoiding excessive concentration of data distribution on a few common scenarios (such as over-concentrated portraits and landscape photos on the web), which leads to noticeably deficient model performance on long-tail and rare scenarios;
- **Annotation Accuracy**: Whether factual errors or hallucinatory descriptions (fabricating content non-existent in the image) exist, regardless of whether annotations are human-crafted or model-generated. If present in large quantities in training data, such errors will directly "teach" the model to exhibit similar hallucinatory behavior;
- **Balance Between Scale and Quality**: Whether quality control criteria were relaxed simply to pursue "impressive numbers" in data scale---the negative impact of massive amounts of low-quality data sometimes even outweighs the issues caused by insufficient data scale, requiring a careful trade-off validated by actual downstream task performance.
In practice, it is usually necessary to combine large-scale preliminary screening via automated metrics (high efficiency but susceptible to misjudgments) with human sample evaluations (accurate but high-cost and limited in coverage), cross-validating to yield a relatively reliable assessment of dataset quality.

</details>

**14. How to Determine the Optimal Multimodal Training Data Ratio via Ablation Studies?**

<details>
<summary>💡 Show Answer</summary>

Because the impact of data ratios on final model capabilities is difficult to predict precisely in theory, systematic ablation studies are commonly employed in practice to identify superior ratios:

1. **Control Variables**: Fix other factors such as model architecture, training steps, and total data volume, while adjusting only the proportion of a specific data category (e.g., pure text data, OCR data, instruction dialogue data) within the total training data, training multiple model versions for comparison.
2. **Small-Scale Proxy Experiments**: Because full-scale training costs are extremely high, ratio experiments are usually conducted first at smaller model scales or shorter training steps (proxy experiments), assuming that relative trends observed on small scales (e.g., whether increasing a data category's ratio improves a specific capability) can approximately transfer to large-scale training, before performing final verification prior to full-scale training.
3. **Multi-Dimensional Evaluation Monitoring**: Each ratio version must be evaluated on fine-grained evaluation benchmark sets covering different capability dimensions (rather than just a single composite score) to observe how increases or decreases in specific data proportions affect targeted capabilities (such as OCR accuracy, instruction-following ability, and pure text dialogue fluency), preventing capability degradation in other areas due to over-focusing on a single metric.
4. **Iterative Adjustment**: Gradually adjust ratios according to ablation study results (for example, if a low proportion of pure text data leads to degraded language capability, appropriately increase that data category's ratio). Multiple iterations are typically required to converge on a relatively balanced final data recipe.



![视觉微调数据配比变化与模型下游任务表现的消融实验曲线图 (Performance scaling curves relative to varying visual instruction tuning data ratios.)](https://img.pricenode.uk/llm_interview_questions/english/chapter12/fig23_ablation_curves.png)

</details>

