# API Integration Skill

Patterns for integrating with external APIs in n8n.

## HTTP Request Node Basics

```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "predefinedCredentialType",
    "nodeCredentialType": "httpHeaderAuth",
    "sendHeaders": true,
    "headerParameters": {
      "parameters": [
        { "name": "Content-Type", "value": "application/json" }
      ]
    },
    "sendBody": true,
    "bodyParameters": {
      "parameters": [
        { "name": "key", "value": "={{$json.value}}" }
      ]
    }
  }
}
```

## Authentication Patterns

### API Key (Header)
```json
{
  "authentication": "genericCredentialType",
  "genericAuthType": "httpHeaderAuth",
  "httpHeaderAuth": {
    "name": "Authorization",
    "value": "Bearer {{$credentials.apiKey}}"
  }
}
```

### OAuth2
Use n8n's built-in OAuth2 credential type:
```json
{
  "authentication": "oAuth2Api",
  "oAuth2Api": {
    "accessTokenUrl": "https://api.example.com/oauth/token",
    "authUrl": "https://api.example.com/oauth/authorize",
    "scope": "read write"
  }
}
```

### Custom Headers
```javascript
// Code node for complex auth
const timestamp = Date.now();
const signature = crypto
  .createHmac('sha256', secretKey)
  .update(`${timestamp}${body}`)
  .digest('hex');

return {
  json: {
    headers: {
      'X-Timestamp': timestamp,
      'X-Signature': signature
    }
  }
};
```

## Polling Pattern

For async APIs that don't support webhooks:

```javascript
// Code node: Poll with timeout
const maxAttempts = 30;
const pollInterval = 2000; // 2 seconds
const taskId = $json.taskId;

for (let i = 0; i < maxAttempts; i++) {
  const response = await $http.request({
    method: 'GET',
    url: `https://api.example.com/status/${taskId}`
  });
  
  if (response.status === 'completed') {
    return { json: response };
  }
  
  if (response.status === 'failed') {
    throw new Error(response.error);
  }
  
  await new Promise(r => setTimeout(r, pollInterval));
}

throw new Error(`Timeout waiting for task ${taskId}`);
```

## Webhook Callback Pattern

For APIs that support callbacks:

1. **Initiate with callback URL**:
```json
{
  "body": {
    "prompt": "{{$json.prompt}}",
    "webhook_url": "https://your-n8n.com/webhook/{{$workflow.id}}/callback"
  }
}
```

2. **Separate webhook workflow**:
- Trigger: `Webhook` node
- Store/update status in database
- Trigger continuation of main workflow

## Rate Limiting

### Built-in Batching
```json
{
  "type": "n8n-nodes-base.splitInBatches",
  "parameters": {
    "batchSize": 10,
    "options": {
      "reset": false
    }
  }
}
```

### Manual Rate Limiting
```javascript
// Code node: Rate limit
const rateLimitMs = 1000; // 1 request per second
const items = $input.all();
const results = [];

for (const item of items) {
  const start = Date.now();
  
  const response = await $http.request({
    method: 'POST',
    url: apiUrl,
    body: item.json
  });
  
  results.push({ json: response });
  
  const elapsed = Date.now() - start;
  if (elapsed < rateLimitMs) {
    await new Promise(r => setTimeout(r, rateLimitMs - elapsed));
  }
}

return results;
```

## Response Normalization

Always normalize after API calls:

```javascript
// Code node: Normalize response
const response = $input.item.json;

// Handle different API response formats
let data;
if (response.data) {
  data = response.data;
} else if (response.result) {
  data = response.result;
} else {
  data = response;
}

// Ensure consistent structure
return {
  json: {
    id: data.id || data._id,
    status: (data.status || data.state || 'unknown').toLowerCase(),
    result: data.output || data.result || null,
    raw: response // Keep raw for debugging
  }
};
```

## Error Handling

### Check HTTP Status
```javascript
// Code node: Handle HTTP errors
const response = $input.item.json;

if (response.statusCode >= 400) {
  const errorMessages = {
    400: 'Bad request - check input parameters',
    401: 'Unauthorized - check API credentials',
    403: 'Forbidden - insufficient permissions',
    404: 'Not found - check endpoint URL',
    429: 'Rate limited - slow down requests',
    500: 'Server error - try again later'
  };
  
  throw new Error(errorMessages[response.statusCode] || `HTTP ${response.statusCode}`);
}

return $input.item;
```

### Retry Configuration
```json
{
  "retryOnFail": true,
  "maxTries": 3,
  "waitBetweenTries": 1000
}
```

## Pagination

### Offset-based
```javascript
// Code node: Paginate
const pageSize = 100;
let offset = 0;
const allResults = [];

while (true) {
  const response = await $http.request({
    method: 'GET',
    url: `${apiUrl}?limit=${pageSize}&offset=${offset}`
  });
  
  allResults.push(...response.items);
  
  if (response.items.length < pageSize) break;
  offset += pageSize;
}

return allResults.map(item => ({ json: item }));
```

### Cursor-based
```javascript
let cursor = null;
const allResults = [];

do {
  const response = await $http.request({
    method: 'GET',
    url: apiUrl,
    qs: { cursor, limit: 100 }
  });
  
  allResults.push(...response.data);
  cursor = response.nextCursor;
} while (cursor);

return allResults.map(item => ({ json: item }));
```

## File Upload

```javascript
// Code node: Upload file
const fileBuffer = Buffer.from($binary.file.data, 'base64');

const formData = new FormData();
formData.append('file', fileBuffer, {
  filename: $binary.file.fileName,
  contentType: $binary.file.mimeType
});

const response = await $http.request({
  method: 'POST',
  url: uploadUrl,
  headers: {
    ...formData.getHeaders(),
    'Authorization': `Bearer ${apiKey}`
  },
  body: formData
});
```
