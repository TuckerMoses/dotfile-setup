# Ideas & Potential Improvements

Researched 2026-03-26. Items roughly ordered by expected value.

---

## Shell History: atuin

Replace the default zsh Ctrl+R with [atuin](https://github.com/atuinsh/atuin) — a
SQLite-backed shell history tool with fuzzy search, per-directory filtering, command
duration tracking, and optional encrypted cross-machine sync.

Works alongside fzf (they serve different purposes). Drop-in setup with `eval "$(atuin init zsh)"`.

## Version Management: mise

[mise](https://mise.jdx.dev/) (formerly rtx) is a fast, Rust-based replacement for
nvm, pyenv, rbenv, and similar tools — all in one binary. Compatible with `.tool-versions`
(asdf ecosystem) and its own `mise.toml`. It also handles per-directory env vars and
project tasks, partially overlapping with direnv and Makefiles.

Adopting mise would let us drop the nvm block and simplify the conda setup in `.zshrc`.

## tmux Plugins

Worth adding to the existing TPM setup:

- **tmux-resurrect** + **tmux-continuum** — persist and auto-restore sessions across reboots.
- **catppuccin/tmux** — official Catppuccin theme; would replace the hand-rolled status bar
  colors and stay in sync with the Ghostty/Starship theme automatically.
- **tmux-sessionx** — fzf-powered session switcher.
- **tmux-thumbs** or **tmux-fingers** — Vimium-style hints to quick-copy visible URLs, paths, and hashes.

## Modern CLI Tools

Mature Rust/Go replacements worth aliasing in `.zshrc`:

| Tool | Replaces | Notes |
|------|----------|-------|
| **eza** | `ls` | git-aware, icons, tree view |
| **bat** | `cat` | syntax highlighting, git gutter; also powers fzf previews |
| **delta** | `diff` | side-by-side git diffs with syntax highlighting (`git config core.pager delta`) |
| **fd** | `find` | simpler syntax, respects `.gitignore` |
| **ripgrep** | `grep` | very fast, used internally by VS Code |
| **dust** | `du` | visual disk usage |
| **lazygit** | — | TUI git client, great for interactive rebase and hunk staging |
| **yazi** | ranger/nnn | async file manager with image preview in Ghostty, Lua plugin system |
| **btop** | `top`/`htop` | polished system monitor TUI |

The bootstrap script should install these, and `.zshrc` should alias `ls`→`eza`,
`cat`→`bat`, `grep`→`rg`, `du`→`dust`, `find`→`fd` where available.

## Plugin Manager: Replace Oh My Zsh

Oh My Zsh is convenient but heavy. Since we already use Starship for the prompt, OMZ is
mainly providing plugin management and a few built-in completions. Lighter alternatives:

- **antidote** — clean, fast, uses a plain-text plugin list. Good balance of speed and
  simplicity.
- **sheldon** — Rust-based, TOML config, cross-shell. Fastest option.

Either would noticeably improve shell startup time. Benchmark first with
`hyperfine 'zsh -ic exit'` to see if the current startup is actually a problem.

## zsh Plugins to Evaluate

- **fast-syntax-highlighting** — faster drop-in replacement for zsh-syntax-highlighting.
- **zsh-completions** — additional completion definitions for common tools.
- **zsh-you-should-use** — reminds you when you have an alias for a command you typed
  longhand.
- **zsh-autopair** — auto-close brackets and quotes on the command line.

## Per-Directory Environments: direnv

[direnv](https://direnv.net/) auto-loads `.envrc` files when you `cd` into a project.
Useful for per-project env vars, secrets, and Nix dev shells. Has a Starship module.

Note: mise covers basic per-directory env vars via `mise.toml [env]`. Only add direnv if
you need complex logic or Nix integration beyond what mise provides.

## Starship Enhancements

- Use the `palette` feature to define Catppuccin Mocha colors by name and reference them
  across modules — cleaner than inline hex codes.
- Add a `direnv` or `mise` module once those tools are adopted.
- Add `git_metrics` module (shows lines added/removed) for richer git context.

## Ghostty Native Splits

Ghostty has built-in splits and tabs, which could reduce reliance on tmux for local
sessions. tmux remains essential for remote/persistent sessions, but for quick local
work, Ghostty splits avoid the overhead and give tighter font/theme integration.

Worth exploring keybinds that make Ghostty splits feel natural alongside the existing
tmux workflow.

## Fonts to Try

JetBrains Mono is solid, but two newer options are worth trying:

- **Monaspace** (GitHub) — family of 5 variable-width fonts with texture healing and
  ligatures. The Neon variant is closest to a traditional monospace feel.
- **Maple Mono** — distinctive rounded style that's gained a large following. Nerd Font
  variant available.

## Dotfile Management: chezmoi

[chezmoi](https://www.chezmoi.io/) offers templating, encrypted secrets, and multi-machine
support over Stow's simple symlinks. Only worth the migration cost if:

- Managing dotfiles across multiple machines with different configs.
- Needing to store secrets (API keys, tokens) in the repo.
- Wanting a single-binary bootstrap (no `stow` dependency).

Current Stow setup is fine for a single-machine workflow.

## Zellij as a tmux Alternative

[Zellij](https://zellij.dev/) is a Rust-based terminal multiplexer with a discoverable UI,
WASM plugin system, floating panes, and built-in session persistence (no plugin needed).
Mature enough for daily use as of 2025, but switching has a real cost if you have deep tmux
muscle memory. Consider running both side-by-side before committing.

## Bootstrap Script Improvements

- Add a `Brewfile` for declarative macOS package management instead of inline `brew install`
  commands.
- Add Linux support for Ghostty installation (currently only macOS packages are installed).
- Add `--dry-run` flag to preview what would be installed/linked.
