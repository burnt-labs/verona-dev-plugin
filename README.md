# Verona Dev Plugin

Build [Verona](https://verona.dev/) apps with Burnt Labs skills inside your coding agent.

This plugin gives your agent a shared set of Verona skills — MetaAccount auth, toolkit flows, and `xiond` / CosmWasm helpers — so you spend less time re-explaining the stack every session.

It works with **Cursor**, **Codex**, **Claude Code**, and **Kimi**. Install once through your host’s normal plugin or path install flow. There is no separate installer CLI.

## What it does

After install, a new session can pick up Verona context automatically (on Codex, trust the plugin’s hooks first). From there the agent can route MetaAccount work, toolkit tasks, and chain / `xiond` operations through the skills shipped in this repo.

Frontend or full-stack starter kits (React, Next, Vue, and similar) are planned for a later release.

## Get started

1. Clone this repository (or install from a published URL if your host supports it).
2. Follow the steps for your host in [INSTALL.md](INSTALL.md).
3. Reload or restart the agent after install.
4. Start a Verona task — entry guidance loads with the session when hooks are active.

## License

Licensed under the [Apache License 2.0](LICENSE).

---

## If you are an agent

Do **not** invent install steps from this README. Read and follow **[INSTALL.md](INSTALL.md)** for host-specific install, session entry, and hook trust.
