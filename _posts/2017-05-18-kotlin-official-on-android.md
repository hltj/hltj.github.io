---
layout:     post
title:      "Kotlin 概览——如何看待 Google 将 Kotlin 选为 Android 官方语言？"
date:       2017-05-18 22:09:48 +0800
categories: kotlin
---
![kotlin_android.png](/assets/kotlin/kotlin_android.png)

Kotlin 作为 Android 开发语言是大势所趋。

在此之前很早，Kotlin 就有[“Android 世界的 Swift ”](https://www.infoq.com/cn/news/2015/06/Android-JVM-JetBrains-Kotlin)的称号。当然在这之前大家这样说难免有些底气不足<!--more-->，与其说是一种事实不如说是一种愿望。而现在这么说就理直气壮多了。当然之前就已经有很多地方在实践用 Kotlin 做安卓开发了，比如魅族、腾讯， [Kotlin 中文站](https://www.kotlincn.net/) 创始人， [Kotlin 中文博客](https://www.kotliner.cn/) 维护人分别来自这两家。

Kotlin 语言相对 Java 有很多优势，比如官网介绍的简洁、安全，示例参见 [Kotlin 中文站](https://www.kotlincn.net/) 首页，部分示例解析见下文。Kotlin 具有现代（也有称下一代的）静态编程语言的很多特点，如类型推断、多范式支持、可空性表达、扩展函数、DSL 支持等。另外对于安卓开发还提供了 Kotlin 安卓扩展和 Anko 库，参见 [Kotlin 用于 Android](https://www.kotlincn.net/docs/reference/android-overview.html) 。

关于与 Java 互操作，尤其是 Java 调用 Kotlin 是大家普遍觉得坑的地方，除了默认 final 外，还有一个主要原因应该就是名字修饰，解决方式可以按照它修饰后名字去引用，或者在 Kotlin 端使用 `@JvmName` 注解来生成便于 Java 使用的名字。具体参见 [Java 中调用 Kotlin](https://www.kotlincn.net/docs/reference/java-to-kotlin-interop.html) 。

让我们看下官方给出的一些例子：

**简洁性**

> 使用一行代码创建一个包含 getters、 setters、 `equals()`、 `hashCode()`、 `toString()` 以及 `copy()` 的 POJO：
> ``` kotlin
data class Customer(val name: String, val email: String, val company: String)
```

这个对于 Java 恐怕要写半屏到一屏代码，如果用 Lombok 能好一些，但也不及 Kotlin 简洁。了解更多请参见[数据类](https://www.kotlincn.net/docs/reference/data-classes.html)。另外可以看到一个小细节，Kotlin 可以不用打`;`。

> 
> 或者使用 lambda 表达式来过滤列表：
> ``` kotlin
val positiveNumbers = list.filter { it > 0 }
```

注意到了吗？它用 `it` 来简化单参数的 Lambda 表达式，了解更多请参见 [Lambda 表达式](https://www.kotlincn.net/docs/reference/lambdas.html)。

> 
> 想要单例？创建一个 object 就可以了：
> ``` kotlin
object ThisIsASingleton {
    val companyName: String = "JetBrains"
}
```

简洁之至，无需赘述。了解更多请参见[对象](https://www.kotlincn.net/docs/reference/object-declarations.html)。

**安全性** ——可空性表达 与 类型判断

> 彻底告别那些烦人的 NullPointerException，毕竟[价值万亿](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)。
> ```kotlin
var output: String
output = null   // 编译错误
```

无特殊标志的变量默认不可空。

> Kotlin 可以保护你避免对可空类型的误操作
> ```kotlin
val name: String? = null    // 可控类型
println(name.length())      // 编译错误
```

可空变量的类型需要后缀 `?`，对于可空变量在未判断其可空性时不可直接调用其方法或访问其属性。了解更多请参见[空安全](https://www.kotlincn.net/docs/reference/null-safety.html)。
另外这里用 `val` 声明的变量是不可变的，对于不可变变量有很多好处，比如并发安全、适合函数式编程等等。参见[基础语法](https://www.kotlincn.net/docs/reference/basic-syntax.html)。

> 并且如果你检查类型是正确的，编译器会为你做自动类型转换
> ```kotlin
fun calculateTotal(obj: Any) {
    if (obj is Invoice)
        obj.calculateTotal()
}
```

类型在判断后自动转换为相应对象；另外，对于可空变量，做非空性判断的相应分支也能自动转成非空值。

**DSL**

让我们看一个复杂一点的例子，构造 HTML 的 DSL 代码：

> ```kotlin
val data = mapOf(1 to "one", 2 to "two")
createHTML().table {
    for ((num, string) in data) {    // 遍历数据
        tr {                         // 创建 HTML 标签的函数
            td { +"$num" }
            td { +string }           // 输出变量的值
        }
    }
}
```

这个例子比较复杂，建议对 Kotlin 熟悉一定程度再来看。
最上方声明了一个不可变的 `data` 作为创建 HTML 用的数据。它是一个由 `mapOf` 函数创建的映射（map），其键值对简单明了。
接下来的 `table` 是一个方法（或[扩展函数](https://www.kotlincn.net/docs/reference/extensions.html)），
它接受一个 Lambda 表达式，对于这种情况的函数调用[可省略括号](https://www.kotlincn.net/docs/reference/lambdas.html)。之后 for 循环中的 `tr` 和 `td` 与 `table` 类似，
只是它们有[隐式接收者](https://www.kotlincn.net/docs/reference/lambdas.html#带接收者的函数字面值)。
此外 for 循环的条件用到了[解构声明](https://www.kotlincn.net/docs/reference/multi-declarations.html)，
两个 `td` 传入的 Lambda 表达式都用到了[操作符重载](https://www.kotlincn.net/docs/reference/operator-overloading.html)，
对于第一个还用到了[字符串模版](https://www.kotlincn.net/docs/reference/basic-types.html#字符串模板)。

---
<br>
本文原是我在知乎上的回答，经整理并补充示例解析而来（然后又更新到回答答案中），知乎原文：[如何看待 Google 将 Kotlin 选为 Android 官方语言？](https://www.zhihu.com/question/59984246/answer/171175189)。

我是 [Kotlin 中文站](https://www.kotlincn.net/)维护人，中文站就是 Kotlin 官方英文站的中文翻译，目前已经完成参考文档的翻译：[参考 - Kotlin 语言中文站](https://www.kotlincn.net/docs/reference/) 。
这应该也是目前唯一一份最新且完整的官方参考文档中文翻译，参见我上一篇[Kotlin 官方参考文档翻译完毕](https://hltj.me/kotlin/2017/05/15/kotlin-reference-translated.html)。欢迎大家反馈问题及一同翻译。

另外欢迎关注 [Kotlin中文博客](https://www.kotliner.cn/) 并积极投稿。

如果好玩的也可以投稿给[千里冰封被吊打的日常 - 知乎专栏](https://zhuanlan.zhihu.com/ice1k)。

