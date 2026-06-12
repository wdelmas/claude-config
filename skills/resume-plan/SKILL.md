---
name: resume-plan
description: Re-trigger the plan approval dialog (Plannotator + options 1/2/3) after a lost or resumed session — recovers the plan, re-enters plan mode, and re-presents it via ExitPlanMode. Use when the user wants to relaunch or re-present an existing plan after /resume, or mentions "resume plan" / "retrigger the plan".
allowed-tools: Bash, Read, Write, AskUserQuestion, ToolSearch, EnterPlanMode, ExitPlanMode
argument-hint: "[optional keyword to find the right plan file]"
---

# Resume Plan

A session died with a plan ready but never approved. The user closed it, relaunched
`claude`, ran `/resume` — and the approval dialog (Plannotator + « 1. Yes, auto-accept
edits / 2. Yes, manually approve edits / 3. No, keep planning ») is gone. Your job:
recover the plan **as it was** and re-present it, so the approval dialog comes back.

Plannotator hooks the **PermissionRequest** of `ExitPlanMode` — calling `ExitPlanMode`
with the plan file restored is all it takes to re-open its browser UI and the 1/2/3
options.

## Arguments

- *(none)* — recover the plan from the resumed conversation, or the most recent one.
- A free-text keyword (e.g. `echeancier`) — find the plan file matching it.

## Instructions

### 1. Recover the plan (in this order)

1. **From the conversation**: if the resumed context already contains the plan — its
   full text, or a plan file path in a "Plan File Info" system reminder — use that.
   Read the file to get the authoritative content.
2. **From an argument**: `ls -t ~/.claude/plans/*.md`, filter by the keyword (filename
   first, then `grep -li` on content), take the most recent match.
3. **Fallback**: list the 3 most recent (`ls -t ~/.claude/plans/*.md | head -3`) and
   read the title/first lines of each. If the most recent clearly matches the session
   context, take it without asking; otherwise confirm with `AskUserQuestion` (one
   option per candidate, title + age as description).

If no plausible plan is found, say so and stop — do not invent or rebuild one.

### 2. Re-enter plan mode

If `EnterPlanMode` / `ExitPlanMode` are deferred, load them first with
`ToolSearch("select:EnterPlanMode,ExitPlanMode")`. Then call `EnterPlanMode` — unless
the session is already in plan mode (a plan-mode system reminder is present), in which
case skip this step.

### 3. Restore the plan file

The plan-mode system message gives the current session's plan file path. If that file
is empty, missing, or differs from the recovered plan, `Write` the recovered content
there **verbatim**. Do not summarize, rephrase, restructure, or "improve" the plan —
the user already iterated on it; byte-for-byte fidelity is the point.

### 4. Re-present

Call `ExitPlanMode` immediately — no re-exploration, no re-planning, no edits. The
Plannotator hook fires on the permission request: browser UI opens, options 1/2/3 are
back, and the user picks up exactly where the dead session left off.
