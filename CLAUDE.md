# do-stuff-helper

A Claude Code plugin providing skills, commands, and subagents for thoughtful execution of personal projects and programs.

## Project Structure

```
do-stuff-helper/
├── .claude-plugin/
│   ├── plugin.json          # Plugin metadata
│   └── marketplace.json     # Marketplace registry for plugin distribution
├── commands/                # Slash commands (user-invoked)
│   └── *.md
├── skills/                  # Skills (model-invoked)
│   └── skill-name/
│       └── SKILL.md
├── agents/                  # Subagents (long-running autonomous tasks)
│   └── *.md
├── activities/              # Activity directories (user creates, organize bootstraps)
│   └── <activity-slug>/
├── CLAUDE.md
└── README.md
```

## Conventions

### Skills (`skills/*/SKILL.md`)
- Frontmatter must include `name` and `description`
- Description should specify trigger phrases and when Claude should auto-invoke
- Keep instructions actionable and concise
- Use examples to clarify expected behavior

### Commands (`commands/*.md`)
- Frontmatter must include `description`
- Use `argument-hint` for commands that accept arguments
- Use `allowed-tools` to pre-approve tools and reduce permission prompts

### Agents (`agents/*.md`)
- Frontmatter must include `name`, `description`, `model`, `tools`
- Include example trigger scenarios in description
- Prefer `sonnet` model unless task requires `opus`

## Activity Conventions

Activity conventions (lifecycle, artifact naming, waypoint storage, statuses, backlog) are maintained in `skills/organize/references/activity-conventions.md`. The organize skill injects these into each activity's CLAUDE.md. See that file for the canonical reference.

## Development Workflow
- Use the `skill-creator` plugin to create and test new skills
- Test skills locally before committing by installing the plugin at project scope
- Keep skill descriptions precise — vague descriptions cause false triggers
- **When you change any skill, command, or agent:** run `/publish` before pushing. This bumps the version, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.

### Plugin Distribution
- `marketplace.json` uses `"./"` as source for self-referencing plugins — other formats (`"."`, `github`, `url`) failed or used SSH
- Users install via: `/plugin marketplace add ryanismert/do-stuff-helper` then `/plugin install do-stuff-helper@do-stuff-helper`

## Activity Brief

**Goal:** Build a skill and agent system on Claude Code that helps define, develop, and execute diverse personal activities — from software projects to life improvement goals — with maximum autonomy and minimum dependency on the user for routine work.

**Success Criteria:**
- 3-4 activities running simultaneously with autonomous workers making real progress
- User time spent on decisions, not grunt work
- Meaningful progress on previously-stalled life goals
- System gets more autonomous over time through ask-and-learn permissions

**Scope:**
- In: Full activity execution pipeline (discover → roadmap → waypoint design → decompose → execute), monitoring dashboard with inbox, forward motion analysis, advisory coaching agent pattern, home server infrastructure (Docker, n8n, browser automation)
- Out: Telegram/chat integration (deferred to coaching build), calendar/email integration, dashboard UI technology selection, custom chat interface

**Key Risks:**
- Air gaps where only the user can unblock progress, compounded by tendency to avoid hard tasks
- Scope is ambitious; risk of building infrastructure without realizing value
- Autonomous agent quality — low-quality unsupervised work could cost more to review than it saves

For full details including background, open questions, and roadmap planning notes, see [the complete brief](docs/brief-do-stuff-helper.md).

## Current Status

To understand what to work on next, read `docs/roadmap-do-stuff-helper.json`. It contains all waypoints with statuses and dependencies. Waypoint design documents live in `docs/waypoints/<waypoint-id>.md`. See `skills/organize/references/activity-conventions.md` for waypoint status definitions.

### Infrastructure in Main Repo

The dashboard and inbox infrastructure live in the **main exoselfai repo** (`~/exoselfai/`), not in this plugin repo. Key locations:

- `scripts/dashboard/` — Dashboard service (server, collector, UI). Runs as Docker container on port 3002.
- `scripts/discuss.js` — Tmux-based Claude remote-control session manager for inbox discussions.
- `scripts/webhook-server.js` — Webhook API server (systemd, port 3001). Hosts discuss endpoints + task execution.
- `docker-compose.yml` — `exoself-dashboard` service definition.

Per-activity data (`docs/inbox.json`, `docs/changelog.md`, `docs/roadmap-*.json`) lives in each activity's own repo. See `docs/waypoints/w10.md` for the full file map.

## do-stuff-helper

This project uses the do-stuff-helper plugin for guided project execution.

### Available Skills
- **organize** — Bootstrap project directory, plugins, and GitHub repo
- **discover** — Expert-driven interview to produce a detailed project brief
- **research** — Multi-angle web research with structured summaries
- **roadmap** — Build an adaptive waypoint-based execution plan from the brief
- **waypoint-design** — Design individual waypoints with sufficient detail for decomposition
- **waypoint-planner** — Decompose a waypoint into executable tasks
- **waypoint-implement** — Execute tasks from the waypoint plan
- **replan** — Process the backlog into roadmap and brief updates

### Task List
This activity uses `CLAUDE_CODE_TASK_LIST_ID=do-stuff-helper` for persistent cross-session task tracking.

### Capabilities

Workers and agents in this project have access to:

**Skills:** organize, discover, research, roadmap, waypoint-design, waypoint-planner, waypoint-implement, replan

**MCP Servers:** None configured

**CLI Tools:** gh 2.86.0, node 25.6.0, docker 29.1.5, python3 3.9.6, npm 11.8.0, npx 11.8.0, make 3.81

**Project Scripts:** None (no package.json or Makefile)

### Usage
Invoke skills via `do-stuff-helper:<skill-name>`.
