# Tools API

The Tools API allows you to discover and execute specialized AI tools.

## Endpoints

### List Tools

```
GET /tools
```

Get a list of all available tools and their schemas.

#### Response

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
  }
]
```

### Execute Tool

```
POST /tools/execute
```

Execute a specific tool with provided arguments.

#### Request Body

**Basic Example (without attachments):**

```json
{
  "tool_name": "web_search",
  "arguments": {
    "query": "artificial intelligence"
  },
  "model": {
    "model_id": "model-uuid",
    "purpose": "image_gen"
  }
}
```

**Example with Attachments (for image_edit, video_analysis, audio_transcription):**

```json
{
  "tool_name": "audio_transcription",
  "arguments": {},
  "model": {
    "model_id": "model-uuid",
    "purpose": "audio_transcription"
  },
  "attachments": [
    "file-uuid-1",
    "https://example.com/audio.mp3"
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `tool_name` | string | Yes | Name of the tool to execute |
| `arguments` | object | Yes | Tool-specific arguments |
| `model` | object | No | Model configuration. If not provided, uses resolved settings |
| `model.model_id` | string | Yes (if model provided) | UUID of the model to use |
| `model.purpose` | string | No | Model purpose: `audio`, `chat_transcription`, `audio_transcription`, `audio_tts`, `image_gen`, `video_understanding`, `video_gen`, `terminal` |
| `attachments` | array[string] | No | File references (UUIDs or URLs). System auto-detects type: file UUIDs (from `/files/upload-many`) or external URLs (downloaded automatically) |

#### Response

```json
{
  "content": {
    "type": "web_search",
    "results": [
      {
        "title": "Example Result",
        "href": "https://example.com",
        "snippet": "Result snippet...",
        "display_link": "example.com"
      }
    ]
  },
  "error": null
}
```

## Available Tools

### web_search

Search the web for current information.

**Arguments:**
- `query` (string, required): The search query

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "web_search",
    "arguments": {"query": "latest AI developments"}
  }'
```

Or with a specific model:

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "web_search",
    "arguments": {"query": "latest AI developments"},
    "model": {
      "model_id": "model-uuid"
    }
  }'
```

### wikipedia_search

Search Wikipedia for articles.

**Arguments:**
- `query` (string, required): The search query

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "wikipedia_search",
    "arguments": {"query": "quantum computing"},
    "model": {"model_id": "model-uuid"}
  }'
```

### image_generation

Generate images from text descriptions.

**Arguments:**
- `prompt` (string, required): The image description
- `n` (integer, optional): Number of images to generate (default: 1)
- `size` (string, optional): Image size (e.g., "1024x1024")

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "image_generation",
    "arguments": {
      "prompt": "A serene lake at sunset",
      "n": 1,
      "size": "1024x1024"
    },
    "model": {"model_id": "model-uuid"}
  }'
```

### image_edit

Edit existing images with AI. Images can be provided via the `attachments` field using either file UUIDs (pre-uploaded) or external URLs.

**Arguments:**
- `prompt` (string, required): Description of the edit

**Attachments:**
- Provide file UUIDs or URLs in the `attachments` array
- The system automatically detects the type

**Example with pre-uploaded file:**

```bash
# First, upload the image
curl -X POST https://api.your-domain.com/files/upload-many \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "files=@image.jpg"

# Use the file_id from the response
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "image_edit",
    "arguments": {
      "prompt": "Add a rainbow in the sky"
    },
    "model": {"model_id": "model-uuid", "purpose": "image_gen"},
    "attachments": ["file-uuid-from-upload"]
  }'
```

**Example with external URL:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "image_edit",
    "arguments": {
      "prompt": "Add a rainbow in the sky"
    },
    "model": {"model_id": "model-uuid", "purpose": "image_gen"},
    "attachments": ["https://example.com/image.jpg"]
  }'
```

**Example mixing both:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "image_edit",
    "arguments": {
      "prompt": "Add a rainbow in the sky"
    },
    "model": {"model_id": "model-uuid", "purpose": "image_gen"},
    "attachments": [
      "file-uuid-from-upload",
      "https://example.com/another-image.jpg"
    ]
  }'
```

### video_analysis

Analyze video content. Videos can be provided via the `attachments` field using either file UUIDs (pre-uploaded) or external URLs.

**Arguments:**
- `question` (string, optional): Specific question about the video

**Attachments:**
- Provide file UUIDs or URLs in the `attachments` array
- The system automatically detects the type

**Example with pre-uploaded file:**

```bash
# First, upload the video
curl -X POST https://api.your-domain.com/files/upload-many \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "files=@video.mp4"

# Use the file_id from the response
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "video_analysis",
    "arguments": {
      "question": "What are the main topics discussed?"
    },
    "model": {"model_id": "model-uuid", "purpose": "video_understanding"},
    "attachments": ["file-uuid-from-upload"]
  }'
```

**Example with external URL:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "video_analysis",
    "arguments": {
      "question": "What are the main topics discussed?"
    },
    "model": {"model_id": "model-uuid", "purpose": "video_understanding"},
    "attachments": ["https://example.com/video.mp4"]
  }'
```

### video_generation

Generate videos from text descriptions.

**Arguments:**
- `prompt` (string, required): Video description
- `duration` (integer, optional): Video duration in seconds

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "video_generation",
    "arguments": {
      "prompt": "A time-lapse of a flower blooming",
      "duration": 5
    },
    "model": {"model_id": "model-uuid"}
  }'
```

### audio_transcription

Transcribe audio files to text. Audio can be provided via the `attachments` field using either file UUIDs (pre-uploaded) or external URLs.

**Arguments:**
- No required arguments (audio is provided via attachments)

**Attachments:**
- Provide file UUIDs or URLs in the `attachments` array
- The system automatically detects the type

**Example with pre-uploaded file:**

```bash
# First, upload the audio file
curl -X POST https://api.your-domain.com/files/upload-many \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "files=@audio.mp3"

# Use the file_id from the response
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "audio_transcription",
    "arguments": {},
    "model": {"model_id": "model-uuid", "purpose": "audio_transcription"},
    "attachments": ["file-uuid-from-upload"]
  }'
```

**Example with external URL:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "audio_transcription",
    "arguments": {},
    "model": {"model_id": "model-uuid", "purpose": "audio_transcription"},
    "attachments": ["https://example.com/audio.mp3"]
  }'
```

### audio_tts

Convert text to speech.

**Arguments:**
- `text` (string, required): Text to convert
- `voice` (string, optional): Voice to use (e.g., "alloy", "nova")

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "audio_tts",
    "arguments": {
      "text": "Hello, world!",
      "voice": "alloy"
    },
    "model": {"model_id": "model-uuid"}
  }'
```

### terminal

Execute safe terminal commands (sandboxed).

**Arguments:**
- `command` (string, required): Command to execute

!!! warning
    Terminal commands are executed in a sandboxed environment with restricted permissions.

**Example:**

```bash
curl -X POST https://api.your-domain.com/tools/execute \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -H "Content-Type: application/json" \
  -d '{
    "tool_name": "terminal",
    "arguments": {
      "command": "ls -la"
    },
    "model": {"model_id": "model-uuid"}
  }'
```

## Code Examples

=== "Python"

    ```python
    import requests

    def execute_tool(tool_name, arguments, model=None):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        payload = {
            "tool_name": tool_name,
            "arguments": arguments
        }
        if model:
            payload["model"] = model
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json=payload
        )
        
        return response.json()

    # Web search example (uses resolved settings)
    result = execute_tool(
        "web_search",
        {"query": "latest AI news"}
    )
    
    # Or with specific model
    result_with_model = execute_tool(
        "web_search",
        {"query": "latest AI news"},
        model={"model_id": "model-uuid", "purpose": "image_gen"}
    )
    
    print(result["content"]["results"])
    ```

=== "JavaScript"

    ```javascript
    async function executeTool(toolName, arguments, model = null) {
      const payload = {
        tool_name: toolName,
        arguments: arguments
      };
      if (model) {
        payload.model = model;
      }
      
      const response = await fetch('https://api.your-domain.com/tools/execute', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_TOKEN',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify(payload)
      });

      return await response.json();
    }

    // Web search example (uses resolved settings)
    const result = await executeTool(
      'web_search',
      { query: 'latest AI news' }
    );

    // Or with specific model
    const resultWithModel = await executeTool(
      'web_search',
      { query: 'latest AI news' },
      { model_id: 'model-uuid', purpose: 'image_gen' }
    );

    console.log(result.content.results);
    ```

=== "Go"

    ```go
    type ModelConfig struct {
        ModelID string  `json:"model_id"`
        Purpose *string `json:"purpose,omitempty"`
    }
    
    type ToolRequest struct {
        ToolName  string                 `json:"tool_name"`
        Arguments map[string]interface{} `json:"arguments"`
        Model     *ModelConfig           `json:"model,omitempty"`
    }

    func executeTool(toolName string, arguments map[string]interface{}, model *ModelConfig) (map[string]interface{}, error) {
        req := ToolRequest{
            ToolName:  toolName,
            Arguments: arguments,
            Model:     model,
        }
        
        body, _ := json.Marshal(req)
        
        httpReq, _ := http.NewRequest("POST", "https://api.your-domain.com/tools/execute", bytes.NewBuffer(body))
        httpReq.Header.Set("Authorization", "Bearer YOUR_TOKEN")
        httpReq.Header.Set("Content-Type", "application/json")
        
        client := &http.Client{}
        resp, err := client.Do(httpReq)
        if err != nil {
            return nil, err
        }
        defer resp.Body.Close()
        
        var result map[string]interface{}
        json.NewDecoder(resp.Body).Decode(&result)
        
        return result, nil
    }
    
    // Example usage with resolved settings
    result, _ := executeTool("web_search", map[string]interface{}{"query": "AI news"}, nil)
    
    // Or with specific model
    purpose := "image_gen"
    resultWithModel, _ := executeTool("web_search", 
        map[string]interface{}{"query": "AI news"},
        &ModelConfig{ModelID: "model-uuid", Purpose: &purpose})
    ```

## Error Handling

### Tool Not Found

```json
{
  "content": null,
  "error": {
    "message": "Function 'invalid_tool' not found"
  }
}
```

### Invalid Arguments

```json
{
  "content": null,
  "error": {
    "message": "Required argument 'query' is missing"
  }
}
```

### Execution Error

```json
{
  "content": null,
  "error": {
    "message": "Tool execution failed: <error details>"
  }
}
```

## Best Practices

### 1. Check Tool Schema First

Always call `GET /tools` first to understand the required arguments for each tool.

### 2. Handle Errors Gracefully

Check the `error` field in responses and handle failures appropriately.

### 3. Use Appropriate Models

Different tools may work better with specific models. Test to find the best combination.

### 4. Cache Tool Schemas

Tool schemas don't change often. Cache them to reduce API calls.

### 5. Validate Arguments Locally

Validate arguments against the tool schema before making API calls to catch errors early.

## Limitations

- Maximum execution time per tool: 5 minutes
- Some tools may have rate limits (e.g., web search)
- File size limits apply to media processing tools

## Next Steps

- Learn about the [Conversation API](conversation.md)
- See [Tool Execution Examples](../examples/tools.md)
- Check [Authentication](../authentication.md) best practices

