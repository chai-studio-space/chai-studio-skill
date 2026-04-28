---
name: chai-studio-skill
description: Use when a project is connected to Chai Studio MCP for application-aware design system work, local DESIGN.md and DESIGN-RULES.md maintenance, preset/ruleset selection, UI audits, audit uploads, and violation follow-up. Triggers when the user mentions Chai Studio, chai-studio.json, Chai MCP, design presets, audit rulesets, uploading UI audit results, or syncing project design guidance with MCP.
license: MIT
metadata:
  version: 1.0.0
  owner: akshayt
  short-description: Use Chai Studio MCP for design rules and audits
---

# Chai Studio MCP

Use this skill to make Chai Studio the source of truth for project design context, while keeping local project docs useful for humans and agents.

## First Move

1. Look for `chai-studio.json` at the project root.
2. If it is missing, call `mcp__chai_studio__.get-applications`, ask the user to choose which application to configure, then create `chai-studio.json` from that selection.
3. Read the configured application, preset, and audit ruleset before editing UI or running audits.
4. Keep `DESIGN.md` and `DESIGN-RULES.md` in sync with the Chai Studio preset, ruleset, and project guidance.

For the config schema and examples, read `references/config.md`.
For full workflows and payload patterns, read `references/workflows.md`.

## Default Project Contract

`chai-studio.json` identifies the project's default Chai Studio application. Once present, use it without repeatedly asking the user:

- `applicationId`: default application for audits and design context.
- `presetId`: preferred design preset; verify against Chai Studio when possible.
- `auditRuleSetId`: default ruleset for audit uploads. Pick this from the selected application's `auditRuleSetIds`, then validate it with `get-rulesets`.
- `designDocFiles`: local docs to create or update, usually `DESIGN.md` and `DESIGN-RULES.md`.

Do not invent IDs. Fetch them from Chai Studio MCP.

## MCP Tool Map

Use read tools early:

- `get-profile`: confirm authentication when the connection is uncertain.
- `get-applications`: find application IDs, preset IDs, and audit ruleset IDs.
- `get-presets`: inspect available design systems and tokens.
- `get-rulesets`: inspect audit rules and severities.
- `get-design-doc`: fetch project docs exposed by Chai Studio MCP (`AGENTS.md` or `README.md`; defaults to `AGENTS.md`).
- `get_audits`: list audit runs, optionally filtered by `applicationId`.
- `get_audit_violations`: inspect violations by audit, category, severity, or status.

Use write tools intentionally:

- `add_audit_results`: upload completed or failed audit runs with actionable violations.
- `update_violation_status`: mark violations `resolved`, `ignored`, or `false_positive` after evidence supports that status.

Name hygiene matters: do not rename MCP tools when calling them. Chai Studio currently exposes both hyphenated (`get-profile`, `get-design-doc`, `get-rulesets`, `get-applications`, `get-presets`) and underscored (`get_audits`, `get_audit_violations`, `add_audit_results`, `update_violation_status`) tool names.

If a client supports MCP resources, you can also read:

- `design://agents-md`
- `design://readme-md`

## Local Design Docs

Maintain two local docs when the user asks to set up or sync Chai Studio for a project:

- `DESIGN.md`: product context, target audience, brand personality, design tokens, typography, color, spacing, components, motion, accessibility, and responsive expectations.
- `DESIGN-RULES.md`: audit rules, severity priorities, rule IDs, what to flag, and how to report/fix violations.

`DESIGN.md` is an override artifact, not a merge artifact:

- Always fetch `AGENTS.md` via `get-design-doc` first.
- Then create or fully replace `DESIGN.md` from current Chai Studio project spec context.
- Do not merge with stale local `DESIGN.md` content.
- If local content conflicts with current spec, prefer replacement aligned to current spec.

When using MCP for project docs, remember `get-design-doc` only supports `AGENTS.md` and `README.md`. Do not call it for `DESIGN.md` or `DESIGN-RULES.md`; read and edit those locally.

## Project Spec Execution Protocol

To follow Chai Studio project specifications effectively, treat project rules as a layered contract:

1. Read `AGENTS.md` first (via `get-design-doc` or `design://agents-md`) and extract hard constraints (`MUST`, `NEVER`, forbidden patterns).
2. Read configured ruleset via `get-rulesets` and map rule IDs/categories/severities.
3. Read configured preset via `get-presets` for token-level styling guidance.
4. Apply local `DESIGN.md` and `DESIGN-RULES.md` as project-specific overlays.

Priority order when rules conflict:

1. Safety and explicit user instruction
2. `AGENTS.md` hard constraints
3. Chai Studio ruleset severity and guidance
4. Preset/token conventions
5. Local style preference

When producing findings or fixes:

- Keep changes minimal and targeted; do not refactor unrelated surfaces.
- Provide exact path + line/snippet + impact + concrete fix.
- Prioritize critical accessibility, keyboard, focus/dialog, and metadata correctness issues before lower-severity visual polish.
- Avoid introducing prohibited patterns from project specs (for example unnecessary decorative motion or forbidden visual cliches).

## Audit Standards

Audit workflow is mandatory and always two-phase:

1. Reconcile previous audits first.
2. Start and upload a new audit run.

When the user asks for an audit, always run phase 1 before phase 2.

Audit depth is mandatory:

- Audit each and every relevant page and each and every relevant component in scope.
- Produce granular findings per page/component (do not collapse multiple issues into one vague item).
- Include exact artifact context in every finding: page/screen, component name, file path, and line/snippet when available.

Before uploading:

1. Identify the configured app and ruleset from `chai-studio.json`.
2. Get previous audits with `get_audits` for the same `applicationId`.
3. For relevant prior runs, fetch findings with `get_audit_violations`.
4. Verify in current code/UI whether prior issues are fixed.
5. For each verified fix, call `update_violation_status` before the new run.
6. Audit the actual files or UI surfaces requested by the user.
7. Classify each finding into supported MCP categories and severities before upload.
8. Report each violation with exact file path, line/snippet, why it matters, and a concrete fix.
9. Ensure each finding is grounded in project spec text (`AGENTS.md`) and/or ruleset guidance.
10. Upload only genuine findings unless the user explicitly asks for dummy/test audits.
11. Always create a new audit run with `add_audit_results` after reconciliation, even when zero new violations are found.

Browser/runtime validation:

- If browser tooling is available, open the app and validate in runtime in addition to static code review.
- Use browser/devtools-style checks for rendered behavior: accessibility tree/labels, keyboard flow, focus handling, responsive behavior, and interaction states.
- Record whether each finding came from static review, runtime validation, or both.

Each uploaded violation must include a self-contained `aiFixPrompt` that another agent could use without reading the whole conversation.

Chai Studio validation constraints to respect:

- `add_audit_results.status` must be `completed` or `failed`.
- `violations` max length is 200 per upload.
- `severity` must be one of: `critical`, `high`, `medium`, `low`.
- `category` must be one of: `accessibility`, `visual`, `typography`, `color`, `layout`, `motion`, `interaction`, `responsive`, `metadata`, `performance`, `content`.
- Provide line numbers as positive integers when present.

## Status Updates

Only mark a violation resolved after verifying the code or UI changed. Use:

- `resolved`: fixed and verified.
- `ignored`: intentionally accepted by the team or outside scope.
- `false_positive`: rule did not actually apply.

When changing statuses, include the resolver identity when available.

When running audits, status reconciliation is mandatory:

- Always check previous audits first.
- Resolve already-fixed violations before creating a fresh audit run.
- Do not skip the new run after reconciliation.

Fix workflow requirement:

- When the user asks to fix audit entries, implement code/UI fixes for the selected violations.
- After each fix, re-validate (runtime when available).
- Then call `update_violation_status` so each fixed item is marked appropriately (`resolved`, `ignored`, or `false_positive` with evidence).

## Safety

- Do not create dummy audits unless the user asks for test data.
- Do not overwrite local design docs without preserving project-specific content.
- Do not migrate UI libraries, rewrite the design system, or refactor broad UI areas just to satisfy an audit.
- Prefer Chai Studio IDs and tokens over memory or guesses.
