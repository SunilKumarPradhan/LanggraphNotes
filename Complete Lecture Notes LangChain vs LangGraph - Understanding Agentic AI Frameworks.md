---
title: "Complete Lecture Notes: LangChain vs LangGraph - Understanding Agentic AI Frameworks"
layout: default
nav_order: 4
parent: "Lecture Notes"
description: "Lecture notes: Complete Lecture Notes: LangChain vs LangGraph - Understanding Agentic AI Frameworks"
last_modified_date: 2026-01-09
source_transcript: "004_LangChain_Vs_LangGraph_Agentic_AI_using_LangGraph_Video_3_CampusX"
generated_by: "NoteyBoy"
---

# Complete Lecture Notes: LangChain vs LangGraph - Understanding Agentic AI Frameworks

## Course Context
This is the third video in an Agentic AI using LangGraph playlist, focusing on understanding why LangGraph exists, what it is, and how it differs from LangChain.

### Prerequisites
- Basic understanding of LangChain
- Familiarity with LangChain components
- Recommended: Watch first two videos of instructor's LangChain playlist

### Learning Objectives
1. Develop deep-level intuition on why LangGraph exists
2. Understand technical overview of LangGraph
3. Compare LangChain vs LangGraph capabilities
4. Learn when to use which framework

---

## Part 1: LangChain Foundation

### What is LangChain?
**Definition**: An open-source library designed to simplify the process of building LLM-based applications.

### Core Building Blocks

#### 1. Models
- Unified interface to interact with any LLM provider
- Supports: OpenAI, Anthropic (Claude), HuggingFace, Ollama
- Enables easy model switching without major code changes

#### 2. Prompts
- Helps engineer prompts of any type
- Facilitates prompt design process

#### 3. Retrievers
- Fetches relevant documents from vector stores/knowledge bases
- Used for RAG (Retrieval Augmented Generation) applications
- Multiple strategies and algorithms available

#### 4. Chains (Most Important)
- Connect different components together
- Output of first block automatically becomes input of second block
- Can create chains of any length

### What Can You Build with LangChain?

1. **Simple Conversational Workflows**
   - Chatbots
   - Text summarizers
   - User prompt → LLM → Response

2. **Multi-step Workflows**
   - Example: Topic → Detailed Report → Summary
   - Chain multiple LLMs and prompts together

3. **RAG-based Applications**
   - Chat with documents
   - Vector store → Retriever → Context + Prompt → LLM → Response

4. **Basic Agent Systems**
   - Tools concept (APIs, Python functions)
   - LLM decides when to call which tool
   - Example: Weather API integration

---

## Part 2: Understanding Workflows vs Agentic AI

### Key Distinction

**Workflow**: 
- Predefined code paths orchestrating LLMs and tools (static)
- Flow chart is pre-built by developer
- Executes in same order every time

**Agents**: 
- LLMs dynamically direct their own processes (dynamic)
- AI creates execution path in real-time

---

## Case Study: Automated Hiring Workflow

This example demonstrates a **workflow** (not true agentic AI) that reveals LangChain's limitations.

### Complete Workflow Steps

1. **Receive Hiring Request**
   - Input: Need backend engineer, remote, 2-4 years experience

2. **Create Job Description (JD)**
   - LLM generates detailed JD

3. **JD Approval** (Human-in-the-Loop)
   - Human supervisor reviews
   - If rejected → loop back to create JD
   - If approved → proceed

4. **Post JD**
   - Use tools: LinkedIn API, Naukri.com API
   - Post to job platforms

5. **Wait Period**
   - Wait 7 days for applications

6. **Monitor Applications**
   - Check number of applications
   - Threshold: 20 applications minimum

7. **Conditional Branch**
   - **If insufficient applications**:
     - Modify JD (reduce requirements, increase salary, etc.)
     - Wait 48 hours
     - Loop back to monitor
   - **If sufficient applications**:
     - Proceed to shortlisting

8. **Shortlist Candidates**
   - Resume parser tool
   - LLM scores each resume
   - Select top candidates (e.g., 5 people)

9. **Schedule Interviews**
   - Calendar API integration
   - Email API for notifications

10. **Conduct Interviews**
    - Provide question bank
    - Send reminders
    - Conduct interviews

11. **Selection Decision** (for each candidate)
    - **If rejected**: Send regret email
    - **If selected**: Send offer letter

12. **Offer Letter Process**
    - LLM creates offer letter
    - Email to candidate
    - Track acceptance

13. **Offer Response**
    - **If rejected**: Renegotiate → send new offer
    - **If accepted**: Start onboarding

14. **Onboarding**
    - HRMS integration
    - Welcome email
    - KT session planning
    - Laptop provisioning

---

## Part 3: Six Critical Challenges with LangChain

### Challenge #1: Control Flow Complexity

#### The Problem
LangChain is built for **linear workflows** (chains), but complex applications require **non-linear workflows**.

#### Three Sources of Non-linearity

1. **Conditional Branches**
   - Flow direction based on conditions
   - Example: If enough applications → proceed; else → modify JD

2. **Loops**
   - Repeat until condition met
   - Example: Keep creating JD until approved

3. **Jumps**
   - Control flow jumps to different parts
   - Example: After 48-hour wait, jump back to monitor applications

#### LangChain Implementation Issues

**Example Code** (partial workflow):
```python
# Hiring prompt
hiring_prompt = "We need to hire a software engineer for backend team"

# Create JD chain
jd_prompt = PromptTemplate("Create job description based on {hiring_request}")
llm = ChatOpenAI()
jd_chain = jd_prompt | llm | StrOutputParser()

# Approval and posting functions
def approve_jd(jd): # dummy function
def post_jd(jd): # dummy function

# Manual loop implementation (GLUE CODE)
approved = False
while not approved:
    jd = jd_chain.invoke({"hiring_request": hiring_prompt})
    approved = approve_jd(jd)
    
post_jd(jd)
```

**Problems**:
- Must write custom Python code for loops
- No native constructs for conditional branching
- No native constructs for jumps
- Results in **glue code** (code outside the library to stitch flow)
- More glue code = harder to maintain, debug, and collaborate

#### LangGraph Solution

**Graph-based Representation**:
- Each task = **Node** in a graph
- Control flow = **Edges** between nodes
- Graphs are inherently non-linear data structures

**Example Code**:
```python
# Create graph
graph = StateGraph()

# Add nodes (each is a Python function)
graph.add_node("hiring_request", hiring_request_func)
graph.add_node("create_jd", create_jd_func)
graph.add_node("check_approval", check_approval_func)
graph.add_node("post_jd", post_jd_func)

# Add edges
graph.add_edge("hiring_request", "create_jd")
graph.add_edge("create_jd", "check_approval")

# Conditional edge (handles loops and branching)
graph.add_conditional_edges(
    "check_approval",
    lambda state: "post_jd" if state["approved"] else "create_jd"
)

graph.add_edge("post_jd", END)
```

**Advantages**:
- **Zero glue code**
- Native support for loops, conditionals, jumps
- All logic implemented within LangGraph
- Better maintainability

---

### Challenge #2: State Handling

#### What is State?

**State** = Collection of important data points and their values that the workflow needs to track.

#### Important Data Points in Hiring Workflow

- Job Description (JD)
- JD approval status (approved/not approved)
- JD posted status
- Number of applications received
- Minimum applications threshold
- Shortlisted candidates and contact details
- Number of offers sent
- Offer status
- Onboarding status

#### State Characteristics

- Exists as **key-value pairs**
- **Evolves over time** as workflow progresses
- Example: JD value is `None` initially, gets set after creation, status changes after approval

#### LangChain Limitations

**LangChain is STATELESS**:
- Has "memory" feature but it's limited to conversation history
- No built-in mechanism for tracking workflow state
- Must manually manage state using external variables
- State management becomes part of glue code
- Difficult to track state evolution across complex workflows

#### LangGraph Solution

**LangGraph is STATEFUL**:
- Built-in state management system
- State automatically passed between nodes
- State persists throughout workflow execution
- Can checkpoint state at any point
- Enables recovery and resumption

---

### Challenge #3: Fault Tolerance

#### Problem with LangChain
- **No built-in fault tolerance**
- If a 5-step chain fails at step 3, you must restart from the beginning
- LangChain assumes chains are **short-lived** (quick execution)
- Not designed for long-running workflows

#### LangGraph Solution
Provides fault tolerance for both small and large failures:

**Small-Level Faults (Retry Logic)**
- Example: LinkedIn API is down while posting JD
- Can implement retry logic to catch errors and try again after delay
- Handles transient failures automatically

**System-Level Faults (Recovery)**
- Example: Server crashes, Docker container goes down
- **Recovery mechanism** allows resuming from exact point of failure
- Uses **checkpoint system**:
  - Creates checkpoint after each node execution
  - Saves state snapshot to memory/external database
  - Has a **persistence layer** for state storage
  - `resume()` function identifies previous state and restarts from failed node

**Key Point**: LangGraph designed for long-running workflows where faults are expected

---

### Challenge #4: Human-in-the-Loop

#### What is Human-in-the-Loop?
- Workflow pauses at specific stages for human decision-making
- Examples:
  - Approving generated JD
  - Confirming before posting to website
  - Manager approval (may take 24 hours)

#### Why It's Important
- **Accountability** should remain with humans for risky decisions
- Not all control should be with the agent
- Critical for production systems

#### LangChain Limitations
- **No default mechanism** for pausing chains
- Can request human input, but only for **short durations**
- Long waits (24 hours) cause problems:
  - Script keeps running, consuming resources
  - Risk of crashes during wait period
  
**Workaround**: Split chain into two parts
- First chain runs until approval needed
- Second chain starts after approval received
- **Problem**: Manual state transfer between chains, lots of glue code

#### LangGraph Solution
- **Human-in-the-loop is a first-class citizen**
- Dedicated documentation section for this feature
- Can pause execution **indefinitely** (minutes, hours, or days)
- **How it works**:
  - Checkpoints graph state after every step
  - Persists execution context
  - Resumes workflow from where it left off
  - Supports **asynchronous human review** without time constraints

**Analogy**: Like saving progress in a video game - save at Stage 3, resume at Stage 3 tomorrow

---

### Challenge #5: Nested Workflows / Subgraphs

#### What are Nested Workflows?
- A **subgraph** is a graph used as a node in another graph
- Allows building workflows within workflows
- Concept of **encapsulation** applied to LangGraph

#### Real-World Example: Automated Hiring Workflow
- "Conduct Interview" node is actually complex:
  - Generate questions per candidate
  - Multiple interview rounds
  - Evaluation after each round
- Can treat "Conduct Interview" as separate workflow embedded in main workflow

#### Two Major Use Cases

**1. Multi-Agent Systems**
- Multiple agents working together
- **Example: Self-Driving Car**
  - Agent 1: Processes sensor information
  - Agent 2: Handles driving capabilities
  - Agent 3: Manages entertainment system
  - Agent 4: CEO agent coordinating all others
- Essential for solving complex problems

**2. Reusability**
- Create reusable workflow components
- **Example**: Approval workflow
  - Needed for JD approval
  - Needed for posting approval
  - Needed for interview scheduling
  - Build once, reuse multiple times
- Like creating functions in programming

#### Key Challenge
- Both parent and child graphs have their own states
- Must understand state interaction between nested graphs

#### LangChain Limitation
- Cannot implement nested workflows
- Already struggling with simple workflows
- No support for this feature

---

### Challenge #6: Observability

#### What is Observability?
> "How easily you can monitor, debug, and understand what your workflow is doing at runtime"

#### Why It's Important
- **Production monitoring**: Track deployed agents/workflows closely
- **Debugging**: Identify what went wrong
- **Auditing**: Trace decisions made by agent
- **Example scenario**: Agent posts unlimited ads on LinkedIn, causing excessive costs
  - Need to audit: What steps led to this decision?

#### LangSmith Tool
- Library for monitoring LLM-based applications
- Integrates with LangChain
- **Records**:
  - LLM calls made
  - Prompts sent
  - Responses received
  - Token counts
  - Response times

#### LangChain + LangSmith Problem
- **Partial observability only**
- LangSmith tracks LangChain code (LLM calls)
- **Cannot track glue code**:
  - Custom loops
  - Which iteration of loop
  - Custom logic between LangChain components
- Result: Incomplete picture for complex applications

#### LangGraph + LangSmith Solution
- **Tight integration** between LangGraph and LangSmith
- **Complete observability** because:
  - No glue code needed
  - All code within LangGraph
  - Stateful execution tracks everything
  
**What's Tracked**:
- Node-to-node transitions
- State changes at each node
- State before and after node execution
- Messages exchanged between human and agent
- Human approval points and timing
- **Chronological timeline** of entire workflow execution

**Result**: Can backtrack entire workflow from start to finish

---

## Part 4: Summary and Decision Framework

### What is LangGraph?

**Definition**: 
An orchestration framework for building **stateful, multi-step, event-driven workflows** using LLMs, ideal for both single-agent and multi-agent AI applications.

**Think of it as**: A **flowchart engine for LLMs**

**You define**:
- Steps as nodes
- Connections as edges
- Logic governing transitions

**LangGraph handles**:
- State management
- Conditional branching
- Looping
- Pausing and resuming
- Fault recovery

---

### When to Use What?

#### Use LangChain for:
- Simple, linear workflows
- Prompt chains
- Summarizers
- Basic RAG systems
- Quick prototypes
- Short-lived operations

#### Use LangGraph for:
- Complex, non-linear workflows
- Conditional paths needed
- Loops required
- Human-in-the-loop steps
- Multi-agent coordination/collaboration
- Asynchronous, event-driven execution
- Long-running workflows
- Production-grade systems requiring fault tolerance

---

### Should We Stop Using LangChain?

**Answer: NO**

#### Why Both Are Needed

**LangGraph is built on top of LangChain**
- LangGraph doesn't replace LangChain
- LangGraph solves more complex problems **using** LangChain

**LangChain provides components**:
- ChatOpenAI
- Prompt templates
- Retrievers
- Document loaders
- Text splitters
- Tools

**LangGraph provides orchestration**:
- Chaining components
- Workflow management
- Complex execution patterns

**Relationship**: 
- **LangChain** = Components library (the building blocks)
- **LangGraph** = Orchestration framework (the blueprint)
- They work **hand-in-hand**

---

## Key Takeaways

1. **All challenges (#3-6) connect to stateful execution and checkpointing** - this is the fundamental difference between LangChain and LangGraph

2. **LangGraph excels at**:
   - Long-running workflows
   - Complex decision-making
   - Production-grade systems
   - Full observability
   - Fault tolerance

3. **LangChain remains essential** for providing the core components that LangGraph orchestrates

4. **Learning both is necessary** for building robust AI applications - they complement each other

5. **The "glue code" problem** is the root cause of most LangChain limitations in complex workflows

6. **Stateful vs Stateless** is the core architectural difference that enables LangGraph's advanced features

---

## Comparison Table

| Feature | LangChain | LangGraph |
|---------|-----------|-----------|
| **Control Flow** | Linear chains only | Non-linear graphs with conditionals, loops, jumps |
| **State Management** | Stateless (manual handling) | Stateful (built-in) |
| **Fault Tolerance** | None | Checkpointing and recovery |
| **Human-in-the-Loop** | Limited (short waits only) | First-class support (indefinite pauses) |
| **Nested Workflows** | Not supported | Subgraphs supported |
| **Observability** | Partial (LLM calls only) | Complete (entire workflow) |
| **Best For** | Simple, linear workflows | Complex, production workflows |
| **Glue Code** | Required for complex logic | Minimal to none |
| **Learning Curve** | Easier | Steeper (but worth it) |

---

## Next Steps

Understanding this motivation behind LangGraph is crucial for appreciating upcoming practical tutorials. The next videos will cover:
- Practical implementation of LangGraph concepts
- Building real-world workflows
- Multi-agent systems
- State management patterns
- Production deployment strategies

---

**Note**: This introduction provides the "why" behind LangGraph. The conceptual understanding developed here will make the practical implementation much more intuitive and meaningful.