---
title: "Iterative Workflows in LangGraph"
layout: default
nav_order: 9
parent: "Lecture Notes"
description: "Lecture notes: Iterative Workflows in LangGraph"
last_modified_date: 2026-01-09
source_transcript: "009_Iterative_Workflows_in_LangGraph_Agentic_AI_using_LangGraph_Video_8_CampusX"
generated_by: "NoteyBoy"
---

# Iterative Workflows in LangGraph

## Overview

This lecture covers the implementation of iterative (looping) workflows in LangGraph, the fourth type of workflow pattern after sequential, parallel, and conditional workflows. The session demonstrates building a real-world automated tweet generation system that uses multiple LLMs in a feedback loop to iteratively improve content quality through generation, evaluation, and optimization cycles.

## Key Concepts

### **Iterative Workflows**
Iterative or looping workflows are systems where tasks cycle between two or more nodes to progressively improve an output. Unlike linear workflows, these allow for repeated refinement until quality criteria are met or a maximum iteration limit is reached.

### **Multi-Agent Architecture**
The workflow employs three specialized LLM agents:
- **Generator**: Creates initial content based on a topic
- **Evaluator**: Assesses content quality against strict criteria
- **Optimizer**: Improves content based on feedback

## Workflow Architecture

### Problem Statement
Creating quality social media content automatically is challenging because:
- First-attempt outputs from LLMs are often mediocre
- Content may be repetitive or lack originality
- Manual quality control is time-consuming

### Solution Design

The workflow implements a three-stage iterative process:

1. **Generation Stage**
   - Input: Topic from user
   - Process: LLM generates initial tweet
   - Output: Draft tweet content

2. **Evaluation Stage**
   - Input: Generated tweet
   - Process: Evaluator LLM judges against criteria
   - Output: "Approved" or "Needs Improvement" + feedback

3. **Optimization Stage** (conditional)
   - Input: Tweet + evaluator feedback
   - Process: Optimizer LLM refines content
   - Output: Improved tweet (loops back to evaluation)

### Workflow Flow Diagram

```
START → Generate → Evaluate → [Decision Point]
                      ↑              ↓
                      |         Approved → END
                      |              ↓
                      └─── Optimize ←┘
                         (Loop)
```

## Implementation Details

### State Definition

```python
class TweetState(TypedDict):
    topic: str  # User-provided topic
    tweet: str  # Generated tweet content
    evaluation: Literal["approved", "needs_improvement"]
    feedback: str  # Evaluator's feedback
    iteration: int  # Current iteration count
    max_iteration: int  # Maximum allowed iterations (default: 5)
    tweet_history: Annotated[list[str], operator.add]
    feedback_history: Annotated[list[str], operator.add]
```

**Key State Variables:**
- **topic**: The subject for tweet generation
- **tweet**: Current version of the tweet
- **evaluation**: Binary decision from evaluator
- **feedback**: Detailed improvement suggestions
- **iteration**: Tracks loop cycles
- **max_iteration**: Prevents infinite loops
- **tweet_history/feedback_history**: Maintains audit trail using reducer functions

### LLM Configuration

```python
# Three specialized LLMs for different tasks
generator_llm = ChatOpenAI(model="gpt-4o")
evaluator_llm = ChatOpenAI(model="gpt-4o-mini")
optimizer_llm = ChatOpenAI(model="gpt-4o")
```

**Design Consideration**: In production, each LLM should be optimized for its specific task (e.g., creative writing for generator, strict rule-following for evaluator).

### Node Functions

#### 1. Generate Tweet Node

```python
def generate_tweet(state: TweetState):
    messages = [
        SystemMessage(content="You are a funny and clever Twitter influencer"),
        HumanMessage(content=f"""Write a short, original, and hilarious tweet on {state['topic']}.
        Rules:
        - No Q&A format
        - Under 280 characters
        - Use observational humor, irony, sarcasm
        - Think in meme logic and punchlines
        - Use simple day-to-day English""")
    ]
    
    response = generator_llm.invoke(messages).content
    return {
        "tweet": response,
        "tweet_history": [response]
    }
```

**Purpose**: Creates initial tweet content with specific style guidelines.

#### 2. Evaluate Tweet Node

```python
class TweetEvaluationSchema(BaseModel):
    evaluation: Literal["approved", "needs_improvement"]
    feedback: str

def evaluate_tweet(state: TweetState):
    structured_evaluator_llm = evaluator_llm.with_structured_output(
        TweetEvaluationSchema
    )
    
    messages = [
        SystemMessage(content="You are a ruthless, no-laugh-given Twitter critic"),
        HumanMessage(content=f"""Evaluate this tweet: {state['tweet']}
        
        Criteria:
        - Originality: Is it fresh or overused?
        - Humor: Does it make you laugh?
        - Punchiness: Is it concise and impactful?
        - Virality potential: Would people share it?
        - Format: Proper structure?
        
        Auto-reject if:
        - Q&A format
        - Over 280 characters
        - Traditional jokes""")
    ]
    
    response = structured_evaluator_llm.invoke(messages)
    return {
        "evaluation": response.evaluation,
        "feedback": response.feedback,
        "feedback_history": [response.feedback]
    }
```

**Key Features:**
- Uses **structured output** to ensure consistent response format
- Implements strict evaluation criteria
- Provides actionable feedback for improvement

#### 3. Optimize Tweet Node

```python
def optimize_tweet(state: TweetState):
    messages = [
        SystemMessage(content="You punch up tweets for virality and humor"),
        HumanMessage(content=f"""Improve this tweet based on feedback:
        
        Topic: {state['topic']}
        Original tweet: {state['tweet']}
        Feedback: {state['feedback']}
        
        Rewrite as a short, viral-worthy tweet.
        Avoid Q&A style and stay under 280 characters.""")
    ]
    
    response = optimizer_llm.invoke(messages).content
    iteration = state['iteration'] + 1
    
    return {
        "tweet": response,
        "iteration": iteration,
        "tweet_history": [response]
    }
```

**Purpose**: Refines content based on evaluator feedback and increments iteration counter.

### Graph Construction

#### Adding Nodes

```python
graph = StateGraph(TweetState)

graph.add_node("generate", generate_tweet)
graph.add_node("evaluate", evaluate_tweet)
graph.add_node("optimize", optimize_tweet)
```

#### Creating Edges

**Sequential Edges:**
```python
graph.add_edge(START, "generate")
graph.add_edge("generate", "evaluate")
```

**Conditional Edge (Decision Point):**
```python
def route_evaluation(state: TweetState):
    if (state['evaluation'] == "approved" or 
        state['iteration'] >= state['max_iteration']):
        return "approved"
    else:
        return "needs_improvement"

graph.add_conditional_edges(
    "evaluate",
    route_evaluation,
    {
        "approved": END,
        "needs_improvement": "optimize"
    }
)
```

**Loop Edge:**
```python
graph.add_edge("optimize", "evaluate")  # Creates the feedback loop
```

### Loop Control Mechanism

The workflow prevents infinite loops through two conditions:

1. **Quality Threshold**: Tweet meets evaluation criteria
2. **Iteration Limit**: Maximum iterations reached (default: 5)

```python
if state['evaluation'] == "approved" or state['iteration'] >= state['max_iteration']:
    # Exit loop
    return "approved"
else:
    # Continue loop
    return "needs_improvement"
```

## Practical Applications

### Use Case: Social Media Automation
- **Problem**: Time-consuming content creation for multiple platforms
- **Solution**: Automated tweet generation with quality control
- **Benefits**: 
  - Saves time while maintaining quality
  - Consistent brand voice
  - Scalable content production

### Quality Assurance Features

1. **Strict Evaluation Criteria**: Prevents generic or low-quality content
2. **Iterative Refinement**: Progressive improvement through feedback
3. **History Tracking**: Audit trail of all iterations
4. **Fail-Safe Mechanism**: Maximum iteration limit prevents resource waste

## Advanced Features

### History Tracking with Reducers

```python
tweet_history: Annotated[list[str], operator.add]
feedback_history: Annotated[list[str], operator.add]
```

**Purpose**: The `operator.add` reducer function ensures new values are appended to lists rather than replacing them, creating a complete history of the refinement process.

### Viewing Iteration History

```python
result = workflow.invoke(initial_state)

# Print all tweet versions
for tweet in result['tweet_history']:
    print(tweet)

# Print all feedback received
for feedback in result['feedback_history']:
    print(feedback)
```

## Key Takeaways

1. **Iterative workflows enable progressive refinement** through feedback loops between specialized agents

2. **Loop control is critical** - always implement maximum iteration limits to prevent infinite loops

3. **Structured outputs ensure consistency** when LLMs need to make binary decisions or provide formatted feedback

4. **State management is crucial** - track iterations, history, and intermediate results for debugging and analysis

5. **Specialized LLMs perform better** - use different models optimized for generation, evaluation, and optimization tasks

6. **Conditional edges create decision points** that determine whether to exit or continue the loop

7. **Real-world applications** include content generation, code review systems, document refinement, and quality assurance workflows

## Future Enhancements

The instructor mentions upcoming improvements:
- **Tools integration**: Adding API calls to post directly to platforms
- **Human-in-the-loop**: Manual approval before final posting
- **Multi-platform support**: Extending beyond Twitter to LinkedIn, Instagram, etc.

## Summary

Iterative workflows in LangGraph enable sophisticated multi-agent systems that can self-improve through feedback loops. By combining generation, evaluation, and optimization nodes with conditional routing and loop control mechanisms, you can build robust automated systems that produce high-quality outputs while preventing common pitfalls like infinite loops. This pattern is essential for building production-grade agentic AI applications that require quality assurance and progressive refinement.