---
layout: getting-started
title: 关键字， map 和字典
redirect_from: /getting_started/7.html
---

# {{ page.title }}

{% include toc.html %}

迄今为止我们还没讨论过任何关联数据结构，即能够关联一个值（或者多个值）到一个键的数据结构。不同的语言对它们有不同的称呼，像字典，哈希，关联数组，映射，等等。

在 Elixir 中，我们有两种关联数据结构： 关键字列表（keyword list）和 map。是时候来更多地了解下它们了！

## 关键字列表

有很多函数式语言中，经常用一个二元元组的列表来表示一个关联数据结构。 在 Elixir 中，当我们有一个元组的列表并且每个元组的第一项（即键）是一个原子的时候，我们称之为关键字列表：

```iex
iex> list = [{:a, 1}, {:b, 2}]
[a: 1, b: 2]
iex> list == [a: 1, b: 2]
true
iex> list[:a]
1
```

正如你以上看到的， Elixir 支持一个特殊的语法来定义这样的列表，在底层只不过是映射到一个元组的列表。由于它们是简单的列表，所有对列表可用的操作，包括它们的性能特征，也都适用于关键字列表。

例如，我们可以用 `++` 把一个新值加入到一个关键字列表：

```iex
iex> list ++ [c: 3]
[a: 1, b: 2, c: 3]
iex> [a: 0] ++ list
[a: 0, a: 1, b: 2]
```

注意在查找的时候，排在前面的值会被取到：

```iex
iex> new_list = [a: 0] ++ list
[a: 0, a: 1, b: 2]
iex> new_list[:a]
0
```

关键字列表非常重要，因为它们有三个特性：

  * 键必须是原子。
  * 键是有序的，由开发者指定。
  * 键可以出现多次。

例如，在 [Ecto 库](https://github.com/elixir-lang/ecto) 中用到了这些特性来提供了一个优雅的 DSL 来编写数据库查询：

```elixir
query = from w in Weather,
      where: w.prcp > 0,
      where: w.temp < 20,
     select: w
```

这些特性使得关键字列表成为了 Elixir 中给函数传递选项的默认机制。 第 5 章中，在我们讨论 `if/2` 宏的时候，我们提到了如下的语法是被支持的：

```iex
iex> if false, do: :this, else: :that
:that
```

这里的 `do:` 和 `else:` 对就是关键字列表！事实上，以上的调用等价于：

```iex
iex> if(false, [do: :this, else: :that])
:that
```

一般来说，当关键字列表是一个函数的最后一个参数的时候，方括号是可选的。

为了操作关键字列表， Elixir 提供了 [`Keyword` 模块](/docs/stable/elixir/#!Keyword.html)。记住尽管关键字列表只是简单的列表，它们还是提供了与列表一样的线性性能特性。列表越长，查找一个 key 的时候也越长，计算元素数量的时间也越长，等等。正由于这个原因，关键字列表在 Elixir 中主要被用作选项。如果你需要存储很多元素，或者找出哪个键的值最大，你应该使用 map 替代。

虽然我们可以在关键字列表上作模式匹配，但是在实际中很少用到，因为列表上的模式匹配需要列表的大小和元素顺序都匹配：

```iex
iex> [a: a] = [a: 1]
[a: 1]
iex> a
1
iex> [a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
iex> [b: b, a: a] = [a: 1, b: 2]
** (MatchError) no match of right hand side value: [a: 1, b: 2]
```

## Map

在 Elixir 中，只要是当你需要使用 key-value 存储的时候， map 都是最佳的数据结构选择。创建一个 map 需要用 `%{}` 语法：

```iex
iex> map = %{:a => 1, 2 => :b}
%{2 => :b, :a => 1}
iex> map[:a]
1
iex> map[2]
:b
iex> map[:c]
nil
```

与关键字列表对比，我们已经能看到不同之处了：

  * map 允许任意值作为键。
  * map 的键没有特定的顺序。

如果你在创建 map 的时候传入了重复的键，只会取最后一个键：

```iex
iex> %{1 => 1, 1 => 2}
%{1 => 2}
```

当一个 map 中所有的键都是原子时，为了方便你可以用关键字语法：

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
```

与关键字列表不同的是， map 在模式匹配中非常有用：

```iex
iex> %{} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> %{:a => a} = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}
iex> a
1
iex> %{:c => c} = %{:a => 1, 2 => :b}
** (MatchError) no match of right hand side value: %{2 => :b, :a => 1}
```

如上所示，只要给出的一个 map 里存在给定的键，就能匹配成功。所以，一个空 map 匹配所有的 map。

[`Map` 模块](/docs/stable/elixir/#!Map.html) 提供了一个与 `Keyword` 模块非常相近的 API，提供了一些方便的函数来操作 map：

```iex
iex> Map.get(%{:a => 1, 2 => :b}, :a)
1
iex> Map.to_list(%{:a => 1, 2 => :b})
[{2, :b}, {:a, 1}]
```

关于 map 有一个有趣的特点，它们提供了一个特殊的语法来更新和访问原子的键：

```iex
iex> map = %{:a => 1, 2 => :b}
%{:a => 1, 2 => :b}

iex> map.a
1
iex> map.c
** (KeyError) key :c not found in: %{2 => :b, :a => 1}

iex> %{map | :a => 2}
%{:a => 2, 2 => :b}
iex> %{map | :c => 3}
** (ArgumentError) argument error
```

上面无论是访问还是更新语法，都需要给定的键存在。例如， 在这个 map 中不存在 `:c` 键，所以访问和更新 `:c` 键的操作都会失败。

在使用 map 的时候， Elixir 开发者一般习惯使用 `map.field` 语法和模式匹配来替代 `Map` 模块中的函数，因为它们代表了自信的编程风格。[这篇博客文章](http://blog.plataformatec.com.br/2014/09/writing-assertive-code-with-elixir/) 给出了关于如何让你在 Elixir 中用自信的代码写出更加简洁快速的软件的观点和例子。

> 注意： Map 是最近才在 [EEP 43](http://www.erlang.org/eeps/eep-0043.html "Erlang Enhancement Proposal #43: Maps") 中加入到 Erlang <abbr title="Virtual Machine">虚拟机</abbr> 中的。 Erlang 17 提供了 <abbr title="Erlang Enhancement Proposal">EEP</abbr> 一部分的实现，只支持 「小 map」。 这意味着只有当最多只保存着几十个键的时候才有性能保证。为了弥补这一点， Elixir 还提供了 [`HashDict` 模块](/docs/stable/elixir/#!HashDict.html) ，用一个哈希算法提供了一个支持几十万键的高性能字典。

## 字典

在 Elixir 中，关键字列表和 map 都被称为字典。 换句话说，一个字典就像一个接口 （我们在 Elixir 中称它们为行为，behaviour），关键字列表和 map 模块都实现了这个接口。

这个接口定义在 [`Dict` 模块](/docs/stable/elixir/#!Dict.html) 中，同时还提供了一个委托给底层实现的 API：

```iex
iex> keyword = []
[]
iex> map = %{}
%{}
iex> Dict.put(keyword, :a, 1)
[a: 1]
iex> Dict.put(map, :a, 1)
%{a: 1}
```

`Dict` 模块允许任何开发者实现他们自己不同的字典，拥有自己的特性，并且整合到现有的 Elixir 代码中。 `Dict` 模块还提供了一些在所有字典类型中通用的函数。例如 `Dict.equal?/2` 可以比较任意类型的两个字典值。

也就是说，你也许会有疑问， 你在代码中到底应该使用 `Keyword`，`Map` 或者 `Dict` 哪个模块？答案是：看情况。

如果你的代码期望一个具体的数据结构作为参数，那就使用能导致写出更多自信的代码的相应模块。例如，如果你期望一个关键字作为参数，那就直接用 `Keyword` 模块替代 `Dict`。然而，如果你的 API 要处理任意的字典，那就使用 `Dict` 模块 （并确保不同的字典类型实现都能通过测试）。

Elixir 中关联数据结构的介绍就到此为止了。你会发现，有了关键字列表和 map，面对 Elixir 中任何需要用到关联数据结构的问题时，你总有适当的工具。
