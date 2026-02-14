# CSS Stacking Context — The z-index Nobody Understands

*Learned: 2026-02-14*

## Context

Building a LinearIssueSelectorModal component with `z-50` fixed positioning in a React/Tailwind app. Needed to understand why z-index values don't always behave as expected.

## Insight

**1. z-index doesn't work globally.** Setting `z-index: 9999` doesn't mean "put this on top of everything." It means "put this on top of everything *within its stacking context*." A stacking context is like a namespace — elements inside it compete with each other, but the whole group competes as one unit with elements outside.

**2. What creates a stacking context?** Several CSS properties silently create one:
- `position: relative/absolute/fixed` + any `z-index` value
- `opacity` less than 1 (e.g., `opacity: 0.99`)
- `transform` (even `transform: translateZ(0)`)
- `filter`, `backdrop-filter`
- `isolation: isolate` (explicitly creates one)

This is why you sometimes see `z-index: 99999` and the element is *still* behind something with `z-index: 1` — the parent created a stacking context, trapping the child.

**3. Why `z-50` works for modals.** Using `fixed inset-0 z-50` places the element relative to the viewport. As long as it's rendered at the top level of the component tree (not inside a `transform`-ed parent), no stacking context traps it.

**Backend analogy:** Think of stacking contexts like Docker networks. A container with the highest priority inside network A still can't reach above a container in network B. The `z-index` is the priority *within* the network, but the network itself has its own layer in the stack.
