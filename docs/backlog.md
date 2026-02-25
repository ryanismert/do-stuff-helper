# Backlog

Ideas, scope changes, and discovered work that are **out of scope** for the current waypoint. Items here will be periodically reviewed by the `replan` skill and categorized as design updates (changes to the brief/roadmap) or tactical waypoints (new work items).

**To add an item:** Append a new section with the date and source.

---

## 2026-02-25 — roadmap cleanup

**Feedback Capture (formerly w7):** User can give thumbs-down or typed feedback on any worker output. Feedback is recorded with context (input, output, rating) in the activity repo. Marked obsolete because the quality strategy is skills/CLAUDE.md refinement and better data sources rather than continuous prompt feedback loops. Revisit if a lightweight feedback mechanism becomes available in Claude Code's UI.

**Skill-Improver Agent (formerly w8):** An autonomous agent periodically reviews accumulated feedback, proposes updates to skills/prompts/CLAUDE.md, and presents changes for user approval before landing. Marked obsolete because it depended on w7's feedback data. Could be reimagined with a different input source (e.g., reviewing worker output quality directly, or user-flagged issues in the backlog).
