# mob (native signing)

**mob** is a Rust signing client for XION with UniFFI bindings for Kotlin, Swift, Python, Ruby, and more. It targets apps that need **on-device transaction signing** without the full Abstraxion dashboard UX layer.

For MetaAccount-first product dApps with dashboard auth modes and treasury SDK config, default to **xion.js** ([xion-js-web.md](./xion-js-web.md)) — not mob. See [integration-paths.md](./integration-paths.md).

## When to use

| Scenario | Default | Alternate |
|----------|---------|-----------|
| React Native / Expo — full MetaAccount UX (auth modes, dashboard, `<AbstraxionEmbed>`) | `@burnt-labs/abstraxion-react-native` | — |
| React Native — native Rust signing, lighter JS, no dashboard UX | — | `@burnt-labs/mob-expo` |
| Swift / Kotlin native app | **mob** (UniFFI) | oauth2-api-service REST if no on-device signing |
| Python / Ruby / Rust backend with signing runtime | **mob** crate | oauth2-api-service REST for HTTP-only |

## React Native: Abstraxion RN vs mob-expo

| | `@burnt-labs/abstraxion-react-native` (default) | `@burnt-labs/mob-expo` (alternate) |
|--|--------------------------------------------------|-------------------------------------|
| **UX** | Full MetaAccount — popup, redirect, embedded, signer modes | Dashboard redirect auth only; no `<AbstraxionEmbed>` |
| **Signing** | JS session keys via abstraxion-core | Native Rust `SessionSigner` via Expo module |
| **Hooks** | `useAbstraxionAccount`, `useAbstraxionSigningClient`, etc. | Same hook names (`useAbstraxionAccount`, `useAbstraxionClient`, `useAbstraxionSigningClient`) for API compatibility |
| **Provider** | `AbstraxionProvider` | `MobProvider` |
| **Peers** | `react-native-get-random-values`, `react-native-quick-crypto` (custom dev client) | Expo modules (`expo-web-browser`, `expo-linking`, etc.) |
| **Pick when** | Product dApp needing full auth mode surface | Native performance, minimal JS bundle, or Rust signing path |

Rule of thumb: if the app needs embedded WebView auth, `requireAuth` direct signing, or the same demo surface as `xion.js/demos/react-native`, use **Abstraxion RN** (DR-3). If the app only needs dashboard redirect + native signing with compatible hooks, use **mob-expo** (DR-4).

## Rust crate (`mob`)

Add to `Cargo.toml`:

```toml
[dependencies]
mob = "0.1.0"
tokio = { version = "1", features = ["full"] }
```

### Chain config

Use current network values from [networks.md](./networks.md):

```rust
use mob::{ChainConfig, Client, Signer};

let config = ChainConfig::new(
    "xion-testnet-2",
    "https://rpc.xion-testnet-2.burnt.com:443",
    "xion"
);
```

Do not target legacy `xion-testnet-1` in new integrations.

### Session key signing (recommended)

Session keys are time-limited and wrap messages in `MsgExec` (authz) so the granter account key stays off-device:

```rust
use mob::{SessionMetadata, SessionSigner, Signer};
use std::sync::Arc;

let session_key = Signer::from_mnemonic("session mnemonic", "xion", None)?;
let metadata = SessionMetadata::with_duration(
    "xion1granter...".to_string(),
    session_key.address(),
    3600, // seconds
);
let session_signer = SessionSigner::new(Arc::new(session_key), metadata)?;
// session_signer.is_expired(), session_signer.remaining_seconds()
```

### Direct signing

For simple transfers without session delegation:

```rust
let mut client = Client::new(config).await?;
let signer = Signer::from_mnemonic("mnemonic", "xion", None)?;
client.attach_signer(signer).await?;
let response = client.send("xion1recipient...", vec![Coin::new("uxion", "1000000")], None).await?;
```

## UniFFI language bindings

Generate bindings from the compiled library:

| Language | Command |
|----------|---------|
| Python | `./scripts/generate_python_bindings.sh` |
| Kotlin | `cargo run --bin uniffi-bindgen generate --library target/release/libmob.so --language kotlin --out-dir bindings/kotlin` |
| Swift | `cargo run --bin uniffi-bindgen generate --library target/release/libmob.dylib --language swift --out-dir bindings/swift` |

See `mob/PYTHON_BINDINGS.md` and `mob/examples/` for language-specific usage.

## React Native (`@burnt-labs/mob-expo`)

Install:

```bash
npm i @burnt-labs/mob-expo \
      @react-native-async-storage/async-storage \
      expo expo-linking expo-web-browser
```

Minimal provider:

```tsx
import { MobProvider, useAbstraxionAccount } from "@burnt-labs/mob-expo";

const config = {
  chain: {
    chainId: "xion-testnet-2",
    rpcEndpoint: "https://rpc.xion-testnet-2.burnt.com:443",
    addressPrefix: "xion",
  },
  treasury: "xion1...",       // fee granter / treasury contract
  callbackUrl: "myapp://mob-auth", // deep link for dashboard redirect
};

export default function App() {
  return (
    <MobProvider config={config}>
      <Account />
    </MobProvider>
  );
}

function Account() {
  const { bech32Address, isConnected, login, logout } = useAbstraxionAccount();
  // login() opens dashboard auth; signing via useAbstraxionSigningClient after connect
}
```

`MobProvider` auto-detects dashboard URL from RPC `/status` when `dashboardUrl` is omitted (`auth.testnet.burnt.com` for `xion-testnet-2`). Treasury address is merged into `chain.feeGranter` when not set explicitly.

### Exports

| Export | Purpose |
|--------|---------|
| `MobProvider`, `useMobContext` | Context and low-level API |
| `useAbstraxionAccount`, `useAbstraxionClient`, `useAbstraxionSigningClient` | xion.js-compatible hooks |
| `SessionManager`, `openDashboardAuth` | Session persistence and dashboard redirect |
| `MobModule` | Direct native module access |

## mob vs xion.js

| | xion.js (Abstraxion) | mob |
|--|----------------------|-----|
| Runtime | JavaScript / TypeScript | Rust core + bindings |
| Auth UX | Full dashboard modes | Dashboard redirect (mob-expo) or bring-your-own |
| Platforms | Web, RN | Native (Swift/Kotlin/Python/Ruby/Rust), RN via mob-expo |
| MetaAccount UX | Primary path | Signing-only alternate |

mob is designed to replace JS signing clients for native platforms. It does **not** replace Abstraxion for product-facing web or full-UX RN apps.

## Related sections

- [networks.md](./networks.md) — chain IDs, RPC endpoints, dashboard URLs
- [xion-js-web.md](./xion-js-web.md) — default RN path (`@burnt-labs/abstraxion-react-native`)
- [auth-and-signing.md](./auth-and-signing.md) — Abstraxion auth modes and session keys
- [integration-paths.md](./integration-paths.md) — DR-3, DR-4, DR-5
- [oauth2-api-service.md](./oauth2-api-service.md) — HTTP-only alternate when no on-device signing

## Source references

Verified against `mob` repo: `README.md`, `react-native/package.json`, `react-native/src/` (`MobProvider.tsx`, `DashboardAuth.ts`, hooks).
