---
layout:     post
title:      "扔掉 OpenSSL，拥抱 LibreSSL——远离心脏出血与溺亡"
date:       2017-05-26 18:12:16 +0800
categories: security
---
## 缘起

早就想写这篇推荐大家用 [LibreSSL](http://www.libressl.org/) 取代 [OpenSSL](https://www.openssl.org/) 的文章，总算能够如愿。

2016 年 3 月初 OpenSSL 再次爆出与 2014 年的[心脏出血（Heartbleed）](https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2014-0160)同样严重的新漏洞——[溺亡（DROWN）](https://cve.mitre.org/cgi-bin/cvename.cgi?name=cve-2016-0800)！<!--more-->在获悉这一消息的时候第一时间我去看 LibreSSL 有没有受影响，正如所料——没有（见下图）！这让我觉得 LibreSSL 取代 OpenSSL 更加必要。

![LibreSSL 不受 DROWN 攻击影响](/assets/libressl/libressl_no_drown.jpg "LibreSSL 不受 DROWN 攻击影响")

关于 DROWN 溺亡漏洞如果还没了解过，可以参考：
- [https://www.openssl.org/news/secadv/20160301.txt](https://www.openssl.org/news/secadv/20160301.txt)
- [https://www.wosign.com/News/2016-0302-00.htm](https://www.wosign.com/News/2016-0302-00.htm)

## LibresSSL 是什么

大家可能对 [LibreSSL](http://www.libressl.org/) 不够熟悉，但对它的[收藏夹图标（favicon）](http://www.libressl.org/favicon.ico)![LibreSSL favicon——心脏滴血](http://www.libressl.org/favicon.ico "LibreSSL favicon——心脏滴血")恐怕会相当熟悉。
你没看错，就是 OpenSSL 心脏出血漏洞的图标。它是 [OpenBSD](http://www.openbsd.org/) 开发者在 OpenSSL 爆出心脏出血漏洞之后 fork 的一个分支，旨在提供一个比 OpenSSL 更安全的替代品。
OpenBSD 是一个以安全著称的操作系统，LibreSSL 遵循其他 OpenBSD 项目的安全指导原则。

LibreSSL 与 OpenSSL 都是和[传输层安全](https://zh.wikipedia.org/wiki/%E4%BC%A0%E8%BE%93%E5%B1%82%E5%AE%89%E5%85%A8 "传输层安全")（TLS，Transport Layer Security）协议的开源实现，如需了解 TLS 可参见其[维基词条](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0)或自行搜索。

字面上看，LibreSSL 有两种断词方式：
- Libre-SSL，Libre 来自拉丁语，意为“自由”，大家应该并不陌生，因为 LibreOffice 已经事实上取代了 OpenOffice，没准 LibreSSL 也会有这一天。
- Lib-Re-SSL，即重新实现的 libssl。

## LibreSSL 更安全吗
**当然**。

如上所述，OpenBSD 当初创立这个项目就是为了能够避免再出现像 Heartbleed 这样的严重漏洞。
而它不受 DROWN 影响也说明了它的努力没有白费，参见 [LibreSSL 不受 DROWN 攻击影响](http://undeadly.org/cgi?action=article&sid=20160301141941)。

如果去看 LibreSSL 的 Release Note，会发现有不少 OpenSSL 的漏洞对于 LibreSSL 根本不存在，比如 [2.1.4 的 Release Note](https://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.1.4-relnotes.txt)，见下图：

![LireSSL 2.1.4 Release Note](/assets/libressl/libressl_214.png "LireSSL 2.1.4 Release Note")

红线所标的标题下的 CVE 分别在 LibreSSL 早期版修复或者对于 LibreSSL 根本不存在。

另外还可以参考 [FreeBSD wiki 上的一项对比](https://wiki.freebsd.org/LibreSSL#LibreSSL_.28and_OpenSSL.29_Security_Vulnerabilities)：

![FreeBSD wiki 上的漏洞比较](/assets/libressl/cmp_at_freebsd.png "FreeBSD wiki 上的漏洞比较")

很明显，LibreSSL 的漏洞数与严重程度要比 OpenSSL 少的多、轻的多。

## 能够取代 OpenSSL 吗

**要分情况**。

有些发行版已经将原有的 OpenSSL 替换为 LibreSSL 了，知名的有：
 - [OpenBSD 自 5.6 起](https://marc.info/?l=openbsd-announce&m=141486254309079)
 - [Mac OSX 自 10.11 El Capitan 起](https://medium.com/@FredericJacobs/apple-ios-9-security-privacy-features-8d82d9da10eb#3ae7)
 - [Alpine Linux  自 3.5.0 起](https://lists.alpinelinux.org/alpine-announce/0022.html)

在以上这些系统中应该除了实测必须要用 OpenSSL 软件外，应该都可以用 LibreSSL 取代 OpenSSL。

而对于其他大多数还没有将系统自带的 OpenSSL 替换为 LibreSSL 的会麻烦一些，并且也可能做不到彻底替换。只是对于编译安装的软件，可以尽量用 LibreSSL 取代 OpenSSL。

## 编译示例
以下示例的参考系统为 CentOS 7 和 Ubuntu 16，分别作为 RedHat 系、Debian 系发行版的代表。通常网站的 HTTPS 证书配置在 [Nginx](https://nginx.org/) 或 [HaProxy](https://www.haproxy.org/) 上，因此编译二者时用 LibreSSL 取代 OpenSSL 对于网站安全性的价值更大一些，下文就以它们为例。

### 删掉 OpenSSL 开发包
注意，要删掉的是**开发包**，换句话说，以 `-devel` 或者 `-dev` 结尾的，不要把基本包也删了。
- CentOS 7：`yum erase openssl-devel`
可能会删掉 `mysql-devel` 等开发包，**不要**在已经使用的机器上做测试。
- Ubuntu 16：`apt remove libssl-dev`
**不要**在已经使用的机器上做测试。

### 编译安装 LibreSSL
确保已安装 gcc、wget、make，如果未安装请分别按以下方式安装：
- CentOS 7：`yum install -y gcc wget`
- Ubuntu 16：`apt install -y gcc make`

编译安装 LibreSSL：

```bash
wget https://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.5.4.tar.gz
tar xvf libressl-2.5.4.tar.gz
cd libressl-2.5.4
./configure --prefix=/usr/local
make
make install
```
确保 /usr/local/lib 已经在 ld.so.conf 中或者 ld.so.conf.d/\*.conf 中。如果未在可通过 `echo /usr/local/lib > /etc/ld.so.conf.d/local.conf` 来添加。
然后更新缓存，并查看 libressl 的 libssl.so 版本

```bash
ldconfig
ldconfig -v | fgrep libssl
```

主版本号至少两位数，例如

```
libssl.so.43 -> libssl.so.43.0.2
```

这个是 LibreSSL 的，主版本号比较小的是系统自带的。

### 编译安装 Nginx
确保 pcre、zlib 的开发库已安装，如果未安装请分别按以下方式安装：
- CentOS 7：`yum install -y pcre-devel zlib-devel`
- Ubuntu 16：`apt install -y zlib1g-dev libpcre3-dev`

编译安装 Nginx（当前最新稳定版为 1.12.0）：

```bash
wget http://nginx.org/download/nginx-1.12.0.tar.gz
tar xvf nginx-1.12.0.tar.gz
cd nginx-1.12.0
./configure --with-http_ssl_module
make
make install
```

验证：`ldd /usr/local/nginx/sbin/nginx | fgrep ssl` 的输出应该是这样的：

```
libssl.so.43 => /usr/local/lib/libssl.so.43 (0x00007f4fc5684000)
```

### 编译安装 HaProxy
需要使用 2.4.5 版 LibreSSL，目前最新稳定版的 LibreSSL 与目前最新稳定版的 HaProxy 不兼容。
```bash
wget http://ftp.openbsd.org/pub/OpenBSD/LibreSSL/libressl-2.4.5.tar.gz
tar xvf libressl-2.4.5.tar.gz 
cd libressl-2.4.5/
./configure --prefix=/usr/local
make
make install
ldconfig
```

编译安装 HaProxy（目前最新稳定版为 1.7.5）：
```bash
wget http://www.haproxy.org/download/1.7/src/haproxy-1.7.5.tar.gz
tar xvf haproxy-1.7.5.tar.gz
cd haproxy-1.7.5
make TARGET=linux \
    USE_ZLIB=1 \
    USE_PCRE=1 \
    USE_OPENSSL=1 \
    SSL_INC=/usr/local/include \
    SSL_LIB=/usr/local/lib
make install
```

验证：`ldd /usr/local/sbin/haproxy | fgrep ssl` 应该输出：
```
libssl.so.39 => /usr/local/lib/libssl.so.39 (0x00007fd34408f000)
```

### 存在问题
有些软件可能用了兼容性不好的 OpenSSL API，需要实测下用 LibreSSL 取代后的兼容性如何，比如 HaProxy 与 LibreSSL 2.4.5 兼容，但是与目前最新版 2.5.4 一起编译会有问题。

## 未来展望
从 LibreSSL 与 OpenSSL 安全漏洞数量及严重程度的对比来看，LibreSSL 要安全很多，从目前来看也应该用 LibreSSL 来取代 OpenSSL。

但是与 OpenSSL 一样，LibreSSL 也是用 C 语言开发的，而由于语言本身的局限性，C 语言无法为内存安全提供良好的保障。可以说像 TLS 支持这样的重视安全且用于安全领域的底层库用 C 语言开发并不适合。目前来看，对于这一领域的首选开发语言当属 [Rust](https://www.rust-lang.org/zh-CN/)，并且已经有一些进展，比如 [Rust TLS](https://github.com/ctz/rustls) ，当然距离成熟稳定乃至流行起来还有很多路要走。
