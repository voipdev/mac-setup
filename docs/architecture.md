# Architecture

How all the pieces fit together in this macOS terminal setup.

---

## System Diagram

```
+------------------------------------------------------------------+
|  macOS                                                           |
|                                                                  |
|  +------------------+     +----------------------------------+   |
|  |   AeroSpace      |     |          iTerm2                  |   |
|  |   (window mgr)   |     |  +----------------------------+ |   |
|  |                   |     |  |  Status Bar (bottom)       | |   |
|  |  Alt-hjkl focus   |     |  |  dir | git | clock         | |   |
|  |  Alt-1..5 spaces  |     |  +----------------------------+ |   |
|  +------------------+     |  |                              | |   |
|                            |  |  +------------------------+ | |   |
|                            |  |  |  tmux (multiplexer)    | | |   |
|                            |  |  |  Ctrl-a prefix         | | |   |
|                            |  |  |  +------------------+  | | |   |
|                            |  |  |  |  zsh (shell)     |  | | |   |
|                            |  |  |  |  Starship (❯)    |  | | |   |
|                            |  |  |  |  vi-mode [N]/[I] |  | | |   |
|                            |  |  |  +------------------+  | | |   |
|                            |  |  |  Catppuccin status bar | | |   |
|                            |  |  +------------------------+ | |   |
|                            |  |                              | |   |
|                            |  +------------------------------+ |   |
|                            +----------------------------------+   |
+------------------------------------------------------------------+
```

---

## Layer Stack

```
 Layer 5   AeroSpace          Tiling WM — arranges windows across workspaces
 Layer 4   iTerm2             Terminal emulator — renders everything, status bar
 Layer 3   tmux               Multiplexer — sessions, panes, persists across disconnects
 Layer 2   zsh + plugins      Shell — command execution, completion, history
 Layer 1   CLI tools          Modern Rust replacements (eza, bat, fd, rg, fzf, etc.)
 Layer 0   Homebrew           Package manager — installs and updates everything above
```

---

## Shell Startup Sequence

When a new zsh session starts, `~/.zshrc` executes in this order:

```
1. PATH + Homebrew
   └─ Prepend ~/go/bin, ~/bin, ~/.local/bin
   └─ eval brew shellenv (adds /opt/homebrew/bin)

2. Zsh Options
   └─ History: SHARE_HISTORY, HIST_IGNORE_ALL_DUPS, HIST_IGNORE_SPACE
   └─ Navigation: AUTO_CD, AUTO_PUSHD

3. Completion System
   └─ compinit (regenerated once per day)
   └─ Case-insensitive matching, LS_COLORS integration

4. antidote (plugin manager)
   └─ Loads plugins from ~/.zsh_plugins.txt
   └─ Generates static ~/.zsh_plugins.zsh bundle
   └─ Plugins loaded:
      ├─ zsh-completions        (extra completions)
      ├─ zsh-autosuggestions     (ghost text from history)
      ├─ zsh-syntax-highlighting (command coloring, deferred)
      ├─ zsh-history-substring-search (arrow key search)
      ├─ zsh-you-should-use      (alias reminders)
      ├─ fzf-tab                 (fuzzy completion with previews)
      └─ zsh-autopair            (auto-close brackets/quotes)

5. Tool Activations (each guarded by command -v)
   ├─ mise activate zsh        (runtime version manager)
   ├─ zoxide init zsh          (smart cd, replaces cd command)
   ├─ atuin init zsh           (SQLite history, local-only)
   ├─ direnv hook zsh          (per-directory env vars)
   ├─ fzf key-bindings.zsh     (Ctrl-T, Alt-C)
   └─ iterm2_shell_integration (marks, imgcat)

6. Environment Variables
   ├─ EDITOR/VISUAL = nvim
   ├─ PAGER = bat
   ├─ BAT_THEME = Catppuccin-mocha
   ├─ FZF_DEFAULT_OPTS (Catppuccin colors)
   └─ RIPGREP_CONFIG_PATH

7. Aliases (conditional on tool availability)
   ├─ ls/ll/la/lt → eza (fallback: ls -G)
   ├─ cat → bat, grep → rg, find → fd, ps → procs, top → btm
   ├─ v/vi/vim → nvim
   ├─ npm → bun, pip → uv pip
   └─ git shortcuts (g, gs, gd, ga, gc, gp, gl, lg)

8. Functions
   ├─ mkcd    (mkdir + cd)
   ├─ fcd     (fuzzy directory jump)
   ├─ fe      (fuzzy file → nvim)
   ├─ glog    (fuzzy git log browser)
   ├─ extract (archive extraction, path-traversal hardened)
   └─ serve   (python HTTP server)

9. Key Bindings
   ├─ bindkey -v              (vi mode)
   ├─ KEYTIMEOUT=1            (10ms mode switch)
   ├─ Ctrl-X Ctrl-E           (edit command in nvim)
   └─ Ctrl-A/E                (beginning/end of line in insert mode)

10. Prompt (must be last)
    ├─ Vi-mode indicator       ([N] red / [I] green in RPROMPT)
    └─ Starship                (minimal ❯ prompt)
```

---

## Data Flow: Command History

```
User types command
       │
       v
   zsh HISTFILE (~/.zsh_history)      Local file, 100k lines
       │
       v
   atuin SQLite (~/.local/share/atuin/history.db)
       │
       x  (sync disabled — auto_sync=false, records=false)
       │
   Stays local. No network traffic.
```

---

## Data Flow: Fuzzy Search

```
Ctrl-R  ──>  atuin (full-screen history search)
Ctrl-T  ──>  fzf + fd (file search with bat preview)
Alt-C   ──>  fzf + fd (directory search with eza preview)
Tab     ──>  fzf-tab (completion with context-aware preview)
fcd     ──>  fd + fzf + eza (jump to any dir)
fe      ──>  fzf + bat (find file → open in nvim)
glog    ──>  git log + fzf + bat (browse commits)
```

---

## Tool Replacement Map

```
Traditional          Modern (Rust/Go)       How Aliased
─────────────────────────────────────────────────────────
ls                   eza                    alias ls='eza ...'
cat                  bat                    alias cat='bat ...'
grep                 ripgrep (rg)           alias grep='rg'
find                 fd                     alias find='fd'
cd                   zoxide                 eval "$(zoxide init --cmd cd)"
ps                   procs                  alias ps='procs'
top                  bottom (btm)           alias top='btm'
du                   dust                   (no alias, use dust)
df                   duf                    (no alias, use duf)
sed                  sd                     (no alias, use sd)
diff (git)           delta                  git config core.pager delta
npm                  bun                    alias npm='bun'
pip                  uv                     alias pip='uv pip'
nvm/pyenv            mise                   eval "$(mise activate zsh)"
```

---

## tmux Architecture

```
tmux server
 └─ Session "work"
     ├─ Window 1: editor (nvim)
     ├─ Window 2: shell
     └─ Window 3: logs
         ├─ Pane (top)
         └─ Pane (bottom)

Plugins (via tpm):
 ├─ tmux-sensible       Sensible defaults (larger history, faster keys)
 ├─ catppuccin/tmux     Mocha theme + status bar (dir, session, time)
 ├─ tmux-resurrect      Save/restore sessions across restarts
 └─ tmux-continuum      Auto-save sessions every 15 min

Key bindings (prefix = Ctrl-a):
 ├─ |     split horizontal
 ├─ -     split vertical
 ├─ hjkl  navigate panes (vim-style)
 ├─ c     new window
 ├─ r     reload config
 └─ d     detach
```

---

## Neovim Architecture

```
LazyVim (distribution)
 └─ lazy.nvim (plugin manager)
     ├─ Installs plugins on first launch
     ├─ Pins commits in lazy-lock.json
     └─ mason.nvim downloads LSP servers, formatters, linters

Config: ~/.config/nvim/
 ├─ init.lua            Entry point
 ├─ lua/config/         Keymaps, options, autocmds
 ├─ lua/plugins/        Custom plugin specs (overrides)
 └─ lazy-lock.json      Pinned plugin versions
```

---

## AeroSpace Window Management

```
Workspace Layout:

 [1: Terminal]  [2: Browser]  [3: Code]  [4: Chat]  [5: Misc]
     Alt-1         Alt-2        Alt-3       Alt-4      Alt-5

Within each workspace:
 ┌──────────┬──────────┐
 │          │          │    BSP (binary space partition) tiling
 │  window  │  window  │    8px gaps between windows
 │          │          │
 └──────────┴──────────┘

 Alt-hjkl         focus window
 Alt-Shift-hjkl   move window
 Alt-f            fullscreen toggle
 Alt-Shift-f      float/tile toggle
```

---

## Color Theme: Catppuccin Mocha

Applied consistently across all layers:

```
Tool                Config Location                     How Applied
──────────────────────────────────────────────────────────────────────
iTerm2              GUI > Profiles > Colors              catppuccin-mocha.itermcolors
bat                 BAT_THEME env var                    "Catppuccin-mocha"
fzf                 FZF_DEFAULT_OPTS --color             Mocha hex values
delta (git diff)    git config delta.syntax-theme        "Catppuccin-mocha"
tmux                @catppuccin_flavour                  "mocha"
Starship            palette in starship.toml             catppuccin_mocha palette
Neovim              LazyVim default                      Catppuccin plugin
```

Base colors:
```
Background  #1e1e2e (base)      Foreground  #cdd6f4 (text)
Surface     #313244 (surface0)  Red         #f38ba8
Green       #a6e3a1             Yellow      #f9e2af
Blue        #89b4fa             Purple      #cba6f7
Teal        #94e2d5             Peach       #fab387
```

---

## File Locations

```
Config Files:
~/.zshrc                           Shell config
~/.zsh_plugins.txt                 Antidote plugin list
~/.config/starship.toml            Prompt (minimal ❯)
~/.tmux.conf                       tmux config
~/.config/aerospace/aerospace.toml AeroSpace WM
~/.config/ripgrep/config           ripgrep defaults
~/.config/atuin/config.toml        Atuin (local-only history)
~/.config/nvim/                    Neovim / LazyVim
~/.claude/CLAUDE.md                Claude Code global instructions

Data Files:
~/.zsh_history                     zsh history (100k lines)
~/.local/share/atuin/history.db    Atuin SQLite database
~/.local/share/atuin/key           Atuin encryption key (back up!)
~/.tmux/resurrect/                 tmux session snapshots
~/.tmux/plugins/                   tmux plugins (via tpm)
~/.zsh_plugins.zsh                 Antidote compiled bundle (auto-generated)
~/.zcompdump                       Completion cache (regenerated daily)

This Repo:
~/Stash/repos/mac-setup/
 ├─ README.md                      Step-by-step setup guide
 ├─ Brewfile                       All Homebrew packages
 ├─ configs/                       Config file copies (for new machines)
 │   ├─ zshrc
 │   ├─ zsh_plugins.txt
 │   ├─ starship.toml
 │   ├─ tmux.conf
 │   ├─ aerospace.toml
 │   ├─ ripgrep.conf
 │   └─ CLAUDE.md
 └─ docs/
     └─ architecture.md            This file
```

---

## Security Model

```
Network Exposure:
 ├─ atuin sync           DISABLED (auto_sync=false, records=false)
 ├─ atuin update checks  DISABLED (update_check=false)
 ├─ Homebrew             Fetches packages over HTTPS, verified with SHA256
 ├─ antidote             git clone over HTTPS (no signature verification)
 ├─ tpm                  git clone over HTTPS (no signature verification)
 └─ lazy.nvim            git clone over HTTPS (commits pinned in lockfile)

Local Hardening:
 ├─ ~/.local/bin          chmod 700 (prevents other users writing to PATH)
 ├─ HIST_IGNORE_SPACE     Commands prefixed with space not saved to history
 ├─ secrets_filter        Atuin filters AWS keys, GitHub PATs, Slack tokens
 ├─ extract()             Hardened against path traversal (--no-absolute-filenames)
 ├─ direnv                Requires explicit `direnv allow` per .envrc
 ├─ Conditional aliases   All tool aliases guarded by `command -v` checks
 └─ git-credential-mgr    Credentials stored in macOS Keychain (encrypted)

Supply Chain:
 ├─ zsh plugins           From zsh-users org (well-maintained, high-star)
 ├─ tmux plugins          From tmux-plugins org
 ├─ Neovim plugins        Commits pinned in lazy-lock.json
 └─ Brewfile              Declarative — review diffs before `brew bundle`
```

---

## Dotfile Management: chezmoi

```
chezmoi init
 └─ ~/.local/share/chezmoi/    (git repo tracking dotfiles)
     ├─ Tracks: zshrc, zsh_plugins.txt, starship.toml,
     │          tmux.conf, ripgrep config, aerospace.toml,
     │          CLAUDE.md
     ├─ Supports: templates, per-machine config, age encryption
     └─ Sync: push to remote git repo, pull on new machine

Workflow:
 chezmoi add ~/.zshrc          Track a file
 chezmoi edit ~/.zshrc         Edit tracked version
 chezmoi diff                  Show pending changes
 chezmoi apply                 Apply tracked configs to system
 chezmoi update                Pull from remote + apply
```
