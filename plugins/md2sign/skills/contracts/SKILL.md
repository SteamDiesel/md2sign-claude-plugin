---
name: contracts
description: Draft contracts and agreements in md2sign markdown and get them signed through the md2sign MCP tools. Use when the user asks to write, format, prepare, preview, send or track a contract, agreement, NDA, quote acceptance or any document that needs signatures, or mentions md2sign or e-signatures.
---

# md2sign: markdown contracts to e-signatures

md2sign turns a markdown document into a signing page and emails each signer a unique link. Signers type their name and consent; every signature is SHA-256 hash-chained and Ed25519 sealed, and everyone gets the signed PDF when the last person signs. The tools come from the `md2sign` MCP server bundled with this plugin.

## The markdown format

Standard GitHub-flavoured markdown (headings, lists, tables, bold), plus one signature field per signer, each on its own line:

```markdown
[[signature: Jane Citizen <jane@example.com>]]
[[signature: Sam Lee <sam@acme.com> | role: Director]]
```

- Use each signer's **full name** (first and last) and a real email. One field = one signer = one signature request on the plan.
- Optional hidden directives at the top (not shown to signers):
  - `<!-- md2sign:style=classic -->` (or `modern`, `formal`)
  - `<!-- md2sign:title=Website Design Agreement -->` (otherwise the first `#` heading is the title)
- Put signature fields at the end, under a heading such as `## Signatures`, with a bold label above each (e.g. `**Signed by the Client**`).
- Signers sign in parallel; there is no signing order.
- Never invent signer names or emails. Ask the user if they're missing.

Call `get_markdown_spec` for the full rules and an example.

## Workflow

1. **Draft** the document in markdown with the user's real terms. Keep it plain and readable; md2sign is not a law firm, so suggest the user has important contracts reviewed.
2. **Validate** with `validate_markdown`. It needs no account. Fix every error (line numbers are given) and mention any warnings.
3. **Preview** with `render_preview` (no account needed) and show the user what signers will see. Get their explicit go-ahead before anything is sent.
4. **Account.** Tools that save or send need an API key.
   - If tools return `unauthenticated` and the user has no account, call `create_account` with their name and email (and company name if they want it shown to signers). It returns an `api_key`. Pass it as the `api_key` argument to later tools in this session, and tell the user to save it by running `/plugin configure md2sign@md2sign` so it's sent automatically from then on.
   - If they already have an account, they can create a key at https://md2sign.com/app/api-keys.
   - The user must click the confirmation email. Their **first sent document is free** (up to 5 signers, no subscription), so they can see the whole flow work. After that they need a plan (Starter US$7.50/mo for 50 signature requests, Pro US$26.41/mo for 500). `get_account` returns `free_document_available`, plan, usage and checkout links.
5. **Draft and test.** `create_draft` saves it (edit later with `update_draft`). Offer `test_send_document`: a free, watermarked copy emailed only to the account owner.
6. **Send** with `send_document` only after the user confirms. It emails every signer. Then use `get_document` or `list_documents` to report who has signed, `remind_recipients` to nudge, `void_document` to cancel, and `get_audit_trail` for the evidence.

## Pro features

- **Templates:** `list_templates`, `get_template`, `save_template`, `create_document_from_template`. Templates use `{{ variable_name }}` placeholders, including in signature fields: `[[signature: {{ client_name }} <{{ client_email }}>]]`.
- **Webhooks:** `list_webhooks`, `save_webhook` post document events (with the markdown and, on completion, the signed PDF) to Zapier, Make, n8n, a CRM or the user's own server. Admins only.

## Errors you'll see

| `error` | What to do |
|---|---|
| `invalid_document` | Fix the listed markdown errors, then validate again |
| `unauthenticated` | Create an account or pass the user's API key (see step 4) |
| `email_unverified` | Ask the user to click the confirmation link in their email |
| `subscription_required` | Give the user a `checkout_urls` link from the error or `get_account` |
| `quota_exceeded` | Not enough signature requests left this month; suggest Pro |
| `not_a_draft` | Sent documents can't be edited; void and create a new one |
| `pro_required` | Templates and webhooks need the Pro plan |

Never send a document, remind signers or void a document without the user's explicit confirmation: these email real people.

Docs: https://md2sign.com/docs
