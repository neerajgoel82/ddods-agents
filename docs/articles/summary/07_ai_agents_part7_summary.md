# AI Agents Crash Course - Part 7: Complete Summary

**A Practical Deep Dive Into Knowledge for Agentic Systems**

**Authors:** Avi Chawla & Akshay Pachaar  
**Source:** Daily Dose of Data Science  
**Date:** March 23, 2025

## 🎯 Executive Summary

Part 7 introduces **Knowledge Systems for AI Agents**, a critical component that transforms agents from prompt-bound to memory-augmented systems. This article covers the fundamental distinction between tools and knowledge, practical implementation of various knowledge sources (string, file-based, and custom), and advanced topics like embedding customization for privacy and performance optimization.

---

## 🆕 New Concepts Beyond Parts 1-6

### **Knowledge vs. Tools: The Critical Distinction**

While previous parts focused on tools and workflows, Part 7 introduces a fundamental concept that hadn't been deeply explored:

**Tools vs. Knowledge Sources:**
- **Tools** → Perform actions (web search, file reading, API calls)
- **Knowledge** → Provide reference context for reasoning (PDFs, CSVs, docs, preloaded text)

**Key Differences:**

| Aspect | Tools | Knowledge |
|--------|-------|-----------|
| **Primary Role** | Perform an action | Provide reference context for reasoning |
| **Invocation** | Explicitly called by the agent | Implicitly accessed via semantic retrieval |
| **Samples** | Web search, file reader, API calls | PDFs, CSVs, docs, preloaded text |
| **Example** | "Go fetch this from the web" | "Use what you already know to answer this question" |
| **Execution Time** | Happens live during the agent's run | Preprocessed and stored prior to task execution |

---

## 📚 Knowledge Source Implementation

### **1. String Knowledge Sources**

**Purpose:** Lightweight in-memory knowledge for small amounts of structured or unstructured information.

**Implementation:**
```python
from crewai.knowledge.source.string_knowledge_source import StringKnowledgeSource

# Define knowledge as raw string
policy_text = """
Our return policy allows customers to return any product within 30 days of purchase. 
Refunds will be issued only if the item is unused and in original packaging.
Customers must provide proof of purchase when requesting a return.
"""

# Create knowledge source
return_policy_knowledge = StringKnowledgeSource(content=policy_text)
```

**Use Cases:**
- Company policies and procedures
- FAQ content
- Product specifications
- Quick reference information

### **2. File-Based Knowledge Sources**

**Supported Formats:**
- **Text files (.txt)**
- **PDFs**
- **Markdown (.md)**
- **CSV/Excel spreadsheets**
- **JSON files**
- **Custom APIs**
- **Raw strings**

**Implementation Examples:**

**Text Files:**
```python
from crewai.knowledge.source.text_file_knowledge_source import TextFileKnowledgeSource

text_source = TextFileKnowledgeSource(
    file_paths=["hr_policy.txt"]
)
```

**PDF Files:**
```python
from crewai.knowledge.source.pdf_knowledge_source import PDFKnowledgeSource

pdf_source = PDFKnowledgeSource(
    file_paths=["meeting_notes.pdf"]
)
```

**CSV Files:**
```python
from crewai.knowledge.source.csv_knowledge_source import CSVKnowledgeSource

csv_source = CSVKnowledgeSource(
    file_paths=["feedback.csv"]
)
```

**JSON Files:**
```python
from crewai.knowledge.source.json_knowledge_source import JSONKnowledgeSource

json_source = JSONKnowledgeSource(
    file_paths=["company_info.json"]
)
```

---

## 🔧 Advanced Knowledge Features

### **Chunking Configuration**

**Purpose:** Control how large documents are broken down for optimal retrieval.

```python
source = StringKnowledgeSource(
    content="Your content here",
    chunk_size=4000,        # Maximum size of each chunk (default: 4000)
    chunk_overlap=200       # Overlap between chunks (default: 200)
)
```

**Chunking Benefits:**
- Ensures text fits the input size of embedding models
- Prevents single embedding for entire documents
- Maintains semantic continuity between chunks
- Enables more precise context retrieval

### **Knowledge Access Levels**

**Agent-Level Access:**
```python
agent = Agent(
    role="Product Launch Specialist",
    knowledge_sources=[product_knowledge],  # Only this agent has access
    ...
)
```

**Crew-Level Access:**
```python
crew = Crew(
    agents=[marketing_agent, support_agent],
    tasks=[marketing_task, support_task],
    knowledge_sources=[product_knowledge],  # All agents have access
    ...
)
```

**Design Philosophy:** Mirrors real-world information flow where some documents are confidential to specific team members, while others are shared across teams.

---

## 🛠️ Custom Knowledge Sources

### **Creating Custom Knowledge Sources**

**Use Case:** When data comes from live APIs, databases, or internal systems rather than static files.

**Example: Weather Knowledge Source**

```python
from crewai.knowledge.source.base_knowledge_source import BaseKnowledgeSource
from typing import Dict, Any
from pydantic import Field
import requests

class WeatherKnowledgeSource(BaseKnowledgeSource):
    city: str = Field(description="City for which weather should be fetched")
    
    def load_content(self) -> Dict[Any, str]:
        # Make API call to weather service
        endpoint = "https://api.open-meteo.com/v1/forecast"
        params = {
            "latitude": 37.77,    # San Francisco coordinates
            "longitude": -122.42,
            "current_weather": True
        }
        
        response = requests.get(endpoint, params=params)
        response.raise_for_status()
        weather_data = response.json().get("current_weather", {})
        
        # Format data for embedding
        formatted = self.format_weather(weather_data)
        return {self.city: formatted}
    
    def format_weather(self, data: dict) -> str:
        if not data:
            return "No weather data available."
        
        return (
            f"Current weather in {self.city}:\n"
            f"- Temperature: {data.get('temperature')}°C\n"
            f"- Wind Speed: {data.get('windspeed')} km/h\n"
            f"- Weather Code: {data.get('weathercode')}\n"
            f"- Time: {data.get('time')}"
        )
    
    def add(self) -> None:
        content = self.load_content()
        for _, text in content.items():
            chunks = self._chunk_text(text)
            self.chunks.extend(chunks)
        self._save_documents()
```

**Key Methods:**
- **`load_content()`**: Required method that returns dictionary of text content to embed
- **`add()`**: Method to handle preprocessing (chunking and saving)
- **Custom processing**: Format data into embeddable text

---

## 🔒 Embedding Customization

### **Privacy and Performance Optimization**

**Problem:** By default, CrewAI uses the same provider for both LLM generation and embeddings, but you might want different providers for:
- Privacy (local embeddings with Ollama)
- Performance (faster/cheaper embedding models)
- Domain-specific encoding

**Solution: Custom Embedding Configuration**

**Step 1: Set up local embedding model**
```bash
ollama pull nomic-embed-text
```

**Step 2: Configure Ollama embedder**
```python
from crewai import LLM

# Configure Ollama for embeddings
ollama_embedder = {
    "provider": "ollama",
    "config": {
        "model": "nomic-embed-text",
        "api_url": "http://localhost:11434"
    }
}
```

**Step 3: Use custom embedder with crew**
```python
crew = Crew(
    agents=[hr_faq_agent],
    tasks=[task],
    knowledge_sources=[faq_knowledge],
    embedder=ollama_embedder,  # Custom embedding configuration
    process=Process.sequential,
    verbose=True
)
```

**Supported Embedding Providers:**
- **openai** (OpenAI embeddings)
- **google** (Google embeddings)  
- **cohere** (Cohere embeddings)
- **huggingface** (HuggingFace models)
- **azure** (Azure OpenAI)
- **AWS bedrock** (Amazon Bedrock)
- **ollama** (Local models)

### **Benefits of Embedding Customization**
- **Privacy**: Run entirely on-premises with Ollama
- **Cost optimization**: Use cheaper embedding models while keeping premium LLMs
- **Performance**: Optimize embedding models for specific data types
- **Domain specialization**: Use embeddings trained on domain-specific data

---

## 📋 Implementation Best Practices

### **✅ Knowledge Source Selection Guidelines**
1. **String Sources**: For small, frequently-accessed policies and procedures
2. **File Sources**: For existing documents and structured data
3. **Custom Sources**: For live data from APIs, databases, or internal systems

### **✅ Chunking Strategy**
1. **Default settings work for most cases**: chunk_size=4000, chunk_overlap=200
2. **Maintain semantic overlap**: Prevent context loss between chunks
3. **Document structure awareness**: Consider natural breakpoints (paragraphs, sections)

### **✅ Access Control Design**
1. **Agent-level**: Confidential information specific to roles
2. **Crew-level**: Shared information across team members
3. **Security**: Mirror real-world information access patterns

### **✅ Embedding Optimization**
1. **Privacy requirements**: Use local embeddings (Ollama) for sensitive data
2. **Cost optimization**: Separate embedding and generation providers
3. **Performance tuning**: Choose models optimized for your data characteristics

---

## 🔍 Real-World Applications Demonstrated

### **1. HR Policy Assistant**
- **Knowledge**: Text file with company policies
- **Access**: Agent-level (HR specialist only)
- **Query**: "What is the sick leave protocol for employees?"

### **2. Meeting Notes Summarizer**
- **Knowledge**: PDF meeting notes with action items
- **Access**: Agent-level (meeting specialist)
- **Query**: "List the action items from last week's meeting"

### **3. Customer Feedback Analyzer**
- **Knowledge**: CSV file with user feedback and ratings
- **Access**: Agent-level (feedback analyst)
- **Query**: "What are the top 2 common issues users are facing?"

### **4. Company Information Expert**
- **Knowledge**: JSON file with organizational structure
- **Access**: Agent-level (org structure specialist)
- **Query**: "List the teams under the Engineering department"

### **5. Weather Information Assistant**
- **Knowledge**: Custom API integration with weather services
- **Access**: Agent-level (weather specialist)
- **Query**: "What is the current temperature and wind speed in SF?"

---

## 🎯 Key Innovations in Part 7

### **1. Knowledge vs. Tools Paradigm**
Clear distinction between active tools (perform actions) and passive knowledge (provide context), enabling more sophisticated agent architectures.

### **2. File Format Agnostic Design**
Native support for text, PDF, CSV, JSON, and Excel files out-of-the-box, eliminating custom parsing requirements.

### **3. Custom Knowledge Source Framework**
Extensible `BaseKnowledgeSource` class enabling integration with any data source (APIs, databases, internal systems).

### **4. Embedding Provider Separation**
Ability to use different providers for generation vs. embeddings, enabling privacy, cost, and performance optimization.

### **5. Flexible Access Control**
Agent-level vs. crew-level knowledge access, mirroring real-world information security patterns.

---

## 🔮 Future Implications

Part 7 establishes the foundation for:
- **Agentic RAG systems** (mentioned as upcoming topic)
- **Long-term memory systems** with cross-task context retention
- **Production-ready knowledge pipelines** that scale across organizations
- **Advanced embedding strategies** for domain-specific applications

---

## 📁 Technical Architecture

**Knowledge Processing Pipeline:**
1. **Content Ingestion** → Load from various sources (string, files, APIs)
2. **Chunking** → Break into manageable pieces with overlap
3. **Embedding** → Convert to vector representations
4. **Storage** → Store in semantic search database
5. **Retrieval** → Query during agent execution via semantic similarity

**Integration with Agents:**
- Knowledge sources attached at agent or crew level
- Automatic semantic retrieval during task execution
- No explicit tool calls required for knowledge access
- Context injection based on query similarity

---

## 🎉 Conclusion

Part 7 transforms AI agents from simple prompt-based systems into memory-augmented, context-aware intelligences. By introducing the tools vs. knowledge distinction and providing comprehensive implementation patterns for various data sources, this article enables the creation of agents that can reason over company documents, datasets, and structured memory.

The introduction of embedding customization opens the door to privacy-preserving, cost-optimized, and performance-tuned knowledge systems that can operate entirely on-premises or leverage specialized embedding models for domain-specific applications.

**Bottom Line:** Part 7 provides the knowledge foundation necessary for building intelligent, context-aware AI agents that can reason over external information sources while maintaining privacy, performance, and security requirements.

---

**🔗 Resources:**
- [CrewAI Documentation](https://docs.crewai.com/)
- [Daily Dose of Data Science](https://www.dailydoseofds.com/)
- [Open-Meteo API](https://open-meteo.com/)
- [Ollama](https://ollama.com/)

**📝 Article Series:**
- ✅ Part 1: Fundamentals and Building Blocks
- ✅ Part 2: Advanced Implementation and Production Patterns  
- ✅ Part 3: Building Flows in Agentic Systems (Part A)
- ✅ Part 4: Building Flows in Agentic Systems (Part B) - Multi-Crew Workflows
- ✅ Part 5: Advanced Techniques to Build Robust Agentic Systems (Part A)
- ✅ Part 6: Advanced Techniques to Build Robust Agentic Systems (Part B)
- ✅ Part 7: A Practical Deep Dive Into Knowledge for Agentic Systems (This article)
- 🔜 Part 8: A Practical Deep Dive Into Memory for Agentic Systems