# oauth2-api-service (headless HTTP)

**oauth2-api-service** is a deployed OAuth2 + REST transaction API for clients that cannot embed an app SDK. Use it when the integration must be HTTP-only: OAuth tokens in, signed transactions out.

**Parallel path (HARD):** This service is **independent of xion.js**. A React or React Native app using Abstraxion does **not** need oauth2-api-service as a dependency. Choose one surface per integration — dashboard/SDK UX via xion.js, or headless OAuth REST via this service. See [integration-paths.md](./integration-paths.md) rule DR-6.

## When to use

| Scenario | Use oauth2-api-service |
|----------|------------------------|
| Swift / Kotlin / server app with no on-device signing | Yes — OAuth token + `/api/v1/*` |
| Headless backend submitting txs on behalf of users | Yes |
| React / Next.js dApp with Abstraxion provider | **No** — use [xion-js-web.md](./xion-js-web.md) |
| Native signing on device | **No** — use [mob.md](./mob.md) |

OAuth hosts: [networks.md](./networks.md) (`oauth2.testnet.burnt.com`, `oauth2.burnt.com`).

## OAuth2 flow (PKCE)

The service implements authorization code flow with **PKCE** and refresh tokens:

1. **Discovery** — `GET /.well-known/oauth-authorization-server`
2. **Authorize** — `GET /oauth/authorize` with PKCE `code_challenge` / `code_challenge_method=S256`
3. **User consent** — dashboard redirect; user approves grant
4. **Token exchange** — `POST /oauth/token` with `grant_type=authorization_code`, `code`, `code_verifier`, `client_id`
5. **Refresh** — `POST /oauth/token` with `grant_type=refresh_token` when access token expires (default TTL 1 day; refresh TTL 30 days)
6. **Submit txs** — `POST /api/v1/transaction` with `Authorization: Bearer <access_token>`

Access tokens are validated by the provider before `/api/v1/*` handlers run. Grant props (scopes, user, client) are available on the execution context.

## Scopes

| Scope | Purpose |
|-------|---------|
| `xion:identity:read` | Identity on `/api/v1/me` |
| `xion:blockchain:read` | Chain read operations |
| `xion:transactions:submit` | Submit and simulate transactions |

Management API scopes (separate from user API):

| Scope | Purpose |
|-------|---------|
| `xion:mgr:read` | Read OAuth client records via `/mgr-api` |
| `xion:mgr:write` | Create, update, delete clients |

Session JWTs issued after wallet challenge (`POST /auth/verify`) embed `grantedScopes` for mgr-api. Tokens issued before the 0.1.0 security release lack scopes and receive `403 INSUFFICIENT_SCOPE` until the user re-runs challenge + verify.

## Protected user API (`/api/v1/*`)

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/api/v1/transaction` | POST | Submit blockchain transactions (protobuf message bodies supported) |
| `/api/v1/transaction/simulate` | POST | Simulate without broadcasting |
| `/api/v1/transaction/{hash}/status` | GET | Poll transaction status |

Build messages with `@burnt-labs/xion-types` for type-safe protobuf encoding. Example message shape:

```json
{
  "messages": [
    {
      "typeUrl": "/cosmos.bank.v1beta1.MsgSend",
      "value": { "fromAddress": "...", "toAddress": "...", "amount": [{ "denom": "uxion", "amount": "1000000" }] }
    }
  ],
  "memo": "optional"
}
```

## Session auth (operator / mgr-api)

Human-operated management calls use wallet challenge-response — **not** SDK session minting:

| Step | Endpoint |
|------|----------|
| Challenge | `POST /auth/challenge` |
| Verify | `POST /auth/verify` → session JWT with `grantedScopes` |

### Disabled route

`POST /auth/sdk-session` returns **410** (`SDK_SESSION_DISABLED`). Do not document, call, or depend on it. Any integration that minted sessions from an arbitrary `xion1` address must migrate to challenge + verify. See [troubleshooting.md](./troubleshooting.md).

## Client management (`/mgr-api`)

OAuth client CRUD is authenticated via session JWT and/or OAuth access token (dual bearer). For CLI-based client management, use the `verona-oauth2-client` skill — do not duplicate CLI reference here.

### Treasury as SSOT (0.1.0+)

On-chain treasury params are the source of truth for client branding and redirect metadata:

| Client field | Treasury on-chain param |
|--------------|-------------------------|
| `redirectUris` | `redirect_url` |
| `logoUri` | `icon_url` |
| `clientUri` | `display_url` |

`POST /mgr-api/clients` requires `bindedTreasury` (caller must be treasury admin). Do **not** pass free-form `redirectUris`, `logoUri`, `clientUri`, or `owner` — they are derived from the bound treasury.

After updating treasury params on-chain, sync the OAuth client:

```
POST /mgr-api/clients/{clientId}/sync-from-treasury
```

`GET /oauth/authorize` fails closed with `400 REDIRECT_URI_TREASURY_MISMATCH` when `redirect_uri` ≠ bound treasury `redirect_url`.

## Public endpoints

| Endpoint | Purpose |
|----------|---------|
| `GET /health` | Health check |
| `GET /.well-known/oauth-authorization-server` | OAuth metadata |
| `GET /oauth/authorize` | Start authorization |
| `POST /oauth/token` | Code exchange or refresh |

## xion.js vs oauth2-api-service

| | xion.js (Abstraxion) | oauth2-api-service |
|--|----------------------|-------------------|
| Integration | npm SDK in app | HTTP + OAuth tokens |
| Auth UX | Dashboard iframe / redirect / embedded | OAuth consent redirect |
| Signing | Session keys in client | Server-side session keys |
| Dependency | Standalone | Standalone — **not** required by xion.js |

## Related sections

- [networks.md](./networks.md) — OAuth hosts, chain IDs
- [integration-paths.md](./integration-paths.md) — DR-6 path selection
- [troubleshooting.md](./troubleshooting.md) — sdk-session 410, mgr scope migration, treasury redirect mismatch
- [mob.md](./mob.md) — on-device signing alternate

## Toolkit cross-links

| Intent | Skill id |
|--------|----------|
| OAuth client CRUD via CLI | `verona-oauth2-client` |
| CLI login / gasless auth | `verona-oauth2` |
| Chain queries / CosmWasm | `verona-bin` |

## Source references

Verified against `oauth2-api-service` repo: `README.md`, `CHANGELOG.md` (0.1.0 breaking changes), `CONCEPTS.md`.
