---
name: dev-pause
description: Use when ending a session with work still in progress — captures completed work, remaining tasks, decisions made, discovered challenges, and exact next action into a continue-here file that /dev-start consumes on resume. Triggers on "stopping for now", "end of session", "pick this up later", "save my progress", "I need to go", or when context window is getting full mid-task.
user-invokable: true
---

# Pause Work

## Overview

Capture state so the next session starts at full speed, not from zero.

**Core principle:** Decisions are expensive. Don't make them twice.

## When to Use

- Ending a session with unfinished work
- Context window getting full mid-task
- Switching to a different project temporarily
- Before any interruption where continuity matters

## The Capture

Write `.continue-here.md` at the project root. This file is ephemeral — `/dev-start` consumes and deletes it on resume.

**Budget: ~500 words max.** This is a compressed summary, not a context dump. Decisions, not deliberations.

### Template

```markdown
# Continue Here

## Branch
<current branch name>

## Status
<one-line: what phase of work you're in>

## Completed
- <what's done, with file paths>
- <key changes made>

## Remaining
- <what's left, in priority order>
- <estimated complexity of each item>

## Decisions Made
- <decision: rationale> — don't re-debate these
- <decision: rationale>

## Discovered Challenges
- <unexpected issues found>
- <gotchas for the next session>

## Next Action
<exact first thing to do on resume — specific enough to start immediately>
```

### What to Include

- **Branch name** — so `/dev-start` can check it out
- **Completed work** — files changed, tests written, what's verified
- **Remaining work** — ordered by priority, with complexity notes
- **Decisions made** — the WHY, not just the WHAT. These prevent re-debate
- **Discovered challenges** — things that surprised you, warnings for next time
- **Exact next action** — "implement the `recall()` TTL filter in engine/memory-engine.ts" not "continue working on memory"

### What NOT to Include

- Full code snippets (they're in git)
- Conversation history or reasoning traces
- Verbose explanations of what the code does
- Anything git already tracks

## After Writing

1. Commit current work if anything is committable (even WIP):
   ```
   chore(<scope>): WIP <feature-description>
   ```
2. State: "Progress saved to `.continue-here.md`. Resume with `/dev-start`."

## Cascade

**On resume:** `/dev-start` detects `.continue-here.md`, loads state, picks up where you left off.

**All dev skills:** `~/.claude/skills/dev-*` — run `/dev-teach` to set up a new project.
