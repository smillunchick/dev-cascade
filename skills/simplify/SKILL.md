---
name: simplify
description: Review changed code for reuse opportunities, code quality issues, and efficiency problems. Runs three parallel subagent reviews and reports findings. Used as Stage 2 of the /dev-check quality chain. Triggers on "simplify", "review code quality", "check for issues".
user-invokable: true
args:
  - name: scope
    description: Limit review to specific files (optional, defaults to all changed files)
    required: false
---

# Simplify

## Overview

Three lenses on the same code, run in parallel. Catches what correctness proofs miss.

**Core principle:** Simple code is correct code. Complexity is where bugs hide.

## When to Use

- As Stage 2 of `/dev-check` (after proving-correctness, before mechanical gates)
- When you want a quality review of changed code
- Before refactoring — find what actually needs simplifying

## The Three Reviews

Run all three in parallel subagents. Each gets the diff and returns findings.

### Review 1: Code Reuse

Look for:
- **Duplicated logic** — same pattern in 2+ places that should be a shared function
- **Existing utilities ignored** — project has a helper for this, but code hand-rolls it
- **Library features unused** — dependency already provides this capability
- **Standard library alternatives** — built-in language/runtime feature covers this

Output:
```
Reuse: [file:function] — duplicates [existing:function]. Use the existing one.
Reuse: [file:function] — lodash.groupBy already does this. Remove hand-rolled version.
Reuse: clean — no duplication found.
```

### Review 2: Code Quality

Look for:
- **Redundant state** — computed values stored when they could be derived
- **Leaky abstractions** — implementation details exposed through interfaces
- **Copy-paste drift** — similar code blocks that diverged slightly (likely a bug)
- **Dead code** — unreachable branches, unused variables, commented-out blocks
- **Naming** — misleading names, inconsistent conventions, abbreviations that obscure meaning
- **Unnecessary complexity** — abstractions, indirection, or generality nobody asked for

Output:
```
Quality: [file:line] — redundant state. `isActive` can be derived from `status === 'active'`.
Quality: [file:function] — abstraction adds indirection without benefit. Inline it.
Quality: clean — no issues found.
```

### Review 3: Efficiency

Look for:
- **N+1 patterns** — database queries or API calls inside loops
- **Missed concurrency** — sequential awaits that could be parallel (Promise.all)
- **Hot-path bloat** — heavy computation in frequently-called code paths
- **Unbounded operations** — queries without LIMIT, iterations without bounds
- **Unnecessary I/O** — reads/writes that could be cached, batched, or eliminated

Output:
```
Efficiency: [file:function] — N+1: query inside forEach. Batch with WHERE IN.
Efficiency: [file:function] — sequential awaits. Use Promise.all for independent calls.
Efficiency: clean — no issues found.
```

## Handling Findings

For each finding:
- **Fix** — change the code (most findings)
- **Accept** — state explicit reasoning for leaving as-is (rare, must justify)
- **False positive** — note it and move on

## Verdict

After all three reviews:

```
Simplify: PASS
- Reuse: clean
- Quality: 1 issue fixed (inlined unnecessary abstraction)
- Efficiency: clean
```

Or:

```
Simplify: 2 issues remaining
- Quality: [file:line] — accepted: abstraction needed for test isolation
- Efficiency: [file:function] — accepted: N+1 bounded to max 5 items
```

## Red Flags

- "The code works, no need to simplify" — working code can still be unnecessarily complex
- "I'll clean it up later" — later never comes
- "It's only used once" — once is enough to matter if it's in a hot path
- Skipping any of the three reviews

## Cascade

**Previous:** Stage 1 (proving-correctness) in `/dev-check`
**Next:** Stage 3 (mechanical gates) in `/dev-check`

**All dev skills:** `~/.claude/skills/dev-*` — run `/dev-teach` to set up a new project.
