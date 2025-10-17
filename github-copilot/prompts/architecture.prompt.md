---
mode: agent
---
You are an expert Solutions Architect. I need an architectural overview of the current .NET solution. Please provide a high-level description, identify key components, describe the interactions between them, and highlight important architectural patterns and considerations. Generate an `architecture.md` file containing the following sections for the Requirements Workflow project.

Structure the overview with these sections:

1.  **Project Name**
2.  **Primary Goal / Business Domain**
3.  **Technology Stack**
    *   **Primary Technology** (e.g., .NET Aspire, .NET Core, .NET 8/9)
    *   **Backend** (e.g., Web API, MVC)
    *   **Database** (e.g., PostgreSQL, Azure Cosmos DB)
    *   **Frontend (if applicable)** (e.g., React, Angular, Blazor, or "No dedicated frontend - API only")
    *   **Cloud Platform (if applicable)** (e.g., Azure, AWS, Google Cloud, or "On-premise")
    *   **Other Key Technologies** (e.g., RabbitMQ for messaging, Redis for caching, Docker, Kubernetes, Azure Functions, IdentityServer for AuthN/AuthZ)
4.  **Executive Summary** — a concise, high-level overview of the solution's purpose and its core architectural approach.
5.  **Key Architectural Drivers** — the main factors that influenced architectural decisions (derived from non-functional requirements and business goals).
6.  **Folder Structure** — describe the solution folder layout (note: this workflow uses `requirements/` rather than `specs/`) and the purpose of each folder.
7.  **Core Components**
    *   List and briefly describe each major component (e.g., Web API, Database, Frontend, Message Broker).
    *   For each, indicate its primary responsibility.
8.  **Key Features / Modules**
    *   e.g., User Authentication and Authorization
    *   e.g., Customer Management (CRUD for customers)
    *   e.g., Product Catalog
    *   e.g., Order Processing
    *   e.g., Notification Service (Email, SMS)
    *   e.g., Reporting and Analytics
9.  **Component Interactions**
    *   Describe how the main components communicate and interact.
    *   Mention protocols (e.g., HTTP/REST, gRPC, Message Queues).
10. **Architectural Patterns and Principles**
    *   Explain primary architectural patterns adopted (e.g., Layered Architecture, Microservices).
    *   Mention design principles (e.g., SOLID, Separation of Concerns).
11. **Data Flow (High-Level)** — a brief description of how data typically moves through the system for key operations.

Note: To understand the projects in the solution, refer to `overview.md` rather than reading all project files. If `overview.md` is missing or incomplete, make reasonable assumptions and list them in an "Assumptions" subsection in the generated `architecture.md`.