---
title: Kotlin笔记-类与对象（上）
categories:
  - Android
  - Kotlin
comments: true
date: 2018-06-22 10:30:12
updated: 2018-06-22 10:30:12
tags: Kotlin
keywords: Kotlin 学习笔记
description:
---

# Kotlin 类与对象笔记（上）

<!-- more -->

## 类与继承

### 类

Kotlin 中使用关键字 `class` 声明类

```kotlin
class Invoice {
}

class Empty
```

### 构造函数

在 Kotlin 中的一个类可以有一个**主构造函数**和一个或多个**次构造函数**。主构造函数是类头的一部分：它跟在类名（和可选的类型参数）后。

```kotlin
class Person constructor(firstName: String) {
}

如果主构造函数没有任何注解或者可见性修饰符，可以省略这个 constructor 关键字。

constructor 可省略
```

主构造函数不能包含任何的代码。初始化的代码可以放到以 `init` 关键字作为前缀的 **初始化块**（initializer blocks）中。

```kotlin
class Person constructor(firstName: String) {

    val firstProperty = "First property: $name".also(::println)

    init {
        println("First initializer block that prints ${name}")
    }
}
```

如果构造函数有注解或可见性修饰符，这个 constructor 关键字是必需的，并且这些修饰符在它前面：

```kotlin
class Customer public @Inject constructor(name: String) { …… }
```

### 次构造函数

类也可以声明前缀有 `constructor` 的**次构造函数**：
```kotlin
class Person {
    constructor(parent: Person) {
        parent.children.add(this)
    }
}
```

### 创建类的实例

要创建一个类的实例，我们就像普通函数一样调用构造函数：

```kotlin
val invoice = Invoice()

val customer = Customer("Joe Smith")
```

注意 Kotlin 并没有 `new` 关键字。

### 继承

在 Kotlin 中所有类都有一个共同的超类 `Any`，这对于没有超类型声明的类是默认超类：

> 注意：Any 并不是 java.lang.Object；尤其是，它除了 equals()、hashCode()和toString()外没有任何成员。

类上的 open 标注与 Java 中 final 相反，它允许其他类从这个类继承。默认情况下，在 Kotlin 中所有的类都是 final

```kotlin
open class Base(p: Int) // Base 需为 open 才可以被 Derived 继承

class Derived(p: Int) : Base(p)
```

### 覆盖方法

```kotlin
open class Base {
    open fun v() {}
    fun nv() {}
}
class Derived() : Base() {
    override fun v() {}
}
```

Derived.v() 函数上必须加上 override标注。如果没写，编译器将会报错。 如果函数没有标注 open 如 Base.nv()，则子类中不允许定义相同签名的函数， 不论加不加 override。在一个 final 类中（没有用 open 标注的类），开放成员是禁止的。
标记为 override 的成员本身是开放的，也就是说，它可以在子类中覆盖。如果你想禁止再次覆盖，使用 final 关键字：

```kotlin
open class AnotherDerived() : Base() {
    final override fun v() {}
}
```

### 覆盖属性

你可以用一个 var 属性覆盖一个 val 属性，但反之则不行。

你可以在主构造函数中使用 override 关键字作为属性声明的一部分。

### 派生类初始化顺序

在构造派生类的新实例的过程中，第一步完成其基类的初始化（在之前只有对基类构造函数参数的求值），因此发生在派生类的初始化逻辑运行之前。

例子
```kotlin
//sampleStart
open class Base(val name: String) {

    init { println("Initializing Base") }

    open val size: Int =
        name.length.also { println("Initializing size in Base: $it") }
}

class Derived(
    name: String,
    val lastName: String
) : Base(name.capitalize().also { println("Argument for Base: $it") }) {

    init { println("Initializing Derived") }

    override val size: Int =
        (super.size + lastName.length).also { println("Initializing size in Derived: $it") }
}
//sampleEnd

fun main(args: Array<String>) {
    println("Constructing Derived(\"hello\", \"world\")")
    val d = Derived("hello", "world")
}


输出
Constructing Derived("hello", "world")
Argument for Base: Hello
Initializing Base
Initializing size in Base: 5
Initializing Derived
Initializing size in Derived: 10

```

这意味着，基类构造函数执行时，派生类中声明或覆盖的属性都还没有初始化。如果在基类初始化逻辑中（直接或通过另一个覆盖的 open 成员的实现间接）使用了任何一个这种属性，那么都可能导致不正确的行为或运行时故障。设计一个基类时，应该避免在构造函数、属性初始化器以及 init 块中使用 open 成员。

### 调用超类实现

派生类中的代码可以使用 super 关键字调用其超类的函数与属性访问器的实现：

在一个内部类中访问外部类的超类，可以通过由外部类名限定的 `super` 关键字来实现：super@Outer：

### 覆盖规则

在 Kotlin 中，实现继承由下述规则规定：如果一个类从它的直接超类继承相同成员的多个实现， 它必须覆盖这个成员并提供其自己的实现（也许用继承来的其中之一）。 为了表示采用从哪个超类型继承的实现，我们使用由尖括号中超类型名限定的 super，如 super<Base>：

```kotlin
open class A {
    open fun f() { print("A") }
    fun a() { print("a") }
}

interface B {
    fun f() { print("B") } // 接口成员默认就是“open”的
    fun b() { print("b") }
}

class C() : A(), B {
    // 编译器要求覆盖 f()：
    override fun f() {
        super<A>.f() // 调用 A.f()
        super<B>.f() // 调用 B.f()
  }
}
```

同时继承 A 和 B 没问题，并且 a() 和 b() 也没问题因为 C 只继承了每个函数的一个实现。 但是 f() 由 C 继承了两个实现，所以我们必须在 C 中覆盖 f() 并且提供我们自己的实现来消除歧义。

### 抽象类

类和其中的某些成员可以声明为 `abstract`。 抽象成员在本类中可以不用实现。 需要注意的是，我们并不需要用 `open` 标注一个抽象类或者函数——因为这不言而喻。

我们可以用一个抽象成员覆盖一个非抽象的开放成员

```kotlin
open class Base {
    open fun f() {}
}

abstract class Derived : Base() {
    override abstract fun f()
}
```

### 伴生对象

与 Java 或 C# 不同，在 Kotlin 中类没有静态方法。在大多数情况下，它建议简单地使用包级函数。

如果你需要写一个可以无需用一个类的实例来调用、但需要访问类内部的函数（例如，工厂方法），你可以把它写成该类内对象声明中的一员。

更具体地讲，如果在你的类内声明了一个伴生对象， 你就可以使用像在 Java/C# 中调用静态方法相同的语法来调用其成员，只使用类名作为限定符。

## 属性与字段

[属性与字段](https://hltj.gitbooks.io/kotlin-reference-chinese/content/txt/properties.html)

## 接口

使用关键字 `interface` 来定义接口

### 接口继承

一个接口可以从其他接口派生，从而既提供基类型成员的实现也声明新的函数与属性。很自然地，实现这样接口的类只需定义所缺少的实现：

```kotlin
interface Named {
    val name: String
}

interface Person : Named {
    val firstName: String
    val lastName: String

    override val name: String get() = "$firstName $lastName"
}

data class Employee(
    // 不必实现“name”
    override val firstName: String,
    override val lastName: String,
    val position: Position
) : Person
```

## 可见性修饰符

类、对象、接口、构造函数、方法、属性和它们的 setter 都可以有 可见性修饰符。 （getter 总是与属性有着相同的可见性。） 在 Kotlin 中有这四个可见性修饰符：`private`、 `protected`、 `internal` 和 `public`。 如果没有显式指定修饰符的话，默认可见性是 `public`。

### 包

函数、属性和类、对象和接口可以在顶层声明，即直接在包内：

- 如果你不指定任何可见性修饰符，默认为 public，这意味着你的声明将随处可见；
- 如果你声明为 private，它只会在声明它的文件内可见；
- 如果你声明为 internal，它会在相同模块内随处可见；
- protected 不适用于顶层声明。

注意：要使用另一包中可见的顶层声明，仍需将其导入进来。

### 类和接口

对于类内部声明的成员：

- private 意味着只在这个类内部（包含其所有成员）可见；
- protected—— 和 private一样 + 在子类中可见。
- internal —— 能见到类声明的 本模块内 的任何客户端都可见其 internal 成员；
- public —— 能见到类声明的任何客户端都可见其 public 成员。

注意 对于Java用户：Kotlin 中外部类不能访问内部类的 private 成员。

### 构造函数

要指定一个类的的主构造函数的可见性，使用以下语法（注意你需要添加一个显式 `constructor` 关键字）：

```kotlin
class C private constructor(a: Int) { …… }
```

这里的构造函数是私有的。默认情况下，所有构造函数都是 `public`，这实际上等于类可见的地方它就可见（即 一个 `internal` 类的构造函数只能在相同模块内可见).

### 局部声明

局部变量、函数和类不能有可见性修饰符。

### 模块

可见性修饰符 `internal` 意味着该成员只在相同模块内可见。更具体地说， 一个模块是编译在一起的一套 Kotlin 文件：

- 一个 IntelliJ IDEA 模块；
- 一个 Maven 项目；
- 一个 Gradle 源集（例外是 test 源集可以访问 main 的 internal 声明）；
- 一次 ＜kotlinc＞ Ant 任务执行所编译的一套文件。

## 扩展

Kotlin 同 C# 和 Gosu 类似，能够扩展一个类的新功能而无需继承该类或使用像装饰者这样的任何类型的设计模式。 这通过叫做 扩展 的特殊声明完成。Kotlin 支持 扩展函数 和 扩展属性。

### 扩展函数

声明一个扩展函数，我们需要用一个 接收者类型 也就是被扩展的类型来作为他的前缀。

### 扩展是静态解析的

扩展不能真正的修改他们所扩展的类。通过定义一个扩展，你并没有在一个类中插入新成员， 仅仅是可以通过该类型的变量用点表达式去调用这个新函数。

我们想强调的是扩展函数是静态分发的，即他们不是根据接收者类型的虚方法。 这意味着调用的扩展函数是由函数调用所在的表达式的类型来决定的， 而不是由表达式运行时求值结果决定的。例如：

```kotlin
open class C

class D: C()

fun C.foo() = "c" // 扩展

fun D.foo() = "d" // 扩展

fun printFoo(c: C) {
    println(c.foo())
}

printFoo(D())

这个例子会输出 "c"，因为调用的扩展函数只取决于参数 c 的声明类型，该类型是 C 类。
```


如果一个类定义有一个成员函数和一个扩展函数，而这两个函数又有相同的接收者类型、相同的名字并且都适用给定的参数，这种情况总是取成员函数。 例如：
```kotlin
class C {
    fun foo() { println("member") }
}

fun C.foo() { println("extension") }

如果我们调用 C 类型 c的 c.foo()，它将输出“member”，而不是“extension”。
```

### 可空接收者

注意可以为可空的接收者类型定义扩展。这样的扩展可以在对象变量上调用， 即使其值为 null，并且可以在函数体内检测 `this == null`，这能让你在没有检测 null 的时候调用 Kotlin 中的toString()：检测发生在扩展函数的内部。

```kotlin
fun Any?.toString(): String {
    if (this == null) return "null"
    // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
    // 解析为 Any 类的成员函数
    return toString()
}
```

### 扩展属性

和函数类似，Kotlin 支持扩展属性：

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

### 伴生对象的扩展

如果一个类定义有一个伴生对象 ，你也可以为伴生对象定义扩展函数和属性：

```kotlin
class MyClass {
    companion object { }  // 将被称为 "Companion"
}

fun MyClass.Companion.foo() {
    // ……
}
```

就像伴生对象的其他普通成员，只需用类名作为限定符去调用他们

```kotlin
MyClass.foo()
```

### 扩展的作用域

[扩展的作用域](https://hltj.gitbooks.io/kotlin-reference-chinese/content/txt/extensions.html)

## 数据类

我们经常创建一些只保存数据的类。 在这些类中，一些标准函数往往是从数据机械推导而来的。在 Kotlin 中，这叫做 数据类 并标记为 `data`：

### 复制

在很多情况下，我们需要复制一个对象改变它的一些属性，但其余部分保持不变。 copy() 函数就是为此而生成。

```kotlin
fun copy(name: String = this.name, age: Int = this.age) = User(name, age)

val jack = User(name = "Jack", age = 1)
val olderJack = jack.copy(age = 2)
```

### 数据类和解构声明

为数据类生成的 Component 函数 使它们可在解构声明中使用：

```kotlin
val jane = User("Jane", 35)
val (name, age) = jane
println("$name, $age years of age") // 输出 "Jane, 35 years of age"
```

## 密封类

密封类用来表示受限的类继承结构：当一个值为有限集中的类型、而不能有任何其他类型时。在某种意义上，他们是枚举类的扩展：枚举类型的值集合也是受限的，但每个枚举常量只存在一个实例，而密封类的一个子类可以有可包含状态的多个实例。

要声明一个密封类，需要在类名前面添加 `sealed` 修饰符。虽然密封类也可以有子类，但是所有子类都必须在与密封类自身相同的文件中声明。（在 Kotlin 1.1 之前， 该规则更加严格：子类必须嵌套在密封类声明的内部）。

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()
data class Sum(val e1: Expr, val e2: Expr) : Expr()
object NotANumber : Expr()
```

（上文示例使用了 Kotlin 1.1 的一个额外的新功能：数据类扩展包括密封类在内的其他类的可能性。 ）

一个密封类是自身抽象的，它不能直接实例化并可以有抽象（`abstract`）成员。

密封类不允许有非 - `private` 构造函数（其构造函数默认为 `private`）。

请注意，扩展密封类子类的类（间接继承者）可以放在任何位置，而无需在同一个文件中。

使用密封类的关键好处在于使用 when 表达式 的时候，如果能够验证语句覆盖了所有情况，就不需要为该语句再添加一个 `else` 子句了。当然，这只有当你用 `when` 作为表达式（使用结果）而不是作为语句时才有用。

```kotlin
fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // 不再需要 `else` 子句，因为我们已经覆盖了所有的情况
}
```


