---
name: bob-adversarial-review
description: Adversarial code review by the current agent, without spawning subagents. Writes a severity-ranked report to .bob/state/review.md.
user-invocable: true
category: workflow
---

# Adversarial Review — Single Reviewer

Perform a hostile, read-only code review yourself. Do not spawn subagents, delegate
work, or modify source code. The goal is to find concrete defects, not to confirm
that the code looks reasonable.

## Scope

If invoked with `DIFF` or `diff`, review only files changed from the merge-base with
`main`:

```bash
git diff --name-only "$(git merge-base HEAD main)..HEAD"
```

Otherwise review the whole repository, excluding `.git`, generated dependencies,
and build artifacts. Read relevant `SPECS.md`, `NOTES.md`, `TESTS.md`,
`BENCHMARKS.md`, and `CLAUDE.md` files before judging behavior.

Create `.bob/state` if needed. Do not write `.bob/review` files in this mode.

## Review method

Inspect the scoped files and verify behavior against the code and applicable specs.
Use focused searches and tests where useful, but do not change files. Check each
of these areas:

1. Spec and contract drift: documented invariants, API promises, error behavior,
   compatibility, and missing updates.
2. Comment accuracy: stale claims, false performance or ownership statements,
   dead TODOs, and misleading annotations.
3. Memory and panic safety: nil dereferences, bounds errors, overflow, resource
   leaks, unsafe pool or buffer lifetime, and unexpected panic paths.
4. Concurrency: races, unsynchronized shared state, goroutine leaks, unbounded
   work, cancellation, ordering, and swallowed errors.
5. Tests and API contracts: untested branches, weak assertions, invalid inputs,
   compatibility gaps, and tests that can pass while behavior is broken.
6. Logic and edge cases: empty input, duplicates, retries, partial failure,
   cleanup, state transitions, and error propagation.
7. Code quality: needless complexity, magic values, duplicated logic, unclear
   ownership, and non-idiomatic implementation that can hide defects.
8. Architecture: unnecessary abstractions, incorrect layering, coupling, and
   structural changes that increase maintenance risk.

Prioritize findings that are actionable and demonstrable. Do not report style-only
preferences unless they create a meaningful correctness or maintenance risk. For
each finding, verify the exact file and line and explain the failure mode.

## Report

Write `.bob/state/review.md` with this structure:

```markdown
# Adversarial Review

**Mode:** single reviewer (no subagents)
**Scope:** [DIFF or full repository]
**Reviewed:** [commit or range]

## Summary
[one-paragraph assessment]

## Findings

### [SEVERITY] [short title]
- **Location:** `path/to/file:line`
- **Problem:** [what is wrong]
- **Impact:** [why it matters]
- **Evidence:** [specific code path or input]
- **Fix direction:** [concise remediation]

## Routing
- **Recommendation:** [PASS, EXECUTE, or BRAINSTORM]
- **Reason:** [highest-severity finding or why no fixes are needed]
```

Use severity levels `CRITICAL`, `HIGH`, `MEDIUM`, and `LOW`. Sort findings from
highest to lowest severity. If no findings are verified, say so explicitly and
recommend `PASS`; do not invent issues to satisfy the adversarial premise.

If `TEST` or `test` is supplied, identify test cases that would reproduce every
feasible CRITICAL or HIGH finding, but do not create tests automatically in this
single-reviewer mode.
