# Shop with OpenMerchant

Find products from merchants that sell through OpenMerchant, compare options, get
quotes, build a cart, and check out without leaving the conversation. Once the shopper
confirms the total, the client asks them to approve the checkout tool call in the
conversation. It then pays with the selected card from their OpenMerchant wallet or
with a payment credential your app can supply, such as a Stripe Shared Payment Token.
A secure OpenMerchant page appears only if the selected saved card needs
authentication. Otherwise the shopper finishes on the merchant's site.

**Coming soon:** this plugin's server address, `https://openmerchant.dev/mcp/marketplace`,
goes live with its directory listing. Until then, use `shop-with-openmerchant-custom`.

## Use it

Install the plugin, then ask for what you want, for example "Recommend a women's
health panel I can get in Las Vegas." Recommendations for products or services count
as shopping searches; the user does not need to say "buy." The `find-and-buy` skill
walks through the steps: search, choose, cart, checkout, and payment. It confirms the
total with the shopper before asking them to approve the destructive checkout call in
the conversation. No OpenMerchant account is needed to search. Optional OpenMerchant
sign-in lets the connector reuse saved contact details, addresses, and cards instead
of asking again. Some merchants separately ask the shopper to sign in with them before
checkout; the plugin then shows a sign-in link and a confirmation code.

## Data it sends

The plugin connects to OpenMerchant's MCP server at
`https://openmerchant.dev/mcp/marketplace`, operated by OpenMerchant. It sends:

- your search terms and the products and quantities you choose
- when you sign in, saved buyer details needed for a quote or checkout, such as name,
  email, phone, preferences, and shipping address
- when you pay, which saved card to use, or the payment token your app issues for that
  checkout

OpenMerchant passes required quote, cart, and checkout details to the merchant selling
the items, and uses the payment details only to pay for that checkout. Each request
also carries your app's IP address and user agent, which OpenMerchant uses for rate
limiting and which the merchant sees in its request log. The plugin never asks for or
sends card numbers. See the [privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/shop. Support:
https://openmerchant.dev/support.
