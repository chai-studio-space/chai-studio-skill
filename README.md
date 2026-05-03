# Chai Studio Skill

Reusable agent skill for:

- verifying Chai Studio MCP + Chrome DevTools MCP availability
- exploring user-provided websites with Chrome DevTools MCP
- extracting design systems into preset-ready structured data
- creating Chai artifacts (`presets_create`, `presets_update`, `rulesets_create`, `rulesets_rules_add`, `applications_create`)

## Requirements

- An agent environment that supports Agent Skills
- Access to a `chai-studio` MCP server
- Access to a Chrome DevTools MCP server (required for website exploration)

## Key Behavior

- MCP availability gate before exploration/creation.
- Pagination-aware discovery for applications/presets/rulesets/rules (20/page default).
- `presets_create` and `presets_update` expect a design-document JSON payload under `preset` (not YAML text). Required: `name`, hex `palette`, `typography.fonts.heading/body`. For full app + linked ruleset creation/sync flows, use `applications_yaml_create` or `applications_yaml_sync`.
- Deterministic font fallback policy with explicit rationale reporting.
- Prompting policy: ask only blocking questions.

## Install from GitHub

```bash
npx skills add chai-studio-space/chai-studio-skill
```

## Verify install

```bash
npx skills list
```

## Command-style usage

- `/chai setup`
- `/chai redesign`
- `/chai canvas`
- `/chai fix-canvas`
- `/chai audit`
- `/chai extract`
- `/chai shadcn`
- `/chai rulesets`
- `/chai journal`

Legacy alias: `/chai website` maps to `/chai extract`.

## Agent Compatibility

- `/chai ...` command routing is driven by `SKILL.md` and `references/workflows.md`, so it works across agents that support Agent Skills.
- Required for skill execution: `SKILL.md`.
- `agents/openai.yaml` is optional and OpenAI-specific UI metadata. It is not required for command routing.

## Typical usage prompts

- "Use $chai-studio-skill to explore https://example.com and create a preset from its design system."
- "Use $chai-studio-skill to create a standalone ruleset and attach existing rules."
- "Use $chai-studio-skill to create an application from preset and ruleset IDs."
- "Use $chai-studio-skill to scaffold themed shadcn components for my app using shadcn_component_get."

## Directory Layout

```text
.
├── SKILL.md
├── agents/
│   └── openai.yaml
└── references/
    ├── config.md
    ├── workflows.md
    ├── usage.md
    └── canvas-design-html-guide.md
```

## Canvas Preview

Canvas pages are authored in `componentTsx` and rendered with the built-in `UI.*` runtime plus preset-mapped shared CSS.

## License

MIT. See `LICENSE`.
