# Changelog

## 0.2.0 — 2026-09-07

- `new` and `insert` take structured content (`--content-file blocks.json`): headings, Title/Subtitle, alignment, tab stops with dot/underscore leaders, tables with column widths, `grid`/`outer`/`none` borders, header rows, merged cells, cell shading. Plain text still works as before.
- `render` returns the first pages as images inside the MCP result, so the model sees the page without opening a file; PNG files are still written.
- `table <n>`: edit an existing table — insert/delete rows, merge cells, column widths, borders, header row, alignment (`edit_table` for MCP clients; `cat` marks cell blocks with `table: { index, row, cell }`). `format` addresses a whole block (`--block_index`) and sets heading, style, font size and tab stops.
- SKILL.md rewritten: footguns first (no tables drawn with characters, no space padding, no typed leaders, no fake headings), a verify step with criteria.
- Requires bapbong 0.35 or newer.

## 0.1.0 — 2026-09-04

- First release: the `bapbong` plugin — skill + command line (Node bundle) + MCP server.
- No top-level `bin/` (claude.ai rejects plugins that ship one — the plugin showed up as Claude Code only); the command lives at `scripts/bapbong`.
