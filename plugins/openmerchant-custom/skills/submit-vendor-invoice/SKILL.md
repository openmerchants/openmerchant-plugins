---
name: submit-vendor-invoice
description: Add or submit a supplier bill or vendor invoice to the connected business account from supplied facts, a PDF or image, or both. Use for "add this bill", "submit this invoice", or "put this in our payment queue", even without naming OpenMerchant. Explaining or extracting an invoice alone does not authorize submission; existing payment-status questions use check-vendor-payments.
---

# Submit a vendor invoice

Submitting an invoice starts OpenMerchant's vendor payment process for the account.
Under the account's rules, known vendors within the account's limits are paid
automatically, without anyone approving that invoice. Everything else waits for a
person in the dashboard, including new vendors, new or changed bank details, and
amounts over a limit. Submit only when the user asks, and only with facts they gave
you or that appear on their document.

## Before you submit

- Confirm the vendor, invoice number, total, currency, and due date with the user.
- Tell the user plainly that an invoice from a known vendor within their limits may be
  paid automatically once it is submitted.
- Never invent or guess a missing fact. Leave the field out instead.
- Don't ask for the vendor's bank details. If they are missing, OpenMerchant can ask
  the vendor through a secure link (see the `check-vendor-payments` skill).

## Attach a document, if there is one

1. Call `v1_create_invoice_document` with the file name, its type
   (`application/pdf`, `image/jpeg`, `image/png`, or `image/tiff`), its size in bytes
   (at most 10,000,000), its SHA-256 hash, and an `idempotency_key`.
2. Upload the file yourself with an HTTP PUT to the returned URL, sending every signed
   header it returns. The URL expires after 15 minutes. Never put file contents in a
   tool argument.
3. Pass the returned document ID in `document_ids` when you create the invoice.

This needs an environment that can read the file and make HTTP requests, such as a
coding agent. Where you can't upload, submit the facts the user gives you instead.

## Create the invoice

Call `v1_create_invoice` with:

- `idempotency_key`: one stable key for this invoice, reused if you retry. A different
  payload under the same key is rejected.
- the facts you have: `invoice_number`, `issued_on`, `due_on`, `currency`,
  `total_minor_units` (minor units of the currency, so cents for USD), `line_items`,
  `document_ids`, and `issuer` (an existing vendor profile with `kind` `existing`, or
  a new vendor with `kind` `new`).
- `recipients` only when the user names who gets paid. With more than one recipient,
  their amounts must add up to the total.

`v1_create_transfer_request` takes the same facts as a transfer request of kind
`invoice_payment`, for users who work with transfer requests.

## Follow it

- `v1_get_invoice` shows the invoice's `status` and its submission's processing
  status. Processing (`queued`, `processing`, `needs_review`, `completed`, `ignored`,
  or `failed`) is about reading the invoice; `status` is its path to payment.
- When a person needs to act, give the user the invoice's `review_url`. Approvals
  happen in the dashboard, and no tool approves, pays, cancels, or retries a vendor
  payment.
- If intake holds the submission for missing facts, `v1_incomplete_intake_candidates`
  and `v1_intake_candidate` show what is missing.

## Changes that can also lead to a payment

Correcting held intake (`v1_correct_intake_candidate`), choosing a vendor's payout
account (`v1_select_invoice_destination`), re-checking a held invoice
(`v1_reevaluate_invoice`), and updating a transfer request
(`v1_update_transfer_request`) can each let an invoice continue to automatic payment.
Make one only on the user's explicit instruction, with facts the user confirmed.
