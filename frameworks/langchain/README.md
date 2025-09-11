# LangChain: Complete Guide

LangChain is the most popular framework for building AI agents and LLM-powered applications. It provides a comprehensive toolkit for creating sophisticated agents with memory, tools, and complex reasoning capabilities.

## Table of Contents

1. [Overview](#overview)
2. [Key Concepts](#key-concepts)
3. [Installation & Setup](#installation--setup)
4. [Core Components](#core-components)
5. [Agent Types](#agent-types)
6. [Tools and Toolkits](#tools-and-toolkits)
7. [Memory Systems](#memory-systems)
8. [Advanced Patterns](#advanced-patterns)
9. [Best Practices](#best-practices)
10. [Examples](#examples)

## Overview

LangChain enables developers to build applications that:
- Connect LLMs to external data sources
- Enable LLMs to interact with their environment through tools
- Create conversational agents with memory
- Build complex reasoning and planning systems
- Chain together multiple LLM calls for sophisticated workflows

### Key Features
- **Modular Architecture**: Mix and match components as needed
- **Extensive Integrations**: 100+ integrations with external services
- **Multiple Agent Types**: From simple to sophisticated reasoning agents
- **Memory Management**: Various memory types for different use cases
- **Production Ready**: Built for real-world applications

## Key Concepts

### Chains
Sequences of calls to LLMs or other utilities, enabling complex workflows.

```python
from langchain.chains import LLMChain
from langchain.llms import OpenAI
from langchain.prompts import PromptTemplate

llm = OpenAI()
prompt = PromptTemplate(
    input_variables=["topic"],
    template="Write a brief summary about {topic}"
)
chain = LLMChain(llm=llm, prompt=prompt)
result = chain.run("artificial intelligence")
```

### Agents
Systems that use LLMs to decide which actions to take and in what order.

```python
from langchain.agents import initialize_agent, AgentType
from langchain.llms import OpenAI
from langchain.tools import DuckDuckGoSearchRun

llm = OpenAI()
search = DuckDuckGoSearchRun()
tools = [search]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION
)
```

### Tools
Functions that agents can use to interact with the outside world.

### Memory
Systems to store and retrieve information across interactions.

## Installation & Setup

### Basic Installation
```bash
pip install langchain openai
```

### With Optional Dependencies
```bash
# For additional integrations
pip install langchain[all]

# Specific integrations
pip install langchain-community
pip install langchain-experimental
```

### Environment Setup
```python
import os
from dotenv import load_dotenv

load_dotenv()

# Set your API keys
os.environ["OPENAI_API_KEY"] = "your-key-here"
os.environ["SERPAPI_API_KEY"] = "your-serpapi-key"  # For search
```

## Core Components

### 1. Language Models (LLMs)

#### OpenAI Integration
```python
from langchain.llms import OpenAI
from langchain.chat_models import ChatOpenAI

# Standard LLM
llm = OpenAI(temperature=0.7, max_tokens=1000)

# Chat model (recommended for agents)
chat_llm = ChatOpenAI(
    model="gpt-3.5-turbo",
    temperature=0.7,
    max_tokens=1000
)
```

#### Other LLM Providers
```python
# Anthropic Claude
from langchain.llms import Anthropic
claude = Anthropic(model="claude-v1")

# Hugging Face
from langchain.llms import HuggingFacePipeline
local_llm = HuggingFacePipeline.from_model_id(
    model_id="gpt2",
    task="text-generation"
)
```

### 2. Prompts and Templates

```python
from langchain.prompts import PromptTemplate, ChatPromptTemplate
from langchain.prompts.chat import SystemMessagePromptTemplate, HumanMessagePromptTemplate

# Simple prompt template
prompt = PromptTemplate(
    input_variables=["task", "context"],
    template="Given this context: {context}\\n\\nPerform this task: {task}"
)

# Chat prompt template
system_template = "You are a helpful AI assistant specialized in {domain}."
system_message_prompt = SystemMessagePromptTemplate.from_template(system_template)

human_template = "{user_input}"
human_message_prompt = HumanMessagePromptTemplate.from_template(human_template)

chat_prompt = ChatPromptTemplate.from_messages([
    system_message_prompt,
    human_message_prompt
])
```

### 3. Output Parsers

```python
from langchain.schema import OutputParserException
from langchain.output_parsers import PydanticOutputParser
from pydantic import BaseModel, Field

class TaskPlan(BaseModel):
    steps: list[str] = Field(description="List of steps to complete the task")
    priority: str = Field(description="Priority level: high, medium, low")
    estimated_time: int = Field(description="Estimated completion time in minutes")

parser = PydanticOutputParser(pydantic_object=TaskPlan)
format_instructions = parser.get_format_instructions()
```

## Agent Types

### 1. Zero-Shot React Agent
Best for general-purpose tasks with access to tools.

```python
from langchain.agents import initialize_agent, AgentType
from langchain.llms import OpenAI
from langchain.tools import DuckDuckGoSearchRun, Tool

llm = OpenAI(temperature=0)
search = DuckDuckGoSearchRun()

tools = [
    Tool(
        name="Search",
        func=search.run,
        description="Search for current information on the internet"
    )
]

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

result = agent.run("What's the latest news about AI developments?")
```

### 2. Conversational React Agent
Includes memory for multi-turn conversations.

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)
```

### 3. Plan-and-Execute Agent
Plans the entire task upfront, then executes step by step.

```python
from langchain_experimental.plan_and_execute import PlanAndExecute, load_agent_executor, load_chat_planner

planner = load_chat_planner(llm)
executor = load_agent_executor(llm, tools, verbose=True)

agent = PlanAndExecute(planner=planner, executor=executor, verbose=True)
```

### 4. Custom Agent with AgentExecutor

```python
from langchain.agents import AgentExecutor, BaseMultiActionAgent
from typing import List, Union
from langchain.schema import AgentAction, AgentFinish

class CustomAgent(BaseMultiActionAgent):
    tools: List[Tool]
    llm: OpenAI

    def plan(self, intermediate_steps, **kwargs) -> Union[List[AgentAction], AgentFinish]:
        # Custom planning logic
        return AgentFinish(
            return_values={"output": "Custom agent response"},
            log="Custom agent completed"
        )

custom_agent = CustomAgent(tools=tools, llm=llm)
agent_executor = AgentExecutor.from_agent_and_tools(
    agent=custom_agent,
    tools=tools,
    verbose=True
)
```

## Tools and Toolkits

### Built-in Tools

```python
from langchain.tools import (
    DuckDuckGoSearchRun,
    ShellTool,
    PythonREPLTool,
    WikipediaQueryRun,
    ArxivQueryRun
)

# Search tools
search = DuckDuckGoSearchRun()
wikipedia = WikipediaQueryRun()
arxiv = ArxivQueryRun()

# Execution tools
shell = ShellTool()
python_repl = PythonREPLTool()

tools = [search, wikipedia, arxiv, python_repl]
```

### Custom Tools

```python
from langchain.tools import BaseTool
from typing import Optional

class WeatherTool(BaseTool):
    name = "get_weather"
    description = "Get current weather for a location"

    def _run(self, location: str) -> str:
        # Implement weather API call
        return f"Weather in {location}: Sunny, 72°F"

    async def _arun(self, location: str) -> str:
        # Async implementation
        return self._run(location)

weather_tool = WeatherTool()
```

### Function-based Tools

```python
from langchain.tools import tool

@tool
def calculate_tip(bill_amount: float, tip_percentage: float) -> str:
    \"\"\"Calculate tip amount and total bill.
    
    Args:
        bill_amount: The original bill amount
        tip_percentage: Tip percentage (e.g., 20 for 20%)
    \"\"\"
    tip = bill_amount * (tip_percentage / 100)
    total = bill_amount + tip
    return f"Tip: ${tip:.2f}, Total: ${total:.2f}"

tools = [calculate_tip]
```

### Toolkits

```python
from langchain.agents.agent_toolkits import (
    FileManagementToolkit,
    SQLDatabaseToolkit,
    VectorStoreToolkit,
    GitHubToolkit
)

# File management
file_toolkit = FileManagementToolkit(root_dir="./sandbox")

# Database toolkit
from langchain.sql_database import SQLDatabase
db = SQLDatabase.from_uri("sqlite:///example.db")
sql_toolkit = SQLDatabaseToolkit(db=db, llm=llm)

tools = file_toolkit.get_tools() + sql_toolkit.get_tools()
```

## Memory Systems

### 1. Conversation Buffer Memory

```python
from langchain.memory import ConversationBufferMemory

memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True,
    input_key="input",
    output_key="output"
)

# Use with agent
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory
)
```

### 2. Conversation Buffer Window Memory

```python
from langchain.memory import ConversationBufferWindowMemory

# Keep only last 5 interactions
memory = ConversationBufferWindowMemory(
    k=5,
    memory_key="chat_history",
    return_messages=True
)
```

### 3. Conversation Summary Memory

```python
from langchain.memory import ConversationSummaryMemory

memory = ConversationSummaryMemory(
    llm=llm,
    memory_key="chat_history",
    return_messages=True
)
```

### 4. Vector Store Memory

```python
from langchain.memory import VectorStoreRetrieverMemory
from langchain.vectorstores import FAISS
from langchain.embeddings.openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_texts([], embeddings)
retriever = vectorstore.as_retriever(search_kwargs=dict(k=3))

memory = VectorStoreRetrieverMemory(
    retriever=retriever,
    memory_key="chat_history"
)
```

### 5. Custom Memory

```python
from langchain.memory.chat_memory import BaseChatMemory
from langchain.schema import BaseMessage

class CustomMemory(BaseChatMemory):
    def clear(self):
        # Custom clear logic
        pass
    
    @property
    def memory_variables(self):
        return ["chat_history"]
    
    def load_memory_variables(self, inputs):
        # Load memory logic
        return {"chat_history": []}
    
    def save_context(self, inputs, outputs):
        # Save context logic
        pass
```

## Advanced Patterns

### 1. Multi-Agent Conversations

```python
from langchain.agents import ConversationalChatAgent
from langchain.memory import ConversationBufferMemory

# Create multiple agents with different roles
researcher_memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)
writer_memory = ConversationBufferMemory(memory_key="chat_history", return_messages=True)

researcher_agent = initialize_agent(
    tools=[search],
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=researcher_memory,
    agent_kwargs={"system_message": "You are a research specialist."}
)

writer_agent = initialize_agent(
    tools=[],
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=writer_memory,
    agent_kwargs={"system_message": "You are a content writer."}
)
```

### 2. Chain of Thought Reasoning

```python
from langchain.chains import SequentialChain
from langchain.chains.llm import LLMChain

# Step 1: Analysis
analysis_prompt = PromptTemplate(
    input_variables=["problem"],
    template="Analyze this problem step by step: {problem}"
)
analysis_chain = LLMChain(llm=llm, prompt=analysis_prompt, output_key="analysis")

# Step 2: Solution
solution_prompt = PromptTemplate(
    input_variables=["analysis"],
    template="Based on this analysis: {analysis}\\n\\nProvide a detailed solution:"
)
solution_chain = LLMChain(llm=llm, prompt=solution_prompt, output_key="solution")

# Chain them together
overall_chain = SequentialChain(
    chains=[analysis_chain, solution_chain],
    input_variables=["problem"],
    output_variables=["analysis", "solution"]
)
```

### 3. Retrieval-Augmented Generation (RAG)

```python
from langchain.vectorstores import Chroma
from langchain.embeddings import OpenAIEmbeddings
from langchain.chains import RetrievalQA
from langchain.document_loaders import TextLoader
from langchain.text_splitter import CharacterTextSplitter

# Load and split documents
loader = TextLoader("knowledge_base.txt")
documents = loader.load()
text_splitter = CharacterTextSplitter(chunk_size=1000, chunk_overlap=0)
texts = text_splitter.split_documents(documents)

# Create vector store
embeddings = OpenAIEmbeddings()
vectorstore = Chroma.from_documents(texts, embeddings)

# Create QA chain
qa = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever()
)

# Create agent with RAG tool
rag_tool = Tool(
    name="Knowledge Base",
    func=qa.run,
    description="Search the knowledge base for relevant information"
)

agent = initialize_agent([rag_tool] + tools, llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)
```

## Best Practices

### 1. Error Handling

```python
from langchain.schema import OutputParserException
import logging

def safe_agent_run(agent, input_text, max_retries=3):
    for attempt in range(max_retries):
        try:
            return agent.run(input_text)
        except OutputParserException as e:
            logging.warning(f"Parsing error on attempt {attempt + 1}: {e}")
            if attempt == max_retries - 1:
                return "I encountered an error processing your request. Please try rephrasing."
        except Exception as e:
            logging.error(f"Unexpected error: {e}")
            return "An unexpected error occurred. Please try again."
```

### 2. Cost Management

```python
from langchain.llms import OpenAI
from langchain.callbacks import get_openai_callback

# Track token usage
with get_openai_callback() as cb:
    result = agent.run("What's the weather like today?")
    print(f"Total tokens: {cb.total_tokens}")
    print(f"Total cost: ${cb.total_cost:.4f}")

# Use cheaper models when possible
cheap_llm = OpenAI(model="gpt-3.5-turbo-instruct", temperature=0)
expensive_llm = OpenAI(model="gpt-4", temperature=0)

# Use expensive model only for complex reasoning
def choose_llm(complexity_level):
    return expensive_llm if complexity_level == "high" else cheap_llm
```

### 3. Performance Optimization

```python
# Cache expensive operations
from langchain.cache import InMemoryCache
from langchain.globals import set_llm_cache

set_llm_cache(InMemoryCache())

# Batch similar requests
from langchain.llms import OpenAI
from langchain.schema import LLMResult

async def batch_process(llm, prompts):
    return await llm.agenerate([prompts])
```

### 4. Testing and Validation

```python
import pytest
from unittest.mock import Mock

def test_agent_response():
    # Mock the LLM response
    mock_llm = Mock()
    mock_llm.predict.return_value = "Expected response"
    
    agent = initialize_agent(tools=[], llm=mock_llm, agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION)
    
    result = agent.run("test input")
    assert "Expected" in result

def test_tool_execution():
    def mock_search(query):
        return f"Search results for: {query}"
    
    search_tool = Tool(name="Search", func=mock_search, description="Search tool")
    
    result = search_tool.run("test query")
    assert "test query" in result
```

## Examples

### 1. Customer Service Agent

```python
class CustomerServiceAgent:
    def __init__(self, api_key):
        self.llm = ChatOpenAI(api_key=api_key, temperature=0.3)
        self.memory = ConversationBufferMemory(
            memory_key="chat_history", 
            return_messages=True
        )
        
        # Customer service specific tools
        self.tools = [
            Tool(
                name="Order Lookup",
                func=self.lookup_order,
                description="Look up customer order by order ID"
            ),
            Tool(
                name="Refund Process",
                func=self.process_refund,
                description="Process refund for an order"
            )
        ]
        
        self.agent = initialize_agent(
            tools=self.tools,
            llm=self.llm,
            agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
            memory=self.memory,
            verbose=True
        )
    
    def lookup_order(self, order_id: str) -> str:
        # Mock order lookup
        return f"Order {order_id}: Status - Shipped, Expected delivery: Tomorrow"
    
    def process_refund(self, order_id: str) -> str:
        # Mock refund processing
        return f"Refund processed for order {order_id}. Expect 3-5 business days."
    
    def chat(self, message: str) -> str:
        return self.agent.run(input=message)
```

### 2. Research Assistant

```python
research_tools = [
    DuckDuckGoSearchRun(),
    ArxivQueryRun(),
    WikipediaQueryRun(),
    Tool(
        name="Summarize",
        func=lambda text: f"Summary: {text[:200]}...",
        description="Summarize long text content"
    )
]

research_agent = initialize_agent(
    tools=research_tools,
    llm=ChatOpenAI(temperature=0.1),
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    max_iterations=5
)

# Use the research agent
result = research_agent.run("""
Research the latest developments in quantum computing. 
Find academic papers and provide a comprehensive summary 
of the current state and future prospects.
""")
```

### 3. Code Review Agent

```python
from langchain.tools import PythonREPLTool

code_review_tools = [
    PythonREPLTool(),
    Tool(
        name="Style Checker",
        func=lambda code: "Code follows PEP 8 standards" if len(code) > 0 else "No code provided",
        description="Check code style and formatting"
    ),
    Tool(
        name="Security Scanner", 
        func=lambda code: "No security issues found" if "eval" not in code else "Security warning: eval usage detected",
        description="Scan code for security vulnerabilities"
    )
]

code_review_agent = initialize_agent(
    tools=code_review_tools,
    llm=ChatOpenAI(temperature=0),
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)
```

## Resources and Next Steps

### Official Documentation
- [LangChain Documentation](https://python.langchain.com/)
- [API Reference](https://api.python.langchain.com/)
- [Community Cookbook](https://github.com/langchain-ai/langchain/tree/master/cookbook)

### Advanced Topics
- [Custom Agent Creation](./custom-agents.md)
- [Production Deployment](../../production/deployment.md)
- [Performance Optimization](../../best-practices/performance.md)

### Community
- [LangChain Discord](https://discord.gg/langchain)
- [GitHub Discussions](https://github.com/langchain-ai/langchain/discussions)
- [Twitter](https://twitter.com/LangChainAI)

---

*Ready to explore other frameworks? Check out [CrewAI](../crewai/README.md) for multi-agent systems or [AutoGen](../autogen/README.md) for conversational AI.*