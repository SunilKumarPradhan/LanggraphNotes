---
title: "Sequential Workflows in LangGraph"
layout: default
nav_order: 6
parent: "Lecture Notes"
description: "Lecture notes: Sequential Workflows in LangGraph"
last_modified_date: 2026-01-09
source_transcript: "006_Sequential_Workflows_in_LangGraph_Agentic_AI_using_LangGraph_Video_5_CampusX"
generated_by: "NoteyBoy"
---

# Sequential Workflows in LangGraph

## Overview

This lecture introduces practical implementation of sequential workflows in LangGraph, marking the transition from theoretical concepts to hands-on coding. The session covers the fundamentals of creating linear workflows where tasks are connected in a sequential manner without branching or parallel paths. Through three progressively complex examples (BMI calculator, simple LLM Q&A, and prompt chaining), students learn the core syntax and structure of LangGraph applications.

## Key Concepts

### **Sequential Workflow**
A workflow where all tasks are connected in a linear fashion - one task leads to the next in a predetermined order without any branching or parallel execution paths.

### **State in LangGraph**
A **TypedDict** object that carries information throughout the workflow execution. The state:
- Is passed as input to every node
- Gets updated (partially or fully) by nodes during execution
- Is returned as output after each node completes
- Evolves throughout the workflow lifecycle

### **State Graph**
The primary class used to create graphs in LangGraph. It requires the state type to be specified during initialization.

## Installation and Setup

### Required Libraries

```python
# Install LangGraph
pip install langgraph

# Install LangChain (for LLM components)
pip install langchain

# Install LangChain OpenAI integration
pip install langchain-openai

# Install python-dotenv (for environment variables)
pip install python-dotenv
```

### Environment Setup

```python
from langgraph.graph import StateGraph
from langchain_openai import ChatOpenAI
from typing import TypedDict
from dotenv import load_dotenv

# Load environment variables
load_dotenv()
```

## Building Workflows: The Standard Process

Every LangGraph workflow follows these **four essential steps**:

1. **Define the State** - Create a TypedDict class representing workflow data
2. **Define the Graph** - Create nodes and their associated functions
3. **Add Edges** - Connect nodes to define execution flow
4. **Compile and Execute** - Compile the graph and invoke it with initial state

## Example 1: BMI Calculator (Non-LLM Workflow)

### Purpose
A simple workflow to understand LangGraph syntax without LLM complexity.

### State Definition

```python
from typing import TypedDict

class BMIState(TypedDict):
    weight: float  # in kg
    height: float  # in meters
    bmi: float
```

### Node Function

```python
def calculate_bmi(state: BMIState) -> BMIState:
    # Extract values from state
    weight = state['weight']
    height = state['height']
    
    # Calculate BMI
    bmi = weight / (height ** 2)
    
    # Update state with calculated BMI
    state['bmi'] = round(bmi, 2)
    
    # Return updated state
    return state
```

### Graph Construction

```python
from langgraph.graph import START, END

# Create graph object
graph = StateGraph(BMIState)

# Add node
graph.add_node("calculate_bmi", calculate_bmi)

# Add edges
graph.add_edge(START, "calculate_bmi")
graph.add_edge("calculate_bmi", END)

# Compile
workflow = graph.compile()
```

### Execution

```python
# Define initial state
initial_state = {
    "weight": 80,
    "height": 1.73
}

# Execute workflow
final_state = workflow.invoke(initial_state)

# Output: {'weight': 80, 'height': 1.73, 'bmi': 26.73}
```

### Enhanced Version: BMI with Category

**Extended State:**

```python
class BMIState(TypedDict):
    weight: float
    height: float
    bmi: float
    category: str  # New attribute
```

**Additional Node:**

```python
def label_bmi(state: BMIState) -> BMIState:
    bmi = state['bmi']
    
    if bmi < 18.5:
        state['category'] = "Underweight"
    elif 18.5 <= bmi < 25:
        state['category'] = "Normal"
    elif 25 <= bmi < 30:
        state['category'] = "Overweight"
    else:
        state['category'] = "Obese"
    
    return state
```

**Updated Graph:**

```python
graph.add_node("label_bmi", label_bmi)
graph.add_edge("calculate_bmi", "label_bmi")
graph.add_edge("label_bmi", END)  # Update final edge
```

## Example 2: Simple LLM Q&A Workflow

### Purpose
Demonstrates integration of LangChain LLM components with LangGraph.

### State Definition

```python
class LLMState(TypedDict):
    question: str
    answer: str
```

### Node Function with LLM

```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI()

def llm_qa(state: LLMState) -> LLMState:
    # Extract question from state
    question = state['question']
    
    # Form prompt
    prompt = f"Answer the following question: {question}"
    
    # Call LLM
    answer = model.invoke(prompt).content
    
    # Update state
    state['answer'] = answer
    
    return state
```

### Complete Workflow

```python
# Create graph
graph = StateGraph(LLMState)

# Add node
graph.add_node("llm_qa", llm_qa)

# Add edges
graph.add_edge(START, "llm_qa")
graph.add_edge("llm_qa", END)

# Compile and execute
workflow = graph.compile()

initial_state = {
    "question": "How far is the moon from Earth?"
}

final_state = workflow.invoke(initial_state)
print(final_state['answer'])
```

## Example 3: Prompt Chaining Workflow

### Concept
**Prompt chaining** involves making multiple sequential LLM calls where the output of one call feeds into the next. This decomposes complex tasks into manageable steps.

### Use Case: Blog Generation
1. Generate a detailed outline for a topic
2. Use the outline to generate the full blog content

### State Definition

```python
class BlogState(TypedDict):
    title: str
    outline: str
    content: str
```

### Node 1: Create Outline

```python
def create_outline(state: BlogState) -> BlogState:
    # Fetch title
    title = state['title']
    
    # Create prompt
    prompt = f"Generate a detailed outline for a blog on the topic: {title}"
    
    # Call LLM
    outline = model.invoke(prompt).content
    
    # Update state
    state['outline'] = outline
    
    return state
```

### Node 2: Create Blog

```python
def create_blog(state: BlogState) -> BlogState:
    # Fetch both title and outline
    title = state['title']
    outline = state['outline']
    
    # Create prompt using both
    prompt = f"Write a detailed blog on the title: {title}\nUsing the following outline:\n{outline}"
    
    # Call LLM
    content = model.invoke(prompt).content
    
    # Update state
    state['content'] = content
    
    return state
```

### Complete Workflow

```python
# Create graph
graph = StateGraph(BlogState)

# Add nodes
graph.add_node("create_outline", create_outline)
graph.add_node("create_blog", create_blog)

# Add edges (sequential flow)
graph.add_edge(START, "create_outline")
graph.add_edge("create_outline", "create_blog")
graph.add_edge("create_blog", END)

# Compile
workflow = graph.compile()

# Execute
initial_state = {
    "title": "Rise of AI in India"
}

final_state = workflow.invoke(initial_state)

# Access all intermediate and final results
print(final_state['outline'])   # Outline is preserved
print(final_state['content'])   # Final blog content
```

## Visualizing Graphs in Jupyter Notebook

LangGraph provides built-in visualization for workflows in Jupyter notebooks:

```python
from IPython.display import Image, display

display(Image(workflow.get_graph().draw_mermaid_png()))
```

This displays the graph structure showing:
- All nodes
- Connections (edges)
- Flow direction (START → nodes → END)

## Key Advantages of LangGraph Over LangChain

### State Preservation
In **LangChain chains**, intermediate outputs are lost - only the final output is accessible. In **LangGraph**, the state object preserves all information throughout the workflow:

```python
# In LangGraph, all intermediate results are accessible
final_state['title']    # Available
final_state['outline']  # Available
final_state['content']  # Available
```

### Flexibility
- Nodes can access any part of the state
- Easy to add new nodes without restructuring
- Better for complex, multi-step workflows

## Important Patterns and Best Practices

### 1. Node Function Signature
Every node function must:
- Accept state as input parameter
- Return state as output
- Use type hints for clarity

```python
def my_node(state: MyStateType) -> MyStateType:
    # Process state
    return state
```

### 2. Partial State Updates
Nodes typically update only specific attributes:

```python
def my_node(state: MyState) -> MyState:
    # Only update what's needed
    state['new_field'] = computed_value
    # Other fields remain unchanged
    return state
```

### 3. START and END Nodes
- **START**: Dummy node marking workflow entry point
- **END**: Dummy node marking workflow termination
- Both must be imported from `langgraph.graph`

### 4. Graph Compilation
Always compile before execution:

```python
workflow = graph.compile()  # Creates executable workflow
```

## Common Workflow Structure

```
START → Node1 → Node2 → Node3 → END
```

Each arrow represents an edge added via `graph.add_edge()`.

## Homework Assignment

Extend the blog generation workflow by adding an **evaluation node**:

### Requirements:
1. Add a new state attribute: `score: int`
2. Create a new node `evaluate_blog` that:
   - Takes the generated blog and outline
   - Asks the LLM to rate the blog (1-10)
   - Stores the score in state
3. Update the graph structure:
   ```
   START → create_outline → create_blog → evaluate_blog → END
   ```

### Hint:
```python
def evaluate_blog(state: BlogState) -> BlogState:
    outline = state['outline']
    content = state['content']
    
    prompt = f"Based on this outline:\n{outline}\n\nRate this blog:\n{content}\n\nProvide an integer score (1-10)."
    
    # Extract score from LLM response
    # Update state['score']
    
    return state
```

## Summary

### Key Takeaways

1. **Sequential workflows** are the simplest type of LangGraph workflows with linear execution paths
2. **State management** is central to LangGraph - it carries data through the entire workflow
3. **Four-step process**: Define State → Build Graph → Add Edges → Compile & Execute
4. **LangChain integration** allows using LLM components (ChatOpenAI, prompts, etc.) within LangGraph nodes
5. **Prompt chaining** enables complex multi-step LLM operations with preserved intermediate results
6. **Jupyter notebooks** provide excellent visualization capabilities for understanding workflow structure

### When to Use Sequential Workflows

- Simple, linear task decomposition
- Learning LangGraph fundamentals
- Prompt chaining scenarios
- When intermediate results need to be preserved

### Next Steps

Future videos will cover:
- Branching workflows (conditional logic)
- Parallel execution paths
- Complex agentic workflows
- Real-world project implementations

---

**Note**: All code examples and notebooks from this lecture are available in the GitHub repository linked in the video description.