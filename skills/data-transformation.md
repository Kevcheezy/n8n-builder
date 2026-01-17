# Data Transformation Skill

Patterns for normalizing, validating, and transforming data.

## Normalize First Pattern

Transform raw API responses immediately after receiving:

```javascript
// Code node: Normalize API Response
const raw = $input.item.json;

return {
  json: {
    id: raw.data?.id || raw.id,
    status: raw.status?.toLowerCase() || 'unknown',
    content: raw.content || raw.data?.content || '',
    url: raw.url || raw.result?.url || null,
    createdAt: raw.created_at || raw.createdAt || new Date().toISOString()
  }
};
```

## Edit Fields (Set) Node

Select and rename fields:

```json
{
  "type": "n8n-nodes-base.set",
  "parameters": {
    "mode": "manual",
    "fields": {
      "values": [
        { "name": "videoId", "stringValue": "={{ $json.id }}" },
        { "name": "status", "stringValue": "={{ $json.status }}" }
      ]
    }
  }
}
```

## Validation Patterns

### Required Field Check
```javascript
const required = ['topic', 'style', 'duration'];
const data = $input.item.json;

const errors = required
  .filter(field => !data[field])
  .map(field => `${field} is required`);

if (errors.length) {
  throw new Error(errors.join(', '));
}
return $input.item;
```

### Type Coercion
```javascript
return {
  json: {
    duration: parseInt($input.item.json.duration) || 60,
    segments: Array.isArray($input.item.json.segments) 
      ? $input.item.json.segments 
      : [$input.item.json.segments].filter(Boolean)
  }
};
```

## Array Operations

### Split Items
Use `Split Out` node to convert array to individual items.

### Aggregate Items
```javascript
// Code node: Aggregate after loop
const items = $input.all().map(i => i.json);
return {
  json: {
    results: items,
    count: items.length,
    successCount: items.filter(i => i.success).length
  }
};
```

### Filter Items
```javascript
// Code node or Filter node
return $input.all()
  .filter(item => item.json.status === 'completed')
  .map(item => ({ json: item.json }));
```

## Merge Patterns

### Merge by ID
```json
{
  "type": "n8n-nodes-base.merge",
  "parameters": {
    "mode": "combine",
    "mergeByFields": {
      "values": [{ "field1": "id", "field2": "videoId" }]
    }
  }
}
```

### Combine All
```json
{
  "mode": "combineBySql",
  "query": "SELECT a.*, b.url FROM input1 a LEFT JOIN input2 b ON a.id = b.refId"
}
```

## Expression Helpers

```javascript
// Date formatting
{{ $now.format('YYYY-MM-DD') }}

// Default values
{{ $json.value ?? 'default' }}

// Conditional
{{ $json.status === 'ready' ? 'proceed' : 'wait' }}

// String manipulation  
{{ $json.title.toLowerCase().replace(/\s+/g, '-') }}

// Array access
{{ $json.items[0].name }}
{{ $json.items.map(i => i.id).join(',') }}
```

## Binary Data Handling

```javascript
// Code node: Convert base64 to binary
const base64Data = $input.item.json.imageBase64;
const binaryData = Buffer.from(base64Data, 'base64');

return {
  json: $input.item.json,
  binary: {
    image: {
      data: binaryData.toString('base64'),
      mimeType: 'image/png',
      fileName: 'generated-image.png'
    }
  }
};
```
