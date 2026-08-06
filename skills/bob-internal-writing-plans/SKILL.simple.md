---
name: bob:internal:writing-plans
description: Direct, single-agent implementation planning.
user-invocable: false
category: internal
---

# Writing Plans — Single Agent

Write the implementation plan yourself. Never call `Task`, `Agent`, `subagent`,
create teammates, or use agent teams.

Read `.bob/state/design.md`, repository guidance, and applicable specs. Write
`.bob/state/plan.md` with the goal, architecture, exact files, bite-sized steps,
complete implementation details, commands, expected verification, and rollback or
risk notes. Include spec-document updates where required. Do not implement code
while planning.
