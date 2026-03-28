# Terminal Workflow Guide

How to use every feature in this setup to work fast. Organized by task, not by tool.

---

## Table of Contents

1. [Navigating the Filesystem](#1-navigating-the-filesystem)
2. [Finding Files and Content](#2-finding-files-and-content)
3. [Reading and Inspecting Files](#3-reading-and-inspecting-files)
4. [Editing with Neovim](#4-editing-with-neovim)
5. [Using the Terminal as an IDE](#5-using-the-terminal-as-an-ide)
6. [tmux: Sessions, Windows, and Panes](#6-tmux-sessions-windows-and-panes)
7. [Git Workflow](#7-git-workflow)
8. [GitHub Workflow with gh](#8-github-workflow-with-gh)
9. [Shell History and Command Recall](#9-shell-history-and-command-recall)
10. [Shell Efficiency: Vi-Mode and Shortcuts](#10-shell-efficiency-vi-mode-and-shortcuts)
11. [Shell Intelligence: What Happens as You Type](#11-shell-intelligence-what-happens-as-you-type)
12. [Window Management with AeroSpace](#12-window-management-with-aerospace)
13. [Working with Multiple Projects](#13-working-with-multiple-projects)
14. [Monitoring and Debugging](#14-monitoring-and-debugging)
15. [iTerm2 Features](#15-iterm2-features)
16. [Text Processing and Data Wrangling](#16-text-processing-and-data-wrangling)
17. [Learning and Help](#17-learning-and-help)
18. [Customizing and Extending](#18-customizing-and-extending)
19. [Quick Reference Card](#19-quick-reference-card)

---

## 1. Navigating the Filesystem

### Smart cd with zoxide

zoxide learns which directories you visit most. It replaces `cd` entirely.

```bash
cd repos            # works like normal cd
cd mac-setup        # jumps to ~/Stash/repos/mac-setup if you've been there before
cd mac              # partial match — zoxide picks the best match
cd -                # go back to previous directory
```

zoxide ranks by frequency and recency. The more you visit a directory, the higher it ranks. After a few days of use, you can jump anywhere with 2-3 characters.

### Fuzzy directory jump (fcd)

When you don't remember the path at all:

```bash
fcd
```

This opens fzf with every directory on screen. Type to filter, arrow keys to navigate, Enter to jump. The preview pane shows the directory tree.

### Quick navigation aliases

```bash
..          # cd ..
...         # cd ../..
....        # cd ../../..
```

### Directory stack

zsh tracks your recent directories in a stack (enabled by `AUTO_PUSHD`):

```bash
dirs -v     # show numbered directory stack
cd -2       # jump to 2nd entry in the stack
```

### Create and enter a directory

```bash
mkcd my-new-project     # mkdir -p + cd in one command
```

### Listing files

```bash
ls          # eza with icons, directories first
ll          # long format with permissions, size, dates
la          # include hidden files
lt          # tree view (2 levels deep)
l           # long format + hidden files (everything)
tree        # colorized tree output (full depth)
```

All listing commands show Nerd Font icons for file types (folders, code files, images, etc.) thanks to JetBrains Mono Nerd Font.

---

## 2. Finding Files and Content

### Find a file by name (Ctrl-T)

Press `Ctrl-T` anywhere in your command line. fzf opens with a file search powered by fd. Type part of the filename, select it, and it gets inserted into your current command.

```bash
vim <Ctrl-T>        # search → select → vim opens that file
cat <Ctrl-T>        # search → select → cat shows contents
```

The preview pane shows file contents (syntax highlighted by bat).

### Find a file and edit it (fe)

```bash
fe
```

Opens fzf, search for any file, press Enter, and it opens directly in nvim. One command, zero typing of paths.

### Search file contents with ripgrep

```bash
grep "TODO"                    # search current dir recursively
grep "function.*export" -t js  # search only JS files
grep "password" -l             # list filenames only
grep "error" -C 3              # show 3 lines of context
grep "TODO" --no-hidden        # skip hidden files
grep -i "readme"               # case-insensitive
```

Remember: `grep` is aliased to `rg` (ripgrep). It's case-smart by default (case-insensitive unless you use uppercase). Hidden files are searched, `.git` and `node_modules` are excluded (configured in `~/.config/ripgrep/config`).

### Find files by name with fd

```bash
find "*.toml"           # find all TOML files
find -e py              # find all Python files
find -t d "config"      # find directories named config
find -H                 # include hidden files
find "test" -x rm       # find and delete (careful!)
```

Remember: `find` is aliased to `fd`. Much faster than GNU find.

### cd into a directory (Alt-C)

Press `Alt-C` to fuzzy-search directories and cd into the selected one. Preview shows the directory tree.

---

## 3. Reading and Inspecting Files

### Syntax-highlighted file viewing

```bash
cat file.py                 # bat: syntax highlighted, line numbers
cat -p file.py              # bat: plain mode (no line numbers, no paging)
cat -l json data.txt        # bat: force language for highlighting
cat -r 10:20 file.py        # bat: show only lines 10-20
cat -A file.py              # bat: show non-printable characters
cat --diff file.py          # bat: show git changes highlighted
```

`cat` is aliased to `bat`. If you ever need the real cat: `/bin/cat`.

### Colored man pages

Man pages are rendered through bat with syntax highlighting:

```bash
man git-rebase              # full-color, searchable man page
man tmux                    # navigate with / to search, q to quit
```

This is powered by `MANPAGER="sh -c 'col -bx | bat -l man -p'"`.

### Watching command output

```bash
watch -n 2 'ls -la'             # repeat every 2 seconds
watch -d 'git status'           # highlight differences between runs
watch -n 1 'curl -s localhost:3000/health'  # monitor an endpoint
```

---

## 4. Editing with Neovim

### Opening files

```bash
v                       # open nvim (empty)
v file.py               # open a specific file
v .                     # open nvim in current directory (file explorer)
vimdiff file1 file2     # side-by-side diff (alias for nvim -d)
fe                      # fuzzy find → open in nvim
```

### LazyVim essentials

LazyVim pre-configures everything. These are the key bindings you'll use daily:

#### File navigation
| Key | Action |
|---|---|
| `Space` | Leader key (opens which-key menu — just wait for hints) |
| `Space f f` | Find file (fuzzy, like Cmd-P in VS Code) |
| `Space f g` | Live grep across project (like Cmd-Shift-F) |
| `Space f r` | Recent files |
| `Space f b` | Open buffers |
| `Space e` | File explorer (neo-tree) sidebar toggle |

#### Code intelligence (LSP)
| Key | Action |
|---|---|
| `gd` | Go to definition |
| `gr` | Go to references |
| `gI` | Go to implementation |
| `K` | Hover documentation |
| `Space c a` | Code actions (quick fixes, refactors) |
| `Space c r` | Rename symbol (project-wide) |
| `]d` / `[d` | Next/previous diagnostic (error, warning) |
| `Space x x` | Toggle diagnostics list |

#### Editing
| Key | Action |
|---|---|
| `gcc` | Toggle comment current line |
| `gc` (visual) | Toggle comment selection |
| `sa` + motion + char | Surround add (e.g., `saiw"` surrounds word with quotes) |
| `sd` + char | Surround delete |
| `sr` + old + new | Surround replace |
| `Space c f` | Format document |

#### Windows and buffers
| Key | Action |
|---|---|
| `Space b d` | Close current buffer |
| `Space b b` | Switch buffer (fuzzy) |
| `Ctrl-h/j/k/l` | Navigate between splits |
| `Space w -` | Split horizontal |
| `Space w \|` | Split vertical |
| `Space w d` | Close split |

#### Terminal in Neovim
| Key | Action |
|---|---|
| `Space f t` | Open floating terminal |
| `Ctrl-/` | Toggle terminal (bottom panel) |

#### LazyVim extras

Enable additional features via `Space` > `Extras`:
- **lang.python** — Python LSP, formatting, debugging
- **lang.typescript** — TypeScript LSP, prettier
- **lang.go** — Go LSP, test runner
- **lang.rust** — Rust analyzer
- **editor.mini-files** — minimal file explorer
- **ui.mini-animate** — smooth scrolling animations

### Neovim as an IDE

To get full IDE features for a language:

```bash
# 1. Open nvim in a project with the language
cd my-python-project && v .

# 2. LazyVim auto-detects the language and prompts to install LSP
#    Or manually: :Mason to browse and install servers

# 3. LSP provides:
#    - Autocomplete (shows automatically, Tab to accept)
#    - Go to definition (gd)
#    - Find references (gr)
#    - Rename across project (Space c r)
#    - Error diagnostics inline
#    - Format on save (Space c f to format manually)
```

---

## 5. Using the Terminal as an IDE

The full IDE setup: tmux for layout, nvim for editing, fzf for navigation.

### Recommended layout

```
┌──────────────────────────────────────┐
│  tmux window 1: "editor"             │
│  ┌────────────────────┬─────────────┐│
│  │                    │             ││
│  │  nvim (main file)  │  nvim       ││
│  │                    │  (test/ref) ││
│  │                    │             ││
│  ├────────────────────┴─────────────┤│
│  │  shell (run commands, tests)     ││
│  └──────────────────────────────────┘│
│  tmux window 2: "server"             │
│  tmux window 3: "logs"               │
└──────────────────────────────────────┘
```

### Building this layout

```bash
# Start a named session
tn work                    # alias for: tmux new-session -s work

# You're in window 1 (editor)
v .                        # open nvim

# Split a pane below for shell commands
Ctrl-a -                   # horizontal split (bottom pane)

# Split right pane for reference file
Ctrl-a |                   # vertical split (right pane)
v tests/test_main.py       # open test file

# Navigate between panes
Ctrl-a h/j/k/l             # vim-style movement

# Create more windows
Ctrl-a c                   # new window (for running servers, etc.)

# Name the window
Ctrl-a ,                   # rename current window
```

### Workflow patterns

**Edit-Test-Fix cycle:**
1. Edit in nvim (top-left pane)
2. `Ctrl-a j` to jump to bottom shell pane
3. Run tests: `bun test` or `python -m pytest`
4. `Ctrl-a k` back to nvim
5. Jump to error: `]d` in nvim

**Multi-file editing:**
1. `Space f f` to fuzzy-find and open files in nvim
2. `Space b b` to switch between open buffers
3. `:vsplit` or `Space w |` for side-by-side editing

**Quick terminal from nvim:**
- `Ctrl-/` toggles a terminal panel at the bottom of nvim
- `Space f t` opens a floating terminal overlay
- Run quick commands without leaving nvim

**Monitoring a server while coding:**
1. `Ctrl-a c` — new tmux window
2. `bun run dev` — start dev server
3. `Ctrl-a 1` — back to editor window
4. `Ctrl-a 2` — check server output anytime

---

## 6. tmux: Sessions, Windows, and Panes

### Core concepts

- **Session** — a workspace (e.g., "work", "personal"). Persists even when you close the terminal.
- **Window** — a tab within a session. Shown in the status bar.
- **Pane** — a split within a window.

### Session management

```bash
tn work             # new session named "work"
tn personal         # new session named "personal"
ta                  # attach to last session (alias: tmux attach)
tl                  # list sessions (alias: tmux ls)
tk work             # kill session "work" (alias: tmux kill-session -t work)
```

Inside tmux:
| Key | Action |
|---|---|
| `Ctrl-a d` | Detach (session keeps running in background) |
| `Ctrl-a s` | Switch between sessions (interactive picker) |
| `Ctrl-a $` | Rename current session |

### Window management

| Key | Action |
|---|---|
| `Ctrl-a c` | New window |
| `Ctrl-a ,` | Rename window |
| `Ctrl-a n` | Next window |
| `Ctrl-a p` | Previous window |
| `Ctrl-a 1-9` | Jump to window by number |
| `Ctrl-a w` | Window picker (interactive) |
| `Ctrl-a &` | Close window |

### Pane management

| Key | Action |
|---|---|
| `Ctrl-a \|` | Split vertically (left/right) |
| `Ctrl-a -` | Split horizontally (top/bottom) |
| `Ctrl-a h/j/k/l` | Navigate panes (vim-style) |
| `Ctrl-a z` | Zoom pane (fullscreen toggle — press again to unzoom) |
| `Ctrl-a x` | Close pane |
| `Ctrl-a Space` | Cycle through pane layouts |
| `Ctrl-a q` | Show pane numbers (press number to jump) |

### Resizing panes

Hold `Ctrl-a` then press arrow keys to resize.

### Copy mode (scrollback)

| Key | Action |
|---|---|
| `Ctrl-a [` | Enter copy mode (scroll with vim keys) |
| `q` | Exit copy mode |
| `/` | Search forward in scrollback |
| `?` | Search backward in scrollback |
| `Space` | Start selection (vim visual mode) |
| `Enter` | Copy selection |
| `Ctrl-a ]` | Paste |

Mouse scrolling also enters copy mode automatically (mouse is enabled).

### Session persistence

Your setup includes tmux-resurrect and tmux-continuum:
- Sessions are auto-saved every 15 minutes
- After a restart, `ta` reattaches and restores your windows, panes, and working directories
- Pane contents are also restored (`@resurrect-capture-pane-contents on`)

---

## 7. Git Workflow

### Quick status and diffs

```bash
gs          # git status
gd          # git diff (uses delta: side-by-side, syntax highlighted)
glg         # git log --oneline --graph --decorate
```

### delta (diff viewer)

All git diffs automatically use delta:
- Side-by-side view with syntax highlighting
- Line numbers
- Navigate with `n` (next file) and `N` (previous file)
- `q` to quit

### Staging and committing

```bash
ga file.py          # git add file.py
ga .                # git add everything
gc                  # git commit (opens nvim for message)
gc -m "fix bug"     # commit with inline message
gp                  # git push
gl                  # git pull (with rebase, configured globally)
```

### Branching

```bash
gco main            # git checkout main
gcb feature/auth    # git checkout -b feature/auth (create + switch)
```

### lazygit (lg)

The most powerful git tool in this setup. Run `lg` to open a full terminal UI:

```bash
lg
```

**lazygit panels** (navigate with Tab or h/l):
1. **Status** — current branch, ahead/behind
2. **Files** — staged/unstaged changes (`Space` to stage/unstage, `a` to stage all)
3. **Branches** — switch, merge, rebase (Enter to checkout)
4. **Commits** — browse history, amend, reword, squash
5. **Stash** — stash and pop changes

**Common lazygit workflows:**

Interactive rebase:
1. Tab to Commits panel
2. Navigate to the commit you want to edit
3. Press `e` to start interactive rebase
4. `s` to squash, `r` to reword, `d` to drop
5. Continue with `m`

Cherry-pick:
1. Navigate to commit, press `C` to copy
2. Switch to target branch
3. Press `V` to paste (cherry-pick)

Resolve merge conflicts:
1. Files panel shows conflicted files
2. Enter a file to see conflicts inline
3. Choose left/right/both for each conflict

Stage individual hunks:
1. Select a file in the Files panel
2. Press Enter to see the diff
3. Select specific lines/hunks to stage

### Fuzzy git log (glog)

```bash
glog
```

Opens an interactive commit browser. Search commits with fzf, preview shows the full diff. Press Enter to view the complete commit in bat.

### Rewriting git history

For heavy history rewrites (removing files, rewriting author):

```bash
git-filter-repo --path secret.env --invert-paths    # remove a file from all history
git-filter-repo --mailmap mailmap.txt                # rewrite author/email
```

---

## 8. GitHub Workflow with gh

The GitHub CLI lets you manage PRs, issues, and repos without leaving the terminal.

### Pull requests

```bash
# Create a PR
gh pr create                          # interactive
gh pr create --title "Add auth" --body "..."  # inline

# List and view PRs
gh pr list                            # all open PRs
gh pr view 42                         # view PR #42
gh pr view --web                      # open current PR in browser

# Review and merge
gh pr checkout 42                     # fetch and switch to PR branch
gh pr diff 42                         # view PR diff (uses delta)
gh pr review 42 --approve             # approve
gh pr merge 42 --squash               # squash and merge

# Check CI status
gh pr checks                          # show CI status for current branch
gh run list                           # list recent workflow runs
gh run view 12345 --log               # view run logs
```

### Issues

```bash
gh issue list                         # list open issues
gh issue create --title "Bug: ..."    # create an issue
gh issue view 10                      # view issue #10
gh issue close 10                     # close issue
```

### Repos

```bash
gh repo clone owner/repo              # clone a repo
gh repo create my-project --private   # create new repo
gh repo view --web                    # open current repo in browser
gh browse                             # open repo in browser
```

### Notifications

```bash
gh api notifications                  # check notifications
```

---

## 9. Shell History and Command Recall

### atuin (Ctrl-R)

Press `Ctrl-R` to open atuin's full-screen history search:

```
┌─────────────────────────────────────────┐
│  Search: docker compose                 │
│                                         │
│  > docker compose up -d          2s ago │
│    docker compose logs -f api    5m ago │
│    docker compose down --volumes 1h ago │
│    docker compose build          2d ago │
│                                         │
│  [host: macbook] [dir: ~/project]       │
└─────────────────────────────────────────┘
```

Features:
- **Fuzzy search** across all history
- **Context** — shows which directory and host the command was run in
- **Duration** — shows how long each command took
- **Exit code** — filter by commands that succeeded or failed
- **Filter modes** — press `Ctrl-R` again to cycle: global > host > session > directory

Filter shortcuts in atuin search:
| Key | Action |
|---|---|
| Type | Filter by content |
| `Ctrl-R` | Cycle filter mode |
| `Up/Down` | Navigate results |
| `Enter` | Execute selected command |
| `Tab` | Put command in prompt for editing first |
| `Esc` | Cancel |

atuin history is local-only in this setup (sync disabled). Data lives in `~/.local/share/atuin/history.db`.

### Autosuggestions

As you type, ghost text appears (from history). Accept it with:
- `Ctrl-Space` — accept full suggestion
- `Right arrow` — accept one character
- Just keep typing to ignore it

### History substring search

After typing some text:
- `Up arrow` — search history for commands containing that text
- `Down arrow` — search forward
- `Ctrl-P` / `Ctrl-N` — same thing

Example: type `docker` then press Up to cycle through all docker commands.

### Sensitive commands

Prefix any command with a space to exclude it from history:

```bash
 export API_KEY=secret123     # note the leading space — not saved
```

This works because `HIST_IGNORE_SPACE` is enabled. atuin also auto-filters patterns that look like AWS keys, GitHub tokens, and Slack tokens (`secrets_filter = true`).

---

## 10. Shell Efficiency: Vi-Mode and Shortcuts

### Vi-mode basics

Your shell runs in vi insert mode by default. You type normally. Press `Esc` to enter normal mode for powerful editing:

**Insert mode** (default — you're typing):
| Key | Action |
|---|---|
| `Ctrl-Space` | Accept autosuggestion |
| `Ctrl-A` | Jump to beginning of line |
| `Ctrl-E` | Jump to end of line |
| `Ctrl-R` | History search (atuin) |
| `Ctrl-T` | Find file (fzf) |
| `Alt-C` | Find directory (fzf) |
| `Ctrl-X Ctrl-E` | Edit current command in nvim |
| `Tab` | Fuzzy completion with preview |

**Normal mode** (press `Esc` first):
| Key | Action |
|---|---|
| `h/l` | Move left/right |
| `w/b` | Jump word forward/backward |
| `0` / `$` | Beginning/end of line |
| `x` | Delete character |
| `dw` | Delete word |
| `dd` | Delete entire line |
| `cw` | Change word (delete + insert mode) |
| `ci"` | Change inside quotes |
| `ca(` | Change around parentheses |
| `p` | Paste |
| `u` | Undo |
| `i` | Back to insert mode |
| `a` | Insert after cursor |
| `A` | Insert at end of line |
| `I` | Insert at beginning of line |

The mode indicator shows in the right prompt: `[I]` green = insert, `[N]` red = normal. Mode switching is near-instant (`KEYTIMEOUT=1`, 10ms).

### Edit long commands in nvim

For complex multi-line commands, press `Ctrl-X Ctrl-E`. Your current command opens in nvim. Edit with full vim powers, save and quit (`:wq`), and the command executes.

This is especially useful for:
- Long piped commands
- Commands with complex quoting
- Building multi-line scripts inline

### Tab completion with previews

Press `Tab` for fzf-powered completion (via fzf-tab):
- File paths show bat-highlighted content preview
- Directories show tree preview
- Switch between completion groups with `,` and `.`
- Works for everything: commands, paths, git branches, env vars, etc.

---

## 11. Shell Intelligence: What Happens as You Type

Several plugins work together to make the shell smart:

### Syntax highlighting (zsh-syntax-highlighting)

Commands are colored as you type:
- **Green** — valid command
- **Red** — invalid command (typo, not installed)
- **Underline** — valid file path
- **Bold** — builtins and aliases

You can see typos before pressing Enter.

### Auto-pairing (zsh-autopair)

Brackets and quotes auto-close:
- Type `(` → get `()`  with cursor between
- Type `"` → get `""`
- Type `{` → get `{}`
- Backspace on opening char deletes both

### Alias reminders (you-should-use)

When you type a command that has an alias:

```
$ git status
Found existing alias for "git status". You can use "gs" instead.
```

Over time, you internalize the aliases and get faster.

### Glob features (zsh)

zsh's extended globbing is enabled (`EXTENDED_GLOB`):
```bash
ls **/*.py              # recursive glob — all Python files
ls *(.)                 # only regular files (no directories)
ls *(/)                 # only directories
ls *(.om[1,5])          # 5 most recently modified files
```

`AUTO_CD` is also on — type a directory name without `cd`:
```bash
~/Stash/repos           # same as: cd ~/Stash/repos
..                      # same as: cd ..
```

---

## 12. Window Management with AeroSpace

AeroSpace tiles windows automatically. No dragging, no resizing with a mouse.

### Focus windows

| Key | Action |
|---|---|
| `Alt-h` | Focus window to the left |
| `Alt-j` | Focus window below |
| `Alt-k` | Focus window above |
| `Alt-l` | Focus window to the right |

### Move windows

| Key | Action |
|---|---|
| `Alt-Shift-h` | Move window left |
| `Alt-Shift-j` | Move window down |
| `Alt-Shift-k` | Move window up |
| `Alt-Shift-l` | Move window right |

### Workspaces

| Key | Action |
|---|---|
| `Alt-1` through `Alt-5` | Switch to workspace |
| `Alt-Shift-1` through `Alt-Shift-5` | Move window to workspace |

**Recommended workspace layout:**
```
1: Terminal (iTerm2)
2: Browser (research, docs)
3: Code (secondary editor, if needed)
4: Communication (Slack, email)
5: Misc (Finder, notes, music)
```

### Layout controls

| Key | Action |
|---|---|
| `Alt-f` | Toggle fullscreen |
| `Alt-Shift-f` | Toggle floating/tiling |
| `Alt-,` | Toggle accordion layout (stacked) |
| `Alt-/` | Toggle tiles layout (side-by-side) |
| `Alt-Shift-minus` | Shrink window |
| `Alt-Shift-equal` | Grow window |
| `Alt-Shift-r` | Reload AeroSpace config |

### Typical workflow

1. `Alt-1` — jump to terminal workspace
2. Open iTerm2, start tmux session
3. `Alt-2` — jump to browser for docs
4. `Alt-Shift-1` — send browser to terminal workspace (side by side)
5. `Alt-h/l` — switch focus between terminal and browser
6. `Alt-f` — fullscreen terminal when you need focus

---

## 13. Working with Multiple Projects

### mise: Runtime versions per project

mise auto-switches Node, Python, Go versions when you cd into a project.

```bash
# Set global defaults
mise use --global node@lts
mise use --global python@3.13

# Set project-specific versions
cd ~/Stash/repos/my-project
mise use node@20          # creates .mise.toml in project
mise use python@3.11      # old project needs older Python

# Check what's active
mise ls                   # list installed runtimes
mise current              # show active versions for current dir
```

When you `cd` into a project with `.mise.toml`, mise automatically activates the right versions. No manual `nvm use` needed.

### direnv: Per-project environment variables

```bash
cd ~/Stash/repos/my-project

# Create .envrc
echo 'export DATABASE_URL="postgres://localhost/mydb"' > .envrc
echo 'export API_KEY="dev-key-123"' >> .envrc

# Approve it (one-time, after reading the file)
direnv allow

# Now every time you cd into this directory, those vars are set
# When you cd out, they're unset automatically
```

**Security note:** Always read `.envrc` before running `direnv allow`, especially in cloned repos.

### Package managers

```bash
# JavaScript (bun replaces npm)
npm init                # actually runs: bun init
npm install express     # actually runs: bun install express
npm run dev             # actually runs: bun run dev
npm test                # actually runs: bun test

# Python (uv replaces pip)
pip install flask       # actually runs: uv pip install flask
pip install -r req.txt  # actually runs: uv pip install -r req.txt

# For virtual environments, use uv directly:
uv venv                 # create .venv
source .venv/bin/activate
uv pip install -e ".[dev]"
```

### Chezmoi: Keeping configs in sync

```bash
# After editing a config file
cza ~/.zshrc            # chezmoi add — start tracking it

# Made a change to a tracked file?
czd                     # chezmoi diff — see what changed

# Edit the tracked version
cze ~/.zshrc            # chezmoi edit — edit in chezmoi's repo

# Apply changes
chezmoi apply           # apply all tracked configs

# On a new machine
chezmoi init <github-user>
chezmoi apply           # all configs restored
```

---

## 14. Monitoring and Debugging

### System resources

```bash
top                     # bottom (btm): interactive system monitor
                        # Press ? for help, / to search processes
                        # Tab to switch between CPU, memory, network, disk
ps                      # procs: colorized process list
ps --tree               # procs: process tree view
```

### Disk usage

```bash
dust                    # visual disk usage (like du but better)
dust -d 2               # limit depth to 2 levels
duf                     # disk free (like df but readable)
```

### Network

```bash
ip                      # your local IP (alias for ipconfig getifaddr en0)
ports                   # list all listening ports
```

### Quick HTTP server

```bash
serve                   # start HTTP server on port 8000 in current dir
serve 3000              # start on port 3000
```

### Extracting archives

```bash
extract file.tar.gz             # auto-detects format, extracts safely
extract file.zip output/        # extract to specific directory
```

Supports: `.tar.gz`, `.tar.bz2`, `.tar.xz`, `.zip`, `.gz`, `.bz2`, `.7z`. Hardened against path traversal attacks.

### Watching for changes

```bash
watch -n 2 'kubectl get pods'   # repeat every 2 seconds
watch -d 'ls -la'               # highlight differences between runs
```

---

## 15. iTerm2 Features

### Status bar (bottom of terminal)

The iTerm2 status bar is pinned at the bottom, always visible. It shows:
- **Current directory** — updates as you cd
- **Git state** — branch name and dirty/clean status
- **Clock** — current time

This is why the Starship prompt is just `❯` — all context info lives in the status bar.

### Shell integration

iTerm2 shell integration (sourced from `~/.iterm2_shell_integration.zsh`) provides:

**imgcat — view images in the terminal:**
```bash
imgcat photo.png                # display image inline
imgcat -w 50% screenshot.png    # resize to 50% width
```

**it2copy / it2paste — clipboard over SSH:**
```bash
echo "text" | it2copy          # copy to macOS clipboard (even over SSH)
it2paste                        # paste from clipboard
```

**Marks and navigation:**
- Each command prompt gets a mark (blue triangle)
- `Cmd-Shift-Up/Down` — jump between command prompts
- Useful for scrolling through long output

**Command status:**
- Shell integration marks commands that failed with a red indicator

### Useful iTerm2 keyboard shortcuts

| Key | Action |
|---|---|
| `Cmd-T` | New tab |
| `Cmd-N` | New window |
| `Cmd-D` | Split pane right |
| `Cmd-Shift-D` | Split pane below |
| `Cmd-[` / `Cmd-]` | Switch panes |
| `Cmd-Shift-Up/Down` | Jump between shell marks |
| `Cmd-F` | Find in terminal |
| `Cmd-Shift-H` | Paste history |
| `Cmd-;` | Autocomplete from scrollback |
| `Cmd-Option-E` | Expose all tabs (search across windows) |

---

## 16. Text Processing and Data Wrangling

### JSON with jq

```bash
jq '.' data.json                        # pretty-print
jq '.name' package.json                 # extract a field
jq '.scripts | keys' package.json       # list keys
jq '.items[] | select(.active)' data.json  # filter array
jq -r '.url' config.json                # raw output (no quotes)
cat api-response.json | jq '.'          # pipe into jq (bat + jq)
```

### YAML with yq

```bash
yq '.services' docker-compose.yml       # query YAML
yq -i '.version = "3.9"' compose.yml    # edit in place
yq eval-all 'select(.kind == "Deployment")' k8s.yml  # filter
```

### Stream editing with sd

`sd` is a modern sed replacement with simpler syntax:

```bash
sd 'old' 'new' file.txt                 # replace in file
sd 'error' 'warning' *.log              # replace across files
sd '\bfoo\b' 'bar' file.txt             # regex word boundary
echo "hello world" | sd 'world' 'there' # pipe mode
sd -F 'exact.match' 'replacement' f.txt # fixed string (no regex)
```

Key difference from sed: `sd` uses normal regex syntax (no escaping needed for `(`, `{`, etc.).

---

## 17. Learning and Help

### tldr — quick command references

Concise, practical examples for any command:

```bash
tldr tar                # show common tar examples
tldr git rebase         # show rebase examples
tldr ffmpeg             # quick reference, not the full man page
```

tldr is like a cheat sheet — shows the 5-10 most common uses. Use `man` when you need the full reference.

### Neovim help

```
:help                   # general help
:Lazy                   # plugin manager UI
:Mason                  # LSP server manager
:checkhealth            # diagnose issues
:Telescope help_tags    # fuzzy-search help topics
```

### which-key (Neovim)

In nvim, press `Space` and wait — which-key shows all available bindings organized by category. This is the best way to discover features.

---

## 18. Customizing and Extending

### Adding a new Homebrew package

```bash
bri ripgrep             # brew install
# Then add to Brewfile so it's tracked:
echo 'brew "ripgrep"' >> ~/Stash/repos/mac-setup/Brewfile
```

### Adding a zsh plugin

Edit `~/.zsh_plugins.txt`:
```
github-user/plugin-name
```

Then reload:
```bash
sz                      # source ~/.zshrc (antidote reloads plugins)
```

### Adding a tmux plugin

Edit `~/.tmux.conf`:
```bash
set -g @plugin 'github-user/plugin-name'
```

Then inside tmux: `Ctrl-a I` (capital I) to install.

### Adding a Neovim plugin

Create a file in `~/.config/nvim/lua/plugins/`:
```lua
-- ~/.config/nvim/lua/plugins/my-plugin.lua
return {
  "github-user/plugin-name",
  opts = {},
}
```

Restart nvim — lazy.nvim auto-installs it.

### Updating everything

```bash
# Homebrew (all packages)
bru                     # brew upgrade

# Neovim plugins (inside nvim)
:Lazy update            # update all plugins

# tmux plugins (inside tmux)
Ctrl-a U                # capital U — update plugins

# Antidote (zsh plugins)
antidote update         # update all plugins

# mise runtimes
mise upgrade            # upgrade installed runtimes
```

---

## 19. Quick Reference Card

### Most-Used Shortcuts

```
Shell:
  Ctrl-R          Search history (atuin)
  Ctrl-T          Find file (fzf)
  Alt-C           Find directory (fzf)
  Ctrl-Space      Accept autosuggestion
  Ctrl-X Ctrl-E   Edit command in nvim
  Tab             Fuzzy completion with preview
  Esc             Vi normal mode
  Up/Down         History substring search

tmux (prefix = Ctrl-a):
  |               Split vertical
  -               Split horizontal
  h/j/k/l         Navigate panes
  z               Zoom pane (toggle)
  c               New window
  1-9             Switch window
  w               Window picker
  s               Session picker
  d               Detach
  r               Reload config
  [               Copy mode (vim keys to scroll)

Neovim (leader = Space):
  Space f f       Find file
  Space f g       Live grep
  Space e         File explorer
  gd              Go to definition
  gr              Go to references
  K               Hover docs
  Space c a       Code actions
  Space c r       Rename symbol
  gcc             Toggle comment
  ]d / [d         Next/prev diagnostic
  Ctrl-/          Toggle terminal

AeroSpace:
  Alt-h/j/k/l        Focus window
  Alt-Shift-h/j/k/l  Move window
  Alt-1..5            Switch workspace
  Alt-Shift-1..5      Move to workspace
  Alt-f               Fullscreen
  Alt-Shift-f         Float toggle

lazygit (inside lg):
  Space           Stage/unstage file
  a               Stage all
  c               Commit
  P               Push
  p               Pull
  e               Interactive rebase
  s               Squash
  r               Reword commit
  ?               Show all keybindings
```

### All Aliases

```
Navigation:     ..  ...  ....
Files:          ls  ll  la  lt  l  cat  tree
Tools:          grep  find  ps  top
Editor:         v  vi  vim  vimdiff  fe  fcd
Git:            g  gs  gd  ga  gc  gp  gl  gco  gcb  glg  lg  glog
tmux:           ta  tl  tn  tk
Homebrew:       bri  brs  bru  brl
chezmoi:        cz  cza  czd  cze  czu
Python:         python  pip
JavaScript:     npm
Shell:          sz  zrc  reload  path
System:         ip  ports  serve  extract  mkcd
```
