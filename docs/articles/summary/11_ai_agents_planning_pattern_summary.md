# AI Agents Crash Course - Part 11: Complete Summary

**Implementing Planning Agentic Pattern From Scratch**

**Authors:** Avi Chawla & Akshay Pachaar  
**Source:** Daily Dose of Data Science  
**Date:** April 20, 2025

## 🎯 Executive Summary

Part 11 introduces the **Planning Pattern** as an alternative to the ReAct pattern, focusing on implementing a complete planning-based agent system from scratch using pure Python and LLMs. This article demonstrates how to build agents that think strategically before acting, creating structured execution plans that enhance reliability and efficiency for complex multi-step tasks.

---

## 🆕 New Concepts Beyond Parts 1-10

### **The Planning Pattern Architecture**

Unlike ReAct's reactive "Thought → Action → Observation" loop, the Planning pattern follows a **"Plan First, Execute Later"** approach:

**Core Philosophy:**
- **Planning Phase:** Agent creates a comprehensive roadmap before taking any actions
- **Execution Phase:** Sequential execution of planned steps with observations
- **Strategic Thinking:** Global view of the problem rather than step-by-step reactions

**Key Workflow:**
1. **User Query** → **Planner Agent** → **Generate Task List**
2. **Task Execution** → **Update State** → **Re-plan if needed**
3. **Final Response** after all planned tasks complete

---

## 🧠 Why Planning Over ReAct?

### **Limitations of ReAct for Complex Tasks**
- **Reactive Decision-Making:** Only plans one action at a time
- **Suboptimal Trajectories:** May wander without clear direction
- **Missing Steps:** Can skip critical intermediate steps
- **No Global Strategy:** Lacks comprehensive problem understanding

### **Planning Pattern Advantages**
- **Strategic Sequencing:** Creates complete action plans upfront
- **Comprehensive Coverage:** Less likely to miss important steps
- **Interpretable Process:** Clear roadmap for complex tasks
- **Better for Multi-Step Problems:** Ideal for tasks requiring coordinated actions

### **Use Case Comparison**
- **ReAct Best For:** Simple tasks, exploratory problems, uncertain environments
- **Planning Best For:** Multi-step workflows, structured problems, strategic execution

**Example Scenarios Where Planning Excels:**
- **Report Writing:** Research → Outline → Write sections → Compile
- **Trip Planning:** Destination → Flights → Hotels → Activities → Itinerary
- **Project Management:** Requirements → Components → Implementation order

---

## 🔍 Literature Review: Planning in AI Agents

### **1. Plan-and-Solve Prompting Research**
**Key Finding:** Forcing LLMs to explicitly plan before executing dramatically improves accuracy on reasoning tasks.

**Research Results:**
- **Zero-shot Plan-and-Solve consistently outperforms Zero-Shot-CoT by large margins**
- Matches accuracy of few-shot methods without requiring examples
- **Planning imposes structure that improves thoroughness**

### **2. Tree-of-Thought (ToT) Planning**
**Innovation:** Explores multiple possible plans rather than committing to single path

**Benefits:**
- **Branching and Backtracking:** Can revise plans when needed
- **Multiple Reasoning Paths:** Evaluates different approaches
- **Dramatic Performance Gains:** 74% success vs 4% for chain-of-thought on puzzle tasks

### **3. Reflection and Self-Correction**
**Concept:** Agents can review and improve their own plans and outputs

**Process:**
1. Generate initial solution
2. Self-reflect: "Did I make an error? What should I do differently?"
3. Refine approach based on self-feedback
4. Retry with improved strategy

### **4. Function Calling Integration**
**OpenAI's Contribution:** Structured tool usage through API contracts

**Benefits:**
- **Deterministic Tool Calls:** Predefined function signatures
- **Structured Outputs:** JSON-formatted action plans
- **Parsing Reliability:** Eliminates regex brittleness

---

## 💻 Technical Implementation Deep Dive

### **Core Agent Architecture**

**MyAgent Class Structure:**
```python
class MyAgent:
    def __init__(self, system=""):
        self.system = system          # Planning protocol
        self.messages = []           # Conversation history
    
    def complete(self, message=""):
        # LLM interaction with context retention
```

### **Planning System Prompt Design**

**Behavioral Protocol:**
```
You are a smart planning agent.
You act in iterations and do JUST ONE thing in a single iteration:

1) "Plan" to plan the steps needed to answer the question
2) "Execute" to execute the planned steps, one step at a time  
3) "Observation" to get the output of the execution
4) "Collect" to just collect the result of all the steps
5) "Answer" to answer the user's question using the collected results
```

**Key Design Elements:**
- **Iterative Structure:** One action per iteration prevents rushing
- **Clear Phase Separation:** Distinct planning vs execution phases
- **Tool Constraints:** Specific syntax requirements for tool calls
- **Sequential Enforcement:** Must plan before executing

### **Tool Integration Pattern**

**Available Tools:**
- **math:** Python expression evaluation
- **lookup_population:** Country population data

**Tool Call Syntax:**
```
Action: tool_name: parameter
Example: lookup_population: Japan
```

### **Execution Approaches**

### **1. Manual Execution (Educational)**
**Process:**
- Step-by-step agent interaction
- Manual observation injection
- Complete visibility into reasoning
- Educational insight into mechanics

**Benefits:**
- **Full Transparency:** See each decision point
- **Control:** Manual intervention capability
- **Understanding:** Deep insight into agent thinking
- **Debugging:** Easy problem identification

### **2. Automated Execution (Production)**
**Process:**
- **agent_loop()** function handles full workflow
- Automatic tool execution and observation injection
- Regex parsing for action extraction
- Loop termination on "Answer:" detection

**Controller Function Features:**
- **Query Processing:** Natural language input
- **Plan Understanding:** Automatic plan parsing
- **Tool Execution:** Dynamic function calling
- **State Management:** Context tracking across iterations

---

## 🔧 Implementation Examples

### **Example Query Processing**
**Input:** "What is the population of Japan plus the population of India?"

**Planning Phase Output:**
```
Plan:
1. Use lookup_population on Japan.
2. Use lookup_population on India.  
3. Use math to add the two populations.
```

**Execution Flow:**
1. **Execute:** lookup_population: Japan → **Observation:** 125000000
2. **Execute:** lookup_population: India → **Observation:** 1400000000  
3. **Execute:** math: (125000000 + 1400000000) → **Observation:** 1525000000
4. **Collect:** All intermediate results summarized
5. **Answer:** "The total population of Japan and India is approximately 1.525 billion."

### **Tool Execution Mechanics**
**Regex Pattern:** `r"Action:\s*(\w+):\s*(.+)"`
- **Tool Name Extraction:** First capture group
- **Parameter Extraction:** Second capture group
- **Function Mapping:** Dynamic tool dictionary lookup
- **Result Formatting:** "Observation: {result}"

---

## 📋 Key Technical Insights

### **✅ Planning Pattern Benefits**
1. **Strategic Thinking:** Forces comprehensive problem analysis
2. **Structured Execution:** Clear step-by-step roadmap
3. **Reduced Errors:** Less likely to skip critical steps
4. **Interpretable Process:** Transparent decision-making
5. **Better Resource Management:** Optimized tool usage

### **✅ Implementation Advantages**
1. **Framework Independence:** Pure Python + LLM implementation
2. **Complete Control:** Full visibility into agent mechanics
3. **Educational Value:** Understanding underlying principles
4. **Customization:** Easy modification for specific needs

### **⚠️ Production Considerations**
1. **Regex Brittleness:** Format deviations can break parsing
2. **Error Handling:** Need robust failure management
3. **Tool Validation:** Ensure all tools execute successfully
4. **Output Formatting:** Structured responses for downstream systems

### **🚀 Recommended Enhancements**
1. **Structured Prompts:** JSON outputs or function calling
2. **Tool Validation:** Retries and exception handling
3. **Guardrails:** Output formatters to constrain LLM behavior
4. **Robust Parsing:** Beyond simple regex matching

---

## 🎯 Key Takeaways

### **When to Use Planning Pattern**
- **Multi-step problems** requiring coordinated actions
- **Strategic tasks** benefiting from upfront planning  
- **Complex workflows** with clear sub-task structure
- **Situations requiring interpretability** and auditability

### **Planning vs ReAct Decision Matrix**
- **Planning:** Strategic, multi-step, structured problems
- **ReAct:** Exploratory, uncertain, adaptive problems
- **Hybrid:** Combine both patterns for complex scenarios

### **Implementation Philosophy**
The Planning pattern teaches agents to **"think before they act"** - mirroring human problem-solving approaches where we outline strategies before execution. This structured approach leads to more reliable, efficient, and interpretable AI agent behavior.

---

## 🔮 Future Applications

**Ideal Use Cases:**
- **Research Assistants:** Multi-source information synthesis
- **Web Agents:** Complex site navigation and data extraction  
- **Retrieval Agents:** Multi-hop query processing
- **Autonomous Systems:** Any task benefiting from strategic planning

The Planning pattern provides a foundation for building sophisticated AI agents that can handle complex, multi-step problems with the strategic thinking capabilities traditionally associated with human intelligence.

---

**🔗 Resources:**
- [Plan-and-Solve Research Paper](https://github.com/AGI-Edgerunners/Plan-and-Solve-Prompting)
- [Tree-of-Thought Implementation](https://github.com/princeton-nlp/tree-of-thought-llm)
- [LiteLLM Documentation](https://litellm.ai/)

**📝 Article Series:**
- ✅ Parts 1-10: Framework-based agent development
- ✅ Part 11: Planning Pattern Implementation From Scratch (This article)
- 🔜 Part 12: Multi-agent Agentic Pattern Implementation