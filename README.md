# Claude Habitat

An agent transportation protocol for Claude Code.

## What this is

The Claude model is infrastructure — identical on every machine. What makes it *your* Claude is the user-authored configuration: commands define what it can do, agents define its roles, skills define its capabilities, CLAUDE.md defines its behavioral instructions, settings.json defines its permission boundaries. These five artifacts are the agent's portable state — the differential that separates one user's Claude from another's.

Habitat is the protocol that serialises this portable state into a verified, git-tracked slot and deserialises it on any other machine. The invariant: what you save is exactly what you load, byte-for-byte, verified at every stage.

## Protocol

```
~/.claude/  ──save──▶  slots/<name>/  ──git──▶  [remote]  ──git──▶  slots/<name>/  ──load──▶  ~/.claude/
  (register)            (serialised)                                  (serialised)              (register)
```

Three properties make this a protocol, not a script:

1. **Format** — the slot structure (META manifest + allowlisted artifacts) is a defined serialisation format
2. **Procedure** — save/load/sync are defined state-transfer operations with atomicity guarantees
3. **Integrity** — SHA-256 verification chain at every stage: save verifies the slot, load verifies the destination round-trip, sync verifies the remote

## What it syncs

| Artifact | Path | Type |
|---|---|---|
| Custom commands | `~/.claude/commands/` | Recursive `*.md` tree (supports subdirs) |
| Agents | `~/.claude/agents/` | Recursive `*.md` tree |
| Skills | `~/.claude/skills/` | Recursive `*.md` tree |
| Settings | `~/.claude/settings.json` | User-level permissions and rules |
| Global instructions | `~/.claude/CLAUDE.md` | User-level Claude instructions |
| Plugin registry | `~/.claude/plugins/installed_plugins.json` | Installed plugin versions |
| Marketplace list | `~/.claude/plugins/known_marketplaces.json` | Known plugin sources |
| Plugin blocklist | `~/.claude/plugins/blocklist.json` | Blocked plugins |

Credentials, local settings, project configs, plugin caches, marketplace clones, and history are **never touched**.

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
  scan     plugins/installed_plugins.json     1651B
  copy     → .tmp.a1b2                      4 files
  meta     sha256 manifest                    written
  swap     → mysetup                        atomic
  card     CARD.md                            written
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
  CARD.md           # human-readable save card with fingerprint (derived from META)
  commands/          # recursive *.md tree
  agents/            # if present in source
  skills/            # if present in source
  plugins/           # installed_plugins.json, known_marketplaces.json, blocklist.json
  settings.json      # if present in source
  CLAUDE.md          # if present in source
```

## Save card (CARD.md)

Each save generates a `CARD.md` — a human-readable identity card for the slot. It includes:

- **Header table**: slot name, fingerprint, timestamp, hostname, OS, habitat version
- **Contents inventory**: what skills, agents, plugins, commands are in the slot
- **Fingerprint chain**: SHA-256 of the META file (since META contains checksums of every file, this single hash uniquely identifies the entire slot state)

CARD.md is excluded from the META manifest — it's derived metadata, not source data. The integrity chain: files → META (checksums) → CARD (human summary + fingerprint).

## Safety

- **Allowlist-only**: only the 8 artifact types above are ever read or written
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
