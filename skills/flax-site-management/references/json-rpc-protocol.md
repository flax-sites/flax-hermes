# Flax MCP — JSON-RPC 2.0 Wire Format

Transport: `HTTP POST`, `Content-Type: application/json`,
`Authorization: Bearer <access_token>`.

## Handshake order (never skip)

1. `initialize` — get `protocolVersion`, `serverInfo`, and `instructions`.
2. `tools/list` — get the available tools and their input schemas.
3. `tools/call` — invoke a tool; read `result.structuredContent`.

## Payloads

### initialize

```json
{"jsonrpc":"2.0","id":1,"method":"initialize",
 "params":{"protocolVersion":"2024-11-05","capabilities":{},
           "clientInfo":{"name":"hermes-agent","version":"1.0"}}}
```

Response:

```json
{"jsonrpc":"2.0","id":1,"result":{
  "protocolVersion":"2024-11-05",
  "capabilities":{"tools":{}},
  "serverInfo":{"name":"flax-agent-api","version":"v1"},
  "instructions":"Read the current site model and hash before proposing content changes..."}}
```

The server's `instructions` field is agent guidance — read and follow it.

### tools/list

```json
{"jsonrpc":"2.0","id":2,"method":"tools/list","params":{}}
```

Returns 10 tools: `flax_list_sites`, `flax_get_site_model`,
`flax_get_site_insights`, `flax_get_search_performance`, `flax_upsert_article`,
`flax_validate_model_update`, `flax_propose_model_update`, `flax_upload_image`,
`flax_begin_image_upload`, `flax_get_change`. Each has `name`, `description`,
and `inputSchema`.

### tools/call

```json
{"jsonrpc":"2.0","id":3,"method":"tools/call",
 "params":{"name":"flax_get_site_model",
           "arguments":{"siteId":"b07cd9e3-..."}}}
```

## Response envelope

- Prefer `result.structuredContent` — typed, canonical data
  (e.g. `structuredContent.model`, `.hash`, `.schemaUrl`, `.agentHints`).
- `result.content[].text` is a JSON-encoded string of the same data — parse it
  only when `structuredContent` is absent.
- `result.isError: true` → the real error object is in `result.content[0].text`.

## Error codes → action

| Error | Meaning | Action |
|-------|---------|--------|
| `401 {"error":"Authentication required"}` | token missing/expired | refresh per `references/token-refresh.md` |
| `{"valid":false,"retryable":true,"error":{"code":"stale_model"}}` | live model changed since `baseModelHash` | re-fetch model, use new hash, resubmit |
| `{"error":{"code":"agent_request_failed","message":"Model path does not exist: X"}}` | field absent from live model | do not add it — use an existing field |
| `{"error":{"code":"site_required","message":"siteId is required."}}` | missing argument | add `siteId` to `arguments` |
