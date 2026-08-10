# Verona Dev Plugin

Build [Verona](https://verona.dev/) apps with Burnt Labs skills inside your coding agent.

This plugin gives your agent a shared set of Verona skills — MetaAccount auth, toolkit flows, and `xiond` / CosmWasm helpers — so you spend less time re-explaining the stack every session.

It works with **Cursor**, **Codex**, **Claude Code**, **Kimi**, and **omp (Oh My Pi)**. Install once through your host’s normal plugin or path install flow.

## What it does

After install, a new session can pick up Verona context automatically on hosts with session-start hooks — Cursor, Kimi, Claude Code, and Codex (on Codex, trust the plugin’s hooks first). On **omp**, entry is manual: invoke **`/skill:verona-dev`** at the start of a session. The entry skill is **`verona-dev`**, which routes work into two clear buckets: **application development** (SDK guides + chain CLI) and **toolkit operations** (MetaAccount CLI flows).

## Skills routing

All skills live under [`skills/`](skills/). Session entry: [`skills/verona-dev/SKILL.md`](skills/verona-dev/SKILL.md).

### 1. Development

Use when building or integrating apps (web, mobile, native, headless OAuth), or when you need chain / CosmWasm / `xiond` help.

| Skill | Role |
|-------|------|
| **`verona-dev`** | Entry router + **application guides** under [`skills/verona-dev/references/`](skills/verona-dev/references/) (prefer **xion.js** / Abstraxion; alternates **mob**, **oauth2-api-service**) |
| **`verona-bin`** | `xiond` install, chain queries/txs, CosmWasm — use when the task needs the binary, not the app SDK |

Start at [`skills/verona-dev/references/README.md`](skills/verona-dev/references/README.md) for stack selection; hand off to `verona-bin` only for `xiond` / wasm / mnemonic-wallet work.

### 2. Toolkit

Use when operating MetaAccount flows through the **verona-toolkit** CLI (install, login, treasury, OAuth clients, assets, faucet).

| Skill | Role |
|-------|------|
| **`verona-toolkit-init`** | Install / set up the toolkit CLI |
| **`verona-oauth2`** | Login / MetaAccount auth (CLI) |
| **`verona-oauth2-client`** | OAuth app / client CRUD (CLI) |
| **`verona-treasury`** | Treasury create / fund / grants (CLI) |
| **`verona-asset`** | NFT / asset operations (CLI) |
| **`verona-faucet`** | Testnet tokens |

Do not mix lanes: app SDK setup stays in `verona-dev` references; toolkit command workflows stay in the toolkit skills above.

## Get started

1. Clone this repository (or install from a published URL if your host supports it).
2. Follow the steps for your host in [INSTALL.md](INSTALL.md) — remove any legacy non-plugin `verona-*` / `xion-*` skills first ([Cleanup legacy skills](INSTALL.md#cleanup-legacy-skills)).
3. Reload or restart the agent after install.
4. Start a Verona task — entry guidance loads with the session when hooks are active.

## License

Licensed under the [Apache License 2.0](LICENSE).

---

## If you are an agent

Do **not** invent install steps from this README. Read and follow **[INSTALL.md](INSTALL.md)** for host-specific install, session entry, and hook trust.

For task routing: load **`verona-dev`** first, then follow **Development** (`references/` or `verona-bin`) vs **Toolkit** (leaf skills) as above.
