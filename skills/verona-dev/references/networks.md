# Networks and endpoints

Canonical network values for Verona / XION application integration. **Source of truth:** `@burnt-labs/constants` (`xion.js/packages/constants`).

This guide documents **two production networks only**:

| Network | Chain ID |
|---------|----------|
| Testnet | `xion-testnet-2` |
| Mainnet | `xion-mainnet-1` |

Do not target legacy `xion-testnet-1` in new integrations. Local dev (`xion-local-testnet-1`) exists in constants for `xiond` workflows but is out of scope here — see `verona-bin` for local chain setup.

## Chain info

Import pre-built `ChainInfo` objects or resolve by chain ID:

```ts
import {
  testnetChainInfo,
  mainnetChainInfo,
  getChainInfo,
  getRpcUrl,
  getRestUrl,
} from "@burnt-labs/constants";

const chainId = "xion-testnet-2";
const info = getChainInfo(chainId); // testnetChainInfo | mainnetChainInfo | undefined
```

### Endpoints

| Chain ID | RPC | REST |
|----------|-----|------|
| `xion-testnet-2` | `https://rpc.xion-testnet-2.burnt.com:443` | `https://api.xion-testnet-2.burnt.com:443` |
| `xion-mainnet-1` | `https://rpc.xion-mainnet-1.burnt.com:443` | `https://api.xion-mainnet-1.burnt.com:443` |

`normalizeAbstraxionConfig()` in `@burnt-labs/abstraxion-js` fills `rpcUrl`, `restUrl`, and `gasPrice` from these values when you pass only `chainId`. Override RPC/REST only for custom infrastructure.

### Gas and denom

| Field | Value |
|-------|-------|
| Native denom (display) | `XION` |
| Minimal denom | `uxion` |
| Decimals | `6` |
| Default gas price | `0.001uxion` (`xionGasValues.gasPrice`) |
| Gas adjustment | `1.4` (`xionGasValues.gasAdjustment`) |

### Bech32 prefix

All supported networks use the `xion` account prefix (`bech32PrefixAccAddr: "xion"`).

## Auth and dashboard URLs

Chain-specific auth hosts are resolved from constants when you omit overrides in `AbstraxionConfig.authentication`:

| Chain ID | Dashboard / auth app (`getIframeUrl`) | Notes |
|----------|---------------------------------------|-------|
| `xion-testnet-2` | `https://auth.testnet.burnt.com` | Testnet dashboard iframe and OAuth UI |
| `xion-mainnet-1` | `https://settings.burnt.com` | Mainnet dashboard |

`fetchConfig(rpcUrl)` queries `/status` on the RPC node, reads `node_info.network`, and returns `{ dashboardUrl, restUrl, networkId, feeGranter }` for supported chain IDs.

## Fee granter (session-key gas sponsorship)

Default fee granter addresses used during grant creation (filled automatically when not overridden):

| Chain ID | Fee granter |
|----------|-------------|
| `xion-testnet-2` | `xion1xrqz2wpt4rw8rtdvrc4n4yn5h54jm0nn4evn2x` |
| `xion-mainnet-1` | `xion12q9q752mta5fvwjj2uevqpuku9y60j33j9rll0` |

Pass `feeGranter` in `AbstraxionConfig` only when you need a non-default sponsor.

## Treasury indexer

DaoDao treasury queries default to:

```
https://daodaoindexer.burnt.com
```

Configure explicitly via `treasuryIndexer: { url }` on signer-mode config, or rely on defaults from `getDaoDaoIndexerUrl(chainId)`.

## OAuth2 API hosts (toolkit / headless)

For CLI and headless HTTP integration (not Abstraxion SDK auth):

| Network | OAuth2 API host |
|---------|-----------------|
| Testnet | `oauth2.testnet.burnt.com` |
| Mainnet | `oauth2.burnt.com` |

Abstraxion dashboard auth uses the dashboard URLs above, not these OAuth hosts. See [oauth2-api-service.md](./oauth2-api-service.md) for the parallel REST path.

## Minimal Abstraxion config

Only `chainId` is required; endpoints and gas price are normalized from constants:

```ts
const config = {
  chainId: "xion-testnet-2",
  treasury: "xion1...", // your treasury contract — see auth-and-signing.md
};
```

## Related sections

- Web SDK setup: [xion-js-web.md](./xion-js-web.md)
- Auth modes and treasury SDK config: [auth-and-signing.md](./auth-and-signing.md)
- Path selection: [integration-paths.md](./integration-paths.md)

## Source references

Values above are taken from `xion.js/packages/constants/src/index.ts` (`testnetChainInfo`, `mainnetChainInfo`, `xionGasValues`, `DASHBOARD_URLS`, `FEE_GRANTERS`, `getChainInfo`, `fetchConfig`).
