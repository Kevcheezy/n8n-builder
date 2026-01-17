# AI Video Generation Skill

Patterns for building AI video generation pipelines optimized for **sales conversion**.

## Sales Conversion Video Principles

> **"Perfect is the enemy of realistic."** — The key to high-converting AI videos is avoiding the uncanny valley through intentional imperfections and natural movement.

### The Realism Framework

| Element | Avoid | Instead |
|---------|-------|---------|
| Skin | Flawless, smooth | Micro-pores, natural oils, fine lines |
| Lighting | Studio-perfect | "Available Light" (window + desk lamp) |
| Background | Clean/minimal | Lived-in (bookshelf, home office clutter) |
| Voice | Overly polished | Natural, slightly imperfect delivery |
| Movement | Static/robotic | Subtle constant micro-movements |

---

## Pipeline Architecture

```
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│   INPUT     │ ──▶ │   SCRIPT    │ ──▶ │   MEDIA     │
│   Topic,    │     │   Generate  │     │   Generate  │
│   Style     │     │   story     │     │   assets    │
└─────────────┘     └─────────────┘     └─────────────┘
                                               │
┌─────────────┐     ┌─────────────┐     ┌──────▼──────┐
│   PUBLISH   │ ◀── │   RENDER    │ ◀── │   AUDIO     │
│   Upload    │     │   Combine   │     │   Generate  │
│   to social │     │   assets    │     │   voiceover │
└─────────────┘     └─────────────┘     └─────────────┘
```

## Script Generation

### Google Gemini (Preferred)

Use Gemini as the primary script generation engine:

```javascript
// Code node: Call Google Gemini (PREFERRED)
const response = await fetch('https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-goog-api-key': $credentials.geminiApiKey
  },
  body: JSON.stringify({
    contents: [{ 
      parts: [{ 
        text: `Create a 60-second video script about ${$json.topic}. Include:
1. Hook (5 seconds)
2. Main content (45 seconds)  
3. Call to action (10 seconds)

Output JSON: {segments: [{text, duration, imagePrompt}]}`
      }] 
    }]
  })
});
```

### OpenAI/GPT (Alternative)
```javascript
// Code node: Call Gemini
const response = await fetch('https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-pro:generateContent', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'x-goog-api-key': $credentials.geminiApiKey
  },
  body: JSON.stringify({
    contents: [{ parts: [{ text: prompt }] }]
  })
});
```

## Image Generation

### Flux (via PiAPI)
```javascript
// HTTP Request node
const response = await $http.request({
  method: 'POST',
  url: 'https://api.piapi.ai/api/flux/v1/generations',
  headers: { 'Authorization': `Bearer ${apiKey}` },
  body: {
    prompt: $json.imagePrompt,
    width: 1080,
    height: 1920,  // Vertical for social
    num_outputs: 1
  }
});
```

### Polling for Result
```javascript
// Wait node + Loop pattern
const taskId = $json.taskId;
let status = 'processing';
let attempts = 0;

while (status === 'processing' && attempts < 30) {
  await new Promise(r => setTimeout(r, 2000));
  const result = await checkStatus(taskId);
  status = result.status;
  attempts++;
}

if (status !== 'completed') {
  throw new Error(`Image generation timeout after ${attempts} attempts`);
}
```

### Realistic Image Prompting (For Talking Heads)

The key to realistic AI avatars is prompting for **imperfections**:

```javascript
// Code node: Build realistic avatar prompt
const avatarPrompt = `
  Portrait photo of a ${$json.gender}, ${$json.ethnicity} complexion,
  ${$json.age}-year-old with ${$json.features}.
  
  SKIN REALISM: visible micro-pores, natural skin oils, fine expression lines,
  subtle under-eye texture, no airbrushing.
  
  LIGHTING: Available light from window mixed with warm desk lamp,
  NOT studio lighting, natural shadows on face.
  
  BACKGROUND: ${$json.background || 'home office with slightly messy bookshelf, lived-in feel'},
  depth of field blur on background.
  
  EXPRESSION: genuine, approachable, mid-conversation moment.
  Camera: 35mm lens, eye-level, slight off-center framing.
`;
```

**Recommended Tools:**
- **Nano Banana Pro** (PREFERRED): Best for realistic human generation
- **Enhancor AI**: Post-upscaling to add realistic texture
- **Flux**: Alternative via PiAPI

---

## Video Clip Generation (Talking Heads)

### The Syllable Method (Critical for Pacing)

Most video models only generate ~10 seconds at a time. To maintain **consistent speaking pace** across stitched clips, break scripts into **55-60 syllables per 10-second generation**.

```javascript
// Code node: Split script by syllables for consistent pacing
function countSyllables(text) {
  return text.toLowerCase()
    .replace(/[^a-z]/g, '')
    .replace(/[^aeiouy]+/g, ' ')
    .trim().split(' ')
    .filter(s => s.length > 0).length;
}

function splitBySyllables(script, targetSyllables = 57) {
  const sentences = script.match(/[^.!?]+[.!?]+/g) || [script];
  const chunks = [];
  let current = '';
  let currentCount = 0;
  
  for (const sentence of sentences) {
    const sentSyllables = countSyllables(sentence);
    if (currentCount + sentSyllables > targetSyllables && current) {
      chunks.push({ text: current.trim(), syllables: currentCount });
      current = sentence;
      currentCount = sentSyllables;
    } else {
      current += ' ' + sentence;
      currentCount += sentSyllables;
    }
  }
  if (current) chunks.push({ text: current.trim(), syllables: currentCount });
  return chunks;
}

return splitBySyllables($json.script).map(chunk => ({ json: chunk }));
```

### Active Idle Movement Prompts

Avoid "statue" look by including **micro-movements** in video prompts:

```javascript
// Code node: Build movement-rich video prompt
const movementPrompt = `
  ${$json.basePrompt}
  
  ACTIVE IDLE MOVEMENTS (throughout):
  - Hand Home Base: fingers gently shifting, subtle wrist rotations
  - Head: micro-tilts every 2-3 seconds, slight nods when emphasizing
  - Eyes: natural blink rate (every 3-4 seconds), brief glances aside
  - Body: subtle weight shifts, micro-sways
  - Expression: eyebrow micro-movements synced to speech emphasis
  
  TIMING CLUSTERS:
  [0.5s-3.0s] Hands in Active Idle + Head tilts slightly right + Brows furrow slightly
  [4.0s-6.0s] Slight forward lean + Hand gesture on key point + Eye contact intensifies
  [7.0s-9.5s] Settles back + Relaxed expression + Slight smile on conclusion
`;
```

### Veo 3.1 Fast (PREFERRED for Video Generation)

Veo 3.1 Fast is the preferred video generation model for speed and quality:

```javascript
// HTTP Request node: Veo 3.1 Fast (PREFERRED)
const response = await $http.request({
  method: 'POST',
  url: 'https://api.veo.google/v1/video/generate',
  headers: { 
    'Authorization': `Bearer ${$credentials.veoApiKey}`,
    'Content-Type': 'application/json'
  },
  body: {
    prompt: $json.videoPrompt,
    model: 'veo-3.1-fast',
    duration: 10,
    aspect_ratio: '9:16',  // Vertical for social
    output_format: 'mp4'
  }
});
```

### Kling 2.6 (Alternative for Talking Heads)
```json
{
  "method": "POST",
  "url": "https://api.piapi.ai/api/kling/v1/generations",
  "body": {
    "prompt": "{{$json.videoPrompt}}",
    "model": "kling-v1.5",
    "duration": 5,
    "aspect_ratio": "9:16"
  }
}
```

### Veo Async Pattern (Alternative)
```javascript
// Async generation with callback
const response = await $http.request({
  method: 'POST',
  url: 'https://api.example.com/veo3/generate',
  body: {
    prompt: $json.prompt,
    callbackUrl: `${n8nWebhookUrl}/video-ready/${$json.requestId}`
  }
});
```

## Audio Generation

### ElevenLabs
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://api.elevenlabs.io/v1/text-to-speech/{{$json.voiceId}}",
    "headers": { "xi-api-key": "{{$credentials.elevenLabsApiKey}}" },
    "body": {
      "text": "{{$json.script}}",
      "model_id": "eleven_turbo_v2",
      "voice_settings": {
        "stability": 0.5,
        "similarity_boost": 0.75
      }
    },
    "responseFormat": "file"
  }
}
```

### Audio Realism Tips

**Avoid overly polished "studio" voices** — they trigger distrust.

| Tool | Purpose |
|------|---------|
| **Resemble AI** | More natural voice cloning than standard ElevenLabs |
| **Adobe Podcast** | Clean audio while preserving natural feel |
| **ElevenLabs** | Good with lower stability settings (0.3-0.5) |

```javascript
// Code node: Configure for natural-sounding voice
const voiceSettings = {
  stability: 0.35,           // Lower = more natural variation
  similarity_boost: 0.7,     // Balance between clone accuracy and natural feel
  style: 0.4,                // Add some expressiveness
  use_speaker_boost: false   // Disable for more organic sound
};
```

---

## Post-Production (Jump Cut Handling)

When stitching 10-second clips, **mask the seams**:

```javascript
// Code node: Insert B-roll overlay at transitions
const transitionOverlays = $json.segments.map((seg, idx) => {
  if (idx === 0) return seg;
  return {
    ...seg,
    transition: {
      type: 'broll_overlay',       // or 'text_animation', 'zoom_effect'
      duration_ms: 500,
      asset: $json.brollUrls[idx % $json.brollUrls.length]
    }
  };
});
```

**Transition Techniques:**
- B-roll footage overlay (product shots, graphics)
- Text/caption animations at cut points
- Slight zoom-in/out across cuts
- Graphics or emoji overlays

---

## Video Assembly

### Creatomate
```json
{
  "method": "POST",
  "url": "https://api.creatomate.com/v1/renders",
  "headers": { "Authorization": "Bearer {{$credentials.creatomateApiKey}}" },
  "body": {
    "template_id": "your-template-id",
    "modifications": {
      "Audio-1": "{{$json.audioUrl}}",
      "Video-1": "{{$json.videoClipUrl}}",
      "Text-1": "{{$json.overlayText}}"
    }
  }
}
```

### JSON2Video
```javascript
const project = {
  scenes: $json.segments.map(seg => ({
    duration: seg.duration,
    elements: [
      { type: 'image', src: seg.imageUrl },
      { type: 'audio', src: seg.audioUrl },
      { type: 'text', text: seg.caption, style: 'caption' }
    ]
  }))
};
```

## Status Tracking Pattern

Use Google Sheets/Airtable for pipeline tracking:

```javascript
// Update status at each stage
const statusUpdate = {
  request_id: $json.requestId,
  stage: 'audio_generation',
  status: 'completed',
  audio_url: $json.audioUrl,
  updated_at: new Date().toISOString()
};
```

## Output Storage

### Google Drive (PREFERRED)

All generated videos should be saved directly to Google Drive:

```json
{
  "type": "n8n-nodes-base.googleDrive",
  "parameters": {
    "operation": "upload",
    "name": "={{$json.videoTitle}}_{{$now.format('yyyy-MM-dd_HHmmss')}}.mp4",
    "folderId": "{{$credentials.googleDriveFolderId}}",
    "binaryPropertyName": "videoFile"
  }
}
```

```javascript
// Code node: Organize outputs in Google Drive folder structure
const folderPath = `AI_Videos/${$json.projectName}/${$now.format('yyyy-MM')}`;
const fileName = `${$json.videoTitle}_${$json.requestId}.mp4`;

return {
  json: {
    folderPath,
    fileName,
    metadata: {
      generatedAt: $now.toISO(),
      topic: $json.topic,
      duration: $json.totalDuration
    }
  }
};
```

## Publishing

### TikTok
```json
{
  "type": "n8n-nodes-base.httpRequest",
  "parameters": {
    "method": "POST",
    "url": "https://open.tiktokapis.com/v2/post/publish/inbox/video/init/",
    "headers": {
      "Authorization": "Bearer {{$credentials.tiktokAccessToken}}"
    },
    "body": {
      "video_url": "{{$json.videoUrl}}",
      "title": "{{$json.caption}}"
    }
  }
}
```

### YouTube Shorts
```json
{
  "type": "n8n-nodes-base.youTube",
  "parameters": {
    "operation": "upload",
    "title": "{{$json.title}}",
    "description": "{{$json.description}}",
    "privacyStatus": "public",
    "categoryId": "22"
  }
}
```

## Cost Optimization

| Service | Tip |
|---------|-----|
| Script | Use GPT-4o-mini for drafts, GPT-4o for final |
| Images | Batch prompts, use lower resolution for previews |
| Video | Generate only final segments, not previews |
| Audio | Use turbo models for speed |
| Rendering | Template-based is cheaper than full editing |

## Error Recovery

```javascript
// Checkpoint pattern
const checkpoint = {
  scriptGenerated: false,
  imagesGenerated: [],
  audioGenerated: false,
  videoRendered: false
};

// On failure, check which assets exist and resume
```
