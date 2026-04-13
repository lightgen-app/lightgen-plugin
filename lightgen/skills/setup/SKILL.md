---
name: lightgen-setup
description: Set up LightGen by purchasing image credits and configuring your API key. Use when the user wants to get started with LightGen or needs an API key.
---

# LightGen Setup

This skill handles the complete LightGen onboarding. Follow these steps exactly.

## Step 1: Check if already configured

Check if LightGen is already set up by looking for the API key:

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
        if auth and 'user_config' not in auth and auth.startswith('Bearer lg_'):
            print(f'CONFIGURED:{auth.replace(\"Bearer \", \"\")}')
            break
    except: pass
else:
    print('NOT_CONFIGURED')
"
```

- If `CONFIGURED:lg_xxx` — tell the user LightGen is already set up. Ask if they want to buy more images or reconfigure.
- If `NOT_CONFIGURED` — continue to Step 2.

## Step 2: Ask the user which tier they want

Present the pricing:

| Tier | Price | Images | Best for |
|------|-------|--------|----------|
| **Trial** | 9 dollars | 30 images | Try it out |
| **Starter** | 49 dollars | 200 images | Regular use |
| **Growth** | 79 dollars | 400 images | Best value — 2x images for only 30 dollars more |

Ask the user which tier they'd like. Default to Trial if unsure.

## Step 3: Create checkout session

```bash
curl -s -X POST https://lightgen.app/billing/checkout \
  -H "Content-Type: application/json" \
  -d '{"tier": "TIER_HERE"}'
```

This returns JSON with `session_id` and `url`. Extract both values.

## Step 4: Open checkout in browser

Tell the user you're opening the checkout page. Then run:

```bash
open "CHECKOUT_URL_HERE"
```

On Linux use `xdg-open` instead of `open`.

Tell the user: "Complete the payment in your browser. I'll wait for confirmation."

## Step 5: Poll for completion

Poll every 5 seconds until payment is confirmed. Maximum 60 attempts (5 minutes):

```bash
for i in $(seq 1 60); do
  RESULT=$(curl -s "https://lightgen.app/billing/status/SESSION_ID_HERE")
  STATUS=$(echo "$RESULT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('status',''))" 2>/dev/null)
  if [ "$STATUS" = "complete" ]; then
    API_KEY=$(echo "$RESULT" | python3 -c "import sys,json; print(json.load(sys.stdin).get('api_key',''))" 2>/dev/null)
    echo "COMPLETE:$API_KEY"
    break
  fi
  sleep 5
done
```

- If output contains `COMPLETE:lg_xxx` — extract the API key and continue to Step 6.
- If the loop ends without completing — tell the user payment wasn't detected and to run `/lightgen-setup` again.

## Step 6: Configure MCP server

Store the API key at the user scope using `claude mcp add-json`. This makes it available across all projects and stores the key directly in the config (not in the keychain):

```bash
claude mcp add-json lightgen '{"type":"http","url":"https://lightgen.app/mcp","headers":{"Authorization":"Bearer API_KEY_HERE"}}' --scope user
```

Replace `API_KEY_HERE` with the actual key from Step 5.

## Step 7: Confirm setup

Tell the user:

"LightGen is configured! Restart Claude Code (or run /reload-plugins) to activate, then ask me to generate any image."

## Buying more images (existing user)

If the user is already configured and wants more images:
1. Run Steps 3-5 only (create checkout, open browser, poll for completion)
2. Do NOT run Step 6 — their existing API key still works
3. Tell them the images have been added to their balance
