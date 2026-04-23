---
name: gsd:progress-deep
description: Deep integrity audit layered on top of /gsd:progress — richer than upstream --forensic. Reads handoff bodies for context, matches decimal phase numbers, cross-references the user's Claude auto-memory, classifies pending todos semantically, and checks git stash list.
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
---

<objective>
Run the standard progress report, then layer a deep integrity audit on top — a
deeper variant of upstream's `/gsd-progress --forensic`.

Use when resuming after leaving work mid-flight (UAT, backfill, unclosed
phase/milestone), or any time you suspect the artifact-based progress count
doesn't reflect true project state and want more than a keyword-grep pass.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/progress-deep.md
</execution_context>

<process>
Execute the progress-deep workflow from @$HOME/.claude/get-shit-done/workflows/progress-deep.md end-to-end.
This runs the standard progress workflow first (all routes A-F), then appends the 6-check deep audit.
</process>
