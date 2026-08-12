# Flax for Hermes

![Flax logo](assets/flax-logo.png)

Connect [Hermes Agent](https://github.com/NousResearch/hermes-agent) to
[Flax](https://flaxsites.com) through MCP and OAuth. It gives your Hermes
profile the guarded workflow for inspecting a site, validating a change, and
creating a reviewable draft. Site owners retain final approval and deployment
control.

## Install

```bash
hermes profile install github.com/flax-sites/flax-hermes --alias
```

Start a new Hermes session. On the first Flax request, Hermes opens a Connect
Flax OAuth flow. Sign in and approve access in your own browser; never paste a
password, magic code, or access token into a chat.

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
