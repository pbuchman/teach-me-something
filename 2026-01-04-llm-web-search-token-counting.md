# Web Search Token Counting: The Hidden Cost Variance Across LLM Providers

*Learned: 2026-01-04*

## Context

Building a multi-provider LLM research system that queries Claude, Gemini, OpenAI, and Perplexity. Noticed massive token count discrepancies (480 vs 89,080 tokens) for identical queries and investigated how each provider counts web search tokens differently.

## Insight

### The Token Count Disparity

For the same weather research query, providers reported vastly different input tokens:

| Provider   | Input Tokens | Notes                                |
|------------|--------------|--------------------------------------|
| Gemini     | 480          | Only counts the prompt               |
| OpenAI     | 37,739       | Includes scraped web content         |
| Claude     | 89,080       | Includes ALL search results in context |

### How Each Provider Handles Web Search Tokens

**Claude (Anthropic)**
- Web search results return as `tool_result` blocks
- Fully counted as input tokens
- Additional $0.01 per search call

**Gemini (Google)**
- Search grounding is "hidden" from token counts
- `promptTokenCount` only reflects the original prompt
- Flat fee per request ($35/1k requests for Gemini 2.5)

**OpenAI**
- Similar to Claude - scraped pages become context
- 50-120x token inflation reported by users
- Additional $0.01 per search call

**Perplexity**
- Returns a ready-made cost object in the response
- Includes: `input_tokens_cost`, `output_tokens_cost`, `request_cost`, and `total_cost`
- No manual calculation needed

### Real Cost Formulas

```typescript
// Claude
const claudeCost =
  (input - cacheRead) * inputPrice +
  cacheRead * 0.1 * inputPrice +
  cacheCreation * 1.25 * inputPrice +
  output * outputPrice +
  searches * 0.01;

// Gemini
const geminiCost =
  promptTokens * inputPrice +
  candidateTokens * outputPrice +
  (groundingEnabled ? 0.035 : 0);

// OpenAI
const openaiCost =
  (input - cached) * inputPrice +
  cached * 0.25 * inputPrice +
  output * outputPrice +
  searches * 0.01;

// Perplexity (already calculated!)
const perplexityCost = response.usage.cost.total_cost;
```

### Key Takeaway

A simple `inputPricePerMillion` / `outputPricePerMillion` pricing model is **insufficient** for accurate cost tracking when web search is involved. Each provider has unique pricing mechanics that must be handled separately.

## Sources

- [Anthropic Web Search Pricing](https://websearchapi.ai/blog/anthropic-claude-web-search-api)
- [OpenAI Community: Token Inflation](https://community.openai.com/t/massive-input-token-inflation-50x-with-o3-web-search/1317167)
- [Google Gemini Grounding](https://ai.google.dev/gemini-api/docs/google-search)
- [Perplexity Pricing](https://docs.perplexity.ai/getting-started/pricing)
