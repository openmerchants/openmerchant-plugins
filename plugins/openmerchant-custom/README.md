# OpenMerchant Custom

Every OpenMerchant API tool for your account, from the conversation or a coding agent:
find products and services for business purchases, check catalog prices and stock,
submit and track vendor invoices, read payment status, manage purchase intents and
approvals, payment methods, budgets and spending policies, and your merchant catalog.
Submitted invoices from known vendors within your limits can be paid automatically
under your account's rules; everything else waits for a person in the dashboard.

This plugin is distributed only through this marketplace, not through the Claude or
ChatGPT directories. For read-only questions about vendor payments, use the listed
`openmerchant` plugin instead.

## Use it

Install the plugin and connect it. Your app opens OpenMerchant in the browser, where
you sign in and choose the account and test or live mode. Only an account owner with a
verified email can connect. The `submit-vendor-invoice` and `check-vendor-payments`
skills cover the vendor payment workflow. `find-business-purchases` handles buying
options and purchase status; `manage-merchant-store` handles your own listings,
publication readiness, and integrations. Ask naturally, such as "find monitors for
our new hires" or "what is blocking my store from publishing"; naming a connector
or tool is unnecessary.

### Use a secret API key instead

Coding agents can use a secret API key (`pr_sk_test_…` or `pr_sk_live_…`) from
Developers → API keys in the dashboard; the key decides test or live mode.

- **Claude Code:** add the same server address yourself with your key. Your entry
  replaces the plugin's, because both point at the same address, and the plugin's
  skills still apply:
  `claude mcp add --transport http openmerchant-custom https://api.openmerchant.dev/mcp/v1 --header "Authorization: Bearer <your secret key>"`
- **Codex:** add your own server to `~/.codex/config.toml` with
  `url = "https://api.openmerchant.dev/mcp/v1"` and `bearer_token_env_var` set to the
  name of an environment variable that holds your key. Then turn off the plugin's
  bundled server in your Codex config, so the tools aren't listed twice.

The plugin itself carries no key field: Claude Code turns off OAuth sign-in for a
server whose `Authorization` header is configured, even when the key is left empty.

## Data it sends and receives

The plugin connects to OpenMerchant's MCP server at `https://api.openmerchant.dev/mcp/v1`,
operated by OpenMerchant. Your app sends its access token or your API key with each
request, along with the tool arguments, such as invoice facts, uploaded document
references, and purchase or policy changes.

OpenMerchant returns your account data to your app, such as invoices, payments,
purchases, and policies, and your app's AI provider processes it under its own terms.
Bank details sent in a tool call are stored encrypted and never returned in full.
Revoke an app under Connections → Connected apps in the dashboard, or rotate the API
key. See the [privacy policy](https://openmerchant.dev/privacy).

## Help

Setup guide: https://openmerchant.dev/connect/account. Support:
https://openmerchant.dev/support.
