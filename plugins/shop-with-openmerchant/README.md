# Shop with OpenMerchant

Find products from merchants that sell through OpenMerchant, compare options, get
quotes, and prepare a cart and checkout without leaving the conversation. This version
can't take payment: when a checkout is ready, it shows where the shopper can finish
paying, usually the merchant's own product page.

**Coming soon:** this plugin's server address, `https://openmerchant.dev/mcp/marketplace`,
goes live with its directory listing. Until then, use `shop-with-openmerchant-custom`.

## Use it

Install the plugin, then ask for what you want, for example "Find a ceramic pour-over
coffee dripper under $40." The `find-and-buy` skill walks through the steps: search,
choose, cart, checkout, and then the link where the shopper pays. No OpenMerchant
account is needed. Some merchants ask the shopper to sign in with them before
checkout; the plugin then shows a sign-in link and a confirmation code.

## Data it sends

The plugin connects to OpenMerchant's MCP server at
`https://openmerchant.dev/mcp/marketplace`, operated by OpenMerchant. It sends:

- your search terms and the products and quantities you choose
- the buyer details a checkout needs, such as name, email, and shipping address

OpenMerchant passes the cart and checkout details to the merchant selling the items.
Each request also carries your app's IP address and user agent, which OpenMerchant uses
for rate limiting and which the merchant sees in its request log. The plugin never asks
for or sends card numbers. See the [privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/shop. Support:
https://openmerchant.dev/support.
