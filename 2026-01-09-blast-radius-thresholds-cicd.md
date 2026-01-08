# Blast Radius Thresholds in CI/CD

*Learned: 2026-01-09*

## Context

Exploring Cloud Build configuration and smart-dispatch system in a monorepo to update documentation for creating new services.

## Insight

The IntexuraOS monorepo uses a "blast radius" pattern for deployment decisions:

### 1. The Threshold Concept

Instead of binary "deploy all" or "deploy changed", smart-dispatch uses a threshold (3 services). This creates three deployment modes:

- **0 affected** → Skip deploy entirely
- **1-3 affected** → Deploy individually in parallel (fast, targeted)
- **>3 affected** → Deploy monolith (simpler orchestration, single build cache)

### 2. Why Not Always Individual?

When many services change, individual deploys create:

- N separate Docker cache pulls (network overhead)
- N Cloud Build trigger invocations (API rate limits)
- Complex failure handling (partial deploys are hard to reason about)

### 3. Transitive Dependency Expansion

The system builds a package dependency graph and expands transitively. If you change `common-core` which `infra-firestore` depends on, and `user-service` depends on `infra-firestore`, all three are marked affected — even though only one file changed.

### 4. Global Triggers as Circuit Breaker

Certain paths (`terraform/`, `package-lock.json`) force MONOLITH regardless of threshold. This prevents subtle misconfigurations where infrastructure changes without corresponding service updates.

---

This pattern balances deployment speed (individual) against operational simplicity (monolith) based on change scope.
