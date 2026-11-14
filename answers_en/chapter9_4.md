# 9.4 Code Generation, Refactoring, Testing, and Review — Questions & Answers
> [⬅ Back to Index](../README_EN.md) · Chapter 9: AI Coding · 22 questions

**1. How Does AI Perform Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Based on an understanding of requirements, as well as the style and interface conventions of the existing codebase, AI generates code that complies with project standards. High-quality generation relies on retrieving sufficient project context, such as using existing implementations of similar functionality as references.

</details>

**2. How to Ensure That Generated Code Style Remains Consistent with the Project?**

<details>
<summary>💡 Show Answer</summary>

Prior to generation, existing code with similar styling can be retrieved as reference exemplars. Project static analysis configurations (such as naming conventions and formatting rules) are incorporated as explicit constraints, followed by automated formatting tools for post-generation normalization, rather than relying solely on the model's default stylistic preferences.

</details>

**3. How Does AI Handle Custom Frameworks and Internal Libraries During Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Dedicated indexes of interface documentation and usage examples must be built for internal libraries and custom frameworks. Because such proprietary libraries are absent from public training corpora, the model lacks prior knowledge and must retrieve genuine in-project invocation examples to generate correct code.

</details>

**4. How Can "Hallucinated" Calls to Non-Existent APIs Be Avoided During Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Automated static checks (such as type checking and symbol resolution) can be performed post-generation to verify that invoked functions or methods actually exist within the target library or module. When hallucinated calls are detected, the error output is fed back into the model for re-generation, replacing blind reliance on model correctness with a verifiable feedback loop.



![Automated type checking and compilation feedback loop for preventing API hallucinations.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig13_anti_hallucination_loop.png)

</details>

**5. How Does AI Balance Readability and Conciseness in Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Existing project conventions should serve as the primary baseline (e.g., if a project favors explicit, clear writing, overly concise or clever tricks should be avoided). In the absence of clear conventions, priority should generally be given to readability and maintainability, avoiding the sacrifice of downstream maintenance costs for the sake of line-count reduction.

</details>

**6. How Can AI Prevent Introduced Security Vulnerabilities (e.g., SQL Injection, XSS) in Generated Code?**

<details>
<summary>💡 Show Answer</summary>

Secure coding guidelines (such as parameterized queries instead of string concatenation, and sanitizing user inputs) must be enforced during generation, combined with automated post-generation static application security testing (SAST). Security checks should be an indispensable pipeline step rather than an afterthought.

</details>

**7. How Does AI Handle Code Generation Tasks with Ambiguous Requirements?**

<details>
<summary>💡 Show Answer</summary>

When faced with ambiguous requirements, the model should not make arbitrary guesses to generate code. A more robust approach is to identify key ambiguities and seek user clarification, or to infer the most probable intent based on similar implementations in the codebase while explicitly annotating all underlying assumptions in the generated output to facilitate user review.

</details>

**8. How Are Multi-Language Polyglot Projects (e.g., Coordinated Frontend-Backend Modifications) Handled in Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Inter-language interface contracts (such as API specifications and type/schema mappings) must serve as anchor points for cross-language changes. This ensures that when backend API definitions change, corresponding frontend call sites are identified and updated in sync, avoiding omitted edits or type mismatches across language boundaries.



![Coordinated cross-language code generation anchored on shared API contracts.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig14_polyglot_edit_sync.png)

</details>

**9. How Does AI Assist in Code Refactoring?**

<details>
<summary>💡 Show Answer</summary>

AI analyzes the functional boundaries and all invocation sites of the code to be refactored, ensures behavioral equivalence before and after refactoring (verifiable via test cases), and generates refactored code according to target design patterns and standards.

</details>

**10. Is AI Capable of Large-Scale Refactoring (e.g., Framework Migration), and What Are the Challenges?**

<details>
<summary>💡 Show Answer</summary>

For highly pattern-based migrations that can be replaced in bulk via systematic rules (e.g., API signature changes), AI can significantly boost efficiency. However, for migrations involving deep architectural shifts or implicit business logic couplings, the core challenge lies in the difficulty of guaranteeing full coverage of edge cases and historical implicit conventions, typically requiring phased human verification rather than a fully automated one-shot execution.

</details>

**11. How Is Backward API Compatibility Preserved During Refactoring?**

<details>
<summary>💡 Show Answer</summary>

Public contract interfaces exposed to external callers must be clearly distinguished from internal implementation details. Modifications to public interfaces should favor additive changes rather than direct replacements, preserving temporary compatibility layers (e.g., deprecation annotations rather than immediate deletion) and validating compatibility through tests covering both old and new execution paths.

</details>

**12. How to Choose the Appropriate Refactoring Strategy in AI-Assisted Workflows (Iterative Small Steps vs. One-Time Major Overhaul)?**

<details>
<summary>💡 Show Answer</summary>

When comprehensive test coverage and controlled scopes exist, an iterative approach (small-scope refactoring with immediate validation) minimizes risk and simplifies bug localization. Only when test coverage is sparse or a complete architectural overhaul is intended should a broader, one-time modification be considered, requiring correspondingly stricter human oversight.

</details>

**13. How Does AI Assist in Test Code Generation?**

<details>
<summary>💡 Show Answer</summary>

Based on the functionality and input-output contracts of the code under test, AI generates test cases covering happy paths, edge cases, and exception conditions. Ideally, this should be combined with code coverage analysis to iteratively supplement uncovered scenarios.

</details>

**14. How Should AI Focus Differ Between Generating Unit Tests and Integration Tests?**

<details>
<summary>💡 Show Answer</summary>

Unit test generation should focus on input-output contracts and boundary conditions of isolated functions or modules, isolating external dependencies using mocks. Integration test generation requires comprehending realistic interaction pathways and data flows across multiple modules, depending heavily on a holistic system architecture understanding. The required context scope and validation mechanisms differ significantly between the two.

</details>

**15. How Can Test Generation Avoid Low-Quality Tests Created Merely to "Inflate Coverage"?**

<details>
<summary>💡 Show Answer</summary>

Code coverage should not be the sole metric. Test generation must synthesize business logic and edge conditions to produce assertions with genuine validation value (rather than merely executing code paths without checking outcomes). Techniques like mutation testing can be incorporated to evaluate whether test cases truly catch latent defects.

</details>

**16. How Can the Correctness of Generated Test Cases Be Validated (Given That Test Code Itself May Have Bugs)?**

<details>
<summary>💡 Show Answer</summary>

Correctness can be verified bi-directionally by executing tests on known-correct code to confirm passing, and on intentionally mutated/faulty code to confirm failure detection. Merely having tests pass does not prove test reliability; bi-directional validation ensures neither false positives nor false negatives occur.



![Bi-directional validation mechanism for generated test suites using code mutations.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig15_bidirectional_test_val.png)

</details>

**17. How Does AI Assist in Code Review?**

<details>
<summary>💡 Show Answer</summary>

AI reviews code for adherence to project standards, potential bugs, security risks, and logical clarity, providing actionable remediation suggestions for identified issues. However, evaluating domain business logic correctness still requires human context.

</details>

**18. How Does AI Identify Performance Issues (Beyond Syntax and Basic Logic Errors) During Code Review?**

<details>
<summary>💡 Show Answer</summary>

It leverages knowledge of common performance anti-patterns (e.g., redundant computations in loops, unnecessary synchronous blocking, inefficient data structure choices) alongside inferences regarding execution hot paths and call frequencies, prioritizing performance analysis on critical execution paths rather than applying uniform checks indiscriminately.

</details>

**19. How Does AI Detect Potential Concurrency and Race Condition Issues in Code Review?**

<details>
<summary>💡 Show Answer</summary>

AI analyzes access paths to shared state, evaluates whether locking mechanisms fully cover all shared-resource accesses, and inspects lock acquisition orders for deadlock risks. Because static rules rarely catch all such issues, AI primarily flags potential risk areas for human investigation rather than issuing definitive verdicts.

</details>

**20. How Can AI-Assisted Code Review Avoid Excessive False Positives That Lead to Developer Fatigue?**

<details>
<summary>💡 Show Answer</summary>

Findings can be categorized by severity (prioritizing security flaws and explicit bugs over stylistic suggestions), providing clear rationale and code examples instead of vague warnings. Additionally, rules with high false-positive rates can be continuously tuned based on historical repository feedback to avoid the "cry wolf" effect that erodes developer trust.

</details>

**21. How Should Human Review Be Tiered for AI-Generated Code (Which Changes Can Be Merged Directly vs. Which Require Mandatory Review)?**

<details>
<summary>💡 Show Answer</summary>

Review can be tiered according to impact radius and risk level: low-risk, localized edits backed by green unit tests may have relaxed review requirements. Conversely, modifications touching core business logic, security-critical modules, or broad cross-module boundaries must undergo mandatory human review prior to merging, regardless of test pass status.



![Risk-based tiered code review workflow for AI-generated changes.](https://img.pricenode.uk/llm_interview_questions/english/chapter9/fig16_tiered_code_review.png)

</details>

**22. How to Ensure the Explainability of Generated Code (Why It Was Written This Way)?**

<details>
<summary>💡 Show Answer</summary>

The model can be prompted to output concise rationale for key design choices alongside code generation (explaining implementation trade-offs and architectural decisions). Retaining these justifications in code comments and commit messages prevents developers from having to reverse-engineer intent later.

</details>

