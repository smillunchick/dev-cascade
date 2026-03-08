# Dev Cascade — Agent Instructions

> For any LLM coding agent: Cursor, Windsurf, Aider, Cline, or raw API.
> Paste this into your system prompt, rules file, or agents.md.

---

## What This Is

A development lifecycle encoded as agent instructions. Eight phases, each with clear inputs, actions, and outputs. The phases cascade — each one hands off to the next.

## The Cascade

```
/start → implement → /check → /commit → /finish → /ship
                                           ↑
                              /pause ← interrupt → /start (resume)
                              /bug  ← bug found → /check → /commit
```

---

## Phase 1: Start Work

**Trigger:** Beginning new feature, fix, refactor, or any development task.

### Instructions

1. **Check for `.continue-here.md`** at project root. If it exists:
   - Read it (contains state from a previous session)
   - State what was done and what's next
   - Delete the file (it's ephemeral)
   - Skip to the step it indicates

2. **Create a branch:**
   ```
   git checkout main && git pull
   git checkout -b <type>/<kebab-description>
   ```
   Types: `feat`, `fix`, `refactor`, `docs`, `test`, `chore`, `perf`

3. **Load context precisely** — grep ROADMAP.md, CHANGELOG.md, docs/bugs.md for keywords related to the task. Read only matching sections, not entire files.

4. **Don't hand-roll** — before implementing anything, search for:
   - Existing codebase utilities that already do this
   - Installed dependencies that solve this
   - Standard library features that cover this
   - If something exists, use it. State what you found.

5. **Discuss gray areas** — for features and complex fixes, identify decisions with multiple reasonable approaches. Ask the user about each one with concrete options, not open-ended questions. Record decisions.

6. **Assess complexity:**
   - Simple (< 3 files): proceed directly
   - Moderate (3-8 files): outline planned commits + write a boundary map
   - Large (8+ files): brainstorm first, then write a detailed plan

7. **Boundary map** (moderate+ only):
   ```markdown
   ## Boundary Map: <feature>
   ### Produces
   - functionName(args) → ReturnType (file.ts)
   ### Consumes
   - existing.table (existing)
   - config.newField (new)
   ### Interface Changes
   - ExistingType gains optional newField?: type
   ```

---

## Phase 2: Implement

### During Implementation

After writing each non-trivial function, answer these correctness questions:

1. What's the **time complexity**? Could it be better?
2. What **I/O** does this do per call?
3. Does this **allocate** where it could reuse?
4. If this runs **10,000 times**, what breaks?
5. Can I state **WHY** this approach, not just what it does?

**Anti-sycophancy:** Before implementing, check:
- Is there a simpler alternative?
- What assumption in the request might be wrong?
- Is this the minimum change needed?

Raise real concerns before coding. Don't rubber-stamp requests.

---

## Phase 3: Quality Check

**Trigger:** Before every commit. Four stages, in order.

### Stage 1: Correctness

For each changed production file, produce three proofs:

1. **Prove correct** — explain WHY the code makes its choices, not just that it compiles
2. **Prove not over-engineered** — could something simpler achieve the same result?
3. **Prove performance acceptable** — state expected cost for I/O and iteration

Output per-file verdicts:
```
- file.ts:functionName — CORRECT: [one-line reasoning]
- file.ts:otherFn — FLAG: [concern]. Accept: [reasoning] / Fix: [what to change]
```

Flags must resolve before proceeding (fix, measure, or explicitly accept with reasoning).

### Stage 2: Code Quality Review

Three parallel reviews on the diff:

**Reuse:** Duplicated logic? Existing utilities ignored? Library features unused?
**Quality:** Redundant state? Leaky abstractions? Dead code? Unnecessary complexity?
**Efficiency:** N+1 patterns? Missed concurrency? Unbounded operations? Hot-path bloat?

Fix issues found. Note false positives and move on.

### Stage 3: Mechanical Gates

Run all project gates. All must pass:
```bash
<build> && <typecheck> && <test>
```
If any fail: fix the issue, re-run from the earliest affected stage.

### Stage 4: Behavioral Verification

Beyond "tests pass" — does the actual behavior work?

| Change | Check |
|--------|-------|
| New API endpoint | Call it, verify response |
| New CLI command | Run it, verify output |
| Config change | Load config, verify effect |
| New integration | Verify registration + basic invocation |

One smoke check. If the change has no observable behavior beyond tests, skip.

### Smart Skip Rules

| Change Type | Correctness | Quality | Gates | Behavioral |
|-------------|-------------|---------|-------|------------|
| Feature/Fix | RUN | RUN | RUN | RUN |
| Refactor | skip | RUN | RUN | skip |
| Docs only | skip | skip | skip | skip |
| Tests only | skip | RUN | RUN | skip |
| Config | RUN | skip | RUN | RUN |

---

## Phase 4: Commit

**Format:** Conventional Commits

```
<type>(<scope>): <description>

[body: explains WHY]

[footer: Fixes: BUG-XXX]
```

**Rules:**
- Imperative mood: "add feature" not "added feature"
- One logical change per commit
- If you need "and" in the description, it's two commits
- Body explains WHY, not WHAT (the diff shows WHAT)
- Scope comes from project's CLAUDE.md or directory structure

**Split signals** — split into multiple commits if:
- You need "and" in the description
- Changes span unrelated modules
- You could revert one part but not the other

---

## Phase 5: Finish Branch

**Trigger:** Feature or fix is complete, ready to merge.

1. **Final verification** — run all gates one last time
2. **Fractal summaries** — three tiers, each regenerated from source (never re-compress):
   - **Commit body** (task-level): what changed, why, key decisions
   - **CHANGELOG entry** (feature-level): one-line per change, grouped by type
   - **Build journal** (milestone-level): for significant work only — decisions, trade-offs, lessons
3. **Update project docs:**
   - ROADMAP.md — mark completed items
   - CHANGELOG.md — add entries under current version
   - docs/bugs.md — check off fixed bugs
   - docs/build-journal.md — narrative entry (significant work only)
4. **Commit doc updates:** `docs: update ROADMAP and CHANGELOG for <feature>`
5. **Merge** — fast-forward or squash. Delete branch after merge.

---

## Phase 6: Ship Release

**Trigger:** Ready to release a new version.

1. **Determine version** — review commits since last tag:
   - Breaking changes → MAJOR
   - New features → MINOR
   - Bug fixes → PATCH

2. **Full verification** — all gates must pass

3. **Update docs:**
   - CHANGELOG.md with version and date
   - ROADMAP.md status
   - Build journal (MINOR+ only)

4. **Bump, tag, deploy:**
   ```bash
   npm version X.Y.Z --no-git-tag-version
   git commit -m "chore: bump version to X.Y.Z"
   git tag vX.Y.Z
   <deploy command>
   ```

5. **Post-deploy verify** — check the release actually works

---

## Side Phases

### Pause (Session Interruption)

**Trigger:** Ending session with unfinished work, context window filling up.

Write `.continue-here.md` at project root (~500 words max):

```markdown
# Continue Here

## Branch
<current branch>

## Status
<one-line phase description>

## Completed
- <what's done, with file paths>

## Remaining
- <what's left, priority order>

## Decisions Made
- <decision: rationale> — don't re-debate these

## Discovered Challenges
- <unexpected issues, gotchas>

## Next Action
<exact first thing to do — specific enough to start immediately>
```

Commit current work (even WIP). The next session's Start phase consumes and deletes this file.

### Bug Workflow

**Trigger:** Discovering or fixing a bug.

1. **Classify:** P0 (system down), P1 (broken, workaround exists), P2 (minor)
2. **Report:** Add to docs/bugs.md: `- [ ] [P1] Title — description. (date)`
3. **Track:** Add to ROADMAP.md with target version
4. **Fix:**
   - Create `fix/<kebab-description>` branch
   - Write failing test FIRST (must fail — proves it catches the bug)
   - Fix the bug
   - Verify test passes
   - Run full quality check
   - Commit with `Fixes: BUG-XXX` footer
5. **Close:** Check off in bugs.md, update CHANGELOG and ROADMAP

**The regression test is non-negotiable.**

---

## Project Setup

Run once per project. Scan for conventions, then scaffold:

**Auto-discover:**
- Language and build system (package.json, pyproject.toml, Cargo.toml, go.mod)
- Test framework (vitest, jest, pytest, cargo test, go test)
- Commit conventions (conventional? freeform?)

**Ask the user about:**
- Commit scopes
- Deploy process
- Hot paths and performance-critical areas
- Known invariants
- Past failures

**Scaffold these files:**
- CLAUDE.md (or equivalent) — workflow section with gate commands
- ROADMAP.md — version-tracked features and bugs
- CHANGELOG.md — grouped by type (Added, Fixed, Changed)
- docs/bugs.md — severity-tagged bug tracker
- .proving-correctness.md — hot paths, invariants, baselines

**Gate commands by language:**

| Language | Build | Typecheck | Test |
|----------|-------|-----------|------|
| TypeScript | `npm run build` | `npm run typecheck` | `npm test` |
| Python | skip | `mypy .` | `pytest` |
| Rust | `cargo build` | (included) | `cargo test` |
| Go | `go build ./...` | `go vet ./...` | `go test ./...` |

---

## Anti-Patterns to Watch For

| Excuse | Reality |
|--------|---------|
| "This is obviously correct" | Then proving it takes 10 seconds. Do it. |
| "Tests pass, good enough" | Tests don't catch over-engineering or performance. |
| "I'll clean it up later" | Later never comes. Simplify now. |
| "Just a small change" | Small changes cause big bugs. |
| "It's just a refactor" | Skip correctness proofs only. Quality + gates still apply. |
| "Tests catch everything" | Tests catch behavior, not performance invariants. |
| "I already checked mentally" | Prove it in writing. |

---

## Key Concepts

**Fractal summaries:** Three tiers of documentation that compress upward. Commit body → CHANGELOG → build journal. Never compress an already-compressed summary — regenerate from source.

**Boundary maps:** Before implementing moderate+ changes, declare what the code produces and consumes. Catches interface mismatches before coding.

**Don't hand-roll:** Always search for existing solutions before implementing. Codebase first, then dependencies, then standard library.

**Continue-here files:** Ephemeral session state that prevents re-debating decisions and losing context across sessions.

**Smart skip rules:** Not every change needs every check. Doc-only commits skip everything. Refactors skip correctness proofs (existing tests are proof). But mechanical gates always run.
