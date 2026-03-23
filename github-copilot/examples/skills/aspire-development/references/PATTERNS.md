# Aspire Implementation Patterns

This document provides detailed code examples and patterns for common Aspire integration scenarios. All examples assume you've already used MCP tools to verify package versions.

## Pattern 1: Adding a Database Integration (PostgreSQL)

### Complete Implementation Flow

```
Step 1: Discover Integration
└─> mcp_aspire_list_integrations
    └─> Find: "Aspire.Hosting.PostgreSQL" v13.1.0

Step 2: Get Documentation
└─> mcp_aspire_get_integration_docs("Aspire.Hosting.PostgreSQL", "13.1.0")
    └─> Review: https://www.nuget.org/packages/Aspire.Hosting.PostgreSQL/13.1.0
```

### AppHost Configuration

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Add PostgreSQL container with persistence and admin UI
var postgres = builder.AddPostgres("postgres")
    .WithDataVolume()           // Persist data across restarts
    .WithPgAdmin()              // Add pgAdmin management UI
    .WithHealthCheck();         // Enable health monitoring

// Create a named database
var database = postgres.AddDatabase("database");

// Reference database in API service
var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(database);   // Injects connection string

builder.Build().Run();
```

### Adding Package

```powershell
# Navigate to AppHost project
cd unifiedaitracker.AppHost

# Add PostgreSQL hosting package (version from MCP)
dotnet add package Aspire.Hosting.PostgreSQL --version 13.1.0
```

### Consumer Service Configuration

```csharp
// File: unifiedaitracker.ApiService/Program.cs

var builder = WebApplication.CreateBuilder(args);

// Add Npgsql client for PostgreSQL
// Note: Aspire handles connection string automatically via "database" name
builder.AddNpgsqlDbContext<AppDbContext>("database");

// Or use raw Npgsql data source
builder.AddNpgsqlDataSource("database");

var app = builder.Build();
app.Run();
```

### Consumer Service Package

```powershell
# Navigate to consuming service
cd unifiedaitracker.ApiService

# Add Npgsql Aspire component (version from MCP)
dotnet add package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL --version 13.1.0
```

## Pattern 2: Azure OpenAI Integration

### Complete Implementation Flow

```
Step 1: Discover Integration
└─> mcp_aspire_list_integrations
    └─> Find: "Aspire.Hosting.OpenAI" v13.1.0

Step 2: Get Documentation
└─> mcp_aspire_get_integration_docs("Aspire.Hosting.OpenAI", "13.1.0")
```

### AppHost Configuration

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Option 1: Connection string from configuration/secrets
var openai = builder.AddConnectionString("openai");

// Option 2: Azure OpenAI with explicit configuration
// var openai = builder.AddAzureOpenAI("openai")
//     .WithEndpoint("https://<resource>.openai.azure.com")
//     .WithApiKey("<api-key>");  // Better: Use User Secrets

var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(openai);

builder.Build().Run();
```

### User Secrets for Connection String

```powershell
# Set connection string in user secrets (recommended for local dev)
cd unifiedaitracker.AppHost

dotnet user-secrets set "ConnectionStrings:openai" "Endpoint=https://<resource>.openai.azure.com/;Key=<api-key>"
```

### Consumer Service Configuration

```csharp
// File: unifiedaitracker.ApiService/Program.cs

var builder = WebApplication.CreateBuilder(args);

// Add Azure OpenAI client
builder.AddAzureOpenAIClient("openai");

// Now you can inject OpenAIClient in services
builder.Services.AddSingleton<ISummarizationService, SummarizationService>();

var app = builder.Build();
app.Run();
```

### Using in Service Implementation

```csharp
// File: unifiedaitracker.Infrastructure/Services/SummarizationService.cs

public class SummarizationService : ISummarizationService
{
    private readonly OpenAIClient _openAIClient;
    private const string DeploymentName = "gpt-4o";  // Match your deployment

    public SummarizationService(OpenAIClient openAIClient)
    {
        _openAIClient = openAIClient;
    }

    public async Task<string> SummarizeAsync(string content)
    {
        var chatClient = _openAIClient.GetChatClient(DeploymentName);
        
        var completion = await chatClient.CompleteChatAsync(new[]
        {
            new SystemChatMessage("Summarize the following content concisely."),
            new UserChatMessage(content)
        });

        return completion.Value.Content[0].Text;
    }
}
```

### Consumer Service Package

```powershell
cd unifiedaitracker.ApiService
dotnet add package Aspire.Azure.AI.OpenAI --version 13.1.0
```

## Pattern 3: Node.js/JavaScript Service Integration

### Complete Implementation Flow

```
Step 1: Discover Integration
└─> mcp_aspire_list_integrations
    └─> Find: "Aspire.Hosting.JavaScript" v13.1.0
    └─> Note: Formerly "Aspire.Hosting.NodeJs" in older versions
```

### AppHost Configuration

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Add Node.js service using npm
var adfGenerator = builder.AddNpmApp("adfgenerator", "../adfgenerator")
    .WithHttpEndpoint(port: 3300, env: "PORT")  // Map port to env var
    .WithExternalHttpEndpoints()                 // Allow external access
    .PublishAsDockerFile();                      // Generate Dockerfile for deployment

// Alternative: Using package.json script
// var service = builder.AddNpmApp("myservice", "../myservice", "start")
//     .WithHttpEndpoint(3000);

// Reference in API
var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(adfGenerator);  // Enables service discovery

builder.Build().Run();
```

### Adding Package

```powershell
cd unifiedaitracker.AppHost
dotnet add package Aspire.Hosting.JavaScript --version 13.1.0
```

### Service Discovery in Consumer

```csharp
// File: unifiedaitracker.ApiService/Program.cs

var builder = WebApplication.CreateBuilder(args);

// Configure HTTP client for service discovery
builder.Services.AddHttpClient<IAdfConversionService, AdfConversionService>(
    (serviceProvider, client) =>
    {
        // Service name matches AppHost reference name
        // Aspire automatically resolves to actual endpoint
        client.BaseAddress = new Uri("http://adfgenerator");
    });

var app = builder.Build();
app.Run();
```

### Using the Service

```csharp
// File: unifiedaitracker.Infrastructure/Services/AdfConversionService.cs

public class AdfConversionService : IAdfConversionService
{
    private readonly HttpClient _httpClient;

    public AdfConversionService(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<string> ConvertToMarkdownAsync(string adfJson)
    {
        var response = await _httpClient.PostAsJsonAsync("/convert", new 
        { 
            adf = adfJson 
        });

        response.EnsureSuccessStatusCode();
        var result = await response.Content.ReadFromJsonAsync<ConversionResult>();
        return result.Markdown;
    }
}
```

## Pattern 4: Redis Cache Integration

### Complete Implementation Flow

```
Step 1: Discover Integration
└─> mcp_aspire_list_integrations
    └─> Find: "Aspire.Hosting.Redis" v13.1.0

Step 2: Get Documentation
└─> mcp_aspire_get_integration_docs("Aspire.Hosting.Redis", "13.1.0")
```

### AppHost Configuration

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Add Redis with management UI and persistence
var redis = builder.AddRedis("redis")
    .WithRedisCommander()       // Add Redis Commander UI
    .WithDataVolume()           // Persist data
    .WithHealthCheck();

var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(redis);

builder.Build().Run();
```

### Adding Packages

```powershell
# AppHost
cd unifiedaitracker.AppHost
dotnet add package Aspire.Hosting.Redis --version 13.1.0

# Consumer service
cd unifiedaitracker.ApiService
dotnet add package Aspire.StackExchange.Redis --version 13.1.0
```

### Consumer Service Configuration

```csharp
// File: unifiedaitracker.ApiService/Program.cs

var builder = WebApplication.CreateBuilder(args);

// Add Redis client (uses StackExchange.Redis)
builder.AddRedisClient("redis");

// Or add distributed cache based on Redis
builder.AddRedisDistributedCache("redis");

var app = builder.Build();
app.Run();
```

### Using Redis in Service

```csharp
// Using IConnectionMultiplexer (raw StackExchange.Redis)
public class CacheService
{
    private readonly IConnectionMultiplexer _redis;
    
    public CacheService(IConnectionMultiplexer redis)
    {
        _redis = redis;
    }
    
    public async Task<string> GetAsync(string key)
    {
        var db = _redis.GetDatabase();
        return await db.StringGetAsync(key);
    }
}

// Using IDistributedCache (ASP.NET Core abstraction)
public class CacheServiceAbstracted
{
    private readonly IDistributedCache _cache;
    
    public CacheServiceAbstracted(IDistributedCache cache)
    {
        _cache = cache;
    }
    
    public async Task<string> GetAsync(string key)
    {
        var bytes = await _cache.GetAsync(key);
        return bytes != null ? Encoding.UTF8.GetString(bytes) : null;
    }
}
```

## Pattern 5: Container with Custom Configuration

### Example: Elasticsearch with Custom Settings

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

var elasticsearch = builder.AddElasticsearch("elasticsearch")
    .WithDataVolume()
    .WithEnvironment("discovery.type", "single-node")  // Custom env var
    .WithEnvironment("xpack.security.enabled", "false") // Disable security for dev
    .WithHealthCheck();

// Add Kibana for visualization
var kibana = builder.AddContainer("kibana", "docker.elastic.co/kibana/kibana", "8.11.0")
    .WithHttpEndpoint(port: 5601, targetPort: 5601)
    .WithEnvironment("ELASTICSEARCH_HOSTS", elasticsearch.GetEndpoint("http"));

var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(elasticsearch);

builder.Build().Run();
```

## Pattern 6: Multi-Environment Configuration

### Using Connection Strings Per Environment

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Development: Use container
if (builder.Environment.IsDevelopment())
{
    var postgres = builder.AddPostgres("postgres")
        .WithDataVolume()
        .WithPgAdmin();
    
    var database = postgres.AddDatabase("database");
    
    builder.AddProject<Projects.ApiService>("api")
        .WithReference(database);
}
else
{
    // Production: Use connection string from configuration
    var database = builder.AddConnectionString("database");
    
    builder.AddProject<Projects.ApiService>("api")
        .WithReference(database);
}

builder.Build().Run();
```

### Configuration Files

```json
// appsettings.Production.json
{
  "ConnectionStrings": {
    "database": "Host=prod-server.database.azure.com;Database=mydb;..."
  }
}
```

## Pattern 7: Wait For Dependencies

### Ensuring Services Start in Order

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres")
    .WithDataVolume();

var database = postgres.AddDatabase("database");

// Worker runs migrations, should start before API
var worker = builder.AddProject<Projects.Worker>("worker")
    .WithReference(database);

// API waits for worker to complete migrations
var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(database)
    .WaitFor(worker);  // Won't start until worker is running

builder.Build().Run();
```

## Pattern 8: Resource Limits

### Setting Container Resource Constraints

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

var postgres = builder.AddPostgres("postgres")
    .WithDataVolume()
    .WithAnnotation(new ResourceAnnotation
    {
        Memory = 512,      // 512 MB memory limit
        CpuCount = 0.5m    // 0.5 CPU cores
    });

builder.Build().Run();
```

## Pattern 9: External Services Reference

### Connecting to External Services

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

// Reference external service by URL
var externalApi = builder.AddConnectionString("external-api");

// Or more explicitly
var externalService = builder.AddResource(new EndpointReference("external-api", 
    new Uri("https://api.external-service.com")));

var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(externalApi);

builder.Build().Run();
```

```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "external-api": "https://api.external-service.com"
  }
}
```

## Pattern 10: Environment Variables

### Passing Configuration to Services

```csharp
// File: unifiedaitracker.AppHost/AppHost.cs

var builder = DistributedApplication.CreateBuilder(args);

var api = builder.AddProject<Projects.ApiService>("api")
    .WithEnvironment("MyApp__FeatureFlag", "true")
    .WithEnvironment("MyApp__MaxConnections", "100")
    .WithEnvironment(context =>
    {
        // Dynamic environment variables
        context.EnvironmentVariables["ASPNETCORE_ENVIRONMENT"] = 
            builder.Environment.EnvironmentName;
    });

builder.Build().Run();
```

### Accessing in Service

```csharp
// File: unifiedaitracker.ApiService/Program.cs

var builder = WebApplication.CreateBuilder(args);

// Read from configuration (environment variables automatically loaded)
var featureFlag = builder.Configuration["MyApp:FeatureFlag"];
var maxConnections = builder.Configuration.GetValue<int>("MyApp:MaxConnections");
```

## Best Practices Summary

1. **Always use MCP tools first** - Verify package names and versions
2. **Use meaningful resource names** - They become connection string names
3. **Enable health checks** for containers - Monitor service health
4. **Persist data with volumes** - Avoid data loss during development
5. **Use management UIs** during development - pgAdmin, Redis Commander, etc.
6. **Configure service discovery properly** - Match AppHost names in HTTP clients
7. **Use User Secrets** for sensitive data - Never commit credentials
8. **Leverage WaitFor** for dependencies - Ensure proper startup order
9. **Configure resource limits** - Prevent resource exhaustion
10. **Test with Aspire Dashboard** - Monitor logs and traces during development

## Next Steps

- Review [INTEGRATIONS.md](INTEGRATIONS.md) for complete integration catalog
- Check [TROUBLESHOOTING.md](TROUBLESHOOTING.md) for common issues
- Use MCP tools to discover latest integrations and patterns

---

**Remember**: These patterns are examples. Always verify with MCP tools and official documentation before implementation.
