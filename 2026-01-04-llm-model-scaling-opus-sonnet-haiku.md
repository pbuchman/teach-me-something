# LLM Model Scaling — What Actually Differentiates Opus, Sonnet, and Haiku

*Learned: 2026-01-04*

## Context

Discussing Claude Code pricing led to questions about when to use which model. Beyond "faster vs smarter", wanted to understand the technical reasons behind model tier differences.

## Insight

### Parameters = "Brain Capacity"

| Model | Parameters (estimate) |
|-------|----------------------|
| Haiku | ~20-30B |
| Sonnet | ~70-100B |
| Opus | ~200-400B+ |

Parameters are weights in the neural network — more parameters = more "patterns" the model can memorize and combine.

### What Actually Happens Inside

**1. Layers (depth)**

A transformer is a stack of attention + feed-forward layers. Each layer is one "thinking step":

```
Input → [Layer 1] → [Layer 2] → ... → [Layer N] → Output
```

- Haiku: ~40 layers
- Sonnet: ~80 layers
- Opus: ~120+ layers

More layers = more transformation steps = ability for more complex reasoning in a single forward pass. Shallow models must "shortcut".

**2. Hidden dimension (width)**

Each token is represented as a vector (e.g., 4096 numbers). Larger dimensionality = richer representations:

```
"bank" in Haiku:  [0.2, -0.1, ...] (4096 dims)
"bank" in Opus:   [0.2, -0.1, ...] (12288+ dims)
```

Opus can encode more semantic nuances in a single vector — "river bank" vs "money bank" are better separated in the embedding space.

**3. Attention heads**

Multi-head attention allows the model to look at different aspects simultaneously:

```
Head 1: syntax (what's the subject?)
Head 2: semantics (what does the word mean in context?)
Head 3: long-range relations (what was 1000 tokens ago?)
...
```

- Haiku: ~32 heads
- Opus: ~96+ heads

More heads = more parallel "perspectives" on each token.

### Emergent Abilities — Qualitative Jump, Not Quantitative

This is key. Certain abilities **emerge** only at specific scale:

```
Parameters:       10B     50B     100B    200B+
                 ─────────────────────────────────
Chain-of-thought:  ❌      ⚠️       ✅       ✅
Analogies:         ❌      ❌       ⚠️       ✅
Self-correction:   ❌      ❌       ❌       ✅
```

Haiku isn't "worse Opus" — it **cannot do** certain things that Opus does naturally. It's like the difference between a calculator and a spreadsheet.

**Example:**
```
Prompt: "Find the bug in this code and explain why it was introduced"

Haiku: Finds obvious bug, doesn't understand "why"
Sonnet: Finds bug, tries to guess intent
Opus: Finds bug, reconstructs author's reasoning, proposes fix preserving intent
```

### KV Cache and "Working Memory"

During generation, the model holds Key-Value cache — its "working memory":

```
KV Cache size ≈ layers × heads × sequence_length × head_dim
```

Larger model = larger KV cache = more "working memory" for context.

That's why Opus handles 100k context better — it has space to maintain relationships between distant fragments.

### Training Differences

All models likely share:
- Same base training corpus
- Same architecture (transformer)
- Same RLHF/Constitutional AI approach

The difference is scale + probably more fine-tuning iterations for larger models (they're more expensive, so more investment in quality).

### Practical Implication

```
Task: "Refactor this code"

Haiku sees:   tokens → pattern matching → output
Sonnet sees:  tokens → structure → dependencies → output
Opus sees:    tokens → structure → dependencies → intents →
              edge cases → tradeoffs → output
```

Haiku applies patterns. Opus **understands** what you're doing and why.

## Sources

- Internal discussion about Claude model pricing and capabilities
- General knowledge of transformer architecture and scaling laws
