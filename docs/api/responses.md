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

## Request Format

The request body follows OpenAI's Responses API format:

```json
{
  "model": "gpt-4",
  "messages": [
    {
      "role": "user",
      "content": "Hello, how are you?"
    }
  ],
  "stream": false,
  "temperature": 0.7,
  "max_tokens": 1000,
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
  ]
}
```

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `model` | string | Yes | The model identifier (name, slug, or UUID) |
| `messages` | array | Yes | Array of message objects with `role` and `content` |
| `stream` | boolean | No | Whether to stream the response (default: false) |
| `temperature` | float | No | Sampling temperature (0.0 to 2.0) |
| `max_tokens` | int | No | Maximum tokens to generate |
| `tools` | array | No | Available tools for the model to use |
| `tool_choice` | string/object | No | Control tool selection behavior |

### Message Format

Each message must have:

- `role`: One of `"user"`, `"assistant"`, or `"system"`
- `content`: The message text content
- `tool_calls`: (Optional) Array of tool calls made by the assistant

## Response Format

### Non-Streaming Response

```json
{
  "id": "resp_abc123",
  "object": "response",
  "created": 1234567890,
  "model": "gpt-4",
  "choices": [
    {
      "index": 0,
      "message": {
        "role": "assistant",
        "content": "I'm doing well, thank you for asking!"
      },
      "finish_reason": "stop"
    }
  ],
  "usage": {
    "prompt_tokens": 10,
    "completion_tokens": 15,
    "total_tokens": 25
  }
}
```

### Streaming Response

When `stream: true`, the response is sent as Server-Sent Events (SSE):

```
data: {"id":"resp_abc123","object":"response.stream","created":1234567890,"model":"gpt-4","choices":[{"index":0,"delta":{"content":"I'm"},"finish_reason":null}]}

data: {"id":"resp_abc123","object":"response.stream","created":1234567890,"model":"gpt-4","choices":[{"index":0,"delta":{"content":" doing"},"finish_reason":null}]}

data: {"id":"resp_abc123","object":"response.stream","created":1234567890,"model":"gpt-4","choices":[{"index":0,"delta":{"content":" well"},"finish_reason":null}]}

data: {"id":"resp_abc123","object":"response.stream","created":1234567890,"model":"gpt-4","choices":[{"index":0,"delta":{},"finish_reason":"stop"}]}

data: [DONE]
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
    "messages": [
      {
        "role": "user",
        "content": "What is the capital of France?"
      }
    ]
  }'
```

### Streaming Request

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {
        "role": "user",
        "content": "Tell me a story"
      }
    ],
    "stream": true
  }'
```

### Request with Tools

```bash
curl -X POST http://localhost:8081/v1/responses \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "model": "gpt-4",
    "messages": [
      {
        "role": "user",
        "content": "Search for recent AI developments"
      }
    ],
    "tools": [
      {
        "type": "function",
        "function": {
          "name": "web_search"
        }
      }
    ]
  }'
```

## Python Example

Using the OpenAI Python client:

```python
from openai import OpenAI

client = OpenAI(
    base_url="http://localhost:8081/v1",
    api_key="YOUR_TOKEN"
)

# Non-streaming
response = client.responses.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Hello!"}
    ]
)
print(response.choices[0].message.content)

# Streaming
stream = client.responses.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "Tell me a story"}
    ],
    stream=True
)

for chunk in stream:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
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



