# Chai Studio Skill Configuration

## Required MCP Dependencies

- Chai Studio MCP
- Chrome DevTools MCP

Both are required for website exploration to artifact workflows.

If Chrome DevTools MCP is unavailable, instruct user to install/enable it and pause.

## Pagination Defaults

List/discovery operations are paginated at 20 items per page by default.

Applies to:

- applications
- presets
- rulesets
- rules

Agents must iterate pages until target is found or pages are exhausted.

## Preset Data Contract

`create_preset` expects JSON representation of design YAML.
Do not pass a raw YAML string.

Suggested normalized payload shape:

```json
{
  "name": "Preset Name",
  "design": {
    "tokens": {
      "color": {},
      "typography": {},
      "spacing": {},
      "radius": {},
      "shadow": {},
      "border": {}
    },
    "components": {},
    "states": {},
    "metadata": {
      "sourceUrl": "https://example.com",
      "capturedAt": "2026-05-01T00:00:00.000Z"
    }
  }
}
```

## Application Creation Contract

`create_application` requires:

- `presetId`
- `rulesetId` or `rulesetIds` (as supported by the MCP schema)

Resolve IDs through paginated list discovery first.

## Ruleset Contracts

- `create_ruleset`: standalone ruleset creation flow.
- `add_rule_to_ruleset`: attach existing rule to explicit `rulesetId`.

## Font Fallback Policy

When extracted fonts are unavailable in Chai/platform context:

1. Keep category match first.
2. Prefer open/platform fonts with similar style metrics.
3. Use deterministic priority ordering.
4. Always report fallback and rationale.
