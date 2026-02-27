---
name: user-profile-builder
description: This skill should be used when the user says "build profile", "life inventory", "update profile", "user profile", "review my life areas", or wants to create or update a structured personal profile covering life areas, goals, strengths, and work habits. Conducts an AI-guided interview adapted from the 8760 Hours methodology and produces a JSON profile at ~/exoselfai/docs/user-profile.json.
---

# User Profile Builder

Conduct a structured life inventory interview to build or update a personal profile. The profile captures present reality, ideal future, major goals, and a self-assessment of skills, strengths, and habits across twelve life areas.

**Tone:** This is a personal interview, not a technical task. Be warm, supportive, patient, and curious. Give the user space to think. Do not rush.

## File Paths

- **Profile:** `~/exoselfai/docs/user-profile.json`
- **Draft:** `~/exoselfai/docs/user-profile-draft.json`
- **Interview guide:** Read from `references/interview-guide.md` (relative to this skill's directory)

## Life Area Keys

`values_purpose`, `contribution_impact`, `location_tangibles`, `money_finances`, `career_work`, `health_fitness`, `education_skill_development`, `social_relationships`, `emotions_wellbeing`, `character_identity`, `productivity_organization`, `adventure_creativity`

## Checklist

Complete each step in strict order. Do not skip steps.

### Step 1: Determine Mode

Check if `~/exoselfai/docs/user-profile.json` exists.

- **If no profile exists:** This is a **Full Interview**. Go to Step 2.
- **If a profile exists:** Ask the user which mode they want:
  - **Update Mode** — Review and revise specific areas of the existing profile. Go to Step 10.
  - **Fresh Start** — Discard the existing profile and run a full interview from scratch. Go to Step 2.

### Step 2: Check for Draft

Check if `~/exoselfai/docs/user-profile-draft.json` exists.

- **If a draft exists:** Read it and present a summary of what has been completed so far. Ask:
  > "I found an in-progress interview from a previous session. Would you like to resume where we left off, or start fresh?"
  - If resume: Load the draft state and skip to the appropriate phase/area.
  - If start fresh: Delete the draft and continue to Step 3.
- **If no draft exists:** Continue to Step 3.

### Step 3: Read Interview Guide

Read `references/interview-guide.md` (relative to this skill's directory). This contains all interview questions, rating scales, Hamming questions, and progressive depth guidance. Use it as your reference throughout the interview. Do not recite it to the user.

### Step 4: Phase 1 — Initial Overview

Run the Phase 1 warm-up from the interview guide. This should take 5-10 minutes.

- Ask the backward-looking and current-situation questions conversationally (not as a numbered list).
- Aim for roughly ten items across all questions.
- Be objective and descriptive. If the user starts making judgments or plans, note them separately and redirect to the descriptive picture.
- Capture responses for the profile's `overview` section: `what_went_well`, `what_didnt_go_well`, `tried_hard`, `didnt_try_hard_enough`.

When Phase 1 is complete, save progress to the draft file. Move to Step 5.

### Step 5: Phase 2 — Present Reality

For each of the twelve life areas, conduct a deep dive using the interview guide's Phase 2 questions.

**For each area:**

1. Briefly introduce the area using the description from the interview guide.
2. Start broad with the general prompt: "How would you describe the current state of [area]?"
3. Apply **progressive depth** — identify themes in the user's response and offer to branch deeper. Use area-specific follow-up questions from the guide as probes. Do not rapid-fire questions; let the conversation flow naturally.
4. When the area feels covered, summarize what you captured and ask: "Does that feel complete, or is there anything to add?"
5. Ask for the **rating (1-7)**: "On a scale of 1 to 7, where 1 is very bad and 7 is very good, how would you rate this area right now?"
6. Ask the **Hamming question**: "What's the single biggest bottleneck or most important problem in this area?"
7. **Save to draft immediately** after completing each area. Write the area data to the draft file so progress is never lost.

**Pacing:**
- It is better to do fewer areas well than to rush through all twelve.
- If the user is fatiguing, suggest saving progress and resuming later. The draft file preserves all progress.
- If the user wants to skip an area, that is fine. Leave it empty in the profile.
- Ask ONE question at a time. Wait for the user's response before continuing.

When all areas are complete (or the user wants to move on), proceed to Step 6.

### Step 6: Phase 3 — Ideal Future & Goals

Run the Phase 3 interview from the guide. Cover these topics:

1. **Per-area ideal future:** For each area, ask what it would look like if it were ideal. Push for specificity. Capture as the `ideal` field.
2. **Importance rating (1-7):** For each area, ask how important it is to focus on over the next year. Capture as the `importance` field.
3. **Yearly theme:** Ask: "If you had to pick a theme for the coming year, what would it be?"
4. **Major goals:** Extract 3-5 concrete goals. For each, capture: the goal description, which life areas it touches, success criteria, and timeframe. Encourage at least one meta-skill goal that improves multiple areas.

Save to draft after completing this phase. Proceed to Step 7.

### Step 7: Phase 4 — Skills, Strengths & Habits

Run the Phase 4 interview from the guide. Cover:

1. **Work habits:** Productive patterns, unproductive patterns, best work times, learning style.
2. **Known tendencies:** Avoidance patterns, what the user gravitates toward, procrastination triggers.
3. **Strengths & skills:** Technical skills, soft skills, domain expertise, key strengths.

Capture results for the `skills_strengths_habits` section. Save to draft after completing this phase. Proceed to Step 8.

### Step 8: Compile and Save Profile

1. Compile all interview data into the final profile JSON. Use this schema:

```json
{
  "version": 1,
  "created": "<ISO timestamp>",
  "updated": "<ISO timestamp>",
  "yearly_theme": "...",
  "overview": {
    "what_went_well": [],
    "what_didnt_go_well": [],
    "tried_hard": [],
    "didnt_try_hard_enough": []
  },
  "life_areas": {
    "<area_key>": {
      "summary": "Free-text summary of current state",
      "rating": 5,
      "ideal": "Free-text description of ideal state",
      "importance": 6,
      "bottleneck": "The single biggest issue here",
      "details": ["Key points captured during interview"],
      "rating_history": [
        { "date": "YYYY-MM-DD", "rating": 5 }
      ]
    }
  },
  "major_goals": [
    {
      "goal": "Description",
      "life_areas": ["area_key_1", "area_key_2"],
      "success_criteria": "How to measure success",
      "timeframe": "next year",
      "status": "active"
    }
  ],
  "skills_strengths_habits": {
    "strengths": [],
    "skills": {
      "technical": [],
      "soft": [],
      "domain": []
    },
    "work_habits": {
      "productive_patterns": [],
      "unproductive_patterns": [],
      "best_work_times": "...",
      "learning_style": "..."
    },
    "known_tendencies": {
      "avoidance_patterns": [],
      "gravitates_toward": [],
      "procrastination_triggers": []
    }
  }
}
```

Each life area uses the keys listed above. All twelve areas should be present in the profile, even if some are empty.

2. Write the profile to `~/exoselfai/docs/user-profile.json`.
3. Delete the draft file `~/exoselfai/docs/user-profile-draft.json`.
4. Present a summary to the user: overall theme, highest/lowest rated areas, top goals.

Proceed to Step 9.

### Step 9: Commit

1. Stage and commit `~/exoselfai/docs/user-profile.json` with message: `feat: build user profile via life inventory interview`
2. Confirm the profile is saved and committed.
3. Let the user know they can update the profile anytime by asking to "update my profile" or "review my life areas."

---

## Update Mode

### Step 10: Load Existing Profile

1. Read `~/exoselfai/docs/user-profile.json`.
2. Ask what triggered the update (new activity, periodic review, life change, etc.).

### Step 11: Show Summary

Present a summary of the current profile:

- Yearly theme
- Each life area with its current rating and last-updated date from `rating_history`
- Major goals and their statuses

### Step 12: Select Areas to Update

Ask which areas the user wants to review. Use a structured question listing all twelve areas with their current ratings. The user can select one or more areas, or choose "all."

### Step 13: Interview Selected Areas

For each selected area:

1. Read the interview guide (`references/interview-guide.md`) if not already loaded.
2. Show the current summary and rating for the area.
3. Run the Phase 2 deep dive for this area (same process as Step 5, but for selected areas only).
4. Ask if the ideal future (Phase 3) for this area needs updating too.
5. If yes, run the Phase 3 ideal/importance questions for this area.

### Step 14: Update Goals and Habits

After updating the selected areas, ask:

- "Do any of your major goals need updating based on what we discussed?"
- "Has anything changed about your work habits or tendencies?"

If yes, interview those sections as well.

### Step 15: Merge and Save

1. Merge the updated data into the existing profile:
   - For updated life areas: replace `summary`, `rating`, `ideal`, `importance`, `bottleneck`, and `details` with new values.
   - **Archive previous ratings:** Append the old rating with its date to `rating_history` before overwriting.
   - For goals: update status, add new goals, or remove obsolete ones as discussed.
   - For skills/strengths/habits: merge new data with existing.
2. Update the `updated` timestamp.
3. Write to `~/exoselfai/docs/user-profile.json`.
4. Present a summary of what changed.
5. Stage and commit with message: `feat: update user profile — <brief description of what changed>`
6. Confirm the update is saved and committed.

## Edge Cases

- **User declines to continue:** Save whatever has been captured to the profile. A partial profile is better than none.
- **User wants to skip areas:** Leave those areas empty. Do not pressure the user to fill every area.
- **Session interrupted:** The draft file preserves all progress. On next invocation, offer to resume.
- **User gives very brief answers:** Offer to go deeper but respect their preference. Some areas may naturally be sparse.
