# PowerPilotStarter

## Copilot Cloud Agent Setup

This repository includes `.github/workflows/copilot-setup-steps.yml` to prepare GitHub Copilot cloud agent runs with Azure CLI, .NET SDK (LTS), and Power Platform CLI (`pac`) before coding starts.

The setup workflow file must exist on the repository default branch for Copilot cloud agent to pick it up.

Set required secrets and variables in the repository **Environment** named `copilot` (Settings -> Environments -> `copilot` -> Environment secrets / Environment variables).

## MCP servers

`.github/copilot-mcp-config.json` is the source of truth for this repo's MCP configuration. GitHub does not read it automatically - it has to be pasted into the repository settings:

1. Settings -> Copilot -> MCP servers.
2. Paste the contents of `.github/copilot-mcp-config.json` into the "MCP configuration" box.
3. Click **Save MCP configuration**.

Currently configured:

- **microsoft-learn** - remote HTTP server at `https://learn.microsoft.com/api/mcp`, giving the agent grounded access to Microsoft Learn docs and code samples for Azure and Power Platform. No secrets or setup steps required.

The GitHub and Playwright MCP servers are enabled by default and do not need to be listed here.

### In VS Code

`.vscode/mcp.json` configures the same Learn MCP server for anyone opening this repo in VS Code. It is picked up automatically - VS Code prompts to start the server the first time agent mode runs in the workspace.

### Other editors

There is no shared MCP config format, so the same server is declared once per client:

| File | Read by |
| --- | --- |
| `.vscode/mcp.json` | VS Code |
| `.mcp.json` | Claude Code and other clients using the project-root convention |
| `.github/copilot-mcp-config.json` | Reference copy to paste into GitHub repository settings |

Adding a server means editing all three. `AGENTS.md` says the same thing for agents that end up doing it.

## Secrets and variables

`.github/copilot-env.example` is the checklist of everything the agent needs for non-interactive auth. It holds names only - set the real values on the repository environment named `copilot`.

| Name                                                 | Kind     | Needed for                                |
| ---------------------------------------------------- | -------- | ----------------------------------------- |
| `AZURE_CLIENT_ID`                                    | Variable | Federated (OIDC) sign-in to Azure         |
| `AZURE_TENANT_ID`                                    | Variable | Federated (OIDC) sign-in to Azure         |
| `AZURE_SUBSCRIPTION_ID`                              | Variable | Only when a skill targets Azure resources |
| `PP_APP_ID`                                          | Variable | `pac` service principal auth              |
| `PP_TENANT_ID`                                       | Variable | `pac` service principal auth              |
| `PP_ENVIRONMENT_URL`                                 | Variable | `pac` service principal auth              |
| `POWER_PLATFORM_SKILLS_TELEMETRY_POWER_PAGES_OPTOUT` | Variable | Suppresses upstream plugin telemetry      |
| `PP_CLIENT_SECRET`                                   | Secret   | `pac` service principal auth              |

`copilot-setup-steps.yml` skips the Azure login and `pac auth create` steps when these are unset, so the baseline setup still succeeds on a repo that has not configured a service principal.

Azure uses OIDC and needs no stored secret. `pac auth create` has no federated credential flow, so `PP_CLIENT_SECRET` is unavoidable if you want tier B skills to work.

## Agent skills

Skills come from [microsoft/power-platform-skills](https://github.com/microsoft/power-platform-skills) and [microsoft/skills-for-copilot-studio](https://github.com/microsoft/skills-for-copilot-studio). Those repos are agent plugins, not MCP servers, so they cannot go in the MCP configuration.

The two runtimes get different sets, because they can do different things:

| | Cloud agent | Editors and CLI |
| --- | --- | --- |
| Gets | 22 vendored cloud-safe skills | Both upstream plugins, in full |
| Source | `.github/cloud-skills/`, pinned to a reviewed commit | Installed from GitHub at their current `main` |
| Configured by | `.github/skills-sources.json` | `.claude/settings.json` |
| Why limited | No interactive sign-in, no browser, no simulator, no plugin-bundled MCP server | None of those constraints apply |

### Editors and CLI: the full suite

`.claude/settings.json` names both upstream marketplaces and enables every plugin in them. VS Code reads this file and offers the plugins as workspace recommendations once you trust the workspace; Claude Code reads it natively. Copilot CLI users can add the same two marketplaces with `/plugin marketplace add microsoft/power-platform-skills`.

This is the set to use when you are working interactively. It includes the skills that sign in, call live Dataverse, drive a browser, or start a local MCP server - all the things the cloud agent cannot do.

### Cloud agent: the vendored subset

See "Vendored agent skills" below.

Copilot only discovers skills in `.github/skills/`, but every local editor scans that same path, so committing the subset there would list all 22 alongside the full plugin set. There is no setting to exclude a skills directory - only `chat.skillTool.enabled`, which is all or nothing.

So the committed copy lives in `.github/cloud-skills/`, which nothing scans, and `copilot-setup-steps.yml` stages it into `.github/skills/` before the agent starts. `.gitignore` keeps the staged copy out of commits. Locally you get the plugins and nothing else.

## Vendored agent skills

Layout:

- `.github/skills-sources.json` - the manifest. Which skills are vendored, what tier each one is, and the upstream commit last synced.
- `.github/cloud-skills/_vendor/<source>/` - verbatim upstream subtrees with the plugin layout preserved, so each skill's own `../../scripts` style paths still resolve.
- `.github/cloud-skills/<source>-<skill>/SKILL.md` - generated pointer that delegates to the vendored copy. Do not edit these.
- `.github/skills/` - generated at session start, gitignored. Pointer directories only; they reference `_vendor` at its committed path, so it is not copied.

### Tiers

Every skill in both upstream repos is classified in the manifest. The tiers describe what the *cloud agent* can run - they say nothing about local use, where everything is available via the plugins.

- **A** - cloud-safe. Local file authoring or analysis only. These are the ones currently enabled.
- **B** - needs non-interactive auth. Flip `enabled` to `true` once the service principal above is configured.
- **C** - not cloud-viable. Interactive browser sign-in, device code flow, a plugin-bundled MCP server, the VS Code extension's LSP binary, a simulator, or a live published endpoint. These stay disabled permanently; use them locally instead.

Some paths are excluded from the vendored copy because only tier B or C skills reach them. If you enable one of those skills, remove the matching entry from `excludePaths`.

### Staying current

`.github/workflows/sync-skills.yml` runs weekly and on demand. It re-reads the manifest, re-copies from upstream, and opens a pull request only when the result differs from what is committed. Review that diff before merging - these skills execute shell commands during agent runs.

To change the set of vendored skills, edit `enabled` in the manifest and run:

```sh
node .github/scripts/sync-skills.mjs
```

Committing a manifest edit without running the script fails the workflow's push check.
