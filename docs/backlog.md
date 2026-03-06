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

## 2026-03-06 — w29 scope reduction

**Session-Restart Task Insertion:** When agent tasks produce new skills or artifacts that require a session restart before subsequent tasks can use them, the implement skill should recognize this and insert a dependency. Deferred from w29 — skill updates are increasingly rare, so the cost-benefit doesn't justify the complexity right now. Revisit if session-restart issues recur.
