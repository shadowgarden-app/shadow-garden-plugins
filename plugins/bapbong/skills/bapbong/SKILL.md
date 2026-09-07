---
name: bapbong
description: "Use this skill to read, edit, create, search, move or delete Word documents (.docx) THROUGH THE RUNNING BAPBONG APP — the user's own editor — instead of manipulating .docx files directly. Triggers: the user has bapbong open, mentions a folder open in bapbong, or wants a document created/edited where they can see and undo the change (letters, forms, reports, anything with headings and tables). Every action goes through the app's permission gate (off / read-only / ask / auto, set per folder by the user): at 'ask' your changes wait in the app for the user's review, so tell them what you did. If bapbong is not running (`bapbong status` exits 3), do not fall back to editing the file directly without telling the user — ask them to open bapbong, or use the docx skill on a copy."
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

Exit 0 with a folder list: connected. Exit 3: the app is not reachable —
bapbong is not running, or the folder you are in is not open in it. Tell the
user exactly that, then stop.

## Task → command

| Task | Command |
|---|---|
| What is open, which folders, what may I do | `bapbong status` |
| Every command with its arguments (read before guessing flags) | `bapbong tools` |
| List a folder / every document | `bapbong ls [folder]` · `bapbong docs` |
| Read a document as numbered blocks (table cells are blocks too) | `bapbong cat <doc>` (omit `<doc>` for the one the user has open) |
| Find text across the folder / inside one document | `bapbong search "<text>"` · `bapbong find "<text>" --doc <doc>` |
| Replace text, formatting kept | `bapbong replace "<old>" "<new>" --doc <doc>` |
| Add plain paragraphs | `bapbong insert "<line1>\n<line2>" --after "<anchor>" --doc <doc>` (or `--before`, `--end`) |
| Add headings, tables, fill-in lines | write blocks as JSON, then `bapbong insert --content-file blocks.json --end --doc <doc>` |
| Create a document | `bapbong new <folder>/<name>.docx --content-file blocks.json` (or `--content "<text>"`) |
| Bold / italic / size / alignment of existing text | `bapbong format "<text>" --bold --font_size 12 --align center --doc <doc>` |
| Make a paragraph a heading / add tab stops (by block number from `cat`) | `bapbong format --block_index <n> --heading 2 --doc <doc>` · `--tabs-file stops.json` |
| Change an existing table (`cat` shows `table: { index, row, cell }` on its cells) | `bapbong table <index> --rows-file rows.json` (append; `--at <row>` inserts) · `--delete_rows 2,3` · `--merge <row>,<from>,<to>` · `--widths 10%,60%,30%` · `--borders none` · `--header` |
| Resize / rotate an image | `bapbong image <block> --width 300 --doc <doc>` |
| New folder / move / rename / delete | `bapbong mkdir <path>` · `bapbong mv <from> <to>` · `bapbong rm <path>` |
| Show the user a document | `bapbong open <doc>` |
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
  "pageBreakBefore": true, "bold"/"italic"/"underline": true }
{ "table": [[cell, …], …], "widths": [cm | "%", … one per column],
  "borders": "grid"|"outer"|"none", "header": true, "align": "center" }
```

Inlines: `"text"` · `{ "text": "…", "bold"/"italic"/"underline": true }` ·
`{ "tab": true }`. A cell: text, inlines, or
`{ "text": …, "colspan": n, "align": …, "bold": true, "shading": "#RRGGBB" }`.
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
- **No `\n` inside a paragraph** — one paragraph per block. No `•` typed by
  hand.
- `replace` needs the exact existing text, matched once — include surrounding
  words or pass `--occurrence n`. `cat` first.
- Give `expectedVersion` (the `docVersion` from your last `cat`) when several
  edits depend on each other; a stale version is refused, you re-read.
- Paths: use the ones you see (`ls` prints them). Do not try to approve
  anything; `pending` shows the queue, the user decides in the app.

## How the permission gate shapes your work

The user sets one level per folder; `status` shows it.

- **read** — reads and searches only. Every mutation is exit 2; do not retry,
  tell the user which folder is read-only.
- **ask** (default) — an edit to the document the user has OPEN lands in their
  editor at once (they see it, ⌘Z undoes it). Anything else — another
  document, a new document, a folder, a move, a delete — is **held in the app
  for review**: the result says `"status": "pending"`, nothing is on disk yet.
  Keep editing a pending new document by its path; the user saves it once.
  Say what you asked for and why.
- **auto** — everything happens at once, each with an undo in the app.

Expect refusals: a document open in a tab cannot be deleted; a document with
unreviewed changes cannot be moved; a path outside the open folders "does not
exist"; a path already waiting for review is refused until the user decides.

## Verify — every time you changed layout

1. `bapbong cat <doc>`: the text is where you put it, cells are separate
   blocks.
2. `bapbong render <doc> --pages 1-2`, then **look at the PNG files** (an MCP
   client gets the pages inline). Check: tables are real tables with straight
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
