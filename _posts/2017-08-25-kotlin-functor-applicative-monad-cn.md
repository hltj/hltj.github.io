---
layout:     post
title:      "Kotlin 版图解 Functor、Applicative 与 Monad"
date:       2017-08-25 12:49:15 +0800
categories: kotlin
---
> 本文是从 Haskell 版 [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) 翻译而来的 Kotlin 版。
> 我同时翻译了中英文两个版本，英文版在[这里](/kotlin/2017/08/25/kotlin-functor-applicative-monad.html)。
>
> 与[从 Swift 版翻译而来的 Kotlin 版](https://medium.com/@aballano/kotlin-functors-applicatives-and-monads-in-pictures-part-1-3-c47a1b1ce251)不同的是，本文是直接从 [Haskell 版原文](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html)翻译而来的。<!--more-->

这是一个简单的值：

![](/assets/fam/value.png)

我们也知道如何将一个函数应用到这个值上：

![](/assets/fam/value_apply.png)

这很简单。
那么扩展一下，我们说任何值都可以放到一个上下文中。
现在你可以把上下文想象为一个可以在其中装进值的盒子：

![](/assets/fam/value_and_context.png)

现在，将一个函数应用到这个值上时，会**根据上下文的不同**而得到不同的结果。
这就是 Functor、 Applicative、 Monad、 Arrow 等概念的基础。
``Maybe`` 数据类型定义了两种相关上下文：

![](/assets/fam/context.png)

``` kotlin
sealed class Maybe<out T> {
    object `Nothing#` : Maybe<Nothing>() {
        override fun toString(): String = "Nothing#"
    }
    data class Just<out T>(val value: T) : Maybe<T>()
}
```

很快我们就会看到将函数应用到 `Just<T>`上 还是应用到 `Nothing#` 上会有多么不同。
首先我们来说说 Functor 吧！

> **注：** 这里用 `Nothing#` 取代原文的 `Nothing`，因为在 Kotlin 中 `Nothing` 是一个特殊类型，参见 [Nothing 类型](https://www.kotlincn.net/docs/reference/exceptions.html#nothing-类型)。
> 另外 Kotlin 有自己的表达可选值的方式，并非使用 `Maybe` 类型这种方式，参见[空安全](https://www.kotlincn.net/docs/reference/null-safety.html)。

## Functor

当一个值被包装在上下文中时，你无法将一个普通函数应用给它：

![](/assets/fam/no_fmap_ouch.png)

这就轮到 `fmap` 出场了。
`fmap` 翩翩而来，从容应对上下文。
`fmap` 知道如何将函数应用到包装在上下文中的值上。
例如，你想将 `{ it + 3 }` 应用到 `Just(2)` 上。
使用 `fmap` 如下：

``` kotlin
> Maybe.Just(2).fmap { it + 3 }
Just(value=5)
```

![](/assets/fam/fmap_apply.png)

**嘭！**
`fmap` 向我们展示了它的成果。
但是 `fmap` 怎么知道如何应用该函数的呢？

## 究竟什么是 Functor 呢？

在 Haskell 中 `Functor` 是一个[类型类](https://learnyoua.haskell.sg/content/zh-cn/ch03/type-and-typeclass.html#typeclasses入门)。
其定义如下：

![](/assets/fam/functor_def.png)

在 Kotlin 中，可以认为 `Functor` 是一种定义了 `fmap` 方法/扩展函数的类型。
以下是 `fmap` 的工作原理：

![](/assets/fam/fmap_def.png)

所以我们可以这么做：

``` kotlin
> Maybe.Just(2).fmap { it + 3 }
Just(value=5)
```

而 `fmap` 神奇地应用了这个函数，因为 `Maybe` 是一个 Functor。
它指定了 `fmap` 如何应用到 `Just` 上与 `Nothing#` 上：

``` kotlin
fun <T, R> Maybe<T>.fmap(transform: (T) -> R): Maybe<R> = when(this) {
    Maybe.`Nothing#` -> Maybe.`Nothing#`
    is Maybe.Just -> Maybe.Just(transform(this.value))
}
```

当我们写 `Maybe.Just(2).fmap { it + 3 }` 时，这是幕后发生的事情：

![](/assets/fam/fmap_just.png)

那么然后，就像这样，`fmap`，请将 `{ it + 3 }` 应用到 `Nothing#` 上如何？

![](/assets/fam/fmap_nothing.png)

``` kotlin
> Maybe.`Nothing#`.fmap { x: Int -> x + 3 }
Nothing#
```

> **注：** 这里该 lambda 表达式的参数必须显式标注类型，因为 Kotlin 中有很多类型可以与整数（`Int`）相加。

![Bill O'Reilly 可是完全不了解 Maybe functor 哦](/assets/fam/bill.png)

就像《黑客帝国》中的 Morpheus，`fmap` 知道都要做什么；如果你从 `Nothing#` 开始，那么你会以 `Nothing#` 结束！
`fmap` 是禅道。
现在它告诉我们了 `Maybe` 数据类型存在的意义。
例如，这是在一个没有 `Maybe` 的语言中处理一个数据库记录的方式：

``` ruby
post = Post.find_by_id(1)
if post
    return post.title
else
    return nil
end
```

而在 Kotlin 中：

``` kotlin
findPost(1).fmap(::getPostTitle)
```

如果 `findPost` 返回一篇文章，我们就会通过 `getPostTitle` 获取其标题。
如果它返回 `Nothing#`，我们就也返回 `Nothing#`！
非常简洁，不是吗？

我们还可以为 `fmap` 定义一个中缀操作符 `($)`（在 Haskell 中是 `<$>`），并且这样更常见：

``` kotlin
infix fun <T, R> ((T) -> R).`($)`(maybe: Maybe<T>) = maybe.fmap(this)

::getPostTitle `($)` findPost(1)
```

再看一个示例：如果将一个函数应用到一个 `Iterable`（Haksell 中是 `List`）上会发生什么？

![](/assets/fam/fmap_list.png)

`Iterable` 也是 functor！
我们可以为其定义 `fmap` 如下：

``` kotlin
fun <T, R> Iterable<T>.fmap(transform: (T) -> R): List<R> = this.map(transform)
```

好了，好了，最后一个示例：如果将一个函数应用到另一个函数上会发生什么？

``` kotlin
{ x: Int - > x + 1 }.fmap { x: Int -> x + 3 }
```

这是一个函数：

![](/assets/fam/function_with_value.png)

这是一个应用到另一个函数上的函数：

![](/assets/fam/fmap_function.png)

其结果是又一个函数！

``` kotlin
> fun <T, U, R> ((T) -> U).fmap(transform: (U) -> R) = { t: T -> transform(this(t)) }
> val foo = { x: Int -> x + 2 }.fmap { x: Int -> x + 3 }
> foo(10)
15
```

所以函数也是 functor！
对一个函数使用 fmap，其实就是函数组合！

## Applicative

Applicative 又提升了一个层次。
对于 Applicative，我们的值像 Functor 一样包装在一个上下文中：

![](/assets/fam/value_and_context.png)

但是我们的函数也包装在一个上下文中！

![](/assets/fam/function_and_context.png)

嗯。
我们继续深入。
Applicative 并没有开玩笑。
Applicative 定义了 `(*)`（在 Haskell 中是 `<*>`），它知道如何将一个 _包装在上下文中的_ 函数应用到一个 _包装在上下文中的_ 值上：

![](/assets/fam/applicative_just.png)

即：

``` kotlin
infix fun <T, R>  Maybe<(T) -> R>.`(*)`(maybe: Maybe<T>): Maybe<R> = when(this) {
    Maybe.`Nothing#` -> Maybe.`Nothing#`
    is Maybe.Just -> this.value `($)` maybe
}

Maybe.Just { x: Int -> x + 3 } `(*)` Maybe.Just(2) == Maybe.Just(5)
```

使用 `(*)` 可能会带来很多有趣的情况。
例如：

``` kotlin
infix fun <T, R>  Iterable<(T) -> R>.`(*)`(iterable: Iterable<T>) = this.flatMap { iterable.map(it) }
```

有了这个定义，我们可以将一个函数列表应用到一个值列表上：

``` kotlin
> listOf<(Int) -> Int>({ it * 2 }, { it + 3 }) `(*)` listOf(1, 2, 3)
[2, 4, 6, 4, 5, 6]
```

![](/assets/fam/applicative_list.png)

这里有 Applicative 能做到而 Functor 不能做到的事情。
如何将一个接受两个参数的函数应用到两个已包装的值上？

``` kotlin
> { y: Int -> { x: Int -> x + y } } `($)` Maybe.Just(5)
Just(value=(kotlin.Int) -> kotlin.Int) // 等于 `Maybe.Just { x: Int -> x + 5 }`
> Maybe.Just { x: Int -> x + 5 } `($)` Maybe.Just(4)
错误 ？？？ 这究竟是什么意思，这个函数为什么包装在 JUST 中？
```

Applicative：

``` kotlin
> { y: Int -> { x: Int -> x + y } } `($)` Maybe.Just(5)
Just(value=(kotlin.Int) -> kotlin.Int) // 等于 `Maybe.Just { x: Int -> x + 5 }`
> Maybe.Just { x: Int -> x + 5 } `(*)` Maybe.Just(3)
Just(value=8)
```

`Applicative` 把 `Functor` 推到一边。
“大人物可以使用具有任意数量参数的函数，”它说。
“装备了 `($)` 与 `(*)` 之后，我可以接受具有任意个数未包装值参数的任意函数。
然后我传给它所有已包装的值，而我会得到一个已包装的值出来！
啊啊啊啊啊！”

``` kotlin
> { y: Int -> { x: Int -> x + y } } `($)` Maybe.Just(5) `(*)` Maybe.Just(3)
Just(value=15)
```

我们也可以定义另一个 Applicative 的函数 `liftA2`：

``` kotlin
fun <T> ((x: T, y: T) -> T).liftA2(m1: Maybe<T>, m2: Maybe<T>) =
    { y: T -> { x: T -> this(x, y) } } `($)` m1 `(*)` m2
```

并使用 `liftA2` 做同样事情：

``` kotlin
> { x: Int, y: Int -> x * y }.liftA2(Maybe.Just(5), Maybe.Just(3))
Just(value=15)
```

## Monad

如何学习 Monad 呢：
  1. 取得计算机科学博士学位。
  2. 然后把它扔掉，因为在本节中你并不需要！

Monad 增加了一个新的转变。

Functor 将一个函数应用到一个已包装的值上：

![](/assets/fam/fmap.png)

Applicative 将一个已包装的函数应用到一个已包装的值上：

![](/assets/fam/applicative.png)

Monad 将一个**返回已包装值**的函数应用到一个已包装的值上。
Monad 有一个函数 `))=`（在 Haskell 中是 `>>=`，读作“绑定”）来做这个。

让我们来看个示例。
老搭档 `Maybe` 是一个 monad：

![正是闲逛的 monad](/assets/fam/context.png)

假设 `half` 是一个只适用于偶数的函数：

``` kotlin
fun half(x: Int) = if (x % 2 == 0)
                       Maybe.Just(x / 2)
                   else
                       Maybe.`Nothing#`
```

![](/assets/fam/half.png)

如果我们喂给它一个已包装的值呢？

![](/assets/fam/half_ouch.png)

我们需要使用 `))=` 来将我们已包装的值塞进该函数。
这是 `))=` 的照片：

![](/assets/fam/plunger.jpg)

以下是它的工作方式：

``` kotlin
> Maybe.Just(3) `))=` ::half
Nothing#
> Maybe.Just(4) `))=` ::half
Just(value=2)
> Maybe.`Nothing#` `))=` ::half
Nothing#
```

内部发生了什么？
`Monad` 是 Haskell 中的另一个类型类。
这是它（在 Haskell 中）的定义的片段：

``` haskell
class Monad m where
    (>>=) :: m a -> (a -> m b) -> m b
```

其中 `>>=` 是：

![](/assets/fam/bind_def.png)

在 Kotlin 中，可以认为 `Monad` 是一种定义了这样中缀函数的类型：

```kotlin
infix fun <T, R> Monad<T>.`))=`(f: ((T) -> Monad<R>)): Monad<R>
```

所以 `Maybe` 是一个 Monad：

``` kotlin
infix fun <T, R> Maybe<T>.`))=`(f: ((T) -> Maybe<R>)): Maybe<R> = when(this) {
    Maybe.`Nothing#` -> Maybe.`Nothing#`
    is Maybe.Just -> f(this.value)
}
```

这是与 `Just(3)` 互动的情况！

![](/assets/fam/monad_just.png)

如果传入一个 `Nothing#` 就更简单了：

![](/assets/fam/monad_nothing.png)

你还可以将这些调用串联起来：

``` kotlin
> Maybe.Just(20) `))=` ::half `))=` ::half `))=` ::half
Nothing#
```

![](/assets/fam/monad_chain.png)
![](/assets/fam/whoa.png)

> **注：** Kotlin 内置的空安全语法可以提供类似 monad 的操作，包括链式调用：
> ``` kotlin
> fun Int.half() = if (this % 2 == 0) this / 2 else null
> 
> val n: Int? = 20
> n?.half()?.half()?.half()
> ```

太酷了！
于是现在我们知道 `Maybe` 既是 `Functor` 、又是 `Applicative` 还是 `Monad`。

现在我们来看看另一个例子：`IO` monad：

![](/assets/fam/io.png)

> **注：** 由于 Kotlin 并不区分纯函数与非纯函数，因此根本不需要 IO monad。
> 这只是一个模拟：
> ``` kotlin
> data class IO<out T>(val `(-`: T)
> 
> infix fun <T, R> IO<T>.`))=`(f: ((T) -> IO<R>)): IO<R> = f(this.`(-`)
> ```

具体来看三个函数。
`getLine` 没有参数并会获取用户输入：

![](/assets/fam/getLine.png)

``` kotlin
fun getLine(): IO<String> = IO(readLine() ?: "")
```

`readFile` 接受一个字符串（文件名）并返回该文件的内容：

![](/assets/fam/readFile.png)

``` kotlin
typealias FilePath = String

fun readFile(filename: FilePath): IO<String> = IO(File(filename).readText())
```

`putStrLn` 接受一个字符串并输出之：

![](/assets/fam/putStrLn.png)

``` kotlin
fun putStrLn(str: String): IO<Unit> = IO(println(str))
```

所有这三个函数都接受普通值（或无值）并返回一个已包装的值。
我们可以使用 `))=` 将它们串联起来！

![](/assets/fam/monad_io.png)

``` kotlin
getLine() `))=` ::readFile `))=` ::putStrLn
```

太棒了！
前排占座来看 monad 展示！
Haskell 还为我们提供了名为 `do` 表示法的语法糖：

``` haskell
foo = do
    filename <- getLine
    contents <- readFile filename
    putStrLn contents
```

它可以在 Kotlin 中模拟（其中 Haskell 的 `<-` 操作符被替换为 `(-` 属性与赋值操作）如下：

``` kotlin
fun <T> `do` (ioOperations: () -> IO<T>) = ioOperations()

val foo = `do` {
  val filename = getLine().`(-`
  val contents = readFile(filename).`(-`
  putStrLn(contents)
}
```

## 结论
  1. （Haskell 中的）functor 是实现了 `Functor` 类型类的数据类型。
  2. （Haskell 中的）applicative 是实现了 `Applicative` 类型类的数据类型。
  3. （Haskell 中的）monad 是实现了 `Monad` 类型类的数据类型。
  4. `Maybe` 实现了这三者，所以它是 functor、 applicative、 _以及_ monad。

这三者有什么区别呢？

![](/assets/fam/recap.png)

  * **functor:** 可通过 `fmap` 或者 `($)` 将一个函数应用到一个已包装的值上。
  * **applicative:** 可通过 `(*)` 或者 `liftA` 将一个已包装的函数应用到已包装的值上。
  * **monad:** 可通过 `))=` 或者 `liftM` 将一个返回已包装值的函数应用到已包装的值上。

所以，亲爱的朋友（我觉得我们现在是朋友了），我想我们都同意 monad 是一个简单且高明的主意（译注：原文是 SMART IDEA(tm)）。
现在你已经通过这篇指南润湿了你的口哨，为什么不拉上 Mel Gibson 并抓住整个瓶子呢。
请参阅《Haskell 趣学指南》的[《来看看几种 Monad》](https://learnyoua.haskell.sg/content/zh-cn/ch12/a-fistful-of-monads.html)。
其中包含很多我已经炫耀过的东西，因为 Miran 深入这些方面做的非常棒。

> **译注：**Miran 即 Miran Lipovača 是《Haskell 趣学指南》英文原版 [Learn You a Haskell](http://learnyouahaskell.com/) 的作者。

在此向 [Functors, Applicatives, And Monads In Pictures](http://adit.io/posts/2013-04-17-functors,_applicatives,_and_monads_in_pictures.html) 原作者 [Aditya Bhargava](http://adit.io/) 致谢，
向 [Learn You a Haskell](http://learnyouahaskell.com/) 作者 Miran Lipovača 以及 [MnO2](http://blog.mno2.org/)、[Fleurer](http://fleurer-lee.com/) 等[《Haskell 趣学指南》](https://learnyoua.haskell.sg/content/)中文版译者致谢。
