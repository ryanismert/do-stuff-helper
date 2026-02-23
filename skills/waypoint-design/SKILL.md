---
name: waypoint-design
description: This skill should be used when the user asks to "design a waypoint", "flesh out a waypoint", "detail a waypoint", or when a waypoint needs a design document before it can be decomposed into tasks. Takes a waypoint stub and produces a design document scaled to the waypoint's complexity. Do NOT trigger for general task work or execution — only for waypoint design.
---

# Waypoint Design

Take a waypoint from the roadmap and produce a design document with enough detail for confident decomposition into parallelizable tasks. Scale effort to complexity — simple waypoints get a sentence, complex ones get full designs.

## Checklist

### Step 1: Select the Waypoint

- If the user specifies a waypoint, use that one
- If not, read `docs/roadmap-<slug>.json` and identify waypoints that are:
  - Status `pending`
  - All dependencies resolved (status `done`)
  - No existing design beyond the stub
- Present the unblocked waypoints and ask the user which to design. If multiple are unblocked, suggest designing them in dependency order (or in parallel if independent)

### Step 2: Gather Context

Read the following to understand what this waypoint needs:
1. The waypoint's entry in `docs/roadmap-<slug>.json` (why, done_when, dependencies)
2. The existing stub at `docs/waypoints/<waypoint-id>.md`
3. The activity brief at `docs/brief-<slug>.md`
4. Design docs of completed dependency waypoints (to understand what's already built)

### Step 3: Assess Complexity

Evaluate the waypoint on these dimensions:
- **Ambiguity** — is it clear what needs to be built, or are there open design questions?
- **Scope** — how many components, files, or systems does it touch?
- **Novelty** — is this well-understood work, or does it require research?
- **Risk** — what could go wrong? Are there irreversible decisions?

Based on this assessment, choose a design depth:

**Minimal** (ambiguity: low, scope: small, novelty: low) — The stub is nearly sufficient. Add a few sentences of implementation notes and move on.

**Standard** (moderate on any dimension) — Flesh out the design with inputs, outputs, key decisions, and a rough approach. A few paragraphs.

**Detailed** (high on any dimension) — Full design document with architecture, edge cases, alternatives considered, and open questions resolved. This is what the W1: Roadmap Skill design doc looks like.

Do not tell the user the complexity label — just produce the appropriate level of detail.

### Step 4: Research (if needed)

If the waypoint involves novelty or open questions that need external information:
- Invoke `do-stuff-helper:research` for domain-specific questions
- Use the Task tool with parallel agents to investigate multiple angles simultaneously

Skip this step if the waypoint is well-understood.

### Step 5: Design the Waypoint

Write the design document. The required sections are:

```markdown
# <Waypoint ID>: <Waypoint Title>

## Objective

<one sentence stating what this waypoint achieves>

## Done When

<concrete acceptance criteria — bulleted list>
```

Everything beyond these two sections is freeform, scaled to complexity. For **standard** and **detailed** waypoints, consider including:

- **Design** — how it works, architecture, key components
- **Inputs / Outputs** — what it consumes and produces
- **Key Decisions** — choices made and why (alternatives considered for detailed)
- **Edge Cases** — what could go wrong, how to handle it
- **Dependencies** — what it builds on, what it enables
- **Implementation Notes** — hints for the implementer, gotchas, sequencing

For **detailed** waypoints, also consider:
- **Skill Checklist** — if the waypoint produces a skill, draft the step-by-step checklist
- **Schema / Format** — if the waypoint defines data structures, specify them
- **Cross-Skill Invocation** — how this interacts with other skills

### Step 6: Decompose into Tasks

Use the TaskCreate tool to break the design into concrete, parallelizable tasks. Each task should be:
- **Specific** — clear enough that an agent could execute it without asking questions
- **Scoped** — small enough to complete in one focused session
- **Independent where possible** — minimize sequential dependencies between tasks

For **minimal** waypoints, this may be a single task. For **detailed** waypoints, this may be 3-8 tasks with dependency ordering via TaskUpdate (addBlockedBy/addBlocks).

Mark all tasks as `pending`. The user decides when to start execution.

### Step 7: Present and Confirm

Present the design to the user. Ask:

> "Here's the design for [waypoint title]. Does this look right, or should we adjust anything before I save it?"

If the user has feedback, incorporate it and update the tasks accordingly.

### Step 8: Save

1. Write the design document to `docs/waypoints/<waypoint-id>.md`
2. Stage and commit with message: `waypoint-design: design <waypoint-id> <waypoint-title>`
3. Report the file path and confirm saved

### Step 9: Suggest Next Steps

After saving, check the roadmap for other unblocked waypoints that are ready for design:

> "Design saved. [Other unblocked waypoints] are also ready for design. Want to continue with another, or start executing tasks for this waypoint?"

## Edge Cases

- **Waypoint has no stub:** Create the design doc from scratch using the JSON entry.
- **Waypoint is already fully designed:** Tell the user and ask if they want to revise it.
- **Design reveals the waypoint should be split:** Propose splitting to the user. If approved, update the roadmap JSON (add new waypoints, update dependencies) and design each piece.
- **Design reveals missing dependencies:** Flag to the user and suggest updating the roadmap.

## Cross-Skill Invocation

- **`do-stuff-helper:research`** — Invoke during Step 4 when external research would improve the design.
- **Invoked by `do-stuff-helper:roadmap`** — The roadmap skill's Step 9 suggests designing unblocked waypoints, which leads here.
