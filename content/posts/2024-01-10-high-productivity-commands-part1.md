---
title: "2024 01 10 High Productivity Commands Part1"
date: 2024-01-10T14:11:04+08:00
draft: true
comment: true
description: "Today, I'm here to introduce three super cool commands that work like a charm on Unix-like systems. They're your go-to pals for changing directories and viiewing files like a boss:"
---

![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-10-high-productivity-command-part1-01.png)

Hey there! Today, I'm here to introduce three super cool commands that work like a charm on Unix-like systems. They're your go-to pals for changing directories and viiewing files like a boss:

- **[exa](https://github.com/ogham/exa)**, the new "ls" in town.
- **[zoxide](https://github.com/ajeetdsouza/zoxide)**, the smart CD buddy that knows where you want to go.
- **[bat](https://github.com/sharkdp/bat)**, a "cat" that's got style with syntax highlighting and even Git superpowers!

These champs directly swap out those everyday commands: ls, cd, and cat.

> Psst! exa is no longer in active development, but fear not! You can hop onto [eza](https://github.com/eza-community/eza) – it's an exa fork. To get it, use the command: brew install eza.

## [exa](https://the.exa.website/)

This command is your new best friend, stepping up to replace the dull old "ls." Let's be honest, we use ls a ton, right? Well, exa not only does what ls does but also brings in more exciting features!

### Quick Install

```bash
brew install exa
```

### How to Use It

Check this out - some sample exa commands and their magic:

Example 1 - exa's default look:

![exa example 1](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-01.png)

Example 2 - exa --icons for file type icons:

![exa example 2](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

Example 3 - exa -alh --git or exa --all --long --header --git for file details and git info:

![exa example 3](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-02.png)

Example 4 - exa --tree --icons for a file tree:

![exa example 4](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-04.png)

### Handy Aliases

Make exa your go-to ls by setting up some aliases:

```bash
# Show icons by default
alias ls="exa --icons"
# Detailed file directory
alias ll="exa --icons --long --header"
# Show all files, including hidden ones
alias la="exa --icons --long --header --all"
# Detailed view with git status
alias lg="exa --icons --long --header --all --git"

# Replace tree command
alias tree="exa --tree --icons"
```

And just like that, exa becomes your new favorite ls command!

> P.S. If you want to use the original command after setting aliases, try using `\` before the command, like `\ls`. It'll bypass the alias and use the system's default ls command.

## [zoxide](https://github.com/ajeetdsouza/zoxide)

Let's be real, does the default Linux "cd" command drive you nuts? It's like taking the smoothest ride and then - boom! - you hit a "cd" and everything slows down.

```
cd ../
cd ../
cd ../
cd ../../../
cd x/
cd y/
cd z/
```

If you're tired of this "cd" struggle, check out [zoxide](https://github.com/ajeetdsouza/zoxide).

[zoxide](https://github.com/ajeetdsouza/zoxide) works its magic inspired by z and autojump. It records where you've been, making directory jumps with minimal keystrokes.

> Note: I know you've heard of the z plugin from oh-my-zsh, but zoxide is even easier. Check out a comparison report: [zoxide vs zsh-z](https://www.libhunt.com/compare-zsh-z-vs-zoxide).

### Installation

```zsh
brew install zoxide
```

### Setup

To use zoxide in zsh, add one line to your `~/.zshrc` file to initialize zoxide. Here's how:

```zsh
## Using z to enable zoxide
echo 'eval "$(zoxide init zsh --cmd z)"' >> ~/.zshrc
## Or directly replace cd command
echo 'eval "$(zoxide init zsh --cmd cd)"' >> ~/.zshrc
```

I'll go with the second way - directly swapping the cd command.

### Let's Dive In

Imagine this directory structure:

```
~/Hello
 |_ ./golang-examples
 |_ ./python-examples
 |_ ./rust-examples
 |_ ./trading-strategies
```

Since zoxide speeds up jumps using your visit history, let's prepare some data.

Initialize the zoxide database with these commands:

```zsh
cd ~/Hello/golang-examples
cd ~/Hello/python-examples
cd ~/Hello/rust-examples
cd ~/Hello/trading-strategies
```

Example 1: cd golang-examples - exact match.

Example 2: cd golang - partial match.

Example 3: cd examples - multiple matches pick the best.

Check out these cool effects:

![zoxide examples](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-05.gif)

Example 4: cd examples + <Space>+<Tab> for interactive mode.

![zoxide interactive mode](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-07.gif)

Apart from using <Space>+<Tab> for interactive mode, you can also directly use the cdi command. Let's dive into Example 5:

![zoxide cdi command](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-06.gif)

## [bat](https://github.com/sharkdp/bat)

[bat](https://github.com/sharkdp/bat) is the "cat" that knows how to shine with syntax highlighting and Git integration.

### Installation

```zsh
brew install bat
```

### Configuration

In my system, the default theme displayed by the 'bat' command isn't clear enough in the dark mode, and I suspect it's due to its default theme being 'light'. Therefore, I always end up setting the theme manually, like `bat --theme=TwoDark main.py`. But guess what? Bat allows configuration through a config file.

> Check out available themes with `bat --list-themes`.

First, set the bat config file's path in `.zshrc`.

```
export BAT_CONFIG_PATH="${XDG_CONFIG_HOME:-~/.config}/bat.conf"
```

Once that's done, execute this command to generate the config file:

```zsh
bat --generate-config-file
```

Configure bat's default options to enable the theme, like `--theme=TwoDark`, in the config file:

```zsh
# Specify desired highlighting theme (e.g. "TwoDark"). Run `bat --list-themes`
# for a list of all available themes
--theme=TwoDark
```

### Time to Show Off!

Example 1: Syntax highlighting and line numbers.

![bat example 1](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-08.gif)

Example 2: Git integration.

![bat example 2](https://cdn.jsdelivr.net/gh/poloxue/images@2023-10/2023-10-28-high-productivity-shell-commands-part1-09.gif)

Example 3: Paging like cat but without breaking pages.

```zsh
bat --pager='never' logger.go
```

If you're a die-hard cat user and want to skip paging by default, add `--paging=never` to the config file.

### Handy Alias

If you fancy bat and want to make it the new cat, just set an alias in your `zshrc`:

```bash
alias cat='bat'
```

## Wrapping Up

These three commands - exa(eza), zoxide, and bat - spice up your terminal life, turning the most mundane commands into a fun experience and boosting your efficiency.

