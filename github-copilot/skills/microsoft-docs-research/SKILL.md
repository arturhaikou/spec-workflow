---
name: microsoft-docs-research
description: Expert guidance for efficiently gathering information from official Microsoft/Azure documentation using MCP tools. Use when implementing Microsoft/Azure features, troubleshooting issues, learning SDKs, verifying API signatures, or finding official code samples. Ensures you work with current documentation rather than outdated cached knowledge.
compatibility: Requires network access to Microsoft Learn and active MCP server for Microsoft documentation tools
metadata:
  author: unified-ai-tracker
  version: "1.0"
  mcp-tools: "microsoft-docs (search, fetch, code-sample-search)"
---

# Microsoft Documentation Research Skill

This skill provides expert guidance for efficiently gathering information from official Microsoft documentation using MCP (Model Context Protocol) tools.

## Overview

Microsoft maintains extensive documentation for all its products, services, and developer platforms. The MCP tools provide direct access to this authoritative content, ensuring you always work with the latest official information.

## Critical Principle: Fresh Information First

**⚠️ LLM training data for Microsoft technologies is outdated.** Microsoft SDKs, APIs, and services evolve rapidly with breaking changes, new features, and updated best practices.

### Mandatory Pre-Implementation Checklist

Before implementing ANY Microsoft/Azure feature:

1. ✅ Search `mcp_microsoft_doc_microsoft_docs_search` for current concepts and patterns
2. ✅ Query `mcp_microsoft_doc_microsoft_code_sample_search` with language parameter for official code examples
3. ✅ Verify package versions and API signatures from documentation
4. ✅ Fetch complete guides with `mcp_microsoft_doc_microsoft_docs_fetch` when needed
5. ❌ **NEVER** rely on cached LLM knowledge for Microsoft SDKs, APIs, or configurations
6. ❌ **NEVER** guess or assume API signatures, package versions, or patterns

### Why This Matters

**Cached knowledge is unreliable**:
- NuGet packages update frequently with breaking changes
- API signatures change between versions
- Configuration patterns get deprecated
- New best practices replace old approaches
- Security recommendations evolve

**Example**: Your training data might show `AddJwtBearer()` configuration from .NET 6, but .NET 8 introduced new required parameters and deprecated old options. Using outdated patterns leads to compilation errors or runtime failures.

### Tool Selection Rules

**For Microsoft/Azure Topics** (SDKs, services, platforms):
- `mcp_microsoft_doc_microsoft_docs_search` - Search Microsoft Learn documentation
- `mcp_microsoft_doc_microsoft_code_sample_search` - Find official code samples (with language filter)
- `mcp_microsoft_doc_microsoft_docs_fetch` - Retrieve complete documentation pages

**For VS Code Extension Development**:
- `get_vscode_api` - VS Code API documentation

**DO NOT** use these tools for third-party libraries or non-Microsoft technologies.

## Available MCP Tools

### 1. `mcp_microsoft_doc_microsoft_docs_search`
**Purpose**: Search official Microsoft/Azure documentation for relevant content.

**When to use**:
- Getting started with new Microsoft technologies
- Understanding concepts and architecture
- Finding API references and SDK documentation
- Looking for best practices and patterns
- Troubleshooting common issues
- Verifying feature availability and requirements

**Returns**: Up to 10 high-quality content chunks (max 500 tokens each) with article titles, URLs, and excerpts.

**Example queries**:
```
- "EF Core PostgreSQL configuration"
- "Azure OpenAI SDK authentication"
- "ASP.NET Core JWT authentication"
- "SignalR hub configuration"
- "C# record types best practices"
```

### 2. `mcp_microsoft_doc_microsoft_code_sample_search`
**Purpose**: Retrieve code samples from official Microsoft Learn documentation.

**When to use**:
- Implementing specific Microsoft/Azure features
- Learning SDK usage patterns
- Finding working code examples
- Understanding API syntax
- Following best practices for specific scenarios

**Supports languages**: `csharp`, `javascript`, `typescript`, `python`, `powershell`, `azurecli`, `sql`, `java`, `kusto`, `cpp`, `go`, `rust`, `ruby`, `php`

**Best practice**: Always specify the `language` parameter for better results.

**Example queries**:
```
- query: "Entity Framework Core DbContext", language: "csharp"
- query: "Azure OpenAI chat completion", language: "csharp"
- query: "PostgreSQL connection string", language: "csharp"
```

### 3. `mcp_microsoft_doc_microsoft_docs_fetch`
**Purpose**: Fetch complete documentation pages in markdown format.

**When to use**:
- Search results show truncated content
- Need complete step-by-step tutorials
- Require full troubleshooting sections
- Want comprehensive API reference
- Need to see all prerequisites and notes
- Search identified a highly relevant page

**URL requirements**: Must be from the `microsoft.com` domain.

**Best practice**: Use this AFTER search when you identify specific high-value pages.

## Critical Rules

<rules>
    <rule type="critical">
        **Mandatory Pre-Check (See: Critical Principle Above)**
        - Follow the mandatory pre-implementation checklist before ANY Microsoft/Azure implementation
        - NEVER skip documentation verification to save time
        - Your training data is outdated - always search official documentation first
    </rule>

    <rule type="critical">
        **Search Before Code**
        - Before writing any Microsoft/Azure related code, search documentation first
        - Use `microsoft_code_sample_search` with language parameter to find official code examples
        - Follow the patterns from official samples, not from memory
        - Verify package versions and API signatures match current documentation
    </rule>

    <rule type="critical">
        **Two-Step Research Pattern**
        1. **Search**: Use `microsoft_docs_search` or `microsoft_code_sample_search` to find relevant pages
        2. **Fetch**: Use `microsoft_docs_fetch` on identified high-value pages for complete information
        
        **DO NOT** skip to fetch without searching first unless you already know the exact URL.
    </rule>

    <rule type="restriction">
        **Scope Limitation**
        - These tools are for **Microsoft/Azure documentation ONLY**
        - For third-party libraries: Use web search or GitHub tools
        - For VS Code extension development: Use `get_vscode_api` tool
    </rule>
</rules>

## Recommended Workflows

### Workflow 1: Implementing a New Feature

```
1. Identify Technology
   └─> Example: "Need to add JWT authentication in ASP.NET Core"

2. Search for Concepts
   └─> mcp_microsoft_doc_microsoft_docs_search("ASP.NET Core JWT authentication")
   └─> Review returned chunks to understand approach

3. Find Code Samples
   └─> mcp_microsoft_doc_microsoft_code_sample_search(
         query: "JWT authentication middleware",
         language: "csharp"
       )
   └─> Review official code patterns

4. Get Complete Guide (if needed)
   └─> mcp_microsoft_doc_microsoft_docs_fetch(url: <identified-url>)
   └─> Get full tutorial with all steps

5. Implement
   └─> Follow official patterns from documentation
   └─> Use verified package versions
   └─> Apply recommended configurations
```

### Workflow 2: Troubleshooting an Issue

```
1. Search for Error/Issue
   └─> mcp_microsoft_doc_microsoft_docs_search("EF Core migration error PostgreSQL")
   └─> Look for troubleshooting sections

2. Fetch Troubleshooting Guide
   └─> If search finds relevant page, fetch complete content
   └─> mcp_microsoft_doc_microsoft_docs_fetch(url: <troubleshooting-page>)

3. Search for Related Code
   └─> mcp_microsoft_doc_microsoft_code_sample_search(
         query: "EF Core PostgreSQL configuration",
         language: "csharp"
       )
   └─> Compare with your implementation

4. Apply Solution
   └─> Follow official recommendations
   └─> Update configuration based on documentation
```

### Workflow 3: Learning New SDK/Service

```
1. Start with Overview
   └─> mcp_microsoft_doc_microsoft_docs_search("Azure OpenAI SDK getting started")
   └─> Understand architecture and concepts

2. Fetch Quickstart
   └─> Identify quickstart guide from search results
   └─> mcp_microsoft_doc_microsoft_docs_fetch(url: <quickstart-url>)
   └─> Get complete setup instructions

3. Explore Code Samples
   └─> mcp_microsoft_doc_microsoft_code_sample_search(
         query: "Azure OpenAI chat completion",
         language: "csharp"
       )
   └─> Study official implementation patterns

4. Search for Specific Features
   └─> mcp_microsoft_doc_microsoft_docs_search("Azure OpenAI streaming responses")
   └─> Learn specific capabilities

5. Implement with Confidence
   └─> Use verified patterns and package versions
   └─> Follow official best practices
```

## Best Practices

### Search Query Design

**Good queries** (specific and descriptive):
- ✅ "EF Core many-to-many relationship configuration"
- ✅ "ASP.NET Core rate limiting middleware"
- ✅ "Azure OpenAI SDK retry policy"

**Poor queries** (too vague):
- ❌ "database"
- ❌ "authentication"
- ❌ "API"

**Tips**:
- Include product name (EF Core, ASP.NET Core, Azure OpenAI)
- Be specific about the feature/concept
- Use Microsoft's terminology (e.g., "middleware" not "filter")

### Code Sample Search

**Always specify language**:
```
✅ mcp_microsoft_doc_microsoft_code_sample_search(
     query: "DbContext configuration",
     language: "csharp"
   )

❌ mcp_microsoft_doc_microsoft_code_sample_search(
     query: "DbContext configuration"
   )  // Missing language - lower quality results
```

**Focus queries on implementation details**:
- ✅ "JWT token validation configuration"
- ✅ "OpenAI client initialization options"
- ❌ "what is JWT" (use docs search for concepts)

### Fetch Strategy

**When to fetch**:
- Search results mention "see full tutorial"
- Need prerequisites and setup steps
- Want complete API reference
- Require troubleshooting flowcharts
- Need to see all configuration options

**When NOT to fetch**:
- Search already provided sufficient information
- Just need a quick answer or code snippet
- Multiple pages returned - haven't identified the best one yet

### Knowledge Verification

**Always verify**:
- NuGet package versions (check documentation for latest stable)
- API signatures (methods may have changed)
- Configuration patterns (best practices evolve)
- Deprecated features (old approaches may be obsolete)

**Never assume**:
- That you know the current API without checking
- That package versions in your training data are current
- That patterns from memory are still recommended

## Common Use Cases

### Use Case 1: Database Integration

```
Task: Configure EF Core with PostgreSQL

Steps:
1. Search for EF Core PostgreSQL setup:
   mcp_microsoft_doc_microsoft_docs_search("Entity Framework Core PostgreSQL configuration")

2. Find code samples:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "DbContext PostgreSQL connection string",
     language: "csharp"
   )

3. Get complete setup guide:
   mcp_microsoft_doc_microsoft_docs_fetch(url: <postgresql-setup-guide>)

4. Implement using official patterns
```

### Use Case 2: Authentication

```
Task: Implement JWT authentication in ASP.NET Core

Steps:
1. Search for authentication concepts:
   mcp_microsoft_doc_microsoft_docs_search("ASP.NET Core JWT bearer authentication")

2. Get complete tutorial:
   mcp_microsoft_doc_microsoft_docs_fetch(url: <tutorial-from-search>)

3. Find code samples:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "AddJwtBearer configuration options",
     language: "csharp"
   )

4. Search for token generation:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "JwtSecurityToken creation",
     language: "csharp"
   )

5. Implement following official patterns
```

### Use Case 3: AI Service Integration

```
Task: Integrate Azure OpenAI for text summarization

Steps:
1. Search for SDK overview:
   mcp_microsoft_doc_microsoft_docs_search("Azure OpenAI SDK .NET getting started")

2. Fetch quickstart:
   mcp_microsoft_doc_microsoft_docs_fetch(url: <quickstart-url>)

3. Find initialization code:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "OpenAIClient authentication",
     language: "csharp"
   )

4. Find completion samples:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "chat completion request",
     language: "csharp"
   )

5. Implement with verified patterns
```

### Use Case 4: API Best Practices

```
Task: Implement rate limiting in ASP.NET Core API

Steps:
1. Search for rate limiting:
   mcp_microsoft_doc_microsoft_docs_search("ASP.NET Core rate limiting middleware")

2. Get complete guide:
   mcp_microsoft_doc_microsoft_docs_fetch(url: <rate-limiting-guide>)

3. Find configuration samples:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "rate limiter options configuration",
     language: "csharp"
   )

4. Search for policy patterns:
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "RequireRateLimiting policy",
     language: "csharp"
   )

5. Implement with official recommendations
```

## Integration with Other Skills

### With VS Code API Documentation

**Use microsoft_docs tools for**:
- .NET language features
- ASP.NET Core web development
- Azure services
- Entity Framework Core
- General C# patterns

**Use get_vscode_api for**:
- VS Code extension development
- Editor API interactions
- VS Code specific patterns
- Extension contribution points

## Troubleshooting

### Issue: No Results Found

**Possible causes**:
- Query too specific or uses wrong terminology
- Feature is very new or doesn't exist
- Looking for third-party library (not Microsoft)

**Solutions**:
1. Broaden the query: "ASP.NET Core authentication" instead of "ASP.NET Core OAuth2 PKCE flow"
2. Use Microsoft terminology: "middleware" not "interceptor"
3. Try related products: "Azure SDK" instead of specific service
4. If still no results, may need web search for third-party content

### Issue: Outdated Information

**Prevention**:
- Always check article dates when fetching full pages
- Look for "Latest version" or version selectors on fetched pages
- Verify package versions on NuGet.org
- Check for deprecation notices in documentation

**Solutions**:
1. Add version to query: "ASP.NET Core 8.0 authentication"
2. Search for migration guides: "migrate from X to Y"
3. Check .NET version compatibility

### Issue: Code Samples Don't Compile

**Possible causes**:
- Package version mismatch
- Missing using statements (samples often abbreviated)
- API signature changed since documentation written
- Incomplete sample (only showing relevant portions)

**Solutions**:
1. Verify package versions from documentation
2. Fetch complete article for full code context
3. Search for updated samples: "ASP.NET Core 8.0 <feature>"
4. Look for "Complete example" links in documentation

## Success Indicators

**You're using this skill effectively when**:
- ✅ You search documentation before every Microsoft/Azure implementation
- ✅ Your code uses current package versions and APIs
- ✅ You follow official patterns from code samples
- ✅ You verify assumptions with documentation searches
- ✅ You fetch complete guides for complex implementations
- ✅ You specify language parameter in code sample searches

**Warning signs**:
- ❌ Implementing from memory without verification
- ❌ Using package versions from training data
- ❌ Skipping documentation search to save time
- ❌ Not checking if APIs have changed
- ❌ Guessing at configuration options

## Quick Reference

### Decision Tree

```
Need Microsoft/Azure information?
│
├─ Conceptual understanding?
│  └─> microsoft_docs_search → (if needed) → microsoft_docs_fetch
│
├─ Code examples?
│  └─> microsoft_code_sample_search (with language!)
│
├─ Complete tutorial?
│  └─> microsoft_docs_search → microsoft_docs_fetch
│
└─ Third-party library?
   └─> Use web search or GitHub tools
```

### Quick Commands

```powershell
# Search for concept
mcp_microsoft_doc_microsoft_docs_search("your query here")

# Find code samples
mcp_microsoft_doc_microsoft_code_sample_search(
  query: "specific implementation",
  language: "csharp"
)

# Get complete page
mcp_microsoft_doc_microsoft_docs_fetch(
  url: "https://learn.microsoft.com/..."
)
```

## Conclusion

The Microsoft documentation MCP tools provide direct access to authoritative, up-to-date information. By following the workflows and best practices in this skill, you ensure:

1. **Accuracy**: Working with current APIs and patterns
2. **Reliability**: Following official best practices
3. **Efficiency**: Finding information quickly with targeted searches
4. **Quality**: Implementing features correctly the first time

**Remember**: The cornerstone of successful Microsoft/Azure development is using MCP tools to verify current documentation before every implementation. Microsoft technologies evolve rapidly - always search documentation before implementing, even for familiar features. Your cached knowledge may be outdated, but official documentation is always current.

**Never skip the mandatory pre-implementation checklist** listed in the "Critical Principle" section above.
