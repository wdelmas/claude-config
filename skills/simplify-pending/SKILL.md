---
name: simplify-pending
description: Run the code-simplifier agent on files with pending changes in git status. Use when the user wants to simplify / clean up their uncommitted work.
allowed-tools: Bash, Task, Glob, Grep, Read, Edit, Write, Skill
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
3. Load the project conventions: invoke the [`quality`](../quality/SKILL.md) skill (Skill tool) to pull
   the non-negotiable rules into context. If the skill isn't installed, skip this step.
4. Run the **code-simplifier** agent on those files via the Task tool
   (`subagent_type: code-simplifier:code-simplifier`), batching the files into one run. Pass the relevant
   `quality` non-negotiables into the agent's prompt so simplifications conform (strict typing with no
   escape hatches, exhaustive dispatch over open-ended branching, validate at the boundary, small
   files/params/nesting, no dead code or debug prints, comments only when the *why* is non-obvious).
5. Recap which files changed; leave edits in the working tree for review (don't commit).

Focus on:

- Clarity and consistency
- Removing unnecessary complexity
- Conforming to the `quality` conventions (above)
- Preserving all functionality — never change behavior, only how the code reads
