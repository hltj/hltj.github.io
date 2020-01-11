---
layout:     post
title:      "在 Kotlin 中“实现”trait/类型类"
date:       2020-01-11 19:31:21 +0800
categories: kotlin
---
## trait 与类型类都是什么
**trait** 与**类型类（type class）**分别是 Rust 与 Haskell 语言中的概念，用于[特设多态（ad-hoc polymorphism）](https://zh.wikipedia.org/wiki/%E7%89%B9%E8%AE%BE%E5%A4%9A%E6%80%81)、[函数式编程](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BC%96%E7%A8%8B)等方面。

<!--more-->

值得一提的是虽然英文都是“trait”， Scala 的特质跟 Rust 的 trait [^1] 却并不相同。
Scala 的特质相当于 Kotlin 与 Java 8+ 的接口，能实现**子类型多态**；而 Rust 的 trait 更类似于 Swift 的协议与 Haskell 的类型类，能实现**特设多态**。简单来说，**trait** 应同时具备以下三项能力[^2]：
  1. 定义“接口”并可提供默认实现
  2. 用作泛型约束
  3. 给既有类型增加功能

Haskell 的**类型类**不仅同时具备这三项能力，还能定义函数式编程中非常重要的 [Functor、Applicative、Monad 等](https://hltj.me/kotlin/2017/08/25/kotlin-functor-applicative-monad-cn.html)。
当然这是废话，因为它们在 Haskell 中本来就是类型类😂。
实际上这也不是 trait 与类型类的差异，能否支持 Functor 等的关键在于语言的泛型参数能否支持[类型构造器](https://en.wikipedia.org/wiki/Kind_(type_theory))（或者说语言能否支持高阶类型）。

[^1]: trait：Scala 中文社区倾向于译为“特质”，Rust 中文社区倾向于不译。

[^2]: 按说只需后两项能力即可实现 Rust/Haskell 式的特设多态，但没有第一项能力其易用性与表现力都要打折扣。

## 在 Kotlin 中寻求对应
在 Kotlin 中并没有同时具备这三项能力的对应，只有分别提供三项能力的特性。
其中 Kotlin 的[接口](https://www.kotlincn.net/docs/reference/interfaces.html)同时具备前两项能力。

### 定义“接口”并可提供默认实现
例如，定义一个带有默认实现的接口：

``` kotlin
interface WithDescription {
    val description: String get() = "The description of $this"
}
```

Kotlin 的接口中可以定义属性与方法，二者都可以有默认实现，简便起见，示例中用了具有默认实现的属性。它可以这么用：

``` kotlin
class Foo: WithDescription {
    // Foo 类为 description 属性提供了自己的实现
    override val description = "This is a Foo object"
}

// 对象 Bar 的 description 属性采用默认实现
object Bar: WithDescription

println(Foo().description)
println(Bar.description)
```

在 [Kotlin REPL](https://hltj.me/kotlin/2017/08/31/2tips-for-kotlin-repl.html) 中执行会得到类似这样的输出：

``` txt
This is a Foo object
The description of Line_7$Bar@5bf4764d
```

### 用作泛型约束
接下来还可以将之前定义的 `WithDescription` 接口用在泛型函数、泛型类或者其他泛型接口中作为[泛型约束](https://www.kotlincn.net/docs/reference/generics.html#%E6%B3%9B%E5%9E%8B%E7%BA%A6%E6%9D%9F)，例如：

``` kotlin
fun <T : WithDescription> T.printDescription() = println(description)
```

在 REPL 中执行：

``` kotlin
>>> Bar.printDescription()
The description of Line_7$Bar@5bf4764d
```

遗憾的是，在 Kotlin 中不能给既有类型（类或接口）实现新的接口，比如不能为 `Boolean` 或者 `Iterable` 实现 `WithDescription`。
即接口不具备第三项能力，因此它不是 trait/类型类。

### 给既有类型增加功能
在 Kotlin 中给既有类型增加功能的方式是[扩展](https://www.kotlincn.net/docs/reference/extensions.html)，可以给任何既有类型声明扩展函数与扩展属性。例如可以分别给 `Int` 与 `String` 实现二者间的乘法操作符函数：

``` kotlin
operator fun Int.times(s: String) = s.repeat(this)

operator fun String.times(n: Int) = repeat(n)
```

于是就可以像 Python/Ruby 那样使用了：

``` kotlin
>>> "Hello" * 3
res11: kotlin.String = HelloHelloHello
>>> 5 * "汉字"
res12: kotlin.String = 汉字汉字汉字汉字汉字
```

## 在 Kotlin 中“实现”trait/类型类
如上文所述，Kotlin 分别用接口与扩展两个不同特性提供了 trait/类型类的三项能力，因此在 Kotlin 中没有其直接对应。
那么如果把两个特性以某种方式结合起来，是不是就可以“实现”trait/类型类了？——还别说，真就可以！
[Arrow 中的类型类](https://arrow-kt.io/docs/typeclasses/intro/)就是这么实现的。

我们继续以 `WithDescription` 为例，不同的是，这回要这么声明：

``` kotlin
interface WithDescription<T> {
    val T.description get() = "The description of $this"
}
```

这里利用了[分发接收者可以子类化、扩展接收者静态解析](https://www.kotlincn.net/docs/reference/extensions.html#%E6%89%A9%E5%B1%95%E5%A3%B0%E6%98%8E%E4%B8%BA%E6%88%90%E5%91%98)的特性，可以为任何既有类型添加实现。例如分别为 `Char`、`String` 实现如下：

``` kotlin
object CharWithDescription : WithDescription<Char> {
    override val Char.description get() = "${this.category} $this"
}

// 采用默认实现
object StringWithDescription: WithDescription<String>
```

不过使用时会麻烦一点，需要借助 `run()` 或者 `with()` 这样的[作用域函数](https://www.kotlincn.net/docs/reference/scope-functions.html)在相应上下文中执行：

``` kotlin
println(StringWithDescription.run { "hello".description })

with(CharWithDescription) {
    println('a'.description)
}
```

在 REPL 中执行的输出如下：

``` txt
The description of hello
LOWERCASE_LETTER a
```

用作泛型约束也不成问题：

``` kotlin
fun <T, Ctx : WithDescription<T>> Ctx.printDescription(t: T) = println(t.description)

StringWithDescription.run {
    CharWithDescription.run {
        printDescription("Kotlin")
        printDescription('①')
    }
}
```

这里实现的 `printDescription()` 与上文的函数签名不同，因为接收者类型用于实现基于作用域上下文的泛型约束了，这也是利用接口、扩展、子类型多态以及作用域函数这些特性来“实现”trait/类型类的关键所在。
当然，如果仍然希望目标类型（如例中的 `Char`、`String`）作为 `printDescription` 的接收者，只要将其接收者与参数互换即可：

``` kotlin
fun <T, Ctx : WithDescription<T>> T.printDescription(ctx: Ctx) = ctx.run {
    println(description)
}

"hltj.me".printDescription(StringWithDescription)
```

上述两种方式中提供泛型约束的上下文要么占用了函数的扩展接收者、要么占用了函数参数。实际上还有一种方式——占用分发接收者，显然只要在 `WithDescription` 内声明 `printDescription()` 就可以了。
不过我们这里要假设 `printDescription()` 是自己定义的函数，而 `WithDescription` 是无法修改的既有类型，那么还能做到吗？——当然不成问题！只要用一个新接口继承  `WithDescription` 就可以了：

``` kotlin
interface WithDescriptionAndItsPrinter<T>: WithDescription<T> {
    fun T.printDescription() = println(description)
}

object StringWithDescriptionAndItsPrinter: WithDescriptionAndItsPrinter<String>

object CharWithDescriptionAndItsPrinter:
    WithDescriptionAndItsPrinter<Char>, WithDescription<Char> by CharWithDescription

StringWithDescriptionAndItsPrinter.run {
   CharWithDescriptionAndItsPrinter.run {
        "hltj.me".printDescription()
        '★'.printDescription()
    }
}
```

将三种方式放一起对比会更直观：

``` kotlin
// 方式 1
StringWithDescription.run {
    printDescription("hltj.me")
}

// 方式 2
"hltj.me".printDescription(StringWithDescription)

// 方式 3
interface WithDescriptionAndItsPrinter { /*……*/ }
object StringWithDescriptionAndItsPrinter: WithDescriptionAndItsPrinter<String>
StringWithDescriptionAndItsPrinter.run {
    "hltj.me".printDescription()
}
```

第三种方式的优点是提供泛型约束的上下文既不占用扩展接收者也不占用参数，但其代价是需要为每个用到的目标类型（如例中的 `Char`、`String`）提供新接口（如例中的 `WithDescriptionAndItsPrinter<T>`）的相应实现，并且依然需要借助作用域函数 `run()` 或 `with()`。
因此通常采用前两种方式即可，但是如果要自定义操作符函数或者中缀函数时就只能采用第三种方式了，例如：

``` kotlin
interface DescriptionMultiplier<T>: WithDescription<T> {
    infix fun T.rep(n: Int) = (1..n).joinToString { description }

    operator fun T.times(n: Int) = this rep n
}

object CharDescriptionMultiplier:
    DescriptionMultiplier<Char>, WithDescription<Char> by CharWithDescription

println(object : DescriptionMultiplier<String> {}.run { "hltj.me" rep 2 })

println(CharDescriptionMultiplier.run { 'A' * 3 })
```

在 REPL 中执行的输出为：

``` txt
The description of hltj.me, The description of hltj.me
UPPERCASE_LETTER A, UPPERCASE_LETTER A, UPPERCASE_LETTER A
```

### 扩展与成员的优先级
我们知道，在 Kotlin 中扩展与成员冲突时[总是取成员](https://www.kotlincn.net/docs/reference/extensions.html#%E6%89%A9%E5%B1%95%E6%98%AF%E9%9D%99%E6%80%81%E8%A7%A3%E6%9E%90%E7%9A%84)。
但是在使用基于作用域上下文的泛型约束时却并非如此，例如：

``` kotlin
interface WithLength<T> {
    val T.length: Int
}

object StringWithFakeLength: WithLength<String> {
    override val String.length get() = 128
}

fun <T, U: WithLength<T>> U.printLength(t: T) = println(t.length)

StringWithFakeLength.run {
    printLength("hltj.me")
}
```

在 REPL 中运行输出是 `128`，表明 `printLenth()` 取到的 `length` 是 `StringWithFakeLength` 中定义的扩展属性而不是 `String` 自身的属性。因此使用时需要特别注意。此外对于 Any 的三个成员 `toString()`、`hashCode()`、`equals()` 会始终调用成员函数，即便在泛型约束上下文中声明了具有相同签名的扩展函数也是一样。

###  “实现”Functor 等
按照上文介绍的方式，我们可以轻松实现 `Show`、`Eq`、`Ord` 等简单类型类，无需赘述。
但是如果要实现 `Functor`、`Applicative`、`Monad` 等却会遇到问题。
以 `Functor` 为例，按说要这么定义：

``` kotlin
interface Functor<C<*>> {
    fun <T, R> C<T>.fmap(f: (T) -> R): C<T>
}
```

但遗憾的是上述代码无法通过编译，因为 Kotlin 目前不支持[高阶类型](https://en.wikipedia.org/wiki/Kind_(type_theory))，在泛型参数中用 `C<*>` 表示类型构造器只是**假想**的语法 。
因此，需要有一种方式来变通。按 [Arrow 的方式](https://arrow-kt.io/docs/patterns/glossary/#higher-kinds) 引入 `Kind` 接口来表示：

``` kotlin
interface Kind<out F, out A>

interface Functor<F> {
    fun <T, R> Kind<F, T>.fmap(f: (T) -> R): Kind<F, R>
}
```

然后写一个标记类，让具体类型作为 `Kind<标记类, T>` 的实现类。再定义一个由 `Kind<标记类, T>` 向具体类型转换的扩展函数 `fix()`，以便在具体实现中使用。
例如：

``` kotlin
class ForMaybe private constructor()

sealed class Maybe<out T> : Kind<ForMaybe, T> {
    object `Nothing#` : Maybe<Nothing>() {
        override fun toString(): String = "Nothing#"
    }
    data class Just<out T>(val value: T) : Maybe<T>()
}

fun <T> Kind<ForMaybe, T>.fix(): Maybe<T> = this as Maybe<T>
```

这样就可以为 `Maybe` 实现 `Functor<ForMaybe>` 了：

``` kotlin
object MaybeFunctor : Functor<ForMaybe> {
    override fun <T, R> Kind<ForMaybe, T>.fmap(f: (T) -> R): Maybe<R> = when (val maybe = fix()) {
        is Maybe.Just -> Maybe.Just(f(maybe.value))
        else -> Maybe.`Nothing#`
    }
}

fun main() = with(MaybeFunctor) {
    println(Maybe.Just(5).fmap { it + 1 })
    println(Maybe.`Nothing#`.fmap { x: Int -> x + 1 })
}
```

可以看出这种实现方式会有明显的局限性：只能为 Arrow 中定义的类型或者按照 Arrow 方式实现的既有类型实现 `Functor`、`Applicative`、`Monad` 等接受类型构造器作为泛型参数的“类型类”。
好在 Arrow 已经自带了大量有用的类型，很多场景都够用。

> 需要注意的是这段代码无法在当前版本（1.3.61）的 Kotlin REPL 中运行，需要放在普通的 Kotlin 文件中编译运行。

## Arrow
[Arrow](https://arrow-kt.io/)（按其官网写作 Λrrow）是 Kotlin 标准库的函数式“伴侣”。目前主要以下四套件：

- [Arrow Core](https://arrow-kt.io/docs/core/) 提供了核心的数据类型与类型类。
- [Arrow FX](https://arrow-kt.io/docs/fx/) 是函数式副作用库，提供了 `do`-表示法/[Monad 推导](https://arrow-kt.io/docs/patterns/monad_comprehensions/#comprehensions-over-coroutines)风格的 DSL。
- [Arrow Optics](https://arrow-kt.io/docs/optics/dsl/) 用于在 Kotlin 中处理不可变数据模型。
- [Arrow Meta](https://meta.arrow-kt.io/) 是 Kotlin 编译器与 IDE 的函数式“伴侣”。

此外还有若干套件/特性还在孵化中。
关于 Arrow 整体与模式的介绍也在 Arrow Core 的文档中，其中 [Functional Programming Glossary](https://arrow-kt.io/docs/patterns/glossary/) 提供了一些使用 Arrow 进行函数式编程的背景知识可供参考。

## 一点意外
在尝试写这些示例时意外发现了一个会导致当前版本的 Kotlin JVM 编译器抛异常的 bug，最小重现代码如下：


``` kotlin
interface WithIntId<T> {
    val T.intId get() = 1
}

object BooleanWithIntId : WithIntId<Boolean>

val x = BooleanWithIntId.run {
    true.intId
}
```

只影响 Kotlin JVM 编译器，Kotlin JS 与 Kotlin Native 都不存在这个问题。
查了下  YouTrack，看起来是个[已知 bug](https://youtrack.jetbrains.com/issue/KT-29331#focus=streamItem-27-3894798.0-0)。
不过文中的其他示例代码都能正常编译运行，尽可放心。

---
