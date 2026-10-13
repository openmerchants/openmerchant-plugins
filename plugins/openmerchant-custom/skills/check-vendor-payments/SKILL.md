---
name: check-vendor-payments
description: Answer questions about an OpenMerchant account's vendor invoices and payments, such as what got paid, what is scheduled, what is waiting for a person, and which vendors still need to send payout details. Use when the user asks about bills, vendor invoices, payables, or payment status in OpenMerchant.
---

# Check vendor payments

This connector carries every OpenMerchant API tool for one account, including changes
that can lead to a payment, such as submitting an invoice (see the
`submit-vendor-invoice` skill). To answer questions about status, use only the reads
below. No tool approves, pays, cancels, or retries a vendor payment; people do those in
the OpenMerchant dashboard. The account and the test or live mode come from the
sign-in or API key, not from the conversation.

## Find invoices

- `v1_list_invoices` lists the account's invoices. Filter by `status`, by
  `processing_status`, by `source` (`gmail`, `manual`, or `api`), or by vendor with
  `issuer_profile_id`. Page with `limit` and `offset`; the response has a `total`.
- `v1_get_invoice` reads one invoice: vendor, amounts, dates, `status`, its
  `review_url`, and the IDs of its payments.

An invoice has two separate lifecycles, so don't mix them up:

- `status` is its path to payment: `received`, `extracted`, `evaluating`, `proposed`,
  `awaiting_confirmation`, `awaiting_human`, `scheduled`, `awaiting_funds`,
  `awaiting_provider_approval`, `payment_processing`, then `paid`. Other outcomes are
  `needs_extraction_review`, `duplicate`, `rejected`, `failed`, `returned`, and
  `cancelled`.
- The submission's processing status describes reading the invoice, never paying it:
  `queued`, `processing`, `needs_review`, `completed`, `ignored`, or `failed`.

Amounts are in minor units of the invoice's currency, so 12345 with `USD` is $123.45.

## Answer common questions

- **What got paid?** `v1_list_invoices` with `status` `paid`.
- **What's scheduled?** `status` `scheduled` or `awaiting_funds`.
- **What's waiting for a person?** `status` `awaiting_human` or
  `needs_extraction_review`. Give the user each invoice's `review_url`: that dashboard
  page shows why it waits, and it is where a person decides. The tools say that an
  invoice is waiting but not always why, so don't guess a reason the results don't
  state.
- **What happened to a payment?** `v1_list_transfer_requests` finds the transfer
  request; `v1_get_transfer_request` with `include` set to `executions` shows its
  payment attempts, and `v1_list_transfer_executions` pages through more. Each attempt
  has a status, amount, payment rail, dates, the last four digits of the payout
  account, and failure or return reasons when there are any.
- **Which vendors still need payout details?** `v1_invoice_payout_requirements` for an
  invoice, or `v1_get_profile_payout_requirements` for a vendor. Each requirement says
  what is missing, who can fix it, and which actions fix it.
  `v1_get_payout_collection` shows the latest request already sent to that vendor.
- **What is stuck in intake?** `v1_incomplete_intake_candidates` lists submissions
  that need more facts before processing can continue, and `v1_intake_candidate`
  reads one. They aren't approved or payable invoices.

## Ask a vendor for payout details

Do this only when the user asks. It can't lead to a payment.

1. Read the vendor's requirements with `v1_get_profile_payout_requirements`, or the
   invoice's with `v1_invoice_payout_requirements`. Use a requirement whose `actions`
   include `request_bank_details`.
2. Call `v1_post_payout_collection` with that requirement's `profile_id`, its
   `profile_updated_at` as `expected_updated_at`, and an `idempotency_key` that you
   reuse if you retry this same request.
3. OpenMerchant emails the vendor a secure link, and reuses an open request rather
   than sending another. Anything the vendor enters stays unverified until a person
   approves it.
4. On a `409` conflict, read the requirements again and use the new
   `profile_updated_at`.

## Rules

- Say an invoice is paid only when its `status` is `paid`.
- When answering a question, don't call a tool that changes anything. Changes that can
  lead to a payment need the user's explicit instruction.
- Never ask the user for bank account numbers. Vendors enter their own through the
  secure link.
