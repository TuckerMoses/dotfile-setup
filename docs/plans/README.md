# Dotfiles Plans

Implementation plans for terminal/workflow improvements. Each plan is self-contained and can be handed off to a Claude Code agent for execution.

## Plans

| Plan | Status | Description |
|------|--------|-------------|
| [tmux-agent-notifications](tmux-agent-notifications.md) | `pending` | macOS desktop notifications when Claude Code finishes, click to switch pane |
| [neovim-claude-sync](neovim-claude-sync.md) | `pending` | Keep Neovim in sync with Claude Code file changes (3 tiers) |
| [cross-platform-fixes](cross-platform-fixes.md) | `pending` | Fix hardcoded paths and cross-platform zsh-vi-mode/clipboard |
| [modern-cli-tools](modern-cli-tools.md) | `pending` | Install eza, bat, delta, fd, ripgrep and add shell aliases |
| [git-config](git-config.md) | `pending` | New stow package for ~/.gitconfig with delta pager and aliases |
| [fzf-enhancements](fzf-enhancements.md) | `pending` | Catppuccin colors, fzf-tab, tmux popups, git helpers |
| [tmux-plugins](tmux-plugins.md) | `pending` | Add resurrect, continuum, tmux-yank, and sessionx plugins |
| [atuin-shell-history](atuin-shell-history.md) | `pending` | SQLite-backed shell history with fuzzy search and per-directory filtering |
| [mise-version-manager](mise-version-manager.md) | `pending` | Replace nvm/pyenv with mise for faster shell startup ⚠️ high-effort |

## Statuses

- `pending` — not started
- `in-progress` — currently being implemented
- `done` — implemented and verified
- `blocked` — waiting on something external

## How to execute a plan

Start a Claude Code session in `~/dotfiles` and run:

```
Execute the plan in docs/plans/<plan-name>.md. Follow the CLAUDE.md instructions and update the plan status, changelog, and commit when done.
```
