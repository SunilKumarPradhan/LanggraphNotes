---
title: "Comprehensive Lecture Notes: Agentic AI using LangGraph"
layout: default
nav_order: 3
parent: "Lecture Notes"
description: "Lecture notes: Comprehensive Lecture Notes: Agentic AI using LangGraph"
last_modified_date: 2026-01-09
source_transcript: "003_What_is_Agentic_AI_Agentic_AI_using_LangGraph_Video_2_CampusX"
generated_by: "NoteyBoy"
---

# Comprehensive Lecture Notes: Agentic AI using LangGraph

## Course Context
This is the second video in the Agentic AI using LangGraph playlist. The first video covered differences between Generative AI and Agentic AI. This lecture provides a formal, comprehensive study of Agentic AI fundamentals.

---

## Table of Contents
1. What is Agentic AI?
2. Six Characteristics of Agentic AI
3. Five Core Components of Agentic AI Systems

---

# 1. What is Agentic AI?

## Definition
**Agentic AI** is a type of artificial intelligence system that can:
- Accept a task and goal from a user
- Work autonomously toward completion with minimal human guidance
- Plan, execute actions, and adapt to changing circumstances
- Seek human assistance only when necessary

## Key Distinction: Generative AI vs. Agentic AI

### Generative AI (Reactive Approach)
- **Behavior:** Responds only to specific prompts
- **User Role:** Must guide every step of the process
- **Example - Planning a Goa Trip:**
  - User: "Best way to reach Goa on 15th?"
  - ChatGPT: Answers only that specific question
  - User: "Which hotels should I book?"
  - ChatGPT: Answers only that specific question
  - **Result:** Step-by-step process with user constantly driving

### Agentic AI (Proactive Approach)
- **Behavior:** Takes initiative and works independently
- **User Role:** Provides only the high-level goal
- **Example - Planning a Goa Trip:**
  - User: "I want to go to Goa from date X to Y"
  - System autonomously:
    - Finds best travel options
    - Recommends hotels
    - Plans complete itinerary
    - Takes initiative without constant prompting

---

# 2. Practical Example: AI HR Recruiter

## Scenario Setup
- **Role:** HR Recruiter
- **Task:** Hire a Backend Engineer
- **Tool:** Agentic AI Chatbot
- **Initial Input:** "I want to hire a backend engineer, remote, 2-4 years experience"

## Complete Workflow Demonstration

### Phase 1: Goal Understanding & Planning
The AI creates a comprehensive plan:
1. Draft Job Description (JD)
2. Post on optimal job platforms (LinkedIn, Naukri)
3. Monitor applications continuously
4. Screen candidates
5. Schedule interviews
6. Send offer letters
7. Initiate onboarding process

### Phase 2: Autonomous Execution

#### **Step 1: JD Creation**
- Accesses company documents
- Determines required technologies
- Identifies appropriate salary range for 2-4 years experience
- Drafts JD and requests approval

#### **Step 2: Job Posting**
- Posts on LinkedIn and Naukri using APIs
- Notifies user: "Job posted, monitoring applications"

#### **Step 3: Adaptive Response**
- **Situation:** After 3 days, only 2 applications received
- **AI Analysis:** Below expectations
- **Suggested Changes:**
  - Modify title from "Backend Engineer" to "Full Stack Engineer"
  - Run LinkedIn ads to promote posting
- **Action:** Requests permission and implements changes

#### **Step 4: Candidate Screening**
- Downloads and analyzes resumes using resume parser tool
- **Categorization:**
  - 2 strong candidates
  - 3 partial matches
  - 3 weak matches
- Suggests scheduling interviews with strong candidates

#### **Step 5: Interview Scheduling**
- Checks HR recruiter's calendar
- Identifies available time slots
- Drafts and sends emails to candidates
- Sends reminder on interview day
- Provides list of interview questions

#### **Step 6: Offer Management**
- Drafts offer letter using company documents
- Sends for human review
- Emails offer letter to selected candidate
- Monitors acceptance status

#### **Step 7: Onboarding**
- Sends welcome email
- Submits IT access request
- Provisions laptop
- Schedules joining day meeting

### Key Observation
The AI operates **completely autonomously**, planning, executing, and adapting with minimal human intervention, seeking permission only for critical decisions.

---

# 3. Six Characteristics of Agentic AI

## 3.1 Autonomy

### Definition
The ability to make decisions and take actions independently to achieve a given goal without requiring step-by-step human instructions.

### Key Features
- **Proactive** (not reactive)
- Takes initiative before being asked
- Example: AI recruiter monitoring applications and suggesting changes without prompting

### Three Types of Autonomy

#### **1. Execution Autonomy**
- Automatically executes plan steps sequentially
- Flow: JD creation → Posting → Screening → Interviewing → Offering

#### **2. Decision-Making Autonomy**
- Decides which candidates to shortlist
- Determines screening criteria
- Example: Categorizing candidates as strong/partial/weak matches

#### **3. Tool Usage Autonomy**
- Selects appropriate tools for each task
- Examples: Mail API, Calendar API, Resume Parser, LinkedIn API

### Controlling Autonomy

**Why Control is Necessary:** Prevent risks and ensure safe operation

#### **Four Control Methods:**

**1. Define Scope**
- Limit which tools and actions agent can perform independently
- Example: "Screen all candidates but ask before rejecting anyone"

**2. Human-in-the-Loop**
- Insert checkpoints requiring human approval
- Example: JD can be drafted but needs approval before posting

**3. Override Controls**
- Allow users to stop, pause, or change agent behavior anytime
- Example: Command "Pause hiring" stops all activities

**4. Guard Rails and Policies**
- Define hard rules and ethical boundaries
- Examples:
  - "Never schedule interviews on weekends"
  - "Never use informal language in communications"

### Risks of Uncontrolled Autonomy
- Rolling out job offers with incorrect salaries/terms
- Biased shortlisting based on nationality or age
- Spending unlimited money on LinkedIn ads without permission

---

## 3.2 Goal-Oriented

### Definition
The AI system operates with a persistent objective in mind and continuously directs its actions to achieve that objective.

### Key Principles
- All planning and execution directed toward achieving the goal
- Goal acts as a **compass for autonomy**
- Without a goal, autonomous functioning is impossible

### Types of Goals

#### **1. Independent Goals**
- Simple, standalone objectives
- Example: "Hire a backend engineer"

#### **2. Goals with Constraints**
- Objectives with specific limitations
- Examples:
  - "Hire a backend engineer from India" (location constraint)
  - "Hire remotely only" (work mode constraint)
  - "Budget: $X" (cost constraint)

### Goal Storage in Memory

Goals are stored in core memory as structured data:

```json
{
  "main_goal": "Hire a backend engineer",
  "constraints": {
    "experience": "2-4 years",
    "remote": true,
    "tech_stack": ["Python", "Django", "AWS"]
  },
  "status": "active",
  "created_at": "2024-01-01",
  "progress": {
    "jd_created": true,
    "posted_on": ["LinkedIn", "Naukri"],
    "applications_received": 8,
    "interviews_scheduled": 2,
    "hired": false,
    "onboarding_started": false
  }
}
```

### Goal Modification
- Goals can be altered mid-execution
- Example: Change from "Hire backend engineer" to "Find a freelancer for project"
- System adapts planning and execution accordingly

---

## 3.3 Planning

### Definition
The agent's ability to break down a high-level goal into a structured sequence of actions and sub-goals.

### Importance
- **One of the most critical characteristics**
- Agentic AI operates in two main iterative steps:
  1. **Planning:** Create plan to achieve goal
  2. **Execution:** Execute the plan step-by-step
- This is an **iterative process** - may need to re-plan if steps fail

### Planning as a Search Problem
- **Initial State:** "Company needs backend engineer"
- **Final State:** "Backend engineer hired"
- **Challenge:** Multiple paths exist from initial to final state
- **Objective:** Find the most optimized path

### Three Steps of Planning

#### **Step 1: Generate Multiple Candidate Plans**
Create alternative approaches, not just one plan:

- **Plan A:** Post on job portals (LinkedIn, GitHub Jobs, AngelList) and promote
- **Plan B:** Use internal referrals or hiring agency

#### **Step 2: Evaluate Plans**
Assess each plan based on evaluation criteria (specific criteria to be covered in advanced sessions)

#### **Step 3: Select Optimal Plan**
Choose the best plan based on evaluation results

---

# 4. Five Core Components of Agentic AI Systems

## Component 1: Brain

### Primary Functions
- **Decision-making center** of the agentic system
- Processes information and determines actions
- Houses the core intelligence

### Sub-components

#### **1. Planner**
- Creates multiple plans
- Generates different approaches to solve problems
- Explores alternative pathways to goals

#### **2. Evaluator**
- Evaluates generated plans
- Selects optimal approach
- Assesses feasibility and effectiveness

### Note on Complexity
- High-level overview provided for beginners
- Production systems contain many more sub-components
- Can be expanded based on specific requirements

---

## Component 2: Orchestrator

### Definition
The workflow coordination engine that manages execution flow.

### Primary Functions
- Coordinates execution of planned steps
- Manages workflow sequences
- Ensures proper order of operations
- Handles transitions between different phases

### Role in System
- Acts as the **execution manager**
- Bridges planning (Brain) and action (Tools)
- Maintains system coherence during complex operations

---

## Component 3: Tools

### Definition
**Tools are like hands and legs** for AI agents - they enable interaction with external systems and execution of actions.

### Functions
- Making database changes
- Sending emails
- Executing various system operations
- Interfacing with external APIs and services

### RAG as a Tool

#### **Knowledge Base = Tool**
- Example: AI Recruiter accessing company documents
- **RAG (Retrieval Augmented Generation)** functions:
  - Retrieves factual information
  - Provides domain-specific information
  - Grounds responses in actual data
  - Ensures accuracy and relevance

### Tool Selection
- Agent autonomously selects appropriate tools for each task
- Examples: Mail API, Calendar API, Resume Parser, LinkedIn API

---

## Component 4: Memory

### Purpose
Enables continuity and context retention across interactions.

### Three Types of Memory

#### **1. Short-term Memory**
Stores current session information:
- User messages
- Tool calls made
- Immediate decisions
- Active conversation context

#### **2. Long-term Memory**
Stores persistent information:
- High-level goals
- Past interactions
- User preferences
- Cross-session decisions
- Historical data

#### **3. State Tracking**
Monitors progress:
- Work completed so far
- Remaining tasks
- Current status of operations
- Progress toward goals

### Importance
- **Enables continuity** both within and across sessions
- Allows agent to maintain context
- Supports learning from past interactions

---

## Component 5: Supervisor

### Purpose
**Implements "Human-in-the-Loop" concept** - facilitates collaboration between agent and human.

### Primary Role
- Acts as intermediary for critical decisions
- Ensures human oversight where necessary
- Maintains safety and compliance

### Four Key Use Cases

#### **1. Approval for High-Risk Actions**
Examples:
- Sending offer letters
- Running LinkedIn ads
- Financial transactions
- Legal commitments

#### **2. Enforcing Guard Rails**
- Ensures system operates within defined boundaries
- Maintains compliance with rules and policies
- Prevents violations of constraints

#### **3. Handling Edge Cases**

**Example Scenario:**
- **Guard rail:** "Only hire candidates from IITs and NITs"
- **Edge case:** Candidate from non-IIT/NIT with exceptional resume
- **Supervisor action:** Alerts human for review and decision
- **Outcome:** Allows flexibility while maintaining oversight

#### **4. Escalations**
- Notifies humans when situations require judgment
- Allows human override of automated decisions
- Manages exceptions to standard workflows

### Importance
- **Human oversight remains crucial** for safety
- Balances automation with accountability
- Provides safety net for autonomous operations

---

# Summary & Key Takeaways

## Topics Covered in This Lecture

### ✅ Core Concepts
1. **Definition** of Agentic AI
2. **Practical scenario** demonstration (AI Recruiter example)
3. **How Agentic AI works** in real applications

### ✅ Six Key Characteristics
1. Autonomy
2. Goal-Oriented
3. Planning
4. (Additional characteristics to be covered in subsequent sessions)

### ✅ Five Main Components
1. **Brain** - Decision-making and planning
2. **Orchestrator** - Workflow coordination
3. **Tools** - External interactions and actions
4. **Memory** - Short-term and long-term information storage
5. **Supervisor** - Human-in-the-loop implementation

## Essential Takeaways

### **Agentic AI is Modular**
- Composed of distinct, interconnected components
- Each component serves specific functions
- Components work together seamlessly

### **Human Oversight Remains Crucial**
- Supervisor component ensures safety
- Human-in-the-loop prevents risks
- Balance between automation and control

### **Memory Enables Continuity**
- Functions both within and across sessions
- Maintains context and learning
- Supports progressive improvement

### **Tools Extend Capabilities**
- Allow agents to interact with real systems
- Enable practical actions beyond conversation
- Bridge AI intelligence with real-world operations

### **Complexity Can Scale**
- Basic components can be subdivided for sophistication
- Production systems involve significantly more complexity
- Framework is extensible and adaptable

---

## Looking Forward

### Future Topics
- Remaining characteristics of Agentic AI
- Advanced planning techniques
- Deeper dive into each component
- Practical implementations using LangGraph
- Production-level considerations

### Note for Learners
*This is a beginner-friendly introduction. Production systems involve significantly more complexity and additional sub-components. Future videos will refine and deepen this foundational knowledge.*

---

**End of Lecture Notes**