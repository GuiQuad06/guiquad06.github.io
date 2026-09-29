---
layout: default
title: Doom Emacs Cheatsheet
---

# DOOM Cheatsheet

- [General](#general)
- [SPC p (project)](#spc-p-project)
- [SPC (general commands)](#spc-general-commands)
- [SPC f](#spc-f)
- [SPC b](#spc-b)
- [SPC o (open a component)](#spc-o-open-a-component)
- [CTRL-w](#ctrl-w)
- [M-x : org-mode (toto.org)](#m-x--org-mode-totoorg)
- [Tags](#tags)
- [Checkboxes](#checkboxes)
- [EDIFF](#ediff)

## General

- VIM bindings ok ! (hjkl - `:q` `:w` - o, dd, 2w, etc...)
- `ALT-x` pour les commandes (packages installed such as tldr...), projectile indexer...

## SPC p (project)

- `D` (iscover project in folder)
- `p` (switch to indexed project)
- `g` (configure)
- `c` (ompile)
- `t` (est)

## SPC (general commands)

- `SPC` (find file in project)
- `.` (find file)
- `*` (search for symbol in code)

## SPC f

- `f` (ind file)
- `F` (ind file from here)
- `D` (elete file)
- `p` (config files)

## SPC b

- `B` (switch buffer)

## SPC o (open a component)

- `p` (treemacs)
- `e` (Shell)
- `a a` (genda)
- `a t` (odo list)

## CTRL-w

- `v` : vertical split editor
- `s` : horiz split editor

## M-x : org-mode (toto.org)

- `*` / `**` / `***` for headlines
- `tab` to expand/hide
- `M-j` / `M-k` to reorder
- `M-Enter` to add a headline
- `C-Enter` to add a headline with the TODO mark
- highlight, `SPC m l`, paste the link
- `SPC h v` : change variable
- TODO lists (`SPC m t` to change status)
- `SPC m d t` to add a deadline and make it visible on Agenda
- `Shift-arrows` to change priority of a task

## Tags

- `SPC m q`
- `SPC o a m` (to search by Tag)

## Checkboxes

- `[ ]` Press enter to change the state
- On headline add `[/]` plus `C-c C-c` to have a status bar

## EDIFF

- `SPC e d` to open the diff view between two existing buffers
