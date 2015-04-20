---
title: "Erlang/Elixir 语法：速成课"
section: home
layout: default
---

# {{ page.title }}

这是一个针对 Erlang 开发者的 Elixir 语法快速介绍，反之亦然。 对于理解 Elixir/Erlang 代码，支持互操作性，阅读文档，示例代码等等来说，这绝对是一份对你的知识要求最少的教程。

{% include toc.html %}

## 运行代码

### Erlang

运行一些代码最快的方式是启动一个 Erlang shell ———— `erl` 。 这个页面上的许多代码片断可以被直接粘贴到 shell 里。 然而，如果你要定义一个命名函数， Erlang 期望把它写到一个模块里，然后模块必须被编译。这里是一个模块的骨架：

```erlang
% module_name.erl
-module(module_name).  % you may use some other name
-compile(export_all).

hello() ->
  io:format("~s~n", ["Hello world!"]).
```
把你的函数添加进去，保存到磁盘，在同一个目录中运行 `erl` 并执行 `compile` 命令：

```erl
Eshell V5.9  (abort with ^G)
1> c(module_name).
ok
1> module_name:hello().
Hello world!
ok
```

当你在编辑文件的时候，你可以保持 shell 一直在运行状态下。 只要别忘了执行 `c(module_name)` 来加载最新的变化。注意到文件名必须和 `-module()` 指令中定义的相同，加上后缀名 `.erl`。

### Elixir

Elixir 同样也有一个交互式的 shell，叫 `iex`。可以用 `elixirc` （与 Erlang 的 `erlc` 相似）来编译 Elixir 的代码。 Elixir 还提供了一个可执行文件 `elixir` 来运行 Elixir 代码。 上面例子中定义的模块可以用 Elixir 来写：

```elixir
# module_name.ex
defmodule ModuleName do
  def hello do
    IO.puts "Hello World"
  end
end
```

然后在 `iex` 中编译：

```iex
Interactive Elixir
iex> c("module_name.ex")
[ModuleName]
iex> ModuleName.hello
Hello world!
:ok
```

然而要注意的是在 Elixir 中如果只要创建一个模块，你不需要创建一个文件， Elixir 模块可以直接在 shell 中定义：

```elixir
defmodule MyModule do
  def hello do
    IO.puts "Another Hello"
  end
end
```


## 明显的差异

本节将介绍一些这两种语言之间的语法差异。

### 操作符名

一些操作符拼写不同。

| Erlang         | Elixir         | 含义                                     |
|----------------|----------------|-----------------------------------------|
| and            | NOT AVAILABLE  | 逻辑「与」，两边参数都计算                   |
| andalso        | and            | 逻辑「与」，短路操作                        |
| or             | NOT AVAILABLE  | 逻辑「或」，两边参数都计算                   |
| orelse         | or             | 逻辑「或」，短路操作                        |
| =:=            | ===            | 相等操作符                                |
| =/=            | !==            | 不等操作符                                |
| /=             | !=             | 不等                                     |
| =<             | <=             | 小于或等于                                |


### 分隔符

Erlang 表达式是用点号 `.` 结束的，逗号 `,` 用于在一个上下文里（例如，在一个函数定义中）计算多个表达式。 在 Elixir 中，表达式用换行符或者分号 `;` 分隔。

**Erlang**

```erlang
X = 2, Y = 3.
X + Y.
```

**Elixir**

```elixir
x = 2; y = 3
x + y
```

### 变量名

Erlang 中的变量只能赋值一次。 Erlang shell 提供了一个特殊的命令 `f` 允许你一次性擦除一个变量或所有变量的绑定。

Elixir 允许你对一个变量赋值多次。如果你要对一个已经赋值的变量作匹配，你要用 `^`：

**Erlang**

```erl
Eshell V5.9  (abort with ^G)
1> X = 10.
10
2> X = X + 1.
** exception error: no match of right hand side value 11
3> X1 = X + 1.
11
4> f(X).
ok
5> X = X1 * X1.
121
6> f().
ok
7> X.
* 1: variable 'X' is unbound
8> X1.
* 1: variable 'X1' is unbound
```

**Elixir**

```iex
iex> a = 1
1
iex> a = 2
2
iex> ^a = 3
** (MatchError) no match of right hand side value: 3
```

### 调用函数

Elixir 允许你在调用函数的时候省略括号， Erlang 却不允许这么做。

| Erlang            | Elixir         |
|-------------------|----------------|
| some_function().  | some_function  |
| sum(A, B)         | sum a, b       |

调用一个模块中的函数使用不同的语法。 有 Erlang 中， 你会这么写

```erlang
orddict:new().
```

来调用 `orddict` 模块中的 `new` 函数。 在 Elixir 中，用点号 `.` 替代冒号 `:`

```elixir
Orddict.new
```

**注意**： 因为 Erlang 模块是用原子表示，你应该用如下的方式在 Elixir 中调用 Erlang 函数：

```elixir
:lists.sort [3, 2, 1]
```

所有的 Erlang 内建函数都在 `:erlang` 模块中。


## 数据类型

很大程度上 Erlang 和 Elixir 有着相同的数据类型，但有一些不同。

### 原子

在 Erlang 中，一个 `atom` 是一个以小写字母开头的标识符，如 `ok`，`tuple`，`donut`。 大写字母开头的标识符总是作为变量名。 另一方面，在 Elixir 中，使用前者作为变量名，后者作为原子的别名。 Elixir 中的原子都是用冒号 `:` 开头。

**Erlang**

```erlang
im_an_atom.
me_too.

Im_a_var.
X = 10.
```

**Elixir**

```elixir
:im_an_atom
:me_too

im_a_var
x = 10

Module  # 这个被称作原子别名；它会被扩展成 :'Elixir.Module'
```

创建一个不以小写字母开头的原子也是可以的。只是这两个语言的语法不同：

**Erlang**

```erlang
is_atom(ok).                %=> true
is_atom('0_ok').            %=> true
is_atom('Multiple words').  %=> true
is_atom('').                %=> true
```

**Elixir**

```elixir
is_atom :ok                 #=> true
is_atom :'ok'               #=> true
is_atom Ok                  #=> true
is_atom :"Multiple words"   #=> true
is_atom :""                 #=> true
```

### 元组

在两个语言中元组的语法是相同的，但 API 却不同。 Elixir 尝试用以下的方式规范 Erlang 的库：

1. 函数的 `subject` 总是第一个参数。
2. 所有的数据结构函数使用从零开始的访问。

不过， Elixir 不提供默认的 `element` 和 `setelement` 函数，替代它们的是 `elem` 和 `put_elem`：

**Erlang**

```erlang
element(1, {a, b, c}).       %=> a
setelement(1, {a, b, c}, d). %=> {d, b, c}
```

**Elixir**

```elixir
elem({:a, :b, :c}, 0)         #=> :a
put_elem({:a, :b, :c}, 0, :d) #=> {:d, :b, :c}
```

### 列表和二进制

Elixir 有一个针对二进制的快捷语法：

**Erlang**

```erlang
is_list('Hello').        %=> false
is_list("Hello").        %=> true
is_binary(<<"Hello">>).  %=> true
```

**Elixir**

```elixir
is_list 'Hello'          #=> true
is_binary "Hello"        #=> true
is_binary <<"Hello">>    #=> true
<<"Hello">> === "Hello"  #=> true
```

在 Elixir 中， **字符串** 这个词的意思是指一个 UTF-8 的二进制，并且有一个 `String` 模块用来处理这种数据。 Elixir 还期望你的源文件是用 UTF-8 编码的。另一方面， Erlang 中的 **字符串** 指的是字符列表，并且有一个非 UTF-8 感知的 `:string` 模块用于处理字符列表。

Elixir 还支持多行字符串 （又被称作 *heredocs*）：

```elixir
is_binary """
This is a binary
spanning several
lines.
"""
#=> true
```

### 关键字列表

Elixir 提供了一个字面语法来创建一个第一个元素是原子二元元组的列表，称为关键字列表：

**Erlang**

```erlang
Proplist = [{another_key, 20}, {key, 10}].
proplists:get_value(another_key, Proplist).
%=> 20
```

**Elixir**

```elixir
kw = [another_key: 20, key: 10]
kw[:another_key]
#=> 20
```

### 映射

Erlang R17 添加了映射，一个无序的 key-value 存储。键和值可以是任意类型。 如下，在两种语言中创建，更新和匹配映射：

**Erlang**

```erlang
Map = #{key => 0}.
Updated = Map#{key := 1}.
#{key := Value} = Updated.
Value =:= 1.
%=> true
```

**Elixir**

```elixir
map = %{:key => 0}
map = %{map | :key => 1}
%{:key => value} = map
value === 1
#=> true
```

如果键全部都是原子， Elixir 允许开发都用 `key: 0` 的方式来定义一个映射，并且可以用 `.key` 的方式来访问字段：

```elixir
map = %{key: 0}
map = %{map | key: 1}
map.key === 1
```

### 正则表达式

Elixir 支持一个正则表达式的字面语法。 这种语法允许正则在编译期编译而不是在运行时，并且不需要你二次转义特殊的正则字符：

**Erlang**

```erlang
{ ok, Pattern } = re:compile("abc\\s").
re:run("abc ", Pattern).
%=> { match, ["abc "] }
```

**Elixir**

```elixir
Regex.run ~r/abc\s/, "abc "
#=> ["abc "]
```

正则表达式同样支持 hrerdocs，用于方便地定义多行正则：

```elixir
Regex.regex? ~r"""
This is a regex
spanning several
lines.
"""
#=> true
```


## 模块

每个 Erlang 模块保存在各自的文件中，并且有如下的结构：

```erlang
-module(hello_module).
-export([some_fun/0, some_fun/1]).

% A "Hello world" function
some_fun() ->
  io:format('~s~n', ['Hello world!']).

% This one works only with lists
some_fun(List) when is_list(List) ->
  io:format('~s~n', List).

% Non-exported functions are private
priv() ->
  secret_info.
```

这里我们创建了一个名为 ``hello_module`` 的模块。其中我们定义了三个函数，前两个通过用开头的 ``export`` 指令可以被其它模块调用。它包含了一个函数的列表，每个函数用 ``<function name>/<arity>`` 这样的格式来写。元数（arity）指的是参数数量。

上面的 Erlang 代码用 Elixir 等效地表示如下：

```elixir
defmodule HelloModule do
  # A "Hello world" function
  def some_fun do
    IO.puts "Hello world!"
  end

  # This one works only with lists
  def some_fun(list) when is_list(list) do
    IO.inspect list
  end

  # A private function
  defp priv do
    :secret_info
  end
end
```

在 Elixir 中，可以在同一个文件中写多个模块，还可以写嵌套模块：

```elixir
defmodule HelloModule do
  defmodule Utils do
    def util do
      IO.puts "Utilize"
    end

    defp priv do
      :cant_touch_this
    end
  end

  def dummy do
    :ok
  end
end

defmodule ByeModule do
end

HelloModule.dummy
#=> :ok

HelloModule.Utils.util
#=> "Utilize"

HelloModule.Utils.priv
#=> ** (UndefinedFunctionError) undefined function: HelloModule.Utils.priv/0
```


## 函数语法

Erlang 书本中的 [这一章][3] 提供了一份关于 Erlang 中模式匹配和函数语法详细的描述。这里，我简单地概括一下要点并给出分别用 Erlang 和 Elixir 写的示例代码。

[3]: http://learnyousomeerlang.com/syntax-in-functions

### 模式匹配

Elixir 中的模式匹配是基于 Erlang 的实现的，基本上是非常相似的：

**Erlang**

```erlang
loop_through([H|T]) ->
  io:format('~p~n', [H]),
  loop_through(T);

loop_through([]) ->
  ok.
```

**Elixir**

```elixir
def loop_through([h|t]) do
  IO.inspect h
  loop_through t
end

def loop_through([]) do
  :ok
end
```

When defining a function with the same name multiple times, each such definition is called a **clause**. In Erlang, clauses always go side by side and are separated by a semicolon ``;``. The last clause is terminated by a dot ``.``.

Elixir doesn't require punctuation to separate clauses, but they must be grouped together.

### 标识函数

In both Erlang and Elixir, a function is not identified only by its name, but by its name and arity. In both examples below, we are defining four different functions (all named `sum`, but with different arity):

**Erlang**

```erlang
sum() -> 0;
sum(A) -> A;
sum(A, B) -> A + B;
sum(A, B, C) -> A + B + C.
```

**Elixir**

```elixir
def sum, do: 0
def sum(a), do: a
def sum(a, b), do: a + b
def sum(a, b, c), do: a + b + c
```

Guard expressions provide a concise way to define functions that accept a limited set of values based on some condition.

**Erlang**

```erlang
sum(A, B) when is_integer(A), is_integer(B) ->
  A + B;

sum(A, B) when is_list(A), is_list(B) ->
  A ++ B;

sum(A, B) when is_binary(A), is_binary(B) ->
  <<A/binary,  B/binary>>.

sum(1, 2).
%=> 3

sum([1], [2]).
%=> [1,2]

sum("a", "b").
%=> "ab"
```

**Elixir**

```elixir
def sum(a, b) when is_integer(a) and is_integer(b) do
  a + b
end

def sum(a, b) when is_list(a) and is_list(b) do
  a ++ b
end

def sum(a, b) when is_binary(a) and is_binary(b) do
  a <> b
end

sum 1, 2
#=> 3

sum [1], [2]
#=> [1,2]

sum "a", "b"
#=> "ab"
```

In addition, Elixir allows for default values for arguments, whereas Erlang does not.

```elixir
def mul_by(x, n \\ 2) do
  x * n
end

mul_by 4, 3 #=> 12
mul_by 4    #=> 8
```

### 匿名函数

匿名函数用以下的方式定义：

**Erlang**

```erlang
Sum = fun(A, B) -> A + B end.
Sum(4, 3).
%=> 7

Square = fun(X) -> X * X end.
lists:map(Square, [1, 2, 3, 4]).
%=> [1, 4, 9, 16]
```

**Elixir**

```elixir
sum = fn(a, b) -> a + b end
sum.(4, 3)
#=> 7

square = fn(x) -> x * x end
Enum.map [1, 2, 3, 4], square
#=> [1, 4, 9, 16]
```

定义匿名函数的时候同样也可以使用模式匹配。

**Erlang**

```erlang
F = fun(Tuple = {a, b}) ->
        io:format("All your ~p are belong to us~n", [Tuple]);
        ([]) ->
        "Empty"
    end.

F([]).
%=> "Empty"

F({a, b}).
%=> "All your {a,b} are belong to us"
```

**Elixir**

```elixir
f = fn
      {:a, :b} = tuple ->
        IO.puts "All your #{inspect tuple} are belong to us"
      [] ->
        "Empty"
    end

f.([])
#=> "Empty"

f.({:a, :b})
#=> "All your {:a, :b} are belong to us"
```


### First-class functions

Anonymous functions are first-class values, so they can be passed as arguments to other functions and also can serve as a return value. There is a special syntax to allow named functions be treated in the same manner.

**Erlang**

```erlang
-module(math).
-export([square/1]).

square(X) -> X * X.

lists:map(fun math:square/1, [1, 2, 3]).
%=> [1, 4, 9]
```

**Elixir**

```elixir
defmodule Math do
  def square(x) do
    x * x
  end
end

Enum.map [1, 2, 3], &Math.square/1
#=> [1, 4, 9]
```


### Partials in Elixir

Elixir supports partial application of functions which can be used to define anonymous functions in a concise way:

```elixir
Enum.map [1, 2, 3, 4], &(&1 * 2)
#=> [2, 4, 6, 8]

List.foldl [1, 2, 3, 4], 0, &(&1 + &2)
#=> 10
```

Partials also allow us to pass named functions as arguments.

```elixir
defmodule Math do
  def square(x) do
    x * x
  end
end

Enum.map [1, 2, 3], &Math.square/1
#=> [1, 4, 9]
```


## 控制流

The constructs `if` and `case` are actually expressions in both Erlang and Elixir, but may be used for control flow as in imperative languages.

### Case

The ``case`` construct provides control flow based purely on pattern matching.

**Erlang**

```erlang
case {X, Y} of
  {a, b} -> ok;
  {b, c} -> good;
  Else -> Else
end
```

**Elixir**

```elixir
case {x, y} do
  {:a, :b} -> :ok
  {:b, :c} -> :good
  other -> other
end
```

### If

**Erlang**

```erlang
Test_fun = fun (X) ->
  if X > 10 ->
       greater_than_ten;
     X < 10, X > 0 ->
       less_than_ten_positive;
     X < 0; X =:= 0 ->
       zero_or_negative;
     true ->
       exactly_ten
  end
end.

Test_fun(11).
%=> greater_than_ten

Test_fun(-2).
%=> zero_or_negative

Test_fun(10).
%=> exactly_ten
```

**Elixir**

```elixir
test_fun = fn(x) ->
  cond do
    x > 10 ->
      :greater_than_ten
    x < 10 and x > 0 ->
      :less_than_ten_positive
    x < 0 or x === 0 ->
      :zero_or_negative
    true ->
      :exactly_ten
  end
end

test_fun.(44)
#=> :greater_than_ten

test_fun.(0)
#=> :zero_or_negative

test_fun.(10)
#=> :exactly_ten
```

There are two important differences between Elixir's `cond` and Erlang's `if`:

1) `cond` allows any expression on the left side while Erlang allows only guard clauses;

2) `cond` uses Elixir's concepts of truthy and falsy values (everything is truthy except `nil` and `false`), Erlang's `if` expects strictly a boolean;

Elixir also provides an `if` function that resembles more imperative languages and is useful when you need to check if one clause is true or false:

```elixir
if x > 10 do
  :greater_than_ten
else
  :not_greater_than_ten
end
```

### 发送和接收消息

The syntax for sending and receiving differs only slightly between Erlang and Elixir.

**Erlang**

```erlang
Pid = self().

Pid ! {hello}.

receive
  {hello} -> ok;
  Other -> Other
after
  10 -> timeout
end.
```

**Elixir**

```elixir
pid = Kernel.self

send pid, {:hello}

receive do
  {:hello} -> :ok
  other -> other
after
  10 -> :timeout
end
```


## 把 Elixir 添加到现有的 Erlang 程序中

Elixir compiles into BEAM byte code (via Erlang Abstract Format). This means that Elixir code can be called from Erlang and vice versa, without the need to write any bindings. All Elixir modules start with the `Elixir.` prefix followed by the regular Elixir name. For example, here is how to use the utf-8 aware `String` downcase from Elixir in Erlang:

```erlang
-module(bstring).
-export([downcase/1]).

downcase(Bin) ->
  'Elixir.String':downcase(Bin).
```

### Rebar 集成

If you are using rebar, you should be able to include Elixir git repository as a dependency:

    https://github.com/elixir-lang/elixir.git

Elixir is structured similar to Erlang's OTP. It is divided into applications that are placed inside the `lib` directory, as seen in its [source code repository](https://github.com/elixir-lang/elixir). Since rebar does not recognize such structure, we need to explicitly add to our `rebar.config` which Elixir apps we want to use, for example:

```erlang
{lib_dirs, [
  "deps/elixir/lib"
]}.
```

This should be enough to invoke Elixir functions straight from your Erlang code. If you are also going to write Elixir code, you can [install Elixir's rebar plugin for automatic compilation](https://github.com/yrashk/rebar_elixir_plugin).

### 手动集成

If you are not using rebar, the easiest approach to use Elixir in your existing Erlang software is to install Elixir using one of the different ways specified in the [Getting Started guide](/getting-started/introduction.html) and add the `lib` directory in your checkout to `ERL_LIBS`.


## 深入阅读

Erlang 的官方文档网站有一批优秀的编程实例。把它们翻译成 Elixir 将会是一个很好的练习。 [Erlang cookbook][5] 提供了更多有用的代码范例。

Elixir 还提供了一份 [入门教程][6] 和 [在线文档][7]。

[4]: http://www.erlang.org/doc/programming_examples/users_guide.html
[5]: http://schemecookbook.org/Erlang/TOC
[6]: /getting-started/introduction.html
[7]: /docs.html
