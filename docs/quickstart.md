# Quickstart 

Get started with the AGG AI Proxy API in just a few minutes.

## Prerequisites

- An API token (contact your administrator or check your dashboard)
- A tool to make HTTP requests (curl, Postman, or your favorite programming language)

## Step 1: Verify Access

Test your API token with a simple health check:

```bash
curl https://api.your-domain.com/health
```

Expected response:
```json
{
  "status": "healthy"
}
```

## Step 2: List Available Tools

Get a list of all available AI tools:

```bash
curl -X GET https://api.your-domain.com/tools \
  -H "Authorization: Bearer YOUR_API_TOKEN"
```

Expected response:
```json
[
  {
    "name": "web_search",
    "description": "Search the web for current information",
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
  },
  ...
]
```

## Step 3: Execute a Tool

Try executing the web search tool:

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "web_search",
    "arguments": {
      "query": "artificial intelligence trends 2024"
    },
    "model": {"model_id": "model-uuid"}
  }'
```

## Step 4: Start a Conversation

Send a message to an AI model:

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -F 'message={
    "model": {"model_id": "model-uuid"},
    "message": {
      "content": "Explain quantum computing in simple terms"
    }
  }'
```

The response will be a server-sent events stream. Here's how to handle it in different languages:

=== "Python"

    ```python
    import requests
    import json

    headers = {"Authorization": "Bearer YOUR_API_TOKEN"}
    
    data = {
        "message": json.dumps({
            "model": {"model_id": "model-uuid"},
            "message": {"content": "Explain quantum computing"}
        })
    }
    
    response = requests.post(
        "https://api.your-domain.com/conversation",
        headers=headers,
        data=data,
        stream=True
    )
    
    for line in response.iter_lines():
        if line:
            print(line.decode())
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('https://api.your-domain.com/conversation', {
      method: 'POST',
      headers: {
        'Authorization': 'Bearer YOUR_API_TOKEN',
      },
      body: formData
    });

    const reader = response.body.getReader();
    const decoder = new TextDecoder();

    while (true) {
      const { done, value } = await reader.read();
      if (done) break;
      console.log(decoder.decode(value));
    }
    ```

=== "Go"

    ```go
    req, _ := http.NewRequest("POST", "https://api.your-domain.com/conversation", body)
    req.Header.Set("Authorization", "Bearer YOUR_API_TOKEN")
    
    resp, _ := http.DefaultClient.Do(req)
    defer resp.Body.Close()
    
    scanner := bufio.NewScanner(resp.Body)
    for scanner.Scan() {
        fmt.Println(scanner.Text())
    }
    ```

## Step 5: Upload Files with Messages

Send a message with file attachments:

```bash
curl -X POST https://api.your-domain.com/conversation \
  -H "Authorization: Bearer YOUR_API_TOKEN" \
  -F 'message={
    "model": {"model_id": "model-uuid"},
    "message": {
      "content": "What is in this image?"
    }
  }' \
  -F 'files=@/path/to/image.jpg'
```

## Next Steps

- Learn about [Authentication](authentication.md) best practices
- Explore the [Responses API](api/responses.md) in detail
- Check out [Tools API](api/tools.md) for all available tools
- See more [Examples](examples/basic.md)

## Common Issues

### 401 Unauthorized

Make sure your API token is correct and included in the `Authorization` header:

```
Authorization: Bearer YOUR_API_TOKEN
```

### 500 Internal Server Error

Check that you have provided valid `model_ids`. Contact support if the issue persists.

### Connection Timeout

For long-running operations, ensure your client supports streaming responses and has appropriate timeout settings.

