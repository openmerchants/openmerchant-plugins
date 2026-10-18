# Shop with OpenMerchant Custom (Sandbox)

The test-mode version of Shop with OpenMerchant Custom. It has the same tools and the
same `find-and-buy` skill, but connects to OpenMerchant's sandbox, so searches, quotes,
carts, and checkouts use the sandbox catalog and test mode. Use it to try a client,
a prompt, or a merchant integration without reaching live merchants or making a live
payment.

The plugin declares its server as `shop-with-openmerchant-custom-sandbox`, so you can
install it beside Shop with OpenMerchant Custom and choose which one a conversation
uses. This plugin is distributed only through this marketplace, not through the
Claude or ChatGPT directories.

## Use it

Install the plugin, then ask for what you want, for example "Recommend a women's
health panel I can get in Las Vegas." The `find-and-buy` skill walks through search,
choice, cart, checkout, and payment, and confirms the total with the shopper before
paying. Completing a purchase needs a test payment credential the merchant accepts,
such as a test-mode Stripe Shared Payment Token; otherwise the shopper finishes on
the merchant's site. No OpenMerchant account is needed to search. Optional
OpenMerchant sign-in lets the connector reuse saved contact details and addresses.

## Data it sends

The plugin connects to OpenMerchant's MCP server at
`https://sandbox.openmerchant.dev/mcp`, operated by OpenMerchant. It sends:

- your search terms and the products and quantities you choose
- when you sign in, saved buyer details needed for a quote or checkout, such as name,
  email, phone, preferences, and shipping address
- when you pay, the test payment token your app issues for that checkout

OpenMerchant passes required quote, cart, and checkout details to the sandbox
merchant selling the items, and uses the payment token only to pay for that checkout.
Each request also carries your app's IP address and user agent, which OpenMerchant
uses for rate limiting and which the merchant sees in its request log. The plugin
never asks for or sends card numbers. See the
[privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/shop. Support:
https://openmerchant.dev/support.
