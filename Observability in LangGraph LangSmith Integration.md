---
title: "Observability in LangGraph: LangSmith Integration"
layout: default
nav_order: 17
parent: "Lecture Notes"
description: "Lecture notes: Observability in LangGraph: LangSmith Integration"
last_modified_date: 2026-01-09
source_transcript: "017_Observability_in_LangGraph_LangSmith_Integration_with_LangGraph"
generated_by: "NoteyBoy"
---

# Observability in LangGraph: LangSmith Integration

## Overview

This lecture covers the implementation of observability in LangGraph applications using LangSmith. The session demonstrates how to integrate LangSmith with an existing chatbot project to trace and monitor all interactions, token usage, latency, and system behavior. The integration enables developers to organize conversations into threads and gain comprehensive insights into their LLM application's performance.

## Key Concepts

### **Observability**
The ability to trace and monitor the end-to-end execution of a chatbot application. In the context of LLM systems, observability means recording:
- User messages and AI responses
- Token usage (input and output tokens)
- Latency and response times
- Internal system operations
- Execution flow and status

### **LangSmith**
A monitoring and tracing platform specifically designed for LangGraph and LangChain applications. It provides a centralized dashboard to visualize, debug, and analyze LLM application behavior.

### **Traces**
Individual records of each interaction turn between a user and the chatbot. Each trace captures:
- Input message
- Output response
- Execution time
- Token consumption
- System metadata

### **Threads**
Organizational units that group related traces together, representing complete conversations. Each thread contains multiple traces (turns) from a single conversation session.

## Project Context

### Current Chatbot Features
Before adding observability, the chatbot already includes:
- **GUI Interface**: User-friendly interface for interaction
- **Streaming**: Real-time response generation without waiting
- **Database Persistence**: Chat history preservation across sessions
- **Thread Management**: Separate conversation threads for different sessions

## LangSmith Setup and Configuration

### Step 1: Account Creation
1. Navigate to `smith.langchain.com`
2. Create a new account or log in to existing account
3. Access the LangSmith user interface

### Step 2: API Key Generation
1. Go to **Settings** → **API Keys**
2. Click **Create API Key**
3. Provide a description
4. Copy the generated API key securely

### Step 3: Environment Variables Configuration

Add the following variables to your `.env` file:

```env
LANGSMITH_TRACING=true
LANGSMITH_ENDPOINT=https://api.smith.langchain.com
LANGSMITH_API_KEY=your_api_key_here
LANGSMITH_PROJECT=chatbot-project
```

**Variable Explanations:**
- `LANGSMITH_TRACING`: Enables automatic tracing (set to `true`)
- `LANGSMITH_ENDPOINT`: LangSmith API endpoint URL
- `LANGSMITH_API_KEY`: Your unique authentication key
- `LANGSMITH_PROJECT`: Project name for organizing traces

### Step 4: Automatic Integration
Once environment variables are configured, LangSmith automatically begins tracing your LangGraph application **without requiring code changes**.

## Understanding the LangSmith Dashboard

### Projects Section
- **Top-level organization**: All traces are organized under projects
- **Project Name**: Defined by `LANGSMITH_PROJECT` environment variable
- **Multiple Projects**: Can manage different applications separately

### Traces View
Each trace displays:
- **Node Name**: The LangGraph node that executed (e.g., `chat_node`)
- **Model Used**: LLM model name (e.g., `ChatOpenAI`)
- **Input/Output**: Complete message exchange
- **Timestamps**: Start and end times
- **First Token Latency**: Time to receive first response token
- **Status**: Execution status (success/failure)
- **Token Counts**: Separate input and output token usage
- **Total Latency**: Complete response generation time

### Example Trace Information
```
Node: chat_node
Model: ChatOpenAI
Input: "Give me a roadmap to study AI engineering"
Output: [AI-generated response]
Execution Start: [timestamp]
Execution End: [timestamp]
First Token: [latency]
Status: Success
Total Tokens: 450
Input Tokens: 12
Output Tokens: 438
Latency: 2.3s
```

## Implementing Thread Organization

### The Problem
Without thread organization, all conversation turns from different sessions are stored in a single flat list, making it difficult to:
- Distinguish between separate conversations
- Track conversation flow
- Analyze specific user sessions

### The Solution: Thread IDs

To organize traces by conversation threads, add metadata to your configuration:

**Original Configuration:**
```python
config = {
    "configurable": {
        "thread_id": session["thread_id"]
    }
}
```

**Enhanced Configuration with Thread Metadata:**
```python
config = {
    "configurable": {
        "thread_id": session["thread_id"]
    },
    "metadata": {
        "thread_id": session["thread_id"]
    },
    "run_name": "chat_turn"
}
```

### Configuration Components

**`configurable` section:**
- Used by LangGraph for internal thread management
- Remains unchanged from previous implementation

**`metadata` section:**
- **Purpose**: Enables LangSmith to organize traces by thread
- **`thread_id`**: Links traces to specific conversation threads
- **Required**: Must be explicitly added for thread organization

**`run_name` (optional):**
- **Purpose**: Provides readable names for traces
- **Default**: "LangGraph" (not very descriptive)
- **Recommended**: "chat_turn" (clearly indicates each conversation turn)

## Practical Implementation Example

### Scenario 1: Single Thread Conversation
```
User: "Hi"
AI: "Hello! How can I assist you?"
User: "My name is Nitesh"
AI: "Nice to meet you, Nitesh!"
User: "Who created you?"
AI: "I was created by OpenAI"
```

**LangSmith Display:**
- **Thread 1**: Contains 3 traces (turns)
  - Turn 1: Hi → Hello
  - Turn 2: My name is Nitesh → Nice to meet you
  - Turn 3: Who created you → I was created by OpenAI

### Scenario 2: Multiple Thread Conversations

**Thread 1:**
```
User: "Hi"
AI: "Hello! How can I assist you?"
User: "Give me a roadmap to study AI engineering"
AI: [Detailed roadmap response]
```

**Thread 2 (New Conversation):**
```
User: "Hi, my name is Rahul"
AI: "Hello Rahul!"
User: "What is the roadmap to study AI?"
AI: [Roadmap response]
```

**LangSmith Organization:**
- **Thread 1**: 2 traces (AI engineering discussion)
- **Thread 2**: 2 traces (Rahul's conversation)
- Each thread is completely separate and independently viewable

## Benefits of Observability

### For Development
1. **Debugging**: Identify exactly where issues occur in conversation flow
2. **Performance Monitoring**: Track latency and optimize response times
3. **Token Management**: Monitor and control API costs
4. **Testing**: Verify chatbot behavior across different scenarios

### For Production
1. **User Analytics**: Understand how users interact with the chatbot
2. **Quality Assurance**: Monitor response quality and accuracy
3. **Cost Tracking**: Detailed token usage for billing analysis
4. **Compliance**: Maintain records of all interactions

### Future Applications
As the chatbot grows more complex with:
- **Tools Integration**: Track tool usage and effectiveness
- **RAG (Retrieval-Augmented Generation)**: Monitor document retrieval and context usage
- **MCP (Model Context Protocol)**: Analyze multi-model interactions

LangSmith's observability becomes increasingly valuable for understanding system behavior.

## Important Notes

### No Code Changes Required
Once environment variables are configured, LangSmith automatically traces your application without modifying the main codebase.

### Thread Organization Requires Metadata
While basic tracing works automatically, organizing traces into threads requires explicitly adding the `metadata` section to your configuration.

### Project Organization
All traces are organized under projects, allowing you to:
- Separate different applications
- Manage multiple environments (development, staging, production)
- Archive historical data

## Summary

### Key Takeaways

1. **Observability is Essential**: For LLM applications, tracking execution flow, token usage, and performance is critical for both development and production.

2. **LangSmith Integration is Simple**: Requires only environment variable configuration for basic tracing.

3. **Thread Organization Improves Clarity**: Adding metadata enables proper conversation grouping, making analysis much easier.

4. **Comprehensive Monitoring**: LangSmith captures all relevant metrics including latency, tokens, status, and complete message history.

5. **Scalable Solution**: As applications grow more complex, LangSmith's organizational features become increasingly valuable.

6. **Production-Ready**: The observability layer provides essential insights for maintaining and improving chatbot performance in real-world deployments.

### Next Steps

- Explore additional LangSmith features (monitoring, datasets, experiments, prompts, playground)
- Prepare for more complex features like tools integration and RAG
- Leverage observability data for continuous improvement of the chatbot

---

**Additional Resources**: For comprehensive LangSmith training, refer to the "LangSmith Crash Course" video mentioned in the lecture for detailed coverage of all platform features.