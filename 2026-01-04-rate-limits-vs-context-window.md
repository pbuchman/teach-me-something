# LLM API Rate Limits vs Context Window: The Hidden Distinction

*Learned: 2026-01-04*

## Context

Comparing LLM providers for a research synthesis feature. Hit rate limit errors that revealed the difference between context window (max tokens per request) and rate limits (tokens per minute across all requests).

## Insight

### Two Different Limits, Often Confused

| Concept | Meaning | Example |
|---------|---------|---------|
| **Context Window** | Max tokens in ONE request (input + output) | Gemini: 1M, GPT-5.2: 400K |
| **Rate Limit (TPM)** | Max tokens per MINUTE (all requests) | Gemini Flash Tier 1: 30K/min |

You can have a 1M context window, but a 30K TPM limit means one large request blocks the API for ~33 minutes.

### Tier System - Key to Scalability

All providers use a tier system based on cumulative spend:

| Provider | Tier 1 → Tier 2 | Tier 2 → Tier 3 |
|----------|-----------------|-----------------|
| Anthropic | $40 | $200 |
| OpenAI | $50 | $100 |
| Google | Cumulative GCP spend | Cumulative GCP spend |

### Rate Limit as Hidden Cost

When choosing a model for synthesis, rate limit can be more important than price:
- Gemini Pro: cheapest ($1.25/1M), but Tier 1 has ~30-100K TPM
- GPT-5.2: pricier ($1.75/1M), but Tier 1 has ~500K TPM
- Claude: middle ($3/1M), but cached tokens don't count toward ITPM

### Strategies to Work Around Low Limits

1. **Batch API** - 50% cheaper, async processing, higher limits
2. **Prompt caching** - especially Anthropic (cached tokens = free ITPM)
3. **Model fallback** - use pricier model when cheaper one is throttled
4. **Request queuing** - spread over time with exponential backoff

## Sources

- [Gemini API rate limits](https://ai.google.dev/gemini-api/docs/rate-limits)
- [Claude Rate limits](https://platform.claude.com/docs/en/api/rate-limits)
- [OpenAI Rate limits](https://platform.openai.com/docs/guides/rate-limits)
