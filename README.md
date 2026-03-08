# Dev Cascade

Eight skills that cascade through the development lifecycle for Claude Code. Each skill invokes the next. The chain catches what you miss.

## Install

Copy `skills/` into `~/.claude/skills/`:

```bash
cp -r skills/* ~/.claude/skills/
```

Then run `/dev-teach` once in any project to scaffold the files the cascade depends on.

## The Cascade

```
/dev-start → implement → /dev-check → /dev-commit → /dev-finish → /dev-ship
                                                        ↑
                                   /dev-pause ← interrupt → /dev-start (resume)
                                   /dev-bug  ← bug found → /dev-check → /dev-commit
```

| Skill | What it does |
|-------|-------------|
| `/dev-start` | Branch, load context, don't-hand-roll, gray areas, boundary maps |
| `/dev-check` | 4-stage quality chain: correctness → simplify → gates → behavioral |
| `/dev-commit` | Conventional commits with project scopes, split signals |
| `/dev-pause` | Session continuity via `.continue-here.md` |
| `/dev-finish` | Fractal summaries, doc updates, merge workflow |
| `/dev-ship` | Version determination, CHANGELOG, tag, deploy |
| `/dev-bug` | Severity classification, mandatory TDD regression test |
| `/dev-teach` | One-time project setup, convention scanning, scaffolding |

## Bundled Dependencies

These skills are invoked by the cascade and included in this repo:

| Skill | Used by | Purpose |
|-------|---------|---------|
| `/proving-correctness` | dev-check (Stage 1), dev-start | Anti-sycophancy gates, correctness proofs |
| `/simplify` | dev-check (Stage 2) | Code reuse, quality, efficiency reviews |

## Optional Dependencies (Superpowers)

These superpowers skills are referenced but not bundled. Install [superpowers](https://github.com/coda-oa/superpowers) separately:

- `superpowers:brainstorming` — used by dev-start for complex work
- `superpowers:writing-plans` — used after brainstorming
- `superpowers:test-driven-development` — used by dev-start, dev-bug
- `superpowers:systematic-debugging` — used by dev-bug
- `superpowers:verification-before-completion` — used by dev-finish
- `superpowers:finishing-a-development-branch` — used by dev-finish

The cascade works without superpowers — they enhance it but aren't required.

## For Non-Claude Agents

See [`agents.md`](agents.md) — a self-contained version of the entire workflow as plain instructions. Works with Cursor, Windsurf, Aider, Cline, or any LLM coding agent. Paste it into your system prompt, rules file, or `.cursorrules`.

## Inspirations

- **[Superpowers](https://github.com/coda-oa/superpowers)** — the agent skill system for Claude Code
- **[Impeccable](https://impeccable.style)** — the cascading skill pattern
- **[GSD 2.0](https://www.lennysnewsletter.com/p/how-to-build-anything-with-ai-gsd)** — Boris Cherny's multi-session AI coding framework (fractal summaries, boundary maps, continue-here files, don't hand-roll)
