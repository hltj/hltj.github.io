---
layout:     post
title:      "体验 Java 9（1）：从 Hello World 到 Lombok"
date:       2017-09-22 12:43:06 +0800
categories: java
---
Java 9 正式版已于当地时间的 9 月 21 日（北京时间大约是今天凌晨）如期发布。可[在 Oracle 官网下载](http://www.oracle.com/technetwork/java/javase/downloads/jdk9-downloads-3848520.html)。
<!--more-->

Java 9 没有像 Java 5/Java 8 那样引入新的编程范式而给语言本身带来革命性的改进，不过 Java 9 的改动还是很大的，尤其是引入模块化对 JDK 与运行时的改动都很大。
现在网上能找到很多介绍 Java 9 新特性的文章，这里不再赘述，只简要列举如下：
  1. 模块化（Jigsaw 项目）
  2. G1 成为默认垃圾收集器
  3. JDK 自带 REPL 即 JShell
  4. 支持 HTTP/2 与 WebSocket 的新版 HTTP 客户端
  5. 多版本 Jar 包
  6. 语言、JDK、运行时与工具的其他优化与改动

如何切换到 Java 9，Oracle 官网上也有详细的[迁移文档](https://docs.oracle.com/javase/9/migrate/toc.htm)。

而如何获得最直观的感受，就不妨跟我一起来亲身体验下了。

## Hello World
千万不要觉得用体验一门新语言的方式来体验一个新版本是小儿科或者小题大做。
因为 Hello World 能运行至少意味着基本环境没问题，如果不能就说明我们发现了问题。

### Hello JShell
JShell 是 JDK 9 自带的交互式编程命令行，即 REPL（Read-Eval-Print Loop 的简写，直译为 “读取-求值-输出”循环），非常适合快速实验一些代码片段。
它遵循通用的命令行操作，如 `Tab` 自动补全、`Ctrl-D` 退出、`Ctrl-R` 反向搜索、`Ctrl-S` 正向搜索等等。
运行 `jshell --help` 可以查看其命令行选项说明，直接运行 `jshell` 进入交互式模式后，可以通过 `/help` 查看交互式模式帮助。
更详细的用法说明可参见其[官方文档](https://docs.oracle.com/javase/9/tools/jshell.htm)。

#### 交互式 Hello World
运行 `jshell` 进入交互式模式：

``` bash
$ jshell
|  欢迎使用 JShell -- 版本 9
|  要大致了解该版本, 请键入: /help intro

jshell> 
```

`jshell> ` 是 JShell 的命令提示符，可在其后直接键入代码。在 JShell 中直接输入可运行的代码就可以，无需定义额外的类与 `main` 函数，还可以省略单行语句的分号。因此运行 Hello World 只需键入 `System.out.println("Hello, World!")` 即可，并且在输入过程中还可以通过 Tab 键补全：

```
jshell> System.out.println("Hello, World!")
Hello, World!
```

大功告成。当然，如果只是想在 JShell 中查看一个表达式的求值结果，并不需要调用 `System.out.println()` 这么麻烦，而只需直接键入表达式然后回车即可：

```
jshell> "Hello, World!"
$2 ==> "Hello, World!"

jshell> Math.PI * 1.5 * 1.5
$3 ==> 7.0685834705770345
```

这实际上是 JShell 的“反馈”，如果希望反馈中包含表达式的类型，可以在启动时加命令行选项 `-v` 或者通过交互模式的命令 `/set feedback verbose` 切换到详细反馈模式：

```
jshell> /set feedback verbose
|  反馈模式: verbose

jshell> "Hello, World!"
$4 ==> "Hello, World!"
|  已创建暂存变量 $4 : String

jshell> double circumference(double radius) {
   ...>     return Math.PI * radius * radius;
   ...> }
|  已创建 方法 circumference(double)

jshell> circumference(1.5)
$6 ==> 7.0685834705770345
|  已创建暂存变量 $6 : double
```

更多交互式模式的用法可以查阅文档或自行发掘，现在通过快捷键 `Ctrl-D` 或者交互式命令 `/exit` 退出 JShell，来看下非交互式 JShell 版 Hello World。

#### Hello JShell 脚本
除了交互式运行，JShell 还可以用作脚本执行。
现在我们将输出 Hello World 的语句写入一个文件：

``` bash
$ echo 'System.out.println("Hello, World!")' > hello.jsh
```

然后用 JShell 运行之：

``` bash
$ jshell hello.jsh
Hello, World!
|  欢迎使用 JShell -- 版本 9
|  要大致了解该版本, 请键入: /help intro

jshell> 
```

在执行完 hello.jsh 中的语句后进入了交互式模式，这并不是预期的结果，我们希望它执行完就退出而不是进入交互式模式。
当然，这只需在文件末尾追加 `/exit` 命令即可：

```
$ echo '/exit' >> hello.jsh
$ jshell hello.jsh
Hello, World!
```

又大功告成了！（奇怪，我为什么要说又 :P）。比较遗憾的是 JShell 目前还不支持 shbang，不能将 `.jsh` 文件直接作为可执行脚本来用。

## 构建运行 Hello World
对于 Hello World 这样简单的代码片段，确实只用 JShell 演练就可以了。只是这并不能验证构建运行也一切正常，因此还要继续来看使用 JDK 9 构建运行 Hello World 程序的情形。

### 直接编译运行
直接编译运行与 JDK 8 及以前版本没有任何区别：

``` bash
$ cat > Hello.java << END
> public class Hello {
>     public static void main(String[] args) {
>         System.out.println("Hello, World!");
>     }
> }
> END
$ javac Hello.java
$ java Hello
Hello, World!
```

而我们在实际项目中通常并不能如此简单地编译运行，而需使用 Maven 或者 Gradle 来构建项目。
> **注：**本文中测过的 Maven 版本为 3.3.9、3.5.0；Gradle 版本为 3.5-RC-2 到 4.0.2。

### Hello Maven
现在我们将 Hello.java 的开头加上包声明 `package me.hltj.java9demo.hello;`，并将其放到 `src/main/java/me/hltj/java9demo/hello` 目录下，然后创建 pom.xml 如下：

``` xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">

    <modelVersion>4.0.0</modelVersion>

    <groupId>me.hltj</groupId>
    <artifactId>java9demo-hello</artifactId>
    <version>1.0-SNAPSHOT</version>

</project>
```

这样就创建了一个简单的 maven 项目。
如果使用 JDK 8，此时就可以通过 `mvn package` 生成 jar 包，并通过以下命令来运行：

``` bash
$ java -cp target/java9demo-hello-1.0-SNAPSHOT.jar me.hltj.java9demo.hello.Hello
```

但是使用 JDK 9 编译会报错：

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project java9demo-hello: Compilation failure: Compilation failure:
[ERROR] Source option 1.5 is no longer supported. Use 1.6 or later.
[ERROR] Target option 1.5 is no longer supported. Use 1.6 or later.
[ERROR] -> [Help 1]
```

这是因为 Maven 默认的源代码和目标版本都是 1.5，而 JDK 9 支持的最低版本是 1.6，因此必须在 pom.xml 中显式指定这两个版本号：

```
    <properties>
        <maven.compiler.source>1.9</maven.compiler.source>
        <maven.compiler.target>1.9</maven.compiler.target>
    </properties>
```

之后就可以像使用 JDK 8 一样正常构建和运行了。

### Hello Gradle
首先为项目创建 gradle wrapper 并基于 maven 项目初始化：

``` bash
$ gradle wrapper --gradle-version=4.0.2 --distribution-type=bin
$ ./gradlew init
```

然后将 build.gradle 简化为：

``` gradle
apply plugin: 'java'

group = 'me.hltj'
version = '1.0-SNAPSHOT'
```

无需指定源代码与目标版本就可以正常构建运行：

```
$ ./gradlew build
BUILD SUCCESSFUL in 1s
$ java -cp build/libs/java9demo-hello-1.0-SNAPSHOT.jar me.hltj.java9demo.hello.Hello
Hello, World!
```

除了构建工具外，在实际项目开发中也离不开 IDE，接下来我们就看下在不同 IDE 中使用 Java 9 构建与运行的情况。

### Hello IDEA
最新稳定版的 IDEA（2017.2.4）已支持 Java 9。
在其中使用 Java 9 构建与运行 Hello World 与 Java 8 无异，只需选用 JDK 9 即可：

![](/assets/java9/idea-jdk9.png)

如果尚未添加点击上图所示的对话框右上方的“New”按钮来添加。
之后可以照常构建与运行 Hello World，无论创建项目时选 Java 项目、Maven 项目还是 Gradle 项目。

### Hello Eclipse
最新稳定版的 Eclipse Oxygen（4.7.0）并没有内置 Java 9 支持，需要安装一个 [Beta 版的 Java 9 支持插件](https://marketplace.eclipse.org/content/java-9-support-beta-oxygen)才行：

![](/assets/java9/eclipse-java9.png)

之后就可以添加 JDK 9 了：

![](/assets/java9/eclipse-jdk9.png)

添加完成后就可以通过 JDK 9 照常构建与运行 Hello World，无论创建项目时选 Java 项目、Maven 项目还是 Gradle 项目。

### Hello NetBeans
最新稳定版的 NetBeans（8.2）并不支持 Java 9，也没有用于 Java 9 支持的插件。想要支持 Java 9 只能用每日构建版，可在这里下载： [http://bits.netbeans.org/download/trunk/nightly/latest/](http://bits.netbeans.org/download/trunk/nightly/latest/)。

在安装时选 JDK 9。之后可以照常构建与运行 Hello World，无论创建项目时选 Java 项目还是 Maven 项目。

> **注：**NetBeans 虽然有 Gradle 插件，但是在每日构建版中安装之后也基本无法使用。

### Hello World 小结
以任何方式通过 JRE 9 运行 Hello World 都如预期般顺利。使用 Gradle 或者 IDEA 通过 JDK 9 构建 Hello World 示例项目也一如既往地顺利。而对于 Maven 需要注意显式指定源代码与目标版本，对于 Eclipse 需要安装插件，对于 NetBeans 目前只能用每日构建版。

> **注：**示例代码已放到 Github 上： [https://github.com/hltj/java9demo/tree/master/hello](https://github.com/hltj/java9demo/tree/master/hello)。

Hello World 总体上看还比较顺利，接下来我们看下更具挑战的 Lombok 支持情况。

## Java 9 中使用 Lombok
代码冗长是 Java 语言广为病诟的缺点之一。而其中很多场景都可以使用 Lombok 来简化，这在实际项目中广泛应用。因此验证 JDK 9 以及使用 JDK 9 的构建工具与 IDE 对 Lombok 的支持情况尤为重要，会关系到很多既有项目能否迁移到 Java 9。

这里以一段简单代码为例，验证 Maven、Gradle、IDEA、Eclipse、NetBeans 在使用 JDK 9 时对 Lombok 的支持情况。代码如下：

``` java
package me.hltj.java9demo.lombok;

import lombok.*;
import lombok.extern.slf4j.Slf4j;

@Value
@ToString
class Language {
    @NonNull
    private String name;
    private int version;
}

@Slf4j
public class LombokDemo {
    public static void main(String[] args) {
        val version = Integer.parseInt(System.getProperty("java.version"));

        val java = new Language("Java", version);

        log.info("Hello Lombok in {}.", java);
    }
}
```

### 使用 Maven 构建
我们只需创建相应代码目录结构，然后在 pom.xml 中配置以下依赖即可在 Java 8 下通过编译：

``` xml
    <dependencies>
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.16.18</version>
            <scope>provided</scope>
        </dependency>
        <dependency>
            <groupId>ch.qos.logback</groupId>
            <artifactId>logback-classic</artifactId>
            <version>1.2.3</version>
        </dependency>
    </dependencies>
```

当然为了便于运行还引入了 assembly 插件，另外设置源代码与目标级别为 1.8，完整的 pom.xml 参见 [https://github.com/hltj/java9demo/blob/7a704a1b4a60cab74e454ab8d2d7edda30af430f/lombok/pom.xml](https://github.com/hltj/java9demo/blob/7a704a1b4a60cab74e454ab8d2d7edda30af430f/lombok/pom.xml)。此时使用 JDK 8、JDK 9 都能正常编译，但只有在 Java 9 环境中才能正常运行，因为 Java 8 的版本号如“1.8.0_144”不能解析为整数，运行时会抛异常。

但是当我们把 pom.xml 中的源代码与目标的版本号升级为 1.9 时，却编译失败了：

```
[ERROR] Failed to execute goal org.apache.maven.plugins:maven-compiler-plugin:3.1:compile (default-compile) on project java9demo-lombok: Compilation failure: Compilation failure: 
[ERROR] /home/hltj/java9demo/lombok/src/main/java/me/hltj/java9demo/lombok/LombokDemo.java:[6,1] 程序包 javax.annotation 不可见
[ERROR]   (程序包 javax.annotation 已在模块 java.xml.ws.annotation 中声明, 但该模块不在模块图中)
```

这是因为 Lombok 默认在每个生成的方法上添加了注解 `@javax.annotation.Generated("lombok")`，而在 Java 9 引入模块化系统后不再默认提供这一注解。
解决方案有：
  1. 在 lombok.config 中添加以下配置来禁用这一行为：  
```
lombok.addJavaxGeneratedAnnotation = false
```
  2. 给 javac 增加参数 `--add-modules=java.xml.ws.annotation`。
  3. 采用安卓项目针对此问题的解决方案，添加一个提供该注解的依赖： `org.glassfish:javax.annotation:10.0-b28`。

例如采用第三种方案，只需在 pom.xml 中追加以下配置：

``` xml
        <dependency>
            <groupId>org.glassfish</groupId>
            <artifactId>javax.annotation</artifactId>
            <version>10.0-b28</version>
            <scope>provided</scope>
        </dependency>
```

之后就可以使用 Java 9 顺利构建运行了：

```
$ java -jar target/java9demo-lombok-1.0-SNAPSHOT-jar-with-dependencies.jar
17:24:57.723 [main] INFO me.hltj.java9demo.lombok.LombokDemo - Hello Lombok in Language(name=Java, version=9).
```

### 使用 Gradle 构建
初始化 Gradle 之后将 build.gradle 改成这样即可使用 JDK 8 编译：

``` gradle
apply plugin: 'java'
apply plugin: 'application'

group = 'me.hltj'
version = '1.0-SNAPSHOT'

mainClassName = 'me.hltj.java9demo.lombok.LombokDemo'

repositories {
    mavenCentral()
}

dependencies {
    compileOnly "org.projectlombok:lombok:1.16.18"
    compile "ch.qos.logback:logback-classic:1.2.3"
}
```

但是使用 JDK 9 编译会遇到与 Maven 构建时相同的问题，这里采用第二种方案，在 build.gradle 中追加以下配置即可：

```
compileJava {
    options.compilerArgs += ['--add-modules', 'java.xml.ws.annotation']
}
```

> **注意：**目前 Gradle 4.1 到 4.2-RC-2 版无法使用 JDK 9 编译这个项目，这也是本文只验证了 Gradle 3.5-RC-2 到 4.0.2 版的原因。

### 在 IDEA 中使用
如果 IDEA 中已安装 Lombok 插件，并且在项目中开启了注解处理，就能够正常解析 Lombok 注解。但是很遗憾的是，无法直接在 IDEA 中使用 JDK 9 构建：

![](/assets/java9/idea-lombok-err.png)

在 IDEA 中要使用 JDK 9 编译这个项目，需采用将构建与运行委托给 Gradle 的方式：

![](/assets/java9/idea-delegate-to-gradle.png)

这样能够通过 Gradle 来构建，而运行还需要再做几步。首先点击第 15/16 行左侧的运行按钮：

![](/assets/java9/idea-lombok-run.png)

在弹出菜单中选择“Run”，此时“Run”窗口中默认显示的是各个 Gradle 任务的执行情况，需要点击侧栏的按钮将其切换到文本模式才能看到输出：

![](/assets/java9/idea-lombok-result.png)

### 在 Eclipse 中使用
如果已运行过 lombok.jar 为 Eclipse 添加支持，并且在项目配置中启用了注解处理，那么 Eclipse 能够正常解析代码中的大多数 Lombok 注解，只有 `@Value`、`@ToString` 这两行有问题。

在 Eclipse 中要使用 JDK 9 编译这个项目，需要通过 Maven 方式来构建运行，而且还要去除代码中 `@Value` 与  `@ToString` 注解并修改相关联代码：

![](/assets/java9/eclipse-lombok.png)

### 在 NetBeans 中使用
在 Netbeans 每日构建版中即便开启注解处理，仍然只能识别出注解本身，而不能进行相应语义处理，可认为无法解析 Lombok 注解。

在 Netbeans 中要使用 JDK 9 编译这个项目，需要通过 Maven 方式来构建运行：

![](/assets/java9/netbeans-lombok-result.png)

### 使用 Lombok 小结
在 Java 9 环境中使用 Lombok 需要确保编译期有能够提供 `javax.annotation` 的模块可用。
Maven/Gradle 项目使用 JDK 9 编译都不成问题，使用 Gradle 4.1 到 4.2-RC-2 版除外。
三款 IDE 中只有 IDEA 能够良好解析 Lombok 注解，Maven 项目只有每日构建版的 NetBeans 能够顺利构建运行，Gradle 项目只有 IDEA 能够构建运行。

> **注：**示例代码已放到 Github 上： [https://github.com/hltj/java9demo/tree/master/lombok](https://github.com/hltj/java9demo/tree/master/lombok)。

如此看来工具支持还有待提升，那么对于既有项目来说，切换到 Java 9 有收益吗？
当然有！Java 9 不仅提供了新功能，也有很多优化的地方。让我们看一个简单、直观的示例。

## 性能提升的示例
以之前在[由“千万字母表问题”看多范式编程语言](https://hltj.me/lang/2016/11/07/10m-letters.html)中的 Java 代码为例，使用 Java 9 运行的效率相对 Java 8 有显著提升。以下是我分别在不同环境中运行三次取中位数的结果：

<style>
table {
 border-collapse: collapse;
 margin-bottom: 15px;
}
td, th {
 border: 1px grey solid;
 border-spacing: 0;
 padding: 5px;
}
</style>

| 　          | Java 8  | Java 9 |
|-------------|--------:|-------:|
| 某云主机    |  6.495s | 3.706s |
| 某 Dell 本  | 10.580s | 6.829s |
| 某 Thinkpad |  8.206s | 5.889s |

当然这只是说明 Java 9 的 JVM/JRE 在一些场景中相对于 Java 8 有显著的性能提升。而新版 JDK、新的 API、新的工具同样能够带来很多收益，这些会在本系列后续文章中展开。

## 结论
JShell 很适合简短代码的演练。
JDK 9 的 IDE 支持还有待改善，短期内并不适合使用 JDK 9 进行项目开发。
但是无论 JRE 9 的性能提升还是 Java 9 引入的新功能都很值得进一步关注。

本系列后续文章会更多关注 Java 9 的新特性，同时如果发现 IDE 支持有改善也会一并提及，以便大家能够尽早用上。
