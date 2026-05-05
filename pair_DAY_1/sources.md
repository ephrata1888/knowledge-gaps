# Sources — Day 1

*Canonical sources cited in Lidya Dagnew's explainer for Efrata's question*

---

## Paper 1

**Title:** Attention Is All You Need  
**Authors:** Vaswani et al.  
**Year:** 2017  
**Link:** https://arxiv.org/abs/1706.03762  
**Why it matters:** The foundational paper introducing the Transformer architecture and self-attention mechanism. Explains why attention computation scales quadratically with sequence length — the core reason prefill cost grows aggressively with prompt length.

---

## Paper 2

**Title:** Efficient Memory Management for Large Language Model Serving with PagedAttention  
**Authors:** Kwon et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2309.06180  
**Why it matters:** Introduces PagedAttention and the vLLM serving framework. Explains how KV cache memory is managed in production inference, why the cache is discarded between stateless calls, and how systems like vLLM implement prefix caching to reuse KV cache across calls with identical prefixes.

---

## Tool

**Name:** OpenAI API (gpt-4o-mini via direct API calls)  
**Used for:** Benchmarking TTFT (time-to-first-token) and decode time across varying prompt and output lengths to empirically verify the prefill vs decode scaling behaviour described in the explainer.
