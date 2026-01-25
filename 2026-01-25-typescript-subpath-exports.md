# TypeScript Subpath Exports: The Modern Package API

*Learned: 2026-01-25*

## Context

Designing a shared `@intexuraos/internal-clients` package with subpath exports for consolidating duplicate HTTP clients across a monorepo.

## Insight

**1. The Dual Export Pattern**

For packages consumed both internally (monorepo) and potentially externally, you need both `types` and `import` conditions:

```json
{
  "exports": {
    "./user-service": {
      "types": "./dist/user-service/index.d.ts",
      "import": "./dist/user-service/index.js"
    }
  }
}
```

**Order matters!** TypeScript resolves `types` first, then Node.js resolves `import`. Putting `types` after `import` causes resolution failures in some tooling.

**2. The `moduleResolution: "bundler"` vs `"nodenext"` Trade-off**

| Setting | Behavior | Use When |
|---------|----------|----------|
| `bundler` | Relaxed — allows extensionless imports | Using Vite, esbuild, webpack |
| `nodenext` | Strict — requires `.js` extensions | Pure Node.js, maximum compatibility |

**3. The "Live Types" Problem**

When developing, you want changes in package source to immediately reflect in consuming apps. Two solutions:

- Point types directly to source (dev only)
- Use TypeScript project references (more robust for large monorepos)

**4. Common Gotcha: Missing `./package.json` Export**

Some tools (Jest, ts-node) need to read package.json directly:

```json
{
  "exports": {
    "./user-service": { /* ... */ },
    "./package.json": "./package.json"
  }
}
```

## Sources

- [Managing TypeScript Packages in Monorepos | Nx Blog](https://nx.dev/blog/managing-ts-packages-in-monorepos)
- [Live types in a TypeScript monorepo](https://colinhacks.com/essays/live-types-typescript-monorepo)
- [Turborepo TypeScript Guide](https://turborepo.dev/docs/guides/tools/typescript)
