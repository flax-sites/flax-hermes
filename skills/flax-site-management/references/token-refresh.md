# Flax OAuth Token Refresh

Flax MCP access tokens expire after **3600 seconds** (1 hour). The token
response includes a `refresh_token` that can be exchanged for a new access
token without re-running the PKCE flow.

## Refresh flow (Supabase OAuth)

```python
import json, subprocess

with open("/tmp/flax_tokens.json") as f:
    tokens = json.load(f)

# client_id from the PKCE flow (references/oauth-pkce-flow.md)
with open("/tmp/flax_oauth_params.json") as f:
    params = json.load(f)

refresh_payload = {
    "grant_type": "refresh_token",
    "refresh_token": tokens["refresh_token"],
    "client_id": params["client_id"],
}

token_endpoint = "https://<project>.supabase.co/auth/v1/oauth/token"  # from discovery
result = subprocess.run([
    "curl", "-sL", "-m", "10", "-X", "POST",
    "-H", "Content-Type: application/json",
    "-d", json.dumps(refresh_payload),
    token_endpoint
], capture_output=True, text=True)

new_tokens = json.loads(result.stdout)
with open("/tmp/flax_tokens.json", "w") as f:
    json.dump(new_tokens, f, indent=2)

print(f"Refreshed: {new_tokens['access_token'][:20]}... expires_in={new_tokens['expires_in']}")
```

## Response shape

```json
{
  "access_token": "eyJhbG...",
  "expires_in": 3600,
  "refresh_token": "abc123...",   // NEW token — replaces the old one
  "token_type": "bearer"
}
```

## Detecting expiry

A `401` with `{"error":"Authentication required"}` from the MCP endpoint means
the token expired. Refresh and retry the call — do NOT re-run the full PKCE flow.

## Hermes long-lived sessions

For persistent Hermes setups, run `hermes mcp login flax` from an interactive
terminal once — Hermes then manages token lifecycle automatically. The manual
refresh above is for non-interactive sessions (cron, execute_code, background).
