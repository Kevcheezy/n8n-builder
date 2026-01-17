# n8n JSON Structure Skill

**Required reading** — This skill defines the n8n workflow JSON format for generating importable workflows.

## Workflow JSON Structure

```json
{
  "name": "My Workflow Name",
  "nodes": [...],
  "connections": {...},
  "settings": {
    "executionOrder": "v1"
  },
  "staticData": null,
  "pinData": {}
}
```

---

## Node Structure

> ⚠️ **CRITICAL: `parameters` MUST be the first field in each node object** for n8n import to work correctly.

Correct field order:

```json
{
  "parameters": {...},
  "id": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
  "name": "Unique Node Name",
  "type": "n8n-nodes-base.nodeName",
  "typeVersion": 1,
  "position": [260, 340]
}
```

### Required Fields
| Field | Format | Notes |
|-------|--------|-------|
| `parameters` | object | **Must be first!** Node configuration |
| `id` | UUID string | Must be unique, use UUID v4 format |
| `name` | string | Must be unique within workflow |
| `type` | string | n8n node type identifier |
| `typeVersion` | number | Node version (check n8n docs) |
| `position` | [x, y] | Canvas position as nested array |

### Position Grid
- Start trigger at `[260, 340]`
- Space nodes `220px` apart horizontally
- Parallel branches: offset `220px` vertically

---

## Common Node Types

### Triggers

**Manual Trigger**
```json
{
  "id": "manual-trigger-1",
  "name": "Manual Trigger",
  "type": "n8n-nodes-base.manualTrigger",
  "typeVersion": 1,
  "position": [250, 300],
  "parameters": {}
}
```

**Webhook**
```json
{
  "id": "webhook-1",
  "name": "Webhook",
  "type": "n8n-nodes-base.webhook",
  "typeVersion": 2,
  "position": [250, 300],
  "parameters": {
    "httpMethod": "POST",
    "path": "video-request",
    "responseMode": "responseNode"
  }
}
```

**Schedule**
```json
{
  "id": "schedule-1",
  "name": "Schedule Trigger",
  "type": "n8n-nodes-base.scheduleTrigger",
  "typeVersion": 1.2,
  "position": [250, 300],
  "parameters": {
    "rule": {
      "interval": [{ "field": "hours", "hoursInterval": 1 }]
    }
  }
}
```

**Error Trigger** (Required in every workflow)
```json
{
  "id": "error-trigger-1",
  "name": "Error Trigger",
  "type": "n8n-nodes-base.errorTrigger",
  "typeVersion": 1,
  "position": [250, 600],
  "parameters": {}
}
```

---

### Data Processing

**Code Node**
```json
{
  "id": "code-1",
  "name": "Process Data",
  "type": "n8n-nodes-base.code",
  "typeVersion": 2,
  "position": [500, 300],
  "parameters": {
    "jsCode": "// Your JavaScript code here\nconst items = $input.all();\n\nreturn items.map(item => ({\n  json: {\n    processed: item.json.data\n  }\n}));"
  }
}
```

**Set/Edit Fields**
```json
{
  "id": "set-1",
  "name": "Set Fields",
  "type": "n8n-nodes-base.set",
  "typeVersion": 3.4,
  "position": [500, 300],
  "parameters": {
    "mode": "manual",
    "duplicateItem": false,
    "assignments": {
      "assignments": [
        {
          "id": "field-1",
          "name": "outputField",
          "value": "={{ $json.inputField }}",
          "type": "string"
        }
      ]
    }
  }
}
```

**IF Node**
```json
{
  "id": "if-1",
  "name": "Check Condition",
  "type": "n8n-nodes-base.if",
  "typeVersion": 2,
  "position": [500, 300],
  "parameters": {
    "conditions": {
      "options": { "caseSensitive": true, "leftValue": "" },
      "conditions": [
        {
          "id": "condition-1",
          "leftValue": "={{ $json.status }}",
          "rightValue": "ready",
          "operator": { "type": "string", "operation": "equals" }
        }
      ],
      "combinator": "and"
    }
  }
}
```

**Switch Node**
```json
{
  "id": "switch-1",
  "name": "Route by Type",
  "type": "n8n-nodes-base.switch",
  "typeVersion": 3,
  "position": [500, 300],
  "parameters": {
    "rules": {
      "rules": [
        { "outputKey": "video", "value": "={{ $json.type === 'video' }}" },
        { "outputKey": "image", "value": "={{ $json.type === 'image' }}" }
      ]
    },
    "fallbackOutput": "extra"
  }
}
```

---

### HTTP & APIs

**HTTP Request**
```json
{
  "id": "http-1",
  "name": "Call API",
  "type": "n8n-nodes-base.httpRequest",
  "typeVersion": 4.2,
  "position": [500, 300],
  "parameters": {
    "method": "POST",
    "url": "https://api.example.com/endpoint",
    "authentication": "genericCredentialType",
    "genericAuthType": "httpHeaderAuth",
    "sendBody": true,
    "specifyBody": "json",
    "jsonBody": "={{ JSON.stringify({ prompt: $json.prompt }) }}",
    "options": {}
  },
  "credentials": {
    "httpHeaderAuth": {
      "id": "cred-id",
      "name": "API Key"
    }
  }
}
```

---

### Flow Control

**Wait Node**
```json
{
  "id": "wait-1",
  "name": "Wait 5 Seconds",
  "type": "n8n-nodes-base.wait",
  "typeVersion": 1.1,
  "position": [500, 300],
  "parameters": {
    "amount": 5,
    "unit": "seconds"
  }
}
```

**Loop Over Items**
```json
{
  "id": "loop-1",
  "name": "Loop Over Items",
  "type": "n8n-nodes-base.splitInBatches",
  "typeVersion": 3,
  "position": [500, 300],
  "parameters": {
    "batchSize": 1,
    "options": {}
  }
}
```

**Merge Node**
```json
{
  "id": "merge-1",
  "name": "Merge Results",
  "type": "n8n-nodes-base.merge",
  "typeVersion": 3,
  "position": [900, 300],
  "parameters": {
    "mode": "combine",
    "mergeByFields": {
      "values": [{ "field1": "id", "field2": "id" }]
    },
    "options": {}
  }
}
```

---

## Connections Format

Connections define the flow between nodes:

```json
{
  "connections": {
    "Source Node Name": {
      "main": [
        [
          { "node": "Target Node Name", "type": "main", "index": 0 }
        ]
      ]
    }
  }
}
```

### Multiple Outputs (IF/Switch)
```json
{
  "IF Node": {
    "main": [
      [{ "node": "True Branch", "type": "main", "index": 0 }],
      [{ "node": "False Branch", "type": "main", "index": 0 }]
    ]
  }
}
```

### Multiple Inputs (Merge)
```json
{
  "Node A": {
    "main": [[{ "node": "Merge", "type": "main", "index": 0 }]]
  },
  "Node B": {
    "main": [[{ "node": "Merge", "type": "main", "index": 1 }]]
  }
}
```

---

## Complete Example

```json
{
  "name": "Simple API Workflow",
  "nodes": [
    {
      "id": "1",
      "name": "Manual Trigger",
      "type": "n8n-nodes-base.manualTrigger",
      "typeVersion": 1,
      "position": [250, 300],
      "parameters": {}
    },
    {
      "id": "2", 
      "name": "Set Request Data",
      "type": "n8n-nodes-base.set",
      "typeVersion": 3.4,
      "position": [450, 300],
      "parameters": {
        "mode": "manual",
        "assignments": {
          "assignments": [
            { "id": "a1", "name": "topic", "value": "AI automation", "type": "string" }
          ]
        }
      }
    },
    {
      "id": "3",
      "name": "Call OpenAI",
      "type": "n8n-nodes-base.httpRequest",
      "typeVersion": 4.2,
      "position": [650, 300],
      "parameters": {
        "method": "POST",
        "url": "https://api.openai.com/v1/chat/completions",
        "authentication": "genericCredentialType",
        "genericAuthType": "httpHeaderAuth",
        "sendBody": true,
        "specifyBody": "json",
        "jsonBody": "={{ JSON.stringify({ model: 'gpt-4o', messages: [{ role: 'user', content: $json.topic }] }) }}"
      }
    },
    {
      "id": "4",
      "name": "Error Trigger",
      "type": "n8n-nodes-base.errorTrigger", 
      "typeVersion": 1,
      "position": [250, 500],
      "parameters": {}
    }
  ],
  "connections": {
    "Manual Trigger": {
      "main": [[{ "node": "Set Request Data", "type": "main", "index": 0 }]]
    },
    "Set Request Data": {
      "main": [[{ "node": "Call OpenAI", "type": "main", "index": 0 }]]
    }
  },
  "settings": {
    "executionOrder": "v1"
  }
}
```

---

## Validation Checklist

Before outputting workflow JSON:
- [ ] All nodes have unique `id` and `name`
- [ ] All node `type` values are valid n8n node types
- [ ] All nodes are connected (no orphans except Error Trigger)
- [ ] Trigger node is first in the flow
- [ ] Error Trigger is included for error handling
- [ ] Positions don't overlap
- [ ] JSON is valid (parseable)
