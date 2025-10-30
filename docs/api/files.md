# Files API

The Files API allows you to upload files that can later be used with tool execution requests.

## Endpoints

### Upload Multiple Files

```
POST /files/upload-many
```

Upload one or more files to use with tools that require file inputs (e.g., image_edit, video_analysis, audio_transcription).

#### Request

Multipart form-data with files:

```bash
curl -X POST https://api.your-domain.com/files/upload-many \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "files=@image.jpg" \
  -F "files=@audio.mp3"
```

#### Response

```json
{
  "files": [
    {
      "file_id": "550e8400-e29b-41d4-a716-446655440000",
      "filename": "image.jpg",
      "content_type": "image/jpeg",
      "status": "completed",
      "error_message": null
    },
    {
      "file_id": "660e8400-e29b-41d4-a716-446655440001",
      "filename": "audio.mp3",
      "content_type": "audio/mpeg",
      "status": "completed",
      "error_message": null
    }
  ],
  "total_count": 2
}
```

| Field | Type | Description |
|-------|------|-------------|
| `files` | array | Array of uploaded file objects |
| `files[].file_id` | string | UUID of the uploaded file |
| `files[].filename` | string | Original filename |
| `files[].content_type` | string | MIME type of the file |
| `files[].status` | string | Status: `in_progress`, `completed`, or `failed` |
| `files[].error_message` | string\|null | Error message if processing failed |
| `total_count` | integer | Total number of files uploaded |

### Get File Status

```
GET /files/{file_id}/status
```

Check the status of an uploaded file.

#### Parameters

| Parameter | Type | Location | Description |
|-----------|------|----------|-------------|
| `file_id` | string | path | The UUID of the file |

#### Response

```json
{
  "file_id": "550e8400-e29b-41d4-a716-446655440000",
  "filename": "image.jpg",
  "content_type": "image/jpeg",
  "status": "completed",
  "error_message": null
}
```

## Usage Examples

### Python

**Upload files and use with tool execution:**

```python
import requests

API_URL = "https://api.your-domain.com"
API_TOKEN = "your_token_here"

headers = {"Authorization": f"Bearer {API_TOKEN}"}

# Step 1: Upload files
with open("image.jpg", "rb") as f1, open("audio.mp3", "rb") as f2:
    files = [
        ("files", ("image.jpg", f1, "image/jpeg")),
        ("files", ("audio.mp3", f2, "audio/mpeg"))
    ]
    response = requests.post(f"{API_URL}/files/upload-many", headers=headers, files=files)
    upload_result = response.json()

# Extract file IDs
file_ids = [file["file_id"] for file in upload_result["files"]]
print(f"Uploaded files: {file_ids}")

# Step 2: Use files with tool execution
response = requests.post(
    f"{API_URL}/tools/execute",
    headers={**headers, "Content-Type": "application/json"},
    json={
        "tool_name": "audio_transcription",
        "arguments": {},
        "attachment_ids": [file_ids[1]]  # Use the audio file
    }
)

result = response.json()
print(result)
```

**Check file status:**

```python
import requests

API_URL = "https://api.your-domain.com"
API_TOKEN = "your_token_here"

headers = {"Authorization": f"Bearer {API_TOKEN}"}
file_id = "550e8400-e29b-41d4-a716-446655440000"

response = requests.get(f"{API_URL}/files/{file_id}/status", headers=headers)
status = response.json()

print(f"File status: {status['status']}")
if status['status'] == 'failed':
    print(f"Error: {status['error_message']}")
```

### JavaScript

**Upload files and use with tool execution:**

```javascript
const API_URL = 'https://api.your-domain.com';
const API_TOKEN = 'your_token_here';

async function uploadAndTranscribe() {
  // Step 1: Upload file
  const formData = new FormData();
  const fileInput = document.querySelector('input[type="file"]');
  
  for (const file of fileInput.files) {
    formData.append('files', file);
  }
  
  const uploadResponse = await fetch(`${API_URL}/files/upload-many`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_TOKEN}`
    },
    body: formData
  });
  
  const uploadResult = await uploadResponse.json();
  const fileIds = uploadResult.files.map(f => f.file_id);
  
  console.log('Uploaded files:', fileIds);
  
  // Step 2: Execute tool with uploaded files
  const toolResponse = await fetch(`${API_URL}/tools/execute`, {
    method: 'POST',
    headers: {
      'Authorization': `Bearer ${API_TOKEN}`,
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      tool_name: 'audio_transcription',
      arguments: {},
      attachment_ids: fileIds
    })
  });
  
  const result = await toolResponse.json();
  console.log(result);
}
```

**Check file status:**

```javascript
const API_URL = 'https://api.your-domain.com';
const API_TOKEN = 'your_token_here';

async function checkFileStatus(fileId) {
  const response = await fetch(`${API_URL}/files/${fileId}/status`, {
    method: 'GET',
    headers: {
      'Authorization': `Bearer ${API_TOKEN}`
    }
  });
  
  const status = await response.json();
  console.log('File status:', status.status);
  
  if (status.status === 'failed') {
    console.error('Error:', status.error_message);
  }
  
  return status;
}
```

### cURL

**Upload files:**

```bash
curl -X POST https://api.your-domain.com/files/upload-many \
  -H "Authorization: Bearer YOUR_TOKEN" \
  -F "files=@/path/to/image.jpg" \
  -F "files=@/path/to/audio.mp3"
```

**Check file status:**

```bash
curl -X GET https://api.your-domain.com/files/550e8400-e29b-41d4-a716-446655440000/status \
  -H "Authorization: Bearer YOUR_TOKEN"
```

### Go

**Upload files and use with tool execution:**

```go
package main

import (
    "bytes"
    "encoding/json"
    "fmt"
    "io"
    "mime/multipart"
    "net/http"
    "os"
)

const (
    APIURL   = "https://api.your-domain.com"
    APIToken = "your_token_here"
)

type FileResponse struct {
    FileID       string  `json:"file_id"`
    Filename     string  `json:"filename"`
    ContentType  string  `json:"content_type"`
    Status       string  `json:"status"`
    ErrorMessage *string `json:"error_message"`
}

type UploadResponse struct {
    Files      []FileResponse `json:"files"`
    TotalCount int           `json:"total_count"`
}

func uploadFiles(filePaths []string) (*UploadResponse, error) {
    var buf bytes.Buffer
    writer := multipart.NewWriter(&buf)
    
    for _, path := range filePaths {
        file, err := os.Open(path)
        if err != nil {
            return nil, err
        }
        defer file.Close()
        
        part, err := writer.CreateFormFile("files", path)
        if err != nil {
            return nil, err
        }
        
        if _, err := io.Copy(part, file); err != nil {
            return nil, err
        }
    }
    
    if err := writer.Close(); err != nil {
        return nil, err
    }
    
    req, err := http.NewRequest("POST", APIURL+"/files/upload-many", &buf)
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Authorization", "Bearer "+APIToken)
    req.Header.Set("Content-Type", writer.FormDataContentType())
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var result UploadResponse
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }
    
    return &result, nil
}

func main() {
    // Upload files
    result, err := uploadFiles([]string{"image.jpg", "audio.mp3"})
    if err != nil {
        fmt.Printf("Error uploading files: %v\n", err)
        return
    }
    
    fmt.Printf("Uploaded %d files\n", result.TotalCount)
    for _, file := range result.Files {
        fmt.Printf("- %s (ID: %s)\n", file.Filename, file.FileID)
    }
}
```

**Check file status:**

```go
func getFileStatus(fileID string) (*FileResponse, error) {
    req, err := http.NewRequest("GET", APIURL+"/files/"+fileID+"/status", nil)
    if err != nil {
        return nil, err
    }
    
    req.Header.Set("Authorization", "Bearer "+APIToken)
    
    client := &http.Client{}
    resp, err := client.Do(req)
    if err != nil {
        return nil, err
    }
    defer resp.Body.Close()
    
    var result FileResponse
    if err := json.NewDecoder(resp.Body).Decode(&result); err != nil {
        return nil, err
    }
    
    return &result, nil
}
```

## Workflow

1. **Upload files** using `/files/upload-many`
2. **Get file IDs** from the response
3. **Use file IDs** in tool execution requests via `attachment_ids` parameter
4. **Check status** (if needed) using `/files/{file_id}/status`

## File Types

The API supports various file types:

- **Images**: `image/jpeg`, `image/png`, `image/gif`, `image/webp`
- **Audio**: `audio/mpeg`, `audio/wav`, `audio/ogg`, `audio/m4a`
- **Video**: `video/mp4`, `video/quicktime`, `video/webm`
- **Documents**: `application/pdf`, `text/plain`

## File Status

Files can have three status values:

- **`in_progress`**: File is being processed (for async operations)
- **`completed`**: File is ready to use
- **`failed`**: Processing failed (check `error_message`)

Most files are immediately available (`completed`), but some operations may require async processing.

