---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: Modern CLI Tools

## Goal
Install modern Rust/Go CLI replacements (eza, bat, delta, fd, ripgrep, dust, lazygit, tldr) via the bootstrap script and add shell aliases so they replace their legacy counterparts transparently.

## Context
- Bootstrap script already installs core tools via Homebrew (macOS) and apt/dnf (Linux)
- `.zshrc` already has an aliases section
- Catppuccin Mocha theme is used everywhere — bat and delta both have Catppuccin themes
- The git config plan (separate) will configure delta as the git pager; this plan installs it

## Architecture

```
bootstrap.sh  →  installs tools per platform
zsh/.zshrc    →  aliases (ls→eza, cat→bat, etc.) guarded by command -v
```

## Implementation Steps

### 1. Add tool installation to bootstrap.sh

**macOS section** — extend the `brew install` line:

```bash
brew install eza bat git-delta fd ripgrep dust lazygit tldr
```

**Linux section** — add after existing package installs:

```bash
# ── Modern CLI tools ─────────────────────────────────────────────────────────
if command -v apt &>/dev/null; then
    sudo apt install -y eza bat fd-find ripgrep
elif command -v dnf &>/dev/null; then
    sudo dnf install -y eza bat fd-find ripgrep
fi

# Tools not in standard repos — install via cargo or binary release
if ! command -v dust &>/dev/null; then
    echo "==> Note: 'dust' not found. Install via: cargo install du-dust"
fi
if ! command -v lazygit &>/dev/null; then
    echo "==> Note: 'lazygit' not found. See https://github.com/jesseduffield/lazygit#installation"
fi
if ! command -v delta &>/dev/null; then
    echo "==> Note: 'delta' not found. See https://github.com/dandavison/delta#installation"
fi
if ! command -v tldr &>/dev/null; then
    echo "==> Note: 'tldr' not found. Install via: npm install -g tldr  OR  brew install tldr"
fi
```

Note: On Debian/Ubuntu, `bat` installs as `batcat` and `fd` as `fdfind`. The aliases below handle this.

### 2. Add aliases to `.zshrc`

Add a new section after the existing aliases:

```bash
# ── Modern CLI aliases (use replacements when available) ─────────────────────
command -v eza &>/dev/null && alias ls='eza --icons --group-directories-first'
command -v eza &>/dev/null && alias ll='eza -la --icons --group-directories-first'
command -v eza &>/dev/null && alias tree='eza --tree --icons'

# bat: handle Debian's 'batcat' rename
if command -v bat &>/dev/null; then
    alias cat='bat --paging=never'
elif command -v batcat &>/dev/null; then
    alias cat='batcat --paging=never'
fi

# fd: handle Debian's 'fdfind' rename
if command -v fd &>/dev/null; then
    alias find='fd'
elif command -v fdfind &>/dev/null; then
    alias find='fdfind'
fi

command -v rg &>/dev/null && alias grep='rg'
command -v dust &>/dev/null && alias du='dust'
```

### 3. Configure bat theme

Add to `.zshrc` in the aliases/config section:

```bash
# ── bat configuration ────────────────────────────────────────────────────────
export BAT_THEME="Catppuccin Mocha"
```

bat ships with Catppuccin themes built-in since v0.24. Verify with `bat --list-themes | grep -i catppuccin`.

### 4. Set fzf to use fd and bat for previews

Add to the fzf section of `.zshrc`:

```bash
# Use fd for faster, .gitignore-respecting file search
if command -v fd &>/dev/null; then
    export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow --exclude .git'
    export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"
    export FZF_ALT_C_COMMAND='fd --type d --hidden --follow --exclude .git'
fi

# Use bat for fzf previews
if command -v bat &>/dev/null; then
    export FZF_CTRL_T_OPTS="--preview 'bat --color=always --style=numbers --line-range=:500 {}'"
elif command -v batcat &>/dev/null; then
    export FZF_CTRL_T_OPTS="--preview 'batcat --color=always --style=numbers --line-range=:500 {}'"
fi
```

## Verification

1. Run `./bootstrap.sh` on macOS — all tools install via Homebrew
2. Run `./bootstrap.sh` on Linux — available tools install, missing ones print install instructions
3. Open a new shell:
   - `ls` shows eza output with icons
   - `cat somefile` shows bat output with syntax highlighting
   - `grep pattern .` uses ripgrep
   - `Ctrl+T` in fzf shows bat preview pane
4. `echo $BAT_THEME` returns `Catppuccin Mocha`

## Edge Cases
- **Debian bat/fd renaming**: Aliases check for both `bat`/`batcat` and `fd`/`fdfind`
- **Tool not installed**: Each alias is guarded by `command -v` — if the tool isn't present, the original command works as before
- **Nested aliases**: Running `\cat` (backslash) bypasses the alias if you need the original
- **bat theme not found**: Older bat versions may not include Catppuccin; `bat --list-themes` will show what's available

## Dependencies
- Homebrew (macOS) — already required by bootstrap
- apt or dnf (Linux) — already used by bootstrap
- Some Linux tools may need manual install (dust, lazygit, delta, tldr) — script prints guidance
