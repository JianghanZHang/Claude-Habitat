# Post-Load Smoke Test

Run a systematic check of all Claude Code capabilities after a `habitat load`. Report results as a summary table with OK/FAIL status.

## Procedure

### 1. Core Tools
Load each deferred tool via ToolSearch and run a trivial validation:
- **Bash**: `select:Bash`, then run `echo ok`
- **Read**: `select:Read`, then read this command file
- **Edit**: `select:Edit` (load only, no mutation needed)
- **Write**: `select:Write` (load only)
- **Glob**: `select:Glob`, then glob for `*.md` in the current directory
- **Grep**: `select:Grep`, then search for a known string in this file
- **Agent**: `select:Agent` (load only)
- **Skill**: `select:Skill` (load only)

Mark each OK if the tool loads without error. Mark FAIL if ToolSearch returns no match or the trivial test errors.

### 2. Skills
List all skills shown in the system config (from the Skill tool's description or system reminders). For each skill, confirm it appears in the registered skill list. Report each as OK (registered) or FAIL (missing).

### 3. Agents
List available agent/subagent types (from the Agent tool's description). Report as OK if at least `Explore` is available.

### 4. Plugins
Read `~/.claude/settings.json` and list all entries under `enabledPlugins`. For each plugin, check if any MCP tools from that plugin are visible in the available-deferred-tools list. Report as OK (has MCP tools) or WARN (enabled but no MCP tools detected) or FAIL (settings read error).

Skip external-service plugins (Gmail, Google Calendar) — mark them as SKIP.

### 5. Output
Produce a summary table like:

```
=== Load Test Results ===

Category        Item                          Status
---------       ----                          ------
Core Tools      Bash                          OK
Core Tools      Read                          OK
...
Skills          performance-time-contraction  OK
...
Agents          Explore                       OK
...
Plugins         github                        OK
Plugins         Gmail                         SKIP (external)
...

Summary: X/Y passed, Z skipped, W failed
```

If everything is OK/SKIP, end with: "Habitat load verified — all local capabilities functional."
If any FAIL, end with: "Habitat load has failures — review items marked FAIL."
