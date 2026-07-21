---
name: verona-dev
description: |
  Default entry router for ALL Verona development in this plugin. Use this skill whenever the user mentions Verona, MetaAccount, gasless auth, Treasury, OAuth2, xiond, CosmWasm, testnet faucet, NFT minting, or building on Burnt Labs — even if they do not say "verona-dev" or "toolkit".

  Routes MetaAccount and toolkit work to in-plugin skills; routes chain queries, txs, xiond install, and CosmWasm to verona-bin. Do not treat external xion-skills repos as the primary install path.

  Triggers on: verona, xion, MetaAccount, gasless, 无 gas, Treasury, OAuth2, xiond, wasm, CosmWasm, faucet, testnet tokens, NFT, mint, burnt labs, build on xion, xion dapp, xion 开发, session key, verona agent toolkit.
metadata:
  author: burnt-labs
  version: "2.0.0"
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

Session entry for Verona work in **verona-dev-plugin**. Read this skill first, then load the routed skill below.

## Philosophy

**MetaAccount-first.** Most app developers (~90%) use the Verona Agent Toolkit for gasless auth, treasury, assets, and faucet flows. Reserve **xiond** (`verona-bin`) for advanced chain queries, transaction submission, wallet mnemonics, and CosmWasm lifecycle.

## Routing table

| User intent | Route to | Examples |
|-------------|----------|----------|
| Toolkit install / env setup | `verona-toolkit-init` | "install verona toolkit", "setup dev environment" |
| Login / OAuth / MetaAccount auth | `verona-oauth2` | "login", "gasless auth", "MetaAccount" |
| OAuth app / client CRUD | `verona-oauth2-client` | "register oauth app", "client manager" |
| Treasury create / fund / withdraw | `verona-treasury` | "treasury", "fee grant", "authz grant" |
| NFT / asset operations | `verona-asset` | "mint NFT", "collection" |
| Testnet tokens | `verona-faucet` | "faucet", "testnet tokens" |
| `xiond` queries, txs, wallet keys | `verona-bin` → `references/usage.md` | "query balance", "send tx", "xiond" |
| `xiond` install / upgrade | `verona-bin` → `references/init.md` | "install xiond", "upgrade xiond" |
| CosmWasm deploy / interact | `verona-bin` → `references/wasm.md` | "deploy contract", "wasm" |
| Frontend / stack choice | Deferred (see below) | "React vs Next", "frontend setup" |

## Workflow

1. **Detect intent** — match the user message to the routing table.
2. **Confirm** — tell the user which skill you are loading and why.
3. **Hand off** — Read that skill's `SKILL.md` and follow it; do not re-implement its commands here.

## Decision matrix

| User needs | Skill | Tool |
|------------|-------|------|
| Login / authentication | `verona-oauth2` | verona-toolkit |
| OAuth client registration / CRUD | `verona-oauth2-client` | verona-toolkit |
| Treasury create, fund, withdraw, grants | `verona-treasury` | verona-toolkit |
| NFT collection / mint / royalties | `verona-asset` | verona-toolkit |
| Testnet tokens / faucet | `verona-faucet` | verona-toolkit |
| Install or upgrade xiond | `verona-bin` → `init.md` | xiond |
| Chain queries, txs, mnemonic wallet | `verona-bin` → `usage.md` | xiond |
| CosmWasm store / instantiate / migrate | `verona-bin` → `wasm.md` | xiond |

## Frontend and stack selection (next iteration)

Frontend framework choice (React, Next, Vue, Svelte, etc.), dApp wiring norms, and stack-selection playbooks are **planned for a future plugin release**. A project stub for frontend and stack selection is reserved for the next iteration — do not invent full frontend guides here. For now, route toolkit auth/treasury/asset skills and defer stack-specific setup.

## Networks

| Network | OAuth2 API | Chain ID |
|---------|------------|----------|
| testnet | oauth2.testnet.burnt.com | xion-testnet-2 |
| mainnet | oauth2.burnt.com | xion-mainnet-1 |

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
