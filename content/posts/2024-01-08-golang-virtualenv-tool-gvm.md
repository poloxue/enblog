---
title: "Go Version Switching: Your Complete Guide to gvm"
date: 2024-01-08T14:45:03+08:00
draft: true
comment: true
description: "In this post, I'll share some of my thoughts on Go version management. Later on, I'll introduce a handy tool called gvm."
---
![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-08-golang-virtualenv-tool-gvm-01.png)

In this post, I'll share some of my thoughts on Go version management. Later on, I'll introduce a handy tool called gvm. While this topic may sound straightforward, to make the most of it, a brief overview is still necessary for smooth usage.

## Overview

Go version management isn't about managing package dependencies; it's about switching between different versions of Go. In our everyday tasks, we usually don't need to do this, so we might not understand its value.

Here's why I wrote this article:

When a new version of Go is released, to quickly try out its new features, switching to the latest Go version is necessary. However, if you download and install the new version, replacing the old one, it can be inconvenient. Having a way to swiftly switch between different Go versions would be much more convenient.

## Solution

When considering environment isolation, there are various options like multiple hosts, VMs, containers, etc. But, specifically for Go version management, we can manage it ourselves.

Switching versions in Go involves managing different environment settings. Before Go 1.10, we had to handle GOROOT, GOPATH, and PATH. After Go 1.10, GOROOT automatically sets to the installed Go path, so we only need to manage GOPATH and PATH.

## How to Implement

Here's how I set up the freedom to switch between two versions of Go on my computer:

Assuming they're in ~/.goversions/sdk/go1.11/ and go1.13/. To activate go1.11, I run:

```bash
$ export PATH=~/.goversions/sdk/go1.11/bin/:$PATH
```

At this point, GOROOT is automatically recognized as ~/.goversions/sdk/go1.11/. All Go-related tools, source code, standard libraries reside here.

Besides Go itself, there are third-party standard libraries and compiled library files that reside within the GOPATH directory. By default, if not set, this path is usually ~/go. When switching between multiple Go versions, this can cause confusion. To manage this better, we can set a separate GOPATH for each Go version

For go1.11, I set GOPATH as ~/.goversions/gopath/go1.11-global/:

```bash
$ mkdir ~/.goversions/gopath/go1.11-global/
$ export GOPATH=~/.goversions/gopath/go1.11-global/
```

A separate environment is created successfully.

Though the requirement is met, the process feels cumbersome. To simplify, we can refine the above steps into a shell script, forming a toolset.

Feeling eager to give it a try, huh?

Unfortunately, the chance has slipped away as someone has already developed a tool similar to what was being envisioned here, but with enhanced capabilities. It's called gvm, and you can find it at moovweb/gvm.

## GVM

gvm, short for Go Version Manager, offers lightweight Go version switching. Compared to other languages, there are similar tools like NodeJS's NVM or Python's virtualenv.

Beyond version switching, gvm can directly install any Go version from source code. After Go 1.5, Go implemented self-compilation. 

Here's a simple and easy-to-understand version in English:

### GVM Installation

The installation process is straightforward. Just run the following command line:

```bash
$ bash < <(curl -s -S -L https://raw.githubusercontent.com/moovweb/gvm/master/binscripts/gvm-installer)
```

You'll see output like this:

```bash
Cloning from https://github.com/moovweb/gvm.git to /home/vagrant/.gvm
which: no go in (/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/home/vagrant/.local/bin:/home/vagrant/bin)
No existing Go versions detected
Installed gvm v1.0.22

Please restart your terminal session or to get started right away run
 `source /home/vagrant/.gvm/scripts/gvm`
```

Installation completed!

Restart your terminal or execute `source $HOME/.gvm/scripts/gvm` to activate gvm.

Keep in mind that different operating systems might require additional dependencies. For specific details, check the [project description](https://github.com/moovweb/gvm#mac-os-x-requirements). 

Note: The information does not cover Windows compatibility, so it might not be available for use on that platform.

### How GVM Installs Go

GVM installs Go by downloading the source code from GitHub and compiling it. It relies on tags within the source code for different versions. Since version 1.5, Go supports self-compilation, requiring an existing Go environment to use GVM for Go installation.

The good thing is, GVM also offers the option to install Go directly by downloading precompiled binary packages.

```bash
$ gvm install go1.19 --binary
Installing go1.19 from binary source
```

Once Go is installed, you can freely use GVM to install and switch between different Go versions and commit hashes.

```bash
$ gvm install go1.11
```

Wait for the process to finish running.

The initial installation might take longer, mainly depending on your internet speed, as it requires downloading the source code from GitHub for the first time.

### Checking Installed Versions

Firstly, let's see which Go versions are installed on my system using the command gvm list.

```bash
$ gvm list

gvm gos (installed)

   go1.11
   go1.12
   go1.13
   go1.13beta1
```

I have installed 4 versions. Note that `go1.13beta1` is an unstable version. If you want to try out new Go features quickly, GVM comes in handy.

Apart from viewing installed versions, you can use `gvm listall` to check all available versions fetched from tags in the source code.

```bash
gvm gos (available)

   go1
   go1.0.1
   go1.0.2
   go1.0.3
   go1.1
   go1.1rc2
   go1.1rc3

   ...

   go1.21rc4
   go1.21.0
   go1.21.1
   go1.21.2
   go1.21.3
   go1.21.4
   go1.21.5
   go1.22rc1
   ...
```

### Selecting a Version

Choosing and activating a version is straightforward:

```bash
$ gvm use go1.11 [--default]
```

Upon successful activation, verify using `go version` and `go env`. To set a default version, add `--default`.

### Package Environment Management

GVM manages package environments with commands like `pkgenv` and `pkgset`, even without package dependency management tools.

For example, creating an environment for a new project called 'blog':

```bash
$ gvm pkgset create blog  # Create
$ gvm pkgset use blog     # Activate
```

Packages installed via go get will be in the 'blog' environment. This is because go get usually places installations in the first directory of GOPATH.

This covers the basics. For those interested, further exploration is recommended. As we now have 'go mod,' this functionality may not be needed much in the future.

### GVM Directory Structure

GVM is shell-written and typically installs in `$HOME/.gvm/`. Understanding its directory structure helps understand its implementation:

The main directories include:

```bash
archive             # Go source code
bin                 # GVM executable files
environments        # Configuration for different environments
scripts             # Subcommand scripts for GVM
logs                # Log information
pkgsets             # Path for each independent environment's GOPATH
```

Upon studying GVM's implementation, we realize that this approach can be applicable to version management in many other tools. If we encounter similar needs in the future without ready-to-use tools, implementing our own commandline tool might be feasible.

## Conclusion

Starting from my needs, this article introduces flexible Go version management.

Past experiences suggested that if other languages have tools for these needs, Go should have them too. Upon searching, I found GVM. While I noticed some bugs and less-than-ideal experiences, overall, it sufficiently meets my requirements.

