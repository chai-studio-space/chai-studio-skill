# Canvas Workflow

Use this workflow when the user asks to plan, wireframe, or design a multi-page flow using Chai Studio canvases.

**Prerequisite:** For local codebase work, run `references/sync.md` first. For non-code MCP clients without filesystem access, resolve the application with `applications_get` and fetch context with `applications_yaml_get` and `rulesets_get` instead.

## REQUIRED 3-Step Canvas Workflow

### Step 1 — DISCOVER Components

Call `canvas_runtime_read` to inspect the actual canvas UI runtime source code (`src/lib/shadcn/components/canvas-runtime.tsx`) and understand exactly which components exist and their props. This is the ULTIMATE source of truth.

For a quick summary, also call `canvas_components_list` or `canvas_components_search`.

### Step 2 — PLAN the Design

Decide which `UI.*` components you will use and how they compose. Choose layout utilities (`.stack`, `.row`, `.grid`, `.card`, `.container`) for structure. Plan color tokens (`var(--color-bg)`, `var(--color-fg)`, etc.). Do NOT skip planning.

### Step 3 — CREATE / UPDATE Pages

- Create or select a canvas with `canvases_create`
- Call `canvases_base_css_get({ canvasId })` to get `presetMappedCss` — do NOT call `canvases_css_update` unless absolutely necessary (presetMappedCss already includes all required tokens)
- For multi-page flows: FIRST create ALL empty pages with `generationStatus: "started"`, THEN add links with `canvases_pages_links_create`, THEN generate each page's UI one by one updating `generationStatus` from `"generating"` to `"done"`
- For single pages: create directly with `generationStatus: "done"` and full `componentTsx`

## Canvas Rules

- Canvas workflow is 3 steps in order: (1) DISCOVER components by reading the canvas runtime source with `canvas_runtime_read` — this is the ultimate source of truth for exact component signatures. Use `canvas_components_list` / `canvas_components_search` for quick summaries only. (2) PLAN the design — decide which `UI.*` components you will use before writing code. (3) CREATE / UPDATE pages.
- For multi-page flows, ALWAYS use this sequence: (a) Create ALL empty pages first with `generationStatus: "started"` and empty `componentTsx`, (b) Add all page-to-page links with `canvases_pages_links_create`, (c) Generate each page one by one by updating `componentTsx` and setting `generationStatus` to `"generating"` then `"done"`. This lets the user see the full page structure and navigation immediately.
- Canvas pages must be responsive across mobile, tablet, and desktop widths.
- All `UI.*` components are theme-aware via CSS variables. Do NOT handle light/dark manually. Use `UI.*` components directly and they adapt automatically.
- Keep page TSX theme-neutral. Canvas theme mode controls light/dark rendering.
- Use `presetMappedCss` from `canvases_base_css_get` as the base CSS. Do NOT call `canvases_css_update` unless absolutely necessary — presetMappedCss already includes all required tokens and bundled components are self-contained.
- For in-progress canvas work, provide progress updates through `generationStatus` on page create/update: `started` or `generating` while working, `done` only after completion.
- When editing pages, use `generationStatus` `started` or `generating` during work and `done` only after the edit is complete.

## Fix Canvas Mode (`/chai fix-canvas`)

Use `canvases_list`, `canvases_pages_list`, and `canvases_annotations_get` with `status: "open"`. Update affected pages, then mark addressed annotations fixed with `canvases_annotations_status_update`.