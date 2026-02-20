---
name: research
description: This skill should be used when the user wants to "research a topic", "look up information about X", "find out about X", or when another skill needs to conduct multi-angle web research and return a structured summary. Performs broad web searches and synthesizes findings.
---

# Research

Conduct multi-angle web research on a topic and return a structured summary.

## Process

1. **Clarify the research question.** Identify the core question or topic to research. If invoked by another skill, the question is provided inline.

2. **Conduct searches.** Perform 3–5 web searches from different angles:
   - Direct factual queries
   - Expert perspectives and best practices
   - Common pitfalls and risks
   - Recent developments or trends

3. **Synthesize findings.** Compile results into a structured summary:

```markdown
## Research Summary: <Topic>

### Key Findings
- <finding 1>
- <finding 2>

### Expert Perspectives
- <perspective>

### Risks & Pitfalls
- <risk>

### Sources
- <source 1>
- <source 2>
```

4. **Return the summary.** When invoked by another skill, return the summary inline. When invoked directly, present the summary to the user.

## Notes

- This is a stub implementation. Future versions will support deeper research strategies, source quality assessment, and persistent research artifacts.
- Other skills invoke this skill via `do-stuff-helper:research` when they need domain knowledge to inform their process.
