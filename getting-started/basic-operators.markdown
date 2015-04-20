---
layout: getting-started
title: 基本操作符
redirect_from: /getting_started/3.html
---

# {{ page.title }}

{% include toc.html %}

在 [上一章](/getting-started/basic-types.html) 中，我们看到 Elixir 提供了 `+`， `-`， `*`， `/` 作为数学运算符，加上函数 `div/2` 和 `rem/2` 作整数除法和余数运算。

Elixir 也提供了 `++` 和 `--` 来操作列表：

```iex
iex> [1,2,3] ++ [4,5,6]
[1,2,3,4,5,6]
iex> [1,2,3] -- [2]
[1,3]
```

字符串连接操作用 `<>`：

```iex
iex> "foo" <> "bar"
"foobar"
```

Elixir 也提供了三个布尔操作符： `or`， `and` 和 `not`。这些操作符严格限定了它们的第一个参数必须是布尔值（`true` 或者 `false`）：

```iex
iex> true and true
true
iex> false or is_atom(:example)
true
```

提供一个非布尔值会引发一个异常：

```iex
iex> 1 and true
** (ArgumentError) argument error
```

`or` 和 `and` 是短路操作符。它们只有当左侧的表达式不满足条件时才会执行右边的表达式：

```iex
iex> false and error("This error will never be raised")
false

iex> true or error("This error will never be raised")
true
```

> 注意： 如果你是一个 Erlang 开发者， Elixir 中的 `and` 和 `or` 实际上是对应 Erlang 中的 `andalso` 和 `orelse` 操作符。

除了这些布尔操作符， Elixir 还提供了 `||`， `&&` 和 `!`，它们接受任意类型的值作为参数。对于这些操作符，除了 `false` 和 `nil` 之外的所有值都为真：

```iex
# or
iex> 1 || true
1
iex> false || 11
11

# and
iex> nil && 13
nil
iex> true && 17
17

# !
iex> !true
false
iex> !1
false
iex> !nil
true
```

一般来说，当你预期是布尔值的时候，使用 `and`， `or` 和 `not`。 如果任意一个参数是非布尔值，就用 `&&`， `||` 和 `!`。

Elixir 也提供了 `==`, `!=`, `===`, `!==`, `<=`, `>=`, `<` 和 `>` 作为比较操作符：

```iex
iex> 1 == 1
true
iex> 1 != 2
true
iex> 1 < 2
true
```

`==` 和 `===` 的区别在于当比较整数和浮点数的时候后者更加严格：

```iex
iex> 1 == 1.0
true
iex> 1 === 1.0
false
```

在 Elixir 中，我们能够比较两个不同数据类型的值：

```iex
iex> 1 < :atom
true
```

我们能够比较不同类型的原因是实用主义。排序算法在排序的时候不需要担心不同的数据类型。整体排序顺序定义如下：

    number < atom < reference < functions < port < pid < tuple < maps < list < bitstring

实际上你不需要去记住这些顺序，更重要的是知道这么一个顺序的存在。

好了，介绍到此为止。在下一章中，我们将会讨论一些基本函数，数据类型转换和一些流程控制。
