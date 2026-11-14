# 7.9 RAG Evaluation (RAGAS, Faithfulness, Relevancy) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 7: RAG (Retrieval-Augmented Generation) · 19 questions

**1. What Metrics Are Used to Evaluate RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

Evaluation of RAG systems needs to cover three levels: retrieval, generation, and system level, as a single metric cannot comprehensively measure system quality:

**Retrieval-Level Metrics** (Measuring "whether the retrieval is correct and complete"):
- **Hit Rate@k**: The proportion of queries where at least one correct document is present in the top-k results; it is the most fundamental recall metric;
- **MRR (Mean Reciprocal Rank)**: The mean of the reciprocal rank of the first correct document, measuring precise ranking ability;
- **NDCG@k**: Normalized Discounted Cumulative Gain considering both relevance level and ranking order, representing the most comprehensive retrieval quality metric;
- **Context Precision / Context Recall**: Context Precision and Context Recall in the RAGAS framework, which can be computed automatically without human-annotated ground-truth documents.

**Generation-Level Metrics** (Measuring "whether the answer is accurate and trustworthy"):
- **Faithfulness**: Whether statements in the answer are supported by retrieved documents, measuring the degree of hallucination;
- **Answer Relevancy**: Whether the answer truly addresses the user query;
- **Answer Correctness**: Consistency with human-annotated standard answers;
- **BLEU/ROUGE**: N-gram overlap rate with reference answers, suitable for evaluation scenarios with ground-truth answers (though unfriendly to synonymous expressions).

**System-Level Metrics** (Measuring "engineering quality"):
- **End-to-End Latency** (P50/P99): Response speed perceived by users;
- **Cost per Query**: Comprehensive cost of Embedding + vector retrieval + LLM API;
- **System Availability** (SLA): Service stability metrics;
- **Customer Satisfaction** (CSAT): The ultimate business metric, serving as a comprehensive reflection of all technical metrics.

Recommended tool: The RAGAS framework can automatically compute the four core metrics—Faithfulness, Answer Relevancy, Context Precision, and Context Recall—and is currently the most popular RAG evaluation tool.





![The RAGAS evaluation triad: Faithfulness, Answer Relevance, and Context Relevance. (RAGAS评估三元组：忠实度、回答相关度与上下文相关度)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig22_rag_evaluation_triad.png)

</details>

**2. What Are the Main Performance Metrics of RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

**Retrieval Phase Metrics**:
1. Context Precision: The proportion of truly relevant documents among all retrieved documents;
2. Context Recall: The proportion of relevant documents retrieved out of all documents that should have been retrieved;
3. MRR (Mean Reciprocal Rank): The mean of the reciprocal rank of the most relevant document.

**Generation Phase Metrics**:
1. Faithfulness: Whether the answer is entirely based on retrieved documents without ungrounded additions;
2. Answer Relevancy: Whether the answer truly addresses the user question;
3. Answer Correctness: Consistency with standard answers.

**Overall System Metrics**:
1. End-to-end latency;
2. Cost per query (token + vector retrieval);
3. Customer satisfaction.

</details>

**3. How to Independently Evaluate the Retrieval Component?**

<details>
<summary>💡 Show Answer</summary>

Independently evaluating the retrieval component (decoupled from the generation phase) allows for accurately determining whether system issues lie in retrieval or generation, avoiding mutual interference between the two components that complicates diagnosis. The evaluation methods are as follows:

**Building a Retrieval Evaluation Dataset**:
- Prepare several test queries, and manually annotate the corresponding "golden documents" for each query—that is, a list of document IDs containing the information needed to answer the question;
- Annotations can be binary relevance (only these few documents) or graded relevance (highly relevant / partially relevant / irrelevant).

**Core Evaluation Metrics**:
1. **Hit Rate@k (Hit Rate)**: The proportion of queries whose top-k retrieval results contain at least one golden document. Simple and intuitive, it is the most commonly used baseline metric. Common settings: Hit Rate@1, @5, @10;
2. **MRR (Mean Reciprocal Rank)**: The reciprocal rank of the first golden document appearing in the retrieval results, averaged across all queries. Measures how quickly the system can rank correct documents near the top;
3. **NDCG@k (Normalized Discounted Cumulative Gain)**: A comprehensive metric considering both relevance level (degree of relevance for different documents) and ranking position (higher ranks carry higher weight), representing the most thorough yet complex retrieval evaluation metric;
4. **Recall@k**: The proportion of golden documents recalled in top-k out of all golden documents, measuring the ability not to miss key documents (especially important when synthesis across multiple documents is required);
5. **Precision@k**: The proportion of relevant documents in top-k, measuring the "purity" of retrieval results, i.e., the noise ratio.

**Evaluation Workflow**: Run the retrieval system on the evaluation dataset, obtain top-k results for each query, calculate the above metrics against golden documents, compare with baseline methods (BM25, different Embedding models), and evaluate the actual performance gains from various optimization techniques (Reranking, hybrid retrieval).

</details>

**4. What Is the RAGAS Framework? What Evaluation Metrics Does It Include?**

<details>
<summary>💡 Show Answer</summary>

RAGAS (Retrieval Augmented Generation Assessment) is an automated evaluation framework designed specifically to assess the quality of RAG systems, utilizing LLM-as-a-judge to score various dimensions. Core metrics:
1. **Faithfulness**: Whether every statement in the generated answer is supported by the retrieved context (no hallucinations);
2. **Answer Relevancy**: Whether the answer truly addresses the user query;
3. **Context Precision**: The proportion of retrieved context that contributes to generating the answer;
4. **Context Recall**: The proportion of information required to generate the correct answer that is present in the retrieved context;
5. **Answer Correctness** (Optional): The similarity between the generated answer and the standard answer.
RAGAS does not require human annotation and can run automatically at scale, making it one of the most popular RAG evaluation frameworks available today.

</details>

**5. What Do Faithfulness and Answer Relevancy Measure Respectively?**

<details>
<summary>💡 Show Answer</summary>

**Faithfulness**: Measures whether each factual statement in the generated answer can be substantiated within the retrieved context, assessing whether the answer is "fabricated out of thin air". Scoring method: Decompose the answer into several statements, determine if each statement is supported by context, and the supported ratio serves as the faithfulness score. A low score indicates that the LLM added hallucinated content beyond the retrieved context.

**Answer Relevancy**: Measures whether the generated answer truly answers the user's question, rather than missing the point or answering only part of the question. Scoring method: Reverse-generate potential questions based on the answer, and calculate the semantic similarity between these reverse-generated questions and the original question. A low score indicates that retrieved documents are not directly relevant to the question, or the LLM answered using relevant but imprecise information.

</details>

**6. What Is the Difference Between Context Precision and Context Recall?**

<details>
<summary>💡 Show Answer</summary>

**Context Precision**: The proportion of all retrieved context (chunks) that actually contributes to generating the correct answer. It measures "how much of the retrieved content is useful", i.e., the noise ratio. A low score indicates that retrieval recalled too many irrelevant chunks, introducing noise for the LLM.

**Context Recall**: The proportion of all necessary information required to generate the correct answer that appears in the retrieved context. It measures "how much of the necessary information was retrieved". A low score indicates that retrieval missed critical information, resulting in an incomplete answer.

Trade-off between the two: Increasing the value of $k$ typically increases Recall but lowers Precision (by introducing more irrelevant content); Reranking can improve both simultaneously.

</details>

**7. How to Comprehensively Evaluate the Performance of a RAG System? Please Propose Evaluation Metrics Separately for the Retrieval and Generation Phases.**

<details>
<summary>💡 Show Answer</summary>

Comprehensively evaluating a RAG system requires decoupling it into two independent yet interconnected phases: retrieval and generation.

**Phase 1: Retrieval Performance Evaluation**, assessing whether the retriever can find content "correctly and completely":
1. **Context Precision**: Measures how many of the retrieved documents are genuinely relevant to the query, reflecting the signal-to-noise ratio of retrieval results.
2. **Context Recall**: Measures what proportion of all relevant documents were successfully retrieved by the retriever, reflecting the completeness of information search.
3. **Hit Rate**: Whether the retrieved documents contain at least one relevant document; serves as a basic baseline ("passing mark") metric.
4. **MRR (Mean Reciprocal Rank)**: The mean of the reciprocal rank of the first relevant document, measuring the speed of finding the first correct answer.
5. **nDCG@k (Normalized Discounted Cumulative Gain)**: One of the most comprehensive and commonly used metrics, considering both the relevance level of retrieval results and their position in the result list.

**Phase 2: Generation Performance Evaluation**, assessing whether the LLM, given the context, can generate "faithful, accurate, and useful" answers:
1. **Faithfulness / Groundedness**: Is the generated answer strictly based on the provided context? Are there fabrications or hallucinations?
2. **Answer Relevancy**: Does the generated answer directly and clearly answer the user's original question?
3. **Answer Correctness**: Is the information in the answer factually accurate (compared against a "gold standard" answer, or fact-checked by domain experts)?
Automated evaluation frameworks: Frameworks such as RAGAS, ARES, and TruLens utilize the LLM-as-a-Judge concept to compute the above metrics automatically, greatly improving evaluation efficiency.

</details>

**8. How to Construct an Evaluation Dataset (Golden Dataset) for a RAG System?**

<details>
<summary>💡 Show Answer</summary>

1. **Extraction from User Logs**: Collect real user queries and perform human annotation on high-frequency and representative queries (correct answer + document containing the answer);
2. **Expert Design**: Invite domain experts to design questions based on the knowledge base content, covering different difficulty levels and types (factual / reasoning / comprehensive);
3. **Synthetic Generation**: Use LLMs to generate QA pairs from knowledge base documents, followed by manual sampling and quality verification;
4. **Edge Case Design**: Specifically construct unanswerable questions for the knowledge base, questions requiring multi-hop reasoning, ambiguous queries, etc.;
5. **Annotated Content**: Each test case contains: query, standard answer, answer source document ID (used for evaluating Context Recall);
6. **Scale Recommendations**: Minimum viable scale of 100 entries covering major query types; production-grade evaluation dataset of 500+ entries;
7. **Periodic Updates**: Expand the test suite periodically as the knowledge base and user requirements evolve.

</details>

**9. How to Use LLM-as-a-Judge to Evaluate the Quality of RAG Outputs?**

<details>
<summary>💡 Show Answer</summary>

1. **Evaluation Dimensions**: Faithfulness, relevancy, completeness, conciseness, which can use the RAGAS framework or custom prompts;
2. **Evaluation Prompt Design**: Provide scoring rubrics (1–5 scale) and reference examples, allowing the LLM to score each dimension and provide rationales;
3. **Reference Answer Comparison** (when ground truth is available): Have the LLM compare the consistency between the generated answer and the standard answer;
4. **Multiple Sampling**: Run 3–5 iterations per input and average scores to reduce randomness;
5. **Bias Control**: Randomize answer presentation order to prevent position bias; use different judge LLMs for cross-comparison to prevent self-enhancement bias;
6. **Human Calibration**: Periodically sample outputs and compare with human scores to verify the consistency of the LLM judge (correlation coefficient $> 0.8$ is considered reliable);
7. **Cost Control**: Perform stratified sampling instead of full evaluations every time.

</details>

**10. How to Evaluate the Overall Performance of a RAG System?**

<details>
<summary>💡 Show Answer</summary>

Evaluation can be conducted in layers: retrieval layer (metrics such as recall, precision, MRR, etc.), generation layer (answer accuracy and faithfulness based on retrieved context), and end-to-end layer (user satisfaction, task completion rate), while constructing an evaluation dataset with annotated ground-truth answers. The complete evaluation framework is as follows:

**Layer 1: Retrieval Quality Evaluation** (offline evaluation, requires annotated data):
- **Hit Rate@k**: The proportion of top-k retrieval results that contain the source document of the correct answer, measuring whether recall is sufficient;
- **MRR (Mean Reciprocal Rank)**: The mean of the reciprocal ranks of the correct document, measuring precise ranking quality;
- **NDCG@k**: A comprehensive metric considering both relevance levels and rankings;
- **Context Precision / Context Recall** (RAGAS metrics): Precision and recall of the retrieved context.

**Layer 2: Generation Quality Evaluation** (can be automated using LLM-as-judge):
- **Faithfulness**: Whether the answer is faithful to the retrieved context without hallucinations;
- **Answer Relevancy**: Whether the answer directly addresses the question;
- **Answer Correctness**: Consistency with the ground-truth answer (requires manually annotated reference answers).

**Layer 3: End-to-End Business Evaluation** (final metrics):
- User satisfaction (CSAT score, thumbs-up rate);
- Task completion rate (whether user questions are satisfactorily resolved);
- Human handoff rate (the proportion of requests transferred to human agents when the system cannot handle them);
- A/B testing (direct business metrics comparing against old versions or competing products).

**Implementation Tools**: The RAGAS framework can automate the calculation of most metrics in Layers 1 and 2 without requiring massive manual annotation, making it the most recommended RAG evaluation tool currently available.





![End-to-end evaluation pipeline for RAG systems. (RAG系统的端到端性能评估体系流程)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig23_overall_performance_eval.png)

</details>

**11. How to Individually Isolate and Locate Issues in the Retrieval and Generation Stages of a RAG System?**

<details>
<summary>💡 Show Answer</summary>

**Retrieval Stage Issue Localization**:
1. Check Context Recall: Was all necessary information retrieved? Low score $\rightarrow$ retrieval strategy issue;
2. Check Context Precision: Was a large amount of irrelevant content introduced? Low score $\rightarrow$ retrieval noise issue;
3. Directly inspect top-k results: Manually review whether the retrieved chunks are relevant;
4. Analyze query types: Which category of query has poor retrieval performance (multi-hop / rare words / long queries)?

**Generation Stage Issue Localization**:
1. Check Faithfulness: Does the answer exceed the retrieved context? Low score $\rightarrow$ generation hallucination issue;
2. Check Answer Relevancy: Does the answer miss the point of the question? Low score $\rightarrow$ prompt issue;
3. Fix retrieval results and test generation quality across different LLMs to eliminate retrieval interference;
4. Analyze quality differences between long and short answers to determine whether it is a lost-in-the-middle issue.

</details>

**12. How to Systematically Troubleshoot the Issue of "Correct Document Retrieved, but Answer Is Still Incorrect"?**

<details>
<summary>💡 Show Answer</summary>

1. **Faithfulness Check**: Is the incorrect information in the answer present in the retrieved documents? If yes, it indicates that the LLM misinterpreted the document; if no, it indicates that the LLM added hallucinations beyond the documents;
2. **Context Position Issue**: Is the correct document ranked near the end of the context passed to the LLM (Lost in the Middle)? Try moving the correct document to the front;
3. **Document Quality Issue**: Does the retrieved document have complex formatting (embedded tables, nested code) causing parsing errors by the LLM? Try preprocessing such documents;
4. **Context Contradiction**: Do multiple provided documents contain conflicting information, leading the LLM to select the wrong source? Check if contradictory documents exist;
5. **Prompt Issue**: Are the prompt instructions clear enough? Does it explicitly require "answering based only on the provided documents"?
6. **Model Capacity Limitation**: Does the question require reasoning abilities beyond the current LLM (complex numerical calculations, multi-step logical reasoning)?

</details>

**13. How Should You Debug a RAG System When Issues Arise?**

<details>
<summary>💡 Show Answer</summary>

Step-by-step troubleshooting: First check whether the original document parsing and chunking are correct $\rightarrow$ check whether the retrieval results contain the chunk with the correct answer $\rightarrow$ check the final prompt sent to the model $\rightarrow$ check the model's generation result based on that prompt, isolating the issue layer by layer. The complete debug workflow:

1. **Step 1: Document Processing Layer Inspection**
    - Are original documents correctly parsed? (Check whether PDF text extraction has garbled characters or formatting errors)
    - Is chunking reasonable? (Check whether key information is split across the boundary of two chunks)
    - Are embeddings successfully generated? (Check vector dimensions and whether there are anomalous values)
2. **Step 2: Retrieval Layer Inspection**
    - Directly query the vector database to check top-k results—is the chunk containing the correct answer within top-k? If not, the issue lies in retrieval;
    - Check the similarity score distribution—is the score gap between top-1 and top-5 reasonable?
    - Try queries with different rephrasings and observe whether retrieval results remain stable.
3. **Step 3: Context Construction Layer Inspection**
    - View the actual full prompt passed to the LLM (enabling prompt logging is recommended);
    - Check whether post-reranking order places the most relevant chunks at the front;
    - Check whether Contextual Compression accidentally deleted key information.
4. **Step 4: Generation Layer Inspection**
    - Pass the chunk containing the correct answer directly as ideal context to the LLM and observe generation results—if still incorrect, the issue lies in prompt design or LLM capabilities;
    - Check whether the LLM operates correctly under System Prompt constraints;
    - Compare performance across different LLMs (GPT-4 vs. Claude) to determine if it is a model capability issue.
5. **Tool Assistance**: Use observability tools such as Langfuse and LangSmith to log the complete trace of each request (retrieval results, prompt, output), supporting offline debugging of historical cases and significantly improving troubleshooting efficiency.

</details>

**14. How to Systematically Locate Root Causes of Issues in RAG?**

<details>
<summary>💡 Show Answer</summary>

Establish a reproducible case collection, record the triplet of "retrieval results", "final prompt", and "model output" for each failure case, and categorize and analyze whether the issues are primarily distributed among three categories of root causes: retrieval miss, retrieval noise, or generation failure. The systematic root-cause localization workflow:

1. **Establish a Failure Case Library**: Collect failure cases from production logs or test evaluations, recording for each case: user query, retrieved top-k chunks (with similarity scores), full prompt finally passed to the LLM, LLM output, and expected ground-truth answer.
2. **Three-Layer Categorized Diagnosis**:
    - **Retrieval Miss**: The top-k results do not contain the required information for the correct answer $\rightarrow$ issue on the retrieval side (chunking, embedding model, k value);
    - **Retrieval Noise**: The top-k results contain correct information, but also a large amount of irrelevant content $\rightarrow$ issue in reranking or k value being too large;
    - **Generation Failure**: The top-k results contain correct information, but the LLM answer is incorrect $\rightarrow$ issue in prompt design, Lost in the Middle, or LLM capabilities.
3. **Quantify Distribution**: Calculate the proportion of the three root cause types among failure cases and prioritize resolving the type with the highest percentage.
4. **Sub-Cause Breakdown**: Further subdivide each root cause (Retrieval Miss $\rightarrow$ chunking granularity issues / embedding incompatibility / k value too small / inappropriate query phrasing; Generation Failure $\rightarrow$ Lost in the Middle / insufficiently strong prompt instructions / context contradiction / LLM capacity limitations).
5. **Controlled Variable Verification**: For identified root causes, verify using the controlled variable method (change only one factor while keeping others constant, and observe whether failure cases are resolved).
6. **Establish Monitoring Dashboard**: Continuously track the occurrence frequency of the three root cause categories in production environments to detect new problem patterns in a timely manner.

</details>

**15. When RAG Performance Is Poor, Is the Issue More in the Retrieval or Generation Stage?**

<details>
<summary>💡 Show Answer</summary>

Both stages need to be inspected, but it is recommended to perform a "divide and conquer" diagnosis first: fix the retrieval results to manually inspect generation quality, and fix the ideal context to check whether generation is correct, thereby isolating which stage the primary issue stems from. The following is the systematic diagnostic framework:

**Step 1: Isolate Retrieval Issues**
- For a batch of failure cases, manually inspect the retrieved top-k chunks—do they contain the information required to answer the question?
- If the top-k results for a large number of cases lack correct information $\rightarrow$ **issue is in retrieval**, requiring optimization of chunking strategies, embedding models, hybrid search, or reranking;
- If the top-k results already contain correct information but the answer is still wrong $\rightarrow$ the issue likely lies in generation.

**Step 2: Isolate Generation Issues**
- Pass the chunk containing the correct answer directly to the LLM as the "ideal context" and inspect generation quality;
- If the model still answers incorrectly under the ideal context $\rightarrow$ **issue is in prompt design or LLM capacity**, requiring prompt optimization or switching to a stronger model;
- If the model answers correctly under the ideal context but incorrectly under the retrieved context $\rightarrow$ **issue is in context quality** (excessive noise, positioning issues, information loss from compression, etc.).

**Empirical Distribution**: In engineering practice, approximately 40% of RAG issues originate from the retrieval side (poor chunking, embedding incompatibility, insufficient recall), about 30% stem from context quality (high noise, poor ranking), and roughly 30% come from the generation side (inappropriate prompts, hallucinations, LLM capacity limitations). There is no such thing as an "issue isolated to just a single stage"; systematic troubleshooting is far more efficient than subjective guessing.

</details>

**16. How to Design an A/B Testing Framework for a RAG System?**

<details>
<summary>💡 Show Answer</summary>

1. **Experiment Design**: Define comparison variables clearly, changing only one variable at a time;
2. **Traffic Splitting**: Hash by user ID or session ID to randomly split traffic into two groups, each using a different RAG configuration;
3. **Metric Definition**: Primary metrics (user satisfaction, task completion rate) + auxiliary metrics (retrieval precision, Faithfulness, latency, cost);
4. **Sample Size Calculation**: Calculate the minimum sample size based on Minimum Detectable Effect (MDE) and significance requirements ($p < 0.05$, $power = 80%$);
5. **Experiment Isolation**: Ensure all other user experience factors are identical between the two groups;
6. **Results Analysis**: After reaching the sample size, use statistical tests (t-test / bootstrap) to determine difference significance;
7. **Decision Rules**: Significant improvement $\rightarrow$ deploy new configuration to 100% traffic; no significant difference $\rightarrow$ select the lower-cost option; significant decline $\rightarrow$ rollback.





![Experimental setup and flow for RAG system A/B testing in production. (线上生产环境中RAG系统A/B测试的实验方案与流程)](https://img.pricenode.uk/llm_interview_questions/english/chapter7/fig24_ab_testing.png)

</details>

**17. How to Continuously Monitor RAG System Quality in Production Environments?**

<details>
<summary>💡 Show Answer</summary>

1. **Real-Time Metrics Monitoring**: Retrieval latency, generation latency, end-to-end latency (P50/P99), zero-retrieval rate, system error rate;
2. **Quality Sampling Evaluation**: Randomly sample 50–100 queries daily from production traffic, evaluate automatically with RAGAS, and track quality trends;
3. **User Feedback Signals**: Collect feedback such as user thumbs-up/thumbs-down and satisfaction ratings as proxy quality metrics;
4. **Knowledge Base Coverage Monitoring**: Track the proportion of "zero-retrieval" queries to identify knowledge coverage gaps;
5. **Anomaly Detection**: Automatically trigger alerts when quality metrics drop suddenly;
6. **Version Tracking**: Record correlations between each deployment change and shifts in quality metrics;
7. **Dashboard**: Visualize the above metrics for real-time team monitoring.

</details>

**18. What Are the Datasets Commonly Used to Train and Evaluate RAG Systems?**

<details>
<summary>💡 Show Answer</summary>

**Retrieval Model Training and Evaluation Datasets**:
- **MS MARCO** (Microsoft Machine Reading Comprehension): A large-scale passage retrieval dataset with 1 million+ real Bing search queries and relevant passage annotations, serving as the primary benchmark for retrieval model fine-tuning and NDCG evaluation;
- **BEIR** (Benchmarking Information Retrieval): A heterogeneous retrieval benchmark encompassing retrieval datasets across 18 diverse domains and tasks (scientific literature, news, healthcare, etc.), used to evaluate out-of-domain generalization of retrieval models;
- **MTEB** (Massive Text Embedding Benchmark): Evaluates the comprehensive capability of embedding models across 56 tasks including retrieval, classification, clustering, and semantic textual similarity, acting as the most authoritative embedding model leaderboard today.

**End-to-End QA Evaluation Datasets**:
- **Natural Questions (NQ)**: Real Google search queries + Wikipedia passage answers, serving as a core benchmark for open-domain question answering;
- **TriviaQA**: Large-scale knowledge-intensive question answering containing evidence documents collected from Wikipedia and the web, testing the knowledge retrieval and reasoning capabilities of models;
- **SQuAD 2.0**: Stanford Question Answering Dataset, containing paragraph-level context QA (including "unanswerable" questions) to evaluate precise answer extraction capabilities within limited context;
- **HotpotQA**: A multi-hop reasoning QA dataset where answers require synthesizing 2 or more Wikipedia passages, specifically evaluating the multi-hop reasoning capability of RAG systems;
- **KILT** (Knowledge Intensive Language Tasks): A benchmark for knowledge-intensive tasks, uniformly evaluating diverse tasks such as fact-checking, QA, dialogue, and relation extraction.

**Chinese Datasets**:
- **CMRC 2018**: Chinese Machine Reading Comprehension dataset;
- **DRCD**: Delta Reading Comprehension Dataset;
- **C-MTEB**: Chinese MTEB, specifically evaluating Chinese embedding models.

</details>

**19. What Challenges Are Faced When Fine-Tuning RAG Models?**

<details>
<summary>💡 Show Answer</summary>

RAG systems involve multiple fine-tunable components (embedding models, rerankers, generative LLMs), each presenting unique fine-tuning challenges:

**Challenges in Fine-Tuning Embedding Models**:
1. **High Training Data Construction Cost**: Requires a large number of question–relevant document pairs (positive samples) and question–irrelevant document pairs (negative samples), incurring extremely high manual annotation costs; automatically generated negative samples (e.g., BM25 hard negatives) vary in quality;
2. **Critical Quality of Negative Samples**: In contrastive learning, the quality of "hard negative samples" (documents that appear relevant but are actually irrelevant) significantly impacts model performance, yet mining hard negatives is inherently difficult;
3. **Overfitting Risk**: When domain-specific data is scarce, models easily overfit—improving in-domain performance at the expense of reduced generalization capability.

**Challenges in Fine-Tuning Generative LLMs**:
1. **Balance Between Faithfulness and Fluency**: The fine-tuning objective is to make the model "more faithful to the retrieved context", but excessive constraints may result in overly mechanical outputs lacking natural language fluency;
2. **Catastrophic Forgetting**: Fine-tuning on domain-specific data may degrade the model's original general capabilities (e.g., reasoning, format following), requiring careful balance;
3. **High Computational Cost**: Full fine-tuning of large LLMs (70B+) requires dozens of high-end GPUs, costing tens of thousands of dollars; parameter-efficient fine-tuning like LoRA reduces costs, but with some trade-off in performance;
4. **Difficult Alignment Evaluation**: There is no unified standard for evaluating the overall performance of fine-tuned models in RAG scenarios (retrieval utilization, faithfulness, response quality), necessitating the construction of custom evaluation datasets.

**Challenges in Joint End-to-End Fine-Tuning**: Jointly training the retriever and generator (as in REALM and the original RAG paper) faces immense challenges in gradient propagation and training stability, with extremely high engineering complexity. The current mainstream practice remains optimizing each component separately.

%% ============================================================
%% 10. Hallucination Control and Factual Consistency in RAG
%% ============================================================

</details>

