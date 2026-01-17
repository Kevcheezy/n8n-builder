# n8n Workflow Builder

AI-assisted creation of high-quality n8n workflows for **AI video generation automations**.

## Quick Start

1. Clone this repository
2. Read the skills in `skills/` to understand n8n workflow patterns
3. Generate workflows and save them to `workflows/`
4. Import into n8n Cloud

## Project Structure

```
n8n-builder/
├── claude.md              # AI assistant instructions
├── skills/                # Reference documentation
│   ├── n8n-json-structure.md      # Required - n8n JSON format
│   ├── ai-creative-agent.md       # Video repurposing system
│   ├── workflow-structure.md      # Core workflow patterns
│   ├── error-handling.md          # Error triggers & retry logic
│   ├── data-transformation.md     # Data normalization
│   ├── ai-video-generation.md     # Video generation integrations
│   ├── video-hooks.md             # Scroll-stopping hooks
│   └── api-integration.md         # HTTP requests & auth
└── workflows/             # Generated workflow JSON files
    └── ai-creative-agent.json
```

## Importing Workflows into n8n

1. Open n8n → Workflows
2. Click "..." menu → "Import from File"
3. Select a `.json` file from `workflows/`

## AI Video Generation Pipeline

```
Trigger → Script Generation → Image Generation → Audio → Video Assembly → Publish
```

### Supported Integrations

| Category | Services |
|----------|----------|
| **Text/Script** | OpenAI, Google Gemini, Anthropic Claude |
| **Images** | Flux (PiAPI), Pollinations.ai, DALL-E |
| **Video** | Kling 2.6 (PiAPI), RunwayML, Veo 3.1 |
| **Audio** | ElevenLabs, Resemble AI, OpenAI TTS |
| **Assembly** | Creatomate, JSON2Video, CapCut API |
| **Storage** | Google Drive, AWS S3, Cloudinary |
| **Publishing** | TikTok, YouTube, Instagram, LinkedIn |

## Best Practices

- **Max 15-20 nodes** per workflow (use sub-workflows for complex flows)
- **Always include** an Error Trigger node
- **Normalize data early** after API calls
- **Use polling patterns** for async video generation APIs
- **55-60 syllables** per 10-second video clip (syllable method)

## Skills Reference

Before building workflows, read the skills documentation:

1. **Start with** `skills/n8n-json-structure.md` (required)
2. **Then read** domain-specific skills as needed

## License

MIT
