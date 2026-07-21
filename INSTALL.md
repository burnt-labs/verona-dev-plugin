# Verona Dev Plugin — Installation

Manual install only. There is **no** install CLI, `npx` package, or custom installer script for this plugin.

## Prerequisites

- A supported host: [Cursor](https://cursor.com), [Codex](https://github.com/openai/codex), Claude Code, or [Kimi Code CLI](https://www.kimi.com/code/docs/kimi-code-cli/)
- **Git** to clone this repository (unless your host installs directly from a URL)

## Get the plugin source

Clone to a stable location on your machine:

```bash
git clone https://github.com/burnt-labs/verona-dev-plugin.git ~/verona-dev-plugin
```

Use any directory you prefer; replace `~/verona-dev-plugin` below with your checkout path.

## What you get

| Component | Location | Status |
|-----------|----------|--------|
| Host manifests | `.cursor-plugin/`, `.codex-plugin/`, `.claude-plugin/`, `.kimi-plugin/` | Available |
| Shared skills | `skills/` | Available (vdp-002 corpus) |
| Session entry | `verona-dev` skill + `hooks/` | Auto on Cursor, Codex, Claude, Kimi (Codex requires trusting plugin hooks) |

Each host discovers skills from the shared `skills/` tree per its manifest conventions.

---

## Cursor

**Install method:** local plugin path (recommended for development) or Cursor plugin marketplace when published.

### Local path (manual)

Cursor discovers plugins under `~/.cursor/plugins/local/<plugin-name>/`. Use a **real directory** (not a symlink to the clone root).

```bash
mkdir -p ~/.cursor/plugins/local
git clone https://github.com/burnt-labs/verona-dev-plugin.git ~/.cursor/plugins/local/verona-dev-plugin
```

To update after `git pull` in that directory, restart Cursor or run **Developer: Reload Window**.

### Marketplace

When listed in the Cursor plugin marketplace, search for **verona-dev-plugin** or use the host's `/add-plugin` flow per [Cursor plugin docs](https://cursor.com/docs/plugins).

### Session entry (v0.1)

| Mode | Behavior |
|------|----------|
| **Automatic** | `hooks/hooks-cursor.json` → `hooks/session-start` injects `verona-dev` context at session start |
| **Fallback** | Mention or invoke the `verona-dev` skill manually if hooks are disabled |

Manifest: `.cursor-plugin/plugin.json` (`skills`: `./skills/`, `hooks`: `./hooks/hooks-cursor.json`).

Verify hook script is executable: `test -x hooks/session-start`.

---

## Codex

**Install method:** Codex plugin marketplace or local path via personal marketplace config.

### Local path (manual)

1. Clone (if not already):

   ```bash
   git clone https://github.com/burnt-labs/verona-dev-plugin.git ~/verona-dev-plugin
   ```

2. Register in your personal marketplace (`~/.agents/plugins/marketplace.json` or equivalent per [Codex plugin docs](https://github.com/openai/codex)):

   ```json
   {
     "name": "personal",
     "interface": {
       "displayName": "Personal"
     },
     "plugins": [
       {
         "name": "verona-dev-plugin",
         "source": {
           "source": "local",
           "path": "/absolute/path/to/verona-dev-plugin"
         },
         "policy": {
           "installation": "AVAILABLE",
           "authentication": "ON_INSTALL"
         }
       }
     ]
   }
   ```

   Replace `/absolute/path/to/verona-dev-plugin` with your checkout (e.g. `/Users/you/verona-dev-plugin`).

3. Install the plugin:

   ```bash
   codex plugin add verona-dev-plugin --marketplace personal
   ```

### Marketplace

When published to an official or community Codex marketplace, use `codex plugin add verona-dev-plugin` per host documentation.

### Session entry (v0.1)

**Automatic via hooks** after you trust the plugin's hooks (Codex requires explicit trust before plugin hooks run — use `/hooks` or the trust prompt).

| Mode | Behavior |
|------|----------|
| Trusted hooks | `hooks/hooks.json` SessionStart → `hooks/session-start` injects `verona-dev` entry context (`additionalContext`) |
| Fallback | Mention or invoke the **`verona-dev`** skill manually |

Manifest: `.codex-plugin/plugin.json` (`skills`: `./skills/`, `hooks`: `./hooks/hooks.json`).

Docs: [Build Codex plugins](https://learn.chatgpt.com/docs/build-plugins) (hooks field) · [Codex hooks](https://developers.openai.com/codex/hooks) (SessionStart schema).

---

## Kimi

**Install method:** Kimi TUI `/plugins install` (URL or path) per [Kimi Code CLI plugin docs](https://www.kimi.com/code/docs/kimi-code-cli/).

### From GitHub URL

In Kimi TUI:

```text
/plugins install https://github.com/burnt-labs/verona-dev-plugin
/plugins reload
```

### From local path

If your Kimi build supports path install, point at your clone root (the directory containing `.kimi-plugin/plugin.json`).

### Session entry (v0.1)

| Mode | Behavior |
|------|----------|
| **Automatic** | `sessionStart.skill: "verona-dev"` in `.kimi-plugin/plugin.json` loads the entry skill natively |
| **Fallback** | Invoke **`verona-dev`** manually if auto-load is disabled |

Manifest: `.kimi-plugin/plugin.json` (`skills`: `./skills/`, `sessionStart.skill`: `verona-dev`).

---

## Claude Code

**Install method:** Claude plugin marketplace or local plugin path per [Claude Code plugin docs](https://docs.anthropic.com/en/docs/claude-code/plugins).

### Marketplace

When published:

```text
/plugin marketplace add burnt-labs/verona-dev-plugin
/plugin install verona-dev-plugin@<marketplace>
```

Adjust marketplace slug per Anthropic's published listing.

### Local path

Clone this repository and register the plugin root (directory containing `.claude-plugin/plugin.json`) through Claude Code's local plugin install flow.

Claude discovers `skills/` by convention (sibling to `.claude-plugin/`).

### Session entry (v0.1)

| Mode | Behavior |
|------|----------|
| **Automatic** | `hooks/hooks.json` SessionStart → `hooks/session-start` when Claude Code loads plugin hooks |
| **Fallback** | Invoke **`/verona-dev`** (or mention the skill) at session start if hooks are disabled |

Manifest: `.claude-plugin/plugin.json`. Hooks: `hooks/hooks.json` (discovered by convention).

---

## After install

1. Reload the host (Cursor reload window, Codex restart, Kimi `/plugins reload`, Claude session refresh).
2. Confirm the plugin name **`verona-dev-plugin`** appears in the host's plugin list.
3. On **Cursor**, **Kimi**, **Claude Code**, and **Codex**, session entry for **`verona-dev`** loads automatically when hooks/manifest are active. On **Codex**, trust plugin hooks first (`/hooks` or the trust prompt).

## Updating

Pull the latest in your checkout, then reload the host:

```bash
cd ~/verona-dev-plugin && git pull
```

For Cursor local install, run `git pull` inside `~/.cursor/plugins/local/verona-dev-plugin`.

## Further reading

- Plugin overview: [README.md](README.md)
- Agent workspace note (maintainers): [AGENTS.md](AGENTS.md)
