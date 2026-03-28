---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: Atuin Shell History

## Goal
Replace the default zsh `Ctrl+R` history search with [atuin](https://github.com/atuinsh/atuin) — a SQLite-backed shell history tool with fuzzy search, per-directory filtering, and command duration tracking.

## Context
- Currently using fzf's `Ctrl+R` for history search (configured via `zvm_after_init` callback)
- fzf and atuin serve different purposes: fzf is a general fuzzy finder, atuin is purpose-built for shell history
- Atuin provides per-directory history, duration stats, and optional encrypted sync — features fzf history can't match
- zsh-vi-mode overrides keybindings, so atuin must be initialized in the `zvm_after_init` callback alongside fzf

## Architecture

```
bootstrap.sh  →  installs atuin
zsh/.zshrc    →  initializes atuin in zvm_after_init callback
```

Atuin stores its data in `~/.local/share/atuin/` and config in `~/.config/atuin/config.toml`. The config file is optional — we'll create an `atuin` stow package only if we want to customize settings.

## Implementation Steps

### 1. Add atuin to bootstrap.sh

**macOS section:**
```bash
brew install atuin
```

**Linux section:**
```bash
if ! command -v atuin &>/dev/null; then
    echo "==> Installing atuin..."
    curl --proto '=https' --tlsv1.2 -LsSf https://setup.atuin.sh | sh
fi
```

### 2. Initialize atuin in `.zshrc`

Update the `zvm_after_init` callback to include atuin:

```bash
# ── fzf (fuzzy finder) & atuin (shell history) ──────────────────────────────
# zsh-vi-mode overrides keybindings, so these must be re-sourced in its callback
function zvm_after_init() {
    source <(fzf --zsh)
    eval "$(atuin init zsh)"
}
```

If zsh-vi-mode isn't loaded, add the fallback:

```bash
if typeset -f zvm_after_init &>/dev/null || [[ -n "$ZVM_LOADED" ]]; then
    function zvm_after_init() {
        source <(fzf --zsh)
        eval "$(atuin init zsh)"
    }
else
    source <(fzf --zsh)
    eval "$(atuin init zsh)"
fi
```

### 3. Create atuin config (optional stow package)

```bash
mkdir -p atuin/.config/atuin
```

Create `atuin/.config/atuin/config.toml`:

```toml
# ── Atuin shell history ──────────────────────────────────────────────────────

# Search mode: fuzzy (default), prefix, fulltext, or skim
search_mode = "fuzzy"

# Filter mode when pressing Ctrl+R: global, host, session, or directory
filter_mode = "global"

# Filter mode for up-arrow: directory-scoped (most useful)
filter_mode_shell_up_key_binding = "directory"

# Style: auto picks compact for small terminals, full otherwise
style = "auto"

# Show preview of full command
show_preview = true

# Inline height (0 = fullscreen overlay)
inline_height = 0

# Disable sync by default (enable manually with: atuin login)
sync_address = ""

# Store duration and exit code
store_failed = true
```

Add `atuin` to the stow loop in `bootstrap.sh`:

```bash
for pkg in tmux zsh ghostty starship git atuin; do
```

### 4. Import existing history

Add a post-install note in bootstrap.sh:

```bash
if command -v atuin &>/dev/null; then
    echo "==> Tip: Run 'atuin import auto' to import your existing shell history"
fi
```

## Verification

1. Run `./bootstrap.sh` — atuin is installed
2. Open a new shell — `Ctrl+R` opens atuin's fuzzy history search (not fzf's)
3. Run a few commands, then `Ctrl+R` — recent commands appear with duration and timestamps
4. `cd` into a project directory, press up arrow — shows only commands run in that directory
5. `atuin stats` shows command usage statistics
6. fzf `Ctrl+T` (file finder) still works separately from atuin

## Edge Cases
- **Atuin not installed**: The `eval "$(atuin init zsh)"` will fail silently if atuin isn't in PATH; guard with `command -v atuin &>/dev/null &&` if needed
- **Conflicts with fzf Ctrl+R**: Atuin takes over `Ctrl+R` by default. fzf's history widget is still available as a widget but won't be bound. This is intended — atuin's history search is superior.
- **zsh-vi-mode ordering**: Both fzf and atuin must be in `zvm_after_init` to avoid keybinding conflicts
- **Sync disabled by default**: Users can opt into cross-machine sync with `atuin login` — not enabled automatically for privacy

## Dependencies
- None beyond the atuin binary itself
- Optional: `atuin login` for cross-machine sync (requires account creation)
