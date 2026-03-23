# Implementation Patterns

This reference provides detailed implementation patterns for common Microsoft Agent Framework scenarios.

## Pattern 1: Single Agent with Tools

### Research First
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework agent tools function calling")
mcp_microsoft_doc_microsoft_code_sample_search(query: "AIAgent function calling tools", language: "csharp")
```

### Implementation Steps
1. Define your tools as functions
2. Create IChatClient with tool definitions
3. Create agent from chat client
4. Tools are automatically invoked during agent execution

## Pattern 2: Sequential Multi-Agent Workflow

### Research First
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework sequential orchestration")
mcp_microsoft_doc_microsoft_code_sample_search(query: "sequential workflow AgentWorkflowBuilder", language: "csharp")
```

### Implementation Steps
1. Create specialized agents for each pipeline stage
2. Use AgentWorkflowBuilder to define sequence
3. Connect agents with edges
4. Execute workflow with input message

## Pattern 3: Group Chat Orchestration

### Research First
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework group chat orchestration")
mcp_microsoft_doc_microsoft_code_sample_search(query: "GroupChatManager RoundRobin", language: "csharp")
```

### Implementation Steps
1. Create specialized agents with different roles
2. Configure group chat manager (RoundRobin or custom)
3. Set maximum iteration count
4. Agents collaborate iteratively to refine output

## Pattern 4: Human-in-the-Loop Workflows

### Research First
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework human in the loop request response")
mcp_microsoft_doc_microsoft_code_sample_search(query: "workflow checkpoint approval", language: "csharp")
```

### Implementation Steps
1. Configure workflow with checkpoints
2. Use request/response message types
3. Pause execution at checkpoints for human approval
4. Resume workflow with human input

## Pattern 5: Agent-to-Agent Communication (A2A)

### Research First
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework A2A agent to agent protocol")
mcp_microsoft_doc_microsoft_code_sample_search(query: "A2AClient AIAgent", language: "csharp")
```

### Implementation Steps
1. Create A2A client with agent URL or resolver
2. Convert to AIAgent using AsAIAgent()
3. Use remote agent like any local agent
4. Supports well-known endpoint discovery
