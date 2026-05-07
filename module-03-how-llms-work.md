# Module 3: How LLMs Actually Work — The PM Mental Model

## What you need to know (and what you don't)

You don't need to understand matrix multiplication or neural network math. You need a mental model accurate enough to predict what will and won't work — and to catch when an engineer is oversimplifying something that will bite you later.

---

## 1. Tokens: the unit of everything

LLMs don't process words or characters — they process **tokens**. A token is roughly 3–4 characters or 0.75 words in English.

Why this matters for you:
- **Cost** is priced per token (input + output separately)
- **Context limits** are measured in tokens
- Non-English text is less token-efficient (Japanese, Arabic, code can use 2–3× more tokens per word)
- A 10-page PDF is roughly 5,000–8,000 tokens

**Quick estimates:**
| Content | Approximate tokens |
|---------|-------------------|
| 1 page of text | ~500 |
| 10-page report | ~6,000 |
| Average code file | ~1,000–3,000 |
| This module | ~2,500 |

---

## 2. The context window: the model's working memory

The context window is the total amount of text a model can "see" at once — your system prompt, the conversation history, any documents you've fed it, and the output it's generating.

**Common context window sizes (2025):**
| Model tier | Context window |
|-----------|---------------|
| Small (Haiku) | 200K tokens |
| Mid (Sonnet) | 200K tokens |
| Large (Opus) | 200K tokens |

200K tokens sounds like a lot (~500 pages of text). But context has costs beyond just fitting:
- **Latency increases** as context grows
- **Cost increases** — you pay for every token in context on every request
- **Attention degrades** — models perform worse on content buried in the middle of very long contexts ("lost in the middle" problem)
- **Relevance matters more than volume** — a focused 2,000-token context often outperforms 50,000 tokens of loosely relevant material

**Product implication:** Don't design features that naively dump everything into context. Be selective. This is why RAG (Module 7) exists.

---

## 3. How models generate text

Models predict the next most likely token, one at a time, based on everything in the context so far. That's it. The magic is that doing this at scale on enough training data produces something that can reason, write, and code.

Key implications:
- **There's no lookup table.** The model isn't retrieving stored facts — it's reconstructing knowledge from learned patterns. This is why it can hallucinate confidently.
- **It can't look things up unless you give it tools.** By default, a model has no access to the internet, your database, or today's date.
- **The same prompt can produce different outputs.** This is controlled by temperature (a setting from 0–1). Temperature 0 = more deterministic. Temperature 1 = more creative/variable.

---

## 4. What models are actually good and bad at

**Strong:**
- Summarizing, synthesizing, and rewriting text
- Following complex instructions when stated clearly
- Generating structured output (JSON, tables, formatted documents)
- Explaining concepts and reasoning through problems step-by-step
- Writing and reviewing code
- Translating between formats (PDF → structured data, notes → action items)

**Weak:**
- Precise counting and arithmetic (use a calculator tool instead)
- Recalling specific facts reliably (use retrieval/RAG instead)
- Knowing anything after their training cutoff (use tools with live data instead)
- Consistency across independent calls (each call is stateless)
- Knowing what they don't know (they'll guess confidently when uncertain)

**Product design principle:** Never rely on a model for something a deterministic system does better. Use the model for language and reasoning; use databases, APIs, and tools for facts, data, and actions.

---

## 5. The model capability ladder

Different model tiers have different capabilities, latency, and cost. The hierarchy for Anthropic's Claude models (which are what you're building with):

| Model | Best for | Speed | Cost (relative) |
|-------|----------|-------|-----------------|
| **Claude Haiku 4.5** | High-volume simple tasks: classification, extraction, short responses | Fast | $ |
| **Claude Sonnet 4.6** | Most product features: writing, analysis, tool use, agents | Medium | $$ |
| **Claude Opus 4.7** | Complex reasoning, nuanced judgment, high-stakes outputs | Slower | $$$$ |

**Decision rule:** Start with Sonnet for new features. Move down to Haiku if latency or cost is a problem and quality holds. Move up to Opus only if you've validated that Sonnet's output quality is genuinely insufficient.

Don't default to Opus because it "feels safer." Haiku on a well-written prompt often outperforms Opus on a poorly-written one.

---

## 6. Training vs. context: two ways to give models knowledge

There are two fundamentally different ways a model "knows" something:

**1. Baked in at training time**
The model was trained on data that included this information. The knowledge is embedded in the model weights. You can't update it — only retrain or fine-tune, which is expensive and slow.

**2. Provided at runtime in context**
You include information in the prompt or conversation. The model can use it immediately. This is how RAG works.

**Rule of thumb:** If the knowledge changes frequently (product prices, user data, recent events), don't try to train it in — deliver it at runtime. If the capability you need is a reasoning style or tone, consider fine-tuning. For most product features, runtime context is what you'll use.

---

## 7. Fine-tuning vs. prompting: the most common misconception

Many stakeholders assume you need to "train the model on our data" to build a useful AI feature. Almost always, this is wrong.

| Approach | What it does | When to use it |
|----------|-------------|----------------|
| **Prompting** | Instructions in the system/user prompt shape behavior | Default for everything |
| **RAG** | Retrieval supplies relevant context at runtime | When you need specific facts or private data |
| **Fine-tuning** | Model weights are updated on new examples | When you need a very specific response style/format and prompting can't achieve it at acceptable cost |

Fine-tuning costs time (weeks), money, and creates maintenance burden. Exhaust prompt engineering and RAG first.

---

## How model capabilities map to PM decisions

Understanding what a model is good and bad at is only useful when it connects to a specific product decision. The table below maps each capability or limitation to the PM decision it most directly informs.

| Model Capability / Limitation | PM Decision It Informs |
|---|---|
| Strong instruction-following | Reliable output format specs are achievable — you can define a structured output (JSON schema, table, numbered list) and trust the model to follow it consistently. You don't need to accept free-form output and parse it with fragile post-processing. |
| Long context handling (200K tokens) | Document analysis features are viable without RAG for many use cases — you can load entire contracts, reports, or conversation histories into context rather than building a retrieval pipeline. Evaluate whether RAG is actually necessary before adding the complexity. |
| Weak at precise math and counting | Always route calculations through tool calls (a code interpreter, a formula, an API call). Never ask the model to compute values you need to be exact. This is a product design constraint, not an edge case. |
| Code generation quality | Developer tooling features (code completion, code review, boilerplate generation) are genuinely viable as core product features, not just demos. Quality is high enough to create real productivity lift for professional developers. |
| Hallucination on rare or specific facts | RAG is required for any feature that needs to return accurate, specific factual information (product data, regulatory details, proprietary knowledge). Don't rely on what the model "knows" — supply the facts at runtime. |
| Context window limits | A chunking strategy is required for any document or conversation above roughly 150K–180K tokens (leaving headroom for output and system prompt). Features that process large documents need an explicit plan for how content is segmented and summarised. |
| No persistent memory between sessions | Session state — what the user said earlier, their preferences, their history — must be managed by your application layer, not assumed to live in the model. Every new session starts blank unless your system explicitly loads prior context. |

Use this table when speccing features: for each capability row that applies to your feature, there is a corresponding product design or architecture decision you need to make explicitly.

---

## PM Decision Checklist — Module 3

Before speccing any AI feature, answer these:

- [ ] What's the expected input size? Does it fit comfortably in context with headroom?
- [ ] Does the feature require facts that change over time or live in our data? → Plan for RAG or tool use, not just prompting
- [ ] Is there anything in the feature that a deterministic system (database, formula, API) should handle instead of the model?
- [ ] What model tier does this realistically need? Have I justified Opus if I'm speccing it?
- [ ] Is the output structured (JSON, table) or free-form? Structured output is more reliable.
- [ ] How will I handle the case where the model is confidently wrong?
