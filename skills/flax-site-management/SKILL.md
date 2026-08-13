---
name: flax-site-management
description: Connect to Flax through MCP and safely inspect sites, validate changes, prepare previewable drafts, and require explicit owner approval before publishing.
---

# Flax site management

## Connection rules

When a user names a website, start with that exact public origin:

```text
https://<site-origin>/.well-known/mcp.json
```

Use the MCP URL it advertises. It is scoped to that site. Do not substitute
the account endpoint, list other sites, or guess a site ID.

If that discovery document is missing or unavailable, explain that the site
has not advertised MCP and ask whether the user wants account-level Flax
administration instead. The account-level endpoint is:

```text
https://agents.flaxsites.com/mcp
```

Authenticate through the MCP client's OAuth flow. Show or open its Connect
Flax authorization link so the owner can sign in and approve access in their
own browser. Never request a password, magic code, access token, CDN
credential, or deployment credential in chat.

## Change workflow

1. Read the current site model before proposing a change.
2. Use the public schemas and the model's real fields; do not invent paths.
3. Validate every update plan before proposing it.
4. Create a draft only after validation passes.
5. State clearly that the draft is not published. The owner must preview and
   explicitly approve it before deployment.

## Safety boundaries

- Flax agents can propose content and model changes, but do not receive
  publishing, Bunny/CDN, payment, or infrastructure credentials.
- Use the least site scope available. A named site's scoped endpoint is
  preferred over account-level access.
- Treat a changed live model hash or a failed validation as a reason to reread
  the model and revise the draft rather than overwriting current content.
- For images, use the Flax MCP image-upload flow. Do not inline image bytes in
  an MCP tool request.

## Mechanics (non-interactive contexts)

The MCP client normally handles OAuth and tool calls for you. When it cannot
(non-interactive environments: `execute_code`, cron, background agents,
subagents), use these references:

- `references/oauth-discovery.md` — 401 → protected-resource → auth-server chain; account-level vs site-scoped.
- `references/oauth-pkce-flow.md` — complete runnable PKCE script (register client → code → exchange).
- `references/json-rpc-protocol.md` — initialize / tools/list / tools/call payloads, response envelope, error codes.
- `references/token-refresh.md` — 3600s expiry; refresh without re-running PKCE.

Order matters: discovery → PKCE → JSON-RPC. Every MCP call uses
`Authorization: Bearer <access_token>` over HTTP POST.

## References

- https://flaxsites.com/docs/agents
- https://flaxsites.com/.well-known/flax-agent.json
- https://flaxsites.com/schemas/flax/v1/site-data-model.schema.json
- https://flaxsites.com/.well-known/flax-agent-recipes.v1.json

