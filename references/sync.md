# Setup and Sync Workflow

**This workflow is mandatory only when the MCP client is a coding agent working in a local codebase or when local files are part of the task.**

For non-code MCP clients without filesystem access, or for MCP-only design work such as canvas planning, ruleset setup, audit record management, and journal notes, skip local file sync. Instead call `applications_get`, ask the user to choose an application if needed, then fetch context directly with `applications_yaml_get({ applicationId })` and `rulesets_get({ applicationId })`.

## Step 1: Resolve chai-studio.yaml

1. Look for `chai-studio.yaml` at the project root and ensure it's listed in `.gitignore`
2. If `chai-studio.yaml` is missing but `design-studio.yaml` exists:
   - Parse `design-studio.yaml`
   - Call `applications_get`
   - Match by exact application name
   - If matched, create `chai-studio.yaml` from that application
   - If not matched, ask for explicit user approval before calling `applications_yaml_create({ content, approved: true })`
3. If both files are missing:
   - Call `applications_get`
   - Ask the user to choose the Chai Studio application
   - Create `chai-studio.yaml` using the selected application
   - Call `applications_yaml_get({ applicationId })` and write `design-studio.yaml`
4. Never invent `applicationId`, `presetId`, `auditRuleSetId`, `auditId`, or `violationId`

## Step 2: Local Sync Gate

After project identity is resolved and before each local-codebase operation that depends on project design context:

1. Call `applications_yaml_get({ applicationId })` and read its `lastUpdated`
2. Read local `design-studio.yaml` and its top-level `lastUpdated`
3. If MCP `lastUpdated` is older than local, call `applications_yaml_sync({ applicationId, content, approved: false })` to get diff
4. Show diff summary to user and ask for approval
5. Only after approval call `applications_yaml_sync({ applicationId, content, approved: true })`
6. Replace local `design-studio.yaml` with returned content and update `chai-studio.yaml`

## Step 3: Fetch Live Design Contract

1. `applications_yaml_get({ applicationId })` — canonical application design payload
2. `rulesets_get({ applicationId })` — build working map of rules
3. `presets_get()` — find configured preset
4. Delete any local `DESIGN.md`, `DESIGN-RULES.md`, `design.yaml` mirrors