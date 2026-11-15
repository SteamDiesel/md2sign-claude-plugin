# md2sign plugin

Draft contracts in markdown and send them for legally binding e-signature with [md2sign](https://md2sign.com), without leaving Claude Code.

This plugin adds:

- **The md2sign MCP server** (`https://md2sign.com/mcp`). Claude can validate and preview a document with no account, and once you've connected your account it can convert your Word (.docx or .doc) contracts to markdown, save drafts, send a free watermarked test copy to you, send for signature (on the Team plans, optionally asking each signer to confirm a code emailed to them before their signature counts), track who has signed, download the signed PDF, delete drafts you don't need, and fix the name and business name your recipients see.
- **The `contracts` skill.** It teaches Claude the md2sign markdown format (one `[[signature: Full Name <email>]]` line per signer, plus `[[field: ...]]` blanks for signers to fill in) and a safe workflow: it validates, shows you a preview and asks before emailing anyone.
- **`/md2sign:send`**, a guided draft → preview → confirm → send command.

## Install

```
/plugin install md2sign --marketplace SteamDiesel/md2sign-claude-plugin
```

## Setup

When you install, Claude Code asks for your md2sign API key (from md2sign.com → API keys) and stores it securely. You can leave it blank and add it later with `/plugin configure md2sign@md2sign`. Without a key, validation and previews still work, and Claude can create an account for you with the `create_account` tool. Once your email is confirmed, choose a plan: **every plan starts with a 7-day free trial** (add a card, and you're not charged until it ends). Plans count envelopes, one per document sent however many sign it: Solo Lite US$45/year for 60 a year, Solo US$7.50/month for 30 a month, Team US$19/month for 100, Business+ US$39/month for 500.

**Two businesses?** Give Claude each md2sign account's API key and say which business a document is for. Claude passes that account's key on each call, so the right business sends it.

**Using claude.ai or Claude Desktop instead of Claude Code?** You don't need this plugin or an API key there. Add md2sign as a connector (Settings → Connectors → Add custom connector, address `https://md2sign.com/mcp/oauth`), then sign in to md2sign and click Approve.

**Spam protection.** Every email md2sign sends to a signer has a "Didn't expect this email? Report it" link. Recipients who report an account aren't emailed by it again, and if several recipients report an account, md2sign pauses its sending (Claude then gets a `sending_suspended` error) until md2sign reviews it. Only send documents to people who expect them: see the [anti-spam policy](https://md2sign.com/anti-spam-policy).

Privacy: https://md2sign.com/privacy · Terms: https://md2sign.com/terms · Support: support@md2sign.com
