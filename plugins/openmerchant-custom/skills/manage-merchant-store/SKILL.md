---
name: manage-merchant-store
description: Inspect or configure the connected account's own merchant catalog, product variants, locations, markets, delivery rate cards, promotions, quote responders, and publication readiness. Use for "why is my store not ready", "update our product listing", or "test our webhook" without requiring the OpenMerchant name. Buying from other merchants uses find-business-purchases.
---

# Manage a merchant store

Start setup or readiness questions with `v1_get_merchant`. For example, "Why can't
customers find my store?" needs the merchant's current blockers and verification
evidence before suggesting a configuration change. Read a known catalog item before
editing it; use the editable catalog-management tools for the account's own items,
and buyer catalog search for things the user wants to purchase.

For integration work, use `v1_search_merchant_docs`, then `v1_get_merchant_doc` for
the relevant current contract. "What is wrong with our quote webhook?" starts with
`v1_list_merchant_destinations` or `v1_get_merchant_destination` if its ID is known;
inspect delivery events before proposing changes. Run a new integration check only
when testing is requested, and inspect that check's result using its returned ID.

Use the per-tool request examples and schemas for catalog variants, locations,
markets, rate cards, promotions, quotes, and order responses. A diagnostic question
does not authorize publishing, changing prices, rotating secrets, or deleting data.
Apply only requested changes with supplied facts. Never guess IDs, availability,
stock reservations, coverage, or quote totals. Quote and order responses are
authoritative seller actions and must preserve the documented payment boundaries.
