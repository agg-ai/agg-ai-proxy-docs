# Authentication

The AGG AI Proxy API uses Bearer token authentication for all protected endpoints.

## API Token

Your API token is a secret credential that identifies your application. Keep it secure and never expose it in client-side code.

### Getting Your Token

Contact your administrator or check your dashboard to obtain an API token.

### Using Your Token

Include the token in the `Authorization` header of every request:

```
Authorization: Bearer YOUR_API_TOKEN
```

## Example Requests

=== "curl"

    ```bash
    curl -X GET https://api.your-domain.com/tools \
      -H "Authorization: Bearer YOUR_API_TOKEN"
    ```

=== "Python"

    ```python
    import requests

    headers = {"Authorization": "Bearer YOUR_API_TOKEN"}
    response = requests.get(
        "https://api.your-domain.com/tools",
        headers=headers
    )
    ```

=== "JavaScript"

    ```javascript
    const response = await fetch('https://api.your-domain.com/tools', {
      headers: {
        'Authorization': 'Bearer YOUR_API_TOKEN'
      }
    });
    ```

=== "Go"

    ```go
    req, _ := http.NewRequest("GET", "https://api.your-domain.com/tools", nil)
    req.Header.Set("Authorization", "Bearer YOUR_API_TOKEN")
    resp, _ := http.DefaultClient.Do(req)
    ```

## Public Endpoints

The following endpoints do not require authentication:

- `GET /` - API information
- `GET /health` - Health check
- `GET /docs` - API documentation (Swagger UI)
- `GET /redoc` - API documentation (ReDoc)
- `GET /openapi.json` - OpenAPI specification

## Security Best Practices

### 1. Keep Tokens Secret

Never commit tokens to version control or expose them in client-side code.

!!! danger "Don't Do This"
    ```javascript
    // ❌ Never hardcode tokens in frontend code
    const API_TOKEN = "secret-token-123";
    ```

!!! success "Do This Instead"
    ```javascript
    // ✅ Use environment variables or secure backend calls
    const API_TOKEN = process.env.API_TOKEN;
    ```

### 2. Use Environment Variables

Store tokens in environment variables:

```bash
export AGG_AI_TOKEN="your-token-here"
```

### 3. Rotate Tokens Regularly

Periodically rotate your API tokens to minimize security risks.

### 4. Use HTTPS

Always use HTTPS when making API requests to prevent token interception.

### 5. Implement Rate Limiting

Implement your own rate limiting to prevent abuse of your token.

## Error Responses

### 401 Unauthorized

Returned when the token is missing or invalid:

```json
{
  "detail": "Unauthorized: Invalid or missing API token"
}
```

**Solutions:**
- Verify the token is correct
- Ensure the `Authorization` header is properly formatted
- Check that the token hasn't expired or been revoked

## Token Management

### Revoking Tokens

If you suspect your token has been compromised, contact your administrator immediately to revoke it.

### Multiple Tokens

You can request multiple tokens for different environments or applications:

- Development token
- Staging token
- Production token

This allows you to revoke specific tokens without affecting other environments.

## Testing Authentication

Test your authentication setup:

```bash
# Should succeed
curl -X GET https://api.your-domain.com/tools \
  -H "Authorization: Bearer YOUR_VALID_TOKEN"

# Should return 401
curl -X GET https://api.your-domain.com/tools \
  -H "Authorization: Bearer invalid-token"
```

## Next Steps

- Start with the [Quickstart Guide](quickstart.md)
- Learn about the [Responses API](api/responses.md)
- Explore [Tool Execution](api/tools.md)

