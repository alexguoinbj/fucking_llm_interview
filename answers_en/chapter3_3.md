# 3.3 Data Engineering — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 3: Pre-training, Inference, and Engineering Optimization · 63 questions

**1. Why Is Pre-training Data Quality More Important Than Quantity?**

<details>
<summary>💡 Show Answer</summary>

The view that "quality is more important than quantity" can be understood from the following aspects:

**1. Low-Quality Data May Introduce Noise or Even Harmful Learning Signals**

- If training data contains a large amount of **grammatical errors, logical confusion, factual errors, or spam content (e.g., SEO keyword-stuffed pages, machine-generated low-quality text)**, training the model to "predict the next token" on such data will cause it to **learn and internalize these faulty/low-quality patterns**—for instance, if repeated factual errors exist across a large amount of training data, the model may "memorize" the incorrect fact; if the training data is filled with grammatically incoherent text, the model's generated output may also be contaminated by this "low-quality style".

**2. Diminishing Marginal Returns or Even Harm from Duplicate Data**

- Numerous studies demonstrate that **repeatedly training on the same content strengthens the model's "memorization", but gains in generalization performance saturate quickly and can even turn negative** (e.g., making the model more prone to verbatim recitation of specific training passages, increasing privacy leakage risks, see Question 405; or over-reinforcing certain repeated patterns, hindering output diversity);
- Under a fixed computational budget (total training tokens), **"filling" training steps with large amounts of duplicate, low-quality data effectively "wastes" valuable compute**—resources that could otherwise yield greater capability gains if spent on high-quality, diverse data.

**3. High-Quality Data Has a More Direct Impact on Model Capabilities (Especially Reasoning and Factual Accuracy)**

- Empirical studies (e.g., technical reports of various LLMs) reveal that **carefully curated high-quality data subsets (e.g., textbooks, academic papers, high-quality code, filtered web pages) significantly boost performance on knowledge-intensive and reasoning tasks**, an improvement that **cannot be compensated for simply by "increasing the quantity of low-quality data"**;
- Some studies even show that **models trained on smaller but rigorously quality-controlled datasets can outperform models trained on larger datasets of uneven quality in certain capability dimensions** (e.g., practices in Microsoft's Phi series models highlight the value of "textbook-quality" data).

**4. "Quantity" Itself Is Meaningful Only When "Quality" Is Guaranteed as a Premise**

- The Scaling Law (see Question 483) describes how "model performance scales with parameter count, dataset size, and compute budget", but this relationship **implicitly assumes that additional data carries non-trivial "information gain"**. If added data consists largely of duplicate/low-quality content, its "effective contribution" to performance falls far short of the "ideal case" suggested by Scaling Laws. That is, **the "marginal information gain" of low-quality data is very low, and growth in "quantity" cannot deliver performance gains equivalent to an equal growth in high-quality data**.

**Summary**: The core logic behind "quality is more important than quantity" in pre-training data is that **low-quality/duplicate data contributes little to capability improvements (low marginal return) while introducing noise, error patterns, and privacy/safety risks. Given finite compute resources (training tokens), filling the training budget with high-quality data enhances final capabilities far more effectively than making up numbers with low-quality data**—which explains why data cleaning, deduplication, and quality filtering receive increasingly heavy investment in modern LLM pre-training pipelines.

</details>

**2. What Are the Key Steps in Data Cleaning for Large Language Models?**

<details>
<summary>💡 Show Answer</summary>

![Pre-training Data Engineering and Cleaning pipeline.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig11_data_cleaning_pipeline.png)




The typical data cleaning pipeline for LLM pre-training comprises the following key steps:

**1. Format Extraction and Standardization**

- **Extract plain text content** from raw scraped data (e.g., web HTML, PDFs, various document formats), stripping away HTML tags, ads, navigation bars, footers, and other non-body content;
- Standardize encoding formats (e.g., UTF-8) and handle special/garbled characters.

**2. Language Identification and Filtering**

- Use language detection tools to identify the language of each document and **filter out unwanted languages** based on pre-training language distribution targets, or categorize by language for proportional mixing later (see Question 465).

**3. Quality Filtering**

- **Rule-based filtering**: Heuristic rules based on parameters such as document length (too short may lack information), proportion of special characters (too high suggests garbled/spam text), average word length, and punctuation distribution;
- **Classifier-based filtering**: Train a lightweight quality classifier (e.g., trained with high-quality sources like Wikipedia as positive examples and random web pages as negative examples) to score documents and filter out low-scoring ones;
- **Perplexity filtering**: Calculate document perplexity using an existing language model and filter out documents with abnormally high perplexity (indicative of garbled or incoherent text).

**4. Deduplication**

- Includes exact deduplication (identical documents/paragraphs) and near-deduplication (highly similar but non-identical content); see detailed discussions in Questions 467 and 468.

**5. Harmful Content Filtering**

- Filter documents containing illegal or non-compliant content such as violence, pornography, and hate speech (typically combining keyword rules and classifiers);
- Filter content containing significant **personally identifiable information (PII)** or perform anonymization (see Question 405).

**6. Test Set Contamination Detection and Removal**

- Detect whether pre-training corpora contain questions or answers from common evaluation benchmarks to prevent "data leakage" from distorting evaluation results (see Questions 478 and 479).

**7. Data Format Conversion and Chunking**

- Convert cleaned text into formats required for training (e.g., tokenized binary formats) and perform **document chunking/packing** based on the model's maximum sequence length (see Question 469).

**Summary**: The key steps of data cleaning follow the sequence: "**Format extraction and standardization $\rightarrow$ Language identification and filtering $\rightarrow$ Quality filtering (rules + classifiers + perplexity) $\rightarrow$ Deduplication (exact + near-deduplication) $\rightarrow$ Harmful content/privacy filtering $\rightarrow$ Test set contamination detection $\rightarrow$ Format conversion and chunking**". This complex data processing pipeline combines multiple technical approaches (rules, classifiers, statistical methods) to filter out high-quality, safe, and uncontaminated training corpora from massive, noisy raw data.

</details>

**3. How Would You Remove Low-Quality, Duplicate, and Junk Data from the Corpus?**

<details>
<summary>💡 Show Answer</summary>

Specific removal methods for low-quality, duplicate, and junk data:

**1. Low-Quality Data**

- **Rule-based heuristic filtering**:
- Document length filtering (too short may lack informative content, while too long may indicate abnormal concatenated text);
- Abnormal symbol/digit/letter ratios (e.g., an excessively high proportion of a certain character type may indicate garbled text or tabular data miscaptured as text);
- Excessively high word repetition rate (e.g., a word repeatedly appearing in the document, which may indicate SEO keyword-stuffed pages);
- Lack of punctuation or abnormal sentence structures (may be non-natural language content scraped by machines, such as code snippets misclassified as text, or log files).

- **Classifier-based filtering**: Train a binary classification model using **high-quality sources (such as Wikipedia, curated books, and academic papers) as positive samples and ordinary web crawl data as negative samples (or adopting other comparative designs)** to score all documents and filter out low-scoring ones; such classifiers can range from simple n-gram features + logistic regression to classifiers fine-tuned on small pre-trained models.

- **Perplexity filtering**: Use a small language model trained on a high-quality corpus to compute the perplexity of the document to be evaluated. **Documents with excessively high perplexity** (which the model finds "difficult to predict", often corresponding to syntactically chaotic or semantically incoherent text) are filtered out; however, threshold setting requires caution—overly aggressive perplexity filtering may mistakenly delete "stylistically unique yet legitimate" content (such as poetry or texts dense with specialized terminology).

**2. Duplicate Data**

- Refer to Questions 467 and 468; combine exact deduplication (e.g., exact match detection based on document hash values) with near-deduplication (e.g., similar document detection based on MinHash + LSH);
- Deduplication should be performed not only at the "document level", but also at the **"paragraph level" or "sentence level"** (the same piece of advertisement or copyright statement may appear in the headers/footers of numerous different documents; such "templated" duplicate fragments need to be identified and removed).

**3. Junk/Spam Data**

- **Explicit spam pattern recognition**: Common patterns such as pure navigation link lists, copyright statement templates, cookie notification text, "click here" button text, and advertisement copy can be identified and removed using regular expressions or specially trained classifiers;
- **Domain/Source blacklisting**: For known low-quality/spam source domains (such as known content farms and SEO spam sites), domain-level filtering can be applied as early as the data collection stage;
- **Template duplicate content identification**: Page structures of many websites are highly templated (navigation bars and sidebar text repeating across all pages of the same website); by **statistically identifying text fragments with unnaturally high co-occurrence frequencies across numerous pages under the same domain**, such "template noise" can be detected and removed.

**4. Integrated Pipeline**

- In practice, this is usually a **cascaded pipeline of multiple methods**—first using low-cost rule-based filtering to quickly eliminate obvious junk content, then using higher-cost classifiers/perplexity models for finer quality scoring and filtering of the remaining data, and finally performing deduplication;
- The threshold setting for each filtering step needs to be verified through **sampled manual audits**—overly lenient thresholds may retain substantial low-quality content, while overly strict thresholds may accidentally delete valuable content (especially "niche yet high-quality" content whose statistical features differ from "mainstream high-quality content", potentially leading to misjudgment by classifiers trained on "mainstream standards").

**Summary**: The methodology for removing low-quality, duplicate, and junk data is a **cascaded pipeline consisting of rule-based heuristic filtering (length, character ratio, repetition rate) + classifier-based quality scoring + perplexity filtering to remove low-quality content; exact deduplication (document-level hashing) + near-deduplication (MinHash/LSH) to remove duplicates; and specific pattern recognition for spam content (template text, domain blacklists) to remove junk data**. It is a multi-layer filtering pipeline that requires combining automated methods with manual sampled audits to strike a balance between filtering strictness and avoiding the accidental deletion of valuable content.

</details>

**4. How Should Multilingual Data Be Handled in Pre-training?**

<details>
<summary>💡 Show Answer</summary>

Key considerations for handling multilingual data during pre-training:

**1. Designing Language Proportions**

- Based on the model's target application scenarios (e.g., primarily targeting Chinese users but also requiring a certain level of English/multilingual capability), determine the **proportions** of different languages in the final training dataset;
- Because the "natural abundance" of different languages in web corpora varies dramatically (English corpora are far more abundant than other languages), **mixing directly according to "natural proportions" may cause low-resource language data to have an excessively low fraction**, resulting in weak model performance on these languages. Typically, low-resource languages are **upsampled (increasing their ratio in the final training data beyond their natural proportion)**, while high-resource languages (such as English) undergo a degree of downsampling to balance capability across languages.

**2. Multilingual Coverage of the Tokenizer**

- The vocabulary needs to **efficiently cover characters/words of all target languages**, preventing text of certain languages from being fragmented into excessively numerous, inefficient subword/byte-level tokens (which causes the same semantic content in that language to require more tokens, indirectly reducing the "effective training data volume" for that language and increasing inference costs);
- When training the tokenizer, the **language proportions of the training corpus itself** affect how well the final vocabulary covers each language—if a language has an overly low ratio in the tokenizer training data, its common words/characters may fail to obtain dedicated tokens, leading to inefficient tokenization (refer to the discussion of Chinese scenarios in Question 357).

**3. Cross-lingual Alignment and Knowledge Transfer**

- Including **multilingual parallel corpora (such as translation pairs)** helps the model establish semantic alignment across different languages and facilitates cross-lingual knowledge transfer—for instance, knowledge acquired from English corpora can, through cross-lingual alignment, "transfer" to some extent to Q&A in other languages.

**4. Language Identification and Quality Control**

- Accurate language identification tools are required to classify mixed corpora by language and perform quality filtering separately (since "spam content" patterns may differ across languages, unified filtering rules may not apply universally);
- For low-resource languages, because "high-quality sources" themselves are relatively scarce, quality filtering standards/thresholds may need targeted adjustments (overly strict standards might further shrink the available data for those languages).

**5. Mixing Strategies in Training Stages**

- Multilingual data can be mixed **throughout the entire pre-training process** (exposing the model to multiple languages from the start), or a **staged strategy** can be adopted (e.g., focusing primarily on high-resource languages during the main pre-training phase, and increasing the proportion of low-resource languages in later/annealing phases for reinforcement);
- The **developmental curves of capabilities in each language** need to be monitored during training (via language-specific evaluation benchmarks) to prevent capability in certain languages from being "overwhelmed" by training signals from dominant languages.

**6. Evaluation System**

- An **evaluation suite covering all target languages** needs to be constructed to separately assess model performance in each language, rather than relying solely on a single (typically English-dominated) evaluation benchmark to represent overall multilingual capabilities.

**Summary**: The core of multilingual data processing lies in **designing language proportions based on target application scenarios (upsampling low-resource languages), ensuring efficient tokenizer coverage for all languages, utilizing parallel corpora to foster cross-lingual knowledge transfer, applying language-specific quality control, and monitoring the progression of language capabilities through language-specific evaluation systems**. In essence, it is a fine-grained balancing act between overall model capabilities and specific language performance achieved through data mixture design and tokenizer construction.

</details>

**5. How to Design Data Mixture Proportions for Corpora from Different Sources?**

<details>
<summary>💡 Show Answer</summary>

![Multi-domain data mixing ratios for balanced LLM pre-training.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig14_data_mixing_ratios.png)




Designing the data mixture ratio is one of the core problems in pre-training. Common design strategies include:

**1. Reference to Empirical Experience and Published Work**

- Refer to data mixtures disclosed in existing technical reports of well-performing large language models (such as the proportions of web data, code, books, academic papers, dialogue data, etc.) as a baseline starting point.

**2. Small-Scale Proxy Experiments**

- Test different data mixture ratios under **smaller model scales and fewer training steps**, evaluate the pros and cons of different mixtures based on performance on downstream evaluation tasks, and then transfer the "mixture that performed well in small-scale experiments" to large-scale training (assuming that findings from small-scale experiments can be extrapolated to large-scale training to some extent; while this is an approximation, costs are significantly reduced compared to experimenting directly at scale).

**3. Trade-off Between Data Quality and Quantity**

- Certain high-quality yet **quantity-limited** data sources (such as Wikipedia, textbooks, and curated books) are typically **upsampled** (with a higher proportion in the final mixture than their natural fraction in the raw data pool), because such data yields a higher "marginal contribution" to model capabilities (refer to Question 462); training over multiple epochs on such data can be more valuable than seeing it only once;
- Web data of huge volume but uneven quality (such as Common Crawl), after undergoing strict quality filtering, is incorporated at a controlled ratio (typically still forming the majority of total data).

**4. Targeted Ratios for Specific Capabilities**

- If the goal is for the model to possess specific capabilities (such as coding, mathematical reasoning, or multilingual skills), the **proportion of corresponding data types (codebases, mathematical corpora, multilingual corpora) is intentionally increased**. Such "capability-oriented" mixture designs require validation and tuning against targeted downstream evaluations (refer to Questions 473 and 482 on the impact of code data on reasoning capabilities).

**5. Dynamic/Staged Mixture Adjustments**

- Some works employ a **staged data mixture**—using one mixture during the main pre-training phase, and switching to another mixture biased toward "high-quality/capability-specific data" during the final phase (annealing phase), capitalizing on the model's heightened sensitivity to recently seen data during learning rate decay to perform focused reinforcement.

**6. Automated/Learning-based Mixture Optimization**

- Some recent studies explore **using algorithms (e.g., based on reinforcement learning or modeling the impact of data mixtures on loss/downstream performance) to automatically search and optimize data mixture ratios**, rather than relying solely on human experience. However, such methods carry high computational costs and complexity and have not yet become a universal standard industry practice.

**7. Continuous Monitoring and Iteration**

- Throughout large-scale training (or via milestone checkpoint evaluations), continuously monitor model performance across various capability dimensions (language understanding, coding, mathematics, multilingualism, knowledge Q&A, etc.). If capability growth in certain dimensions lags behind, the data mixture can be adjusted in subsequent training phases (if training is staged or chunked) to reinforce weak spots.

**Summary**: Designing data mixture proportions is an integrative process involving **using empirical experience as a baseline + verifying relative advantages of different mixtures via small-scale proxy experiments + upsampling high-quality scarce data while weighting capability-specific data + employing staged mixture strategies when necessary (such as adjustments during annealing) + continuously monitoring capability dimensions and iterating**. It is an engineering discipline blending art and science that relies on experience, experimental validation, and iterative refinement, without any single "one-size-fits-all" fixed formula.

</details>

**6. What Are the Common Methods for Pre-training Data Deduplication?**

<details>
<summary>💡 Show Answer</summary>

Common methods for pre-training data deduplication include:

**1. Exact Deduplication**

- **Hash-based exact match detection**: Compute a hash value (such as MD5 or SHA256) for each document (or paragraph/sentence), treat documents with identical hash values as duplicates, and retain only one copy;
- This method is fast and simple to implement, but **it can only detect "verbatim identical" content**, failing to identify duplicates that are "similar in content with minor differences" (such as differing punctuation, spaces, or subtle rephrasings).

**2. Near/Fuzzy Deduplication**

- **MinHash + LSH (Locality-Sensitive Hashing)**: This is the most widely used method for near-deduplication of large-scale corpora (refer to Question 468 for details)—represent documents as sets of n-grams, compute document "signatures" via MinHash such that signature similarity approximates the Jaccard similarity of the original sets, and then use LSH to bucket similar documents together. Document pairs only need to be compared within the same bucket, drastically reducing the computational complexity of pairwise comparisons (avoiding $O(N^2)$ brute-force comparisons).

**3. Substring/Paragraph-Level Deduplication**

- In addition to document-level deduplication, duplicate content can also be detected at a **finer granularity (paragraphs, sentences)**—for example, headers, footers, copyright notices, and advertisement template text from the same website may recur across many different "documents", even though those documents are not identical overall (meaning document-level exact or fuzzy deduplication might miss such "local repetition"). By constructing **n-gram frequency statistics**, continuous text fragments that appear with abnormally high frequencies across numerous documents can be identified and removed as "template noise".

**4. Suffix Array-Based Exact Substring Deduplication**

- Some works (such as deduplication pipelines for large-scale corpora) utilize data structures like **Suffix Arrays** to efficiently locate **all substrings that are "sufficiently long and recur multiple times"** across the entire corpus (viewed as one massive string). This method achieves "exact" substring-level deduplication (whereas MinHash is approximate), though computation and storage overhead remain substantial for ultra-large corpora, requiring efficient algorithms and engineering implementations.

**5. Embedding-Based Semantic Deduplication**

- Compute semantic embedding vectors for documents (e.g., using Sentence Transformers) and detect documents with "highly similar semantics but different wording" (such as different re-posts or rewrites of the same news event) via vector similarity (e.g., cosine similarity). This method carries high computational cost (requiring model inference on every document) and is typically used for **fine-grained secondary filtering on data already passed through initial deduplication (hash/MinHash)**, or in specific scenarios with relatively smaller data volumes.

**Summary**: Commonly used methods include **hash-based exact deduplication (document-level exact duplicates), MinHash + LSH near-deduplication (efficient detection of similar documents in large-scale corpora), n-gram frequency-based template/local repetition removal, Suffix Array-based exact substring deduplication, and embedding-based semantic deduplication**. In practice, a combination of multiple methods is typically employed, starting with low-computational-cost methods capable of handling ultra-large datasets (hash, MinHash) for the first round of coarse filtering, followed by higher-cost yet finer methods (semantic deduplication) for supplementary processing.

</details>

**7. What Is the Difference Between Exact Deduplication and Near-Deduplication? What Are Their Respective Application Scenarios?**

<details>
<summary>💡 Show Answer</summary>

**Exact Deduplication**

- **Definition**: Only removes content that is **verbatim character-for-character identical** (typically implemented via hash value comparisons);
- **Characteristics**:
- Fast computation speed and simple implementation (low complexity for hash calculation and comparison);
- **Limited recall**: Cannot identify duplicate content that is "essentially identical in meaning but contains minor differences", such as:
- The same article re-posted on different websites, but with differing ads inserted, formatting variations, or minor editorial changes;
- Different "versions" of the same document (e.g., web snapshots before and after revisions);
- Templated content (e.g., press release templates) filled with slightly different specific details in different places.
- For such "near-identical but not verbatim equal" content, exact deduplication treats them as "different documents" and **retains all of them**, leaving substantial substantive duplicate information in the corpus.

**Near/Fuzzy Deduplication**

- **Definition**: Identifies and removes documents with **highly similar content (without requiring verbatim identity)**, typically based on a certain "similarity metric" (such as Jaccard similarity, edit distance, or semantic similarity) and a threshold to determine whether documents are "considered duplicates";
- **Characteristics**:
- Capable of capturing "substantive duplicates" missed by exact deduplication (re-posts, rewrites, templated content, etc.), resulting in a more significant boost to the corpus's "information density";
- **Higher computational complexity**: Computing pairwise similarity for all documents has a complexity of $O(N^2)$, which is intractable for corpora containing billions of documents; thus, approximate algorithms such as **LSH** are required to reduce the complexity to near-linear ($O(N)$);
- **Trade-off between false positives and false negatives**: Setting the similarity threshold too loosely may misjudge documents that are "thematically similar but substantively different" as duplicates (false positives, mistakenly discarding valuable content); setting it too strictly may miss genuinely duplicate content (false negatives). This trade-off must be adjusted according to specific corpus characteristics and downstream impacts.

**Applicable Scenarios**:

- **Exact Deduplication**:
- Suitable for removing **completely duplicate crawl records** (such as the same URL crawled multiple times, or identical content from mirror sites);
- Serves as the **first and lowest-cost filtering stage** of the overall deduplication pipeline to quickly remove "obvious" exact duplicates, reducing the data volume for subsequent near-deduplication.

- **Near-Deduplication**:
- Suitable for addressing the **widespread phenomena of re-posting, rewriting, and templating in web corpora**—which constitute the primary form of repetition in large-scale web data (such as Common Crawl), against which exact deduplication is largely ineffective;
- Acts as the **core component** of the pre-training data deduplication pipeline; multiple studies (such as data processing efforts for GPT-3, LLaMA, etc.) emphasize the importance of near-deduplication in improving training outcomes;
- In scenarios where data scale has not reached "ultra-large scale" (e.g., millions to tens of millions of documents) and higher deduplication precision is required, **embedding-based semantic deduplication** can also be considered (which carries higher computational cost but can identify content that is "phrased completely differently but semantically identical", serving as a "more advanced" form of near-deduplication).

**Summary**: Exact deduplication handles "verbatim identical" duplicates with low computational cost but limited recall. Near-deduplication handles "highly similar but non-identical" duplicates (the more prevalent and impactful form of repetition in web corpora), relying on techniques like LSH to manage computational complexity, making it the core component of deduplication pipelines. The two are typically **used in a cascaded fashion**—first using exact deduplication to rapidly prune "obvious verbatim duplicates", followed by near-deduplication to address "hidden similar duplicates", jointly boosting the "information density" and overall quality of the pre-training corpus.

</details>

**8. How Does Document Chunking Strategy Impact Training Effectiveness and Efficiency?**

<details>
<summary>💡 Show Answer</summary>

Document chunking (how to organize raw documents into fixed-length input sequences required for model training) strategies affect training in multiple ways:

**1. Training Efficiency (Computational Resource Utilization)**

- Model training typically uses a **fixed sequence length (e.g., 4096/8192 tokens)** as input, whereas the lengths of raw documents vary drastically (some are very short, while others are very long);
- If we simply **pad each document individually to a fixed length**, a large number of short documents will generate many padding tokens (these tokens do not participate in effective language modeling loss calculation but still consume computational resources), **significantly reducing training efficiency (low proportion of effective tokens)**;
- A common and more efficient approach is to **concatenate multiple (short) documents into a fixed-length sequence** (separated by special delimiter tokens such as EOS) to minimize padding and increase the proportion of "effective tokens" in each training sample.

**2. Attention Issues Across Document Boundaries**

- When multiple documents are concatenated into the same sequence, **the self-attention mechanism by default allows tokens in subsequent documents to "see" content from preceding documents** (under causal masking, later tokens can attend to all preceding tokens, regardless of whether they belong to the "same document")—this may introduce **spurious, non-existent cross-document dependencies** (the model might learn that "the end of Document A is somehow correlated with the beginning of Document B", when this correlation is an artificial artifact of concatenation rather than a genuine semantic relationship);
- Solution: Use a **document-level attention mask** to ensure that in the concatenated sequence, the attention calculation for each token **can only attend to tokens belonging to the "same raw document"**, blocking cross-document attention—however, this approach **increases computational complexity** (standard causal mask implementation is simple and efficient, whereas document-level masks require extra mask construction and potential computational overhead; especially when using highly optimized kernels like FlashAttention, whether flexible custom masks are supported affects implementation efficiency);
- Another simplified approach is to **not use document-level masks and accept this "slight cross-document information leakage"** (treating its impact as noise-level and negligible in large-scale training)—different training frameworks/teams handle this issue differently, representing a trade-off between efficiency and "theoretical purity".

**3. Impact of Long Document Splitting on Long-Range Dependency Learning**

- If a **long document is split into multiple independent sequences** (because it exceeds the model's maximum sequence length), the "long-range dependencies" originally existing within the document (such as a concept mentioned at the beginning of an article being referenced at the end) will be **severed across different training samples**, preventing the model from learning such dependencies across split points in a single forward pass;
- For application scenarios requiring the model to possess **long-context/long-range reasoning capabilities**, chunking strategies need to make trade-offs—such as the choice of split points (e.g., splitting as much as possible at natural paragraph/chapter boundaries rather than blindly truncating by a fixed token count), and whether it is necessary to **specifically construct "long-context training samples"** (such as the long-text fine-tuning discussed in Question 374) to make up for the limitations introduced by pre-training chunking.

**4. Relationship Between Positional Encoding and Chunking Length**

- The sequence length after chunking directly determines the positional encoding range actually "seen" by the model during training—if all chunked sequences during pre-training are "exactly equal" to the maximum length (filled via concatenation), the model may be **under-trained** on positional encoding regions corresponding to "shorter sequences that do not fill the maximum length", which may affect the model's performance when handling "shorter inputs" in certain scenarios (though this impact is relatively minor, as short sequences themselves are "prefixes" of long sequences, and prefix positional encodings are also fully trained during long-sequence training).

**Summary**: The core trade-off of document chunking strategies lies between "**maximizing training efficiency (reducing padding through concatenation to increase the effective token ratio)**" and "**avoiding spurious cross-document dependencies (requiring document-level attention masks, which increases implementation complexity) as well as preserving genuine long-range dependencies within documents (choice of split points, handling long documents)**"—different training frameworks/teams select whether to implement document-level masks and specific chunking/concatenation strategies based on their own balance between "efficiency" and "theoretical rigor".

</details>

**9. How Are Long and Short Texts Jointly Trained in Pre-training?**

<details>
<summary>💡 Show Answer</summary>

Considerations for joint training of long and short texts:

**1. Concatenation Strategy (Refer to Question 469)**

- Short texts are typically formed by **concatenating multiple short documents** into training sequences close to the maximum length, improving token utilization;
- Long texts (exceeding the maximum length) need to be **chunked** into multiple sequences, but splitting should ideally occur at natural boundaries (paragraphs/chapters). Consider whether association information between split segments needs to be preserved (e.g., adding "context summaries" as prompts at the beginning of each split segment to help the model understand its position within the overall document—though this increases data processing complexity and is not adopted in all practices).

**2. Design of Data Proportions**

- Short texts (such as news, social media posts, Q&A pairs) are abundant in quantity and broad in coverage, helping the model learn extensive language patterns and world knowledge;
- Long texts (such as books, long papers, technical documents) contain **more complex discourse structures, long-range dependencies, and deep reasoning processes**, which are crucial for cultivating the model's "long-context understanding and utilization capabilities";
- The ratio between the two needs to be balanced according to the target application scenarios—if the primary application scenarios of the model involve processing large amounts of long documents (such as document summarization, long conversations), the proportion and "exposure" of long text in the training data should be appropriately increased.

**3. Curriculum-Learning-Based Length Increasing Strategy**

- Some pre-training practices adopt a **"short-to-long" curriculum learning strategy**: in the early training stage, shorter sequence lengths (e.g., 2K/4K) are mainly used, yielding high computational efficiency (the $O(n^2)$ complexity of attention computation has low overhead on short sequences) and allowing the model to quickly establish basic language capabilities; in the later/specific training stages, sequence length is gradually increased (e.g., extending to 8K/32K or even longer), introducing more long-text data to **specifically train the model's long-context processing capabilities**;
- This strategy strikes a balance between "training efficiency" (using short sequences early on to reduce computational overhead) and "final long-context capabilities" (specifically training on long sequences later on), avoiding the prohibitive computational cost of training with the maximum length from start to finish.

**4. Positional Encoding Considerations**

- As discussed in Question 374, if later training stages need to expand to sequence lengths longer than early stages, the extrapolation capability of positional encodings (such as RoPE scaling techniques) must be considered to ensure that long-distance positional encoding regions "only seen heavily in mid-to-late training" receive sufficient and correct training.

**5. Evaluation**

- It is necessary to separately evaluate the model's performance on **short-text tasks** (such as sentence-level classification, short Q&A) and **long-text tasks** (such as long document summarization, long-context Q&A, "needle-in-a-haystack" tests) to ensure that the mixed training strategy achieves good results on both types of tasks, rather than "biasing" toward one end.

**Summary**: Core strategies for mixed training of long and short texts include "**improving short-text token utilization via concatenation, splitting long text by natural boundaries; designing long/short text ratios based on application scenarios (long text is vital for cultivating long-context capability); adopting a short-to-long curriculum learning strategy to balance training efficiency and long-context capability; coordinating with positional encoding extrapolation; and validating the mixed strategy performance through separate evaluations on short and long texts**". It is a comprehensive design integrating computational efficiency, application requirements, and positional encoding techniques.

</details>

**10. How Does Data Format Impact Training Speed?**

<details>
<summary>💡 Show Answer</summary>

![Sequence packing memory efficiency vs standard padded sequence format.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig15_data_packing_format.png)




The impact of data format on pre-training speed is mainly reflected in the following aspects:

**1. Offline Preprocessing (Tokenization)**

- If **on-the-fly tokenization of raw text** is performed during training, CPU-intensive tokenization calculations are required for every training step, which can easily become a bottleneck for GPU computation (GPUs waiting for CPUs to complete tokenization);
- A more efficient approach is to **tokenize all text in advance (offline) and store it in a binary format (such as Megatron's indexed dataset format, which directly stores token ID sequences as memory-mapped binary files)**, so that numeric token ID sequences can be read directly and efficiently from disk during training, eliminating real-time tokenization overhead during training.

**2. Data Reading I/O Efficiency**

- **File formats**: Using formats optimized for "random access and efficient reading of large-scale data" (such as memory-mapped files mmap, columnar storage formats like Parquet, TFRecord, etc.), compared to simple text files (which require line-by-line parsing for every read), can significantly improve I/O throughput;
- **Data sharding**: Splitting data into multiple shard files facilitates different data-parallel ranks in distributed training to read their respective shards concurrently (refer to Question 326), avoiding a single large file becoming an I/O bottleneck or multiple processes competing for read locks on the same file.

**3. Serialization and Deserialization Overhead**

- Avoid formats requiring complex parsing (such as line-by-line JSON parsing) and prefer **fixed-format binary encodings** (such as directly storing int32/int64 token ID arrays), which minimize deserialization overhead.

**4. Data Loading and Prefetching**

- Even if the data format itself is efficient, if the data loading speed (reading from disk + deserializing + assembling into batches) cannot keep up with the GPU compute speed, GPUs will still idle while waiting;
- It is necessary to incorporate **multi-process/multi-threaded data loading (such as PyTorch DataLoader's num_workers) and prefetching mechanisms** to overlap data loading with GPU computation (refer to Question 326).

**5. Dynamic Length and Padding (Refer to Question 469)**

- If sample lengths vary in the data format and training requires **padding to a unified length**, having metadata like "actual length of each sequence" stored in the data format supports **dynamic batching** (length bucketing, refer to Question 326) to reduce computational waste from padding; if the format does not support efficient retrieval of length information (e.g., requiring a full read to know the length), implementing dynamic batching becomes more complex and inefficient.

**6. Data Format Consistency in Distributed Scenarios**

- In multi-node multi-GPU training, data formats must support **efficient sharded reading** (each rank reads only its assigned portion, refer to Question 326). Improper format design (e.g., all ranks needing to read the entire index file to locate their data shard) can incur extra I/O and memory overhead in large-scale distributed setups.

**Summary**: The impact of data format on training speed is mainly reflected in aspects such as "**whether real-time tokenization during training is needed (offline preprocessing is usually superior), I/O read efficiency (binary/memory-mapped formats excel over line-by-line text parsing), serialization/deserialization overhead, ease of multi-process prefetching and data-parallel sharded reading, and support for efficient dynamic lengths/dynamic batching**"—the core principle is to **shift data-processing-related computations (tokenization, format parsing) prior to training as much as possible (completed offline), performing only the simplest and most efficient binary data reads and batch assembly during training**, maximizing GPU compute resource utilization and avoiding data processing becoming a bottleneck in the training pipeline.

</details>

**11. How to Optimize Token Utilization to Improve Training Efficiency?**

<details>
<summary>💡 Show Answer</summary>

Methods to improve token utilization (i.e., the ratio of "effectively computed tokens / actually processed tokens"):

**1. Reducing Padding (Refer to Questions 469 and 471)**

- **Document concatenation**: Concatenating multiple short documents into sequences close to the maximum length to reduce large amounts of ineffective padding tokens caused by individually padding short documents;
- **Length bucketing**: Grouping samples of similar lengths into the same batch and padding each batch according to the maximum length within that batch (rather than the global maximum length), reducing unnecessary padding.

**2. Increasing the Density of "Effective Signals" Within Sequences**

- For Causal Language Modeling (CLM), **all tokens in the sequence except the first participate in loss computation**, so token utilization is theoretically already high;
- However, if a sequence contains a large amount of **low-information content** (such as repetitive patterns, boilerplate text), even though these tokens "participate in computation", their "effective contribution" to model learning remains low—this relates to the "data quality" issues discussed in Questions 462--464; high-quality, high-information-density data is itself an important avenue to boost "effective token utilization".

**3. Efficient Attention Implementations**

- Using efficient attention kernels like FlashAttention reduces the memory and time complexity of attention computation (reducing memory footprint from $O(n^2)$ to linear), enabling the processing of **longer sequences / larger batches** under the same hardware resources and indirectly raising overall token processing throughput.

**4. Mixed Precision and Efficient Operators**

- As discussed in Questions 328--330, mixed precision training (BF16/FP16) combined with efficient matrix multiplication kernels (leveraging Tensor Cores) speeds up computation per token.

**5. Activation Recomputation and Memory Optimization (Refer to Chapter 13)**

- Memory optimization techniques like activation recomputation (gradient checkpointing) and ZeRO support larger batch sizes/sequence lengths within a limited GPU memory budget, enhancing GPU utilization (MFU).

**6. Data Deduplication and Quality Filtering (Refer to Questions 462--468)**

- Removing duplicate and low-quality data essentially **prevents "wasting" precious training compute resources on "low-information/repetitive" tokens**—ensuring that tokens come from "high-information-density" content as much as possible under a fixed total training token budget.

**7. Document-Level Attention Masks (If Concatenation Strategy is Adopted, Refer to Question 469)**

- Although this is intended to "avoid erroneous signals", from another perspective, **ensuring that each token's attention calculation "focuses" on truly relevant context** is also a way to improve the efficiency of converting "computation" into "effective learning signals".

**8. Vocabulary Size and Tokenizer Efficiency (Refer to Question 477)**

- An efficient, well-covering tokenizer can represent the **same semantic content with fewer tokens** (higher "compression ratio"). Under a fixed "token budget", this is equivalent to accommodating more actual information, indirectly boosting training efficiency.

**Summary**: Methods to improve token utilization encompass multiple dimensions, including "**reducing padding (concatenation, length bucketing), enhancing the information density of data itself (deduplication, quality filtering), using efficient computational kernels (FlashAttention, mixed precision) to support larger batches/longer sequences, reasonable attention mask designs, and efficient tokenizer designs**"—essentially optimizing jointly from the dual perspectives of "**reducing invalid computations (padding)**" and "**increasing the density of 'effective learning signals' per unit of computation (data quality, tokenizer efficiency)**", with the ultimate goal of maximizing the model's "effective learning" given a fixed compute budget (GPU time).

</details>

**12. Why Include a Large Amount of Code Data in Pre-training Corpora?**

<details>
<summary>💡 Show Answer</summary>

Reasons for incorporating large amounts of code data into pre-training corpora (even when the model's primary downstream use is not programming):

**1. The "Logical Structure" of Code Helps Cultivate Reasoning Capabilities**

- Code is a **highly structured and logically rigorous** form of text—variable definitions, control flows (conditional branches, loops), function call relationships, and data structure operations all embody clear **logical reasoning and step-by-step execution processes**;
- When learning to "predict the next token in code", the model needs to understand and "simulate" this logical structure (such as understanding changes in a loop variable or correspondences between function parameters and return values). Such training may help the model develop more generalized "step-by-step reasoning" and "structured thinking" capabilities (refer to Question 482 for further discussion on specific mechanisms).

**2. Code Provides Training Signals for "Long-Range Dependencies" and "Exact Matching"**

- Code exhibits extensive **long-range dependencies** (e.g., functions defined at the beginning of a file and called multiple times later; variables assigned earlier and used later), and such dependencies usually demand **exact matching** (variable and function names must match perfectly, otherwise code fails to run)—this requirement for "exactness" is far stricter than in natural language (where pronoun or synonym substitutions rarely hurt comprehension, whereas variable name typos cause syntax/logic errors), potentially helping the model improve its capacity for modeling "exact consistency".

**3. Code Corpora Have Relatively High Quality and Clear Structure**

- Compared to web text, code in open-source repositories (such as GitHub) has generally undergone some degree of "human vetting" (at least being runnable/compilable), so the overall "noise level" tends to be relatively low; moreover, built-in code **comments, docstrings**, etc., provide alignment signals between "code and natural language descriptions", helping the model learn mappings between code and its functional specifications.

**4. Empirical Effects—Code Data Proves Beneficial for Non-Coding Tasks As Well**

- Experimental reports from multiple LLMs mention that **incorporating code data into pre-training data not only enhances the model's coding abilities, but also positively impacts performance on other tasks (such as certain reasoning benchmarks)**—although the exact mechanism behind this "cross-domain transfer" remains not fully clear (refer to discussion in Question 482), empirical results support the observation that "code data possesses value far beyond 'coding ability' itself".

**5. Meeting the Application Demands of the Model Itself**

- Of course, the most direct reason is: **one major application scenario for modern LLMs is code generation/assisted programming**, and incorporating substantial code data is the most direct and indispensable way to cultivate this capability.

**Summary**: Reasons for including large amounts of code data encompass both the obvious goal of "**directly satisfying application needs such as code generation**" and **deeper hypotheses and empirical observations that code data helps foster general reasoning/logical skills**—the structured, logically rigorous, and exact-matching properties of code provide a unique "thought training" material complementary to natural language text, which is why even models primarily targeted at "dialogue/general tasks" typically contain a significant proportion of code data in their pre-training corpora.

</details>

**13. How is Multimodal Data Aligned with Text Data?**

<details>
<summary>💡 Show Answer</summary>

![Multimodal feature alignment projecting vision embeddings into LLM text space.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig16_multimodal_data_alignment.png)




Common methods for aligning multimodal (e.g., image-text) data:

**1. Contrastive Learning (e.g., CLIP Paradigm)**

- As discussed in Question 344, an image encoder and a text encoder are used to map images and text into a **shared vector space**, respectively. Training with contrastive loss (InfoNCE) ensures that **paired image-text samples (such as an image and its description) are close to each other in the vector space, while unpaired samples are far apart**;
- This approach learns "global semantic alignment" between images and text, commonly used in tasks such as retrieval and zero-shot classification, and serves as the pre-training strategy for many multimodal large models (e.g., the vision encoder component).

**2. Unified Sequence Modeling with Interleaved Image-Text**

- Images are represented as a series of **discrete "visual tokens"** (encoded into token sequences from a discrete codebook using image tokenizers such as VQ-VAE/VQGAN) and **concatenated with text tokens into a unified token sequence**. A single Transformer model (similar to CLM) is then used for autoregressive training—this approach unifies "image understanding/generation" and "text understanding/generation" within the same sequence modeling framework. The model learns the associations between images and text by predicting the "next token in the visual token sequence" or the "next token in the text token sequence" (as seen in native multimodal models like GPT-4o and Chameleon).

**3. Vision Encoder + Projection Layer Connected to Language Models (e.g., LLaVA-like Architectures)**

- A pre-trained vision encoder (such as CLIP's image encoder) is used to extract image feature representations. A **projection layer (typically a simple linear layer or MLP) then maps these image features to the embedding dimension of the language model**, allowing image features to be inserted as "special tokens" into the text token sequence and fed into the language model alongside text tokens;
- During training, typically the **vision encoder and language model are frozen first, and only the projection layer is trained** (to align image features with the embedding space of the language model), followed by **end-to-end instruction tuning** (fine-tuning both the projection layer and the language model, and sometimes fine-tuning the vision encoder as well) to gradually achieve deep fusion of image and text.

**4. Sources and Alignment Quality of Large-Scale Image-Text Pair Data**

- "Image + surrounding text (alt text, titles, context paragraphs)" on the Internet is the primary source of image-text alignment data, but the **alignment quality of such data is inconsistent** (many alt texts are meaningless placeholders or have little relevance to the image content);
- Quality enhancement requires **filtering/scoring** (such as using existing image-text matching models to score the relevance of image-text pairs and filter out low-scoring samples) to improve the quality of alignment data.

**5. Cross-Modal Temporal/Structural Alignment (For Complex Multimodal Inputs like Video and Audio)**

- For video (image sequences + audio + text subtitles), **alignment along the temporal dimension** must be considered—namely, the correspondence among video frames, audio segments, and subtitle text on the timeline. Training data needs to include such temporal alignment information (e.g., timestamped subtitles), and the model architecture must handle the synchronization of multimodal sequences in the temporal dimension accordingly.

**Summary**: Methods for aligning multimodal data with text data primarily include several main paradigms: "**contrastive learning (CLIP paradigm, learning global alignment in a shared semantic space), unified discrete token sequence modeling (converting all multimodal content into token sequences for autoregressive training under a single framework), and vision encoder + projection layer connected to language models (LLaVA paradigm, mapping multimodal features to the embedding space of the language model)**." At the data level, focus must be placed on the **alignment quality of large-scale image-text pair data (filtering low-relevance samples)**, and for more complex modalities like video, **cross-modal alignment along the temporal dimension** must also be considered.

</details>

**14. How Does Data Distribution Affect Model Generalization Ability?**

<details>
<summary>💡 Show Answer</summary>

The impact of data distribution on model generalization ability is primarily reflected in the following aspects:

**1. "Coverage" Between Training Distribution and Application Distribution**

- Generalization ability is essentially the capability of a model to **perform well on inputs unseen during training**. The upper bound of this capability is constrained by **how well the training data distribution covers the "actual application scenario distribution"**. If there is a significant discrepancy between the training data distribution (topics, styles, languages, task types, etc.) and the input distribution encountered after deployment (**Out-of-Distribution, OOD**), the model's performance on these OOD inputs will typically decline significantly.

**2. Data Diversity and "Interpolation/Extrapolation" Capabilities**

- A more **diverse** training data distribution provides denser and broader "sampling points" in the input space. The functions (mapping relationships) learned by the model based on these sampling points usually achieve good generalization in the "regions between sampling points (interpolation)"; however, in "regions far from all sampling points (extrapolation)," generalization performance is typically poor;
- Therefore, **increasing training data diversity (covering more topics, styles, task types, and languages)** essentially expands the scope covered by interpolation, thereby enhancing the model's generalization capability over a broader range of inputs.

**3. Data Distribution Imbalance and the "Long-Tail" Problem**

- If the training data distribution exhibits severe **imbalance** (certain topics/patterns dominate while "long-tail" topics/patterns have very few samples), the model may perform well on the "head" distribution but poorly on the "long-tail" distribution (because long-tail samples provide too few training signals for the model to learn proper representations);
- This aligns with the manifestation of the "class imbalance" problem discussed in Question 358 at a broader "data distribution" level.

**4. Data Distribution and "Shortcut Learning"**

- If the training data contains certain **"surface features" that are unrelated to the true target of the task but happen to have strong predictive power under that distribution (spurious correlation)**, the model may learn a "shortcut solution" that relies on these "shortcut features" rather than learning truly generalizable features related to the essence of the task;
- Such "shortcut solutions" may perform well on the "training distribution" (because the spurious correlation holds in the training distribution), but on a **new distribution where the spurious correlation no longer holds**, the model's generalization ability will collapse. This is an important and subtle aspect of how data distribution impacts generalization, often requiring **specially designed "adversarial" or "out-of-distribution" evaluation benchmarks that break such spurious correlations** to be uncovered.

**5. The "Temporal" Dimension of Data Distribution (Distribution Drift)**

- As discussed in Question 346, training data reflects the state of the world and language distribution "prior to a certain point in time." As time progresses, the real-world distribution undergoes **drift (concept drift)**—new vocabulary, new events, and new expressions continuously emerge. Consequently, the model's generalization performance on new inputs after "distribution drift" gradually degrades, requiring continual learning or updates to maintain.

**Summary**: The impact of data distribution on model generalization capability is multidimensional: **the "breadth of coverage" of the distribution determines the range of inputs where the model generalizes well (broader coverage yields a larger interpolation range); the "balance" of the distribution affects performance in long-tail scenarios; potential "spurious correlations" within the distribution can induce the model to learn non-generalizable "shortcut solutions"; and the "timeliness" of the distribution determines the rate at which model capability degrades over time**. When constructing training data, these dimensions must be comprehensively considered to maximize the coverage and representativeness of the data distribution relative to the "real application scenario distribution," while remaining vigilant against spurious correlation patterns that might induce "shortcut learning."

</details>

**15. How to Construct High-Quality Corpora When the Data Volume Is Very Small?**

<details>
<summary>💡 Show Answer</summary>

Methods for constructing high-quality pre-training / continued pre-training corpora when data volume is very small (e.g., specific vertical domains, low-resource languages, emerging fields):

**1. Maximize the "Quality Density" of Existing Data**

- Perform **extremely rigorous cleaning and filtering** on the limited raw data (refer to Questions 463 and 464). Even if this further reduces data volume, ensure that every piece of retained data is "high information density and low noise." When the data volume is already small, the principle of "quality over quantity" (refer to Question 462) becomes even more critical—training on a small amount of high-quality data often yields better results than using a larger volume of inconsistent quality data.

**2. Data Augmentation and Synthesis (Extending the Idea from Question 376)**

- Leveraging **LLMs for data generation**: Use existing powerful general-purpose models, based on seed documents or knowledge points, to generate large amounts of synthetic text via **rewriting, expansion, and multi-angle explanations**, thereby expanding corpus scale and diversity;
- Traditional data augmentation methods such as **back-translation and paraphrasing**.

**3. Fully Utilize Related Domains / General Corpora for "Transfer"**

- Instead of relying entirely on scarce domain-specific data to train from scratch, **perform continued pre-training starting from a model already pre-trained on general or related domain corpora (refer to Questions 351--353)**—using the small amount of target-domain data for "adaptation/alignment" rather than "building all model capabilities from scratch."

**4. Integration of Multi-Source Data**

- Even if the volume of data from a single source is small, try integrating **multiple related sources** (e.g., papers, patents, industry reports, Q&A community posts, and textbooks in the target domain) to increase the overall data volume and the diversity of perspectives covered through integration.

**5. Manual Writing and Crowdsourced Supplementation**

- For extremely scarce yet critical content (such as key knowledge points or demonstrations in specific formats), consider **manually writing** a small amount of high-quality "seed" content, and then expanding upon it using the aforementioned LLM augmentation methods.

**6. Cross-Lingual Transfer (For Low-Resource Language Scenarios)**

- As discussed in Question 357, leverage high-resource language data (e.g., English) that has **parallel corpora** with the target low-resource language, transferring rich corpora from high-resource languages to the target language via translation/alignment.

**7. "Small Data" Challenges in Quality Assessment and Solutions**

- When data volume is small, the traditional approach of "training a quality classifier requiring large numbers of positive and negative samples" may not be applicable. Consider **using LLMs (e.g., GPT-4 class models) to score/audit the quality of each data entry** (acceptable cost due to the small data volume itself), replacing traditional classifier training methods.

**Summary**: The core strategy for constructing high-quality corpora in low-data scenarios involves: "**rigorous quality filtering (quality over quantity) + leveraging LLMs for data augmentation/synthesis to expand scale and diversity + continued pre-training based on existing general/related domain models (transferring existing capabilities rather than building from scratch) + integrating multiple related data sources + manually writing key seed content when necessary + utilizing cross-lingual transfer (for low-resource language scenarios)**." In essence, it uses "**more refined manual/AI-assisted processing**" to compensate for the "insufficiency of raw data volume," maximizing the value of limited data resources.

</details>

**16. What Impact Does Vocabulary Size Have on Training and Inference?**

<details>
<summary>💡 Show Answer</summary>

Vocabulary size is a core parameter in tokenizer design, impacting training and inference across multiple dimensions:

**1. Sequence Length (Compression Ratio)**

- **Larger vocabulary size allows each average token to represent more characters / longer subwords**, allowing identical text content to be expressed in **fewer tokens** (higher "compression ratio");
- This means: under a fixed "maximum sequence length (e.g., 4096 tokens)" constraint, a **larger vocabulary can accommodate more actual text content** (longer effective context); at the same time, for a fixed amount of training text, a **larger vocabulary corresponds to a smaller total token count**, which may affect the "total training tokens" key metric in Scaling Laws.

**2. Parameter Count of Embedding Layer and Output Layer (lm_head)**

- Embedding layer parameter count = Vocabulary size $\times$ Hidden dimension; the parameter count of the output layer (language model head) is likewise proportional to vocabulary size;
- **Larger vocabulary size results in larger parameter counts for both parts**—for smaller models, the vocabulary parameters may represent a non-negligible fraction of total parameters; for ultra-large models, while their relative proportion is smaller, the absolute numbers remain substantial (e.g., a vocabulary of 200,000 with a hidden dimension of 8,192 yields roughly 1.6 billion parameters for the Embedding layer alone).

**3. Computational Overhead—Especially Output Layer Softmax**

- The output layer needs to compute softmax over logits of the **vocabulary size dimension** (cross-entropy loss computation during training also occurs along this dimension). A **larger vocabulary increases computational cost (and corresponding GPU memory footprint, especially gradients during backpropagation) along this dimension**—for models with very large vocabularies, this computation can become a non-negligible source of overhead, prompting engineering optimizations such as **chunked/approximated softmax computation**.

**4. Multilingual Coverage and "Fairness" Across Languages (Refer to Questions 465 and 357)**

- The degree to which a vocabulary covers each language directly affects the tokenization efficiency of text in that language—if common words/characters in a language lack corresponding efficient tokens in the vocabulary, text in that language will be split into **more and finer-grained (e.g., byte-level) tokens**, leading to:
- The same semantic content "consuming" more tokens in that language (resulting in less "effective content" within the context window);
- Generating content in that language requiring more decoding steps during inference (slower and more costly).
- Therefore, **vocabulary design requires a balance between "total vocabulary size" and "balanced coverage across target languages"**—a small vocabulary optimized for a single language will yield poor efficiency for other languages, whereas a large vocabulary attempting to cover all languages will significantly increase Embedding/output layer parameter count.

**5. Rare Word / OOV (Out-of-Vocabulary) Handling**

- Modern subword/byte-based tokenizers (such as BPE and SentencePiece) generally do not suffer from strict OOV issues (any text can be split into subword/byte combinations within the vocabulary). However, **vocabulary size affects the "granularity of segmentation for rare words"**—a larger vocabulary allows more "complete words" to directly map to a single token, whereas with a smaller vocabulary, rare words must be split into finer pieces, impacting the representation quality and processing efficiency of those words.

**6. Inference Speed**

- Inference (autoregressive generation) proceeds token-by-token. **Fewer tokens required to generate identical semantic content (higher vocabulary compression ratio) results in fewer total steps, which generally speeds up inference (measured by "time required to generate a complete response")**—however, the output layer softmax computation overhead increases as vocabulary grows (increasing per-step computation), requiring a comprehensive trade-off between "fewer steps" and "increased per-step compute."

**Summary**: The impact of vocabulary size encompasses multiple dimensions: "**sequence compression ratio (affecting effective context length and total training token count), Embedding/output layer parameter counts and computational overhead (especially softmax), balanced multilingual coverage, and inference speed (the trade-off between step count and single-step compute)**." It represents a design choice requiring a holistic balance based on target language distribution, model scale, context length requirements, and other factors. Current mainstream LLMs typically have vocabulary sizes ranging from tens of thousands up to around 200,000, with an increasing number of models adopting larger vocabularies to meet multilingual and multimodal requirements.

</details>

**17. How to Detect Test Set Leakage in Pre-Training Data?**

<details>
<summary>💡 Show Answer</summary>

Methods for detecting whether pre-training corpora contain evaluation benchmark (test set) content:

**1. n-gram Overlap Detection**

- The most commonly used method—extracting **n-grams (such as 10-grams or longer)** from each sample in the evaluation benchmark (test set) and searching the pre-training corpus for **exact n-gram matches**;
- If a longer n-gram sequence within the question or answer text of an evaluation sample finds an exact match in the pre-training corpus, the sample is considered "potentially contaminated."

**2. Detection Based on Near-Deduplication Techniques (Refer to Questions 467 and 468)**

- Treat the "evaluation benchmark" as a special "document collection" and apply the same techniques used in near-deduplication (such as MinHash + LSH) to detect content in the pre-training corpus that is **highly similar** to the evaluation samples (without requiring exact n-gram matches, allowing detection of paraphrasing, reprinting, etc.).

**3. Probing Model Memorization Capabilities (Indirect Method)**

- Use the **question portion of evaluation samples as prompts** to prompt the model (trained on the pre-training corpus) for continuation, checking whether generated content **closely aligns with the standard answers of evaluation samples**—if the model can "accurately complete" test set answers (especially specific standard answer formulations unlikely to be derived purely through reasoning), it suggests the sample may have been "seen" during training.

**4. Comparing "Anomalous Performance Gains" Before and After Model Training**

- If a model's performance on a specific evaluation benchmark exhibits a **disproportionate and anomalously prominent advantage** compared to its performance on other benchmarks of "similar nature but verified un-contaminated," this may signal potential contamination of that benchmark (requiring further confirmation via n-gram / similarity detection).

**5. Targeted Auditing of Data Sources**

- Evaluation benchmarks are typically published on specific websites, in papers, or in GitHub repositories. If the crawling scope of the pre-training corpus **includes these publishing channels** (such as GitHub repositories containing benchmark datasets, arXiv paper pages, or related discussion forums), the likelihood of the "raw content" of evaluation data being included in the pre-training corpus increases significantly—**targeted source auditing and exclusion** can be conducted for these known "evaluation benchmark publishing channels."

**Summary**: Methods for detecting test set leakage include: "**exact/high-similarity matching detection based on n-gram overlap, utilizing near-deduplication techniques (MinHash/LSH) to detect paraphrased similar content, indirectly evaluating whether answers were 'memorized' by probing model completion capabilities on evaluation questions, comparing whether the model displays anomalous performance gains on the benchmark, and conducting targeted data source audits on known benchmark publishing channels**." This work needs to be conducted at both the data processing stage (pre-prevention: excluding known benchmark publishing channels and filtering with n-gram detection) and the model evaluation stage (post-auditing: probing and comparative analysis).

</details>

**18. What Impact Does Data Contamination Have on Model Evaluation?**

<details>
<summary>💡 Show Answer</summary>

**Data Contamination**: Refers to the situation where pre-training (or fine-tuning) data contains questions and/or answers from evaluation benchmarks (test sets), leading to "inflated" model performance during evaluation that fails to truly reflect the model's generalization capability.

**Impacts**:

**1. Distortion of evaluation scores, failing to reflect true capabilities**

- If a model has already "seen" evaluation test items (or even standard answers) during the pre-training phase, it is very likely not deriving answers through "reasoning/understanding" during evaluation, but rather **directly "recalling/retrieving" content seen in the training data**—in this case, the evaluation score measures the model's "memorization ability" more than the "understanding/reasoning/generalization ability" that the benchmark intended to assess, leading to a **decoupling between evaluation scores and the model's true capabilities**.

**2. Unfairness in cross-model comparisons**

- Different models/teams may have varying "degrees of contamination" during data collection and processing—if a model's pre-training corpus "happens to" contain more content from an evaluation benchmark (whether intentionally or unintentionally), its score on that benchmark will be "unfairly" higher than that of other models with lower contamination, causing **model leaderboards and comparative results to lose fairness and credibility**.

**3. Misleading research directions and practical application decisions**

- If researchers or users judge the "effectiveness" of a model, training method, or data mixture ratio based on "contaminated high scores," it may lead to **erroneous conclusions**—for example, falsely believing that a certain data mixture strategy significantly improves "reasoning capability," when in fact the improvement stems from the strategy accidentally introducing more benchmark-related content;
- In practical applications, if a model is selected for a task based on "inflated" evaluation scores, **the model's performance in real-world application scenarios (unseen inputs distinct from evaluation test items) may fall far short of what the evaluation score suggests**, resulting in a gap between expected and actual performance.

**4. "Devaluation" of evaluation benchmarks themselves**

- Once an evaluation benchmark is found to suffer from widespread contamination, its validity for "measuring model capability" is greatly compromised. The research community may need to **redesign or update evaluation benchmarks** (such as using "newly constructed test items that have never been publicly released"), thereby increasing the maintenance cost of the evaluation ecosystem.

**5. The dilemma of confounding "contamination" with "genuine capability improvement"**

- A nuanced issue is that if **certain knowledge points or question types** of an evaluation benchmark naturally occur in "broad, reasonable pre-training corpora" (e.g., standard math problems or widely known common-sense questions), the boundary between this "natural overlap" and "deliberate/accidental inclusion of the evaluation set itself" is sometimes **blurry**—overly strict "exclusion of all content related to test items" might inadvertently harm the model's opportunity to learn "general domain knowledge." This makes threshold setting and processing methods for "contamination detection" themselves a subject of ongoing debate and challenge.

**Summary**: Data contamination leads to **inflated evaluation scores that fail to reflect true model generalization capability, disrupts the fairness of cross-model comparisons, misleads research and application decisions based on evaluation results, and damages the credibility of benchmarks themselves**—this is a highly focused issue in current LLM evaluation, driving the development of evaluation methods such as "dynamic benchmarks" (where test items are continuously updated to avoid inclusion in the training data of any released models) and "private/unreleased test sets" to mitigate the impact of contamination.

</details>

**19. How to Construct Vertical Domain Pre-training Corpora?**

<details>
<summary>💡 Show Answer</summary>

![Domain-specific pre-training data extraction and quality filtering pipeline.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig18_vertical_domain_corpus.png)




Methods for constructing pre-training/continued pre-training corpora for vertical domains (such as legal, medical, and financial):

**1. Systematic categorization of domain data sources**

- **Public Data**: Academic papers/journals within the domain (e.g., PubMed in medicine), industry reports, public government documents (e.g., laws, regulations, judicial verdicts), industry news, professional forums/Q&A communities (e.g., medical Q&A platforms, legal consultation websites), textbooks, and reference books;
- **Semi-Public/Authorized Data**: Professional data obtained through cooperation with industry institutions (authorized and anonymized), such as de-identified medical case summaries from medical institutions and research reports from financial institutions;
- **Filtering from General Corpora**: In existing general pre-training corpora (e.g., Common Crawl), **use domain classifiers or keyword rules to filter out content relevant to the target domain** as a supplement.

**2. Domain-specific cleaning and quality control**

- Domain data may possess **special formats** (e.g., fixed structures in legal documents, IMRAD structures in medical papers, tabular data in financial reports). Specialized parsing/extraction logic needs to be designed for these formats to prevent crude "general web cleaning rules" from destroying critical structural information (e.g., tables, formulas) in domain data;
- Domain-specific "quality" standards may also differ from general corpora (e.g., while academic papers generally have high overall quality, sections like "reference lists" and "acknowledgments" may contain relatively "low-information" content and can be targeted for special handling).

**3. Considerations for domain terminology and tokenizers**

- As discussed in Question 357, if a domain contains a large amount of **specialized terminology, abbreviations, or special symbols/codes** (e.g., chemical formulas, legal clause numbers, medical diagnostic codes), it is necessary to evaluate the tokenization efficiency of existing tokenizers on these contents, and when needed, consider **expanding the vocabulary** or making targeted tokenizer adjustments.

**4. Data mixture—blending domain data with general data**

- As discussed in Questions 351--352, when performing continued pre-training using purely domain data, one must consider the **mixing ratio with general data** to balance "domain knowledge injection" with "preservation of general capabilities"; since the volume of domain data is typically much smaller than general corpora, the design of the "mixing ratio" is particularly critical.

**5. Textualization of structured data**

- Vertical domains contain vast amounts of **structured/semi-structured data** (e.g., medical lab reports, financial statements, structured fields in legal judgments). One must consider how to **convert this data into natural language text suitable for language model training** (e.g., "templatizing" structured fields into descriptive sentences, or retaining specific structural tags to allow the model to learn "language representations of structured data").

**6. Privacy and compliance processing (especially in medical and financial domains)**

- Data involving personal privacy (medical records, financial transactions) must undergo **rigorous de-identification/anonymization** (refer to Question 405) to ensure that data usage complies with relevant laws, regulations, and data use agreements.

**7. Domain expert involvement**

- As discussed in Question 373, the construction and quality evaluation of vertical domain corpora **indispensably require domain expert involvement**—experts can help judge the professional accuracy of data, identify standards for "high-quality sources" unique to the domain, and evaluate the completeness of domain knowledge coverage in the final corpus.

**8. Evaluation systems**

- Construct or leverage **domain-specific evaluation benchmarks** (e.g., MedQA in medicine, legal reasoning benchmark sets in law) to evaluate the model's knowledge mastery and application capabilities in the domain after continued pre-training, guiding the iterative optimization of corpus construction and data mixture ratios.

**Summary**: Constructing vertical domain pre-training corpora is a systematic pipeline: "**Systematically categorizing multiple domain data sources (public/authorized/filtered from general corpora) $\rightarrow$ Designing cleaning and extraction logic for domain-specific formats $\rightarrow$ Evaluating and adapting tokenizers for domain terminology coverage $\rightarrow$ Designing the mixture ratio with general data $\rightarrow$ Processing textualization of structured data $\rightarrow$ Strict privacy compliance processing (for sensitive domains) $\rightarrow$ Engaging domain experts for quality control $\rightarrow$ Guiding iterative optimization via domain-specific evaluation benchmarks**." Compared to general corpus construction, it places greater emphasis on **targeted handling of domain specifics (formats, terms, structured data)** and **deep involvement of domain expertise (expert participation)**.

</details>

**20. Why Can Such a Simple Objective as Next-Token Prediction Enable Models to Learn World Knowledge?**

<details>
<summary>💡 Show Answer</summary>

This is a frequently discussed core question regarding the source of capabilities in large language models, which can be understood from the following perspectives:

**1. "Predicting the next token" requires modeling the "entire process" of text generation**

- On the surface, "predicting the next token" appears to be a simple objective. However, to excel at this objective across **massive and diverse** real-world texts, the model must actually "implicitly" model the **underlying process** that produces these texts—texts created by humans based on their cognition, knowledge, logic, and intentions regarding the world;
- For example, to accurately predict that the token following "The capital of France is ____" is "Paris," the model must in some sense "know" this fact; to accurately predict the next step in a logical reasoning sequence, the model needs to "understand" the preceding reasoning logic; to accurately predict the next line of code, the model needs to "understand" the execution logic of the program—**the objective of "accurate prediction" forces the model to encode vast amounts of information related to "world knowledge," "logical reasoning," and "causal relationships" within its parameters**, as this information is essential to performing prediction well.

**2. Massive data contains "compressed traces" of human knowledge and thought processes**

- Text on the Internet is the product of collective human intelligence, knowledge, and expression, encompassing rich content such as encyclopedic knowledge, news facts, scientific discourses, logical reasoning, and conversational dialogues;
- A model trained to "predict such texts" is essentially learning a probability distribution capable of **generating (or reproducing with high probability) this content**—a probability distribution that can "generate factually correct statements and logically sound reasoning" must inevitably "internalize" these facts and logic themselves to a certain degree.

**3. Theoretical connection to "compression"**

- From an information-theoretic perspective, the "optimal code length" of a sequence equals the sum of its negative log probabilities under the true distribution (i.e., cross-entropy). The "next-token prediction" objective is essentially learning the probability distribution of data so that the model's assigned probabilities are as close as possible to the true distribution;
- **"Better prediction" is equivalent to "better compression"**—to achieve "better compression" on data that contains vast amounts of facts, logic, and reasoning processes, the compression algorithm (in this case, the model) must in some sense "understand" the regularities, patterns, and underlying structures within the data ("understanding" makes compression more efficient—if the model "knows" a fact, when encountering relevant context, the predicted probability of that fact will be higher, requiring a shorter "code length"). This profound connection between "compression" and "understanding/intelligence" provides an important theoretical lens for explaining "why a simple prediction objective can give rise to complex capabilities."

**4. The role of scale effects**

- Under small model and data scales, "predicting the next token" might only allow the model to learn superficial statistical co-occurrence patterns (such as word frequencies or simple $n$-gram patterns);
- However, when **both model scale and data scale become sufficiently large** (refer to Scaling Laws in Question 483), the model gains sufficient "capacity" to encode more complex and abstract patterns and knowledge, causing capabilities that do not exist at small scales to "emerge"—this phenomenon of "scale bringing qualitative change" is another key factor enabling a "simple objective + sufficient scale" to yield complex capabilities.

**5. Diverse tasks are naturally "embedded" in the prediction objective**

- Because the training data naturally encompasses various text formats (Q&A, dialogue, reasoning steps, code, explanations, etc.), the unified objective of "predicting the next token" in different contexts **actually corresponds to multiple distinct "subtasks"** (e.g., "predicting the answer to a question," "predicting the next step of reasoning," "predicting the next line of code")—the model simultaneously learns these diverse "subtasks" during training, allowing the final model to demonstrate generalist "multitask" capabilities without needing task-specific training objectives.

**Summary**: The objective of "predicting the next token" appears simple, but **to maximize performance on this objective across massive real-world texts rich in knowledge and logical structures, the model is compelled to (implicitly) learn to encode vast factual knowledge, logical principles, and reasoning patterns**—this can be understood through perspectives such as the "theoretical connection between compression and understanding," "knowledge/logic traces embedded in massive data," and "emergent abilities driven by scale effects." It stands as one of the core theoretical foundations explaining the source of LLM capabilities (though deeper underlying mechanisms remain an area of active research and discussion in academia).

</details>

**21. Why Does Incorporating Code into Pre-training Data Improve Reasoning Capabilities? What Is the Underlying Mechanism?**

<details>
<summary>💡 Show Answer</summary>

![Code corpus pre-training mechanism for LLM reasoning capabilities.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig10_code_pretraining_reasoning.png)




This phenomenon (code data enhancing non-coding reasoning tasks) is currently largely based on **empirical observations**. While its underlying mechanisms are not yet fully clear, several potential explanations/hypotheses exist:

**1. Code training strengthens the ability for "step-by-step execution / state tracking"**

- The execution of code is fundamentally a **step-by-step, deterministic state-transition process**—each line of code generates a new "state" based on the current "state" (variable values). Learning to "predict the next token in code" requires the model, to some extent, to **simulate/track these state changes** (such as understanding how variables accumulate in a loop or which code path a conditional branch executes under specific conditions);
- This ability to "track state changes and derive step-by-step based on rules" shares structural similarities with **multi-step reasoning in natural language (e.g., step-by-step solving of math word problems, logical reasoning chains)**—both involve "deriving the next conclusion/state based on current known information and certain rules." One hypothesis suggests that **the "state tracking" capability cultivated during code training may transfer to the ability to "track intermediate states in reasoning chains" within natural language reasoning tasks**.

**2. Code provides high-quality "Chain-of-Thought (CoT)" exemplars**

- Code itself, especially code with detailed comments, can be regarded as a **highly structured and explicit record of a "thought process"**—code decomposes a complex task into a sequence of clear, executable steps, which is formally highly similar to "Chain-of-Thought reasoning" (decomposing complex problems into intermediate reasoning steps);
- Training a model on vast amounts of code data may be equivalent to "implicitly" exposing it to numerous exemplars of "task decomposition" and "step-by-step problem-solving." This pattern may then be transferred by the model to natural language reasoning tasks.

**3. The "precision" of code corpora trains symbolic manipulation capabilities**

- Mathematical and logical reasoning rely heavily on the **precise manipulation of symbols** (e.g., algebraic transformations, logical formula derivations), which connects closely with code properties where "variable names must match precisely and syntax rules must be strictly obeyed";
- Code training may strengthen the model's capacity to perform precise, rule-based operations on "symbolic systems," a capability that partially transfers to the requirement of "symbolic transformation according to rules" in mathematical and logical reasoning tasks.

**4. Training signal quality derived from "verifiability"**

- This aspect relates more to the **post-training phase (RL)** than pure pre-training—code possesses "executable and verifiable (e.g., pass/fail test cases)" characteristics, enabling code-related tasks to provide **high-quality, objective reward signals** during the RL phase (refer to Questions 438 and 444). RL training on such "verifiable tasks" is itself a major pathway to enhancing reasoning capabilities, while heavy exposure to code during pre-training **lays the foundation** for subsequent effective RL training on code (and structurally similar "verifiable" tasks like mathematics). That is, "code data in pre-training" and "verifiable-reward RL in post-training" likely act as two synergistic stages.

**5. "Bi-directional supervision" between code and natural language**

- Codebases contain vast amounts of **paired code and natural language (comments, docstrings, commit messages, issue discussions)**. In learning this bi-directional mapping between code and natural language, the model may build an **intrinsic connection between "logics/processes described in natural language" and "precise logics/processes embodied in code"**. This connection may enhance the model's ability to translate between "logical descriptions in natural language" and "precise, executable step-by-step procedures," a translation capacity that itself is closely tied to "reasoning."

**Important Note on Uncertainty**:

- It must be emphasized that **the observed effect size of "code training improving reasoning capabilities" varies across different studies**. Its exact causal mechanism (whether it is one or a combination of the above mechanisms, or involves other unidentified factors) **currently lacks a consensus or definitive conclusion in academia**, remaining an open research question. The explanations listed above are primarily **reasonable conjectures/hypotheses based on empirical observations** rather than rigorously proven facts.

**Summary**: The potential underlying mechanisms by which code data enhances reasoning capabilities include several hypotheses: **the structural similarity between the "step-by-step execution / state tracking" nature of code and multi-step reasoning, code serving as exemplars of "structured Chain-of-Thought," code training reinforcing precise rule-based manipulation of "symbolic systems," and the "verifiability" of code laying a foundation of high-quality training signals for subsequent RL phases**. However, the exact causal mechanism of this phenomenon remains an open research question, and responses should reflect this perspective of "reasonable conjectures based on observations rather than settled, proven conclusions."

</details>

**22. How Does Pre-training on Synthetic Data Benefit LLMs?**

<details>
<summary>💡 Show Answer</summary>

![Synthetic Data Generation and Quality Filtering workflow.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig13_synthetic_data_generation.png)




Pre-training on synthetic data involves generating artificial data to supplement the model's training process. This approach provides several benefits:

- **Dataset Expansion**: Synthetic data can be used to create larger and more diverse datasets, thereby helping to improve the model's generalization capabilities.
- **Coverage of Rare Cases**: Synthetic data can generate edge cases or rare scenarios that may not exist in the original training data, thereby enhancing model robustness.
- **Cost-Effectiveness**: Generating synthetic data is generally less expensive than collecting and labeling large amounts of real-world data, making it a cost-effective method to boost model performance.
Pre-training on synthetic data is particularly useful in fields where collecting real-world data is challenging, such as healthcare or autonomous driving applications.

</details>

**23. How Does Knowledge Graph Integration Enhance LLMs?**

<details>
<summary>💡 Show Answer</summary>

Integrating Knowledge Graphs with LLMs enables the model to combine its predictions with structured factual knowledge. This integration offers several advantages:

- **Factual Accuracy**: By referencing a knowledge graph, the model can cross-check facts, reducing the likelihood of hallucinations or generating incorrect information.
- **Enhanced Reasoning Capabilities**: Knowledge graphs enable the model to perform logical reasoning and inference based on relationships between entities, improving its capacity to answer complex queries.
- **Contextual Understanding**: LLMs benefit from the structured nature of knowledge graphs, which helps them better comprehend the context and relationships between different pieces of information.
Knowledge graph integration is particularly useful in applications such as question answering, entity recognition, and recommendation systems.

</details>

**24. What Is Data Cleaning? What Are Common Cleaning Operations?**

<details>
<summary>💡 Show Answer</summary>

Data cleaning is the process of identifying and correcting or removing errors, inconsistencies, missing values, or duplicate content in data. It is the most fundamental and time-consuming stage of a data engineering pipeline. High-quality training data relies on a systematic cleaning workflow; "Garbage In, Garbage Out" serves as the core warning in data engineering. Common operations include:

- **Deduplication**: Identify and remove exact or near-duplicate samples to prevent the model from overfitting to repetitive content (see Near-Deduplication for details).
- **Missing Value Handling**: Apply strategies such as deletion, mean/mode imputation, interpolation, or model-based prediction filling for missing fields, depending on the missing ratio and field importance.
- **Format Standardization**: Standardize date formats, numeric representations, units, capitalization, etc., ensuring consistent representations for semantically identical data.
- **Outlier Detection and Handling**: Identify anomalous data points via statistical methods (e.g., Z-score, IQR) or business rules, choosing to correct, remove, or retain them depending on context (some outliers may represent valuable boundary samples).
- **Encoding Unification**: Standardize text from different sources into a single character encoding (typically UTF-8), handling garbled text, special control characters, HTML entities, etc.
- **Noise Filtering**: For text data, filter out non-main-body noise such as advertisements, navigation bars, and copyright notices (e.g., boilerplate text in web crawler data).
- **Language Identification and Filtering**: Identify and filter out non-target language text to maintain the language purity of the corpus.

In LLM pre-training data processing, the cleaning workflow is typically organized as a pipeline where steps are executed sequentially, recording the amount of data filtered at each step to analyze data loss and adjust strategies.

</details>

**25. What Is Data Deduplication? Why Is It Particularly Important in LLM Training Data?**

<details>
<summary>💡 Show Answer</summary>

Data deduplication is the process of identifying and removing exact or near-duplicate samples from a dataset, categorized into Exact Deduplication and Near Deduplication. In LLM training data, deduplication is particularly critical for several key reasons:

- **Preventing Overfitting and Weight Skew**: Duplicate data causes the model to assign disproportionate weight to such content, effectively performing implicit upsampling of repeated items. Research indicates that high duplication rates in training data lead the model toward verbatim memorization and repetition during generation, diminishing generalization ability.
- **Avoiding Compute Waste**: In trillion-token pre-training, duplicate data means compute resources are squandered on previously seen content rather than acquiring new knowledge, thereby reducing training efficiency.
- **Preventing Data Contamination**: If training data overlaps with evaluation benchmark datasets (e.g., MMLU, HumanEval), the model may achieve artificially inflated scores due to having "seen" test questions, making evaluation results lose reference value. Evaluation benchmark datasets should be strictly excluded from training data during deduplication.
- **Enhancing Model Diversity**: A deduplicated dataset covers a broader spectrum of language patterns and knowledge, aiding the model in learning richer and more generalizable language representations.

In practice, exact deduplication is typically completed rapidly based on document-level or paragraph-level hash values (e.g., MD5, SHA-256); near-deduplication requires more complex methods (e.g., MinHash + LSH) to capture slightly rephrased duplicate content.

</details>

**26. What Is Data Annotation? What Are Common Annotation Methods?**

<details>
<summary>💡 Show Answer</summary>

Data annotation is the process of adding human- or automatically generated labels/ground truths to raw data, making it usable for supervised training, reinforcement learning, or evaluation. Annotation quality directly determines the upper bound of downstream model capabilities. Common annotation methods include:

- **Manual Annotation --- Professional Team**: Completed by professional annotators with domain-specific knowledge, suitable for tasks requiring expert judgment (e.g., medical and legal text annotation, code correctness evaluation). High quality, but high cost and slow speed.
- **Manual Annotation --- Crowdsourcing**: Annotation tasks are distributed to a large number of non-professional annotators via platforms (such as Amazon Mechanical Turk), suitable for relatively simple tasks that do not require specialized backgrounds. Low cost and fast speed, but requires quality control mechanisms (e.g., majority voting among multiple annotators, setting "gold standard" test questions to verify annotator quality).
- **Semi-Automated Annotation (Pre-annotation)**: Existing models are first used to pre-annotate the data, which is then verified and corrected by humans, significantly reducing manual effort. Suitable for scenarios where model pre-annotation accuracy is already high.
- **Strong Model-Based Automatic Annotation (Distillation Annotation)**: Leveraging strong models such as GPT-4 and Claude to directly generate annotations or synthetic instruction-response pairs (e.g., the data generation approach in Self-Instruct and Alpaca). Extremely low cost and scalable to massive volumes, but quality is limited by the generative model's capability, and there are copyright and terms of service compliance risks.
- **Feedback-Based Annotation (RLHF Annotation)**: Annotators perform preference ranking on multiple model outputs rather than generating answers directly, which is used to train a Reward Model. This method reduces annotation difficulty (comparing relative quality is easier than writing answers from scratch) and serves as the data foundation for current mainstream alignment methods.

</details>

**27. What Is Data Versioning? Why Is It Needed?**

<details>
<summary>💡 Show Answer</summary>

Data versioning is the systematic practice of tracking and managing different versions of a dataset, recording the contents, timestamp, and source of changes for each version. It is similar to code version control (Git), but targets datasets as a special artifact. The core reasons why data versioning is needed include:

- **Experimental Reproducibility**: In machine learning experiments, model performance is jointly determined by code, hyperparameters, and data. If the exact data version used for a training run cannot be pinpointed, experiment results cannot be reproduced even if code and hyperparameters are fixed, bringing difficulties to troubleshooting and paper credibility.
- **Problem Traceability and Root Cause Analysis**: When model performance suffers an anomaly (degradation) after an iteration, it is necessary to quickly determine whether it was caused by code changes or data changes. Data versioning enables "rolling back to the previous data version for retraining."
- **Auditability and Compliance**: In scenarios involving copyright and privacy compliance, it is necessary to trace the exact origin and processing workflow of a training dataset to handle potential legal audits.
- **Team Collaboration**: When multiple team members collaboratively maintain datasets, version control records "who made what changes at what time," preventing overwriting each other's work.

Commonly used tools include DVC (Data Version Control, a data version management tool integrated with Git), Delta Lake (a data lake format supporting ACID transactions that inherently records data change history), etc.

</details>

**28. What Is a Data Schema? What Is Its Role in Data Engineering?**

<details>
<summary>💡 Show Answer</summary>

A schema defines the structural specification of data, including field names, data types (integer, string, floating-point, etc.), nullability, enumeration ranges, and inter-field constraint relationships. It acts as a "contract" for data and plays the following core roles in data engineering:

- **Ensuring Data Consistency**: Schemas constrain data to conform to a predefined structure during writing, preventing malformed data from upstream systems from silently polluting downstream systems.
- **Supporting Interoperability Across Systems**: Different systems (data collection, storage, processing, training) agree on data interfaces via schemas. Each system can be developed independently and collaborate correctly as long as it adheres to the schema.
- **Accelerating Data Processing**: Columnar storage formats (such as Parquet) utilize schema information to load only required fields during reading, and leverage type information for efficient compression and encoding, significantly boosting query performance.
- **Supporting Schema Validation**: Setting up schema validation nodes in data pipelines to automatically detect data that does not conform to expected structures and trigger alerts serves as the first line of defense in data quality monitoring.
- **Documentation Role**: The schema itself serves as documentation of the data structure; new team members can quickly understand the contents and meaning of a dataset by reading the schema.

In LLM data engineering, schema management also needs to address "schema evolution" issues---as business needs evolve, fields may be added/removed or types may change, requiring the choice of formats that support backward compatibility (such as Avro and Protobuf) and the formulation of evolution strategies.

</details>

**29. What Is Data Sampling? Why Is Sampling Needed When Building Training/Evaluation Datasets?**

<details>
<summary>💡 Show Answer</summary>

Data sampling is the process of selecting a subset from a larger data population according to a specific strategy. When building training and evaluation datasets, sampling is an unavoidable step. The reasons and common strategies are as follows:

- **Scale Control**: The full volume of data can reach petabyte (PB) scale, making it impossible and unnecessary to use all of it for training or evaluation. Sampling controls the scale within a manageable range while preserving the representativeness of data distribution.
- **Simple Random Sampling**: Sampling uniformly at random from the population. It is suitable for scenarios with relatively uniform data distribution; it is simple to implement but may undersample rare categories.
- **Stratified Sampling**: Stratifying the data according to key attributes (such as category, language, or data source) and performing random sampling within each stratum. This ensures that each stratum has a reasonable representative proportion in the final dataset, avoiding rare categories from being omitted by random sampling.
- **Oversampling and Undersampling**: To address class imbalance, minority classes are repeatedly sampled (oversampling) or majority classes are reduced in sampling (undersampling), making the class distribution of training data more balanced.
- **Importance Sampling**: Assigning different sampling weights based on the sample's value to model training (such as difficulty or informativeness), prioritizing high-value samples to improve training efficiency.
- **Special Considerations for Evaluation Set Sampling**: In addition to focusing on distributional representativeness, evaluation set sampling must ensure no overlap with the training set (preventing data contamination), cover a sufficient number of edge cases, and guarantee statistical significance (sufficiently large sample size).

</details>

**30. What Are the Common Evaluation Dimensions of Data Quality?**

<details>
<summary>💡 Show Answer</summary>

Data quality is a comprehensive indicator measuring whether data is suitable for its intended purpose. It is typically evaluated along the following dimensions:

- **Accuracy**: Whether the data truly and correctly reflects the reality it describes. For example, whether the annotated sentiment polarity aligns with the text content, or whether answers in Q&A datasets are factually correct. This is the most core dimension of data quality and the hardest to evaluate automatically.
- **Completeness**: Whether there are missing values or missing samples in the dataset. This includes field-level completeness (missing rate of a required field) and record-level completeness (whether data for a given scenario provides sufficient coverage).
- **Consistency**: Whether representations of the same entity across different data sources or fields are contradictory. For example, the same person's name spelled differently across documents, or annotation results contradicting text logic.
- **Timeliness**: Whether the data reflects the current, up-to-date state. For time-sensitive scenarios (e.g., news, financial data), outdated data can be more dangerous than having no data.
- **Uniqueness**: Whether duplicate records exist in the dataset. A high duplication rate not only wastes storage but also leads to implicit upsampling bias during model training.
- **Relevance**: Whether the data is relevant to the target task. Even if the data itself is accurate and complete, if its content is irrelevant to the training objective (e.g., using large amounts of celebrity gossip data to train a code generation model), it will not yield effective improvements in model capability.
- **Diversity**: Whether the data covers sufficiently diverse situations of the target scenario, avoiding excessive concentration on certain patterns or styles. This is particularly crucial for the generalization capability of the model.

In practical evaluations, data quality is typically assessed by combining automated metrics (such as statistics on missing rates and duplicate rates) with manual sampling audits.

</details>

**31. What Is Near-Deduplication? What Are Common Implementation Methods?**

<details>
<summary>💡 Show Answer</summary>

Near-deduplication is a technique for identifying and removing data that is highly similar in content but not completely identical (such as duplicate text that has undergone paraphrasing, minor modifications, or formatting adjustments). Exact deduplication (hash matching) cannot capture such "soft duplicates," and near-deduplication fills this gap. Common implementation methods include:

- **MinHash + LSH (Locality-Sensitive Hashing)**: Converts text into N-gram sets, uses the MinHash algorithm to generate compact document signatures, and then leverages LSH to map similar signatures to the same hash bucket, performing exact similarity comparisons only on documents within the same bucket. This is currently the mainstream method for near-deduplication of large-scale corpora, with near-linear time complexity, suitable for trillion-token-scale data processing (adopted by CCNet, RefinedWeb, RedPajama, etc.).
- **SimHash**: Another locality-sensitive hashing method that maps documents to fixed-length binary fingerprints and measures similarity by calculating Hamming Distance. Suitable for web page deduplication scenarios, and was once used at scale by Google.
- **Embedding-Based Semantic Deduplication**: Encodes documents into dense vectors and identifies semantically similar document pairs via cosine similarity or approximate nearest neighbor (ANN, e.g., FAISS) search. Capable of capturing deeper semantic-level duplicates with significant rewriting, but the computational cost is noticeably higher than MinHash schemes; typically used for smaller-scale datasets or scenarios with extremely high quality requirements.
- **Sliding Window Fingerprinting (Suffix Array)**: Constructs suffix arrays to detect shared long substrings between documents, suitable for identifying paragraph-level duplicates (rather than whole-document duplicates).

In practice, exact deduplication and near-deduplication are usually used in series: first using hash-based exact deduplication to rapidly filter out exact duplicates, and then using MinHash + LSH to handle near-duplicates. Combining both steps covers the vast majority of duplication scenarios.

</details>

**32. What Is MinHash? How Is It Applied in Large-Scale Text Deduplication?**

<details>
<summary>💡 Show Answer</summary>

MinHash (Min-wise independent permutations) is a probabilistic algorithm used for rapidly estimating the Jaccard similarity between two sets, proposed by Andrei Broder in 1997, originally used for web page deduplication.

**Core Principle**: Given two sets $A$ and $B$, their Jaccard similarity is defined as $J(A,B) = |A \cap B| / |A \cup B|$. The key theorem of MinHash states that if a random hash function $h$ is applied to the sets and the minimum hash value of the elements is taken as the signature, the probability that the signatures of two sets are equal is precisely equal to their Jaccard similarity:

$$P(\min h(A) = \min h(B)) = J(A, B)$$

By applying $k$ independent random hash functions, a signature vector of length $k$ is obtained. The proportion of matching values at the same positions in two signature vectors serves as an unbiased estimate of Jaccard similarity; the larger $k$ is, the more accurate the estimate.

**Application Workflow in Large-Scale Text Deduplication**:

1. **Text Tokenization**: Split documents into character-level or word-level N-grams (typically character N-grams with $n=5$) to form set representations.
2. **Generate MinHash Signatures**: Apply $k$ (typically 128 or 256) hash functions to the N-gram set of each document to generate fixed-length signature vectors, significantly compressing document representations (from tens of thousands of N-grams down to $k$ integers).
3. **LSH Bucketing**: Divide the signature vector into $b$ bands, each containing $r = k/b$ hash values, and hash the values in each band again to map them into buckets. Theoretically, when the similarity between two documents exceeds a threshold $t \approx (1/b)^{1/r}$, they will drop into at least one identical bucket with extremely high probability.
4. **Candidate Pair Verification**: Compute exact similarity (or directly compare signature similarity) only for document pairs falling into the same bucket, filtering candidate duplicate pairs exceeding the threshold.
5. **Deduplication Decision**: For detected duplicate pairs, retain one version (typically the one with higher quality or more authoritative source) and remove the rest.

The entire workflow can be parallelized using distributed frameworks such as Apache Spark, reducing the deduplication time for hundreds of millions of documents from an infeasible $O(N^2)$ to near-linear, making it one of the core technologies for large-scale corpus deduplication (CCNet, RefinedWeb, RedPajama, etc.).

</details>

**33. What Is a "Quality Filter" in Data Cleaning? What Are Common Filtering Rules?**

<details>
<summary>💡 Show Answer</summary>

A quality filter is a set of rules or models used in a data cleaning pipeline to automatically filter out low-quality content from raw corpora. In web crawl data (such as Common Crawl), a large volume of content consists of advertisements, spam, random strings, and automatically generated low-quality articles. Quality filters represent a critical stage for improving training data quality. Common filtering rules fall into two major categories:

**Heuristic Rule-Based Filtering**:

- **Text Length Filtering**: Filter documents that are too short (e.g., fewer than 100 words) or too long (exceeding reasonable limits). Extremely short texts often lack sufficient information, while extremely long texts may consist of repetitively concatenated content.
- **Special Character Ratio**: If the proportion of special symbols, digits, and punctuation in the text is abnormally high, it usually indicates garbled text, mixed-in code snippets, or formatted tabular text, requiring filtering or special handling.
- **Duplicate Content Ratio**: Calculate the ratio of duplicate lines, duplicate paragraphs, or duplicate N-grams within a document. A high ratio indicates significant copy-pasting or boilerplate content.
- **Lexical Diversity**: Measure the richness of text vocabulary using metrics such as Type-Token Ratio (TTR); a very low value indicates repetitive and monotonous text.
- **Language Confidence**: Use language identification tools (such as fastText's language identification model) to filter non-target languages or texts with high language mixing.
- **Perplexity Filtering**: Use an N-gram language model trained on high-quality data to compute text perplexity. Texts with extremely high perplexity are often out-of-order, garbled, or highly non-natural language content (one of the core steps in the CCNet pipeline).

**Classification Model-Based Filtering**:

- **Quality Classifier**: Train a binary classifier to distinguish high-quality text (positive samples, e.g., Wikipedia, high-quality books) from low-quality web text (negative samples), score new data, and apply threshold filtering. Datasets such as FineWeb and Dolma adopt such classifiers.
- **Toxic Content Filter**: Detect and filter toxic content such as pornography, violence, and hate speech, typically implemented by combining rules (keyword blacklists) with classification models.

</details>

**34. What Is Data Mixture? How Does It Affect Model Training?**

<details>
<summary>💡 Show Answer</summary>

Data mixture refers to the proportional composition of data from different sources, domains, and types in the training set, such as the percentages of web text, code, books, academic papers, dialogue data, and multilingual data. Data mixture is one of the most critical hyperparameters affecting model capability during the pre-training stage, standing equal in importance to model architecture and training algorithms.

**Impact of Data Mixture on Models**:

- **Shaping Capability Directions**: Increasing the proportion of code data (e.g., The Stack, GitHub data) typically boosts programming capabilities and logical reasoning; increasing the proportion of math/scientific papers helps with mathematical reasoning; book data helps with long-text comprehension and writing style.
- **Trade-offs Between Capabilities**: Given a fixed total data volume, increasing one type of data inevitably squeezes out the proportion of other types, which may lead to degradation in other capabilities. For instance, an excessively high code ratio may affect the fluency of natural language.
- **Domain Knowledge Density**: The proportion of domain-specific data (such as medical or legal) determines the model's knowledge depth in that domain; however, domain data is often scarce, and an overly high proportion carries overfitting risks.
- **Language Capability Balance**: In multilingual models, the ratio of data in each language directly affects generation quality and comprehension in that language; low-resource languages often require upsampling.

**Methods for Determining Optimal Data Mixture**:

- **Ablation Study**: Experiment with different mixtures on small-scale proxy models, observe the impact of each mixture on target evaluation metrics, and transfer the optimal mixture to large-scale training.
- **Curriculum Learning**: Dynamically adjust data mixtures during training, such as using massive general web text first to lay a linguistic foundation, and gradually increasing the proportion of high-quality specialized data in later stages.
- **Automated Methods like DoReMi**: Use optimization algorithms to automatically search for data mixtures that yield optimal overall performance on benchmark sets across domains, reducing manual tuning costs.

</details>

**35. What Is Synthetic Data Generation? What Are Its Typical Applications in LLM Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Synthetic data generation is a technique that uses models (typically existing strong generative models) to automatically generate training or evaluation data. It is an important approach to address real-world data scarcity and high annotation costs. In LLM scenarios, synthetic data generation has become one of the core pathways for enhancing model capabilities. Typical applications include:

- **Instruction Tuning Data Generation**: Represented by Self-Instruct, leveraging strong models like GPT-3/4 to automatically generate diverse instruction-response pairs for SFT training. Alpaca (52K instruction data) and WizardLM (complex instruction evolution) are both built on this idea, drastically reducing manual annotation costs for instruction data.
- **Chain-of-Thought Data Generation**: Prompting strong models to generate solution processes with detailed reasoning steps, used to train smaller models to acquire complex reasoning capabilities. DeepSeek-R1 adopted this approach by distilling its reasoning capabilities into the Qwen/LLaMA series.
- **Code and Test Case Generation**: Utilizing code generation models to automatically generate diverse test cases for functions, or generate code samples with comments and documentation for training code models.
- **Multimodal Data Synthesis**: Generating image-text caption pairs (Image Caption), Visual Question Answering (VQA) data, chart interpretation data, and other multimodal training samples to solve the scarcity of annotated multimodal data.
- **Data Augmentation for Scarce Scenarios**: Targeting long-tail scarce scenarios in training data (such as industry-specific domain conversations, texts in rare languages) with targeted synthesis to balance data distributions.
- **Adversarial Sample Generation**: Synthesizing hard samples prone to model errors (such as logic traps or hallucination-triggering scenarios) to selectively improve model robustness.

</details>

**36. What Are the Potential Risks of Training Models with Synthetic Data?**

<details>
<summary>💡 Show Answer</summary>

Although synthetic data offers low cost and high scalability, it also introduces several risks that cannot be overlooked:

- **Bias Inheritance and Amplification**: Biases, misconceptions, and toxic content tendencies present in the generative model itself will be "copied" into the synthetic data. Training new models on this data can cause biases to be systematically amplified and entrenched. This is particularly severe in multi-generational iterations (Model Collapse).
- **Model Collapse**: When a model is trained completely or heavily on self-generated data (i.e., "self-distillation"), after multiple generations of iteration, the model's output distribution gradually narrows, forgetting low-frequency but important knowledge, ultimately causing a sharp drop in diversity and degradation of generation quality. Recent research indicates that continuously injecting real data is key to preventing model collapse.
- **Distribution Shift**: The distribution of synthetic data is influenced by the generative model's preferences and may systematically differ from the distribution of real user interactions. For example, synthetic instruction data might be overly standardized, while real user inputs are often more colloquial and vague, leading to insufficient generalization in real-world scenarios.
- **Propagation of Factual Errors**: Generative models suffer from "hallucination" problems; synthesized knowledge-based data may contain incorrect facts. If directly used for training without human verification, erroneous information will be injected into the new model's parameters.
- **Copyright and Compliance Risks**: Using commercial model APIs (such as OpenAI GPT-4) to generate synthetic data for training competitive models may violate Terms of Service and pose legal risks (Terms of Service for OpenAI, Anthropic, etc., explicitly prohibit such usage).
- **Evaluation Contamination Risk**: If the prompts used to generate synthetic data contain evaluation benchmark questions or similar content, it may lead to overlap between training and evaluation data, resulting in artificially inflated evaluation metrics.

</details>

**37. What Is Feature Engineering? How Has Its Role Changed in the LLM Era?**

<details>
<summary>💡 Show Answer</summary>

Feature engineering is the process of extracting, constructing, and selecting input features from raw data that aid model predictions, serving as a core component of traditional machine learning workflows.

**Traditional Machine Learning Era**: Feature engineering almost single-handedly determined the upper bound of model performance. Taking NLP as an example, feature engineers needed to manually design numerous features such as Bag of Words, TF-IDF weights, part-of-speech tagging features, syntax tree features, and sentiment lexicon matching before feeding them into models like Logistic Regression, SVM, or Random Forest. Feature quality directly dictated model capability, leading to the joke that "feature engineering is the alchemy of machine learning."

**Transition in the Deep Learning Era**: Deep models such as CNNs and RNNs began to automatically learn feature representations from raw inputs, reducing the need for some manual feature engineering, though task-specific feature selection and input design remained important.

**Changes in the LLM Era**:

- **Significant Decline in Importance**: Large language models (GPT, BERT, LLaMA, etc.) automatically learn rich semantic representations from massive raw text through pre-training. The vast majority of NLP tasks require only raw text inputs to achieve powerful performance, making traditional feature engineering almost obsolete.
- **Rise of Prompt Engineering**: "Feature engineering" in the LLM era emerges in a new form---how to design effective prompts (input formatting, example selection, instruction wording) becomes key to influencing model output quality, which is essentially the design and optimization of model inputs.
- **Scenarios Where It Remains Important**: In structured data scenarios (tabular data, recommendation systems, time-series forecasting), feature engineering remains irreplaceable; in RAG/Agent systems, constructing effective metadata features for retrieved documents (e.g., document type, timestamp, source authority) has a major impact on retrieval quality.

</details>

**38. What Is "Inter-Annotator Agreement" in Data Annotation? How Is It Measured?**

<details>
<summary>💡 Show Answer</summary>

Inter-annotator agreement (also known as inter-rater reliability) refers to the degree to which results independently produced by multiple annotators on the same dataset coincide with one another. It is a core metric for measuring whether an annotation task definition is clear and whether annotation quality is reliable.

**Why Inter-Annotator Agreement Is Important**: If two annotators frequently give different annotations for the same sample, it indicates that the annotation standards are ambiguous or annotator comprehension diverges. Under such circumstances, the quality of annotated data is untrustworthy, and models trained on it will find it difficult to learn stable patterns.

**Commonly Used Evaluation Metrics**:

- **Percent Agreement**: The proportion of samples where two annotators give identical annotations. Simple to compute, but fails to account for chance agreement (even with random guessing, a binary classification task has a 50% chance of coincidental agreement).
- **Cohen's Kappa ($\kappa$)**: Based on percent agreement, it subtracts the expected probability of random agreement, serving as a standard metric for agreement between two annotators. $\kappa = (P_o - P_e) / (1 - P_e)$, where $P_o$ is the observed agreement rate and $P_e$ is the expected random agreement rate. Generally, $\kappa > 0.8$ is considered high agreement, $0.6 \sim 0.8$ is moderate agreement, and $< 0.6$ requires re-evaluating annotation guidelines.
- **Fleiss' Kappa**: A multi-annotator extension of Cohen's Kappa, suitable for scenarios with three or more annotators.
- **Krippendorff's Alpha**: A general agreement metric suitable for multiple annotators and various data types (nominal, ordinal, interval, ratio), with stronger capability to handle missing values.
- **Preference Agreement in RLHF Scenarios**: In preference annotation (Preference Ranking), agreement is typically measured by "the proportion of times two annotators select the same preference in the same pairwise comparison."

**Methods to Improve Inter-Annotator Agreement**: Refining annotation guidelines with ample examples, conducting annotator training and alignment calibration prior to annotation, setting up "gold standard" samples to periodically verify annotator performance, and holding dedicated discussions for samples with major disagreements.

</details>

**39. What Is Active Learning? How Does It Reduce Data Annotation Costs?**

<details>
<summary>💡 Show Answer</summary>

Active learning is an intelligent iterative annotation strategy. Its core idea is that instead of passively receiving all annotated data, the model actively participates in selecting the "most valuable to annotate" samples, achieving maximum model performance improvement at minimum annotation cost.

**Basic Workflow**:

1. Train a base model on a small set of initial annotated data.
2. Apply the base model to a large volume of unannotated data and compute the "informativeness" or "uncertainty" of each sample.
3. Select a batch of samples with the highest informativeness and submit them to annotators for labeling.
4. Add the newly annotated data to the training set and retrain the model.
5. Repeat steps 2--4 until model performance reaches the target or the annotation budget is exhausted.

**Sample Selection Strategies**:

- **Uncertainty Sampling**: Prioritizes samples for which the model's predictions are most uncertain (e.g., samples with classification probabilities closest to 0.5). These samples lie near decision boundaries, and annotating them provides the greatest aid in determining boundaries.
- **Query by Committee**: Trains multiple distinct models (a committee) and selects samples with the highest prediction disagreement among committee members. High disagreement signifies that these samples contain information not yet mastered by the models.
- **Expected Model Change**: Selects samples that maximize the magnitude of model parameter updates---that is, samples with the highest "educational value" for the current model.
- **Diversity Sampling**: While selecting high-uncertainty samples, ensures the diversity of the selected sample set to prevent clustering in a localized region.

Studies demonstrate that active learning can typically achieve higher model performance than random sampling given the same amount of annotated data, or save over 50% of annotation costs while reaching identical performance. It is particularly valuable in high-cost scenarios such as medical image annotation.

</details>

**40. What Are Data Drift and Concept Drift? How Do They Affect Online Models?**

<details>
<summary>💡 Show Answer</summary>

**Data Drift (Covariate Shift)** refers to the statistical distribution of online input data $P(X)$ gradually deviating over time from the data distribution used during model training, while the input-to-output mapping $P(Y|X)$ itself remains unchanged. A classic example: user search term distributions changed drastically during the pandemic, causing recommendation models trained on pre-pandemic data to face completely unfamiliar input distributions.

**Concept Drift** refers to changes in the input-to-output mapping relationship $P(Y|X)$ itself while the input distribution remains unchanged---that is, for the same input, the "ground truth answer" changes over time. Classic examples: the sentiment polarity of certain words in sentiment analysis shifting following societal events; fraud patterns in financial risk control constantly evolving such that historical fraud features no longer apply.

**Impact on Online Models**:

- **Drop in Predictive Accuracy**: Model performance degrades on data outside the training distribution; however, without monitoring, this degradation is often slow and subtle, making it difficult to detect promptly.
- **Out-of-Distribution (OOD) Inference Risk**: The model may output overly confident yet erroneous predictions on inputs never seen during training (exhibiting overconfidence), which is especially dangerous in high-risk scenarios.
- **Detection and Response**: Monitor changes in input feature distributions via statistical tests (e.g., KS test, Population Stability Index); track changes in prediction distribution (whether predicted class ratios shift) as early warning signals; once drift is detected, trigger data re-collection, re-annotation, and model retraining pipelines.

</details>

**41. How to Monitor the Health Status of Data Pipelines in Production Environments?**

<details>
<summary>💡 Show Answer</summary>

![Production Data Pipeline health monitoring and anomaly detection dashboard.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig19_data_pipeline_monitoring.png)




Once an issue occurs in a production data pipeline (such as abnormal data volume, missing fields, or processing latency), it can lead to model training using incorrect data, or online inference services generating wrong predictions due to missing features, resulting in serious business impact. A comprehensive monitoring system should cover the following dimensions:

- **Data Volume Monitoring**: Set dynamic thresholds based on historical periods for the input/output data volume of each pipeline node. Sudden surges (potentially caused by data duplication or upstream error replay) or sudden drops (potentially caused by upstream stream interruption or abnormal filtering rules) should trigger alerts.
- **Schema Consistency Monitoring**: Automatically detect additions, deletions, and type changes of data fields. If schema changes in upstream systems are not synchronized in advance, downstream parsing failures or field misalignment will occur.
- **Data Quality Metric Monitoring**: Track missing rates, null rates, outlier proportions of key fields, and the distribution of values for enumerated fields, triggering alerts when thresholds are exceeded.
- **Pipeline Task Health Monitoring**: Record the execution duration, success/failure rate, and retry count of each task. A sudden spike in execution duration may indicate a surge in data volume or insufficient computing resources; an increase in failure rate requires immediate investigation.
- **End-to-End Latency Monitoring**: Monitor the end-to-end latency from data generation to availability for training/inference. Excessive latency causes models to use stale data, impacting latency-sensitive businesses.
- **Downstream Performance Feedback Monitoring**: Incorporate the online performance metrics of model inference (such as click-through rate, conversion rate, and user feedback) into indirect data quality monitoring as the ultimate measure of health.

At the tool level, common solutions include Great Expectations (a data quality assertion framework), Apache Airflow (pipeline scheduling and monitoring), and custom Prometheus + Grafana metric dashboards.

</details>

**42. How to Handle "Personally Identifiable Information" (PII) in LLM Training Data Preparation?**

<details>
<summary>💡 Show Answer</summary>

The handling of Personally Identifiable Information (PII) is a critical compliance step in LLM data engineering. Improper handling can cause the model to leak real personal information during generation, leading to legal risks under regulations such as GDPR and CCPA. A complete PII processing workflow includes:

**PII Identification**:

- **Rule/Regular Expression-based**: Write regular expressions for exact matching of fixed-format PII (such as phone numbers, ID numbers, bank card numbers, email addresses, and IP addresses), which is fast and has a low false-positive rate.
- **NER Model-based**: Use Named Entity Recognition models (such as BERT-based fine-tuned NER models) to identify variable-format PII (such as person names, organization names, and addresses), which can handle more complex context-dependent PII.
- **Hybrid Solution**: Combine rule-based methods with NER models to cover different types of PII, and verify identification accuracy through human sample auditing.

**PII Processing Strategies**:

- **Deletion**: Directly delete documents or paragraphs containing PII, suitable for scenarios with high PII density and low content value.
- **Redaction/Masking**: Replace PII with uniform placeholders (such as `[PHONE]`, `[NAME]`), preserving the semantic value of the remainder of the text. This is the most common handling approach.
- **Anonymization**: Replace real PII with fabricated values of consistent format (e.g., replacing real names with randomly generated pseudonyms), making the redacted text more natural and fluent without affecting the language model's ability to learn normal named entity usage.
- **Differential Privacy**: Add noise at the model training level (rather than the data level) to mathematically guarantee that the trained model cannot be used to infer PII of specific training samples. This is a stricter privacy protection measure, but incurs some loss of model utility.

</details>

**43. What Is Data Augmentation? What Are the Common Methods in Text and Multimodal Data Scenarios?**

<details>
<summary>💡 Show Answer</summary>

Data augmentation is a technique that generates additional training samples by applying transformations to existing data. Its core objective is to increase the quantity and diversity of training data without collecting new real-world data, thereby enhancing the model's generalization ability and robustness against input variations.

**Text Data Augmentation Methods**:

- **Synonym Replacement**: Randomly select non-key words in the text and replace them with synonyms from a dictionary (e.g., WordNet), introducing lexical diversity while preserving semantics.
- **Back Translation**: Translate the text into an intermediate language (e.g., Chinese $\to$ English) and then translate it back to the original language (English $\to$ Chinese), leveraging translation imprecision to obtain paraphrased versions with similar semantics but different expressions. This is one of the most stable augmentation methods in NLP.
- **Random Insertion/Deletion/Swap**: Randomly insert synonyms into text, delete non-key words, or swap the positions of adjacent words to introduce slight perturbations and enhance model robustness (EDA methods).
- **LLM Paraphrasing**: Use Large Language Models to perform diverse paraphrasing of text, generating semantically equivalent but phrased-differently versions. This is currently the highest-quality text augmentation method, capable of generating highly natural rewrites.
- **Mixup**: Perform linear interpolation on the embedding vectors of two samples to generate a "mixed" sample between them, training with the interpolated labels. It is a regularization technique that performs augmentation in the embedding space.

**Image Data Augmentation Methods**:

- **Geometric Transformations**: Random cropping, flipping (horizontal/vertical), rotation, scaling, affine transformations, etc., making the model invariant to changes in object position and orientation.
- **Color Transformations**: Adjust brightness, contrast, saturation, and hue (Color Jitter) to simulate lighting variations under different shooting conditions.
- **Random Erasing / CutOut**: Randomly mask partial regions of an image, forcing the model to learn to infer the whole from local information, improving robustness against occlusion.
- **Mixup / CutMix**: Blend two images (via linear interpolation or splicing) to generate mixed samples, serving as highly effective regularization augmentation methods in image classification tasks.

**Special Considerations for Multimodal Augmentation**: In multimodal data such as image-text pairs and video captions, augmentation must preserve the semantic alignment across different modalities. For instance, when applying image augmentations that significantly alter semantics (such as extreme color transformations), the original text description may no longer accurately match.

</details>

**44. What Is the Data-Level Manifestation of the Cold Start Problem, and How to Address It?**

<details>
<summary>💡 Show Answer</summary>

The core manifestation of the cold start problem at the data level is that during the early stages of launching a new business, exploring a new scenario, or iterating a new product, there is a lack of sufficient historical data to train models or construct evaluation sets, making it impossible to initiate the data-driven model development workflow. Specific manifestations include:

- **New Businesses / New Scenarios**: Complete absence of historical user behavior data, making it impossible to train models such as recommendation or ranking that rely on historical data.
- **New Domain Model Fine-Tuning**: Lack of annotated data in the target domain, rendering domain-adaptation fine-tuning impossible.
- **New Evaluation Benchmark Construction**: Lack of evaluation samples for the target capability, making it impossible to measure model performance on that capability.

**Mitigation Strategies**:

- **Transfer Learning and Domain Adaptation**: Use existing data from similar scenarios or general domains (even if the distributions do not fully match) as initial training data. Leverage the strong generalization capability of pre-trained LLMs to narrow the distribution gap, and fine-tune after accumulating a small amount of target domain data.
- **Synthetic Data Bootstrapping**: Use powerful generative models (such as GPT-4) to generate initial synthetic datasets based on descriptions of the target scenario, rapidly establishing a data foundation for training and evaluation to solve the "zero-data" bootstrap problem.
- **Few-Shot Learning**: Leverage the In-Context Learning capabilities of LLMs to accomplish new tasks with only a tiny number of examples (or even zero-shot), bridging the cold start phase.
- **Prioritize High-Quality Human Annotation**: In early stages with limited resources, prioritize high-quality human annotation for the most representative core scenarios to build a reliable "seed dataset", which then serves as the foundation for semi-automated expansion.
- **Progressive Data Accumulation**: Design proper data collection mechanisms (event tracking, user feedback collection) to gradually accumulate real data as the product operates. Formulate clear data accumulation milestones and model iteration plans to progressively replace synthetic data with real-world data.

</details>

**45. What Is Event Tracking/Logging in Data Collection, and What Should Be Considered When Designing It?**

<details>
<summary>💡 Show Answer</summary>

Event tracking (also known as behavior logging) is a data collection mechanism that embeds collection code into key interaction points of a product or system to automatically log user behaviors (such as clicks, dwell time, and search queries) or system operational states (such as API call parameters and response times). It serves as the foundational infrastructure for building a data flywheel.

**Key Considerations When Designing Event Tracking**:

- **Proactive Requirement Alignment to Prevent Missing Key Fields**: Event tracking design should be thoroughly aligned with data analysis and algorithm teams prior to product/feature development. Clarify all fields required for downstream analysis to prevent the need for re-releases when critical information is found unrecorded after launch. Retroactive additions of tracking can only cover future data and cannot remedy historical gaps.
- **Field Completeness and Self-Descriptiveness**: Each log entry should contain sufficient contextual information (such as User ID, Session ID, timestamp, page path, event type, and related entity IDs) to ensure the log can be understood independently without relying on join queries with other systems.
- **Version Management and Compatibility**: Schema changes in tracking (adding/removing fields, modifying semantics) must be version-controlled, with effective timestamps recorded for each version. This ensures that historical and new data can be correctly distinguished and compared during analysis, avoiding non-comparability of historical trends caused by schema drift.
- **Data Minimization Principle (Privacy by Design)**: Collect only data strictly necessary for business purposes, avoiding user privacy risks and compliance burdens from over-collection. Any PII that must be collected should be anonymized/redacted at the time of collection rather than storing raw sensitive data.
- **Data Quality Verification**: Perform end-to-end verification after tracking deployment to confirm accurate reporting (data counts and field values match expectations), and set up continuous monitoring alerts (e.g., for abnormal log volume or sudden spikes in field missing rates).
- **Sampling Strategies**: For high-frequency events (such as page impressions, which may reach billions daily), full logging cost is prohibitively high. A reasonable sampling strategy (e.g., 1% random sampling) should be designed, with real magnitudes restored during analysis.

</details>

**46. What Is Idempotency in Data Pipelines, and Why Is It Important?**

<details>
<summary>💡 Show Answer</summary>

Idempotency refers to the property where executing the same data processing task multiple times on the same input data yields completely identical output results, without creating duplicate data or introducing inconsistent states in the target storage due to repeated executions. This concept is borrowed from mathematics (idempotent operations) and distributed system design.

**Why Idempotency Is Crucial in Data Pipelines**:

- **Enables Safe Retries**: In distributed data processing tasks, mid-way failures due to network faults, machine reboots, or timeouts are the norm rather than the exception. Idempotency ensures that failed tasks or sub-tasks can be safely re-executed without worrying about duplicate writes of already-processed data, greatly simplifying error recovery logic.
- **Simplifies Pipeline Design**: Pipelines with idempotency do not need to maintain complex checkpointing states (tracking which data has been processed and which has not), reducing system complexity.
- **Guarantees Data Consistency**: In distributed environments, At-Least-Once message delivery semantics are common. Idempotency ensures that even if messages are delivered repeatedly, no duplicate data writes occur, effectively achieving Exactly-Once semantics.

**Common Methods for Achieving Idempotency**:

- **PrimaryKey-Based Upsert**: Use INSERT OR REPLACE (based on unique primary keys) when writing data, updating existing records and inserting new ones, naturally avoiding duplicate writes.
- **Unique Output Pathing**: Write processing outputs to paths named by input data identifiers (such as date partitions or batch IDs). Re-executions overwrite the same path rather than appending data.
- **Delete-Before-Write**: Before writing new results, delete (or clear) existing data within the same time partition/batch to guarantee output uniqueness.

</details>

**47. How Is "Distributed Deduplication" Implemented on Ultra-Large-Scale Corpora (Trillion-Token Scale) in Data Preprocessing Pipelines?**

<details>
<summary>💡 Show Answer</summary>

In pre-training corpus processing at the trillion-token scale, the number of documents can reach billions. The time complexity of traditional pairwise comparison deduplication is $O(N^2)$, which is entirely infeasible (even if each comparison takes only 1 nanosecond, pairwise comparison of $10^9$ documents would take approximately 15 years). The standard solution for large-scale distributed deduplication is as follows:

**Core Solution: MinHash + LSH + Distributed Computing Framework**

1. **Document Characterization**: Tokenize each document into character-level N-grams (typically $n=5$) to build an N-gram set representation. This step can be highly parallelized on Spark/MapReduce.
2. **MinHash Signature Generation**: Apply $k$ (typically 128--256) hash functions to each document's N-gram set to generate a fixed-length MinHash signature vector. This step is computed independently per document and is naturally parallel.
3. **LSH Banding**: Divide the signature vector into $b$ bands. The values within each band are concatenated and hashed again to a bucket ID. Each document is mapped to $b$ buckets; only document pairs sharing at least one bucket ID are considered candidate duplicate pairs. This step is realized via distributed GroupBy operations.
4. **Precise Candidate Pair Comparison**: Calculate the exact MinHash similarity (Hamming similarity of signature vectors) for document pairs falling into the same bucket, filtering out true duplicate pairs exceeding a threshold (typically 0.8 or 0.9).
5. **Connected Component Clustering**: Perform connected component analysis on the graph formed by duplicate pairs (where nodes are documents and edges are duplicate relationships). Retain one representative document per connected component and delete the rest. Spark GraphX or similar frameworks support large-scale graph computation.
6. **Deduplication Result Write-Back**: Broadcast the final set of retained document IDs back to the original dataset, filter out documents marked as duplicates, and output the deduplicated corpus.

The time complexity of the entire pipeline is reduced from $O(N^2)$ to approximately $O(N \log N)$. Processing in parallel across hundreds of compute nodes, deduplication of billions of documents can be completed within a few hours (the feasibility of this approach has been validated by practices such as RefinedWeb and RedPajama-v2).

</details>

**48. How Should the "Duplication Rate" in Pre-Training Data Be Controlled? What Problems Arise from Being Too High or Too Low?**

<details>
<summary>💡 Show Answer</summary>

Controlling the duplication rate in pre-training data is a problem requiring careful trade-offs; completely eliminating duplication is not the optimal strategy:

**Hazards of Excessively High Duplication Rate**:

- **Low Training Efficiency**: The model repeatedly computes gradients on content it has seen multiple times, wasting valuable compute resources. The effective training steps (amount of unique knowledge covered) become far lower than the actual training steps.
- **Overfitting and Memorization**: Highly repetitive content is "memorized verbatim" by the model, causing it to tend toward verbatim reproduction of training data during generation. This not only impairs diversity but also introduces copyright risks (the model may directly output duplicated passages protected by copyright).
- **Bias Amplification**: Sources with high duplication rates often represent specific viewpoints or styles. Excessively high duplication causes the biases of such sources to be amplified in the model, adversely affecting model fairness and balance.

**Issues with Excessively Low Duplication Rate (Over-Deduplication)**:

- **Loss of Valuable High-Quality Content**: Certain high-quality content (such as classic literature, authoritative textbooks, and widely cited formulations of scientific theorems) naturally has multiple copies on the Internet. Over-deduplication aggressively removes such content, thereby hurting data quality.
- **Loss of Knowledge Reinforcement**: Studies show that moderate repetition of high-quality content (2--4 times) helps the model better learn and memorize such knowledge; complete deduplication may actually impair mastery of core knowledge.
- **Processing Costs**: Pursuing an extremely low duplication rate requires more aggressive approximate deduplication thresholds, which may mistakenly remove documents that merely share similar topics but have distinct content.

**Balanced Strategies in Practice**:

- Adopt different deduplication strategies for different data quality tiers: apply strict deduplication (lower similarity threshold) to low-quality web data, while applying lenient deduplication or even allowing upsampling (reusing 2--4 times) for high-quality data (such as books and academic papers).
- After deduplication, explicitly control the usage frequency of high-quality data through a Data Mixture mechanism, shifting the decision power of "reuse" from the deduplication step to the sampling step to achieve finer-grained control.

</details>

**49. How to Design a Data Loading System Supporting "Dynamic Adjustment of Training Ratios by Data Source/Quality"?**

<details>
<summary>💡 Show Answer</summary>

A data loading system that supports dynamic training ratio adjustments is an essential component of LLM training infrastructure. When designing such a system, the following core elements must be considered:

**Data Organization Layer**:

- Store data separately according to source/quality level, with each data source serving as an independent dataset shard supporting individual indexing and access.
- Maintain metadata (total tokens, language, quality tags, source information) for each data source for ratio calculation and monitoring.
- Adopt a unified serialization format (e.g., MDS, WebDataset, Arrow) to support efficient random access and streaming reads.

**Weighted Sampling Layer**:

- Implement a Weighted Sampling mechanism to sample from each data source proportionally according to preset ratio weights to construct training batches.
- Support "sampling with replacement" (for repeating high-quality small datasets) and "sampling without replacement" (ensuring full traversal of large datasets), configurable per data source.
- Ratio parameters should support configuration files (YAML/JSON), allowing adjustments to data source proportions without code modifications.

**Dynamic Adjustment Support**:

- Support dynamic hot-reloading of ratios during training (e.g., triggered by configuration file refresh or semaphores), enabling adjustments without interrupting or restarting training.
- Support Curriculum Learning strategies: predefine scheduling plans where ratios change with training steps (e.g., steps 0--100K use general data, while steps 100K--200K gradually increase the proportion of specialized data).

**Reproducibility Assurance**:

- Fix random seeds for all random sampling operations and support exact reproduction of subsequent data ordering when resuming from any training step checkpoint.
- Record the actual data source distribution used in each training step for post-hoc analysis and experiment replication.

</details>

**50. What Is the "Tokenizer Consistency" Problem in Data Processing? Why Must the Same Tokenizer Be Used for Training Data Preprocessing and Inference?**

<details>
<summary>💡 Show Answer</summary>

![Byte Pair Encoding (BPE) subword tokenization iterative merging process.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig12_bpe_tokenization_flow.png)




A Tokenizer is a key component that converts raw text into Token ID sequences processable by models. Internally, it includes a vocabulary, tokenization rules (such as merge rules for BPE, WordPiece, SentencePiece), and special token definitions (such as `<|endoftext|>`, `<pad>`, `<bos>`). Tokenizer consistency requires that training data preprocessing and inference use the exact same Tokenizer version for the following reasons:

**Severe Problems Caused by Inconsistency**:

- **Token ID Mapping Misalignment**: Vocabularies in different Tokenizer versions may differ, meaning the same word might correspond to different Token IDs in different versions. If Token ID 1024 represents "apple" during preprocessing, but 1024 represents "banana" in the Tokenizer used during inference, all knowledge learned by the model becomes misaligned.
- **Inconsistent Tokenization Boundaries**: Different versions of BPE merge rules may cause the same text to be split into different subword sequences. The model learns Token relationships under one segmentation pattern during training, but encounters inputs with a different segmentation pattern during inference, preventing it from properly activating relevant knowledge.
- **Special Token Semantic Confusion**: If IDs for special tokens such as `<pad>`, `<bos>`, and `<eos>` are inconsistent, the model will fail to correctly recognize sequence boundaries and cannot stop generation at appropriate positions.

**Engineering Practices to Guarantee Consistency**:

- Treat the Tokenizer as a companion component to the model, version-controlling and distributing it alongside model weights (e.g., HuggingFace's `tokenizer.json` stored in the same directory as model weights).
- Record the Tokenizer version used in the data preprocessing pipeline, and automatically verify at training startup whether the Tokenizer used for preprocessing matches the currently configured Tokenizer.
- Avoid arbitrarily changing Tokenizers during model iterations. If a replacement is mandatory (e.g., expanding vocabulary to support new languages), all training data must be re-preprocessed, treating it as a major architectural change.

</details>

**51. How to Design a Data Processing Pipeline for RAG Systems to Handle Heterogeneous Document Formats (PDF, Word, Webpages, etc.)?**

<details>
<summary>💡 Show Answer</summary>

The data processing pipeline of a Retrieval-Augmented Generation (RAG) system needs to transform heterogeneous document formats into a knowledge base that can be efficiently retrieved and utilized. Design-wise, it is typically structured into the following layers:

1. **Format Parsing Layer**: Use specialized parsers tailored to different document formats:
    - PDF: Use tools such as PyMuPDF and pdfplumber to extract text while identifying and retaining heading hierarchies, table structures, and images. Scanned PDFs require OCR (such as Tesseract or PaddleOCR).
    - Word (.docx): Parse using python-docx, preserving paragraph styles (heading levels), tables, and embedded images.
    - Webpages (HTML): Use tools like BeautifulSoup and Trafilatura to extract body text while removing noise such as navigation bars and advertisements (Boilerplate Removal).
    - Structured Documents (Excel, CSV): Identify headers and data regions, converting them into natural language descriptions or Markdown tables.

2. **Normalization Layer**: Convert content from diverse sources into a unified intermediate representation format (such as Markdown or JSON with structural metadata), uniformly recording metadata such as document source, creation time, document type, and heading hierarchy to lay the foundation for subsequent processing and provenance tracking.

3. **Chunking Layer**:
    - Prioritize splitting along semantic boundaries (chapters, paragraphs, clauses) rather than fixed character counts to maintain semantic integrity.
    - Set an appropriate chunk overlap to prevent key information falling right on chunk boundaries from being truncated.
    - Perform secondary splitting on overly long paragraphs and consider merging short ones, controlling chunk lengths within the effective context window of the embedding model (typically 256--512 tokens).
    - Preserve position metadata within the original document for each chunk (chapter title, page number, clause number).

4. **Indexing Layer**:
    - Convert chunks into dense vectors using embedding models (such as text-embedding-3-small or BGE-M3) and store them in vector databases (Milvus, Qdrant, pgvector).
    - Simultaneously build keyword inverted indices (BM25) to support hybrid search, compensating for pure semantic search's limitations in exact term matching.
    - Retain original document citation information to support provenance display in retrieval results.

**Engineering Robustness Design**: Support incremental processing (adding/updating documents without re-indexing the entire corpus); single document parsing failure does not affect the overall workflow (error isolation and log recording); support synchronous removal of corresponding chunks from vector stores and inverted indices when documents are deleted.

</details>

**52. What Is the Data "Long-Tail Distribution" Problem? How to Address It When Building Training/Evaluation Datasets?**

<details>
<summary>💡 Show Answer</summary>

A long-tail distribution refers to a situation where a small number of categories/scenarios account for the vast majority of samples (the "head"), while a large number of categories/scenarios each have only a very small number of samples (the "long tail"), with the overall distribution exhibiting typical power law characteristics. This phenomenon is pervasive in natural language data (for example, word frequency distribution follows Zipf's law) and presents severe challenges in training data construction:

**Problems Caused by Long-Tail Distributions**:

- **Huge Performance Gap Between Head and Tail**: The model has sufficient training samples and performs excellently on high-frequency head categories (such as common Q&A and popular topics); on low-frequency long-tail categories (such as rare diseases, low-resource languages, and specific domain jargon), due to scarce samples, the model can hardly learn and performs very poorly.
- **Falsely High Evaluation Metrics**: If the evaluation dataset shares a similar distribution with the training set (also a long-tail distribution), the overall metrics will be dominated by head categories, potentially masking the model's severe shortcomings in long-tail scenarios.
- **Fairness Issues**: Long-tail distributions are often associated with minority groups and small linguistic communities; performance gaps may give rise to fairness concerns.

**Mitigation Strategies**:

- **Data Level**: Perform oversampling or resampling on long-tail samples; leverage strong models to generate targeted synthetic data for long-tail categories to supplement training samples; collaborate with data providers to perform targeted collection of real-world data for long-tail scenarios.
- **Training Level**: Introduce class-weighted loss functions to assign higher loss weights to long-tail categories, forcing the model to learn more earnestly on these categories; adopt curriculum learning strategies to increase the sampling ratio of long-tail data in later stages of training.
- **Evaluation Level**: Construct dedicated long-tail evaluation subsets to track as independent evaluation metrics, preventing long-tail performance degradation from being masked by overall metrics; report performance metrics separately across different frequency intervals (stratified evaluation).

</details>

**53. How to Design a Dataset "Split" (Train/Validation/Test Split) Strategy to Avoid Data Leakage?**

<details>
<summary>💡 Show Answer</summary>

Data leakage refers to the situation where information from the test set "leaks" into the training process through various channels, causing falsely high model performance on the test set that fails to truly reflect its generalization ability. A scientific dataset splitting strategy is the first line of defense against data leakage:

**Core Principles**:

- **Group-Based Split**: Identify natural grouping relationships in data—multiple records from the same user, multiple samples derived from the same document, or multiple turns in the same conversation. Splitting must be executed at the group level to ensure that all data within the same group belongs to the same dataset partition, preventing data from the same source from appearing in both the training set and test set simultaneously (within-group leakage is the most common source of leakage).
- **Time-Series Split by Chronological Order**: For data with temporal attributes (such as news, transaction records, and user behaviors), historical data must be used for training and future data for testing (forward chaining), rather than random shuffling and splitting. Random splitting allows the model to "see the future," yielding unrealistically high accuracy during testing.
- **Near-Duplicate Detection**: After completing the initial split, run near-duplicate detection (such as MinHash) between the training set and test set to identify and remove samples from the test set that are highly similar to those in the training set, preventing implicit leakage caused by content duplication.
- **Feature Temporal Alignment**: For samples involving aggregated statistical features (such as user historical averages or product sales trends), calculating test set features must strictly use data prior to the test timestamp, and must not use data during or after the test period.

**Special Considerations for the Validation Set**: The validation set is used for model selection and hyperparameter tuning. If extensive model adjustments are made based on validation set results, information from the validation set will indirectly leak into the model (known as "validation set overfitting"). It is recommended to keep a true holdout test set completely isolated from the development process, using it only once after the model is finalized.

</details>

**54. What Is the "Incremental Update" Mechanism in Data Processing? What Are Its Advantages and Challenges Compared to Full Rebuilding?**

<details>
<summary>💡 Show Answer</summary>

Incremental update refers to the process where, when data sources change (new records added, existing records modified, or records deleted), only the changed portion of the data is reprocessed and synchronized to the target storage, rather than reprocessing the full dataset from scratch.

**Advantages Compared to Full Rebuilding**:

- **Significantly Lower Processing Costs**: For massive datasets (such as pre-training corpora at the scale of hundreds of billions of tokens), full rebuilding may take days or even weeks. Incremental updates limit the scope of processing to changed data, making the cost proportional to the volume of changes rather than the total volume.
- **Higher Data Freshness**: Incremental updates support higher frequencies of data refreshing (elevated from "weekly" for full rebuilding to "hourly" or even "near real-time"), allowing knowledge bases or training datasets to reflect up-to-date information in a timely manner.
- **Smoother Resource Consumption**: Full rebuilding causes periodic peak consumption of computing resources, whereas incremental updates exhibit much smoother resource consumption, contributing to stable system operation.

**Core Challenges of Incremental Updates**:

- **Change Detection**: Reliable mechanisms are required to identify which data has changed (additions, modifications, deletions). Common solutions include timestamp-based change detection, CDC (Change Data Capture from database transaction logs), and document content hash comparison.
- **Propagation of Deletion Operations**: While additions and modifications are relatively straightforward to handle, deletion operations are particularly complex—deletion events must be propagated to all downstream storage systems (vector databases, inverted indexes, training datasets) to ensure deleted data is no longer utilized, which is especially critical in RAG systems (to avoid retrieving deprecated or invalid content).
- **Consistency Guarantees**: During incremental updates, if the system continues serving live traffic (online updates), intermediate states such as reading partially updated data must be handled. Versioned snapshots or blue-green deployments are typically employed to guarantee consistency of the external view.
- **Handling Dependencies**: If downstream processing steps depend on global statistics (e.g., deduplication requiring comparison across the entire document collection), incremental updates necessitate re-evaluating whether these statistics need updating to prevent partial updates from disrupting global consistency.

</details>

**55. In Data Engineering for Multilingual LLMs, How to Handle the Severe Imbalance in Data Volume Across Different Languages?**

<details>
<summary>💡 Show Answer</summary>

![Multilingual data re-balancing via temperature-based sampling.](https://img.pricenode.uk/llm_interview_questions/english/chapter3/fig20_multilingual_data_balancing.png)




One of the core challenges faced by multilingual large language models is the extreme data imbalance across languages: high-resource languages such as English and Chinese have trillions of tokens of data on the internet, whereas low-resource languages (such as Swahili or dialects of Bengali) may only have hundreds of millions of tokens or even less. Training directly according to the natural distribution leads to extremely poor model performance on low-resource languages, making true multilingual coverage unattainable.

**Data-Level Approaches**:

- **Oversampling Low-Resource Languages**: Increase the sampling frequency of low-resource language data in training batches via weighted sampling (far above its natural proportion in raw data). A trade-off is required for the oversampling multiplier: a multiplier that is too high leads to excessive repetition of low-resource data, risking overfitting. Temperature sampling is commonly combined: sampling probabilities for each language are calculated as $p_i \propto n_i^{1/T}$. When $T > 1$ (e.g., $T=5$), data volume disparities among languages can be smoothed out.
- **Machine Translation Data Augmentation**: Utilize high-quality machine translation systems (such as NLLB, DeepL) to translate high-resource language data into low-resource languages, rapidly expanding low-resource language corpora. Attention must be paid to the impact of translation quality on final performance, along with rigorous quality filtering of translated data.
- **Targeted Data Collection**: Collaborate with low-resource language communities, local media, and government agencies to acquire localized native corpora, including news, government documents, textbooks, and other high-quality texts, fundamentally increasing the volume of real data for low-resource languages.
- **Cross-Lingual Transfer Assistance**: Incorporate parallel corpora (bilingual parallel texts) into the dataset to help the model learn cross-lingual semantic alignment relationships, enabling knowledge acquired in high-resource languages to transfer to low-resource languages and improving overall multilingual capabilities.

**Evaluation and Monitoring**: Establish separate evaluation subsets for each target language, reporting performance on a per-language basis after each model iteration to prevent overall metrics from masking performance degradation in individual languages. Focus on tracking evaluation trends for low-resource languages to ensure the effectiveness of optimization measures.

</details>

**56. What Is a "Data Flywheel"? How to Build a Data Flywheel in AI Products?**

<details>
<summary>💡 Show Answer</summary>

A Data Flywheel is a powerful positive feedback loop growth mechanism in AI products: large-scale product usage generates more user interaction data $\to$ this data is used to improve the AI model $\to$ a better model yields a superior product experience $\to$ attracting more users $\to$ generating even more data. Once the flywheel is initiated and accelerates, the catch-up difficulty for latecomers increases exponentially, forming a formidable data moat.

**Core Elements of Building a Data Flywheel**:

1. **Design High-Signal-Density Data Collection Mechanisms**: Not all user behavior data carries equal training value. Priority should be given to capturing behaviors with explicit signals: user edits to model outputs (which explicitly express "what the correct answer should be"), explicit user feedback (thumbs up/down, satisfaction ratings), user selection behavior (which candidate was chosen among multiple options), and user abandonment behavior (generated content left unused, potentially indicating insufficient quality). Product interactions should be intentionally designed to elicit these high-value signals (e.g., ChatGPT's Regenerate button logs the signal of user dissatisfaction with the initial generation).

2. **Establish Data Processing and Annotation Pipelines**: Automatically transform raw collected user behavior data into formats suitable for training and evaluation, including filtering out low-quality signals (such as random clicks or malicious feedback), converting implicit signals into training samples (such as using user-edited versions as target outputs for SFT training), and automatically constructing preference pairs for RLHF training.

3. **Establish a Rapid Model Iteration and Validation Closed Loop**: The shorter the cycle from data collection to model improvement, the faster the flywheel spins. It is essential to establish: automated model training trigger mechanisms (automatically triggering training once data accumulates to a certain volume), robust A/B testing frameworks (verifying on real users whether the new model genuinely brings product improvements), and rapid online evaluation systems (quickly validating new model safety and quality prior to full rollout).

4. **Identify and Address Key Flywheel Constraints**: The rotation speed of the flywheel is constrained by its weakest link—whether that is data annotation speed, model training compute, or the traffic scale required for A/B testing. Current constraints must be identified so resources can be focused on breaking bottlenecks, rather than spending evenly across all stages.

</details>

**57. How to Design a Quality Evaluation and Filtering Pipeline for LLM Supervised Fine-Tuning (SFT) Datasets?**

<details>
<summary>💡 Show Answer</summary>

The quality of Supervised Fine-Tuning (SFT) data directly determines the upper bound of a model's capabilities after fine-tuning. "Small and refined" high-quality data is often superior to "large and mixed" low-quality data (a core finding of the LIMA paper: just 1,000 carefully curated samples can achieve exceptionally strong SFT performance). A complete quality evaluation and filtering workflow is structured as follows:

**Evaluation Dimensions**:

- **Instruction Quality**: Whether the instruction is clear and explicit (unambiguous), reasonable and executable (not exceeding model capability limits), and sufficiently difficult and diverse (avoiding overly simplistic or excessively concentrated task types).
- **Response Quality**: Whether the response is accurate (factual correctness), complete (fully addressing all requirements of the instruction), appropriately formatted (suitable length, clear structure, executable code), and matching expected style (e.g., avoiding verbosity or unnecessary disclaimers).
- **Safety**: Whether it contains harmful, discriminatory, or non-compliant content, and whether it complies with enterprise content policies.

**Multi-Tier Filtering Pipeline**:

1. **Rule-Based Filtering (Tier 1, Fast Coarse Screening)**: Remove obvious formatting errors (such as truncated responses or corrupted JSON formats), samples that are overly short or long (setting reasonable token count thresholds), samples containing known prohibited content (keyword blacklists), and samples with language mismatch between instruction and response.
2. **Automated Model Scoring (Tier 2, Fine-Grained Selection)**: Use strong models (such as GPT-4) or specially trained quality scoring models to rate samples from 1 to 5 across multiple dimensions, filtering out low-scoring samples (typically removing those with overall scores below a threshold). Simultaneously, check sample redundancy relative to existing training data to filter out highly similar samples.
3. **Diversity Assurance**: After quality filtering, perform distribution analysis on remaining data across dimensions such as task type, domain subject, and instruction length. If a certain type of sample is overly concentrated, balance the distribution via undersampling to ensure SFT data covers a sufficiently broad range of task types.
4. **Manual Sampling Audit (Tier 3, Quality Verification)**: Randomly sample a certain proportion from data passing automated filtering for manual auditing to evaluate precision (how many passing samples are truly high quality) and recall (how many high-quality samples were erroneously filtered out). Adjust automated filtering thresholds and rules based on audit findings.

</details>

**58. When Constructing Evaluation Datasets, How to Avoid "Evaluation Set Bias" Misleading the Model Optimization Direction?**

<details>
<summary>💡 Show Answer</summary>

The quality and fairness of an evaluation dataset form the bedrock of credibility for the entire model iteration process. If the evaluation set itself contains systematic bias, optimization based on it is very likely to cause the model to evolve in the wrong direction. The following aspects require close attention:

**Source and Distribution Representativeness**:

- Data sources for the evaluation set should be diversified, avoiding relying exclusively on a single source (e.g., solely Wikipedia or a single community forum), ensuring it represents real user demand distributions across diverse scenarios.
- Periodically audit the source distribution of the evaluation dataset, analyze overlap with training data (by running near-duplicate detection), and prune or replace highly overlapping samples from the evaluation set.
- Explicitly incorporate long-tail scenarios, hard examples, and edge cases into the evaluation dataset to prevent it from becoming overly "head-heavy," which would only reflect model performance on simple, routine problems.

**Annotation Bias Control**:

- Annotator subjective preferences systematically impact evaluation results (such as preferring longer responses, favoring specific writing styles, or exhibiting "position bias"—always favoring the first option); controlled experiments are required to identify and quantify these biases.
- Conduct multi-annotator labeling for evaluation samples and calculate inter-annotator agreement; low-agreement samples (high divergence among annotators) should be revised or removed to prevent introducing random noise.
- When designing evaluation tasks, minimize dimensions reliant on subjective judgment and maximize objectively verifiable dimensions (such as factual correctness, code executability, and format compliance).

**Preventing "Evaluation Set Overfitting"**:

- Keep a true holdout test set completely isolated from the development team, using it only once after the model is finalized to prevent implicit leakage caused by repeated adjustments based on test set results.
- Periodically rotate the evaluation dataset (introducing new samples and retiring old samples used multiple times) to prevent models and development teams from developing "adaptation" to a fixed evaluation set.
- Cross-validate using multiple evaluation sets from different sources and styles. If model performance improves on one evaluation set but regresses on others, it indicates potential issues with the optimization direction.
- The evaluation dataset size must be sufficiently large to guarantee statistical significance (capable of distinguishing genuine performance differences from random fluctuations); conclusions drawn from small-scale evaluation sets should be treated with caution.

</details>

**59. What Is the "Schema Evolution" Problem in Data Processing? How to Design Data Storage Solutions Supporting Schema Evolution?**

<details>
<summary>💡 Show Answer</summary>

Schema Evolution refers to the requirement where, as business needs evolve, the structure of a dataset (field definitions, data types, constraints) must continuously adapt—adding fields, removing deprecated fields, or modifying field types or semantics—while historical data remains stored under old schemas, requiring data under old and new schemas to coexist and be correctly processed within the same system.

**Typical Problems Caused by Schema Evolution**:

- **Read Failures**: New code attempts to parse old-format data and throws errors due to missing newly added required fields.
- **Semantic Misalignment**: Field names remain identical but their underlying semantics changed during a revision, causing old data to be misinterpreted by new code.
- **Type Incompatibility**: A field changes from a string type to an integer type, making string values in historical data unconvertible.

**Storage Solution Design Supporting Schema Evolution**:

- **Select Serialization Formats with Native Support for Schema Evolution**:
    - **Apache Avro**: The schema is stored alongside data files, supporting reading old data according to a new schema (handling missing fields via field mappings and default values).
    - **Protocol Buffers (Protobuf)**: Fields are identified by numbers rather than names; newly added fields are defined as optional with default values. Old code ignores unknown fields, while new code uses default values for missing fields, achieving inherent forward and backward compatibility.
    - **Apache Parquet**: A columnar storage format supporting new columns (read as null when missing from old data files), unifying different schema versions at read time via Schema Merge.
- **Schema Evolution Rule Constraints**: Establish schema modification guidelines—allow adding fields (default values must be set), allow changing required fields to optional, prohibit directly deleting fields (mark as deprecated and retain for a period before removal), and prohibit modifying data types of existing fields (if modification is necessary, add a new field and deprecate the old one).
- **Schema Registry**: Introduce a centralized Schema Registry (such as Confluent Schema Registry) to manage schema version histories across all datasets, validating schema compatibility on write and retrieving correct schemas by version number on read for parsing.
- **Data Migration Strategies**: For breaking changes (unresolvable via compatibility rules), design data migration scripts to convert historical data into the new format, performing data integrity validation before and after migration to ensure zero data loss.

</details>

**60. How to Design a Training Data Preparation Pipeline Supporting "Data Reproducibility"?**

<details>
<summary>💡 Show Answer</summary>

Data reproducibility is a vital guarantee of AI engineering quality, referring to the ability to fully reproduce identical training datasets given the same source data and processing code. Achieving data reproducibility requires systematically eliminating non-determinism across all stages of pipeline design:

**Pipeline Code Enforcement and Version Control**:

- All data processing steps (cleaning rules, filtering thresholds, sampling strategies, tokenizer configurations) must be implemented as code and committed to Git version control; manual ad-hoc operations are strictly prohibited (as manual operations cannot be reproduced).
- For every training run, log the Git commit hash of the data processing code used, archiving it alongside model weights and experiment logs.

**Randomness Control**:

- All operations involving randomness (data sampling, shuffling, random seed-dependent data augmentation) must fix random seeds, explicitly logging seed values as configuration parameters.
- In distributed processing, ensure random seed propagation mechanisms maintain consistent partition assignments and processing orders across multiple runs.

**Source Data Version Locking**:

- Create snapshots or version locks for raw input datasets, recording exact versions of data sources (URL + download timestamp, database snapshot version, etc.) to prevent reproduction failures caused by upstream data updates.
- Calculate and record hashes/checksums of raw datasets, verifying input data integrity and consistency first when reproducing.

**Output Dataset Fingerprint Logging**:

- Compute dataset-level fingerprints for the final generated training dataset (such as MD5 hashes of shard files, total sample count, and token statistics), archiving them as unique dataset identifiers.
- During reproduction, compare final generated dataset fingerprints with historical records to verify reproduction success.

**Environment Freezing**:

- Use containerization (Docker) to freeze execution environments for data processing (Python version, dependency library versions), preventing behavioral discrepancies caused by environmental differences (e.g., different tokenizer library versions producing distinct tokenizations for identical text).

</details>

**61. What Is the "Diversity Collapse" Problem in Synthetic Data Generation? How Can It Be Mitigated?**

<details>
<summary>💡 Show Answer</summary>

Diversity collapse refers to the phenomenon where, when a single model is extensively used to iteratively generate synthetic data, the model's inherent output distribution is relatively concentrated (particularly pronounced at low sampling temperatures, as the model tends to produce statistically most probable "standard answers"). Consequently, the generated data exhibits convergent styles, clustered topics, and monotonous expression patterns, lacking the rich diversity present in real corpora. When training new models on such data, downstream models inherit this monotony, limiting both generative capabilities and knowledge coverage. In multi-generational iteration scenarios (where a generative model trains a new model, which in turn generates data to train the next generation), diversity collapse intensifies generation by generation, ultimately culminating in "model collapse."

**Methods to Mitigate Diversity Collapse**:

- **Multi-Model Generation**: Use multiple models with differing architectures, training data, or model sizes to generate data. Since distinct models exhibit different output preferences, blending them yields richer diversity.
- **Increasing Sampling Temperature**: Apply a higher temperature (e.g., 0.8--1.2) during generation to encourage the model to sample more diverse tokens, at the cost of potentially producing less fluent content that requires subsequent quality filtering.
- **Diversified Prompt Templates**: Design a broad array of prompt templates with varying styles and phrasing, applying different templates to the same topic/task to drive output diversity through input diversity.
- **Constraint-Guided Prompting**: Explicitly specify diverse constraints in prompts (such as "use a colloquial tone," "assume the reader is an expert," or "include counterexamples in the response"), guiding the model to generate diverse outputs covering different scenarios.
- **Diversity Metrics and Proactive Supplementation**: Quantitatively evaluate dataset diversity (e.g., embedding-based clustering analysis to observe cluster size distributions), identifying under-represented regions and targeting the collection/generation of scarce data types.
- **Continuous Real Data Injection**: Mix a certain proportion of real-world data into synthetic data to anchor and calibrate synthetic distributions using the natural diversity of real data, preventing cumulative drift across multi-generational iterations.

</details>

**62. In Data Engineering Pipelines, How to Balance "Data Processing Flexibility" with "System Stability/Maintainability"?**

<details>
<summary>💡 Show Answer</summary>

In data engineering practice, a natural tension exists between flexibility and stability: overly flexible systems are difficult to maintain and suffer from poor stability, whereas overly rigid systems cannot adapt to rapidly changing business needs. The key to resolving this contradiction lies in a clear layered architecture design:

**Layered Architecture**:

- **Stable Core Layer**: Encapsulate processing steps with high generality and low modification frequency (such as text decoding, basic format parsing, standardized cleaning, and vector index writing) into well-tested, standardized components that expose stable API contracts. Changes to the core layer require strict review and testing processes, with low change frequency (on a monthly basis).
- **Flexible Extension Layer**: Implement customized logic tailored to specific data sources or tasks (such as custom field parsing rules for a specific data source or domain-specific filtering rules) as pluggable extension modules. These are independently developed and tested, integrating into the core layer via configuration files or plugin mechanisms, allowing rapid iteration without modifying core code (on a daily/weekly basis).

**Configuration-Driven Rather Than Code-Driven**:

- Externalize volatile parameters (filtering thresholds, sampling ratios, field mapping rules) into configuration files. Business teams adjust behaviors by modifying configurations rather than code, reducing change risks (configuration changes are easier to review and roll back than code changes).
- Support configuration versioning and A/B testing (comparing results of different data processing parameters).

		**Testing System Guarantees**:

- Core layer components require complete unit and integration test coverage; every modification must pass tests before deployment.
- Testing bars for extension modules can be appropriately relaxed, but basic data quality validation is required (e.g., verifying whether processed data volume and key field distributions meet expectations).
- Establish end-to-end data quality benchmarks that run automatically after every pipeline change, comparing processing results against benchmarks to prevent unintended quality regression.

</details>

**63. How to Design Data Processing Pipelines for "Long Documents" (e.g., Contracts, Regulations) to Support Downstream Retrieval and Comprehension Tasks?**

<details>
<summary>💡 Show Answer</summary>

Data processing for long documents (such as contracts, laws and regulations, technical specifications, and academic papers) faces unique challenges: complex internal hierarchical structures and cross-reference relationships, high content density, and abundant domain-specific terminology. A single retrieval often requires integrating information from different parts of the document. Processing workflows tailored for such documents need dedicated design extensions built upon standard RAG pipelines:

		**Parsing Phase — Preserving Structural Information**:

- While extracting text, identify and preserve the hierarchical structure of the document (chapter/section/article/clause numbers, heading levels) to build a Document Tree representation.
- Identify and extract tables (converting table contents into natural language descriptions or structured JSON to prevent tables from being broken apart), definition lists (definitions of technical terms are often critical for understanding subsequent content), and cross-reference relationships (such as "refer to Chapter 3, Article 2").
- For legal and regulatory documents, identify and annotate semantic attributes such as the legal effect hierarchy of clauses (mandatory clauses vs. advisory clauses).

		**Chunking Phase — Prioritizing Semantic Boundaries**:

- Strictly partition text along document semantic boundaries (article $\to$ clause $\to$ item) rather than by fixed character counts, ensuring the semantic completeness of each chunk (preventing individual clauses from being truncated).
- For overly short clauses (such as a single-sentence provision), merge them with parent headings or adjacent clauses during chunking to guarantee semantic completeness.
- Implement Contextual Chunking: automatically prepend the title and summary of the parent section to each chunk as a prefix, ensuring that the chunk can still be correctly understood when detached from the full text.

		**Metadata Annotation — Supporting Precise Retrieval**:

- Attach rich structured metadata to each chunk: document source, effective date, clause identifier (e.g., "Chapter 3, Article 2, Clause 1"), parent section title, and clause type (definition clause, obligation clause, liability clause, etc.).
- Extract and store document-level summaries (overall content overview) and section-level summaries (core contents of each section) to support a hierarchical retrieval strategy of "summary first, details second."

		**Indexing Phase — Supporting Hybrid Retrieval**:

- In addition to semantic vector indexes, build precise retrieval indexes based on structured fields such as clause numbers, keywords, and dates, supporting the hybrid execution of structured queries (e.g., "pinpoint Chapter 3, Article 2") and semantic retrieval.
- Build multi-granularity indexes for the same clause (article-level, clause-level, item-level) to accommodate retrieval demands at different precisions.
- Preserve the reference relationship graph of the original document so that when a clause is retrieved, other clauses referenced by it can be automatically linked (cross-reference expansion), providing complete context for downstream comprehension tasks.

</details>

