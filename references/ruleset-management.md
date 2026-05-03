# Ruleset Management Workflow

Use this workflow when the user asks to create, extend, or inspect audit rulesets.

**Prerequisite:** For local codebase work, run `references/sync.md` first.

## MCP Tool Sequence

1. `rulesets_create({ name, description, rules })` — create standalone ruleset
2. `rulesets_rules_add({ rulesetId, rule })` — append rules to existing ruleset
3. For full replacement: use `applications_yaml_sync({ applicationId, content, approved })` with approval

## Rules

- Use `rulesets_create` for standalone rulesets and `rulesets_rules_add` to append rules
- There is no individual rule update or delete tool; use `applications_yaml_sync` for full replacement with approval
- When creating only a ruleset (without an application), always prefer `rulesets_create` over `applications_yaml_create`