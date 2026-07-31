# Troubleshooting and migration traps

Known pitfalls when integrating Verona / XION application SDKs. For path selection mistakes, see [integration-paths.md](./integration-paths.md) § Common misroutes.

## Source precedence

When documentation conflicts, trust sources in this order:

1. **Local library repos** — `xion.js`, `mob`, `oauth2-api-service` package READMEs, `CHANGELOG.md`, constants, and type definitions
2. **This plugin guide** — `skills/verona-dev/references/`
3. **docs.verona.dev** — supplementary; treat conflicts as **stale** and prefer items 1–2

If a public tutorial references removed endpoints, legacy chain IDs, or renamed packages, follow the local repo and this guide.

## Package renames

| Legacy | Current | Notes |
|--------|---------|-------|
| `@burnt-labs/abstraxion` | `@burnt-labs/abstraxion-react` | Same React provider/hooks; update `package.json` and imports |
| `@burnt-labs/ui` modal | `<AbstraxionEmbed>` in `@burnt-labs/abstraxion-react` | Modal package superseded |

Search-replace imports:

```diff
- import { AbstraxionProvider } from "@burnt-labs/abstraxion";
+ import { AbstraxionProvider } from "@burnt-labs/abstraxion-react";
```

Non-React web apps use `@burnt-labs/abstraxion-js`, not the React package.

## testnet-2 migration

Legacy `xion-testnet-1` is **not** a target for new integrations. Use `xion-testnet-2`:

| Field | Legacy (`xion-testnet-1`) | Current (`xion-testnet-2`) |
|-------|---------------------------|------------------------------|
| Chain ID | `xion-testnet-1` | `xion-testnet-2` |
| RPC | `https://rpc.xion-testnet-1.burnt.com:443` | `https://rpc.xion-testnet-2.burnt.com:443` |
| REST | `https://api.xion-testnet-1.burnt.com:443` | `https://api.xion-testnet-2.burnt.com:443` |
| Dashboard | `https://auth.testnet.burnt.com` | `https://auth.testnet.burnt.com` |

Update `chainId` in Abstraxion config, mob `ChainConfig`, and any hardcoded RPC URLs. `@burnt-labs/constants` resolves correct endpoints when you pass only `chainId`.

**mob README note:** upstream examples may still show `xion-testnet-1`; integration tests in mob target testnet-2 RPC. Always align with [networks.md](./networks.md).

## oauth2-api-service breaking changes (0.1.0)

| Trap | Fix |
|------|-----|
| `POST /auth/sdk-session` | **Removed** — returns `410 SDK_SESSION_DISABLED`. Use `POST /auth/challenge` → `POST /auth/verify` |
| Create client with `redirectUris` / `logoUri` / `clientUri` | Set treasury `redirect_url` / `icon_url` / `display_url` on-chain first; create with `bindedTreasury` only |
| PUT client redirect fields | Update treasury on-chain → `POST /mgr-api/clients/{id}/sync-from-treasury` |
| Session JWT without `grantedScopes` | Re-run challenge + verify (TTL typically ≤60 minutes) |
| `GET /mgr-api/clients/{id}` returns 403 not 404 | Expected — no existence enumeration for inaccessible clients |
| `redirect_uri` mismatch at authorize | `400 REDIRECT_URI_TREASURY_MISMATCH` — must match bound treasury `redirect_url` |

Details: [oauth2-api-service.md](./oauth2-api-service.md).

## Parallel path confusion

| Mistake | Correct approach |
|---------|------------------|
| Adding oauth2-api-service to a React Abstraxion app | xion.js handles auth UX; oauth2 REST is a **parallel** alternate, not a dependency |
| Using `verona-oauth2` skill for in-app OAuth PKCE | App integration → xion.js or [oauth2-api-service.md](./oauth2-api-service.md); CLI skill is for toolkit commands |
| Using xion.js for CosmWasm deploy scripts | Route to `verona-bin` skill |
| Using mob-expo when full auth modes are needed | Default to `@burnt-labs/abstraxion-react-native` — see [mob.md](./mob.md) |

## Stale docs.verona.dev symptoms

Treat public docs as stale when you see:

- `@burnt-labs/abstraxion` without rename note
- `xion-testnet-1` as the recommended testnet
- `POST /auth/sdk-session` for session minting
- oauth2-api-service described as required by xion.js
- RPC URLs or dashboard hosts that differ from [networks.md](./networks.md)

Prefer `@burnt-labs/constants` and this guide's [networks.md](./networks.md) for endpoint values.

## React Native signing

| Symptom | Likely cause |
|---------|--------------|
| Signing fails in Expo Go | Abstraxion RN requires custom dev client (`react-native-quick-crypto`, `react-native-get-random-values`) |
| Missing embedded auth | mob-expo supports dashboard redirect only — use Abstraxion RN for embedded WebView mode |
| "Grants not found on-chain" after mob-expo login | Treasury or grant config mismatch; verify `treasury` address and grant params |

## Chain / CosmWasm / mnemonic operations

Application SDK guides do not cover `xiond` CLI, CosmWasm lifecycle, or mnemonic wallet flows. Route to the `verona-bin` skill.

Treasury contract **creation** and CLI grant management route to the `verona-treasury` skill. SDK config only needs the treasury **contract address**.

## Quick symptom → section map

| Symptom | Read |
|---------|------|
| Wrong SDK for stack | [integration-paths.md](./integration-paths.md) |
| Wrong chain ID or RPC | [networks.md](./networks.md) |
| Auth mode / session key confusion | [auth-and-signing.md](./auth-and-signing.md) |
| Headless OAuth / 410 sdk-session | [oauth2-api-service.md](./oauth2-api-service.md) |
| RN native signing vs full UX | [mob.md](./mob.md) |
| Web quick start | [xion-js-web.md](./xion-js-web.md) |
