# Chai Studio MCP Workflows

This reference covers the operations the skill should perform after reading `chai-studio.json`.

## Connection Smoke Test

Use this when the user asks whether Chai Studio MCP works:

1. `get-profile`
2. `get-applications`
3. `get-presets`
4. `get-rulesets`
5. `get-design-doc` for `AGENTS.md`
6. `get_audits` for the configured `applicationId`

Avoid write tools during smoke tests unless the user explicitly asks for test data.

## Sync Local Design Docs

1. Read `chai-studio.json`.
2. Fetch apps, presets, and rulesets.
3. Match:
   - application by `applicationId`
   - preset by `presetId`
   - ruleset by `auditRuleSetId`
4. Fetch `AGENTS.md` with `get-design-doc` (required) and `README.md` if relevant.
5. Create or replace `DESIGN.md` from current project spec + preset context (override behavior, not merge).
6. Update `DESIGN-RULES.md` with ruleset IDs, categories, severities, and reporting format.

## Audit Local UI

Every audit request must follow this sequence:

1. Reconcile previous audits.
2. Start a new audit run.

Detailed flow:

1. Confirm configured application and ruleset.
2. Call `get_audits` for the configured `applicationId`.
3. For previous runs, call `get_audit_violations` and inspect open findings.
4. Verify whether previously reported issues are now fixed in the current code/UI.
5. For each verified fix, call `update_violation_status` with `status: "resolved"` (and `resolvedBy` when available).
6. Read spec context in this order: `AGENTS.md` (`get-design-doc` or `design://agents-md`), then ruleset (`get-rulesets`), then preset (`get-presets`), then local design docs.
7. Read the requested files or inspect the requested browser view.
8. Audit against project hard rules first (`MUST`/`NEVER`), then ruleset severity guidance.
9. Prioritize critical and high findings (especially accessibility, keyboard/focus/dialog, metadata correctness).
10. Keep findings tight: exact code, why it matters, and code-level fix.
11. Upload a fresh run with `add_audit_results` (even when zero new violations are found).

Granularity requirements:

- Cover each relevant page and each relevant component in the requested scope.
- Log findings at page/component granularity instead of broad aggregate notes.
- For each finding, include where it was observed (page/view + component + file reference when available).

Runtime validation requirements:

- If browser tooling is available, open the app and validate behavior in runtime, not just source code.
- Validate keyboard navigation, focus/dialog behavior, accessibility semantics, responsive behavior, and interaction states.
- Annotate findings as static-only, runtime-only, or confirmed by both.

Useful categories:

- `accessibility`
- `visual`
- `typography`
- `color`
- `layout`
- `motion`
- `interaction`
- `responsive`
- `metadata`
- `performance`
- `content`

Useful severities:

- `critical`
- `high`
- `medium`
- `low`

Recommended category mapping from common project spec areas:

- accessibility and semantics -> `accessibility`
- typography hierarchy and readability -> `typography`
- color contrast and token misuse -> `color`
- spacing/layout rhythm -> `layout`
- animation/motion safety -> `motion`
- interaction patterns and dialog behavior -> `interaction`
- viewport/breakpoint issues -> `responsive`
- SEO/head/meta/canonical issues -> `metadata`
- rendering/perf anti-patterns -> `performance`
- clarity/microcopy/content quality -> `content`

## Upload Audit Results

Use `add_audit_results` after a real audit, or when the user explicitly asks for dummy/test audits.
Always upload a fresh audit run after the reconciliation step above, even if all old issues were resolved and the new run contains zero violations.

Required run fields:

- `applicationId`
- `auditRuleSetId`
- `status`: `completed` or `failed`
- `violations`: array

Recommended run fields:

- `summary`
- `totalRulesEvaluated`

Violation fields should be as complete as possible:

```json
{
  "ruleId": "impeccable-a11y-contrast",
  "ruleTitle": "Verify accessible contrast",
  "category": "accessibility",
  "severity": "critical",
  "guidance": "Flag foreground/background pairs below WCAG text contrast expectations.",
  "violationDescription": "The muted text has insufficient contrast on the accent surface.",
  "aiFixPrompt": "In src/app/page.tsx, replace the low-contrast foreground token on the affected paragraph with a semantic on-accent foreground token. Verify WCAG AA contrast and keep the existing layout unchanged.",
  "filePath": "src/app/page.tsx",
  "lineStart": 42,
  "lineEnd": 42,
  "codeSnippet": "<p className=\"text-gray-400 bg-blue-500\">...</p>",
  "frameworkHint": "React, Tailwind CSS",
  "suggestedFix": "Use `text-white` or a project-defined `text-on-accent` token."
}
```

`aiFixPrompt` must be self-contained. It should name the file, the issue, the desired fix, and the constraints.
`violations` is capped at 200 items per `add_audit_results` call.

## Read Audits and Violations

- Use `get_audits` with `applicationId` to list runs for the configured app.
- Use `get_audit_violations` with `auditId` to inspect findings.
- Filter by `category`, `severity`, or `status` when triaging.

## Update Violation Status

Use `update_violation_status` only after triage or verification:

```json
{
  "violationId": "violation-id",
  "status": "resolved",
  "resolvedBy": "agent-or-user-name"
}
```

Status meanings:

- `open`: still needs attention.
- `resolved`: fix verified.
- `ignored`: accepted exception or out of scope.
- `false_positive`: finding is incorrect.

Fix-and-close workflow:

1. Apply code/UI fix for selected audit entries.
2. Re-check the fixed behavior (runtime when available).
3. Call `update_violation_status` for each fixed/triaged violation.
4. Keep unresolved items as `open` and explain blockers.

### Required sequencing for every audit run

1. `get_audits` (by `applicationId`)
2. `get_audit_violations` (for prior runs that matter)
3. `update_violation_status` for verified fixes
4. fresh audit analysis
5. `add_audit_results` to create a new run

## Dummy Audit Policy

Dummy audits are allowed only when requested. Make them unmistakable:

- Put `Dummy` or `smoke-test` in the summary.
- Add `metadata.dummy: true` to every violation.
- Use fake file paths that cannot be confused with verified findings, or clearly mark the descriptions as synthetic.

## Failure Handling

- If Chai Studio has no applications, tell the user to create one in Chai Studio first.
- If `chai-studio.json` references missing IDs, fetch current apps/presets/rulesets and ask before changing the config.
- If an upload fails, preserve the prepared payload in the response or a local scratch file only if the user asks.
- If local docs conflict with Chai Studio rules, treat Chai Studio as source of truth and call out the mismatch.
