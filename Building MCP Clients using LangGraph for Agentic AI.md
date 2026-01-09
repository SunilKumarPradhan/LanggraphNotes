---
title: "Building MCP Clients using LangGraph for Agentic AI"
layout: default
nav_order: 19
parent: "Lecture Notes"
description: "Lecture notes: Building MCP Clients using LangGraph for Agentic AI"
last_modified_date: 2026-01-09
source_transcript: "019_How_to_build_MCP_Client_using_LangGraph_Agentic_AI_using_LangGraph_CampusX"
generated_by: "NoteyBoy"
---

# Building MCP Clients using LangGraph for Agentic AI

## Overview

This lecture covers the implementation of Model Context Protocol (MCP) clients in LangGraph as an advanced alternative to traditional tools. The session demonstrates how MCP provides a standardized, maintainable approach to connecting AI chatbots with external services, solving critical maintenance problems inherent in the traditional tools-based approach. The tutorial includes practical implementation of both local and remote MCP servers with a LangGraph chatbot.

## Key Concepts

### What is MCP (Model Context Protocol)?

**MCP** is a standardized protocol for connecting tools to LLM applications that provides:
- **Improved version of tools**: A better way to integrate external functionality
- **Standardized connections**: Consistent interface between clients and servers
- **Separation of concerns**: Clear division between client-side and server-side code
- **Future-proof architecture**: Changes to server implementations don't require client code updates

### The Problem with Traditional Tools

Traditional tool integration suffers from a critical **brittleness problem**:

1. **Tight Coupling**: Tool implementation code exists directly in the chatbot codebase
2. **Maintenance Burden**: API changes require updates across all chatbots using the tool
3. **Scalability Issues**: With `n` tools and `m` chatbots, you face an `n × m` maintenance problem
4. **Breaking Changes**: External API updates can break your application without warning

**Example Scenario**:
```
GitHub API v1.0 → GitHub API v2.0
- URL changes: /repos/pulls → /repos/pull_requests
- Field changes: title → title_name
- Result: All chatbots using GitHub tools break immediately
```

### How MCP Solves These Problems

MCP introduces a **client-server architecture**:

```
Chatbot (Client) ←→ MCP Protocol ←→ External Service (Server)
```

**Key Benefits**:
- **Configuration-based**: Client only needs minimal config code
- **Server-side logic**: All heavy lifting happens on the server
- **Version independence**: Server updates don't affect client code
- **Reusability**: One MCP server can serve multiple clients

## Implementation Guide

### Step 1: Converting to Asynchronous Code

MCP clients require asynchronous execution. Convert synchronous LangGraph code to async:

**Before (Synchronous)**:
```python
def chat_node(state):
    response = llm_with_tools.invoke(state["messages"])
    return {"messages": [response]}
```

**After (Asynchronous)**:
```python
async def chat_node(state):
    response = await llm_with_tools.ainvoke(state["messages"])
    return {"messages": [response]}
```

**Key Changes**:
- Add `async` keyword to function definitions
- Use `await` with asynchronous operations
- Change `.invoke()` to `.ainvoke()`
- Wrap execution in `asyncio.run()`

### Step 2: Setting Up MCP Client

**Required Library**:
```python
from langchain_mcp_adapters.client import MultiServerMCPClient
```

**Installation**:
```bash
pip install langchain-mcp-adapters
# or with uv
uv add langchain-mcp-adapters
```

### Step 3: Configuring MCP Servers

#### Local MCP Server Configuration

```python
client = MultiServerMCPClient({
    "math_server": {
        "transport": "stdio",
        "command": "python",
        "args": ["/path/to/mcp_math_server/main.py"]
    }
})
```

**Components**:
- **Server name**: Identifier for the MCP server
- **Transport**: `stdio` for local servers
- **Command**: How to execute the server
- **Args**: Path to server file

#### Remote MCP Server Configuration

```python
client = MultiServerMCPClient({
    "expense_tracker": {
        "url": "https://your-server-url.com",
        "transport": "streamable_http"
    }
})
```

**Components**:
- **URL**: Remote server endpoint
- **Transport**: `streamable_http` for remote servers

### Step 4: Fetching and Using Tools

```python
async def build_graph():
    # Fetch tools from MCP server
    tools = await client.get_tools()
    
    # Bind tools to LLM
    llm_with_tools = llm.bind_tools(tools)
    
    # Continue with graph construction
    # ...
```

## Practical Examples

### Example 1: Math Calculator MCP Server

**Server Capabilities**:
- Addition
- Subtraction
- Multiplication
- Division
- Power
- Modulus

**Client Implementation**:
```python
# Minimal configuration code
client = MultiServerMCPClient({
    "calculator": {
        "transport": "stdio",
        "command": "python",
        "args": ["./calculator_server/main.py"]
    }
})

# Fetch all available tools
tools = await client.get_tools()
```

### Example 2: Expense Tracker MCP Server

**Server Capabilities**:
- Add expense
- List expenses
- Summarize expenses

**Usage Example**:
```
User: "Add an expense of ₹500 for a Udemy course on 10th November"
Bot: [Uses add_expense tool] "Added your expense successfully"

User: "List all my November expenses"
Bot: [Uses list_expenses tool] "Here are your expenses from 1st to 30th November..."
```

### Example 3: Combining Multiple MCP Servers

```python
client = MultiServerMCPClient({
    "math_server": {
        "transport": "stdio",
        "command": "python",
        "args": ["./math_server/main.py"]
    },
    "expense_tracker": {
        "url": "https://expense-api.example.com",
        "transport": "streamable_http"
    }
})
```

**Result**: Chatbot can perform both mathematical operations and expense tracking without writing tool-specific code.

## Mixing Tools and MCP

You can combine traditional tools with MCP clients:

```python
# Traditional tool
search_tool = DuckDuckGoSearchRun()

# MCP tools
mcp_tools = await client.get_tools()

# Combine both
all_tools = [search_tool] + mcp_tools
llm_with_tools = llm.bind_tools(all_tools)
```

**Recommendation**: Prefer MCP over traditional tools for better maintainability and future-proofing.

## Asynchronous Programming Basics

### Why Asynchronous?

**Synchronous Execution**:
```
Fetch weather → Wait → Fetch cricket score → Wait → Return results
```

**Asynchronous Execution**:
```
Fetch weather ──┐
                ├→ Return results (faster)
Fetch cricket ──┘
```

**Benefits**:
- Parallel execution of independent tasks
- Improved performance
- Better resource utilization
- Required for MCP client library

### Key Async Patterns

```python
# Async function definition
async def my_function():
    result = await some_async_operation()
    return result

# Running async code
import asyncio
asyncio.run(my_function())

# Async main pattern
async def main():
    # Your async code here
    pass

if __name__ == "__main__":
    asyncio.run(main())
```

## Integration with Streamlit

### Challenges

- **Streamlit**: Fundamentally synchronous
- **MCP**: Requires asynchronous execution
- **Solution**: Use async wrappers (hacky approach)

### Better Alternatives for Production

1. **FastAPI** for backend API
2. **React** or **Next.js** for frontend
3. Proper async/await support throughout

### Current Implementation (Development Only)

```python
# Backend: Async LangGraph with MCP
async def chat_node(state):
    response = await llm_with_tools.ainvoke(state["messages"])
    return {"messages": [response]}

# Frontend: Streamlit with async wrapper
async def stream_response():
    async for chunk in graph.astream(...):
        yield chunk
```

## Database Considerations

When using MCP with async code, database operations must also be async:

**Traditional SQLite**:
```python
import sqlite3
```

**Async SQLite**:
```python
import aiosqlite  # Async counterpart
```

## Complete Workflow

### 1. Project Structure
```
chatbot_project/
├── chatbot_mcp_backend.py    # Async LangGraph + MCP
├── streamlit_frontend_mcp.py  # UI (with async wrappers)
└── mcp_servers/
    ├── math_server/
    │   └── main.py
    └── expense_tracker/
        └── main.py
```

### 2. Backend Implementation

```python
import asyncio
from langchain_mcp_adapters.client import MultiServerMCPClient
from langgraph.graph import StateGraph

# Configure MCP clients
client = MultiServerMCPClient({...})

# Build graph
async def build_graph():
    tools = await client.get_tools()
    llm_with_tools = llm.bind_tools(tools)
    # ... graph construction
    return compiled_graph

# Main execution
async def main():
    graph = await build_graph()
    response = await graph.ainvoke(...)
    
if __name__ == "__main__":
    asyncio.run(main())
```

### 3. Key Differences from Traditional Tools

| Aspect | Traditional Tools | MCP Approach |
|--------|------------------|--------------|
| Code Location | Client-side | Server-side |
| Maintenance | Update all clients | Update server only |
| Scalability | n × m problem | Linear scaling |
| Standardization | Custom per tool | Protocol-based |
| Version Changes | Breaking changes | Transparent updates |

## Summary

### Key Takeaways

1. **MCP is superior to traditional tools** for production applications due to better maintainability and standardization

2. **Asynchronous programming is required** when working with MCP clients in LangGraph

3. **Separation of concerns** is the core principle: server handles logic, client handles configuration

4. **Multiple MCP servers** can be easily integrated into a single chatbot without code complexity

5. **Version independence** means server updates don't break client applications

6. **Mixed approach is possible** but prefer MCP for new implementations

### Best Practices

- Use MCP for external service integrations
- Keep client code minimal (configuration only)
- Prefer async implementations throughout
- Use FastAPI + React for production (not Streamlit)
- Leverage existing MCP servers from the community
- Document server capabilities clearly

### Next Steps

The next video will cover **RAG (Retrieval-Augmented Generation)** implementation, enabling the chatbot to answer questions based on internal documents.

### Resources

- MCP Playlist: Detailed coverage of MCP concepts, architecture, and server creation
- GitHub Repository: Complete code examples
- LangGraph Documentation: Official async patterns and best practices