---
layout: getting-started
title: 可枚举开类和流
redirect_from: /getting_started/10.html
---

# {{ page.title }}

{% include toc.html %}

## 可枚举类型

Elixir 提供了可枚举类型的概念和用于处理它们的 [`Enum` 模块](/docs/stable/elixir/#!Enum.html)。我们已经学过两种可枚举类型：列表和 map。

```iex
iex> Enum.map([1, 2, 3], fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.map(%{1 => 2, 3 => 4}, fn {k, v} -> k * v end)
[2, 12]
```

`Enum` 模块提供了大量的函数用于转换，排序，分组，过滤和获取可枚举类型中的元素。 在 Elixir 代码中它是开发者最常用的模块之一。

Elixir 还提供了范围类型（range）：

```iex
iex> Enum.map(1..3, fn x -> x * 2 end)
[2, 4, 6]
iex> Enum.reduce(1..3, 0, &+/2)
6
```

由于 Enum 模块被设计成能够跨不同数据类型工作，它的 API 只包含那些能够通用于多种不同数据类型的函数。对一些特殊的操作，你也许需要用到专门针对那个数据类型的模块。例如，如果你要向一个列表的特定位置插入一个元素，你要用到 [`List` 模块](/docs/stable/elixir/#!List.html) 中的 `List.insert_at/3` 函数，但是向一个范围类型（range）中插入一个值就显得没有意义了。

我们说在 `Enum` 模块中的函数是多态的，因为它们能够用于多种数据类型。在实践中， `Enum` 模块中的函数能够用于任何实现了 [`Enumerable` 协议](/docs/stable/elixir/#!Enumerable.html) 的数据类型。我们将在之后的章节中讨论协议（Protocol），现在我们继续学习一种可枚举类型的特例，流。

## 及早求值 vs 惰性求值

所有 `Enum` 模块中的函数都是及早求值（eager）的。很多函数期望一个可枚举类型并且返回一个列表：

```iex
iex> odd? = &(rem(&1, 2) != 0)
#Function<6.80484245/1 in :erl_eval.expr/5>
iex> Enum.filter(1..3, odd?)
[1, 3]
```

这意味着当用 `Enum` 进行多次操作的时候，每个操作都会生成一个中间列表，直到我们得到结果。

```iex
iex> 1..100_000 |> Enum.map(&(&1 * 3)) |> Enum.filter(odd?) |> Enum.sum
7500000000
```

上面是一个管道操作的例子。我们从一个范围开始，然后对范围中的每个值乘 3 。第一个操作会创建并返回一个包含 `100_000` 个元素的列表。然后我们返回列表中所有的奇数元素，生成一个新的列表，现在有 `50_000` 个元素，然后对所有的元素求和。

### 管道操作符

上面代码片断中用到的 `|>` 符号是 **管道操作符**：它简单地将它左边表达式的输出传给右边的函数调用。它和 Unix 的 `|` 操作符类似。其目的是突出数据被一系列函数转换的过程。如果将上面的例子不用 `|>` 操作符来重写，就可以看出管道操作符是如何简化代码了：

```iex
iex> Enum.sum(Enum.filter(Enum.map(1..100_000, &(&1 * 3)), odd?))
7500000000
```

[阅读管道操作符的文档](/docs/stable/elixir/#!Kernel.html#|>/2) 来了解更多。

## 流

作为 `Enum` 的一个替代品， Elixir 提供了 [`Stream` 模块](/docs/stable/elixir/#!Stream.html) 支持惰性（lazy）操作：

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?) |> Enum.sum
7500000000
```

流是惰性的，可组合的可枚举类型。
流创建了一系列的计算，只有在传给 `Enum` 模块的时候才会调用，而不是生成一个中间结果的列表。当处理大型的，*可能是无限的*，数据集的时候，流是非常有用的。

它们是惰性的是因为，如上面的例子所示， `1..100_000 |> Stream.map(&(&1 * 3))` 返回一个数据类型，一个流，它表示对范围 `1..100_00` 的 `map` 运算。

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3))
#Stream<[enum: 1..100000, funs: [#Function<34.16982430/1 in Stream.map/2>]]>
```

进一步说，它们是可组合的是因为我们可以把多个流的操作用管道连接起来：

```iex
iex> 1..100_000 |> Stream.map(&(&1 * 3)) |> Stream.filter(odd?)
#Stream<[enum: 1..100000, funs: [...]]>
```

很多在 `Stream` 模块中的函数接受任意的可枚举类型作为参数并返回一个流作为结果。它还提供了用于创建流，可能是无限的流的函数。例如， `Stream.cycle/1` 可以用来创建一个在给定的可枚举类型上的无限反复的流。注意不要对这样的流调用像 `Enum.map/2` 这样的函数，这可能会导致死循环：

```iex
iex> stream = Stream.cycle([1, 2, 3])
#Function<15.16982430/2 in Stream.cycle/1>
iex> Enum.take(stream, 10)
[1, 2, 3, 1, 2, 3, 1, 2, 3, 1]
```

另一方面， `Stream.unfold/2` 可以用来从一个给定的初始值生成一个流：

```iex
iex> stream = Stream.unfold("hełło", &String.next_codepoint/1)
#Function<39.75994740/2 in Stream.unfold/2>
iex> Enum.take(stream, 3)
["h", "e", "ł"]
```

另一个有趣的函数是 `Stream.resource/3`，可以用来包装资源，保证它们在枚举操作之前打开，结束之后关闭，甚至是失败的情况。例如，我们可以用它来流化一个文件：

```iex
iex> stream = File.stream!("path/to/file")
#Function<18.16982430/2 in Stream.resource/3>
iex> Enum.take(stream, 10)
```

上面的例子会取到你所选择的文件的前 10 行。这意味着对于处理大型文件或者如网络资源这类的慢资源的时候，流会非常有用。

在 [`Enum`](/docs/stable/elixir/#!Enum.html) 和 [`Stream`](/docs/stable/elixir/#!Stream.html) 模块中的函数和功能和数量咋看起来挺吓人的，但你通过一个个用例来熟悉它们。尤其是，先专注于 `Enum` 模块，只有在处理慢资源，大型的，可能是无限的数据集，这些需要惰性操作的时候，这些特定场景才选择用 `Stream`。

接下来我们将会看看 Elixir 的一个核心特性，进程，它允许我们用简单的而且容易理解的方式编写并发并发的，并行的和分布式的程序。
