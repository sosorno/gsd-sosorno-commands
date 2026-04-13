<purpose>
Session close-out checklist. Runs before /clear or exit to ensure no context is lost
between sessions. Verifies state accuracy, captures deferred scope, ensures operational
steps are tracked, and offers to create handoff artifacts if work is mid-flight.

This is the preventive counterpart to /gsd:progress-forensic (which is the detective).
Close-session prevents the bad state; forensic catches it after the fact.
</purpose>

<required_reading>
Read all files referenced by the invoking prompt's execution_context before starting.
</required_reading>

<process>

<step name="init">
**Load project context:**

```bash
INIT=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" init progress 2>&1)
if [[ "$INIT" == @file:* ]]; then INIT=$(cat "${INIT#@file:}"); fi
```

If `project_exists` is false: "No GSD project found. Nothing to close out." Exit.

Read STATE.md for current position.
</step>

<step name="header">
Present:

```
# Session Close-Out Checklist

Running 6 checks before you leave...
```
</step>

<step name="check_1_state_accuracy">
**1. STATE.md `stopped_at` accuracy**

Read STATE.md frontmatter `stopped_at` field and `status` field.

Ask yourself: based on what happened in THIS conversation, is `stopped_at` still accurate?

Scan the conversation for:
- Plans executed (commits made)
- Phases completed or started
- UAT performed
- Bugs found or fixed
- Operational steps completed or deferred

If `stopped_at` is stale (doesn't reflect current session's work):

```
⚠ CHECK 1 — STATE.md stopped_at is stale
  Current value: "{stopped_at}"
  Should reflect: "{what actually happened this session}"
  → Updating STATE.md...
```

Update STATE.md `stopped_at` in frontmatter AND the Session Continuity section to reflect
what the user was last doing and what comes next.

If accurate:
```
✓ Check 1 — STATE.md stopped_at is accurate
```
</step>

<step name="check_2_mid_flight_work">
**2. Mid-flight work detection**

Determine if work is mid-flight (not cleanly at a phase/plan boundary):

Signs of mid-flight work:
- A PLAN.md exists without a matching SUMMARY.md in the current phase
- Uncommitted code changes in `src/` or `scripts/`
- Tasks were started but not completed
- UAT was started but not all items resolved
- A script was discussed but not yet run

```bash
# Check for plans without summaries in current/recent phase
for plan in .planning/phases/*/*-PLAN.md; do
  [ -e "$plan" ] || continue
  summary="${plan/PLAN/SUMMARY}"
  [ ! -f "$summary" ] && echo "No summary: $plan"
done 2>/dev/null | tail -5 || true

# Check uncommitted code changes
git status --short -- src/ scripts/ 2>/dev/null | head -10
```

If mid-flight work detected:
```
⚠ CHECK 2 — Work is mid-flight
  {details of what's incomplete}
  → Recommend running /gsd:pause-work to create handoff artifacts
  → Or describe the stopping point below so STATE.md captures it
```

Ask the user: "Want me to run /gsd:pause-work to create a full handoff, or just update STATE.md with where you stopped?"

If at a clean boundary:
```
✓ Check 2 — Work is at a clean phase/plan boundary
```
</step>

<step name="check_3_deferred_scope">
**3. Deferred scope capture**

Scan conversation context for any decisions to defer work to future phases.
Look for patterns like:
- "defer to phase X"
- "future phase"
- "out of scope for now"
- "we'll handle that later"
- Bug reports acknowledged but not fixed
- Features discussed but not planned

For each piece of deferred scope found, check if it's already in ROADMAP.md:

```bash
grep -c "Phase {number}" .planning/ROADMAP.md 2>/dev/null || echo "0"
```

Also check if it's captured as a todo:
```bash
ls .planning/todos/pending/ 2>/dev/null || true
```

If deferred scope exists but is NOT in ROADMAP or todos:
```
⚠ CHECK 3 — Deferred scope not captured
  "{description of deferred work}" — not in ROADMAP.md or pending todos
  → Capture now?
```

Offer to:
- Insert as a phase: describe for `/gsd:add-phase` or `/gsd:insert-phase`
- Capture as a todo: write to `.planning/todos/pending/`
- Add to backlog: `/gsd:add-backlog`

If all deferred scope is captured:
```
✓ Check 3 — All deferred scope captured in ROADMAP or todos
```
</step>

<step name="check_4_operational_steps">
**4. Operational steps tracking**

Scan conversation for operational steps that were discussed but not completed:
- Scripts to run (backfill, migration, seed, verification)
- API keys or credentials to set up
- External service configuration (managed service dashboards, RPC provider accounts)
- Manual testing or UAT items
- Deployments or pushes

For each unfinished operational step, check if it's captured as a todo:

```bash
grep -rl "{keyword}" .planning/todos/pending/ 2>/dev/null || true
```

If operational steps are not tracked:
```
⚠ CHECK 4 — Untracked operational steps
  "{step description}" — not captured as a todo
  → Creating todo...
```

Create a todo file in `.planning/todos/pending/` with the operational step details.
Use the naming convention: `YYYY-MM-DD-{slug}.md`

```bash
timestamp=$(node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" current-timestamp date --raw)
```

If all operational steps are tracked:
```
✓ Check 4 — All operational steps tracked as todos
```
</step>

<step name="check_5_memory_updates">
**5. Memory updates**

Review the conversation for information worth persisting to memory:
- User corrections or feedback (→ feedback memory)
- Project decisions or context (→ project memory)
- New user preferences learned (→ user memory)
- External resource references (→ reference memory)

Also check if any existing memory entries became stale during this session:
- Phase status memories that should be updated
- "Pending" items that were completed
- Decisions that were reversed

If updates needed:
```
⚠ CHECK 5 — Memory updates recommended
  New: "{description of new memory to save}"
  Update: "{description of stale memory to refresh}"
  → Save now?
```

If the user approves, write/update the memory files and MEMORY.md index.

If no updates needed:
```
✓ Check 5 — Memory is up to date
```
</step>

<step name="check_6_roadmap_progress">
**6. ROADMAP progress table accuracy**

Check that the ROADMAP.md Progress table matches reality:
- Phase completion dates are accurate
- Plan counts reflect actual files on disk
- Status markers ([x] vs [ ]) are correct

```bash
node "$HOME/.claude/get-shit-done/bin/gsd-tools.cjs" roadmap analyze 2>&1
```

Compare the tool's disk-based analysis against what ROADMAP.md says.

If ROADMAP is out of date:
```
⚠ CHECK 6 — ROADMAP progress table is stale
  {specific discrepancy}
  → Updating ROADMAP.md...
```

Update the progress table to match reality.

If accurate:
```
✓ Check 6 — ROADMAP progress table is accurate
```
</step>

<step name="verdict">
**Present close-out verdict**

Count passes and failures.

If all 6 pass:
```
---

### Session Close-Out: CLEAN

All 6 checks passed. Safe to /clear or exit.
Context will be fully recoverable in the next session.
```

If any failures were found and fixed during this workflow:
```
---

### Session Close-Out: FIXED

{N} issues found and resolved during close-out.
- {summary of each fix}

Safe to /clear or exit now.
```

If any failures remain unresolved:
```
---

### Session Close-Out: {N} OPEN ITEM(S)

The following items could not be auto-resolved:
- {item 1 — why it needs human action}
- {item 2}

Address these before leaving, or accept the risk of context loss.
```
</step>

</process>

<success_criteria>
- [ ] All 6 checks executed
- [ ] STATE.md stopped_at reflects this session's actual ending state
- [ ] Deferred scope captured in ROADMAP or todos (nothing lost)
- [ ] Operational steps tracked as todos (scripts, API keys, manual steps)
- [ ] Memory updated with session learnings
- [ ] ROADMAP progress table matches disk reality
- [ ] User knows it's safe to exit
</success_criteria>
