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
2. **discover** — Conducts an expert interview and produces a detailed brief
3. **roadmap** _(future)_ — Builds an execution plan from the brief

### Cross-Skill Invocation

Skills invoke each other using the pattern `do-stuff-helper:<skill-name>`. For example, the discovery skill invokes `do-stuff-helper:organize` to set up the directory before saving the brief.

### Artifact Naming

Activity artifacts follow the pattern `<type>-<activity-slug>.md`:
- `brief-my-fitness-app.md` — Discovery brief
- `roadmap-my-fitness-app.md` — Execution plan _(future)_

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
