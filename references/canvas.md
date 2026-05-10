# Canvas Workflow

Use this workflow when the user asks to plan, wireframe, or design a multi-page flow using Chai Studio canvases.

**Prerequisite:** For local codebase work, run `references/sync.md` first. For non-code MCP clients without filesystem access, resolve the application with `applications_get` and fetch context with `applications_yaml_get` and `rulesets_get` instead.

## MCP Boundary

Agents using this skill do not need access to the `chai-studio` repository. Use only Chai Studio MCP tool outputs for canvas runtime/component knowledge.

- `canvas_runtime_read` returns MCP-exposed canvas runtime API/source snippets for canvas authoring.
- It is not arbitrary filesystem access to the Chai Studio server codebase.
- Do not ask agents to inspect local `chai-studio` files directly.

## REQUIRED 3-Step Canvas Workflow

### Step 1 — DISCOVER Components And Icons

Call `canvas_runtime_read` to inspect the MCP-exposed canvas runtime API/source snippets and understand exactly which components exist and their props. This is the ULTIMATE source of truth for canvas authoring, but it is not general access to the Chai Studio repository.

For a quick summary, also call `canvas_components_list` or `canvas_components_search`.

Before using icons in page UI, page thumbnails, navigation, buttons, headers, or empty states, call `canvas_icons_list`, `canvas_icons_search`, or `canvas_icons_get`. Use returned `UI.CommonIcons.<Name>` / `UI.Icons.<Name>` names exactly.

### Step 2 — PLAN the Design

Decide which `UI.*` components, documented icons, safe SVG thumbnails, and page links you will use. Choose layout utilities (`.stack`, `.row`, `.grid`, `.card`, `.container`) for structure. Plan color tokens (`var(--color-bg)`, `var(--color-fg)`, etc.). Do NOT skip planning.

### Step 3 — CREATE / UPDATE Pages

- Create or select a canvas with `canvases_create`
- Call `canvases_base_css_get({ canvasId })` to get `presetMappedCss` — do NOT call `canvases_css_update` unless absolutely necessary (presetMappedCss already includes all required tokens)
- For multi-page flows: FIRST create ALL empty pages with `generationStatus: "started"`, empty `componentTsx`, meaningful inline `svg`, and non-overlapping `layout`, THEN add links with `canvases_pages_links_create`, THEN generate each page's UI one by one updating `generationStatus` from `"generating"` to `"done"`
- For single pages: create directly with `generationStatus: "done"`, full `componentTsx`, meaningful inline `svg`, and non-overlapping `layout`
- In canvas `componentTsx`, do not use imports, exports, TypeScript annotations, external libraries, real external navigation, or Tailwind utilities. Tailwind is not loaded in the canvas iframe.
- For interactive page navigation inside `componentTsx`, use `toCanvasPage("page-id")`, `href="toCanvasPage(page-id)"`, `href="canvas-page:page-id"`, or `href="#canvas-page:page-id"`, and also create/update visual flow links with `canvases_pages_links_create` / `canvases_pages_links_update`.

## Canvas Rules

- Canvas workflow is 3 steps in order: (1) DISCOVER components through MCP with `canvas_runtime_read` — this is the ultimate source of truth for exact canvas runtime signatures exposed by the server. Use `canvas_components_list` / `canvas_components_search` for quick summaries only, and discover icons with `canvas_icons_list` / `canvas_icons_search` / `canvas_icons_get`. (2) PLAN the design — decide which `UI.*` components, documented icons, SVG thumbnails, and links you will use before writing code. (3) CREATE / UPDATE pages.
- For multi-page flows, ALWAYS use this sequence: (a) Create ALL empty pages first with `generationStatus: "started"`, empty `componentTsx`, meaningful inline `svg`, and non-overlapping `layout`, (b) Add all page-to-page links with `canvases_pages_links_create`, (c) Generate each page one by one by updating `componentTsx` and setting `generationStatus` to `"generating"` then `"done"`. This lets the user see the full page structure and navigation immediately.
- Every canvas page requires a safe inline SVG thumbnail in the `svg` field. Use compact inline `<svg viewBox="0 0 64 64" ...>` markup matching the page purpose; do not include scripts, event handlers, `foreignObject`, external hrefs, images, data URLs, or javascript URLs. If an existing page lacks `svg`, include one on the next `canvases_pages_update`.
- Canvas pages must be responsive across mobile, tablet, and desktop widths.
- All `UI.*` components are theme-aware via CSS variables. Do NOT handle light/dark manually. Use `UI.*` components directly and they adapt automatically.
- Keep page TSX theme-neutral. Canvas theme mode controls light/dark rendering.
- Do not use Tailwind utilities in canvas `componentTsx`; use `UI.*`, inline styles with CSS variables, and base utility classes such as `.stack`, `.row`, `.grid`, `.card`, and `.container`.
- Use `presetMappedCss` from `canvases_base_css_get` as the base CSS. Do NOT call `canvases_css_update` unless absolutely necessary — presetMappedCss already includes all required tokens and bundled components are self-contained.
- For in-progress canvas work, provide progress updates through `generationStatus` on page create/update: `started` or `generating` while working, `done` only after completion.
- When editing pages, use `generationStatus` `started` or `generating` during work and `done` only after the edit is complete.

## Fix Canvas Mode (`/chai fix-canvas`)

Use `canvases_list`, `canvases_pages_list`, and `canvases_annotations_get` with `status: "open"`. Update affected pages, then mark addressed annotations fixed with `canvases_annotations_status_update`.
