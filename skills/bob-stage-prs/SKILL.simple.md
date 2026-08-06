---
name: bob:stage-prs
description: Single-agent staging of a large changeset into ordered reviewable commits.
user-invocable: true
category: workflow
---

# Stage PRs — Single Agent

Inspect and stage the changeset yourself. Never call `Task`, `Agent`, `subagent`,
create teammates, or use agent teams.

Review history, branch state, dependencies, and file ownership. Propose an
ordered stack of cohesive commits in `.bob/state/stage-plan.md`, preserving
dependency order and keeping each commit reviewable. When authorized, create the
commits directly, verify each boundary, and report the resulting hashes. Never
force-push, delete branches, or open PRs without explicit authorization.
