# Chai Studio MCP Workflows

This reference defines repeatable flows for website exploration with Chrome DevTools MCP and Chai artifact creation.

## 1) MCP Availability Gate (Always First)

1. Verify Chai Studio MCP is available.
2. Verify Chrome DevTools MCP is available.
3. If Chrome DevTools MCP is missing, explicitly ask the user to install/enable it and pause.
4. Continue only when both MCPs are available.

## 2) Website Exploration Workflow (Preset-Oriented)

Use this when user provides a website URL and wants Chai artifacts.

1. Validate URL and reachability.
2. Explore top-level IA and route templates in Chrome DevTools MCP.
3. Capture design primitives:
   - color and semantic usage
   - typography system
   - spacing/radius/border/shadow systems
4. Capture components and states:
   - core components and their variants
   - hover/focus/active/disabled/loading/error/empty/selected states
   - interaction and responsive patterns
5. Normalize findings into a structured JSON model for preset creation.
6. Ask for preset name (blocking) with suggestions.
7. Call `create_preset` with JSON representation of design YAML.

## 3) Artifact Creation Contracts

### create_preset

- Input must be JSON representation of design YAML (not raw YAML string).
- Build JSON from exploration findings:
  - `tokens` (color/typography/spacing/radius/shadow/border)
  - `components`
  - `states`
  - `metadata` and source notes
- Ask for preset name before create, provide suggestions.

### create_ruleset

- Supports standalone creation.
- Use directly when user asks for ruleset-only flow.

### add_rule_to_ruleset

- Attaches existing rule to a `rulesetId`.
- Discover rules with pagination before attaching.

### create_application

- Must include `presetId` plus `rulesetId`(s).
- Resolve/select IDs via list discovery and confirm if ambiguous before calling.

## 4) Pagination Guidance (Mandatory)

Default list pagination is 20 items/page for:

- applications
- presets
- rulesets
- rules

Agents must iterate all required pages before concluding “not found”.

## 5) Font Fallback Guidance

If extracted font is unavailable:

1. Match family category (sans/serif/mono/display).
2. Match weight and style coverage.
3. Pick closest open/platform font using deterministic priority.
4. Report fallback decision and rationale in output.

## 6) Prompting Guidance

- Ask only blocking questions.
- Prefer tool discovery over user questions when MCP can provide the answer.
- Use exact tool names; no aliases.
