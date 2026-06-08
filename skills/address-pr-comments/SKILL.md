---
name: address-pr-comments
description: List unresolved PR comments and fix them with a recap
allowed-tools: Bash, Read, Edit, Write, Grep, Glob, Agent
---

# Fix Unresolved PR Comments

Given a PR URL or number, fetch all unresolved review comments, fix them pragmatically, and generate short French replies.

## Arguments

`$ARGUMENTS` — A GitHub PR URL (e.g. `https://github.com/owner/repo/pull/123`) or just a PR number. If only a number is given, use the current git remote.

## Instructions

### Step 0: Parse the PR URL

Extract `owner`, `repo`, and `pr_number` from the URL. If only a number is given, detect owner/repo from `gh repo view --json owner,name`.

### Step 1: Fetch unresolved comments

Run this GraphQL query via `gh api graphql` to get all unresolved review threads AND top-level PR comments:

```bash
gh api graphql -f query='
query($owner: String!, $repo: String!, $pr: Int!) {
  repository(owner: $owner, name: $repo) {
    pullRequest(number: $pr) {
      comments(first: 100) {
        nodes {
          author { login }
          body
          createdAt
        }
      }
      reviewThreads(first: 100) {
        nodes {
          isResolved
          comments(first: 10) {
            nodes {
              author { login }
              body
              path
              line
            }
          }
        }
      }
    }
  }
}' -f owner=OWNER -f repo=REPO -F pr=PR_NUMBER
```

Extract:
- **Review threads:** only unresolved threads (inline code comments)
- **PR comments:** top-level conversation comments that contain actionable feedback (requests, questions, suggestions)

Filter out bot comments that are purely informational (e.g. CI status bots) unless they contain actionable bug reports. Also filter out simple acknowledgment comments ("ok", "thanks", etc.).

### Step 2: List all unresolved comments

Display a numbered list with: file path, line number, author, and comment body (truncated if very long).

### Step 3: Analyze and fix each comment

For each unresolved comment:

1. **Read the referenced file** at the specified line to understand the context
2. **Classify the comment** as one of:
   - **Code fix** — the reviewer wants a code change (refactor, bug fix, style change)
   - **Question** — the reviewer is asking for clarification (needs a PR reply, possibly a code comment)
   - **Skippable** — the comment is already addressed, outdated, or not actionable
3. **Apply the fix** if it's a code change. Be pragmatic: search for existing patterns in the codebase and make the simplest change that addresses the comment. No over-engineering.
4. **Draft a PR reply** for each comment — in French, very short (1 sentence max). Just state what was done or answer the question directly. No fluff.

### Step 4: Recap

At the end, output a summary table:

```
| # | File | Commentaire | Action | Statut |
|---|------|-------------|--------|--------|
| 1 | path/to/file.tsx:42 | Résumé court | Ce qui a été fait | Corrigé / Répondu / Ignoré |
```

Then list each draft reply so the user can copy-paste them:

```
**1. path/to/file.tsx:42** (réponse à @author)
> Texte de la réponse en français
```

## Rules

- All PR replies in French, bref (1 phrase max)
- Be pragmatic: fix what makes sense, skip what doesn't
- Follow all project conventions from CLAUDE.md
- Search for existing patterns before introducing new code
- Do not auto-commit — show the diff to the user for review
- If a comment requires a large refactor or is ambiguous, flag it and ask the user rather than guessing
- Run type checking after all fixes to verify nothing is broken
