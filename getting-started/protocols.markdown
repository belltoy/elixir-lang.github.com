---
layout: getting-started
title: 协议
redirect_from: /getting_started/16.html
---

# {{ page.title }}

{% include toc.html %}

协议是 Elixir 中实现多态的机制。对于任何只要实现了协议数据类型都可以使用该协议上的函数。让我们来看一个例子。

在 Elixir 中，只有 `false` 和 `nil` 被当作「假」。其它所有的都为真。 对于某些应用，也许规范一个 `blank?` 协议是非常重要的，它对其它数据类型中应该被认为是空的值返回一个布尔值。例如，一个空列表或者一个空的二进制可以被认为是空值。

我们可以这样定义这个协议：

```elixir
defprotocol Blank do
  @doc "Returns true if data is considered blank/empty"
  def blank?(data)
end
```

这个协议期待一个名为 `blank?` 的函数实现，它接收一个参数。我们可以这样为 Elixir 中的不同数据类型实现这个协议：

```elixir
# Integers are never blank
defimpl Blank, for: Integer do
  def blank?(_), do: false
end

# Just empty list is blank
defimpl Blank, for: List do
  def blank?([]), do: true
  def blank?(_),  do: false
end

# Just empty map is blank
defimpl Blank, for: Map do
  # Keep in mind we could not pattern match on %{} because
  # it matches on all maps. We can however check if the size
  # is zero (and size is a fast operation).
  def blank?(map), do: map_size(map) == 0
end

# Just the atoms false and nil are blank
defimpl Blank, for: Atom do
  def blank?(false), do: true
  def blank?(nil),   do: true
  def blank?(_),     do: false
end
```

我们也可以对所有的内置数据类型这么做。这些类型包括：

* `Atom`
* `BitString`
* `Float`
* `Function`
* `Integer`
* `List`
* `Map`
* `PID`
* `Port`
* `Reference`
* `Tuple`

现在我们手里有了这个协议的定义和实现，我们可以调用它了：

```iex
iex> Blank.blank?(0)
false
iex> Blank.blank?([])
true
iex> Blank.blank?([1, 2, 3])
false
```

传入一个未实现这个协议的数据类型会引发一个错误：

```iex
iex> Blank.blank?("hello")
** (Protocol.UndefinedError) protocol Blank not implemented for "hello"
```

## 协议和 Struct

当协议和 struct 结合在一起使用的时候， Elixir 的可扩展性的力量就能显现出来了。

在 [上一章](/getting-started/structs.html) 中，我们已经学习了虽然 struct 是 map，但它并没有与 map 共享协议的实现。让我们像在上一章中那样定义一个 `User` 的 struct：

```iex
iex> defmodule User do
...>   defstruct name: "john", age: 27
...> end
{:module, User,
 <<70, 79, 82, ...>>, {:__struct__, 0}}
```

然后检查：

```iex
iex> Blank.blank?(%{})
true
iex> Blank.blank?(%User{})
** (Protocol.UndefinedError) protocol Blank not implemented for %User{age: 27, name: "john"}
```

Struct 并没有和 map 共享协议的实现，struct 需要它们自己的协议实现：

```elixir
defimpl Blank, for: User do
  def blank?(_), do: false
end
```

如果有需要，你可以为一个空白用户提供你自己的语义。不仅如此，你可以使用 struct 来构建更加健壮的数据类型，如队列，并为这个数据类型实现所有相关的协议，例如 `Enumerable` 和 `Blank`。

尽管在很多情况下，开发者也许想为 struct 提供一个默认的实现，因为为所有的 struct 明确地实现一个协议会很单调。这种时候 falling back to `Any` 就显得很有用。

## Falling back to `Any`

为所有的数据类型提供一个默认的实现，将会是非常方便的。这可以通过在协议的定义中设置 `@fallback_to_any` 为 `true` 来得到：

```elixir
defprotocol Blank do
  @fallback_to_any true
  def blank?(data)
end
```

它也可以这样实现：

```elixir
defimpl Blank, for: Any do
  def blank?(_), do: false
end
```

现在所有我们未实现 `Blank` 协议的数据类型 （包括 struct ）都会被当作非空。

## 内建协议

Elixir 包含了一些内建协议。在上一章中，我们已经讨论了 `Enum` 模块，它提供了许多函数作用于任意实现了 `Enumerable` 协议的数据类型：

```iex
iex> Enum.map [1, 2, 3], fn(x) -> x * 2 end
[2,4,6]
iex> Enum.reduce 1..3, 0, fn(x, acc) -> x + acc end
6
```

另一个有用的例子是 `String.Chars` 协议，它规定了如何将一个包括字符的数据结构转化为字符串。它通过 `to_string` 函数来暴露：

```iex
iex> to_string :hello
"hello"
```

注意到 Elixir 中的字符串插值会调用这个 `to_string` 函数：

```iex
iex> "age: #{25}"
"age: 25"
```

上面的片断能工作是因为数字实现了 `String.Chars` 协议。例如，传入一个元组会导致一个错误：

```iex
iex> tuple = {1, 2, 3}
{1, 2, 3}
iex> "tuple: #{tuple}"
** (Protocol.UndefinedError) protocol String.Chars not implemented for {1, 2, 3}
```

当如果需要打印更加复杂的数据结构的时候，可以简单地使用 `inspect` 函数，它是基于 `Inspect` 协议的：

```iex
iex> "tuple: #{inspect tuple}"
"tuple: {1, 2, 3}"
```

`Inspect` 协议是用于把任意的数据结构转化为一个可读的文本表示。这是像 Iex 这样的工具用于打印结果的方式：

```iex
iex> {1, 2, 3}
{1,2,3}
iex> %User{}
%User{name: "john", age: 27}
```

牢记，按照惯例，只要打印出来的检视值是以 `#` 开头，它就是用 Elixir 中的非法语法来表示一个数据结构。这意味着 inspect 协议是不可逆的，因为在转化过程中信息可能丢失了：

```iex
iex> inspect &(&1+2)
"#Function<6.71889879/1 in :erl_eval.expr/5>"
```

Elixir 中还包含其它协议，但本章包含了最常见的几个。
