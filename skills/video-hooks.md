# Video Hooks Skill

Principles for creating scroll-stopping hooks that drive views and conversions.

---

## Core Psychology

> People swipe on **autopilot** until something hits their brain receptors and makes them think "hmm, that's interesting."

The algorithm tracks when people:
1. Stop scrolling
2. Swipe past, then **swipe back** (major trigger signal)

Your hook must trigger this pattern within the **first 0.5-1 seconds**.

---

## The Open Loop Framework

**Never close a loop without opening another.**

| Concept | Description |
|---------|-------------|
| **Open Loop** | Plant a question/curiosity that demands an answer |
| **Keep Dangling** | Don't reveal the answer immediately |
| **Stack Loops** | When closing one, open another to maintain tension |

### Hook Types That Open Loops

1. **Challenge Common Beliefs** (strongest)
   - "Lean at 45 and I eat carbs every day"
   - "This coffee should NOT taste good"
   
2. **Curiosity Gap**
   - Show partial information, hide the key detail
   - Make them need to watch to get the answer

3. **Contrast/Transformation**
   - "My photos 10 years ago vs now"
   - Before/after that makes people anticipate

4. **Planted Question**
   - What question pops in the viewer's head?
   - If no question = no hook

---

## First Frame Rules

### The Safe Zone
```
┌─────────────────────────────────┐
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │  ← Top: Username/icons overlap
│                                 │
│     ┌─────────────────────┐     │
│     │   SAFE ZONE FOR     │     │
│     │   TEXT OVERLAY      │     │
│     └─────────────────────┘     │
│                                 │
│  ░░░░░░░░░░░░░░░░░░░░░░░░░░░░  │  ← Bottom: Comments/captions overlap
└─────────────────────────────────┘
```

### Visual Decluttering
- **One focal point** — don't compete for attention
- Remove background clutter or blur it
- Text should NOT overlap with visual focus
- Less is more — clean frames stop scrolls

### Text Overlay Best Practices
- Keep captions **smaller than you think**
- Use font weight variation: bold keywords, medium supporting text
- Add subtle background boxes behind text for readability
- Color pick from the image for cohesion (or use contrast)

---

## Hook Construction Formula

```javascript
// Code node: Generate hook structure
const hookStructure = {
  // Text hook (on screen)
  textHook: {
    format: "[Curiosity trigger] + [Open loop]",
    examples: [
      "This coffee should NOT taste good...",
      "Lean at 45 and I eat carbs every day",
      "POV: The shot that got 100K views"
    ],
    rules: [
      "Don't reveal the answer",
      "Challenge expectations",
      "Create FOMO if they swipe away"
    ]
  },
  
  // Visual hook (first frame)
  visualHook: {
    requirements: [
      "Clear focal point",
      "Decluttered background",
      "Intrigue element (blur key detail, unusual composition)"
    ]
  },
  
  // Spoken hook (if talking head)  
  spokenHook: {
    timing: "First 2-3 seconds",
    rules: [
      "Add context immediately",
      "Don't assume they read the caption",
      "State the hook verbally within first sentence"
    ]
  }
};
```

---

## The Blur Trick

**Hide key elements to force watching:**

```javascript
// Prompt for AI image generation with blur technique
const blurHookPrompt = `
  ${$json.baseImagePrompt}
  
  BLUR TECHNIQUE: The key element (${$json.secretElement}) should be 
  slightly blurred or partially obscured, creating visual intrigue.
  Viewer should sense something interesting but not fully see it.
  
  Add text overlay area that says "${$json.hookText}" with 
  punctuation that opens a loop (...).
`;
```

Example: Coffee with blueberries → blur the blueberries, text says "This coffee should NOT taste good..."

---

## Context is Everything

**Bad hook** (no context):
> "I definitely don't cut out carbs"

Viewer thinks: "Okay? Why should I care?"

**Good hook** (with context):
> "Lean at 45 and I eat carbs every day"

Viewer thinks: "Wait, how? That challenges what I believe. I need to know."

### Adding Context Checklist
- [ ] Does the viewer know WHY they should care in first frame?
- [ ] Is there a clear "what's in it for me" for target audience?
- [ ] Would someone scrolling fast understand the promise?

---

## Emotional Connection

For creator/personal brand content:

| Element | Purpose |
|---------|---------|
| Show yourself | Builds parasocial connection |
| Show emotion | Makes content relatable |
| Show journey | "10 years ago vs now" creates investment |
| Show personality | Differentiates from millions of others |

> "There's a gazillion [creators] out there and you're not special — until they understand your personality, your why, or the emotions behind it."

---

## Hook Scoring Checklist

Before publishing, score your hook:

| Criteria | Score (0-2) |
|----------|-------------|
| Opens a curiosity loop? | |
| Challenges a common belief? | |
| First frame is decluttered? | |
| Text is readable & in safe zone? | |
| Context is immediately clear? | |
| Plants a question in viewer's mind? | |
| Would YOU stop scrolling for this? | |

**Total: /14** — Aim for 10+ before publishing

---

## Prompt Template for AI Hook Generation

```javascript
// Code node: Generate hook ideas via AI
const hookPrompt = `
You are a viral content strategist. Create 5 scroll-stopping hooks for this topic:

Topic: ${$json.topic}
Target Audience: ${$json.audience}
Video Content: ${$json.contentSummary}

For each hook, provide:
1. TEXT HOOK (on-screen text, max 8 words, ends with "...")
2. VISUAL DESCRIPTION (first frame composition)
3. SPOKEN HOOK (first 3 seconds of speech)
4. OPEN LOOP (what question does this plant?)

Rules:
- Challenge common beliefs when possible
- Never reveal the answer in the hook
- Create FOMO if viewer swipes away
- Keep text SHORT and punchy

Output as JSON array.
`;
```

---

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Giving away the answer in hook | End with "..." to keep loop open |
| Too much text on screen | Max 8-10 words, use hierarchy |
| Text overlapping with visuals | Use safe zones, add background boxes |
| No context for cold viewers | Add "why should I care" context |
| Cluttered first frame | One focal point, blur/remove distractions |
| "Who can relate?" hooks | Too vague — be specific about WHAT |

---

## Integration with Video Pipeline

When generating scripts, structure with hooks first:

```javascript
const scriptStructure = {
  hook: {
    textOverlay: "This should NOT work...",
    spokenWords: "Everyone told me this wouldn't work, but...",
    visualDescription: "Close-up of result, key element slightly blurred"
  },
  content: [...],  // Main content
  payoff: {
    // Deliver on hook promise
    // Then open NEW loop for next video/follow
  }
};
```
