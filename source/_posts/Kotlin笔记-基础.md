---
title: Kotlin笔记-基础
categories:
  - Android
  - Kotlin
comments: true
date: 2018-06-22 09:30:12
updated: 2018-06-22 09:30:12
tags: Kotlin
keywords: Kotlin 学习笔记
description:
---


# Kotlin 语法基础

<!-- more -->

## 基本类型

在 Kotlin 中，所有东西都是对象，在这个意义上讲我们可以在任何变量上调用成员函数和属性。 一些类型可以有特殊的内部表示——例如，数字、字符和布尔值可以在运行时表示为原生类型值，但是对于用户来说，它们看起来就像普通的类。 在本节中，我们会描述 Kotlin 中使用的基本类型：数字、字符、布尔值、数组与字符串。

### 表示方式

在 Java 平台数字是物理存储为 JVM 的原生类型，除非我们需要一个可空的引用（如 `Int?` ）或泛型。 后者情况下会把数字装箱。

注意数字装箱不必保留同一性:

```kotlin
val a: Int = 10000
print(a === a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA === anotherBoxedA) // ！！！输出“false”！！！
```

另一方面，它保留了相等性:

```kotliln
val a: Int = 10000
print(a == a) // 输出“true”
val boxedA: Int? = a
val anotherBoxedA: Int? = a
print(boxedA == anotherBoxedA) // 输出“true”
```

### 显式转换

由于不同的表示方式，较小类型并不是较大类型的子类型。 如果它们是的话，就会出现下述问题：

```kotliln
// 假想的代码，实际上并不能编译：
val a: Int? = 1 // 一个装箱的 Int (java.lang.Integer)
val b: Long? = a // 隐式转换产生一个装箱的 Long (java.lang.Long)
print(b == a) // 惊！这将输出“false”鉴于 Long 的 equals() 会检测另一个是否也为 Long
```

所以相等性会在所有地方悄无声息地失去，更别说同一性了。

因此较小的类型不能隐式转换为较大的类型。 这意味着在不进行显式转换的情况下我们不能把 Byte 型值赋给一个 Int 变量。

```kotliln
val b: Byte = 1 // OK, 字面值是静态检测的
val i: Int = b // 错误
```

我们可以显式转换来拓宽数字

```kotliln
val i: Int = b.toInt() // OK: 显式拓宽
```

每个数字类型支持如下的转换:

- toByte(): Byte
- toShort(): Short
- toInt(): Int
- toLong(): Long
- toFloat(): Float
- toDouble(): Double
- toChar(): Char

缺乏隐式类型转换并不显著，因为类型会从上下文推断出来，而算术运算会有重载做适当转换，例如：

```kotliln
val l = 1L + 3 // Long + Int => Long
```

### 运算

Kotlin支持数字运算的标准集，运算被定义为相应的类成员（但编译器会将函数调用优化为相应的指令）。

对于位运算，没有特殊字符来表示，而只可用中缀方式调用命名函数，例如:

```kotliln
val x = (1 shl 2) and 0x000FF000
```

这是完整的位运算列表（只用于 Int 和 Long）：

- shl(bits) – 有符号左移 (Java 的 <<)
- shr(bits) – 有符号右移 (Java 的 >>)
- ushr(bits) – 无符号右移 (Java 的 >>>)
- and(bits) – 位与
- or(bits) – 位或
- xor(bits) – 位异或
- inv() – 位非

### 数组

数组在 Kotlin 中使用 `Array` 类来表示，它定义了 `get` 和 `set` 函数（按照运算符重载约定这会转变为 `[]`）和 `size` 属性，以及一些其他有用的成员函数：

```kotliln
class Array<T> private constructor() {
    val size: Int
    operator fun get(index: Int): T
    operator fun set(index: Int, value: T): Unit

    operator fun iterator(): Iterator<T>
    // ……
}
```

我们可以使用库函数 `arrayOf()` 来创建一个数组并传递元素值给它，这样 `arrayOf(1, 2, 3)` 创建了 `array [1, 2, 3]`。 或者，库函数 `arrayOfNulls()` 可以用于创建一个指定大小的、所有元素都为空的数组。

另一个选项是用接受数组大小和一个函数参数的 `Array` 构造函数，用作参数的函数能够返回给定索引的每个元素初始值：

```kotliln
// 创建一个 Array<String> 初始化为 ["0", "1", "4", "9", "16"]
val asc = Array(5, { i -> (i * i).toString() })
```

注意: 与 Java 不同的是，Kotlin 中数组是不型变的（invariant）。这意味着 Kotlin 不让我们把 `Array<String>` 赋值给 `Array<Any>`，以防止可能的运行时失败（但是你可以使用 `Array<out Any>`, 参见类型投影）。

Kotlin 也有无装箱开销的专门的类来表示原生类型数组: `ByteArray`、 `ShortArray`、`IntArray` 等等。这些类和 `Array` 并没有继承关系，但是它们有同样的方法属性集。它们也都有相应的工厂方法:

```kotliln
val x: IntArray = intArrayOf(1, 2, 3)
x[0] = x[1] + x[2]
```

### 字符串

字符串用 `String` 类型表示。字符串是不可变的。 字符串的元素——字符可以使用索引运算符访问: `s[i]`。 可以用 `for` 循环迭代字符串:

请注意，在大多数情况下，优先使用 `字符串模板` 或 `原始字符串` 而不是字符串连接。

### 字符串字面值

Kotlin 有两种类型的字符串字面值: 转义字符串可以有转义字符，以及原始字符串可以包含换行和任意文本。

原始字符串 使用三个引号（`"""`）分界符括起来，内部 `没有转义` 并且可以包含换行和任何其他字符:


```kotliln
//输入值
val text = """
    for (c in "foo")
        print(c)
"""

//输出值
    for (c in "foo")
        print(c)
```

你可以通过 `trimMargin()` 函数去除前导空格：

```kotliln
//输入值
val text = """
    |Tell me and I forget.
     Teach me and I remember.
    |Involve me and I learn.
    |(Benjamin Franklin)
    """.trimMargin()

//输出值
Tell me and I forget.
    Teach me and I remember.
Involve me and I learn.
(Benjamin Franklin)
```

默认 `|` 用作边界前缀，但你可以选择其他字符并作为参数传入，比如 `trimMargin(">")`。

### 字符串模板

字符串可以包含模板表达式 ，即一些小段代码，会求值并把结果合并到字符串中。 模板表达式以美元符（`$`）开头，由一个简单的名字构成:

原始字符串和转义字符串内部都支持模板。 如果你需要在原始字符串中表示字面值 `$` 字符（它不支持反斜杠转义），你可以用下列语法：

```kotliln
//输入值
val price = """
${'$'}9.99
"""

//输出值
$9.99
```

## 包

### 默认导入

有多个包会默认导入到每个 Kotlin 文件中：

- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.* （自 1.1 起）
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*

根据目标平台还会导入额外的包：

- JVM:
    - java.lang.*
    - kotlin.jvm.*
- JS:
    - kotlin.js.*

### 导入

如果出现名字冲突，可以使用 `as` 关键字在本地重命名冲突项来消歧义：

```kotliln
import foo.Bar // Bar 可访问
import bar.Bar as bBar // bBar 代表“bar.Bar”
```

关键字 `import` 并不仅限于导入类；也可用它来导入其他声明：

- 顶层函数及属性；
- 在对象声明中声明的函数和属性;
- 枚举常量。

与 Java 不同，Kotlin 没有单独的“import static”语法； 所有这些声明都用 `import` 关键字导入。

### 顶层声明的可见性

如果顶层声明是 `private` 的，它是声明它的文件所私有的（参见 可见性修饰符）。

## 控制流

### If 表达式

`if`的分支可以是代码块，最后的表达式作为该块的值：

```kotliln
val max = if (a > b) {
    print("Choose a")
    a
} else {
    print("Choose b")
    b
}
```

如果你使用 `if` 作为表达式而不是语句（例如：返回它的值或者把它赋给变量），该表达式需要有 `else` 分支。

### When 表达式

`when` 将它的参数和所有的分支条件顺序比较，直到某个分支满足条件。 `when` 既可以被当做表达式使用也可以被当做语句使用。如果它被当做表达式， 符合条件的分支的值就是整个表达式的值，如果当做语句使用， 则忽略个别分支的值。（像 `if` 一样，每一个分支可以是一个代码块，它的值是块中最后的表达式的值。）

如果 `when` 作为一个表达式使用，则必须有 `else` 分支， 除非编译器能够检测出所有的可能情况都已经覆盖了［例如，对于 枚举（enum）类条目与密封（sealed）类子类型］。

如果很多分支需要用相同的方式处理，则可以把多个分支条件放在一起，用逗号分隔：

```kotliln
when (x) {
    0, 1 -> print("x == 0 or x == 1")
    else -> print("otherwise")
}
```

我们可以用任意表达式（而不只是常量）作为分支条件

我们也可以检测一个值在（`in`）或者不在（`!in`）一个区间或者集合中：

另一种可能性是检测一个值是（`is`）或者不是（`!is`）一个特定类型的值。注意： 由于智能转换，你可以访问该类型的方法和属性而无需任何额外的检测。

`when` 也可以用来取代 `if`-`else` `if`链。 如果不提供参数，所有的分支条件都是简单的布尔表达式，而当一个分支的条件为真时则执行该分支：

### For 循环

一些例子

```kotlin
for (item in collection) print(item)

for (item: Int in ints) {
    // ……
}

for (i in 1..3) {
    println(i)
}

for (i in 6 downTo 0 step 2) {
    println(i)
}

val array = arrayOf("a", "b", "c")
for (i in array.indices) {
    println(array[i])
}

val array = arrayOf("a", "b", "c")
for ((index, value) in array.withIndex()) {
    println("the element at $index is $value")
}
```

## 返回和跳转

Kotlin 有三种结构化跳转表达式：

- `return`。默认从最直接包围它的函数或者匿名函数返回。
- `break`。终止最直接包围它的循环。
- `continue`。继续下一次最直接包围它的循环。

所有这些表达式都可以用作更大表达式的一部分：

```kotlin
val s = person.name ?: return
```

这些表达式的类型是 Nothing 类型。

### Break 与 Continue 标签

在 Kotlin 中任何表达式都可以用标签（`label`）来标记。 标签的格式为标识符后跟 `@` 符号，例如：`abc@`、`fooBar@`都是有效的标签（参见语法）。 要为一个表达式加标签，我们只要在其前加标签即可。

现在，我们可以用标签限制 `break` 或者`continue`：

```kotlin
loop@ for (i in 1..100) {
    for (j in 1..100) {
        if (……) break@loop //表示跳出整个循环
        if (……) continue@loop //表示继续loop循环的下一次迭代，既i++
    }
}
```

标签限制的 break 跳转到刚好位于该标签指定的循环后面的执行点。 `continue` 继续标签指定的循环的下一次迭代。

### 标签处返回

例子
```kotlin

//直接退出循环
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return // 非局部直接返回到 foo() 的调用者
        print(it)
    }
    println("this point is unreachable")//不会输出
}
输出：12


//continue
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach lit@{
        if (it == 3) return@lit // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with explicit label")
}
输出：1245 done with explicit label


//隐式标签 continue
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach {
        if (it == 3) return@forEach // 局部返回到该 lambda 表达式的调用者，即 forEach 循环
        print(it)
    }
    print(" done with explicit label")
}
输出：1245 done with explicit label


//用匿名函数代替 lambda 表达式 continue
fun foo() {
    listOf(1, 2, 3, 4, 5).forEach(fun(value: Int) {
        if (value == 3) return  // 局部返回到匿名函数的调用者，即 forEach 循环
        print(it)
    })
    print(" done with anonymous function")
}
输出：1245 done with anonymous function


fun foo() {
    run loop@{
        listOf(1, 2, 3, 4, 5).forEach {
            if (it == 3) return@loop // 从传入 run 的 lambda 表达式非局部返回
            print(it)
        }
    }
    print(" done with nested loop")
}
输出：12 done with nested loop

```

当要返一个回值的时候，解析器优先选用标签限制的 return，即

```kotlin
return@a 1
```

意为“从标签 `@a` 返回 1”，而不是“返回一个标签标注的表达式 `(@a 1)`”。

