# Signoff — Day 1

**Gap owner:** Efrata Wolde  
**Explainer author:** Lidya Dagnew  
**Status: CLOSED ✅**

---

## What I Now Understand

Before today I had written that my pipeline's latency would suffer "when the prompt grows too long" — a correct intuition with no mechanism behind it. I could not have told you *why* a longer prompt causes latency to increase, or why that is a different problem from a longer output.

After reading this explainer I can now explain both phases clearly. Prefill processes all input tokens in parallel in a single forward pass — but attention computation scales quadratically with sequence length (O(N²)), so a prompt that doubles in size costs more than double to process. During that pass, the model builds and stores the KV cache in GPU memory. Then decode begins: one token at a time, sequentially, reading the full KV cache from memory on every step. Decode is O(1) per token but memory-bandwidth-bound — the GPU idles waiting for data rather than computing.

The practical result: my conversion engine's 6.6s p50 latency is likely dominated by prefill on the large enrichment briefs I send on every call, not by the email output being generated. Cutting my output length saves less than fixing my prompt structure.

The section on prefix caching was the part I didn't know I needed. The KV cache being thrown away after every stateless API call means I am paying full prefill cost on the same static system prompt every single time. Restructuring my calls to keep the static system prompt identical across requests — and enabling prefix caching — is now a concrete next action in my Week 10 work.

---

## One Thing That Could Be Sharper

The benchmark table has some noisy rows (the 2,000-word prompt TTFT of 4,174ms is lower than the 500-word row at 6,258ms) and the note acknowledges this as "network variability." A cleaner version would either run more trials and report median, or remove the noisy rows. The core argument holds without them.

---

## Grounding Commit

See `grounding_commit.md` — I will add a latency decomposition note to my Week 10 README and a prefix caching TODO to the agent architecture section.
