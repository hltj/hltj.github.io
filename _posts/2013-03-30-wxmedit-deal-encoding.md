---
layout:     post
title:      "处理编码问题利器之文本编辑器⑴——wxMEdit"
date:       2013-03-30 22:47:00 +0800
categories: encoding
---
***工欲善其事，必先利其器。***  
***　　　　　　——《论语》***


好的工具的确能让工作事半功倍，尤其处理棘手的疑难问题时。接触过不少文本编辑器，其中有两个我觉得称得上处理编码问题的利器：[Vim](http://www.vim.org/) 和 [wxMEdit](https://wxmedit.github.io/zh_CN/)，都是跨平台的开源软件。

- ![vim.png](/assets/vim_64.png) [http://www.vim.org/](https://www.vim.org/)
- ![wxmedit.png](/assets/wxmedit_64.png) [https://wxmedit.github.io/zh_CN/](https://wxmedit.github.io/zh_CN/)

先说后者吧，它有些像 UltraEdit，可能更平易一些。本文早期版的这个编辑器本来是 [MadEdit](http://sourceforge.net/projects/madedit/) 的，大约是一位台湾朋友写的软件，中文支持不错，可惜目前无人维护。
本来这也无妨，可是发现了它一个严重的 bug，可能会造成数据损失，于是 fork 了这个项目，就是 [wxMEdit](https://wxmedit.github.io/zh_CN/)。
wxMEdit 在 MadEdit 的基础上做了很多改善，其中包括处理编码相关功能改进：改善编码识别效果、支持 GB18030 及其 BOM、添加编码分组、添加 ISO-8859-16、Windows 1258、KOI8-R、KOI8-U 等编码支持、部分 bug 修复等。

![wxmedit_ubuntu.png](/assets/wxmedit/wxmedit_ubuntu.png)
wxMEdit 内置四种语法高亮风格，这是 Dark 风格下 python 程序语法高亮效果。

UltraEdit 所具有的语法高亮、多文件查找替换、正则匹配、编码识别和转换、列编辑和十六进制编辑这些功能，wxMEdit/MadEdit 都具有。若未用到 UltraEdit 更多的功能，甚至可以用它作为 UltraEdit 的替代品。

值得一提的是，wxMEdit/MadEdit 有两个方面**完胜** UltraEdit，分别是**十六进制编辑**和**编码的识别/转换**。我就是在寻找更好的十六进制编辑器时发现 MadEdit 的，它的十六进制编辑功能的确强大，wxMEdit 在这方面改进包括十六进制串复制粘贴设置、覆盖粘贴等。这里只说编码相关的，它的十六进制编辑功能有两处编码相关的特性明显优于 UltraEdit：
1. 十六进制编辑模式，右侧文本区域可以按各种支持的编码显示，而 UltraEdit 只有当前 locale 或者 UTF-16 两种编码支持。
2. wxMEdit 十六进制编辑显示的时原始二进制信息，如有 [BOM](https://zh.wikipedia.org/wiki/位元組順序記號) 的 UTF-8 文本开头一定是 EF BB BF,而 UltraEdit 无论有没有 BOM 一律按照有显示，无论 UTF-8、UTF-16、UTF-32 一律按照 UTF-16 显示。

以经典的 [Big5“許功蓋”问题](https://zh.wikipedia.org/wiki/大五碼#.E8.A1.9D.E7.A2.BC.E5.95.8F.E9.A1.8C)为例：

![wxmedit_xgg0.png](/assets/wxmedit/wxmedit_xgg0.png)

打开该对应文本，然后从“查看->编码->东亚文字”选择“MS950”（Big5 的超集，微软扩展的 Big5 实现），然后切换到十六进制编辑模式，如上图。可以看出 wxMEdit 在简体中文（GBK/CP936）的操作系统环境中编辑 Big5 文本，其十六进制编辑模式的右侧文本区域还是可以正常按 Big5 编码显示（**这在 UltraEdit 中是不可能的**），且显示的是原始十六进制数据。另外可以看到其状态栏会显示文件的编码及行尾符信息。

然后从“查看->编码->西欧文字”选择“ISO-8859-1”（或者从“查看->编码->OEM”选择“CP437”等，总之就是选一个单字节编码）再看：

![wxmedit_xgg1.png](/assets/wxmedit/wxmedit_xgg1.png)

切换编码的同时，右侧文本区域会同步更新，如图所示。这样就不难理解“許功蓋”问题本质了：因为这些汉字的 Big5 编码的第二个字节是“`\`”（`'\x5C'`）,“`\`”这个字符在很多编程语言如 C、Bourne Shell、MySQL 等中都是用作转义字符，如果把这样的汉字当作西文（如 ISO-8859-1）来处理就可能引发编译错误、SQL 注入等问题。
 
我们再以只“wxMEdit - 跨平臺的文本/十六進制編輯器”这几个字的文本为例来看各种编码属性情况。先列下文件列表，其中文件名以 b 结尾的包含 BOM，文件名中 le 表示小端次序、be 表示大端次序：

```
-rw-r--r-- 1 jyw jyw  37 10月 30 16:04 wxmedit_b5.txt
-rw-r--r-- 1 jyw jyw  37 10月 30 16:04 wxmedit_g.txt
-rw-r--r-- 1 jyw jyw  41 10月 30 16:04 wxmedit_gb.txt
-rw-r--r-- 1 jyw jyw  50 10月 30 16:04 wxmedit_u8.txt
-rw-r--r-- 1 jyw jyw  53 10月 30 16:04 wxmedit_u8b.txt
-rw-r--r-- 1 jyw jyw  48 10月 30 16:04 wxmedit_u16be.txt
-rw-r--r-- 1 jyw jyw  50 10月 30 16:04 wxmedit_u16beb.txt
-rw-r--r-- 1 jyw jyw  48 10月 30 16:04 wxmedit_u16le.txt
-rw-r--r-- 1 jyw jyw  50 10月 30 16:04 wxmedit_u16leb.txt
-rw-r--r-- 1 jyw jyw  96 10月 30 16:04 wxmedit_u32be.txt
-rw-r--r-- 1 jyw jyw 100 10月 30 16:04 wxmedit_u32beb.txt
-rw-r--r-- 1 jyw jyw  96 10月 30 16:04 wxmedit_u32le.txt
-rw-r--r-- 1 jyw jyw 100 10月 30 16:04 wxmedit_u32leb.txt
```

可以看出包含 BOM 的文件要比同编码不含 BOM 的文件多几个字节（GB18030：4、UTF-8：3、UTF-16\*：2、UTF-32\*：4），即 BOM 所占字节数。

直接打开这些文件，wxMEdit 当可自动识别它们的编码。这也是 wxMEdit 适合编码（尤其中文）相关处理的又一亮点。编码识别效果改进也是 wxMEdit 相对于 MadEdit 的重要改进之一。分别切换各个文件，可以从状态栏很方便地看到各文件的编码属性，以“wxmedit_gb.txt”为例：

![wxmedit_gbbom.png](/assets/wxmedit/wxmedit_gbbom.png)

这些文件状态栏显示编码属性信息为：

```
wxmedit_b5.txt       MS950
wxmedit_g.txt        GB18030
wxmedit_gb.txt       GB18030.BOM
wxmedit_u8.txt       UTF-8
wxmedit_u8b.txt      UTF-8.BOM
wxmedit_u16be.txt    UTF-16BE
wxmedit_u16beb.txt   UTF-16BE.BOM
wxmedit_u16le.txt    UTF-16LE
wxmedit_u16leb.txt   UTF-16LE.BOM
wxmedit_u32be.txt    UTF-32BE
wxmedit_u32beb.txt   UTF-32BE.BOM
wxmedit_u32le.txt    UTF-32LE
wxmedit_u32leb.txt   UTF-32LE.BOM
```

编码、大小端次序、有无 BOM 都包含了。

通过 wxMEdit/MadEdit 也可以很容易转换编码或者增删 BOM：
- 转换编码：“工具->转换文件编码…”然后选择新的编码即可。
- 增删BOM：“工具->有/没有Unicode BOM->删除/插入BOM”即可。

此外 wxMEdit 的十六进制插入、删除、替换都要比 UltraEdit 更加强大和易用。
wxMEdit 还支持字数统计、大小写转换、全半角转换、中文简繁转换等中文相关特殊功能。当然从 MadEdit 继承过来的的简繁转换效果不好，这是我打算接手并改善的地方之一。
wxMEdit 还适合打开比较大（如 100M 以上）的文件，它打开大文件时默认以十六进制模式打开。

其他的 wxMEdit 功能可以浏览下它的各个菜单。

如果没有其他 UltraEdit 特有的高级功能还在用，以免费、开源的 wxMEdit 来代替昂贵的 UltraEdit 是个不错的选择。

有任何问题和建议可以留言，或者发邮件给我 jiaywe[ａｔ]gmail.com。
