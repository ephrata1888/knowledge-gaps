# Portfolio Update — Week 12

**Author:** Efrata Wolde  
**For:** FDE Hiring Manager  
**Repos:** github.com/ephrata1888/conversion-engine · github.com/ephrata1888/tenacious-bench

---

## What Week 12 Did to My Week 10 and 11 Work

Week 12 was gap-finding week. The work was to go back through the systems I had built, find the places where I was using language I couldn't defend, close those gaps through paired research, and make concrete improvements to the existing portfolio. This document summarises the five grounding commits that came out of that process and what they mean for the quality of my Week 10 and 11 submissions.

---

## Commit 1 — Latency benchmark honestly labelled (conversion-engine README)

**Before:** My README reported `p50 = 6.6s · p95 = 7.3s` as a clean performance number.

**After:** The same numbers now carry an explicit note: these are end-to-end figures only. The instrumentation cannot distinguish prefill latency (model reading the prompt), decode latency (model generating the email), or external API roundtrip time (HubSpot, Resend, Cal.com). The number is real but it is a black box — it cannot tell you where the bottleneck is or what to optimise.

**Why this matters to an FDE hiring manager:** Reporting an undecomposed latency number as a benchmark is a common junior mistake. Catching it, labelling it honestly, and explaining why the decomposition matters demonstrates measurement discipline. An FDE who can't instrument their own pipeline can't debug it under production conditions.

---

## Commit 2 — Prefix caching TODO added to architecture (conversion-engine method.md)

**Before:** My pipeline sent the same large static system prompt on every LLM call. The KV cache for that prompt was rebuilt from scratch on every call. The architecture notes made no mention of this cost.

**After:** method.md now contains a prefix caching TODO that identifies the specific fix: separate the static system prompt from the dynamic per-prospect enrichment brief, keep the static portion byte-for-byte identical across calls, and enable prefix caching. Expected impact: reduced TTFT at scale, lower cost per call.

**Why this matters:** This is not a cosmetic note — it is a concrete architecture improvement that emerged from understanding the mechanism behind my own latency numbers. The TODO is actionable and scoped. A hiring manager reading this can see that I understand the cost structure of my pipeline and have a specific next step planned.

---

## Commit 3 — Agent loop hardened against silent failures (conversion-engine agent)

**Before:** My agent loop did not handle `stop_reason: "max_tokens"` explicitly. Tool execution paths did not guarantee a `tool_result` block on failure. Both are latent looping bugs — conditions under which the agent would call the same tool indefinitely until the environment terminated it.

**After:** Two code fixes. Explicit handling for all three `stop_reason` values — `end_turn`, `tool_use`, and `max_tokens`. Try/catch around every tool execution path with `is_error: true` returned on failure. The agent loop now has a defined exit condition for every state it can enter.

**Why this matters:** An agent that loops indefinitely in production is a support ticket and a cost incident. The ability to read a 682-second loop trace, identify the failure mode at the mechanism level, and close the latent bug in my own pipeline is exactly the diagnostic skill an FDE needs. I found this bug in my own code by understanding what caused it in a partner's traces.

---

## Commit 4 — Week 11 blog mechanistically grounded (tenacious-bench)

**Before:** My Week 11 blog argued that fine-tuning was necessary for tone transfer and reported partial transfer results (tone transferred, product knowledge didn't). The recommendation was correct but I couldn't defend it at the mechanism level.

**After:** The blog now contains a paragraph explaining *why* fine-tuning was the right intervention for tone and *why* it was the wrong intervention for product knowledge — at the weight level. Tone is a distributed stylistic signal present in every training example; the model updates toward it efficiently with small datasets. Product knowledge is sparse and specific; small datasets cannot provide enough signal for reliable factual recall. The blog also now contains a RAG TODO for the product knowledge gap.

**Why this matters:** Making the right architectural call by intuition is not the same as being able to defend it in a client conversation. The updated blog demonstrates that I understand the mechanism well enough to teach it, not just apply it.

---

## Commit 5 — Judge validation protocol added to tenacious-bench

**Before:** My tenacious-bench published an 81.4 mean rubric score from an LLM judge that had never been validated. No inter-rater reliability check, no prompt sensitivity measurement, no agreement with a human reference standard.

**After:** Two additions. The README now carries an explicit validation status warning: the 81.4 figure should be treated as a hypothesis until the three-step IRR protocol passes (temperature audit, JSS ≥ 0.85, QWK ≥ 0.80 against human annotations). A `scripts/judge_validation.py` script implements the protocol and can be run before any future benchmark results are published.

**Why this matters:** Publishing benchmark scores without validating the measurement instrument is the evaluation equivalent of publishing a p50 latency without instrumentation. Both look like rigorous numbers; neither can be trusted without the underlying validation work. An FDE who ships evaluation infrastructure needs to know whether their judge is measuring model quality or measuring the judge's own biases.

---

## The Collective Picture

Taken together, these five commits do one thing: they make my Week 10 and 11 portfolio honest. The systems I built still work. The results I reported are still real. But five places where I was asserting more certainty than I had earned are now either labelled as gaps with concrete next steps, or fixed outright.

The conversion-engine is now a system whose latency limitations are documented, whose agent loop won't silently spiral, and whose architecture notes reflect the actual cost structure. The tenacious-bench is now a system whose scores come with a validation status and whose measurement instrument has a published protocol for verification.

That is what Week 12 was for.
