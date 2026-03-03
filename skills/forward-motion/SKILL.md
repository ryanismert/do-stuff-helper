---
name: forward-motion
description: >
  This skill should be used when the user asks to "plan my week", "what should I focus on",
  "update my priorities", "forward motion", "what's next", "review my week", "weekly review",
  or when an n8n trigger fires for plan refresh or weekly review. It evaluates all activities
  against the user's life priorities and produces a prioritized week-ahead plan. Do NOT trigger
  for work on a specific activity or waypoint — only for cross-activity prioritization and review.
---

# Forward Motion Analyst

Maintain a living week-ahead plan that tells the user what 2-3 things to focus on next, updated as work progresses. Start from life priorities and map downward to concrete next steps. Operates in two modes: **plan** (default) and **review** (weekly retrospective + full re-evaluation).

## Modes

**Plan mode** (default): Reads all activity roadmaps, changelogs, inboxes, and user profile. Evaluates priorities using the life-priority-first lens. Produces or updates `~/exoselfai/docs/forward-motion/current-plan.json`.

**Review mode** (invoked with "review" argument, or by Sunday n8n trigger): Reads the week's plan history and activity changelogs. Produces `~/exoselfai/docs/forward-motion/review-YYYY-WNN.md`. Then runs a full plan re-evaluation for the new week.

---

## Plan Mode Checklist

### Step 1: Gather State

Read all of the following. If a file is missing, note it but continue with what is available.

1. **User profile** — `~/exoselfai/docs/user-profile.json`
   - `yearly_theme`, `overview`, `life_areas` (with `importance` ratings), `ideal_future` (major goals with `life_areas`, `success_criteria`, `timeframe`, `status`), `work_habits`, `known_tendencies`
2. **All activity roadmaps** — scan `~/exoselfai/activities/*/docs/roadmap-*.json`
   - For each: waypoint statuses, dependencies, due dates, phase membership
3. **All activity inboxes** — scan `~/exoselfai/activities/*/docs/inbox.json`
   - Pending blockers, their age, who they are waiting on
4. **All activity changelogs** — scan `~/exoselfai/activities/*/docs/changelog.md`
   - Recent progress (last 7 days) to understand momentum
5. **Existing plan** — `~/exoselfai/docs/forward-motion/current-plan.json`
   - Previous priorities, `first_appeared` timestamps, `refreshes_without_progress` counts, user feedback

If this is the first run and no plan exists, proceed to Step 2 — all tracking fields will be initialized fresh.

### Step 2: Build the Life Priority Lens

Start from the user's stated life priorities, not from activities.

1. Read major goals from `user-profile.json` → `ideal_future` — each has `life_areas`, `success_criteria`, `timeframe`, `status`
2. Read life area importance ratings from `life_areas` → `importance` (1-7 scale)
3. For each major goal, identify which activities and which waypoints advance it
4. Note which major goals have **no active work** advancing them — these are **blind spots**
5. Note which goals have approaching timeframes (e.g., "lose 10 lbs by June")

This produces a goal-to-activity mapping. Example:

```
Goal: "Launch side businesses" (importance areas: money 7, career 6, adventure 5)
  -> wildlife-cards: w3 (implementing), w4 (pending)
  -> ai-consulting: no activity exists yet  <-- blind spot

Goal: "Lose 10 lbs by June" (importance area: health 5)
  -> no activity exists  <-- blind spot, approaching deadline
```

### Step 3: Evaluate Candidates

Gather all actionable candidates:
- Waypoints ready for design, planning, or implementation (status `pending` with all deps `done`, or `implementing`/`waiting`)
- Inbox items waiting on the user
- User-facing tasks (customer outreach, decisions, phone calls — surfaced from waypoint plans or inbox)
- Goals with no activity or waypoint advancing them (blind spots to flag)

Score each candidate by:

1. **Goal alignment** — Does this advance a major goal? Weight by the importance ratings of the goal's life areas. A candidate advancing a goal tied to life areas rated 7 scores higher than one tied to areas rated 3.
2. **Leverage** — Does completing this unblock other work? Check the dependency graph across all roadmaps.
3. **Urgency** — Approaching due date or timeframe? Aging blockers (inbox items > 3 days)?
4. **Tendency correction** — Is the user's pattern of avoidance causing this to be skipped? Cross-reference with `known_tendencies` from the profile. Boost priority if the candidate addresses a tendency the user is exhibiting.

### Step 4: Select Top Priorities

Pick 2-3 items. Apply these rules:

- **At most one "comfortable" item** (building, coding, designing) — the rest should push toward harder or neglected goals
- Items aligned with the yearly theme get a boost
- If a major goal has no concrete next step, the priority might be "create an activity for X" or "unblock Y by answering inbox item Z"
- If all ready items are genuinely comfortable/easy and harder work is not actionable, note this but do not manufacture urgency

### Step 4b: Stability Principle and Staleness Tracking

The plan should feel like a reliable to-do list, not something that reshuffles unpredictably.

**On each refresh:**

- **Preserve the existing priority ranking** unless there is a concrete reason to change it
- For each item carried forward from the previous plan, increment `refreshes_without_progress` by 1 unless there is evidence of progress (changelog entry, waypoint status change, inbox resolution)
- If progress occurred, reset `refreshes_without_progress` to 0
- Preserve each item's `first_appeared` timestamp from when it was first added to the plan

**Reasons to change the list:**
- A priority item is marked done (rotate in a new one)
- User gives feedback on an item (defer, skip, blocked — re-evaluate)
- A due date becomes urgent
- The weekly Sunday run (this is where bigger re-evaluations happen)

**Reasons NOT to change the list (mid-week):**
- A new waypoint became unblocked in some activity
- A new activity was created
- Minor progress on a non-priority waypoint
- Routine inbox items arriving

**Staleness signal:** Items with `refreshes_without_progress >= 3` are flagged as stalled. This is the primary staleness indicator — not waypoint age. Stalled items get a tendency flag noting the pattern.

Mid-week refreshes are **incremental**. The Sunday run is the **full re-evaluation** where rankings can shift more freely.

### Step 5: Tendency Check

Read `user-profile.json` -> `known_tendencies` and check for these patterns:

1. **Easy over hard** — Are completed items mostly low-complexity while high-impact priorities linger? Look at recent changelog entries vs. plan priorities.
2. **Building over selling** — For business activities, is progress all on product/code while customer-facing tasks (outreach, sales, marketing) persist untouched?
3. **Air gap persistence** — Inbox items aging beyond 3 days, waypoints in `waiting` with no movement.
4. **Self-deprioritization** — Personal goals (health, relationships, fun) consistently absent from the plan or always ranked last.
5. **Plan-item avoidance** — Items lingering on the plan for 3+ refreshes without progress (`refreshes_without_progress >= 3`).

**Tendency flags are constructive nudges, not guilt trips.** Only surface a flag when the pattern is actually present in the data. Do not flag every week by default.

Attach relevant `tendency_flag` strings to individual priority items when applicable. Example flags:
- `"This has been on the plan for 4 refreshes without progress — possible avoidance pattern"`
- `"All recent progress is on building; customer outreach remains untouched"`
- `"Health goals have no active work despite approaching timeframe"`

### Step 6: Process User Feedback

Check each priority item for `user_feedback`. If set, process it:

| Feedback | Action |
|---|---|
| "Not important right now" / "Deprioritize" | Drop the item, replace with next best candidate |
| "Deferred until [date]" | Set `deferred_until` date, remove from active plan |
| "Blocked on [something]" | Hold item with blocker note visible, do not replace |
| "Already working on this" / "Started" | Update `status` to `in_progress` |
| "Done" | Update `status` to `done`, trigger refresh to rotate in new priority |
| Free-form context | Store in `user_feedback`, factor into next reasoning pass |

When user provides any feedback, reset `refreshes_without_progress` to 0 for that item.

### Step 7: Write Plan

Ensure the directory exists: `~/exoselfai/docs/forward-motion/`

Write `~/exoselfai/docs/forward-motion/current-plan.json` with the following schema:

```json
{
  "week": "YYYY-WNN",
  "created": "ISO-8601 timestamp of first creation this week",
  "refreshed": "ISO-8601 timestamp of this refresh",
  "life_priority_snapshot": [
    {
      "goal": "Goal description from profile",
      "timeframe": "end of year | by June | etc.",
      "status": "active | not_started | completed",
      "coverage": "Summary of which activities/waypoints advance this goal, or 'not started'"
    }
  ],
  "priorities": [
    {
      "rank": 1,
      "summary": "Short description of what to focus on",
      "activity": "activity-slug or null for blind spots",
      "waypoint": "w6 or null",
      "type": "waypoint | inbox | user_task | blind_spot",
      "rationale": "Why this is the most important thing right now",
      "goal_alignment": ["Goal description 1", "Goal description 2"],
      "status": "not_started | in_progress | done",
      "first_appeared": "ISO-8601 timestamp",
      "refreshes_without_progress": 0,
      "tendency_flag": "string or null",
      "user_feedback": "string or null",
      "deferred_until": "ISO-8601 date or null"
    }
  ],
  "blocked_on_user": [
    {
      "activity": "activity-slug",
      "item_id": "q-1",
      "subject": "Description of what's blocked",
      "age_days": 5
    }
  ],
  "blind_spots": [
    {
      "goal": "Goal with no active work",
      "observation": "Description of the gap",
      "suggestion": "Concrete next step to address it"
    }
  ],
  "tendency_observations": [
    "Only present when a pattern is actually detected. Constructive, not guilt-inducing."
  ],
  "next_review": "YYYY-MM-DD (next Sunday)"
}
```

**Field notes:**
- `week` uses ISO week numbering (e.g., `2026-W10`)
- `created` is set once when the plan is first written for a new week; `refreshed` updates on every run
- `priorities` should contain 2-3 items (rarely more)
- `blocked_on_user` surfaces inbox items that need the user's attention, sorted by age descending
- `blind_spots` surfaces major goals with no active work
- `tendency_observations` is a top-level array for general patterns; item-level `tendency_flag` is for per-priority nudges

### Step 8: Present the Plan

After writing the file, present the plan to the user in a readable format:

1. Show the 2-3 priorities with rank, summary, and rationale
2. If there are items blocked on the user, list them with age
3. If there are blind spots, mention them
4. If tendency flags are present, share them as constructive observations
5. Ask if the user wants to adjust anything (feedback will be processed on next refresh)

---

## Review Mode Checklist

Review mode runs on Sundays (via n8n trigger) or when the user asks for a "weekly review". It produces a retrospective, then runs a full plan re-evaluation.

### Review Step 1: Gather Week Data

1. Read the existing `current-plan.json` — note the `created` date to determine the week boundary
2. Read all activity changelogs (`~/exoselfai/activities/*/docs/changelog.md`) for entries within the past 7 days
3. Read all activity roadmaps for current waypoint statuses
4. Read all activity inboxes for aging items

### Review Step 2: Analyze the Week

For each activity with recent changelog entries:
- Summarize what moved forward
- Note what was planned but did not move

Cross-reference with the week's plan priorities:
- Which priorities were addressed?
- Which were ignored or stalled?
- Was effort spent on unplanned work?

### Review Step 3: Write the Review

Produce `~/exoselfai/docs/forward-motion/review-YYYY-WNN.md` using this template:

```markdown
# Week NN Review — YYYY-MM-DD

## Progress
- [activity]: [what moved forward]
- [activity]: [what moved forward]

## Stalled
- [activity]: [what didn't move, why if known]

## Effort vs. Priority
[Were the week's priorities addressed? Where did time actually go?
Note any divergence between planned priorities and actual work.]

## Aging Blockers
- [item]: [age], [what's needed to unblock]

## Tendency Watch
[Only include this section when a pattern is actually present.
Reference specific evidence, not general warnings.]

## Goal Coverage
[Which major goals got attention this week, which didn't.
Blind spots identified or resolved.]
```

### Review Step 4: Present the Review

Show the review to the user. Ask for any reflections or corrections before saving.

### Review Step 5: Full Re-Evaluation

After the review is saved, run a **full plan re-evaluation** for the new week:
- Execute Plan Mode Steps 1-8 with fresh scoring (not incremental)
- The new plan's `created` timestamp marks the start of the new week
- All `refreshes_without_progress` counters carry forward (they do not reset on new week — only on actual progress)
- Rankings can shift freely on the Sunday run, unlike mid-week incremental refreshes

---

## Edge Cases

- **No user profile exists:** Warn the user and suggest running `do-stuff-helper:user-profile-builder`. Proceed with activity data only, but skip life-priority lens and tendency checks.
- **No activities exist:** Produce an empty plan noting that no activities are set up. Suggest the user start with `do-stuff-helper:discover`.
- **Only one activity exists:** Still apply the life-priority lens — the user may have major goals with no activity yet (blind spots).
- **All priorities are comfortable/easy:** Note this in the plan but do not force uncomfortable items if none are actionable. The tendency observation should mention it.
- **User feedback conflicts with analysis:** User feedback always wins. If the user deprioritizes something the analysis ranks highly, respect it and note the reasoning.
- **Plan file is corrupted or has unexpected schema:** Re-create from scratch. Log that the previous plan was unreadable.
- **Changelog has no entries for the review period:** Note "no recorded progress" in the review. This itself is a signal worth surfacing.
- **Multiple activities advance the same goal:** This is fine and expected. The plan should pick the highest-leverage waypoint across all of them, not one per activity.

## Cross-Skill Invocation

- **Invoked by n8n triggers** — Plan mode fires on waypoint completion, inbox resolution, or roadmap changes. Review mode fires on the Sunday schedule.
- **`do-stuff-helper:user-profile-builder`** — Suggest if no user profile exists (Step 1 fallback).
- **`do-stuff-helper:discover`** — Suggest if blind spots identify goals with no activity at all.
- **`do-stuff-helper:waypoint-implement`** — After presenting the plan, the user may choose to start working on a priority. Suggest invoking the implement skill for waypoint-type priorities.
- **`do-stuff-helper:replan`** — If the review surfaces structural issues (waypoints that should be split, dependencies that are wrong), suggest running replan.
