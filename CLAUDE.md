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
├── activities/              # Activity directories (created by organize skill)
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

## Activity Model

Activities are the core unit of work in do-stuff-helper. Each activity represents a project, program, or goal the user wants to execute.

### Directory Structure

Activities live under `activities/<activity-slug>/`:

```
activities/
└── my-fitness-app/
    ├── README.md                      # Created by organize
    ├── brief-my-fitness-app.md        # Created by discovery
    └── ...                            # Future artifacts from other skills
```

### Activity Lifecycle

Skills are invoked in this order to take an activity from idea to execution:

1. **organize** — Creates the activity directory structure
2. **discover** — Conducts an expert interview and produces a detailed brief (suggest-and-confirm transition to roadmap)
3. **roadmap** — Builds an adaptive waypoint-based execution plan from the brief
4. **waypoint-design** — Designs individual waypoints with sufficient detail for decomposition
5. **decompose & execute** _(future)_ — Recursive task decomposition and parallel worker execution

### Cross-Skill Invocation

Skills invoke each other using the pattern `do-stuff-helper:<skill-name>`. For example, the discovery skill invokes `do-stuff-helper:organize` to set up the directory before saving the brief.

### Artifact Naming

Activity artifacts follow the pattern `<type>-<activity-slug>.<ext>`:
- `brief-my-fitness-app.md` — Discovery brief
- `roadmap-my-fitness-app.json` — Waypoint graph (source of truth)

### Waypoint Storage

Roadmaps use two artifact types:

1. **`docs/roadmap-<slug>.json`** — Source of truth. Contains waypoint metadata (id, status, dependencies, phase membership) in a structured JSON format. Skills read and write this file. LLMs read JSON directly — no auto-generated markdown needed.
2. **`docs/waypoints/<waypoint-id>.md`** — Individual waypoint design documents. Referenced by the JSON.

**Waypoint design documents** have two required sections:
- **Objective** — one sentence stating what this waypoint achieves
- **Done When** — concrete acceptance criteria

Everything else is freeform. The waypoint design agent scales detail to complexity — simple items get a sentence, complex ones get full designs.

**Phases** are tracked in the JSON as grouping mechanisms with their own completion status, giving humans a sense of progress.

### Backlog & Discovered Work

- **In-scope discoveries:** When a worker discovers additional work needed for the current waypoint, it adds tasks directly to the task list and does them. No special process needed.
- **Out-of-scope discoveries:** When a worker or the user discovers work that belongs to a different waypoint or is entirely new, append it to `docs/backlog.md`. The `replan` skill periodically processes the backlog into roadmap and brief updates.
- **Backlog format:** Append-only, freeform sections with date and source. See `docs/backlog.md` for the format.

## Development Workflow
- Use the `skill-creator` plugin to create and test new skills
- Test skills locally before committing by installing the plugin at project scope
- Keep skill descriptions precise — vague descriptions cause false triggers

### Plugin Distribution
- `marketplace.json` uses `"./"` as source for self-referencing plugins — other formats (`"."`, `github`, `url`) failed or used SSH
- After updating skills, bump `version` in both `plugin.json` and `marketplace.json`
- Users install via: `/plugin marketplace add ryanismert/do-stuff-helper` then `/plugin install do-stuff-helper@do-stuff-helper`
- Skills are not hotloaded — a new conversation is required after installing or updating

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

To understand what to work on next, read `docs/roadmap-do-stuff-helper.json`. It contains all waypoints with statuses and dependencies. Waypoint design documents live in `docs/waypoints/<waypoint-id>.md`.

**Waypoint statuses:**
- `pending` — Not started; waiting for dependencies or prioritization
- `implementing` — A worker/agent is actively executing this waypoint
- `waiting` — Blocked on user input; questions or human tasks are outstanding in the inbox. Set by the implement skill when all remaining tasks need user action. n8n monitors for this status and triggers resume when answers arrive.
- `done` — Completed; acceptance criteria met
- `obsolete` — No longer relevant; skip when calculating what's unblocked

Waypoints with status `pending` and all dependencies `done` are ready for design or execution. Waypoints in `waiting` status may resume automatically when the user answers inbox items.

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
