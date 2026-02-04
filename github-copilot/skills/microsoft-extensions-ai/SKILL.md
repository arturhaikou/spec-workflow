---
name: microsoft-extensions-ai
description: Implement AI chat completions, embeddings, and vector storage using Microsoft.Extensions.AI library. Use when adding LLM capabilities, semantic search, RAG patterns, or AI-powered features to .NET applications. Covers IChatClient, IEmbeddingGenerator, middleware pipelines, and provider-agnostic patterns.
license: MIT
metadata:
  author: unifiedaitracker
  version: "1.0.0"
---

# Microsoft.Extensions.AI

## Overview

Microsoft.Extensions.AI provides provider-agnostic abstractions for AI services in .NET. It enables switching between providers (Azure OpenAI, OpenAI, Ollama) without code changes, and supports middleware pipelines for caching, telemetry, and function invocation.

**Core Interfaces:**
- `IChatClient` - Chat completions and streaming
- `IEmbeddingGenerator<TInput, TEmbedding>` - Generate embeddings for vector storage

## When to Use This Skill

Use this skill when you need to:
- Add LLM chat capabilities to a .NET application
- Generate embeddings for semantic search
- Implement RAG (Retrieval-Augmented Generation)
- Build provider-agnostic AI integrations
- Create middleware pipelines for AI operations
- Integrate with vector databases

## Critical Rule: Always Verify Current Documentation

**IMPORTANT:** Microsoft.Extensions.AI is actively evolving. NEVER rely on cached LLM knowledge.

**Before implementing:**
1. Search Microsoft docs: `mcp_microsoft_doc_microsoft_docs_search("Microsoft.Extensions.AI <feature>")`
2. Get code samples: `mcp_microsoft_doc_microsoft_code_sample_search("IChatClient", language="csharp")`
3. Verify API signatures from official documentation
4. Check for preview features and breaking changes

## Installation

```bash
# Core library
dotnet add package Microsoft.Extensions.AI

# Provider package (choose one)
dotnet add package Azure.AI.OpenAI         # Azure OpenAI
dotnet add package OpenAI                  # OpenAI
dotnet add package OllamaSharp             # Ollama (local)
```

## Quick Start: Chat Completions

### 1. Create a Chat Client

```csharp
using Microsoft.Extensions.AI;

// Azure OpenAI
var client = new AzureOpenAIClient(
    new Uri("https://your-resource.openai.azure.com/"),
    new AzureKeyCredential("your-api-key"))
    .AsChatClient("gpt-4");

// OR OpenAI
var client = new OpenAIClient("your-api-key")
    .AsChatClient("gpt-4");

// OR Ollama (local)
IChatClient client = new OllamaApiClient(
    new Uri("http://localhost:11434"), 
    "llama3.1");
```

### 2. Simple Chat Completion

```csharp
// Single request
var response = await client.GetResponseAsync("What is AI?");
Console.WriteLine(response.Text);

// With conversation history
List<ChatMessage> history = 
[
    new(ChatRole.System, "You are a helpful assistant"),
    new(ChatRole.User, "What is .NET?")
];
var response = await client.GetResponseAsync(history);
```

### 3. Streaming Responses

```csharp
await foreach (var update in client.GetStreamingResponseAsync("Explain quantum computing"))
{
    Console.Write(update.Text);
}
```

### 4. Maintaining Conversation Context

```csharp
List<ChatMessage> history = [];

while (true)
{
    Console.Write("Q: ");
    history.Add(new ChatMessage(ChatRole.User, Console.ReadLine()));
    
    var response = await client.GetResponseAsync(history);
    Console.WriteLine(response.Text);
    
    // Add response back to history
    history.AddMessages(response);
}
```

## Quick Start: Embeddings

### 1. Create an Embedding Generator

```csharp
using Microsoft.Extensions.AI;

var generator = new AzureOpenAIClient(/* ... */)
    .AsEmbeddingGenerator("text-embedding-3-small");

// OR
IEmbeddingGenerator<string, Embedding<float>> generator = 
    new OllamaApiClient(
        new Uri("http://localhost:11434"), 
        "nomic-embed-text");
```

### 2. Generate Embeddings

```csharp
// Single embedding
ReadOnlyMemory<float> vector = await generator.GenerateVectorAsync(
    "Machine learning is a subset of AI");

// Batch embeddings
GeneratedEmbeddings<Embedding<float>> embeddings = await generator.GenerateAsync(
[
    "First document",
    "Second document",
    "Third document"
]);
```

### 3. Vector Storage Integration

```csharp
// Example with SQL Server 2025 vectors
public class Document
{
    public int Id { get; set; }
    public string Content { get; set; }
    
    [Column(TypeName = "vector(1536)")]
    public SqlVector<float> Embedding { get; set; }
}

// Index document
var vector = await generator.GenerateVectorAsync(document.Content);
document.Embedding = new SqlVector<float>(vector);
await context.SaveChangesAsync();

// Search similar documents
var queryVector = await generator.GenerateVectorAsync(searchQuery);
var results = await context.Documents
    .OrderBy(d => EF.Functions.VectorDistance(d.Embedding, queryVector))
    .Take(5)
    .ToListAsync();
```

## Middleware Pipeline Pattern

Build composable pipelines with middleware layers:

```csharp
using Microsoft.Extensions.AI;

IChatClient client = new ChatClientBuilder(providerClient)
    .UseDistributedCache()           // Cache responses
    .UseFunctionInvocation()         // Enable tool calling
    .UseOpenTelemetry()              // Add tracing
    .UseLogging(loggerFactory)       // Log requests
    .Build();
```

**Common Middleware:**
- `.UseDistributedCache()` - Cache responses to reduce costs
- `.UseFunctionInvocation()` - Automatic tool/function calling
- `.UseOpenTelemetry()` - Distributed tracing and metrics
- `.UseLogging()` - Request/response logging
- `.ConfigureOptions()` - Set default options (temperature, model, etc.)

## Dependency Injection

Register with .NET DI container:

```csharp
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = Host.CreateApplicationBuilder();

// Register caching
builder.Services.AddDistributedMemoryCache();

// Register chat client with pipeline
builder.Services.AddChatClient(services =>
{
    var providerClient = /* create provider client */;
    
    return new ChatClientBuilder(providerClient)
        .UseDistributedCache()
        .UseOpenTelemetry()
        .Build(services);
});

// Use in services
public class MyService(IChatClient chatClient)
{
    public async Task<string> AskQuestion(string question)
    {
        var response = await chatClient.GetResponseAsync(question);
        return response.Text;
    }
}
```

## Common Patterns

### RAG (Retrieval-Augmented Generation)

```csharp
public async Task<string> AskWithContext(string question)
{
    // 1. Generate embedding for question
    var queryVector = await embeddingGenerator.GenerateVectorAsync(question);
    
    // 2. Search for relevant documents
    var relevantDocs = await context.Documents
        .OrderBy(d => EF.Functions.VectorDistance(d.Embedding, queryVector))
        .Take(5)
        .Select(d => d.Content)
        .ToListAsync();
    
    // 3. Build prompt with context
    var context = string.Join("\n\n", relevantDocs);
    List<ChatMessage> messages = 
    [
        new(ChatRole.System, "Answer based on the provided context."),
        new(ChatRole.User, $"Context:\n{context}\n\nQuestion: {question}")
    ];
    
    // 4. Get response from LLM
    var response = await chatClient.GetResponseAsync(messages);
    return response.Text;
}
```

### Function/Tool Calling

```csharp
// Define a function
[Description("Gets the current weather for a location")]
string GetWeather([Description("City name")] string city)
{
    return $"Weather in {city}: Sunny, 72°F";
}

// Use with chat client
IChatClient client = new ChatClientBuilder(providerClient)
    .UseFunctionInvocation()
    .Build();

ChatOptions options = new()
{
    Tools = [AIFunctionFactory.Create(GetWeather)]
};

var response = await client.GetResponseAsync(
    "What's the weather in Seattle?", 
    options);
```

### Structured Output

```csharp
public record SentimentAnalysis(
    [property: JsonPropertyName("sentiment")] string Sentiment,
    [property: JsonPropertyName("confidence")] float Confidence
);

var prompt = "Analyze sentiment: 'I love this product!'";
ChatOptions options = new() { ResponseFormat = ChatResponseFormat.Json };

var response = await chatClient.GetResponseAsync(prompt, options);
var result = JsonSerializer.Deserialize<SentimentAnalysis>(response.Text);
```

## Configuration Options

Set options per request or globally:

```csharp
ChatOptions options = new()
{
    ModelId = "gpt-4",
    Temperature = 0.7f,            // 0.0 = deterministic, 2.0 = creative
    MaxOutputTokens = 2000,         // Limit response length
    TopP = 0.9f,                    // Nucleus sampling
    FrequencyPenalty = 0.5f,        // Reduce repetition
    PresencePenalty = 0.5f,         // Encourage new topics
    StopSequences = ["END"]         // Stop generation at sequence
};

var response = await client.GetResponseAsync(messages, options);
```

## Testing

Mock implementations for testing:

```csharp
using Moq;

var mockClient = new Mock<IChatClient>();
mockClient
    .Setup(c => c.GetResponseAsync(
        It.IsAny<IEnumerable<ChatMessage>>(),
        It.IsAny<ChatOptions>(),
        It.IsAny<CancellationToken>()))
    .ReturnsAsync(new ChatResponse(
        new ChatMessage(ChatRole.Assistant, "Test response")));

var service = new ChatService(mockClient.Object);
```

## Error Handling

Implement retry logic for transient failures:

```csharp
public async Task<ChatResponse?> SafeGetResponse(string prompt, int retries = 3)
{
    for (int i = 0; i < retries; i++)
    {
        try
        {
            return await client.GetResponseAsync(prompt);
        }
        catch (HttpRequestException) when (i < retries - 1)
        {
            // Exponential backoff
            await Task.Delay(TimeSpan.FromSeconds(Math.Pow(2, i)));
        }
    }
    return null;
}
```

## Workflow Checklist

When implementing Microsoft.Extensions.AI features:

1. **Research** - Search Microsoft docs for current API patterns
2. **Install** - Add `Microsoft.Extensions.AI` + provider package
3. **Choose Interface** - `IChatClient` for chat, `IEmbeddingGenerator` for embeddings
4. **Create Provider Client** - Azure OpenAI, OpenAI, or Ollama
5. **Build Pipeline** - Add middleware (caching, telemetry, etc.)
6. **Configure Options** - Set temperature, tokens, model ID
7. **Register DI** - Add to service container if applicable
8. **Implement Logic** - Follow patterns (RAG, functions, etc.)
9. **Test** - Use mocks or test implementations
10. **Monitor** - Add telemetry and logging

## Reference Documentation

For detailed implementations and advanced patterns, see:

- **[references/VECTOR_STORAGE.md](references/VECTOR_STORAGE.md)** - Vector databases, semantic search, complete RAG implementation, chunking strategies
- **[references/ADVANCED_PATTERNS.md](references/ADVANCED_PATTERNS.md)** - Custom middleware, production pipelines, circuit breakers, A/B testing, benchmarking

## Key Resources

- **Official Docs**: https://learn.microsoft.com/en-us/dotnet/ai/microsoft-extensions-ai
- **IChatClient Guide**: https://learn.microsoft.com/en-us/dotnet/ai/ichatclient
- **IEmbeddingGenerator Guide**: https://learn.microsoft.com/en-us/dotnet/ai/iembeddinggenerator
- **Code Samples**: https://github.com/dotnet/ai-samples
- **Example App**: https://github.com/dotnet/eShopSupport

## Common Pitfalls

**❌ Don't hard-code providers** - Use abstractions (IChatClient, IEmbeddingGenerator)
**❌ Don't forget conversation context** - Maintain message history for stateless services
**❌ Don't ignore rate limits** - Use rate limiting middleware
**❌ Don't skip error handling** - Implement retries for transient failures
**❌ Don't trust cached knowledge** - Always verify current API documentation

**✅ Design against abstractions** for provider portability
**✅ Maintain conversation history** for context
**✅ Use middleware pipelines** for cross-cutting concerns
**✅ Implement proper error handling** with retries
**✅ Search official docs** before implementing
