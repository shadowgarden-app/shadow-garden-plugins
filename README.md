# shadow-garden-skill-official

Official plugins and skills for [bapbong](https://bapbong.app), the `.docx`
editor by Shadow Garden. Everything here talks to the **running bapbong app**
— the user's own editor — and goes through the permission the user set per
folder (off / read-only / ask / auto). Nothing edits a file behind their back.

| Plugin | What it gives an agent |
|---|---|
| `bapbong` | the `bapbong` command line as a skill (read, search, edit, create, move, delete, render pages), and the bapbong MCP server |

Requires the bapbong app to be running. On a Mac the app installs the
`bapbong` command itself; the skill carries a Node bundle of the same command
for a shell that has no `bapbong` on its PATH (Cowork's sandboxed shell, a
VM, a remote session).

## Claude Code

```
/plugin marketplace add shadowgarden-app/shadow-garden-skill-official
/plugin install bapbong@shadow-garden
```

That gives the skill and the MCP server (`bapbong` itself is on the PATH
because the app put it there when it launched).
MCP only, without the plugin:

```
claude mcp add bapbong -- bapbong mcp
```

## Claude Desktop

**Skill** — Settings › Skills › Browse › Plugins › + › Add from a repository, paste
`shadowgarden-app/shadow-garden-skill-official`, then install **bapbong**
from the shadow-garden marketplace. (No git? Download `bapbong-skill.zip`
from the [latest release](https://github.com/shadowgarden-app/shadow-garden-skill-official/releases/latest)
and use Customize › Skills › + › Upload a skill.)
Cowork runs the skill on the Mac, where it finds the running app by
itself. If your organization enforces Cowork's full VM sandbox, bapbong
notices the VM and opens a door to it while it runs, through the folder you
have open — nothing to turn on.

**MCP** — Settings › Developer › Edit Config, merge (Claude Desktop starts
servers with a bare PATH, so use the full path of the launcher the app
keeps at `~/Library/Application Support/bapbong/bin/bapbong` — bapbong's
Settings › AI agents › How to connect shows the snippet with your path):

```json
{ "mcpServers": { "bapbong": { "command": "/Users/<you>/Library/Application Support/bapbong/bin/bapbong", "args": ["mcp"] } } }
```

## Codex

**Skill** — copy the skill folder into Codex's user skills directory:

```
git clone --depth 1 https://github.com/shadowgarden-app/shadow-garden-skill-official /tmp/sgs \
  && mkdir -p ~/.agents/skills && cp -R /tmp/sgs/plugins/bapbong/skills/bapbong ~/.agents/skills/bapbong
```

**MCP** — `codex mcp add bapbong -- bapbong mcp`, or in `~/.codex/config.toml`:

```toml
[mcp_servers.bapbong]
command = "bapbong"
args = ["mcp"]
```

## Layout

```
.claude-plugin/marketplace.json     the marketplace (name: shadow-garden)
plugins/bapbong/
  .claude-plugin/plugin.json        the plugin
  .mcp.json                         MCP server: scripts/bapbong mcp
  scripts/bapbong                   the command for .mcp.json and for shells without `bapbong`
  skills/bapbong/SKILL.md           the skill
  skills/bapbong/scripts/           the command, bundled for Node
```

The skill's source of truth is the bapbong desktop app's repository; this
repository is the distribution.

There is deliberately no top-level `bin/` in the plugin: claude.ai rejects a
plugin that has one (marketplace sync and upload alike), which would leave it
usable from Claude Code only. On the Mac the app itself puts `bapbong` on the
PATH; elsewhere the skill calls `scripts/bapbong` by path.
