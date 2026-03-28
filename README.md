# Mac Setup Guide

Modern macOS power user setup. Run these steps in order on a fresh machine.

## Stack

| Layer | Tool |
|---|---|
| Terminal | iTerm2 |
| Shell | zsh + antidote (plugin mgr) + vi-mode |
| Prompt | Starship (minimal `❯`) + iTerm2 status bar (dir, git, clock) |
| Shell history | atuin (SQLite, cross-machine sync) |
| Multiplexer | tmux + tpm |
| Runtime manager | mise (replaces nvm + pyenv) |
| JS packages | bun (replaces npm) |
| Python packages | uv (replaces pip) |
| Editor | Neovim + LazyVim |
| Dotfiles | chezmoi |
| Window manager | AeroSpace |
| Color scheme | Catppuccin Mocha (all tools) |
| Font | JetBrains Mono Nerd Font |

---

## Step 1 — Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
eval "$(/opt/homebrew/bin/brew shellenv)"
```

---

## Step 2 — Install Everything via Brewfile

All packages (formulae, casks, fonts) are tracked in `Brewfile`:

```bash
brew bundle --file=Brewfile
```

**Note:** `git-credential-manager` requires sudo — install separately if it fails:
```bash
brew install --cask git-credential-manager
```

To add a new package later, add it to `Brewfile` and re-run `brew bundle`.

---

## Step 3 — Shell & tmux Integrations

```bash
# fzf key bindings (non-interactive)
$(brew --prefix)/opt/fzf/install --all

# tmux plugin manager
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# iTerm2 shell integration (marks, imgcat, jump-to-command)
curl -L https://iterm2.com/shell_integration/zsh -o ~/.iterm2_shell_integration.zsh
```

---

## Step 4 — bat Catppuccin Theme

```bash
mkdir -p "$(bat --config-dir)/themes"
curl -Lo "$(bat --config-dir)/themes/Catppuccin-mocha.tmTheme" \
  "https://raw.githubusercontent.com/catppuccin/bat/main/themes/Catppuccin%20Mocha.tmTheme"
bat cache --build
```

---

## Step 5 — iTerm2 Catppuccin Colors

```bash
curl -LO https://raw.githubusercontent.com/catppuccin/iterm/main/colors/catppuccin-mocha.itermcolors
open catppuccin-mocha.itermcolors
```

In iTerm2: **Profiles > Colors > Color Presets... > catppuccin-mocha**

---

## Step 6 — iTerm2 GUI Settings (manual, one-time)

- **Appearance > General**: Theme = `Minimal`, Status bar location = `Bottom`
- **Profiles > Text**:
  - Font: `JetBrainsMono Nerd Font`, size `14`, enable ligatures
  - Non-ASCII Font: `Symbols Nerd Font Mono`, same size
- **Profiles > Window**: Columns `220`, Rows `50`, Transparency `5%`, Blur `5`
- **Profiles > Terminal**: Scrollback lines = `100000`, enable mouse reporting
- **Profiles > Keys > Key Mappings**: click **Presets...** button at bottom > `Natural Text Editing`
- **Profiles > Session**: Enable status bar, click **Configure Status Bar**, add: Current Directory, Git State, Clock
- **Advanced**: search "GPU" > enable GPU renderer

---

## Step 7 — Write Config Files

Copy the config files from the `configs/` directory in this repo:

```bash
# zsh
cp configs/zshrc ~/.zshrc
cp configs/zsh_plugins.txt ~/.zsh_plugins.txt

# Starship prompt
cp configs/starship.toml ~/.config/starship.toml

# tmux
cp configs/tmux.conf ~/.tmux.conf

# AeroSpace window manager
mkdir -p ~/.config/aerospace
cp configs/aerospace.toml ~/.config/aerospace/aerospace.toml

# ripgrep
mkdir -p ~/.config/ripgrep
cp configs/ripgrep.conf ~/.config/ripgrep/config

# Claude Code global instructions
mkdir -p ~/.claude
cp configs/CLAUDE.md ~/.claude/CLAUDE.md
```

---

## Step 8 — Git Configuration

```bash
# Identity — change these to your own
git config --global user.name "Your Name"
git config --global user.email "you@example.com"

# Sensible defaults
git config --global init.defaultBranch main
git config --global pull.rebase true
git config --global push.autoSetupRemote true
git config --global core.editor nvim

# Delta as diff pager (side-by-side, syntax highlighted)
git config --global core.pager delta
git config --global interactive.diffFilter "delta --color-only"
git config --global delta.navigate true
git config --global delta.side-by-side true
git config --global delta.line-numbers true
git config --global delta.syntax-theme Catppuccin-mocha
git config --global merge.conflictstyle diff3
git config --global diff.colorMoved default
```

---

## Step 9 — SSH Keys

```bash
ssh-keygen -t ed25519 -C "you@example.com"
```

Add to GitHub:
```bash
cat ~/.ssh/id_ed25519.pub | pbcopy
# Paste in GitHub > Settings > SSH and GPG keys > New SSH key
```

---

## Step 10 — Runtimes & Neovim

```bash
# Reload shell to activate new .zshrc
source ~/.zshrc

# Global runtimes via mise
mise use --global node@lts
mise use --global python@3.13

# LazyVim (neovim config framework)
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
nvim  # plugins auto-install on first launch — wait, then :q
```

---

## Step 11 — tmux Plugins

```bash
tmux new-session -d -s init
tmux run-shell ~/.tmux/plugins/tpm/scripts/install_plugins.sh
```

---

## Step 12 — chezmoi Dotfile Tracking

```bash
chezmoi init
chezmoi add ~/.zshrc ~/.zsh_plugins.txt \
             ~/.config/starship.toml \
             ~/.tmux.conf \
             ~/.config/ripgrep/config \
             ~/.config/aerospace/aerospace.toml \
             ~/.claude/CLAUDE.md
```

To push to a remote git repo for cross-machine sync:
```bash
chezmoi cd
git remote add origin <your-dotfiles-repo-url>
git push -u origin main
```

On a new machine, restore with:
```bash
chezmoi init <your-github-username>
chezmoi apply
```

---

## Step 13 — iTerm2 Config Backup

Export iTerm2 preferences (for future machine migration):
```bash
plutil -convert xml1 -o ~/Stash/repos/mac-setup/configs/iterm2-config.xml \
  ~/Library/Preferences/com.googlecode.iterm2.plist
```

Import on a new machine:
```bash
plutil -convert binary1 -o ~/Library/Preferences/com.googlecode.iterm2.plist \
  ~/Stash/repos/mac-setup/configs/iterm2-config.xml
```

---

## Verification Checklist

```bash
starship --version          # prompt
atuin --version             # shell history
mise ls                     # node + python runtimes
bun --version               # JS package manager
uv --version                # Python package manager
ls ~                        # eza icons visible
cat ~/.zshrc                # bat syntax highlighting
nvim                        # LazyVim loads
tmux new -s test            # Catppuccin statusbar
chezmoi diff                # no diff = dotfiles in sync
```

---

## Key Bindings Reference

### Shell (vi-mode)
| Key | Action |
|---|---|
| `ESC` | enter vi normal mode (hjkl, w, b, 0, $, etc.) |
| `i` / `a` | back to insert mode |
| `Ctrl-R` | atuin history search (full-screen, filterable) |
| `Ctrl-T` | fzf file search |
| `Alt-C` | fzf cd into directory |
| `Ctrl-Space` | accept autosuggestion |
| `Ctrl-X Ctrl-E` | edit current command in nvim |
| `Ctrl-A` / `Ctrl-E` | beginning/end of line (in insert mode) |
| `Tab` | fzf-tab completion with preview |

### tmux (prefix = `Ctrl-a`)
| Key | Action |
|---|---|
| `Ctrl-a \|` | split pane horizontally |
| `Ctrl-a -` | split pane vertically |
| `Ctrl-a hjkl` | navigate panes (vim-style) |
| `Ctrl-a c` | new window |
| `Ctrl-a r` | reload config |
| `Ctrl-a d` | detach session |

### AeroSpace
| Key | Action |
|---|---|
| `Alt-hjkl` | focus window |
| `Alt-Shift-hjkl` | move window |
| `Alt-1..5` | switch workspace |
| `Alt-Shift-1..5` | move window to workspace |
| `Alt-f` | fullscreen |
| `Alt-Shift-f` | toggle floating |
| `Alt-Shift-r` | reload config |

---

## Tool Aliases Quick Reference

| Alias | Expands to |
|---|---|
| `v` / `vim` / `vi` | `nvim` |
| `vimdiff` | `nvim -d` |
| `python` | `python3` |
| `npm` | `bun` |
| `pip` | `uv pip` |
| `cat` | `bat --paging=never` |
| `ls` / `ll` / `l` / `la` / `lt` | `eza` variants |
| `grep` | `rg` (ripgrep) |
| `find` | `fd` |
| `top` | `btm` (bottom) |
| `ps` | `procs` |
| `lg` | `lazygit` |
| `ta` | `tmux attach` |
| `cz` | `chezmoi` |
| `sz` | `source ~/.zshrc` |

---

## Claude Code Global Instructions

`~/.claude/CLAUDE.md` is loaded in every Claude Code conversation. Current rules:
- Use `bun` instead of npm/yarn/pnpm
- Use `uv` instead of pip/pip3/pipenv
