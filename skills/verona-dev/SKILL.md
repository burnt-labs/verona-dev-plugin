---
name: verona-dev
description: |
  Default entry router for ALL Verona development in this plugin. Use this skill whenever the user mentions Verona, MetaAccount, gasless auth, Treasury, OAuth2, xiond, CosmWasm, testnet faucet, NFT minting, or building on Burnt Labs — even if they do not say "verona-dev" or "toolkit".

  Routes MetaAccount and toolkit work to in-plugin skills; routes chain queries, txs, xiond install, and CosmWasm to verona-bin. Do not treat external xion-skills repos as the primary install path.

  Triggers on: verona, xion, MetaAccount, gasless, 无 gas, Treasury, OAuth2, xiond, wasm, CosmWasm, faucet, testnet tokens, NFT, mint, burnt labs, build on xion, xion dapp, xion 开发, session key, verona agent toolkit.
metadata:
  author: burnt-labs
  version: "2.1.0"
  requires:
    - verona-toolkit-init
  recommends:
    - verona-toolkit-init
    - verona-oauth2
    - verona-oauth2-client
    - verona-treasury
    - verona-asset
    - verona-faucet
    - verona-bin
  compatibility: Entry router for Verona development in verona-dev-plugin
---

# verona-dev

Session entry for Verona work in **verona-dev-plugin**. Read this skill first, then load the routed skill or reference below.

## Philosophy

**MetaAccount-first.** For product-facing dApps targeting gasless MetaAccount UX, default to **xion.js** (Abstraxion) before considering **mob** or **oauth2-api-service**.

**Two-lane routing:**

- **App integration** (web, mobile, native signing, headless OAuth REST) → read `references/` guide files under this skill. Start at [references/README.md](./references/README.md) when the path is unclear.
- **CLI operations** (toolkit login, treasury admin, OAuth client CRUD, faucet, NFT CLI) → hand off to leaf skills (`verona-oauth2`, `verona-treasury`, etc.). Do not inline CLI command reference here.
- **Chain / CosmWasm** → `verona-bin` for `xiond` queries, txs, mnemonic wallet, and wasm lifecycle.

When both app-build and CLI signals appear, prefer **app-sdk** when the primary goal is application integration; prefer **cli** when the primary goal is running toolkit commands.

## Routing table

| User intent | Route to | Examples |
|-------------|----------|----------|
| Stack selection / which SDK | `references/integration-paths.md` | "which sdk", "stack selection", "framework choice" |
| Web dApp / frontend setup | `references/xion-js-web.md` | "react dapp", "next.js", "vue", "svelte", "abstraxion", "xion.js" |
| React Native / Expo (MetaAccount UX) | `references/xion-js-web.md` + `references/auth-and-signing.md` | "react native", "expo", "mobile app", "abstraxion-react-native" |
| Native signing (mob) | `references/mob.md` | "mob", "mob-expo", "swift", "kotlin", "native signing", "uniffi" |
| OAuth REST / headless HTTP txs | `references/oauth2-api-service.md` | "oauth rest", "pkce", "headless oauth", "http transaction", "no sdk" |
| Networks / chain IDs / endpoints | `references/networks.md` | "chain id", "testnet-2", "rpc endpoint", "oauth host" |
| Auth modes / signing / treasury SDK config | `references/auth-and-signing.md` | "auth modes", "session key", "requireAuth", "treasury config" |
| Migration / stale docs | `references/troubleshooting.md` | "package rename", "testnet migration", "stale docs" |
| Toolkit install / env setup | `verona-toolkit-init` | "install verona toolkit", "setup dev environment" |
| Login / OAuth / MetaAccount auth (CLI) | `verona-oauth2` | "login via cli", "gasless auth cli", "toolkit login" |
| OAuth app / client CRUD (CLI) | `verona-oauth2-client` | "register oauth app", "client manager" |
| Treasury create / fund / withdraw (CLI) | `verona-treasury` | "fund treasury", "fee grant", "authz grant" |
| NFT / asset operations (CLI) | `verona-asset` | "mint NFT", "collection" |
| Testnet tokens (CLI) | `verona-faucet` | "faucet", "testnet tokens" |
| `xiond` queries, txs, wallet keys | `verona-bin` → `references/usage.md` | "query balance", "send tx", "xiond" |
| `xiond` install / upgrade | `verona-bin` → `references/init.md` | "install xiond", "upgrade xiond" |
| CosmWasm deploy / interact | `verona-bin` → `references/wasm.md` | "deploy contract", "wasm" |

## Workflow

1. **Detect intent** — match the user message to the routing table (app-sdk → `references/`; cli → leaf skill).
2. **Confirm** — tell the user which reference file or skill you are loading and why.
3. **Hand off** — for leaf skills, Read that skill's `SKILL.md` and follow it; for app-sdk, Read the matched `references/<file>.md` and follow it. Do not re-implement commands or SDK setup here.

## Decision matrix

| User needs | Skill / guide | Tool |
|------------|---------------|------|
| Stack selection / integration path | `references/integration-paths.md` | xion.js / mob / oauth2-api-service |
| Web dApp (React, Next, Vue, Svelte, vanilla) | `references/xion-js-web.md` | `@burnt-labs/abstraxion-react` or `-js` |
| React Native / Expo (full MetaAccount UX) | `references/xion-js-web.md` + `references/auth-and-signing.md` | `@burnt-labs/abstraxion-react-native` |
| Native signing (Swift, Kotlin, mob, mob-expo) | `references/mob.md` | `mob` crate / `@burnt-labs/mob-expo` |
| Headless OAuth + REST transactions | `references/oauth2-api-service.md` | oauth2-api-service HTTP API |
| Chain IDs, RPC, OAuth hosts | `references/networks.md` | `@burnt-labs/constants` |
| Auth modes, session key, treasury SDK bridge | `references/auth-and-signing.md` | Abstraxion SDK |
| Login / authentication (CLI) | `verona-oauth2` | verona-toolkit |
| OAuth client registration / CRUD (CLI) | `verona-oauth2-client` | verona-toolkit |
| Treasury create, fund, withdraw, grants (CLI) | `verona-treasury` | verona-toolkit |
| NFT collection / mint / royalties (CLI) | `verona-asset` | verona-toolkit |
| Testnet tokens / faucet (CLI) | `verona-faucet` | verona-toolkit |
| Install or upgrade xiond | `verona-bin` → `init.md` | xiond |
| Chain queries, txs, mnemonic wallet | `verona-bin` → `usage.md` | xiond |
| CosmWasm store / instantiate / migrate | `verona-bin` → `wasm.md` | xiond |

## Networks

| Network | OAuth2 API | Chain ID |
|---------|------------|----------|
| testnet | oauth2.testnet.burnt.com | xion-testnet-2 |
| mainnet | oauth2.burnt.com | xion-mainnet-1 |

For full RPC/REST/dashboard endpoints, read [references/networks.md](./references/networks.md).

## Parameter validation

`scripts/validate-params.sh` validates JSON against sibling skill schemas under the install root:

```bash
skills/verona-dev/scripts/validate-params.sh verona-treasury grant-config-add '{"address": "xion1...", "preset": "send"}'
```

## Install / update

Manual host install only — see [INSTALL.md](../../INSTALL.md) at the plugin repository root (Cursor local path or marketplace, Kimi, Claude Code, Codex). There is no `npx` package or install CLI for this plugin.

Changelog: https://github.com/burnt-labs/verona-dev-plugin/releases

## Resources

- [Verona Documentation](https://docs.verona.dev/en)
- [verona-dev-plugin](https://github.com/burnt-labs/verona-dev-plugin)
- [Development guide index](./references/README.md)
