# Basic Usage Examples

Common patterns and examples for using the AGG AI Proxy API.

## Simple Question-Answer

Ask a simple question and get a response:

=== "Python"

    ```python
    import requests
    import json

    API_TOKEN = "your-token"
    MODEL_ID = "your-model-uuid"

    def ask_question(question):
        headers = {"Authorization": f"Bearer {API_TOKEN}"}
        
        message_data = {
            "model": [MODEL_ID],
            "message": {"content": question}
        }
        
        response = requests.post(
            "https://api.your-domain.com/conversation",
            headers=headers,
            data={"message": json.dumps(message_data)},
            stream=True
        )
        
        full_response = ""
        for line in response.iter_lines():
            if line:
                event = line.decode().replace("data: ", "")
                try:
                    data = json.loads(event)
                    if data["type"] == "message_content":
                        text = data["content"]["text"]
                        full_response += text
                        print(text, end="", flush=True)
                except:
                    pass
        
        return full_response

    answer = ask_question("What is the capital of France?")
    ```

=== "JavaScript"

    ```javascript
    const API_TOKEN = 'your-token';
    const MODEL_ID = 'your-model-uuid';

    async function askQuestion(question) {
      const formData = new FormData();
      formData.append('message', JSON.stringify({
        model: [MODEL_ID],
        message: { content: question }
      }));

      const response = await fetch('https://api.your-domain.com/conversation', {
        method: 'POST',
        headers: { 'Authorization': `Bearer ${API_TOKEN}` },
        body: formData
      });

      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let fullResponse = '';

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;
        
        const chunk = decoder.decode(value);
        const lines = chunk.split('\n');
        
        for (const line of lines) {
          if (line.startsWith('data: ')) {
            try {
              const data = JSON.parse(line.slice(6));
              if (data.type === 'message_content') {
                const text = data.content.text;
                fullResponse += text;
                process.stdout.write(text);
              }
            } catch (e) {}
          }
        }
      }

      return fullResponse;
    }

    askQuestion('What is the capital of France?');
    ```

## Image Analysis

Upload and analyze an image:

=== "Python"

    ```python
    import requests
    import json

    def analyze_image(image_path, question):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            ,
            "message": {"content": question}
        }
        
        with open(image_path, 'rb') as f:
            files = {'files': f}
            data = {'message': json.dumps(message_data)}
            
            response = requests.post(
                "https://api.your-domain.com/conversation",
                headers=headers,
                data=data,
                files=files,
                stream=True
            )
            
            for line in response.iter_lines():
                if line:
                    event = line.decode().replace("data: ", "")
                    try:
                        data = json.loads(event)
                        if data["type"] == "message_content":
                            print(data["content"]["text"], end="", flush=True)
                    except:
                        pass

    analyze_image("photo.jpg", "What is in this image?")
    ```

=== "JavaScript"

    ```javascript
    async function analyzeImage(imagePath, question) {
      const formData = new FormData();
      
      const fileBlob = await fetch(imagePath).then(r => r.blob());
      formData.append('files', fileBlob, 'image.jpg');
      
      formData.append('message', JSON.stringify({
        model: ['your-model-uuid'],
        message: { content: question }
      }));

      const response = await fetch('https://api.your-domain.com/conversation', {
        method: 'POST',
        headers: { 'Authorization': 'Bearer YOUR_TOKEN' },
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
            try {
              const data = JSON.parse(line.slice(6));
              if (data.type === 'message_content') {
                process.stdout.write(data.content.text);
              }
            } catch (e) {}
          }
        }
      }
    }

    analyzeImage('photo.jpg', 'What is in this image?');
    ```

## Multiple Models Comparison

Compare responses from multiple models:

=== "Python"

    ```python
    import requests
    import json
    from collections import defaultdict

    def compare_models(question, model):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            "model": model,
            "message": {"content": question}
        }
        
        response = requests.post(
            "https://api.your-domain.com/conversation",
            headers=headers,
            data={"message": json.dumps(message_data)},
            stream=True
        )
        
        responses_by_model = defaultdict(str)
        model_names = {}
        
        for line in response.iter_lines():
            if line:
                event = line.decode().replace("data: ", "")
                try:
                    data = json.loads(event)
                    
                    if data["type"] == "message_start":
                        model_id = data["message"]["model_id"]
                        model_names[model_id] = data["message"]["model_name"]
                    
                    elif data["type"] == "message_content":
                        model_id = data["model_id"]
                        text = data["content"]["text"]
                        responses_by_model[model_id] += text
                except:
                    pass
        
        for model_id, response_text in responses_by_model.items():
            print(f"\n{'='*60}")
            print(f"Model: {model_names.get(model_id, model_id)}")
            print(f"{'='*60}")
            print(response_text)

    compare_models(
        "Explain quantum computing",
        ["model-uuid-1", "model-uuid-2"]
    )
    ```

## With Tool Usage

Enable specific tools for a conversation:

=== "Python"

    ```python
    import requests
    import json

    def ask_with_tools(question, tools):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            ,
            "message": {"content": question},
            "options": {
                "tools": tools
            }
        }
        
        response = requests.post(
            "https://api.your-domain.com/conversation",
            headers=headers,
            data={"message": json.dumps(message_data)},
            stream=True
        )
        
        for line in response.iter_lines():
            if line:
                event = line.decode().replace("data: ", "")
                try:
                    data = json.loads(event)
                    
                    if data["type"] == "tool_calls":
                        print(f"\n[Using tools: {data['tool_calls']}]")
                    
                    elif data["type"] == "message_content":
                        print(data["content"]["text"], end="", flush=True)
                except:
                    pass

    ask_with_tools(
        "What are the latest developments in AI?",
        ["web_search"]
    )
    ```

## Streaming with Progress Indication

Show progress while streaming:

=== "Python"

    ```python
    import requests
    import json
    import sys

    def chat_with_progress(message):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            ,
            "message": {"content": message}
        }
        
        response = requests.post(
            "https://api.your-domain.com/conversation",
            headers=headers,
            data={"message": json.dumps(message_data)},
            stream=True
        )
        
        print("Waiting for response", end="", flush=True)
        started = False
        
        for line in response.iter_lines():
            if line:
                event = line.decode().replace("data: ", "")
                try:
                    data = json.loads(event)
                    
                    if data["type"] == "message_start" and not started:
                        print("\n\nResponse:\n")
                        started = True
                    
                    elif data["type"] == "message_content":
                        print(data["content"]["text"], end="", flush=True)
                    
                    elif data["type"] == "compression_started":
                        print("\n[Compressing conversation history...]", flush=True)
                    
                    elif data["type"] == "compression_completed":
                        print("[Compression complete]\n", flush=True)
                        
                except:
                    if not started:
                        print(".", end="", flush=True)

    chat_with_progress("Explain relativity theory")
    ```

## Error Handling

Proper error handling for API calls:

=== "Python"

    ```python
    import requests
    import json

    def safe_chat(message):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        message_data = {
            ,
            "message": {"content": message}
        }
        
        try:
            response = requests.post(
                "https://api.your-domain.com/conversation",
                headers=headers,
                data={"message": json.dumps(message_data)},
                stream=True,
                timeout=300
            )
            
            response.raise_for_status()
            
            for line in response.iter_lines():
                if line:
                    event = line.decode().replace("data: ", "")
                    try:
                        data = json.loads(event)
                        
                        if data["type"] == "error":
                            print(f"Error: {data['error']}")
                            return None
                        
                        elif data["type"] == "message_content":
                            print(data["content"]["text"], end="", flush=True)
                    except:
                        pass
                        
        except requests.exceptions.Timeout:
            print("Request timed out")
        except requests.exceptions.HTTPError as e:
            print(f"HTTP Error: {e}")
        except requests.exceptions.RequestException as e:
            print(f"Request failed: {e}")

    safe_chat("Hello, how are you?")
    ```

## Next Steps

- Learn about [Tool Execution](tools.md)
- Check out the [Conversation API](../api/conversation.md) reference
- Review [Authentication](../authentication.md) best practices

