# Microsoft Agent Framework - Quick Reference

This is a quick reference guide for the Microsoft Agent Framework skill. For comprehensive guidance, see [SKILL.md](SKILL.md).

## ⚠️ Critical Reminder

**Microsoft Agent Framework is BRAND NEW (2025) - Your LLM training data predates it entirely!**

**NEVER SKIP THIS STEP**: Search official documentation BEFORE implementing anything.

## Essential Research Commands

```powershell
# Always search first - framework is too new for cached knowledge
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <topic>")

# Find official code samples
mcp_microsoft_doc_microsoft_code_sample_search(
  query: "Microsoft Agent Framework <implementation>", 
  language: "csharp"
)

# Get complete tutorial
mcp_microsoft_doc_microsoft_docs_fetch(url: "<tutorial-url>")
```

## Package Installation

**Remember**: ALL packages require `--prerelease` flag!

```powershell
# Core framework
dotnet add package Microsoft.Agents.AI --prerelease

# Azure OpenAI integration
dotnet add package Azure.AI.OpenAI --prerelease
dotnet add package Azure.Identity
dotnet add package Microsoft.Agents.AI.OpenAI --prerelease

# Anthropic integration
dotnet add package Microsoft.Agents.AI.Anthropic --prerelease

# GitHub Copilot integration
dotnet add package GitHub.Copilot.SDK --prerelease

# A2A protocol
dotnet add package Microsoft.Agents.AI.A2A --prerelease

# AG-UI hosting (ASP.NET Core)
dotnet add package Microsoft.Agents.AI.Hosting.AGUI.AspNetCore --prerelease
dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
```

## Quick Decision Tree

```
Need AI agent capabilities?
│
├─ Single agent with tools?
│  └─> Search: "Microsoft Agent Framework create agent tools"
│
├─ Pipeline processing?
│  └─> Search: "Microsoft Agent Framework sequential orchestration"
│
├─ Parallel analysis?
│  └─> Search: "Microsoft Agent Framework concurrent orchestration"
│
├─ Collaborative refinement?
│  └─> Search: "Microsoft Agent Framework group chat orchestration"
│
├─ Dynamic expert routing?
│  └─> Search: "Microsoft Agent Framework handoff orchestration"
│
├─ Complex task planning?
│  └─> Search: "Microsoft Agent Framework magentic orchestration"
│
├─ MCP tool integration?
│  └─> Search: "Microsoft Agent Framework MCP tools"
│
└─ Human-in-the-loop?
   └─> Search: "Microsoft Agent Framework workflow checkpoint"
```

## Common Search Queries

### Agent Creation
- "Microsoft Agent Framework create agent Azure OpenAI"
- "Microsoft Agent Framework AIAgent IChatClient"
- "Microsoft Agent Framework agent instructions"

### Tools & MCP
- "Microsoft Agent Framework MCP tools"
- "Microsoft Agent Framework function calling"
- "Microsoft Agent Framework tool definition"

### Orchestration Patterns
- "Microsoft Agent Framework sequential orchestration"
- "Microsoft Agent Framework concurrent orchestration"
- "Microsoft Agent Framework group chat RoundRobin"
- "Microsoft Agent Framework handoff mesh"
- "Microsoft Agent Framework magentic manager planner"

### State & Sessions
- "Microsoft Agent Framework session state management"
- "Microsoft Agent Framework context provider"
- "Microsoft Agent Framework agent memory"

### Workflows
- "Microsoft Agent Framework workflow builder"
- "Microsoft Agent Framework checkpoint request response"
- "Microsoft Agent Framework workflow edges executors"

## Orchestration Pattern Comparison

| Pattern | Topology | Manager | Use Case |
|---------|----------|---------|----------|
| **Sequential** | Chain | None | Pipeline, multi-stage |
| **Concurrent** | Broadcast | None | Parallel analysis |
| **Group Chat** | Star | Yes (RoundRobin, Prompt-based) | Iterative refinement |
| **Handoff** | Mesh | None | Dynamic delegation |
| **Magentic** | Star | Yes (Planner-based) | Complex planning |

## Common Mistakes to Avoid

❌ **Using Semantic Kernel patterns**
```csharp
var kernel = new Kernel();  // WRONG - This is Semantic Kernel
```

❌ **Using AutoGen patterns**
```csharp
var agent = new ConversableAgent(...);  // WRONG - This is AutoGen
```

❌ **Forgetting --prerelease flag**
```powershell
dotnet add package Microsoft.Agents.AI  # WRONG - Missing flag
```

❌ **Assuming API signatures**
```csharp
// WRONG - Don't assume, search for current API
agent.Run("prompt");  // Might be RunAsync() now
```

✅ **Always search documentation first**
```powershell
# CORRECT - Verify before implementing
mcp_microsoft_doc_microsoft_code_sample_search(
  query: "AIAgent run method", 
  language: "csharp"
)
```

## Authentication Patterns

### Azure OpenAI
```powershell
# Environment variables
$env:AZURE_OPENAI_ENDPOINT="https://your-resource.openai.azure.com/"
$env:AZURE_OPENAI_DEPLOYMENT_NAME="gpt-4o-mini"
```

### Anthropic
```powershell
# Direct API
$env:ANTHROPIC_API_KEY="your-api-key"
$env:ANTHROPIC_DEPLOYMENT_NAME="claude-haiku-4-5"

# Azure Foundry
$env:ANTHROPIC_RESOURCE="your-foundry-resource"
$env:ANTHROPIC_DEPLOYMENT_NAME="claude-haiku-4-5"
```

## Pre-Implementation Checklist

Before writing ANY code:

- [ ] Read "Critical Principle: Fresh Information First" in SKILL.md
- [ ] Understand this is a PREVIEW framework
- [ ] Search Microsoft Learn for your use case
- [ ] Find official code samples
- [ ] Verify package names from documentation
- [ ] Prepare to use `--prerelease` flag
- [ ] Commit to NOT using Semantic Kernel/AutoGen patterns

## Mandatory Workflow

```
1. SEARCH → Find concept documentation
2. FIND → Get official code samples  
3. FETCH → Retrieve complete tutorial (if needed)
4. VERIFY → Check package names and API signatures
5. IMPLEMENT → Follow official patterns exactly
6. TEST → Verify behavior matches expectations
```

## Getting Help

**Official Resources**:
- Overview: https://learn.microsoft.com/en-us/agent-framework/overview/agent-framework-overview
- Quickstart: https://learn.microsoft.com/en-us/agent-framework/tutorials/quick-start
- User Guide: https://learn.microsoft.com/en-us/agent-framework/user-guide/overview
- GitHub: https://github.com/microsoft/agent-framework

**When Stuck**:
1. Search issue in documentation: `mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <your-issue>")`
2. Look for code samples: `mcp_microsoft_doc_microsoft_code_sample_search(query: "<your-scenario>", language: "csharp")`
3. Check GitHub issues/discussions
4. Review migration guides if coming from SK/AutoGen

## Success Checklist

You're using the skill effectively when:

- ✅ You search documentation before every implementation
- ✅ You use official code samples as templates
- ✅ You verify all package names and API signatures
- ✅ You use `--prerelease` flag on all packages
- ✅ Your code matches current official patterns
- ✅ You don't assume based on Semantic Kernel/AutoGen

## Red Flags

Stop and search documentation if:

- ❌ You're about to use Semantic Kernel or AutoGen code
- ❌ You're guessing at package names or namespaces
- ❌ You're assuming API signatures from memory
- ❌ You're skipping documentation to save time
- ❌ Code won't compile due to "type not found" errors

---

**Remember**: This framework is BRAND NEW. Your training data has ZERO information about it. ALWAYS search official documentation first. Never skip the research phase!

For comprehensive guidance, see [SKILL.md](SKILL.md).
