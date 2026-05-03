# Chai Studio Project Config

Create `chai-studio.yaml` in the project root. This local file tells the skill which Chai Studio application, preset, and rulesets to use by default. Add `chai-studio.yaml` to the project `.gitignore`.

`design-studio.yaml` is the synced application contract. It is generated from `applications_yaml_get`, includes `lastUpdated`, and contains application details, the full linked preset, and all linked rulesets. It is not supposed to be manually edited; update it through `applications_yaml_get` or the approved `applications_yaml_sync` flow.

## Required MCP Dependencies

- Chai Studio MCP
- Chrome DevTools MCP (required for website exploration workflows only)

## Pagination Defaults

List/discovery operations are paginated at 20 items per page by default. Applies to:
- `applications_get`
- `presets_get`
- `rulesets_get`
- `rulesets_rules_get`

Agents must iterate pages until target is found or pages are exhausted.

## Minimal `chai-studio.yaml`

```yaml
applicationId: d7f3c8cd-4288-4f21-9a42-386711866539
applicationName: Customer Portal
presetId: heritage-seed
auditRuleSetIds:
  - ruleset-default-craft
sync:
  source: chai-studio
  file: design-studio.yaml
  lastApplicationYamlUpdatedAt: "2026-04-28T00:00:00.000Z"
```