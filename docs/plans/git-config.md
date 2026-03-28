---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: Git Configuration Stow Package

## Goal
Create a `git` stow package managing `~/.gitconfig` with delta as the pager (Catppuccin Mocha theme), useful aliases, and sensible defaults.

## Context
- No git config is currently tracked in this repo
- Delta is installed via the modern-cli-tools plan (or manually)
- Catppuccin Mocha is the standard theme across all tools
- Git is used heavily with Claude Code for agentic development workflows

## Architecture

```
git/
└── .gitconfig    ← stow symlinks this to ~/.gitconfig
```

New stow package added to the `for pkg in ...` loop in `bootstrap.sh`.

## Implementation Steps

### 1. Create the stow package directory

```bash
mkdir -p git
```

### 2. Create `git/.gitconfig`

```ini
# ── Core ─────────────────────────────────────────────────────────────────────
[user]
    # Set in ~/.gitconfig.local (not tracked)
    # name = Your Name
    # email = you@example.com

[include]
    path = ~/.gitconfig.local

[init]
    defaultBranch = main

[core]
    editor = nvim
    pager = delta

[pull]
    rebase = true

[push]
    autoSetupRemote = true

[fetch]
    prune = true

[rerere]
    enabled = true

# ── Delta (git pager) ───────────────────────────────────────────────────────
[interactive]
    diffFilter = delta --color-only

[delta]
    navigate = true
    side-by-side = true
    line-numbers = true
    hyperlinks = true
    syntax-theme = "Catppuccin Mocha"

[merge]
    conflictstyle = zdiff3

[diff]
    colorMoved = default

# ── Aliases ──────────────────────────────────────────────────────────────────
[alias]
    lg = log --oneline --graph --decorate --all -20
    recent = branch --sort=-committerdate --format='%(committerdate:relative)\t%(refname:short)'
    undo = reset --soft HEAD~1
    amend = commit --amend --no-edit
    wip = !git add -A && git commit -m 'wip'
    stat = diff --stat
    branches = branch -a
    stashes = stash list
    aliases = config --get-regexp alias
```

### 3. Create a `.gitconfig.local` template

Add a comment in the plan noting that users should create `~/.gitconfig.local` for machine-specific settings (name, email, signing keys). This file is NOT tracked. The `[include]` directive in `.gitconfig` picks it up automatically.

### 4. Add `git` to bootstrap.sh stow loop

Change:
```bash
for pkg in tmux zsh ghostty starship; do
```
To:
```bash
for pkg in tmux zsh ghostty starship git; do
```

### 5. Install delta in bootstrap.sh

This is handled by the modern-cli-tools plan. If implementing this plan standalone, add to macOS:
```bash
brew install git-delta
```

And print a note on Linux if delta isn't found (binary install varies by distro).

## Verification

1. Run `stow git` from the dotfiles directory — `~/.gitconfig` symlink is created
2. Run `git lg` — shows pretty log graph
3. Run `git diff` on any repo — delta renders side-by-side with Catppuccin syntax highlighting
4. Run `git recent` — lists branches sorted by last commit
5. `git undo` soft-resets the last commit
6. Machine-specific `~/.gitconfig.local` is included if present, ignored if absent

## Edge Cases
- **Delta not installed**: Git falls back to its default pager (less). The config is safe without delta.
- **User already has ~/.gitconfig**: Stow will refuse to overwrite — user must back up or remove the existing file first. `stow --adopt git` can pull the existing file into the repo.
- **No ~/.gitconfig.local**: The `[include]` directive silently ignores missing files — no error.
- **User identity not set**: The template deliberately leaves `[user]` commented out, prompting users to set it in `.gitconfig.local`. Git will warn on first commit if not set.

## Dependencies
- GNU Stow (already required)
- git-delta (installed by modern-cli-tools plan, or manually)
- Neovim (for `core.editor`; already installed by bootstrap)
