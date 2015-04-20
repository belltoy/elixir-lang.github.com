---
layout: getting-started
title: IO 和文件系统
redirect_from: /getting_started/12.html
---

# {{ page.title }}

{% include toc.html %}

本章对输入/输出机制和文件系统相关的任务作一个快速的介绍，以及相关的模块如 [`IO`](/docs/stable/elixir/#!IO.html)， [`File`](/docs/stable/elixir/#!File.html) 和 [`Path`](/docs/stable/elixir/#!Path.html)。

我们原本的规划是在这个入门教程中更早地进入本章节。然而，我们注意到 IO 系统提供了一个很好的机会来轻松得阐明一些 Elixir 和 <abbr title="Virtual Machine">VM</abbr> 的哲学和独特之处。

## `IO` 模块

`IO` 模块是 Elixir 中用于读写标准输入/输出（`:stdio`），标准错误（`:stderr`），文件和其它 IO 设备的主要机制。这个模块的用法非常简单明了：

```iex
iex> IO.puts "hello world"
hello world
:ok
iex> IO.gets "yes or no? "
yes or no? yes
"yes\n"
```

默认情况下，IO 模块中的函数从标准输入读，写到标准输出。我们可以通过传入 `:stderr` 作为参数来修改这个默认行为（写到标准错误设备）：

```iex
iex> IO.puts :stderr, "hello world"
hello world
:ok
```

## `File` 模块

[`File`](/docs/stable/elixir/#!File.html) 模块包含了很多函数，允许我们像 IO 设备那样打开文件。默认情况下，文件以二进制模式打开，这需要开发者明确使用 `IO` 模块中的 `IO.binread/2` 和 `IO.binwrite/2` 函数：

```iex
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
iex> IO.binwrite file, "world"
:ok
iex> File.close file
:ok
iex> File.read "hello"
{:ok, "world"}
```

文件也可以用 `:utf8` 编码打开，它告诉 `File` 模块将从文件中读取到的字节解释成 UTF-8 编码的字节。

除了用于打开，读取和写入文件的函数， `File` 模块还有许多针对文件系统的函数。这些函数都是以 UNIX 对应的命令来命名的。例如， `File.rm/1` 用于删除文件， `File.mkdir/1` 用于创建目录， `File.mkdir_p/1` 用于创建目录和它的父目录。甚至还有 `File.cp_r/2` 和 `File.rm_rf/2` 分别用于递归地复制和删除文件和目录（即复制和删除目录里的内容）。

你还要注意到 `File` 模块中的函数有两个变体：一个「普通」版和另一个相同名字但以感叹号（`!`）结尾的版本。例如，上面例子中我们用 `File.read/1` 从 `"hello"` 文件中读取。我们也可以使用 `File.read!/1`：

```iex
iex> File.read "hello"
{:ok, "world"}
iex> File.read! "hello"
"world"
iex> File.read "unknown"
{:error, :enoent}
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
```

注意到如果文件不存在，带 `!` 的版本会引发一个错误。如果你要用模式匹配来处理不同的情况，就选择没有 `!` 的版本：

```elixir
case File.read(file) do
  {:ok, body}      -> # do something with the `body`
  {:error, reason} -> # handle the error caused by `reason`
end
```

然而，如果你期待目标文件存在，感叹号版本会更有用，因为它会引发一个有意义的错误信息。避免这样写：

```elixir
{:ok, body} = File.read(file)
```

假如出错， `File.read/1` 会返回 `{:error, reason}` 然后模式匹配会失败。你还是会得到预期的结果（引发一个错误），但是错误信息描述的却是模式不匹配（这样会隐藏错误真正的原因）。

如果你不想处理可能的错误（即，你想让它向上传递），选择 `File.read!/1`。

## `Path` 模块

`File` 模块中的大部分函数期待路径作为参数。更常见的是，那些路径是二进制。 [`Path`](/docs/stable/elixir/#!Path.html) 模块提供了处理这些路径的工具：

```iex
iex> Path.join("foo", "bar")
"foo/bar"
iex> Path.expand("~/hello")
"/Users/jose/hello"
```

如果仅仅只是操作二进制，请不要使用 `Path` 模块中的函数，因为 `Path` 模块负责透明地处理不同操作系统。例如， `Path.join/2` 在类 Unix 的系统上用斜线 （`/`）连接路径，而在 Windows 上用反斜线（`\\`）。

目前为止我们已经涵盖了 Elixir 提供的用于处理 IO 和与文件系统交互的主要模块。下一段落中，我们将讨论一些关于 IO 的高级话题。这些段落并不是编写 Elixir 代码必需的知识，所以请随意跳过这些内容，但是它们提供了关于 <abbr title="Virtual Machine">VM</abbr> IO 系统是如何实现的一个很好的概述，以及其它一些趣闻。

## 进程和 group leader

你也许已经注意到 `File.open/2` 返回一个像 `{:ok, pid}` 的元组：

```iex
iex> {:ok, file} = File.open "hello", [:write]
{:ok, #PID<0.47.0>}
```

那是因为 `IO` 模块实际上是通过进程工作的（请阅读 [第 11 章](/getting-started/processes.html)）。 当你写 `IO.write(pid, binary)` 的时候， `IO` 模块会发送一个消息到 `pid` 对应的进程，告诉它要做的操作。让我们来看看假如我们用自己的进程会发生什么：

```iex
iex> pid = spawn fn ->
...>  receive do: (msg -> IO.inspect msg)
...> end
#PID<0.57.0>
iex> IO.write(pid, "hello")
{:io_request, #PID<0.41.0>, #PID<0.57.0>, {:put_chars, :unicode, "hello"}}
** (ErlangError) erlang error: :terminated
```

调用 `IO.write/2` 之后，我们可以看到 `IO` 模块发出的消息被打印出来 （一个四元的元组）。接着我们能看到它失败了，因为我们没有返回 `IO` 模块期待的一些结果。

[`StringIO`](/docs/stable/elixir/#!StringIO.html) 模块提供了字符串上的 `IO` 设备消息的实现：

```iex
iex> {:ok, pid} = StringIO.open("hello")
{:ok, #PID<0.43.0>}
iex> IO.read(pid, 2)
"he"
```

通过用进程对 IO 设备建模， Erlang <abbr title="Virtual Machine">VM</abbr> 允许同一网络中的不同节点交换文件进程来在节点之间读写文件。在所有的 IO 设备中，有一个对每个进程来说都是特殊的： **group leader**。

当你向 `:stdio` 写入的时候，你实际上是发送一个消息到 group leader，通过它向标准输入文件描述符写入：

```iex
iex> IO.puts :stdio, "hello"
hello
:ok
iex> IO.puts Process.group_leader, "hello"
hello
:ok
```

可以为每个进程配置 group leader，用在不同的场景中。例如，当在远程终端中执行代码的时候，保证远程节点上的消息会被重定向回发起请求的终端。

## `iodata` 和 `chardata`

在上面所有的例子中，我们用二进制来写文件。在 ["二进制，字符串和字符列表"](/getting-started/binaries-strings-and-char-lists.html) 这章中，我们提到了字符串实际上就是简单的字节，字符列表是码点的列表。

`IO` 和 `File` 模块中的函数允许列表作为参数。不仅如此，它们还允许包含了列表，整数和二进制的混合列表作为参数：

```iex
iex> IO.puts 'hello world'
hello world
:ok
iex> IO.puts ['hello', ?\s, "world"]
hello world
:ok
```

然而，这需要一些注意。一个列表可以列表一堆字节或者一堆字符，它们依赖于 IO 设备的编码。如果打开一个文件时候没有指定编码，文件就会在原始模式下，必须使用 `IO` 模块中以 `bin*` 开头的函数。那些函数需要一个 `iodata` 作为参数；即，它们期待一个用整数表示字节和二进制的列表作为参数。

另一方面， `:stdio` 和以 `:utf8` 编码打开的文件则用 `IO` 模块中的其它函数处理。那些函数期待一个 `char_data` 作为参数，即，一个字符列表或者字符串。

虽然这是一个细微的区别，你只有在你想要列表给那些函数的时候才需要担心那些细节。二进制已经是用底层的字节来表示的了，所以它们总是在原始模式下。

到此，我们结束了 IO 设备和 IO 相关函数的教程。我们已经学习了四个 Elixir 模块 - [`IO`](/docs/stable/elixir/#!IO.html), [`File`](/docs/stable/elixir/#!File.html), [`Path`](/docs/stable/elixir/#!Path.html) 和 [`StringIO`](/docs/stable/elixir/#!StringIO.html)，还学习了 <abbr title="Virtual Machine">VM</abbr> 在底层是如何利用进程来实现 IO 机制的，以及如何使用 `chardata` 和 `iodata` 来进行 IO 操作。
