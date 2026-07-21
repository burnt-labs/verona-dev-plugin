# xion.js web quick start

**xion.js** is the preferred integration path for JavaScript web and React Native apps that need MetaAccount UX (dashboard auth, session keys, treasury-backed grants). See [integration-paths.md](./integration-paths.md) for when to choose alternates (`mob`, `oauth2-api-service`).

## Package map

| Package | Use when |
|---------|----------|
| `@burnt-labs/abstraxion-react` | **Default** — React, Next.js, or any React-based web app |
| `@burnt-labs/abstraxion-js` | Vue, Svelte, vanilla JS, or custom framework hosts |
| `@burnt-labs/abstraxion-react-native` | Expo / React Native (same hook surface, RN transports) |
| `@burnt-labs/constants` | Chain IDs, RPC/REST URLs, gas defaults |

Legacy rename: `@burnt-labs/abstraxion` → `@burnt-labs/abstraxion-react`. Legacy `@burnt-labs/ui` modal is superseded by `<AbstraxionEmbed>`.

## React / Next.js (preferred)

Install:

```bash
npm i @burnt-labs/abstraxion-react
```

Minimal app shell — provider, embed, account hook:

```tsx
import {
  AbstraxionProvider,
  AbstraxionEmbed,
  useAbstraxionAccount,
  useAbstraxionSigningClient,
} from "@burnt-labs/abstraxion-react";

const config = {
  chainId: "xion-testnet-2",
  treasury: "xion1...",
  authentication: { type: "auto" }, // popup on desktop, redirect on mobile/PWA
};

export default function App() {
  return (
    <AbstraxionProvider config={config}>
      <AbstraxionEmbed />
      <Account />
    </AbstraxionProvider>
  );
}

function Account() {
  const { data: account, isConnected } = useAbstraxionAccount();
  const { client } = useAbstraxionSigningClient(); // session key (default)

  if (!isConnected) return <p>Connect via the embed above</p>;
  return <p>{account?.bech32Address}</p>;
}
```

### Recommended auth mode

Use `authentication: { type: "auto" }` for new web apps. It resolves to popup on desktop and redirect on mobile/PWA during config normalization. Other dashboard modes (`popup`, `redirect`, `embedded`) are documented in [auth-and-signing.md](./auth-and-signing.md).

### Next.js

Mark client components with `"use client"` on files that import hooks or `<AbstraxionEmbed>`. Wrap the root layout (or a dedicated provider layout) with `<AbstraxionProvider config={...}>`.

### Network config

Set `chainId` to `xion-testnet-2` or `xion-mainnet-1`. RPC, REST, and gas price default from `@burnt-labs/constants` — see [networks.md](./networks.md).

## Non-React web (`@burnt-labs/abstraxion-js`)

For Svelte, Vue, or vanilla JS, use the framework-agnostic runtime:

```bash
npm i @burnt-labs/abstraxion-js
```

```ts
import { createAbstraxionRuntime } from "@burnt-labs/abstraxion-js";

const runtime = createAbstraxionRuntime({
  chainId: "xion-testnet-2",
  treasury: "xion1...",
  authentication: { type: "auto" },
});

runtime.subscribe((state) => {
  // mirror state into your framework's reactivity primitive
});

await runtime.login();
```

Worked example: `xion.js/demos/svelte` — `createAbstraxionStore()` wraps `createAbstraxionRuntime` with Svelte stores; port the same pattern to Vue/Solid by swapping the store layer.

Key runtime methods (from demos):

- `runtime.login()` / `runtime.logout()`
- `runtime.createReadClient()` — CosmWasm / chain queries
- `runtime.createDirectSigningClient()` — meta-account direct signing (`requireAuth` path)

## React Native (`@burnt-labs/abstraxion-react-native`)

Same public hooks as React (`useAbstraxionAccount`, `useAbstraxionSigningClient`, `<AbstraxionEmbed>`), with RN-specific transports:

```bash
npm i @burnt-labs/abstraxion-react-native \
      @react-native-async-storage/async-storage \
      expo-web-browser expo-linking
# embedded mode only:
npm i react-native-webview
```

Peer dependencies include `react-native-get-random-values` and `react-native-quick-crypto` (signing requires a custom dev client — not Expo Go).

Redirect mode uses Expo WebBrowser + deep-link `callbackUrl`. Embedded mode uses in-app WebView via `<AbstraxionEmbed>`.

Demo: `xion.js/demos/react-native`.

For native Rust signing without dashboard UX, see [mob.md](./mob.md) and integration rule DR-4.

## Treasury address

The `treasury` field in config is your on-chain treasury **contract address** (bech32 `xion1...`). The SDK queries it (via DaoDao indexer when available) to show grant permissions during connect.

To **create or manage** a treasury contract, use the `verona-treasury` skill (CLI) — do not duplicate CLI steps here. After creation, paste the contract address into `AbstraxionProvider` config.

## Signing quick reference

| Goal | Hook / API | Gas |
|------|------------|-----|
| Normal dApp txs (gasless) | `useAbstraxionSigningClient()` (default) | Fee grant via session key |
| Security-sensitive txs | `useAbstraxionSigningClient({ requireAuth: true })` | User meta-account pays gas |

Details: [auth-and-signing.md](./auth-and-signing.md).

## Demos and upstream docs

| Demo | Path |
|------|------|
| React (auto, embedded, signer modes) | `xion.js/demos/react` |
| Svelte (`abstraxion-js`) | `xion.js/demos/svelte` |
| React Native | `xion.js/demos/react-native` |

Package READMEs: `xion.js/packages/abstraxion-react`, `abstraxion-js`, `abstraxion-react-native`.

## Related sections

- [networks.md](./networks.md) — chain IDs and endpoints
- [auth-and-signing.md](./auth-and-signing.md) — auth modes, session key vs `requireAuth`, treasury config
- [integration-paths.md](./integration-paths.md) — path selection
