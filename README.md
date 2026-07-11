# lightgen — Claude Code plugin marketplace

Image and video generation inside Claude via [lightgen](https://lightgen.app).

> This directory is published as the public repository
> `lightgen-app/lightgen-plugin` so users can install it.

## Install

```
/plugin marketplace add lightgen-app/lightgen-plugin
/plugin install lightgen@lightgen
```

Then run `/mcp` to authenticate (magic-link sign-in in your browser).

The plugin bundles the remote MCP connector (`https://lightgen.app/mcp`) and
a skill that teaches Claude prompt-writing, image and video format choices,
credit-aware iteration, and the required final step of downloading generated
media into the current project when filesystem access is available.

New accounts get 3 free credits after a $0 card verification. Credit packs
start at $10 for 50 credits — see https://lightgen.app/pricing.
