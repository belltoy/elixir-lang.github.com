---
layout: getting-started
title: 模块
redirect_from: /getting_started/8.html
---

# {{ page.title }}

{% include toc.html %}

在 Elixir 里我们会把一些函数组合到一个模块中。我们在之前的章节中已经用过许多模块，例如 [字符串模块](/docs/stable/elixir/#!String.html) ：

```iex
iex> String.length "hello"
5
```

要在 Elixir 中创建我们自己的模块，我们需要用到 `defmodule` 宏。我们用 `def` 宏在模块中定义函数：

```iex
iex> defmodule Math do
...>   def sum(a, b) do
...>     a + b
...>   end
...> end

iex> Math.sum(1, 2)
3
```

在接下来的段落中，我们的例子将会变得更加复杂，很难在 shell 中完整地输入。是时候让我们来学习如何编译 Elixir 代码和如何运行 Elixir 脚本了。

## 编译

大多数时候将模块写入文件会更方便，这样就可以编译和重用它们了。假设我们有一个名为 `math.ex` 的文件，内容如下：

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end
```

这个文件可以用 `elixirc` 来编译：

```bash
$ elixirc math.ex
```

这会生成一个模块对应的字节码文件 `Elixir.Math.beam` 。如果我们再运行 `iex`，我们定义的模块就可以用了（假设 `iex` 是在字节码文件相同的目录中启动的）：

```iex
iex> Math.sum(1, 2)
3
```

Elixir 项目通常被组织在三个目录中：

* ebin - 包含编译后的字节码文件
* lib - 包含 elixir 代码 （通常是 `.ex` 文件）
* test - 包含测试 （通常是 `.exs` 文件）

在实际项目中，名用 `mix` 的构建工具会用来负责编译和为你设置路径。为了学习的目的， Elixir 还提供了一个更灵活的脚本模式，而且不会生成任何的编译产物。

## 脚本模式

除了 Elixir 的文件扩展名 `.ex`， Elixir 还支持 `.exs` 文件作为脚本。 Elixir 对两种文件一视同仁，唯一的区别在于其目的。 `.ex` 文件是会被编译，而 `.exs` 文件用作脚本而无需编译。举例来说，我们可以创建一个文件 `math.exs`：

```elixir
defmodule Math do
  def sum(a, b) do
    a + b
  end
end

IO.puts Math.sum(1, 2)
```

And execute it as:

```bash
$ elixir math.exs
```

这个文件会被在内存中编译并执行，打印出结果 「3」。 不会生成没有字节码文件。 在接下来的例子中，我们推荐你把代码写入脚本文件中并按照上述的方式执行。

## 命名的函数

在一个模块中，我们可以用 `def/2` 定义函数，用 `defp/2` 定义私有函数。用 `def/2` 定义的函数可以被其它模块调用，而私有函数只能在自身模块内部调用。

```elixir
defmodule Math do
  def sum(a, b) do
    do_sum(a, b)
  end

  defp do_sum(a, b) do
    a + b
  end
end

Math.sum(1, 2)    #=> 3
Math.do_sum(1, 2) #=> ** (UndefinedFunctionError)
```

函数定义还支持守卫语句（guard）和多分句（clause）。如果一个函数有多个分句， Elixir 会尝试每个分句直到找到匹配的那个。这是一个检查给定的数字是否是零的函数实现：

```elixir
defmodule Math do
  def zero?(0) do
    true
  end

  def zero?(x) when is_number(x) do
    false
  end
end

Math.zero?(0)  #=> true
Math.zero?(1)  #=> false

Math.zero?([1,2,3])
#=> ** (FunctionClauseError)
```

给出一个不匹配任何分句的参数会引发一个错误。

## 函数捕获 (Function capturing)

在这个教程中，我们一直会用记号 `name/arity` 来引用函数。恰好这种记法实际上也可以用来获取一个命名函数作为一个函数类型。我们打开 `iex` 并运行上面定义的 `math.exs` 文件：

```bash
$ iex math.exs
```

```iex
iex> Math.zero?(0)
true
iex> fun = &Math.zero?/1
&Math.zero?/1
iex> is_function fun
true
iex> fun.(0)
true
````

本地函数或引入的函数，如 `is_function/1`，可以被捕获而不用写模块名：

```iex
iex> &is_function/1
&:erlang.is_function/1
iex> (&is_function/1).(fun)
true
```

注意捕获语法也可以用作创建函数的快捷方式：

```iex
iex> fun = &(&1 + 1)
#Function<6.71889879/1 in :erl_eval.expr/5>
iex> fun.(1)
2
```

`&1` 相当于传入函数的第一个参数。 上面的 `&(&1+1)` 等价于 `fn x -> x + 1 end` 。 上面的语法在定义短函数的时候很有用。

同样地，如果你想调用一个模块中的函数，你可以这样写 `&Module.function()` ：

```iex
iex> fun = &List.flatten(&1, &2)
&List.flatten/2
iex> fun.([1, [[2], 3]], [4, 5])
[1, 2, 3, 4, 5]
```
`&List.flatten(&1, &2)` 相当于这种写法 `fn(list, tail) -> List.flatten(list, tail) end` 。 你可以在 [`Kernel.SpecialForms` 文档](/docs/stable/elixir/#!Kernel.SpecialForms.html#&/1) 中了解更多关于捕获操作符。

## 默认参数

Elixir 中的命名函数还支持默认参数：

```elixir
defmodule Concat do
  def join(a, b, sep \\ " ") do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
```

任何表达式都可以作为一个默认值，但不会在函数定义的时候执行；它会被简单地保存起来，以备之后使用。每次函数被调用的时候，所有的默认值都会被用到，默认参数的表达式会被执行：

```elixir
defmodule DefaultTest do
  def dowork(x \\ IO.puts "hello") do
    x
  end
end
```

```iex
iex> DefaultTest.dowork
hello
:ok
iex> DefaultTest.dowork 123
123
iex> DefaultTest.dowork
hello
:ok
```

如果一个带有默认参数的函数有多个分句，建议创建一个函数头 （不带实际的函数体），用于声明默认参数：

```elixir
defmodule Concat do
  def join(a, b \\ nil, sep \\ " ")

  def join(a, b, _sep) when is_nil(b) do
    a
  end

  def join(a, b, sep) do
    a <> sep <> b
  end
end

IO.puts Concat.join("Hello", "world")      #=> Hello world
IO.puts Concat.join("Hello", "world", "_") #=> Hello_world
IO.puts Concat.join("Hello")               #=> Hello
```

当用到默认值的时候，一个必须小心的地方是要避免覆盖函数定义。 考虑下面的例子：

```elixir
defmodule Concat do
  def join(a, b) do
    IO.puts "***First join"
    a <> b
  end

  def join(a, b, sep \\ " ") do
    IO.puts "***Second join"
    a <> sep <> b
  end
end
```

如果我们将上面的代码保存在一个名为 「concat.ex」的文件中，并编译它， Elixir 会发出下面的警告：

    concat.ex:7: this clause cannot match because a previous clause at line 2 always matches

编译器告诉我们当我们调用带有两个参数的 `join` 函数的时候会总是选择 `join` 的第一个定义，而第二个只会在传进三个参数的时候被调用：

```bash
$ iex concat.exs
```

```iex
iex> Concat.join "Hello", "world"
***First join
"Helloworld"
```

```iex
iex> Concat.join "Hello", "world", "_"
***Second join
"Hello_world"
```

简短的模块介绍到此为止。在接下来章节中，我们将会学到如何用命名函数实现递归，了解 Elixir 中可以用于从其它模块引入函数的词法指令，讨论模块属性。
