<purpose>
Deep integrity audit layered on top of the standard progress report.
Catches: false completions (artifact count says done but STATE.md says mid-work),
orphaned handoff files, deferred scope not yet in ROADMAP, pending operational steps
(scripts to run, API keys to set up), and stale memory entries that contradict current state.

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

Do NOT skip the standard report. The forensic layer supplements it; it does not replace it.

After the standard report is presented, continue to the forensic steps below.
</step>

<step name="forensic_header">
Present:

```
---

## Forensic Integrity Audit

Running 6 deep checks against project state...
```
</step>

<step name="check_state_integrity">
**Check 1 — STATE.md vs artifact completion**

Read STATE.md and extract:
- `stopped_at` field (frontmatter)
- `status` field (frontmatter)
- Session Continuity section → `Stopped at:` line

If ANY of these mention pending/awaiting/in-progress/blocked work BUT the standard progress
report shows the current phase as "complete" or all phases done, flag:

```
⚠ CHECK 1 FAILED — State says work pending, artifacts say complete
  STATE.md stopped_at: "{stopped_at value}"
  STATE.md status: "{status value}"
  Artifact count: {X}/{Y} plans complete
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
# Global handoff (from /gsd:pause-work)
ls .planning/HANDOFF.json 2>/dev/null || true

# Per-phase continue-here files
ls .planning/phases/*/.continue-here.md 2>/dev/null || true

# Per-phase handoff docs (custom naming, e.g. PHASE-HANDOFF.md)
find .planning/phases/ -name "*HANDOFF*" -o -name "*handoff*" 2>/dev/null || true
```

For each found file, read the first 10 lines to extract context.

If ANY handoff files exist:
```
⚠ CHECK 2 FAILED — Orphaned handoff files found
  {path}: {first-line summary or "You are here" extract}
  → Work was paused mid-flight. Read the handoff before continuing.
```

If none found:
```
✓ Check 2 — No orphaned handoff files
```
</step>

<step name="check_deferred_scope">
**Check 3 — Deferred scope not in ROADMAP**

Scan phase artifacts for mentions of deferred future phases:

```bash
# Search for "defer to phase X", "future phase X", "Phase X.Y" mentions in discussion/bug/context docs
grep -rn "defer.*[Pp]hase\|future [Pp]hase\|[Pp]hase [0-9]\+\.[0-9]\+.*deferred\|[Pp]hase [0-9]\+\.[0-9]\+.*NOT.*scope\|out of.*scope.*[Pp]hase" \
  .planning/phases/*/*-BUG-BRIEF.md \
  .planning/phases/*/*-DISCUSSION-LOG.md \
  .planning/phases/*/*-CONTEXT.md \
  .planning/phases/*/*-HANDOFF.md \
  .planning/phases/*/*-VERIFICATION.md \
  .planning/phases/*/*-SUMMARY.md \
  2>/dev/null | head -30 || true
```

For each deferred phase number found, check if it exists in ROADMAP.md:

```bash
grep -c "Phase {deferred_phase_number}" .planning/ROADMAP.md 2>/dev/null || echo "0"
```

If any deferred phase is NOT in ROADMAP:
```
⚠ CHECK 3 FAILED — Deferred scope not captured in ROADMAP
  "{source_file}": mentions Phase {X} deferred — NOT in ROADMAP.md
  → Insert the phase via /gsd:insert-phase or /gsd:add-phase before it gets lost
```

If all deferred phases are in ROADMAP (or none found):
```
✓ Check 3 — All deferred scope captured in ROADMAP (or none found)
```
</step>

<step name="check_memory_cross_ref">
**Check 4 — Memory entries flagging pending work**

Read the memory index at the user's memory directory MEMORY.md.
Identify entries whose title/description mentions: "pending", "status", "pause", "deferred",
"NOT YET RUN", "needs", "backfill", "blocking", or current milestone phases.

For each such entry, read the memory file and check if it describes work that is:
- Still pending (not completed)
- Contradicted by the artifact-based progress report
- Describing operational steps not captured as todos

If any memory entry flags unfinished work:
```
⚠ CHECK 4 FAILED — Memory entries indicate pending work
  {memory_file}: "{key finding from memory content}"
  → Verify this work is done or captured as a todo/phase
```

If no actionable entries:
```
✓ Check 4 — No memory entries flagging pending work
```
</step>

<step name="check_blocking_todos">
**Check 5 — Blocking operational todos**

```bash
ls .planning/todos/pending/ 2>/dev/null || true
```

Read each pending todo file. Flag any that describe:
- Scripts that need to be run (backfill, migration, seed)
- API keys or credentials to configure
- Manual verification steps
- External actions (managed service dashboards, third-party setup)

These are operational blockers that artifact counting can't detect.

If blocking operational todos exist:
```
⚠ CHECK 5 FAILED — Operational todos pending
  {todo_file}: "{description}"
  → These require human action before the phase/milestone is truly complete
```

If no blocking todos (or only idea-type todos):
```
✓ Check 5 — No blocking operational todos
```
</step>

<step name="check_uncommitted_work">
**Check 6 — Uncommitted changes suggesting unfinished work**

```bash
git status --short 2>/dev/null | head -20
git stash list 2>/dev/null | head -5
```

If there are uncommitted changes in `src/` or `scripts/` (not just `.planning/`):
```
⚠ CHECK 6 FAILED — Uncommitted code changes
  {file list}
  → May represent unfinished implementation work
```

If working tree is clean or only has planning docs:
```
✓ Check 6 — Working tree clean (or planning-only changes)
```
</step>

<step name="forensic_verdict">
**Present forensic verdict**

Count failures from checks 1-6.

If 0 failures:
```
---

### Verdict: CLEAN

All 6 forensic checks passed. The standard progress report is trustworthy.
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
- Check 1 fail → "Read STATE.md stopped_at context and verify the current phase is actually complete"
- Check 2 fail → "Read the handoff file(s) and resume from where work was paused"
- Check 3 fail → "Insert deferred phases into ROADMAP via /gsd:insert-phase or /gsd:add-phase"
- Check 4 fail → "Verify memory-flagged work is done; update or delete stale memory entries"
- Check 5 fail → "Complete operational steps before marking phase/milestone as done"
- Check 6 fail → "Review uncommitted changes — commit, stash, or discard"
</step>

</process>

<success_criteria>
- [ ] Standard progress report presented first (complete, with routing)
- [ ] All 6 forensic checks executed with clear pass/fail
- [ ] Each failure includes the specific evidence and a concrete action
- [ ] Verdict summarizes overall integrity state
- [ ] User can trust the report to make an informed decision about what to do next
</success_criteria>
