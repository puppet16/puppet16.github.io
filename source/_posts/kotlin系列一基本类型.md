---
title: Kotlin 系列一：基本类型
date: 2020-12-07 16:51:45
tags: [Kotlin, 类型]
summary: Kotlin 基本数据类型
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 基本数据类型**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
<!--3. Kotlin 系列文章：
    * {% post_link 源码系列一 源码系列一：APP 的启动过程 %}-
-->
# 二、基本类型

| 类型   | 大小 (bits) | 最小值                                       | 最大值                                         |
| ------ | ---------- | -------------------------------------------- | ---------------------------------------------- |
| Byte   | 8          | -128                                         | 127                                            |
| Short  | 16         | -32768                                       | 32767                                          |
| Int    | 32         | -2,147,483,648 (-2<sup>31</sup>)             | 2,147,483,647 (2<sup>31</sup> - 1)             |
| Long   | 64         | -9,223,372,036,854,775,808 (-2<sup>63</sup>) | 9,223,372,036,854,775,807 (2<sup>63</sup> - 1) |
| Float  | 32         | ——                                           | ——                                             | —— |
| Double | 64         | ——           | ——                                             | —— |

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

![kotlin_type_1](kotlin系列一基本类型/kotlin_type_1.png)

**修饰符：**
**`val`**：声明只读变量，对应 java 里加了`final`修饰符的变量
**`var`**：声明可读写变量。

`Kotlin`里的`val a:Int = 2`等同于`Kotlin`里的`val a = 2`等同于`Java`里的`final int a = 2`

## 3. 类型推导

声明变量时可不加类型，`Kotlin`会根据初始化的值去自动匹配变量对应类型。

1. 整型默认为**Int**，浮点型默认为**Double**。
2. 变量声明为`Long`或`Float`时，要在初始值后加上大写`L`或`F`表示，小写会报错。例如：

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
println("Value of j is $j , j.length is ${j.length})
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


# 附录

参考文章：[https://kotlinlang.org/docs/reference/basic-types.html](https://kotlinlang.org/docs/reference/basic-types.html)
