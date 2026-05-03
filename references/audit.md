# Audit Workflow

Use this workflow when the user asks to audit, check compliance, find issues, or upload an audit.

**Prerequisite:** For local codebase work, run `references/sync.md` first. For non-code MCP clients without filesystem access, resolve the application with `applications_get` and fetch context with `applications_yaml_get` and `rulesets_get` instead.

**Auditing and fixing are SEPARATE flows:**
- "Audit this" / "find issues" / "check compliance" → Audit ONLY
- "Fix audit findings" / "remediate" / "resolve violations" → Fix workflow (only when explicitly asked)

## MCP Tool Sequence

1. Reconcile prior audits: `audits_get({ applicationId })`
2. For each open run: `audits_violations_get({ auditId, status: "open" })`
3. Start audit: `audits_runs_start({ applicationId, auditRuleSetId, summary })`
4. Upload findings: `audits_violations_add({ auditId, violation })` — one at a time
5. Complete: `audits_runs_complete({ auditId, status })`

## Violation Input Format

Each violation must include:
- `ruleId`, `ruleTitle`
- `severity`: "critical" | "high" | "medium" | "low"
- `category`: "accessibility" | "visual" | "typography" | "color" | "layout" | "motion" | "interaction" | "responsive" | "metadata" | "performance" | "content"
- `filePath`, `lineStart`, `lineEnd`, `codeSnippet`
- `violationDescription`, `guidance`
- `aiFixPrompt`: Self-contained prompt for fixing
- Optional: `suggestedFix`, `frameworkHint`, `metadata`

## Fix Audit (Only When Explicitly Asked)

When the user explicitly asks to fix, remediate, resolve, or close violations:
1. Verify the finding before marking resolved
2. `audits_violations_status_update({ violationId, status, resolvedBy })` with status: "resolved" | "ignored" | "false_positive"
3. If all violations are resolved, the audit run status auto-resolves

## Rules

- Upload one finding at a time. The server deduplicates identical rule/file/line/snippet/description findings within a run.
- If multiple `auditRuleSetIds` apply, audit each ruleset separately.