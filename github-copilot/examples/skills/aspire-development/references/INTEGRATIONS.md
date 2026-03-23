# Aspire Integration Catalog

This document provides a comprehensive list of available .NET Aspire integrations. **Always verify current versions using `mcp_aspire_list_integrations` and `mcp_aspire_get_integration_docs` before implementation.**

> **Note**: This catalog is for reference only. Package versions and availability change frequently. Always use MCP tools to get the latest information.

## Databases

### Relational Databases
- **PostgreSQL** - `Aspire.Hosting.PostgreSQL` (v13.1.0)
  - Supports pgAdmin management UI
  - Data volume persistence
  - Health check monitoring

- **MySQL** - `Aspire.Hosting.MySql` (v13.1.0)
  - phpMyAdmin support
  - Data persistence
  - Compatible with MariaDB

- **SQL Server** - `Aspire.Hosting.SqlServer` (v13.1.0)
  - Express and Developer editions
  - Azure SQL support
  - Entity Framework Core integration

- **Oracle** - `Aspire.Hosting.Oracle` (v13.1.0)
  - Oracle Database Express Edition
  - Data volume support

- **Sqlite** - `CommunityToolkit.Aspire.Hosting.Sqlite` (v13.1.2-beta.514)
  - Lightweight embedded database
  - File-based storage

### NoSQL Databases
- **MongoDB** - `Aspire.Hosting.MongoDB` (v13.1.0)
  - MongoDB Express UI
  - Replica set support
  - Document storage

- **RavenDB** - `CommunityToolkit.Aspire.Hosting.RavenDB` (v13.1.2-beta.514)
  - Management Studio
  - Multi-model database

- **Azure Cosmos DB** - `Aspire.Hosting.Azure.CosmosDB` (v13.1.0)
  - Multiple API support (SQL, MongoDB, Cassandra)
  - Global distribution

### Cache & In-Memory Stores
- **Redis** - `Aspire.Hosting.Redis` (v13.1.0)
  - Redis Commander UI
  - Data persistence options
  - Pub/sub support

- **Garnet** - `Aspire.Hosting.Garnet` (v13.1.0)
  - Microsoft's Redis alternative
  - High performance

- **Valkey** - `Aspire.Hosting.Valkey` (v13.1.0)
  - Open-source Redis fork
  - Redis-compatible

- **Azure Redis** - `Aspire.Hosting.Azure.Redis` (v13.1.0)
  - Managed Redis service
  - Enterprise features

### Search & Analytics
- **Elasticsearch** - `Aspire.Hosting.Elasticsearch` (v13.1.0)
  - Kibana dashboard
  - Full-text search
  - Log analytics

- **Azure Search** - `Aspire.Hosting.Azure.Search` (v13.1.0)
  - AI-powered search
  - Semantic search capabilities

## Messaging & Event Streaming

### Message Brokers
- **Kafka** - `Aspire.Hosting.Kafka` (v13.1.0)
  - Kafka UI management
  - High-throughput streaming
  - Event sourcing

- **RabbitMQ** - `Aspire.Hosting.RabbitMQ` (v13.1.0)
  - Management UI
  - Queue-based messaging
  - AMQP protocol

- **Nats** - `Aspire.Hosting.Nats` (v13.1.0)
  - Lightweight messaging
  - Pub/sub and request/reply
  - Streaming support

- **ActiveMQ** - `CommunityToolkit.Aspire.Hosting.ActiveMQ` (v13.1.2-beta.514)
  - JMS support
  - Multiple protocols

- **LavinMQ** - `CommunityToolkit.Aspire.Hosting.LavinMQ` (v13.1.2-beta.514)
  - AMQP broker
  - High performance

### Azure Messaging
- **Azure Service Bus** - `Aspire.Hosting.Azure.ServiceBus` (v13.1.0)
  - Topics and queues
  - Dead-letter queues
  - Sessions and transactions

- **Azure Event Hubs** - `Aspire.Hosting.Azure.EventHubs` (v13.1.0)
  - Big data streaming
  - Apache Kafka protocol support
  - Capture to storage

## AI & Machine Learning

### AI Services
- **Azure OpenAI** - `Aspire.Hosting.OpenAI` (v13.1.0)
  - GPT models (3.5, 4, 4o)
  - DALL-E image generation
  - Embeddings

- **Azure Cognitive Services** - `Aspire.Hosting.Azure.CognitiveServices` (v13.1.0)
  - Vision, Speech, Language services
  - Custom models

- **Azure AI Foundry** - `Aspire.Hosting.Azure.AIFoundry` (v13.1.0-preview.1.25616.3)
  - Next-generation AI platform
  - Model orchestration

- **GitHub Models** - `Aspire.Hosting.GitHub.Models` (v13.1.0)
  - Access to various AI models
  - GitHub integration

- **Ollama** - `CommunityToolkit.Aspire.Hosting.Ollama` (v13.1.2-beta.514)
  - Local LLM hosting
  - Multiple model support
  - No cloud dependency

### Vector Databases
- **Milvus** - `Aspire.Hosting.Milvus` (v13.1.0)
  - High-performance vector search
  - Attu management UI
  - Similarity search

- **Qdrant** - `Aspire.Hosting.Qdrant` (v13.1.0)
  - Vector search engine
  - Web UI included
  - Filtering support

- **Meilisearch** - `CommunityToolkit.Aspire.Hosting.Meilisearch` (v13.1.2-beta.514)
  - Fast search engine
  - Typo tolerance
  - Faceted search

## Azure Cloud Services

### Storage & Data
- **Azure Storage** - `Aspire.Hosting.Azure.Storage` (v13.1.0)
  - Blob, Queue, Table storage
  - Azurite emulator support
  - File shares

- **Azure Key Vault** - `Aspire.Hosting.Azure.KeyVault` (v13.1.0)
  - Secrets management
  - Key management
  - Certificate storage

- **Azure App Configuration** - `Aspire.Hosting.Azure.AppConfiguration` (v13.1.0)
  - Feature flags
  - Dynamic configuration
  - Key-value store

### Compute & Hosting
- **Azure Functions** - `Aspire.Hosting.Azure.Functions` (v13.1.0)
  - Serverless compute
  - Event-driven execution
  - Multiple triggers

- **Azure App Containers** - `Aspire.Hosting.Azure.AppContainers` (v13.1.0)
  - Container hosting
  - Dapr support
  - Scale to zero

### Communication
- **Azure SignalR** - `Aspire.Hosting.Azure.SignalR` (v13.1.0)
  - Real-time communication
  - Scalable WebSockets
  - Broadcasting

## Language Runtimes

### JavaScript/TypeScript
- **JavaScript (Node.js)** - `Aspire.Hosting.JavaScript` (v13.1.0)
  - _Note: Formerly Aspire.Hosting.NodeJs_
  - npm and yarn support
  - Hot reload
  - Multiple entry points

- **Deno** - `CommunityToolkit.Aspire.Hosting.Deno` (v13.1.2-beta.514)
  - Secure by default
  - TypeScript native
  - Modern JS runtime

- **Bun** - `CommunityToolkit.Aspire.Hosting.Bun` (v13.1.2-beta.514)
  - Fast JavaScript runtime
  - Built-in bundler
  - npm compatible

### Other Languages
- **Python** - `Aspire.Hosting.Python` (v13.1.0)
  - Virtual environment support
  - pip package management
  - uvicorn for FastAPI

- **Golang** - `CommunityToolkit.Aspire.Hosting.Golang` (v13.1.2-beta.514)
  - Go module support
  - Cross-compilation

- **Java** - `CommunityToolkit.Aspire.Hosting.Java` (v13.1.2-beta.514)
  - Maven and Gradle support
  - Spring Boot integration

- **Rust** - `CommunityToolkit.Aspire.Hosting.Rust` (v13.1.2-beta.514)
  - Cargo support
  - Release optimization

## Infrastructure & DevOps

### Orchestration
- **Dapr** - `CommunityToolkit.Aspire.Hosting.Dapr` (v13.1.2-beta.514)
  - Distributed application runtime
  - Building blocks for microservices
  - Sidecar pattern

- **Orleans** - `Aspire.Hosting.Orleans` (v13.1.0)
  - Virtual actors
  - Distributed state management
  - Cross-platform

- **Yarp** - `Aspire.Hosting.Yarp` (v13.1.0)
  - Reverse proxy
  - Load balancing
  - Request routing

### Container Management
- **Docker** - `Aspire.Hosting.Docker` (v13.1.0-preview.1.25616.3)
  - Container orchestration
  - Image management
  - Docker Compose support

- **Kubernetes** - `Aspire.Hosting.Kubernetes` (v13.1.0-preview.1.25616.3)
  - K8s deployment
  - Helm chart support
  - Service mesh integration

### Tunneling & Networking
- **DevTunnels** - `Aspire.Hosting.DevTunnels` (v13.1.0)
  - Microsoft's tunneling service
  - Expose local services
  - Webhook testing

- **Ngrok** - `CommunityToolkit.Aspire.Hosting.Ngrok` (v13.1.2-beta.514)
  - Public URL tunneling
  - Webhook development
  - HTTPS support

## Observability & Monitoring

### Logging & Monitoring
- **Seq** - `Aspire.Hosting.Seq` (v13.1.0)
  - Structured logging
  - Search and analysis
  - Query language

- **Azure Application Insights** - `Aspire.Hosting.Azure.ApplicationInsights` (v13.1.0)
  - APM monitoring
  - Distributed tracing
  - Live metrics

- **OpenTelemetry Collector** - `CommunityToolkit.Aspire.Hosting.OpenTelemetryCollector` (v13.1.2-beta.514)
  - Telemetry collection
  - Multiple exporters
  - Vendor-neutral

## Development Tools

### Authentication & Security
- **Keycloak** - `Aspire.Hosting.Keycloak` (v13.1.0-preview.1.25616.3)
  - Identity and access management
  - SSO support
  - OAuth 2.0 / OIDC

### Database Administration
- **Adminer** - `CommunityToolkit.Aspire.Hosting.Adminer` (v13.1.2-beta.514)
  - Web-based database management
  - Multiple database support
  - Lightweight

- **DbGate** - `CommunityToolkit.Aspire.Hosting.DbGate` (v13.1.2-beta.514)
  - Database manager
  - Query builder
  - Schema comparison

### Email Testing
- **MailPit** - `CommunityToolkit.Aspire.Hosting.MailPit` (v13.1.2-beta.514)
  - Email testing
  - SMTP server
  - Web UI inbox

- **PapercutSmtp** - `CommunityToolkit.Aspire.Hosting.PapercutSmtp` (v13.1.2-beta.514)
  - Email capture
  - Local SMTP server
  - Development tool

## Package Sources

### Official Packages
Published by Microsoft, versioned as `Aspire.Hosting.*`:
- Stable releases on NuGet.org
- Full support and documentation
- Regular updates

### Community Toolkit
Published by .NET Foundation Community Toolkit, versioned as `CommunityToolkit.Aspire.*`:
- Community-maintained
- Extended integrations
- Active development
- GitHub: `CommunityToolkit/Aspire`

## Version Information

**Last Updated**: January 31, 2026  
**Aspire Version Range**: 9.5.x - 13.1.x  
**Official Packages**: v13.1.0 (stable)  
**Community Toolkit**: v13.1.2-beta.514 (preview)

> **Important**: Always use MCP tools to get current versions:
> - `mcp_aspire_list_integrations` - List all integrations
> - `mcp_aspire_get_integration_docs(packageId, version)` - Get specific documentation

## Finding More Integrations

New integrations are added regularly. To discover the latest:

1. **Query MCP**: `mcp_aspire_list_integrations`
2. **Filter by category**: Review results by type (Database, Messaging, etc.)
3. **Check NuGet**: Search for `Aspire.Hosting.*` and `CommunityToolkit.Aspire.Hosting.*`
4. **GitHub**: Browse [dotnet/aspire](https://github.com/dotnet/aspire) and [CommunityToolkit/Aspire](https://github.com/CommunityToolkit/Aspire)

---

**Remember**: This catalog is a reference guide. Always verify package availability and versions using MCP tools before implementation.
