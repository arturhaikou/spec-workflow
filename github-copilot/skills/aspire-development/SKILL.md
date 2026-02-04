---
name: aspire-development
description: Expert guidance for .NET Aspire distributed application development. Use when adding integrations (databases, messaging, AI services), configuring AppHost orchestration, implementing service discovery, or troubleshooting Aspire applications. Emphasizes MCP tools for discovering up-to-date package information before implementation.
compatibility: Requires .NET Aspire workload, Aspire CLI, and network access for NuGet packages
metadata:
  author: unified-ai-tracker
  version: "1.0"
  aspire-version: "13.1.x"
---

# .NET Aspire Development

This skill provides comprehensive guidance for working with .NET Aspire distributed applications, including integration discovery, implementation patterns, and troubleshooting.

## When to Use This Skill

- **Adding integrations**: Databases, messaging systems, AI services, Azure resources
- **Orchestrating services**: Configuring AppHost, managing service references
- **Service discovery**: Implementing inter-service communication
- **Container management**: Working with Docker containers in Aspire
- **Troubleshooting**: Diagnosing Aspire application issues

## Critical Principle: Fresh Information First

**⚠️ LLM training data for Aspire is outdated.** The framework evolves rapidly with new integrations, breaking changes, and updated patterns.

### Mandatory Pre-Implementation Checklist

Before implementing ANY Aspire integration:

1. ✅ Query `mcp_aspire_list_integrations` to discover available integrations
2. ✅ Call `mcp_aspire_get_integration_docs(packageId, version)` for specific package documentation
3. ✅ Verify exact package IDs and versions from MCP responses
4. ✅ Review NuGet documentation URLs provided in MCP responses
5. ❌ **NEVER** rely on cached LLM knowledge for package names, versions, or patterns
6. ❌ **NEVER** guess or assume integration details

### Tool Selection Rules

**For Aspire-Specific Queries** (integrations, packages, AppHost patterns):
- `mcp_aspire_list_integrations` - List all available Aspire hosting integrations
- `mcp_aspire_get_integration_docs(packageId, version)` - Get specific integration documentation

**For General .NET Topics** (C#, EF Core, ASP.NET Core, etc.):
- `mcp_microsoft_doc_microsoft_docs_search` - Search Microsoft Learn
- `mcp_microsoft_doc_microsoft_code_sample_search` - Find code samples
- `mcp_microsoft_doc_microsoft_docs_fetch` - Fetch complete documentation

**DO NOT** use Microsoft documentation tools for Aspire integration discovery.

## Standard Workflow

### Phase 1: Discovery & Planning

1. **Identify Requirements**
   - Understand what integration or service is needed
   - Clarify configuration requirements

2. **Query Available Integrations**
   ```
   mcp_aspire_list_integrations
   → Review results, note package IDs
   → Filter by category (Database, Messaging, AI, etc.)
   ```

3. **Fetch Integration Documentation**
   ```
   mcp_aspire_get_integration_docs(packageId, version)
   → Review NuGet documentation URL
   → Verify usage patterns
   → Check prerequisites and dependencies
   ```

4. **Validate Compatibility**
   - Check version compatibility with existing packages
   - Review configuration requirements
   - Identify client packages needed

### Phase 2: Implementation

1. **Add Package to AppHost**
   ```powershell
   cd unifiedaitracker.AppHost
   dotnet add package <PackageId> --version <Version>
   ```

2. **Configure in AppHost.cs**
   ```csharp
   // Follow exact patterns from MCP documentation
   var resource = builder.Add<Integration>("resource-name")
       .WithConfiguration()
       .WithHealthCheck();
   ```

3. **Add Service References**
   ```csharp
   builder.AddProject<Projects.ApiService>("api")
       .WithReference(resource);
   ```

4. **Configure Client Services**
   - Add client NuGet packages to consuming projects
   - Configure dependency injection
   - Set up connection handling

### Phase 3: Execution & Validation

1. **Build Solution**
   ```powershell
   dotnet build unifiedaitracker.slnx
   ```

2. **Run with Aspire**
   ```powershell
   aspire run --project unifiedaitracker.AppHost
   ```

3. **Verify in Aspire Dashboard**
   - Check all services start successfully
   - Verify container health
   - Review connection status
   - Check logs for errors

4. **Test Integration**
   - Verify service discovery works
   - Test data operations
   - Validate telemetry and logs

## Quick Reference: Common Operations

### Running Aspire Applications

```powershell
# Preferred method (from solution root)
aspire run --project unifiedaitracker.AppHost

# With specific configuration
aspire run --project unifiedaitracker.AppHost --configuration Release

# Alternative (from AppHost directory)
cd unifiedaitracker.AppHost
dotnet run
```

The Aspire Dashboard automatically opens (typically `http://localhost:15xxx`) providing:
- Real-time service status
- Container health and logs
- Distributed traces
- Metrics and telemetry
- Service dependency graph

### Service Discovery Pattern

```csharp
// In AppHost.cs - Define services with names
var api = builder.AddProject<Projects.ApiService>("api");
var service = builder.AddProject<Projects.Worker>("worker")
    .WithReference(api);  // 'worker' can discover 'api'

// In consuming service - Use service name as HTTP client base address
builder.Services.AddHttpClient<IMyService, MyService>(client =>
{
    client.BaseAddress = new Uri("http://api");  // Matches AppHost name
});
```

## Integration Categories (Overview)

See [references/INTEGRATIONS.md](references/INTEGRATIONS.md) for the complete catalog (100+ integrations).

**Major Categories:**
- **Databases**: PostgreSQL, MySQL, MongoDB, SQL Server, Redis, Elasticsearch, etc.
- **Messaging**: Kafka, RabbitMQ, Azure Service Bus, NATS, etc.
- **AI Services**: Azure OpenAI, GitHub Models, Ollama, Cognitive Services
- **Vector Databases**: Milvus, Qdrant, Meilisearch
- **Azure Services**: Storage, Key Vault, Cosmos DB, Functions, etc.
- **Language Runtimes**: JavaScript/Node.js, Python, Java, Golang, Rust
- **Infrastructure**: Dapr, Orleans, Kubernetes, Docker
- **Observability**: Seq, Application Insights, OpenTelemetry
- **Development Tools**: Keycloak, Adminer, MailPit, etc.

## Common Implementation Patterns

See [references/PATTERNS.md](references/PATTERNS.md) for detailed patterns with code examples:

- Adding database integrations (with persistence and admin UI)
- Configuring Azure OpenAI
- Adding Node.js/JavaScript services
- Implementing service discovery
- Container health checks
- Environment variable configuration

## Troubleshooting

See [references/TROUBLESHOOTING.md](references/TROUBLESHOOTING.md) for detailed solutions.

**Quick Checks:**
- **Package not found**: Verify exact package ID with `mcp_aspire_list_integrations`
- **Service discovery fails**: Check service names match AppHost configuration
- **Container won't start**: Review logs in Aspire Dashboard
- **Connection string issues**: Verify `WithReference()` calls and appsettings.json

## Integration Verification Checklist

Before considering an integration complete:

- [ ] Package added with exact version from MCP
- [ ] AppHost.cs configuration matches official documentation
- [ ] Consumer services have client packages installed
- [ ] Service references configured with `WithReference()`
- [ ] Connection strings/endpoints named correctly
- [ ] Health checks configured (for containers)
- [ ] Container starts successfully in Aspire Dashboard
- [ ] Service discovery working (if applicable)
- [ ] Logs show successful connection
- [ ] No errors in Aspire Dashboard traces

## Essential Commands

```powershell
# Discover integrations (use MCP tool)
mcp_aspire_list_integrations

# Get integration docs (use MCP tool)
mcp_aspire_get_integration_docs(packageId, version)

# Add package
dotnet add package <PackageId> --version <Version>

# Build solution
dotnet build unifiedaitracker.slnx

# Run Aspire application
aspire run --project unifiedaitracker.AppHost

# Create new Aspire project
dotnet new aspire-apphost -n MyApp.AppHost

# Update Aspire workload
dotnet workload update
```

## Best Practices

1. **Always use MCP tools** for integration discovery - never rely on cached knowledge
2. **Use exact versions** from MCP responses when adding packages
3. **Configure health checks** for all container-based integrations
4. **Name resources consistently** between AppHost and consuming services
5. **Use service discovery** instead of hardcoded endpoints
6. **Monitor Aspire Dashboard** during development for immediate feedback
7. **Enable telemetry** to debug distributed application issues
8. **Persist data volumes** for databases to avoid data loss during restarts

## Additional Resources

- **[Integration Catalog](references/INTEGRATIONS.md)**: Complete list of 100+ available integrations
- **[Implementation Patterns](references/PATTERNS.md)**: Code examples for common scenarios
- **[Troubleshooting Guide](references/TROUBLESHOOTING.md)**: Solutions for common issues
- **Aspire Dashboard**: Primary tool for monitoring and debugging (auto-opens on run)
- **NuGet Package Pages**: Official documentation URLs provided by MCP tools

---

**Remember**: The cornerstone of successful Aspire development is using MCP tools (`mcp_aspire_list_integrations`, `mcp_aspire_get_integration_docs`) to get current, accurate information before every implementation.
