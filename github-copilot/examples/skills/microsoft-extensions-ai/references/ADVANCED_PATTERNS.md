# Advanced Middleware and Pipeline Patterns

## Overview

This reference provides advanced patterns for building custom middleware, complex pipelines, and production-ready implementations with Microsoft.Extensions.AI.

## Custom Middleware Patterns

### 1. Retry with Exponential Backoff

```csharp
using Microsoft.Extensions.AI;
using Polly;
using Polly.Retry;

public sealed class RetryPolicyChatClient(
    IChatClient innerClient,
    ILogger<RetryPolicyChatClient> logger) : DelegatingChatClient(innerClient)
{
    private readonly ResiliencePipeline _pipeline = new ResiliencePipelineBuilder()
        .AddRetry(new RetryStrategyOptions
        {
            MaxRetryAttempts = 3,
            Delay = TimeSpan.FromSeconds(1),
            BackoffType = DelayBackoffType.Exponential,
            UseJitter = true,
            ShouldHandle = new PredicateBuilder().Handle<HttpRequestException>()
        })
        .Build();
    
    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        return await _pipeline.ExecuteAsync(
            async ct => await base.GetResponseAsync(messages, options, ct),
            cancellationToken);
    }
    
    public override async IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        await foreach (var update in base.GetStreamingResponseAsync(
            messages, options, cancellationToken))
        {
            yield return update;
        }
    }
}

// Extension method
public static class RetryPolicyExtensions
{
    public static ChatClientBuilder UseRetryPolicy(
        this ChatClientBuilder builder,
        int maxRetries = 3) =>
        builder.Use((innerClient, services) =>
        {
            var logger = services.GetRequiredService<ILogger<RetryPolicyChatClient>>();
            return new RetryPolicyChatClient(innerClient, logger);
        });
}
```

### 2. Request/Response Validation

```csharp
public sealed class ValidationChatClient(
    IChatClient innerClient,
    ILogger<ValidationChatClient> logger) : DelegatingChatClient(innerClient)
{
    private const int MaxMessageLength = 10000;
    private const int MaxMessagesInHistory = 50;
    
    public override Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        ValidateMessages(messages);
        ValidateOptions(options);
        
        return base.GetResponseAsync(messages, options, cancellationToken);
    }
    
    private void ValidateMessages(IEnumerable<ChatMessage> messages)
    {
        var messageList = messages.ToList();
        
        if (messageList.Count == 0)
            throw new ArgumentException("Messages cannot be empty", nameof(messages));
        
        if (messageList.Count > MaxMessagesInHistory)
            throw new ArgumentException(
                $"Message history exceeds maximum of {MaxMessagesInHistory}", 
                nameof(messages));
        
        foreach (var message in messageList)
        {
            if (string.IsNullOrWhiteSpace(message.Text))
                throw new ArgumentException("Message text cannot be empty", nameof(messages));
            
            if (message.Text.Length > MaxMessageLength)
                throw new ArgumentException(
                    $"Message exceeds maximum length of {MaxMessageLength} characters",
                    nameof(messages));
        }
    }
    
    private void ValidateOptions(ChatOptions? options)
    {
        if (options == null)
            return;
        
        if (options.Temperature is < 0 or > 2)
            throw new ArgumentException("Temperature must be between 0 and 2");
        
        if (options.TopP is < 0 or > 1)
            throw new ArgumentException("TopP must be between 0 and 1");
        
        if (options.MaxOutputTokens <= 0)
            throw new ArgumentException("MaxOutputTokens must be positive");
    }
}
```

### 3. Token Budget Tracking

```csharp
public sealed class TokenBudgetChatClient : DelegatingChatClient
{
    private readonly ITokenBudgetTracker _budgetTracker;
    private readonly ILogger<TokenBudgetChatClient> _logger;
    
    public TokenBudgetChatClient(
        IChatClient innerClient,
        ITokenBudgetTracker budgetTracker,
        ILogger<TokenBudgetChatClient> logger) : base(innerClient)
    {
        _budgetTracker = budgetTracker;
        _logger = logger;
    }
    
    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        // Check budget before request
        if (!await _budgetTracker.HasBudgetAsync(cancellationToken))
        {
            _logger.LogWarning("Token budget exceeded");
            throw new InvalidOperationException("Token budget exceeded");
        }
        
        var response = await base.GetResponseAsync(messages, options, cancellationToken);
        
        // Track usage
        if (response.Usage != null)
        {
            await _budgetTracker.TrackUsageAsync(
                response.Usage.InputTokenCount,
                response.Usage.OutputTokenCount,
                cancellationToken);
            
            _logger.LogInformation(
                "Tokens used - Input: {Input}, Output: {Output}, Total: {Total}, Remaining: {Remaining}",
                response.Usage.InputTokenCount,
                response.Usage.OutputTokenCount,
                response.Usage.TotalTokenCount,
                await _budgetTracker.GetRemainingBudgetAsync(cancellationToken));
        }
        
        return response;
    }
}

public interface ITokenBudgetTracker
{
    Task<bool> HasBudgetAsync(CancellationToken cancellationToken = default);
    Task<int> GetRemainingBudgetAsync(CancellationToken cancellationToken = default);
    Task TrackUsageAsync(int inputTokens, int outputTokens, CancellationToken cancellationToken = default);
    Task ResetBudgetAsync(int newBudget, CancellationToken cancellationToken = default);
}

public class InMemoryTokenBudgetTracker : ITokenBudgetTracker
{
    private int _budget;
    private int _used;
    private readonly SemaphoreSlim _lock = new(1, 1);
    
    public InMemoryTokenBudgetTracker(int initialBudget)
    {
        _budget = initialBudget;
    }
    
    public async Task<bool> HasBudgetAsync(CancellationToken cancellationToken = default)
    {
        await _lock.WaitAsync(cancellationToken);
        try
        {
            return _used < _budget;
        }
        finally
        {
            _lock.Release();
        }
    }
    
    public async Task<int> GetRemainingBudgetAsync(CancellationToken cancellationToken = default)
    {
        await _lock.WaitAsync(cancellationToken);
        try
        {
            return Math.Max(0, _budget - _used);
        }
        finally
        {
            _lock.Release();
        }
    }
    
    public async Task TrackUsageAsync(
        int inputTokens, 
        int outputTokens, 
        CancellationToken cancellationToken = default)
    {
        await _lock.WaitAsync(cancellationToken);
        try
        {
            _used += inputTokens + outputTokens;
        }
        finally
        {
            _lock.Release();
        }
    }
    
    public async Task ResetBudgetAsync(int newBudget, CancellationToken cancellationToken = default)
    {
        await _lock.WaitAsync(cancellationToken);
        try
        {
            _budget = newBudget;
            _used = 0;
        }
        finally
        {
            _lock.Release();
        }
    }
}
```

### 4. Content Filtering and Moderation

```csharp
public sealed class ContentModerationChatClient : DelegatingChatClient
{
    private readonly IContentModerator _moderator;
    private readonly ILogger<ContentModerationChatClient> _logger;
    
    public ContentModerationChatClient(
        IChatClient innerClient,
        IContentModerator moderator,
        ILogger<ContentModerationChatClient> logger) : base(innerClient)
    {
        _moderator = moderator;
        _logger = logger;
    }
    
    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        // Check input messages
        foreach (var message in messages)
        {
            if (message.Role == ChatRole.User)
            {
                var result = await _moderator.ModerateAsync(message.Text, cancellationToken);
                
                if (result.IsBlocked)
                {
                    _logger.LogWarning(
                        "Content blocked: {Reason}", 
                        result.Reason);
                    
                    return new ChatResponse(new ChatMessage(
                        ChatRole.Assistant,
                        "I'm sorry, but I cannot process that request as it violates content policies."));
                }
            }
        }
        
        // Get response
        var response = await base.GetResponseAsync(messages, options, cancellationToken);
        
        // Check output
        if (response.Message?.Text != null)
        {
            var result = await _moderator.ModerateAsync(
                response.Message.Text, 
                cancellationToken);
            
            if (result.IsBlocked)
            {
                _logger.LogWarning(
                    "Response blocked: {Reason}", 
                    result.Reason);
                
                return new ChatResponse(new ChatMessage(
                    ChatRole.Assistant,
                    "I apologize, but I cannot provide that response."));
            }
        }
        
        return response;
    }
}

public interface IContentModerator
{
    Task<ModerationResult> ModerateAsync(
        string content, 
        CancellationToken cancellationToken = default);
}

public class ModerationResult
{
    public bool IsBlocked { get; set; }
    public string Reason { get; set; }
    public Dictionary<string, float> CategoryScores { get; set; }
}

public class AzureContentModerator : IContentModerator
{
    private readonly ContentSafetyClient _client;
    private readonly ILogger<AzureContentModerator> _logger;
    
    public async Task<ModerationResult> ModerateAsync(
        string content, 
        CancellationToken cancellationToken = default)
    {
        try
        {
            var response = await _client.AnalyzeTextAsync(content, cancellationToken);
            
            var categoryScores = new Dictionary<string, float>
            {
                ["Hate"] = response.Value.HateResult.Severity / 6f,
                ["SelfHarm"] = response.Value.SelfHarmResult.Severity / 6f,
                ["Sexual"] = response.Value.SexualResult.Severity / 6f,
                ["Violence"] = response.Value.ViolenceResult.Severity / 6f
            };
            
            bool isBlocked = categoryScores.Values.Any(score => score > 0.7f);
            
            return new ModerationResult
            {
                IsBlocked = isBlocked,
                Reason = isBlocked 
                    ? $"Content flagged: {string.Join(", ", categoryScores.Where(kvp => kvp.Value > 0.7f).Select(kvp => kvp.Key))}"
                    : null,
                CategoryScores = categoryScores
            };
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Content moderation failed");
            
            // Fail open or closed based on policy
            return new ModerationResult { IsBlocked = false };
        }
    }
}
```

### 5. Multi-Model Fallback

```csharp
public sealed class FallbackChatClient : IChatClient
{
    private readonly List<(IChatClient Client, string Name)> _clients;
    private readonly ILogger<FallbackChatClient> _logger;
    
    public FallbackChatClient(
        IEnumerable<(IChatClient, string)> clients,
        ILogger<FallbackChatClient> logger)
    {
        _clients = clients.ToList();
        _logger = logger;
        
        if (_clients.Count == 0)
            throw new ArgumentException("At least one client must be provided");
    }
    
    public ChatClientMetadata Metadata => _clients[0].Client.Metadata;
    
    public async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        Exception lastException = null;
        
        foreach (var (client, name) in _clients)
        {
            try
            {
                _logger.LogInformation("Attempting with client: {ClientName}", name);
                
                var response = await client.GetResponseAsync(
                    messages, 
                    options, 
                    cancellationToken);
                
                _logger.LogInformation("Success with client: {ClientName}", name);
                
                return response;
            }
            catch (Exception ex)
            {
                _logger.LogWarning(
                    ex, 
                    "Client {ClientName} failed, trying next", 
                    name);
                
                lastException = ex;
            }
        }
        
        _logger.LogError("All clients failed");
        throw new AggregateException(
            "All chat clients failed", 
            lastException);
    }
    
    public async IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        foreach (var (client, name) in _clients)
        {
            bool success = false;
            
            await foreach (var update in client.GetStreamingResponseAsync(
                messages, options, cancellationToken))
            {
                success = true;
                yield return update;
            }
            
            if (success)
                yield break;
        }
        
        throw new InvalidOperationException("All clients failed for streaming");
    }
    
    public TService? GetService<TService>(object? key = null) where TService : class =>
        _clients[0].Client.GetService<TService>(key);
    
    public void Dispose()
    {
        foreach (var (client, _) in _clients)
        {
            (client as IDisposable)?.Dispose();
        }
    }
}

// Registration
builder.Services.AddSingleton<IChatClient>(services =>
{
    var logger = services.GetRequiredService<ILogger<FallbackChatClient>>();
    
    var clients = new List<(IChatClient, string)>
    {
        (CreateGPT4Client(), "GPT-4"),
        (CreateGPT35Client(), "GPT-3.5"),
        (CreateOllamaClient(), "Ollama-Local")
    };
    
    return new FallbackChatClient(clients, logger);
});
```

### 6. Response Streaming with Aggregation

```csharp
public sealed class StreamAggregatorChatClient : DelegatingChatClient
{
    private readonly ILogger<StreamAggregatorChatClient> _logger;
    
    public StreamAggregatorChatClient(
        IChatClient innerClient,
        ILogger<StreamAggregatorChatClient> logger) : base(innerClient)
    {
        _logger = logger;
    }
    
    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var updates = new List<ChatResponseUpdate>();
        var combinedText = new StringBuilder();
        
        await foreach (var update in GetStreamingResponseAsync(
            messages, options, cancellationToken))
        {
            updates.Add(update);
            if (update.Text != null)
                combinedText.Append(update.Text);
        }
        
        // Convert to ChatResponse
        var finalMessage = new ChatMessage(
            updates.FirstOrDefault()?.Role ?? ChatRole.Assistant,
            combinedText.ToString());
        
        return new ChatResponse(finalMessage)
        {
            FinishReason = updates.LastOrDefault()?.FinishReason,
            ModelId = updates.FirstOrDefault()?.ModelId
        };
    }
}
```

## Complex Pipeline Compositions

### 1. Production-Ready Pipeline

```csharp
public static class ProductionChatClientFactory
{
    public static IChatClient CreateProductionClient(
        IServiceProvider services,
        IChatClient providerClient,
        ProductionOptions options)
    {
        var builder = new ChatClientBuilder(providerClient);
        
        // 1. Validation (fail fast on bad input)
        builder.Use((innerClient, svc) =>
        {
            var logger = svc.GetRequiredService<ILogger<ValidationChatClient>>();
            return new ValidationChatClient(innerClient, logger);
        });
        
        // 2. Content moderation (filter harmful content)
        if (options.EnableModeration)
        {
            builder.Use((innerClient, svc) =>
            {
                var moderator = svc.GetRequiredService<IContentModerator>();
                var logger = svc.GetRequiredService<ILogger<ContentModerationChatClient>>();
                return new ContentModerationChatClient(innerClient, moderator, logger);
            });
        }
        
        // 3. Rate limiting (prevent abuse)
        builder.UseRateLimiting();
        
        // 4. Token budget tracking (cost control)
        if (options.TokenBudget > 0)
        {
            builder.Use((innerClient, svc) =>
            {
                var tracker = new InMemoryTokenBudgetTracker(options.TokenBudget);
                var logger = svc.GetRequiredService<ILogger<TokenBudgetChatClient>>();
                return new TokenBudgetChatClient(innerClient, tracker, logger);
            });
        }
        
        // 5. Caching (reduce redundant calls)
        if (options.EnableCaching)
        {
            builder.UseDistributedCache();
        }
        
        // 6. Function invocation (enable tool use)
        if (options.EnableFunctions)
        {
            builder.UseFunctionInvocation();
        }
        
        // 7. Retry with backoff (handle transient failures)
        builder.UseRetryPolicy(maxRetries: 3);
        
        // 8. Telemetry (monitoring)
        builder.UseOpenTelemetry(
            sourceName: options.ServiceName,
            configure: c => c.EnableSensitiveData = options.LogSensitiveData);
        
        // 9. Logging (debugging)
        builder.UseLogging(
            services.GetRequiredService<ILoggerFactory>());
        
        return builder.Build(services);
    }
}

public class ProductionOptions
{
    public string ServiceName { get; set; } = "ChatService";
    public bool EnableModeration { get; set; } = true;
    public bool EnableCaching { get; set; } = true;
    public bool EnableFunctions { get; set; } = true;
    public bool LogSensitiveData { get; set; } = false;
    public int TokenBudget { get; set; } = 100000;
}
```

### 2. A/B Testing Pipeline

```csharp
public sealed class ABTestingChatClient : IChatClient
{
    private readonly IChatClient _clientA;
    private readonly IChatClient _clientB;
    private readonly IExperimentTracker _tracker;
    private readonly ILogger<ABTestingChatClient> _logger;
    private readonly double _splitRatio;
    
    public ABTestingChatClient(
        IChatClient clientA,
        IChatClient clientB,
        IExperimentTracker tracker,
        ILogger<ABTestingChatClient> logger,
        double splitRatio = 0.5)
    {
        _clientA = clientA;
        _clientB = clientB;
        _tracker = tracker;
        _logger = logger;
        _splitRatio = splitRatio;
    }
    
    public ChatClientMetadata Metadata => _clientA.Metadata;
    
    public async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        var useVariantA = Random.Shared.NextDouble() < _splitRatio;
        var client = useVariantA ? _clientA : _clientB;
        var variant = useVariantA ? "A" : "B";
        
        var stopwatch = Stopwatch.StartNew();
        
        try
        {
            var response = await client.GetResponseAsync(
                messages, 
                options, 
                cancellationToken);
            
            stopwatch.Stop();
            
            await _tracker.TrackExperimentAsync(new ExperimentResult
            {
                Variant = variant,
                Success = true,
                LatencyMs = stopwatch.ElapsedMilliseconds,
                TokenCount = response.Usage?.TotalTokenCount ?? 0,
                Timestamp = DateTime.UtcNow
            });
            
            return response;
        }
        catch (Exception ex)
        {
            stopwatch.Stop();
            
            await _tracker.TrackExperimentAsync(new ExperimentResult
            {
                Variant = variant,
                Success = false,
                LatencyMs = stopwatch.ElapsedMilliseconds,
                ErrorMessage = ex.Message,
                Timestamp = DateTime.UtcNow
            });
            
            throw;
        }
    }
    
    // Implement other members...
}

public interface IExperimentTracker
{
    Task TrackExperimentAsync(ExperimentResult result);
    Task<Dictionary<string, ExperimentMetrics>> GetMetricsAsync();
}

public class ExperimentResult
{
    public string Variant { get; set; }
    public bool Success { get; set; }
    public long LatencyMs { get; set; }
    public int TokenCount { get; set; }
    public string ErrorMessage { get; set; }
    public DateTime Timestamp { get; set; }
}

public class ExperimentMetrics
{
    public int TotalRequests { get; set; }
    public int SuccessCount { get; set; }
    public double SuccessRate => TotalRequests > 0 ? (double)SuccessCount / TotalRequests : 0;
    public double AvgLatencyMs { get; set; }
    public double AvgTokenCount { get; set; }
}
```

### 3. Circuit Breaker Pattern

```csharp
using Polly.CircuitBreaker;

public sealed class CircuitBreakerChatClient : DelegatingChatClient
{
    private readonly ResiliencePipeline _pipeline;
    private readonly ILogger<CircuitBreakerChatClient> _logger;
    
    public CircuitBreakerChatClient(
        IChatClient innerClient,
        ILogger<CircuitBreakerChatClient> logger) : base(innerClient)
    {
        _logger = logger;
        _pipeline = new ResiliencePipelineBuilder()
            .AddCircuitBreaker(new CircuitBreakerStrategyOptions
            {
                FailureRatio = 0.5,
                MinimumThroughput = 10,
                BreakDuration = TimeSpan.FromSeconds(30),
                ShouldHandle = new PredicateBuilder()
                    .Handle<HttpRequestException>()
                    .Handle<TimeoutException>(),
                OnOpened = args =>
                {
                    _logger.LogWarning("Circuit breaker opened");
                    return default;
                },
                OnClosed = args =>
                {
                    _logger.LogInformation("Circuit breaker closed");
                    return default;
                },
                OnHalfOpened = args =>
                {
                    _logger.LogInformation("Circuit breaker half-opened");
                    return default;
                }
            })
            .Build();
    }
    
    public override async Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        try
        {
            return await _pipeline.ExecuteAsync(
                async ct => await base.GetResponseAsync(messages, options, ct),
                cancellationToken);
        }
        catch (BrokenCircuitException)
        {
            _logger.LogError("Circuit breaker is open, request rejected");
            throw new InvalidOperationException(
                "Service is temporarily unavailable. Please try again later.");
        }
    }
}
```

## Testing Infrastructure

### Mock Implementations

```csharp
public class MockChatClient : IChatClient
{
    private readonly Queue<ChatResponse> _responses;
    private readonly List<IEnumerable<ChatMessage>> _receivedMessages = new();
    
    public MockChatClient(params ChatResponse[] responses)
    {
        _responses = new Queue<ChatResponse>(responses);
    }
    
    public ChatClientMetadata Metadata => new("MockClient", null, "mock-1");
    
    public IReadOnlyList<IEnumerable<ChatMessage>> ReceivedMessages => _receivedMessages;
    
    public Task<ChatResponse> GetResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        CancellationToken cancellationToken = default)
    {
        _receivedMessages.Add(messages.ToList());
        
        if (_responses.Count == 0)
            throw new InvalidOperationException("No more mock responses available");
        
        return Task.FromResult(_responses.Dequeue());
    }
    
    public async IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(
        IEnumerable<ChatMessage> messages,
        ChatOptions? options = null,
        [EnumeratorCancellation] CancellationToken cancellationToken = default)
    {
        var response = await GetResponseAsync(messages, options, cancellationToken);
        
        // Split response into updates
        var words = response.Text.Split(' ');
        foreach (var word in words)
        {
            yield return new ChatResponseUpdate(ChatRole.Assistant, word + " ");
        }
    }
    
    public TService? GetService<TService>(object? key = null) where TService : class =>
        this as TService;
    
    public void Dispose() { }
}

// Usage in tests
[Fact]
public async Task Test_Pipeline_WithMockClient()
{
    // Arrange
    var mockClient = new MockChatClient(
        new ChatResponse(new ChatMessage(ChatRole.Assistant, "Response 1")),
        new ChatResponse(new ChatMessage(ChatRole.Assistant, "Response 2"))
    );
    
    var client = new ChatClientBuilder(mockClient)
        .UseLogging(Mock.Of<ILoggerFactory>())
        .Build();
    
    // Act
    var response1 = await client.GetResponseAsync("Question 1");
    var response2 = await client.GetResponseAsync("Question 2");
    
    // Assert
    Assert.Equal("Response 1", response1.Text);
    Assert.Equal("Response 2", response2.Text);
    Assert.Equal(2, mockClient.ReceivedMessages.Count);
}
```

## Performance Benchmarking

```csharp
using BenchmarkDotNet.Attributes;
using BenchmarkDotNet.Running;

[MemoryDiagnoser]
[MinColumn, MaxColumn, MeanColumn, MedianColumn]
public class ChatClientBenchmarks
{
    private IChatClient _directClient;
    private IChatClient _pipelineClient;
    private List<ChatMessage> _messages;
    
    [GlobalSetup]
    public void Setup()
    {
        _directClient = CreateDirectClient();
        _pipelineClient = CreatePipelineClient();
        _messages = 
        [
            new(ChatRole.User, "What is AI?")
        ];
    }
    
    [Benchmark(Baseline = true)]
    public async Task<ChatResponse> DirectClient()
    {
        return await _directClient.GetResponseAsync(_messages);
    }
    
    [Benchmark]
    public async Task<ChatResponse> PipelineClient()
    {
        return await _pipelineClient.GetResponseAsync(_messages);
    }
    
    [Benchmark]
    public async Task<ChatResponse> PipelineClientWithCache()
    {
        // First call to populate cache
        await _pipelineClient.GetResponseAsync(_messages);
        // Second call from cache
        return await _pipelineClient.GetResponseAsync(_messages);
    }
}
```

## Best Practices Summary

1. **Order matters**: Place validation early, caching mid-pipeline, telemetry late
2. **Fail fast**: Validate inputs before expensive operations
3. **Use circuit breakers**: Protect downstream services
4. **Implement proper logging**: Track requests through the pipeline
5. **Monitor token usage**: Set budgets and alerts
6. **Test each layer**: Unit test middleware in isolation
7. **Use typed options**: Make configuration explicit and testable
8. **Handle failures gracefully**: Provide fallbacks where appropriate
9. **Cache strategically**: Balance cost savings vs. staleness
10. **Profile performance**: Measure overhead of middleware layers
