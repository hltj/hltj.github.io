---
layout:     post
title:      "【译】当 Kotlin 遇见 Gradle"
date:       2017-06-13 19:38:01 +0800
categories: kotlin
---
> 本文是 Gradle 官网文章 [Kotlin Meets Gradle](https://blog.gradle.org/kotlin-meets-gradle) 的译文。
原文发布于 2016-05-18，所以文中的时间都要再往前算一年。
如今 Kotlin 已经发布了 1.1，Gradle 已经发布了 3.5，Gradle Script Kotlin 也已经发布了 0.9.1，相对当时都更完善了很多。<!--more-->

![gradle_kotlin.jpg](/assets/kotlin/gradle_kotlin.jpg)

很多读者会对 JetBrains 的优秀编程语言 [Kotlin](http://kotlinlang.org/) 比较熟悉。
自 2010 年以来一直在开发中，在 2012 首次公开发布，并在今年年初发布了 1.0 GA。

这些年来我们一直在关注 Kotlin，并且对语言所提供的功能及其强大的吸引力（尤其对安卓社区）的印象愈加深刻。

去年年底，Hans 与 JetBrains 团队的几个成员一起坐下来想：一个基于 Kotlin 来写 Gradle 构建脚本及插件的方式可能会是什么样的？
它对团队的帮助如何——尤其是大型团队——加快工作速度并编写结构更好、更易于维护的构建？

这些可能性非常诱人。

因为 Kotlin 是一种静态类型语言，在 IDEA 和 Eclipse 中都有深入的支持，所以可以从自动补全到重构以及其间的一切都能为 Gradle 用户提供适当的 IDE 支持。
而且由于 Kotlin 具有丰富的功能，如一等函数和扩展方法，因此它可以保留和改进 Gradle 构建脚本的最佳部分——包括简明的声明式语法以及轻松制作 DSL 的能力。

所以我们认真地考察了这些可能性，在过去的几个月里，我们很高兴与 Kotlin 团队密切合作，为 Gradle 开发一种新的基于 Kotlin 的构建语言。

我们称之为 _Gradle Script Kotlin_，并且在旧金山的 JetBrains 的 [Kotlin 之夜](http://info.jetbrains.com/Kotlin-Night-2016.html)活动中，[Hans 刚刚在舞台上发布了第一个演示版](https://youtu.be/4gmanjWNZ8E)。
我们今天发布了这个作品向 1.0 发展的[第一个预览版本](https://github.com/gradle/gradle-script-kotlin/releases/tag/v0.1.0)，并在 [github.com/gradle/gradle-script-kotlin](https://github.com/gradle/gradle-script-kotlin) 上开源了它的版本库。

那么它是什么样的，而你能用它做什么呢？乍一看，它看起来与你今天所知道的 Gradle 构建脚本并没有 _多大_ 不同。

![build.gradle.kts](https://blog.gradle.org/images/kotlin-hello-world.png)

但是，当你在 IDE 中开始探索各种可能性时，事情会变得非常有趣。
你会发现，突然间你以往期望的东西在 IDE 中 _可用了_，包括：

*   自动补全和内容辅助
*   快速文档
*   跳转到源代码
*   重构等等

效果是戏剧性的，并且我们认为这会对 Gradle 用户产生很大的影响。
关于这点，现在你可能对几件事情有疑问——如现有的 Gradle 插件是否可以与 Gradle Script Kotlin 一起使用（是的，可以），以及是否已经弃用了 Groovy 编写构建脚本（不，并不是）。
你可以在[项目常见问题](https://github.com/gradle/gradle-script-kotlin/wiki/Frequently-Asked-Questions)中找到这些问题以及其他问题的完整答案。
如果你有一个没有答案的问题，请告诉我们。

当然，这还只是开始。
我们很高兴地宣布，会在 Gradle 3.0 中提供 Kotlin 脚本支持，我们也会很快发布关于我们路线图的更多信息。
在此期间，也无需等待——你现在可以通过[我们的入门样例](https://github.com/gradle/gradle-script-kotlin/tree/master/samples#readme)来亲自试用 Gradle Script Kotlin。

并且我们希望你这么做，因为我们乐于收到你的反馈。
我们很乐于听到你的想法，以及你有多期待这项新工作的进展。
你可以通过项目的 [GitHub Issues](https://github.com/gradle/gradle-script-kotlin/issues) 提出问题，并请在公共的 [Kotlin Slack](http://kotlinslackin.herokuapp.com/) 的 #gradle 频道中与我们聊天。

我非常感谢我的同事 Rodrigo B.de Oliveira 最近几个月在这个项目上合作——有很多乐趣！
还要非常感谢 Kotlin 团队，尤其是 Ilya Chernikov 和 Ilya Ryzhenkov，积极响应我们对 Kotlin 编译器以及 Kotlin IDEA 插件的每个需求。
继续加油！
