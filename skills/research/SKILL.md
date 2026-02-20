---
name: research
description: This skill should be used when the user wants to "research a topic", "look up information about X", "find out about X", or when another skill needs to conduct multi-angle web research and return a structured summary. Performs broad web searches, identifies knowledge gaps, offers deep research handoff, and synthesizes findings into a persistent research artifact.
---

# Research

Conduct multi-angle web research on a topic, identify gaps, offer deep research handoff, and save a structured summary to the activity's research archive.

## Checklist

Complete each step in order.

### Step 1: Clarify Research Question

- If invoked by another skill, the question is provided inline. Use it directly.
- If invoked directly by the user, ask: "What would you like me to research?"

Once you have the main question, identify **3–5 specific sub-questions** that together would answer it comprehensively. Present these to the user briefly so they can confirm or adjust the scope.

### Step 2: Multi-Angle Web Search

Conduct **5–8 web searches** using `WebSearch`, targeting different angles:

- Direct factual queries (definitions, current state)
- Expert perspectives and best practices
- Common pitfalls, risks, and failure modes
- Recent developments and trends
- Comparative/alternative approaches
- Cost, timeline, and resource considerations (if applicable)

For each search, use `WebFetch` to read the **2–3 most relevant results** in full.

**Parallelization:** Use the `Task` tool with `subagent_type=general-purpose` to run **up to 3 parallel search agents**, each covering a different angle. Give each agent a clear research angle and instruct it to return structured findings with source URLs.

### Step 3: Assess Coverage & Gaps

Evaluate which sub-questions from Step 1 are:
- **Well-answered** — multiple corroborating sources, clear consensus
- **Thin** — only one source or surface-level coverage
- **Conflicting** — sources disagree on key points
- **Unanswered** — no useful results found

Identify critical unknowns that web search alone cannot resolve.

### Step 4: Deep Research Offer

**If significant gaps exist** (thin, conflicting, or unanswered sub-questions on important topics), tell the user:

> "I can't run deep research from Claude Code, but if you have a deep research report on **[suggested topic/query]** — or can run one — you can paste the content or point me to a saved file. Otherwise, I'll proceed with what I have and note the gaps."

If the user provides deep research results:
- Accept pasted text, file paths, or uploaded files
- Read and incorporate the content
- Save any user-provided deep research to `<activity-dir>/docs/research/` with a descriptive filename

**If no significant gaps exist**, skip this step entirely.

### Step 5: Synthesize Findings

Compile all results (web search + any deep research) into a structured summary:

```markdown
# Research Summary: <Topic>

## Key Findings
- <finding with source citation>

## Expert Perspectives
- <perspective with source>

## Risks & Pitfalls
- <risk with source>

## Comparative Analysis
- <comparison if applicable>

## Open Questions
- <unresolved questions, gaps noted>

## Sources
- [Source title](URL) — brief relevance note
```

**Source quality:** Prefer authoritative sources (official docs, academic papers, established industry publications) over SEO content farms. Note source quality where relevant.

### Step 6: Save

1. Ensure the activity directory exists. If invoked by another skill, the activity directory is already set up. If invoked directly, ask the user which activity this research belongs to, or create a standalone `docs/research/` directory if no activity applies.
2. Save the research output to `<activity-dir>/docs/research/research-<topic-slug>.md`
3. If invoked by another skill, return the summary inline for incorporation into their process.
4. If invoked directly, present the summary to the user and confirm the save location.

## Cross-Skill Invocation

- Other skills invoke this skill via `do-stuff-helper:research` when they need domain knowledge.
- When invoked by another skill, skip user-facing prompts (Step 1 clarification, Step 4 deep research offer) unless the invoking skill explicitly requests user interaction.
