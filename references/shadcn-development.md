# Shadcn Development Workflow

Use this workflow when the user asks to build real UI components in the codebase (not canvas planning).

**Prerequisite:** For local codebase work, run `references/sync.md` first.

## MCP Tool Sequence

1. `shadcn_css_get({ applicationId })` — obtain preset-mapped CSS tokens
2. `shadcn_component_list({ applicationId })` — inspect available themed components
3. `shadcn_component_get({ applicationId, componentName })` — get themed component TSX
4. Integrate returned TSX into product code paths (`components/ui/...`) following repository conventions
5. Validate build/tests when available