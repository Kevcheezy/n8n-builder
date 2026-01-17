# AI Creative Agent Skill

A complete system for repurposing viral ads and content using AI. This workflow analyzes existing videos, breaks them into scenes, and recreates them with your products/characters.

---

## System Overview

```
┌────────────────┐     ┌────────────────┐     ┌────────────────┐
│  INPUT VIDEO   │ ──▶ │    GEMINI      │ ──▶ │   AI AGENT     │
│  + Character   │     │   Analyze &    │     │  Write Scene   │
│  + Products    │     │  Break Scenes  │     │    Prompts     │
│  + Request     │     │  (SEAL CAM)    │     │                │
└────────────────┘     └────────────────┘     └────────────────┘
                                                      │
┌────────────────┐     ┌────────────────┐     ┌───────▼────────┐
│   FINAL AD     │ ◀── │  KLING 2.6     │ ◀── │ NANO BANANA    │
│  (Manual Edit) │     │  Video Gen     │     │  Image Gen     │
└────────────────┘     └────────────────┘     └────────────────┘
                              │
                       ┌──────▼──────┐
                       │    SUNO     │
                       │  Music Gen  │
                       └─────────────┘
```

---

## The SEAL CAM Framework

For consistent scene analysis and prompt generation:

| Element | Description | Example |
|---------|-------------|---------|
| **S** - Subject | Main character/object in frame | "Polar bear wearing gold necklace" |
| **E** - Environment | Physical setting/location | "Luxury studio set with gold backdrop" |
| **A** - Action | Movement/activity in scene | "Slow head turn, minimal movement" |
| **L** - Lighting | Light setup affecting mood | "Soft key light, golden rim lighting" |
| **Cam** - Camera | Lens, angle, movement | "85mm lens, f/1.8, slow dolly in" |
| **M** - Meta | Technical/stylistic tokens | "8K, film grain, cinematic color grade" |

---

## Step 1: Video Analysis (Gemini)

Upload reference video and analyze with this prompt:

```javascript
const videoAnalysisPrompt = `
Analyze this video and break it into individual scenes.

For EACH scene, provide a detailed breakdown using the SEAL CAM framework:

**S (Subject):** Describe the main subject/character in detail
**E (Environment):** Describe the physical setting and background
**A (Action):** What movement or action occurs in this scene
**L (Lighting):** Describe the lighting setup (key light, fill, rim, color temperature)
**Cam (Camera):** 
  - Camera type (cinema camera, DSLR, etc.)
  - Lens focal length (24mm, 50mm, 85mm, etc.)
  - Aperture estimate (f/1.4, f/2.8, etc.)
  - Camera movement (static, dolly, pan, tilt, handheld)
  - Shot type (wide, medium, close-up, extreme close-up)
**M (Meta tokens):** Technical and stylistic cues (resolution, color grade, film grain, aspect ratio)

Also provide:
- **Music description:** Describe the background music style, tempo, mood
- **Scene duration:** Estimated length in seconds
- **Transition:** How this scene transitions to the next

Output as structured JSON with an array of scenes.
`;
```

---

## Step 2: Scene Prompt Generation (AI Agent)

The agent transforms video analysis into new prompts:

```javascript
const agentSystemPrompt = `
You are an AI Creative Director. Your job is to repurpose video content.

Given:
1. A detailed video analysis (SEAL CAM breakdown)
2. Reference images of character/products
3. Creative direction from the user

Create NEW prompts for each scene that:
- Maintain the same shot composition and camera work
- Replace subjects/products with the provided references
- Adjust environment/lighting per user direction
- Keep the same pacing and scene structure

For each scene, output:

IMAGE PROMPT (for Nano Banana Pro):
- Use SEAL CAM framework
- Include all character/product details from reference images
- Specify exact lighting and camera settings

VIDEO PROMPT (for Kling 2.6):
- Describe the motion/action clearly
- Specify camera movement
- Keep prompts under 500 characters

MUSIC PROMPT (for Suno - one for entire video):
- Genre and style
- Tempo and mood
- Instrumentation
- Duration

Guidelines:
- Never include text overlays unless specifically requested
- Maintain aspect ratio consistency
- Each scene should be 5-10 seconds for video generation
`;
```

---

## Step 3: Image Generation (Nano Banana Pro via Wavespeed)

```json
{
  "method": "POST",
  "url": "https://api.wavespeed.ai/v1/nano-banana-pro/generate",
  "headers": {
    "Authorization": "Bearer {{$credentials.wavespeedApiKey}}",
    "Content-Type": "application/json"
  },
  "body": {
    "prompt": "{{$json.imagePrompt}}",
    "reference_images": ["{{$json.characterImageUrl}}", "{{$json.productImageUrl}}"],
    "aspect_ratio": "9:16",
    "num_outputs": 1
  }
}
```

### Alternative: Gemini Image Generation
```javascript
// Note: Gemini adds watermarks
const geminiImagePrompt = `
Generate an image based on this description:
${$json.imagePrompt}

Reference the style and character from the uploaded images.
Aspect ratio: 9:16 (vertical for social media)
`;
```

---

## Step 4: Video Generation (Kling 2.6 via Wavespeed)

```json
{
  "method": "POST",
  "url": "https://api.wavespeed.ai/api/v1/kling/2.6/generate",
  "headers": {
    "Authorization": "Bearer {{$credentials.wavespeedApiKey}}"
  },
  "body": {
    "prompt": "{{$json.videoPrompt}}",
    "image_url": "{{$json.generatedImageUrl}}",
    "duration": 5,
    "aspect_ratio": "9:16"
  }
}
```

### Polling for Video Completion
```javascript
// Video generation is async - poll for completion
const maxAttempts = 60;
const pollInterval = 5000; // 5 seconds

for (let i = 0; i < maxAttempts; i++) {
  const status = await checkVideoStatus(taskId);
  
  if (status.state === 'completed') {
    return { videoUrl: status.output_url };
  }
  
  if (status.state === 'failed') {
    throw new Error(`Video generation failed: ${status.error}`);
  }
  
  await new Promise(r => setTimeout(r, pollInterval));
}
```

---

## Step 5: Music Generation (Suno via Krea.ai)

```json
{
  "method": "POST", 
  "url": "https://api.krea.ai/v1/suno/generate",
  "headers": {
    "Authorization": "Bearer {{$credentials.kreaApiKey}}"
  },
  "body": {
    "prompt": "{{$json.musicPrompt}}",
    "duration": 30,
    "instrumental": true
  }
}
```

---

## Input Structure (Airtable)

| Field | Type | Description |
|-------|------|-------------|
| `project_name` | Text | Identifier for the project |
| `input_video` | Attachment | Reference video to repurpose |
| `character_image` | Attachment | Image of character/model to use |
| `product_image` | Attachment | Product images to feature |
| `request` | Long Text | Creative direction (see below) |
| `aspect_ratio` | Select | "9:16" (vertical) or "16:9" (horizontal) |
| `status` | Select | "pending", "processing", "done" |

### Request Template
```
Have the character in image 1 appear in scene [X].
Feature the products from image 2 instead of original products.
Use [gold/blue/etc.] color scheme for backgrounds.
Create [X] scenes total.
Do NOT include text overlays.
```

---

## Output Structure (Airtable - Scenes)

| Field | Type | Description |
|-------|------|-------------|
| `scene_number` | Number | Scene order |
| `image_prompt` | Long Text | SEAL CAM prompt for image |
| `video_prompt` | Long Text | Motion prompt for video |
| `generated_image` | Attachment | Output from Nano Banana |
| `generated_video` | Attachment | Output from Kling 2.6 |
| `status` | Select | "pending", "image_done", "video_done" |

---

## Workflow Execution Order

```
1. GET INPUTS
   └── Fetch project from Airtable (video, images, request)

2. ANALYZE VIDEO  
   └── Send to Gemini → Get SEAL CAM breakdown

3. GENERATE PROMPTS
   └── AI Agent → Create image/video/music prompts per scene

4. CREATE IMAGES (batch)
   └── For each scene → Nano Banana Pro → Store in Airtable

5. CREATE VIDEOS (batch)
   └── For each scene → Kling 2.6 → Store in Airtable

6. CREATE MUSIC (optional)
   └── Suno → Store audio files

7. MARK COMPLETE
   └── Update project status
```

---

## Finding Winning Ads

### Magic Brief (magicbrief.com)
- Filter by industry, theme, language
- Filter by "Winners" performance tag
- Filter by runtime > 90 days (proven ad spend)
- Right-click → Save video

### Instagram (Sort Feed Extension)
- Install "Sort Feed for Instagram" Chrome extension
- Browse brand's reels
- Sort by views to find viral content
- Download using extension

---

## Best Practices

1. **Quality Checkpoints**: Run in stages (prompts → images → videos) to inspect before proceeding
2. **Scene Limit**: 5-8 scenes maximum for manageable output
3. **Character Consistency**: Use same reference image across all scenes
4. **Manual Final Edit**: Use Premiere/CapCut to assemble final video with music
5. **Prompt Refinement**: Edit generated prompts before image/video generation if needed

---

## API Alternatives

| Service | Alternative | Notes |
|---------|-------------|-------|
| Wavespeed | Fal.ai, Krea.ai | Similar pricing, $5 min topup |
| Kling 2.6 | Veo 3.1, Sora 2 | Kling best for glamour shots |
| Nano Banana | Flux, Gemini | Gemini has watermarks |
| Suno | Udio | Suno currently best for music |
