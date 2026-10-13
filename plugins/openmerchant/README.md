# OpenMerchant

Ask about your OpenMerchant vendor invoices and payments from the conversation: what
got paid, what is scheduled, what is waiting for a person, and which vendors still need
to send payout details. It can also ask a vendor for missing payout details, which
emails them a secure link. It never submits an invoice, and it never approves, pays,
cancels, or retries a payment; those stay in the OpenMerchant dashboard.

## Use it

Install the plugin and connect it. Your app opens OpenMerchant in the browser, where
you sign in and choose the account and test or live mode. Only an account owner with a
verified email can connect, and the account needs Pay vendors turned on in the mode you
choose. Then ask, for example, "Which invoices are waiting for me?" The
`check-vendor-payments` skill picks the right reads and links you to the dashboard
when a person needs to act.

## Data it sends and receives

The plugin connects to OpenMerchant's MCP server at
`https://api.openmerchant.dev/mcp/marketplace/v1`, operated by OpenMerchant. Your app
holds an access token for this connector only, and sends it with each request, along
with the details of what you asked, such as filters and invoice IDs.

OpenMerchant returns your vendor invoices and payments to your app: vendor names,
invoice numbers, amounts, dates, statuses, document metadata, and the last four digits
of payout accounts, never full account numbers. Your app's AI provider processes that
data under its own terms. When you ask OpenMerchant to request payout details,
OpenMerchant emails the vendor. Revoke access at any time under Connections → Connected
apps in the dashboard. See the [privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/account. Support:
https://openmerchant.dev/support.
