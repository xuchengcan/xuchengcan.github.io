---
title: Kotlin笔记-函数与 Lambda 表达式
tags: Kotlin
categories:
  - Android
keywords: Kotlin 学习笔记
comments: true
date: 2018-06-25 10:30:05
updated: 2018-06-25 10:30:05
description:
---

# 函数与 Lambda 表达式笔记

<!-- more -->

## 函数

### 函数声明

Kotlin 中的函数使用 fun 关键字声明：

### 默认参数

```kotlin
fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size) {
……
}
```

覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：

```kotlin
open class A {
    open fun foo(i: Int = 10) { …… }
}

class B : A() {
    override fun foo(i: Int) { …… }  // 不能有默认值
}
```

### 可变数量的参数（Varargs）

函数的参数（通常是最后一个）可以用 vararg 修饰符标记：

```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
```

允许将可变数量的参数传递给函数：

```kotlin
val list = asList(1, 2, 3)
```

当我们调用 `vararg` 函数时，我们可以一个接一个地传参，例如 `asList(1, 2, 3)`，或者，如果我们已经有一个数组并希望将其内容传给该函数，我们使用伸展（spread）操作符（在数组前面加 `*`）：

```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```

### 中缀表示法

标有 `infix` 关键字的函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：

- 它们必须是成员函数或扩展函数；
- 它们必须只有一个参数；
- 其参数不得接受可变数量的参数且不能有默认值。

```kotlin
infix fun Int.shl(x: Int): Int {
    // ……
}

// 用中缀表示法调用该函数
1 shl 2

// 等同于这样
1.shl(2)
```

### 函数作用域

在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样创建一个类来保存一个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

### 局部函数

Kotlin 支持局部函数，即一个函数在另一个函数内部：

局部函数可以访问外部函数（即闭包）的局部变量

### 尾递归函数

Kotlin 支持一种称为尾递归的函数式编程风格。 这允许一些通常用循环写的算法改用递归函数来写，而无堆栈溢出的风险。 当一个函数用 `tailrec` 修饰符标记并满足所需的形式时，编译器会优化该递归，留下一个快速而高效的基于循环的版本：

```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```

这段代码计算余弦的不动点（fixpoint of cosine），这是一个数学常数。 它只是重复地从 1.0 开始调用 Math.cos，直到结果不再改变，产生0.7390851332151607的结果。最终代码相当于这种更传统风格的代码：

```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return x
        x = y
    }
}
```

要符合 `tailrec` 修饰符的条件的话，函数必须将其自身调用作为它执行的最后一个操作。在递归调用后有更多代码时，不能使用尾递归，并且不能用在 try/catch/finally 块中。目前尾部递归只在 JVM 后端中支持。

## 高阶函数与 lambda 表达式

Kotlin 函数都是头等的，这意味着它们可以存储在变量与数据结构中、作为参数传递给其他高阶函数以及从其他高阶函数返回。可以像操作任何其他非函数值一样操作函数。

为促成这点，作为一门静态类型编程语言的 Kotlin 使用一系列函数类型来表示函数并提供一组特定的语言结构，例如 lambda 表达式。

### 高阶函数

高阶函数是将函数用作参数或返回值的函数。

```kotlin
fun <T, R> Collection<T>.fold(
    initial: R,
    combine: (acc: R, nextElement: T) -> R
): R {
    var accumulator: R = initial
    for (element: T in this) {
        accumulator = combine(accumulator, element)
    }
    return accumulator
}

fun main(args: Array<String>) {
    //sampleStart
    val items = listOf(1, 2, 3, 4, 5)

    // Lambdas 表达式是花括号括起来的代码块。
    items.fold(0, {
        // 如果一个 lambda 表达式有参数，前面是参数，后跟“->”
        acc: Int, i: Int ->
        print("acc = $acc, i = $i, ")
        val result = acc + i
        println("result = $result")
        // lambda 表达式中的最后一个表达式是返回值：
        result
    })

    // lambda 表达式的参数类型是可选的，如果能够推断出来的话：
    val joinedToString = items.fold("Elements:", { acc, i -> acc + " " + i })

    // 函数引用也可以用于高阶函数调用：
    val product = items.fold(1, Int::times)
    //sampleEnd
    println("joinedToString = $joinedToString")
    println("product = $product")
}

输出：
acc = 0, i = 1, result = 1
acc = 1, i = 2, result = 3
acc = 3, i = 3, result = 6
acc = 6, i = 4, result = 10
acc = 10, i = 5, result = 15
joinedToString = Elements: 1 2 3 4 5
product = 120
```

### 函数类型

Kotlin 使用类似 (Int) -> String 的一系列函数类型来处理函数的声明： val onClick: () -> Unit = ……。

这些类型具有与函数签名相对应的特殊表示法，即它们的参数和返回值：

> 如需将函数类型指定为可空，请使用圆括号：((Int, Int) -> Int)?。
> 箭头表示法是右结合的，(Int) -> (Int) -> Unit 与前述示例等价，但不等于 ((Int) -> (Int)) -> Unit。

### 函数式编程

```kotlin
fun main(args: Array<String>) {
    val oddLength = compose(::isOdd, ::length)
    val strings = listOf("a", "ab", "abc")
    println(strings.filter(oddLength))
}

fun isOdd(x: Int) = x % 2 != 0 //求基数
fun length(s: String) = s.length

fun <A, B, C> compose (  f: (B) -> C  ,  g: (A) -> B  )  : (A) -> C {
    return { x -> f(g(x)) }
}

fun <String, Int, Boolean> compose2 (  f: (Int) -> Boolean  ,  g: (String) -> Int  )  : (String) -> Boolean {
    return { x -> f(g(x)) }
}

输出：
[a,abc]
```

### Lambda 表达式语法

Lambda 表达式的完整语法形式如下：

```kotlin
val sum = { x: Int, y: Int -> x + y }
```

如果我们把所有可选标注都留下，看起来如下：

```kotlin
val sum: (Int, Int) -> Int = { x, y -> x + y }
```

### 将 lambda 表达式传给最后一个参数

在 Kotlin 中有一个约定：如果函数的最后一个参数接受函数，那么作为相应参数传入的 lambda 表达式可以放在圆括号之外：

```kotlin
val product = items.fold(1) { acc, e -> acc * e }
```

如果该 lambda 表达式是调用时唯一的参数，那么圆括号可以完全省略：

```kotlin
run { println("...") }
```

### `it`：单个参数的隐式名称

一个 lambda 表达式只有一个参数是很常见的。

如果编译器自己可以识别出签名，也可以不用声明唯一的参数并忽略 `->`。 该参数会隐式声明为 `it`：

```kotlin
ints.filter { it > 0 } // 这个字面值是“(it: Int) -> Boolean”类型的
```

### 从 lambda 表达式中返回一个值

我们可以使用限定的返回语法从 lambda 显式返回一个值。 否则，将隐式返回最后一个表达式的值。

因此，以下两个片段是等价的：

```kotlin
ints.filter {
    val shouldFilter = it > 0
    shouldFilter
}

ints.filter {
    val shouldFilter = it > 0
    return@filter shouldFilter
}
```

### 下划线用于未使用的变量

如果 lambda 表达式的参数未使用，那么可以用下划线取代其名称：

```kotlin
map.forEach { _, value -> println("$value!") }
```

### 闭包

Lambda 表达式或者匿名函数（以及局部函数和对象表达式） 可以访问其 闭包 ，即在外部作用域中声明的变量。 与 Java 不同的是可以修改闭包中捕获的变量：

```kotlin
var sum = 0
ints.filter { it > 0 }.forEach {
    sum += it
}
print(sum)
```

## 内联函数

使用高阶函数会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。 即那些在函数体内会访问到的变量。 内存分配（对于函数对象和类）和虚拟调用会引入运行时间开销。

但是在许多情况下通过内联化 lambda 表达式可以消除这类的开销。 下述函数是这种情况的很好的例子。即 lock() 函数可以很容易地在调用处内联。 考虑下面的情况：

[内联函数](https://hltj.gitbooks.io/kotlin-reference-chinese/content/txt/inline-functions.html)

## 协程

协程通过将复杂性放入库来简化异步编程。程序的逻辑可以在协程中顺序地表达，而底层库会为我们解决其异步性。该库可以将用户代码的相关部分包装为回调、订阅相关事件、在不同线程（甚至不同机器！）上调度执行，而代码则保持如同顺序执行一样简单。

### 挂起函数

当我们调用标记有特殊修饰符 suspend 的函数时，会发生挂起：

```kotlin
suspend fun doSomething(foo: Foo): Bar {
    ……
}
```

这样的函数称为挂起函数，因为调用它们可能挂起协程（如果相关调用的结果已经可用，库可以决定继续进行而不挂起）。挂起函数能够以与普通函数相同的方式获取参数和返回值，但它们只能从协程、其他挂起函数以及内联到其中的函数字面值中调用。

事实上，要启动协程，必须至少有一个挂起函数，它通常是匿名的（即它是一个挂起 lambda 表达式）。让我们来看一个例子，一个简化的 async() 函数（源自 kotlinx.coroutines 库）：

```kotlin
fun <T> async(block: suspend () -> T)
```
















