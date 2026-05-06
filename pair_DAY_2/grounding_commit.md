# Grounding Commit — Day 2

**Gap closed:** What happens at the token level when a model calls a tool, how the model was trained to produce tool calls, and what causes agent loops.

---

## Commit 1 — agent loop stop_reason handling

**File:** `agent/main_agent.py` (or equivalent agent loop file)  
**What to change:** Add explicit handling for `stop_reason: "max_tokens"` in the agent loop. Currently the loop only checks for `tool_use` and `end_turn`. A response truncated mid-tool-call returns `stop_reason: "max_tokens"` — if this isn't caught explicitly, the loop may silently retry with an incomplete tool call.

**Before (current behaviour):**
```python
while True:
    response = client.messages.create(...)
    if response.stop_reason == "end_turn":
        break
    # tool_use handling...
```

**After (fix):**
```python
while True:
    response = client.messages.create(...)
    if response.stop_reason == "end_turn":
        break
    elif response.stop_reason == "max_tokens":
        raise RuntimeError("Response truncated mid-generation — increase max_tokens or reduce context")
    elif response.stop_reason == "tool_use":
        # tool execution...
```

**Why this matters:** Without this, a truncated tool call becomes a silent infinite retry. This is a latent looping bug in the current pipeline.

---

## Commit 2 — tool execution error handling

**File:** `agent/main_agent.py` (or equivalent)  
**What to change:** Wrap every tool execution in try/catch and always return a `tool_result` block, even on failure, with `is_error: true`. Currently if a tool call throws an exception (HubSpot timeout, Resend rate limit, Cal.com error), the `tool_result` block may not be returned — leaving the model without a result and causing it to retry the same tool call indefinitely.

**Before (current behaviour):**
```python
result = execute_tool(block.name, block.input)
tool_results.append({
    "type": "tool_result",
    "tool_use_id": block.id,
    "content": result
})
```

**After (fix):**
```python
try:
    result = execute_tool(block.name, block.input)
    tool_results.append({
        "type": "tool_result",
        "tool_use_id": block.id,
        "content": result
    })
except Exception as e:
    tool_results.append({
        "type": "tool_result",
        "tool_use_id": block.id,
        "content": str(e),
        "is_error": True
    })
```

**Why this matters:** The model expects a `tool_result` for every `tool_use` block. If it doesn't receive one, it calls the same tool again — indefinitely. This fix ensures every tool execution path, including failures, returns a result the model can reason about.
