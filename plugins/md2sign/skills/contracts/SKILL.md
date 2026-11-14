---
name: contracts
description: Draft contracts and agreements in md2sign markdown and get them signed through the md2sign MCP tools. Use when the user asks to write, format, prepare, preview, send or track a contract, agreement, NDA, quote acceptance or any document that needs signatures, wants a Word (.docx or .doc) contract signed, or mentions md2sign or e-signatures.
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
  - `<!-- md2sign:order=sequential -->` (Pro): signers sign one at a time, in the order their signature fields appear. Only the first is emailed when sent; each signature emails the next. Use it when order matters (e.g. the client signs before the director countersigns). Without it everyone is emailed at once.
- Put signature fields at the end, under a heading such as `## Signatures`, with a bold label above each (e.g. `**Signed by the Client**`).

### Input fields (Pro)

When a signer must supply details (start date, job title, ABN, PO number, an "I have read X" tick), add an input field instead of leaving a blank line:

```
[[field: Start date | for: jane@example.com | type: date | required]]
[[field: PO number | for: any]]
[[field: I have read the safety policy | for: jane@example.com | type: checkbox | required]]
```

- `for:` is a signer's email (they must also have a signature field) or `any` (the default): any signer may fill it, the first to sign with a value locks it.
- `type:` is `text` (default), `date` or `checkbox`. Fields are optional unless `| required` is added.
- Fields can sit inline, mid-sentence. Values are sealed with that signer's signature, appear in the signed PDF, and come back in `input_fields` on the document.
- Don't use fields for things the sender already knows; write those into the document.
- **Never add a "date signed" field** (or a blank `Date: ______` line under a signature). Every signature already records a tamper-proof UTC timestamp that appears on the signature line and the certificate. A hand-typed date adds friction, and one that disagrees with the recorded time can undermine the signature. If a Word import or the user's draft has "Date:" lines in the signature blocks, remove them and explain why. Validation warns with `redundant_signing_date`. Date fields are for other dates (start date, date of birth).
- **Pro plan only.** Sending a document with input fields returns `pro_required` on other plans and on the free first document. Check `get_account` → `pro_features.input_fields` first; if it's false, leave blanks out (or ask the user to fill those details in themselves) unless they want to upgrade.
- Signers sign in parallel; there is no signing order.
- Never invent signer names or emails. Ask the user if they're missing.

Call `get_markdown_spec` for the full rules and an example.

## Who needs to sign

Add a signature field only for people who must agree to the document. Don't add fields "just in case": every field emails a real person and uses a signature request.

- **One-sided documents: only the recipient signs.** Quotes, proposals, consents, acknowledgements, waivers, one-way NDAs, engagement letters and offer or terms acceptances. Sending the document is the sender's offer or request, and the recipient's signature accepts it. Never add a field for the sender.
- **Two-sided agreements: usually every party signs, including the sender.** Services and contractor agreements, mutual NDAs, leases, partnerships, settlements. Ask the user "Do you want to sign this too?" and add them only if they say yes.
- **To add the sender,** pass `sender_signs: true` (and optionally `sender_role`, e.g. "Client") to `create_draft`, `update_draft` or `create_document_from_template`. md2sign adds a field with the account holder's name and email. Don't type the sender's field yourself; `sender_signs: false` removes it.
- Every validation result includes `signing_advice`: `document_kind` (`two_sided`, `one_sided` or `unknown`), `sender_should_sign` and a `message`. Use it, but the user's answer wins.

## Workflow

1. **Draft** the document in markdown with the user's real terms. Keep it plain and readable; md2sign is not a law firm, so suggest the user has important contracts reviewed.
   - **Word files:** if the user already has the contract as a .docx or .doc, don't retype it. Base64-encode the file and call `import_word_document` (needs an account). It returns `markdown`, a suggested `title` and `import_warnings` (images dropped, tracked changes accepted, old .doc files keep text, bold and italics only). Tell the user about the warnings, check the wording survived, then add the signature fields.
2. **Validate** with `validate_markdown`. It needs no account. Fix every error (line numbers are given), mention any warnings, and check `signing_advice` to decide whether the sender should sign (see "Who needs to sign").
3. **Preview** with `render_preview` (no account needed) and show the user what signers will see. Get their explicit go-ahead before anything is sent.
4. **Account.** Tools that save or send need an API key.
   - If tools return `unauthenticated` and the user has no account, call `create_account` with their name and email (and company name if they want it shown to signers). It returns an `api_key`. Pass it as the `api_key` argument to later tools in this session, and tell the user to save it by running `/plugin configure md2sign@md2sign` so it's sent automatically from then on.
   - If they already have an account, they can create a key at https://md2sign.com/app/api-keys.
   - If md2sign is connected through OAuth (for example as a claude.ai connector at `https://md2sign.com/mcp/oauth`), tools are already signed in as the user: don't ask for a key or call `create_account`.
   - The user must click the confirmation email. Their **first sent document is free** (up to 5 signers, no subscription), so they can see the whole flow work. After that they need a plan (Starter US$7.50/mo for 50 signature requests, Pro US$39/mo for 500). Yearly billing is 12 months for the price of 10 (US$75 or US$390 a year). `get_account` returns `free_document_available`, plan, usage, `checkout_urls` (monthly) and `annual_checkout_urls` (yearly).
5. **Draft and test.** `create_draft` saves it (edit later with `update_draft`; `delete_document` permanently removes a draft the user no longer wants). Offer `test_send_document`: a free, watermarked copy emailed only to the account owner.
6. **Send** with `send_document` only after the user confirms. It emails every signer. Then use `get_document` or `list_documents` to report who has signed, `remind_recipients` to nudge, `void_document` to cancel, `get_audit_trail` for the evidence, and `get_document_pdf` for the PDF. It returns a `download_url` that needs no API key and expires after 30 minutes, so give it to the user or download it straight away; `signed_and_sealed` says whether everyone has signed.

## Pro features

- **Templates:** `list_templates`, `get_template`, `save_template`, `create_document_from_template`, `delete_template` (the template's creator or the company admin; documents made from it are kept). Templates use `{{ variable_name }}` placeholders, including in signature fields: `[[signature: {{ client_name }} <{{ client_email }}>]]`.
- **Input fields:** `[[field: ...]]` blanks for signers to fill in (see the format above).
- **Signing in order:** `<!-- md2sign:order=sequential -->`. `get_document` returns `signing_order` and `next_signer`; `remind_recipients` only reminds the person whose turn it is.
- **Webhooks:** `list_webhooks`, `save_webhook`, `delete_webhook` post document events (with the markdown and, on completion, the signed PDF) to Zapier, Make, n8n, a CRM or the user's own server. Admins only. To pause an endpoint without deleting it, use `save_webhook` with `is_active: false`.

## Errors you'll see

| `error` | What to do |
|---|---|
| `invalid_document` | Fix the listed markdown errors, then validate again |
| `unauthenticated` | Create an account or pass the user's API key (see step 4) |
| `email_unverified` | Ask the user to click the confirmation link in their email |
| `subscription_required` | The free first document has been used. Give the user a `checkout_urls` link from the error or `get_account` |
| `quota_exceeded` | Not enough signature requests left this month; suggest Pro |
| `not_a_draft` | Sent documents can't be edited or deleted; void and create a new one |
| `not_sent` | Drafts have no PDF yet; use `render_preview` or `test_send_document` |
| `admin_required` | Only the company admin (or, for a template, its creator) can do this; ask them |
| `not_found` | Wrong ID, or the document belongs to someone else in the company; check `list_documents` |
| `pro_required` | Templates, webhooks, input fields and signing in order need the Pro plan; suggest Pro, or remove the fields or the order directive |
| `unreadable_document` | The Word file couldn't be converted (damaged, password protected or not Word). Give the user the `message`; suggest saving as .docx |
| `free_document_limit` | The free first document allows up to 5 signers. Remove signers or subscribe; the free document isn't used up |

Never send a document, remind signers or void a document without the user's explicit confirmation: these email real people. Confirm before deleting a draft, template or webhook too: deletes can't be undone.

Docs: https://md2sign.com/docs
