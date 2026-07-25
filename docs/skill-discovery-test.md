# Skill discovery test

## Scope
Verification of skill staging and discoverability for this repository setup.

## 1) Staging check
- `.github/skills/` exists: yes.
- Found 22 staged skill directories:
  - `copilot-studio-add-adaptive-card`
  - `copilot-studio-add-generative-answers`
  - `copilot-studio-add-global-variable`
  - `copilot-studio-add-node`
  - `copilot-studio-add-other-agents`
  - `copilot-studio-analyze-evals`
  - `copilot-studio-create-eval-set`
  - `copilot-studio-edit-agent`
  - `copilot-studio-edit-triggers`
  - `copilot-studio-int-patterns`
  - `copilot-studio-int-project-context`
  - `copilot-studio-int-reference`
  - `copilot-studio-list-kinds`
  - `copilot-studio-list-topics`
  - `copilot-studio-lookup-schema`
  - `copilot-studio-new-topic`
  - `mcp-apps-generate-mcp-app-ui`
  - `mobile-apps-design-system`
  - `power-pages-add-seo`
  - `power-pages-integrate-backend`
  - `power-pages-integrate-webapi`
  - `power-pages-scan-code`

## 2) Discoverability in available skills
- Key question answer: yes.
- Count visible in my available-skills list: 22 of 22 staged skill names appear.

## 3) One-skill exercise and vendored path resolution
Skill exercised: `copilot-studio-new-topic`.

What was done:
- Read `.github/skills/copilot-studio-new-topic/SKILL.md` (pointer file).
- Followed its pointer to `.github/cloud-skills/_vendor/copilot-studio/skills/new-topic/SKILL.md`.
- Invoked the skill mechanism with `copilot-studio-new-topic` and confirmed it loaded successfully.
- Checked relative path references used by the vendored skill.

Relative path resolution results (from vendored `new-topic/SKILL.md`):
- `../../templates/topics/` -> exists
- `../../scripts/schema-lookup.bundle.js` -> exists
- `../../scripts/manage-agent.bundle.js` -> exists

Broken relative paths found: none.

## 4) Invocation distinction
- Skill mechanism invocation: yes (skill load succeeded).
- Direct file reading: also yes (pointer and vendored `SKILL.md` were read directly for verification).
- This report distinguishes both because they are not the same action.
