# Conditional Object Spreading: The Omit-vs-Default Pattern

*Learned: 2026-01-07*

## Context

Fixing failing tests in IntexuraOS where test expectations didn't match code behavior. Tests expected `.toBe(0)` for cache tokens, but the `normalizeUsage` function used conditional spreading to omit zero-value properties entirely.

## Insight

When building API responses or data objects, there's an important design choice: should optional fields be omitted when empty, or always included with default values?

### The Pattern

```typescript
return {
  inputTokens,
  outputTokens,
  ...(cachedTokens > 0 && { cacheTokens: cachedTokens }),
  ...(webSearchCalls > 0 && { webSearchCalls }),
};
```

### Trade-offs

1. **Omit when empty** (what the code does): Cleaner JSON responses, smaller payloads, but consumers must handle undefined. Tests must use `.toBeUndefined()` not `.toBe(0)`.

2. **Always include defaults**: Predictable shape, easier type checking, but verbose responses. Tests use `.toBe(0)`.

### The Gotcha

The test assumed "always include" (`expect(cacheTokens).toBe(0)`) but the code used "omit when empty". This mismatch is common when:

- Tests are written before implementation
- Code evolves without updating tests
- Different team members have different assumptions

### Best Practice

Document the pattern in your types. TypeScript's `?` operator signals "may be omitted":

```typescript
interface Usage {
  inputTokens: number;      // always present
  cacheTokens?: number;     // omitted when 0
}
```

## Sources

None - learned from codebase context
