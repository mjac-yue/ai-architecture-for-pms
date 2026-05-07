# Quick Reference — AI Architecture for PMs

Use this in planning sessions, spec reviews, and design critiques.

---

## AI opportunity filter
*Run every potential AI feature through these before adding it to the roadmap. (→ Modules 1, 5)*

- [ ] Is there a **language or reasoning task** at the core — not just data display or processing?
- [ ] Would a **skilled human doing this manually** be genuinely valuable?
- [ ] Is the **error tolerance acceptable** — can this be occasionally wrong?
- [ ] Does AI **beat the non-AI alternative** meaningfully — not just marginally?

If any answer is no, reconsider before building.

**Always run a spike first** (1–3 days): validate the model can do this task on your actual data before committing to a timeline.

---

## Architecture pattern selector

| User does this | Pattern | Modules |
|---------------|---------|---------|
| Asks a question about your docs/data | RAG-powered Q&A | 7 |
| Uploads a document for analysis | Single-call augmentation | 3, 6 |
| Requests a multi-step task ("research and write") | Agentic workflow | 8, 9 |
| Needs real-time data (prices, live status) | Tool use + direct API | 8 |
| Needs to classify/categorize input | Single call, Haiku tier | 3, 14 |
| Needs generated content (drafts, summaries) | Single call, Sonnet | 3, 10 |

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

## Build vs. buy quick guide
*Full framework in Module 13. Use this for fast triage.*

| Capability | Default recommendation |
|-----------|----------------------|
| Base LLM API | Always buy |
| Embeddings / vectorisation | Buy |
| Vector database | Buy (pgvector if already on Postgres) |
| Transcription | Buy (Whisper or AssemblyAI) |
| Document parsing | Buy (pdfplumber, Unstructured.io) |
| Content moderation | Buy (Perspective API, provider built-ins) |
| Observability / tracing | Buy |
| Retrieval logic + ranking | Build — close to core differentiation |
| System prompts | Always build — this is your product behaviour |
| Eval criteria + datasets | Always build — your quality bar is unique |

**Build when:** core to differentiation, vendor quality insufficient, or data sensitivity prevents external use.  
**Buy when:** commodity capability, not on the critical path, vendor quality tested and acceptable.  
**Wait when:** the capability is about to commoditise — check back in 6 months.

---

## Safety harm checklist
*Use during spec and pre-launch red-team. Full coverage in Module 12.*

- [ ] **Hallucination** — Is the feature making factual claims? Are sources/RAG in place?
- [ ] **Bias** — Does the feature evaluate or rank people? Is demographic testing planned?
- [ ] **Privacy** — Is access control enforced at the retrieval layer, not just trusted to the model?
- [ ] **Harmful content** — Is there a content moderation layer for open-ended outputs?
- [ ] **Prompt injection** — Can user input or retrieved content hijack the system prompt?
- [ ] **Overreliance** — Does the UI signal that outputs need review, not blind trust?

**Red-team minimum:** 5+ adversarial inputs per harm category before launch. Add all failures to the eval dataset.

---

## AI product metrics cheat sheet
*Full definitions in Module 19.*

| Metric | Formula / method | What a change signals |
|--------|-----------------|----------------------|
| **Acceptance rate** | Accepted outputs ÷ total outputs shown | Overall quality signal |
| **Regeneration rate** | Regenerate clicks ÷ total outputs | User didn't get what they needed |
| **Edit rate** | Edits after acceptance ÷ accepted outputs | Output needed significant correction |
| **Thumbs down rate** | Negative ratings ÷ total outputs | Direct dissatisfaction signal |
| **Task completion rate** | Tasks completed with AI vs. without | Core utility of the feature |
| **Escalation rate** | AI hand-offs to human ÷ total sessions | AI coverage declining or query complexity rising |

**Instrument from day one** — interaction events (accept, edit, regenerate, dismiss, rating) cannot be added retroactively.

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

## Multi-agent decision (→ Module 9)

**Stay single-agent unless you see one of these signals:**
- Context window saturation
- Role confusion (one agent has too many responsibilities)
- Tool sprawl (agent picks wrong tool with too many options)
- Quality degradation on long multi-step tasks
- Independent subtasks that benefit from parallel execution
- Different parts need different permission levels

**5 coordination patterns:**
| Pattern | Best for | Cost / latency |
|---------|----------|----------------|
| Sequential Handoff | Pipeline tasks | Low cost, high latency |
| Fan-out / Fan-in | Independent parallel subtasks | High cost, low latency |
| Supervisor / Worker | Open-ended planning tasks | Medium / medium |
| Debate / Critique | Quality-critical outputs | High / high |
| Hierarchical Delegation | Almost never the answer | Very high / very high |

---

## Observability minimum (→ Module 18)

For every AI request, log:
- [ ] Input
- [ ] Retrieved context (RAG chunks, tool results)
- [ ] Model response (raw, before post-processing)
- [ ] Latency (per component, end-to-end)
- [ ] Cost (input + output tokens, model used)
- [ ] User feedback (thumbs / edits / regenerations)

**Alert thresholds:** error rate >1%, P95 latency above budget, safety filter spike, cost above forecast, empty response rate >0.5%.

---

## Production launch gate (→ Module 20)

Define before shipping:
- [ ] Quality bar as a number on a defined dataset (e.g. "≥75% acceptance on n=50 eval set")
- [ ] Hallucination ceiling for factual features (e.g. "≤3% on n=100 sample")
- [ ] Latency budget (P95)
- [ ] Cost budget per request at expected volume
- [ ] Zero unresolved critical safety findings
- [ ] Kill switch implemented and **tested in production**
- [ ] Staged rollout plan: internal → beta → 5% → 25% → 50% → 100%
- [ ] Named owner for ongoing weekly/monthly/quarterly maintenance

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

## Decision trees

### Should this be AI?

- Does the task require understanding natural language or unstructured input?
  - Yes → Does the correct response vary by context (not a fixed lookup)?
    - Yes → Is a "good enough" answer acceptable (not exact)?
      - Yes → AI is appropriate
      - No → AI may work, but requires strong guardrails and explicit quality bar
    - No → Could be solved with search or lookup — consider a simpler approach first
  - No → Is the logic too complex to express as rules?
    - Yes → AI might help; evaluate a hybrid approach with deterministic fallback
    - No → Use deterministic logic — don't add AI for AI's sake

### Model tier selection

- What is the task complexity?
  - Simple (classification, routing, extraction, yes/no) → Fast tier (Haiku / GPT-4o mini)
  - Moderate (generation, analysis, multi-step with guidance) → Balanced tier (Sonnet / GPT-4o)
  - Complex (novel reasoning, ambiguous problems, creative synthesis) → Frontier tier (Opus / GPT-4)
- Refinement factors — adjust tier after initial selection:
  - Volume is high → downgrade tier (cost scales linearly)
  - Stakes are high (mistakes expensive or dangerous) → upgrade tier
  - Latency budget is tight (< 500ms) → downgrade tier
  - Eval scores fail at lower tier → upgrade tier
  - Lower tier passes eval at acceptable quality → stay or downgrade

### Context architecture choice

- How much knowledge does the skill need?
  - Small (< 5 pages of rules/facts) → Static context (always loaded in system prompt)
  - Medium (5–50 pages, structured) → Indexed lookup (YAML/JSON keyed retrieval)
  - Large (50+ pages, or growing, prose) → RAG / vector database
- Is every piece of context needed for every query?
  - Yes → Load everything (Tier 1 static). Accept the token cost.
  - No → Use tiered resolution:
    - Tier 1: Always loaded (identity, universal rules)
    - Tier 2: Loaded on demand (domain knowledge matched to query type)
    - Tier 3: Retrieved by search (large knowledge base, specific facts)
- Is the knowledge structured with known keys, or prose with unpredictable queries?
  - Structured with known keys → Indexed lookup
  - Prose, unpredictable queries → RAG
  - Real-time or live data → Tool call to live system

### Guardrail strategy

- What is the failure mode?
  - Predictable pattern (PII, credentials, profanity, injection strings) → Deterministic hook (regex, blocklist, pattern match) — zero latency cost
  - Semantic failure (hallucination, tone, relevance, off-brand) → AI-based guardrail (model-as-judge on output) — adds latency and cost
  - Behavioral failure (wrong tool use, unauthorized action, loop) → Deterministic hook + circuit breaker at PreToolUse layer
- By product type:
  - Consumer product, open-ended input → prioritize input injection defense + output content scan
  - Enterprise internal tool → prioritize data leakage (PII/credentials) + tool access control
  - High-stakes domain (legal, medical, financial) → add AI-based hallucination check on every factual claim
  - Low-stakes utility (drafting, summarization) → deterministic format check sufficient to start

### Multi-agent vs. single agent

- Does the task require multiple distinct capabilities?
  - No → Single agent, single skill. Keep it simple.
  - Yes → Can a single agent handle all capabilities with good quality?
    - Yes (fewer than 5 skills, shared context) → Single agent with multiple skills and a router
    - No → Multi-agent required. Choose pattern:
      - Tasks are independent → Fan-out / Fan-in (parallel execution)
      - Tasks build on each other sequentially → Sequential handoff (pipeline)
      - One task needs to oversee and review others → Supervisor / Worker
      - Multiple levels of delegation needed → Hierarchical delegation (use sparingly)
- Additional signals that force multi-agent:
  - Context window too small to hold all skills' knowledge
  - Different skills need different model tiers
  - Different teams own different skills (organizational boundary = agent boundary)
  - Skills have incompatible or dangerous tool sets that must be isolated

### Evaluation strategy

- What aspect of quality are you evaluating?
  - Format (structure, length, schema compliance) → Automated checks (regex, JSON parse, length count) — run on every request in production
  - Factual accuracy (correct information, no hallucination) → Model-as-judge with reference answer + deterministic checks — run eval suite on every code or prompt change
  - Reasoning quality (logic, completeness, insight) → Model-as-judge with detailed rubric — run eval suite on every change
  - User satisfaction (helpful, appropriate, trustworthy) → Human evaluation + production feedback signals — monthly human eval cycle + continuous monitoring
  - Safety (harmful content, data leakage, injection) → Deterministic checks + adversarial eval cases — deterministic on every request, adversarial on every change
- What stage is the product at?
  - Prototype → 10–20 golden examples minimum
  - Beta → 50–100 cases (golden + edge cases + production failures)
  - Production → 100–500 cases (all above + adversarial + cross-domain regression)
  - Platform → 500+ cases with per-domain suites and routing eval

### Launch strategy / rollout gate criteria

- What is the blast radius if the feature fails?
  - Low (internal users, non-critical workflow) → Ship to all internal users; monitor basic quality dashboard
  - Medium (external users, non-critical feature) → Phased rollout: 1% → 10% → 50% → 100%; advance only after 24–48 hours with no quality degradation
  - High (external users, critical workflow, regulated domain) → Shadow mode first → opt-in beta → phased rollout with human review at each gate
- Gate criteria before advancing each rollout phase:
  - Eval scores meet defined quality bar
  - Error rate below 5%
  - Guardrail fire rate below 10%
  - Cost per request within 2× estimate
  - No P0 safety issues open
  - Rollback plan tested and named owner assigned

### Cost optimization

- Where is the highest cost?
  - Input tokens (large prompts / context) → Context tiering: load less per request (20–40% savings); prompt compression (10–20% savings)
  - Output tokens (long responses) → Output length constraints (5–15% savings); structured output to force concise formats
  - Model tier (expensive model for all queries) → Model routing: route simple queries to Fast tier (40–70% savings) — implement this first
  - Volume (too many requests) → Response caching for repeated queries (10–30% savings); batch processing for non-real-time work (20–30% savings)
- Optimization order (highest impact, lowest risk first):
  - 1. Model routing (simple queries to Fast tier)
  - 2. Context tiering (load only what each query needs)
  - 3. Response caching (repeated queries)
  - 4. Prompt compression (shorten system prompt)
  - 5. Output constraints (shorter responses)

### When to iterate vs. ship vs. stop

- Do eval scores meet the quality bar?
  - Yes → Are guardrails in place for known failure modes?
    - Yes → Ship. Improve from production signals.
    - No → Add guardrails, then ship.
  - No → Is the last iteration showing meaningful improvement (> 2%)?
    - Yes → Keep iterating — you are still learning
    - No → Diagnose root cause:
      - Stuck on context → Add more or better knowledge
      - Stuck on reasoning → Try a higher model tier
      - Stuck on scope → Reduce scope (narrow the skill)
      - Fundamentally limited → Reassess whether AI is the right approach
- When to stop investing entirely:
  - 3+ iterations with < 2% improvement (diminishing returns)
  - Remaining failures are all edge cases (< 5% of queries) — ship with guardrails
  - Improvement requires a fundamentally different architecture — decide: pivot or accept
  - Cost of further improvement exceeds the value it would deliver
  - User testing shows current quality is "good enough" — ship and invest elsewhere

---

## Red flags

| Signal | Threshold | What it likely means | Immediate action |
|--------|-----------|---------------------|-----------------|
| Token fill rate (context approaching full) | > 80% of context window used | Tiered resolution is missing; static context is too large | Implement context tiering; audit what is loaded on every request |
| Eval score improving but specific failure cases worsening | Any regression on previously passing cases | Overfitting the prompt to recent failures; earlier behaviors broken | Run full eval suite; treat regression as a blocker before shipping |
| P95 latency above budget | > 10 minutes sustained | Model provider degradation, context size growth, or routing failure | Check provider status; review recent context size changes; alert on-call |
| Cost per request above baseline | > 2× baseline | Context bloat, model tier regression, caching miss, or abuse pattern | Review per-request token logs; check caching hit rate; inspect recent prompt changes |
| Regeneration rate | > 20% | Output quality not meeting user expectations | Investigate failure patterns; add cases to eval suite; review top regenerated queries |
| Safety filter trigger rate | > 5% | Over-sensitive guardrails or underlying input quality issue; possible abuse | Audit triggered cases; tune thresholds; check for coordinated adversarial input |
| Empty response rate | > 1% | Guardrail over-blocking, context overflow, or model refusal pattern | Review empty response logs; check guardrail false positive rate; inspect context sizes |
| Acceptance rate drop | > 5% week-over-week | Quality regression, model drift after provider update, or scope mismatch | Compare recent model version; re-run eval suite; review what changed in the past week |

---

## Glossary

| Term | Plain English |
|------|--------------|
| **Acceptance rate** | % of AI outputs users accept without modification — primary quality proxy |
| **Agent** | AI system that takes multiple steps and uses tools |
| **Batch API** | Async, lower-cost processing for non-real-time workloads |
| **Blast radius** | The set of system components affected if a given change or failure occurs |
| **Calibrated trust** | UI and UX design that helps users develop accurate mental models of when to trust AI output and when to verify |
| **Compound AI system** | A system that combines multiple AI models, tools, and retrieval mechanisms to accomplish tasks no single model call could handle |
| **Constitution** | The foundational layer of an agent's system prompt: role, purpose, scope, and behavioural boundaries |
| **Context window** | Total text the model can see at once |
| **Data flywheel** | Self-reinforcing cycle: more users → more data → better AI → more users |
| **Embedding** | Converting text to numbers for semantic search |
| **Eval** | Structured test of AI quality on known inputs |
| **Eval-driven development** | The practice of writing eval cases before or alongside building AI features, analogous to test-driven development |
| **Fine-tuning** | Updating model weights on new training data |
| **Hallucination** | Confident, plausible-sounding output that is factually wrong |
| **LLM-as-judge** | Using a model to evaluate another model's output |
| **MCP** | Standard for connecting AI models to tools |
| **Model drift** | Gradual change in a model's output distribution over time due to provider-side updates, distinct from data drift |
| **Multimodal** | AI that processes or generates multiple content types (text, image, audio, video) |
| **Prompt caching** | Storing repeated context to reduce cost |
| **Prompt injection** | User input that attempts to override system instructions |
| **Quality flywheel** | The self-reinforcing cycle where better evals → better prompts → better outputs → more user acceptance → more labeled data → better evals |
| **RAG** | Finding relevant docs before the model call |
| **Red-teaming** | Deliberately trying to make the AI produce unsafe or policy-violating outputs before launch |
| **Regeneration rate** | % of outputs where the user asks the model to try again |
| **Shadow mode** | Running an AI system in parallel with an existing system to compare outputs without exposing users to the AI output |
| **Skill** | The atomic unit of an AI product: a system prompt + context injection + tool definitions + guardrails + output specification, composable into larger workflows |
| **Spike** | 1–3 day engineering investigation to validate if AI can do a task before committing to a timeline |
| **Streaming** | Returning tokens as they generate, not waiting for completion |
| **Temperature** | Controls output randomness (0=deterministic, 1=varied) |
| **Tiered resolution** | A cost-optimisation pattern where cheap/fast paths (cache, retrieval) handle the majority of queries; expensive generation paths handle only what lower tiers can't |
| **Token** | ~4 characters; the unit of cost and context |
| **Tool use** | Model requesting an action from an external system |
| **Vector database** | Database optimized for similarity search |
| **Vertical AI** | AI product purpose-built for a specific industry or workflow |
| **AI-augmented** | Existing software product with AI capabilities added |
| **AI-native** | Product built AI-first from the ground up, no pre-AI baseline |
