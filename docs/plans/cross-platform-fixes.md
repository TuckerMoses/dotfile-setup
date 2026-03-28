---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: Cross-Platform Fixes

## Goal
Fix hardcoded paths in `.zshrc` and improve cross-platform clipboard support in tmux so the dotfiles work reliably on both macOS and Linux.

## Context
- `.zshrc` contains a `brew --prefix` call that fails on Linux (no Homebrew)
- `.tmux.conf` already has platform-conditional clipboard — but the `zsh-vi-mode` sourcing does not
- The `.zshrc.local` pattern exists but isn't used for the platform-specific bits yet
- These are bugs, not features — they should be fixed in the main configs

## Architecture

Two targeted fixes:

1. **zsh-vi-mode sourcing** — guard the `brew --prefix` call so it only runs on macOS, and add a Linux fallback path
2. **Audit remaining hardcoded paths** — ensure no `/Users/...` or `/home/...` literals remain in tracked configs

## Implementation Steps

### 1. Fix zsh-vi-mode sourcing in `.zshrc`

Replace the current unconditional `brew --prefix` line:

```bash
source $(brew --prefix)/opt/zsh-vi-mode/share/zsh-vi-mode/zsh-vi-mode.plugin.zsh
```

With platform-aware sourcing:

```bash
# ── zsh-vi-mode (must be after oh-my-zsh, before fzf) ────────────────────────
if [[ "$(uname)" == "Darwin" ]] && command -v brew &>/dev/null; then
    source "$(brew --prefix)/opt/zsh-vi-mode/share/zsh-vi-mode/zsh-vi-mode.plugin.zsh"
elif [[ -f /usr/share/zsh-vi-mode/zsh-vi-mode.plugin.zsh ]]; then
    source /usr/share/zsh-vi-mode/zsh-vi-mode.plugin.zsh
elif [[ -f "$HOME/.zsh-vi-mode/zsh-vi-mode.plugin.zsh" ]]; then
    source "$HOME/.zsh-vi-mode/zsh-vi-mode.plugin.zsh"
fi
```

This tries: Homebrew (macOS) → system package (Debian/Ubuntu) → manual install (`$HOME`).

### 2. Guard the fzf callback for the same condition

The `zvm_after_init` callback only makes sense when zsh-vi-mode is loaded. Wrap it:

```bash
# ── fzf (fuzzy finder) ──────────────────────────────────────────────────────
# zsh-vi-mode overrides keybindings, so fzf must be re-sourced in its callback
if typeset -f zvm_after_init &>/dev/null || [[ -n "$ZVM_LOADED" ]]; then
    function zvm_after_init() {
        source <(fzf --zsh)
    }
else
    source <(fzf --zsh)
fi
```

If zsh-vi-mode didn't load (e.g., not installed), fzf should still initialize normally.

### 3. Add zsh-vi-mode to bootstrap.sh

Add installation for both platforms:

**macOS section** — add to the `brew install` line:
```bash
brew install zsh-vi-mode
```

**Linux section** — add after the package manager block:
```bash
# zsh-vi-mode (not in most package managers)
if [[ ! -d "$HOME/.zsh-vi-mode" ]]; then
    git clone https://github.com/jeffreytse/zsh-vi-mode.git "$HOME/.zsh-vi-mode"
fi
```

### 4. Audit all tracked files for hardcoded paths

Run through each config file and confirm no hardcoded user paths remain. The only allowed exceptions are auto-generated blocks (conda, nvm) which should live in `.zshrc.local`, not the tracked `.zshrc`.

Check these patterns:
```bash
grep -rn '/Users/' zsh/ tmux/ ghostty/ starship/
grep -rn '/home/' zsh/ tmux/ ghostty/ starship/
```

If any are found, replace with `$HOME` or move to `.zshrc.local`.

## Verification

1. **macOS test**: Source `.zshrc` — zsh-vi-mode loads via Homebrew, fzf keybindings work
2. **Linux test**: Source `.zshrc` on a system without Homebrew — zsh-vi-mode loads from fallback path (or is skipped gracefully), fzf still initializes
3. **No hardcoded paths**: `grep -rn '/Users/\|/home/' zsh/ tmux/ ghostty/ starship/` returns nothing

## Edge Cases
- **zsh-vi-mode not installed at all**: The `if` block silently skips it, fzf still loads normally
- **`brew --prefix` is slow (~200ms)**: Only called on macOS where Homebrew is expected
- **Linux package name varies**: The manual git clone fallback covers distros where the package isn't available

## Dependencies
- None — this is a fix for existing configs
