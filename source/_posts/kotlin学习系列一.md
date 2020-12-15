---
title: Kotlin 系列一：内置类型
date: 2020-12-07 16:51:45
tags: [Kotlin, 类型]
summary: Kotlin 内置类型
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 基本数据类型**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列二 kotlin学习系列二：kotlin函数 %}


# 二、基本类型

| 类型   | 大小 (bits) | 最小值                                       | 最大值                                         |
| ------ | ---------- | -------------------------------------------- | ---------------------------------------------- |
| Byte   | 8          | -128                                         | 127                                            |
| Short  | 16         | -32768                                       | 32767                                          |
| Int    | 32         | -2,147,483,648 (-2<sup>31</sup>)             | 2,147,483,647 (2<sup>31</sup> - 1)             |
| Long   | 64         | -9,223,372,036,854,775,808 (-2<sup>63</sup>) | 9,223,372,036,854,775,807 (2<sup>63</sup> - 1) |
| Float  | 32         | ——                                           | ——                                             | —— |
| Double | 64         | ——           | ——                                             | —— |
| Boolean | ——         | ——           | ——                                             | —— |

## 1. Kotlin 与 java 基本类型对比

| 名称   | Kotlin         | Java                        |
| ------ | -------------- | --------------------------- |
| 字节   | Byte           | byte/Byte                   |
| 整型   | Int & Long     | int/Integer & long/Long     |
| 浮点型 | Float & Double | float/Float & double/Double |
| 字符   | Char           | char/Character              |
| 字符串 | String         | String                      |

## 2. 变量声明

**格式：** <font size = 5>修饰符 变量名 [: 类型】 = 初始值</font>

![kotlin_type_1](kotlin学习系列一/kotlin_type_1.png)

**修饰符：**
**`val`**：声明只读变量，对应 java 里加了`final`修饰符的变量
**`var`**：声明可读写变量。

`Kotlin`里的`val a:Int = 2`等同于`Kotlin`里的`val a = 2`等同于`Java`里的`final int a = 2`

## 3. 类型推导

声明变量时可不加类型，`Kotlin`会根据初始化的值去自动匹配变量对应类型。

1. 整型默认为**Int**，浮点型默认为**Double**。
2. 变量声明为`Long`或`Float`时，要在初始值后加上`L`或`F`表示，其中`L`在一些字体中易与`i`混淆，所以**不能用小写的`l`声明长整型变量**。例如：

   ```kotlin
    val a = 32//Int类型
    val b = 23.2//Double类型
    val c = 32L//Long类型
    val d = 23.2F//Float类型
   ```

3. 数值类型转换时，必须要调用对应类型的转换函数，否则报错。例如

   ```kotlin
   val e = 10
   val f:Long = e.toLong()
   val g:Int = f.toInt()

   ```

## 4. 无符号类型

| 名称   | 有符号类型 | 无符号类型 |
| ------ | ---------- | ---------- |
| 字节   | Byte       | UByte      |
| 短整型 | Short      | UShort     |
| 整型   | Int        | UInt       |
| 长整型 | Long       | ULong      |
| 字符串 | String     | String     |

**无符号类型的数值范围：有符号类型去掉负数部分，正数部分再乘以 2，即无符号类型变量不能赋值负数**

## 5. 输出语句

除支持`java`中用加号拼接字符串名，还支持在字符串内直接引用表达式的方式

**格式：** <font size = 5>println("内容 ${表达式}")</font>
例：

```kotlin
val j = "321"
println("Value of j is $j , j.length is ${j.length}")
```

`println`等同于`java`中的`System.out.println`

## 6. 字符串比较

`==`：比较字符串的内容，等同于`Java`里的`equals`语句
`===`：比较字符串的引用，等同于`Java`里的`==`

```kotlin
val a = "32"
val b = "32"
a == b 返回true, a === b 返回false
```

## 7. Raw String

 ```kotlin
 val 变量名 = """
    内容
 """[.trimIndent()]
```

`trimIndent`函数作用：删除多余公共缩进

```kotlin
val n = """
        <html>
            <body>
                <H1>Hello World</H1>
            </body>
        </html>
        """.trimIndent()
```

打印结果为：

```
<html>
    <body>
        <H1>Hello World</H1>
    </body>
</html>
```

如果不加函数`trimIndent()`，则打印结果为：

        <html>
            <body>
                <H1>Hello World</H1>
            </body>
        </html>

# 三、数组

| 名称   | Kotlin         | Java                        |
| ------ | -------------- | --------------------------- |
| 整型   | IntArray        | int[]                   |
| 整型装箱   | Array<Int>     | Integer[]     |
| 字符 | CharArray | char[] |
| 字符装箱   | Array<Char>           | Character[]              |
| 字符串 | Array<String>         | String[]                      |

## 1. 数组创建

 `Java`创建方式：`int[] c = new int[]{1,2,3,4,5}`

 `Kotlin`创建方式：
    1. `val c0 = intArrayOf(1,2,3,4,5)`
    2. `val c1 = IntArray(5){it + 1}`
    说明：其中 5 表示数组长度，it 表示数组下标。
    3. `val c3 = arrayOf("Hello", "World")`

## 2. 数组长度

格式：`数组名.size`
其中数组的函数`contentToString()`，返回指定数组内容的字符串表示形式。
例：

```kotlin
val c2 = IntArray(5){it}
println(c2.size)
println(c2.contentToString())
```

打印结果为：

```
5
[0, 0, 0, 0, 0]
```

## 3. 数组读写

与`Java`一致

例：

```kotlin
val d = arrayOf("hello", "world")
d[0] = "kotlin"
println("${d[0]}, ${d[1]}")
println(d.contentToString())
```

打印结果为：

```
kotlin, world
[kotlin, world]
```

## 4. 数组遍历

### 1. java 中的 foreach 循环遍历

```java
float[] e = new float[] {1,3,5,7};
for(float elem : e) {
    System.out.println(elem);
}
```

kotlin 中如下方式：

```kotlin
val e = floatArrayOf(1F,3F,5f,7F);
for(elem in e) {
    println(elem)
}
e.forEach { elem ->
    println(elem)
}
```

## 5. 数组的包含关系

判断某元素是否在数组中
格式：`元素 [!]in 数组名`
说明：判断元素在或不在数组内，返回布尔值
例：

```kotlin
val e = floatArrayOf(1F,3F,5f,7F);
println(2f in e)
println(2f !in e)
```

打印结果：

```
false
true
```

# 四、区间

## 1. 区间创建

### 1. 闭区间

格式：**小值 .. 大值**
说明：该区间包含起止值，**只能创建升序区间**
例：

```kotlin
val intRange = 1 .. 10 //[1, 10]
val charRange = 'a' .. 'z' //['a', 'z']
val longRange = 1L .. 100L //[1L, 100L]
```

格式：**大值 downTo 小值**
说明：该区间为倒序区间，包含起止值。
例：

```kotlin
val intRangeReverse = 10 downTo 1 //[10, 1]
val charRangeReverse = 'z' downTo 'a' //['z', 'a']
val longRangeReverse = 100L downTo 1L //[100L, 1L]
println(intRangeReverse.joinToString())
```

打印：
`10, 9, 8, 7, 6, 5, 4, 3, 2, 1`

### 2. 半闭半开区间

格式：**小值 until 大值**
说明：该区间包含起始值，**只能创建升序区间**
例：

```kotlin
val intRangeExclusive = 1 until 10 //[1, 10)
val charRangeExclusive = 'a' until 'z' //['a', 'z')
val longRangeExclusive = 1L until 100L //[1L, 100L)
```

### 3. 区间的步长

在创建区间的格式再加上**step 数值**
例：

```kotlin
val intRange = 1 .. 10 step 2//[1, 3, 5, 7, 9]
val charRange = 'a' .. 'z' step 2//['a', 'c', 'e', ... , 'y']

val intRangeReverse = 10 downTo 1 step 2//[10, 8, 6, 4, 2]
val charRangeReverse = 'z' downTo 'a' step 2//['z', 'x', 'v', ... , 'b']
```

**整型、字符型、长整型区间是离散，就是一个个的数值，而 Float 类型和 Double 类型区间是连续的，包含区间内的所有数值，所以 Float 类型和 Double 类型不能使用步长**

## 2. 区间的迭代与包含

**只有离散型区间才可以迭代，格式与数组一致**
**所有区间都可以判断包含关系，格式与数组一致**

```kotlin
val intRange = 1 .. 10 step 2
val floatRange = 1f .. 2f
for(elem in intRange) {
    println(elem)
}
intRange.forEach { elem ->
    println(elem)
}
println(1.3235f in floatRange)
```

## 3. 区间应用

可以使用区间来实现`java`中`fori`循环

```kotlin
val array = intArrayOf(1, 3, 5, 7)
for(i in 0 until array.size) {
    println(array[i])
}
for (i in array.indices) {
    println("i:$i, value: ${array[i]}")
}
```

其中，数组的属性**`indices`**，表示一个 [0, array.size] 的整型区间
打印结果为：

```
1
3
5
7
i:0, value: 1
i:1, value: 3
i:2, value: 5
i:3, value: 7
```

# 五、集合

* 增加了“不可变”集合框架接口
* 复用`Java API`的所有实现类型
* 运算符级别的支持，简化集合框架的访问

<table>
    <tr>
        <td>名称</td>
        <td>Kotlin</td>
        <td>Java</td>
    </tr>
    <tr>
        <td>不可变List</td>
        <td>List&lt;T></td>
        <td rowspan="2">List&lt;T></td>
    </tr>
    <tr>
        <td>可变List</td>
        <td>MutableList&lt;T></td>
    </tr>
    <tr>
        <td>不可变Map</td>
        <td>Map&lt;K,V></td>
        <td rowspan="2">Map&lt;K,V></td>
    </tr>
    <tr>
        <td>可变Map</td>
        <td>MutableMap&lt;K,V></td>
    </tr>
    <tr>
        <td>不可变Set</td>
        <td>Set&lt;T></td>
        <td rowspan="2">Set&lt;T></td>
    </tr>
    <tr>
        <td>可变Set</td>
        <td>MutableSet&lt;T></td>
    </tr>
</table>

## 1. 集合的创建

### 1. 方法一：`Kotlin`特有方式

格式：`val 变量名:List类型 = 对应集合类型的Of方法`
例：

```kotlin
    val initList:List<Int> = listOf(1,2,3);
    val initList:MutableList<Int> = mutableListOf(1,2,3);

    val map:Map<String, Any> = mapOf("name" to "张三", "age" to 32)
    val map:Map<String, String> = mapOf("Name" to "张三", "age" to "32")
```

其中： 
1. 语句`A值 to B值`为`pair`类型，可以简单理解为`Java`中的`K-V`键值对
2. `Any`等同于`Java`中的`Object`类型

### 2. 方法二：使用`Java`中的实现类

格式：`val strList = 集合实现类`
例：

```kotlin
val strList = ArrayList<String>()
```

该方法与`Java`声明集合的只差一个`new`字段，但是引用的包名不一致。

`Java`中的`ArrayList`包名为`java.util.ArrayList`
`Kotlin`中的`ArrayList`包名为`kotlin.collections.ArrayList`
虽然包名不一致，但引用的是同一个东西。

## 2. 集合实现类复用与类型别名

`kotlin.collections`源码如下：

```kotlin
package kotlin.collections

@SinceKotlin("1.1") public actual typealias RandomAccess = java.util.RandomAccess
@SinceKotlin("1.1") public actual typealias ArrayList<E> = java.util.ArrayList<E>
@SinceKotlin("1.1") public actual typealias LinkedHashMap<K, V> = java.util.LinkedHashMap<K, V>
@SinceKotlin("1.1") public actual typealias HashMap<K, V> = java.util.HashMap<K, V>
@SinceKotlin("1.1") public actual typealias LinkedHashSet<E> = java.util.LinkedHashSet<E>
@SinceKotlin("1.1") public actual typealias HashSet<E> = java.util.HashSet<E>
```

通过**类型别名**调用`kotlin`包内的`ArrayList`等类型时等同于使用`Java`中的`ArrayList`等类型
`使用类型别名`是为了方便开发者跨平台开发。

## 3. 集合框架的操作

### 1. `List`列表


<strong>+=</strong>等价于`add`
<strong>-=</strong>等价于`remove`

`stringList.add("aa")`等同于`stringList += "aa"`
`stringList.remove("aa")`等同于`stringList -= "aa"`

```kotlin
val listStr = ArrayList<String>()
listStr += "Hello"
println(listStr)
listStr[0] = "World"
println(listStr)
val listZero = listStr[0]
println(listZero)
```
打印结果为

```
[Hello]
[World]
World
```

### 2. `Map`

访问`Map`时也可以用**方括号**的方式，`[]`内的值实际上就是`key`

```kotlin
val map = HashMap<String, Any>()
map["Hello"] = 10
println(map["Hello"])
```

打印结果为：

```
10
```

## 4. Pair键值对

### 1. 创建

```kotlin
val pair = "Hello" to "Kotlin"
val pair2 = Pair("Hello", "Kotlin")

println(pair)
println(pair2)

```

打印结果为：

```
(Hello, Kotlin)
(Hello, Kotlin)
```

### 2. 获取对应元素

```kotlin
val first = pair.first
val second = pair.second

println(first)
println(second)
```
打印结果为：

```
Hello
Kotlin
```

### 3. 解构

将`Pair`拆解成两个变量

```kotlin
val (x, y) = pair

println(x)
println(y)
```

打印结果为：

```
Hello
Kotlin
```

## 4. Triple

### 1. 创建

```kotlin
val triple = Triple("Hello", 3, 4.0)
println(triple)
```

打印结果为：

```
(Hello, 3, 4.0)
```

### 2. 获取对应元素

```kotlin

val first = triple.first
val second = triple.second
val third = triple.third
println(first)
println(second)
println(third)
```
打印结果为：

```
Hello
3
4.0
```

### 3. 解构

将`Triple`拆解成三个变量

```kotlin
val (x, y, z) = triple

println(x)
println(y)
println(z)
```

打印结果为：

```
Hello
3
4.0
```







# 附录

参考文章：[https://kotlinlang.org/docs/reference/basic-types.html](https://kotlinlang.org/docs/reference/basic-types.html)
