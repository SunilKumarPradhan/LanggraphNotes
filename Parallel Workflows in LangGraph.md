---
title: "Parallel Workflows in LangGraph"
layout: default
nav_order: 7
parent: "Lecture Notes"
description: "Lecture notes: Parallel Workflows in LangGraph"
last_modified_date: 2026-01-09
source_transcript: "007_Parallel_Workflows_in_LangGraph_Agentic_AI_using_LangGraph_Video_6_CampusX"
generated_by: "NoteyBoy"
---

# Parallel Workflows in LangGraph

## Overview

This lecture covers the implementation of parallel workflows in LangGraph, building upon the sequential workflows learned in the previous session. The session demonstrates two practical examples: a non-LLM based cricket statistics calculator and an LLM-based UPSC essay evaluation system. Key concepts include partial state updates, structured outputs, and reducer functions for handling parallel node execution.

## Key Concepts

### Parallel Workflows vs Sequential Workflows

**Parallel Workflows** allow multiple nodes to execute simultaneously when their operations are independent of each other. Unlike sequential workflows where nodes execute one after another, parallel workflows can process multiple tasks concurrently and then merge their results.

**Key Difference**: In parallel workflows, you must use **partial state updates** instead of returning the entire state to avoid conflicts when multiple nodes update the state simultaneously.

### Partial State Updates

**Definition**: Returning only the specific attributes that were modified in a node, rather than the entire state object.

**Why it's necessary**: When multiple nodes execute in parallel and all return the complete state, LangGraph cannot determine which updates are valid, leading to `InvalidUpdateError`. Partial updates solve this by explicitly specifying which attributes each node modifies.

**Example**:
```python
# Instead of returning entire state
return state

# Return only modified attributes
return {
    "strike_rate": strike_rate
}
```

## Example 1: Cricket Batsman Statistics Workflow

### Workflow Structure

The workflow calculates three metrics in parallel from batsman data:
1. **Strike Rate**: (Runs / Balls) × 100
2. **Boundary Percentage**: ((Fours × 4 + Sixes × 6) / Total Runs) × 100
3. **Balls Per Boundary**: Total Balls / (Fours + Sixes)

### State Definition

```python
class BatsmanState(TypedDict):
    runs: int
    balls: int
    fours: int
    sixes: int
    strike_rate: float
    bpb: float  # Balls per boundary
    boundary_percent: float
    summary: str
```

### Node Implementation

**Calculate Strike Rate Node**:
```python
def calculate_strike_rate(state: BatsmanState):
    strike_rate = (state['runs'] / state['balls']) * 100
    return {
        "strike_rate": strike_rate
    }
```

**Calculate Balls Per Boundary Node**:
```python
def calculate_bpb(state: BatsmanState):
    bpb = state['balls'] / (state['fours'] + state['sixes'])
    return {
        "bpb": bpb
    }
```

**Calculate Boundary Percentage Node**:
```python
def calculate_boundary_percent(state: BatsmanState):
    boundary_percent = ((state['fours'] * 4 + state['sixes'] * 6) / state['runs']) * 100
    return {
        "boundary_percent": boundary_percent
    }
```

### Graph Construction

```python
graph = StateGraph(BatsmanState)

# Add nodes
graph.add_node("calculate_strike_rate", calculate_strike_rate)
graph.add_node("calculate_bpb", calculate_bpb)
graph.add_node("calculate_boundary_percent", calculate_boundary_percent)
graph.add_node("summary", summary)

# Add parallel edges from START
graph.add_edge(START, "calculate_strike_rate")
graph.add_edge(START, "calculate_bpb")
graph.add_edge(START, "calculate_boundary_percent")

# Converge to summary
graph.add_edge("calculate_strike_rate", "summary")
graph.add_edge("calculate_bpb", "summary")
graph.add_edge("calculate_boundary_percent", "summary")

# End workflow
graph.add_edge("summary", END)
```

## Example 2: UPSC Essay Evaluation Workflow

### Workflow Architecture

This advanced workflow evaluates essays on three aspects in parallel:
1. **Clarity of Thought**
2. **Depth of Analysis**
3. **Language Quality**

Each evaluation produces:
- Textual feedback
- Numerical score (0-10)

Results are then merged into a final evaluation with summarized feedback and average score.

### Structured Output with Pydantic

**Evaluation Schema**:
```python
from pydantic import BaseModel, Field

class EvaluationSchema(BaseModel):
    feedback: str = Field(description="Detailed feedback for the essay")
    score: int = Field(
        description="Score out of 10",
        ge=0,
        le=10
    )
```

**Creating Structured Model**:
```python
from langchain_openai import ChatOpenAI

model = ChatOpenAI(model="gpt-4o-mini")
structured_model = model.with_structured_output(EvaluationSchema)
```

### State Definition with Reducer Functions

```python
from typing import Annotated
from operator import add

class UPSCState(TypedDict):
    essay_text: str
    language_feedback: str
    analysis_feedback: str
    clarity_feedback: str
    overall_feedback: str
    individual_scores: Annotated[list[int], add]  # Reducer function
    average_score: float
```

### Understanding Reducer Functions

**Purpose**: Merge values from parallel nodes instead of overwriting them.

**The `add` operator**: When multiple nodes return lists for the same attribute, the `add` reducer concatenates them.

**Example**:
```python
# Node 1 returns: {"individual_scores": [8]}
# Node 2 returns: {"individual_scores": [7]}
# Node 3 returns: {"individual_scores": [6]}
# Result: {"individual_scores": [8, 7, 6]}
```

**Why use `operator.add`**: Direct use of `+` operator is not allowed in type annotations, so we use the functional equivalent from the `operator` module.

### Node Implementations

**Evaluate Language Node**:
```python
def evaluate_language(state: UPSCState):
    prompt = f"""Evaluate the language quality of the following essay and provide 
    a feedback and assign a score out of 10.
    
    Essay: {state['essay_text']}"""
    
    output = structured_model.invoke(prompt)
    
    return {
        "language_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

**Evaluate Analysis Node**:
```python
def evaluate_analysis(state: UPSCState):
    prompt = f"""Evaluate the depth of analysis of the following essay.
    
    Essay: {state['essay_text']}"""
    
    output = structured_model.invoke(prompt)
    
    return {
        "analysis_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

**Evaluate Thought Node**:
```python
def evaluate_thought(state: UPSCState):
    prompt = f"""Evaluate the clarity of thought of the following essay.
    
    Essay: {state['essay_text']}"""
    
    output = structured_model.invoke(prompt)
    
    return {
        "clarity_feedback": output.feedback,
        "individual_scores": [output.score]
    }
```

**Final Evaluation Node**:
```python
def final_evaluation(state: UPSCState):
    # Generate summarized feedback
    prompt = f"""Based on the following feedbacks, create a summarized feedback:
    
    Language Feedback: {state['language_feedback']}
    Depth of Analysis Feedback: {state['analysis_feedback']}
    Clarity of Thought Feedback: {state['clarity_feedback']}"""
    
    overall_feedback = model.invoke(prompt).content
    
    # Calculate average score
    average_score = sum(state['individual_scores']) / len(state['individual_scores'])
    
    return {
        "overall_feedback": overall_feedback,
        "average_score": average_score
    }
```

### Complete Graph Setup

```python
graph = StateGraph(UPSCState)

# Add nodes
graph.add_node("evaluate_language", evaluate_language)
graph.add_node("evaluate_analysis", evaluate_analysis)
graph.add_node("evaluate_thought", evaluate_thought)
graph.add_node("final_evaluation", final_evaluation)

# Parallel edges from START
graph.add_edge(START, "evaluate_language")
graph.add_edge(START, "evaluate_analysis")
graph.add_edge(START, "evaluate_thought")

# Converge to final evaluation
graph.add_edge("evaluate_language", "final_evaluation")
graph.add_edge("evaluate_analysis", "final_evaluation")
graph.add_edge("evaluate_thought", "final_evaluation")

# End workflow
graph.add_edge("final_evaluation", END)

# Compile
workflow = graph.compile()
```

### Execution Example

```python
initial_state = {
    "essay_text": "India's role in AI development..."
}

result = workflow.invoke(initial_state)

print(f"Language Feedback: {result['language_feedback']}")
print(f"Analysis Feedback: {result['analysis_feedback']}")
print(f"Clarity Feedback: {result['clarity_feedback']}")
print(f"Individual Scores: {result['individual_scores']}")
print(f"Average Score: {result['average_score']}")
print(f"Overall Feedback: {result['overall_feedback']}")
```

## Important Concepts Summary

### 1. Structured Outputs
- Ensures consistent, reliable output format from LLMs
- Uses Pydantic models to define schema
- Prevents errors from inconsistent LLM responses
- Essential for production workflows

### 2. Partial State Updates
- **Best Practice**: Always use partial updates in parallel workflows
- Works in both sequential and parallel workflows
- Prevents `InvalidUpdateError` in parallel execution
- Only return the attributes you're modifying

### 3. Reducer Functions
- Handle merging of values from parallel nodes
- Common reducers: `add` (concatenate), `max`, `min`
- Required when multiple nodes update the same attribute
- Imported from `operator` module for type annotations

## Key Takeaways

1. **Parallel workflows** enable concurrent execution of independent tasks, improving efficiency
2. **Partial state updates** are mandatory in parallel workflows to avoid conflicts
3. **Structured outputs** ensure reliable, parseable responses from LLMs
4. **Reducer functions** enable proper merging of parallel node outputs
5. LangGraph and LangChain work seamlessly together for complex workflows
6. The visual nature of nodes and edges makes workflow logic clear and maintainable

## Best Practices

- Always use partial state updates (return dictionaries with only modified keys)
- Define clear schemas for LLM outputs using Pydantic
- Use reducer functions when multiple nodes update the same state attribute
- Keep node functions focused on single responsibilities
- Test workflows with different inputs to ensure reliability
- Leverage structured outputs for any LLM-based nodes requiring consistent formatting