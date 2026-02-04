# Vector Storage Integration Reference

## Overview

This reference guide provides detailed patterns for integrating Microsoft.Extensions.AI with vector storage solutions for semantic search, RAG (Retrieval-Augmented Generation), and other AI-powered applications.

## Vector Database Options

### 1. SQL Server 2025+ with Native Vector Support

**Ideal for**: Applications already using SQL Server, need ACID transactions with vectors

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.AI;

public class AppDbContext : DbContext
{
    public DbSet<Document> Documents { get; set; }
}

[Table("Documents")]
public class Document
{
    [Key]
    public int Id { get; set; }
    
    [Required, MaxLength(1000)]
    public string Title { get; set; }
    
    [Required]
    public string Content { get; set; }
    
    [Column(TypeName = "vector(1536)")]
    public SqlVector<float> Embedding { get; set; }
    
    public DateTime CreatedAt { get; set; }
}
```

### 2. PostgreSQL with pgvector

**Ideal for**: Open-source preference, cross-platform, mature vector support

```csharp
using Npgsql;
using Pgvector;
using Pgvector.EntityFrameworkCore;

public class AppDbContext : DbContext
{
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasPostgresExtension("vector");
        
        modelBuilder.Entity<Document>()
            .Property(d => d.Embedding)
            .HasColumnType("vector(1536)");
    }
}

public class Document
{
    public int Id { get; set; }
    public string Content { get; set; }
    public Vector Embedding { get; set; } // Pgvector.Vector
}
```

### 3. Azure AI Search

**Ideal for**: Azure-native apps, need full-text + vector hybrid search

```csharp
using Azure;
using Azure.Search.Documents;
using Azure.Search.Documents.Indexes;
using Microsoft.Extensions.AI;

public class DocumentIndex
{
    [SimpleField(IsKey = true, IsFilterable = true)]
    public string Id { get; set; }
    
    [SearchableField(IsFilterable = true, IsSortable = true)]
    public string Title { get; set; }
    
    [SearchableField]
    public string Content { get; set; }
    
    [VectorSearchField(VectorSearchDimensions = 1536, VectorSearchProfile = "default")]
    public IReadOnlyList<float> Embedding { get; set; }
}
```

### 4. Qdrant

**Ideal for**: Purpose-built vector database, advanced filtering, high performance

```csharp
using Qdrant.Client;
using Microsoft.Extensions.AI;

public class QdrantVectorStore
{
    private readonly QdrantClient _client;
    private const string CollectionName = "documents";
    
    public async Task CreateCollectionAsync()
    {
        await _client.CreateCollectionAsync(
            CollectionName,
            new VectorParams
            {
                Size = 1536,
                Distance = Distance.Cosine
            });
    }
}
```

## Complete RAG Implementation

### 1. Document Indexing Service

```csharp
using Microsoft.Extensions.AI;
using Microsoft.EntityFrameworkCore;

public class DocumentIndexingService
{
    private readonly AppDbContext _context;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _embeddingGenerator;
    private readonly ILogger<DocumentIndexingService> _logger;
    
    public DocumentIndexingService(
        AppDbContext context,
        IEmbeddingGenerator<string, Embedding<float>> embeddingGenerator,
        ILogger<DocumentIndexingService> logger)
    {
        _context = context;
        _embeddingGenerator = embeddingGenerator;
        _logger = logger;
    }
    
    public async Task<int> IndexDocumentAsync(string title, string content)
    {
        try
        {
            // Generate embedding
            var vector = await _embeddingGenerator.GenerateVectorAsync(content);
            
            // Create document
            var document = new Document
            {
                Title = title,
                Content = content,
                Embedding = new SqlVector<float>(vector),
                CreatedAt = DateTime.UtcNow
            };
            
            _context.Documents.Add(document);
            await _context.SaveChangesAsync();
            
            _logger.LogInformation(
                "Indexed document {Id}: {Title}", 
                document.Id, 
                document.Title);
            
            return document.Id;
        }
        catch (Exception ex)
        {
            _logger.LogError(ex, "Failed to index document: {Title}", title);
            throw;
        }
    }
    
    public async Task<int> IndexDocumentBatchAsync(
        List<(string Title, string Content)> documents)
    {
        const int batchSize = 100;
        int totalIndexed = 0;
        
        for (int i = 0; i < documents.Count; i += batchSize)
        {
            var batch = documents.Skip(i).Take(batchSize).ToList();
            
            // Generate embeddings in batch
            var contents = batch.Select(d => d.Content).ToList();
            var embeddings = await _embeddingGenerator.GenerateAsync(contents);
            
            // Create document entities
            var entities = batch.Zip(embeddings, (doc, emb) => new Document
            {
                Title = doc.Title,
                Content = doc.Content,
                Embedding = new SqlVector<float>(emb.Vector),
                CreatedAt = DateTime.UtcNow
            }).ToList();
            
            _context.Documents.AddRange(entities);
            await _context.SaveChangesAsync();
            
            totalIndexed += entities.Count;
            
            _logger.LogInformation(
                "Indexed batch of {Count} documents ({Total}/{TotalCount})",
                entities.Count,
                totalIndexed,
                documents.Count);
        }
        
        return totalIndexed;
    }
    
    public async Task ReindexDocumentAsync(int documentId)
    {
        var document = await _context.Documents.FindAsync(documentId);
        if (document == null)
            throw new InvalidOperationException($"Document {documentId} not found");
        
        // Regenerate embedding
        var vector = await _embeddingGenerator.GenerateVectorAsync(document.Content);
        document.Embedding = new SqlVector<float>(vector);
        
        await _context.SaveChangesAsync();
        
        _logger.LogInformation("Reindexed document {Id}", documentId);
    }
}
```

### 2. Semantic Search Service

```csharp
using Microsoft.Extensions.AI;
using Microsoft.EntityFrameworkCore;

public class SemanticSearchService
{
    private readonly AppDbContext _context;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _embeddingGenerator;
    private readonly ILogger<SemanticSearchService> _logger;
    
    public async Task<List<SearchResult>> SearchAsync(
        string query,
        int topK = 10,
        float similarityThreshold = 0.7f)
    {
        // Generate query embedding
        var queryVector = await _embeddingGenerator.GenerateVectorAsync(query);
        
        // Vector similarity search
        var results = await _context.Documents
            .Select(d => new
            {
                Document = d,
                Distance = EF.Functions.VectorDistance(d.Embedding, queryVector)
            })
            .Where(x => x.Distance <= (1.0 - similarityThreshold)) // Convert similarity to distance
            .OrderBy(x => x.Distance)
            .Take(topK)
            .ToListAsync();
        
        return results.Select(r => new SearchResult
        {
            DocumentId = r.Document.Id,
            Title = r.Document.Title,
            Content = r.Document.Content,
            Similarity = 1.0f - (float)r.Distance,
            Snippet = GetSnippet(r.Document.Content, query)
        }).ToList();
    }
    
    public async Task<List<SearchResult>> HybridSearchAsync(
        string query,
        int topK = 10,
        float vectorWeight = 0.7f)
    {
        // Generate query embedding for vector search
        var queryVector = await _embeddingGenerator.GenerateVectorAsync(query);
        
        // Perform hybrid search (vector + full-text)
        var results = await _context.Documents
            .Select(d => new
            {
                Document = d,
                VectorDistance = EF.Functions.VectorDistance(d.Embedding, queryVector),
                TextScore = EF.Functions.FreeText(d.Content, query) ? 1.0 : 0.0
            })
            .Select(x => new
            {
                x.Document,
                x.VectorDistance,
                x.TextScore,
                HybridScore = (vectorWeight * (1.0 - x.VectorDistance)) + 
                             ((1.0 - vectorWeight) * x.TextScore)
            })
            .OrderByDescending(x => x.HybridScore)
            .Take(topK)
            .ToListAsync();
        
        return results.Select(r => new SearchResult
        {
            DocumentId = r.Document.Id,
            Title = r.Document.Title,
            Content = r.Document.Content,
            Similarity = (float)r.HybridScore,
            Snippet = GetSnippet(r.Document.Content, query)
        }).ToList();
    }
    
    private string GetSnippet(string content, string query, int maxLength = 200)
    {
        var queryWords = query.Split(' ', StringSplitOptions.RemoveEmptyEntries);
        
        // Find first occurrence of any query word
        int startIndex = -1;
        foreach (var word in queryWords)
        {
            startIndex = content.IndexOf(
                word, 
                StringComparison.OrdinalIgnoreCase);
            if (startIndex >= 0)
                break;
        }
        
        if (startIndex < 0)
        {
            // No match, return beginning
            return content.Length <= maxLength 
                ? content 
                : content.Substring(0, maxLength) + "...";
        }
        
        // Center the snippet around the match
        int snippetStart = Math.Max(0, startIndex - (maxLength / 2));
        int snippetLength = Math.Min(maxLength, content.Length - snippetStart);
        
        var snippet = content.Substring(snippetStart, snippetLength);
        
        if (snippetStart > 0)
            snippet = "..." + snippet;
        if (snippetStart + snippetLength < content.Length)
            snippet = snippet + "...";
        
        return snippet;
    }
}

public class SearchResult
{
    public int DocumentId { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    public float Similarity { get; set; }
    public string Snippet { get; set; }
}
```

### 3. RAG Service

```csharp
using Microsoft.Extensions.AI;

public class RAGService
{
    private readonly IChatClient _chatClient;
    private readonly SemanticSearchService _searchService;
    private readonly ILogger<RAGService> _logger;
    
    public async Task<RAGResponse> AskWithContextAsync(
        string question,
        int topK = 5,
        float temperature = 0.7f)
    {
        // 1. Search for relevant documents
        var searchResults = await _searchService.SearchAsync(question, topK);
        
        if (!searchResults.Any())
        {
            return new RAGResponse
            {
                Answer = "I don't have enough context to answer this question.",
                SourceDocuments = [],
                ConfidenceScore = 0
            };
        }
        
        // 2. Build context from search results
        var context = BuildContext(searchResults);
        
        // 3. Create prompt with context
        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, """
                You are a helpful assistant that answers questions based on the provided context.
                
                Rules:
                - Only use information from the provided context
                - If the context doesn't contain enough information, say so
                - Cite sources by document title when possible
                - Be concise and accurate
                """),
            new(ChatRole.User, $"""
                Context:
                {context}
                
                Question: {question}
                """)
        };
        
        // 4. Get response from LLM
        var options = new ChatOptions
        {
            Temperature = temperature,
            MaxOutputTokens = 1000
        };
        
        var response = await _chatClient.GetResponseAsync(messages, options);
        
        // 5. Calculate confidence based on source similarity
        float avgSimilarity = searchResults.Average(r => r.Similarity);
        
        return new RAGResponse
        {
            Answer = response.Text,
            SourceDocuments = searchResults.Select(r => new SourceDocument
            {
                Title = r.Title,
                Snippet = r.Snippet,
                Similarity = r.Similarity
            }).ToList(),
            ConfidenceScore = avgSimilarity,
            TokensUsed = response.Usage?.TotalTokenCount ?? 0
        };
    }
    
    public async Task<RAGResponse> ConversationalRAGAsync(
        string question,
        List<ChatMessage> conversationHistory,
        int topK = 5)
    {
        // 1. Generate a standalone question from conversation context
        var standaloneQuestion = await GenerateStandaloneQuestionAsync(
            question, 
            conversationHistory);
        
        // 2. Search using standalone question
        var searchResults = await _searchService.SearchAsync(standaloneQuestion, topK);
        
        if (!searchResults.Any())
        {
            return new RAGResponse
            {
                Answer = "I don't have enough context to answer this question.",
                SourceDocuments = [],
                ConfidenceScore = 0
            };
        }
        
        // 3. Build context
        var context = BuildContext(searchResults);
        
        // 4. Add context to conversation
        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, """
                Answer questions based on the provided context and conversation history.
                Only use information from the context provided.
                """),
            new(ChatRole.System, $"Context:\n{context}")
        };
        
        messages.AddRange(conversationHistory);
        messages.Add(new ChatMessage(ChatRole.User, question));
        
        // 5. Get response
        var response = await _chatClient.GetResponseAsync(messages);
        
        return new RAGResponse
        {
            Answer = response.Text,
            SourceDocuments = searchResults.Select(r => new SourceDocument
            {
                Title = r.Title,
                Snippet = r.Snippet,
                Similarity = r.Similarity
            }).ToList(),
            ConfidenceScore = searchResults.Average(r => r.Similarity),
            TokensUsed = response.Usage?.TotalTokenCount ?? 0
        };
    }
    
    private async Task<string> GenerateStandaloneQuestionAsync(
        string question,
        List<ChatMessage> history)
    {
        var messages = new List<ChatMessage>
        {
            new(ChatRole.System, """
                Given a conversation history and a follow-up question, 
                rephrase the follow-up question to be a standalone question 
                that captures all necessary context.
                
                Return ONLY the rephrased question, nothing else.
                """)
        };
        
        if (history.Any())
        {
            var historyText = string.Join("\n", 
                history.Select(m => $"{m.Role}: {m.Text}"));
            messages.Add(new ChatMessage(ChatRole.User, 
                $"Conversation:\n{historyText}\n\nFollow-up: {question}"));
        }
        else
        {
            return question; // No history, use as-is
        }
        
        var response = await _chatClient.GetResponseAsync(messages);
        return response.Text.Trim();
    }
    
    private string BuildContext(List<SearchResult> results)
    {
        return string.Join("\n\n", results.Select((r, i) => 
            $"[Document {i + 1}: {r.Title}]\n{r.Content}"));
    }
}

public class RAGResponse
{
    public string Answer { get; set; }
    public List<SourceDocument> SourceDocuments { get; set; }
    public float ConfidenceScore { get; set; }
    public int TokensUsed { get; set; }
}

public class SourceDocument
{
    public string Title { get; set; }
    public string Snippet { get; set; }
    public float Similarity { get; set; }
}
```

## Advanced Patterns

### 1. Chunking Strategy

```csharp
public class DocumentChunker
{
    private const int ChunkSize = 1000;
    private const int ChunkOverlap = 200;
    
    public List<DocumentChunk> ChunkDocument(
        int documentId,
        string title,
        string content)
    {
        var chunks = new List<DocumentChunk>();
        
        // Split by paragraphs first
        var paragraphs = content.Split(
            new[] { "\n\n", "\r\n\r\n" },
            StringSplitOptions.RemoveEmptyEntries);
        
        var currentChunk = new StringBuilder();
        int chunkIndex = 0;
        
        foreach (var paragraph in paragraphs)
        {
            if (currentChunk.Length + paragraph.Length > ChunkSize)
            {
                // Save current chunk
                if (currentChunk.Length > 0)
                {
                    chunks.Add(new DocumentChunk
                    {
                        DocumentId = documentId,
                        ChunkIndex = chunkIndex++,
                        Content = currentChunk.ToString().Trim(),
                        Title = $"{title} (Part {chunkIndex})"
                    });
                    
                    // Keep overlap for context
                    var overlapText = GetLastNCharacters(
                        currentChunk.ToString(), 
                        ChunkOverlap);
                    currentChunk.Clear();
                    currentChunk.Append(overlapText);
                }
            }
            
            currentChunk.AppendLine(paragraph);
        }
        
        // Add final chunk
        if (currentChunk.Length > 0)
        {
            chunks.Add(new DocumentChunk
            {
                DocumentId = documentId,
                ChunkIndex = chunkIndex,
                Content = currentChunk.ToString().Trim(),
                Title = $"{title} (Part {chunkIndex + 1})"
            });
        }
        
        return chunks;
    }
    
    private string GetLastNCharacters(string text, int n)
    {
        if (text.Length <= n)
            return text;
        
        return text.Substring(text.Length - n);
    }
}

public class DocumentChunk
{
    public int Id { get; set; }
    public int DocumentId { get; set; }
    public int ChunkIndex { get; set; }
    public string Title { get; set; }
    public string Content { get; set; }
    
    [Column(TypeName = "vector(1536)")]
    public SqlVector<float> Embedding { get; set; }
}
```

### 2. Reranking

```csharp
public class RerankerService
{
    private readonly IChatClient _chatClient;
    
    public async Task<List<SearchResult>> RerankResultsAsync(
        string query,
        List<SearchResult> results,
        int topK = 5)
    {
        // Use LLM to rerank results based on relevance
        var resultsText = string.Join("\n\n", 
            results.Select((r, i) => $"[{i}] {r.Title}\n{r.Snippet}"));
        
        var prompt = $"""
            Given a query and a list of search results, rank the results by relevance.
            Return ONLY a JSON array of indices in order of relevance (most relevant first).
            
            Query: {query}
            
            Results:
            {resultsText}
            
            Response format: [2, 0, 4, 1, 3]
            """;
        
        var options = new ChatOptions
        {
            ResponseFormat = ChatResponseFormat.Json,
            Temperature = 0
        };
        
        var response = await _chatClient.GetResponseAsync(prompt, options);
        var indices = JsonSerializer.Deserialize<int[]>(response.Text);
        
        // Reorder results
        var reranked = indices
            .Take(topK)
            .Select(i => results[i])
            .ToList();
        
        return reranked;
    }
}
```

### 3. Caching with Invalidation

```csharp
public class CachedVectorService
{
    private readonly IDistributedCache _cache;
    private readonly IEmbeddingGenerator<string, Embedding<float>> _generator;
    private readonly TimeSpan _cacheExpiration = TimeSpan.FromHours(24);
    
    public async Task<ReadOnlyMemory<float>> GetOrCreateEmbeddingAsync(
        string text,
        CancellationToken cancellationToken = default)
    {
        var cacheKey = $"embedding:{ComputeHash(text)}";
        
        // Try to get from cache
        var cachedBytes = await _cache.GetAsync(cacheKey, cancellationToken);
        if (cachedBytes != null)
        {
            return DeserializeVector(cachedBytes);
        }
        
        // Generate embedding
        var vector = await _generator.GenerateVectorAsync(text);
        
        // Cache it
        var serialized = SerializeVector(vector);
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = _cacheExpiration
        };
        
        await _cache.SetAsync(cacheKey, serialized, options, cancellationToken);
        
        return vector;
    }
    
    private string ComputeHash(string text)
    {
        using var sha256 = SHA256.Create();
        var hashBytes = sha256.ComputeHash(Encoding.UTF8.GetBytes(text));
        return Convert.ToBase64String(hashBytes);
    }
    
    private byte[] SerializeVector(ReadOnlyMemory<float> vector)
    {
        var array = vector.ToArray();
        var bytes = new byte[array.Length * sizeof(float)];
        Buffer.BlockCopy(array, 0, bytes, 0, bytes.Length);
        return bytes;
    }
    
    private ReadOnlyMemory<float> DeserializeVector(byte[] bytes)
    {
        var floats = new float[bytes.Length / sizeof(float)];
        Buffer.BlockCopy(bytes, 0, floats, 0, bytes.Length);
        return new ReadOnlyMemory<float>(floats);
    }
}
```

## Performance Optimization

### 1. Batch Processing

```csharp
public class BatchEmbeddingService
{
    private readonly IEmbeddingGenerator<string, Embedding<float>> _generator;
    private const int MaxBatchSize = 100;
    private const int MaxConcurrentBatches = 5;
    
    public async Task<Dictionary<string, ReadOnlyMemory<float>>> GenerateBatchAsync(
        List<string> texts)
    {
        var results = new ConcurrentDictionary<string, ReadOnlyMemory<float>>();
        var semaphore = new SemaphoreSlim(MaxConcurrentBatches);
        
        var batches = texts
            .Select((text, index) => new { text, index })
            .GroupBy(x => x.index / MaxBatchSize)
            .Select(g => g.Select(x => x.text).ToList())
            .ToList();
        
        var tasks = batches.Select(async batch =>
        {
            await semaphore.WaitAsync();
            try
            {
                var embeddings = await _generator.GenerateAsync(batch);
                
                for (int i = 0; i < batch.Count; i++)
                {
                    results[batch[i]] = embeddings[i].Vector;
                }
            }
            finally
            {
                semaphore.Release();
            }
        });
        
        await Task.WhenAll(tasks);
        
        return results.ToDictionary(kvp => kvp.Key, kvp => kvp.Value);
    }
}
```

### 2. Approximate Nearest Neighbor (ANN) Indexing

```sql
-- SQL Server 2025 - Create vector index for faster search
CREATE INDEX IX_Documents_Embedding 
ON Documents(Embedding)
USING VECTOR;

-- Configure index parameters
ALTER INDEX IX_Documents_Embedding 
ON Documents
REBUILD WITH (
    DIMENSION = 1536,
    METRIC = 'cosine',
    INDEX_TYPE = 'HNSW'
);
```

## Monitoring and Observability

```csharp
public class VectorStoreMetrics
{
    private readonly ILogger<VectorStoreMetrics> _logger;
    private readonly Counter<int> _indexOperations;
    private readonly Histogram<double> _searchLatency;
    private readonly Counter<int> _cacheHits;
    
    public VectorStoreMetrics(ILogger<VectorStoreMetrics> logger)
    {
        _logger = logger;
        var meter = new Meter("VectorStore");
        
        _indexOperations = meter.CreateCounter<int>("vector_store.index_operations");
        _searchLatency = meter.CreateHistogram<double>("vector_store.search_latency_ms");
        _cacheHits = meter.CreateCounter<int>("vector_store.cache_hits");
    }
    
    public void RecordIndexOperation(string operation, bool success)
    {
        _indexOperations.Add(1, new KeyValuePair<string, object?>[]
        {
            new("operation", operation),
            new("success", success)
        });
    }
    
    public void RecordSearchLatency(double latencyMs, int resultCount)
    {
        _searchLatency.Record(latencyMs, new KeyValuePair<string, object?>[]
        {
            new("result_count", resultCount)
        });
    }
    
    public void RecordCacheHit(bool hit)
    {
        _cacheHits.Add(1, new KeyValuePair<string, object?>[]
        {
            new("hit", hit)
        });
    }
}
```

## Testing Strategies

```csharp
public class VectorStoreTests
{
    [Fact]
    public async Task TestSemanticSearch_FindsRelevantDocuments()
    {
        // Arrange
        var mockGenerator = new Mock<IEmbeddingGenerator<string, Embedding<float>>>();
        mockGenerator
            .Setup(g => g.GenerateVectorAsync(
                It.IsAny<string>(),
                It.IsAny<EmbeddingGenerationOptions>(),
                It.IsAny<CancellationToken>()))
            .ReturnsAsync(CreateMockEmbedding());
        
        var context = CreateInMemoryContext();
        var service = new SemanticSearchService(context, mockGenerator.Object, Mock.Of<ILogger<SemanticSearchService>>());
        
        // Act
        var results = await service.SearchAsync("test query");
        
        // Assert
        Assert.NotEmpty(results);
        Assert.All(results, r => Assert.True(r.Similarity > 0.5f));
    }
    
    private ReadOnlyMemory<float> CreateMockEmbedding()
    {
        return new ReadOnlyMemory<float>(
            Enumerable.Range(0, 1536)
                .Select(_ => Random.Shared.NextSingle())
                .ToArray());
    }
}
```

## Best Practices Summary

1. **Always batch operations** when processing multiple documents
2. **Use chunking** for large documents to maintain context windows
3. **Implement caching** for frequently accessed embeddings
4. **Monitor performance** with metrics and logging
5. **Use appropriate distance metrics** (cosine for text, euclidean for images)
6. **Set similarity thresholds** to filter low-quality results
7. **Consider hybrid search** (vector + keyword) for better results
8. **Implement reranking** for improved relevance
9. **Use connection pooling** for database operations
10. **Test with real data** to tune parameters
