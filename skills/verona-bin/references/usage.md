# Accounts, transactions, and chain queries

Use `xiond` for traditional Cosmos SDK wallet operations and read-only chain queries on Verona. Install `xiond` first (`init.md` / `scripts/install.sh`).

## Query strengths

| Query type | Typical command |
|------------|-----------------|
| Block / node status | `xiond status --node <rpc>` |
| Transaction by hash | `xiond query tx <txhash> --node <rpc>` |
| Balance | `xiond query bank balances <address> --node <rpc>` |
| Local keys | `xiond keys list` |

## When to use `xiond` vs `verona-toolkit`

| Scenario | Recommended tool |
|----------|------------------|
| Gasless transactions | `verona-toolkit` |
| Treasury operations | `verona-treasury` skill |
| Chain / tx / balance queries | `xiond` |
| Mnemonic wallet | `xiond` |

## Prerequisites

- `xiond` installed (`references/init.md`)
- Funded account for transactions (testnet faucet below)
- RPC connectivity

## Network configuration

| Network | Chain ID | RPC |
|---------|----------|-----|
| testnet (default) | `xion-testnet-2` | `https://rpc.xion-testnet-2.burnt.com:443` |
| mainnet | `xion-mainnet-1` | `https://rpc.xion-mainnet-1.burnt.com` |

**Testnet extras**

- Faucet: https://dev.testnet.burnt.com/faucet
- Explorer: https://explorer.burnt.com/xion-testnet
- REST API: `https://api.xion-testnet-2.burnt.com`

**Mainnet extras**

- Explorer: https://explorer.burnt.com/xion-mainnet
- REST API: `https://api.xion-mainnet-1.burnt.com`

Set defaults for examples:

```bash
export XION_NETWORK=testnet   # or mainnet
export NODE=https://rpc.xion-testnet-2.burnt.com:443
export CHAIN_ID=xion-testnet-2
```

Public endpoints: [Verona public endpoints](https://docs.verona.dev/en/build-on-verona/references/public-endpoints-and-resources)

## Account management

### Create key (mnemonic wallet)

```bash
xiond keys add my-wallet
```

Save the mnemonic securely; it cannot be recovered.

### Restore from mnemonic

```bash
xiond keys add recovered-wallet --recover
```

### Show key / address

```bash
xiond keys show my-wallet
xiond keys show my-wallet -a    # address only
```

### List keys

```bash
xiond keys list
```

## Send tokens

```bash
xiond tx bank send my-wallet xion1abc... 1000uxion \
  --chain-id xion-testnet-2 \
  --node https://rpc.xion-testnet-2.burnt.com:443 \
  --from my-wallet \
  --gas-prices 0.025uxion \
  --gas auto \
  --gas-adjustment 1.3 \
  -y
```

Record the returned `txhash` for status lookup.

## Queries

Queries need `--node` only (no `--chain-id`).

### Balance

```bash
xiond query bank balances xion1abc... \
  --node https://rpc.xion-testnet-2.burnt.com:443
```

### Transaction status

```bash
xiond query tx <txhash> \
  --node https://rpc.xion-testnet-2.burnt.com:443
```

### Chain status

```bash
xiond status --node https://rpc.xion-testnet-2.burnt.com:443
```

### Account info

```bash
xiond query auth account xion1abc... \
  --node https://rpc.xion-testnet-2.burnt.com:443
```

## Gas configuration

| Network | Suggested gas price | Adjustment |
|---------|---------------------|------------|
| testnet | `0.025uxion` | `1.3` |
| mainnet | check current conditions | `1.3` or higher |

Flags: `--gas auto`, `--gas-adjustment 1.3`, or `--gas 200000` for a fixed limit.

## Testnet funding

1. Web faucet: https://dev.testnet.burnt.com/faucet
2. Discord faucet bot (Verona Discord)

Mainnet: acquire XION through supported exchanges.

## Troubleshooting

**`xiond` not found**

- Run `bash scripts/install.sh` or see `init.md`

**Key not found**

- `xiond keys list` and verify key name spelling

**Transaction fails**

- Sufficient balance including gas
- Correct `chain-id` and RPC URL
- Query tx with hash for error code

**Query fails**

- Address format `xion1...`
- RPC reachable from your network

## Validator / node (advanced)

```bash
xiond init <moniker> --chain-id <chain-id>
xiond start
```

Full daemon setup: [Daemon CLI](https://docs.verona.dev/en/build-on-verona/tools/daemon-cli)

## Related

- CosmWasm lifecycle → `wasm.md`
- CLI rename policy → `cli-rename-note.md`
