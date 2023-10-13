---
title: "Setup Your Own iTerm2"
date: "2023-10-12T10:22:36+08:00"
draft: false
comment: true
tags: ["iTerm2"]
description: "This post will introduce three solutions about how to autostart Tmux when starting iTerm2."
---

{{< youtube id="4N9wFVj9LVE" autoplay="true" >}}

Hi guys, This is POLO X.

Today, I plan to talk about how to install iTerm2 and configure iTerm2 color presets. At the end of the post, there will be some tips about iTerm2 to share with you.

## What is iTerm2?

Quote from the official site.

> iTerm2 is a replacement for Terminal and the successor to iTerm. It works on Macs with macOS 10.14 or newer. iTerm2 brings the terminal into the modern age with features you never knew you always wanted.

Shortly, it is a better terminal than the terminal built in MacOS and provides more powerful features.

## Installation

Now, let's start to install it.

There are two ways to install iTerm2.

- One is download the installer from the official site.
- Another is use homebrew command.

In this tutorial, I'm going to demonstrate how to install it using homebrew.

### Install Homebrew

Homebrew is not a function built in MacOS, so it must be installed in advance.

Let's open the [homebrew official site](https://brew.sh).

Copy the command from this site.

```
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Open the built-in terminal, and paste it into our terminal. Input the password and confirm.

Wait until it is completed.

After all done, type the command `brew` to see if it works.

### Install iTerm2

Now , let us run 

```bash
brew install --cask iterm2
```

Wait for a while.

Now, we succeed in  installing iTerm2.

## Configure iTerm2's color presets?

It's very easy.

To teach you how to configure it. Three color presets, Material Design Colors, Snazzy and Dracula, will be installed.

### Install Material Design Colors

Firstly, let's use Material Design Colors as iTerm2 color preset. Download the iterm colors file using curl command.

```bash
curl -Ls https://raw.githubusercontent.com/mbadolato/iTerm2-Color-Schemes/master/schemes/MaterialDesignColors.itermcolors > /tmp/material-design-colors.itermcolors
open /tmp/material-design-colors.itermcolors
```

Open the iterm colors file.  

It will be installed.

Now let's use `Command + ,` to the open the preferences of iTerm2.

- Select profile.
- Select colors tab.
- Click the color presets. 

You will see the color preset we just installed. Select it, and then you will see a change in iterm2.

### Install Snazzy and Dracula

Now, we can install more color presets into our iTerm2, such as Snazzy and Dracula.

Install Snazzy.

```bash
curl -Ls https://raw.githubusercontent.com/sindresorhus/iterm2-snazzy/main/Snazzy.itermcolors > /tmp/Snazzy.itermcolors
open /tmp/Snazzy.itermcolors
```

Install Dracula.

```bash
curl -Ls https://raw.githubusercontent.com/dracula/iterm/master/Dracula.itermcolors > /tmp/Dracula.itermcolors
open /tmp/Dracula.itermcolors
```

Now, all color presets have been installed.

### More Color Presets

Do you wanna more choices for color presets?

Let me share you a web site: [iterm2colorsschemes](https://iterm2colorschemes.com).

Here you can get more color presets. Let's chose one from the site - 3024 Night. 

Download its itermcolors file, and install it.

```bash
curl -Ls https://raw.githubusercontent.com/mbadolato/iTerm2-Color-Schemes/master/schemes/3024%20Night.itermcolors > /tmp/3024Night.itermcolors
open /tmp/3024Night.itermcolors
```

Open the preferences -> Profiles -> Colors -> Click color presets, and check whether 3024 Night has been installed.

## iTerm2 Tips

At last, let me introduce three tips about how to use iTerm2.

### Split Panes

Use multiple panes in iTerm2.

Compared to Terminal, iTerm2 supports multiple panels. You can work in different panels in the same window.

How?

We can use shortcut `CMD+D` to split panes horizontally. And use  shift command D to split panes vertically.

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-11-setup-your-own-iterm2-01.gif)

Now, we can do different tasks in different panels.

### toggle a Pane Full Screen.

iTerm2 allows you to maximize and minimize a pane on demand, without messing with existing open panes.

This shortcut for this function is `Shift+Cmd+Enter`.

Press `Shift+Cmd+Enter`, and then it will maximize or minimize the panel where your cursor is now.

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-11-setup-your-own-iterm2-02.gif)

### search text.

Press `Command+F`. Input the text we want to search. It looks more comfortable than the default Terminal.

![](https://cdn.jsdelivr.net/gh/poloxue/images@main/2023-10-11-setup-your-own-iterm2-03.gif)


That's all for this post.

