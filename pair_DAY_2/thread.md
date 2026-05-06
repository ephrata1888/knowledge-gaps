# Tweet Thread — Day 2

*For Zemzem Hibet's question on tool-use token mechanics*

---

**1/**
Your agent looped for 682 seconds, got terminated by the environment, and returned reward 0.0.

You can't diagnose it because you don't know what the model is actually doing when it "calls" a tool.

Here's the full mechanism — token level 🧵

---

**2/**
First, the most important thing to understand:

The model never calls a tool.

It produces a structured output block. Your framework reads that block and runs the actual function. The model just predicts tokens — a tool call is a specific kind of token sequence it was trained to produce.

---

**3/**
When you send tool schemas to the API, they get serialized into the context window as tokens — usually in the system prompt block.

The model reads your tool descriptions the same way it reads everything else. This is why description quality matters: better descriptions = more accurate tool selection.

---

**4/**
When the model decides to call a tool, the raw API response looks like this:

```json
{
  "type": "tool_use",
  "id": "toolu_01abc",
  "name": "get_company_data",
  "input": { "company": "Acme Corp" },
  "stop_reason": "tool_use"
}
```

The key field: **stop_reason: "tool_use"**

Your loop should be keyed on this — nothing else.

---

**5/**
Three mechanisms are commonly discussed for how this output is produced:

🔹 Special tokens (e.g. `<tool_call>`) — used by some open-source models like Llama/Mistral

🔹 Learned JSON schema — used by Anthropic and OpenAI. Fine-tuning, not token constraints.

🔹 Constrained decoding (logit masking) — used in frameworks like Outlines. NOT how standard APIs work.

---

**6/**
So what caused the 682-second loop?

Three failure modes:

1️⃣ **Repeating the same tool call** — model gets a result it can't interpret as progress, retries forever

2️⃣ **Alternating tools without progress** — planning loop, neither result moves the model forward

3️⃣ **Never emitting a tool call** — model generates text instead of structured output

For a 682-second loop: almost certainly #1 or #2. The agent was actively executing something.

---

**7/**
The most common root cause of #1: your code failed to return a `tool_result` block after a tool error.

The model expects a result. If it doesn't get one, it calls the tool again.

Fix: wrap every tool execution in try/catch and ALWAYS return a tool_result — even on failure — with `is_error: true`.

---

**8/**
How to diagnose which failure mode hit your traces:

1. List all `mcp_tool_use` blocks in order — same tool + same input repeating? → Mode 1
2. Check for alternating tool names with no final text response → Mode 2
3. Count `text` blocks in assistant turns — all text, no tool_use? → Mode 3
4. Check `mcp_tool_result` content — are results returning errors?

---

**9/**
Also check your loop's `stop_reason` handling.

Most looping bugs come down to this:

```python
if response.stop_reason == "end_turn":
    break
elif response.stop_reason == "max_tokens":
    raise RuntimeError("Truncated")  # don't silently retry
elif response.stop_reason == "tool_use":
    # execute tool, return result, continue
```

If `max_tokens` isn't handled explicitly, a truncated tool call silently retries forever.

---

**10/**
TL;DR

- Tool calls = structured JSON the model was fine-tuned to produce, not constrained decoding
- The model never runs the tool — your framework does
- 682-second loops = tool result not returned on error, or planning loop with no state tracking
- Key your agent loop on `stop_reason`, handle all three values explicitly
- Always return `tool_result` even when the tool fails
