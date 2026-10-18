---
name: find-business-purchases
description: Find and compare products or services for a business purchase, check current catalog prices and stock, or inspect existing purchase requests. Use for equipment recommendations, sourcing services, cheaper alternatives, and "what did we order", even without naming OpenMerchant. Supplier invoice payments use check-vendor-payments; managing what your own store sells uses manage-merchant-store.
---

# Find business purchases

Use the enabled account connector's catalog reads when the user wants buying
options. "What webcam should we get for the meeting room?" starts with
`v1_search_catalog`; the user need not explicitly request a connector or tool.
Include the known item, budget, location, and other constraints. Do not make a
buyer profile, payment method, or a complete budget prerequisites for discovery.
Search again when the user changes the item or asks for cheaper alternatives.

A supplied product URL or SKU uses `v1_lookup_catalog_products`; inspect a known
item with `v1_get_catalog_product`. Questions about a provider's APIs or payment
protocols use the merchant registry tools, not catalog-management writes.
Use tool results for prices, stock, requirements, and merchant identity. If the
catalog has no suitable match, explain its coverage and use other sources when
appropriate. Pure explanations or a request to avoid tools need no catalog call.

Once the user chooses concrete items and a merchant, follow the purchase tools'
requirements to prepare a purchase intent. Discovery does not authorize approval,
payment, a credential, or a retry. Preserve the ordinary approval and verification
flow; a live retry needs explicit confirmation. "What did we order?" starts with
`v1_list_purchase_intents`; a known purchase ID uses `v1_get_purchase_intent`.
Read its state before changing or retrying it.
