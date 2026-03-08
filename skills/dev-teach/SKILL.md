---
name: dev-teach
description: Use once per project to set up the dev cascade — scans the project for conventions (test framework, build commands, commit patterns), asks about workflow preferences, and scaffolds all files the /dev-* skills depend on (CLAUDE.md workflow section, ref/workflow.md, .proving-correctness.md, docs/bugs.md, ROADMAP.md, CHANGELOG.md). Triggers on "set up dev workflow", "init dev skills", "configure dev cascade", /dev-teach.
user-invokable: true
args:
  - name: project
    description: Path to the project root (optional, defaults to cwd)
    required: false
---

# Teach Dev Cascade

## Overview

One-time setup that configures a project for the `/dev-*` skill cascade. Scans the project, asks targeted questions, and scaffolds the files the cascade depends on.

**Core principle:** The dev cascade is portable. This skill makes it work anywhere.

**Skills location:** `~/.claude/skills/dev-*` (dev-start, dev-check, dev-commit, dev-finish, dev-ship, dev-bug, dev-teach)

## When to Use

- New project that should use the dev cascade
- Existing project missing files the cascade expects
- Porting the workflow to a different tech stack

## The Process

### Step 1: Scan the Project

Read existing files to gather context automatically. Don't ask about what you can discover:

**Build & test:**
- `package.json` → scripts (build, test, typecheck, lint), dependencies, version
- `pyproject.toml` / `setup.cfg` → Python equivalents
- `Cargo.toml` → Rust equivalents
- `go.mod` → Go equivalents
- `Makefile` / `justfile` → custom commands

**Conventions:**
- `git log --oneline -20` → existing commit style (conventional? freeform?)
- `.eslintrc*` / `biome.json` / `ruff.toml` → linting setup
- `tsconfig.json` → TypeScript strictness
- Test files → framework (vitest, jest, pytest, cargo test, go test)

**Existing workflow files:**
- `CLAUDE.md` → already has workflow section?
- `ref/workflow.md` → already exists?
- `.proving-correctness.md` → already configured?
- `ROADMAP.md`, `CHANGELOG.md`, `docs/bugs.md` → already exist?

Report what you found. Skip questions about things you've already discovered.

### Step 2: Ask the User

One question at a time, multiple choice where possible. Skip questions already answered by the scan.

1. **Commit scopes** — "Based on your source layout, these scopes make sense: [list]. Add or remove any?"

2. **Bug tracking** — "Want to track bugs in `docs/bugs.md` (markdown checklist) or do you use an external tracker (GitHub Issues, Linear, etc.)?"

3. **Versioning** — "You're at version [X]. Using semver? Any special rules for major bumps?"

4. **Deploy process** — "How do you deploy? (commands, CI/CD pipeline, manual)" — capture for `/dev-ship`

5. **Hot paths** — "What are the performance-critical paths in this codebase?" — for `.proving-correctness.md`

6. **Known invariants** — "What must always be true? (e.g., 'one request per user at a time', 'all API responses validated')"

7. **Past failures** — "Any subtle bugs that tests missed? Performance regressions? Patterns to watch for?"

### Step 3: Scaffold Files

Create or update each file. Never overwrite existing content — merge with what's there.

#### CLAUDE.md — Workflow Section

Add the dev cascade table if not present:

```markdown
## Workflow — Dev Cascade

Six skills drive the development lifecycle. Use them:

| Phase | Skill | What it does |
|-------|-------|-------------|
| Start work | `/dev-start` | Branch, load context, assess complexity |
| Quality gate | `/dev-check` | proving-correctness → simplify → build+typecheck+test |
| Commit | `/dev-commit` | Conventional Commits format, discipline, scope validation |
| Finish branch | `/dev-finish` | Update docs (ROADMAP, CHANGELOG), merge workflow |
| Release | `/dev-ship` | Version bump, tag, changelog, deploy |
| Bugs | `/dev-bug` | Report, track, TDD fix, close |

During implementation: `/proving-correctness` gates apply (pushback, inline checkpoints).
```

Also add/update the Pre-Commit Quality Chain section:

```markdown
## Pre-Commit Quality Chain

Run `/dev-check` before every commit. It orchestrates the full chain:

1. `/proving-correctness` Gate 3 on each changed file
2. `/simplify` on each changed file
3. <project-specific gate commands>

Smart skip rules by change type — see `/dev-check` for details.
```

#### ref/workflow.md

Create with project-specific details:
- Branching model (trunk-based by default)
- Commit format with project-specific scopes
- Versioning rules
- Bug workflow (with project-specific tracker)
- Feature workflow
- Release process (with project-specific deploy steps)
- Code review standards

Add the skills note at top:
```markdown
> **Skills:** This workflow is encoded in the `/dev-*` skill cascade. Use the skills — they enforce this process. This document is the reference spec.
```

#### .proving-correctness.md

If it doesn't exist, run `/proving-correctness teach` — it handles its own setup.

If it exists, confirm it's still accurate and offer to update.

#### ROADMAP.md

Create if missing:
```markdown
# Roadmap

## Current Version: vX.Y.Z

### Features
<!-- - [ ] Description → vX.Y.Z -->

### Tech Debt
<!-- - [ ] Description → vX.Y.Z -->

### Bugs
<!-- - [ ] BUG-XXX: Description [P1] → vX.Y.Z -->
```

#### CHANGELOG.md

Create if missing:
```markdown
# Changelog

## [Unreleased]

## [X.Y.Z] - YYYY-MM-DD
- Initial release with dev cascade workflow
```

#### docs/bugs.md

Create if missing (and user chose markdown tracking):
```markdown
# Bug Tracker

## Open

<!-- - [ ] [P1] Title — description. (YYYY-MM-DD) -->

## Fix Log

| Bug | Severity | Fixed In | Date |
|-----|----------|----------|------|
```

### Step 4: Verify Setup

Run the project's gate commands to confirm they work:
```bash
<build command> && <typecheck command> && <test command>
```

### Step 5: Confirm

Report:
- Files created/updated (with paths)
- Commit scopes configured
- Gate commands configured
- Next step: "Start working with `/dev-start`"

## Adapting to Non-TypeScript Projects

The dev cascade is language-agnostic. `/dev-teach` adapts:

| Project type | Build gate | Type gate | Test gate |
|-------------|-----------|-----------|-----------|
| TypeScript | `npm run build` | `npm run typecheck` | `npm test` |
| Python | skip (or `python -m py_compile`) | `mypy .` | `pytest` |
| Rust | `cargo build` | (included in build) | `cargo test` |
| Go | `go build ./...` | `go vet ./...` | `go test ./...` |

The `/dev-check` skill reads gate commands from CLAUDE.md, so whatever you configure here is what it runs.

## Red Flags

- Don't scaffold files the user explicitly doesn't want
- Don't overwrite existing content — merge
- Don't assume TypeScript — detect the stack
- Don't skip the scan — auto-discovery reduces questions
