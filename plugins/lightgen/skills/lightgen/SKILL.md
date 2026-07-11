---
name: lightgen
description: Generate and edit images with Nano Banana Pro and create short videos with Gemini Omni Flash via the lightgen MCP tools. Use whenever the user asks to create, generate, draw, edit, restyle, combine, or upscale an image, or generate a short video, and the lightgen MCP server is connected.
---

# Generating images and videos with lightgen

lightgen exposes five MCP tools backed by Nano Banana Pro (Google's
`gemini-3-pro-image`) and Gemini Omni Flash. Each successful generation costs
prepaid credits: 1 credit at 1K/2K resolution, 2 credits at 4K, and 8 credits
for a short video. Failed generations are never charged. Results come back as
hosted URLs — embed images in your reply as markdown so the user sees them and
link videos directly.

## Choosing a tool

- **generate_image** — text-to-image. Use for anything created from scratch.
- **edit_image** — pass 1–14 https image URLs plus instructions. Use for
  restyling, background changes, combining references, or iterating on a
  previous lightgen result (use its URL from the earlier tool result or
  `list_generations`).
- **generate_video** — make a short 16:9 or 9:16 video from a prompt and
  optional reference images. It costs 8 credits and returns a hosted MP4.
- **get_account** — balance check. Call it when the user asks about credits,
  or after a generation fails for credit reasons.
- **list_generations** — recent images and videos with URLs. Use to find an
  earlier generation to edit, re-share, or use as a reference.

## Parameter guidance

- **aspect_ratio**: match intent, don't default blindly — `16:9` for
  hero/banner/landscape scenes, `9:16` for phone wallpapers and stories,
  `1:1` for avatars and icons, `3:2`/`2:3` for photographic prints.
- **resolution**: default `2K` (same price as 1K, better quality). Only use
  `4K` when the user explicitly wants maximum detail or print quality — it
  costs double. Use `1K` for quick drafts and iteration loops.
- **n**: generate variations (2–4) when the user is exploring a concept;
  each one is charged, so say so when you do it.
- **video aspect ratio**: use `16:9` for landscape and product demos or `9:16`
  for social and mobile. Tell the user that each video costs 8 credits before
  generating it.

## Writing good prompts

Nano Banana Pro rewards specific, photographic prompts. Include: subject,
setting, lighting, camera/style, and mood. It renders legible text well —
quote any words that must appear verbatim ("a neon sign reading 'OPEN'").
For edits, describe the change AND what must stay the same ("replace the sky
with a stormy sunset; keep the building and foreground unchanged").

## Iterating

Treat image work as a loop: generate at 2K, show the result, ask what to
change, then call edit_image with the previous image's URL rather than
regenerating from scratch — it preserves composition and identity. Pro
maintains consistency for up to ~5 people / 10 objects across edits.

## Credits

If a tool reports insufficient credits, relay its message verbatim — it
contains the top-up link and the free-grant offer (3 free credits after a $0
card verification at https://lightgen.app/dashboard). Don't retry until the
user has topped up.
