# Grounding Commit — Day 3

**Gap closed:** What a prompt does vs what fine-tuning does at the weight level, and why tone transfers better than product knowledge with small-dataset fine-tuning.

---

## Commit 1 — Week 11 blog post

**File:** Week 11 blog (tenacious-bench repo)  
**What to add:** A paragraph explaining *why* fine-tuning was chosen over continued prompt engineering, now that the mechanism is understood.

**Add after the section reporting partial transfer results:**

> The tone/knowledge transfer gap is not surprising once you understand what fine-tuning is doing. A prompt activates behaviours the model already knows — it cannot install new ones or change the model's learned associations. Fine-tuning modifies the weights directly via gradient descent, permanently shifting the model's internal representations toward the training distribution. Tone transfers efficiently because it is a distributed stylistic signal present across every training example. Product knowledge transfers poorly because it is sparse and specific — a small dataset cannot provide enough signal for reliable factual recall. The right fix for the knowledge gap is RAG, not more training data.

**Why this matters:** My original blog reported the result without explaining the mechanism. This addition makes the architectural reasoning explicit and defensible.

---

## Commit 2 — method.md (conversion-engine)

**File:** `method.md`  
**What to add:** An architecture note on the fine-tune vs RAG split for future iterations.

**Add to the architecture section:**

> TODO (RAG for product knowledge): The current pipeline relies on prompt injection for Tenacious product details (pricing, integrations, case studies). Week 12 gap research confirmed that fine-tuning is the wrong intervention for sparse factual knowledge on small datasets — it risks increasing hallucination confidence without improving factual accuracy. Future iteration should implement RAG over a Tenacious product knowledge base for factual grounding, reserving fine-tuning for tone and style only.

**Why this matters:** This closes the loop between the Week 11 finding (partial transfer) and a concrete next architectural step, grounded in the mechanism now understood.
