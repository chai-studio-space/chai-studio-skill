# Chai Studio MCP Command Router

**Run `references/sync.md` first only for local codebase work or when local files are part of the task.**

For non-code MCP clients without filesystem access, skip local sync files. Resolve the app with `applications_get`, fetch context with `applications_yaml_get` and `rulesets_get`, then run the selected Chai Studio workflow entirely through MCP tools.

This file supports both automatic intent routing and explicit command-style invocations. Automatic routing is the default; command mode is the deterministic override when you want exact control.

## Routing Modes

- Auto mode (default): infer intent from user language and run the matching workflow file
- Command mode (override): if the user provides `/chai ...`, run that command mapping directly
- Conflict rule: explicit command always wins over inferred intent

## Primary Commands

| Command | Workflow File | Description |
|---------|-------------|-------------|
| `/chai setup` | `references/sync.md` | Setup + sync gate for local codebase |
| `/chai redesign` | `references/redesign.md` | Fetch apps_yaml_get + rulesets_get, apply constraints, validate, audit, document |
| `/chai canvas` | `references/canvas.md` | 3-step canvas: DISCOVER → PLAN → CREATE pages |
| `/chai fix-canvas` | `references/canvas.md` | Fix mode: list canvases, annotations, update, mark fixed |
| `/chai audit` | `references/audit.md` | Reconcile audits, runs_start, violations_add, runs_complete |
| `/chai extract` | `references/website-exploration.md` | Website exploration (skip sync when extraction-only) |
| `/chai shadcn` | `references/shadcn-development.md` | shadcn_css_get → component_list → component_get → integrate |
| `/chai rulesets` | `references/ruleset-management.md` | rulesets_create, rules_add |
| `/chai journal` | `references/journal.md` | journals_list → entries_create/update |

## Execution Rules

- For local codebase work, command order is strict: run `/chai setup` before other commands
- For non-code MCP clients, replace `/chai setup` with direct app/context resolution via `applications_get`
- `/chai extract` may run without setup when extraction-only
- If app exists and command is `/chai canvas`, feature discovery is mandatory before page creation
- If command is `/chai fix-canvas`, always list canvases first and get explicit user selection
- Every canvas page must be responsive
- During canvas creation/edits, publish progress via `generationStatus`
- Use exact underscore MCP tool names; do not alias tools