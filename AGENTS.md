# Project: codex-multi

## Overview
Bash wrapper for managing multiple OpenAI Codex CLI accounts. Lets users switch between ChatGPT subscriptions without login/logout cycles. Optionally integrates with **codex-lb** (a load balancer) for quota-aware account selection — configured via `~/.codex-multi/config`.

## Tech Stack
- **Bash** — single script `codex-multi` (all logic lives here)
- **Python3** — inline snippets for JWT decoding, OpenAI API calls, JSON parsing
- **curl** — health checks, codex-lb API, OAuth callback forwarding
- **Dependencies:** `codex` CLI, `python3`, `bash`, `curl`

## Architecture
```
codex-multi              # main script (executable, symlinked to /usr/local/bin/)
scripts/                 # Windows helpers (WSL SSH port proxy)
docs/                    # setup guides (WSL SSH Task Scheduler)
.ai/                     # AI context files

~/.codex/                # real directory (NOT a symlink)
├── auth.json → .../<active>/auth.json  (symlink — only this swaps)
├── config.toml          # shared global config
├── sessions/            # persistent across account switches
├── plugins/             # persistent
└── ...                  # all other state persists

~/.codex-multi/          # runtime data (not in repo)
├── config               # optional config (lb_url=http://host:port)
├── default              # name of current default account
└── accounts/
    └── <name>/
        └── auth.json    # OAuth tokens only
```

Key design: `cm use <name>` symlinks only `~/.codex/auth.json` → account's auth.json. All other codex state (sessions, plugins, hooks, config) persists across switches.

## Commands
| Command | Description |
|---------|-------------|
| `add <name>` | OAuth login, save new account (SSH-aware) |
| `rm <name>` | Remove account |
| `ls` | List accounts (* = default) |
| `use <name>` | Set default (symlinks ~/.codex) |
| `balance` | Switch to account with most remaining quota |
| `doctor` | Health check: all accounts (+ codex-lb if configured) |
| `reauth [--lb] <name>` | Re-login, backs up old auth (--lb to import to codex-lb) |
| `import <name> <file>` | Import auth.json |
| `export <name>` | Print auth.json to stdout |
| `backup <file>` | Archive all accounts/config |
| `restore <file>` | Restore from archive |
| `status [name]` | Check login status |
| `quota` | Show quota/usage across all accounts |
| `<name> [args]` | Run codex with specific account |

## Build & Test
No build step — single bash script. To install:
```bash
sudo ln -sf $(pwd)/codex-multi /usr/local/bin/codex-multi
```
No test suite currently. Manual testing via `cm doctor` and `cm status`.

## Conventions
- Single-file architecture — all commands in one bash script
- Inline Python for anything requiring JSON/JWT/HTTP (no external Python files)
- Color output: `red()` for errors, `green()` for success, `dim()` for hints
- No hardcoded URLs — codex-lb integration via `~/.codex-multi/config`
- Commit messages in English, imperative mood
- `CODEX_HOME` env var controls which account `codex` uses

## Context Files
- `.ai/STATUS.md` — Current progress and next steps
- `.ai/DECISIONS.md` — Architecture decisions and rationale
- `.ai/GLOSSARY.md` — Domain-specific terms
