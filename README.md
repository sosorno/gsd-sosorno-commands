# gsd-sosorno-commands

Custom commands for [Get Shit Done (GSD)](https://github.com/gsd-build/get-shit-done), a spec-driven development system for Claude Code.

These commands extend the GSD workflow with two session-hygiene tools:

| Command | Purpose |
|---|---|
| `/gsd-close-session` | Session close-out checklist. Verifies `STATE.md` accuracy, captures deferred scope, ensures operational steps are tracked as todos, updates memory, and checks the ROADMAP progress table against disk before you exit. Prevents context loss between sessions. |
| `/gsd-progress-deep` | Deep integrity audit layered on top of `/gsd-progress`. Sibling to upstream's `/gsd-progress --forensic`, but deeper: reads handoff file bodies for context, matches decimal phase numbers (e.g. `72.1`), cross-references the user's Claude auto-memory (not just `.planning/MEMORY.md`), classifies pending todos semantically, and checks `git stash list` alongside working-tree status. Use when resuming mid-flight work. |

The two commands are complementary: **close-session is the preventive** (run before you leave), **progress-deep is the detective** (run when you return and suspect the mechanical progress count — or upstream's lighter `--forensic` pass — is lying).

> **Note on invocation:** GSD v1.31.0 shipped the Claude Code 2.1.88+ skills migration, which changed slash-command invocation from `/gsd:xxx` (colon) to `/gsd-xxx` (dash) and moved commands to `~/.claude/skills/gsd-*/SKILL.md`. This repo ships both layouts — the new `skills/` tree is what you want on any current GSD install, and the `commands/` + `workflows/` tree is kept around for anyone still on v1.30.x or earlier.

## Install — GSD v1.31.0 and newer (recommended)

Assumes you already have GSD installed globally via `npx get-shit-done-cc --claude --global`.

```bash
git clone https://github.com/sosorno/gsd-sosorno-commands.git /tmp/gsd-sosorno-commands

# Copy the SKILL.md files into their own directories under ~/.claude/skills/
mkdir -p ~/.claude/skills/gsd-close-session ~/.claude/skills/gsd-progress-deep
cp /tmp/gsd-sosorno-commands/skills/gsd-close-session/SKILL.md   ~/.claude/skills/gsd-close-session/
cp /tmp/gsd-sosorno-commands/skills/gsd-progress-deep/SKILL.md   ~/.claude/skills/gsd-progress-deep/

# Copy the workflow bodies into the unchanged workflows directory
cp /tmp/gsd-sosorno-commands/workflows/close-session.md   ~/.claude/get-shit-done/workflows/
cp /tmp/gsd-sosorno-commands/workflows/progress-deep.md   ~/.claude/get-shit-done/workflows/

rm -rf /tmp/gsd-sosorno-commands
```

Restart Claude Code and `/gsd-close-session` and `/gsd-progress-deep` will appear in the slash-command list.

## Install — GSD v1.30.x and earlier (legacy)

If you're still on a pre-migration install, use the legacy `commands/gsd/` layout:

```bash
git clone https://github.com/sosorno/gsd-sosorno-commands.git /tmp/gsd-sosorno-commands
cp /tmp/gsd-sosorno-commands/commands/gsd/*.md   ~/.claude/commands/gsd/
cp /tmp/gsd-sosorno-commands/workflows/*.md      ~/.claude/get-shit-done/workflows/
rm -rf /tmp/gsd-sosorno-commands
```

Invoke as `/gsd:close-session` and `/gsd:progress-deep` (colon form).

## Re-apply after `/gsd:update`

Every `/gsd-update` performs a clean install of the GSD-managed directories, which wipes anything it didn't ship. Because these commands are **new files** (not modifications of shipped files), GSD's `gsd-local-patches/` auto-backup does not catch them. After every update, re-run the relevant install block above to restore them.

Specifically: a v1.30.x → v1.31.x+ update will auto-delete the entire legacy `~/.claude/commands/gsd/` directory, so anything you had there will be gone. The new `skills/` install paths are safer going forward but will still be wiped on every clean install until these commands land upstream.

Once these commands land upstream in GSD, this repo becomes obsolete.

## Status

A lighter variant of this audit was contributed upstream as **[#2189](https://github.com/gsd-build/get-shit-done/issues/2189) / [PR #2235](https://github.com/gsd-build/get-shit-done/pull/2235)** — an opt-in `--forensic` flag on the existing `/gsd-progress` command. Once merged, any GSD install has the keyword-grep pass available via `/gsd-progress --forensic`.

`/gsd-progress-deep` in this repo is the **deeper sibling** that was not contributed upstream because it goes beyond the approved issue scope: it reads handoff file bodies for context, matches decimal phase numbers, cross-references the user's Claude auto-memory (not just `.planning/MEMORY.md`), classifies pending todos semantically rather than by keyword grep, and checks `git stash list`. Run `/gsd-progress --forensic` for a cheap check; run `/gsd-progress-deep` when you want the full audit.

The companion `/gsd-close-session` command is kept local-only.

## Acknowledgments

Built on top of [get-shit-done](https://github.com/gsd-build/get-shit-done) by TÂCHES. GSD is MIT-licensed; this repo extends it under the same terms.

## License

MIT — see [LICENSE](LICENSE).
