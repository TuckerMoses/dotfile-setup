---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: tmux Plugin Expansion

## Goal
Add session persistence (resurrect + continuum), cross-platform clipboard (tmux-yank), and a session switcher (tmux-sessionx) to the existing TPM-managed tmux setup.

## Context
- TPM is already configured in `.tmux.conf` with `tmux-sensible`
- Clipboard currently uses platform-conditional `pbcopy`/`xclip` bindings (works but fragile)
- No session persistence — reboot loses all tmux sessions
- Catppuccin Mocha theme is hand-rolled in the status bar (keeping it; the official plugin is optional for later)
- User runs multiple tmux sessions for different projects and Claude Code agents

## Architecture

All changes go into `tmux/.tmux.conf` in the existing Plugins section. TPM handles installation via `prefix + I`.

## Implementation Steps

### 1. Add tmux-resurrect and tmux-continuum

Add to the Plugins section in `.tmux.conf`, before the `run '~/.tmux/plugins/tpm/tpm'` line:

```bash
# ── Session persistence ──────────────────────────────────────────────────────
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'

# Restore neovim sessions (requires vim-obsession or similar)
set -g @resurrect-strategy-nvim 'session'
# Restore pane contents
set -g @resurrect-capture-pane-contents 'on'
# Auto-save every 15 minutes, auto-restore on tmux start
set -g @continuum-restore 'on'
set -g @continuum-save-interval '15'
```

**What this does:**
- `tmux-resurrect`: Save (`prefix + Ctrl-s`) and restore (`prefix + Ctrl-r`) tmux sessions, windows, panes, and their working directories
- `tmux-continuum`: Automatically saves every 15 minutes and restores on tmux server start

### 2. Add tmux-yank

Replace the manual clipboard bindings with tmux-yank:

```bash
# ── Clipboard ────────────────────────────────────────────────────────────────
set -g @plugin 'tmux-plugins/tmux-yank'
```

**Remove** the existing manual clipboard lines from the Vi copy mode section:

```bash
# Remove these lines:
if-shell "uname | grep -q Darwin" \
    "bind -T copy-mode-vi y send -X copy-pipe-and-cancel 'pbcopy'" \
    "bind -T copy-mode-vi y send -X copy-pipe-and-cancel 'xclip -in -selection clipboard'"
```

tmux-yank auto-detects the clipboard tool (`pbcopy`, `xclip`, `xsel`, `wl-copy`) and handles `y` in copy mode automatically. This fixes cross-platform clipboard without manual conditionals.

### 3. Add tmux-sessionx

```bash
# ── Session management ───────────────────────────────────────────────────────
set -g @plugin 'omerxx/tmux-sessionx'
set -g @sessionx-bind 'o'
```

**What this does:**
- `prefix + o` opens an fzf-powered session switcher
- Shows all sessions with preview of their windows/panes
- Can create new sessions from the picker
- Particularly useful when running multiple Claude Code agents across sessions

### 4. Ensure TPM is auto-installed by bootstrap.sh

Add to `bootstrap.sh` after the stow section:

```bash
# ── TPM (tmux plugin manager) ────────────────────────────────────────────────
if [[ ! -d "$HOME/.tmux/plugins/tpm" ]]; then
    echo "==> Installing TPM..."
    git clone https://github.com/tmux-plugins/tpm "$HOME/.tmux/plugins/tpm"
fi
```

### 5. Updated `.tmux.conf` Plugins section (complete)

The full Plugins section should look like:

```bash
# ── Plugins (TPM) ─────────────────────────────────────────────────────────────
set -g @plugin 'tmux-plugins/tpm'
set -g @plugin 'tmux-plugins/tmux-sensible'

# ── Session persistence ──────────────────────────────────────────────────────
set -g @plugin 'tmux-plugins/tmux-resurrect'
set -g @plugin 'tmux-plugins/tmux-continuum'
set -g @resurrect-strategy-nvim 'session'
set -g @resurrect-capture-pane-contents 'on'
set -g @continuum-restore 'on'
set -g @continuum-save-interval '15'

# ── Clipboard ────────────────────────────────────────────────────────────────
set -g @plugin 'tmux-plugins/tmux-yank'

# ── Session management ───────────────────────────────────────────────────────
set -g @plugin 'omerxx/tmux-sessionx'
set -g @sessionx-bind 'o'

run '~/.tmux/plugins/tpm/tpm'
```

## Verification

1. Run `prefix + I` to install new plugins via TPM
2. **Resurrect test**: Open a multi-window tmux layout → `prefix + Ctrl-s` to save → kill tmux server → restart tmux → `prefix + Ctrl-r` → layout is restored
3. **Continuum test**: Check `~/.tmux/resurrect/` for auto-saved files after 15 minutes
4. **Yank test**: Enter copy mode (`prefix + [`), select text (`v`), yank (`y`) — verify it's in system clipboard on both macOS and Linux
5. **Sessionx test**: `prefix + o` opens session picker with fzf
6. Run bootstrap.sh — TPM is auto-installed if missing

## Edge Cases
- **TPM not installed**: Bootstrap.sh now handles this; previously was manual
- **No clipboard tool on Linux**: tmux-yank falls back gracefully and prints a warning. Install `xclip` or `xsel` as needed.
- **tmux-sessionx requires fzf**: Already installed via bootstrap. If missing, sessionx will error — but this is caught by bootstrap.
- **Continuum auto-restore**: Only triggers on fresh tmux server start, not when attaching to an existing server
- **Resurrect with Claude Code processes**: Resurrect restores pane working directories but not running processes. Claude Code sessions need to be restarted manually.

## Dependencies
- TPM (auto-installed by updated bootstrap.sh)
- fzf (for tmux-sessionx; already installed)
- `xclip` or `xsel` on Linux (for tmux-yank clipboard)
