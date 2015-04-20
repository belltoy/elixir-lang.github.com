---
title: "安装 Elixir"
section: install
layout: default
---

# {{ page.title }}

{% include toc.html %}

安装 Elixir 最快速的方法是通过发行版或者使用一个可用的安装程序。如果没有，我们推荐预编译的包或者从源代码编译。

注意 Elixir 要求 Erlang 17.0 或更高的版本。以下的许多指令会自动为你安装 Erlang。如果没有，请阅读下面的 "安装 Erlang" 一节。

## 发行版

选择你的操作系统和工具。

### Mac OS X

  * Homebrew
    * 更新你的 homebrew 到最新： `brew update`
    * 运行: `brew install elixir`
  * Macports
    * 运行: `sudo port install elixir`

### Unix (和 Unix-like)

  * Arch Linux (Community repo)
    * 运行: `pacman -S elixir`
  * openSUSE (和 SLES 11 SP3+)
    * 添加 Erlang devel repo: `zypper ar -f obs://devel:languages:erlang/ erlang`
    * 运行: `zypper in elixir`
  * Gentoo
    * 运行: `emerge --ask dev-lang/elixir`
  * Fedora 17 和更新的版本
    * 运行: `yum install elixir`
  * FreeBSD
    * From ports: `cd /usr/ports/lang/elixir && make install clean`
    * From pkg: `pkg install elixir`
  * Ubuntu 12.04 和 14.04 / Debian 7
    * 添加 Erlang Solutions repo: `wget http://packages.erlang-solutions.com/erlang-solutions_1.0_all.deb && sudo dpkg -i erlang-solutions_1.0_all.deb`
    * 运行: `sudo apt-get update`
    * 运行: `sudo apt-get install elixir`

### Windows

  * Web installer
    * [下载安装程序](http://s3.hex.pm/elixir-websetup.exe)
    * 点击 next, next, ..., finish
  * Chocolatey
    * `cinst elixir`

这些发行版很可能会自动为你安装 Erlang。如果没有，请阅读下面的 [安装 Erlang](/install.html#installing-erlang) 一节。

## 预编译包

Elixir 为每个版本提供了一个预编译包。首先 [安装 Erlang](/install.html#installing-erlang) ，然后下载和解压 [最新版本的 Precompiled.zip 文件](https://github.com/elixir-lang/elixir/releases/) 。

一旦解压好，你就可以从 `bin` 目录中运行 `elixir` 和 `iex` 命令了，但是为了简化开发，我们建议你 [添加 Elixir 的 bin 目录到你的 PATH 环境变量](#setting-path-environment-variable) 。

## 从源代码编译（Unix 和 MinGW）

通过几个步骤，你就可以下载并编译 Elixir。首先是 [安装 Erlang](/install.html#installing-erlang) 。

然后你可以下载 [最新的版本](https://github.com/elixir-lang/elixir/releases/) ，解压，然后在解压的目录中运行 `make`（注意：如果你是在 Windows 上， [阅读这个页面来设置你的 Elixir 编译环境](https://github.com/elixir-lang/elixir/wiki/Windows)）。

编译完成之后，你就可以从 bin 目录中运行 `elixir` 和 `iex` 命令了。 为了简化开发， 我们建议你 [添加 Elixir 的 bin 目录到你的 PATH 环境变量](#setting-path-environment-variable) 。

如果你想尝鲜，你可以从主分支编译：

```bash
$ git clone https://github.com/elixir-lang/elixir.git
$ cd elixir
$ make clean test
```

如果测试都通过，你就大功告成了。否则，你可以 [在 Github 的 issues tracker](https://github.com/elixir-lang/elixir) 报告一个 issue。

## 安装 Erlang

Elixir 唯一的先决条件是 Erlang 17.0 或者更高版本，可以从 [预编译包](https://www.erlang-solutions.com/downloads/download-erlang-otp) 轻松安装。如果你想直接从源代码安装，可以从 [Erlang 官网](http://www.erlang.org/download.html) 或者按照 [Riak 文档](http://docs.basho.com/riak/1.3.0/tutorials/installation/Installing-Erlang/) 中优秀的教程。

对于 Windows 开发者，我们推荐预编译包。 在 Unix 平台上或许可以通过众多发行版工具之一来安装 Erlang。

Erlang 安装完成后，你应该可以打开命令行，输入 `erl` 检查 Erlang 版本。你将会看到如下的一些信息：

    Erlang/OTP 17 (erts-6) [64-bit] [smp:2:2] [async-threads:0] [hipe] [kernel-poll:false]

注意，根据你安装 Erlang 的方式不同， Erlang 的二进制文件可能不在你的 PATH 中。请确保 Erlang 二进制文件在你的 [PATH](http://en.wikipedia.org/wiki/Environment_variable) 中，否则 Elixir 不会工作！


## 设置 PATH 环境变量

为了简化开发，我们强烈建议将 Elixir 的 bin 目录添加到你的 PATH 环境变量中。

在 **Windows** 上，请参照 [不同版本的指令](http://www.computerhope.com/issues/ch000549.htm) 说明。

在 **Unix 系统** 上，你需要 [找到你的 shell 配置文件](http://unix.stackexchange.com/a/117470/101951) ，然后把下面这行添加到这个文件的末尾，它指向你的 Elixir 的安装路径：

```bash
export PATH="$PATH:/path/to/elixir/bin"
```