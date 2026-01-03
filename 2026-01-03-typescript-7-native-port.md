# TypeScript 7: The Native Port (10x Faster Compilation)

*Learned: 2026-01-03*

## Context

Working on IntexuraOS monorepo with strict TypeScript settings, implementing research enhancement features and fixing model selection bugs. The project has 24+ workspaces and uses parallel typecheck scripts.

## Insight

### 1. Complete Rewrite in Native Code

TypeScript 7 ("Project Corsa") is a full rewrite of the compiler from JavaScript to native code. This isn't just optimization—it's a ground-up rebuild to leverage parallelism and better memory management. Real-world results show ~10x speedups (Sentry: 133s → 16s).

### 2. Breaking Changes to Know About

TypeScript 6.0 will be the final JS-based release. Version 7 makes `--strict` the default, removes `--target es5`, eliminates `--baseUrl`, and replaces `node10` module resolution. If your codebase still uses these, start planning migration now.

### 3. Why This Matters for Your Codebase

The IntexuraOS monorepo already uses strict TypeScript settings (`noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`), so you're well-positioned for TS7. The parallel type-checking in the native port will significantly speed up your `npm run typecheck` across 24+ workspaces.

### 4. Timeline

VS Code native preview is already available with most editor features. Full release expected in 2025-2026. Worth monitoring if you're experiencing slow type-checking in large monorepos.

## Sources

- [Progress on TypeScript 7 - December 2025](https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/)
- [Announcing TypeScript 5.7](https://devblogs.microsoft.com/typescript/announcing-typescript-5-7/)
- [Effective TypeScript - 2025 Year in Review](https://effectivetypescript.com/2025/12/19/ts-2025/)
