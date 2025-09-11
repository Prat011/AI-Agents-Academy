# Build Your First AI Agent

In this tutorial, you'll create a functional AI agent from scratch in under 20 minutes. We'll build a helpful assistant that can search the web, perform calculations, and maintain conversation memory.

## What You'll Build

By the end of this tutorial, you'll have an AI agent that can:
- Search the internet for current information
- Perform mathematical calculations
- Remember conversation context
- Handle errors gracefully
- Run as a command-line application

## Prerequisites

- Python 3.8+ installed on your system
- Basic familiarity with Python
- OpenAI API key (get one at [platform.openai.com](https://platform.openai.com))
- 20 minutes of your time

## Step 1: Environment Setup (3 minutes)

### Create Project Directory
```bash
mkdir my-first-agent
cd my-first-agent
```

### Install Dependencies
```bash
pip install langchain openai python-dotenv duckduckgo-search
```

### Environment Variables
Create a `.env` file in your project directory:
```env
OPENAI_API_KEY=your_actual_openai_api_key_here
```

**Important**: Replace `your_actual_openai_api_key_here` with your real OpenAI API key!

## Step 2: Basic Agent Structure (5 minutes)

Create `agent.py` and add the foundation:

```python
import os
import logging
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType
from langchain.memory import ConversationBufferMemory

# Load environment variables
load_dotenv()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class MyFirstAgent:
    def __init__(self):
        # Validate API key
        self.api_key = os.getenv("OPENAI_API_KEY")
        if not self.api_key:
            raise ValueError("OPENAI_API_KEY not found in environment variables")
        
        # Initialize language model
        self.llm = OpenAI(
            temperature=0.7,  # Controls creativity (0.0 = deterministic, 1.0 = very creative)
            api_key=self.api_key
        )
        
        print("✅ Language model initialized")
        
        # We'll add tools and memory in the next steps
        self.tools = []
        self.memory = None
        self.agent = None
    
    def setup_tools(self):
        \"\"\"Add capabilities to the agent\"\"\"
        # We'll implement this in Step 3
        pass
    
    def setup_memory(self):
        \"\"\"Enable conversation memory\"\"\"
        # We'll implement this in Step 4
        pass
    
    def create_agent(self):
        \"\"\"Create the agent with tools and memory\"\"\"
        # We'll implement this in Step 5
        pass
    
    def chat(self, message):
        \"\"\"Send a message to the agent\"\"\"
        # We'll implement this in Step 6
        pass

# Test basic setup
if __name__ == "__main__":
    try:
        agent = MyFirstAgent()
        print("🎉 Agent created successfully!")
    except Exception as e:
        print(f"❌ Error: {e}")
```

### Test Your Setup
```bash
python agent.py
```

You should see:
```
✅ Language model initialized
🎉 Agent created successfully!
```

If you see an error, check your API key and internet connection.

## Step 3: Add Tools (4 minutes)

Now let's give your agent some capabilities. Add these methods to your `MyFirstAgent` class:

```python
from langchain.tools import DuckDuckGoSearchRun, Tool

def setup_tools(self):
    \"\"\"Add capabilities to the agent\"\"\"
    
    # 1. Internet Search Tool
    search = DuckDuckGoSearchRun()
    search_tool = Tool(
        name="Search",
        func=search.run,
        description="Search the internet for current information. Use this when you need up-to-date facts, news, or information not in your training data."
    )
    
    # 2. Calculator Tool
    def calculate(expression):
        \"\"\"Safely evaluate mathematical expressions\"\"\"
        try:
            # Only allow safe mathematical characters
            allowed_chars = set('0123456789+-*/().,e ')
            if all(c in allowed_chars for c in expression.replace(' ', '')):
                result = eval(expression)
                return f"Result: {result}"
            else:
                return "Error: Invalid characters in expression. Only numbers and basic math operators (+, -, *, /, parentheses) are allowed."
        except ZeroDivisionError:
            return "Error: Division by zero"
        except Exception as e:
            return f"Error: Could not calculate '{expression}'. Please check the syntax."
    
    calculator_tool = Tool(
        name="Calculator",
        func=calculate,
        description="Perform mathematical calculations. Input should be a mathematical expression like '2+2' or '10*5/2'."
    )
    
    # 3. Text Analysis Tool (bonus)
    def analyze_text(text):
        \"\"\"Analyze text for basic statistics\"\"\"
        words = text.split()
        chars = len(text)
        sentences = text.count('.') + text.count('!') + text.count('?')
        return f"Text Analysis:\\n- Characters: {chars}\\n- Words: {len(words)}\\n- Sentences: {sentences}"
    
    text_tool = Tool(
        name="TextAnalyzer",
        func=analyze_text,
        description="Analyze text to count words, characters, and sentences. Input should be the text to analyze."
    )
    
    # Store tools
    self.tools = [search_tool, calculator_tool, text_tool]
    
    print(f"🔧 Added {len(self.tools)} tools to agent")
```

## Step 4: Add Memory (3 minutes)

Enable your agent to remember conversations:

```python
def setup_memory(self):
    \"\"\"Enable conversation memory\"\"\"
    self.memory = ConversationBufferMemory(
        memory_key="chat_history",
        return_messages=True,
        input_key="input",
        output_key="output"
    )
    print("🧠 Memory system enabled")
```

## Step 5: Create the Agent (3 minutes)

Bring it all together:

```python
def create_agent(self):
    \"\"\"Create the agent with tools and memory\"\"\"
    if not self.tools:
        self.setup_tools()
    
    if not self.memory:
        self.setup_memory()
    
    self.agent = initialize_agent(
        tools=self.tools,
        llm=self.llm,
        agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
        memory=self.memory,
        verbose=True,  # Shows agent's thinking process
        max_iterations=3,  # Prevents infinite loops
        early_stopping_method="generate"
    )
    
    print("🤖 Agent ready to chat!")
```

## Step 6: Add Chat Interface (2 minutes)

Make your agent interactive:

```python
def chat(self, message):
    \"\"\"Send a message to the agent\"\"\"
    if not self.agent:
        self.create_agent()
    
    try:
        response = self.agent.run(input=message)
        return response
    except Exception as e:
        logger.error(f"Error processing message: {e}")
        return "I encountered an error processing your request. Could you try rephrasing it?"

def run_chat(self):
    \"\"\"Start interactive chat session\"\"\"
    print("\\n🤖 AI Agent Started!")
    print("Commands: 'quit' to exit, 'reset' to clear memory, 'help' for tips")
    print("-" * 50)
    
    if not self.agent:
        self.create_agent()
    
    while True:
        try:
            user_input = input("\\nYou: ").strip()
            
            # Handle special commands
            if user_input.lower() in ['quit', 'exit', 'bye']:
                print("👋 Goodbye!")
                break
            elif user_input.lower() == 'reset':
                self.memory.clear()
                print("🧹 Memory cleared!")
                continue
            elif user_input.lower() == 'help':
                self.show_help()
                continue
            elif not user_input:
                print("Please enter a message or 'quit' to exit.")
                continue
            
            # Process user input
            print("\\n🤖 Agent:", end=" ")
            response = self.chat(user_input)
            print(response)
            
        except KeyboardInterrupt:
            print("\\n\\n👋 Goodbye!")
            break
        except Exception as e:
            print(f"\\n❌ Unexpected error: {e}")

def show_help(self):
    \"\"\"Show available capabilities\"\"\"
    print("\\n🔧 What I can do:")
    print("• Search the internet for current information")
    print("• Perform mathematical calculations")
    print("• Analyze text (count words, characters, sentences)")
    print("• Remember our conversation")
    print("\\n💡 Try asking:")
    print("• 'What's the weather like in New York today?'")
    print("• 'Calculate 15% tip on $45.67'")
    print("• 'Analyze this text: Hello world, how are you?'")
    print("• 'What did we talk about earlier?'")
```

## Complete Code

Here's your complete `agent.py` file:

```python
import os
import logging
from dotenv import load_dotenv
from langchain.llms import OpenAI
from langchain.agents import initialize_agent, AgentType, Tool
from langchain.memory import ConversationBufferMemory
from langchain.tools import DuckDuckGoSearchRun

# Load environment variables
load_dotenv()

# Configure logging
logging.basicConfig(level=logging.INFO)
logger = logging.getLogger(__name__)

class MyFirstAgent:
    def __init__(self):
        # Validate API key
        self.api_key = os.getenv("OPENAI_API_KEY")
        if not self.api_key:
            raise ValueError("OPENAI_API_KEY not found in environment variables")
        
        # Initialize language model
        self.llm = OpenAI(
            temperature=0.7,
            api_key=self.api_key
        )
        
        print("✅ Language model initialized")
        
        # Initialize components
        self.tools = []
        self.memory = None
        self.agent = None
    
    def setup_tools(self):
        \"\"\"Add capabilities to the agent\"\"\"
        
        # 1. Internet Search Tool
        search = DuckDuckGoSearchRun()
        search_tool = Tool(
            name="Search",
            func=search.run,
            description="Search the internet for current information. Use this when you need up-to-date facts, news, or information not in your training data."
        )
        
        # 2. Calculator Tool
        def calculate(expression):
            \"\"\"Safely evaluate mathematical expressions\"\"\"
            try:
                # Only allow safe mathematical characters
                allowed_chars = set('0123456789+-*/().,e ')
                if all(c in allowed_chars for c in expression.replace(' ', '')):
                    result = eval(expression)
                    return f"Result: {result}"
                else:
                    return "Error: Invalid characters in expression. Only numbers and basic math operators (+, -, *, /, parentheses) are allowed."
            except ZeroDivisionError:
                return "Error: Division by zero"
            except Exception as e:
                return f"Error: Could not calculate '{expression}'. Please check the syntax."
        
        calculator_tool = Tool(
            name="Calculator",
            func=calculate,
            description="Perform mathematical calculations. Input should be a mathematical expression like '2+2' or '10*5/2'."
        )
        
        # 3. Text Analysis Tool
        def analyze_text(text):
            \"\"\"Analyze text for basic statistics\"\"\"
            words = text.split()
            chars = len(text)
            sentences = text.count('.') + text.count('!') + text.count('?')
            return f"Text Analysis:\\n- Characters: {chars}\\n- Words: {len(words)}\\n- Sentences: {sentences}"
        
        text_tool = Tool(
            name="TextAnalyzer",
            func=analyze_text,
            description="Analyze text to count words, characters, and sentences. Input should be the text to analyze."
        )
        
        # Store tools
        self.tools = [search_tool, calculator_tool, text_tool]
        print(f"🔧 Added {len(self.tools)} tools to agent")
    
    def setup_memory(self):
        \"\"\"Enable conversation memory\"\"\"
        self.memory = ConversationBufferMemory(
            memory_key="chat_history",
            return_messages=True,
            input_key="input",
            output_key="output"
        )
        print("🧠 Memory system enabled")
    
    def create_agent(self):
        \"\"\"Create the agent with tools and memory\"\"\"
        if not self.tools:
            self.setup_tools()
        
        if not self.memory:
            self.setup_memory()
        
        self.agent = initialize_agent(
            tools=self.tools,
            llm=self.llm,
            agent=AgentType.CONVERSATIONAL_REACT_DESCRIPTION,
            memory=self.memory,
            verbose=True,
            max_iterations=3,
            early_stopping_method="generate"
        )
        
        print("🤖 Agent ready to chat!")
    
    def chat(self, message):
        \"\"\"Send a message to the agent\"\"\"
        if not self.agent:
            self.create_agent()
        
        try:
            response = self.agent.run(input=message)
            return response
        except Exception as e:
            logger.error(f"Error processing message: {e}")
            return "I encountered an error processing your request. Could you try rephrasing it?"
    
    def run_chat(self):
        \"\"\"Start interactive chat session\"\"\"
        print("\\n🤖 AI Agent Started!")
        print("Commands: 'quit' to exit, 'reset' to clear memory, 'help' for tips")
        print("-" * 50)
        
        if not self.agent:
            self.create_agent()
        
        while True:
            try:
                user_input = input("\\nYou: ").strip()
                
                # Handle special commands
                if user_input.lower() in ['quit', 'exit', 'bye']:
                    print("👋 Goodbye!")
                    break
                elif user_input.lower() == 'reset':
                    self.memory.clear()
                    print("🧹 Memory cleared!")
                    continue
                elif user_input.lower() == 'help':
                    self.show_help()
                    continue
                elif not user_input:
                    print("Please enter a message or 'quit' to exit.")
                    continue
                
                # Process user input
                print("\\n🤖 Agent:", end=" ")
                response = self.chat(user_input)
                print(response)
                
            except KeyboardInterrupt:
                print("\\n\\n👋 Goodbye!")
                break
            except Exception as e:
                print(f"\\n❌ Unexpected error: {e}")
    
    def show_help(self):
        \"\"\"Show available capabilities\"\"\"
        print("\\n🔧 What I can do:")
        print("• Search the internet for current information")
        print("• Perform mathematical calculations") 
        print("• Analyze text (count words, characters, sentences)")
        print("• Remember our conversation")
        print("\\n💡 Try asking:")
        print("• 'What's the weather like in New York today?'")
        print("• 'Calculate 15% tip on $45.67'")
        print("• 'Analyze this text: Hello world, how are you?'")
        print("• 'What did we talk about earlier?'")

# Run the agent
if __name__ == "__main__":
    try:
        agent = MyFirstAgent()
        agent.run_chat()
    except Exception as e:
        print(f"❌ Failed to start agent: {e}")
```

## Step 7: Test Your Agent

Run your agent:
```bash
python agent.py
```

Try these conversations:

### Test 1: Basic Conversation
```
You: Hello! What can you help me with?
```

### Test 2: Internet Search
```
You: What's the latest news about artificial intelligence?
```

### Test 3: Math Calculations
```
You: Calculate 15% tip on a $47.50 bill
```

### Test 4: Text Analysis
```
You: Analyze this text: "The quick brown fox jumps over the lazy dog. This sentence contains every letter of the alphabet!"
```

### Test 5: Memory Test
```
You: My name is John
You: What did I tell you my name was?
```

## Understanding What Happened

### The Agent's Brain
Your agent uses the ReAct (Reasoning + Acting) pattern:

1. **Reasoning**: The agent thinks about what to do
2. **Acting**: The agent uses tools to gather information  
3. **Observing**: The agent processes the results
4. **Repeating**: Until the task is complete

### Example Agent Thinking Process
When you ask "What's 15% of $50?", the agent thinks:

```
Thought: I need to calculate 15% of $50. I should use the Calculator tool.
Action: Calculator
Action Input: 50 * 0.15
Observation: Result: 7.5
Thought: The 15% tip on $50 is $7.50. I now know the final answer.
Final Answer: 15% of $50 is $7.50.
```

## Common Issues & Solutions

### "Invalid API Key" Error
- Check your `.env` file exists and contains the correct key
- Verify your OpenAI API key is active and has credits
- Make sure there are no extra spaces in your API key

### "Module not found" Errors
```bash
# Install missing packages
pip install langchain openai python-dotenv duckduckgo-search
```

### Agent Not Using Tools
- Check your tool descriptions are clear and specific
- Try simpler queries first
- Verify your internet connection for search functionality

### High API Costs
- Lower the temperature: `temperature=0.1`
- Reduce max_iterations: `max_iterations=2`
- Use the reset command to clear memory periodically

## Next Steps

Congratulations! You've built your first AI agent. Here's what to explore next:

### Immediate Enhancements
1. **Add More Tools**: Weather API, email sending, file operations
2. **Improve Error Handling**: Better validation and recovery
3. **Add a Web Interface**: Flask or Streamlit GUI
4. **Save Conversations**: Store chat history to files

### Advanced Features
1. **Custom Memory**: Implement persistent memory with databases
2. **Multi-modal Capabilities**: Add image and voice processing
3. **Agent Specialization**: Create domain-specific versions
4. **Production Deployment**: Deploy to cloud platforms

### Learning Resources
- [Tool Integration Tutorial](./tool-integration.md) - Add external services
- [Memory Systems Guide](../advanced/memory-systems.md) - Advanced memory patterns
- [LangChain Documentation](../frameworks/langchain/README.md) - Deep dive into the framework

## Key Concepts You've Learned

✅ **Agent Architecture**: How agents combine LLMs, tools, and memory  
✅ **Tool Integration**: Adding external capabilities to agents  
✅ **Memory Management**: Maintaining conversation context  
✅ **Error Handling**: Building robust, user-friendly applications  
✅ **ReAct Pattern**: The reasoning and acting loop that powers agents  

## Challenge: Extend Your Agent

Try adding a new tool on your own:

```python
def get_current_time(timezone="UTC"):
    \"\"\"Get current time in specified timezone\"\"\"
    from datetime import datetime
    import pytz
    
    tz = pytz.timezone(timezone)
    current_time = datetime.now(tz)
    return f"Current time in {timezone}: {current_time.strftime('%Y-%m-%d %H:%M:%S %Z')}"

time_tool = Tool(
    name="CurrentTime",
    func=get_current_time,
    description="Get the current time. Input should be a timezone like 'US/Eastern' or 'UTC'."
)

# Add to your tools list
self.tools.append(time_tool)
```

*Ready for the next challenge? Try building a [Customer Support Agent](./customer-support-agent.md) or explore [CrewAI Multi-Agent Systems](../frameworks/crewai/README.md).*