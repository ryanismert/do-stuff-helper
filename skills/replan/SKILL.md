---
name: replan
description: This skill should be used when the user asks to "replan", "review the backlog", "process the backlog", "update the roadmap based on new ideas", or when accumulated backlog items need to be incorporated into the roadmap and brief. Do NOT trigger for initial roadmap creation — use the roadmap skill for that.
---

# Replan

Read the backlog, categorize items, and propose changes to the activity brief and roadmap. Each backlog item becomes either a design update (modifies the brief and may reorder/change waypoints) or a tactical waypoint (new discrete work added to the roadmap).

## Checklist

Complete each step in order. Do not skip steps.

### Step 1: Read Current State

Read the following files to understand the activity's current state:

1. `docs/backlog.md` — accumulated ideas, scope changes, and discovered work
2. `docs/roadmap-<slug>.json` — current waypoint graph
3. `docs/brief-<slug>.md` — original activity brief

If the backlog is empty or doesn't exist, tell the user there's nothing to process and stop.

### Step 2: Categorize Backlog Items

For each item in the backlog, categorize it as one of:

- **Design update** — changes understanding of the activity, its goals, scope, or approach. Will result in updates to the brief and potentially waypoint reordering, modification, or removal.
- **Tactical waypoint** — a new discrete work item that should be added to the roadmap as a waypoint with its own dependencies and acceptance criteria.

If an item is ambiguous, make your best judgment and note it — the user will review everything before changes are applied.

### Step 3: Draft Proposed Changes

For each categorized item, draft the specific change:

**For design updates:**
- Identify which section(s) of the brief need updating
- Draft the updated text
- Identify any waypoints that need modification as a consequence (scope change, new dependencies, reordering)

**For tactical waypoints:**
- Draft the waypoint entry: id, title, phase, dependencies, why, done_when
- Assign the next available waypoint ID
- Identify which phase it belongs to (or propose a new phase if needed)

### Step 4: Present to User

Present the full proposal organized by category:

> **Design Updates:**
> 1. [summary of change] — affects brief section X, waypoints Y and Z
>
> **New Waypoints:**
> 1. [waypoint title] — [why], depends on [deps]
>
> **No Action (informational only):**
> 1. [items that don't require changes, if any]
>
> "Does this look right? Want to adjust anything before I apply these changes?"

Iterate on feedback until the user confirms.

### Step 5: Apply Changes

On user confirmation:

1. **Update the brief** — edit `docs/brief-<slug>.md` with design updates
2. **Update the roadmap** — edit `docs/roadmap-<slug>.json`:
   - Add new waypoints
   - Modify existing waypoints (scope, dependencies, status)
   - Reorder if priorities have shifted
   - Update the `updated` date
3. **Create waypoint stubs** — for any new waypoints, create `docs/waypoints/<id>.md` with Objective and Done When sections
4. **Clear processed items** — remove the processed entries from `docs/backlog.md`, leaving the header and any unprocessed items
5. **Commit** — stage all changed files and commit with a message summarizing what was changed:
   ```
   replan: process backlog — <brief summary>

   Design updates:
   - <list>

   New waypoints:
   - <list>
   ```

### Step 6: Report

Summarize what was done:
- Number of items processed
- Brief sections updated
- Waypoints added or modified
- Any items left in the backlog (unprocessed)

Suggest next steps — e.g., designing newly added waypoints, or continuing with implementation.

## Edge Cases

- **Empty backlog:** Tell the user and stop. Nothing to process.
- **All items are informational:** Some backlog items may just be observations that don't require action. Categorize them as "no action" and ask the user if they should be cleared.
- **Conflicting items:** If two backlog items contradict each other, flag the conflict and ask the user to resolve before applying.
- **Item affects an in-progress waypoint:** Flag this clearly — changing a waypoint that's currently being worked on may invalidate active tasks.
- **No brief exists:** If the activity has no brief, the replan skill can still add tactical waypoints to the roadmap. Design updates without a brief should prompt the user to run discover first.

## Cross-Skill Invocation

- **Invoked by user** — "let's replan", "review the backlog", "process the backlog", "update the roadmap"
- **Fed by workers** — waypoint-implement workers append out-of-scope discoveries to `docs/backlog.md`
- **Could be scheduled** — n8n weekly trigger via Claude webhook (future)
- **May invoke `do-stuff-helper:waypoint-design`** — after adding new waypoints, suggest designing them
