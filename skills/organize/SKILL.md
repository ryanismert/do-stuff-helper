---
name: organize
description: This skill should be used when the user wants to "create a new activity", "set up a project folder", "organize a new project", or when another skill needs to initialize an activity directory under `activities/`. Creates the standard directory structure for a new activity.
---

# Organize

Create and initialize the directory structure for a new activity.

## Process

1. **Determine the activity slug.** Convert the activity name to a lowercase, hyphenated slug (e.g., "My Fitness App" → `my-fitness-app`). Confirm the slug with the user if ambiguous.

2. **Create the activity directory.**

```
activities/<activity-slug>/
└── README.md
```

3. **Write the README.** Include the activity name, creation date, and a placeholder description.

```markdown
# <Activity Name>

Created: <YYYY-MM-DD>

## Description

_To be filled in by the discovery process._
```

4. **Confirm creation.** Report the path of the new activity directory to the caller or user.

## Notes

- This is a stub implementation. Future versions will support additional structure (e.g., `research/`, `plans/`, `artifacts/` subdirectories).
- Other skills invoke this skill via `do-stuff-helper:organize` when initializing new activities.
