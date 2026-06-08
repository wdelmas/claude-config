# claude-config

My personal Claude Code configuration, packaged as a **plugin marketplace** so I (and my team) can add it to any machine straight from this GitHub repo. It bundles the skills I rely on, notes on the tools I use, and a reusable team `CLAUDE.md`.

## Install

```
/plugin marketplace add wdelmas/claude-config
/plugin install will-skills@claude-config
```

The skills then load as `/will-skills:grill-me` and `/will-skills:grill-with-docs`.

## Structure

```
claude-config/
├── CLAUDE.md                 # maintainer guide: how to add a skill/tool
├── .claude-plugin/           # marketplace + plugin manifests
├── skills/                   # the will-skills plugin's skills
├── tools/                    # one note per tool I use
├── claude-md/                # reusable CLAUDE.md templates
└── README.md
```

## Skills

| Skill | What it does | Credit / Source |
|-------|--------------|-----------------|
| [`grill-me`](skills/grill-me) | Interviews you relentlessly about a plan/design until shared understanding, one question at a time | **Matt Pocock** — [mattpocock/skills](https://github.com/mattpocock/skills) |
| [`grill-with-docs`](skills/grill-with-docs) | Same grilling, but challenges the plan against your domain docs and updates `CONTEXT.md` / ADRs inline | **Matt Pocock** — [mattpocock/skills](https://github.com/mattpocock/skills) |
| [`address-pr-comments`](skills/address-pr-comments) | Fetches unresolved PR review comments, fixes them pragmatically, and drafts short French replies + a recap table | wdelmas |
| [`pr-description`](skills/pr-description) | Generates a beautiful, dev-friendly PR description in my house style (context-first, decision tables, mermaid, FAQ) in FR or EN, interviews you for the *why*, then offers to apply it via `gh` | wdelmas |

## Tools

| Tool | What it is | Why I use it | Credit / Source |
|------|-----------|--------------|-----------------|
| [Plannotator](tools/plannotator.md) | Browser UI to annotate plans and review code/markdown | Tightens the plan-review loop — mark up plans and diffs visually instead of a long chat back-and-forth | **backnotprop** — [plannotator.ai](https://plannotator.ai/) · [GitHub](https://github.com/backnotprop/plannotator) |

## MCP servers

MCP servers I connect Claude Code to. Remote connectors are added with `claude mcp add --transport http <name> <url>`; local servers run via `npx`.

| MCP | What it does | Why I use it | Credit / Source |
|-----|--------------|--------------|-----------------|
| Slack | Read/search channels & threads, post messages | Track team threads and post updates without leaving the editor | **Slack** — [mcp.slack.com](https://mcp.slack.com/mcp) |
| Linear | Read/create/update issues | Pull issue context and update tickets while coding | **Linear** — [mcp.linear.app](https://mcp.linear.app/mcp) |
| Datadog | Logs, metrics, traces, dashboards, incidents | Pull observability data when debugging prod issues | **Datadog** — [mcp.datadoghq.eu](https://mcp.datadoghq.eu) |
| Figma | Read designs, specs, measurements | Pull design details straight into implementation | **Figma** — [mcp.figma.com](https://mcp.figma.com/mcp) |
| Postgres | Query a Postgres database | Inspect/query the dev database directly while building | **modelcontextprotocol** — [server-postgres](https://github.com/modelcontextprotocol/servers/tree/main/src/postgres) |
| Playwright | Drive a real browser: navigate, click, screenshot, a11y snapshots | Verify UI flows and scrape pages end-to-end from Claude | **Microsoft** — [playwright-mcp](https://github.com/microsoft/playwright-mcp) |

```bash
# Remote claude.ai connectors
claude mcp add --transport http slack   https://mcp.slack.com/mcp
claude mcp add --transport http linear  https://mcp.linear.app/mcp
claude mcp add --transport http datadog "https://mcp.datadoghq.eu/api/unstable/mcp-server/mcp?toolsets=all,visualizations"
claude mcp add --transport http figma   https://mcp.figma.com/mcp

# Local servers
claude mcp add playwright -- npx -y @playwright/mcp@latest
# Postgres — replace the placeholder connection string (never commit real creds)
claude mcp add postgres -- npx -y @modelcontextprotocol/server-postgres \
  "postgresql://USER:PASSWORD@HOST:5432/DB?sslmode=require"
```

## Conventions

| File | Purpose | Credit / Source |
|------|---------|-----------------|
| [`claude-md/claude-team.md`](claude-md/claude-team.md) | Reusable `CLAUDE.md` of generic TS/React rules for small teams | Adapted from my team's internal rules |
