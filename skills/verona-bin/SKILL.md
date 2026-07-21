---
name: verona-bin
description: |
  Verona chain CLI via `xiond`: install and configure the daemon, run mnemonic wallet
  accounts/transactions/queries, and deploy or interact with CosmWasm contracts.

  Use for xiond install, xiond upgrade, validator setup, chain query, tx status,
  balance query, mnemonic wallet, cosmwasm, contract deploy, wasm upload/instantiate,
  execute contract, migrate contract, and traditional Cosmos SDK workflows on Verona.

  The skill is Verona-branded; the installed binary remains `xiond` (see
  references/cli-rename-note.md). For MetaAccount gasless flows use verona-toolkit skills instead.
metadata:
  author: burnt-labs
  version: "1.0.0"
  provides:
    - xiond install, upgrade, and version checks
    - Chain queries, accounts, and token transfers
    - CosmWasm optimize, upload, instantiate, execute, migrate
  recommends:
    - verona-toolkit-init
    - verona-treasury
  compatibility: macOS, Linux (Debian/Red Hat/Alpine); requires bash, python3, jq for wasm flows
---

# verona-bin

Single entry for Verona **`xiond`** CLI work: installation, chain operations, and CosmWasm. Read the reference that matches the user intent; do not load all references up front.

## Route by intent

| User need | Read |
|-----------|------|
| Install, upgrade, version check | `references/init.md` |
| Accounts, send, balance/tx/chain query | `references/usage.md` |
| CosmWasm optimize → migrate | `references/wasm.md` |
| Why skill says Verona but commands use `xiond` | `references/cli-rename-note.md` |

## Install scripts

Run from this skill directory (paths relative to skill root):

```bash
bash scripts/install.sh
bash scripts/check-version.sh
bash scripts/upgrade.sh
```

Scripts output JSON on stdout. Requires `bash` and `python3`.

## Quick routing

**`xiond` missing** → `references/init.md` → `scripts/install.sh`

**Wallet / query / transfer** → `references/usage.md` (commands invoke `xiond` directly)

**Contract deploy or interact** → `references/wasm.md` (Docker + `xiond tx wasm` / `xiond query wasm`)

## vs verona-toolkit

| Use `xiond` (this skill) | Use verona-toolkit |
|--------------------------|-------------------|
| CosmWasm deploy / migrate | MetaAccount / OAuth2 |
| Chain & tx queries | Gasless transactions |
| Mnemonic wallets | Treasury operations |
| Validator / node CLI | Application SDK flows |

## Triggers

xiond, xiond install, xiond upgrade, validator, mnemonic wallet, chain query, tx status,
balance query, cosmwasm, contract deploy, wasm upload, instantiate, execute contract,
migrate contract, cosmos CLI, smart contract, Code ID.

## Dependencies

- `bash`, `python3` — install scripts
- `jq` — parsing wasm tx output (see `wasm.md`)
- `docker` — CosmWasm optimization
- `sudo` — Linux package installs
