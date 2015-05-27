---
layout: getting-started
title: Sigils
redirect_from: /getting_started/18.html
---

# {{ page.title }}

{% include toc.html %}

我们已经学过，Elixir 提供了双引号字符串和单引号字符列表。然而这仅仅包含了语言中具有文本表示的结构的表面。例如，原子常常用 `:atom` 表示法来创建。

Elixir 的目标之一是可扩展性： 开发者应该能够扩展这个语言以适应任意特定领域。计算机科学已经变成一个如此宽广的领域，以至于一个语言的核心部分不可能解决这么多领域的问题。我们最好的选择是使语言变得可扩展，这样开发人员，公司和社区可以扩展这个语言到他们相关的领域。

本章中，我们将学习 sigils，它是这个语言提供的用于扩展文本表示的机制之一。Sigils 以波浪符（`~`）开始，跟着一个字母（用于标识这个 sigil）接着是一个分隔符；可选的，在分隔符之后可以添加修饰符。

## 正则表达式

在 Elixir 中最常见的 sigil 是 `~r`，它用于创建 [正则表达式](https://en.wikipedia.org/wiki/Regular_Expressions):

```iex
# A regular expression that matches strings which contain "foo" or "bar":
iex> regex = ~r/foo|bar/
~r/foo|bar/
iex> "foo" =~ regex
true
iex> "bat" =~ regex
false
```

Elixir 提供了 Perl 兼容的正则表达式（regexes），类似于 [PCRE](http://www.pcre.org/) 库的实现。正则表达式也支持修饰符。例如，`i` 修饰符可以使一个正则表达式不区分大小写：

```iex
iex> "HELLO" =~ ~r/hello/
false
iex> "HELLO" =~ ~r/hello/i
true
```

阅读 [`Regex` 模块](/docs/stable/exlir/#!Regex.html) 以了解更多正则表达式的其它修饰符和支持的操作。

到目前为止，所有的例子都是使用 `/` 来分隔一个正则表达式。然而 sigils 支持 8 种不同的分隔符：

```
~r/hello/
~r|hello|
~r"hello"
~r'hello'
~r(hello)
~r[hello]
~r{hello}
~r<hello>
```

支持不同的分隔符背后的原因是不同的分隔符对于不同的 sigils 会更加适用。例如，在正则表达式中使用圆括号可能是一个不适当的选择，因为它们会和正则表达式内部的圆括号混合。然而，圆括号在其它的 sigils 中可能就非常便利，正如我们在下一节将要看到的。

## 字符串，字符列表和和 words sigils

除了正则表达式， Elixir  还提供了其它的 sigils。

### 字符串

`s` sigils 用于生成字符串，就像双引号那样。例如，当一个字符串包含了双引号和单引号的时候，`s` sigil 非常有用：

```iex
iex> ~s(this is a string with "double" quotes, not 'single' ones)
"this is a string with \"double\" quotes, not 'single' ones"
```

### 字符列表

`~c` sigil 用于生成字符列表：

```iex
iex> ~c(this is a char list containing 'single quotes')
'this is a char list containing \'single quotes\''
```

### 单词列表

`~w` sigil 用于生成单词的列表（*单词* 只是普通的字符串）。在 `~w` sigil 内部，单词用空格分隔。

```iex
iex> ~w(foo bar bat)
["foo", "bar", "bat"]
```

`~w` sigil 还接受 `c`, `s` 和 `a` 修饰符（分别表示字符列表，字符串和原子），用于指定结果列表中元素的数据类型：

```iex
iex> ~w(foo bar bat)a
[:foo, :bar, :bat]
```

## Sigil 中的插值和转义

除了小写的 sigils， Elixir 还支持大写的 sigils 用于处理转义字符和插值。 虽然 `~s` 和 `~S` 都返回字符串，前者允许转码和插值，后者则不能：

```elixir
iex> ~s(String with escape codes \x26 #{"inter" <> "polation"})
"String with escape codes & interpolation"
iex> ~S(String without escape codes and without #{interpolation})
"String without escape codes and without \#{interpolation}"
```

下列转义码可以用在字符串和字符列表中：

* `\"` – 双引号
* `\'` – 单引号
* `\\` – 单反斜杠
* `\a` – bell/alert
* `\b` – 退格
* `\d` - 删除
* `\e` - escape
* `\f` - form feed
* `\n` – 换行
* `\r` – 回车
* `\s` – 空格
* `\t` – tab
* `\v` – 垂直 tab
* `\0` - 空字节
* `\xDD` - 十六进制 DD （例如，`\x13`） 表示的字符
* `\x{D...}` - 十六进制表示的字符，包含一个或多个十六进制数字（例如， `\x{abc13}`）

Sigils 还支持 heredocs 文档，它是用三个双引号或者单引号作为分隔符：

```iex
iex> ~s"""
...> this is
...> a heredoc string
...> """
```

heredoc 文档最常见的使用案例是编写文档。例如，在文档中编写转义字符很容易出错，因为它需要双重转义一些字符：

```elixir
@doc """
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\\\"foo\\\"")
    "'foo'"

"""
def convert(...)
```

使用 `~S`， 这个问题可以完全避免这个问题：

```elixir
@doc ~S"""
Converts double-quotes to single-quotes.

## Examples

    iex> convert("\"foo\"")
    "'foo'"

"""
def convert(...)
```

## 自定义 sigils

正如在本章开头处提到的， Elixir 中的 sigil 是可扩展的。事实上，使用 sigil `~r/foo/i` 相当于是用一个二进制串和一个字符列表作为参数调用 `sigil_r` 函数：

```iex
iex> sigil_r(<<"foo">>, 'i')
~r"foo"i
```

我们可以通过 `sigil_r` 函数来访问 `~r` sigil 的文档：

```iex
iex> h sigil_r
...
```

通过简单地实现类似 `sigil_{identifier}` 模式的函数，我们还可以提供我们自己的 sigil。例如，让我们来实现一个 `~i` sigil，它返回一个整数（支持 `n` 修饰符选项，使它返回负值）：

```iex
iex> defmodule MySigils do
...>   def sigil_i(string, []), do: String.to_integer(string)
...>   def sigil_i(string, [?n]), do: -String.to_integer(string)
...> end
iex> import MySigils
iex> ~i(13)
13
iex> ~i(42)n
-42
```

Sigil 在宏的帮助下，还可以用于编译时的工作。例如， Elixir 中的正则表达式会在编译源码的时候被编译成一个高效的表示，所以在运行时就可以跳过这些步骤。如果你对这个主题感兴趣，我们推荐你深入学习宏并查阅 sigil 是如果在 `Kernel` 模块（定义 `sigil_*` 一系列函数的地方）中实现的。
