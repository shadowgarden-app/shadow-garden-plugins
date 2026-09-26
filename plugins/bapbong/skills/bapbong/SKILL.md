---
name: bapbong
description: "Use this skill to read, edit, create, search, move or delete Word documents (.docx) THROUGH THE RUNNING BAPBONG APP — the user's own editor — instead of manipulating .docx files directly. Triggers: the user has bapbong open, mentions a folder open in bapbong, or wants a document created/edited where they can see and undo the change (letters, forms, reports, anything with headings and tables). Every action goes through the app's permission gate (off / read-only / ask / auto, set per folder by the user): at 'ask' your changes wait in the app for the user's review, so tell them what you did. If this session also has the bapbong MCP tools (app_status, get_document, replace_text, …), prefer them: they reach the app from anywhere, including a cloud or sandboxed session where this shell cannot. If `bapbong status` exits 3 or 4, that means THIS SHELL cannot reach the app — not that the app is closed — so switch to the MCP tools; only when there are none, ask the user to open bapbong (never edit the file directly without telling them, or use the docx skill on a copy)."
---

# bapbong — documents through the user's editor

bapbong is the user's .docx editor. This skill drives it: the same commands an
MCP client gets, from a shell. Nothing here touches a file directly — the app
does, under the permission the user set for that folder.

> Script paths are relative to this skill's directory. `scripts/bapbong` is the
> app's own command line (bundled for Node); when `bapbong` is on the PATH it
> is the same command. Below, `bapbong …` means `scripts/bapbong …`.

## First command

```bash
scripts/bapbong status
```

Exit 0 with a folder list: connected. Exit 3 or 4: THIS SHELL cannot reach
the app. That is the normal state of a cloud or sandboxed session — the app
runs on the user's Mac, and a shell that is not on that Mac only reaches it
through a folder the user opened in bapbong AND mounted here. It does not
mean bapbong is closed. Do this, in order:

1. If the session has the bapbong MCP tools (`app_status`, `get_document`,
   `replace_text`, …), use them for everything below instead of this shell.
   They are the same commands, and they reach the app from anywhere.
2. Otherwise, tell the user: "I can't reach bapbong from here. Open bapbong
   on your Mac and make sure it is running; if this is a cloud session, add
   the bapbong plugin's tools or work from a local session." Then stop.

Never conclude "bapbong is not running" from an exit code alone.

## Task → command

| Task | Command |
|---|---|
| What the user has open — folders, documents on their own, what you may do | `bapbong status` |
| Every command with its arguments (read before guessing flags) | `bapbong tools` |
| List a folder / every document | `bapbong ls [folder]` · `bapbong docs` |
| Read a document as numbered blocks (table cells are blocks too; headers and footers come as `chrome`, read-only) | `bapbong cat <doc>` (omit `<doc>` for the one the user has open) |
| Find text across everything open / inside one document | `bapbong search "<text>"` · `bapbong find "<text>" --doc <doc>` |
| Replace text, formatting kept | `bapbong replace "<old>" "<new>" --doc <doc>` |
| Add plain paragraphs | `bapbong insert "<line1>\n<line2>" --after "<anchor>" --doc <doc>` (or `--before`, `--end`) |
| Add headings, lists, tables, fill-in lines | write blocks as JSON, then `bapbong insert --content-file blocks.json --end --doc <doc>` |
| Create a document | `bapbong new <folder>/<name>.docx --content-file blocks.json` (or `--content "<text>"`) |
| Remove whole paragraphs (an empty `replace` only empties one) | `bapbong block-rm <block> --doc <doc>` · `--count 3` for several in a row |
| Bold / italic / size / alignment of existing text | `bapbong format "<text>" --bold --font_size 12 --align center --doc <doc>` |
| Link text to a web page, an e-mail address or a bookmark (`cat` lists each block's `links`) | `bapbong format "<text>" --link https://example.com --doc <doc>` · `--link "mailto:…"` · `--link "#Bookmark"` · `--link none` unlinks |
| Space above/below, line spacing, indents of a paragraph | `bapbong format --block_index <n> --space_after 6 --line_spacing 1.15 --doc <doc>` · `--line_spacing 18pt` (exact) · `--indent_left 1 --first_line 0.5` (cm) · `--hanging 1` · `0` removes |
| Font, colour, highlight, super/subscript, or back to plain | `bapbong format "<text>" --font Georgia --color "#C00000" --highlight "#FFFF00" --doc <doc>` · `--vertical_align superscript` · `--clear_formatting` (links stay) |
| The document's own paragraph styles, and applying one (its look and its name for Word) | `bapbong styles <doc>` · `bapbong format --block_index <n> --style "Quote" --doc <doc>` · a block's `"style": "Quote"` |
| Make a paragraph a heading / add tab stops (by block number from `cat`) | `bapbong format --block_index <n> --heading 2 --doc <doc>` · `--tabs-file stops.json` |
| Turn paragraphs into a list, nest an item, end the list (`cat` shows `list: { kind, level }`) | `bapbong format --block_index <n> --list number` (one call per paragraph, top to bottom — each joins the list above it) · `--list_level 2` · `--list none` |
| Change an existing table (`cat` shows `table: { index, row, cell }` on its cells) | `bapbong table <index> --rows-file rows.json` (append; `--at <row>` inserts) · `--delete_rows 2,3` · `--insert_columns <at>[,<count>]` · `--delete_columns 1` · `--merge <row>,<from>,<to>` · `--widths 10%,60%,30%` · `--borders none` · `--header` · `--delete_table` (the whole table) |
| A footnote on a word or sentence (`cat` lists them under `footnotes`) | `bapbong footnote "<the text it follows>" "<the note>" --doc <doc>` · `--content-file note.json` for formatting · numbers follow the document |
| A table of contents (headings must be real headings) | `bapbong toc --after "<title text>" --title Contents --doc <doc>` · `--levels 2` · the result says whether page numbers are `updated` or `pending` (bapbong fills them when the user updates the TOC, Word when it opens the file) |
| Change a header or footer (`cat` lists them under `chrome`; `{page}` is the page number) | `bapbong footer --old_text "Draft" --new_text "Final" --doc <doc>` · `bapbong footer --content-file f.json` where a block holds `{ "field": "page" }` / `{ "field": "pages" }` · `--section 2` for one section · `bapbong header --content "Acme Ltd"` |
| Page orientation, paper, margins, columns — one section or all | `bapbong page --orientation landscape --paper A4 --margins narrow --doc <doc>` · `--margins 2,2.5,2,2.5` (cm: top,right,bottom,left) · `--columns 2` · `--section 2` |
| Landscape pages inside a portrait document | `bapbong page --section_break_after <block>` before them and after them (one call each; `cat` then shows each block's `section`), then `bapbong page --section <n> --orientation landscape` · `--remove_section_break <n>` undoes a break |
| Resize / rotate a picture | `bapbong image <block> --width 300 --doc <doc>` |
| Put a picture in (a file you made, or one you draw as SVG) | `bapbong image-add <picture.png> --position document_end --doc <doc>` · `--svg-file diagram.svg --position after --anchor_text "<text>"` |
| Swap one picture for another, keeping the layout around it | `bapbong image-set <block> <picture.png> --doc <doc>` · `--svg-file diagram.svg` |
| Remove a picture | `bapbong image-rm <block> --doc <doc>` |
| New folder / move / rename / delete | `bapbong mkdir <path>` · `bapbong mv <from> <to>` · `bapbong rm <path>` |
| Show the user a document (a file on disk; a new document waiting for review is opened by the user from the review list) | `bapbong open <doc>` |
| Look at the pages yourself | `bapbong render <doc> --pages 1-2` → PNG files, view them |
| Page count, page size, sections | `bapbong check <doc>` |
| What is waiting for the user | `bapbong pending` |

Output is JSON when piped, a short rendering on a TTY; `--json` forces JSON.
Exit codes: 0 ok · 1 wrong usage · 2 the app refused (the message says why) ·
3 app not running · 4 could not reach the app. `--<arg>-file <path|->` passes
any argument as JSON from a file or stdin.

## Content: text or blocks

`content` (for `new` and `insert`) is either plain text — one paragraph per
line — or an array of blocks. A block is a string (a paragraph) or one of:

```
{ "paragraph": text | inlines, "heading": 1-6, "style": "Title"|"Subtitle",
  "align": "left"|"center"|"right"|"justify",
  "tabs": [{ "at": cm | "100%", "align": "right"|"center", "leader": "dot"|"underscore" }],
  "pageBreakBefore": true, "list": "bullet"|"number", "level": 1-3,
  "spaceBefore"/"spaceAfter": pt, "lineSpacing": 1.15 | { "exact": pt },
  "indent": { "left", "right", "firstLine", "hanging": cm }, …format }
{ "table": [[cell, …], …], "widths": [cm | "%", … one per column],
  "borders": "grid"|"outer"|"none", "header": true, "align": "center" }
```

Inlines: `"text"` · `{ "text": "…", "link": "https://…", …format }` (link is
optional) · `{ "tab": true }` · `{ "field": "page" }` / `{ "field": "pages" }`
(page number / count, for headers and footers), where
format is any of `"bold"`/`"italic"`/`"underline"`/`"strike"`/
`"superscript"`/`"subscript"`: true, `"color"`/`"highlight"`: `"#RRGGBB"`,
`"font"`: a name, `"size"`: points. A paragraph block or a cell takes the same
format for all its text. A cell: text, inlines, or
`{ "text": …, "colspan": n, "align": …, "shading": "#RRGGBB", …format }`.
Every row must cover the same number of columns (merge with `colspan`).

The two shapes you will need most:

```json
{ "paragraph": ["Name: ", { "tab": true }],
  "tabs": [{ "at": "100%", "align": "right", "leader": "dot" }] }
```
```json
{ "table": [["Left column", "Right column"], ["", ""]], "borders": "none" }
```

## Do not (the app cannot fix these afterwards)

- **Never draw a table, a box or a rule with characters** (`┌─┬│`, `|---|`,
  `+---+`). Use a table block. Word cannot edit a picture made of text.
- **Never line things up with spaces**, and never type `........` or `_____`
  for a fill-in line. Two things side by side (a signature area, a label and a
  value) are a two-column table with `"borders": "none"`; a fill-in line is a
  tab with a `leader`.
- **Never fake a heading** with bold + centered text. Use `"heading"`.
- **Never fake a footnote** with a superscript number and a line at the
  bottom: `bapbong footnote` makes one Word numbers and places.
- **Never type a table of contents.** `bapbong toc` makes a real one from
  the headings; typed dots and page numbers go stale with the first edit.
- **Use the document's own styles** (`bapbong styles`) before formatting by
  hand: a paragraph in "Quote" or a company style looks right and stays
  that style for whoever edits it next in Word.
- **No `\n` inside a paragraph** — one paragraph per block.
- **Never type `•`, `-` or `1.` to make a list.** Each item is a paragraph
  block with `"list": "bullet"` or `"number"`; consecutive items are one list
  and the app numbers them (a second numbered list starts at 1 again).
- **Never delete a picture and insert a new one in its place.** `image-set`
  keeps the anchor, the text wrap and the width; delete + add loses all three
  and moves the page. And check what you are replacing: `cat` reports each
  picture's `kind`, and `drawing` (art the file describes shape by shape) or
  `equation` becomes a flat picture that the user cannot edit again — say so
  before you do it.
- A picture must be a PNG, JPEG, GIF or BMP file inside a folder the user
  opened, or SVG you write (`--svg-file`, self-contained: no links out).
- `replace` needs the exact existing text, matched once — include surrounding
  words or pass `--occurrence n`. `cat` first.
- Give `expectedVersion` (the `docVersion` from your last `cat`) when several
  edits depend on each other; a stale version is refused, you re-read.
- Paths: use the ones you see (`ls` prints them). Do not try to approve
  anything; `pending` shows the queue, the user decides in the app.
- The workspace is what the user can see in bapbong: the documents in their
  open folders AND the ones they opened on their own. `docs` and `search`
  cover both. A document opened on its own is in no folder, so `ls`, `mv` and
  `new` inside it are refused — read, edit and `rm` work as usual.

## How the permission gate shapes your work

The user sets ONE level for their whole workspace — every folder and every
document they opened on its own; `status` shows it.

- **read** — reads and searches only. Every mutation is exit 2; do not retry,
  tell the user which folder is read-only.
- **ask** (default) — an edit to the document the user has OPEN lands in their
  editor at once (they see it, ⌘Z undoes it). Anything else — another
  document, a new document, a folder, a move, a delete — is **held in the app
  for review**: the result says `"status": "pending"`, nothing is on disk yet.
  Keep editing a pending new document by its path; `render` and `check` work
  on it too (they use its pending version), only `open` does not. The user
  saves it once. Say what you asked for and why.
- **auto** — everything happens at once, each with an undo in the app.

Expect refusals: a document open in a tab cannot be deleted; a document with
unreviewed changes cannot be moved; a path the user has not opened "does not
exist"; a path already waiting for review is refused until the user decides.

## Verify — every time you changed layout

1. `bapbong cat <doc>`: the text is where you put it, cells are separate
   blocks.
2. `bapbong render <doc> --pages 1-2`, then **look at the PNG files** (an MCP
   client gets the pages inline). This works on a document still waiting for
   the user's review, so verify before you report. Check: tables are real tables with straight
   column lines, nothing is drawn with characters, side-by-side text is
   aligned, fill-in lines are dotted leaders, headings look like headings,
   nothing spills onto an unexpected page.
3. `bapbong check <doc>` for the page count.

If anything is off, fix it and render again. Do not report done on a page you
have not looked at.

## Files

- `scripts/bapbong` — the command (a wrapper over `scripts/bapbong.cjs`, the
  app's CLI bundled for Node; needs `node` on the PATH, nothing else).
- Discovery: `BAPBONG_HOST_JSON=<file>` overrides; otherwise
  `.bapbong/host.json` in the current folder or above; on the Mac itself,
  `~/Library/Application Support/bapbong/mcp.json`.
