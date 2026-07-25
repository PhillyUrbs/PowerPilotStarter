# PowerPilotStarter

## Copilot Cloud Agent Setup

This repository includes `.github/workflows/copilot-setup-steps.yml` to prepare GitHub Copilot cloud agent runs with Azure CLI, .NET SDK (LTS), and Power Platform CLI (`pac`) before coding starts.

The setup workflow file must exist on the repository default branch for Copilot cloud agent to pick it up.

Set required secrets and variables in the repository **Environment** named `copilot` (Settings → Environments → `copilot` → Environment secrets / Environment variables).