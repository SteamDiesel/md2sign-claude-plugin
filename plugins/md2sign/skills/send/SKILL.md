---
name: send
description: Prepare a contract and send it for e-signature with md2sign, step by step with a preview and confirmation.
disable-model-invocation: true
argument-hint: "[markdown file, or describe the contract and signers]"
---

Get a document signed with md2sign. Follow the `contracts` skill for the markdown format and tool details.

Input: $ARGUMENTS

1. If the input is a file path, read it. If it describes a contract, draft it in md2sign markdown with one `[[signature: Full Name <email>]]` field per signer. If anything essential is missing (signer names, emails, key terms), ask before drafting.
2. Run `validate_markdown` and fix every error.
3. Run `render_preview`, summarise the document and list each signer with their email, and ask: "Send this to these signers?"
4. Only after the user says yes: make sure there's an account (use `get_account`; if unauthenticated, follow the account step in the `contracts` skill), then `create_draft` and `send_document`.
   - A new account's first sent document is free (up to 5 signers, no plan needed, email must be confirmed). `get_account` shows `free_document_available`. If it's their free document, tell them so before sending.
   - If sending is blocked by `email_unverified`, `free_document_limit` (more than 5 signers on the free document), `subscription_required` or `quota_exceeded`, explain what the user needs to do, give them the link from the error, and tell them the draft is saved.
5. Confirm what was sent and to whom, and offer to check signing progress later with `get_document`.

If the user only wants to check it first, offer `test_send_document` (a free watermarked copy to themselves) instead of sending.
