# spec-workflow

A lightweight, **technology-agnostic** spec-first SDD toolkit for AI-assisted feature delivery. Requirements, system designs, and implementation tasks are written as documents before any code is touched. Each agent reads the outputs of the previous step — no context is passed manually between steps.

## How it works

The workflow has two phases:

**Phase 1 — Prepare the repository for coding with AI**:  
Generate machine-readable context files that all agents use for codebase lookups.

**Phase 2 — Deliver features spec-first** (run per feature):  
A chain of agents turns a plain-language description into a traceable set of documents, then into implemented code.

| Step | Agent | Reads | Writes |
|------|-------|-------|--------|
| 1. Requirements | `business-analyst` | `architecture.md`, `overview.md` | `fr.md` — user story with BDD acceptance criteria |
| 2. System design | `architect` | `fr.md`, `architecture.md`, `overview.md` | `design.md` — tech-agnostic component impact, API contracts, schema changes |
| 3. Task decomposition | `team-lead` | `fr.md`, `design.md`, source code | `task-*.md` + `checklist.md` — copy-paste-ready implementation tasks |
| 4. Implementation | `dev-manager` | `checklist.md`, `task-*.md` | Code changes via `dev` sub-agent, checklist progress |

## Repo structure

```
github-copilot/
├── .spec-workflow/               # Templates consumed by agents at runtime
│   ├── fr_template.md
│   ├── task_file_template.md
│   └── checklist_file_template.md
├── agents/                       # Agent definitions
│   ├── business-analyst.agent.md
│   ├── architect.agent.md
│   ├── team-lead.agent.md
│   ├── dev-manager.agent.md
│   └── dev.agent.md
├── prompts/                      # Prompts for repository enrichment
│   ├── component-overview.prompt.md
│   └── project-architecture.prompt.md
├── copilot-instructions.md       # Project rules template (fill in before use)
└── examples/                     # Reference examples
```

## Setup

Copy these into your target project:

| Source | Destination |
|--------|-------------|
| `github-copilot/.spec-workflow/` | `.spec-workflow/` (project root) |
| `github-copilot/agents/` | `.github/agents/` |
| `github-copilot/prompts/` | `.github/prompts/` |
| `github-copilot/copilot-instructions.md` | `.github/copilot-instructions.md` |

Fill in the placeholder sections in `copilot-instructions.md` with your project's conventions. See [github-copilot/examples/copilot-instructions.md](github-copilot/examples/copilot-instructions.md) for a completed reference.

After setup your project root will contain:

```
your-repo/
├── .github/
│   ├── agents/
│   ├── prompts/
│   └── copilot-instructions.md
├── .spec-workflow/
└── (your project files)
```

## Usage

### Phase 1: Prepare the repository

This phase runs once and provides agents with machine-readable context for your project.

**1. Configure copilot-instructions.md**  
Fill in the placeholder sections in `.github/copilot-instructions.md` with your project's conventions:
- Project scaffolding rules
- Database integration rules  
- Existing project modification rules
- Data flow update rules
- Any other technology-specific or architectural guardrails

Reference [github-copilot/examples/copilot-instructions.md](github-copilot/examples/copilot-instructions.md) for a completed example.

**2. Create custom skills**  
For technology-specific guidance, create skill files in `.github/skills/`. These define patterns, best practices, and implementation standards for your tech stack (e.g., API design, UI patterns, database schemas). Agents reference these during task decomposition to generate accurate implementations. See the examples in `github-copilot/examples/skills/` for reference patterns.

**3. Generate context files**  
Generate the system architecture once for the entire project, and component overviews for each major component:

```
/project-architecture ./
/component-overview {path_to_component}
```

The `project-architecture` prompt produces a single root-level `architecture.md` showing how all services communicate. The `component-overview` prompt generates a detailed `overview.md` for each major component, describing its internal structure, responsibilities, and interactions.

**4. Enrich overviews**  
Review and add any details to the generated `overview.md` files that are not obvious from code alone (integration points, constraints, key architectural decisions). These files become the reference for all agents.

### Phase 2: Deliver features spec-first

Select agents from the **Agents** dropdown in VS Code and run them in order:

**1. `business-analyst` — Write functional requirements**  
Describe the feature. The agent asks clarifying questions, presents a requirement plan for approval, then writes one `fr.md` per functional requirement.

**2. `architect` — Write system designs**  
Provide the requirements folder path. The agent reads each `fr.md` and produces a tech-agnostic `design.md` covering impacted components, API contract changes, and abstract schema updates. No programming-language-specific content.

**3. `team-lead` — Decompose into tasks**  
Provide the FR folder path. The agent scans the actual source code and produces `task-001-[name].md`, `task-002-[name].md`, etc. with exact file paths and insertion points, plus a `checklist.md` tracking execution order.

**4. `dev-manager` — Implement**  
Provide the `checklist.md` path. The agent works through the checklist sequentially, delegating each task file to the `dev` sub-agent and updating task status. Stops immediately on any failure and reports the error.

### Output structure

```
requirements/
└── [REQ_ID]/
    ├── initial_user_request.md
    └── [fr_index]/
        ├── fr.md
        ├── design.md
        ├── task-001-[name].md
        ├── task-002-[name].md
        └── checklist.md
```

## Contributing

Open an issue describing what you'd like to add or improve.

## License

This project is licensed under the terms in the `LICENSE` file.

## Contact / Questions

If you have questions about the workflow, open an issue or reach out to the maintainer.

---

Happy requirementing!
