# CrewAI: Multi-Agent Collaboration Framework

CrewAI is a cutting-edge framework for orchestrating role-playing, autonomous AI agents. It enables agents to work together seamlessly, tackling complex tasks through collaborative intelligence and structured teamwork.

## Table of Contents

1. [Overview](#overview)
2. [Key Concepts](#key-concepts)
3. [Installation & Setup](#installation--setup)
4. [Core Components](#core-components)
5. [Creating Your First Crew](#creating-your-first-crew)
6. [Agent Roles and Responsibilities](#agent-roles-and-responsibilities)
7. [Task Management](#task-management)
8. [Tools Integration](#tools-integration)
9. [Advanced Patterns](#advanced-patterns)
10. [Examples](#examples)

## Overview

CrewAI revolutionizes the way AI agents collaborate by:
- **Role-Based Agents**: Each agent has specific roles, goals, and backstories
- **Hierarchical Teams**: Organized crew structures with clear responsibilities  
- **Task Orchestration**: Sequential and parallel task execution
- **Memory Sharing**: Agents can share context and learnings
- **Tool Ecosystem**: Rich integration with external tools and APIs

### Why Choose CrewAI?

- **Structured Collaboration**: Unlike single agents, crews can divide complex work
- **Role Specialization**: Agents excel in their specific domains
- **Process Management**: Built-in workflows for common patterns
- **Scalability**: Easy to add new agents and capabilities
- **Production Ready**: Designed for real-world applications

## Key Concepts

### Crews
A collection of agents working together towards a common goal.

### Agents
Individual AI entities with specific roles, goals, and capabilities.

### Tasks
Discrete work items assigned to agents or crews.

### Processes
The workflow that governs how tasks are executed (sequential, hierarchical, etc.).

### Tools
External capabilities that agents can use to accomplish their tasks.

## Installation & Setup

### Basic Installation
```bash
pip install crewai
```

### With Tools
```bash
pip install 'crewai[tools]'
```

### Environment Setup
```python
import os
from dotenv import load_dotenv

load_dotenv()

# Required API keys
os.environ["OPENAI_API_KEY"] = "your-openai-api-key"
os.environ["SERPER_API_KEY"] = "your-serper-api-key"  # For search capabilities
```

## Core Components

### 1. Agent Creation

```python
from crewai import Agent
from crewai_tools import SerperDevTool

# Create search tool
search_tool = SerperDevTool()

# Define an agent
researcher = Agent(
    role="Senior Research Analyst",
    goal="Discover groundbreaking technologies and trends",
    backstory="""You are a seasoned research analyst with a keen eye for 
    emerging technologies. You have a knack for dissecting complex information 
    and presenting it in an understandable manner.""",
    tools=[search_tool],
    verbose=True,
    allow_delegation=True,
    max_iter=5,
    memory=True
)
```

### 2. Task Definition

```python
from crewai import Task

research_task = Task(
    description="""Research the latest trends in AI agent development. 
    Focus on multi-agent systems, tool integration, and real-world applications. 
    Provide a comprehensive report with key findings.""",
    expected_output="A detailed research report with trends, key players, and future predictions",
    agent=researcher,
    tools=[search_tool]
)
```

### 3. Crew Assembly

```python
from crewai import Crew, Process

crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, writing_task, editing_task],
    process=Process.sequential,
    memory=True,
    cache=True,
    max_rpm=100,
    share_crew=True
)
```

## Creating Your First Crew

Let's build a content creation crew with researcher, writer, and editor roles:

```python
from crewai import Agent, Task, Crew, Process
from crewai_tools import SerperDevTool, FileReadTool

# Initialize tools
search_tool = SerperDevTool()
file_tool = FileReadTool()

# 1. Create Agents
researcher = Agent(
    role="Content Researcher",
    goal="Gather comprehensive information on given topics",
    backstory="""You're an experienced researcher with a talent for finding 
    accurate, relevant information from multiple sources.""",
    tools=[search_tool],
    verbose=True
)

writer = Agent(
    role="Content Writer",
    goal="Create engaging, well-structured content",
    backstory="""You're a skilled writer who can transform research into 
    compelling narratives that engage readers.""",
    verbose=True
)

editor = Agent(
    role="Content Editor",
    goal="Review and improve written content for quality and clarity",
    backstory="""You're a meticulous editor with an eye for detail, 
    ensuring content is polished and professional.""",
    verbose=True
)

# 2. Define Tasks
research_task = Task(
    description="Research the topic: 'Future of AI in Healthcare'",
    expected_output="Comprehensive research findings with sources",
    agent=researcher
)

writing_task = Task(
    description="Write a 1000-word article based on the research findings",
    expected_output="Well-structured article with introduction, body, and conclusion",
    agent=writer,
    context=[research_task]  # Depends on research task output
)

editing_task = Task(
    description="Edit and improve the article for clarity and engagement",
    expected_output="Final polished article ready for publication",
    agent=editor,
    context=[writing_task]  # Depends on writing task output
)

# 3. Form the Crew
content_crew = Crew(
    agents=[researcher, writer, editor],
    tasks=[research_task, writing_task, editing_task],
    process=Process.sequential,
    memory=True,
    cache=True
)

# 4. Execute the Mission
result = content_crew.kickoff()
print(result)
```

## Agent Roles and Responsibilities

### Common Agent Archetypes

#### 1. Research Agents
```python
research_agent = Agent(
    role="Senior Research Analyst",
    goal="Conduct thorough research on assigned topics",
    backstory="""Expert researcher with 10+ years of experience in data analysis 
    and market research. Known for uncovering hidden insights.""",
    tools=[search_tool, database_tool],
    max_execution_time=300,
    max_iter=5
)
```

#### 2. Analysis Agents
```python
analyst = Agent(
    role="Data Analyst",
    goal="Analyze data and extract meaningful insights",
    backstory="""Skilled analyst who can interpret complex datasets 
    and identify trends and patterns.""",
    tools=[python_tool, sql_tool, visualization_tool],
    allow_delegation=False
)
```

#### 3. Writing Agents
```python
writer = Agent(
    role="Technical Writer",
    goal="Create clear, comprehensive documentation",
    backstory="""Technical writer with expertise in making complex 
    topics accessible to various audiences.""",
    verbose=True,
    allow_delegation=True
)
```

#### 4. Review Agents
```python
reviewer = Agent(
    role="Quality Assurance Specialist",
    goal="Ensure all outputs meet quality standards",
    backstory="""Experienced QA professional with attention to detail 
    and commitment to excellence.""",
    max_iter=3
)
```

### Agent Capabilities

#### Memory
```python
agent_with_memory = Agent(
    role="Customer Service Representative",
    goal="Provide excellent customer support",
    backstory="Experienced customer service professional",
    memory=True,  # Enables long-term memory
    verbose=True
)
```

#### Delegation
```python
manager_agent = Agent(
    role="Project Manager", 
    goal="Coordinate team activities and delegate tasks",
    backstory="Experienced project manager with leadership skills",
    allow_delegation=True,  # Can delegate to other agents
    max_delegation=2
)
```

#### Custom LLM
```python
from langchain.llms import OpenAI

custom_llm = OpenAI(temperature=0.3)

specialized_agent = Agent(
    role="Code Reviewer",
    goal="Review code for quality and security",
    backstory="Senior developer with security expertise",
    llm=custom_llm
)
```

## Task Management

### Task Types

#### 1. Simple Tasks
```python
simple_task = Task(
    description="Summarize the main points of the research document",
    expected_output="Bullet-pointed summary of key findings",
    agent=researcher
)
```

#### 2. Conditional Tasks
```python
conditional_task = Task(
    description="If the research indicates high market potential, create a business plan",
    expected_output="Business plan document or explanation why not created",
    agent=business_analyst,
    context=[research_task]
)
```

#### 3. Collaborative Tasks
```python
collaborative_task = Task(
    description="Review and improve the content collaboratively",
    expected_output="Improved content with tracked changes",
    agent=editor,
    context=[writing_task],
    collaboration=True
)
```

### Task Dependencies
```python
# Sequential dependencies
task2 = Task(
    description="Analyze the research findings",
    agent=analyst,
    context=[task1]  # Waits for task1 to complete
)

task3 = Task(
    description="Write report based on analysis",
    agent=writer,
    context=[task1, task2]  # Waits for both tasks
)
```

### Task Callbacks
```python
def on_task_complete(task_output):
    print(f"Task completed: {task_output.description}")
    # Custom logging or notifications

task_with_callback = Task(
    description="Process data analysis",
    agent=analyst,
    callback=on_task_complete
)
```

## Tools Integration

### Built-in Tools

```python
from crewai_tools import (
    SerperDevTool,
    SeleniumScrapingTool, 
    FileReadTool,
    DirectoryReadTool,
    CodeDocsSearchTool,
    CSVSearchTool
)

# Search tool
search = SerperDevTool()

# Web scraping
scraper = SeleniumScrapingTool(
    website_url="https://example.com",
    css_element=".content"
)

# File operations
file_reader = FileReadTool()
dir_reader = DirectoryReadTool(directory="./documents")

# Code analysis
code_search = CodeDocsSearchTool()

# Data analysis
csv_tool = CSVSearchTool(csv_file_path="data.csv")
```

### Custom Tools

```python
from crewai_tools import BaseTool
from typing import Type
from pydantic import BaseModel, Field

class CalculatorInput(BaseModel):
    operation: str = Field(description="Mathematical operation to perform")

class CalculatorTool(BaseTool):
    name: str = "Calculator"
    description: str = "Perform mathematical calculations"
    args_schema: Type[BaseModel] = CalculatorInput

    def _run(self, operation: str) -> str:
        try:
            result = eval(operation)
            return f"The result of {operation} is {result}"
        except Exception as e:
            return f"Error calculating {operation}: {str(e)}"

calculator = CalculatorTool()
```

### API Integration Tools

```python
import requests
from crewai_tools import BaseTool

class WeatherTool(BaseTool):
    name: str = "Weather API"
    description: str = "Get current weather information"

    def _run(self, location: str) -> str:
        api_key = os.getenv("WEATHER_API_KEY")
        url = f"http://api.openweathermap.org/data/2.5/weather?q={location}&appid={api_key}"
        
        response = requests.get(url)
        if response.status_code == 200:
            data = response.json()
            return f"Weather in {location}: {data['weather'][0]['description']}, {data['main']['temp']}°K"
        else:
            return f"Could not get weather for {location}"

weather_tool = WeatherTool()
```

## Advanced Patterns

### 1. Hierarchical Process

```python
# Manager agent delegates to subordinates
manager = Agent(
    role="Project Manager",
    goal="Oversee project completion",
    backstory="Experienced manager who delegates effectively",
    allow_delegation=True
)

crew_hierarchical = Crew(
    agents=[manager, developer, tester, writer],
    tasks=[planning_task, development_task, testing_task, documentation_task],
    process=Process.hierarchical,
    manager_llm=ChatOpenAI(model="gpt-4")  # Manager uses more powerful model
)
```

### 2. Parallel Processing

```python
# Tasks that can run simultaneously
parallel_crew = Crew(
    agents=[researcher1, researcher2, researcher3],
    tasks=[research_task1, research_task2, research_task3],
    process=Process.sequential,  # Will run in parallel where possible
    max_rpm=200,
    share_crew=True
)
```

### 3. Dynamic Agent Creation

```python
def create_specialist_agent(domain: str, tools: list) -> Agent:
    return Agent(
        role=f"{domain} Specialist",
        goal=f"Provide expert knowledge in {domain}",
        backstory=f"Expert in {domain} with years of experience",
        tools=tools,
        verbose=True
    )

# Create agents dynamically based on needs
marketing_agent = create_specialist_agent("Marketing", [search_tool, analytics_tool])
tech_agent = create_specialist_agent("Technology", [code_tool, api_tool])
```

### 4. Crew Composition

```python
class ContentCreationCrew:
    def __init__(self, topic: str):
        self.topic = topic
        self.setup_agents()
        self.setup_tasks()
        self.setup_crew()
    
    def setup_agents(self):
        self.researcher = Agent(
            role="Topic Researcher",
            goal=f"Research comprehensive information about {self.topic}",
            backstory="Expert researcher specializing in thorough analysis",
            tools=[search_tool],
            verbose=True
        )
        
        self.writer = Agent(
            role="Content Writer",
            goal="Create engaging content from research",
            backstory="Skilled writer who creates compelling narratives",
            verbose=True
        )
    
    def setup_tasks(self):
        self.research_task = Task(
            description=f"Research {self.topic} comprehensively",
            expected_output="Detailed research report with sources",
            agent=self.researcher
        )
        
        self.writing_task = Task(
            description="Create article from research",
            expected_output="Polished 1000-word article",
            agent=self.writer,
            context=[self.research_task]
        )
    
    def setup_crew(self):
        self.crew = Crew(
            agents=[self.researcher, self.writer],
            tasks=[self.research_task, self.writing_task],
            process=Process.sequential
        )
    
    def execute(self):
        return self.crew.kickoff()

# Usage
ai_crew = ContentCreationCrew("Artificial Intelligence in Healthcare")
result = ai_crew.execute()
```

## Examples

### 1. Market Research Crew

```python
# Market Research Crew
market_researcher = Agent(
    role="Market Research Analyst",
    goal="Analyze market trends and opportunities",
    backstory="Expert in market analysis with 15 years experience",
    tools=[search_tool, web_scraper],
    verbose=True
)

competitive_analyst = Agent(
    role="Competitive Intelligence Analyst", 
    goal="Analyze competitors and market positioning",
    backstory="Specialist in competitive analysis and strategic planning",
    tools=[search_tool, data_analysis_tool],
    verbose=True
)

report_writer = Agent(
    role="Business Report Writer",
    goal="Create comprehensive market reports",
    backstory="Professional business writer with MBA background",
    verbose=True
)

# Tasks
market_analysis_task = Task(
    description="Analyze the AI chatbot market size, growth, and trends",
    expected_output="Market analysis report with size, growth rate, and key trends",
    agent=market_researcher
)

competitor_analysis_task = Task(
    description="Analyze top 5 competitors in the AI chatbot space",
    expected_output="Competitive analysis with strengths, weaknesses, and positioning",
    agent=competitive_analyst
)

final_report_task = Task(
    description="Compile market and competitive analysis into executive summary",
    expected_output="Executive summary with recommendations and next steps",
    agent=report_writer,
    context=[market_analysis_task, competitor_analysis_task]
)

market_crew = Crew(
    agents=[market_researcher, competitive_analyst, report_writer],
    tasks=[market_analysis_task, competitor_analysis_task, final_report_task],
    process=Process.sequential,
    memory=True
)

market_result = market_crew.kickoff()
```

### 2. Software Development Crew

```python
from crewai_tools import CodeDocsSearchTool, FileReadTool

# Development tools
code_search = CodeDocsSearchTool()
file_reader = FileReadTool()

# Software Development Crew
product_manager = Agent(
    role="Product Manager",
    goal="Define product requirements and specifications",
    backstory="Experienced PM who bridges business and technical requirements",
    verbose=True
)

developer = Agent(
    role="Senior Developer",
    goal="Design and implement software solutions",
    backstory="Full-stack developer with expertise in modern frameworks",
    tools=[code_search, file_reader],
    verbose=True
)

tester = Agent(
    role="QA Engineer",
    goal="Ensure software quality through comprehensive testing",
    backstory="Quality assurance expert with automated testing expertise",
    tools=[code_search],
    verbose=True
)

# Development tasks
requirements_task = Task(
    description="Define requirements for a task management application",
    expected_output="Detailed product requirements document",
    agent=product_manager
)

development_task = Task(
    description="Create implementation plan and code structure",
    expected_output="Technical design document and code samples",
    agent=developer,
    context=[requirements_task]
)

testing_task = Task(
    description="Create comprehensive test plan and test cases",
    expected_output="Test plan with unit and integration test scenarios",
    agent=tester,
    context=[requirements_task, development_task]
)

dev_crew = Crew(
    agents=[product_manager, developer, tester],
    tasks=[requirements_task, development_task, testing_task],
    process=Process.sequential
)
```

### 3. Customer Support Crew

```python
# Customer Support Crew
support_agent = Agent(
    role="Customer Support Specialist",
    goal="Provide excellent customer service and resolve issues",
    backstory="Experienced support specialist with problem-solving skills",
    tools=[knowledge_base_tool, ticket_system_tool],
    verbose=True
)

technical_agent = Agent(
    role="Technical Support Engineer",
    goal="Resolve complex technical issues",
    backstory="Technical expert who can diagnose and fix complex problems",
    tools=[system_diagnostic_tool, code_search],
    verbose=True
)

escalation_manager = Agent(
    role="Support Manager",
    goal="Handle escalated issues and ensure customer satisfaction",
    backstory="Experienced manager who handles difficult situations",
    allow_delegation=True,
    verbose=True
)

# Support tasks would be dynamically created based on tickets
def create_support_task(ticket_info):
    if ticket_info['complexity'] == 'high':
        agent = technical_agent
    else:
        agent = support_agent
        
    return Task(
        description=f"Resolve customer issue: {ticket_info['description']}",
        expected_output="Resolution with steps taken and outcome",
        agent=agent
    )

support_crew = Crew(
    agents=[support_agent, technical_agent, escalation_manager],
    tasks=[],  # Tasks added dynamically
    process=Process.hierarchical,
    manager_llm=ChatOpenAI(model="gpt-4")
)
```

## Best Practices

### 1. Agent Design
- Give agents clear, specific roles and goals
- Provide detailed backstories for better role-playing
- Limit the number of tools per agent (3-5 tools max)
- Use appropriate LLM models based on task complexity

### 2. Task Management
- Write clear, specific task descriptions
- Define expected outputs precisely
- Use context dependencies wisely to avoid bottlenecks
- Set realistic execution limits

### 3. Crew Optimization
- Start with small crews (2-4 agents) and scale up
- Use hierarchical processes for complex workflows
- Enable memory for tasks requiring context
- Monitor and adjust max_rpm based on API limits

### 4. Error Handling
```python
import logging
from crewai.utilities import Logger

# Setup logging
logger = Logger()

def safe_crew_execution(crew, max_retries=3):
    for attempt in range(max_retries):
        try:
            return crew.kickoff()
        except Exception as e:
            logger.log(f"Attempt {attempt + 1} failed: {str(e)}")
            if attempt == max_retries - 1:
                raise
```

## Resources and Next Steps

### Official Resources
- [CrewAI Documentation](https://docs.crewai.com/)
- [CrewAI GitHub](https://github.com/joaomdmoura/crewAI)
- [CrewAI Tools](https://github.com/joaomdmoura/crewAI-tools)

### Community
- [CrewAI Discord](https://discord.gg/crewai)
- [CrewAI Twitter](https://twitter.com/crewAIInc)

### Advanced Learning
- [Multi-Agent Orchestration](../../advanced/multi-agent-systems.md)
- [Production Deployment](../../production/README.md)
- [Tool Development](../../tutorials/custom-tools.md)

---

*Ready to explore other frameworks? Check out [LangChain](../langchain/README.md) for comprehensive agent building or [AutoGen](../autogen/README.md) for conversational agents.*