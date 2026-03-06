---
name: roadmap
description: This skill should be used when the user explicitly asks to "build a roadmap", "create a roadmap", or "make a roadmap" for an activity, or when transitioning from the discover skill after a brief is saved. Do NOT trigger for general planning, task work, or "what should we work on" — those are task-level activities, not roadmap creation.
---

# Roadmap

Read an activity brief, conduct expert analysis, and produce an adaptive waypoint-based roadmap autonomously. The roadmap is stored as a JSON file (source of truth) with individual waypoint design stubs. A review waypoint (w0) is created and surfaced to the dashboard inbox so the user can validate the roadmap asynchronously.

## Checklist

Complete each step in strict order. Do not skip steps.

### Step 1: Locate the Brief

- If invoked via suggest-and-confirm from discover, the brief path is already known
- If invoked directly, look for `docs/brief-*.md` in the current activity directory
- If multiple briefs exist, ask the user which one
- If no brief exists, tell the user to run discover first and stop
- Read the brief in full

### Step 2: Expert Analysis (Internal)

Analyze the brief to identify:

1. **Natural phases** — what groups of work form logical stages?
2. **Critical path** — what must happen before other things can start?
3. **Quick wins** — what small items could build early momentum?
4. **Knowledge gaps** — where does the brief lack enough detail for planning?
5. **Risks that affect sequencing** — which risks suggest doing certain work first?
6. **Domain best practices** — what would an expert in this domain recommend for execution order?

Do not present this analysis to the user. Use it internally to inform the waypoint graph in Step 3. Note any questions, knowledge gaps, or sequencing uncertainties — these will be included in the review waypoint's design doc (Step 5).

**Research:** Invoke `do-stuff-helper:research` during this step if domain-specific sequencing questions arise that would benefit from investigation.

### Step 3: Build the Waypoint Graph

Build the full waypoint graph autonomously from the expert analysis. Do not propose the structure to the user or wait for feedback.

For each waypoint, define:
- `id` — short identifier (e.g., `w1`, `w2`)
- `title` — human-readable name
- `status` — initially `pending` (see w0 exception below)
- `phase` — which phase it belongs to
- `dependencies` — list of waypoint IDs that must complete first
- `why` — one sentence on why this waypoint matters
- `done_when` — concrete acceptance criteria (1-3 sentences)
- `due` — (optional) ISO date string (e.g., `2026-04-15`). Only include when the user indicates a time constraint, deadline, or natural target date for the waypoint. Do not invent due dates — omit the field entirely when there is no time pressure.

For each phase, define:
- `id` — short identifier (e.g., `phase-1`)
- `title` — human-readable phase name
- `goal` — what this phase achieves overall

**Review waypoint and dependency injection:**

1. Prepend a `phase-0: Review` phase with goal `"Validate the roadmap with the user before execution begins"`
2. Create `w0: Review roadmap` with `status: "started"`, `phase: "phase-0"`, `dependencies: []`
3. All waypoints that originally had empty dependencies get `["w0"]` added — transitive blocking covers the rest
4. All other waypoints remain `status: "pending"`

### Step 4: Write the JSON

Write `docs/roadmap-<slug>.json` using this schema:

```json
{
  "activity": "<activity-slug>",
  "name": "<Human-readable activity name>",
  "description": "<One sentence describing the activity>",
  "created": "<ISO date>",
  "updated": "<ISO date>",
  "phases": [
    {
      "id": "phase-0",
      "title": "Review",
      "goal": "Validate the roadmap with the user before execution begins"
    },
    {
      "id": "phase-1",
      "title": "Phase Name",
      "goal": "What this phase achieves"
    }
  ],
  "waypoints": [
    {
      "id": "w0",
      "title": "Review roadmap",
      "status": "started",
      "phase": "phase-0",
      "dependencies": [],
      "why": "Ensure the roadmap matches user intent before committing effort",
      "done_when": "User has reviewed the roadmap, confirmed or requested changes"
    },
    {
      "id": "w1",
      "title": "Waypoint Title",
      "status": "pending",
      "phase": "phase-1",
      "dependencies": ["w0"],
      "why": "Why this matters",
      "done_when": "Concrete acceptance criteria"
    }
  ]
}
```

**`due` is optional:** Only include it when the user specifies a deadline or natural target date. Most waypoints will not have one. Omit the field entirely rather than setting it to `null`.

**Phase status is derived, not stored:** `done` if all waypoints in the phase are `done`, `implementing` if any are `implementing`, `pending` otherwise.

### Step 5: Create Review Waypoint Design Doc

Write `docs/waypoints/w0.md` with a full design doc (not a stub):

```markdown
# w0: Review roadmap

## Objective

Validate the roadmap matches user intent before committing effort to execution.

## Done When

User has reviewed the roadmap, confirmed or requested changes.

## Questions from Planning

<List any knowledge gaps, ambiguities, sequencing uncertainties, or thin-brief concerns from Step 2. If none, write "None — brief was comprehensive.">

## Review Checklist

- [ ] Phases are in the right order
- [ ] Waypoints capture everything in the brief
- [ ] Dependencies make sense
- [ ] Nothing critical is missing
- [ ] Granularity feels right (not too big, not too small)
```

### Step 6: Create Other Waypoint Design Stubs

For each waypoint **except w0**, create `docs/waypoints/<waypoint-id>.md` with:

```markdown
# <Waypoint ID>: <Waypoint Title>

## Objective

<one sentence from done_when>

## Done When

<acceptance criteria from the JSON>
```

These are stubs — the waypoint design skill (or manual design) will flesh them out before decomposition.

### Step 7: Create Review Task and Surface to Inbox

This step inlines planner+implement logic so the human review task appears in the dashboard inbox immediately.

1. **Create task:** Call `TaskCreate` with:
   - `subject`: `[w0] Review and approve the roadmap for <activity-name>`
   - `description`: `Review the roadmap at docs/roadmap-<slug>.json and the design doc at docs/waypoints/w0.md. Confirm the roadmap matches your intent, or request changes. Check the review checklist in w0.md.`
   - `metadata`: `{ "waypoint": "w0", "assignee": "human" }`

2. **Write inbox entry:** Write (or append to) `docs/inbox.json`. Auto-increment the `q-N` ID based on existing entries. Use this format:
   ```json
   {
     "id": "q-<next>",
     "type": "blocker",
     "task_id": "<from TaskCreate>",
     "waypoint": "w0",
     "subject": "[w0] Review and approve the roadmap for <activity-name>",
     "body": "The roadmap has been created with <N> phases and <M> waypoints. Please review docs/roadmap-<slug>.json and docs/waypoints/w0.md. All waypoints are blocked until you approve or request changes.",
     "context": "Roadmap created on <date>. Key files: docs/roadmap-<slug>.json, docs/waypoints/w0.md, docs/brief-<slug>.md",
     "created": "<ISO datetime>",
     "status": "pending",
     "resolution": null,
     "resolved_at": null
   }
   ```

3. w0 status is already `started` from Step 3 — no additional update needed.

### Step 8: Save and Commit

1. Ensure the JSON file, all waypoint docs, and inbox.json are written
2. Prepend a milestone entry to `docs/changelog.md` (insert after the `# Changelog` header, newest first):
   ```
   ## YYYY-MM-DD — Roadmap created: <Activity Name>
   <N> phases, <M> waypoints. Review waypoint (w0) surfaced to inbox.
   ```
3. Stage and commit with message: `roadmap: create roadmap for <activity-slug>`

### Step 9: Report

Report what was created:
- Number of phases and waypoints
- Tell the user: the review task is in the inbox, and all waypoints are blocked until w0 is approved

Do **not** prompt the user to design waypoints or ask follow-up questions. The review will happen asynchronously via the inbox.

## Edge Cases

- **Activity already has a roadmap:** Ask the user if they want to replace it or update it. For now, replacing is fine — the backlog/replanning skill will handle incremental updates later.
- **Brief is thin:** Flag specific gaps in w0's "Questions from Planning" section instead of asking the user interactively. The review waypoint becomes the place to surface these concerns.
- **Very small activity:** Some activities may only need 3-5 waypoints with no phases beyond phase-0. Scale down — don't force phases on a simple activity. w0 is always included for consistency.
- **Very large activity:** For complex activities, keep the first phase detailed and later phases coarser. They'll be refined when we get there.
- **Existing inbox.json:** Append to the existing array. Auto-increment the `q-N` ID based on the highest existing ID.

## Cross-Skill Invocation

- **`do-stuff-helper:research`** — May invoke during Step 2 if domain-specific sequencing questions arise that benefit from research.
- **Invoked by `do-stuff-helper:discover`** — Via suggest-and-confirm after the brief is saved.
