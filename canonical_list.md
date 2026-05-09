# Canonical List — Week 12

**Author:** Efrata Wolde  
**Purpose:** Annotated contribution to the cohort canon — papers, tools, and patterns worth other Forward-Deployed Engineers reading.

---

## Papers

### 1. Efficient Memory Management for Large Language Model Serving with PagedAttention
**Authors:** Kwon et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2309.06180  
**Why read it:** The paper that introduced vLLM and explained how KV cache memory is managed in production inference. Essential for understanding why prompt length affects latency, what prefix caching is doing under the hood, and why stateless API calls discard the KV cache. If you are building any LLM pipeline and care about latency or cost, this is the foundational text.  
**Key insight:** KV cache memory fragmentation is the primary bottleneck in LLM serving at scale — PagedAttention solves this the same way virtual memory solved RAM fragmentation in operating systems.

---

### 2. Attention Is All You Need
**Authors:** Vaswani et al.  
**Year:** 2017  
**Link:** https://arxiv.org/abs/1706.03762  
**Why read it:** The foundational Transformer paper. You need to understand why attention scales quadratically with sequence length (O(N²)) to explain why doubling a prompt more than doubles prefill cost. Read the self-attention section specifically — everything about KV caches, prefill cost, and context window limits traces back to this paper.  
**Key insight:** Every token must attend to every other token — this is both the source of the Transformer's power and the origin of its scaling costs.

---

### 3. Gorilla: Large Language Model Connected with Massive APIs
**Authors:** Patil et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2305.15334  
**Why read it:** The foundational paper on training LLMs to call APIs reliably. Explains why tool description quality directly affects call accuracy — the model pattern-matches user intent against serialised tool descriptions, so descriptions that name the condition under which a tool should be called outperform generic ones. Directly relevant to anyone building agent pipelines.  
**Key insight:** Tool descriptions are not documentation for humans — they are training signal for the model. Write them like prompts, not like API docs.

---

### 4. LIMA: Less Is More for Alignment
**Authors:** Zhou et al.  
**Year:** 2023  
**Link:** https://arxiv.org/abs/2305.11206  
**Why read it:** Demonstrates that 1,000 high-quality fine-tuning examples can produce strong stylistic alignment — the model learns *how* to respond from a tiny dataset, because the underlying knowledge was already in the base model weights. Essential context for understanding why tone transfers efficiently with small fine-tuning datasets but factual knowledge doesn't.  
**Key insight:** Fine-tuning teaches style and format far more efficiently than it teaches facts. If you need the model to know things, use RAG. If you need it to sound a certain way, fine-tune.

---

### 5. Judging the Judges: Evaluating Alignment and Vulnerabilities in LLMs-as-Judges
**Authors:** Panickssery et al.  
**Year:** 2024  
**Link:** https://arxiv.org/abs/2406.12334  
**Why read it:** The paper that catalogues the failure modes of LLM judges: position bias, length bias, self-enhancement bias, and prompt-phrasing sensitivity. Essential reading before you publish any benchmark score produced by an LLM judge. Shows that biases are systematic and directional — not random noise — which means they don't average out.  
**Key insight:** An inflated benchmark score and a legitimately earned benchmark score look identical in the score distribution. The only way to tell them apart is to validate the judge against a reference standard.

---

### 6. JudgeSense: A Benchmark for Prompt Sensitivity in LLM-as-a-Judge Systems
**Authors:** Bellibatlu et al.  
**Year:** 2026  
**Link:** https://arxiv.org/abs/2604.23478  
**Why read it:** Introduces the Judge Sensitivity Score (JSS) and establishes 0.85 as the threshold below which prompt-surface variance is dominating score variance. The paper that makes LLM judge validation actionable — you can now run a perturbation sweep and get a number that tells you whether your judge is reliable.  
**Key insight:** JSS < 0.85 means your benchmark is measuring prompt phrasing, not model quality.

---

### 7. ReAct: Synergizing Reasoning and Acting in Language Models
**Authors:** Yao et al.  
**Year:** 2022  
**Link:** https://arxiv.org/abs/2210.03629  
**Why read it:** The paper that formalised the thought-action-observation loop underlying modern LLM agents. If you are building or debugging an agent that uses tools, understanding ReAct is essential context for why the loop structure works and what breaks it. Explains why the model needs to see tool results in context to make progress.  
**Key insight:** Interleaving reasoning traces with tool calls dramatically reduces agent loops compared to pure tool-calling without intermediate reasoning steps.

---

## Tools

### Anthropic Prompt Caching
**Link:** https://docs.anthropic.com/en/docs/build-with-claude/prompt-caching  
**Why use it:** Reduces TTFT and cost for pipelines that send a static system prompt on every call. Requires the static prefix to be byte-for-byte identical across calls. One-line implementation change with meaningful impact at scale.

### vLLM
**Link:** https://github.com/vllm-project/vllm  
**Why use it:** Open-source LLM serving engine that implements PagedAttention and prefix caching. If you are self-hosting models, vLLM is the production-grade serving layer. Also useful for understanding how KV cache management works in practice even if you're using a hosted API.

### Outlines
**Link:** https://github.com/outlines-dev/outlines  
**Why use it:** Structured generation via constrained decoding — forces model output to conform to a JSON schema at the logit level. Useful when you need guaranteed valid structured output and fine-tuning isn't an option. Important to understand how this differs from the fine-tuned structured output in standard Anthropic/OpenAI tool calling.

---

## Patterns

### Pattern 1 — Separate static from dynamic context
For any pipeline that makes repeated LLM calls with a shared system prompt, keep the static portion byte-for-byte identical across calls. This enables prefix caching and eliminates redundant prefill computation. Structure: `[static system prompt] + [dynamic per-call context]`. The static portion gets cached; the dynamic portion gets freshly prefilled each time.

### Pattern 2 — Key your agent loop on stop_reason
Agent loops should check `stop_reason` explicitly for all values: `end_turn` (done), `tool_use` (execute tool and continue), `max_tokens` (handle truncation — do not silently retry). Every tool execution path must return a `tool_result` block, even on failure, with `is_error: true`. Without this, a failed tool call causes the model to retry indefinitely.

### Pattern 3 — Validate evaluator inputs at load time
Deterministic evaluators should validate all task dimension names against a known schema before any scoring runs. Replace silent defaults (`return 1.0`) with explicit errors. In evaluation code, a loud failure is always better than a silent one — silent failures produce corrupted measurements that look like correct results.

### Pattern 4 — Fine-tune for style, RAG for facts
Small fine-tuning datasets transfer stylistic patterns efficiently (tone, format, energy level) but transfer factual knowledge poorly. For domain-specific product knowledge, use RAG to retrieve and inject facts at inference time rather than trying to bake them into weights. Combine both for production deployments that need both style and accuracy.

### Pattern 5 — Validate your judge before publishing scores
Before interpreting any LLM judge score as a measurement: (1) run at temperature = 0 for determinism, (2) compute JSS across prompt rephrasings — target ≥ 0.85, (3) compute QWK against a human-annotated reference set — target ≥ 0.80. A judge that hasn't passed these checks is a hypothesis, not an instrument.
