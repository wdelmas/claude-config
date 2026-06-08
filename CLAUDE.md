# claude-config — maintainer guide

This repo is a Claude Code marketplace (`claude-config`) bundling the `will-skills` plugin. When asked to extend it:

- **Add a skill** → create `skills/<name>/SKILL.md` (+ any companion files), then add a row to the **Skills** table in `README.md`. Skills auto-load via the `will-skills` plugin — no manifest edit needed.
- **Add a tool** → create `tools/<name>.md`, then add a row to the **Tools** table in `README.md`.
- **Add a plugin** (external, from another marketplace) → add a row to the **Plugins** table in `README.md`, crediting the author with a link.
- **Credit is mandatory** when a skill/tool isn't Will's own work — fill the Credit / Source column with the author + a link.
- Bump `version` in `.claude-plugin/plugin.json` when cutting a pinned release.

Note: `claude-md/claude-team.md` is a separate, reusable team template — not rules for this repo.
