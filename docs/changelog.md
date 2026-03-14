# Changelog

## 2026-03-14 — Completed w32: ClaudePluginBuild Integration
Added software-build detection to waypoint-design, delegation handling to waypoint-implement, ClaudePluginBuild installation to organize, and documented the delegation field in activity conventions.

## 2026-03-14 — Implementing w32: ClaudePluginBuild Integration
Adding software-build detection and ClaudePluginBuild delegation to the waypoint pipeline.

## 2026-03-14 — Planned w32: ClaudePluginBuild Integration (4 tasks)
Schema documentation, organize plugin install, waypoint-design detection/delegation, waypoint-implement delegated handling. 4 agent tasks, 2 parallel tracks after schema task.

## 2026-03-14 — Designed w32: ClaudePluginBuild Integration
Waypoint-design detects software-build waypoints and recommends delegating to ClaudePluginBuild's pipeline (PRD → design → plan → implement). PRD seeded from brief + waypoint context. Organize installs the plugin. Waypoint-implement recognizes delegated waypoints via `delegation` field in roadmap JSON.

## 2026-03-14 — Added w32: ClaudePluginBuild Integration
New waypoint to delegate software-build waypoints to ClaudePluginBuild's specialized pipeline (TDD, multi-lens review, CI scaffolding). Changes organize, waypoint-design, and waypoint-implement skills.

## 2026-03-06 — Completed w30: Worker Verification Step
Added verify-before-finishing section and verification checklist to worker prompt. Step 6 now flags NEEDS REVIEW items for user instead of auto-completing.

## 2026-03-06 — Completed w29: Human Task Inbox Write Guardrail
Added verify-after-write and immediate commit to implement skill's Step 2. Session-restart bug deferred to backlog.

## 2026-03-06 — Designed w29: Human Task Inbox Write Guardrail
Add verify-after-write check to implement skill's Step 2. Narrowed from original two-bug scope after confirming the inbox write works but isn't reliable. Session-restart bug deferred to backlog.

## 2026-03-06 — Completed w27: Add Replan Reminder via n8n
Dashboard backlog section (collector reads docs/backlog.md, UI shows collapsible item list per activity) and weekly n8n workflow that checks backlog counts and creates inbox reminders for activities with 6+ items.

## 2026-03-06 — Completed w31: Front-End Design Plugin in Organize
Added frontend-design plugin to organize skill's Step 2 install list. All new activities will now have design system guidance available.

## 2026-03-06 — Planned w31: Front-End Design Plugin in Organize (1 task)
Single agent task: add frontend-design plugin to organize skill's Step 2 install list.

## 2026-03-06 — Designed w31: Front-End Design Plugin in Organize
Add frontend-design plugin to organize skill's Step 2 install list. No conditional detection — install for all activities since it's cheap and avoids predicting which need UI work.

## 2026-03-05 — Replanned: backlog review, 7 new waypoints
Processed 13 backlog items. Added w25-w31 (brief-roadmap sync, rhythm capability, replan reminder, auto-dispatch, implement bugs, worker verification, front-end plugin). Moved w24 (Agent Teams) to backlog. Marked w13, w14, w16, w17, w22, w23 obsolete/done. Updated brief with rhythm, ongoing monitoring, and brief-sync scope.

## 2026-03-04 — Completed w6: Pilot Activity — End-to-End
Summit Mt Whitney validated the full pipeline: discover → roadmap → waypoint design → plan → implement. Multiple waypoints designed, planned, and progressing with workers.

## 2026-03-03 — Implementing w6: Pilot Activity — End-to-End
Summit Mt Whitney is the pilot activity. Running through the full discover → roadmap → waypoint design → decompose → execute pipeline to validate end-to-end.

## 2026-03-03 — Completed w12: Forward Motion Analyst
Forward-motion skill with plan and review modes, "This Week" dashboard panel with per-item feedback, plan API endpoints, refresh hooks on inbox resolve and waypoint completion, n8n Sunday review workflow, and initial plan seeded from real activity data. Plugin bumped to 0.10.0.

## 2026-03-03 — Implementing w12: Forward Motion Analyst
Weekly priority agent that reviews all activity state, scores candidates by goal alignment and leverage, and produces a week-ahead plan with 2-3 priorities.

## 2026-03-03 — Planned w12: Forward Motion Analyst (7 tasks)
Weekly agent that reviews progress across activities, produces a week-ahead plan with priorities, and refreshes as work completes. Includes skill, dashboard panel, API endpoint, n8n workflow, and lifecycle hooks.

## 2026-03-02 — Completed w15: User Profile Builder
Full four-phase profile interview completed: overview, all 12 life areas with ratings and bottlenecks, ideal future with importance ratings and yearly theme ("Year of Launch"), work habits and known tendencies. Final profile saved to ~/exoselfai/docs/user-profile.json.

## 2026-02-27 — Implementing w15: User Profile Builder
Created user-profile-builder skill with interview guide distilled from 8760 Hours methodology. Skill supports full interview and update modes across twelve life areas. Updated discover skill with profile check integration. Awaiting user to conduct initial profile interview.

## 2026-02-26 — Completed w11: Inbox
Unified inbox system replacing separate blockers.json. Workers write questions and human-task blockers to docs/inbox.json. Dashboard shows inbox section with reply/resolve/discuss actions. n8n resume-implement workflow polls for answered items and triggers webhook. Implement skill updated with waiting status, inbox tracking, and auto-resume flow. Research spike evaluated Claude remote control and Telegram bot for discussion channel.

## 2026-02-26 — Implementing w11: Inbox
Unified inbox for async worker questions and human-task blockers, with dashboard resolution UI and n8n auto-resume.

## 2026-02-26 — Planned w11: Inbox (11 tasks)
Unified inbox for worker questions and human-task blockers, dashboard resolution UI with reply/resolve/discuss paths, n8n auto-resume workflow, implement skill updates, and discussion channel research spike.

## 2026-02-25 — Completed w10: Dashboard — Core
Single-page activity dashboard with progress tracking, changelog timeline, blocker alerts, and next actions. Updated all lifecycle skills with changelog milestone writing, smart waypoint-implement mode, and blocker convention.

## 2026-02-25 — Completed w9: Backlog & Replanning Process
Created replan skill and backlog conventions for capturing out-of-scope work and periodically incorporating it into the roadmap.

## 2026-02-24 — Completed w18: Home Server Browser Automation Setup
Installed Xvfb + VNC + Chrome + Claude in Chrome extension on the N100 with systemd services that survive reboot. Claude Code can control Chrome via --chrome flag and users can VNC in to observe or intervene.

## 2026-02-24 — Completed w5: Autonomy Model
Workers now receive capability awareness and self-unblocking instructions, using available tools (including writing ad-hoc scripts) before escalating to the user.

## 2026-02-24 — Completed w4: Decompose & Execute Architecture
Built waypoint-planner (recursive task decomposition) and waypoint-implement (parallel worker orchestration in git worktrees) skills, completing the core execution engine.

## 2026-02-24 — Completed w19: Fix Discover Skill Invocation Bug
Identified and fixed a name collision between the discover command and discover skill caused by `disable-model-invocation: true` on the command file.

## 2026-02-23 — Completed w20: Plugin Update Workflow
Created the `/publish` slash command that automates version bump, commit, push, marketplace update, and plugin reinstall.

## 2026-02-23 — Completed w3: Waypoint Design Skill
Built a skill that evaluates waypoints, scales design effort to complexity, and produces descriptions sufficient for confident decomposition.

## 2026-02-23 — Completed w21: Suggest-and-Confirm Transition in Discover
Added a suggest-and-confirm prompt at the end of the discover skill to offer transitioning directly into roadmap planning.

## 2026-02-23 — Completed w1: Roadmap Skill
Implemented a roadmap skill that reads a brief, interviews the user about priorities and sequencing, and produces an adaptive waypoint-based roadmap as JSON.

## 2026-02-23 — Completed w2: Waypoint Storage Format
Defined and documented the waypoint storage format (JSON source of truth + individual markdown design docs) and converted the roadmap to use it.

## 2026-02-23 — Roadmap created: Do-Stuff-Helper
Bootstrap roadmap created with phased waypoints covering the full pipeline from core execution to life coaching.

## 2026-02-22 — Activity defined: Do-Stuff-Helper
Expert interview completed, brief saved covering goals, scope, success criteria, and key risks for the do-stuff-helper activity.

---

## W12: Forward Motion Analyst

### 2026-03-03 - [#38] [w12] Create forward-motion skill SKILL.md
- Created `skills/forward-motion/SKILL.md` with plan mode (8 steps) and review mode (5 steps)
- Includes full JSON schema for current-plan.json, stability principle, staleness tracking, tendency checks, feedback processing
- Files changed: skills/forward-motion/SKILL.md

### 2026-03-03 - [#40] [w12] Add plan feedback API endpoint to dashboard server
- Added GET /api/plan, GET /api/plan/review, POST /api/plan/feedback endpoints to dashboard server
- Feedback endpoint triggers plan refresh via webhook (fire-and-forget)
- Files changed: ~/exoselfai/scripts/dashboard/server.js

### 2026-03-03 - [#39] [w12] Create forward-motion artifact directory and seed initial plan
- Generated initial current-plan.json with real priorities from user profile and activity data
- Priority 1: Create wildlife-cards activity (blind spot, Year of Launch theme)
- Priority 2: Plan spring Oregon trip to visit parents (blind spot, time-sensitive)
- Priority 3: Run do-stuff-helper pilot activity w6 (waypoint, comfortable item)
- Files changed: ~/exoselfai/docs/forward-motion/current-plan.json

### 2026-03-03 - [#41] [w12] Build "This Week" dashboard panel
- Added full "This Week" plan panel to dashboard UI above inbox and activity cards
- Priority cards with rank, activity badge, rationale, status, feedback input
- Lingering/stuck indicators, age badges for blocked items, blind spots, deferred section, weekly review toggle
- Files changed: ~/exoselfai/scripts/dashboard/public/index.html

### 2026-03-03 - [#42] [w12] Add plan refresh hooks to waypoint-implement and inbox resolve
- Added triggerPlanRefresh() calls to inbox reply and resolve endpoints in dashboard server
- Added plan refresh trigger instruction to waypoint-implement Step 8 (curl command)
- Files changed: ~/exoselfai/scripts/dashboard/server.js, skills/waypoint-implement/SKILL.md

### 2026-03-03 - [#43] [w12] Create n8n workflow for Sunday weekly review
- Created forward-motion-weekly.json: Sunday 8 PM Pacific schedule trigger → webhook POST
- Workflow pushed to n8n and activated (ID: ogeXnkbo98bbZOTt)
- Files changed: ~/exoselfai/n8n-workflows/forward-motion-weekly.json

### 2026-03-03 - Waypoint Complete
- Built full forward-motion analyst: skill with plan + review modes, dashboard panel, API endpoints, refresh hooks, n8n workflow, and initial plan from real data

## W5: Autonomy Model

### 2026-02-24 - [#2] [w5] Update organize skill with capabilities discovery
- Added Step 5b to organize skill: discovers MCP servers, CLI tools, and project scripts
- Extended canonical CLAUDE.md template with `### Capabilities` subsection
- Updated Step 8 summary to report discovered capabilities
- Files changed: skills/organize/SKILL.md

### 2026-02-24 - [#3] [w5] Update waypoint-implement worker prompt
- Added `## Capabilities` section instructing workers to read CLAUDE.md and write/execute ad-hoc scripts
- Added self-unblocking instructions to `## Handling Ambiguity` section
- Resolved merge conflict to preserve both `## Available Skills` and new `## Capabilities` sections
- Files changed: skills/waypoint-implement/SKILL.md

### 2026-02-24 - [#4] [w5] Run organize on do-stuff-helper to seed capabilities
- Discovered CLI tools: gh, node, docker, python3, npm, npx, make
- No MCP servers or project scripts detected
- Populated `### Capabilities` subsection in CLAUDE.md
- Files changed: CLAUDE.md

### 2026-02-24 - Waypoint Complete
- Workers now receive capability awareness (skills, MCP servers, CLI tools, project scripts) via CLAUDE.md and are instructed to self-unblock using available tools — including writing and executing ad-hoc code — before escalating.

## W4: Decompose & Execute Architecture

### 2026-02-24 - Waypoint Complete
- Built `waypoint-planner` skill: decomposes designed waypoints into task DAGs with dependencies, metadata, and cross-session persistence
- Built `waypoint-implement` skill: orchestrates parallel worker agents in git worktrees, handles merge, changelog, and waypoint lifecycle
- Validated end-to-end on W20 (plugin update workflow) and W19 (discover bug fix), including merge conflict resolution

## W19: Fix Discover Skill Invocation Bug

### 2026-02-24 - [#1] [w19] Remove discover command and update references
- Deleted `commands/discover.md` which had `disable-model-invocation: true` causing name collision with the discover skill
- Updated `/discover` command references in `CLAUDE.md` and `skills/organize/SKILL.md` to remove the dead reference
- Resolved merge conflict in CLAUDE.md (worktree had older version without the do-stuff-helper section)
- Files changed: commands/discover.md (deleted), CLAUDE.md, skills/organize/SKILL.md

### 2026-02-24 - Waypoint Complete
- Fixed discover skill invocation by removing the redundant command that shared its name. The `disable-model-invocation: true` flag on the command was being picked up when the Skill tool resolved `do-stuff-helper:discover`.

## W20: Plugin Update Workflow

### 2026-02-23 - [#1] [w20] Write commands/publish.md
- Created `commands/publish.md` slash command with version bump (patch/minor/major), commit, push, marketplace update, and plugin reinstall
- Assumptions: Used `git add` with specific file paths instead of `git add -A` for safety
- Files changed: commands/publish.md

### 2026-02-23 - [#2] [w20] Test publish command
- Reviewed command logic for correctness: version parsing, bump semantics, commit message format, CLI commands
- Fixed `git add -A` to stage specific files (`plugin.json`, `marketplace.json`) instead
- Files changed: commands/publish.md (minor fix)

### 2026-02-23 - Waypoint Complete
- Created the `/publish` slash command that automates the full plugin update workflow: version bump, commit, push, marketplace update, and plugin reinstall. User only needs to start a new conversation afterward.
