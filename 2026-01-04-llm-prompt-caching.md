# LLM Prompt Caching: How It Works and Why It Matters

*Learned: 2026-01-04*

## Context

Comparing rate limits across LLM providers (OpenAI, Anthropic, Google) for a research synthesis feature. Discovered that Anthropic's cached tokens don't count toward rate limits, which is a significant advantage.

## Insight

### Technical Mechanism

When processing a prompt, LLMs generate key-value pairs for tokens (used in self-attention). Prompt caching stores these K-V pairs for static parts of the prompt. On subsequent requests with the same prefix, the model retrieves stored K-V pairs instead of recomputing them - only new, dynamic parts need processing.

### Provider Comparison

| Provider | Control | Min. tokens | TTL | Savings |
|----------|---------|-------------|-----|---------|
| **OpenAI** | Automatic | 1,024 | 5-10 min (24h for GPT-5.1) | 50% cheaper |
| **Anthropic** | Manual (`cache_control`) | no min | 5 min | 90% cheaper (but +25% for writes) |
| **Gemini** | Manual (`CachedContent.create`) | 32,768 | 1h | Charged for storage |

### Rate Limit Bonus (Anthropic)

`cache_read_input_tokens` **don't count** toward ITPM on most Claude models. With 80% cache hit rate and 2M ITPM limit, you can effectively process 10M tokens/min (2M uncached + 8M cached).

### Best Practices

- Static content (system prompt, instructions) at the **top** of the prompt
- Dynamic content (user input) at the **bottom**
- Break-even: 3-5 requests within cache lifetime
- Don't cache: unique prompts, small prompts, infrequent requests

## Sources

- [Prompt caching: 10x cheaper LLM tokens | ngrok](https://ngrok.com/blog/prompt-caching)
- [How prompt caching works | sankalp's blog](https://sankalp.bearblog.dev/how-prompt-caching-works/)
- [Prompt Caching Guide 2025 | promptbuilder](https://promptbuilder.cc/blog/prompt-caching-token-economics-2025)
