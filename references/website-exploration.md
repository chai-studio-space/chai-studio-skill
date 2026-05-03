# Website Exploration Workflow

Use this workflow when the user asks to create artifacts from an existing website or explore a website.

**Prerequisite:** Skip `references/sync.md` when extraction-only.

## MCP Tool Sequence

1. Skip sync when extraction-only
2. Run website exploration via Chrome DevTools MCP
3. If artifact creation/update is needed, run `references/sync.md` first, then continue

## Rules

- Extraction-only flow: run directly without sync required
- Artifact creation/update flow: run `/chai setup` first, then continue