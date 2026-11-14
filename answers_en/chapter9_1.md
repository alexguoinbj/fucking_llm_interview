# 9.1 AI Coding Tool Evaluation (Cursor, Claude Code) — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 9: AI Coding · 22 questions

**1. Why Have AI Coding Tools Like Cursor Gained Widespread Popularity?**

<details>
<summary>💡 Show Answer</summary>

It deeply integrates AI capabilities into developers' existing workflows (editors), reducing context-switching costs. Simultaneously, it supports code understanding and generation based on the context of the entire codebase, making the user experience much closer to actual development scenarios than standalone conversational tools.

</details>

**2. Where Does Cursor's Greatest Advantage Lie?**

<details>
<summary>💡 Show Answer</summary>

Its deeply integrated in-editor interactive experience, combined with the ability to understand the entire project context, allows developers to receive AI assistance with low friction during daily coding workflows.

</details>

**3. What Problems Do Cursor's Tab Auto-Completion and Chat Mode Address?**

<details>
<summary>💡 Show Answer</summary>

Tab completion addresses high-frequency, fine-grained tasks such as "conveniently completing the current code snippet," emphasizing low latency and accurate prediction of context around the cursor. Chat mode addresses complex tasks requiring cross-file and cross-module reasoning, emphasizing conversational intent clarification and multi-step planning. Together, they complement each other to cover needs across different granularities, from "writing a single line" to "modifying an entire feature."



![Comparison and collaboration workflow between inline Tab Auto-Completion and Chat interactive modes.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig01_tab_vs_chat.png)

</details>

**4. What Pain Points Does Cursor's Composer/Multi-File Editing Feature Solve?**

<details>
<summary>💡 Show Answer</summary>

It resolves the pain point in traditional Chat modes where developers had to manually copy and paste AI suggestions line-by-line into multiple files. Composer can plan and apply modifications across multiple files in a single pass while maintaining consistency across files, closely mirroring real-world multi-file collaborative development scenarios.



![Multi-file concurrent editing and modification application flow in Composer.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig02_composer_workflow.png)

</details>

**5. Why Has the User Experience of Tools Like Claude Code Received High Praise?**

<details>
<summary>💡 Show Answer</summary>

It stems from its strong understanding of codebase structure, the coherence of multi-file collaborative edits, and its ability as a command-line Agent to autonomously execute operations like compilation and testing and iterate based on feedback, reducing the burden on developers to manually paste code back and forth.

</details>

**6. Where Does Claude Code's Greatest Advantage Lie?**

<details>
<summary>💡 Show Answer</summary>

As a command-line Agent, it can autonomously perform multi-step operations (running commands, reading and writing files, executing tests) and iterate based on real execution feedback, making it ideal for complex coding tasks that require exploration and verification.



![Autonomous execution loop of Claude Code as a command-line developer Agent.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig03_claude_code_agent.png)

</details>

**7. What Problem Does Claude Code's "Plan Mode" Solve?**

<details>
<summary>💡 Show Answer</summary>

Before executing risky or broad modifications, it requires the model to output an execution plan for human review rather than making direct code changes. This prevents the Agent from diving headfirst into an incorrect modification path due to misaligned understanding, moving error correction costs forward from "rollback after editing" to "confirmation before editing."

</details>

**8. What Is the Core Difference Between GitHub Copilot and Cursor?**

<details>
<summary>💡 Show Answer</summary>

Copilot leans toward being a "lightweight, embedded" completion enhancement that attaches to existing IDEs (such as VS Code or JetBrains) as a plugin, causing minimal disruption to mainstream editor workflows. Cursor itself is a standalone editor product capable of deeper customization across the entire user interface and context management mechanism, offering a higher ceiling but also incurring higher migration costs.

</details>

**9. How Are Products Like Windsurf (Codeium) Positioned?**

<details>
<summary>💡 Show Answer</summary>

They also follow the "AI-native editor" path. Similar to Cursor, they emphasize understanding the overall project context and automating multi-step execution (such as its Cascade feature). Differentiation lies mostly in interaction design details and trade-offs in execution autonomy, making them direct competitors in the same space.

</details>

**10. What Is the Fundamental Difference Between Fully Autonomous Agents Like Devin and Cursor?**

<details>
<summary>💡 Show Answer</summary>

Cursor's core assumption is "human-in-the-loop," where every modification allows real-time developer inspection and intervention. Devin's core assumption is allowing the Agent to independently complete the full closed loop from task comprehension to acceptance, with humans intervening only at the start and end of a task. This places much higher demands on the model's planning and self-correction capabilities while leaving less margin for error.



![Comparison of Fully Autonomous Agent (Devin) vs. Human-in-the-Loop (Cursor) interaction patterns.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig04_devin_vs_cursor.png)

</details>

**11. What Are the Characteristics of Open-Source Command-Line Tools Like Aider?**

<details>
<summary>💡 Show Answer</summary>

Lightweight, open-source, and capable of freely swapping underlying models, they focus on the core closed loop of "code modification + Git integration." Without complex graphical interfaces or vendor lock-in, they are ideal for developers and teams seeking flexibility, building custom workflows, or managing costs tightly.

</details>

**12. What Are the Advantages of Cloud Vendor Tools Like Amazon Q Developer?**

<details>
<summary>💡 Show Answer</summary>

Their main advantage lies in deep integration with the cloud vendor's own ecosystem (cloud service APIs, internal permission systems, enterprise security and compliance). For enterprise clients already heavily invested in that cloud ecosystem, integration and trust costs are lower, though their general coding experience may not necessarily surpass products from independent startups.

</details>

**13. Why Do Some Teams Prefer Open-Source Tools Over Commercial Products?**

<details>
<summary>💡 Show Answer</summary>

Primary considerations include data privacy (keeping code within the intranet, supporting on-premises deployment), controllable costs (avoiding lock-in to subscription price changes), and customizability (freedom to swap or fine-tune underlying models and customize code according to team habits). The trade-off is assuming the burden of maintenance and iteration costs.

</details>

**14. Between the Fully Autonomous Agent Approach and the Human-in-the-Loop Approach, Which Is More Suitable for the Current Stage?**

<details>
<summary>💡 Show Answer</summary>

Given current model capabilities, the human-in-the-loop approach (retaining manual confirmation at critical nodes) presents more controllable risks and faster adoption in production environments. The fully autonomous approach is better suited for tasks with clear boundaries and high verifiability (such as scenarios with comprehensive test coverage). As model planning and self-verification capabilities improve, the applicability of fully autonomous agents will gradually expand.

</details>

**15. What Are the Pros and Cons of IDE Plugin-Based vs. Standalone Editor-Based Products?**

<details>
<summary>💡 Show Answer</summary>

Plugin-based products feature low migration costs and reuse developers' existing editor configurations and plugin ecosystems, but are limited by the host IDE's extension APIs, offering less freedom for deep interaction design. Standalone editor products can deeply customize the entire interface and context management mechanism, yielding a higher ceiling for user experience, but require developers to migrate their work environments, presenting a certain adoption threshold.

</details>

**16. What Is the Fundamental Difference Between AI Coding Tools and Traditional IDE Code Completion (e.g., IntelliSense)?**

<details>
<summary>💡 Show Answer</summary>

Traditional code completion relies on static type inference and symbol tables, covering syntactically "valid" completions. AI coding tools, based on LLMs' understanding of semantics and intent, can generate entire blocks of logic, cross-file modifications, and even interpret natural language requirements, expanding coverage from "syntactically correct" to "semantically reasonable."

</details>

**17. Which Dimensions Should Be Considered When Evaluating AI Coding Tools?**

<details>
<summary>💡 Show Answer</summary>

Key dimensions include code generation correctness and maintainability, depth of project context understanding, coherence of multi-file collaborative editing, execution feedback loops (ability to autonomously run tests and iterate), as well as engineering and user experience dimensions such as response speed, ease of use, and pricing.

</details>

**18. How Much Does Context Window Size Impact Product Experience?**

<details>
<summary>💡 Show Answer</summary>

The context window determines how much codebase information can be included in a single interaction. A context window that is too small often leads to the model missing critical dependency code, producing locally correct but globally inconsistent output. However, a larger window is not unconditionally better; it must be paired with retrieval and context filtering mechanisms. Otherwise, excessive irrelevant information in a large window will dilute model attention and degrade generation quality.

</details>

**19. How Should Response Speed and Generation Quality Be Balanced?**

<details>
<summary>💡 Show Answer</summary>

High-frequency, fine-grained scenarios (such as inline completion) must prioritize response speed to maintain interactive fluidity, even if sacrificing some quality. Low-frequency, high-value scenarios (such as cross-file refactoring) afford higher user latency tolerance, so priority should shift to generation quality and correctness. These two types of scenarios typically require different model sizes for support.

</details>

**20. How Should Multi-Language and Multi-Framework Support Capabilities Be Evaluated?**

<details>
<summary>💡 Show Answer</summary>

Evaluation can be conducted by constructing a unified task benchmark (similar tasks across different tech stacks) on real-world projects in various languages and frameworks for cross-sectional comparison. Attention should also be paid to whether tools provide specialized adaptation for niche languages/frameworks rather than performing well solely on mainstream languages (such as Python or JavaScript).

</details>

**21. How Does the Product Pricing Model (Subscription vs. Pay-as-You-Go) Impact User Choices?**

<details>
<summary>💡 Show Answer</summary>

Subscriptions are more favorable for high-volume users with predictable usage due to strong cost control. Pay-as-you-go is better suited for light users or those with fluctuating usage, avoiding paying for rarely used capabilities. For enterprise procurement, convenience regarding bulk licensing and usage auditing across team sizes must also be considered.

</details>

**22. What Do Enterprises Value Most When Selecting AI Coding Tools?**

<details>
<summary>💡 Show Answer</summary>

Beyond generation quality itself, enterprises typically prioritize data security and compliance (whether code will be used for training, availability of on-premises deployment), integration costs with existing R&D workflows (CI/CD, code review standards), and team-level manageability (permission control, usage auditing). Pure model capabilities are often not the sole deciding factor.

</details>

