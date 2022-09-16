---
layout:     post
title:      "【灰蓝 Java 训练】如何处理空值"
date:       2021-01-09 19:06:55 +0800
categories: java
---

> 这个系列以练习为主，可能不会有多少讲述（当然本篇例外），可以作为初学者的自学验收之用。

Java 中有非受限的空值，并且不知哪时会引发 NPE（即 `NullPointerException`），解决这个问题对于 Android 开发来说很简单——用 Kotlin 就好了。
其实不仅限于 Android，对于服务端开发来说终极方案也应该是迁移到 Kotlin。
因为只要用 Java，空值问题就没办法彻底解决（之前在[《现代编程语言系列2：安全表达可选值》](https://hltj.me/lang/2019/07/08/modern-lang-optional-value.html)中也提到过这点），而 JVM 平台主流工业级语言中只有 Kotlin 很好地解决了这一问题。

但是对于服务端开发来说，常有各种非技术原因不能在项目中以 Kotlin 取代 Java，对于这些项目来说显然没办法彻底解决空值问题。
那么有没有一些方法与工具可以让空值问题处理起来尽可能规范、简易些呢？这里有几点经验分享。
<!--more-->

## NPE 防御
一些典型场景的 NPE 可以通过编码习惯来防御——某些静态分析工具或许也能检测到一些问题，但很难完美覆盖；
没办法，只能通过编码规范、程序员的自律来解决了。
其中比较常见的两个场景是空值比较以及使用不可变集合。

### 空值比较
对实际值为 `null` 的变量调用包括 `equals()` 在内的任何方法都会导致 NPE。
因此比较可空值（通常为变量）与非空值（通常为常量，不尽然）时，以可空值为参数对非空值调 `equals()` 即可避免这个问题。
例如：

``` java
if ("Hello".equals(nullableStr)) {
    ……
}
```

如果比较两个可空值怎么办呢？用 `Objects.equals()`，例如：

``` java
if (Objects.equals(nullableObj1, nullableObj2)) {
    ……
}
```

### 不可变集合不支持空值
Java 9 引入的 `List.of()`、`Set.of()`、`Map.of()`、`Map.entry()` 以及 Java 10 引入的 `List.copyOf()`、`Set.copyOf()`、`Map.copyOf()`、`Collectors.toUnmodifiableList()`、`Collectors.toUnmodifiableSet()`、`Collectors.toUnmodifiableMap()` 等均不支持 `null`，其中构造不可变的 `Map` 与 `Map.Entry` 时 key、value 均不能为 `null`。
还需要注意的一点是不能以 `null` 值调用不可变集合的 `contains()` / `containsKey()` / `containsValue()` / `containsAll()` 方法，其中 `containsAll()` 还要求参数集合中不能有 `null`。

例如：

``` java
List.of("a", null); // NPE：元素不可以有空值
Map.of("a", 1, null, 2); // NPE：key 不可有空值
Map.of("a", 1, "b", null); // NPE：value 不可有空值
Map.entry("hello", null); // NPE：value 不可有空值

var map = new HashMap<String, Integer>();
map.put("a", 1);
Map.copyOf(map); // OK
map.put(null, 2);
Map.copyOf(map); // NPE：key 不可有空值

// NPE：元素不可以有空值
Stream.of((String)null).collect(Collectors.toUnmodifiableList()); 

// NPE：不能用 null 调用不可变集合的 containsKey() 方法
Map.of("a", "b").containsKey(null);
```

千万不要因为这点而放弃不可变集合。
不可变集合本身有很多优势，Java 10 及以后版本也推荐使用不可变集合。
只是需要特别注意上述几点：构造字面值时不能有 `null`、调用 `contains()` / `containsKey()` 等之前需要判断参数是否为 `null`、调用 `collect()` / `copyOf()` / `containsAll()` 之前去除参数集合中的 `null` 值。
例如：

``` java
var isInSet = nullableStr != null && Set.of("hello", "world").contains(nullableStr);

Stream.of("hello", (String)null, "world").filter(Objects::nonNull).collect(Collectors.toList());

var map = new HashMap<String, Integer>();
map.put("a", 1);
map.put(null, 2);
map.put("b", null);

map.entrySet().stream()
        .filter(entry -> entry.getKey() != null && entry.getValue() != null)
        .collect(Collectors.toUnmodifiableMap(Map.Entry::getKey, Map.Entry::getValue));
```

## Optional
Java 中解决可选值问题，首先应该想到的就是 [`Optional`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html)。
`Optional` 是 Java 8 引入的可选值类型，用于在很多场景中取代 `null` 来表达可选值，进而避免 `null` 所带来问题。

### `Optional` 的用法
`Optional` 的核心方法有 [`map()`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#map(java.util.function.Function))、[`flatMap()`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#flatMap(java.util.function.Function))、[`orElse()`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElse(T)) 与 [`orElseGet()`](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#orElseGet(java.util.function.Supplier))。

**`map()`：由一个可选值得到另一个可选值**

例如对于一个可选的字符串，计算其长度：

``` java
var lengthOpt = Optional.ofNullable(nullableString).map(String::length);
```

**`flatMap()`：处理可选值嵌套情况**

例如，从可空整数列表中取第一个正值，从列表中取第一个元素可以用 `Stream<Integer>#findFirst()`，该方法返回 `Optional<Integer>`，如果继续用 `map()` 的话，会得到可选值的可选值：

``` java
Optional<Optional<Integer>> intOptOpt = Optional.ofNullable(nullableIntList).map(intList ->
        intList.stream().filter(i -> i > 0).findFirst()
);
```

而用 `flatMap()` 会将两层 `Optional` 打平为一层：

``` java
Optional<Integer> intOpt = Optional.ofNullable(nullableIntList).flatMap(intList ->
        intList.stream().filter(i -> i > 0).findFirst()
);
```

**`orElse()`、`orElseGet()`：由可选值得到值，如果无值取默认值**

例如，对于可选字符串，有值取长度，无值取 `0`，可以用 `orElse()` 得到一个整数：

``` java
int length = strOpt.map(String::length).orElse(0);
```

如果默认值需要惰性求值，那么还可以用 `orElseGet()`：

``` java
int lengthOrRandom = strOpt.map(String::length).orElseGet(() ->
        new Random().nextInt()
);
```

还有一个特别的场景，就是将 `Optional<T>` 转换为可空的 `T` 值时千万不能想当然地调 `get()`——它在无值时抛 `NoSuchElementException` 而不是返回 `null`（参见其[文档](https://docs.oracle.com/en/java/javase/11/docs/api/java.base/java/util/Optional.html#get())）。
正确的方式是 `orElse(null)`。例如：

``` java
Integer lengthOrNull = strOpt.map(String::length).orElse(null);
```

`Optional` 的其他方法在特定场景也很实用，由于方法不多，大家可以直接读其文档，在此不再赘述。

### `Optional` 的问题
很多文章称 `Optional` 是 Java 8 针对 NPE 问题甚至[十亿美元问题](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)的解决方案，实际上远非如此。且不说在 Java 中没办法强制使用 `Optional` 而不用 `null`，即便 `Optional` 自身用起来也有很多局限性。

**`Optional` 对象自身也可能为 `null`**

一个特别坑的问题是 `Optional` 是引用类型，因此一个 `Optional` 对象自身也可能为 `null`，例如：

``` java
Optional<String> strOpt = null;
strOpt.map(String::length); // NPE
```

当然 `Optional` 的官方文档称 `Optional` 类型值不应该为 `null`，在 IDEA 中写上述代码也会标出警告，但 Java 语法与编译器并不会为此提供任何保障或约束。

**`Optional` 不可序列化**

`Optional` 不可序列化，这就意味着需要序列化的场景还得用 `null` 来表达可选值，这也是上文特别提到由 `Optional` 转换可空值的主要原因。
鉴于此，`Optional` 不适合作为类的字段，也不适合作为方法的入参，只适用于局部变量、返回值以及表达式中间结果等场景。
IDEA 也会对 `Optional` 用作字段或者参数时标记警告。

**不够简洁**

与受限空值语法相比，`Optional` 用法要冗长的多，当然这很符合 Java 的历史风格。
对于受限空值的 `?.` 语法，`Optional` 要用 `map()` 与 `flatMap()`；对于 `?:`/`??` 语法要用 `orElse()` 或 `orElseGet()`。
其实对于采用可选值类型的其他语言来说可能都有类似问题，但是 `Haskell`、`Scala` 有推导式，`Haskell`、`Scala`、`OCaml`、`F#` 等语言还支持自定义操作符，从而简化可选值的用法。
不幸的是 Java 不具备这些语法。

## 可空性注解
在 Java 中没办法强制区分可空与非空类型，而有时又希望能在字段、参数或者返回值上标注是否可空，以便 IDE 或者其他静态检查工具能够识别并给出提醒。

我们看个示例，实现一个简陋版的 `?:`/`??`：

``` java
public static <T> T defaultWith(T obj, T defaultVal) {
    return Optional.of(obj).orElse(defaultVal);
}
```

当 `obj` 非空时该方法返回 `obj`，否则返回 `defaultVal`。
但是实际上事与愿违，这个方法在 `obj` 为空时不会返回 `defaultVal`，而是抛 NPE。
原因是 `Optional.of()` 只接受非空参数，如果参数为空就会抛 NPE。
遗憾的是编写与编译这段代码都不会收到任何警告或提醒，因为无论 IDE 还是编译器都无从知晓这个方法的入参 `obj` 可能为空。

此时，如果我们给 `obj` 加一个 `org.jetbrains.annotations.Nullable` 注解，IDEA 就会对方法中使用 `obj` 调用 `Optional.of()` 标出警告提醒：“Argument 'obj' might be null”。
改为调用 `Optional.ofNullable()` 警告就消失了。
而如果希望 `defaultVal` 要求非空的话，还可以对 `defaultVal` 标注 `org.jetbrains.annotations.NotNull`，这样一来方法的返回值也会非空，同样可以标注 `NotNull`：

``` java
@NotNull
public static <T> T defaultWith(@Nullable T obj, @NotNull T defaultVal) {
    return Optional.ofNullable(obj).orElse(defaultVal);
}
```

当然，这只是关于可空性注解使用场景的一个示例，实际上并不需要我们自己造一个这样的轮子，因为有现成的轮子可用（见下文）。
上述 `Nullable` 与 `NotNull` 两个注解来自于 [JetBrains 的 java-annotations](https://www.jetbrains.com/help/idea/nullable-and-notnull-annotations.html)。类似的还有 [Checker Framework 的 Nullness Checker](https://checkerframework.org/manual/#nullness-checker)、[spotbugs-annotations](https://javadoc.io/doc/com.github.spotbugs/spotbugs-annotations/latest/index.html)、[Lombok 的 NonNull](https://projectlombok.org/features/NonNull) 等。

## Apache Commons
[Apache Commons](https://commons.apache.org/) 有多个库提供了简化空值使用的工具。例如实现类似 `?:`/`??` 的功能，使用 `Optional` 需要这样写：

``` java
var nonNullVal = Optional.ofNullable(obj).orElse(nonNullDefault);
```

而使用 Apache Commons 只需调用一个类似上文实现的静态方法就可以了。

### StringUtils
[`StringUtils`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html) 是 [Commons Lang](https://commons.apache.org/proper/commons-lang/) 中的一个工具类，其中包含一系列与字符串空值相关的方法。

**[`isEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isEmpty-java.lang.CharSequence-) 与 [`isNotEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isNotEmpty-java.lang.CharSequence-)**

前者判断一个字符序列是否为 `null` 或空序列，后者与之相反——判断一个字符序列既非 `null` 也非空序列。
这两个方法非常实用，现实业务中对字符串为 `null` 或空串时走同样处理分支的场景很常见。

**[`defaultString()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#defaultString-java.lang.String-java.lang.String-)**

`defaultString(str, defaultStr)` 如果 `str` 非 `null` 返回 `str`，否则返回 `defaultStr`。
还有一个单参重载版本：`defaultString(str)` 如果非 `null` 返回 `str`，否则返回空串。

**[`defaultIfEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#defaultIfEmpty-T-T-)**

`defaultIfEmpty(str, defaultStr)` 如果 `str` 既非 `null` 也非空串返回 `str`，否则返回 `defaultStr`。

**[`getIfEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#getIfEmpty-T-java.util.function.Supplier-)**

相当于默认值采用惰性求值的 `defaultIfEmpty()`，`getIfEmpty(str, () -> ……)` 当 `str` 既非 `null` 也非空串时返回 `str`，否则对 lambda 表达式求值并以其返回值作为 `getIfEmpty()` 的返回值。

**[`firstNonEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#firstNonEmpty-T...-)**

对于一系列值，返回第一个既非 `null` 也非空串的。

**[`isAllEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isAllEmpty-java.lang.CharSequence...-)、[`isAnyEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isAnyEmpty-java.lang.CharSequence...-)、[`isNoneEmpty()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/StringUtils.html#isNoneEmpty-java.lang.CharSequence...-)**

对于一系列值，判断是否**全都是**、**其中有**、**全都不是** `null` 或空串。

除了这些明显与空值相关的方法外，`StringUtils` 的其他方法也都会对 `null` 特殊处理而不是引发 NPE（个别的会抛其他异常）。

### Commons Collections
与 `StringUtils` 类似，[Commons Collections](https://commons.apache.org/proper/commons-collections/) 中的 `CollectionUtils`、`IterableUtils`、`ListUtils`、`SetUtils`、`MapUtils` 等也有提供 `null` 与空集合合并处理的方法，只是没有那么丰富。

**`emptyIfNull()`**

`CollectionUtils`、`IterableUtils`、`ListUtils`、`SetUtils`、`MapUtils` 等均有提供该方法，如果参数非 `null` 返回参数本身，否则返回对应类型的空集合。

**`isEmpty()` 与 `isNotEmpty()`**

`CollectionUtils`（可以用于 `Collection` 及其子类型如 `List`、`Set`） 与 `MapUtils` 提供了这两个方法。前者判断是否为 `null` 或为空集合，后者相反——判断既非 `null` 也非空集合。

`CollectionUtils`、`IterableUtils`、`ListUtils`、`SetUtils`、`MapUtils` 等提供的大多数其他方法也都会对 `null` 特殊处理而不是引发 NPE，个别会抛 NPE 的方法文档中也有说明。

### ObjectUtils
更通用的情况还可以用 Commons Lang 中的工具类 [`ObjectUtils`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html)。

**[`defaultIfNull()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#defaultIfNull-T-T-)**

类似上文自行实现的 `defaultWith()`，不过并没有标可空性注解。
`defaultIfNull(obj, defaultVal)` 当 `obj` 非空时返回 `obj` 否则返回 `defaultVal`。

**[`getIfNull()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#getIfNull-T-java.util.function.Supplier-)**

相当于默认值采用惰性求值的 `defaultIfNull()`，`getIfNull(obj, () -> ……)` 当 `obj` 非空时返回 `obj`，否则对 lambda 表达式求值并以其返回值作为 `getIfNull()` 的返回值。

**[`firstNonNull()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#firstNonNull-T...-)**

对于一系列值，返回第一个非空的。

**[`getFirstNonNull()`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#getFirstNonNull-java.util.function.Supplier...-)**

可以看作是惰性求值版的 `firstNonNull()`，对于一系列求值过程返回第一个求值结果非空的结果。
例如 `getFirstNonNull(() -> null, () -> "hello", () -> throw new IllegalStateException())` 会返回 `"hello"` 而不会执行后面的求值过程，因此不会抛 `IllegalStateException`。

**[`allNotNull`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#allNotNull-java.lang.Object...-)、[`allNull`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#allNull-java.lang.Object...-)、[`anyNotNull`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#anyNotNull-java.lang.Object...-)、[`anyNull`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/ObjectUtils.html#anyNull-java.lang.Object...-)**

对于一系列值，判断是否**全都非**、**全都是**、**其中有非**、**其中有**空值。

`ObjectUtils` 中的大多数其他方法也都会对空值特殊处理而不是引发 NPE。除了 `StringUtils`、`ObjectUtils` 之外，Commons Lang 中的 [`BooleanUtils`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/BooleanUtils.html)、[`NumberUtils`](https://commons.apache.org/proper/commons-lang/apidocs/org/apache/commons/lang3/math/NumberUtils.html) 也都提供了一系列空安全的工具方法。

## 其他
抛砖引玉，欢迎补充。

## 小结
Java 语言自身目前没办法彻底解决空值问题，不过有一些方法、工具可以用：
- NPE 防御：空值比较、不可变集合不支持空值
- `Optional`
- 可空性注解
- Apache Commons

光说不练无异于纸上谈兵，接下来的练习才是重中之重。

## 练习
以下练习中未标注解的变量，如果变量名以 `x` 开头也表示可能为空。
> 为什么不直接用 `nullableXyz` 这种更明显的方式？因为现实代码中通常更不明显。

### 1、 纠错

``` java
if (xMethod.equals("POST")) {
   doPost();
}

if (xArg1.equals(xArg2)) {
    System.out.println("arg1 == arg2");
}
```

### 2、纠错

``` java
var map1 = Map.of("abc", 10, "def", 20, xStr, 30);

var list1 = Arrays.asList(1, 2, -3, 9, null, 15);
var set1 = Set.copyOf(list1);

if (Map.of("hello", 1, "world", 2).containsKey(xStr)) {
    System.out.println("either 'hello' or 'world'");
}
```

### 3、加注可空性注解

``` java
public static <T, U> U mapSome(T x, Function<T, U> mapper) {
    return x == null ? null: mapper.apply(x);
}
```

### 4、用 `Optional` 重构上题 `mapSome()`
> 注：只是练习 `Optional` 的使用，上题的实现并不需要以 `Optional` 取代。

### 5、用 `Optional` 重构

``` java
Integer xInt = Math.random() > 0.8 ? null : Math.random() > 0.5 ? 5 : 12;

// 重构以下代码
var x1 = (xInt == null || xInt % 2 != 0) ? null : xInt / 2;
if (x1 != null) {
    System.out.println(x1);
}
```

### 6、用 `Optional` 重构 `getTitledContent()`

``` java
@NotNull
public static String getUpperTitle(@Nullable Post post) {
    if (post == null || post.getTitle() == null) {
        log.warning("no title")
        return "- UNTITLED -";
    }

    return post.getTitle().toUpperCase();
}

public class Post {
    @Nullable
    public String getTitle();
}
```

### 7、用 `StringUtils` 将 `getChoice()` 的实现重构成一行代码

``` java
String getChoice(String choice, boolean highest) {
    if (choice != null && !choice.isEmpty())
        return choice;

    if (highest)
        return "High";

    return "Low";
}
```

### 8、使用 Common Collections 重构 `getIdsString()`

``` java
private static final List<Integer> IMPLICIT_IDS = List.of(101, 111, 191);
public static String getIdsString(@Nullable Collection<@NotNull Integer> ids) {
    if (ids == null) {
        return IMPLICIT_IDS.stream()
                .map(Object::toString)
                .collect(Collectors.joining());
    }

    return Stream.concat(ids.stream(), IMPLICIT_IDS.stream())
            .map(Object::toString)
            .collect(Collectors.joining());
}
```

### 9、使用 `BooleanUtils` 重构

``` java
public static String toHex(int n, @Nullable Boolean useUpper) {
    String s = Integer.toString(n, 16);
    return useUpper != null && useUpper ? s.toUpperCase() : s;
}
```

### 10、使用 `ObjectUtils` 重构

``` java
public static Instant tomorrowOf(@Nullable Instant x) {
    if (x == null) {
        log.debug("the base Date is null");
        x = Instant.now();
    }
    
    return x.plus(Duration.ofDays(1));
}
```
