---
title: Kotlin学习系列三：表达式
date: 2021-01-15 10:14:31
tags: [Kotlin, 表达式]
summary: Kotlin表达式
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 表达式**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
4. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}
    * {% post_link kotlin学习系列五 kotlin学习系列五：类型进阶 %}

# 二、常量和变量

## 1. 变量 

**修饰符：**
**`var`**：声明可读写变量。
**`val`**：声明**只读变量**，对应 java 里加了`final`修饰符的变量，若作为属性使用，可以创建`get`方法，但没有`set`方法。
例：

```kotlin
class X {
    val b : Int
    get() {
        return (Math.random() * 100).toInt()
    }
    fun test () {
        var ac = 2
        ac = 3
    }
}
```

## 2. 常量

### 1. 常量值创建 

**修饰符：** <font size = 5>`const`</font>
**说明：**
   1. 只能定义在全局范围
   2. 只能修饰基本类型，如`Int`、`Float`等
   3. 必须立即用*字面量*初始化

例：

```kotlin
const val b = 3
```

对标java中的静态常量

```java
static final int b = 3;
```

### 2. 常量引用

对应自定义对象，

```kotlin
val person = Person(18, "Lee")
person.age = 20
```

创建的自定义对象时是在堆上的，对象里属性可以改变，但引用没变

### 编译期和运行时常量

编译期常量：编译时即可确定常量值，并用值替换调用处

例：

```kotlin
const val b = 3
```

运行时常量：运行时才能确定值，调用处通过引用获取值

例：

```kotlin
val c:Int
if (a == 3) {
    c = 5
} else {
    c = 6
}
```

# 三、分支表达式

## 1. `if ... else`

`kotlin`中`if ... else`是**表达式**，并非语句。

```java
c = a == 3 ? 5 : 6
```

```kotlin
c = if (a == 3) 5 else 6
```

## 2. `when ...`

`kotlin`中的`when`表达式功能等价于`java`中的`switch case`语句

```java
switch(a) {
    case 0:
        c = 5;
        break;
    case 1:
        c = 100;
        break;
    default:
        c = 20;
}
```

```kotlin
when(a) {
    0 -> c = 5
    1 -> c = 100
    else -> c = 200
}

var c = when(a) {
        0 -> 5
        1 -> 100
        else -> 200
    }
```

### 1. `when`语句中的条件可以下放到分支上

```kotlin
var x: Any ...
when {
    x is String -> c = x.length
    x == 1 -> c = 100
    else -> c = 20
}

var c = when {
        x is String -> x.length
        x == 1 -> 100
        else -> 20
    }
```

### 2. `when`条件中可以声明变量

**注意：** 在1.3版本之后可用

```kotlin
c = when (val input = readLine()) {
        null -> 0
        else -> input.length
    }
```

## 3. `try ... catch`

`kotlin`中的`try ... catch`表达式功能等价于`java`中的`try ... catch`语句

其中，`try`和`catch`可以看做是不同分支。

```kotlin
c = try {
        a / b
    } catch (e : Exception) {
        e.printStackTrace()
        0
    }
```

# 四、 运算符与中缀表达式

## 1. 运算符

**运算符重载**

* 运算符重载，就是对已有的运算符重新进行定义，赋予其另一种功能，以适应不同的数据类型。
* Java不支持运算符重载
* `kotlin`支持运算符重载，但运算符范围仅限官方指定的符号。[官网:运算符重载](https://www.kotlincn.net/docs/reference/operator-overloading.html)

* 二目算术运算符

    |表达式|翻译为|
    |----|----|
    |a + b|a.plus(b)|
    |a - b|a.minus(b)|
    |a * b|a.times(b)|
    |a / b|a.div(b)|
    |a % b|a.rem(b)、 a.mod(b) （已弃用）|
    |a..b|a.rangeTo(b)|

### 1. 一些`Kotlin`预置运算符重载

 *  `==`与`equals`

    `kotlin`中运算符`==`相当于调用函数`equals`，都是比较表达式的值

    ```kotlin
    var d = "Hello"
    var e = "World"
    d.equals(e)
    d == e
    ```

* `+`与`plus`

    `kotlin`中运算符`+`相当于调用函数`plus`

    ```kotlin
    2 + 3

    2.plus(3)
    ```

* `in`与`contains`

    `kotlin`中运算符`in`相当于调用函数`contains`

    ```kotlin
    val list = listOf<Int>(1,2,3,4)
    println(2 in list)
    println(list.contains(2))
    ```

* `[]`与`get`与`set`

    `kotlin`中运算符`[]`在某些条件下相当于调用函数`get`或函数`set`

    ```kotlin
    val map = mutableMapOf(
                "Hello" to 2,
                "World" to 3
        )
        val value = map["Hello"]
        val value1 = map.get("Hello")


        map["Hello"] = 4
        map.set("World", 4)
    ```

    若运算符`[]`在等号**左侧**，则等价于方法**`get`**
    若运算符`[]`在等号**右侧**，则等价于方法**`set`**

* `>`与`compareTo`

    函数`compareTo`
    **作用：** 与指定值进行比较。
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果此值等于指定的其他值，则返回零；
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果小于指定的其他值，则返回负数；
    &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;如果大于指定的其他值，则返回正数


    ```kotlin
    2 > 3
    2.compareTo(3) > 0
    ```

* `()`与`invoke`

    `kotlin`中运算符`()`相当于调用函数`invoke`

    ```kotlin
    val func = fun():Int {
            println("Hello")
            return 1
        }
    func()
    func.invoke()
    ```

### 2. 自定义运算符重载

自定义运算符时需要使用关键字 **`operator`**，作用：将一个函数标记为重载一个操作符，也就是操作符重载。

```kotlin
//复数的类
class Complex(var real: Double, var image: Double) {
    override fun toString(): String {
        return "$real + ${image}i"
    }

    operator fun plus(other: Complex): Complex {
        return Complex(this.real + other.real, this.image + other.image)
    }
    operator fun plus(other: Double) :Complex {
        return Complex(this.real + other, this.image)
    }
}
//扩展函数
operator fun Complex.plus(other:Int): Complex {
    return Complex(this.real + other, this.image)
}
//扩展函数
operator fun Complex.minus(real: Double): Complex {
    return Complex(this.real - real, this.image)
}
//扩展函数
operator fun Complex.get(index: Int):Double {
    return when(index) {
            0 -> this.real
            1 -> this.image
            else -> throw IndexOutOfBoundsException()
        }
}

fun main() {
    val a = Complex(3.0, 4.0)
    val b = Complex(2.0, 2.0)
    println(a + 20)
    println(a + b)
    println(a + 3)
    println(a - 3.0)
    println(a[0])
    println(a[1])
    println(a[2])
}
```

打印结果为:

```kotlin
23.0 + 4.0i
5.0 + 6.0i
6.0 + 4.0i
0.0 + 4.0i
3.0
4.0
Exception in thread "main" java.lang.IndexOutOfBoundsException
	at cn.ltt.projectcollection.kotlin.KotlinLabKt.get(KotlinLab.kt:137)
	at cn.ltt.projectcollection.kotlin.KotlinLabKt.main(KotlinLab.kt:150)
	at cn.ltt.projectcollection.kotlin.KotlinLabKt.main(KotlinLab.kt)
```

## 2. 中缀表达式

### 1. 定义

例如： `2 to 3` 等价于`2.to(3)`
所以`to`不是运算符

函数`to`的实现如下：

```kotlin
public infix fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

其中关键字 `infix` 表示该函数也可以使用中缀表示法（忽略该调用的点与圆括号）调用。中缀函数必须满足以下要求：

* 它们必须是成员函数或扩展函数；
* 它们必须只有一个参数；
* 其参数不得接受可变数量的参数且不能有默认值

### 2. 自定义

```kotlin
infix fun String.rotate(count:Int): String {
    val index = count % length
    return this.substring(index) + this.substring(0, index)
}

fun main() {
    println("HelloWorld" rotate 5)
}
```

如上，定义了一个字符串的可使用中缀表示法的扩展函数`rotate`，作用是将字符串在某个位置左右翻转

# 五、Lambda表达式

Kotlin 函数都是头等的，这意味着它们可以存储在变量与数据结构中、作为参数传递给其他高阶函数以及从其他高阶函数返回。可以像操作任何其他非函数值一样操作函数。

`Java`中的`Lambda`是`SAM(Single abstract method)`类型的语法糖，它需要一个接收类型，该接收类型只能是`只有一个方法的接口`，而`Kotlin`中的`Lambda`是匿名函数的语法糖。

```java
//java 8
Function1 f1 = (p) -> {
    System.out.println(p);
    return "Hello";
};
Function2 f2 = (p0, p1) -> {
    System.out.println(p0 + p1);
    return "World";
};
String result = f1.invoke(2);
f2.apply(1, 2);


interface Function1 {
    String invoke(int p);
}

interface Function2 {
    String apply(int p0, long p1);
}
```

## 1. 匿名函数

### 1. 匿名函数定义

普通的函数一般使用关键字`fun`后加函数名来创建，而匿名函数是**将函数名省略**

```kotlin
fun func() {
    println("Hello")
}

//匿名函数
fun() {
    println("Hello")
}
```

### 2. 匿名函数传递

可以将匿名函数赋值给变量

```kotlin
val func = fun(){
    println("Hello")
}
```

如上，`func`是一个变量名，并非函数名

### 3. 匿名函数类型

匿名函数只是少了函数名，类型与普通函数一致

```kotlin
val func : () -> Unit = fun() {
    println("Hello")
}
```

**若变量类型为*函数类型*，则该变量可以通过他的`invoke`函数调用或直接在变量后面加上操作符`()`**

```kotlin
//函数类型变量调用
func.invoke()
func()
```


## 2. `Lambda`表达式定义

`lambda` 表达式与匿名函数是 ***“函数字面值”*** ，即未声明的函数，但应立即做为表达式传递。

### 1. 表达式的完整语法形式

``` kotlin
val 函数名[：函数类型] = { 形参名[:形参类型] ->
    lambda表达式
}
```

**说明：**

* 函数类型包括形参类型和返回值类型
* `lambda` 表达式总是括在花括号中
* 完整语法形参声明放在花括号内，并有可选的类型标注， 函数体跟在一个 **` -> `** 符号之后。
* 如果推断出的该 `lambda` 的返回类型不是 `Unit`，那么该 `lambda` 主体中的最后一个（或可能是单个） 表达式会视为返回值。


例：

```kotlin
val sum: (Int, Int) -> Int = { x: Int, y: Int -> 
    x + y 
}
```

如上：

1. 声明了一个函数`sum`，`(Int,Int) -> Int`是函数`sum`的类型，表示函数`sum`有两个`Int`形参和一个返回`Int`类型的返回值。
2. 有两个`Int`类型的形参`x`、`y`，函数体只有一行`x + y`。
3. 其中操作符`+`是函数`plus`的重载，而函数`plus`返回结果为`Int`，所以函数`sum`返回值类型是`Int`


### 2. 表达式的函数类型

`lambda`表达式的类型还可以使用`FunctionN`

```kotlin
val sum: Function2<Int, Int, Int> = {x: Int, y: Int ->
    x + y
}
```

其中，`Function2`前两个`Int`表示两个`Int`类型的形参，最后一个`Int`表示`Int`类型的返回值。

### 3. 表达式参数类型推断

若表达式参数类型可从表达式类型推断出来，可不加类型

```kotlin
val sum: Function2<Int, Int, Int> = {x, y ->
    x + y
}

val sum2:(Int,Int) ->Int ={x, y ->
    x + y
}
```

其中，参数`x`和`y`的类型可由表达式类型推断为`Int`，所以可不加类型

### 4. 表达式类型推断

若表达式的类型可从声明中推断出来，可不加表达式类型

```kotlin
val sum ={x: Int, y: Int ->
    x + y
}
```

### 5. 表达式的参数省略形式

1. 若没有参数，则可省略`() ->`，*而`Java`中必须要有`() ->`*

    ```java
    //java 8
    Runnable lambda = ()-> {
        System.out.println("Hello");
    }
    ```

    ```kotlin
    val lambda = {
        println("Hello")
    }
    ```

2. 若参数只有一个，则可将参数及`() ->`都省略，**参数名可默认为关键字 `it`**  

    ```kotlin
    val f2:Function1<Int, Unit> = {
        println(it)
    }
    ```

# 六、应用

## 1. 为`Person` 实现`equals`和`hashCode`方法

```kotlin
class Person (var name: String, var age: Int) {
    override fun equals(other: Any?): Boolean {
        val other = (other as? Person)?:return false
        return other.age == age && other.name == name
    }

    override fun hashCode(): Int {
        return 11 + 2 * age + 10 * name.hashCode()
    }
}

fun main() {
    val persons = HashSet<Person>()
    (0..5).forEach {
        persons += Person("Lee", 20)
    }
    println(persons.size)
}
```

1. `HashSet`数据结构对象中不能存储相同的数据，存储数据时是无序的。
2. 操作符`+=`是函数`plusAssign`的重载，其作用为将某元素添加到`mutable collection`*(可变集合)*
3. 如果不覆写类`Person`的`equals`方法和`hashcode`方法，则结果打印为`6`，即认为每个`Person("Lee", 20)`都是不同对象
4. 若覆写了`Persona`的`equals`方法和`hashcode`方法，则结果打印为`1`，即认为每个`Person("Lee", 20)`是同一个对象

```kotlin
class Person (var name: String, var age: Int) {
    override fun equals(other: Any?): Boolean {
        val other = (other as? Person)?:return false
        return other.age == age && other.name == name
    }

    override fun hashCode(): Int {
        return 11 + 2 * age + 10 * name.hashCode()
    }
}
fun main() {
    val persons = HashSet<Person>()
    val person = Person("Lee", 18)
    persons += person
    println(persons.size)

    person.age = 20
    persons -= person

    println(persons.size)
    persons.forEach {
        println("Person{name:${it.name}, age:${it.age}}")
    }
}
```

打印结果：

```kotlin
1
1
Person{name:Lee, age:20}
```

如上自定义类`Person`，并覆写了`equals`方法和`hashcode`方法，但是这两个方法中均使用了`Person`类中的成员变量。
此时更改了已放入`HashSet`的对象属性后，根据打印结果可见，不能将该对象从`HashSet`对象中删除。

**结论：** 

1. `HashMap`的`key`和`HashSet`的对象本身的`equals`和`hashcode`方法一定不要在对象存续期间发生变化
2. 若覆写的`equals`和`hashcode`方法使用了对象的成员变量，则要保证成员变量不可更改，即声明为`val`
3. 若要修改声明为`val`的成员变量值，则只能重新创建对象

## 2. 为`String`添加四则运算

实现效果:

声明一个`String`类型变量`value`，其中除法规则是返回除数在字符串中出现的次数

|运算|结果|
|--|--|
|value + "!"| HelloWorld!|
|value - "World"| Hello|
|value * 2| HelloWorldHelloWorld|
|value / 3| 0|
|value / "l"| 3|

```kotlin
operator fun String.minus(right: Any?): String {
    return this.replaceFirst(right.toString(), "")
}

operator fun String.times(right: Int): String {
    return (1..right).joinToString("") { this }
}

operator fun String.div(right: Any?): Int {
    val right = right.toString()
    return this.windowed(right.length, 1) {
        it == right
    }//[false, false, true, true, false ... false, true, false]
     .count { it }
}

fun main() {
    val value = "HelloWorld"
    println(value - "World")
    println(value * 2)
    println("*" * 20)
    println(value / 3)
    println(value / "l")
}
```

打印结果为：

```kotlin
Hello
HelloWorldHelloWorld
********************
0
3
```

1. 形参`right`表示运算符右侧的运算数

2. `joinToString`函数是将可迭代元素拼接成一个字符串，默认以逗号分隔

3. `windowed(size: Int, step: Int = 1)`函数返回给定`[size]`窗口的**快照列表** ，并沿着给定的`[step]`沿着此`char`序列滑动，其中每个快照默认都是一个字符串。

4. `count{}` 函数返回符合某个条件的元素的个数




# 七、附录

参考文章：

1. [https://www.kotlincn.net/docs/reference/operator-overloading.html](https://www.kotlincn.net/docs/reference/operator-overloading.html)
2. [https://www.kotlincn.net/docs/reference/lambdas.html](https://www.kotlincn.net/docs/reference/lambdas.html)