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
