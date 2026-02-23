# W4: Decompose & Execute Architecture

## Objective

Build the two-skill execution engine — waypoint-planner for recursive task decomposition into a DAG, and waypoint-implement for orchestrating parallel workers — using Claude Code's native persistent task system for cross-session coordination.

## Done When

- A `waypoint-planner` skill exists that reads a designed waypoint and recursively decomposes it into a task DAG using native TaskCreate with dependencies
- A `waypoint-implement` skill exists that orchestrates execution: dispatches parallel workers in worktrees, handles failure/retry, surfaces human tasks, merges code changes
- Tasks persist across sessions via `CLAUDE_CODE_TASK_LIST_ID` — no custom task storage
- Tasks are tagged with waypoint ID via metadata
- Human-assigned tasks are surfaced to the user and treated as blockers
- A changelog at `docs/changelog.md` records what was done per waypoint
- Non-code outputs (research, etc.) are written to files, referenced from task metadata, and linked from relevant docs
- Workers can flag genuine blocking questions back to the manager for user input
- At least one waypoint has been planned and implemented end-to-end to validate the architecture
- The waypoint-design skill is updated to check waypoint coherence (unit of value) before handoff to planning
- The organize skill is updated to set `CLAUDE_CODE_TASK_LIST_ID` in `.claude/settings.json` and note it in the activity's CLAUDE.md, and is idempotent

## Design

### Native Task System

All tasks use Claude Code's built-in TaskCreate/TaskUpdate/TaskList/TaskGet with `CLAUDE_CODE_TASK_LIST_ID` set to the activity slug (e.g., `do-stuff-helper`).

This gives us:
- **Persistence** — tasks survive across sessions, stored at `~/.claude/tasks/{id}/`
- **DAG dependencies** — `addBlockedBy`/`addBlocks` with automatic unblocking
- **Concurrent access** — file locking for multi-session safety
- **Ownership** — `owner` field for task claiming

The organize skill sets `CLAUDE_CODE_TASK_LIST_ID` in the activity's `.claude/settings.json` so every session in that directory automatically uses the right task list. The organize skill is idempotent — running it again updates settings without duplicating work.

### Task Model

Tasks have two assignee types: `agent` and `human`. That's it — no rigid type enum. The kinds of work agents do will constantly expand as tools and skills grow.

Every task created by waypoint-planner includes:

- `subject` — imperative title, prefixed with waypoint ID: `[w4] Create waypoint-planner skill`
- `description` — detailed requirements, file paths, acceptance criteria
- `activeForm` — present-continuous form for progress display
- `metadata`:
  - `waypoint`: waypoint ID (e.g., `"w4"`)
  - `assignee`: `"agent"` or `"human"`
  - `output_path`: (optional) where non-code output should be written
  - `context_files`: array of file paths the worker should read
  - `link_from`: (optional) array of docs that should link to the output when done

**Human tasks** are created with a clear description of what the human needs to do. The implement skill surfaces these and treats them as blockers for dependent tasks. What constitutes a "human task" should reference the output of W5 (Autonomy Model), which defines what agents can do on their own. Until W5 is complete, reasonable defaults apply: agents handle code, research, writing, and configuration; humans handle external communications, account creation, physical actions, and decisions with significant irreversible consequences.

### All Workers Get Worktrees

Every agent worker runs in a git worktree, regardless of what kind of work it does. Any worker that writes files to the activity directory is making changes in git. Universal worktree isolation means:
- No merge conflicts from parallel workers
- The implement skill doesn't need to reason about which tasks need isolation
- Failed work is trivially discarded
- The model is simple: every worker is isolated, always

### Skill 1: waypoint-planner

**Trigger**: After waypoint-design saves a design doc, or when user asks to plan/decompose a waypoint.

**Checklist**:

1. **Select waypoint** — user specifies, or identify designed waypoints ready for planning
2. **Read the waypoint design doc** — `docs/waypoints/<waypoint-id>.md`
3. **Verify task list is configured** — check that `CLAUDE_CODE_TASK_LIST_ID` is set (the organize skill should have done this). If not, instruct the user to run the organize skill or set it manually
4. **Recursive decomposition** — break the waypoint into a task DAG:
   - Start with the top-level deliverables from the design doc
   - For each deliverable, ask: can this be done in a single focused session by one worker? If not, decompose further
   - Continue until all leaf nodes are atomic, executable tasks
   - Identify dependencies between tasks (what must complete before what can start)
   - Identify parallelism (independent tasks that can run simultaneously)
   - Identify human tasks — things that require human action (see Human Tasks guidance above)
5. **Present the DAG to the user** — show the task tree with dependencies, parallelism groups, and human tasks called out. Ask for confirmation
6. **Create tasks** — use TaskCreate for each leaf task with proper metadata, then TaskUpdate to wire up dependencies via `addBlockedBy`/`addBlocks`
7. **Suggest next step** — offer to invoke waypoint-implement, or let the user review first

### Skill 2: waypoint-implement

**Trigger**: After waypoint-planner creates tasks, or when user wants to continue executing a waypoint.

**Checklist**:

1. **Read the task list** — TaskList to see current state. Filter by waypoint metadata if implementing a specific waypoint
2. **Surface human tasks** — identify any unblocked human tasks and present them to the user. Note what downstream tasks are waiting on them
3. **Check for worker questions** — check if any tasks have been returned with questions in metadata (see Worker Q&A below). Present questions to the user, get answers, update task descriptions with the answers
4. **Identify ready agent tasks** — pending tasks with no unresolved dependencies and `assignee: "agent"`
5. **Dispatch workers** — for each ready task, spawn via Task tool with `isolation: "worktree"` and `subagent_type: "general-purpose"`. Worker prompt includes:
   - Task description and acceptance criteria
   - Context files from metadata
   - Activity conventions from CLAUDE.md
   - Instruction: "If you encounter ambiguity, make a reasonable decision and document your assumption. Only stop and ask if you truly cannot proceed without user input."
   - Dispatch independent tasks in parallel
6. **Collect results** — as workers return:
   - **Success**: TaskUpdate status to `completed`. Merge the worktree branch. If `link_from` is set in metadata, update those docs with a link to the output. Append to changelog
   - **Question**: Worker flagged a genuine blocking question. Update task metadata with the question, set status back to `pending`. Will be surfaced on next loop iteration
   - **Failure**: Report failure to user with details. Decide: retry (transient), re-decompose (too complex), or escalate (needs human). Dependent tasks remain blocked
7. **Loop** — repeat steps 2-6 until:
   - All tasks complete (waypoint done)
   - Only human tasks remain (surface them and pause)
   - A failure blocks further progress (escalate to user)
8. **Complete waypoint** — when all tasks are done, update `docs/roadmap-<slug>.json` waypoint status to `done`. Append a summary entry to the changelog

**Worker Q&A protocol**:
Workers should bias heavily toward action. When they encounter ambiguity:
- **Default**: Make a reasonable assumption, document it in commit messages and output, keep going. Assumptions can be revisited later.
- **Only ask** when they truly cannot proceed — e.g., two valid approaches with significantly different consequences, or missing information that can't be inferred.
- To ask: include a `question` field in the task result/output describing what's needed and why they can't proceed. The implement skill picks this up and routes it to the user.

**Parallelism guidance**:
- Independent tasks (no shared dependencies in the DAG) dispatch in parallel
- The DAG is the source of truth for ordering — if two tasks shouldn't run concurrently, the planner must encode that as a dependency
- The implement skill respects the DAG strictly — never dispatches a task whose dependencies aren't complete

### Changelog

A single file at `docs/changelog.md` with waypoint headings:

```markdown
# Changelog

## W4: Decompose & Execute Architecture

### 2026-02-23 - [t1] Create waypoint-planner skill
- Created `skills/waypoint-planner/SKILL.md` with full checklist
- Assumption: used depth limit of 4 levels for recursive decomposition
- Files changed: skills/waypoint-planner/SKILL.md

### 2026-02-23 - [t2] Research task list persistence
- Confirmed CLAUDE_CODE_TASK_LIST_ID enables cross-session persistence
- Output: docs/research/task-list-persistence.md
- Linked from: docs/waypoints/w4-decompose-execute-architecture.md
```

### Updates to Existing Skills

**waypoint-design skill** — Add to Step 6 (Evaluate the Design):
- [ ] **Coherent** — does this waypoint represent a single coherent unit of *value* (not just work)? Could the value it delivers stand on its own? If the waypoint mixes unrelated value streams, it should be split before handoff to planning.

**organize skill** — Add:
- Set `CLAUDE_CODE_TASK_LIST_ID` in the activity's `.claude/settings.json` (value = activity slug)
- Note the task list ID in the activity's CLAUDE.md
- Make the entire skill idempotent — running it again updates settings, fills gaps, but doesn't duplicate or overwrite existing content

### Key Decisions

| Decision | Choice | Why |
|----------|--------|-----|
| Task system | Native Claude Code tasks with `CLAUDE_CODE_TASK_LIST_ID` | Persistent, supports DAGs, concurrent access, no custom storage needed |
| Two skills | waypoint-planner + waypoint-implement | Clean separation: planning is cognitive, execution is operational. Different sessions may do each |
| Recursive decomposition | In the planner, not workers | Workers execute leaf tasks only. Keeps workers simple and predictable |
| Assignee model | `agent` vs `human` only | No rigid type enum. What agents can do will expand as tools and skills grow. W5 will formalize this |
| Universal worktrees | Every agent worker gets a worktree | Any file write is a git change. Universal isolation eliminates conflict reasoning and simplifies the model |
| Worker Q&A | Bias toward action, only ask when truly blocked | Workers document assumptions and keep going. Questions are routed through the implement skill to the user in the main console |
| Changelog | Single `docs/changelog.md` with waypoint headings | Easy to scan, easy to parse, avoids file proliferation |
| Agent teams | Not in V1, future waypoint | Current design is compatible — same `CLAUDE_CODE_TASK_LIST_ID` mechanism. Clear migration path |

### Out of Scope

- **Permission system** — W5 (Autonomy Model)
- **Quality review** — W13 (Work Checker)
- **Async inbox** — W11 (adds async Q&A beyond the main console)
- **Feedback on output** — W7 (Feedback Capture)
- **Agent teams** — future waypoint, will enhance waypoint-implement

### Implementation Sequence

1. Update organize skill (task list ID in settings, idempotent)
2. Update waypoint-design skill (coherence = unit of value check)
3. Build waypoint-planner skill (recursive decomposition into TaskCreate DAG)
4. Build waypoint-implement skill (dispatch, collect, merge, changelog, Q&A)
5. Test end-to-end on a small waypoint (W19 or W20)

### Cross-Skill Invocation

- **Invoked after `do-stuff-helper:waypoint-design`** — design suggests planning after save
- **`waypoint-planner` invokes `waypoint-implement`** — via suggest-and-confirm after planning
- **Leverages `superpowers:dispatching-parallel-agents`** — for parallel dispatch pattern
- **Enables W5** (Autonomy Model), **W7** (Feedback Capture), **W13** (Work Checker)
