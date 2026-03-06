---
name: waypoint-implement
description: This skill should be used when the user asks to "implement a waypoint", "execute tasks", "run the workers", "continue working on <waypoint>", or when transitioning from waypoint-planner after tasks are created. Orchestrates parallel worker agents to execute tasks from the task list. Do NOT trigger for planning or decomposition — use waypoint-planner for that.
---

# Waypoint Implement

Orchestrate execution of tasks from the native task list. Dispatch parallel worker agents in git worktrees, handle failures and retries, surface human tasks as blockers, route worker questions to the user, merge completed work, and maintain the changelog.

## Checklist

Complete each step in strict order. Do not skip steps.

### Step 1: Read the Task List

- Call `TaskList` to see current state
- If the user specified a waypoint, use `TaskGet` on each task to filter by `metadata.waypoint`
- Show a summary to the user:
  > "Task status: X total, Y done, Z pending, W implementing, H human tasks"
- If no tasks exist for the waypoint, tell the user to run waypoint-planner first and stop

### Step 1b: Set Status and Log Start

- Read `docs/roadmap-<slug>.json` and check the waypoint status
- If status is already `started`: skip this step (a previous run already set the status)
- Otherwise, set the waypoint status to `started` and update the `updated` field to today's date (YYYY-MM-DD)
- Prepend a milestone entry to `docs/changelog.md` (right after the `# Changelog` header line, above all existing entries):
  ```
  ## YYYY-MM-DD — Implementing <wN>: <Title>
  <1 sentence about what's being worked on>
  ```
- If `docs/changelog.md` doesn't exist, create it with a `# Changelog` header first

### Step 1c: Assess Complexity

Evaluate whether the waypoint work warrants full parallel worker dispatch or inline execution:

- **Use workers (complex mode)** when:
  - More than 3 agent tasks AND at least some can run in parallel
  - Tasks have significant scope or touch different parts of the codebase
  - Multiple worktrees would avoid merge contention

- **Execute inline (simple mode)** when:
  - 3 or fewer agent tasks, OR all tasks are sequential (linear dependency chain)
  - Tasks are straightforward changes (config edits, doc updates, small code changes)
  - Spawning workers would add overhead without benefit

**Simple mode**: Execute tasks directly in the current session — no worktrees, no subagents. Process tasks in dependency order, updating task status (`in_progress` → `completed`) and writing changelog entries as you go. Follow the same lifecycle as workers: commit changes, handle blockers, write to changelog. Skip Steps 5-6 and go straight from Step 4 to Step 7 (loop), doing the work inline.

**Complex mode**: Proceed with the full worker pipeline (Steps 5-6) as designed below.

Either path must handle the full lifecycle: status updates, changelog entries, blocker tracking, and waypoint completion.

### Step 2: Surface Human Tasks

- Find tasks with `metadata.assignee == "human"` that are not blocked (no unresolved `blockedBy`)
- Present them clearly:
  > "These tasks need you: [list with descriptions]. [N] other tasks are waiting on them."
- Do not attempt to execute human tasks
- **Write to inbox**: For each unresolved human task, append a blocker entry to `docs/inbox.json` (see Inbox Tracking below). This ensures human-required work is visible on the dashboard even outside the task list.

  ```json
  {
    "id": "q-N",
    "type": "blocker",
    "task_id": "<task-id>",
    "waypoint": "<waypoint-id>",
    "subject": "[wN] <task subject>",
    "body": "<description of what user needs to do>",
    "context": "<what the worker was doing>",
    "created": "<ISO timestamp>",
    "status": "pending",
    "resolution": null,
    "resolved_at": null
  }
  ```

- **Verify the write**: After writing, read `docs/inbox.json` back. For each unblocked human task, confirm an entry exists with a matching `task_id` and `status: "pending"`. If any are missing, write them now.
- **Commit immediately**: If any entries were written (or the file was created), commit `docs/inbox.json` right away so the dashboard picks it up. Do not defer this to a later step.

### Step 3: Check for Answered Items

Process answered items from two sources: in-session task metadata and the persistent inbox.

**In-session questions (metadata.question):**
- Find tasks with `metadata.question` set (from a previous worker run that flagged a question)
- Present each question to the user with context: task subject, what the worker was trying to do, the question text
- After user answers, update the task description with an "Additional Context" section containing the answer
- Clear the `metadata.question` field via `TaskUpdate`
- The task is now ready to be re-dispatched

**Inbox items (docs/inbox.json):**
- Read `docs/inbox.json` and find entries with `status: "answered"` for this waypoint
- For each answered entry:
  - If `type: "question"`: append the `resolution` text to the corresponding task's description as "Additional Context"
  - If `type: "blocker"`: the task's human prerequisite is satisfied; the task is now unblocked
  - Update the inbox entry's `status` to `"resolved"`
  - Clear `metadata.question` on the task (if applicable) via `TaskUpdate`
  - The task is now ready for re-dispatch

### Step 4: Identify Ready Agent Tasks

Find pending tasks where all of the following are true:
- `metadata.assignee == "agent"`
- All `blockedBy` tasks are completed
- No `metadata.question` set

If no ready tasks exist but incomplete tasks remain, report which tasks are blocked and by what.

### Step 5: Dispatch Workers

For each ready task, spawn via the `Task` tool:
- `subagent_type: "general-purpose"`
- `isolation: "worktree"`
- Construct the worker prompt (see Worker Prompt Construction below)

Dispatch independent tasks in parallel (multiple `Task` tool calls in one message). Do NOT dispatch tasks that share dependencies concurrently — the DAG prevents this via `blockedBy`.

**Worker Prompt Construction:**

```
You are a worker agent executing a task for the [activity name] project.

## Your Task

[Full task description from TaskGet]

## Context Files

[Content of each file listed in metadata.context_files, or instruction to read them]

## Conventions

[Content of the activity's CLAUDE.md, or instruction to read it]

## How to Work

1. Read any context files you need
2. Implement what the task describes
3. Commit your changes with a clear commit message that includes the task subject
4. If you create non-code output (research, docs), write it to: [metadata.output_path if set]

## Available Skills

For coding tasks, consider using these superpowers skills:
- **superpowers:test-driven-development** — Write failing tests first, then implement
- **superpowers:systematic-debugging** — When you encounter bugs or unexpected behavior
- **superpowers:dispatching-parallel-agents** — When you have multiple independent sub-problems

These are optional — use them when the task involves writing or debugging code. Skip for research, docs, or configuration tasks.

## Capabilities

Read CLAUDE.md for available skills, MCP servers, and project tools.

You can also write and execute code on the fly. If a task would benefit
from a script — data processing, API calls, file transformations, testing,
automation — write it and run it via Bash. Use whatever language fits the
problem (Python, Node, shell, etc.). Clean up temporary scripts when done.

## Handling Ambiguity

Before flagging a QUESTION, exhaust your options:
- Check if an available MCP server, skill, or CLI tool solves the problem
- Use WebSearch/WebFetch for research
- Write a script to automate or solve the problem
- Only escalate when truly blocked

Bias toward action. If you encounter ambiguity:
- Make a reasonable decision and document your assumption in your commit message
- Only flag a question if you truly cannot proceed — two valid approaches with significantly different consequences, or missing information you can't infer
- To flag a question: clearly state what you need and why at the END of your response, prefixed with "QUESTION:". This will be written to the project inbox so the user can answer asynchronously from any device.

## Discovered Work

- **In-scope:** If you discover additional work needed for this waypoint, just do it and note it in your summary.
- **Out-of-scope:** If you discover work that belongs to a different waypoint or is entirely new, append it to `docs/backlog.md` using the format: `## <date> — <source>\n\n<description>`

## Verify Before Finishing

Before reporting done, review the task description's acceptance criteria:
1. List each acceptance criterion or requirement from the task description
2. For each one, verify it is satisfied — run the code, check the file exists, confirm the behavior
3. If any criterion is NOT met, fix it before finishing
4. If a criterion cannot be verified (e.g., requires user judgment), note it as "cannot verify — NEEDS REVIEW"

## When Done

Report:
- What you did (summary)
- Files changed
- Verification: [for each acceptance criterion: criterion — PASS or NEEDS REVIEW]
- Any assumptions you made
- Any questions (if truly blocked — see above)
```

### Step 6: Collect Results and Merge

As each worker returns, handle the outcome:

**Success (worker completed, worktree has changes):**
- Check the worker's summary for verification results. If any criterion is marked "NEEDS REVIEW", do NOT auto-complete — present the items to the user and ask whether to accept or re-dispatch the task.
- If all criteria pass (or no NEEDS REVIEW items): `TaskUpdate` status to `completed`
- Merge the worktree branch: run `git merge <branch> --no-edit` on main
- If merge conflict: attempt auto-resolution. If that fails, report conflicting files to the user and pause
- If `metadata.link_from` is set, update those docs with a link to `metadata.output_path`
- Append entry to `docs/changelog.md` (see Changelog Management below)

**Success (no file changes):**
- `TaskUpdate` status to `completed`
- Append entry to changelog noting what was done (e.g., research, validation)

**Question (worker flagged QUESTION: in response):**
- Write the question to `docs/inbox.json` as `type: "question"` (append to array, auto-increment ID based on highest existing `q-N` ID):
  ```json
  {
    "id": "q-N",
    "type": "question",
    "task_id": "<task-id>",
    "waypoint": "<waypoint-id>",
    "subject": "[wN] <task subject>",
    "body": "<the worker's question text>",
    "context": "<what the worker was doing when it hit the question>",
    "created": "<ISO timestamp>",
    "status": "pending",
    "resolution": null,
    "resolved_at": null
  }
  ```
- Also set `metadata.question` on the task for in-session visibility
- Set status back to `pending`
- Commit the `docs/inbox.json` change so it is visible to the dashboard immediately
- Will be surfaced in Step 3 on next loop iteration (or answered asynchronously via the dashboard)

**Failure (worker errored or couldn't complete):**
- Report failure details to the user
- Ask: retry the task, re-decompose into subtasks, or mark as human-blocked?
- Dependent tasks remain blocked until this is resolved

### Step 7: Loop

Repeat steps 2-6 until one of these conditions is met:

- **All tasks for this waypoint are `completed`** — proceed to Step 8
- **All remaining tasks are blocked on inbox items** (questions pending or blockers pending with no answerable tasks left) — set waypoint status to `waiting` in `docs/roadmap-<slug>.json`, update the `updated` date, and report to the user:
  > "All dispatchable tasks are done. These items are waiting on you:
  > [list of pending inbox items with subjects]
  >
  > Answer them on the dashboard: http://exoself-dev1:3002
  > The system will automatically resume when you respond."
- **Only human tasks remain (not tracked in inbox)** — surface them and pause:
  > "All agent tasks are done. These human tasks remain: [list]. Resume when you've completed them."
- **A failure blocks all remaining progress** — escalate:
  > "Execution is blocked. [details]. What would you like to do?"

### Step 8: Complete Waypoint

When all tasks (including human tasks) are done:

1. Update `docs/roadmap-<slug>.json`: set waypoint status to `done`, update the `updated` date (YYYY-MM-DD)
2. Prepend a completion milestone to `docs/changelog.md` (right after the `# Changelog` header line):
   ```
   ## YYYY-MM-DD — Completed <wN>: <Title>
   <1-2 sentence summary of what was accomplished>
   ```
3. Resolve any open inbox items for this waypoint in `docs/inbox.json` — set `status` to `"resolved"` and `resolved_at` to the current ISO timestamp
4. Report to the user:
   > "Waypoint [title] is complete. [summary of what was accomplished]"
5. Trigger a plan refresh so forward-motion priorities reflect the completed waypoint. Use a fire-and-forget Bash command:
   ```bash
   curl -s -X POST http://localhost:3001/webhook/execute \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer $CLAUDE_WEBHOOK_TOKEN" \
     -d '{"task":"Run the forward-motion skill in plan mode to refresh priorities. Invoke do-stuff-helper:forward-motion"}' &
   ```
6. Check for other waypoints now unblocked by this completion and suggest next steps:
   > "These waypoints are now unblocked: [list]. Want to design or plan any of them?"

### Manual Waypoint Completion

Sometimes a waypoint is completed outside this skill's lifecycle — for example, a human task is finished in a discuss session or the user marks it done in conversation. Whoever completes the waypoint must perform the same actions as Step 8:

1. Update `docs/roadmap-<slug>.json`: set waypoint status to `done`, update the `updated` date
2. Prepend a "Completed" milestone to `docs/changelog.md`
3. Resolve any open inbox items for this waypoint in `docs/inbox.json`
4. Commit the changes

If you are marking a waypoint as done and the changelog doesn't already have a "Completed" entry for it, add one.

## Changelog Management

The skill manages `docs/changelog.md` at two levels:

### Waypoint-Level Milestones (## headings)

Milestone entries are **prepended** after the `# Changelog` header (newest first). They mark lifecycle transitions:

- **At implementation start** (Step 1b):
  ```markdown
  ## YYYY-MM-DD — Implementing <wN>: <Title>
  <1 sentence about what's being worked on>
  ```

- **At waypoint completion** (Step 8):
  ```markdown
  ## YYYY-MM-DD — Completed <wN>: <Title>
  <1-2 sentence summary of what was accomplished>
  ```

### Task-Level Detail (### headings)

Task entries are appended under a waypoint section heading (`## <Waypoint ID>: <Waypoint Title>`) as work progresses:

- If the file doesn't exist, create it with a `# Changelog` header
- If the waypoint section heading doesn't exist, add it below the milestone entries
- Each task completion appends an entry under the waypoint section heading:

```markdown
### YYYY-MM-DD - [<task-id>] <task subject>
- <summary of what was done>
- Assumptions: <any assumptions the worker documented>
- Files changed: <list>
```

- When a waypoint completes (Step 8), append a summary entry under the waypoint section:

```markdown
### YYYY-MM-DD - Waypoint Complete
- <overall summary of what the waypoint achieved>
```

## Inbox Tracking

The skill manages `docs/inbox.json` as a unified store for all items waiting on the user — both questions from workers and human-task blockers. This replaces the previous `docs/blockers.json` format.

- If the file doesn't exist, create it with an empty array `[]`
- All entries follow the same schema:

```json
[
  {
    "id": "q-N",
    "type": "question | blocker",
    "task_id": "<task-id>",
    "waypoint": "<waypoint-id>",
    "subject": "[wN] <task subject>",
    "body": "<question text or description of what user needs to do>",
    "context": "<what the worker was doing>",
    "created": "<ISO timestamp>",
    "status": "pending | answered | resolved",
    "resolution": null,
    "resolved_at": null
  }
]
```

**Fields:**
- `type` — `"question"` (needs a text answer) or `"blocker"` (needs human action + confirmation)
- `body` — the question text or description of what the user needs to do
- `context` — what the worker was doing when it hit the question or blocker
- `resolution` — the user's reply text (for questions) or a note about what they did (for blockers, optional)
- `resolved_at` — ISO timestamp when the user responded

**Statuses:**
- `pending` — waiting on user (written by implement skill)
- `answered` — user responded via dashboard or discussion (written by dashboard)
- `resolved` — implement skill picked up the response and re-dispatched the task (written by implement skill)

**Adding entries:** Append to the array. Auto-increment the ID based on the highest existing `q-N` ID in the file.

**Processing answers (Step 3):** When an entry has `status: "answered"`, the implement skill reads the `resolution`, applies it to the task, and sets the entry to `status: "resolved"`.

**Waypoint completion (Step 8):** Resolve all open inbox items for the completed waypoint.

## Edge Cases

- **No tasks exist for the waypoint:** Tell the user to run waypoint-planner first and stop.
- **All tasks already completed:** Tell the user the waypoint is done. Offer to mark it done in the roadmap if not already.
- **Session ends mid-execution:** Tasks persist via `CLAUDE_CODE_TASK_LIST_ID`. The user can re-invoke to resume. The skill picks up from current task states — Step 1 reads whatever state tasks are in. The roadmap status will remain `started` until Step 7 sets it to `waiting` or Step 8 sets it to `done`.
- **Waypoint already in `started` status (resuming a previous run):** Step 1b detects this and skips. Proceed directly to Step 1c.
- **Resuming a waypoint in `waiting` status:** Set status to `started` in the roadmap JSON, then proceed to Step 1c. Step 3 will pick up any answered inbox items and unblock the corresponding tasks. Continue the loop normally from there.
- **Worker creates merge conflict:** Attempt auto-merge. If that fails, escalate with conflicting file details so the user can resolve manually.
- **Task has no context_files:** Worker gets just the task description and the activity's CLAUDE.md.

## Cross-Skill Invocation

- **Invoked by `do-stuff-helper:waypoint-planner`** — via suggest-and-confirm after task creation (Step 7 of waypoint-planner).
- Uses `superpowers:dispatching-parallel-agents` pattern for parallel dispatch via the `Task` tool with `isolation: "worktree"`.
- Future: W24 (Agent Teams) will enhance the dispatch mechanism with persistent teammate sessions.
