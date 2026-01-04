# Where Do Output Tokens Come From in Claude Code?

*Learned: 2026-01-04*

## Context

Looking at a Claude Code session with 917k output tokens and wondering: does this mean 917k × 4 characters were sent from the LLM? The answer reveals how much happens "behind the scenes."

## Insight

### Tokens ≠ Characters

The ~4 chars/token is a rough average for English prose, but varies:
- Code: ~3 chars/token (more symbols, short keywords)
- Non-English: ~2-3 chars/token (diacritics, longer words)
- JSON: ~3-4 chars/token

917k tokens ≈ **2.5-3.5M characters** (not 3.7M).

### What Makes Up Output Tokens in Claude Code?

What you **see** is only part of the output. Full output includes:

**1. Text to User (~10-15%)**
The responses and explanations you read.

**2. Tool Calls (~70-80%) — The Big Cost!**
Every tool invocation is structured output:

```json
{
  "type": "tool_use",
  "name": "Edit",
  "input": {
    "file_path": "/Users/name/project/src/file.ts",
    "old_string": "... entire old code block ...",
    "new_string": "... entire new code block ..."
  }
}
```

**A single Edit with 50 lines of code = ~500-1000 output tokens.**

**3. Thinking/Reasoning (~10-15%)**
Extended thinking generates "reasoning" tokens before the response — these count toward output too.

### Breakdown Example

```
917k output tokens =
├── ~100k  text to user (10-15%)
├── ~700k  tool calls (70-80%)  ← main cost!
│   ├── Read (large files echoed)
│   ├── Edit (old_string + new_string)
│   ├── Write (entire file content)
│   ├── Bash (commands + full output)
│   └── Task (prompts to subagents)
└── ~100k  thinking/reasoning (10-15%)
```

### Why Are Tool Calls So Expensive?

```python
# Edit tool must output:
old_string = "entire fragment being changed"  # ~200 tokens
new_string = "entire new fragment"            # ~200 tokens
file_path  = "/full/path/to/file.ts"         # ~20 tokens
# = 420 tokens for one small edit

# Write tool:
content = "ENTIRE FILE CONTENTS"  # can be 2000+ tokens
```

50 edits in a session = 20k+ tokens just in tool calls.

### Key Takeaway

917k output isn't "917k characters of text to you" — it's mostly **structured tool calls** that Claude Code executes behind the scenes. You see maybe 15% of it as visible text.

This is why coding sessions are output-heavy compared to chat conversations.

## Sources

- Direct analysis of Claude Code session token metrics
