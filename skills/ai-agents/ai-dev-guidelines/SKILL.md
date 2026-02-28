---
name: ai-dev-guidelines
description: Comprehensive AI/ML development guide for LangChain, LangGraph, and ML model integration in FastAPI. Use when building LLM applications, agents, RAG systems, sentiment analysis, aspect-based analysis, chain orchestration, prompt engineering, vector stores, embeddings, or integrating ML models with FastAPI endpoints. Covers LangChain patterns, LangGraph state machines, model deployment, API integration, streaming, error handling, and best practices.
---

# AI/ML Development Guidelines (LangChain/LangGraph/FastAPI)

## Purpose

Establish best practices for integrating AI/ML capabilities into FastAPI applications, with focus on LangChain, LangGraph, and ML model deployment.

## When to Use This Skill

Automatically activates when working on:

- LangChain chains, agents, tools
- LangGraph workflows and state machines
- RAG (Retrieval Augmented Generation)
- Prompt engineering and templates
- Vector stores and embeddings
- ML model integration (sentiment analysis, aspect-based analysis, etc.)
- Streaming LLM responses
- AI service layers
- Model deployment and optimization

---

## Quick Start

### New AI Feature Checklist

- [ ] **Model Selection**: Choose appropriate model/API (OpenAI, Anthropic, local)
- [ ] **Prompt Design**: Create prompt templates
- [ ] **Chain/Graph**: Build LangChain chain or LangGraph workflow
- [ ] **API Endpoint**: FastAPI route with streaming support
- [ ] **Error Handling**: Retry logic, fallbacks, timeouts
- [ ] **Validation**: Input/output validation with Pydantic
- [ ] **Monitoring**: Log tokens, latency, errors
- [ ] **Testing**: Unit tests with mocked LLM responses
- [ ] **Documentation**: Document prompts and expected behavior

---

## Architecture for AI Features

### Layered AI Architecture

```
HTTP Request
    ↓
FastAPI Route (streaming setup)
    ↓
AI Service (chain/graph orchestration)
    ↓
LangChain/LangGraph (LLM calls)
    ↓
Model/API (OpenAI, Anthropic, HuggingFace, local)
```

**Current Project ML Structure:**

```
backend/app/
├── routes/
│   └── ML_Routes.py        # ML API endpoints
├── machine_learning/
│   └── mlendpoint.py       # Sentiment analysis functions
└── services/               # TO BE CREATED for LangChain/LangGraph
    └── ai_services/
        ├── sentiment.py    # Refactored sentiment analysis
        ├── chains.py       # LangChain chains
        ├── graphs.py       # LangGraph workflows
        └── prompts.py      # Prompt templates
```

---

## Package Management

**This project uses `uv` for Python package management.**

```bash
# Add LangChain dependencies
uv add langchain langchain-openai langchain-anthropic langgraph

# Add vector store dependencies (when needed)
uv add faiss-cpu chromadb

# Add other AI dependencies
uv add tiktoken sentence-transformers

# Run Python scripts with uv
uv run python script.py
```

**❌ NEVER use `pip install`** - Always use `uv add` instead.

---

## Core Principles for AI Development

### 1. Separate Prompts from Code

```python
# ❌ NEVER: Hardcoded prompts in code
def analyze_text(text: str):
    response = llm.invoke(f"Analyze sentiment of: {text}")
    return response

# ✅ ALWAYS: Templated prompts
from langchain.prompts import ChatPromptTemplate

SENTIMENT_PROMPT = ChatPromptTemplate.from_messages([
    ("system", "You are a sentiment analysis expert."),
    ("user", "Analyze the sentiment of the following text:\n\n{text}\n\nProvide: sentiment (positive/neutral/negative) and confidence (0-1).")
])

def analyze_text(text: str):
    chain = SENTIMENT_PROMPT | llm | output_parser
    return chain.invoke({"text": text})
```

### 2. Use Pydantic for LLM Output Validation

```python
from pydantic import BaseModel, Field
from langchain.output_parsers import PydanticOutputParser

class SentimentResult(BaseModel):
    sentiment: str = Field(description="positive, neutral, or negative")
    confidence: float = Field(ge=0.0, le=1.0)
    reasoning: str = Field(description="Brief explanation")

# Create parser
parser = PydanticOutputParser(pydantic_object=SentimentResult)

# Add format instructions to prompt
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a sentiment expert."),
    ("user", "{text}\n\n{format_instructions}")
])

# Use in chain
chain = prompt | llm | parser

result: SentimentResult = chain.invoke({
    "text": "I love this product!",
    "format_instructions": parser.get_format_instructions()
})
```

### 3. Handle Streaming for Better UX

```python
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
from langchain.callbacks.streaming_stdout import StreamingStdOutCallbackHandler

# Setup streaming LLM
from langchain_openai import ChatOpenAI

llm = ChatOpenAI(
    model="gpt-4",
    streaming=True,
    callbacks=[StreamingStdOutCallbackHandler()]
)

@app.post("/ai/generate-stream")
async def generate_stream(request: GenerateRequest):
    """Stream LLM responses for real-time feedback"""

    async def event_generator():
        async for chunk in chain.astream({"input": request.text}):
            if chunk:
                yield f"data: {chunk}\n\n"

    return StreamingResponse(
        event_generator(),
        media_type="text/event-stream"
    )
```

### 4. Implement Retry Logic and Fallbacks

```python
from tenacity import retry, stop_after_attempt, wait_exponential
from langchain.chat_models import ChatOpenAI, ChatAnthropic

@retry(
    stop=stop_after_attempt(3),
    wait=wait_exponential(multiplier=1, min=2, max=10)
)
async def call_llm_with_retry(prompt: str):
    try:
        # Try primary model
        response = await primary_llm.ainvoke(prompt)
        return response
    except Exception as e:
        # Log and fallback
        logger.error(f"Primary LLM failed: {e}")
        # Try fallback model
        response = await fallback_llm.ainvoke(prompt)
        return response
```

### 5. Use Environment Variables for API Keys

```python
# ❌ NEVER: Hardcoded API keys
llm = ChatOpenAI(api_key="sk-...")

# ✅ ALWAYS: From environment
import os
from pydantic_settings import BaseSettings

class AISettings(BaseSettings):
    openai_api_key: str
    anthropic_api_key: str
    model_name: str = "gpt-4"
    max_tokens: int = 1000
    temperature: float = 0.7

    class Config:
        env_file = ".env"

settings = AISettings()

llm = ChatOpenAI(
    api_key=settings.openai_api_key,
    model=settings.model_name,
    max_tokens=settings.max_tokens
)
```

### 6. Track Token Usage and Costs

```python
from langchain.callbacks import get_openai_callback

@app.post("/ai/analyze")
async def analyze_with_tracking(text: str):
    with get_openai_callback() as cb:
        result = chain.invoke({"text": text})

        # Log metrics
        logger.info(f"""
        Tokens used: {cb.total_tokens}
        Prompt tokens: {cb.prompt_tokens}
        Completion tokens: {cb.completion_tokens}
        Total cost: ${cb.total_cost}
        """)

    return {
        "result": result,
        "metadata": {
            "tokens": cb.total_tokens,
            "cost": cb.total_cost
        }
    }
```

### 7. Implement Proper Error Boundaries

```python
from fastapi import HTTPException
from langchain.schema import LLMResult
from openai.error import RateLimitError, APIError

@app.post("/ai/process")
async def process_with_ai(request: AIRequest):
    try:
        result = await ai_service.process(request.text)
        return result

    except RateLimitError:
        raise HTTPException(
            status_code=429,
            detail="Rate limit exceeded. Please try again later."
        )

    except APIError as e:
        logger.error(f"LLM API error: {e}")
        raise HTTPException(
            status_code=503,
            detail="AI service temporarily unavailable"
        )

    except Exception as e:
        logger.error(f"Unexpected error in AI processing: {e}")
        raise HTTPException(
            status_code=500,
            detail="Error processing request"
        )
```

---

## LangChain Patterns

### Basic Chain Pattern

```python
from langchain_openai import ChatOpenAI
from langchain.prompts import ChatPromptTemplate
from langchain.output_parsers import StrOutputParser

# Components
llm = ChatOpenAI(model="gpt-4")
prompt = ChatPromptTemplate.from_messages([
    ("system", "You are a helpful assistant."),
    ("user", "{input}")
])
output_parser = StrOutputParser()

# Chain
chain = prompt | llm | output_parser

# Invoke
result = chain.invoke({"input": "Hello!"})
```

### Chain with Memory

```python
from langchain.memory import ConversationBufferMemory
from langchain.chains import ConversationChain

memory = ConversationBufferMemory()

conversation = ConversationChain(
    llm=llm,
    memory=memory,
    verbose=True
)

# First interaction
response1 = conversation.predict(input="Hi, I'm Aaron")

# Memory persists
response2 = conversation.predict(input="What's my name?")
# Will respond: "Your name is Aaron"
```

### RAG Chain Pattern

```python
from langchain.vectorstores import FAISS
from langchain.embeddings import OpenAIEmbeddings
from langchain.text_splitter import RecursiveCharacterTextSplitter
from langchain.chains import RetrievalQA

# 1. Prepare documents
text_splitter = RecursiveCharacterTextSplitter(
    chunk_size=1000,
    chunk_overlap=200
)
texts = text_splitter.split_documents(documents)

# 2. Create vector store
embeddings = OpenAIEmbeddings()
vectorstore = FAISS.from_documents(texts, embeddings)

# 3. Create retrieval chain
qa_chain = RetrievalQA.from_chain_type(
    llm=llm,
    chain_type="stuff",
    retriever=vectorstore.as_retriever(search_kwargs={"k": 3})
)

# 4. Query
result = qa_chain.invoke({"query": "What are the main findings?"})
```

### Tool-Using Agent

```python
from langchain.agents import create_openai_functions_agent, AgentExecutor
from langchain.tools import Tool

# Define tools
def search_projects(query: str) -> str:
    """Search projects in database"""
    # Your search logic
    return f"Found 3 projects matching {query}"

def get_sentiment(text: str) -> str:
    """Analyze sentiment of text"""
    # Your sentiment analysis
    return "positive"

tools = [
    Tool(
        name="search_projects",
        func=search_projects,
        description="Search for urban planning projects by name or city"
    ),
    Tool(
        name="analyze_sentiment",
        func=get_sentiment,
        description="Analyze sentiment of text (positive/neutral/negative)"
    )
]

# Create agent
agent = create_openai_functions_agent(llm, tools, prompt)
agent_executor = AgentExecutor(agent=agent, tools=tools)

# Execute
result = agent_executor.invoke({
    "input": "Find projects in Mexico City and analyze sentiment"
})
```

---

## LangGraph Patterns

### Simple State Machine

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
from operator import add

# Define state
class AgentState(TypedDict):
    input: str
    analysis: str
    sentiment: str
    final_output: str

# Define nodes
def analyze_node(state: AgentState):
    # Perform analysis
    analysis = llm.invoke(f"Analyze: {state['input']}")
    return {"analysis": analysis}

def sentiment_node(state: AgentState):
    # Extract sentiment
    sentiment = extract_sentiment(state['analysis'])
    return {"sentiment": sentiment}

def format_node(state: AgentState):
    # Format output
    output = f"Analysis: {state['analysis']}\nSentiment: {state['sentiment']}"
    return {"final_output": output}

# Build graph
workflow = StateGraph(AgentState)

workflow.add_node("analyze", analyze_node)
workflow.add_node("sentiment", sentiment_node)
workflow.add_node("format", format_node)

workflow.set_entry_point("analyze")
workflow.add_edge("analyze", "sentiment")
workflow.add_edge("sentiment", "format")
workflow.add_edge("format", END)

app = workflow.compile()

# Run
result = app.invoke({"input": "This project is amazing!"})
```

### Conditional Routing

```python
def route_by_sentiment(state: AgentState):
    """Route based on sentiment"""
    if state["sentiment"] == "negative":
        return "handle_negative"
    elif state["sentiment"] == "positive":
        return "handle_positive"
    else:
        return "handle_neutral"

# Add conditional edges
workflow.add_conditional_edges(
    "sentiment",
    route_by_sentiment,
    {
        "handle_negative": "negative_handler",
        "handle_positive": "positive_handler",
        "handle_neutral": "neutral_handler"
    }
)
```

## Detailed Guides

### [LangChain Patterns](resources/langchain-patterns.md)

- Chain composition
- Memory and context
- Tools and agents
- RAG implementation

### [LangGraph Workflows](resources/langgraph-workflows.md)

- State machines
- Conditional routing
- Multi-agent systems
- Complex orchestration

### [Prompt Engineering](resources/prompt-engineering.md)

- Effective prompt design
- Few-shot learning
- Chain-of-thought prompting
- Prompt templates

### [Model Deployment](resources/model-deployment.md)

- Local model serving
- API integration
- Optimization and caching
- Cost management

### [Testing AI Systems](resources/testing-ai.md)

- Unit testing with mocks
- Integration testing
- Prompt testing
- Evaluation metrics

---

## Quick Reference

### Common LangChain Imports

```python
from langchain_openai import ChatOpenAI, OpenAI
from langchain_anthropic import ChatAnthropic
from langchain.prompts import ChatPromptTemplate, PromptTemplate
from langchain.output_parsers import PydanticOutputParser, StrOutputParser
from langchain.chains import LLMChain, SequentialChain
from langchain.memory import ConversationBufferMemory
from langchain.agents import create_openai_functions_agent, AgentExecutor
from langchain.tools import Tool
from langchain.callbacks import get_openai_callback
```

### Common LangGraph Imports

```python
from langgraph.graph import StateGraph, END
from typing import TypedDict, Annotated
from operator import add
```

### FastAPI + Streaming

```python
from fastapi.responses import StreamingResponse

async def stream_response():
    async for chunk in chain.astream(input):
        yield f"data: {chunk}\n\n"

return StreamingResponse(stream_response(), media_type="text/event-stream")
```

---

## Resources

- [LangChain Docs](https://python.langchain.com/)
- [LangGraph Docs](https://langchain-ai.github.io/langgraph/)
- [OpenAI Cookbook](https://cookbook.openai.com/)
- [Anthropic Docs](https://docs.anthropic.com/)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

---

**Remember:** AI features require careful prompt design, error handling, and monitoring. Always validate LLM outputs, implement retry logic, and track costs!
