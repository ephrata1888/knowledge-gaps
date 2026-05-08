# Question — Day 4

**Gap owner:** Efrata Wolde  
**Partner:** Mikias Dagem  
**Topic:** Evaluation and statistics  
**Status:** Final (post morning call)

---

## Question

My tenacious-bench uses an LLM as the judge to score model outputs. I report those scores as if they are reliable measurements, but I never tested whether the judge itself is consistent. What is inter-rater reliability, how do you measure whether an LLM judge is trustworthy, and what does an unreliable judge do to benchmark results?

---

## Grounding

This question is tied directly to `tenacious_bench_v0.1.json` which reports an 81.4 mean rubric score. That number has been used to compare model checkpoints and report fine-tuning progress. The judge that produced it has never been validated for consistency, prompt sensitivity, or agreement with a human reference. The score may be measuring model quality — or it may be measuring what the judge tends to award. I cannot tell which without running the validation protocol.
