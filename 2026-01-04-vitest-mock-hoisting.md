# Vitest/Jest vi.mock() Hoisting and Variable Access

*Learned: 2026-01-04*

## Context

Working on test files that mock audit logging for image generation, encountered ReferenceError when trying to access mock variables.

## Insight

The `vi.mock()` (and Jest's `jest.mock()`) calls are **hoisted** to the top of the file during transformation. This means they execute BEFORE any variable declarations, even if you write them after.

### The Problem

```typescript
const mockUsage = vi.fn(); // Not defined yet when vi.mock runs!

vi.mock('@intexuraos/llm-audit', () => ({
  createAuditContext: () => ({ success: mockUsage }) // ReferenceError!
}));
```

### Solution 1: Use `vi.hoisted()` (Vitest 0.31+)

```typescript
const { mockUsage } = vi.hoisted(() => ({
  mockUsage: vi.fn()
}));

vi.mock('@intexuraos/llm-audit', () => ({
  createAuditContext: () => ({ success: mockUsage }) // Works!
}));
```

### Solution 2: Define mock inside factory

```typescript
vi.mock('@intexuraos/llm-audit', () => {
  const mockUsage = vi.fn();
  return {
    createAuditContext: () => ({ success: mockUsage }),
    __mockUsage: mockUsage // Export for test access
  };
});

// Access in tests:
const { __mockUsage } = await import('@intexuraos/llm-audit');
```

### Solution 3: Use `vi.mocked()` with `mockImplementation`

```typescript
vi.mock('@intexuraos/llm-audit');

import { createAuditContext } from '@intexuraos/llm-audit';

beforeEach(() => {
  vi.mocked(createAuditContext).mockReturnValue({
    success: vi.fn(),
    error: vi.fn()
  });
});
```

### Key Takeaway

`vi.hoisted()` is the cleanest solution when you need to share mock references between the mock factory and your tests.

## Sources

- None (from practical experience)
