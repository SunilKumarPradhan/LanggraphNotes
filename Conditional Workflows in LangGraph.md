---
title: "Conditional Workflows in LangGraph"
layout: default
nav_order: 8
parent: "Lecture Notes"
description: "Lecture notes: Conditional Workflows in LangGraph"
last_modified_date: 2026-01-09
source_transcript: "008_Conditional_Workflows_in_LangGraph_Agentic_AI_using_LangGraph_Video_7_CampusX"
generated_by: "NoteyBoy"
---

# Conditional Workflows in LangGraph

## Overview

This lecture covers conditional workflows in LangGraph, the third type of workflow after sequential and parallel workflows. Conditional workflows allow branching based on conditions, similar to if-else statements in programming, where execution follows one path among multiple options based on specific conditions. The lecture demonstrates this concept through two practical examples: a non-LLM based quadratic equation solver and an LLM-based customer support review response system.

## Key Concepts

### **Conditional Workflows**
A workflow pattern where multiple branches exist, but execution follows only one branch based on a condition. Unlike parallel workflows where all branches execute simultaneously, conditional workflows evaluate a condition and route execution to a single appropriate branch.

**Key Characteristics:**
- Multiple possible execution paths
- Only one path executes at a time
- Decision based on conditional logic
- Similar to if-else statements in programming

### **Workflow Types Comparison**

1. **Sequential Workflows**: Tasks execute linearly, one after another
2. **Parallel Workflows**: Multiple tasks execute simultaneously across different branches
3. **Conditional Workflows**: One branch selected from multiple options based on conditions

## Building Conditional Workflows

### Core Components

#### **1. Conditional Routing Function**
A special function that:
- Receives the current state as input
- Evaluates conditions
- Returns the name of the next node to execute

```python
def check_condition(state):
    if condition:
        return "node_name_1"
    else:
        return "node_name_2"
```

#### **2. Add Conditional Edges**
Uses `add_conditional_edges()` instead of regular `add_edge()`:

```python
graph.add_conditional_edges(
    "source_node",
    routing_function
)
```

## Example 1: Quadratic Equation Solver (Non-LLM)

### Problem Overview
Solve quadratic equations of the form: **ax² + bx + c = 0**

### Mathematical Background

**Discriminant Formula**: `D = b² - 4ac`

**Three Possible Cases:**
1. **D > 0**: Two distinct real roots
   - Root₁ = (-b + √D) / 2a
   - Root₂ = (-b - √D) / 2a

2. **D = 0**: One repeated root
   - Root = -b / 2a

3. **D < 0**: No real roots

### Workflow Structure

```
Input (a, b, c) → Show Equation → Calculate Discriminant
                                        ↓
                        ┌───────────────┼───────────────┐
                        ↓               ↓               ↓
                  Real Roots    Repeated Root    No Real Roots
                        ↓               ↓               ↓
                        └───────────────┼───────────────┘
                                        ↓
                                      End
```

### Implementation Steps

#### **State Definition**
```python
class QuadState(TypedDict):
    a: float
    b: float
    c: float
    equation: str
    discriminant: float
    result: str
```

#### **Node Functions**

**1. Show Equation Node:**
```python
def show_equation(state):
    equation = f"{state['a']}x² + {state['b']}x + {state['c']}"
    return {"equation": equation}
```

**2. Calculate Discriminant Node:**
```python
def calculate_discriminant(state):
    discriminant = state['b']**2 - 4*state['a']*state['c']
    return {"discriminant": discriminant}
```

**3. Conditional Routing:**
```python
def check_condition(state):
    if state['discriminant'] > 0:
        return "real_roots"
    elif state['discriminant'] == 0:
        return "repeated_roots"
    else:
        return "no_real_roots"
```

#### **Building the Graph**
```python
graph = StateGraph(QuadState)
graph.add_node("show_equation", show_equation)
graph.add_node("calculate_discriminant", calculate_discriminant)
graph.add_node("real_roots", real_roots)
graph.add_node("repeated_roots", repeated_roots)
graph.add_node("no_real_roots", no_real_roots)

graph.add_edge(START, "show_equation")
graph.add_edge("show_equation", "calculate_discriminant")
graph.add_conditional_edges("calculate_discriminant", check_condition)
graph.add_edge("real_roots", END)
graph.add_edge("repeated_roots", END)
graph.add_edge("no_real_roots", END)

workflow = graph.compile()
```

## Example 2: Customer Support Review Response (LLM-Based)

### Workflow Overview

The system processes customer reviews and generates appropriate responses based on sentiment analysis.

### Workflow Architecture

```
Review Input → Sentiment Analysis
                      ↓
        ┌─────────────┴─────────────┐
        ↓                           ↓
  Positive?                    Negative?
        ↓                           ↓
Positive Response          Run Diagnosis
                          (Issue Type, Tone, Urgency)
                                    ↓
                          Negative Response
        ↓                           ↓
        └─────────────┬─────────────┘
                      ↓
                    End
```

### Implementation Details

#### **State Definition**
```python
class ReviewState(TypedDict):
    review: str
    sentiment: Literal["positive", "negative"]
    diagnosis: dict
    response: str
```

#### **Structured Output Schemas**

**1. Sentiment Schema:**
```python
class SentimentSchema(BaseModel):
    sentiment: Literal["positive", "negative"] = Field(
        description="Sentiment of the review"
    )
```

**2. Diagnosis Schema:**
```python
class DiagnosisSchema(BaseModel):
    issue_type: Literal["UI", "performance", "bug", "support", "other"] = Field(
        description="The category of issue mentioned in the review"
    )
    tone: Literal["frustrated", "angry", "disappointed", "neutral"] = Field(
        description="The emotional tone expressed by the user"
    )
    urgency: Literal["low", "medium", "high"] = Field(
        description="How urgent and critical the issue appears to be"
    )
```

#### **LLM Model Setup**

```python
# Base model
model = ChatOpenAI(model="gpt-4o-mini")

# Structured models
structured_model_1 = model.with_structured_output(SentimentSchema)
structured_model_2 = model.with_structured_output(DiagnosisSchema)
```

#### **Node Functions**

**1. Find Sentiment:**
```python
def find_sentiment(state: ReviewState):
    prompt = f"For the following review, find out the sentiment:\n{state['review']}"
    sentiment = structured_model_1.invoke(prompt).sentiment
    return {"sentiment": sentiment}
```

**2. Conditional Router:**
```python
def check_sentiment(state: ReviewState):
    if state['sentiment'] == "positive":
        return "positive_response"
    else:
        return "run_diagnosis"
```

**3. Positive Response:**
```python
def positive_response(state: ReviewState):
    prompt = f"""Write a warm thank you message in response to this review:
    {state['review']}
    Also kindly ask the user to leave feedback on our website."""
    
    response = model.invoke(prompt).content
    return {"response": response}
```

**4. Run Diagnosis:**
```python
def run_diagnosis(state: ReviewState):
    prompt = f"""Diagnose this negative review:
    {state['review']}
    Return issue_type, tone, and urgency."""
    
    response = structured_model_2.invoke(prompt)
    diagnosis = response.model_dump()
    return {"diagnosis": diagnosis}
```

**5. Negative Response:**
```python
def negative_response(state: ReviewState):
    diagnosis = state['diagnosis']
    prompt = f"""You are a support assistant. The user had a {diagnosis['issue_type']} issue,
    sounded {diagnosis['tone']}, and marked urgency as {diagnosis['urgency']}.
    Write an empathetic, helpful resolution message."""
    
    response = model.invoke(prompt).content
    return {"response": response}
```

#### **Graph Construction**

```python
graph = StateGraph(ReviewState)

# Add nodes
graph.add_node("find_sentiment", find_sentiment)
graph.add_node("positive_response", positive_response)
graph.add_node("run_diagnosis", run_diagnosis)
graph.add_node("negative_response", negative_response)

# Add edges
graph.add_edge(START, "find_sentiment")
graph.add_conditional_edges("find_sentiment", check_sentiment)
graph.add_edge("positive_response", END)
graph.add_edge("run_diagnosis", "negative_response")
graph.add_edge("negative_response", END)

workflow = graph.compile()
```

### Example Outputs

**Positive Review Example:**
- **Input**: "I have been using this app for about a month and I must say the user interface is incredibly clean and intuitive..."
- **Sentiment**: Positive
- **Response**: Warm thank you message with request for website feedback

**Negative Review Example:**
- **Input**: "I have been trying to log in for over an hour now and the app keeps freezing..."
- **Sentiment**: Negative
- **Diagnosis**:
  - Issue Type: Bug
  - Tone: Frustrated
  - Urgency: High
- **Response**: Empathetic resolution message addressing the specific issue

## Important Concepts

### **Structured Output with Pydantic**

Using `BaseModel` from Pydantic to define schemas ensures:
- Type safety
- Validation
- Consistent JSON structure
- Easy extraction of specific fields

### **Model Dump Method**

Converting Pydantic objects to dictionaries:
```python
diagnosis = response.model_dump()
```

### **Conditional Edges vs Regular Edges**

| Feature | Regular Edges | Conditional Edges |
|---------|--------------|-------------------|
| Syntax | `add_edge()` | `add_conditional_edges()` |
| Routing | Fixed path | Dynamic based on condition |
| Visualization | Solid lines | Dotted lines |
| Use case | Sequential flow | Branching logic |

## Best Practices

1. **Clear Routing Logic**: Keep conditional functions simple and readable
2. **Structured Outputs**: Use Pydantic schemas for consistent LLM responses
3. **State Management**: Define comprehensive state with all necessary fields
4. **Error Handling**: Consider edge cases in conditional logic
5. **Testing**: Test all possible branches with different inputs

## Key Takeaways

- **Conditional workflows are essential** for building complex agentic systems, similar to if-else in programming
- **Routing functions** determine which branch to execute based on state conditions
- **`add_conditional_edges()`** is the key method for implementing conditional logic
- **Structured outputs** using Pydantic ensure reliable LLM responses
- **Multiple structured models** can be used in the same workflow for different purposes
- **Conditional workflows enable intelligent decision-making** in AI agents
- This pattern will be used extensively in more complex workflows going forward

## Additional Notes

- There's an alternative method for creating conditional edges using a "command" function (to be covered in dynamic workflows)
- Conditional workflows can have more than two branches - there's no limit
- The discriminant example demonstrates non-LLM conditional logic
- The customer support example shows real-world LLM-based conditional routing
- Proper prompt engineering is crucial for accurate sentiment analysis and diagnosis