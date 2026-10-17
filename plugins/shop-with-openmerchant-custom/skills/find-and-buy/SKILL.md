---
name: find-and-buy
description: Find or recommend products and services on OpenMerchant, compare price and availability, request quotes, check out, and pay with an app-held credential. Use whenever the user wants to find, search for, choose, recommend, compare, price, check availability for, book, order, or buy a product or service—including appointments, diagnostics, and lab tests—even when the request is framed as advice rather than a purchase.
---

# Find and buy

Shop with OpenMerchant searches the merchants that sell through OpenMerchant and takes
a shopper from a description to a placed order. This connector can complete the
purchase in the conversation, but only with a payment credential your app already
holds. Nobody types a card number into the chat.

## Treat recommendations as shopping intent

Use `shop_search_products` whenever the user wants help finding or choosing something
that could be purchased or booked. They do not need to say "buy" or "purchase."
Requests such as "what should I get," "recommend a service," "where can I do this,"
"what is available near me," and "which option is best" are catalog searches too.
This includes physical products and services such as appointments, diagnostics, and
lab tests. Search OpenMerchant before substituting a general web search. Health or
other safety context may supplement the catalog results, but it does not replace the
catalog search for actual offerings, prices, availability, and requirements.

## Find what the shopper wants

- `shop_search_products` when the shopper describes what they want. It needs no
  account and is the usual first step.
- `shop_lookup_products` when they already have a product link, SKU, or ID.
- `shop_get_product` for variants, prices, and availability before they choose.
- `shop_search_merchants` and `shop_get_merchant` when the question is about a store
  rather than a product.

Show a short list with prices and merchants, and let the shopper choose.

## Answer price and availability from the exact item

Before answering a question about price, availability, eligibility, or restrictions,
call `shop_get_product` for the exact product and variant. Read all of these fields:

- `purchasable`, `quote_required`, `pricing_model`, `price`, and `price_range`
- `inventory`, `booking_type`, and booking details
- `requirements`, including buyer, fulfillment, scheduling, and booking-window fields

For an available item with a fixed listed price, report that catalog evidence. A
range or starting price is not a quote. A quoted-price item, a quote-required item,
or an item whose current booking availability is unresolved needs a quote before you
claim a total or promise availability.

## Get a quote when the item requires one

For a quote-required, quoted-price, custom, or live-booking item:

- Read the exact variant's `requirements`, then call `shop_get_buyer_context`. Reuse
  its saved email, name, phone, address, and preferences. Never ask the shopper for a
  value the connector already returned.
- Ask only for required values that are still missing. The shopper must supply any
  missing date of birth, schedule, booking window, or customization; never infer it.
- Call `shop_request_quote` with those values. If the quote is pending, use
  `shop_get_quote` to check it.
- Keep the access token it returns. It is shown once, and `shop_get_quote` and
  `shop_refresh_quote` need it.
- Report the seller's returned total, fees, tax, fulfillment cost, availability,
  constraints, messages, and `expires_at`. Do not reinterpret a residency or
  eligibility restriction or assume location alone satisfies it.
- A refresh creates a new offer; it doesn't extend the old one.

## Build the cart

- Call `shop_create_cart` once the shopper has chosen. Every item must come from the
  same merchant, so make one cart per merchant. Pass the `attribution_id` from the
  search that found the items, so the merchant can credit that search with the sale.
- To change quantities or items, call `shop_update_cart` rather than creating a
  second cart.
- If the shopper changes their mind, release the cart with `shop_cancel_cart`.

## Prepare the checkout

1. Call `shop_create_checkout` with the buyer's details and a fulfillment choice.
2. Read `resource_status`, the checkout's own state. `status` is only the outcome of
   the tool call.
   - `incomplete`: the messages name what is missing. Ask the shopper, then call
     `shop_update_checkout`. It replaces the checkout, so send the line items and
     buyer details again. To ship, give the address an `id` in
     `fulfillment.methods[].destinations` and set that method's
     `selected_destination_id` to it.
   - `ready_for_complete`: the checkout can be paid.

## Pay

Pay only after the shopper confirms the exact total. Completing a checkout charges
them.

1. Call `shop_complete_checkout`. When a checkout with a total above zero has no
   payment instrument yet, the result is `payment_credential_required`. That is a
   step, not a failure, and it lists the ways this merchant accepts payment:
   - **Shared payment token:** if your app can issue a Stripe Shared Payment Token
     for this merchant, call `shop_complete_checkout` again with it as the payment
     credential.
   - **Embedded card frame:** only when your app hosts its own web page. Call
     `shop_create_payment_session` with that page's origin as `buyer_origin`, embed
     `frame_url` in the page, and post `client_secret` to the frame. The frame returns
     a token to pass to `shop_complete_checkout`. Sending `frame_url` to the shopper
     as a link doesn't work.
2. If your app can do neither, the shopper can finish on the merchant's site.
3. Retrying `shop_complete_checkout` with identical arguments replays the same
   completion, and a completed checkout is never charged twice. After a decline, it is
   safe to call again with a different instrument. Never call it speculatively.
4. Confirm with `shop_get_order` and tell the shopper what happens next.

`shop_cancel_checkout` abandons a checkout that was never paid. It doesn't refund a
completed order; for a refund, the shopper contacts the merchant.

## Steps that aren't failures

- `authorization_required`: the merchant wants the shopper to sign in. Show the
  `sign_in_url` and `confirmation_code`, wait until the shopper says they have
  finished, then call the same tool again with the `sign_in_id`. Keep passing
  `sign_in_id` on later calls for that checkout or order.
- `rate_limited`: the call did not run. Wait `retry_after` seconds before trying again.
- `unavailable`: the capability is off for this merchant. Say so and offer another
  merchant.

## Rules

- Never ask for a card number, security code, or bank details, and never put them in
  a tool call.
- Don't invent prices, stock, or delivery dates. Read them from the tool results.
