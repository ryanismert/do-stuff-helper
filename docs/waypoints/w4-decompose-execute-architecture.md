# W4: Decompose & Execute Architecture

## Objective

Build the core engine for recursive task decomposition and parallel worker execution from designed waypoints.

## Done When

A manager agent can take a designed waypoint, decompose it into parallelizable tasks, and orchestrate worker agents to execute them. Workers operate in their own context and report status back.

## Design Notes (from planning sessions)

- **Waypoint parallelism:** Waypoints should NOT block on each other for execution — multiple waypoints can be in-progress simultaneously. Don't require full completion of one waypoint before starting another.
- **Task-to-waypoint tracking:** Tasks must be tagged with their waypoint ID so we know which tasks belong to which waypoint when multiple are in-flight.
- **Task decomposition uses TaskCreate:** The decompose step should use Claude Code's Task tools (TaskCreate, TaskUpdate, TaskList) for tracking individual execution tasks.
- **This is the most architecturally complex waypoint** — will need significant design work before implementation.
