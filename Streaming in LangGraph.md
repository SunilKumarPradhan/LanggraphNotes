---
title: "Streaming in LangGraph"
layout: default
nav_order: 13
parent: "Lecture Notes"
description: "Lecture notes: Streaming in LangGraph"
last_modified_date: 2026-01-09
source_transcript: "013_Streaming_in_LangGraph_CampusX"
generated_by: "NoteyBoy"
---

# Streaming in LangGraph

## Overview

This lecture covers the implementation of streaming functionality in LangGraph-based chatbot applications. Streaming enables real-time, token-by-token display of LLM responses instead of waiting for the complete response, significantly improving user experience. The session demonstrates both the theoretical foundation of streaming and its practical implementation in a chatbot with a Streamlit UI.

## Key Concepts

### What is Streaming?

**Streaming** in LLMs means the model starts sending tokens as soon as they are generated, instead of waiting for the entire response to be ready before returning it.

- **Traditional approach**: LLM generates complete response → entire response sent at once
- **Streaming approach**: LLM generates response → tokens sent progressively as they're created

### The Problem Streaming Solves

When requesting long outputs (essays, blogs, code) from a chatbot without streaming:
- Users experience significant wait times (5-10 seconds or more)
- The entire response appears abruptly on screen
- Poor readability and user experience
- Users may think the application has frozen

## Why Streaming is Important

### 1. **Faster Response Time (Perceived)**
- Users see output immediately rather than waiting for complete generation
- Reduces perceived latency
- Prevents users from thinking the app has frozen
- Critical for non-technical users who may abandon the application

### 2. **Mimics Human-Like Conversation**
- Creates a typewriter effect similar to ChatGPT
- Builds trust with users
- Feels alive and engaging
- Keeps users engaged throughout the response

### 3. **Essential for Multimodal UI**
- Particularly important for voice-based interfaces (like Alexa)
- Without streaming, voice assistants would have awkward silences
- Creates seamless conversation flow
- Prevents the "bad phone connection" feeling

### 4. **Better UX for Long Outputs**
- Especially beneficial for code generation
- Step-by-step display helps users understand the structure
- Easier to follow along with complex outputs
- More readable than sudden appearance of large text blocks

### 5. **Cost Optimization**
- Users can stop generation mid-response if unsatisfied
- Saves tokens by not generating complete unwanted responses
- Reduces API costs (LLM providers charge per token)

### 6. **Progress Updates for AI Agents**
- Can show step-by-step updates for complex tasks
- Example: "Opening BookMyShow → Selecting movie → Selecting seats → Processing payment"
- Reduces user uncertainty during long operations
- Builds confidence in the system

## Technical Implementation

### Python Generators

**Generator**: A special type of iterator in Python that generates values on-the-fly, one at a time, using the `yield` keyword instead of `return`.

```python
# Generator allows lazy evaluation
# Values produced one at a time as needed
```

### Backend Implementation (LangGraph)

**Key Change**: Replace `.invoke()` with `.stream()`

```python
# OLD METHOD (without streaming)
response = chatbot.invoke(initial_state)

# NEW METHOD (with streaming)
stream = chatbot.stream(
    {
        "messages": [HumanMessage(content="What is the recipe to make pasta?")]
    },
    config={
        "configurable": {"thread_id": "1"}
    },
    stream_mode="messages"
)
```

### Stream Modes in LangGraph

LangGraph provides multiple streaming modes:
- **messages**: For streaming LLM responses token-by-token
- **updates**: For intermediate updates
- **values**: For state values
- **custom**: For custom streaming logic

For chatbot LLM responses, use `stream_mode="messages"`

### Processing the Stream

```python
# The stream object returns a generator
# Each iteration provides: (message_chunk, metadata)

for message_chunk, metadata in stream:
    if message_chunk.content:
        print(message_chunk.content, end=" ")
```

### Frontend Implementation (Streamlit)

**Key Component**: `st.write_stream()`

Streamlit provides `st.write_stream()` function that:
- Accepts a generator object
- Automatically handles typewriter effect
- Displays content progressively

```python
# Implementation in Streamlit
with st.chat_message("assistant"):
    ai_message = st.write_stream(
        message_chunk.content 
        for message_chunk, metadata 
        in chatbot.stream(
            {"messages": [HumanMessage(content=user_input)]},
            config={"configurable": {"thread_id": "1"}},
            stream_mode="messages"
        )
    )
    
# Store the complete response in session state
st.session_state.messages.append({"role": "assistant", "content": ai_message})
```

## Implementation Steps

### Step 1: Backend Modification
1. Import necessary components (HumanMessage)
2. Replace `chatbot.invoke()` with `chatbot.stream()`
3. Specify three parameters:
   - Initial state (messages)
   - Config (thread_id)
   - Stream mode ("messages")

### Step 2: Frontend Integration
1. Use `st.chat_message("assistant")` context
2. Call `st.write_stream()` with generator
3. Pass the stream from `chatbot.stream()`
4. Extract `message_chunk.content` for each chunk
5. Store final response in session state

### Step 3: Testing
- Test with short queries
- Test with long outputs (500-word blogs, essays)
- Verify typewriter effect appears
- Confirm responses are stored in chat history

## Common Pitfalls

1. **Hardcoded inputs**: Remember to use `user_input` variable instead of hardcoded messages
2. **Missing imports**: Ensure `HumanMessage` is imported
3. **Wrong stream mode**: Use "messages" for LLM responses
4. **Not storing response**: Save the complete response to session state after streaming

## Practical Benefits Demonstrated

### Before Streaming:
- 5-10 second blank screen
- Sudden appearance of complete response
- Poor user experience
- Feels unresponsive

### After Streaming:
- Immediate feedback
- Progressive display
- Engaging user experience
- Professional appearance

## Summary

**Key Takeaways:**

1. **Streaming transforms user experience** by displaying LLM responses progressively rather than all at once
2. **Implementation is simple**: Change `.invoke()` to `.stream()` and use `st.write_stream()` in Streamlit
3. **Generators are central**: Understanding Python generators is crucial for working with streaming
4. **Multiple benefits**: Faster perceived response time, better readability, cost savings, and human-like interaction
5. **Essential for production**: All modern chatbot applications should implement streaming for professional UX
6. **Stream modes matter**: Use "messages" mode for LLM responses; other modes exist for agent updates
7. **Works across modalities**: Critical for both text and voice-based interfaces

Streaming is a small feature with massive impact on user experience, making it an essential component of any LLM-based application.