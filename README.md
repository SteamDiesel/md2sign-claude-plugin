# md2sign for Claude Code

Ask Claude to draft a contract and get it signed. This plugin connects Claude Code to [md2sign](https://md2sign.com): markdown in, legally binding e-signatures out, usually about three minutes from asking to sending.

- **MCP server**: `https://md2sign.com/mcp`, to import Word files, validate, preview, draft, test-send, send and track documents
- **`contracts` skill**: teaches Claude the md2sign markdown format and a safe workflow (it always previews and asks before emailing anyone)
- **`/md2sign:send`**: a guided "draft, preview, confirm, send" command

## Install

In Claude Code (2.1.275 or later):

```
/plugin install md2sign --marketplace SteamDiesel/md2sign-claude-plugin
```

Or in two steps on older versions:

```
/plugin marketplace add SteamDiesel/md2sign-claude-plugin
/plugin install md2sign@md2sign
```

Or from your terminal:

```bash
claude plugin marketplace add SteamDiesel/md2sign-claude-plugin
claude plugin install md2sign@md2sign
```

## Connect your account

Validation and previews work without an account. To save and send documents, add your API key when Claude Code asks during install. It's stored in your system's secure credential store. You can skip it and add or change it later:

```
/plugin configure md2sign@md2sign
```

Don't have an account? Just ask Claude to send something: it can create one for you with the `create_account` tool. You'll get a confirmation email. **Your first document is free**: send one real contract to up to 5 signers and get the sealed, signed PDF, with no subscription. After that, plans start at US$7.50/month for 50 signature requests. You can also sign up at [md2sign.com](https://md2sign.com) and create a key under **API keys**.

**Using claude.ai or Claude Desktop instead of Claude Code?** You don't need this plugin or an API key there. Add md2sign as a connector (Settings → Connectors → Add custom connector, address `https://md2sign.com/mcp/oauth`), then sign in to md2sign and click Approve.

## Try it

```
/md2sign:send A one-page NDA between me (Jane Citizen, jane@example.com) and Sam Lee (sam@acme.com), mutual, 2 years, NSW law
```

or just say: *"Draft a contractor agreement with Sam Lee for $4,800, 50% upfront, and get it signed by both of us."*

## How it works

You write (or Claude writes) normal markdown with one line for each person who needs to sign. Claude only adds you as a signer for two-sided agreements, and asks first; quotes, consents and acknowledgements just need the recipient:

```markdown
[[signature: Sam Lee <sam@acme.com> | role: Contractor]]
```

md2sign emails each signer a link to a clean signing page. Every signature is SHA-256 hash-chained and Ed25519 sealed, and everyone gets the signed PDF with an audit certificate. Anyone can check a document's seal at its public verification page.

Pro (US$26.41/month) adds team members, your logo on emails and PDFs, versioned templates with `{{ variables }}`, and webhooks that post signed contracts to your CRM or automations.

## Links

- Docs and API: https://md2sign.com/docs
- For agents: https://md2sign.com/mcp
- Support: support@md2sign.com

MIT licensed. md2sign is a service by md2sign.com; using it is subject to the [Terms](https://md2sign.com/terms) and [Privacy Policy](https://md2sign.com/privacy).
