---
name: send
description: Prepare a contract and send it for e-signature with md2sign, step by step with a preview and confirmation.
disable-model-invocation: true
argument-hint: "[markdown or Word file, or describe the contract and signers]"
---

Get a document signed with md2sign. Follow the `contracts` skill for the markdown format and tool details. Only send to people who expect the document: every email lets the recipient report it as unwanted, and repeated reports suspend the account's sending.

Input: $ARGUMENTS

1. If the input is a .docx or .doc file, base64-encode it and convert it with `import_word_document` (see the `contracts` skill), then mention any `import_warnings`. If it's another file path, read it. If it describes a contract, draft it in md2sign markdown with one `[[signature: Full Name <email>]]` field per signer, and `[[field: Label | for: email]]` input fields for details a signer must supply (see the `contracts` skill). If anything essential is missing (signer names, emails, key terms), ask before drafting.
   - Add only the people who must agree. For quotes, consents, acknowledgements, waivers and one-way NDAs, only the recipient signs. For two-sided agreements (services or contractor agreements, mutual NDAs, leases, partnerships), ask "Do you want to sign this too?" and, if yes, pass `sender_signs: true` when creating the draft rather than writing their field yourself.
   - On the Team or Business+ plan, if the user wants extra assurance about who signs (or their account verifies signers by default), pass `verify_signers: true` (or `false` to skip it for this document) when creating the draft, and mention that each signer will get an emailed code to enter before their signature counts.
   - If the user mentions their own IDs for this document (a customer, job, quote or invoice number), pass them as `metadata` when creating the draft, e.g. `{"customer_number": "C-1001"}`, so they come back in webhooks and can be searched with `list_documents`.
2. Run `validate_markdown` and fix every error.
3. Run `render_preview`, summarise the document and list each signer with their email. Point out anything you added or assumed and anything still missing. Then save it with `create_draft` (with an account, see step 4) and share the `preview_url`, so the user can see it in a browser and compare styles. Offer a free `test_send_document` copy to their inbox. Then ask: "Send this to these signers?"
4. Only after the user says yes: make sure there's an account (use `get_account`; if unauthenticated, follow the account step in the `contracts` skill), then `send_document` (create the draft first if you haven't yet). If the user has more than one md2sign account, pass the right account's `api_key` on every call.
   - Sending needs a plan. A new account's first plan starts with a 7-day free trial (card at checkout, not charged until it ends; 3 envelopes with up to 5 signers each). `get_account` shows `account.trial` and `account.usage`. If they're on the trial, mention how many trial envelopes are left before sending.
   - If sending is blocked by `email_unverified`, `subscription_required` (no plan yet), `trial_limit` (more than 5 signers, or 3 envelopes used, during the trial), `quota_exceeded` (envelopes used up; the error has a `top_up_url` for extra envelopes) or `team_plan_required` (input fields, signing in order or signer verification on Solo Lite or Solo), explain what the user needs to do, give them the link from the error, and tell them the draft is saved.
   - If it's blocked by `recipient_opted_out`, those signers reported this account's emails (or their address bounced): remove them and suggest the user contacts them directly. If it's `sending_suspended`, recipients have reported the account and md2sign is reviewing it: don't retry, tell the user, and give them the `appeal_email` from the error.
5. Confirm what was sent and to whom, and offer to check signing progress later with `get_document`.
6. If the user later wants changes to the sent document, make a new draft, share its `preview_url`, and void and re-send only after they approve the new version.

If the user only wants to check it first, offer `test_send_document` (a free watermarked copy to themselves) instead of sending.
