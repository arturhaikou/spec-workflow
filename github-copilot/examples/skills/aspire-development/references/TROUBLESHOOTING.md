# Aspire Troubleshooting Guide

Common issues and solutions when working with .NET Aspire distributed applications.

## Package and Integration Issues

### Issue: Package Not Found

**Symptoms:**
- `dotnet add package` fails with "Unable to find package"
- Build errors about missing NuGet packages

**Solution:**
```powershell
# Step 1: Verify package exists with correct name
# Use MCP tool: mcp_aspire_list_integrations

# Step 2: Check exact package ID and version
# Use MCP tool: mcp_aspire_get_integration_docs(packageId, version)

# Step 3: Verify NuGet sources
dotnet nuget list source

# Step 4: Add package with exact version from MCP
dotnet add package <ExactPackageId> --version <ExactVersion>

# Step 5: If still failing, clear NuGet cache
dotnet nuget locals all --clear
dotnet restore
```

**Common Causes:**
- Typo in package name (e.g., "AspireHosting.Redis" instead of "Aspire.Hosting.Redis")
- Incorrect version number
- NuGet source not configured
- Package is preview/beta and requires explicit version

### Issue: Integration Package Version Mismatch

**Symptoms:**
- Build warnings about incompatible package versions
- Runtime errors about missing dependencies
- Aspire Dashboard shows service in error state

**Solution:**
```powershell
# Check all Aspire package versions in solution
dotnet list package | findstr Aspire

# Update all Aspire packages to consistent version
# Target stable version: 13.1.0
dotnet add package Aspire.Hosting.PostgreSQL --version 13.1.0
dotnet add package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL --version 13.1.0

# Or use MCP to get latest compatible versions
# mcp_aspire_list_integrations
```

**Best Practice:**
- Keep all Aspire packages at same version within a project
- Use MCP tools to verify versions before adding packages

### Issue: Wrong Package Imported (Hosting vs Client)

**Symptoms:**
- Build errors: "Cannot resolve method 'AddPostgres'"
- Missing extension methods
- Namespace not found

**Solution:**

**In AppHost project** (orchestration):
```powershell
# Use Aspire.Hosting.* packages
dotnet add package Aspire.Hosting.PostgreSQL --version 13.1.0
```

**In consuming service** (client):
```powershell
# Use Aspire.<Provider>.* packages
dotnet add package Aspire.Npgsql.EntityFrameworkCore.PostgreSQL --version 13.1.0
```

**Reference:**
- **Hosting packages**: `Aspire.Hosting.*` - Used in AppHost for orchestration
- **Client packages**: `Aspire.<Provider>.*` - Used in services for consumption
- **Example**: 
  - Hosting: `Aspire.Hosting.Redis` (AppHost)
  - Client: `Aspire.StackExchange.Redis` (ApiService)

## Service Discovery Issues

### Issue: Service Discovery Not Working

**Symptoms:**
- HTTP client throws connection refused
- "No such host is known" error
- 404 errors when calling other services

**Solution:**

**Step 1: Verify AppHost Configuration**
```csharp
// AppHost.cs - Check resource names are defined
var api = builder.AddProject<Projects.ApiService>("api");  // Name: "api"
var worker = builder.AddProject<Projects.Worker>("worker")
    .WithReference(api);  // Worker can discover "api"
```

**Step 2: Verify HTTP Client Configuration**
```csharp
// Worker/Program.cs - Use exact name from AppHost
builder.Services.AddHttpClient<IMyService, MyService>(client =>
{
    // Must match AppHost name exactly (case-sensitive)
    client.BaseAddress = new Uri("http://api");  
});
```

**Step 3: Check Service Registration in Aspire Dashboard**
- Open Aspire Dashboard (auto-opens on run)
- Navigate to **Resources** tab
- Verify both services appear in the list
- Check service URLs and endpoints

**Common Mistakes:**
- Case mismatch: `"API"` vs `"api"`
- Typo in service name
- Service not added to AppHost
- Missing `WithReference()` call

### Issue: Connection String Not Found

**Symptoms:**
- "ConnectionStrings:database configuration is missing" error
- Service fails to start
- Database connection errors

**Solution:**

**Step 1: Verify AppHost Reference**
```csharp
// AppHost.cs
var postgres = builder.AddPostgres("postgres");
var database = postgres.AddDatabase("database");  // Connection name: "database"

var api = builder.AddProject<Projects.ApiService>("api")
    .WithReference(database);  // Must have this!
```

**Step 2: Verify Consumer Configuration**
```csharp
// ApiService/Program.cs - Use same name
builder.AddNpgsqlDbContext<AppDbContext>("database");  // Match AppHost name
```

**Step 3: Check Connection String Exists**
```csharp
// Debug helper in Program.cs
var connectionString = builder.Configuration.GetConnectionString("database");
Console.WriteLine($"Database connection: {connectionString}");
```

**Alternative: Manual Connection String**
```json
// appsettings.Development.json
{
  "ConnectionStrings": {
    "database": "Host=localhost;Database=mydb;Username=postgres;Password=postgres"
  }
}
```

## Container and Runtime Issues

### Issue: Container Fails to Start

**Symptoms:**
- Container shows "Exited" or "Failed" in Aspire Dashboard
- Red status indicator
- Service never becomes healthy

**Solution:**

**Step 1: Check Container Logs**
1. Open Aspire Dashboard
2. Navigate to **Resources** tab
3. Click on failed container
4. Select **Logs** tab
5. Review error messages

**Step 2: Common Container Issues**

**Port Conflict:**
```
Error: Port 5432 is already in use
```
Solution: Stop existing service using that port or change port:
```csharp
var postgres = builder.AddPostgres("postgres")
    .WithEndpoint(port: 5433);  // Use different port
```

**Volume Mount Issues:**
```
Error: Cannot mount volume
```
Solution: Check volume permissions or remove volume temporarily:
```csharp
var postgres = builder.AddPostgres("postgres");  // No .WithDataVolume()
```

**Image Pull Failure:**
```
Error: Unable to pull image
```
Solution: Check Docker is running and internet connectivity:
```powershell
docker pull postgres:16
```

**Step 3: Docker Health Check**
```powershell
# Verify Docker is running
docker ps

# Check Docker daemon
docker info

# Restart Docker if needed
# (Restart Docker Desktop on Windows)
```

### Issue: Container Starts but Health Check Fails

**Symptoms:**
- Container status "Unhealthy" in Dashboard
- Service dependencies won't start (using WaitFor)

**Solution:**

**Step 1: Review Health Check Configuration**
```csharp
// AppHost.cs - Verify health check is configured correctly
var postgres = builder.AddPostgres("postgres")
    .WithHealthCheck();  // Uses default health check

// Custom health check (if needed)
var redis = builder.AddRedis("redis")
    .WithHealthCheck(timeout: TimeSpan.FromSeconds(30));
```

**Step 2: Check Container Readiness**
- Some containers take time to initialize (databases especially)
- View logs to confirm startup complete
- Default health check might be too aggressive

**Step 3: Temporary Workaround**
```csharp
// Remove health check temporarily for debugging
var postgres = builder.AddPostgres("postgres");  // No .WithHealthCheck()
```

### Issue: Volume Data Not Persisting

**Symptoms:**
- Database data lost after restart
- Container starts with empty database

**Solution:**

**Step 1: Verify Volume Configuration**
```csharp
// AppHost.cs - Ensure volume is configured
var postgres = builder.AddPostgres("postgres")
    .WithDataVolume();  // Must have this for persistence
```

**Step 2: Check Volume Exists**
```powershell
# List Docker volumes
docker volume ls

# Inspect specific volume
docker volume inspect <volume-name>
```

**Step 3: Named Volumes (for explicit control)**
```csharp
var postgres = builder.AddPostgres("postgres")
    .WithDataVolume("my-postgres-data");  // Explicit volume name
```

## Build and Deployment Issues

### Issue: Aspire Workload Not Installed

**Symptoms:**
- `aspire` command not found
- Missing project templates
- Build errors about Aspire SDK

**Solution:**
```powershell
# Install Aspire workload
dotnet workload install aspire

# Verify installation
dotnet workload list

# Update to latest
dotnet workload update

# If issues persist, repair
dotnet workload repair
```

### Issue: Multiple Aspire Hosts in Solution

**Symptoms:**
- Build errors about duplicate AppHost
- Confusion about which AppHost to run

**Solution:**
- Only one AppHost per solution (by design)
- If you need multiple environments, use configuration:

```csharp
// AppHost.cs
var builder = DistributedApplication.CreateBuilder(args);

if (builder.Environment.IsDevelopment())
{
    // Development-specific configuration
}
else if (builder.Environment.IsProduction())
{
    // Production-specific configuration
}
```

### Issue: Project Reference Not Found

**Symptoms:**
- Build error: "Projects.ApiService does not exist"
- Cannot add project with AddProject<>

**Solution:**

**Step 1: Verify Project Reference in AppHost.csproj**
```xml
<ItemGroup>
  <ProjectReference Include="..\unifiedaitracker.ApiService\unifiedaitracker.ApiService.csproj" />
</ItemGroup>
```

**Step 2: Add Missing Reference**
```powershell
cd unifiedaitracker.AppHost
dotnet add reference ..\unifiedaitracker.ApiService\unifiedaitracker.ApiService.csproj
```

**Step 3: Rebuild Solution**
```powershell
dotnet build unifiedaitracker.slnx
```

## Configuration Issues

### Issue: Environment Variables Not Loading

**Symptoms:**
- Configuration values are null
- Service can't find expected configuration

**Solution:**

**Step 1: Verify Environment Variable Syntax**
```csharp
// AppHost.cs - Check environment variable naming
var api = builder.AddProject<Projects.ApiService>("api")
    .WithEnvironment("MyApp__Setting", "value");  // Note: Double underscore
```

**Step 2: Access in Service**
```csharp
// Program.cs - Configuration uses colon notation
var setting = builder.Configuration["MyApp:Setting"];  // Colon, not underscore
```

**Configuration Binding:**
- Environment variables use `__` (double underscore)
- Configuration keys use `:` (colon)
- `MyApp__FeatureFlag__Enabled` → `MyApp:FeatureFlag:Enabled`

### Issue: User Secrets Not Loading

**Symptoms:**
- Connection strings work for others but not for you
- "Configuration is missing" errors

**Solution:**

**Step 1: Initialize User Secrets**
```powershell
cd unifiedaitracker.AppHost
dotnet user-secrets init
```

**Step 2: Set Secrets**
```powershell
dotnet user-secrets set "ConnectionStrings:openai" "Endpoint=https://...;Key=..."
```

**Step 3: List Secrets (Verify)**
```powershell
dotnet user-secrets list
```

**Step 4: Check .csproj Has UserSecretsId**
```xml
<!-- Should be auto-added by init -->
<UserSecretsId>aspire-12345678-1234-1234-1234-123456789012</UserSecretsId>
```

## Performance Issues

### Issue: Slow Startup Time

**Symptoms:**
- Aspire takes minutes to start
- Services timeout during startup

**Common Causes & Solutions:**

**1. Too Many Containers**
```csharp
// Consider using connection strings for external services instead
// Development:
if (builder.Environment.IsDevelopment())
{
    var postgres = builder.AddPostgres("postgres");  // Container
}
else
{
    var postgres = builder.AddConnectionString("database");  // External
}
```

**2. Large Docker Images**
- Use specific image tags instead of `latest`
- Review image sizes: `docker images`

**3. Resource Constraints**
- Increase Docker Desktop resources (Memory, CPU)
- Close unnecessary applications

### Issue: High Memory Usage

**Symptoms:**
- System runs out of memory
- Docker containers killed

**Solution:**

**Step 1: Set Resource Limits**
```csharp
// AppHost.cs - Limit container resources
var postgres = builder.AddPostgres("postgres")
    .WithAnnotation(new ContainerAnnotation
    {
        Memory = 512,      // 512 MB
        CpuCount = 0.5m    // Half a core
    });
```

**Step 2: Check Docker Settings**
- Docker Desktop → Settings → Resources
- Increase memory limit if needed
- Default: 2GB (recommended: 4GB+ for Aspire)

## Dashboard Issues

### Issue: Aspire Dashboard Won't Open

**Symptoms:**
- Dashboard URL doesn't open automatically
- Browser shows connection refused

**Solution:**

**Step 1: Find Dashboard URL**
```
Console output shows:
Aspire Dashboard: http://localhost:15234
```

**Step 2: Manually Open**
- Copy URL from console
- Open in browser

**Step 3: Check Port Not in Use**
```powershell
# Windows
netstat -ano | findstr :15234

# If port in use, Aspire will choose different port
# Check console for actual URL
```

### Issue: Dashboard Shows No Resources

**Symptoms:**
- Dashboard opens but no services listed
- Resources tab is empty

**Solution:**

**Check AppHost is Running:**
- Verify `aspire run` command completed successfully
- Check for build errors in console
- Ensure AppHost.cs has `builder.Build().Run()`

**Rebuild Solution:**
```powershell
dotnet clean
dotnet build unifiedaitracker.slnx
aspire run --project unifiedaitracker.AppHost
```

## Advanced Diagnostics

### Enable Detailed Logging

```csharp
// AppHost.cs
var builder = DistributedApplication.CreateBuilder(args);

// Enable detailed logging for diagnostics
builder.Services.AddLogging(logging =>
{
    logging.SetMinimumLevel(LogLevel.Debug);
});
```

### Inspect Service Configuration

```csharp
// Service/Program.cs - Debug configuration values
var builder = WebApplication.CreateBuilder(args);

// Log all configuration (development only!)
if (builder.Environment.IsDevelopment())
{
    foreach (var kvp in builder.Configuration.AsEnumerable())
    {
        Console.WriteLine($"{kvp.Key}: {kvp.Value}");
    }
}
```

### Docker Diagnostics

```powershell
# List all containers
docker ps -a

# View container logs
docker logs <container-id>

# Inspect container
docker inspect <container-id>

# Check networks
docker network ls
docker network inspect <network-name>

# Clean up stopped containers
docker container prune

# Clean up unused volumes
docker volume prune
```

## Getting Help

### Information to Gather

When seeking help, collect:

1. **Aspire Version**: `dotnet workload list | findstr aspire`
2. **Docker Version**: `docker --version`
3. **OS/Environment**: Windows, Linux, macOS
4. **Error Messages**: Full stack trace from logs
5. **Dashboard Status**: Screenshot of Aspire Dashboard
6. **Container Logs**: From Dashboard or `docker logs`
7. **AppHost Configuration**: Relevant AppHost.cs code

### Useful Commands for Diagnostics

```powershell
# List all Aspire packages in solution
dotnet list package | findstr Aspire

# Check .NET SDK version
dotnet --version

# Check installed workloads
dotnet workload list

# Verify project builds
dotnet build unifiedaitracker.slnx --verbosity detailed

# Run with detailed output
aspire run --project unifiedaitracker.AppHost --verbosity diagnostic
```

## Prevention Tips

1. **Always use MCP tools** before adding packages
2. **Keep Aspire packages at same version** across solution
3. **Test with Aspire Dashboard** during development
4. **Use health checks** for all containers
5. **Configure resource limits** to prevent resource exhaustion
6. **Enable logging** appropriate for environment
7. **Use User Secrets** for sensitive data
8. **Document custom configurations** for team members
9. **Regularly update workloads**: `dotnet workload update`
10. **Monitor Dashboard** for warnings and errors

---

**Still stuck?** 
- Check [PATTERNS.md](PATTERNS.md) for implementation examples
- Review [INTEGRATIONS.md](INTEGRATIONS.md) for integration-specific notes
- Use MCP tools to verify latest package information
- Review official Aspire documentation at https://learn.microsoft.com/dotnet/aspire/
