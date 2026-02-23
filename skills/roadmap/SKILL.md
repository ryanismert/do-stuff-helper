---
name: roadmap
description: This skill should be used when the user explicitly asks to "build a roadmap", "create a roadmap", or "make a roadmap" for an activity, or when transitioning from the discover skill after a brief is saved. Do NOT trigger for general planning, task work, or "what should we work on" — those are task-level activities, not roadmap creation.
---

# Roadmap

Read an activity brief, conduct an expert-driven interview about priorities and sequencing, and produce an adaptive waypoint-based roadmap. The roadmap is stored as a JSON file (source of truth) with individual waypoint design stubs.

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

Do not present this analysis to the user. Use it internally to inform the proposed structure in Step 3.

### Step 3: Propose Initial Structure

Present the user with a proposed roadmap structure:
- Suggested phases with rationale
- Key waypoints grouped by phase
- Critical dependencies between waypoints
- Suggested tactical items (unblockers, knowledge gaps, low-hanging fruit)

Frame this as a starting point for discussion, not a final plan. Ask:

> "Here's how I'd structure this. What feels right? What's wrong? What's missing?"

### Step 4: Iterative Refinement

Refine based on user feedback. This is conversational — not a rigid loop. Topics to cover as needed:

- **Phase ordering** — does the user agree with the proposed sequence?
- **Waypoint granularity** — are waypoints too big, too small, or right-sized?
- **Missing waypoints** — are there things the brief mentions that aren't captured?
- **Dependencies** — are there constraints the skill missed?
- **Tactical items** — small things that unblock progress or fill gaps
- **Priority within phases** — which waypoints matter most within each phase?

Continue until the user confirms the roadmap looks good. Don't over-refine — the roadmap is adaptive and will be replanned as learning occurs.

**Research:** Invoke `do-stuff-helper:research` at any point during refinement if domain-specific sequencing questions arise that would benefit from investigation.

### Step 5: Build the Waypoint Graph

For each waypoint, define:
- `id` — short identifier (e.g., `w1`, `w2`)
- `title` — human-readable name
- `status` — initially `pending` (or `in-progress`/`done` if work has already started)
- `phase` — which phase it belongs to
- `dependencies` — list of waypoint IDs that must complete first
- `why` — one sentence on why this waypoint matters
- `done_when` — concrete acceptance criteria (1-3 sentences)

For each phase, define:
- `id` — short identifier (e.g., `phase-1`)
- `title` — human-readable phase name
- `goal` — what this phase achieves overall

### Step 6: Write the JSON

Write `docs/roadmap-<slug>.json` using this schema:

```json
{
  "activity": "<activity-slug>",
  "created": "<ISO date>",
  "updated": "<ISO date>",
  "phases": [
    {
      "id": "phase-1",
      "title": "Phase Name",
      "goal": "What this phase achieves"
    }
  ],
  "waypoints": [
    {
      "id": "w1",
      "title": "Waypoint Title",
      "status": "pending",
      "phase": "phase-1",
      "dependencies": [],
      "why": "Why this matters",
      "done_when": "Concrete acceptance criteria"
    }
  ]
}
```

**Phase status is derived, not stored:** `done` if all waypoints in the phase are `done`, `in-progress` if any are `in-progress`, `pending` otherwise.

### Step 7: Create Waypoint Design Stubs

For each waypoint, create `docs/waypoints/<waypoint-id>.md` with:

```markdown
# <Waypoint ID>: <Waypoint Title>

## Objective

<one sentence from done_when>

## Done When

<acceptance criteria from the JSON>
```

These are stubs — the waypoint design skill (or manual design) will flesh them out before decomposition.

### Step 8: Save and Commit

1. Write the JSON file to `docs/roadmap-<slug>.json`
2. Create the `docs/waypoints/` directory if it doesn't exist
3. Write all waypoint design stubs
4. Stage and commit with message: `roadmap: create roadmap for <activity-slug>`
5. Report file paths and confirm everything is saved

### Step 9: Suggest Next Steps

Identify all waypoints with no unresolved dependencies (i.e., dependencies are either empty or all `done`). Present them to the user:

> "The roadmap is saved. These waypoints have no blockers and are ready for design: [list waypoint IDs and titles]. Designing them now will enable parallel execution once the decompose & execute skill is built. Want to start designing them?"

If the waypoint-design skill exists, offer to invoke it. If not, offer to do manual design (flesh out the waypoint design docs directly).

## Edge Cases

- **Activity already has a roadmap:** Ask the user if they want to replace it or update it. For now, replacing is fine — the backlog/replanning skill will handle incremental updates later.
- **Brief is thin:** If the brief lacks enough detail for meaningful waypoints, flag specific gaps and ask the user to fill them (or suggest re-running discover).
- **Very small activity:** Some activities may only need 3-5 waypoints with no phases. Scale down — don't force phases on a simple activity.
- **Very large activity:** For complex activities, suggest keeping the first phase detailed and later phases coarser. They'll be refined when we get there.

## Cross-Skill Invocation

- **`do-stuff-helper:research`** — May invoke during Step 4 if domain-specific sequencing questions arise that benefit from research.
- **Invoked by `do-stuff-helper:discover`** — Via suggest-and-confirm after the brief is saved.
