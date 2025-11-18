# Responses API

The Responses API provides OpenAI-compatible stateless inference with support for both streaming and non-streaming responses.

## Endpoint

```
POST /v1/responses
```

## Authentication

Requires Bearer token authentication:

```
Authorization: Bearer YOUR_API_TOKEN
```

## Features

- **Streaming & Non-Streaming**: Real-time SSE streaming or complete responses
- **Internal Tools**: Built-in tools (web_search, image_generation, etc.) are executed automatically
- **Custom Tools**: Client-defined tools are returned for client-side execution
- **Structured Outputs**: Enforce JSON schema compliance with strict validation
- **System Instructions**: Control model behavior with instructions parameter

## Request Format

The request body follows OpenAI's Responses API format:

```json
{
  "model": "gpt-4",
  "input": [
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "stream": false,
  "temperature": 0.7,
  "max_tokens": 1000,
  "instructions": "You are a helpful assistant.",
  "tools": [
    {
      "type": "function",
      "function": {
        "name": "web_search",
        "description": "Search the web for information",
        "parameters": {
          "type": "object",
          "properties": {
            "query": {
              "type": "string",
              "description": "The search query"
            }
          },
          "required": ["query"]
        }
      }
    }
  ],
  "text": {
    "format": {
      "type": "json_schema",
      "name": "response_format",
      "schema": {
        "type": "object",
        "properties": {
          "answer": {"type": "string"}
        },
        "required": ["answer"]
      },
      "strict": true
    }
  }
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | The model identifier (name, slug, or UUID) |
| `input` | string or array | Yes | Text string or array of message objects |
| `stream` | boolean | No | Whether to stream the response (default: false) |
| `temperature` | float | No | Sampling temperature (0.0 to 2.0) |
| `max_tokens` | int | No | Maximum tokens to generate |
| `tools` | array | No | Available tools (internal or custom) |
| `tool_choice` | string | No | Control tool selection: "auto", "required", "none" |
| `instructions` | string | No | System instructions for the model |
| `text` | object | No | Text output configuration including structured output |

### Message Format

Each message in the `input` array must have:

- `role`: One of `"user"`, `"assistant"`, `"system"`, or `"developer"`
- `content`: The message text content
- `tool_calls`: (Optional) Array of tool calls made by the assistant

### Tool Types

**Internal Tools** (executed automatically):
- `web_search`, `wikipedia_search`
- `image_generation`, `image_edit`
- `video_analysis`, `video_generation`
- `audio_transcription`, `audio_tts`
- `terminal`, `workflow_integration`

**Custom Tools** (returned for client execution):
- Any tool not in the internal list
- Tool calls are returned in the response
- Client must execute and provide results

### Structured Output Format

The `text.format` parameter enforces JSON schema compliance:

```json
{
  "text": {
    "format": {
      "type": "json_schema",
      "name": "my_schema",
      "schema": {
        "type": "object",
        "properties": {
          "field": {"type": "string"}
        },
        "required": ["field"]
      },
      "strict": true
    }
  }
}
```

## Response Format

### Non-Streaming Response

```json
{
  "id": "resp_abc123",
  "object": "response",
  "created_at": 1234567890.0,
  "status": "completed",
  "model": "gpt-4",
  "output": [
    {
      "type": "message",
      "id": "msg_xyz",
      "status": "completed",
      "role": "assistant",
      "content": [
        {
          "type": "output_text",
          "text": "I'm doing well, thank you for asking!",
          "annotations": []
        }
      ],
      "tool_calls": []
    }
  ],
  "usage": {
    "input_tokens": 10,
    "input_tokens_details": {"cached_tokens": 0},
    "output_tokens": 15,
    "output_tokens_details": {"reasoning_tokens": 0},
    "total_tokens": 25
  },
  "tools": [],
  "tool_choice": "auto"
}
```

### Streaming Response

When `stream: true`, the response is sent as Server-Sent Events (SSE):

```
event: response.created
data: {"type":"response.created","response":{...}}

event: response.output_text.delta
data: {"type":"response.output_text.delta","delta":"Hello","item_id":"msg_1"}

event: response.output_text.delta
data: {"type":"response.output_text.delta","delta":" world","item_id":"msg_1"}

event: response.done
data: {"type":"response.done","response":{...}}
```

## Available Tools

The following tools can be specified in the `tools` parameter:

- `web_search` - Search the web for information
- `wikipedia_search` - Search Wikipedia
- `image_generation` - Generate images from text descriptions
- `image_edit` - Edit existing images
- `video_analysis` - Analyze video content
- `video_generation` - Generate videos from descriptions
- `video_highlights` - Extract highlights from videos
- `audio_transcription` - Transcribe audio to text
- `audio_tts` - Convert text to speech
- `terminal` - Execute terminal commands
- `workflow_integration` - Execute custom workflow integrations

## Examples

### Basic Non-Streaming Request

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "input": "What is the capital of France?"
  }'
```

### Streaming Request

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "input": "Tell me a story",
    "stream": true
  }'
```

### Request with Internal Tools

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "input": "Search for recent AI developments"
  }'
```

### Request with Custom Tools

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "input": "Get the weather in San Francisco",
    "tools": [
      {
        "type": "function",
        "function": {
          "name": "get_weather",
          "description": "Get current weather",
          "parameters": {
            "type": "object",
            "properties": {
              "location": {"type": "string"}
            },
            "required": ["location"]
          }
        }
      }
    ]
  }'
```

### Request with Structured Output

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "input": "Extract the title and author from: The Great Gatsby by F. Scott Fitzgerald",
    "text": {
      "format": {
        "type": "json_schema",
        "name": "book_info",
        "schema": {
          "type": "object",
          "properties": {
            "title": {"type": "string"},
            "author": {"type": "string"}
          },
          "required": ["title", "author"]
        },
        "strict": true
      }
    }
  }'
```

## Python Examples

Using the OpenAI Python client:

### Basic Usage

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8081/v1",
    api_key="YOUR_TOKEN"
)

# Non-streaming
response = client.responses.create(
    model="gpt-4",
    input="What is quantum computing?"
)
print(response.output_text)

# Streaming
stream = client.responses.create(
    model="gpt-4",
    input="Tell me a story",
    stream=True
)

for event in stream:
    if event.type == "response.output_text.delta":
        print(event.delta, end="", flush=True)
```

### Custom Tools

```python
response = client.responses.create(
    model="gpt-4",
    input="What's the weather in Tokyo?",
    tools=[
        {
            "type": "function",
            "function": {
                "name": "get_weather",
                "description": "Get weather for a location",
                "parameters": {
                    "type": "object",
                    "properties": {
                        "location": {"type": "string"}
                    },
                    "required": ["location"]
                }
            }
        }
    ]
)

# Check for tool calls
if response.output[0].tool_calls:
    for tool_call in response.output[0].tool_calls:
        print(f"Tool: {tool_call.function.name}")
        print(f"Args: {tool_call.function.arguments}")
```

### Structured Output

```python
from pydantic import BaseModel

class BookInfo(BaseModel):
    title: str
    author: str
    year: int

response = client.responses.parse(
    model="gpt-4",
    input="Extract info: '1984 by George Orwell, published in 1949'",
    text_format=BookInfo
)

book = response.output_parsed
print(f"{book.title} by {book.author} ({book.year})")
```

## Error Handling

The API returns standard HTTP status codes:

- `200` - Success
- `400` - Bad request (invalid parameters)
- `401` - Unauthorized (invalid or missing token)
- `404` - Model not found
- `500` - Server error

Error responses include a message:

```json
{
  "detail": "Model 'invalid-model' not found"
}
```



