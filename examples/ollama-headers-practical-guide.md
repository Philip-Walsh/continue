# Ollama Custom Headers - Practical Guide & Testing

## ✅ **Our Implementation Status**

✅ **Backend Implementation**: Complete - Custom headers working in all Ollama HTTP requests  
✅ **Schema Support**: requestOptions.headers already supported  
✅ **Backward Compatibility**: 100% - No breaking changes  
❌ **UI Support**: Not implemented - Users must edit config files manually  

---

## 🚀 **How to Use Custom Headers with Ollama**

### 1. **Basic Configuration (YAML)**

```yaml
models:
  - name: "Secured Ollama"
    provider: ollama
    model: llama3.1:8b
    apiBase: "https://ollama.mydomain.com:11434"
    requestOptions:
      headers:
        Authorization: "Bearer your-custom-token"
        X-API-Version: "v1"
        X-Client-ID: "continue-client"
```

### 2. **Corporate Proxy Setup (JSON)**

```json
{
  "models": [
    {
      "title": "Corporate Ollama",
      "provider": "ollama", 
      "model": "llama3.1:8b",
      "apiBase": "https://ollama-proxy.company.com:11434",
      "requestOptions": {
        "headers": {
          "X-Forwarded-User": "john.doe@company.com",
          "X-Proxy-Auth": "proxy-secret-token",
          "X-Tenant-ID": "department-engineering"
        }
      }
    }
  ]
}
```

### 3. **Multi-tenant Deployment**

```yaml
models:
  - name: "Multi-tenant Ollama"
    provider: ollama
    model: codellama:7b
    apiBase: "https://shared-ollama.example.com:11434"
    requestOptions:
      headers:
        X-Tenant-ID: "customer-123"
        X-Resource-Pool: "gpu-cluster-a"
        Authorization: "Bearer tenant-specific-token"
```

---

## 🧪 **Testing Your Configuration**

### **Test 1: Verify Headers are Sent**

1. **Start a local echo server:**
```bash
npx http-echo-server
# Server starts on http://localhost:8081
```

2. **Configure Continue to use the echo server:**
```yaml
models:
  - name: "Test Ollama"
    provider: ollama
    model: llama3.1:8b
    apiBase: "http://localhost:8081"  # Echo server
    requestOptions:
      headers:
        X-Test-Header: "test-value"
        Authorization: "Bearer test-token"
```

3. **Use Continue and check the echo server logs** - you should see your custom headers.

### **Test 2: Real Ollama with Logging**

1. **Enable Ollama request logging:**
```bash
OLLAMA_DEBUG=1 ollama serve
```

2. **Configure Continue with custom headers and use the chat** - check Ollama logs for headers.

### **Test 3: API Key Precedence**

```yaml
models:
  - name: "Precedence Test"
    provider: ollama 
    model: llama3.1:8b
    apiKey: "official-api-key"  # This should win
    requestOptions:
      headers:
        Authorization: "Bearer should-be-overridden"  # This gets overridden
        X-Custom: "this-should-remain"  # This should remain
```

**Expected behavior**: Authorization header uses `Bearer official-api-key`, not the custom one.

---

## 🛠️ **Real-World Use Cases**

### **1. Corporate Environment with Reverse Proxy**

**Scenario**: Company runs Ollama behind nginx with authentication

```nginx
# nginx.conf
location /ollama/ {
    proxy_pass http://ollama-backend:11434/;
    # Require custom header for auth
    if ($http_x_proxy_auth != "company-secret") {
        return 403;
    }
}
```

**Continue Config**:
```yaml
models:
  - name: "Corporate Ollama"
    provider: ollama
    model: llama3.1:8b  
    apiBase: "https://api.company.com/ollama"
    requestOptions:
      headers:
        X-Proxy-Auth: "company-secret"
        X-Employee-ID: "12345"
```

### **2. Load Balancer with Routing Headers**

**Scenario**: Route requests to different Ollama instances based on headers

```yaml
models:
  - name: "GPU Cluster A"
    provider: ollama
    model: llama3.1:70b
    apiBase: "https://ollama-lb.example.com:11434"
    requestOptions:
      headers:
        X-Route-Target: "gpu-cluster-a"  # Routes to high-end GPUs
        X-Priority: "high"
  
  - name: "CPU Cluster" 
    provider: ollama
    model: llama3.1:8b
    apiBase: "https://ollama-lb.example.com:11434"
    requestOptions:
      headers:
        X-Route-Target: "cpu-cluster"   # Routes to CPU-only instances
        X-Priority: "normal"
```

### **3. API Gateway Integration**

**Scenario**: Ollama behind Kong/AWS API Gateway requiring custom headers

```yaml
models:
  - name: "Gateway Ollama"
    provider: ollama
    model: mistral:7b
    apiBase: "https://api-gateway.example.com/v1/ollama"
    requestOptions:
      headers:
        X-Gateway-Key: "gw-abc123"
        X-Rate-Limit-Plan: "premium"
        X-Source-Application: "continue-ide"
```

---

## 🔍 **No UI Changes Needed - Why Manual Config is OK**

### **Current UI Limitations**
- UI only exposes basic fields: `apiKey`, `apiBase`, completion parameters
- `requestOptions.headers` is **not exposed** in the Add Model form  
- Advanced users who need custom headers are comfortable editing config files

### **Why This is Acceptable**
1. **Advanced feature**: Custom headers are for power users, not casual users
2. **Diverse use cases**: Too many possible headers to create a good UI
3. **Security sensitivity**: Headers often contain secrets better kept in config files
4. **Documentation**: We provide clear examples and documentation

### **Alternative: Manual Config Editing**
Users can easily:
1. Use UI to add basic Ollama model
2. Edit config file to add `requestOptions.headers`
3. Restart Continue to pick up changes

This is the same pattern used by other advanced features in Continue.

---

## ⚡ **Quick Validation Checklist**

✅ **Headers included in all HTTP requests**:
- [x] `/api/show` (constructor)
- [x] `/api/generate` (completion)  
- [x] `/api/chat` (chat)
- [x] `/api/tags` (listModels)
- [x] `/api/embed` (embedding)
- [x] `/api/pull` (installModel)

✅ **Header precedence works correctly**:
- [x] `apiKey` overrides custom `Authorization` header
- [x] Other custom headers preserved alongside `apiKey`

✅ **Backward compatibility maintained**:
- [x] Existing configs work without changes
- [x] No breaking changes to schema or behavior

✅ **Real-world scenarios supported**:
- [x] Reverse proxy authentication
- [x] API gateway integration  
- [x] Multi-tenant deployments
- [x] Load balancer routing

---

## 📋 **Summary: Ready for Production**

Our Ollama headers implementation is **production-ready** and enables:

- ✅ **Secure corporate deployments** with proxy authentication
- ✅ **Multi-tenant setups** with tenant-specific headers  
- ✅ **Load balancer integration** with routing headers
- ✅ **API gateway compatibility** with custom auth schemes

The feature works through manual config editing (which is appropriate for this advanced use case), maintains full backward compatibility, and follows Continue's existing patterns for `requestOptions`.

**No UI changes are required** - this is an advanced feature that power users will configure through config files, just like many other advanced Continue features. 