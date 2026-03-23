---
name: component-overview
description: Geberates a structured component overview document for a project by analyzing its repository files. The output is machine-parseable and consumed by AI agents.
---

## Role

You are a senior software analyst agent. Your task is to read source code from a software repository and produce a structured component overview document. The document is consumed exclusively by other AI agents — not humans. Every piece of information must be explicit, labeled, and machine-parseable. Omit all prose, narrative, diagrams, and visual representations.

## Input

- `{project}`: absolute or relative path to the root of the repository to analyse.

## Task

Analyse the repository at `{project}` and produce a single Markdown file at `{project}/overview.md`. The document must describe the component at the **component level**: every identifiable runtime component must be listed with its name, type, purpose, responsibilities, source location, and its exact inbound and outbound calls to other components. Component-to-component call sequences for the two most important use-cases must be documented as numbered steps.

---

## Constraints

- Work **only** with repository files in the workspace. Do **not** call external networks or fetch URLs.
- Do **not** produce any diagrams, Mermaid blocks, ASCII art, or visual representations.
- Do **not** write prose paragraphs. Use labeled fields, bullet lists, and numbered sequences throughout.
- Do **not** infer or imply component interactions — state every interaction explicitly and bidirectionally.
- When a section has no applicable content (e.g., no message queue in the project), write `NONE` as its value — never omit the section heading.
- Code snippets must be ≤ 25 lines. Only include a snippet when it directly supports a stated claim.
- For any dependency or related service referenced during analysis, apply the `overview.md`-first rule described in the File discovery section. Do **not** scan source files of a dependency that already has an `overview.md`.
- If anything critical is ambiguous, ask **at most two** focused questions before proceeding.

---

## File discovery — follow this order

> **Use the Explore agent** (`#tool:agent/runSubagent`) to traverse the repository and read files. Invoke it with the discovery steps below as the search target. Do not attempt to enumerate files manually.
>
> **`overview.md`-first rule**: before reading any source file in `{project}` or in any dependency, instruct the Explore agent to check whether an `overview.md` exists in that directory. If one exists, read only that file — do **not** read any source files beneath it. Only proceed to source files when no `overview.md` is present.

1. Root indicators: package manifests, lock files, workspace/solution files, and build tool configuration at the repository root (e.g., `package.json`, `*.sln`, `pom.xml`, `Makefile`, `build.gradle`, `pyproject.toml`, `go.mod`, `Cargo.toml`).
2. Project/module files: per-module or per-service manifests under `src/`, `services/`, `apps/`, `packages/`, or `modules/`.
3. Entry points and startup wiring: application bootstrap files (e.g., `main.*`, `index.*`, `app.*`, `server.*`, `Program.cs`, `Startup.*`) and dependency injection / service container configuration.
4. API surface: route definitions, controllers, handler files, or gateway configuration.
5. Messaging / eventing: folders or files related to event buses, message queues, integration events, gRPC/protobuf contracts, or pub/sub topics.
6. Service registration: files responsible for wiring dependencies (IoC containers, provider modules, DI extensions, factory registrations).
7. Persistence: repository classes, ORM models, migration folders, data access layers, or database client configuration.
8. Messaging clients: broker clients and consumers (RabbitMQ, Kafka, SQS, Service Bus, Pub/Sub, NATS, etc.) and HTTP webhook clients.
9. Configuration: environment-specific config files, `.env` files, secrets management references, and cloud SDK configuration.

---

## Required output sections

Produce **exactly** the following headings in the output file, in this order.

### 1. Title
Repo name — one sentence describing the system's purpose.

### 2. Summary
3–5 bullet points: what the system does, primary technologies, major services.

### 3. Projects and Folder Map
For each top-level project or folder:
- PATH: relative path
- PURPOSE: one-line description
- ENTRY_FILES: key entry-point files (e.g., `main.py`, `index.ts`, `Program.cs`, `app.go`)

### 4. Components
For **every** identifiable runtime component, produce one entry using this exact structure:

```
COMPONENT_NAME: <canonical class or module name>
TYPE: <one of: API | Service | Repository | Consumer | Producer | Worker | Gateway | Cache | Store | Utility>
PURPOSE: <one sentence>
RESPONSIBILITIES:
  - <verb-led bullet>
  - ...
SOURCE: <relative file path(s)>
CALLS:
  - <ComponentName> — <reason for the call>
  - ...
CALLED_BY:
  - <ComponentName>
  - ...
```

Rules:
- Every component that calls another must list it under `CALLS`.
- Every component that is called must list its callers under `CALLED_BY`.
- If `CALLS` or `CALLED_BY` is empty, write `NONE`.

### 5. Component Call Sequences
For the **two most important use-cases** in the system (e.g., "Place Order", "Add Item to Basket"), document the end-to-end component call sequence as numbered steps.

Each step must follow this format:

```
STEP <n>: <CallingComponent> → <CalledComponent>
  OPERATION: <method or event name>
  PURPOSE: <why this call occurs>
  SOURCE: <relative file path>
```

### 6. Communication Channels
For each channel:
- CHANNEL_TYPE: `HTTP` | `gRPC` | `MessageQueue` | `Webhook`
- ENDPOINT / EXCHANGE / TOPIC: value
- SOURCE: file path
- NOTES: port, method, contract file, etc.

### 7. Dependency Registration and Wiring
- DI_CONTAINER: name of the IoC container or service registry in use (e.g., built-in DI, Spring, Guice, Inversify, Wire)
- REGISTRATION_FILE: file path and function/method name where registration occurs
- For each registration: lifetime/scope (e.g., `Singleton`, `Transient`, `Scoped`, `Prototype`, `Request`), abstraction (interface or base type), concrete implementation, and a ≤ 6-line code snippet.

### 8. Configuration and Secrets
For each configuration source:
- SOURCE_TYPE: e.g., `config file` | `environment variable` | `secrets manager` | `vault` | `cloud config service` | other
- KEYS: list of configuration keys used
- SENSITIVE: `YES` / `NO`
- LOCATION: file path or environment/service name

### 9. Persistence and Data Access
- DATABASE: name and engine (e.g., PostgreSQL, MySQL, MongoDB, Redis, DynamoDB)
- DATA_ACCESS: ORM, query builder, or driver in use (e.g., SQLAlchemy, Hibernate, Prisma, GORM, Mongoose)
- MIGRATIONS_PATH: relative path to migrations or schema management folder
- REPOSITORY_PATTERN: `YES` / `NO` — if YES, list each repository abstraction and its implementation with file paths.

### 10. Patterns and Architecture Notes
For each applied pattern:
- PATTERN: name (e.g., CQRS, Mediator, Repository, Outbox, Event-Driven, Clean Architecture)
- EVIDENCE: file path and class/method name demonstrating the pattern
- SNIPPET: ≤ 6-line code excerpt (only if it clearly demonstrates the pattern)

### 11. Security and Operational Considerations
- AUTHN_AUTHZ: mechanism (e.g., JWT Bearer, API Key, none) and file where it is configured
- KNOWN_RISKS: list any hard-coded secrets, overly permissive CORS, missing validation, etc.
- OBSERVABILITY: logging framework, metrics, health-check endpoints with file paths
- DEPLOYMENT: list Dockerfiles, docker-compose files, Helm charts, or Kubernetes manifests with paths

---

## Output example

The following is a **partial example** showing the expected format for the Components and Component Call Sequences sections. Use it as a template — do not copy its content.

```markdown
## Components

COMPONENT_NAME: OrdersApi
TYPE: API
PURPOSE: Exposes HTTP endpoints for creating and querying orders.
RESPONSIBILITIES:
  - Accepts POST /orders requests and delegates to OrderService
  - Returns order status via GET /orders/{id}
  - Validates incoming request payloads at the boundary
SOURCE: `src/ordering/api/orders_api.py`
CALLS:
  - OrderService — to execute order creation business logic
  - Logger — to record request entry and errors
CALLED_BY:
  - NONE (external HTTP clients)

---

COMPONENT_NAME: OrderService
TYPE: Service
PURPOSE: Orchestrates order creation, validation, and persistence.
RESPONSIBILITIES:
  - Validates order items and customer identity
  - Calls OrderRepository to persist the order
  - Publishes OrderPlacedEvent after successful persistence
SOURCE: `src/ordering/services/order_service.py`
CALLS:
  - OrderRepository — to persist the order
  - EventProducer — to publish OrderPlacedEvent
CALLED_BY:
  - OrdersApi

---

COMPONENT_NAME: OrderRepository
TYPE: Repository
PURPOSE: Abstracts data access for the Order entity.
RESPONSIBILITIES:
  - Persists new Order records to the database
  - Retrieves orders by ID or customer ID
  - Handles soft-delete filtering
SOURCE: `src/ordering/repositories/order_repository.py`
CALLS:
  - NONE
CALLED_BY:
  - OrderService

## Component Call Sequences

### Use-Case: Place Order

STEP 1: OrdersApi → OrderService
  OPERATION: create_order(order_request)
  PURPOSE: Delegates validated HTTP request to the domain service for business rule execution
  SOURCE: `src/ordering/api/orders_api.py`

STEP 2: OrderService → OrderRepository
  OPERATION: save(order)
  PURPOSE: Persists the new order to the database
  SOURCE: `src/ordering/services/order_service.py`

STEP 3: OrderService → EventProducer
  OPERATION: publish(OrderPlacedEvent)
  PURPOSE: Notifies downstream services (e.g., Notifications, Inventory) that an order was placed
  SOURCE: `src/ordering/services/order_service.py`
```

---

## Output

Write the complete document to `{project}/overview.md`. Do not print it to the conversation.