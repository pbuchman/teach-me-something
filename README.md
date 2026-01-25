# Teach Me Something

A personal knowledge base of tech insights, powered by a Claude Code skill.

## What is This?

This repository is a growing collection of tech insights captured during coding sessions. Each insight is a markdown file documenting something interesting learned while building software — patterns, techniques, gotchas, and architectural decisions.

## The Claude Skill

This repo is populated by a **Claude Code skill** — a reusable prompt that extends Claude Code's capabilities. When you type `/teach-me-something` in Claude Code, it:

1. **Checks for duplicates** — Lists recent files to avoid teaching the same topic twice
2. **Picks a topic** from:
   - Your explicit request (`/teach-me-something [topic]`)
   - Recent conversation context (patterns, architecture decisions)
   - [Threads ClaudeCode community](https://www.threads.com/search?q=ClaudeCode) for trending tips
3. **Presents the insight** in a structured format
4. **Logs it here** as a markdown file (background agent)
5. **Commits and pushes** automatically

### What are Claude Code Skills?

[Claude Code](https://claude.ai/code) is Anthropic's CLI for AI-assisted software development. Skills are markdown files that define custom slash commands. They live in `.claude/commands/` and are invoked with `/skill-name`.

**Key benefits:**
- **Reusable** — Define once, use across sessions
- **Composable** — Skills can use tools, spawn agents, call APIs
- **Portable** — Copy to any project to add the capability
- **Version-controlled** — Skills are just files in your repo

---

## Setup: Add This Skill to Any Project

### Step 1: Create the Skill File

Create `.claude/commands/teach-me-something.md` in your project:

```markdown
# Teach Me Something

Share an interesting tech insight and log it for future reference.

**IMPORTANT: Always write content in English, regardless of the conversation language.**

**Usage:**

- `/teach-me-something` — pick topic from recent context or Threads/ClaudeCode community
- `/teach-me-something [topic]` — teach about specific topic

---

## Step 1: Check for Duplicates

**MANDATORY:** Before selecting a topic, check existing insights to avoid duplicates:

```bash
ls -t ~/personal/teach-me-something/*.md | head -20
```

Note the recent topics from filenames (format: `YYYY-MM-DD-topic-slug.md`). Do NOT teach about topics already covered.

---

## Step 2: Choose a Topic

**Priority order:**

1. If user provided a topic argument → teach about that (use WebSearch if needed for depth)
2. If interesting patterns, architecture decisions, or techniques were discussed recently → teach about that
3. Otherwise → search for trending ClaudeCode topics:

**ClaudeCode Community Search:**

```
WebFetch: https://www.threads.com/search?q=ClaudeCode&serp_type=default
Prompt: "Extract the most interesting technical insights, tips, or discoveries about Claude Code from the last few posts. Focus on practical techniques, workflows, or features. List 3-5 topics with brief descriptions."
```

Then pick the most interesting topic that:
- Is NOT already in `~/personal/teach-me-something/`
- Is practical and applicable
- Relates to: AI coding assistants, prompt engineering, developer workflows, TypeScript/Node.js

---

## Step 3: Teach (Immediate)

Present the insight directly in chat using this format:

```
★ Insight ─────────────────────────────────────
**[Topic Title]**

[2-4 key educational points with clear explanations and examples where helpful]

Sources:
- [Source Title 1](url)
- [Source Title 2](url)
─────────────────────────────────────────────────
```

If teaching from recent context, no web sources needed.
If teaching from web search, include 1-3 relevant sources.

---

## Step 4: Log to Repository (Background)

After presenting the insight, spawn a background agent using the Task tool:

```
Task tool parameters:
- subagent_type: "general-purpose"
- run_in_background: true
- prompt: |
    Create a markdown file documenting what was just taught.

    Topic: [TOPIC_TITLE]
    Context: [BRIEF_DESCRIPTION_OF_WHAT_USER_WAS_BUILDING - 1-2 sentences]
    Content: [FULL_EXPLANATION_FROM_STEP_3]
    Sources: [LIST_OF_SOURCES_IF_ANY]

    Steps:
    1. Create file: ~/personal/teach-me-something/YYYY-MM-DD-[topic-slug].md
       - Use today's date
       - Slug: lowercase, hyphens, max 50 chars

    2. File content format:
       # [Topic Title]

       *Learned: YYYY-MM-DD*

       ## Context

       [What I was building when I learned this - helps trigger memory later]

       ## Insight

       [Full explanation]

       ## Sources

       - [Title](url)

    3. Git operations:
       cd ~/personal/teach-me-something
       git add .
       git commit -m "Add: [topic-slug]"
       git push
```

Do NOT wait for the background task to complete. Continue conversation immediately after spawning.

---

## Step 5: Suggest Next Steps

After presenting the insight:

```
💡 **Related Explorations:**
- [Related topic 1 to explore next]
- [Related topic 2 to explore next]
- [How this applies to current work]
```
```

### Step 2: Clone This Repo

```bash
mkdir -p ~/personal
cd ~/personal
git clone https://github.com/YOUR_USERNAME/teach-me-something.git
```

### Step 3: Use It

In any Claude Code session:

```
/teach-me-something
```

Or with a specific topic:

```
/teach-me-something circuit breaker pattern
```

---

## Topic Discovery

When no specific topic is requested and recent context isn't interesting, the skill searches **Threads** for ClaudeCode community discussions:

```
https://www.threads.com/search?q=ClaudeCode&serp_type=default
```

This surfaces trending tips, workflows, and discoveries from the Claude Code community — keeping your knowledge base current with real-world usage patterns.

---

## Customization

### Change the Storage Location

Update the path in Step 4 of the skill to point to your preferred location:

```markdown
1. Create file: /path/to/your/knowledge-base/YYYY-MM-DD-[topic-slug].md
```

### Add Topic Categories

You can extend the skill to organize by category:

```markdown
1. Create file: ~/personal/teach-me-something/[category]/YYYY-MM-DD-[topic-slug].md
   - Categories: distributed-systems, typescript, devops, ai-ml, security
```

---

## File Format

Each insight follows this structure:

```markdown
# [Topic Title]

*Learned: YYYY-MM-DD*

## Context

What I was building when I learned this.

## Insight

The actual learning content with:
- Key points
- Code examples
- Trade-offs discussed

## Sources

- [Source Title](url)
```

---

## Example Insights

Browse the repository to see real examples of captured insights:
- TypeScript techniques (subpath exports, branded types)
- Cloud architecture decisions
- Claude Code workflows and tips
- Testing strategies
- AI/ML concepts

---

## License

MIT
