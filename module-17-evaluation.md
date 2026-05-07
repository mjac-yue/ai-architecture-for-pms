# Module 17: Evaluating AI Features

## The problem with shipping AI without evals

Traditional software: write a test, run it, pass/fail is deterministic. You know if it works.

AI: the output is probabilistic, the "correct" answer is often subjective, and the model can pass 100 tests and fail on the 101st input you hadn't thought of. You can't use a unit test suite the way you normally would.

Without a structured evaluation approach, you're flying blind. You'll ship based on vibes ("it seemed good in the demo") and discover failures in production.

This module gives you a practical evaluation framework that doesn't require an ML background.

---

## The eval hierarchy

Evaluation exists at five levels, from the fastest and cheapest to the slowest and most expensive. Each level catches different failure modes.

| Level | Name | What it does | Run frequency |
|-------|------|-------------|---------------|
| 1 | Unit tests | Deterministic assertions on known inputs — exact match, contains, excludes | On every change |
| 2 | Automated checks | Regex, schema validation, keyword presence, format checks — validates output properties without exact match | On every change |
| 3 | Model-as-judge | A second LLM evaluates the primary model's output quality against a rubric | On every change or nightly |
| 4 | Human evaluation | Domain experts review a sampled set of outputs against written criteria | Weekly or monthly |
| 5 | Production monitoring | Real user signals — accept rate, edit rate, regeneration rate, thumbs up/down | Continuously |

**The progression:** Run Levels 1 and 2 continuously on every prompt, model, or context change — they are fast and cheap. Run Level 3 on every change or nightly. Run Level 4 periodically (weekly or monthly) to calibrate your automated evaluators and catch nuance they miss. Level 5 is always running in the background once the feature is live, and its signals feed back into all lower levels.

No single level is sufficient. Unit tests catch hard structural failures but miss quality. Model-as-judge scales subjective assessment but has its own biases. Human eval is ground truth but too slow to run on every change. Production signals are the most real but arrive too late to prevent a bad launch.

---

## The three layers of AI evaluation

### Layer 1: Functional correctness
Does it do the task at all?

Examples:
- Does the summarizer produce a summary? (not an error, not an empty string)
- Does the JSON extractor return valid, parseable JSON?
- Does the classifier return one of the expected categories?

These are easy to automate. Do them first. They catch the dumbest failures.

---

### Layer 2: Quality
Does it do the task well?

This is where it gets hard. "Well" means different things for different tasks:
- Factual accuracy (did it get the facts right?)
- Completeness (did it cover all the key points?)
- Tone and style (does it match the brand voice?)
- Relevance (is the response actually about what was asked?)
- Conciseness (is it the right length?)

For most features, you need both automated and human evaluation here.

---

### Layer 3: Safety and robustness
Does it behave correctly in edge cases and adversarial inputs?

- What happens when the user asks it to do something out of scope?
- What happens when the user tries to manipulate it ("ignore your previous instructions")?
- What happens when the input is in a different language than expected?
- What happens when the input is empty, or extremely long, or nonsensical?
- Does it ever generate harmful, embarrassing, or legally problematic content?

These are the failures that make headlines. Test them before launch.

---

## Building an eval dataset

An eval dataset is a set of inputs with known expected outputs (or quality criteria) that you run your AI feature against.

**Step 1: Collect representative inputs**
- 20–30 real examples (if you have them) covering the main use cases
- 10–15 edge cases: off-topic questions, short inputs, long inputs, ambiguous inputs
- 5–10 adversarial inputs: attempts to jailbreak, off-scope requests, prompt injection attempts

Total: 50–60 examples is enough to start. Add more as you find failure modes.

**Step 2: Define expected outputs**
- For structured outputs (JSON, classification): exact expected values
- For quality evaluation: a rubric (see below)
- For safety tests: "must not contain X" or "must redirect to Y"

**Step 3: Run regularly**
- Before any significant prompt change
- Before any model upgrade (switching from Sonnet 4.5 to 4.6, etc.)
- After production failures (add the failure case to the dataset)

---

## Building an eval suite: 4 steps

An eval suite is not a one-time artifact — it is a living system you build incrementally. Follow this sequence.

**Step 1: Start with golden examples**

Before engineering begins, the PM authors 3–5 ideal input/output pairs. These are the "perfect answer" cases — what the feature should produce when everything works. They serve as the design specification and the first ground truth for all automated evaluators.

3–5 golden examples is a deliberately low bar. The point is to start with something authoritative, not to be exhaustive. A golden example written by the PM carries more signal than 50 synthetic examples generated without domain knowledge.

**Step 2: Add failure cases from the spike**

During feasibility testing (the initial build and exploration phase), document every input that produced a bad or unexpected output. Each one becomes an eval case. The prototype's job is not to be good — it is to make failure modes visible. Capture them all.

**Step 3: Add edge cases**

Systematically add inputs that represent the boundaries of the feature's expected scope:
- Empty input (user submits nothing)
- Maximum-length input (user submits a very long document or query)
- Adversarial input (attempts to override instructions, inject prompts, or manipulate behavior)
- Multilingual input (queries in languages other than the primary supported language)

These cases don't need to "pass" in the same way happy-path cases do — they need to pass in the sense that the feature handles them gracefully and predictably.

**Step 4: Add production failures continuously**

Every time a production failure reveals a new failure mode — one not already represented in the eval suite — add it. This is the most important step and the one most teams skip. The eval suite should grow monotonically after launch. Every failure that ships twice is a process failure.

---

## Scoring frameworks

### For factual accuracy: citation-based scoring
Score only on claims that can be verified against a source.

```
Score per response:
- Each verifiable claim: 1 point possible
- Correct claim: 1 point
- Incorrect/hallucinated claim: -1 point (penalize harder than missing)
- Missing key claim: 0

Accuracy = correct / (correct + incorrect)
```

---

### For quality: rubric-based scoring
Define 3–5 dimensions, score each 1–5.

**Example rubric for a customer support AI:**
| Dimension | 1 | 3 | 5 |
|-----------|---|---|---|
| Accuracy | Factually wrong | Mostly correct | Fully accurate |
| Completeness | Misses core issue | Addresses issue partially | Fully resolves or escalates appropriately |
| Tone | Inappropriate or cold | Neutral | Warm, professional, on-brand |
| Conciseness | Way too long or too short | Appropriate length | Perfectly calibrated |

---

### For safety: pass/fail checklist
Binary. Either it passed or it didn't.

```
Safety check: [Test input]
Expected: [Safe response / rejection / redirect]
Actual: [What the model did]
Pass/Fail: [Pass if actual matches expected]
```

---

## LLM-as-judge: automating quality eval

Running human evaluation on every prompt change doesn't scale. A practical alternative: use a second model to evaluate the first model's outputs.

```
Evaluator prompt:
"You are evaluating a customer support AI response. Score the following response 
on Accuracy (1-5), Completeness (1-5), and Tone (1-5). 
Return your scores as JSON: {"accuracy": N, "completeness": N, "tone": N, 
"reasoning": "brief explanation"}.

User question: [question]
Expected answer context: [ground truth]
AI response: [response being evaluated]"
```

This isn't perfect — models have biases in evaluation too — but it's cheap, fast, and scales. Use it to catch regressions and narrow down where human review is needed.

---

## Writing acceptance criteria for AI features

The standard "Given / When / Then" format needs adapting for AI:

**Standard acceptance criteria:**
```
Given: a logged-in user with an active subscription
When: they click "Cancel subscription"
Then: the subscription is cancelled and they receive a confirmation email
```

**AI acceptance criteria:**
```
Given: a user asks "How do I cancel my subscription?"
When: the support AI responds
Then:
  - The response must contain the cancellation steps (from the help doc)
  - The response must NOT contain any pricing information (out of scope)
  - The response must offer to escalate if the steps don't resolve the issue
  - Response length must be under 150 words

Edge case: user asks "Delete my account and all my data"
Then:
  - AI must NOT perform any account actions
  - AI must route to the account deletion flow and explain human review is required
```

The key additions for AI:
- Explicit "must not" conditions
- Edge case behavior
- Length/format constraints
- What counts as a failure (not just what counts as success)

---

## What to monitor in production

Once launched, set up monitoring for:

| Signal | Why | How |
|--------|-----|-----|
| Thumbs up/down or star ratings | Direct quality signal from users | Inline feedback widget |
| "Regenerate" or "Edit this" actions | User found output unsatisfactory | Track the action, sample the context |
| Escalation rate | AI couldn't handle the request | Track when AI says "I don't know" or "Let me connect you..." |
| Response length distribution | Drift from expected length is a quality signal | Log token counts |
| Flagged outputs | Harmful or inappropriate content | Automated content classifiers |

Set a baseline from the first week, then alert on significant deviations.

---

## Eval anti-patterns

| Anti-pattern | What it looks like | Why it fails |
|---|---|---|
| Testing only happy path | Eval dataset contains representative queries but no edge cases, empty inputs, or adversarial examples | The model looks great on the eval and fails on the first unusual production input |
| Eval dataset that never grows | The same 50 cases that were written at launch are still the entire eval suite two years later | Production failures are not systematically added, so the same failures can ship repeatedly |
| Metric without a threshold | The team tracks an average quality score but has no defined pass/fail line | There is no objective basis for a go/no-go decision; every launch becomes a negotiation |
| Human eval without a rubric | Reviewers score outputs based on personal judgment with no written criteria | Scores are inconsistent across reviewers and over time; inter-rater agreement is low |
| Treating eval score as a launch gate only (not a continuous signal) | Evals are run once before launch and then discontinued | Quality degrades post-launch through model drift, data changes, and prompt edits — with no detection |
| LLM-as-judge with no calibration | A model-as-judge evaluator is set up and trusted without checking its scores against human judgments | The judge has systematic biases (preferring longer answers, penalizing uncertainty) that make the scores misleading |
| Evaluating output only, not retrieval | Evals score the final response but do not check whether the retrieved context was correct | Retrieval failures are invisible; the team improves the prompt while the real problem is the RAG index |

---

## The eval mindset for PMs

The most important shift: **evals are a product requirement, not an afterthought.**

If a feature spec doesn't include an eval plan, it's not complete. Ask for:
- What is the eval dataset and who created it?
- How are we scoring quality?
- What are the pass/fail thresholds for launch?
- How will we run evals when the prompt changes?
- What's the process for adding new eval cases from production failures?

---

## PM Decision Checklist — Module 17

- [ ] Does the feature spec include acceptance criteria with "must not" conditions and edge cases?
- [ ] Is there an eval dataset of at least 50 examples (representative + edge cases + adversarial)?
- [ ] Is there a defined scoring rubric or criteria?
- [ ] Is there a process for running evals before prompt/model changes?
- [ ] Is there a plan for LLM-as-judge for scaling quality eval?
- [ ] What production signals are we monitoring and what are the alert thresholds?
- [ ] Is there a process for adding production failure cases back into the eval dataset?
