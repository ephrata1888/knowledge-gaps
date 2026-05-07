# Question — Day 3

**Gap owner:** Efrata Wolde  
**Topic:** Training and post-training mechanics  
**Status:** Final (solo — partner unreachable)

---

## Question

In my Week 11 blog I argued that fine-tuning was necessary because prompt engineering alone couldn't fully capture Tenacious's sales tone. What is the actual difference between what a prompt does and what fine-tuning does — at the weight level, what does fine-tuning change that a prompt cannot change, and why does that distinction explain why tone transfer requires training rather than better instructions?

---

## Grounding

This question is tied to a specific architectural decision in my Week 11 work. I recommended fine-tuning over continued prompt engineering for tone transfer and reported partial transfer results. The recommendation was correct but I made it on intuition — I could not explain at the weight level why a prompt has a ceiling or why fine-tuning crosses it. This gap sits underneath every claim I made about why the ORPO approach was necessary.
