# Grounding Commit — Day 4

**Gap closed:** What inter-rater reliability is, how to measure whether an LLM judge is trustworthy, and why unreliable judge bias is systematic and directional rather than random noise.

---

## Commit 1 — tenacious-bench README

**File:** `README.md` (tenacious-bench repo)  
**What to add:** A validation status note under the benchmark scores section.

**Add under the reported scores:**

> ⚠️ **Judge Validation Status:** The LLM judge used in v0.1 has not yet been validated for inter-rater reliability. The 81.4 mean rubric score should be treated as a hypothesis rather than a confirmed measurement until the following three checks pass:
> 1. Temperature audit — judge confirmed running at temperature = 0
> 2. Perturbation sweep — JSS ≥ 0.85 across prompt rephrasing variants
> 3. QWK ≥ 0.80 against a human-annotated reference set (n=30)
>
> See `scripts/judge_validation.py` for the validation protocol.

**Why this matters:** Anyone reading the benchmark — including future me — needs to know the score has not been validated. Publishing it without this caveat implies a reliability that hasn't been earned.

---

## Commit 2 — judge validation script

**File:** `scripts/judge_validation.py` (new file, tenacious-bench repo)  
**What to add:** A script that runs the three-step validation protocol described in the explainer.

```python
"""
Judge validation protocol for tenacious-bench.
Run this before interpreting any benchmark scores as reliable measurements.

Three checks:
1. Intra-rater stability (temperature audit)
2. Prompt sensitivity (JSS across rephrasing variants)  
3. Agreement with human reference (QWK)

Targets: JSS >= 0.85, QWK >= 0.80
"""

import json
import random
from sklearn.metrics import cohen_kappa_score

KNOWN_DIMENSIONS = {"no_bench_word", "tone_match", "objection_handling", "icp_fit"}
JSS_THRESHOLD = 0.85
QWK_THRESHOLD = 0.80

def run_stability_check(examples, judge_fn, n=50):
    """Re-score n examples and check for disagreements."""
    sample = random.sample(examples, min(n, len(examples)))
    disagreements = 0
    for ex in sample:
        score_1 = judge_fn(ex)
        score_2 = judge_fn(ex)
        if score_1 != score_2:
            disagreements += 1
    jss = 1 - (disagreements / len(sample))
    print(f"Intra-rater stability: {jss:.2f} ({disagreements}/{len(sample)} disagreements)")
    return jss

def run_perturbation_sweep(examples, judge_variants, n=50):
    """Compute JSS across prompt rephrasing variants."""
    sample = random.sample(examples, min(n, len(examples)))
    stable = 0
    for ex in sample:
        scores = [judge(ex) for judge in judge_variants]
        if len(set(scores)) == 1:
            stable += 1
    jss = stable / len(sample)
    print(f"Prompt sensitivity JSS: {jss:.2f} (target >= {JSS_THRESHOLD})")
    if jss < JSS_THRESHOLD:
        print("  WARNING: Score variance is coming from prompt surface, not response quality.")
    return jss

def run_qwk_check(judge_scores, human_scores):
    """Compute Quadratic Weighted Kappa against human annotations."""
    qwk = cohen_kappa_score(judge_scores, human_scores, weights="quadratic")
    print(f"QWK vs human reference: {qwk:.2f} (target >= {QWK_THRESHOLD})")
    if qwk < QWK_THRESHOLD:
        print("  WARNING: Judge is not reliably measuring rubric criteria. Do not use for model selection.")
    return qwk
```

**Why this matters:** The validation protocol now exists as runnable code in the repo, not just as a note. Anyone inheriting this benchmark can run it before trusting the scores.
