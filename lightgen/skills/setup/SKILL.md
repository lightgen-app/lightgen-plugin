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

## Step 2: Sign in via browser

Create an auth session, then open the browser for the user to sign in or create an account:

```bash
curl -s -X POST https://lightgen.app/auth/session
```

This returns JSON with `session_token` and `url`. Extract both values.

Tell the user: "Opening your browser to sign in or create your account..."

Open the login URL:

```bash
open "LOGIN_URL_HERE"
```

On Linux use `xdg-open`.

Tell the user: "Complete sign in in your browser. I'll wait."

Poll for completion every 3 seconds, max 100 attempts (5 minutes):

```bash
for i in $(seq 1 100); do
  RESULT=$(curl -s "https://lightgen.app/auth/status/SESSION_TOKEN_HERE")
  STATUS=$(echo $RESULT | python3 -c "import sys,json; print(json.load(sys.stdin).get('status',''))" 2>/dev/null)
  if [ "$STATUS" = "complete" ]; then
    echo $RESULT | python3 -c "import sys,json; print('AUTH_COMPLETE:' + json.load(sys.stdin).get('access_token',''))"
    break
  elif [ "$STATUS" = "expired" ]; then
    echo "AUTH_EXPIRED"
    break
  fi
  sleep 3
done
```

- If `AUTH_COMPLETE` — extract the access_token from after the colon. This is the JWT for all subsequent requests.
- If `AUTH_EXPIRED` — tell the user the session expired and ask them to try again.

## Step 3: Configure MCP connection

Store the JWT in the MCP config so the connection works:

```bash
claude mcp add-json lightgen '{"type":"http","url":"https://lightgen.app/mcp","headers":{"Authorization":"Bearer ACCESS_TOKEN_HERE"}}' --scope user
```

Replace `ACCESS_TOKEN_HERE` with the access_token from Step 2.

If the MCP server already exists, remove it first:

```bash
claude mcp remove lightgen --scope user
```

Then add it again.

Tell the user: "You're connected. Now let's get you some image credits."

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
  -H "Authorization: Bearer ACCESS_TOKEN_HERE" \
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
