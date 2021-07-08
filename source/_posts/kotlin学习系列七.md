---
title: kotlin学习系列七：反射
date: 2021-03-15 11:03:02
tags: [Kotlin, 反射]
summary: Kotlin 反射
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 反射**
2. *本文是对[Bennyhuo老师](https://github.com/enbandari)讲解的`Kotlin`系列视频的总结笔记*
3. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
4. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
5. **Kotlin源码：[https://github.com/JetBrains/kotlin](https://github.com/JetBrains/kotlin)**
6. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}
    * {% post_link kotlin学习系列五 kotlin学习系列五：类型进阶 %}
    * {% post_link kotlin学习系列六 kotlin学习系列六：泛型 %}
    * {% post_link kotlin学习系列八 kotlin学习系列八：注解 %}
    * {% post_link kotlin学习系列九 kotlin学习系列九：协程初解 %}
    * {% post_link kotlin学习系列十 kotlin学习系列十：协程进阶 %}
    * {% post_link kotlin学习系列十一 kotlin学习系列十一：协程应用 %}

# 二、反射的基本概念

## 1. 反射概念

1. 反射是允许程序在运行时访问程序结构的一类特性
2. 程序结构包括：类、接口、方法、属性等语法特性

## 2. 反射的依赖

![kotlin反射依赖](/kotlin学习系列七/kotlin_reflection_dependences.png)

**说明：**

1. `Java`中有反射的概念，`Java`的`JDK`中包含了`Java`的反射`API`
2. 对于`Kotlin`来说，`Kotlin`自己做了一套反射的`API`,是一个独立的库

### 1. 添加反射库依赖

```kotlin
dependencies {
    implementation(kotlin("stdlib-jdk8", KotlinCompilerVersion.VERSION))
    implementation("org.jetbrains.kotlin:kotlin-reflect")
}
```

**说明：**

1. 使用`Kotlin`的反射，需要单独引入依赖库`kotlin-reflect`
2. `stdlib`是`Kotlin`的标准库

## 3. 反射常见用途

1. 列出类型的所有属性 *(java->filed，kotlin->property)*、方法 *(java->method，kotlin->funtion)*、内部类等等
2. 调用给定名称及签名的方法或访问指定名称的属性
3. 通过签名信息获取泛型实参的具体类型
4. 访问运行时注解及其信息完成注入或者配置操作

## 4. 反射常用的数据结构

### 1. kotlin常用反射数据结构

|数据结构|概念及使用说明|
|--|--|
|KType|描述未擦除的类型或泛型参数等，例如：Map<String,Int>;</br>可通过typeOf或者以下类型获取对应的父类、属性、函数参数等|
|KClass|描述对象的实际类型，不包含泛型参数，例如：Map; </br> 可通过对象、类型名直接获得|
|KProperty|描述属性，可通过属性引用、属性所在类的KClass获取|
|KFunction|描述函数，可通过函数引用、函数所在类的KClass获取|

**说明：**

1. `KType`描述的是类型，与写出来的类型是一致的，而`KClass`描述的是真实的类型，编译完成后的泛型擦除掉的类型。
2. 属性包括`backing-field`、`getter`、`setter`，`KProperty`有对应属性可以获取到`getter`、`setter`

### 2. kotlin与java的反射数据结构对比

![kotlin与java的反射数据结构对比](/kotlin学习系列七/reflection_compare.png)

**说明：**

1. `KType`基本对应`Java`中的`Type`，使用场景也类似
2. `KClass`对应`Java`中的`Class`
3. `KProperty`与`Java`中的`Field`作用类似，但是`KProperty`功能更强大
4. `KFunction`与`Java`中的`Method`作用类似。`Java`中没有函数的概念，只有方法的概念。而`Function`既可以当顶级函数使用，也可以当作方法使用

## 5. Kotlin中使用反射

### 1. 在`Kotlin`中使用`Java`反射和`Kotlin`反射优缺点

`Kotlin`中也可以使用`Java`反射

* `Java`反射
  * 优点：无需引入额外依赖，首次使用速度相对较快
  * 缺点：无法访问Kotlin语法特性，需对Kotlin生成的字节码足够了解

* `Kotlin`反射
  * 优点：支持访问Kotlin几乎所有特性，API设计更友好
  * 缺点：引用Kotlin反射库(2.5MB，编译后400KB)，首次调用慢

`Kotlin`反射首次调用慢的原因：`Kotlin`反射信息是写到注解`@Metadata`中，而这些信息序列化为二进制，首次调用会获取注解，并将信息反序列化。

### 2. 使用`Kotlin`反射

```java
//java
public static void main(String[] args) throws NoSuchFieldException {
    Class<String> cls = String.class;
    Class<? extends Object> clsOfObj = String.class;
    Field field = cls.getDeclaredField("value");
}
```

```kotlin
//kotlin
@ExperimentalStdlibApi
fun main() {
    var cls: KClass<String> = String::class
    val mapCls = Map::class
    println(mapCls)
    val property = cls.declaredMemberProperties.firstOrNull()
    val mapType = typeOf<Map<String, Int>>()
    cls.members.forEach {
        println(it.name)
    }
}
```

**说明：**

1. 如上所示，先获取`String`类的`KClass`，然后通过方法`declaredMemberProperties`获取声明在`String`类中的属性
2. 使用`::class`获取某个类的`KClass`
3. `Java`的`Class`与`Kotin`的`KClass`相互转化:
   * `Java`的`Class`转换为`KClass`：在`Class`类后加`.kotlin`
   * `Kotlin`的`KClass`转换为`Class`：在`KClass`类后加`.java`

### 3. `KClass`类中几个函数

* `is`开头的函数：判断是否为某个类型。例：`isAbstract`*(是不是抽象类)*、`isCompanion`*(是不是伴生对象)*
* `nestedClasses()`：获取内部类
* `objectInstance()`：获取`object`实例，如果调用的`KClass`是单例类，可通过该方法直接获取单例类的实例
* `supertypes()`：获取父类的`KType`列表。
* `declaredMemberProperties`只能获取声明在某个类中的属性，拿不到该类的扩展属性。

    ```kotlin
    var cls: KClass<String> = String::class
    cls.declaredMemberProperties
    ```

    **说明：**

    1. 代码`cls.declaredMemberProperties`的返回类型为`Collection<KProperty1<String, *>>`
    2. `KProperty`与`KFunction`后数字释义：
        * 如果没有`receiver`，则是`KProperty0`与`KFunction0`
        * 如果有一个`receiver`，则是`KProperty1`与`KFunction1`
        * 如果有两个`receiver`，则是`KProperty2`与`KFunction2`

* `declaredMemberExtensionProperties`只能获取 **声明在某个类中**的扩展属性，获取的扩展属性不是该类的扩展属性

   ```kotlin
    class A {
        fun String.printStar() {

        }
    }
   ```

   **说明：** 如上代码所示，在类`A`中为类`String`定义了一个扩展函数`printStar()`，则代码`A::class.declaredMemberExtensionFunctions`可以获取到扩展函数`printStar()`，但是`String.class.declaredMemberExtensionFunctions`获取不到。函数`printStar()`就有两个`receiver`：`A`，`String`

### 4. `KClass`与`KType`区别

```kotlin
fun main() {
    val mapCls = Map::class
    println(mapCls)
    val mapType = typeOf<Map<out String, Int>>()
    mapType.arguments.forEach {
        println(it)
    }
}
```

打印结果：

```java
class kotlin.collections.Map
out kotlin.String
kotlin.Int
```

**说明：**

1. `Map::class`获取的`KClass`打印出来就只是该类型的`class`
2. `mapType.arguments`返回值类型是`List<KTypeProjection>`，打印每个`KTypeProjection`
3. `KTypeProjection`的构造器中有两个参数`Kvariance`*(使用处的型变声明)*、`KType`*(泛型实参类型)*
4. 在获取`KType`时，为`Map`的第一个泛型`String`添加了逆变声明`out`，所以在打印类型时会先打印出参数的型变类型，之后再打印出类型，如果没有定义型变则只显示类型

### 5. 类型擦除

```kotlin
class Person(val name: String, val age: String)

interface Api {
    fun getUsers(): List<Person>
}
```

上述代码的关键二进制代码如下：

```java
public abstract interface cn/ltt/projectcollection/kotlin/kotlinreflection/Api {


  // access flags 0x401
  // signature ()Ljava/util/List<Lcn/ltt/projectcollection/kotlin/kotlinreflection/Person;>;
  // declaration: java.util.List<cn.ltt.projectcollection.kotlin.kotlinreflection.Person> getUsers()
  public abstract getUsers()Ljava/util/List;
  @Lorg/jetbrains/annotations/NotNull;() // invisible
    LOCALVARIABLE this Lcn/ltt/projectcollection/kotlin/kotlinreflection/Api; L0 L1 0

  @Lkotlin/Metadata;(mv={1, 4, 2}, bv={1, 0, 3}, k=1, d1={"\u0000\u0014\n\u0002\u0018\u0002\n\u0002\u0010\u0000\n\u0000\n\u0002\u0010 \n\u0002\u0018\u0002\n\u0000\u0008f\u0018\u00002\u00020\u0001J\u000e\u0010\u0002\u001a\u0008\u0012\u0004\u0012\u00020\u00040\u0003H&\u00a8\u0006\u0005"}, d2={"Lcn/ltt/projectcollection/kotlin/kotlinreflection/Api;", "", "getUsers", "", "Lcn/ltt/projectcollection/kotlin/kotlinreflection/Person;", "app_debug"})
  // compiled from: KotlinReflectionLab.kt
}

```

**说明：**

1. 可以看到在类`Api`中的`getUsers()`函数返回结果是`Person`类型的集合，而编译后该方法返回值是`List`,将`Person`擦除了。
2. 可以通过`signature`拿到这个类型`Person`。
3. 混淆时，`proguard`会将`signature`混淆，所以要`keep`住`signature`才可以拿到泛型实参。
   `Proguard`配置：`-keepattributes Signature`


# 三、 示例 -- 获取泛型实参

## 1. 获取`Function`返回值的类型实参

### 1. 通过`Kotlin`反射获取

```kotlin
class Person(val name:String, val age: String)

interface Api {
    fun getUsers(): List<Person>
}
fun main() {
    //方法一
    Api::class.declaredMemberFunctions.first { it.name == "getUsers" }
            .returnType.arguments.forEach {
                println(it)
            }
    //方法二
    Api::getUsers.returnType.arguments.forEach { println(it) }
}
```

打印结果：

```java
cn.ltt.projectcollection.kotlin.kotlinreflection.Person
cn.ltt.projectcollection.kotlin.kotlinreflection.Person
```

**说明：**

1. 方法一：通过`declaredMemberFunctions`方法获取`Api`类中的`KFunction`集合，然后通过函数名称过滤得到`getUsers()`函数的`KFunction`，之后调用`returnType`函数返回`KFunction`中的`KType`，然后通过`KType`拿到泛型的实参`Person`
2. 方法二：直接拿到`getUsers()`的函数引用`KFunction1<Api, List<Person>>`，之后调用`returnType`函数返回`KFunction`中的`KType`，然后通过`KType`拿到泛型的实参`Person`
3. `first`函数作用是返回第一个满足条件的对象

### 2. 通过`Java`反射获取

```kotlin
class Person(val name:String, val age: String)

interface Api {
    fun getUsers(): List<Person>
}
fun main() {
    (Api::class.java.getDeclaredMethod("getUsers")
            .genericReturnType as ParameterizedType).actualTypeArguments.forEach { println(it) }
}
```

打印结果：

```java
class cn.ltt.projectcollection.kotlin.kotlinreflection.Person
```

**说明：**

1. 先拿到`Java`中的`class`文件，再通过方法`getDeclaredMethod()`得到函数`getUsers()`的`Method`，再调用方法`genericReturnType`返回`getUsers()`的`Type`，之后将`Type`强转为`ParameterizedType`,再调用方法`actualTypeArguments`得到返回参数的列表，然后得到泛型的实参`Person`
2. `ParameterizedType`是`Type`的子接口，代表泛型参数

**优化强转代码：**

```kotlin
fun main() {
    Api::class.java.getDeclaredMethod("getUsers")
            .genericReturnType.safeAs<ParameterizedType>()?.actualTypeArguments?.forEach { println(it) }
}

fun<T> Any.safeAs(): T? {
    return this as? T
}
```

**说明：**

1. 对任何类型声明了一个安全强转方法`safeAs()`，如果转换失败返回`null`
2. 如上代码所示，使用安全强转方法`safeAs()`就可以在强转时不再添加括号，增强可读性

## 2. 在抽象父类里获取该父类泛型的实参

```kotlin

abstract class SuperType<T> {
    val typeParameter by lazy {
        this::class.supertypes.first().arguments.first().type!!
    }
    val typeParameterJava by lazy {
        this.javaClass.genericSuperclass.safeAs<ParameterizedType>()!!.actualTypeArguments.first()
    }
}

class SubType : SuperType<String>()


fun<T> Any.safeAs(): T? {
    return this as? T
}

fun main() {
    val subType = SubType()
    subType.typeParameter.let(::println)
    subType.typeParameterJava.let(::println)
}
```

打印结果：

```java
kotlin.String
class java.lang.String
```

**说明：**

1. 方法一使用`Kotlin`反射：父类里的`this`一定是子类的实例，所以`this::class`获取的是子类`SubType`的`KClass`，之后调用`supertypes`获取继承的父类的集合，再调用`first()`方法获取在声明子类时第一个继承的父类的`KType`，之后再调用`arguments`获取父类的泛型参数列表，调用`first()`取第一个泛型参数`KTypeProjection`，最后取他的`type`属性值
2. 方法二使用`Java`反射：调用`this.javaClass`获取`java`的`class`实例，之后调用`genericSuperclass`获取带有泛型实参的`Type!`，之后调用`safeAs()`方法将`Type!`类型转为`ParameterizedType`类型，再调用方法`actualTypeArguments`得到参数的列表，再调用`first()`取第一个参数
3. 打印结果一个是`Kotlin`的`String`，一个是`Java`的`String`。`Kotlin`的`String`的编译完成后会编译为`Java`的`String`,所以在`Java`的反射中所有的`Kotlin`类型都是`Java`类型

# 四、 示例 -- 为数据类实现`DeepCopy`

定义两个数据类`Person`、`Group`，`Group`是`Person`的一个参数

```kotlin
data class Person(val name:String = "Lee", val age:Int = 18, val group: Group)

data class Group(val id:Int, val name:String, val location:String)
```

## 1. 数据类自带的`copy`方法只是浅拷贝

```kotlin
fun main() {
    val person = Person(group = Group(1,"student","beijing"))
    val copiedPerson = person.copy()
    println(person === copiedPerson)
    println(person.name === copiedPerson.name)
    println(person.group === copiedPerson.group)
}
```

**打印结果：**

```java
false
true
true
```

**`Person`类反编译`class`文件后部分代码如下：**

```java
public final class Person {
    ...
   @NotNull
   public final Person copy(@NotNull String name, int age, @NotNull Group group) {
      Intrinsics.checkNotNullParameter(name, "name");
      Intrinsics.checkNotNullParameter(group, "group");
      return new Person(name, age, group);
   }

   // $FF: synthetic method
   public static Person copy$default(Person var0, String var1, int var2, Group var3, int var4, Object var5) {
      if ((var4 & 1) != 0) {
         var1 = var0.name;
      }

      if ((var4 & 2) != 0) {
         var2 = var0.age;
      }

      if ((var4 & 4) != 0) {
         var3 = var0.group;
      }

      return var0.copy(var1, var2, var3);
   }
}
```

**说明：**

1. `Kotlin`中`==`等价于`Java`中的`equals()`方法，而`===`是比较内存地址
2. 由反编译后的代码可知，调用`Person`默认的`copy`方法时会调用`copy$default()`，之后调用`Person`的`copy()`函数，传入三个参数，这三个参数，就是`Person`自身的三个值，所以虽然创建了一个新的`Person`类，但是`Person`类里的数据类`Group`的还是一样的

## 2. 为数据类添加深拷贝方法

```kotlin
fun <T : Any> T.deepCopy(): T {
    if(!this::class.isData){
        return this
    }
    return this::class.primaryConstructor!!.let {
        primaryConstructor ->
        primaryConstructor.parameters.map { parameter ->
            val value = (this::class as KClass<T>).memberProperties.first { it.name == parameter.name }
                .get(this)
            if((parameter.type.classifier as? KClass<*>)?.isData == true){
                parameter to value?.deepCopy()
            } else {
                parameter to value
            }
        }.toMap()
            .let(primaryConstructor::callBy)
    }
}

fun main() {
    val deepCopiedPerson = person.deepCopy()
    println(person === deepCopiedPerson)
    println(person.name === deepCopiedPerson.name)
    println(person.group === deepCopiedPerson.group)
    println(deepCopiedPerson)
}
```

**打印结果：**

```java
false
true
false
Person(name=Lee, age=18, group=Group(id=1, name=student, location=beijing))
```

**数据类扩展函数`deepCopy()`说明：**

1. 只为数据类添加深拷贝的方法，所在通过语句`this::class.isData`判断调用该方法的类是不是数据类
2. 数据类一定有主构造器，可以通过`this::class.primaryConstructor`获取数据类主构造器
3. 将`primaryConstructor`作为参数传入`let`函数中，再通过`parameters`获取主构造器中的参数列表
4. 再调用`map`函数，将主构造器中的每个参数做一些操作，函数返回的是键值对列表
5. `this::class.memberProperties`语句得到某个类的成员属性列表，之后取第一个过滤条件成功的成员属性，过滤条件为该成员属性名字与之前取的参数列表中的参数名称一致
6. `this::class`中的返回结果类型是`KClass<out T>`，其中泛型是协变的，而调用`KProperty`的`get()`函数时要将泛型传入，此时泛型要是逆变的，而数据类没有继承关系，所以将`this::class`强转为`KClass<T>`，之后就可以通过`get()`函数将`receiver`传入，得到该属性的值了
7. 此时需要判断属性的值若是基本类型，则直接赋值，若是数据类则深拷贝
8. `parameter.type.classifier`获取`parameter`的`KClass`，再判断`KClass`如果是数据类则需要进行深拷贝，若是其它的类型则直接赋值
9. 通过`map`函数获取了`parameter`和其对应值的列表后，再调用`toMap()`函数，将列表转为集合
10. 将集合作为参数传入`let`方法中，最后调用构造器的`callBy`方法创建该类的实例，`callBy`方法入参即为`parameter`和其对应值的集合，所以可以使用`::`代替`lambda`表达式
11. 若代码块仅包含以 `it` 作为参数的单个函数，则可以使用方法引用 `::` 代替 `lambda` 表达式

**深拷贝思路：**

1. 先判断如果是数据类再进行深拷贝
2. 深拷贝就是构造对象，所以先获取数据类的主构造器
3. 再去获取主构造器的参数，数据类的主构造器的参数同时也是数据类的属性
4. 根据参数的名字拿到属性名字再拿到属性的值
5. 再判断属性值的类型：若为基本类型，直接取值；若是数据类，则进行深拷贝 *(这里使用了递归的思路)*
6. 之后调用主构造器的调用方法，并将主构造器的参数和参数值传入。
7. 最后获得了一个新的数据类

**打印结果说明：**

1. 通过反射创建新的`Person`实例，所以两个实例不会相等
2. 基本数据类型有一个常量共享池，所以基本数据类型的参数相等
3. 对数据类`Person`进行深拷贝时，也对它的参数数据类`Group`进行了深拷贝，而深拷贝使用反射创建新的实例，所以两个`Group`实例不相等
4. 打印出深拷贝的`Person`实例，与一开始创建的`Person`实例里的基本数据都是一致的

# 五、 示例 -- Model映射

>在工程中可能会有数据类之间转换的需要，本章讲述如何通过反射进行类之间快速的转换

```kotlin
data class Human(val name: String, val avatarUrl: String)

data class Person(
        var id: Int,
        var name: String,
        var avatarUrl: String,
        var smallUrl: String,
        var detailUrl: String
)
```

如上定义两个数据类`Human`、`Person`

## 1. 通过`Map`转为任意类型

```kotlin

fun main() {

    val personMap = mapOf(
            "id" to 0,
            "login" to "Lee",
            "avatarUrl" to "https://api.github.com/users/Lee",
            "detailUrl" to "https://api.github.com/users/Lee"
    )

    val personFromMap: Person = personMap.mapAs()
    println(personFromMap)
}


inline fun <reified To : Any> Map<String, Any?>.mapAs(): To {
    return To::class.primaryConstructor!!.let {
        it.parameters.map {
            parameter ->
            parameter to (this[parameter.name] ?: if(parameter.type.isMarkedNullable) null
            else throw IllegalArgumentException("${parameter.name} is required but missing."))
        }.toMap()
                .let(it::callBy)
    }
}
```

**打印结果：**

```java
Person(id=0, name=Lee, avatarUrl=https://avatars2.githubusercontent.com/u/30511713?v=4, smallUrl=https://api.github.com/users/Lee, detailUrl=https://puppet16.github.io/)
```

**说明：**

1. 获取要转换成的目标`To`的`KClass`实例，之后再获取该类的主构造器
2. 通过`parameters`方法获取主构造器的参数列表
3. 再通过`map`函数对参数列表里每个参数进行如下操作
4. 构造参数及参数值的键值对，若参数值为`null`，则判断若参数类型可以为空，则返回空，否则直接抛异常
5. 之后通过`toMap()`函数，将参数及参数值的键值对列表转为集合
6. 最后调用主构造器的`callBy()`函数，将之前的参数及参数值键值对集合传入其中
7. 如此便将`map`集合转换为了一个任意类型
8. **注意：** `map`的`key`值需要与转换目标的参数名称一致，且转换目标的参数要定义在主构造器中



## 2. 通过任意类型映射为任意类型

```kotlin

fun main() {
    val person = Person(
            0,
            "Lee",
            "https://avatars2.githubusercontent.com/u/30511713?v=4",
            "https://api.github.com/users/Lee",
            "https://puppet16.github.io/"
    )

    val human: Person = person.mapAs()
    println(human)
}

inline fun <reified From : Any, reified To : Any> From.mapAs(): To {
    return From::class.memberProperties.map { it.name to it.get(this) }
            .toMap().mapAs()
}

```

**打印结果：**

```java
Person(id=0, name=Lee, avatarUrl=https://api.github.com/users/Lee, smallUrl=https://api.github.com/users/Lee, detailUrl=https://api.github.com/users/Lee)
```

**说明：**

1. 获取被转换的类`From`的成员属性列表，之后将列表转换为集合，集合键为属性名称，值为属性值，最后再调用上一节所讲的`Map`的转换函数`mapAs()`
2. **注意：** 两个任意类型中的参数名称要一致，且参数要定义在主构造器中

# 六、 示例 -- 可释放对象引用的不可空类型

>创建一个引用，该引用持有一个占用较大内存资源的类型对象，在最后不需要该引用时，我们需要将其释放，也就是将引用置为`null`，`Kotlin`中只有声明为可空类型才能置为`null`，但是可空类型属性使用比较繁琐，可以通过`属性代理`将定义不可空类型引用释放

```kotlin

class ReleasableNotNull<T: Any>: ReadWriteProperty<Any, T> {
    private var value: T? = null

    override fun getValue(thisRef: Any, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("Not initialized or released already.")
    }

    override fun setValue(thisRef: Any, property: KProperty<*>, value: T) {
        this.value = value
    }

    fun isInitialized() = value != null

    fun release() {
        value = null
    }
}

inline val KProperty0<*>.isInitialized: Boolean
    get() {
        isAccessible = true
        return (this.getDelegate() as? ReleasableNotNull<*>)?.isInitialized()
            ?: throw IllegalAccessException("Delegate is not an instance of ReleasableNotNull or is null.")
    }

fun KProperty0<*>.release() {
    isAccessible = true
    (this.getDelegate() as? ReleasableNotNull<*>)?.release()
        ?: throw IllegalAccessException("Delegate is not an instance of ReleasableNotNull or is null.")
}

class Bitmap(val width: Int, val height: Int)

class Activity {
    private var bitmap by ReleasableNotNull<Bitmap>()

    fun onCreate(){
        println(this::bitmap.isInitialized)
        bitmap = Bitmap(1920, 1080)
        println(::bitmap.isInitialized)
    }

    fun onDestroy(){
        println(::bitmap.isInitialized)
        ::bitmap.release()
        println(::bitmap.isInitialized)
    }
}

fun main() {
    val activity = Activity()
    activity.onCreate()
    activity.onDestroy()
}
```

打印结果：

```java
false
true
true
false
```

**说明：**

1. 定义了一个自定义属性代理类`ReleasableNotNull`，该类接收一个泛型，这个泛型就是可以被释放的引用类型
2. 自定义属性代理类只需要实现`getValue()`和`setValue`方法即可，不一定非要实现`ReadWirtProperty`接口
3. 在`ReleasableNotNull`类中创建了属性`value`，类型为可空的泛型`T`
4. 在`ReleasableNotNull`类中的`getValue()`方法，如果`value`为空的话要抛出异常，因为传进来的类型不可能为空，为空的情况也是没有初始化或是手动调用`release()`方法，将该属性释放了，此时也不能再使用该类获取类型了
5. `isInitialized()`方法是判断`ReleasableNotNull`类是否初始化
6. `release()`方法是将传进来的类型释放
7. 因为`isInitialized()`和`release()`方法是定义在属性代理类`ReleasableNotNull`内的方法，而该类的属性`value`才是最终要调用这两个方法的`receiver`，所以要为属性添加两个扩展方法
8. 根据`lateInit`中对属性添加扩展方法的样式为自定义属性代理类`ReleasableNotNull`里的属性添加代理
9. `KProperty0<*>.isInitialized`是为所有无`receiver`的属性添加扩展属性`isInitialized`，获取该属性值时应该返回自定义属性代理类`ReleasableNotNull`里的`isInitialized`方法结果，所以通过`getDelegate()`方法先获取该属性的代理对象，然后再强转为`ReleasableNotNull`,若强转失败则抛出异常`IllegalAccessException`；若强转成功则直接调用`isInitialized()`方法
10. `KProperty0<*>.release()`与`KProperty0<*>.isInitialized`内部处理基本一致，只是`KProperty0<*>.release()`是为`KProperty0<*>`添加了一个扩展函数，`KProperty0<*>.isInitialized`为`KProperty0<*>`添加了一个扩展属性
11. 使用`getDelegate()`前必须加上`isAccessible = true`语句给予访问权限，否则会报错:`kotlin.reflect.full.IllegalPropertyDelegateAccessException: Cannot obtain the delegate of a non-accessible property`
12. 使用如上代码时必须要添加`kotlin`的反射库

# 七、 插件化加载类

>插件加载运行，并实现插件文件监听及热部署即自动监听`jar`包的修改，若进行了修改则重新进行加载运行里面的类

## 1. `ClassLoader`

![classLoader](/kotlin学习系列七/reflection_classloader.png)

**说明：**

1. **启动类加载器 _(BootstrapClassLoader)_：** 启动类加载器主要加载的是`JVM`自身需要的类，这个类加载使用`C++`语言实现的，是虚拟机自身的一部分，负责加载存放在 `JDK\jre\lib` *(JDK代表JDK的安装目录，下同)* 下，或被 `-Xbootclasspath`参数指定的路径中的，并且能被虚拟机识别的类库 *（如rt.jar，所有的java.开头的类均被 BootstrapClassLoader加载）*。启动类加载器是无法被`Java`程序直接引用的。
   **总结：** 启动类加载器加载`java`运行过程中的核心类库`JRE\lib\rt.jar`, `sunrsasign.jar`, `charsets.jar`, `jce.jar`, `jsse.jar`, `plugin.jar`以及存放在`JRE\classes`里的类，也就是`JDK`提供的类等。*常见的比如：Object、Stirng、List…*
2. **扩展类加载器 *(ExtClassLoader)*：** 该加载器由 `sun.misc.Launcher$ExtClassLoader`实现，它负责加载 `JDK\jre\lib\ext`目录中，或者由 `java.ext.dirs`系统变量指定的路径中的所有类库 *（如javax.开头的类）*，开发者可以直接使用扩展类加载器。
3. **应用程序类加载器 *(AppClassLoader)*：** 该类加载器由 `sun.misc.Launcher$AppClassLoader`来实现，会加载 `java` 环境变量 `CLASSPATH` 所指定的路径下的类库。而 `CLASSPATH` 所指定的路径可以通过 `System.getProperty("java.class.path")` 获取；该变量也可以覆盖，可以使用参数 `-cp`，*例如：`java -cp` 路径（可以指定要执行的 `class` 目录）*；开发者可以直接使用该类加载器，如果应用程序中没有自定义过自己的类加载器，一般情况下这个就是程序中 **默认的类加载器**。
   **总结：** 应用程序类加载器加载`CLASSPATH`变量指定路径下的类，即指你自已在项目工程中编写的类
4. **自定义类加载器 *(CustomClassLoader)*：** 该 `ClassLoader` 是指我们自定义的 `ClassLoader`，大部分情况下使用 `AppClassLoader` 就足够了。


### 1. `ClassLoader`的双亲委派机制

`ClassLoader`使用的是双亲委托模型来搜索类的，每个`ClassLoader`实例都有一个父类加载器的引用 *（不是继承的关系，是一个包含的关系）*，虚拟机内置的类加载器 *(Bootstrap ClassLoader)* 本身没有父类加载器，但可以用作其它`ClassLoader`实例的的父类加载器。
当一个`ClassLoader`实例需要加载某个类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是 **由上至下依次检查**的。

**加载步骤：**

1. 当`AppClassLoader`加载一个`class`时，它首先不会自己去尝试加载这个类，而是把类加载请求委派给父类加载器`ExtClassLoader`去完成
2. 当`ExtClassLoader`加载一个`class`时，它首先也不会自己去尝试加载这个类，而是把类加载请求委派给`BootStrapClassLoader`去完成
3. 如果`BootStrapClassLoader`加载失败 *(例如在`$JAVA_HOME/jre/lib`里未查找到该`class`）*，会使用`ExtClassLoader`来尝试加载
4. 若`ExtClassLoader`也加载失败，则会使用`AppClassLoader`来加载。
5. 如果`AppClassLoader`也加载失败，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出`ClassNotFoundException`异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的`Class`实例对象。
6. **总结：** 自底向上检查类是否已经加载，自顶向下尝试加载类

**双亲委派机制作用：**

1. 防止重复加载同一个 `.class`。通过委托去向上面问一问是否加载过了，加载过了就不用再加载一遍
2. 保证核心 `.class` 不能被篡改。假如我们自定义了一个`java.lang.Integer`类，当使用它时，因为双亲委派，会先使用`BootStrapClassLoader`来进行加载，这样加载的便是`jdk`的`Integer`类，而不是自定义的这个，避免因为加载自定义核心类而造成`JVM`运行错误。

**JVM如何判断两个`class`是否相同：**

 `JVM`在判定两个`class`是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，`JVM`才认为这两个`class`是相同的。就算两个`class`是同一份`class`字节码，如果被两个不同的`ClassLoader`实例所加载，`JVM`也会认为它们是两个不同`class`

## 2. 插件化加载类实现

为方便的实现类的加载和卸载必须要自定义`ClassLoader`。必须保证类和它的`ClassLoader`都没有被使用的情况下才可以卸载类


**实现思路：** 有一个名为`Host`的项目，它依赖了一个公共库`Common`，这个库中定义了一堆的接口，为`Host`访问插件里的类做入口。如此所有插件的入口类都是`Common`中定义的接口的实现。如此`Host`反射取到插件里的入口实现类并将其强转为`Common`里的接口后就可以访问插件内容了

![project_struct](/kotlin学习系列七/reflection_plugin_struct.png)

### 1. 创建`plugin`接口

在一个名为`plugin_common`的单独的`Module`中创建插件的入口接口

```kotlin
interface Plugin {
    companion object {
        const val CONFIG = "plugin.config"
        const val KEY = "plugin.impl"
    }

    fun start()

    fun stop()
}
```

**说明：**

1. 如上在名为`plugin_common`的单独的`Module`中创建一个接口`Plugin`，其中有两个方法`start()`、`stop()`，还
2. 该接口的伴生对象中创建了两个常量`CONFIG`、`KEY`，`CONFIG`是实现`Plugin`接口的类的配置文件名；`KEY`是配置文件中的键值名称

### 2. 创建两个继承`plugin`接口的类

在名为`plugin_1`的单独的`Module`中创建`Plugin`接口的实现类`PluginImpl1`，在名为`plugin_2`的单独的`Module`中创建`Plugin`接口的实现类`PluginImpl2`

```kotlin
class PluginImpl1: Plugin{
    override fun start() {
        println("Plugin1: Start")
        newMethod()
    }

    fun newMethod(){
        println("newMethod called!!")
    }

    override fun stop() {
        println("Plugin1: Stop")
    }
}
```

```kotlin
class PluginImpl2: Plugin{
    override fun start() {
        println("Plugin2: Start")
    }

    override fun stop() {
        println("Plugin2: Stop")
    }
}
```

**说明：**

1. 如上是两个接口`Plugin`的实现类`PluginImpl1`、`PluginImpl2`，在各自的实现方法中打印一些信息，只是在`PluginImpl1`中的`start()`方法多调用了一个方法`newMethod()`，该方法也是打印信息

### 3. 将实现类所在`Module`打包成`jar`

`plugin_1`、`plugin_2`的`build.gradle`文件内容如下：

```groovy
plugins {
    id 'java'
    id 'org.jetbrains.kotlin.jvm'
}

group 'cn.ltt.kotlin.plugin'
version '1.0-SNAPSHOT'

sourceCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk8"

    implementation project(":plugin_common")

    testCompile(group: 'junit', name: 'junit', version: '4.12')
}

compileKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
compileTestKotlin {
    kotlinOptions.jvmTarget = "1.8"
}
```

**说明：**

1. 点击`Gradle`窗口中`plugin_1\Tasks\build\assemble`条目，生成`jar`包
   ![attar](/kotlin学习系列七/reflection_project_jar_location.png)
2. `jar`包位置：`plugin_1\build\libs\plugin_1-1.0-SNAPSHOT.jar`

### 4. 为两个实现接口的类添加配置文件

因为实现`Plugiin`接口的类的名字没有要求，如果想使用反射拿到该类还需要知道类的名字，所以添加一个配置文件，该文件中写出该类的名字

```java
//plugin_1
plugin.impl=cn.ltt.kotlin.plugin1.PluginImpl1
```

```java
//plugin_2
plugin.impl=cn.ltt.kotlin.plugin1.PluginImpl2
```

**说明：**

1. 该配置文件在各自`Module`下的`resources`文件夹内，名为`plugin.config`

### 5. 递归监听文件夹

为了实现热加载插件需要实时监听文件是否有改动

```kotlin
//FileWatcher.kt

typealias FileEventListener = (file: File) -> Unit
private val EMPTY: FileEventListener = {}

@RequiresApi(Build.VERSION_CODES.O)
class FileWatcher(private val watchFile: File,
                  private val recursively: Boolean = true,
                  private val onCreated: FileEventListener = EMPTY,
                  private val onModified: FileEventListener = EMPTY,
                  private val onDeleted: FileEventListener = EMPTY) {

    private val folderPath by lazy {
        Paths.get(watchFile.canonicalPath).let {
            if (Files.isRegularFile(it)) it.parent else it
        } ?: throw IllegalArgumentException("Illegal path: $watchFile")
    }

    @Volatile
    private var isWatching = false

    private fun File.isWatched() = watchFile.isDirectory || this.name == watchFile.name

    private fun Path.register(watchService: WatchService, recursively: Boolean, vararg events: Kind<Path>) {
        when (recursively && watchFile.isDirectory) {
            true -> // register all subfolders
                Files.walkFileTree(this, object : SimpleFileVisitor<Path>() {
                    override fun preVisitDirectory(dir: Path, attrs: BasicFileAttributes): FileVisitResult {
                        dir.register(watchService, *events)
                        return FileVisitResult.CONTINUE
                    }
                })
            false -> register(watchService, *events)
        }
    }

    @Synchronized
    fun start() {
        if(!isWatching){
            isWatching = true
        }

        thread {
            // We obtain the file system of the Path
            val fileSystem = folderPath.fileSystem

            // We create the new WatchService using the try-with-resources block(in kotlin we use `use` block)
            fileSystem.newWatchService().use { service ->
                // We watch for modification events
                folderPath.register(service, recursively,
                    StandardWatchEventKinds.ENTRY_CREATE,
                    StandardWatchEventKinds.ENTRY_DELETE,
                    StandardWatchEventKinds.ENTRY_MODIFY
                )

                // Start the infinite polling loop
                while (isWatching) {
                    // Wait for the next event
                    val watchKey = service.take()

                    watchKey.pollEvents().forEach { watchEvent ->
                        // Get the type of the event
                        Paths.get(folderPath.toString(), (watchEvent.context() as Path).toString()).toFile()
                            .takeIf {
                                it.isWatched()
                            }
                            ?.let(
                                when (watchEvent.kind()) {
                                    StandardWatchEventKinds.ENTRY_CREATE -> {
                                        println("onCreated")
                                        onCreated
                                    }
                                    StandardWatchEventKinds.ENTRY_DELETE -> {
                                        println("onDeleted")
                                        onDeleted
                                    }
                                    else ->{
                                        println("onModified")
                                        onModified // modified.
                                    }
                                }
                            )
                    }

                    if (!watchKey.reset()) {
                        // Exit if no longer valid
                        break
                    }
                }
            }
        }
    }

    @Synchronized
    fun stop(){
        isWatching = false
    }
}
```

**说明：**

1. 如上代码功能为递归的监听某个文件夹和文件的修改
2. 类`FileWatcher`有三个事件：`onCreated`创建、`onModified`修改、`onDeleted`删除
3. 这三个事件的类型是`FileEventListener`，可以看到这只是一个别名，事件只是一个接口`File`类型参数，不返回结果的函数

### 6. 创建自定义类加载器

```kotlin
class PluginClassLoader(private val classPath: String) : URLClassLoader(arrayOf(File(classPath).toURI().toURL())) {
    init {
        println("Classloader init @${hashCode()}")
    }

    protected fun finalize() {
        println("Classloader will be gc @${hashCode()}")
    }
}

@RequiresApi(Build.VERSION_CODES.O)
class PluginLoader(private val classPath: String) {

    private val watcher = FileWatcher(
        File(classPath),
        onCreated = ::onFileChanged,
        onModified = ::onFileChanged,
        onDeleted = ::onFileChanged
    )

    private var classLoader: PluginClassLoader? = null
    private var plugin: Plugin? = null

    fun load() {
        reload()
        watcher.start()
    }

    private fun onFileChanged(file: File) {
        println("$file changed, reloading...")
        reload()
    }

    @Synchronized
    private fun reload() {
        plugin?.stop()
        this.plugin = null
        this.classLoader?.close()
        this.classLoader = null

        val classLoader = PluginClassLoader(classPath)
        val properties = classLoader.getResourceAsStream(Plugin.CONFIG)?.use {
            Properties().also { properties ->
                properties.load(it)
            }
        } ?: run {
            classLoader.close()
            return println("Cannot find config file for $classPath")
        }

        plugin = properties.getProperty(Plugin.KEY)?.let {
            val pluginImplClass = classLoader.loadClass(it) as? Class<Plugin>
                ?: run {
                    classLoader.close()
                    return println("Plugin Impl from $classPath: $it should be derived from Plugin.")
                }

            pluginImplClass.kotlin.primaryConstructor?.call()
                ?: run {
                    classLoader.close()
                    return println("Illegal! Plugin has no primaryConstructor!")
                }
        }

        plugin?.start()
        this.classLoader = classLoader

        System.gc()
    }

}

@RequiresApi(Build.VERSION_CODES.O)
fun main() {
    arrayOf(
        "plugin_1/build/libs/plugin_1-1.0-SNAPSHOT.jar",
        "plugin_2/build/libs/plugin_2-1.0-SNAPSHOT.jar"
    ).map {
        PluginLoader(it).apply { load() }
    }
}
```

**说明：**

1. 创建一个插件`jar`包的加载器`PluginClassLoader`，该类继承自`URLClassLoader`,`URLClassLoader`入参为`URL`类型的数组列表`URLClassLoader`可以加载`jar`文件
2. `PluginClassLoader`传入一个`jar`文件的`String`类型地址，之后将这个地址转为`URL`再将其创建为一个数组传入`URLClassLoader`中，如此就加载了`jar`文件
3. 创建插件加载器`PluginLoader`，每个插件对应一个`PluginLoader`对象。该类需要传入插件所在的地址名为`classPath`
   1. 在`PluginLoader`中创建`FileWahter`对象`watcher`，传入插件所在地址、三个对应事件，因为这三个事件都是入参为`File`不需要返回结果的函数
   2. 定义函数`onFileChanged()`，入参为`File`类型，不需要返回结果，则可将该函数放到创建对象`FileWathcer`语句中，在文件的创建、修改、删除事件都使用函数`onFileChanged`监听
   3. 如果文件有更改的话，应该重新加载文件，所以`onFileChanged()`函数中要重新加载文件
   4. 定义函数`reload()`，在该函数中真正实现加载插件的逻辑
   5. 定义函数`load()`，该函数用于开始执行文件的监听和插件的加载，是整个加载插件功能的入口
   6. 加载插件需要`PluginClassLoader`类，及加载的插件，所以要定义变量`classLoader`，类型为可空的`PluginClassLoader`；定义变量`plugin`，类型为可空的`Plugin`
   7. 重新加载时要先将之前的插件关闭，调用`plugin`的`stop`方法
   8. 每个`plugin`对应一个`PluginClassLoader`，所以在`reload`方法中去重新创建`PluginClassLoader`的实例
   9. 加载插件前要先获取该插件的配置定义属性名为`properties`，使用语句`classLoader.getResourceAsStream(Plugin.CONFIG)`获取配置文件的`InputStream`
   10. 再创建一个 `Properties`对象，通过它的`load()`方法，加载配置文件的输入流
   11. 再通过 `elves`操作符，判断如果没有拿到`Plugin.CONFIG`的配置文件，则关闭当前`ClassLoader`，打印异常信息：不能获取配置文件
   12. 通过语句`properties.getProperty(Plugin.KEY)`获取配置文件中插件的入口类名
   13. 通过语句 `classLoader.loadClass(it) as? Class<Plugin>`加载该类，并将该类强转为`Plugin`类型将其赋值给属性`pluginImplClass`，如果转换失败则关闭当前`ClassLoader`，打印异常信息：该类并未实现接口`plugin`
   14. 通过语句`pluginImplClass.kotlin.primaryConstructor?.call()`创建类实例： `classLoader.loadClass`语句获取的类是`Java`中的类，所以要先将`pluginImplClass`转为`kotlin`的类，再调用该类的主构造器的创建方法创建该类。**注意：此时默认实现接口`Plugin`的类都有主构造器**。如果创建失败则关闭当前`ClassLoader`，打印异常信息：该类没有主构造器
   15. 如此，拿到了`plugin`类实例，此时可以调用实例的`start()`方法
   16. 之后重新为`classloader`对象赋值为之前加载`plugin`类所用的`ClassLoader`
   17. 因为创建了大量的局部的变量，在最后的时候调用`System.gc()`，通知系统及时回收
   18. 对于函数来说添加注解`@Synchronized`来加锁，如此保证函数在多线程下也可以正常使用
4. `main()`方法中操作：
   1. 创建了一个包含两个插件`jar`包地址的`String`类型的数组
   2. 通过`map`函数将数组每个元素都做了如下的操作
   3. 创建`PluginLoader`类的实例，传入参数即为插件`jar`包地址，之后再调用该实例的`apply`方法，如此`PluginLoader`实例即为`apply`方法中的`receiver`，在`apply`方法中调用`PluginLoader`类的实例的`load`方法开始加载插件

5. 验证是否为实时监听热更新方法：修改插件里类的某个方法，之后再重新打包

# 八、参考文章

1. [深入分析Java ClassLoader原理](https://blog.csdn.net/xyang81/article/details/7292380#NetWorkClassLoader)
2. [这篇文章绝对让你深刻理解java类的加载以及ClassLoader源码分析](https://zhuanlan.zhihu.com/p/94847256)



























