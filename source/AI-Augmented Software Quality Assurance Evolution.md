# **The Multi-Layered Software Quality Funnel: Engineering Governance and Architectural Oversight in the AI Integration Era**

The software engineering landscape in 2026 has transitioned from the "Generation Era," characterized by the novelty of artificial intelligence producing isolated functions, into the "Integration Era," where the primary challenge is the orchestration and governance of massive, machine-generated codebases.1 As organizations adopt autonomous development workflows, the volume of code produced has increased at an unprecedented pace, with AI now accounting for approximately 41% of all new code committed.2 This surge has fundamentally altered the structural mechanics of software quality assurance (QA). Traditional testing models, which relied on human developers to maintain a shared mental model through line-by-line inspection, have become unsustainable.3 In this contemporary environment, technology leaders are shifting toward a developer-first QA framework centered on independent verification and a multi-layered quality funnel.5 This funnel integrates five critical dimensions: compilation, deterministic linting, automated testing, agentic AI reviewers, and high-level human architectural oversight.2

## **The Evolution of the Quality Funnel: From Manual Inspection to Automated Multi-Layered Filtration**

The historical progression of quality assurance has moved from manual, hand-executed test cases to a complex, automated ecosystem.7 Early software development was marred by repetitive manual tasks that were prone to human error and often resulted in significant maintenance burdens, consuming 30% to 40% of a team’s QA capacity.7 The shift to automation testing was the first revolution, enabling repetitive execution across multiple environments and configurations.7 However, as the industry enters the AI era, automation alone is insufficient. The current objective is to transform code review and testing from a passive gate into a series of active filters designed to catch different classes of defects.9

This evolution is driven by the "Productivity Paradox," where AI tools allow developers to write code 25% to 35% faster, yet the time required for verification and integration has increased, often offsetting these gains.3 Research indicates that 96% of developers do not fully trust AI-generated code, and with good reason: AI-generated pull requests (PRs) wait 4.6 times longer for review than human-written code and have a significantly lower acceptance rate—32.7% compared to 84.4% for manual contributions.3 To address this, engineering organizations are implementing a hierarchical funnel that reduces the cognitive load on humans by offloading objective, low-level checks to automated agents, thereby preserving human expertise for high-stakes decisions.6

| Feature | Manual Era (Pre-2010s) | Automation Era (2010s-2022) | AI Integration Era (2023-Present) |
| :---- | :---- | :---- | :---- |
| **Primary Code Source** | Human Developers | Human Developers | AI-Assisted / Agentic 1 |
| **Review Method** | Manual Line-by-Line | PR Reviews \+ Linters | AI-First Review \+ Architectural Audit 2 |
| **Testing Focus** | Scripted Execution | CI/CD Pipelines 14 | Outcome Verification in Previews 4 |
| **Human Role** | Implementation | Quality Enforcement | Architectural Governance 1 |
| **Bottleneck** | Coding Speed | Testing Speed | Review Throughput 2 |

## **Layer 1: Deterministic Foundations—Compilation and Syntax Verification**

The first filter in the modern quality funnel is the build stage, which serves as a deterministic gate for code integrity.16 Compilation and syntax checking verify that the code adheres to the formal grammar of the language and that all required dependencies are resolvable.17 While this layer has always been present, its importance has been magnified by AI's tendency to produce "hallucinated" syntax or reference non-existent libraries.19

In an agentic workflow, the build phase must be repeatable and consistent to provide a foundation for further analysis.16 This layer identifies basic failures before any computational resources are spent on expensive LLM inference or complex integration tests.19 Automated build checks also include dependency resolution, which ensures that all third-party components are correctly resolved and included.17 Given that AI tools often generate complex dependency trees for simple tasks—a phenomenon known as "dependency overuse"—this stage is critical for preventing the expansion of the attack surface.22

## **Layer 2: Standardized Enforcement—Linters and Static Analysis**

The second layer consists of linters and static analysis tools, which are distinguished from AI reviewers by their deterministic nature.23 These tools apply a fixed set of rules to catch stylistic errors, improper formatting, and adherence to coding standards.12 In large-scale polyglot monorepos, tools like SonarQube serve as the backbone of "verify" in the modern software development life cycle (SDLC), analyzing code across more than 30 languages with a unified policy engine.5

Linters like ESLint for JavaScript or Flake8 for Python provide a level of insight that goes beyond basic syntax, flagging potential memory leaks, concurrency problems, and security vulnerabilities like cross-site scripting (XSS).17 By catching these routine issues automatically, linters free up human reviewers to focus on more significant matters such as architectural decisions.24 However, traditional static analysis is often "architecturally blind," focused on file-level quality rather than how changes affect dependent services.23 This limitation necessitates the subsequent layers of the funnel.

| Tool Type | Accuracy | Context Depth | Best Use Case |
| :---- | :---- | :---- | :---- |
| **Linters** | High (Deterministic) 23 | Low (File/Line) | Style and basic syntax rules 24 |
| **Static Analysis** | High 23 | Medium (Intra-repo) | Vulnerabilities and code smells 5 |
| **AI Reviewers** | Variable (Probabilistic) 23 | High (Cross-repo/Service) | Intent and architectural fit 2 |

## **Layer 3: Behavioral Validation—The Testing Tier**

Automated testing remains the cornerstone of software quality assurance, shifting from manual regression to continuous, parallelized execution within CI/CD pipelines.8 In the AI era, the focus has shifted toward a "fail-fast" approach, where automated tests—including unit, integration, and end-to-end tests—are triggered upon every commit to identify issues early.14

AI agents have transformed test creation, allowing developers to generate test scaffolds and surface edge cases directly from source code or natural language requirements.21 This has enabled a continuous expansion of coverage; however, it has also introduced "flaky" tests that can lead to pipeline noise.8 To mitigate this, modern platforms leverage AI to analyze application state and provide context-rich feedback, moving from reactive testing to proactive quality engineering.8 For instance, AI can now reuse functional tests for load testing and accessibility checks, organically expanding coverage without requiring separate initiatives.8

## **Layer 4: The Agentic Tier—Context-Aware AI Reviewers**

The fourth layer is the introduction of AI-powered code review systems that act as an autonomous reviewer before human intervention.2 These agents, such as Qodo, CodeRabbit, and Amazon Q, differ from traditional static tools because they possess "system-aware reasoning," allowing them to understand the intent behind a change rather than just the syntax of the diff.2

An effective AI reviewer must align the code changes with the associated business requirements, typically by reading Jira or Azure DevOps tickets.2 This alignment allows the AI to flag problems that humans often catch late, such as changes that drift beyond the requested scope or implementation of edge cases that were described in the ticket but missed in the code.2 Furthermore, agentic reviewers can detect runtime failures that are hidden beyond the immediate diff, such as breaking changes across service boundaries or misalignments with shared SDKs.2

| AI Review Tool | Enterprise Focus | Integration Point | Unique Capability |
| :---- | :---- | :---- | :---- |
| **Qodo** | Multi-agent workflows 2 | CI/CD & IDE | Ticket-to-code intent validation 2 |
| **GitHub Copilot** | Native ecosystem 2 | GitHub PRs | Automated PR summaries and diff reasoning 25 |
| **Amazon Q** | Cloud correctness 26 | AWS Environments | Security scanning and AWS best practices 2 |
| **Snyk Code** | Security/Compliance 2 | SDLC/CI-CD | Repo-wide semantic security analysis 2 |
| **CodeRabbit** | Real-time feedback 27 | IDE & Git | Sub-second feedback and automated fixes 27 |

The integration of these agents into the CI/CD pipeline requires specific operational guardrails.19 To maintain velocity, AI reviews are often moved to an asynchronous path, running only after the primary build and tests have passed, thereby ensuring the critical path is not blocked by inference latency.19 Additionally, "circuit breakers" are implemented to monitor token-heavy prompts and prevent pipeline stalls if the AI generation exceeds specific time thresholds.19

## **Layer 5: Strategic Oversight—The Human as Architect and Editor-in-Chief**

The final and most critical layer of the quality funnel is human review. In the AI era, the role of the human reviewer has evolved from "code production" to "technical evaluation" and "architectural governance".1 As AI increases the volume of code produced, senior engineers must transition to an "Editor-in-Chief" mindset, focusing on high-level, subjective tasks that require deep business context and architectural judgment.1

### **Architectural Governance and System Integrity**

Human reviewers are now responsible for "global reasoning" across large-scale systems, a task where AI models—which excel at "local reasoning" within single files—often struggle.12 A senior architect evaluates the long-term impact of a change, determining if it introduces a problematic dependency, violates a domain boundary, or duplicates business logic across microservices.12 This oversight is essential for preventing technical debt and maintaining a scalable system.12

Specific architectural concerns that require human intervention include:

* **Domain Boundary Violations:** Detecting if a service is reaching directly into another service's domain or database.15  
* **Layered Architecture Consistency:** Ensuring the API layer does not bypass service logic to access the database directly.15  
* **Legacy Context:** Guiding AI agents through complex legacy codebases where context is not explicitly documented.1

### **Sampled Inspection and Risk-Based Methodologies**

To manage the surge in code volume, engineering leaders are adopting "Risk-Based Inspection" (RBI) methodologies.28 This approach prioritizes review resources based on the likelihood and consequence of failure.28 Risk is calculated using the formula ![][image1], where POF is the Probability of Failure and COF is the Consequence of Failure.28

At organizations like Meta, an AI-powered technology called "Diff Risk Score" (DRS) predicts the likelihood of a code change causing a production incident.30 This allows the team to prioritize high-risk diffs for immediate senior review while allowing lower-risk changes to progress through automated filters.30 This nuanced approach minimizes production incidents while maximizing the rate of innovation.30

| Review Tier | Risk Level | Scrutiny Intensity | Method |
| :---- | :---- | :---- | :---- |
| **Tier 1: High-Stakes** | High (Auth, Payments) | Maximum | Senior Review \+ External Audit 9 |
| **Tier 2: Business Logic** | Medium | Moderate | AI Review \+ Sampled Human Audit 9 |
| **Tier 3: Boilerplate/UI** | Low | Low | Fully Automated with Periodic Audit 9 |

### **Verification of Test Quality and Logic Alignment**

A critical dimension of high-level human review is the verification of the "right" test cases. While AI can generate tests, it cannot always determine if those tests cover the strategic intent of the business requirement.2 A human reviewer assesses whether the test suite provides adequate coverage for edge cases described in the ticket and ensures that the implementation accounts for business trade-offs discussed in team meetings.2

Furthermore, the human reviewer serves a vital role in mentorship and knowledge sharing.12 AI can flag an error, but it cannot replicate the mentorship loop where a senior developer explains *why* a different approach is better within the specific context of the team's system.12 This transfer of knowledge is essential for the long-term growth of the engineering organization.12

## **Outcome Verification: The Shift to Behavioral Review**

The rise of AI coding agents has led to the claim that "traditional code review is dead".4 The argument posits that if humans can no longer parse the sheer volume of code produced by parallel-working AI agents, they must shift their focus from "reviewing the recipe" (the code) to "tasting the dish" (the outcome).4 This is known as "Outcome Verification".4

### **Previews as the Source of Truth**

In this paradigm, the most important artifact in a PR is no longer the diff, but the "Preview" environment.4 Verification becomes an active session where reviewers interact with the live implementation to ensure it behaves as requested.4 This is particularly critical for backend systems, where race conditions, database migration locks, and complex API interactions cannot be easily seen in a text diff or unit test mock.4

### **Infrastructure Multiplexing and Environment Virtualization**

Moving toward behavioral verification creates a massive infrastructure challenge: the explosion of concurrency.4 An AI agent might spin up ten different strategies to solve a single bug, requiring ten parallel environments for validation.4 To manage this, teams are turning to "environment virtualization" or "multiplexing".4 By using isolation at the application layer, organizations can create thousands of ephemeral sandboxes on a single shared cluster, allowing for end-to-end validation without duplicating underlying infrastructure or exploding cloud budgets.4

## **Security and Risk in the AI-Driven SDLC**

As automation accelerates code production, it also increases the attack surface.13 AI-generated code introduces unique security risks, including "slopsquatting" (recommending non-existent or malicious packages) and prompt injection vulnerabilities.20 Research indicates that 45% of AI-generated code contains OWASP Top 10 vulnerabilities, with Java exhibiting a 72% security failure rate.22

To counter these risks, security signals must be integrated earlier in the funnel.31 High-risk patterns, particularly those touching authentication, authorization, or state management, should trigger additional human scrutiny.31 Organizations are adopting a "trust but verify" mindset, treating AI-generated code as untrusted by default, similar to code copied from an external repository.31

| Security Risk | Hallucination Rate / Failure Rate | Mitigation Strategy |
| :---- | :---- | :---- |
| **CWE-80 (XSS)** | 86% 22 | Mandatory sanitization checks in AI reviewers 22 |
| **CWE-89 (SQL Injection)** | 20% 22 | Static analysis for parameterized queries 22 |
| **Hard-coded Secrets** | 35%+ 22 | Secret detection gates in CI/CD 22 |
| **Non-existent Packages** | 20% of samples 20 | Dependency allow-lists and registry checks 19 |

## **Conclusion: The New Engineering Equilibrium**

The convergence of AI generation and automated governance has established a new equilibrium in software engineering. Velocity is no longer defined by code production but by "review throughput".2 To maintain this velocity, the multi-layered funnel must operate with high-precision automated filters—compilation, linters, tests, and AI reviewers—that handle the vast majority of objective tasks.2

Human reviewers, acting as strategic architects, provide the final and most critical layer of defense. By focusing on architectural fit, business logic alignment, and risk-based inspection, they ensure that the productivity gains of AI do not lead to a "Frankenstein" codebase or unsustainable technical debt.1 The future of software quality lies in this hybrid model, where AI handles the administrative "heavy lifting" while humans provide the "heart" and strategic oversight, ensuring that every line of code—whether human or machine-written—serves the ultimate goal of user satisfaction and system reliability.14

#### **Works cited**

1. From Coder to Architect: Surviving the 2026 AI Shift \- Sequoia Connect, accessed January 20, 2026, [https://sequoia-connect.com/from-coder-to-architect-surviving-the-2026-ai-shift/](https://sequoia-connect.com/from-coder-to-architect-surviving-the-2026-ai-shift/)  
2. AI Code Review Tools Compared: Context, Automation, and Enterprise Scale \- Qodo, accessed January 20, 2026, [https://www.qodo.ai/blog/best-ai-code-review-tools-2026/](https://www.qodo.ai/blog/best-ai-code-review-tools-2026/)  
3. Developers still don't trust AI-generated code \- CIO, accessed January 20, 2026, [https://www.cio.com/article/4117049/developers-still-dont-trust-ai-generated-code.html](https://www.cio.com/article/4117049/developers-still-dont-trust-ai-generated-code.html)  
4. Traditional Code Review Is Dead. What Comes Next? \- The New Stack, accessed January 20, 2026, [https://thenewstack.io/traditional-code-review-is-dead-what-comes-next/](https://thenewstack.io/traditional-code-review-is-dead-what-comes-next/)  
5. Quality assurance in the AI era: a leadership imperative, according to S\&P Global Market Intelligence | Sonar, accessed January 20, 2026, [https://www.sonarsource.com/blog/quality-assurance-in-the-ai-era/](https://www.sonarsource.com/blog/quality-assurance-in-the-ai-era/)  
6. AI Code Review \- CREATEQ, accessed January 20, 2026, [https://www.createq.com/en/software-engineering-hub/ai-code-review](https://www.createq.com/en/software-engineering-hub/ai-code-review)  
7. AI for Quality Assurance: Elevating Software Testing to New Heights \- NiCE, accessed January 20, 2026, [https://www.nice.com/info/ai-for-quality-assurance-elevating-software-testing-to-new-heights](https://www.nice.com/info/ai-for-quality-assurance-elevating-software-testing-to-new-heights)  
8. AI Agents in CI/CD Pipelines for Continuous Quality \- Mabl, accessed January 20, 2026, [https://www.mabl.com/blog/ai-agents-cicd-pipelines-continuous-quality](https://www.mabl.com/blog/ai-agents-cicd-pipelines-continuous-quality)  
9. Almost Right But Not Quite—Building Trust, Validation Processes, and Quality Control for AI-Generated Code \- SoftwareSeni, accessed January 20, 2026, [https://www.softwareseni.com/almost-right-but-not-quite-building-trust-validation-processes-and-quality-control-for-ai-generated-code/](https://www.softwareseni.com/almost-right-but-not-quite-building-trust-validation-processes-and-quality-control-for-ai-generated-code/)  
10. AI PRs Wait 4.6x Longer: LinearB 2026 Benchmarks | byteiota, accessed January 20, 2026, [https://byteiota.com/ai-prs-wait-4-6x-longer-linearb-2026-benchmarks/](https://byteiota.com/ai-prs-wait-4-6x-longer-linearb-2026-benchmarks/)  
11. 2026 Software Engineering Benchmarks Report \- LinearB, accessed January 20, 2026, [https://linearb.io/resources/software-engineering-benchmarks-report](https://linearb.io/resources/software-engineering-benchmarks-report)  
12. Why AI will never replace human code review \- Graphite, accessed January 20, 2026, [https://graphite.com/blog/ai-wont-replace-human-code-review](https://graphite.com/blog/ai-wont-replace-human-code-review)  
13. When AI Writes Code: How the Role of Developers is Changing \- RTInsights, accessed January 20, 2026, [https://www.rtinsights.com/when-ai-writes-code-how-the-role-of-developers-is-changing/](https://www.rtinsights.com/when-ai-writes-code-how-the-role-of-developers-is-changing/)  
14. Implementing Quality Assurance in a CI/CD Pipeline \- F22 Labs, accessed January 20, 2026, [https://www.f22labs.com/blogs/implementing-quality-assurance-in-a-ci-cd-pipeline/](https://www.f22labs.com/blogs/implementing-quality-assurance-in-a-ci-cd-pipeline/)  
15. AI Code Review for Solution Architects: How to Enforce Architectural Patterns Across 100+ Microservices \- DEV Community, accessed January 20, 2026, [https://dev.to/uss/ai-code-review-for-solution-architects-how-to-enforce-architectural-patterns-across-100-3fa4](https://dev.to/uss/ai-code-review-for-solution-architects-how-to-enforce-architectural-patterns-across-100-3fa4)  
16. What Is the CI/CD Pipeline? \- Palo Alto Networks, accessed January 20, 2026, [https://www.paloaltonetworks.com/cyberpedia/what-is-the-ci-cd-pipeline-and-ci-cd-security](https://www.paloaltonetworks.com/cyberpedia/what-is-the-ci-cd-pipeline-and-ci-cd-security)  
17. AI-Assisted Fixes to Code Review Comments at Scale \- arXiv, accessed January 20, 2026, [https://arxiv.org/html/2507.13499v1](https://arxiv.org/html/2507.13499v1)  
18. 10 CI/CD Pipeline Examples To Help You Get Started | Zeet.co, accessed January 20, 2026, [https://zeet.co/blog/ci-cd-pipeline-examples](https://zeet.co/blog/ci-cd-pipeline-examples)  
19. AI in the Pipeline: Reliability Lessons from Adding an LLM to CI/CD ..., accessed January 20, 2026, [https://www.usenix.org/publications/loginonline/ai-pipeline-reliability-lessons-adding-llm-cicd](https://www.usenix.org/publications/loginonline/ai-pipeline-reliability-lessons-adding-llm-cicd)  
20. Emergent Code Review Patterns for AI-Generated Code | Propel, accessed January 20, 2026, [https://www.propelcode.ai/blog/emergent-code-review-patterns-ai-generated-code](https://www.propelcode.ai/blog/emergent-code-review-patterns-ai-generated-code)  
21. The Copilot Era: How Generative AI Is Reshaping Quality Assurance Team Roles in 2025 and Beyond \- Qt, accessed January 20, 2026, [https://www.qt.io/how-generative-ai-is-reshaping-quality-assurance-team-roles-whitepaper](https://www.qt.io/how-generative-ai-is-reshaping-quality-assurance-team-roles-whitepaper)  
22. The State of AI Code Security in 2026: What You Need to Know Right Now : r/vibecoding, accessed January 20, 2026, [https://www.reddit.com/r/vibecoding/comments/1qa1pye/the\_state\_of\_ai\_code\_security\_in\_2026\_what\_you/](https://www.reddit.com/r/vibecoding/comments/1qa1pye/the_state_of_ai_code_security_in_2026_what_you/)  
23. 10 Open Source AI Code Review Tools Worth Trying, accessed January 20, 2026, [https://www.augmentcode.com/tools/open-source-ai-code-review-tools-worth-trying](https://www.augmentcode.com/tools/open-source-ai-code-review-tools-worth-trying)  
24. Linting versus other code quality tools \- Graphite, accessed January 20, 2026, [https://graphite.com/guides/linting-vs-other-code-quality-tools](https://graphite.com/guides/linting-vs-other-code-quality-tools)  
25. Best Automated Code Review Tools for Enterprise Software Teams \- Qodo, accessed January 20, 2026, [https://www.qodo.ai/blog/best-automated-code-review-tools-2026/](https://www.qodo.ai/blog/best-automated-code-review-tools-2026/)  
26. Reviewing code with Amazon Q Developer, accessed January 20, 2026, [https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/code-reviews.html](https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/code-reviews.html)  
27. 12 Best AI Code Reviewers for Infrastructure-as-Code in 2026 \- Panto AI, accessed January 20, 2026, [https://www.getpanto.ai/blog/iac-code-reviewers](https://www.getpanto.ai/blog/iac-code-reviewers)  
28. Risk-based Inspection (RBI) \- Inspectioneering, accessed January 20, 2026, [https://inspectioneering.com/tag/risk-based+inspection](https://inspectioneering.com/tag/risk-based+inspection)  
29. Risk-Based Inspection (RBI) & Corrosion Management \- MISTRAS Group, accessed January 20, 2026, [https://www.mistrasgroup.com/data-solutions/engineering/risk-based-inspection/](https://www.mistrasgroup.com/data-solutions/engineering/risk-based-inspection/)  
30. Diff Risk Score: AI-driven risk-aware software development \- Engineering at Meta, accessed January 20, 2026, [https://engineering.fb.com/2025/08/06/developer-tools/diff-risk-score-drs-ai-risk-aware-software-development-meta/](https://engineering.fb.com/2025/08/06/developer-tools/diff-risk-score-drs-ai-risk-aware-software-development-meta/)  
31. 5 Best Practices for Reviewing and Approving AI-Generated Code \- Bright Security, accessed January 20, 2026, [https://brightsec.com/blog/5-best-practices-for-reviewing-and-approving-ai-generated-code/](https://brightsec.com/blog/5-best-practices-for-reviewing-and-approving-ai-generated-code/)  
32. Best Practices for Human Oversight and Where Human Intervention Is Necessary in AI-Driven Processes \- Dialzara, accessed January 20, 2026, [https://dialzara.com/blog/human-oversight-in-ai-best-practices](https://dialzara.com/blog/human-oversight-in-ai-best-practices)  
33. Code Reviews in Large-Scale Projects: Best Practices for Managers, accessed January 20, 2026, [https://blog.codacy.com/code-reviews-best-practices](https://blog.codacy.com/code-reviews-best-practices)  
34. Why Code Quality Still Matters in the Era of AI \- Thinslices, accessed January 20, 2026, [https://www.thinslices.com/insights/why-code-quality-still-matters-in-the-era-of-ai](https://www.thinslices.com/insights/why-code-quality-still-matters-in-the-era-of-ai)  
35. Why Most AI Coding Tools Fail (And How They Succeed) \- DEV Community, accessed January 20, 2026, [https://dev.to/lofcz/why-most-ai-coding-tools-fail-and-how-they-succeed-i31](https://dev.to/lofcz/why-most-ai-coding-tools-fail-and-how-they-succeed-i31)  
36. AI in Performance Reviews: Balancing Technology and Human Insight in 2025 \- SkillCycle, accessed January 20, 2026, [https://www.skillcycle.com/blog/how-do-i-implement-ai-in-performance-reviews-without-losing-human-insight/](https://www.skillcycle.com/blog/how-do-i-implement-ai-in-performance-reviews-without-losing-human-insight/)

[image1]: <data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAALEAAAAYCAYAAAC1OhzjAAAFeUlEQVR4Xu2aachuUxSAl1nmefhBn3kefhgyFCHDDxJChh8S5Q+KEuKWzGRK5tyLUCglKUJJxggJKQkZQuZ5tp72Xvdd33r3Pu957+d+33u1n1q9Z6+1zt777LOHtfd5RRqNRqPRaDT+Fyyrsm1ULuGsoHKxyq0qOwfbksyaKqeq3Kaym9Ov765LHKFyu8rp0fBfc7XKdyr/ZPlF5SuVP5xuypwD81VOjMoefCGDvCeJY1W+kUHdfs/pb1X+zrpXF3oP2FeS7TKVpVSWUXks69YeuE3Dyhgl59kNc8AFkurwpcrRkvoBut9U1su2CM9Pe70ug37DAMD3+pyOPCHDz12TTmpOT0rSbxb0VBb95UHfF+sUk0itLZaTpP/a6T6XNOBL3CvlfAxmNezXREMG2z5ROUv8Kqn8LaJB2UOSjXfoOTnrtw56WFGS7cxocNTaHR6QejsvhJufi0plP0m2t6JhhpDnTVE5IVC3WoP5hrZZuwvsz0Zl5lFJ9tWiIfOKpLBrtmFg9nmuG1z6+KzbyekizOC1fFm9sL0cDZlVVZ6JSg/LKBkcEA2S4iBsC4J+JmwoKc8NomECoE7UjdCghHXis/LvdtPNQ5h/iZKNZdp4zV3X2D0qCpRmxhqEAdRpq2gI4LNWSL/v0iVsVS9xtiTbwUHPDA6rq1zkDZG3pZ55qaG3lxQPLx30npNU7lPZOxpkMDA816rsEnRzAasDdVslGpQzJNmoa6ldSnT5oY8zvveNL7QEM3Vc1j21skuwMcX/z2go4PO1PQ6zZRdvSPIrbXptX+ZhP7B5viaUW9nZhig1NAXxMB8E/RUqh0jahcd7DPRT+fpcGd6c+PKItVkmNnW6uaTUFjAlSf+UypX5ep53qFDLb1dJevIyLlH5zKX7sryUyyjpuvhQ0j37R8MIas8YMb81okGG89gmpEeCMycSxCMsJxa7MDIjH+XfN6VcyNMqL7o0PnR8DzqOXhgodHLTlfLzMNLvqcjdKndJWiHuVLlDUhnjYvVgZmAD93NOf6qyQ/BhdujiOEl+pY5ppxdRTvNOYxA78qi2LNHnHUQ4neKex6OhQC1/i4dL0guLh9nAed7Jeg8nElP5GhvLauR7Sbb5KhsFG1g8/JCkkGOSsHiYAdFF3wa2AbBXNEg5DwYNbbyoWEdGFiWfUp1GYaEoJxNdHCjJj81w5BxJtsOcbkeVB126k3elXHGOztD7jYZBYaV7wJYBk1jpW7L+4/xLelLgIwV12iQaAn1eNvuFLj/0fxV0M2FUmaPoey/n54Zt1kZtMO3Ibt1oUH6Q4XJPkdSXelGr+I+S9KURzUPEDUmEgJxlNObty1spXx81MHdCeEMMOY6MQ60tIn38CD/wWScalD0l2a6KhhlgHRhseR6XPs/F17sFLk1n454TnC7Chh2fR6Ih06fcTrj5+aiU7ozRH5mvWTK9ns7viXmQ9ufDpC0OjL6zTdcze/g4gd+W0ZAhzsfOElrCZq9Ru/m+2Icnz6J0ZPYS3LNxNDji6mHl2F4pYoOLQV3CQqCXoqEv50vK4NBokOEXatfMsHbNWak/qEfPSDV4WT6uwRcf/82dNGetLDOXOv1sQ9xKXe6Phgq1L45sMtF3hSSxbWdKLS8/O/eFvx1wT/xczn6htvra52mOXj1TWc+XyxrXSfI5JhpGcaOkOIR4lVMJvnXHEUaFyJwRxIP5WYMNCLb4xw4+ltjLRejknsOz3jMv60YdlC8uaDzfFqwktMWokwd4WFLd31P5JF9zaF/jJxmceiBc9zmT7eKgqAgwS3OcNw43y+Ad0jf4vXCaxzDExPgR+76Qrznpqn1LYLKIfZB7KbvRaDQajUaj0Wg0Go1Go9FoLF7+BfPr0ww9n9KrAAAAAElFTkSuQmCC>