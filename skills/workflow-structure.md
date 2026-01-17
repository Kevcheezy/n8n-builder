# Workflow Structure Skill

Core patterns for organizing n8n workflows.

## Trigger Types

| Type | Use Case | Node |
|------|----------|------|
| Webhook | External events, APIs, callbacks | `Webhook` |
| Schedule | Time-based automation | `Schedule Trigger` |
| Manual | Testing, on-demand | `Manual Trigger` |
| Event | n8n internal events | `n8n Trigger` |
| Sub-workflow | Called by other workflows | `Execute Workflow Trigger` |

## Standard Flow Pattern

```
Trigger → Normalize → Validate → Process → Output
```

1. **Trigger**: Entry point
2. **Normalize**: Transform input to standard format (Code node)
3. **Validate**: Check required fields, filter invalid (IF/Filter node)
4. **Process**: Core logic (multiple nodes)
5. **Output**: Save results, send notifications

## Sub-Workflow Architecture

Break workflows when:
- More than 15-20 nodes
- Reusable logic across workflows  
- Different error handling needs
- Parallel processing required

```
Main Workflow
├── Sub: Generate Content
├── Sub: Process Media  
└── Sub: Publish & Notify
```

Call with: `Execute Workflow` node

## Branching Patterns

### Switch (Preferred for 3+ paths)
```javascript
// Switch node expression
{{ $json.type }}

// Routes: "video", "image", "audio", "default"
```

### IF (For binary decisions)
```javascript
{{ $json.status === "ready" }}
```

## Node Naming Convention

Format: `[Action] [Target] [Detail]`

Examples:
- `GET User Profile`
- `TRANSFORM Response Data`
- `SEND Slack Notification`  
- `CHECK Video Status`
- `WAIT For Processing`

## Connection Best Practices

- Main path: solid connection
- Error path: connect from Error output → Error Handler
- Always have an error output path from critical nodes
- Use **Merge** node to rejoin split paths
