---
layout: getting-started
title: 类型声明和行为
redirect_from: /getting_started/20.html
---

# {{ page.title }}

{% include toc.html %}

## 类型和声明

Elixir 是一个动态类型语言，所以 Elixir 中所有的类型都是在运行时推断出来的。尽管如此， Elixir 附带了 **类型声明** 标记，它用于：

1. 声明自定义的数据类型；
2. 声明类型的函数签名（规格）。

### 函数规格

Elixir 默认提供了一些基本类型，像 `integer` 或 `pid`，还有更复杂的类型：例如 `round/1` 函数，它把浮点数舍位成近似的整数，接受一个 `number` 作为参数（一个 `integer` 或 `float`）并返回一个 `integer`。正如你 [在它的文档](/docs/stable/elixir/#!Kernel.html#round/1) 里看到的，`round/1` 的类型签名是这么写的：

```
round(number) :: integer
```

`::` 意思是它左侧的函数 *返回* 的是它右侧所示的类型的一个值。函数规格是用 `@spec` 指令来写的，放在函数定义之前。 `round/1` 函数可以如下面这样写：

```elixir
@spec round(number) :: integer
def round(number), do: # implementation...
```

Elixir 还支持混合类型。例如，一个整数列表的类型是 `[integer]`。你可以在 [类型规格文档](/docs/stable/elixir/#!Kernel.Typespec.html) 中看到 Elixir 提供的所有类型。

### 定义自定义类型

虽然 Elixir 提供了很多有用的内置类型，但它也很方便在适当的时候定义自定义类型。这可以通过在定义模块的时候使用 `@type` 指令定义来定义。

假设我们有一个 `LousyCalculator` 模块，它提供了有用的算术操作（求和，乘积等等），但是它不是返回数字，而是返回一个元组，它的第一个元素是该操作结果，第二个元素是一个随机字符串。

```elixir
defmodule LousyCalculator do
  @spec add(number, number) :: {number, String.t}
  def add(x, y), do: {x + y, "You need a calculator to do that?!"}

  @spec multiply(number, number) :: {number, String.t}
  def multiply(x, y), do: {x * y, "Jeez, come on!"}
end
```

正如你在例子中所看到的，元组是一个混合类型，而且每个元组是用它内部的类型标识的。为了理解为什么 `String.t` 不写成 `string`，再看一次 [类型规格文档](/docs/stable/elixir/#!Kernel.Typespec.html)。

定义函数规格可以这么做，但是当我们一次双一次得重复输入 `{number, String.t}` 的时候，它就变得很烦。我们可以用 `@type` 指令来声明我们自己的自定义类型。

```elixir
defmodule LousyCalculator do
  @typedoc """
  Just a number followed by a string.
  """
  @type number_with_offense :: {number, String.t}

  @spec add(number, number) :: number_with_offense
  def add(x, y), do: {x + y, "You need a calculator to do that?!"}

  @spec multiply(number, number) :: number_with_offense
  def multiply(x, y), do: {x * y, "Jeez, come on!"}
end
```

`@typedoc` 指令类似 `@doc` 和 `@moduledoc` 指令，它用于为自定义类型标注文档。

通过 `@type` 定义的自定义类型会被导出，在定义它们模块之后是可用的：

```elixir
defmodule PoliteCalculator do
  @spec add(number, number) :: number
  def add(x, y), do: make_polite(LousyCalculator.add(x, y))

  @spec make_polite(LousyCalculator.number_with_offense) :: number
  defp make_polite({num, _offense}), do: num
end
```

如果你想要保持一个自定义类型为私有的，你可以使用 `@typep` 指令来替代 `@type`。

### 静态代码分析

类型规格不是唯一一个对开发者和附加文档有用的东西。 例如， Erlang 工具 [Dialyzer](http://www.erlang.org/doc/man/dialyzer.html) 使用类型规格来静态地分析代码。这正是为什么在 `PoliteCaculator` 例子中，即使 `make_polite/1` 被定义为一个私有函数，我们仍然为它写了一个规格。


## 行为

很多模块共享了相同的公开 API。看看 [Plug](https://github.com/elixir-lang/plug)，正如它的描述，它是 web 应用中可组合模块的一个 **规范**。每个 *plug* 都是一个模块，它 *必须* 实现至少这两个公开函数： `init/1` 和 `call/2`。

行为提供了一个方式来：

* 定义一个模块必须要实现的一组函数；
* 确保一个模块实现了那个集合里的所有函数。

如果需要，你可以认为行为就像是 Java 这种面向对象语言中的接口：一个模块必须要实现的一组函数签名。

### 定义行为

假设我们要实现一系列解析器，每个解析器都解析结构化的数据：例如，一个 JSON 解析器和一个 YMAL 解析器。这两个解析器每个都以相同的方式 *运行*：两者都提供一个 `parse/1` 函数和一个 `extensions/0` 函数。 `parse/1` 函数会返回一个 Elixir 表示的结构化数据， `extensions/0` 函数会返回一个文件名后缀的列表，它可以用于每种类型的数据（例如， `.json` 对应 JSON 文件）。

我们可以创建一个 `Parser` 行为：

```elixir
defmodule Parser do
  use Behaviour

  defcallback parse(String.t) :: any
  defcallback extensions() :: [String.t]
end
```

适配 `Parser` 行为的模块必须实现所有 `defcallback` 定义的函数。正如你所看到的， `defcallback` 接受一个函数名，也接受一个函数规格，就像我们上面看到过的 `@spec` 指令那样。

### 适配行为

适配一个行为很简单：

```elixir
defmodule JSONParser do
  @behaviour Parser

  def parse(str), do: # ... parse JSON
  def extensions, do: ["json"]
end
```

```elixir
defmodule YAMLParser do
  @behaviour Parser

  def parse(str), do: # ... parse YAML
  def extensions, do: ["yml"]
end
```

如果一个适配了一个给定行为的模块没有实现该行为要求的 callback 中的一个，会生成一个编译期的警告。
