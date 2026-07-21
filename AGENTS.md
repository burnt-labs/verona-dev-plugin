# AGENTS.md

This repository is the Burnt Labs **Verona Dev Plugin** — a multi-host agent skills plugin for Verona / Xion development.

## Session entry

The primary entry skill is **`verona-dev`**. Use it at session start for Verona / Xion development context. Host-specific auto-load (Cursor, Kimi, Claude hooks) arrives in a follow-up update; until then, invoke `verona-dev` manually on every host.

Shared skills live under `skills/` (populated in vdp-002). Install per host: [INSTALL.md](INSTALL.md).

## Local agent workspace

`.mstar/` is a **local-only** agent workspace directory (gitignored). Do not commit it. Do not reference it from other project files outside this root `AGENTS.md`.
