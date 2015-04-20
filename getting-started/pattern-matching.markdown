---
layout: getting-started
title: 模式匹配
redirect_from: /getting_started/4.html
---

# {{ page.title }}<span hidden>.</span>

{% include toc.html %}

本章中，我们将会看到 Elixir 中的 `=` 操作符实际上是一个匹配操作符，以及如何用它在数据结构中使用模式匹配。最后，我们将会学习如何用固定操作符 `^` 来访问之前绑定的值。

## 匹配操作符

我们已经在 Elixir 中用过几次 `=` 操作符来为变量赋值了：

```iex
iex> x = 1
1
iex> x
1
```

实际上，在 Elixir 中 `=` 被称作 **匹配操作符**。 让我们来看看这是为什么：

```iex
iex> 1 = x
1
iex> 2 = x
** (MatchError) no match of right hand side value: 1
```

注意这里的 `1 = x` 是一个合法的表达式，它能够匹配是因为左右两侧都等于 1。 当两侧不匹配的时候，会引发一个 `MatchError` 错误。

只有当一个变量在 `=` 左侧的时候才能被赋值：

```iex
iex> 1 = unknown
** (RuntimeError) undefined function: unknown/0
````

由于之前没有定义过一个 `unknown` 变量， Elixir 认为你在试图调用一个名为 `unknown/0` 的函数， 但并不存在这样的函数。

## 模式匹配

匹配操作符不仅仅用于匹配简单的值，它也可以用来解构复杂的数据类型。例如，我们可以在元组上进行模式匹配：

```iex
iex> {a, b, c} = {:hello, "world", 42}
{:hello, "world", 42}
iex> a
:hello
iex> b
"world"
```

假如两边无法匹配，模式匹配就会报错。例如，当两个元组大小不一样时：

```iex
iex> {a, b, c} = {:hello, "world"}
** (MatchError) no match of right hand side value: {:hello, "world"}
```

或者当匹配不同的类型的时候：

```iex
iex> {a, b, c} = [:hello, "world", "!"]
** (MatchError) no match of right hand side value: [:hello, "world", "!"]
```

更有趣的是，我们可以匹配指定的值。下面的例子中可以看出，只有当右侧是一个以原子 `:ok` 开头的元组时，左侧才会与之匹配：

```iex
iex> {:ok, result} = {:ok, 13}
{:ok, 13}
iex> result
13

iex> {:ok, result} = {:error, :oops}
** (MatchError) no match of right hand side value: {:error, :oops}
```

我们可以在列表上进行模式匹配：

```iex
iex> [a, b, c] = [1, 2, 3]
[1, 2, 3]
iex> a
1
```

也可以对一个列表的头部和尾部进行匹配：

```iex
iex> [head | tail] = [1, 2, 3]
[1, 2, 3]
iex> head
1
iex> tail
[2, 3]
```

与 `hd/1` 和 `tl/1` 函数类似，我们无法对一个空列表匹配头部和尾部：

```iex
iex> [h|t] = []
** (MatchError) no match of right hand side value: []
```

`[head | tail]` 格式不仅仅用在模式匹配上，还可以用于添加项到一个列表的头部：

```iex
iex> list = [1, 2, 3]
[1, 2, 3]
iex> [0|list]
[0, 1, 2, 3]
```

模式匹配允许开发者可以很简单地解构如元组和列表的数据类型。 正如我们将在接来的章节中看到的，它是 Elixir 中递归的一个基础，另外它也可以用在其它类型上，像是 map 和 binary。

## 固定操作符

Elixir 中的变量可以被重新绑定：

```iex
iex> x = 1
1
iex> x = 2
2
```

如果不想重新绑定一个变量，而是为了匹配之前的值，就需要用到固定操作符 `^`了：

```iex
iex> x = 1
1
iex> ^x = 2
** (MatchError) no match of right hand side value: 2
iex> {x, ^x} = {2, 1}
{2, 1}
iex> x
2
```

注意如果一个变量在一个模式中多次出现，所有的引用都应该绑定到同一个模式上：

```iex
iex> {x, x} = {1, 1}
1
iex> {x, x} = {1, 2}
** (MatchError) no match of right hand side value: {1, 2}
```

在某些情况下，你不关心一个模式中的某个值。通常的做法是将那些值绑定到下划线 `_`。 举个例子，如果我们只关心列表的头部，我们可以把尾部赋值给下划线：

```iex
iex> [h | _] = [1, 2, 3]
[1, 2, 3]
iex> h
1
```

这里 `_` 是个特殊的变量，它永远无法被读取。 试图读取的话，会得到一个未绑定变量的错误：

```iex
iex> _
** (CompileError) iex:1: unbound variable _
```

尽管模式匹配允许我们构建强大的结构，但它却不是万能的。例如，你无法在一个匹配的左侧调用函数。下面的例子是无效的：

```iex
iex> length([1,[2],3]) = 3
** (CompileError) iex:1: illegal pattern
```

我们对模式匹配的介绍就到此为止。 在接下来的一章中你会看到，模式匹配在很多语言结构中是非常常见的。
