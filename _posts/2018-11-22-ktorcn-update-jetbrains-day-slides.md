---
layout:     post
title:      "庆祝 Ktor 1.0 发布，分享 JetBrains 日讲稿及代码"
date:       2018-11-22 23:58:29 +0800
categories: kotlin
---
非常值得庆祝的是，[Ktor 1.0 正式发布了](https://www.kotliner.cn/2018/11/ktor-1-0/)，[Ktor 中文站](https://ktor.kotlincn.net/)也已更新。

Ktor 是 [JetBrains](https://www.jetbrains.com/) 官方出品的互联应用框架。
使用该框架非常易于开发异步的服务器与客户端，并且能够充分利用 [Kotlin](https://www.kotlincn.net/) 以及[协程](https://www.kotlincn.net/docs/reference/coroutines/coroutines-guide.html)的优势。

<!--more-->

[Ktor 中文站](https://ktor.kotlincn.net/)是[官方英文站](https://ktor.io/)的中文翻译（目前还在翻译中，欢迎组团一起）。
初学者可以从[快速入门](https://ktor.kotlincn.net/quickstart/index.html)入手来学习与了解 Ktor，这一章大多数内容均已翻译。

上周六，有幸在 [JetBrains 开发者日](https://info.jetbrains.com/jetbrains-day-beijing-2018.html)上分享了《Ktor——Kotlin 多平台异步 Web 框架实践》 ，这两天也把讲稿及相关 demo 整理了下。

讲稿可在这里下载：

> 链接： [https://share.weiyun.com/5UqjtTc](https://share.weiyun.com/5UqjtTc)  
> 密码： eauq37
> 
> 我猜你还想看 Benny 分享的讲稿，传送门在这里：[2018 JetBrains 开发者大会见闻](https://www.bennyhuo.com/2018/11/18/2018-JetBrains-Day/)

这份讲稿比当天用的那份要新一些（其中的截图也能看出是 11 月 20 日的），补充了当场提到但没有在讲稿中列出的 Ktor 适用场景：
多平台项目，同时开发客户端与服务端，比如同时开发 WebSocket 或者直接套接字通讯的客户端与服务器。

CallID 与 Call Logging MDC 的 demo 在这里：

[https://github.com/hltj/ktor-callid-demo](https://github.com/hltj/ktor-callid-demo)

接口聚合服务 demo 在这里：

[https://github.com/hltj/kaggregator-demo](https://github.com/hltj/kaggregator-demo)

最后出场的这个是原打算在分享中讲的开源缩略图服务 Kthumbor，终于完成了第一个可用版。服务框架使用 Ktor，100% Kotlin 开发，见下图：

![](/assets/kotlin/kthumbor-github.png)

另外，在 Kthumbor 项目中采用了测试驱动开发的方式（其中测试框架使用的是 [KotlinTest](https://github.com/kotlintest/kotlintest)），先写测试用例后写实现。
目前只实现了最简单的生成指定宽高内的缩略图的功能，后续还会实现放大、剪裁等功能，最终会实现一个生产级可用的缩略图服务。

Kthumbor 的源代码在这里：

[https://github.com/hltj/kthumbor](https://github.com/hltj/kthumbor)

欢迎反馈与交流。
需要说明的一点是，我并不想做纯雷锋，该项目采用 [AGPL-3.0](https://www.gnu.org/licenses/agpl-3.0.en.html) 协议发布，因此可以用于商业目的，但是任何修改都需要以同样协议（AGPL-3.0）开源出来。

关于分享中讲到的点以及 Kthumbor 项目，有机会再展开来看。
