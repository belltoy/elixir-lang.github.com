---
layout: getting-started
title: 基本类型
redirect_from: /getting_started/2.html
---

# {{ page.title }}

{% include toc.html %}

在这个章节中我们将会学到更多关于 Elixir 的基本类型：整型，浮点型，布尔型，原子，字符串，列表和元组。下面举了些例子：

```iex
iex> 1          # integer
iex> 0x1F       # integer
iex> 1.0        # float
iex> true       # boolean
iex> :atom      # atom / symbol
iex> "elixir"   # string
iex> [1, 2, 3]  # list
iex> {1, 2, 3}  # tuple
```

## 基本运算

打开 `iex` 并输入下面的表达式：

```iex
iex> 1 + 2
3
iex> 5 * 5
25
iex> 10 / 2
5.0
```

注意到 `10/2` 返回的是一个浮点值 `5.0` 而不是整型 `5`。这与预期的情况相符。在 Elixir 中， `/` 操作符总是返回一个浮点值。如果你要做整数除法或者求余数，可以调用 `div` 和 `rem` 函数：

```iex
iex> div(10, 2)
5
iex> div 10, 2
5
iex> rem 10, 3
1
```

注意到，调用函数的时候圆括号不是必需的。

Elixir 也支持输入二进制，八进制和十六进制的数字。

```iex
iex> 0b1010
10
iex> 0o777
511
iex> 0x1F
31
```

浮点数要求小数点跟在至少一个数字之后，并且支持 `e` 表示阶数（译注：科学计数法）：

```iex
iex> 1.0
1.0
iex> 1.0e-10
1.0e-10
```

Elixir 中浮点数是 64 位双精度的。

你可以调用 `round` 函数来对一个浮点数求整数近似值，或者 `trunc` 函数来获取一个浮点数的整数部分。

```iex
iex> round 3.58
4
iex> trunc 3.58
3
```

## 布尔值

Elixir 支持布尔值 `true` 和 `false`：

```iex
iex> true
true
iex> true == false
false
```

Elixir 提供了一系列判定函数来检查值的类型。例如， `is_boolean/1` 函数可以用来检查一个值是否是布尔值：

> 注意： Elixir 中的函数是用名字和元数（即参数数量 arity）来标识的。因此，`is_boolean/1` 表示一个名为 `is_boolean` 并且接受一个参数的函数。`is_boolean/2` 表示一个不同的函数（这个函数实际中不存在），虽然名字相同但元数不同。

```iex
iex> is_boolean(true)
true
iex> is_boolean(1)
false
```
你可以使用 `is_integer/1`，`is_float/1` 或者 `is_number/1` 来分别检查一个参数是否是整数，浮点数或者两者之一。

> 注意：任何时候你都可以在 shell 中输入 `h` 来打印关于 shell 使用帮助信息。`h` 帮助也可以用来访问任意函数的文档。例如，输入 `h is_integer/1` 将会打印出 `is_integer/1` 函数的文档。它也可以用在操作符和其它结构上（试试 `h ==/2`）。

## 原子

原子是以名字为值的恒量。一些其它语言中称之为符号（Symbols）：

```iex
iex> :hello
:hello
iex> :hello == :world
false
```

布尔值 `true` 和 `false` 实际上也是原子：

```iex
iex> true == :true
true
iex> is_atom(false)
true
iex> is_boolean(:false)
true
```

## 字符串

Elixir 中的字符串是用双引号包围的，而且是用 UTF-8 编码的：

```iex
iex> "hellö"
"hellö"
```

> 注意： 如果你是在 Windows 上，你的终端有可能默认不是使用 UTF-8 。你可以在进入 iex 之前通过运行 `chcp 65001` 来修改当前会话的编码。

Elixir 也支持字符串插值：

```iex
iex> "hellö #{:world}"
"hellö world"
```

字符串中可以包含换行符，或者用转义序列表示：

```iex
iex> "hello
...> world"
"hello\nworld"
iex> "hello\nworld"
"hello\nworld"
```

你可以用 `IO` 模块中的 `IO.outs/1` 函数来打印一个字符串：

```iex
iex> IO.puts "hello\nworld"
hello
world
:ok
```

注意 `IO.puts/1` 函数在打印完之后，返回的结果是原子 `:ok`。

Elixir 中的字符串在内部是用字节序列的二进制来表示的：

```iex
iex> is_binary("hellö")
true
```

我们也能获得一个字符串包含的字节数：

```iex
iex> byte_size("hellö")
6
```

注意上面这个字符串包含的字节数是 6，即使它只包含了 5 个字符。那是因为字符 "ö" 用 UTF-8 表示需要占两个字节。我们可以用 `String.length/1` 函数来得到字符串中字符数的实际长度：

```iex
iex> String.length("hellö")
5
```

[String 模块](/docs/stable/elixir/#!String.html) 包含了一系列对字符串进行 Unicode 标准操作的函数：

```iex
iex> String.upcase("hellö")
"HELLÖ"
```

## 匿名函数

函数是用关键词 `fn` 和 `end` 来界定的：

```iex
iex> add = fn a, b -> a + b end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> is_function(add)
true
iex> is_function(add, 2)
true
iex> is_function(add, 1)
false
iex> add.(1, 2)
3
```

在 Elixir 中函数是 「一等公民」， 这意味着它们可以被像整型和字符串一样当作参数传递到其它函数中。 在上面的例子中，我们把变量 `add` 指向的函数传递给 `is_function/1` 函数， 然后正确得返回了 `true`。 我们还能够通过调用 `is_function/2` 检查函数的元数。

注意调用匿名函数的时候，在变量名和括号之间的点（`.`)是必需的。

匿名函数是闭包，因此它们能够访问在它被定义的时候作用域里的变量：

```iex
iex> add_two = fn a -> add.(a, 2) end
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> add_two.(2)
4
```

请记住，在函数内部赋值变量不会影响其外部的环境：

```iex
iex> x = 42
42
iex> (fn -> x = 0 end).()
0
iex> x
42
```

## (链式) 列表

Elixir 用方括号来指定一个列表。它的值可以是任意类型：

```iex
iex> [1, 2, true, 3]
[1, 2, true, 3]
iex> length [1, 2, 3]
3
```

两个列表可以用 `++/2` 和 `--/2` 操作符进行连接和消除操作：

```iex
iex> [1, 2, 3] ++ [4, 5, 6]
[1, 2, 3, 4, 5, 6]
iex> [1, true, 2, false, 3, true] -- [true, false]
[1, 2, 3, true]
```

在本教程中，我们将会讨论许多关于列表头部（head）和尾部（tail）。头部是一个列表的第一个元素，尾部是剩余的部分。它们可以用函数 `hd/1` 和 `tl/1` 来取出。让我们来指定一个列表给一个变量，然后取出它的头部和尾部：

```iex
iex> list = [1,2,3]
iex> hd(list)
1
iex> tl(list)
[2, 3]
```

尝试取出一个空列表的头部和尾部会引发一个错误：

```iex
iex> hd []
** (ArgumentError) argument error
```

有时候你创建一个列表，返回的是一个用单引号包围的值。例如：

```iex
iex> [11, 12, 13]
'\v\f\r'
iex> [104, 101, 108, 108, 111]
'hello'
```

当 Elixir 看见一个包含 ASCII 数字的列表时， Elixir 会将它打印成一个字符列表（显示成一个字符的列表）。当与 Erlang 现有代码时对接时，字符列表是非常常见的。

请记住， 在 Elixir 中，用单引号包围的和用双引号包围的表示法不是等价的，因为它们表示了不同的类型：

```iex
iex> 'hello' == "hello"
false
```

单引号包围的是字符列表，双引号包围的是字符串。我们将在 「二进制，字符串和字符列表」 这章中更多地讨论它们。

## 元组

Elixir 用花括号来定义元组。与列表一样，元组里可以放入任意的值：

```iex
iex> {:ok, "hello"}
{:ok, "hello"}
iex> tuple_size {:ok, "hello"}
2
```

元组将元素连续地存放在内存中。这意味着按照索引访问一个元组的元素或者获得元组在大小是一个快速的操作（索引从零开始）：

```iex
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
iex> tuple_size(tuple)
2
```

用 `put_elem/3` 来设置元组中一个特定索引的元素也是可能的：

```iex
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> put_elem(tuple, 1, "world")
{:ok, "world"}
iex> tuple
{:ok, "hello"}
```

注意到 `put_elem/3` 返回一个新的元组。存储在 `tuple` 变量中原始的元组并没有被修改，这里因为 Elixir 的数据类型是不可变的。由于是不可变的， Elixir 代码更为容易理解为什么你永远不用担心一个特定的代码是否会就地修改你的数据结构。

由于是不可变的， Elixir 也有助于消除这种常见的情况，并发代码中由于不同实体同时尝试修改一个数据结构引发的竞态条件。

## 列表还是元组？

列表和元组之间有什么不同之处呢？

列表是以链表的形式存储在内存中的，这意味着列表中的每个元素保存其值的同时还指向下一个元素，直到列表结束。我们将每一对值和指针称为一个 **结构单元**（cons cell）：

```iex
iex> list = [1|[2|[3|[]]]]
[1, 2, 3]
```

这意味着访问一个列表的长度是一个线性操作： 我们需要遍历整个列表来算出它的大小。 更新一个列表很快，我们只要在前面加上元素：

```iex
iex> [0] ++ list
[0, 1, 2, 3]
iex> list ++ [4]
[1, 2, 3, 4]
```

上面的例子中，第一个操作很快，因为我们只是简单地添加了一个结构，并将它指向 `list`。第二个操作很慢，因为我们需要重建整个列表，然后添加一个新的元素到末尾。

另一方面，元组是连续地存储在内存中的。这意味着获取它的大小或者以索引的方式访问一个元素是很快的。但是，更新或者添加元素到一个元组中的操作是昂贵的，因为这需要在内存中拷贝整个元组。

这些性能特性决定了这些数据结构的使用方式。元组的一个非常常见的用例是用它们从函数中返回附加信息。例如， `File.read/1` 是一个用来读取文件内容的函数，它返回一个元组：

```iex
iex> File.read("path/to/existing/file")
{:ok, "... contents ..."}
iex> File.read("path/to/unknown/file")
{:error, :enoent}
```

如果传给 `File.read/1` 的路径存在，它返回一个元组，第一个元素是原子 `:ok`，第二个元素是文件内容。否则，返回一个包含 `:error` 和错误描述的元组。

大多数时候， Elixir 会引导你做正确的事情。例如，有一个 `elem/2` 函数用来访问一个元组中的项，但对于列表来说却没有等价的操作：

```iex
iex> tuple = {:ok, "hello"}
{:ok, "hello"}
iex> elem(tuple, 1)
"hello"
```

当「计算」一个数据结构中的元素数量的时候， Elixir 同样遵循着这么一条简单的原则：如果操作时间是恒量（也就是说这个值是预先计算好的），那么这个函数应该命名为 `size`，如果操作需要明确地计数，函数则应该命名为 `length`。

例如，我们到目前为止用过 4 个计数的函数： `byte_size/1` （获取字符串中的字节数量）， `tuple_size/1` （获取元组的大小）， `length/1` （获取列表的长度）， `String.length/1` （获取一个字符串中的字符数量）。这就是说，我们用 `byte_size` 来获取一个字符串中的字节数是廉价的，但是用 `String.length` 来获取字符串中 unicode 字符的数量却是昂贵的操作，因为需要遍历整个字符串。

Elixir 还提供了 `Port`，`Reference` 和 `PID` 作为数据类型 （一般用在进程通讯中），我们将在讨论进程的时候快速浏览一下这些类型。现在，我们来看一下我们这些基本类型相应的一些基本操作符。
