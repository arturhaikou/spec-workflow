# Troubleshooting Guide

Common issues and solutions when working with Microsoft Agent Framework.

## Package Not Found

**Symptoms**: `dotnet add package` fails with package not found error

**Causes**:
- Incorrect package name
- Missing `--prerelease` flag
- Package name changed in recent update

**Solutions**:
1. **Always search first**:
   ```
   mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework package installation")
   ```

2. **Verify exact package name** from official documentation

3. **Always use --prerelease flag**:
   ```powershell
   dotnet add package Microsoft.Agents.AI --prerelease
   ```

4. **Check NuGet for current package names** at https://www.nuget.org/profiles/MicrosoftAgentFramework

## API Signature Mismatch

**Symptoms**: Compilation errors about methods not found or wrong parameters

**Causes**:
- Using outdated API patterns
- Assuming Semantic Kernel or AutoGen compatibility
- API changed in recent preview release

**Solutions**:
1. **Search for current API**:
   ```
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "Microsoft Agent Framework <class-name> <method-name>",
     language: "csharp"
   )
   ```

2. **Never assume based on old frameworks**

3. **Verify method signatures** from official code samples

4. **Check namespace imports** - may have changed

## Authentication Failures

**Symptoms**: Runtime errors related to credentials or authorization

**For Azure OpenAI**:
1. **Verify environment variables**:
   ```powershell
   $env:AZURE_OPENAI_ENDPOINT
   $env:AZURE_OPENAI_DEPLOYMENT_NAME
   ```

2. **Check Azure RBAC permissions**: User needs "Cognitive Services OpenAI User" or "Contributor" role

3. **Test authentication separately**:
   ```
   az login
   az account show
   ```

**For Anthropic**:
1. **Verify API key**:
   ```powershell
   $env:ANTHROPIC_API_KEY
   ```

2. **For Azure Foundry**:
   ```powershell
   $env:ANTHROPIC_RESOURCE
   $env:ANTHROPIC_DEPLOYMENT_NAME
   ```

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_code_sample_search(
  query: "Azure OpenAI authentication credential",
  language: "csharp"
)
```

## Workflow Not Executing

**Symptoms**: Workflow builds but doesn't execute, or execution hangs

**Causes**:
- Incorrect workflow configuration
- Message routing issues
- Missing edge definitions

**Solutions**:
1. **Search for orchestration-specific guidance**:
   ```
   mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <pattern> orchestration")
   ```

2. **Verify workflow builder syntax** from official samples

3. **Check edge definitions** connect all agents properly

4. **Validate message types** match expected inputs/outputs

## Agent Not Using Tools

**Symptoms**: Agent doesn't call tools even when appropriate

**Causes**:
- Tools not properly registered
- Incorrect tool definition format
- Agent configuration missing tool reference

**Solutions**:
1. **Search for current tool integration**:
   ```
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "Microsoft Agent Framework tool definition",
     language: "csharp"
   )
   ```

2. **Verify tool definition syntax**

3. **Check agent was created with tools**

4. **Test tool functions independently**

## Namespace or Type Not Found

**Symptoms**: Compilation errors about missing types or namespaces

**Causes**:
- Wrong package installed
- Missing package reference
- Namespace changed in preview release

**Solutions**:
1. **Verify all required packages installed**:
   ```powershell
   # Core
   dotnet add package Microsoft.Agents.AI --prerelease
   
   # Azure OpenAI
   dotnet add package Azure.AI.OpenAI --prerelease
   dotnet add package Microsoft.Agents.AI.OpenAI --prerelease
   ```

2. **Search for current namespaces**:
   ```
   mcp_microsoft_doc_microsoft_code_sample_search(
     query: "Microsoft Agent Framework <type-name>",
     language: "csharp"
   )
   ```

3. **Check using statements** match official samples

## Semantic Kernel or AutoGen Code Won't Work

**Symptoms**: Trying to use old framework patterns results in errors

**Cause**: Agent Framework is a new framework with different APIs

**Solution**:
1. **Do NOT try to adapt old code**

2. **Search for Agent Framework equivalent**:
   ```
   mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework <feature-name>")
   ```

3. **Follow official migration guides**:
   - From Semantic Kernel: https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-semantic-kernel/
   - From AutoGen: https://learn.microsoft.com/en-us/agent-framework/migration-guide/from-autogen/

4. **Start fresh with Agent Framework patterns**

## Preview Version Conflicts

**Symptoms**: Package version conflicts, incompatible dependencies

**Causes**:
- Mixing stable and preview packages
- Multiple preview versions

**Solutions**:
1. **Use --prerelease consistently** for all Agent Framework packages

2. **Check version alignment**:
   ```powershell
   dotnet list package --include-prerelease
   ```

3. **Search for current version recommendations**:
   ```
   mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework package versions compatibility")
   ```

## Resource Not Found in Documentation

**Symptoms**: Can't find information for specific feature

**Causes**:
- Feature is very new
- Using wrong search terms
- Feature might not exist yet (it's preview!)

**Solutions**:
1. **Try broader search terms**:
   ```
   # Instead of specific feature
   mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework overview")
   ```

2. **Check GitHub repository** for examples and issues

3. **Look for related concepts** that might include what you need

4. **Feature might be in development** - check roadmap
