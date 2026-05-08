# Why a Silent Default Breaks Your Benchmark: Defensive Patterns for Deterministic Evaluators

*Week 12 · Day 4 · Written by Efrata Wolde for [Partner Name]*

---

## The Bug That Looks Like a Feature

In `scoring_evaluator.py`, the `score_dimension()` dispatcher ends with `return 1.0` for any dimension it doesn't recognise. This looks harmless — maybe even defensive. The code doesn't crash. The benchmark runs to completion. Scores come out.

But the scores are wrong, and worse, they're wrong in a direction that makes your model look better than it is. That's the most dangerous kind of bug in an evaluation system: one that fails silently and optimistically.

---

## What "Systematic Inflation" Actually Means

When a task JSONL contains `no_bench_words` instead of `no_bench_word`, here is what happens:

1. The dispatcher receives `"no_bench_words"` as the dimension name
2. It checks its known cases — no match found
3. It falls through to `return 1.0`
4. Your benchmark records a perfect score for that dimension
5. That perfect score gets averaged into the task's overall score
6. The task scores higher than it should
7. Every task with this typo scores higher than it should
8. Your aggregate benchmark results are inflated

The word "systematic" is important here. This isn't random noise — it inflates scores in one direction only, always upward, for every task that hits the unrecognised dimension. If 30% of your tasks have this typo, 30% of your tasks have an artificially boosted score. The inflation is correlated with the typo frequency, not distributed randomly. That means it doesn't average out — it accumulates.

---

## Why This Is Worse Than a Crash

A crash is honest. If `score_dimension()` raised a `KeyError` on an unrecognised dimension, you would know immediately that something was wrong. You'd fix the typo, rerun the benchmark, and get correct results.

A silent `return 1.0` is dishonest. The benchmark appears to have run correctly. The scores look plausible. You publish them, compare models against them, make fine-tuning decisions based on them — and all of it is built on corrupted measurements you have no reason to question.

In evaluation systems, silent failures are categorically worse than loud failures. A loud failure stops you. A silent failure misdirects you.

---

## The Correct Defensive Pattern

There are three layers of defence a deterministic evaluator should have. Apply all three.

### Layer 1 — Fail loudly on unrecognised dimensions

Replace the silent default with an explicit error:

```python
def score_dimension(dimension: str, output: str, rubric: dict) -> float:
    if dimension == "no_bench_word":
        return score_no_bench_word(output, rubric)
    elif dimension == "tone_match":
        return score_tone_match(output, rubric)
    elif dimension == "objection_handling":
        return score_objection_handling(output, rubric)
    # ... other known dimensions ...
    else:
        raise ValueError(
            f"Unrecognised dimension: '{dimension}'. "
            f"Known dimensions: {list(KNOWN_DIMENSIONS)}. "
            f"Check your task JSONL for typos."
        )
```

Now a typo crashes the run immediately with a message that tells you exactly what went wrong and where to look. You cannot accidentally publish inflated scores.

### Layer 2 — Validate task JSONL against a known schema at load time

Don't wait until scoring time to discover bad dimension names. Validate your task definitions when you load them:

```python
KNOWN_DIMENSIONS = {"no_bench_word", "tone_match", "objection_handling", "icp_fit"}

def load_task(task: dict) -> dict:
    for dimension in task.get("dimensions", []):
        if dimension not in KNOWN_DIMENSIONS:
            raise ValueError(
                f"Task '{task['id']}' contains unknown dimension '{dimension}'. "
                f"Valid dimensions: {KNOWN_DIMENSIONS}"
            )
    return task
```

This catches the problem before any scoring runs. Your benchmark fails at load time, not mid-run, with a clear message pointing to the exact task and dimension that's wrong.

### Layer 3 — Assert score coverage after each task

After scoring a task, verify that every expected dimension produced a score and no unexpected dimensions were scored:

```python
def score_task(task: dict, output: str) -> dict:
    expected_dimensions = set(task["dimensions"])
    scores = {}
    
    for dimension in expected_dimensions:
        scores[dimension] = score_dimension(dimension, output, task["rubric"])
    
    # Assert nothing was missed or added
    assert set(scores.keys()) == expected_dimensions, (
        f"Score coverage mismatch for task '{task['id']}': "
        f"expected {expected_dimensions}, got {set(scores.keys())}"
    )
    
    return scores
```

This makes the contract explicit: every task must be scored against exactly its declared dimensions — no more, no less.

---

## The Deeper Principle: Deterministic Evaluators Must Be Strict

The reason these three layers matter is that an evaluation system has a different failure mode profile than a production system.

In production, graceful degradation is often correct. If a feature flag is missing, use the default. If a config key is absent, fall back to a safe value. Keeping the system running is usually more important than being precise about edge cases.

In evaluation, graceful degradation is almost always wrong. The entire purpose of an evaluator is to produce reliable measurements. If the evaluator is lenient about its inputs, its outputs are not measurements — they are guesses. And unlike production failures, evaluation failures don't surface as errors in your logs. They surface as subtly wrong beliefs about your model's performance.

A good heuristic: **in evaluation code, every ambiguous case should raise, not default.** If you don't know what to score, you shouldn't score anything. An honest `null` or a loud error is always more useful than a silent `1.0`.

---

## What to Fix in Tenacious-Bench Right Now

Three concrete changes:

1. **Replace `return 1.0` with `raise ValueError`** in the `score_dimension()` fallthrough
2. **Add a JSONL validation step** at benchmark load time that checks all dimension names against `KNOWN_DIMENSIONS`
3. **Audit your existing benchmark results** — any run that included tasks with `no_bench_words` (or other unrecognised dimensions) produced inflated scores. Those results need to be rerun after the fix before being used for model comparisons

The third point is the uncomfortable one. The fix isn't just making the code correct going forward — it means acknowledging that some of your existing scores are wrong and rerunning the affected tasks.

---

## The One-Paragraph Answer

A silent permissive default inflates benchmark scores because it awards full marks for dimensions it cannot evaluate, systematically biasing results upward for every task that triggers the fallthrough — and unlike random noise, this bias doesn't average out across runs. The correct defensive pattern has three layers: fail loudly on unrecognised dimension names so the error surfaces immediately; validate task JSONL against a known schema at load time so bad inputs never reach the scorer; and assert score coverage after each task so the contract between task definition and scoring is enforced explicitly. In evaluation systems, a loud failure is always better than a silent one — an evaluator that degrades gracefully produces corrupt measurements, not safe defaults.

---

*Sources: Holistic Evaluation of Language Models (Liang et al., 2022, arxiv.org/abs/2211.09110) · Judging the Judges: Evaluating Alignment and Vulnerabilities in LLMs-as-Judges (Panickssery et al., 2024, arxiv.org/abs/2406.12334)*
