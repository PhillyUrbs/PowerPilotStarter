# AGENTS.md

Guidance for any coding agent working in this repository, regardless of editor or CLI.

This repo targets Azure and the Power Platform. Tooling assumptions: Azure CLI, .NET SDK (LTS), and the Power Platform CLI (`pac`).

## Agent skills

Two sets exist, and which one you get depends on where you are running.

**Running locally (VS Code, Claude Code, Copilot CLI, any interactive editor):** use the full upstream plugins. `.claude/settings.json` declares both marketplaces (`microsoft/power-platform-skills`, `microsoft/skills-for-copilot-studio`) and enables every plugin. VS Code surfaces them as workspace recommendations once the workspace is trusted; Claude Code reads the file natively. Prefer these - they are complete and current.

**Running as the GitHub Copilot cloud agent:** you only get the vendored subset in `.github/skills/`. Each `<source>-<skill>/SKILL.md` is a short pointer; the full instructions are in the vendored upstream copy it names, under `.github/cloud-skills/_vendor/`. This subset is deliberately limited to skills that need no interactive sign-in, no browser, no simulator, and no plugin-bundled MCP server.

If a task needs a skill that is not in `.github/skills/`, it was excluded on purpose. Say so rather than improvising a substitute, and point the user at running it locally.

Three notes for any agent editing this repo:

- `.github/skills/` is generated at session start by `copilot-setup-steps.yml`, copied from `.github/cloud-skills/`, and is gitignored. Never commit it, and never edit it - edit nothing here at all, since `.github/cloud-skills/` is itself generated. See "Vendored agent skills" in the README for how to change what is vendored.
- Do not move the committed copy into `.github/skills/`. It is staged at runtime precisely so that local editors, which scan `.github/skills/`, do not list it next to the full upstream plugins.
- Do not duplicate the vendored skills into `.claude/skills` or `.agents/skills`. GitHub Copilot reads all three locations, so mirroring makes every skill appear multiple times in the picker.

## MCP servers

The Microsoft Learn MCP server (`https://learn.microsoft.com/api/mcp`, HTTP, unauthenticated) should be available for grounded Azure and Power Platform documentation lookups.

There is no shared MCP config format, so the same server is declared once per client:

| File | Read by |
| --- | --- |
| `.vscode/mcp.json` | VS Code |
| `.mcp.json` | Claude Code and other clients using the project-root convention |
| `.github/copilot-mcp-config.json` | Reference copy to paste into GitHub repository settings for the cloud agent |

If you add or change a server, update all three.

## Authentication

Never prompt for interactive sign-in or a device code. This repo is set up for non-interactive auth only:

- Azure uses federated (OIDC) sign-in via the `copilot` environment.
- `pac` uses a service principal (`pac auth create --applicationId ... --clientSecret ...`).

`.github/copilot-env.example` lists every required variable and secret. Values live on the repository environment named `copilot`, never in the repo.

If a task needs credentials that are not configured, say so and stop. Do not fall back to an interactive flow.

## Conventions

- No em dashes, en dashes, curly quotes, ellipsis characters, or decorative arrows in any file. Use ASCII equivalents.
- Workflow changes must keep `copilot-setup-steps.yml` passing when no secrets are configured. Guard new steps the same way the existing ones are guarded.
