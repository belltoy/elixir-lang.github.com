---
layout: getting-started
title: case, cond 和 if
redirect_from: /getting_started/5.html
---

# {{ page.title }}

{% include toc.html %}

在本章中，我们将会学习关于 `case`, `cond` 和 `if` 的流程控制结构。

## `case`

`case` 允许我们将一个值与多个模式对比，直到我们找到匹配的那一个：

```iex
iex> case {1, 2, 3} do
...>   {4, 5, 6} ->
...>     "This clause won't match"
...>   {1, x, 3} ->
...>     "This clause will match and bind x to 2 in this clause"
...>   _ ->
...>     "This clause would match any value"
...> end
```

如果你要匹配一个已存在的变量，你需要用到 `^` 操作符：

```iex
iex> x = 1
1
iex> case 10 do
...>   ^x -> "Won't match"
...>   _  -> "Will match"
...> end
```

分句也允许通过守卫指定附加条件：

```iex
iex> case {1, 2, 3} do
...>   {1, x, 3} when x > 0 ->
...>     "Will match"
...>   _ ->
...>     "Won't match"
...> end
```

上面的第一个分句只有当 `x` 为正时才会匹配。

##守卫语句中的表达式

在 Erlang 虚拟机 (VM) 中，只允许一些有限的表达式作为守卫：

* 比较操作符 (`==`, `!=`, `===`, `!==`, `>`, `<`, `<=`, `>=`)
* 布尔操作符 (`and`, `or`) 和 非操作符 (`not`, `!`)
* 算术操作符 (`+`, `-`, `*`, `/`)
* 只要左侧是一个字面值就可以用 `<>` 和 `++`
* `in` 操作符
* 所有下列的类型检查函数：
    * `is_atom/1`
    * `is_binary/1`
    * `is_bitstring/1`
    * `is_boolean/1`
    * `is_float/1`
    * `is_function/1`
    * `is_function/2`
    * `is_integer/1`
    * `is_list/1`
    * `is_map/1`
    * `is_nil/1`
    * `is_number/1`
    * `is_pid/1`
    * `is_port/1`
    * `is_reference/1`
    * `is_tuple/1`
* 加上这些函数：
    * `abs(number)`
    * `bit_size(bitstring)`
    * `byte_size(bitstring)`
    * `div(integer, integer)`
    * `elem(tuple, n)`
    * `hd(list)`
    * `length(list)`
    * `map_size(map)`
    * `node()`
    * `node(pid | ref | port)`
    * `rem(integer, integer)`
    * `round(number)`
    * `self()`
    * `tl(list)`
    * `trunc(number)`
    * `tuple_size(tuple)`

另外，用户可能会定义他们自己的守卫函数，通常以 "is_" 开头。

记住守卫中的错误是不可见的，但是会使守卫失败：

```iex
iex> hd(1)
** (ArgumentError) argument error
    :erlang.hd(1)
iex> case 1 do
...>   x when hd(x) -> "Won't match"
...>   x -> "Got: #{x}"
...> end
"Got 1"
```

如果没有一个分句匹配，会引发一个错误：

```iex
iex> case :ok do
...>   :error -> "Won't match"
...> end
** (CaseClauseError) no case clause matching: :ok
```

注意匿名函数也可以有多个分句和守卫：

```elixir
iex> f = fn
...>   x, y when x > 0 -> x + y
...>   x, y -> x * y
...> end
#Function<12.71889879/2 in :erl_eval.expr/5>
iex> f.(1, 3)
4
iex> f.(-1, 3)
-3
```

匿名函数中每个分句的参数数量必须相同，否则会报错。

## `cond`

当你需要匹配不同的值的时候 `case` 很有用。然而，在很多情况下，我们要检查不同的条件并找到第一个为真。在这种情况下，可以使用 `cond`：

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This will not be true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   1 + 1 == 2 ->
...>     "But this will"
...> end
"But this will"
```

这等价于很多命令式语言中的 `else if` 分句 （尽管 Elixir 中很少用到）。

如果没有条件返回真，会报错。因此，可能有必要加一个最终条件为 `true`，这样总能匹配到：

```iex
iex> cond do
...>   2 + 2 == 5 ->
...>     "This is never true"
...>   2 * 2 == 3 ->
...>     "Nor this"
...>   true ->
...>     "This is always true (equivalent to else)"
...> end
```

最后，注意 `cond` 会把除了 `nil` 和 `false` 之后的所有值认为真：

```iex
iex> cond do
...>   hd([1,2,3]) ->
...>     "1 is considered as true"
...> end
"1 is considered as true"
```

## `if` 和 `unless`

除了 `case` 和 `cond`， 当你只需要检查一个条件的时候， Elixir 还提供了 `if/2` 和 `unless/2` 这两个有用的宏：

```iex
iex> if true do
...>   "This works!"
...> end
"This works!"
iex> unless true do
...>   "This will never be seen"
...> end
nil
```

如果给到 `if/2` 的条件为 `false` 或 `nil`， 在 `do/end` 之间的代码就不会执行，而是简单地返回 `nil`。 `unless/2` 则相反。

它们还支持 `else` 块：

```iex
iex> if nil do
...>   "This won't be seen"
...> else
...>   "This will"
...> end
"This will"
```

> 注意： 有趣的是 `if/2` 和 `unless/2` 在这个语言中是以宏的形式实现的； 它们不是特殊的语言结构，不像其它语言中那样。 你可以在 [`Kernel` 模块文档](/docs/stable/elixir/#!Kernel.html) 中查阅 `if/2` 的文档和代码。 类似 `+/2` 这样的操作符和 `is_function/2` 这样的函数都定义在 `Kernel` 模块中，默认会自动引入你的代码。

## `do/end` 块

到目前为止，我们已经学习了四种控制结构： `case`, `cond`, `if` 和 `unless`, 他们会被 `do/end` 块所包围。 往往 `if` 也可以写成如下的形式：

```iex
iex> if true, do: 1 + 2
3
```

在 Elixir 中， `do/end` 块是把一组表达式传给 `do:` 的简便写法。 下面两者是等价的：

```iex
iex> if true do
...>   a = 1 + 2
...>   a + 10
...> end
13
iex> if true, do: (
...>   a = 1 + 2
...>   a + 10
...> )
13
```

我们说第二种语法是用到了 **关键字列表**。 我们也可以用这种语法传递 `else`：

```iex
iex> if false, do: :this, else: :that
:that
```

要牢记的是，当使用 `do/end` 块时，它们总是与最外层的函数绑定。例如，下面的表达式：

```iex
iex> is_number if true do
...>  1 + 2
...> end
```

会被解析成：

```iex
iex> is_number(if true) do
...>  1 + 2
...> end
```

这会导致 Elixir 尝试去调用 `is_number/2` 而引发一个未定义函数的错误。 加上圆括号可以解决这种歧义：

```iex
iex> is_number(if true do
...>  1 + 2
...> end)
true
```

关键字列表在这个语言中扮演了一个很重要的角色，在很多函数和宏中都很普遍。我们将在以后的一章中对它们进行更进一步的探索。 现在是时候谈谈关于 「二进制，字符串和字符列表」了。
