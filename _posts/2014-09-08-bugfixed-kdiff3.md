---
layout:     post
title:      "用于 TortoiseHg 的 bugfix 版 KDiff3.exe"
date:       2014-09-08 19:05:00 +0800
categories: tips
---
[KDiff3](http://kdiff3.sourceforge.net/) 是自由软件，是一款相当不错的比较工具，尤其以三路比较/三路合并见长。
[TortoiseHg](https://tortoisehg.bitbucket.io/) 是最好用的 Mercurial 客户端之一，与 TortoiseSVN、TortoiseGit 不同，它是跨平台的自由软件，早期版本基于 PyGTK，从 2.0 新版开始基于 PyQt。其 Windows 版安装包中自带 KDiff3 作为版本比较/合并工具。
 
但是 KDiff3 旧版本（TortoiseHg 自带的 0.9.96a，2014.09.04 最新版 3.1.1 自带仍是这个版本）有[中文按照西文宽度处理导致显示错乱](http://sourceforge.net/p/kdiff3/bugs/162/)的问题：

![kdiff3_bug.png](/assets/kdiff3/kdiff3_bug.png)

最新版本 0.9.98 已修正该问题，但官网新版 KDiff3 的 Windows 可执行文件是基于 Qt5 的与基于 Qt4 的 TortoiseHg 不兼容，不能直接替换。于是分别编译了基于 Qt4 的 win32 及 win64 版的 KDiff3 0.9.98 二进制文件，可用来替换 TortoiseHg 自带的 KDiff3.exe。文件放在百度网盘分享了，下载地址：

[http://pan.baidu.com/s/1mg3b9uK#path=%252Fkdiff3_for_tortoisehg](http://pan.baidu.com/s/1mg3b9uK#path=%252Fkdiff3_for_tortoisehg)

新版的 KDiff3，打开上面中文文件显示正常：

![kdiff3_fixed.png](/assets/kdiff3/kdiff3_fixed.png)
