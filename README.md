# Belayin' Pin Bob

```
                                     |    |    |
                                    )_)  )_)  )_)
                                   )___))___))___)\
                                  )____)____)_____)\\
                                _____|____|____|____\\\__
                       ---------\                   /---------
                         ^^^^^ ^^^^^^^^^^^^^^^^^^^^^
                           ^^^^      ^^^^     ^^^    ^^
                                ^^^^      ^^^
```

Workflow orchestration for Claude Code through skills and subagents.

## What is Bob?

Bob coordinates AI agent workflows for feature development. Skills invoke specialized subagents, pass state through `.bob/` artifacts, and enforce quality gates automatically.

## Spec-Driven Development

Bob treats **SPECS.md as the source of truth** for module behavior. Every workflow is spec-aware:

- **`/bob:work`** reads existing specs before making changes. If a request contradicts a contract or invariant in SPECS.md, the workflow will question it — specs can be changed, but only deliberately. Code changes to spec-driven modules must be reflected in the corresponding spec docs.

- **`/bob:explore`** prioritizes spec docs when analyzing a codebase. For spec-driven modules, it reads SPECS.md and NOTES.md first to understand contracts and design decisions before diving into implementation code. Uses concurrent analysis and adversarial challenge phases for deep, reliable exploration.

A spec-driven module is any directory containing SPECS.md, NOTES.md, TESTS.md, BENCHMARKS.md, or `.go` files with this comment:

```go
// NOTE: Any changes to this file must be reflected in the corresponding SPECS.md or NOTES.md.
```

## Quick Start

```bash
git clone https://github.com/mattdurham/bob.git
cd bob
make install
```

This installs workflow skills to `~/.claude/skills/` and subagents to `~/.claude/agents/`. Restart Claude Code after installation.

## Workflows

### `/bob:work` — Concurrent Agent Team Workflow

```
INIT → WORKTREE → BRAINSTORM → PLAN → SPAWN TEAM → EXECUTE ↔ REVIEW → COMMIT → MONITOR → COMPLETE
```

Multiple coder and reviewer teammates work in parallel through a shared task list. Requires `CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS=1`.

### `/bob:audit` — Spec Audit

```
INIT → DISCOVER → AUDIT → REPORT → COMPLETE
```

Verify code satisfies stated invariants in spec-driven modules. Read-only — reports drift but doesn't fix it.

### `/bob:explore` — Team-Based Exploration with Adversarial Challenge

```
INIT → DISCOVER → ANALYZE (4 agents) → CHALLENGE (5 agents) → DOCUMENT → COMPLETE
                     ↑                       ↓
                     └───────────────────────┘
                          (any FAIL, max 2 loops)
```

Concurrent specialist agents for codebase exploration. ANALYZE spawns 4 agents (structure, flow, patterns, dependencies). CHALLENGE spawns adversarial agents that stress-test the analysis. Failures loop back to re-analyze. No code changes.

## Loop-Back Rules

All work workflows enforce these routing rules:

| Trigger | Route to | Reason |
|---------|----------|--------|
| CRITICAL/HIGH review issues | BRAINSTORM | Re-think the approach |
| MEDIUM/LOW review issues | EXECUTE | Targeted fixes |
| Test failures | EXECUTE | Fix the code |
| CI failures or PR feedback | BRAINSTORM | Always re-brainstorm |

REVIEW is mandatory — it cannot be skipped even if tests pass.

## Subagents

| Agent | Phase | Purpose |
|-------|-------|---------|
| workflow-brainstormer | BRAINSTORM | Research and creative ideation |
| workflow-planner | PLAN | Implementation planning |
| workflow-coder | EXECUTE | Code implementation (TDD) |
| workflow-implementer | EXECUTE | Used by workflow-coder and design |
| workflow-tester | TEST | Test execution and quality checks |
| review-consolidator | REVIEW | Multi-domain code review |
| commit-agent | COMMIT | Git operations and PR creation |
| monitor-agent | MONITOR | CI/CD and PR monitoring |
| team-coder | EXECUTE | Concurrent coder teammate |
| team-reviewer | REVIEW | Concurrent reviewer teammate |
| Explore | DISCOVER | Codebase exploration |

## Pre-Push Gate Hook

A repo can gate bob's publication step by committing an executable script at `.bob/hooks/pre-push`. commit-agent runs it from the repository top level immediately before `git push`/`gh pr create`: exit 0 lets publication proceed (after a post-hook check that the branch and HEAD the hook ran against are still exactly what is being pushed); any nonzero exit — or a hook that is present but broken (non-executable, or a dangling symlink), times out, or yields no readable result — blocks both the push and the PR, with the script's output captured in `.bob/state/commit.md`. No hook file means no change to today's behavior. Use it for size limits, scope checks, secret scans — anything you want enforced before code leaves the machine.

To commit the hook in a repo that ignores `.bob` paths, make sure `.bob/hooks/pre-push` itself isn't ignored. Gitignore semantics: a file cannot be re-included while any parent directory of it is still excluded — git doesn't descend into excluded directories, so a bare `!.bob/hooks/pre-push` (or `!.bob/hooks/`) line is silently ineffective when `.bob/` or `.bob` is ignored. Walk the re-include down level by level — re-include the directory, re-ignore its children, then re-include the next level down:

```
!.bob/
.bob/*
!.bob/hooks/
.bob/hooks/*
!.bob/hooks/pre-push
```

This works whether the repo ignored `.bob/`, `.bob`, or only `.bob/*` (this repo's own `.gitignore` ignores `.bob/*`, so it can skip the first two lines), and keeps everything else under `.bob/` — including `.bob/state/` — ignored. Verify with `git check-ignore -q .bob/hooks/pre-push`: exit status `1` (and no output) means the hook is trackable; exit `0` means it is still ignored.

### Recovering after a block

- **The hook's outcome changed without touching the repo** (external condition cleared, hook config outside the repo): just rerun — the resume check finds the recorded state intact and skips commit creation. With confirm mode off (and no `REAPPROVAL_REQUIRED: yes` in the marker) the rerun skips straight to the publish step, the gate re-runs, and if the hook now passes the existing blocked commit is published with no duplicate commit; under confirm mode it routes through preview and fresh approval first (see the confirm-before-push bullet below), then the gate re-runs at publish.
- **You fixed the code or the hook script (tracked files):** a rerun with the fix uncommitted is correctly refused — the resume check pins the blocked publication to a clean tree on the recorded branch and HEAD, so a dirty tree is a state-drift STOP, never a silent commit on top of a blocked publication. The block marker is durable by design, so the recovery is to acknowledge it: remove the stale marker (`rm .bob/state/commit.md` — bob-owned working state, never committed), keep your fix uncommitted, and rerun. Before removing it, check the marker for `REAPPROVAL_REQUIRED: yes` — that line records a consumed push approval, so if it is present keep confirm mode on (or re-enable it) for the rerun, which then demands a fresh approval anyway; only a marker without that field is safe to plain-remove with confirm mode off. The flow then commits the fix on top of the blocked commit and the gate re-runs at publish, covering both commits; under confirm-before-push the same rerun first shows a fresh preview and waits for a fresh approval. (With confirm mode off, don't pre-commit the fix: with everything already committed a rerun has nothing to commit and stops. Under confirm mode a pre-committed fix is fine once the stale marker is removed — the rerun's existing-HEAD transition accepts a clean unpushed commit and regenerates the preview.)
- **Blocked under confirm-before-push:** the block consumed the push approval and recorded `REAPPROVAL_REQUIRED: yes` in the block marker, so the rerun reuses the existing HEAD, regenerates the preview, and waits for a fresh approval before the hook re-runs — a pre-block approval never becomes a silent push.
- **Known limitation:** a `HOOK_STATE_DRIFT` block (branch or HEAD changed across the gate — a mutating hook, or HEAD already detached at the snapshot) is not resumable — the record keeps the expected and actual `branch@sha` rather than a resumable publication point, and drift never routes through the resume check. Inspect what the hook did and restore the intended branch and HEAD. Under confirm mode, remove the drift marker and rerun: the confirm flow's existing-HEAD (PREPARE) transition regenerates the preview and publishes on a fresh approval. Without confirm mode there is no automatic republication path for an already-committed HEAD — the restored state leaves nothing to commit, so a rerun stops.

## Git Worktrees

All work workflows create isolated git worktrees before any file operations:

```
repo/
repo-worktrees/
  ├── add-auth/          # Feature worktree
  │   ├── .bob/state/    # Workflow artifacts
  │   └── ...
  └── fix-parser/
      ├── .bob/state/
      └── ...
```

## Installation

```bash
make install                # Everything (skills + agents + LSP)
make install-skills         # Skills only
make install-agents         # Subagents only
make enable-agent-teams     # Enable /bob:work
make hooks                  # Optional: pre-commit quality checks
```

## Requirements

- Claude Code CLI
- Git

Optional: Go, golangci-lint, gocyclo (for Go-specific features)

---

*Bob - Captain of Your Agents*
