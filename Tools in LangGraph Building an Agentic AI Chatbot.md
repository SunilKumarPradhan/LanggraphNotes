---
title: "Tools in LangGraph: Building an Agentic AI Chatbot"
layout: default
nav_order: 18
parent: "Lecture Notes"
description: "Lecture notes: Tools in LangGraph: Building an Agentic AI Chatbot"
last_modified_date: 2026-01-09
source_transcript: "018_Tools_in_LangGraph_Agentic_AI_using_LangGraph_CampusX"
generated_by: "NoteyBoy"
---

# Tools in LangGraph: Building an Agentic AI Chatbot

## Overview

This lecture covers the integration of tools into a LangGraph-based chatbot to enable it to perform actions beyond simple conversation. The tutorial demonstrates how to add three specific tools—a calculator, an internet search function, and a stock price retriever—transforming a basic chatbot into an agentic AI system capable of executing tasks. The implementation involves understanding tool nodes, tool conditions, and creating proper workflow loops between the LLM and tools.

## Key Concepts

### **What are Tools in LangGraph?**

**Tools** are external functions or capabilities that an LLM can invoke to perform specific actions or retrieve information it cannot generate on its own. They extend the chatbot's functionality beyond text generation to include:

- Performing calculations
- Searching the internet
- Retrieving real-time data (like stock prices)
- Any custom functionality you define

### **Tool Node**

A **Tool Node** is a prebuilt node in LangGraph that acts as a bridge between your graph and external tools. Key characteristics:

- It's a ready-made node provided by LangGraph (not custom-built)
- Handles a collection of LangChain tools
- Listens for tool calls from the LLM
- Automatically routes requests to the correct tool
- Acts as a "tool executor" for your system

```python
from langgraph.prebuilt import ToolNode

# Creating a tool node with your tools
tool_node = ToolNode(tools)
```

### **Tools Condition**

**Tools Condition** is a prebuilt conditional edge function that helps your graph decide the execution flow. It determines:

- Whether to proceed to the tool node (if action is needed)
- Whether to go to the end node (if normal chat response is sufficient)
- Whether to loop back to the LLM node (after tool execution)

This function analyzes the LLM's output to detect if a tool call was requested.

## Workflow Architecture

### Basic Chatbot Flow (Without Tools)

```
START → CHAT NODE (LLM) → END
```

### Enhanced Flow (With Tools)

```
START → CHAT NODE → [Tools Condition] → TOOL NODE
                ↓                           ↓
               END ←─────────────────────────┘
```

**Important**: The tool node should loop back to the chat node, not directly to the end node. This allows:
1. The LLM to refine tool outputs into user-friendly responses
2. Multi-step reasoning (using multiple tools sequentially)
3. Better error handling and response formatting

## Implementation Details

### 1. Types of Tools

**Prebuilt Tools**: Provided by LangChain for common use cases
```python
from langchain_community.tools import DuckDuckGoSearchRun

search_tool = DuckDuckGoSearchRun()
```

**Custom Tools**: Created for specific requirements using the `@tool` decorator
```python
from langchain_core.tools import tool

@tool
def calculator(first_number: float, second_number: float, operation: str):
    """Performs basic arithmetic operations.
    
    Args:
        first_number: The first number
        second_number: The second number
        operation: The operation (add, subtract, multiply, divide)
    """
    if operation == "add":
        return first_number + second_number
    # ... other operations
```

**Note**: Always include a detailed docstring—the LLM reads this to determine when to use the tool.

### 2. Three Tools Implemented

#### **Calculator Tool**
- Performs arithmetic operations (add, subtract, multiply, divide)
- Custom-built using `@tool` decorator
- Takes three inputs: two numbers and an operation

#### **DuckDuckGo Search Tool**
- Enables internet searching capabilities
- Prebuilt tool from LangChain community
- Allows chatbot to retrieve current information

#### **Stock Price Tool**
- Retrieves real-time stock prices
- Uses Alpha Vantage API
- Custom-built to fetch specific company stock data

```python
@tool
def get_stock_price(company_symbol: str):
    """Fetches the current stock price for a given company symbol."""
    # API call to Alpha Vantage
    # Returns JSON with stock information
```

### 3. Binding Tools to LLM

```python
# Create list of all tools
tools = [search_tool, calculator, get_stock_price]

# Bind tools to the LLM
llm_with_tools = llm.bind_tools(tools)
```

### 4. Building the Graph

```python
from langgraph.graph import StateGraph
from langgraph.prebuilt import ToolNode, tools_condition

# Define state
class State(TypedDict):
    messages: Annotated[list, add_messages]

# Create graph
graph = StateGraph(State)

# Add nodes
graph.add_node("chat", chat_node)
graph.add_node("tools", ToolNode(tools))

# Add edges
graph.add_edge(START, "chat")
graph.add_conditional_edges("chat", tools_condition)
graph.add_edge("tools", "chat")  # Loop back to chat

# Compile
chatbot = graph.compile(checkpointer=memory)
```

## Common Problems and Solutions

### Problem 1: Raw Tool Output Displayed to Users

**Issue**: Tool outputs (like JSON responses) are shown directly to users without formatting.

**Solution**: Create a loop from tool node back to chat node. This allows the LLM to:
- Receive the raw tool output
- Process and format it
- Generate a user-friendly response

### Problem 2: Multi-Step Reasoning Not Possible

**Issue**: Complex queries requiring multiple tools fail (e.g., "What is Apple's stock price and how much would 50 shares cost?")

**Solution**: The loop structure enables:
1. First tool call (get stock price)
2. Return to LLM with result
3. LLM decides to use calculator
4. Second tool call (multiply price × quantity)
5. Final formatted response

**Example Flow**:
```
User Query → Chat Node → Stock Tool → Chat Node → Calculator Tool → Chat Node → End
```

## Frontend Integration

### Streaming Only AI Messages

Filter out tool messages from the stream to avoid showing raw tool outputs:

```python
from langchain_core.messages import AIMessage

for event in chatbot.stream(input_data):
    for value in event.values():
        if isinstance(value["messages"][-1], AIMessage):
            # Stream only AI messages
            print(value["messages"][-1].content)
```

### Status Container (Optional Enhancement)

Display which tool is being used in real-time using Streamlit's status container:

```python
import streamlit as st

with st.status("Using DuckDuckGo Search..."):
    # Tool execution happens here
```

This provides better user experience by showing:
- When a tool is being invoked
- Which specific tool is running
- Progress indication during execution

## Practical Applications

### Use Cases for Tools

1. **Real-time Data Retrieval**: Stock prices, weather, news
2. **Calculations**: Complex mathematical operations
3. **External API Integration**: Payment processing, database queries
4. **File Operations**: Reading/writing files, data processing
5. **Custom Business Logic**: Domain-specific operations

### Best Practices

- **Clear Docstrings**: Essential for LLM to understand tool purpose
- **Error Handling**: Implement try-catch blocks in custom tools
- **API Key Management**: Use environment variables for sensitive data
- **Rate Limiting**: Be aware of API usage limits (e.g., Alpha Vantage)
- **Tool Selection**: Only include necessary tools to reduce decision complexity

## Summary

### Key Takeaways

1. **Tools extend LLM capabilities** beyond text generation to perform real-world actions
2. **Tool Node** acts as the executor for all tools in your system
3. **Tools Condition** determines whether to use tools or provide direct responses
4. **Loop structure** (Chat → Tools → Chat) is essential for:
   - Formatting tool outputs properly
   - Enabling multi-step reasoning
   - Better error handling
5. **Two types of tools**: Prebuilt (from LangChain) and Custom (using `@tool` decorator)
6. **Proper workflow design** prevents raw tool outputs from reaching users
7. **LangSmith integration** helps visualize and debug tool execution flows

### Implementation Checklist

- [ ] Import necessary modules (ToolNode, tools_condition)
- [ ] Create or import required tools
- [ ] Bind tools to LLM
- [ ] Create tool node with tool list
- [ ] Add conditional edges using tools_condition
- [ ] Add loop edge from tools back to chat
- [ ] Filter streaming to show only AI messages
- [ ] Test with various query types (chat vs. action)

### Homework Assignment

Experiment by adding your own custom tool to the chatbot based on your specific use case or interest area. Consider tools for:
- Weather information
- Database queries
- File system operations
- Custom calculations
- API integrations