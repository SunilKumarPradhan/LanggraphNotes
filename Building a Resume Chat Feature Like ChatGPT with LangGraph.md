---
title: "Building a Resume Chat Feature Like ChatGPT with LangGraph"
layout: default
nav_order: 14
parent: "Lecture Notes"
description: "Lecture notes: Building a Resume Chat Feature Like ChatGPT with LangGraph"
last_modified_date: 2026-01-09
source_transcript: "014_How_to_build_a_Resume_Chat_feature_like_ChatGPT_CampusX"
generated_by: "NoteyBoy"
---

# Building a Resume Chat Feature Like ChatGPT with LangGraph

## Overview

This lecture demonstrates how to implement a conversation resume feature in a chatbot application, similar to ChatGPT's ability to manage multiple conversation threads. The tutorial builds upon a previously created chatbot by adding the capability to start new conversations, switch between existing threads, and maintain conversation history across different chat sessions. The implementation focuses on frontend changes using Streamlit while keeping the LangGraph backend intact.

## Key Concepts

### **Thread ID Management**
A unique identifier assigned to each conversation that allows the system to track and retrieve specific chat histories. Thread IDs are dynamically generated using Python's UUID library and stored in the session state.

### **Session State**
A Streamlit feature that maintains data persistence across user interactions within a single session. It stores critical information like current thread IDs, message history, and the list of all conversation threads.

### **Message History**
A list structure that stores all exchanged messages between the user and AI in a specific format, enabling the display of conversation flow in the UI.

### **Conversation Persistence**
The ability to save and retrieve past conversations, allowing users to resume previous chats even after starting new ones.

## Implementation Roadmap

The feature development was broken down into four main task sets:

### Task Set 1: Basic UI Setup

**Components Added:**
- Sidebar with title "LangGraph ChatBot"
- "New Chat" button for starting fresh conversations
- "My Conversations" section header

**Code Implementation:**
```python
st.sidebar.title("LangGraph ChatBot")
st.sidebar.button("New Chat")
st.sidebar.header("My Conversations")
```

### Task Set 2: Dynamic Thread ID Generation

**Objective:** Replace manual thread ID creation with programmatic generation

**Key Steps:**
1. **Create Utility Function:**
```python
def generate_thread_id():
    thread_id = uuid.uuid4()
    return thread_id
```

2. **Initialize Thread ID in Session:**
```python
if 'thread_id' not in st.session_state:
    st.session_state.thread_id = generate_thread_id()
```

3. **Update Configuration:**
Replace hardcoded thread IDs with `st.session_state.thread_id` in the chatbot invocation

**Benefits:**
- Automatic unique ID generation for each new conversation
- Eliminates manual thread management
- Scalable solution for unlimited conversations

### Task Set 3: New Chat Functionality

**Requirements:**
- Clear the UI when "New Chat" is clicked
- Generate new thread ID
- Reset message history
- Maintain previous thread IDs

**Implementation:**

1. **Reset Chat Function:**
```python
def reset_chat():
    # Generate new thread ID
    thread_id = generate_thread_id()
    
    # Store in session
    st.session_state.thread_id = thread_id
    
    # Clear message history
    st.session_state.messages = []
```

2. **Button Click Handler:**
```python
if st.sidebar.button("New Chat"):
    reset_chat()
```

**Result:** Users can start fresh conversations while preserving access to previous ones

### Task Set 4: Thread Storage and Display

**Challenge:** Prevent loss of previous thread IDs when creating new conversations

**Solution - Chat Threads List:**

1. **Create Persistent List:**
```python
if 'chat_threads' not in st.session_state:
    st.session_state.chat_threads = []
```

2. **Add Thread Utility Function:**
```python
def add_thread(thread_id):
    if thread_id not in st.session_state.chat_threads:
        st.session_state.chat_threads.append(thread_id)
```

3. **Call at Two Points:**
   - When page first loads
   - When new chat is created

4. **Display All Threads:**
```python
for thread_id in reversed(st.session_state.chat_threads):
    st.sidebar.button(str(thread_id))
```

**Note:** The `reversed()` function ensures newest conversations appear at the top

## Resume Conversation Feature

### Loading Past Conversations

**Objective:** Allow users to click on any thread and view its complete conversation history

**Step-by-Step Process:**

1. **Extract Messages from Thread:**
```python
def load_conversation(thread_id):
    response = chatbot.get_state(config={'configurable': {'thread_id': thread_id}})
    return response.values['messages']
```

2. **Handle Button Clicks:**
```python
if st.sidebar.button(str(thread_id)):
    # Update current thread ID
    st.session_state.thread_id = thread_id
    
    # Load messages
    messages = load_conversation(thread_id)
```

3. **Format Conversion:**
The messages from LangGraph come in a different format than what the UI expects. Conversion is needed:

**LangGraph Format:**
```python
[HumanMessage(content="..."), AIMessage(content="...")]
```

**UI Expected Format:**
```python
[
    {"role": "user", "content": "..."},
    {"role": "assistant", "content": "..."}
]
```

**Conversion Code:**
```python
temp_messages = []
for msg in messages:
    if isinstance(msg, HumanMessage):
        role = "user"
    else:
        role = "assistant"
    
    temp_messages.append({
        "role": role,
        "content": msg.content
    })

st.session_state.messages = temp_messages
```

## Important Details

### Session State Variables

- **`thread_id`**: Current active conversation's unique identifier
- **`messages`**: List of messages in current conversation
- **`chat_threads`**: List of all thread IDs created in the session

### UI/UX Considerations

1. **Instant Feedback:** Thread switching happens immediately
2. **Visual Hierarchy:** Most recent conversations appear at top
3. **Context Preservation:** Each thread maintains its own conversation context
4. **Clean Separation:** New chats don't interfere with existing ones

### Code Organization

**Utility Functions Section:**
- `generate_thread_id()`: Creates unique identifiers
- `reset_chat()`: Handles new conversation creation
- `add_thread()`: Manages thread list
- `load_conversation()`: Retrieves thread messages

**Sidebar UI Section:**
- Title and branding
- New chat button
- Thread list display

**Main Chat Area:**
- Message input
- Conversation display
- Streaming responses

## Current Limitations

### Memory Persistence Issue

**Problem:** When the application is refreshed, all conversation history is lost because the system uses `MemorySaver`, which stores data in RAM.

**Why This Happens:**
- In-memory storage is volatile
- Program termination clears RAM
- No persistent storage mechanism

**Planned Solution (Next Video):**
Connect the LangGraph backend to a database to ensure:
- Conversations survive application restarts
- Long-term conversation history
- Access to chats from days or weeks ago

## Practical Applications

1. **Customer Support Bots:** Users can return to previous support tickets
2. **Educational Assistants:** Students can resume learning sessions
3. **Personal Assistants:** Maintain context across multiple topics
4. **Research Tools:** Organize different research threads separately

## Summary

### Key Takeaways

1. **Thread-based Architecture:** Each conversation has a unique ID enabling independent management
2. **Session State Management:** Critical for maintaining state across user interactions in Streamlit
3. **Dynamic ID Generation:** UUID library provides reliable unique identifiers
4. **Format Compatibility:** Converting between different message formats is essential for UI integration
5. **Incremental Development:** Breaking complex features into smaller tasks makes implementation manageable

### What Was Achieved

✅ Sidebar UI with conversation management  
✅ Dynamic thread ID generation  
✅ New chat creation functionality  
✅ Thread list persistence  
✅ Conversation resume capability  
✅ Context preservation across threads  

### Next Steps

- **Database Integration:** Implement persistent storage for conversations
- **Tool Integration:** Add external tool capabilities
- **MCP Principles:** Implement Model Context Protocol
- **RAG Features:** Add retrieval-augmented generation
- **UI Improvements:** Better conversation naming and organization

### Homework Assignment

Implement a feature to display logical conversation names instead of thread IDs in the sidebar, similar to how ChatGPT automatically titles conversations based on their content.

---

**Note:** This implementation demonstrates professional chatbot development practices, emphasizing modularity, user experience, and scalable architecture. The step-by-step approach makes complex features accessible while maintaining code quality and organization.