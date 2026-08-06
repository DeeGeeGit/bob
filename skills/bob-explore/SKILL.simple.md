---
name: bob:explore
description: Single-agent repository exploration with adversarial validation.
user-invocable: true
category: workflow
---

# Bob Explore — Single Agent

Explore the repository directly. Never call `Task`, `Agent`, `subagent`, create
teammates, or use agent teams.

Discover structure, entry points, data flow, dependencies, conventions, tests, and
specs. Form an initial understanding, then challenge it against the source and
look for missing assumptions, stale documentation, and edge cases. Write a
grounded report to `.bob/state/exploration.md` with file locations, findings,
open questions, and confidence levels. Do not modify source code.
