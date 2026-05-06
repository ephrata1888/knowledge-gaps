# Sources — Day 2

*Canonical sources cited in Zemzem Hibet's explainer for Efrata's question*

---

## Source 1

**Title:** Anthropic Tool Use Documentation — Implement Tool Use  
**Link:** https://platform.claude.com/docs/en/agents-and-tools/tool-use/implement-tool-use  
**Why it matters:** Canonical documentation for the `tool_use` content block structure, `stop_reason` values, the tool result contract, and the `tool_choice` parameter. The authoritative reference for how Anthropic's API implements tool calling at the API level.

---

## Source 2

**Title:** Gorilla: Large Language Model Connected with Massive APIs  
**Authors:** Patil et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2305.15334  
**Why it matters:** The foundational paper on training LLMs to call APIs reliably. Explains why tool description quality directly affects call accuracy — the model pattern-matches user intent against serialized tool descriptions, so descriptions that name the condition under which a tool should be called outperform generic descriptions.

---

## Tool Used

**Name:** Anthropic Python SDK — `client.messages.create` with `tools` parameter  
**Used for:** Verifying `stop_reason` behaviour and `tool_use` block structure by running the canonical loop pattern against a test tool to confirm the raw API response format described in the explainer.
