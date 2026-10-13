# OpenMerchant plugins

Plugins and connector configuration for using OpenMerchant from AI assistants and coding
agents: Claude (connectors and plugins), ChatGPT and Codex (plugins), and any client that
speaks the Model Context Protocol (MCP).

## Plugins

| Plugin | For | What it does | MCP server | Sign-in |
| --- | --- | --- | --- | --- |
| `shop-with-openmerchant` | Buyers | Find products, get quotes, and prepare a checkout; the shopper pays on the merchant's site | `https://openmerchant.dev/mcp/marketplace` (coming soon) | None |
| `shop-with-openmerchant-custom` | Buyers | All of the above, plus paying in the conversation with a payment credential your app holds | `https://openmerchant.dev/mcp` | None |
| `openmerchant` | Sellers and businesses | Ask what got paid and what is waiting across your vendor invoices; never starts a payment | `https://api.openmerchant.dev/mcp/marketplace/v1` | OpenMerchant account |
| `openmerchant-custom` | Sellers and businesses | Every OpenMerchant API tool, including submitting vendor invoices | `https://api.openmerchant.dev/mcp/v1` | OpenMerchant account, or a secret API key |

`shop-with-openmerchant` and `openmerchant` are the versions submitted to the Claude and
ChatGPT directories. The two `-custom` plugins are available only here. Each plugin's
README says what it sends where. The `shop-with-openmerchant` server address goes live
with its directory listing; until then, use `shop-with-openmerchant-custom`.

## Install

**Claude Code**

```
/plugin marketplace add openmerchants/openmerchant-plugins
/plugin install shop-with-openmerchant-custom@openmerchant
```

**Claude** (claude.ai and the desktop app): Customize → Plugins → Add → Add marketplace,
then enter `https://github.com/openmerchants/openmerchant-plugins`.

**Codex**

```
codex plugin marketplace add openmerchants/openmerchant-plugins
```

Then open `/plugins` in Codex to install a plugin.

**Any other MCP client:** add the server address from the table. The guides at
https://openmerchant.dev/connect/shop and https://openmerchant.dev/connect/account cover
each app.

## What's here

- `.claude-plugin/marketplace.json`: the Claude marketplace, named `openmerchant`
- `.agents/plugins/marketplace.json`: the Codex marketplace
- `plugins/<name>/`: one folder per plugin, with its Claude manifest
  (`.claude-plugin/plugin.json` and `.mcp.json`), its portable manifest for ChatGPT and
  Codex (`plugin.json` and `mcp.json`), its skills, README, license, and icon
- `registry/`: entries for the official MCP registry

This repository is published from OpenMerchant's main codebase, and each update replaces
its contents, so changes made here directly don't last. Send suggestions to
support@openmerchant.dev.

## Security

Report security issues to security@openmerchant.dev.

## License

[MIT](LICENSE)
