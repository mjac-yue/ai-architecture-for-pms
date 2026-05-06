# Quick Reference — AI Architecture for PMs

Use this in planning sessions, spec reviews, and design critiques.

---

## Architecture pattern selector

| User does this | Pattern | Modules |
|---------------|---------|---------|
| Asks a question about your docs/data | RAG-powered Q&A | 3 |
| Uploads a document for analysis | Single-call augmentation | 1, 2 |
| Requests a multi-step task ("research and write") | Agentic workflow | 4 |
| Needs real-time data (prices, live status) | Tool use + direct API | 4 |
| Needs to classify/categorize input | Single call, Haiku tier | 1, 7 |
| Needs generated content (drafts, summaries) | Single call, Sonnet | 1, 5 |

---

## Model tier selector

```
Default → Sonnet

High volume + simple task (classify, extract)?
  → Test Haiku. Use if quality is acceptable.

Complex reasoning, nuanced judgment, high stakes?
  → Test Sonnet. Use Opus only if Sonnet fails.

Cost is a problem?
  → Drop one tier and test quality.
  → Enable prompt caching for large system prompts.
  → Add output length constraints.
  → Move background tasks to Batch API (50% discount).
```

---

## Cost estimation (Sonnet baseline, 2025)

| Scenario | Approx cost/request |
|----------|-------------------|
| Simple Q&A (500 in, 150 out) | ~$0.004 |
| Document summary (3K in, 400 out) | ~$0.016 |
| RAG answer (4K in with docs, 300 out) | ~$0.017 |
| Agent step (2K in, 500 out) × 5 steps | ~$0.045 |

Monthly cost = (cost/request) × (daily requests) × 30

---

## Prompt system checklist (5 elements)

- [ ] **Role** — Who is the AI, what domain does it operate in?
- [ ] **Task** — What should it do? What are the boundaries?
- [ ] **Constraints** — What must it never do?
- [ ] **Format** — Prose, JSON, structured text? Include example.
- [ ] **Examples** — 2–3 few-shot examples for complex behaviors.

---

## Spec completeness check for AI features

Every AI feature spec should answer:

| Question | If not answered |
|----------|----------------|
| What model tier and why? | Engineers will default to whatever's easiest |
| What's in context (prompt + data)? | Unclear scope, unreliable behavior |
| What's the output format? | Parsing bugs in production |
| What happens when the model doesn't know? | Confident hallucinations |
| What are the "must not" constraints? | Safety failures |
| How do we evaluate quality? | No definition of done |
| What are the edge cases? | Failures discovered in prod |
| What's the cost estimate at volume? | Budget surprises |

---

## Retrieval (RAG) decision

Use RAG when:
- Users need to query your private/proprietary data
- Knowledge changes faster than model training cycles (always)
- You need verifiable, citable answers

Don't use RAG when:
- Data is real-time (use tool calls to live systems instead)
- Data fits in the system prompt without degrading quality
- The question is about the model's general capabilities, not your data

---

## Agent complexity check

Before spec'ing an agent, verify:
- A single model call can't handle this → ✓ Agent needed
- Multiple tools are required → ✓ Agent needed
- The path through the task isn't fully known upfront → ✓ Agent needed
- Any reversible action is in scope → Require human-in-the-loop
- Max model calls per user action is defined → Budget and guardrails needed

---

## Eval requirements for launch

Minimum viable eval:
- [ ] 30+ representative inputs with expected outputs
- [ ] 10+ edge case inputs
- [ ] 5+ adversarial inputs (jailbreak attempts, out-of-scope requests)
- [ ] Scoring rubric defined
- [ ] Process for running evals before prompt/model changes
- [ ] Thumbs up/down in UI with logging

---

## The 10 things most AI features get wrong

1. No output format spec → parsing breaks in production
2. No "I don't know" instruction → confident hallucinations
3. No scope boundary → users try things AI can't do
4. No human-in-the-loop for irreversible actions → trust failures
5. Opus used for everything → unnecessary cost
6. No prompt caching for large stable context → unnecessary cost
7. No streaming → perceived latency is bad
8. No feedback mechanism → can't detect or improve
9. No eval plan → launch based on vibes
10. AI where a database query would do → complexity without value

---

## Glossary

| Term | Plain English |
|------|--------------|
| **Token** | ~4 characters; the unit of cost and context |
| **Context window** | Total text the model can see at once |
| **RAG** | Finding relevant docs before the model call |
| **Embedding** | Converting text to numbers for semantic search |
| **Vector database** | Database optimized for similarity search |
| **Tool use** | Model requesting an action from an external system |
| **Agent** | AI system that takes multiple steps and uses tools |
| **MCP** | Standard for connecting AI models to tools |
| **Fine-tuning** | Updating model weights on new training data |
| **Prompt injection** | User input that attempts to override system instructions |
| **Temperature** | Controls output randomness (0=deterministic, 1=varied) |
| **Streaming** | Returning tokens as they generate, not waiting for completion |
| **Prompt caching** | Storing repeated context to reduce cost |
| **Batch API** | Async, lower-cost processing for non-real-time workloads |
| **Hallucination** | Confident, plausible-sounding output that is factually wrong |
| **Eval** | Structured test of AI quality on known inputs |
| **LLM-as-judge** | Using a model to evaluate another model's output |
