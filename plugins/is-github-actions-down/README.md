# Is GitHub Actions down?

Check GitHub Actions status without leaving your conversation. This plugin
connects to the public status service at https://isgithubactionsdown.com and
includes a skill that distinguishes official incidents, possible disruptions,
and stale information. No account or credentials are required.

## Install in Claude Code

```
/plugin marketplace add openmerchants/openmerchant-plugins
/plugin install is-github-actions-down@openmerchant
```

For Claude on the web or desktop, add the OpenMerchant marketplace, or add
`https://isgithubactionsdown.com/mcp` as a custom connector. Other MCP clients
can connect to the same endpoint using Streamable HTTP.

## Ask

- “Is GitHub Actions down right now?”
- “My workflow is stuck in the queue. Is GitHub reporting an incident?”
- “Has GitHub resolved the Actions incident?”

The `get_github_actions_status` tool has no arguments and returns the same
assessment as the site and JSON API. “Yes” means an unresolved official Actions
incident. “Probably” can come from component disruption or unusual visitor
interest, which is not confirmation. “No” does not rule out workflow-specific
problems. Stale results always retain their original check time.

## Data and permissions

The plugin only reads public status. It sends requests to
`https://isgithubactionsdown.com/mcp`, without repository content, account
information, payment data, or credentials. Ordinary connection metadata is
processed by the hosting provider. API and MCP polling never contribute to
the site's visitor signal. The plugin does not change or rerun workflows.

See [agent setup](https://isgithubactionsdown.com/agents) and
[methodology and privacy](https://isgithubactionsdown.com/how-it-works#privacy).
This independently maintained service is not affiliated with GitHub.
