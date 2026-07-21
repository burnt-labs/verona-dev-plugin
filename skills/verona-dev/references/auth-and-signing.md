# Authentication and signing

How Abstraxion authenticates users and signs transactions. Applies to `@burnt-labs/abstraxion-react`, `@burnt-labs/abstraxion-js`, and `@burnt-labs/abstraxion-react-native` (same `AbstraxionConfig` shape).

Types referenced here: `xion.js/packages/abstraxion-js/src/types.ts`.

## Authentication modes

Set `authentication` on `AbstraxionConfig`. When omitted, the SDK defaults to **redirect** (dashboard OAuth flow).

| `type` | Behavior | Typical use |
|--------|----------|-------------|
| `auto` | Resolves to popup on desktop, redirect on mobile/PWA | **Recommended** for new web apps |
| `popup` | Dashboard OAuth in a popup; opener receives `CONNECT_SUCCESS` via `postMessage` | Desktop-only custom UX |
| `redirect` | Full-page navigation to auth app; returns with `?granted=true` | Mobile / PWA fallback |
| `embedded` | Dashboard rendered in an inline iframe (`<AbstraxionEmbed>`) | Inline connect / approve UI |
| `signer` | External signer (Turnkey, Privy, MetaMask, Keplr, etc.) via `getSignerConfig()` | Wallet-native or custodial signer integration |

### Mode-specific options

```ts
// auto (recommended)
authentication: { type: "auto", authAppUrl?: string, callbackUrl?: string }

// popup
authentication: { type: "popup", authAppUrl?: string }

// redirect
authentication: { type: "redirect", callbackUrl?: string, authAppUrl?: string }

// embedded
authentication: { type: "embedded", iframeUrl?: string, containerElement?: HTMLElement }

// signer (external wallet / AA provider)
authentication: {
  type: "signer",
  aaApiUrl: string,
  getSignerConfig: () => Promise<SignerConfig>,
  smartAccountContract: SmartAccountContractConfig,
  indexer?: IndexerConfig,
  treasuryIndexer?: { url: string },
}
```

`authAppUrl` / `iframeUrl` default from chain-specific constants — see [networks.md](./networks.md). Override only when testing against a non-default dashboard host.

### React Native notes

`@burnt-labs/abstraxion-react-native` supports **redirect** (Expo WebBrowser + deep link) and **embedded** (WebView). Signer mode is available where the host provides `getSignerConfig`. See `xion.js/demos/react-native`.

## Session key vs direct signing (`requireAuth`)

After connect, transaction signing follows one of two paths:

### Session key signing (default)

```tsx
const { client } = useAbstraxionSigningClient();
// or runtime.getState() → signingClient in abstraxion-js
```

- Uses `GranteeSignerClient` — a delegated session keypair granted via treasury/authz during connect.
- **Gasless** for normal operations (fee granter pays gas).
- No per-tx dashboard popup for standard sends and contract calls within granted permissions.
- Requires grant configuration at connect time (`treasury` or legacy `contracts` / `stake` / `bank`).

### Direct signing (`requireAuth: true`)

```tsx
const { client, error, signResult, clearSignResult } =
  useAbstraxionSigningClient({ requireAuth: true });
```

- Meta-account signs each transaction directly — not the session key.
- User **pays gas** from their XION balance.
- Dashboard mediates approval in popup / redirect / embedded modes (`RequireSigningClient`); signer mode uses `AAClient`.
- Use for security-sensitive operations (large transfers, smart-account management, actions outside session-key scope).

`SigningClient` union (`GranteeSignerClient` | `AAClient` | `RequireSigningClient`) is exported from `@burnt-labs/abstraxion-js`. All expose `signAndBroadcast`, `sendTokens`, and `simulate`.

### Redirect signing results

In redirect mode with `requireAuth: true`, `signResult` is populated after returning from the dashboard signing redirect. Call `clearSignResult()` after consuming it.

### Embed approval UI

`<AbstraxionEmbed approvalView="modal" | "inline" />` controls how the iframe appears when a `requireAuth` signing request is pending. Context exposes `isAwaitingApproval: boolean` while approval is in flight.

## Grant configuration at connect

`AbstraxionConfig` controls what permissions the session key receives during connect.

### Preferred: `treasury`

```ts
const config = {
  chainId: "xion-testnet-2",
  treasury: "xion1...", // treasury contract address
};
```

When `treasury` is set, the dashboard queries that contract (DaoDao indexer when available) for grant configs to display and approve. This is the **modern, preferred** approach.

Optional: `treasuryIndexer: { url: "https://daodaoindexer.burnt.com" }` on signer-mode config.

### Legacy grant fields

Prefer `treasury` for new work. Legacy alternatives on the same config object:

| Field | Purpose |
|-------|---------|
| `contracts` | Authz for specific contract addresses |
| `stake` | Staking and governance grants |
| `bank` | Bank send limits (`{ denom, amount }[]`) |

### No-grants path

If `treasury`, `contracts`, `stake`, and `bank` are **all omitted**, connect shows authentication only — no on-chain grant approval step. The user receives a session key with **no delegated permissions**.

Valid when the dApp uses **`requireAuth: true`** for all transactions (direct meta-account signing). See `AbstraxionConfig.treasury` JSDoc in `types.ts`.

### `feeGranter`

Optional override for the address that sponsors grant-creation fees. Defaults per chain in `@burnt-labs/constants` — see [networks.md](./networks.md).

## Treasury: SDK config vs CLI

| Layer | Responsibility |
|-------|----------------|
| **SDK (`treasury` in config)** | Contract address the dApp passes to Abstraxion so the dashboard knows which grant configs to offer at connect |
| **CLI (`verona-treasury` skill)** | Create treasury, fund, withdraw, configure grant/fee policies, admin operations |

**Bridge workflow:**

1. Use **`verona-treasury`** to create and configure a treasury contract (grant configs, fee sponsorship, funding).
2. Copy the deployed treasury **contract address** into your app's `AbstraxionProvider` config as `treasury: "xion1..."`.
3. At connect, Abstraxion reads grant configs from that contract and issues a session key with matching permissions.

Do not inline `verona-treasury` CLI commands here — load the `verona-treasury` skill for toolkit execution. Prerequisites: `verona-toolkit-init`, `verona-oauth2`.

For contract deployment shortcuts (UserMap + treasury), the upstream quickstart at [quickstart.dev.testnet.burnt.com](https://quickstart.dev.testnet.burnt.com) can emit env vars for Abstraxion integration.

## Decision guide

```
Need gasless UX for routine dApp actions?
├─ Yes → treasury in config + useAbstraxionSigningClient() (default)
└─ No (or high-value / admin txs only) → omit grants or use requireAuth: true

How should users sign in?
├─ Standard web dApp → authentication: { type: "auto" }
├─ Inline dashboard UI → embedded + <AbstraxionEmbed>
├─ Mobile web / PWA → auto (redirect fallback) or redirect
└─ External wallet / custodial signer → type: "signer"
```

## Related sections

- [xion-js-web.md](./xion-js-web.md) — provider setup and package choice
- [networks.md](./networks.md) — chain IDs, dashboard URLs, fee granter defaults
- [integration-paths.md](./integration-paths.md) — when not to use Abstraxion (mob, oauth2-api-service)

## Source references

- `xion.js/packages/abstraxion-js/src/types.ts` — `AuthenticationConfig`, `AbstraxionConfig`, `SigningClient`
- `xion.js/packages/abstraxion-react/src/hooks/useAbstraxionSigningClient.ts` — `requireAuth` behavior
- `xion.js/demos/react` — `SessionKeySendCard` (session key), `DirectSigningPanel` (`requireAuth: true`)
- `xion.js/packages/constants/src/index.ts` — per-chain dashboard and fee-granter defaults
