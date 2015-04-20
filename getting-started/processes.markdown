---
layout: getting-started
title: 进程
redirect_from: /getting_started/11.html
---

# {{ page.title }}

{% include toc.html %}

在 Elixir 中， 所有的代码运行在进程里。 每个进程彼此隔离，并发地运行，并通过消息传递来通讯。进程不仅仅是 Elixir 中并发的基础，它们还提供了用于创建分布式和容错性程序的手段。

Elixir 中的进程不应该与操作系统进程混淆。与许多其它编程语言中的线程不同，就消耗的内存和 CPU 来说， Elixir 中的进程是极为轻量的。正因如此，同时运行成千上万个进程并不少见。

本章中，我们将学习关于创建新进程的基本结构，并且在不同的进程之间发送和接收消息。

## `spawn`

创建一个新进程的基本方式是使用自动引入的 `spawn/1` 函数：

```iex
iex> spawn fn -> 1 + 2 end
#PID<0.43.0>
```

`spawn/1` 接收一个函数，它会在另一个进程中执行。

注意 `spawn/1` 返回一个 PID （进程标识符）。在这个时候，你创建的进程很可能已经死了。刚创建的进程会执行给定的函数，执行完成然后退出：

```iex
iex> pid = spawn fn -> 1 + 2 end
#PID<0.44.0>
iex> Process.alive?(pid)
false
```

> 注意： 你很可能会得到一个与我们教程中得到的不同的进程标识符。

我们可以通过调用 `self/0` 来得到当前进程的 PID：

```iex
iex> self()
#PID<0.41.0>
iex> Process.alive?(self())
true
```

当我们能够在进程之间发送和接收消息的时候，它们会变得更加有趣。

## `send` 和 `receive`

我们可以用 `send/2` 来给一个进程发送消息，用 `receive/1` 来接收消息：

```iex
iex> send self(), {:hello, "world"}
{:hello, "world"}
iex> receive do
...>   {:hello, msg} -> msg
...>   {:world, msg} -> "won't match"
...> end
"world"
```

当一个消息被发送到一个进程的时候，这个消息会被存储在这个进程的邮箱里。 `receive/1` 块会在当前进程的邮箱里查找一个与任意给定的模式相匹配的消息。 `receive/1` 支持守卫和多分句，如 `case/2`。

如果邮箱中没有消息能够与任意一个模式相匹配，当前的进程会一直等待直到一个匹配的消息到达。也可以指定一个超时时间：

```iex
iex> receive do
...>   {:hello, msg}  -> msg
...> after
...>   1_000 -> "nothing after 1s"
...> end
"nothing after 1s"
```

如果你已经预计一个消息已经在邮箱里，可以设置一个超时时间设置为 0 。

让我们把以上的合起来，并在进程中发送消息：

```iex
iex> parent = self()
#PID<0.41.0>
iex> spawn fn -> send(parent, {:hello, self()}) end
#PID<0.48.0>
iex> receive do
...>   {:hello, pid} -> "Got hello from #{inspect pid}"
...> end
"Got hello from #PID<0.48.0>"
```

如果是在 shell 中，你会发现帮助函数 `flush/0` 非常有用。它清空并打倒邮箱中的所有消息。

```iex
iex> send self(), :hello
:hello
iex> flush()
:hello
:ok
```

## 链接

在 Elixir 中，最常见的创建进程的方式实际上是通过 `spawn_link/1` 。在我们展示 `spawn_link/1` 的例子前，让我们尝试看看当一个进程失败的时候会发生什么：

```iex
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Error in process <0.58.0> with exit value: ...
```

仅仅是打印出错误日志，用于创建的进程仍然在运行。这是因为进程是彼此隔离的。如果我们希望一个进程的失败能够传递到另外一个进程，我们应该把它们链接起来。这就要用到 `spawn_link/1` 了：

```iex
iex> spawn_link fn -> raise "oops" end
#PID<0.41.0>

** (EXIT from #PID<0.41.0>) an exception was raised:
    ** (RuntimeError) oops
        :erlang.apply/2
```

当 shell 中有错误发生，shell 会自动捕获到错误并打印出格式化的信息。为了理解我们代码中真正发生了什么，让我们把 `spawn_link/1` 写到文件中并运行：

```iex
# spawn.exs
spawn_link fn -> raise "oops" end

receive do
  :hello -> "let's wait until the process fails"
end
```

这一次，子进程出错退出，父进程也挂掉了，因为他们是链接在一起的。链接也可以通过调用 `Process.link/1` 手动完成。我们推荐你看看 [`Process` 模块](/docs/stable/elixir/#!Process.html) 中提供的其它函数。

当构建一个高容错系统的时候，进程和链接扮演着一个重要的角色。在 Elixir 的应用中，我们常常把我们的进程链接到监控进程，当一个进程死亡，它会检测到并在原地重启一个新的进程。之所以能这么做，是因为进程之间是相互隔离的，而且默认不共享任何数据。当进程是相互隔离的时候，一个进程的错误不会影响到其它的进程。

其它一些语言可能会要求我们捕获和处理异常，而在 Elixir 中我们实际上可以不用这么做，就让出错的进程出错好了，因为我们可以让监控进程来正确得重启我们的系统。「Failing fast」 是编写 Elixir 软件的一个常见的哲学思想。

在继续下一章之前，让我们先来看一个在 Elixir 中创建进程的最常见的用例。

## Tasks

在上一节中当我们的进程崩溃时，你也许已经注意到了错误信息很少：

```iex
iex> spawn fn -> raise "oops" end
#PID<0.58.0>

[error] Error in process <0.58.0> with exit value: ...
```

使用 `spawn/1` 和 `spawn_link/1` 函数时，错误信息由虚拟机直接生成，所以比较简单而且缺少细节。在实践中，开发者们更愿意使用 Task 模块中的函数，它们更加明确， `Task.start/1` 和 `Task.start_link/1`：

```iex
iex(1)> Task.start fn -> raise "oops" end
{:ok, #PID<0.55.0>}

15:22:33.046 [error] Task #PID<0.55.0> started from #PID<0.53.0> terminating
Function: #Function<20.90072148/0 in :erl_eval.expr/5>
    Args: []
** (exit) an exception was raised:
    ** (RuntimeError) oops
        (elixir) lib/task/supervised.ex:74: Task.Supervised.do_apply/2
        (stdlib) proc_lib.erl:239: :proc_lib.init_p_do_apply/3
```

除了提供了更好的错误日志，它们还有点区别： `start/1` 和 `start_link/1` 回返 `{:ok, pid}` 而不仅仅是 PID。 这就允许 Task 被用在监控树中。更进一步，task 提供了方便的函数例如 `Task.async/1` 和 `Task.await/1`，以及用于分布式的函数。

我们将在 **Mix 和 OTP 教程** 中探讨那些函数，现在我们只要记住使用 Task 来得到更好的日志。

## 状态

到目前为止我们还没有讲到状态。假如你要构建一个需要状态的应用，例如保存应用的配置，或者你需要解析一个文件并保持在内存中，那么你要把它存在哪呢？

对于这个问题，进程是最常见的一个答案。我们可以写一个无限循环的进程，维护状态，并发送和接收消息。举个例子，我们在 `kv.exs` 文件中写一个模块，启动一个新的进程，像 key-value 存储那样工作：

```elixir
defmodule KV do
  def start_link do
    Task.start_link(fn -> loop(%{}) end)
  end

  defp loop(map) do
    receive do
      {:get, key, caller} ->
        send caller, Map.get(map, key)
        loop(map)
      {:put, key, value} ->
        loop(Map.put(map, key, value))
    end
  end
end
```

注意到 `start_link` 函数只是创建一个新的进程，从一个空的 map 开始执行 `loop/1` 函数。然后 `loop/1` 函数等待消息，并执行相应的动作。假如收到一个 `:get` 消息，它会向调用者发回一个结果消息，然后等待新的消息。当收到 `:put` 消息的时候，实际上是用给定的 `key` 和 `value` 更新了 map 并调用了 `loop/1`。

让我们运行 `iex kv.exs` 试一下：

```iex
iex> {:ok, pid} = KV.start_link
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
nil
```

一开始，进程的 map 没有键，所以发送一个 `:get` 消息并清空当前进程的邮箱，返回 `nil`。让我们传一个 `:put` 消息再试一下：

```iex
iex> send pid, {:put, :hello, :world}
#PID<0.62.0>
iex> send pid, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

注意进程是如何保存一个状态，我们可以通过向这个进程发送消息来获取和更新这个状态。事实上，任何进程只要知道上面的 `pid` 都可以向它发送消息和操纵状态。

也可以注册这个 `pid`， 给它一个名字，这样允许知道这个名字的任何进程都可以向它发送消息：

```iex
iex> Process.register(pid, :kv)
true
iex> send :kv, {:get, :hello, self()}
{:get, :hello, #PID<0.41.0>}
iex> flush
:world
```

在进程中使用状态和为进程注册一个名字，这在在 Elixir 应用中是很常见的模式。然而，大多数时候，我们不会手动实现上面的模式，而是使用 Elixir 提供的众多抽象。例如， Elixir 提供了 [代理](/docs/stable/elixir/#!Agent.html) 模块，它对状态进行了简单的抽象：

```iex
iex> {:ok, pid} = Agent.start_link(fn -> %{} end)
{:ok, #PID<0.72.0>}
iex> Agent.update(pid, fn map -> Map.put(map, :hello, :world) end)
:ok
iex> Agent.get(pid, fn map -> Map.get(map, :hello) end)
:world
```

给 `Agent.start_link/2` 函数传递一个 `:name` 选项，它会自动用这个名字注册。除了代理， Elixir 还提供了一个用于构建通用服务器（称作 GenServer）的 API，通用事件管理器和事件处理器（称作 GenEvent），任务等等，底层都是由进程提供的。这些 API，加上监控树，将会在 **Mix 和 OTP 教程** 中通过从头到尾构建一个完整的 Elixir 应用来了解。

现在，让我们先继续了解 Elixir 中的 I/O 世界。
