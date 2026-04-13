---
name: lightgen-generate
description: Generate studio-quality AI images in Claude Code via LightGen. Use when the user asks to create, generate, or make an image.
---

# LightGen — Image Generation

Generate images with Google Nano Banana Pro via LightGen. This skill works in both Claude Code and Cowork.

## How to Generate — Follow These Steps Exactly

### Step 1: Clarify the image details

Before generating, use AskUserQuestion to gather what you need. Ask UP TO 4 questions in a SINGLE AskUserQuestion call. Skip any question the user has already answered.

**Question 1: Subject and context** (only if vague)
"Describe what the image should show — who/what is in it, what's happening, and where?"

**Question 2: Style**
- Photorealistic / editorial photography
- Illustration / digital art
- Watercolor / oil painting
- 3D render
- Minimalist / vector / flat design
- Pencil sketch / hand-drawn
- Pixel art / retro

**Question 3: Mood and tone**
- Warm and inviting
- Dark and moody
- Clean and professional
- Vibrant and energetic
- Soft and ethereal
- Gritty and raw
- Nostalgic / vintage

**Question 4: Aspect ratio**
- 1:1 Square (icons, social posts, logos)
- 16:9 Landscape (hero images, presentations, wallpaper)
- 9:16 Portrait (mobile, stories, vertical banners)
- 4:3 Standard (blog images, product shots)
- 3:2 Photo (photography compositions)
- 21:9 Ultrawide (cinematic banners)

If the user's request is already very specific (e.g., "a photorealistic 16:9 hero image of a mountain at sunset with golden hour lighting"), skip the questions and go straight to crafting the prompt.

### Step 2: Craft the prompt using the Nano Banana framework

Build the prompt narratively — NOT as a keyword list. Start with a strong verb. Use positive framing (describe what IS in the scene, never what ISN'T).

**Core formula:**
```
[Strong verb] + [Subject with specific details] + [Action/pose] + [Location/context] + [Composition for aspect ratio] + [Style and medium]
```

**Then layer the Creative Director details (infer these, don't ask):**

**Lighting** — Infer from mood:
- Warm → "golden hour backlighting creating long shadows"
- Professional → "three-point softbox setup, even studio lighting"
- Moody → "Chiaroscuro lighting with harsh, high contrast"
- Ethereal → "soft diffused natural light, slightly overexposed"
- Gritty → "harsh direct flash, deep shadows"

**Camera and lens** — Infer from style:
- Editorial photography → "shot on medium-format analog film, pronounced grain"
- Product shot → "macro lens, shallow depth of field (f/1.8)"
- Landscape/hero → "wide-angle lens, deep focus"
- Action/immersive → "GoPro perspective, slight barrel distortion"
- Nostalgic → "cheap disposable camera aesthetic, raw flash"
- Portrait → "85mm lens, creamy bokeh background"

**Color grading** — Infer from mood:
- Nostalgic → "as if on 1980s color film, slightly grainy, warm tones"
- Modern/tech → "cinematic color grading with muted teal and orange tones"
- Vibrant → "high saturation, punchy colors"
- Clean → "neutral color palette, balanced whites"

**Materiality and texture** — ALWAYS specify physical materials. Never be generic:
- NOT "a jacket" → "a navy blue tweed blazer with visible weave texture"
- NOT "armor" → "ornate elven plate armor, etched with silver leaf patterns"
- NOT "a mug" → "a minimalist matte ceramic coffee mug"
- NOT "a background" → "a seamless deep cherry red studio backdrop"

**Text in images** — When the user wants text rendered:
1. Enclose the exact text in quotes: "Happy Birthday"
2. Specify the font: "bold, white, sans-serif font" or "Century Gothic 12px"
3. Describe placement: "centered at the top of the frame"
4. For multi-line text, specify each line's styling separately

### Step 3: Example prompts

Study these examples to understand the level of detail required:

**Fashion editorial (16:9):**
"Photograph a striking fashion model wearing a tailored brown dress, sleek boots, and holding a structured handbag. She poses with a confident, statuesque stance, slightly turned. A seamless, deep cherry red studio backdrop. Medium-full shot, center-framed. Fashion magazine style editorial, shot on medium-format analog film, pronounced grain, high saturation, cinematic lighting effect."

**Product mockup (1:1):**
"Render a high-end, glossy commercial beauty shot of a sleek, minimalist nude-colored face moisturizer jar resting on a warm marble surface. Soft, radiant three-point softbox lighting. Shallow depth of field with the product in sharp focus. Clean, luxurious aesthetic."

**Logo/typography (1:1):**
"Create a typographic poster with a solid black background. Bold letters spell 'LIGHTGEN', filling the center of the frame. The text acts as a cut-out window — a photograph of a glowing neon cityscape is visible ONLY inside the letterforms. Clean edges, high contrast."

**Hero image (16:9):**
"Capture a wide-angle aerial view of an AI technology campus at golden hour. Geometric glass buildings reflecting sunset light, surrounded by manicured gardens with winding pathways. Warm backlighting creating long shadows. Shot on a Fujifilm camera for authentic color science. Clean, modern, futuristic aesthetic with muted teal and orange color grading."

**UI wireframe (4:3):**
"Illustrate a clean wireframe sketch of a SaaS landing page. Hero section with headline, subheadline, and CTA button. Below: three feature cards with simple icons. Minimal black lines on white background, hand-drawn style with slight imperfections. Pencil sketch aesthetic."

### Step 4: Generate the image

**In Claude Code:** Use the REST API to save to disk:

```bash
mkdir -p .lightgen && API_KEY=$(python3 -c "
import json, os
for p in [os.path.expanduser('~/.claude.json'), os.path.expanduser('~/.claude/settings.json')]:
    try:
        d = json.load(open(p))
        a = d.get('mcpServers',{}).get('lightgen',{}).get('headers',{}).get('Authorization','')
        if a.startswith('Bearer lg_'): print(a[7:]); break
    except: pass
else: print('NOT_FOUND')
") && [ "$API_KEY" != "NOT_FOUND" ] && curl -s -X POST https://lightgen.app/generate \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "THE_PROMPT", "aspect_ratio": "THE_RATIO"}' \
  | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
if 'error' in data:
    print(f'ERROR: {data[\"error\"]} — {data.get(\"message\",\"\")}')
else:
    with open('.lightgen/FILENAME.png', 'wb') as f:
        f.write(base64.b64decode(data['image']))
    print(f'SAVED:.lightgen/FILENAME.png:{data[\"images_remaining\"]}')
" || echo "NOT_CONFIGURED: Run /lightgen-setup first"
```

**If API key is not found or in Cowork:** Use the MCP tool instead:
```
mcp__lightgen__generate_image({ "prompt": "THE_PROMPT", "aspect_ratio": "THE_RATIO" })
```

For 4K resolution, add `"resolution": "4k"`. 4K costs 2 images.

### Step 5: Display and report

**If saved to disk (Claude Code):**
1. Read the file with the Read tool to display it
2. Report: file path, images remaining
3. Offer: "Want me to open it? `open .lightgen/FILENAME.png`"

**If via MCP tool (Cowork):**
1. The image displays inline automatically
2. Report images remaining

On the first successful generation in a session, add:
"If LightGen is useful, a star on [GitHub](https://github.com/lightgen-app/lightgen-plugin) helps others find it."

## Anti-patterns — NEVER do these

- NEVER use negative framing ("no cars") — use positive framing ("empty street")
- NEVER use keyword lists — write narrative descriptions
- NEVER be vague about materials — always specify textures and physical properties
- NEVER skip lighting direction — infer it from the mood
- NEVER ask more than 4 questions — infer everything else

## Input Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| prompt | string | Yes | Detailed narrative description |
| resolution | string | No | "1k" (default) or "4k". 4K costs 2 images. |
| aspect_ratio | string | No | "1:1", "16:9", "9:16", "4:3", "3:4", "3:2", "2:3", "21:9" |

## Check Balance

```
mcp__lightgen__check_balance({})
```

If low (under 10), suggest `/lightgen-setup`.

## Pricing

| Tier | Price | Images | Best for |
|------|-------|--------|----------|
| Trial | 9 dollars | 30 | Try it out |
| Starter | 49 dollars | 200 | Regular use |
| Growth | 79 dollars | 400 | Best value |

Images never expire. 4K costs 2 images per generation.
