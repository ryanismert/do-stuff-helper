---
name: organize
description: This skill should only be invoked directly by other skills (e.g., `do-stuff-helper:organize`) or by the user explicitly asking to "organize a new activity" or "set up a new project". It bootstraps the current activity directory with docs, CLAUDE.md, plugins, and a GitHub repository.
disable-model-invocation: true
---

# Organize

Bootstrap the current activity directory: scaffold it with docs and configuration, install plugins at project level, initialize CLAUDE.md, and set up a GitHub repository.

**Assumption:** The user has already created the activity directory and is running inside it. The do-stuff-helper plugin is installed at the user level, making this skill available. This skill ensures do-stuff-helper is also installed at the project level so collaborators who check out the repo get the same skills.

## Checklist

Complete each step in order. Do not skip steps.

### Step 1: Create docs/ Subdirectory

Check if `docs/` exists in the current directory. If not, create it:

```bash
mkdir -p docs
```

### Step 2: Install Plugins

Check whether each plugin is already installed **at the project level**. Install any that are missing:

1. **do-stuff-helper** — `claude plugin install do-stuff-helper@do-stuff-helper`
   This ensures collaborators who clone the repo also get the do-stuff-helper skills. The user already has it at user level, but project-level installation makes it portable.
2. **superpowers** — `claude plugin install superpowers@claude-plugins-official`
3. **claude-md-improver** — Install the `claude-md-management` marketplace and the `claude-md-improver` plugin from it

Only run install commands for plugins that are not already present at project level.

### Step 3: Initialize CLAUDE.md

Check if `CLAUDE.md` exists in the current directory.

**If it does not exist**, run `claude init` to create a baseline CLAUDE.md.

**Then, regardless of whether CLAUDE.md existed or was just created:**

1. Read the current contents of `CLAUDE.md`.
2. Check whether a `## do-stuff-helper` section exists in the file.
3. **If the section is missing**, append the full block below to the end of CLAUDE.md.
4. **If the section already exists**, update the `### Available Skills` list to match the canonical list below, and ensure the `### Task List` subsection exists with the correct task list ID (see Step 4).

The canonical do-stuff-helper section:

```markdown
## do-stuff-helper

This project uses the do-stuff-helper plugin for guided project execution.

### Available Skills
- **organize** — Bootstrap project directory, plugins, and GitHub repo
- **discover** — Expert-driven interview to produce a detailed project brief
- **research** — Multi-angle web research with structured summaries
- **roadmap** — Build an adaptive waypoint-based execution plan from the brief
- **waypoint-design** — Design individual waypoints with sufficient detail for decomposition
- **waypoint-planner** — Decompose a waypoint into executable tasks
- **waypoint-implement** — Execute tasks from the waypoint plan
- **replan** — Process the backlog into roadmap and brief updates

### Task List
This activity uses `CLAUDE_CODE_TASK_LIST_ID=<activity-slug>` for persistent cross-session task tracking.

### Capabilities

Workers and agents in this project have access to:

**Skills:** organize, discover, research, roadmap, waypoint-design, waypoint-planner, waypoint-implement, replan

**MCP Servers:** <populated by organize>

**CLI Tools:** <populated by organize>

**Project Scripts:** <populated by organize>

### Usage
Invoke skills via `do-stuff-helper:<skill-name>`.
```

Replace `<activity-slug>` with the basename of the current directory (e.g., if the directory is `/home/user/projects/my-fitness-app`, the slug is `my-fitness-app`).

### Step 3b: Discover and Populate Capabilities

After initializing CLAUDE.md, discover the capabilities available in the activity environment and populate the `### Capabilities` subsection.

#### MCP Servers

1. Check if `.mcp.json` exists in the current directory. If so, read it and extract the server names from the top-level keys. For each server, include the name and a brief description of what it does based on the server configuration (e.g., command, args, or URL).
2. Also check `~/.claude.json` for user-level MCP servers under the `mcpServers` key. Include these as well with a note that they are user-level.
3. Combine both lists. If no MCP servers are found, write "None configured".

#### CLI Tools

Check PATH for these key CLI tools: `gh`, `node`, `bun`, `docker`, `python3`, `npm`, `npx`, `make`.

For each tool, run `which <tool>` to check if it exists. List only the tools that are found, including their version (run `<tool> --version` to get it). If none are found, write "None detected".

#### Project Scripts

1. Check if `package.json` exists. If so, read the `scripts` section and list each script name and its command.
2. Check if `Makefile` exists. If so, list the available targets (parse lines matching `^target-name:` pattern).
3. If neither file exists or no scripts/targets are found, write "None (no package.json or Makefile)".

#### Update CLAUDE.md

Replace the placeholder values in the `### Capabilities` subsection of `CLAUDE.md` with the discovered values. Format each category as a comma-separated list or a brief bullet list depending on the number of items.

### Step 4: Configure Task List ID

Derive the task list ID from the current directory basename (`<activity-slug>`).

1. Check if `.claude/settings.json` exists.
2. **If it does not exist**, create the directory and file:
   ```bash
   mkdir -p .claude
   ```
   Write `.claude/settings.json` with:
   ```json
   {
     "env": {
       "CLAUDE_CODE_TASK_LIST_ID": "<activity-slug>"
     }
   }
   ```
3. **If it already exists**, read the file, merge the `env.CLAUDE_CODE_TASK_LIST_ID` key without overwriting other settings, and write it back.

### Step 5: Git and GitHub Setup

Check if the current directory is inside a git repository:

```bash
git rev-parse --is-inside-work-tree
```

**If not a git repo:**

1. Initialize: `git init`
2. Create a `.gitignore` with sensible defaults (`.DS_Store`, `node_modules/`, `.env`, etc.)
3. Determine the GitHub username. Try `gh api user -q .login` first. If that fails, ask the user.
4. Derive the repo name from the directory basename.
5. Create the repo: `gh repo create <username>/<repo-name> --private --source=. --remote=origin`
6. Stage all files: `git add -A`
7. Commit: `git commit -m "Initial commit"`
8. Push: `git push -u origin main`

**If a git repo but no remote named `origin`:**

1. Determine GitHub username as above.
2. Derive repo name from directory basename.
3. Create the repo: `gh repo create <username>/<repo-name> --private --source=. --remote=origin`
4. Stage, commit, and push any changes from setup steps.

**If a git repo with a remote:**

1. Stage, commit, and push any uncommitted changes from the setup steps.

### Step 6: Confirm

Report a summary of what was done:

- Activity directory path (current directory)
- Whether `docs/` was created
- Whether CLAUDE.md was initialized or updated (and what changed)
- Whether plugins were installed (and which ones)
- Whether `.claude/settings.json` was created or updated
- The task list ID configured (`CLAUDE_CODE_TASK_LIST_ID=<activity-slug>`)
- Capabilities discovered (MCP servers, CLI tools, project scripts found)
- Whether a new GitHub repo was created (include the URL)
- The current git remote URL

## Cross-Skill Invocation

Other skills invoke this skill via `do-stuff-helper:organize`. The discover skill invokes it as a hard gate for new activities in step 2 of its checklist.
