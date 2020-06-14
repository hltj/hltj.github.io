---
layout:     post
title:      "追随 Kotlin/Scala，看 Java 12-15 的现代语言特性"
date:       2020-06-14 22:40:03 +0800
categories: java
---
Java 14 发布已经过去了三个月，Java 15 目前也已经到了“Rampdown Phase One ”阶段，其新特性均已敲定。
由于 12-15 都是短期版本，无需考虑也不应该将其用于生产环境。但可以提前了解新特性，以免在下一个 LTS（Java17）正式发布时毫无心理准备。
Java 12-15 引入了一系列改进，本文只讨论语言层面的新特性，它们看起来似曾相识——没错，这些特性让人感觉 Java 在沿 Kotlin/Scala 走过的路线前行。
<!--more-->

虽然不能说 Java 就是借鉴的它们（毕竟这些特性既非它们独有也非它们首创），但可以说是 Java 官方对 Kotlin/Scala 这些特性的充分肯定。而这些新特性也让 Java 向现代语言的方向又迈进了一些，我们逐个来看。

## switch 表达式

> [Java 12 预览](https://openjdk.java.net/jeps/325)、[Java 13 二次预览](https://openjdk.java.net/jeps/354)、[Java 14 正式](https://openjdk.java.net/jeps/361)。

相当于只支持值匹配的 Kotlin [`when` 表达式](https://www.kotlincn.net/docs/reference/control-flow.html#when-%E8%A1%A8%E8%BE%BE%E5%BC%8F)/Scala [`match` 表达式](https://docs.scala-lang.org/overviews/scala-book/match-expressions.html)。

我们看一个不严谨的示例：判断一个非空对象对应哪种  JSON 类型，使用传统的 `switch` 语句实现如下：

``` java
switch (obj.getClass().getSimpleName()) {
    case "Integer":
    case "Long":
    case "Float":
    case "Double":
        System.out.println("number");
        break;
    case "List":
    case "Set":
        System.out.println("array");
        break;
    case "String":
        System.out.println("string");
        break;
    case "Boolean":
        System.out.println("boolean");
        break;
    case "Map":
        System.out.println("object");
        break;
    default:
        System.out.println("object/unknown");
}
```

而使用 `switch` 表达式可以简化为：

``` java
switch (obj.getClass().getSimpleName()) {
    case "Integer", "Long", "Float", "Double" -> System.out.println("number");
    case "List", "Set" -> System.out.println("array");
    case "String" -> System.out.println("string");
    case "Boolean" -> System.out.println("boolean");
    case "Map" -> System.out.println("object");
    default -> System.out.println("object/unknown");
}
```

是不是简洁了很多？当然如果只能这样用，那还称不上 `switch` 表达式，既然是表达式那么一定可以有一个返回值。
我们可以利用 `switch` 表达式的返回值来进一步重构上述代码：

``` java
String jsonType = switch (obj.getClass().getSimpleName()) {
    case "Integer", "Long", "Float", "Double" -> "number";
    case "List", "Set" -> "array";
    case "String" -> "string";
    case "Boolean" -> "boolean";
    case "Map" -> "object";
    default -> "object/unknown";
};
System.out.println(jsonType);
```

`switch` 表达式中箭头的右侧不仅可以是常规表达式，还可以是一个代码块，在块中通过 `yield` 来指定返回值。
例如在上述 `default` 分支打一条错误信息，可以这样改：

``` java
String javaType = obj.getClass().getSimpleName();
String jsonType = switch (javaType) {
    case "Integer", "Long", "Float", "Double" -> "number";
    case "List", "Set" -> "array";
    case "String" -> "string";
    case "Boolean" -> "boolean";
    case "Map" -> "object";
    default -> {
        System.err.println("unknown type: " + javaType);
        yield "object";
    }
};
```

是不是有点类似 Kotlin 的 `when` 与 Scala 的 `match` 了？非常像，只是目前只支持简单的值匹配，还不支持 Kotlin `when` 的 `is`/`in` 以及 Scala `match` 的模式匹配。
当然如果综合 Java 12-15 引入的所有语言特性来看，不难预见 `switch` 表达式未来会支持模式匹配的。

`switch` 表达式的优点不仅是简洁且具有返回值，还避免了传统 `switch` 语句的一些坑（如忘记写 `break` 语句，再如各 `case`/`default` 子句共享同一个局部作用域）。
因此，在 Java 14 及以上版本中，应该尽量采用新语法、避免使用传统的 `switch` 语句。IDEA 甚至会对传统 `switch` 语句标记警告，并且提供了自动将传统语法重构为新语法的 quick fix 功能。

## 文本块

> [Java 13 预览](https://openjdk.java.net/jeps/355)、[Java 14 二次预览](https://openjdk.java.net/jeps/368)、[Java 15 正式](https://openjdk.java.net/jeps/378)。

Java 的文本块（多行字符串）语法与 Kotlin [原始字符串](https://www.kotlincn.net/docs/reference/basic-types.html#%E5%AD%97%E7%AC%A6%E4%B8%B2%E5%AD%97%E9%9D%A2%E5%80%BC)/Scala 多行字符串类似，都是采用三重双引号括起，不过具体语法、语义不尽相同。
Java 文本块起始的三重双引号后只能跟空白符和换行，因此不能像 Kotlin/Scala 那样写 `"""hello"""`，而必须这样写：

``` java
"""
hello"""
```

Java 会自动去掉第一个换行以及每行末尾的空白，因此上述字符串等同于 `"hello"`，但是结尾处三重引号前的换行并不会去掉，例如：

``` java
// 等同于 "hello"
var s1 = """
        hello""";

// 等同于 "hello"
var s2 = """
        hello  """;

// 等同于 "hello\n"
var s3 = """
        hello
        """;
```

文本块中的少于连续三个的双引号都无需转义，这也是文本块的用途之一，例如：

``` java
System.out.println("""                 
        This is a string literal in Java: "Hello".""");
```

这段代码会输出 `This is a string literal in Java: "Hello".`。
编译过程中会自动去掉缩进用的空白符，如果存在多行，会以前导空白最少的一行作为基准。

文本块的另一个用途是便于书写预排版的文本，例如  ASCII Art 或者竖排文字：

``` java
String ci = "┆蝶┆觀┆月┆池┆遊┆　┆獨┆錢┆古┆來┆端┆\n" +
        "┆自┆音┆老┆畔┆人┆　┆賞┆江┆塔┆客┆陽┆\n" +
        "┆舞┆堂┆祠┆問┆醉┆　┆亦┆西┆聽┆尚┆至┆\n" +
        "┆翩┆外┆前┆奇┆　┆　┆悠┆子┆濤┆流┆　┆\n" +
        "┆躚┆雨┆人┆緣┆　┆　┆然┆匯┆於┆連┆　┆\n" +
        "┆　┆連┆絡┆　┆　┆　┆　┆吳┆越┆　┆　┆\n" +
        "┆　┆綿┆繹┆　┆　┆　┆　┆山┆地┆　┆　┆\n";
```

这段竖排文本是我几年前写的[《雙調憶江南·庚寅年端午遊杭州》](https://hltj.me/poetry/2012/04/05/yijiangnan-hangzhou2.html)，可以看到自动格式化后第一行没有与后续几行对齐，虽然还有变通办法，但是这本身就已经比较复杂了。
而如果采用文本块就会简单很多：

``` java
String ci = """
        ┆蝶┆觀┆月┆池┆遊┆　┆獨┆錢┆古┆來┆端┆
        ┆自┆音┆老┆畔┆人┆　┆賞┆江┆塔┆客┆陽┆
        ┆舞┆堂┆祠┆問┆醉┆　┆亦┆西┆聽┆尚┆至┆
        ┆翩┆外┆前┆奇┆　┆　┆悠┆子┆濤┆流┆　┆
        ┆躚┆雨┆人┆緣┆　┆　┆然┆匯┆於┆連┆　┆
        ┆　┆連┆絡┆　┆　┆　┆　┆吳┆越┆　┆　┆
        ┆　┆綿┆繹┆　┆　┆　┆　┆山┆地┆　┆　┆
        """;
```

而既有双引号又有预排版的多行文本就更适合使用文本块了，例如 XML 、JSON 或其他配置/代码文本的字面值：

``` java
var languages = "[\n" +
        "  {\n" +
        "    \"name\": \"kotlin\",\n" +
        "    \"type\": \"static\"\n" +
        "  },\n" +
        "  {\n" +
        "    \"name\": \"julia\",\n" +
        "    \"type\": \"dynamic\"\n" +
        "  }\n" +
        "]";
```

上述 JSON 字面值看起来很凌乱，而用文本块就会清晰很多：

``` java
var languages = """
        [
          {
            "name": "kotlin",
            "type": "static"
          },
          {
            "name": "julia",
            "type": "dynamic"
          }
        ]""";
```

## `instanceof` 模式匹配

> [Java 14 预览](https://openjdk.java.net/jeps/305)、[Java 15 二次预览](https://openjdk.java.net/jeps/375)，预计 Java 16 正式。

类似于 Kotlin 的[智能转换](https://www.kotlincn.net/docs/reference/typecasts.html#%E6%99%BA%E8%83%BD%E8%BD%AC%E6%8D%A2)，但语法不同，在 Scala 中没有直接对应。

传统的 `instanceof` 判断成功之后仍然需要强制转换才能按相应类型使用，例如：

``` java
if (obj instanceof String) {
    System.out.println(((String) obj).length());
}
```

而使用模式匹配之后，可以在判断成功时绑定为一个对应类型的变量，之后直接使用该变量即可：

``` java
if (obj instanceof String s) {
    System.out.println(s.length());
}
```

需要注意的是，只有成功匹配的分支才能绑定该变量：

``` java
if (obj instanceof String s) {
    System.out.println(s.length());
} else {
    // 错误：此处 s 不可用
    // System.out.println(s.length());
}
```

`instanceof` 模式匹配不仅能用于 `if` 语句中，还可以用于 `while` 语句、三目运算符以及 `&&`、`||` 等能使用布尔逻辑的地方，例如：

``` java
int i = obj instanceof String s ? s.length() : -1;
var isEmptyString = obj instanceof String s && s.isEmpty();
var isNotEmptyString = !(obj instanceof String s) || !s.isEmpty();
```

目前 Java 中只引入了这一种非常简单的模式匹配形式，未来应该会引入更多模式匹配语法。

## 记录类型

> [Java 14 预览](https://openjdk.java.net/jeps/359)、[Java 15 二次预览](https://openjdk.java.net/jeps/384)，预计 Java 16 正式。

记录类型（record）类似于 Kotlin 的[数据类（data class）](https://www.kotlincn.net/docs/reference/data-classes.html)与 Scala 的[样例类（case class）](https://docs.scala-lang.org/tour/case-classes.html)，只是更加严格。

在没有记录类型之前，创建一个具有各字段对应 getter、为所有字段初始化的构造函数、基于所有字段的 `equals()`/`hashCode()`/`toString()` 的简单类却需要写一大堆代码，其中大部分都是样板代码。
例如：

``` java
class Font {
    private final String name;
    private final int size;

    public Font(String name, int size) {
        this.name = name;
        this.size = size;
    }

    public String name() {
        return name;
    }

    public int size() {
        return size;
    }

    public boolean equals(Object o) {
        if (!(o instanceof Font other)) return false;
        return other.name.equals(name) && other.size == size;
    }

    public int hashCode() {
        return Objects.hash(name, size);
    }

    public String toString() {
        return String.format("Font[name=%s, size=%d]", name, size);
    }
}
```

上述代码中，除了类名、字段类型与字段名之外，其他的全部都是样板代码。而使用记录只需非常简单的一行代码即可：

``` java
record Font(String name, int size) { }
```

跟一般类相比，记录有以下限制：

- 总是隐式继承自 `java.lang.Record` 而无法显式继承任何任何类
- 记录隐含了 final 并且不能声明为抽象
- 不能显式声明字段，也不能定义初始化块
- 隐式声明的所有字段均为 final
- 如果显式声明任何会隐式生成的成员，其类型必须严格匹配
- 不能声明 native method（通常译为“本地方法”，按说应该叫“原生方法”）

除了这些限制之外，它与普通类一致：

- 用 `new` 实例化
- 可以在顶层声明，也可以在类内部、局部作用域中声明
- 可以声明静态方法与实例方法
- 可以声明静态字段与静态初始化块
- 可以实现接口
- 可以有其内部类型
- 可以标注注解

记录类型还可以与接下来提到的密封类/密封接口很好协作，另外记录还适用于未来版本的模式匹配。

## 密封类与密封接口

> [Java 15 预览](https://openjdk.java.net/jeps/360)，预计 Java 16 二次预览、Java 17 正式。

Java 15 引入的密封类（sealed class）类似于 Kotlin/Scala 的[密封类](https://www.kotlincn.net/docs/reference/sealed-classes.html)、密封接口类似于 Scala 的密封特质（sealed trait）。
不妨将二者统称为密封类型，与普通类/接口不同的是，密封类型限定了哪些类/接口作为其直接子类型。例如：

``` java
sealed interface JvmLanguage
    permits DynamicTypedJvmLanguage, StaticTypedJvmLanguage {}

sealed interface StaticTypedJvmLanguage extends JvmLanguage {}
sealed class DynamicTypedJvmLanguage implements JvmLanguage
    permits Clojure, JvmScriptLanguage {}

final class Java implements StaticTypedJvmLanguage {}
final class Scala implements StaticTypedJvmLanguage {}
final class Kotlin implements StaticTypedJvmLanguage {}

final class Clojure extends DynamicTypedJvmLanguage {}
non-sealed abstract class JvmScriptLanguage extends DynamicTypedJvmLanguage {}

class Groovy extends JvmScriptLanguage {}
```

在密封类型的声明中可以通过 `permits` 显式声明其直接子类型列表，也可以省略——编译器会根据当前文件中的直接子类型的声明推断出来。
一个密封类型的直接子类型必须标注 `sealed`、`non-sealed`、`final` 三者之一。

密封类型与记录是相互独立的功能，但是二者能够很好协作，例如：

``` java
sealed interface Json {}
record NullVal() implements Json {}
record BooleanVal(boolean val) implements Json {}
record NumberVal(double val) implements Json {}
record StringVal(String val) implements Json {}
record ArrayVal(Object...vals) implements Json {}
record ObjectVal(Map<String, Object> kvs) implements Json {}
```

此外，还可以用记录与密封类型来实现[代数数据类型（ADT）](https://en.wikipedia.org/wiki/Algebraic_data_type)：记录为积类型、密封类型为和类型。
与记录类似，密封类型也将适用于未来版本的模式匹配。

## 小结

Java 12-15 引入了 `switch` 表达式、文本块、`instanceof` 模式匹配、记录、密封类型这几个语言新特性，这些特性在 Kotlin/Scala 中基本上都有对应，如同 Java 在追随 Kotlin/Scala 的步伐。

另外，不知大家有没有注意到这一点：除了文本块外，其他几个特性都直接或间接指向了同一个关键词——**模式匹配**。
这些特性除了自身价值之外，也都在为未来版本的模式匹配做铺垫。因此不妨做个大胆预测：在未来的几个版本中，Java 会引入更完善的模式匹配机制。

## 些许遗憾

Java 12-15 中引入语言层面的新特性并不很多，很多令人期待新特性都没有包含在内。
当然语言需要渐进式演化，这也是情理之中的事。唯有两点我觉得有些遗憾：

### 空安全

[安全表达可选值是现代语言的一个特征](https://hltj.me/lang/2019/07/08/modern-lang-optional-value.html)，隔壁 C# 8 引入空安全的经验告诉我们：
**即便语言当初做了错误的设计，假如迷途知返，仍然能够回到正轨**。
Java会不会也有这么一天？也许会，不过 Java 12-15 显然没有，在接下来的几个版本中这么做可能性也很渺茫，也许还会在“迷途”中继续前行很久。

当然关于空安全这块也不是什么长进都没有，Java 14 就有一处：[JEP 358: Helpful NullPointerExceptions](https://openjdk.java.net/jeps/358) 改善了 NullPointerException 所携带的信息，以便定位是哪个变量出现空值导致的。

### 协程

协程即将成为现代工业级语言的标配，不仅 Python、JavaScript、Rust 等语言纷纷引入了协程支持（`async`-`await` 尤为便利），就连隔壁 C++ 也都已将协程纳入了今年的新标准（C++ 20）中。
再对比灵活、强大的 [Kotlin 协程](https://www.kotlincn.net/docs/reference/coroutines/coroutines-guide.html)在异步程序设计中所带来的便利，还在坚守 Java 的开发者对协程的期待就不难理解了。
遗憾的是，Project Loom 未能合入 Java 15，那么按照其他特性的演进周期来看，在下一个 LTS（即 Java 17）发布时就算包含进来也不会是稳定特性。

当然，目前 JVM 平台的 5 门主流工业级语言（Java、Kotlin、Scala、Groovy、Clojure）中，只有 Kotlin 具备上述两个特性，欢迎来 [Kotlin 中文站]([https://www.kotlincn.net/docs/reference/](https://www.kotlincn.net/docs/reference/))学习。
