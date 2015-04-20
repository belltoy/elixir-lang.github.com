---
layout: getting-started
title: Structs
redirect_from: /getting_started/15.html
redirect_from: /getting-started/struct.html
---

# {{ page.title }}

{% include toc.html %}

在 [第七章](/getting-started/maps-and-dicts.html) 中我们已经学习了 map：

```iex
iex> map = %{a: 1, b: 2}
%{a: 1, b: 2}
iex> map[:a]
1
iex> %{map | a: 3}
%{a: 3, b: 2}
```

Struct 只是构建在 map 之上的扩展，它提供了编译期检查和默认值。

## 定义 struct

定义一个 struct，要用到 `defstruct` 语法结构：

```iex
iex> defmodule User do
...>   defstruct name: "John", age: 27
...> end
```

用在 `defstruct` 后面的关键字列表定义了这个 struct 的字段，并且指定了他们的默认值。

Struct 使用定义它们的模块名作为名字。在上面的例子中，我们定义了一个名为 `User` 的 struct。

我们现在可以通过使用一个类似用于创建 map 的语法来创建一个 `User` struct：

```iex
iex> %User{}
%User{age: 27, name: "John"}
iex> %User{name: "Meg"}
%User{age: 27, name: "Meg"}
```

Struct 提供了 *编译期* 的保证，只有用 `defstruct` 定义的字段（它们中的 *全部*）才会被允许存在于 struct 中：

```iex
iex> %User{oops: :field}
** (CompileError) iex:3: unknown key :oops for struct User
```

## 访问和更新 struct

当我们谈论到 map 的时候，我们展示了我们如何访问和更新一个 map 的字段。同样的技术（和同样的语法）也适用于 struct：

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john.name
"John"
iex> meg = %{john | name: "Meg"}
%User{age: 27, name: "meg"}
iex> %{meg | oops: :field}
** (ArgumentError) argument error
```

当使用更新语法 （`|`） 的时候，<abbr title="Virtual Machine">VM</abbr> 知道到没有新的键会被加入到 struct 中，允许底层的 map 在内存中共享它们的结构。在上面的例子中， `john` 和 `meg` 在内存中共享了键结构。

Struct 同样可以用在模式匹配，无论是在指定键的值上匹配还是匹配一个 struct 的类型。

```iex
iex> %User{name: name} = john
%User{age: 27, name: "John"}
iex> name
"John"
iex> %User{} = %{}
** (MatchError) no match of right hand side value: %{}
```

## Struct 内部只是裸 map

在上面的例子中，模式匹配可以工作是因为在底层 struct 只是裸 map 和一个固定大小的字段集。与 map 相比，struct 保存了一个名为 `__struct__` 的特殊字段，用于保存 struct 的名字：

```iex
iex> is_map(john)
true
iex> john.__struct__
User
```

注意到我们提到 struct 的时候称 **裸** map，是因为 map 实现的协议都无法用于 struct。例如，你不能枚举或访问一个 struct：

```iex
iex> john = %User{}
%User{age: 27, name: "John"}
iex> john[:name]
** (Protocol.UndefinedError) protocol Access not implemented for %User{age: 27, name: "John"}
iex> Enum.each john, fn({field, value}) -> IO.puts(value) end
** (Protocol.UndefinedError) protocol Enumerable not implemented for %User{age: 27, name: "John"}
```

一个 struct 也不是一个字典，因此也不能用于 `Dict` 模块中的函数：

```iex
iex> Dict.get(%User{}, :name)
** (UndefinedFunctionError) undefined function: User.fetch/2
```

然而，由于 struct 只是 map，你可以将它们用于 `Map` 模块中的函数：

```iex
iex> kurt = Map.put(%User{}, :name, "Kurt")
%User{age: 27, name: "Kurt"}
iex> Map.merge(kurt, %User{name: "Takashi"})
%User{age: 27, name: "Takashi"}
iex> Map.keys(john)
[:__struct__, :age, :name]
```

在下一章中我们将涉及 struct 如何与协议交互。
