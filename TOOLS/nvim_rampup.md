---
layout: default
title: NeoVim Ramp-up
---

# NVIM editing Ramp-up and Tricks
## General notes

- nvim should be version > 0.8 for plugins working well
    - Download tarball and link the binary in case it's < 0.8
- In case of NeoVIM, Path for the config file is : ~/.config/nvim/init.vim
- This file is usefull to list Github repo of the plugins and configure some settings :
    - colorscheme
    - set number ! # for the line numbers

### Tricks & Commands

#### 1) Commands & Modes

##### a) Normal mode

| Action | Command |
|---|---|
| Move cursor | arrows (+ `Ctrl` for faster) or `h` `j` `k` `l` |
| Quit without saving | `:q!` |
| Quit with saving | `:x` |
| Save the file | `:w` |
| Delete a character | `x` |
| Insert mode | `i` |
| Insert a new line below | `o` |
| Append text at the end of the line | `A` |
| Delete a word | `dw` (`d2w` for 2 words, etc.) |
| Delete from cursor to end of line | `d$` |
| Delete an entire line | `dd` (`3dd` for 3 lines, etc.) |
| Move to beginning of line | `0` |
| Move to end of line | `$` |
| Change 2 words with "toto" | `c2wtoto` |
| Undo a change | `u` |
| Restore an entire line | `U` |
| Redo an undone change | `Ctrl-r` |
| Paste | `p` |
| Replace one character (e.g. with 't') | `rt` |
| Replace more than one character | `R` |
| Move to start of file | `gg` |
| Move to end of file | `GG` |
| Search text | `/theTextToFind` |
| Next / previous occurrence | `n` / `N` |
| Find matching parenthesis | `%` (on a parenthesis) |
| Find & replace (current line) | `:s/wrongWord/rightWord` |
| Find & replace (whole file) | `:%s/wrongWord/rightWord` |
| Copy from cursor to end of line | `y$` |
| Enter visual mode | `v` (or select with the mouse) |
| Go to definition/declaration | `gD` (or `Ctrl-]`, works better) |
| Go to tag (best way) | `Ctrl-]` |
| Go back from tag | `Ctrl-t` |

##### b) Insert mode / Visual mode

| Action | Command |
|---|---|
| Return to normal mode | `Esc` |
| Copy / paste in visual mode | select, then `y`, then `p` |

##### c) Other commands

| Action | Command |
|---|---|
| Split | `:split` |
| Close a split view | `:close` |
| Vertical split | `:vsplit` |
| Move between windows | `Ctrl-w` |

#### 2) Plugins

| Plugin / Action | Command / Note |
|---|---|
| GitGutter | preconfigured, enabled |
| nightfox colorscheme | set persistent in `init.vim` |
| Telescope | `Ctrl-k` |
| Telescope find_files | `Ctrl-f` |
| Diffview | `Ctrl-d` |
| NerdCommenter: comment | `\cc` |
| NerdCommenter: uncomment | `\cu` |
| NerdCommenter: sexy comment | `\cs` |
| NerdCommenter: toggle comments | `\c<Space>` |
| NerdTree | opens automatically at vim startup |
| NerdTree: open file in new split | `i` |
| NerdTree: open file in new vsplit | `s` |

[Back to Tools](./)
