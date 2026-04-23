<purpose>
Deep integrity audit layered on top of the standard progress report.

Runs richer versions of the 6 checks found in upstream `/gsd-progress --forensic`:
reads handoff-file bodies for context, matches decimal phase numbers like `72.1`,
cross-references the user's Claude auto-memory (not just `.planning/MEMORY.md`),
performs semantic classification on pending todos, and checks `git stash list`
alongside working-tree status.

Use when resuming after leaving work mid-flight, or when you suspect the mechanical
PLAN/SUMMARY count doesn't reflect true project state.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

<step name="run_standard_progress">
**First, run the standard progress workflow end-to-end.**

Execute every step from `@$HOME/.claude/get-shit-done/workflows/progress.md` — init_context
through route — and present the standard progress report with routing suggestion.

Do NOT pass `--forensic` to the underlying progress workflow; this deep audit supersedes
upstream's lighter variant. Do NOT skip the standard report either — the deep layer
supplements it, it does not replace it.

After the standard report and routing suggestion have been presented, continue to the
deep steps below.
</step>

<step name="deep_header">
Present:

```
---

## Deep Integrity Audit

Running 6 deep checks against project state...
```
</step>

<step name="check_state_integrity">
**Check 1 — STATE.md vs artifact completion**

Read STATE.md and extract:
- `stopped_at` field (frontmatter)
- `status` field (frontmatter)
- First non-frontmatter paragraph body (often begins with `**Last updated:**` or `Session Continuity` — capture whichever exists)

Search all three for the keywords: `pending`, `awaiting`, `in-progress`, `in progress`, `blocked`, `mid-flight`, `not yet`, `pendiente` (project may use Spanish).

If ANY of the three contains one of these keywords AND the standard progress report marks the current phase as "complete" or routes to the next phase, flag:

```
⚠ CHECK 1 FAILED — State says work pending, artifacts say complete
  STATE.md stopped_at: "{stopped_at value}"
  STATE.md status:     "{status value}"
  Body hint:           "{first matching body line, truncated to 100 chars}"
  Artifact count:      {X}/{Y} plans complete in current phase
  → The phase may have been marked complete prematurely
```

If state and artifacts agree, report:
```
✓ Check 1 — STATE.md consistent with artifact count
```
</step>

<step name="check_handoff_files">
**Check 2 — Orphaned handoff files**

Scan for handoff artifacts that indicate interrupted work:

```bash
ls .planning/HANDOFF.json 2>/dev/null || true
ls .planning/continue-here.md 2>/dev/null || true
ls .planning/phases/*/.continue-here.md 2>/dev/null || true
find .planning/phases/ -type f \( -iname "*HANDOFF*" -o -iname "*handoff*" \) 2>/dev/null || true
```

For each found file, read the first 10 lines to extract context — look for phrases like
"You are here", "Stopped at", "Resume from", or the first heading.

If ANY handoff files exist:
```
⚠ CHECK 2 FAILED — Orphaned handoff files found
  {path}: {first-line summary or resume-point extract}
  → Work was paused mid-flight. Read the handoff before continuing.
```

If none found:
```
✓ Check 2 — No orphaned handoff files
```
</step>

<step name="check_deferred_scope">
**Check 3 — Deferred scope not in ROADMAP**

Scan phase artifacts for deferred future phases (match both whole-integer AND decimal phase numbers like `72.1`):

```bash
grep -rnE "defer.*[Pp]hase|future [Pp]hase|[Pp]hase [0-9]+(\.[0-9]+)? *(is *)?deferred|[Pp]hase [0-9]+(\.[0-9]+)? *(is *)?NOT.*scope|out of.*scope.*[Pp]hase|deferred to [Pp]hase" \
  .planning/phases/*/*-BUG-BRIEF.md \
  .planning/phases/*/*-DISCUSSION-LOG.md \
  .planning/phases/*/*-CONTEXT.md \
  .planning/phases/*/*-HANDOFF.md \
  .planning/phases/*/*-VERIFICATION.md \
  .planning/phases/*/*-SUMMARY.md \
  2>/dev/null | head -30 || true
```

For each deferred phase number found (including decimals), check if it exists in ROADMAP.md:

```bash
grep -cE "Phase {deferred_phase_number}( |:|\.|$)" .planning/ROADMAP.md 2>/dev/null || echo "0"
```

If any deferred phase is NOT in ROADMAP:
```
⚠ CHECK 3 FAILED — Deferred scope not captured in ROADMAP
  "{source_file}": mentions Phase {X} deferred — NOT in ROADMAP.md
  → Insert via /gsd-insert-phase {X} ${GSD_WS} or /gsd-add-phase ${GSD_WS} before it gets lost
```

If all deferred phases are in ROADMAP (or none found):
```
✓ Check 3 — All deferred scope captured in ROADMAP (or none found)
```
</step>

<step name="check_memory_cross_ref">
**Check 4 — Claude auto-memory flagging pending work**

Locate the user's Claude auto-memory for the current project. Claude Code stores
per-project memory at `~/.claude/projects/<project-slug>/memory/MEMORY.md`, where
`<project-slug>` is the absolute CWD with `:` stripped and `\`/`/` replaced by `-`.

```bash
SLUG=$(pwd | sed -e 's|:||g' -e 's|[/\\]|-|g')
MEMORY_INDEX="$HOME/.claude/projects/$SLUG/memory/MEMORY.md"
ls "$MEMORY_INDEX" 2>/dev/null || true
```

Also check the project-local planning memory as a fallback (may exist in older projects):
```bash
ls .planning/MEMORY.md .planning/memory/*.md 2>/dev/null || true
```

Read whichever index exists. Scan titles and descriptions for the keywords:
`pending`, `status`, `pause`, `deferred`, `NOT YET RUN`, `needs`, `backfill`, `blocking`,
`pendiente`, or any current milestone phase number.

For each matching entry, READ the linked memory file and assess whether it describes:
- Work that is still pending (not completed)
- Work that contradicts the standard progress report's routing
- Operational steps (scripts, credentials, manual verification) not captured as todos

If any memory entry flags unfinished or contradictory work:
```
⚠ CHECK 4 FAILED — Memory entries indicate pending or contradictory work
  {memory_file}: "{one-line finding from the memory body}"
  → Verify this work is done or captured as a todo/phase; update or delete the memory
```

If no actionable entries (or no memory index exists):
```
✓ Check 4 — No memory entries flagging pending work
```
</step>

<step name="check_blocking_todos">
**Check 5 — Blocking operational todos**

```bash
ls .planning/todos/pending/*.md 2>/dev/null || true
```

For each pending todo, READ the file body (not just filename) and classify whether it describes:
- **Scripts to run** (backfill, migration, seed, reconciliation)
- **Credentials / API keys** to configure
- **Manual verification** steps (UAT, QA sign-off, spot-check)
- **External actions** (managed service dashboards, third-party setup, RPC provider accounts, DNS, SSL)

These are operational blockers that filename-grep alone would miss — the semantic intent
lives in the body.

If blocking operational todos exist, list each with its classification:
```
⚠ CHECK 5 FAILED — Operational todos pending
  {todo_file}: [{classification}] "{one-line body extract}"
  → These require human action before the phase/milestone is truly complete
```

If no blocking todos (or only idea/thought-type todos):
```
✓ Check 5 — No blocking operational todos
```
</step>

<step name="check_uncommitted_work">
**Check 6 — Uncommitted changes and stashed WIP**

```bash
git status --short 2>/dev/null | grep -v "^?? \.planning/" | grep -v "^ M \.planning/" | grep -v "^M  \.planning/" | head -20
git stash list 2>/dev/null | head -5
```

Two sub-signals:
- Uncommitted changes outside `.planning/` (source code, scripts, tests, config)
- Stashed WIP (any stash entry — stashes are work that was explicitly parked)

If EITHER is non-empty:
```
⚠ CHECK 6 FAILED — Uncommitted code or stashed WIP
  Working tree: {file list if any, up to 10 lines}
  Stash:        {stash list if any}
  → Review — commit, stash message, or discard as appropriate
```

If working tree is clean (or only has `.planning/` changes) and no stashes:
```
✓ Check 6 — Working tree clean and no stashed WIP
```
</step>

<step name="deep_verdict">
**Present deep verdict**

Count failures from checks 1-6.

If 0 failures:
```
---

### Verdict: CLEAN

All 6 deep checks passed. The standard progress report is trustworthy.
Proceed with the routing suggestion above.
```

If 1+ failures:
```
---

### Verdict: {N} INTEGRITY ISSUE(S) FOUND

The standard progress report may not reflect true project state.
Review the flagged items above before proceeding with the routing suggestion.

**Suggested actions:**
- [list specific actions based on which checks failed]
```

Suggested actions mapping:
- Check 1 fail → "Read STATE.md `stopped_at` context and run `/gsd-verify-work {phase} ${GSD_WS}` to reconcile"
- Check 2 fail → "Read the handoff file(s) and resume with `/gsd-resume-work ${GSD_WS}`"
- Check 3 fail → "Insert deferred phases via `/gsd-insert-phase {N} ${GSD_WS}` or `/gsd-add-phase ${GSD_WS}`"
- Check 4 fail → "Verify memory-flagged work is done; update or delete stale memory entries"
- Check 5 fail → "Complete the operational steps in `.planning/todos/pending/` before marking phase/milestone done"
- Check 6 fail → "Review the working tree — commit, stash with a message, or discard"
</step>

</process>

<success_criteria>
- [ ] Standard progress report presented first (complete, with routing)
- [ ] All 6 deep checks executed with clear pass/fail
- [ ] Each failure includes the specific evidence and a concrete action
- [ ] Verdict summarizes overall integrity state
- [ ] User can trust the report to make an informed decision about what to do next
</success_criteria>
