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

## Skill Priorities

When a do-stuff-helper skill and a superpowers skill both apply to the same task, prefer the do-stuff-helper skill. Use superpowers skills only when no do-stuff-helper skill covers the need. Specific overlaps:

- **Starting a new project/idea:** Use `discover`, not `superpowers:brainstorming`
- **Creating a plan:** Use `roadmap`, `waypoint-design`, or `waypoint-planner`, not `superpowers:writing-plans`
- **Executing a plan:** Use `waypoint-implement`, not `superpowers:executing-plans` or `superpowers:subagent-driven-development`
- **Creating/editing skills:** Use `skill-creator:skill-creator`, not `superpowers:writing-skills`

Superpowers skills that have no do-stuff-helper equivalent should still be used normally: `test-driven-development`, `systematic-debugging`, `verification-before-completion`, `requesting-code-review`, `receiving-code-review`, `finishing-a-development-branch`, `using-git-worktrees`, `dispatching-parallel-agents`.

## Git Branching Policy: Work Only on Branches

**No session — worker, primary/owner, or XO — commits directly to local `main` in an activity repo** (Ryan, 2026-07-24). Every change, including roadmap and changelog edits, goes onto a branch and merges via a PR into `origin/main`.

- Local `main` is a **pure mirror** of `origin/main`. It only advances by pulling (`git fetch && git merge --ff-only origin/main`), never by a local commit.
- Why: PRs are the check-in control on `main`, and fast-forward-only pulls keep the working copy (and anything reading it, like the waypoint dashboard) showing merged reality. A local main that diverges from origin hides merged work — this is the failure that hid PR work from the dashboard on 2026-07-24.
- The primary session still *authors* roadmap/changelog changes, but as branch commits that land via PR — it does not commit them to local main.

## Git Worktrees

Two locations, by who runs the session (clarified with Ryan 2026-07-24):

- **Dispatched/automated sessions** (visualizer dispatches, XO workers, native
  tooling) use the repo's own `.claude/worktrees/<task>/` — Claude Code's
  native location. It's inside the project directory, so no permission
  prompt, and it's invisible to forward-motion discovery: the scan glob
  `activities/*/docs/roadmap-*.json` cannot reach
  `activities/<name>/.claude/worktrees/<wt>/docs/`.
- **Manually-run secondary sessions** use `~/worktrees/`
  (e.g. `git worktree add ~/worktrees/wildlifecards-feature feature-branch`).
- What's banned is a worktree as a sibling directory directly under
  `activities/` — that WOULD match the scan glob and appear as a phantom
  duplicate activity on the dashboard.
- The primary Claude session stays in `activities/<name>/` and remains the
  author of roadmap and changelog PRs (see branching policy above).

## Plugin Version Bumps

**Always bump the do-stuff-helper plugin version when changing any skill, command, or agent.** Run `/publish` before pushing. This bumps the version, commits, pushes, and updates the marketplace so other projects pick up the changes. Skills are not hotloaded — a new conversation is required after updating.
