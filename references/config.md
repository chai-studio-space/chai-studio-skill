# Chai Studio Project Config

Create `chai-studio.json` in the project root. This file tells the skill which Chai Studio application, preset, and ruleset to use by default.

## Minimal Schema

```json
{
  "$schema": "./chai-studio.schema.json",
  "applicationId": "d7f3c8cd-4288-4f21-9a42-386711866539",
  "applicationName": "Customer Portal",
  "presetId": "heritage-seed",
  "auditRuleSetId": "ruleset-default-craft",
  "designDocFiles": {
    "design": "DESIGN.md",
    "rules": "DESIGN-RULES.md"
  },
  "sync": {
    "source": "chai-studio",
    "lastSyncedAt": "2026-04-28T00:00:00.000Z"
  }
}
```

## Recommended Fields

- `applicationId`: required. Use `get-applications`; never invent this.
- `applicationName`: optional but helpful for humans.
- `presetId`: required for design sync. Usually present on the application record.
- `auditRuleSetId`: required for audit upload. Choose from the application's `auditRuleSetIds` list returned by `get-applications`, then verify details via `get-rulesets`.
- `designDocFiles.design`: local design documentation path. Default: `DESIGN.md`.
- `designDocFiles.rules`: local audit rules documentation path. Default: `DESIGN-RULES.md`.
- `sync.source`: usually `chai-studio`.
- `sync.lastSyncedAt`: update after meaningful doc sync.
- `metadata`: optional object for repo, branch, team, or environment notes.

## Setup Workflow

1. Call `get-applications`.
2. If one application clearly matches the project, use it.
3. If several plausible apps exist, ask the user which one to configure.
4. Write `chai-studio.json`.
5. Call `get-presets` and `get-rulesets` to verify `presetId` and `auditRuleSetId`.
6. Create or update `DESIGN.md` and `DESIGN-RULES.md`.

## Optional JSON Schema

If the project benefits from editor validation, create `chai-studio.schema.json`:

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Chai Studio Project Config",
  "type": "object",
  "required": ["applicationId", "presetId", "auditRuleSetId"],
  "properties": {
    "$schema": { "type": "string" },
    "applicationId": { "type": "string", "minLength": 1 },
    "applicationName": { "type": "string" },
    "presetId": { "type": "string", "minLength": 1 },
    "auditRuleSetId": { "type": "string", "minLength": 1 },
    "designDocFiles": {
      "type": "object",
      "properties": {
        "design": { "type": "string" },
        "rules": { "type": "string" }
      },
      "additionalProperties": false
    },
    "sync": {
      "type": "object",
      "properties": {
        "source": { "type": "string" },
        "lastSyncedAt": { "type": "string" }
      },
      "additionalProperties": true
    },
    "metadata": { "type": "object" }
  },
  "additionalProperties": false
}
```

## DESIGN.md Outline

Use concise sections:

```markdown
# Design System

## Product Context

- Target audience:
- Primary use case:
- Brand personality:

## Chai Studio Source

- Application:
- Preset:
- Last synced:

## Tokens

### Color

### Typography

### Spacing, Radius, Shadow

## Components

## Accessibility

## Motion and Performance

## Responsive Behavior
```

## DESIGN-RULES.md Outline

Use rule IDs so audit payloads map back to Chai Studio:

```markdown
# Design Audit Rules

## Chai Studio Source

- Application:
- Ruleset:
- Last synced:

## Severity Order

1. Critical
2. High
3. Medium
4. Low

## Rules

### rule-id

- Title:
- Category:
- Severity:
- Guidance:
- What to flag:
- Preferred fix:

## Reporting Format

For every violation, include file, line/snippet, why it matters, and a concrete fix.
```
