---
name: chai-studio-skill
description: Use when a project is connected to Chai Studio MCP and the user wants website exploration via Chrome DevTools MCP plus Chai artifact creation (preset/application/ruleset), redesign workflows, or audits.
license: MIT
metadata:
  version: 1.1.0
  owner: akshayt
  short-description: Explore websites with DevTools MCP and create Chai artifacts
---

# Chai Studio MCP + Chrome DevTools MCP

Use this skill to make Chai Studio MCP the source of truth for artifact creation and to use Chrome DevTools MCP for evidence-driven website exploration.

## Mandatory MCP Availability Gate

Run this gate before artifact discovery or creation:

1. Verify Chai Studio MCP tools are available.
2. Verify Chrome DevTools MCP tools are available.
3. If Chrome DevTools MCP is missing, stop and tell the user exactly: they must install/enable Chrome DevTools MCP to continue website exploration for preset creation.
4. Resume only after both MCPs are available.

Do not proceed with website exploration without Chrome DevTools MCP.

## Prompting Contract

- Ask only blocking questions.
- If required data can be discovered via Chrome DevTools MCP or Chai Studio MCP, discover first and ask later.
- Keep tool names exact; do not alias or rename tools.

## Pagination Contract (Required)

Discovery/list operations are paginated with default 20 items per page.
This applies to: applications, presets, rulesets, and rules.

When searching for an artifact:

1. Iterate page-by-page until found or pages are exhausted.
2. Never conclude “not found” after only the first page.

## Website Exploration Workflow (User-Provided URL)

When the user asks to create artifacts from an existing website, follow this exact order:

1. **Validate input URL**
   - Ensure a URL is present and syntactically valid.
   - Ask the user only if URL is missing or unreachable.
2. **Open and map site structure in Chrome DevTools MCP**
   - Identify main routes/surfaces, header/footer/nav structure, primary flows, and key templates.
3. **Extract design primitives**
   - Colors/tokens: brand, neutrals, semantic states, opacity usage.
   - Typography: families, weights, sizes, line-heights, letter spacing, heading/body/label scales.
   - Spacing/radius/shadows/borders: spacing steps, container widths, corner radius system, elevation model, border usage.
4. **Extract component patterns and states**
   - Buttons, inputs, cards, modals, tables, navigation, badges, alerts, menus.
   - State behavior: hover/focus/active/disabled/loading/error/empty/selected.
   - Interaction patterns: transitions, motion, keyboard affordances, responsive breakpoints.
5. **Normalize findings into structured preset-ready data**
   - Convert extracted styles into a deterministic token model suitable for preset creation.
   - Keep raw evidence links/notes tied to each inferred token.
6. **Propose preset options and ask for preset name**
   - Suggest names from site brand/domain/style.
   - Ask user to choose or provide preset name (blocking question).
7. **Create artifacts through Chai Studio MCP tools per contracts below**.

Keep exploration efficient and repeatable: same route order, same extraction checklist, same normalization schema for each run.

## Tool Usage Contracts (Required)

### create_preset

- `create_preset` accepts **JSON representation of design YAML** (not a raw YAML string).
- Build this JSON from normalized website findings:
  - token groups (color, typography, spacing, radius, shadow, border)
  - component guidance
  - interaction/state guidance
  - metadata/source notes
- Before calling `create_preset`, ask user for preset name with 2–3 suggestions.

### create_ruleset

- `create_ruleset` supports standalone creation.
- Use when user wants rules without creating an application.
- Confirm scope/objective only when missing.

### add_rule_to_ruleset

- `add_rule_to_ruleset` attaches an existing rule to a target `rulesetId`.
- Resolve candidate rules through paginated rule discovery first.
- Confirm selected `rulesetId` only if ambiguous.

### create_application

- `create_application` accepts `presetId` and `rulesetId`(s).
- Before creation, resolve candidate preset/ruleset IDs via paginated listing.
- Show selected IDs and ask for confirmation only if there is ambiguity or competing matches.

## Deterministic Font Fallback Contract

When extracted font families are unavailable in Chai/platform context:

1. Match by category first: serif, sans-serif, monospace, display.
2. Match by x-height/shape impression and weight coverage.
3. Prefer widely available open/platform fonts.
4. Keep a deterministic priority order per category (example: sans -> Inter -> Noto Sans -> Arial).
5. Report fallback rationale explicitly for each substituted family.

Never silently replace fonts.

## Reference Documents

- For workflows and artifact creation playbooks: `references/workflows.md`
- For configuration and pagination notes: `references/config.md`
