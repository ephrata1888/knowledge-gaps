# Tweet Thread — Day 3

*For own question on prompt vs fine-tuning — what changes at the weight level*

---

**1/**
I argued in my Week 11 blog that fine-tuning was necessary because prompt engineering couldn't fully capture Tenacious's sales tone.

I was right. But I couldn't explain why.

Here's the actual mechanism — what a prompt does vs what fine-tuning does at the weight level 🧵

---

**2/**
A prompt adds tokens to the context window.

That's it.

The model reads your instructions the same way it reads everything else — as input that influences which output tokens are most probable given everything it has already learned.

Prompts work within existing weights. They do not change them.

---

**3/**
This means a prompt can only activate behaviours the model already knows how to do.

It cannot install new behaviours.
It cannot change what the model has learned to associate with certain patterns.

No matter how well you write it, a prompt cannot cross this ceiling.

---

**4/**
Fine-tuning is different.

It runs gradient descent on your training data. The loss function measures how wrong the model's predictions are on your examples. The optimizer adjusts the weights to make them less wrong.

After fine-tuning, the model's weights have permanently shifted toward your data.

---

**5/**
The key distinction:

🔹 A prompt changes what the model **attends to**
🔹 Fine-tuning changes what the model has **learned**

One is a temporary contextual adjustment.
The other is a permanent modification to the model's internal representations.

---

**6/**
This explains why my fine-tuned model transferred tone better than product knowledge.

Tone is a distributed stylistic signal — it shows up in every sentence of every training example. The model updates toward it efficiently because the signal is everywhere.

---

**7/**
Product knowledge is specific and sparse.

"Tenacious's pricing starts at $X" appears in only a handful of examples. Fine-tuning on a small dataset doesn't give the model enough signal to reliably store and retrieve specific facts.

The model picks up the *style* of confident factual claims without learning which facts are actually true.

---

**8/**
This is also why fine-tuning on small domain-specific datasets often makes hallucination *worse* for factual queries.

More confidence. Less accuracy. The model learned to sound like an expert without becoming one.

---

**9/**
Practical implication for production:

🔹 **Tone** → fine-tune. Prompting approximates it but drifts.
🔹 **Product knowledge** → RAG. Retrieve facts at inference time, inject into prompt. Don't try to bake specifics into weights with small data.

The right architecture is often both — fine-tune for style, RAG for facts.

---

**10/**
TL;DR

- Prompts activate existing behaviours — they cannot install new ones
- Fine-tuning modifies weights permanently via gradient descent
- Tone transfers well because it's a distributed signal across all training examples
- Facts transfer poorly because they're sparse and require precise recall
- Prompting ceiling = can't change weights. Fine-tuning crosses it.
