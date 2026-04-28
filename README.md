# Chai Studio Skill

Reusable agent skill for Chai Studio MCP workflows:

- connect a repo to the correct Chai Studio application
- sync local design docs (`DESIGN.md`, `DESIGN-RULES.md`)
- run audit reconciliation
- upload fresh audit runs with actionable violations

## Requirements

- An agent environment that supports Agent Skills
- Access to a `chai_studio` MCP server
- A project where you want Chai Studio-backed design docs and audits

## Install from GitHub

Install all skills from a repo:

```bash
npx skills add chai-studio-space/chai-studio-skill
```

Install globally (available across projects):

```bash
npx skills add -g chai-studio-space/chai-studio-skill
```

Install a specific skill only:

```bash
npx skills add chai-studio-space/chai-studio-skill --skill chai-studio-skill
```

List skills in the repo before installing:

```bash
npx skills add chai-studio-space/chai-studio-skill --list
```

## Install from local folder

```bash
npx skills add ./chai-studio-skill
```

Global local install:

```bash
npx skills add -g ./chai-studio-skill
```

## Verify install

```bash
npx skills list
```

You should see `chai-studio-skill` in the installed skills list.

## Skill Included

- `chai-studio-skill` (folder root)

## Directory Layout

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── config.md
    └── workflows.md
```

## What This Skill Expects

- Access to a `chai_studio` MCP server
- A project-level `chai-studio.json` configuration file (created during setup if missing)

## Typical usage

Ask your agent to invoke the skill directly:

```text
Use $chai-studio-skill to configure this project with Chai Studio and sync local design rules.
```

Common requests:

- "Set up `chai-studio.json` for this repo."
- "Sync `DESIGN.md` and `DESIGN-RULES.md` from Chai Studio."
- "Reconcile old audit violations and upload a fresh audit run."

## Safety Notes

- Do not upload dummy audits unless explicitly requested.
- Verify prior violations and resolve fixed items before uploading a new run.
- Treat all skill scripts/references as code and review before use.

## License

MIT. See `LICENSE`.
