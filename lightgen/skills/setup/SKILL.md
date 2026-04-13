---
name: lightgen-setup
description: Set up LightGen by purchasing image credits and configuring your API key. Use when the user wants to get started with LightGen or needs an API key.
---

# LightGen Setup

Help the user get started with LightGen image generation.

## New User Flow

1. Ask the user which tier they'd like:

| Tier | Price | Images | Per Image |
|------|-------|--------|-----------|
| Trial | $9 | 30 | $0.30 |
| Starter | $49 | 200 | $0.245 |
| Growth | $79 | 400 | $0.198 |

2. Call the LightGen checkout API to create a payment session:

```bash
curl -s -X POST https://lightgen.app/billing/checkout \
  -H "Content-Type: application/json" \
  -d '{"tier": "trial"}'
```

This returns `{ "session_id": "...", "url": "..." }`.

3. Tell the user to open the checkout URL in their browser to complete payment.

4. After payment, poll for the API key:

```bash
curl -s https://lightgen.app/billing/status/SESSION_ID_HERE
```

This returns `{ "status": "pending" }` or `{ "status": "complete", "api_key": "lg_xxx" }`.

5. Once you have the API key, tell the user to configure the plugin:
   - Run `/plugin` in Claude Code
   - Find LightGen in the Installed tab
   - Click Configure and enter the API key

   Or via terminal:
   ```
   claude mcp add --transport http lightgen https://lightgen.app/mcp --header "Authorization: Bearer API_KEY_HERE" --scope user
   ```

6. Tell them to restart Claude Code. They're ready to generate images.

## Existing User (already has an API key)

Tell them to configure the plugin:

1. Run `/plugin` in Claude Code
2. Find LightGen in the Installed tab
3. Click Configure and enter their API key

Or via terminal:
```
claude mcp add --transport http lightgen https://lightgen.app/mcp --header "Authorization: Bearer THEIR_KEY" --scope user
```

## Buying More Images

Same flow as new user — call the checkout API, complete payment. Images are added to their existing balance automatically.
