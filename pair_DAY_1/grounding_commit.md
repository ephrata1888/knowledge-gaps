# Grounding Commit — Day 1

**Gap closed:** What the KV cache is, what happens during prefill vs decode, and why prompt length affects latency differently from output length.

---

## Commit 1 — conversion-engine README

**File:** `README.md`  
**What I changed:** Added a latency decomposition note under the existing p50/p95 benchmark numbers.

**Before:**
> p50 = 6.6s · p95 = 7.3s · Source: eval/latency_results.json

**After:**
> p50 = 6.6s · p95 = 7.3s · Source: eval/latency_results.json  
> ⚠️ Note: These are end-to-end numbers only. Current instrumentation does not decompose latency into prefill (input processing) vs decode (output generation) vs external API roundtrip (HubSpot, Resend, Cal.com). Based on Day 1 gap research, the dominant cost is likely prefill on the enrichment brief sent on every call — not decode. A per-phase breakdown is a known gap in this benchmark.

**Why this matters:** The number I was reporting was real but misleading. Anyone reading it could not tell whether to optimise the prompt, the generation, or the external APIs. This note makes the gap explicit rather than hiding it.

---

## Commit 2 — conversion-engine agent architecture

**File:** `method.md`  
**What I changed:** Added a prefix caching TODO to the architecture section that describes how the system prompt and enrichment brief are assembled.

**Added:**
> TODO (prefix caching): The static system prompt is currently regenerated and sent in full on every LLM call. Because the KV cache is discarded after each stateless API call, we pay full prefill cost on identical content every time. Fix: separate static system prompt (identical across all calls) from dynamic enrichment brief (per-prospect), and enable prefix caching on the static portion. Expected impact: meaningful TTFT reduction at scale, lower cost per call. See: Anthropic prompt caching docs, vLLM prefix caching.

**Why this matters:** This is a direct architecture improvement unlocked by today's gap. It did not exist in my Week 10 submission. It now does.
