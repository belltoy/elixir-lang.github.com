---
layout: getting-started
title: 介绍
redirect_from: /getting_started/1.html
---

# {{ page.title }}

{% include toc.html %}

欢迎！

在这个教程中我们将讲解关于 Elixir 的基础，该语言的语法，如何定义模块，如何处理常见的数据结构的特性，以及更多。本章的重点是确认 Elixir 正确安装，并且能够成功运行 Elixir 的交互式 shell，即 iex。

我们要求以下版本：

  * Elixir - Version 1.0.0 或以上
  * Erlang - Version 17.0 或以上

我们开始吧！

> 如果你在这个教程或者网站上发现了任何错误， [请提交错误报告或者在我们的 issue tracker 上发起一个 pull request](https://github.com/elixir-lang/elixir-lang.github.com) 。

## 安装

如果你还没安装 Elixir，请稳步到我们的 [安装页面](/install.html) 。一旦完成安装，你就能够通过执行 `elixir -v` 来查看当前的 Elixir 版本。

## 交互模式

当安装 Elixir 的时候，你将会有三个新的可执行命令: `iex`，`elixir`，`elixirc`。如果你是从源代码编译 Elixir 或者通过预编译的包来安装，你会在 `bin` 目录中找到这些命令。

现在，我们从运行 `iex` 开始（或者假如你是在 Windows 上，运行 `iex.bat`），它是交互式 Elixir （Interactive Elixir）的缩写。在交互模式下，我们能输入任意的 Elixir 表达式并获得它的结果。我们通过一些基本的表达式来热热身吧：

```iex
Interactive Elixir - press Ctrl+C to exit (type h() ENTER for help)

iex> 40 + 2
42
iex> "hello" <> " world"
"hello world"
```

看起来我们已经就绪了！从下一章节开始，我们将在接下来的章节中大量使用交互式 shell，来帮助我们熟悉语言结构和基本类型。

> 注意：如果你是在 Windows 上，你也可以试试 `iex --werl`，它可以根据你所使用的控制台提供一个更好的体验。

## 运行脚本

在熟悉了语言的基础之后，你可能会想尝试写一些简单的程序。可以把 Elixir 代码放在一个文件中，然后用 `elixir` 来执行它：

```bash
$ cat simple.exs
IO.puts "Hello world
from Elixir"

$ elixir simple.exs
Hello world
from Elixir
```
稍后我们将会学到如何编译 Elixir 代码（在 [第 8 章](/getting-started/modules.html) 中）以及如何使用 Mix 构建工具（在 [Mix & OTP 指南](/getting-started/mix-otp/introduction-to-mix.html) 中）。现在，让我们继续 [第 2 章](/getting-started/basic-types.html) 。
