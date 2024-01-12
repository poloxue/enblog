---
title: "🚀 Supercharge Your Searches: A Deep Dive into fd, ripgrep, and fzf Commands"
date: 2024-01-11T15:17:43+08:00
draft: true
comment: true
---

![cover](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-11-high-productivity-command-part2-01.png)

🚀 **Boost Your Terminal Productivity with fd, ripgrep, and fzf!**

Welcome to a world where searching and finding in your terminal becomes lightning-fast and user-friendly. Meet your new command-line allies: `fd`, `ripgrep`, and `fzf`.

## Unleash the Power of fd

[fd](https://github.com/sharkdp/fd) is not just a file and directory search command; it's a game-changer. Say goodbye to the sluggish `find` command and embrace the speed and simplicity of `fd`.

### 🔍 Simple Search
```zsh
fd pattern
```
![Simple Search](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-01.gif)

### 🧠 Regular Expression Brilliance
```zsh
fd '.*\.go$'
```
![Regular Expression Search](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-02.gif)

### 🌟 Wildcards for the Win
```zsh
fd pattern*
```
![Wildcards](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-04.gif)

### 🎯 Specify File Type
```zsh
fd -e go pattern
```
![Specify File Type](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-05.gif)

### 🕵️ Hidden Files and .gitignore
```zsh
fd -H pattern
fd -I pattern
```
![Hidden Files and .gitignore](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-06.png)

### 🚀 Supercharge Your Search with fd
Explore more with case-insensitive searches, smartcase mode, detailed results, size filtering, and much more!

- Ignore Case: fd -i readme.md
- Smartcase Mode: fd -s readme.md or fd -s README.md
- Detailed Results: fd -l .go
- Size Filtering: fd -S +1000k
- Only Directories: fd --type/-t directory golang
- Search and Execute Command: fd --type/-t file -x wc -l

### ⚡ Performance Unleashed
Tested against `find` in a performance showdown. `fd` emerges as the undisputed champion.

![Performance Comparison](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-08.gif)

## Dive into the Text with ripgrep

Meet [ripgrep](https://github.com/BurntSushi/ripgrep), your new best friend for lightning-fast text searches.

### 🔍 Default Recursive Search
```zsh
rg pattern
```
![Default Recursive Search](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-09.gif)

### 🚀 Turbo-Charged Searches
`ripgrep` doesn't just search; it conquers. Explore directory-specific searches, file-specific hunts, and even disable recursion with ease.

```bash
rg main ~/Code/golang-examples/ # 搜索指定目录
rg main ~/Code/golang-examples/main.go # 索索制定文件
rg main -g '!/*/*' # 通配符禁用递归目录
rg main -g '!main.go' # 统配符排除文件
rg -g '!directory' # 通配符排除目录
rg -e '[0-9]{2}:[0-9]{2}'
```

### 🔍 Unveiling Default Filtering Magic

ripgrep is celebrated for its unparalleled search prowess, owing in part to its default exclusion of specific files. 

It intelligently bypasses hidden files and those enumerated in .gitignore, .ignore, .rgignore. 

Easily disable ignore with the --no-ignore option. For hidden files, employ --hidden to search inclusively. To exclude all hidden files, opt for -uuu, as in rg -uuu pattern.

### 🎨 Customize Your Search

To tweak ripgrep's default behavior, modify its configuration file by specifying the environment variable `RIPGREP_CONFIG_PATH`.

```zsh
# Avoid displaying excessively long lines and enable previews.
--max-columns=150
--max-columns-preview

# Introduce a new file type 'web'.
--type-add
web:*.{html,css,js}*

# Search hidden files/directories (e.g., dotfiles) by default.
--hidden

# Utilize glob patterns for file/folder inclusion/exclusion.
--glob=!.git/*

# Alternatively
--glob
!.git/*

# Define color schemes.
--colors=line:none
--colors=line:style:bold

# Case insensitivity because who cares about case!?
--smart-case
```

Specify your preferences in the configuration file to enhance your ripgrep experience! 🛠️✨

## Supercharge Your Workflow with fzf

Introducing [fzf](https://github.com/junegunn/fzf), the fuzzy finder that elevates your command-line experience.

### 🌐 Directory Search Made Fun
```zsh
fzf
```
![Directory Search](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-13.gif)

### 🚀 Unleash fzf's Magic
Combine fzf with other commands, turning mundane tasks into interactive adventures. Explore the possibilities with ls, fd, and more.

```bash
fd --type file | fzf
vim `fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-16.gif)

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-17.gif)

### 🎯 Your Commands, Your Rules
Create powerful command combinations by sending fzf's search results as input to your favorite commands.

```bash
cd `zoxide query --list {querystring} | fzf`
```

![](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-30-high-productivity-shell-commands-part2-19.gif)

### ⚙️ Configuration for the Win
Tweak fzf's behavior with a configuration file. Tailor it to your preferences for the ultimate experience.

```bash
# Enable preview window with bat (syntax highlighting)
export FZF_PREVIEW_COMMAND='[[ $(file --mime {}) =~ binary ]] && echo {} || bat --style=numbers,changes --color=always {} 2> /dev/null | sed "s/\[[0-9;]*m//g"'

# FZF options
export FZF_DEFAULT_OPTS='
--layout=reverse
--inline-info
--height=80%
--border
--ansi
--prompt="> "'

# Use ripgrep as the default source for files
export FZF_DEFAULT_COMMAND='rg --files --no-ignore --hidden --follow --glob "!.git" 2>/dev/null'

# Use ripgrep for fuzzy searching within files
export FZF_ALT_C_COMMAND='rg --files --no-ignore --hidden --follow --glob "!.git" 2>/dev/null'
```

## In Conclusion

Say farewell to sluggish searches and complicated commands. With `fd`, `ripgrep`, and `fzf`, your terminal becomes a powerhouse of efficiency. Embrace these tools, and watch your productivity skyrocket!

🚀 **Ready to revolutionize your terminal experience? Dive in and command your way to greatness!**

