# Question — Day 1

**Gap owner:** Efrata Wolde  
**Topic:** Inference-time mechanics  
**Status:** Final (post morning call)

---

## Question

My conversion-engine blog says the system prompt approach works "until the prompt grows so long that latency suffers." My pipeline currently sends a system prompt + a full enrichment brief on every LLM call. What is the KV cache, what actually happens computationally when the model processes a long input prompt, and why does prompt length affect latency in a way that output length does not affect it the same way?

---

## Grounding

This question is tied to a specific claim in my Week 11 blog post for the tenacious-bench repo. The claim was correct as an engineering intuition but I could not explain the mechanism behind it. The relevant line: *"prompt engineering is the right first intervention... The training path becomes interesting when the prompt grows so long that latency suffers."*

My pipeline (conversion-engine) sends a static system prompt plus a dynamically generated enrichment brief (Crunchbase, hiring signals, competitor gap analysis) on every LLM call. I have no instrumentation that separates prefill latency from decode latency. This question is the first step toward understanding where my p95 = 7.3s is actually coming from.

---

## Why This Is a Real Gap

In my Week 10 and 11 work I used phrases like "latency suffers when the prompt is too long" without being able to explain why at the computational level. I could not have described what the KV cache is, why it exists, or why input and output tokens are processed differently. This gap was confirmed during the morning call — my partner's question revealed she had the same blind spot from a cost angle, which means this is a common FDE gap worth documenting publicly.
