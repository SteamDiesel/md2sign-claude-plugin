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

- Use each signer's **full name** (first and last) and a real email. One field = one signer. A sent document uses one **envelope** on the plan, however many people sign it (up to 25 signers).
- Optional hidden directives at the top (not shown to signers):
  - `<!-- md2sign:style=classic -->` (or `modern`, `formal`)
  - `<!-- md2sign:title=Website Design Agreement -->` (otherwise the first `#` heading is the title)
  - `<!-- md2sign:order=sequential -->` (Team plans): signers sign one at a time, in the order their signature fields appear. Only the first is emailed when sent; each signature emails the next. Use it when order matters (e.g. the client signs before the director countersigns). Without it everyone is emailed at once.
  - `<!-- md2sign:verify=email -->` (Team plans): after typing their name, each signer is emailed a 6-digit code and must enter it within 15 minutes before their signature is recorded. It stops someone who was forwarded the signing link from signing as the recipient, and the confirmation time is sealed into the signature. `verify=none` turns it off for one document when the account has it on by default. Prefer the `verify_signers` argument (below) to typing the directive.
- Put signature fields at the end, under a heading such as `## Signatures`, with a bold label above each (e.g. `**Signed by the Client**`).

### Input fields (Team plans)

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
- **Team and Business+ plans only.** Sending a document with input fields returns `team_plan_required` on Solo Lite and Solo. Check `get_account` → `account.plan_features.input_fields` first; if it's false, leave blanks out (or ask the user to fill those details in themselves) unless they want to upgrade.
- Never invent signer names or emails. Ask the user if they're missing.

Call `get_markdown_spec` for the full rules and an example.

## Who needs to sign

Add a signature field only for people who must agree to the document. Don't add fields "just in case": every field emails a real person.

- **One-sided documents: only the recipient signs.** Quotes, proposals, consents, acknowledgements, waivers, one-way NDAs, engagement letters and offer or terms acceptances. Sending the document is the sender's offer or request, and the recipient's signature accepts it. Never add a field for the sender.
- **Two-sided agreements: usually every party signs, including the sender.** Services and contractor agreements, mutual NDAs, leases, partnerships, settlements. Ask the user "Do you want to sign this too?" and add them only if they say yes.
- **To add the sender,** pass `sender_signs: true` (and optionally `sender_role`, e.g. "Client") to `create_draft`, `update_draft` or `create_document_from_template`. md2sign adds a field with the account holder's name and email. Don't type the sender's field yourself; `sender_signs: false` removes it.
- Every validation result includes `signing_advice`: `document_kind` (`two_sided`, `one_sided` or `unknown`), `sender_should_sign` and a `message`. Use it, but the user's answer wins.

## Guide the user

Most people who ask for a contract aren't technical and won't know what to ask for. "Draft a contract for X and send it to Y" may be all you get. Lead them through it, and offer each of these before anything is sent:

- **Missing details:** ask in plain words for names, emails, amounts, dates and addresses. Don't leave `[PLACEHOLDERS]` in the document.
- **Look and style:** after `create_draft`, give them the `preview_url`. It opens in any browser for 7 days, needs no sign-in, and has a switcher to compare the Classic, Modern and Formal styles. Ask which one they like, then apply it with `update_draft` `style`.
- **Proofreading:** summarise the key terms, then list every term you added that they didn't ask for, any assumptions you made, and anything still missing.
- **Test copy:** offer `test_send_document`, so they see the exact email in their own inbox.
- **Team plan extras that fit:** input fields, signing in order, or `verify_signers` when it matters who signs.

Show documents only through `preview_url` or `render_preview`. Never point the user at files on their computer.

Send only when they clearly say to send.

**Changes after sending:** if the user wants a sent document changed, make the new version as a draft and share its `preview_url` first. Void the old one and send the new one only after they've seen it and said yes, even if they asked you to "re-send".

## Workflow

1. **Draft** the document in markdown with the user's real terms. Keep it plain and readable; md2sign is not a law firm, so suggest the user has important contracts reviewed.
   - **Word files:** if the user already has the contract as a .docx or .doc, don't retype it. Base64-encode the file and call `import_word_document` (needs an account). It returns `markdown`, a suggested `title` and `import_warnings` (images dropped, tracked changes accepted, old .doc files keep text, bold and italics only). Tell the user about the warnings, check the wording survived, then add the signature fields.
2. **Validate** with `validate_markdown`. It needs no account. Fix every error (line numbers are given), mention any warnings, and check `signing_advice` to decide whether the sender should sign (see "Who needs to sign").
3. **Preview** with `render_preview` (no account needed) and show the user what signers will see. Once the document is saved as a draft (step 5), give them its `preview_url` and ask which style they like. Get their explicit go-ahead before anything is sent.
4. **Account.** Tools that save or send need an API key.
   - If tools return `unauthenticated` and the user has no account, ask for their full name and business name (if they have one), then call `create_account` with those and their email. Recipients see emails from "<business> via md2sign", so never guess the name from the email address. It returns an `api_key`. Pass it as the `api_key` argument to later tools in this session, and tell the user to save it by running `/plugin configure md2sign@md2sign` so it's sent automatically from then on.
   - **Wrong name or business name** on an account (it shows in every email): fix it with `update_account` (`name`, `company_name`; an empty `company_name` removes it; only the company admin can change it).
   - **Company logo** (company admin; uploading needs Team or Business+, removing works on any plan): if the user wants their logo on emails, signing pages and PDFs, call `set_company_logo` with no arguments to get `upload_url`, a private link that works for 30 minutes. If you can run commands and have the file, upload it with `curl -F file=@logo.png "<upload_url>"` so the image never passes through the conversation; otherwise give the user the link to open and choose the file, and ask them to tell you when the page says "Logo saved" (curl gets a JSON reply instead). Only use `content_base64` for a small image you already have as bytes. Logos must be uploaded: never pass a link to an image on a website (it's rejected). PNG, JPG, WebP or GIF, up to 300 KB, no SVG; a wide logo about 400×100 px looks best. It replaces any existing logo and also applies to documents already sent; `remove: true` deletes it. `get_account` → `user.company_logo` only says whether some logo is set, so it can't confirm a replacement. Offer `test_send_document` so they see it in an email. The logo is stored privately and shown through signed links that can't be guessed, which md2sign only puts in the company's account, emails and documents. Before setting or removing it, tell the user it changes the logo on every email, signing page and PDF, including documents already sent, and get their go-ahead.
   - **Several accounts** (for example two businesses): keep each account's API key and pass the right one as `api_key` on every call. An `api_key` argument overrides the connection's own sign-in, including OAuth and a configured key. Check with `get_account` (`user.email`, `authenticated_with`) before you save or send. If a key is wrong you get `invalid_api_key`; the tool doesn't fall back to another account.
   - If they already have an account, they can create a key at https://md2sign.com/app/api-keys.
   - If md2sign is connected through OAuth (for example as a claude.ai connector at `https://md2sign.com/mcp/oauth`), tools are already signed in as the user: don't ask for a key or call `create_account`.
   - The user must click the confirmation email, then choose a plan at checkout. **Every plan starts with a 7-day free trial**: they add a card and aren't charged until it ends, and cancelling before then costs nothing. During the trial they can send 3 envelopes, each with up to 5 signers. There's one trial per person and company: check `get_account` → `account.trial.available` before promising one. If it's false, the plan is charged at checkout (the checkout page says so too).
   - Plans count **envelopes**: one envelope is one document sent, however many people sign it.
     - **Solo Lite** US$45/year for 60 envelopes a year (yearly only).
     - **Solo** US$7.50/mo for 30 envelopes a month.
     - **Team** US$19/mo for 100 a month, adding teammates on the company domain (no seat fees), the company logo, input fields, signing in order and signer email verification.
     - **Business+** US$39/mo for 500 a month, with everything in Team.
     - Yearly billing is 12 months for the price of 10 (US$75, US$190 or US$390 a year). Templates and webhooks come with every plan.
   - `get_account` returns `account.plan`, `account.trial` (`on_trial`, `ends_at`, `available`), `account.usage` (`envelopes_used`, `envelopes_allowance`, `envelopes_remaining`, `top_up_envelopes`, `max_signers_per_envelope`), `account.plan_features`, `checkout_urls` (monthly) and `annual_checkout_urls` (yearly; Solo Lite is only here). Help the user pick: Solo Lite for a handful of contracts a year, Solo for regular solo use, Team when they want teammates, branding or the extras above.
5. **Draft and test.** `create_draft` saves it (edit later with `update_draft`; `delete_document` permanently removes a draft the user no longer wants). Offer `test_send_document`: a free, watermarked copy emailed only to the account owner.
6. **Send** with `send_document` only after the user confirms. It emails every signer. Then use `get_document` or `list_documents` to report who has signed, `remind_recipients` to nudge, `void_document` to cancel, `get_audit_trail` for the evidence, and `get_document_pdf` for the PDF. It returns a `download_url` that needs no API key and expires after 30 minutes, so give it to the user or download it straight away; `signed_and_sealed` says whether everyone has signed.

## Your own references

When the user (or their system) has its own IDs for a document, such as a customer, job or quote number, attach them so the signed document can be matched up later:

- `metadata` on `create_draft`, `update_draft` or `create_document_from_template`: a flat object of up to 20 keys, e.g. `{"customer_number": "C-1001", "job_number": "J-77", "quote_number": "Q-310"}`. Keys start with a letter or `_`, then use letters, numbers, `_` and `-` (up to 40 characters); values are text or numbers up to 500 characters, stored as trimmed text (`77` comes back as `"77"`). No nested objects, lists or true/false. On `update_draft` it replaces the whole object; `null` clears it. It can't change once the document is sent.
- `external_reference`: a single ID string, if one is all they need.
- Both come back on every document and in every webhook (`document.metadata`, `document.external_reference`), so their CRM or automation can file the signed PDF against the right record.
- To find documents later, `list_documents` takes `metadata` (every given key must match exactly, case-sensitive) and `external_reference`, e.g. `metadata: {"customer_number": "C-1001"}`. Over REST: `GET /api/v1/documents?metadata[customer_number]=C-1001`.

Only add these when the user gives you their own IDs or asks for them; don't invent them.

## Plan features

Every plan includes templates and webhooks. Input fields, signing in order, signer email verification, teammates and the company logo need the Team or Business+ plan.

- **Templates (every plan):** `list_templates`, `get_template`, `save_template`, `create_document_from_template`, `delete_template` (the template's creator or the company admin; documents made from it are kept). Templates use `{{ variable_name }}` placeholders, including in signature fields: `[[signature: {{ client_name }} <{{ client_email }}>]]`.
- **Input fields:** `[[field: ...]]` blanks for signers to fill in (see the format above).
- **Signing in order:** `<!-- md2sign:order=sequential -->`. `get_document` returns `signing_order` and `next_signer`; `remind_recipients` only reminds the person whose turn it is.
- **Signer email verification:** pass `verify_signers: true` to `create_draft`, `update_draft` or `create_document_from_template` to make every signer confirm a code emailed to them before signing (`false` turns it off for that document; omit it to use the account default). Suggest it for high-value or sensitive agreements, or when the user worries about who actually signs. The user can turn it on for every document on their md2sign profile; `get_account` shows `user.verify_signers_by_default` and `account.plan_features.signer_verification`. Documents return `signer_verification` (`email` or `none`), fixed when sent. Tell the user signers will get a second email with a code, so they aren't surprised. It proves inbox access at signing time, not identity: don't call it two-factor or ID verification.
- **Webhooks (every plan; Solo Lite 1 endpoint, Solo 3, Team and Business+ 10):** `list_webhooks`, `save_webhook`, `delete_webhook` post document events (with the markdown and, on completion, the signed PDF) to Zapier, Make, n8n, a CRM or the user's own server. Admins only. To pause an endpoint without deleting it, use `save_webhook` with `is_active: false`.

## Errors you'll see

| `error` | What to do |
|---|---|
| `invalid_document` | Fix the listed markdown errors, then validate again |
| `unauthenticated` | Create an account or pass the user's API key (see step 4) |
| `nothing_to_update` | Call `update_account` with `name`, `company_name` or both |
| `invalid_api_key` | The `api_key` you passed isn't valid. Ask the user for the right key (md2sign.com/app/api-keys) |
| `email_unverified` | Ask the user to click the confirmation link in their email |
| `subscription_required` | The account has no plan yet. Give the user a `checkout_urls` or `annual_checkout_urls` link from the error or `get_account`; their first plan starts with a 7-day free trial if `account.trial.available` is true, otherwise it's charged at checkout |
| `quota_exceeded` | The plan's envelopes for this period are used up. The admin can buy extra envelopes (packs of 10 for US$5, using the `top_up_url` in the error) or upgrade; the allowance resets at `period_end` |
| `trial_limit` | The free trial allows 3 envelopes with up to 5 signers each. Remove signers, or wait for the trial to end (`trial_ends_at`); the admin can also end the trial early from Billing to start the full plan now |
| `not_a_draft` | Sent documents can't be edited or deleted; void and create a new one |
| `not_sent` | Drafts have no PDF yet; use `render_preview` or `test_send_document` |
| `admin_required` | Only the company admin (or, for a template, its creator) can do this, including changing the company name or logo; ask them |
| `not_found` | Wrong ID, or the document belongs to someone else in the company; check `list_documents` |
| `team_plan_required` | Input fields, signing in order, signer email verification, the company logo and adding teammates need the Team or Business+ plan; suggest upgrading, or remove the fields, the order directive or `verify=email` (`verify_signers: false`) |
| `webhook_limit` | The plan's webhook endpoints are all in use; delete one or upgrade |
| `unreadable_document` | The Word file couldn't be converted (damaged, password protected or not Word). Give the user the `message`; suggest saving as .docx |
| `sending_suspended` | Recipients reported this account's emails as unwanted, so md2sign has paused sending, reminders and test copies while it reviews the account. Don't retry or try another way to send. Tell the user plainly, and that they can still see their documents and appeal at the `appeal_email` in the error |
| `recipient_opted_out` | The listed `emails` reported this account's emails or bounced, so md2sign won't email them for this account. Remove them from the document and suggest the user contacts them another way first |
| `signup_blocked` | This email address (or its domain) can't be used for a new md2sign account. Give the user the `message`; don't try other addresses |

**Spam protection.** Every email md2sign sends to a recipient has a "Didn't expect this email? Report it" link. A recipient who reports (or marks it as spam) is never emailed by that account again and their unsigned document is declined, and repeated reports suspend the account's sending. So only help send documents to people who expect them: never to bought or scraped lists, as cold outreach or marketing, or to people the user doesn't know. See https://md2sign.com/anti-spam-policy.

Never send a document, remind signers or void a document without the user's explicit confirmation: these email real people. Confirm before deleting a draft, template or webhook too: deletes can't be undone.

Docs: https://md2sign.com/docs
