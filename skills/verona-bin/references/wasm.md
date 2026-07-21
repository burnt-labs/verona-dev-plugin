# CosmWasm deploy and interact

Deploy and manage CosmWasm smart contracts on Verona using **`xiond`**. Covers optimize → upload → instantiate → query/execute → migrate.

Install `xiond` first (`init.md`). For gasless contract ops after deploy, use `verona-treasury` / `verona-toolkit`.

## Lifecycle

1. **Optimize** — Docker CosmWasm optimizer
2. **Upload** — store WASM on chain (Code ID)
3. **Instantiate** — create contract instance
4. **Query / execute** — read or mutate state
5. **Migrate** — upgrade logic (requires admin)

## Prerequisites

- `xiond` installed and wallet funded
- Docker running (optimization)
- CosmWasm contract source (Rust / `Cargo.toml`)

## Network defaults

Use testnet unless deploying to mainnet:

```bash
export WALLET=my-wallet
export NODE=https://rpc.xion-testnet-2.burnt.com:443
export CHAIN_ID=xion-testnet-2
```

| Network | Chain ID | RPC |
|---------|----------|-----|
| testnet | `xion-testnet-2` | `https://rpc.xion-testnet-2.burnt.com:443` |
| mainnet | `xion-mainnet-1` | `https://rpc.xion-mainnet-1.burnt.com` |

## 1. Optimize contract

```bash
docker run --rm -v "$(pwd)":/code \
  --mount type=volume,source="$(basename "$(pwd)")_cache",target=/target \
  --mount type=volume,source=registry_cache,target=/usr/local/cargo/registry \
  cosmwasm/optimizer:0.16.1
```

Output: `./artifacts/{contract_name}.wasm`

## 2. Upload WASM

```bash
RES=$(xiond tx wasm store ./artifacts/cw_counter.wasm \
  --chain-id "$CHAIN_ID" \
  --node "$NODE" \
  --from "$WALLET" \
  --gas-adjustment 1.3 \
  --gas-prices 0.001uxion \
  --gas auto \
  -y --output json)

TXHASH=$(echo "$RES" | jq -r '.txhash')
```

Extract Code ID:

```bash
CODE_ID=$(xiond query tx "$TXHASH" --node "$NODE" --output json \
  | jq -r '.events[] | select(.type == "store_code") | .attributes[] | select(.key == "code_id") | .value')
```

List uploaded codes:

```bash
xiond query wasm list-code --node "$NODE" --output json
```

## 3. Instantiate

```bash
MSG='{"count": 1}'
INST=$(xiond tx wasm instantiate "$CODE_ID" "$MSG" \
  --from "$WALLET" \
  --label "cw-counter" \
  --admin "$WALLET" \
  --chain-id "$CHAIN_ID" \
  --node "$NODE" \
  --gas-prices 0.025uxion \
  --gas auto \
  --gas-adjustment 1.3 \
  -y --output json)

INST_TX=$(echo "$INST" | jq -r '.txhash')
CONTRACT=$(xiond query tx "$INST_TX" --node "$NODE" --output json \
  | jq -r '.events[] | select(.type == "instantiate") | .attributes[] | select(.key == "_contract_address") | .value')
```

Use `--no-admin` only if migration will never be needed.

## 4. Query contract

```bash
QUERY='{"get_count":{}}'
xiond query wasm contract-state smart "$CONTRACT" "$QUERY" \
  --node "$NODE" --output json
```

Contract metadata:

```bash
xiond query wasm contract "$CONTRACT" --node "$NODE" --output json
```

## 5. Execute contract

```bash
EXEC='{"increment":{}}'
xiond tx wasm execute "$CONTRACT" "$EXEC" \
  --from "$WALLET" \
  --chain-id "$CHAIN_ID" \
  --node "$NODE" \
  --gas-prices 0.025uxion \
  --gas auto \
  --gas-adjustment 1.3 \
  -y
```

## 6. Migrate contract

Requires contract admin and a newly uploaded Code ID:

```bash
NEW_CODE_ID=456
MIGRATE='{}'
xiond tx wasm migrate "$CONTRACT" "$NEW_CODE_ID" "$MIGRATE" \
  --from "$WALLET" \
  --chain-id "$CHAIN_ID" \
  --node "$NODE" \
  --gas-prices 0.025uxion \
  --gas auto \
  --gas-adjustment 1.3 \
  -y
```

Address and state are preserved; logic comes from the new code.

## Gas hints

| Step | Gas price | Notes |
|------|-----------|-------|
| Upload | `0.001uxion` | large WASM may need higher limit |
| Instantiate / execute | `0.025uxion` | `--gas auto --gas-adjustment 1.3` |

## Troubleshooting

**Optimization**

- `docker ps` — daemon running
- Valid CosmWasm project with `Cargo.toml`

**Upload / instantiate / execute**

- Wallet balance covers gas
- JSON messages match contract schema
- Unique `--label` per instance

**Query**

- Contract address `xion1...`
- Query JSON matches `QueryMsg` schema

**Migration**

- Admin set at instantiate (not `--no-admin`)
- New code uploaded first
- `MigrateMsg` matches new contract

## Example project

Counter reference: https://github.com/burnt-labs/cw-counter

Official guide: [Deploy a CosmWasm smart contract](https://docs.burnt.com/xion/developers/getting-started-advanced/your-first-contract/deploy-a-cosmwasm-smart-contract)

CosmWasm docs: https://docs.cosmwasm.com/

## Related

- Install CLI → `init.md`
- Wallet and queries → `usage.md`
- CLI rename policy → `cli-rename-note.md`
