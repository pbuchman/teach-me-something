# Authentication Middleware Short-Circuit Testing

*Learned: 2026-01-09*

## Context

Adding 401 authentication tests to bookmarks-agent endpoints (PATCH, DELETE, archive, unarchive) for code coverage improvement in IntexuraOS.

## Insight

When testing authentication in Fastify/Express routes, there's a subtle but important distinction between testing the middleware itself vs. testing endpoints protected by it.

### The Pattern

In the bookmarks-agent, `requireAuth()` returns `null` when auth fails and the middleware sends the 401 response directly. The route handler checks `if (!user) return;` — this is a short-circuit pattern where the early return is technically "unreachable" from the function's perspective because the response was already sent.

### Why Coverage Matters Here

These 401 tests ensure the middleware integration works correctly at the route level. Without them, you'd only know the middleware works in isolation, not that it's properly wired into each endpoint.

### The Testing Trade-off

You have two choices:
- Test each endpoint's 401 path — verbose but catches wiring bugs
- Trust middleware tests + one integration test — DRY but misses per-route issues

### The Hidden Bug Risk

The reason per-endpoint 401 tests are valuable: developers occasionally add new routes and forget to add `requireAuth()`, or add it in the wrong position. These tests catch that immediately.

## Sources

From codebase context (IntexuraOS bookmarks-agent)
