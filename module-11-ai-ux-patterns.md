# Module 11: AI UX Patterns

## Why AI UX is different

Traditional software UX: deterministic, consistent, predictable. The button does the same thing every time. 

AI UX: probabilistic, variable, occasionally wrong. The model produces different output each time. It can be confidently wrong. It can fail in ways you didn't anticipate. Users need to calibrate how much to trust it.

Most AI products fail at UX, not at model quality. The model is good enough — the product doesn't set users up to use it well.

---

## Pattern 1: Setting the right expectations

The biggest UX mistake is making AI feel more reliable than it is. Users over-trust, get burned by a confident hallucination, and abandon the feature.

**Calibrate confidence in the UI:**
- Don't present AI output with the same visual weight as verified data
- Use language that signals provisionality: "Here's a draft...", "Based on what I found...", "You may want to verify..."
- Show sources when you can: "Based on your Q3 report (uploaded 2 hours ago)"

**Be specific about what the AI can and can't do:**
```
Good: "Ask me anything about your account history or current plan."
Bad: "Ask me anything."
```

The more specifically you bound the feature, the easier it is for users to calibrate when to trust it.

---

## Pattern 2: Progressive disclosure

Don't dump a 500-word AI response on the user. Lead with the most important information, then allow expansion.

**Tiered response pattern:**
```
[Immediate answer] → [See more] → [Full explanation]
```

**Example:**
```
SUMMARY: Your subscription renews on June 15 for $49.

▼ Details
Your current plan is Pro Monthly. It includes: [...]
You were last charged on May 15. Here's your billing history: [...]
```

Users who got what they needed don't have to scroll. Users who need more can get it.

---

## Pattern 3: Streaming and "thinking" states

Never show users a blank loading state for AI. Use one of these:

**Streaming:** Return text as it generates. Best for conversational or generative features.

**Progress indicators with stages:**
```
⟳ Reading your documents...
⟳ Finding relevant sections...
✓ Analyzing pricing terms...
⟳ Drafting summary...
```

This does two things: reduces perceived wait time, and teaches users what the AI is doing (which builds understanding and trust).

**Thinking animations:** A pulsing ellipsis or skeleton loader is better than nothing, but stage labels are better.

---

## Pattern 4: Inline editing and regeneration

AI output should always be editable. Users may want to:
- Correct a minor error without regenerating everything
- Adjust tone or length
- Accept most of it but change one part

**Essential controls:**
- Edit the AI output directly (like a draft)
- "Regenerate" with optional instructions ("Make it shorter", "More formal")
- "Undo" back to the AI's original suggestion

If your AI feature produces content users will use (emails, documents, summaries), and they can't edit it, you've made them dependent on perfection from a probabilistic system.

---

## Pattern 5: Showing sources and citations

For any AI feature making factual claims (Q&A, research, analysis), show where the information came from.

```
Your enterprise plan includes:
• Unlimited users ¹
• SSO and SAML support ¹
• Dedicated customer success manager ²
• 99.9% SLA ³

Sources:
¹ Enterprise Plan Features — help.acme.com/enterprise
² Enterprise Onboarding Guide — last updated May 1
³ Service Level Agreement — legal.acme.com/sla
```

Benefits:
- Users can verify claims that matter to them
- Reduces liability when the model is wrong
- Helps users understand what data the system is using
- Builds trust through transparency

**When to require citations:** Any feature in a professional or high-stakes context (finance, legal, healthcare, customer contracts).

---

## Pattern 6: Graceful degradation

Design explicitly for when the AI fails, doesn't know, or can't help. Three tiers:

**Tier 1 — Partial answer with acknowledged gaps:**
```
"I found information about your basic plan pricing, but I don't have details 
on the enterprise tier you asked about. For enterprise pricing, contact our 
sales team."
```

**Tier 2 — Honest "I don't know":**
```
"I don't have enough information to answer that question accurately. 
Try rephrasing, or search our help center."
```

**Tier 3 — Human escalation:**
```
"This seems like something that needs a human to review. Want me to 
connect you with a support agent? [Yes, connect me] [No, I'll try again]"
```

Never let the AI confidently make something up instead of these tiers. You must prompt it to do this (Module 10), and you must design the UI to present it gracefully.

---

## Pattern 7: Feedback mechanisms

Capture quality signals from users to improve the feature and detect failures.

**Minimal (always):** Thumbs up / thumbs down. Log the full context when thumbs down.

**Enhanced:** Star rating (1–5) with optional comment.

**Contextual:** Inline corrections. If the user edits the AI output, log what changed.

**Review queue:** Surface the worst-rated outputs to a human reviewer weekly. This is your qualitative eval signal.

**Critical:** If someone clicks thumbs down on the customer support AI, the problem didn't go away. Build a path to resolution — don't just log it.

---

## Pattern 8: Scope and off-topic handling

Design and communicate the AI's scope explicitly. Build UX for out-of-scope requests.

**In-scope boundary in UI copy:**
```
"Ask about your account, billing, and plans"
```
(Not: "Ask me anything")

**Out-of-scope response:**
```
User: "Can you write me a poem?"
AI: "I'm here to help with your Acme account questions. 
Is there something about your plan or billing I can help with?"
```

The key: don't just refuse. Redirect to the scope you do support. And make it easy to escalate if the user's real need is something the AI can't handle.

---

## Anti-patterns to avoid

| Anti-pattern | Problem | Fix |
|-------------|---------|-----|
| **Presenting AI output as fact** | Over-trust → embarrassing failures | Visual treatment that signals "draft" or "AI-generated" |
| **No edit/regenerate** | Users stuck with imperfect output | Always allow editing |
| **Generic loading spinner** | Users feel the AI is a black box | Stage labels during processing |
| **Silent failure** | AI returns empty/error, user sees nothing | Explicit "I couldn't complete this" state |
| **Infinite chat history** | Long conversations degrade quality (context issues) | Summarize or restart context periodically |
| **No scope boundary** | Users try things AI can't do, get bad results | Define and communicate scope in UI |
| **Showing raw JSON/errors to users** | Technical output leaked to UI | Handle all output types gracefully |
| **No feedback mechanism** | No signal to improve or detect failures | At minimum: thumbs up/down |

---

## When not to use AI

Not every feature needs AI. Adding AI to the wrong place creates complexity without value.

**Don't use AI when:**
- A deterministic system (database query, formula, rule engine) does the job reliably
- Speed is critical and AI latency is unacceptable
- The output must be 100% consistent (AI will vary)
- You can't accept occasional wrong answers (safety-critical systems)
- The task is simple enough that a dropdown or button handles it better
- The user doesn't need language — they need data

**The right question:** "Does this feature benefit from language understanding or generation?" If yes, AI probably helps. If the feature is fundamentally about displaying or processing structured data, AI is likely adding cost and complexity without benefit.

---

## Progressive autonomy model

How much agency you give the AI is a product decision with real consequences. Too little autonomy and the feature adds friction without saving work. Too much and errors compound before anyone catches them. The right level depends on trust earned and stakes involved.

There are five levels of AI autonomy:

**Level 1 — Suggest**
The AI shows options and the human picks from them. The user takes every action.

- When appropriate: New features, new users, high-stakes domains, any situation where the AI's reliability is unproven.
- Trust/stakes threshold: No established trust; any stakes level.
- Example: An AI that generates three possible email subject lines and the user clicks one to use it.

**Level 2 — Recommend**
The AI shows ranked options with reasoning. The user still takes the action, but the AI signals which option it thinks is best and why.

- When appropriate: The AI has demonstrated reasonable accuracy on the task, and users benefit from the reasoning but want final say.
- Trust/stakes threshold: Some established reliability; medium or lower stakes.
- Example: An AI that ranks three pricing tiers for a new customer with a brief rationale for each, and the sales rep selects one to present.

**Level 3 — Act with confirmation**
The AI proposes a specific action and the user approves before it executes. The AI does the work of deciding; the human does the work of reviewing.

- When appropriate: The AI is reliable enough to propose the right action most of the time, but the action has consequences that make a brief human checkpoint worthwhile.
- Trust/stakes threshold: Established reliability; medium stakes with some reversibility.
- Example: An AI that drafts and shows a support reply and waits for the agent to click "Send."

**Level 4 — Act with notification**
The AI acts and then tells the user what it did. The user is informed rather than asked.

- When appropriate: The AI is highly reliable on a specific, well-bounded task; actions are low-stakes or easily reversible; requiring confirmation would create more friction than value.
- Trust/stakes threshold: High reliability demonstrated over time; low stakes or high reversibility.
- Example: An AI that automatically files a support ticket into the correct category and notifies the agent: "I've tagged this as a billing issue and routed it to the billing queue."

**Level 5 — Fully autonomous**
The AI acts silently within defined boundaries. The user audits periodically rather than reviewing each action.

- When appropriate: Routine, repetitive tasks where the AI's accuracy is very high, the actions are low-risk, and constant notification would be noise.
- Trust/stakes threshold: Very high proven reliability; low stakes; user has explicitly opted in.
- Example: An AI that runs nightly data quality checks, flags anomalies for human review, and takes no action unless a threshold is crossed.

**The governing principle:** Start at Level 1 for every new feature and every new user. Advance to higher levels only when reliability at the current level is proven, users have opted in, and guardrails are strong enough to catch errors at the higher level. Trust is earned incrementally, not granted upfront.

---

## Failure severity hierarchy

Not all AI failures are equal. Knowing the severity of a potential failure helps you decide how much to invest in preventing it, how quickly to respond when it happens, and what recovery action is required.

| Severity | Description | Example | User impact | Required recovery action |
|---|---|---|---|---|
| **Cosmetic** | Wrong tone, minor formatting error, slightly awkward phrasing | Response is technically correct but overly formal for the context | Mild friction; user still gets the right answer | Improve prompt; no urgency |
| **Functional** | Wrong answer, missing key information, incorrect data | AI states the wrong renewal date for a user's subscription | User makes a decision based on incorrect information | Fix the root cause (prompt or context gap); add to eval dataset |
| **Trust-eroding** | Confidently wrong, contradicts itself, or contradicts what the AI said earlier in the same session | AI states a product has a feature it does not have, with high confidence | User loses faith in the feature; may stop using it | High-priority fix; review all similar outputs; consider temporary scope reduction |
| **Harmful** | Offensive content, privacy violation, dangerous advice | AI reveals another user's account details; provides advice that could cause physical harm | Direct harm to user; legal and reputational exposure | Immediate incident response; pull feature if necessary; legal review |
| **Catastrophic** | Irreversible real-world action taken based on AI error | AI-triggered automation deletes production data; AI sends an external communication with false claims | Irreversible damage; cannot be undone by fixing the prompt | Immediate feature rollback; incident post-mortem; architectural review of action permissions |

**PM decision:** For Cosmetic and Functional failures, standard engineering processes apply. For Trust-eroding failures, treat as a P1 — user trust is a product asset that takes time to rebuild. For Harmful and Catastrophic failures, the response is an incident, not a bug, and requires executive involvement, user communication, and architectural remediation.

---

## PM Decision Checklist — Module 11

- [ ] Does the UI communicate the AI's scope explicitly — what it can and can't do?
- [ ] Is AI output visually distinguished from verified data?
- [ ] Can users edit or regenerate AI output?
- [ ] Is there streaming or a staged progress indicator (not a blank spinner)?
- [ ] Are sources/citations shown for factual features?
- [ ] Have I specced all three tiers of graceful degradation?
- [ ] Is there a feedback mechanism (at minimum thumbs up/down)?
- [ ] Have I reviewed this feature against the anti-patterns list?
- [ ] Does this feature genuinely need AI — or would a deterministic approach work better?
- [ ] What autonomy level is appropriate for launch, and what conditions would justify advancing to the next level?
- [ ] Have I mapped the failure severity levels for this feature and defined recovery actions for each?
