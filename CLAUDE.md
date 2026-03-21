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
- **Always bump the plugin version when changing any skill, command, or agent.** Run `/publish` before pushing — every time, no exceptions. This bumps the version, commits, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.

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

### Activity Lifecycle

Skills are invoked in this order to take an activity from idea to execution:

1. **organize** — Bootstrap directory, plugins, CLAUDE.md, and GitHub repo
2. **discover** — Expert interview producing a detailed brief
3. **roadmap** — Adaptive waypoint-based execution plan from the brief
4. **waypoint-design** — Design individual waypoints with sufficient detail for task decomposition
5. **waypoint-planner** — Decompose a designed waypoint into an executable task DAG
6. **waypoint-implement** — Orchestrate parallel workers to execute tasks

Skills invoke each other via `do-stuff-helper:<skill-name>`.

**Software-build delegation:** When `waypoint-design` detects a waypoint is primarily a software build, it can delegate to ClaudePluginBuild's pipeline (`/build:prd` → `/build:design` → `/build:plan` → `/build:implement`) instead of steps 5-6 above. The waypoint's `delegation` field in the roadmap JSON signals this. See the Roadmap JSON Fields section.

### Artifact Naming

Activity artifacts follow the pattern `<type>-<activity-slug>.<ext>`:
- `docs/brief-<slug>.md` — Discovery brief
- `docs/roadmap-<slug>.json` — Waypoint graph (source of truth)
- `docs/waypoints/<id>.md` — Individual waypoint design documents
- `docs/changelog.md` — Work log maintained by the implement skill
- `docs/inbox.json` — Questions and blockers waiting on the user
- `docs/backlog.md` — Out-of-scope discoveries for future replanning

### Waypoint Storage

- **`docs/roadmap-<slug>.json`** — Source of truth. Contains waypoint metadata (id, status, dependencies, phases) as structured JSON.
- **`docs/waypoints/<waypoint-id>.md`** — Design documents with two required sections: **Objective** (one sentence) and **Done When** (acceptance criteria). Everything else is freeform, scaled to complexity.

#### Roadmap JSON Fields

Each waypoint entry in the roadmap JSON supports these fields:

| Field | Required | Description |
|-------|----------|-------------|
| `id` | yes | Waypoint identifier (e.g., `"w4"`) |
| `title` | yes | Short descriptive title |
| `status` | yes | Lifecycle status (see Waypoint Statuses below) |
| `phase` | yes | Which roadmap phase this belongs to |
| `dependencies` | yes | Array of waypoint IDs that must complete first |
| `why` | yes | Motivation for this waypoint |
| `done_when` | yes | Acceptance criteria |
| `updated` | no | Date of last status change (YYYY-MM-DD) |
| `delegation` | no | External plugin handling this waypoint's planning and execution (e.g., `"ClaudePluginBuild"`). When set, `waypoint-planner` is skipped and `waypoint-implement` delegates to the named plugin instead of dispatching its own workers. |

### Waypoint Statuses

- `pending` — Not started; waiting for dependencies or prioritization
- `designed` — Design document complete; ready for task decomposition (planning)
- `planned` — Tasks created; ready for workers to execute
- `started` — Workers actively executing this waypoint
- `waiting` — Blocked on user input; questions or human tasks outstanding in inbox
- `done` — Completed; acceptance criteria met
- `obsolete` — No longer relevant; skip when calculating what's unblocked

### Keeping the Brief Current

The brief (`docs/brief-<slug>.md`) is the source of truth for what an activity is about. When a conversation results in material scope changes — new capabilities, obsoleted waypoints, or shifted priorities — update the brief to reflect the change. Don't wait for a replan cycle; update it while the context is fresh.

### Backlog

- **In-scope discoveries:** Workers add tasks directly and do them. No special process.
- **Out-of-scope discoveries:** Append to `docs/backlog.md` with date and source. The `replan` skill processes the backlog into roadmap updates.
- **Format:** Append-only sections: `## YYYY-MM-DD — <source>\n\n<description>`

### Skill Priorities

When a do-stuff-helper skill and a superpowers skill both apply to the same task, prefer the do-stuff-helper skill. Use superpowers skills only when no do-stuff-helper skill covers the need. Specific overlaps:

- **Starting a new project/idea:** Use `discover`, not `superpowers:brainstorming`
- **Creating a plan:** Use `roadmap`, `waypoint-design`, or `waypoint-planner`, not `superpowers:writing-plans`
- **Executing a plan:** Use `waypoint-implement`, not `superpowers:executing-plans` or `superpowers:subagent-driven-development`
- **Creating/editing skills:** Use `skill-creator:skill-creator`, not `superpowers:writing-skills`

Superpowers skills that have no do-stuff-helper equivalent should still be used normally: `test-driven-development`, `systematic-debugging`, `verification-before-completion`, `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch`, `using-git-worktrees`, `dispatching-parallel-agents`.

### Plugin Version Bumps

**Always bump the do-stuff-helper plugin version when changing any skill, command, or agent.** Run `/publish` before pushing. This bumps the version, commits, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.
