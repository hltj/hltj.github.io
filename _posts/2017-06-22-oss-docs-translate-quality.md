---
layout:     post
title:      "开源文档翻译的质量保障实践"
date:       2017-06-22 17:30:16 +0800
categories: translate
---
五月份，我宣布了 [Kotlin 官方参考文档翻译完毕](/kotlin/2017/05/15/kotlin-reference-translated.html)的消息，其中有提到这也是唯一一份完整且最新的官方参考文档翻译。不仅如此，其中值得一提的还有翻译质量。<!--more-->

Kotlin 中文站良好的翻译质量跟很多不错的翻译实践是分不开的。这些实践对于其他文档翻译项目也有很高的参考价值，特单独拿出来分享。

## 直接 fork 外文源站
这个在 [Kotlin 官方参考文档翻译完毕](/kotlin/2017/05/15/kotlin-reference-translated.html)已经介绍过：这样做的显著优势是官方站有任何更新可以及时合并进来。虽然这可能会引入冲突解决环节，并可能会影响翻译完整度，但这些与所带来的优势相比都是微不足道的。毕竟很难想象错过勘误的文档甚至内容陈旧的文档如何称之为质量好。

## 行级对照翻译
Kotlin 中文站翻译参考文档时，我们要求翻译后的中文与英文原文能够行级对应，即每行都是一一对应的。如果英文的一段文字分作了多行，中文也要按照原文分作多行，如下图所示：

![line_diff1.png](/assets/translate/line_diff1.png)

这样做有两点优势：
1. 源站更新后合并过来时，能够减少冲突发生，并且在发生冲突时也便于解决。
2. 能够逐行对照中英文，便于检查是否有遗漏或者失误的地方。

当然这样一来，会引出一个问题。比如上图的倒数第二段，英文加了两个断行，分别发生在 `of` 与 `inside` 之前。中文与之对应，分别断在`函数`与`中的`之前，而这样一来渲染出的网页就会在断词处多出空格，如下图红圈标注的地方：

![extra_spaces.png](/assets/translate/extra_spaces.png)

当然，如果现在打开 [https://www.kotlincn.net/docs/reference/classes.html](https://www.kotlincn.net/docs/reference/classes.html)，在文末并不会看到相应的空格，那是因为我们通过利用 HTML 注释的方式解决了这一问题，见下图：
![line_diff2.png](/assets/translate/line_diff2.png)

## 小屏校对
我基于 Kotlin 中文站的参考文档制作了[相应的 GitBook](https://www.gitbook.com/book/hltj/kotlin-reference-chinese/details)，之后发现用手机看 ePub 版电子书的时候更容易发现问题，因为手机屏幕尺寸较小，每屏的字数要比电脑屏少很多，更容易聚焦在局部。如果细读这页，就会发现第二段代码有些问题：

![extscope1.jpg](/assets/translate/extscope1.jpg)

也许你已经看出来了，注释`导入所有名为“goo”扩展`读起来不通顺，因为`扩展`前面少了个`的`。这样就发现了一处疏忽，加上之后就好了：

![extscope2.jpg](/assets/translate/extscope2.jpg)

上图再往下看，会发现代码中还有问题。没错，括号不对称，`fun usage` 应该以大括号结尾，但是代码中却是圆括号。通过对比我发现源站就有这一拼写错误，修正后提 PR 给源站，然后再合并回来，就是现在的样子了：

![extscope3.jpg](/assets/translate/extscope3.jpg)

## 统一术语
相信统一术语的重要性已经深入人心，这里只举一些实际的例子：
1. “**方法**”还是“**函数**”：受既有 Java 术语影响，早期的翻译中很多本该译为“函数”的地方也翻译成了“方法”，因为 Java 中统一称“方法”。
但 Kotlin 不一样，它既有“方法”也有“函数”，甚至也可以像 C++ 那样将方法称为“成员函数”。
所以最好的翻译方式就是“忠于原文”，即原文用“function”则翻译为“**函数**”，原文用“method”则翻译为“**方法**”。
2. “**范围**”不准确：在统一校对前，译文中有不少“范围”这一术语，其中有的原文是“range”也有的是“scope”。
对于“scope”更确切的译法应该是“**作用域**”。
而对于“range”更确切的译法应该是“**区间**”，尤其是“close range”自然应该译作“闭区间”。
3. “**型变**”：泛型相关的术语“variance”译为“型变”、“covariant”译为“协变的”、“contravariant”译为“逆变的”。而“invariant”没有译为“不变的”，
为避免歧义，结合型变这一词根，将其翻译为“**不型变的**”。
4. “**注解**”怎么办：通常注解的名词形式“annotation”很好翻译，即“注解”。而其动词形式如果也翻译为“注解”就比较麻烦了，尤其遇到 `annotated with the annotation` 这样的词句时。
因此，Kotlin 中文站通常将参考文档中的“annotate”翻译成“**标注**”或者“**用注解标注**”。这样刚刚提到的词句就翻译为“用该注解标注的”。
5. “**参见**”与“**参阅**”：在统一校对前，“see ... for”有多种译法，包括“查看”、“参见”、“参阅”等。统一校对后，将“see ... for”统一译为“**关于……请参见**”。
而将“read ... for”统一译为“**关于……请参阅**”。

## 注重细节
其实上面介绍的几条都包含了不少细节在里面。Kotlin 中文站参考部分在翻译过程中所注重的细节还有：
1. “the”有时也要翻译。尤其用于特指上文中提到的一项事物时，通常译作“该”。
2. 注释也翻译。参见上面的例图。
3. 中文与英文之间留空格。
4. 文中以及代码注释中出现的标点都替换为全角。
5. 表示代码省略的 `...` 也替换为中文省略号`……`。
6. 英文中表示并列关系的逗号，翻译后转换为顿号。如  
“The following escape sequences are supported: `\t`, `\b`, `\n`, `\r`, `\'`, `\"`, `\\` and `\$`.”  
翻译为  
“
支持这几个转义序列：`\t`、 `\b`、`\n`、`\r`、`\'`、`\"`、`\\` 与 `\$`。”

## 群策群力
- 翻译  
一个人能力毕竟有限，Kotlin 中文站是由大家共同翻译的，参见[贡献者名单](https://www.kotlincn.net/contribute.html#中文站翻译贡献者)。
- 校对  
后期主要由我来统一校对，使用了行级比较、小屏校对等方法，还用到了一些[脚本](https://github.com/hltj/markdown-hash-checker)与[小工具](https://github.com/DemoJameson/prevent-markdown-newline-become-space)作为辅助。
但是即便如此，还是会有在所难免的疏漏之处。
这时多人的力量再次凸显：多人评审、读者反馈与[协作改进](https://github.com/hltj/kotlin-web-site-cn/pulls?q=is%3Apr+is%3Aclosed)这些实践中的多人参与让 Kotlin 中文站的翻译质量越来越好。

由衷感谢参与 Kotlin 中文站翻译与改进的每个人。
Kotlin 中文站的翻译工作还在进行中，教程部分还有很多章节需要翻译，另外随着官方参考文档的更新中文站也需及时更新，欢迎更多人参与进来。
