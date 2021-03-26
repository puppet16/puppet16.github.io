---
title: kotlin 学习系列八：注解
date: 2021-03-23 16:57:07
tags: [Kotlin, 注解]
summary: Kotlin 注解
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 注解**
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
    * {% post_link kotlin学习系列七 kotlin学习系列七：反射 %}

# 二、注解的基本概念

**注解** 实际上就是一种代码标签，它作用的对象是代码。它可以给特定的注解代码标注一些额外的信息。然而这些信息可以选择不同保留时期，比如源码期、编译期、运行期。然后在不同时期，可以通过某种方式获取标签的信息来处理实际的代码逻辑，这种方式常常就是我们所说的 **反射**。

总结如下：

1. 注解是对程序的附加信息说明
2. 注解可以对类、函数、函数参数、属性等做标注
3. 注解的信息可用于源码级、编译期、运行时

## 1. 注解定义

定义注解类使用关键字 **`annotation`**

```kotlin
annotation class State
```

```java
public @interface State {

}
```

**说明：**

1. `java`中注解通过`@interface`关键字进行定义，它和接口声明类似，只不过在前面多加`@`
2. `kotlin`中注解和一般类的声明很类似，只是在`class`前面加上了`annotation`修饰符

## 2. 限定标注对象

定义注解类时，限定定义的注解的标注对象使用 **注解`@Target`**

```kotlin
@Target(AnnotationTarget.CLASS)
annotation class State
```

**说明：**

1. 如上所示，定义了一个注解`State`，限制了`State`只能在类、接口、单例上标注
2. 注解`@Target`用于限制定义的注解的标注对象
3. 常用的范围有：
   1. `CLASS`：用于类、接口、单例
   2. `CONSTRUCTOR`：用于主构造器或副构造器
   3. `PROPERTY`：用于属性
   4. `FUNCTION`：用于函数，不包括构造器

4. 注意`Target`和`AnnotationTarget`都是包`kotlin.annotation`下的类

## 3. 指定作用时机

定义注解类时，指定作用时机使用 **注解`@Retention`**

```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.CLASS)
annotation class State
```

**说明：**

1. 如上所示，定义的注解`State`，只能标注在类、接口、单例上，同时指定在运行时再去解释注解
2. 时机范围：
   1. `SOURCE`：代表这个注解在编译器处理完之后就被抛弃，这意味着它的信息仅存在于编译器处理期间，不会存到`.class`文件中
   2. `BINARY`：对应`Java`中的`CLASS`，代表编译器将注解存储于类对应的`.class`文件中，但是在运行时不能通过反射读取，**在`Java`中这是`Annotation`的默认行为**
   3. `RUNTIME`：代表编译器将注解存储于`.class`文件中，并且 **可由反射获取**，由于`Kotlin`的语言特性，`它是`Kotlin`中的默认策略`

## 4. 添加参数

可以为注解类添加参数，与一般类添加主构造器参数一致

```kotlin
@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.CLASS)
annotation class State(val status:String)
```

**说明：**

1. 注解类的参数类型有限制只能是如下类型：基本类型、`KClass`、枚举、其他注解
2. 这些类型都是在编译期能确定值的类型

## 5. 注解使用

直接在对应的限定对象上添加注解即可

```kotlin
@State("ready")
class Start
```

## 6. 明确注解的标注对象

```kotlin
@get:JvmName("getStudentName")
var name:String = "Lee"
```

**说明：**

1. 如上定义了一个属性`name`，而注解`@get:JvmName("getStudentName")`作用是将`name`的`getter`方法名字指定为`getStudentName`
2. 变更名字的操作还可以指定为`receiver`、`get`、`file`等

# 三、内置注解

## 1. 元注解

**元注解** 是为其他注解进行说明的注解，当自定义一个新的注解类型时，其中可以使用元注解。元注解位于`kotlin.annotation.*`包下

### 1 `@Target`

`@Target` 用来 **指定一个新注解的使用目标**。
`@Target` 注解有一个 `allowedTargets` 属性，该属性用来设置注解的使用目标，`allowedTargets` 是 `kotlin.annotation.AnnotationTarget` 枚举类型。
`AnnotationTarget`描述 `Kotlin` 代码中可以被注解的元素类型，它有 **15** 个枚举常量。如下：

| 常量             | 使用目标                       |
| ---------------- | ------------------------------ |
| CLASS            | 类、接口、对象声明和注解类声明 |
| ANNOTATION_CLASS | 其他注解类型声明               |
| TYPE_PARAMERTER  | 用于泛型中类型参数声明         |
| PROPERTY         | 属性声明                       |
| FIELD            | 字段声明，包括属性的支持字段   |
| LOCAL_VARIABLE   | 局部变量声明                   |
| VALUE_PARAMETER  | 用于函数或构造函数参数值声明   |
| CONSTRUCTOR      | 用于构造函数声明               |
| FUNCTION         | 用于函数声明，不包括构造函数   |
| PROPERTY_GETTER  | 只用于属性的 getter 访问器声明   |
| PROPERTY_SETTER  | 只用于属性的 setter 访问器声明   |
| TYPE             | 类型使用                       |
| EXPRESSION       | 任何表达式                     |
| FILE             | 文件                           |
| TYPEALIAS        | 类型别名                       |

### 2. `@Retention`

`@Retention` 用来指定一个新注解的有效范围。
`@Retention` 注解有一个 `value` 属性，该属性用来设置保留期，`value` 是 `kotlin.annotation.AnnotationRetention` 枚举类型。
`AnnotationRetention` 描述注解保留期，它有 **3** 个常量，如下：

|常量|保留期|
|--|--|
SOURCE|只适用于源代码文件中，此范围最小
BINARY|编译器把注解信息记录在编译之后的二进制文件中，对于反射是不可见的，此范围居中
RUNTIME|编译器把注解信息记录在编译之后的二进制文件中，对于反射是可见的，此范围最大，这是默认保留期

## 2. 标准库通用注解

 **标准库通用注解** 位于`kotlin.*`

### 1. `@Metadata`

`Java` 反射对于 `Kotlin` 的很多特性都无法访问和识别，而`Kotlin` 的反射可以访问和识别，就是因为注解`@Metadata`

`@Metadata`会由`Kotlin`的编译器添加到由编译器生成的`class`文件中，它可以被编译器和反射读取。

`Kotlin` 反射过程中，注解的内容解析之后会实例化一个叫做 `KotlinClassHeader` 的类。下表为注解中的属性与类中成员对应关系：

官网源码：[https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-metadata/](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-metadata/)

|Metadata|KotlinClassHeader|说明|
|--|--|--|
|k|kind|注解标注目标类型，例如类、文件等等
mv|metadataVersion|该元数据的版本
bv|bytecodeVersion|字节码版本
d1|data|自定义元数据
d2|strings|自定义元数据补充字段
xs|extraString|附加字段
xi|extraInt|1.1 加入的附加标记，标记类文件的来源类型

**说明：**

**d1：** 存储了自定义格式的元数据，官方声称针对不同的类型格式不定，甚至可以为空，研究发现目前采用 **Protobuf** 进行序列化存储。**这些数据会被 `Kotlin` 反射读取，是反射的一个非常重要的数据来源**。其中包含不限于 **类型、函数、属性等的可见性、类型是否可空、函数是否为 suspend**等等信息。

**d2：** 存储明文字符串字面量，主要存储 `Jvm` 签名等信息。之所以这样设计，主要是为了将这些字符串在运行时直接加载到虚拟机内存的常量池中予以复用，减少内存开销。

### 2. `@UnsafeVariance`

该注解用于泛型破除型变限制

**源码如下：**

```kotlin
@Target(TYPE)
@Retention(SOURCE)
@MustBeDocumented
public annotation class UnsafeVariance
```

### 3. `@Suppress`

在 `Java` 中，使用注解 `@SuppressWarnings("xxx")` 消除一些编译时的警告
在 `Kotlin` 中，使用内置的 `@Suppress("xxx")` 消除一些编译时的警告

使用时，将警告类型作为参数传入注解入参

**示例如下：**

```kotlin
val map: Map<Any,Any> = emptyMap()

@Suppress("UNCHECKED_CAST")
map as Map<String, Any>
```

**说明：**

1. 如果不添加注解，则会有警告提示：`Unchecked cast: Map<Any, Any> to Map<String, Any>`，加上注解后警告消失


**注解的源码如下：**

```kotlin
@Target(CLASS, ANNOTATION_CLASS, PROPERTY, FIELD, LOCAL_VARIABLE, VALUE_PARAMETER,
        CONSTRUCTOR, FUNCTION, PROPERTY_GETTER, PROPERTY_SETTER, TYPE, EXPRESSION, FILE, TYPEALIAS)
@Retention(SOURCE)
public annotation class Suppress(vararg val names: String)
```

## 3. 与`Java`虚拟机交互的注解

用于与`Java`虚拟机交互的注解，位于`kotlin.jvm.*`

官网源码：[https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/)
参考文章：[正确使用Kotlin注解，兼容Java代码](https://www.jianshu.com/p/eb091dd229f6)

### 1. `@JvmField`

使`Kotlin`编译器不再对该字段生成`getter/setter`，且其作用域为`public`

```kotlin
class Person(var name:String, var age:Int)
```

反编译上述代码`class`文件后代码如下：

```java
public final class Person {
   @NotNull
   private String name;
   private int age;

   @NotNull
   public final String getName() {
      return this.name;
   }

   public final void setName(@NotNull String var1) {
      Intrinsics.checkNotNullParameter(var1, "<set-?>");
      this.name = var1;
   }

   public final int getAge() {
      return this.age;
   }

   public final void setAge(int var1) {
      this.age = var1;
   }

   public Person(@NotNull String name, int age) {
      this.name = name;
      this.age = age;
   }
}
```

为属性添加注解`@JvmField`如下：

```kotlin
class Person(@JvmField var name:String, var age:Int)
```

反编译上述代码`class`文件后代码如下：

```java
public final class Person {
   @JvmField
   @NotNull
   public String name;
   private int age;

   public final int getAge() {
      return this.age;
   }

   public final void setAge(int var1) {
      this.age = var1;
   }

   public Person(@NotNull String name, int age) {
      this.name = name;
      this.age = age;
   }
}
```

**说明：**

1. 可以看到添加注解`JvmField`后，`name`属性没有了`getter/setter`，但是作用域从`private`变为了`public`

### 2. `@JvmName`

告诉使`Kotlin`编译器生成的`Java`类或者方法的名称
**注意：该注解只能用于函数、属性的 `getter/setter`、文件**

```kotlin
package cn.ltt.projectcollection.kotlin.annotationlab

var kotlinField: Int = 2
    set(value) {
        field = value
    }
    get() {
        return field
    }

fun kotlinFunction() {
}

fun main() {
    kotlinFunction()
}
```

反编译上述代码`class`文件后代码如下：

```java
// KotlinAnnotationLabKt.java
package cn.ltt.projectcollection.kotlin.annotationlab;

public final class KotlinAnnotationLabKt {
   private static int kotlinField = 2;

   public static final int getKotlinField() {
      return kotlinField;
   }

   public static final void setKotlinField(int value) {
      kotlinField = value;
   }

   public static final void kotlinFunction() {
   }

   public static final void main() {
      kotlinFunction();
   }

   // $FF: synthetic method
   public static void main(String[] var0) {
      main();
   }
}
```

为文件、函数、属性的`getter/setter`添加注解`@JvmName`如下：

```kotlin
//修改生成文件名
@file:JvmName("JavaClassLab")

package cn.ltt.projectcollection.kotlin.annotationlab

var kotlinField: Int = 2
    //修改属性的set方法名
    @JvmName("setJavaField")
    set(value) {
        field = value
    }
    //修改属性的get方法名
    @JvmName("getJavaField")
    get() {
        return field
    }

//修改普通的方法名
@JvmName("JavaFunction")
fun kotlinFunction() {
}

fun main() {
    kotlinFunction()
}
```

反编译上述代码`class`文件后代码如下：

```java
// JavaClassLab.java
package cn.ltt.projectcollection.kotlin.annotationlab;

import kotlin.jvm.JvmName;

@JvmName(
   name = "JavaClassLab"
)
public final class JavaClassLab {
   private static int kotlinField = 2;

   @JvmName(
      name = "getJavaField"
   )
   public static final int getJavaField() {
      return kotlinField;
   }

   @JvmName(
      name = "setJavaField"
   )
   public static final void setJavaField(int value) {
      kotlinField = value;
   }

   @JvmName(
      name = "JavaFunction"
   )
   public static final void JavaFunction() {
   }

   public static final void main() {
      JavaFunction();
   }

}
```

**说明：**

1. 注解`@JvmName`只能用于函数、属性的 `getter/setter`、文件
2. `Kotlin`可以正常使用添加了注解`@JvmName`的对象，但是编译器会自动替换成`@JvmName`声明的名字
3. `Java`直接使用`@JvmName`修改后的名字，由此可以用来应对各种类名修改以后的兼容性问题

### 3. `@JvmMultifileClass`

`Kotlin`编译器会将多个文件生成一个类，该文件具有在此文件中 **声明的顶级函数和属性** 作为其中的一部分，`JvmName`注解提供了相应的多文件的名称

```kotlin
//MultiClassA.kt

@file:JvmName("Utils")
@file:JvmMultifileClass
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest

fun getDeviceId() : String = "deviceId"

lateinit var id: String

const val DEVICE_ID_DEFAULT = "123"
```

反编译上述代码`class`文件后代码如下：

```java
// Utils__MultiClassAKt.java
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest;

final class Utils__MultiClassAKt {
   public static String id;

   @NotNull
   public static final String getDeviceId() {
      return "deviceId";
   }

   @NotNull
   public static final String getId() {
      String var10000 = id;
      return var10000;
   }

   public static final void setId(@NotNull String var0) {
      id = var0;
   }
}
// Utils.java
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest;
public final class Utils {
   @NotNull
   public static final String DEVICE_ID_DEFAULT = "123";

   @NotNull
   public static final String getId() {
      return Utils__MultiClassAKt.getId();
   }

   public static final void setId(@NotNull String var0) {
      Utils__MultiClassAKt.setId(var0);
   }

   @NotNull
   public static final String getDeviceId() {
      return Utils__MultiClassAKt.getDeviceId();
   }
}

```

```kotlin
//MultiClassB.kt
@file:JvmName("Utils")
@file:JvmMultifileClass
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest

fun getDeviceWidth() : Int = width

var width: Int = 333

const val DEVICE_WIDTH_DEFAULT = 233
```

反编译上述代码`class`文件后代码如下：

```java
// Utils__MultiClassBKt.java
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest;

final class Utils__MultiClassBKt {
   private static int width = 333;

   public static final int getDeviceWidth() {
      return width;
   }

   public static final int getWidth() {
      return width;
   }

   public static final void setWidth(int var0) {
      width = var0;
   }
}
// Utils.java
package cn.ltt.projectcollection.kotlin.annotationlab.multiclasstest;

public final class Utils {
   public static final int DEVICE_WIDTH_DEFAULT = 233;

   public static final int getWidth() {
      return Utils__MultiClassBKt.getWidth();
   }

   public static final void setWidth(int var0) {
      Utils__MultiClassBKt.setWidth(var0);
   }

   public static final int getDeviceWidth() {
      return Utils__MultiClassBKt.getDeviceWidth();
   }
}

```

`Java`调用上述两个文件的属性、函数如下所示：

```java
public class MultiClassTest {
    public static void main(String[] args) {
        Utils.getDeviceId();
        Utils.getDeviceWidth();
        Utils.setId("111");
        Utils.getId();
        Utils.getWidth();
        System.out.println(Utils.DEVICE_ID_DEFAULT);
    }
}
```

**说明：**

1. 通过反编译的代码可以看出，添加了`@JvmMultifileClass`注解，在生成自己的类文件同时还会生成一个由`@JvmName`提供名字的公共类，然后，两个添加了`@JvmMultifileClass`注解的`Kotlin`文件中的顶级函数、属性的都可以在公共类中调用。
2. 即将两个添加了`@JvmMultifileClass`注解的文件`MultiClassA.kt`、`MultiClassB.kt` 合并到了公共类`Utils`中，注解可以消除我们去手动创建一个公共类，向公共类中添加方法更加灵活和方便


### 4. `@JvmOverloads`

Kotlin编译器为添加该注解的函数生成替换默认参数值的重载

```kotlin

fun defaultParamFun(x:Int = 5, y:String, z: Long = 0L) {
    println("x:$x, y:$y, z: $z")
}
```

反编译上述代码`class`文件后代码如下：

```java
public static final void defaultParamFun(int x, @NotNull String y, long z) {
      Intrinsics.checkNotNullParameter(y, "y");
      String var4 = "x:" + x + ", y:" + y + ", z: " + z;
      boolean var5 = false;
      System.out.println(var4);
   }

// $FF: synthetic method
public static void defaultParamFun$default(int var0, String var1, long var2, int var4, Object var5) {
   if ((var4 & 1) != 0) {
      var0 = 5;
   }

   if ((var4 & 4) != 0) {
      var2 = 0L;
   }

   defaultParamFun(var0, var1, var2);
}
```

为函数添加注解`@JvmOveerloads`如下：

```kotlin
@JvmOverloads
fun defaultParamFun(x:Int = 5, y:String, z: Long = 0L) {
    println("x:$x, y:$y, z: $z")
}
```

反编译上述代码`class`文件后代码如下：

```java
@JvmOverloads
public static final void defaultParamFun(int x, @NotNull String y, long z) {
   Intrinsics.checkNotNullParameter(y, "y");
   String var4 = "x:" + x + ", y:" + y + ", z: " + z;
   boolean var5 = false;
   System.out.println(var4);
}

// $FF: synthetic method
public static void defaultParamFun$default(int var0, String var1, long var2, int var4, Object var5) {
   if ((var4 & 1) != 0) {
      var0 = 5;
   }

   if ((var4 & 4) != 0) {
      var2 = 0L;
   }

   defaultParamFun(var0, var1, var2);
}

@JvmOverloads
public static final void defaultParamFun(int x, @NotNull String y) {
   defaultParamFun$default(x, y, 0L, 4, (Object)null);
}

@JvmOverloads
public static final void defaultParamFun(@NotNull String y) {
   defaultParamFun$default(0, y, 0L, 5, (Object)null);
}
```

**说明：**

1. 可以看到添加了注解 `@JvmOverloads`的函数会重载多个名字相同，参数不同的函数
2. 该注解应该使用在带默认参数的函数上，由此可以让`Java`享受到`Koltin`的 **默认参数**的特性
3. 重载的规则是顺序重载，**只有有默认值的参数会参与重载**

### 5. `@JvmStatic`

对 **函数**使用该注解，`kotlin`编译器将生成另一个静态方法
对 **属性**使用该注解，`kotlin`编译器将生成其他的`setter`和`getter`方法
**作用：** 消除`Java`调用`Kotlin`的`companion object`对象时不能直接调用其静态方法和属性的问题
**注意：此注解只能在`companion object`中使用**

```kotlin
class ClassA {
    companion object {
        var nameDefault: String = "Lee"

        fun getName(): String = "name:$nameDefault"
    }
}
```

反编译上述代码`class`文件后代码如下：

```java
public final class ClassA {
   @NotNull
   private static String nameDefault = "Lee";
   @NotNull
   public static final ClassA.Companion Companion = new ClassA.Companion((DefaultConstructorMarker)null);

   public static final class Companion {
      @NotNull
      public final String getNameDefault() {
         return ClassA.nameDefault;
      }

      public final void setNameDefault(@NotNull String var1) {
         Intrinsics.checkNotNullParameter(var1, "<set-?>");
         ClassA.nameDefault = var1;
      }

      @NotNull
      public final String getName() {
         return "name:" + ((ClassA.Companion)this).getNameDefault();
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

```java
//未添加注解java调用companion object中的属性函数
public static void main(String[] args) {
   ClassA.Companion.getName();
   ClassA.Companion.getNameDefault();
   ClassA.Companion.setNameDefault("Lee2");
}
```

**说明：**

1. 由反编译的代码可以看出，在`companion object`中定义的函数和属性生成的`Java`方法也只是在`Companion`类中。即如果需要调用`companion object`中定义的函数和属性必须借助`Companion`

为属性和函数添加注解`@JvmStatic`如下：

```kotlin
class ClassA {
    companion object {
        @JvmStatic
        var nameDefault: String = "Lee"
        @JvmStatic
        fun getName(): String = "name:$nameDefault"
    }
}
```

反编译上述代码`class`文件后代码如下：

```java
public final class ClassA {
   @NotNull
   private static String nameDefault = "Lee";
   @NotNull
   public static final ClassA.Companion Companion = new ClassA.Companion((DefaultConstructorMarker)null);

   @NotNull
   public static final String getNameDefault() {
      ClassA.Companion var10000 = Companion;
      return nameDefault;
   }

   public static final void setNameDefault(@NotNull String var0) {
      ClassA.Companion var10000 = Companion;
      nameDefault = var0;
   }

   @JvmStatic
   @NotNull
   public static final String getName() {
      return Companion.getName();
   }

   public static final class Companion {
      /** @deprecated */
      // $FF: synthetic method
      @JvmStatic
      public static void getNameDefault$annotations() {
      }

      @NotNull
      public final String getNameDefault() {
         return ClassA.nameDefault;
      }

      public final void setNameDefault(@NotNull String var1) {
         Intrinsics.checkNotNullParameter(var1, "<set-?>");
         ClassA.nameDefault = var1;
      }

      @JvmStatic
      @NotNull
      public final String getName() {
         return "name:" + ((ClassA.Companion)this).getNameDefault();
      }

      private Companion() {
      }

      // $FF: synthetic method
      public Companion(DefaultConstructorMarker $constructor_marker) {
         this();
      }
   }
}
```

```java

public static void main(String[] args) {
   //添加注解后java调用companion object中的属性函数
   ClassA.getName();
   ClassA.getNameDefault();
   ClassA.setNameDefault("Lee2");
}
```

**说明：**

1. 可以看到添加了注解 `@JvmStatic`后的属性和函数都会在伴生对象所在的类中生成各自对应的静态方法，虽然`Companio`n这个静态内部类还在，但是`Java`现在就不需要再借助`Companion`类而可以直接调用对应的静态方法和属性


### 6. `@Synchronized`

`Kotlin`语言不支持`synchronized`关键字，使用该注解生成同步方法

```kotlin
@Synchronized
fun start() {

}
```

反编译上述代码`class`文件后代码如下：

```java
public final synchronized void start() {
}
```

**说明：**

1. 注解`@Synchronized`只能添加在 **函数、属性的`getter/setter`方法**
2. 设置了该注解的函数编译器生成`java`方法时会自动添加一个`synchronized`关键字

### 7. `@Volatile`

该注解对应`Java`端的关键字`volatile`，功能与其一致

```kotlin
@Volatile
lateinit var name: String
```

反编译上述代码`class`文件后代码如下：

```java
public volatile String name;
```

**说明：**

1. 注解`@Volatile`只用于属性
2. 设置了该注解的属性编译器生成`java`变量时会自动添加一个`volatile`关键字

### 8. `@Throws`

**`Checked exception`:** 继承自 `Exception` 类。代码需要处理 `API` 抛出的 `checked exception`，要么用 `catch` 语句，要么直接用 `throws` 语句抛出去。可以理解为 **编译时异常**，不处理编译不能通过。

由于`Kotlin`语言不支持`CE`*(Checked Exception)*，为了兼容这种写法，`Kotlin`语言增加了`@Throws`注解，该注解的接收一个可变参数，参数类型是多个异常的`KClass`实例。`Kotlin`编译器通过读取注解参数，在生成的字节码中自动添加`CE`声明

```kotlin
@Throws(IllegalAccessException::class)
fun hello(){

}
```

反编译上述代码`class`文件后代码如下：

```java
public static final void hello() throws IllegalAccessException {
}
```

**说明：**

1. 注解只能用于 **函数、属性的`getter/setter`、构造器**上

# 四、 应用 -- 仿 `Retrofit`反射读取注解请求网络

## 1. 创建一些需要的注解

```kotlin

data class User(
    var login: String,
    var location: String,
    var avatar_url: String,
    var bio: String)

@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.CLASS)
annotation class Api(val url: String)

@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.CLASS)
annotation class Path(val url: String = "")

@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.FUNCTION)
annotation class Get(val url: String = "")

@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.VALUE_PARAMETER)
annotation class PathVariable(val name: String = "")

@Retention(AnnotationRetention.RUNTIME)
@Target(AnnotationTarget.VALUE_PARAMETER)
annotation class Query(val name: String = "")

@Api("https://api.github.com")
interface GitHubApi {

    @Api("users")
    interface Users {

        @Get("{name}")
        fun get(name: String): User

        @Get("{name}/followers")
        fun followers(name: String): List<User>

    }

    @Api("repos")
    interface Repos {

        @Get("{owner}/{repo}/forks")
        fun forks(owner: String, repo: String)

    }
}
```

**说明：**

1. 定义一个数据类`User`，用于接收从接口`https://api.github.com/users/puppet16`返回的数据，分别有登录名 *(login)*、所在地 *(location)* 、头像 *(avatar_url)*、个性签名 *(bio)*
2. `https://api.github.com/users/用户名`接口可以获取`Github`网站上某用户的一些公开信息
3. 定义了两个作用于接口上的注解`Api`、`Path`:
   1. `Api`注解：有一个参数用于保存网络地址的`host`
   2. `Path`注解：有一个参数用于保存网络地址的`path`路径，参数默认为空串
4. 定义了一个作用于函数上的注解`Get`,有一个参数用于保存网络地址的
5. 定义了两个作用于函数或构造函数参数值声明的注解`PathVariable`、`Query`，这两个注解中都有一个默认值为空串的参数`name`
6. 定义一个接口`GitHubApi`，在该接口定义上添加注解`@Api`，参数传入 `https://api.github.com`，在该接口内又定义了接口接口`Users`，添加注解`@Api`，传入的是该类的`host`值：`users`，该类又定义了两个函数：
   1. 函数`get()`，用于获取用户的信息
   2. 函数`followers()`，用于获取助关注用户的用户列表
