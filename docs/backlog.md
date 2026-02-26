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

**Dashboard SSH entry points:** The dashboard should eventually provide entry points to SSH into active Claude sessions (e.g., tmux session names, connect commands). Deferred from w10 MVP to keep scope focused on read-only status visibility.

**Dashboard Git repository status:** Show per-activity Git status on the dashboard — open pull requests, pending reviews, unpushed commits, branch state. May also need to invest in proper CI pipelines (GitHub Actions) for activity repos so the dashboard can surface build/test status alongside PR state.

## 2026-02-25 — user

**Dashboard mobile and at-a-glance improvements:** The dashboard looks nice on mobile but needs to be more compact and show more information at a glance. Consider condensing the layout — tighter spacing, smaller cards, collapsible sections, or a summary bar at the top with key stats. Same improvements would benefit the desktop version too.

## 2026-02-25 — w10 implementation

**Verification step in waypoint-implement:** The implement skill has no structured testing or verification step. Workers may or may not test their output — it's up to their initiative. Add a verification step to the worker prompt and/or the post-merge flow (Step 6) so that implemented work is validated before being marked complete. Could range from "run the tests if they exist" to "verify the output matches the acceptance criteria in the task description."

## 2026-02-26 — user

**Ongoing monitoring activities (non-project):** Not everything the user cares about is a project with a clear endpoint. Things like personal health, spending more time with kids, or maintaining habits are ongoing concerns that benefit from monitoring, gentle nudges, and periodic reflection — but don't fit the discover → roadmap → implement pipeline. Consider a new activity type or waypoint pattern for ongoing monitoring: periodic check-ins, trend tracking, goal reminders, and advisory-style engagement rather than task execution.
