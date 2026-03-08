# Correctness Interrogation Reference

## Pushback Block (Before Implementation)

Run before writing implementation code for any feature or significant change.

```markdown
## Pushback
- **Simpler alternative?** [Could a one-liner, existing library, config change, or shell command replace this? What's the minimum viable approach?]
- **Assumption challenge:** [What assumption in the request might be wrong? What would a senior engineer question here?]
- **Scope check:** [Is this the minimum change needed? What could be removed and still solve the problem?]
```

If all three are clean, state that and proceed. If any flag trips, raise it with the user before implementing. No "noted, proceeding anyway."

## Inline Checkpoint (During Implementation)

Run after writing each non-trivial function, module, or config change. Skip only for: type definitions, test files, documentation.

| # | Question | Failure it catches |
|---|----------|-------------------|
| 1 | **What's the time complexity? Could it be better?** | O(n^2) when O(log n) exists. Full scans when index lookups work. Nested loops over collections. |
| 2 | **What I/O does this do per call?** (disk, network, syscalls) | fsync per row instead of batched. N+1 queries. Unbuffered writes. Redundant network calls. |
| 3 | **Does this allocate where it could reuse?** (copies, clones, new objects) | `.to_vec()` on cache hits. Object creation in hot loops. Cloning when borrowing works. |
| 4 | **If this runs 10,000 times, what breaks?** | Memory leaks, connection exhaustion, file handle limits, unbounded queues, lock contention. |
| 5 | **Can I state WHY this approach, not just WHAT it does?** | Pattern-matching without understanding. "It works" is not an explanation. State the design reason. |

For config changes specifically, also ask:
- What does this value control at runtime?
- What happens if it's 10x too high? 10x too low?
- Is there a way to validate this value is correct, not just syntactically valid?

## Pre-Commit Review (Before Committing)

Run alongside `/simplify` on the full diff. For each changed file:

### Three Proofs

1. **Prove correctness.** Explain *why* this code makes the choices it does. "It compiles and tests pass" is not proof. If you can't articulate the reasoning, flag it.

2. **Prove it's not over-engineered.** Could a simpler approach achieve the same result? Are you adding abstraction, generality, or configurability nobody asked for? The 82,000-line Rust vs `find -mtime +7` test.

3. **Prove performance is acceptable.** For code touching known hot paths or doing I/O/iteration, state the expected cost. "Should be fine" is not acceptable. Use project context if available.

### Output Format

```markdown
## Correctness Review: <commit summary>
- file.ts:fn_name — CORRECT: [one-line reasoning why]
- other.ts:fn_name — FLAG: [concern + what to measure/verify]
- config.yaml:setting — CORRECT: [why this value is right]
```

Flags must resolve before committing:
- **Fix** — change the code
- **Measure** — run a benchmark or test that proves the concern is unfounded
- **Accept** — state the reasoning for accepting the risk (must be explicit, not hand-wavy)

## Using Project Context

If `.proving-correctness.md` exists at the project root, load it and incorporate:
- Check changes against **Hot Paths** — any change to a hot path gets extra scrutiny
- Verify **Known Invariants** still hold after the change
- Compare against **Performance Baselines** where they exist
- Cross-reference **Past Failures** — are we repeating a known mistake?

If the file doesn't exist, work generically. Suggest running `/proving-correctness teach` for projects the user works on regularly.
