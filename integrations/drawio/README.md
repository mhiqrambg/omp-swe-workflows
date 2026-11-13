# draw.io MCP setup

The design phase produces every technical diagram through a draw.io MCP server. OMP config for it lives in `integrations/drawio/mcp.json`.

Two official routes exist, and they are not interchangeable:

| Route | Package | Output | Use it when |
| --- | --- | --- | --- |
| MCP Tool Server | `@drawio/mcp` (stdio) | Opens the draw.io editor, reads and edits local `.drawio` files | You want native files under `design/diagrams/` |
| MCP App Server | `https://mcp.draw.io/mcp` (http) | Renders a diagram inline in chat | You only need a picture in the conversation |

The design agents need editable `.drawio` files, so the stdio Tool Server is the one to enable.

## Install the server

Node.js 20 or newer. No global install needed:

```sh
npx -y @drawio/mcp
```

The server opens diagrams through `app.diagrams.net`. Set `DRAWIO_BASE_URL` for a self-hosted draw.io.

## Enable it

Edit `~/.omp/agent/mcp.json` for user scope, or `.omp/mcp.json` for project scope. Keep the rest of the file.

```json
{
  "command": "npx",
  "args": ["-y", "@drawio/mcp"],
  "env": {
    "DRAWIO_BROWSER": "off"
  },
  "enabled": true
}
```

Restart OMP or run `/mcp reload`, then check with `/mcp list`.

`DRAWIO_BROWSER=off` returns the diagram URL without launching a browser. Leave it on and every `open_drawio_*` call opens a tab, which is noisy when an agent draws ten diagrams.

## Headless machines

The `open_drawio_*` tools hand back a URL, not a file. On a server with no browser, use either of these:

Set `DRAWIO_BROWSER=off` so the agent works from URLs and you save the XML yourself.

Or run the draw.io Desktop CLI, which writes `.drawio` files and exports PNG, SVG, and PDF without a browser. The MCP Tool Server does not call it, so agents cannot reach it through MCP. The Claude Code plugin in the same repository uses it, if you want a skill-driven path instead.

## Self-hosted draw.io

```json
{
  "command": "npx",
  "args": ["-y", "@drawio/mcp"],
  "env": {
    "DRAWIO_BASE_URL": "https://drawio.example.com/"
  },
  "enabled": true
}
```

## What the agents do with it

The diagram engineer tests the server before drawing: create an XML diagram, list and edit pages in an existing file, search the shape library, and fetch the generation reference. It records what worked.

Then it draws with native shapes and connectors, saves `.drawio` sources under `design/diagrams/`, and reopens each file to check labels and endpoints.

When the server cannot write a file, the engineer reports `Blocked` or `Partial`. It does not fall back to FigCraft, a Mermaid-only render, or a screenshot, and it never presents an editor URL or a PNG as a saved project file.

Source: https://github.com/jgraph/drawio-mcp/tree/main/mcp-tool-server