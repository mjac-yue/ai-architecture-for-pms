# AI Architecture for Product Managers

A practical course covering the technology behind AI products — not to make you an engineer, but to make you a better decision-maker when building with AI.

---

## Who this is for

You're a PM who ships AI-powered features. You don't need to write the code, but you need to understand what's possible, what's hard, what costs money, and what to ask your engineers.

---

## Core mental models

Before starting, internalise these four ideas. They reframe how you'll read everything that follows.

**AI is a probabilistic component.** Unlike a database query that returns the same result every time, an LLM produces different outputs for the same input. Design for this — don't fight it.

**The model is not your product.** The model is infrastructure. Your product is the system around it: what data you give it, how you constrain its behaviour, how you handle failures.

**Garbage in, garbage out — but worse.** With databases, bad input returns an error. With AI, bad input returns a confident-sounding wrong answer. Validation and evaluation matter more, not less.

**You are shipping a system, not a feature.** An AI feature involves a model, a prompt, data retrieval, output handling, fallbacks, and monitoring. Spec all of it.

---

## How to use this course

Read the modules in order the first time. Return to individual modules as reference when making product decisions. Each module ends with a **PM Decision Checklist** — use those in planning and spec reviews.

When starting any AI feature, open the **[AI PM Playbook](ai-pm-playbook.md)** — it maps the full product lifecycle (discover → spec → build → launch → measure) with stage-by-stage checklists and module references. The **[Quick Reference](quick-reference.md)** is for fast lookups during planning sessions and spec reviews.

---

## Modules

| # | Module | What you'll be able to do after |
|---|--------|----------------------------------|
| **Part 1: Orientation** | | |
| 1 | [Why AI Products Exist](module-01-why-ai-products-exist.md) | Articulate the capability shift LLMs created; identify the value pattern of any AI feature; apply the AI rightness test |
| 2 | [The AI Product Landscape](module-02-ai-product-landscape.md) | Map any AI product to its category; understand competitive dynamics and where defensibility comes from |
| 3 | [How LLMs Actually Work](module-03-how-llms-work.md) | Explain context windows, tokens, and model tiers to stakeholders; set realistic expectations |
| 4 | [Multimodal AI](module-04-multimodal-ai.md) | Know what's production-ready across vision, audio, and video; spec multimodal features correctly |
| 5 | [AI Product Management](module-05-ai-product-management.md) | Run the full AI PM lifecycle: opportunity identification, problem framing, roadmapping, stakeholder communication |
| **Part 2: Architecture** | | |
| 6 | [The AI Product Stack](module-06-ai-product-stack.md) | Map any AI feature to its architecture layers; know where complexity lives |
| 7 | [RAG and Context Architecture](module-07-rag.md) | Choose between 5 context strategies; design tiered resolution; spec retrieval correctly |
| 8 | [Agents and Tool Use](module-08-agents-tool-use.md) | Define agentic features; know when agents are overkill; diagnose using the Agent Development Stack |
| 9 | [Multi-Agent Architecture](module-09-multi-agent-architecture.md) | Recognise when single-agent breaks down; choose between 5 coordination patterns; reason about agent isolation |
| **Part 3: PM Craft** | | |
| 10 | [Prompt Engineering for PMs](module-10-prompt-engineering.md) | Write AI behaviour specs engineers can build from; review prompts in PRs |
| 11 | [AI UX Patterns](module-11-ai-ux-patterns.md) | Design AI interactions that build trust; know when not to use AI |
| 12 | [AI Safety & Responsible AI](module-12-ai-safety-responsible-ai.md) | Identify and mitigate AI harms; run red-team exercises; understand the regulatory landscape |
| **Part 4: Strategy** | | |
| 13 | [Build vs. Buy for AI Features](module-13-build-vs-buy.md) | Make principled build/buy decisions; evaluate vendors on quality, data risk, and cost |
| 14 | [Cost, Latency & Model Selection](module-14-cost-latency.md) | Make informed model-tier decisions; estimate AI infrastructure cost |
| 15 | [Data Strategy for AI Products](module-15-data-strategy.md) | Build a data flywheel; identify your data moat; govern AI data correctly |
| **Part 5: Execution** | | |
| 16 | [Working with AI Engineering Teams](module-16-working-with-ai-engineers.md) | Write specs engineers can build from; estimate AI work; manage prompt and model changes safely |
| 17 | [Evaluating AI Features](module-17-evaluation.md) | Write acceptance criteria for AI; run basic evals before launch |
| 18 | [Observability & Monitoring](module-18-observability-monitoring.md) | Instrument AI features for debugging; detect quality drift; manage cost in production |
| 19 | [Measuring AI Product Success](module-19-measuring-ai-product-success.md) | Define AI quality metrics; instrument features correctly; connect AI performance to business outcomes |
| 20 | [Production Readiness & Operations](module-20-production-readiness-operations.md) | Define go/no-go bars; design rollouts and kill switches; run AI features long-term |

---

## Appendix

| Document | Purpose |
|----------|---------|
| [AI PM Playbook](ai-pm-playbook.md) | Full product lifecycle workflow with stage-by-stage checklists — open this when starting any AI feature |
| [Quick Reference](quick-reference.md) | Fast lookup: decision trees, pattern selector, cost estimates, prompt checklist, glossary |
| [Anti-Patterns](anti-patterns.md) | Catalogue of common AI PM mistakes by lifecycle phase, with fixes |

---

## Getting the materials

**Clone the repo:**
```bash
git clone https://github.com/mjac-yue/ai-architecture-for-pms.git
cd ai-architecture-for-pms
```

**Download without git** (ZIP):
Go to [github.com/mjac-yue/ai-architecture-for-pms](https://github.com/mjac-yue/ai-architecture-for-pms) → **Code** → **Download ZIP**, then unzip.

All modules are plain Markdown files. Open them in any Markdown viewer, editor (VS Code, Obsidian, Notion import), or read directly on GitHub — diagrams render automatically there.
