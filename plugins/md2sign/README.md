# md2sign plugin

Draft contracts in markdown and send them for legally binding e-signature with [md2sign](https://md2sign.com), without leaving Claude Code.

This plugin adds:

- **The md2sign MCP server** (`https://md2sign.com/mcp`). Claude can validate and preview a document with no account, and once you've connected your account it can save drafts, send a free watermarked test copy to you, send for signature, and track who has signed.
- **The `contracts` skill.** It teaches Claude the md2sign markdown format (one `[[signature: Full Name <email>]]` line per signer) and a safe workflow: it validates, shows you a preview and asks before emailing anyone.
- **`/md2sign:send`**, a guided draft → preview → confirm → send command.

## Install

```
/plugin install md2sign --marketplace SteamDiesel/md2sign-claude-plugin
```

## Setup

Set `MD2SIGN_API_KEY` to your md2sign API key (from md2sign.com → API keys) and restart Claude Code. Without a key, validation and previews still work, and Claude can create an account for you with the `create_account` tool. Sending needs a confirmed email and a plan, from US$4.76/month for 50 signature requests.

Privacy: https://md2sign.com/privacy · Terms: https://md2sign.com/terms · Support: support@md2sign.com
