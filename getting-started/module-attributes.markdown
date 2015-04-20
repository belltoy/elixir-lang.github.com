---
layout: getting-started
title: 模块属性
redirect_from: /getting_started/14.html
---

# {{ page.title }}

{% include toc.html %}

Elixir 中的模块属性有三个作用：

1. 它们为模块提供注释，通常包含了对用户和 <abbr title="Virtual Machine">VM</abbr> 有用的信息。
2. 它们可以作为常量。
3. 它们可以在编译时作为模块的临时存储。

让我们一个个来看。

## 作为注释

Elixir 是从 Erlang 中带来模块属性的概念。例如：

```elixir
defmodule MyServer do
  @vsn 2
end
```

在上面的例子中，我们明确地为那个模块设定了版本属性。 `@vsn` 是用于 Erlang <abbr title="Virtual Machine">VM</abbr> 中的代码重载机制，用来检查一个模块是否已经更新。如果没有指定版本，版本号就是模块函数的 MD5 校验值。

Elixir 有一些保留的属性。这里只是其中最常用的一些：

* `@moduledoc` - 为当前模块提供文档。
* `@doc` - 为紧跟着这个属性之后的函数或者宏提供文档。
* `@behaviour` - （注意是英式英语拼写）用于指定一个 <abbr title="Open Telecom Platform">OTP</abbr> 或者用户自定义的行为。
* `@before_compile` - 提供一个在模块编译之前会被调用的 hook。 这使得可以在编译之前为模块注入函数。

`@moduledoc` 和 `@doc` 是目前为止最常用的属性，我们建议你多使用它们。 Elixir 把文档当作一等公民，并提供了很多用于访问文档的函数。

让我们回头看看在之前的章节中定义的 `Math` 模块，为它添加一些文档，并保存在 `math.ex` 文件中：

```elixir
defmodule Math do
  @moduledoc """
  Provides math-related functions.

  ## Examples

      iex> Math.sum(1, 2)
      3

  """

  @doc """
  Calculates the sum of two numbers.
  """
  def sum(a, b), do: a + b
end
```

Elixir 提供使用 markdown 和 heredoc 来编写高可读性的文档。 heredoc 是多行字符串，它们以三个引号作为起止，保持内部文本的格式。我们可以直接从 IEx 中访问任意已编译的模块的文档：

```bash
$ elixirc math.ex
$ iex
```

```iex
iex> h Math # Access the docs for the module Math
...
iex> h Math.sum # Access the docs for the sum function
...
```

我们还提供了一个名为 [ExDoc](https://github.com/elixir-lang/ex_doc) 的工具，用于生成 HTML 格式的文档。

我们可以在 [Module](/docs/stable/elixir/#!Module.html) 文档中看到支持的属性的完整列表。 Elixir 还用属性来定义 [typespecs](/docs/stable/elixir/#!Kernel.Typespec.html)，通过：

* `@spec` - 为函数提供一个规范。
* `@callback` - 为行为的回调提供一个规范。
* `@type` - 定义一个在 `@spec` 中用到的类型。
* `@typep` - 定义一个在 `@spec` 中用到的私有类型。
* `@opaque` - 定义一个在 `@spec` 中用到的 opaque 类型。

本节包含了内建的属性。然而，属性也可以被开发者使用，或者被库扩展为支持自定义的行为。

## 作为常量

Elixir 开发者经常将模块属性当作常量：

```elixir
defmodule MyServer do
  @initial_state %{host: "147.0.0.1", port: 3456}
  IO.inspect @initial_state
end
```

> 注意： 不像 Erlang，用户定义的属性默认不是存储在模块里的。它们的值只有在编译期才存在。开发者可以通过调用 [`Module.register_attribute/3`](/docs/stable/elixir/#!Module.html#register_attribute/3) 来配置一个属性使得它接近 Erlang 中的行为。

尝试访问一个未定义的属性会打印出一个警告信息：

```elixir
defmodule MyServer do
  @unknown
end
warning: undefined module attribute @unknown, please remove access to @unknown or explicitly set it to nil before access
```

最后，属性也可以在函数内部访问：

```elixir
defmodule MyServer do
  @my_data 14
  def first_data, do: @my_data
  @my_data 13
  def second_data, do: @my_data
end

MyServer.first_data #=> 14
MyServer.second_data #=> 13
```

注意到，从函数内部读取一个属性会拿到当前值的快照。换言之，该值是在编译期读取的而不是在运行时。正如我们将要看到的，这使得属性可以用于在编译期作为临时存储。

## 作为临时存储

Elixir 组织的其中一个项目 [`Plug` 项目](https://github.com/elixir-lang/plug) ，它旨在成为 Elixir 中构建 web 库和框架的通用基础。

Plug 库也允许开发者定义他们自己的能运行在 web server 中的 plug：

```elixir
defmodule MyPlug do
  use Plug.Builder

  plug :set_header
  plug :send_ok

  def set_header(conn, _opts) do
    put_resp_header(conn, "x-header", "set")
  end

  def send_ok(conn, _opts) do
    send(conn, 200, "ok")
  end
end

IO.puts "Running MyPlug with Cowboy on http://localhost:4000"
Plug.Adapters.Cowboy.http MyPlug, []
```

在上面的例子中，我们用了 `plug/1` 宏来连接那些会在一个 web 请求发起的时候被调用的函数。从内部来说，每次你调用 `plug/1`， Plug 库会将传入的参数保存在 `@plugs` 属性中。就在模块被编译的时候，Plug 运行一个定义了一个方法 （`call/2`） 的回调，用于处理 http 请求。这个方法会按顺序运行在 `@plugs` 里的所有 plug。

为了理解底层代码，我们需要宏，所以我们将在元编程教程中再次学习这个模式。然而这里关注的是如何使用模块属性作为存储来允许开发者创建 DSL。

另一个来自 ExUnit 框架的例子，它使用模块属性作为注释和存储：

```elixir
defmodule MyTest do
  use ExUnit.Case

  @tag :external
  test "contacts external service" do
    # ...
  end
end
```

ExUnit 中的 tag 是用来注释测试用例的。 Tag 可以在之后用来过滤测试用例。例如，你可以防止在你的机器上运行外部测试，因为它们很慢且依赖其它服务，但它们仍然可以在你的构建系统上启用。

我们希望本节能够让你领略到 Elixir 如何支持元编程以及模块属性如何在其中扮演一个重要的角色。

在接下来的章节中，在继续深入异常处理和其它像 sigil 和列表解析的结构之前，我们将先探索 struct 和协议。
