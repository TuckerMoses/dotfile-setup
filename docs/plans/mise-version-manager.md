---
status: pending
created: 2026-03-28
updated: 2026-03-28
---

# Plan: mise Version Manager

> **High-effort / High-reward.** This plan involves replacing nvm and potentially conda/pyenv with a single tool, and modifying shell startup significantly. Expect ~1 hour of implementation and testing, versus ~15 minutes for the simpler plans. The payoff is a noticeably faster shell startup and unified version management.

## Goal
Adopt [mise](https://mise.jdx.dev/) as the single version manager for Node.js, Python, and other language runtimes, replacing nvm and simplifying the conda/pyenv setup in `.zshrc`. This also improves shell startup time since mise activates in ~10ms vs nvm's ~200-500ms.

## Context
- `.zshrc.local` likely contains nvm and conda initialization blocks that add 300-700ms to shell startup
- mise is a single Rust binary that replaces nvm, pyenv, rbenv, and similar tools
- Compatible with `.tool-versions` (asdf ecosystem) and its own `mise.toml`
- Also handles per-directory env vars, partially overlapping with direnv
- Agentic development workflows benefit from fast shell startup (agents spawn many shells)

## Architecture

```
bootstrap.sh    →  installs mise, removes nvm if present
zsh/.zshrc      →  replaces nvm/pyenv init with mise activate
mise/.config/mise/config.toml  →  global tool versions (stow package)
```

## Implementation Steps

### 1. Install mise via bootstrap.sh

**macOS section:**
```bash
brew install mise
```

**Linux section:**
```bash
if ! command -v mise &>/dev/null; then
    curl https://mise.jdx.dev/install.sh | sh
fi
```

### 2. Create mise stow package

```bash
mkdir -p mise/.config/mise
```

Create `mise/.config/mise/config.toml`:

```toml
# ── Global tool versions ─────────────────────────────────────────────────────
# mise manages runtime versions (replaces nvm, pyenv, rbenv).
# Override per-project with a local .mise.toml or .tool-versions file.

[tools]
node = "lts"
python = "latest"

[settings]
# Auto-install missing tool versions when entering a directory
auto_install = true
# Use .tool-versions files for asdf compatibility
legacy_version_file = true
```

Add `mise` to the stow loop:
```bash
for pkg in tmux zsh ghostty starship git atuin mise; do
```

### 3. Activate mise in `.zshrc`

Add after the Oh My Zsh section:

```bash
# ── mise (version manager for node, python, etc.) ───────────────────────────
if command -v mise &>/dev/null; then
    eval "$(mise activate zsh)"
fi
```

This replaces any nvm/pyenv/conda initialization blocks. mise's `activate` runs in ~10ms.

### 4. Migrate from nvm

If `.zshrc.local` contains nvm initialization:

```bash
# REMOVE these lines from .zshrc.local:
# export NVM_DIR="$HOME/.nvm"
# [ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
# [ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```

Then install the same Node version via mise:

```bash
mise use --global node@lts
```

mise reads existing `.nvmrc` and `.node-version` files automatically (`legacy_version_file = true`), so per-project versions continue to work.

### 5. Migrate from conda/pyenv (optional)

If conda is in use, mise can manage Python versions but does NOT replace conda environments for data science workflows. Recommended approach:

- Use mise for Python version management (`mise use python@3.12`)
- Keep conda for environment management only (remove conda's version management)
- Or: use mise's `venv` backend for simple virtual environments

If pyenv is in use, mise is a direct replacement:
```bash
# Remove pyenv init from .zshrc.local
# Install same Python version:
mise use --global python@3.12
```

### 6. Add Starship mise module

Add to `starship/.config/starship.toml` format string and add module config:

In the format string, add `$mise` (or rely on the individual language modules which mise makes work automatically since it shims the binaries).

No Starship config change is strictly needed — Starship's existing `nodejs`, `python`, etc. modules detect versions from the PATH, which mise manages.

## Verification

1. Run `./bootstrap.sh` — mise is installed
2. `mise --version` prints version
3. `mise ls` shows installed tools
4. `cd` into a project with `.nvmrc` or `.node-version` — `node --version` matches the specified version
5. `cd` into a project with `.python-version` — `python --version` matches
6. Benchmark shell startup: `hyperfine 'zsh -ic exit'` — should be under 200ms (vs 500ms+ with nvm)
7. `mise use node@20` in a project directory creates a local `.mise.toml`

## Edge Cases
- **nvm still installed**: mise and nvm can coexist but will conflict on PATH. Recommend removing nvm init from `.zshrc.local` after migration.
- **conda environments**: mise does NOT replace `conda activate`. Keep conda for env activation if needed, but let mise manage the Python binary version.
- **`.tool-versions` files**: mise reads these for asdf compatibility. Existing asdf users can switch with zero config changes.
- **Global vs local**: `mise use --global` sets default versions; `mise use` (without `--global`) creates a project-local `.mise.toml`.
- **Shell startup regression**: If mise is misconfigured or has many tools, startup could slow down. Monitor with `mise doctor`.
- **CI environments**: mise is not needed in CI — the `.tool-versions` / `.mise.toml` files work with both mise and asdf for CI portability.

## Dependencies
- mise binary (installed via Homebrew or install script)
- Existing `.nvmrc` / `.node-version` / `.python-version` files continue to work
- Does NOT require removing conda — the two can coexist with care
