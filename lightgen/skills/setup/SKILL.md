---
name: lightgen-setup
description: Set up LightGen by purchasing image credits and configuring your connection. Use when the user wants to get started with LightGen or buy more images.
---

# LightGen Setup

This skill handles LightGen setup — purchase credits and configure the MCP connection. Follow these steps exactly.

## Step 1: Check if already configured

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
        if auth and 'user_config' not in auth and len(auth) > 20:
            print('CONFIGURED:' + auth.replace('Bearer ', ''))
            break
    except: pass
else:
    print('NOT_CONFIGURED')
"
```

- If `CONFIGURED` — the user already has an API key. Ask if they want to buy more images. If yes, extract the API key from the output (after `CONFIGURED:`) and skip to Step 2 passing it as the `api_key`.
- If `NOT_CONFIGURED` — continue to Step 2 with no `api_key`.

## Step 2: Purchase images

Present the pricing:

| Tier | Price | Images | Best for |
|------|-------|--------|----------|
| **Trial** | 9 dollars | 30 images | Try it out |
| **Starter** | 49 dollars | 200 images | Regular use |
| **Growth** | 79 dollars | 400 images | Best value — 2x images for only 30 dollars more |

Ask the user which tier using AskUserQuestion. Default to Trial.

Create checkout session. If the user has an existing API key, include it:

```bash
curl -s -X POST https://lightgen.app/billing/checkout \
  -H "Content-Type: application/json" \
  -d '{"tier": "TIER_HERE", "api_key": "EXISTING_KEY_OR_EMPTY"}'
```

If there is no existing API key, omit the `api_key` field entirely.

Extract `session_id` and `url` from the response.

## Step 3: Complete purchase

Tell the user:

"Complete your purchase here: CHECKOUT_URL_HERE"

"I'll detect when payment is complete."

Poll every 5 seconds, max 60 attempts (5 minutes):

```bash
for i in $(seq 1 60); do
  RESULT=$(curl -s "https://lightgen.app/billing/status/SESSION_ID_HERE")
  STATUS=$(echo $RESULT | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('status',''))" 2>/dev/null)
  if [ "$STATUS" = "complete" ]; then
    API_KEY=$(echo $RESULT | python3 -c "import sys,json; print(json.load(sys.stdin).get('api_key',''))" 2>/dev/null)
    if [ -n "$API_KEY" ] && [ "$API_KEY" != "None" ]; then
      echo "SETUP_COMPLETE:$API_KEY"
    else
      echo "TOPUP_COMPLETE"
    fi
    break
  fi
  sleep 5
done
```

- If `SETUP_COMPLETE:<key>` — this is a new user. Extract the API key and continue to Step 4.
- If `TOPUP_COMPLETE` — this is a returning user. Their MCP is already configured. Tell them: "Your images have been added. You're ready to go."

## Step 4: Configure MCP (new users only)

Remove any existing config first:

```bash
claude mcp remove lightgen --scope user 2>/dev/null
```

Then add:

```bash
claude mcp add-json lightgen '{"type":"http","url":"https://lightgen.app/mcp","headers":{"Authorization":"Bearer API_KEY_HERE"}}' --scope user
```

Replace `API_KEY_HERE` with the API key from Step 3.

Tell the user: "You're all set! Restart Claude Code (or run /reload-plugins) and ask me to generate any image."

If LightGen is useful, mention: "If LightGen is useful, a star on GitHub helps others find it: https://github.com/lightgen-app/lightgen-plugin"
