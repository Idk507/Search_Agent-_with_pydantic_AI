# Research Agent with Pydantic AI

## Overview

This project demonstrates a powerful research agent built using Pydantic AI, capable of conducting in-depth internet searches and generating comprehensive research reports on various topics. The agent leverages advanced search capabilities and AI-powered analysis to provide detailed, structured information.

## Features

- Flexible search functionality using Tavily or DuckDuckGo
- AI-powered research generation
- Customizable search parameters
- Structured research output with title, main content, and bullet point summaries
- Date-aware research capabilities

## Prerequisites

Before running the project, ensure you have the following dependencies installed:

- Python 3.8+
- pip
- The following Python libraries:
  - pydantic-ai
  - nest_asyncio
  - devtools
  - duckduckgo-search (optional)
  - tavily-python (optional)

## Installation

1. Clone the repository:
```bash
git clone https://github.com/yourusername/research-agent.git
cd research-agent
```

2. Install required dependencies:
```bash
pip install pydantic-ai nest_asyncio devtools duckduckgo-search tavily-python
```

## Configuration

### API Keys

You'll need to set up environment variables for optional services:

- `GEMINI_API_KEY`: Google Gemini API key (optional)
- `GROQ_API_KEY`: Groq API key (optional)
- `TAVILY_API_KEY`: Tavily Search API key (recommended)

### Search Provider

The script supports two search providers:
1. **Tavily Search** (Recommended): Requires an API key
2. **DuckDuckGo Search** (Free alternative): No API key needed

## Project Structure

### Key Components

#### 1. Search Functionality
- `get_search()` tool: Handles search queries using either Tavily or DuckDuckGo
- Supports asynchronous search operations
- Configurable maximum results

#### 2. Agent Configuration
- Uses OpenAI's GPT-4o model
- Customizable system prompt
- Structured research result generation

#### 3. Research Dependencies
- `SearchDataclass`: Manages search parameters
- `ResearchResult`: Defines the structure of research output

## Usage Example

```python
# Set up dependencies
deps = SearchDataclass(max_results=3, todays_date=current_date)

# Run research on a topic
result = await search_agent.run(
    'What are the major AI News announcements in the last few days?', 
    deps=deps
)

# Access research results
print(result.data.research_title)
print(result.data.research_main)
print(result.data.research_bullets)
```

## Research Output Structure

The agent generates a structured research report with:
- A Markdown title
- Main research content
- Bullet point summary

## Advanced Features

### Dynamic System Prompt
The `add_current_date()` method dynamically updates the system prompt with the current date, enhancing context-awareness.

## Customization

### Modifying Search Parameters
- Adjust `max_results` to control the number of search results
- Switch between Tavily and DuckDuckGo by commenting/uncommenting the appropriate `get_search()` tool

## Limitations and Considerations

- Rate limits may apply with DuckDuckGo search
- Tavily offers more reliable search capabilities
- Requires an active internet connection
- Performance depends on the selected AI model and search provider

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.




- Pydantic AI
- OpenAI
- Tavily
- DuckDuckGo


I'll break down the code step by step, explaining its functionality and purpose:

### 1. Dependencies and Setup

```python
# Import necessary libraries
import os
from IPython.display import display, Markdown
import nest_asyncio
import datetime
from duckduckgo_search import DDGS, AsyncDDGS
from tavily import TavilyClient, AsyncTavilyClient
from pydantic_ai import Agent, ModelRetry, RunContext
from pydantic import BaseModel, Field
```

This section imports the required libraries:
- `os`: For environment variable management
- `nest_asyncio`: Allows nested event loops in Jupyter notebooks
- `datetime`: For handling current date
- Search libraries: DuckDuckGo and Tavily
- Pydantic AI for creating the research agent
- Pydantic for data modeling

### 2. API Key Setup

```python
# Set environment variables for API keys
os.environ["GEMINI_API_KEY"] = ""
os.environ["GROQ_API_KEY"] = ""
os.environ["TAVILY_API_KEY"] = ""

# Apply nest_asyncio to enable async functionality
nest_asyncio.apply()
```

This step sets up placeholder environment variables for different AI and search APIs. `nest_asyncio.apply()` allows running asynchronous code in Jupyter notebooks.

### 3. Search Functionality

The code provides two search options:

#### DuckDuckGo Search
```python
# Synchronous and Asynchronous DuckDuckGo searches
results = DDGS().text("python programming", max_results=5)
results = AsyncDDGS().text("python programming", max_results=5)
results = await AsyncDDGS().achat('describe human characteristics')
```

#### Tavily Search
```python
# Setup Tavily Client
tavily_client = AsyncTavilyClient(api_key=os.environ["TAVILY_API_KEY"])

# Simple search
response = await tavily_client.search("Who is Leo Messi?", max_results=3)

# Context search
context = await tavily_client.get_search_context(query="Who is Leo Messi?", max_results=3)
```

### 4. Data Models and Agent Setup

```python
# Dataclass for search parameters
@dataclass
class SearchDataclass:
    max_results: int
    todays_date: str

# Dataclass for research dependencies
@dataclass
class ResearchDependencies:
    todays_date: str

# Pydantic model for research results
class ResearchResult(BaseModel):
    research_title: str = Field(description='Markdown heading for the topic')
    research_main: str = Field(description='Main research content')
    research_bullets: str = Field(description='Summary bullet points')
```

These classes define the structure for:
- Search parameters
- Research dependencies
- Research result format

### 5. Agent Creation

```python
# Create the search agent
search_agent = Agent('openai:gpt-4o',
                     deps_type=ResearchDependencies,
                     result_type=ResearchResult,
                     system_prompt='You are a helpful research assistant...')
```

This creates an AI agent using GPT-4o, specifying:
- Dependency type
- Result type
- System prompt to guide the agent's behavior

### 6. Search Tool Definition

```python
@search_agent.tool  # Tavily search tool
async def get_search(search_data:RunContext[SearchDataclass], query: str, query_number: int) -> dict[str, Any]:
    """Perform search for a given keyword query"""
    print(f"Search query {query_number}: {query}")
    max_results = search_data.deps.max_results
    results = await tavily_client.get_search_context(query=query, max_results=max_results)
    return results
```

This defines a search tool that:
- Takes search parameters
- Performs a search using Tavily
- Returns search results

### 7. Running the Research Agent

```python
# Get current date
current_date = datetime.date.today()
date_string = current_date.strftime("%Y-%m-%d")

# Set up dependencies
deps = SearchDataclass(max_results=3, todays_date=date_string)

# Run research on a topic
result = await search_agent.run(
    'can you give me a very detailed bio of Sam Altman?', 
    deps=deps
)

# Print and format results
print(result.data)
result.data.research_title = '#' + result.data.research_title
print(result.data.research_title)
print(result.data.research_main)
print(result.data.research_bullets)
```

This section:
- Gets the current date
- Sets up search dependencies
- Runs the research agent on a specific query
- Formats and displays the results

### 8. Dynamic System Prompt

```python
@search_agent.system_prompt
async def add_current_date(ctx: RunContext[ResearchDependencies]) -> str:
    todays_date = ctx.deps.todays_date
    system_prompt = f'You are a helpful research assistant... if you need today\'s date it is {todays_date}'
    return system_prompt
```

This method dynamically updates the system prompt to include the current date, enhancing the agent's context awareness.

### Key Concepts

1. **Asynchronous Programming**: Uses `async/await` for non-blocking I/O operations
2. **Flexible Search**: Supports multiple search providers
3. **Structured Results**: Uses Pydantic models to define result format
4. **Dynamic Prompting**: Adapts system prompt based on context

### Potential Improvements
- Error handling
- More robust API key management
- Enhanced result formatting
- Support for more search providers

# Comprehensive Guide to Research Agent Architecture

## 1. Asynchronous Programming in the Code

### Core Concepts
Asynchronous programming in this project is implemented using Python's `async/await` syntax, which allows non-blocking I/O operations. This is crucial for search and AI operations that might take time.

#### Key Async Patterns
```python
# Async search method example
async def get_search(search_data: RunContext[SearchDataclass], query: str, query_number: int):
    # Non-blocking search operation
    results = await tavily_client.get_search_context(
        query=query, 
        max_results=search_data.deps.max_results
    )
    return results

# Async agent run
result = await search_agent.run(
    'Research topic', 
    deps=deps
)
```

### Benefits
- Concurrent execution
- Improved performance for I/O-bound tasks
- Efficient handling of multiple search queries
- Reduced waiting time for API responses

### Advanced Async Techniques
- Uses `nest_asyncio.apply()` to enable async in Jupyter notebooks
- Leverages Python's `asyncio` for managing asynchronous operations
- Supports multiple concurrent search queries

## 2. Pydantic AI Agent Architecture

### Design Components
```python
# Agent Creation Blueprint
search_agent = Agent(
    'openai:gpt-4o',  # AI Model
    deps_type=ResearchDependencies,  # Dependency Type
    result_type=ResearchResult,  # Result Structure
    system_prompt='Research assistant guidelines...'  # Initial Instruction Set
)
```

### Architecture Layers
1. **Model Layer**: OpenAI GPT-4o as the core intelligence
2. **Dependency Layer**: Manages contextual information
3. **Result Layer**: Structures output format
4. **Tool Layer**: Extends agent capabilities

### Dependency Management
```python
@dataclass
class ResearchDependencies:
    todays_date: str  # Context-aware dependency

@dataclass
class SearchDataclass:
    max_results: int  # Search configuration
    todays_date: str
```

## 3. Search Tool Mechanics

### Search Provider Abstraction
```python
@search_agent.tool
async def get_search(
    search_data: RunContext[SearchDataclass],
    query: str, 
    query_number: int
) -> dict[str, Any]:
    """Flexible search tool with configurable parameters"""
    max_results = search_data.deps.max_results
    results = await tavily_client.get_search_context(
        query=query, 
        max_results=max_results
    )
    return results
```

### Multi-Provider Support
- **Tavily**: Professional, API-based search
- **DuckDuckGo**: Free alternative search method
- Easily switchable implementation

### Search Strategy
1. Generate search keywords
2. Perform multiple targeted searches
3. Aggregate and synthesize results

## 4. Result Structuring

### Pydantic Result Model
```python
class ResearchResult(BaseModel):
    # Structured output with clear semantic meaning
    research_title: str = Field(
        description='Markdown heading for the topic'
    )
    research_main: str = Field(
        description='Comprehensive research content'
    )
    research_bullets: str = Field(
        description='Concise summary points'
    )
```

### Output Transformation
- Converts unstructured data to structured format
- Ensures consistent research presentation
- Enables easy markdown generation

## 5. Dynamic Prompt Generation

### Context-Aware Prompting
```python
@search_agent.system_prompt
async def add_current_date(ctx: RunContext[ResearchDependencies]) -> str:
    todays_date = ctx.deps.todays_date
    system_prompt = f'''
    You are a helpful research assistant:
    - Generate comprehensive research
    - Use current context: {todays_date}
    - Provide structured, actionable insights
    '''
    return system_prompt
```

### Prompt Evolution Techniques
- Dynamically injects current context
- Adapts instructions based on research requirements
- Maintains consistent agent behavior

## 6. Advanced Features and Design Patterns

### Composition Patterns
- Dependency Injection
- Strategy Pattern for Search Providers
- Decorator-based Tool Extension

### Extensibility
- Easily add new search tools
- Modify result structures
- Swap AI models

## 7. Error Handling and Improvements

### Potential Enhancements
```python
# Error Handling Decorator (Conceptual)
def handle_search_errors(func):
    async def wrapper(*args, **kwargs):
        try:
            return await func(*args, **kwargs)
        except Exception as e:
            # Log error
            # Implement fallback strategy
            print(f"Search error: {e}")
            return None
    return wrapper
```

### Recommended Improvements
- Robust error handling
- Timeout mechanisms
- Fallback search strategies
- Comprehensive logging
- Rate limit management

## 8. Integration of Search Providers

### Flexible Search Architecture
```python
# Abstracted Search Interface
class SearchProvider:
    async def search(self, query: str, max_results: int):
        """Common search method interface"""
        pass

# Concrete Implementations
class TavilySearchProvider(SearchProvider):
    async def search(self, query, max_results):
        # Tavily-specific implementation

class DuckDuckGoSearchProvider(SearchProvider):
    async def search(self, query, max_results):
        # DuckDuckGo-specific implementation
```

### Provider Switching Mechanism
- Interchangeable search backends
- Consistent interface
- Easy integration of new providers

## Conclusion

This research agent demonstrates a sophisticated, flexible architecture combining:
- Asynchronous programming
- AI-powered research
- Structured output generation
- Adaptive context management

