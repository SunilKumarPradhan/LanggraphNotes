---
title: "Building a Chatbot with UI in LangGraph & Streamlit"
layout: default
nav_order: 12
parent: "Lecture Notes"
description: "Lecture notes: Building a Chatbot with UI in LangGraph & Streamlit"
last_modified_date: 2026-01-09
source_transcript: "012_Building_a_Chatbot_with_UI_in_LangGraph_&_Streamlit_CampusX"
generated_by: "NoteyBoy"
---

# Building a Chatbot with UI in LangGraph & Streamlit

## Overview

This lecture demonstrates how to build a complete chatbot application by integrating LangGraph (backend) with Streamlit (frontend). The tutorial covers creating an interactive web interface for a chatbot that was previously only accessible through Jupyter notebooks, implementing conversation history management, and connecting the UI components with the LangGraph workflow to create a fully functional AI assistant.

## Key Concepts

### **Two-Component Architecture**

The chatbot application is divided into two main parts:

- **Backend (LangGraph)**: Contains the AI workflow, graph building, and message processing logic
- **Frontend (Streamlit)**: Provides the web interface where users interact with the chatbot

The user interacts with the frontend, which sends messages to the LangGraph backend, receives responses, and displays them in the UI.

### **Streamlit Chat Components**

Two essential Streamlit components power the chat interface:

1. **`st.chat_message`**: Displays individual messages with role-based icons (user or assistant)
2. **`st.chat_input`**: Provides the input box where users type their messages

## Building the Chat Interface

### Step 1: Creating Chat Messages

To display a user message:

```python
with st.chat_message("user"):
    st.text("Hi")
```

To display an assistant message:

```python
with st.chat_message("assistant"):
    st.text("How can I help you?")
```

The `role` parameter ("user" or "assistant") automatically assigns appropriate icons to distinguish between speakers.

### Step 2: Adding Chat Input

Create an input box for user messages:

```python
user_input = st.chat_input(placeholder="Type here")

if user_input:
    with st.chat_message("user"):
        st.text(user_input)
```

This captures user input and displays it when the user presses Enter.

## Managing Conversation History

### The Problem with Standard Variables

When using regular Python variables to store conversation history, Streamlit reruns the entire script on each interaction, resetting all variables and losing previous messages.

### Solution: Session State

**Session State** is a special Streamlit dictionary that persists data across reruns:

```python
# Initialize session state
if "message_history" not in st.session_state:
    st.session_state.message_history = []
```

### Storing Messages

Messages are stored as dictionaries with two keys:
- **`role`**: Identifies who sent the message ("user" or "assistant")
- **`content`**: The actual message text

```python
# Add user message to history
st.session_state.message_history.append({
    "role": "user",
    "content": user_input
})

# Add assistant message to history
st.session_state.message_history.append({
    "role": "assistant",
    "content": ai_message
})
```

### Displaying Conversation History

Load and display all previous messages at the top of the interface:

```python
for message in st.session_state.message_history:
    with st.chat_message(message["role"]):
        st.text(message["content"])
```

## Integrating LangGraph Backend

### File Structure

The project consists of two Python files:

1. **`langgraph_backend.py`**: Contains the LangGraph workflow, graph definition, and chatbot object
2. **`streamlit_frontend.py`**: Contains the Streamlit UI code

### Importing the Chatbot

Import the chatbot object from the backend:

```python
from langgraph_backend import chatbot
from langchain_core.messages import HumanMessage
```

### Invoking the Chatbot

When a user sends a message, invoke the LangGraph chatbot:

```python
# Define configuration with thread ID for checkpoint
CONFIG = {
    "configurable": {"thread_id": "thread1"}
}

# Invoke chatbot with user input
response = chatbot.invoke(
    {
        "messages": [HumanMessage(content=user_input)]
    },
    config=CONFIG
)

# Extract AI message from response
ai_message = response["messages"][-1].content
```

### Complete Integration Flow

1. User types a message in the chat input
2. Message is added to session state history
3. Message is displayed in the UI with user icon
4. Message is sent to LangGraph backend as `HumanMessage`
5. Backend processes the message and returns a response
6. AI response is extracted from the response object
7. AI message is added to session state history
8. AI message is displayed in the UI with assistant icon

## Complete Frontend Code Structure

```python
import streamlit as st
from langgraph_backend import chatbot
from langchain_core.messages import HumanMessage

# Configuration
CONFIG = {
    "configurable": {"thread_id": "thread1"}
}

# Initialize session state
if "message_history" not in st.session_state:
    st.session_state.message_history = []

# Display conversation history
for message in st.session_state.message_history:
    with st.chat_message(message["role"]):
        st.text(message["content"])

# Chat input
user_input = st.chat_input(placeholder="Type here")

if user_input:
    # Add and display user message
    st.session_state.message_history.append({
        "role": "user",
        "content": user_input
    })
    with st.chat_message("user"):
        st.text(user_input)
    
    # Get AI response
    response = chatbot.invoke(
        {"messages": [HumanMessage(content=user_input)]},
        config=CONFIG
    )
    ai_message = response["messages"][-1].content
    
    # Add and display AI message
    st.session_state.message_history.append({
        "role": "assistant",
        "content": ai_message
    })
    with st.chat_message("assistant"):
        st.text(ai_message)
```

## Running the Application

To run the Streamlit application:

```bash
streamlit run streamlit_frontend.py
```

The application will open in a web browser with a clean chat interface where users can interact with the AI assistant.

## Important Features

### Short-term Memory

The chatbot maintains conversation context through:
- **Session State**: Stores conversation history during the current session
- **Checkpoint Saver**: LangGraph's memory mechanism that remembers past interactions within a thread

### Clean UI Design

The interface provides:
- Clear visual distinction between user and assistant messages
- Automatic scrolling for long conversations
- Persistent chat history during the session
- Simple, intuitive input mechanism

## Summary

### Key Takeaways

1. **Separation of Concerns**: Backend (LangGraph) handles AI logic; frontend (Streamlit) manages user interaction
2. **Session State is Critical**: Use `st.session_state` to persist data across Streamlit reruns
3. **Message Format**: Store messages as dictionaries with `role` and `content` keys
4. **Integration Pattern**: Import backend objects, invoke with proper configuration, extract responses
5. **Thread Management**: Use thread IDs with checkpoint savers to maintain conversation context
6. **Minimal Code Changes**: Most backend code remains unchanged; frontend requires specific Streamlit components

### Prerequisites

- Basic understanding of Streamlit fundamentals
- Knowledge of LangGraph concepts and workflow building
- Familiarity with Python dictionaries and lists

### Best Practices

- Always initialize session state before using it
- Display conversation history before current messages
- Add messages to history before displaying them
- Use proper configuration when invoking chatbots with checkpoints
- Test the application incrementally as you build features