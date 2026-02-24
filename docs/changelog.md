# Changelog

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
