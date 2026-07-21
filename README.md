# Verona Dev Plugin

Multi-host agent skills plugin for [Verona](https://docs.burnt.com/xion/verona) and Xion development from Burnt Labs.

Install once per coding agent host (Cursor, Codex, Claude Code, Kimi) using each host's official plugin or path install flow — **no custom install CLI**.

## What you get

| Component | Location | Status |
|-----------|----------|--------|
| Host manifests | `.cursor-plugin/`, `.codex-plugin/`, `.claude-plugin/`, `.kimi-plugin/` | Available |
| Shared skills | `skills/` | Populated in a follow-up release (vdp-002) |
| Session entry | `verona-dev` skill | Manual invoke in v0.1; auto session hooks in a follow-up update |

Each host discovers skills from the shared `skills/` tree per its manifest conventions.

## Quick start

1. Clone this repository (or install from a published path — see [INSTALL.md](INSTALL.md)).
2. Follow the install section for your host in [INSTALL.md](INSTALL.md).
3. Reload or restart the host after install.
4. At session start, invoke the **`verona-dev`** skill manually (automatic entry once session hooks ship in a follow-up update).

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
├── skills/                 # Shared skill corpus (vdp-002)
├── hooks/                  # Session hooks (follow-up update)
├── README.md
├── INSTALL.md
├── AGENTS.md
└── LICENSE
```

## License

Licensed under the [Apache License 2.0](LICENSE).
