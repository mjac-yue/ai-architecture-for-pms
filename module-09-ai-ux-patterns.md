# Module 9: AI UX Patterns

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

Never let the AI confidently make something up instead of these tiers. You must prompt it to do this (Module 8), and you must design the UI to present it gracefully.

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

## PM Decision Checklist — Module 9

- [ ] Does the UI communicate the AI's scope explicitly — what it can and can't do?
- [ ] Is AI output visually distinguished from verified data?
- [ ] Can users edit or regenerate AI output?
- [ ] Is there streaming or a staged progress indicator (not a blank spinner)?
- [ ] Are sources/citations shown for factual features?
- [ ] Have I specced all three tiers of graceful degradation?
- [ ] Is there a feedback mechanism (at minimum thumbs up/down)?
- [ ] Have I reviewed this feature against the anti-patterns list?
- [ ] Does this feature genuinely need AI — or would a deterministic approach work better?
