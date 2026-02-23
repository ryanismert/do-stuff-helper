# Roadmap: Do-Stuff-Helper

## How to Read This Roadmap

This is an adaptive roadmap. Waypoints are outcome-focused checkpoints, not rigid tasks. They will be refined, reordered, and replanned as we learn. Each waypoint has a definition of done and its key dependencies.

**Status key:** `pending` | `in-progress` | `done` | `blocked`

---

## Phase 1: Complete the Core Pipeline

Goal: Get the full discover → roadmap → waypoint design → decompose → execute loop working end-to-end.

### W1: Roadmap Skill
**Status:** pending
**Why:** Every future activity needs this. We're bootstrapping manually right now, which doesn't scale.
**Done when:** A `roadmap` skill exists that reads a brief, interviews the user about priorities and sequencing, and produces an adaptive waypoint-based roadmap saved to `docs/roadmap-<slug>.md`. Includes suggest-and-confirm transition from discover.
**Dependencies:** None (brief and discover skill already exist)

### W2: Waypoint Storage Format
**Status:** in-progress
**Why:** We need to decide how waypoints are stored before other skills can read/write them. Affects every downstream skill.
**Done when:** A storage format is defined, documented in CLAUDE.md, and the roadmap skill uses it.
**Dependencies:** Informed by W1 design
**Decisions made:**
- JSON file (`roadmap-<slug>.json`) as source of truth for waypoint graph (status, dependencies, phase membership)
- Human-readable markdown (`roadmap-<slug>.md`) auto-generated from JSON
- Individual waypoint designs in `docs/waypoints/<waypoint-id>.md`
- Waypoint design docs require **Objective** and **Done When** sections; everything else is freeform scaled to complexity
- Phases tracked in JSON with completion status
**Remaining:** Define the JSON schema and implement it in W1

### W3: Waypoint Design Skill
**Status:** pending
**Why:** Waypoints need sufficient descriptions before they can be decomposed. This is the bridge between high-level planning and task execution.
**Done when:** A skill exists that evaluates a waypoint, scales its design effort to complexity, and produces a description sufficient for confident decomposition. Simple waypoints get a sentence; complex ones get detailed designs or decision docs.
**Dependencies:** W2 (needs to read/write waypoint format)

### W4: Decompose & Execute Architecture
**Status:** pending
**Why:** This is the core engine — recursive task decomposition and parallel worker execution.
**Done when:** A manager agent can take a designed waypoint, decompose it into parallelizable tasks, and orchestrate worker agents to execute them. Workers operate in their own context and report status back.
**Dependencies:** W3 (needs designed waypoints as input)
**Note:** This is the most architecturally complex waypoint. Will likely need significant design work in W3-style detail before building.

### W5: Autonomy Model
**Status:** pending
**Why:** Workers need to know what they're allowed to do without asking.
**Done when:** Permission tier system is implemented — built-ins and installed skills/MCP servers allowed by default, ask-and-learn for everything else with "manual / once / always / never" options. New "always" approvals create skills automatically.
**Dependencies:** W4 (needs workers to exist)

### W6: Pilot Activity — End-to-End
**Status:** pending
**Why:** Validates the entire pipeline on a real activity before broadening.
**Done when:** One pilot activity (wildlife cards or personal branding) has been run through discover → roadmap → waypoint design → decompose → execute, with at least one waypoint fully completed by workers.
**Dependencies:** W1–W5

---

## Phase 2: Feedback & Replanning

Goal: Close the learning loops so the system improves itself and adapts to change.

### W7: Feedback Capture
**Status:** pending
**Why:** Without feedback, agent quality can't improve.
**Done when:** User can give thumbs-down or typed feedback on any worker output. Feedback is recorded with context (input, output, rating) in the activity repo.
**Dependencies:** W4 (needs worker output to give feedback on)

### W8: Skill-Improver Agent
**Status:** pending
**Why:** Turns feedback into actual improvements without requiring manual skill editing.
**Done when:** An autonomous agent periodically reviews accumulated feedback, proposes updates to skills/prompts/CLAUDE.md, and presents changes for user approval before landing.
**Dependencies:** W7

### W9: Backlog & Replanning Process
**Status:** pending
**Why:** The roadmap needs to adapt as we learn. New insights, scope changes, and discovered work need to be incorporated.
**Done when:** An autonomous process captures updates (new waypoints, scope changes, reprioritizations) and periodically incorporates them into the roadmap. User is notified of significant changes.
**Dependencies:** W1, W2 (needs roadmap format to modify)

---

## Phase 3: Monitoring & Dashboard

Goal: Unified visibility across all activities with independent analysis agents.

### W10: Dashboard — Core
**Status:** pending
**Why:** As multiple activities run in parallel, the user needs a single place to see everything.
**Done when:** A dashboard shows all activities with: current status, recent changelog, blockers assigned to user, next actions, and whether a Claude instance is actively working.
**Dependencies:** W4 (needs running workers to report on)

### W11: Inbox
**Status:** pending
**Why:** Workers need a way to ask questions async without blocking.
**Done when:** Workers can send questions to a message queue. User can view and reply at their convenience. Worker picks up the response and continues.
**Dependencies:** W4, W10 (can be built alongside dashboard)

### W12: Forward Motion Analyst
**Status:** pending
**Why:** Keeps the user focused on what matters most each week.
**Done when:** A weekly agent reviews progress across activities, assesses alignment with goals, produces a week-ahead plan with 2-3 priorities, and refreshes as priorities are completed.
**Dependencies:** W10 (needs activity status data)

### W13: Work Checker
**Status:** pending
**Why:** Quality gate for autonomous worker output.
**Done when:** An agent reviews worker output for quality, either at the manager level or as a standalone reviewer. Flags low-quality work before it's committed.
**Dependencies:** W4

### W14: Priority Alignment & Blocker Auto-Resolution
**Status:** pending
**Why:** Surfaces misalignment between effort and stated priorities; auto-resolves blockers where possible.
**Done when:** Agents periodically check effort distribution, surface observations on dashboard, and attempt to resolve blockers autonomously before escalating.
**Dependencies:** W10, W12

---

## Phase 4: Life Coaching & Advisory

Goal: Advisory agents that help the user stay aligned with their values and life goals.

### W15: User Profile Builder
**Status:** pending
**Why:** All advisory agents need a profile of the user's personality, habits, and goals.
**Done when:** An interview-based skill builds a user profile covering personality, work styles, habits, and life goals. Profile is stored and accessible to other agents.
**Dependencies:** None (can start anytime, but lower priority)

### W16: Advisory Agent Pattern
**Status:** pending
**Why:** Multiple coach types share the same shape — design it once.
**Done when:** A reusable pattern exists for advisory agents: maintain user profile access, observe across activities, initiate conversations, advisory-only. At least one concrete agent (life coach) built on the pattern.
**Dependencies:** W15, W10 (needs cross-activity visibility)

### W17: Chat Integration & Notifications
**Status:** pending
**Why:** Advisory agents need a channel to engage in dialog with the user.
**Done when:** Advisory agents can send messages and receive replies via a chat platform (likely Telegram). Monitoring notifications also route through this channel.
**Dependencies:** W16

---

## Tactical Items

These are smaller items that unblock work, fill knowledge gaps, or are low-hanging fruit.

### T1: Verify Home Server Browser Automation
**Status:** pending
**Why:** Execution agents may depend on browser capability. Needs verification before we build skills that assume it.
**Done when:** Browser automation is confirmed working on the home server, or we know what setup is needed.

### T2: Fix Discover Skill Invocation Bug
**Status:** pending
**Why:** The `do-stuff-helper:discover` skill returned a `disable-model-invocation` error when invoked via the Skill tool, despite not having that flag set.
**Done when:** Root cause identified and fixed, or workaround documented.

### T3: Plugin Update Workflow
**Status:** pending
**Why:** Updating the plugin currently requires manually pulling the marketplace clone. This friction slows development.
**Done when:** A documented (or automated) workflow exists for: bump version → commit → push → pull marketplace clone → reinstall plugin.

### T4: Suggest-and-Confirm Transition in Discover
**Status:** pending
**Why:** Discover should prompt the user to continue to roadmap planning when it finishes, per Option C decision.
**Done when:** Discover skill's Step 11 includes a suggest-and-confirm prompt to invoke the roadmap skill.
