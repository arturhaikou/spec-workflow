Follow only these instructions when contributing to the Requirements Workflow solution.

## Rules

### 1. New project scaffolding
- Use an appropriate `dotnet new` template.
- Add the new project to the solution file.
- Add the project to the `eShop.AppHost.csproj` project references.
- Register the project in Aspire's distributed application builder in `Program.cs`.
- Use minimal APIs for Web API projects.
- Avoid adding Swagger for new services.

---

### 2. New database integration
- Before you do anything, gather official documentation using the Microsoft Learn MCP tools (`microsoft_docs_search` and `microsoft_docs_fetch`) to understand how to integrate the chosen database or any Microsoft technology with .NET Aspire projects. This research is mandatory and should be completed before preparing an execution plan or making code changes.
- Integrate database services in the Orchestrator project using the appropriate Aspire hosting package.
- Integrate database services in the target project using the appropriate Aspire client package.
- Wire the database integration into Aspire's distributed application builder in `Program.cs`.
- Add connection strings to the target project's `appsettings.json` and `appsettings.Development.json`, following the project's convention for connection string names and secret management.

---

#### 2.1 Documentation gathering
- For any work involving Microsoft products, services, or SDKs, always use the MCP tools (`microsoft_docs_search` and `microsoft_docs_fetch`) to locate and fetch the latest official documentation before proceeding.
- Proactively use MCP tools as part of your standard workflow for Microsoft-related tasks; do not wait for explicit instruction.
- Use the gathered documentation to inform your execution plan, configuration choices, and code changes.

---

### 3. Existing project modification
- Follow existing patterns for dependency injection (DI), configuration, coding style, and UI integration described in the relevant `overview.md`.
- Apply the architecture, patterns, and best practices documented in the project's `overview.md`.
- If you add a new service or feature, ensure the solution includes a project reference and that the feature is registered in Aspire's distributed application builder (`Program.cs`).
- If changes introduce or modify database models, create and test migrations for those changes.
- Keep all service contracts and public interfaces up-to-date to reflect any behavioral changes.

---

#### 3.1 WebApp UI feature additions
- For any UI feature or form added to the WebApp, follow the "UI Form Integration Example" in `src/WebApp/overview.md`. Key requirements:
  1. Define the form in Razor markup (for example: `<form method="post" data-enhance>`).
  2. Connect the form to a handler using the component form helper and `@onsubmit` (for example: `<form @formname="update-cart" @onsubmit="@UpdateQuantityAsync">`) further down in the code.
  3. Bind form fields to component properties using `@bind-Value` or use `[SupplyParameterFromForm]` on component parameters where appropriate.
  4. Implement the handler method for submission and wire it to the form's submit event.
  5. Update state and UI after submission, following the example in `CartPage.razor`.
 - Always review and apply the markup, binding, and handler conventions shown in the example to ensure consistent behavior.

---

### 4. Data flow updates

#### 4.1 Data flow update rules
- If a new feature affects an existing data flow:
  1. Analyze `architecture.md` to understand the current data flow and service interactions.
  2. Identify all dependent and affected services (for example: `Basket.API`, `Catalog.API`, `Ordering.API`, `WebApp`).
  3. Update each affected service to handle the new or changed data flow, following established patterns and contracts.
  4. Ensure service interfaces, contracts, and event/message definitions remain compatible across services, and update them when necessary.

---

## MCP tools reminder
You have access to the Microsoft Learn MCP tools (`microsoft_docs_search` and `microsoft_docs_fetch`). Use them to search for and fetch the latest official documentation whenever your work involves Microsoft products, SDKs, or services. Use that documentation to guide design decisions and implementation details.