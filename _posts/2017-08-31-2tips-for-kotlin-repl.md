---
layout:     post
title:      "关于 Kotlin REPL 的两条小贴士"
date:       2017-08-31 19:52:38 +0800
categories: kotlin
---
Kotlin 自带了交互式编程命令行，即 REPL（Read-Eval-Print Loop 的简写，直译为 “读取-求值-输出”循环），尤其适合快速实验一些东西。
本文只讲关于 Kotlin REPL 的两条 tips：
  1. 如何运行 REPL；
  2. 如何在 REPL 中查看推断出的类型。
<!--more-->

## 如何运行 Kotlin REPL

运行 Kotlin REPL 主要有两种方式：
  1. 在 IntelliJ IDEA 中运行；
  2. 运行独立的命令行。

### 直接在 IntelliJ IDEA 中运行 REPL

较新版本的 IntelliJ IDEA（以下简称 IDEA）中已经内置了 Kotlin 支持，包括 Kotlin REPL。如果已经安装了 2017 年版的 IDEA，就可以直接在其中运行 Kotlin REPL。

首先需要在 IDEA 中打开/创建一个 Kotlin 或者 Java 项目，待项目加载完毕之后，点击如下图所示的菜单：Tools -> Kotlin -> Kotlin REPL：

![](/assets/kotlin/repl_idea1.png)

就会出现 Kotlin REPL 窗口：

![](/assets/kotlin/repl_idea2.png)

在 IDEA 内置的 Kotlin REPL 窗口中键入的代码，需要按 Ctrl-回车（mac 下为 ⌘↩︎）运行。如果想退出 REPL，点窗口左侧的叉号按钮即可。

IDEA 内置的 REPL 有一些优势，例如像在代码窗口当中一样拥有语法高亮、智能提示、代码补全等，并且能够运行项目中的代码；但是内置的 REPL 也有一些问题，例如，目前版本在 Windows 下汉字输出为乱码等。

### 运行独立的 REPL 命令行

某些情况下，我们并不适合使用 IDEA 内置的 Kotlin REPL，比如在远程 Linux 服务器上，比如需要在 Windows 下输出汉字时，再如不需要运行项目相关代码并希望少占资源时。这些情况下都更适合使用独立的 Kotlin REPL 命令行。

如果本机已安装较新版本的 IDEA，想要运行 Kotlin REPL 就只需找到它然后运行它即可。它位于 IDEA 所安装目录下的 `plugins/Kotlin/kotlinc/bin` 子目录中，一般来说如果已安装 JDK 并已设置好 `JAVA_HOME` 环境变量，只需将上述子目录设置为命令搜索路径即可通过 `kotlinc` 命令来运行 Kotlin REPL。对于 Windows 在安装 JDK 并设置好 `JAVA_HOME` 之后，可以打开安装目录下的相应子目录，然后直接双击 `kotlinc.bat` 来运行 REPL。

> **注：**如果 IDEA 有更新过小版本（比如 2017.1.3）或者单独升级过 Kotlin 插件，那么较新版本的 Kotlin REPL 有可能不是在 IDEA 的安装目录的子目录下，而是在类似 `~/.IdeaIC2017.1/config/plugins/Kotlin/kotlinc/bin` 这样的目录中（Windows 下在 `%USERPROFILE%\.IdeaIC2017.1\config\plugins\Kotlin\kotlinc\bin` 目录中）。

如果本机没有安装 IDEA 或者在远程 Linux 服务器上，还可以安装独立的 Kotlin 编译器。打开 Kotlin 最新稳定版在 GitHub 上的发布页（[https://github.com/JetBrains/kotlin/releases/latest](https://github.com/JetBrains/kotlin/releases/latest)）。下载其中的 kotlin-compiler-*.zip 文件，将其解压到指定的目录，然后可以将其中 `bin` 所在路径加入到系统的搜索路径中。当然这也要求提前安装好 JDK 并配置好 `JAVA_HOME`。然后就可以在命令行通过 `kotlinc` 命令来运行 Kotlin REPL 了（Windows 下还是可以找到对应的 `kotlinc.bat` 双击运行）。

独立运行的 REPL 命令行遵循通用的命令行操作，如 Ctrl-D 退出、Ctrl-R 反向搜索、Ctrl-S 正向搜索等等。

## 如何在 Kotlin REPL 中查看推断出的类型

昨天看了 Benny 新发的文章[《val b = a?: 0，a 是 Double 类型，那 b 是什么类型？》](https://blog.kotliner.cn/2017/08/30/SuperTypes-in-Type-Inferring/)，文中详述了相关现象并分析了原因，是篇深度好文，在此也推荐给大家。

当时就想边看边在 REPL 中做实验，毕竟做实验这种事情最适合 REPL 做不过了。

遗憾的是 Kotlin REPL 并不回显类型。在 Kotlin REPL 中键入 Benny 文中的示例代码：

> ``` kotlin
> var a: Double? = null
> val b = a ?: 0
> ```

并不会有任何回显，如果想看 `b` 的类型，确实可以这样来做：

``` kotlin
>>> b::class
class kotlin.Int
```

这回看到的是 `Int`，但是这是有问题的。
通过 `b::class` 这种方式得到的是 `b` 实际求值结果 `0` 的类型，而不是 Kotlin 针对 `a ?: 0` 这个表达式，在实际求值之前（编译阶段）为 `b` 推断出的类型。
如果想看 Kotlin 求值之前推断出的类型，该怎么办呢？

答案是用 lambda 表达式，实际上我在上篇文章[《Kotlin 版图解 Functor、Applicative 与 Monad》](https://hltj.me/kotlin/2017/08/25/kotlin-functor-applicative-monad-cn.html) 中有提及过，只是不明显：

> ``` kotlin
> > {y: Int -> {x: Int -> x + y}} `($)` Maybe.Just(5)
> Just(value=(kotlin.Int) -> kotlin.Int)
> ```

`value=` 后面的部分就是 Kotlin 对一个 lambda 表达式的输出形式，我们可以看一个更直观的例子：

``` kotlin
>>> val f = { 1 }
>>> f
() -> kotlin.Int
```

`f` 是一个无参且返回值为 `1` 的 lambda 表达式。当在 REPL 中对 `f` 求值时，REPL 中输出了该 lambda 表达式的类型。这个例子还可以进一步简化为：

``` kotlin
>>> {1}
() -> kotlin.Int
```

这样通过 lambda 表达式的返回值类型就能看出 `1` 在 Kotlin 中被推断为 `Int`。如果要在 REPL 中看 Benny 文章标题所说的示例，只需这样就可以了：

``` kotlin
>>> {
... val a: Double? = 2.0
... a ?: 0
... }
() -> kotlin.Any
```

通过 lambda 返回值的类型可以看出，`a ?: 0` 会被推断为 `Any`。这里 `a` 的值是 `null` 还是 `2.0` 并不影响类型推断的结果。
我们还可以再看几个例子来印证 Benny 的文章：

``` kotlin
>>> {if (true) 1 else 2.0}
() -> kotlin.Any
>>> interface I; class A : I; class B : I;
>>> {if (false) A() else B()}
() -> Line_8.I
```

`Line_8.I` 表示第 8 行定义的类型 `I`，也就是说，对于 `Int` 与 `Double` 推断出的公共基类是 `Any`，而对于 `A` 与 `B` 推断出的公共基类是 `I`，完全印证了 Benny 文中所讲的内容。

