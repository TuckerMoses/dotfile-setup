---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: fzf Enhancements

## Goal
Add Catppuccin Mocha colors, fzf-tab completion, tmux popup mode, and git helpers to the existing fzf setup for a more polished and consistent fuzzy-finding experience.

## Context
- fzf is already installed and configured in `.zshrc`
- zsh-vi-mode requires fzf to be sourced in a callback (`zvm_after_init`)
- Catppuccin Mocha is the standard theme but fzf currently uses default colors (a theme consistency gap)
- tmux is configured with `focus-events on`, tmux 3.3+ supports fzf `--tmux` popup mode
- The modern-cli-tools plan sets `FZF_DEFAULT_COMMAND` to use fd — this plan builds on that

## Architecture

All changes go into `zsh/.zshrc`. No new files or packages needed.

## Implementation Steps

### 1. Add Catppuccin Mocha colors to `FZF_DEFAULT_OPTS`

Add to `.zshrc` before the fzf sourcing section:

```bash
# ── fzf Catppuccin Mocha theme ───────────────────────────────────────────────
export FZF_DEFAULT_OPTS=" \
  --color=bg+:#313244,bg:#1e1e2e,spinner:#f5e0dc,hl:#f38ba8 \
  --color=fg:#cdd6f4,header:#f38ba8,info:#cba6f7,pointer:#f5e0dc \
  --color=marker:#b4befe,fg+:#cdd6f4,prompt:#cba6f7,hl+:#f38ba8 \
  --color=selected-bg:#45475a \
  --color=border:#585b70,label:#cdd6f4"
```

These are the official Catppuccin Mocha fzf colors from `catppuccin/fzf`.

### 2. Enable tmux popup mode

Append to `FZF_DEFAULT_OPTS` (only takes effect inside tmux):

```bash
# Use tmux popup when running inside tmux (requires tmux 3.3+)
if [[ -n "$TMUX" ]]; then
    export FZF_DEFAULT_OPTS="$FZF_DEFAULT_OPTS --tmux center,80%,60%"
fi
```

The `--tmux` flag is available in fzf 0.53+. It opens fzf in a centered tmux popup instead of inline. Falls back to inline gracefully on older fzf versions.

### 3. Install fzf-tab

**Add to bootstrap.sh** after the Oh My Zsh plugin section:

```bash
# fzf-tab (replaces zsh completion menu with fzf)
FZF_TAB_DIR="${ZSH_CUSTOM:-$HOME/.oh-my-zsh/custom}/plugins/fzf-tab"
[[ -d "$FZF_TAB_DIR" ]] || \
    git clone https://github.com/Aloxaf/fzf-tab "$FZF_TAB_DIR"
```

**Add to `.zshrc` plugins list:**

Change:
```bash
plugins=(git wd zsh-autosuggestions zsh-syntax-highlighting)
```
To:
```bash
plugins=(git wd fzf-tab zsh-autosuggestions zsh-syntax-highlighting)
```

`fzf-tab` must come before `zsh-autosuggestions` and `zsh-syntax-highlighting` in the plugin list.

### 4. Configure fzf-tab previews

Add after the fzf section in `.zshrc`:

```bash
# ── fzf-tab configuration ───────────────────────────────────────────────────
# Preview directories with eza, files with bat
zstyle ':fzf-tab:complete:cd:*' fzf-preview 'eza -1 --color=always --icons $realpath 2>/dev/null || ls -1 $realpath'
zstyle ':fzf-tab:complete:*:*' fzf-preview 'bat --color=always --style=numbers --line-range=:200 $realpath 2>/dev/null || cat $realpath 2>/dev/null || eza -1 --color=always $realpath 2>/dev/null'
# Use tmux popup for fzf-tab
zstyle ':fzf-tab:*' fzf-command ftb-tmux-popup
```

### 5. Add fzf git helpers

Add a new section to `.zshrc`:

```bash
# ── fzf git helpers ──────────────────────────────────────────────────────────
# Fuzzy-find git branches
fzf-git-branch() {
    git branch --all --sort=-committerdate --format='%(refname:short)' |
        fzf --tmux center,60%,40% --preview 'git log --oneline -10 {}' |
        sed 's|^origin/||'
}

# Fuzzy-find git log entries
fzf-git-log() {
    git log --oneline --decorate --all -50 |
        fzf --tmux center,80%,60% --preview 'git show --color=always {1}' |
        awk '{print $1}'
}

alias gb='fzf-git-branch'
alias gl='fzf-git-log'
```

## Verification

1. Open a new shell — fzf should show Catppuccin Mocha colors (dark background, pink highlights, purple prompt)
2. Inside tmux, press `Ctrl+T` — fzf opens in a centered popup, not inline
3. Type `cd <Tab>` — fzf-tab shows directory completion with eza preview
4. Run `gb` in a git repo — fuzzy-finds branches with log preview
5. Run `gl` in a git repo — fuzzy-finds commits with diff preview
6. Outside tmux, fzf still works normally (inline mode)

## Edge Cases
- **Old fzf (< 0.53)**: The `--tmux` flag is silently ignored; fzf falls back to inline mode
- **tmux < 3.3**: Popup mode won't work; fzf falls back to inline
- **eza/bat not installed**: fzf-tab preview falls back to `ls`/`cat`
- **fzf-tab + zsh-vi-mode**: Both modify completion; `fzf-tab` before other plugins in the list ensures it takes priority
- **Not in a git repo**: `gb`/`gl` will show git errors — expected behavior

## Dependencies
- fzf 0.53+ (for `--tmux` popup mode; older versions work without popups)
- tmux 3.3+ (for popup support)
- `Aloxaf/fzf-tab` plugin (installed via bootstrap.sh)
- eza and bat (from modern-cli-tools plan) for previews; optional, falls back gracefully
