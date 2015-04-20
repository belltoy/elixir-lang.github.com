---
layout: getting-started
title: 二进制，字符串和字符列表
redirect_from: /getting_started/6.html
---

# {{ page.title }}

{% include toc.html %}

在「基本类型」中，我们已经学习了关于字符串和使用 `is_binary/1` 函数来检查字符串：

```iex
iex> string = "hello"
"hello"
iex> is_binary string
true
```

本章中，我们将理解什么是二进制（binary），如何与字符串联系起来，以及在 Elixir 中一个例如 `'like this'` 这样的单引号值是什么意思。

## UTF-8 和 Unicode

字符串是用 UTF-8 编码的二进制。为了确切理解我们这句话，我们需要先理解字节和码点 （code points） 之间的不同。

Unicode 标准给很多我们熟知的字符都赋于一个码点（code point）。例如，字母 `a` 的码点是 `97`，字母 `ł` 的码点是 `322`。当把字符串 `"hełło"` 写入磁盘的时候，我们必须把码点转换成字节。我们假定一个规则，用一个字节代表一个码点，我们就不可能写 `"hełło"` 了，因为 `ł` 的码点是 `322` 而一个字节只能表示一个从 `0` 到 `255` 的数字。但是当然了，既然我们可以从屏幕上看到 `"hełło"`，它就必须以 **某种** 方式表示。这种方式就是所谓的编码。

当用字节表示码点的时候，我们需要用某种方式编码。 Elixir 选择用 UTF-8 编码作为它的主要的也是默认的编码方式。当我们说一个字符串是一个 UTF-8 编码的二进制，我们的意思是一个字符串是一堆以 UTF-8 的编码方式来表示码点的字节。

因为我们有像 `ł` 这样的字符，它的码点是 `322`，我们实际上需要用一个以上的字节来表示。这就是为什么当我们对一个字符串计算 `byte_size/1` 和 `String.length/1` 的时候会有不同的结果。

```iex
iex> string = "hełło"
"hełło"
iex> byte_size string
7
iex> String.length string
5
```

> 注意： 如果你是在 Windows 上，你的终端有可能默认不是使用 UTF-8 。你可以在进入 iex 之前通过运行 `chcp 65001` 来修改当前会话的编码。

UTF-8 中要求用一个字节表示 `h`, `e` 和 `o` 这样的码点，用两个字节表示 `ł` 。 在 Elixir 中，你可以用 `?` 来获得码点：

```iex
iex> ?a
97
iex> ?ł
322
```

你也可以使用在 [`String` 模块](/docs/stable/elixir/#!String.html) 中的函数把一个字符串按照码点切分：

```iex
iex> String.codepoints("hełło")
["h", "e", "ł", "ł", "o"]
```

你会看到 Elixir 对字符串的支持非常好。它也支持许多 Unicode 操作。事实上， Elixir 通过了 ["The string type is broken"](http://mortoray.com/2013/11/27/the-string-type-is-broken/) 这篇文章中的所有测试用例。

然而，字符串只是故事的一部分。如果一个字符串是二进制，而我们使用 `is_binary/1` 函数， Elixir 就必须有一个底层类型来支持字符串。就是这样。让我们来谈谈二进制吧！

## 二进制（和二进制串）

在 Elixir 中，你可以用 `<<>>` 来定义一个二进制（binary）：

```iex
iex> <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> byte_size <<0, 1, 2, 3>>
4
```

一个二进制只是一个字节序列。当然，那些序列可以以任意的方式来组织，即使用一个不能使它能够表示成一个有效的字符串的方式：

```iex
iex> String.valid?(<<239, 191, 191>>)
false
```

字符串连接操作实际上是一个二进制连接操作：

```iex
iex> <<0, 1>> <> <<2, 3>>
<<0, 1, 2, 3>>
```

Elixir 中的一个常见技巧是将空字节 `<<0>>` 连接到一个字符串后面来查看这个字符串的二进制表示：

```iex
iex> "hełło" <> <<0>>
<<104, 101, 197, 130, 197, 130, 111, 0>>
```

二进制中的每个数字都表示一个字节，所以最大是 255 。二进制允许通过修饰符来表示大于 255 的数字，或者用码点转化成它的 UTF-8 表示：

```iex
iex> <<255>>
<<255>>
iex> <<256>> # truncated
<<0>>
iex> <<256 :: size(16)>> # use 16 bits (2 bytes) to store the number
<<1, 0>>
iex> <<256 :: utf8>> # the number is a code point
"Ā"
iex> <<256 :: utf8, 0>>
<<196, 128, 0>>
```

如果一个字节有 8 位，我们设成 1 位会怎么样？

```iex
iex> <<1 :: size(1)>>
<<1::size(1)>>
iex> <<2 :: size(1)>> # truncated
<<0::size(1)>>
iex> is_binary(<< 1 :: size(1)>>)
false
iex> is_bitstring(<< 1 :: size(1)>>)
true
iex> bit_size(<< 1 :: size(1)>>)
1
```

这个值不再是一个二进制，而是一个二进制串 （bitstring） ———— 只是一堆比特位！所以一个二进制是一个位数刚好可以被 8 整除的二进制串！

We can also pattern match on binaries / bitstrings:

```iex
iex> <<0, 1, x>> = <<0, 1, 2>>
<<0, 1, 2>>
iex> x
2
iex> <<0, 1, x>> = <<0, 1, 2, 3>>
** (MatchError) no match of right hand side value: <<0, 1, 2, 3>>
```

注意二进制中的每个部分都要求匹配 8 位。然而，我们可以用二进制修饰符匹配剩下的：

```iex
iex> <<0, 1, x :: binary>> = <<0, 1, 2, 3>>
<<0, 1, 2, 3>>
iex> x
<<2, 3>>
```

以上的模式只有当二进制在 `<<>>` 末尾的时候才有效。在字符串连接操作符 `<>` 上也能得到类似的结果：

```iex
iex> "he" <> rest = "hello"
"hello"
iex> rest
"llo"
```

二进制串，二进制和字符串就讲解到此。一个字符串是一个用 UTF-8 编码的二进制，而一个二进制是一个位数刚好可以被 8 整除的二进制串。尽管这体现了 Elixir 在处理位和字节上的灵活性，但我们在 99% 的时候只会乃至二进制以及 `is_binary/1` 和 `byte_size/1` 函数。

## 字符列表

一个字符列表只不过仅仅是一个一系列字符的列表：

```iex
iex> 'hełło'
[104, 101, 322, 322, 111]
iex> is_list 'hełło'
true
iex> 'hello'
'hello'
```

你可以看到，一个字符列表包含了这些位于单引号之间的字符的码点（注意 iex 只会将 ASCII 范围之外的字节用码点打印出来），而不是包含字节。所以双引号表示一个字符串 （即一个二进制），单引号表示一个字符列表（即一个列表）。

在实践中，字符列表多用于与 Erlang 对接的时候，尤其是那些不接受二进制作为参数的老库。你可以用 `to_string/1` 和 `to_char_list/1` 函数在字符列表和字符串之间转换：

```iex
iex> to_char_list "hełło"
[104, 101, 322, 322, 111]
iex> to_string 'hełło'
"hełło"
iex> to_string :hello
"hello"
iex> to_string 1
"1"
```

注意那些函数是多态的。它们不仅仅只转换字符列表到字符串，也能转换整数、原子以及其它类型到字符串。

把二进制，字符串和字符列表放一边，是时候来说说关于 key-value 数据类型了。
