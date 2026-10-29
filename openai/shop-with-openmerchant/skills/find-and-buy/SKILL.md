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

## Addresses come from the shopper's profile

This plugin ships only to addresses saved on the shopper's OpenMerchant profile. It never
takes a typed-in street, city, or postal code, so shipping and delivery need the shopper
signed in with a saved address.

- `shop_get_buyer_context` lists the saved addresses, their IDs, and the shipping,
  billing, and pickup defaults. Suggest the matching default, or ask which saved address
  to use.
- When none is suitable, share the `address_book_url` it returns. The shopper adds or
  edits the address on their OpenMerchant profile; call `shop_get_buyer_context` again
  to get its ID.
- Search results and quotes without an address use the approximate location ChatGPT
  shares, so prices, shipping, and tax are estimates until a saved address is chosen.

## Get a quote when the item requires one

For a quote-required or quoted-price product:

- Read the exact variant's `requirements`, then call `shop_get_buyer_context` to reuse
  saved contact details and addresses. Ask only for required values that are still
  missing.
- For a shipping or delivery price, pass the chosen saved address's ID as
  `fulfillment.address_id`. Before buying, request the quote again with the saved
  address if the first one was an estimate.
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
   pickup choice. For shipping or delivery, set
   `fulfillment.methods[].selected_destination_id` to a saved address's ID, or leave it
   unset to use the shopper's default. Merchant pickup uses the store's location ID.
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
2. Call `shop_complete_checkout` with the `checkout_id` and the chosen card's `id` as
   `payment_method_id`. ChatGPT asks the shopper to approve that destructive tool call
   in the conversation; ordinary checkout approval does not require a browser page.
3. If that call returns `payment_credential_required` with a `confirmation_url`, the
   exact selected card needs authentication. Share that URL only for card
   authentication, wait for the shopper to finish, then retry
   `shop_complete_checkout` with the same `checkout_id` and `payment_method_id`. Do
   not select a different card or change between test and live mode.
4. Confirm with `shop_get_order` and tell the shopper what happens next. Until the order
   is confirmed, say that the purchase isn't complete.

`shop_cancel_checkout` abandons a checkout that was never paid. It doesn't refund a
completed order; for a refund, the shopper contacts the merchant.

## Steps that aren't failures

- `authorization_required`: the merchant wants the shopper to sign in. Show the
  `sign_in_url` and `confirmation_code`, wait until the shopper says they have finished,
  then call the same tool again with the `sign_in_id`.
- `payment_credential_required` with `confirmation_url`: the exact selected card needs
  authentication at that secure URL, then the same call is retried with the same card.
- `rate_limited`: the call did not run. Wait `retry_after` seconds before trying again.
- `unavailable`: the capability is off for this merchant. Say so.

## Rules

- Never ask for a card number, security code, or bank details, and never put them in a
  tool call.
- Don't invent prices, stock, or delivery dates. Read them from the tool results.
