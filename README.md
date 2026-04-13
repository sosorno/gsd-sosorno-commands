# gsd-sosorno-commands

Custom commands for [Get Shit Done (GSD)](https://github.com/gsd-build/get-shit-done), a spec-driven development system for Claude Code.

These commands extend the GSD workflow with two session-hygiene tools that aren't in upstream (yet):

| Command | Purpose |
|---|---|
| `/gsd:close-session` | Session close-out checklist. Verifies `STATE.md` accuracy, captures deferred scope, ensures operational steps are tracked as todos, updates memory, and checks the ROADMAP progress table against disk before you exit. Prevents context loss between sessions. |
| `/gsd:progress-forensic` | Deep integrity audit layered on top of `/gsd:progress`. Catches false completions (artifact count says done but `STATE.md` says mid-work), orphaned handoff files, deferred scope missing from ROADMAP, and pending operational steps. Use when resuming mid-flight work. |

The two commands are complementary: **close-session is the preventive** (run before you leave), **progress-forensic is the detective** (run when you return and suspect the mechanical progress count is lying).

## Install

Assumes you already have GSD installed globally via `npx get-shit-done-cc --claude --global`.

```bash
git clone https://github.com/sosorno/gsd-sosorno-commands.git /tmp/gsd-sosorno-commands
cp /tmp/gsd-sosorno-commands/commands/gsd/*.md   ~/.claude/commands/gsd/
cp /tmp/gsd-sosorno-commands/workflows/*.md      ~/.claude/get-shit-done/workflows/
rm -rf /tmp/gsd-sosorno-commands
```

Restart Claude Code and the commands will appear in the slash-command list.

## Re-apply after `/gsd:update`

`/gsd:update` performs a clean install of `commands/gsd/` and `get-shit-done/`, which wipes any files it didn't ship. Because these two commands are **new files** (not modifications of shipped files), GSD's auto-backup to `gsd-local-patches/` may not catch them. After every `/gsd:update`, re-run the install steps above to restore them.

Once these commands land upstream in GSD, this repo becomes obsolete.

## Status

Proposed upstream to `gsd-build/get-shit-done` — PR link will be added here once opened.

## Acknowledgments

Built on top of [get-shit-done](https://github.com/gsd-build/get-shit-done) by TÂCHES. GSD is MIT-licensed; this repo extends it under the same terms.

## License

MIT — see [LICENSE](LICENSE).
