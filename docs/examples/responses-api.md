# Responses API Examples with OpenAI SDK

Complete examples using the OpenAI Python SDK with the Responses API.

## Setup

Install the OpenAI SDK:

```bash
pip install openai
```

Initialize the client:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8081/v1",  # or your proxy URL
    api_key="your-proxy-token"
)
```

## Basic Usage

### Simple Question (Non-Streaming)

```python
response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="What is quantum computing?"
)

print(response.output_text)
print(f"Tokens used: {response.usage.total_tokens}")
```

### Streaming Response

```python
stream = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Tell me a story about a robot",
    stream=True
)

for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
```

### With System Instructions

```python
response = client.responses.create(
    model="openai/gpt-4o-mini",
    instructions="You are a helpful assistant that explains concepts simply.",
    input="Explain neural networks"
)

print(response.output_text)
```

## Internal Tools

Internal tools are executed automatically by the API.

### Web Search (Non-Streaming)

```python
response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="What are the latest news about artificial intelligence? Search the web."
)

print(f"Response: {response.output_text[:200]}...")

# Check if tools were used
if response.output[0].tool_calls:
    print("\nTools used:")
    for tc in response.output[0].tool_calls:
        print(f"  - {tc.function.name}")
```

### Web Search (Streaming)

```python
stream = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Find recent news about OpenAI GPT models using web search.",
    stream=True
)

full_text = ""

for event in stream:
    if event.type == "response.output_text.delta":
        full_text += event.delta
        print(event.delta, end="", flush=True)
    elif event.type == "response.done":
        print(f"\n\nResponse ID: {event.response.id}")
        print(f"Tokens used: {event.response.usage.total_tokens}")
```

## Custom Tools

Custom tools are returned to the client for execution (not executed by the API).

### Single Custom Tool (Non-Streaming)

```python
from typing import Any, cast

custom_tools = [
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "Performs mathematical calculations",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {
                        "type": "string",
                        "description": "Mathematical expression to evaluate"
                    }
                },
                "required": ["expression"]
            }
        }
    }
]

response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Calculate 123 * 456 using the calculator tool.",
    tools=cast(Any, custom_tools)
)

print(f"Response ID: {response.id}")
print(f"Tools provided: {len(response.tools)}")

# Check for tool calls
if response.output[0].tool_calls:
    print(f"\nTool calls made: {len(response.output[0].tool_calls)}")
    for tc in response.output[0].tool_calls:
        print(f"  Function: {tc.function.name}")
        print(f"  Arguments: {tc.function.arguments}")
```

### Custom Tool (Streaming)

```python
custom_tools = [
    {
        "type": "function",
        "function": {
            "name": "get_weather",
            "description": "Get weather information for a location",
            "parameters": {
                "type": "object",
                "properties": {
                    "location": {
                        "type": "string",
                        "description": "City name"
                    },
                    "units": {
                        "type": "string",
                        "enum": ["celsius", "fahrenheit"],
                        "description": "Temperature units"
                    }
                },
                "required": ["location"]
            }
        }
    }
]

stream = client.responses.create(
    model="openai/gpt-4o-mini",
    input="What's the weather in London? Use the get_weather tool.",
    tools=cast(Any, custom_tools),
    stream=True
)

for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
    elif event.type == "response.done":
        print(f"\n\nResponse ID: {event.response.id}")
        print(f"Tools provided: {len(event.response.tools)}")
        
        # Check for tool calls
        for output_item in event.response.output:
            if output_item.tool_calls:
                print(f"\nTool calls made: {len(output_item.tool_calls)}")
                for tc in output_item.tool_calls:
                    print(f"  Function: {tc.function.name}")
                    print(f"  Arguments: {tc.function.arguments}")
```

### Multiple Custom Tools

```python
custom_tools = [
    {
        "type": "function",
        "function": {
            "name": "calculator",
            "description": "Performs calculations",
            "parameters": {
                "type": "object",
                "properties": {
                    "expression": {"type": "string"}
                },
                "required": ["expression"]
            }
        }
    },
    {
        "type": "function",
        "function": {
            "name": "unit_converter",
            "description": "Converts units",
            "parameters": {
                "type": "object",
                "properties": {
                    "value": {"type": "number"},
                    "from_unit": {"type": "string"},
                    "to_unit": {"type": "string"}
                },
                "required": ["value", "from_unit", "to_unit"]
            }
        }
    }
]

response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Calculate 50 * 20, then convert 100 kilometers to miles.",
    tools=cast(Any, custom_tools)
)

if response.output[0].tool_calls:
    print(f"Tool calls made: {len(response.output[0].tool_calls)}")
    for i, tc in enumerate(response.output[0].tool_calls):
        print(f"  {i+1}. {tc.function.name}")
```

## Structured Outputs

Enforce JSON schema compliance in responses.

### Using `.parse()` Method (Recommended)

```python
from pydantic import BaseModel

class PersonProfile(BaseModel):
    name: str
    age: int
    interests: list[str]

response = client.responses.parse(
    model="openai/gpt-4o-mini",
    input="Create a profile for Alice, 28 years old, likes reading, hiking, and photography.",
    text_format=PersonProfile
)

profile = response.output_parsed
print(f"Name: {profile.name}")
print(f"Age: {profile.age}")
print(f"Interests: {profile.interests}")
```

### Using Raw JSON Schema

```python
import json

schema = {
    "type": "object",
    "properties": {
        "name": {"type": "string"},
        "age": {"type": "number"},
        "interests": {
            "type": "array",
            "items": {"type": "string"}
        }
    },
    "required": ["name", "age", "interests"],
    "additionalProperties": False
}

response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Create a profile for Bob, 35 years old, likes coding, gaming, and cooking.",
    text={
        "format": {
            "type": "json_schema",
            "name": "person_profile",
            "schema": schema,
            "strict": True
        }
    }
)

# Parse the structured output
parsed = json.loads(response.output_text)
print(f"Name: {parsed['name']}")
print(f"Age: {parsed['age']}")
print(f"Interests: {parsed['interests']}")
```

### Structured Output with Streaming

```python
from pydantic import BaseModel

class MLSummary(BaseModel):
    title: str
    summary: str
    tags: list[str]

with client.responses.stream(
    model="openai/gpt-4o-mini",
    input="Summarize machine learning with a title, brief summary, and tags.",
    text_format=MLSummary
) as stream:
    for event in stream:
        if event.type == "response.output_text.delta":
            print(event.delta, end="", flush=True)
    
    final_response = stream.get_final_response()

summary = final_response.output_parsed
print(f"\n\nTitle: {summary.title}")
print(f"Summary: {summary.summary[:100]}...")
print(f"Tags: {summary.tags}")
```

## Advanced Patterns

### Error Handling

```python
try:
    response = client.responses.create(
        model="openai/gpt-4o-mini",
        input="Hello, world!",
        max_tokens=100
    )
    print(response.output_text)
except Exception as e:
    print(f"Error: {e}")
```

### With Temperature Control

```python
# More creative (higher temperature)
response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Write a creative story opening",
    temperature=0.9
)
print(response.output_text)

# More deterministic (lower temperature)
response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="What is 2+2?",
    temperature=0.1
)
print(response.output_text)
```

### Conversation History

```python
messages = [
    {"role": "user", "content": "What is Python?"},
    {"role": "assistant", "content": "Python is a high-level programming language..."},
    {"role": "user", "content": "What are its main features?"}
]

response = client.responses.create(
    model="openai/gpt-4o-mini",
    input=messages
)

print(response.output_text)
```

## Tool Execution Examples

Using the `/api/tools` endpoints directly.

### Get Available Tools

```python
import requests

response = requests.get(
    "http://localhost:8081/api/tools",
    headers={"Authorization": "Bearer your-token"}
)

tools = response.json()
print(f"Found {len(tools)} tools\n")

for tool in tools[:3]:
    print(f"Name: {tool['name']}")
    print(f"Description: {tool['description'][:80]}...")
    print()
```

### Execute Tool with Dict Arguments

```python
import requests

payload = {
    "type": "function",
    "function": {
        "name": "web_search",
        "arguments": {"query": "OpenAI Responses API"}
    }
}

response = requests.post(
    "http://localhost:8081/api/tools/execute",
    headers={"Authorization": "Bearer your-token"},
    json=payload
)

result = response.json()
if result.get('content', {}).get('results'):
    print(f"Found {len(result['content']['results'])} results")
    print(f"First result: {result['content']['results'][0]['title']}")
```

### Execute Tool with String Arguments

```python
import requests
import json

payload = {
    "type": "function",
    "function": {
        "name": "web_search",
        "arguments": json.dumps({"query": "Python async programming"})
    }
}

response = requests.post(
    "http://localhost:8081/api/tools/execute",
    headers={"Authorization": "Bearer your-token"},
    json=payload
)

result = response.json()
print(f"Result type: {result.get('content', {}).get('type')}")
```

## Best Practices

### 1. Always Handle Streaming Events

```python
stream = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Explain quantum computing",
    stream=True
)

for event in stream:
    if event.type == "response.output_text.delta":
        # Handle text chunks
        print(event.delta, end="", flush=True)
    elif event.type == "response.done":
        # Handle completion
        print(f"\n\nCompleted: {event.response.id}")
    elif event.type == "error":
        # Handle errors
        print(f"\nError: {event.error}")
```

### 2. Use Structured Outputs for Reliable Parsing

Instead of parsing free-form JSON, use structured outputs:

```python
from pydantic import BaseModel

class ExtractedData(BaseModel):
    key_points: list[str]
    sentiment: str
    confidence: float

response = client.responses.parse(
    model="openai/gpt-4o-mini",
    input="Analyze this review: 'Great product, highly recommend!'",
    text_format=ExtractedData
)

data = response.output_parsed
# Guaranteed to have the correct structure
```

### 3. Implement Retry Logic

```python
import time

def create_response_with_retry(max_retries=3, **kwargs):
    for attempt in range(max_retries):
        try:
            return client.responses.create(**kwargs)
        except Exception as e:
            if attempt < max_retries - 1:
                time.sleep(2 ** attempt)
            else:
                raise

response = create_response_with_retry(
    model="openai/gpt-4o-mini",
    input="Hello!"
)
```

### 4. Monitor Token Usage

```python
response = client.responses.create(
    model="openai/gpt-4o-mini",
    input="Explain machine learning"
)

print(f"Input tokens: {response.usage.input_tokens}")
print(f"Output tokens: {response.usage.output_tokens}")
print(f"Total tokens: {response.usage.total_tokens}")
```

## Next Steps

- Review [API Reference](../api/responses.md) for complete parameter documentation
- Check [Authentication](../authentication.md) for security best practices
- Explore [Basic Examples](basic.md) for more patterns

