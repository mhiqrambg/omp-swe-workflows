# FigCraft MCP setup

The design phase drives FigCraft through `figcraft-cli mcp`. The server speaks MCP over stdio. OMP config for it lives in `integrations/figcraft/mcp.json`.

## Install the CLI

FigCraft ships the CLI with the app, and its location differs per platform:

| Platform | Where `figcraft-cli` lands |
| --- | --- |
| Linux (`.deb`, `.rpm`, `.pkg.tar.zst`, tarball) | Installed beside the app |
| Linux (AppImage) | No binary, run `/path/to/FigCraft.AppImage cli` |
| Linux (Flatpak) | `flatpak run --command=figcraft-cli ai.storyteller.figcraft` |
| Windows | `figcraft-cli.exe` next to `figcraft.exe` |
| macOS | Download `figcraft-cli-<version>-macos-universal.zip` from the release page |

Check the install:

```sh
figcraft-cli --version
figcraft-cli mcp --help
```

macOS and Linux terminals need `figcraft-cli` on `PATH` or an absolute path in the config. Desktop launches often have neither, so the absolute path is the reliable option.

## Register the server

Two entries in `integrations/figcraft/mcp.json` cover the two modes:

`figcraft` runs headless. The server keeps an in-process engine and one untitled document, renders with the CPU renderer, and writes `.figcraft` files to disk. Use it for batch work and CI.

`figcraft-attached` connects to the running app on port `7979`. You see each edit as it happens, with the agent's avatar and the layers it touched. Start the app first:

```sh
figcraft --control 7979
```

Attach mode refuses automation roots. The app confines its own file access instead, from the flags it started with:

```sh
figcraft --control 7979 --automation-read-root /work/project --automation-write-root /work/project/out
```

For everything the workflow needs (screens, design system, prototype), headless is enough. Use attach when you want to watch the work.

## Enable it

Edit `~/.omp/agent/mcp.json` for user scope, or `.omp/mcp.json` for project scope. Copy the server block you need and keep the rest of the file.

```json
{
  "command": "figcraft-cli",
  "args": ["mcp"],
  "enabled": true
}
```

Use an absolute path when `which figcraft-cli` comes back empty in the session that starts OMP.

Then restart OMP or run `/mcp reload`. Confirm the tool roster with `/mcp list`.

## Optional variables

The template expands these before launch. Defaults in parentheses.

| Variable | Effect |
| --- | --- |
| `FIGCRAFT_CLI` (`figcraft-cli`) | Path to the CLI binary |
| `FIGCRAFT_ROOT` (`.`) | Directory the agent may read |
| `FIGCRAFT_OUT` (`.`) | Directory the agent may write |
| `FIGCRAFT_CONTROL_PORT` (`7979`) | Control port for attach mode |

Roots must be existing directories. Without a write root the agent can still export bytes, but nothing reaches disk.

## What the agents check

The design orchestrator does not assume the server is usable. It inspects the live tool roster, then tests the operations it needs: create and open a Design Canvas document, edit native layers, use variables and components, wire prototype transitions, and save a `.figcraft` file.

It records one of `Verified`, `Partial`, `Unavailable`, or `Not tested` per operation. A connection without a callable save path is `Partial`. FigCraft Boards do not count as a diagram surface in this workflow, so the orchestrator refuses them even if the server exposes them.

Source: https://github.com/storytold/figcraft/blob/main/docs/mcp.md