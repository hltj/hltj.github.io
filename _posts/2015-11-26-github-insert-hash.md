---
layout:     post
title:      "在 github 的 wiki/issue 中插入 #数字 但不生成链接"
date:       2015-11-26 14:36:41 +0800
categories: tips
---
[github](https://github.com/) 的 wiki（及 issue/pull request 注释）采用扩展的 markdown 语法。

<!--more-->

其中的一项扩展项就是输入`#数字`会被视作 issue/pull request 编号，并生成链接。这样如果要在 wiki、issue 正文及注释或者 pull request 注释中想输入`#数字`的普通文本反倒是件不容易的事。
 
当然，通常情况下不会有这种需求，在 github 上输入`#数字`，一般都是指 issue 或者 pull request。例外情况是引用其他的地方`#数字`表示。比如要在[这里](https://github.com/hltj/wxMEdit/issues/129)写“Unicode® Standard Annex #14”。
 
而输入“`&#35;14`”或者“`\#14`” 得到的结果均与“`#14`”相同，即被 github 渲染为带有对应 issue/pull request 链接的文本。
 
这时 `U+2026`（Word Joiner）就派上用场了，在原本的“零宽度不断行空白（Zero-width Non-breaking Space）”`U+FFFE` 作为 [BOM（字节序标记，Byte Order Mark）](https://zh.wikipedia.org/wiki/位元組順序記號)专用后，这个字符才是真正意义的“的零宽度不断行空白”。把它插入在“`#`”和数字之间，就不会被当成成 issue/pull request 编号渲染了。
 
这样输入就能得到普通文本“`#14`”了：
```html
#&#x2026;14 
```
