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

**Software-build delegation:** When `waypoint-design` detects a waypoint is primarily a software build, it can delegate to ClaudePluginBuild's pipeline (`/build:prd` → `/build:design` → `/build:plan` → `/build:implement`) instead of steps 5-6 above. The waypoint's `delegation` field in the roadmap JSON signals this. See the Roadmap JSON Fields section.

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

### Roadmap JSON Fields

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

## Waypoint Statuses

- `pending` — Not started; waiting for dependencies or prioritization
- `designed` — Design document complete; ready for task decomposition (planning)
- `planned` — Tasks created; ready for workers to execute
- `started` — Workers actively executing this waypoint
- `waiting` — Blocked on user input; questions or human tasks outstanding in inbox
- `done` — Completed; acceptance criteria met
- `obsolete` — No longer relevant; skip when calculating what's unblocked

## Keeping the Brief Current

The brief (`docs/brief-<slug>.md`) is the source of truth for what an activity is about. When a conversation results in material scope changes — new capabilities, obsoleted waypoints, or shifted priorities — update the brief to reflect the change. Don't wait for a replan cycle; update it while the context is fresh.

## Backlog

- **In-scope discoveries:** Workers add tasks directly and do them. No special process.
- **Out-of-scope discoveries:** Append to `docs/backlog.md` with date and source. The `replan` skill processes the backlog into roadmap updates.
- **Format:** Append-only sections: `## YYYY-MM-DD — <source>\n\n<description>`

## Plugin Version Bumps

**Always bump the do-stuff-helper plugin version when changing any skill, command, or agent.** Run `/publish` before pushing. This bumps the version, commits, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.
