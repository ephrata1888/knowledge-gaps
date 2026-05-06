# Signoff — Day 2

**Gap owner:** Efrata Wolde  
**Explainer author:** Zemzem Hibet  
**Status: CLOSED ✅**

---

## What I Now Understand

Before today I was using phrases like "the agent calls HubSpot" as if the model itself executed the function. I had no understanding of what actually happens at the token level. I couldn't have told you what a tool call looks like in the raw API response, what field my loop should be keyed on, or what causes an agent to loop indefinitely.

Zemzem's explainer closed all three parts of my question clearly.

On the raw output: when the model decides to call a tool it emits a structured `tool_use` content block with `type`, `id`, `name`, and `input` fields. The critical signal is `stop_reason: "tool_use"` — this is what my agent loop should check to decide whether to execute a tool or render a final response. The model never executes anything. My framework does.

On how the model was trained to produce it: this is fine-tuning, not constrained decoding. The model was trained on large quantities of tool-use examples until producing valid structured JSON became the highest-probability output when context matches a tool call pattern. Constrained decoding (logit masking) exists for structured outputs but is not how standard Anthropic tool calling works. This corrects an assumption I had been carrying without realising it.

On what determines tool call vs plain text: tool description quality and the `tool_choice` parameter. The model pattern-matches user intent against serialized tool descriptions in the context window — better descriptions that name the condition under which the tool should be called produce more accurate routing. For production steps where tool calls are mandatory, `tool_choice: {type: "any"}` is more reliable than prompt instructions alone.

The loop failure section was the part I didn't know I needed. My conversion-engine loop does not currently handle `stop_reason: "max_tokens"` explicitly, and not every tool execution path returns a `tool_result` on failure. Both are latent looping bugs in my live pipeline.

---

## One Thing That Could Be Sharper

The planning loop failure mode (Failure Mode 3 in Zemzem's explainer) was described well in theory but the fix — adding a step-tracking field to the system prompt — could have included a concrete example of what that looks like in practice. A one-line example like `"Already completed: [get_company_info for sarah@tenacious.com]"` would have made it immediately actionable.

---

## Grounding Commit

See `day2_grounding_commit.md` — two fixes to the conversion-engine agent loop based on what I learned today.
