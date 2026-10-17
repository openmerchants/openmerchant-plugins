# Shop with OpenMerchant Custom

Find products from merchants that sell through OpenMerchant, compare options, get
quotes, build a cart, and check out without leaving the conversation. Unlike the listed
Shop with OpenMerchant plugin, this version can also complete the purchase, when your
app can supply a payment credential the merchant accepts, such as a Stripe Shared
Payment Token. Otherwise the shopper finishes on the merchant's site.

This plugin is distributed only through this marketplace, not through the Claude or
ChatGPT directories.

## Use it

Install the plugin, then ask for what you want, for example "Recommend a women's
health panel I can get in Las Vegas." Recommendations for products or services count
as shopping searches; the user does not need to say "buy." The `find-and-buy` skill
walks through the steps: search, choose, cart, checkout, and payment. It confirms the
total with the shopper before paying. No OpenMerchant account is needed to search.
Optional OpenMerchant sign-in lets the connector reuse saved contact details and
addresses for quotes and checkout instead of asking again. Some merchants separately
ask the shopper to sign in with them before checkout; the plugin then shows a sign-in
link and a confirmation code.

## Data it sends

The plugin connects to OpenMerchant's MCP server at `https://openmerchant.dev/mcp`,
operated by OpenMerchant. It sends:

- your search terms and the products and quantities you choose
- when you sign in, saved buyer details needed for a quote or checkout, such as name,
  email, phone, preferences, and shipping address
- when you pay, the payment token your app issues for that checkout

OpenMerchant passes required quote, cart, and checkout details to the merchant selling
the items, and uses the payment token only to pay for that checkout. Each request also
carries your app's IP address and user agent, which
OpenMerchant uses for rate limiting and which the merchant sees in its request log.
The plugin never asks for or sends card numbers. See the
[privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/shop. Support:
https://openmerchant.dev/support.
