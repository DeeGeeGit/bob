---
name: bob-work
description: Single-agent development workflow — INIT → PLAN → EXECUTE → TEST → REVIEW → COMPLETE.
user-invocable: true
category: workflow
---

# Bob Work — Single Agent

Run the development workflow yourself in the current workspace. This variant is
strictly single-agent: never call `Task`, `Agent`, `subagent`, create teammates,
or use agent teams. Do not delegate exploration, planning, implementation,
testing, review, commits, or monitoring.

1. Inspect the repository, current branch, working tree, relevant guidance, and
   applicable specs. Preserve unrelated user changes.
2. Form a concise implementation plan and record it in `.bob/state/plan.md`.
3. Implement the requested change directly, keeping the scope minimal.
4. Run the most relevant tests, linters, formatters, or verification commands.
5. Review the diff yourself for correctness, regressions, spec drift, and missing
   tests. Fix issues found and rerun verification.
6. Report changed files, verification results, remaining risks, and the final
   routing recommendation. Commit only when the user requested a commit.

Use `.bob/state/brainstorm.md`, `.bob/state/plan.md`, and
`.bob/state/test-results.md` for direct artifacts when useful. If blocked by a
missing decision or external state, explain the exact blocker instead of spawning
another agent.
