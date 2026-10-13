---
name: find-and-buy
description: Find products and merchants on OpenMerchant, compare options, request quotes, and prepare a cart and checkout, then show the shopper where to finish paying. Use when the user wants to find, compare, price, or order a product or service from an online store.
---

# Find and buy

Shop with OpenMerchant searches the merchants that sell through OpenMerchant and takes
a shopper from a description to a ready checkout. This connector can't take payment.
When a checkout is ready, its result carries a `handoff` that says where the shopper
can finish paying.

## Find what the shopper wants

- `shop_search_products` when the shopper describes what they want. It needs no
  account and is the usual first step.
- `shop_lookup_products` when they already have a product link, SKU, or ID.
- `shop_get_product` for variants, prices, and availability before they choose.
- `shop_search_merchants` and `shop_get_merchant` when the question is about a store
  rather than a product.

Show a short list with prices and merchants, and let the shopper choose.

## Get a quote, only when one is needed

Some items are priced per request, such as a booking or a custom configuration. Call
`shop_request_quote` only when a product says it needs a quote, or a cart refuses it
for that reason.

- Keep the access token it returns. It is shown once, and `shop_get_quote` and
  `shop_refresh_quote` need it.
- Check `expires_at` before you present a price. A refresh creates a new offer; it
  doesn't extend the old one.

## Build the cart

- Call `shop_create_cart` once the shopper has chosen. Every item must come from the
  same merchant, so make one cart per merchant.
- To change quantities or items, call `shop_update_cart` rather than creating a
  second cart.
- If the shopper changes their mind, release the cart with `shop_cancel_cart`.

## Prepare the checkout

1. Confirm the items, quantities, and price with the shopper.
2. Call `shop_create_checkout` with the buyer's details and a fulfillment choice.
3. Read `resource_status`, the checkout's own state. `status` is only the outcome of
   the tool call.
   - `incomplete`: the messages name what is missing. Ask the shopper, then call
     `shop_update_checkout`. It replaces the checkout, so send the line items and
     buyer details again. To ship, give the address an `id` in
     `fulfillment.methods[].destinations` and set that method's
     `selected_destination_id` to it.
   - `ready_for_complete`: only payment is left. Go to the hand-off.

## Hand off

Share the result's `handoff` with the shopper, together with the checkout total. It
lists the merchant's own product pages when they are known, or says to finish on the
merchant's site. Say plainly that the purchase isn't complete until the shopper pays
there, and never say the order is placed.

## Steps that aren't failures

- `authorization_required`: the merchant wants the shopper to sign in. Show the
  `sign_in_url` and `confirmation_code`, wait until the shopper says they have
  finished, then call the same tool again with the `sign_in_id`. Keep passing
  `sign_in_id` on later calls for that checkout.
- `rate_limited`: the call did not run. Wait `retry_after` seconds before trying again.
- `unavailable`: the capability is off for this merchant. Say so and offer another
  merchant.

## Rules

- Never ask for a card number, security code, or bank details, and never put them in
  a tool call.
- Don't invent prices, stock, or delivery dates. Read them from the tool results.
- `shop_cancel_checkout` abandons a checkout the shopper no longer wants.
