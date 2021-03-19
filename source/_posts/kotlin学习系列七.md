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
5. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}
    * {% post_link kotlin学习系列五 kotlin学习系列五：类型进阶 %}
    * {% post_link kotlin学习系列六 kotlin学习系列六：泛型 %}

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


# 三、 应用 -- 获取泛型实参

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

# 四、 应用 -- 为数据类实现`DeepCopy`

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


# 五、 应用 -- Model映射

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

# 六、 应用 -- 可释放对象引用的不可空类型






























