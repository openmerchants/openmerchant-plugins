---
name: check-vendor-payments
description: Check the connected business account's supplier bills, vendor invoices, payment status, holds, missing payout details, and stuck intake. Use for "which bills were paid", "why is this invoice stuck", or "what needs my review", even when OpenMerchant is not named. General accounting explanations and personal shopping recommendations do not need these account tools.
---

# Check vendor payments

This connector reads one OpenMerchant account's vendor payments: invoices, transfer
requests, and the payments made for them. It can also ask a vendor for missing payout
details. It can't submit an invoice, and it can't approve, pay, cancel, or retry a
payment; people do those in the OpenMerchant dashboard. The account and the test or
live mode come from the sign-in, not from the conversation.

## Start with the account evidence

For an account-status question, call the matching read tool without asking whether
to use the connector. Use an invoice ID already in the conversation with
`v1_get_invoice`; otherwise begin with `v1_list_invoices` and only known filters.
Do not ask for a vendor ID before making a useful list call. The connection supplies
the account and mode. If sign-in is required, follow the tool's authentication step.
For example, "Did our bills go through?" starts with invoice or transfer reads;
"What does accounts payable mean?" is an explanation and needs no account access.

## Find invoices

- `v1_list_invoices` lists the account's invoices. Filter by `status`, by
  `processing_status`, by `source` (`gmail`, `manual`, or `api`), or by vendor with
  `issuer_profile_id`. Page with `limit` and `offset`; the response has a `total`.
  Each row has its `status`, `review_reasons`, and `review_url`.
- `v1_get_invoice` reads one invoice: vendor, amounts, dates, `status`, why it waits
  for a person (`review_reasons`), its `review_url`, and the IDs of its payments.

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
  `needs_extraction_review`. Say why from each invoice's `review_reasons`, and give
  the user its `review_url`: that dashboard page is where a person decides.
- **Why is an invoice on hold?** Find it with `v1_list_invoices` (by vendor with
  `issuer_profile_id`, or by `status`), then read its `review_reasons`, on the row or
  from `v1_get_invoice`. Each reason has a `code`, such as `bank_details_changed`, and
  a plain-words `description`, such as "Bank details changed", the words the dashboard
  uses. They come most serious first, so lead with the first. The list is filled only
  while the invoice waits for a person. An invoice in `needs_extraction_review` with
  no reasons waits because some of its fields couldn't be read. Don't add reasons the
  results don't state.
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
- **Documents and evidence:** `v1_get_transfer_request` with `include` set to
  `documents`, `evidence`, or `submissions`, then `v1_list_transfer_documents`,
  `v1_list_transfer_evidence`, `v1_list_transfer_submissions`, and
  `v1_get_transfer_document` for more. Document reads return metadata, never the file.

## Ask a vendor for payout details

Do this only when the user asks.

1. Read the vendor's requirements with `v1_get_profile_payout_requirements`, or the
   invoice's with `v1_invoice_payout_requirements`. Use a requirement whose `actions`
   include `request_bank_details`.
2. Call `v1_post_payout_collection` with that requirement's `profile_id`, its
   `profile_updated_at` as `expected_updated_at`, and an `idempotency_key` that you
   reuse if you retry this same request.
3. OpenMerchant emails the vendor a secure link, and reuses an open request rather
   than sending another. A sent request doesn't mean the vendor has replied, and
   anything the vendor enters stays unverified until a person approves it.
4. On a `409` conflict, read the requirements again and use the new
   `profile_updated_at`.

When a requirement's `actions` include `check_for_updates`, for example after the
vendor says they have finished, `v1_post_recipient_refresh` reads the vendor's details
back from the payout provider, with the same `expected_updated_at` and
`idempotency_key` rules. It approves nothing.

## Rules

- Say an invoice is paid only when its `status` is `paid`.
- Never offer to approve, pay, cancel, or retry. Give the user the `review_url`
  instead.
- Never ask the user for bank account numbers. Vendors enter their own through the
  secure link.
