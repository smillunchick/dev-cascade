---
name: dev-finish
description: Use when feature or fix work is complete on a branch and ready to integrate — runs final verification, produces fractal summaries (commit body → CHANGELOG → build journal), updates project tracking docs (ROADMAP, CHANGELOG, build-journal, bugs), then hands off to finishing-a-development-branch for merge workflow. Triggers on "done with this feature", "ready to merge", "finish this branch", "work is complete".
user-invokable: true
---

# Finish Work

## Overview

Verify, document, integrate. The last mile of a change.

**Core principle:** Work isn't done until the docs reflect reality.

## Step 1: Final Verification

Use `superpowers:verification-before-completion` — evidence before claims.

Run the full gate one final time:
```bash
npm run build && npm run typecheck && npm test
```

Do NOT proceed if any gate fails.

## Step 2: Fractal Summaries

Produce three tiers of summary that compress upward. Each tier serves a different audience and time horizon:

### Tier 1: Commit Body (task-level detail)
What changed, why, key decisions made. Lives in git history forever. Written during `/dev-commit` — verify it exists and is substantive for the merge commit.

### Tier 2: CHANGELOG Entry (feature-level summary)
One-line description per change, grouped by type. Read by users and future developers scanning "what shipped." Should be specific enough to understand without reading commits.

### Tier 3: Build Journal Entry (milestone-level narrative)
For significant work only (new engine, integration, architectural change). Documents decisions, trade-offs, anti-patterns avoided, lessons learned. The drill-down for anyone who wants the full story.

Each tier compresses from the one below — CHANGELOG derives from commit messages, build journal synthesizes the CHANGELOG entries plus broader context. **Never compress an already-compressed summary** — regenerate from the source material.

## Step 3: Update Project Docs

Before merging, update tracking docs. Only update what applies:

### ROADMAP.md
- Mark completed items as done
- Update status for partially complete work
- Note the version this ships in

### CHANGELOG.md
Add entries under current version or "Unreleased." Group by type:
```markdown
## [Unreleased]
### Added
- feat commits (specific: "add TTL-based memory expiration" not "memory improvements")

### Fixed
- fix commits

### Changed
- refactor/perf commits
```

### docs/bugs.md
If this was a bug fix:
- Check off the bug entry: `- [x] [P1] ...`
- Add to Fix Log table if one exists

### docs/build-journal.md
For significant work only:
- Add narrative entry with decisions, trade-offs, anti-patterns avoided

## Step 4: Commit Doc Updates

If any docs changed:
```
docs: update ROADMAP and CHANGELOG for <feature/fix-name>
```

## Step 5: Branch Completion

Use `superpowers:finishing-a-development-branch` for merge/PR options:
1. Merge locally (fast-forward or squash)
2. Push and create PR
3. Keep branch as-is
4. Discard

After merge: delete the branch (trunk-based, short-lived branches).

## Cascade

**Next:**
- Releasing now → `/dev-ship`
- Starting next task → `/dev-start`

**All dev skills:** `~/.claude/skills/dev-*` — run `/dev-teach` to set up a new project.
