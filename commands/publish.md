---
description: Bump version, commit, push, and update the installed plugin
argument-hint: "[patch|minor|major]"
allowed-tools: Bash, Read, Edit
---

Publish a new version of the do-stuff-helper plugin.

## Steps

1. **Parse the bump type** from `$ARGUMENTS`. If empty or not one of `patch`, `minor`, `major`, default to `patch`.

2. **Read the current version** from `.claude-plugin/plugin.json`. Extract the `"version"` field (e.g. `"0.5.0"`).

3. **Compute the new version** by splitting the current version into `major.minor.patch` integers and incrementing based on the bump type:
   - `patch`: increment patch, e.g. `0.5.0` -> `0.5.1`
   - `minor`: increment minor and reset patch to 0, e.g. `0.5.0` -> `0.6.0`
   - `major`: increment major and reset minor and patch to 0, e.g. `0.5.0` -> `1.0.0`

4. **Update both version files** using the Edit tool:
   - `.claude-plugin/plugin.json` — update the `"version"` value to the new version
   - `.claude-plugin/marketplace.json` — update the `"version"` value inside the plugin entry to the new version

5. **Stage, commit, and push**:
   ```
   git add -A && git commit -m "chore: bump version to X.Y.Z" && git push
   ```
   Replace `X.Y.Z` with the new version string.

6. **Update the marketplace registry**:
   ```
   claude plugin marketplace update do-stuff-helper
   ```

7. **Update the installed plugin**:
   ```
   claude plugin update do-stuff-helper@do-stuff-helper
   ```

8. **Tell the user**: "Version bumped to X.Y.Z and plugin updated. Start a new conversation for the updated skills to take effect."
