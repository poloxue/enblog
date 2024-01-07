---
title: "IDE for Go Development: Exploring Features and Popular Options"
date: 2024-01-07T13:23:03+08:00
draft: false
comment: true
---

![cover.jpeg](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-07-why-we-should-use-golang-ide-01.jpeg)

Why do programmers need to use an IDE?" 

This question frequently surfaces on various community forums. When it comes to whether one should use an Integrated Development Environment (IDE), everyone has their own viewpoint.


![](https://cdn.jsdelivr.net/gh/poloxue/images@2024-01/2024-01-07-why-we-should-use-golang-ide-02.jpeg)

In the early days, programming didn't necessitate an IDE as it was primarily centered around machine code. However, as the computer industry progressed, IDEs emerged to enhance engineering development efficiency.

It's essential to grasp that an IDE primarily integrates various command tools into a user-friendly software, facilitating efficient programming practices. Its ultimate goal is to elevate project development efficiency.

Understanding the essence of an IDE, for those inclined towards exploration, it's entirely possible to fashion a personalized IDE using text editors like Vim or Emacs.

## Supported Features

Regardless of whether you're using an existing IDE available in the market or manually crafting an IDE like Vim, the discussion always revolves around the functionalities an IDE provides. Text editing capability doesn't need much introduction as it's the fundamental feature.

### Keyboard Shortcuts

Remaining hands-on-the-keyboard is crucial for efficient development, and this is achieved through powerful keyboard shortcuts. IDEs typically offer a unique set of shortcut conventions. Once accustomed to an IDE, the keyboard shortcuts might be one significant reason why users are hesitant to switch to another IDE.

### Code Highlighting

Code highlighting involves variables, function definitions, classes, constants, special symbols, keywords, and more. It enhances code readability by using distinct color schemes for various syntaxes, reducing the chances of coding errors. Additionally, IDEs usually support customizable color schemes, allowing users to personalize them according to their preferences.

### Code Formatting

For streamlined team development, standardized code conventions are usually established before project development. Adhering to these standards is essential, and IDEs generally offer code formatting capabilities, making it more convenient to achieve these objectives. It's worth noting that unlike Go, many programming languages lack a command like 'gofmt', and code conventions vary.

### Code Suggestions

IDEs provide code suggestions based on inputs, quickly offering a list of suggestions like parameter information, member lists, code snippets, and more. Some IDEs even analyze users' historical operations to offer more accurate suggestions, resembling a miniature recommendation system.

### Navigation

In large-scale projects, code volume and file count tend to be substantial. During development, navigating between variables, functions, classes, and other code elements is frequent. IDEs typically facilitate swift navigation among variables, type definitions, function definitions, and files, often through keyboard shortcuts or mouse actions.

### Code Debugging

While simple debugging can often be achieved through print functions, systematic debugging tools cater to complex scenarios. These tools usually support various debugging capabilities such as breakpoints, variable observation, and more.

### Build Compilation

Commonly used on Linux, Makefile is a frequently used build tool, particularly in C/C++ development. However, for some language projects, using Makefile might be complex, like in Java. IDEs offer build compilation functions that swiftly generate target files. Compilation functions usually utilize the language's native compiler, such as Go using the 'go build' command.

### Additional Features

Besides the functionalities mentioned above, IDEs might encompass various other capabilities like code refactoring, file history tracking, language environment management, database management, and more. Virtually any conceivable feature can be integrated into modern IDEs, far surpassing the traditional scope of IDE functionalities.

## GO IDE Options

Over the past decade, GO's development has spanned more than ten years. During this time, numerous IDEs capable of coding in the GO language have emerged. However, providing a detailed introduction to each of them would be impractical. Let's focus on a few IDEs that I am more familiar with for comparison.

### Goland


![goland](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-goland.png)

Goland, an integrated development environment for Go launched by the commercial company JetBrains, is undeniably powerful.

I believe many developers have used the IDEs from JetBrains, such as IntelliJ IDEA for Java, PHPStorm for PHP, PyCharm for Python, CLion for C++, and WebStorm for frontend development. Using JetBrains' IDEs, we can experience their excellent out-of-the-box usability and benefit from the plugins accumulated by JetBrains over more than a decade.

A few years ago, before Goland was released, if we wanted to develop in Go using JetBrains' IDE, we needed plugin support provided by them. However, after the release of Goland, it seems these plugins have been deprecated.

Admittedly, Goland excels in terms of functionality. However, I have a few gripes to mention. Firstly, JetBrains' IDEs often suffer from performance issues and significant resource consumption. While some experts have proposed optimization solutions, the user experience still doesn't quite compare to other IDEs in term of performance.

Goland's ease of use means there's not much to critique; it has minimal issues and is almost ready for coding once installation is complete!

### VSCode

![vscode](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-vscode.png)

VS Code, a modern, lightweight, and robust code editor IDE developed by Microsoft, is an open-source tool. Its remarkable plugin extension capabilities enable support for projects in almost every major programming language, and GO is certainly among them.

My choice to explore VS Code wasn't driven by a typical geeky inclination or a mere desire to experiment aimlessly. It stemmed from the frustration caused by the frequent lags in Jetbrains' IDEs and the necessity to frequently switch between different programming languages. Starting multiple Jetbrains IDEs simultaneously was a painful experience.

To enable GO development capabilities in VS Code, you just need to install a single plugin. You can refer to the [VsCode Golang Documentation](https://code.visualstudio.com/docs/languages/go) for assistance.

On a side note, it's worth mentioning that VS Code is developed using Electron, an open-source library that uses HTML, CSS, and JavaScript to build cross-platform desktop applications. Leveraging the browser's features, we can implement a wide array of unique plugins using VS Code.

### Vim Go

![vim](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-vim.png)

Vim, fundamentally a text editor, surprisingly possesses a multitude of functionalities that extend beyond traditional text editing. It encompasses features like word completion, ctags tag jumping, window splitting, crash file recovery, file diffs, and over 400 syntax highlighting options. One of its most crucial attributes is its scripting language, enabling Vim to expand its capabilities through plugin extensions.

To transform Vim into a suitable GO IDE, extensive configurations, scripts, and a combination of various plugins are necessary. The common functionalities found in other IDEs, as mentioned earlier, must be configured individually within Vim.

Setting up a GO development environment in Vim requires a crucial plugin called vim-go. There's also a helpful [tutorial video on YouTube](https://www.youtube.com/watch?v=7BqJ8dzygtU) about it, which might be of interest to you.

Vim-go offers an array of functionalities such as code compilation, execution, testing, code refactoring, error prompts, and more. You can delve deeper into its capabilities through the [vim-go tutorial](https://github.com/fatih/vim-go-tutorial).

It's important to note that while Vim supports plugin extensions, achieving an experience similar to that of VS Code is exceptionally challenging.

Presently, I primarily use three IDEs: Goland, VSC, and Vim. Of course, there are numerous other IDEs available. Although I haven't extensively used them, I'll briefly introduce a few. 

![neovim](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-neovim.png)

Lately, I migrated from Vim to Neovim, and the experience has been fantastic.

### Sublime Text

![sublime text](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-sublime.png)

Initially, I used VS Code, finding its usability quite similar to Sublime. When it comes to Sublime, it's often hailed as a powerful text editor. Its coding capabilities are primarily extended through plugins, and GoSublime is one such plugin that enriches Sublime's functionality specifically for Go.

### Atom

![atom](https://cdn.jsdelivr.net/gh/poloxue/images@logo/logo-atom.png)

Similar to VS Code, Atom is built on Node-Webkit, or Electron. It's an open-source text editor launched by GitHub. The go-plus plugin tailors Atom specifically for Golang development.

### LiteIDE

LiteIDE is a lightweight IDE, reportedly developed by individuals from China. It might have been more popular before the emergence of Goland. Perhaps due to my limited exposure, I'm unaware of its current user base.

### Eclipse

This open-source IDE has enjoyed considerable popularity over the years, boasting a rich set of resources and a sizable fan base, particularly favored among Java developers. GoEclipse is an Eclipse plugin designed for Go development. However, from my exploration on GitHub, it seems this project hasn't seen recent updates.


## Conclusion

This article began with a discussion on the significance of using IDEs, delving into some of their developmental histories. It also summarized the typical functionalities that most basic IDEs tend to offer. Understanding these aspects can aid us in better utilizing them in the future. 

Finally, the article introduced several popular IDEs available in the market, along with an analysis of their respective strengths and weaknesses within my scope of knowledge.

