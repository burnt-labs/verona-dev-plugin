# Verona development guide

Progressive reference for building applications on XION with MetaAccount, gasless auth, and treasury-backed transactions. Use this guide when the user is integrating an **app SDK** (web, mobile, native, or headless HTTP). For toolkit CLI operations (login, treasury admin, OAuth client CRUD), route to the leaf skills listed below instead.

## How to use this guide

1. **Start here** if the integration path is unclear — read [integration-paths.md](./integration-paths.md) for the decision tree and default rules (DR-1–DR-8).
2. **Load one section** that matches the chosen path; do not read the entire guide unless the task spans multiple layers (e.g. web app + network config).
3. **Cross-link toolkit skills** for CLI steps — the guide explains SDK concepts and links by skill id; it does not duplicate CLI command reference.
4. **Prefer local source** over public docs when they conflict: xion.js, mob, and oauth2-api-service package READMEs, constants, and CHANGELOGs override stale tutorials. See [troubleshooting.md](./troubleshooting.md) for known traps.

### MetaAccount-first default

For product-facing dApps targeting gasless MetaAccount UX, default to **xion.js** (Abstraxion) before considering **mob** or **oauth2-api-service**. Use alternates only when the matrix row or an explicit user constraint requires them.

## Guide sections

| Section | Purpose |
|---------|---------|
| [integration-paths.md](./integration-paths.md) | Choose integration path — matrix summary, default rules DR-1–DR-8, parallel-path boundaries |
| [networks.md](./networks.md) | Chain IDs (`xion-testnet-2`, `xion-mainnet-1`), RPC/REST, OAuth hosts, dashboard URLs |
| [xion-js-web.md](./xion-js-web.md) | Web quick start — `@burnt-labs/abstraxion-react` (React/Next) and `@burnt-labs/abstraxion-js` (non-React) |
| [auth-and-signing.md](./auth-and-signing.md) | Auth modes, session key vs `requireAuth`, treasury SDK config; bridge to `verona-treasury` for CLI |
| [oauth2-api-service.md](./oauth2-api-service.md) | Alternate headless path — PKCE, scopes, REST transaction submission (no app SDK) |
| [mob.md](./mob.md) | Alternate native signing — Rust `mob` crate, `@burnt-labs/mob-expo` for RN; when to pick over Abstraxion RN |
| [troubleshooting.md](./troubleshooting.md) | Package renames, testnet-2 migration, stale docs; chain/wasm queries delegate to `verona-bin` |

## Read next (by scenario)

| Scenario | Read first | Then if needed |
|----------|------------|----------------|
| Ambiguous stack / "which SDK?" | [integration-paths.md](./integration-paths.md) | Target section from decision outcome |
| React or Next.js web dApp | [xion-js-web.md](./xion-js-web.md) | [networks.md](./networks.md), [auth-and-signing.md](./auth-and-signing.md) |
| Vue, Svelte, or vanilla web | [xion-js-web.md](./xion-js-web.md) (Abstraxion JS path) | [auth-and-signing.md](./auth-and-signing.md) |
| React Native / Expo (full MetaAccount UX) | [xion-js-web.md](./xion-js-web.md) | [auth-and-signing.md](./auth-and-signing.md) |
| RN with native Rust signing / minimal JS | [mob.md](./mob.md) | [networks.md](./networks.md) |
| Swift, Kotlin, Python, Ruby, or Rust app signing | [mob.md](./mob.md) | [networks.md](./networks.md) |
| No SDK — OAuth token + HTTP txs only | [oauth2-api-service.md](./oauth2-api-service.md) | [networks.md](./networks.md) |
| Chain ID, RPC, or OAuth host lookup | [networks.md](./networks.md) | — |
| Migration, rename, or doc conflicts | [troubleshooting.md](./troubleshooting.md) | Relevant path section |

## Toolkit leaf skills (CLI boundary)

These skills execute toolkit commands. The guide cross-links them; do not inline their CLI reference here.

| Skill id | Use when |
|----------|----------|
| `verona-toolkit-init` | Dev environment and toolkit install |
| `verona-oauth2` | CLI login and gasless auth flows |
| `verona-oauth2-client` | OAuth application CRUD via CLI |
| `verona-treasury` | Treasury create, fund, withdraw, and grant operations via CLI |
| `verona-asset` | NFT and asset CLI operations |
| `verona-faucet` | Testnet faucet via CLI |
| `verona-bin` | `xiond` chain queries, mnemonic wallet, CosmWasm — not MetaAccount app setup |

## Parallel integration surfaces

- **xion.js** and **oauth2-api-service** are **parallel** paths (dashboard/SDK UX vs OAuth REST). Do not treat oauth2-api-service as a required dependency of xion.js.
- **mob** is the native / multi-language **signing** stack. **Abstraxion** remains the full MetaAccount **UX** stack for JS web and RN when dashboard auth modes and treasury SDK config are needed.

## Upstream packages (quick reference)

| Repo | Published names |
|------|-----------------|
| xion.js | `@burnt-labs/abstraxion-react`, `@burnt-labs/abstraxion-js`, `@burnt-labs/abstraxion-react-native`, `@burnt-labs/abstraxion-core`, `@burnt-labs/constants` |
| mob | `mob` (Rust crate), `@burnt-labs/mob-expo` (React Native) |
| oauth2-api-service | Deployed at `oauth2.testnet.burnt.com` / `oauth2.burnt.com` — HTTP integration, not an npm app SDK |

Legacy note: `@burnt-labs/abstraxion` was renamed to `@burnt-labs/abstraxion-react`. See [troubleshooting.md](./troubleshooting.md).
