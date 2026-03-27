# Mac Setup Guide

Modern macOS power user setup. Run these steps in order on a fresh machine.

## Stack

| Layer | Tool |
|---|---|
| Terminal | iTerm2 |
| Shell | zsh |
| Plugin manager | antidote |
| Prompt | Starship (Catppuccin Mocha) |
| Shell history | atuin |
| Multiplexer | tmux + tpm |
| Runtime manager | mise |
| Editor | Neovim + LazyVim |
| Dotfiles | chezmoi |
| Window manager | AeroSpace |
| Color scheme | Catppuccin Mocha (all tools) |
| Font | JetBrains Mono Nerd Font |

---

## Step 1 — Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

After install, add Homebrew to PATH (Apple Silicon):
```bash
eval "$(/opt/homebrew/bin/brew shellenv)"
```

Add the eval line to `~/.zshrc` permanently (already included in the zshrc below).

---

## Step 2 — Install Everything via Brewfile

All packages (formulae, casks, fonts) are tracked in `Brewfile`. One command installs everything:

```bash
brew bundle --file=Brewfile
```

To add a new package later, add it to `Brewfile` and re-run `brew bundle`.

To dump your current installed packages into the Brewfile:
```bash
brew bundle dump --file=Brewfile --force
```

---

## Step 4 — Shell & tmux Integrations

```bash
# fzf key bindings (non-interactive)
$(brew --prefix)/opt/fzf/install --all

# tmux plugin manager
git clone https://github.com/tmux-plugins/tpm ~/.tmux/plugins/tpm

# iTerm2 shell integration (marks, imgcat, jump-to-command)
curl -L https://iterm2.com/shell_integration/zsh -o ~/.iterm2_shell_integration.zsh
```

---

## Step 5 — bat Catppuccin Theme

```bash
mkdir -p "$(bat --config-dir)/themes"
curl -Lo "$(bat --config-dir)/themes/Catppuccin-mocha.tmTheme" \
  "https://raw.githubusercontent.com/catppuccin/bat/main/themes/Catppuccin%20Mocha.tmTheme"
bat cache --build
```

---

## Step 6 — iTerm2 Catppuccin Colors

```bash
curl -LO https://raw.githubusercontent.com/catppuccin/iterm/main/colors/Catppuccin-Mocha.itermcolors
open Catppuccin-Mocha.itermcolors
```

In iTerm2: **Color Presets > Catppuccin-Mocha**

---

## Step 7 — iTerm2 GUI Settings (manual, one-time)

- **Appearance > General**: Theme = `Minimal`
- **Profiles > Text**:
  - Font: `JetBrainsMono Nerd Font`, size `14`, enable ligatures
  - Non-ASCII Font: `Symbols Nerd Font Mono`, same size
- **Profiles > Window**: Columns `220`, Rows `50`, Transparency `5%`, Blur `5`
- **Profiles > Terminal**: Scrollback lines = `100000`, enable mouse reporting
- **Profiles > Keys > Presets**: `Natural Text Editing`
- **Profiles > Session**: Enable status bar (add CPU, memory, git branch components)
- **Advanced**: GPU renderer = enabled

---

## Step 8 — Write Config Files

Copy the config files from the `configs/` directory in this repo:

```bash
# zsh
cp configs/zshrc ~/.zshrc
cp configs/zsh_plugins.txt ~/.zsh_plugins.txt

# Starship prompt
mkdir -p ~/.config/starship
cp configs/starship.toml ~/.config/starship.toml

# tmux
cp configs/tmux.conf ~/.tmux.conf

# AeroSpace window manager
mkdir -p ~/.config/aerospace
cp configs/aerospace.toml ~/.config/aerospace/aerospace.toml

# ripgrep
mkdir -p ~/.config/ripgrep
cp configs/ripgrep.conf ~/.config/ripgrep/config
```

---

## Step 9 — Runtimes & Neovim

```bash
# Reload shell to activate new .zshrc
source ~/.zshrc

# Global runtimes via mise
mise use --global node@lts
mise use --global python@3.13

# LazyVim
git clone https://github.com/LazyVim/starter ~/.config/nvim
rm -rf ~/.config/nvim/.git
nvim  # plugins auto-install on first launch — wait, then :q
```

---

## Step 10 — tmux Plugins

```bash
tmux new-session -d -s init
tmux run-shell ~/.tmux/plugins/tpm/scripts/install_plugins.sh
```

---

## Step 11 — git delta

```bash
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

## Step 12 — chezmoi Dotfile Tracking

```bash
chezmoi init
chezmoi add ~/.zshrc ~/.zsh_plugins.txt \
             ~/.config/starship.toml \
             ~/.tmux.conf \
             ~/.config/ripgrep/config \
             ~/.config/aerospace/aerospace.toml
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

## Verification Checklist

```bash
starship --version          # prompt installed
atuin --version             # shell history installed
mise ls                     # node + python runtimes
ls ~                        # eza icons visible
cat ~/.zshrc                # bat syntax highlighting
nvim                        # LazyVim + Catppuccin loads
tmux new -s test            # Catppuccin statusbar visible
chezmoi diff                # no diff = dotfiles in sync
```

---

## Key Bindings Reference

### Shell
| Key | Action |
|---|---|
| `Ctrl-R` | atuin history search (full-screen, filterable) |
| `Ctrl-T` | fzf file search |
| `Alt-C` | fzf cd into directory |
| `Ctrl-Space` | accept autosuggestion |
| `Ctrl-X Ctrl-E` | edit current command in nvim |
| `Tab` | fzf-tab completion with preview |

### tmux (prefix = `Ctrl-a`)
| Key | Action |
|---|---|
| `Ctrl-a \|` | split pane horizontally |
| `Ctrl-a -` | split pane vertically |
| `Ctrl-a hjkl` | navigate panes |
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

---

## Tool Aliases Quick Reference

| Alias | Expands to |
|---|---|
| `v` / `vim` / `vi` | `nvim` |
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
