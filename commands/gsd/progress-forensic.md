---
name: gsd:progress-forensic
description: Deep integrity audit of project state — catches false completions, orphaned handoffs, deferred scope not in roadmap, and pending operational steps
allowed-tools:
  - Read
  - Bash
  - Grep
  - Glob
  - SlashCommand
---
<objective>
Run the standard progress report, then layer a forensic integrity audit on top.
Use this when resuming after leaving work mid-flight (UAT, backfill, unclosed phase/milestone),
or any time you suspect the artifact-based progress count doesn't reflect true project state.
</objective>

<execution_context>
@$HOME/.claude/get-shit-done/workflows/progress-forensic.md
</execution_context>

<process>
Execute the progress-forensic workflow from @$HOME/.claude/get-shit-done/workflows/progress-forensic.md end-to-end.
This runs the standard progress workflow first (all routes A-F), then appends the forensic layer.
</process>
