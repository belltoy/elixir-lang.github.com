---
layout: getting-started
title: try, catch 和 rescue
redirect_from: /getting_started/19.html
---

# {{ page.title }}

{% include toc.html %}

Elixir 有三种错误机制： 错误， 抛出和退出。本章中我们将一一探索它们，包括应该何时使用它们。

## 错误

错误（或者 *异常*）用于当代码中异常的情况发生的时候。尝试把一个数字与原子相加，就会得到一个简单的错误：

```iex
iex> :foo + 1
** (ArithmeticError) bad argument in arithmetic expression
     :erlang.+(:foo, 1)
```

可以用 `raise/1` 在任何时候引发一个运行时错误：

```iex
iex> raise "oops"
** (RuntimeError) oops
```

其它错误可以用 `raise/2` 来引发，传入错误名字和一个关键词列表参数：

```iex
iex> raise ArgumentError, message: "invalid argument foo"
** (ArgumentError) invalid argument foo
```

你也可以通过创建一个模块并在其中使用 `defexception` 结构来定义你自己的错误；这种方法，你会创建一个与模块名相同名字的错误。最常见的例子是定义一个带 message 字段的自定义异常：

```iex
iex> defmodule MyError do
iex>   defexception message: "default message"
iex> end
iex> raise MyError
** (MyError) default message
iex> raise MyError, message: "custom message"
** (MyError) custom message
```

错误可以用 `try/rescue` 结构 **挽救**：

```iex
iex> try do
...>   raise "oops"
...> rescue
...>   e in RuntimeError -> e
...> end
%RuntimeError{message: "oops"}
```

上面的例子中，挽救了一个运行时错误并返回这个错误本身，然后在 `iex` 会话中打印出来。然而在实践中，Elixir 开发者很少使用 `try/rescue` 结构。例如，当一个文件无法被成功打开时，许多语言会强制你挽救错误。相反 Elixir 提供了一个 `File.read/1` 函数，它会返回一个包含关于文件是否被成功打开的信息的元组：

```iex
iex> File.read "hello"
{:error, :enoent}
iex> File.write "hello", "world"
:ok
iex> File.read "hello"
{:ok, "world"}
```

这里没有 `try/rescue`。如果你要处理打开文件的不同结果，你可以简单地用 `case` 结构使用模式匹配：

```iex
iex> case File.read "hello" do
...>   {:ok, body}      -> IO.puts "Success: #{body}"
...>   {:error, reason} -> IO.puts "Error: #{reason}"
...> end
```

最终还是由你的应用来决定打开一个文件出现的错误是否是异常。这也是为什么 Elixir 没有在 `File.read/1` 和其它许多函数中强制使用异常。取而代之的是，它把选择最佳处理方式的决定权交给开发者。

有些情况下你期待一个文件存在（如果不存在那个文件，确实就是一个错误），你可以简单地使用 `File.read!/1`：

```iex
iex> File.read! "unknown"
** (File.Error) could not read file unknown: no such file or directory
    (elixir) lib/file.ex:305: File.read!/1
```

许多标准库中的函数都遵循这种模式，存在一个引发异常的副本而不是一个返回用于匹配的元组。惯例是创建一个返回 `{:ok, result}` 或 `{:error, reason}` 元组的函数（`foo`）和另一个与 `foo` 接受相同参数但如果有错误则引发一个异常的函数（`foo!`，相同的名字但多后面一个 `!`）。 如果执行正常，`foo!` 应该返回结果（不是包装在元组中）。 对于这个惯例， [`File` 模块](/docs/stable/elixir/#!File.html) 是一个很好的例子。

有 Elixir 中，我们避免使用 `try/rescue`，因为 **我们不用错误来进行流程控制**。我们仅按字面对待错误：它们就是留给意料之外和/或异常情况的。如果你确实需要流程控制结构，应用使用 *throws*。这正是我们接下来要看到的。

## 抛出

在 Elixir 中，一个值可以被抛出稍后再被捕获。 `throw` 和 `catch` 用于除了用 `throw` 和 `catch` 之外无法获取一个值的情况。

那些情况在实践中很不常见，除非在与一些没有提供良好 API 的库对接的时候。举个例子，试想 `Enum` 模块没有提供查找值的 API，我们又需要在一个数字列表中找到第一个 13 的倍数：

```iex
iex> try do
...>   Enum.each -50..50, fn(x) ->
...>     if rem(x, 13) == 0, do: throw(x)
...>   end
...>   "Got nothing"
...> catch
...>   x -> "Got #{x}"
...> end
"Got -39"
```

然而 `Enum` *确实* 提供了适当的 API ，实践中使用 `Enum.find/2` 就行了：

```iex
iex> Enum.find -50..50, &(rem(&1, 13) == 0)
-39
```

## 退出

所有的 Elixir 代码都运行在进程中，进程之间相互通信。当一个进程由于 「正常原因」死亡（例如，未处理异常）的时候，它会发出一个 `exit` 信号。一个进程也可以通过显示地向它发送一个退出信号来杀死；

```iex
iex> spawn_link fn -> exit(1) end
#PID<0.56.0>
** (EXIT from #PID<0.56.0>) 1
```

在上面的例子中，由于向它发送了一个值为 1 的 `exit` 信号，被链接的进程死亡。Elixir shell 自动处理那些信息并打出在终端里。

`exit` 也可以用 `try/catch` 来「捕获」：

```iex
iex> try do
...>   exit "I am exiting"
...> catch
...>   :exit, _ -> "not really"
...> end
"not really"
```

使用 `try/catch` 已经是不常见的，用它来捕获退出更是少见。

`exit` 信号对于由 Erlang <abbr title="Virtual Machine">VM</abbr> 提供的容错系统来说是一个重要的组成部分。进程一般运行在监控树里，监控树其实是等待被监控的进程的 `exit` 信号的进程。一旦收到一个退出信号，监控策略会被触发，然后重启被监控的进程。

这正是为什么这个监控系统使得在 Elixir 中的像 `try/catch` 和 `try/rescue` 这种结构如此少见的原因。与其挽救一个错误，我们更愿意让它「快速失败」，因为监控树会保证我们的应用在出现错误的时候，会回到一个已知的初始状态。

## After

有时候在一些可能会引发错误操作之后，有必要确保一个资源被清理。 `try/after` 结构允许你这么做。例如，我们可以用 `try/after` 块打开一个文件并确保它会被关闭（甚至是出现一些错误）：

```iex
iex> {:ok, file} = File.open "sample", [:utf8, :write]
iex> try do
...>   IO.write file, "olá"
...>   raise "oops, something went wrong"
...> after
...>   File.close(file)
...> end
** (RuntimeError) oops, something went wrong
```

## 变量作用域

牢记在 `try/catch/rescue/after` 块中定义的变量不会泄漏到外部的上下文。这是因为 `try` 块可能失败，这样变量可能不会在第一个地方绑定。换句话说，这样的代码是无效的：

```iex
iex> try do
...>   from_try = true
...> after
...>   from_after = true
...> end
iex> from_try
** (RuntimeError) undefined function: from_try/0
iex> from_after
** (RuntimeError) undefined function: from_after/0
```

到这里我们结束了对 `try`， `catch` 和 `rescue` 的介绍。你会发现它们在 Elixir 中比在其它语言中更少使用，但在一些情况下，某个库或者某些特定的代码不「按规矩」出牌的时候，它们也许会有用。
