---
title: "Human in the Loop (HITL) using LangGraph"
layout: default
nav_order: 21
parent: "Lecture Notes"
description: "Lecture notes: Human in the Loop (HITL) using LangGraph"
last_modified_date: 2026-01-09
source_transcript: "021_Human_in_the_loop_(HITL)_using_LangGraph_CampusX"
generated_by: "NoteyBoy"
---

# Human in the Loop (HITL) using LangGraph

## Overview

Human in the Loop (HITL) is a critical design approach in agentic AI systems where humans actively participate at crucial decision points to supervise, approve, correct, and guide the AI model's output. This lecture covers the theoretical foundations of HITL, its implementation in LangGraph, and practical examples demonstrating how to integrate human checkpoints into AI workflows to improve accuracy, safety, and accountability.

## Key Concepts

### What is HITL?

**Human in the Loop (HITL)** is a design approach in AI systems where a human actively participates at critical points of the AI workflow to supervise, approve, correct, and guide the model's output. Think of HITL as putting a human checkpoint inside an AI pipeline so that important decisions are not made autonomously by the model.

**Key Characteristics:**
- Pauses AI execution at critical junctures
- Requires human judgment before proceeding
- Maintains human oversight over important decisions
- Balances automation with human control

### Why HITL Exists in Agentic Systems

#### 1. **Helping Imperfect AI Systems**

Current LLMs are not perfect and can:
- **Misinterpret user goals** - Understanding user intent incorrectly
- **Handle ambiguity poorly** - Struggling with unclear or vague requests
- **Hallucinate** - Generating incorrect or fabricated information

**Example:** A user requests "Book flight tickets for next Friday." If today is Monday, this could mean:
- This week's Friday (4 days away)
- Next week's Friday (11 days away)

HITL allows the system to clarify ambiguity with the user before proceeding.

#### 2. **Adding Accountability**

AI systems cannot be held accountable for mistakes. HITL ensures:
- A human is responsible for critical decisions
- Users can be blamed (not just the AI) if something goes wrong
- Legal and ethical responsibility is maintained

**Example:** Email reply generation - Instead of automatically sending replies, the system shows the draft to the user for approval before sending.

## Benefits of HITL

### 1. **Improved Accuracy**
- Reduces errors by incorporating human judgment
- Catches misinterpretations before they cause problems
- Example: Invoice processing where AI might misread ₹1,200 as ₹12,000

### 2. **Enhanced Safety**
- Prevents dangerous or irreversible actions
- Provides safeguards for critical operations
- Example: Confirming before deleting files that are part of current projects

### 3. **Ethical Alignment**
- Ensures AI responses align with company values
- Allows adjustment of tone and empathy
- Example: Customer support responses can be modified to show more empathy

### 4. **Better User Experience**
- Creates synergy between human intelligence and AI capabilities
- Builds user trust through transparency
- Provides control over important decisions

## Common HITL Patterns

### 1. **Action Approval Pattern** (Most Common)
The AI requests permission before performing critical actions.

**Use Cases:**
- Making payments
- Sending important emails
- Deleting files from servers
- Booking tickets

**Flow:** AI prepares action → Asks human for approval → Proceeds only if approved

### 2. **Output Review/Edit Pattern**
The AI generates content but allows human review before final action.

**Use Cases:**
- Blog post generation and publishing
- Social media content creation
- Research report generation

**Flow:** AI generates draft → Human reviews/edits → AI publishes approved version

### 3. **Ambiguity Clarification Pattern**
The AI asks for human input when confused or uncertain.

**Use Cases:**
- Unclear user requests
- Multiple valid interpretations
- Missing information

**Flow:** AI detects ambiguity → Asks clarifying questions → Proceeds with clarified information

### 4. **Escalation Pattern**
The AI recognizes its limitations and transfers control to humans.

**Use Cases:**
- Complex customer support issues
- Edge cases beyond AI capabilities
- Situations requiring human expertise

**Flow:** AI attempts resolution → Recognizes limitation → Escalates to human agent

## HITL Implementation in LangGraph

### Core Components

#### 1. **The `interrupt()` Function**
Pauses graph execution and requests human input.

**What it does:**
1. **Pauses execution** at the current node
2. **Saves current state** using checkpointers
3. **Prepares a message** for the frontend
4. **Sends the message** to request human input

#### 2. **The `command` Parameter**
Resumes graph execution with human input.

**Usage:**
```python
graph.invoke(input, config={"configurable": {"thread_id": "123"}}, 
             command={"resume": {"approved": user_decision}})
```

### Technical Flow

**Step-by-Step Process:**

1. **Initial Invocation**
   - User submits input through frontend
   - `graph.invoke()` is called
   - Graph execution begins

2. **Interrupt Triggered**
   - Node encounters `interrupt()` function
   - Execution pauses
   - Current state saved to checkpointer
   - Message sent to frontend

3. **Human Input**
   - Frontend receives interrupt message
   - Displays message to user
   - Collects user decision (yes/no)

4. **Resume Execution**
   - Frontend calls `graph.invoke()` again with `command` parameter
   - Graph loads saved state from checkpointer
   - Continues from exact interruption point
   - Processes human decision

5. **Completion**
   - Graph completes based on human input
   - Final output returned

### State Management

**Checkpointers are essential** for HITL because:
- State must be saved when execution pauses
- State must be restored when execution resumes
- Options include:
  - `MemorySaver` (in-memory, for development)
  - `SqliteSaver` (persistent, for production)

## Practical Examples

### Example 1: Simple Question Approval System

**Scenario:** User asks a question to an LLM, but system confirms before sending the query.

**Workflow:**
```
User asks question → System asks "Are you sure?" → 
If Yes: Send to LLM → Return answer
If No: Reject and exit
```

**Key Code Components:**
```python
def chat_node(state: ChatState):
    # Interrupt and ask for approval
    decision = interrupt({
        "type": "approval",
        "reason": "Confirm query submission",
        "question": state["messages"][-1].content,
        "instruction": "Approve or reject?"
    })
    
    if decision.get("approved") == "no":
        return {"messages": [AIMessage(content="Not approved")]}
    
    # If approved, proceed with LLM
    response = llm.invoke(state["messages"])
    return {"messages": [response]}
```

### Example 2: Stock Trading Chatbot

**Scenario:** Chatbot can check stock prices and purchase stocks, but requires approval for purchases.

**Tools:**
1. **get_stock_price** - Fetches current stock price (no approval needed)
2. **purchase_stocks** - Buys stocks (requires HITL approval)

**Implementation:**
```python
@tool
def purchase_stocks(company: str, quantity: int):
    """Purchase stocks with human approval"""
    # HITL checkpoint
    decision = interrupt({
        "message": f"Approve buying {quantity} shares of {company}?",
        "type": "approval"
    })
    
    if isinstance(decision, str) and decision.lower() == "yes":
        return f"Successfully purchased {quantity} shares of {company}"
    else:
        return "Purchase cancelled"
```

**User Experience:**

**Without HITL:**
```
User: What is the stock price of Apple?
Bot: Current price is $278
User: Purchase 10 stocks
Bot: Successfully purchased 10 shares of Apple
```
*Problem: No confirmation, risky for financial transactions*

**With HITL:**
```
User: What is the stock price of Apple?
Bot: Current price is $278
User: Purchase 10 stocks
Bot: Approve buying 10 shares of Apple? (yes/no)
User: yes
Bot: Successfully purchased 10 shares of Apple
```
*Solution: Human approval required before financial transaction*

## Code Structure

### Basic HITL Node Structure

```python
# Define state
class ChatState(TypedDict):
    messages: Annotated[list, add_messages]

# Create HITL node
def hitl_node(state: ChatState):
    # Interrupt execution
    decision = interrupt({
        "type": "approval",
        "message": "Your approval message here",
        "data": relevant_data
    })
    
    # Process decision
    if decision == "approved":
        # Proceed with action
        return perform_action(state)
    else:
        # Reject action
        return {"messages": [AIMessage(content="Action rejected")]}

# Build graph
graph = StateGraph(ChatState)
graph.add_node("hitl_node", hitl_node)
graph.add_edge(START, "hitl_node")
graph.add_edge("hitl_node", END)

# Compile with checkpointer (required!)
memory = MemorySaver()
app = graph.compile(checkpointer=memory)
```

### Frontend Interaction Pattern

```python
# First invocation
thread_id = "unique_thread_id"
config = {"configurable": {"thread_id": thread_id}}

result = app.invoke(initial_input, config)

# Check for interrupt
if "interrupt" in result:
    interrupt_msg = result["interrupt"]
    
    # Display to user and get input
    user_decision = input(interrupt_msg["message"])
    
    # Resume with command
    final_result = app.invoke(
        None,
        config=config,
        command={"resume": user_decision}
    )
```

## Best Practices

### When to Use HITL

**Always use HITL for:**
- Financial transactions (payments, purchases)
- Data deletion or modification
- Sending communications on behalf of users
- Actions with legal implications
- Irreversible operations

**Consider HITL for:**
- Ambiguous user requests
- High-stakes decisions
- Operations requiring domain expertise
- Situations where AI confidence is low

### Design Considerations

1. **Clear Messages**: Make interrupt messages clear and actionable
2. **Minimal Friction**: Don't overuse HITL - only for critical points
3. **Timeout Handling**: Consider what happens if user doesn't respond
4. **State Persistence**: Always use appropriate checkpointers
5. **Error Handling**: Handle cases where user provides unexpected input

## Summary

### Key Takeaways

1. **HITL is Essential**: Nearly all production agentic AI systems require HITL for safety and accountability

2. **Two Main Reasons for HITL**:
   - Helping imperfect AI systems avoid errors
   - Adding accountability to AI decisions

3. **Implementation is Simple**: LangGraph makes HITL implementation intuitive using `interrupt()` and `command` parameters

4. **Checkpointers are Required**: State must be saved and restored when execution pauses and resumes

5. **Multiple Invocations**: HITL workflows require multiple `invoke()` calls - one initial call and one for each human input point

6. **Balance is Key**: Use HITL strategically at critical points without creating excessive friction

7. **Four Common Patterns**: Action Approval, Output Review, Ambiguity Clarification, and Escalation

### Practical Applications

- **Customer Support**: Escalate complex issues to human agents
- **Content Generation**: Review before publishing
- **Financial Services**: Approve transactions before execution
- **Travel Booking**: Confirm bookings before payment
- **Email Management**: Review generated replies before sending

HITL represents the sweet spot between full automation and complete human control, enabling AI systems to be both powerful and trustworthy.