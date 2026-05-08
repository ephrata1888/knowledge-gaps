# Signoff — Day 4

**Gap owner:** Efrata Wolde  
**Explainer author:** Mikias Dagem  
**Status: CLOSED ✅**

---

## What I Now Understand

Before today I was reporting an 81.4 mean rubric score from tenacious-bench as if it were a measurement of model quality. I had never tested whether the judge that produced it was consistent, prompt-sensitive, or aligned with what the rubric was actually trying to capture. I was treating a noisy probabilistic scorer as ground truth.

Mikias's explainer gave me a precise framework for what to test and how.

Inter-rater reliability has two distinct components I had conflated. Intra-rater reliability asks whether the same judge returns the same verdict on the same input across repeated calls — at temperature > 0 this is not guaranteed, because the judge is sampling from a distribution at the scoring token. Inter-rater reliability asks whether the judge agrees with a separate reference standard applying the same rubric. High intra-rater reliability is a necessary precondition for trusting scores but not sufficient — a judge can be perfectly consistent and consistently wrong if it is biased toward surface features rather than rubric criteria.

The part that changed my thinking most: unreliable judge bias is systematic and directional, not random. I had assumed noise would average out across enough examples. It doesn't. Position bias, length bias, and prompt-phrasing sensitivity each push scores in a consistent direction. An inflated 81.4 looks identical to a legitimately earned 81.4 — the only way to detect it is to measure agreement against a reference that doesn't share the same biases.

The three-step validation protocol is now a concrete next action:
1. Temperature audit — confirm judge runs at temperature = 0
2. Perturbation sweep — compute JSS across prompt rephrasing; target ≥ 0.85
3. QWK against human reference — annotate 30 examples myself; target ≥ 0.80

Until those three checks pass, the 81.4 figure is a hypothesis, not a measurement.

---

## One Thing That Could Be Sharper

The explainer introduces Quadratic Weighted Kappa (QWK) without explaining why it's preferable to simpler agreement metrics like percentage agreement or Cohen's Kappa. A one-sentence explanation — that QWK penalises large disagreements more than small ones, which matters for ordinal rubrics — would have made the metric choice feel motivated rather than prescribed.

---

## Grounding Commit

See `day4_grounding_commit.md` — one addition to the tenacious-bench README and one validation script to add to the repo.
