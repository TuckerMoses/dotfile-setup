# Ideas & Potential Improvements

Last reviewed: 2026-03-30. Items roughly ordered by expected value.

---

## Shell History: atuin

Replace the default zsh Ctrl+R with [atuin](https://github.com/atuinsh/atuin) — a
SQLite-backed shell history tool with fuzzy search, per-directory filtering, command
duration tracking, and optional encrypted cross-machine sync.

Works alongside fzf (they serve different purposes). Drop-in setup with `eval "$(atuin init zsh)"`.
Now at v18.4 with Atuin Desktop (executable runbooks for teams) and shell alias/env var management.

## Version Management: mise

[mise](https://mise.jdx.dev/) (formerly rtx) is a fast, Rust-based replacement for
nvm, pyenv, rbenv, and similar tools — all in one binary. Compatible with `.tool-versions`
(asdf ecosystem) and its own `mise.toml`. It also handles per-directory env vars and
project tasks, partially overlapping with direnv and Makefiles.

Adopting mise would let us drop the nvm block and simplify the conda setup in `.zshrc`,
fixing shell startup time in the process. Latest version (2026.3.17) has graduated tasks
feature and SOPS integration for secrets.

## tmux Plugins

Worth adding to the existing TPM setup:

- **tmux-resurrect** + **tmux-continuum** — persist and auto-restore sessions across reboots.
- **catppuccin/tmux** — official Catppuccin theme plugin; would replace the hand-rolled status bar
  colors and stay in sync with the Ghostty/Starship theme automatically. Supports customizable
  status modules (git, battery, weather, etc.).
- **tmux-sessionx** — fzf-powered session switcher.
- **tmux-thumbs** or **tmux-fingers** — Vimium-style hints to quick-copy visible URLs, paths, and hashes.
- **tmux-yank** — cross-platform clipboard integration. Auto-detects `pbcopy`, `xclip`, `xsel`,
  `wl-copy`. Would fix the current macOS-only `pbcopy` hardcoding in the tmux config.
- **tmux-fzf-url** — `prefix + u` to fuzzy-find and open any URL visible in the terminal.
  Useful for quickly opening GitHub links, error URLs, or documentation from command output.

## tmux 3.6 Features

tmux 3.6 (November 2025) added features worth enabling:

- **Native scrollbars** — `set -g pane-scrollbars modal` shows overlay scrollbars in copy mode.
  No pane space consumed. Also `pane-scrollbars-position` for left/right placement.
- **Theme auto-detection** (mode 2031) — tmux reports dark/light theme to applications.
- **Enhanced format expressions** — `&&`/`||` with arbitrary arguments, else-if in conditionals.

Note: tmux 3.5 reduced the default `escape-time` to 10ms, so our manual `set -sg escape-time 10`
is now redundant and could be removed if on 3.5+.

## Modern CLI Tools

Mature Rust/Go replacements worth aliasing in `.zshrc`:

| Tool | Replaces | Notes |
|------|----------|-------|
| **eza** | `ls` | git-aware, icons, tree view |
| **bat** | `cat` | syntax highlighting, git gutter; also powers fzf previews |
| **delta** | `diff` | side-by-side git diffs with syntax highlighting; has a Catppuccin theme |
| **fd** | `find` | simpler syntax, respects `.gitignore` |
| **ripgrep** | `grep` | very fast, used internally by VS Code and Neovim |
| **dust** | `du` | visual disk usage |
| **lazygit** | — | TUI git client, great for interactive rebase and hunk staging (~55k stars) |
| **yazi** | ranger/nnn | async file manager with image preview in Ghostty, Lua plugin system |
| **btop** | `top`/`htop` | polished system monitor TUI |
| **tldr** | `man` | community-maintained simplified command examples |

The bootstrap script should install these, and `.zshrc` should alias `ls`→`eza`,
`cat`→`bat`, `grep`→`rg`, `du`→`dust`, `find`→`fd` where available.

## Git Configuration (new stow package)

Create a `git` stow package to manage `~/.gitconfig`. Would include:

- **delta** as the default pager with Catppuccin Mocha theme.
- Useful aliases: `git lg` (pretty log graph), `git recent` (recent branches), `git undo` (soft reset last commit).
- **git-absorb** — automatically fixup commits based on staged changes. Great for cleaning up
  PR history, especially with AI-generated code that needs post-hoc fixes.
- Default branch, pull strategy, and other sensible defaults.

## fzf Enhancements

- **fzf-tab** — replace zsh's default completion menu with fzf. Tab-complete becomes fuzzy
  with preview windows. Very natural if you already use fzf everywhere.
- **Catppuccin Mocha colors for fzf** — official theme at `catppuccin/fzf`. Match the color
  scheme via `FZF_DEFAULT_OPTS` color flags. Currently a gap in theme consistency.
- **`--tmux` popup mode** — fzf 0.53+ can open in a tmux popup instead of inline. Cleaner
  experience, requires tmux 3.3+.
- **Preview integration** — use bat for file preview, eza for directory preview in fzf widgets.
- **Use fd as default finder** — `export FZF_DEFAULT_COMMAND='fd --type f --hidden --follow'`
  for faster, `.gitignore`-respecting file search.
- **fzf git helpers** — fuzzy-find branches, log entries, and stashes.
- **New in fzf v0.70** — raw mode (non-matching items stay visible but dimmed), `--freeze-right`
  / `--freeze-left` for pinned text, 50%+ performance improvement over v0.49, resizable preview
  windows via border dragging, and `--ghost` placeholder text.

## zsh Plugins to Evaluate

- **fast-syntax-highlighting** — faster drop-in replacement for zsh-syntax-highlighting.
- **zsh-completions** — additional completion definitions for common tools.
- **zsh-you-should-use** — reminds you when you have an alias for a command you typed longhand.

## Plugin Manager: Consider Replacing Oh My Zsh

Oh My Zsh is convenient but heavy (~200 files loaded on startup). Since we already use Starship
for the prompt, OMZ is mainly providing plugin management and a few built-in completions. Lighter
alternatives:

- **antidote** — clean, fast, uses a plain-text plugin list. Good balance of speed and simplicity.
- **zinit** — "Turbo mode" lazy-loads plugins in background after prompt appears. 50-80% faster
  startup. More complex config syntax. Community fork (original author deleted the repo).
- **sheldon** — Rust-based, TOML config, cross-shell. Fastest option.

Benchmark first with `hyperfine 'zsh -ic exit'` to see if the current startup is actually a problem.

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
- Add `docker_context` module to show container context when relevant.
- Add `os` module to show a platform icon (Apple/Linux) — useful since the setup targets both
  macOS and Linux, especially when SSH-ing between machines.

## Ghostty

- **Ghostty 1.3 features** — scrollback search (`Ctrl+Shift+F`), click-to-move cursor in
  prompts, native scrollbars, and keybind chaining are all available now. Key config additions:
  ```
  notify-on-command-finish = unfocused
  notify-on-command-finish-action = notify
  notify-on-command-finish-after = 10s
  scrollbar = system
  ```
- **Memory leak fix** — Ghostty 1.3.0 fixed a memory leak triggered by "emoji-heavy, hyperlink-
  heavy, or Claude Code output." Previously could grow to 37GB over 10 days. Upgrade if not
  on 1.3.0+.
- **Scrollback limit** — default is 10MB, which users report is insufficient for long agentic
  coding sessions. Consider increasing via `scrollback-limit` in config.
- **Native splits** — Ghostty has built-in splits and tabs that could complement tmux for
  quick local sessions. tmux remains essential for remote/persistent work, but Ghostty splits
  give tighter font/theme integration. Worth setting up keybinds alongside the tmux workflow.
- **Shell integration** — re-evaluate enabling cursor shape per mode, command marking, and
  clickable file paths.
- **Add Ghostty to bootstrap.sh** — currently not installed by the bootstrap script.

## AI-Assisted Development Workflows

- **Claude Code agent teams** — multi-agent mode where each teammate gets its own tmux pane.
  Set `teammateMode: "tmux"` in `~/.claude.json`. One lead agent coordinates, teammates work
  independently. Recommended 3-4 teammates + 1 lead for medium projects.
- **Claude Code `--worktree`** — creates an isolated git working directory per agent at
  `.claude/worktrees/<name>/` with a dedicated branch. Zero file conflicts between agents.
  Add a `.worktreeinclude` file to auto-copy untracked files like `.env` into new worktrees.
- **TmuxAI** ([tmuxai.dev](https://tmuxai.dev/)) — terminal assistant that observes tmux panes
  and understands cross-pane context. Executes commands in a dedicated pane. Non-intrusive,
  layers on top of existing tmux setup.
- **Shell aliases/functions** — add convenience wrappers for `claude --worktree <name>` that
  create a tmux window + worktree together.

## Cross-Platform Fixes

These are bugs/limitations in the current setup that should be addressed:

- **Fix hardcoded paths in `.zshrc`** — replace `/Users/johnmoses/` with `$HOME` where possible
  (conda, OPAM, Perl, Java blocks).
- **Fix tmux clipboard on Linux** — current config uses `pbcopy` (macOS-only). Either add
  platform detection or adopt `tmux-yank` plugin.

## Fonts to Try

JetBrains Mono is solid, but two newer options are worth trying:

- **Monaspace** (GitHub) — family of 5 variable-width fonts with texture healing and
  ligatures. The Neon variant is closest to a traditional monospace feel.
- **Maple Mono** — distinctive rounded style that's gained a large following. Nerd Font
  variant available.

## Bootstrap Script Improvements

- Add a `Brewfile` for declarative macOS package management instead of inline `brew install`
  commands.
- Add Linux support for Ghostty installation.
- Add `--dry-run` flag to preview what would be installed/linked.

## Lower Priority / Explore Later

- **Zellij** — Rust-based terminal multiplexer with floating panes, WASM plugins, and built-in
  session persistence. Mature enough for daily use, but switching from tmux has real muscle
  memory cost. Benchmarks show higher memory (~22MB vs tmux's minimal) and higher latency at
  scale (183ms vs <50ms at 100 panes). Consider running side-by-side.
- **Neovim stow package** — if not managed elsewhere, track Neovim config in this repo for
  full reproducibility.
