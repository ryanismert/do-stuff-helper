# Changelog

## 2026-02-25 — Completed w9: Backlog & Replanning Process
Created replan skill and backlog conventions for capturing out-of-scope work and periodically incorporating it into the roadmap.

## 2026-02-24 — Completed w18: Home Server Browser Automation Setup
Installed Xvfb + VNC + Chrome + Claude in Chrome extension on the N100 with systemd services that survive reboot. Claude Code can control Chrome via --chrome flag and users can VNC in to observe or intervene.

## 2026-02-24 — Completed w5: Autonomy Model
Workers now receive capability awareness and self-unblocking instructions, using available tools (including writing ad-hoc scripts) before escalating to the user.

## 2026-02-24 — Completed w4: Decompose & Execute Architecture
Built waypoint-planner (recursive task decomposition) and waypoint-implement (parallel worker orchestration in git worktrees) skills, completing the core execution engine.

## 2026-02-24 — Completed w19: Fix Discover Skill Invocation Bug
Identified and fixed a name collision between the discover command and discover skill caused by `disable-model-invocation: true` on the command file.

## 2026-02-23 — Completed w20: Plugin Update Workflow
Created the `/publish` slash command that automates version bump, commit, push, marketplace update, and plugin reinstall.

## 2026-02-23 — Completed w3: Waypoint Design Skill
Built a skill that evaluates waypoints, scales design effort to complexity, and produces descriptions sufficient for confident decomposition.

## 2026-02-23 — Completed w21: Suggest-and-Confirm Transition in Discover
Added a suggest-and-confirm prompt at the end of the discover skill to offer transitioning directly into roadmap planning.

## 2026-02-23 — Completed w1: Roadmap Skill
Implemented a roadmap skill that reads a brief, interviews the user about priorities and sequencing, and produces an adaptive waypoint-based roadmap as JSON.

## 2026-02-23 — Completed w2: Waypoint Storage Format
Defined and documented the waypoint storage format (JSON source of truth + individual markdown design docs) and converted the roadmap to use it.

## 2026-02-23 — Roadmap created: Do-Stuff-Helper
Bootstrap roadmap created with phased waypoints covering the full pipeline from core execution to life coaching.

## 2026-02-22 — Activity defined: Do-Stuff-Helper
Expert interview completed, brief saved covering goals, scope, success criteria, and key risks for the do-stuff-helper activity.

---

## W5: Autonomy Model

### 2026-02-24 - [#2] [w5] Update organize skill with capabilities discovery
- Added Step 5b to organize skill: discovers MCP servers, CLI tools, and project scripts
- Extended canonical CLAUDE.md template with `### Capabilities` subsection
- Updated Step 8 summary to report discovered capabilities
- Files changed: skills/organize/SKILL.md

### 2026-02-24 - [#3] [w5] Update waypoint-implement worker prompt
- Added `## Capabilities` section instructing workers to read CLAUDE.md and write/execute ad-hoc scripts
- Added self-unblocking instructions to `## Handling Ambiguity` section
- Resolved merge conflict to preserve both `## Available Skills` and new `## Capabilities` sections
- Files changed: skills/waypoint-implement/SKILL.md

### 2026-02-24 - [#4] [w5] Run organize on do-stuff-helper to seed capabilities
- Discovered CLI tools: gh, node, docker, python3, npm, npx, make
- No MCP servers or project scripts detected
- Populated `### Capabilities` subsection in CLAUDE.md
- Files changed: CLAUDE.md

### 2026-02-24 - Waypoint Complete
- Workers now receive capability awareness (skills, MCP servers, CLI tools, project scripts) via CLAUDE.md and are instructed to self-unblock using available tools — including writing and executing ad-hoc code — before escalating.

## W4: Decompose & Execute Architecture

### 2026-02-24 - Waypoint Complete
- Built `waypoint-planner` skill: decomposes designed waypoints into task DAGs with dependencies, metadata, and cross-session persistence
- Built `waypoint-implement` skill: orchestrates parallel worker agents in git worktrees, handles merge, changelog, and waypoint lifecycle
- Validated end-to-end on W20 (plugin update workflow) and W19 (discover bug fix), including merge conflict resolution

## W19: Fix Discover Skill Invocation Bug

### 2026-02-24 - [#1] [w19] Remove discover command and update references
- Deleted `commands/discover.md` which had `disable-model-invocation: true` causing name collision with the discover skill
- Updated `/discover` command references in `CLAUDE.md` and `skills/organize/SKILL.md` to remove the dead reference
- Resolved merge conflict in CLAUDE.md (worktree had older version without the do-stuff-helper section)
- Files changed: commands/discover.md (deleted), CLAUDE.md, skills/organize/SKILL.md

### 2026-02-24 - Waypoint Complete
- Fixed discover skill invocation by removing the redundant command that shared its name. The `disable-model-invocation: true` flag on the command was being picked up when the Skill tool resolved `do-stuff-helper:discover`.

## W20: Plugin Update Workflow

### 2026-02-23 - [#1] [w20] Write commands/publish.md
- Created `commands/publish.md` slash command with version bump (patch/minor/major), commit, push, marketplace update, and plugin reinstall
- Assumptions: Used `git add` with specific file paths instead of `git add -A` for safety
- Files changed: commands/publish.md

### 2026-02-23 - [#2] [w20] Test publish command
- Reviewed command logic for correctness: version parsing, bump semantics, commit message format, CLI commands
- Fixed `git add -A` to stage specific files (`plugin.json`, `marketplace.json`) instead
- Files changed: commands/publish.md (minor fix)

### 2026-02-23 - Waypoint Complete
- Created the `/publish` slash command that automates the full plugin update workflow: version bump, commit, push, marketplace update, and plugin reinstall. User only needs to start a new conversation afterward.
