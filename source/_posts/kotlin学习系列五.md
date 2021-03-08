---
title: Kotlin 学习系列五：类型进阶
date: 2021-02-02 10:24:05
tags: [Kotlin, 类型]
summary: Kotlin 类型进阶
---

<!-- toc -->

# 一、前言

1. 本文主要讲述**Kotlin 类型进阶**
2. **Kotlin官网：[https://kotlinlang.org/](https://kotlinlang.org/)**
3. **Kotlin中文官网：[https://www.kotlincn.net/](https://www.kotlincn.net/)**
4. Kotlin 学习系列文章：
    * {% post_link kotlin学习系列一 kotlin学习系列一：内置类型 %}
    * {% post_link kotlin学习系列二 kotlin学习系列二：类与接口初解 %}
    * {% post_link kotlin学习系列三 kotlin学习系列三：表达式 %}
    * {% post_link kotlin学习系列四 kotlin学习系列四：函数进阶 %}

# 二、 类的构造器

## 1. 基本写法

将构造器写到类的定义上

```kotlin
class Person constructor(var age:Int, name:String) {

}
```

**说明：**
1. 如上定义一个类`Person`，其中关键字`constructor`可省略
2. 同时定义了类内的属性`age`, 类内全局可见
3. 同时定义了一个形参`name`，构造器内可见 (init 块，属性初始化）

**init 块：**

```kotlin
class Person(var age:Int, name:String) {
    var name: String
    init {
        this.name = name
    }
    val firstName = name.split(" ")[0]
    init {
        println("firstName:$firstName")
    }
}
```

**说明：**

1. `init`块类似于**主构造器** 的方法体，`init`块可以有多个
2. 多个`init`块会按**从上到下** 的顺序合并执行，上面示例执行顺序如下：

    ```kotlin
    class Person(var age: Int, name: String) {
        var name: String
        var firstName: String

        init {
            this.name = name
            firstName = name.split(" ")[0]
        }
    }
    ```

`Java`中有类似功能，名为构造块

```java
public static class Person {
    private String name;
    private int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    {
        System.out.println("Person-构造块");
    }

    static {
        System.out.println("Person-静态构造块");
    }
}
```

**说明：**

1. `java`中执行顺序为：静态构造块 -->构造块 -->构造方法
2. 构造块访问不到构造方法内的形参

## 2. 类的继承

`Kotlin`中类的继承需要在子类后而加`:父类构造器`

```kotlin
class Person(var age: Int,name: String) :Animal(){
    var personName: String
    var firstName: String

    init {
        this.personName = name
        firstName = name.split(" ")[0]
    }
}

abstract class Animal(var name: String) {
    constructor():this("unknown")
}

```

## 3. 副构造器

定义了主构造器后，在类内部再定义的构造器都被称为**副构造器**

```kotlin
fun main() {
    val person = Person()
}
class Person(var age: Int,name: String){
    constructor():this(20, "unknown")
}
```

**说明：**

1. 副构造器必须调用主构造器
2. 若子类没有主构造器，则不必在继承父类时调用构造器，可在**副构造器时调用父类的构造器**

    ```kotlin
    class Person :Animal{
        var personName: String
        constructor(personName: String): super("unknown"){
            this.personName = personName
        }
    }

    abstract class Animal(var name: String) {
    }
    ```

3. 建议创建类时采用**主构造器 + 默认参数**的方式，这样构造路径较少，减少复杂度

**@JvmOverloads**注解

**作用：** 主构造器的**默认参数** 在`Java`代码中可以以重载的形式调用，即可省略

```kotlin
//kotlin

class Person @JvmOverloads constructor(var age: Int = 18,name: String){
    constructor():this(20, "unknown")
}
```

```java
//java
Person person = new Person("lee");
```

如上，若`kotlin`中主构造器前不加`@JvmOverloads`注解，则`java`中的调用语句会报错。

## 3. 构造同名的工厂函数

在`Kotlin`中，定义了一个类情况下，还可以定义同名的函数，该同名函数一般用于构建同名类对象

```kotlin
val str = String()
val str1 = String(charArrayOf('1','2'))
```

如上，第一个语句调用的是类`String`的构造器来创建一个`String`对象，第二条语句是调用的函数`String()`来创建一个`String`对象，该函数入参为`CharArray`。还可以自定义同名的函数，如下

```kotlin
fun String(ints: IntArray): String {
    return ints.contentToString()
}
```

# 三、 类与成员的可见性

## 1. 可见性对比

|可见性类型|Java|Kotlin|
|--|--|--|
|public|公开|与 java 相同，默认类型|
|internal|X|模块内可见|
|default|包内可见，默认|X|
|protected|包内及子类可见|类内及子类可见|
|private|类内可见|类或文件内可见|

## 2. 修饰对象

|可见性类型|顶级声明|类|成员|
|--|--|--|--|
|public|可以|可以|可以|
|internal|可以|可以|可以|
|protected|不可|不可|可以|
|private|可以|可以|可以|

**说明：**

1. `internal`虽然所有都可以修饰，但是被修饰的对象是模块内可见
2. `private`虽然所有都可以修饰，但是修饰顶级声明和类时，只是文件内可见；修饰成员的话只是类内可见

**模块**：在一次调用命令`kotlinc`的文件当中，`internal`可见，直观的讲，大致可认为一个`Jar`包、一个`aar`

## 3. `internal`与`default`的对比

1. 一般由 SDK 或公共组件开发者用于隐藏模块内部细节实现
2. `default`可通过外部创建相同包名来访问，访问控制非常弱
3. `default`会导致不同抽象层次的类聚集到相同包之下
4. `internal`可方便处理内外隔离，提升模块代码内聚、减少接口暴露
5. `internal`修饰的`Kotlin`类或成员在`Java`当中可直接访问，此时若想`Java`不能调用，需要借助注解`@JvmName()`为将类或成员添加一个`Java`识别的**不符合 java 命名规范的名称**

    ```kotlin
    //kotlin
    internal class Person(var name: String = "Lee", var age:Int = 18) {
        internal fun printPerson() {
            println("Person{name=$name, age=$age}")
        }
    }
    ```

    ```java
    //java
    Person person = new Person();
    person.printPerson$ProjectCollection_app();
    ```

    如上所示，在`kotlin`中声明了一个`internal`的类`Person`，其中有一个用于打印信息的函数`printPerson()`，正常情况下，在`Java`文件中，创建`Person`对象后，可直接调用该函数，在`java`中该函数名字是*方法名 $ 模块名*
    <br/>

    ```kotlin
    //kotlin
    internal class Person(var name: String = "Lee", var age:Int = 18) {
        @JvmName("123")
        internal fun printPerson() {
            println("Person{name=$name, age=$age}")
        }
    }
    ```

    ```java
    //java
    Person person = new Person();
    person.123();
    ```

    如上所示，为函数`printPerson()`添加了注解`@JvmName`, 让`java`识别该函数名为`123`, 但因`Java`不支持该命名方式，所以不可以调用该函数

## 4. 构造器的可见性

在声明类时若要为声明的主构造器限制作用域，则不可以省略关键字`constructor`
比如说要将该类写成一个单例或写成工厂，则应该将构造器私有化

```kotlin
class Person private constructor(var age:Int, var name:String)
```

## 5. 属性的可见性

可在声明属性时直接加可见性关键字进行修饰

```kotlin
class Person private constructor(private var age:Int, var name:String)
```

如上所示，将属性`age`私有化，则创建对象`person`后，无法访问`age`属性。

还可以为属性的`setter`添加可见性

```kotlin
class Person(var name: String, private var age:Int) {
     var firstName: String = ""
        private set(value) {
            field = value
        }
}
```

1. 不可以设置`getter`的可见性，因为`getter`可见性要与属性保持一致
2. `setter`的可见性不得大于属性的可见性

## 6. 顶级声明的可见性

1. 顶级声明批文件内直接定义的属性、函数、类等
2. 顶级声明不支持`projected`，在`kotlin`中只有子类可见的含义
3. 顶级声明被`private`修饰表示文件内部可见

# 四、 类属性的延迟初始化

## 1. 延迟初始化原因

1. 类属性必须在构造时初始化
2. 某些成员只有在类构造之后才会被初始化

```java
private TextView mTvName;

@Override
protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_test);
    mTvName = findViewById(R.id.tv_name);
}
```

如上，`Android`中创建了一个名为`mTvName`的`TextView`，并在`onCreate()`方法中为其赋值。但是转换为`Kotlin`代码时却会在声明变量时报错，因为没有初始化，此时可将变量先定义为可空，然后赋空，再在`onCreate()`方法中为其赋正确的值。

## 2. 延迟初始化实现

### 1. 为需要延迟初始化的成员赋空值

```kotlin
private var mTvName: TextView? = null

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_test)
    mTvName = findViewById(R.id.tv_name);
}
```

如上，将变量`mTvName`初始化为了`null`，再在`onCreate()`方法中为其赋正确的值，虽然解决了要延后初始化的问题，但每次使用`mTvName`的属性时都要将其做判空处理或进行类型强转。为解决以上问题可使用关ß键字`lateinit`

### 2. 使用关键字`lateinit`

可在需要延迟初始化的成员声明前加上关键字`lateinit`

```kotlin
private lateinit var mTvName: TextView

override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_test)
    mTvName = findViewById(R.id.tv_name);
}
```

**说明：**

1. 使用关键字`lateinit`修饰的属性必须定义为`var`，允许再次赋值
2. 关键字`lateinit`会让编译器忽略变量的初始化，不支持`Int`等基本类型，只能修饰**非基本类型**
3. 在`Kotlin`的 1.2 版本之后，为属性添加了扩展方法`isInitialized`，可使用该方法来判断被`lateinit`修饰的属性是否被初始化，已初始化返回`true`
4. 必须在能够完全确定变量值的生命周期下使用`lateinit`
5. 不要在复杂的逻辑中使用`lateinit`，否则可读性会很差，不能很清晰的知道在哪里初始化

### 3. 使用`lazy`

```kotlin
private val mTvName by lazy {
    findViewById<TextView>(R.id.tv_name)
}
```

**说明：**

1. 如上代码使用的是属性代理，`lazy`本身是属性代理的实现，只有`mTvName`首次被访问时才会执行`lambda`表达式内代码
2. 如上将声明和初始化放到了一起，可读性较强

### 4. 方案对比

|方案名称|优缺点|推荐|
|--|--|--|
|可空类型|增代码复杂度；初始化与声明分离；调用处需要做判空处理|不推荐|
|lateinit|初始化与声明分离；调用处虽无需判空处理，但潜在的初始化问题可能被掩盖|一般|
|lazy|初始化与声明内聚；无需声明可空类型|推荐|

# 五、 代理 Delegate

**接口代理：** 某对象代替某个类实现某个接口
**属性代理：** 某对象代替某属性实现`getter/setter`方法

## 1. 接口代理

```kotlin

interface TextWatcher {

    fun beforeTextChanged()

    fun onTextChanged()

    fun afterTextChanged();
}
```

如上声明一个接口`TextWatcher`，其中三个方法，用于监听文本改变之前，文本改变时，文本改变之后三个状态。

```kotlin
class TextWatcherImpl : TextWatcher {
    override fun beforeTextChanged() {
        println("beforeTextChanged")
    }

    override fun onTextChanged() {
        println("onTextChanged")
    }

    override fun afterTextChanged() {
        println("afterTextChanged")
    }
}
```

如上所示，正常的一个类实现接口时，需要将接口的所有方法全部实现。此时若有一个业务场景：需要在`afterTextChanged()`方法内做一些其它逻辑，其它两个方法还沿用之前的逻辑，此时应创建一个`wrapper`类，包装一下`TextWathcerImpl`

```kotlin
class TextWatcherImplWrapper(val watcher: TextWatcher) : TextWatcher {
    override fun beforeTextChanged() {
        watcher.beforeTextChanged()
    }

    override fun onTextChanged() {
        watcher.onTextChanged()
    }

    override fun afterTextChanged() {
        watcher.afterTextChanged()
        println("wrapper")
    }
}

```

如上所示，创建一个类`TextWatcherImplWrapper`除了继承`TextWatcherImpl`类的功能外，在`afterTextChanged()`方法里又做了其它操作，相较于`TextWatcherImpl`类，除了在`afterTextChanged()`方法做了修改，另外两个方法没有修改，但是还必须创建出来

```kotlin
class TextWatcherImplWrapper(val watcher: TextWatcher) : TextWatcher by watcher {

    override fun afterTextChanged() {
        watcher.afterTextChanged()
        println("wrapper")
    }
}

```

**说明：**

1. `by`关键字可以看作是委托，将某操作委托给某对象
2. `:TextWatcher by watcher`作用是使对象`watcher`代替类`TextWatcherImplWrapper`实现接口`TextWatcher`
3. 对于对象`watcher`的唯一要求是实现了被代理的接口也就是`TextWatcher`
4. 使用此方法后，只需要在包装类内实现需要修改的方法即可，编译器会在编译时将其它方法添加到该类内，编译后的`TextWatcherImplWrapper`代码如下：

    ```java
    public final class TextWatcherImplWrapper implements TextWatcher {
        @NotNull
        private final TextWatcher watcher;

        public TextWatcherImplWrapper(@NotNull TextWatcher watcher) {
            this.watcher = watcher;
        }

        @NotNull
        public final TextWatcher getWatcher() {
            return this.watcher;
        }

        public void beforeTextChanged() {
            this.watcher.beforeTextChanged();
        }

        public void onTextChanged() {
            this.watcher.onTextChanged();
        }

        public void afterTextChanged() {
            this.watcher.afterTextChanged();
            String var1 = "wrapper";
            System.out.println(var1);
        }
    }
    ```

## 2. 属性代理

### 1. lazy

**lazy 的源码如下：**

```kotlin
@file:kotlin.jvm.JvmName("LazyKt")
import kotlin.reflect.KProperty

public actual fun <T> lazy(initializer: () -> T): Lazy<T> = SynchronizedLazyImpl(initializer)

private class SynchronizedLazyImpl<out T>(initializer: () -> T, lock: Any? = null) : Lazy<T>, Serializable {
    private var initializer: (() -> T)? = initializer
    @Volatile private var _value: Any? = UNINITIALIZED_VALUE
    // final field is required to enable safe publication of constructed instance
    private val lock = lock ?: this

    override val value: T
        get() {
            val _v1 = _value
            if (_v1 !== UNINITIALIZED_VALUE) {
                @Suppress("UNCHECKED_CAST")
                return _v1 as T
            }

            return synchronized(lock) {
                val _v2 = _value
                if (_v2 !== UNINITIALIZED_VALUE) {
                    @Suppress("UNCHECKED_CAST") (_v2 as T)
                } else {
                    val typedValue = initializer!!()
                    _value = typedValue
                    initializer = null
                    typedValue
                }
            }
        }

    override fun isInitialized(): Boolean = _value !== UNINITIALIZED_VALUE

    override fun toString(): String = if (isInitialized()) value.toString() else "Lazy value not initialized yet."

    private fun writeReplace(): Any = InitializedLazyImpl(value)
}

@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value

```

**说明：**

1. `lazy`实际上是一个函数，它接收了一个函数，即`lazy`为高阶函数
2. `lazy`函数返回一个`lazy`对象，该对象代理了**某个对象的实例的某属性的`getter`**
3. `lazy`只能代理只读属性 *（即用`val`修饰的属性）* 的`getter`
4. 其`lazy`的扩展函数`getValue()`, 其中入参`thisRef: Any?`表示属性所在类的实例；`property: KProperty<*>`表示所代理的属性的引用

```kotlin
class Person(val name: String) {
    val firstName by lazy { name.split(" ")[0] }
}
```

如上，接口`lazy`的实例代理了类`Person`的实例的属性`firstName`的`getter`方法，每次访问`firstName`时都会将`lazy`内的值返回出来，只有第一次调用时会计算，之后会存储起来。经过编译器编译后如下：

```kotlin

public final class Person {
   @NotNull
   private final Lazy firstName$delegate;
   @NotNull
   private final String name;

   @NotNull
   public final List getFirstName() {
      Lazy var1 = this.firstName$delegate;
      return (List)var1.getValue();
   }

   @NotNull
   public final String getName() {
      return this.name;
   }

   public Person(@NotNull String name) {
      this.name = name;
      this.firstName$delegate = LazyKt.lazy((Function0)(new Function0() {
         // $FF: synthetic method
         // $FF: bridge method
         public Object invoke() {
            return this.invoke();
         }

         @NotNull
         public final List invoke() {
            return StringsKt.split$default((CharSequence)Person.this.getName(), new char[]{" ".charAt(0)}, false, 0, 6, (Object)null);
         }
      }));
   }
}
```

### 2. observable

`observable`源码如下：

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public object Delegates {

public inline fun <T> observable(initialValue: T, crossinline onChange: (property: KProperty<*>, oldValue: T, newValue: T) -> Unit):
        ReadWriteProperty<Any?, T> =
    object : ObservableProperty<T>(initialValue) {
        override fun afterChange(property: KProperty<*>, oldValue: T, newValue: T) = onChange(property, oldValue, newValue)
    }
}
...
```

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public abstract class ObservableProperty<V>(initialValue: V) : ReadWriteProperty<Any?, V> {
    private var value = initialValue

    protected open fun beforeChange(property: KProperty<*>, oldValue: V, newValue: V): Boolean = true

    protected open fun afterChange(property: KProperty<*>, oldValue: V, newValue: V): Unit {}

    public override fun getValue(thisRef: Any?, property: KProperty<*>): V {
        return value
    }

    public override fun setValue(thisRef: Any?, property: KProperty<*>, value: V) {
        val oldValue = this.value
        if (!beforeChange(property, oldValue, value)) {
            return
        }
        this.value = value
        afterChange(property, oldValue, value)
    }
}
```

```kotlin
package kotlin.properties
import kotlin.reflect.KProperty

public interface ReadWriteProperty<in T, V> : ReadOnlyProperty<T, V> {

    public override operator fun getValue(thisRef: T, property: KProperty<*>): V

    public operator fun setValue(thisRef: T, property: KProperty<*>, value: V)
}
```

**说明：**

1. `Delegates.observable`返回了一个对象`ObservableProperty`, 该对象实现了接口`ReadWriteProperty`，因此有`getValue()`方法和`setValue()`方法，因此所代理的属性应为**可读写属性** *（即用 var 声明的属性）*
2. 接口`ReadWriteProperty`的`getValue()`方法和`setValue()`方法第一个参数为该属性所在类，第二个参数为所要代理的属性
3. `ObservableProperty`的实例代理了属性的`getter`和`setter`
4. 在`setValue()`方法中会执行`afterChange()`函数，`afterChange()`函数包含三个入参`property`表示当前属性，`oldValue`表示改变之前的值，`value`表示改变之后的值

示例代码如下：

```kotlin
class Person(var name: String) {
    //代理getter
    val firstName by lazy { name.split(" ")[0] }
    var secondName: String by Delegates.observable(name.split(" ")[1]) { property, oldValue, newValue ->
        println("value changed from $oldValue -> $newValue")
    }
}
```

**说明：**

1. 定义属性时使用关键字`by`定义要代理属性的对象
2. `Delegates.observable`对象后跟的`lambda`表达式即为上面提到的`afterChange()`函数，它可以获取当前属性、改变之前的值、改变之后的值

### 3. 自定义属性代理类

经过看`lazy`和`observale`源码可以发现，他们之所以可以代理属性，是因为实现了`getVaue()`、`setValue()`这两个运算符，所以只要实现了这两个运算符可即代理属性

```kotlin
fun main() {
    var str:String by ProxyX("321")
    println(str)
    str = "123"
    println(str)
}

class ProxyX(private var initialValue: String) {
    private var value = initialValue

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return "$value-Proxy"
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        this.value = this.value + value
    }
}
```

打印结果为

```kotlin
321-Proxy
321123-Proxy
```

**说明：**

1. 声明了一个代理类`ProxyX`，代理可读写属性，传入一个初始值，实现`getValue()`方法和`setValue()`方法
2. `getValue()`方法功能为：取值时返回一个字符串，该字符串是值加一个常量
3. `setValue()`方法功能为：将新添加的值拼接到原值的后面

### 4. 自定义属性代理

**实现效果：** 将`.properties`文件中的内容以变量的方式展示，以方便用户读写，文件中包含`author`、`version`、`desc`等信息

在工程里的`resources`文件夹下创建`Config.properties`文件，如下图所示：

![Config.properties位置](/kotlin学习系列五/kotlin_proxy_properties_1.png)

```kotlin
class PropertiesDelegate(private val path: String, private val defaultValue: String = "") {
    private lateinit var url: URL
    private val properties: Properties by lazy {
        val prop = Properties()
        url = try {
            javaClass.getResourceAsStream(path).use {
                prop.load(it)
            }
            javaClass.getResource(path)!!
        } catch (e: Exception) {
            try {
                ClassLoader.getSystemClassLoader().getResourceAsStream(path).use {
                    prop.load(it)
                }
                ClassLoader.getSystemClassLoader().getResource(path)!!
            } catch (e: Exception) {
                FileInputStream(path).use {
                    prop.load(it)
                }
                URL("file://${File(path).canonicalPath}")
            }
        }
        prop
    }

    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        return properties.getProperty(property.name, defaultValue)
    }

    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        properties.setProperty(property.name, value)
        File(url.toURI()).outputStream().use {
            properties.store(it, "Hello")
        }
    }
}

abstract class AbsProperties(path: String) {
    protected val prop = PropertiesDelegate(path)
}

class Config: AbsProperties("Config.properties") {
    var author by prop
    var version by prop
    var desc by prop
}

fun main() {
    val config = Config()
    println(config.author)
    println(config.version)
    config.version = "1.1.0"
    println(config.desc)
    println(config.version)
}
```

**说明：**

1. `resources`文件夹下内容编译完成后会被复制到`classpath`下，所以使用`ClassLoader`可以读取到文件
2. `resources`文件夹内容编译后位置如下图所示：

    ![confg.properties编译后位置](/kotlin学习系列五/kotlin_proxy_properties_2.png)

3. 上述代码创建一个`Properties`的代理类`PropertiesDelegate`，入参为一个地址，功能为：获取该地址文件里的所有属性值。
4. 注意创建的变量名要与`Config.properties`文件中的键值对里的键名一致，否则无法读取出属性
5. 代理类`PropertiesDelegate`也可以重新设置属性值，设置的属性值保存在编译后的文件内，并没有改变源文件，在修改时会自动添加所写的`comments`即`Hello`，并且自动添加了修改时间，效果上看上去就像是一个提交纪录

    ![修改后的config.properties](/kotlin学习系列五/kotlin_proxy_properties_3.png)

# 六、 单例 `object`

## 1. 定义

`Kotlin`中使用关键字`object`来定义**饿汉式单例**

```kotlin
object Singleton {

}
```

如上代码所示，只需要关键字`object`后加一个类名即可创建一个饿汉式单例，它等价于`Java`中如下所示的单例定义：

```java
public class Singleton {
    public static final Singleton INSTANCE = new Singleton();
}
```

## 2. 访问`object`的成员

`object`定义的单例可以当做类一样去添加成员属性，成员函数

```kotlin
object Singleton {
    var value: Int = 26
    fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
public static void main(String[] args) {
    Singleton.INSTANCE.plus();
    System.out.println("value:"+ Singleton.INSTANCE.getValue());
}
```

如上所示，在`kotlin`定义了一个单例`Singleton`，在`kotlin`中调用时直接使用单例的名字即可调用，但是在`java`调用时需要加一个`INSTANCE`，上述定义单例代码经过编译反后的`class`文件代码如下：

```java
public final class Singleton {
   private static int value;
   @NotNull
   public static final Singleton INSTANCE;

   public final int getValue() {
      return value;
   }

   public final void setValue(int var1) {
      value = var1;
   }

   public final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

## 3. 静态成员`@JvmStatic`

因`kotlin`是跨平台语言，其中`C`语言、`JavaScript`等都没有静态成员概念，所在`kotlin`没有静态成员的概念，但可以使用注解`JvmStatic`来模拟。

### 1. 在单例中使用`@JvmStatic`

```kotlin
object Singleton {
    @JvmStatic var value: Int = 26
    @JvmStatic fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
public static void main(String[] args) {
    Singleton.plus();
    System.out.println("value:"+ Singleton.getValue());
}
```

**说明：**

1. 如上所示，在`kotlin`中定义单例时为其成员添加了注解`@JvmStatic`。
2. 在`kotlin`中调用单例成员时没有任何变化
3. 在`java`中调用单例成员时可省略`INSTANCE`，可直接视同其调用静态成员

上述定义添加注解的单例代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   private static int value;
   @NotNull
   public static final Singleton INSTANCE;

   public static final int getValue() {
      return value;
   }

   public static final void setValue(int var0) {
      value = var0;
   }

   @JvmStatic
   public static final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

### 2. 在普通类中使用`@JvmStatic`

**注解`JvmStatic`只能使用在单例或普通类的伴生对象里的成员上**
**伴生对象 *(companion objects)*：** 是一个类的伴生对象，定义一个类时可以同时在类内定义定义一个`object`

```kotlin
class Foo {
    companion object {
        @JvmField var value:String = "Test"
        @JvmStatic fun y() {
            println("Test")
        }
    }
}

fun main() {
    Foo.y()
    println("value:${Foo.value}")
}
```

```java
public static void main(String[] args) {
    Foo.y();
    System.out.println("value:"+ Foo.value);
}
```

**说明：**

1. 如上所示，在`kotlin`中定义伴生对象时使用`companion object`
2. 在`kotlin`中调用类的伴生对象中的静态成员时，直接使用类名即可
3. 在`java`中调用类的伴生对象中的静态成员时，直接使用类名即可

上述定义伴生对象的类代码经过反编译后的`class`文件代码如下：

```java
public final class Foo {
   @JvmField
   @NotNull
   public static String value = "Test";
   @NotNull
   public static final Foo.Companion Companion = new Foo.Companion((DefaultConstructorMarker)null);

   @JvmStatic
   public static final void y() {
      Companion.y();
   }

   public static final class Companion {
      @JvmStatic
      public final void y() {
         String var1 = "Test";
         boolean var2 = false;
         System.out.println(var1);
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

## 4. `@JvmField`

**注解`@JvmField`作用：** 不生成`getter/setter`，但不会给所修饰的属性添加静态属性，只是该属性变成`public`

### 1. 在单例中使用`@JvmField`

```kotlin
object Singleton {
    @JvmField var value: Int = 26
    @JvmStatic fun plus() {
        value++
    }
}

fun main() {
    Singleton.plus()
    println("value${Singleton.value}")
}
```

```java
//java

public static void main(String[] args) {
    Singleton.plus();
    System.out.println("value:"+ Singleton.value);
}
```

**说明：**

1. 如上所示，为单例中的属性`value`添加注解`@JvmField`
2. `kotlin`中调用并没有改变
3. `java`中调用时直接使用该变量名，它成为了一个全局静态变量

上述定义添加注解的单例代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   @JvmField
   public static int value;
   @NotNull
   public static final Singleton INSTANCE;

   @JvmStatic
   public static final void plus() {
      int var10000 = value++;
   }

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
   }
}
```

### 2. 在普通类中使用`@JvmField`

```kotlin
class Foo {
    @JvmField var x: Int = 22
}

fun main() {
    val foo = Foo()
    println(foo.x)
}
```

```java
public static void main(String[] args) {
    Foo foo = new Foo();
    System.out.println(foo.x);
}
```

**说明：**

1. 如上所示，在普通类中使用注解`@JvmField`修饰一个属性
2. 注解`@JvmField`并没有给属性静态属性，所以必须要声明一个对象再调用它

上述定义添加注解的普通类经过反编译后的`class`文件代码如下：

```java
public final class Foo {
   @JvmField
   public int x = 22;
}
```

## 5. `object`的构造器

`kotlin`中使用`object`定义的单例不能自定义构造器，系统会自动生成一个无参构造器，但是可以自定义若干个`init`块

`kotlin`中单例可以与普通类一样继承或实现其它的类或接口

```kotlin
object Singleton {
    @JvmField var value: Int = 26
    init {
        value *= 2
    }
}
```

上述代码经过反编译后的`class`文件代码如下：

```java
public final class Singleton {
   @JvmField
   public static int value;
   @NotNull
   public static final Singleton INSTANCE;

   private Singleton() {
   }

   static {
      Singleton var0 = new Singleton();
      INSTANCE = var0;
      value = 26;
      value *= 2;
   }
}
```

# 七、内部类`inner class`

## 1. 定义

在`Java`中定义内部类，只需在一个类内再定义一个类，类内的类即为内部类，内部类分为一般内部类、静态内部类。一般内部类持有外部类的引用，所以有可能引起内存泄露，静态内部类没有外部类的引用，所以不能直接调用外部的方法。

```java
public class Outer {
    class Inner {

    }
    static class StaticInner {

    }
}
```

在`kotlin`中定义**非静态内部类** 使用关键字`inner`，而不加关键字的类内类是**静态内部类**

```kotlin
class Outer {
    inner class Inner
    class StaticInner
}
```

## 2. 实例化

对于内部类的实例化，`java`和`kotlin`操作相同。
初始化非静态内部类时需要**先构造外部类对象** ，然后再拿对象去初始化内部类
初始化静态内部类时，只引用了**外部类的类名** ，不需要初始化外部类

```kotlin
//kotlin --非静态内部类
val inner = Outer().Inner()
```

```java
//java --非静态内部类
Outer.Inner inner = new Outer().new Inner();
```

```kotlin
//kotlin --静态内部类
val staticInner = Outer.StaticInner()
```

```java
//java --静态内部类
Outer.StaticInner inner = new Outer.StaticInner();
```

## 3. 单例`object`的内部`object`

```kotlin
object OuterObject {
    object InnerObject
}
```

`object`一旦定义完成后即会被实例化，不存在非静态的情况，故不可用`inner`修饰

## 4. 匿名内部类

`java`中的匿名内部类表现形式如下：

```java
new Runnable() {

    @Override
    public void run() {

    }
};
```

`kotlin`中的匿名内部类形式如下：

```kotlin
object: Runnable {
    override fun run() {
        TODO("Not yet implemented")
    }
}
```

```kotlin
val btn: Button = Button(context)
btn.post { object: Runnable, Closeable {
    override fun run() {
        TODO("Not yet implemented")
    }

    override fun close() {

    }

}}
```

**说明：**

1. 匿名内部类如果定义在非静态区域内就会持有外部引用，容易造成内存泄漏。
2. **静态区域** 即静态方法、静态类。在`kotlin`中如果定义在伴生对象中或顶级函数内都不会持有外部引用
3. 如上所示，`kotlin`中定义匿名内部类只是省略了名字，并且可以**继承父类或实现多个接口**
4. 实现多个接口的匿名内部类的类型为 ***交叉类型***，如上代码`Buttn`的`post`方法里的匿名内部类类型为`Cloneable & Runnable`
5. 若`java`中要实现`kotlin`中匿名内部类的可以继承父类或实现多个接口的效果，可以使用**本地类**

**本地类**

本地类又称为局部内部类，理解为有名字的匿名类。它是定义在一个方法或者一个作用域里面的类，它和成员内部类的区别在于**局部内部类的访问仅限于方法内或者该作用域内**。局部内部类就像是方法里面的一个局部变量一样，是不能有 `public`、`protected`、`private` 以及 `static` 修饰符的

```java
public static void main(String[] args) {
    class PreOperation implements Runnable, Closeable {

        @Override
        public void close() throws IOException {

        }

        @Override
        public void run() {

        }
    }
    PreOperation preOperation = new PreOperation();
    preOperation.run();
}
```

# 八、数据类`data class`

## 1. 定义

只需要在普通类前面加关键字`data`即为数据类。
数据类对标`java`中的`bean`类，但两者并不相等
**数据类必须至少有一个主构造函数参数**

```kotlin
class Person(var name: String, var age: Int)
data class Book(val id: Long, val name: String, val brand:Person)
```

## 2. component

定义在主构造器中的属性又称之为 ***component***，数据类所有的东西都基于`component`实现，如下所示为上述代码反编译`class`文件后代码：

```java
public final class Computer {
   private final long id;
   @NotNull
   private final String name;
   @NotNull
   private final Person brand;

   public final long getId() {
      return this.id;
   }

   @NotNull
   public final String getName() {
      return this.name;
   }

   @NotNull
   public final Person getBrand() {
      return this.brand;
   }

   public Computer(long id, @NotNull String name, @NotNull Person brand) {
      this.id = id;
      this.name = name;
      this.brand = brand;
   }

   public final long component1() {
      return this.id;
   }

   @NotNull
   public final String component2() {
      return this.name;
   }

   @NotNull
   public final Person component3() {
      return this.brand;
   }

   @NotNull
   public final Computer copy(long id, @NotNull String name, @NotNull Person brand) {
      return new Computer(id, name, brand);
   }

   @NotNull
   public String toString() {
      return "Computer(id=" + this.id + ", name=" + this.name + ", brand=" + this.brand + ")";
   }

   public int hashCode() {
      long var10000 = this.id;
      int var1 = (int)(var10000 ^ var10000 >>> 32) * 31;
      String var10001 = this.name;
      var1 = (var1 + (var10001 != null ? var10001.hashCode() : 0)) * 31;
      Person var2 = this.brand;
      return var1 + (var2 != null ? var2.hashCode() : 0);
   }

   public boolean equals(@Nullable Object var1) {
      if (this != var1) {
         if (var1 instanceof Computer) {
            Computer var2 = (Computer)var1;
            if (this.id == var2.id && Intrinsics.areEqual(this.name, var2.name) && Intrinsics.areEqual(this.brand, var2.brand)) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }

   // $FF: synthetic method
   public static Computer copy$default(Computer var0, long var1, String var3, Person var4, int var5, Object var6) {
      if ((var5 & 1) != 0) {
         var1 = var0.id;
      }

      if ((var5 & 2) != 0) {
         var3 = var0.name;
      }

      if ((var5 & 4) != 0) {
         var4 = var0.brand;
      }

      return var0.copy(var1, var3, var4);
   }
}
```

**说明：**

1. 数据类会自动生成`N`个`component`，序号顺序即为主构造器中参数顺序
2. 可以根据`component`方法获取数据类的属性值

    ```kotlin
    val computer = Computer(1001L, "mackBook Pro", Person("Apple", 45))
    val id = computer.component1()
    val name = computer.component2()
    val brand = computer.component3()
    ```

3. 相较于普通类，数据类还自动创建了`copy()`、`toString()`、`hashCode()`、`equals()`等方法

## 3. 数据类解构

数据类解构就是将一个数据类解构 *(destructure)* 为多个变量，也就是意味着一个解构声明会一次性创建多个变量

`Pair`的源码如下：

```kotlin
//Pair类源码
public data class Pair<out A, out B>(
    public val first: A,
    public val second: B
) : Serializable {

    public override fun toString(): String = "($first, $second)"
}
```

```kotlin
val pair = "Hello" to "World"
val(hello, world) = pair
```

**说明：**

1. 如上所示，先声明一个`Pair`类型的属性`pair`，之后将`pair`解构成变量`hello`和`world`
2. `Pair`类是一个数据类，有两个`component`
3. 上述代码反编译`class`文件后代码如下：

   ```java
    Pair pair = TuplesKt.to("Hello", "World");
    String var6 = (String)pair.component1();
    String world = (String)pair.component2();
   ```

   由此可见**数据类解构是借助`component`实现的**

上一节中定义的`Computer`实现类可以使用如下代码解构：

```kotlin
fun main() {
    val computer = Computer(31L, "mackBook Pro", Person("Apple", 45))
    val(id, name, brand) = computer
}
```

## 4. JavaBean 与 data class 区别

||JavaBean|data class|
|--|--|--|
|构造方法|默认无参构造|属性作为参数|
|字段|字段私有，Getter/Setter 公开|属性|
|继承性|可继承也可被继承|不可被继承|
|component|无|相等性、解析等|

数据类不可被继承原因：**违反相等性的对称性和传递性**

假设定义一个数据基类`BaseClass`内含一个属性`name`，之后再定义一个数据子类`ChildClass`继承了`BaseClass`，并在其基础上又扩展了一个属性`age`。之后再创建如下三个实例

|实例|name 值|age 值|
|--|--|--|
|基类`BaseClass`的实例：a|Lee|无|
|子类`BaseClass`的实例：b|Lee|18|
|子类`BaseClass`的实例：c|Lee|20|

则：
|表达式|结果|原因|
|--|--|--|
|a == b|true|双等号其实是调用的类实例 a 的`equals`方法，由反编译的代码可知`equals`方法只比较本类有的属性，而两个实例的`name`属性值相同，故相等|
|b == a|false|实例 b 的`equals`方法，要比较`name`和`age`两个属性，而实例`a`没有`age`属性，故不相等|
|a == c|true|同第一条原因|
|c == a|false|同第二条原因|
|b == c|false|虽然实例`b`和实例`c`都有`name`和`age`两个属性，但是`age`属性值不同，故不相等|

由上表可知：

`a == b` 但是 `b != a`违反了相等性的**对称性**
`a == b` 并且 `a == c` 但`b != c`违反了相等性的**传递性**

## 5. data class 总结

1. 提供了`JavaBean`的功能
2. 被`final`字段修饰不可被继承
3. 核心的`Component`，它导致必须有有参数的主构造器，并决定了数据类的相等性，还可解构数据类
4. 定义一个数据类应该将其当作一个纯数据结构来使用，大多数情况下不需要额外实现，即一般省略`{}`
5. 数据类的属性类型最好为基本类型、String、其它数据类等，以此保证不会有逻辑，保证数据类是纯数据
6. `Component`不可以自定义`Getter/Setter`，以保证数据特性不会被篡改
7. 数据类的属性最好使用`val`修饰，使属性不可变，以保证其同一个对象的`hashcode`和`equals`结果前后一样

**将`data class`当作`JavaBean`使用**

**问题：** 必须有有参数的主构造器；被 final 修饰不可被继承。

**解决方案：**

1. 使用<span id="jumpNoArg">`NoArg`插件</span>，给数据类加一个注解，使用数据类在编译时生成一个无参构造器，但无法在代码里直接访问该构造器，只能通过反射拿到。
2. 使用`AllOpen`插件，去掉`final`修饰，以保证数据类可以被继承

**实例：**

```groovy
//build.gradle.kts
plugins {
    ...
    id ("org.jetbrains.kotlin.plugin.allopen") version "1.4.21"
    id ("org.jetbrains.kotlin.plugin.noarg") version "1.4.21"
}

noArg {
    invokeInitializers = true
    annotations("cn.ltt.projectcollection.kotlin.DataClassAnnotation")
}

allOpen {
    annotations("cn.ltt.projectcollection.kotlin.DataClassAnnotation")
}
```

```java
package cn.ltt.projectcollection.kotlin;

public @interface DataClassAnnotation {
}
```

**说明：**

1. 插件需要添加到`module`模块下的`build.gradle.kts`中
2. 如上所示，将注解`DataClassAnnotation`会导致被标注的类生成无参构造函数且类本身及其所有成员会变为开放
3. `invokeInitializers`为`false`时，只会调用被标注的类的父类的构造方法；为`true`时会调用被标注类的`init`块，默认为`false`。
4. 参考文章：[https://www.kotlincn.net/docs/reference/compiler-plugins.html](https://www.kotlincn.net/docs/reference/compiler-plugins.html)


```kotlin
@DataClassAnnotation
data class Computer(val id: Long, val name: String, val brand:Person) {
    init{
        println(name)
    }
}
fun main() {
    val computer = Computer::class.java.newInstance()
}
```

将注解添加到类`Computer`上，并通过反射生成一个`Computer`的实例
反编译`Computer`类的`class`文件后代码如下：

```java
public class Computer {
   private final long id;
   @NotNull
   private final String name;
   @NotNull
   private final Person brand;

   public long getId() {
      return this.id;
   }

   @NotNull
   public String getName() {
      return this.name;
   }

   @NotNull
   public Person getBrand() {
      return this.brand;
   }

   public Computer(long id, @NotNull String name, @NotNull Person brand) {
      Intrinsics.checkNotNullParameter(name, "name");
      Intrinsics.checkNotNullParameter(brand, "brand");
      super();
      this.id = id;
      this.name = name;
      this.brand = brand;
      String var5 = this.getName();
      boolean var6 = false;
      System.out.println(var5);
   }

   public final long component1() {
      return this.getId();
   }

   @NotNull
   public final String component2() {
      return this.getName();
   }

   @NotNull
   public final Person component3() {
      return this.getBrand();
   }

   @NotNull
   public final Computer copy(long id, @NotNull String name, @NotNull Person brand) {
      Intrinsics.checkNotNullParameter(name, "name");
      Intrinsics.checkNotNullParameter(brand, "brand");
      return new Computer(id, name, brand);
   }

   // $FF: synthetic method
   public static Computer copy$default(Computer var0, long var1, String var3, Person var4, int var5, Object var6) {
      if (var6 != null) {
         throw new UnsupportedOperationException("Super calls with default arguments not supported in this target, function: copy");
      } else {
         if ((var5 & 1) != 0) {
            var1 = var0.getId();
         }

         if ((var5 & 2) != 0) {
            var3 = var0.getName();
         }

         if ((var5 & 4) != 0) {
            var4 = var0.getBrand();
         }

         return var0.copy(var1, var3, var4);
      }
   }

   @NotNull
   public String toString() {
      return "Computer(id=" + this.getId() + ", name=" + this.getName() + ", brand=" + this.getBrand() + ")";
   }

   public int hashCode() {
      long var10000 = this.getId();
      int var1 = (int)(var10000 ^ var10000 >>> 32) * 31;
      String var10001 = this.getName();
      var1 = (var1 + (var10001 != null ? var10001.hashCode() : 0)) * 31;
      Person var2 = this.getBrand();
      return var1 + (var2 != null ? var2.hashCode() : 0);
   }

   public boolean equals(@Nullable Object var1) {
      if (this != var1) {
         if (var1 instanceof Computer) {
            Computer var2 = (Computer)var1;
            if (this.getId() == var2.getId() && Intrinsics.areEqual(this.getName(), var2.getName()) && Intrinsics.areEqual(this.getBrand(), var2.getBrand())) {
               return true;
            }
         }

         return false;
      } else {
         return true;
      }
   }

   public Computer() {
   }
}
```

**说明：**

1. 可以看到类`computer`已经没有`final`修饰了，它可以被继承，且`computer`里的属性的`Getter/Setter`也没有了`final`修饰
2. 相较于之前的反编译后的代码，现在在最后的位置多了一个无参的构造器
3. 虽然在插件`noarg`里将`invokeInitializers`置为了`true`，但是`IntelliJ`在反编译时没有识别到该属性，所以在无参构造器中没有`init`块中的代码，但是`gradle`可以识别到该属性，所以在运行时会执行`init`块中代码，会打印出`computer`类实例的`name`值

# 九、枚举类`enum class`

## 1. 定义

使用关键字 **`enum`**

```java
enum State {
    START, FINISHED
}
```

```kotlin
enum class State {
    START, FINISHED
}
```

## 2. 属性

**Java:**

1. 可以通过枚举项的`name()`方法，获取枚举项的名称
2. 可以通过枚举项的`ordinal()`方法，获取枚举项的在枚举声明里的位置

```java
System.out.println(State.START.name());
System.out.println(State.START.ordinal());
```

**Kotlin:**

1. 枚举项有属性`name`，持有枚举项的名称
2. 枚举项有属性`ordinal`，持有枚举项在枚举声明中的位置
3. 枚举有方法`values`，该方法返回枚举项列表

```kotlin
println(State.START.name)
println(State.START.ordinal)
```

## 3. 构造器

**Java:**

```java
enum State {
    START(0), FINISHED(1);
    
    int id;
    State(int id) {
        this.id = id;
    }
}
```

**Kotlin:**

```kotlin
enum class State(val id: Int) {
    START(0), FINISHED(1)
}
```

**说明：**

1. 声明构造器后，每个枚举项都要调用构造器。只有通过这种方式来创建有构造器的枚举项。


## 4. 实现接口

### 1. 统一实现

**Java:**

```java
enum State implements Runnable{
    START, FINISHED;

    @Override
    public void run() {
        System.out.println("For every state");
    }
}
```

**Kotlin:**

```kotlin
enum class State : Runnable {
    START, FINISHED;
    
    override fun run() {
        println("For every state")
    }
}
```

**说明：**

1. 如上所示，每个枚举项的`run`方法实现都是同样的
2. 其中`Kotlin`中最后一个枚举项后必须加上分号，否则报错：`Expecting ';' after the last enum entry or '}' to close enum class body`

### 2. 各自实现

**Java:**

```java
enum State implements Runnable {
    START {
        @Override
        public void run() {
            System.out.println("For START");
        }
    }, FINISHED {
        @Override
        public void run() {
            System.out.println("For FINISHED");
        }
    }
}
```

**Kotlin:**

```kotlin
enum class State : Runnable {
    START {
        override fun run() {
            println("For START")
        }
    },
    FINISHED {
        override fun run() {
            println("For FINISHED")
        }
    }
}
```

**说明：**

1. 每个实例单独实现`run`方法，这样各个枚举项就只调用自己的`run`方法
2. 枚举的父类是`Enum`，所以不能继承其它类

## 5. 为枚举定义扩展

```kotlin
fun State.next(): State {
    return State.values().let { 
        val nextOrdinal = (ordinal + 1) % it.size
        it[nextOrdinal]
    }
}
```

**说明：**

1. 枚举也是一个类，可以为其定义扩展方法
2. 如上代码为枚举类`State`定义了一个扩展方法`next()`，该方法可以获取某个枚举项的下一个枚举，且是循环的

## 6. 条件分支


```kotlin
val state = State.START
val value = when(state) {
    State.START -> {0}
    State.FINISHED -> {1}
}
```

**说明：**

1. 枚举是有限个数的，所以可以将其作为条件分支

## 7. 比较大小

```kotlin
val state = State.START
if (state <= State.FINISHED) {
    println("yes")
}
```

**说明：**

1. 同一个枚举类下的枚举项之间可以比较大小
2. 枚举项之间比较的是`ordinal`

## 8. 区间

枚举是有顺序的，可以创建枚举区间

```kotlin

enum class Status {
    NEW, RUNNABLE, RUNNING, BLOCKED, DEAD
}
fun main() {
    val statusRange = Status.NEW .. Status.BLOCKED
    val status = Status.RUNNING
    println(status in statusRange)
}
```

**说明：**

1. 如上所示，先创建了一个线程状态的枚举类，之后又创建了一个枚举项的区间
2. 判断某个状态是否在某几个状态之内，这时使用区间会很方便

# 十、密封类`sealed class`

## 1. 概念

1. 密封类是一种特殊的抽象类，它首先是一个抽象类，其次才是密封类，即密封类可以被继承
2. 密封类的子类定义只能在与自身相同的文件中，即其子类只在一个有限范围内
3. 密封类的子类的个数是有限的

## 2. 定义

使用关键字`sealed`来定义密封类


```kotlin
sealed class PlayerState {
    var id:Int = 0
    constructor()
    constructor(id:Int) {
        this.id = id
    }
}
```

上述代码反编译后代码如下：

```java
public abstract class PlayerState {
   private int id;

   public final int getId() {
      return this.id;
   }

   public final void setId(int var1) {
      this.id = var1;
   }

   private PlayerState() {
   }

   private PlayerState(int id) {
      this.id = id;
   }
}
```

**说明：**

1. 密封类是一个抽象类
2. 密封类的构造方法是私有的


## 3. 密封类的子类

```kotlin
object Idle : PlayerState()

class Playing(val song: String): PlayerState()

class Error(val errorInfo: String):PlayerState()
```

**说明：**

1. `Idle`是一个普通单例对象
2. `Playing`类的实例每次播放的歌曲不同，所以它每次的实例不一样
3. `Error`类的实例每次错误信息不同，所以它每次的实例也不一样
4. 但是`Idle`、`Playing`、 `Error`的类型是一样的
5. 继承密封类的时候要调用父类的构造器
   
## 4. 子类分支

密封类的子类是可数的，分支就可完备，就可以使用`when`分支语句

```kotlin
val state : PlayerState = Idle
when(state) {
    Idle -> {
        println("Idle")
    }
    is Playing -> {
        println("Playing")
    }
    is Error -> {
        println("Error")
    }
}
```

## 5. 密封类使用

```kotlin

fun main() {
   val player = Player()
    player.play("The Nights")
}
sealed class PlayerState

object Idle : PlayerState()

class Playing(val song: String): PlayerState() {
    fun start(){}
    fun stop(){}
}

class Error(val errorInfo: String):PlayerState() {
    fun recover() {}
}

class Player {
    var state: PlayerState = Idle

    fun play(song: String) {
        this.state = when(val state = this.state) {
            Idle -> {
                Playing(song).also(Playing::start)
            }
            is Playing -> {
                state.stop()
                Playing(song).also(Playing::start)
            }
            is Error -> {
                state.recover()
                Playing(song).also(Playing::start)
            }
        }
    }
}

```

**说明：**

1. 定义一个播放状态的密封类`PlayerState`，然后三个类实现密封类：`Idle`初始化状态类、`Playing`播放状态类、`Error`错误状态类
2. 创建一个播放器类`Player`，该类中的方法`play()`通过`when`语句判断不同的状态采用不同的处理方案：如果是`Idle`状态直接播放；如果是`Playing`状态则先将停止播放再播放新歌曲；如果是`Error`状态则先复位再播放新歌曲。
3. 其中`Playing`和`Error`状态的判断还进行了智能类型转换，以方便调用各自状态类中的方法。
4. 其中的`also`方法会将`receiver`返回，所以实现了状态机`state`的流转。
5. `when`语句在`kotlin1.3`以后可以在其中创建一个新变量

## 6. 密封类和枚举类对比

||密封类|枚举类|
|--|--|--|
|状态实现|子类继承|类实例化|
|状态可数|子类可数|实例可数|
|状态差异|类型差异|值差异|

**说明：**

1. 密封类可以让子类继承子类可以是`object`类，而枚举类内的枚举对象都是枚举类的实例

# 十一、内联类`inline class`

## 1. 概念

1. 内联类是对某一个类型的包装
2. 内联类是类似于`java`装箱类型 *(Float, Double等)* 的一种类型
3. 编译器会尽可能使用被包装的类型进行优化
4. 内联类在`Kotlin 1.3`版本中处于公测阶段，谨慎使用

## 2. 定义

使用关键字`inline`来定义内联类

```kotlin
inline class BoxInt(val value:Int)
```

**说明：**

1. 如上代码所示，定义内联类`BoxInt`是对类型`Int`的包装
2. 被包装的类型只能定义在主构造器内
3. 主构造器只能是`public`
4. 主构造器只能有一个参数即被包装的类型，
5. 主构造器里的参数只能用`val`修饰

## 3. 方法

```kotlin
inline class BoxInt(val value:Int) {
    operator fun inc():BoxInt {
        return BoxInt(value + 1)
    }
}
```

**说明：**

1. 内联类可以定义方法，如上代码所示，定义了一个重载运算符的函数`inc()`,如此可以执行运行符`++`


## 4. 属性

```kotlin
inline class BoxInt(val value: Int) {
    val name
        get() = "BoxInt($value)"

}
```

**说明：**

1. *Inline class cannot have properties with backing fields*
2. 内联类可以定义只有`getter`的属性，即内联类只能定义方法

## 5. 继承关系

```kotlin
inline class BoxInt(val value: Int) : Comparable<Int> {
    override fun compareTo(other: Int): Int {
        return value.compareTo(other)
    }
}
fun main() {
    val boxInt = BoxInt(5)
    println(boxInt > 10)
}
```

**说明：**

1. 如上代码内联类`BoxInt`实现了接口`Comparable`，则`BoxInt`类型的对象可以与`Int`类型比较大小
2. 内联类可以实现接口
3. 内联类不能继承父类，也不能被继承

## 6. 编译优化

```kotlin
fun main() {
    var boxInt = BoxInt(5)
    val newValue = boxInt.value * 200
    println(newValue)
    boxInt++
    println(boxInt)
}
```

将如上代码反编译后代码如下：

```java
public final class BoxInt implements Comparable {
   private final int value;

   public int compareTo(int var1) {
      return compareTo-impl(this.value, var1);
   }

   // $FF: synthetic method
   // $FF: bridge method
   public int compareTo(Object var1) {
      return this.compareTo(((Number)var1).intValue());
   }

   public final int getValue() {
      return this.value;
   }

   // $FF: synthetic method
   private BoxInt(int value) {
      this.value = value;
   }

   public static final int inc_xSVgFbQ/* $FF was: inc-xSVgFbQ*/(int $this) {
      return constructor-impl($this + 1);
   }

   public static int compareTo_impl/* $FF was: compareTo-impl*/(int $this, int other) {
      return Intrinsics.compare($this, other);
   }

   public static int constructor_impl/* $FF was: constructor-impl*/(int value) {
      return value;
   }

   // $FF: synthetic method
   public static final BoxInt box_impl/* $FF was: box-impl*/(int v) {
      return new BoxInt(v);
   }

   public static String toString_impl/* $FF was: toString-impl*/(int var0) {
      return "BoxInt(value=" + var0 + ")";
   }

   public static int hashCode_impl/* $FF was: hashCode-impl*/(int var0) {
      return var0;
   }

   public static boolean equals_impl/* $FF was: equals-impl*/(int var0, Object var1) {
      if (var1 instanceof BoxInt) {
         int var2 = ((BoxInt)var1).unbox-impl();
         if (var0 == var2) {
            return true;
         }
      }

      return false;
   }

   public static final boolean equals_impl0/* $FF was: equals-impl0*/(int p1, int p2) {
      return p1 == p2;
   }

   // $FF: synthetic method
   public final int unbox_impl/* $FF was: unbox-impl*/() {
      return this.value;
   }

   public String toString() {
      return toString-impl(this.value);
   }

   public int hashCode() {
      return hashCode-impl(this.value);
   }

   public boolean equals(Object var1) {
      return equals-impl(this.value, var1);
   }
}

public static final void main() {
    int boxInt = BoxInt.constructor-impl(5);
    int newValue = boxInt * 200;
    System.out.println(newValue);
    boxInt = BoxInt.inc-xSVgFbQ(boxInt);
    BoxInt var4 = BoxInt.box-impl(boxInt);
    System.out.println(var4);
}

```

**编译优化说明：**

1. 调用类`BoxInt`的`constructor-impl`方法为属性`boxInt`赋值，而`constructor-impl`方法只是将形参返回，即第一条语句优化为：`var boxInt: Int = 5`
2. 第二条语句直接使用类型为被封闭类型的`boxInt`属性：`val newValue = boxInt * 200`
3. 打印语句只是将属性打印
4. 第四条作为为自加一的语句调用类`BoxInt`的`inc-xSVgFbQ`方法，而该方法又调用了`contructor-impl`方法，只是入参传入的是其本身的值加一
5. 而最后一条语句是打印类`BoxInt`的实例，即调用该类的`toString()`方法，所以将之前的`boxInt`属性再装箱。调用`BoxInt`类的`box-impl`方法，创建一个`BoxInt`的实例，之后打印该实例
6. 综上所述，只有在必要的时候才使用包装类型，大多数情况下优化为被包装的类型

## 7. 内联类使用场景

### 1. 官方例子

```kotlin
public inline class UInt @PublishedApi internal constructor(@PublishedApi internal val data: Int) : Comparable<UInt> {
...
}

```

**说明：**

1. 如上所示无符号整型是有符号整形的包装
2. 所有的无符号类型都是对应有符号类型的包装

### 2. 使用内联类模拟枚举

```kotlin
inline class State(val ordinal:Int) {
    companion object {
        val Start = State(0)
        val Running = State(1)
        val Finished = State(2)
    }
    fun values() = arrayOf(Start, Finished)
    val name: String
        get() {
            return when (ordinal) {
                0 -> {
                    "Start"
                }
                1 -> {
                    "Running"
                }
                2 -> {
                    "Finished"
                }
                else -> {
                    "other"
                }
            }
        }
}

fun main() {
    setState(State.Running)
}

fun setState(state:State) {
    println("setState-State:$state")
}
```

**说明：**

1. 枚举的内存开销大
2. 如上所示，创建一个内联类`State`模拟枚举，该类提供了`values`函数和只有`getter`的`name`属性，`State`类的单例中创建了三个内联对象
3. 在`java`中使用注解`IntDef`、`StingDef`等来模拟枚举类，虽然`IDE`会报错但是可以编译通过。但是在`kotlin`中该注解不会起到规定范围的作用
4. 内联类虽然在`companion object`中定义了几个对象，但编译器会将其优化为整型，如此相较于枚举内存开销更小
5. 在某函数中设置该内联类类型时，只能传该内联类单例中定义的对象
6. 两种语言模拟枚举类的区别：在`java`中使用注解`IntDef`、`StingDef`等，虽然`IDE`会报错但是可以编译通过，而`kotlin`中的使用内联类，`IDE`会报错且不会编译通过

## 8. 内联类的限制

1. 主构造器必须有且仅有一个只读属性 *(被包装的类型)*
2. 不能定义有`backing-field`的其它属性
3. 被包装类型必须不能是泛型类型，必须是一个确定的类型
4. 不能继承父类也不能被继承
5. 内联类不能定义为其他类的内部类
6. 以上的限制是为了编译器优化类型

## 9. 别名 *(`typealias`)* 与内联类 *(`inline class`)* 区别

||typealias|inline class|
|--|--|--|
|类型|没有新类型|有包装类型产生|
|实例|与原类型一致|必要时使用包装类型|
|场景|类型更直观|优化包装类型性能|

**说明：**

1. `typealias`是为某类型定义了一个别名，而内联类是将某类型进行包装
2. `typealias`是为程序员看的更直观，而内联类是为编译器优化类型的

# 十二、应用 - 数据类序列化

常用的三个`Kotlin`中的`Json`序列化工具：

1. `Gson`框架是`Google`的。官网：[https://github.com/google/gson](https://github.com/google/gson)
2. `Moshi`是大神`JakeWharton`等人写的。官网：[https://github.com/square/moshi](https://github.com/square/moshi)
3. `Kotlinx.serialization`是`Kotlin`官方提供的。官网：[https://github.com/Kotlin/kotlinx.serialization](https://github.com/Kotlin/kotlinx.serialization)

## 1. 引入工具

### 1. Gson

直接在在模块下的`build.gradle`文件里的`dependences`块添加依赖即可

```groovy
//app/src/build.gradle.kts
dependencies {
    //Gson
    implementation ("com.google.code.gson:gson:2.8.6")
}
```

### 2. Moshi

要先添加一下插件，再添加依赖

```groovy
//app/src/build.gradle.kts

plugins {
    kotlin("kapt")
}

dependencies {
    implementation("com.squareup.moshi:moshi:1.11.0")
    implementation( "com.squareup.moshi:moshi-kotlin:1.8.0") // for KotlinJsonAdapterFactory
    kapt( "com.squareup.moshi:moshi-kotlin-codegen:1.11.0") // for generated Json Adapter
}
```


### 3. Kotlinx.serialization

要先添加一下插件，再添加依赖

```groovy
//app/src/build.gradle.kts
plugins {
    ...
    kotlin("plugin.serialization") version "1.4.30"
}
dependencies {
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.1.0")
}
```

## 2. 示例代码

**Kotlin版本：** `Kotlin version 1.4.20-release-308 (JRE 1.8.0_251-b08)
`
**Kotlin Plugin版本：** `1.4.31`

### 1. 简单序列化json


```kotlin
fun main() {
    val jsonStr = """{"name":"Lee","age":20}"""
    val person = Person("Lee", 18)


    //Gson
    println("Gson")
    val gson = Gson()
    println(gson.toJson(person))
    println(gson.fromJson(jsonStr, Person::class.java))
    println()


    //Moshi
    println("Moshi")
    val moshi = Moshi.Builder()
            .add(KotlinJsonAdapterFactory())
            .build()
    val jsonAdapter = moshi.adapter(Person::class.java)
    println(jsonAdapter.toJson(person))
    println(jsonAdapter.fromJson(jsonStr))
    println()


    println("Kotlinx.serialization")
    val data = PersonForSerialize("Lee", 18)
    println(Json.encodeToString(PersonForSerialize.serializer(),data))
    println(Json.decodeFromString(PersonForSerialize.serializer(),jsonStr))
}


data class Person(val name: String, val age: Int)

@Serializable
data class PersonForrSerialize(val name: String, val age: Int)
```

打印结果如下：

```java
Gson
{"name":"Lee","age":18}
Person(name=Lee, age=20)

Moshi
{"name":"Lee","age":18}
Person(name=Lee, age=20)

Kotlinx.serialization
{"name":"Lee","age":18}
PersonFroSerialize(name=Lee, age=20)
```

**说明：**

1. 区别：

    ||Gson|Moshi|Kotlinx.serialization|
    |--|--|--|--|
    |对Json对象的需求|无|无|需要Json对象的类添加注解`@Serializable`|
    |需要创建工具类对象|需要|需要，且还要再创建对应`json`对象的`adapter`|不需要|

2. `Moshi`在使用`Builder`模式创建对象时还要添加`KotlinJsonAdapterFactory`以支持`kotlin`
3. `Kotlinx.serialization`使用方法`encodeToString`和`decodeFromString`序列化`json`是，第一个参数要传递对应`json`的数据类的`serizlizer`对象
4. 注解`@Serializable`会将添加该注解的类的序列化工具生成到对应类的伴生对象中，添加该注解的类`PersonForSerizlize`反编译后的关键代码如下：
   

    ```kotlin
    public final class PersonFroSerialize {
        ...
    @NotNull
    public static final PersonFroSerialize.Companion Companion = new PersonFroSerialize.Companion((DefaultConstructorMarker)null);
    ...
        public static final class Companion {
        private Companion() {
        }

        // $FF: synthetic method
        public Companion(DefaultConstructorMarker $constructor_marker) {
            this();
        }

        @NotNull
        public final KSerializer serializer() {
            return (KSerializer)PersonFroSerialize.$serializer.INSTANCE;
        }
    }
 
    public static final class $serializer implements GeneratedSerializer {
        @NotNull
        public static final PersonFroSerialize.$serializer INSTANCE;
        // $FF: synthetic field
        private static final SerialDescriptor $$serialDesc;

        private $serializer() {
        }
        ...
    }
    }
    ```

### 2. 带默认参数的json序列化

#### 1. Gson处理默认参数

```kotlin
//Gson
fun main() {
    val jsonStr = """{"name":"Lee"}"""
    val person = PersonWithDefaults("Lee")

    //Gson
    val gson = Gson()
    println(gson.toJson(person))
    println(gson.fromJson(jsonStr, PersonWithDefaults::class.java))
}


data class PersonWithDefaults(val name: String, val age: Int = 18)
```

打印结果：

```java
{"name":"Lee","age":18}
PersonWithDefaults(name=Lee, age=0)
```

**说明：**

1. `Gson`在将带有默认参数的数据类序列化时会将默认参数带上
2. `Gson`在反序列化时会通过反射来判断有没有默认无参的构造方法，如果没有的话会通过`UnSafe`类直接创建对象，所以反序列化时数据类的默认参数不会起作用


#### 2. Moshi处理默认参数

```kotlin
//Moshi
fun main() {
    val jsonStr = """{"name":"Lee"}"""

    //方法一
     val moshi = Moshi.Builder()
//            .add(KotlinJsonAdapterFactory())
            .build()
    val jsonAdapter = moshi.adapter(PersonWithDefaultsAnnotation::class.java)
    println(jsonAdapter.toJson(PersonWithDefaultsAnnotation("Lee")))
    println(jsonAdapter.fromJson(jsonStr))
    println()
    //方法二
    val moshi = Moshi.Builder()
            .add(KotlinJsonAdapterFactory())
            .build()
    val jsonAdapter = moshi.adapter(PersonWithDefaults::class.java)
    println(jsonAdapter.toJson(person))
    println(jsonAdapter.fromJson(jsonStr))
}

@JsonClass(generateAdapter = true)
data class PersonWithDefaultsAnnotation(val name: String, val age: Int = 18)

data class PersonWithDefaults(val name: String, val age: Int = 18)
```

打印结果：

```java
{"name":"Lee","age":18}
PersonWithDefaults(name=Lee, age=18)

{"name":"Lee","age":18}
PersonWithDefaults(name=Lee, age=18)
```

**说明：**

1. 由打印结果可以看出，`Moshi`序列化和反序列化都可以支持默认值
2. 方法一需要使用注解处理器，还需要添加远程依赖包：`com.squareup.moshi:moshi-kotlin-codegen:1.11.0`
3. 注解`@JsonClass(generateAdapter = true)`会在路径：`build/generated/source/kapt/debug/注解文件所在包名/`下生成`JsonAdapter`，用以对被注解的类进行序列化与反序列化，路径如下图所示：
    ![moshi生成的序列化adapter路径](/kotlin学习系列五/kotlin_moshi_generated_path.png)
4. 方法二不需要添加注解，但是需要远程依赖包：`com.squareup.moshi:moshi-kotlin:1.11.0`，之后在创建`Moshi`对象时添加`KotlinJsonAdapterFactory`对象

#### 3. K.S处理默认参数

```kotlin
//kotlinx.serialization
fun main() {
    val jsonStr = """{"name":"Lee"}"""

    println(Json.encodeToString(PersonWithDefaults.serializer(),PersonWithDefaults("Lee")))
    println(Json.decodeFromString(PersonWithDefaults.serializer(),jsonStr))
}

@Serializable
data class PersonWithDefaults(val name: String, val age: Int = 18)
```

打印结果：

```java
{"name":"Lee"}
PersonWithDefaults(name=Lee, age=18)
```

**说明：**

1. 由打印结果可以看出，`kotlinx.serialization`在序列化时忽略了默认参数，反序列化时支持默认参数
2. 注解`@Serializable`会在数据类的伴生对象中创建序列化和反序列化的工具

### 3. 带init块或成员初始化的数据类序列化

#### 1. Gson序列化带init块的数据类

```kotlin
fun main() {
    val gson = Gson()
    println(gson.toJson(PersonWithInits("Lee", 18)))
    val person = gson.fromJson(jsonStr, PersonWithInits::class.java)
    println(person)
    println(person.firstName)
}

@DataClassAnnotation
data class PersonWithInits(val name: String, val age:Int) {
    init {
        println("PersonWithInits#init()")
    }
    val firstName by lazy {
        name.split(" ")[0]
    }
}
```

打印结果：

```java
PersonWithInits#init()
{"firstName$delegate":{"initializer":{},"_value":{}},"name":"Lee","age":18}
PersonWithInits#init()
PersonWithInits(name=Lee, age=18)
Lee
```

**说明：**

1. `Gson`每次构造数据类时会检查数据类有没有默认无参构造器，若没有的话，则会通过`UnSafe`类创建数据类实例。通过`UnSafe`创建的实例不会调用`init`块和成员初始化代码
2. 数据类默认是没有无参构造器的，可以使用前面数据类章节中提到的[`NoArg`插件](#jumpNoArg)
3. 如上代码所示，数据类添加了使用`NoArg`插件的注解`@DataClassAnnotation`使用该数据类有了无参构造方法
4. 如上打印结果所示，数据类有了无参构造方法，则`Gson`可以执行数据类中的`init`块的代码

#### 2. Moshi序列化带init块的数据类

```kotlin
fun main() {
    val jsonStr = """{"name":"Lee", "age":18}"""

    //方法一
    val moshi = Moshi.Builder()
//            .add(KotlinJsonAdapterFactory())
            .build()
    val jsonAdapter = moshi.adapter(PersonWithInitsAnnotation::class.java)
    println(jsonAdapter.toJson(PersonWithInitsAnnotation("Lee", 18)))
    val person = jsonAdapter.fromJson(jsonStr)
    println(person)
    println(person?.firstName)
    //方法二
    val moshi2 = Moshi.Builder()
            .add(KotlinJsonAdapterFactory())
            .build()
    val jsonAdapter2 = moshi2.adapter(PersonWithInits::class.java)
    println(jsonAdapter2.toJson(PersonWithInits("Lee", 18)))

    val person2 = jsonAdapter2.fromJson(jsonStr)
    println(person2)
    println(person2?.firstName)

}

@JsonClass(generateAdapter = true)
data class PersonWithInitsAnnotation(val name: String, val age:Int) {
    init {
        println("PersonWithInits#init()")
    }
    val firstName by lazy {
        name.split(" ")[0]
    }
}
data class PersonWithInits(val name: String, val age:Int) {
    init {
        println("PersonWithInits#init()")
    }
    val firstName by lazy {
        name.split(" ")[0]
    }
}
```

打印结果：

```java
PersonWithInits#init()
{"name":"Lee","age":18}
PersonWithInits#init()
PersonWithInitsAnnotation(name=Lee, age=18)
Lee
PersonWithInits#init()
{"name":"Lee","age":18}
PersonWithInits#init()
PersonWithInits(name=Lee, age=18)
Lee
```

**说明：**

1. 使用注解`@JsonClass`和使用反射`KotlinJsonAdapterFactory`这两种方式都会执行`init`块和成员变量的初始化
2. `Moshi`反序列化时会调用数据类的构造器生成数据类对象

#### 3. K.S序列化带init块的数据类

```kotlin
fun main() {
    val jsonStr = """{"name":"Lee", "age":18}"""

    println(Json.encodeToString(PersonWithInits.serializer(), PersonWithInits("Lee", 18)))
    val person = Json.decodeFromString(PersonWithInits.serializer(), jsonStr)
    println(person)
    println(person.firstName)
}
@Serializable
data class PersonWithInits(val name: String, val age:Int) {
    init {
        println("PersonWithInits#init()")
    }
    val firstName by lazy {
        name.split(" ")[0]
    }
}

```

打印结果：

```java
PersonWithInits#init()
{"name":"Lee","age":18}
PersonWithInits#init()
PersonWithInits(name=Lee, age=18)
Lee
```

**说明：**

1. `kotlinx.serialization`会执行`init`块和成员变量的初始化
2. 注解`@Serializable`会在数据类的伴生对象中生成序列化和反序列化的工具

## 3. 框架对比

||Gson|Moshi|K.S|
|--|--|--|--|
|空类型|否|反射、注解|是|
|默认值|否|反射、注解|否|
|init块|NoArg插件|反射、注解|是|
|java类|是|是|否|
|跨平台|否|否|是|

**说明：**

1. 如果使用的纯`java`工程，建议使用`Gson`和`Moshi`
2. 如果使用的`java`和`kotlin`混合工程，建议使用`Moshi`
3. 如果使用的纯`kotlin`工程，建议使用`K.S`

# 十三、应用 -  递归整型列表

```kotlin
fun main() {
    //[0,1,2,3]
    //实现方式一
//    val list = IntList.Cons(0, IntList.Cons(1, IntList.Cons(2, IntList.Cons(3, IntList.Nil))))
    val list = intListOf(0,1,2,3)
    println(list)
    println(list.joinToString('-'))
    println(list.sum())

    val (first, second, third) = list
    println("$first, $second, $third")
}

//实现嵌套列表
fun intListOf(vararg ints: Int): IntList {
    return when(ints.size) {
        0 -> IntList.Nil
        else -> {
            IntList.Cons(
                    ints[0],
                    intListOf(*(ints.slice(1 until ints.size).toIntArray()))
            )
        }
    }
}

//扩展方法--求和
fun IntList.sum(): Int {
    return when (this) {
        IntList.Nil -> 0
        is IntList.Cons -> {
            return head + tail.sum()
        }
    }
}

operator fun IntList.component1(): Int? {
    return when(this) {
        IntList.Nil -> null
        is IntList.Cons -> {
            head
        }
    }
}
operator fun IntList.component2(): Int? {
    return when(this) {
        IntList.Nil -> null
        is IntList.Cons -> {
            tail.component1()
        }
    }
}

operator fun IntList.component3(): Int? {
    return when(this) {
        IntList.Nil -> null
        is IntList.Cons -> {
            tail.component2()
        }
    }
}
sealed class IntList {
    //链的最后一个
    object  Nil: IntList() {
        override fun toString(): String {
            return "Nil"
        }
    }
    data class Cons(val head: Int, val tail:IntList): IntList() {
        override fun toString(): String {
            return "$head, $tail"
        }
    }

    fun joinToString(sep: Char = ','):String {
        return when(this) {
            Nil -> {
                "Nil"
            }
            is Cons -> {
                "${head}$sep${tail.joinToString(sep)}"
            }
        }
    }
}
```

打印结果：

```java
0, 1, 2, 3, Nil
0-1-2-3-Nil
6
0, 1, 2
```

**说明：**

1. 在一个数组前加星号，表示将该数组元素变成一个个的参数传递给变长参数 *(vararg)*
2. 因为密封类的子类有限，所以可以使用`when`语句判断密封类
3. 密封类中的`joinToString()`方法、它的扩展方法`sum()`和数据类`Cons`的`toString()`方法都使用了递归操作
4. 只要有`component()`方法就可以解构,列表定义了几个`component`方法，就可以解构几个元素



# 十四、参考文章

1. [https://www.runoob.com/w3cnote/java-inner-class-intro.html](https://www.runoob.com/w3cnote/java-inner-class-intro.html)
2. [https://www.kotlincn.net/docs/reference/compiler-plugins.html](https://www.kotlincn.net/docs/reference/compiler-plugins.html)
3. [https://www.kotlincn.net/docs/reference/compiler-plugins.html](https://www.kotlincn.net/docs/reference/compiler-plugins.html)
4. [https://www.kotlincn.net/docs/reference/kapt.html](https://www.kotlincn.net/docs/reference/kapt.html)
  