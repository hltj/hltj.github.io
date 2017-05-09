---
layout:     post
title:      "wxMEdit 正式启航"
date:       2013-05-13 01:12:00 +0800
categories: wxmedit
---
## wxMEdit
[![wxmedit.png](/assets/wxmedit.png) wxMEdit](https://wxmedit.github.io/zh_CN/) 是一个跨平台的文本编辑器，支持列编辑、十六进制编辑、语法高亮、自动换行、编码识别和转换、字数统计等功能。
wxMEdit 是 MadEdit 的后继，这些功能都是从 MadEdit 继承而来。

<!--more-->

MadEdit 本是一个我用过的非常不错的文本编辑器，尤其在十六进制编辑、编码识别和转换方面。可惜很久没人维护了，一直有接手它的想法。这些在之前写的[《处理编码问题利器之文本编辑器⑴——wxMEdit》](/encoding/2013/03/30/wxmedit-deal-encoding.html)中也提到了，还有之前这篇本来的标题和内容都是讲 MadEdit 的。
我们需要感谢 MadEdit 作者提供了这么好用的编辑器，向原作者致敬。

## 源起
虽然早就有接手这个久未维护项目的打算，但没有想到会这么快。

之所以做这个决定，是我自己在使用中发现了一个 bug。而这个 bug 不仅影响它擅长处理编码问题的美誉，还有可能造成数据损失——并且我发现的时候就是这样的情况，N 多 latin1 字母一不留神都被替换成了空格，好在及时撤消了。
就是[这个bug](https://github.com/hltj/wxMEdit/issues/2)，我后来调试跟踪：这些字母在忽略大小写查找或替换过程中，会被转小写，MadEdit 转小写是通过调用 towlower() 实现的，而 Windows CRT 的这个函数在当前 locale 是简体中文或者繁体中文时就会把一堆 latin1 字母转换为空格。也许这个应该算是 Windows CRT 的 bug。

这个 bug 的发现，让我开始了 MadEdit 的维护工作，开发环境的搭建比我预想中的顺利。定位 bug 不难，甚至为了修复这个 bug 引入 ICU 也不麻烦。让我觉得比较有挑战的是维护起这个新项目，这也是我之前没想到这么快接手它的原因之一。具体实施上也确实是这样：花在了解 googe code、github 和在两个站点创建项目完善信息、导入版本历史这些事情的工作量不亚于目前 wxMEdit 2.9.2 相对 MadEdit 改善的工作量。

## 进展现状
从发现上面提到的 bug 到如今，做了以下这些事情：
1. 决定 fork 原有 MadEdit 项目，取名 wxMEdit，M 取自 MadEdit，wx 取自所用跨平台框架 wxWidgets。
2. 为它设计了个 svg 图标，目前 wxMEdit 程序图标和 google code 项目头像用的都是它导出的位图。这个图标比较简陋不够美观，不过我想表达的含义都有了。三色块由 wxWidgets 图标简化而来，背景的记事本和笔为自绘，XM 两字母用的是 ubuntu-title 的字体。实际上这其中包含了我另外的设想和规划的含义，有机会以后再讲。
3. 改善原有编译配置，使 VC Express、MinGW、linux 都能够编译。
4. 从项目代码中移除了当年尚未集成到 wxWidgets 和 boost 的 wxAUI 和 xpressive，它们其实不应该存在于版本库中。
5. 从项目代码中移除图片转 xpm 的 GUI 小工具，它是独立程序，也不应该存在于版本库中。并且制作 xpm 本可用 GIMP/XnView/IfranView 等图片处理软件的。
6. 改善简体中文翻译，原有翻译中用了很多台湾习惯的词汇，改动量还是不小的，参见 [967f068](https://github.com/hltj/wxMEdit/commit/967f068b9864e86e0643bde2c27b05a39dab256d)。
7. 修正上面提到的 bug，另外因为原有实现的大小写转换用的是 towlower()，所以在 Windows 下不能支持 non-BMP 字符的大小写转换，实测也是如此。这两个问题都需要替换掉 towlower()，而在这方面做的比较好的是 ICU，于是在 wxMEdit 中引入了 ICU，虽然有点大材小用，不过考虑到以后还可以用它取代既有的很多功能，这样做还是合适的。具体实施时发现原有的大小写转换还不能简单的替换，并且有些重复代码，于是还对原有代码做了重构，具体参见 [83ee144](https://github.com/hltj/wxMEdit/commit/83ee144bbcdd8bd541f06ac10e11abe0b46d18c8)。
8. 在 google code 创建了项目 wxMEdit，并添加了简介、上传了相关下载。
9. MadEdit 的版本库中有不少无关的提交纪录，比如 sqrat、jkbind、squirrel，也许当初为了完善 MadEdit 的插件机制为实现脚本语言接口做准备的，但都只是第三方库，跟 MadEdit 主体代码没有任何交互，它们与 wxAUI、xpressive 类似不应该存在于版本库中。并且因为 MadEdit 程序代码从未用到它们，所以实际上可以不要这些版本纪录。导入 git 的时候忽略了这些版本，目前 github 的版本库中包含 MadEdit 的所有历史提交纪录除了这些忽略掉的，具体参见[这里](https://github.com/hltj/wxMEdit/issues/1)。
10. 查看原 MadEdit 项目的 bug 纪录，其中十六进制替换为空串[导致崩溃](https://github.com/hltj/wxMEdit/issues/5)还能重现。解决的同时还发现[十六进制查找空白也会导致崩溃](https://github.com/hltj/wxMEdit/issues/6)，也将其修复了，至此 wxMEdit 相对于 MadEdit 修正了三个严重 bug。
11. 相关功能点和 bug 修复都在 github 上创建对应 issue，并在版本提交日志中引用对应 issue 编号，这样在 github 上能够相互引用。

至此，wxMEdit 可以算是正式启航了。

## 规划展望
- wxMEdit 的 Unicode 支持还可进一步改善，可以更好利用 ICU。包括目前的全半角转换、Unicode字符分类、编码转换。还可引入区域语言相关大小写转换及排序等。
- 改善简繁转换，还可增加其他语言相关转换。
- 修复更多原 MadEdit 项目中用户反馈的 bug。
- 开发部分原 MadEdit 项目中用户提议的功能。
- 改进项目构建系统，改用 bakefile。
- 编译/打包更多平台的二进制。
- 对原有代码进行重构。这在上述工作中可能用到，另外也可改善部分模块/结构。

愿今后发展顺利！
