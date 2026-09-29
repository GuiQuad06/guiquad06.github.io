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

1) Commands & Modes
a) Normal mode
- Moving cursor : arrows (+ ctrl for faster) or hjkl
- Quit => :q! (without saving) :x (with saving)
- Just save the file => :w
- Delete a character => x
- Insert mode => i
- Insert after carriage reurn new line => o
- Append text at the end of the line => A
- Delete a word => dw => d2w for 2 words to delete etc...
- Delete a line from cursor to the end of the line => d$
- Delete an entire line => dd => 3dd for 3 entire lines e.g
- Move at the begining of the line => 0
- Move at the eol => $
- change 2 words with toto => c2wtoto
- undo a change => u
- restore an entire line => U
- redo an undo change => Ctrl + R
- Paste (from cut & paste) => p
- Replace => rt (replace what is on cursor with 't') for one character
- Move at start => gg
- Move to the end => GG
- Search some text => /'theTextToFind' => n for next occurence forward , N for next occurence backward
- Find matching parenthesis => % (on a parenthesis)
- Find & replace => :s/wrongWord/rightWord
- Find & replace in whole file => :%s/wrongWord/rightWord
- Replace more than one character => R
- Copy something from cursor to EOL : y$
- Enter visual mode => v (or directly use the mouse to select something)
- Go to definition/declaration : gD (or <Ctrl-]> it works better)
    - Best way: <Ctrl-]> for go to tag , <Ctrl-t> for going back !!!

b) Insert mode/ Visual mode :
- Return to normal mode => <ESC>
- copy paste in visual mode => select -> y -> p

c) Other commands
- Split mode => :split
- Close a splitted view => :close
- Vertical split => :vsplit
- Moving between windows => Ctrl+w

2) Plugins
- GitGutter preconfigured, enabled...
- nightfox colorsheme set persistent in init.vim file
- telescope shortcut => <Ctrl+k>
- telescope find_files => <Ctrl+f>
- Diffview shortcut => <Ctrl+d>
- NerdCommenter preconfigured => \cc for comment
    - \cc for comment
    - \cu for uncomment
    - \cs for sexy comment
    - \c<Space> for toggling comments
- NerdTree open automatically at vim startup
    - open file in new split => i
    - open file in new vsplit => s

[Back to Tools](./)
