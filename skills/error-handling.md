# Error Handling Skill

Patterns for robust error handling in n8n workflows.

## Error Trigger Setup

Every workflow needs a global error handler:

```json
{
  "nodes": [
    {
      "name": "Error Trigger",
      "type": "n8n-nodes-base.errorTrigger"
    },
    {
      "name": "Send Error Notification",
      "type": "n8n-nodes-base.slack",
      "parameters": {
        "text": "Workflow {{$node['Error Trigger'].json.workflow.name}} failed: {{$node['Error Trigger'].json.execution.error.message}}"
      }
    }
  ]
}
```

## Continue On Error

For loops where single item failures shouldn't stop the batch:

```json
{
  "onError": "continueRegularOutput",
  "continueOnFail": true
}
```

Access error info: `{{ $json.error }}`

## Retry Pattern

For unreliable APIs (e.g., AI services):

```javascript
// Code node: Retry with exponential backoff
const maxRetries = 3;
const baseDelay = 1000;

let lastError;
for (let i = 0; i < maxRetries; i++) {
  try {
    const response = await fetch(url, options);
    if (response.ok) return response.json();
    throw new Error(`HTTP ${response.status}`);
  } catch (error) {
    lastError = error;
    await new Promise(r => setTimeout(r, baseDelay * Math.pow(2, i)));
  }
}
throw lastError;
```

## Error Categories

| Category | Action | Example |
|----------|--------|---------|
| Retryable | Retry with backoff | Rate limit, timeout |
| Fatal | Stop and notify | Invalid credentials |
| Recoverable | Skip and continue | Single item bad data |
| External | Wait and retry | Service outage |

## Validation Errors

Check before processing:

```javascript
// Code node: Validate input
const required = ['topic', 'duration', 'style'];
const missing = required.filter(f => !$input.item.json[f]);

if (missing.length > 0) {
  throw new Error(`Missing required fields: ${missing.join(', ')}`);
}

return $input.item;
```

## Error Notification Template

```
🚨 Workflow Error

Workflow: {{ $node['Error Trigger'].json.workflow.name }}
Execution: {{ $node['Error Trigger'].json.execution.id }}
Error: {{ $node['Error Trigger'].json.execution.error.message }}
Node: {{ $node['Error Trigger'].json.execution.error.node.name }}
Time: {{ $now.toISO() }}
```

## Compensation Pattern

For multi-step operations, track what succeeded:

```javascript
// Track completed steps
const completed = [];
try {
  await step1(); completed.push('step1');
  await step2(); completed.push('step2');
  await step3(); completed.push('step3');
} catch (error) {
  // Rollback completed steps
  for (const step of completed.reverse()) {
    await rollback[step]();
  }
  throw error;
}
```
