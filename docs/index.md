# AGG AI Proxy API

Welcome to the AGG AI Proxy API documentation. This API provides a simplified interface to interact with the AGG AI Service, enabling you to build AI-powered applications with ease.

## Overview

The AGG AI Proxy API is a streamlined gateway that provides:

- **Simple Authentication**: Use Bearer tokens for straightforward API access
- **OpenAI-Compatible Inference**: Stateless AI inference with streaming support
- **Tool Execution**: Execute specialized AI tools for web search, image generation, and more
- **Full OpenAPI Support**: Interactive documentation and client generation

## Why Use the Proxy API?

The Proxy API simplifies integration by:

1. **Unified Authentication**: Single API token instead of managing multiple credentials
2. **Simplified Schemas**: Cleaner request/response formats optimized for external use
3. **Focused Functionality**: Core features without internal complexity
4. **Easy Deployment**: Standalone service that can be scaled independently

## Key Features

### 🤖 OpenAI-Compatible Inference

Use the OpenAI Responses API format for stateless inference:

```python
from openai import OpenAI

client = OpenAI(
    base_url="https://api.your-domain.com/v1",
    api_key="your-token"
)

response = client.responses.create(
    model="gpt-4",
    messages=[
        {"role": "user", "content": "What is quantum computing?"}
    ],
    stream=True
)

for chunk in response:
    if chunk.choices[0].delta.content:
        print(chunk.choices[0].delta.content, end="")
```

### 🛠️ AI Tool Execution

Execute specialized AI tools directly:

```python
import requests

response = requests.post(
    "https://api.your-domain.com/tools/execute",
    headers={"Authorization": "Bearer your-token"},
    json={
        "tool_name": "web_search",
        "arguments": {"query": "latest AI developments"},
        "model": {"model_id": "model-uuid"}
    }
)

result = response.json()
```

### 📊 Available Tools

- **web_search**: Search the web for current information
- **wikipedia_search**: Query Wikipedia articles
- **image_generation**: Generate images from text descriptions
- **image_edit**: Edit existing images with AI
- **video_analysis**: Analyze video content
- **video_generation**: Generate videos from descriptions
- **audio_transcription**: Transcribe audio to text
- **audio_tts**: Convert text to speech
- **terminal**: Execute safe terminal commands
- **workflow_integration**: Custom workflow integrations

## Quick Links

- [Quickstart Guide](quickstart.md) - Get up and running in minutes
- [Authentication](authentication.md) - Learn about API authentication
- [Responses API](api/responses.md) - OpenAI-compatible inference endpoint
- [Tools API](api/tools.md) - Execute AI tools
- [Responses API Examples](examples/responses-api.md) - Complete examples with OpenAI SDK

## API Endpoints

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/v1/responses` | POST | Create AI responses (OpenAI-compatible) |
| `/tools` | GET | List available tools |
| `/tools/execute` | POST | Execute a specific tool |
| `/health` | GET | Check API health status |

## Getting Help

- Check the [API Reference](api/responses.md) for detailed endpoint documentation
- See [Examples](examples/basic.md) for common use cases
- Review error codes and troubleshooting in each API section

## Rate Limits

The API is subject to rate limits to ensure fair usage:

- 100 requests per minute per token
- 1000 requests per hour per token

Contact support for higher limits if needed.

## Support

For issues, questions, or feature requests:

- Email: support@your-domain.com
- GitHub Issues: [github.com/your-org/agg-ai-proxy](https://github.com/your-org/agg-ai-proxy)

