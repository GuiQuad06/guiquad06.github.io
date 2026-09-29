---
layout: default
title: Command Line Cheatsheet
---

# Command line

## Powershell
- Env variable :
```
$env:PATH
```
- List env variables :
```
Get-ChildItem Env:   # All
Get-Item Env:PATH    # Only PATH
```
- [Powershell Config](https://github.com/GuiQuad06/powershell-profile)
- Some shortcuts that applies to that configuration:
    - z (zoxide)
    - np (notepad++)
    - Ctrl+f (fzf)
    - Ctrl+r (commandline history + fzf)
    - tig
    - which <commande> (find out command path)

## Bash (Debian Linux / WSL2)
- [Dotfiles config](https://github.com/GuiQuad06/linux-dotfiles)

### TMUX
- Ctrl + s ==> mode commande
- Ctrl + s Ctrl + c ==> crée une session
- Ctrl + s 1 ==> go session no 1
- Ctrl + s Ctrl + % ==> Vertical split
- Ctrl + s Ctrl + " ==> Horizontal split

### Ripgrep
Git grep like / Eclipse Ctrl+H like / VSCode loupe like
```
rg [pattern] [path]
```

### FZF
Exemples d'utilisation :
```
fzf
fzf --preview='cat {}'
nvim $(fzf --preview='cat {}')
nvim $(fzf --preview 'batcat --style=numbers --color=always {}')
rg void | fzf | cut -d':' -f 1 | xargs -n 1 nvim
```

Avec alias :
| Alias | Description |
| --- | --- |
| `f` | fzf avec le preview |
| `vi $(f)` | fzf avec le preview puis vim en sortie |
| `Fgrep <pattern>` | rg + fzf + vim |

### Misc
Trick pour rechercher rapidement des symboles dans un fichier source :
```
find . -name "*.c" -exec grep TOTO
```
Other tools:
- z (zoxide) - a way much better cd
- bat (batcat) - a better cat

[Back to Tools](./)
