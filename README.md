# codex-multi

Wrapper for managing multiple [OpenAI Codex CLI](https://github.com/openai/codex) accounts. Switch between ChatGPT subscriptions without logging in/out.

## Install

```bash
git clone https://github.com/sgnjfk/codex-multi.git
cd codex-multi
sudo ln -sf $(pwd)/codex-multi /usr/local/bin/codex-multi
```

Add alias to `.zshrc` or `.bashrc`:
```bash
alias cm="codex-multi"
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

# Switch default (switches for both codex-multi AND bare codex)
cm use work

# Run codex with default account
codex exec "list files"
cm exec "list files"

# Run codex with specific account
cm personal exec "list files"

# Check login status
cm status
cm status work

# Export / import auth
cm export personal > backup.json
cm import restored backup.json

# Save codex-lb URL in ~/.codex-multi/lb_url
cm set lb 192.168.1.88:2455
cm get lb

# Back up / restore all accounts + config
cm backup /tmp/codex-multi.tar.gz
cm restore /tmp/codex-multi.tar.gz
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
cm backup /tmp/codex-multi.tar.gz
scp /tmp/codex-multi.tar.gz user@wsl-host:~/

# On WSL
cm restore ~/codex-multi.tar.gz
cm ls
```

If SSH on the target machine uses a custom port, pass it to `scp`:
```bash
scp -P 2222 /tmp/codex-multi.tar.gz user@wsl-host:~/
```

## How it works

Each account gets its own directory under `~/.codex-multi/accounts/<name>/`.

`cm use <name>` creates a symlink `~/.codex → ~/.codex-multi/accounts/<name>/`, so both `codex-multi` and bare `codex` use the same account. Original `~/.codex` is backed up to `~/.codex.bak` on first run.

`cm backup` archives the full `~/.codex-multi` state, including all accounts, `auth.json`, `config.toml`, and the default account marker. `cm restore` extracts that archive and recreates `~/.codex` for the restored default account when possible.

```
~/.codex-multi/
├── default
└── accounts/
    ├── personal/
    │   ├── auth.json
    │   └── config.toml
    └── work/
        ├── auth.json
        └── config.toml
~/.codex → ~/.codex-multi/accounts/personal/  (symlink)
```

## Config

Set `CODEX_MULTI_HOME` to change the base directory (default: `~/.codex-multi`).

`cm set lb <addr>` stores the codex-lb URL in `~/.codex-multi/lb_url`, so it persists across shells and machines when you back up / restore `codex-multi`.

Accepted forms:
```bash
cm set lb 192.168.1.88:2455
cm set lb pi.local:2455
cm set lb http://192.168.1.88:2455
cm set lb https://codex-lb.local:2455
```

## License

MIT
