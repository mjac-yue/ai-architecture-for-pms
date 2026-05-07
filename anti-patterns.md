# AI Product Anti-Patterns

A catalogue of common mistakes PMs make when shipping AI features, organised by lifecycle phase. Each entry: the pattern, why it fails, and what to do instead.

---

## Discovery anti-patterns

### "AI for AI's sake"
Adding AI to a feature because the company wants to ship "AI features," not because users have a real problem AI solves.

**Why it fails:** The feature ships, gets a press mention, and quietly degrades because no one cares enough to maintain it. Users don't use it. Engineers resent the work. Trust in AI features overall declines.

**Fix:** Run the AI rightness test (Module 1). If the feature doesn't pass, push back even when leadership wants AI on the slide.

---

### Skipping the spike
Committing to a launch date for an AI feature before proving the model can do the task on real data.

**Why it fails:** Mid-build, the team discovers the quality is unacceptable. Now you're choosing between missing the date, shipping a bad feature, or rebuilding from scratch.

**Fix:** Always run a 1–3 day spike before estimation. No spike = no estimate = no roadmap commitment.

---

### Solving the wrong job
Building an AI feature that technically works but addresses a job no user has.

**Why it fails:** Looks good in demos. Adoption is flat. Users tried it, didn't see value, didn't come back.

**Fix:** Articulate the AI problem statement (Module 5) before development. If you can't fill in "so that [measurable user outcome]" with something users actually care about, stop.

---

### Ignoring the deterministic alternative
Building an AI feature for a task a database query, formula, or rule engine handles better.

**Why it fails:** AI introduces cost, complexity, and hallucination risk. The deterministic version would have been faster, cheaper, and more reliable.

**Fix:** Before any AI feature, ask: "Could a deterministic system do this acceptably?" If yes, build that.

---

### Deterministic Expectations
Treating AI output like a database query: expecting the same input to always produce the same output, and designing systems or stakeholder demos around that assumption.

**Why it fails:** Stakeholder demos show different output than last time and confidence collapses. Integrations that depend on precise output format break when the model phrases things differently. Users become confused when identical questions get slightly different answers. The team spends time chasing variance that is a feature of the system, not a bug.

**Fix:** Set stakeholder expectations during framing that AI is probabilistic by design. Design for variance — use semantic evaluation instead of exact-match testing, parse structured outputs (JSON) rather than free text, and build downstream logic that handles output variation gracefully. Don't fight the probabilistic nature; design around it.

---

## Spec anti-patterns

### Underspecified output format
"The AI should produce a summary." OK — bullet points or paragraphs? How long? What sections?

**Why it fails:** Inconsistent outputs in production. Parsers break when format drifts. Every prompt iteration takes longer because you're re-discovering the format you wanted.

**Fix:** Specify output format with an example in the system prompt. For programmatic consumption, define exact JSON schema.

---

### Missing "I don't know" instruction
Not telling the model what to do when it lacks information.

**Why it fails:** The model fills in gaps with plausible-sounding hallucinations. Users trust them. Bad outputs reach customers.

**Fix:** Add an explicit instruction: "If you don't have enough information, say so and don't guess." Test this with eval cases that lack the needed info.

---

### No edge case spec
Specs that only describe the happy path.

**Why it fails:** Engineers ship the happy path. Edge cases break in production — empty input, wrong language, unexpected length, off-scope requests.

**Fix:** Every AI spec must include a section on edge cases and what should happen for each.

---

### Subjective quality bar
"It should work well" or "Output should be high quality."

**Why it fails:** Iteration never converges because there's no agreement on done. Different reviewers have different standards. The feature ships when someone's tired.

**Fix:** Define quality as a measurable metric on a defined dataset. "≥75% acceptance on the 50-case eval set" beats "high quality."

---

### The Disclaimer Sandwich
Adding so many caveats, hedges, and "please verify this" instructions to AI output that the output loses practical utility. Every response starts with "As an AI..." or ends with three paragraphs of disclaimers regardless of confidence or risk level.

**Why it fails:** Users stop reading disclaimers immediately — the same banner blindness that kills cookie consent notices. Uniform disclaimers provide no signal: if everything is uncertain, nothing is. The AI appears less useful than it actually is, and adoption suffers. Blanket disclaimers are a substitute for actual confidence calibration, not a replacement for it.

**Fix:** Design specific, scoped disclaimers for specific risks rather than blanket hedging. Show high confidence by default through source attribution and clear assertions. Flag uncertainty only when it is meaningful — a specific claim that may be wrong, a source that may be outdated. Trust the user to understand AI limitations if the UI communicates scope correctly.

---

## Build anti-patterns

### Prompt sprawl
The system prompt grows over weeks of iteration to thousands of tokens of accumulated rules and edge case handling.

**Why it fails:** Long prompts hide earlier instructions. Quality plateaus. Cost rises. New rules contradict old ones. Nobody understands what each section does.

**Fix:** Periodically prune. Test removing sections to see if quality holds. Keep prompts as short as quality allows.

---

### Prompt outside version control
The system prompt lives in a database row, a CMS field, or a config UI anyone can edit without review.

**Why it fails:** Someone makes a "small improvement" that silently breaks production. There's no diff, no review, no rollback path.

**Fix:** System prompts are code. Version control them. Review changes. Tag deployable versions. Never let prompts be edited in a system without git history.

---

### No eval dataset
Iterating on prompts based on "looking at outputs" without a structured eval.

**Why it fails:** You can't tell if a change helped or hurt. Regressions go undetected. Each prompt change is a guess.

**Fix:** Build the eval dataset before iterating on the prompt (Module 17). 30+ representative cases is enough to start. Run it before every prompt change.

---

### Premature multi-agent
Splitting a feature into multiple agents before validating that single-agent doesn't work.

**Why it fails:** Multiplies cost and latency. Adds debugging complexity. Often the single-agent version would have been fine with a better prompt.

**Fix:** Build the simplest version first. Move to multi-agent only when you can articulate which signal (Module 9) drove the decision.

---

### Trusting the model with sensitive operations
Letting the model directly send emails, delete data, or make payments without human review or strong guardrails.

**Why it fails:** Prompt injection or model errors trigger irreversible actions. Lawsuits, lost data, lost trust.

**Fix:** Anything irreversible requires human-in-the-loop. Tools that take destructive actions need both prompt-level guardrails and code-level confirmation steps.

---

### Prompt Engineering Theater
Iterating on prompts based on vibes and spot-checks rather than eval measurements. The prompt "seems better" after tweaks, but there is no before/after eval score comparison and the same failure modes keep reappearing after each fix.

**Why it fails:** Without measurement, you have no way to confirm improvement. You may be fixing one visible failure while reintroducing an earlier one. Prompt sensitivity varies by model — what appears to work today may silently break after a provider update. Weeks of prompt iteration can produce no net progress while creating the appearance of diligence.

**Fix:** Every prompt change must be accompanied by an eval run. Compare scores before and after the change. If you don't have an eval dataset yet, write one before changing the prompt — not after. Treat prompt changes with the same rigor as code changes: one change at a time, measured result, no ship without a passing score.

---

## Launch anti-patterns

### No staged rollout
Shipping AI features to 100% of users on day one.

**Why it fails:** Failure modes that didn't appear in eval show up at scale. Bad outputs reach high-volume users immediately. Recovery is harder.

**Fix:** Internal → beta → gradual percentage → full. Every stage has a quality gate.

---

### Missing kill switch
Going to production without a fast way to disable the AI feature.

**Why it fails:** When something goes wrong (jailbreak, model regression, cost runaway), the only options are slow code deploys while the problem continues.

**Fix:** Feature flag controlling AI vs. fallback path. Tested before launch. Flippable by ops, not just engineering.

---

### "Powered by AI" instead of specific scope
Marketing the feature with vague AI claims rather than specific capabilities.

**Why it fails:** Users don't know what to ask. Adoption is low because expectations are vague. Some users try things outside scope and get bad outputs.

**Fix:** Communicate scope specifically. "Ask about your account history" beats "Powered by AI."

---

### Confident UI for uncertain output
Showing AI output with the same visual treatment as verified data.

**Why it fails:** Users over-trust. They act on hallucinations because the UI didn't signal uncertainty.

**Fix:** Visual treatment that signals "draft" or "AI-generated." Show sources. Use language like "Based on what I found..."

---

### No feedback mechanism
Launching without thumbs up/down or any user feedback signal.

**Why it fails:** You can't detect quality problems early. You can't build a queue of cases to review. You can't improve.

**Fix:** Thumbs up/down at minimum, with logging of full context on negative feedback. Build before launch, not after.

---

### Testing in Production Only
No eval dataset exists before launch; quality is only visible when real users complain. The team believes demo quality is a sufficient signal to ship.

**Why it fails:** By the time you know something is wrong, many users have already seen bad output and formed a negative impression. User complaints are a lagging, incomplete signal — they capture only the failures users noticed and bothered to report. You have no baseline to compare against, no way to confirm whether a fix actually worked, and no protection against regressions.

**Fix:** An eval dataset is a launch gate. If it does not exist, the feature is not ready to ship regardless of demo quality. Start with 30 representative inputs and expected outputs. Production signals supplement the eval suite — they do not replace it.

---

## Operations anti-patterns

### Ship and forget
Treating AI features like traditional software — ship it, fix occasional bugs, move on.

**Why it fails:** AI features degrade over time without active maintenance (model drift, data drift, prompt rot). The feature that worked at launch silently gets worse.

**Fix:** Schedule weekly/monthly/quarterly reviews (Module 20). Assign a named owner. Treat ongoing maintenance as the cost of having the feature.

---

### Conflating uptime with quality
Monitoring API availability and latency but not output quality.

**Why it fails:** Everything looks "green" while users are getting hallucinated answers. Quality drift is invisible.

**Fix:** Monitor product quality metrics (Module 17) alongside operational metrics. Acceptance rate trending down is an alert-worthy signal even when API uptime is 100%.

---

### No re-eval on model changes
Switching to a new model version without re-running the eval dataset first.

**Why it fails:** The new model may handle your specific use case worse, even if it's "better" on benchmarks. Quality regresses silently.

**Fix:** Every model change runs against the eval dataset first. Compare scores. Stage rollout. Keep the previous version on standby.

---

### Cost surprise
Discovering AI costs are 5× expected at the end of the month.

**Why it fails:** Budget overruns. Hard conversations with finance. Sometimes the feature has to be killed to control cost.

**Fix:** Cost monitoring alerts (Module 18). Cost dashboards reviewed weekly. Output length constraints in prompts. Prompt caching where applicable.

---

### Letting technical debt accumulate in prompts
Working around model issues with increasingly complex prompt instructions instead of addressing root causes.

**Why it fails:** Prompts become brittle, expensive, and hard to maintain. New issues get layered on top of old workarounds.

**Fix:** When the prompt grows complex, reset and ask: "What's the simplest prompt that could meet the quality bar with current models?" Periodically refactor.

---

### The AI Expert Silo
Only one person on the team understands the AI feature — the prompts, the eval logic, the failure modes, the architecture. Everyone else treats it as a black box. When that person is unavailable, the feature cannot be maintained, incidents cannot be debugged, and improvements stall.

**Why it fails:** Single point of failure means any incident during that person's absence causes extended downtime or unresolved quality degradation. Without peer review, prompt changes and eval design go unchecked. Knowledge does not transfer — the team cannot maintain the system long-term. The PM cannot effectively manage or advocate for a system they do not understand.

**Fix:** AI features need the same knowledge-sharing standards as any other production system. System prompts in version control with inline comments explaining the reasoning behind key instructions. Eval datasets documented with rationale for each case. Runbooks for common failure modes. At minimum two people who can diagnose a prompt regression, tune a guardrail, and run the eval suite independently.

---

## Cross-cutting anti-patterns

### Letting eng make product decisions by default
The PM doesn't understand the AI architecture, so engineering makes choices about model tier, prompt structure, retrieval approach — choices that shape product behaviour.

**Why it fails:** The product reflects engineering convenience rather than user needs. The PM can't review prompts critically. Decisions get made without product input.

**Fix:** Learn enough to participate in the decisions (this whole course). Write AI behaviour specs. Review prompts in PRs.

---

### Overpromising to leadership
"The AI will handle X" or "Quality will be 95%+" before validating.

**Why it fails:** Reality doesn't match the promise. Trust in the team — and in AI features generally — declines. Future AI proposals get harder approval.

**Fix:** Communicate uncertainty in terms of measurable bars and ranges. "We expect 70–85% acceptance based on current eval results, with planned improvements in the following sprint."

---

### Building for the demo, not the use case
Optimising the feature for a 90-second demo to leadership rather than the messy real-world context users will use it in.

**Why it fails:** Demo looks great. Real usage shows the cracks. The team is now defending a feature that doesn't work in practice.

**Fix:** Spike on real user data, not curated demo data. Show leadership the failure cases too, not just the success cases.

---

### Treating AI as a feature, not a system
Speccing the AI part and assuming the surrounding system (data freshness, observability, fallbacks, monitoring) will follow.

**Why it fails:** The AI ships without the system around it. When things go wrong (and they will), there's nothing in place to detect, diagnose, or recover.

**Fix:** Spec the system, not just the feature. Data, prompt, model, fallback, monitoring, kill switch — all of it is the product.
