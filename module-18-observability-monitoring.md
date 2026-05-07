# Module 18: Observability & Monitoring

## Why observability is different for AI products

In traditional software, observability means knowing whether your servers are up and your APIs are responding. Errors are deterministic — when something fails, you get a stack trace and you fix it.

In AI products, observability means something more: you need to know whether the AI is *behaving correctly*, not just whether the system is up. A request can succeed (200 OK, low latency, no errors) and still produce a hallucinated answer that misleads the user. A request can fail in subtle ways the model never raises (silently returning truncated output, hitting safety filters, retrieving the wrong context). Without specific instrumentation, these failures are invisible.

This module is about the operational instrumentation that lets you know what your AI is actually doing — distinct from Module 17's product metrics, which are about user-facing quality outcomes. Both layers matter; this one is the engineering and ops layer.

---

## What to log: the six things you need

For every AI interaction, log these. If any are missing, you'll find yourself debugging blind in production.

```mermaid
flowchart LR
    U["User input"] --> L1["Log: input"]
    L1 --> R["Retrieval / RAG"]
    R --> L2["Log: retrieved context"]
    L2 --> M["Model call"]
    M --> L3["Log: model response"]
    M --> L4["Log: latency"]
    M --> L5["Log: cost / tokens"]
    L3 --> O["Output to user"]
    O --> L6["Log: user feedback"]
```

| What to log | Why it matters |
|------------|----------------|
| **Input** | The exact user request — needed to reproduce any issue |
| **Retrieved context** | What documents/data the system pulled — most RAG failures are retrieval failures, not model failures |
| **Model response** | The raw model output before any post-processing |
| **Latency** | End-to-end time and time-per-component (retrieval, model, post-processing) |
| **Cost / tokens** | Input + output tokens, model used, dollars spent |
| **User feedback** | Thumbs up/down, edits, regenerations — connects technical events to quality signals |

This isn't optional. If you can't see all six, you can't operate the product.

---

## Trace-based observability

For multi-step AI systems (RAG, agents, multi-agent), single events aren't enough. You need to trace a full request across all its components.

A trace shows the complete lifecycle of one user request:

```
[Trace ID: abc-123]
├─ User input received (0ms)
├─ Embedding query (45ms, $0.0001)
├─ Vector search (120ms)
├─ Top-5 chunks retrieved
├─ Prompt assembled (5ms, 4,200 input tokens)
├─ Model call (3,400ms, $0.013, Sonnet)
├─ Output post-processing (10ms)
└─ Response delivered to user (3,580ms total, $0.0131)
```

This is the level of detail you need to debug "why was this response slow?" or "why did this answer ignore the obvious source document?"

**Tooling options:**

| Tool | Strength |
|------|---------|
| LangSmith | Tightly integrated with LangChain; rich trace UI |
| Helicone | Drop-in proxy; works with any LLM provider |
| Arize / Phoenix | Production ML observability; strong on quality drift |
| OpenTelemetry + custom | Most flexible; integrates with existing observability stack |
| Provider built-in | Anthropic and OpenAI both offer basic tracing dashboards |

For early-stage products, the provider's built-in dashboards plus structured logs are usually enough. Add a dedicated tool when traces become hard to navigate manually.

---

## Production metrics: what to monitor and alert on

Different metrics serve different operational purposes.

### Real-time / on-call metrics
Watched continuously, alerted on thresholds.

| Metric | Healthy range | What a spike means |
|--------|--------------|-------------------|
| **API error rate** | <1% | Provider issue or auth/rate limit problem |
| **P95 latency** | Within budget | Provider degradation, retrieval slowdown, prompt size growth |
| **Safety filter trigger rate** | Stable baseline | Adversarial usage spike or new failure pattern |
| **Cost per hour** | Within forecast | Bug, abuse, or runaway agent loop |
| **Empty / null response rate** | <0.5% | Model returning nothing — usually prompt or content filter issue |

### Daily ops review
Reviewed every morning by whoever's on AI ops duty.

| Metric | What to look for |
|--------|-----------------|
| Daily request volume | Matches expected pattern? |
| Error breakdown by type | Any new error patterns? |
| Top 10 slowest requests | What made them slow — can we fix? |
| Top 10 most expensive requests | Cost outliers, especially in agentic features |
| Out-of-scope rate | Users trying things the AI can't do |

### Weekly PM review
Connects technical signals to product quality.

| Metric | Connects to |
|--------|------------|
| Acceptance rate vs. last week | Module 17 — overall quality direction |
| Regeneration rate trends | User satisfaction with first outputs |
| Retrieval hit rate | RAG quality — is the index serving the right content? |
| Cost per successful interaction | Unit economics of the feature |

### Monthly health review
Strategic view of the AI system's health.

- Cost growth vs. revenue/value generated
- Quality metrics vs. baseline at launch (regression check)
- New failure modes added to eval dataset
- Model and prompt version changes and their impact

---

## Cost monitoring and model routing optimisation

Cost is a first-class observability signal — not just a finance concern.

### What to track

- **Cost per request** (broken down by feature, model, user segment)
- **Cost per successful outcome** (successful requests only)
- **Token efficiency** (output tokens / input tokens — high ratios may signal verbose models)
- **Cache hit rate** (for prompt caching — low rates mean you're paying for repeated context)
- **Distribution of model usage** (% Haiku / % Sonnet / % Opus — drift up the tier ladder is expensive)

### Model routing as an optimisation lever

Once you have cost data, model routing — sending different requests to different models based on complexity — becomes a real optimisation. Patterns:

```mermaid
flowchart TD
    R["Incoming request"] --> C{"Classify complexity"}
    C -- "Simple" --> H["Haiku ($0.001/req)"]
    C -- "Standard" --> S["Sonnet ($0.01/req)"]
    C -- "Complex" --> O["Opus ($0.05/req)"]
```

Routing logic can be:
- Rule-based (input length, user tier, feature flag)
- Classifier-based (a fast Haiku call decides which model handles the request)
- Confidence-based (try Haiku first; if confidence is low, escalate to Sonnet)

**PM consideration:** Routing creates complexity. Don't introduce it until cost is actually a problem and the volume justifies the engineering work. A rule of thumb: routing is worth building when AI costs exceed ~$5,000/month.

---

## Debugging AI failures: from symptom to root cause

When something goes wrong in production, the diagnostic flow is different from traditional software:

```mermaid
flowchart TD
    S["User reports bad output"] --> Q1{"Reproducible?"}
    Q1 -- "No" --> P1["Check temperature settings Log full trace next time"]
    Q1 -- "Yes" --> Q2{"Where in trace did it go wrong?"}
    Q2 -- "Retrieval" --> R1["Wrong chunks retrieved → Improve indexing or query"]
    Q2 -- "Model call" --> R2["Model produced bad output → Prompt issue or model regression"]
    Q2 -- "Post-processing" --> R3["Output parsing or formatting → Code bug, easier to fix"]
    Q2 -- "Tool use" --> R4["Wrong tool called or bad params → Tool description or routing logic"]
```

The biggest mistake in AI debugging: assuming the model was wrong. Most AI failures are *retrieval* failures or *prompt* issues, not the model getting the answer wrong. Trace the full request before blaming the model.

---

## Quality drift detection

Unlike traditional software where behaviour is stable until someone changes it, AI behaviour can drift even when nothing in your system changes. Causes:

- **Model version updates** (the provider quietly improves the model — sometimes worse for your use case)
- **Data drift** (your RAG index content changes; new types of user queries emerge)
- **Prompt drift** (engineers make small "improvements" that accidentally regress quality)

**Drift detection tactics:**

1. **Run the eval dataset weekly** against the production system. Score regressions immediately.
2. **Track acceptance/regeneration trends.** Persistent decline = drift.
3. **Sample 50–100 outputs weekly** for human review. Spot-check for quality.
4. **Track output length distribution.** If average output suddenly grows or shrinks, something changed.
5. **Subscribe to model deprecation notices.** Don't be surprised by a forced model upgrade.

---

## Feedback loop types

Not all feedback signals are equal in what they tell you, how hard they are to collect, and how directly they drive quality improvement.

| Type | How collected | Example | How it feeds quality improvement |
|------|--------------|---------|----------------------------------|
| **Explicit** | User-initiated ratings, thumbs up/down, star ratings, comment fields | User clicks thumbs down on a response | Direct signal — log the interaction, sample for review, add persistent failures to eval dataset |
| **Implicit** | System-detected user behavior indicating dissatisfaction | User clicks "Regenerate", user edits the AI's output before using it | Indirect signal — aggregate to spot quality trends; high regeneration rate on a specific query type surfaces an eval gap |
| **Behavioral** | Downstream actions that indicate whether the user accomplished their goal with or without AI assistance | User completed the task in one session vs. abandoning and returning; time-to-task with AI vs. historical baseline | Measures AI effectiveness, not just AI quality — the difference between "user liked the output" and "AI actually helped" |
| **Outcome** | Did the AI-assisted action lead to the desired business result? | Customer support ticket resolved without escalation; generated SQL query returned correct data on first run | The strongest signal — connects AI output quality to actual business value |

**A note on instrumentation timing:** Behavioral and outcome signals are the most valuable signals you can collect, and the hardest to instrument after the fact. Both require you to define what "success" looks like at the task and business level — not just at the response level — and to instrument that definition into your product analytics. Spec these signals before engineering begins, not as a follow-up after launch. Teams that plan feedback loops as an afterthought end up with only explicit feedback (the weakest and most biased signal) and lose months of behavioral and outcome data that cannot be reconstructed.

---

## Root cause diagnosis framework

When a quality metric declines, the instinct is to "fix the prompt." That instinct is wrong about half the time. Use this diagnostic sequence before touching anything.

Diagnose in this order:

1. **Was there a recent change?** Check the change log for prompt edits, model version changes, context/data updates, or shifts in traffic pattern (new user segment, new use case emerging). If something changed, you have a starting point. If nothing changed, something external did.

2. **Is it in the context?** Pull sample traces from the failing period. Is retrieval returning the wrong chunks, empty results, or outdated content? Most quality regressions in RAG systems are retrieval failures, not model failures. Check the retrieval layer first.

3. **Is it in the instructions?** Review the system prompt for drift — instructions that were edited, constraints that conflict with each other, or guidance that no longer matches the current product. Prompt drift is common in systems where multiple engineers touch the prompt over time.

4. **Is it in the tools?** If the feature uses tools, check whether the tools are returning correct data, whether the model is selecting the right tool, and whether tool descriptions have drifted from actual tool behavior.

5. **Is it a capability wall?** If the context is right, the instructions are right, and the tools are right, but the outputs are still failing — the task may have become harder than the model can handle. This could mean the user base has evolved to more complex queries, or a new use case has emerged that the model is not capable of.

This framework is the PM's debugging protocol. Your job is not to fix the code — it is to correctly identify which layer is broken so the right team member investigates. "The model is hallucinating" is not a diagnosis. "Retrieval is returning empty results for queries about the new pricing tier" is a diagnosis.

---

## The instrumentation gate

Before any AI feature ships to production, verify:

- [ ] All six log fields are captured (input, retrieved context, response, latency, cost, feedback)
- [ ] Traces are stitched together by a request ID across all components
- [ ] Real-time alerts are configured for error rate, latency, cost spikes, safety filter rates
- [ ] Daily/weekly/monthly dashboards exist with the right metrics for each cadence
- [ ] At least one engineer knows how to look up "what happened on this specific request" within 2 minutes
- [ ] Eval dataset can be re-run against production prompts on demand

Missing any of these = not ready for production.

---

## PM Decision Checklist — Module 18

- [ ] Are all six log fields captured for every AI interaction?
- [ ] Can we trace any individual request across retrieval, model, and post-processing?
- [ ] Are real-time alerts configured for error rate, latency, cost, and safety filter spikes?
- [ ] Is there a daily ops review with named owner and clear runbook?
- [ ] Is there a weekly review connecting technical signals to product quality (Module 17)?
- [ ] Is cost broken down by feature and model tier?
- [ ] Is there a process for detecting quality drift (eval re-runs, sampling, distribution monitoring)?
- [ ] When a failure is reported, can we get from symptom to root cause via the trace, not by guessing?
- [ ] Are behavioral and outcome feedback signals specced and instrumented — not left as a post-launch afterthought?
