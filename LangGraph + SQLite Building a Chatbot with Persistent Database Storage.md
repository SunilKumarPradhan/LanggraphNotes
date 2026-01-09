---
title: "LangGraph + SQLite: Building a Chatbot with Persistent Database Storage"
layout: default
nav_order: 15
parent: "Lecture Notes"
description: "Lecture notes: LangGraph + SQLite: Building a Chatbot with Persistent Database Storage"
last_modified_date: 2026-01-09
source_transcript: "015_LangGraph_+_SQLite_Chatbot_with_Database_Integration_CampusX"
generated_by: "NoteyBoy"
---

# LangGraph + SQLite: Building a Chatbot with Persistent Database Storage

## Overview

This lecture demonstrates how to upgrade a chatbot built with LangGraph by replacing in-memory storage with SQLite database integration. This enhancement ensures that conversation history persists even after the application closes or refreshes, allowing users to resume conversations from exactly where they left off, even days later.

## Key Concepts

### **Checkpointer**
A LangGraph component that manages conversation state and history. There are three types:
- **InMemorySaver**: Stores data in RAM (temporary, lost on restart)
- **SqliteSaver**: Stores data in SQLite database (persistent, good for prototyping)
- **PostgreSaver**: Production-grade database storage (for deployed applications)

### **Thread ID**
A unique identifier for each conversation session. Multiple threads allow users to maintain separate, independent conversations with the chatbot.

### **Persistent Storage**
Data storage that survives application restarts, as opposed to temporary RAM-based storage that clears when the program ends.

## Implementation Steps

### 1. Backend Setup

#### Install Required Library
```bash
pip install langgraph-checkpoint-sqlite
```

#### Import SQLite Checkpointer
Replace the memory-based import:
```python
# Old import
from langgraph.checkpoint.memory import MemorySaver

# New import
from langgraph.checkpoint.sqlite import SqliteSaver
```

#### Create Database Connection
```python
import sqlite3

# Create SQLite database connection
conn = sqlite3.connect(
    "chatbot.db",
    check_same_thread=False
)
```

**Important Parameters:**
- `"chatbot.db"`: Database filename (created automatically if it doesn't exist)
- `check_same_thread=False`: Allows the database to be used across multiple threads (required for handling multiple conversations)

#### Initialize Checkpointer
```python
# Replace InMemorySaver with SqliteSaver
checkpointer = SqliteSaver(conn)
```

### 2. Testing the Database Integration

#### Basic Message Exchange
```python
# Send a message to the chatbot
response = chatbot.invoke(
    {"user_input": "Hi, my name is Nitish"},
    config={"configurable": {"thread_id": "thread_1"}}
)
print(response)
```

**Key Observations:**
- A `chatbot.db` file is automatically created in the project directory
- All messages (user and AI) are stored in this database
- Messages persist even after the program terminates

#### Verifying Persistence
```python
# In a new program run, ask about previous conversation
response = chatbot.invoke(
    {"user_input": "What is my name?"},
    config={"configurable": {"thread_id": "thread_1"}}
)
# Output: "Your name is Nitish"
```

### 3. Managing Multiple Threads

Different thread IDs maintain separate conversations:

```python
# Thread 1 conversation
chatbot.invoke(
    {"user_input": "Hi, my name is Nitish"},
    config={"configurable": {"thread_id": "thread_1"}}
)

# Thread 2 conversation (completely separate)
chatbot.invoke(
    {"user_input": "Hi, my name is Rahul"},
    config={"configurable": {"thread_id": "thread_2"}}
)
```

### 4. Visualizing the Database

#### Install SQLite Viewer Extension
In VS Code:
1. Go to Extensions
2. Search for "SQLite Viewer"
3. Install the extension by Florian Klampfer

#### View Database Contents
- Click on `chatbot.db` file in VS Code
- View all checkpoints organized by thread ID
- Each execution creates three checkpoints: start, chat node, and end

### 5. Retrieving Existing Threads

#### Function to Get All Threads
```python
def retrieve_all_threads():
    all_threads = set()
    
    # List all checkpoints in database
    for checkpoint in checkpointer.list(None):
        # Extract thread ID from each checkpoint
        thread_id = checkpoint.config['configurable']['thread_id']
        all_threads.add(thread_id)
    
    return list(all_threads)
```

**How it works:**
- `checkpointer.list(None)`: Returns all checkpoints (None means no specific thread filter)
- Extracts thread IDs from checkpoint configurations
- Uses a set to ensure only unique thread IDs
- Returns a list of all existing threads

### 6. Frontend Integration

#### Update Session Initialization

**Old Code (In-Memory):**
```python
if 'chat_threads' not in st.session_state:
    st.session_state.chat_threads = []  # Empty list
```

**New Code (Database-Backed):**
```python
if 'chat_threads' not in st.session_state:
    st.session_state.chat_threads = retrieve_all_threads()  # Load existing threads
```

#### Import Backend Functions
```python
from langgraph_database_backend import chatbot, retrieve_all_threads
```

## Practical Applications

### Use Case 1: Resuming Conversations
Users can close the application and return days later to continue exactly where they left off, with full conversation context preserved.

### Use Case 2: Multiple Conversation Topics
Users can maintain separate threads for different topics (e.g., one for recipes, another for travel planning) without conversations interfering with each other.

### Use Case 3: Conversation History Analysis
Developers can query the database to analyze conversation patterns, user preferences, and chatbot performance.

## Technical Details

### Checkpoint Structure
Each checkpoint contains:
- **Thread ID**: Identifies the conversation
- **Messages**: Complete conversation history
- **Metadata**: Timestamp, configuration details
- **State**: Current conversation state

### Database Schema
The SQLite database automatically creates tables to store:
- Checkpoint configurations
- Message history
- Thread associations
- Timestamps and metadata

### Thread Safety Considerations
The `check_same_thread=False` parameter is crucial because:
- SQLite by default restricts database access to the thread that created it
- Streamlit applications use multiple threads for handling user interactions
- Setting this to `False` removes the restriction, allowing multi-threaded access

## Summary

### Key Takeaways

1. **Persistent Storage**: Replacing `InMemorySaver` with `SqliteSaver` enables conversation persistence across application restarts

2. **Simple Integration**: Only two main code changes required:
   - Backend: Switch checkpointer from memory to SQLite
   - Frontend: Initialize chat threads from database instead of empty list

3. **Thread Management**: Each conversation maintains its own thread ID, allowing multiple independent conversations

4. **Automatic Database Creation**: The SQLite database file is created automatically when the connection is established

5. **Production Considerations**: For production applications, consider upgrading from SQLite to PostgreSQL for better scalability and concurrent access

6. **Data Visualization**: VS Code extensions make it easy to inspect and verify database contents during development

This implementation transforms a basic chatbot into a production-ready application with reliable conversation history management, significantly improving user experience and enabling advanced features like conversation analytics and long-term user engagement.