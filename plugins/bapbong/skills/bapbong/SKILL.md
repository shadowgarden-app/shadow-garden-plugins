---
name: bapbong
description: "Use this skill to read, edit, create, search, move or delete Word documents (.docx) THROUGH THE RUNNING BAPBONG APP — the user's own editor — instead of manipulating .docx files directly. Triggers: the user has bapbong open, mentions a folder open in bapbong, or wants a document created/edited where they can see and undo the change. Every action goes through the app's permission gate (off / read-only / ask / auto, set per folder by the user): at 'ask' your changes wait in the app for the user's review, so tell them what you did. If bapbong is not running (`bapbong status` exits 3), do not fall back to editing the file directly without telling the user — ask them to open bapbong, or use the docx skill on a copy."
---

# bapbong — documents through the user's editor

bapbong is the user's .docx editor. This skill drives it: the same commands
an MCP client gets, from a shell. Nothing here touches a file directly —
the app does, under the permission the user set for that folder.

## First command

```bash
scripts/bapbong status
```

> Script paths are relative to this skill's directory. `scripts/bapbong` is
> the app's own command line (bundled for Node); on the user's Mac the same
> command is simply `bapbong`. Below, `bapbong …` means `scripts/bapbong …`.

`status` with exit 0 and a folder list means you are connected. Exit 3
means the app is not reachable. In the sandbox that means one of: bapbong
is not running; the folder you are in is not open in bapbong (the app
writes `.bapbong/host.json` into each open folder, which is how the command
finds it); or **Settings › AI agents › Claude Desktop sandbox** was set to
Off (Auto, the default, opens the door while Claude Desktop's VM runs).
Tell the user exactly that, then stop.

## Task → command

| Task | Command |
|---|---|
| What is open, which folders, what may I do | `bapbong status` |
| Every command with its arguments (read this before guessing flags) | `bapbong tools` |
| List a folder / every document | `bapbong ls [folder]` · `bapbong docs` |
| Read a document as numbered blocks | `bapbong cat <doc>` (omit `<doc>` for the one the user has open) |
| Find where text is, across the folder | `bapbong search "<text>"` (cheap: text only) |
| Find text inside one document | `bapbong find "<text>" --doc <doc>` |
| Replace text (formatting kept) | `bapbong replace "<old>" "<new>" --doc <doc>` |
| Add paragraphs | `bapbong insert "<line1>\n<line2>" --after "<anchor text>" --doc <doc>` (or `--before`, `--end`) |
| Bold / italic / alignment | `bapbong format "<text>" --bold --align center --doc <doc>` |
| Resize / rotate an image | `bapbong image <block> --width 300 --doc <doc>` |
| Create a document | `bapbong new <folder>/<name>.docx --content "<line1>\n<line2>"` |
| New folder / move / rename / delete | `bapbong mkdir <path>` · `bapbong mv <from> <to>` · `bapbong rm <path>` |
| Show the user a document (opens a tab) | `bapbong open <doc>` |
| Page count, page size, sections | `bapbong check <doc>` |
| Look at the pages yourself (PNG files, as bapbong renders them) | `bapbong render <doc> --pages 1-3` |
| What is waiting for the user | `bapbong pending` |
| Anything else in `tools` | `bapbong call <name> --<flag> <value> …` |

Output is JSON when piped (parse it), a short rendering on a TTY. Add
`--json` to force JSON. Exit codes: 0 ok · 1 wrong usage · 2 the app
refused (the message says why and what to do) · 3 app not running ·
4 could not reach the app.

## How the permission gate shapes your work

The user sets one level per folder in bapbong. `status` shows it.

- **read** — you can read and search. Every edit, create, move or delete is
  refused with exit 2. Do not retry; tell the user which folder is read-only.
- **ask** (default) — reads work. An edit to the document the user has OPEN
  lands in their editor at once (they see it, ⌘Z undoes it). An edit to any
  other document, a new document, a folder, a move or a delete is **held in
  the app for review** — the result says `"status": "pending"`. Nothing is on
  disk yet. Keep editing a pending new document by its path; the user saves
  it once. Say what you asked for and why, so they can decide.
- **auto** — everything happens at once, each with an undo in the app.

Refusals to expect: a document open in a tab cannot be deleted (ask the user
to close it); a document with unreviewed changes cannot be moved; a path
outside the open folders "does not exist"; a second request on a path
already waiting for review is refused until the user decides.

## Working rules

1. `cat` before you edit: `replace` needs the exact existing text, and it
   must match once — include surrounding words, or pass `--occurrence n`.
2. Pass `expectedVersion` (the `docVersion` from your last `cat`) on edits
   when several steps depend on each other; a stale version is refused and
   you re-read.
3. Paths: use the paths you see (`ls` prints them). They are translated to
   the Mac's paths for the app and back again in results.
4. One document, one writer: if a document is open in bapbong, your edits
   go through the user's editor; do not also rewrite the file yourself.
5. Do not try to approve anything. `pending` shows the queue; the user
   decides in the app.

## Verify

Read the document back (`cat`) after editing. For layout — page breaks,
where a table falls, an image's size — `bapbong render <doc> --pages 1-2`
writes PNG files rendered by the app itself (the paths come back in the
result; view them). `bapbong check <doc>` gives page count and geometry
without pictures. Both open the document in a tab for the user if it is not
open yet.

## Files

- `scripts/bapbong` — the command (a wrapper over `scripts/bapbong.cjs`, the
  app's CLI bundled for Node; needs `node` on the PATH, nothing else).
- Discovery: `BAPBONG_HOST_JSON=<file>` overrides; otherwise
  `.bapbong/host.json` in the current folder or above; on the Mac itself,
  `~/Library/Application Support/bapbong/mcp.json`.
