---
layout:     post
title:      "体验 Java 9（2）：更新与非重大改动"
date:       2018-04-02 18:07:29 +0800
categories: java
---
本篇介绍 Java 9 更新以及一些非重大改动。

## 更新
Java 9 已经正式发布半年多了。这期间不仅 Java 9 发布了更新，就连 Java 10 也已正式发布。[上一篇](https://hltj.me/java/2017/09/22/experience-java9-lombok.html)中提到的工具也都有更新。其中 IDEA 新版改进了不少 Java 9 支持，Eclipse 新版内置了 Java 9 支持（不再需要 Beta 版插件，但可能需要重装，不能直接从旧版升级）。最值得一提的是 lombok 1.16.20 发布。
<!--more-->

### Lombok 兼容性改善
期待已久的 lombok 1.16.20 终于在 2018-01-09 正式发布。这一版解决了[上一篇](https://hltj.me/java/2017/09/22/experience-java9-lombok.html)中提到的大多数兼容问题。具体如下：
  1. 可以与 Gradle 4.1 及以上版本配合使用。
  这点很重要，因为 Gradle 4.2 及以上版本才能支持 Java 9.0.1/9.0.4。
  2. 改进 IDE 支持。
  将上一篇中提到的 [Lombok 示例](https://github.com/hltj/java9demo/tree/master/lombok) 中的 Lombok 版本升级到 1.16.20 后，在新版 IDEA 中无论创建普通项目、Maven 项目还是 Gradle 项目，都能正常编辑与运行；在新版 Eclipse 中，Maven、Gradle 项目正常，普通项目还是有问题；在新版 NetBeans 开发版中普通项目、Maven 项目均正常，Gradle 项目在 JDK 9 环境正常，JDK 9.0.1/9.0.4 环境有问题。

结合一篇的内容，汇总 Lombok 1.16.18 到 1.16.20 兼容情况变化如下表所示：

| &nbsp;                          | Java 9 | Java 9.0.1/9.0.4
|---------------------------------|--------|-----------------
| Maven 3.3.9-3.5.2               | ✓ ⇨ ✓  | ✓ ⇨ ✓
| Gradle 3.5rc-4.0.2              | ✓ ⇨ ✓  | ✗ ⇨ ✗
| Gradle 4.2+                     | ✘ ⇨ ✔  | ✘ ⇨ ✔
| IDEA 2018.1 [普通/Maven/Gradle] | ✘ ⇨ ✔  | ✘ ⇨ ✔
| Eclipse 4.7.3 [Maven/Gradle]    | ✘ ⇨ ✔  | ✘ ⇨ ✔
| Eclipse 4.7.3 [普通]            | ✗ ⇨ ✗  | ✗ ⇨ ✗
| NetBeans 开发版 [普通/Maven]    | ✘ ⇨ ✔  | ✘ ⇨ ✔
| NetBeans 开发版 [Gradle]        | ✘ ⇨ ✔  | ✗ ⇨ ✗

表中箭头（⇨）之前表示 Lombok 1.16.18 对工具的兼容情况，箭头之后表示 Lombok 1.16.20 的兼容情况。

### Java 9 更新与版本号模式
Java 9 的安全更新版 9.0.1、9.0.4 分别于 2017-10-17、2018-01-16 发布。可以看出这里的 Java 版本号与以往 `1.8.0_162` 这种版本号差异很明显。Java 9 的版本号模式（version schema）为：

``` txt
$MAJOR.$MINOR.$SECURITY
```

即“主版本号.次版本号.安全级别”，这样很容易看出 9.0.1 与 9.0.4 都是安全修复版本，应该升级。

Java 9 还增加了用于获取版本号的 API，以前取 Java 版本号主要通过系统属性来取，现在可以直接调用 API 了。`java.lang.Runtime` 新添了一个静态方法 `version()`，其返回值类型为 `Runtime.Version`，通过它可以很方便地获取不同格式的版本号。例如：

``` java
jshell> Runtime.version()
$1 ==> 9.0.4+11

jshell> Runtime.version().version()
$2 ==> [9, 0, 4]

jshell> Runtime.version().major()
$3 ==> 9

jshell> Runtime.version().security()
$4 ==> 4

jshell> Runtime.version().build()
$5 ==> Optional[11]
```

值得补充的是，Java 10 再次修改版本号模式为：

``` txt
$FEATURE.$INTERIM.$UPDATE.$EMERG
```

即“特性版本号.中间版本号.更新版本号.紧急修复版本号”。 同时 `Runtime.Version` 的相应方法 `major()`、`minor()`、`security()` 也被弃用，建议分别使用新增的 `feature()`、`interim()`、`update()` 方法取代，另外新增 `patch()` 方法用于取紧急修复版本号。例如：

``` java
jshell> Runtime.version()
$1 ==> 10+46

jshell> Runtime.version().feature()
$2 ==> 10

jshell> Runtime.version().build()
$3 ==> Optional[46]
```

当然 Java 10 与 Java 9 的前三位版本号的含义并不完全相同。特别是自 Java 10 起将严格按照 6 个月的节奏发版，即今年三月份发布 Java 10、九月份发布 Java 11、明年三月份发布 Java 12……以此类推。

关于 Java 9 与 Java 10 版本号模式的更多内容请参见 [JEP 223: New Version-String Scheme](http://openjdk.java.net/jeps/223) 以及 [JEP 322: Time-Based Release Versioning](http://openjdk.java.net/jeps/322)。

## 非重大改动

接下来介绍以下内容：

- 下划线成为保留字
- try-with-resource 改进
- InputStream 改进
- 接口的私有默认方法
- 集合的工厂方法
- Optional 改进
- Stream 改进
- 进程 API 改进

如果已经了解过就不需要再往下看了。

### 下划线成为保留字
单独一个下划线（`_`）作为标识符在 Java 8 中就已经弃用，但是仍然可以通过编译。例如下述代码：

``` java
public class Demo0 {
    public static void main(String[] args) {
        int _ = 1;
        System.out.println(_);
    }
}
```

在 Java 8 环境中编译会出现以下警告：

```  bash
$ javac Demo0.java 
Demo0.java:3: warning: '_' used as an identifier
        int _ = 1;
            ^
  (use of '_' as an identifier might not be supported in releases after Java SE 8)
Demo0.java:4: warning: '_' used as an identifier
        System.out.println(_);
                           ^
  (use of '_' as an identifier might not be supported in releases after Java SE 8)
2 warnings
$ java Demo0
1
```

而在 Java 9 环境中则不能通过编译：

``` bash
$ javac Demo0.java
Demo0.java:3: error: as of release 9, '_' is a keyword, and may not be used as an identifier
        int _ = 1;
            ^
Demo0.java:4: error: as of release 9, '_' is a keyword, and may not be used as an identifier
        System.out.println(_);
                           ^
2 errors
```

在 JShell 中使用也会出现同样的报错：

``` java
jshell> String _ = "foo";
|  Error:
|  as of release 9, '_' is a keyword, and may not be used as an identifier
|  String _ = "foo";
|         ^
```

这是因为 `_` 已经成为保留字，在未来的 Java 版本中有可能用作占位符，类似 Scala 的模式匹配语法或者 Kotlin 的解构语法中的占位符。

### try-with-resource 改进

Java 7 引入了 try-with-resource 语法，对于支持自动清理的资源（实现了 `AutoCloseable` 接口的类型），将引用创建的语句放在 try 与左花括号之间圆括号中，就可以执行自动清理。例如

``` java
try (BufferedReader br = new BufferedReader(new FileReader(path))) {
    return br.readLine();
}
```

对于既有引用，在 Java 7/8 中需要引入临时变量，例如：

``` java
BufferedReader br = new BufferedReader(new FileReader(path));
try (BufferedReader br1 = br) {
    return br1.readLine();
}
```

而在 Java 9 中，使用既有 final（或相当于 final）的可自动清理资源引用时无需引入临时变量。上述代码可以简化为：

``` java
BufferedReader br = new BufferedReader(new FileReader(path));
try (br) {
    return br.readLine();
}
```
### `InputStream` 改进
Java 9 的 `InputStream` 新添了一个 `transferTo()` 方法，可以从输入流读取并写到输出流。例如，在 JShell 中用一行代码实现终端输入回显（按 Ctrl-C 结束）：

``` java
jshell> System.in.transferTo(System.out)
Hello
Hello
0123456789
0123456789

$1 ==>
```

有了这个函数，实现类似 `IOUtils.toString()` 的逻辑就很方便了：

``` java
jshell> String inputToString(InputStream is) throws IOException {
   ...>    ByteArrayOutputStream os = new ByteArrayOutputStream();
   ...>    is.transferTo(os);
   ...>    return os.toString();
   ...> }
|  created method inputToString(InputStream)

jshell> inputToString(new ByteArrayInputStream("Hello".getBytes()))
$2 ==> "Hello"
```

### 接口的私有默认方法
Java 8 引入了接口的默认方法，可以为接口提供默认实现。例如我们有一个接口 `Person` 以及很多实现该接口的类，当我们为 `Person` 接口新增 `greeting()`、`farewell()` 两个方法时，可以为其提供默认实现，而无需每个实现 `Person` 接口的类都实现一遍：

``` java
public interface Person {
    // 其他接口 ……

    String getName();

    default void greeting() {
        System.out.println(getName() + ": Hello!");
    }

    default void farewell() {
        System.out.println(getName() + ": Goodbye!");
    }
}
```

Java 9 的接口支持私有默认方法，可以让多个默认方法复用代码：

``` java
public interface Person {
    // 其他接口 ……

    String getName();

    default void greeting() {
        say("Hello!");
    }

    default void farewell() {
        say("Goodbye!");
    }

    private void say(String words) {
        System.out.println(getName() + ": " + words);
    }
}
```

假如这里的私有默认方法 `say()` 并没有调用实例方法 `getName()`，那么还可以使用静态私有方法：

``` java
public interface Person {
    // 其他接口 ……

    default void greeting() {
        say("Hello!");
    }

    default void farewell() {
        say("Goodbye!");
    }

    private static void say(String words) {
        System.out.println(words);
    }
}
```

### 集合的工厂方法
在 Java 9 之前，创建带有初始字面值的只读 List、Set 与 Map 的代码语法风格各异，甚至有些繁琐麻烦。例如：

``` java
// import static java.util.Collections.unmodifiableSet;
// import static java.util.Collections.unmodifiableMap;
// import static java.util.stream.Collectors.collectingAndThen;
// import static java.util.stream.Collectors.toSet
// import static java.util.stream.Collectors.toMap
// import static java.util.AbstractMap.SimpleEntry;

final List<String> list = Arrays.asList("foo", "bar", "baz");

final Set<Integer> numbers = unmodifiableSet(new HashSet<Integer>() {
    {
        add(1); add(2); add(3);
    }
});

// 或者
final Set<Integer> numbers1 =
    unmodifiableSet(new HashSet<Integer>(Arrays.asList(1, 2, 3)));


// 或者（Java 8）
final Set<Integer> numbers2 =
Stream.of(1, 2, 3)
    .collect(collectingAndThen(toSet(), Collections::unmodifiableSet));

final Map<Integer, String> map =
    unmodifiableMap(new HashMap<Integer, String>() {
        {
            put(1, "foo");
            put(2, "bar");
            put(3, "baz");
        }
    });

// 或者（Java 8）
final Map<Integer, String> map1 =
    unmodifiableMap(Stream.of(
        new SimpleEntry<>(1, "foo"),
        new SimpleEntry<>(2, "bar"),
        new SimpleEntry<>(3, "baz"))
            .collect(toMap((e) -> e.getKey(), (e) -> e.getValue()))
    );
```

而在 Java 9 中可以使用更简洁、更直观、更统一的方式：

``` java
// import static java.util.Map.entry

final List<String> list = List.of("foo", "bar", "baz");

final Set<Integer> numbers = Set.of(1, 2, 3);

final Map<Integer, String> map = Map.of(1, "foo", 2, "bar", 3, "barz");

// 或者
final Map<Integer, String> map1 = Map.ofEntries(
    entry(1, "foo"),
    entry(2, "bar"),
    entry(3, "baz")
);
```

### Optional 改进

**新增 `or` 方法**

`Optional` 类新增 `or()` 方法，它与 `orElseGet()` 类似，只是其参数 lambda 的返回值以及自身的返回值都是 `Optional`，可以用于从一系列可选值中选取第一个有值的。例如：

``` java
jshell> import static java.util.Optional.*;

jshell> empty().or(() -> empty()).or(() -> empty())
$2 ==> Optional.empty

jshell> empty().or(() -> empty()).or(() -> of("Hello")).or(() -> empty())
$3 ==> Optional[Hello]

jshell> empty().or(() -> of("Wolrd")).or(() -> of("Hello")).or(() -> empty())
$4 ==> Optional[Wolrd]
```

> 注：只有 `Optional` 类新添了这个方法，而 `OptionalInt` 等可选类并未添加。

**新增 `ifPresentOrElse` 方法**

在 Java 8 中可选类都实现了 `ifPresent`，可以很容易实现有值时输出其内容：

``` java
jshell> OptionalDouble.of(Math.PI).ifPresent(System.out::println)
3.141592653589793
```

但如果要同时实现无值时输出默认值，它就无能为力了。这时 Java 9 的 `ifPresentOrElse` 就派上用场了：

``` java
jshell> OptionalDouble.of(Math.PI).ifPresentOrElse(System.out::println, () -> System.out.println("No Value"))
3.141592653589793

jshell> OptionalDouble.empty().ifPresentOrElse(System.out::println, () -> System.out.println("No Value"))
No Value
```

**新增 `stream` 方法**

在 Java 8 中如果想对 `OptionalInt` 等基本类型可选值做流式集合处理会非常麻烦：

``` java
jshell> OptionalLong optlong = OptionalLong.of(1L)
optlong ==> OptionalLong[1]

jshell> (optlong.isPresent()? LongStream.of(optlong.getAsLong()): LongStream.empty()).map(x ->x + 1).boxed().collect(Collectors.toList())
$2 ==> [2]
```

Java 9 添加了 `stream()` 方法可使其简化很多：

``` java
jshell> optlong.stream().map(x ->x + 1).boxed().collect(Collectors.toList())
$3 ==> [2]
```

### Stream 类改进

**新增 `ofNullable` 工厂方法**

在 Java 9 中如需对可空对象进行流式集合处理，可以借助 `Optional` 来实现：

``` java
jshell> Optional.ofNullable("hello").stream().count()
$1 ==> 1

jshell> Optional.ofNullable(null).stream().count()
$2 ==> 0
```

另外，Java 9 也为 `Stream` 类添加了 `ofNullable()` 方法，上述代码还可以简化为：

``` java
jshell> Stream.ofNullable("hello").count()
$3 ==> 1

jshell> Stream.ofNullable(null).count()
$4 ==> 0
```

> 注：只有 `Stream` 类新添了这个方法，而 `IntStream` 等并未添加。

**新增 `takeWhile` 与 `dropWhile` 方法**

分别用于“取元素直到指定值出现”、“直到指定值出现开始取元素”，例如：

``` java
jshell> import static java.util.stream.Collectors.toList;

jshell> IntStream.of(9, 6, 3, 0, 2, 4, 6).takeWhile(n -> n != 0).boxed().collect(toList())
$2 ==> [9, 6, 3]

jshell> IntStream.of(9, 6, 3, 0, 2, 4, 6).dropWhile(n -> n != 0).boxed().collect(toList())
$3 ==> [0, 2, 4, 6]
```

**`iterate` 方法新增重载**

使用 Java 8 的 `iterate` 以及上述 `takeWhile` 实现输出 10 以内的偶数如下：

``` java
jshell> IntStream.iterate(0, i -> i + 2).takeWhile(i -> i <= 10).forEach(System.out::println)
0
2
4
6
8
10
```

不过 Java 9 还为 `iterate` 新添了支持判断迭代条件的重载形式，因此上述代码可改为：

``` java
jshell> IntStream.iterate(0, i -> i <= 10, i -> i + 2).forEach(System.out::println)
0
2
4
6
8
10
```
### 进程 API 改进
Java 9 新增了 `ProcessHandle` 接口，可以很方便地获得进程的 pid 与进程信息。`ProcessHandle` 还提供了两个静态方法，分别用于获取当前进程与指定 pid 的 `ProcessHandle` 实例。例如：

``` java
jshell> ProcessHandle.current().pid()
$1 ==> 27137

jshell> ProcessHandle.current().parent()
$2 ==> Optional[27112]

jshell> ProcessHandle.of(27112).get().children().count()
$3 ==> 1

jshell> ProcessHandle.current().info().command()
$4 ==> Optional[/usr/local/jdk-9.0.4/bin/java]

jshell> ProcessHandle.current().info().totalCpuDuration()
$5 ==> Optional[PT0.81S]
```

关于这些新特性的更多细节请参见官方文档。

本系列未完待续。
