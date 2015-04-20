---
layout: getting-started
title: 接下来
redirect_from: /getting_started/21.html
---

# {{ page.title }}

{% include toc.html %}

渴望了解更多？请继续阅读！

## 创建你的第一个 Elixir 项目

为了开始你的第一个项目， Elixir 提供了一个构建工具 Mix 。你可以通过简单地执行下面的命令开始一个新的项目：

    mix new path/to/new/project

我们已经编写了一个指南，涵盖了如何构建一个 Elixir 应用，包含自己的监控树，配置，测试等等。 应用是一个以分布式 key-value 存储，我们将 key-value 对组织在 bucket 中，这些 buckets 的分布跨越了多个节点：

* [Mix 和 OTP](/getting-started/mix-otp/introduction-to-mix.html)

## 元编译

由于 Elixir 对元编程的支持，使得它成为一个可扩展的，可定制的编程语言。 Elixir 中的大部分元编程是通过宏完成的，在一些情况下非常有用，尤其是针对编写领域特定语言（ DSL）。我们写了一个简短的教程，解释宏背后的基本机制，并展示了如何编写宏和使用它们创建领域特定语言：

* [Elixir 中的元编程](/getting-started/meta/quote-and-unquote.html)

## 社区和其它资源

我们有一篇 [学习资源](/learning.html) 介绍了关于学习 Elixir 的书，教学视频和其它资源，介绍了它的生态系统。此外，还有大量的 Elixir 资源，像是会议演讲，开源项目，以及社区制作的其它学习材料。

记住假如有任何困难，你都可以访问 **irc.freenode.net** 上的 **#elixir-lang** 频道或者发送一封邮件到 [邮件列表](http://groups.google.com/group/elixir-lang-talk) 。你可以放心肯定会有人愿意提供帮助。要及时地了解最新的新闻和公告，请关注 [官方博客](http://elixir-lang.org/blog/) 和 [elixir-core 邮件列表] 上的语言发展。

别忘了你也可以直接查看 [Elixir 源代码](https://github.com/elixir-lang/elixir) ，大部分源代码都是用 Elixir （主要是在 `lib` 目录下） 编写的，或者访问 [Elixir 文档](/docs.html) 。

## Erlang 简明教程

Elixir 运行在 Erlang 虚拟机上，而且 Elixir 开发都迟早都会与 Elixir 现有的库对接。下面是一份涵盖了 Erlang 基础和高级特性的在线资源列表：

* 这篇 [Erlang 语法：速成课](/crash-course.html) 提供了一份 Erlang 语法的简明介绍。每一份代码片断都有一分相应的 Elixir 代码对照。 这对于你来说是一个学习机会，不仅可以接触到 Erlang 的语法，还能够复习一些你已经在本教程中学到的东西。

* Erlang 的官方网站上有一份带有图片的 [教程](http://www.erlang.org/course/concurrent_programming.html) ，它简单描述了 Erlang 中的并发编程原语。

* [Learn You Some Erlang for Great Good!](http://learnyousomeerlang.com/) 《学些 Erlang 挺好！》 一书对 Erlang 作了非常棒的介绍， 包括它的设计原则、标准库、最佳实践以及更多。 一旦你阅读过上面提到的速成课程，你可以放心地跳过这本书前面有关语法的几章。 当你读到 [The Hitchhiker's Guide to Concurrency](http://learnyousomeerlang.com/the-hitchhikers-guide-to-concurrency) 《并发漫游指南》这一章的时候，真正乐趣才开始。
