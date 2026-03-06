# Claude Habitat

Save and load your portable Claude Code configuration across machines. Like game save slots.

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
# Save current config to a named slot
./habitat save mysetup

# Load config from a slot (auto-backs up current state first)
./habitat load mysetup

# List available slots
./habitat list
```

## Cross-machine sync

```bash
# Machine A: save and push
./habitat save work
git add -A && git commit -m "save work" && git push

# Machine B: pull and load
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

## Config

Set `CLAUDE_HOME` to override the default `~/.claude` path:

```bash
CLAUDE_HOME=/tmp/test-claude ./habitat save test
```
