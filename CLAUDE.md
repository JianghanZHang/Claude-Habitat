# Claude-Habitat

An agent transportation protocol for Claude Code.

The model is infrastructure. The user config (commands, agents, skills, instructions, permissions) is the portable agent state — the differential. Habitat serialises this differential into a git-tracked slot with a SHA-256 manifest, and deserialises it on any other machine. `~/.claude/` is the register file (active state), `slots/<name>/` is persistent storage, `habitat` is the sole store/load unit. Nothing outside the hardcoded allowlist is ever read or written.

## Single deliverable

All logic lives in the `habitat` bash script. No other executables.

## Allowlist (portable, user-authored) — ONLY these are ever touched

| Artifact | Path | Type |
|---|---|---|
| commands | `~/.claude/commands/` | dir tree of `*.md` |
| agents | `~/.claude/agents/` | dir tree of `*.md` |
| skills | `~/.claude/skills/` | dir tree of `*.md` |
| settings.json | `~/.claude/settings.json` | single file |
| CLAUDE.md | `~/.claude/CLAUDE.md` | single file |
| installed_plugins.json | `~/.claude/plugins/installed_plugins.json` | single file |
| known_marketplaces.json | `~/.claude/plugins/known_marketplaces.json` | single file |
| blocklist.json | `~/.claude/plugins/blocklist.json` | single file |

## NEVER touched — do not expand scope

`.credentials.json`, `settings.local.json`, `~/.claude.json`, `projects/`, `debug/`, `backups/`, `cache/`, `file-history/`, `session-env/`, `shell-snapshots/`, `plugins/cache/`, `plugins/marketplaces/`, `history.jsonl`, `plans/`

## Safety invariants

1. Whitelist-only: enumerate exactly what to copy, never glob `~/.claude/*`
2. Atomic save: write to temp dir, then `mv` (same-fs rename). Trap cleans on failure
3. Pre-load backup: `load` auto-saves current state to `_backup_<timestamp>` before overwriting
4. Symlink rejection: hard error if any symlink found in source or slot
5. Input sanitization: slot names must match `[a-zA-Z0-9_-]+`, `_backup_*` reserved for save
6. No credential leak: only `*.md` from dirs, `settings.json` has no secrets by design
7. Round-trip fidelity: `save X; load X` = identity on the portable set
8. Additive load: missing categories not wiped (loading commands-only slot keeps settings.json)
9. Recursive dir copy: subdirectories supported for namespaced commands
10. Verification chain: every save verifies slot checksums, every load verifies destination round-trip, sync verifies remote META match

## `$CLAUDE_HOME` env var

Overrides `~/.claude` for all operations. Useful for testing.

## Testing

```bash
./habitat save test1        # scan → copy → swap → verify checksums
./habitat verify test1      # standalone integrity check
./habitat load test1        # check → backup → write → round-trip verify
./habitat sync test1        # save + verify + git push + remote verify
./habitat list              # metadata display
./habitat save "../bad"     # must fail: invalid name
```
