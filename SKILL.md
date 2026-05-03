---
name: chai-studio-skill
description: Use when a project is connected to Chai Studio MCP for application-aware design system work, Chai-governed redesigns, preset/ruleset selection, UI audits, audit uploads, canvas management, journal entries, violation follow-up, or website exploration via Chrome DevTools MCP. Triggers when the user mentions Chai Studio, chai-studio.yaml, design-studio.yaml, Chai MCP, design presets, audit rulesets, redesign workflow, uploading UI audit results, syncing project design guidance with MCP, canvases, journals, or website exploration.
license: MIT
metadata:
  version: 2.1.0
  owner: akshayt
  short-description: Use Chai Studio MCP for design rules, audits, canvases, journals, and website exploration
---

# Chai Studio MCP

Use this skill to make Chai Studio MCP the only source of truth for design context, audit rules, canvas planning, and journal documentation. Also supports Chrome DevTools MCP for evidence-driven website exploration and artifact creation.

## First Move

For application-aware work, resolve an application with `applications_get` unless the user already provided an `applicationId`.

- **If the MCP client is a coding agent with filesystem access and the user asks for code changes**: use the local project contract — `chai-studio.yaml` identifies the application and `design-studio.yaml` is the synced source of truth.
- **If the MCP client is a non-code assistant without a local filesystem**: do not require `chai-studio.yaml`, `design-studio.yaml`, or local sync. Use `applications_get` to choose an application and `applications_yaml_get`/`rulesets_get` to fetch context directly.
- **Non-code MCP clients can still create and edit canvases**, inspect presets/rulesets, manage audit records, and create journal entries entirely through MCP tools.
- **For codebase work**: fetch or refresh `design-studio.yaml` with `applications_yaml_get` before redesigns, audits, canvas-to-code work, ruleset work, shadcn generation, or journal documentation.
- **Do not invent Chai Studio IDs**. Do not maintain separate local `DESIGN.md`, `DESIGN-RULES.md`, or `design.yaml` mirrors.
- **If syncing local design-studio.yaml back to Chai Studio**: call `applications_yaml_sync` with `approved=false` first, show the diff, then call `approved=true` only after explicit user approval.

## Mandatory Sync Gate

For local codebase work, after project identity is resolved and before each Chai Studio operation that depends on local design context:

1. Call `applications_yaml_get({ applicationId })` and read its `lastUpdated`.
2. Read local `design-studio.yaml` and its top-level `lastUpdated`.
3. If the MCP `lastUpdated` is older than local `design-studio.yaml`, call `applications_yaml_sync({ applicationId, content, approved: false })` to get the diff. Show the diff summary to the user and ask for approval. Only after approval call `applications_yaml_sync({ applicationId, content, approved: true })`, then replace local `design-studio.yaml` with the returned `content` and update `chai-studio.yaml`.
4. If the MCP `lastUpdated` is newer than local `design-studio.yaml`, replace local `design-studio.yaml` with `applications_yaml_get` content and update `chai-studio.yaml`.
5. If timestamps match, keep using local `design-studio.yaml`; if the file is missing or invalid, replace it with `applications_yaml_get` content.

For non-code MCP or MCP-only design work without local filesystem access, skip this local file sync gate. Fetch fresh context with `applications_yaml_get({ applicationId })` and `rulesets_get({ applicationId })` instead.

## Canvas Rules (Required)

Canvas workflow is 3 steps in order:

1. **DISCOVER** components by reading the canvas runtime source with `canvas_runtime_read` — this is the ultimate source of truth for exact component signatures. Use `canvas_components_list` / `canvas_components_search` for quick summaries only.
2. **PLAN** the design — decide which `UI.*` components you will use before writing code.
3. **CREATE / UPDATE** pages.

**Additional rules:**
- For multi-page flows, ALWAYS use this sequence: (a) Create ALL empty pages first with `generationStatus: "started"` and empty `componentTsx`, (b) Add all page-to-page links with `canvases_pages_links_create`, (c) Generate each page one by one by updating `componentTsx` and setting `generationStatus` to `"generating"` then `"done"`. This lets the user see the full page structure and navigation immediately.
- Canvas pages must be responsive across mobile, tablet, and desktop widths.
- All `UI.*` components are theme-aware via CSS variables. Do NOT handle light/dark manually. Use `UI.*` components directly and they adapt automatically.
- Keep page TSX theme-neutral. Canvas theme mode controls light/dark rendering.
- Use `presetMappedCss` from `canvases_base_css_get` as the base CSS. Do NOT call `canvases_css_update` unless absolutely necessary — presetMappedCss already includes all required tokens and bundled components are self-contained.
- For in-progress canvas work, provide progress updates through `generationStatus` on page create/update: `started` or `generating` while working, `done` only after completion.
- When editing pages, use `generationStatus` `started` or `generating` during work and `done` only after the edit is complete.

## Audit Rules (Required)

- Each violation should include exact file path, line/snippet when available, severity, category, rule guidance, and a self-contained `aiFixPrompt`.
- Upload one finding at a time. The server deduplicates identical rule/file/line/snippet/description findings within a run.
- If multiple `auditRuleSetIds` apply, audit each ruleset separately.

For the config schema and examples, read `references/config.md`.
For workflow routing (which file to use when), read `references/workflows.md`.
For the mandatory setup/sync gate, read `references/sync.md`.
For redesign steps, read `references/redesign.md`.
For canvas planning, read `references/canvas.md`.
For audit flows, read `references/audit.md`.

## Workflow Routing References

| Workflow | Reference File | Description |
|---|---|---|
| redesign | `references/redesign.md` | Fetch applications_yaml_get + rulesets_get, apply preset/ruleset constraints in code, validate, audit, document in journal |
| canvas | `references/canvas.md` | 3-step canvas: (1) DISCOVER with canvas_runtime_read, (2) PLAN UI.* components, (3) CREATE pages |
| fixCanvas | `references/canvas.md` | canvases_list → canvases_annotations_get status=open → update pages → canvases_annotations_status_update |
| audit | `references/audit.md` | Reconcile audits, audits_runs_start, audits_violations_add, audits_runs_complete |
| fixAudit | `references/audit.md` | Only change code or mark violations resolved when explicitly asked |
| shadcn | `references/shadcn-development.md` | shadcn_css_get → shadcn_component_list → shadcn_component_get → integrate |
| rulesets | `references/ruleset-management.md` | rulesets_create → rulesets_rules_add, use applications_yaml_sync for full replacement |
| journal | `references/journal.md` | journals_list → journals_entries_create/update |

## Default Project Contract

`chai-studio.yaml` identifies the project's default Chai Studio application. Once present, use it without repeatedly asking the user:

- `applicationId`: default application for audits, canvases, and design context.
- `presetId`: preferred design preset; verify against Chai Studio when possible.
- `auditRuleSetIds`: default rulesets for audit uploads. Pick this from the selected application's `auditRuleSetIds`, then validate them with `rulesets_get` using `applicationId`.
- `sync.lastApplicationYamlUpdatedAt`: last MCP application YAML timestamp written locally.

Do not invent IDs. Fetch them from Chai Studio MCP.

## MCP Tool Map

### Read / Context Tools

- `workflow_guide`
  - **Purpose**: First move guidance when Chai Studio skill is not installed. Returns setup, sync, canvas, audit, ruleset, shadcn, and journal workflows.
- `profile_get`
  - **Purpose**: Confirm authentication when the connection is uncertain.
- `applications_get`
  - **Purpose**: Fetches all applications belonging to the authenticated user.
  - **Includes**: Crucial metadata linking each app to its design system (`presetId`) and its array of connected audit rulesets (`auditRuleSetIds`).
- `applications_yaml_get`
  - **Input**: `applicationId`
  - **Purpose**: Returns `design-studio.yaml` content with application details, full linked preset, full linked rulesets, and `lastUpdated`.
- `design_docs_get`
  - **Input**: `applicationId`
  - **Purpose**: Legacy markdown design representation derived from the linked preset. Prefer `applications_yaml_get` + `design-studio.yaml` as the project source-of-truth file.
- `rulesets_get`
  - **Input**: `applicationId`, optional paging (`page`, `pageSize`)
  - **Purpose**: Fetches audit rule sets linked to a specific application.
- `rulesets_rules_get`
  - **Input**: `rulesetId`, optional paging (`page`, `pageSize`)
  - **Purpose**: Fetches rules from a specific ruleset.
- `presets_get`
  - **Input**: optional paging (`page`, `pageSize`)
  - **Purpose**: Fetches all raw design presets configuration data associated with the user.
- `fonts_list`
  - **Input**: optional `query` string for filtering
  - **Purpose**: Lists all Google Fonts supported by Chai Studio preset typography selectors.
- `fonts_search`
  - **Input**: `query` string, optional paging
  - **Purpose**: Search Google Fonts by name with case-insensitive matching.
- `audits_get`
  - **Input**: optional `applicationId`, optional `includeArchived`
  - **Purpose**: Fetches the history of audit runs (id, status, severity counts, etc).
- `audits_violations_get`
  - **Input**: `auditId`, optional filters (`status`, `severity`, `category`)
  - **Purpose**: Fetches the granular violations for a specific audit. Crucially, each finding returns an `aiFixPrompt`, the exact `codeSnippet`, and guidance to allow an agent to immediately patch the failing file.
- `canvases_list`
  - **Input**: `applicationId`
  - **Purpose**: Lists all canvases for an application. Use to discover existing canvases before creating or referencing context.
- `canvases_get`
  - **Input**: `canvasId`
  - **Purpose**: Gets a single canvas by ID. Returns canvas metadata and CSS. Use to read full canvas state before page operations.
- `canvases_pages_list`
  - **Input**: `canvasId`
  - **Purpose**: Lists all pages inside a canvas. Use to discover pages and their current state before editing.
- `canvases_pages_get`
  - **Input**: `pageId`
  - **Purpose**: Gets a single canvas page by ID. Returns page details including componentTsx and layout. Use to read a specific page before applying annotations or edits.
- `canvases_annotations_get`
  - **Input**: `pageId`, optional `status` filter (`open` | `fixed`)
  - **Purpose**: Fetches annotations for a canvas page. Use this to read open instructions before fixing them.
- `journals_list`
  - **Purpose**: Lists user journals and entries.

### Write / Management Tools

- `applications_create`
  - **Input**: `name`, `description`, `presetId`, `rulesetIds`
  - **Purpose**: Creates an application linked to an existing preset and one or more existing rulesets.
- `rulesets_create`
  - **Input**: `name`, `description`, `rules` (optional `citationUrl` only when the user explicitly requests it)
  - **Purpose**: Creates a standalone ruleset.
- `rulesets_rules_add`
  - **Input**: `rulesetId`, `rule`
  - **Purpose**: Attaches an existing rule object to a ruleset by ID.
- `presets_create`
  - **Input**: `preset` (JSON preset object in latest design-document shape; not YAML text)
  - **Purpose**: Creates a preset record.
  - **Important**: Provide `name`, `modes.light` and/or `modes.dark` with `palette` containing hex colors (`primary`, `secondary`, `tertiary`, `neutral`, `text`), and `typography.fonts.heading` / `typography.fonts.body`. Optional semantic values must be complete if supplied. If semantic blocks are omitted, complete semantic tokens are auto-generated from the palette. Do not include server-side fields such as `id`, `createdAt`, `updatedAt`, or `source`.
  - **Safe usage**: Prefer `applications_yaml_create` when you have `design-studio.yaml` content for full app + linked ruleset creation. Use `presets_create` when intentionally creating a standalone preset record.
- `presets_update`
  - **Input**: `presetId`, `preset` (JSON preset object in latest design-document shape; not YAML text)
  - **Purpose**: Updates an existing preset record.
  - **Important**: Uses the same design-document preset schema requirements as `presets_create`.
- `applications_yaml_create`
  - **Input**: `content`, `approved`
  - **Purpose**: Creates a new application, linked preset, and linked rulesets from valid `design-studio.yaml`. Requires explicit user approval before `approved: true`.
- `applications_yaml_sync`
  - **Input**: `applicationId`, `content`, `approved`
  - **Purpose**: Diffs local `design-studio.yaml` against Chai Studio and updates the linked preset/rulesets only after explicit user approval.

### Audit Tools

- `audits_runs_start`
  - **Input**: `applicationId`, `auditRuleSetId`, optional `summary`, optional `totalRulesEvaluated`
  - **Purpose**: Creates one audit run before analysis so findings can be uploaded as they are discovered.
- `audits_violations_add`
  - **Input**: `auditId`, `violation`
  - **Purpose**: Uploads one violation to the existing audit run. The server skips duplicates for the same audit when the same rule/file/line/snippet/description is uploaded again.
- `audits_runs_complete`
  - **Input**: `auditId`, `status`, optional `summary`, optional `totalRulesEvaluated`
  - **Purpose**: Marks a streaming audit run `completed` or `failed` after all findings have been uploaded.
- `audits_violations_status_update`
  - **Input**: `violationId`, `status`, optional `resolvedBy`
  - **Purpose**: Allows the agent or IDE to automatically mark an open violation as resolved, ignored, or a `false_positive`. If all violations in an audit run are squashed, this tool auto-resolves the parent audit status.

### Canvas Tools

- `canvases_create`
  - **Input**: `applicationId`, `name`
  - **Purpose**: Creates a canvas under an application for planning UI surfaces. Canvas pages are built as React component JSX.
- `canvases_pages_create`
  - **Input**: `canvasId`, `title`, optional `generationStatus`, `layout`, `componentTsx`
  - **Purpose**: Creates a page inside a canvas using React component JSX.
- `canvases_pages_update`
  - **Input**: `pageId`, optional `title`, `generationStatus`, `layout`, `componentTsx`
  - **Purpose**: Updates an existing canvas page JSX and generation state.
- `canvases_pages_links_create`
  - **Input**: `canvasId`, `fromPageId`, `toPageId`, optional `label`, `labelOffsetX`, `labelOffsetY`, `conditionExpr`, `metadata`
  - **Purpose**: Creates a directed link between two pages in the same canvas. Use `labelOffsetX` and `labelOffsetY` to position the label text relative to the midpoint of the connecting line (positive = right/down, negative = left/up).
- `canvases_pages_links_update`
  - **Input**: `linkId`, optional `fromPageId`, `toPageId`, `label`, `labelOffsetX`, `labelOffsetY`, `conditionExpr`, `metadata`
  - **Purpose**: Updates an existing directed link between pages in an owned canvas. Use `labelOffsetX` and `labelOffsetY` to reposition the label text.
- `canvases_annotations_create`
  - **Input**: `pageId`, `x`, `y`, optional `componentPath`, `instruction`
  - **Purpose**: Adds an annotation to a canvas page at a specific coordinate with an instruction for the AI agent to fix.
- `canvases_annotations_status_update`
  - **Input**: `annotationId`, `status` (`open` | `fixed`)
  - **Purpose**: Marks a canvas annotation as fixed or reopens it. Use this after the AI agent has addressed the instruction.
- `canvases_base_css_get`
  - **Input**: optional `canvasId`
  - **Purpose**: Returns the base utility CSS framework mapped to the linked application preset. Provides `presetMappedCss` with `:root` design tokens, component classes, and dark mode variants. Use this as the foundation for canvas shared CSS.
- `canvases_css_get`
  - **Input**: `canvasId`
  - **Purpose**: Fetches the shared CSS stylesheet for a canvas. Pages inherit this CSS automatically — use it to understand existing design tokens before writing new page markup.
- `canvases_css_update`
  - **Input**: `canvasId`, `css`
  - **Purpose**: Updates the shared CSS stylesheet for a canvas. All pages in the canvas inherit this CSS automatically across mobile, tablet, and desktop widths — write page React TSX in `componentTsx` and keep styling token-driven. Use this to define design tokens, component styles, and layout primitives once per canvas.
- `canvases_delete`
  - **Input**: `canvasId`
  - **Purpose**: Deletes a canvas and all its pages, links, and annotations.
- `canvases_pages_delete`
  - **Input**: `pageId`
  - **Purpose**: Deletes a single canvas page.
- `canvases_links_delete`
  - **Input**: `linkId`
  - **Purpose**: Deletes a directed link between pages.
- `canvases_artifacts_render_preview`
  - **Input**: `markup`
  - **Purpose**: Sanitizes AI-generated markup and returns secure rendering metadata (`sanitizedMarkup`, `csp`, `sandbox`).

### Canvas Component Explorer Tools

- `canvas_components_list`
  - **Input**: optional `page`, `pageSize`, optional `category`
  - **Purpose**: List all available UI components in the canvas runtime with pagination. Quick summary only — use `canvas_runtime_read` for the ultimate source of truth on exact component signatures.
- `canvas_components_search`
  - **Input**: `query`, optional `page`, `pageSize`, optional `category`
  - **Purpose**: Search canvas UI components by name, description, or category. Returns paginated results.
- `canvas_components_get`
  - **Input**: `id` (component ID like 'button', 'bar-chart', 'card')
  - **Purpose**: Get full documentation for a specific canvas UI component including description, usage syntax, props, example code, and related components.
- `canvas_runtime_read`
  - **Input**: optional `startLine`, `endLine`, optional `search`
  - **Purpose**: Read sections of the canvas UI runtime source file (`src/lib/shadcn/components/canvas-runtime.tsx`). This is the ULTIMATE source of truth for exactly which components exist, their props, and how they are implemented.

### Shadcn Development Tools (Non-Canvas)

- `shadcn_component_list`
  - **Input**: `applicationId`, optional `page`, optional `limit`
  - **Purpose**: Lists themed component names available for generation via `shadcn_component_get`. Supports pagination.
- `shadcn_component_search`
  - **Input**: `applicationId`, `query`
  - **Purpose**: Search available shadcn component names by keyword/query.
- `shadcn_component_get`
  - **Input**: `applicationId`, `componentName`
  - **Purpose**: Returns themed component TSX for real product development (not canvas-only mockups).
- `shadcn_css_get`
  - **Input**: `applicationId`
  - **Purpose**: Returns preset-mapped CSS tokens for product development.

### Journal Tools

- `journals_list`
  - **Purpose**: Lists user journals and entries. If the account has no journals yet, the server seeds default design and personal journals.
- `journals_entries_create`
  - **Input**: `journalId`, `title`, `markdown`
  - **Purpose**: Creates a markdown journal entry in a journal.
- `journals_entries_update`
  - **Input**: `entryId`, optional `title`, optional `markdown`
  - **Purpose**: Updates an existing journal entry's title and/or markdown content.

Name hygiene matters: use the exact underscore MCP tool names when calling them. Do not rename or alias tools.

`design-studio.yaml` is the only local design source-of-truth file for this skill, and it must be generated or synced through Chai Studio using `applications_yaml_get` and `applications_yaml_sync`.

If there is a `DESIGN.md`, `DESIGN-RULES.md`, `design.yaml`, or other local design mirror file already present in the workspace, you MUST delete it. The source of truth is strictly the Chai Studio MCP server. Do not create or maintain local design mirror docs outside `design-studio.yaml` synced from Chai Studio.

## Feature Discovery Requirement (Required)

If a functional application already exists, the agent MUST extract full granular feature details BEFORE creating or updating any canvas designs.

- Discover every feature, screen, flow, state, and edge case the app supports
- Understand data models, user roles, permissions, and business logic
- Map out navigation structure, form fields, validation rules, and error states
- Note third-party integrations, API endpoints, and async behaviors
- Capture feature flags, admin capabilities, and user settings

The canvas design MUST reflect the exact features supported by the app. Never design in a vacuum — always design with full feature context.

## Responsive Generation Requirement (Required)

**The agent MUST generate responsive pages without fail, always.**

All canvas pages must be responsive across common viewport sizes:
- Use Tailwind-style responsive prefixes or CSS media queries
- Ensure layouts adapt across mobile, tablet (`md:`), and desktop (`lg:`/`xl:`)
- Test for readability and usability at 320px, 768px, and 1280px widths
- Never deliver a page that is desktop-only or fixed-width

This is non-negotiable: responsive layout is part of every page generation.

## Command Routing

Support both routing modes:

- Auto intent routing (default): infer workflow from user request language.
- Command routing (override): `/chai ...` explicitly selects the workflow.

`/chai` command routing is defined in this `SKILL.md` + `references/workflows.md`, so it works across agents that support Agent Skills. `agents/openai.yaml` is OpenAI UI metadata only and is not required for command routing.

Use command-style routing for deterministic control:

- `/chai setup` -> `references/sync.md` (mandatory first command)
- `/chai redesign` -> `references/redesign.md`
- `/chai canvas` -> `references/canvas.md`
- `/chai fix-canvas` -> `references/canvas.md` (fix mode: list canvases, user chooses one, then resolve open annotations)
- `/chai audit` -> `references/audit.md`
- `/chai extract` -> `references/website-exploration.md`
- `/chai shadcn` -> `references/shadcn-development.md`
- `/chai rulesets` -> `references/ruleset-management.md`
- `/chai journal` -> `references/journal.md`

Then apply the detailed routing rules below.

## Workflow Routing

When a route below says to run `references/sync.md`, apply that only when operating as a coding agent in a local codebase or when local files are part of the task. For non-code MCP clients or MCP-only Chai Studio design work, resolve the app with `applications_get`, fetch context with `applications_yaml_get` and `rulesets_get`, then continue the selected workflow without creating local sync files.

**IF user asks to redesign, restyle, refresh, polish, modernize, rework, improve, or rebuild a UI surface:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/redesign.md`
→ Workflow: fetch applications_yaml_get + rulesets_get, apply preset/ruleset constraints, validate, audit, document in journal

**IF user asks to plan, wireframe, or design a multi-page flow:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/canvas.md`
→ Workflow: REQUIRED 3-step canvas — (1) DISCOVER with canvas_runtime_read, (2) PLAN UI.* components, (3) CREATE pages

**IF user asks to fix canvas annotations or uses `/chai fix-canvas`:**
→ Run `references/sync.md` first (mandatory)
→ Call `canvases_list({ applicationId })` and ask user to choose one canvas
→ Workflow: canvases_list → canvases_pages_list → canvases_annotations_get status=open → update pages → canvases_annotations_status_update

**IF user asks to audit, check compliance, or find issues:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/audit.md`
→ Workflow: reconcile audits → audits_runs_start → audits_violations_add → audits_runs_complete

**IF user asks to fix audit violations:**
→ Only change code or mark violations resolved when explicitly asked
→ Verify before using audits_violations_status_update

**IF user asks to create artifacts from an existing website:**
→ Skip `references/sync.md` when extraction-only
→ Run `references/website-exploration.md`

**IF user asks to build real UI components in the codebase:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/shadcn-development.md`
→ Workflow: shadcn_css_get → shadcn_component_list → shadcn_component_get → integrate

**IF user asks to create, extend, or inspect audit rulesets:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/ruleset-management.md`
→ Workflow: rulesets_create → rulesets_rules_add for rules, use applications_yaml_sync for full replacement

**IF user asks to document design decisions or audit results:**
→ Run `references/sync.md` first (mandatory)
→ Run `references/journal.md`
→ Workflow: journals_list → journals_entries_create/update

## Redesign Flow Summary

When a user asks to redesign a page, component, app surface, visual system, or interaction, read `references/redesign.md` for the full protocol. The mandatory steps are:

1. **Fetch context**: Call `applications_yaml_get` and `rulesets_get` to get preset and ruleset constraints.
2. **Apply constraints**: Translate preset/ruleset constraints into implementation decisions.
3. **Implement**: Redesign with `design-studio.yaml` and linked rulesets as the source of truth.
4. **Validate**: Run project build when available, plus static checks.
5. **Audit**: Reconcile prior audits, start an audit with `audits_runs_start`, upload findings with `audits_violations_add`, complete with `audits_runs_complete`.
6. **Document**: Create or update a design journal entry with `journals_entries_create` or `journals_entries_update`.

Do not stop after a visual pass. A Chai Studio redesign is complete only after implementation, validation, audit upload, and a design journal entry.

## Shadcn Development Flow (Product Code)

Use this flow when the user asks to implement or scaffold real UI in the codebase (not canvas planning):

1. Resolve `applicationId` from `chai-studio.yaml`.
2. Call `shadcn_css_get({ applicationId })` to obtain preset-mapped tokens.
3. Call `shadcn_component_list({ applicationId })` to inspect available themed components.
4. Call `shadcn_component_get({ applicationId, componentName })` for component TSX you need.
5. Integrate returned TSX into product code paths (`components/ui/...`) following repo conventions.
6. Validate build/tests, then continue normal audit/journal process when applicable.

## Audit Flow Summary

When the user asks to audit, check compliance, or find issues:

1. Reconcile prior audits.
2. Start an audit with `audits_runs_start({ applicationId, auditRuleSetId })`.
3. Upload findings one at a time with `audits_violations_add({ auditId, violation })`.
4. Complete with `audits_runs_complete({ auditId, status })`.

**Do NOT fix audit violations unless the user explicitly asks to fix, remediate, resolve, close, or update findings.**

## Canvas Flow Summary

When the user asks to plan, wireframe, or design a multi-page flow using Chai Studio canvases, or asks to fix canvas annotations via `/chai fix-canvas`, read `references/canvas.md` for the full protocol. Follow the REQUIRED 3-step canvas workflow:

### Step 1 — DISCOVER Components
Call `canvas_runtime_read` to inspect the actual canvas UI runtime source code (`src/lib/shadcn/components/canvas-runtime.tsx`) and understand exactly which components exist and their props. This is the ULTIMATE source of truth. For a quick summary, also call `canvas_components_list` or `canvas_components_search`.

### Step 2 — PLAN the Design
Decide which `UI.*` components you will use and how they compose. Choose layout utilities (`.stack`, `.row`, `.grid`, `.card`, `.container`) for structure. Plan color tokens (`var(--color-bg)`, `var(--color-fg)`, etc.). Do NOT skip planning.

### Step 3 — CREATE / UPDATE Pages
- Create or select a canvas with `canvases_create`
- Call `canvases_base_css_get({ canvasId })` to get `presetMappedCss` — do NOT call `canvases_css_update` unless absolutely necessary (presetMappedCss already includes all required tokens)
- For multi-page flows: FIRST create ALL empty pages with `generationStatus: "started"`, THEN add links with `canvases_pages_links_create`, THEN generate each page's UI one by one updating `generationStatus` from `"generating"` to `"done"`
- For single pages: create directly with `generationStatus: "done"` and full `componentTsx`

### Additional Steps
1. **Feature Discovery (REQUIRED if app exists)**: Granular analysis of current app features, screens, flows, data models, permissions, and business logic. Never design without full feature context.
2. **Resolve**: In a codebase, run the setup/sync gate. In a non-code MCP client, use `applications_get`, `applications_yaml_get`, and `rulesets_get` directly.
3. **Link pages**: Call `canvases_pages_links_create` to define navigation flows.
4. **Read open annotations first**: Call `canvases_annotations_get({ pageId, status: "open" })` before modifying existing pages.
5. **Annotate**: Call `canvases_annotations_create` to place fix instructions.
6. **Update page status**: Use streaming pattern — `started` → `generating` → `done`. Never set `done` while actively editing.
7. **Reconcile annotations**: Mark fixed annotations after addressing them.
8. **Document in design journal**: Create a design journal entry after canvas work is complete.

For `/chai fix-canvas`, this flow is constrained: list canvases, ask user to choose one canvas, read open annotations across every page in that canvas, fix all of them, and mark each annotation as `fixed`.

Canvas pages carry a `constraintsSnapshot` derived from the application's preset and rulesets at creation time. This lets future agents know which design system was in force when the page was planned.

### Canvas-To-Development Handoff (Required)

When the user asks to redesign or design a new feature, and canvas is used (created or updated), do this before any dev implementation:

1. **Ask user to review canvas** — "Please review the canvas and let me know if you want to add any annotations or changes."
2. **Ask user to add annotations** — "Please add any annotations for fixes you'd like me to address."
3. **Agent resolves annotations in one pass** — Read all open annotations, apply fixes to componentTsx/layout, then mark ALL as fixed in a single annotation-update loop.
4. ONLY after all annotations are fixed, ask user explicitly: "I've resolved all canvas annotations. Can I continue with development/code changes now?"
5. Proceed only after explicit user confirmation (unless user already explicitly asked immediate development).

**The agent MUST NOT start code implementation until user confirms after annotation resolution.**

### Page Edit Status Rule (Required)

When editing/updating a canvas page, do NOT set status to "done" during editing. Always:
1. Set `generationStatus: "started"` or `"generating"` BEFORE making edits via `canvases_pages_update`
2. Make the componentTsx/layout changes
3. Set `generationStatus: "done"` AFTER edits are complete

The agent MUST NOT call `canvases_pages_update` with `generationStatus: "done"` while actively editing — only set to done after finishing all changes.

### Design Journal Entry Contract

After completing any design work — redesigns, canvas wireframes, ruleset changes, audit remediation, or preset updates — you MUST document the work in a design journal. This creates an auditable history of design decisions.

To handle same-date entries:

1. **List first**: Call `journals_list` to find existing design journals and entries.
2. **Find journal**: Look for a journal with `kind: "design"`. If none exists, call `journals_list` again to allow default journal seeding, then use the seeded design journal.
3. **Check for today's entry**: Look for an entry with today's date in the title (e.g., "Design Log - 2024-01-15").
4. **Update or create**:
   - If an entry exists for today, **update it** with `journals_entries_update({ entryId, markdown: updatedContent })`. Append the new work summary to the existing entry.
   - If no entry exists for today, create one with `journals_entries_create({ journalId, title: "Design Log - 2024-01-15", markdown })`.

**Entry format:**
```markdown
## Work Summary
Brief description of what was done.

## Decisions
- Decision 1 and rationale
- Decision 2 and rationale

## Files/Routes Affected
- `file1.tsx`
- `file2.tsx`

## Preset/Canvas References
- Canvas: [canvas name]
- Preset: [preset name]

## Notes
Any follow-up items or open questions.
```

## Journal Flow Summary

When the user asks to document design decisions, audit results, or project notes, read `references/journal.md` for the full protocol. Key steps:

1. **List**: Call `journals_list` to see existing journals.
2. **Select journal**: Use the seeded `kind: "design"` journal for design documentation or `kind: "personal"` for general notes.
3. **Add entries**: Call `journals_entries_create` with the `journalId`, a `title`, and markdown `content`.

Use journals for post-audit summaries, redesign rationales, and long-form design documentation that does not belong in `design-studio.yaml`.

## Ruleset Management Flow Summary

When the user asks to create or extend audit rulesets, read `references/ruleset-management.md` for the full protocol. Key steps:

1. **Create ruleset**: Call `rulesets_create` with a name, description, and an array of rule objects. Do not pass `citationUrl` unless the user explicitly requests it.
2. **Add rules**: Call `rulesets_rules_add` to append individual rules to an existing ruleset.
3. **Read rules**: Call `rulesets_rules_get` to inspect the rules in a ruleset.
4. **Link to application**: Use `applications_create` or `applications_yaml_sync` to associate the ruleset with an application.
5. **Document in design journal**: After ruleset work is complete, create a design journal entry summarizing the rules added, rationale, and linked application.

Every ruleset automatically includes the design-token compliance rule. Rules must have `id`, `title`, `severity` (`critical` | `high` | `medium` | `low`), `category`, and `guidance`.

### Ruleset Modification Limits

The MCP does not expose tools to edit or remove individual rules from an existing ruleset. To change existing rules:
- **Append only**: Use `rulesets_rules_add` to add new rules.
- **Full replacement**: Modify the ruleset in `design-studio.yaml` and call `applications_yaml_sync` with user approval. This replaces the entire ruleset content.
- **No partial edits**: Do not attempt to call non-existent tools like `rulesets_rules_update` or `rulesets_rules_remove`.

## Website Exploration Flow Summary

When the user asks to create artifacts from an existing website using Chrome DevTools MCP, read `references/website-exploration.md` for the full protocol. Key steps:

1. **Validate input URL** — Ensure URL is present and valid.
2. **Open and map site structure** in Chrome DevTools MCP.
3. **Extract design primitives** — colors, typography, spacing, radius, shadows, borders. Extract exact source color tokens wherever possible (variables/aliases/literals) and avoid approximation when exact values exist.
4. **Extract component patterns and states** — buttons, inputs, cards, modals, navigation, etc.
5. **Normalize findings** into structured preset-ready data, preserving exact extracted color tokens.
6. **Propose preset options and ask for preset name** (blocking question).
7. **Run a mandatory self-review** before artifact creation: confirm typography visibility/contrast in light theme (especially secondary text), confirm typography-color harmony, and block white/near-white secondary text on light backgrounds.
8. **Create preset/application artifacts only** through Chai Studio MCP tools (no ruleset extraction in this workflow).
9. **Document in design journal** after exploration is complete.

Keep exploration efficient and repeatable: same route order, same extraction checklist, same normalization schema for each run.

## Project Spec Execution Protocol

To follow Chai Studio project specifications effectively, treat project rules as a layered contract:

1. In a local codebase, run the setup/sync gate, then read application-level `design-studio.yaml` and extract hard constraints. In non-code MCP clients, resolve the app with `applications_get`, then fetch hard constraints with `applications_yaml_get` and `rulesets_get`.
2. Read configured rulesets via `rulesets_get({ applicationId })` and map rule IDs/categories/severities.
3. Read configured preset via `presets_get` for token-level styling guidance.

Priority order when rules conflict:

1. Safety and explicit user instruction
2. Chai Studio `design-studio.yaml` hard constraints
3. Chai Studio ruleset severity and guidance
4. Preset/token conventions
5. Explicit user preferences for this task

When producing findings or fixes:

- Keep changes minimal and targeted; do not refactor unrelated surfaces.
- Provide exact path + line/snippet + impact + concrete fix.
- Prioritize critical accessibility, keyboard, focus/dialog, and metadata correctness issues before lower-severity visual polish.
- Avoid introducing prohibited patterns from project specs (for example unnecessary decorative motion or forbidden visual cliches).

## Font Asset Requirement

When implementing or auditing typography, always resolve each configured or chosen font family against Google Fonts first. If the family is available there, download the required font files into the codebase (for example `public/fonts/`, `src/assets/fonts/`, or the repo's established font asset directory) and wire the app to load those local files with the framework's local font API or `@font-face`.

If a required font is not available on Google Fonts, look for the closest suitable alternative, use the best fit for the Chai Studio design context, and document the fallback in the completion report. Use `fonts_list` to discover supported fonts.

## Deterministic Font Fallback Contract

When extracted fonts are unavailable in Chai/platform context:

1. Match by category first: serif, sans-serif, monospace, display.
2. Match by x-height/shape impression and weight coverage.
3. Prefer widely available open/platform fonts.
4. Keep a deterministic priority order per category (example: sans -> Inter -> Noto Sans -> Arial).
5. Report fallback rationale explicitly for each substituted family.

Never silently replace fonts.

## Audit And Fix Boundaries

Auditing and fixing are separate flows. Read `references/audit.md` for the full protocol.

- **Audit request** ("audit this", "find issues", "check compliance") → inspect, reconcile prior violations, validate, upload, report. Does NOT authorize code/UI changes.
- **Fix/remediation request** ("fix findings", "remediate", "resolve violations") → implement selected fixes, re-validate, update violation statuses.
- Only fix audit findings when the user explicitly asks to fix, remediate, resolve, close, or update those violations.

### Audit Standards Summary

Audit workflow is mandatory and always two-phase:
1. **Reconcile previous audits first** — check prior runs, mark verified fixes as resolved.
2. **Start and upload a new audit run** — one run per configured ruleset.

Key requirements:
- Audit every relevant page and component in scope.
- Granular findings per page/component. No vague aggregate notes.
- Include exact context: page/screen, component, file path, line/snippet.
- Component-level adherence required: border radius, spacing, typography, tokens, shadow/border, states.
- Each violation needs `aiFixPrompt` (self-contained for another agent).
- Keep per-run fingerprint set to skip duplicates before upload.
- If multiple `auditRuleSetIds`, audit each ruleset separately.
- Run project build before final audit upload when feasible.
- If browser MCP available, ask user if they want runtime validation.

### Status Updates And Fix Flow Summary

Only mark a violation resolved after verifying the code or UI changed:
- `resolved`: fix verified.
- `ignored`: accepted exception or out of scope.
- `false_positive`: finding is incorrect.

Fix workflow runs ONLY when user explicitly asks to fix, remediate, resolve, close, or update audit entries.

## Preset & Palette Creation

Before creating or updating a preset, call `fonts_list` to see all available fonts, then choose the families closest to the user's description. Only fonts returned by `fonts_list` are supported. Do not invent font names.

When creating only a preset (without an application), always use `presets_create` instead of `applications_yaml_create`.

When creating only a ruleset, always use `rulesets_create` instead of `applications_yaml_create`.

### Color Palette Rules

Every mode defines exactly 5 hex colors: `primary`, `secondary`, `tertiary`, `neutral`, `text`.

- **Light mode** = dark text on light surfaces. Background is hardcoded `#ffffff`.
- **Dark mode** = light text on dark surfaces. Background is `neutral`.
- `primary`: dark in light mode (headings on white), light in dark mode (headings on dark bg).
- `secondary`: medium-dark muted text in light mode, medium-light in dark mode.
- `tertiary`: accent/CTA color. Must be visible against the mode background.
- `neutral`: very light surface in light mode (e.g. `#F7F5F2`), very dark background in dark mode (e.g. `#15171A`).
- `text`: dark body text in light mode, light body text in dark mode. Usually identical to `primary`.

Bad examples:
- Light mode `text="#fffacd"` (light yellow on white = invisible)
- Light mode `neutral="#1a1a1a"` (dark card surfaces in light mode)
- Dark mode `text="#1a1a1a"` (dark text on dark bg = invisible)
- Dark mode `neutral="#f8fafc"` (blinding light background in dark mode)

Good examples:
- Light: `{primary:"#1A1C1E", secondary:"#6C7278", tertiary:"#B8422E", neutral:"#F7F5F2", text:"#1A1C1E"}`
- Dark: `{primary:"#F7F5F2", secondary:"#9A9CA0", tertiary:"#E2624A", neutral:"#15171A", text:"#F7F5F2"}`

Always supply BOTH light and dark modes. If only one is supplied, the other is cloned and will likely break contrast.

## Canvas Page Creation

### Theme Switch Compatibility

The canvas has a platform-level light/dark toggle. Generated pages MUST work with it:

- Keep JSX theme-neutral; do not generate separate light/dark markup trees.
- Canvas theme button controls dark mode via `html.dark` class and preset-mapped CSS variables.
- The `<html>` element gets `class="dark"` when dark mode is active; CSS variables automatically switch via the `.dark` selector.
- ALWAYS use CSS variables for colors (`var(--color-bg)`, `var(--color-fg)`, `var(--color-border)`) — NEVER hardcode colors.
- Prefer semantic Tailwind classes: `bg-background`, `text-foreground`, `border-border`, `text-muted-foreground`, `bg-primary`, `text-primary-foreground`.
- Use canvas shared css only for specific overrides, not full theming.

### Page Layout Positioning

Use the `layout` field to place page cards on the canvas:

- Pass `{ x: <number>, y: <number> }` to set the page card position in pixels.
- Example linear flow: page 1 at `{ x: 80, y: 60 }`, page 2 at `{ x: 480, y: 60 }`, page 3 at `{ x: 880, y: 60 }`.
- If `layout` is omitted, the page is auto-placed which may create spaghetti.
- To reorganize an existing canvas, call `canvases_pages_list` to read current positions, then `canvases_pages_update` with new `layout` values.
- Do not recreate an entire canvas just to fix layout — update existing pages via `canvases_pages_update`.

### Generation Status

Set `generationStatus` to `started` or `generating` while working, and `done` when complete.

## Safety

- Do not create dummy audits unless the user asks for test data.
- Do not create or update local design-spec mirror files (`DESIGN.md`, `DESIGN-RULES.md`, `design.yaml`, or alternatives) as part of this skill. Keep only `design-studio.yaml` synced from Chai Studio.
- Do not migrate UI libraries, rewrite the design system, or refactor broad UI areas just to satisfy an audit.
- Prefer Chai Studio IDs and tokens over memory or guesses.
