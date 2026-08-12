# Flax for Hermes: agentic sites, with owners in control

![Flax logo](assets/flax-logo.png)

Connect [Hermes Agent](https://github.com/NousResearch/hermes-agent) to
[Flax](https://flaxsites.com) through MCP and OAuth.

Flax makes websites agentic without handing agents the keys. An agent can
understand a site's published model, validate a proposed update, and prepare a
previewable draft. The site owner remains in control: every change must be
reviewed and explicitly approved before deployment.

That makes Flax a practical path from "ask an agent to update my website" to
a safe, auditable site-management workflow for businesses, agencies, and
product teams.

## Install

```bash
hermes profile install github.com/flax-sites/flax-hermes --alias
```

Start a new Hermes session. On the first Flax request, Hermes opens a Connect
Flax OAuth flow. Sign in and approve access in your own browser; never paste a
password, magic code, or access token into a chat.

## Add Flax to an existing Hermes profile

[`mcp.json`](mcp.json) is the standard MCP document for the integration. In
Hermes Desktop, paste it into the MCP editor and save; Hermes stores the same
entry in `config.yaml` as `mcp_servers.flax`. In the CLI, the equivalent is:

```bash
hermes mcp add flax --url https://agents.flaxsites.com/mcp --auth oauth
hermes mcp login flax
```

`flax` is the MCP server name. It is not a plugin name, and `mcp.json` does
not make GitHub repositories searchable by itself. The pending Hermes catalog
entry is what will make Flax discoverable through `hermes mcp catalog` after
Nous reviews and merges it.

## Use

For a particular site, name its domain in your request. The included skill
first checks that exact site's `/.well-known/mcp.json` and uses the
site-scoped endpoint it advertises.

For account-level work without a named site, Hermes connects to
`https://agents.flaxsites.com/mcp` and Flax asks you to choose the site to
authorize.

Example:

```text
Connect to https://example.com and prepare a draft that updates our contact number.
```

Every change is validated and proposed as a draft. A site owner must preview
and explicitly approve it before it is deployed.

## Why agentic sites need guardrails

An agentic site should be useful without becoming an uncontrolled publishing
bot. This integration is designed around that boundary:

- **Site-scoped access:** when a site is named, the agent discovers and uses
  only that site's MCP endpoint.
- **OAuth consent:** the owner authorizes access in their own browser; no
  password, magic code, or token is ever pasted into chat.
- **Schema-validated changes:** updates are checked against the site's real
  data model before a draft can be created.
- **Preview before publish:** agents prepare drafts; owners approve deployment.
- **No infrastructure credentials:** MCP clients do not receive CDN,
  publishing, payment, or deployment credentials.

## For AI-agent builders

The hosted Flax MCP endpoint is:

```text
https://agents.flaxsites.com/mcp
```

For a named site, begin at its exact `/.well-known/mcp.json` document and use
the site-scoped URL it advertises. Read the
[Flax agent documentation](https://flaxsites.com/docs/agents) for schemas,
recipes, and the full OAuth flow.

## Updating

To receive changes to this distribution:

```bash
hermes profile update flax-hermes
```

The distribution contains no user credentials. Hermes stores OAuth tokens
locally for the installing user.

## Flax MCP tools

The Flax MCP server is hosted by Flax, so tool additions are discovered by
existing installations when Hermes reconnects or starts a new session. If a
profile has an explicit tool allowlist, run `hermes mcp configure flax` to
enable newly added tools.

## Support

See [Flax agent documentation](https://flaxsites.com/docs/agents) or contact
[support@flaxsites.com](mailto:support@flaxsites.com).

## License

[MIT](LICENSE)
