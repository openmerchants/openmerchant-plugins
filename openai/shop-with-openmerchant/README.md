# Shop with OpenMerchant for ChatGPT

The ChatGPT version of Shop with OpenMerchant, packaged for OpenAI's plugin portal. It
finds physical products from merchants that sell through OpenMerchant, compares price and
availability, requests quotes, and checks out once the shopper confirms the total: it pays
with a card saved in the shopper's OpenMerchant wallet, and OpenMerchant's secure page
takes any approval or card authentication. It never asks for card details.

OpenAI's plugin guidelines allow commerce for physical goods only, so this package and its
server are limited to them. It connects to its own server,
`https://openmerchant.dev/mcp/openai`, which leaves services, appointments, lab tests,
travel, subscriptions, and digital products out of its catalog results and refuses to
quote, cart, or check them out. The Claude-listed `shop-with-openmerchant` plugin connects
to `https://openmerchant.dev/mcp/marketplace` instead, whose catalog includes them.

This package is uploaded to the portal as a ZIP. It isn't listed in this repository's
Claude or Codex marketplaces, which carry `plugins/shop-with-openmerchant` instead.

## Build the ZIP

From this folder, with a new `version` in `plugin.json` for every upload after the first:

```
zip -r -X ~/shop-with-openmerchant.zip plugin.json mcp.json skills assets README.md LICENSE
```

The portal reads `plugin.json`, `mcp.json`, `skills/`, and `assets/` from the ZIP root.
Reviewer sign-in details, test cases, the demo recording, and release notes go in the
portal's forms, never in the ZIP.

## Data it sends

The plugin connects to OpenMerchant's MCP server at
`https://openmerchant.dev/mcp/openai`, operated by OpenMerchant. It sends your search
terms and the products and quantities you choose and, when you sign in, the saved buyer
details a quote or checkout needs, such as name, email, and phone, the ID of the saved
address to ship to, and which saved card to pay with. Its tools never take a street,
city, or postal code: addresses are added on your OpenMerchant profile. ChatGPT also
shares an approximate location with each request; OpenMerchant uses its country, region,
and a ZIP code estimated from its coordinates to show local prices and estimate shipping
and tax. OpenMerchant passes required quote, cart, and checkout details to the merchant
selling the items, and uses your card only to pay for that checkout. The plugin never
asks for or sends card numbers. See the
[Shop with OpenMerchant privacy notice](https://openmerchant.dev/shop-privacy), which
supplements the [privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/shop. Support:
https://openmerchant.dev/support.
