# AGENTS.md

This repository is the Burnt Labs **Verona Dev Plugin** — a multi-host agent skills plugin for Verona development.

## Session entry

The primary entry skill is **`verona-dev`**. Use it for Verona development context.

| Host | Session entry (v0.1) |
|------|----------------------|
| **Cursor** | `hooks/hooks-cursor.json` → `hooks/session-start` |
| **Kimi** | `sessionStart.skill: "verona-dev"` in `.kimi-plugin/plugin.json` |
| **Claude Code** | `hooks/hooks.json` SessionStart → `hooks/session-start` |
| **Codex** | `hooks/hooks.json` SessionStart → `hooks/session-start` (trust plugin hooks first) |
| **omp** | Manual: `/skill:verona-dev` (no session-start hooks; discovery via `package.json#omp` + conventional `skills/`) |

Shared skills live under `skills/`. Brand SVGs live under `assets/`. Install per host: [INSTALL.md](INSTALL.md).

## Local agent workspace

`.mstar/` is a **local-only** agent workspace directory (gitignored). Do not commit it. Do not reference it from other project files outside this root `AGENTS.md`.
