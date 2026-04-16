# codex-multi Phase 1: Fix Account Switching

## Problem

`cm use <name>` symlinks the entire `~/.codex` directory, swapping sessions, plugins, hooks, config, and logs. Only `auth.json` should swap.

## New Architecture

```
~/.codex/                              (real directory — never a symlink)
  ├── auth.json → ~/.codex-multi/accounts/<active>/auth.json  (symlink)
  ├── config.toml                      ← global shared (or merged w/ per-account)
  ├── sessions/                        ← persistent across switches
  ├── plugins/                         ← persistent
  ├── hooks/                           ← persistent
  └── ...all other state               ← persistent

~/.codex-multi/
  ├── accounts/
  │   ├── <name>/
  │   │   ├── auth.json                ← OAuth tokens only
  │   │   └── config.toml              ← optional per-account overrides
  │   └── ...
  ├── default                          ← active account name
  ├── config                           ← codex-multi settings (lb_url, etc.)
  └── global-config.toml.bak           ← backup of unmerged global config
```

### Config merge behavior

On `cm use <name>`:
1. Symlink `~/.codex/auth.json` → account's `auth.json`
2. Restore `~/.codex/config.toml` from `global-config.toml.bak`
3. If `accounts/<name>/config.toml` exists, merge it on top (Python inline TOML merge)
4. Write `default` file

On `cm use` (switch away): global config is always restorable from `.bak`.

## Tasks

### Task 1: Rewrite `cmd_use()`
**File:** `codex-multi` lines 296-340
**Changes:**
- Remove: `rm -f "$codex_home" && ln -s "$ACCOUNTS_DIR/$name" "$codex_home"`
- Add: ensure `~/.codex` is a real directory (not a symlink)
- Add: `ln -sf "$ACCOUNTS_DIR/$name/auth.json" "$codex_home/auth.json"`
- Add: config.toml merge logic (restore global, overlay per-account if present)
- Add: save global config.toml backup before first merge
- Keep: `echo "$name" > "$DEFAULT_FILE"`

### Task 2: Auto-migration with confirmation
**Trigger:** `cmd_use()` detects `~/.codex` is a symlink pointing to `accounts/`
**Flow:**
1. Detect: `[ -L "$codex_home" ]` and target is under `$ACCOUNTS_DIR`
2. Show explanation of what will change
3. Prompt `[y/N]`
4. Migrate:
   - Resolve current symlink target to get current account data
   - `rm ~/.codex` (the symlink)
   - Copy resolved directory contents to `~/.codex/` (real dir)
   - For each account in `accounts/`, keep only `auth.json` (and optional `config.toml`)
   - Move other files (sessions, plugins, etc.) that were duplicated
   - Symlink `~/.codex/auth.json` → active account's `auth.json`

### Task 3: Fix `cmd_add()`
**File:** `codex-multi` lines 144-294
**Changes:**
- After OAuth callback captures tokens, save only `auth.json` to `$ACCOUNTS_DIR/$name/auth.json`
- Do NOT copy entire `~/.codex` state into the account directory
- Remove `config.toml` creation in account dir (user adds it manually if needed)

### Task 4: Fix import/export
**File:** `codex-multi` lines 422-460
**Changes:**
- `cmd_import()`: copy file to `$ACCOUNTS_DIR/$name/auth.json` only
- `cmd_export()`: cat `$ACCOUNTS_DIR/$name/auth.json` only
- Remove any logic that copies config.toml or other state

### Task 5: Fix backup/restore
**File:** `codex-multi` lines 463-534
**Changes:**
- `cmd_backup()`: archive `~/.codex-multi/` (accounts with auth.json only + config + default)
- `cmd_restore()`: detect old vs new format, restore accordingly
- New format backups are smaller (no duplicated sessions/plugins)

### Task 6: Verify `cmd_run()`
**File:** `codex-multi` line 665
**Current:** `CODEX_HOME="$ACCOUNTS_DIR/$name" codex "$@"` — already correct for auth
**Changes:**
- Verify it works with new account structure (account dir has only auth.json)
- May need: create temp dir with symlinked auth.json + merged config if per-account config exists
- Or: set env var approach may need adjustment since CODEX_HOME expects full dir structure

### Task 7: Update doctor
**File:** `codex-multi` lines 538-633
**Changes:**
- Check `~/.codex` is a real directory (not a symlink) — warn if old layout
- Check `~/.codex/auth.json` is a symlink to an account's auth.json
- Check each account dir contains `auth.json`
- Check `default` file matches current auth.json symlink target
- Check `global-config.toml.bak` exists

### Task 8: Add quota/usage monitor
**New command:** `cm quota` or `cm usage`
**Behavior:**
- For each account: decode JWT from auth.json, call OpenAI usage API
- Display table: account name, plan type, usage %, remaining quota
- Highlight accounts near quota limits
- If codex-lb is configured, also show lb-reported quotas

### Task 9: Update README
- Document new architecture
- Document migration (automatic on first `cm use`)
- Document per-account config.toml overrides
- Document `cm quota` command
- Update architecture diagram

## Future Phases

### Phase 2: Python Rewrite
Rewrite in Python (click/typer, PyJWT, httpx, pytest, pyproject.toml).

### Phase 3: Web UI
Localhost dashboard (FastAPI + frontend) for account management.
