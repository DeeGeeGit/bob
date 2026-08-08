---
name: bob:code-review
description: Single-agent code review with optional direct fixes and verification.
user-invocable: true
category: workflow
---

# Bob Code Review — Single Agent

Review the requested change yourself. Never call `Task`, `Agent`, `subagent`,
create teammates, or use agent teams.

Inspect the diff and relevant surrounding code. Check contracts and specs, logic,
edge cases, error handling, security, concurrency, resource lifetime, tests,
comments, and maintainability. Write findings to `.bob/state/review.md`, sorted
by CRITICAL/HIGH/MEDIUM/LOW, with exact locations and evidence. If the user asked
for fixes, apply them directly, run verification, and review the resulting diff.
Do not commit or push unless explicitly requested. End with PASS, EXECUTE, or
BRAINSTORM and the reason.
