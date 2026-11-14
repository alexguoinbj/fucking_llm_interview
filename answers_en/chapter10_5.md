# 10.5 Decision Framework for Model Fine-Tuning vs. RAG vs. Prompting — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 10: LLM Product Deployment and Commercialization · 1 questions

**1. In What Scenarios Should Prompt Engineering, RAG, SFT, RL, and RLHF Be Applied?**

<details>
<summary>💡 Show Answer</summary>

**Selection Framework: From Lightweight to Heavyweight, From Low Data to High Data**

\begin{table}[htbp]



| Method | Applicable Scenarios | Prerequisites | Advantages | Limitations |
| --- | --- | --- | --- | --- |
| **Prompt Engineering** | Clear tasks, no customization needed; rapid prototyping | Capable foundation model available | Zero training cost, flexible and fast | Performance ceiling, high token consumption |
| **RAG** | Requires up-to-date/private knowledge; knowledge-intensive tasks | Knowledge base can be constructed | Updatable knowledge, traceable sources | Retrieval quality limits performance ceiling, high latency |
| **SFT** | Requires specific output formats, styles, or domain adaptation | High-quality annotated data available (1k--100k) | Excellent format/style alignment | Requires annotated data, risk of catastrophic forgetting |
| **RL (without Human Feedback)** | Clear and verifiable reward signals (code execution, math verification) | Automated reward functions can be programmed | No human annotation needed, highly scalable | Reward design is difficult, potential for reward hacking |
| **RLHF** | Requires alignment with human preferences; safety alignment; general assistants | Capacity for human preference annotation | Outputs best aligned with human expectations | High cost, complex implementation |


\end{table}

**Decision Workflow:**
1. Start with Prompt Engineering $\rightarrow$ Stop if performance meets requirements
2. Requires private knowledge $\rightarrow$ Add RAG
3. Format/style adaptation issues $\rightarrow$ Apply SFT
4. Clear and verifiable metrics exist (code/math) $\rightarrow$ Apply RL
5. Comprehensive human preference alignment required $\rightarrow$ Apply RLHF



![Decision Workflow for Prompt Engineering, RAG, SFT, and RL](https://img.pricenode.uk/llm_interview_questions/english/chapter10/fig05_tech_decision.png)

</details>

