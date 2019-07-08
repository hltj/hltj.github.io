---
layout:     post
title:      "现代编程语言系列2：安全表达可选值"
date:       2019-07-08 13:15:16 +0800
categories: lang
---
这里的可选值（optional value）是指可能无值也可能有一个值的情况，在一些编程语言中称为可空值（nullable value）。


## 问题与解决方案
传统编程语言中往往使用空值（`null` 或者 `None`、`nil` 等）来表达可选值，可谓简单粗暴。
<!--more-->
因为这样一来，就需要在每一处使用的地方判断相应的值是否为空，一旦疏忽大意就可能导致程序出错甚至崩溃。
不仅如此，正如著名的[《十亿美元的错误》](https://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)与[《计算机科学中的最严重错误》](·https://www.lucidchart.com/techblog/2015/08/31/the-worst-mistake-of-computer-science/)所说，传统空值还引入了一系列其他问题：破坏了类型系统、易与空容器混为一谈、表意模凌两可、难以调试、不便同语言的其他特性结合使用等。

因此，现代编程语言基本都会避免使用传统空值，而采用更安全的方式来表达可选值，具体方式主要有两种：
  1. 受限的空值：Kotlin、Swift、Hack 等。
  2. 可选值类型：Haskell、Rust、Julia、OCaml、Swift、<em style="color:gray">F#、Scala、Java 8+、C++17+</em> 等。

你没看错，Swift 在两边都站队了。这倒并不是它采用了两种不同的机制，相反，它在一致的底层机制基础上，同时兼容两种上层语法。

另外 F#、Scala、Java 8+、C++ 17+ 实际上处于灰色地带，它们虽然推荐使用可选值类型，却也支持传统空值。

> 值得一提的是，无论采用哪种方式，其安全性都是由类型系统来保障的。
虽然这并不仅限于静态类型语言（Hack 与 Julia 都是动态类型语言），不过确实需要一定程度的静态类型支持，这也从侧面反映了[现代编程语言的静态类型趋势](https://hltj.me/lang/2017/08/01/morden-lang-static-type.html)。

## 历史包袱
采用受限空值的编程语言与处于灰色地带的编程语言一般都存在历史包袱。

采用受限空值的语言可能都与历史包袱有关：Kotlin 中有 `null` 因为 Java 中有，Hack 中有 `null` 因为 PHP 中有，Swift 中有 `nil` 因为 Objective-C 中有。

处于灰色地带的 F#、Scala、Java 8+、C++ 17+ 同样有历史包袱，因为需要与 .Net/JVM 平台的其他语言互操作或者要兼容本语言的旧版本。

但是两队语言做了不同的抉择：一队采用受限的空值取代了传统空值；另一队引入了可选值类型的同时，却还保留传统空值。
于是后面这队语言虽有安全的方式，却也无法摆脱传统空值的纠缠。
Java 8 与 C++ 17 为了兼容历史版本或是无奈之选，但是如果历史重新给 F# 与 Scala 选择机会的话，会不会采用 Swift 的方案更好一些？

## 受限的空值
在采用受限的空值来表达可选值的编程语言中，对空值的使用有以下限制：
  1. 语言中严格区分可空类型与非空类型，不能直接将可空值用于只接受非空值的地方。
  2. 语言中通过特定语法访问可空类型对象的成员，也需要特定语法由可空值得到非空值。

### 区分可空与非空类型
Kotlin、Swift、Hack 都严格区分可空类型与非空类型，并且类型都默认非空，对于可空类型也都采用加注 `?` 的方式来表达（只是 Hack 放在类型名前，Kotlin 与 Swift 放在类型名后）。我们看些具体的示例：

``` kotlin
// Kotlin
val a: Int = 1 // OK
val b: Int = null // 错误，b 不接受空值
val c: Int? = null // OK，c 是可空类型

val d: Int = a // OK
val e: Int? = a // OK，非空表达式可以赋给可空变量
val f: Int = c // 错误，e 不接受可空表达式赋值
val g: Int? = c // OK
val h = c // OK，h 的类型会推断为 Int?
```
Swift 版与之非常类似，只需将 `val` 替换为 `let`、`null` 替换为 `nil` 即可。
Hack 语法与它们差异大些、并且需要采用函数形式来表达上述示例，但其相似性还是很明显的：

``` php
// Hack
function a(): int { return 1; }
function b(): int { return null; } // 错误
function c(): ?int { return null; }
function d(): int { return a(); }
function e(): ?int { return a(); }
function f(): int { return c(); }
function g(): ?int { return c(); } // 错误
function h() { return c(); }
```

除了变量声明与赋值、函数返回值之外，三门语言对函数传参、数学运算等各种表达式都会严格区分可空与非空类型。

### 可空性传播
在采用受限空值的编程语言中，无法直接访问可空类型对象的成员，需要使用特殊语法。在 Kotlin 与 Swift 中使用 `?.` 语法，在 Hack 中使用 `?->` 语法。例如输出一个可空字符串的长度：

Kotlin 代码，输出是 `12`：

``` kotlin
val hello: String? = "Hello, World"
println(hello?.length)
hello?.length.plus(10) // 错误，可空值不能直接调用方法
```

Swift 代码，输出是 `Optional(12)`：

``` swift
let hello: String? = "Hello, World"
print(hello?.count)
hello?.count + 10 // 错误，可空值不能用于算术运算
```

Hack 代码有些复杂，因为内置字符串值不是对象，所以需要模拟出一个对象，其输出是 `12`:

``` php
class String0 {
    private string $s;
    public function __construct(string $s) {
        $this->s = $s;
    }
    public function length(): int {
        return strlen($this->s);
    }
}

function hello(): ?String0 {
    return new String0("Hello, World");
}

function printLength(?String0 $s) {
   echo $s?->length(), "\n";
}

printLength(new String0("Hello, World"));

function foo(): int { return hello()?->length(); } // 错误，函数签名要求非空返回值
```

Swift 输出的是 `Optional(12)`，它明确表明这是一个 `Int?` 值。Kotlin 与 Swift 虽然直接输出了数字，但其值同样是可空整型，不能用于只接受非空整数的地方。`?.`/`?->`的求值逻辑为：
  1. 如果对象非空，那么访问相应成员。
  2. 如果对象为空，返回空。
  3. 返回类型为可空类型。

以 Kotlin 为例，虽然 `String` 的 `length` 属性是非空成员，但因为 `hello` 是可空的，进而导致 `hello?.length` 也是可空的，因此如需继续调用 `plus` 也要使用 `?.` 语法：

``` kotlin
>>> hello?.length?.plus(10)
22
```

并且这一表达式依然是可空的，因此如果还有后续成员访问，就还需使用 `?.` 语法：

``` kotlin
>>> hello?.length?.plus(10)?.times(1.2)?.toLong()
26
```

可空性就像病毒一样感染了整个调用链条，并且会继续传播下去。在 Hack 中与此类似，只是使用 `?->` 语法。Swift 的语法与它们不同，对于上述情况，后续链条中用 `.` 即可，但是最终结果仍然是可空值：

``` swift
3> let a = hello?.utf8.count.advanced(by: 10).distance(to: 100)
a: Int? = 78
```

Swift 中只有后续成员的返回值本身也是可空类型时才需要再次使用 `?.`，参见其[官网介绍](https://docs.swift.org/swift-book/LanguageGuide/OptionalChaining.html#ID252)：

> ``` swift
> if let johnsStreet = john.residence?.address?.street {
>     print("John's street name is \(johnsStreet).")
> } else {
>     print("Unable to retrieve the address.")
> }
> ```

当然，在 Kotlin 中也可以通过高阶函数 `let` 来简化多级 `?.` 的语法，进而达到接近 Swift 的效果：

``` kotlin
>>> hello?.let{ it.length.plus(10).times(1.2).toLong() }
26
```

### 可空值用于常规函数
如果不是访问成员，而是用于普通函数，例如将上述链式调用的结果传给 `sin` 函数并输出其结果，该如何实现呢？
这在 Kotlin 与 Swift 中分别有不同的语法：

Kotlin 代码：

``` kotlin
>>> hello?.length?.plus(10)?.times(1.2)?.toLong()?.let {
...     println(Math.sin(it * 1.0))
... }
0.7625584504796028
```

Swift 代码：

``` swift
4> import Foundation
5> if let a = hello?.utf8.count.advanced(by: 10).distance(to: 100) {
6.     print(sin(Double(a)))
7. }
0.513978455987535
```

目前在 Hack 中没有类似语法，可以通过更通用的由可空表达式获得非空值的方式实现。

### 由可空表达式获得非空值
这里只讨论安全获得非空值的方式。由可空表达式安全地获得非空值还需要提供一个默认值，这样就一定能够取得非空值：当表达式求值结果非空时取求值结果，否则取默认值。这在  Kotlin 中通过 Elvis 操作符（`?:`）来实现，在 Swift 与 Hack 中通过空接合操作符（`??`）来实现。

> 现在有没有觉得受限空值与问号（`?`）结下了不解之缘 :P

Kotlin 示例：

``` kotlin
>>> val hello: String? = "Hello, World"
>>> val len: Int = hello?.length ?: 0
>>> ((hello?.length ?: 0) + 10) * 1.2
26.4
>>> val hello: String? = null
>>> ((hello?.length ?: 0) + 10) * 1.2
12.0
```

Swift 示例：
``` swift
  1> let hello: String? = "Hello, World" 
hello: String? = "Hello, World"
  2> let len: Int = hello?.count ?? 0 
len: Int = 12
  3> 100 - ((hello?.count ?? 0) + 10)
$R0: Int = 78
  4> let hello: String? = nil
hello: String? = nil
  5> 100 - ((hello?.count ?? 0) + 10) 
$R1: Int = 90
```

## 可选值类型
更多的现代编程语言都是采用可选值类型的方式。在这些语言中，都是通过一种专门的包装类型来表达可选值。
下表列举了一些语言中可选字符串的类型以及有无值的字面值表示法：

| 语言 | 可选值类型 | 无值 | 有值
|-----|-------|------|---|
| Haskell | `Maybe String` | `Nothing` | `Just "Hello"` |
| Rust | `Option<&str>` | `None` | `Some("Hello")` |
| Julia | `Union{Some{T}, Nothing}` | `nothing` | `Some("Hello")` |
| OCaml/F# | `string option` | `None` | `Some "Hello"` |
| Swift | `Optional<String>` | `Optional.none` | `Optional.some("Hello")` |
| Scala | `Option[String]` | `None` | `Some("Hello")` |
| Java 8+  | `Optional<String>` | `Optional.empty()` | `Optional.of("Hello")` |
| C++ 17+ | `optional<string>` | `nullopt` | `optional{"Hello"}` |

包装后的类型与原类型明显不同，因此无法当作原类型来用。那么应该如何使用呢？

### 显式判断与模式匹配
最简单的使用方式是显式判断，例如求一个可选字符串的长度（Julia）：

``` julia
julia> lengthOfOptionalString(us::Union{Some{String}, Nothing}) =
           if us === nothing
               0
           else
               length(us.value)
           end
lengthOfOptionalString (generic function with 1 method)

julia> lengthOfOptionalString(nothing)
0

julia> lengthOfOptionalString(Some("Hello"))
5
```

如果用 C++ 或者 Java 实现会与之非常相似。而对于上文所列的其他使用可选值类型的语言，都可以采用模式匹配的方式实现类似功能。例如（Haskell）：

``` haskell
ghci > :{
ghci | lengthOfMaybeString:: Maybe String -> Int
ghci | lengthOfMaybeString Nothing = 0
ghci | lengthOfMaybeString (Just s) = length s
ghci | :}
ghci > lengthOfMaybeString Nothing
0
ghci > lengthOfMaybeString (Just "Hello")
5
```

在 Haskell 中只需在声明函数时对 `Maybe String` 类型参数的不同模式 `Nothing` 与 `Just s` 分别编写实现即可。函数调用时 Haskell 会根据实参类型自动匹配到相应实现。

> 实际上，Julia 虽然没有在语言级支持完整的模式匹配，但是在 Julia 中可以通过泛型函数实现与上述 Haskell 代码类似的写法：
> 
> ``` julia
> julia> lengthOfOptionalString(nothing) = 0
> lengthOfOptionalString (generic function with 1 method)
> 
> julia> lengthOfOptionalString(ss::Some{String}) = length(ss.value)
> lengthOfOptionalString (generic function with 2 methods)
> 
> julia> lengthOfOptionalString(nothing)
> 0
> 
> julia> lengthOfOptionalString(Some("Hello"))
> 5
> ```

我们再看一下 Rust 的写法：

``` rust
irust> let a: Option<&str> = None;
()
irust> let b: Option<&str> = Some("Hello");
()
irust> match a { None => 0, Some(ref s) => s.len() }
0
irust> match b { None => 0, Some(ref s) => s.len() }
5
```

这段代码乍一看跟传统语言的 switch-case 很类似，但实际上要强大的多。
上述代码中的 `Some(ref s) => 含 s 的表达式` 就是传统 switch-case 无法支持的。对于匹配到模式 `Some(ref s)` 的 `Option`，Rust 能够自动提取模式中对应的 `s`，并用于后续处理。

我们可以通过显式判断或模式匹配来处理可选值类型，但通常并不这么做，因为还有更便捷的方式。

### 函数式方式
以函数式方式实现求一个可选字符串的长度的代码，可以这样写（Java）：

``` java
jshell> int lengthOfOptionalString(Optional<String> opStr) {
   ...>     return opStr.map(String::length).orElse(0);
   ...> }
   ...>
| 已创建 方法 lengthOfOptionalString(Optional<String>)

jshell> lengthOfOptionalString(Optional.empty())
$2 ==> 0

jshell> lengthOfOptionalString(Optional.of("Hello"))
$3 ==> 5
```

这里用到了 `Optional<T>` 的两个方法：`map()` 与 `orElse()`。

其中 `Optional<T>.map()` 接受一个函数式接口参数 `mapper`（可以传入 lambda 表达式或者方法引用），如果可选值无值，那么直接返回 `Optional.empty()`；而如果有值，那么返回对其值调用 `mapper` 所得结果的 `Optional<U>` 包装。 

`Optional<U>.orElse()` 接受一个 `U` 类型的参数 `default`，如果可选值有值则返回其值，如果无值返回 `default`，因此通过 `Optional<U>.orElse()` 总能得到一个 `U` 类型的值。

实际上，可选值类型是 [Functor、Applicative、Monad](https://hltj.me/kotlin/2017/08/25/kotlin-functor-applicative-monad-cn.html)，上述 `Optional.map()` 相当于 Haskell 中 `Functor` 的 `fmap`/`<$>`。此外，常用的还有相当于 `Monad` 的 `>>=` 的函数，如 Java 的 `Optional.flatMap()`、Rust 的 `Option::and_then()`、F# 的 `Option.bind` 等。我们看一个 F# 的示例——对一个整数可选值求余：

``` fsharp
> let (%?) a b =
-     match b with
-     | 0 -> None
-     | _ -> Some(a % b);;
val ( %? ) : a:int -> b:int -> int option

> 18 %? 5;;
val it : int option = Some 3

> 18 %? 0;;
val it : int option = None

> Option.map ((%?) 18) (Some 5);;
val it : int option option = Some (Some 3)

> Option.map ((%?) 18) (Some 0);;
val it : int option option = Some None

> Option.map ((%?) 18) None;;
val it : int option option = None

> Option.bind ((%?) 18) (Some 5);;
val it : int option = Some 3

> Option.bind ((%?) 18) (Some 0);;
val it : int option = None

> Option.bind ((%?) 18) None;;
val it : int option = None
```
示例中首先定义了一个安全求余运算符 `%?`，当除数为 `0` 时它返回 `None`，否则返回 `Some 余数`。
`%?` 只接受整数作除数，如需将其应用到 `int option` 可以借助 `Option.map`，但是这样得到的结果是嵌套的 `option`（即 `int option option`）。 有没有可能直接得到单层的 `int option` 呢？——这就需要 `Option.bind` 大显身手了，如例中所示。

> 注：上文所列的其他使用可选值类型的语言的标准库没有为可选值类型实现类似 Haskell 中 `Applicative` 的 `liftA2`/`<*>` 的函数，可选用第三方实现或者参考 [Applicative 文档](http://hackage.haskell.org/package/base-4.12.0.0/docs/Control-Applicative.html#t:Applicative)或 [Kotlin 版图解 Functor、Applicative 与 Monad](https://hltj.me/kotlin/2017/08/25/kotlin-functor-applicative-monad-cn.html) 自行实现。

## 综合示例
我们看一个 [Kotlin 心印中的示例](https://play.kotlinlang.org/koans/Introduction/Nullable%20types/Task.kt)：实现一个给客户发消息的函数，其中客户、消息、客户的个人信息字段、个人信息的邮箱地址字段都可能无值。
用传统 Java 代码实现如下所示：

``` java
public void sendMessageToClient(
    @Nullable Client client, @Nullable String message, @NotNull Mailer mailer
) {
    if (client == null || message == null) return;

    PersonalInfo personalInfo = client.getPersonalInfo();
    if (personalInfo == null) return;

    String email = personalInfo.getEmail();
    if (email == null) return;

    mailer.sendMessage(email, message);
}
```

上述代码使用了卫语句，可以说已经是质量很高的传统 Java 代码了。
但由于对其中每处可空值都需要判空，有意义的代码与判空代码各有三行，可以说一半的代码都浪费在了毫无业务价值而又不得不做的事情上了。
同时该代码中有 4 个分支、4 个出口，代码虽不多，流程却已略显复杂。
而如果使用 Java 8 的 `Optional`，就可以流畅很多：

``` java
public void sendMessageToClient(
    Optional<Client> client, Optional<String> message, @NotNull Mailer mailer
) {
    message.ifPresent(
            message1 -> client
                    .flatMap(Client::getPersonalInfo)
                    .flatMap(PersonalInfo::getEmail)
                    .ifPresent(
                        email -> mailer.sendMessage(email, message1)
                    )
    );
}
```
在 Swift 语言中，可以综合使用受限空值与可选值类型的语法，代码会更简洁一些：
``` swift
func sendMessageToClient(client: Client?, message: String?, mailer: Mailer) {
    message.map {
        message1 in client?.personalInfo?.email.map {
            email in mailer.sendMessage(email, message1)
        }
    }
}
```
在 Kotlin 语言中，可以综合使用受限空值与 `return` 表达式，代码会非常简洁：
``` kotlin
fun sendMessageToClient(client: Client?, message: String?, mailer: Mailer) {
    mailer.sendMessage(client?.personalInfo?.email ?: return, message ?: return)
}
```

## 综述
传统空值会带来一系列问题，为避免这些问题，现代编程语言通常采用**受限的空值**或者**可选值类型**来表达可选值。
这些现代编程语言不仅通过类型系统确保了可选值的安全性，还提供了各种相对便利的使用方式来提升可选值的易用性。
