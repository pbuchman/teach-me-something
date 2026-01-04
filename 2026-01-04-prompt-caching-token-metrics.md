# Prompt Caching in Claude API — Token Metrics Anatomy

*Learned: 2026-01-04*

## Context

Customizing Claude Code statusline to show token metrics (total_input, total_output, cache_read). Noticed that total_input grew slowly despite large context size, which led to investigating how prompt caching actually works.

## Insight

### Four Key Metrics Per API Call

| Metric | Scope | What It Measures |
|--------|-------|------------------|
| `input_tokens` | per-call | Tokens processed fresh (full price) |
| `cache_read_input_tokens` | per-call | Tokens read from cache (~10% price) |
| `cache_creation_input_tokens` | per-call | Tokens written to cache (first turn) |
| `total_input_tokens` | cumulative | Sum of all input_tokens in session |
| `total_output_tokens` | cumulative | Sum of all output_tokens in session |

### Why Does total_input Grow Slowly Despite Large Context?

Real session example:

```
Context: 53k tokens
Cache read: 53k tokens
total_input increase: only +1k
```

This means 53k tokens of context were already cached (system prompt, CLAUDE.md, conversation history). Only ~1k new tokens (user's new message) were processed at full price.

### How Cache Works in Practice

```
Turn 1: [system_prompt + docs] → cache_creation: 50k, input: 1k
Turn 2: [cached_context + new_message] → cache_read: 50k, input: 2k
Turn 3: [cached_context + more_history] → cache_read: 52k, input: 1k
```

Cache acts as a "sliding window" — previous turns are added to cache, so each subsequent message reuses more context.

### Cost Implications

- Cache read: ~$0.30 / 1M tokens (Sonnet)
- Fresh input: ~$3.00 / 1M tokens (Sonnet)

With 53k context in cache, you save ~90% on input costs per turn!

## Sources

- Direct experimentation with Claude Code statusline metrics
