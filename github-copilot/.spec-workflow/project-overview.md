Given the `{project}` path as an argument.

Produce a single Markdown report that documents the project purpose, high-level architecture, folder/project structure, communication channels (HTTP, gRPC, message brokers), data flow, primary technologies, dependency registration and DI wiring, key patterns, configuration and secrets, and tests/build status. Include file links and small code snippets to support claims. The report should follow the Requirements Workflow conventions (use `requirements/` for requirement artifacts).

When external documentation or guidance is relevant, prefer official Microsoft Learn/.NET guidance. NOTE: work only with the repository files available in the workspace — do not call external networks. Use official documentation only as a conceptual reference; do not fetch external content.

## Scope & constraints

Work only with the repository files available in the workspace. Do not call external networks.

When referencing files or symbols, include the relative path in backticks and, where helpful, include short code excerpts (<= 25 lines).

When working with related projects, prefer using an `overview.md` file from the related project (if present) rather than reading every file in that dependent project.

Prefer reading whole files (for example: `Program.cs`, `Startup.cs` or `Program.*.Testing.cs`, `.csproj` files, and important integration classes) rather than isolated lines.

If the repository is large, prioritize service/API projects and shared libraries (top-level folders, projects under `src/` and `tests/`).

If you need clarification, ask at most two concise questions.


## Search order and key files to inspect (follow roughly this order):
1. Root indicators: `global.json`, `*.sln*`, `nuget.config`, `Directory.*.props`/`Directory.*.targets`.
2. Solution and project files: all `*.sln*`, `src/**/**/*.csproj`, `tests/**/**/*.csproj`.
3. Startup wiring: `Program.cs`, `Program.*.Testing.cs`, `Startup.cs`, `Startup.*.cs`, `Extensions/*.cs`.
4. API surface: `src/**/Apis/`, controllers, minimal APIs in `Program.cs`, `Controllers/`.
5. Communication/eventing: folders/files named `EventBus`, `EventBusRabbitMQ`, `IntegrationEvents`, `Grpc`, `Proto`, `Contracts/`.
6. DI and service registration: files named `DependencyInjection`, `Extensions`, `Services`, `*Extensions.cs`, or `ServiceCollection` extension methods.
7. Repositories/EF: folders named `Repositories`, `Infrastructure`, `Persistence`, `IntegrationEventLogEF`.
8. Messaging clients/consumers: RabbitMQ, Kafka, Azure Service Bus, or HTTP webhook clients (e.g., `WebhookClient`, `Webhooks.API`).
9. Config & secrets: `appsettings*.json`, `appsettings.Development.json`, `Directory.Packages.props`, and any `UserSecrets` configuration.

## Deliverables and required sections in the report (produce these exact headings)

- Title line: repo name and one-sentence purpose.
- Summary: 3–5 bullet overview (what it does, primary techs, major services).
- Projects & folder map: list each top-level project/folder (e.g., `src/Catalog.API`) with a 1-line purpose and main entry files.
- Component diagram (text or Mermaid): represent services, databases, message brokers, and external dependencies.
- Communication channels: enumerate HTTP endpoints, gRPC services (proto files), message queues/exchanges, and webhook endpoints with file links and example endpoints/ports.
- Data flow: for two main use-cases (choose the two most important, e.g., "Place Order" and "Add Item to Basket"), provide a numbered sequence of calls/events and reference the files that implement each step.
- Dependency registration and DI wiring: list where services are registered (file and method), the DI container in use, and examples of transient/scoped/singleton registrations. Include code snippets showing the registration lines.
- Configuration and secrets: list configuration sources and keys, any use of cloud SDKs (Azure/AWS/GCP), and locations of sensitive configuration (appsettings, environment variables). Note any use of user secrets or Key Vault.
- Persistence & data access: list databases and ORMs used (EF Core, Dapper), migrations locations, and repository patterns.
- Patterns & architecture notes: list applied patterns (CQRS, Mediator, Repository, Event-driven, Hexagonal, Clean Architecture) with examples and file references.
- Security & operational considerations: authentication/authorization mechanisms, known risks (hard-coded secrets, permissive CORS), observability (logging, metrics, health checks), and deployment notes (Dockerfiles, manifests, Helm).

## Formatting & style requirements

- Output must be a single Markdown document. Use the headings from the "Deliverables" section above.
- Where you reference code or symbols, show the relative path in backticks and include a 2–6 line code snippet where it supports a claim (for example, showing DI registration or an event class).
- For diagrams, prefer Mermaid syntax where possible and also provide an ASCII fallback.
- Keep the report actionable and concise — prefer bullet lists and short paragraphs. Use tables sparingly.

Notes:
1. If anything is missing or ambiguous, ask at most two focused follow-up questions that will unblock analysis.
2. Produce the full Markdown report in `{project}/overview.md`.