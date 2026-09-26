# Changelog

## 0.4.0 — 2026-09-26

- **Headers and footers:** `header` / `footer` (MCP `edit_header_footer`) rewrite one — `{ "field": "page" }` and `{ "field": "pages" }` are the page number and count — or replace text inside it, for one section or all. `cat` shows them as `chrome`, with `{page}` where the number goes.
- **Spacing and indents:** `format --space_before/--space_after` (pt), `--line_spacing` (1.15, `18pt`, `at least 12pt`), `--indent_left/--indent_right/--first_line/--hanging`; paragraph blocks take `spaceBefore`, `spaceAfter`, `lineSpacing`, `indent`.
- **The document's own styles:** `styles` (MCP `list_styles`) lists them; `format --style` and a block's `style` take any of them by name — the paragraph gets the style's look and keeps its name in Word. `Heading N` always works.
- **Table of contents:** `toc --after <text>` (MCP `insert_toc`) makes a real Word TOC from the headings; on the document the user has open the page numbers are filled in at once, otherwise Word updates them on open.
- **Footnotes:** `footnote <text> <note>` (MCP `insert_footnote`); `cat` lists them.
- Needs a bapbong app built with these commands (after 0.44.0).

## 0.3.0 — 2026-09-26

- An agent can now make **real lists** (`"list": "bullet" | "number"`, `"level"` in blocks; `format --list`, `--list_level`); `cat` shows each list item's kind and level. A second numbered list starts at 1 again; an item added right after a list joins it.
- `block-rm <block> [--count n]` (MCP `delete_block`) takes whole paragraphs away — an empty `replace` only empties one.
- `table <n>`: `--insert_columns at[,count]`, `--delete_columns`, `--delete_table`. Merged cells are handled on the grid; the table keeps its width.
- `page` (MCP `page_setup`): orientation, paper, margins, columns for one section or all, and section breaks — e.g. landscape pages inside a portrait document. `cat` marks each block with its section once there are several.
- `format`: `--font`, `--color`, `--highlight`, `--vertical_align`, `--clear_formatting` (links stay), `--link` (web, `mailto:`, `#bookmark`; `javascript:` and `file:` are refused). Blocks take the same formatting and `link` on an inline, a paragraph or a cell.
- `cat` lists each block's links, and the document's headers and footers as `chrome` (read-only).
- Pictures from a shell: `image-add`, `image-set`, `image-rm`, `--svg-file`.
- One permission covers the whole workspace — the folders open in bapbong and the documents opened on their own; `status` reports it.
- The MCP server now starts when Claude Desktop launches it with a bare PATH (a Node from Homebrew, Volta or nvm was not found, and Claude showed only "Connection closed"); with no Node at all it uses the bapbong app's own runtime.
- Fixed: `find` on a terminal printed `[undefined]` for every match; `status` on a terminal printed `undefined` for the folders.
- The new commands need a bapbong app built after 0.44.0; with an older one, `bapbong tools` shows what it offers.

## 0.2.2 — 2026-09-14

- The MCP server outlives the app: it starts, lists the tools and answers while bapbong is closed, and connects when the app opens. The skill no longer reads "cannot reach the app" as "the app is closed".

## 0.2.1 — 2026-09-08

- `render` and `check` work on a new document that is still waiting for the user's review (rendered from its pending version in a hidden editor; nothing is promoted). `open` on such a document now explains that the user opens it from the review list. Requires the matching bapbong app build.

## 0.2.0 — 2026-09-07

- Repository and marketplace renamed to `shadow-garden-plugins` (Anthropic's naming: `claude-plugins-official`, `knowledge-work-plugins`). The marketplace `name` must equal the repository name: Claude Desktop refreshes a marketplace added from a repository under the repository's name, so the earlier `shadow-garden` made its Update button fail (`NOT_REGISTERED`). Claude Code users re-add the marketplace and install `bapbong@shadow-garden-plugins`.
- `new` and `insert` take structured content (`--content-file blocks.json`): headings, Title/Subtitle, alignment, tab stops with dot/underscore leaders, tables with column widths, `grid`/`outer`/`none` borders, header rows, merged cells, cell shading. Plain text still works as before.
- `render` returns the first pages as images inside the MCP result, so the model sees the page without opening a file; PNG files are still written.
- `table <n>`: edit an existing table — insert/delete rows, merge cells, column widths, borders, header row, alignment (`edit_table` for MCP clients; `cat` marks cell blocks with `table: { index, row, cell }`). `format` addresses a whole block (`--block_index`) and sets heading, style, font size and tab stops.
- SKILL.md rewritten: footguns first (no tables drawn with characters, no space padding, no typed leaders, no fake headings), a verify step with criteria.
- Requires bapbong 0.35 or newer.

## 0.1.0 — 2026-09-04

- First release: the `bapbong` plugin — skill + command line (Node bundle) + MCP server.
- No top-level `bin/` (claude.ai rejects plugins that ship one — the plugin showed up as Claude Code only); the command lives at `scripts/bapbong`.
