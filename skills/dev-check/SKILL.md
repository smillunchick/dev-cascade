---
name: dev-check
description: Use before every commit — runs the full pre-commit quality chain (proving-correctness in subagent, simplify, build+typecheck+test, behavioral verification) on changed files with smart skip rules based on change type. Also use when you want to validate work-in-progress, check code quality, or verify changes are ready. Triggers on "check my code", "run quality chain", "validate changes", "ready to commit?".
user-invokable: true
args:
  - name: scope
    description: Limit check to specific files (optional, defaults to all changed files)
    required: false
---

# Quality Gate

## Overview

The pre-commit quality chain. Four stages, in order, no shortcuts.

**Core principle:** Every commit passes through the chain. The chain catches what you miss.

**Violating the letter of these rules is violating the spirit of these rules.**

## The Chain

```
Stage 1: /proving-correctness  →  Stage 2: /simplify  →  Stage 3: mechanical gates  →  Stage 4: behavioral verify
   (correctness, in subagent)       (quality, subagents)     (build+typecheck+test)       (does it actually work?)
```

## Step 1: Identify Changes and Change Type

```bash
git diff --name-only HEAD
```

Classify the change:

| Change Type | How to identify |
|-------------|----------------|
| Feature/fix code | Production source files changed |
| Refactor | Source files changed, no behavior change (user states this) |
| Docs only | Only `.md` files changed |
| Test only | Only `*.test.ts` or `tests/` files changed |
| Config change | Only `config/`, `*.yaml`, `*.json` config files changed |

If mixed (e.g., feature + docs), use the strictest applicable rules.

## Step 2: Apply Skip Rules

| Change Type | Stage 1 (correctness) | Stage 2 (simplify) | Stage 3 (gates) | Stage 4 (behavioral) |
|-------------|----------------------|--------------------|-----------------|---------------------|
| Feature/fix | RUN | RUN | RUN | RUN |
| Refactor | SKIP — existing tests are proof | RUN | RUN | SKIP |
| Docs only | SKIP | SKIP | SKIP | SKIP |
| Test only | SKIP | RUN | RUN | SKIP |
| Config | RUN (config questions) | SKIP | RUN | RUN |

## Stage 1: Proving Correctness (Subagent)

**Run in a subagent** to keep correctness reasoning out of the main context window. The subagent receives the diff and returns only the verdicts.

Invoke `/proving-correctness` — specifically Gate 3 (Pre-Commit Review).

For each changed production file, produce three proofs:

1. **Prove correct** — explain WHY the code makes its choices, not just that it compiles
2. **Prove not over-engineered** — could something simpler achieve the same result?
3. **Prove performance acceptable** — state expected cost for I/O and iteration

Output per-file verdicts:
```
- engine/memory.ts:recallMemories — CORRECT: FTS5 query bounded by LIMIT, O(log n) lookup
- engine/qa.ts:reviewMessage — FLAG: Haiku call per message. Accept: QA is inherently per-message.
```

**Flags must resolve before proceeding:**
- **Fix** — change the code
- **Measure** — run a test that proves the concern is unfounded
- **Accept** — state explicit reasoning for accepting the risk

## Stage 2: Simplify

Invoke `/simplify` on all changed files.

Three parallel subagent reviews: code reuse, code quality, efficiency. Fix any issues found. If a finding is a false positive, note it and move on.

## Stage 3: Mechanical Gates

Run all three. All must pass.

```bash
npm run build && npm run typecheck && npm test
```

If any gate fails:
1. Read the error output
2. Fix the issue
3. Re-run from the earliest stage affected by the fix

## Stage 4: Behavioral Verification

Beyond "tests pass" — does the actual behavior work?

Determine what's verifiable based on the change:

| Change | Behavioral Check |
|--------|-----------------|
| New API endpoint | Call it, verify response shape and status |
| New CLI command | Run it, verify output |
| Config change | Load the config, verify the value takes effect |
| Message handler | Send a test message (or dry-run), verify pipeline |
| New integration | Verify tool registration and basic invocation |

If the change has no observable behavior beyond what tests cover, note that and skip.

**This is not exhaustive testing.** It's a single smoke check: "Does the thing I just built actually do the thing?" If you can't answer that from test output alone, run it.

## Verdict

After all applicable stages pass, state the verdict:

```
Quality chain: PASS
- Correctness: 4 files reviewed, 0 flags (subagent)
- Simplify: clean (or: 2 issues fixed)
- Gates: build OK, typecheck OK, 984 tests pass
- Behavioral: memory recall with TTL returns only fresh memories (verified)
Ready to commit.
```

## Red Flags — STOP

- "Tests pass, good enough" — stages 1, 2, and 4 catch what tests miss
- "I already checked correctness mentally" — prove it in writing
- "Just a small change" — small changes cause big bugs
- Skipping any stage without matching the skip rules above
- Running gates out of order
- Re-running only stage 3 after fixing a correctness issue (re-run from stage 1)

## Rationalization Prevention

| Excuse | Reality |
|--------|---------|
| "This is obviously correct" | Then proving it takes 10 seconds. Do it. |
| "I'll run simplify after" | The chain is sequential for a reason. Now. |
| "Tests catch everything" | Tests don't catch over-engineering or performance. |
| "It's just a refactor" | Skip stages 1 and 4 only. Stages 2-3 still apply. |
| "Config changes don't need checks" | Config changes need config-specific correctness questions. |
| "I already ran this" | Show the output. If you can't, run it again. |
| "Behavioral check is overkill" | One smoke check takes 30 seconds. Ship with confidence. |

## Cascade

**Next:** `/dev-commit`

**All dev skills:** `~/.claude/skills/dev-*` — run `/dev-teach` to set up a new project.
