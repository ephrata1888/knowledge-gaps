# Week 12 Synthesis

**Author:** Efrata Wolde  
**Week:** 12 — Knowledge Gap Formulation for Compounding  
**Submitted:** Saturday, May 2026

---

## Overview

This week I closed ten gaps across five days of paired research. Five gaps I named — places in my own Week 10 and 11 work where I was using language I couldn't defend. Five gaps I researched for partners — questions grounded in their work that I had to understand well enough to explain in public. What follows is a synthesis of what I learned, what surprised me, and what I would do differently in my existing portfolio.

---

## The Ten Gaps Closed

### Gap 1 — What the KV cache is and why prompt length affects latency (Day 1, named)

I had written in my conversion-engine blog that "latency suffers when the prompt grows too long." Correct intuition, no mechanism. I couldn't explain why prompt length and output length affect latency differently.

The mechanism: every LLM call has two phases. Prefill processes all input tokens in parallel in a single forward pass, but attention scales quadratically with sequence length — doubling the prompt costs more than double. During prefill, the model builds the KV cache: key and value matrices for every token at every layer, stored in GPU VRAM. Decode then generates output one token at a time sequentially, reading the KV cache on every step. This is memory-bandwidth-bound and cannot be parallelised. The cache disappears when the API call ends — meaning my pipeline was paying full prefill cost on the same static system prompt every single call.

The grounding commit: flagged my p50/p95 numbers as end-to-end only, and added a prefix caching TODO to method.md.

---

### Gap 2 — Why output tokens cost 3× more than input tokens (Day 1, researched for Lidya Dagnew)

Lidya's question: her API charges 3× more for output tokens and she had assumed cost scales linearly with prompt length.

The mechanism is the same prefill/decode asymmetry. 1,000 input tokens = ~1 forward pass. 1,000 output tokens = ~1,000 forward passes, each one reading the full KV cache from GPU memory. API providers sell GPU time — the 3× pricing is honest. The practical implication: cutting output length has higher ROI than cutting input length when optimising cost.

---

### Gap 3 — What happens at the token level when a model calls a tool (Day 2, named)

My conversion-engine passes tool schemas to an LLM and I had been writing "the agent calls HubSpot" as if the model executed the function. It doesn't.

The mechanism: tool schemas are serialised into the context window as tokens. When the model determines a tool call is appropriate, it produces a structured `tool_use` content block — JSON with type, id, name, and input fields. The framework intercepts this and runs the actual function. The model never executes anything. The key field is `stop_reason: "tool_use"` — the agent loop should be keyed on this, not on parsing the content blocks directly. This is fine-tuning, not constrained decoding.

The grounding commit: two code fixes to the agent loop — explicit `stop_reason: "max_tokens"` handling, and try/catch around all tool executions to always return a `tool_result` even on failure.

---

### Gap 4 — What causes a 682-second agent loop (Day 2, researched for Zemzem Hibet)

Zemzem's traces showed her lead gen agent looping for 682–687 seconds before `user_stop` with `reward 0.0`. She couldn't diagnose it without understanding the token-level mechanism.

Three failure modes: (1) repeating the same tool call because the result came back in a format the model can't interpret as progress; (2) alternating between tools in a planning loop with no state change; (3) failing to emit a tool call at all. For a 682-second loop with active execution, failure mode 1 or 2 is most likely. The root cause is usually that the tool execution path threw an exception and no `tool_result` was returned — so the model called the same tool again, indefinitely. The environment terminated it because the model never emitted a final text response.

---

### Gap 5 — What fine-tuning changes that a prompt cannot (Day 3, named)

I had argued in my Week 11 blog that fine-tuning was necessary for tone transfer without being able to defend why a better prompt wouldn't suffice.

A prompt adds tokens to the context window and activates behaviours the model already knows. It cannot modify weights and therefore cannot install new behaviours. Fine-tuning runs gradient descent on training data and permanently modifies the model's weights — shifting internal representations toward the training distribution. Tone transfers efficiently because it is a distributed stylistic signal present in every training example. Product knowledge transfers poorly because it is sparse and specific — a small dataset cannot provide enough signal for reliable factual recall. The right architecture is fine-tune for tone, RAG for facts.

The grounding commit: added a mechanistic explanation to the Week 11 blog, and a RAG TODO to method.md for product knowledge.

---

### Gap 6 — [Day 3 partner gap — to be completed after Day 5]

*Partner was unreachable on Day 3. This gap reflects solo research. See pair_DAY_3 for full documentation.*

---

### Gap 7 — Why a silent permissive default inflates benchmark scores (Day 4, researched for Mikias Dagem)

Mikias's `score_dimension()` dispatcher returned `return 1.0` for any unrecognised dimension name — silently awarding full marks rather than raising an error. A typo (`no_bench_words` vs `no_bench_word`) caused perfect scores for dimensions never evaluated.

The key insight: this inflation is systematic and directional, not random. It only goes upward. 30% typo rate = 30% of tasks artificially boosted — it accumulates, it doesn't average out. The correct defensive pattern has three layers: raise on unrecognised dimensions, validate task JSONL at load time against a known schema, and assert score coverage after each task. In evaluation code, loud failures are always better than silent defaults.

---

### Gap 8 — What inter-rater reliability is and what an unreliable LLM judge does to benchmark results (Day 4, named)

My tenacious-bench reports an 81.4 mean rubric score from an LLM judge I had never validated. I was treating a noisy probabilistic scorer as ground truth.

IRR has two components: intra-rater reliability (does the same judge return the same verdict on the same input across repeated calls?) and inter-rater reliability (does the judge agree with a separate reference standard?). The critical insight: LLM judge bias is systematic and directional — position bias, length bias, and prompt-phrasing sensitivity each push scores in a consistent direction, not randomly. An inflated 81.4 looks identical to a legitimately earned 81.4. The three-step validation protocol: temperature audit, perturbation sweep targeting JSS ≥ 0.85, and QWK ≥ 0.80 against a human-annotated reference set.

The grounding commit: added a validation status warning to the README and a `judge_validation.py` script to the repo.

---

### Gaps 9 and 10 — [Day 5 — to be completed]

*To be added after Day 5 is complete.*

---

## The Most Surprising Thing I Learned

The thing that most changed how I think: **systematic bias is invisible in score distributions.**

I had assumed that if my LLM judge was noisy, the noise would average out across enough examples. It doesn't — and can't. Position bias, length bias, and prompt-phrasing sensitivity are directional. They consistently push scores in one direction. An inflated benchmark looks exactly like a good benchmark. The only way to detect it is to measure agreement against a reference that doesn't share the same biases.

This applies beyond evaluation. The same logic holds for any measurement system where the instrument has a consistent directional bias — A/B test metrics, sales pipeline scores, model comparisons. Random noise averages out. Systematic bias doesn't. And systematic bias is exactly what you get when your measurement instrument is a fine-tuned model trained on the same distribution as the thing you're measuring.

---

## What I Would Do Differently in Weeks 10 and 11

Three things, in priority order:

**First:** Instrument my latency numbers properly. Reporting a single end-to-end p95 without decomposing into prefill, decode, and external API time is not a benchmark — it's a black box number. I cannot optimise what I cannot measure.

**Second:** Validate my judge before publishing scores. The 81.4 figure in tenacious-bench is a hypothesis until the three-step IRR protocol passes. I should have run this before Week 11 submission.

**Third:** Separate static from dynamic context in every LLM call. My pipeline sends a static system prompt on every call and rebuilds the KV cache every time. Prefix caching is a one-time architectural fix that would reduce TTFT and cost at scale.

---

## Canonical Reading List

See `canonical_list.md` for the full annotated list. The five papers I would most strongly recommend to another FDE building LLM pipelines:

1. Kwon et al. (2023) — PagedAttention / vLLM: the paper that explains KV cache memory management in production serving
2. Patil et al. (2023) — Gorilla: why tool description quality directly affects call accuracy
3. Zhou et al. (2023) — LIMA: why a small number of high-quality fine-tuning examples produces strong stylistic alignment
4. Panickssery et al. (2024) — Judging the Judges: LLM judge reliability and bias
5. Bellibatlu et al. (2026) — JudgeSense: the JSS metric and prompt sensitivity in LLM-as-Judge systems
