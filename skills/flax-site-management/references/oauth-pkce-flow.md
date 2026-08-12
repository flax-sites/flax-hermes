# Flax OAuth PKCE Flow — Scripted Pattern

Use when the MCP client's own OAuth flow cannot run (non-interactive
environments: `execute_code`, cron, background agents, subagents). This runs
the full Supabase PKCE flow with a local callback server on `127.0.0.1:8888`.

Prerequisite: endpoints discovered per `references/oauth-discovery.md`.

## Complete script

```python
import subprocess, json, hashlib, base64, secrets, http.server, urllib.parse, threading

# ---- 1. Discovery (see references/oauth-discovery.md) ----
# For site-scoped: start from the site's /.well-known/mcp.json
site_mcp_json = "https://<site>/.well-known/mcp.json"   # or account-level 401 header
result = subprocess.run(["curl", "-sL", "-m", "10", site_mcp_json], capture_output=True, text=True)
mcp_info = json.loads(result.stdout)
protected_metadata_url = mcp_info["mcp"]["protectedResourceMetadataUrl"]
mcp_url = mcp_info["mcp"]["url"]

result = subprocess.run(["curl", "-sL", "-m", "10", protected_metadata_url], capture_output=True, text=True)
prot_meta = json.loads(result.stdout)
auth_server = prot_meta["authorization_servers"][0]

result = subprocess.run(["curl", "-sL", "-m", "10", f"{auth_server}/.well-known/oauth-authorization-server"], capture_output=True, text=True)
oauth_meta = json.loads(result.stdout)
auth_endpoint = oauth_meta["authorization_endpoint"]
token_endpoint = oauth_meta["token_endpoint"]
reg_endpoint = oauth_meta.get("registration_endpoint")

# ---- 2. Register dynamic client (public client, no secret) ----
reg_payload = {
    "client_name": "Hermes Agent - Flax Sites",
    "redirect_uris": ["http://localhost:8888/callback"],
    "grant_types": ["authorization_code", "refresh_token"],
    "response_types": ["code"],
    "token_endpoint_auth_method": "none"
}
result = subprocess.run(["curl", "-sL", "-m", "10", "-X", "POST",
    "-H", "Content-Type: application/json", "-d", json.dumps(reg_payload), reg_endpoint],
    capture_output=True, text=True)
client = json.loads(result.stdout)
client_id = client["client_id"]

# ---- 3. Generate PKCE ----
code_verifier = base64.urlsafe_b64encode(secrets.token_bytes(32)).rstrip(b'=').decode()
code_challenge = base64.urlsafe_b64encode(
    hashlib.sha256(code_verifier.encode()).digest()
).rstrip(b'=').decode()
state = secrets.token_urlsafe(16)

params = {
    "client_id": client_id,
    "code_verifier": code_verifier,
    "state": state,
    "redirect_uri": "http://localhost:8888/callback",
    "token_endpoint": token_endpoint,
}

# ---- 4. Authorization URL ----
auth_params = urllib.parse.urlencode({
    "response_type": "code",
    "client_id": client_id,
    "redirect_uri": "http://localhost:8888/callback",
    "scope": "openid email profile",
    "state": state,
    "code_challenge": code_challenge,
    "code_challenge_method": "S256"
})
auth_url = f"{auth_endpoint}?{auth_params}"
print(f"\n🔗 Connect Flax: {auth_url}\n")

# ---- 5. Callback server (verify state, exchange code) ----
class OAuthHandler(http.server.BaseHTTPRequestHandler):
    def do_GET(self):
        parsed = urllib.parse.urlparse(self.path)
        qs = urllib.parse.parse_qs(parsed.query)

        if parsed.path == "/callback":
            code = qs.get("code", [None])[0]
            state_in = qs.get("state", [None])[0]
            error = qs.get("error", [None])[0]

            if error:
                self.send_error(400, error)
                print(f"ERROR: {error}")
                return
            if state_in != params["state"]:      # CSRF guard — must match
                self.send_error(400, "State mismatch")
                return
            if not code:
                self.send_error(400, "No code received")
                return

            token_payload = {
                "grant_type": "authorization_code",
                "code": code,
                "redirect_uri": params["redirect_uri"],
                "client_id": params["client_id"],
                "code_verifier": params["code_verifier"]
            }
            result = subprocess.run(["curl", "-sL", "-m", "10", "-X", "POST",
                "-H", "Content-Type: application/json", "-d", json.dumps(token_payload),
                params["token_endpoint"]], capture_output=True, text=True)
            token_data = json.loads(result.stdout)

            if "error" in token_data:
                self.send_error(400, f"Token exchange failed: {result.stdout[:200]}")
                print(f"TOKEN ERROR: {result.stdout}")
                return

            with open("/tmp/flax_tokens.json", "w") as f:
                json.dump(token_data, f, indent=2)

            self.send_response(200)
            self.send_header("Content-Type", "text/html")
            self.end_headers()
            self.wfile.write(b"<h1>Authorization Successful!</h1><p>You can close this window.</p>")
            print("SUCCESS: Tokens saved to /tmp/flax_tokens.json")
            threading.Thread(target=self.server.shutdown).start()
            return

        self.send_error(404)

server = http.server.HTTPServer(("127.0.0.1", 8888), OAuthHandler)
print("Waiting for authorization...")
server.serve_forever()
```

## After authorization

```python
with open("/tmp/flax_tokens.json") as f:
    tokens = json.load(f)

payload = json.dumps({
    "jsonrpc": "2.0", "id": 1, "method": "initialize",
    "params": {"protocolVersion": "2024-11-05", "capabilities": {},
               "clientInfo": {"name": "hermes-agent", "version": "1.0"}}
})
result = subprocess.run(["curl", "-sL", "-m", "10", "-X", "POST",
    "-H", "Content-Type: application/json",
    "-H", f"Authorization: Bearer {tokens['access_token']}",
    "-d", payload, mcp_url], capture_output=True, text=True)
```

## Token refresh

Access tokens expire after 3600s. Refresh via `references/token-refresh.md` —
no need to re-run the full PKCE flow.
