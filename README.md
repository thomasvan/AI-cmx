# cmx

> **Note:** This project was originally forked from [cmx](https://github.com/sgnjfk/cmx) by sgnjfk. `cmx` is a continuation with additional features including session persistence, Python rewrite, and web UI.

Wrapper for managing multiple [OpenAI Codex CLI](https://github.com/openai/codex) accounts. Switch between ChatGPT subscriptions without logging in/out.

## Install

```bash
git clone https://github.com/thomasvan/AI-cmx.git
cd AI-cmx
sudo ln -sf $(pwd)/cmx /usr/local/bin/cmx
```

Add alias to `.zshrc` or `.bashrc`:
```bash
alias cm="cmx"
```

Requires: `codex` CLI, `python3`, `bash`, `curl`

## Usage

```bash
# Add accounts (works over SSH — prompts for callback URL)
cm add personal
cm add work

# List accounts (* = default)
cm ls
# * personal (you@gmail.com)
#   work (you@company.com)

# Switch default (symlinks only auth.json — sessions/plugins/hooks persist)
cm use work

# Run codex with default account
codex exec "list files"
cm exec "list files"

# Run codex with specific account
cm personal exec "list files"

# Check login status
cm status
cm status work

# Check quota/usage across all accounts
cm quota

# Export / import auth
cm export personal > backup.json
cm import restored backup.json

# Save codex-lb URL in ~/.cmx/lb_url
cm set lb 192.168.1.88:2455
cm get lb

# Back up / restore all accounts + config
cm backup /tmp/cmx.tar.gz
cm restore /tmp/cmx.tar.gz
```

## Remote / Headless Login (SSH)

`cm add <name>` detects SSH and handles OAuth automatically:

1. `codex login` runs in background on the remote machine
2. OAuth URL is printed — open it in your local browser
3. After login, browser redirects to `http://127.0.0.1:1455/...` (fails locally)
4. Copy the full URL from the browser address bar
5. Paste it into the terminal — the script forwards the callback

Alternatively, login on a machine with a browser and import:
```bash
# On PC
CODEX_HOME=/tmp/myaccount codex login
scp /tmp/myaccount/auth.json user@remote:~/

# On remote
cm import myaccount ~/auth.json
```

To move everything to another machine (for example Pi -> WSL):
```bash
# On Pi
cm backup /tmp/cmx.tar.gz
scp /tmp/cmx.tar.gz user@wsl-host:~/

# On WSL
cm restore ~/cmx.tar.gz
cm ls
```

If SSH on the target machine uses a custom port, pass it to `scp`:
```bash
scp -P 2222 /tmp/cmx.tar.gz user@wsl-host:~/
```

## How it works

Each account stores only its `auth.json` (OAuth tokens) under `~/.cmx/accounts/<name>/`.

`cm use <name>` symlinks only `~/.codex/auth.json` → the account's `auth.json`. Everything else in `~/.codex` (sessions, plugins, hooks, config, logs) stays untouched. This means switching accounts preserves all your state.

If a per-account `config.toml` exists (e.g. `~/.cmx/accounts/work/config.toml`), it gets merged on top of the global `~/.codex/config.toml` when you switch to that account.

```
~/.codex/                                         (real directory)
  ├── auth.json → ~/.cmx/accounts/personal/auth.json  (symlink)
  ├── config.toml                                 (global, shared)
  ├── sessions/                                   (persistent)
  ├── plugins/                                    (persistent)
  └── hooks/                                      (persistent)

~/.cmx/
├── default
├── config
└── accounts/
    ├── personal/
    │   └── auth.json
    └── work/
        ├── auth.json
        └── config.toml  (optional per-account overrides)
```

**Migration:** If you're upgrading from the old layout (where `~/.codex` was a symlink to the entire account directory), run `cm use <name>` — it auto-detects and migrates with confirmation.

## Config

Set `CODEX_MULTI_HOME` to change the base directory (default: `~/.cmx`).

`cm set lb <addr>` stores the codex-lb URL in `~/.cmx/lb_url`, so it persists across shells and machines when you back up / restore `cmx`.

Accepted forms:
```bash
cm set lb 192.168.1.88:2455
cm set lb pi.local:2455
cm set lb http://192.168.1.88:2455
cm set lb https://codex-lb.local:2455
```

## Origin

This project is based on [cmx](https://github.com/sgnjfk/cmx) by [sgnjfk](https://github.com/sgnjfk). The original project provided the foundation for multi-account Codex CLI management. `cmx` extends it with:

- **Session persistence** — switching accounts preserves sessions, plugins, and config
- **Python rewrite** — structured CLI with proper testing
- **Web UI** — localhost dashboard for visual account management

## License

MIT
