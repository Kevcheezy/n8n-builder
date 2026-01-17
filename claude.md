# n8n Workflow Builder

This project enables AI-assisted creation of high-quality n8n workflows for **AI video generation automations**.

## Output Format

When building workflows, generate a `workflow.json` file in the `workflows/` directory that can be manually imported into n8n.

```
workflows/
├── my-video-automation.json
├── content-scheduler.json
└── ...
```

**Import in n8n Cloud:**
1. Open n8n → Workflows
2. Click "..." menu → "Import from File"
3. Select the generated `.json` file

---

## Skills Reference

Read the skills in `skills/` before building workflows:

| Skill | Purpose |
|-------|---------|
| [n8n-json-structure](skills/n8n-json-structure.md) | **Required** - n8n workflow JSON format |
| [ai-creative-agent](skills/ai-creative-agent.md) | **NEW** - Video repurposing system (SEAL CAM) |
| [workflow-structure](skills/workflow-structure.md) | Core workflow patterns and node organization |
| [error-handling](skills/error-handling.md) | Error triggers, retry logic, recovery |
| [data-transformation](skills/data-transformation.md) | Normalize, validate, and transform data |
| [ai-video-generation](skills/ai-video-generation.md) | Video generation integrations |
| [video-hooks](skills/video-hooks.md) | Scroll-stopping hooks for conversion |
| [api-integration](skills/api-integration.md) | HTTP requests, authentication, polling |

---

## Workflow Development Process

1. **Understand Requirements**: Clarify what automation is needed
2. **Read Skills**: Start with `n8n-json-structure.md`, then relevant domain skills
3. **Design Structure**: Plan the node flow with error handling
4. **Generate JSON**: Create a valid `workflow.json` file
5. **Output**: Save to `workflows/` directory for user to import

---

## Best Practices

### Workflow Structure
- Use **descriptive names** for workflows and nodes
- Break complex flows into **sub-workflows** (max 15-20 nodes per workflow)
- Always start with a **trigger node** (Webhook, Schedule, Manual)
- Implement **Switch nodes** instead of nested If statements

### Error Handling (Required)
- Every workflow must have an **Error Trigger** node
- Use **Continue on Error** for loops that shouldn't fail on single items
- Implement **retry logic** with exponential backoff for API calls
- Send notifications on critical failures (Slack, email, etc.)

### Data Flow
- **Normalize once early**: Transform raw API responses at entry point
- **Validate before processing**: Filter invalid records before expensive operations
- Use **Edit Fields (Set)** to select only needed properties
- Add a **Code node** after every external API call for response normalization

### AI Video Generation Specific
- Use the **syllable method** (55-60 syllables per 10-second clip)
- Prompt for **imperfections** in avatar images (micro-pores, natural lighting)
- Include **Active Idle movements** in video generation prompts
- Store video generation status in **Google Sheets** or **Airtable**
- Use **polling patterns** for async video generation APIs

---

## AI Video Generation Pipeline

```
┌─────────────┐   ┌───────────────┐   ┌─────────────┐
│   Trigger   │ → │ Script/Prompt │ → │   Images    │
│  (Webhook/  │   │  Generation   │   │ Generation  │
│   Manual)   │   │   (GPT/Gemini)│   │ (Flux/DALL-E)│
└─────────────┘   └───────────────┘   └─────────────┘
                                              ↓
┌─────────────┐   ┌───────────────┐   ┌─────────────┐
│   Publish   │ ← │    Video      │ ← │   Audio     │
│ (TikTok/YT) │   │   Assembly    │   │ (ElevenLabs)│
└─────────────┘   └───────────────┘   └─────────────┘
```

### Common Integrations
- **Text/Script**: OpenAI, Google Gemini, Anthropic Claude
- **Images**: Flux (via PiAPI), Pollinations.ai, DALL-E, Nano Banana Pro
- **Video Clips**: Kling 2.6 (via PiAPI), RunwayML, Veo 3.1
- **Audio/Voice**: ElevenLabs, Resemble AI, OpenAI TTS
- **Assembly**: Creatomate, JSON2Video, CapCut API
- **Storage**: Google Drive, AWS S3, Cloudinary
- **Publishing**: TikTok, YouTube, Instagram, LinkedIn

---

## Example Workflow Request

> "Create a workflow that takes a topic via webhook, generates a 60-second video with AI narration, and publishes to TikTok"

When receiving such requests:
1. Read `n8n-json-structure.md` first (required for valid output)
2. Read `ai-video-generation.md` for video-specific patterns
3. Design the full pipeline with error handling
4. Generate complete `workflow.json` with all nodes properly connected
5. Save to `workflows/` directory
