# Integration path selection

Normative decision guide for choosing a Verona / XION application integration surface. When multiple options apply, use the **default rules** below unless the user explicitly constrains otherwise.

**MetaAccount-first:** For product-facing dApps with gasless MetaAccount UX, prefer **xion.js** (DR-1–DR-3) before **mob** or **oauth2-api-service**.

## Decision flow

```
What are you building?
│
├─ Web UI
│   ├─ React or Next.js ──────────────► @burnt-labs/abstraxion-react     [DR-1]
│   └─ Vue, Svelte, vanilla, other ────► @burnt-labs/abstraxion-js       [DR-2]
│
├─ React Native / Expo
│   ├─ Full MetaAccount UX (auth modes, dashboard) ► @burnt-labs/abstraxion-react-native [DR-3]
│   └─ Native Rust signing, lighter JS, no dashboard ► @burnt-labs/mob-expo [DR-4]
│
├─ Mobile native (Swift / Kotlin)
│   ├─ On-device signing ─────────────► mob (UniFFI)                      [DR-5]
│   └─ No on-device signing ──────────► oauth2-api-service REST           [DR-6]
│
├─ Server / backend (Python, Ruby, Rust)
│   ├─ Signing runtime on platform ───► mob                             [DR-5]
│   └─ HTTP-only client ──────────────► oauth2-api-service REST           [DR-6]
│
├─ Headless — no SDK in client
│   └─ OAuth token + HTTP txs only ───► oauth2-api-service                [DR-6]
│
├─ Server automation / agent CLI
│   └─ Treasury or OAuth admin ───────► verona-toolkit leaf skills        [DR-7]
│
└─ Chain queries, CosmWasm, mnemonic wallet
    └─ Not app MetaAccount setup ─────► verona-bin (xiond)                [DR-8]
```

After choosing a path, read the matching guide section: [xion-js-web.md](./xion-js-web.md), [mob.md](./mob.md), [oauth2-api-service.md](./oauth2-api-service.md), or route to `verona-bin` / toolkit leaf skills as appropriate. Use [networks.md](./networks.md) for endpoints on any path.

## Default rules (DR-1–DR-8)

| Rule | Scenario | Default |
|------|----------|---------|
| **DR-1** | Web UI with React or Next.js | `@burnt-labs/abstraxion-react` |
| **DR-2** | Web UI without React (Vue, Svelte, vanilla) | `@burnt-labs/abstraxion-js` |
| **DR-3** | React Native / Expo needing full MetaAccount UX (auth modes, dashboard) | `@burnt-labs/abstraxion-react-native` |
| **DR-4** | React Native needing native Rust signing or minimal JS surface | `@burnt-labs/mob-expo` |
| **DR-5** | Swift / Kotlin / Python / Ruby / Rust app with on-device signing | **mob** |
| **DR-6** | Client cannot embed SDK; OAuth token + HTTP txs only | **oauth2-api-service** |
| **DR-7** | Server automation / agent CLI for treasury or OAuth admin | **verona-toolkit** leaf skills (not app SDK) |
| **DR-8** | Chain queries, CosmWasm, mnemonic wallet | **verona-bin** (`xiond`) |

## Tech selection matrix

| Layer | Developer scenario | Preferred | Alternate(s) | When to use alternate |
|-------|--------------------|-----------|--------------|------------------------|
| **web** | Web dApp (React / Next) | `@burnt-labs/abstraxion-react` | `@burnt-labs/abstraxion-js` | Non-React web only |
| **web** | Web (Vue / Svelte / vanilla) | `@burnt-labs/abstraxion-js` | — | — |
| **rn** | React Native / Expo (full MetaAccount UX) | `@burnt-labs/abstraxion-react-native` | `@burnt-labs/mob-expo` | Native Rust signing; lighter JS; no dashboard UX |
| **native** | Mobile native (Swift / Kotlin) | **mob** | oauth2-api-service REST | No on-device signing; token-only HTTP client |
| **server** | Python / Ruby / Rust backend services | **mob** | oauth2-api-service REST | HTTP-only client; no mob runtime on platform |
| **server** | Server OAuth + REST txs (headless) | **oauth2-api-service** | verona-toolkit CLI | Human/agent treasury or OAuth admin → toolkit leaf skills |
| **chain** | Chain queries / CosmWasm / mnemonic | **verona-bin** (`xiond`) | — | Not MetaAccount gasless UX |

## Path details

### xion.js (preferred for JS web and RN)

**xion.js** ships Abstraxion — the full MetaAccount UX stack for JavaScript ecosystems:

| Package | Use for |
|---------|---------|
| `@burnt-labs/abstraxion-react` | React and Next.js apps — provider, hooks, embed component |
| `@burnt-labs/abstraxion-js` | Non-React web (Vue, Svelte, vanilla) and custom hosts |
| `@burnt-labs/abstraxion-react-native` | Expo / React Native with dashboard auth modes and treasury SDK config |
| `@burnt-labs/constants` | Canonical chain IDs and endpoint URLs |

→ Continue with [xion-js-web.md](./xion-js-web.md) and [auth-and-signing.md](./auth-and-signing.md).

**Not** a substitute for: native Swift/Kotlin signing (use **mob**), headless HTTP-only clients (use **oauth2-api-service**), or chain/wasm operations (use **verona-bin**).

### mob (alternate — native signing)

**mob** is a Rust signing client with UniFFI bindings for Kotlin, Swift, Python, Ruby, and more. Use when the app needs on-device transaction signing without the Abstraxion dashboard UX layer.

| Artifact | Use for |
|----------|---------|
| `mob` crate | Swift, Kotlin, Python, Ruby, Rust server or native apps |
| `@burnt-labs/mob-expo` | React Native / Expo when native Rust signing is preferred over full Abstraxion RN |

→ Continue with [mob.md](./mob.md).

Pick **mob** over Abstraxion RN when you need native Rust signing, a minimal JS surface, or no dashboard-driven auth modes. Pick **Abstraxion RN** (DR-3) when you need the full MetaAccount UX (popup, redirect, embedded, treasury config via SDK).

### oauth2-api-service (alternate — headless HTTP)

**oauth2-api-service** is a deployed OAuth2 + REST transaction API — not an npm app SDK. Use when the client cannot embed any SDK and submits transactions via OAuth tokens and HTTP only.

Key characteristics:

- OAuth2 authorization code flow with **PKCE**
- Scopes such as `xion:identity:read`, `xion:blockchain:read`, `xion:transactions:submit`
- Transaction submission via `/api/v1/transaction`
- `POST /auth/sdk-session` is **disabled** (410) — do not document or depend on SDK session minting via that route

→ Continue with [oauth2-api-service.md](./oauth2-api-service.md).

**Parallel path (HARD):** xion.js and oauth2-api-service are independent integration surfaces. A xion.js app does **not** require oauth2-api-service as a dependency.

### verona-toolkit leaf skills (CLI — not app SDK)

When the goal is running toolkit commands (not embedding an SDK in an application), route to leaf skills:

| Intent | Skill id |
|--------|----------|
| Toolkit install / env | `verona-toolkit-init` |
| CLI login / gasless auth | `verona-oauth2` |
| OAuth app CRUD | `verona-oauth2-client` |
| Treasury CLI ops | `verona-treasury` |
| NFT / faucet CLI | `verona-asset`, `verona-faucet` |

The guide covers SDK treasury config and OAuth concepts for app builders; CLI execution stays in leaf skills.

### verona-bin (chain layer — not MetaAccount app setup)

Use `verona-bin` (`xiond`) for chain queries, transaction submission with mnemonic wallets, and CosmWasm lifecycle — not for MetaAccount gasless app integration.

→ Route to `verona-bin` skill references (`usage.md`, `wasm.md`, `init.md`).

## Disambiguation

| Signals in user message | Route |
|-------------------------|-------|
| "Build React dApp", "scaffold frontend", "Abstraxion provider" | App SDK → xion.js path |
| "Login via CLI", "fund treasury from command line" | CLI → `verona-oauth2` or `verona-treasury` |
| Both app-build and CLI signals | Prefer **app-sdk** when primary goal is application integration; prefer **cli** when primary goal is running toolkit commands |

## Common misroutes

| Mistake | Correct path |
|---------|--------------|
| Using oauth2-api-service inside a React Abstraxion app | xion.js handles auth UX; oauth2 REST is a parallel alternate |
| Using `verona-oauth2` for in-app OAuth PKCE integration | App SDK path or [oauth2-api-service.md](./oauth2-api-service.md) for REST |
| Using xion.js for CosmWasm deploy scripts | `verona-bin` → wasm reference |
| Assuming `@burnt-labs/abstraxion` package name | Renamed to `@burnt-labs/abstraxion-react` — see [troubleshooting.md](./troubleshooting.md) |

## Source of truth

When public documentation conflicts with package source:

1. Local library repos (xion.js, mob, oauth2-api-service) — package names, endpoints, disabled routes, CHANGELOG migrations
2. This plugin guide under `skills/verona-dev/references/`
3. docs.verona.dev — supplementary; treat conflicts as stale and note in [troubleshooting.md](./troubleshooting.md)
