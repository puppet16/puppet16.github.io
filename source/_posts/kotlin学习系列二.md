---
title: Kotlin学习系列二：类与接口初解
date: 2020-12-15 17:32:47
tags: [Kotlin, 类型]
summary: Kotlin类与接口
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 类型进阶**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
4. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}

# 二、类与接口

## 1. 类定义

**Kotlin中类的概念与Java中类的概念一致**

**注意：**

   1. **Kotlin中不加作用域修饰符时默认<font size = 5>public</font>**  
   2. **Kotlin中类内无内容时可省略{}**
   3. **类内的变量必须初始化值，否则会报错*Variable must be initialized***
例：

```java
//java中的类定义
public class SimpleClass {

}
```

```kotlin
//kotlin中的类定义
class SimpleClass
```

### 1. Kotlin中构造方法使用关键字`constructor`

Java中构造方法名称必须与类名一致，且不添加方法返回值，而Kotlin中没有这种限制

例：

```java
//java中构造方法创建
public class SimpleClass {
    public int x;
    public SimpleClass(int x) {
        this.x = x;
    }
}
```

```kotlin
//kotlin中构造方法创建
class SimpleClass {
    var x : Int
    constructor(x:Int) {
        this.x = x
    }

    fun getValue() : Int {
        return x
    }
}

fun main() {
    val simple = SimpleClass(5)
    println(simple.getValue())
}
```

### 2.创建构造方法时类内变量分情况判断是否需要初始化值 

**当类中只有一个带入参的构造方法时，类内声明入参变量时可不初始化值，但是若有无参构造方法则声明变量时一定要初始化值**

```kotlin
//kotlin中构造方法创建
class SimpleClass {
    var x : Int = 6
    constructor() {}
    constructor(x:Int) {
        this.x = x
    }

    fun getValue() : Int {
        return x
    }
}

fun main() {
    val simple = SimpleClass(5)
    println(simple.getValue())
}
```

如上，当类`SimpleClass`中有两个构造方法，其中一个是无参构造方法，则变量`x`需要初始化值

### 3. 构造方法分为主构造器与副构造器

1. 如`java`一般，将构造方法定义在类内，则为副构造器*secondary constructor*
2. 将构造方法定义在类的定义上，则为主构造器*primary constructor*
3. 当为主构造器时，可将关键字`constructor`省略

例：

```kotlin
//副构造器
class SimpleClass {
    var x : Int;
    constructor(x:Int) {
        this.x = x;
    }
}

//主构造器
class SimpleClass constructor(x:Int) {
    var x : Int = x
}

//省略关键字的主构造器
class SimpleClass (x:Int) {
    var x : Int = x
}
```

**注意：**

1. **主构造器要求类内的其它构造器都要调用它**

   例：

   ```kotlin
    class SimpleClass constructor(x:Int) {
        var x : Int = x
        constructor():this(2) {}
    }

    class SimpleClass constructor(x:Int = 3) {
        var x : Int = x
    }
   ```

   声明一个主构造器之后又声明了一个无参副构造器，在无参副构造器执行前使用`this`先执行主构造器，或者在主构造器里为入参赋默认值也可不声明无参副构造器，而调用时不需要传参

2. **如果在主构造器内入参前加上`var`或`val`则表示声明一个属性，类内不需要再声明该属性**

    例：

    ```kotlin
    class SimpleClass constructor(var x:Int) {

    }
    ```

## 2. 类的实例化

`Kotlin`中的实例化不需要`new`

例：

```kotlin
class SimpleClass constructor(var x:Int) {

}

fun main() {
    val simple = SimpleClass(3)
}
```

## 3. 接口的定义

* `Kotlin`中接口定义使用关键字 **`interface`**

例：

```kotlin
interface SimpleInf {
    fun simpleMethod()
}
```

## 4. 接口的实现

* `Java`中接口的实现使用关键字`implements`，而`Kotlin`中使用英文冒号`:`
* `Java`中重写使用注解`@Override`，而`Kotlin`中使用关键字`override`，`Java`中重写可加可不加注解，**但`Kotlin`中一定要加`override`关键字**

例：

```java
//java接口的实现
public class SimpleClass implements SimpleInf {
    @Override
    public void simpleMethod() {

    }
}
```

```kotlin
//kotlin接口的实现，属性实现方式1
class SimpleClass(var x: Int) : SimpleInf {
    override fun simpleMethod() {

    }
}
```

## 5. 抽象类的定义

* `Kotlin`中抽象类定义使用关键字`abstract`
* **`Java`中类和方法默认可以被覆写，而`Kotlin`中默认不可覆写，需要添加关键字`open`来标识可覆写的类和方法**

例：

```java
//java抽象类的定义
public abstract class AbsClass {
    public abstract void absMethod();
    protected void overridable(){}
    public final void nonOverridable(){}
} 
```

```kotlin
//kotlin抽象类的定义及实现
abstract class AbsClass {
    abstract fun absMethod()
    open fun overridable(){}
    fun nonOverridable(){}
}

class AbsClassImpl : AbsClass() {
    override fun absMethod() {
        TODO("Not yet implemented")
    }

    override fun overridable() {
        super.overridable()
    }
}
```

## 6. 类的继承

* `Java`中接口的实现使用关键字`extends`，而`Kotlin`中使用英文冒号`:`，且类需要加一对小括号
* `Kotlin`中使用`open`标识了某方法可覆写，则类的继承不会改变这种特性，除非在该方法再加上关键字`final`

例：

```java
//java
public class SimpleClass extends AbsClass implements SimpleInf {
    @Override
    public void simpleMethod() {

    }
}
```

```kotlin
//kotlin
open class SimpleClass(var x:Int) :AbsClass(), SimpleInf {
    override fun absMethod() {
    }

    override fun simpleMethod() {
    }

    override fun overridable() {
        super.overridable()
    }
}
open class SimpleClass2 : SimpleClass(3) {
    final override fun overridable() {
        super.overridable()
    }
}

class SimpleClass3 : SimpleClass2() {
    //不可再覆写overridable方法
}
```

如上，`SimpleClass2`继承了`SimpleClass`，`SimpleClass3`继承了`SimpleClass2`。
因`SimpleClass`类中`overridable()`方法未做其它关键字修饰，则`SimpleClass2`可以覆写`overridable()`方法。
而`SimpleClass2`类中`overridable()`方法被关键字`final`修饰，表示不可再改变，则`SimpleClass3`不可以覆写`overridable()`方法。

## 7. 属性`Property`

`Java`中定义属性称之为`field`，`Kotlin`中声明的变量称之为`property`

### 1. 属性定义

若使用`var`在**类中**声明则包含三部分：**`backing field`**、**`get`方法**、**`set`方法**；若使用`val`声明因不可更改特性，则不包含`set`方法。
例：

```java
//java
public class Person {
    private int age;//field
    private String name;//field

    public int getAge() {
        return age;
    }
    public void setAge(int age) {
        this.age = age;
    }
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

```kotlin
//kotlin
class Person(age:Int, name:String) {
    val age = age//property
    var name = name//property
}
//上述Person类还可写成如下形式，以方便在属性中的get、set方法中做其它操作
class Person(age:Int, name:String) {
 
    val age = age
    get() {
        return field
    }

    var name = name
    get() {
        return field
    }
    set(value) {
        field = value
    }
}
fun main(vararg args:String) {
    val simpleClass = SimpleClass(3)
    println(simpleClass.simpleProperty)
    val person : Person = Person(3, "李四")
    println("Person:age=${person.age},name=${person.name}")
    person.name = "王五"
    //不能为age赋值，因为age被val声明
    person.age = 10
}

```

`Java`中接口定义属性，则该属性直接是`public`且`final`不可修改，而`Kotlin`在接口中定义属性不可赋值且需要在实现类里实现，因为在接口中没有`backing field`
例：

```kotlin
interface SimpleInf {
    val simpleProperty:Int//property
    fun simpleMethod()
}
```

### 2. 实现在接口中定义的属性

若`Kotlin`接口中声明了属性，则在实现类中使用必须实现它，例：

```kotlin
//kotlin接口的实现，属性实现方式1
class SimpleClass(var x: Int) : SimpleInf {
    override val simpleProperty: Int
        get() {
            return 3
        }
    override fun simpleMethod() {

    }
}
//属性实现方式二，直接写在主构造器中，初始值在类创建时传入
class SimpleClass(override val simpleProperty: Int) : SimpleInf {
    override fun absMethod() {
    }
}
```

### 3. 属性的引用

属性的引用和函数的引用是一样的都是使用双冒号`::`，例：

```kotlin
fun main() {
    val person : Person = Person(3, "李四")
    println("Person:age=${person.age},name=${person.name}")
    val ageRef = Person::age
    val nameRef = Person::name
    nameRef.set(person, "王五")

    val ageRef2 = person::age
    val nameRef2 = person::name
    nameRef2.set("王五")
}
```

其中，变量`ageRef`和`ageRef2`类型为`KProperty0<Int>`，变量`ageRef2`和`nameRef2`类型为`KMutableProperty0<String>`

# 三、 扩展

`Kotlin`能够扩展一个类的新功能而无需继承该类或者使用像装饰者这样的设计模式。例如，可以为一个不能修改的、来自第三方库中的类编写一个新的函数。 这个新增的函数就像那个原始类本来就有的函数一样，可以用普通的方法调用。 这种机制称为**扩展函数**。此外，也有**扩展属性**， 允许你为一个已经存在的类添加新的属性。

## 1. 扩展方法定义

与类外函数定义差不多，只是在函数名前加一个`receiver`，表示定义的函数是这个`receiver`类的方法。例：

```kotlin
fun main(vararg args:String) {
    val list = mutableListOf<Int>(1,2,3,4,5)
    println(list)
    list.swap(0,4)//交换下标0，4位置的元素
    println(list)
}
fun MutableList<Int>.swap(index1: Int, index2: Int) {
    val tmp = this[index1] // “this”对应该列表
    this[index1] = this[index2]
    this[index2] = tmp
}
```

如上，为`MutableList<Int>`类添加了一个扩展方法，该方法名为`swap`，作用为交换两个位置的元素。`this`关键字在扩展函数内部对应到`receiver`对象（传过来的在点符号前的对象）。


* **扩展不能真正的修改他们所扩展的类**
    通过定义一个扩展，你并没有在一个类中插入新成员， 仅仅是可以通过该类型的变量用点表达式去调用这个新函数。
    扩展其实是生成一个静态方法，其参数是它声明时点前的`receiver`, 如上为`MutableList<Int>`类添加了一个方法名为`swap`扩展方法反编译为：
    
    ```java
    public final class KotlinLabKt {
        public static final void swap(@NotNull List<Number> $this$swap, int index1, int index2) {
            Intrinsics.checkNotNullParameter($this$swap, "$this$swap");
            int tmp = ((Number)$this$swap.get(index1)).intValue();
            $this$swap.set(index1, $this$swap.get(index2));
            $this$swap.set(index2, Integer.valueOf(tmp));
        }
    }
    ```
    
* **扩展是静态的**
  扩展函数是静态分发的，即他们不是根据接收者类型的虚方法。 这意味着调用的扩展函数是由函数调用所在的表达式的类型来决定的， 而不是由表达式运行时求值结果决定的。
  如果类本身和其子类都进行了同一个函数的扩展，这函数是不会有重写关系的，在使用的时候，只会根据需要使用该方法的对象的实际类型来决定是调用了哪个，就是相当于调用静态方法。而不是动态分派。
  例：
  
  ```kotlin
    open class Shape

    class Rectangle: Shape()

    fun Shape.getName() = "Shape"

    fun Rectangle.getName() = "Rectangle"

    fun printClassName(s: Shape) {
        println(s.getName())
    }    
    printClassName(Rectangle())
  ```

  打印结果是`Shape`因为`s`的类型是`Shape`

* **成员函数与扩展函数**
  
  如果一个类定义有一个成员函数与一个扩展函数，而这两个函数又有相同的接收者类型、 相同的名字，并且都适用给定的参数，这种情况总是取成员函数。但是，**扩展函数可以重载同样名字但不同签名成员函数**
  
* **扩展的`receiver`可以为空**

    注意可以为**可空的接收者类型**定义扩展。这样的扩展可以在对象(即`Any`)变量上调用， 即使其值为`null`，并且可以在函数体内检测`this == null`，这能让你在没有检测`null`的时候调用`Kotlin`中的`toString()`。例：

    ```kotlin
    fun Any?.toString2(): String {
        if (this == null) return "null"
        // 空检测之后，“this”会自动转换为非空类型，所以下面的 toString()
        // 解析为 Any 类的成员函数
        return toString()
    }
    ```

## 2. 扩展方法的类型

```kotlin
//定义一个String类的扩展方法
fun String.times(count:Int):String {
    return count.toString()
}

fun main() {
    val str1 = (String::times)("str", 3)
    val str2 = ("*"::times)(3)
}
```

其中`str1`使用的是未绑定`receiver`的函数引用，其类型为`(String, Int) ->String`，`str2`使用的是绑定`receiver`的函数引用，其类型为`(Int) ->String`

## 3. 扩展属性

与函数类似，Kotlin 支持扩展属性，例：

```kotlin
val <T> List<T>.lastIndex: Int
    get() = size - 1
```

由于扩展没有实际的将成员插入类中，因此对扩展属性来说`backing field`(即存储空间)是无效的，不能存值。所以扩展属性不能有初值。他们的行为只能由显式提供的 getters/setters 定义。但是扩展属性可通过`this`来获取类中的成员变量。例：

```kotlin

class PoorGuy{
    var pocket: Double = 0.0
}

var PoorGuy.monkeyLeft : Double
    get() {
        return this.pocket
    }
    set(value) {
        pocket = value
    } 
```

`backing field`无效的还有在接口中定义的属性。**接口中只能定义行为，不能持有状态**

```kotlin
 interface Guy {
     var moneyLeft: Double
         get() {
             return 0.0
         }
         set(value) {

         }
 }
```

而在接口中的属性可以写get和set方法，是因为接口方法的默认实现

# 四、空类型安全

## 1. 空类型异常原因

`Kotlin` 的类型系统旨在消除来自代码空引用的危险的，`Kotlin` 中唯一可能引起空类型异常原因可能是：

1. 显式调用`throw NullPointerException()`
2. 使用了下文描述的 `!!` 操作符；
3. 有些数据在初始化时不一致，例如当：
    * 传递一个在构造函数中出现的未初始化的 `this` 并用于其他地方（“泄漏 `this`”）；
    * 超类的构造函数调用一个开放成员，该成员在派生中类的实现使用了未初始化的状态；
4. Java 互操作：
    * 企图访问平台类型的 `null` 引用的成员；
    * 用于具有错误可空性的 `Java` 互操作的泛型类型，例如一段 `Java` 代码可能会向 `Kotlin` 的 `MutableList<String>` 中加入 `null`，这意味着应该使用 `MutableList<String?>` 来处理它；
5. 由外部 `Java` 代码引发的其他问题。


声明变量时，在不支持空类型的类型后面加一个 **`?`**，表示该变量可为空，例：

```kotlin
 var nullable: String? = "Hello"
```
调用可空变量的属性时会报编译错误。

## 2. 可空变量属性调用

### 1. 非空断言运算符`!!`

**作用：** 将任何值转换为非空类型，若该值为空则抛出异常。
**注意：** 使用时一定要确定可空变量**一定不为空**。
**使用方法：** 在调用时变量后跟 **`!!`** 将变量强转为不可空类型，例：

```kotlin
val length = nullable!!.length
```

若`nullable`为空，则会报错`NullPointerException`，否则返回`nullable.length`的值，变量`length`不会为空

### 2. 安全调用操作符`?.`

**作用：** 如果变量非空，就返回调用的变量属性，否则返回`null`
**注意：** 可能返回`null`
**使用方法：** 在调用时变量后跟 **`？`** 表示可空变量的属性变量也可能为空。例：
安全调用也可以出现在赋值的左侧。这样，如果调用链中的任何一个接收者为空都会跳过赋值，而右侧的表达式根本不会求值

```kotlin
val length = nullable?.length //表示length变量也是可空的
```

若`nullable`为空，则返回`null`，否则返回`nullable.length`的值，变量`length`可能会空

### 3. **`elvis`运算符`?:`**

**作用：** 若运算符左侧非空，就返回运算符左侧表达式，否则返回运算符右侧非空值
**注意：** 当且仅当左侧为空时，才会对右侧表达式求值。

```kotlin
val length nullable?.length ?:0
```

如果`nullable.length`为空，则返回0，所以变量`length`不会为空。

## 3. 空类型的继承关系

空类型可接收对应非空类型变量

```kotlin
var x: String = "hello"
var y: String? = "world"

x = y // Type mismatch
y = x // OK

var a: Int = 0
var b: Number = 10.1

a = b // Type mismatch
b = a // OK
```

## 4. 平台类型

客观存在，不能主观定义，指`kotlin`外其它语言平台的类型，在`kotlin`中的表现为其它语言平台类型后面加`!`，例：

```java
//java
public class Person {
    public String getTitle() {
        return null;
    }
}
```

在`kotlin`中调用

```kotlin
val person = Person()
val title = person.title
```

其中变量`title`就是`Java`平台中的`String`类型，该类型表示形式为`String!`，但不可自定义。
**`kotlin`不能推断出平台类型是否为空**，即调用平台类型的属性时最好使用安全调用操作符`?.`

```kotlin
val titleLength = title.length // 直接报空指针错误
val titleLength = title?.length //
```

# 五、智能类型转换


## 1. 强制类型转换

`Kotlin`中如果之前已经判断过某变量是某类型，则不需要再强转为该类型，`Kotlin`可以自动转换为该类型，而可以直接使用。

```java
//java 定义一个接口和一个实现该接口的类
public interface Kotliner{}
public class Person implements Kotliner {
    public final String name;
    public final int age;
    
    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }
}
```

* `java`的类型转换

    ```java
    Kotliner kotliner = new Person("Lee", 20);
    if (kotliner instanceof Person) {
        System.out.println(((Person)kotliner).name);
    }
    ```

    其中已使用操作符`instanceof`判断变量`kotliner`为`Person`类型，但是在调用`name`属性时仍需要强制转换为`Person`

* `kotlin`的类型转换

    ```kotlin
    val kotliner: Kotiner = Person("Lee", 20)
    if (kotliner is Person) {
        println((kotliner as Person).name)
        println(kotliner.name)
    }
    ```
    
    **操作符`is`** 类似于`Java`中的`instanceof`，用于判断某变量是否属于某个类型
    **操作符`as`** 将某变量强制转换为某类型
    `kotlin`智能类型转换自动将变量`kotliner`转换为了`Person`类型，所以上述两个输出语句等价
    **类型自动转换作用范围：在`if`语句之内是转换后的类型，在`if`语句之外还是之前类型**

**不支持智能转换的情况**

定义的顶级`property`,在多个方法里都可以调用，虽然在某个方法中调用时判断了该变量是某个值，但是其它线程可能对它做修改，所以不支持智能转换。

```kotlin
var tag : String? = null
fun main() {
    if (tag != null) {
        println(tag.length)
    }
}
```

上述代码会在`tag.length`报错`Smart cast to 'String' is impossible, because 'tag' is a mutable property that could have been changed by this time`，即*无法将类型强制转换为“字符串”，因为“tag”是可变属性，而此时可能已更改*

## 2. 类型的安全转换

强制转换时要考虑空情况

```kotlin
val kotliner: Kotiner = Person("Lee", 20)
println((kotliner as? Person)?.name)
```

上述代码，在将变量`kotliner`通过操作符`as`强制转换为`Person`时添加了操作符`?`，表示如果强制转换失败的话返回`null`，又因变量`kotliner`强转后的结果可能为`null`，则调用`name`属性之前也要加操作符`?`以判断非空时再调用`name`属性

## 3. 智能类型转换延伸的几点建议

* 尽可能使用`val`来声明不可变引用，让程序含义更加清晰确定
* 尽可能减少函数对外部变量的访问，也为函数式编程提供基础
* 必要时创建局部变量指向外部变量，避免因它变化引起程序错误

# 六、应用--使用Retrofit访问网络请求  

```kotlin
interface GitHubApi {

    @GET("/repos/{owner}/{repo}")
    fun getRespository(@Path("owner") owner: String, @Path("repo") repo:String): Call<Respository>
}

fun main() {
    val gitHubApi = Retrofit.Builder().baseUrl("https://api.github.com")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
            .create(GitHubApi::class.java)

    val response = gitHubApi.getRespository("JetBrains", "Kotlin").execute()
    val repository = response.body()
    if (repository == null) {
        println("Error! ${response.code()} - ${response.message()}")
    } else {
        println(repository.name)
        println(repository.full_name)
        println(repository.owner.login)
        File("Kotlin.html").writeText("""
            <!DOCTYPE html>
            <html>
                <head>
                    <meta charset="UTF-8">
                    <title>${repository.owner.login} - ${repository.name}</title>
                </head>
                <body>
                    <h1><a href='${repository.html_url}'>${repository.owner.login} - ${repository.name}</a></h1>
                </body>
            </html>
        """.trimIndent())
    }

}
```

获取`kotlin`项目在`GitHub`中的一些数据，并将其生成一个网页文件。其中的`Respository`数据类可使用`Android Studio`的插件`NewDataClassAction`生成。

# 七、附录

参考文章：

[https://www.kotlincn.net/docs/reference/null-safety.html](https://www.kotlincn.net/docs/reference/null-safety.html)
[https://www.kotlincn.net/docs/reference/properties.html](https://www.kotlincn.net/docs/reference/properties.html)
[https://www.kotlincn.net/docs/reference/extensions.html](https://www.kotlincn.net/docs/reference/extensions.html)