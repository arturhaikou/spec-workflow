# Multi-Agent Orchestration Reference

Detailed guide to all orchestration patterns in Microsoft Agent Framework.

## Overview

| Pattern | Topology | Manager | Use Case |
|---------|----------|---------|----------|
| **Sequential** | Chain | None | Pipeline, multi-stage processing |
| **Concurrent** | Broadcast | None | Parallel analysis, independent tasks |
| **Group Chat** | Star | Yes (RoundRobin, Prompt-based) | Iterative refinement, collaboration |
| **Handoff** | Mesh | None | Dynamic delegation, expert routing |
| **Magentic** | Star | Yes (Planner-based) | Complex planning, generalist tasks |

## Sequential Orchestration

**When to use**: Tasks that must be processed in order, where each agent's output feeds the next.

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework sequential orchestration")
```

**Key concepts to verify**:
- AgentWorkflowBuilder syntax
- Edge definitions between agents
- Output message routing

## Concurrent Orchestration

**When to use**: Independent tasks that can run in parallel, aggregate results from multiple agents.

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework concurrent orchestration")
```

**Key concepts to verify**:
- Broadcast execution pattern
- Result aggregation methods
- Error handling for parallel tasks

## Group Chat Orchestration

**When to use**: Iterative refinement, collaborative problem-solving, content review.

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework group chat orchestration")
```

**Speaker selection strategies**:
1. **RoundRobin**: Agents speak in turn
2. **Prompt-based**: Manager selects based on context
3. **Custom**: Implement your own selection logic

**Key concepts to verify**:
- GroupChatManager configuration
- MaximumIterationCount setting
- Participant registration

## Handoff Orchestration

**When to use**: Dynamic delegation, escalation scenarios, expert routing.

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework handoff orchestration")
```

**Important notes**:
- Only supports ChatAgent
- Agents must support local tools execution
- Mesh topology (direct agent-to-agent handoff)
- No central manager

**Key concepts to verify**:
- Handoff rules configuration
- Context transfer between agents
- Termination conditions

## Magentic Orchestration

**When to use**: Complex, open-ended tasks requiring dynamic planning and collaboration.

**Search for current patterns**:
```
mcp_microsoft_doc_microsoft_docs_search("Microsoft Agent Framework magentic orchestration")
```

**Key features**:
- Planner-based manager coordinates agents
- Maintains shared context
- Tracks progress and adapts in real-time
- Sub-task delegation
- Iterative solution refinement

**Key concepts to verify**:
- MagenticManager configuration
- Specialized agent definitions
- Plan review and approval (human-in-the-loop)
- Progress tracking

## Common Searches by Use Case

### Customer Support System
```
"Microsoft Agent Framework handoff orchestration"
"Microsoft Agent Framework agent handoff rules"
```

### Document Processing Pipeline
```
"Microsoft Agent Framework sequential orchestration"
"Microsoft Agent Framework workflow edges"
```

### Collaborative Analysis
```
"Microsoft Agent Framework group chat RoundRobin"
"Microsoft Agent Framework iterative refinement"
```

### Research and Synthesis
```
"Microsoft Agent Framework magentic orchestration"
"Microsoft Agent Framework planner coordination"
```

### Parallel Data Processing
```
"Microsoft Agent Framework concurrent orchestration"
"Microsoft Agent Framework parallel execution"
```
