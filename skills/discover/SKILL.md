---
name: discover
description: This skill should be used when the user says "help me plan X", "let's figure out X", "I'm thinking about X", "help me think through X", or otherwise describes a new project, program, or activity they want to undertake. Conducts an expert-driven interview to produce a detailed brief that serves as the foundation for future planning.
---

# Discover

Conduct an iterative, expert-driven interview to produce a descriptive brief for any project, program, or activity. The brief must contain enough detail that a future roadmap skill can plan execution without re-interviewing the user.

## Checklist

Complete each step in strict order. Do not skip steps. Do not proceed to brief construction (step 8) until the interview is complete and the user confirms they are done (step 7).

### Step 1: Determine New vs. Existing Activity

Ask the user if they want to plan a new activity. 

- **If the user says no:** Do not use this skill. Just proceed with the conversation and ask the user how you can help.
- **If the user says yes:** This is a new activity. Proceed to step 2.

### Step 2: Invoke Organize

Invoke `do-stuff-helper:organize` to create the activity directory structure. Pass the activity name. Confirm the slug and path with the user.

### Step 3: Collect Existing Materials

Ask the user if they have any existing materials related to this activity:

- Notes, documents, prior research
- Links, bookmarks, references
- Previous conversations or plans
- Relevant files on their machine

If materials exist, read and incorporate them. If not, acknowledge and move on.

### Step 4: Initial Description

Let the user describe the activity in their own words. Do not interrupt or steer — capture their natural framing. Ask a single open-ended question:

> "Describe what you want to do and why. Include as much or as little detail as feels right."

Listen actively. Note key themes, goals, constraints, and motivations for use in subsequent steps.

### Step 5: Expert Context Analysis

Based on the user's description, determine:

1. **Domain(s):** What fields or disciplines does this activity span?
2. **Expert knowledge:** What would a domain expert know that a generalist would not?
3. **Expert questions:** What would an expert ask to evaluate this activity's feasibility and design?
4. **Expert concerns:** What risks or pitfalls would an expert worry about?

Do not present this analysis to the user. Use it internally to guide the interview in step 6.

### Step 6: Iterative Expert Interview

Conduct the interview one question at a time. Each question should:

- Draw on the expert context from step 5
- Build on previous answers (do not repeat ground already covered)
- Target a specific aspect of the brief template (goal, scope, success criteria, risks, etc.)
- Be concrete and specific, not generic

**Pacing rules:**
- Ask ONE question per message
- Wait for the user's response before asking the next question
- Adapt follow-up questions based on what the user reveals
- Research can be invoked **at any point during the interview** when a topic surfaces that would benefit from investigation — trust your judgment on when research helps

**Coverage targets** (ensure the interview addresses all of these before moving to step 7):
- Goal and motivation
- Success criteria and how to measure them
- Scope boundaries (what is in, what is out)
- Key risks and mitigation ideas
- Timeline expectations or constraints
- Resource requirements or constraints
- Dependencies on external factors
- Background context that a planner would need

### Step 7: Coverage Check

When the interview has covered all targets from step 6, ask the user:

> "We've covered [list topics]. Are there other angles or topics you'd like to explore before I draft the brief? Or does this feel complete?"

- If the user raises new topics, return to step 6 and continue the interview.
- If the user confirms they are done, proceed to step 8.

**Hard gate: Cannot proceed to step 8 until the user explicitly confirms the interview is complete.**

### Step 8: Construct the Brief

Draft the brief using the template below. Fill every section with specific, concrete content drawn from the interview. Do not use placeholder text.

```markdown
# Brief: <Activity Name>

## Goal
What the activity aims to achieve, stated clearly and concisely.

## Why
The motivation — why this matters to the user.

## Success Criteria
Concrete, observable indicators that the activity has succeeded.

## Measurement
How each success criterion will be measured or evaluated.

## Description
What is the end state? What intermediate steps or accomplishments does the user imagine taking on the way. If the activity includes building something, what is that thing and what does it do?

## Scope

### In Scope
What is included in this activity.

### Out of Scope
What is explicitly excluded.

## Key Risks
Identified risks with brief mitigation notes where discussed.

## Open Questions
Unresolved questions that need answers before or during execution.

## Background & Context
Domain knowledge, prior work, constraints, and any other context a planner would need.

## Notes for Roadmap Planning
Timeline expectations, resource constraints, dependencies, phasing suggestions, and anything else relevant to building an execution plan.
```

### Step 9: Self-Review

Before presenting the brief, review it against this checklist:

- [ ] Understandable by someone unfamiliar with the conversation
- [ ] Success criteria are concrete and observable
- [ ] Risks reflect expert-level thinking (not just generic risks)
- [ ] Scope boundaries are clear
- [ ] Roadmap notes contain enough detail for a planning skill to build an execution plan
- [ ] No placeholder or vague language remains
- [ ] Open questions are genuine (not things already answered in the interview)

Note any gaps found during self-review for step 10.

### Step 10: Final Questions

If the self-review surfaced gaps, ask the user targeted questions to fill them. Keep this round brief — 1–3 questions maximum. Incorporate answers into the brief.

If no gaps were found, skip to step 11.

### Step 11: Save and Update CLAUDE.md

1. Save the brief to `<activity-dir>/docs/brief-<activity-slug>.md`
2. Open `<activity-dir>/CLAUDE.md` and append an `## Activity Brief` section containing:
   - **Goal:** 1–2 sentence summary of the goal
   - **Success Criteria:** Bulleted list of the key success criteria
   - **Scope:** Brief summary of what's in and out of scope
   - **Key Risks:** Top 2–3 risks
   - A link to the full brief with a note to read it for complete context:
     ```
     For full details including background, open questions, and roadmap planning notes, see [the complete brief](docs/brief-<activity-slug>.md).
     ```
3. Stage and commit all changes with message: `discover: add brief for <activity-slug>`
4. Report the file paths and confirm everything is saved.

## Cross-Skill Invocation

- **`do-stuff-helper:organize`** — Invoke in step 2 for new activities. Required before any files can be saved to the activity directory.
- **`do-stuff-helper:research`** — Invoke at any point during the interview (step 6) when a topic surfaces that would benefit from investigation.
