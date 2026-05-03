# Redesign Workflow

Use this workflow when the user asks to redesign, restyle, refresh, polish, modernize, rework, improve, or rebuild a UI surface.

**Prerequisite:** For local codebase work, run `references/sync.md` first. For non-code MCP clients without filesystem access, resolve the application with `applications_get` and fetch context with `applications_yaml_get` and `rulesets_get` instead.

## MCP Tool Sequence

1. `applications_yaml_get({ applicationId })` — fetch preset and ruleset constraints
2. `rulesets_get({ applicationId })` — fetch linked rulesets
3. Apply preset/ruleset constraints in code implementation
4. Validate with project build when available
5. `audits_runs_start({ applicationId, auditRuleSetId })` — start audit
6. `audits_violations_add({ auditId, violation })` — upload findings one at a time
7. `audits_runs_complete({ auditId, status })` — complete audit
8. `journals_entries_create({ journalId, title, markdown })` or `journals_entries_update({ entryId, title, markdown })` — document decisions

## Implementation Rules

- Use Chai Studio preset tokens and semantic roles over arbitrary colors, fonts, radii, shadows, or spacing
- Enforce component-level adherence: border radius, spacing, typography, semantic colors/tokens, borders/shadows, and interaction states must match `design-studio.yaml` and linked rulesets
- Preserve application behavior, data flow, routing, auth boundaries, form submission, and analytics unless the user asked to change them
- Build real UI states: loading, empty, error, disabled, hover, focus, active, selected, expanded/collapsed, validation, and long-content states when they apply
- Keep accessibility first: semantic landmarks, labels, keyboard flow, focus visibility, dialog behavior, hit targets, reduced motion, and contrast

## Validation

- Run project build when available (e.g., `npm run build`, `pnpm build`, `yarn build`, or equivalent)
- Run relevant focused checks: typecheck, lint, tests
- Fix implementation defects before audit upload

## Completion Report

Report to user:
- Files changed
- MCP app/preset/rulesets used
- Build command run and result
- Audit runs created with violation counts
- Violations fixed and statuses updated