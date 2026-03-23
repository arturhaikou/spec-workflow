# Ollama Support - Local AI Models

Microsoft Agent Framework supports **Ollama** for running local AI models without cloud dependencies.

## Why Use Ollama?

- **Cost-effective development** - No API costs during development
- **Offline capabilities** - Work without internet in air-gapped environments
- **Fast inner development loop** - Local inference with low latency
- **Privacy** - Keep data on-premises
- **Access to open-source models** - Including Microsoft's Phi-3 family

## Prerequisites

1. **.NET 8.0 SDK** or later
2. **Ollama installed** locally ([Download Ollama](https://ollama.com/))
3. **OllamaSharp NuGet package** (implements `IChatClient`)

## Recommended Models

Microsoft provides several high-quality Small Language Models (SLMs) that run efficiently with Ollama:

| Model | Description | Use Case |
|-------|-------------|----------|
| **phi3** / **phi3:mini** | Powerful SLM with groundbreaking performance at low cost and latency | General chat, coding assistance |
| **phi3.5** | Enhanced version with improved capabilities | Production-grade applications |
| **orca** | Research model for reasoning, reading comprehension, math | Complex reasoning tasks |
| **llama3.1** | Meta's Llama model | General purpose, multilingual |

## Quick Setup

### Step 1: Install Ollama

**Windows**: Download installer from https://ollama.com/  
**macOS**: `brew install ollama`  
**Linux**: `curl -fsSL https://ollama.com/install.sh | sh`

Verify installation:
```powershell
ollama
```

### Step 2: Start Ollama Service

```powershell
ollama serve  # Runs on http://localhost:11434 by default
```

### Step 3: Pull a Model

```powershell
ollama pull phi3:mini  # Downloads Microsoft Phi-3 model
```

### Step 4: Install NuGet Packages

```powershell
dotnet add package Microsoft.Agents.AI --prerelease
dotnet add package OllamaSharp  # Implements IChatClient
```

## Creating Agents with Ollama

### Basic Agent Example

```csharp
using Microsoft.Agents.AI;
using OllamaSharp;

// Create OllamaApiClient (implements IChatClient)
using OllamaApiClient chatClient = new(
    new Uri("http://localhost:11434"), 
    "phi3");

// Create agent from IChatClient
AIAgent agent = new ChatClientAgent(
    chatClient,
    instructions: "You are a helpful coding assistant.",
    name: "LocalAssistant");

// Run the agent
Console.WriteLine(await agent.RunAsync("Explain async/await in C#"));
```

### With Chat History

```csharp
using Microsoft.Extensions.AI;
using OllamaSharp;

IChatClient chatClient = new OllamaApiClient(
    new Uri("http://localhost:11434/"), 
    "phi3:mini");

List<ChatMessage> chatHistory = new();

while (true)
{
    Console.WriteLine("Your prompt:");
    var userPrompt = Console.ReadLine();
    chatHistory.Add(new ChatMessage(ChatRole.User, userPrompt));

    Console.WriteLine("AI Response:");
    var response = "";
    await foreach (ChatResponseUpdate item in 
        chatClient.GetStreamingResponseAsync(chatHistory))
    {
        Console.Write(item.Text);
        response += item.Text;
    }
    chatHistory.Add(new ChatMessage(ChatRole.Assistant, response));
    Console.WriteLine();
}
```

### With Tools/Function Calling

```csharp
AIAgent agent = new ChatClientAgent(
    chatClient,
    instructions: "You help users with date and time queries.",
    name: "TimeAssistant");

// Add tools to agent (ensure model supports function calling)
agent.AddTool(AIFunctionFactory.Create(
    () => DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"),
    "GetCurrentTime",
    "Gets the current system time"));
```

## Docker Setup for Ollama

### CPU-based Container

```powershell
docker run -d `
    -v "c:\temp\ollama:/root/.ollama" `
    -p 11434:11434 `
    --name ollama `
    ollama/ollama
```

### GPU-based Container (NVIDIA)

```powershell
docker run -d `
    --gpus=all `
    -v "c:\temp\ollama:/root/.ollama" `
    -p 11434:11434 `
    --name ollama `
    ollama/ollama
```

### Pull Models in Container

```powershell
# Open terminal in container
docker exec -it ollama bash

# Download model
ollama pull phi3
```

## IChatClient Integration

Ollama integrates seamlessly with `Microsoft.Extensions.AI.IChatClient`, enabling:

1. **Middleware Support** - Add caching, telemetry, custom logic
2. **Observability** - OpenTelemetry integration
3. **Configuration** - Default options, model selection
4. **Portability** - Swap providers without code changes

### Example with Middleware

```csharp
using Microsoft.Extensions.AI;
using Microsoft.Extensions.Caching.Distributed;
using Microsoft.Extensions.Caching.Memory;
using Microsoft.Extensions.Options;
using OllamaSharp;

var ollamaClient = new OllamaApiClient(
    new Uri("http://localhost:11434"), 
    "phi3:mini");

// Add caching and telemetry
IChatClient client = new ChatClientBuilder(ollamaClient)
    .UseDistributedCache(new MemoryDistributedCache(
        Options.Create(new MemoryDistributedCacheOptions())))
    .UseOpenTelemetry(sourceName: "MyAgent")
    .Build();

var agent = new ChatClientAgent(client, 
    instructions: "You are helpful.");
```

### Example with Configuration

```csharp
using Microsoft.Extensions.AI;
using OllamaSharp;

IChatClient client = new OllamaApiClient(new Uri("http://localhost:11434"));

client = client.AsBuilder()
    .ConfigureOptions(options => options.ModelId ??= "phi3")
    .Build();

// Will request "phi3"
Console.WriteLine(await client.GetResponseAsync("What is AI?"));

// Will request "llama3.1"
Console.WriteLine(await client.GetResponseAsync(
    "What is AI?", 
    new() { ModelId = "llama3.1" }));
```

## OpenAI Compatibility Mode

Ollama also supports OpenAI-compatible endpoints, allowing use with `OpenAIAgent`:

```csharp
// Use OpenAI agent with Ollama endpoint
// Configure endpoint to http://localhost:11434
// Ollama will translate OpenAI API calls
```

**Supported OpenAI-Compatible Services**:
- Ollama (local)
- LM Studio (local)
- LocalAI (local)
- Foundry Local (Microsoft's on-device AI)
- And others implementing OpenAI API spec

## Best Practices

### 1. Model Selection

- Use `phi3:mini` for development (small, fast)
- Use `phi3.5` for production workloads
- Verify model supports function calling if using tools

### 2. Performance

- Use GPU acceleration when available
- Consider model quantization for resource-constrained environments
- Batch requests when possible

### 3. Development Workflow

- Develop locally with Ollama
- Test with same model architecture cloud-hosted (e.g., Azure AI)
- Deploy to cloud for production scale

### 4. Resource Management

- Ollama caches models in memory after first use
- Monitor GPU/CPU usage during inference
- Use volume mounts in Docker to persist model downloads

### 5. Error Handling

- Verify Ollama service is running before agent initialization
- Handle model download timeouts gracefully
- Provide fallback to cloud providers if local inference fails

## Troubleshooting

### Common Issues

| Issue | Solution |
|-------|----------|
| "Connection refused" | Start Ollama with `ollama serve` |
| "Model not found" | Pull model with `ollama pull <model>` |
| Slow inference | Enable GPU support or use smaller model |
| Out of memory | Reduce context length or use quantized model |
| Function calling not working | Verify model supports tool use (phi3.5, llama3.1) |
| Agent timeout | Model still loading, wait or check Ollama logs |
| Docker container issues | Ensure volume mount persists, verify port mapping 11434 |

### Verification Steps

1. **Check Ollama is running**:
   ```powershell
   curl http://localhost:11434/api/tags
   ```

2. **List available models**:
   ```powershell
   ollama list
   ```

3. **Test model directly**:
   ```powershell
   ollama run phi3:mini
   ```

4. **Check Docker logs** (if using Docker):
   ```powershell
   docker logs ollama
   ```

## Integration with unifiedaitracker

### Local Development Setup

```csharp
// In unifiedaitracker.Application
// Add local AI service for development
services.AddScoped<ITicketAnalysisService>(sp =>
{
    var chatClient = new OllamaApiClient(
        new Uri("http://localhost:11434"), 
        "phi3:mini");
    
    return new ChatClientAgent(chatClient,
        instructions: "Analyze ticket content for sentiment and priority.");
});
```

### Configuration in AppHost

```csharp
// Use environment variable to toggle between Ollama and Azure OpenAI
var useLocal = builder.Configuration["UseLocalAI"] == "true";

if (useLocal)
{
    // Development: Use Ollama
    builder.Services.AddSingleton<IChatClient>(sp => 
        new OllamaApiClient(new Uri("http://localhost:11434"), "phi3:mini"));
}
else
{
    // Production: Use Azure OpenAI
    builder.AddAzureOpenAIClient("openai");
}
```

### appsettings Configuration

```json
{
  "AI": {
    "Provider": "Ollama",  // or "AzureOpenAI"
    "Ollama": {
      "Endpoint": "http://localhost:11434",
      "Model": "phi3:mini"
    },
    "AzureOpenAI": {
      "Endpoint": "https://your-resource.openai.azure.com/",
      "DeploymentName": "gpt-4o-mini"
    }
  }
}
```

## Essential Commands

```powershell
# Ollama service management
ollama serve                    # Start Ollama service
ollama list                     # List downloaded models
ollama pull phi3:mini          # Download model
ollama pull phi3.5             # Download enhanced model
ollama run phi3:mini          # Test model in CLI
ollama rm phi3:mini           # Remove model

# Docker commands
docker run -d -p 11434:11434 --name ollama ollama/ollama  # CPU
docker run -d --gpus=all -p 11434:11434 --name ollama ollama/ollama  # GPU
docker exec -it ollama bash    # Access container
docker logs ollama             # View logs
docker stop ollama             # Stop container
docker start ollama            # Start container

# Testing connectivity
curl http://localhost:11434/api/tags           # List models via API
curl http://localhost:11434/api/version        # Check version
```

## Additional Resources

- **Ollama Website**: https://ollama.com/
- **Ollama Models Library**: https://ollama.com/library
- **OllamaSharp NuGet**: https://www.nuget.org/packages/OllamaSharp/
- **Local AI Quickstart**: https://learn.microsoft.com/en-us/dotnet/ai/quickstarts/chat-local-model
- **Microsoft SLMs (Phi-3)**: https://azure.microsoft.com/products/phi-3
- **Microsoft.Extensions.AI**: https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai
- **IChatClient Interface**: https://learn.microsoft.com/en-us/dotnet/ai/ichatclient
- **Agent Framework ChatClientAgent**: https://learn.microsoft.com/en-us/agent-framework/user-guide/agents/agent-types/chat-client-agent

## Use Cases

### Offline/Air-Gapped Development
- **Scenario**: Development without cloud dependencies
- **Setup**: Ollama with downloaded models (phi3, llama3.1)
- **Benefits**: Zero API costs, data privacy, full offline capability
- **Testing**: Same agent code works with cloud providers later

### Cost-Effective Testing
- **Scenario**: Testing agent workflows without API costs
- **Setup**: Use Ollama for development, Azure OpenAI for production
- **Benefits**: Iterate quickly, test function calling, validate workflows
- **Deployment**: Switch to cloud by changing configuration only

### Privacy-Sensitive Applications
- **Scenario**: Processing sensitive data that can't leave premises
- **Setup**: Ollama on-premises with local models
- **Benefits**: Complete data control, compliance with data regulations
- **Scale**: Can run multiple instances for high throughput

### Hybrid Development
- **Scenario**: Mix local and cloud models based on workload
- **Setup**: Use Ollama for simple tasks, Azure OpenAI for complex reasoning
- **Benefits**: Optimize cost vs. quality tradeoffs
- **Implementation**: Route requests based on complexity heuristics
