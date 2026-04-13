---
name: lightgen-generate
description: Generate studio-quality AI images in Claude Code via LightGen. Use when the user asks to create, generate, or make an image.
---

# LightGen — Image Generation

Generate images with Google Nano Banana Pro via the LightGen REST API. Images are saved as files to `.lightgen/` in the current working directory.

## How to Generate — Follow These Steps Exactly

### Step 1: Get the API key

The API key is stored in the Claude MCP config after running `/lightgen-setup`. Read it:

```bash
python3 -c "
import json, os
for path in [os.path.expanduser('~/.claude.json'), os.path.expanduser('~/.claude/settings.json')]:
    try:
        data = json.load(open(path))
        servers = data.get('mcpServers', {})
        lg = servers.get('lightgen', {})
        headers = lg.get('headers', {})
        auth = headers.get('Authorization', '')
        if auth and auth.startswith('Bearer lg_'):
            print(auth.replace('Bearer ', ''))
            break
    except: pass
else:
    print('NOT_FOUND')
"
```

If `NOT_FOUND` — tell the user to run `/lightgen-setup` first, then stop.

### Step 2: Clarify the image details

Before generating, use AskUserQuestion to clarify these details (unless the user's request is already very specific):

**Question 1: Subject** — "What exactly should the image show?" (only if vague)

**Question 2: Style** — Ask the user to pick a style:
- Photorealistic
- Illustration
- Watercolor
- Digital art
- 3D render
- Minimalist/vector
- Pixel art
- Pencil sketch

**Question 3: Mood** — Ask the user to pick a mood/tone:
- Warm and inviting
- Dark and moody
- Clean and professional
- Vibrant and energetic
- Soft and ethereal
- Gritty and raw

**Question 4: Aspect ratio** — Ask the user what format they need:
- 1:1 Square (default — icons, social posts, logos)
- 16:9 Landscape (hero images, presentations, desktop wallpaper)
- 9:16 Portrait (mobile, stories, vertical banners)
- 4:3 Standard (blog images, product shots)
- 3:2 Photo (photography-style compositions)

Skip questions where the user has already provided enough detail. For example, if they say "a photorealistic 16:9 hero image of a mountain at sunset" — you have everything, just generate it.

### Step 3: Craft the prompt

Combine the user's answers into a detailed prompt using this formula:

```
[Subject with specific details] + [Style] + [Composition appropriate for aspect ratio] + [Lighting] + [Mood/Atmosphere] + [Background/Context]
```

**Always infer** lighting, composition, and background from the style and mood — don't ask about these separately.

Examples:
- User: "logo for my coffee shop Bean There", Style: minimalist, Mood: warm, Ratio: 1:1
  Prompt: "A minimalist logo for a coffee shop called 'Bean There'. Clean typography with a coffee bean icon integrated into the letter B. Warm brown and cream color palette on a white background. Vector style, centered composition."

- User: "hero image for AI startup", Style: digital art, Mood: professional, Ratio: 16:9
  Prompt: "A wide-angle hero image for an AI technology startup. Abstract neural network visualization with glowing blue and purple nodes connected by light trails. Dark background with subtle gradient. Clean, modern, futuristic aesthetic. Studio-quality digital art, 16:9 composition with breathing room for text overlay."

### Step 4: Generate and save

Choose a short descriptive filename (lowercase, hyphens, no spaces). Run this command, replacing API_KEY, PROMPT, ASPECT_RATIO, and FILENAME:

```bash
mkdir -p .lightgen && API_KEY="THE_KEY" && curl -s -X POST https://lightgen.app/generate \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"prompt": "THE_PROMPT", "aspect_ratio": "ASPECT_RATIO"}' \
  | python3 -c "
import sys, json, base64
data = json.load(sys.stdin)
if 'error' in data:
    print(f'ERROR: {data[\"error\"]} — {data.get(\"message\",\"\")}')
else:
    with open('.lightgen/FILENAME.png', 'wb') as f:
        f.write(base64.b64decode(data['image']))
    print(f'SAVED:.lightgen/FILENAME.png:{data[\"images_remaining\"]}')
"
```

For 4K resolution, add `"resolution": "4k"` to the JSON body. 4K costs 2 images.

### Step 5: Handle the result

Parse the output:
- If `ERROR:` — report the error. If `insufficient_images`, suggest `/lightgen-setup`.
- If `SAVED:path:remaining` — continue to Step 6.

### Step 6: Display the image

Use the Read tool to display the saved file to the user.

Then report:
- The file path
- Images remaining
- Offer to open: "Want me to open it? `open .lightgen/FILENAME.png`"

On the first successful generation in a session, add:
"If LightGen is useful, a star on [GitHub](https://github.com/lightgen-app/lightgen-plugin) helps others find it."

## Input Parameters

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| prompt | string | Yes | Detailed text description of the image |
| resolution | string | No | "1k" (default) or "4k". 4K costs 2 images. |
| aspect_ratio | string | No | "1:1" (default), "16:9", "9:16", "4:3", "3:4", "3:2", "2:3" |

## Check Balance

Use the MCP tool:
```
mcp__lightgen__check_balance({})
```

If low (under 10), suggest `/lightgen-setup` to purchase more.

## Pricing

| Tier | Price | Images | Best for |
|------|-------|--------|----------|
| Trial | 9 dollars | 30 | Try it out |
| Starter | 49 dollars | 200 | Regular use |
| Growth | 79 dollars | 400 | Best value |

Images never expire. 4K costs 2 images per generation.
