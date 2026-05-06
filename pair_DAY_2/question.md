# Question — Day 2

**Gap owner:** Efrata Wolde  
**Topic:** Agent and tool-use internals  
**Status:** Final (post morning call)

---

## Question

My conversion-engine passes tool schemas to an LLM which then decides to call tools like HubSpot and Resend. What actually happens at the token level when a model decides to make a tool call — what does the raw output look like, how was the model trained to produce it, and what determines whether the model calls a tool vs responds in plain text?

---

## Grounding

This question is tied directly to my conversion-engine repo. My pipeline defines tools for HubSpot, Resend, and Cal.com and passes their schemas to the model on every call. I have been building and shipping this agent without understanding what actually happens when the model "decides" to call one of those tools. I could describe the outcome but not the mechanism — and without the mechanism I cannot diagnose failures like loops, skipped tool calls, or malformed tool outputs.

---

## Why This Is a Real Gap

Nowhere in my Week 10 or 11 work do I explain the tool-calling mechanism at the token level. I used phrases like "the agent calls HubSpot" without understanding that the model never calls anything — it produces a structured output block that the framework intercepts and executes. This gap was confirmed on the morning call with Zemzem, whose own traces showed a 682-second loop she couldn't diagnose for the same reason.
