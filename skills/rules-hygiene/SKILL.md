---
name: rules-hygiene
description: Audit .cursor/rules/**/*.mdc for length, duplication, and stale claims
allowed-tools: Bash, Glob, Grep, Read
---

# Rules hygiene

Audit `.cursor/rules/` for drift against best practice — **read-only, no edits**. Produces a punch-list of file:line → suggested fix that the user reviews.

Scope:
- `/rules-hygiene` → whole `.cursor/rules/` tree.
- `/rules-hygiene bo` → just `.cursor/rules/bo/`.
- `/rules-hygiene <path>` → that subfolder.

## Three-pass audit

### 1. Length budget

```bash
wc -l <scope>/**/*.mdc
```

- Flag any file > 200 lines (Anthropic threshold for instruction adherence).
- Flag total > 800 lines for a subfolder (heuristic — past that, signal/noise drops).
- For each over-budget file, identify the longest section (`grep -n '^## ' <file>` + line deltas) and propose what to cut.

### 2. Duplication scan

- Grep for "Canonical example:" / "❌" / "Hard rules" across files in scope — list files that share the same bullet stems.
- For each topical rule (`form.mdc`, `table.mdc`, …) compare its `Hard rules` and `What NOT to do` sections against `claude-root.mdc` and the sub-app rule file. Any bullet whose substance appears in both is a dup — name canonical home + suggest the deletion.
- Use a 6+ word phrase match: `grep -rn "<phrase>" <scope>` for sample phrases ("Use `cva` for variants", "No relative imports", "Skeleton, never spinner", etc.).

### 3. Stale check

For each cited symbol / path / version in the rule, verify it still exists:

- **Versions:** grep `"vite"`, `"@tanstack/react-router"`, `"react"`, etc. in rule files; cross-check against the relevant `package.json`.
- **Helper / hook / provider names:** grep rule for `\b[A-Z][A-Za-z]+(Provider|Modal|Cell|Form)\b` or `^use[A-Z]`, then `grep -rn '<name>' projects/` to verify exports exist.
- **File paths cited in fenced code or backticks:** `ls <path>` or `Glob` to confirm presence.
- **Module names + routes in module-map tables:** cross-check against `src/routes/` and `src/modules/`.

Report each stale claim with the rule's line number, the cited symbol/path, and what it should be (or that the line should be dropped).

## Output

A concise punch-list grouped by file:

```
.cursor/rules/bo/claude-bo-projects.mdc
  - L76 STALE: "Vite 8" → actual 7.2.2 (package.json)
  - L144-147 STALE: "Not built yet" list — capitalRepayments, spvsWallets, pendingPayouts are built
  - L36 DUP of claude-root.mdc:20 (strict typing) — delete

.cursor/rules/bo/form.mdc
  - 211 lines, over 200 — collapse "Submit handler" block
  - L78 DUP of @bricks-common-bo.mdc:34 (peer-dep wording)
```

Don't auto-edit. The user reviews and applies the fixes.

## Self-check before running

Skip the audit if `.cursor/rules/` is empty for the chosen scope. Otherwise: glob the scope, run the three passes in order, return the punch-list.
