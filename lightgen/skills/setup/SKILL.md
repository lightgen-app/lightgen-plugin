---
name: lightgen-setup
description: Set up LightGen by purchasing image credits and configuring your API key. Use when the user wants to get started with LightGen or needs an API key.
---

# LightGen Setup

Help the user get started with LightGen image generation.

## If the user already has an API key

Tell them to reconfigure the plugin:

1. Run `/plugin` in Claude Code
2. Find LightGen in the Installed tab
3. Click Configure and enter their API key

Or via terminal:
```
claude mcp add --transport http lightgen https://lightgen.app/mcp --header "Authorization: Bearer THEIR_KEY" --scope user
```

## If the user needs to purchase images

Direct them to purchase via terminal:

```
npx @lightgen/cli setup
```

This will:
1. Open Stripe checkout in their browser
2. After payment, automatically configure the MCP connection
3. They can then start generating images immediately

## Pricing

| Tier | Price | Images |
|------|-------|--------|
| Trial | $9 | 30 |
| Starter | $49 | 200 |
| Growth | $79 | 400 |

Images never expire. To select a specific tier:
```
npx @lightgen/cli setup --tier starter
```

## After Setup

Once configured, the user can:
- Ask you to generate any image naturally
- Run `/lightgen-generate` for guided generation
- Run `/lightgen-balance` to check remaining images
