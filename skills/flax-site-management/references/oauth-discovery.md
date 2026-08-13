# Flax OAuth Discovery Chain

All Flax MCP endpoints are OAuth 2.1 PKCE-protected. Discover endpoints in
exactly this order — never hardcode the Supabase project URL (it can change).

## Chain (both variants)

1. Trigger or read the 401 challenge from the MCP endpoint you want to use.

   - Account-level:  `GET https://agents.flaxsites.com/mcp`
   - Site-scoped:    `GET https://agents.flaxsites.com/site-mcp/{siteId}/mcp`

   The response is `401` with:

   ```
   www-authenticate: Bearer resource_metadata="{protectedResourceMetadataUrl}"
   ```

2. `GET {protectedResourceMetadataUrl}`

   ```json
   {
     "resource": "https://agents.flaxsites.com",
     "authorization_servers": ["https://<project>.supabase.co/auth/v1"],
     "scopes_supported": ["openid", "email", "profile"]
   }
   ```

   Take `authorization_servers[0]` as the auth server.

3. `GET {authServer}/.well-known/oauth-authorization-server`

   ```json
   {
     "authorization_endpoint": ".../oauth/authorize",
     "token_endpoint":         ".../oauth/token",
     "registration_endpoint":  ".../oauth/clients/register",
     "code_challenge_methods_supported": ["S256", "plain"],
     "token_endpoint_auth_methods_supported": ["client_secret_basic", "client_secret_post", "none"]
   }
   ```

## Which variant?

| Goal | Variant | Discovery start |
|------|---------|-----------------|
| Read/write one named site's model | **site-scoped** | the site's `/.well-known/mcp.json` → `mcp.url` + `mcp.protectedResourceMetadataUrl` |
| List sites, upload images, account admin, no site named | **account-level** | `GET https://agents.flaxsites.com/mcp` → 401 + header |

## Rules

- ALWAYS derive endpoints from the chain. The Supabase subdomain is an
  implementation detail and may change.
- Use `code_challenge_method: S256` (never `plain`).
- Use `token_endpoint_auth_method: "none"` (public client) at registration.
- Tokens are scoped to the resource they were minted for — re-authenticate
  when switching between account-level and site-scoped endpoints.

## Next step

Once endpoints are known, run the PKCE flow in `references/oauth-pkce-flow.md`.
