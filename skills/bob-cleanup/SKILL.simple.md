---
name: bob:cleanup
description: Single-agent code cleanup that removes complexity without adding functionality.
user-invocable: true
category: workflow
---

# Bob Cleanup — Single Agent

Perform cleanup directly in the current workspace. Never call `Task`, `Agent`,
`subagent`, create teammates, or use agent teams.

Inspect the code and its tests, identify safe simplifications, and record a brief
plan in `.bob/state/plan.md`. Make only behavior-preserving changes: remove dead
code, reduce needless abstraction, clarify names and comments, and update stale
documentation. Preserve public contracts and user changes. Run formatting, tests,
and relevant static checks, then review the final diff yourself. Report changes,
verification, and any cleanup intentionally left for later.
