# Install and configure `xiond`

Install, upgrade, and verify the `xiond` CLI for Verona chain work. See also `cli-rename-note.md` for the Verona branding vs `xiond` binary naming.

## When to use `xiond` vs `verona-toolkit`

| Scenario | Recommended tool |
|----------|------------------|
| MetaAccount / gasless development | `verona-toolkit` (`verona-toolkit-init` skill) |
| OAuth2 authentication | `verona-oauth2` skill |
| Contract deployment (CosmWasm) | `xiond` (this skill → `wasm.md`) |
| Chain queries, tx lookup | `xiond` (this skill → `usage.md`) |
| Validator / node operations | `xiond` |
| Mnemonic-based wallets | `xiond` |

## When to use init workflows

- First-time `xiond` install
- Check whether `xiond` is installed and which version
- Upgrade to the latest packaged release
- Fix `xiond: command not found`
- Set up a traditional Cosmos SDK CLI environment

## How it works

1. **Detect OS** — macOS, Debian, Red Hat, or Alpine Linux
2. **Check status** — installed version and binary path
3. **Install or upgrade** — OS-appropriate package manager
4. **Verify** — confirm with `xiond version`

## Compatibility

- Requires `bash` and `python3`
- May require `sudo` (Linux installs)
- Network access to Burnt package repositories

## Scripts (relative to skill root)

```bash
bash scripts/install.sh
bash scripts/check-version.sh
bash scripts/upgrade.sh
```

All scripts emit JSON on stdout and progress on stderr.

### Install output (already installed)

```json
{
  "success": true,
  "os": "macOS",
  "installed": true,
  "version": "xiond version 1.0.0",
  "message": "xiond is already installed"
}
```

### Check version (not installed)

```json
{
  "success": true,
  "installed": false,
  "version": null,
  "path": null,
  "message": "xiond is not installed"
}
```

## Manual installation by OS

### macOS

```bash
brew tap burnt-labs/xion
brew install xiond
xiond version
```

### Debian-based Linux (Ubuntu, Debian)

```bash
wget -qO - https://packages.burnt.com/apt/gpg.key | sudo gpg --dearmor -o /usr/share/keyrings/burnt-keyring.gpg
echo "deb [signed-by=/usr/share/keyrings/burnt-keyring.gpg] http://packages.burnt.com/apt /" | sudo tee /etc/apt/sources.list.d/burnt.list
sudo apt update
sudo apt install xiond
xiond version
```

### Red Hat-based Linux (CentOS, Fedora, RHEL)

```bash
sudo rpm --import https://packages.burnt.com/yum/gpg.key
printf "[burnt]\nname=Burnt Repo\nenabled=1\nbaseurl=https://packages.burnt.com/yum/\n" | sudo tee /etc/yum.repos.d/burnt.repo
sudo dnf install xiond   # or: sudo yum install xiond
xiond version
```

### Alpine Linux

```bash
wget -qO - https://alpine.fury.io/burnt/burnt@fury.io-b8abd990.rsa.pub | sudo tee /etc/apk/keys/burnt@fury.io-b8abd990.rsa.pub
echo "https://alpine.fury.io/burnt" | sudo tee -a /etc/apk/repositories
sudo apk update
sudo apk add xiond
xiond version
```

## Troubleshooting

**macOS**

- Install Homebrew from https://brew.sh if missing
- If tap fails: `brew tap burnt-labs/xion` manually

**Linux**

- GPG or repository errors: verify network and repository URLs
- Package not found: run `apt update`, `dnf check-update`, or `apk update` first

**General**

- Alternative installs (binaries, Docker, source): [xiond installation docs](https://docs.verona.dev/en/build-on-verona/computation/local-development/setting-up-env/installation-prerequisites-setup-local-environment)
- After install: `xiond --help`, `xiond keys add <keyname>`, `xiond init <moniker>`

## Next steps

- Accounts, transfers, queries → `usage.md`
- CosmWasm deploy and interact → `wasm.md`
