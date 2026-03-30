---
status: pending
created: 2026-03-30
updated: 2026-03-30
---

# Plan: Agentic Workflow Helpers

## Goal
Set up Claude Code multi-agent workflows with tmux: teammate mode configuration, git worktree convenience functions, and shell aliases that spin up isolated agent workspaces in dedicated tmux windows.

## Context
- User runs multiple Claude Code agents across tmux panes simultaneously
- tmux 3.6a, TPM, Ghostty, zsh with Oh My Zsh
- Dotfiles repo managed with GNU Stow
- The tmux-agent-notifications plan (separate) handles completion alerts; this plan focuses on **launching and organizing** agent workspaces

## Effort
**Medium-high** — touches multiple config files and introduces new shell functions. Selected despite higher effort because it directly improves the core agentic development workflow.

## Architecture

```
User runs: cw myfeature
  → Shell function creates git worktree at .claude/worktrees/myfeature/
  → Creates a new tmux window named "myfeature"
  → Opens Claude Code with --worktree in that window
  → User works in main window; agent works independently

User runs: cw-clean
  → Lists finished worktrees and removes them + their tmux windows
```

## Implementation Steps

### 1. Create a `claude` stow package

```bash
mkdir -p claude/.claude
```

This package will manage `~/.claude/settings.json` and `~/.claude/.worktreeinclude`.

### 2. Configure teammate mode in settings.json

Create `claude/.claude/settings.json`:

```json
{
  "teammateMode": "tmux"
}
```

This tells Claude Code to spawn teammate agents in separate tmux panes when using multi-agent mode. One lead agent coordinates; teammates work independently in their own panes.

**Note:** If the user already has a `~/.claude/settings.json`, this file will be symlinked over it. Before running `stow claude`, back up or merge any existing settings. The plan-executor should check for an existing file and merge keys rather than overwrite.

### 3. Create .worktreeinclude

Create `claude/.claude/.worktreeinclude`:

```
.env
.env.local
.envrc
```

When Claude Code creates a worktree with `--worktree`, it copies files listed here into the new worktree directory. This ensures environment variables and secrets are available without being tracked in git.

### 4. Add shell functions to .zshrc

Add the following to `zsh/.zshrc` in a new section before the aliases section:

```bash
# ── Claude Code worktree helpers ────────────────────────────────────────────
# cw: Create a Claude Code worktree + tmux window
# Usage: cw <name> [directory]
#   <name>      worktree/branch name (e.g., "fix-auth")
#   [directory] project directory (defaults to current directory)
function cw() {
  local name="${1:?Usage: cw <name> [directory]}"
  local dir="${2:-$(pwd)}"

  if [[ -z "$TMUX" ]]; then
    echo "Error: not inside a tmux session" >&2
    return 1
  fi

  # Create tmux window, cd to project, launch claude with worktree
  tmux new-window -n "$name" -c "$dir" "claude --worktree $name; zsh"
}

# cw-list: List active Claude Code worktrees
function cw-list() {
  local worktree_dir=".claude/worktrees"
  if [[ -d "$worktree_dir" ]]; then
    echo "Active worktrees:"
    for wt in "$worktree_dir"/*/; do
      [[ -d "$wt" ]] && echo "  $(basename "$wt")"
    done
  else
    echo "No worktrees found in $worktree_dir"
  fi
}

# cw-clean: Remove finished worktrees and their tmux windows
function cw-clean() {
  local worktree_dir=".claude/worktrees"
  if [[ ! -d "$worktree_dir" ]]; then
    echo "No worktrees to clean"
    return 0
  fi

  for wt in "$worktree_dir"/*/; do
    [[ ! -d "$wt" ]] && continue
    local name
    name=$(basename "$wt")

    echo -n "Remove worktree '$name'? [y/N] "
    read -r reply
    if [[ "$reply" =~ ^[Yy]$ ]]; then
      # Remove the git worktree
      git worktree remove "$wt" --force 2>/dev/null
      # Kill the tmux window if it still exists
      tmux kill-window -t "=$name" 2>/dev/null
      echo "  Removed $name"
    fi
  done
}
```

### 5. Add the `claude` package to bootstrap.sh

In `bootstrap.sh`, add `claude` to the stow packages loop:

```bash
for pkg in ghostty tmux zsh starship claude; do
```

Also add a guard to merge settings rather than blindly stow:

```bash
# Merge claude settings if existing settings.json has user content
if [[ -f "$HOME/.claude/settings.json" ]] && [[ ! -L "$HOME/.claude/settings.json" ]]; then
  echo "Merging existing ~/.claude/settings.json with dotfiles version..."
  # Back up existing settings
  cp "$HOME/.claude/settings.json" "$HOME/.claude/settings.json.bak"
fi
```

### 6. Add quick-reference aliases

Add to the aliases section in `zsh/.zshrc`:

```bash
alias ct='claude --teammates 3'   # start claude with 3 teammates
alias ctr='claude --resume'       # resume last claude session
```

## Verification

1. **Stow test:**
   ```bash
   cd ~/dotfiles && stow claude
   ls -la ~/.claude/settings.json  # should be symlink
   ls -la ~/.claude/.worktreeinclude  # should be symlink
   ```

2. **Worktree function test:**
   ```bash
   cd ~/some-git-project
   cw test-feature
   # Should: open new tmux window named "test-feature" running claude --worktree test-feature
   ```

3. **List and clean:**
   ```bash
   cw-list        # shows "test-feature"
   cw-clean       # prompts to remove, cleans up worktree + tmux window
   ```

4. **Teammate mode test:**
   ```bash
   ct   # launches claude with 3 teammates in separate tmux panes
   ```

## Edge Cases

- **Not in tmux:** `cw` exits with an error if `$TMUX` is unset
- **Existing settings.json:** bootstrap.sh backs up existing file before stowing; plan-executor should check and merge
- **No git repo:** `cw` will fail if run outside a git repo (claude --worktree requires git); the error from claude is descriptive enough
- **Worktree name conflicts:** git worktree will error if a branch with that name already exists on a different worktree; user must pick a unique name
- **tmux window name collisions:** if a window named "test-feature" already exists, tmux still creates a new one (windows can share names)
- **Linux compatibility:** all commands are cross-platform (no macOS-specific tools)

## Dependencies

- Claude Code CLI with `--worktree` and `--teammates` support
- tmux (already configured)
- git (for worktree commands)
- No additional packages to install
