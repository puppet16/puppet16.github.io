---
title: kotlin 学习系列六：泛型
date: 2021-03-08 15:22:26
tags: [Kotlin, 泛型]
summary: Kotlin 泛型
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 泛型**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
4. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin 学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin 学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin 学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin 学习系列四：函数进阶 %}
    * {% post_link kotlin学习系列五 kotlin 学习系列五：类型进阶 %}

# 二、泛型的基本概念

1. 泛型是一种类型层面的抽象
2. 泛型通过泛型参数实现构造更加通用的类型的能力
3. 泛型可以让符合继承关系的类型批量实现某些能力

## 1. 基本声明

使用泛型前要先用尖括号声明泛型

### 1. 函数声明泛型

```kotlin

package kotlin.comparisons

@SinceKotlin("1.1")
public actual fun <T : Comparable<T>> maxOf(a: T, b: T): T {
    return if (a >= b) a else b
}
```

**说明：**

1. 函数声明泛型，泛型的声明要放到函数名之前
2. 如上代码是比较大小的源码，比较两个泛型对象大小，返回较大的对象
3. 上述代码声明了一个泛型`T`该泛型代表的类型要实现`Comparable`接口，以确保可以比较大小

### 2. 类声明泛型

```kotlin
sealed class List<T> {
    object  Nil: List<Nothing>() {
        //...
    }
    data class Cons<E> (val head: E, val tail:List<E>): List<E>() {
        //...
    }
}
```

**说明：**

1. 在类上声明泛型时，泛型声明位于类名后
2. `Nothing`类是所有类型的子类
3. 数据类`Cons`的泛型声明，是声明了一个泛型形参，而它实现`List`类时将这个形参作为一个实参传递给了密封类`List`

### 3. 使用

声明泛型时只是声明了泛型形参，而调用时传递进去一个具体类型的实参

```kotlin
val max = maxOf("Hello", "World")
println(max)
//val list = List.Cons<Double>(1.0, List.Nil)
```

### 4. Nothing

![kotlin类型结构图](/kotlin学习系列六/kotlin_type_construct.png)

```kotlin
package kotlin
public class Nothing private constructor()
```

**说明：**

1. `Kotlin` 里所有东西都有类型，而且有明显的层级关系如上图所示
2. `Kotlin`类型层次结构的最底层就是类型 `Nothing` ，最顶层是类型`Any?`，即`Nothing`类是所有类型的子类，`Any?`类是所有类型的父类
3. `Nothing`类型的表达式计算结果是永远不会返回的 *（跟`Java` 中的 `void` 相同）*
4. `Nothing`的构造方法私有的，所以是没有实例的。可以使用`Nothing`来表示“一个不存在的值”
5. `Nothing?`可以只包含一个值：`null`
6. [参考文章1](https://www.kancloud.cn/alex_wsc/android_kotlin/1053641)  &emsp;[参考文章2](https://kotlinlang.org/docs/exceptions.html#java-interoperability)&emsp;[参考文章3](https://zhuanlan.zhihu.com/p/26890263)
7. *图中线条颜色无实际意义，只为了方便查看*

# 三、泛型约束

## 1. 添加泛型约束

**在声明泛型时通过继承父类或实现接口的方式为泛型添加约束**

```kotlin
package kotlin.comparisons

@SinceKotlin("1.1")
public actual fun <T : Comparable<T>> maxOf(a: T, b: T): T {
    return if (a >= b) a else b
}
```

```java
//java
public static <T extends Comparable<T>> T maxOf(T a, T b) {
    if (a.compareTo(b) > 0) return a;
    return b;
}
```

**说明：**

1. 如上代码所示，方法`maxOf()`比较两个泛型类型的大小，但是泛型必须是能比较大小的类型，即`T`需要实现接口`Comparable`
2. `Java`中也是通过继承父类或实现接口的方式为泛型添加约束

## 2. 添加多个约束

实现目标：比较两个类大小，并调用较大类的`invoke()`方法

```kotlin
//kotlin
fun <T> callMax(a: T, b: T) where T : Comparable<T>, T : () -> Unit {
    if (a > b) a() else b()
}
```

**说明：**

1. 通过`where`语句为泛型添加多个约束
2. 如上代码所示，泛型`T`必须实现接口`Comparable`能比较大小，而且是一个返回值是`Unit`的函数支持`invoke()`方法

```java
//java
public static <T extends Comparable<T> & Supplier<R>, R extends Number> R callMax(T a, T b) {
    if (a.compareTo(b) > 0) return a.get();
    return b.get();
}
```

**说明：**

1. 如上所示，在声明泛型`T`时为它添加了约束, 它必须是`Comparable`和`Supplier`的，并且返回结果必须是数值型的

## 3. 多个泛型参数

```kotlin
fun <T,R> callMax(a:T, b:T) where T: Comparable<T>, T:()->R, R: Number {
    if (a > b) a() else b()
}
```

**说明：**

1. 如上代码所示，声明了两个泛型：`T`代表要比较的类，`R`代表比较类的返回值是`R`
2. 要比较的类型`T`必须实现接口`Comparable`，而且是一个返回值是数值类型的`R`的函数

# 四、泛型型变

 泛型实参的继承关系对泛型类型的继承关系的影响

 泛型型变有三类：不变、协变、逆变

## 1. 不变

泛型形参没有继承关系，泛型实参也没有继承关系

```kotlin
sealed class List<T> {
    object Nil:List<Nothing>()
    //...
    data class Cons<E> (val head: E, val tail:List<E>): List<E>() {
        //...
    }
}

fun main() {
    val list = List.Cons(1.0, List.Nil)
}
```

**说明：**

1. 声明的泛型`T`之前 **没有任何修饰词** ，表示泛型型变不变，即泛型没有任何继承关系
2. `List<Nothing>`不是`List<T>`的子类，`List.Nil`也不是`List<T>`的子类
3. `List.Nil`不能满足`List<T>`，所以`list`的赋值语句会报错

## 2. 协变

### 1. 实现协变

泛型形参的继承关系与泛型实参的继承关系一致

```kotlin
sealed class List<out T> {
    object Nil:List<Nothing>()
    //...
    data class Cons<E> (val head: E, val tail:List<E>): List<E>() {
        //...
    }
}

fun main() {
    val list = List.Cons(1.0, List.Nil)
}
```

**说明：**

1. 声明泛型`T`之前添加了 **关键字`out`**，表示泛型形变是协变，即泛型的形参和实参的继承关系一致
2. 而`Noting`是任何类型的子类，作为协变类型则`List<Nothing>`是`List<T>`的子类，`List.Nil`也就是`List<T>`的子类
3. 如此`List.Nil`可以满足`List<T>`，所以`list`的赋值语句不会报错
4. 对应`Java`中的`? extend `操作

### 2. 协变点

**协变点：** 在一个类里的函数的返回值为我们定义的泛型参数，则该函数返回的泛型参数为协变点

```kotlin

interface Book

interface  EduBook:Book

class BookStore<out T: Book> {
    fun getBook(): T {
        TODO()
    }
}

fun main() {
    val eduBookStore:BookStore<EduBook> = BookStore<EduBook>()
    val bookStore: BookStore<Book> = eduBookStore

    val book : Book = bookStore.getBook()
    val eduBook: EduBook = eduBookStore.getBook()
}
```

**说明：**

1. 声明了两个类：子类教辅书`EduBook`，父类书籍`Book`
2. 声明了书店类`BookStore`，其接收一个协变的泛型参数`T`，该参数有一个约束，必须是`Book`的子类。书店类还有一个方法`getBook()`，其返回值为`T`，这个`T`即为协变点
3. `main()`方法中首先创建了一个教辅书店`eduBookStore`。之后又创建了一个书店`bookStore`, 并为其赋值前面创建的教辅书店
4. 根据 **里氏替换原则** *(子类永远可以替代父类)* 教辅书店是书店的子类，书店是其父类

### 3. 协变总结

1. 子类`Derived`兼容父类`Base`
2. 生产者`Producer<Derived>`兼容`Producer<Base>`
3. 存在 **协变点** 的类的泛型参数必须声明为 **协变或不变**
4. 当泛型类作为泛型参数类实例的 **生产者** 时用 **协变**
5. 协变就是泛型类型实例的生产者

## 3. 逆变

### 1. 实现逆变

泛型形参的继承关系与泛型实参的继承关系相反

```kotlin

package kotlin

public interface Comparable<in T> {

    public operator fun compareTo(other: T): Int
}
```

**说明：**

1. 声明泛型`T`之前添加了 **关键字`in`**，表示泛型形变是逆变，即泛型的形参和实参的继承关系相反
2. 数值类`Number`是整型类`Int`的父类
3. 能够比较`Number`的类肯定可以比较`Int`，反过来，能比较`Int`的不一定可以比较`Number`
4. 根据里氏替换原则，数值比较类`Comparable<Number>`是整型比较类`Comparable<Int>`的 **子类**
5. 对应`Java`中的`? super`操作

### 2. 逆变点

**逆变点：** 在一个类里的函数的入参为我们定义的泛型参数，则该函数形参的泛型参数为逆变点

```kotlin
open class Car

class Audi : Car()

class Factory<in T: Car> {
    fun put(t: T) {

    }
}

fun main() {
    val factory:Factory<Car> = Factory<Car>()
    val audiFactory:Factory<Audi> = factory

    val car = Car()
    val audi = Audi()

    factory.put(car)
    factory.put(audi)

//    audiFactory.put(car)//报错
    audiFactory.put(audi)
}
```

**说明：**

1. 声明了两个类：父类汽车类`Car`，子类奥迪类`Audi`
2. 声明了汽车工厂类`Factory`，其接收一个逆变的泛型参数`T`，该参数有一个约束，必须是`Car`的子类。汽车工厂类还有一个方法`put()`，其入参为`T`，这个`T`即为逆变点
3. `main()`方法中首先创建了一个汽车工厂`factory`。之后又创建了一个奥迪汽车工厂`audiFactory`, 并为其赋值前面创建的汽车工厂
4. 根据 **里氏替换原则** *(子类永远可以替代父类)* 汽车工厂是奥迪汽车工厂的子类，奥迪汽车工厂是其父类
5. 汽车工厂可以放所有的汽车，奥迪汽车工厂只能放奥迪汽车

### 3. 逆变总结

1. 子类`Derived`兼容父类`Base`
2. 消费者`Consumer<Derived>`兼容`Consumer<Base>`
3. 存在 **逆变点** 的类的泛型参数必须声明为 **逆变或不变**
4. 当泛型类作为泛型参数类实例的 **消费者** 时用 **逆变**
5. 消费者就是逆变的场景

# 五、 星投影

## 1. 定义

有些时候可以使用星号替代泛型参数

1. 星号可以在变量类型声明的位置
2. `*`可用以描述一个未知的类型
3. `*`所替换的类型在：
   * 协变点返回泛型参数上限类型
   * 逆变点接收泛型参数下限类型

### 1. 协变点示例

```kotlin
fun main() {
    val queryMap: QueryMap<*, *> = QueryMap<String, Int>()
    queryMap.getKey()
    queryMap.getValue()
}

class QueryMap<out K: CharSequence, out V : Any> {
    fun getKey(): K = TODO()
    fun getValue(): V = TODO()
}
```

**说明：**

1. 声明了一个类`QueryMap`，接收两个泛型：协变泛型`K`，约束上界为`CharSequence`; 协变泛型`V`, 约束上界为`Any`
2. 声明变量`queryMap`时类型为`QueeryMap`，但是传入的两个泛型为星号。
3. 变量`queryMap`的`getKey()`方法返回值的类型就是泛型`K`的上界，即`CharSequence`
4. 变量`queryMap`的`getValue()`方法返回值的类型就是泛型`V`的上界，即`Any`

### 2. 逆变点示例

```kotlin
fun main() {
    val f: Function<*,*> = Function<Number, Any>()
    f.invoke()
}

class Function<in P1, in P2> {
    fun invoke(p1: P1, p2: P2) = Unit
}
```

**说明：**

1. 声明了一个类`Function`，接收两个泛型：逆变泛型`P1`、逆变泛型`P2`
2. 声明变量`f`时类型为`Function`，但是传入的两个泛型为星号
3. 调用变量`f`的`invoke()`方法，时传入的两个参数的类型取泛型的下限
4. 定义泛型时没办法添加下限约束，所以是`Nothing`
5. 因为`Nothing`没有实例，所以变量`f`的`invoke()`方法无法调用

## 2. 适用范围

**`*`不能直接或间接应用在属性或函数上，即不能将`*`作为一个类型传递给泛型**

```kotlin
QueryMap<Sting,*>()
maxOf<*>(1,3)
```

**说明：**

1. 如上两种`*`的使用方式都是 **不正确的**，不能将`*`作为一个类型传递给泛型
2. 第一条语句将`*`间接作用到属性上
3. 第二条语句将`*`直接作用到函数上

**`*`适用于作为类型描述的场景**

```kotlin
val queryMap:QueryMap<*, *>
if(f is Function<*, *>){}
HashMap<String, List<*>>()
```

**说明：**

1. 类型实例化时或真正使用时可以使用`*`
2. 第一条语句：定义一个`queryMap`变量时，该变量类型是`QueryMap`类，`QueryMap`类传入的两个泛型使用`*` 描述，之后会根据`QueryMap`类里定义泛型时是协变还是逆变，来取上限还是下限
3. 第二条语句：使用操作符`is`来判断变量是什么类型时，`is`后的类型可以使用`*`，因其也是类型描述
4. 第三条语句：是一个`HashMap`的类型构造，本来是不允许使用`*`，但是`HashMap`有两个泛型参数`K`、`V`, 这两个泛型参数传的都是具体的类型：`K`对应`String`,`V`对应`List<*>`，其中`List`可以传递一个泛型参数，这个泛型参数可以使用`*`。因为`List<*>`在这个位置作为参数实际上是一个描述。
   **总结：直接的类型构造是不能用`*`，但参数里还有泛型，则该参数里的泛型可以使用`*`**

# 六、泛型实现原理及内联特化

## 1. 泛型实现原理

### 1. 泛型擦除

```kotlin
fun <T : Comparable<T>> maxOf(a: T, b: T): T {
    return if (a > b) a else b
}
```

编译后代码：

```kotlin
public static final Comparable maxOf(@NotNull Comparable a, @NotNull Comparable b) {
    return a.compareTo(b) > 0 ? a : b;
}
```

**说明：**

1. 如下代码所示，写了一个函数`maxOf()`比较两个约束为`Comparable`的泛型形参的大小
2. 编译后泛型`T`直接取的它的上限`Comparable`，此为**类型擦除**

### 2. 泛型实现对比

* **伪泛型：** 编译时擦除类型，运行时无实际类型生成。如`Java`、`Kotlin`
* **真泛型：** 编译时生成真实类型，运行时也存在该类。如`C#`、`C++`

![泛型实现对比图](/kotlin学习系列六/kotlin_genericity_comparison.png)

**说明：**

1. 在`C#`语言中，基本类型可以直接作为泛型参数，而在`Java`中不可以，只能使用基本类型的装箱类型
2. `IR` 是`intermediate representation`*（中间表示)* 的缩写
3. `Java`字节码中会将泛型类型擦除`List<String>` 、`List<Double>`只会有一个`List`。而在`C#.Net`的字节码中会真正生成`List<string`、`List<double>`两个类
4. 运行时，`Java`中内存也只会加载一个`List`，而在`C#`中内存会加载两个
5. 类型擦除会使内存占用减少，但是类型问题在运行时处理进行类型强转，增加了开销

**泛型类型无法当做真实类型**

```kotlin
fun<T> genericMethod(t: T) {
    val t = T()//报错
    val ts = Array<T>(3){TODO()}//报错
    val jclass = T::class.java//报错
    val list = ArrayList<T>()
}
```

**说明：**

1. 定义一个函数`genericMethod()`，传入一个泛型参数
2. 第一行代码会 **报错：** 因为无法确定泛型`T`是否有一个默认无参构造器
3. 第二行代码会 **报错：** `Array`虽然写成了泛型的形式，但并不是泛型，编译后也不会将泛型擦除，因为它有固定的类型`Unit`，所以泛型`T`无法作为数组的元素类型
4. 第三行代码 **报错：** 因为泛型无法作为一个真正在类型，编译时会将泛型擦除，所以无法获取泛型`T`的`class`文件
5. 第四行代码 **正确：** 因为`ArrayList`中的泛型会在编译时被擦除，所以传啥都可以
6. 而在`C#`中，以上语句都是正确的

## 2. 内联特化

### 1. 内联特化概念

`Kotlin`独有，`Java`没有。
如果一个函数是内联函数，则会将内联函数内容替换到调用内联函数的位置，一旦替换过去，则内联函数的泛型也就能确定类型

```
inline fun<reified T> genericMethod(t: T) {
    val t = T()//报错
    val ts = Array<T>(3){TODO()}
    val jclass = T::class.java
    val list = ArrayList<T>()
}

fun main() {
    genericMethod("str")
}
```

**说明：**

1. 如上代码，声明了一个内联函数`genericMethod()`。且该函数接收的泛型`T`前加 **关键字`reified`修饰**，表示该泛型可被具体化 *(reified)*
2. 因内联类具体化的特性，调用`genericMethod()`时会将该函数体放到调用位置，此时该函数的泛型`T`也能确定类型，即`T`此时特化成一个具体类型。
3. 调用`genericMethod()`时, 传入一个`String`，则内联函数的泛型`T`特化为了`String`类型
4. 第一条语句错误：虽然`T`特化为了`String`类型，但还是无法确定该类型是否有一个默认无参构造器。*但是`C#`可以添加构造器的约束*
5. 第二条语句正确：因`T`特化为了`String`类型，所以是创建了一个`String`类型的数组
6. 第三条语句正确：因`T`特化为了`String`类型，所以获取了`String`类型的`class`类

### 2. 内联特化实际应用

```kotlin
package com.google.gson;

public <T> T fromJson(String json, Class<T> classOfT){
    ...
}
```

**说明：**

1. 如上所示代码为`Gson`框架中的一个反序列化的一个方法
2. `fromJson()`方法接收一个泛型`T`参数，并返回一个该类型的结果，同时还接收了该泛型`T`类型的`class`
3. 反序列化操作步骤：构造对象 --> 填充数据 --> 返回对象
4. 而构造对象一般通过反射构造，这就必须知道该对象的`class`

```kotlin
fun main() {
    val gson = Gson()
    val garen : Person = gson.fromJson("""{"name":"盖伦", "age":20}""")
    val ashe = gson.fromJson<Person>("""{"name":"艾希", "age":18}""")
}

data class Person(val name:String, val age:Int)

inline fun <reified T> Gson.fromJson(json:String): T = fromJson(json, T::class.java)

```

**说明：**

1. 如上代码所示，创建一个`Gson`类的扩展函数，该函数为内联函数，且只需要传入一个反序列化的字符串，不需要返回值类型的`class`
2. 该函数直接内联到调用处，所以泛型`T`也就特化为一个具体的类型，也就可以获取该泛型的`class`
3. `main()`函数中声明了变量`garen`及其类型为`Person`，并调用扩展的内联函数`fromJson()`为其赋值，虽然没有为`fromJson()`中传入泛型的具体类型，但通过 **类型推导**的形式，将泛型参数的类型推导出来，进而将泛型`T`特化
4. `main()`函数中声明了变量`ashe`，并调用扩展的内联函数`fromJson()`为其赋值，为`fromJson()`中传入泛型的具体类型`Person`，直接将泛型`T`特化

# 七、 应用

## 1. 模拟 `Self Type`

### 1. `Builder`模式中的类型继承问题

```kotlin
typealias OnConfirm = () -> Unit
typealias OnCancel = () -> Unit

private val EmptyFunction = {}

open class Dialog(val title: String, val content: String)

class ConfirmDialog(title: String, content: String,
                    val onConfirm: OnConfirm,
                    val onCancel: OnCancel) : Dialog(title, content)

open class DialogBuilder {
    protected var title = ""
    protected var content = ""

    fun title(title: String): DialogBuilder {
        this.title = title
        return this
    }

    fun content(content: String): DialogBuilder {
        this.content = content
        return this
    }

    open fun build() = Dialog(this.title, this.content)
}

class ConfirmDialogBuilder : DialogBuilder() {
    private var onConfirm: OnConfirm = EmptyFunction
    private var onCancel: OnCancel = EmptyFunction

    fun onConfirm(onConfirm: OnConfirm): ConfirmDialogBuilder {
        this.onConfirm = onConfirm
        return this
    }

    fun onCancel(onCancel: OnCancel): ConfirmDialogBuilder {
        this.onCancel = onCancel
        return this
    }

    override fun build() = ConfirmDialog(title, content, onConfirm, onCancel)
}

fun main() {
    val confirmDialog = ConfirmDialogBuilder()
            .title("提交弹窗")
            .onCancel { //这行开始报错
                println("点击取消")
            }
            .content("确定提交吗？")
            .onConfirm {
                println("点击确定")
            }
            .build()
    
    confirmDialog.onConfirm()
}
```

**说明：**

1. 声明了一个弹窗基类`Dialog`，该类提供了标题`title`和内容`content`两个属性
2. 声明了一个继承弹窗基类的子类提交弹窗类`ConfirmDialog`，该类除了基类的两个属性外又添加了两个属性：确定点击事件`onConfirm()`、取消点击事件`onCancel`
3. 声明了一个`Builder`模式的基类`DialogBuilder`，用以构造`Dialog`，基类中的方法返回基类类型
4. 声明了一个继承基类`DialogBuilder`的子类`ConfirmDialogBuilder`，用以构造`ConfirmDialog`
5. **问题：** 子类 `Builder` 中的函数返回的类型是基类 `Builder` 的类型，所以使用子类`Builder`创建出来的子类对象`ConfirmDialog`无法使用其独有的函数，即在`main()`方法中在`onCancel`处的代码会错：`Unresolved reference: onCancel`

### 2. `Scala`的`Self Type`

`Scala`语言的`Self Type`可以解决`Builder`类继承后类型错误问题

```scala

object Main {
    type OnConfirm = () => Unit
    type OnCancel = () => Unit

    final val EmptyFunction = () => {}

    class Dialog(val title: String, val content: String)

    class ConfirmDialog(
                        title: String,
                        content: String,
                        val onConfirm: OnConfirm,
                        val onCancel: OnCancel
                    ) extends Dialog(title, content)

    class DialogBuilder {
        self =>
        protected var title: String = ""
        protected var content: String = ""

        def title(title: String): self.type = {
            this.title = title
            return this
        }

        def content(content: String): self.type = {
            this.content = content
            return this
        }

        def build() = new Dialog(this.title, this.content)
    }

    class ConfirmDialogBuilder extends DialogBuilder() {
        private var onConfirm: OnConfirm = EmptyFunction
        private var onCancel: OnCancel = EmptyFunction

        def onConfirm(onConfirm: => Unit): ConfirmDialogBuilder = {
            this.onConfirm = () => {
                onConfirm
            }
            return this
        }

        def onCancel(onCancel: => Unit): ConfirmDialogBuilder = {
            this.onCancel = () => {
                onCancel
            }
            return this
        }

        override def build() = new ConfirmDialog(title, content, onConfirm, onCancel)
    }

    def main(args: Array[String]): Unit = {
        new ConfirmDialogBuilder()
            .title("Hello")
            .onCancel {
                println("onCancel")
            }
            .content("World")
            .onConfirm {
                println("onConfirm")
            }
            .build()
            .onConfirm()
    }

}

```

**说明：**

1. 如上为`Scala`语言代码，其中`def`相当于`kotlin`的`fun`，该函数返回类型为 `self.type`。其中 `self`在类后添加了声明，表示该类在使用过程中真实的类型，如果是子类的话，就是子类类型。
2. 在`Kotlin`中可以使用泛型来实现`Self Type`

### 3. 使用泛型模拟`Self Type`

```kotlin
typealias OnConfirm = () -> Unit
typealias OnCancel = () -> Unit

private val EmptyFunction = {}

open class Dialog(val title: String, val content: String)

class ConfirmDialog(title: String, content: String,
                    val onConfirm: OnConfirm,
                    val onCancel: OnCancel) : Dialog(title, content)

interface SelfType<Self> {
    val self: Self
        get() = this as Self
}

open class DialogBuilder<Self : DialogBuilder<Self>> : SelfType<Self> {
    protected var title = ""
    protected var content = ""

    fun title(title: String): Self {
        this.title = title
        return self
    }

    fun content(content: String): Self {
        this.content = content
        return self
    }

    open fun build() = Dialog(this.title, this.content)
}

class ConfirmDialogBuilder : DialogBuilder<ConfirmDialogBuilder>() {
    private var onConfirm: OnConfirm = EmptyFunction
    private var onCancel: OnCancel = EmptyFunction

    fun onConfirm(onConfirm: OnConfirm): ConfirmDialogBuilder {
        this.onConfirm = onConfirm
        return this
    }

    fun onCancel(onCancel: OnCancel): ConfirmDialogBuilder {
        this.onCancel = onCancel
        return this
    }

    override fun build() = ConfirmDialog(title, content, onConfirm, onCancel)
}

fun main() {
    val confirmDialog = ConfirmDialogBuilder()
            .title("提交弹窗")
            .onCancel {
                println("点击取消")
            }
            .content("确定提交吗？")
            .onConfirm {
                println("点击确定")
            }
            .build()

    confirmDialog.onConfirm()
}
```

打印结果：

```kotlin
点击确定
```

**说明：**

1. 定义了一个接口`SelfType`，接收一个泛型`Self`，该接口内定义了一个成员`self`，类型就是传入的泛型类型，设置该成员的`getter`方法中将该成员强转为`Self`
2. 基类`DialogBuilder`实现接口`SelfType`，接口传入的泛型在声明时添加了约束：该泛型上界为`DialogBuilder`类，该类中的函数返回类型改为泛型`Self`
3. 子类`ConfirmDialogBuilder`，继承基类`DialogBuilder`时传入的泛型实参使用子类`ConfirmDialogBuilder`，如此使用子类`ConfirmDialogBuilder`中函数返回的就是`ConfirmDialogBuilder`，则子类创建出来的子类`ConfirmDialog` 就可以使用子类`ConfirmDialog`独有的函数


## 2. 基于泛型实现`Model`实例的注入





# 八、参考文章

1. [Kotlin 中的 Nothing 和 Unit](https://zhuanlan.zhihu.com/p/26890263)
2. [Java和Kotlin中泛型的协变、逆变和不变](https://www.jianshu.com/p/0c2948f7e656)
3. [https://www.kotlincn.net/docs/reference/generics.html](https://www.kotlincn.net/docs/reference/generics.html)
4. [Scala之自身类型(Self Type)](https://blog.csdn.net/bluishglc/article/details/60739183)
