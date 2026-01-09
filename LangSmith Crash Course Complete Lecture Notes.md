---
title: "LangSmith Crash Course: Complete Lecture Notes"
layout: default
nav_order: 16
parent: "Lecture Notes"
description: "Lecture notes: LangSmith Crash Course: Complete Lecture Notes"
last_modified_date: 2026-01-09
source_transcript: "016_LangSmith_Crash_Course_LangSmith_Tutorial_for_Beginners_Observability_in_GenAI_CampusX"
generated_by: "NoteyBoy"
---

# LangSmith Crash Course: Complete Lecture Notes
## A Comprehensive Guide to Observability and Evaluation for LLM Applications

**Instructor**: Nitesh from CampusX YouTube Channel

---

## Table of Contents
1. [Introduction and Prerequisites](#introduction)
2. [Why LangSmith? Real-World Problems](#why-langsmith)
3. [Understanding Observability](#observability)
4. [LangSmith Core Concepts](#core-concepts)
5. [Setup and Configuration](#setup)
6. [Tracing Applications](#tracing)
7. [LangGraph Integration](#langgraph)
8. [Advanced LangSmith Features](#advanced-features)
9. [LLM Ops Overview](#llm-ops)

---

## <a name="introduction"></a>1. Introduction and Prerequisites

### What is LangSmith?

**LangSmith** is a unified observability and evaluation platform where teams can debug, test, and monitor AI application performance. It addresses the unique challenges of building and maintaining production-ready LLM applications.

### Course Structure
- Theoretical background on observability needs
- Practical integration with LangChain and LangGraph
- Introduction to LLM Ops (emerging field)
- Advanced features: monitoring, evaluation, and collaboration

### Prerequisites
- Working knowledge of LangChain
- Understanding of LangGraph fundamentals
- Basic Python programming skills

---

## <a name="why-langsmith"></a>2. Why LangSmith? Real-World Problems

### The Core Challenge: Non-Deterministic Behavior

> **Key Insight**: Unlike traditional software (where 2×4 always equals 8), LLMs can produce different outputs for the same or similar inputs. This creates unique debugging challenges.

**Problems This Creates:**
- Issues don't leave proper error traces
- Systems become complex black boxes
- Lack of explainability
- Difficult to debug in production environments

---

### Scenario 1: Job Application Assistant (Latency Problem)

#### The Application
An LLM-based tool that helps students with job applications:

**Workflow:**
1. Takes Job Description (JD) link/PDF as input
2. Studies the JD requirements
3. Accesses student's Google Drive for portfolio/resume
4. Matches relevant skills with JD requirements
5. Generates customized cover letter
6. Proofreads and checks tonality

#### The Problem
- **Normal latency**: ~2 minutes
- **Sudden increase**: 7-10 minutes
- **Impact**: Users frustrated, leaving platform, revenue loss

#### The Debugging Challenge
**What you have:**
- Initial input
- Final output
- Total execution time

**What you DON'T have:**
- Time breakdown per component
- Which stage is causing the delay
- JD reading time vs. document fetching time vs. generation time

#### Possible Root Cause
A bug causing the system to scan the entire Google Drive instead of a specific folder, adding 8 minutes of unnecessary processing.

**Without observability, identifying this is nearly impossible.**

---

### Scenario 2: Research Assistant Agent (Cost Problem)

#### The Application
An autonomous agent that creates research reports:

**Features:**
- Takes research topic as input (e.g., "Solar Energy")
- Fetches academic papers from Google Scholar, arxiv.com
- Studies each paper and extracts key points
- Summarizes into comprehensive report
- Allows Q&A about the report

#### The Problem
- **Normal cost per report**: ₹0.50
- **Sudden spike**: Some reports cost ₹2.00
- **Inconsistency**: Others still cost ₹0.50
- **Impact**: Cost spike threatening profitability at scale

#### Understanding Agent Behavior
Agents are autonomous systems that:
- Receive a goal
- Apply reasoning to achieve it
- Perform actions in a loop
- Self-evaluate if goal is achieved

#### Possible Root Cause
A recent prompt update: *"Keep generating report until it's perfect"*

**The "Aamir Khan" Effect:**
For certain topics, the agent now repeats the entire process:
- Fetches papers again
- Re-studies content
- Re-extracts key points
- Re-generates summary
- Re-evaluates quality

#### The Debugging Challenge
- Error not consistent across all executions
- No error traces in code
- Multiple stages in workflow
- Cannot identify which stage is consuming extra tokens

---

### Scenario 3: RAG-based HR Chatbot (Hallucination Problem)

#### The Context
- Large organization (e.g., TCS) with thousands of employees
- Many company policies: leave, notice period, health insurance
- Freshers constantly ask HR the same questions
- HR productivity decreasing

#### The Solution: RAG-based Chatbot

**RAG Workflow:**
1. User asks question about company policy
2. Retriever fetches relevant documents from knowledge base
3. LLM receives question + context
4. Generates natural language response

#### The Problem
- Chatbot started hallucinating
- **Example**: User asks about leave policy
- **Response**: "No worries, take leave whenever you want, go to Goa!"
- **Impact**: Misinformation spreading throughout company

#### Why RAG Systems Hallucinate

**Two Main Failure Points:**

**1. Retriever Failure**
- Fetches irrelevant documents
- Example: Question about notice period → retrieves company history
- LLM has no relevant context to answer accurately

**2. Generator Failure**
- Low-quality local LLM used
- OpenAI model upgrade changed behavior unexpectedly
- Prompt not strict enough about using only provided context
- LLM "makes up" answers when context is insufficient

#### The Debugging Challenge
**What you CANNOT see:**
- Which documents the retriever actually fetched
- Exact final prompt sent to LLM
- Whether problem is retriever or generator
- System operates as a complete "black box"

---

## <a name="observability"></a>3. Understanding Observability

### Definition

> **Observability** is the ability to understand a system's internal state by examining its external outputs like logs, metrics, and traces. It allows you to diagnose issues, understand performance, and improve reliability by analyzing data generated by the system. Essentially, it's about being able to answer **"why something is happening"** within a system, even if you didn't anticipate the problem.

### Observability in LLM Context

**Core Capabilities:**
- Ability to see inside the system
- Understand component-by-component execution
- Trace entire workflow from input to output
- Identify bottlenecks and failures
- Analyze patterns across multiple executions

**Why It's Critical for LLMs:**
- Non-deterministic behavior requires deeper inspection
- Complex multi-step workflows need granular visibility
- Production issues often manifest as subtle performance degradation
- Traditional debugging tools insufficient for AI systems

---

## <a name="core-concepts"></a>4. LangSmith Core Concepts

### Three Fundamental Concepts

#### 1. **Project**
- The entire LLM application you build
- Top-level organizational unit
- **Examples**: 
  - Job application assistant
  - Research assistant
  - HR chatbot

#### 2. **Trace**
- One single execution of your project
- Each time a user runs the application = 1 trace
- **Example**: User asks "What is the capital of India?" → entire execution from input to output = 1 trace
- Contains all intermediate steps and component executions

#### 3. **Run**
- Execution through each component
- Each component's input→output = 1 run
- **Example in simple LLM app:**
  - Prompt template execution = Run 1
  - LLM execution = Run 2
  - Parser execution = Run 3
  - **Total**: 3 runs in 1 trace

### Hierarchy Visualization

```
Project (e.g., "HR Chatbot")
├── Trace 1 (User Query: "What is leave policy?")
│   ├── Run 1: Retriever
│   ├── Run 2: Prompt Template
│   ├── Run 3: LLM
│   └── Run 4: Output Parser
├── Trace 2 (User Query: "Notice period details?")
│   ├── Run 1: Retriever
│   ├── Run 2: Prompt Template
│   ├── Run 3: LLM
│   └── Run 4: Output Parser
└── Trace 3...
```

---

### What LangSmith Tracks

LangSmith provides comprehensive tracking across nine key dimensions:

#### 1. **Inputs and Outputs**
- Every user input to the system
- Every system output/response
- Captured at both trace and run levels

#### 2. **Intermediate Steps**
- All steps between input and final output
- What retriever received and returned
- What context was generated
- What prompt was sent to LLM
- What LLM generated
- What parser received and output

#### 3. **Latency**
- Time taken at application level (total trace time)
- Time taken at component level (each run)
- Helps identify bottlenecks

#### 4. **Token Usage**
- Input tokens consumed
- Output tokens generated
- Tracked per LLM call
- Aggregated at trace level

#### 5. **Cost**
- Calculated based on model pricing
- Token usage × model cost
- Per-run and per-trace costs
- Cumulative project costs

#### 6. **Errors**
- Any errors in components
- Stack traces
- Error messages
- Failed run identification

#### 7. **Tags**
- System-generated tags (e.g., model name: GPT-4o)
- Custom tags for categorization
- Searchable and filterable
- **Examples**: "llm_app", "report_generation", "summarization"

#### 8. **Metadata**
- System metadata (LangChain version, dependencies)
- Custom metadata (model parameters, configurations)
- **Examples**:
  - Model temperature settings
  - Parser type
  - Embedding model dimensions

#### 9. **User Feedback**
- Optional feedback attachment to traces
- Thumbs up/down ratings
- Custom feedback forms
- Tied to exact execution state

---

## <a name="setup"></a>5. Setup and Configuration

### Initial Setup

#### Step 1: Clone Repository
```bash
git clone [repository-url]
```

#### Step 2: Open in VS Code
```bash
cd langsmith-masterclass
code .
```

#### Step 3: Create Virtual Environment
```bash
python -m venv myenv
```

#### Step 4: Activate Virtual Environment
```bash
# Windows
myenv\Scripts\activate

# Mac/Linux
source myenv/bin/activate
```

#### Step 5: Install Dependencies
```bash
pip install -r requirements.txt
```

#### Step 6: Create LangSmith Account
1. Visit [LangSmith website](https://smith.langchain.com)
2. Sign up / Login
3. Navigate to Settings → API Keys
4. Create new API key:
   - Description: "Personal Project"
   - Copy the generated key

#### Step 7: Configure Environment Variables
Create `.env` file:
```bash
LANGCHAIN_TRACING_V2=true
LANGCHAIN_API_KEY=your_api_key_here
LANGCHAIN_PROJECT=your_project_name
OPENAI_API_KEY=your_openai_key
```

---

### Project Management

#### Setting Project Names

**Method 1: Via .env File**
```bash
LANGCHAIN_PROJECT="sequential_llm_app"
```

**Method 2: Via Code (Recommended)**
```python
import os
os.environ["LANGCHAIN_PROJECT"] = "sequential_llm_app"
```

> **Best Practice**: Code-level project names override .env settings, allowing dynamic project organization per application.

---

### Adding Tags and Metadata

#### Configuration Dictionary Method

```python
config = {
    "tags": ["llm_app", "report_generation", "summarization"],
    "metadata": {
        "model_1": "gpt-4o-mini",
        "model_1_temperature": 0.7,
        "parser": "string_output_parser"
    },
    "run_name": "sequential_chain"
}

# Apply to chain invocation
chain.invoke(input_data, config=config)
```

#### Benefits
- **Custom tags** for categorization and filtering
- **Metadata** for tracking model parameters and configurations
- **Custom run names** instead of auto-generated ones
- Makes traces searchable and organized
- Enables pattern analysis across similar executions

---

## <a name="tracing"></a>6. Tracing Applications

### Basic LLM Application Tracing

#### Simple Chain Example
```python
from langchain_openai import ChatOpenAI
from langchain_core.prompts import ChatPromptTemplate
from langchain_core.output_parsers import StrOutputParser

# Components
llm = ChatOpenAI(model="gpt-4o-mini")
prompt = ChatPromptTemplate.from_template("Tell me about {topic}")
parser = StrOutputParser()

# Chain
chain = prompt | llm | parser

# Execute (automatically traced)
response = chain.invoke({"topic": "artificial intelligence"})
```

#### What Gets Traced
```
Trace: "Simple LLM Chain"
├── Run 1: Prompt Template
│   ├── Input: {"topic": "artificial intelligence"}
│   └── Output: "Tell me about artificial intelligence"
├── Run 2: ChatOpenAI
│   ├── Input: "Tell me about artificial intelligence"
│   ├── Tokens: 150 input, 300 output
│   ├── Cost: $0.0023
│   └── Output: [Generated text about AI]
└── Run 3: String Output Parser
    ├── Input: [LLM response object]
    └── Output: [Parsed string]
```

---

### RAG Application Tracing

#### Why LangSmith is Critical for RAG

**Two Types of RAG Errors:**

**1. Retriever Errors**
- Retriever fails to fetch relevant document chunks
- Wrong context provided to LLM
- Results in irrelevant or hallucinated answers

**2. Generator Errors**
- LLM hallucinates despite receiving correct context
- Poor quality final answers
- Ignores provided context

**The Problem Without Tracing:**
- Only see final response
- No visibility into intermediate steps
- Impossible to identify which component failed

**LangSmith Solution:**
Traces every intermediate step:
- User question
- Retrieved documents (with relevance scores)
- Final prompt (question + context)
- LLM response
- Easy identification of failure point

---

#### Basic RAG Implementation

```python
from langchain_community.document_loaders import PyPDFLoader
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain_openai import OpenAIEmbeddings
from langchain_community.vectorstores import FAISS

# Step 1: Load PDF
def load_pdf(pdf_path):
    loader = PyPDFLoader(pdf_path)
    return loader.load()

# Step 2: Chunking
def split_documents(docs, chunk_size=1000, overlap=150):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=overlap
    )
    return splitter.split_documents(docs)

# Step 3: Embedding & Retriever
def build_vector_store(chunks):
    embeddings = OpenAIEmbeddings()
    vectorstore = FAISS.from_documents(chunks, embeddings)
    return vectorstore.as_retriever()
```

#### RAG Chain Structure

```python
from langchain_core.runnables import RunnableParallel, RunnablePassthrough

# Format retrieved documents
def format_docs(docs):
    return "\n\n".join(doc.page_content for doc in docs)

# RAG Chain
rag_chain = (
    RunnableParallel({
        "question": RunnablePassthrough(),
        "context": retriever | format_docs
    })
    | prompt
    | llm
    | parser
)
```

**Execution Flow:**
```
Parallel Chain:
├── Path 1: question → pass through → question
└── Path 2: question → retriever → format_docs → context
         ↓
{question, context} → prompt → LLM → parser
```

---

### Tracing Custom Functions with @traceable

#### The Problem
- LangSmith only traces LangChain runnables by default
- PDF loading, chunking, embedding steps not traced
- Results in partial application tracing
- Missing critical preprocessing steps

#### The Solution: @traceable Decorator

```python
from langsmith import traceable

@traceable(name="Load PDF")
def load_pdf(pdf_path):
    loader = PyPDFLoader(pdf_path)
    return loader.load()

@traceable(name="Split Documents")
def split_documents(docs, chunk_size, chunk_overlap):
    splitter = RecursiveCharacterTextSplitter(
        chunk_size=chunk_size,
        chunk_overlap=chunk_overlap
    )
    return splitter.split_documents(docs)

@traceable(name="Build Vector Store")
def build_vector_store(chunks):
    embeddings = OpenAIEmbeddings()
    vectorstore = FAISS.from_documents(chunks, embeddings)
    return vectorstore.as_retriever()

@traceable(name="Setup Pipeline")
def setup_pipeline(pdf_path):
    docs = load_pdf(pdf_path)
    chunks = split_documents(docs, 1000, 150)
    retriever = build_vector_store(chunks)
    return retriever
```

---

#### Adding Tags and Metadata to Custom Functions

```python
@traceable(
    name="Load PDF",
    tags=["pdf", "loader"],
    metadata={"loader": "PyPDFLoader"}
)
def load_pdf(pdf_path):
    loader = PyPDFLoader(pdf_path)
    return loader.load()

@traceable(
    name="Build Vector Store",
    tags=["embedding", "vector_store"],
    metadata={
        "embedding_model": "text-embedding-3-small",
        "dimensions": 1536
    }
)
def build_vector_store(chunks):
    embeddings = OpenAIEmbeddings()
    vectorstore = FAISS.from_documents(chunks, embeddings)
    return vectorstore.as_retriever()
```

**Benefits:**
- Component-specific tags and metadata
- Searchable traces
- Better organization
- Performance tracking per component
- Easy identification of bottlenecks

---

### Optimizing RAG Performance

#### Problem: Repeated Processing
Every query triggers the full pipeline:
- PDF loading
- Chunking
- Embedding generation
- **Result**: High latency (~20-30 seconds per query)

#### Solution: Index Persistence with FAISS

```python
import os
import pickle
from pathlib import Path

def should_rebuild_index(pdf_path, index_path):
    """Check if index needs rebuilding"""
    if not os.path.exists(index_path):
        return True
    
    # Check PDF modification time
    pdf_mtime = os.path.getmtime(pdf_path)
    index_mtime = os.path.getmtime(index_path)
    
    return pdf_mtime > index_mtime

def save_index(retriever, index_path):
    """Save FAISS index to disk"""
    retriever.vectorstore.save_local(index_path)

def load_index(index_path):
    """Load FAISS index from disk"""
    embeddings = OpenAIEmbeddings()
    vectorstore = FAISS.load_local(index_path, embeddings)
    return vectorstore.as_retriever()

@traceable(name="Setup Pipeline with Caching")
def setup_pipeline(pdf_path):
    index_path = "./indexes"
    
    if should_rebuild_index(pdf_path, index_path):
        # Build new index
        docs = load_pdf(pdf_path)
        chunks = split_documents(docs)
        retriever = build_vector_store(chunks)
        save_index(retriever, index_path)
    else:
        # Load existing index
        retriever = load_index(index_path)
    
    return retriever
```

#### Conditions for Rebuilding Index
1. First-time run (no existing index)
2. PDF content changes
3. PDF metadata changes (size, last modified time)
4. Chunking parameters change (chunk_size, chunk_overlap)
5. Different embedding model used

#### Performance Improvement
- **First run**: ~20-30 seconds
- **Subsequent runs**: ~1.5-4.5 seconds
- **Improvement**: ~85-90% latency reduction

---

### Agent Tracing (ReAct Pattern)

#### Agent Components Traced

**1. Scratchpad Initialization**
- Empty at start
- Accumulates thought-action-observation history
- Updated after each tool execution

**2. Prompt Template**
- System instructions
- Available tools description
- Format specification (Thought, Action, Observation)
- User question

**3. LLM Decision Making**
- Decides which tool to use
- Generates action input
- Reasons about next steps

**4. Tool Execution**
- DuckDuckGo search
- Weather API call
- Custom tool execution
- Results captured

**5. Observation Recording**
- Tool output added to scratchpad
- Updated prompt sent to LLM
- Iterative refinement

**6. Final Answer**
- LLM generates final response
- Based on accumulated observations

---

#### Example Trace Flow

**Query**: "What is the current temperature of Gurgaon?"

```
Step 1: Initialize Scratchpad
  └─ Content: (empty)

Step 2: Create Prompt Template
  ├─ Question: "What is the current temperature of Gurgaon?"
  └─ Available tools: [DuckDuckGo, GetWeatherData]

Step 3: LLM Response
  ├─ Thought: "I should use GetWeatherData function"
  ├─ Action: get_weather_data
  └─ Action Input: "Gurgaon"

Step 4: Add to Scratchpad
  └─ Thought + Action + Action Input recorded

Step 5: Execute Tool
  ├─ Input: "Gurgaon"
  └─ Output: {temperature: 30°C, humidity: 65%, wind_speed: 10km/h}

Step 6: Update Scratchpad with Observation
  └─ Observation: "Temperature is 30°C..."

Step 7: Modified Prompt with Observation
  └─ Full context sent back to LLM

Step 8: LLM Final Answer
  └─ "The current temperature of Gurgaon is 30°C"
```

---

#### Multi-Tool Agent Example

**Complex Query:**
```
"Identify the birthplace of Kalpana Chawla and give its current temperature"
```

**Trace Breakdown:**

```
Iteration 1:
├─ Thought: "I need to find Kalpana Chawla's birthplace"
├─ Action: duckduckgo_search
├─ Action Input: "Kalpana Chawla birthplace"
├─ Observation: "Karnal, Haryana, India"
└─ Scratchpad Updated

Iteration 2:
├─ Thought: "Now I need the temperature of Karnal"
├─ Action: get_weather_data
├─ Action Input: "Karnal, India"
├─ Observation: "Temperature: 28°C, Humidity: 70%"
└─ Scratchpad Updated

Final Answer:
└─ "Kalpana Chawla was born in Karnal, Haryana, India. 
    The current temperature there is 28°C."
```

**What LangSmith Shows:**
- Each iteration as separate run
- Tool selection reasoning
- Tool inputs and outputs
- Scratchpad evolution
- Total iterations before final answer
- Cost per iteration
- Time per tool execution

---

## <a name="langgraph"></a>7. LangGraph Integration

### Core Integration Principles

> **Two Fundamental Rules:**
> 1. **Trace Creation**: When you execute a LangGraph workflow, the entire execution becomes ONE trace in LangSmith
> 2. **Node as