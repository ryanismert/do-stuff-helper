# W1: Roadmap Skill

## Objective

Create a `roadmap` skill that reads an activity brief, interviews the user about priorities and sequencing, and produces an adaptive waypoint-based roadmap as a JSON file with waypoint design stubs.

## Done When

- A `skills/roadmap/SKILL.md` exists with frontmatter and a complete checklist
- The skill reads the brief from `docs/brief-<slug>.md`
- The skill conducts a structured interview about priorities, phasing, and sequencing
- The skill produces `docs/roadmap-<slug>.json` (waypoint graph)
- The JSON schema matches the format documented in CLAUDE.md
- The discover skill's Step 11 includes a suggest-and-confirm prompt to invoke roadmap
- W2 (Waypoint Storage Format) is marked done — JSON schema defined and documented
- No auto-generated markdown — the JSON is the source of truth and LLMs read it directly

## Design

### Inputs

The skill needs:
1. **Activity brief** — located at `docs/brief-<slug>.md` in the activity directory
2. **User interview** — priorities, phasing preferences, known dependencies, timeline

The brief provides: goal, success criteria, scope, risks, open questions, background, and roadmap planning notes. The "Notes for Roadmap Planning" section is especially relevant — it contains the user's initial thinking about sequencing and phasing.

### Skill Checklist

#### Step 1: Locate the Brief

- If invoked via suggest-and-confirm from discover, the brief path is known
- If invoked directly, look for `docs/brief-*.md` in the current activity directory
- If multiple briefs exist, ask the user which one
- If no brief exists, tell the user to run discover first
- Read the brief in full

#### Step 2: Expert Analysis (Internal)

Analyze the brief to identify:
1. **Natural phases** — what groups of work form logical stages?
2. **Critical path** — what must happen before other things can start?
3. **Quick wins** — what small items could build early momentum?
4. **Knowledge gaps** — where does the brief lack enough detail for planning?
5. **Risks that affect sequencing** — which risks suggest doing certain work first?
6. **Domain best practices** — what would an expert in this domain recommend for execution order?

Do not present this analysis to the user. Use it to propose an initial structure in Step 3.

#### Step 3: Propose Initial Structure

Present the user with a proposed roadmap structure:
- Suggested phases with rationale
- Key waypoints grouped by phase
- Critical dependencies between waypoints
- Suggested tactical items (unblockers, knowledge gaps, low-hanging fruit)

Frame this as a starting point for discussion, not a final plan. Ask:
> "Here's how I'd structure this. What feels right? What's wrong? What's missing?"

#### Step 4: Iterative Refinement

Refine based on user feedback. This is conversational — not a rigid loop. Topics to cover:
- **Phase ordering** — does the user agree with the proposed sequence?
- **Waypoint granularity** — are waypoints too big, too small, or right-sized?
- **Missing waypoints** — are there things the brief mentions that aren't captured?
- **Dependencies** — are there constraints the skill missed?
- **Tactical items** — small things that unblock progress or fill gaps
- **Priority within phases** — which waypoints matter most within each phase?

Continue until the user says it looks good. Don't over-refine — the roadmap is adaptive and will be replanned.

#### Step 5: Build the Waypoint Graph

For each waypoint, define:
- `id` — short identifier (e.g., `w1`, `t1`)
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

#### Step 6: Write the JSON

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
      "goal": "What this phase achieves",
      "status": "in-progress"
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

Phase status is derived: `done` if all waypoints in the phase are `done`, `in-progress` if any are `in-progress`, `pending` otherwise.

#### Step 7: Create Waypoint Design Stubs

For each waypoint, create `docs/waypoints/<waypoint-id>.md` with:

```markdown
# <Waypoint ID>: <Waypoint Title>

## Objective

<one sentence from done_when>

## Done When

<acceptance criteria from the JSON>
```

These are stubs — the waypoint design skill (or manual design) will flesh them out before decomposition.

#### Step 8: Save and Commit

1. Write the JSON file
2. Create waypoint design stubs
3. Stage and commit with message: `roadmap: create roadmap for <activity-slug>`
4. Report file paths and confirm everything is saved

#### Step 9: Suggest Next Steps

Identify all waypoints with no unresolved dependencies (i.e., ready for design). Present them to the user:

> "The roadmap is saved. These waypoints have no blockers and are ready for design: [list waypoint IDs and titles]. Designing them now will enable parallel execution once the decompose & execute skill is built. Want to start designing them?"

If the waypoint-design skill exists, offer to invoke it. If not, offer to do manual design (create/flesh out the waypoint design docs).

### Invocation

- **Trigger phrases:** "create a roadmap", "plan this activity", "let's build a roadmap", "what should we work on"
- **From discover:** Suggest-and-confirm at the end of discover's Step 11
- **Direct:** User invokes `do-stuff-helper:roadmap` or natural language

### Cross-Skill Invocation

- **`do-stuff-helper:research`** — May invoke during Step 4 if domain-specific sequencing questions arise that benefit from research
- **Invoked by `do-stuff-helper:discover`** — Via suggest-and-confirm after brief is saved

### Edge Cases

- **Activity already has a roadmap:** Ask the user if they want to replace it or update it. For now, replacing is fine — the backlog/replanning skill (W9) will handle incremental updates later.
- **Brief is thin:** If the brief lacks enough detail for meaningful waypoints, flag specific gaps and ask the user to fill them (or suggest re-running discover).
- **Very small activity:** Some activities may only need 3-5 waypoints with no phases. The skill should scale down — don't force phases on a simple activity.
- **Very large activity:** For complex activities, suggest keeping the first phase detailed and later phases coarser. They'll be refined when we get there.

## Dependencies

- W2 (Waypoint Storage Format) — the JSON schema must be finalized. **Status: decisions made, needs implementation as part of this waypoint.**

## Implementation Notes

- The JSON schema defined in Step 6 above IS the W2 deliverable — implementing W1 completes W2
- The suggest-and-confirm transition from discover (T4) should be implemented as part of this work
- Phase status should be computed, not manually set — derive from waypoint statuses
- No auto-generated markdown — the JSON is readable by LLMs directly, and a human-readable view can be generated on-demand later if needed (e.g., dashboard)
