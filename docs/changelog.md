# Changelog

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
