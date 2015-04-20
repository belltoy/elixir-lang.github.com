---
layout: getting-started
title: alias, require 和 import
redirect_from: /getting_started/13.html
---

# {{ page.title }}

{% include toc.html %}

为了方便软件的复用， Elixir 提供了三个指令。正如我们将在以下看到的，它们被称作指令是因为它们有 **词法作用域**。

## `alias`

`alias` 允许你给任意给定的模块设置别名。想像下我们的 `Math` 模块使用一个特殊的列表实现用于执行特殊的数学操作：

```elixir
defmodule Math do
  alias Math.List, as: List
end
```

从现在起，所有对 `List` 的引用都会被自动扩展成 `Math.List`。假如你想要访问原始的 `List` 模块，可以给模块名加上 `Elixir.` 前缀：

```elixir
List.flatten             #=> uses Math.List.flatten
Elixir.List.flatten      #=> uses List.flatten
Elixir.Math.List.flatten #=> uses Math.List.flatten
```

> 注意： Elixir 中定义的所有模块都是定义在一个主命名空间 Elixir 下的。然而，为了方便，在引用它们的时候你可以省略 "Elixir."。

别名经常用于定义快捷写法。事实上，调用 `alias` 的时候没有带 `:as` 选项，会自动将别名设置为模块名的最后一个部分，例如：

```elixir
alias Math.List
```

相当于:

```elixir
alias Math.List, as: List
```

注意到 `alias` 是 **词法作用域** 的，它允许你在指定的函数里设置别名：

```elixir
defmodule Math do
  def plus(a, b) do
    alias Math.List
    # ...
  end

  def minus(a, b) do
    # ...
  end
end
```

在上面的例子中，因为我们在函数 `plus/2` 中调用了 `alias`，别名只会在函数 `plus/2` 中有效，而 `minus/2` 完全不会受到影响。

## `require`

Elixir 提供了宏作为元编程（编写代码来生成代码）的一个机制。

宏是一段在编译时被执行是展开的代码。这意味着，为了使用宏，我们需要保证它的模块和实现在编译时是可用的。这可以用 `require` 指令来完成：

```iex
iex> Integer.is_odd(3)
** (CompileError) iex:1: you must require Integer before invoking the macro Integer.is_odd/1
iex> require Integer
nil
iex> Integer.is_odd(3)
true
```

在 Elixir 中， `Integer.is_odd/1` 是被定义成宏的，因此它可以用作守卫。这意味着，为了调用 `Integer.is_odd/1`，我们需要先 require `Integer` 模块。

一般来说模块在使用前是不需要 require 的，除非如果你想在那个模块中使用宏。尝试调用一个未加载的宏会引发一个错误。注意与 `alias` 指令一样，`require` 也是词法作用域。我们将在以后的章节中深入讨论。

## `import`

当我们要简单地访问其它模块中的函数或宏而不用写全名，我们使用 `import` 。举个例子，如果我们想要多次使用 `List` 模块中的 `duplicate/2` 函数，我们可以 import 它：

```iex
iex> import List, only: [duplicate: 2]
nil
iex> duplicate :ok, 3
[:ok, :ok, :ok]
```

在这个例子中，我们只从 `List` 中引入了 `duplicate` （元数为 2）函数。虽然 `:only` 是可选的，但还是推荐使用，因为这样做可以防止引入给定的模块中的所有函数到命名空间内。 还有一个 `:except` 选项，可以用于引入一个模块中的所有函数时 *排除* 一些函数。

`import` 还可以将 `:only` 选项指定为 `:macros` 或 `:functions` 。例如，要引入所有的宏，可以这么写：

```elixir
import Integer, only: :macros
```

或者要引入所有的函数，你可以这么写：

```elixir
import Integer, only: :functions
```

注意 `import` 也是 **词法作用域** 的。这意味着我们可以在函数定义中引入指定的宏或者函数。

```elixir
defmodule Math do
  def some_function do
    import List, only: [duplicate: 2]
    duplicate(:ok, 10)
  end
end
```

在上面的例子中，被引入的 `List.duplicate/2` 只在这个特定的函数中可见。 `duplicate/2` 在这个模块（或者其它模块中）的其它函数中不可用。

注意使用 `import` 引入一个模块的时候，会自动 `require` 它。

## 别名

此时此刻，你也许会在想： Elixir 别名到底是什么，是如何表示的？

Elixir 中的别名是一个首字母大写的标识符（如 `String`，`Keyword` 等等），它会在编译的时候被转换成一个原子。举个例子， `String` 别名默认会被转化为原子 `:"Elixir.String"`：

```iex
iex> is_atom(String)
true
iex> to_string(String)
"Elixir.String"
iex> :"Elixir.String" == String
true
```

使用 `alias/2` 指令，我们改变了别名转化的结果。

别名之所以能够工作，是因为在 Erlang <abbr title="Virtual Machine">VM</abbr>  （同样地在 Elixir）中，模块用原子表示。如下例子所示，这是我们用来调用 Erlang 模块的机制：

```iex
iex> :lists.flatten([1, [2], 3])
[1, 2, 3]
```

这也是允许我们动态地调用一个模块中的函数的机制：

```iex
iex> mod = :lists
:lists
iex> mod.flatten([1, [2], 3])
[1, 2, 3]
```

我们简单地在原子 `:lists` 上调用了函数 `flatten`。

## 嵌套

现在我们已经讨论了别名，我们现在可以讨论嵌套以及它在 Elixir 中是如何工作的。考虑以下的例子：

```elixir
defmodule Foo do
  defmodule Bar do
  end
end
```

上面的例子中将会定义两个模块： `Foo` 和 `Foo.Bar`。第二个可以在 `Foo` 中用 `Bar` 访问，因为它们在同一个词法作用域下。

如果以后将 `Bar` 模块从 `Foo` 模块的定义中移走，就需要用命名（`Foo.Bar`）来引用它，或者像之前讨论的那样用 `alias` 指令设置一个别名。 `Bar` 模块的定义也会改变。上面例子等价于：

```elixir
defmodule Foo.Bar do
end

defmodule Foo do
  alias Foo.Bar, as: Bar
end
```

上面的代码等同于：

```elixir
defmodule Elixir.Foo do
  defmodule Elixir.Foo.Bar do
  end
  alias Elixir.Foo.Bar, as: Bar
end
```

**注意**： 在 Elixir 中，你不需要在定义 `Foo.Bar` 模块之前先定义 `Foo` 模块，因为 Elixir 语言最终会将所有的模块名转化成原子。你可以直接定义名字上看起来是嵌套的（arbitrarily-nested）模块而不用定义名字链之前所有的模块（例如，可以直接定义 `Foo.Bar.Baz` 而不用先定义 `Foo` 或者 `Foo.Bar`）。

正如我们将在接下来的章节中看到的，别名还在宏里扮演了重要的角色，来保证它们是卫生的（hygienic）。

到此为止，我们几乎要结束我们关于 Elixir 模块的教程。下一主题是模块属性。
