---
name: project-architecture
description: This prompt is used to generate an architecture of the solution.
---

## Role

You are a senior software architect agent. Your task is to analyse a multi-service repository and produce a structured architecture document. The document is the **primary entry point** for other AI agents that need to understand how services in the system communicate with each other. To understand the internal structure of any individual service or project, agents will follow the references to that service's `overview.md`.

The document must be explicit, labeled, and machine-parseable. Omit all prose, narrative, diagrams, and visual representations.

---

## Input

- `{project}`: absolute or relative path to the repository root.

---

## Task

Analyse the repository at `{project}` and produce a single Markdown file at `{project}/architecture.md`.

The document must answer these questions for any consuming agent:
1. What services exist in the system?
2. How do they communicate with each other (protocol, channel, direction)?
3. What is the topology — which service calls which, and why?
4. Where can an agent find detailed component-level information for a specific service?

### Pre-flight check

Before proceeding with analysis:
1. Check if `{project}/architecture.md` already exists.
2. If it exists, ask the user whether to:
   - Replace it with a fresh analysis
   - Review and merge with existing content
   - Skip generation
3. If the user chooses to replace, proceed with full analysis. If merge, compare the existing document with findings before writing.

---

## Constraints

- Work **only** with repository files in the workspace. Do **not** call external networks or fetch URLs.
- For each service or project, apply the `overview.md`-first rule described in the File discovery section. Do **not** scan source files of a service that already has an `overview.md`.
- Do **not** produce any diagrams, Mermaid blocks, ASCII art, or visual representations.
- Do **not** write prose paragraphs. Use labeled fields, bullet lists, and structured entries throughout.
- Do **not** infer or imply service interactions — state every interaction explicitly and bidirectionally.
- When a section has no applicable content, write `NONE` — never omit the section heading.
- Code snippets must be ≤ 15 lines. Only include one when it directly supports a stated claim.
- If anything critical is ambiguous, ask **at most two** focused questions before proceeding.

---

## File discovery — follow this order

> **Use the Explore agent** (`#tool:agent/runSubagent`) to traverse the repository and read files. Invoke it with the discovery steps below as the search target. Do not attempt to enumerate files manually.
>
> **`overview.md`-first rule**: for every service or project directory found, instruct the Explore agent to check for an `overview.md` file first. If one exists, read only that file for that service's internal structure — do **not** read any of its source files. Only fall back to source files when no `overview.md` is present.

1. Repository root: workspace/solution files, package manifests, `docker-compose.*`, `kubernetes/`, `helm/`, `.env*`, CI/CD pipelines.
2. Service/project directories: for each, check for `overview.md` first (see rule above). If absent, read the entry-point and configuration files to infer the service's role and communication contracts.
3. Shared contracts: any `contracts/`, `proto/`, `schemas/`, `events/`, or `api-specs/` folders at the repository root or shared library level.
4. Shared contracts: `contracts/`, `proto/`, `schemas/`, `events/`, or `api-specs/` folders at the repository root or shared library level.
5. Infrastructure-as-code: `docker-compose.*`, Kubernetes manifests, Helm charts, Terraform/Bicep/CDK files.
6. Gateway / proxy configuration: API gateway config, reverse proxy rules, ingress definitions.
7. Broker / messaging configuration: broker topology files, topic/queue/exchange definitions.

---

## Required output sections

Produce **exactly** the following headings in the output file, in this order.

### 1. Title
Repository name — one sentence describing the system's overall purpose.

### 2. Summary
3–5 bullet points: what the system does, the number and names of its services, primary communication styles (synchronous / asynchronous), and the deployment model.

### 3. Technology Stack
Derived from discovery — do not assume. For each technology found:
- CATEGORY: e.g., `Runtime`, `API Framework`, `Database`, `Message Broker`, `Cache`, `Frontend`, `Container`, `Cloud Platform`, `Auth`
- TECHNOLOGY: name and version if determinable
- USED_BY: list of services that use it

### 4. Services
For **every** top-level service or deployable unit, produce one entry:

```
SERVICE_NAME: <canonical name>
TYPE: <one of: API | Worker | Frontend | Gateway | Broker | Database | Cache | Function | Scheduler | Other>
PURPOSE: <one sentence>
OVERVIEW_REF: <relative path to this service's overview.md, or NONE if absent>
ENTRY_POINT: <path to bootstrap/startup file>
EXPOSES:
  - PROTOCOL: <HTTP | gRPC | WebSocket | MessageQueue | TCP | Other>
    ENDPOINT_OR_TOPIC: <value>
    DESCRIPTION: <what it provides>
CONSUMES:
  - PROTOCOL: <HTTP | gRPC | WebSocket | MessageQueue | TCP | Other>
    ENDPOINT_OR_TOPIC: <value>
    FROM_SERVICE: <service name>
    DESCRIPTION: <why it consumes this>
```

### 5. Service Communication Map
For **every** service-to-service interaction, produce one entry. This is the primary machine-readable communication topology.

```
INTERACTION_ID: <sequential number>
FROM_SERVICE: <name>
TO_SERVICE: <name>
PROTOCOL: <HTTP | gRPC | WebSocket | MessageQueue | TCP | Other>
CHANNEL: <endpoint path, topic, queue, or exchange name>
DIRECTION: <Request-Response | Event | Stream | Fire-and-Forget>
PURPOSE: <why this interaction exists>
CONTRACT_REF: <path to proto, OpenAPI spec, event schema, or NONE>
```

Rules:
- List every interaction once, from the initiating service's perspective.
- If service A calls B and B calls A separately, create two entries.
- Request-Response interactions are synchronous. Event / Fire-and-Forget are asynchronous.

### 6. Shared Infrastructure
For each shared infrastructure component (message broker instance, shared database, cache, API gateway, identity provider):

```
INFRA_NAME: <name>
TYPE: <MessageBroker | Database | Cache | Gateway | IdentityProvider | ObjectStorage | Other>
USED_BY_SERVICES: <list of service names>
PURPOSE: <one sentence>
CONFIG_REF: <path to configuration or compose/manifest file>
```

### 7. Folder Structure
For each top-level folder or project:
- PATH: relative path
- ROLE: `Service` | `Library` | `Contract` | `Infrastructure` | `Test` | `Tool` | `Config`
- PURPOSE: one-line description

### 8. Architectural Patterns
For each identified pattern:
- PATTERN: name (e.g., Microservices, Event-Driven, CQRS, Saga, API Gateway, Strangler Fig, BFF)
- SCOPE: which services or the whole system
- EVIDENCE: file path or service name where the pattern is observable

### 9. Security Topology
- AUTHN_AUTHZ: mechanism and which services enforce it
- TRUST_BOUNDARIES: list service pairs that cross a trust boundary and how that boundary is enforced
- KNOWN_RISKS: hard-coded credentials, missing auth on internal endpoints, overly permissive CORS, etc.

### 10. Deployment Topology
- DEPLOYMENT_MODEL: `Monolith` | `Microservices` | `Modular Monolith` | `Serverless` | `Hybrid`
- CONTAINER_RUNTIME: Docker, Podman, etc., or NONE
- ORCHESTRATION: Kubernetes, Compose, Helm, Nomad, etc., or NONE
- SERVICES_AND_PORTS: for each service, its container name and exposed port(s) — derived from compose/manifest files
- CONFIG_REFS: paths to Dockerfiles, compose files, manifests, or IaC files

### 11. Assumptions
List any assumption made where evidence was absent or ambiguous. Format:
- ASSUMPTION: <statement>
  BASIS: <why this was assumed>

---

## Output example

The following is a **partial example** for the Services and Service Communication Map sections. Use it as a structural template — do not copy its content.

```markdown
## Services

SERVICE_NAME: OrdersApi
TYPE: API
PURPOSE: Handles order placement and order status queries from external clients.
OVERVIEW_REF: `src/orders-api/overview.md`
ENTRY_POINT: `src/orders-api/main.py`
EXPOSES:
  - PROTOCOL: HTTP
    ENDPOINT_OR_TOPIC: /orders, /orders/{id}
    DESCRIPTION: REST endpoints for creating and retrieving orders
CONSUMES:
  - PROTOCOL: NONE

---

SERVICE_NAME: InventoryWorker
TYPE: Worker
PURPOSE: Listens for OrderPlaced events and updates inventory stock levels.
OVERVIEW_REF: `src/inventory-worker/overview.md`
ENTRY_POINT: `src/inventory-worker/worker.ts`
EXPOSES:
  - PROTOCOL: NONE
CONSUMES:
  - PROTOCOL: MessageQueue
    ENDPOINT_OR_TOPIC: orders.placed
    FROM_SERVICE: OrdersApi
    DESCRIPTION: Reacts to new orders to deduct stock

## Service Communication Map

INTERACTION_ID: 1
FROM_SERVICE: OrdersApi
TO_SERVICE: InventoryWorker
PROTOCOL: MessageQueue
CHANNEL: orders.placed
DIRECTION: Event
PURPOSE: Notify inventory of a new order so stock can be reserved
CONTRACT_REF: `contracts/events/order-placed.schema.json`

INTERACTION_ID: 2
FROM_SERVICE: OrdersApi
TO_SERVICE: PaymentsService
PROTOCOL: HTTP
CHANNEL: POST /payments/authorise
DIRECTION: Request-Response
PURPOSE: Authorise payment before confirming the order
CONTRACT_REF: `contracts/openapi/payments.yaml`
```

---

## Output

Write the complete document to `{project}/architecture.md`. Do not print it to the conversation.