# Tweet Thread — Day 4

*For Mikias Dagem's question on silent permissive defaults in benchmark evaluators*

---

**1/**
There's a bug in my benchmark that makes my model look better than it is.

The scorer returns a perfect score for any dimension name it doesn't recognise.
No error. No warning. Just 1.0.

Here's why this is the most dangerous kind of evaluation bug 🧵

---

**2/**
The specific failure: `score_dimension()` in `scoring_evaluator.py` ends with `return 1.0`.

When a task JSONL has a typo — `no_bench_words` instead of `no_bench_word` — the scorer can't find a match, falls through, and awards a perfect score for a dimension it never actually evaluated.

---

**3/**
You might think: "that's just a little noise, it'll average out."

It won't.

The inflation is **systematic**, not random. It only goes in one direction — upward. Every task with the typo gets boosted. 30% typo rate = 30% of tasks artificially inflated. It accumulates, it doesn't cancel.

---

**4/**
The reason silent failures are worse than crashes:

A crash is honest. You know immediately something is wrong. You fix it.

A `return 1.0` is dishonest. The benchmark runs to completion. Scores look plausible. You publish them, compare models against them, make fine-tuning decisions — all built on corrupted measurements.

---

**5/**
The fix has three layers.

**Layer 1 — Fail loudly:**
```python
else:
    raise ValueError(
        f"Unrecognised dimension: '{dimension}'. "
        f"Known: {list(KNOWN_DIMENSIONS)}"
    )
```

Now a typo crashes immediately with a message pointing exactly to the problem.

---

**6/**
**Layer 2 — Validate at load time:**

Don't wait until scoring to discover bad dimension names. Check task JSONL against `KNOWN_DIMENSIONS` when you load it.

The benchmark fails before any scoring runs, with a message pointing to the exact task and dimension that's wrong.

---

**7/**
**Layer 3 — Assert score coverage:**

After scoring each task, verify that every expected dimension produced a score and no unexpected ones were scored.

Make the contract explicit: every task must be scored against exactly its declared dimensions — no more, no less.

---

**8/**
The deeper principle:

In production code, graceful degradation is often correct. If a config key is missing, use a safe default. Keep the system running.

In evaluation code, graceful degradation is almost always wrong. If you don't know what to score, you shouldn't score anything.

---

**9/**
A good heuristic for eval code:

**Every ambiguous case should raise, not default.**

An honest null or a loud error is always more useful than a silent 1.0. You can fix an error. You cannot fix a belief about your model's performance that was built on a corrupted measurement.

---

**10/**
TL;DR

- Silent `return 1.0` = perfect score for dimensions never evaluated
- Inflation is systematic and directional — it doesn't average out
- Fix: raise on unrecognised dimensions, validate JSONL at load time, assert score coverage after each task
- Also: audit existing results — any run that hit the fallthrough produced inflated scores and needs to be rerun
- In eval code: loud failures > silent defaults, always
