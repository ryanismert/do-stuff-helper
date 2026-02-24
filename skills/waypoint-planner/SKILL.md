---
name: waypoint-planner
description: This skill should be used when the user asks to "plan a waypoint", "decompose a waypoint", "break down a waypoint into tasks", or when transitioning from waypoint-design after a design is saved. Reads a designed waypoint and recursively decomposes it into a task DAG using Claude Code's native task system. Do NOT trigger for roadmap-level planning — use roadmap skill for that.
---

# Waypoint Planner

Read a designed waypoint and recursively decompose it into a task DAG using Claude Code's native TaskCreate. Tasks are tagged with waypoint metadata, have proper dependencies via addBlockedBy/addBlocks, and persist across sessions via `CLAUDE_CODE_TASK_LIST_ID`.

## Checklist

Complete each step in strict order. Do not skip steps.

### Step 1: Select Waypoint

- If the user specifies a waypoint ID, use that one
- If not, read `docs/roadmap-<slug>.json` and identify waypoints where:
  - Status is `pending` or has a design doc beyond the stub
  - All dependencies are `done`
  - Has a design doc at `docs/waypoints/<waypoint-id>.md` with content beyond the stub (Objective + Done When only = stub)
- Present candidates and ask the user which to plan
- If no candidates, tell the user to run waypoint-design first and stop

### Step 2: Read the Waypoint Design Doc

- Read `docs/waypoints/<waypoint-id>.md` in full
- Also read the activity brief (`docs/brief-<slug>.md`) for context
- Read design docs of dependency waypoints to understand what's already built

### Step 3: Verify Task List is Configured

- Check that `CLAUDE_CODE_TASK_LIST_ID` is set (environment variable)
- If not, check `.claude/settings.json` for the env var configuration
- If not configured, instruct the user to run the organize skill or set it manually
- Do not proceed until the task list is confirmed

### Step 4: Recursive Decomposition

If the design doc leaves anything ambiguous that would significantly affect the task structure, ask the user before proceeding. Otherwise, use your best judgment and go straight to creating tasks — the user trusts the planner and can review or adjust tasks after creation.

Start with the top-level deliverables from the design doc's "Done When" criteria. For each deliverable, ask: can this be done in a single focused session by one worker? If not, decompose further. Continue until all leaf nodes are atomic, executable tasks.

Identify:
- **Dependencies** between tasks (what must complete before what can start)
- **Parallelism** (independent tasks that can run simultaneously)
- **Human tasks** (things that require human action)

**Human task heuristics** (until W5 Autonomy Model formalizes this):
- Agents handle: code changes, research, writing, file configuration, documentation
- Humans handle: external communications (emails, phone calls), account creation, physical actions, purchases, decisions with significant irreversible consequences
- When in doubt, make it an agent task — the implement skill can escalate if needed

**Task model** — each leaf task needs:
- `subject`: imperative title prefixed with waypoint ID — `[<waypoint-id>] <task title>`
- `description`: detailed requirements including file paths, acceptance criteria, and enough context for a worker with no prior knowledge
- `activeForm`: present-continuous form for progress display
- `metadata`:
  - `waypoint`: waypoint ID (e.g., "w4")
  - `assignee`: "agent" or "human"
  - `output_path`: (optional) where non-code output should be written
  - `context_files`: array of file paths the worker should read for context
  - `link_from`: (optional) array of docs that should link to the output when done

### Step 5: Create Tasks

- For each leaf task, call `TaskCreate` with subject, description, activeForm, and metadata
- After all tasks are created, use `TaskUpdate` to wire up dependencies via `addBlockedBy`/`addBlocks`
- Report the task IDs created

### Step 6: Suggest Next Step

Offer to invoke `do-stuff-helper:waypoint-implement` to start execution, or let the user review the task list first. Suggest-and-confirm pattern:

> "Tasks created. Want to start implementing this waypoint now? (invokes waypoint-implement)"

## Edge Cases

- **Existing tasks for this waypoint:** Check TaskList for tasks with matching waypoint metadata. If found, ask the user: clear and re-plan, or add to existing?
- **Waypoint design is a stub:** Tell the user to run waypoint-design first and stop.
- **Decomposition reveals missing dependency:** Flag and suggest updating the roadmap.
- **Very large decomposition:** It's OK for waypoints to have many tasks as long as they're coherent. The waypoint-design skill handles coherence checking.

## Cross-Skill Invocation

- **Invoked by `do-stuff-helper:waypoint-design`** — After saving a design (Step 9), the user can choose to plan the waypoint.
- **`do-stuff-helper:waypoint-implement`** — Suggest in Step 7 after tasks are created: offer to start execution.
- **`do-stuff-helper:research`** — May invoke during Step 4 if decomposition reveals knowledge gaps that need investigation.
