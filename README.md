# Claude Habitat

The portable habitat for Claude Code — user side.

`~/.claude/` is the live register file. `slots/<name>/` is persistent storage. The `habitat` script is a store/load unit with a hardcoded allowlist: it moves exactly the user-authored, portable subset of Claude's configuration between machines via git, and nothing else. Every transfer is SHA-256 verified end-to-end.

## Model

Claude Code stores user-level configuration in `~/.claude/`. This directory mixes portable, user-authored artifacts (commands, agents, skills, instructions, permissions) with machine-local state (credentials, caches, session history, project bindings). Habitat isolates the portable subset, serialises it into a git-tracked slot with a SHA-256 manifest, and deserialises it on any other machine. The invariant: what you save is exactly what you load, byte-for-byte, verified at every stage.

## What it syncs

| Artifact | Path | Type |
|---|---|---|
| Custom commands | `~/.claude/commands/` | Recursive `*.md` tree (supports subdirs) |
| Agents | `~/.claude/agents/` | Recursive `*.md` tree |
| Skills | `~/.claude/skills/` | Recursive `*.md` tree |
| Settings | `~/.claude/settings.json` | User-level permissions and rules |
| Global instructions | `~/.claude/CLAUDE.md` | User-level Claude instructions |

Credentials, local settings, project configs, caches, and history are **never touched**.

## Setup

```bash
git clone git@github.com:JianghanZHang/Claude-Habitat.git
cd Claude-Habitat
chmod +x habitat
```

## Usage

```bash
# Save config to a slot (scan → copy → swap → verify)
./habitat save mysetup

# Load from slot (check → backup → write → round-trip verify)
./habitat load mysetup

# Check slot integrity against sha256 manifest
./habitat verify mysetup

# Save + verify + git push + verify remote (one command)
./habitat sync mysetup

# List available slots
./habitat list
```

Every operation prints a compiler-style verification chain:

```
habitat · save mysetup
  scan     commands/                          3 md
  scan     settings.json                      412B
  copy     → .tmp.a1b2                      4 files
  meta     sha256 manifest                    written
  swap     → mysetup                        atomic
  check    commands/test.md                   dd06a354
  check    settings.json                      a1b2c3d4
  ────────────────────────────────────────────────
  PASS  mysetup  4 files  sha256 verified
```

## Cross-machine sync

```bash
# Machine A: save, verify, and push (all in one)
./habitat sync work

# Machine B: pull, verify, and load
git pull
./habitat load work
```

## Slot structure

```
slots/<name>/
  META              # version, timestamp, hostname, os, sha256 manifest
  commands/          # recursive *.md tree
  agents/            # if present in source
  skills/            # if present in source
  settings.json      # if present in source
  CLAUDE.md          # if present in source
```

## Safety

- **Allowlist-only**: only the 5 artifact types above are ever read or written
- **Atomic save**: writes to temp dir, then renames (no partial state on failure)
- **Auto-backup on load**: current config is saved to `_backup_<timestamp>` before any load overwrites
- **Symlink rejection**: hard error if symlinks found anywhere in source or slot
- **No credential leak**: dir artifacts filter `*.md` only; `settings.json` contains no secrets by design
- **Additive load**: loading a slot only writes what the slot contains; missing categories are not wiped
- **Input sanitization**: slot names restricted to `[a-zA-Z0-9_-]`
- **Verification chain**: save verifies slot checksums, load verifies destination round-trip, sync verifies remote META match — nothing can be silently lost

## Config

Set `CLAUDE_HOME` to override the default `~/.claude` path:

```bash
CLAUDE_HOME=/tmp/test-claude ./habitat save test
```
