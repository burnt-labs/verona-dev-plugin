# Verona Dev Plugin

Multi-host agent skills plugin for [Verona](https://docs.burnt.com/xion/verona) and Xion development from Burnt Labs.

Install once per coding agent host (Cursor, Codex, Claude Code, Kimi) using each host's official plugin or path install flow — **no custom install CLI**.

## What you get

| Component | Location | Status |
|-----------|----------|--------|
| Host manifests | `.cursor-plugin/`, `.codex-plugin/`, `.claude-plugin/`, `.kimi-plugin/` | Available |
| Shared skills | `skills/` | Available (vdp-002 corpus) |
| Session entry | `verona-dev` skill + `hooks/` | Auto on Cursor, Kimi, Claude; manual on Codex |

Each host discovers skills from the shared `skills/` tree per its manifest conventions.

## Quick start

1. Clone this repository (or install from a published path — see [INSTALL.md](INSTALL.md)).
2. Follow the install section for your host in [INSTALL.md](INSTALL.md).
3. Reload or restart the host after install.
4. Start Verona work — **`verona-dev`** loads automatically on Cursor, Kimi, and Claude Code; on Codex, invoke **`verona-dev`** manually at session start.

## Supported hosts

- **Cursor** — local plugin path or marketplace
- **Codex** — plugin marketplace or local path
- **Claude Code** — plugin marketplace or local path
- **Kimi** — `/plugins install` or path install

Per-host steps: [INSTALL.md](INSTALL.md).

## Repository layout

```
verona-dev-plugin/
├── .cursor-plugin/plugin.json
├── .codex-plugin/plugin.json
├── .claude-plugin/plugin.json
├── .kimi-plugin/plugin.json
├── skills/                 # Shared skill corpus
├── hooks/                  # Session hooks (Cursor, Claude)
├── README.md
├── INSTALL.md
├── AGENTS.md
└── LICENSE
```

## Roadmap

- **Frontend / stack selection** (React, Next, Vue, dApp wiring) — planned for a future plugin release; `verona-dev` routes those requests to a stub until then.

## License

Licensed under the [Apache License 2.0](LICENSE).
