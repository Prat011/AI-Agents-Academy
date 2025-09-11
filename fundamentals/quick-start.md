# Quick Start Guide

Get up and running with AI agents in under 30 minutes! This guide will take you from zero to your first working agent.

## Prerequisites

Before we begin, make sure you have:
- Python 3.8+ installed
- Basic programming knowledge
- OpenAI API key (or similar LLM API access)
- 30 minutes of time

## Step 1: Environment Setup

### Install Python Dependencies
```bash
pip install langchain openai python-dotenv
```

### Set Up Environment Variables
Create a `.env` file in your project directory:
```env
OPENAI_API_KEY=your_openai_api_key_here
```

## Step 2: Your First Agent (5 minutes)

Let's create a simple conversational agent that can answer questions:

```python
# simple_agent.py
import os
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType

# Load environment variables
load_dotenv()

# Initialize the language model
llm = OpenAI(temperature=0.7, api_key=os.getenv("OPENAI_API_KEY"))

# Create a simple agent
agent = initialize_agent(
    tools=[],  # No tools for now
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True
)

# Test the agent
if __name__ == "__main__":
    print("🤖 Simple AI Agent Started!")
    
    while True:
        user_input = input("\\nYou: ")
        if user_input.lower() in ['quit', 'exit', 'bye']:
            break
            
        response = agent.run(user_input)
        print(f"Agent: {response}")
```

### Run Your First Agent
```bash
python simple_agent.py
```

**Try asking:**
- "What is the weather like?"
- "Explain quantum computing in simple terms"
- "Help me plan a weekend trip"

## Step 3: Adding Tools (10 minutes)

Let's make our agent more powerful by adding tools:

```python
# agent_with_tools.py
import os
import requests
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType, Tool
from langchain.tools import DuckDuckGoSearchRun

load_dotenv()

# Initialize LLM
llm = OpenAI(temperature=0.7, api_key=os.getenv("OPENAI_API_KEY"))

# Create tools
search = DuckDuckGoSearchRun()

def get_weather(location):
    \"\"\"Get current weather for a location\"\"\"
    # This is a simplified example - use a real weather API
    return f"Weather in {location}: Sunny, 72°F"

def calculate(expression):
    \"\"\"Safely evaluate mathematical expressions\"\"\"
    try:
        # Only allow safe mathematical operations
        allowed_chars = set('0123456789+-*/().,= ')
        if all(c in allowed_chars for c in expression):
            result = eval(expression)
            return f"Result: {result}"
        else:
            return "Error: Invalid characters in expression"
    except:
        return "Error: Could not calculate expression"

# Define tools
tools = [
    Tool(
        name="Search",
        func=search.run,
        description="Search the internet for current information"
    ),
    Tool(
        name="Weather",
        func=get_weather,
        description="Get weather information for a specific location"
    ),
    Tool(
        name="Calculator", 
        func=calculate,
        description="Perform mathematical calculations"
    )
]

# Create agent with tools
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.ZERO_SHOT_REACT_DESCRIPTION,
    verbose=True,
    max_iterations=3
)

if __name__ == "__main__":
    print("🤖 AI Agent with Tools Started!")
    print("Available capabilities:")
    print("- Internet search")
    print("- Weather information") 
    print("- Mathematical calculations")
    
    while True:
        user_input = input("\\nYou: ")
        if user_input.lower() in ['quit', 'exit', 'bye']:
            break
            
        try:
            response = agent.run(user_input)
            print(f"\\nAgent: {response}")
        except Exception as e:
            print(f"Error: {str(e)}")
```

### Install Additional Dependencies
```bash
pip install duckduckgo-search
```

**Try asking:**
- "What's the latest news about AI?"
- "What's the weather in New York?"
- "Calculate 15% tip on $45.67"

## Step 4: Memory and Context (10 minutes)

Add memory so your agent remembers the conversation:

```python
# agent_with_memory.py
import os
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType
from langchain.memory import ConversationBufferMemory
from langchain.tools import DuckDuckGoSearchRun, Tool

load_dotenv()

# Initialize LLM
llm = OpenAI(temperature=0.7, api_key=os.getenv("OPENAI_API_KEY"))

# Create memory
memory = ConversationBufferMemory(
    memory_key="chat_history",
    return_messages=True
)

# Tools (same as before)
search = DuckDuckGoSearchRun()

tools = [
    Tool(
        name="Search",
        func=search.run,
        description="Search the internet for current information"
    )
]

# Agent with memory
agent = initialize_agent(
    tools=tools,
    llm=llm,
    agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
    memory=memory,
    verbose=True
)

if __name__ == "__main__":
    print("🤖 AI Agent with Memory Started!")
    print("I'll remember our conversation context!")
    
    while True:
        user_input = input("\\nYou: ")
        if user_input.lower() in ['quit', 'exit', 'bye']:
            break
            
        try:
            response = agent.run(input=user_input)
            print(f"\\nAgent: {response}")
        except Exception as e:
            print(f"Error: {str(e)}")
```

**Try this conversation flow:**
1. "My name is John and I live in Seattle"
2. "What's the weather like where I live?"
3. "What did I tell you my name was?"

## Step 5: Making it Production Ready (5 minutes)

Add error handling, logging, and configuration:

```python
# production_agent.py
import os
import logging
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType
from langchain.memory import ConversationBufferMemory
from langchain.tools import DuckDuckGoSearchRun, Tool

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class AIAgent:
    def __init__(self, api_key=None, temperature=0.7):
        load_dotenv()
        
        self.api_key = api_key or os.getenv("OPENAI_API_KEY")
        if not self.api_key:
            raise ValueError("OpenAI API key is required")
            
        # Initialize components
        self.llm = OpenAI(temperature=temperature, api_key=self.api_key)
        self.memory = ConversationBufferMemory(
            memory_key="chat_history",
            return_messages=True
        )
        
        # Setup tools
        self.tools = self._create_tools()
        
        # Create agent
        self.agent = initialize_agent(
            tools=self.tools,
            llm=self.llm,
            agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
            memory=self.memory,
            verbose=True,
            max_iterations=3,
            early_stopping_method="generate"
        )
        
        logger.info("AI Agent initialized successfully")
    
    def _create_tools(self):
        \"\"\"Create and return list of tools\"\"\"
        search = DuckDuckGoSearchRun()
        
        return [
            Tool(
                name="Search",
                func=search.run,
                description="Search the internet for current information"
            )
        ]
    
    def chat(self, message):
        \"\"\"Send message to agent and get response\"\"\"
        try:
            logger.info(f"User message: {message}")
            response = self.agent.run(input=message)
            logger.info(f"Agent response: {response}")
            return response
        except Exception as e:
            error_msg = f"Error processing message: {str(e)}"
            logger.error(error_msg)
            return "I'm sorry, I encountered an error processing your request."
    
    def reset_memory(self):
        \"\"\"Clear conversation memory\"\"\"
        self.memory.clear()
        logger.info("Memory cleared")

def main():
    try:
        agent = AIAgent()
        print("🤖 Production AI Agent Started!")
        print("Type 'quit' to exit, 'reset' to clear memory")
        
        while True:
            user_input = input("\\nYou: ").strip()
            
            if user_input.lower() in ['quit', 'exit', 'bye']:
                break
            elif user_input.lower() == 'reset':
                agent.reset_memory()
                print("Memory cleared!")
                continue
            elif not user_input:
                continue
                
            response = agent.chat(user_input)
            print(f"\\nAgent: {response}")
            
    except KeyboardInterrupt:
        print("\\n\\nGoodbye!")
    except Exception as e:
        print(f"Fatal error: {e}")

if __name__ == "__main__":
    main()
```

## Next Steps

Congratulations! You've built your first AI agent. Here's what to explore next:

### Immediate Next Steps
1. **[Framework Deep Dive](../frameworks/langchain/README.md)** - Learn LangChain in detail
2. **[Tool Integration Tutorial](../tutorials/tool-integration.md)** - Connect to more external services
3. **[Memory Systems](../advanced/memory-systems.md)** - Advanced memory patterns

### Popular Extensions
- **Web Interface**: Add Flask/FastAPI web interface
- **Database Integration**: Store conversations in database
- **Multi-modal**: Add image and voice capabilities
- **Custom Tools**: Build domain-specific tools

### Production Considerations
- **Error Handling**: Robust error recovery
- **Rate Limiting**: Manage API costs
- **Monitoring**: Track performance and usage
- **Security**: Validate inputs and outputs

## Common Issues & Solutions

### "Invalid API Key" Error
- Check your `.env` file setup
- Verify API key is correct and active
- Ensure you have credits in your OpenAI account

### "Module Not Found" Errors
- Install missing packages: `pip install package-name`
- Check your virtual environment is activated
- Verify Python version compatibility

### Agent Not Using Tools
- Check tool descriptions are clear
- Verify tool functions work independently
- Try simpler prompts first

### High API Costs
- Set temperature lower (0.1-0.3)
- Use `max_iterations` to limit loops
- Implement caching for repeated queries

## Key Concepts Learned

✅ **Agent Initialization** - Setting up LLM and configuration  
✅ **Tool Integration** - Adding external capabilities  
✅ **Memory Management** - Maintaining conversation context  
✅ **Error Handling** - Building robust applications  
✅ **Production Patterns** - Structuring for real-world use  

## Resources

- **[LangChain Documentation](https://langchain.readthedocs.io/)**
- **[OpenAI API Reference](https://platform.openai.com/docs/api-reference)**
- **[Agent Examples Repository](../tutorials/)**

*Ready for more? Continue with [Agent Architecture](./agent-architecture.md) or jump into [Framework Comparisons](../frameworks/comparison.md)*