# Brief: Do-Stuff-Helper

## Goal
Build a skill and agent system on Claude Code that helps define, develop, and execute diverse personal activities — from software projects to life improvement goals — with maximum autonomy and minimum dependency on the user for routine work.

## Why
Life goals and side projects stall because they depend entirely on the user's limited time and attention. An AI-assisted execution system can keep multiple activities moving forward in parallel, handle research and grunt work autonomously, and ensure the user's scarce hours are spent on high-leverage decisions rather than busywork. Success means previously-stalled goals actually making meaningful progress.

## Success Criteria
- 3-4 activities running simultaneously with autonomous workers making real, quality progress
- User spends their 1-2 weekday hours (and longer weekend sessions) on interesting decisions rather than routine execution
- Meaningful progress on previously-stalled life goals that weren't advancing before the system existed
- Workers rarely block on the user when they could unblock themselves
- The system gets more autonomous over time through the ask-and-learn permission model

## Measurement
- Number of activities actively progressing (target: 3-4 within 3 months)
- Ratio of user time spent on decisions vs. grunt work (qualitative self-assessment)
- User-identified life goals that have moved from stalled to progressing
- Frequency and duration of worker-blocked-on-user states (trending down over time)
- Volume of autonomy expansions (new skills created from "always OK" approvals)

## Description

The system has three subsystems, prioritized in build order:

### 1. Activity Execution (First Priority)
The core pipeline for taking an activity from idea to ongoing execution:
- **Discover** — expert-driven interview producing a detailed brief (goal, success criteria, scope, vision)
- **Roadmap/Plan** — creates an adaptive graph of waypoints (outcome-focused checkpoints), informed by domain best practices, psychology, and the user's personal profile. The roadmap includes large strategic waypoints, small tactical items (unblockers, knowledge gaps, low-hanging fruit), and is regularly replanned as learning occurs
- **Waypoint Design** — before a waypoint can be decomposed into tasks, it needs a description sufficient for high-quality decomposition. A waypoint design agent evaluates each waypoint and scales its design effort to the complexity: simple tactical items may need only a sentence, while complex or ambiguous waypoints may require detailed designs, research, or decision documents. Every waypoint exits this phase with enough clarity for confident decomposition
- **Decompose & Execute** — waypoints are recursively decomposed into parallelizable tasks. A manager agent orchestrates worker agents that each iteratively decompose and execute tasks. Each activity runs in its own Claude Code instance on the home server (Docker)
- **Autonomy model** — workers operate on a whitelist: Claude Code built-ins, installed skills, and installed MCP servers are all allowed by default. For anything else, the worker asks the user with options: "I'll do it manually," "automatically just this time," "always OK," "not OK." If the user approves automation, the system creates a skill for it. Workers should aggressively self-unblock through research, MCP servers, and skill discovery before asking the user
- **Backlog & Replanning** — as work progresses, new insights, bugs, and scope changes are captured as updates. An autonomous process periodically incorporates updates into the roadmap
- **Feedback loop** — the user can give quick feedback (thumbs down / minus) on any worker output. A skill-improver agent periodically reviews accumulated feedback and proposes updates to skills, prompts, and CLAUDE.md files. Changes require user approval before landing

### 2. Monitoring & Dashboard (Second Priority)
An aggregation surface and set of independent analysis agents:
- **Dashboard** — shows all activities with current status, recent changelog, blockers (tasks assigned to user), next actions, and whether a Claude instance is actively working. Provides entry points to SSH into active sessions
- **Inbox** — async message queue where workers send questions and the user replies at their convenience, decoupled from execution. Primary mechanism for worker-to-user communication in the execution layer
- **Forward Motion Analyst** — weekly agent that reviews progress, assesses alignment with goals, and prepares a week-ahead plan with 2-3 super-important priorities. Continuously checks whether those priorities are being accomplished and refreshes the plan as needed
- **Priority Alignment Agent** — periodically checks whether effort distribution across activities matches stated priorities and values. Surfaces observations on the dashboard
- **Blocker Auto-Resolution** — a component that autonomously attempts to resolve blockers before escalating to the user
- **Work Checker** — reviews worker output quality, either at the manager level or as a separate agent

### 3. Life Coaching & Advisory Agents (Third Priority)
Advisory agents that analyze across activities and engage in async dialog:
- **Profile Builder** — interviews the user to establish personality, habits, work styles, and life goals. Uses life inventories for goals; needs a developed strategy for personality/habits assessment
- **Advisory Agent Pattern** — extensible pattern for multiple coach types: life coach, business coach, time management/scaling agent, self-improvement agent. All share the same shape: maintain a user profile, observe across activities, initiate conversations with insights and questions. Advisory only — no authority to pause or reprioritize, but engages in dialog about what it finds
- **Chat Integration** — advisory agents communicate via a chat platform (likely Telegram, to be evaluated when this subsystem is built). Notifications from the monitoring layer (e.g., aging blockers, priority drift) may also route through this channel, since notifications are likely advisory in nature
- The coaching system specifically watches for the user's known tendencies: avoiding hard/important tasks in favor of easy progress, and letting air gaps (tasks only the user can do) persist too long

## Scope

### In Scope
- Skills and agents for the full activity execution pipeline (discover → roadmap → waypoint design → decompose → execute)
- Adaptive roadmap with waypoints, recursive decomposition, and autonomous replanning
- Waypoint design agent that scales effort to complexity and ensures decomposition-ready descriptions
- Permission tier system with ask-and-learn autonomy expansion (including installed MCP servers as always-allowed)
- Feedback capture and autonomous skill improvement
- Monitoring dashboard with status, blockers, changelog, and active instance indicators
- Async inbox for worker-to-user communication
- Forward motion analysis and weekly planning
- Advisory coaching agent pattern (extensible)
- User profile system for personality, habits, and life goals
- Infrastructure: Docker containers on home server, n8n for scheduling/automation, browser automation (needs verification)
- Voice input pipeline (recordings from physical device into project context)
- do-stuff-helper as first test case; wildlife cards business and personal branding as pilot activities

### Out of Scope
- Telegram/chat platform integration (deferred to life coaching subsystem build)
- Push notifications (deferred; bundled with chat platform integration)
- Calendar integration (future)
- Email integration for forward motion analysis (future — infrastructure exists but deferred)
- Auto-rater agent for learning what "good" looks like (deferred until concrete use case emerges)
- Custom chat interface (evaluate existing platforms first when coaching is built)
- Goal-to-activity mapping model (deferred for further design)
- Dashboard UI technology selection (deferred)

## Key Risks
- **Air gaps** — activities stall when blocked on non-delegatable user actions (phone calls, interviews, decisions). The user's known tendency to avoid these in favor of easier work compounds this risk. Mitigation: monitoring agents aggressively surface aging blockers; coaching agents challenge avoidance patterns
- **Scope and momentum** — the system is ambitious and the user has limited hours. Risk of building infrastructure without ever realizing the value. Mitigation: strict priority ordering (execution → monitoring → coaching); validate with real pilot activities early
- **Autonomous agent quality** — workers operating unsupervised could produce low-quality work that costs more to review than it saves. Mitigation: work checker agent, feedback loop with skill improvement, mandatory checkpoints for high-stakes decisions
- **Platform coupling** — tight coupling to Claude Code's evolving plugin/skill/agent architecture means breaking changes could require significant rework. Mitigation: acceptable risk per user decision; MCP servers provide extension points to other tooling
- **Bootstrap problem** — do-stuff-helper needs its own roadmap planner to plan itself, but the roadmap skill doesn't exist yet. Mitigation: bootstrap with a lighter-weight planning approach, then formalize once the roadmap skill is built
- **Infrastructure readiness** — home server browser automation capability is unverified. Mitigation: validate early before building skills that depend on it

## Open Questions
- What life inventories or frameworks should the profile builder use for life goals assessment?
- What strategy should we use for personality and habits profiling?
- ~~What's the right storage format for adaptive roadmaps with waypoints?~~ **Resolved:** JSON file for waypoint graph (status, deps, phases), auto-generated markdown for human readability, individual markdown files per waypoint design. See CLAUDE.md Waypoint Storage section.
- What does the worker → manager → user escalation protocol look like in detail?
- How should the dashboard UI be built? (Technology choice deferred)
- Should the forward motion analyst and coaching agents share a unified user profile, or maintain separate views?
- What's the right architecture for the manager/worker relationship? (Manager orchestrates workers who each decompose and execute, but exact boundaries need design)
- How much waypoint design detail is "sufficient" for different activity types? (Software vs. life improvement may have different thresholds)

## Background & Context
- The user has a home server running Claude Code in Docker, n8n for automation, email integration, and browser automation capability (unverified — needs setup and testing)
- Telegram is set up but unverified; deferred until coaching/advisory subsystem
- Each activity gets its own GitHub repo and Claude Code instance
- The do-stuff-helper plugin (v0.2.0) already has working skills: organize (directory/repo bootstrapping), discover (expert interview → brief), and research (multi-angle web search with gap analysis)
- The user is data-driven and prefers tracking and measurement over subjective assessment
- For non-technical activities, agents serve as researchers, planners, trackers, and advisors — the user does the execution, agents make it easier and provide accountability
- n8n can handle scheduled triggers (e.g., weekly forward motion analysis) and deterministic workflow steps
- Voice input is available via physical recorder with an upload pipeline, but not live streaming

## Notes for Roadmap Planning
- **Build order:** Activity execution pipeline first, monitoring/dashboard second, life coaching/advisory third
- **Bootstrap strategy:** Use a lightweight planning approach for do-stuff-helper itself before the formal roadmap skill exists
- **Pilot activities:** do-stuff-helper (self-referential first test), wildlife cards business idea, personal branding effort
- **Early validation:** Get the discover → plan → execute loop working end-to-end on one pilot before broadening
- **Infrastructure validation:** Home server browser automation needs early verification since execution agents may depend on it
- **The advisory agent pattern** (used by life coach and related agents) should be designed as a reusable pattern from the start since multiple agents will share it
- **Naming conventions:** Activities (not projects), waypoints (not epics), workers/manager (execution agents)
- **1-2 hours weekday, more on weekends** — the system should be designed to maximize async autonomous work and minimize synchronous user dependency
- **Telegram and notifications** — deferred and bundled with the coaching/advisory agent build; all notifications are likely advisory in nature
