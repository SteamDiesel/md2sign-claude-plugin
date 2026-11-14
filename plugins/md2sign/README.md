# md2sign plugin

Draft contracts in markdown and send them for legally binding e-signature with [md2sign](https://md2sign.com), without leaving Claude Code.

This plugin adds:

- **The md2sign MCP server** (`https://md2sign.com/mcp`). Claude can validate and preview a document with no account, and once you've connected your account it can convert your Word (.docx or .doc) contracts to markdown, save drafts, send a free watermarked test copy to you, send for signature (on Pro, optionally asking each signer to confirm a code emailed to them before their signature counts), track who has signed, download the signed PDF, and delete drafts you don't need.
- **The `contracts` skill.** It teaches Claude the md2sign markdown format (one `[[signature: Full Name <email>]]` line per signer, plus `[[field: ...]]` blanks for signers to fill in) and a safe workflow: it validates, shows you a preview and asks before emailing anyone.
- **`/md2sign:send`**, a guided draft → preview → confirm → send command.

## Install

```
/plugin install md2sign --marketplace SteamDiesel/md2sign-claude-plugin
```

## Setup

When you install, Claude Code asks for your md2sign API key (from md2sign.com → API keys) and stores it securely. You can leave it blank and add it later with `/plugin configure md2sign@md2sign`. Without a key, validation and previews still work, and Claude can create an account for you with the `create_account` tool. Once your email is confirmed, your **first document is free** (up to 5 signers, no subscription). After that, sending needs a plan, from US$7.50/month for 50 signature requests.

**Two businesses?** Give Claude each md2sign account's API key and say which business a document is for. Claude passes that account's key on each call, so the right business sends it.

**Using claude.ai or Claude Desktop instead of Claude Code?** You don't need this plugin or an API key there. Add md2sign as a connector (Settings → Connectors → Add custom connector, address `https://md2sign.com/mcp/oauth`), then sign in to md2sign and click Approve.

Privacy: https://md2sign.com/privacy · Terms: https://md2sign.com/terms · Support: support@md2sign.com
