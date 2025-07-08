# Ollama Headers Support Example

This example demonstrates how to use custom headers with the Ollama provider in Continue, which is useful for:
- Adding authentication to secured Ollama endpoints
- Custom reverse proxy configurations  
- Additional request metadata

## Configuration

### Using requestOptions.headers

Add custom headers to your Continue configuration:

```yaml
models:
  - name: "Llama 3.1 8B"
    provider: ollama
    model: llama3.1:8b
    apiBase: "https://ollama.mydomain.com:11434"
    apiKey: "your-api-key"  # Optional: becomes Authorization: Bearer header
    requestOptions:
      headers:
        Authorization: "Bearer your-custom-token"
        X-Custom-Header: "custom-value"
        X-API-Version: "v1"

  - name: "Secured Ollama"
    provider: ollama  
    model: llama2:7b
    apiBase: "https://secured-ollama.company.com"
    requestOptions:
      headers:
        Authorization: "Basic dXNlcjpwYXNz"  # base64 encoded user:pass
        X-Forwarded-For: "192.168.1.100"
        Custom-Auth-Token: "abc123xyz"
```

### JSON Configuration

```json
{
  "models": [
    {
      "name": "Llama 3.1 8B",
      "provider": "ollama",
      "model": "llama3.1:8b", 
      "apiBase": "https://ollama.mydomain.com:11434",
      "requestOptions": {
        "headers": {
          "Authorization": "Bearer your-api-key",
          "X-Custom-Header": "custom-value"
        }
      }
    }
  ]
}
```

## Use Cases

### 1. Reverse Proxy Authentication

When using Ollama behind a reverse proxy that requires authentication:

```yaml
requestOptions:
  headers:
    Authorization: "Bearer proxy-auth-token"
    X-Proxy-Auth: "additional-auth-header"
```

### 2. Custom API Gateway

For custom API gateways or middleware:

```yaml
requestOptions:
  headers:
    X-API-Key: "gateway-api-key"
    X-Client-ID: "continue-client"
    X-Version: "1.0"
```

### 3. Load Balancer Headers

For load balancer affinity or routing:

```yaml
requestOptions:
  headers:
    X-Session-Affinity: "server-01"
    X-Load-Balancer-Route: "ollama-cluster"
```

## Header Precedence

- Custom headers from `requestOptions.headers` are applied first
- If `apiKey` is specified, it will override any `Authorization` header with `Bearer ${apiKey}`
- Content-Type defaults to `application/json` but can be overridden

Example showing precedence:

```yaml
apiKey: "my-api-key"
requestOptions:
  headers:
    Authorization: "Basic override"  # This will be overridden
    Content-Type: "application/xml"  # This will be used
    X-Custom: "value"               # This will be included
```

Results in headers:
```
Authorization: Bearer my-api-key
Content-Type: application/xml  
X-Custom: value
```

## Implementation Details

The headers are included in all Ollama API calls:
- `/api/generate` (completion)
- `/api/chat` (chat)
- `/api/embed` (embeddings)
- `/api/tags` (list models)
- `/api/pull` (install model)
- `/api/show` (model info)

## Testing Your Configuration

You can verify headers are being sent by:

1. **Using a proxy tool like mitmproxy:**
   ```bash
   mitmproxy -p 8080 --mode reverse:http://localhost:11434
   ```

2. **Check your reverse proxy/server logs** for the custom headers

3. **Use a simple HTTP echo server** to see what Continue sends:
   ```bash
   # Simple Python echo server
   python3 -c "
   from http.server import HTTPServer, BaseHTTPRequestHandler
   import json
   
   class EchoHandler(BaseHTTPRequestHandler):
       def do_POST(self):
           print('Headers:', dict(self.headers))
           self.send_response(200)
           self.send_header('Content-type', 'application/json')  
           self.end_headers()
           self.wfile.write(b'{\"response\": \"test\"}')
   
   HTTPServer(('localhost', 11434), EchoHandler).serve_forever()
   "
   ```

## Backward Compatibility

This feature is fully backward compatible:
- Existing configurations without `requestOptions.headers` work unchanged
- Default behavior (Content-Type, Authorization) is preserved
- No breaking changes to the Ollama provider interface 