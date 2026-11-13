# MCP integrations

The design phase needs two MCP servers. Neither is bundled here; the files in this folder are the OMP configs and the setup notes for each.

| Folder | Server | Carries |
| --- | --- | --- |
| `figcraft/` | FigCraft CLI, `figcraft-cli mcp` | Product screens, design system, prototype |
| `drawio/` | `@drawio/mcp` (stdio) | Every technical diagram |

Both templates keep `enabled: false` so a checkout never turns on a server you did not ask for. Enable one server at a time, reload, and check `/mcp list`.

Config files go in `~/.omp/agent/mcp.json` for user scope or `.omp/mcp.json` for project scope. Merge the server block into the file you already have; keep the top-level `$schema`, `mcpServers`, and `disabledServers` keys.

Variables in the templates expand at discovery. Defaults in parentheses.

| Variable | Used by | Default |
| --- | --- | --- |
| `FIGCRAFT_CLI` | figcraft | `figcraft-cli` |
| `FIGCRAFT_ROOT` | figcraft headless | `.` |
| `FIGCRAFT_OUT` | figcraft headless | `.` |
| `FIGCRAFT_CONTROL_PORT` | figcraft attach | `7979` |
| `DRAWIO_MCP_COMMAND` | drawio | `npx` |
| `DRAWIO_BASE_URL` | drawio | `https://app.diagrams.net/` |
| `DRAWIO_BROWSER` | drawio | `off` |

`figcraft-attached` and `figcraft` are separate entries. Use one. Attach mode needs the app running with `figcraft --control 7979`, and it refuses automation roots because the app applies its own.