# W4: Decompose & Execute Architecture — Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Build waypoint-planner and waypoint-implement skills, update organize and waypoint-design skills, to complete the core execution pipeline.

**Architecture:** Two new skills (waypoint-planner, waypoint-implement) use Claude Code's native persistent task system via `CLAUDE_CODE_TASK_LIST_ID`. The planner recursively decomposes waypoints into task DAGs. The implementer orchestrates parallel workers in git worktrees, handles failures, surfaces human tasks, and maintains a changelog.

**Tech Stack:** Claude Code skills (SKILL.md markdown), Claude Code Task tools (TaskCreate/TaskUpdate/TaskList/TaskGet), git worktrees via Task tool isolation

---

## Task 1: Make Organize Skill Idempotent and Add Task List Configuration

**Files:**
- Modify: `skills/organize/SKILL.md`

**Context:** The organize skill currently runs once to bootstrap an activity directory. It needs to: (1) be safe to re-run (idempotent), and (2) configure `CLAUDE_CODE_TASK_LIST_ID` for the activity.

**Step 1: Read the current organize skill**

Read `skills/organize/SKILL.md` to understand the current structure.

**Step 2: Add idempotency to existing steps**

Update each step to check-before-acting:
- Step 1 (Choose Directory): No change needed — already asks.
- Step 2 (Add to Scope): Already checks if needed. No change.
- Step 3 (Create docs/): Already uses `mkdir -p`. No change.
- Step 4 (Install Plugins): Already checks if installed. No change.
- Step 5 (Initialize CLAUDE.md): Already skips if exists. **Change**: Instead of skipping entirely, check if the do-stuff-helper section exists and update it if the content has changed (new skills added, etc.). Only skip if content matches.
- Step 6 (Git/GitHub): Already handles all cases. No change.

**Step 3: Add new Step 5.5 — Configure Task List ID**

Insert a new step between the current Step 5 and Step 6. This step:

1. Derives the task list ID from the activity slug (directory basename)
2. Creates or updates `.claude/settings.json` in the activity directory:
   ```json
   {
     "env": {
       "CLAUDE_CODE_TASK_LIST_ID": "<activity-slug>"
     }
   }
   ```
3. If `.claude/settings.json` already exists, merges the `env.CLAUDE_CODE_TASK_LIST_ID` key without overwriting other settings
4. Adds a note to the activity's CLAUDE.md under the do-stuff-helper section:
   ```markdown
   ### Task List
   This activity uses `CLAUDE_CODE_TASK_LIST_ID=<activity-slug>` for persistent cross-session task tracking.
   ```

**Step 4: Update CLAUDE.md section to be idempotent**

Change Step 5 so that instead of "skip if CLAUDE.md exists", it:
1. Reads the existing CLAUDE.md
2. Checks if the `## do-stuff-helper` section exists
3. If missing, appends it
4. If present, updates it with the current skill list (organize, discover, research, roadmap, waypoint-design, waypoint-planner, waypoint-implement)
5. Ensures the Task List subsection exists with the correct task list ID

**Step 5: Update the confirmation step**

Add to the Step 7 summary:
- Whether `.claude/settings.json` was created/updated
- The task list ID configured

**Step 6: Commit**

```bash
git add skills/organize/SKILL.md
git commit -m "feat: make organize skill idempotent, add task list configuration"
```

---

## Task 2: Update Waypoint-Design Skill with Coherence Check

**Files:**
- Modify: `skills/waypoint-design/SKILL.md`

**Context:** The waypoint-design skill's Step 6 (Evaluate the Design) needs a new check: does this waypoint represent a coherent unit of *value*?

**Step 1: Read the current waypoint-design skill**

Read `skills/waypoint-design/SKILL.md` to find Step 6.

**Step 2: Add coherence check to Step 6**

In Step 6 (Evaluate the Design), add this item to the checklist:

```markdown
- [ ] **Coherent** — does this waypoint represent a single coherent unit of *value* (not just work)? Could the value it delivers stand on its own? If the waypoint mixes unrelated value streams, it should be split before handoff to planning.
```

**Step 3: Update the post-check behavior**

After the existing text "If any check fails, go back to Step 5...", add:

```markdown
If the coherence check fails, propose splitting the waypoint to the user before continuing. If approved, update the roadmap JSON (add new waypoints, update dependencies) and design each piece separately.
```

**Step 4: Update the cross-skill invocation section**

Add that waypoint-design suggests invoking waypoint-planner after saving:

```markdown
- **`do-stuff-helper:waypoint-planner`** — Suggest in Step 9 after saving: "Design saved. Want to plan the tasks for this waypoint?"
```

**Step 5: Commit**

```bash
git add skills/waypoint-design/SKILL.md
git commit -m "feat: add coherence check and planner handoff to waypoint-design"
```

---

## Task 3: Build Waypoint-Planner Skill

**Files:**
- Create: `skills/waypoint-planner/SKILL.md`

**Context:** This skill reads a designed waypoint and recursively decomposes it into a task DAG using Claude Code's native TaskCreate with dependencies. Full design in `docs/waypoints/w4-decompose-execute-architecture.md` under "Skill 1: waypoint-planner".

**Step 1: Create the skill directory**

```bash
mkdir -p skills/waypoint-planner
```

**Step 2: Write the SKILL.md frontmatter and overview**

```markdown
---
name: waypoint-planner
description: This skill should be used when the user asks to "plan a waypoint", "decompose a waypoint", "break down a waypoint into tasks", or when transitioning from waypoint-design after a design is saved. Reads a designed waypoint and recursively decomposes it into a task DAG using Claude Code's native task system. Do NOT trigger for roadmap-level planning — use roadmap skill for that.
---

# Waypoint Planner

Read a designed waypoint and recursively decompose it into a task DAG using Claude Code's native TaskCreate. Tasks are tagged with waypoint metadata, have proper dependencies, and persist across sessions via `CLAUDE_CODE_TASK_LIST_ID`.
```

**Step 3: Write the checklist**

The checklist implements the 7 steps from the W4 design doc's "Skill 1: waypoint-planner" section:

1. **Select waypoint** — user specifies or auto-detect designed waypoints ready for planning. Check `docs/roadmap-<slug>.json` for waypoints with status `pending` or `designed` whose dependencies are `done`. Check if `docs/waypoints/<waypoint-id>.md` has content beyond the stub.
2. **Read the waypoint design doc** — read `docs/waypoints/<waypoint-id>.md` in full. Also read the brief and any dependency waypoint designs for context.
3. **Verify task list is configured** — check that `CLAUDE_CODE_TASK_LIST_ID` is set. If not, check `.claude/settings.json` for the env var. If not configured, instruct the user to run the organize skill or set it manually.
4. **Recursive decomposition** — full decomposition logic per the design doc. Key details:
   - Start with top-level deliverables from "Done When" criteria
   - For each deliverable: can one worker do this in a single focused session? If not, break it down further
   - Leaf nodes must be atomic and executable by a single worker
   - Tag each task as `agent` or `human` (default heuristics: agents do code, research, writing, config; humans do external comms, account creation, physical actions, irreversible high-stakes decisions — reference W5 once it exists)
   - Map dependencies: which tasks must complete before others can start?
   - Identify parallelism groups: independent tasks that can run simultaneously
5. **Present the DAG to the user** — render the task tree showing:
   - Task hierarchy with indentation
   - Dependencies (what blocks what)
   - Parallelism groups marked
   - Human tasks flagged with a marker
   - Ask: "Does this decomposition look right? Any tasks missing, too big, or wrongly assigned?"
6. **Create tasks** — for each leaf task:
   - `TaskCreate` with subject `[<waypoint-id>] <task title>`, description (full requirements, file paths, acceptance criteria), activeForm, and metadata (`waypoint`, `assignee`, `output_path`, `context_files`, `link_from`)
   - After all created, use `TaskUpdate` to wire `addBlockedBy`/`addBlocks` for the dependency edges
7. **Suggest next step** — offer to invoke `do-stuff-helper:waypoint-implement` or let user review the task list first

**Step 4: Write edge cases section**

- Existing tasks for this waypoint: ask user if they want to clear and re-plan, or add to existing
- Waypoint design is a stub: tell user to run waypoint-design first
- Decomposition reveals missing dependency: flag and suggest updating the roadmap

**Step 5: Write cross-skill invocation section**

- Invoked by `do-stuff-helper:waypoint-design` after saving a design
- Invokes `do-stuff-helper:waypoint-implement` via suggest-and-confirm
- May invoke `do-stuff-helper:research` if decomposition reveals knowledge gaps

**Step 6: Commit**

```bash
git add skills/waypoint-planner/SKILL.md
git commit -m "feat: create waypoint-planner skill for recursive task decomposition"
```

---

## Task 4: Build Waypoint-Implement Skill

**Files:**
- Create: `skills/waypoint-implement/SKILL.md`

**Context:** This skill orchestrates execution of tasks created by waypoint-planner. It dispatches workers in worktrees, handles failures, surfaces human tasks, manages the changelog, and routes worker questions to the user. Full design in `docs/waypoints/w4-decompose-execute-architecture.md` under "Skill 2: waypoint-implement".

**Step 1: Create the skill directory**

```bash
mkdir -p skills/waypoint-implement
```

**Step 2: Write the SKILL.md frontmatter and overview**

```markdown
---
name: waypoint-implement
description: This skill should be used when the user asks to "implement a waypoint", "execute tasks", "run the workers", "continue working on <waypoint>", or when transitioning from waypoint-planner after tasks are created. Orchestrates parallel worker agents to execute tasks from the task list. Do NOT trigger for planning or decomposition — use waypoint-planner for that.
---

# Waypoint Implement

Orchestrate execution of tasks from the native task list. Dispatch parallel worker agents in git worktrees, handle failures and retries, surface human tasks as blockers, route worker questions to the user, merge completed work, and maintain the changelog.
```

**Step 3: Write the main execution loop checklist**

Implements the 8 steps from the W4 design doc's "Skill 2: waypoint-implement" section:

1. **Read the task list** — `TaskList` to see current state. If user specified a waypoint, use `TaskGet` on each task to filter by `metadata.waypoint`. Show a summary: X total, Y done, Z pending, W in-progress, H human.
2. **Surface human tasks** — find tasks with `metadata.assignee == "human"` that are not blocked. Present them clearly: "These tasks need you: [list]. [downstream task count] other tasks are waiting on them."
3. **Check for worker questions** — find tasks with `metadata.question` set. Present each question to the user. After user answers, update the task description with the answer (append an "Additional Context" section), clear the question metadata, and re-queue for dispatch.
4. **Identify ready agent tasks** — pending tasks where: `metadata.assignee == "agent"`, all `blockedBy` tasks are completed, no `metadata.question` set.
5. **Dispatch workers** — for each ready task, spawn via `Task` tool:
   - `subagent_type: "general-purpose"`
   - `isolation: "worktree"`
   - Prompt construction (detailed in step):
     - Include the full task description
     - Include content of each file listed in `metadata.context_files`
     - Include the activity's CLAUDE.md
     - Include: "Bias toward action. If you encounter ambiguity, make a reasonable decision and document your assumption in your commit message. Only flag a question if you truly cannot proceed — state what you need and why in your response."
     - Include: "When done, summarize what you did, what files you changed, and any assumptions you made."
   - Dispatch independent tasks in parallel (multiple Task tool calls in one message)
6. **Collect results and merge** — as each worker returns:
   - **Success** (worker completed, worktree has changes):
     - `TaskUpdate` status to `completed`
     - Merge the worktree branch: `git merge <branch> --no-edit`
     - If merge conflict: attempt auto-resolution, escalate to user if it fails
     - If `metadata.link_from` is set, update those docs with a link to `metadata.output_path`
     - Append entry to `docs/changelog.md` under the waypoint heading
   - **Success** (worker completed, no file changes):
     - `TaskUpdate` status to `completed`
     - Append entry to changelog noting what was done
   - **Question** (worker returned saying it can't proceed):
     - Set `metadata.question` on the task with the worker's question
     - Set status back to `pending`
     - Will be surfaced on next loop iteration
   - **Failure** (worker errored or couldn't complete):
     - Report failure details to user
     - Ask user: retry the task, re-decompose it into subtasks, or mark as blocked?
     - Dependent tasks remain blocked
7. **Loop** — repeat steps 2-6 until:
   - All tasks for this waypoint are `completed` → proceed to step 8
   - Only human tasks remain → surface them and pause: "All agent tasks are done. These human tasks remain: [list]. Resume when you've completed them."
   - A failure blocks all remaining progress → escalate: "Execution is blocked. [details]. What would you like to do?"
8. **Complete waypoint** — when all tasks are done:
   - Update `docs/roadmap-<slug>.json`: set waypoint status to `done`, update the `updated` date
   - Append a completion summary to the changelog
   - Report: "Waypoint [title] is complete. [summary of what was done]"
   - Check for other waypoints that are now unblocked and suggest next steps

**Step 4: Write the changelog management section**

The skill manages `docs/changelog.md`:
- If the file doesn't exist, create it with `# Changelog` header
- If the waypoint heading doesn't exist, add `## <Waypoint ID>: <Waypoint Title>`
- Each task completion appends an entry under the waypoint heading:
  ```markdown
  ### YYYY-MM-DD - [<task-id>] <task subject>
  - <summary of what was done>
  - Assumptions: <any assumptions documented>
  - Files changed: <list>
  ```

**Step 5: Write edge cases section**

- No tasks exist for the waypoint: tell user to run waypoint-planner first
- All tasks already completed: tell user the waypoint is done
- Session ends mid-execution: tasks persist via `CLAUDE_CODE_TASK_LIST_ID`. User can re-invoke to resume
- Worker creates merge conflict: attempt auto-merge, escalate with diff if it fails
- Task has no `context_files`: worker gets just the task description and CLAUDE.md

**Step 6: Write cross-skill invocation section**

- Invoked by `do-stuff-helper:waypoint-planner` via suggest-and-confirm
- Uses `superpowers:dispatching-parallel-agents` pattern for parallel dispatch
- Future: W24 (Agent Teams) will enhance the dispatch mechanism

**Step 7: Commit**

```bash
git add skills/waypoint-implement/SKILL.md
git commit -m "feat: create waypoint-implement skill for worker orchestration"
```

---

## Task 5: Update Plugin Version

**Files:**
- Modify: `plugin.json`
- Modify: `.claude-plugin/marketplace.json`

**Step 1: Read current versions**

Read `plugin.json` and `.claude-plugin/marketplace.json`.

**Step 2: Bump version**

Bump from `0.4.0` to `0.5.0` (new skills = minor version bump) in both files.

**Step 3: Commit**

```bash
git add plugin.json .claude-plugin/marketplace.json
git commit -m "chore: bump version to 0.5.0 for W4 skills"
```

---

## Task 6: End-to-End Validation

**Context:** Test the full pipeline by planning and implementing a small waypoint (W19: Fix Discover Skill Invocation Bug or W20: Plugin Update Workflow).

**Step 1: Pick a test waypoint**

Choose W19 or W20 — whichever is simpler. Both have no dependencies and are small.

**Step 2: Design the waypoint (if not already designed)**

Invoke `do-stuff-helper:waypoint-design` on the chosen waypoint.

**Step 3: Plan the waypoint**

Invoke `do-stuff-helper:waypoint-planner` on the designed waypoint. Verify:
- Tasks are created via TaskCreate
- Dependencies are wired correctly
- Metadata is populated (waypoint, assignee, context_files)
- The DAG is presented clearly

**Step 4: Implement the waypoint**

Invoke `do-stuff-helper:waypoint-implement`. Verify:
- Workers are dispatched in worktrees
- Results are collected and merged
- Changelog is updated
- Waypoint status is updated in the roadmap JSON

**Step 5: Review results**

Check:
- `docs/changelog.md` has entries for the completed tasks
- `docs/roadmap-do-stuff-helper.json` shows the waypoint as `done`
- All task changes are merged into main
- No leftover worktree branches

**Step 6: Commit any fixes**

If the validation revealed issues, fix them and commit.

---

## Dependency Graph

```
Task 1 (organize) ─┐
                    ├─→ Task 3 (waypoint-planner) ─→ Task 4 (waypoint-implement) ─→ Task 5 (version) ─→ Task 6 (e2e test)
Task 2 (waypoint-design) ─┘
```

Tasks 1 and 2 are independent and can be done in parallel. Task 3 depends on Task 1 (organize sets up the task list ID that planner verifies). Task 4 depends on Task 3 (implement dispatches tasks created by planner). Task 5 is a quick version bump after all skills are written. Task 6 validates everything end-to-end.
