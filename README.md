# Requirements Workflow

Requirements Workflow is a lightweight framework and set of guidelines to help .NET developers delegate the technical work of creating and integrating new features into brownfield projects. It focuses on requirement-driven handoffs, repeatable execution flow, and clear responsibilities so teams can move faster while maintaining stability in large existing codebases.

## What is Requirements Workflow?

Requirements Workflow is not a single tool or library. It's a workflow and supporting project layout patterns that make it easier to:

 - Capture feature intent as a concise requirement (the "requirement").
 - Translate the requirement into actionable tasks for engineers who will implement the feature in a brownfield .NET project.
 - Provide repeatable steps and checks so the integration is low-risk and traceable.

It helps product, design, and engineering collaborate by turning requirements into concrete artifacts that developers can execute against in an existing codebase.

## Goals

- Reduce friction when adding features to large, existing (.NET) projects.
- Keep the feature delivery process auditable and consistent.
- Make it easy to split responsibilities between the feature author (requirement creator) and the implementer (engineer integrating into the brownfield app).

## Key Concepts

- **Requirement:** A concise, human-readable document that captures the feature's intent, acceptance criteria, rationale, and any architectural constraints. It serves as the foundation for implementation and traceability.
- **Design Document:** Details design considerations, data flow, affected components, dependencies, and implementation steps for each functional requirement.
- **Adapter/Bridge:** Code, configuration, or scaffolding that connects the requirement to the target project, such as dependency injection wiring, feature flags, or integration points.
- **Checklist:** A step-by-step integration checklist that validates feature implementation, tracks task completion, and ensures low-risk, auditable delivery.
- **Decomposition:** The process of breaking down requirements into small, actionable, and non-overlapping implementation tasks, each documented for clarity and delegation.
- **Project Overview & Architecture:** High-level documentation that provides essential context about the repository, main components, technologies, and architectural decisions, supporting onboarding and consistent execution.
- **Workflow Prompts:** Structured automation commands (e.g., `/requirements`, `/decomposition`, `/tasks`) that guide the creation, breakdown, and implementation of features in a repeatable, transparent manner.
- **Agents:** Specialized AI agents that guide you through different roles in the workflow (Architect, BA, Dev, Team Lead). Agents provide interactive, context-aware assistance for each workflow step.

## Repo layout

This repository contains the framework and templates used to author requirements and guide integration.

- `github-copilot/` - Copilot instructions, prompts, and agents for automation and workflow guidance.
  - `copilot-instructions.md` - Main Copilot usage and instruction file.
  - **`agents/`** - AI agents for specialized workflow roles (Architect, BA, Dev, Team Lead).
  - `examples/` - Example Copilot instructions.
  - `prompts/` - Prompt templates for architecture, decomposition, project overview, requirements, and tasks.
- `LICENSE` - Project license.
- `README.md` - This documentation file.

## Quick Setup

1. Clone the repository:

   ```sh
   git clone https://github.com/arturhaikou/spec-workflow
   ```

2. Review and fulfill the rules in [`github-copilot/copilot-instructions.md`](github-copilot/copilot-instructions.md) that are relevant to your project.
   - You can see an example of a fulfilled file in [`github-copilot/examples/copilot-instructions.md`](github-copilot/examples/copilot-instructions.md).

3. Copy the following into the `.github` directory of your target project:
   - The entire [`github-copilot/prompts/`](github-copilot/prompts/) folder
   - The [`github-copilot/copilot-instructions.md`](github-copilot/copilot-instructions.md) file
   - The entire [`github-copilot/agents/`](github-copilot/agents/) folder

4. Copy the `.mcp.json` file into the appropriate folder for your editor:
   - **For VS Code:** Copy to one of the following locations:
     - `.vscode\mcp.json` - Recommended, keeps VS Code configuration organized
     - `.mcp.json` - Solution-level configuration (can be tracked in source control)
   - **For Visual Studio:** Copy to one of the following locations (in order of precedence):
     - `.vs\mcp.json` - User-specific per solution (automatically created by Visual Studio)
     - `.mcp.json` - Solution-level configuration (can be tracked in source control)

5. After completing the above steps, your project root will look like (depending on your editor):
   
   **For VS Code:**
   ```
   your-repo/
   ├── .github/
   │   ├── prompts/
   │   ├── agents/
   │   └── copilot-instructions.md
   ├── .vscode/
   │   └── mcp.json
   ├── .spec-workflow/
   ├── (other project files)
   ```
   
   **For Visual Studio:**
   ```
   your-repo/
   ├── .github/
   │   ├── prompts/
   │   ├── agents/
   │   └── copilot-instructions.md
   ├── .vs/
   │   └── mcp.json
   ├── .spec-workflow/
   ├── (other project files)
   ```
   
   **For both editors (shared config):**
   ```
   your-repo/
   ├── .github/
   │   ├── prompts/
   │   ├── agents/
   │   └── copilot-instructions.md
   ├── .mcp.json
   ├── .spec-workflow/
   ├── (other project files)
   ```

### AI Agents

**Agents are the recommended approach** for the best experience with Requirements Workflow.

**Supported Agents:**

| Agent | Role | Purpose |
|-------|------|---------|
| **Architect Agent** | Architecture & Design | Design software architectures and generate project overviews with comprehensive documentation |
| **BA Agent** | Business Analysis | Analyze requirements, identify constraints, and validate feature scope with stakeholders |
| **Dev Agent** | Implementation | Execute tasks, implement features in the codebase, and ensure code quality |
| **Team Lead Agent** | Orchestration | Coordinate workflow steps, manage task prioritization, and oversee feature delivery |

**Setup:**
1. Copy the [`github-copilot/agents/`](github-copilot/agents/) folder to your project's `.github/` directory
2. In VS Code, navigate to the Agents section and select the appropriate agent from the dropdown menu

**Usage:**
- Select the agent corresponding to your current workflow step from the Agents dropdown menu
- The agent will guide you through the process with specialized knowledge and context
- Follow the agent's instructions and provide feedback as needed

---

## Execution Flow

The following workflow uses AI agents to guide you through each step:

### 1. Repository Enrichment: Overview and Architecture

Before implementing features, ensure the repository is enriched with high-level documentation and architecture files. This provides essential context for all contributors and supports consistent onboarding and decision-making.

To enrich the repository with appropriate documents, perform the following actions:

- **Using Architect Agent:** Set architect agent and request project overview and architecture generation. The agent will guide you through the enrichment process.

- **Alternative - Direct Prompts:** You can also run the workflow prompts directly:
   - For VS Code: `/project-overview {the_path_to_the_project}`  
      Example: `/project-overview src/WebApp`
   - For Visual Studio: `#project-overview {the_path_to_the_project}`  
      Example: `#project-overview src/WebApp`

- **Analyze and adjust the final `overview.md` file for each project:**
   - Review the generated overview and add key details that can help Copilot understand the project, such as main components, technologies used, integration points, and any unique architectural decisions.

- **Run architecture prompt:**
   - For VS Code: `/architecture`
   - For Visual Studio: `#architecture`

All necessary overview and architecture files will be generated automatically by these prompts, with manual enrichment of the overview files to maximize Copilot's understanding.

### 2. Feature Implementation Flow

#### Step 1: Generate Requirements

Start by generating a requirement for the feature you want to implement. Use the requirements prompt to capture the feature intent, acceptance criteria, and any architectural constraints.

- **Using BA Agent:** Set `ba` agent and provide your feature requirements. The agent will guide you through requirement generation and validation.

- **Alternative - Direct Prompts:**
   - **VS Code:** `/requirements {user_requirements}`  
      Example: `/requirements Add a login form`
   - **Visual Studio:** `#requirements {user_requirements}`  
      Example: `#requirements Add a login form`

#### Requirements Prompt Workflow & Outputs

When you run the requirements prompt, the agent will:

- Analyze your feature description and, if needed, ask clarifying questions before generating any files. This ensures requirements are precise and actionable.
- Create a feature folder under `requirements/{FEATURE_ID}-{slug}/`, using a ticket ID if provided or generating one automatically.
- For each functional requirement (FR), generate:
   - `fr-###/fr.md`: Contains the requirement statement, rationale, acceptance criteria (Given/When/Then), and a test outline.
   - `fr-###/design.md`: Contains design considerations, data flow, affected components, dependencies, and implementation steps.
- Use RFC 2119 keywords (MUST, SHOULD, MAY) for clarity and traceability.
- If multiple features are detected, the agent may split them into separate folders and will ask for confirmation if unsure.

This process produces clear, testable requirements and design documentation, ready for implementation in your project.

#### Step 2: Break Down Requirements into Executable Tasks

After generating the requirement and design documentation, break down each functional requirement (FR) into small, executable tasks using the decomposition prompt. This step ensures that implementation work is actionable, traceable, and easy to delegate.

- **Using Team Lead Agent:** Set `team-lead` agent and request decomposition of your feature into tasks. The agent will break down the requirements and generate a task checklist.

- **Alternative - Direct Prompts:**
   - **VS Code:** `/decomposition {feature} and {fr id}`  
      Example: `/decomposition REQ-20250101-login-form and fr-001`
   - **Visual Studio:** `#decomposition {feature} and {fr id}`  
      Example: `#decomposition REQ-20250101-login-form and fr-001`

### Decomposition Phase: Outputs and Process

During the decomposition phase, the agent reviews the requirement and design documents, then breaks down the functional requirement into small, actionable implementation tasks. Each task is saved as a separate Markdown file (`task-###.md`) in the relevant requirement folder. A checklist file (`checklist.md`) is also generated, listing all tasks and their statuses, with a "Next Task for Implementation" section.

The agent may use necessary MCP tools to gather context from architecture, overview, and requirement files, but only reads what is needed. The agent does not implement tasks—only designs and documents them, ensuring tasks are clear, non-overlapping, and ready for approval before any coding begins.

#### Step 3: Implement Tasks

Once the tasks are documented and approved, proceed to implement each task by running the tasks prompt. The agent may call any necessary tools to perform the task. This step ensures that each actionable item is executed and tracked.

- **Using Dev Agent:** Set `dev` agent and provide the feature and FR IDs. The agent will implement each task step-by-step, build the solution, and track progress in the checklist.

- **Alternative - Direct Prompts:**
   - **VS Code:** `/tasks {feature} and {fr id}`  
      Example: `/tasks REQ-20250101-login-form and fr-001`
   - **Visual Studio:** `#tasks {feature} and {fr id}`  
      Example: `#tasks REQ-20250101-login-form and fr-001`

The agent will guide you through the implementation of each task, updating the checklist and marking tasks as completed. This keeps the workflow transparent and traceable, ensuring that all requirements are fulfilled and integrated into the project.

---

## Quick Reference

| Scenario | Recommended Approach | Setup Required |
|----------|----------------------|-----------------|
| Using AI Agents (VS Code) | Select agent from Agents dropdown menu | Copy `agents/` folder to `.github/` |
| Using Direct Prompts | Run `/requirements`, `/decomposition`, `/tasks` directly | Copy `prompts/` folder to `.github/` |
| Using Visual Studio | Use `#requirement`, `#decomposition`, `#tasks` syntax | Copy `prompts/` folder to `.github/` |

**Note:** Agents and direct prompts provide different interfaces to guide you through the same workflow process. Choose whichever best fits your workflow preferences.

---

## Contributing

Contributions are welcome. Use the following flow:

1. Open an issue describing what you'd like to add or improve.

## License

This project is licensed under the terms in the `LICENSE` file.

## Contact / Questions

If you have questions about the workflow, open an issue or reach out to the maintainer.

---

Happy requirementing!