---
name: session-handoff
description: Compose a self-contained handoff brief of the current session and copy it to the clipboard, so another AI session (Claude, ChatGPT, …) can pick up the work. Use when the user wants to hand off, transfer, or sum up the session for another agent.
allowed-tools: Bash, Read, Write
argument-hint: "[optional focus, e.g. 'just the auth refactor']"
---

# Session Handoff

Compose a **self-contained handoff brief** of the current session and copy it to the
clipboard, so the user can paste it into another AI session — a fresh Claude Code
session, ChatGPT, anything — and continue the work without losing context. The reader
has **zero shared context**: no access to this conversation, this repo's state in your
head, or any tool output you've seen.

## Arguments

- *(none)* — hand off the whole session.
- A free-text focus (e.g. `just the auth refactor`) — scope the brief to that thread of
  work only; omit unrelated tangents.

## Instructions

### 1. Gather fresh repo state

The conversation may be stale — re-check reality before writing:

```bash
pwd
git branch --show-current
git status --short
git diff --stat
git log --oneline -5
```

### 2. Compose the brief

Plain markdown, with these sections (skip a section only if genuinely empty):

- **Mission** — what the user is trying to achieve overall, in 1–2 sentences.
- **State of play** — what has been done so far; what currently works / is verified.
- **Key decisions & why** — choices made during the session with their rationale, so
  the next AI doesn't relitigate them.
- **Files touched** — full paths, one-line note each.
- **Open items / next steps** — concrete and ordered; the first one must be actionable
  immediately.
- **Gotchas & constraints** — anything learned the hard way: failing commands, env
  quirks, user preferences stated during the session.
- **Environment** — repo path, branch, uncommitted-changes summary, the commands to
  build/test/run.

### 3. Writing rules

- **Model-agnostic**: no references to Claude-specific tools, this conversation, or
  "as discussed above". Write for a stranger.
- **Self-contained**: every file by full path; every decision restated, not referenced.
- **Honest about state**: distinguish *verified working* from *written but untested*.
- Don't pad. A short accurate brief beats a long vague one.

### 4. Deliver

1. Write the brief to `/tmp/session-handoff.md` (avoids shell-escaping issues with
   heredocs).
2. Copy it: `pbcopy < /tmp/session-handoff.md`.
3. Confirm: `pbpaste | head -1` matches the first line.
4. Print the **full brief inline** in your response so the user can read and review it,
   followed by a one-line confirmation that it's on the clipboard.
