# Signoff — Day 3

**Gap owner:** Efrata Wolde  
**Explainer author:** Efrata Wolde (solo — partner unreachable)  
**Status: SELF-ASSESSED CLOSED ✅**

---

## What I Now Understand

Before today I had made a correct architectural recommendation — fine-tuning over prompting for tone transfer — without being able to defend it. If someone had asked "why can't you just write a better prompt?" I would have had no answer.

I can now answer that question precisely.

A prompt adds tokens to the context window and activates behaviours the model already knows how to perform. It cannot modify the model's weights and therefore cannot install new behaviours or change what the model has learned to associate with certain patterns. No matter how well-written, a prompt works within existing weight space — it cannot cross that ceiling.

Fine-tuning runs gradient descent on training data and permanently modifies the model's weights. After fine-tuning on Tenacious sales transcripts, the model's internal representations for B2B sales contexts have shifted toward Tenacious-style outputs. This is not contextual — it persists across all inference calls without needing prompt instructions to reinforce it.

The partial transfer result I reported in Week 11 — tone transferred, product knowledge didn't — now makes complete sense. Tone is a distributed stylistic signal present in every sentence of every training example. The model updates toward it efficiently. Product knowledge is sparse and specific — appearing in only a handful of examples — and a small fine-tuning dataset doesn't provide enough signal for reliable factual recall. The model learned to sound confident without learning which specific facts are true.

The practical consequence for my pipeline: RAG is the right intervention for product knowledge, not more fine-tuning data. Fine-tuning for facts on small datasets risks making hallucination worse, not better.

---

## Grounding Commit

See `day3_grounding_commit.md` — one addition to the Week 11 blog and one architecture note in the conversion-engine method.md.
