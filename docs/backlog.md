# Backlog

Ideas, scope changes, and discovered work that are **out of scope** for the current waypoint. Items here will be periodically reviewed by the `replan` skill and categorized as design updates (changes to the brief/roadmap) or tactical waypoints (new work items).

**To add an item:** Append a new section with the date and source.

---

## 2026-02-25 — roadmap cleanup

**Feedback Capture:** A way for users to give quick feedback (thumbs-down, typed comments) on worker output, recorded with context (input, output, rating) in the activity repo. Revisit if a lightweight feedback mechanism becomes available in Claude Code's UI.

**Skill-Improver Agent:** An agent that periodically reviews worker output quality or user-flagged issues, proposes updates to skills/prompts/CLAUDE.md, and presents changes for user approval before landing. Could be triggered by patterns in the backlog, poor worker output, or direct user requests.

## 2026-02-25 — user

**Dashboard mobile and at-a-glance improvements:** The dashboard looks nice on mobile but needs to be more compact and show more information at a glance. Consider condensing the layout — tighter spacing, smaller cards, collapsible sections, or a summary bar at the top with key stats. Same improvements would benefit the desktop version too.

## 2026-03-05 — replan (from w24)

**Agent Teams Integration:** Claude Code's experimental agent teams feature uses the same CLAUDE_CODE_TASK_LIST_ID mechanism we build on. Integrating it would let waypoint-implement spawn persistent teammate sessions instead of one-shot Task subagents, enabling longer-running workers and native coordination. Revisit when agent teams matures or when worker session duration becomes a bottleneck.

## 2026-03-06 — w27 verification

**Verify W27 Replan Reminder End-to-End:** The weekly n8n workflow (replan-reminder-weekly) needs real-world verification with an activity that has 6+ backlog items. Likely candidate is the upcoming Overland activity once it accumulates enough backlog. Trigger the workflow manually or wait for Sunday and confirm it creates inbox notifications correctly.

## 2026-03-15 — user feedback

**Skip Human Review in Roadmap Skill:** The roadmap skill currently asks the user to review and confirm the roadmap before saving. So far, every review round has been the user approving without changes — the agent's judgment on roadmap structure has been reliable. Consider removing the human review gate and proceeding directly from roadmap construction to saving. Problems can be caught during implementation and fed back through replan. This would reduce friction and let activities flow from discover → roadmap → waypoint-design without pausing for approval.

## 2026-03-06 — w29 scope reduction

**Session-Restart Task Insertion:** When agent tasks produce new skills or artifacts that require a session restart before subsequent tasks can use them, the implement skill should recognize this and insert a dependency. Deferred from w29 — skill updates are increasingly rare, so the cost-benefit doesn't justify the complexity right now. Revisit if session-restart issues recur.

## 2026-03-17 — user feedback

**Dashboard Activity Ordering:** Add the ability to reorder activities on the dashboard so the most important ones appear at the top. Currently activities are displayed in an uncontrolled order — the user wants manual priority ordering to keep focus on what matters most.

## 2026-03-17 — user feedback

**Forward Motion Planner Revisit:** Two issues: (1) The skill always tops up to three items in the weekly plan, which may not be useful once activities are completed or priorities shift — the fixed count doesn't adapt to actual capacity or remaining work. (2) The n8n trigger for forward motion isn't firing consistently, so the automated weekly planning cycle isn't reliable. Review both the skill logic and the n8n workflow to ensure forward motion works as intended.
