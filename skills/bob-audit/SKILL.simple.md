---
name: bob:audit
description: Single-agent audit of documented invariants and codebase health.
user-invocable: true
category: workflow
---

# Bob Audit — Single Agent

Audit the repository yourself. Never call `Task`, `Agent`, `subagent`, create
teammates, or use agent teams.

Discover `CLAUDE.md`, `SPECS.md`, `NOTES.md`, `TESTS.md`, and `BENCHMARKS.md`
files plus relevant source. Compare every applicable invariant and contract with
the implementation. Check error handling, tests, concurrency, resource lifetime,
complexity, and structural health directly. Run read-only analysis or tests when
useful; do not modify source code.

Write a severity-ranked report to `.bob/state/audit.md` with file and line
locations, evidence, impact, and remediation direction. End with PASS, EXECUTE,
or BRAINSTORM and explain why. If no issue is verified, say so explicitly.
