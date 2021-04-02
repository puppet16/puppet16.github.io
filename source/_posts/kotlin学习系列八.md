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
@Target(AnnotationTarget.FUNCTION)
annotation class Get(val url: String = "")

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
5. 定义一个接口`GitHubApi`，在该接口定义上添加注解`@Api`，参数传入 `https://api.github.com`，在该接口内又定义了接口接口`Users`，添加注解`@Api`，传入的是该类的`host`值：`users`，该类又定义了两个函数：
   1. 函数`get()`，用于获取用户的信息
   2. 函数`followers()`，用于获取助关注用户的用户列表

## 2. 创建解析单例`RetroApi`

```kotlin
object RetroApi {
    const val PATH_PATTERN = """(\{(\w+)\})"""

    val okHttp = OkHttpClient()
    val gson = Gson()

    val enclosing = {
        cls: Class<*> ->
        var currentCls: Class<*>? = cls
        sequence {
            while(currentCls != null){
                currentCls = currentCls?.also { yield(it) }?.enclosingClass
            }
        }
    }

    inline fun <reified T> create(): T {
        val functionMap = T::class.functions.map{ it.name to it }.toMap()
        val interfaces = enclosing(T::class.java).takeWhile { it.isInterface }.toList()

        val apiPath = interfaces.foldRight(StringBuilder()) {
            clazz, acc ->
            acc.append(clazz.getAnnotation(Api::class.java)
                ?.url?.takeIf { it.isNotEmpty() }
                ?:clazz.name).append("/")
        }.toString()

        return Proxy.newProxyInstance(RetroApi.javaClass.classLoader, arrayOf(T::class.java)) {
            proxy, method, args ->
            functionMap[method.name]?.takeIf { it.isAbstract }?.let {
                function ->
                val parameterMap = function.valueParameters.map {
                    it.name to args[it.index - 1]
                }.toMap()

                //{name}
                val endPoint = function.findAnnotation<Get>()!!.url.takeIf { it.isNotEmpty() }?: function.name
                val compiledEndPoint = Regex(PATH_PATTERN).findAll(endPoint).map {
                    matchResult ->
                    matchResult.groups[1]!!.range to parameterMap[matchResult.groups[2]!!.value]
                }.fold(endPoint) {
                    acc, pair ->
                    acc.replaceRange(pair.first, pair.second.toString())
                }
                val url = apiPath + compiledEndPoint
                println(url)

                okHttp.newCall(Request.Builder().url(url).get().build()).execute().body?.charStream()?.use {
                    gson.fromJson(JsonReader(it), method.genericReturnType)
                }

            }
        } as T
    }
}
```

**说明：**

1. 定义一个正则表达式`PATH_PATTERN`，用于匹配带大括号的大括号内做任意文本的字符串，其中分为了两组，一组是带大括号的内容文本，第二组是大括号内的文本。
2. 定义一个`OkHttpClient`的实例，名为`okHttp`
3. 定义一个`Gson`的实例，名为`gson`
4. 定义了一个名为`enclosing`的`lambda`表达式，它接收一个`Class<*>`类型的参数`cls`，定义一个属性，用于保存当前的`Class<*>`,之后在函数中定义一个懒序列，在序列中做如下操作：
   1. 创建一个循环，循环条件是当前`class`不为空
   2. `enclosingClass()`方法可以获取对应类的直接外部类`class`对象
   3. 序列的`next`方法就可以拿到`yield()`方法传递出的内容
   4. 通过`yield()`方法将当前类`currentCls`放置到队列中
   5. 通过`currentCls?.enclosingClass`获取当前`class`的直接外部类，并将其赋值给当前类`currentCls`
   6. 总结：`enclosing`表达式的功能就是返回了一个序列，该序列中有某类的一层层的所有外部类
5. `create()`方法通过动态代理形式去创建一个泛型类型`T`对象，该函数使用了内联特化。
6. 定义了一个`Map`类型的属性`functionMap`，存储泛型`T`所有函数的名字和函数的键值对
7. 定义了一个`List`类型的属性`interfaces`，存储泛型`T`所有的接口类型的外部类及他自身。通过`enclosing()`函数将泛型 `T`的`java`的`class`传递进去，返回一个包含泛型`T`所有外部类的队列，先从泛型`T`本身类型取逐层往外层的类取，直到外层的类**不是接口**为止，然后将符合的数据转为`list`，所以该`list`中外部类元素的顺序是 **从内往外的**
8. 定义了一个`String`类型的属性`apiPath`，用于存储泛型`T`所包含的网络路径。通过定义的`intetfaces`属性取得所有外部类`class`对象列表，然后通过`foldRight()`方法从后向前遍历元素 *(因为`interfaces`列表中类顺序是从内向外，而地址是从外往里的)*。`foldRight()`函数有两个参数：一个是初始值，这里定义的初始值是一个`StringBuilder`对象，一个是以列表中元素和初始值为参数的函数，在该函数中通过 `clazz.getAnnotation(Api::class.java)`反射得出该类的`Api`注解对象，之后获取该注解中的`url`属性的值，若 `url`是空的，则取该类的类名，之后将取到的取到的地址段通过`append`方法拼接成一个字符串
9. 通过`java`反射包中的`Proxy.newProxyInstance()`创建一个代理类，其中
   1. 第一个参数传递的是`RetroApi`类的`classloader`，
   2. 第二个参数传递的是数组类型的接口列表，即传递以泛型`T`为元素的数组
   3. 第三个参数是`InvocationHandler`类型，也就是一个函数，该函数是一个接口且只有一个方法，所以可以使用`lambda`表达式，将其从参数列表中往外提
10. `InvocationHandler`接口中的`invoke`方法有三个参数： 第一个参数是 `Object`类型的`proxy`，第二个参数是 `Method`类型的`method`，第三个参数是 `Object[]`类型的`args`:
    1. 在该方法中通过属性`functionMap`可以得到以`method.name`为`Key`值的对应的泛型`T`中的函数，通过`takeIf`判断必须该函数必须是抽象的，这样的才需要实现。
    2. 之后通过`let`函数，将之前的取得的抽象的函数作为名为`function`的参数创建一个匿名函数。
    3. 在该匿名函数中，先通过调用函数的`valueParameters()`方法，获取该函数的形参列表，之后生成形参名称为`key`，对应形参值为`value`的键值对组成的`map`,其名为`parameterMap`
    4. 接着通过`findAnnotation`方法获取该函数某注解实例，在此获取`Get`注解的实例，取出该注解中`url`属性的值，若该值为空，则取该函数名，将结果保存到属性`endPoint`中
    5. 之后通过函数`Regex`匹配正则表达式，再调用`findAll`获取所有满足正则的内容，之后调用`map`函数，对每个满足正则文本`matchResult`进行操作，`matchResult.groups`有三个元素，第一个元素是匹配的整个字符串；第二个元素是匹配的第一组结果，第三个元素是匹配的第二组结果。此时第一个元素就是属性`endPoint`存储的字符串，第二个元素就是带大括号的文本，第三个元素就是大括号内的文本，再通过属性`parameterMap`取得以第三个元素为`key`值的形参值，将`matchResult`中的第二个元素的文本的下标范围`range`作为`key`，再将形参值作为`value`，以此构造成键值对组成`map`
    6. 通过`fold()`函数将上一步骤得到的`map`每个元素进行操作，初始值设置为`endPoint`,该步骤功能为将`endPoint`中对应大括号包裹的文本替换为大括号内文本作为形参名对应的形参值
    7. 如此得到了将大括号及大括号中广西替换为形参值的字符串，其名为`compiledEndPoint`
    8. 定义属性`url`,用于存储完整的网络地址。也就是将`apiPath`和`compiledEndPoint`拼接到一起
    9. 调用 `okHttp`的`newCall`方法，进行网络请求。先通过`Request.Builder()`创建一个带`url`的`request`，之后调用`execute()`同步方法发起网络请求，之后获取返回体的`charStream`，通过`use`函数对流进行操作，结束时会自动关闭：通过`gson`的`fromJson()`方法将返回的内容反序列化，`fromJson()`方法第一个参数为`JsonReader`对象，构建对象时将`charStream`作为参数传入构造方法中；第二个参数为泛型的实参，可通过`method.genericReturnType`获取

## 3. 测试反射读取注解请求网络功能

```kotlin
fun main() {
    val usersApi = RetroApi.create<GitHubApi.Users>()
    println(usersApi.get("puppet16"))
    println(usersApi.followers("puppet16").map { it.login })
}
```

**打印结果如下：**

```java
https://api.github.com/users/puppet16
User(login=puppet16, location=Beijing, avatar_url=https://avatars.githubusercontent.com/u/14159272?v=4, bio=A day is a miniature of eternity. )
https://api.github.com/users/puppet16/followers
[xLily, lianxiaolei, Fakekid]
```

**说明：**

1. 通过单例`RetroApi`的`create`方法，创建了一个`GitHubApi.Users`的实例`userApi`
2. 先获取了用户名为`puppet16`的`GitHub`账号信息
3. 又获取了用户名为`puppet16`的`GitHub`账号的关注列表

# 五、应用 -- 注解加持反射版`Model`映射

在{% post_link kotlin学习系列七 kotlin学习系列七：反射 %}中的 **应用 -- Model映射**章节中，讲到可以通过`map`映射为任意类型数据类，但是当前实现有缺陷： `map`中的 `key`值必须与数据类的属性名字一致

```kotlin

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

**说明：**

1. `map`中的`key`值与数据类中的构造方法中的属性名称对应，若`map`中没有构造方法中参数的值，则判断这个参数是否可以为`null`，若可以则返回空，否则直接抛异常
2. 判断`map`中有没有构造方法中参数的值的依据是：将构造方法中的参数名当作了`key`值去`map`中取`value`，如此就必须保证`map`中存储的`key`必须与数据类中的构造方法中的属性名称相同

## 1. 创建支持单个属性别名的注解`FildName`

```kotlin

@Target(AnnotationTarget.VALUE_PARAMETER)
annotation class FieldName(val name: String)

inline fun <reified To : Any> Map<String, Any?>.mapAs(): To {
    return To::class.primaryConstructor!!.let {
        it.parameters.map { parameter ->
            parameter to (this[parameter.name]
                ?: (parameter.annotations.filterIsInstance<FieldName>().firstOrNull()?.name?.let(this::get))
                ?: if (parameter.type.isMarkedNullable) null
                else throw IllegalArgumentException("${parameter.name} is required but missing."))
        }.toMap()
            .let(it::callBy)
    }
}
data class Human(val name: String,
                 @FieldName("avatar_url")
                 val avatarUrl: String,
                 @FieldName("detail_url")
                 val detailUrl: String)

data class Person(
        var id: Int,
        var name: String,
        var avatar_url: String,
        var smallUrl: String,
        var detail_url: String
)

fun main() {
    val personMap = mapOf(
            "id" to 0,
            "name" to "Lee",
            "avatar_url" to "https://api.github.com/users/Lee",
            "detail_url" to "https://api.github.com/users/Lee"
    )

    val humanFromMap: Human = personMap.mapAs()
    println(humanFromMap)
}
```

**打印结果：**

```java
Human(name=Lee, avatarUrl=https://api.github.com/users/Lee, detailUrl=https://api.github.com/users/Lee)
```

**说明：**

1. 定义了名为`FieldName`的注解类，包含一个名为`name`的属性，使用于函数中的参数
2. 改进之前的代码思路：将构造方法中的参数名当作了`key`值去`map`中取`value`时，若返回了`null`，则去判断这个构造方法的参数是否添加了注解，若有注解`FieldName`，则取出该注解实例中的`name`值，否则返回`null`，之后再走之前逻辑
3. 通过`parameter.annotations`获取标注在该参数上的所有注解，再通过`filterIsInstance<FieldName>()`筛选中注解`FieldName`的实例组成的列表，再通过`firstOrNull()`方法，取第一个实例，再取其`name`属性值，再将`name`中的属性值作为`key`值，去`map`中取`value`
4. 该注解只能给单个的属性修改`name`，若有多个不匹配的属性则要添加多个比较繁琐

## 2. 创建支持数据类的注解`MappingStrategy`

```kotlin

@Target(AnnotationTarget.CLASS)
annotation class MappingStrategy(val klass: KClass<out NameStrategy>)

interface NameStrategy {
    fun mapTo(name: String): String
}

object UnderScoreToCamel : NameStrategy {
    override fun mapTo(name: String): String {
        return name.toCharArray().fold(StringBuilder()) { acc, c ->
            when (acc.lastOrNull()) {
                '_' -> acc[acc.lastIndex] = c.toUpperCase()
                else -> acc.append(c)
            }
            acc
        }.toString()
    }
}

object CamelToUnderScore : NameStrategy {
    override fun mapTo(name: String): String {
        return name.toCharArray().fold(StringBuilder()) { acc, c ->
            when {
                c.isUpperCase() -> acc.append('_').append(c.toLowerCase())
                else -> acc.append(c)
            }
            acc
        }.toString()
    }
}


inline fun <reified To : Any> Map<String, Any?>.mapAs(): To {
    return To::class.primaryConstructor!!.let {
        it.parameters.map { parameter ->
            parameter to (this[parameter.name]
                    ?: (parameter.annotations.filterIsInstance<FieldName>().firstOrNull()?.name?.let(this::get))
                    ?: To::class.findAnnotation<MappingStrategy>()?.klass?.objectInstance?.mapTo(parameter.name!!)?.let(this::get)
                    ?: if (parameter.type.isMarkedNullable) null
                    else throw IllegalArgumentException("${parameter.name} is required but missing."))
        }.toMap()
                .let(it::callBy)
    }
}


@MappingStrategy(CamelToUnderScore::class)
data class Human(val name: String,
                 val avatarUrl: String,
                 val detailUrl: String)

data class Person(
        var id: Int,
        var name: String,
        var avatar_url: String,
        var smallUrl: String,
        var detail_url: String
)

fun main() {
    val personMap = mapOf(
            "id" to 0,
            "name" to "Lee",
            "avatar_url" to "https://api.github.com/users/Lee",
            "detail_url" to "https://api.github.com/users/Lee"
    )

    val humanFromMap: Human = personMap.mapAs()
    println(humanFromMap)
}
```

**打印结果：**

```java
Human(name=Lee, avatarUrl=https://api.github.com/users/Lee, detailUrl=https://api.github.com/users/Lee)
```

**说明：**

1. 定义名为`MappingStrategy`的注解，接口类型为`KClass`的策略类，这个策略类必须实现接口`NameStrategy`，该接口中有一个函数`mapTo()`，功能是将入参`name`转为另一个`name`，使用于类上
2. 该注解用于解决多个参数名称不匹配的问题，避免多次使用注解`FieldName`
3. 创建一个实现接口`NameStrategy`的单例类`UnderScoreToCamel`，用于 **将下划线风格的属性名称转为驼峰风格**的属性名称。将传入的`name`转为字符数组，再通过`fold()`函数遍历每个字符，`acc`是`StringBuilder`类型，每次循环都将当前字符`c`添加到该属性中，判断如果`acc`最后一位是下划线，就将下划线的位置替换为大写的当前字符，否则就继续给`acc`添加当前字符。之后返回`StringBuilder`类型的`acc`，最后再调用`toString()`方法转为字符串。
4. 创建一个实现接口`NameStrategy`的单例类`CamelToUnderScore`，用于 **将驼峰风格的属性名称转为下划线风格**的属性名称。实现逻辑与`UnderScoreToCamel`类似，只是判断条件变为了：如果当前字符`c`是大写的，就在其之前加个下划线，再添加小写的该字符；否则就直接给`acc`添加当前字符。之后返回`StringBuilder`类型的`acc`，最后再调用`toString()`方法转为字符串。
5. 因为策略不需要每次做映射的时候都去创建，所以两个实现接口`NameStrategy`的策略类都定义为单例类型
6. 之后再对`mapAs()`方法进行改造，在判断了`FieldName`注解之后，再判断注解`MappingStrategy`：
   1. 先获取要转换成的类的`To`上修饰的注解`MappingStrategy`
   2. 再获取该注解类中的策略类的`KClass`
   3. 因为策略类是单例，调用`objectInstance`，创建出该单例的实例，若存在则不会再次创建
   4. 再调用该单例中的`mapTo`方法，将目标类`To`中的参数名称按策略中的规则转换为其它风格的名称
   5. 最后再使用该名称去`map`中取出对应的`value`值
7. 给`Human`数据类添加注解`MappingStrategy`，可以看到该数据类中属性名字是驼峰类型的，而`Person`中对应的属性名字是下划线类型的，要将`person`转为`human`，则策略类使用`CamelToUnderScore`，否则无法在`map`中找到应对的属性名

# 六、 应用 -- 注解处理器版 `Model` 映射

> 上一章节中讲述通过注解处理不同命名风格的属性的映射，但是每次都要调用自己写的`mapAs()`。
> 本章主要讲如何通过注解处理器自动生成`modul`自己的`mapAs()`方法

## 1. 简述`java`编译过程

**编译流程：**

1. 词法分析：词法分析的输入是 **源程序** *(即`java`文件)*，输出是识别出的 **记号流**。目的是识别单词。单词至少分以下几类：关键字 *(保留字)*、标识符、字面量、特殊符号
2. 语法分析：输入是词法分析器返回的 **记号流**，输出是 **语法树**。目的是得到语言结构并以树的形式表示。对于声明性语句，进行符号表的查填，对于可执行语句，检查结构合理的表达式运算是否有意义。
3. 语义分析：根据语义规则对语法树中的语法单元 **进行静态语义检查**，如类型检查和转换等，目的在于保证语法正确的结构在语义分析上也是合法的。
4. 中间代码生成(可选)：生成一种既接近目标语言，又与具体机器无关的表示，便于代码优化与代码生成。
   *(到目前为止，编译器与解释器可以一致)*
5. 中间代码优化(可选)：局部优化、循环优化、全局优化等；优化实际上是一个等价变换，变换前后的指令序列完成同样的功能，但在占用的空间上和程序执行的时间上都更省、更有效
6. 目标代码生成：不同形式的目标代码—汇编语言形式、可重定位二进制代码形式、内存形式(Load-and-Go)
7. 符号表管理：合理组织符号，便于各阶段查找\填写等。
8. 出错处理
9. 参考文章：[.java文件编译过程和执行过程分析](https://www.jianshu.com/p/af78a314c6fc)

![attar](/kotlin学习系列八/java_complier_process.png)

**说明：**

1. 抽象语法树 *(Abstract Syntax Tree, AST)* 是源代码语法结构的一种抽象表示。它以树状的形式表现编程语言的结构。
2. 将源文件进行解析，构造抽象语法树，符号表填充，最输出扩展名为`.class`的字节码文件
3. 在编译过程中会调用注解处理器`APT`*(Annotation Process Tool)* 扫描和处理注解，按照一定的规则，生成相应的`java`文件，然后再进行编译，注解处理器一般生成新的源文件，而不是更改之前源文件的语法树

**代码生成三方工具**：

* 生成`Java`代码： **JavaPoet**
* 生成`Kotlin`代码： **KotlinPoet**

## 2. 实现注解处理器版 `Model` 映射
















# 七、 参考文章

1. [https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-metadata/](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-metadata/)
2. [https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.jvm/)
3. [正确使用Kotlin注解，兼容Java代码](https://www.jianshu.com/p/eb091dd229f6)
4. [.java文件编译过程和执行过程分析](https://www.jianshu.com/p/af78a314c6fc)
5. [Android APT（Java注解应用）](https://www.jianshu.com/p/92e4a6159a1a)