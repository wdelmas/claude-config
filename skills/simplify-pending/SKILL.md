---
name: simplify-pending
description: Run the code-simplifier agent on files with pending changes in git status. Use when the user wants to simplify / clean up their uncommitted work.
allowed-tools: Bash, Task, Glob, Grep, Read, Edit, Write
---

# Simplify Recently Modified Code

Run [`code-simplifier@claude-plugins-official`](https://github.com/anthropics/claude-plugins-official/tree/main/plugins/code-simplifier)
on all files that appear in `git status`. Requires the plugin installed
(`/plugin install code-simplifier@claude-plugins-official`).

## Instructions

1. Get the list of modified/staged/untracked files from `git status --porcelain`
   (for renames `R old -> new`, keep the new path).
2. Filter to only code files (exclude `.md`, `.json`, `.yaml`/`.yml`, lock files, images, generated/vendored).
   If nothing's left, say so and stop.
3. Run the **code-simplifier** agent on those files via the Task tool
   (`subagent_type: code-simplifier:code-simplifier`), batching the files into one run.
4. Recap which files changed; leave edits in the working tree for review (don't commit).

Focus on:

- Clarity and consistency
- Removing unnecessary complexity
- Preserving all functionality — never change behavior, only how the code reads

> [!NOTE]
> Project quality-checklist enforcement will be added here later; for now this skill just simplifies the
> pending changes.
