# Sources — Day 4

*Canonical sources cited in Mikias Dagem's explainer for Efrata's question*

---

## Paper 1

**Title:** Rating Roulette: Self-Inconsistency in LLM-As-A-Judge Frameworks  
**Authors:** Haldar, R. & Hockenmaier, J.  
**Year:** 2025  
**Venue:** EMNLP 2025  
**Why it matters:** Directly measures intra-rater inconsistency in LLM judges — demonstrates that at temperature > 0, the same judge on the same input returns different scores, and quantifies how much variance comes from sampling noise vs genuine quality signal.

---

## Paper 2

**Title:** JudgeSense: A Benchmark for Prompt Sensitivity in LLM-as-a-Judge Systems  
**Authors:** Bellibatlu et al.  
**Year:** 2026  
**Link:** arxiv.org/abs/2604.23478  
**Why it matters:** Introduces and validates the Judge Sensitivity Score (JSS) metric used in the three-step validation protocol. Establishes the 0.85 threshold as the boundary below which prompt-surface variance is dominating score variance.

---

## Paper 3

**Title:** Diagnosing LLM Judge Reliability Using Item Response Theory  
**Authors:** Choi et al.  
**Year:** 2026  
**Link:** arxiv.org/abs/2602.00521  
**Why it matters:** Provides the theoretical grounding for why QWK is the appropriate metric for ordinal rubric agreement — penalises large disagreements more than small ones, which matches the structure of a 1–5 scoring rubric.

---

## Tool Used

**Name:** Anthropic Python SDK — LLM judge calls via `client.messages.create`  
**Used for:** Running the three-step validation protocol against a sample of tenacious-bench examples to verify the JSS and QWK measurement approach described in the explainer.
