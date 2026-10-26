---
name: find-and-buy
description: Find and buy physical products from merchants that sell through OpenMerchant, including recommendations, gifts, alternatives, cheaper options, price and availability, quotes, and checkout. Physical goods only, not services, appointments, lab tests, travel, subscriptions, or digital products. Pure explanations, writing tasks, and requests to avoid tools need no catalog search.
---

# Find and buy

Shop with OpenMerchant searches the merchants that sell through OpenMerchant and takes
a shopper from a description of a physical product to a placed order. It completes the
purchase in the conversation after the shopper confirms the total, with a card saved in
their OpenMerchant wallet. Nobody types a card number into the chat.

## Physical goods only

This plugin covers physical goods: products that ship, are delivered, or are picked up.
Services, appointments, diagnostics and lab tests, travel and other bookings, software,
subscriptions, and digital products are out of scope. When a shopper asks for one, say
that this plugin covers physical goods only.

Search, lookup, and product results leave out items the catalog marks as services,
bookings, or digital items, and the result says how many it left out. A quote, cart,
checkout, or payment for one is refused with `not_physical_good`; nothing is reserved,
and the shopper can choose a physical product instead or leave that item out.

Results from other merchants' public catalogs don't always say what they are. Mention
one only when it is plainly a physical product, and point the shopper to its page on the
merchant's site.

## Find what the shopper wants

| User request | First action |
| --- | --- |
| "Suggest a quiet blender for a studio apartment" | `shop_search_products` with the item and constraints. |
| "What tea does this store sell?" | `shop_search_products` scoped to the merchant. |
| "Anything smaller than those options?" | Search again with the previous item and the new size constraint. |
| "Can you check this product link?" | `shop_lookup_products` with the exact supplied identifier. |
| "Explain why stainless steel rusts" | Answer the explanation; there is no shopping request. |

- `shop_search_products` takes a description and needs no account. Search again for each
  follow-up that narrows the request, such as a cheaper option or a different size.
- `shop_lookup_products` takes a product link, SKU, or ID the shopper already has.
- `shop_get_product` returns variants, prices, and availability before the shopper
  chooses.

Search, lookup, and product detail return product cards directly in supported apps.
Summarize the results briefly; there is no separate card-display step. Use
`shop_show_products` only to curate or reorder a selection: pass each product's `id`,
its exact `merchant` selector, and the `attribution_id` from the result's `analytics`.
If the catalog has no match or is unavailable, say so. Do not invent a match, a product
ID, or a price.

## Answer price and availability from the exact item

Before answering a question about price, availability, or restrictions, call
`shop_get_product` for the exact product and variant. Read `purchasable`,
`quote_required`, `pricing_model`, `price`, `price_range`, `inventory`, and
`requirements`. A range or starting price is not a quote.

When a price carries a seller discount, report the price as given: a fiat price's
`original_amount` is its pre-discount price, and a quote offer's
`dev.openmerchant.promotion` gives the discount and its `expires_at`.

## Get a quote when the item requires one

For a quote-required or quoted-price product:

- Read the exact variant's `requirements`, then call `shop_get_buyer_context` to reuse
  saved contact details and addresses. Ask only for required values that are still
  missing.
- Call `shop_request_quote`. If the quote is pending, check it with `shop_get_quote`.
- Keep the access token it returns. It is shown once, and `shop_get_quote` and
  `shop_refresh_quote` need it.
- Report the returned total, fees, tax, shipping, availability, constraints, and
  `expires_at`. A refresh creates a new offer; it doesn't extend the old one.

## Build the cart

- Call `shop_create_cart` once the shopper has chosen. Every item must come from the
  same merchant, so make one cart per merchant. Pass the `attribution_id` from the search
  that found the items.
- To change quantities or items, call `shop_update_cart` rather than creating a second
  cart. If the shopper changes their mind, release it with `shop_cancel_cart`.

## Prepare the checkout

1. Call `shop_create_checkout` with the buyer's details and a shipping, delivery, or
   pickup choice.
2. Read `resource_status`, the checkout's own state. `status` is only the outcome of the
   tool call.
   - `incomplete`: the messages name what is missing. Ask the shopper, then call
     `shop_update_checkout`. It replaces the checkout, so send the line items and buyer
     details again.
   - `ready_for_complete`: the checkout can be paid.

## Pay

Pay only after the shopper confirms the exact total. Paying places the order and charges
them.

1. To see the cards saved in the shopper's OpenMerchant wallet, call
   `shop_list_saved_payment_methods`. It returns only the brand, last four digits, and
   expiry, and asks the shopper to sign in if needed. To add or change a card, call
   `shop_open_wallet` and share its link.
2. Call `shop_pay_with_saved_card` with the `checkout_id` and, as `payment_method_id`,
   the chosen card's `id`. When the result is `confirmation_required`, share its
   `pay_url`: the shopper confirms the purchase or authenticates the card on that
   secure OpenMerchant page. Then call the same tool again with the same arguments.
3. A checkout result can also carry a `pay_url` for OpenMerchant's secure checkout
   page, where the shopper can review the order and pay in the browser.
4. Confirm with `shop_get_order` and tell the shopper what happens next. Until the order
   is confirmed, say that the purchase isn't complete.

`shop_cancel_checkout` abandons a checkout that was never paid. It doesn't refund a
completed order; for a refund, the shopper contacts the merchant.

## Steps that aren't failures

- `authorization_required`: the merchant wants the shopper to sign in. Show the
  `sign_in_url` and `confirmation_code`, wait until the shopper says they have finished,
  then call the same tool again with the `sign_in_id`.
- `confirmation_required`: the shopper approves the payment or authenticates the card
  at the `pay_url`, then the same call is retried.
- `rate_limited`: the call did not run. Wait `retry_after` seconds before trying again.
- `unavailable`: the capability is off for this merchant. Say so.

## Rules

- Never ask for a card number, security code, or bank details, and never put them in a
  tool call.
- Don't invent prices, stock, or delivery dates. Read them from the tool results.
