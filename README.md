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

## Tools

| Tool | What it is | Why I use it | Credit / Source |
|------|-----------|--------------|-----------------|
| [Plannotator](tools/plannotator.md) | Browser UI to annotate plans and review code/markdown | Tightens the plan-review loop — mark up plans and diffs visually instead of a long chat back-and-forth | **backnotprop** — [plannotator.ai](https://plannotator.ai/) · [GitHub](https://github.com/backnotprop/plannotator) |

## Conventions

| File | Purpose | Credit / Source |
|------|---------|-----------------|
| [`claude-md/claude-team.md`](claude-md/claude-team.md) | Reusable `CLAUDE.md` of generic TS/React rules for small teams | Adapted from my team's internal rules |
