---
title: "Autostart Tmux in iTerm2"
date: "2023-09-15T21:22:36+08:00"
draft: false
comment: true
tags: ["iTerm2", "Tmux"]
---

This post will introduce how to make iTerm2 start into Tmux mode by default. 

By default, every time you start iTerm2, you need to enter tmux attach to enter tmux mode.

I use Tmux to manage workspaces for different projects. Common IDEs generally provide an interface for users to select projects. Naturally, can iTerm2 + Tmux have a similar function?

Very simple!

## Solution 1: A single-line shell script

First, let's see a bash script:

```bash
tmux ls && read -p "Select a session<default>:" tmux_session && tmux attach -t ${tmux_session:-default} || tmux new -s ${tmux_session:-default}
```

The description of this script is as follows:

- `tmux ls`, output the currently available sessions;
- `read xxx`, read user input and store the name of the session you want to open into variable `tmux_session`;
- `tmux attach`, try to open the session, if `tmux_session` is empty, open the default session;
- `tmux new`, if opening fails, try to create a new session;

Note: Because of the `read` command option `-p`, you must use bash to run this script.

**Configure iTerm2 startup loading**

Choose 'Preference -> Profile -> General -> Command -> Select "Login Sell"，"Send text at start"' and input the above script.

Restart your iTerm2, and what you will see is like this:

```bash
hello: 1 windows (created Tue Sep 12 17:13:54 2023) (group hello) (attached)
default: 1 windows (created Wed Sep 13 19:54:36 2023) (group default)
Select a session<default>:
```

Input "hello" and enter , iTerm2 will enter the "hello" session. 

It's simple and efficient, and a bit like the IDE that lets us select a project after it's started.

The session list is not concise enough. You can configure the output formatting of `tmux ls` by the command `tmux -F '\#{session_name}'`.

Output: 

```bash
hello
default
```

## Solution 2: python script

Can I just enter the session index to select? This single line shell script is not easy to implement. I wrote a Python script as follows:

```python
#!/usr/bin/env python
import os

output = os.popen("tmux ls -F \\#{session_name}").read()
sessions = output.strip().split("\n")

print("Sessions:")
for index in range(len(sessions)):
    print(f"{index} - {sessions[index]}")

input_value = input("Please select a session <Index or Name>(default):")
if input_value.isdigit():
    sess_name = sessions[int(input_value)]
elif not input_value:
    sess_name = "default"
else:
    sess_name = input_value

if sess_name not in sessions:
    answer = input(f"New a session `{sess_name}`?(Y/N)")
    answer == "Y" and os.system(f"tmux new -s {sess_name}")  # pyright: ignore
else:
    os.system(f"tmux attach -t {sess_name}")
```

Suppose this file's name is tmux_selector and put it under the home directory. 

Let's configure this script as iTerm2 startup script. Restart our iTerm2.

We will see:

```bash
❯ ~/tmux_selector
Sessions:
0 - hello
1 - default
Please select a session <Index or Name>(default):
```


This is just a demo. The input of tmux ls has other formatting variables. If you are interested, you can continue to expand. Code snippet [github address](https://gist.github.com/poloxue/37d3d79b35964ab8d885296b84ab4b5a).

It's good, but wouldn’t it be better if it uses the built-in menu? 

Let's continue!

## Solution 3: Use tmux built-in menu tree

After entering tmux mode, you can use the shortcut key `prefix-key + s` to start the tmux selection menu by default. One strength is the built-in menu can be moved by keys `jk`. 

```zsh
hello: 1 windows (created Tue Sep 12 17:13:54 2023) (group hello) (attached)
default: 1 windows (created Wed Sep 13 19:54:36 2023) (group default)
```

Is there a way to start this menu directly when iTerm2 starts? What a great idea! Actually it's not difficult to implement.

First, use `tmux list-keys` to check which command the `prefix-key + s`  is bound.

```zsh
$ tmux list-keys
...
bind-key    -T prefix       s                    choose-tree -Zs
...
```

We can display the menu by `choose-tree -Zs`. In tmux mode, use `tmux choose-tree -Zs` to test whether it works.

How to configure it as the iTerm2 startup command?

This command must be run in tmux mode. How to do? We can use the format `tmux attach\; <commands running on tmux mode>` to achieve this goal.

The command is as follows:

```zsh
$ tmux attach\; choose-tree -zS
```

Try testing this code in non-tmux mode to see if you can successfully show the menu. 

Finally, in order to prevent when the first entry without a session, I further optimized the script. It will create a default session window when no session exists.

The final script:

```bash
$ tmux attach\; choose-tree -Zs || read -p "Create a default session?(Y/N)" anwser && [[ "${anwser}" == "Y" ]] && tmux new -t default
```

But there is another disadvantage here. Pressing `q` can only exit the menu and the terminal is still in the tmux mode. One more step of detach is required to exit. This is a difference from the steps we generally think of.

