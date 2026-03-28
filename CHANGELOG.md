# Changelog

All notable changes to this dotfiles setup.

## [Unreleased]

### Planned
- tmux agent completion notifications ([plan](docs/plans/tmux-agent-notifications.md))
- Neovim ↔ Claude Code sync ([plan](docs/plans/neovim-claude-sync.md))
- Cross-platform fixes for zsh-vi-mode and hardcoded paths ([plan](docs/plans/cross-platform-fixes.md))
- Modern CLI tools: eza, bat, delta, fd, ripgrep ([plan](docs/plans/modern-cli-tools.md))
- Git configuration stow package with delta pager ([plan](docs/plans/git-config.md))
- fzf enhancements: Catppuccin colors, fzf-tab, tmux popups ([plan](docs/plans/fzf-enhancements.md))
- tmux plugins: resurrect, continuum, yank, sessionx ([plan](docs/plans/tmux-plugins.md))
- Atuin shell history replacement ([plan](docs/plans/atuin-shell-history.md))
- mise version manager (replaces nvm/pyenv) ([plan](docs/plans/mise-version-manager.md))

## [2026-03-27]

### Added
- Implementation plans for agent notifications and neovim sync

### Changed
- Moved zoxide init to end of .zshrc (fixes config warning)
- Merged system-agnostic dotfiles PR (portable paths, platform guards)

## [2026-03-26]

### Added
- Initial dotfiles repo with GNU Stow
- Ghostty config: Catppuccin Mocha theme, JetBrains Mono Nerd Font
- tmux config: Catppuccin status bar, vim-tmux-navigator, vi copy mode
- zsh config: Starship prompt, fzf, zoxide, autosuggestions, syntax-highlighting
- Bootstrap script for one-command setup on new machines

### Fixed
- Ghostty theme name (spaces not dashes)
- Removed invalid `repaint-after-input` from Ghostty config
- Cleaned up `.zshenv` self-replicating Android SDK exports (392KB → 2 lines)
