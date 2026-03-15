# codex-multi

Wrapper for managing multiple [OpenAI Codex CLI](https://github.com/openai/codex) accounts. Switch between ChatGPT subscriptions without logging in/out.

## Install

```bash
# Clone
git clone https://github.com/sgnjfk/codex-multi.git
cd codex-multi

# Add to PATH
sudo ln -sf $(pwd)/codex-multi /usr/local/bin/codex-multi
# or
cp codex-multi ~/.local/bin/
```

Requires: `codex` CLI, `python3`, `bash`

## Usage

```bash
# Add accounts
codex-multi add personal    # opens browser for ChatGPT login
codex-multi add work        # login with another account

# List accounts (* = default)
codex-multi ls
# * personal (you@gmail.com)
#   work (you@company.com)

# Set default
codex-multi use work

# Run codex with default account
codex-multi exec "list files"

# Run codex with specific account
codex-multi personal exec "list files"

# Check status
codex-multi status
codex-multi status work
```

## How it works

Each account gets its own directory under `~/.codex-multi/accounts/<name>/`. The wrapper sets `CODEX_HOME` to the correct directory before running `codex`, so each account has isolated auth tokens and config.

```
~/.codex-multi/
├── default           # name of default account
└── accounts/
    ├── personal/
    │   ├── auth.json
    │   └── config.toml
    └── work/
        ├── auth.json
        └── config.toml
```

## Config

Set `CODEX_MULTI_HOME` to change the base directory (default: `~/.codex-multi`).

## Import existing account

If you already have Codex logged in:

```bash
mkdir -p ~/.codex-multi/accounts/main
cp ~/.codex/auth.json ~/.codex-multi/accounts/main/
cp ~/.codex/config.toml ~/.codex-multi/accounts/main/ 2>/dev/null
codex-multi use main
```

## License

MIT
