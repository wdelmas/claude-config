---
name: pr-description
description: Generate a beautiful, dev-friendly PR description in Will's house style (context-first, decision tables, mermaid, FAQ collapsibles), in French or English. Use when the user wants to write or rewrite a pull request description.
allowed-tools: Bash, Read, Grep, Glob, Agent
argument-hint: "[FR|EN] [big|small] [PR URL or number]"
---

# Generate a PR Description

Produce a rich, reviewer-friendly pull request description in the house style documented in
[`FORMAT.md`](FORMAT.md): a conventional-commit title, a context-first opening (the *why* before the
*what*), emoji section headers, decision tables that name rejected alternatives, mermaid/ASCII diagrams,
`<details>` collapsibles, a reviewer-focus section, and a test-plan checklist.

**Read [`FORMAT.md`](FORMAT.md) first** — it holds the section catalog, templates, and condensed examples
this skill assembles from.

## Arguments

`$ARGUMENTS` (optional) may carry, in any order:
- a **language** — `FR` or `EN` (case-insensitive)
- a **PR type** — `big` or `small` (synonyms accepted: `doc`/`structural` → big, `known`/`quick` → small)
- a **PR reference** — a GitHub PR URL (`https://github.com/owner/repo/pull/123`) or a bare number

Parse all three out of whatever is given. Any of them may be absent.

## Instructions

### Step 0: Language & PR type

Take `FR`/`EN` and `big`/`small` from `$ARGUMENTS`. **Ask the user for whichever is missing** (one
AskUserQuestion batch covering both). The final description **and every question you ask** must be in the
chosen language.

The **PR type** is about the description's *documentation value*, not the diff size:

- **big** — the PR is documentation for the team: structural/foundational changes, new concepts, anything
  the team doesn't already know. Structural PRs get the **same full house format** as feature PRs.
- **small** — routine change the team already knows about; a compact description is enough.

### Step 1: Locate the change (auto-detect)

Resolve, in this order:

1. **Explicit PR ref in `$ARGUMENTS`** → use it:
   `gh pr view <ref> --json number,url,baseRefName,headRefName,title,body`
2. **No ref, but the current branch has an open PR** → `gh pr view --json number,url,baseRefName,headRefName,title,body`
   (no ref defaults to the current branch).
3. **No PR at all** → resolve the **base branch**: prefer `develop` if it exists
   (`git rev-parse --verify origin/develop`), otherwise the repo default
   (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`, fallback
   `git symbolic-ref refs/remotes/origin/HEAD`).

Record the **base branch** and any **existing title/body** — you will *improve* an existing description,
not throw it away.

### Step 2: Gather raw material

- **Commits** (group by theme for an optional history section):
  `git log --reverse <base>..HEAD --pretty='%h %s'`
- **Scope line** (file count + insertions/deletions): `git diff --stat <base>...HEAD` (three-dot).
- **Changes**: start with `git diff --name-status <base>...HEAD`, then read the key files/hunks with
  Read/Grep. If the diff is large, dispatch an **Explore (or general-purpose) Agent** to summarize it into
  per-area bullets instead of loading the whole thing into context.
- **Linear tickets**: extract IDs (`BRI-\d+`, `MAR-\d+`, …) from the branch name, commit messages, and any
  existing body. If a **Linear MCP tool** is connected (find it with ToolSearch `linear`), fetch the
  ticket titles/summaries for richer context; otherwise just keep the IDs to link them.

### Step 3: Draft internally — do NOT show yet

Build the candidate sections from the raw material. **The PR type from Step 0 picks the tier** (see the
size tiers in `FORMAT.md`); the diff's size/complexity only modulates within it:

- **small** → Small tier: title · 🎯 Contexte · what changed (bullets) · ✅ test plan · 🔗 links.
- **big** → Medium or Large tier depending on scope: 🏗️ architecture overview · decision table(s) ·
  reviewer notes, up to the full treatment — mermaid diagram(s), `<details>` vertical-slices + ❓ FAQ,
  🛣️ phasing table, 🧯 gotchas, ⚠️ action-required, screenshot placeholders. Structural PRs are **not**
  an excuse to skimp: they get the same format as feature PRs.

Note the gaps the diff **cannot** fill (motivation, decisions, validation) — those drive the next step.

### Step 4: Interview for the *why* (one tight batch)

Use **AskUserQuestion** (in the chosen language) to fill *only* the gaps from Step 3. Candidate topics —
skip any the raw material already answered:

- **Motivation / problem** being solved (if not clear from commits/Linear)
- **Key decisions + rejected alternatives** — the *"why X, not Y"* the diff can't show
- **Reviewer focus / gotchas** — what to look at first, fragile spots
- **Stakeholder validation** — who signed off, on what (for a `> [!NOTE]` callout)
- **Required actions** — env vars, secrets, migrations, manual steps (→ ⚠️ section)
- **QA / preview URL** if there's a deploy to test against

Keep it to one batch; don't interrogate beyond what the description needs.

### Step 5: Produce the final description

Assemble the description per `FORMAT.md`, in the chosen language, with a **conventional-commit title**
(derive the scope from the touched package/app). **Print the full markdown inside a fenced ```` ```md ````
block** so the user can copy-paste it verbatim.

### Step 6: Offer to apply (only after explicit confirmation)

Ask whether to apply it. Only if the user confirms:

- **Existing PR** → write the body to a temp file and
  `gh pr edit <number> --body-file <tmp>` (optionally `--title "<title>"`).
- **No PR yet** → offer
  `gh pr create --base <base> --title "<title>" --body-file <tmp>`; confirm once more before creating.

Never apply without an explicit yes.

## Rules

- **Ask for missing arguments**: language and PR type are never guessed — when `$ARGUMENTS` doesn't
  carry them, ask (one batch). The whole description and every question are in the chosen language.
- **Context-first, always**: lead with *why* before *what*; surface rejected alternatives wherever a
  non-obvious choice was made.
- **Type drives richness**: `big` (doc-worthy — structural or new to the team) gets the full house
  format, structural PRs included; `small` (already known by the team) stays compact. Within a tier,
  never force a mermaid diagram or FAQ where the content doesn't earn it.
- **Use the GFM features the style relies on**: `> [!NOTE]` / `> [!IMPORTANT]` callouts, `<details>`
  collapsibles, ` ```mermaid ` fences, and tables.
- **Real code only**: code snippets must be lifted from the actual diff, never invented.
- **Don't fabricate the *why***: if the user can't supply the context for a section, omit that section
  rather than guess.
- **Safe `gh` usage**: always `--body-file <tmp>` (never inline `--body` with large content), and never
  auto-apply — show the markdown and wait for confirmation.
- **Title** = conventional commit (`feat(scope): …`, `fix(scope): …`), scope from the touched package/app.
