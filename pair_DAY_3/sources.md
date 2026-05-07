# Sources — Day 3

*Sources used in Efrata's self-written explainer on prompt vs fine-tuning mechanics*

---

## Source 1

**Title:** The Illustrated Transformer  
**Author:** Jay Alammar  
**Link:** https://jalammar.github.io/illustrated-transformer/  
**Why it matters:** The clearest visual explanation of how transformer weights encode learned associations and how forward passes produce token predictions. Essential grounding for understanding why prompts work within existing weight space rather than modifying it.

---

## Source 2

**Title:** LIMA: Less Is More for Alignment  
**Authors:** Zhou et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2305.11206  
**Why it matters:** Demonstrates that a small number of high-quality fine-tuning examples can produce strong stylistic alignment, supporting the claim that tone is a distributed signal that transfers efficiently with limited data. Also implicitly supports the point that factual knowledge is harder to install via fine-tuning alone.

---

## Note

No partner tool used today — partner was unreachable. Research conducted independently using the above sources plus Anthropic's fine-tuning documentation.
