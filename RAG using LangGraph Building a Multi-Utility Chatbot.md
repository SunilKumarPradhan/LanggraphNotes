---
title: "RAG using LangGraph: Building a Multi-Utility Chatbot"
layout: default
nav_order: 20
parent: "Lecture Notes"
description: "Lecture notes: RAG using LangGraph: Building a Multi-Utility Chatbot"
last_modified_date: 2026-01-09
source_transcript: "020_RAG_using_LangGraph_Agentic_AI_using_LangGraph_CampusX"
generated_by: "NoteyBoy"
---

# RAG using LangGraph: Building a Multi-Utility Chatbot

## Overview

This lecture demonstrates how to integrate Retrieval Augmented Generation (RAG) into a chatbot using LangGraph, transforming a simple chatbot into a multi-utility application. The implementation treats RAG as a tool within the agentic AI framework, allowing users to upload documents and perform question-answering on their private data while maintaining access to other tools and normal chat functionality.

## Key Concepts

### What is RAG (Retrieval Augmented Generation)?

**RAG** is a technique that enhances Large Language Models (LLMs) by providing them with additional context from external documents, enabling them to answer questions about information not present in their training data.

**Core Principle**: In-context learning - when you provide an LLM with additional context alongside your query, it can generate responses based on both its parametric knowledge and the provided context.

## Why RAG is Needed

### 1. **Outdated Knowledge**
- Every LLM has a **knowledge cutoff date** - the date when training was completed
- Cannot answer questions about events or information after this date
- Example: If GPT-5 was trained until August 31, it cannot provide information about events after that date
- Modern chatbots solve this by performing web searches (which is essentially RAG)

### 2. **Privacy and Private Data**
- LLMs only know publicly available information from their training data
- Cannot answer questions about:
  - Personal documents
  - Company financial reports
  - Private expense sheets
  - Internal codebases
- **RAG enables querying private documents** without exposing them during training

### 3. **Hallucination Reduction**
- **Hallucination**: When LLMs generate false information with confidence
- Example: Providing non-existent research paper links
- RAG helps **ground responses** in actual documents, reducing fabricated answers

## How RAG Works: Architecture

### Step-by-Step Process

#### 1. **Document Processing**
```
Knowledge Source (e.g., 100-page book)
         ↓
    Split into smaller parts (pages/chunks)
         ↓
    Generate embeddings for each chunk
         ↓
    Store in Vector Database
```

#### 2. **Query Processing**
```
User Question
     ↓
Convert to embedding
     ↓
Search Vector Database for similar chunks
     ↓
Retrieve top-k most relevant chunks
     ↓
Combine with original question
     ↓
Send to LLM for response generation
```

### Key Components

**Embeddings**: Numerical representations (vectors) that capture the semantic meaning of text chunks

**Vector Store**: Specialized database (e.g., FAISS, Chroma) that stores embeddings and enables similarity search

**Retriever**: Component that:
- Converts user queries to embeddings
- Searches the vector store for similar content
- Returns the most relevant text chunks

**Context Window**: The limited amount of text an LLM can process at once (measured in tokens)

## Implementation in LangGraph

### Architecture Overview

The implementation treats **RAG as a tool** within the LangGraph framework, making it compatible with other tools in an agentic system.

### Step 1: Document Ingestion

```python
# Load the document
from langchain.document_loaders import PyPDFLoader
loader = PyPDFLoader("path/to/document.pdf")
documents = loader.load()

# Split into chunks
from langchain.text_splitter import RecursiveCharacterTextSplitter
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
chunks = text_splitter.split_documents(documents)

# Generate embeddings and create vector store
from langchain.embeddings import OpenAIEmbeddings
from langchain.vectorstores import FAISS

embeddings = OpenAIEmbeddings(model="text-embedding-3-small")
vectorstore = FAISS.from_documents(chunks, embeddings)

# Create retriever
retriever = vectorstore.as_retriever(
    search_type="similarity",
    search_kwargs={"k": 4}
)
```

### Step 2: Create RAG Tool

```python
@tool
def rag_tool(query: str):
    """Use this tool to answer questions based on uploaded documents."""
    
    # Retrieve relevant documents
    results = retriever.invoke(query)
    
    # Extract content and metadata
    context = [doc.page_content for doc in results]
    metadata = [doc.metadata for doc in results]
    
    return {
        "query": query,
        "context": context,
        "metadata": metadata
    }
```

### Step 3: Build LangGraph Workflow

**Graph Structure**:
```
START → Chat Node → Decision
                    ↓
            Need RAG? → Tool Node → Chat Node → END
                    ↓
            No RAG needed → END
```

**Node Definitions**:
- **Chat Node**: Processes queries and decides whether to use RAG tool
- **Tool Node**: Executes the RAG tool to retrieve relevant context
- **Edges**: Define the flow between nodes based on decisions

### Step 4: Integration Flow

1. User submits question
2. Chat node receives question
3. LLM decides if RAG is needed based on query content
4. If needed:
   - Tool node invokes RAG tool
   - Retriever searches vector store
   - Returns top-k relevant chunks
   - Chat node combines context with query
   - Generates final response
5. If not needed:
   - Chat node responds directly

## Multi-Utility Chatbot Features

The final implementation includes:

### 1. **Normal Chat Functionality**
- General conversation capabilities
- Uses LLM's parametric knowledge

### 2. **Tool Integration**
- Stock price lookup
- Calculator functions
- Web search capabilities
- MCP (Model Context Protocol) integration

### 3. **RAG Capabilities**
- PDF document upload
- Question-answering on uploaded documents
- Context-aware responses

### 4. **User Interface Features**
- File upload sidebar
- Past conversation history
- Streaming responses
- Session persistence

## Practical Example

**Scenario**: Uploading a machine learning textbook

**Query**: "Based on the uploaded document, what is Google Brain?"

**Process**:
1. Query converted to embedding
2. Vector store searched for similar content
3. Relevant pages about Google Brain retrieved
4. Context + query sent to LLM
5. Response generated: Information about Google Brain, TensorFlow development, and transformer models

## Important Considerations

### Context Window Limitations
- Cannot paste entire large documents into prompts
- Solution: Filter and retrieve only relevant chunks
- Typical approach: Retrieve top 4-5 most relevant chunks

### Embedding Models
- Convert text to numerical vectors
- Capture semantic meaning
- Example: OpenAI's `text-embedding-3-small`

### Vector Similarity Search
- Finds semantically similar content
- Not just keyword matching
- Understands context and meaning

## Code Organization

### Backend (`langgraph_rag_backend.py`)
- `ingest_pdf()`: Document loading and processing
- RAG tool definition
- LangGraph workflow setup
- Error handling

### Frontend (`streamlit_rag_frontend.py`)
- File upload interface
- Chat interface
- Conversation history
- Thread management

## Best Practices

1. **Chunk Size**: Balance between context and specificity (typically 500-1500 characters)
2. **Chunk Overlap**: Maintain context between chunks (typically 10-20% of chunk size)
3. **Retrieval Count**: Retrieve 3-5 most relevant chunks to stay within context limits
4. **Tool Description**: Write clear descriptions so LLM knows when to use RAG
5. **Error Handling**: Implement robust error handling for file uploads and processing

## Summary

### Key Takeaways

- **RAG solves three major problems**: outdated knowledge, private data access, and hallucination
- **Core mechanism**: Retrieve relevant context and combine with user queries
- **LangGraph implementation**: Treat RAG as a tool within the agentic framework
- **Architecture components**: Document loader → Splitter → Embeddings → Vector store → Retriever
- **Multi-utility approach**: Combine RAG with other tools for comprehensive chatbot functionality
- **Practical applications**: Document Q&A, knowledge base querying, private data analysis

### Next Steps

- Explore advanced RAG techniques
- Build more complex agentic AI applications
- Integrate additional tools and capabilities
- Optimize retrieval strategies for better performance

---

**Resources**: Complete code available in the video description. For detailed RAG concepts, refer to the dedicated RAG video in the LangChain playlist.