---
layout:     post
title:      "适用于 mipsel 架构 Debian 7（wheezy）的 wxMEdit 包"
date:       2014-06-15 00:25:53 +0800
categories: wxmedit
---
*原本于 2014.5.24 完成的，现在补记下。*

<!--more-->

wxMEdit 是对编码和十六进制编辑支持很好的文本编辑器。
wxMEdit 是 MadEdit 的后继，MadEdit 本身就是对编码和十六进制支持不错的高级文本编辑器，可惜很久没人维护了。创建 wxMEdit 的初衷就是提供一个持续维护、有 bug 修复、功能改善及重构的文本/十六进制编辑器。参见 [wxMEdit 简体中文简介](https://wxmedit.github.io/zh_CN/)、[处理编码问题利器之文本编辑器⑴——wxMEdit](/encoding/2013/03/30/wxmedit-deal-encoding.html)。
 
另外在接手 wxMEdit 后，对其编码支持还做了进一步的改善。参见 
[wxMEdit 正式启航](/wxmedit/2013/05/13/wxmedit-set-sail.html)、
[重构编码实现以便于支持更多编码](https://github.com/hltj/wxMEdit/issues/14)、
[添加编码分组](https://github.com/hltj/wxMEdit/issues/64)。
 
这是第一次在非 x86/x64 体系结构的系统中编译 wxMEdit 并打包。一切顺利。

安装包下载：
[wxmedit_2.9.7-1_mipsel-icu48.deb](http://downloads.sourceforge.net/project/wxmedit/2.9.7/wxmedit_2.9.7-1_mipsel-icu48.deb)

在 [wxMEdit 下载页](https://wxmedit.github.io/zh_CN/downloads.html) 中放了它的链接。
 
另外也在龙芯论坛发布了：
http://bbs.lemote.com/viewthread.php?tid=73268 。
