# Dev Cascade

Eight skills that cascade through the development lifecycle for Claude Code. Each skill invokes the next. The chain catches what you miss.

**[Documentation site](https://smillunchick.github.io/dev-cascade/)**

## Install — Claude Code

**Prerequisites:** Install [superpowers](https://github.com/coda-oa/superpowers) first — the cascade composes with it for brainstorming, planning, TDD, debugging, verification, and branch completion.

```bash
claude install github:coda-oa/superpowers
```

Then install the dev cascade skills:

```bash
git clone https://github.com/smillunchick/dev-cascade.git /tmp/dev-cascade
cp -r /tmp/dev-cascade/skills/* ~/.claude/skills/
rm -rf /tmp/dev-cascade
```

Run `/dev-teach` once in any project to scaffold the files the cascade depends on.

## Install — Any Other Agent (Cursor, Windsurf, Aider, Cline)

```bash
curl -sL https://raw.githubusercontent.com/smillunchick/dev-cascade/master/agents.md -o .cursorrules
```

Or copy [`agents.md`](agents.md) into your `.windsurfrules`, `AGENTS.md`, system prompt, or whatever your tool uses.

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

## Superpowers Dependency

The cascade directly invokes these superpowers skills:

| Superpowers skill | Used by | When |
|-------------------|---------|------|
| `brainstorming` | dev-start | Large complexity (8+ files) |
| `writing-plans` | dev-start | After brainstorming |
| `test-driven-development` | dev-start, dev-bug | Feature implementation, regression tests |
| `systematic-debugging` | dev-bug | Root cause analysis |
| `verification-before-completion` | dev-finish | Final verification |
| `finishing-a-development-branch` | dev-finish | Merge/PR workflow |

## For Non-Claude Agents

See [`agents.md`](agents.md) — a self-contained version of the entire workflow as plain instructions. Works with Cursor, Windsurf, Aider, Cline, or any LLM coding agent. Paste it into your system prompt, rules file, or `.cursorrules`.

## Inspirations

- **[Superpowers](https://github.com/coda-oa/superpowers)** — the agent skill system for Claude Code
- **[Impeccable](https://impeccable.style)** — the cascading skill pattern
- **[GSD 2.0](https://www.lennysnewsletter.com/p/how-to-build-anything-with-ai-gsd)** — Boris Cherny's multi-session AI coding framework (fractal summaries, boundary maps, continue-here files, don't hand-roll)
