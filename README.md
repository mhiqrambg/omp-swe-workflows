# omp-swe-workflows

Agent definitions and slash commands for two OMP workflows: a software blueprint phase and a design phase.

The blueprint phase runs requirements, architecture, data, API, UX, and security analysis, then returns one integrated proposal. The design phase takes that proposal and produces FigCraft screens, a design system, draw.io diagrams, a clickable prototype, and a reviewed handoff.

The workflow files are plain Markdown. There is no build step and nothing to compile.

## What is in the repo

| Path | Contents |
| --- | --- |
| `agents/blueprint/` | 7 agents for requirements, architecture, database, API, UX, and security work, plus the orchestrator that coordinates them |
| `agents/design/` | 6 agents for product screens, design system, diagrams, prototypes, review, plus the orchestrator |
| `commands/` | `/swe-blueprint` and `/swe-design` |

| Agent | Group | Job | Writes files |
| --- | --- | --- | --- |
| `software-architect` | blueprint | Runs the phase, delegates to the six analysts, integrates the result | no |
| `project-requirements-analyst` | blueprint | Scope, roles, functional and non-functional requirements, acceptance criteria | no |
| `system-architecture-analyst` | blueprint | System boundaries, architecture options, deployment shape, ADRs | no |
| `database-designer` | blueprint | Conceptual and logical model, ERD, constraints, physical design notes | no |
| `api-designer` | blueprint | Resources, operations, schemas, errors, pagination, authorization | no |
| `ui-ux-analyst` | blueprint | Journeys, navigation, screen inventory, states, accessibility | no |
| `security-auditor` | blueprint | Threat model, control placement, prioritized mitigations | no |
| `design-orchestrator` | design | Plans the phase, verifies the MCP lanes, delegates, reconciles review | yes |
| `product-designer` | design | Screen inventory and high fidelity screens in FigCraft | yes |
| `design-system-engineer` | design | Variables, modes, styles, components, variants | yes |
| `diagram-engineer` | design | All technical diagrams through draw.io | yes |
| `prototype-engineer` | design | Interaction wiring and prototype test report | yes |
| `design-reviewer` | design | Independent review of both lanes | no |

The blueprint agents are read-only on purpose. They report findings to the orchestrator, and the orchestrator returns one document in the conversation. The design agents write to a `design/` directory in the project they run against.

## Installation

OMP loads agents from `~/.omp/agent/agents/*.md` (user scope) or `.omp/agents/*.md` (project scope), and commands from the matching `commands` directory. Both directories are scanned at the top level only.

This repo keeps agents in subfolders, so copy the files themselves. Copying the folders as they are puts the Markdown one level too deep, OMP finds nothing, and `/swe-blueprint` fails with `Unknown agent "software-architect"`.

### User scope

Available in every project on the machine:

```sh
git clone https://github.com/mhiqrambg/omp-swe-workflows.git
cd omp-swe-workflows

mkdir -p ~/.omp/agent/agents ~/.omp/agent/commands
cp agents/blueprint/*.md agents/design/*.md ~/.omp/agent/agents/
cp commands/*.md ~/.omp/agent/commands/
```

### Project scope

Available only in the repo where you place them, and it lets a project pin its own copy:

```sh
mkdir -p .omp/agents .omp/commands
cp path/to/omp-swe-workflows/agents/blueprint/*.md .omp/agents/
cp path/to/omp-swe-workflows/agents/design/*.md .omp/agents/
cp path/to/omp-swe-workflows/commands/*.md .omp/commands/
```

Project scope wins over user scope when both define the same name.

### Check the install

Open `/agents` in OMP and look for the 13 names in the table above. Typing `/swe` should offer both commands. If an agent is missing, check that its file sits directly in `~/.omp/agent/agents/` and that the frontmatter starts with `---` on line 1.

### Updating

```sh
git pull
cp agents/blueprint/*.md agents/design/*.md ~/.omp/agent/agents/
cp commands/*.md ~/.omp/agent/commands/
```

OMP rediscovers agent files before each spawn, so updates apply without restarting OMP.

## Requirements for the design phase

The design phase drives two MCP servers, and the agents check each one before doing any work:

- FigCraft MCP for Design Canvas documents, design system work, and the interactive prototype.
- draw.io MCP for ERD, architecture, deployment, data flow, sequence, and process diagrams.

MCP tools are additive in OMP. The `tools:` line in an agent file restricts built-in tools only, so the design agents reach these servers through whatever session spawns them.

Ready-made configs and setup notes for both servers live in [`integrations/`](integrations/README.md).

If the draw.io server is missing or only returns a preview, the orchestrator marks diagram work Blocked. It does not fall back to FigCraft or a Mermaid export. If FigCraft is missing, the UI, design system, and prototype work is marked Blocked or Partial. The capability report records what was tested and what came back, so a gap stays visible in the handoff.

## Usage

### 1. Blueprint

Run this inside the project you want to analyze:

```
/swe-blueprint
```

The `software-architect` agent inspects the repo, plans its own sequence, delegates to the six analysts, then integrates everything into one document: requirements with confirmed or assumed status, architecture options, data and API summaries, UX structure, security findings, a decision register, and a roadmap.

Every decision stays in `Proposed` status until you approve it.

The orchestrator has no write tool, so the blueprint lives in the reply. Save it to the project before moving on. The design phase reads it from disk, and the design orchestrator expects files along the lines of `00-overview.md`, `01-requirements.md`, `02-ux.md`, `03-architecture.md`, `04-data-model.md`, `05-api.md`, and `06-security.md`. Point it at whatever names you actually used.

### 2. Design

```
/swe-design
```

The `design-orchestrator` reads the blueprint, inventories the existing design files, and tests both MCP servers separately. Only after that does it spawn the specialists:

1. `product-designer` builds screens and the screen inventory.
2. `design-system-engineer` builds tokens, components, and variants.
3. `product-designer` returns to apply the shared components.
4. `diagram-engineer` produces every technical diagram through draw.io.
5. `prototype-engineer` wires the screens into tested flows.
6. `design-reviewer` reviews both lanes against the blueprint.
7. The orchestrator reconciles findings and writes the handoff.

Output lands in `design/`:

```
design/00-design-plan.md
design/01-tool-capability-report.md
design/02-screen-inventory.md
design/03-design-system-spec.md
design/04-component-and-state-spec.md
design/05-interaction-spec.md
design/06-diagram-index.md
design/07-prototype-test-report.md
design/08-design-review.md
design/09-implementation-handoff.md
design/diagrams/*.drawio
```

Native `.figcraft` and `.drawio` sources are the primary artefacts. Exports and editor links are noted as such, never as a saved project file.

The phase ends with one of three statuses: `Ready for owner approval`, `Partially ready`, or `Blocked`. Approval is yours to give. The agents cannot approve their own output, and the design phase does not authorize implementation.

## Notes and limits

- Both orchestrators inherit the session model. No agent sets `model` or `thinkingLevel`, so routing is yours to control through OMP settings.
- The blueprint phase marks unknown requirements as assumed or open, so a missing input does not turn into an invented value.
- The design phase needs real project files to read. A blueprint that only exists in a chat transcript is not enough.
- `design-reviewer` is read-only. It reports findings to the orchestrator and never edits artefacts.
- The repo has no installer. The copy commands above are the install.

## License

[PolyForm Noncommercial 1.0.0](LICENSE). You may use, change, and redistribute this project for noncommercial purposes. Selling it, or using it in a product or service you sell, is not permitted under these terms.

The license does not stop you from using these agents in your own job, even at a company. What it blocks is commercial distribution: reselling the agents, bundling them into a paid product, or offering them as part of a service.

Commercial rights are not included. If you need them, ask the copyright holder for a separate agreement.
