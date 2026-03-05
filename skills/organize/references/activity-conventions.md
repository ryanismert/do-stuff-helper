# Activity Conventions

Conventions for activities managed by the do-stuff-helper plugin. This content is injected into each activity's CLAUDE.md by the organize skill so that all agents and workers share the same understanding.

## Activity Lifecycle

Skills are invoked in this order to take an activity from idea to execution:

1. **organize** — Bootstrap directory, plugins, CLAUDE.md, and GitHub repo
2. **discover** — Expert interview producing a detailed brief
3. **roadmap** — Adaptive waypoint-based execution plan from the brief
4. **waypoint-design** — Design individual waypoints with sufficient detail for task decomposition
5. **waypoint-planner** — Decompose a designed waypoint into an executable task DAG
6. **waypoint-implement** — Orchestrate parallel workers to execute tasks

Skills invoke each other via `do-stuff-helper:<skill-name>`.

## Artifact Naming

Activity artifacts follow the pattern `<type>-<activity-slug>.<ext>`:
- `docs/brief-<slug>.md` — Discovery brief
- `docs/roadmap-<slug>.json` — Waypoint graph (source of truth)
- `docs/waypoints/<id>.md` — Individual waypoint design documents
- `docs/changelog.md` — Work log maintained by the implement skill
- `docs/inbox.json` — Questions and blockers waiting on the user
- `docs/backlog.md` — Out-of-scope discoveries for future replanning

## Waypoint Storage

- **`docs/roadmap-<slug>.json`** — Source of truth. Contains waypoint metadata (id, status, dependencies, phases) as structured JSON.
- **`docs/waypoints/<waypoint-id>.md`** — Design documents with two required sections: **Objective** (one sentence) and **Done When** (acceptance criteria). Everything else is freeform, scaled to complexity.

## Waypoint Statuses

- `pending` — Not started; waiting for dependencies or prioritization
- `designed` — Design document complete; ready for task decomposition (planning)
- `planned` — Tasks created; ready for workers to execute
- `started` — Workers actively executing this waypoint
- `waiting` — Blocked on user input; questions or human tasks outstanding in inbox
- `done` — Completed; acceptance criteria met
- `obsolete` — No longer relevant; skip when calculating what's unblocked

## Backlog

- **In-scope discoveries:** Workers add tasks directly and do them. No special process.
- **Out-of-scope discoveries:** Append to `docs/backlog.md` with date and source. The `replan` skill processes the backlog into roadmap updates.
- **Format:** Append-only sections: `## YYYY-MM-DD — <source>\n\n<description>`

## Plugin Version Bumps

**Always bump the do-stuff-helper plugin version when changing any skill, command, or agent.** Run `/publish` before pushing. This bumps the version, commits, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.
