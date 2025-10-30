# Conversation API

The Conversation API allows you to send messages to AI models and receive streaming responses.

## Endpoint

```
POST /conversation
```

## Authentication

Requires Bearer token authentication.

## Request Format

Content-Type: `multipart/form-data`

### Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `message` | string (JSON) | Yes | JSON string containing the conversation request |
| `files` | file[] | No | Optional file attachments |

### Message JSON Structure

```json
{
  "model_ids": ["uuid1", "uuid2"],
  "message": {
    "content": "Your message here",
    "attachments": ["attachment-uuid"]
  },
  "options": {
    "disable_tool_calls": false,
    "tools": ["web_search", "image_generation"]
  }
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `model_ids` | string[] | Yes | Array of model UUIDs to use |
| `message.content` | string | Yes | The message content |
| `message.attachments` | string[] | No | UUIDs of previously uploaded attachments |
| `options.disable_tool_calls` | boolean | No | Disable tool execution |
| `options.tools` | string[] | No | Specific tools to enable |

## Response Format

The response is a server-sent events (SSE) stream with the following event types:

### chat_metadata

Sent first with basic chat information:

```json
{
  "type": "chat_metadata",
  "chat": {
    "id": "chat-uuid",
    "title": "New Chat"
  }
}
```

### message_start

Sent for each model when it starts generating:

```json
{
  "type": "message_start",
  "message": {
    "id": "message-uuid",
    "role": "assistant",
    "model_id": "model-uuid",
    "model_name": "GPT-4"
  }
}
```

### message_content

Content chunks as they're generated:

```json
{
  "type": "message_content",
  "model_id": "model-uuid",
  "message_id": "message-uuid",
  "content": {
    "type": "text",
    "text": "Here is "
  }
}
```

### tool_calls

When tools are being executed:

```json
{
  "type": "tool_calls",
  "model_id": "model-uuid",
  "message_id": "message-uuid",
  "tool_calls": ["web_search"]
}
```

### message_content_stop

Sent when a model finishes:

```json
{
  "type": "message_content_stop",
  "model_id": "model-uuid",
  "message_id": "message-uuid"
}
```

## Examples

### Basic Conversation

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F 'message={
    "model_ids": ["model-uuid"],
    "message": {
      "content": "What is machine learning?"
    }
  }'
```

### With File Upload

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F 'message={
    "model_ids": ["model-uuid"],
    "message": {
      "content": "Analyze this image"
    }
  }' \
  -F 'files=@image.jpg'
```

### Multiple Files

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F 'message={
    "model_ids": ["model-uuid"],
    "message": {
      "content": "Compare these images"
    }
  }' \
  -F 'files=@image1.jpg' \
  -F 'files=@image2.jpg'
```

### With Tool Configuration

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F 'message={
    "model_ids": ["model-uuid"],
    "message": {
      "content": "Search for recent AI news"
    },
    "options": {
      "tools": ["web_search"]
    }
  }'
```

## Code Examples

=== "Python"

    ```python
    import requests
    import json

    def send_message(message_content, model_ids):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            "model_ids": model_ids,
            "message": {"content": message_content}
        }
        
        data = {"message": json.dumps(message_data)}
        
        response = requests.post(
            "https://api.your-domain.com/conversation",
            headers=headers,
            data=data,
            stream=True
        )
        
        for line in response.iter_lines():
            if line:
                event = line.decode().replace("data: ", "")
                data = json.loads(event)
                
                if data["type"] == "message_content":
                    print(data["content"]["text"], end="", flush=True)
    
    send_message("Explain quantum physics", ["model-uuid"])
    ```

=== "JavaScript"

    ```javascript
    async function sendMessage(content, modelIds) {
      const formData = new FormData();
      formData.append('message', JSON.stringify({
        model_ids: modelIds,
        message: { content }
      }));

      const response = await fetch('https://api.your-domain.com/conversation', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_TOKEN'
        },
        body: formData
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');
        
        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = JSON.parse(line.slice(6));
            if (data.type === 'message_content') {
              process.stdout.write(data.content.text);
            }
          }
        }
      }
    }

    sendMessage('Explain quantum physics', ['model-uuid']);
    ```

## Error Responses

### 400 Bad Request

Invalid JSON in message field:

```json
{
  "detail": "Invalid JSON in message field"
}
```

### 401 Unauthorized

Missing or invalid authentication:

```json
{
  "detail": "Unauthorized: Invalid or missing API token"
}
```

### 500 Internal Server Error

Server error during processing:

```json
{
  "detail": "Failed to send message: <error details>"
}
```

## Best Practices

### 1. Handle Streaming Properly

Always handle the streaming response correctly in your client to avoid memory issues.

### 2. Process Events by Type

Different event types contain different data structures. Always check the `type` field.

### 3. Handle Errors Gracefully

The stream may contain error events. Check for the `error` type in events.

### 4. Set Appropriate Timeouts

Conversations can take time. Set timeouts of at least 60 seconds, preferably 300 seconds.

### 5. Use Multiple Models for Comparison

The API supports multiple models simultaneously. Use this for A/B testing or comparison.

## Limitations

- Maximum file size: 100MB per file
- Maximum files per request: 10
- Supported file types: images (jpg, png, gif, webp), documents (pdf, txt), audio (mp3, wav), video (mp4, mov)
- Conversation timeout: 5 minutes per request

## Next Steps

- Learn about [Tool Execution](tools.md)
- See [Examples](../examples/basic.md)
- Check [Authentication](../authentication.md) for security practices

