# 💰 Claude API Cost Optimizer

[![ClawHub Skill](https://img.shields.io/badge/ClawHub-Skill-blue)](https://clawhub.ai/djc00p/claude-api-cost-optimizer) [![Agent Skill](https://img.shields.io/badge/Agent-Skill-blue)](#) [![Anthropic](https://img.shields.io/badge/Anthropic-Claude-D4A574.svg)](https://www.anthropic.com) [![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)

**Claude API Cost Optimizer** cuts Anthropic API spend by **70–90%** using four levers: intelligent model selection, prompt caching, request batching, and cost tracking.

Most teams overpay without knowing it. The default answer to "which model?" is the most capable one, applied to everything — including the formatting task that would run fine on the cheapest tier. This skill audits where the money actually goes and routes each task to the cheapest model that can reliably do it.

---

## ⚠️ Local Data Warning

This skill creates `~/.claude_cache/`, `costs.log`, and `batch_results.jsonl` which may contain **source code, API responses, and prompts**.

* Set `chmod 600` on these files
* **Never cache secrets or PII**
* Add cache directories to `.gitignore`

---

## 🎯 Lever 1 — Model Selection

The biggest single win. Model selection alone saves **10–18x** on simple tasks.

| Tier | Use For | Notes |
|------|---------|-------|
| **Haiku** | Formatting, comments, explanations, renames | Cheapest tier — default for anything mechanical |
| **Sonnet** | Refactoring, debugging, feature work | The mid tier — most real work lands here |
| **Opus** | Architecture, algorithms, security audits | Most expensive — use sparingly, and only where it genuinely changes the outcome |

```python
def select_model(task_type: str) -> str:
    simple_tasks = ["formatting", "comments", "explanation", "rename"]
    complex_tasks = ["architecture", "algorithm", "security_audit"]
    return ("claude-haiku-4-5-20251001" if task_type in simple_tasks else
            "claude-opus-4-6" if task_type in complex_tasks else
            "claude-sonnet-4-6")
```

---

## ⚡ Lever 2 — Prompt Caching

Cache large static content — system prompts, codebase context. **Cache reads cost 90% less**, and writes pay off after just 1–2 reuses.

```python
response = client.messages.create(
    model="claude-sonnet-4-6",
    max_tokens=1024,
    system=[{
        "type": "text",
        "text": system_prompt,
        "cache_control": {"type": "ephemeral"}
    }],
    messages=[{
        "role": "user",
        "content": [
            {"type": "text", "text": f"Code:\n{source_code}",
             "cache_control": {"type": "ephemeral"}},
            {"type": "text", "text": query}
        ]
    }]
)
```

Cache the **static** prefix, not the varying suffix — put the cache breakpoint *before* the part that changes.

---

## 📦 Lever 3 — Batching

Combine multiple requests into one API call to eliminate per-request overhead. **80% fewer calls ≈ 80% lower cost.**

---

## 🗄️ Lever 4 — Local Caching

Cache identical responses locally to skip redundant API calls entirely. Pair it with a short TTL (`max_age_hours`) and clear the cache regularly.

---

## ✂️ Two More Free Wins

* **Context extraction** — send only relevant snippets, not whole files. Smaller inputs, lower cost.
* **`max_tokens` discipline** — set realistic limits. Unused token budget is wasted money.

---

## 📊 Cost Tracking

Track every request so the routing decisions are based on data, not guesses.

```python
usage = response.usage
cost = (usage.input_tokens * INPUT_RATE +
        usage.cache_creation_input_tokens * CACHE_WRITE_RATE +
        usage.cache_read_input_tokens * CACHE_READ_RATE +
        usage.output_tokens * OUTPUT_RATE)
```

---

## 🛡️ Security Checklist

Before enabling cost tracking and local caching:

- [ ] Set `chmod 600 ~/.claude_cache/` and all log files
- [ ] **Never cache** content containing secrets, credentials, or PII
- [ ] Set a short TTL (`max_age_hours`) and clear cache regularly
- [ ] Add cache directories to `.gitignore`
- [ ] Use encrypted volumes or ephemeral storage in CI environments
- [ ] Audit cached content periodically for sensitive data leaks

---

## 🚀 Quick Start

1. **Audit current API calls** — find tasks using Opus/Sonnet that could use Haiku
2. **Pick the cheapest tier per task** — Haiku → Sonnet → Opus
3. **Enable prompt caching** for repeated context via `"cache_control": {"type": "ephemeral"}`
4. **Implement cost reporting** — track `input_tokens`, `output_tokens`, and cache metrics

Requires `ANTHROPIC_API_KEY` in the environment.

---

## 📚 References

* `references/implementation.md` — full implementation patterns: model routing, caching setup, batching, retry logic, anti-patterns
* `references/pricing.md` — current pricing, cache break-even math, savings calculations, batch API details

---

**Published by [@djc00p](https://github.com/djc00p)**
