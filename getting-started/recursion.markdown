---
layout: getting-started
title: 递归
redirect_from: /getting_started/9.html
---

# {{ page.title }}

{% include toc.html %}

## 用递归实现循环

由于不可变性，循环在 Elixir 中（在所有的函数式语言中）的写法与命令式语言中不太一样。举例说明，在一个命令式语言（以下以 C 为例）中，可能会这样写：

```c
for(i = 0; i < array.length; i++) {
  array[i] = array[i] * 2
}
```

在上面的例子中，我们修改了数组和变量 `i`。 Elixir 中是不允许修改的。取而代之的是，函数式语言依赖于递归： 一个函数反复被自身调用，直到达到停止条件，这样的函数被称作递归函数。 考虑以下的例子，打印一个字符串任意次数：

```elixir
defmodule Recursion do
  def print_multiple_times(msg, n) when n <= 1 do
    IO.puts msg
  end

  def print_multiple_times(msg, n) do
    IO.puts msg
    print_multiple_times(msg, n - 1)
  end
end

Recursion.print_multiple_times("Hello!", 3)
# Hello!
# Hello!
# Hello!
```

与 `case` 类似，一个函数可以有多个分句。一个分句当接收的参数匹配分句的参数模式并且守卫条件等于 `true` 的时候才会被执行。

上面的例子中，当 `print_multiple_times/2` 被第一次调用的时候，参数 `n` 等于 `3`。

第一个分句有个守卫说「当且仅当 `n` 小于或等于 `1` 的时候才使用这个定义」。由于不满足这个情况， Elixir 继续尝试下一个分句定义。

第二个定义匹配这个模式，且没有守卫，所以它会被执行。它的第一次打印出我们的 `msg` 并且调用它自己，传入 `n - 1` （`2`） 作为第二个参数。

我们的 `msg` 被打印出来了，接着 `print_multiple_times/2` 再次被调用，这次第二个参数为 `1`。
因为 `n` 现在被设为了 `1`， 我们 `print_multiple_times/1` 第一个字义中的守卫的计算结果为真，于是这个定义被执行了。 `msg` 被打印出来了，然后没有了。

根据我们定义的 `print_multiple_times/2` ，无论第二个参数传入什么值，它都会触发第一个定义（被称为 _基本情况_）或者第二个定义，确保每次调用都会离我们的基本情况更近一步。

## Reduce 和 map 算法

现在让我们来看看如何利用递归的威力来对一个列表的数字求和：

```elixir
defmodule Math do
  def sum_list([head|tail], accumulator) do
    sum_list(tail, head + accumulator)
  end

  def sum_list([], accumulator) do
    accumulator
  end
end

Math.sum_list([1, 2, 3], 0) #=> 6
```
我们用列表 `[1, 2, 3]` 和初始化值 `0` 作为参数来调用 `sum_list`。我们会尝试每个分句，直到我们找到匹配模式匹配规则的那个。在这个例子中，列表 `[1, 2, 3]` 匹配 `[head|tail]` 并且 `head` 被绑定为 `1`， `tail` 被绑定为 `[2, 3]`； `accumulator` 被设为 `0`。

然后，我们把列表的头部与累加器相加 `head + accumulator` 然后再次递归地调用 `sum_list` ，传入列表的尾部作为第一个参数。尾部会再一次匹配 `[head|tail]` 直到列表为空， 如下所示：

```elixir
sum_list [1, 2, 3], 0
sum_list [2, 3], 1
sum_list [3], 3
sum_list [], 6
```

当列表为空时，它会匹配最后一个分句，并且返回最终结果 `6`。

这种把一个列表 _递减_ 到一个值的过程被称为 _递减算法_ ，它是函数式编译的核心。

如果我们想要把列表中的值加倍，该怎么办呢？

```elixir
defmodule Math do
  def double_each([head|tail]) do
    [head * 2|double_each(tail)]
  end

  def double_each([]) do
    []
  end
end

Math.double_each([1, 2, 3]) #=> [2, 4, 6]
```

这里我们用递归来遍历一个列表，把其中的元素乘二，然后返回一个新的列表。这种把一个列表的元素对应地 _映射_ 到新的列表的过程，被称作 _映射算法_。

递归和 [尾调用](http://en.wikipedia.org/wiki/Tail_call) 优化是 Elixir 中的一个重要组成部分，常用于创建循环。然而，在用 Elixir 编程的时候你会很少会用到如上的递归去操作一个列表。

我们将在下一章中看到的 [`Enum` 模块](/docs/stable/elixir/#!Enum.html) 已经为列表提供了很多便利。举个例子，上面的例子可以被写成这样：

```iex
iex> Enum.reduce([1, 2, 3], 0, fn(x, acc) -> x + acc end)
6
iex> Enum.map([1, 2, 3], fn(x) -> x * 2 end)
[2, 4, 6]
```

或者，使用捕获语法：

```iex
iex> Enum.reduce([1, 2, 3], 0, &+/2)
6
iex> Enum.map([1, 2, 3], &(&1 * 2))
[2, 4, 6]
```

让我们来深入了解下可枚举类型（`Enumerable`）和它们对应惰性求值的流（`Stream`）。
