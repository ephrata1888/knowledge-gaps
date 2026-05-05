# Tweet Thread — Day 1

*For: Lidya Dagnew's question on prefill vs decode*

---

**1/**
Your API charges 3× more for output tokens than input tokens.

Most engineers assume this is arbitrary pricing.

It's not. It reflects a real asymmetry in how the GPU does work.

Here's what actually happens inside an LLM inference call 🧵

---

**2/**
Every LLM call has two distinct phases: Prefill and Decode.

They run on completely different computational principles.

Understanding both changes how you build and optimise LLM pipelines.

---

**3/**
**Prefill** = processing your input prompt.

All input tokens are processed in parallel in a single forward pass.

The GPU loves this — it's built for parallel matrix multiplication.

But attention computation scales quadratically with sequence length (O(N²)).

Double the prompt → more than double the prefill cost.

---

**4/**
During prefill, the model builds and stores the **KV cache** — key and value matrices for every layer, for every input token.

This lives in GPU memory and becomes the model's "memory" of your prompt.

It won't be recomputed. But it will be read — a lot.

---

**5/**
**Decode** = generating your output, one token at a time.

Token #1 → forward pass.
Token #2 → another forward pass (reads full KV cache).
Token #3 → another pass (KV cache just got bigger).

There is no parallelism here. Each token depends on the last.

---

**6/**
Decode is memory-bandwidth-bound.

The GPU isn't limited by arithmetic — it's limited by how fast it can read the KV cache from memory on every step.

The arithmetic units sit mostly idle, waiting for data.

This is the bottleneck nobody talks about.

---

**7/**
So why does output cost 3×?

1,000 input tokens = ~1 forward pass (parallel, GPU fully used)
1,000 output tokens = ~1,000 forward passes (sequential, GPU mostly waiting)

API providers sell you GPU time. Output tokens consume far more of it per token.

The pricing is honest.

---

**8/**
Practical implication:

If you want to cut costs, **reducing output length has higher ROI than reducing input length.**

A shorter system prompt saves some prefill cost.
A shorter generation target saves more — per token removed.

Optimise decode first.

---

**9/**
One more gotcha: the KV cache is thrown away after every stateless API call.

If your pipeline sends the same 2,000-token system prompt on every call, you're paying full prefill cost each time — even for identical content.

The fix is **prefix caching** — keep the static prefix identical so the infrastructure can reuse the KV cache across calls.

---

**10/**
TL;DR

- Prefill: parallel, compute-bound, scales with prompt length (super-linearly)
- Decode: sequential, memory-bandwidth-bound, scales linearly with output length
- 3× output pricing = real GPU time difference, not arbitrary markup
- Long static prompts = wasted prefill cost → fix with prefix caching
