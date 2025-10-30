# Tool Execution Examples

Examples of executing various AI tools using the Tools API.

## Web Search

Search the web for current information:

=== "Python"

    ```python
    import requests

    def web_search(query):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": "web_search",
                "arguments": {"query": query},
                
            }
        )
        
        result = response.json()
        
        if result["error"]:
            print(f"Error: {result['error']['message']}")
            return
        
        for item in result["content"]["results"]:
            print(f"\nTitle: {item['title']}")
            print(f"Link: {item['href']}")
            print(f"Snippet: {item['snippet']}\n")

    web_search("latest AI developments 2024")
    ```

=== "JavaScript"

    ```javascript
    async function webSearch(query) {
      const response = await fetch('https://api.your-domain.com/tools/execute', {
        method: 'POST',
        headers: {
          'Authorization': 'Bearer YOUR_TOKEN',
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({
          tool_name: 'web_search',
          arguments: { query },
          model: ['your-model-uuid']
        })
      });

      const result = await response.json();

      if (result.error) {
        console.error('Error:', result.error.message);
        return;
      }

      result.content.results.forEach(item => {
        console.log(`\nTitle: ${item.title}`);
        console.log(`Link: ${item.href}`);
        console.log(`Snippet: ${item.snippet}\n`);
      });
    }

    webSearch('latest AI developments 2024');
    ```

## Image Generation

Generate images from text:

=== "Python"

    ```python
    import requests

    def generate_image(prompt, size="1024x1024"):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": "image_generation",
                "arguments": {
                    "prompt": prompt,
                    "n": 1,
                    "size": size
                },
                
            }
        )
        
        result = response.json()
        
        if result["error"]:
            print(f"Error: {result['error']['message']}")
            return
        
        for item in result["content"]["results"]:
            file_info = item["file"]
            print(f"Image URL: {file_info['url']}")
            print(f"Revised prompt: {item['revised_prompt']}")

    generate_image("A serene mountain landscape at sunset")
    ```

## Audio Transcription

Transcribe audio files:

=== "Python"

    ```python
    import requests

    def transcribe_audio(audio_url):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": "audio_transcription",
                "arguments": {"audio_url": audio_url},
                
            }
        )
        
        result = response.json()
        
        if result["error"]:
            print(f"Error: {result['error']['message']}")
            return
        
        content = result["content"]
        print(f"Transcription: {content['text']}")
        print(f"Language: {content.get('language_code', 'unknown')}")

    transcribe_audio("https://example.com/audio.mp3")
    ```

## Text to Speech

Convert text to speech:

=== "Python"

    ```python
    import requests

    def text_to_speech(text, voice="alloy"):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": "audio_tts",
                "arguments": {
                    "text": text,
                    "voice": voice
                },
                
            }
        )
        
        result = response.json()
        
        if result["error"]:
            print(f"Error: {result['error']['message']}")
            return
        
        file_info = result["content"]["file"]
        print(f"Audio URL: {file_info['url']}")
        print(f"File ID: {file_info['file_id']}")

    text_to_speech("Hello, this is a test of text to speech.")
    ```

## Video Analysis

Analyze video content:

=== "Python"

    ```python
    import requests

    def analyze_video(video_url, question=None):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        arguments = {"video_url": video_url}
        if question:
            arguments["question"] = question
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": "video_analysis",
                "arguments": arguments,
                
            }
        )
        
        result = response.json()
        
        if result["error"]:
            print(f"Error: {result['error']['message']}")
            return
        
        print(f"Analysis: {result['content']['analysis']}")

    analyze_video(
        "https://example.com/video.mp4",
        "What are the main topics discussed?"
    )
    ```

## Batch Tool Execution

Execute multiple tools in sequence:

=== "Python"

    ```python
    import requests

    def execute_tool(tool_name, arguments, model):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": tool_name,
                "arguments": arguments,
                "model": model
            }
        )
        
        return response.json()

    def research_topic(topic):
        model = ["your-model-uuid"]
        
        # Step 1: Web search
        print("Searching the web...")
        web_results = execute_tool(
            "web_search",
            {"query": topic},
            model
        )
        
        if web_results["error"]:
            print(f"Web search failed: {web_results['error']['message']}")
            return
        
        print(f"Found {len(web_results['content']['results'])} results")
        
        # Step 2: Wikipedia search
        print("\nSearching Wikipedia...")
        wiki_results = execute_tool(
            "wikipedia_search",
            {"query": topic},
            model
        )
        
        if wiki_results["error"]:
            print(f"Wikipedia search failed: {wiki_results['error']['message']}")
            return
        
        print(f"Found {len(wiki_results['content']['results'])} Wikipedia articles")
        
        # Combine results
        print(f"\n{'='*60}")
        print(f"Research Results for: {topic}")
        print(f"{'='*60}\n")
        
        print("Web Results:")
        for item in web_results["content"]["results"][:3]:
            print(f"  - {item['title']}: {item['href']}")
        
        print("\nWikipedia Results:")
        for item in wiki_results["content"]["results"][:3]:
            print(f"  - {item['title']}: {item['href']}")

    research_topic("quantum computing")
    ```

## Tool with Error Handling

Robust tool execution with retry logic:

=== "Python"

    ```python
    import requests
    import time

    def execute_tool_with_retry(tool_name, arguments, model, max_retries=3):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        for attempt in range(max_retries):
            try:
                response = requests.post(
                    "https://api.your-domain.com/tools/execute",
                    headers=headers,
                    json={
                        "tool_name": tool_name,
                        "arguments": arguments,
                        "model": model
                    },
                    timeout=60
                )
                
                response.raise_for_status()
                result = response.json()
                
                if result["error"]:
                    print(f"Tool error: {result['error']['message']}")
                    return None
                
                return result["content"]
                
            except requests.exceptions.Timeout:
                print(f"Attempt {attempt + 1} timed out")
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)  # Exponential backoff
            except requests.exceptions.RequestException as e:
                print(f"Request failed: {e}")
                if attempt < max_retries - 1:
                    time.sleep(2 ** attempt)
        
        print(f"Failed after {max_retries} attempts")
        return None

    # Use it
    result = execute_tool_with_retry(
        "web_search",
        {"query": "machine learning"},
        ["your-model-uuid"]
    )

    if result:
        print("Success!")
        print(result)
    ```

## Parallel Tool Execution

Execute multiple tools in parallel:

=== "Python"

    ```python
    import requests
    from concurrent.futures import ThreadPoolExecutor, as_completed

    def execute_tool(tool_name, arguments, model):
        headers = {"Authorization": "Bearer YOUR_TOKEN"}
        
        response = requests.post(
            "https://api.your-domain.com/tools/execute",
            headers=headers,
            json={
                "tool_name": tool_name,
                "arguments": arguments,
                "model": model
            }
        )
        
        return tool_name, response.json()

    def parallel_research(query):
        model = ["your-model-uuid"]
        
        tasks = [
            ("web_search", {"query": query}),
            ("wikipedia_search", {"query": query}),
        ]
        
        results = {}
        
        with ThreadPoolExecutor(max_workers=3) as executor:
            futures = {
                executor.submit(execute_tool, tool, args, model): tool
                for tool, args in tasks
            }
            
            for future in as_completed(futures):
                tool_name, result = future.result()
                results[tool_name] = result
        
        return results

    # Execute searches in parallel
    results = parallel_research("artificial intelligence")

    for tool_name, result in results.items():
        print(f"\n{tool_name}:")
        if result["error"]:
            print(f"  Error: {result['error']['message']}")
        else:
            print(f"  Success! {len(result['content']['results'])} results")
    ```

## Next Steps

- Review [Basic Examples](basic.md) for conversation patterns
- Check the [Tools API Reference](../api/tools.md) for all available tools
- Learn about [Authentication](../authentication.md) best practices

