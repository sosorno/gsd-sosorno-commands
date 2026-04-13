---
name: gsd:close-session
description: Session close checklist — verify STATE.md, capture deferred scope, ensure operational steps are tracked, update memory before leaving
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - Edit
  - Write
  - SlashCommand
---
<objective>
Run a structured close-out checklist before ending a session. Ensures no context is lost
between sessions by verifying: STATE.md accuracy, deferred scope captured in ROADMAP,
operational steps tracked as todos, handoff artifacts created if mid-work, and memory updated.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/close-session.md
</execution_context>

<process>
Execute the close-session workflow from @$HOME/.claude/get-shit-done/workflows/close-session.md end-to-end.
Present checklist results and fix any gaps before the user exits.
</process>
