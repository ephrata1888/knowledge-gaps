# Morning Call Summary — Day 2

**Participants:** Efrata Wolde · Zemzem Hibet  
**Duration:** ~20 minutes  
**Topic:** Agent and tool-use internals

---

## What We Each Brought In

**Efrata's draft question** was about what happens at the token level when a model decides to call a tool vs respond in plain text, grounded in the conversion-engine passing HubSpot and Resend schemas to the model.

**Zemzem's draft question** was grounded in two specific traces (`19d13ac9` and `293b3bbb`) from her Week 10 lead gen agent showing a 682–687 second loop terminated by `user_stop` with `reward 0.0`. She wanted to understand the token-level mechanism in order to diagnose whether the loop was caused by repeated tool calls, alternating tools, or a failure to emit a tool call at all.

---

## How We Sharpened Each Other

On Efrata's question: Zemzem pushed to make it more diagnostic — "what determines whether the model calls a tool vs responds in plain text" was good but needed to include the failure case. What breaks this mechanism and causes bad behaviour? The question was tightened to include the training mechanism and the failure mode.

On Zemzem's question: Efrata noted that her three hypotheses (special tokens, learned JSON schema, constrained decoding) were the right framing and should stay in the question explicitly — it makes the explainer writer's job clear and the answer more useful to the cohort.

---

## Final Agreed Questions

**Efrata:** "My conversion-engine passes tool schemas to an LLM which then decides to call tools like HubSpot and Resend. What actually happens at the token level when a model decides to make a tool call — what does the raw output look like, how was the model trained to produce it, and what determines whether the model calls a tool vs responds in plain text?"

**Zemzem:** "In my Week 10 lead gen agent, traces 19d13ac9 and 293b3bbb on task 92 show the agent looping for 682–687 seconds before hitting user_stop with reward 0.0. I cannot diagnose what happened inside that loop because I do not understand what is happening at the token level when a model 'chooses' to call a tool versus generate text. Specifically: is the model outputting special tokens, a learned JSON schema, or something the API intercepts via constrained decoding? And which of those mechanisms, if broken or degraded, would produce a 682-second loop that the environment terminates rather than the agent?"
