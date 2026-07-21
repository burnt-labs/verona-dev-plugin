# AGENTS.md

This repository is the Burnt Labs **Verona Dev Plugin** — a multi-host agent skills plugin for Verona / Xion development.

## Session entry

The primary entry skill is **`verona-dev`**. Use it for Verona / Xion development context.

| Host | Auto-entry (v0.1) |
|------|-------------------|
| **Cursor** | `hooks/hooks-cursor.json` → `hooks/session-start` |
| **Kimi** | `sessionStart.skill: "verona-dev"` in `.kimi-plugin/plugin.json` |
| **Claude Code** | `hooks/hooks.json` SessionStart → `hooks/session-start` |
| **Codex** | Manual — invoke or mention **`verona-dev`** at session start (no manifest hook) |

Shared skills live under `skills/`. Install per host: [INSTALL.md](INSTALL.md).

## Local agent workspace

`.mstar/` is a **local-only** agent workspace directory (gitignored). Do not commit it. Do not reference it from other project files outside this root `AGENTS.md`.
