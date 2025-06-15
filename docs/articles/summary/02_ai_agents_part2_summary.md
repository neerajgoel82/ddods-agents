# AI Agents Crash Course - Part 2: Complete Summary

**Authors:** Avi Chawla & Akshay Pachaar  
**Source:** Daily Dose of Data Science  
**Date:** February 16, 2025

## 🎯 Executive Summary

Part 2 builds upon the foundational concepts from Part 1 by focusing on advanced implementation techniques for production-ready AI agent systems. This article introduces modular crew architecture using YAML configuration, structured output generation with Pydantic, custom tool development, and practical deployment patterns that separate configuration from execution logic.

---

## 📚 Key Advances Beyond Part 1

### **1. Modular Architecture with YAML Configuration**

**Problem with Direct Python Implementation:**
- Constant code modification for agent tweaks
- Increased error risk during development-to-production transitions  
- Difficult maintenance of large-scale workflows
- Tight coupling between configuration and execution logic

**Solution: Configuration-Driven Development**
- **Separate Logic from Configuration:** Agent definitions, tasks, and workflows stored in YAML files
- **Easy Modification:** Change agent behavior without touching core execution code
- **Version Control:** Track workflow changes through YAML file versioning
- **Better Maintainability:** Clean separation of concerns for production systems

**Implementation Structure:**
```
project/
├── research_crew.py          # Main execution logic
├── config/
│   ├── agents.yaml          # Agent definitions
│   └── tasks.yaml           # Task specifications
├── .env                     # Environment variables
└── notebook.ipynb          # Usage examples
```

### **2. YAML-Based Agent and Task Definition**

**Agent Configuration (`config/agents.yaml`):**
```yaml
research_agent:
  role: "Internet Researcher"
  goal: "Find the most relevant and up-to-date information on a given topic"
  backstory: "You are a skilled researcher with expertise in retrieving credible, real-time information from online sources"
  verbose: true

summarization_agent:
  role: "Content Summarizer"
  goal: "Condense research findings into an easy-to-read summary"
  backstory: "You are an expert in breaking down complex information into clear, structured insights"
  verbose: true

fact_checker_agent:
  role: "Fact-Checking Specialist"
  goal: "Verify research findings and ensure factual accuracy"
  backstory: "You specialize in detecting misinformation and validating claims using credible sources"
  verbose: true
```

**Task Configuration (`config/tasks.yaml`):**
```yaml
research_task:
  description: "Use the SerperDevTool to find the most relevant and recent data on {topic}"
  agent: research_agent
  expected_output: "A detailed research report with key insights and source references"
  verbose: true

summarization_task:
  description: "Summarize the research findings into a well-structured, concise report"
  agent: summarization_agent
  depends_on: research_task
  expected_output: "A summary highlighting the key takeaways from the research"
  verbose: true

fact_checking_task:
  description: "Cross-check the summarized information for accuracy and remove any misleading claims"
  agent: fact_checker_agent
  depends_on: summarization_task
  expected_output: "A fact-checked and verified research summary"
  verbose: true
```

### **3. CrewBase Decorator for Modular Crews**

**Implementation Pattern:**
```python
from crewai.project import CrewBase, agent, task, crew
from crewai_tools import SerperDevTool

@CrewBase
class ResearchCrew:
    """A crew for conducting research, summarizing findings, and fact-checking"""
    
    agents_config = 'config/agents.yaml'
    tasks_config = 'config/tasks.yaml'
    
    def __init__(self):
        self.search_tool = SerperDevTool()
    
    @agent
    def research_agent(self) -> Agent:
        return Agent(
            config=self.agents_config['research_agent'],
            tools=[self.search_tool],
        )
    
    @agent
    def summarization_agent(self) -> Agent:
        return Agent(
            config=self.agents_config['summarization_agent'],
        )
    
    @agent
    def fact_checker_agent(self) -> Agent:
        return Agent(
            config=self.agents_config['fact_checker_agent'],
            tools=[self.search_tool],
        )
    
    @task
    def research_task(self) -> Task:
        return Task(
            config=self.tasks_config['research_task'],
            tools=[self.search_tool],
        )
    
    @task
    def summarization_task(self) -> Task:
        return Task(
            config=self.tasks_config['summarization_task']
        )
    
    @task
    def fact_checking_task(self) -> Task:
        return Task(
            config=self.tasks_config['fact_checking_task'],
            tools=[self.search_tool]
        )
    
    @crew
    def crew(self) -> Crew:
        return Crew(
            agents=self.agents,
            tasks=self.tasks,
            process=Process.sequential,
        )
```

---

## 🔧 Structured Output Generation

### **The Problem with Raw Text Responses**
- **Inconsistent Formatting:** LLMs generate freeform text that varies between runs
- **Manual Processing Required:** Downstream applications need additional parsing
- **No Automation Compatibility:** Difficult to integrate with APIs or databases expecting structured data

### **Pydantic-Based Solution**

**Step 1: Define Output Schema**
```python
from pydantic import BaseModel, Field

class EntityRelationEntity(BaseModel):
    entity1: str = Field(description="The first entity in the triplet")
    relation: str = Field(description="The relation between the first and second entity")
    entity2: str = Field(description="The second entity in the triplet")
```

**Step 2: Configure Agent with Output Schema**
```python
agent = Agent(
    role="Senior Linguist",
    goal="Analyse the query and extract entity-relation-entity triplets",
    backstory="You are a senior linguist that is known for your analytical skills.",
    verbose=True
)
```

**Step 3: Define Task with Structured Output**
```python
task = Task(
    description="""Analyse the query and return structured JSON output in the form of
    - entity
    - relation  
    - entity
    
    The query is: {query}""",
    expected_output="A structured JSON object with the entity-relation-entity triplets",
    output_pydantic=EntityRelationEntity,
    verbose=True,
    agent=agent
)
```

**Results:**
- **Input:** "Paris is the capital of France. The Eiffel Tower is in Paris."
- **Output:** 
```json
{
  "entity1": "Paris",
  "relation": "is the capital of", 
  "entity2": "France"
}
```

**Benefits:**
- **Guaranteed Structure:** Output always follows the defined schema
- **Type Safety:** Automatic validation of data types
- **Integration Ready:** Direct compatibility with APIs and databases
- **Consistent Results:** Eliminates parsing inconsistencies

---

## 🛠️ Custom Tool Development

### **Why Custom Tools Matter**
While CrewAI provides 12+ built-in tools (web search, file operations, code interpretation, etc.), real-world applications often require specialized functionality that integrates with:
- **External APIs:** Currency rates, weather data, stock prices
- **Internal Systems:** Company databases, proprietary calculations
- **Specialized Computations:** Domain-specific algorithms

### **Building a Currency Conversion Tool**

**Step 1: Define Input Schema**
```python
class CurrencyConverterInput(BaseModel):
    """Input schema for CurrencyConverterTool"""
    amount: float = Field(..., description="The amount to convert")
    from_currency: str = Field(..., description="The source currency code (e.g., 'USD')")
    to_currency: str = Field(..., description="The target currency code (e.g., 'EUR')")
```

**Step 2: Implement Tool Class**
```python
class CurrencyConverterTool(BaseTool):
    name: str = "Currency Converter Tool"
    description: str = "Converts an amount from one currency to another"
    args_schema: Type[BaseModel] = CurrencyConverterInput
    api_key: str = os.getenv("EXCHANGE_RATE_API_KEY")
    
    def _run(self, amount: float, from_currency: str, to_currency: str) -> str:
        url = f"https://v6.exchangerate-api.com/v6/{self.api_key}/latest/{from_currency}"
        response = requests.get(url)
        
        if response.status_code != 200:
            return "Failed to fetch exchange rates."
        
        data = response.json()
        if "conversion_rates" not in data or to_currency not in data["conversion_rates"]:
            return f"Invalid currency code: {to_currency}"
        
        rate = data["conversion_rates"][to_currency]
        converted_amount = amount * rate
        return f"{amount} {from_currency} is equivalent to {converted_amount:.2f} {to_currency}"
```

**Step 3: Integration with Agent**
```python
currency_analyst = Agent(
    role="Currency Analyst",
    goal="Provide real-time currency conversions and financial insights",
    backstory="You are a finance expert with deep knowledge of global exchange rates. You help users with currency conversion and financial decision-making.",
    tools=[CurrencyConverterTool()],  # Attach custom tool
    verbose=True
)
```

**Real-World Usage:**
- **Input:** "How much is 100 dollars in euros today?"
- **Agent Processing:** Parses natural language → Structured query → API call → Financial context
- **Output:** "Converting 100 USD to EUR using real-time exchange rates results in approximately 95.40 EUR. In the financial context, it's worth noting that exchange rates can fluctuate due to various factors like economic indicators, interest rates, and geopolitical events..."

---

## 🔄 Advanced Implementation Patterns

### **Multi-Agent Query Processing Pipeline**

**Challenge:** Users provide unstructured queries like "How much is 100 dollars in euros today?" instead of structured JSON input.

**Solution:** Query Parsing Agent + Currency Analysis Agent
1. **Query Parser Agent:** Extracts relevant details (amount, source currency, target currency)
2. **Converts Natural Language:** Transforms to structured format using Pydantic
3. **Currency Analyst Agent:** Receives structured input and performs conversion
4. **Sequential Processing:** Ensures smooth execution flow

**Benefits:**
- **User-Friendly:** Accepts natural language queries
- **Structured Processing:** Internal systems use validated data structures  
- **Error Handling:** Robust parsing and validation at each step
- **Scalable:** Easy to extend with additional processing agents

### **Production Deployment Considerations**

**Environment Configuration:**
```bash
# .env file
OPENAI_API_KEY="sk-4..."
SERPER_API_KEY="42131..."
EXCHANGE_RATE_API_KEY="753..."
```

**LLM Integration Options:**
- **Local Development:** Ollama with Llama 3.2 1B model
- **Production:** OpenAI GPT-4 for reliable performance
- **Cost Optimization:** Model selection based on task complexity

**Scalability Features:**
- **Modular Design:** Easy instantiation with different configurations
- **YAML Flexibility:** Runtime configuration changes without code deployment
- **Tool Extensibility:** Add new capabilities without core system changes

---

## 📋 Key Takeaways & Best Practices

### **✅ Configuration Management**
1. **Separate Configuration from Logic:** Use YAML files for agent and task definitions
2. **Version Control Configuration:** Track changes to workflows separately from code
3. **Environment-Specific Configs:** Different settings for development, staging, production
4. **Parameter Validation:** Ensure YAML configurations are properly validated

### **✅ Structured Output Design**
1. **Always Use Pydantic:** For any output that requires downstream processing
2. **Detailed Field Descriptions:** Help LLMs understand expected output format
3. **Validation Rules:** Include constraints and data type specifications
4. **Error Handling:** Plan for cases where structured output fails

### **✅ Custom Tool Development**
1. **Clear Input Schemas:** Define precise parameter requirements
2. **Error Handling:** Robust response to API failures and invalid inputs
3. **Documentation:** Comprehensive descriptions for agent tool selection
4. **Testing:** Thorough validation of tool functionality

### **✅ Production Readiness**
1. **Modular Architecture:** Enable easy testing and deployment
2. **Configuration Flexibility:** Support multiple environments and use cases
3. **Monitoring:** Plan for logging and performance tracking
4. **Scalability:** Design for horizontal scaling and increased load

---

## 🚀 Next Steps & Advanced Topics

### **Upcoming in the Series:**
1. **Flow-Based Coordination:** Advanced multi-agent orchestration patterns
2. **Production Pipelines:** Scaling and deployment strategies  
3. **Custom Tool Ecosystems:** Building comprehensive tool libraries
4. **Agentic RAG:** Combining retrieval-augmented generation with agents
5. **Business Optimization:** Real-world applications and automation

### **Recommended Exercises:**
1. **Implement the Currency Conversion Pipeline:** Build the complete query parsing + conversion system
2. **Create Domain-Specific Tools:** Weather APIs, stock market data, document processing
3. **Design Multi-Agent Workflows:** Customer service, content creation, data analysis pipelines
4. **Production Configuration:** Set up environment-specific YAML configurations

---

## 🎉 Conclusion

Part 2 significantly advances beyond the foundational concepts of Part 1 by introducing production-ready patterns for AI agent development. The emphasis on modular architecture through YAML configuration, structured output generation, and custom tool development provides the framework for building scalable, maintainable agentic systems.

The key innovation is the separation of configuration from execution logic, enabling teams to iterate on agent behavior without code changes while maintaining the flexibility to extend capabilities through custom tools. This approach bridges the gap between experimental agent prototypes and production-ready automation systems.

**Bottom Line:** Part 2 transforms basic agent concepts into enterprise-ready implementation patterns that can scale from individual use cases to complex business automation workflows.

---

**🔗 Resources:**
- [CrewAI Documentation](https://docs.crewai.com/)
- [Daily Dose of Data Science](https://www.dailydoseofds.com/)
- [Pydantic Documentation](https://pydantic-docs.helpmanual.io/)
- [Exchange Rate API](https://exchangerate-api.com/)

**📝 Article Series:**
- ✅ Part 1: Fundamentals and Building Blocks
- ✅ Part 2: Advanced Implementation and Production Patterns (This article)
- 🔜 Part 3: Flow-Based Coordination and Advanced Orchestration
- 🔜 Part 4: Agentic RAG and Knowledge Systems