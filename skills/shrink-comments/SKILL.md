---
name: shrink-comments
description: Remove comments that aren't strictly necessary from a change set (pending changes, a PR, or the current branch vs develop), following the quality comment convention. Use when the user wants to strip redundant/obvious comments from their changes.
allowed-tools: Bash, Read, Edit, Grep, Glob, Skill
argument-hint: "[pending | <base-branch> | <PR url or number>]"
---

# Shrink Comments

Remove comments that aren't *strictly necessary* from a **change set** — pending
working-tree changes, a PR, or the current branch diffed against `develop`. The bar comes
from the [`quality`](../quality/SKILL.md) skill: default to **none**; keep a comment only
when the *why* is non-obvious (hidden constraint, invariant, workaround); never narrate
*what* the code does.

**Scope is only the changed lines** — prune only comments introduced or touched by the diff
(the `+` side of hunks). Comments on lines the change didn't touch are left alone, so the
cleanup stays a minimal, reviewable diff.

## Arguments

`$ARGUMENTS` — optional:
- *empty* → **pending** changes if the working tree is dirty, else **branch vs `develop`**.
- a branch name (e.g. `develop`, `main`) → diff the current branch against it.
- a PR URL or bare number → scope to that PR's diff.

## Instructions

### Step 1 — Resolve scope (mode + changed lines)

- **PR** — arg is a PR URL or bare number. Get files with `gh pr diff <n> --name-only` and
  the patch with `gh pr diff <n>` (derive owner/repo via `gh repo view --json owner,name`
  when only a number is given). The PR branch must be checked out locally — we edit local files.
- **Branch-vs-base** — arg is a branch name, OR empty with a clean working tree. Diff against
  the base: `git diff --merge-base <base>`. **Base defaults to `develop`** when it exists
  (`git rev-parse --verify develop`), else `main`/`master`.
- **Pending** (default) — empty arg with uncommitted changes. Use `git diff` (staged +
  unstaged) plus untracked files from `git status --porcelain` (`R old -> new` → keep `new`).

From the chosen diff, record per file the **added/changed hunks**. Only act on comments
sitting on the `+` side of those hunks.

### Step 2 — Filter to code files

Drop `.md`, `.json`, `.yaml`/`.yml`, lock files, images, and generated/vendored paths —
comments only matter in code. If nothing's left, say so and stop.

### Step 3 — Load the bar

Invoke the [`quality`](../quality/SKILL.md) skill (Skill tool) to pull the Comments
convention into context. If it isn't installed, use the rubric below.

### Step 4 — Classify & prune (inline, per file)

Read each changed file, locate comments on its added/changed lines, and classify each.

**Remove — not strictly necessary:**
- Narrates *what* the code does, or restates the next line / a self-evident identifier.
- Commented-out code blocks (dead code).
- Decorative banners, dividers, or section headers when the structure is already clear.
- Docstrings that only restate the signature; changelog / "added by" / dated notes (git's
  job); leftover scaffolding (`// TODO: implement` that's now implemented); debug notes.

**Keep — strictly necessary (bias to keep when unsure):**
- *Why* comments: hidden constraint, non-obvious invariant, workaround + reason, "must stay
  in sync with X", ordering/perf rationale, links to issues/specs.
- **Functional directives — NEVER remove:** `eslint-disable*`, `// @ts-expect-error`,
  `// @ts-ignore`, `# type: ignore`, `# noqa`, `# pylint:`, `# nosec`, `// nolint`,
  `#pragma`, shebangs (`#!`), encoding declarations (`# -*- coding`).
- Legal / license / copyright headers.
- Public-API doc comments that add real info beyond the signature (param semantics, return
  contract, examples) — keep; only trim pure restatement.
- `TODO` / `FIXME` / `HACK` carrying actionable info or a ticket reference — keep; don't
  silently drop tracked work.

**Safety:**
- Never touch text inside string literals that resembles a comment.
- Preserve indentation and syntax when removing a trailing comment.
- When a comment is the only thing explaining a magic value, keep it or flag it — don't remove.

Apply removals with Edit. Leave the changes in the working tree — **do not commit**.

### Step 5 — Recap

Output a table: file | comment removed (truncated) | reason. Add a total count and a short
"kept but borderline — review?" list for ambiguous cases. Suggest running the project's
type-check / lint as a sanity check that no protected directive was touched.

## Rules

- Scope is **only the changed lines** — never touch comments outside the diff hunks.
- Bias toward keeping when a comment's *why* is ambiguous; losing real context is worse than
  leaving one borderline comment.
- No auto-commit; show the diff for review.
- Follow `CLAUDE.md` and project conventions.
