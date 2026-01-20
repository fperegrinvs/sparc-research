# AI-friendly software architecture in 2025: The modular monorepo wins

**The verdict is clear from Q3 2025 research: AI coding assistants work dramatically better with modular monoliths in monorepos using unified full-stack codebases.** Context window limitations—not team preferences—now drive architecture decisions. The emerging consensus challenges a decade of microservices orthodoxy: when your development partner is an LLM, code consolidation beats distribution. This shift is significant enough that companies are restructuring codebases specifically to optimize for AI-assisted development.

The key insight from 2025 is deceptively simple: AI tools can only work with what they can "see." A typical enterprise monorepo spans several million tokens, while even the largest AI context windows max out at **1 million tokens**. This fundamental mismatch means architecture choices that scatter code across services or repositories directly cripple AI effectiveness.

---

## Modular monoliths outperform microservices for AI development

The most striking finding from late 2025 research is the strong pivot toward modular monoliths among teams heavily using AI coding tools. The reason is purely practical: AI assistants need to see the complete picture to make useful suggestions.

When working with a monolith, an AI can trace a feature from database schema to API endpoint to frontend component in a single context. As one developer noted in November 2025: "You can ask the AI to add a company_name field to the user sign-up flow, and it can trace the change from database migration to frontend form." With microservices, the same request fragments across repositories—the AI might update the user-service but "has no idea that the onboarding-service also needs to be changed."

**Quantitative evidence supports this pattern.** Testing by Augment Code on a 450,000-file codebase showed tools with full-repo understanding achieved **89% accuracy on multi-file refactoring** versus **62% for file-isolated tools**. That 27-point gap represents the difference between AI as a genuine productivity multiplier and AI as a source of subtle bugs.

The modular monolith sweet spot emerges when teams organize by business domains (Auth, Billing, Orders) rather than technical layers (Controllers, Models). A January 2026 analysis found that scoping AI conversations to individual modules—typically **30-50 files** versus **500+ in a full monolith**—achieves "excellent understanding." Before this approach, developers reported spending **2 hours** fixing AI-generated code that violated architectural patterns. After adopting module-scoped contexts, that dropped to **15 minutes**.

| Architecture | Files in AI Context | AI Comprehension | Cross-Cutting Changes |
|-------------|--------------------|-----------------|-----------------------|
| Full Monolith | 500+ | Poor - sees fragments | Excellent if context fits |
| Modular Monolith | 30-50 per module | Excellent | Excellent |
| Microservices | Varies by service | Good within service | Poor across services |

Industry consensus now suggests teams under **10 developers should default to monoliths**. Java Code Geeks reported in December 2025 that "major tech companies like Amazon Prime Video have actually moved back to monolithic architectures for specific use cases." The recommendation: extract services only when hitting genuine pain points—team contention, resource constraints, or regulatory requirements—not preemptively.

---

## Monorepos dramatically improve AI tool effectiveness

Repository structure matters as much as architecture. The 2025-2026 evidence strongly favors monorepos for AI-assisted development, primarily because they eliminate what researchers call "contextual fragmentation."

In a monorepo, AI agents can see the authentication service, API gateway, and client implementations simultaneously. This enables what monorepo.tools calls **"atomic cross-project changes"**—an AI can update a database schema, modify the backend API, add a frontend component, and adjust mobile app logic in a **single commit**. With polyrepos, this becomes "a coordination nightmare requiring multiple pull requests, careful timing, and manual synchronization."

**The dependency advantage is substantial.** Monorepo tools like Nx build project graphs that provide AI with architectural maps of the entire codebase. This means AI can answer questions like "Which shared libraries does this service depend on?" and trace impact across the system. In polyrepos, AI lacks visibility into how changes in one repository affect dependent services.

However, monorepos present their own AI challenges. Research on "context rot" reveals that even million-token context windows suffer **performance degradation with longer inputs**. One study found that 65% of developers experience missing context during AI-assisted refactoring, suggesting that simply having all code available doesn't guarantee AI comprehension.

**Emerging solutions address these limitations.** The Model Context Protocol (MCP), which became an industry standard in late 2025 after adoption by OpenAI, Anthropic, and Mistral, provides structured workspace access. Nx's official MCP server gives AI tools project graphs, dependency information, and task intelligence without requiring the AI to scan all files. Nx users report that AI-assisted migrations now increase completion rates from **80% to near 100%**, reducing "multi-week refactoring efforts to hours."

For teams committed to polyrepos, new tools are closing the gap. Moderne's multi-repo agent "Moddy" can analyze billions of lines across thousands of repositories using a Lossless Semantic Tree. Nx Polygraph connects multiple repositories into a "federated meta-repo." Shell-based workarounds allow AI agents to access files across repositories via explicit paths—a practical solution that requires zero changes to existing repo structures.

---

## Unified full-stack codebases enable smarter AI suggestions

The frontend/backend separation question follows the same pattern: unified codebases win because AI needs visibility across the entire feature implementation.

**The context advantage is measurable.** Developer reports from ts-rest + Next.js implementations describe a "drastic bug reduction" when AI can see both frontend and backend simultaneously. The type-safe contract "virtually eliminated a whole class of common and frustrating frontend-to-backend integration errors." More critically, the AI became "far better at catching errors, flagging incompatible data types between frontend and backend because it could see both sides of the conversation."

Consider the alternative. When frontend and backend live in separate repositories, an AI asked to "fix the bug where user orders aren't showing in the dashboard" will examine frontend code, but cannot see backend API responses, database queries, or service-to-service communication. It makes assumptions about root causes and "implements a frontend workaround instead of fixing the backend issue."

**TypeScript emerges as the enabling technology for full-stack AI development.** As Builder.io noted in January 2026: "TypeScript's types tell the AI exactly what your functions expect, what your components accept, and what your data structures look like. Without types, the AI is guessing. With types, it's reading a spec."

Framework-specific patterns reinforce this finding:

- **Next.js** leads for AI-assisted full-stack development. Its API routes provide a built-in backend in the same codebase, shared TypeScript types work across frontend/backend, and server components enable unified data flow. Vercel's v0 AI is specifically optimized for Next.js patterns.
- **tRPC** eliminates API contracts entirely. One team reported: "Same app, refactored in 3 days. Zero API contracts. Zero code generation. If the backend compiles, the frontend is guaranteed to work."
- **Nx with MCP integration** provides metadata-driven generators that are "inherently LLM-friendly" for larger organizations maintaining multiple applications.

For teams that cannot migrate to unified codebases, practical workarounds exist. The "Root Repository Solution" involves creating a coordination layer with an Agents.md file describing all repos, a clone script to pull them into a consistent structure, and MCP configuration. Opening this root directory in an AI coding assistant provides full context while requiring zero changes to existing repositories.

---

## New architectural patterns have emerged specifically for AI development

Beyond choosing monoliths over microservices, 2025 saw the emergence of entirely new patterns designed to optimize codebases for AI tools.

**CLAUDE.md and AGENTS.md files** have become essential. These markdown files at repository roots provide AI assistants with project context—build commands, code style conventions, testing instructions, and architectural guidelines. Anthropic's guidance describes CLAUDE.md as "the most important tool you have for guiding the AI." The files are "naively dropped into context up front," meaning they shape every AI interaction.

Best practices for these files have crystallized:

- Keep content to approximately **50 instructions** for reliable following
- Define the project's WHY, WHAT, and HOW—not code style (that's the linter's job)
- Use hierarchical files in monorepos: root directory, parent directories, and child directories each get their own context
- Keep contents concise—one team reduced their CLAUDE.md from **47,000 words to 9,000** by splitting context across services

**The llms.txt standard** has gained significant traction, with over **600 sites** adopting it by July 2025. Proposed by Jeremy Howard, this machine-readable documentation index lives at `/llms.txt` in the domain root and provides AI tools with structured access to documentation. Adopters include Anthropic, Cursor, Stripe, Perplexity, Hugging Face, and Cloudflare.

**Spec-driven development** represents a more fundamental shift. Rather than code-first development, specifications become the source of truth. The spec defines intent; AI generates code as an artifact. This prevents "vibe coding" chaos—the phenomenon where 25% of Y Combinator Winter 2025 startups had codebases **95% AI-generated** without architectural rigor.

**Context engineering** has emerged as a distinct discipline. Andrej Karpathy defined it as "the delicate art and science of filling the context window with just the right information for the next step." Core strategies include:

- **Write**: Persist key information outside context (scratchpads, files)
- **Select**: Retrieve relevant memories/documents dynamically  
- **Compress**: Summarize or trim context to essentials
- **Isolate**: Split context across sub-agents for focused tasks

The hierarchy of development effort has fundamentally shifted—research and planning now deliver exponentially greater returns than focusing on implementation. Time spent crafting context yields better outcomes than time spent reviewing generated code.

---

## Real-world results show architecture determines AI effectiveness

The most revealing data comes from actual developer experiences, which paint a nuanced picture that challenges both AI skeptics and enthusiasts.

**The landmark METR study from July 2025** delivered a surprising finding: experienced developers were **19% slower** with AI tools despite believing they were 20% faster. The randomized controlled trial involved 16 experienced developers completing 246 real-world tasks on repositories averaging over 1 million lines of code. Developers predicted AI would speed them up by 24%; actual results showed the opposite.

Contributing factors included extra cognitive load from context-switching, time spent prompting and reviewing AI outputs, and low AI reliability requiring double-checking. Researchers described AI tools as "like a new contributor who doesn't yet understand the codebase"—a characterization that directly connects to architecture: in well-structured monorepos, that "new contributor" can learn quickly; in scattered microservices, they remain perpetually confused.

**The Puzzmo case study offers a counterpoint.** Game developer Orta Therox accomplished remarkable results in 6 weeks using Claude Code solo: converting hundreds of React Native components to React, replacing three non-trivial RedwoodJS systems, migrating from Jest to Vitest, and building iPad support. The key? A monorepo with "boring, explicit technologies" like React, Relay, GraphQL, and TypeScript.

His explanation: "A monorepo is perfect for working with an LLM, because it can read the file which represents our schema, it can read the SDL files defining the public GraphQL API, read the per-screen requests and figure out what you're trying to do."

**Success factors that consistently emerge across case studies:**

- Monorepo structure providing single source of truth
- Explicit, typed technologies (TypeScript, GraphQL) that offer validation
- Strong test suites enabling AI to verify its work
- Standard patterns (CRUD, well-documented frameworks) rather than cutting-edge approaches
- Deep developer expertise—AI "amplifies your expertise" rather than replacing it

The pattern is clear: architecture determines whether AI tools become productivity multipliers or sources of subtle bugs requiring extensive review.

---

## Tool-specific recommendations have solidified

Different AI coding tools have distinct strengths that interact with architecture choices. Understanding these can guide both tool selection and codebase structure.

**Claude Code** excels at autonomous, multi-file operations and large-scale refactoring. Its terminal-first approach and true **200,000-token capacity** make it particularly effective for CLI workflows and monorepo navigation. The tool supports async sub-agents for parallel work and is "exceptionally good at navigating large codebases, searching for patterns" according to Builder.io testing.

**Cursor** with its v2.1 "Instant Grep" feature makes agent-driven search instant in large monorepos. Its semantic search understands symbols and relationships, not just text. Multi-model flexibility allows switching between GPT-4, Claude, and custom models. Cursor passed **$1 billion in annual revenue** by December 2025, with **53% of Fortune 1000 companies** having engineers using it.

**GitHub Copilot** introduced Copilot Spaces at Universe 2025—curated knowledge bases for specific areas of monorepos—and Agent HQ, a unified platform for multiple AI agents. Its **64,000-token context window** is smaller than competitors but sufficient for well-scoped tasks.

**Sourcegraph Cody** uses full codebase indexing via vector embeddings, enabling natural language navigation of entire monorepos. It's "powerful for monorepos—you can navigate code via natural language."

**Augment Code** demonstrated superior performance in a 450,000-file monorepo test, building semantic dependency graphs and maintaining cross-service context. Its Context Engine indexed the entire repo in **27 minutes** with incremental updates under 20 seconds.

The consistent recommendation across tools: structure matters more than raw context size. Providing AI with architectural maps—through MCP servers, project graphs, and well-crafted context files—yields better results than simply expanding context windows.

---

## Conclusion

The 2025-2026 research establishes clear guidance for AI-friendly architecture: **modular monoliths in monorepos using unified full-stack codebases with TypeScript** represent the optimal configuration for AI-assisted development. This isn't about following trends—it's a direct response to the fundamental limitation that AI tools can only work effectively with code they can see and understand in context.

The key insight reshaping software architecture is that context is the bottleneck. Million-token context windows still fall short of enterprise codebases. Microservices scatter context across service boundaries. Polyrepos fragment it across repositories. Separated frontend/backend projects split it across concerns. Each architectural decision that distributes code trades AI comprehensibility for other benefits.

For new projects, the path is clear: start with a modular monolith, use a monorepo, adopt full-stack TypeScript frameworks like Next.js, and invest in CLAUDE.md/AGENTS.md context files from day one. For existing projects, practical migration paths exist—from the Root Repository Solution that provides AI context without restructuring, to gradual modularization that improves AI effectiveness incrementally.

The most successful developers in 2025 are those who recognize that AI tools are "like a capable junior developer requiring supervision"—powerful enough to execute complex changes, but dependent on clear architectural guidance and comprehensive context to do so effectively.