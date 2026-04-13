---
name: lightgen-setup
description: Set up LightGen by creating an account, purchasing image credits, and configuring your connection. Use when the user wants to get started with LightGen or buy more images.
---

# LightGen Setup

This skill handles LightGen account creation, credit purchases, and MCP configuration. Follow these steps exactly.

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
            print('CONFIGURED')
            break
    except: pass
else:
    print('NOT_CONFIGURED')
"
```

- If `CONFIGURED` — ask if they want to buy more images (skip to Step 4) or reconfigure.
- If `NOT_CONFIGURED` — continue to Step 2.

## Step 2: Create account

Ask the user for their email and password using AskUserQuestion.

Then sign up:

```bash
curl -s -X POST https://lightgen.app/auth/signup \
  -H "Content-Type: application/json" \
  -d '{"email": "USER_EMAIL", "password": "USER_PASSWORD"}'
```

This returns JSON with `access_token`, `refresh_token`, and `user`.

If the user already has an account, use login instead:

```bash
curl -s -X POST https://lightgen.app/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email": "USER_EMAIL", "password": "USER_PASSWORD"}'
```

Extract the `access_token` from the response. This is the JWT used for all subsequent requests.

## Step 3: Configure MCP connection

Store the JWT in the MCP config immediately so the connection works:

```bash
claude mcp add-json lightgen '{"type":"http","url":"https://lightgen.app/mcp","headers":{"Authorization":"Bearer JWT_HERE"}}' --scope user
```

Replace `JWT_HERE` with the actual access_token from Step 2.

Tell the user: "Account created and MCP configured. Now let's get you some image credits."

## Step 4: Purchase images

Present the pricing:

| Tier | Price | Images | Best for |
|------|-------|--------|----------|
| **Trial** | 9 dollars | 30 images | Try it out |
| **Starter** | 49 dollars | 200 images | Regular use |
| **Growth** | 79 dollars | 400 images | Best value — 2x images for only 30 dollars more |

Ask the user which tier using AskUserQuestion. Default to Trial.

Create checkout session (requires the JWT from Step 2):

```bash
curl -s -X POST https://lightgen.app/billing/checkout \
  -H "Authorization: Bearer JWT_HERE" \
  -H "Content-Type: application/json" \
  -d '{"tier": "TIER_HERE"}'
```

Extract `session_id` and `url` from the response.

## Step 5: Open checkout and wait

Tell the user you're opening the payment page. Run:

```bash
open "CHECKOUT_URL_HERE"
```

On Linux use `xdg-open`.

Tell the user: "Complete the payment in your browser. I'll wait for confirmation."

## Step 6: Poll for completion

Poll every 5 seconds, max 60 attempts (5 minutes):

```bash
for i in $(seq 1 60); do
  STATUS=$(curl -s "https://lightgen.app/billing/status/SESSION_ID_HERE" | python3 -c "import sys,json; print(json.load(sys.stdin).get('status',''))" 2>/dev/null)
  if [ "$STATUS" = "complete" ]; then
    echo "PAYMENT_COMPLETE"
    break
  fi
  sleep 5
done
```

## Step 7: Confirm

If payment complete, tell the user:

"You're all set! Your images have been added. Restart Claude Code (or run /reload-plugins) and ask me to generate any image."

If the user already had MCP configured (buying more images), just tell them the images have been added to their balance — no restart needed.
