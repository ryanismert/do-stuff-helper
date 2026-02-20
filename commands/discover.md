---
description: Start a guided discovery interview to produce a detailed brief for a project or activity
argument-hint: <optional activity description>
allowed-tools: Read, Write, Edit, Glob, Grep, WebSearch, WebFetch, Bash(git add:git commit), Skill, Task, AskUserQuestion
disable-model-invocation: true
---

Invoke the `do-stuff-helper:discover` skill to begin a discovery interview.

If `$ARGUMENTS` is provided, pass it as the initial activity description to the skill — treat it as the user's answer to "What do you want to do?" and skip directly to determining whether this is a new or existing activity.

If `$ARGUMENTS` is empty, the skill will ask the user to describe what they want to do.
