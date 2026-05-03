# Journal Workflow

Use this workflow when the user asks to document design decisions, audit results, or project notes.

## MCP Tool Sequence

1. `journals_list` — discover seeded journals and entries
2. `journals_entries_create({ journalId, title, markdown })` — create entry
3. `journals_entries_update({ entryId, title, markdown })` — update existing entry