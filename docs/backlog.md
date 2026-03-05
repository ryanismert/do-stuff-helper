# Backlog

Ideas, scope changes, and discovered work that are **out of scope** for the current waypoint. Items here will be periodically reviewed by the `replan` skill and categorized as design updates (changes to the brief/roadmap) or tactical waypoints (new work items).

**To add an item:** Append a new section with the date and source.

---

## 2026-02-25 — user

**Scheduled backlog review via n8n:** Once user notifications are implemented (w17 or similar), add an n8n workflow that triggers the replan skill weekly to review the backlog. This keeps the roadmap fresh without requiring the user to remember to run it.

## 2026-02-25 — roadmap cleanup

**Feedback Capture:** A way for users to give quick feedback (thumbs-down, typed comments) on worker output, recorded with context (input, output, rating) in the activity repo. Revisit if a lightweight feedback mechanism becomes available in Claude Code's UI.

**Skill-Improver Agent:** An agent that periodically reviews worker output quality or user-flagged issues, proposes updates to skills/prompts/CLAUDE.md, and presents changes for user approval before landing. Could be triggered by patterns in the backlog, poor worker output, or direct user requests.

## 2026-02-25 — user

**Keep brief in sync with roadmap changes:** Any skill that adds waypoints or modifies the roadmap (roadmap, replan, or manual edits) should also update the activity brief to reflect the change. The brief should stay current as the source of "what this activity is about" — if we add new scope, the brief should mention it.

## 2026-02-25 — w10 design

## 2026-02-25 — user

**Dashboard mobile and at-a-glance improvements:** The dashboard looks nice on mobile but needs to be more compact and show more information at a glance. Consider condensing the layout — tighter spacing, smaller cards, collapsible sections, or a summary bar at the top with key stats. Same improvements would benefit the desktop version too.

## 2026-02-25 — w10 implementation

**Verification step in waypoint-implement:** The implement skill has no structured testing or verification step. Workers may or may not test their output — it's up to their initiative. Add a verification step to the worker prompt and/or the post-merge flow (Step 6) so that implemented work is validated before being marked complete. Could range from "run the tests if they exist" to "verify the output matches the acceptance criteria in the task description."

## 2026-02-27 — w15 implementation

**Implement skill not writing human tasks to inbox.json:** During w15 implementation, the implement skill's Step 2 should have written the human task (#37, conduct profile interview) to `docs/inbox.json` as a blocker so it appeared on the dashboard. It didn't — possibly because the skill hadn't been reloaded since w11 changes, or because the implement skill code doesn't actually perform the inbox write yet. Verify this works correctly after a fresh session with the updated plugin. If it's a skill bug, fix the implement skill's Step 2 to write human tasks to inbox.json when surfacing them.

**Implement skill should insert session-restart tasks in the dependency graph:** When agent tasks produce new skills or other artifacts that require a session restart before subsequent tasks can use them, the implement skill should recognize this and insert a human task (e.g., "Restart session to reload updated plugin") into the dependency graph between the producing task and the consuming task. During w15, the interview task (#37) depended on the SKILL.md being loaded, but the user would have needed to restart the session first — this wasn't captured as a task or surfaced as a blocker.

## 2026-03-02 — user

**Comprehensive business skill set:** The do-stuff-helper plugin should provide a full set of skills covering the major domains needed to run a business or side project end-to-end. This includes marketing (content strategy, SEO, social media, copywriting), software development (architecture, code review, testing strategy, deployment), business operations (financial planning, pricing, competitive analysis), and product management (user research, feature prioritization, launch planning). Audit the current skill set for gaps, research what Anthropic and community plugins already cover, and either build new skills or integrate existing ones so that activities spanning these domains have expert-level guidance available.

## 2026-03-01 — user

**Install front-end design plugin in organize skill:** The organize skill should install a front-end/UI design plugin (e.g., `design-tokens` or similar) when bootstrapping activities that involve building user interfaces. This ensures agents working on front-end waypoints have access to design system guidance, component patterns, and UI best practices from the start. Evaluate which plugin is best suited and add it to the organize skill's Step 2 plugin installation list.

## 2026-02-26 — user

**Ongoing monitoring activities (non-project):** Not everything the user cares about is a project with a clear endpoint. Things like personal health, spending more time with kids, or maintaining habits are ongoing concerns that benefit from monitoring, gentle nudges, and periodic reflection — but don't fit the discover → roadmap → implement pipeline. Consider a new activity type or waypoint pattern for ongoing monitoring: periodic check-ins, trend tracking, goal reminders, and advisory-style engagement rather than task execution.

## 2026-02-28 — user (summit-mt-whitney discover)

## 2026-03-05 — user

**Auto-dispatch ready tasks without human kick-off:** When agent tasks become unblocked (dependencies completed, no pending questions), the system should automatically start executing them rather than waiting for a human to invoke waypoint-implement. This could be an n8n workflow that polls for ready tasks and triggers execution via the webhook, or a post-merge hook in the implement skill that checks for newly unblocked work. The goal is continuous autonomous progress — the user should only need to intervene for human tasks and decisions, not to say "go" on work that's already planned and ready.

## 2026-02-28 — user (summit-mt-whitney discover)

**Rhythm: recurring progressive schedules as activity artifacts.** Activities often produce not just one-time deliverables but ongoing, time-bound routines that the user follows for weeks or months. Example: a Mount Whitney training plan is a phased weekly schedule with progressive difficulty, milestone checkpoints, and calendar integration. This doesn't fit the task/waypoint model (tasks are one-time; waypoints are deliverable chunks). A new concept — tentatively called a **rhythm** — would capture recurring behavioral patterns that an activity produces. Key properties: (1) recurring on a schedule (daily/weekly), (2) progressive (evolves in phases over time), (3) checkpointed (milestones to verify progress), (4) living (not "done" when created — done when the activity is done), (5) calendarable (maps to actual time slots). Generalizes beyond fitness: a writing rhythm, a savings rhythm, a language-learning practice rhythm. Consider how rhythms integrate with the roadmap (a waypoint might produce a rhythm as its deliverable), the dashboard (show current rhythm status and adherence), and calendar tools (generate recurring events). Related to the "ongoing monitoring activities" backlog item — rhythms may be one concrete implementation pattern for non-project ongoing concerns.
